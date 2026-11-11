---
tipo: adr
estado: aceptada
fecha: 2026-11-09
actualizado: 2026-11-11
tags:
  - adr
  - calendario
  - dias-no-lectivos
---

# ADR-036 — Calendario con rangos, tipos y origen manual

> [!note] Actualización del 2026-11-11
> La decisión de registrar el calendario a mano se mantiene. `Centro.localidad` se sustituye por [[Municipio]] en [[ADR-040 - Municipio y color único por municipio]]; el municipio sigue siendo informativo y no dispara ninguna importación de festivos.

## Contexto

[[ADR-017 - Cancelación de sesiones y días no lectivos]] resolvió los festivos con [[Día no lectivo]]: una fila por fecha, ámbito de mancomunidad o centro y registro manual. La revisión de la Fase 0 detectó tres fricciones:

- Las vacaciones y periodos largos obligaban a crear una fila por día, sin operación de rango.
- El motivo era texto libre (`nombre`), sin un tipo cerrado que permitiera filtrar, resumir o distinguir una huelga de unas vacaciones.
- No estaba escrito de dónde salían los festivos locales. [[Centro]] no tenía localidad y la documentación ya decía que no se importaban calendarios oficiales, pero faltaba cerrar la decisión.

## Decisión

### Una fila por fecha, alta y baja por rango

- Se mantiene **una fila de `dia_no_lectivo` por fecha**. El generador, la unicidad y la cancelación de sesiones no cambian.
- `POST /dias-no-lectivos` acepta `fechaInicio` y `fechaFin` (iguales para un solo día) y crea en **una sola transacción** una fila por cada fecha natural del rango, todas con el mismo `centroId`, `tipo` y `nombre`.
- El rango es **todo o nada**: si alguna fecha ya existe para el mismo ámbito, no se crea ninguna y se responde `409` con la lista `fechasDuplicadas`.
- `fechaInicio` y `fechaFin` deben pertenecer al curso indicado, `fechaInicio <= fechaFin` y el rango no puede superar **366 días**.
- `DELETE /dias-no-lectivos` acepta los mismos filtros de rango y elimina todas las filas del ámbito en una transacción, devolviendo `diasEliminados`. El borrado por `id` sigue disponible para una fecha suelta.
- Eliminar un día o un rango **no reactiva** las sesiones ya canceladas; el generador volverá a crear las `PROGRAMADA` futuras en la siguiente ejecución.

### Tipo cerrado

- Se añade `tipo` obligatorio a [[Día no lectivo]], con los valores `FESTIVO`, `VACACIONES`, `HUELGA` y `OTRO`.
- `nombre` sigue siendo obligatorio y libre, para el detalle legible («Festivo local», «Vacaciones de Navidad», «Huelga del profesorado»).
- El tipo no altera la generación ni la cancelación: todas las filas de un día no lectivo se comportan igual.

### Origen manual y localidad informativa

- El calendario se **registra a mano** por la administración. La v1 **no importa** calendarios oficiales ni los deduce de ninguna fuente externa.
- [[Centro]] añade `localidad` opcional (`VARCHAR(120)`), como dato identificativo y de ayuda al alta manual. **No** dispara ninguna búsqueda ni carga automática de festivos.
- La elección del ámbito —toda la mancomunidad o un centro concreto— es lo que permite que dos centros tengan festivos locales distintos.

## Consecuencias

### Positivas

- Vacaciones y periodos largos se registran y se eliminan en una operación, sin perder el detalle por fecha que necesita el generador.
- El tipo permite filtrar el calendario y separar festivos, vacaciones y huelgas sin depender del texto libre.
- Queda cerrada la expectativa de importación automática: el calendario es manual y la localidad no es una fuente de datos.
- La operación por rango es atómica y evita calendarios a medias.

### Negativas / compromisos

- El alta por rango crea muchas filas y puede ser costosa en cursos con vacaciones largas; se acota a 366 días y se ejecuta en una transacción.
- El borrado por rango debe respetar los mismos filtros de ámbito para no eliminar días de otro centro.
- `localidad` no aporta automatización; es un dato de identificación del centro.
- La importación de calendarios oficiales sigue siendo una ampliación futura.

## Alternativas consideradas

| Alternativa | Motivo de descarte |
| --- | --- |
| Cambiar la entidad a `fecha_inicio`/`fecha_fin` | Complica la unicidad por fecha, los solapamientos y la consulta diaria del generador. |
| Mantener solo el alta de un día | Obliga a decenas de peticiones para unas vacaciones y deja calendarios a medias si falla una. |
| Deducir festivos de `Centro.localidad` | No hay fuente fiable ni importación en la v1, y los festivos locales cambian por año y por centro. |
| Importar calendarios oficiales | Fuera de alcance de la v1; requiere integraciones y revisión manual de todas formas. |
| Usar solo `nombre` sin `tipo` | Impide filtrar y resumir por tipo; el texto libre acaba siendo inconsistente. |

## Relacionado

[[Día no lectivo]] · [[Centro]] · [[Curso escolar]] · [[Sesión de clase]] · [[API REST]] · [[Modelo de datos]] · [[Requisitos funcionales]] · [[Roadmap]] · [[ADR-017 - Cancelación de sesiones y días no lectivos]] · [[ADR-037 - Ausencias del profesor y huelgas]]
