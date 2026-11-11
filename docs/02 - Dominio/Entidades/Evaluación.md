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

# Evaluación

## Propósito

Periodo del curso escolar en el que se agrupan las calificaciones de los alumnos, por ejemplo «Primera evaluación», «Segunda evaluación» o «Evaluación final». La administración define el calendario una vez por curso y todos los alumnos y asignaturas lo comparten.

## Campos

| Campo | Tipo | Obligatorio | Reglas |
| --- | --- | --- | --- |
| `id` | BIGINT | Sí | Clave primaria. |
| `mancomunidad_id` | BIGINT | Sí | FK a [[Mancomunidad]]; tenant de la evaluación. |
| `curso_escolar_id` | BIGINT | Sí | FK a [[Curso escolar]] de la misma mancomunidad. |
| `nombre` | VARCHAR(80) | Sí | Único dentro del curso; ej. «Primera evaluación». |
| `orden` | SMALLINT | Sí | Único dentro del curso; define el orden de presentación. |
| `fecha_inicio` | DATE | Sí | Dentro del rango del curso. |
| `fecha_fin` | DATE | Sí | Igual o posterior a `fecha_inicio` y dentro del curso. |
| `activa` | BOOLEAN | Sí | Por defecto `true`; una evaluación inactiva no se ofrece para nuevas notas. |
| `created_at` | DATETIME | Sí | Auditoría. |
| `updated_at` | DATETIME | Sí | Auditoría. |

## Relaciones

- **N:1 con [[Mancomunidad]]**.
- **N:1 con [[Curso escolar]]**.
- **1:N con [[Calificación]]**.

## Reglas

- El nombre y el orden son únicos dentro de `mancomunidad + curso escolar`.
- Las fechas deben estar dentro del rango `fecha_inicio`–`fecha_fin` del [[Curso escolar]].
- `fecha_fin` no puede ser anterior a `fecha_inicio`.
- Solo la administración crea, edita y desactiva evaluaciones.
- No se borra una evaluación con calificaciones asociadas: se desactiva.
- Una evaluación inactiva conserva sus calificaciones y no se ofrece para nuevas.
- En un curso `CERRADO` no se crean ni editan evaluaciones; solo se consultan.
- Las evaluaciones son del curso, no de una asignatura concreta, para que el cuaderno de notas comparta el mismo calendario en todas las asignaturas.

## Dudas

- ¿Puede haber evaluaciones distintas por centro? En la v1 no: son por curso escolar y mancomunidad.
- ¿Se pueden solapar las fechas de dos evaluaciones? Sí, la regla no lo impide; el orden es el que manda en la presentación.
- ¿Se permite una evaluación sin fechas? No, ambas son obligatorias.

## Relacionado

[[Mancomunidad]] · [[Curso escolar]] · [[Calificación]] · [[Asignatura]] · [[Modelo de datos]] · [[API REST]] · [[ADR-039 - Calificaciones por asignatura y escala configurable]]
