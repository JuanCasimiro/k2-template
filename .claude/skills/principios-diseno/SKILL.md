# Skill: principios-diseno

**Trigger:** cuando se está diseñando un proceso, un flujo de trabajo, una forma de organizar información, o cómo debe funcionar algo.

---

## Propósito

Aplicar principios que hacen que los procesos y sistemas se usen de verdad — no que existan en papel.

---

## Los 4 principios

### 1. Facilidad operativa — "10 → 1000 acciones por minuto"

Lo que se hace seguido tiene que ser rápido. Las acciones repetitivas se agrupan y se hacen en lote.

**Ejemplo:** si todos los lunes revisás 20 facturas, el sistema debería poder mostrarte solo las pendientes y dejarte aprobarlas todas de una vez, no una por una.

**Preguntas para diseñar:**
- ¿Cuánto tiempo tarda hacer esto una vez? ¿Y 10 veces?
- ¿Qué pasos son repetitivos y podrían agruparse?
- ¿El filtro podría preparar la lista exacta de lo que hay que actuar?

---

### 2. Red de seguridad para lo importante

Las acciones que tienen consecuencias (dinero, compromisos, datos sensibles) necesitan un paso de confirmación. Y el historial de lo que se aprobó/rechazó no se borra.

**Ejemplo:** antes de confirmar un pedido grande a un proveedor, el sistema muestra el resumen y pide confirmación. Los pedidos rechazados quedan en el historial por si hay que consultarlos después.

**Preguntas para diseñar:**
- ¿Qué pasa si esto sale mal? ¿Es reversible?
- ¿Hay un paso de revisión antes de comprometer algo importante?
- ¿Se guarda el historial de decisiones (incluyendo los "no")?

---

### 3. Diseñar por flujos, no por listas

Las personas piensan en tareas y flujos, no en menús y secciones. El sistema debería organizarse como la gente trabaja.

**Ejemplo:** un vendedor no piensa "voy a Módulo Clientes, luego a Módulo Pedidos". Piensa "estoy hablando con [cliente] y quiero saber qué me compró antes y registrar este nuevo pedido".

**Preguntas para diseñar:**
- ¿Cómo describe esta persona su trabajo en palabras? (esas son las tareas reales)
- ¿Dónde está el usuario cuando necesita hacer esto?
- ¿Qué viene después de esta acción? ¿Está conectado?

---

### 4. Construir para hoy, dejar costuras para mañana

Resolver el problema actual bien. No construir para un futuro hipotético que quizás no llega. Pero sí dejar la estructura lista para crecer sin tener que rehacer todo.

**Ejemplo:** hoy manejamos un solo depósito. El proceso de inventario se diseña para uno, pero el campo "depósito" ya existe en caso de que haya más después.

**Preguntas para diseñar:**
- ¿Qué problema concreto resolvemos hoy?
- ¿Hay algo en el diseño que bloquea el crecimiento futuro?
- ¿Qué cosas deberían ser genéricas desde el inicio (aunque hoy solo haya un caso)?

---

## Cómo aplicar estos principios

Al diseñar cualquier proceso, checklist:

- [ ] ¿Las acciones repetitivas están en lote?
- [ ] ¿Hay red de seguridad para decisiones importantes?
- [ ] ¿Está organizado por flujo de trabajo, no por sección/menú?
- [ ] ¿El diseño de hoy no bloquea el crecimiento?
