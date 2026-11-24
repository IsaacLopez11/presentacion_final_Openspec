# Spec Delta

## Purpose

Define los roles del sistema, cómo se acumulan en un mismo usuario y cómo se aplican los permisos en el servidor, de modo que ninguna operación quede autorizada por decisión del cliente.

## ADDED Requirements

### Requirement: Roles del sistema y permisos asociados

El sistema SHALL reconocer exactamente los roles `administrador`, `solicitante` y `aprobador`, cada uno con un conjunto fijo de permisos: el administrador mantiene usuarios, roles y espacios; el solicitante crea y cancela sus propias solicitudes; el aprobador revisa pendientes y decide sobre ellas.

#### Scenario: Permisos del administrador

- **GIVEN** que un usuario tiene únicamente el rol de administrador
- **WHEN** opera sobre usuarios, roles o espacios
- **THEN** el sistema le permite la operación

#### Scenario: Permisos del solicitante

- **GIVEN** que un usuario tiene únicamente el rol de solicitante
- **WHEN** crea una solicitud de reserva para sí mismo
- **THEN** el sistema acepta la operación

#### Scenario: Permisos del aprobador

- **GIVEN** que un usuario tiene únicamente el rol de aprobador
- **WHEN** aprueba una solicitud pendiente que no es suya
- **THEN** el sistema acepta la operación

### Requirement: Acumulación de roles

El sistema SHALL permitir que un mismo usuario tenga más de un rol simultáneamente, y sus permisos efectivos SHALL ser la unión de los permisos de sus roles.

#### Scenario: Usuario con dos roles

- **GIVEN** que un usuario tiene los roles de solicitante y aprobador
- **WHEN** aprueba una solicitud de otro usuario
- **THEN** el sistema acepta la operación por su rol de aprobador

#### Scenario: Un rol no anula al otro

- **GIVEN** que un usuario tiene los roles de solicitante y aprobador
- **WHEN** crea una solicitud de reserva para sí mismo
- **THEN** el sistema acepta la operación por su rol de solicitante

#### Scenario: Retiro de un rol

- **GIVEN** que un usuario tiene los roles de solicitante y aprobador
- **WHEN** un administrador le retira el rol de aprobador y el usuario intenta aprobar una solicitud
- **THEN** el sistema rechaza la operación con un error de permisos

### Requirement: Autorización calculada en el servidor

El sistema MUST derivar los permisos de los roles almacenados del usuario autenticado y MUST ignorar cualquier rol, permiso o identificador de usuario enviado por el cliente. Una petición que declare un rol distinto al almacenado SHALL ser tratada con los permisos reales.

#### Scenario: Rol falsificado en la petición

- **GIVEN** que un usuario solo tiene el rol de solicitante
- **WHEN** envía una petición que declara el rol de administrador en el cuerpo o en una cabecera
- **THEN** el sistema aplica los permisos de solicitante y rechaza la operación administrativa

#### Scenario: Identificador de usuario ajeno en la petición

- **GIVEN** que un solicitante ha iniciado sesión
- **WHEN** intenta cancelar una reserva indicando el identificador de otro usuario como autor
- **THEN** el sistema rechaza la operación con un error de permisos

### Requirement: Permisos insuficientes sin efectos colaterales

Cuando una operación se rechaza por permisos insuficientes, el sistema MUST responder con un error de autorización explícito y MUST NOT modificar ningún dato.

#### Scenario: Intento denegado que no altera el estado

- **GIVEN** que existe una solicitud pendiente y el usuario no tiene permiso para aprobarla
- **WHEN** intenta aprobarla
- **THEN** el sistema responde con un error de permisos y la solicitud permanece en estado pendiente

#### Scenario: Operación sobre recurso inexistente o ajeno

- **GIVEN** que un solicitante consulta una reserva que pertenece a otro usuario
- **WHEN** el sistema evalúa la petición
- **THEN** responde con un error de recurso no encontrado o de permisos, sin revelar los datos de la reserva ajena

### Requirement: Todo usuario activo tiene al menos un rol

El sistema MUST impedir dejar una cuenta activa sin roles y MUST impedir que el sistema quede sin ningún administrador activo.

#### Scenario: Retiro del último rol

- **GIVEN** que un usuario activo tiene un único rol
- **WHEN** un administrador intenta retirarle ese rol sin asignar otro
- **THEN** el sistema rechaza la operación e indica que la cuenta debe conservar al menos un rol

#### Scenario: Retiro del último administrador

- **GIVEN** que existe un solo usuario activo con rol de administrador
- **WHEN** se intenta retirar ese rol
- **THEN** el sistema rechaza la operación para preservar la administración del sistema
