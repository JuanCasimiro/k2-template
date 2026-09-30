---
name: reuso-codigo-limpio
description: Método para reusar antes de crear y mantener código limpio — buscar antes de escribir, una fuente de verdad por regla, cuándo extraer, cuándo NO abstraer, tamaño de funciones y archivos, nombres del dominio. Usala al agregar o modificar cualquier componente, servicio, hook o función, o al ver la misma lógica en 2+ lugares. Triggers on "agregar", "crear nuevo", "nuevo componente/servicio/hook", "duplicado", "copiar y pegar", "refactor", "mismo X en Y", "código limpio", "DRY".
---

# Reuso + código limpio

## Primer paso, siempre: buscar antes de crear

Antes de escribir algo nuevo, grep por nombres, verbos y conceptos parecidos (`grep` del dominio, del término en inglés y en español, de la query o del texto de la UI). Mirá los lugares donde vive lo compartido: servicios, hooks, utils, componentes comunes. Si existe algo parecido: **extendelo**. Si no, creá y dejá la pieza en el lugar donde el próximo la va a buscar.

Mantené en el repo un mapa corto "necesitás X → reusá Y" (cliente de DB, manejo de errores, diálogos de confirmación, normalización de códigos, permisos). Las reglas que ya costaron un bug (ej. normalizar códigos, nunca `window.confirm`) van ahí.

## Una fuente de verdad por regla

Cada regla de negocio, cálculo o texto vive en **un** lugar. Si cambia, se cambia una vez.

- Regla de negocio → capa de servicios/dominio, no dentro del componente.
- Estado/efectos compartidos por dos pantallas → un hook.
- Markup repetido → un componente.
- Constantes y enums (estados, roles, tipos) → un módulo; nunca strings sueltos repetidos.

**Caso tipo:** la edición de cliente estaba copiada en dos modales; al corregir un bug en uno, el otro seguía roto. Se extrajo un panel único consumido por ambos: una fuente de verdad, siempre iguales.

## Cuándo extraer

- **1ª vez:** escribila donde está.
- **2ª copia:** extraé ahora. Es el momento barato: ya viste dos casos reales y sabés qué varía.
- Regla práctica: si aparece lógica, markup o query equivalente en 2+ archivos → unificá antes de mergear.
- Si lo que cambia entre copias es poco, parametrizá; si cambia casi todo, no eran la misma cosa.

## Cuándo NO abstraer

- **Parecido por casualidad:** dos bloques que hoy se ven iguales pero cambian por razones distintas (reglas de dos áreas del negocio). Unirlos los ata.
- **Abstracción especulativa:** "por si después lo necesitamos". Sin segundo caso real, no hay abstracción.
- **Flags en cascada:** si la función compartida acumula `if (modo === ...)`, `opciones`, booleanos, volvé a dos funciones y compartí solo el núcleo común.
- **Indirección sin ganancia:** un wrapper que solo renombra otra función.
- Duplicar 3 líneas es más barato que una abstracción equivocada. Borrarla después no lo es.

## Tamaño y forma

| Unidad | Señal de alarma | Qué hacer |
|--------|----------------|-----------|
| Función | > ~40 líneas o hace "y además…" | partir por responsabilidad |
| Archivo | > ~300 líneas o mezcla capas | partir por concepto, no por tipo |
| Componente | mezcla fetch + reglas + markup | servicio + hook + presentación |
| Parámetros | > 4 | pasá un objeto con nombres |

- Una función, un nivel de abstracción; retornos tempranos en lugar de anidar.
- La UI no habla con la base directo si hay (o puede haber) servicio: negocio en servicios, estado en hooks, presentación en componentes.
- **Consultas en lote:** nunca un `await` por fila en un loop (N+1); usá upsert/`in(...)`/RPC con array (ver `queries-seguras`).

## Nombres del dominio

Usá el lenguaje del negocio (el de la spec y las historias), el mismo en código, tests, UI y conversación: `liberarReserva`, no `updateStatus2`. Un concepto, un nombre; si el negocio dice "pedido", no alternar con "orden" y "request". Evitá `data`, `info`, `manager`, `helper`, `utils2`. Ver `ddd` (lenguaje ubicuo) y `sdd`.

## Gate de revisión

```
□ ¿Busqué si ya existía algo parecido antes de crearlo?
□ ¿Hay lógica/markup/query equivalente en dos archivos? → bloquear hasta unificar
□ ¿Cada regla tiene una sola fuente de verdad?
□ ¿No abstraje sin segundo caso real? ¿No hay flags en cascada?
□ ¿Funciones y archivos de tamaño razonable, capas separadas?
□ ¿Los nombres son los del dominio?
□ ¿Sin loops de await por fila?
```

Relacionadas: `tdd` (el refactor va con tests en verde), `integridad-ledger` (un solo writer por flujo es esta misma regla), `pre-ship`, `auditoria-cto`.
