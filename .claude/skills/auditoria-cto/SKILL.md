---
name: auditoria-cto
description: Auditoría CTO sistemática de un proyecto o de un diff grande — seguridad, performance, integridad de datos, calidad de código y plomería de producción (secrets de build, variables horneadas en el bundle, backups, observabilidad, CI verde ≠ imagen usable). Orquesta las skills migraciones-db, rls-multi-tenant, queries-seguras, backfill-seguro, ciclo-de-vida-entidades, integridad-ledger y reuso-codigo-limpio. Usala antes de mergear una rama larga a la principal, antes de escalar usuarios o cuando aparecen problemas de performance o seguridad. Triggers on "auditoría cto", "auditar", "revisar seguridad", "performance", "antes de mergear", "preparar producción", "salir a producción".
---

# Auditoría CTO

Protocolo que combina las skills técnicas en una sola pasada ordenada. Produce un **reporte de hallazgos con severidad** y una decisión: se puede mergear/escalar, o no.

**Alcance:** 1) seguridad · 2) performance · 3) integridad de datos · 4) calidad de código · 5) plomería de producción · 6) features nuevos vs su spec.

**Reglas de la auditoría:** cada hallazgo lleva evidencia (comando, query, archivo:línea), no impresión. Los comandos de abajo son ejemplos — adaptalos al stack real (carpetas, nombres de scripts, tablas). Lo que no se pudo verificar se declara "no verificado", nunca "OK".

---

## 1. Mapa del diff

```bash
git diff main..<rama> --name-only
git diff main..<rama> --stat
```
Clasificar cada archivo: **nuevo**, **modificado**, **migración nueva**. Cada migración nueva se lee completa y se le aplica `migraciones-db`; si toca datos existentes, `backfill-seguro`.

## 2. Seguridad (`rls-multi-tenant`)

```sql
-- 2.1 Tablas sin RLS (solo las de una whitelist documentada)
SELECT tablename FROM pg_tables WHERE schemaname='public' AND rowsecurity=false;

-- 2.2 Policies permisivas duplicadas (ideal: vacío o justificado)
SELECT tablename, cmd, count(*) FROM pg_policies WHERE schemaname='public'
GROUP BY tablename, cmd HAVING count(*) > 1 ORDER BY count(*) DESC;
```
```bash
# 2.3 auth.uid() sin envolver en SELECT (bug de initplan: se evalúa por fila)
grep -rn "auth\.uid()" migrations/ | grep -v "(SELECT auth.uid())"
# 2.4 REVOKE a anon no sirve (hereda de PUBLIC): debe ser FROM PUBLIC
grep -rn "REVOKE.*FROM anon" migrations/
# 2.5 SECURITY DEFINER sin search_path
grep -n "SECURITY DEFINER" migrations/*.sql | grep -v "search_path"
# 2.6 Tablas de backup sin RLS
grep -rn "_backup_" migrations/ | grep -v "ENABLE ROW LEVEL SECURITY"
# 2.7 Secrets en el repo o su historial
git diff main -- .env ; git log --all --full-history -- .env
# 2.8 CORS abierto en funciones/endpoints
grep -rn "Access-Control-Allow-Origin" <carpeta-de-funciones>/
```
- Si hubo secrets en historial y el repo fue público o compartido: **rotar** (borrar el commit no alcanza).
- CORS: dominio de la app, no `*`; excepción solo en endpoints públicos documentados.
- **IDOR:** todo UPDATE/DELETE por `id` agrega el filtro de tenant en el cliente además de RLS:
```ts
// MAL: depende 100% de RLS
db.from('tabla').update(data).eq('id', id);
// BIEN: defensa en profundidad
db.from('tabla').update(data).eq('id', id).eq('tenant_id', tenantId);
```

## 3. Performance (`queries-seguras`)

```bash
# 3.1 N+1: await de query dentro de loops (revisar cada hit)
grep -rnE "forEach|for .* of|\.map\(async" src/services/
# 3.2 Doble-fetch: recarga global en el .then() de una escritura
grep -rnE "fetchAll|refetch" src/components/ | grep -E "\.then|then\("
# 3.3 SELECT * sobre tablas con JSONB/columnas grandes
grep -rnE "\.select\(['\"]\*['\"]\)" src/services/
# 3.4 Budget de queries al montar la pantalla principal (objetivo ≤ 3)
grep -cE "\.from\(|\.rpc\(" <pantalla-principal>
```
```sql
-- 3.5 Foreign keys sin índice
SELECT tc.table_name, kcu.column_name
FROM information_schema.table_constraints tc
JOIN information_schema.key_column_usage kcu ON tc.constraint_name = kcu.constraint_name
WHERE tc.constraint_type = 'FOREIGN KEY'
  AND NOT EXISTS (SELECT 1 FROM pg_indexes pi
    WHERE pi.tablename = tc.table_name AND pi.indexdef LIKE '%' || kcu.column_name || '%')
ORDER BY tc.table_name;
```
- 3.6 Vistas con subqueries correlacionadas → candidatas a `LATERAL JOIN` o índice parcial en el `WHERE` frecuente.
- 3.7 Componentes que hacen fetch individual y se montan N veces en una lista: N fetches en paralelo agotan el pool. Subir el fetch a un solo request.
- 3.8 Si el proveedor ofrece advisors (ej. `supabase db advisors`), correrlos y adjuntar el resultado.

## 4. Integridad de datos (`integridad-ledger`, `ciclo-de-vida-entidades`)

```sql
-- 4.1 Invariante de signo en el ledger (debe ser 0)
SELECT count(*) FROM <tabla_movimientos> WHERE cantidad < 0;
```
```bash
# 4.2 Tablas nuevas sin CHECK en columnas numéricas críticas
grep -A5 "CREATE TABLE" migrations/*.sql | grep -v "CHECK"
# 4.3 getOrCreate/upsert sin UNIQUE index de soporte
grep -rnE "getOrCreate|findOrCreate|upsert" src/services/
# 4.4 TEMP TABLE en RPCs (fallan con doble click o llamadas anidadas; usar CTEs)
grep -rnE "TEMP TABLE" migrations/*.sql
```
- **4.5 Ciclo de vida:** toda entidad con estado (`estado`, `status`, `decision`) trae **todas** sus transiciones antes de producción. Por cada entidad: ¿quién la crea, quién la libera/consume/cancela, qué cierra cada estado? No se deploya media entidad.
- **4.6 Filtro de decisión:** si una tabla marca ítems como rechazados/descartados, **todos** los flujos que procesan filtran explícitamente (ej. `.is('decision', null)`). Un flujo que filtra `status='pending'` pero no la decisión ejecuta lo que el usuario rechazó.

## 5. Calidad de código (`reuso-codigo-limpio`)

```bash
grep -rnE "window\.(confirm|prompt|alert)" src/          # bloqueantes e inaccesibles → diálogo del sistema de UI
grep -rnE "group-hover|hover:opacity|hover:flex" src/     # acciones solo con hover: invisibles en mobile
grep -rln "fallback\|catch" src/services/                 # revisar código que NUNCA se ejecuta
<comando-de-tipos>                                        # ej. npx tsc --noEmit → 0 errores
```
Además: duplicación que debería ser un módulo compartido, dead code y fallbacks muertos, y lógica de dominio metida en componentes (ver `ddd`).

## 6. Plomería de producción

Es donde "anda en mi máquina" se rompe. Verificar, no suponer:

- **CI verde ≠ imagen usable.** Verde solo dice que compiló. Levantar la imagen/artefacto que va a producción y abrir la app: que cargue, loguee y haga una lectura real.
- **Variables horneadas en el bundle.** Las del frontend (`VITE_*`, `NEXT_PUBLIC_*`, etc.) se incrustan **en build**: si faltan en el CI la app sale en blanco, sin error. Confirmar que el pipeline las recibe (secrets del CI o build args) y que ninguna sensible lleva prefijo público.
- **Variables de runtime** (servidor, funciones): presentes en el entorno de producción y listadas en `.env.example`.
- **Backups.** Existe backup automático de la base, y **se probó restaurarlo** al menos una vez. Antes de una migración destructiva, backup puntual verificado.
- **Observabilidad.** Logs accesibles, errores del frontend/backend llegan a algún lugar que alguien mira, hay alerta de caída. Sin esto el primer aviso es el usuario.
- **Healthcheck y rollback.** Endpoint de salud, imagen/tag anterior disponible, y el camino de vuelta atrás escrito (ver `pre-ship` ítem de rollback).
- **Límites de infra.** Pool de conexiones y rate limits (email/OTP, APIs de terceros) dimensionados para los usuarios esperados.

## 7. Features nuevos vs su spec (`sdd`, `historias-de-usuario`)

Por cada feature nuevo: ¿hay spec? ¿el flujo implementado coincide? ¿happy path funcional? ¿manejo de error? ¿flujo inverso (deshacer, cancelar, limpiar)? ¿tiene tests (`tdd`)?

---

## Clasificación de hallazgos

| Nivel | Criterio | Merge | Cuándo resolver |
|-------|----------|-------|-----------------|
| **CRÍTICO** | Corrompe datos, ejecuta acciones no autorizadas, expone datos de otro tenant, secret filtrado | Bloquea | Mismo día |
| **ALTO** | Feature incompleto que se degrada con el tiempo, race condition, N+1 en la carga principal | Bloquea (o issue abierto con dueño y fecha) | Sprint actual |
| **MEDIO** | Performance subóptima, dead code con overhead, patrones inconsistentes | No bloquea | Sprint siguiente |
| **BAJO** | Cosmético, naming, refactor menor | No bloquea | Backlog |

**Formato de cada hallazgo:**
```
[SEVERIDAD] ID — título corto
Dónde: archivo:línea / tabla / endpoint
Evidencia: comando o query + resultado
Impacto: qué pasa si no se arregla
Fix propuesto: cambio concreto (y skill que aplica)
```
El reporte se guarda en el repo (ej. `docs/AUDITORIA_<fecha>.md`) y los pendientes entran al roadmap con ID.

## Checklist de entrega final

```
□ Todos los CRÍTICOS resueltos; los ALTOS resueltos o con issue asignado
□ Tipos y lint → 0 errores; tests y E2E verdes
□ Migraciones versionadas en el repo y aplicadas al entorno destino
□ 0 tablas sin RLS (salvo whitelist); 0 policies duplicadas nuevas
□ 0 FKs críticas sin índice
□ Invariante del ledger → 0 violaciones
□ IDOR: toda operación por id filtra tenant en el cliente
□ Toda entidad con estado tiene todas sus transiciones; procesamiento filtra descartados
□ 0 window.confirm/alert/prompt; 0 acciones importantes solo con hover
□ Secrets limpios; variables de build presentes en el CI; imagen probada de verdad
□ Backup restaurable, observabilidad y rollback definidos
□ Sin coautorías de agentes en el último commit si el repo lo prohíbe (git log -1 --format=%B)
□ Docs/roadmap actualizados con el reporte y los pendientes
```

---

## Incidentes recurrentes — no repetir

- **Pool de conexiones agotado.** Un `while (hasMore)` en un efecto de montaje cargaba toda una tabla con columnas JSONB pesadas; con 10 usuarios simultáneos se agotó el pool y cayó la base. *Regla:* nunca loops paginados en efectos de montaje; paginar server-side.
- **Timeout por RLS lenta.** Una policy con `IN (SELECT … FROM otra_tabla)` sobre decenas de miles de filas hacía timeout en toda query que embebía esa tabla. *Regla:* `EXISTS` correlacionado + índice de soporte.
- **Cientos de queries al cargar.** Una recarga global dentro del `.then()` de una sincronización + cientos de UPDATE individuales + N+1 + vista sin índice parcial. *Regla:* sin recarga global tras escrituras, batch/RPC, índice parcial en el `WHERE` frecuente.
- **Rechazados que igual se ejecutaban.** El flujo de envío filtraba por `status='pending'` pero no por la decisión del usuario; lo rechazado en la UI salía igual hacia el sistema externo. *Regla:* filtrar la decisión en **todos** los queries que procesan ítems.
- **Reservas sin ciclo de vida.** Se creó la entidad reserva sin liberar ni consumir: se acumularon para siempre. *Regla:* no deployar la mitad de una entidad.
- **`REVOKE FROM anon` sin efecto.** `anon` hereda de `PUBLIC`. *Regla:* siempre `REVOKE ... FROM PUBLIC`.
- **Build verde, app en blanco.** El CI compiló sin las variables `VITE_*`; la imagen salió sin configuración y la pantalla quedó vacía sin ningún error. *Regla:* verificar las variables de build y abrir la imagen antes de dar por listo.
