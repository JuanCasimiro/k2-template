---
name: migraciones-db
description: Patrón seguro para escribir, aplicar y revisar cambios de esquema en Postgres (referencia Supabase) — migraciones versionadas e inmutables, plantilla con RLS multi-tenant, hardening de funciones, CHECK en dos pasos, índices en FKs y verificación post-apply. Usala cuando se toca el esquema de la base. Triggers on "migración", "nueva tabla", "alter table", "agregar columna", "trigger", "policy", "RPC", "cambio de esquema", "db push".
---

# Migraciones de base de datos — patrón seguro

Stack de referencia: Postgres en Supabase. Todo lo que es SQL aplica a cualquier Postgres; lo específico de Supabase (`auth.uid()`, roles `anon`/`authenticated`/`service_role`, `supabase db push`) está marcado.

## Reglas base

- **El esquema se cambia SOLO con migraciones versionadas** (`YYYYMMDDHHMMSS_descripcion_corta.sql`). Un `schema.sql` de snapshot es referencia para leer, no algo que se ejecute: editarlo esperando que aplique no hace nada.
- **El timestamp debe ser mayor al de la última migración existente.** Si no, el orden de aplicación se rompe en otros entornos.
- **Una migración aplicada en producción es inmutable.** Los arreglos van en una migración nueva, nunca editando una ya aplicada (los entornos que ya la corrieron no la vuelven a correr y divergen).
- **Escribí el `.sql` en el repo ANTES de aplicarlo** (por CLI o por un MCP). Si aplicás "a mano" y después te olvidás, el repo deja de describir la base real.
- Scripts sueltos de SQL manual son legacy: lo nuevo va como migración.

## Plantilla

```sql
-- 20260101120000_crear_reservas.sql
-- Qué hace y por qué. Idempotente donde se pueda.

-- 1) DDL
create table if not exists public.reservas (
  id uuid primary key default gen_random_uuid(),
  tenant_id uuid not null references public.tenants(id) on delete cascade,
  producto_id uuid not null references public.productos(id) on delete cascade,
  estado text not null default 'activa' check (estado in ('activa', 'cumplida', 'cancelada')),
  cantidad integer not null check (cantidad > 0),
  created_at timestamptz not null default now()
);

-- 2) RLS (obligatoria en toda tabla con tenant_id)
alter table public.reservas enable row level security;

-- UNA permisiva por (rol, acción), siempre TO authenticated y con el tenant en el predicado.
-- get_my_tenant_id() / is_superadmin() son helpers propios (ver skill rls-multi-tenant).
create policy "reservas_select" on public.reservas
  for select to authenticated using (
    tenant_id = (select public.get_my_tenant_id()) or (select public.is_superadmin()));

create policy "reservas_insert" on public.reservas
  for insert to authenticated with check (
    tenant_id = (select public.get_my_tenant_id()));

create policy "reservas_update" on public.reservas
  for update to authenticated
  using      (tenant_id = (select public.get_my_tenant_id()))
  with check (tenant_id = (select public.get_my_tenant_id()));

create policy "reservas_delete" on public.reservas
  for delete to authenticated using (
    tenant_id = (select public.get_my_tenant_id()) and (select public.get_my_role()) = 'owner');

-- Gate "extra" (ej. suscripción vigente) => SIEMPRE restrictiva (AND), nunca permisiva (OR amplía).
create policy "reservas_subscription_required" on public.reservas
  as restrictive for insert to authenticated
  with check ((select public.is_subscription_valid()) or (select public.is_superadmin()));

-- 3) Índices: tenant y toda FK usada en joins/filtros
create index if not exists idx_reservas_tenant on public.reservas(tenant_id);
create index if not exists idx_reservas_producto on public.reservas(producto_id);
```

> **Por qué las policies de esta plantilla son así:** las permisivas se combinan con OR y la más laxa gana. Una vez una policy "extra" de suscripción, permisiva y sin `tenant_id` en el predicado, dejó que cualquier cliente con suscripción escribiera filas de OTROS clientes. Detalle completo y checklist en la skill `rls-multi-tenant`.

## Funciones y triggers: hardening obligatorio

```sql
create or replace function public.mi_trigger() returns trigger
  language plpgsql
  security definer
  set search_path = public        -- evita search_path hijacking
as $$ begin /* ... */ return new; end; $$;

revoke all on function public.mi_trigger() from public;
grant execute on function public.mi_trigger() to service_role, postgres;
```

**`REVOKE` siempre `FROM PUBLIC`, no `FROM anon`.** Postgres otorga EXECUTE a PUBLIC por defecto y `anon`/`authenticated` heredan de PUBLIC: revocar solo de `anon` no tiene efecto.

```sql
-- MAL: no hace nada (anon hereda de PUBLIC)
revoke execute on function public.fn() from anon;
-- BIEN
revoke execute on function public.fn() from public;
grant  execute on function public.fn() to authenticated;  -- si la usan usuarios logueados
```

Excepción: los helpers que se invocan dentro de policies (`get_my_tenant_id()`, etc.) no se revocan para `authenticated`/`anon`: hacerlo rompe el acceso con *permission denied*.

## Triggers existentes: volcalos y versionalos antes de tocar

Si una tabla tiene un trigger crítico (ej. uno que recalcula saldos o stock) cuyo cuerpo **no está en el repo** porque se creó a mano:

1. Volcá la definición viva: `select pg_get_functiondef('public.mi_trigger()'::regprocedure);`
2. Versionala tal cual en una migración **antes** de modificarla.
3. Recién ahí cambiala. Modificar un trigger sin haberlo leído es cambiar lógica de negocio a ciegas.

Además, un trigger que hace `UPDATE` sobre otra tabla en cada edición aunque no cambie nada genera lock contention en cascada (varios usuarios → writes cancelados en silencio). Guard al inicio:

```sql
if tg_op = 'UPDATE' and new.estado is not distinct from old.estado then
  return new;
end if;
```

## CHECK sobre datos existentes: dos pasos

Para no bloquear inserts en vuelo ni fallar contra datos legacy:

```sql
alter table public.reservas add constraint reservas_cantidad_pos check (cantidad > 0) not valid; -- no valida lo viejo
-- (correr el backfill que arregla lo viejo — ver skill backfill-seguro)
alter table public.reservas validate constraint reservas_cantidad_pos;                            -- valida en el segundo paso
```

## FK sin índice: Postgres no lo crea

Postgres **no** indexa la columna hija de una FK. Toda FK que participe de un JOIN o WHERE necesita índice explícito; sin él, el borrado del padre y los joins escanean la tabla hija entera (una tabla de movimientos con 2M de filas sin índice en su FK tarda segundos por consulta).

```sql
-- Compuesto para la query principal (reservas activas por producto dentro de un tenant)
create index idx_reservas_tenant_producto_estado on public.reservas(tenant_id, producto_id, estado);
-- O parcial si el filtro es constante
create index idx_reservas_activas on public.reservas(producto_id) where estado = 'activa';
```

Auditoría post-apply (no debe devolver filas para FKs usadas en JOIN/filtro):

```sql
select c.conname, a.attname
from pg_constraint c
join pg_attribute a on a.attrelid = c.conrelid and a.attnum = any(c.conkey)
where c.contype = 'f' and c.conrelid = 'public.reservas'::regclass
  and not exists (
    select 1 from pg_index i
    where i.indrelid = c.conrelid and a.attnum = any(i.indkey)
  );
```

## Checklist antes de pushear

- [ ] Timestamp del archivo > última migración; el `.sql` está en el repo antes de aplicarse.
- [ ] RLS habilitada + policies select/write si la tabla tiene `tenant_id`.
- [ ] Índice en `tenant_id` y en **cada FK usada en JOIN/WHERE**.
- [ ] Funciones `security definer` con `set search_path = public` + `revoke ... from public`.
- [ ] ¿Toca un trigger existente? Volcalo y versionalo primero.
- [ ] Rollback escrito o pensado (qué migración inversa correría si sale mal).
- [ ] Idempotente (`if not exists`, `create or replace`) donde aplique.
- [ ] Tablas `_backup_*`: `enable row level security` en la misma migración.
- [ ] Columnas numéricas críticas (`cantidad`, `precio`, mínimos) con `CHECK (col > 0)` o `>= 0` según el dominio.
- [ ] Campos de estado con `CHECK (estado in (...))` que documenta todos los valores permitidos.
- [ ] UNIQUE donde el código asume unicidad (si hay un `getOrCreate`, tiene que haber un UNIQUE que lo respalde).

## Verificación post-apply

- La migración figura como aplicada (`supabase migration list` o equivalente).
- Test de aislamiento: consultar la tabla con un `tenant_id` distinto no devuelve filas ajenas.
- La query de FKs sin índice devuelve 0 filas.
- Si cambian tipos que consume el front, regenerar tipos y compilar (`tsc --noEmit`).
- En Supabase: `get_advisors` de seguridad y performance sin hallazgos nuevos.

Skills relacionadas: `rls-multi-tenant`, `backfill-seguro`, `queries-seguras`, `integridad-ledger`.
