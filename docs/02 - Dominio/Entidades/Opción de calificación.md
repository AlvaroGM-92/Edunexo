---
tipo: entidad
estado: activo
fase: 1
actualizado: 2026-11-10
tags:
  - dominio
  - entidad
  - calificaciones
  - multi-tenant
---

# Opción de calificación

## Propósito

Valor cerrado de una asignatura con escala alfanumérica, por ejemplo «Insuficiente», «Suficiente», «Notable» o «Apto». La administración los define al crear o editar la asignatura y el profesorado los elige en el cuaderno de notas.

## Campos

| Campo | Tipo | Obligatorio | Reglas |
| --- | --- | --- | --- |
| `id` | BIGINT | Sí | Clave primaria. |
| `mancomunidad_id` | BIGINT | Sí | FK a [[Mancomunidad]]; tenant de la opción. |
| `asignatura_id` | BIGINT | Sí | FK a [[Asignatura]] de la misma mancomunidad y con escala `ALFANUMERICA`. |
| `etiqueta` | VARCHAR(40) | Sí | Única dentro de la asignatura; ej. «Notable». |
| `orden` | SMALLINT | Sí | Único dentro de la asignatura; define el orden del desplegable. |
| `aprobado` | BOOLEAN | Sí | Indica si la opción cuenta como superada. |
| `activo` | BOOLEAN | Sí | Por defecto `true`; una opción inactiva no se ofrece para nuevas notas. |
| `created_at` | DATETIME | Sí | Auditoría. |
| `updated_at` | DATETIME | Sí | Auditoría. |

## Relaciones

- **N:1 con [[Mancomunidad]]**.
- **N:1 con [[Asignatura]]**.
- **1:N con [[Calificación]]**.

## Reglas

- Solo existen opciones en asignaturas con `escala_calificacion = ALFANUMERICA`.
- La etiqueta y el orden son únicos dentro de `mancomunidad + asignatura`.
- Una asignatura alfanumérica debe tener entre 2 y 20 opciones.
- Al menos una opción debe estar marcada como aprobada.
- No se borra una opción con calificaciones asociadas: se desactiva.
- Una opción inactiva conserva sus calificaciones y no se ofrece para nuevas.
- Cambiar la etiqueta de una opción con notas no reescribe el histórico: la nota guarda el identificador y se muestra la etiqueta vigente.
- No se puede cambiar la escala de la asignatura si ya tiene calificaciones ([[ADR-039 - Calificaciones por asignatura y escala configurable]]).

## Dudas

- ¿Se permite una opción «No presentado»? Sí, como una opción más, marcada como no aprobada.
- ¿Se puede reordenar una opción en uso? Sí; el orden solo afecta a la presentación.
- ¿Se pueden importar opciones de otra asignatura? En la v1 se copian manualmente al crear la asignatura.

## Relacionado

[[Mancomunidad]] · [[Asignatura]] · [[Calificación]] · [[Modelo de datos]] · [[API REST]] · [[ADR-039 - Calificaciones por asignatura y escala configurable]]
