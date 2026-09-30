---
name: ciclo-de-vida-entidades
description: Protocolo para garantizar que toda entidad con estado tiene TODAS sus transiciones implementadas antes de ir a producción. Previene el anti-patrón de "media entidad deployada" que deja reservas, colas o estados colgados indefinidamente. Usala al diseñar o revisar cualquier tabla/objeto con campo de estado. Triggers on "reservas", "estado", "status", "ciclo de vida", "cancelar", "entidad con estado", "crear y olvidar", "estados huérfanos", "cola de procesamiento".
---

# Ciclo de vida de entidades — toda entidad con estado, completa

## El principio

> **No deployar la mitad de una entidad.** Si se puede crear, también se debe poder cerrar, liberar, consumir o cancelar — antes de que llegue a producción.

El bug recurrente: se implementa `crearX()` y se deja para "el próximo sprint" el `liberarX()` / `consumirX()`. Con usuarios reales, las entidades incompletas se acumulan y corrompen los cálculos derivados (stock disponible, totales de caja, saldos).

**Caso tipo:** un servicio de reservas de stock tenía `crearReserva` pero no `liberarReserva` ni `consumirReserva`. Las reservas activas se acumularon hasta dejar el stock disponible en 0 aunque hubiera stock físico. Nadie lo vio en desarrollo porque con datos de prueba nunca se acumulan.

## Qué es una entidad con estado

Cualquier tabla o registro que:
- Tiene un campo `estado`, `status`, `decision`, `activo` o similar.
- Pasa por distintos estados a lo largo del tiempo.
- Su estado afecta cálculos, dashboards o flujos de otros módulos.

Típicas: reservas, colas de sincronización, órdenes, sesiones (caja, conteo), solicitudes, suscripciones.

## Protocolo antes de deployar

### Paso 1 — Mapear el ciclo de vida completo

```
inexistente → activa → consumida   (cuando el recurso se usa)
                     → liberada    (cuando el padre se cancela)
                     → expirada    (si aplica un vencimiento)
```

Preguntas: ¿cómo se crea? ¿cómo termina? ¿qué estados son finales (terminales)?

### Paso 2 — Una función por cada transición

```ts
// MAL — solo la mitad
async function crearReserva(params): Promise<Reserva>
// liberarReserva → falta
// consumirReserva → falta

// BIEN — ciclo completo
async function crearReserva(params): Promise<Reserva>
async function liberarReserva(id: string, tenantId: string): Promise<void>
async function consumirReserva(id: string, tenantId: string): Promise<void>
```

### Paso 3 — Conectar cada transición a un evento del dominio

| Transición | Evento que la dispara | Se implementa en |
|-----------|----------------------|------------------|
| liberar | el padre se cancela | handler de cancelación del padre |
| liberar | el pedido es rechazado | flujo de aprobación |
| consumir | el recurso se entrega | flujo de despacho |
| consumir | se cierra la venta | cierre de venta |

**Regla:** una transición que ningún evento dispara es lo mismo que no existir. Trazá el hilo completo, del evento a la fila.

### Paso 4 — Auditar huérfanos

```sql
-- Hijos activos con padre ya cerrado (debe dar 0 filas):
SELECT h.*, p.estado AS padre_estado
FROM reservas h
JOIN ordenes p ON p.id = h.origen_id
WHERE h.estado = 'activa' AND p.estado IN ('cerrada', 'cancelada');

-- Ítems rechazados que siguen "pendientes" en la cola (debe dar 0):
SELECT * FROM cola_sync WHERE decision = 'rejected' AND status = 'pending';
```

Dejá estas queries guardadas como auditoría recurrente (ver `auditoria-cto`).

## Inventario de entidades con estado

Mantené en el repo una tabla viva (entidad · campo de estado · transiciones requeridas · estado de implementación: completa / falta X). Actualizala cada vez que se agrega una entidad con estado; es el mapa para saber qué está a medias.

## Checklist (por feature)

```
□ ¿La entidad tiene campo de estado?
□ ¿La spec (ver sdd / historias-de-usuario) lista TODOS los estados posibles?
□ ¿Hay función implementada para CADA transición hacia un estado final?
□ ¿Cada transición recibe el tenant/dueño y lo usa en el filtro? (ver rls-multi-tenant)
□ ¿Los eventos del dominio disparan las transiciones (handlers, efectos)?
□ ¿El campo de estado tiene CHECK en DB? CHECK (estado IN ('activa','liberada','consumida'))
□ ¿Hay query de auditoría que detecte huérfanos (activo + padre cerrado)?
□ ¿La UI refleja el estado nuevo tras la transición?
```

## Anti-patrón: estado visual desincronizado

Tras una acción que cambia el estado, la UI debe reflejarlo sin depender de un refetch completo:

```ts
// MAL — el ítem rechazado sigue figurando como "pendiente"
await marcarDecision(id, 'rejected');

// BIEN — actualizar el estado local al instante
await marcarDecision(id, 'rejected');
setItems(prev => prev.map(i => i.id === id ? { ...i, decision: 'rejected' } : i));
```

## Anti-patrón: reprocesar estados terminales

Los flujos de procesamiento (colas, jobs, sincronizaciones) deben filtrar los estados terminales (rechazado, cerrado, cancelado). Un `SELECT` de "pendientes" que ignora `decision` reprocesa lo rechazado (ver `queries-seguras`).

## Relación con otras skills

- `integridad-ledger` — las reservas afectan el saldo disponible; si no se liberan, el saldo miente. Leer ambas al tocar reservas.
- `migraciones-db` — toda tabla nueva con estado lleva CHECK en el campo de estado.
- `backfill-seguro` — limpiar huérfanos existentes se hace con backfill verificado, no con un UPDATE a ciegas.
- `pre-ship` y `auditoria-cto` — verifican el ciclo de vida antes de mergear y en las auditorías periódicas.
