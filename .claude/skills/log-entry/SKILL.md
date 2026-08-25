# Skill: log-entry

**Trigger:** el usuario dice "anotá que", "no quiero olvidar", "apuntá", "guardá esto rápido", "para que no se pierda", "quiero dejar registro de", o hay algo que surgió en la conversación que merece quedar capturado pero no es una reunión, aprendizaje formal ni decisión.

---

## Propósito

Captura rápida de cualquier cosa que el CEO no quiere perder: una observación, un número, un contacto, una idea, un obstáculo, una tarea que apareció. Sin fricción, sin categorización obligatoria.

El inbox es el buffer del sistema. Lo que empieza acá puede migrar después a un área más específica.

**Output:** entrada en `empresa/inbox/YYYY-MM-DD.md` del día actual.

---

## Distinción con otras skills

| log-entry | log-decision | k2-aprender | capture-anything |
|---|---|---|---|
| Captura explícita ("anotá") | Decisión con compromiso | Insight que cambia procesos | Captura implícita (info compartida) |
| Sin categoría obligatoria | Requiere "qué + por qué" | Alta barra de relevancia | Auto-detecta tipo y destino |
| `empresa/inbox/` | `empresa/decisiones/` | `empresa/APRENDIZAJES.md` | Donde corresponda |

---

## Protocolo

### Paso 1 — Capturar sin interrumpir

No pedir aclaraciones antes de guardar. Capturar primero, clarificar solo si la entrada es ambigua al punto de no poder escribirse.

---

### Paso 2 — Verificar si el archivo del día existe

Ruta: `empresa/inbox/YYYY-MM-DD.md` (fecha de hoy).

- **Existe:** hacer append al final
- **No existe:** crear con encabezado y la primera entrada

---

### Paso 3 — Escribir la entrada

**Formato básico:**

```markdown
- [HH:MM] [La entrada tal como la dijo el usuario, editada mínimamente para que sea legible]
```

**Formato con tipo (opcional, si es obvio):**

```markdown
- [HH:MM] 💡 Idea: [...]
- [HH:MM] ⚠️ Obstáculo: [...]
- [HH:MM] 📞 Contacto: [nombre] — [dato relevante]
- [HH:MM] 📌 Pendiente: [tarea]
- [HH:MM] 🔢 Dato: [número o métrica con contexto]
```

Usar emojis solo si el tipo es claro. Si no, entrada simple sin emoji.

---

### Paso 4 — Confirmar en una línea

> "Anotado."

Nada más. El inbox es captura rápida — no necesita confirmación elaborada.

Si el usuario quiere expandirlo → hacerlo en la misma respuesta antes de guardar.

---

### Paso 5 — Proponer migración (solo si aplica)

Si la entrada capturada claramente pertenece a otra área, al final decir:

> "¿Lo muevo a [reuniones / aprendizajes / decisiones] o lo dejamos en el inbox?"

Solo si es muy obvio. No proponer migración para entradas ambiguas.

---

## Encabezado del archivo diario (cuando no existe)

```markdown
# Inbox — [DD de mes de YYYY]

> Captura del día. Sin filtro — todo lo que no querés perder.
> Al final del día o la semana: revisarlo y migrar lo que valga.

---

```

---

## Reglas

- Velocidad sobre perfección — capturar en <5 segundos de procesamiento
- No corregir ni "mejorar" lo que dijo el usuario — capturar la voz real
- No pedir que categorice antes de guardar
- Nunca mostrar rutas técnicas al usuario
- Si en el morning-review aparece inbox con entradas sin procesar → mencionarlo: "Tenés [N] notas en el inbox de ayer sin revisar."
