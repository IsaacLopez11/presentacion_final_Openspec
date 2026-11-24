# Tasks

Implementación del MVP por incrementos. Cada tarea declara cómo se verifica. Precondición de entorno: Go, Node.js, npm y PostgreSQL disponibles (riesgo RG-03); sin ellos las tareas de compilación y prueba no pueden verificarse.

## 1. Esqueleto y configuración

- [ ] 1.1 Crear el módulo Go en `backend/` (`go.mod`, `cmd/api/main.go`, `internal/`) y verificar que `go build ./...` y `go vet ./...` terminan sin errores
- [ ] 1.2 Implementar carga y validación de configuración por variables de entorno (`APP_ENV`, `HTTP_ADDR`, `DATABASE_URL`, `SESSION_TTL`, `COOKIE_SECURE`, `FRONTEND_ORIGIN`, `BCRYPT_COST`, `LOG_LEVEL`) y verificar con prueba unitaria que falta `DATABASE_URL` impide arrancar con mensaje claro
- [ ] 1.3 Exponer `GET /health` y verificar con `httptest` que responde 200 y un cuerpo JSON con el estado
- [ ] 1.4 Añadir `.gitignore` (binarios, `.env`, `node_modules`, `dist`) y `.env.example` sin secretos, y verificar que `git status` no muestra archivos de entorno
- [ ] 1.5 Documentar en `README.md` los requisitos previos y el arranque del backend, y verificar siguiendo el README en una terminal limpia

## 2. Base de datos y migraciones

- [ ] 2.1 Escribir `0001_init.up.sql` con `btree_gist`, tablas `users`, `roles`, `user_roles`, `spaces`, `bookings`, `booking_events`, `sessions`, `holidays`, índices y restricciones, y verificar aplicando desde una base vacía con `golang-migrate up`
- [ ] 2.2 Escribir los `.down.sql` correspondientes y verificar ciclo `up` → `down` → `up` sin residuos
- [ ] 2.3 Sembrar el catálogo de roles y sus permisos en `0002_seed_roles.up.sql` y verificar con consulta que los tres roles existen con sus permisos
- [ ] 2.4 Sembrar los festivos del año en curso en `0003_seed_holidays.up.sql` y verificar con consulta que las fechas sembradas coinciden con el calendario oficial usado
- [ ] 2.5 Implementar el adaptador `postgres` con `pgxpool`, verificación de conexión y de existencia de `btree_gist` al arrancar, y verificar con prueba de integración que una base sin la extensión falla con mensaje explícito
- [ ] 2.6 Escribir la prueba de integración que demuestra que la restricción de exclusión rechaza dos reservas aprobadas solapadas y permite rangos contiguos, y verificarla con `go test ./internal/adapters/postgres/...`

## 3. Dominio y casos de uso de identidad y espacios

- [ ] 3.1 Implementar `domain/identity` (User, Role, Permission, unión de permisos) con pruebas de tabla que cubran acumulación de roles y retiro del último rol
- [ ] 3.2 Implementar `domain/spaces` (Space, SpaceType, validaciones de nombre, capacidad y tipo) con pruebas unitarias de cada validación
- [ ] 3.3 Implementar los repositorios de usuarios, roles y espacios como puertos más su implementación `pgx`, y verificar con pruebas de integración de alta, búsqueda por correo y desactivación lógica
- [ ] 3.4 Implementar los casos de uso de gestión de usuarios y espacios con autorización por permiso, y verificar con pruebas que un solicitante recibe error de permisos sin efectos colaterales
- [ ] 3.5 Documentar los endpoints de usuarios y espacios en `docs/api.md` y verificar que las rutas documentadas responden según lo descrito al ejecutarlas

## 4. Autenticación y autorización

- [ ] 4.1 Implementar el hash de contraseñas con `bcrypt` y su verificación, y verificar con pruebas que la contraseña nunca se devuelve en la representación del usuario
- [ ] 4.2 Implementar el repositorio de sesiones con token opaco hasheado, expiración y revocación, y verificar con prueba de integración que una sesión revocada deja de autenticar
- [ ] 4.3 Implementar login y logout con cookie `HttpOnly` y verificar con `httptest` que la cookie se emite, que el token no aparece en el cuerpo y que el logout invalida la sesión
- [ ] 4.4 Implementar el middleware de autenticación y el de autorización por permiso, y verificar con pruebas por endpoint que falta de sesión devuelve 401 y falta de permiso devuelve 403
- [ ] 4.5 Implementar el comando de bootstrap del primer administrador desde variables de entorno y verificar que no exista endpoint público que cree administradores
- [ ] 4.6 Verificar con prueba que una petición con rol falsificado en el cuerpo o en cabecera se procesa con los permisos reales del usuario
- [ ] 4.7 Documentar el flujo de autenticación y la configuración de la cookie en `README.md` y verificar que el flujo descrito funciona con `curl`

## 5. Solicitudes de reserva

- [ ] 5.1 Implementar `domain/booking` (Booking, Status, transiciones permitidas) con pruebas de tabla de todas las transiciones válidas e inválidas
- [ ] 5.2 Implementar `domain/availability` con `Interval` y `Overlaps` semiabierto, y verificar con pruebas de borde: contiguos, idénticos, contenidos, cruce de medianoche y rangos disjuntos
- [ ] 5.3 Implementar las reglas de horario operativo, festivos y anticipación en el dominio con el puerto `Clock`, y verificar con pruebas de límites exactos (24 h y 90 días) y de días no habilitados
- [ ] 5.4 Implementar el caso de uso de creación de solicitud que fuerza estado `pendiente` y autor autenticado, y verificar con pruebas de que el estado y el autor enviados por el cliente se ignoran
- [ ] 5.5 Implementar los endpoints de creación y consulta de solicitudes propias, y verificar con `httptest` los casos de validación: fin anterior al inicio, duración menor a 30 minutos, granularidad distinta de 15 minutos y espacio inactivo
- [ ] 5.6 Verificar con prueba que las horas se interpretan en `America/Bogota` y se almacenan en UTC (10:00 local corresponde a 15:00 UTC)

## 6. Disponibilidad

- [ ] 6.1 Implementar el caso de uso de consulta de disponibilidad que devuelve franjas libres y ocupadas recortadas al horario operativo, y verificar con pruebas de rango parcialmente ocupado y completamente libre
- [ ] 6.2 Verificar con prueba que solo las reservas `aprobada` aparecen como ocupadas y que una solicitud `pendiente` no bloquea la franja
- [ ] 6.3 Verificar con prueba que la respuesta de disponibilidad no incluye nombre, correo ni identificador del autor de las reservas existentes
- [ ] 6.4 Implementar el endpoint de disponibilidad y verificar con `httptest` los casos de rango inválido, espacio inexistente y espacio inactivo

## 7. Aprobación, rechazo y no solapamiento

- [ ] 7.1 Implementar el caso de uso de aprobación en una transacción que bloquea la fila, revalida disponibilidad y registra auditoría, y verificar con prueba de integración de aprobación exitosa con evento asociado
- [ ] 7.2 Traducir la violación de exclusión `23P01` a un error de conflicto y verificar con prueba que la API responde 409 y la solicitud permanece `pendiente`
- [ ] 7.3 Implementar la prohibición de autoaprobación en el dominio y verificar con pruebas de aprobación y rechazo de la propia solicitud
- [ ] 7.4 Implementar el rechazo con motivo obligatorio y verificar con prueba que un motivo vacío no cambia el estado
- [ ] 7.5 Implementar la bandeja de pendientes ordenada por fecha de creación y verificar con prueba que excluye los demás estados y que un solicitante recibe 403
- [ ] 7.6 Escribir la prueba de concurrencia con 8 goroutines que aprueban el mismo rango, y verificar que exactamente una tiene éxito y que la consulta de solapados devuelve cero
- [ ] 7.7 Repetir la prueba de concurrencia con rangos parcialmente solapados y verificar que el estado final no contiene solapamientos
- [ ] 7.8 Documentar en `docs/api.md` los endpoints de aprobación y rechazo con sus códigos de error y verificar que las respuestas reales coinciden con lo documentado

## 8. Cancelación, historial y auditoría

- [ ] 8.1 Implementar la cancelación con plazos por rol (pendiente sin plazo; aprobada hasta 24 h antes; administrador sin límite) y verificar con pruebas de límite exacto y de cancelación administrativa
- [ ] 8.2 Verificar con prueba que tras cancelar una reserva aprobada una nueva solicitud del mismo rango puede aprobarse y que la disponibilidad refleja la liberación
- [ ] 8.3 Verificar con prueba que un solicitante no puede cancelar la reserva de otro y que la respuesta no revela datos ajenos
- [ ] 8.4 Implementar el historial propio, el global del administrador y el de decisiones del aprobador, y verificar con pruebas de aislamiento entre usuarios y de filtros por estado, espacio y fechas
- [ ] 8.5 Verificar con prueba que el historial se ordena por fecha de inicio descendente, que pagina e informa el total, y que una reserva pasada se presenta como finalizada sin cambiar su estado almacenado
- [ ] 8.6 Implementar la bitácora inmutable con evento de creación y consulta administrativa, y verificar con pruebas de que no existe operación de modificación o borrado y de que los eventos sobreviven a la desactivación del usuario y del espacio
- [ ] 8.7 Verificar con prueba que el estado almacenado coincide con el estado nuevo del último evento y que una aprobación fallida no deja evento

## 9. Frontend

- [ ] 9.1 Crear el proyecto Vite con TypeScript estricto y React Router, y verificar con `npm run build` y `tsc --noEmit` sin errores
- [ ] 9.2 Implementar el cliente HTTP centralizado con manejo uniforme de errores y verificar con prueba que un 409 y un 403 se convierten en errores tipados distintos
- [ ] 9.3 Implementar login, contexto de sesión y protección de rutas, y verificar con pruebas de acceso denegado sin sesión e inicio de sesión correcto
- [ ] 9.4 Implementar las páginas de espacios y disponibilidad con estados de carga, error y vacío, y verificar con pruebas de RTL de los tres estados
- [ ] 9.5 Implementar la página de creación de solicitud con validación de formulario y manejo del conflicto 409, y verificar con pruebas de envío válido, error de validación y conflicto del servidor
- [ ] 9.6 Implementar la página de aprobaciones con acciones de aprobar y rechazar, y verificar con pruebas que el motivo es obligatorio y que un error de permisos se muestra sin romper la vista
- [ ] 9.7 Implementar el historial y las páginas de administración de usuarios y espacios, y verificar con pruebas de visibilidad de acciones según permisos y de estados de carga y error
- [ ] 9.8 Documentar en `README.md` cómo ejecutar el frontend y sus pruebas, y verificar los comandos del README en una terminal limpia

## 10. Integración

- [ ] 10.1 Ejecutar la suite completa del backend (`gofmt -l`, `go vet ./...`, `go test ./...`) y verificar que no hay diferencias de formato ni fallos
- [ ] 10.2 Ejecutar la suite completa del frontend (`npm run test`, `npm run build`) y verificar que termina sin fallos
- [ ] 10.3 Verificar el flujo completo de extremo a extremo: autenticación → espacios → disponibilidad → solicitud → aprobación → historial
- [ ] 10.4 Verificar el flujo de rechazo, el de cancelación y el de conflicto de disponibilidad de extremo a extremo
- [ ] 10.5 Verificar la migración desde una base completamente limpia siguiendo el README y observar que el sistema arranca y responde
- [ ] 10.6 Confirmar que no hay secretos ni datos reales en el repositorio y que `.env` no está versionado

## Workflow follow-up

- Actualizar `docs/REVISION_ESPECIFICACIONES.md` con el resultado de la implementación frente a lo aprobado.
- Completar `docs/traceability/MATRIZ_TRAZABILIDAD.md` con los identificadores reales de especificación, código y pruebas.
- Archivar el cambio con `openspec archive` cuando el estudiante apruebe la implementación y la revisión.
- Verificar el resultado archivado y que `openspec/specs/` quedó actualizado.
