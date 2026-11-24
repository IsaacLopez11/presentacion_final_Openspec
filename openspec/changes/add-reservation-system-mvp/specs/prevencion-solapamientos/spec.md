# Spec Delta

## Purpose

Establece la invariante central del producto: dos reservas aprobadas del mismo espacio nunca pueden ocupar el mismo instante, incluso si dos aprobaciones ocurren simultáneamente. La garantía reside en la persistencia, no en la buena voluntad de una validación previa.

## ADDED Requirements

### Requirement: Invariante de no solapamiento

El sistema MUST impedir que coexistan dos reservas en estado `aprobada` para el mismo espacio cuyos rangos temporales se intersecten. La verificación SHALL evaluarse por espacio: dos reservas del mismo rango en espacios distintos son válidas.

#### Scenario: Dos reservas aprobadas que se solapan

- **GIVEN** que existe una reserva aprobada del espacio A de 10:00 a 12:00
- **WHEN** se intenta aprobar otra reserva del espacio A de 11:00 a 13:00
- **THEN** el sistema rechaza la segunda aprobación y la reserva permanece sin aprobar

#### Scenario: Mismo horario en espacios distintos

- **GIVEN** que existe una reserva aprobada del espacio A de 10:00 a 12:00
- **WHEN** se aprueba una reserva del espacio B de 10:00 a 12:00
- **THEN** el sistema acepta la aprobación porque los espacios son distintos

#### Scenario: Reservas contiguas sin solapamiento

- **GIVEN** que existe una reserva aprobada del espacio A de 10:00 a 12:00
- **WHEN** se aprueba una reserva del espacio A de 12:00 a 14:00
- **THEN** el sistema acepta la aprobación porque los rangos son contiguos y no se intersectan

### Requirement: Garantía en la capa de persistencia

La prevención de solapamientos MUST estar garantizada por una restricción declarativa de la base de datos que abarque el espacio, el rango temporal y el estado `aprobada`. Una validación previa en la aplicación SHALL considerarse únicamente una mejora del mensaje al usuario y MUST NOT ser la única defensa.

#### Scenario: Rechazo aun cuando la validación previa no detecta el conflicto

- **GIVEN** que la validación previa de disponibilidad no detectó conflictos por un error en la consulta
- **WHEN** se persiste la aprobación que sí se solapa con otra reserva aprobada
- **THEN** la persistencia rechaza la operación y el sistema informa el conflicto

#### Scenario: Rechazo independiente del cliente

- **GIVEN** que un cliente envía directamente una petición de aprobación que produciría solapamiento
- **WHEN** el sistema procesa la petición
- **THEN** la operación no deja dos reservas aprobadas solapadas, con independencia de lo que el cliente haya validado antes

### Requirement: Revalidación de disponibilidad al aprobar

Al aprobar una solicitud, el sistema MUST verificar la disponibilidad del espacio en ese momento, porque la situación pudo cambiar desde la creación de la solicitud. Si existe conflicto, la aprobación MUST fallar y la solicitud MUST permanecer en estado `pendiente`.

#### Scenario: Conflicto surgido después de la solicitud

- **GIVEN** que existe una solicitud pendiente de 10:00 a 12:00 sobre el espacio A
- **AND** que otra reserva del espacio A fue aprobada para 11:00 a 13:00 después de creada la solicitud
- **WHEN** un aprobador intenta aprobar la solicitud pendiente
- **THEN** el sistema rechaza la aprobación por conflicto y la solicitud sigue pendiente

#### Scenario: Conflicto informado con detalle útil

- **GIVEN** que existe un conflicto con una reserva aprobada
- **WHEN** el sistema rechaza la aprobación
- **THEN** informa el rango en conflicto y el espacio afectado, sin exponer datos personales del autor de la reserva existente

### Requirement: Comportamiento bajo concurrencia

Cuando dos o más aprobaciones del mismo espacio y con rangos solapados se procesan simultáneamente, el sistema MUST confirmar exactamente una de ellas y MUST fallar las demás con un error de conflicto. El estado final MUST NOT contener reservas aprobadas solapadas.

#### Scenario: Dos aprobaciones simultáneas del mismo rango

- **GIVEN** que dos solicitudes pendientes del espacio A cubren el mismo rango de 10:00 a 12:00
- **WHEN** ambas aprobaciones se procesan al mismo tiempo
- **THEN** exactamente una queda aprobada, la otra falla por conflicto y una consulta posterior no encuentra solapamientos aprobados

#### Scenario: Aprobaciones simultáneas de rangos parcialmente solapados

- **GIVEN** que una solicitud cubre 10:00 a 12:00 y otra cubre 11:00 a 13:00 sobre el mismo espacio
- **WHEN** ambas aprobaciones se procesan simultáneamente
- **THEN** solo una queda aprobada y el estado final no contiene solapamientos

#### Scenario: Reintento tras conflicto

- **GIVEN** que una aprobación falló por conflicto
- **WHEN** el conflicto se resuelve en el espacio y se reintenta la aprobación
- **THEN** el sistema acepta la aprobación en el nuevo intento
