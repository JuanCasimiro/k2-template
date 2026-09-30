---
name: integridad-ledger
description: Reglas para tablas de movimientos / ledger append-only (stock, caja, saldos, puntos, créditos). Fija la convención de signo canónica, un único writer por flujo, saldo derivado vs almacenado y queries de auditoría. Usala al escribir, revisar o auditar cualquier INSERT en una tabla de movimientos, o al leer esos datos para analítica. Triggers on "movimientos", "ledger", "saldo", "stock", "cantidad", "entrada/salida", "débito/crédito", "registrar movimiento", "ajuste", "forecasting sobre movimientos".
---

# Integridad de ledger — tablas de movimientos

Una tabla de movimientos es un **libro mayor append-only**: cada fila es un hecho que ocurrió. El saldo es consecuencia de los hechos, no al revés.

## Regla 1 — Signo canónico (INVARIANTE)

> **La cantidad se guarda SIEMPRE como magnitud positiva (≥ 0). La dirección la define EXCLUSIVAMENTE el campo `tipo` ('entrada' | 'salida').**

Nunca guardes negativos. Nunca infieras dirección del signo. Blindalo en DB: `CHECK (cantidad >= 0)`.

Por qué: con signos mezclados (algunos flujos guardan `-5` y otros `5` + tipo `salida`), cualquier `SUM(cantidad)` crudo, forecast o dashboard miente. Un único criterio permite agregar con `abs()` + filtro por `tipo`.

```ts
await db.from('movimientos').insert([{
  tenant_id,
  producto_id,
  cantidad: Math.abs(qty),                 // SIEMPRE positivo
  tipo: esEntrada ? 'entrada' : 'salida',  // la dirección vive acá
  precio_aplicado,                         // snapshot del momento, no recalcular después
  referencia,                              // qué originó el movimiento
}]);
```

Ajustes por diferencia (conteo, ajuste rápido):

```ts
if (diff === 0) return;                    // no generes movimientos de 0
const mov = { cantidad: Math.abs(diff), tipo: diff > 0 ? 'entrada' : 'salida' };
```

## Regla 2 — Un solo writer por flujo

Cada flujo de negocio (venta mostrador, import de canal externo, conteo, ajuste, devolución, alta con stock inicial…) debe tener **un único punto de código** que escribe el movimiento. Dos caminos que registran la misma operación generan doble registro.

- Mantené en el repo la **lista de flujos escritores** (flujo · archivo). Al tocar el invariante, revisás todos. Al agregar un flujo, lo sumás.
- **Idempotencia:** el flujo lleva una referencia externa única (ticket, id de orden del canal) con `UNIQUE` o chequeo previo. Reintentos de red no deben duplicar.
- El mismo hecho no entra por dos vías (ej. despacho **y** import de la misma venta): decidí cuál es la fuente y la otra solo lee.

## Regla 3 — Saldo derivado vs almacenado

Si guardás el saldo (`stock_actual`, `saldo`) por performance, es una **caché del ledger**, y debe mutar solo por un trigger/función del ledger:

```sql
-- saldo := saldo + f(tipo, cantidad), nunca el signo de cantidad
UPDATE productos SET stock_actual = stock_actual + CASE NEW.tipo
  WHEN 'entrada' THEN abs(NEW.cantidad)
  WHEN 'salida'  THEN -abs(NEW.cantidad)
END WHERE id = NEW.producto_id;
```

- Nadie hace `UPDATE productos SET stock_actual = ...` a mano fuera del ledger (salvo un ajuste que a su vez genera un movimiento).
- El trigger suele ser `AFTER INSERT` únicamente: un backfill que normaliza `cantidad` con `UPDATE` **no** lo dispara y no recalcula saldos. Si corregís datos históricos, recalculá el saldo explícitamente (ver `backfill-seguro`).
- Antes de modificar el trigger, volcá su definición viva de la base (ver `migraciones-db`): el repo puede estar desfasado.
- Invariante de auditoría: `saldo almacenado == SUM(entradas) - SUM(salidas)`. Verificalo periódicamente.
- **Saldo disponible** = saldo físico − reservado. Si las reservas no se liberan/consumen, el disponible miente (ver `ciclo-de-vida-entidades`).

## Regla 4 — Campos de referencia: real vs placeholder

Un campo como `nro_orden` / `referencia` suele tener dos clases de valor: **referencia real** (orden cargada por el usuario) y **placeholders sintéticos** (`MOSTRADOR-{ts}`, `CANAL-{id}`, `CONTEO-{id}`) que indican origen, no un documento.

Código que vincule o re-asigne ese campo debe tratar `null` y los placeholders como **re-vinculables**; solo una referencia real distinta bloquea. Caso tipo: un `vincularTicket` veía el placeholder como orden real y bloqueaba con "ya está vinculado a la orden MOSTRADOR-…". Para analítica, cada placeholder es único por transacción: contar referencias distintas cuenta cada venta sin orden como una.

## Queries de auditoría

```sql
-- Sin negativos:
SELECT count(*) FROM movimientos WHERE cantidad < 0;

-- Anomalías por tipo:
SELECT tipo, count(*), min(cantidad), max(cantidad) FROM movimientos GROUP BY tipo;

-- Posible doble registro de la misma operación externa:
SELECT referencia, count(*) FROM movimientos
WHERE referencia LIKE 'CANAL-%' GROUP BY referencia HAVING count(*) > 1;

-- Saldo almacenado vs derivado (debe dar 0 filas):
SELECT p.id FROM productos p
LEFT JOIN (SELECT producto_id,
  SUM(CASE tipo WHEN 'entrada' THEN cantidad ELSE -cantidad END) AS s
  FROM movimientos GROUP BY producto_id) m ON m.producto_id = p.id
WHERE p.stock_actual <> COALESCE(m.s, 0);
```

## Gate de PR

- Grep de `cantidad:` cerca de los INSERT al ledger: ninguno escribe un valor negativo ni `-variable`.
- Toda fila nueva tiene `tipo`, magnitud ≥ 0 y referencia.
- Si el PR agrega un flujo escritor: está en la lista y es idempotente.
- Si toca reservas: el ciclo de vida está completo (crear + liberar + consumir).
- Correcciones históricas: pasan por `backfill-seguro`, con la auditoría antes y después.

Ver también `rls-multi-tenant` (toda fila del ledger lleva tenant) y `queries-seguras`.
