---
tipo: adr
estado: aceptada
fecha: 2026-11-09
actualizado: 2026-11-10
tags:
  - adr
  - calificaciones
  - alumnado
  - evaluacion
---

# ADR-039 — Calificaciones por asignatura y escala configurable

## Contexto

La v1 dejaba fuera de alcance las notas numéricas, los boletines y los expedientes académicos. La revisión del 2026-11-09 pidió poder registrar calificaciones de alumnos con dos escalas: una numérica y otra alfanumérica con opciones cerradas definidas por la administración al crear la asignatura.

El requisito tiene tres decisiones encadenadas: dónde se elige la escala, cómo se guardan las opciones alfanuméricas y cuántas notas puede tener un alumno. La escala por alumno haría incomparables las notas del mismo grupo; la opción más coherente es fijarla por asignatura.

## Decisión

- La **escala es por asignatura**, no por alumno ni por profesor. `Asignatura.escala_calificacion` admite `NUMERICA` o `ALFANUMERICA`.
- En la escala **numérica** se configura el rango, la nota de aprobado y los decimales: por defecto `0`–`10`, aprobado en `5` y dos decimales.
- En la escala **alfanumérica** la administración define una lista ordenada de opciones al crear o editar la asignatura. Cada opción tiene etiqueta, orden, indicador de aprobado y estado. Mínimo 2 y máximo 20.
- Las opciones viven en la entidad [[Opción de calificación]], no en texto libre, para conservar identificadores estables y poder desactivarlas sin romper el histórico.
- Las **evaluaciones son configurables por curso escolar** y las define la administración: nombre, orden, fechas y estado. Todos los alumnos de ese curso comparten el mismo calendario de evaluaciones.
- Una [[Calificación]] se guarda por **matrícula y evaluación**. El profesor puede registrar y corregir las de las asignaturas que imparte; la administración consulta todas.
- No se puede cambiar la escala de una asignatura que ya tiene calificaciones ni borrar una opción o evaluación en uso: se desactiva.
- Las calificaciones se muestran en la ficha del alumno y en el cuaderno de notas de la asignatura, filtradas por curso y evaluación.
- Quedan fuera de alcance los boletines, los expedientes académicos, las certificaciones oficiales, el cálculo de medias y la exportación a `.docx` o PDF.

## Consecuencias

### Positivas

- El profesorado puede usar la escala que ya emplea su mancomunidad sin forzar una conversión.
- Las notas son datos estructurados y comparables, no texto libre.
- La escala única por asignatura evita mezclar criterios dentro del mismo grupo.
- La administración mantiene el control de las opciones y de las evaluaciones.

### Negativas / compromisos

- Añade tres tablas y una pantalla nueva a la Fase 5, que ya era amplia.
- Cambiar de escala o de opciones con notas vivas exige desactivar y crear, no reescribir el histórico.
- No hay historial de cambios de calificación en la v1: solo `updated_at` y `version`. Si la administración lo necesita, se añadirá como historial específico.
- Las calificaciones son datos académicos de menores; el acceso debe limitarse por rol y por asignatura.
- La retención se mantiene como histórico académico y no se purga automáticamente en la v1; se revisará con el resto de la política de conservación.

## Alternativas consideradas

| Alternativa | Motivo de descarte |
| --- | --- |
| Escala por alumno | Rompe la comparabilidad del grupo y complica el desplegable de opciones. |
| Texto libre para la escala alfanumérica | No es comparable, no se puede validar y no permite saber qué opciones aprueban. |
| Escala fija por mancomunidad | Menos flexible; una mancomunidad puede mezclar áreas y criterios. |
| Una única nota final por asignatura | No cubre el seguimiento por evaluaciones que pidió el usuario. |
| Integrar un LMS externo | Fuera de alcance y desproporcionado para la v1. |
| Boletines y expedientes | Requieren plantillas, firmas y requisitos oficiales; quedan fuera. |

## Relacionado

[[Asignatura]] · [[Opción de calificación]] · [[Evaluación]] · [[Calificación]] · [[Matrícula]] · [[Alumno]] · [[Curso escolar]] · [[Modelo de dominio]] · [[Modelo de datos]] · [[API REST]] · [[Requisitos funcionales]] · [[Roles y permisos]] · [[Roadmap]] · [[Fase 0 - Definición y documentación]]
