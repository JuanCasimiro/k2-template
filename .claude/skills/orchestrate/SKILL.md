# Skill: orchestrate

**Trigger:** cuando hay una tarea grande que involucra múltiples pasos, áreas, o que Claude no puede resolver de una sola vez bien.

---

## Propósito

Dividir tareas complejas en partes manejables. Claude coordina el proceso pero delega cada parte al momento correcto.

---

## Cuándo usar esta skill

- La tarea toca más de 2 áreas de la empresa
- El resultado final depende de información que hay que recopilar primero
- Hay pasos que deben hacerse en orden específico
- La tarea es tan amplia que respondida de una vez sería genérica e inútil

---

## Protocolo

### Paso 1 — Mapear la tarea

Antes de ejecutar, identificar:
- ¿Cuáles son las partes? ¿En qué orden van?
- ¿Qué información necesitamos antes de empezar?
- ¿Qué depende de qué?

### Paso 2 — Recopilar contexto

Leer los archivos relevantes de las áreas involucradas antes de proponer cualquier cosa. No adivinar.

### Paso 3 — Ejecutar por partes

Completar cada parte con calidad antes de pasar a la siguiente. Reportar el avance al usuario.

### Paso 4 — Integrar y entregar

Cuando todas las partes están listas, integrar en un resultado coherente.

---

## Ejemplo

**Tarea:** "Preparame el cierre de mes"

**Sin orchestrate:** Claude improvisa un documento genérico sin saber qué datos tiene la empresa.

**Con orchestrate:**
1. Leer `proyectos/finanzas/docs/PRODUCT_MEMORY.md` para entender qué existe
2. Preguntar: ¿tenés los números de ventas del mes?
3. Una vez que hay contexto, armar el resumen real
4. Entregar con un próximo paso claro

---

## Regla

Si la respuesta correcta requiere información que Claude no tiene en los archivos del repo, preguntarla primero. No inventar datos. No adivinar.
