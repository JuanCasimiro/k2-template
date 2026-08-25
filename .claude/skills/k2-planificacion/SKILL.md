# Skill: k2-planificacion

**Trigger:** cuando el usuario quiere planificar un proyecto, definir prioridades, decidir qué hacer primero, escribir una spec, o pensar el próximo paso de un área.

---

## Propósito

Ayudar a planificar proyectos y features de forma que queden como **contratos de implementación**, no ideas vagas.

Un plan bien escrito permite que cualquier persona (o Claude mismo en otra sesión) lo ejecute sin preguntar qué se quiso decir.

---

## Cuándo usar esta skill

- "¿Qué debería hacer primero en [área]?"
- "Necesito planificar [proyecto]"
- "¿Cómo encaramos [proceso nuevo]?"
- "Escribime el plan para [cosa]"
- "¿Qué falta en [área]?"

---

## Regla previa obligatoria — leer antes de planificar

Antes de agregar algo al plan de un área, leer lo que ya existe:
- `proyectos/[area]/docs/PRODUCT_MEMORY.md` — qué está hecho, qué está en curso
- Cualquier archivo de esa carpeta que describa el estado actual

**Nunca planificar lo que ya está hecho. Separar con certeza "ya existe" de "falta".**

---

## Cómo escribir un plan que funciona

Cada ítem del plan debe incluir:

1. **Qué hace** — en una frase concreta (verbo + resultado)
2. **Por qué importa** — qué problema resuelve o qué mejora
3. **Cómo hacerlo** — pasos concretos
4. **Quién lo hace** — si hay más de una persona
5. **Cuándo está listo** — criterio de "terminado" (no vago: "cuando funcione")
6. **Qué NO hacer** — el scope negativo evita desvíos

---

## Formato del plan

```markdown
## [Nombre del proyecto / área]

### Qué se quiere lograr
[descripción en 1-2 oraciones]

### Estado actual
[qué existe hoy — con certeza verificada]

### Plan

| # | Qué hacer | Por qué | Cómo | Listo cuando |
|---|-----------|---------|------|-------------|
| 1 | [acción] | [razón] | [pasos] | [criterio] |

### Fuera de scope
- [qué NO se va a hacer en esta iteración]

### Próximo paso inmediato
[la primera acción concreta para arrancar]
```

---

## Grados de certeza

Al describir el estado actual, indicar:
- **Alta:** verificado hoy, existe de hecho
- **Media:** basado en un documento que puede estar desactualizado
- **Baja:** inferencia o memoria — verificar antes de actuar

---

## Output esperado

- El plan escrito en `proyectos/[area]/docs/PRODUCT_MEMORY.md` (sección "Plan")
- Commit con el plan: `docs: plan [área] — [fecha]`
- Próximo paso claro para que el usuario o Claude puedan ejecutar en la siguiente sesión
