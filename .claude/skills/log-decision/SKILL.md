# Skill: log-decision

**Trigger:** cuando el usuario menciona una decisión que tomó ("decidí", "quedamos en", "vamos a hacer", "la decisión es", "lo vamos a hacer así", "elegimos", "descartamos", "la dirección es"), o cuando retroactivamente aclara una decisión anterior ("en realidad decidí", "cambié de idea", "al final fue").

La diferencia con `k2-aprender`: log-decision tiene la vara baja — cualquier decisión vale, aunque no cambie un proceso. k2-aprender es para cuando algo cambió cómo se va a hacer algo de ahora en más.

---

## Propósito

Registrar decisiones con contexto suficiente para entenderlas en 3 meses: qué se decidió, por qué, y qué alternativas se descartaron.

**Output:** entrada en `empresa/decisiones/YYYY-MM.md` (archivo mensual, se agrega al final).

---

## Protocolo

### Paso 1 — Extraer la decisión

De lo que dijo el usuario, identificar:
- **La decisión concreta** (qué se hizo / qué se va a hacer)
- **El contexto o por qué** (si lo mencionó — si no, dejar vacío)
- **Alternativas descartadas** (si las mencionó)

No pedir más información a menos que la decisión sea demasiado ambigua para registrarla.

### Paso 2 — Confirmar antes de guardar

Mostrar el resumen de la decisión en formato compacto:

```
Decisión: [la decisión en una línea]
Por qué: [contexto — o "no especificado"]
Alternativas descartadas: [o "ninguna mencionada"]
```

Preguntar: "¿Lo guardo así?"

Si el usuario quiere ajustar → ajustar y confirmar de nuevo.

### Paso 3 — Guardar

Archivo destino: `empresa/decisiones/[YYYY-MM].md`

Si el archivo no existe, crearlo con el header:
```markdown
# Decisiones — [Mes YYYY]
```

Agregar al final del archivo:

```markdown
---

## [Fecha] — [Decisión en una línea]

**Decidido:** [descripción completa]
**Por qué:** [contexto / razón]
**Alternativas descartadas:** [si aplica]
**Tags:** [área relevante — ej: producto, ventas, equipo, operaciones]
```

Confirmar: "Guardado en empresa/decisiones/[YYYY-MM].md."

---

## Reglas

- Fecha = fecha en que se tomó la decisión, si la mencionó. Si no → fecha de hoy.
- Una decisión por entrada. Si el usuario menciona 3 → crear 3 entradas separadas.
- No pedir justificación si no la dio. El registro mínimo válido es solo la decisión.
- Si el usuario revierte una decisión anterior → agregar nueva entrada marcada como "Revisión de decisión anterior" y referenciar la fecha de la original.
- Archivos mensuales: `empresa/decisiones/2026-06.md`, `empresa/decisiones/2026-07.md`, etc.
