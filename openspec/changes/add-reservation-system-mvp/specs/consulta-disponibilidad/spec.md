# Spec Delta

## Purpose

Permite conocer, antes de solicitar una reserva, qué espacios existen y en qué franjas están realmente disponibles, de modo que el solicitante deje de depender de preguntas por correo y la consulta coincida con la ocupación efectiva del espacio.

## ADDED Requirements

### Requirement: Consulta de disponibilidad por espacio y rango

El sistema SHALL permitir a cualquier usuario autenticado consultar la disponibilidad de un espacio para un rango temporal, devolviendo las franjas libres y las ocupadas dentro de ese rango, expresadas en hora local `America/Bogota`.

#### Scenario: Consulta de un rango parcialmente ocupado

- **GIVEN** que un espacio tiene una reserva aprobada de 10:00 a 12:00
- **WHEN** un usuario autenticado consulta la disponibilidad de ese espacio de 08:00 a 14:00
- **THEN** el sistema informa como ocupada la franja de 10:00 a 12:00 y como libres las franjas restantes

#### Scenario: Rango completamente libre

- **GIVEN** que un espacio no tiene reservas aprobadas en el día consultado
- **WHEN** un usuario autenticado consulta su disponibilidad para ese día
- **THEN** el sistema devuelve el rango como disponible

#### Scenario: Rango inválido

- **WHEN** un usuario consulta la disponibilidad con hora de fin anterior al inicio
- **THEN** el sistema rechaza la consulta con un error de validación

### Requirement: Coherencia con la política de qué estados bloquean

La consulta de disponibilidad MUST considerar ocupadas únicamente las reservas en estado `aprobada`. Las solicitudes en estado `pendiente`, `rechazada` o `cancelada` MUST NOT aparecer como bloqueo.

#### Scenario: Solicitud pendiente no bloquea

- **GIVEN** que existe una solicitud pendiente de 09:00 a 11:00 sobre un espacio
- **WHEN** un usuario consulta la disponibilidad de ese espacio
- **THEN** el sistema informa la franja de 09:00 a 11:00 como disponible

#### Scenario: Reserva cancelada libera la franja

- **GIVEN** que una reserva aprobada de 15:00 a 16:00 fue cancelada posteriormente
- **WHEN** un usuario consulta la disponibilidad de ese espacio
- **THEN** el sistema informa la franja de 15:00 a 16:00 como disponible

#### Scenario: Reserva rechazada no bloquea

- **GIVEN** que una solicitud fue rechazada para un rango determinado
- **WHEN** un usuario consulta la disponibilidad de ese espacio
- **THEN** el sistema informa ese rango como disponible

### Requirement: Privacidad de la información de ocupación

La consulta de disponibilidad MUST exponer únicamente la condición de ocupado o libre de cada franja y MUST NOT revelar la identidad, el motivo ni los datos de contacto del usuario cuya reserva ocupa el espacio.

#### Scenario: El solicitante no ve quién reservó

- **GIVEN** que un espacio tiene reservas aprobadas de otros usuarios
- **WHEN** un solicitante consulta la disponibilidad de ese espacio
- **THEN** la respuesta indica las franjas ocupadas sin incluir el nombre, el correo ni el identificador del autor de esas reservas

#### Scenario: Consulta anónima rechazada

- **GIVEN** que un usuario no ha iniciado sesión
- **WHEN** intenta consultar la disponibilidad de un espacio
- **THEN** el sistema rechaza la consulta con un error de autenticación

### Requirement: Existencia del espacio en la consulta

El sistema SHALL responder con un error de recurso no encontrado cuando el espacio consultado no exista, y SHALL informar la inexistencia de disponibilidad cuando el espacio esté inactivo.

#### Scenario: Espacio inexistente

- **WHEN** un usuario consulta la disponibilidad de un identificador de espacio que no existe
- **THEN** el sistema responde con un error de recurso no encontrado

#### Scenario: Espacio inactivo

- **GIVEN** que un espacio está inactivo
- **WHEN** un usuario consulta su disponibilidad
- **THEN** el sistema responde que el espacio no está disponible para reservas, sin listar franjas libres
