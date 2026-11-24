# Spec Delta

## Purpose

Define cómo el administrador registra y mantiene las cuentas de usuario de la institución, asegurando que las credenciales se almacenen de forma segura y que ninguna cuenta desactivada conserve acceso al sistema.

## ADDED Requirements

### Requirement: Registro de usuarios por el administrador

El sistema SHALL permitir al administrador registrar usuarios con nombre completo, correo institucional, contraseña inicial y uno o más roles. El correo SHALL ser único en el sistema, comparado sin distinguir mayúsculas, y el usuario SHALL quedar activo al crearse.

#### Scenario: Alta de usuario válida

- **WHEN** un administrador registra un usuario con nombre, correo único, contraseña inicial y rol de solicitante
- **THEN** el sistema crea la cuenta activa y le asigna el rol indicado, sin devolver la contraseña en la respuesta

#### Scenario: Correo duplicado

- **GIVEN** que existe una cuenta con el correo "docente@institucion.edu.co"
- **WHEN** un administrador intenta registrar otro usuario con el mismo correo en mayúsculas distintas
- **THEN** el sistema rechaza la operación por correo duplicado y no crea la cuenta

#### Scenario: Contraseña inicial insuficiente

- **WHEN** un administrador intenta registrar un usuario con una contraseña que no cumple la longitud mínima definida
- **THEN** el sistema rechaza la operación con un error de validación

### Requirement: Almacenamiento seguro de credenciales

El sistema MUST almacenar las contraseñas únicamente como hash con un algoritmo de derivación de clave lento, y MUST NOT exponer la contraseña, su hash ni ningún dato derivado en respuestas, registros o mensajes de error.

#### Scenario: La contraseña no se devuelve

- **WHEN** un administrador consulta los datos de un usuario recién creado
- **THEN** la respuesta contiene los datos de la cuenta pero ningún campo de contraseña ni de hash

#### Scenario: Verificación de acceso con la contraseña correcta

- **GIVEN** que existe una cuenta activa con una contraseña conocida
- **WHEN** el usuario se autentica con esa contraseña
- **THEN** el sistema concede el acceso

#### Scenario: Verificación de acceso con contraseña incorrecta

- **WHEN** un usuario se autentica con una contraseña distinta de la registrada
- **THEN** el sistema deniega el acceso sin revelar si el correo existe

### Requirement: Desactivación lógica de usuarios

El sistema SHALL permitir al administrador desactivar una cuenta sin eliminarla. Una cuenta desactivada MUST NOT poder autenticarse y SHALL conservar su historial de reservas y sus registros de auditoría.

#### Scenario: Usuario desactivado no puede iniciar sesión

- **GIVEN** que una cuenta está activa y tiene sesión iniciada
- **WHEN** un administrador la desactiva y el usuario intenta acceder de nuevo
- **THEN** el sistema deniega el acceso e invalida la sesión existente

#### Scenario: Historial de una cuenta desactivada

- **GIVEN** que una cuenta desactivada tiene reservas anteriores
- **WHEN** un administrador consulta el historial global
- **THEN** el sistema sigue mostrando las reservas de esa cuenta

#### Scenario: El usuario no puede desactivarse a sí mismo si es el único administrador

- **GIVEN** que existe un único usuario con rol de administrador y es el que opera
- **WHEN** intenta desactivar su propia cuenta
- **THEN** el sistema rechaza la operación para no dejar el sistema sin administración

### Requirement: Gestión de usuarios restringida al administrador

El sistema SHALL permitir registrar, consultar, modificar y desactivar usuarios únicamente a usuarios con rol de administrador. Los demás usuarios autenticados MUST recibir un error de permisos insuficientes y MUST NOT poder consultar datos de otras cuentas.

#### Scenario: Solicitante consulta datos de otro usuario

- **GIVEN** que el usuario tiene únicamente el rol de solicitante
- **WHEN** intenta consultar los datos de otra cuenta
- **THEN** el sistema rechaza la operación con un error de permisos

#### Scenario: Aprobador intenta desactivar una cuenta

- **GIVEN** que el usuario tiene únicamente el rol de aprobador
- **WHEN** intenta desactivar otra cuenta
- **THEN** el sistema rechaza la operación con un error de permisos y la cuenta permanece activa
