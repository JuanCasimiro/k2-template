---
name: ddd
description: Domain Driven Design — cómo estructurar código alrededor del modelo de negocio (aggregates, value objects, repositorios, bounded contexts, lenguaje ubicuo). Aplica cuando hay lógica de dominio compleja (validaciones, estados, invariantes); en CRUD puro es overkill. Triggers on dominio, aggregate, bounded context, entidad, value object, modelo de dominio, DDD, arquitectura de dominio, estructura de código, invariantes.
---

# Domain Driven Design (DDD)

DDD organiza el código alrededor del **modelo de negocio**, no de la base de datos ni del HTTP. El dominio manda; la infraestructura lo sirve. Ejemplos en TypeScript; el enfoque aplica a cualquier lenguaje.

## Cuándo usarlo

Cuando hay **lógica de negocio no trivial**: estados con transiciones, reglas de consistencia, objetos que evolucionan con identidad propia. Si el módulo es CRUD puro (guardar y leer sin reglas), es overkill: un repositorio simple alcanza.

- **Sí:** compras, inventario, ventas, facturación, sincronización con canales externos.
- **No hace falta:** configuración, reportes de solo lectura, integraciones simples.

## Patrones tácticos (los 5 que importan)

### 1. Entity — objeto con identidad
Dos instancias con el mismo ID son el mismo objeto aunque sus atributos cambien.

```ts
class SolicitudDeCompra {
  constructor(readonly id: string, /* ... */) {}
  // cambia de estado pero sigue siendo la misma solicitud
}
```

### 2. Value Object — sin identidad, inmutable
Definido solo por sus atributos; para cambiarlo, creás uno nuevo. Se valida al construirse.

```ts
class Monto {
  constructor(readonly amount: number, readonly currency: string = 'ARS') {
    if (amount < 0) throw new Error('Monto no puede ser negativo');
  }
  equals(o: Monto) { return this.amount === o.amount && this.currency === o.currency; }
}
```

Usalo para dinero, estados tipados, emails, fechas con semántica de negocio.

### 3. Aggregate — frontera de consistencia
Cluster de entidades y value objects que deben quedar consistentes juntos. El **Aggregate Root** es la única puerta de entrada.

```ts
class SolicitudDeCompra {                  // Aggregate Root
  private lineas: LineaDeArticulo[] = [];  // hijo: no se accede directo

  submit(): void {
    if (this.estado !== 'DRAFT') throw new Error('Solo se envía un borrador');
    this.estado = 'SUBMITTED';
    this.addEvent(new SolicitudSubmitted(this.id));
  }
}
```

**Regla de oro:** los aggregates se referencian entre sí **solo por ID**, nunca por objeto. Una transacción = un aggregate. Si dos cosas deben cambiar juntas siempre, van en el mismo aggregate; si pueden ser eventualmente consistentes, son dos conectados por eventos.

### 4. Repository — abstracción de persistencia
Interface en el dominio, implementación en infraestructura. El dominio no sabe de SQL.

```ts
interface ISolicitudRepository {          // dominio
  save(s: SolicitudDeCompra): Promise<void>;
  getById(id: string): Promise<SolicitudDeCompra | null>;
}
class SqlSolicitudRepository implements ISolicitudRepository { /* infraestructura */ }
class InMemorySolicitudRepository implements ISolicitudRepository { /* tests (ver tdd) */ }
```

### 5. Domain Service — lógica cross-aggregate
Regla que no pertenece a un aggregate solo. Sin estado propio.

```ts
class AprobadorDeCompras {
  async puedoAprobar(s: SolicitudDeCompra): Promise<boolean> {
    // cruza proveedor + inventario
  }
}
```

## Patrones estratégicos

### Bounded Context — frontera del modelo
Límite explícito donde un modelo aplica con su propio lenguaje. Dos contextos pueden tener "Producto" con significados distintos, y está bien.

| Contexto | Qué significa "Producto" acá |
|----------|-----------------------------|
| Compras | Artículo a reponer con proveedor y precio |
| Tienda online | Publicación con stock compartido entre variantes |
| Ventas multicanal | Ítem vendido o reservado por canal |
| Permisos | No existe |

Cada contexto = una carpeta propia en `src/domains/`.

### Lenguaje ubicuo
El mismo término, con el mismo significado, en la spec, el código, los tests y las conversaciones. No "orden de compra" acá y "purchase request" allá. Las specs (`sdd`) y las historias (`historias-de-usuario`) definen el lenguaje; el código lo usa tal cual (ver `reuso-codigo-limpio`).

### Anti-Corruption Layer (ACL)
Adaptador que traduce el modelo externo (marketplace, pasarela de pago, planilla) al modelo interno, para que conceptos ajenos no contaminen el dominio.

```ts
class MarketplaceTranslator {
  toVenta(ext: ExternalOrderItem): Venta {
    return new Venta(ext.order_id, new Monto(ext.unit_price), /* ... */);
  }
}
```

## Estructura de carpetas

```
src/
├── domains/
│   ├── compras/                  # un bounded context
│   │   ├── entities/             # Aggregate Root
│   │   ├── value-objects/
│   │   ├── events/
│   │   ├── repositories/         # interfaces (sin implementación)
│   │   └── services/
│   ├── ventas/
│   │   └── anti-corruption/      # traductores de sistemas externos
│   └── shared/                   # value objects entre contextos, DomainEvent
├── application/                  # casos de uso (orquestación)
├── infrastructure/               # repositorios reales, clientes externos
└── api/                          # handlers HTTP / rutas
```

Dependencias hacia adentro: `api → application → domains`. `domains` no importa de infraestructura.

## Domain Events

Desacoplan contextos: Compras publica `SolicitudAprobada`; Inventario lo escucha y reserva stock. Los eventos se guardan en el aggregate y se publican al persistir (outbox).

```ts
abstract class DomainEvent {
  constructor(readonly aggregateId: string, readonly occurredAt = new Date()) {}
}
class SolicitudAprobada extends DomainEvent {
  constructor(id: string, readonly proveedorId: string, readonly monto: Monto) { super(id); }
}
```

Los eventos son las transiciones de `ciclo-de-vida-entidades` hechas explícitas: cada estado final tiene su evento y su handler.

## Cómo extraer el modelo de una spec

Al leer una spec, buscá:

1. **Objetos con ciclo de vida y estados** → candidatos a Aggregate Root.
2. **Estados y transiciones posibles** → invariantes del aggregate.
3. **Atributos sin identidad** (precio, monto, estado) → Value Objects.
4. **Reglas que cruzan objetos** ("no se aprueba si el proveedor está inactivo") → Domain Service.
5. **"Cuando X pasa, Y debe enterarse"** → Domain Event.
6. **Integraciones externas** → ACL.

**Señal de buen aggregate:** podés describir sus reglas sin mencionar la base de datos.

## Skip list — qué NO hacer en un SaaS chico

| Patrón | Motivo para saltearlo |
|--------|----------------------|
| Factories complejas | El constructor alcanza |
| Specifications/predicates | La regla va inline en el aggregate |
| CQRS | Solo si lecturas y escrituras son radicalmente distintas |
| Event Sourcing | Una base relacional + logs cubre casi todo (un ledger append-only ya da auditoría: ver `integridad-ledger`) |
| Sagas distribuidas | Si no hay transacciones entre servicios |
| Context maps formales | Con 3-4 contextos, un README alcanza |

## Encaje con sdd y tdd

```
Spec (sdd)  → lenguaje ubicuo + reglas de negocio
   ↓
Modelo DDD  → aggregates + invariantes
   ↓          (cada invariante se vuelve un caso de test)
TDD (tdd)   → tests en rojo → implementación → verde
```

Ver `sdd` para el proceso de spec y `tdd` para el ciclo de testing.
