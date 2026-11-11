---
tipo: referencia
estado: borrador
fase: 0
actualizado: 2026-11-11
tags:
  - referencia
  - trazabilidad
  - requisitos
---

# Trazabilidad

> [!info] Para qué sirve
> Matriz que enlaza cada requisito con su endpoint, su prueba y su fase, para saber qué falta antes de la Fase 10. Se mantiene actualizada al cerrar cada fase; en la Fase 0 solo se fija la estructura.

## Estructura

| Requisito | Endpoint principal | Prueba | Fase | Estado |
| --- | --- | --- | --- | --- |
| `AUTH-01` | `POST /auth/login` | Login correcto/incorrecto, 0/1/varias pertenencias | 2 | Pendiente |
| `AUTH-16` | `POST /auth/login` | Bloqueo por 5/20 fallos en 15 min | 2 | Pendiente |
| `USR-02` | `POST /usuarios` | Alta de profesor y asignación de pertenencia | 3 | Pendiente |
| `MUN-01` | `POST /municipios` | Alta y edición de municipio con nombre único | 3 | Pendiente |
| `MUN-02` | `POST /municipios` · `PUT /municipios/{id}` | Color `#RRGGBB` único por mancomunidad; duplicado responde `409` | 3 | Pendiente |
| `MUN-04` | `PATCH /municipios/{id}/estado` | Desactivación bloqueada con centros activos | 3 | Pendiente |
| `MUN-06` | `GET /horarios/{id}/rejilla` | Color y nombre del municipio en cada celda | 6 | Pendiente |
| `CEN-05` | `POST /centros` · `PUT /centros/{id}` | Municipio obligatorio de la misma mancomunidad y reasignación | 3 | Pendiente |
| `HOR-01` | `POST /horarios` | Creación de horario en borrador | 6 | Pendiente |
| `SES-01` | `POST /sesiones/{id}/abrir` | Apertura y carga de alumnos como presentes | 8 | Pendiente |
| `CAL-01` | `POST /dias-no-lectivos` | Alta por fecha y por rango con tipo y cancelación de sesiones | 8 | Pendiente |
| `CAL-05` | `POST /dias-no-lectivos` · `DELETE /dias-no-lectivos` | Rango todo o nada: duplicados, fechas fuera de curso y más de 366 días | 8 | Pendiente |
| `AUS-01` | `POST /ausencias` | Profesor solicita asuntos propios; administrador crea para otro profesor | 8 | Pendiente |
| `AUS-03` | `POST /ausencias/{id}/aprobar` · `POST /ausencias/{id}/rechazar` | Resolución con respuesta, autor y fecha; rechazo exige 10–500 caracteres | 8 | Pendiente |
| `AUS-06` | `POST /ausencias/{id}/aprobar` | Devuelve `sesionesAfectadas` sin cancelar ni reasignar sesiones | 8 | Pendiente |
| `NOT-01` | `POST /asignaturas` · `PUT /asignaturas/{id}` | Escala numérica o alfanumérica, bloqueo al existir calificaciones | 5 | Pendiente |
| `NOT-03` | `PUT /asignaturas/{id}/opciones-calificacion` | Opciones ordenadas, 2–20, al menos una aprobada | 5 | Pendiente |
| `EVA-01` | `POST /evaluaciones` | Evaluación con nombre, orden y fechas dentro del curso | 5 | Pendiente |
| `NOT-04` | `PUT /calificaciones/lote` | Nota numérica o alfanumérica coherente con la escala | 5 | Pendiente |
| `NOT-07` | `POST /calificaciones` | Duplicado de matrícula y evaluación responde `409` | 5 | Pendiente |
| `NOT-09` | `GET /alumnos/{id}/calificaciones` | Profesor solo sus asignaturas; administración todas | 5 | Pendiente |
| `PRE-09` | `POST /preparaciones` · `GET /preparaciones/{id}` | Bloques ordenados de `1` a `N` con título, fechas y contenido | 9 | Pendiente |
| `PRE-10` | `POST /preparaciones/{id}/bloques` · `PUT /preparaciones/{id}/bloques/{bloqueId}` | Autoguardado de un bloque sin reenviar la preparación | 9 | Pendiente |
| `PRE-11` | `PUT /preparaciones/{id}` | Guardado completo de cabecera y bloques en una transacción | 9 | Pendiente |
| `PRE-12` | `PUT /preparaciones/{id}/bloques/orden` | Reordenación compacta y máximo de 100 bloques | 9 | Pendiente |
| `PRE-13` | `POST/PUT /preparaciones/{id}/bloques` | Conflicto de `version` de cabecera o bloque responde `409` | 9 | Pendiente |

## Cómo se mantiene

- Cada requisito de [[Requisitos funcionales]] debe tener al menos una fila cuando su fase empiece.
- El estado sigue el de la fase: `Pendiente`, `En curso`, `Hecho`.
- Si un requisito se divide en varios endpoints, se añade una fila por endpoint.
- Las pruebas de aislamiento se referencian en la columna `Prueba` junto a la funcional.
- Al cerrar una fase se revisa que no queden requisitos huérfanos.

## Relacionado

[[Requisitos funcionales]] · [[API REST]] · [[Roadmap]] · [[Convenciones]] · [[Modelo de datos]] · [[Municipio]] · [[Bloque de preparación]] · [[Calificación]] · [[Evaluación]] · [[ADR-039 - Calificaciones por asignatura y escala configurable]] · [[ADR-040 - Municipio y color único por municipio]] · [[ADR-041 - Bloques de preparación y guardado incremental]]