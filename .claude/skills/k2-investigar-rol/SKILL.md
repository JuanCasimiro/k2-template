# Skill: k2-investigar-rol

**Trigger:** después de completar `/k2-onboarding`, o cuando el usuario pide "buscá mejores prácticas", "investigá cómo hacen X en mi sector", "hay alguna skill específica para mi industria".

---

> **Razonamiento profundo requerido.** Antes de responder, tomá el tiempo necesario para analizar bien. Priorizá calidad sobre velocidad — una investigación superficial es peor que no investigar.

## Propósito

Para cada proceso clave de la empresa, determinar si el cliente ya tiene una metodología propia o necesita una externa — y actuar distinto en cada caso.

- **Tiene metodología propia** → entenderla y documentarla como skill (no tocar internet)
- **No tiene metodología** → buscar mejores prácticas del sector y construir una

---

## Protocolo

### Paso 1 — Leer el contexto

Leer `empresa/CONTEXTO.md` para identificar:
- Industria / sector
- Procesos principales
- Dolores identificados
- Herramientas actuales

---

### Paso 2 — Encuestar al cliente por proceso

Para cada proceso clave identificado, preguntar al cliente:

> "Para [proceso], ¿ya tenés una forma de hacerlo que funciona? ¿O es algo donde sentís que falta método?"

Registrar la respuesta en una de dos categorías:

| Categoría | Señales |
|-----------|---------|
| **Experto / metodología propia** | "Sí, lo hago así…", "Tengo mi forma", "Soy bueno en esto", lo explica con detalle |
| **Sin metodología / necesita framework** | "Lo hago medio al voleo", "No sé bien cómo", "Ahí me pierdo", "Buscaría ayuda" |

No asumir. Si no queda claro, preguntar: *"¿Me podés explicar cómo lo hacés hoy, paso a paso?"* — si puede, es experto. Si no puede, necesita framework.

---

### Paso 3A — Proceso donde el cliente ES experto

**No buscar en internet.** El conocimiento tácito del cliente vale más que cualquier blog.

**Si el cliente tiene material existente** (transcripciones de cursos, documentos, PDFs, grabaciones):
1. Leer ese material primero — es la fuente más fiel de su metodología
2. Extraer los pasos, criterios y ejemplos directamente del material
3. Complementar con preguntas solo donde el material no sea suficiente
4. Documentar la skill y mostrarle el draft: *"¿Esto refleja cómo lo hacés?"*

**Si no tiene material, entrevistar:**
1. Pedirle que explique su proceso paso a paso
2. Hacer preguntas hasta entender el criterio detrás de cada paso (*"¿Por qué hacés X antes que Y?"*)
3. Documentar su metodología como skill
4. Mostrarle el draft y pedirle validación: *"¿Esto refleja cómo lo hacés?"*

**El objetivo:** que Claude pueda replicar su metodología, no reemplazarla.

---

### Paso 3B — Proceso donde el cliente NO tiene metodología

1. Formular 3-5 búsquedas específicas basadas en el sector y el dolor:
   - `[industria] mejores prácticas [proceso]`
   - `cómo hacer [proceso] en [sector] ejemplos concretos`
   - `[dolor específico] framework solución`
2. Buscar con WebSearch, leer fuentes relevantes con WebFetch
3. Filtrar: ¿aplicable a esta empresa? ¿concreto? ¿tiene casos reales? Descartar contenido genérico o vendido
4. Adaptar lo encontrado al contexto real de la empresa (tamaño, herramientas, recursos)
5. Documentar como skill en `.claude/skills/`
6. Mostrarle el draft al cliente y ajustar según su feedback

---

### Paso 4 — Formato de cada skill generada

**Naming:** `[dominio]-[proceso]/SKILL.md`

**Ejemplos:**
- `guiones-marca-personal/SKILL.md` ← metodología propia del cliente
- `clinica-gestion-turnos/SKILL.md` ← basada en investigación
- `contabilidad-cierre-mensual/SKILL.md`

**Formato:**

```markdown
# Skill: [nombre]

**Origen:** [metodología propia del cliente | investigación sectorial]
**Contexto:** [para qué tipo de empresa aplica]
**Basado en:** [el cliente / fuentes consultadas]

## Qué hace esta skill

[descripción breve]

## Cuándo usarla

[triggers]

## Protocolo

[pasos concretos — si es metodología propia, refleja exactamente cómo lo hace el cliente]

## Ejemplos

[casos reales o ejemplos del cliente / sector]
```

---

### Paso 5 — Agregar triggers al CLAUDE.md

Por cada skill creada, agregar una fila a la tabla de triggers en `CLAUDE.md` (sección "Triggers automáticos"):

| Si el usuario... | Aplicar |
|-----------------|---------|
| [intención que dispara la skill] | `[nombre-skill]` |

Esto es obligatorio si el cliente usa el sistema desde **Claude.ai Chat o Cowork** — sin el trigger en CLAUDE.md, el agente no va a aplicar la skill automáticamente.

---

### Paso 7 — Registrar en APRENDIZAJES.md

```markdown
## [fecha] — Investigación de rol ([industria])

**Qué pasó:** Mapeo de procesos post-onboarding.
**Skills de metodología propia:** [lista]
**Skills de investigación externa:** [lista]
**Insight clave:** [algo que aprendiste del cliente que no estaba en el CONTEXTO.md]
**Tags:** #investigación #sector #setup
```

### Paso 8 — Commit y confirmación

Confirmá al usuario: "Skills de rol guardadas en `.claude/skills/`. El sistema ya las va a usar la próxima vez."

```bash
git add .
git commit -m "feat: skills de rol — [sector]

Skills de metodología propia: [lista]
Skills de investigación externa: [lista]
"
```

---

## Criterio de calidad para skills generadas

Una skill vale la pena si:
- Es **accionable** (el usuario puede ejecutarla hoy)
- Es **fiel** (si es metodología propia, refleja exactamente cómo piensa el cliente)
- **Ahorra tiempo** en algo que hacen regularmente
- **Claude puede ayudar** con ella de verdad

No crear skills que:
- Impongan una forma externa sobre alguien que ya tiene la suya
- Dupliquen lo que ya tienen
- Sean demasiado genéricas

---

## Ejemplos por sector

| Sector | Metodología propia (aprender) | Sin metodología (investigar) |
|--------|------------------------------|------------------------------|
| Creador de contenido | guiones, edición, estilo de marca | distribución, métricas, monetización |
| Estudio contable | criterio fiscal del contador | herramientas de automatización |
| Comercio retail | proceso de compra al proveedor | gestión de stock, pricing |
| Construcción | criterio de presupuesto del maestro | gestión de obra, proveedores |
| Agencia marketing | proceso creativo del equipo | reporting, herramientas nuevas |
