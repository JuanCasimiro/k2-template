# Skill: k2-onboarding

**Trigger:** cuando el usuario ejecuta `/k2-onboarding` o cuando es la primera vez que se abre el repo.

---

## Voz como modo primario

**El CEO no tiene que escribir nada.** El modo natural de uso es voz:
- En Claude.ai móvil: botón de micrófono → dicta → Claude procesa aunque haya ruido o cortes
- En Claude Code: el CEO dicta, el sistema transcribe y procesa
- Las respuestas del CEO pueden ser cortadas, con errores de transcripción, en mitad de una frase — Claude igual extrae lo importante
- Si una respuesta por voz es ambigua: Claude interpreta lo más probable y avisa al final del bloque, no interrumpe el flujo

**Ajuste de tono para voz:** cuando el CEO responde por voz, las respuestas son más largas, más informales y contienen más información de la necesaria. Claude filtra y estructura — no pide que reformulen.

---

## Propósito

Entrevistar al usuario para entender su empresa y su cabeza, y configurar el repo para su caso específico.

**Output:**
- `empresa/CONTEXTO.md` — descripción completa de la empresa (Claude la lee en cada sesión)
- `empresa/APRENDIZAJES.md` — archivo vacío listo para el chain log
- `empresa/PREFERENCIAS.md` — archivo vacío listo para la adaptación
- `CLAUDE.md` actualizado con el nombre y contexto de la empresa
- Primeros `proyectos/` sugeridos según los dolores detectados

---

## Cuándo ejecutar

- Primera vez que el cliente abre el repo
- Cuando la empresa cambia significativamente (nueva área, nueva persona clave, pivot)
- Si el fundador lo pide explícitamente

---

## Protocolo de entrevista

Conducir la entrevista como conversación natural, no interrogatorio. Hablar en el idioma y registro del cliente.

---

### Apertura — Las 3 preguntas (antes de los 6 bloques)

Antes de hacer cualquier pregunta operativa, abrir con esto. No explicar para qué sirven — simplemente preguntar.

**Decir primero:**
> "Antes de arrancar con el setup, te hago 3 preguntas. No tienen respuesta correcta — decí lo primero que te salga."

**Pregunta 1 — El estado deseado:**
> "¿Qué te hace feliz en tu trabajo? ¿Lo estás viviendo?"

Dejar que hable. Si dice que sí → "¿Todo el tiempo, o hay momentos donde no?" Si dice que no → ir directo a la Pregunta 2.

**Pregunta 2 — El bloqueo:**
> "¿Por qué no lo sos? ¿Qué ponés en el medio?"

Escuchar sin interrumpir. Cuando termina:

> "¿Y para qué sostenés eso? ¿Qué te da mantener esa situación?"

Esta pregunta genera el momento de reconocimiento. El CEO se ve desde afuera. No apurar — el silencio después de esta pregunta es valioso.

**Pregunta 3 — La solución propia:**
> "¿Cómo creés que se podría solucionar?"

**Con lo que respondió en estas 3 preguntas:**
1. Identificar el bloqueo de mayor peso emocional
2. Determinar qué skill del sistema puede resolverlo ahora mismo
3. Resolver UNO en el acto antes de continuar con los 6 bloques

Decir: *"Lo que me estás diciendo de [X] — lo resolvemos ahora antes de seguir. Dame un segundo."*

Ejecutar. Mostrar el resultado. No explicar cómo funciona — mostrar que funciona.

Recién después de ese momento → continuar con los 6 bloques.

---

**Antes de los 6 bloques, decir:**
> "Bueno. Ahora sí — te hago unas preguntas para que el sistema te conozca de verdad y pueda hacer esto siempre, no solo hoy. Son 6 bloques, unos 20-25 minutos. ¿Arrancamos?"

---

### Bloque 1 — Quién sos

- ¿Cómo se llama la empresa y qué hace en una frase?
- ¿Cuántas personas trabajan ahí? ¿Cómo es la estructura (vos solo, socio, equipo)?
- ¿En qué industria/sector operan?
- ¿Cuánto tiempo lleva la empresa? ¿En qué etapa está (arranque, crecimiento, maduro)?
- ¿Cuál es el producto o servicio principal que genera ingresos?
- ¿Cuál es el sueño grande detrás de todo esto?

---

### Bloque 2 — Qué hacés y tu día

- ¿Cómo es un día típico tuyo en la empresa?
- ¿Tenés un ritual de mañana? ¿Cuál? (si no tiene, anotar para diseñarlo después)
- ¿Cuándo rendís más para crear cosas nuevas? ¿Cuándo para las tareas administrativas?
- ¿Cuáles son los 3 procesos más importantes que hacés regularmente?
- ¿Hay ciclos recurrentes (semanal, mensual, por temporada)?
- ¿Cómo fluye el dinero? (cómo cobran, cómo pagan)
- ¿Cómo llegan los clientes? ¿Cómo los atienden?

---

### Bloque 3 — Con quién trabajás

- ¿Quiénes son las personas clave en la empresa (roles, no nombres necesariamente)?
- ¿Tenés clientes fijos o es transaccional?
- ¿Tenés proveedores importantes con los que interactuás seguido?
- ¿Hay alguien que tome decisiones además de vos?
- ¿Trabajás con externos (freelancers, contadores, abogados)?

---

### Bloque 4 — Qué te regula

Este bloque es el más importante. Tomarse el tiempo necesario.

- ¿Qué te saca de la cabeza cuando estás trabado o ansioso: arrancar a producir, o frenar y pensar?
- ¿Qué es lo primero que hacés en el día que te da sensación de que arrancaste bien?
- ¿Querés que te marque cuando no cumplís algo, o preferís que te lo diga más suave?
- ¿Cuál es tu auto-saboteo más común? (ej: cerrar un cliente y quedarse quieto, empezar muchas cosas sin terminar)
- ¿Cuáles son tus fugas de energía principales? (celular, mensajería en vivo, reuniones sin foco)

*(Registrar para la PARTE 3 del CLAUDE.md: qué lo regula, cómo hablarle, qué activar cuando entra en modo supervivencia.)*

---

### Bloque 5 — Qué duele

- ¿Qué es lo que más tiempo te consume y sentís que no debería?
- ¿Qué información perdés o te cuesta encontrar cuando la necesitás?
- ¿Qué decisiones tomás seguido que te gustaría poder delegar o automatizar?
- ¿Qué se repite semana a semana que es tedioso?
- ¿Qué haría diferente si tuvieras un asistente que conociera todo el negocio?

---

### Bloque 6 — Dónde está el conocimiento hoy

- ¿Dónde guardás la información del negocio? (hojas de cálculo, Word, WhatsApp, tu cabeza)
- ¿Tenés algo documentado (procesos, listas de clientes, precios)?
- ¿Cuánto de lo que sabés sobre el negocio está solo en tu cabeza?
- ¿Si alguien nuevo entrara mañana, podría entender cómo funciona la empresa?
- ¿Hay herramientas que usás hoy que van a seguir (ERP, CRM, etc.)?

---

### Cierre

Después de los 6 bloques:
1. Hacer un **resumen en voz alta** de lo entendido: "Lo que me estás diciendo es que..."
2. Preguntar: "¿Hay algo importante que no te pregunté?"
3. Decirle los **3 cuellos de botella más urgentes** (no preguntarlos — decírselos)
4. **Mapear cada dolor a una solución concreta** — para cada cuello de botella, decir explícitamente qué puede hacer el sistema hoy:

| Si el dolor es... | El sistema ya puede... |
|-------------------|----------------------|
| "Salgo de reuniones y pierdo todo" | Procesar cualquier reunión en 30 segundos desde el audio |
| "Nunca mando las propuestas a tiempo" | Generar la propuesta lista para enviar después de procesar la reunión |
| "No sé qué hacer primero cada mañana" | Armar los 3 frentes del día con revenue primero, en 15 segundos |
| "Tengo info dispersa en WhatsApp y hojas de cálculo" | Procesar cualquier archivo, foto o texto y guardarlo estructurado |
| "No sé cómo preparar reuniones importantes" | Generar un perfil del cliente con cómo encararlo antes de entrar |
| "No sé cómo viene la semana" | Hacer el reporte semanal al instante con todo lo que el sistema capturó |
| "Quiero documentar mis procesos pero nunca tengo tiempo" | Diseñar el proceso en una conversación y generar el documento listo |
| "No sé si una herramienta de IA me sirve" | Evaluarla en contexto de su operación específica, no en abstracto |

Si el dolor que mencionó no aparece en la tabla → proponer crear una skill específica: "Eso todavía no lo tiene el sistema, pero lo armamos en 5 minutos. ¿Lo hacemos ahora?"

5. **Resolver uno en el acto** — elegir el dolor de mayor peso emocional (el que más veces mencionó, el que dijo con más carga) y ofrecer resolverlo ahí mismo:
   > "Ese que dijiste de [X] — lo resolvemos ahora. Dame los datos y lo hacemos."
   
   Ejecutar la skill correspondiente dentro del mismo onboarding. El CEO termina la sesión habiendo resuelto algo real, no solo habiendo "configurado un sistema".

6. Proponer qué áreas/proyectos crear primero
7. Proponer y arrancar el **primer loop del día**: "Los 3 frentes de hoy, con revenue al frente. ¿Los armamos ahora?"

---

## Generación de archivos

### `empresa/CONTEXTO.md`

```markdown
# Contexto de [Nombre Empresa]

> Generado: [fecha]
> Revisado: [fecha]

## La empresa

**Nombre:** [nombre]
**Industria:** [sector]
**Etapa:** [arranque / crecimiento / maduro]
**Equipo:** [cantidad y estructura]

**Qué hace en una frase:** [descripción]

## Procesos principales

1. [proceso 1 — qué es, quién lo hace, qué frecuencia]
2. [proceso 2]
3. [proceso 3]

## Personas clave

| Rol | Qué hace | Quién toma decisiones de |
|-----|----------|--------------------------|
| [rol 1] | [descripción] | [área] |

## Clientes y proveedores

**Perfil de cliente:** [descripción breve]
**Principales proveedores:** [si aplica]

## Dolores identificados

1. **[dolor principal]** — [descripción breve]
2. [dolor 2]
3. [dolor 3]

## Cómo se regula

**Qué lo activa:** [qué tipo de acción apaga la ansiedad/supervivencia — ej: primer acto de revenue del día]
**Auto-saboteo típico:** [ej: cerrar un cliente y frenar]
**Fugas de energía:** [ej: mensajería en vivo, celular durante la mañana]
**Cómo hablarle cuando entra en modo supervivencia:** [acción concreta, nunca "tranquilo"]

## El día ideal

**Ritual de mañana:** [descripción]
**Pico de foco:** [mañana para crear / tarde para producir / etc.]
**Bloques de trabajo:** [estructura]

## Revenue

**Qué cuenta como revenue hoy:** [descripción]
**Frentes que generan plata hoy:** [lista]
**Frentes que son apuesta futura:** [lista]

## Herramientas actuales

- [lista de lo que ya usan]

## Dónde está el conocimiento hoy

- [hojas de cálculo, WhatsApp, etc.]

## Primeras áreas a trabajar con Claude

1. [área 1 — por qué es prioritaria]
2. [área 2]
```

### Actualizar `CLAUDE.md`

Completar las PARTES 1 al 7 del CLAUDE.md con toda la información recopilada. Reemplazar todos los [corchetes] con datos reales del cliente. No dejar ningún [corchete] vacío — si no se obtuvo el dato, escribir "a definir".

### Crear primeros proyectos

Basado en los dolores identificados, crear carpetas en `proyectos/`:
- `proyectos/[area-principal]/COPILOT.md` con descripción del área
- `proyectos/[area-principal]/docs/PRODUCT_MEMORY.md` vacío

---

### Mostrar al usuario al cerrar

```
Sistema configurado.

Desde la computadora:
→ Abrí Claude Code en esta carpeta y escribí lo que necesitás.

Desde el teléfono:
→ Activá Remote Control y manejás la misma sesión desde el celular.

Para arrancar, mirá EMPEZA-ACA.md
```

---

## Comportamiento en el repo del cliente

Cuando el cliente ejecuta `/k2-onboarding` (o Claude lo detecta como primer uso):
1. Claude conduce la entrevista en 6 bloques
2. Genera los archivos (`empresa/CONTEXTO.md`, completa `CLAUDE.md`, crea proyectos/)
3. Hace los commits
4. Le muestra el mensaje de cierre

