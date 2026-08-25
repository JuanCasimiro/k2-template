# Skill: k2-adaptar

**Trigger:** cuando el usuario modifica o rechaza algo que propuso Claude ("no, hacélo así", "eso no me gusta porque...", "prefiero que...", cuando reescribe algo de Claude, cuando pide un formato específico por segunda vez).

---

## Propósito

Aprender las preferencias del usuario sobre cómo quiere los entregables, el tono, el formato y el nivel de detalle. Guardar en `empresa/PREFERENCIAS.md` para que las próximas sesiones empiecen calibradas.

**El objetivo:** que el usuario nunca tenga que repetir la misma preferencia dos veces.

---

## Cuándo activar

La skill se activa cuando detectás alguno de estos patrones:

- El usuario dice "no, hacélo así" / "prefiero que..." / "eso no me sirve porque..."
- El usuario reescribe algo que generó Claude de forma significativa
- El usuario pide un formato o tono específico más de una vez
- El usuario expresa frustración con una respuesta ("esto es muy largo", "demasiado técnico")
- El usuario aprueba algo no-estándar ("perfecto, así siempre")

---

## Protocolo

### Paso 1 — Identificar qué cambió

Antes de guardar, entender:
- ¿Qué propuso Claude?
- ¿Qué modificó o rechazó el usuario?
- ¿Cuál es el patrón? (formato, tono, nivel de detalle, estructura, idioma)

### Paso 2 — Extraer la preferencia

Formular la preferencia de forma **reutilizable** (no "quiso que este documento fuera corto", sino "prefiere documentos concisos, máximo 1 página").

### Paso 3 — Actualizar `empresa/PREFERENCIAS.md`

Agregar o actualizar la entrada correspondiente.

### Paso 4 — Confirmar con el usuario (brevemente)

"Anotado: para los próximos documentos voy a [hacer X]." — rápido, sin ceremonia.

---

## Formato de `empresa/PREFERENCIAS.md`

```markdown
# Preferencias de [Nombre Empresa]

> Actualizado: [fecha]

## Formato de documentos

- [preferencia 1]
- [preferencia 2]

## Tono y lenguaje

- [preferencia sobre formalidad]
- [idioma, regionalismos preferidos]
- [nivel técnico preferido]

## Nivel de detalle

- [cuándo quiere detalle vs resumen]
- [cómo prefiere que se le presenten opciones]

## Estructura

- [cómo prefiere las listas vs párrafos]
- [si quiere siempre un "qué hacer ahora" al final]
- [preferencias sobre tablas, diagramas, etc.]

## Otros

- [cualquier preferencia que no entre en las categorías anteriores]
```

**Ejemplo real:**
```markdown
## Formato de documentos

- Prefiere listas por sobre párrafos largos
- Siempre incluir "próximo paso" al final de cada spec
- Máximo 1 página por documento; si hay más, poner resumen al inicio

## Tono y lenguaje

- Español rioplatense (vos, che, etc.)
- Directo, sin vueltas
- No usar palabras como "sinergia", "paradigma", "ecosistema"

## Nivel de detalle

- En planificación: quiere el por qué, no solo el qué
- En documentos operativos: solo lo necesario para ejecutar
- No quiere alternativas si ya hay una opción clara

## Estructura

- Tablas por sobre descripciones cuando hay datos comparativos
- No usa emojis en documentos formales
- Prefiere archivos separados por tema, no todo en uno
```

---

## Leer preferencias al inicio de sesión

Al comenzar una sesión nueva, leer `empresa/PREFERENCIAS.md` antes de generar cualquier entregable importante. Si el archivo existe, aplicar las preferencias sin mencionarlas explícitamente.

Si hay preferencias que afectan la sesión actual, tenerlas en cuenta desde el primer mensaje.

---

## No guardar

- Preferencias sobre contenido específico de un documento (no sobre el formato en general)
- Cambios de opinión únicos que no revelan un patrón
- Preferencias contradictorias sin resolución

Si hay duda sobre si algo es una preferencia real, preguntar: "¿Querés que recuerde esto para siempre o fue solo para este caso?"
