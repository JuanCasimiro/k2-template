---
name: backfill-seguro
description: Patrón para correr un backfill o UPDATE masivo sobre filas existentes sin romper producción — dry-run, respaldo reversible, idempotencia, batches, cuidado con triggers, verificación y orden de deploy. Usala al normalizar datos históricos, poblar una columna nueva o deduplicar. Triggers on "backfill", "UPDATE masivo", "normalizar datos históricos", "poblar columna", "migrar datos", "deduplicar", "corregir datos viejos".
---

# Backfill seguro

Un backfill toca datos reales de clientes en producción y no hay "deshacer" gratis. Seguí este orden siempre. Todo el SQL aplica a cualquier Postgres.

## 1. Caracterizar antes de tocar (dry-run de lectura)

Contá exactamente qué vas a cambiar con un `SELECT` que use la **misma condición** que el `UPDATE`:

```sql
select count(*) as filas_afectadas
from public.movimientos
where cantidad < 0;   -- la misma condición que va a usar el update
```

Mirá una muestra real: `select * from public.movimientos where cantidad < 0 limit 20;`. Si el número o la muestra te sorprenden, parás y entendés por qué antes de seguir.

## 2. Respaldo reversible

Antes de un UPDATE no trivial, snapshot de las filas afectadas:

```sql
create table if not exists public._backup_movimientos_20260101 as
select * from public.movimientos where cantidad < 0;

alter table public._backup_movimientos_20260101 enable row level security; -- sin policies = nadie accede por la API
```

Dejá escrito cómo revertir desde ese backup (un `update ... from _backup_... b where t.id = b.id`). Borrá el backup recién cuando el cambio esté verificado y estable. Ojo: un backup sin RLS queda expuesto por la API REST (ver `rls-multi-tenant`).

## 3. Idempotencia

El UPDATE tiene que poder correrse dos veces sin daño.
- `set cantidad = abs(cantidad)` → idempotente (correrlo otra vez no cambia nada).
- `set contador = contador + 1` → **no** idempotente: acumula. Reescribilo a un valor absoluto, no incremental.

Si no se puede, agregá un marcador y filtrá por él: `where backfilled_at is null` y en el set `backfilled_at = now()`.

## 4. Batches en tablas grandes

Para tablas con muchas filas, por lotes, para no tomar locks largos ni inflar el WAL:

```sql
update public.movimientos set cantidad = abs(cantidad)
where id in (select id from public.movimientos where cantidad < 0 limit 5000);
-- repetir hasta filas_afectadas = 0 (pausa corta entre lotes si hay carga)
```

## 5. Cuidado con los triggers

¿La tabla tiene triggers `BEFORE/AFTER UPDATE`? Un backfill los dispara y puede producir efectos colaterales: recalcular saldos, escribir logs, mandar webhooks, encolar syncs a sistemas externos.
- Confirmá qué triggers existen (`select pg_get_functiondef(...)` / `select tgname from pg_trigger where tgrelid = 'public.movimientos'::regclass and not tgisinternal;`).
- Ejemplo: si el trigger que recalcula stock corre `AFTER INSERT` (no `UPDATE`), un UPDATE de normalización no recalcula nada. Verificá que **sigue siendo así** leyendo la definición viva, no la memoria.

## 6. Aplicar y verificar

Volvé a correr el `SELECT count(*)` de caracterización: tras el backfill debe dar **0** (o el target esperado). Chequeá invariantes de negocio: los dashboards muestran los mismos totales que antes; el saldo/stock no cambió si no debía cambiar.

## Checklist

- [ ] `count` y muestra revisados antes.
- [ ] Backup de las filas afectadas creado (con RLS).
- [ ] UPDATE idempotente (o con marcador).
- [ ] Batches si la tabla es grande.
- [ ] Triggers de la tabla leídos y descartados como riesgo.
- [ ] Verificación posterior: count = 0 e invariantes de negocio intactos.
- [ ] Plan de rollback escrito.
- [ ] Backup borrado solo después de que todo estabilizó.

## Orden de deploy cuando cambia la semántica de un dato

Si el backfill acompaña un cambio de cómo se **escriben** los datos nuevos (escritores) y cómo se **procesan** (trigger/lectura) — por ejemplo, pasar cantidades con signo a cantidades siempre positivas más un campo de tipo:

1. Desplegar el **trigger/lectura robustos**: que entiendan ambos formatos (viejo y nuevo).
2. **Backfill** de los datos históricos.
3. Desplegar los **escritores** nuevos.
4. Recién entonces el **CHECK constraint**, en dos pasos: `add constraint ... not valid` → `validate constraint ...` (ver `migraciones-db`).

**Nunca al revés:** escritores nuevos sin el trigger adaptado = ventana de corrupción en la que cada dato nuevo se interpreta mal y después hay que distinguirlo del viejo.

Skills relacionadas: `migraciones-db`, `integridad-ledger`, `rls-multi-tenant`, `queries-seguras`.
