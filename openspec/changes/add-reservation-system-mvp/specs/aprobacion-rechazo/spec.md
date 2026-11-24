# Spec Delta

## Purpose

Define cómo un aprobador revisa las solicitudes pendientes y decide aprobarlas o rechazarlas, con motivo obligatorio al rechazar y sin posibilidad de resolver sus propias solicitudes.

## ADDED Requirements

### Requirement: Bandeja de solicitudes pendientes

El sistema SHALL permitir al aprobador consultar las solicitudes en estado `pendiente`, con espacio, rango solicitado y fecha de creación, y ordenadas por fecha de creación ascendente.

#### Scenario: Consulta de pendientes

- **GIVEN** que existen solicitudes en estado `pendiente` de varios solicitantes
- **WHEN** un aprobador consulta la bandeja de pendientes
- **THEN** el sistema devuelve esas solicitudes ordenadas por fecha de creación ascendente

#### Scenario: Pendientes no incluye otros estados

- **GIVEN** que existen solicitudes aprobadas, rechazadas y canceladas
- **WHEN** un aprobador consulta la bandeja de pendientes
- **THEN** el sistema no incluye ninguna de ellas en el resultado

#### Scenario: Solicitante sin permiso de bandeja

- **GIVEN** que el usuario tiene únicamente el rol de solicitante
- **WHEN** intenta consultar la bandeja de pendientes
- **THEN** el sistema rechaza la operación con un error de permisos

### Requirement: Aprobación de solicitudes

El sistema SHALL permitir al aprobador aprobar una solicitud en estado `pendiente`, siempre que se supere la revalidación de disponibilidad. La aprobación SHALL registrar el aprobador y el instante de la decisión.

#### Scenario: Aprobación exitosa

- **GIVEN** que existe una solicitud pendiente de otro usuario y el espacio está libre en ese rango
- **WHEN** un aprobador la aprueba
- **THEN** la solicitud pasa a estado `aprobada` y queda registrado el aprobador y la fecha de decisión

#### Scenario: Aprobación bloqueada por conflicto

- **GIVEN** que existe una solicitud pendiente cuyo rango ya fue aprobado para el mismo espacio
- **WHEN** un aprobador intenta aprobarla
- **THEN** el sistema rechaza la operación por conflicto y la solicitud permanece `pendiente`

#### Scenario: Aprobación de una solicitud que ya no está pendiente

- **GIVEN** que una solicitud fue aprobada previamente
- **WHEN** un aprobador intenta aprobarla de nuevo
- **THEN** el sistema rechaza la operación por estado inválido y no duplica la decisión

### Requirement: Prohibición de autoaprobación

El sistema MUST impedir que un usuario apruebe o rechace una solicitud de la que es autor, incluso si tiene el rol de aprobador.

#### Scenario: Aprobador autor de la solicitud

- **GIVEN** que un usuario con roles de solicitante y aprobador creó una solicitud
- **WHEN** intenta aprobar su propia solicitud
- **THEN** el sistema rechaza la operación por autoaprobación y la solicitud permanece `pendiente`

#### Scenario: Rechazo de la propia solicitud

- **GIVEN** que un aprobador es autor de una solicitud pendiente
- **WHEN** intenta rechazarla
- **THEN** el sistema rechaza la operación y la solicitud permanece `pendiente`

#### Scenario: Administrador autor de la solicitud

- **GIVEN** que un administrador es autor de una solicitud pendiente
- **WHEN** intenta aprobarla o rechazarla
- **THEN** el sistema rechaza la operación por autoaprobación y la solicitud permanece `pendiente`

#### Scenario: Otro aprobador sí puede decidir

- **GIVEN** que el autor de una solicitud es un aprobador
- **WHEN** un aprobador distinto decide sobre ella
- **THEN** el sistema acepta la operación

### Requirement: Rechazo con motivo obligatorio

El sistema MUST exigir un motivo no vacío para rechazar una solicitud, y SHALL registrar el motivo junto con el aprobador y el instante del rechazo. Una solicitud rechazada es terminal y MUST NOT volver a estado pendiente en el mismo registro.

#### Scenario: Rechazo con motivo

- **GIVEN** que existe una solicitud pendiente
- **WHEN** un aprobador la rechaza indicando un motivo
- **THEN** la solicitud pasa a estado `rechazada` y el motivo queda registrado

#### Scenario: Rechazo sin motivo

- **WHEN** un aprobador intenta rechazar una solicitud sin indicar motivo o con un motivo vacío
- **THEN** el sistema rechaza la operación y la solicitud permanece `pendiente`

#### Scenario: Rechazo de una solicitud ya decidida

- **GIVEN** que una solicitud ya fue rechazada
- **WHEN** un aprobador intenta rechazarla nuevamente
- **THEN** el sistema rechaza la operación por estado inválido

### Requirement: Autorización de la decisión

El sistema SHALL permitir aprobar y rechazar únicamente a usuarios con rol de aprobador o de administrador, y MUST NOT permitir que el rol se tome de datos enviados por el cliente.

#### Scenario: Solicitante intenta aprobar

- **GIVEN** que el usuario tiene únicamente el rol de solicitante
- **WHEN** intenta aprobar una solicitud pendiente
- **THEN** el sistema rechaza la operación con un error de permisos y la solicitud permanece `pendiente`

#### Scenario: Administrador decide sobre una solicitud

- **GIVEN** que un usuario tiene el rol de administrador
- **WHEN** aprueba una solicitud pendiente de otro usuario sin conflicto de disponibilidad
- **THEN** el sistema acepta la operación
