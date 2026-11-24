# Spec Delta

## Purpose

Define cuándo y por quién puede cancelarse una solicitud o una reserva, y cómo la cancelación libera disponibilidad para otros usuarios sin eliminar el registro histórico.

## ADDED Requirements

### Requirement: Cancelación de solicitudes pendientes por su autor

El sistema SHALL permitir al solicitante cancelar su propia solicitud mientras esté en estado `pendiente`, sin restricción de plazo. La solicitud SHALL pasar a estado `cancelada` y SHALL conservarse en el historial.

#### Scenario: Cancelación de una solicitud pendiente

- **GIVEN** que el solicitante tiene una solicitud en estado `pendiente`
- **WHEN** la cancela
- **THEN** la solicitud pasa a estado `cancelada` y permanece visible en su historial

#### Scenario: Cancelación de una solicitud ya aprobada fuera de plazo

- **GIVEN** que el solicitante tiene una reserva aprobada que inicia en 10 horas
- **WHEN** intenta cancelarla
- **THEN** el sistema rechaza la operación porque falta menos de 24 horas para el inicio

#### Scenario: Cancelación de una solicitud ya rechazada

- **GIVEN** que una solicitud está en estado `rechazada`
- **WHEN** el solicitante intenta cancelarla
- **THEN** el sistema rechaza la operación por estado inválido

### Requirement: Cancelación de reservas aprobadas según plazo

El sistema SHALL permitir al solicitante cancelar su reserva en estado `aprobada` únicamente cuando falten al menos 24 horas para la hora de inicio. El plazo SHALL calcularse sobre instantes absolutos.

#### Scenario: Cancelación dentro del plazo permitido

- **GIVEN** que una reserva aprobada inicia en 48 horas
- **WHEN** el solicitante la cancela
- **THEN** la reserva pasa a estado `cancelada`

#### Scenario: Cancelación justo en el límite del plazo

- **GIVEN** que una reserva aprobada inicia exactamente en 24 horas
- **WHEN** el solicitante la cancela
- **THEN** el sistema acepta la cancelación

#### Scenario: Cancelación de una reserva ya iniciada

- **GIVEN** que una reserva aprobada ya comenzó
- **WHEN** el solicitante intenta cancelarla
- **THEN** el sistema rechaza la operación por estado y plazo inválidos

### Requirement: Cancelación administrativa sin restricción de plazo

El sistema SHALL permitir al administrador cancelar cualquier solicitud o reserva no terminal, sin restricción de plazo y sin exigir que sea su autor.

#### Scenario: Administrador cancela una reserva en curso

- **GIVEN** que una reserva aprobada ya comenzó
- **WHEN** un administrador la cancela
- **THEN** la reserva pasa a estado `cancelada`

#### Scenario: Administrador cancela la reserva de otro usuario

- **GIVEN** que una reserva aprobada pertenece a otro usuario
- **WHEN** un administrador la cancela
- **THEN** el sistema acepta la operación y registra al administrador como responsable

### Requirement: La cancelación libera disponibilidad

Al cancelarse una reserva aprobada, el sistema MUST liberar esa franja de inmediato: la franja SHALL dejar de bloquear disponibilidad y SHALL poder aprobarse una nueva reserva en ese rango.

#### Scenario: Franja liberada tras cancelar

- **GIVEN** que una reserva aprobada ocupa 10:00 a 12:00 del espacio A
- **WHEN** la reserva se cancela
- **THEN** una nueva solicitud de 10:00 a 12:00 del espacio A puede aprobarse

#### Scenario: Consulta de disponibilidad actualizada

- **GIVEN** que una reserva aprobada fue cancelada
- **WHEN** un usuario consulta la disponibilidad de ese espacio
- **THEN** el rango cancelado aparece como disponible

### Requirement: Cancelación ajena denegada

El sistema MUST impedir que un usuario cancele solicitudes o reservas de las que no es autor, salvo que tenga el rol de administrador, y MUST NOT revelar la existencia de reservas ajenas al denegar.

#### Scenario: Solicitante cancela la reserva de otro

- **GIVEN** que una reserva pertenece a otro solicitante
- **WHEN** un solicitante intenta cancelarla
- **THEN** el sistema rechaza la operación con un error de recurso no encontrado o de permisos, sin alterar la reserva

#### Scenario: Aprobador sin permiso de cancelación ajena

- **GIVEN** que el usuario tiene únicamente el rol de aprobador
- **WHEN** intenta cancelar la reserva de otra persona
- **THEN** el sistema rechaza la operación con un error de permisos

### Requirement: Motivo de cancelación

El sistema SHALL aceptar un motivo opcional en la cancelación y SHALL registrarlo cuando se proporcione. Cuando el administrador cancela sin indicar motivo, el sistema SHALL registrar que la cancelación fue administrativa y sin motivo.

#### Scenario: Cancelación con motivo

- **GIVEN** que un solicitante cancela su reserva indicando un motivo
- **WHEN** se consulta el historial
- **THEN** el motivo aparece asociado a la cancelación

#### Scenario: Cancelación administrativa sin motivo

- **GIVEN** que un administrador cancela una reserva sin indicar motivo
- **WHEN** se consulta el historial
- **THEN** el registro indica que la cancelación fue administrativa y no presenta un motivo
