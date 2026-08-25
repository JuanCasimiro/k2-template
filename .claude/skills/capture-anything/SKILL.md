# Skill: capture-anything

**Trigger:** cuando el usuario comparte información que no es una pregunta ni un pedido explícito. Es el router de último recurso: si ningún otro trigger matcheó y el usuario está compartiendo algo, este skill lo captura y lo clasifica.

Ejemplos de lo que activa este skill:
- "Hoy me llamó Juan Pérez, es el dueño de Repuestera El Norte, tiene 3 sucursales"
- "Acabo de saber que nuestro precio está 15% arriba del mercado"
- "La reunión de ayer con el banco fue bien, nos aprueban el crédito"
- "El delivery falló tres veces esta semana"
- "Empezamos a usar Notion para el equipo"
- "Pablo renunció"
- "Firmamos con el proveedor de China"

Lo que NO activa este skill (porque tienen su propio skill):
- Transcripción/resumen de reunión → `reunion-a-insights`
- Decisión explícita ("decidí", "vamos a hacer") → `log-decision`
- Aprendizaje de proceso ("de ahora en más") → `k2-aprender`
- Archivo para procesar → `procesar-archivo`

---

## Propósito

Asegurarse de que ninguna información que el usuario comparte se pierda. Si el usuario lo dijo, queda guardado — sin fricciones, sin preguntas, sin tener que saber dónde va cada cosa.

---

## Protocolo

### Paso 1 — Clasificar el input

Determinar qué tipo de información es:

| Tipo | Cómo se detecta | Destino |
|------|----------------|---------|
| Persona nueva o update de persona | nombre + descripción / empresa / rol | `empresa/personas/[Nombre].md` |
| Evento o situación que pasó | algo que ocurrió recientemente | `empresa/inbox/YYYY-MM-DD.md` |
| Update de proyecto o área | mención de avance, problema, estado | `proyectos/[nombre]/estado.md` (si existe) o inbox |
| Dato del mercado / competencia | info sobre el sector o competidores | `empresa/inbox/YYYY-MM-DD.md` |
| Cambio en el equipo / estructura | alguien entró, salió, cambió de rol | `empresa/inbox/YYYY-MM-DD.md` |
| Otra información sin categoría clara | cualquier cosa que no encaje arriba | `empresa/inbox/YYYY-MM-DD.md` |

**Si hay duda entre dos categorías → usar inbox.** La claridad de categoría no es lo importante: que quede guardado es lo importante.

### Paso 2 — Extraer y estructurar

Antes de guardar, procesar la información:
- ¿Quién está involucrado?
- ¿Qué pasó o qué se sabe?
- ¿Tiene fecha asociada?
- ¿Implica algún próximo paso evidente?

### Paso 3 — Guardar sin confirmar primero

A diferencia de otras skills, **no pedir confirmación antes de guardar**. Guardar directamente y luego reportar.

La razón: el CEO que comparte info rápida (por voz, en movimiento) no quiere ser interrumpido con una pregunta. El objetivo es que el sistema capture y confirme en un solo paso.

**Excepción:** si el input es ambiguo y hay riesgo de guardar en el lugar incorrecto → preguntar solo si la confusión afecta significativamente dónde va a ir el dato.

### Paso 4 — Confirmar con un insight

Después de guardar, responder con:
1. Dónde se guardó (ruta)
2. Un insight procesado del input (no solo repetir lo que dijo)

Formato de confirmación:
```
Guardado en [ruta].

[Un insight o conexión que el CEO no dijo explícitamente — algo que el sistema vio al procesar el input]
```

Ejemplos:
- "Guardado en empresa/personas/Juan-Perez.md. → Repuestera con 3 sucursales = prospecto de alto potencial para [nombre del producto]. ¿Tenés una reunión con él?"
- "Guardado en empresa/inbox/2026-06-23.md. → Precio 15% arriba del mercado + [cliente X mencionado la semana pasada que dudaba por el precio] = ¿revisamos el pricing antes de la próxima propuesta?"
- "Guardado en empresa/inbox/2026-06-23.md. → Tres fallas de delivery esta semana. ¿Querés que armemos un ticket de seguimiento o se lo pasamos a [persona responsable]?"

Si no hay insight obvio → no inventar uno. Confirmar solo la ruta.

---

## Formato de entrada en inbox

```markdown
---

## [Hora] — [Título en una línea]

[Descripción de lo que pasó o se sabe]

**Quién:** [personas involucradas — si aplica]
**Próximo paso:** [si es evidente — si no, omitir]
```

## Formato de entrada en personas/[Nombre].md

Si el archivo ya existe → agregar sección de update:
```markdown
---

### Update [fecha]
[Lo nuevo que se sabe]
```

Si es persona nueva → crear el archivo:
```markdown
# [Nombre]

> Primera mención: [fecha]

**Empresa:** [si aplica]
**Rol:** [si se mencionó]
**Cómo llegó:** [cómo lo conoció el CEO]

## Lo que sabemos

[Descripción inicial]

## Historial de interacciones

_(se completa con el uso)_
```

---

## Reglas

- Nunca inventar información que no estuvo en el input
- Si el usuario dice algo que es claramente un error o test → no guardar, responder: "¿Esto era para guardar o fue un test?"
- No interrumpir el flujo del CEO con preguntas de clasificación — clasificar y guardar
- Si en el mismo mensaje hay múltiples tipos de info → guardar cada tipo donde corresponde (puede haber más de un archivo actualizado por mensaje)
