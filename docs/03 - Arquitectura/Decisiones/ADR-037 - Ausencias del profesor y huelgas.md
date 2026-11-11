---
tipo: adr
estado: aceptada
fecha: 2026-11-09
actualizado: 2026-11-10
tags:
  - adr
  - ausencias
  - calendario
  - profesorado
---

# ADR-037 — Ausencias del profesor y huelgas

## Contexto

La Fase 0 no tenía forma de pedir un día de **asuntos propios** ni de dejar constancia de una ausencia del profesor. Lo más parecido era cancelar sesiones `PROGRAMADA` una a una ([[ADR-017 - Cancelación de sesiones y días no lectivos]]) o registrar un [[Día no lectivo]], que afecta a todo un centro o a toda la mancomunidad y solo lo gestiona la administración.

Tampoco estaba resuelta la **huelga**. Una huelga total de un centro encaja como día no lectivo, pero una huelga parcial —solo parte del profesorado— no puede representarse con un día no lectivo sin ocultar a quién afecta.

## Decisión

### Entidad `Ausencia del profesor`

Se crea [[Ausencia del profesor]] con estos datos:

- `profesor_id`, `curso_escolar_id`, `tipo`, `fecha_inicio`, `fecha_fin` y `motivo`.
- `estado`: `PENDIENTE`, `APROBADA`, `RECHAZADA` o `CANCELADA`.
- Traza de solicitud y resolución: `solicitada_por`, `resuelta_por`, `resuelta_at` y `respuesta_admin`; opcionalmente `cancelada_por`, `cancelada_at` y `motivo_cancelacion`.

Tipos cerrados: `ASUNTOS_PROPIOS`, `PERMISO_OFICIAL`, `FORMACION`, `HUELGA` y `OTRO`.

### Flujo

- El **profesor** solicita sus propias ausencias y puede cancelar las que sigan `PENDIENTE`.
- El **administrador** puede crear ausencias para cualquier profesor de su mancomunidad, aprobarlas o rechazarlas. El rechazo exige una respuesta de 10–500 caracteres; la aprobación puede incluir respuesta.
- El administrador puede cancelar una ausencia `PENDIENTE` o `APROBADA` cuya fecha de inicio no haya pasado, con motivo de 10–500 caracteres.
- No puede haber dos ausencias `PENDIENTE` o `APROBADA` solapadas para el mismo profesor.
- El rango debe estar dentro del curso escolar y no puede superar 366 días. No se admiten ausencias nuevas en un curso `CERRADO`.
- La ausencia es **informativa y organizativa**: aprobarla **no cancela ni reasigna sesiones**. El sistema devuelve el número de `sesionesAfectadas` y la administración decide después entre una [[Sustitución]] ([[ADR-025 - Sustituciones y cambio de titular]]) o la cancelación de las sesiones `PROGRAMADA` ([[ADR-017 - Cancelación de sesiones y días no lectivos]]).
- La ausencia se muestra en la agenda del profesor y en el panel de administración, pero no modifica el horario semanal.

### Huelga

- **Huelga total** de un centro o de la mancomunidad: se registra como [[Día no lectivo]] de tipo `HUELGA` ([[ADR-036 - Calendario con rangos, tipos y origen manual]]). No genera sesiones y cancela las `PROGRAMADA` sin asistencia.
- **Huelga parcial**: se registra una [[Ausencia del profesor]] de tipo `HUELGA` por cada profesor afectado, creada por la administración. No es una solicitud del profesor ni pasa por aprobación: la administración la deja constancia con su autor y fecha.
- La huelga no se modela como un permiso retribuido ni como una solicitud; es un hecho organizativo que se refleja en el calendario o en la ausencia del profesor.

### API

- `GET /ausencias` y `GET /ausencias/{id}`: el profesor ve las suyas; la administración, todas.
- `POST /ausencias`: el profesor crea las propias; la administración, las de cualquier profesor.
- `POST /ausencias/{id}/aprobar`, `/rechazar` y `/cancelar`.
- La respuesta de alta y resolución incluye `sesionesAfectadas` y `version`.

## Consecuencias

### Positivas

- Los asuntos propios dejan de depender de cancelar clases una a una y quedan con estado, autor y fechas.
- La administración ve las ausencias y puede decidir sustitución o cancelación con la información de sesiones afectadas.
- La huelga total y la parcial tienen representaciones distintas y coherentes con el calendario y las sustituciones.
- El profesor no necesita permisos de administración para solicitar sus asuntos propios.

### Negativas / compromisos

- Hay una entidad, un flujo de estados y endpoints nuevos.
- Aprobar una ausencia no resuelve por sí solo las clases; la administración debe crear la sustitución o cancelar las sesiones.
- La ausencia no sustituye al control horario ni al fichaje: son ámbitos distintos.
- El motivo es texto libre y no debe usarse para almacenar diagnósticos ni datos de salud; la ausencia registra el hecho administrativo, no el detalle médico.

## Alternativas consideradas

| Alternativa | Motivo de descarte |
| --- | --- |
| Usar solo la cancelación de sesiones | No deja una solicitud ni un estado; el profesor tendría que cancelar cada clase y no habría aprobación. |
| Usar un día no lectivo para cualquier ausencia | Afecta a todo el centro o a toda la mancomunidad y solo lo gestiona la administración. |
| Modelar la ausencia como una sustitución | Una sustitución exige un sustituto y una asignatura; una ausencia puede no tener sustituto todavía. |
| Aprobar y cancelar sesiones automáticamente | Puede haber una sustitución prevista; cancelar automáticamente borraría esa opción y sorprendería al usuario. |
| Un tipo `BAJA_MEDICA` con detalle | Introduce datos de salud y obligaciones adicionales; el motivo libre ya permite dejar constancia administrativa sin diagnóstico. |

## Relacionado

[[Ausencia del profesor]] · [[Día no lectivo]] · [[Sustitución]] · [[Sesión de clase]] · [[Curso escolar]] · [[Roles y permisos]] · [[API REST]] · [[Modelo de datos]] · [[Requisitos funcionales]] · [[Roadmap]] · [[ADR-017 - Cancelación de sesiones y días no lectivos]] · [[ADR-025 - Sustituciones y cambio de titular]] · [[ADR-036 - Calendario con rangos, tipos y origen manual]]
