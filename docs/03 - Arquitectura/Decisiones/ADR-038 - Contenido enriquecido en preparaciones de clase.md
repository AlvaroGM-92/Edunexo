---
tipo: adr
estado: aceptada
fecha: 2026-11-09
actualizado: 2026-11-11
tags:
  - adr
  - preparaciones
  - contenido
  - seguridad
---

# ADR-038 — Contenido enriquecido en preparaciones de clase

> [!note] Actualización del 2026-11-11
> El formato y el saneado se mantienen, pero el contenido deja de ser un campo único de la preparación: desde [[ADR-041 - Bloques de preparación y guardado incremental]] vive en bloques ordenados y el límite de 256 KB se aplica **por bloque**. `preparacion_clase.contenido` desaparece en la migración `V2`.

## Contexto

[[Preparación de clase]] guardaba `contenido` como `TEXT` plano, con un máximo de 65.535 bytes. Eso basta para varias páginas de texto, pero no permite negrita, cursiva, listas ni tablas. La revisión del 2026-11-09 pidió un campo tipo Word básico para «contenido y actividades», sin imágenes.

## Decisión

- `contenido` pasa a **`MEDIUMTEXT`** y se guarda como **HTML enriquecido saneado**.
- El límite de aplicación es **256 KB de HTML** por valor; superarlo responde `422`.
- El editor permite un subconjunto básico: párrafos, negrita, cursiva, encabezados, listas, enlaces, citas, código y **tablas**. No permite imágenes, iframes, scripts ni estilos libres.
- El saneado se hace **en el servidor** con lista blanca de etiquetas y atributos. Se guarda ya saneado y se vuelve a sanear al renderizar.
- Las tablas admiten encabezados y `colspan`/`rowspan`; en móvil se envuelven en un contenedor con scroll horizontal.
- `objetivos`, `recursos` y `observaciones` siguen siendo texto plano en la v1.
- Los adjuntos y los documentos `.docx` reales siguen fuera de alcance.

## Consecuencias

### Positivas

- El profesor puede preparar la clase con el formato mínimo que necesita y tablas de ejercicios.
- El HTML es portable, imprimible y convertible a PDF.
- No hay imágenes ni contenido activo, así que el riesgo de XSS se acota con el saneado.

### Negativas / compromisos

- Hay que mantener una lista blanca y pruebas con cargas maliciosas.
- El HTML dificulta la búsqueda textual; si se necesita, se añadirá una versión plana para indexar.
- El límite de 256 KB hay que validarlo también en la API y en el editor.
- Editar un `.docx` real requeriría OnlyOffice o Collabora y queda fuera de la v1.

## Alternativas consideradas

| Alternativa | Motivo de descarte |
| --- | --- |
| Mantener `TEXT` plano | No permite tablas ni formato básico, que es justo lo que se pide. |
| Guardar JSON del editor | Es válido, pero exige un renderizador propio y complica la exportación; el HTML saneado es más directo. |
| Markdown | Las tablas y el formato son posibles, pero la edición WYSIWYG de tablas es incómoda para el profesorado. |
| Subir y editar `.docx` | Requiere OnlyOffice/Collabora, almacenamiento, antivirus y licencias; fuera de la v1. |
| HTML sin saneado | Riesgo alto de XSS almacenado; inaceptable. |

## Relacionado

[[Preparación de clase]] · [[Modelo de datos]] · [[API REST]] · [[Requisitos funcionales]] · [[Requisitos no funcionales]] · [[Roadmap]]
