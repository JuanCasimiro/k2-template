# Skill: analizar-cliente

**Trigger:** cuando el usuario quiere entender mejor a un cliente o prospecto, dice "¿cómo encarar a [nombre]?", "¿qué le ofrezco a [cliente]?", "analizame a este cliente", "¿qué necesita [nombre]?", "tuve una reunión con un prospecto y no sé por dónde ir", o cuando quiere saber cómo hacer el seguimiento de alguien.

---

> **Razonamiento profundo requerido.** Antes de generar el perfil, procesá toda la información disponible sin apresurarte. Un análisis superficial puede costarle una venta al CEO.

## Propósito

A partir de lo que el usuario comparte sobre un cliente o prospecto — transcripción, descripción, historial — generar un perfil accionable: quién es, qué necesita, desde dónde habla, qué ofrecerle y cómo encarar el seguimiento. El CEO sale con un plan concreto de qué hacer con esa persona.

---

## Cuándo ejecutar

- El usuario quiere entender a un cliente o prospecto específico
- El usuario pregunta cómo encarar a alguien o qué ofrecerle
- El usuario quiere hacer seguimiento y no sabe por dónde empezar
- Se puede combinar con `/reunion-a-insights` para profundizar el perfil del participante de una reunión

---

## Protocolo

### Paso 1 — Identificar de quién se habla

Si hay varios clientes mencionados:
→ Preguntar: "¿De cuál de ellos querés el análisis?"

### Paso 2 — Verificar contexto de la empresa

Leer `empresa/CONTEXTO.md` para entender qué servicios o productos ofrece el CEO.

Si no existe o no tiene información de servicios:
→ Preguntar: "Para recomendarte qué ofrecerle, necesito saber qué productos o servicios tenés. ¿Me contás brevemente?"

### Paso 3 — Reunir información del cliente

Si hay reuniones previas con esa persona en `empresa/reuniones/`: leerlas para tener historial.

Si la información disponible es muy escasa (menos de 3 datos concretos):
→ Procesar lo que hay y avisar: "Trabajé con poca información. Si compartís más contexto puedo mejorar el análisis."

### Paso 4 — Generar el perfil

**Quién es:**
- Rol o posición (si se sabe)
- Tipo de empresa o contexto (si aplica)
- Cómo llegó o por qué están hablando

**Desde dónde habla:**
- Estado emocional predominante: ¿miedo, urgencia, ambición, escepticismo, confianza?
- Cómo procesa la información: ¿quiere datos, quiere historia, quiere garantías?
- Qué lo mueve a tomar decisiones

**Dolores identificados:**
- Explícitos: lo que dijo directamente
- Implícitos: lo que se puede inferir de cómo habló y qué preguntó

**Qué ofrecerle:**
- De los servicios o productos del CEO, cuáles son más relevantes para este cliente
- En qué orden presentarlos y por qué
- Qué no mencionar si puede generar resistencia

**Cómo encarar el seguimiento:**
- Tono recomendado (formal, informal, técnico, emocional)
- Timing (cuándo contactar, cada cuánto)
- Argumento central: la frase clave para conectar con su dolor
- Canal sugerido

**Red flags:**
- Señales de que no está listo para decidir
- Objeciones probables
- Situaciones que podrían hacer caer el negocio

### Paso 5 — Confirmar el diagnóstico

Antes de cerrar, mostrar los dolores identificados y preguntar:
"¿Te parece que capturo bien lo que necesita [nombre], o hay algo que no vi?"

Si el CEO corrige → ajustar y mostrar el perfil actualizado.

### Paso 6 — Guardar (opcional)

Preguntar: "¿Querés que guarde este perfil para tenerlo como referencia?"

Si sí → guardar en `empresa/reuniones/perfil-[nombre].md`

---

## Reglas

- Nunca inventar dolores o necesidades que no estén en la información provista
- Si no hay suficiente información → decirlo explícitamente, no rellenar con suposiciones
- El análisis es una hipótesis de trabajo, no un diagnóstico cerrado — el CEO puede corregirlo
- Priorizar siempre los servicios que el CEO ya ofrece, no inventar nuevos
- Si hay historial de reuniones con esa persona: usarlo para detectar patrones y evolución
