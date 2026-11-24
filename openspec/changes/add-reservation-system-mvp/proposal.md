# Proposal — Sistema de gestión de reservas de espacios académicos (MVP)

## Why

La institución administra hoy las reservas de laboratorios, auditorios y salas de cómputo con correo electrónico y hojas de cálculo. El resultado es duplicidad de solicitudes, conflictos de horario descubiertos tarde, disponibilidad no consultable e historial no confiable. El problema de fondo no es la comodidad, sino la **ausencia de una fuente de verdad**: nada garantiza que dos reservas no ocupen el mismo espacio a la misma hora, ni que una solicitud "pendiente" no se confunda con una reserva aprobada.

Este cambio introduce el MVP de un sistema web centralizado que sustituye ese circuito por un flujo con estado explícito y una base de datos que **impone** las invariantes críticas, en lugar de confiar en la disciplina de quien usa la hoja de cálculo.

## What Changes

- Se crea el catálogo de **espacios académicos** con tipo, capacidad, ubicación y estado activo/inactivo.
- Se incorpora **gestión de usuarios** con alta, consulta, actualización y desactivación lógica, con contraseñas almacenadas como hash.
- Se introduce un modelo de **roles y permisos** (administrador, solicitante, aprobador) con acumulación de roles y autorización verificada en el servidor en cada endpoint protegido.
- Se añade la **creación de solicitudes de reserva**, que nacen en estado `pendiente`, con validación de fechas, anticipación, horario operativo y duración mínima.
- Se añade la **consulta de disponibilidad** de un espacio para un rango temporal.
- Se garantiza la **prevención de solapamientos** mediante una restricción de exclusión en PostgreSQL sobre rangos de tiempo, con revalidación al aprobar.
- Se incorpora el flujo de **aprobación y rechazo** con motivo obligatorio al rechazar y prohibición de autoaprobación.
- Se añade la **cancelación** de solicitudes y reservas conforme a la política de plazos.
- Se añade el **historial de reservas** propio y global según rol, más una **bitácora de transiciones de estado** con autor, fecha y motivo.
- Se define la **política de disponibilidad**: solo las reservas `aprobada` bloquean el espacio; una solicitud `pendiente` no bloquea y su disponibilidad se revalida al aprobar.

**Sin cambios de ruptura:** no existe sistema previo, por lo que no hay compatibilidad que romper.

**Fuera de alcance de este cambio:** pagos, notificaciones externas, integración con calendarios institucionales, reservas recurrentes, aplicación móvil nativa, carga masiva, registro público o recuperación de contraseña por correo, aprobaciones multinivel y reportería avanzada. El MVP tampoco impone un límite a la duración de una reserva individual; cualquier límite futuro será un cambio formal independiente.

## Capabilities

### New Capabilities

- `gestion-espacios`: catálogo de espacios académicos, sus atributos y su estado; un espacio desactivado no admite nuevas solicitudes pero conserva su historial.
- `gestion-usuarios`: registro y administración de usuarios por parte del administrador, incluida la desactivación lógica y el resguardo de credenciales.
- `roles-permisos`: roles del sistema, acumulación de roles por usuario, permisos derivados y autorización aplicada en el servidor.
- `solicitudes-reserva`: creación de solicitudes en estado `pendiente`, con validación de rango temporal, anticipación, horario operativo y duración mínima.
- `consulta-disponibilidad`: consulta de espacios y de su disponibilidad real para un rango temporal, coherente con la política de qué estados bloquean.
- `prevencion-solapamientos`: invariante que impide que dos reservas aprobadas se solapen en el mismo espacio, garantizada en la persistencia.
- `aprobacion-rechazo`: revisión de solicitudes pendientes, decisión de aprobar o rechazar con motivo y control de autoaprobación.
- `cancelacion-reservas`: cancelación de solicitudes y reservas conforme a plazos y rol, con efecto sobre la disponibilidad.
- `historial-reservas`: consulta del historial propio y global según rol, con aislamiento entre usuarios.
- `auditoria-estados`: registro inmutable de cada transición de estado con autor, fecha y motivo.

### Modified Capabilities

Ninguna: el proyecto no tiene capacidades especificadas previamente.

## Impact

- **Código nuevo desde cero:** `backend/` (Go, hexagonal) y `frontend/` (React + TypeScript + Vite). No hay código existente que modificar.
- **Base de datos:** esquema PostgreSQL con `users`, `roles`, `user_roles`, `spaces`, `bookings`, `booking_events` y `sessions`; extensión `btree_gist` y restricción de exclusión sobre `bookings`.
- **API HTTP:** endpoints REST bajo `/api` para autenticación, usuarios, roles, espacios, disponibilidad, reservas y aprobaciones.
- **Dependencias:** `pgx/v5`, `bcrypt`, `golang-migrate` (desarrollo); React, Vite, React Router, Vitest y React Testing Library. Cada una justificada en `PLAN_INGENIERIA.md` (sección 3.6).
- **Documentación:** `README.md` de ejecución, `docs/REVISION_ESPECIFICACIONES.md` y `docs/traceability/MATRIZ_TRAZABILIDAD.md`.
- **Proceso:** este cambio es la base de las etapas E1–E9 del plan; su implementación requiere especificaciones y tareas aprobadas.
- **Entorno:** la máquina de desarrollo aún no tiene Go, Node.js, npm ni PostgreSQL instalados; la ejecución y las pruebas de la implementación dependen de resolverlo (riesgo RG-03 del plan).
