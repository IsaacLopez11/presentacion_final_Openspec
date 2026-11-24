# Spec Delta

## Purpose

Garantiza que cada cambio de estado de una solicitud o reserva quede registrado de forma inmutable, con su autor, su instante y su motivo, para que el historial sea auditable y no dependa de la memoria de quien operó el sistema.

## ADDED Requirements

### Requirement: Registro de cada transición de estado

El sistema MUST registrar un evento por cada cambio de estado de una solicitud o reserva, indicando el estado anterior, el estado nuevo, el usuario responsable, el instante en UTC y el motivo cuando corresponda.

#### Scenario: Aprobación registrada

- **GIVEN** que una solicitud cambia de `pendiente` a `aprobada`
- **WHEN** se inspecciona la bitácora de esa solicitud
- **THEN** existe un evento con estado anterior `pendiente`, estado nuevo `aprobada`, el aprobador y el instante de la decisión

#### Scenario: Rechazo registrado con motivo

- **GIVEN** que una solicitud fue rechazada con un motivo
- **WHEN** se inspecciona la bitácora
- **THEN** el evento contiene el motivo indicado por el aprobador

#### Scenario: Cancelación registrada

- **GIVEN** que una reserva aprobada fue cancelada
- **WHEN** se inspecciona la bitácora
- **THEN** existe un evento de paso de `aprobada` a `cancelada` con el usuario que la canceló

### Requirement: Evento de creación

El sistema SHALL registrar un evento inicial cuando se crea una solicitud, con estado anterior ausente, estado nuevo `pendiente` y el autor de la solicitud.

#### Scenario: Creación registrada

- **WHEN** un solicitante crea una solicitud
- **THEN** la bitácora contiene un evento de creación con estado nuevo `pendiente` y el solicitante como responsable

#### Scenario: Toda solicitud tiene al menos un evento

- **GIVEN** que existe cualquier solicitud registrada
- **WHEN** se consulta su bitácora
- **THEN** existen al menos un evento de creación y un evento por cada transición posterior

### Requirement: Inmutabilidad de la bitácora

El sistema MUST impedir la modificación y el borrado de los eventos de auditoría a través de su interfaz. Los eventos SHALL persistir aunque la cuenta del usuario responsable o el espacio asociado se desactiven.

#### Scenario: Intento de modificar un evento por la interfaz

- **WHEN** un usuario envía una petición para modificar o eliminar un evento de auditoría
- **THEN** el sistema rechaza la operación y el evento permanece sin cambios

#### Scenario: Usuario responsable desactivado

- **GIVEN** que un aprobador que decidió sobre una solicitud fue desactivado después
- **WHEN** un administrador consulta la bitácora de esa solicitud
- **THEN** el evento sigue mostrando al aprobador que tomó la decisión

#### Scenario: Espacio desactivado

- **GIVEN** que un espacio fue desactivado
- **WHEN** se consulta la bitácora de una reserva de ese espacio
- **THEN** los eventos anteriores siguen disponibles y sin alteración

### Requirement: Consulta de la bitácora por el administrador

El sistema SHALL permitir al administrador consultar la bitácora de una solicitud o reserva concreta, y MUST impedir esa consulta a usuarios sin rol de administrador.

#### Scenario: Administrador consulta la bitácora

- **GIVEN** que existe una reserva con varias transiciones
- **WHEN** un administrador consulta su bitácora
- **THEN** el sistema devuelve los eventos ordenados cronológicamente

#### Scenario: Solicitante intenta consultar la bitácora

- **GIVEN** que el usuario tiene únicamente el rol de solicitante
- **WHEN** intenta consultar la bitácora de una reserva
- **THEN** el sistema rechaza la operación con un error de permisos

### Requirement: Coherencia entre estado y bitácora

El estado almacenado de una solicitud o reserva MUST corresponder al estado nuevo del último evento de su bitácora. Ninguna transición SHALL persistirse sin su evento correspondiente.

#### Scenario: Estado coincide con el último evento

- **GIVEN** que una reserva se encuentra en estado `cancelada`
- **WHEN** se compara con el último evento de su bitácora
- **THEN** el estado nuevo de ese último evento es `cancelada`

#### Scenario: Transición fallida no deja evento

- **GIVEN** que una aprobación falla por conflicto de disponibilidad
- **WHEN** se inspecciona la bitácora de esa solicitud
- **THEN** no existe ningún evento de aprobación y la solicitud sigue en `pendiente`
