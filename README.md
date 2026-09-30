# Tu sistema de trabajo con IA

Un punto de partida para que Claude **conozca tu negocio** y trabaje con vos todos los días — en vez de arrancar de cero en cada conversación.

No hace falta saber programar.

---

## Qué es

Una carpeta con dos cosas:

- **Estructura** — dónde se guarda cada cosa (reuniones, decisiones, aprendizajes, proyectos).
- **Skills** — instrucciones que Claude aplica solo cuando detecta que hacen falta. No hay comandos que memorizar.

La diferencia con usar un chat suelto: acá **queda escrito**. La próxima sesión Claude ya sabe lo que hablaron la anterior.

---

## Cómo arrancar

### 1. Instalá Claude Code
Descargalo desde **[claude.ai/download](https://claude.ai/download)** e instalalo.

### 2. Abrí una terminal y escribí `claude`

Si nunca abriste una terminal: en Windows buscá "Terminal" en el menú inicio; en Mac buscá "Terminal" en Spotlight.

### 3. Pegá este prompt

```
Quiero instalar mi sistema de trabajo con IA.
Cloná este repo en mi PC: https://github.com/JuanCasimiro/k2-template
Si no estoy logueado en GitHub, explicame cómo hacerlo antes de continuar.
Cuando esté todo listo, arrancá el setup.
```

Claude se encarga del resto — te loguea, lo baja, lo abre y arranca la configuración. Si algo no entendés, preguntale ahí mismo.

### 4. Contestá el onboarding

Te hace preguntas sobre tu empresa durante unos 20 minutos. Con eso queda configurado para vos.

### 5. Leé `EMPEZA-ACA.md`

Las primeras cosas que le podés pedir hoy.

---

## Desde el teléfono

Activá **Remote Control** en Claude Code. La sesión sigue corriendo en tu computadora y la manejás desde el celular — mismo sistema, mismos archivos.

---

## Qué hay adentro

```
CLAUDE.md          ← el cerebro: quién sos, cómo trabajás (se completa solo)
EMPEZA-ACA.md      ← qué pedirle el primer día
empresa/           ← contexto, aprendizajes, preferencias, reuniones, reportes
proyectos/         ← una carpeta por área de tu negocio
.claude/skills/    ← las skills que Claude aplica solo
```

---

## Si programás: pack de ingeniería

Además de las skills de negocio, el template trae 12 skills para construir software con disciplina. Se activan solas cuando la tarea toca código:

| Frente | Skills |
|--------|--------|
| Definir antes de codear | `sdd` · `historias-de-usuario` · `ddd` · `tdd` |
| Base de datos | `migraciones-db` · `rls-multi-tenant` · `queries-seguras` · `backfill-seguro` |
| Integridad del dominio | `ciclo-de-vida-entidades` · `integridad-ledger` · `reuso-codigo-limpio` |
| Salida a producción | `pre-ship` · `auditoria-cto` |

Nacieron de operar un SaaS multi-tenant en producción: cada regla existe porque algo se rompió sin ella. Stack de referencia Postgres/Supabase + TypeScript; los principios aplican a cualquier stack. Para Supabase sumá las oficiales: `npx skills add supabase/agent-skills`.

---

## ¿Ya tenés información en otro lado?

Pegale esto:

```
Tengo información en [Obsidian / Documentos / Notion / etc.].
Tomá lo que sirva y pasalo al sistema, sin tocar esas carpetas.
```

---

## Licencia

MIT — usalo, copialo y adaptalo libremente. Ver [`LICENSE`](LICENSE).
