---
tipo: adr
estado: aceptada
fecha: 2026-11-11
actualizado: 2026-11-11
tags:
  - adr
  - preparaciones
  - contenido
  - concurrencia
---

# ADR-041 — Bloques de preparación y guardado incremental

## Contexto

[[Preparación de clase]] guardaba todo el contenido en un único campo `contenido`. Para una clase larga, el profesor trabaja con un editor enorme y, si la sesión se reinicia o se pierde la conexión, puede perder todo lo escrito. La revisión del 2026-11-11 pide dividir la preparación en varios bloques ordenados, cada uno con título, fechas y contenido, con autoguardado por bloque y un guardado completo al final.

## Decisión

- `preparacion_clase` queda como **cabecera**: profesor, asignatura, curso, sesión opcional, fecha, título, objetivos, recursos, observaciones, estado, auditoría y `version`. **Pierde `contenido`**.
- Se crea **`preparacion_bloque`** con `mancomunidad_id`, `preparacion_id`, `orden`, `titulo`, `fecha_inicio`, `fecha_fin`, `contenido`, auditoría y `version`.
- `orden` es único dentro de la preparación: `UNIQUE (mancomunidad_id, preparacion_id, orden)`. La lista se guarda compacta, de `1` a `N`, y al borrar un bloque se reordenan los siguientes.
- `fecha_inicio` y `fecha_fin` son opcionales; si ambas vienen, `fecha_fin >= fecha_inicio`. Permiten un bloque de un día o un rango.
- `contenido` es HTML enriquecido saneado con la misma lista blanca de [[ADR-038 - Contenido enriquecido en preparaciones de clase]]. El límite es de **256 KB por bloque**, no por preparación.
- Una preparación admite como máximo **100 bloques**; es un límite técnico de protección, no una regla pedagógica.
- **Autoguardado por bloque:** `POST /preparaciones/{id}/bloques` y `PUT /preparaciones/{id}/bloques/{bloqueId}` guardan un solo bloque. Editar título, fechas o contenido incrementa solo la `version` del bloque.
- **Guardado completo:** `PUT /preparaciones/{id}` guarda la cabecera y todos los bloques en una transacción, con la `version` de la preparación. Es la acción explícita de cierre y la red de seguridad antes de salir o recargar.
- Añadir, borrar o reordenar bloques es un cambio estructural e incrementa también la `version` de la preparación. Reordenar usa `PUT /preparaciones/{id}/bloques/orden` con la lista completa de identificadores.
- La concurrencia es optimista: un desajuste de `version` de la preparación o del bloque responde `409`. En el autoguardado, la respuesta incluye las versiones actuales para que la interfaz muestre el conflicto; nunca se sobrescribe en silencio.
- La migración `V2` convierte el `contenido` existente en un bloque `orden = 1` con el título de la cabecera y después elimina la columna, sin perder texto.
- Los bloques se eliminan en cascada con su preparación. Solo se puede borrar la preparación en `BORRADOR`, como antes.

## Consecuencias

### Positivas

- El profesor puede trabajar por partes y no pierde lo ya escrito si se reinicia la sesión.
- El editor es más pequeño y la lectura de la preparación es más clara.
- El bloque tiene su propia `version`, así que dos pestañas no se pisan al editar bloques distintos.
- El guardado completo deja una instantánea coherente de toda la preparación.

### Negativas / compromisos

- Hay más endpoints y dos niveles de concurrencia: cabecera y bloque.
- Reordenar exige una operación específica para no chocar con la restricción única de `orden`.
- El cliente debe decidir cuándo autoguardar y cuándo hacer el guardado completo; se documenta en la API.
- El límite de 256 KB ahora se valida por bloque, no por preparación.

## Alternativas consideradas

| Alternativa | Motivo de descarte |
| --- | --- |
| Mantener un único `contenido` | No permite bloques ordenados ni autoguardado parcial. |
| Guardar los bloques como JSON en la cabecera | Dificulta el autoguardado por bloque, la concurrencia fina y las consultas. |
| Autoguardado solo en el navegador | No protege si se cambia de dispositivo ni si se limpia el almacenamiento local. |
| Guardado completo en cada pulsación | Genera escrituras grandes y conflictos de versión innecesarios. |
| Sin límite de bloques | Permite abusos de almacenamiento y preparaciones inmanejables. |

## Relacionado

[[Preparación de clase]] · [[Bloque de preparación]] · [[Modelo de datos]] · [[API REST]] · [[Ejemplos de API]] · [[Requisitos funcionales]] · [[ADR-038 - Contenido enriquecido en preparaciones de clase]]
