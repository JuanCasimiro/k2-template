---
name: pre-ship
description: Checklist ejecutable antes de mergear a la rama principal o deployar — tipos, lint, tests, migraciones versionadas en el repo antes de aplicarlas, RLS, invariantes de datos, queries, secrets, variables de entorno y rollback. Usala como última compuerta antes de subir a producción. Triggers on "pre-ship", "deploy", "mergear", "subir a producción", "checklist de deploy", "antes de pushear", "listo para mergear".
---

# Pre-ship — checklist antes de mergear o deployar

Ejecutar en orden. Cada ítem se marca ✅ (pasó), ➖ (no aplica, con motivo) o ❌ (bloquea). **Solo se mergea con todo lo aplicable en ✅.**

Los comandos son ejemplos: usá los scripts reales del proyecto (mirá `package.json`, `Makefile` o el CI). Si un check no existe en el proyecto, decilo — no lo inventes ni lo saltees en silencio.

---

## 1. Tipos y lint — bloquea

Comando de tipos del proyecto (ej. `npx tsc --noEmit`) y de lint (ej. `npm run lint`). Cero errores. No se deploya con errores de tipos.

## 2. Tests unitarios y cobertura — bloquea

Comando de tests del proyecto (ej. `npm test`). Si el proyecto define umbrales de cobertura, que pasen; si fallan, se agregan tests antes de mergear (ver skill `tdd`).

## 3. Tests E2E de los flujos críticos — bloquea

Comando E2E del proyecto (ej. `npm run test:e2e`). Sin credenciales corren solo los públicos; los autenticados (login, cobro, alta de la entidad principal) se corren con credenciales de un entorno de prueba, nunca de producción.

## 4. Migraciones — versionadas ANTES de aplicarlas — bloquea

- [ ] Todo cambio de esquema tiene su archivo en el repo (ej. `migrations/` o `supabase/migrations/`), **escrito antes de aplicarlo** a cualquier base. Aplicar por consola/MCP/SQL editor sin archivo deja al repo describiendo una base que ya no existe.
- [ ] Cada migración se leyó completa (ver skill `migraciones-db`).
- [ ] Backfills sobre datos existentes siguen `backfill-seguro`.
- [ ] No hay migraciones en el repo sin aplicar en el entorno destino ni aplicadas sin estar en el repo (ej. `supabase migration list`, o el equivalente de tu herramienta).

## 5. RLS en tablas nuevas — bloquea

```bash
grep -rn "ENABLE ROW LEVEL SECURITY" migrations/
```
Cada tabla nueva debe aparecer, incluidas las de backup. Además, en la base:
```sql
SELECT tablename FROM pg_tables WHERE schemaname = 'public' AND rowsecurity = false;
-- Solo las justificadas en una whitelist documentada
```
Protocolo completo en `rls-multi-tenant`.

## 6. Policies sin duplicados ni REVOKE incorrecto — bloquea

```sql
SELECT tablename, cmd, count(*) AS policies
FROM pg_policies WHERE schemaname = 'public'
GROUP BY tablename, cmd HAVING count(*) > 1 ORDER BY tablename, cmd;
-- Esperado: vacío, o cada fila justificada
```
- [ ] Sin policies permisivas duplicadas para el mismo comando en la misma tabla.
- [ ] `auth.uid()` dentro de policies va envuelto: `(SELECT auth.uid())`.
- [ ] Los `REVOKE` de funciones son `FROM PUBLIC` (revocar solo a `anon` no sirve: hereda de `PUBLIC`).
- [ ] Toda función `SECURITY DEFINER` fija `SET search_path`.

## 7. Invariantes de datos — bloquea

Si el PR toca flujos que escriben en un ledger o registro de movimientos (ver `integridad-ledger`): las cantidades se guardan siempre positivas y la dirección la da un campo de tipo.
```bash
grep -rnE "(cantidad|amount|quantity): *-" src/
```
```sql
SELECT count(*) FROM <tabla_movimientos> WHERE cantidad < 0;  -- debe ser 0, también post-deploy
```
Columnas numéricas críticas nuevas (cantidad, precio, monto) con `CHECK`; columnas con unicidad implícita con índice `UNIQUE`.

## 8. Aislamiento de tenant en el cliente — bloquea

```bash
grep -rn "\.update\|\.delete" src/services/ | grep "\.eq('id'"
```
- [ ] Todo UPDATE/DELETE por `id` agrega también el filtro de tenant (`.eq('tenant_id', tenantId)`): defensa en profundidad, no depender solo de RLS (IDOR).

## 9. Ciclo de vida y filtros de estado — bloquea

Si el PR crea o modifica una entidad con estado (`estado`, `status`, `decision`; ver `ciclo-de-vida-entidades`):
- [ ] Toda entidad que se crea tiene su cierre/liberación/consumo, y el flujo inverso existe (cancelar libera lo reservado).
- [ ] Sin `getOrCreate`/`upsert` sin `UNIQUE` de soporte en la base.
- [ ] Los flujos que **ejecutan** algo filtran explícitamente los descartados (ej. `decision IS NULL`): un ítem rechazado con `status='pending'` no puede procesarse.

## 10. Seguridad rápida — bloquea

```bash
git diff main --name-only | xargs grep -lEi "sk_live|service_role|password *=|secret *=|BEGIN (RSA|PRIVATE)" 2>/dev/null
```
Todo hit se revisa. Un secret que llegó a un commit (aunque se haya borrado después) se **rota**; borrarlo del historial no alcanza.

## 11. Variables de entorno y build — bloquea

- [ ] Toda variable nueva está en `.env.example` (nunca el valor real) y configurada en el entorno de producción.
- [ ] Las variables que el bundle del frontend hornea en build (ej. prefijo `VITE_`, `NEXT_PUBLIC_`) están disponibles **en el CI/build**, no solo en runtime: si faltan, el build pasa verde y la app sale en blanco.
- [ ] Nada sensible con prefijo público: lo que va al bundle lo lee cualquiera.

## 12. Queries — bloquea si es flujo de carga principal, advertencia si no

Ver `queries-seguras`.
- [ ] Sin `SELECT *` en tablas con columnas JSONB/grandes.
- [ ] Sin loops paginados (`while (hasMore)`) en efectos de montaje.
- [ ] Agregaciones vía RPC/vista, no filas crudas sumadas en JS; joins server-side.
- [ ] Máx. 2-3 queries al montar una página nueva; gate de módulo antes de consultar tablas opcionales.
```bash
# N+1: queries dentro de loops (revisar cada hit a mano)
grep -rnE "forEach|for .* of|\.map\(async" src/services/
# Doble-fetch: recarga global dentro de .then() de una escritura
grep -rnE "fetchAll|refetch|fetchInventory" src/components/ | grep -E "\.then|then\("
```
- [ ] Operaciones batch con `upsert([...])`, `.in(...)` o RPC con array; una escritura no dispara una recarga global al terminar.

## 13. UX y docs — advertencia

```bash
grep -rnE "window\.(confirm|prompt|alert)" src/
```
- [ ] Sin `window.confirm/alert/prompt` (bloquean, se cierran solos en mobile, son inaccesibles): usar el diálogo del sistema de UI.
- [ ] Acciones importantes no dependen de hover (en mobile no existe).
- [ ] Si cambió un feature visible al usuario, la doc/README/changelog se actualizó en el mismo commit.

## 14. Rollback — bloquea

Antes de deployar, una línea por cambio de riesgo:
- **Código:** cómo se vuelve atrás (revert del merge, tag/imagen anterior).
- **Migración:** cómo se deshace (migración inversa escrita, o es aditiva y se ignora). Las destructivas (`DROP`, cambios de tipo) requieren backup verificado antes.
- **Dato:** si un backfill toca filas, hay snapshot o tabla de respaldo con RLS.

---

## Resultado esperado

| Check | Estado |
|-------|--------|
| Tipos + lint | ✅ |
| Tests + cobertura | ✅ |
| E2E | ✅ |
| Migraciones en repo y aplicadas | ✅ o ➖ |
| RLS en tablas nuevas | ✅ o ➖ |
| Policies sin duplicados / REVOKE | ✅ o ➖ |
| Invariantes de datos | ✅ o ➖ |
| Tenant en operaciones por id | ✅ o ➖ |
| Ciclo de vida / filtros de estado | ✅ o ➖ |
| Secrets limpios | ✅ |
| Env vars + secrets de build | ✅ o ➖ |
| Queries (N+1, doble-fetch) | ✅ |
| UX y docs | ✅ o ➖ |
| Rollback definido | ✅ |

Para una revisión profunda (no solo la compuerta del PR) usá `auditoria-cto`.
