# Spec Delta

## Purpose

Define el catálogo de espacios académicos reservables (laboratorios, auditorios y salas de cómputo) y las reglas para crearlos, modificarlos y desactivarlos, de modo que exista una fuente de verdad única de qué se puede reservar y bajo qué condiciones.

## ADDED Requirements

### Requirement: Catálogo de espacios con atributos mínimos

El sistema SHALL mantener un catálogo de espacios académicos donde cada espacio tiene nombre, tipo, capacidad, ubicación y estado (activo o inactivo). El tipo SHALL pertenecer a un conjunto definido de tipos de espacio académico.

#### Scenario: Consulta del catálogo por un usuario autenticado

- **GIVEN** que existe al menos un espacio registrado
- **WHEN** un usuario autenticado consulta el catálogo de espacios
- **THEN** el sistema devuelve cada espacio con su nombre, tipo, capacidad, ubicación y estado

#### Scenario: Espacio creado con atributos completos

- **WHEN** un administrador registra un espacio indicando nombre, tipo, capacidad, ubicación y estado
- **THEN** el sistema lo persiste y lo devuelve con un identificador único

#### Scenario: Consulta del catálogo sin autenticación

- **GIVEN** que el solicitante no ha iniciado sesión
- **WHEN** intenta consultar el catálogo de espacios
- **THEN** el sistema rechaza la operación con un error de autenticación y no devuelve datos

### Requirement: Validación de los datos del espacio

El sistema MUST rechazar la creación o modificación de un espacio cuando el nombre esté vacío, cuando la capacidad no sea un entero mayor que cero, o cuando el tipo no pertenezca al conjunto de tipos admitidos. El nombre SHALL ser único en el catálogo, comparado sin distinguir mayúsculas.

#### Scenario: Capacidad inválida

- **WHEN** un administrador intenta registrar un espacio con capacidad cero o negativa
- **THEN** el sistema rechaza la operación con un error de validación y no crea el espacio

#### Scenario: Nombre duplicado

- **GIVEN** que existe un espacio llamado "Laboratorio de Redes"
- **WHEN** un administrador intenta crear otro espacio con el nombre "laboratorio de redes"
- **THEN** el sistema rechaza la operación por nombre duplicado y no crea el espacio

#### Scenario: Tipo de espacio no admitido

- **WHEN** un administrador registra un espacio con un tipo fuera del conjunto definido
- **THEN** el sistema rechaza la operación con un error de validación

### Requirement: Solo el administrador modifica el catálogo

El sistema SHALL permitir crear, modificar y desactivar espacios únicamente a usuarios con el rol de administrador. Cualquier otro usuario autenticado SHALL recibir un error de permisos insuficientes.

#### Scenario: Solicitante intenta crear un espacio

- **GIVEN** que el usuario tiene únicamente el rol de solicitante
- **WHEN** intenta registrar un espacio académico
- **THEN** el sistema rechaza la operación con un error de permisos y no crea el espacio

#### Scenario: Aprobador intenta modificar un espacio

- **GIVEN** que el usuario tiene únicamente el rol de aprobador
- **WHEN** intenta modificar la capacidad de un espacio existente
- **THEN** el sistema rechaza la operación con un error de permisos y el espacio conserva sus valores

#### Scenario: Administrador modifica un espacio

- **GIVEN** que existe un espacio activo
- **WHEN** un administrador cambia su capacidad por un valor válido
- **THEN** el sistema guarda el cambio y lo devuelve actualizado

### Requirement: Desactivación de espacios con preservación del historial

Cuando un espacio se marca como inactivo, el sistema MUST impedir la creación de nuevas solicitudes de reserva sobre él y MUST conservar intactas sus reservas y su historial previos. La desactivación SHALL ser lógica: el registro no se elimina.

#### Scenario: Solicitud sobre un espacio inactivo

- **GIVEN** que un espacio está marcado como inactivo
- **WHEN** un solicitante intenta crear una solicitud de reserva para ese espacio
- **THEN** el sistema rechaza la solicitud e indica que el espacio no está disponible para reservas

#### Scenario: Historial de un espacio desactivado

- **GIVEN** que un espacio inactivo tiene reservas aprobadas anteriores a su desactivación
- **WHEN** un administrador consulta el historial de ese espacio
- **THEN** el sistema devuelve las reservas anteriores sin alteración

#### Scenario: Reactivación de un espacio

- **GIVEN** que un espacio está inactivo
- **WHEN** un administrador lo marca como activo
- **THEN** el sistema vuelve a admitir solicitudes de reserva sobre ese espacio
