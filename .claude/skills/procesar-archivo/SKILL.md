# Skill: procesar-archivo

**Trigger:** el usuario sube un archivo, foto, PDF o documento — o dice "tengo esto", "mirá lo que me mandaron", "procesame este archivo", "tengo una foto del pizarrón".

---

## Propósito

Que nada de lo que el CEO comparte se pierda. Extraer lo importante de cualquier archivo y guardarlo al vault en el lugar correcto.

---

## Protocolo

### Paso 1 — Leer el archivo

Procesar lo que el CEO compartió: PDF, imagen, Excel, texto, audio transcripto, captura de pantalla.

Si el contenido no es legible o está en un formato que no podés procesar → decirlo sin rodeos: "No puedo leer este formato. ¿Me lo pasás como texto o imagen?"

### Paso 2 — Identificar qué es

Determinar la naturaleza del contenido:
- **Reunión o conversación** → usar `reunion-a-insights` para el análisis
- **Documento de cliente o propuesta** → extraer datos clave del cliente
- **Contrato o acuerdo** → extraer partes, condiciones principales, fechas clave
- **Información de sector o competencia** → extraer insights relevantes para la empresa
- **Foto de pizarrón o notas** → transcribir y estructurar
- **Otro** → resumir en bullets y preguntar dónde guardarlo

### Paso 3 — Extraer y mostrar

Mostrar un resumen estructurado de lo importante. No reproducir el documento completo — solo lo accionable:
- Qué es
- Puntos clave (máximo 5 bullets)
- Próximos pasos o fechas relevantes (si hay)
- Quién está involucrado (si aplica)

### Paso 4 — Confirmar y guardar

Preguntar: "¿Lo guardamos?" (sí/no — no dar opciones complejas).

Si sí → determinar la carpeta correcta según el tipo:
- Reunión/cliente: `empresa/reuniones/YYYY-MM-DD-[nombre].md`
- Documento general: `empresa/documentos/YYYY-MM-DD-[descripcion].md`
- Aprendizaje o insight: agregar a `empresa/APRENDIZAJES.md`
- Info de proyecto específico: `proyectos/[proyecto]/docs/[descripcion].md`

Crear la carpeta si no existe. Guardar. Confirmar con la ruta.

---

## Reglas

- No inventar información que no está en el archivo
- Si el archivo tiene información sensible (contratos, datos personales): guardarlo igual — el vault es privado
- Siempre preguntar antes de guardar — nunca guardar automáticamente sin confirmación
- Un archivo a la vez — si el CEO sube varios, procesar uno por uno
