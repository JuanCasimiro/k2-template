# Skill: importar-historial

**Trigger:** el usuario pega un bloque de texto con información estructurada de su empresa (output del prompt extractor), o dice "te mando el resumen que me dio Claude/ChatGPT".

---

## Propósito

Recibir el contexto pre-formateado de otro sistema de IA y escribirlo al vault. Luego arrancar con las 3 preguntas emocionales del onboarding para completar lo que los datos no pueden dar.

---

## Protocolo

### Paso 1 — Detectar si tienen data o necesitan extraerla

Si el usuario **ya pegó** un bloque de texto con información de su empresa → ir directo al Paso 2.

Si el usuario **solo mencionó** que usa otra IA pero no trajo nada → generar el prompt extractor ahí mismo:

> "Perfecto. Copiá este texto y pegalo en tu Claude/ChatGPT — lo que te responda, traémelo acá:"

Mostrar este bloque exacto para que copie:

---
```
Necesito que analices toda nuestra conversación y el contexto que tenés de mí y mi empresa, y lo organices en este formato. Donde no tenés información, escribí "A completar".

## MI EMPRESA
**Nombre:** 
**Qué hace en una frase:** 
**Industria / sector:** 
**Etapa:** (arranque / crecimiento / consolidada)
**Equipo:** 

## CÓMO GENERO DINERO HOY
**Producto o servicio principal:** 
**Cómo llegan los clientes:** 
**Frentes activos que generan ingresos hoy:** 
**Frentes que son apuesta futura:** 

## MIS PROCESOS PRINCIPALES
1. 
2. 
3. 

## MIS CLIENTES Y PERSONAS CLAVE
**Perfil de cliente típico:** 
**Clientes actuales que mencioné:** 
**Personas clave:** 

## MIS DOLORES Y CUELLOS DE BOTELLA
1. 
2. 
3. 

## DECISIONES IMPORTANTES QUE TOMÉ
- 
- 

## HERRAMIENTAS QUE USO
- 

## CÓMO SOY YO
**Cómo trabajo mejor:** 
**Qué me saca del foco:** 
**Mi auto-saboteo más común:** 

## LO QUE QUIERO CONSTRUIR
```
---

Cerrar con: *"Cuando tengas el resultado, pegalo acá y arrancamos."*

Esperar. No continuar hasta que pegue el output.

### Paso 2 — Recibir y validar

Leer el bloque recibido. Verificar que tiene estructura reconocible.

Si el formato es muy libre o incompleto → procesarlo igual, extraer lo que haya, marcar los gaps.

### Paso 2 — Escribir al vault

Generar y guardar:

- `empresa/CONTEXTO.md` — con todo lo extraído del historial, usando la estructura estándar
- `empresa/APRENDIZAJES.md` — con las decisiones y aprendizajes que mencionó
- Actualizar `CLAUDE.md` PARTE 1 y PARTE 2 con los datos reales

Confirmar: *"Listo, tu contexto quedó cargado."* Sin mostrar rutas ni tecnicismos.

### Paso 3 — Arrancar el onboarding emocional

Inmediatamente después de guardar, decir:

> "Ahora te hago 3 preguntas. No tienen respuesta correcta — decí lo primero que te salga."

**Pregunta 1:**
> "¿Qué te hace feliz en tu trabajo? ¿Lo estás viviendo?"

**Pregunta 2:**
> "¿Por qué no lo sos? ¿Qué ponés en el medio?"

Dejar que hable. Luego:
> "¿Y para qué sostenés eso?"

**Pregunta 3:**
> "¿Cómo creés que se podría solucionar?"

### Paso 4 — Resolver uno en el acto

Con la respuesta de las 3 preguntas, identificar el bloqueo de mayor peso emocional y ejecutar la solución ahora mismo usando el contexto que ya está en el vault.

### Paso 5 — Completar gaps

Preguntar solo lo que faltó en el historial importado. Máximo 3 preguntas puntuales.

### Paso 6 — Cerrar

Hacer commit final:
```
feat: vault inicializado desde historial + onboarding emocional — [empresa]
```

---

## Reglas

- No inventar lo que no estaba en el historial
- Los gaps se completan con preguntas, no con suposiciones
- Las 3 preguntas emocionales siempre se hacen aunque el historial esté completo — los datos no reemplazan el momento de reconocimiento
- Al terminar: cerrar con "¿Por dónde arrancamos?"
