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

# Calificación

## Propósito

Nota que un profesor registra para un alumno en una asignatura y una evaluación concretas. Puede ser numérica o alfanumérica según la escala configurada en la asignatura. La administración la consulta, pero no la crea ni la modifica.

## Campos

| Campo | Tipo | Obligatorio | Reglas |
| --- | --- | --- | --- |
| `id` | BIGINT | Sí | Clave primaria. |
| `mancomunidad_id` | BIGINT | Sí | FK a [[Mancomunidad]]; tenant de la calificación. |
| `matricula_id` | BIGINT | Sí | FK a [[Matrícula]]; fija alumno, asignatura y curso. |
| `evaluacion_id` | BIGINT | Sí | FK a [[Evaluación]] del mismo curso que la matrícula. |
| `profesor_id` | BIGINT | Sí | FK a [[Usuario]] con pertenencia activa `PROFESOR` y que imparte la asignatura. |
| `valor_numerico` | DECIMAL(4,2) | No | Solo si la asignatura es numérica; dentro del rango configurado. |
| `opcion_calificacion_id` | BIGINT | No | Solo si la asignatura es alfanumérica; FK a [[Opción de calificación]] de la misma asignatura. |
| `comentario` | VARCHAR(500) | No | Observación breve de la nota. |
| `created_at` | DATETIME | Sí | Auditoría; fecha de registro. |
| `updated_at` | DATETIME | Sí | Auditoría. |
| `version` | BIGINT | Sí | Bloqueo optimista. |

## Relaciones

- **N:1 con [[Mancomunidad]]**.
- **N:1 con [[Matrícula]]**.
- **N:1 con [[Evaluación]]**.
- **N:1 con [[Usuario]]** (profesor).
- **N:1 con [[Opción de calificación]]** (solo alfanumérica).

## Reglas

- Exactamente uno de `valor_numerico` u `opcion_calificacion_id` debe estar informado; nunca los dos.
- La forma del valor debe coincidir con `Asignatura.escala_calificacion`.
- El `valor_numerico` debe estar entre `nota_minima` y `nota_maxima` y respetar `decimales`.
- La `opcion_calificacion_id` debe pertenecer a la misma asignatura y estar activa.
- La matrícula debe estar `ACTIVA` en el momento del alta; una matrícula `BAJA` conserva sus calificaciones como histórico.
- El profesor debe impartir la asignatura de la matrícula.
- La evaluación debe pertenecer al mismo curso escolar que la matrícula.
- Solo puede existir una calificación por `matrícula + evaluación`.
- El profesor puede crear, corregir y eliminar sus propias calificaciones; la administración solo consulta.
- En un curso `CERRADO` no se crean ni editan calificaciones; solo se consultan.
- Las calificaciones se muestran en la ficha del alumno y en el cuaderno de notas de la asignatura, filtradas por curso y evaluación.
- No hay historial de cambios de calificación en la v1; se guardan `updated_at` y `version` ([[ADR-039 - Calificaciones por asignatura y escala configurable]]).

## Dudas

- ¿Se puede calificar a un alumno sin matrícula activa? No; primero hay que matricularlo o reactivarlo.
- ¿Se puede borrar una calificación? Sí, el profesor puede eliminarla mientras el curso no esté cerrado.
- ¿Se calcula la media? No en la v1; queda fuera de alcance junto con boletines y expedientes.
- ¿Se muestra al alumno o a la familia? No hay acceso de alumno en la v1; solo profesorado y administración.

## Relacionado

[[Mancomunidad]] · [[Matrícula]] · [[Evaluación]] · [[Opción de calificación]] · [[Asignatura]] · [[Alumno]] · [[Usuario]] · [[Curso escolar]] · [[Modelo de datos]] · [[API REST]] · [[ADR-039 - Calificaciones por asignatura y escala configurable]]
