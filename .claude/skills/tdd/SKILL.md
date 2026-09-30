---
name: tdd
description: Test Driven Development — ciclo Red-Green-Refactor. Se testea primero el dominio y después la infraestructura, con repositorios in-memory en vez de mocks. Los ejemplos están en TypeScript + Vitest pero el método aplica a cualquier stack. Triggers on tests, TDD, test driven, escribir tests, testing, vitest, jest, pruebas unitarias, integration tests, red green refactor.
---

# Test Driven Development (TDD)

TDD invierte el orden: **primero el test (rojo), después el código mínimo para que pase (verde), después refactor**. El test describe el comportamiento antes de que exista.

## Cuándo usarlo

Siempre en **lógica de dominio** (aggregates, value objects, servicios de dominio, reglas de negocio) y en todo bug (primero el test que lo reproduce). Opcional en CRUD puro o glue code de infraestructura.

## Ciclo Red-Green-Refactor

```
1. RED      — escribís el test. Falla porque el código no existe.
2. GREEN    — el mínimo código para que pase. No sobre-construyas.
3. REFACTOR — limpiás sin romper; los tests garantizan que no rompiste nada.
```

**Regla:** nunca escribas más código del necesario para pasar el test en rojo. Verificá que el test **falla por la razón correcta** antes de pasarlo a verde; un test que nunca estuvo rojo no prueba nada.

## Pirámide

```
        /\
       /e2e\       pocos — flujos completos (Playwright)
      /------\
     / integr \    algunos — workflow completo in-memory o contra una DB de test
    /----------\
   /    unit    \  muchos — aggregates, value objects, servicios de dominio
  /______________\
```

Stack de ejemplo: Vitest para unit + integración; Playwright para e2e si hace falta.

## Orden de testing (del dominio hacia afuera)

### 1. Value Objects (los más simples, empezá acá)

```ts
describe('Monto', () => {
  it('no permite montos negativos', () => {
    expect(() => new Monto(-1)).toThrow('Monto no puede ser negativo');
  });
  it('suma dos montos', () => {
    expect(new Monto(100).suma(new Monto(200))).toEqual(new Monto(300));
  });
  it('es igual con mismo valor y moneda', () => {
    expect(new Monto(100).equals(new Monto(100))).toBe(true);
  });
});
```

### 2. Aggregates — invariantes, transiciones, eventos

```ts
describe('SolicitudDeCompra', () => {
  it('no se crea sin artículos', () => {
    expect(() => new SolicitudDeCompra('s1', 'prov1', [], new Date(), 'user1'))
      .toThrow('Una solicitud debe tener al menos un artículo');
  });

  describe('transiciones', () => {
    let s: SolicitudDeCompra;
    beforeEach(() => { s = solicitudValida(); });

    it('pasa de DRAFT a SUBMITTED', () => {
      s.submit();
      expect(s.estado.type).toBe('SUBMITTED');
    });
    it('no se envía dos veces', () => {
      s.submit();
      expect(() => s.submit()).toThrow();
    });
    it('emite SolicitudSubmitted al enviarse', () => {
      s.submit();
      expect(s.getPendingEvents()).toContainEqual(expect.any(SolicitudSubmitted));
    });
  });

  it('no recibe más de lo pedido', () => {
    expect(() => solicitudAprobada().recibirArticulos(new Map([['art1', 99]])))
      .toThrow('Recibiste 99 pero solicitaste');
  });
});
```

### 3. Servicios de dominio (lógica cross-aggregate)

```ts
it('rechaza si el proveedor está inactivo', async () => {
  const repo = new InMemoryProveedorRepository();
  await repo.save(proveedor({ estado: 'INACTIVO' }));

  const r = await new AprobadorDeCompras(repo).puedoAprobar(solicitudValida());

  expect(r.canApprove).toBe(false);
  expect(r.reason).toMatch(/Proveedor no activo/);
});
```

### 4. Casos de uso (application services)

```ts
it('guarda la solicitud y publica el evento', async () => {
  const repo = new InMemorySolicitudRepository();
  const bus = new InMemoryEventBus();

  await new CrearSolicitudUseCase(repo, new InMemoryProveedorRepository(), bus)
    .execute({ proveedorId: 'prov1', items: [itemValido()], createdBy: 'user1' });

  expect(await repo.count()).toBe(1);
  expect(bus.published).toHaveLength(1);
});
```

### 5. Repositorios reales (integración)

Contra una DB de test (o local), verificá que persistir y reconstruir el aggregate devuelve el mismo estado. Probá también las reglas que viven en la base (constraints, RLS — ver `rls-multi-tenant`, `migraciones-db`).

## Builders / factories de test

Nunca repitas la construcción de objetos de prueba:

```ts
export function solicitudValida(o?: Partial<SolicitudParams>): SolicitudDeCompra {
  return new SolicitudDeCompra(o?.id ?? 'test-id', o?.proveedorId ?? 'prov-test',
    o?.lineas ?? [lineaValida()], new Date(), 'user-test');
}
export function solicitudAprobada() {
  const s = solicitudValida(); s.submit(); s.approve('approver-test'); return s;
}
```

## Repositorios in-memory en vez de mocks

Implementá la interfaz del repositorio en memoria; no la mockees.

```ts
export class InMemorySolicitudRepository implements ISolicitudRepository {
  private store = new Map<string, SolicitudDeCompra>();
  async save(s: SolicitudDeCompra) { this.store.set(s.id, s); }
  async getById(id: string) { return this.store.get(id) ?? null; }
}
```

Un mock verifica que llamaste a los métodos "correctos"; un in-memory prueba el comportamiento real y sobrevive a refactors.

## Qué NO testear

| No testear | Razón |
|-----------|-------|
| Getters sin lógica | No hay comportamiento |
| El cliente de DB directamente | Testeá el repositorio, no la librería |
| UI sin lógica | e2e si hace falta |
| Implementación interna | Testeá el contrato, no el cómo |
| Casos que el tipo ya impide | El compilador ya lo prueba |

## Configuración mínima (Vitest)

```ts
// vitest.config.ts
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    environment: 'node',
    include: ['tests/**/*.spec.ts', 'src/**/*.spec.ts'],
    coverage: {
      provider: 'v8',
      include: ['src/domains/**', 'src/application/**'],
      exclude: ['src/infrastructure/**', 'src/api/**'],
    },
  },
});
```

Cobertura exigida en dominio y aplicación; opcional en infraestructura.

## Encaje con sdd, historias-de-usuario y ddd

```
Spec (sdd)
  "Casos edge" e invariantes  → tests en rojo
  Flujo principal             → test de integración
Historias de usuario (historias-de-usuario)
  cada escenario Gherkin      → un test (dominio: unit; recorrido: integración/e2e)
Aggregate (ddd)
  cada invariante             → un it('no permite ...')
  cada transición de estado   → un it('pasa de A a B')
  cada evento de dominio      → un it('emite ...')
```

Un bug reportado sigue la misma vía: test que lo reproduce (rojo) → fix (verde) → refactor. Para el modelo de dominio, `ddd`; para el proceso de spec, `sdd`; antes de mergear, `pre-ship`.
