---
tipo: entidad
estado: activo
fase: 1
actualizado: 2026-11-10
tags:
  - dominio
  - entidad
  - ausencias
  - calendario
  - multi-tenant
---

# Ausencia del profesor

## Propósito

Registra una ausencia de un profesor —asuntos propios, permiso oficial, formación, huelga parcial u otro motivo— con su periodo, estado y traza de solicitud y resolución. Sirve para que el profesor pida sus días y para que la administración los apruebe o rechace sin confundirlos con una clase cancelada ni con un [[Día no lectivo]].

Aprobar una ausencia **no modifica el horario ni las sesiones**. La administración decide después si cubre el periodo con una [[Sustitución]] o si cancela las sesiones `PROGRAMADA` afectadas. Decisión registrada en [[ADR-037 - Ausencias del profesor y huelgas]].

## Campos

| Campo | Tipo | Obligatorio | Reglas |
| --- | --- | --- | --- |
| `id` | BIGINT | Sí | Clave primaria. |
| `mancomunidad_id` | BIGINT | Sí | FK a [[Mancomunidad]]; tenant de la ausencia. |
| `curso_escolar_id` | BIGINT | Sí | FK a [[Curso escolar]] de la misma mancomunidad. |
| `profesor_id` | BIGINT | Sí | FK a [[Usuario]] con pertenencia activa `PROFESOR` en la mancomunidad. |
| `tipo` | VARCHAR(30) | Sí | `ASUNTOS_PROPIOS`, `PERMISO_OFICIAL`, `FORMACION`, `HUELGA` u `OTRO`. |
| `fecha_inicio` | DATE | Sí | Dentro del curso y anterior o igual a `fecha_fin`. |
| `fecha_fin` | DATE | Sí | Dentro del curso; el rango no puede superar 366 días. |
| `motivo` | VARCHAR(500) | Sí | 10–500 caracteres; no debe contener diagnósticos ni datos de salud. |
| `estado` | VARCHAR(20) | Sí | `PENDIENTE`, `APROBADA`, `RECHAZADA` o `CANCELADA`. |
| `respuesta_admin` | VARCHAR(500) | No | Obligatoria al rechazar, 10–500 caracteres; opcional al aprobar. |
| `solicitada_por` | BIGINT | Sí | FK a [[Usuario]] que crea la ausencia: el propio profesor o un administrador. |
| `resuelta_por` | BIGINT | No | FK a [[Usuario]] que aprueba o rechaza; obligatorio si el estado es `APROBADA` o `RECHAZADA`. |
| `resuelta_at` | DATETIME | No | Momento de la resolución. |
| `cancelada_por` | BIGINT | No | FK a [[Usuario]] que cancela; obligatorio si el estado es `CANCELADA`. |
| `cancelada_at` | DATETIME | No | Momento de la cancelación. |
| `motivo_cancelacion` | VARCHAR(500) | No | Obligatorio al cancelar una ausencia ya resuelta; 10–500 caracteres. |
| `created_at` | DATETIME | Sí | Auditoría. |
| `updated_at` | DATETIME | Sí | Auditoría. |

## Relaciones

- **N:1 con [[Mancomunidad]]**.
- **N:1 con [[Curso escolar]]**.
- **N:1 con [[Usuario]]** como profesor, solicitante, resolutor y autor de la cancelación.

## Reglas

- El profesor, el curso y la ausencia deben pertenecer a la misma mancomunidad.
- El profesor debe tener una pertenencia activa `PROFESOR` en la mancomunidad.
- Las fechas deben estar dentro del curso escolar y no puede haber ausencias nuevas en un curso `CERRADO`.
- No puede haber dos ausencias `PENDIENTE` o `APROBADA` solapadas para el mismo profesor.
- El profesor crea y consulta solo las suyas y puede cancelar las que sigan `PENDIENTE`. La administración crea, consulta, aprueba, rechaza y cancela las de cualquier profesor de su mancomunidad.
- La aprobación no cancela ni reasigna sesiones: devuelve el número de `sesionesAfectadas` y la administración decide entre [[Sustitución]] o cancelación de las `PROGRAMADA`.
- Una ausencia `RECHAZADA` o `CANCELADA` no bloquea una nueva solicitud para el mismo periodo.
- El motivo es una etiqueta administrativa: no se guardan diagnósticos, partes médicos ni datos de salud.
- La ausencia se muestra en la agenda del profesor y en el panel de administración, pero no cambia el horario semanal.
- Una huelga total de un centro o de la mancomunidad se registra como [[Día no lectivo]] de tipo `HUELGA`; una huelga parcial, como una ausencia de tipo `HUELGA` creada por la administración.

## Dudas

- ¿Puede el profesor aprobar su propia ausencia? No; solo la administración resuelve.
- ¿La ausencia descuenta días de un saldo? No en la v1; no hay saldo ni nómina.
- ¿Se puede pedir una ausencia para un curso `PLANIFICACION`? Sí, si las fechas están dentro del curso y el profesor tiene pertenencia activa; no para un curso `CERRADO`.
- ¿Se admiten ausencias de varios profesores en una sola operación? No; una fila por profesor.
- ¿Sustituye al control horario? No; el fichaje es un ámbito distinto y queda fuera de la v1.

## Relacionado

[[Mancomunidad]] · [[Curso escolar]] · [[Usuario]] · [[UsuarioMancomunidad]] · [[Sustitución]] · [[Sesión de clase]] · [[Día no lectivo]] · [[Modelo de datos]] · [[API REST]] · [[Requisitos funcionales]] · [[Roles y permisos]] · [[ADR-037 - Ausencias del profesor y huelgas]] · [[ADR-036 - Calendario con rangos, tipos y origen manual]]
