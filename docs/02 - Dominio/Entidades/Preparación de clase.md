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

# Preparación de clase

## Propósito

Planificación docente de una clase concreta. Es la **cabecera** de la preparación: guarda el contexto (profesor, asignatura, curso, fecha y estado) y los datos generales. El contenido y las actividades se organizan en varios [[Bloque de preparación|bloques ordenados]], cada uno con su propio título, fechas y HTML saneado, para poder autoguardar por partes.

## Campos

| Campo | Tipo | Obligatorio | Reglas |
| --- | --- | --- | --- |
| `id` | BIGINT | Sí | Clave primaria. |
| `mancomunidad_id` | BIGINT | Sí | FK a [[Mancomunidad]]; tenant de la preparación. |
| `profesor_id` | BIGINT | Sí | FK a [[Usuario]]; autor con pertenencia activa en esa mancomunidad. |
| `asignatura_id` | BIGINT | Sí | FK a [[Asignatura]] que imparte, en la misma mancomunidad. |
| `curso_escolar_id` | BIGINT | Sí | FK a [[Curso escolar]] de la misma mancomunidad. |
| `sesion_id` | BIGINT | No | FK a [[Sesión de clase]] de la misma mancomunidad; vínculo opcional. |
| `fecha` | DATE | Sí | Puede ser futura. |
| `titulo` | VARCHAR(150) | Sí | Tema de la clase. |
| `objetivos` | TEXT | No | Qué se quiere conseguir. |
| `recursos` | TEXT | No | Materiales necesarios. |
| `observaciones` | TEXT | No | Notas del profesor. |
| `estado` | VARCHAR(20) | Sí | `BORRADOR`, `PREPARADA` o `IMPARTIDA`. |
| `created_at` | DATETIME | Sí | Auditoría. |
| `updated_at` | DATETIME | Sí | Auditoría. |
| `version` | BIGINT | Sí | Bloqueo optimista; por defecto `0`. |

## Relaciones

- **N:1 con [[Mancomunidad]]**.
- **N:1 con [[Usuario]]**.
- **N:1 con [[Asignatura]]**.
- **N:1 con [[Curso escolar]]**.
- **N:1 con [[Sesión de clase]]** (opcional).
- **1:N con [[Bloque de preparación]]**.

## Reglas

- Profesor, asignatura, curso y sesión deben pertenecer a la misma mancomunidad.
- El profesor solo prepara asignaturas que imparte.
- Si se vincula a una sesión, esta debe ser suya y de la misma asignatura.
- Solo se puede eliminar una preparación en estado `BORRADOR`.
- Las preparaciones `PREPARADA` e `IMPARTIDA` se conservan como histórico. Una preparación `IMPARTIDA` no admite edición de sus bloques.
- La administración puede ver todas, incluidas las de borrador, pero no editarlas.
- No puede haber dos preparaciones para la misma sesión.
- El estado `IMPARTIDA` solo tiene sentido con fecha pasada o presente.
- El contenido ya no vive en la cabecera: se reparte en bloques ordenados de `1` a `N`, sin huecos ni duplicados ([[ADR-041 - Bloques de preparación y guardado incremental]]).
- Cada bloque se autoguarda individualmente; `PUT /preparaciones/{id}` guarda la cabecera y todos los bloques en una transacción como red de seguridad.
- La cabecera tiene `version` y cada bloque tiene la suya. Un desajuste de versión responde `409`; nunca se sobrescribe en silencio.
- `objetivos`, `recursos` y `observaciones` siguen siendo texto plano.
- El HTML de los bloques se sanea en el servidor con lista blanca; se guarda saneado y se vuelve a sanear al renderizar ([[ADR-038 - Contenido enriquecido en preparaciones de clase]]).
- El límite es de 256 KB de HTML **por bloque**; superarlo responde `422`.
- Las tablas deben incluir encabezados y se muestran con scroll horizontal en móvil.

## Dudas

- ¿Se puede duplicar una preparación para otra fecha? Podría ser una mejora posterior.
- ¿Se adjuntan archivos? Fuera de alcance.
- ¿Se permiten imágenes dentro del contenido? No en la v1; si se necesitan, se valorarán como adjuntos en una fase posterior.
- ¿Se permite preparar sin sesión? Sí, con asignatura + fecha.
- ~~¿El contenido es un campo único?~~ **Resuelto: no; la preparación se compone de bloques ordenados con autoguardado** ([[ADR-041 - Bloques de preparación y guardado incremental]]).

## Relacionado

[[Mancomunidad]] · [[Multi-tenencia]] · [[Sesión de clase]] · [[Asignatura]] · [[Usuario]] · [[Bloque de preparación]] · [[Modelo de datos]] · [[API REST]] · [[ADR-038 - Contenido enriquecido en preparaciones de clase]] · [[ADR-041 - Bloques de preparación y guardado incremental]]
