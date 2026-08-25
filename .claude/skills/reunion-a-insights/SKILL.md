# Skill: reunion-a-insights

**Trigger:** cuando el usuario menciona una reunión que tuvo, pega una transcripción, dice "resumime la reunión con X", "qué saqué de la reunión de hoy", "tuve una reunión con [nombre]", "analizame esta conversación", o cuando comparte texto que parece ser una transcripción o resumen de una reunión.

---

## Propósito

Convertir la transcripción o descripción de una reunión en un documento estructurado con insights accionables: resumen, puntos clave, perfil del otro participante y próximos pasos claros. Guardar en el repo para construir memoria de empresa.

**Output:** archivo `empresa/reuniones/YYYY-MM-DD-[nombre].md` + commit automático.

---

## Cuándo ejecutar

- El usuario pega una transcripción de una reunión
- El usuario describe de qué fue una reunión (aunque sea brevemente)
- El usuario dice "resumime la reunión con X", "qué saqué de hablar con Y"
- El usuario comparte texto de una conversación importante

---

## Protocolo

### Paso 1 — Verificar que hay material para trabajar

Si el usuario solo dice "tuve una reunión con Martín" sin dar más contexto:
→ Preguntar: "¿Me podés pegar la transcripción o contarme de qué trataron? Con eso puedo darte un análisis completo."

Si la transcripción es muy corta (menos de 5 líneas) o muy vaga:
→ Procesarla igual, pero avisar al final: "Trabajé con poca información — si querés ampliar algo, avisame y mejoro el análisis."

### Paso 2 — Identificar fecha y participante

- Si no se menciona fecha: usar la fecha de hoy
- Si no se menciona el nombre del otro participante: preguntar antes de generar el archivo — "¿Cómo se llama o cómo lo querés identificar?"

### Paso 3 — Generar el análisis

**Resumen ejecutivo** (máximo 3 bullets):
- Qué se discutió en una línea
- Cuál fue el resultado o conclusión principal
- Cuál es el próximo paso más importante

**Puntos clave y decisiones:**
- Lista de lo más relevante dicho o decidido
- Marcar explícitamente qué quedó pendiente vs. qué se cerró

**Perfil del participante** (si es cliente, prospecto o proveedor):
- Cómo se comunicó: ¿desde el miedo, la ambición, la urgencia, el escepticismo?
- Qué necesita (explícito e implícito)
- Puntos de dolor que mencionó
- Señales positivas y red flags

**Próximos pasos:**
- Lista numerada, con responsable (el CEO o el otro) y fecha si se mencionó
- Si no se dio fecha: proponer una razonable y marcarla como "sugerida"

**Recomendación de seguimiento:**
- Cuándo contactar
- Por qué medio (WhatsApp, email, llamada)
- Qué decir o proponer en el próximo contacto

### Paso 4 — Coaching de comunicación (solo si hay diálogo del CEO en la transcripción)

Si la transcripción permite observar cómo se comunicó el CEO (hay diálogo real, no solo un resumen de lo que dijo el otro), agregar al final del análisis:

**Cómo te comunicaste:**
- Qué hiciste bien: [observación concreta de la transcripción]
- Una cosa para mejorar la próxima vez: [observación concreta y accionable]
- Próximo paso accionable: [algo específico para hacer antes de la próxima reunión similar]

**El tono del coaching (no negociable):**

1. **Directo a ejecutar.** No dar "reflexiones" — dar acciones. Si la reunión fue bien: "Bien. ¿Cuándo es el próximo contacto con [nombre]? Armemos el mensaje ahora." Si fue mal: una sola cosa para corregir, concreta, y ya.
2. **Entrenador, no sí-señor.** Si el CEO evitó el cierre, se lo decís — sin culpa, pero sin dejarlo pasar. "Faltó cerrar el próximo paso en vivo. La próxima: antes de terminar la reunión, agendás la siguiente llamada."
3. **Anti-conformismo.** Si la reunión resultó bien (cerró algo, buen vínculo), empujás al siguiente: "Bien. ¿Cuándo mandamos la propuesta?" No te quedás en el elogio.
4. **Apagá supervivencia vía acción.** Si el CEO viene frustrado o inseguro después de la reunión, no decís "tranquilo, estuvo bien". Decís: "La próxima reunión con alguien similar es [X]. Armemos la apertura ahora para que no arranques de cero."
5. **Honesto y ambicioso.** Subí la vara con observaciones reales, nunca con halagos. "Hablaste mucho de tu proceso, poco de su dolor. La próxima: primero su dolor, después tu solución."

**Reglas:**
- Siempre empezar por lo positivo (una sola observación concreta de lo que funcionó)
- Una sola cosa para mejorar — no una lista
- Si la transcripción no muestra cómo habló el CEO (solo hay notas o resumen sin diálogo): omitir esta sección completamente, sin aclarar que la omitís
- Nunca genérico ("podrías mejorar la comunicación") — si no hay observación concreta, no decir nada
- El objetivo no es que se sienta bien — es que en la próxima reunión lo haga mejor

### Paso 5 — Confirmar antes de guardar

Mostrar el análisis completo (incluyendo coaching si aplica) y preguntar:
"¿Está bien esto o querés cambiar algo antes de guardarlo?"

Si el usuario aprueba → continuar. Si pide cambios → ajustar y mostrar de nuevo.

### Paso 6 — Guardar

Guardar en: `empresa/reuniones/YYYY-MM-DD-[nombre-en-kebab-case].md`

Si la carpeta `empresa/reuniones/` no existe: crearla.

Confirmar al usuario: "Guardado. Lo vas a encontrar en empresa/reuniones/."

---

## Formato del archivo generado

```markdown
# Reunión con [Nombre] — [Fecha]

> Tipo: [cliente / prospecto / proveedor / otro]
> Canal: [presencial / Zoom / WhatsApp / otro]

## Resumen

- [bullet 1]
- [bullet 2]
- [bullet 3]

## Puntos clave

- [punto 1]
- [punto 2]

**Quedó pendiente:**
- [pendiente 1]

## Perfil de [nombre]

**Cómo se comunica:** [descripción]
**Lo que necesita:** [descripción]
**Dolores detectados:** [lista]
**Red flags:** [si hay / ninguno]

## Próximos pasos

1. [acción] — [responsable] — [fecha o "a definir"]
2. [acción] — [responsable] — [fecha o "a definir"]

## Recomendación de seguimiento

**Cuándo:** [fecha o plazo]
**Cómo:** [medio]
**Mensaje sugerido:** [breve descripción de qué decir]

## Cómo te comunicaste

**Lo que funcionó:** [observación concreta — una sola]
**Para la próxima:** [una sola cosa accionable — específica, no genérica]
**Acción inmediata:** [algo concreto para hacer ahora o antes de la próxima reunión similar]
```

_(La sección "Cómo te comunicaste" solo aparece si la transcripción muestra diálogo del CEO. Si no hay suficiente material, se omite.)_

---

## Reglas

- Nunca borrar un archivo existente — siempre crear uno nuevo con la fecha en el nombre
- Nunca inventar datos que no estén en la transcripción — si falta info, dejarlo en blanco o marcarlo como "sin datos"
- Si hay varias personas en la reunión: hacer una sección de perfil por cada una relevante
