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

# Municipio

## Propósito

Pueblo o término municipal al que pertenecen uno o varios [[Centro|centros]] de la mancomunidad. Existe para representar los desplazamientos del profesorado entre localidades y para dar un color estable a cada pueblo en el horario, el calendario y los listados.

No debe confundirse con [[Centro]]: un municipio puede tener varios centros y el color pertenece al municipio, no al centro.

## Campos

| Campo | Tipo | Obligatorio | Reglas |
| --- | --- | --- | --- |
| `id` | BIGINT | Sí | Clave primaria. |
| `mancomunidad_id` | BIGINT | Sí | FK a [[Mancomunidad]]; tenant del municipio. |
| `nombre` | VARCHAR(120) | Sí | Único dentro de la mancomunidad. |
| `color` | VARCHAR(7) | Sí | `#RRGGBB`; único dentro de la mancomunidad; se guarda en mayúsculas. |
| `activo` | BOOLEAN | Sí | Por defecto `true`. |
| `created_at` | DATETIME | Sí | Auditoría. |
| `updated_at` | DATETIME | Sí | Auditoría. |
| `version` | BIGINT | Sí | Bloqueo optimista; por defecto `0`. |

## Relaciones

- **N:1 con [[Mancomunidad]]**.
- **1:N con [[Centro]]**.

## Reglas

- El nombre es único dentro de la mancomunidad: `UNIQUE (mancomunidad_id, nombre)`.
- El color es obligatorio, cumple `^#[0-9A-Fa-f]{6}$` y es **único dentro de la mancomunidad**: `UNIQUE (mancomunidad_id, color)`. Se normaliza a mayúsculas antes de comparar y guardar.
- Al crear un municipio sin color, el servidor asigna el primer color libre de la paleta accesible. Si la paleta se agota, responde `422` y pide un color libre.
- Un color ya usado por otro municipio de la mancomunidad responde `409`, también si el municipio que lo usa está inactivo.
- El municipio y sus centros deben pertenecer a la misma mancomunidad.
- No se puede desactivar un municipio con centros activos. Los centros existentes conservan un municipio inactivo y su histórico sigue siendo consultable.
- Un municipio inactivo no se ofrece para centros nuevos.
- El color identifica el municipio en horarios, calendario y listados, pero **nunca es el único indicador**: siempre aparece el nombre del municipio al lado.
- Cambiar el color de un municipio afecta a todos sus centros. El color no se copia en la asignatura ni en la franja: se obtiene por `Asignatura → Centro → Municipio` al leer.
- El municipio es local a la mancomunidad: dos mancomunidades pueden tener un municipio con el mismo nombre y color sin colisionar.
- El municipio es un dato de identificación y ayuda visual; **no se validan tiempos de desplazamiento** entre municipios en la v1.
- El municipio no importa ni deduce festivos: el calendario sigue siendo manual ([[ADR-036 - Calendario con rangos, tipos y origen manual]]).

## Dudas

- ¿Un municipio puede tener varios centros? Sí, y todos comparten su color.
- ¿Un centro puede cambiar de municipio? Sí, la administración puede reasignarlo; el horario recalcula el color al leer.
- ¿Se puede reutilizar el color de un municipio inactivo? No directamente: hay que cambiar el color del inactivo antes de asignarlo a otro.
- ¿Hay un catálogo global de municipios? No; cada mancomunidad gestiona el suyo.
- ¿Se calculan desplazamientos o tiempos de viaje? No en la v1; el color es solo una ayuda visual.

## Relacionado

[[Mancomunidad]] · [[Centro]] · [[Asignatura]] · [[Horario del profesor]] · [[Multi-tenencia]] · [[Modelo de dominio]] · [[Modelo de datos]] · [[API REST]] · [[ADR-040 - Municipio y color único por municipio]]
