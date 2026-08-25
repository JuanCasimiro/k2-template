# Skill: sdd (Spec-Driven Development)

**Trigger:** cuando el usuario quiere encarar algo nuevo importante — un proceso, un proyecto, una mejora significativa — antes de ejecutar.

---

## Propósito

Antes de hacer algo complejo, escribir primero una spec clara. La spec es el contrato: define qué se hace, cómo se sabe que está listo, y qué queda fuera.

**Regla:** si no está en la spec, no existe.

---

## Cuándo usar esta skill

- Nuevo proceso que involucra a más de una persona
- Mejora que afecta algo que ya funciona
- Decisión importante con varias alternativas posibles
- Algo que si sale mal es costoso de corregir

---

## Los tres pasos del SDD

### Paso 1 — Spec (qué y por qué)

Responder estas preguntas:
- **Problema:** ¿Qué no funciona bien hoy? ¿Qué duele?
- **Objetivo:** ¿Qué queremos lograr? ¿Cómo se ve el éxito?
- **Usuarios/Roles:** ¿Quién lo usa? ¿Qué necesita cada uno?
- **Flujo principal:** ¿Cómo es el camino ideal de punta a punta?
- **Casos edge:** ¿Qué pasa cuando algo sale mal?
- **Fuera de scope:** ¿Qué explícitamente NO se va a resolver acá?

### Paso 2 — Design (cómo)

- ¿Qué recursos necesitamos? (personas, herramientas, información)
- ¿Hay algo que ya existe y podemos reutilizar?
- ¿Cuáles son los riesgos principales?
- ¿Cuál es el orden de los pasos?

### Paso 3 — Tasks (plan de ejecución)

Lista de tareas concretas, ordenadas, con criterio de "terminado" para cada una.

---

## Formato de spec

```markdown
# Spec: [nombre del proceso / mejora]

## Problema
[qué no funciona bien hoy]

## Objetivo
[qué queremos lograr — medible]

## Quién lo usa
| Rol | Qué necesita |
|-----|-------------|
| [rol] | [necesidad] |

## Flujo principal
1. [paso 1]
2. [paso 2]
3. [...]

## Casos edge
- Si pasa X → hacer Y
- Si falta Z → [qué hacer]

## Fuera de scope
- [qué NO se hace acá]

## Plan de ejecución
| # | Tarea | Listo cuando |
|---|-------|-------------|
| 1 | [tarea] | [criterio] |
```

---

## Cuándo la spec es suficiente para empezar

La spec está lista cuando:
- Alguien que no participó en la conversación puede leerla y entender qué hacer
- El criterio de "terminado" es claro y verificable
- Está escrito qué NO se hace
