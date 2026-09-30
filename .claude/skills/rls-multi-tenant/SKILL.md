---
name: rls-multi-tenant
description: Auditoría de Row Level Security para apps multi-tenant en Postgres (referencia Supabase) — checklist para tablas, RPCs y vistas nuevas, policies permisivas vs restrictivas (el agujero cross-tenant más fácil de meter), performance de RLS, IDOR y gates de PR. Usala al crear o tocar cualquier superficie de datos con tenant_id. Triggers on "RLS", "row level security", "policy", "política", "nueva tabla", "nueva RPC", "nueva vista", "aislamiento entre clientes", "multi-tenant", "cross-tenant".
---

# RLS multi-tenant — checklist de auditoría

## Por qué importa

En una app multi-tenant donde **cada cliente es un tenant aislado exclusivamente por RLS sobre `tenant_id`**, una tabla sin RLS o con una policy mal escrita expone los datos de todos los clientes a cualquier usuario autenticado. Si el cliente habla directo con la base (Supabase/PostgREST), no hay otra capa de seguridad en el medio.

Convenciones de esta skill: `tenant_id` es la columna de aislamiento; `get_my_tenant_id()`, `is_superadmin()`, `get_my_role()` son helpers `STABLE` propios que leen el tenant/rol del usuario actual (reemplazalos por los de tu proyecto). `TO authenticated` / `anon` son roles de Supabase; en otro Postgres, usá tus roles de aplicación.

## 1. Checklist para cada tabla nueva

### RLS habilitado y forzado
```sql
alter table nueva_tabla enable row level security;
alter table nueva_tabla force row level security;  -- aplica RLS también al owner de la tabla
```

### Policies: patrón canónico con initplan caching

> **Nunca** `select tenant_id from profiles where id = auth.uid()` dentro de una policy: se ejecuta por cada fila. Una vez, con 40k filas, eso fue un full scan por fila → `statement timeout` → la app entera "no guardaba". Siempre helper envuelto en `(select fn())`, para que el planner lo evalúe **una vez por query**.

```sql
create policy "tenant_select" on nueva_tabla
  for select to authenticated using (
    tenant_id = (select public.get_my_tenant_id())
    or (select public.is_superadmin())
  );

create policy "tenant_insert" on nueva_tabla
  for insert to authenticated with check (
    tenant_id = (select public.get_my_tenant_id())
  );

create policy "tenant_update" on nueva_tabla
  for update to authenticated
  using      (tenant_id = (select public.get_my_tenant_id()) or (select public.is_superadmin()))
  with check (tenant_id = (select public.get_my_tenant_id()));

create policy "tenant_delete" on nueva_tabla
  for delete to authenticated using (
    (tenant_id = (select public.get_my_tenant_id()) and (select public.get_my_role()) = 'owner')
    or (select public.is_superadmin())
  );
```

### Tabla sin `tenant_id` directo (FK a un padre que sí lo tiene)
```sql
create policy "tenant_select" on hija
  for select to authenticated using (
    exists (
      select 1 from public.padre p
      where p.id = hija.padre_id
        and p.tenant_id = (select public.get_my_tenant_id())
    )
    or (select public.is_superadmin())
  );
```
Preferí denormalizar `tenant_id` en la hija si la tabla es grande: evita el join en cada evaluación.

---

## Permisivas vs restrictivas — el agujero cross-tenant más fácil de meter

> **Caso real:** un set de policies "Subscription required for insert/update/delete X" en varias tablas core (ventas, productos, proveedores, personas…) abrió **escritura y lectura entre tenants**. Eran PERMISSIVE y su predicado (`is_subscription_valid()`) no referenciaba `tenant_id`. Quien tenía suscripción vigente pasaba el OR de esa policy y escribía filas de cualquier otro cliente. La intención era "sumar un requisito"; el efecto fue ampliar el acceso.

### Regla 1 — las permisivas se combinan con OR: la más laxa gana
Si una tabla tiene 2+ policies **permisivas** para el mismo `(rol, acción)`, alcanza con que UNA permita para que la fila pase.
- **Nunca** apiles una permisiva "extra" pensando que restringe: **agranda** el acceso.
- **Apuntá a UNA permisiva por `(rol, acción)`.** Varios criterios (superadmin, manager, dueño) se combinan con `OR` **dentro de una misma policy**.

### Regla 2 — todo gate "extra" va `AS RESTRICTIVE` (se combina con AND)
```sql
-- MAL: permisiva sin scope de tenant => OR con la granular => cualquiera con suscripción
--      escribe filas de OTROS tenants.
create policy "subscription_required_insert" on x
  for insert with check (is_subscription_valid() or is_superadmin());

-- BIEN: candado extra que se SUMA (AND) a la policy granular, sin ampliar acceso.
create policy "x_subscription_required" on x
  as restrictive for insert to authenticated
  with check ((select public.is_subscription_valid()) or (select public.is_superadmin()));
```
No uses `RESTRICTIVE FOR ALL` si querés que SELECT siga funcionando sin el gate (un tenant con suscripción vencida debería poder LEER su data): creá restrictivas por `INSERT/UPDATE/DELETE`.

### Regla 3 — toda policy de escritura referencia al tenant en su predicado
El `USING`/`WITH CHECK` de INSERT/UPDATE/DELETE tiene que atar la fila al tenant (`tenant_id = (select get_my_tenant_id())` o `EXISTS` correlacionado al padre). Un predicado **global del caller** (`is_subscription_valid()`, `can_edit()`, `get_my_role()='owner'`) **por sí solo no aísla**: aplica a filas de todos los tenants. `DELETE` y `UPDATE` miran `USING`: un `USING` sin `tenant_id` = borrado/edición cross-tenant aunque el `WITH CHECK` esté scopeado.

### Regla 4 — policies de la app `TO authenticated`, no `public`
`public` incluye al rol `anon`. Una policy de la app en `public` se solapa con las policies de flujos anónimos (booking, formularios públicos) y ensucia los advisors. Los flujos anónimos van en policies separadas `TO anon` que **no** referencian helpers de tenant.

### Una sola policy por operación (performance)
Las permisivas se evalúan todas. Tres policies SELECT no son "más seguras": son 3× el costo por fila (en una auditoría real aparecieron 400+ instancias, y fue el principal overhead en tablas grandes). Antes de crear una policy en tabla existente:
```sql
select policyname, cmd from pg_policies where tablename = '<tabla>';
```
Si ya hay una para ese `cmd`, modificala en vez de agregar otra.

---

## RPCs (funciones)

```sql
create or replace function mi_funcion(p_tenant_id uuid)
returns void
language plpgsql
security invoker                         -- preferir INVOKER: la RLS del caller aplica
set search_path = public, extensions     -- previene search_path injection
as $$
begin
  -- Si fuera DEFINER (bypassea RLS), verificar el tenant explícitamente:
  if p_tenant_id is distinct from (select public.get_my_tenant_id()) then
    raise exception 'unauthorized';
  end if;
  -- lógica...
end;
$$;
```

- **Preferí `SECURITY INVOKER`.** Si necesitás `DEFINER` (leer tablas que el caller no ve), el check del tenant adentro es obligatorio, más `set search_path` y `revoke execute ... from public` (no `from anon`: anon hereda de PUBLIC).
- Helpers usados dentro de policies no se revocan a `anon`/`authenticated`: rompe el acceso con *permission denied*.

## Vistas

Una vista corre con los permisos de su dueño y **salta la RLS** salvo que la crees `with (security_invoker = true)` (Postgres 15+). Toda vista sobre tablas con RLS debe declararlo.

Vistas con varias subqueries correlacionadas se re-ejecutan por fila (4 subqueries = 4 scans por fila). Consolidá con `LATERAL`:
```sql
-- MAL: 2 scans de reservas por producto
select p.*,
  (select coalesce(sum(cantidad),0) from reservas where producto_id=p.id and estado='activa' and canal='web')    as reservado_web,
  (select coalesce(sum(cantidad),0) from reservas where producto_id=p.id and estado='activa' and canal='local') as reservado_local
from productos p;

-- BIEN: un LATERAL, un scan por producto
select p.*, r.reservado_web, r.reservado_local
from productos p
left join lateral (
  select coalesce(sum(cantidad) filter (where canal='web'),   0) as reservado_web,
         coalesce(sum(cantidad) filter (where canal='local'), 0) as reservado_local
  from reservas where producto_id = p.id and estado = 'activa'
) r on true;
```

---

## Performance de RLS — policies correctas que igual tumban la DB

La RLS se evalúa **por cada fila** que toca la query. Una policy lenta sobre una tabla grande = `statement timeout` → HTTP 500, que el front muestra como "no se guarda", "no se ve", "tarda en cargar", no como error de DB. Auditar performance es tan obligatorio como auditar aislamiento.

1. **`EXISTS` correlacionado, nunca `IN (select … from tabla)`.** Caso real: `vehiculos_select` hacía `persona_id in (select persona_id from clientes where tenant_id = …)`; tras una importación a ~40k clientes, el planner escaneaba toda `clientes` en cada fila (1,24 s para 267 filas, costo estimado 11,5M).
   ```sql
   -- BIEN: Index Only Scan, Heap Fetches: 0  (1,24 s → 42 ms, mismo conjunto de filas)
   exists (
     select 1 from clientes c
     where c.persona_id = vehiculos.persona_id
       and c.tenant_id = (select get_my_tenant_id())
   )
   ```
2. **Funciones estables envueltas en `(select fn())`** — `auth.uid()`, `get_my_tenant_id()`, `is_superadmin()`, etc. Una vez por query, no por fila.
   ```sql
   tenant_id = get_my_tenant_id()            -- MAL: re-evalúa por fila
   tenant_id = (select get_my_tenant_id())   -- BIEN
   ```
3. **Índice de soporte obligatorio** para toda columna usada en un `EXISTS`/JOIN de policy (ej. `clientes(tenant_id, persona_id)`). Policy sin índice sobre tabla grande = timeout garantizado apenas crece.
4. **Cuidado con los embeds de PostgREST:** la RLS de una tabla embebida se evalúa dentro de las queries de las padres. Una policy lenta en `vehiculos` contaminó todas las queries de `turnos` y `ordenes` que la embebían. No afecta solo a su tabla.

**Cómo verificar:** `EXPLAIN ANALYZE` de la query real (con embeds) **bajo el rol de la app**, no como owner (el owner puede saltarse RLS). Buscás `Index Scan` / `Index Only Scan`, sin scans de miles de filas de la tabla que consulta la policy.
```sql
set role authenticated;
set request.jwt.claims = '{"sub": "<uuid-de-un-usuario-de-prueba>"}';  -- simular usuario/tenant
explain analyze select * from turnos;   -- la query que dispara el front, con sus embeds
reset role;
```

---

## Tablas de backup y tablas sin RLS

- Toda tabla `_backup_*` creada para un script de migración **lleva RLS habilitada en la misma migración**. Sin policies = deny-by-default (correcto). Sin RLS, PostgREST expone el backup a `anon` y `authenticated` vía la API REST.
  ```sql
  alter table public._backup_ventas_20260101 enable row level security;  -- sin policies = nadie accede vía API
  ```
- Por defecto **toda tabla nueva lleva RLS**. Una tabla sin RLS legítima (catálogo público, por ejemplo) se documenta con un comentario en la migración.

## IDOR — defensa en profundidad en el cliente

RLS protege a nivel DB, pero si falla (policy mal escrita, contexto `service_role` en una Edge Function) la única defensa restante es el filtro explícito del cliente.

```ts
// MAL — depende 100% de RLS; un bug en la policy es un IDOR
supabase.from('cola_sync').update({ decision }).eq('id', itemId);

// BIEN — defensa en profundidad
supabase.from('cola_sync').update({ decision }).eq('id', itemId).eq('tenant_id', tenantId);
```

**Regla:** toda función de servicio que hace UPDATE/DELETE por `id` recibe `tenantId` y lo usa en el filtro. Excepción documentada: server-side con `service_role` y check de autorización explícito.

---

## Verificación post-migración

```sql
-- RLS habilitado y forzado
select tablename, rowsecurity, forcerowsecurity
from pg_tables where schemaname = 'public' and tablename = 'nueva_tabla';

-- Policies creadas
select policyname, cmd, permissive, roles, qual, with_check
from pg_policies where tablename = 'nueva_tabla';

-- Tablas públicas sin RLS (ninguna nueva sin justificación)
select tablename from pg_tables where schemaname = 'public' and rowsecurity = false;

-- Múltiples permisivas por (tabla, rol, acción): debe dar 0 filas.
-- Se expande 'public' a anon+authenticated y ALL a las 4 acciones para no perder solapes.
with expanded as (
  select p.tablename, p.policyname, p.cmd, r.role_name
  from pg_policies p
  cross join lateral (
    select unnest(case when p.roles = '{public}' then array['anon','authenticated']::text[]
                       else p.roles::text[] end) as role_name
  ) r
  where p.schemaname = 'public' and p.permissive = 'PERMISSIVE'
), acts as (
  select e.tablename, e.policyname, e.role_name, a.act as action
  from expanded e
  cross join lateral (values ('SELECT'),('INSERT'),('UPDATE'),('DELETE')) a(act)
  where e.cmd = 'ALL' or e.cmd = a.act
)
select tablename, role_name, action, count(*), string_agg(policyname, ', ')
from acts group by 1,2,3 having count(*) > 1;
```

## Gate de PR

```bash
# La migración nueva debe habilitar RLS
grep -n "ENABLE ROW LEVEL SECURITY" supabase/migrations/<nueva>.sql
# RPCs: SECURITY DEFINER debe ir con search_path
grep -n "SECURITY DEFINER\|search_path" supabase/migrations/<nueva>.sql
# REVOKE debe ser FROM PUBLIC, no FROM anon (si aparece, corregir)
grep -n "REVOKE.*FROM anon" supabase/migrations/<nueva>.sql
# Tablas backup sin RLS (no debe aparecer nada)
grep -n "_backup_" supabase/migrations/<nueva>.sql | grep -v -i "ENABLE ROW LEVEL SECURITY"
# IDOR: updates/deletes por id sin tenantId (revisar a mano cada resultado)
grep -rn "\.update\|\.delete" services/ --include="*.ts" | grep "\.eq('id'"
```

Sobre la DB ya aplicada (o un branch): la query de **múltiples permisivas** debe dar 0 filas, y en Supabase `get_advisors` de seguridad y performance no deben mostrar `multiple_permissive_policies` nuevos ni `rls_policy_always_true`.

**Bloqueantes de merge:** tabla nueva sin RLS · gate "extra" permisivo · `USING`/`WITH CHECK` de escritura sin `tenant_id` (ni `EXISTS` al padre) · `REVOKE ... FROM anon` sin `FROM PUBLIC`.

Skills relacionadas: `migraciones-db`, `queries-seguras`, `pre-ship`, `auditoria-cto`.
