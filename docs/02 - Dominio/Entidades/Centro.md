---
tipo: entidad
estado: activo
fase: 1
actualizado: 2026-11-11
tags:
  - dominio
  - entidad
  - multi-tenant
---

# Centro

## Propósito

Centro de enseñanza que forma parte de la mancomunidad. Agrupa asignaturas y alumnos; su área depende de las asignaturas que oferte, no de una restricción del centro. Cada centro pertenece a un [[Municipio]], que aporta el color con el que se reconoce el pueblo en el horario.

## Campos

| Campo | Tipo | Obligatorio | Reglas |
| --- | --- | --- | --- |
| `id` | BIGINT | Sí | Clave primaria. |
| `mancomunidad_id` | BIGINT | Sí | FK a [[Mancomunidad]]; tenant al que pertenece el centro. |
| `municipio_id` | BIGINT | Sí | FK a [[Municipio]] de la misma mancomunidad. |
| `nombre` | VARCHAR(150) | Sí | Único dentro de la mancomunidad. |
| `direccion` | VARCHAR(255) | No | Texto libre. |
| `telefono` | VARCHAR(30) | No | Formato flexible. |
| `email` | VARCHAR(150) | No | Formato válido si se informa. |
| `activo` | BOOLEAN | Sí | Por defecto `true`. |
| `created_at` | DATETIME | Sí | Auditoría. |
| `updated_at` | DATETIME | Sí | Auditoría. |
| `version` | BIGINT | Sí | Bloqueo optimista; por defecto `0`. |

## Relaciones

- **N:1 con [[Mancomunidad]]**.
- **N:1 con [[Municipio]]**.
- **1:N con [[Asignatura]]**.
- **1:N con [[Alumno]]**.

## Reglas

- El nombre es único dentro de la mancomunidad: `UNIQUE (mancomunidad_id, nombre)`.
- El municipio debe pertenecer a la misma mancomunidad que el centro.
- El centro y sus datos solo son visibles desde su mancomunidad.
- Un centro no puede relacionarse con asignaturas o alumnos de otra mancomunidad.
- No se puede desactivar un centro con asignaturas activas, alumnos activos o usuarios vinculados.
- Los centros inactivos no aparecen en nuevas asignaciones.
- El histórico de un centro inactivo se conserva y sigue siendo consultable.
- El centro puede cambiar de municipio; el color del horario se obtiene por `Asignatura → Centro → Municipio` al leer y no se guarda en la asignatura ni en la franja.
- El color no pertenece al centro: es del municipio y es único dentro de la mancomunidad ([[ADR-040 - Municipio y color único por municipio]]).
- El municipio es un dato identificativo y de ayuda al alta manual del calendario; no se usa para deducir ni importar festivos ([[ADR-036 - Calendario con rangos, tipos y origen manual]]).

## Dudas

- ¿Debe existir un centro «sede» por defecto? No en la v1.
- ¿Se guarda el CIF u otro dato fiscal? Fuera de alcance.
- ~~¿El color es del pueblo o del centro?~~ **Resuelto: del municipio; si un pueblo tiene varios centros, todos comparten el color** ([[ADR-040 - Municipio y color único por municipio]]).
- ¿Un centro puede cambiar de municipio? Sí; la administración lo reasigna y el horario recalcula el color.

## Relacionado

[[Mancomunidad]] · [[Municipio]] · [[Multi-tenencia]] · [[Visión y alcance]] · [[Día no lectivo]] · [[Modelo de datos]] · [[API REST]] · [[ADR-036 - Calendario con rangos, tipos y origen manual]] · [[ADR-040 - Municipio y color único por municipio]]
