---
name: queries-seguras
description: Reglas para que el código que consulta la base no la tire abajo — proyecciones, paginación, agregaciones en SQL, N+1, doble-fetch, math del connection pool. Usala al escribir fetchs de listas, useEffect con queries, loops con await a la DB o queries sobre tablas grandes. Triggers on "query lenta", "timeout", "N+1", "fetchAll", "traer todos los registros", "pool de conexiones", "la base se cayó", "useEffect con query", "paginar", "agregación".
---

# Queries seguras — no tires la DB abajo

Ejemplos con el cliente JS de Supabase; los principios aplican a cualquier ORM o driver contra Postgres.

## Cuándo activar

- Se escribe `fetchAll`, "todos los registros", o se consulta una tabla grande (movimientos, eventos, logs, publicaciones).
- Un `useEffect` tiene `from(` / `rpc(` en el mismo componente.
- Aparece un loop `while (hasMore)` / `while (page < total)` dentro de un componente UI.
- Un `.forEach`, `for...of` o `.map(async` ejecuta `await` de DB adentro.
- Se llama a un fetch de lista dentro del `.then()` de otra operación.

## Las reglas

### 1. Nunca `SELECT *` en tablas con JSONB/texto pesado
Una tabla con `raw_data` JSONB de ~3 KB por fila multiplicada por miles de filas es megabytes por carga. Proyección explícita (`id, titulo, estado, precio`); el JSONB solo en el detalle de UNA fila concreta.

### 2. Nunca paginar tablas completas en un `useEffect` de montaje
```js
// MAL — cada iteración ocupa 1 conexión mientras dura el loop
while (hasMore) { data = await supabase.from('items').range(from, from + 1000) }
```
Los loops paginados son para jobs y migraciones, no para UI. **10 usuarios × 5 páginas = 50 conexiones sostenidas → pool agotado → DB caída.** Regla dura: si un `useEffect` dispara >2 queries al montar, rediseñalo.

### 3. Agregaciones en SQL (RPC/vista), nunca en JS sobre filas crudas
"Ventas de 30 días por producto" es un `GROUP BY` en una RPC. Nunca traer 50.000 filas para contarlas en el navegador. Para contar, `count` exacto sin traer filas:
```js
const { count } = await supabase.from('items')
  .select('id', { count: 'exact', head: true }).eq('tenant_id', tenantId);
```

### 4. Joins en el servidor (VIEW o RPC), nunca en JS
Dos queries que se cruzan en el cliente con `find`/`filter` son O(n×m) y duplican transferencia. Una vista (con `LATERAL` si hace falta) o una RPC. Ver `rls-multi-tenant` para vistas con `security_invoker`.

### 5. Gate de módulo/feature antes de consultar
Si una funcionalidad es opcional por plan o por tenant (`if (!hasModule('integraciones')) return;`), quien no la tiene hace **0 queries** a sus tablas.

### 6. Math del connection pool: razoná antes de escribir
Un plan chico de Supabase ronda las 60 conexiones; cada `await query` ocupa una hasta resolver.
`N usuarios × M queries seriales = N×M conexiones sostenidas`. Superado el límite, las queries quedan pendientes y **la app parece caída** aunque la DB esté sana.

### 7. Anti-patrón N+1: nunca queries en loop sobre filas
```js
// MAL — 400 UPDATEs, 400 round-trips
for (const item of items) {
  await supabase.from('items').update({ precio: item.precio }).eq('id', item.id);
}
// BIEN — 1 llamada a una RPC con un array JSON
await supabase.rpc('bulk_update_precios', { p_updates: items });
```
```sql
create or replace function public.bulk_update_precios(p_updates jsonb)
returns void language sql security invoker set search_path = public as $$
  update items i
  set precio = (u->>'precio')::numeric
  from jsonb_array_elements(p_updates) as u
  where i.id = (u->>'id')::uuid;
$$;
```
Vale también para INSERT (`.upsert([...array])` o RPC, nunca loop de `.insert()`) y SELECT (`.in('id', ids)` en vez de un select por id).

### 8. Anti-patrón doble-fetch: no refetchear todo tras cada operación
```js
// MAL — duplica TODAS las queries de la pantalla tras cada sync
actualizarPrecios().then(() => { fetchInventario(); });
// BIEN — actualizar solo el slice afectado del estado local (o invalidar de forma lazy)
const nuevos = await actualizarPrecios();
setInventario(prev => prev.map(p => ({ ...p, ...nuevos[p.sku] })));
```
Una escritura o sync no dispara un refetch completo de la lista.

### 9. COUNT/agrupar client-side sobre tablas grandes: RPC con GROUP BY
Si el caller usa las filas solo para `.length`, `.reduce()` o `.filter().length`, es un anti-patrón: `count` exacto (regla 3) o una RPC `group by`.

### 10. Sin código muerto de fallback
```js
// MAL — el segundo rpc() nunca corre, pero da falsa sensación de resiliencia y oculta errores reales
const { data } = await supabase.rpc('get_stats_v2', params);
if (!data) { await supabase.rpc('get_stats_v1', params); }
// BIEN — una ruta. Si hay que cambiar, migrar y eliminar la vieja.
```

## Checklist de revisión

```bash
# SELECT * en servicios
grep -rn "\.select('\*')" services/
# Loops paginados en componentes UI
grep -rn "while.*hasMore\|while.*page" components/
# N+1: loops con await de DB adentro (revisar a mano cada resultado)
grep -rn "for.*of\|forEach\|map.*async" services/ | grep -v node_modules
# Doble-fetch: refetch de lista dentro de .then()
grep -rn "fetch" components/ | grep "then"
# Budget de queries por pantalla (debe ser <= 3 al montar)
grep -c "\.from(\|\.rpc(" components/PantallaPrincipal.tsx
# Columnas traídas solo para contar
grep -rn "\.select('tenant_id')" services/ | grep -v "rpc\|count"
```

## Por qué existen estas reglas (incidentes)

**Pool agotado por carga completa al montar.** Una feature nueva cargaba TODAS las filas de una tabla con JSONB pesado en un loop paginado al montar el dashboard, más 50k filas de movimientos. Con ~10 usuarios simultáneos: ~160 conexiones contra un pool de ~60 → DB completamente caída; hizo falta rollback y reinicio. Motivó las reglas 1, 2, 3 y 6.

**RLS + trigger → statement timeout.** Mismo síntoma ("no se guarda", "no se ve", "tarda") que en realidad era HTTP 500 por timeout. Dos causas:
1. Una policy con `col IN (select … from clientes …)` escaneaba 40k filas por evaluación de fila tras una importación masiva; como otras tablas la embebían, toda query con embed expiraba. Fix: `EXISTS` correlacionado por índice (1,24 s → 42 ms). Detalle en `rls-multi-tenant`.
2. Un trigger AFTER UPDATE hacía un UPDATE en otra tabla en cada edición aunque el estado no cambiara → lock contention en cascada con varios usuarios → writes cancelados en silencio. Fix: guard al inicio (`if new.estado is not distinct from old.estado then return new;`).

**500+ requests al abrir una pantalla.** Un refresco de precios hacía N UPDATEs individuales (uno por publicación, 400 en total) y además llamaba al fetch completo del inventario al terminar: 400 UPDATEs seriales + 100+ queries de carga = 500+ round-trips en cada apertura. Fix: RPC bulk (regla 7) y eliminar el refetch del `.then()` (regla 8).

Skills relacionadas: `rls-multi-tenant`, `migraciones-db`, `backfill-seguro`, `pre-ship`.
