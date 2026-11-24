# Spec Delta

## Purpose

Define cómo un solicitante crea una solicitud de reserva sobre un espacio académico, qué validaciones debe superar para ser aceptada y en qué estado nace, garantizando que una solicitud nunca se registre como si ya estuviera aprobada.

## ADDED Requirements

### Requirement: Creación de solicitudes en estado pendiente

El sistema SHALL permitir a un usuario con permiso de solicitud crear una solicitud de reserva indicando espacio, fecha, hora de inicio y hora de fin. Toda solicitud creada SHALL quedar en estado `pendiente`, con el usuario autenticado como autor y la fecha de creación registrada.

#### Scenario: Solicitud creada correctamente

- **GIVEN** que un solicitante autenticado elige un espacio activo y un rango válido dentro del horario operativo
- **WHEN** crea la solicitud
- **THEN** el sistema la registra en estado `pendiente`, con el solicitante como autor

#### Scenario: Estado impuesto por el cliente

- **WHEN** un solicitante envía una petición de creación que incluye el estado `aprobada`
- **THEN** el sistema ignora ese valor y registra la solicitud en estado `pendiente`

#### Scenario: Autor impuesto por el cliente

- **WHEN** un solicitante envía una petición de creación indicando a otro usuario como autor
- **THEN** el sistema registra como autor al usuario autenticado y no al indicado

### Requirement: Validación del rango temporal

El sistema MUST rechazar toda solicitud cuyo fin no sea posterior al inicio, cuya duración sea inferior a 30 minutos, o cuyos instantes no respeten una granularidad de 15 minutos. El rango SHALL interpretarse como semiabierto: la hora de fin es el instante en que el espacio queda libre.

#### Scenario: Fin anterior o igual al inicio

- **WHEN** un solicitante envía una solicitud con hora de fin anterior o igual a la hora de inicio
- **THEN** el sistema rechaza la solicitud con un error de validación y no la registra

#### Scenario: Duración menor que el mínimo

- **WHEN** un solicitante envía una solicitud de 15 minutos de duración
- **THEN** el sistema rechaza la solicitud por no alcanzar la duración mínima de 30 minutos

#### Scenario: Granularidad distinta de 15 minutos

- **WHEN** un solicitante envía una solicitud que inicia a las 10:07
- **THEN** el sistema rechaza la solicitud por no respetar la granularidad de 15 minutos

#### Scenario: Límites exactos aceptados

- **WHEN** un solicitante envía una solicitud de exactamente 30 minutos que inicia en un múltiplo de 15 minutos
- **THEN** el sistema acepta la solicitud

### Requirement: Validación de anticipación

El sistema MUST rechazar solicitudes cuyo inicio esté a menos de 24 horas del momento de la creación, y también aquellas cuyo inicio esté a más de 90 días. La comparación SHALL realizarse sobre instantes absolutos, no sobre texto de fecha.

#### Scenario: Solicitud con menos de la anticipación mínima

- **GIVEN** que son las 08:00 del día actual
- **WHEN** un solicitante solicita un espacio para el mismo día a las 15:00
- **THEN** el sistema rechaza la solicitud por incumplir la anticipación mínima de 24 horas

#### Scenario: Solicitud justo en el límite mínimo

- **GIVEN** que son las 08:00 del día actual
- **WHEN** un solicitante solicita un espacio para el día siguiente a las 08:00
- **THEN** el sistema acepta la solicitud

#### Scenario: Solicitud más allá del límite máximo

- **WHEN** un solicitante solicita un espacio para dentro de 120 días
- **THEN** el sistema rechaza la solicitud por superar la anticipación máxima de 90 días

#### Scenario: Solicitud en el pasado

- **WHEN** un solicitante solicita un rango cuyo inicio ya ocurrió
- **THEN** el sistema rechaza la solicitud con un error de validación

### Requirement: Validación del horario operativo

El sistema SHALL aceptar solicitudes únicamente dentro del horario operativo institucional expresado en la zona horaria `America/Bogota`: de lunes a viernes entre 06:00 y 22:00, y sábados entre 07:00 y 18:00. Las solicitudes de domingo, de días festivos o que excedan el horario del día SHALL rechazarse.

#### Scenario: Solicitud dentro del horario de un día hábil

- **WHEN** un solicitante solicita un martes de 08:00 a 10:00
- **THEN** el sistema acepta la solicitud

#### Scenario: Solicitud que excede el cierre

- **WHEN** un solicitante solicita un miércoles de 21:00 a 23:00
- **THEN** el sistema rechaza la solicitud por exceder el horario operativo

#### Scenario: Solicitud en domingo

- **WHEN** un solicitante solicita un domingo en cualquier horario
- **THEN** el sistema rechaza la solicitud por ser un día no habilitado

#### Scenario: Solicitud en sábado fuera de horario

- **WHEN** un solicitante solicita un sábado de 18:00 a 19:00
- **THEN** el sistema rechaza la solicitud por exceder el horario del sábado

### Requirement: Zona horaria consistente en la interpretación de fechas

El sistema SHALL interpretar las fechas y horas recibidas en la zona horaria `America/Bogota` y MUST almacenar los instantes en UTC. La representación devuelta al usuario SHALL expresarse nuevamente en `America/Bogota`.

#### Scenario: Misma hora mostrada al crear y consultar

- **WHEN** un solicitante crea una solicitud para las 10:00 y luego la consulta
- **THEN** el sistema muestra la solicitud con hora de inicio 10:00 en hora local

#### Scenario: Almacenamiento en UTC

- **GIVEN** que una solicitud se registra para las 10:00 hora local de Bogotá
- **WHEN** se inspecciona el instante almacenado
- **THEN** corresponde a las 15:00 UTC del mismo día

### Requirement: Solo espacios activos admiten solicitudes

El sistema MUST rechazar una solicitud cuyo espacio no exista o esté inactivo, sin revelar información adicional sobre espacios que el usuario no puede reservar.

#### Scenario: Espacio inexistente

- **WHEN** un solicitante envía una solicitud con un identificador de espacio que no existe
- **THEN** el sistema rechaza la solicitud con un error de recurso no encontrado

#### Scenario: Espacio inactivo

- **GIVEN** que un espacio está marcado como inactivo
- **WHEN** un solicitante envía una solicitud sobre ese espacio
- **THEN** el sistema rechaza la solicitud e indica que el espacio no admite reservas
