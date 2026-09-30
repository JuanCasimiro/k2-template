---
name: historias-de-usuario
description: Escribe historias de usuario con escenarios Gherkin en español para cualquier funcionalidad, existente o nueva. Lee el código si hace falta, pregunta lo mínimo y produce historias con cobertura completa. Son el puente entre la spec (sdd) y los tests (tdd). Triggers on "historia de usuario", "user story", "escribir historia", "gherkin", "escenarios", "documentar funcionalidad", "escribir casos", "criterios de aceptación".
---

# Historias de usuario — historias y escenarios

Una historia dice **quién quiere qué y para qué**; los escenarios dicen **cómo se ve cuando funciona y cuando no**. Salen de la spec (`sdd`) y cada escenario se convierte en un test (`tdd`).

## Cuándo se activa

- **Código existente:** "escribí las historias del módulo de ventas" → leés el código y las generás.
- **Feature nueva:** "escribí la historia para X" → preguntás lo mínimo y las generás.
- **Actualización:** "actualizá la US-03" → leés el archivo existente y lo modificás.

## Paso 1 — Recolectar contexto

**Código existente:** leé los archivos del módulo para entender qué acciones puede hacer cada rol, qué datos entran y salen, qué estados existen y qué restricciones de permiso hay. Preguntá solo lo que no se infiere: el **"para qué"** (casi nunca está en el código) y los bordes no obvios.

**Feature nueva:** preguntas mínimas indispensables:
- ¿Quién la usa? (rol exacto)
- ¿Qué acción realiza? (verbo + objeto)
- ¿Qué beneficio obtiene? (el "para qué")
- Errores o bordes relevantes.

## Paso 2 — Escribir la historia

```markdown
## US-[N]: [Título corto en imperativo]

**Como** [rol],
**quiero** [acción concreta],
**para** [beneficio de negocio].

**Roles con acceso:** [lista]
**Módulo:** [ruta o componente]
**Plan requerido:** [si hay planes]
```

Reglas:
- **Una historia = un actor + un objetivo.** Dos roles con objetivos distintos son dos historias.
- **El "para" es obligatorio y no tautológico.** "Para poder verlo" no vale; "para decidir compras sin pedir ayuda al dueño" sí.
- **Granularidad media:** ni "gestionar stock" ni "hacer clic en guardar".
- **Título en imperativo:** "Ver historial de ventas", no "Historia del historial".

## Paso 3 — Escenarios Gherkin

```gherkin
### Escenario [N]: [nombre descriptivo]
Dado [precondición: estado]
Cuando [acción del usuario]
Entonces [resultado observable]
Y [efecto adicional]
```

### Cobertura mínima por historia

| Tipo | Descripción | Obligatorio |
|------|-------------|-------------|
| Camino feliz | El flujo normal funciona | siempre |
| Sin permiso | Rol incorrecto no accede | si hay restricción de rol |
| Estado vacío | Aún no hay datos | si hay listas o grillas |
| Error de validación | Input inválido o faltante | si hay formularios |
| Error de sistema | Falla de red o de backend | si hay llamadas async |
| Borde de negocio | Stock = 0, trial vencido, primer uso | según dominio |
| Aislamiento | No ve datos de otro tenant | si es multi-tenant (ver `rls-multi-tenant`) |

### Reglas de escenarios
- **Gherkin en español:** `Dado` / `Cuando` / `Entonces` / `Y` / `Pero`.
- **El "Dado" describe estado, no acción:** "Dado que el usuario tiene sesión iniciada" sí; "Dado que hace login" no.
- **El "Entonces" es observable:** lo que el usuario ve o lo que queda en los datos; nunca "el sistema procesa".
- **Sin implementación:** nada de nombres de componentes, SQL ni endpoints. Solo comportamiento visible.
- **Un escenario = una situación.** No combinar dos casos.
- Si la historia toca una entidad con estado, hay escenario por cada transición (ver `ciclo-de-vida-entidades`).

## Paso 4 — Archivos

```
docs/historias/
├── README.md              ← índice
├── US-01-stock-consulta.md
├── US-02-ventas-mostrador.md
└── ...
```

Archivo individual:

```markdown
# US-[N]: [Título]

**Módulo:** [nombre] · **Roles:** [lista] · **Plan:** [..]
**Estado:** [borrador | aprobada | implementada] · **Creada:** [fecha]

## Historia
Como [rol], quiero [acción], para [beneficio].

## Escenarios
### Escenario 1: [nombre]
Dado ... Cuando ... Entonces ...
```

Índice `README.md`: tabla `ID | Título | Módulo | Estado`.

## Ejemplo

```markdown
# US-03: Registrar una venta en mostrador

Como operario, quiero registrar una venta desde el mostrador,
para descontar el stock automáticamente y entregar el comprobante al cliente.

### Escenario 1: venta exitosa
Dado que el operario tiene una caja abierta
Y existe el producto "Filtro" con stock mayor a 0
Cuando lo agrega al carrito y confirma el pago en efectivo
Entonces se registra la venta y el stock baja en la cantidad vendida
Y se muestra el comprobante

### Escenario 2: sin caja abierta
Dado que no hay ninguna caja abierta
Cuando el operario entra a ventas
Entonces se le pide abrir una caja antes de operar

### Escenario 3: sin stock suficiente
Dado que "Filtro" tiene stock 0
Cuando intenta agregarlo al carrito
Entonces ve una advertencia de stock insuficiente
Y no puede confirmar esa cantidad

### Escenario 4: rol sin acceso a finanzas
Dado que el usuario tiene rol operario
Cuando intenta entrar a finanzas
Entonces ve acceso denegado
```

## Numeración

Consultá `docs/historias/README.md` para el último ID y seguí la secuencia (si no existe, `US-01`). Los IDs no se reutilizan: una historia descartada queda marcada `[deprecada]` en el índice.

## Encaje con el flujo

`sdd` (spec: problema, flujo, casos edge) → **historias-de-usuario** (escenarios verificables) → `tdd` (cada escenario pasa a ser un test; los de dominio como unit, los de recorrido como integración/e2e) → `ddd` (los términos de las historias son el lenguaje ubicuo del código).
