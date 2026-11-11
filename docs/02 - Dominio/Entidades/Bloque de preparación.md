---
tipo: entidad
estado: activo
fase: 1
actualizado: 2026-11-11
tags:
  - dominio
  - entidad
  - docencia
  - multi-tenant
---

# Bloque de preparación

## Propósito

Unidad ordenada de una [[Preparación de clase]]. Cada bloque agrupa un título, unas fechas y un contenido enriquecido, de modo que el profesor puede preparar la clase por partes y autoguardar cada bloque sin reenviar toda la preparación.

## Campos

| Campo | Tipo | Obligatorio | Reglas |
| --- | --- | --- | --- |
| `id` | BIGINT | Sí | Clave primaria. |
| `mancomunidad_id` | BIGINT | Sí | FK a [[Mancomunidad]]; tenant del bloque. |
| `preparacion_id` | BIGINT | Sí | FK a [[Preparación de clase]]; se elimina en cascada con ella. |
| `orden` | SMALLINT | Sí | Posición dentro de la preparación; de `1` a `N`, sin huecos ni duplicados. |
| `titulo` | VARCHAR(150) | Sí | Título del bloque. |
| `fecha_inicio` | DATE | No | Fecha del bloque o inicio del rango. |
| `fecha_fin` | DATE | No | Fin del rango; si existe, debe ser igual o posterior a `fecha_inicio`. |
| `contenido` | MEDIUMTEXT | Sí | HTML enriquecido saneado; máximo 256 KB por bloque. |
| `created_at` | DATETIME | Sí | Auditoría. |
| `updated_at` | DATETIME | Sí | Auditoría. |
| `version` | BIGINT | Sí | Bloqueo optimista; por defecto `0`. |

## Relaciones

- **N:1 con [[Mancomunidad]]**.
- **N:1 con [[Preparación de clase]]**.

## Reglas

- El `orden` es único dentro de la preparación: `UNIQUE (mancomunidad_id, preparacion_id, orden)`. La lista se guarda compacta, de `1` a `N`; al borrar un bloque se reordenan los siguientes.
- Una preparación admite como máximo **100 bloques**; superarlo responde `422`.
- `titulo` y `contenido` son obligatorios. `fecha_inicio` y `fecha_fin` son opcionales; si ambas vienen, `fecha_fin >= fecha_inicio`.
- Si el bloque informa fechas, deben caer dentro del curso escolar de la preparación.
- `contenido` admite el mismo formato básico y las mismas tablas que la preparación: párrafos, negrita, cursiva, encabezados, listas, enlaces, citas, código y tablas. No admite imágenes, iframes, scripts ni estilos libres ([[ADR-038 - Contenido enriquecido en preparaciones de clase]]).
- El HTML se sanea en el servidor con lista blanca; se guarda saneado y se vuelve a sanear al renderizar.
- El límite es de **256 KB de HTML por bloque**, no por preparación. Superarlo responde `422`.
- El bloque se puede autoguardar individualmente. Editar título, fechas o contenido incrementa solo la `version` del bloque.
- Añadir, borrar o reordenar bloques incrementa también la `version` de la preparación.
- El bloque pertenece a la misma mancomunidad que la preparación y no se puede editar si la preparación está `IMPARTIDA`.
- Al eliminar la preparación se eliminan sus bloques; solo se puede eliminar la preparación en `BORRADOR`.

## Dudas

- ¿Un bloque puede repetirse en otra preparación? Se puede copiar su contenido manualmente; duplicar preparaciones completas queda para una mejora posterior.
- ¿Se adjuntan archivos por bloque? Fuera de alcance en la v1.
- ¿Se permiten imágenes? No en la v1; si se necesitan, se valorarán como adjuntos.
- ¿El título del bloque sustituye al de la preparación? No; la preparación conserva su título y cada bloque tiene el suyo.

## Relacionado

[[Preparación de clase]] · [[Mancomunidad]] · [[Modelo de datos]] · [[API REST]] · [[ADR-038 - Contenido enriquecido en preparaciones de clase]] · [[ADR-041 - Bloques de preparación y guardado incremental]]
