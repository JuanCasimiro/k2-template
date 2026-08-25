# Skill: k2-aprender

**Trigger:** cuando el usuario dice "acordate de esto", "guardá esto", "aprendizaje", o al final de una sesión larga cuando hubo algo relevante.

---

## Propósito

Capturar aprendizajes significativos en un chain log (`empresa/APRENDIZAJES.md`) y hacer un commit para que queden en el historial de git.

El repo es la memoria histórica de la empresa. Cada entrada en `APRENDIZAJES.md` + su commit es un punto en la línea de tiempo de cómo evolucionó el negocio.

---

## Qué vale la pena guardar

✅ Guardar:
- Decisión importante que cambia cómo se hace algo
- Corrección que el usuario hizo a una propuesta de Claude
- Insight de negocio que emergió en la conversación
- Patrón nuevo que aplica a este proyecto o área
- Algo que "no era obvio y ahora sí lo es"

❌ No guardar:
- Información operativa trivial (una fecha, un número sin contexto)
- Lo que ya está documentado en otro lugar
- Detalles que van a cambiar en días

---

## Protocolo

1. **Identificar** qué es el aprendizaje (el insight reutilizable, no el anécdota)
2. **Preguntar** si aplica a un proyecto específico o es general de empresa
3. **Escribir la entrada** en `empresa/APRENDIZAJES.md`
4. **Hacer commit** con el aprendizaje en el mensaje

---

## Formato de entrada

```markdown
## [YYYY-MM-DD] — [Área o proyecto]

**Qué pasó:** [breve descripción del contexto]  
**Aprendizaje:** [el insight reutilizable — qué cambió en cómo pensamos/hacemos esto]  
**Tags:** #decisión | #corrección | #proceso | #cliente | #proveedor | #producto
```

**Ejemplo:**
```markdown
## 2026-06-18 — Proceso de compras

**Qué pasó:** Pedimos a un proveedor sin confirmar stock disponible primero.  
**Aprendizaje:** Siempre confirmar stock del proveedor antes de comprometer fecha al cliente. Agregar ese paso al flujo de compras.  
**Tags:** #proceso #proveedor
```

---

## Guardar y confirmar

En Claude Code: al terminar la sesión, hacer commit con todos los archivos modificados.

**Cuándo guardar:**
- Al final de la sesión si se generó o modificó algo relevante
- Después de guardar un aprendizaje importante
- Después de completar una spec o planificación
- El usuario lo pide explícitamente

---

## Acceso al historial

Para revisar el historial de aprendizajes y decisiones:
- `git log --oneline` — línea de tiempo de la empresa
- `git show [hash]` — qué se generó en esa sesión
- Leer `empresa/APRENDIZAJES.md` — chain log estructurado
