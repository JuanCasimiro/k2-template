# Skill: morning-review

**Trigger:** el usuario dice "buenos días", "qué tengo hoy", "cómo viene el día", "arranquemos", "qué hago hoy", o inicia la primera sesión del día antes de las 10am.

---

## Propósito

Analizar las últimas 24 horas de actividad y lo que viene hoy para que el CEO arranque con claridad: qué pasó, qué viene, dónde está el foco — y si el sistema detectó algo que podría aprender a hacer solo.

**Output:**
- Síntesis de ayer (qué pasó, qué quedó pendiente)
- Los 3 frentes del día (con revenue al frente)
- Patrón detectado + propuesta de skill nueva (si aplica)
- Briefing guardado en `empresa/briefings/YYYY-MM-DD-morning.md`

---

## Protocolo

### Paso 1 — Verificar si ya se corrió hoy

Buscar si existe `empresa/briefings/[fecha de hoy]-morning.md`.

- **Existe:** "Ya tenés el review de hoy. ¿Lo repaso o arrancamos con algo específico?"
- **No existe:** continuar con el paso 2.

---

### Paso 2 — Recolectar datos de las últimas 24h

Leer en este orden:

1. `empresa/CONTEXTO.md` — frentes activos, revenue, estructura
2. `empresa/reuniones/` — archivos cuya fecha en el nombre sea ayer (`YYYY-MM-DD-*.md` con la fecha de ayer)
3. `empresa/APRENDIZAJES.md` — últimas entradas (las más recientes)
4. `empresa/PREFERENCIAS.md` — si existe, aplicar tono sin mencionarlo
5. Si hay conector de **Google Calendar** disponible: leer eventos de ayer y de hoy
6. Si hay conector de **Gmail** disponible: leer asuntos de emails importantes de ayer

**Si el vault tiene poca información** (primer día o CEO no lo usó ayer):
→ Decirlo con una sola línea: "El sistema todavía tiene poca historia — cuanto más lo uses, más completo va a ser el review."
→ Igualmente generar el briefing con lo que hay en CONTEXTO.md.

---

### Paso 3 — Detectar patrones (antes de generar el output)

Antes de escribir el briefing, analizar mentalmente:

- ¿Hay algo que el CEO hizo 2+ veces esta semana sin una skill que lo ayude?
- ¿Hay una tarea que aparece en reuniones o aprendizajes como recurrente y tediosa?
- ¿Hay info que el CEO buscó manualmente que el sistema debería tener lista?

Si encontrás un patrón claro → preparar propuesta de skill para el paso 5.
Si no → continuar sin propuesta.

---

### Paso 4 — Generar el briefing

**Síntesis de ayer** (si hay datos):
- Qué se hizo: reuniones procesadas, decisiones tomadas, aprendizajes guardados
- Qué quedó pendiente: próximos pasos de reuniones sin cerrar
- Una sola observación: "Lo más importante de ayer fue..."

Si no hubo actividad registrada ayer → decirlo: "No quedó nada registrado de ayer. Hoy arrancamos de cero."

**Los 3 frentes de hoy:**
- Basado en: pendientes de ayer + frentes de revenue de CONTEXTO.md + calendario (si hay)
- **Revenue siempre primero**
- Formato: nombre del frente + acción concreta + por qué hoy

**El sistema aprendió:** (solo si hay patrón del paso 3 — omitir si no hay)
- Una línea describiendo el patrón detectado
- "Podría armarte una herramienta para esto — te lo propongo abajo"

Mostrar el briefing completo y continuar al paso 5 si hay propuesta.

---

### Paso 5 — Proponer skill si hay patrón

Si detectaste un patrón en el paso 3, después del briefing decir:

> "Noté que [descripción concreta del patrón]. ¿Querés que arme una herramienta específica para que el sistema lo haga solo la próxima vez?"

**Si el usuario dice sí:**
1. Generar el borrador de la skill (nombre, trigger, protocolo básico)
2. Mostrarlo: "Esto sería:"
3. Preguntar: "¿Lo guardamos así o cambiás algo?"
4. Al confirmar: escribir `.claude/skills/[nombre-en-kebab]/SKILL.md` con el contenido
5. Confirmar: "Listo. La próxima vez que [situación], el sistema ya sabe qué hacer."

**Si el usuario dice no:** no insistir. Registrar en `empresa/APRENDIZAJES.md`: "El CEO prefirió no crear la skill de [tema] — revisitar si vuelve a aparecer el patrón."

---

### Paso 6 — Guardar y cerrar

Guardar el briefing en `empresa/briefings/[YYYY-MM-DD]-morning.md`.

Si la carpeta `empresa/briefings/` no existe: crearla.

Cerrar siempre con: **"¿Por dónde arrancamos?"**
Nunca con "¿En qué más puedo ayudarte?" ni con opciones de menú.

---

## Formato del archivo guardado

```markdown
# Morning Review — [fecha]
> Generado: [hora]

## Ayer

### Qué pasó
- [bullet con reunión o decisión]
- [bullet con aprendizaje o avance]

### Quedó pendiente
- [pendiente] — [próximo paso]

### Lo más importante de ayer
[Una línea]

---

## Hoy

### Los 3 frentes
1. **[frente de revenue]** — [acción concreta]
2. **[segundo frente]** — [acción concreta]
3. **[tercer frente]** — [acción concreta]

---

## El sistema aprendió
_(Solo si se detectó patrón y/o se creó skill — omitir si no aplica)_

**Patrón:** [descripción]
**Skill creada:** [nombre] / **Propuesta rechazada:** [motivo]
```

---

## Reglas

- Si no hay datos de ayer: decirlo en una línea, generar el briefing igual con CONTEXTO.md
- No inventar reuniones ni pendientes que no estén en el vault
- Revenue siempre primero en los 3 frentes — sin excepciones
- La propuesta de skill es una sugerencia, nunca una imposición
- Si el CEO ya usó la misma skill 3+ veces y está funcionando bien: no proponer variaciones innecesarias
- El tono del coaching aplica: si ayer fue productivo → empujar al siguiente nivel. Si fue flojo → una acción concreta para hoy, nunca "tranquilo"
- Nunca mostrar rutas de archivo al usuario ("empresa/reuniones/") — hablar en lenguaje natural ("tus notas de ayer")
- Si hay Calendar y Gmail conectados: usarlos. Si no: trabajar solo con el vault, sin aclarar que faltan
