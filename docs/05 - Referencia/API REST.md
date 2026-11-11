---
tipo: referencia
estado: activo
fase: 0
actualizado: 2026-11-11
tags:
  - api
  - referencia
---

# API REST

> [!info] Contrato inicial
> Los endpoints pueden ajustarse en cada fase. La fuente definitiva será OpenAPI en la Fase 10. Todas las rutas llevan el prefijo `/api/v1`.
>
> **Multi-tenencia:** salvo login, refresh y selección, todas las peticiones van autenticadas y operan sobre la mancomunidad activa del token. El cliente **no** envía `mancomunidadId` para decidir permisos. Los recursos de otra mancomunidad responden `404`.

## 1. Convenciones

- Todas las rutas de la API se sirven bajo `/api/v1`. En las tablas de este documento se muestran sin el prefijo para no repetirlo.
- JSON en `camelCase`.
- Fechas `YYYY-MM-DD`; timestamps ISO 8601 en UTC.
- Autenticación: `Authorization: Bearer <accessToken>`.
- La mancomunidad activa se deduce del token; se muestra en la cabecera de la interfaz.
- Paginación: `?page=0&size=20&sort=nombre,asc`.
- Respuesta paginada:

```json
{
  "content": [],
  "page": 0,
  "size": 20,
  "totalElements": 0,
  "totalPages": 0
}
```

- Errores con `application/problem+json`:

```json
{
  "type": "https://api.mancomunidad.local/errors/validation",
  "title": "Datos inválidos",
  "status": 400,
  "detail": "El email ya está registrado",
  "instance": "/api/v1/usuarios",
  "errors": {
    "email": "Ya existe un usuario con este email"
  },
  "traceId": "..."
}
```

- `201 Created` con `Location` al crear.
- `204 No Content` al eliminar o al guardar una lista sin cuerpo de respuesta.
- `409 Conflict` para duplicados y solapamientos.
- `403` para permisos; `404` para recursos inexistentes **o de otra mancomunidad**.
- Los recursos compartidos por administración y profesor usan la misma ruta y ajustan el alcance por rol y propiedad; las vistas propias del profesor usan `/mis-...` o `/mios`.
- Swagger UI y el contrato OpenAPI solo se exponen en `dev`; en producción se desactivan.
- `GET /.well-known/jwks.json` publica las claves públicas de firma y queda fuera de `/api/v1` ([[ADR-029 - Claves JWT, rotación y secretos]]).
- Los recursos editables incluyen `version` en las respuestas; las actualizaciones lo envían y, si no coincide con el servidor, responden `409`.
- No se usan ETags HTTP en la v1: el control de concurrencia viaja en el cuerpo con `version`.
- Las escrituras sensibles aceptan o exigen **`Idempotency-Key`** (16–64 caracteres `[A-Za-z0-9_-]`). El reintento con la misma clave y cuerpo devuelve la respuesta guardada; otra ruta o cuerpo responde `409` ([[ADR-033 - Idempotencia de peticiones sensibles]]).
- Los extremos limitados responden `429` con `Retry-After` y, cuando se conoce, `X-RateLimit-Limit`, `X-RateLimit-Remaining` y `X-RateLimit-Reset` ([[ADR-032 - Rate limiting de extremos sensibles]]).
- Los errores de idempotencia usan los códigos `CLAVE_IDEMPOTENCIA_REUTILIZADA` y `OPERACION_EN_CURSO` en el campo `code` del Problem Details.

## 2. Autenticación

| Método | Ruta | Rol | Descripción |
| --- | --- | --- | --- |
| POST | `/auth/login` | Público | Valida credenciales globales. Devuelve tokens directamente si hay una sola opción o un `selectionToken` con la lista si hay varias. |
| POST | `/auth/seleccionar-mancomunidad` | Token de selección | Valida la pertenencia y devuelve access token, refresh y datos de la mancomunidad. |
| POST | `/auth/cambiar-mancomunidad` | Autenticado | Cambia la mancomunidad activa, valida la pertenencia y rota los tokens. |
| POST | `/auth/refresh` | Cookie | Valida el refresh, la pertenencia y el estado de la mancomunidad; rota el token dentro de su familia. Un token ya rotado revoca toda la familia y responde `401`. |
| POST | `/auth/logout` | Autenticado | Revoca la familia de refresh actual y borra la cookie. |
| POST | `/auth/logout-all` | Autenticado | Exige la contraseña actual, revoca todas las familias, incrementa `tokenVersion` y borra la cookie. |
| GET | `/auth/sesiones` | Autenticado | Lista las familias activas con `familiaId`, creación, último uso, IP y `userAgent`; marca la actual. No devuelve tokens. |
| GET | `/auth/me` | Autenticado | Perfil, rol en la mancomunidad activa, scope, estado de verificación del email y lista de pertenencias. |
| PATCH | `/auth/password` | Autenticado o token de cambio | Cambia la contraseña actual; desactiva `debeCambiarPassword`, revoca todas las sesiones e incrementa `tokenVersion`. |
| PATCH | `/auth/email` | Autenticado | Exige la contraseña actual y la dirección nueva; no la aplica hasta confirmar el enlace enviado a la dirección nueva. |
| POST | `/auth/verificar-email` | Público con token | Confirma el alta o el cambio de email; en un cambio revoca todas las sesiones. |
| POST | `/auth/reenviar-verificacion` | Autenticado | Reenvía el enlace de verificación; responde siempre `202` y está limitado a 3 por hora y usuario. |
| POST | `/auth/solicitar-recuperacion` | Público | Envía un enlace de un solo uso al email si la cuenta está activa y verificada. Responde siempre `202`, exista o no la cuenta. |
| POST | `/auth/restablecer-password` | Público con token | Valida el token, guarda la nueva contraseña y revoca todas las sesiones. |

Ejemplo de respuesta de login con varias pertenencias:

```json
{
  "seleccionRequerida": true,
  "selectionToken": "…",
  "opciones": [
    { "mancomunidadId": 1, "nombre": "Mancomunidad A", "rol": "PROFESOR" },
    { "mancomunidadId": 2, "nombre": "Mancomunidad B", "rol": "ADMINISTRADOR" },
    { "mancomunidadId": null, "nombre": "Plataforma", "rol": "SUPERADMIN" }
  ]
}
```

Ejemplo de respuesta con una sola pertenencia:

```json
{
  "seleccionRequerida": false,
  "accessToken": "…",
  "usuario": { "id": 7, "nombre": "Ana", "email": "ana@example.com" },
  "mancomunidad": { "id": 1, "nombre": "Mancomunidad A", "codigo": "a" },
  "rol": "PROFESOR",
  "scope": "TENANT"
}
```

Si la cuenta tiene una contraseña temporal o fue reseteada, el login no emite tokens de negocio:

```json
{
  "debeCambiarPassword": true,
  "passwordChangeToken": "…",
  "usuario": { "id": 7, "nombre": "Ana", "email": "ana@example.com" }
}
```

Reglas del contrato:

- `POST /auth/solicitar-recuperacion` responde `202` con el mismo cuerpo siempre; no revela si el email existe. Solo envía correo si la cuenta está activa y el email verificado ([[ADR-028 - Verificación y cambio de email]]).
- El enlace de recuperación caduca en 60 minutos, es de un solo uso y se invalida al pedir otro.
- `PATCH /auth/password` con un token de cambio no exige la contraseña anterior, pero sí aplica la política; con una sesión normal sí la exige.
- Todo cambio o restablecimiento revoca todas las sesiones e incrementa `tokenVersion`.
- Un login bloqueado por intentos fallidos responde `429` con un mensaje genérico, sin revelar si el email existe.
- `PATCH /auth/email` deja la dirección nueva como pendiente; hasta confirmarla, el login y la recuperación siguen usando la anterior.
- `POST /auth/verificar-email` de un cambio confirmado revoca todas las sesiones; la verificación de alta no las revoca.
- El máximo de familias activas es 5 (`APP_MAX_SESIONES`); al superarlo se desaloja la más antigua por `ultimoUsoAt` y la desalojada recibe `401` en su siguiente refresh.
- `GET /auth/sesiones` y `POST /auth/logout-all` son personales: ningún rol puede listar ni cerrar las sesiones de otro usuario.

## 2.1 Configuración de la mancomunidad

| Método | Ruta | Rol | Descripción |
| --- | --- | --- | --- |
| GET | `/configuracion` | Autenticado | Configuración de la mancomunidad activa: tipo, duración máxima de franja y módulos disponibles. |

Ejemplo de respuesta:

```json
{
  "tipoMancomunidad": "MUSICA",
  "duracionMaximaFranjaMin": 120,
  "modulos": {
    "instrumentos": true
  }
}
```

Reglas del contrato:

- `modulos.instrumentos` es `true` si existe al menos una asignatura activa con `requiereInstrumento = true`; en caso contrario, el frontend oculta el módulo y la API rechaza sus operaciones con `422`.
- El frontend carga la configuración al iniciar sesión y la refresca cuando se crea, edita, activa o desactiva una asignatura.
- `tipoMancomunidad` es informativo y no limita las asignaturas ([[ADR-015 - Tipos de enseñanza y módulo de instrumentos opcional]]).

## 3. Usuarios

| Método | Ruta | Rol | Descripción |
| --- | --- | --- | --- |
| GET | `/usuarios` | Admin | Usuarios con pertenencia en la mancomunidad activa; filtros `q`, `rol`, `activo`. |
| POST | `/usuarios` | Admin | Crea el usuario global si no existe y le asigna una pertenencia con rol en la mancomunidad. |
| GET | `/usuarios/{id}` | Admin | Detalle, solo si tiene pertenencia en la mancomunidad activa. |
| PUT | `/usuarios/{id}` | Admin | Edita datos globales (sin contraseña). |
| PATCH | `/usuarios/{id}/estado` | Admin | Activa/desactiva la **pertenencia** en la mancomunidad activa. |
| PATCH | `/usuarios/{id}/rol` | Admin | Cambia el rol de la pertenencia en la mancomunidad activa. |
| POST | `/usuarios/{id}/reset-password` | Admin | Genera contraseña temporal para un usuario de la mancomunidad. |

> [!note] Un usuario puede existir en varias mancomunidades
> `POST /usuarios` reutiliza la identidad global si el email ya existe, pero **no** revela datos de otras mancomunidades. Solo añade o actualiza la pertenencia de la mancomunidad activa.
>
> El administrador de mancomunidad no puede cambiar el email de un usuario de su tenant. La respuesta incluye `emailVerificado`; si es `false`, el administrador puede resetear la contraseña, pero la recuperación autoservicio no se habilita hasta que el titular confirme la dirección ([[ADR-028 - Verificación y cambio de email]]).

## 3.1 Plataforma

Solo accesible con `scope = PLATFORM` y `admin_plataforma = true`. No expone datos académicos de las mancomunidades.

| Método | Ruta | Rol | Descripción |
| --- | --- | --- | --- |
| GET | `/plataforma/mancomunidades` | Superadmin | Lista con `q`, `activo` y métricas bajo demanda (centros, usuarios y alumnos activos), sin tiempo real. |
| POST | `/plataforma/mancomunidades` | Superadmin | Crea nombre, código, tipo y primer administrador; siembra curso inicial. |
| GET | `/plataforma/mancomunidades/{id}` | Superadmin | Detalle y métricas. |
| PATCH | `/plataforma/mancomunidades/{id}` | Superadmin | Cambia nombre, tipo o duración máxima de franja, con motivo y auditoría. |
| PATCH | `/plataforma/mancomunidades/{id}/estado` | Superadmin | Suspende o reactiva. |
| POST | `/plataforma/mancomunidades/{id}/administradores` | Superadmin | Añade un administrador adicional. |

Reglas del contrato:

- Crear una mancomunidad es una transacción: mancomunidad + curso inicial + primer administrador + pertenencia.
- El código es único, se fija al crear la mancomunidad y no se modifica ni se reutiliza.
- El código es una etiqueta legible: no decide el tenant, no aparece en URLs ni en subdominios y no sirve para autorizar.
- El `PATCH /plataforma/mancomunidades/{id}` exige motivo de 10–500 caracteres y deja traza en la auditoría de plataforma; el `codigo` no se puede cambiar.
- Cambiar el tipo no altera las asignaturas existentes. Si la nueva `duracionMaximaFranjaMin` es menor que una franja ya existente, responde `422`.
- Suspender una mancomunidad impide login y refresh de sus pertenencias, pero no borra datos.
- Un administrador de mancomunidad no puede llamar a estas rutas aunque conozca la URL.
- El panel no permite leer alumnos, horarios ni asistencias.
- Las métricas se calculan al consultar el listado o el detalle; no hay panel en tiempo real en la v1.

## 3.2 Soporte de cuentas

Solo accesible con `scope = PLATFORM` y `admin_plataforma = true`. Administra cuentas de cualquier mancomunidad, nunca datos académicos ([[ADR-012 - Soporte de cuentas del superadministrador]]).

| Método | Ruta | Rol | Descripción |
| --- | --- | --- | --- |
| GET | `/plataforma/usuarios` | Superadmin | Búsqueda global por `q`, `activo`, `rol`, `mancomunidadId`; devuelve pertenencias y estado. |
| GET | `/plataforma/usuarios/{id}` | Superadmin | Detalle global y pertenencias, sin datos académicos. |
| POST | `/plataforma/usuarios/{id}/reset-password` | Superadmin | Genera una contraseña temporal con cambio obligatorio; exige motivo y no devuelve la contraseña. |
| POST | `/plataforma/usuarios/{id}/desbloquear` | Superadmin | Levanta el bloqueo por intentos fallidos; exige motivo y queda auditado. |
| PATCH | `/plataforma/usuarios/{id}/estado` | Superadmin | Activa o desactiva la cuenta global; exige motivo y revoca sesiones. |
| PATCH | `/plataforma/usuarios/{id}/email` | Superadmin | Inicia la corrección del email con motivo; la dirección nueva queda pendiente hasta confirmar el enlace y no se aplica de inmediato. |
| PATCH | `/plataforma/usuarios/{id}/pertenencias/{mancomunidadId}` | Superadmin | Activa o desactiva una pertenencia; exige motivo. |
| PATCH | `/plataforma/usuarios/{id}/pertenencias/{mancomunidadId}/rol` | Superadmin | Cambia el rol entre `PROFESOR` y `ADMINISTRADOR`; exige motivo. |
| GET | `/plataforma/auditoria` | Superadmin | Filtros por actor, afectado, mancomunidad, acción y fechas. |

Reglas del contrato:

- Ninguna ruta de soporte devuelve alumnos, horarios, asistencias, observaciones, preparaciones ni matrículas.
- Toda acción sensible exige `motivo` de 10–500 caracteres y escribe en `auditoria_plataforma`.
- El desbloqueo de una cuenta también exige motivo y auditoría; el reset de contraseña limpia el bloqueo.
- No se puede conceder ni retirar `admin_plataforma` desde estas rutas.
- No se puede desactivar al último superadministrador activo.
- Cambiar el rol o desactivar una pertenencia revoca los refresh tokens del afectado e incrementa `tokenVersion`; el access token deja de valer en menos de 60 segundos.
- El reset no devuelve la contraseña temporal; la genera el servidor y se entrega al titular fuera de banda. El soporte no puede elegirla ni usarla para entrar como el afectado.
- El soporte puede **iniciar** un cambio de email con motivo, pero no confirmarlo: el enlace va a la dirección nueva y la anterior sigue vigente hasta entonces ([[ADR-028 - Verificación y cambio de email]]).
- El soporte no puede listar ni cerrar sesiones de otro usuario; su reset, desactivación o cambio de rol ya revoca las sesiones del afectado.
- Las acciones de soporte están limitadas a 60 por hora y actor ([[ADR-032 - Rate limiting de extremos sensibles]]).
- La impersonación no existe en la v1.

## 3.3 Bootstrap de plataforma

No hay endpoint HTTP. El primer `SUPERADMIN` se crea con el comando `--bootstrap-superadmin` y `APP_BOOTSTRAP_ENABLED=true`; la contraseña se lee de un fichero secreto o de la consola y nunca viaja en la API ([[ADR-034 - Bootstrap del primer superadministrador]]). El comando es idempotente si ya existe un superadministrador activo.

## 3.4 Derechos del interesado (preproducción)

No hay endpoints dedicados en la v1. Se resuelven con las rutas existentes y el soporte:

- **Acceso y rectificación:** usuarios y alumnos se consultan y editan con las rutas de las secciones 3 y 6; el soporte corrige cuentas con motivo.
- **Supresión:** se respeta la retención legal (asistencia, jornada, auditoría); si el dato no se puede borrar, se responde con el motivo y el plazo.
- **Portabilidad:** se cubre con el script de exportación lógica por mancomunidad ([[Copias de seguridad y exportación]] §5), filtrado por `mancomunidad_id`.
- **Limitación y oposición:** se tramitan por la mancomunidad responsable y se registran en la auditoría cuando interviene el soporte.

Los plazos legales (1 mes, prorrogable 2) y el procedimiento están en [[Privacidad y tratamiento de datos]] y [[Copias de seguridad y exportación]] §9.

## 4. Municipios

| Método | Ruta | Rol | Descripción |
| --- | --- | --- | --- |
| GET | `/municipios` | Admin | Lista con `q`, `activo`. |
| POST | `/municipios` | Admin | Crea con nombre y color opcional; si falta, asigna el primero libre de la paleta. |
| GET | `/municipios/{id}` | Admin | Detalle con sus centros. |
| PUT | `/municipios/{id}` | Admin | Edita nombre o color. |
| PATCH | `/municipios/{id}/estado` | Admin | Activa/desactiva con validación de centros activos. |

Reglas del contrato:

- `nombre` y `color` son únicos dentro de la mancomunidad. Un nombre repetido responde `409`; un color repetido responde `409` con el municipio que ya lo usa.
- `color` es `#RRGGBB` y se normaliza a mayúsculas. Si no se envía al crear, el servidor asigna el primero libre de la paleta; si la paleta se agota, responde `422` y exige un color libre.
- No se puede desactivar un municipio con centros activos; responde `409` con la lista de centros que lo impiden.
- Un municipio inactivo no se ofrece para centros nuevos, pero los centros existentes lo conservan.
- El color es del municipio y no del centro: cambiar el color afecta a todos sus centros y el horario lo recalcula al leer ([[ADR-040 - Municipio y color único por municipio]]).
- El municipio no dispara la carga de festivos ni valida tiempos de desplazamiento.

## 4.1 Centros

| Método | Ruta | Rol | Descripción |
| --- | --- | --- | --- |
| GET | `/centros` | Admin | Lista con `q`, `municipioId`, `activo`. |
| POST | `/centros` | Admin | Crea con `municipioId` obligatorio. |
| GET | `/centros/{id}` | Admin | Detalle con su municipio. |
| PUT | `/centros/{id}` | Admin | Edita, incluido el cambio de municipio. |
| PATCH | `/centros/{id}/estado` | Admin | Activa/desactiva con validación de dependencias. |

Reglas del contrato:

- `municipioId` es obligatorio y debe pertenecer a la mancomunidad activa; si no, responde `404`.
- El centro ya no tiene `localidad` ni `color`. La respuesta incluye `municipio: { id, nombre, color }` para que la interfaz pinte el color sin una segunda llamada.
- Cambiar de municipio no reescribe el histórico: las asignaturas y el horario muestran el municipio actual al leer.
- Un centro inactivo conserva su municipio y su histórico.

## 4.2 Cursos escolares

| Método | Ruta | Rol | Descripción |
| --- | --- | --- | --- |
| GET | `/cursos-escolares` | Admin | Lista con `estado`, paginación. |
| GET | `/cursos-escolares/activo` | Autenticado | Curso activo. |
| POST | `/cursos-escolares` | Admin | Crea un curso en `PLANIFICACION`. |
| PUT | `/cursos-escolares/{id}` | Admin | Edita fechas o código mientras esté en `PLANIFICACION`. |
| POST | `/cursos-escolares/{id}/activar` | Admin | Activa el curso y cierra el anterior en una transacción; `409` si el curso está `CERRADO`. |
| POST | `/cursos-escolares/{id}/cerrar` | Admin | Cierra el curso con preflight: cancela sesiones `PROGRAMADA` sin asistencia, congela horarios y matrículas y devuelve `sesionesCanceladas` y `horariosCongelados`. |
| POST | `/cursos-escolares/{id}/reabrir` | Admin | Reapertura excepcional con `motivo` de 10–500 caracteres; si hay otro curso activo, lo cierra en la misma transacción. |

Reglas del contrato:

- `GET /cursos-escolares/activo` devuelve `404` si no hay ninguno activo; el frontend muestra un error de configuración.
- Activar un curso distinto cierra el anterior en la misma transacción; no puede haber dos activos en ningún momento.
- Un curso `CERRADO` debe reabrirse antes de poder activarse.
- Las operaciones de matrícula, horario y preparación validan que el curso esté `ACTIVO`.
- `cerrar` responde `409` si el curso ya está `CERRADO` o si está en `PLANIFICACION`. No borra nada y deja la asistencia de sesiones existentes corregible ([[ADR-024 - Cierre y reapertura de curso escolar]]).
- `reabrir` solo acepta un curso `CERRADO`; no reactiva las sesiones canceladas ni genera sesiones retroactivas. El cuerpo exige `motivo`.

## 4.3 Días no lectivos

| Método | Ruta | Rol | Descripción |
| --- | --- | --- | --- |
| GET | `/dias-no-lectivos` | Admin | Lista con `cursoEscolarId`, `centroId`, `tipo`, `desde`, `hasta`. |
| POST | `/dias-no-lectivos` | Admin | Crea uno o varios días globales (`centroId` nulo) o de un centro, con `tipo`, y cancela las sesiones `PROGRAMADA` sin asistencia de su ámbito. |
| DELETE | `/dias-no-lectivos/{id}` | Admin | Elimina un día si el curso no está `CERRADO`; no reactiva las sesiones ya canceladas. |
| DELETE | `/dias-no-lectivos` | Admin | Elimina un rango completo con `cursoEscolarId`, `centroId`, `desde` y `hasta`; devuelve `diasEliminados` y no reactiva sesiones. |

Reglas del contrato:

- `POST` acepta `fechaInicio` y `fechaFin` (iguales para un solo día), `tipo` obligatorio (`FESTIVO`, `VACACIONES`, `HUELGA`, `OTRO`) y `nombre`. Crea una fila por cada fecha natural del rango en una sola transacción ([[ADR-036 - Calendario con rangos, tipos y origen manual]]).
- `fechaInicio` y `fechaFin` deben pertenecer al curso, `fechaInicio <= fechaFin` y el rango no puede superar 366 días. Es todo o nada: si alguna fecha ya existe para el mismo ámbito, no se crea ninguna y se responde `409` con `fechasDuplicadas`.
- Un día global afecta a todos los centros; uno con `centroId` solo a las asignaturas de ese centro. Un día global y uno de centro para la misma fecha son redundantes y responden `409`.
- La creación devuelve `201` con `diasCreados`, `sesionesCanceladas` y la lista `dias`.
- La baja por rango usa los mismos filtros de ámbito y curso; el borrado no reactiva sesiones y el generador volverá a crear las `PROGRAMADA` futuras en la siguiente ejecución.
- El calendario de un curso `CERRADO` es de solo lectura.
- Solo la administración gestiona el calendario; un profesor recibe `403`. No se importan calendarios oficiales en la v1.

## 5. Asignaturas

| Método | Ruta | Rol | Descripción |
| --- | --- | --- | --- |
| GET | `/asignaturas` | Admin | Lista con `q`, `centroId`, `profesorId`, `tipo`, `requiereInstrumento`, `activo`. |
| POST | `/asignaturas` | Admin | Crea con `tipo` obligatorio, `requiereInstrumento` opcional y `escalaCalificacion` (`NUMERICA` por defecto). |
| GET | `/asignaturas/{id}` | Admin | Detalle con profesores, escala y opciones de calificación. |
| PUT | `/asignaturas/{id}` | Admin | Edita `tipo`, `requiereInstrumento` y la configuración numérica; rechaza con `422` si el indicador no es válido para el tipo o si se intenta cambiar la escala con calificaciones. |
| PATCH | `/asignaturas/{id}/estado` | Admin | Activa/desactiva. |
| PUT | `/asignaturas/{id}/opciones-calificacion` | Admin | Reemplaza la lista ordenada de opciones de una asignatura alfanumérica. |
| POST | `/asignaturas/{id}/profesores` | Admin | Asigna profesor. |
| DELETE | `/asignaturas/{id}/profesores/{profesorId}` | Admin | Retira profesor. |
| GET | `/mis-asignaturas` | Profesor | Asignaturas que imparte. |

Reglas del contrato:

- `tipo` es obligatorio y pertenece a la lista cerrada `MUSICA`, `DANZA`, `ARTES_ESCENICAS`, `IDIOMAS`, `DEPORTES` u `OTRO`; no depende del tipo de la mancomunidad.
- `requiereInstrumento` solo puede ser `true` cuando `tipo = MUSICA`; si no, responde `422`.
- `escalaCalificacion` es `NUMERICA` o `ALFANUMERICA`. En la numérica se aceptan `notaMinima`, `notaMaxima`, `notaAprobado` y `decimales`; en la alfanumérica se ignoran y se envían `opcionesCalificacion`.
- Una asignatura alfanumérica debe tener entre 2 y 20 opciones, con etiqueta y orden únicos y al menos una aprobada. No se borra una opción con calificaciones: se desactiva.
- No se puede cambiar la escala de una asignatura que ya tiene calificaciones; responde `422`.
- El alta, la edición y el cambio de estado de una asignatura pueden alterar `modulos.instrumentos`; el frontend debe volver a pedir `GET /configuracion`.

## 6. Alumnos, instrumentos y matrículas

| Método | Ruta | Rol | Descripción |
| --- | --- | --- | --- |
| GET | `/alumnos` | Admin | Lista con `q`, `centroId`, `instrumentoId`, `activo`. |
| POST | `/alumnos` | Admin | Crea con instrumentos. |
| GET | `/alumnos/{id}` | Admin | Ficha completa. |
| PUT | `/alumnos/{id}` | Admin | Edita. |
| PATCH | `/alumnos/{id}/estado` | Admin | Activa/desactiva. |
| PUT | `/alumnos/{id}/instrumentos` | Admin | Reemplaza la lista de instrumentos. |
| GET | `/alumnos/{id}/matriculas` | Admin | Matrículas por curso. |
| POST | `/alumnos/{id}/matriculas` | Admin | Matricula en una asignatura. |
| PATCH | `/matriculas/{id}/baja` | Admin | Da de baja conservando el histórico. |
| PATCH | `/matriculas/{id}/reactivar` | Admin | Reactiva la matrícula y limpia `bajaAt`, `purgaAt` y `purgadaAt`. |
| GET | `/mis-alumnos` | Profesor | Alumnos de sus asignaturas. |
| GET | `/instrumentos` | Autenticado | Catálogo con `q`, `activo`. |
| POST | `/instrumentos` | Admin | Crea manualmente. |
| PATCH | `/instrumentos/{id}` | Admin | Renombra. |
| POST | `/instrumentos/{id}/fusionar` | Admin | Fusiona con otro. |
| PATCH | `/instrumentos/{id}/estado` | Admin | Activa/desactiva. |

Reglas del contrato:

- Si `modulos.instrumentos` es `false`, todas las rutas de instrumentos y el campo `instrumentos` del alumno responden `422`; los datos existentes no se borran ([[ADR-015 - Tipos de enseñanza y módulo de instrumentos opcional]]).
- El módulo depende de las asignaturas activas, no del tipo de la mancomunidad.
- La baja de matrícula no borra asistencias ni historial; excluye al alumno de las listas que se abran desde `bajaAt` y la reactivación lo devuelve a las listas futuras ([[ADR-026 - Baja de alumno a mitad de curso]]).
- No se puede desactivar un alumno con matrículas `ACTIVA`; responde `409` y exige dar de baja primero.
- El alumno es local a la mancomunidad: no hay búsqueda global de menores ni deduplicación entre tenants.

## 6.1 Calificaciones y evaluaciones

| Método | Ruta | Rol | Descripción |
| --- | --- | --- | --- |
| GET | `/evaluaciones` | Ambos | Lista por `cursoEscolarId`, con `activa`. |
| POST | `/evaluaciones` | Admin | Crea una evaluación del curso. |
| PUT | `/evaluaciones/{id}` | Admin | Edita nombre, orden y fechas. |
| PATCH | `/evaluaciones/{id}/estado` | Admin | Activa/desactiva. |
| GET | `/calificaciones` | Ambos | Filtros `asignaturaId`, `evaluacionId`, `matriculaId`; el profesor solo ve sus asignaturas. |
| POST | `/calificaciones` | Profesor | Crea una calificación. |
| PUT | `/calificaciones/{id}` | Profesor | Corrige una calificación propia. |
| DELETE | `/calificaciones/{id}` | Profesor | Elimina una calificación propia mientras el curso no esté cerrado. |
| PUT | `/calificaciones/lote` | Profesor | Guarda varias calificaciones de una asignatura y evaluación. |
| GET | `/alumnos/{id}/calificaciones` | Ambos | Histórico por curso, asignatura y evaluación; el profesor solo ve sus asignaturas. |

Reglas del contrato:

- Una calificación se guarda por `matriculaId` y `evaluacionId`. En la escala numérica se envía `valorNumerico`; en la alfanumérica, `opcionCalificacionId`. Exactamente uno de los dos, nunca ambos.
- El valor numérico debe estar entre `notaMinima` y `notaMaxima` y respetar `decimales`; si no, responde `422`.
- La opción debe pertenecer a la misma asignatura y estar activa; si no, responde `422`.
- Solo puede existir una calificación por matrícula y evaluación; el duplicado responde `409`.
- El profesor solo califica asignaturas que imparte y alumnos con matrícula `ACTIVA`; la administración solo consulta.
- En un curso `CERRADO` no se crean, editan ni eliminan calificaciones; solo se consultan.
- La baja de matrícula conserva las calificaciones como histórico y no permite añadir nuevas.
- No se calculan medias, boletines ni expedientes en la v1 ([[ADR-039 - Calificaciones por asignatura y escala configurable]]).

## 7. Horarios

El horario se organiza en torno a [[Horario del profesor]]: un registro por profesor y curso con estado `BORRADOR` o `FINALIZADO`. Las franjas se crean dentro de ese horario.

| Método | Ruta | Rol | Descripción |
| --- | --- | --- | --- |
| GET | `/horarios` | Admin | Lista con `curso`, `centroId`, `profesorId`, `dia`, `estado`. |
| GET | `/horarios/semana` | Admin | Rejilla semanal por profesor (`profesorId`) o por grupo (`centroId` + `curso` + `grupo`). |
| GET | `/horarios/mios/semana` | Profesor | Rejilla semanal propia del curso indicado. |
| GET | `/horarios/grupos/semana` | Admin y profesor | Rejilla semanal de un grupo con todas sus asignaturas; el profesor solo si imparte alguna. |
| GET | `/horarios/{id}` | Admin | Detalle del horario de un profesor con estado, franjas y autores. |
| POST | `/horarios` | Admin | Crea el horario `BORRADOR` de un profesor (`profesorId` + `cursoEscolarId`); si ya existe, lo devuelve. |
| POST | `/horarios/{id}/franjas` | Admin | Crea una franja en el horario en `BORRADOR`. |
| PUT | `/horarios/{id}/franjas/{franjaId}` | Admin | Edita una franja en `BORRADOR`. |
| DELETE | `/horarios/{id}/franjas/{franjaId}` | Admin | Elimina una franja en `BORRADOR`. |
| POST | `/horarios/{id}/finalizar` | Admin | Valida y pasa el horario a `FINALIZADO`; registra la transición con el autor. |
| GET | `/horarios/mios` | Profesor | Horario propio del curso indicado, con estado y franjas. |
| POST | `/horarios/mios/franjas` | Profesor | Crea una franja en el horario en `BORRADOR`. |
| PUT | `/horarios/mios/franjas/{franjaId}` | Profesor | Edita una franja en `BORRADOR`. |
| DELETE | `/horarios/mios/franjas/{franjaId}` | Profesor | Elimina una franja en `BORRADOR`. |
| POST | `/horarios/mios/finalizar` | Profesor | Valida y pasa el horario a `FINALIZADO`; registra la transición en el historial. |
| POST | `/horarios/{id}/reabrir` | Admin | Reabre un horario `FINALIZADO`; exige `motivo`, registra la traza y cancela las solicitudes pendientes. |
| GET | `/horarios/{id}/copia` | Admin y profesor titular | Previsualiza la copia al curso destino (`cursoEscolarDestinoId`) con `franjasCopiables` y `franjasOmitidas`; no escribe nada. |
| POST | `/horarios/{id}/copiar` | Admin y profesor titular | Crea un horario `BORRADOR` en el curso destino con las franjas válidas y devuelve `franjasOmitidas`. |
| GET | `/horarios/{id}/historial` | Admin | Traza de creación, finalización, reapertura y copia, con autor, fecha y motivo. |

Reglas del contrato:

- Crear la primera franja crea el horario en `BORRADOR` si no existe.
- El profesor titular y cualquier administrador de la mancomunidad pueden crear, editar, eliminar y finalizar franjas mientras el horario está en `BORRADOR`.
- Las operaciones de escritura devuelven `409` si el horario está `FINALIZADO`.
- `finalizar` devuelve `422` con el detalle de los conflictos si la validación falla.
- La consulta del horario incluye `estado`, `finalizadoAt`, `creadoPor` y `actualizadoPor` para que el frontend decida qué acciones mostrar.
- Las respuestas de horario y franja exponen `creadoPor` y `actualizadoPor`; la edición normal del administrador no exige `motivo`.
- `POST /horarios/{id}/reabrir` solo acepta horarios `FINALIZADO`; en otro estado devuelve `409`. El cuerpo exige `motivo` de 10–500 caracteres.
- La reapertura no modifica franjas, devuelve el horario a `BORRADOR`, registra la transición en el historial y cancela las solicitudes `PENDIENTE` en la misma transacción.
- El historial del horario solo lo consulta la administración; el profesor ve el estado y el motivo de la última reapertura.
- El profesor no puede llamar a `reabrir` ni a `historial` aunque conozca el ID.
- El administrador escribe franjas solo a través de `/horarios/{id}/franjas` y solo si el horario está en `BORRADOR`; no usa las rutas `/horarios/mios/**`.
- Un horario de otra mancomunidad responde `404` en lectura y escritura, también para el administrador.
- La rejilla cubre **lunes a domingo** y devuelve las franjas ordenadas por día y hora de inicio; los días vacíos también forman parte de la vista.
- Cada franja y cada celda de la rejilla incluyen `municipio: { id, nombre, color }`, obtenido del centro de la asignatura. El color siempre va acompañado del nombre y no se guarda en la franja: si cambia el municipio o su color, la siguiente lectura lo refleja ([[ADR-040 - Municipio y color único por municipio]]).
- La vista por profesor muestra solo sus franjas; la vista por grupo muestra **todas las asignaturas** del mismo `centroId + curso + grupo`, con el profesor de cada franja.
- El profesor solo puede consultar la rejilla de un grupo donde imparte alguna asignatura; si no, responde `404`.
- La vista por grupo es siempre de solo lectura, también para el profesor.
- `cursoEscolarId` es opcional: si se omite, se usa el curso activo de la mancomunidad.
- Si se envían `centroId`, `curso` y `grupo`, deben llegar los tres; si falta alguno, responde `400`.
- Crear o editar una franja que comparta al menos un alumno `ACTIVA` con otra franja solapada del mismo curso responde `422` con `conflictos`. La misma validación se repite al finalizar ([[ADR-021 - Validación de conflictos de alumno]]).
- Los conflictos de profesor se validan solo dentro de la mancomunidad activa. Un mismo profesor global puede tener horarios en otra mancomunidad y el sistema no comprueba esos solapamientos ([[ADR-022 - Conflictos de profesor entre mancomunidades]]).
- Al borrar una franja, las sesiones `PROGRAMADA` sin asistencia se eliminan y las que tienen lista se conservan con `franjaId` nulo ([[ADR-020 - Borrado de franjas y conservación de sesiones]]).
- `POST /horarios/{id}/copiar` solo copia franjas cuya asignatura siga activa, cuyo profesor siga asignado y que pertenezcan a la misma mancomunidad. Si el curso destino ya tiene horario para ese profesor, responde `409`; si no queda ninguna franja válida, `422`. No copia sesiones, solicitudes ni historial y registra `COPIA` con `origenHorarioId` ([[ADR-023 - Copia de horario al curso siguiente]]).
- La copia solo se permite si el curso destino está en `PLANIFICACION` o `ACTIVO`; un curso `CERRADO` responde `409`.
- Un horario de un curso `CERRADO` queda congelado: se consulta, pero no se editan franjas ni se finaliza hasta reabrir el curso ([[ADR-024 - Cierre y reapertura de curso escolar]]).

Ejemplo de reapertura:

```json
{
  "motivo": "Corregir el aula de dos franjas antes de publicar el horario"
}
```

## 7.1 Sustituciones

| Método | Ruta | Rol | Descripción |
| --- | --- | --- | --- |
| GET | `/sustituciones` | Admin | Lista con `asignaturaId`, `cursoEscolarId`, `profesorTitularId`, `profesorSustitutoId`, `estado`, `desde`, `hasta`. |
| GET | `/sustituciones/{id}` | Admin | Detalle con periodo, motivo y sesiones afectadas. |
| POST | `/sustituciones` | Admin | Crea una sustitución y reasigna las sesiones `PROGRAMADA` sin lista del periodo. |
| PUT | `/sustituciones/{id}` | Admin | Edita `desde`, `hasta` o `motivo` mientras esté `ACTIVA`. |
| POST | `/sustituciones/{id}/anular` | Admin | Anula con `motivo` de 10–500 caracteres; revierte las sesiones `PROGRAMADA` futuras. |
| GET | `/mis-sustituciones` | Profesor | Sustituciones donde es titular o sustituto. |

Reglas del contrato:

- Solo la administración crea, edita y anula; el profesor recibe `403` en las rutas de escritura.
- El cuerpo de alta incluye `asignaturaId`, `cursoEscolarId`, `profesorTitularId`, `profesorSustitutoId`, `desde`, `hasta` y `motivo`.
- Las fechas son inclusivas, deben caer dentro del curso y no pueden solaparse con otra sustitución `ACTIVA` de la misma asignatura.
- El sustituto debe tener pertenencia activa `PROFESOR`, no puede ser el titular y no puede tener una sustitución o una franja `FINALIZADA` que se solape con las de la asignatura. Los conflictos responden `422` con `conflictos` ([[ADR-025 - Sustituciones y cambio de titular]]).
- La sustitución no cambia el horario del titular ni crea franjas; solo reasigna las sesiones del periodo.
- Las sesiones afectadas devuelven `profesorId` del sustituto y `sustitucionId`; las ya abiertas o cerradas conservan su profesor.
- `anular` exige `motivo` y no borra la fila; las sesiones `PROGRAMADA` futuras vuelven al titular.
- Un curso `CERRADO` no admite altas ni ediciones de sustituciones.

## 8. Solicitudes de horario

| Método | Ruta | Rol | Descripción |
| --- | --- | --- | --- |
| GET | `/solicitudes-horario` | Admin | Lista con `estado`, `profesorId`, `desde`, `hasta`. |
| GET | `/solicitudes-horario/mias` | Profesor | Propias solicitudes. |
| POST | `/solicitudes-horario` | Profesor | Crea solicitud. |
| POST | `/solicitudes-horario/{id}/resolver` | Admin | Aprueba o rechaza. |
| POST | `/solicitudes-horario/{id}/cancelar` | Profesor | Cancela si está pendiente. |

El cuerpo de `resolver` envía `decision` (`APROBAR` o `RECHAZAR`) y `respuestaAdmin` opcional. La respuesta incluye `estado`, `resueltaPor` y `resueltaAt`.

Reglas del contrato:

- No se envía correo ni push en la v1. El profesor ve el resultado en `/solicitudes-horario/mias` y en un contador de solicitudes resueltas recientemente que la interfaz muestra en su panel (SOL-07).
- La aprobación revalida conflictos y aplica el cambio en la misma transacción; si falla, la solicitud queda `RECHAZADA` con el motivo.

## 9. Observaciones

| Método | Ruta | Rol | Descripción |
| --- | --- | --- | --- |
| GET | `/observaciones` | Ambos | Filtros `alumnoId`, `asignaturaId`, `cursoEscolarId`, `tipo`, `desde`, `hasta`. |
| POST | `/observaciones` | Profesor | Crea sobre un alumno propio; `cursoEscolarId` es opcional y se infiere de `fecha`. |
| PUT | `/observaciones/{id}` | Profesor | Edita la propia. |
| DELETE | `/observaciones/{id}` | Profesor | Elimina la propia. |

Reglas del contrato:

- La `fecha` debe caer dentro del curso; si `cursoEscolarId` no llega y no hay un curso único que la contenga, responde `422` ([[ADR-018 - Curso escolar en las observaciones]]).
- La respuesta incluye `cursoEscolarId` y el listado se puede filtrar por él.
- Los cursos `CERRADO` se consultan, pero no admiten observaciones nuevas salvo importación histórica.

## 10. Sesiones y asistencia

| Método | Ruta | Rol | Descripción |
| --- | --- | --- | --- |
| GET | `/sesiones` | Profesor/Admin | Sesiones propias del profesor con `asignaturaId`, `cursoEscolarId`, `fecha`, `estado`. El admin puede filtrar además por `profesorId` y `centroId`. |
| POST | `/sesiones` | Profesor | Abre una sesión puntual y carga alumnos; `cursoEscolarId` es opcional y por defecto se usa el activo. |
| GET | `/sesiones/{id}` | Profesor/Admin | Detalle con asistencias y datos de cancelación si aplica. |
| PUT | `/sesiones/{id}` | Profesor | Edita hora, estado u observaciones; no admite una sesión `CANCELADA`. |
| POST | `/sesiones/{id}/cancelar` | Profesor/Admin | Cancela una sesión `PROGRAMADA` con `motivo`; `409` si ya está `ABIERTA`, `CERRADA` o `CANCELADA`. |
| POST | `/sesiones/{id}/asistencias` | Profesor | Guarda la lista completa; registra el historial; `409` si la sesión está `CANCELADA`. |
| PATCH | `/sesiones/{id}/asistencias/{asistenciaId}` | Profesor | Corrige una asistencia en cualquier momento; registra el historial; `409` si la sesión está `CANCELADA`. |
| GET | `/asistencias` | Ambos | Filtros `alumnoId`, `asignaturaId`, `cursoEscolarId`, `desde`, `hasta`. |
| GET | `/asistencias/resumen` | Ambos | Contadores por alumno, asignatura y curso; `historico=true` incluye los del [[Resumen de asistencia]] ya purgado. |
| GET | `/asistencias/{id}/historial` | Admin | Historial de cambios de una asistencia. |
| GET | `/asistencias/historial` | Admin | Historial filtrado por `alumnoId`, `sesionId`, `profesorId`, `desde`, `hasta`. |

Reglas del contrato:

- `POST /sesiones/{id}/cancelar` exige `motivo` de 10–500 caracteres; guarda `canceladaPor` y `canceladaAt`. El profesor solo cancela sus propias sesiones; el administrador puede cancelar cualquier sesión `PROGRAMADA` de su mancomunidad y la acción queda auditada.
- Una sesión `CANCELADA` no admite guardar ni corregir asistencias: responde `409`. Tampoco aparece en la bandeja de pendientes ni cuenta en los contadores.
- Si se intenta cancelar una sesión `ABIERTA` o `CERRADA`, responde `409`; la incidencia se resuelve cerrando la sesión y anotándola en `observaciones` ([[ADR-017 - Cancelación de sesiones y días no lectivos]]).
- Las sesiones generadas se filtran por `cursoEscolarId` y por `estado`; los listados del profesor nunca incluyen sesiones de otro profesor.
- Si la sesión cae dentro de una sustitución `ACTIVA`, su `profesorId` es el del sustituto y devuelve `sustitucionId`; el titular puede consultarla, pero no abrirla ni pasar lista ([[ADR-025 - Sustituciones y cambio de titular]]).
- Al abrir una sesión solo se cargan alumnos con matrícula `ACTIVA` en esa fecha. Los alumnos con baja anterior no aparecen en la lista; las sesiones ya abiertas conservan sus filas ([[ADR-026 - Baja de alumno a mitad de curso]]).
- En un curso `CERRADO` no se crean ni abren sesiones nuevas; la corrección de asistencias de sesiones existentes sigue permitida ([[ADR-024 - Cierre y reapertura de curso escolar]]).
- `GET /asistencias/resumen` calcula los contadores en vivo desde `asistencia`. Con `historico=true` añade los agregados anonimizados de [[Resumen de asistencia]] y devuelve `historico: true` cuando el alumno solicitado ya fue purgado ([[ADR-019 - Resumen de asistencia y purga anonimizada]]).
- El resumen no guarda `alumnoId`; el servicio calcula el `alumnoHash` del alumno solicitado para localizarlo.

Ejemplo de guardado masivo:

```json
{
  "asistencias": [
    { "alumnoId": 1, "estado": "PRESENTE" },
    { "alumnoId": 2, "estado": "AUSENTE", "motivo": "Enfermedad" },
    { "alumnoId": 3, "estado": "RETRASO", "minutosRetraso": 10 },
    { "alumnoId": 4, "estado": "AUSENCIA_JUSTIFICADA", "motivo": "Cita médica" }
  ]
}
```

Ejemplo de entrada del historial:

```json
{
  "id": 91,
  "asistenciaId": 12,
  "estadoAnterior": "AUSENTE",
  "estadoNuevo": "AUSENCIA_JUSTIFICADA",
  "minutosRetrasoAnterior": null,
  "minutosRetrasoNuevo": null,
  "motivoAnterior": null,
  "motivoNuevo": "Justificante entregado",
  "cambiadoPor": { "id": 3, "nombre": "Ana" },
  "cambiadoAt": "2026-10-27T18:42:00Z",
  "origen": "CORRECCION"
}
```

> [!warning] Visibilidad
> El historial solo se expone a `ADMINISTRADOR`. Un `PROFESOR` recibe `403` aunque sea el autor del cambio. La administración no dispone de endpoints de escritura sobre asistencias, salvo la cancelación de una sesión `PROGRAMADA`, que queda auditada.
>
> La purga por retención (matrícula + 2 años) no se expone en la API: la ejecuta un proceso programado por mancomunidad y conserva los contadores anonimizados de [[Resumen de asistencia]].

## 10.1 Ausencias del profesor

| Método | Ruta | Rol | Descripción |
| --- | --- | --- | --- |
| GET | `/ausencias` | Profesor/Admin | El profesor ve las suyas; la administración todas, con filtros `profesorId`, `cursoEscolarId`, `tipo`, `estado`, `desde`, `hasta`. |
| GET | `/ausencias/{id}` | Profesor/Admin | Detalle con traza de solicitud y resolución; `404` si es de otro profesor o de otra mancomunidad. |
| POST | `/ausencias` | Profesor/Admin | El profesor crea una ausencia propia; la administración, una de cualquier profesor. Nace `PENDIENTE`. |
| POST | `/ausencias/{id}/aprobar` | Admin | Aprueba con `respuestaAdmin` opcional; devuelve `sesionesAfectadas`. |
| POST | `/ausencias/{id}/rechazar` | Admin | Rechaza con `respuestaAdmin` de 10–500 caracteres. |
| POST | `/ausencias/{id}/cancelar` | Profesor/Admin | El profesor cancela su propia ausencia `PENDIENTE`; la administración cancela una `PENDIENTE` o `APROBADA` futura con `motivo`. |

Reglas del contrato:

- `tipo` pertenece a `ASUNTOS_PROPIOS`, `PERMISO_OFICIAL`, `FORMACION`, `HUELGA` u `OTRO`; `fechaInicio` y `fechaFin` deben estar dentro del curso, `fechaInicio <= fechaFin` y el rango no supera 366 días.
- No puede haber dos ausencias `PENDIENTE` o `APROBADA` solapadas para el mismo profesor: responde `409`.
- El profesor solo opera sobre sus propias ausencias y solo puede cancelar las `PENDIENTE`. La administración resuelve y cancela las de cualquier profesor de su mancomunidad.
- Aprobar una ausencia **no** cancela ni reasigna sesiones: devuelve `sesionesAfectadas` y la administración decide después entre una [[Sustitución]] o la cancelación de las sesiones `PROGRAMADA` ([[ADR-037 - Ausencias del profesor y huelgas]]).
- Una huelga total se registra como día no lectivo de tipo `HUELGA`; una parcial, como ausencia de tipo `HUELGA` creada por la administración.
- En un curso `CERRADO` no se crean ausencias nuevas; las existentes se consultan.
- `respuestaAdmin` y `motivoCancelacion` se guardan con autor y fecha; no se admiten diagnósticos ni datos de salud en `motivo`.

## 11. Preparaciones

| Método | Ruta | Rol | Descripción |
| --- | --- | --- | --- |
| GET | `/preparaciones` | Ambos | Filtros `asignaturaId`, `estado`, `desde`, `hasta`. |
| POST | `/preparaciones` | Profesor | Crea la cabecera, con bloques opcionales. |
| GET | `/preparaciones/{id}` | Ambos | Detalle con la cabecera y los bloques ordenados. |
| PUT | `/preparaciones/{id}` | Profesor | Guardado completo de cabecera y bloques en una transacción. |
| DELETE | `/preparaciones/{id}` | Profesor | Solo en `BORRADOR`; elimina también sus bloques. |
| POST | `/preparaciones/{id}/bloques` | Profesor | Crea y autoguarda un bloque. |
| PUT | `/preparaciones/{id}/bloques/{bloqueId}` | Profesor | Autoguarda un bloque. |
| DELETE | `/preparaciones/{id}/bloques/{bloqueId}` | Profesor | Elimina un bloque y compacta el orden. |
| PUT | `/preparaciones/{id}/bloques/orden` | Profesor | Reordena la lista completa de bloques. |

Reglas del contrato:

- La preparación es la cabecera y ya no tiene `contenido`. El contenido vive en bloques ordenados de `1` a `N`, sin huecos ni duplicados; una preparación admite como máximo **100 bloques** ([[ADR-041 - Bloques de preparación y guardado incremental]]).
- Cada bloque tiene `titulo` y `contenido` obligatorios, y `fechaInicio` y `fechaFin` opcionales. Si ambas fechas vienen, `fechaFin >= fechaInicio` y deben caer dentro del curso de la preparación.
- `POST` y `PUT` de un bloque son el **autoguardado**: actualizan solo ese bloque y su `version`. Editar un bloque no reenvía la preparación completa.
- `PUT /preparaciones/{id}` es el **guardado completo**: escribe cabecera y bloques en una transacción, con la `version` de la preparación, y devuelve el recurso completo. Es la red de seguridad antes de salir o recargar.
- Añadir, borrar o reordenar bloques incrementa la `version` de la preparación. Reordenar exige la lista completa de `bloqueIds` y la `version` de la preparación.
- Un desajuste de `version` de la preparación o de un bloque responde `409`; el autoguardado devuelve las versiones actuales para que la interfaz resuelva el conflicto sin sobrescribir.
- `contenido` es HTML enriquecido saneado. Se admiten párrafos, negrita, cursiva, encabezados, listas, enlaces, citas, código y tablas; no imágenes, iframes, scripts ni estilos libres.
- El servidor sanea con lista blanca antes de guardar y vuelve a sanear al devolver el recurso. Una etiqueta no permitida se elimina; si el resultado queda vacío, responde `422`.
- El límite es de **256 KB de HTML por bloque**; superarlo responde `422`.
- `objetivos`, `recursos` y `observaciones` son texto plano.
- Una preparación `IMPARTIDA` se consulta, pero no admite cambios en sus bloques.

## 12. Catálogo de errores

| Código | Uso |
| --- | --- |
| 400 | Validación o petición mal formada. |
| 401 | Sin token, token inválido, credenciales incorrectas, refresh revocado o reutilizado. |
| 403 | Autenticado sin permiso. |
| 404 | Recurso inexistente o no visible. |
| 409 | Duplicado, solapamiento, estado incompatible, `version` desactualizada, clave de idempotencia reutilizada u operación en curso. |
| 422 | Regla de negocio incumplida. |
| 429 | Demasiadas peticiones, login bloqueado por intentos fallidos o cubo de rate limiting agotado; incluye `Retry-After`. |
| 500 | Error no controlado. |

## 13. Ejemplos

Los pares de petición y respuesta por módulo, el aislamiento entre mancomunidades y el conflicto de `version` están en [[Ejemplos de API]]. Los ejemplos de autenticación, configuración, asistencia y reapertura viven en las secciones anteriores de esta nota.

## Relacionado

[[Arquitectura general]] · [[Multi-tenencia]] · [[Seguridad y autenticación]] · [[Requisitos funcionales]] · [[Convenciones]] · [[Modelo de datos]] · [[Ejemplos de API]] · [[Día no lectivo]] · [[Ausencia del profesor]] · [[Asignatura]] · [[Opción de calificación]] · [[Evaluación]] · [[Calificación]] · [[Resumen de asistencia]] · [[Sustitución]] · [[ADR-003 - JWT con refresh token]] · [[ADR-009 - Multi-tenant con mancomunidad e identidad global]] · [[ADR-011 - Recuperación de contraseña y correo transaccional]] · [[ADR-012 - Soporte de cuentas del superadministrador]] · [[ADR-014 - Bloqueo temporal de cuentas por intentos fallidos]] · [[ADR-015 - Tipos de enseñanza y módulo de instrumentos opcional]] · [[ADR-017 - Cancelación de sesiones y días no lectivos]] · [[ADR-018 - Curso escolar en las observaciones]] · [[ADR-019 - Resumen de asistencia y purga anonimizada]] · [[ADR-020 - Borrado de franjas y conservación de sesiones]] · [[ADR-021 - Validación de conflictos de alumno]] · [[ADR-022 - Conflictos de profesor entre mancomunidades]] · [[ADR-023 - Copia de horario al curso siguiente]] · [[ADR-024 - Cierre y reapertura de curso escolar]] · [[ADR-025 - Sustituciones y cambio de titular]] · [[ADR-026 - Baja de alumno a mitad de curso]] · [[ADR-027 - MFA y riesgo aceptado del superadministrador]] · [[ADR-028 - Verificación y cambio de email]] · [[ADR-029 - Claves JWT, rotación y secretos]] · [[ADR-030 - Detección de reutilización de refresh tokens]] · [[ADR-031 - Límite de sesiones concurrentes y cierre global]] · [[ADR-032 - Rate limiting de extremos sensibles]] · [[ADR-033 - Idempotencia de peticiones sensibles]] · [[ADR-034 - Bootstrap del primer superadministrador]] · [[ADR-036 - Calendario con rangos, tipos y origen manual]] · [[ADR-037 - Ausencias del profesor y huelgas]] · [[ADR-038 - Contenido enriquecido en preparaciones de clase]] · [[ADR-039 - Calificaciones por asignatura y escala configurable]]
