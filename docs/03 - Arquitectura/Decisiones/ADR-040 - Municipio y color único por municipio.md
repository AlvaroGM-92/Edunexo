---
tipo: adr
estado: aceptada
fecha: 2026-11-11
actualizado: 2026-11-11
tags:
  - adr
  - dominio
  - centros
  - horarios
  - multi-tenant
---

# ADR-040 — Municipio y color único por municipio

## Contexto

Hasta ahora el pueblo vivía en `Centro.localidad` y el color identificativo en `Centro.color`. El profesorado de una mancomunidad se desplaza entre pueblos, y el horario tiene que mostrar esos desplazamientos de un vistazo. El color del centro no resuelve bien ese caso: si un pueblo tiene varios centros, cada uno puede tener un color distinto y el horario pierde la lectura por municipio. Además, el color del centro no era único y dos centros podían compartirlo.

La revisión del 2026-11-11 pide separar el municipio del centro y que el color sea **único dentro de la mancomunidad**, siempre acompañado del nombre.

## Decisión

- Se crea la entidad **`Municipio`** con `mancomunidad_id`, `nombre`, `color`, `activo`, auditoría y `version`.
- `nombre` es único dentro de la mancomunidad: `UNIQUE (mancomunidad_id, nombre)`.
- `color` es obligatorio, en formato `#RRGGBB`, y **único dentro de la mancomunidad**: `UNIQUE (mancomunidad_id, color)`. Se normaliza a mayúsculas al guardar.
- **`Centro` pasa a tener `municipio_id` obligatorio** y deja de tener `localidad` y `color`. El centro debe pertenecer a la misma mancomunidad que su municipio.
- `Asignatura` sigue colgando de `Centro`. El color del horario se obtiene por `Asignatura → Centro → Municipio`; no se copia en la franja ni en la asignatura.
- El horario, la rejilla semanal, el calendario y los listados devuelven el municipio como objeto `{ id, nombre, color }`. **El color nunca aparece solo**: siempre va acompañado del nombre del municipio.
- La administración puede crear, editar, activar y desactivar municipios y reasignar un centro a otro municipio de su mancomunidad. El cambio de municipio no reescribe el histórico: las lecturas recalculan el color actual.
- Al crear un municipio sin color, el servidor asigna el primer color libre de una paleta accesible. Si la paleta se agota, responde `422` y pide un color libre. Un color ya usado en la mancomunidad responde `409`.
- No se puede desactivar un municipio con centros activos. Un municipio inactivo no se ofrece para centros nuevos, pero los centros existentes lo conservan y su histórico sigue siendo consultable.
- La migración `V2` crea `municipio`, rellena `centro.municipio_id`, elimina `centro.localidad` y `centro.color` y conserva los centros existentes.
- **No se validan tiempos de desplazamiento** entre municipios en la v1: el color es una ayuda visual, no un motor de planificación.

## Consecuencias

### Positivas

- El horario muestra de un vistazo en qué pueblo trabaja el profesor cada día.
- El color deja de depender de cuántos centros tenga un pueblo y no se duplica dentro de la mancomunidad.
- El municipio es un dato maestro reutilizable y permite filtrar por pueblo sin depender de texto libre.
- La migración conserva los centros y sus localidades actuales como municipios.

### Negativas / compromisos

- Hay una entidad más y una relación obligatoria en `Centro`.
- Cambiar el color de un municipio afecta a todos sus centros; es intencionado, pero hay que avisar en la interfaz.
- La paleta puede agotarse en mancomunidades con muchos municipios; en ese caso la administración debe elegir un color libre.
- Los datos existentes con `localidad` nula se agrupan en un municipio «Sin municipio» que la administración puede renombrar después.

## Alternativas consideradas

| Alternativa | Motivo de descarte |
| --- | --- |
| Mantener el color en `Centro` | No representa los desplazamientos entre pueblos y permite colores distintos para centros del mismo municipio. |
| `Municipio` sin color y color en `Centro` | Obliga a duplicar el color por centro y no garantiza la lectura por pueblo. |
| Tabla intermedia `centro_municipio` | Un centro pertenece a un municipio; una N:M añade complejidad sin un caso de uso real. |
| Color no único | La petición explícita es que el color sea único dentro de la mancomunidad. |
| Validar tiempos de desplazamiento | No se ha pedido y requiere horarios de transporte y reglas que la v1 no tiene. |

## Relacionado

[[Municipio]] · [[Centro]] · [[Asignatura]] · [[Horario del profesor]] · [[Modelo de dominio]] · [[Modelo de datos]] · [[API REST]] · [[Requisitos funcionales]] · [[ADR-036 - Calendario con rangos, tipos y origen manual]]
