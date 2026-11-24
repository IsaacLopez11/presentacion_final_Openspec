# Spec Delta

## Purpose

Permite consultar el historial de solicitudes y reservas —propio o global según el rol— con aislamiento entre usuarios, de modo que la institución pueda reconstruir qué se pidió, qué se aprobó y qué se canceló.

## ADDED Requirements

### Requirement: Historial propio del solicitante

El sistema SHALL permitir a cada usuario consultar el historial completo de sus propias solicitudes y reservas, en todos los estados, con al menos el espacio, el rango solicitado, el estado actual y las fechas de creación y de última decisión.

#### Scenario: Historial con varias solicitudes

- **GIVEN** que un solicitante tiene una solicitud pendiente, una aprobada y una rechazada
- **WHEN** consulta su historial
- **THEN** el sistema devuelve las tres con su estado actual y sus datos de espacio y rango

#### Scenario: Historial vacío

- **GIVEN** que un solicitante no ha creado solicitudes
- **WHEN** consulta su historial
- **THEN** el sistema devuelve una lista vacía sin error

#### Scenario: Filtro por estado

- **GIVEN** que un solicitante tiene solicitudes en distintos estados
- **WHEN** consulta su historial filtrando por estado `aprobada`
- **THEN** el sistema devuelve únicamente las aprobadas

### Requirement: Aislamiento del historial entre usuarios

El sistema MUST impedir que un usuario consulte el historial de otro usuario, y MUST responder con un error de recurso no encontrado o de permisos sin revelar los datos ajenos.

#### Scenario: Consulta del historial ajeno

- **GIVEN** que existen reservas de otro solicitante
- **WHEN** un solicitante intenta consultar el historial de ese otro usuario
- **THEN** el sistema rechaza la operación sin revelar datos de esas reservas

#### Scenario: Acceso a una reserva ajena por identificador

- **GIVEN** que una reserva pertenece a otro usuario
- **WHEN** un solicitante consulta su detalle por identificador
- **THEN** el sistema responde como recurso no encontrado

### Requirement: Historial global del administrador

El sistema SHALL permitir al administrador consultar el historial global de todas las solicitudes y reservas, con filtros por espacio, por usuario, por estado y por rango de fechas.

#### Scenario: Historial global sin filtros

- **GIVEN** que existen reservas de varios usuarios
- **WHEN** un administrador consulta el historial global
- **THEN** el sistema devuelve las reservas de todos los usuarios

#### Scenario: Filtro por espacio y rango de fechas

- **GIVEN** que existen reservas de varios espacios en distintas fechas
- **WHEN** un administrador filtra por un espacio y un rango de fechas
- **THEN** el sistema devuelve únicamente las reservas que cumplen ambos criterios

#### Scenario: Solicitante intenta el historial global

- **GIVEN** que el usuario tiene únicamente el rol de solicitante
- **WHEN** intenta consultar el historial global
- **THEN** el sistema rechaza la operación con un error de permisos

### Requirement: Historial de decisiones del aprobador

El sistema SHALL permitir al aprobador consultar las solicitudes sobre las que emitió una decisión, indicando la decisión tomada y su fecha.

#### Scenario: Consulta de decisiones propias

- **GIVEN** que un aprobador aprobó y rechazó varias solicitudes
- **WHEN** consulta su historial de decisiones
- **THEN** el sistema devuelve esas solicitudes con la decisión y la fecha correspondientes

#### Scenario: Un aprobador no ve decisiones de otro

- **GIVEN** que otro aprobador decidió sobre solicitudes distintas
- **WHEN** un aprobador consulta su historial de decisiones
- **THEN** el sistema no incluye las decisiones del otro aprobador

### Requirement: Representación del estado finalizado por fecha

El sistema MUST NOT almacenar un estado `finalizada`. Cuando una reserva aprobada ya terminó según la hora actual, el sistema SHALL presentarla como finalizada de forma derivada, conservando su estado almacenado `aprobada`.

#### Scenario: Reserva pasada mostrada como finalizada

- **GIVEN** que una reserva aprobada terminó ayer
- **WHEN** un usuario consulta su historial
- **THEN** el sistema la presenta como finalizada, sin haber cambiado su estado almacenado

#### Scenario: Reserva futura mostrada como aprobada

- **GIVEN** que una reserva aprobada inicia mañana
- **WHEN** un usuario consulta su historial
- **THEN** el sistema la presenta como aprobada y no como finalizada

### Requirement: Orden y paginación del historial

El sistema SHALL devolver el historial ordenado por fecha de inicio descendente y SHALL permitir paginar los resultados, informando el total disponible.

#### Scenario: Orden descendente

- **GIVEN** que un usuario tiene reservas en fechas distintas
- **WHEN** consulta su historial
- **THEN** las reservas aparecen de la más reciente a la más antigua

#### Scenario: Página con total

- **GIVEN** que existen más reservas que el tamaño de página solicitado
- **WHEN** el usuario consulta una página del historial
- **THEN** el sistema devuelve esa página e informa el total de elementos disponibles
