---
tipo: documento
estado: borrador
fase: 1
actualizado: 2026-11-11
tags:
  - proyecto
  - pruebas
  - datos-demo
---

# Plan de pruebas manuales y datos de demo

> [!info] Alcance
> Seed estable para desarrollo y pruebas manuales. Se implementa en la Fase 1 junto con la migración inicial.

> [!note] Estado del seed (sub-fase 1.3)
> `db/seed/R__datos_demo.sql` ya siembra las dos mancomunidades `demo-a` (`MUSICA`) y `demo-b` (`IDIOMAS`), cada una con su curso `2026-2027` activo, un municipio, un centro y sus cuentas. El profesor `compartido@mancomunidad.local` pertenece a ambas para probar las pertenencias múltiples. El alumnado, los instrumentos, los horarios y las sesiones se añaden en las fases 4–8. La tabla siguiente describe el objetivo completo.

## 1. Datos de demo

Seed con **dos mancomunidades** para poder probar el aislamiento:

| Mancomunidad | Código | Tipo | Centros | Cursos |
| --- | --- | --- | --- | --- |
| Mancomunidad A | `demo-a` | `MUSICA` | 2 | `2026-2027` activo |
| Mancomunidad B | `demo-b` | `IDIOMAS` | 1 | `2026-2027` activo |

Por mancomunidad:

- 1 administrador de prueba (con `admin_plataforma = true` solo en `dev`).
- Varios municipios con colores distintos y únicos dentro de la mancomunidad, con al menos un municipio con dos centros.
- 3 profesores, uno compartido entre A y B para probar el aislamiento de conflictos.
- 5 asignaturas por mancomunidad (al menos una musical instrumental en A y una de idiomas en B).
- 20 alumnos en A y 10 en B, con tutores informados.
- Instrumentos solo en A.
- Horarios en `BORRADOR` y `FINALIZADO`, con franjas solapadas y no solapadas.
- Sesiones `PROGRAMADA`, `ABIERTA` y `CANCELADA`, y al menos un día no lectivo.
- Matrículas activas y alguna en `BAJA` para probar la purga.

## 2. Casos de prueba manual mínimos

- Login con 0, 1 y varias pertenencias.
- Cambio de mancomunidad sin volver a introducir la contraseña.
- Contraseña temporal y cambio obligatorio.
- Recuperación por email y bloqueo por intentos fallidos.
- Alta y edición de usuarios, municipios, centros, asignaturas, alumnos e instrumentos.
- Color único por mancomunidad: repetir un color responde `409` y el horario muestra color y nombre del municipio.
- Preparación por bloques: autoguardar un bloque, reordenar, recargar y recuperar todo con el guardado completo.
- Matrícula, baja y reactivación.
- Montaje, finalización, solicitud, aprobación/rechazo y reapertura de un horario.
- Conflicto de alumno entre grupos distintos.
- Copia de horario al curso siguiente.
- Sustitución de un profesor y reasignación de sesiones.
- Cancelación de una clase y gestión de días no lectivos.
- Cierre y reapertura de curso.
- Purga de historial tras 2 años de baja.
- Exportación lógica por mancomunidad.
- Aislamiento: un usuario de A no ve datos de B cambiando IDs.

## 3. Reglas del seed

- El seed solo existe en el perfil `dev` y `test`.
- Los datos son deterministas y reproducibles: mismos IDs y mismos correos en cada ejecución.
- No se usan datos personales reales; los nombres son ficticios.
- La restauración de una copia de staging debe dejar el seed intacto.
- En `dev` y `test`, la mancomunidad activa se fija con la cabecera `X-Mancomunidad-Id: <id>`; sin cabecera, o con un id desconocido o inactivo, no hay tenant y las consultas de negocio fallan. Es un mecanismo temporal hasta que la Fase 2 lea la mancomunidad del token JWT.

**Credenciales de demo** (cambio obligatorio en el primer acceso):

| Cuenta | Contraseña | Pertenencia |
| --- | --- | --- |
| `admin@mancomunidad.local` | `Admin1234!` | `demo-a`, `ADMINISTRADOR` |
| `profesor@mancomunidad.local` | `Profesor1234!` | `demo-a`, `PROFESOR` |
| `admin-b@mancomunidad.local` | `Admin1234!` | `demo-b`, `ADMINISTRADOR` |
| `profesor-b@mancomunidad.local` | `Profesor1234!` | `demo-b`, `PROFESOR` |
| `compartido@mancomunidad.local` | `Profesor1234!` | `demo-a` y `demo-b`, `PROFESOR` |

## Relacionado

[[Roadmap]] · [[Copias de seguridad y exportación]] · [[Modelo de datos]] · [[Requisitos funcionales]] · [[Mapa de pantallas]] · [[Municipio]] · [[Bloque de preparación]]