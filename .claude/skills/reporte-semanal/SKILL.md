# Skill: reporte-semanal

**Trigger:** cuando el usuario pide un resumen de la semana, dice "¿cómo vino la semana?", "¿qué hice esta semana?", "dame el reporte semanal", "¿cómo está la empresa?", "¿qué tengo que hacer la semana que viene?", "cerremos la semana", o cuando quiere revisar el estado general de sus proyectos.

---

## Propósito

Generar un reporte ejecutivo del estado de la empresa en la semana: qué avanzó, qué está trabado, reuniones importantes, aprendizajes y foco recomendado para la semana siguiente. Guardarlo en el repo para construir historial.

**Output:** reporte en pantalla + archivo `empresa/reportes/reporte-YYYY-semana-NN.md` + commit.

---

## Cuándo ejecutar

- El CEO lo pide explícitamente
- Es viernes y el CEO quiere hacer un cierre de semana
- El CEO quiere ver el estado general de sus proyectos

---

## Protocolo

### Paso 1 — Reunir la información disponible

Leer en este orden si los archivos existen:
1. `empresa/CONTEXTO.md` — proyectos activos y descripción del negocio
2. `proyectos/` — listar todas las carpetas de proyectos activos y leer cada `PRODUCT_MEMORY.md` que exista
3. `empresa/reuniones/` — filtrar reuniones de los últimos 7 días (por fecha en el nombre del archivo)
4. `empresa/APRENDIZAJES.md` — entradas de los últimos 7 días

**Si no hay proyectos registrados:**
→ Avisar: "No tengo proyectos registrados todavía. ¿Querés que creemos uno ahora?" y no continuar hasta que el CEO responda.

**Si el repo está vacío o con muy poca información:**
→ Reportar lo que existe y aclarar: "El sistema todavía tiene poca información — cuanto más lo usés durante la semana, más completo y útil va a ser el reporte del viernes."

### Paso 2 — Identificar la semana

Calcular el número de semana ISO del año y el rango de fechas (lunes a viernes de la semana actual).

### Paso 3 — Generar el reporte

**Estado de proyectos:**
Para cada proyecto activo:
- Nombre del proyecto
- Qué avanzó esta semana (basado en archivos con fechas recientes o notas en PRODUCT_MEMORY)
- Qué está trabado o pendiente
- Estado general: en marcha / en pausa / bloqueado

Si un proyecto no tuvo actividad esta semana → decirlo explícitamente ("sin movimiento esta semana"), no omitirlo.

**Reuniones de la semana:**
- Lista de reuniones de `empresa/reuniones/` de los últimos 7 días
- Para cada una: con quién fue y el punto más importante
- Si no hubo reuniones registradas: decirlo

**Aprendizajes de la semana:**
- Entradas recientes de `empresa/APRENDIZAJES.md`
- Si no hay: decirlo

**Foco sugerido para la semana próxima:**
- Top 3 prioridades en orden de importancia
- Basadas en: lo que está trabado, las reuniones que hubo, los proyectos activos
- Justificar brevemente por qué cada una es prioritaria

### Paso 4 — Mostrar y preguntar si guardar

Mostrar el reporte completo y preguntar:
"¿Guardamos el reporte de esta semana?"

Si el usuario confirma → calcular el nombre del archivo y guardar:

Nombre: `empresa/reportes/reporte-[YYYY]-semana-[NN].md`

Si la carpeta `empresa/reportes/` no existe: crearla.

Confirmar: "Guardado. Lo vas a encontrar en empresa/reportes/."

---

## Formato del archivo generado

```markdown
# Reporte semanal — Semana [NN] ([fecha inicio] al [fecha fin])

## Estado de proyectos

### [Nombre proyecto]
- **Avanzó:** [qué]
- **Pendiente:** [qué]
- **Estado:** en marcha / en pausa / bloqueado

### [Siguiente proyecto...]

## Reuniones de la semana

- **[fecha] — [nombre]:** [punto más importante]
- _(Sin reuniones registradas esta semana)_

## Aprendizajes

- [aprendizaje 1]
- _(Sin aprendizajes registrados esta semana)_

## Foco para la semana que viene

1. **[prioridad 1]** — [por qué]
2. **[prioridad 2]** — [por qué]
3. **[prioridad 3]** — [por qué]
```

---

## Reglas

- Nunca inventar datos, métricas o avances que no estén en el repo
- Si un proyecto no tuvo actividad: decirlo, no omitirlo ni inventar progreso
- El foco sugerido es una recomendación, no una orden — el CEO puede cambiarlo
- Si es el primer reporte (repo nuevo): avisarlo y proponer cómo empezar a cargar datos
- No guardar sin confirmación del usuario
