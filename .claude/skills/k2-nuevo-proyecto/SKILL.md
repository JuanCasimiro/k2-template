# Skill: k2-nuevo-proyecto

**Trigger:** cuando el usuario quiere crear una nueva área en su sistema ("quiero agregar un proyecto de ventas", "armame una sección para proveedores", "necesito una carpeta para el equipo").

---

## Propósito

Crear la estructura de un nuevo proyecto (área de la empresa) en el sistema.

---

## Protocolo

### Paso 1 — Entender el área

Preguntar:
- ¿Qué área de la empresa es? (ventas, operaciones, finanzas, etc.)
- ¿Cuál es el principal dolor o necesidad que cubre?
- ¿Hay alguien específico responsable de esta área?

### Paso 2 — Crear la estructura

```
proyectos/[slug]/
├── COPILOT.md          # Descripción del área + rol de Claude en ella
└── docs/
    └── PRODUCT_MEMORY.md   # Estado actual + plan
```

**Naming del slug:** kebab-case, descriptivo, sin tildes.
Ejemplos: `ventas`, `operaciones`, `finanzas`, `equipo`, `proveedores`, `clientes`

### Paso 3 — Completar COPILOT.md

```markdown
# [Nombre del área]

## Qué cubre esta área

[descripción breve — qué pasa acá, quién la maneja]

## Rol de Claude en esta área

[cómo puede ayudar Claude específicamente — redactar, planificar, analizar, recordar]

## Links relacionados

- [otros proyectos o archivos relacionados]
```

### Paso 4 — Completar PRODUCT_MEMORY.md inicial

```markdown
# [Nombre del área] — Estado actual

> Creado: [fecha]

## Qué existe hoy

[descripción honesta del estado actual — qué hay, cómo se hace hoy]

## Plan

[primeras cosas a mejorar u organizar — ver /k2-planificacion]

## Aprendizajes

_(se va completando con el uso)_
```

### Paso 5 — Confirmar

Confirmá al usuario: "Proyecto [nombre] creado. Lo encontrás en proyectos/[nombre]/."

---

## Proyectos típicos por tipo de empresa

| Empresa | Proyectos comunes |
|---------|-------------------|
| Comercio | ventas, compras, inventario, clientes, proveedores |
| Servicios | proyectos-clientes, propuestas, facturacion, equipo |
| Clínica | pacientes, turnos, proveedores, administracion |
| Constructora | obras, proveedores, presupuestos, equipo, finanzas |
| Agencia | clientes, campanas, proveedores, finanzas |
