---
tipo: documento
estado: borrador
fase: 10
actualizado: 2026-11-11
tags:
  - proyecto
  - documentacion
  - usuario
---

# Manual de usuario

> [!info] Alcance
> Esqueleto del manual por rol. El contenido detallado se redacta en la Fase 10, junto con la ayuda contextual de la aplicación.

## 1. Profesor

- Iniciar sesión y, si tiene varias mancomunidades, elegir con cuál trabajar.
- Cambiar la contraseña y el email.
- Consultar sus asignaturas y su horario.
- Montar y finalizar su horario en `BORRADOR`.
- Pasar lista de una sesión y editarla después.
- Registrar y consultar observaciones.
- Registrar y corregir las calificaciones de sus asignaturas, con escala numérica o alfanumérica según la asignatura.
- Preparar clases en varios bloques ordenados, con formato básico (negrita, cursiva, listas y tablas) sin imágenes; cada bloque se autoguarda y hay un guardado completo antes de salir.
- Solicitar cambios sobre un horario finalizado y consultar su estado.
- Cancelar una clase no impartida con motivo.
- Solicitar asuntos propios u otras ausencias y consultar su estado.
- Consultar días no lectivos.
- Cerrar sesión y cerrar todas las sesiones si sospecha de un acceso.

## 2. Administrador

- Todo lo del profesor que le corresponda como miembro.
- Gestionar usuarios, municipios, centros, asignaturas, alumnos e instrumentos.
- Montar y finalizar el horario de cualquier profesor de su mancomunidad en `BORRADOR`.
- Reabrir un horario finalizado con motivo.
- Gestionar cursos escolares, días no lectivos (por fecha o rango y con tipo), sustituciones y ausencias del profesorado.
- Definir la escala de calificación y las opciones alfanuméricas de cada asignatura.
- Gestionar las evaluaciones del curso y consultar las calificaciones (solo lectura).
- Asignar a cada municipio un nombre y un color único dentro de la mancomunidad, para reconocer de un vistazo en horarios, calendario y listados en qué pueblo trabaja el profesor.
- Aprobar o rechazar solicitudes de cambio.
- Consultar observaciones, calificaciones, asistencia e historial (solo lectura).
- Consultar y filtrar solicitudes y horarios.

## 3. Superadministrador

- Crear, suspender y reactivar mancomunidades.
- Dar de alta el primer administrador de una mancomunidad.
- Dar soporte a cuentas: búsqueda, reset, activación, desactivación y cambio de rol, con motivo.
- Iniciar un cambio de email con doble confirmación.
- Consultar la auditoría de plataforma.

## 4. Preguntas frecuentes

- ¿Por qué no puedo editar un horario finalizado? Porque está bloqueado; hay que solicitar el cambio o reabrirlo con motivo.
- ¿Por qué no veo el historial de asistencia? Porque es de solo lectura para la administración.
- ¿Qué pasa si un alumno causa baja? Conserva su historial y sale de las listas futuras.
- ¿Cómo recupero mi contraseña? Desde el login, con un enlace de un solo uso enviado al email verificado.
- ¿Cómo pido un día de asuntos propios? Desde «Mis ausencias»; la administración lo aprueba o rechaza y después decide si cubre las clases con un sustituto o las cancela.
- ¿Cómo se registran las vacaciones y los festivos? La administración los da de alta en el calendario por fecha o rango, con tipo `FESTIVO` o `VACACIONES`; no se importan automáticamente.
- ¿Cómo se pone una nota? El profesor abre el cuaderno de notas de la asignatura, elige la evaluación y guarda un número o una opción del desplegable, según la escala que haya definido la administración.
- ¿Puedo cambiar una nota? Sí, el profesor puede corregirla mientras el curso no esté cerrado; la administración solo la consulta.

## Relacionado

[[Roles y permisos]] · [[Requisitos funcionales]] · [[Mapa de pantallas]] · [[Roadmap]] · [[Glosario]] · [[Calificación]] · [[Evaluación]] · [[ADR-039 - Calificaciones por asignatura y escala configurable]]