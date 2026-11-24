# Design — Sistema de gestión de reservas de espacios académicos

Referencias: motivación y alcance en `proposal.md`; comportamiento exigido en `specs/*/spec.md`; decisiones de ingeniería y su justificación ampliada en `PLAN_INGENIERIA.md` (sección 3).

## Context

Repositorio vacío: no hay código, esquema ni pruebas. El diseño parte de cero y debe ser ejecutable en un proyecto académico individual, con arquitectura hexagonal y sin framework web.

Restricciones vigentes:

- Go con `net/http` (patrones de método y ruta de Go 1.22); PostgreSQL con `pgx/v5`; migraciones SQL con `golang-migrate`; `bcrypt` para contraseñas.
- React + TypeScript estricto + Vite; Vitest y React Testing Library.
- Dominio y aplicación sin dependencias de HTTP, React ni el driver de base de datos.
- La invariante de no solapamiento debe vivir en la persistencia, no solo en el código.
- El entorno de desarrollo aún no tiene Go, Node.js, npm ni PostgreSQL (riesgo RG-03 del plan).

## Goals / Non-Goals

**Goals**

- Un esquema de datos en el que la regla "dos reservas aprobadas no se solapan" sea imposible de violar, incluso con aprobaciones simultáneas.
- Un flujo de aprobación en una sola transacción que revalida disponibilidad y registra auditoría de forma atómica.
- Autorización derivada de roles almacenados, verificada en el servidor en cada endpoint protegido.
- Cálculo de tiempo correcto con almacenamiento en UTC y presentación en `America/Bogota`.
- Suites de prueba que permitan demostrar cada requisito: dominio puro, repositorios contra PostgreSQL real, HTTP y concurrencia.

**Non-Goals**

- No se diseña alta disponibilidad, caché distribuida ni despliegue multi-instancia.
- No se modelan recursos adicionales reservables (proyectores, equipos) ni reservas recurrentes.
- No se diseña un sistema de notificaciones: los cambios de estado solo se reflejan en la API y la interfaz.
- No se define el mecanismo de bootstrap del primer administrador como API pública (se resuelve con un comando de línea de órdenes).

## Decisions

### D1. Organización de paquetes

```text
backend/
  cmd/api/main.go            composición, configuración, arranque, apagado
  internal/domain/identity/  User, Role, Permission, invariantes
  internal/domain/spaces/    Space, SpaceType, invariantes
  internal/domain/booking/   Booking, Status, transiciones
  internal/domain/availability/ Interval, solapamiento, franjas
  internal/application/      casos de uso y transacciones
  internal/ports/            interfaces de repositorio y servicios
  internal/adapters/http/    rutas, DTOs, middleware, mapeo de errores
  internal/adapters/postgres/ repositorios pgx
  migrations/                SQL versionado
```

**Decisión:** un paquete por concepto de dominio y adaptadores separados; composición explícita en `main` mediante constructores que reciben interfaces.
**Alternativa descartada:** un paquete `service` único con acceso directo a `pgxpool`. Acoplaría dominio y aplicación al driver e impediría probar reglas sin base de datos, que es justamente lo que el plan exige demostrar.

### D2. Esquema y restricción de exclusión

```sql
-- 0001_init.up.sql (extracto)
CREATE EXTENSION IF NOT EXISTS btree_gist;

CREATE TABLE spaces (
  id          uuid PRIMARY KEY,
  name        text NOT NULL,
  type        text NOT NULL,
  capacity    integer NOT NULL CHECK (capacity > 0),
  location    text NOT NULL,
  is_active   boolean NOT NULL DEFAULT true,
  created_at  timestamptz NOT NULL DEFAULT now()
);
CREATE UNIQUE INDEX spaces_name_unique ON spaces (lower(name));

CREATE TABLE bookings (
  id             uuid PRIMARY KEY,
  space_id       uuid NOT NULL REFERENCES spaces(id),
  requester_id   uuid NOT NULL REFERENCES users(id),
  starts_at      timestamptz NOT NULL,
  ends_at        timestamptz NOT NULL,
  status         text NOT NULL CHECK (status IN ('pendiente','aprobada','rechazada','cancelada')),
  decided_by     uuid REFERENCES users(id),
  decided_at     timestamptz,
  reason         text,
  created_at     timestamptz NOT NULL DEFAULT now(),
  CONSTRAINT bookings_range_valid CHECK (ends_at > starts_at),
  CONSTRAINT bookings_min_duration CHECK (ends_at - starts_at >= interval '30 minutes'),
  CONSTRAINT bookings_no_overlap
    EXCLUDE USING gist (
      space_id WITH =,
      tstzrange(starts_at, ends_at, '[)') WITH &&
    ) WHERE (status = 'aprobada')
);
```

**Decisión:** la restricción de exclusión parcial es la garantía. `CHECK` de duración y de coherencia del rango son la segunda línea defensiva en datos.
**Alternativa descartada:** `SELECT ... FOR UPDATE` sobre filas solapadas. Requiere bloquear correctamente en todos los caminos de escritura y reintroduce la posibilidad de deadlocks; la restricción es declarativa y verificable por inspección del esquema.
**Consecuencia asumida:** la violación llega como error de PostgreSQL `23P01` (exclusion_violation), que el adaptador traduce a un error de dominio `ErrSlotConflict` y la API a `409 Conflict`.

### D3. Transacción de aprobación

Secuencia dentro de una única transacción `read committed`:

1. `SELECT ... FOR UPDATE` de la reserva por id, verificando que siga en `pendiente` (evita doble decisión sobre el mismo registro).
2. Verificar que el aprobador no es el autor y que tiene permiso.
3. Consultar solapamientos aprobados del espacio con `tstzrange && tstzrange` (para mensaje útil).
4. `UPDATE bookings SET status='aprobada', decided_by=..., decided_at=...`.
5. `INSERT booking_events` con estado anterior, nuevo, autor y motivo.
6. `COMMIT`. Si el `UPDATE` devuelve `23P01`, se revierte y se responde conflicto.

**Decisión:** la auditoría se escribe en la misma transacción que el cambio de estado. Si la auditoría fallara, la transición no se confirma: no puede existir estado sin evidencia.
**Alternativa descartada:** escritura de auditoría asíncrona. Abriría una ventana en la que el estado cambió sin registro, contrario al requisito de coherencia entre estado y bitácora.

### D4. Sesión y autenticación

- Al iniciar sesión se genera un token aleatorio de 256 bits; se guarda en `sessions` su **hash SHA-256**, el usuario, la expiración absoluta (12 h) y el último uso.
- El token en claro solo viaja en la cookie `HttpOnly`, `SameSite=Lax`, `Secure` en producción.
- Cada petición autenticada resuelve la sesión por hash, verifica expiración y estado activo del usuario, y carga sus roles.
- Logout y desactivación de usuario marcan la sesión como revocada; la verificación ocurre en cada petición, por lo que la revocación es inmediata.

**Alternativa descartada:** JWT sin estado. No permite revocar y obliga a exponer el token a JavaScript o a construir una lista negra, que reintroduce estado.

### D5. Manejo del tiempo

- Toda hora de entrada se parsea como hora local `America/Bogota` (`time.LoadLocation`) y se convierte a `time.Time` en UTC antes de llegar al dominio.
- El dominio opera exclusivamente con instantes UTC; la conversión a hora local ocurre solo en el adaptador HTTP de salida y en el frontend.
- El reloj se inyecta mediante el puerto `Clock` para que las reglas de anticipación, plazos de cancelación y "finalizada" sean deterministas en pruebas.

**Decisión:** un único puerto `Clock` en lugar de llamar a `time.Now()` disperso. Sin él, las pruebas de plazos dependerían del reloj real y serían frágiles.

### D6. Cálculo de disponibilidad y solapamiento

- El dominio representa los rangos con un tipo `Interval{Start, End time.Time}` y una función pura `Overlaps(a, b)` con semántica semiabierta `[start, end)`.
- La disponibilidad se calcula intersectando el rango consultado (recortado al horario operativo del día) con las reservas aprobadas, y devolviendo las franjas libres complementarias.
- El cálculo en el dominio se usa para la respuesta de disponibilidad; la comprobación de conflicto al aprobar se apoya en la consulta SQL y, en última instancia, en la restricción.

**Decisión:** una sola definición de solapamiento en el dominio, reutilizada por disponibilidad y por los mensajes de conflicto. Duplicar la regla en SQL y en Go es inevitable, pero la fuente semántica es una y las pruebas de borde cubren ambas implementaciones.

### D7. Taxonomía de errores y mapeo HTTP

| Error de dominio | HTTP | Uso |
|---|---|---|
| `ErrValidation` | 400 | Datos mal formados o fuera de rango |
| `ErrUnauthenticated` | 401 | Sin sesión válida |
| `ErrForbidden` | 403 | Sesión válida sin permiso |
| `ErrNotFound` | 404 | Recurso inexistente o no visible |
| `ErrSlotConflict` | 409 | Solapamiento o estado incompatible |
| `ErrStateConflict` | 409 | Transición no permitida |
| `ErrRateLimited` | 429 | Intentos de login excesivos |
| `ErrInternal` | 500 | Fallo no clasificado |

Cuerpo uniforme: `{"error": {"code": "...", "message": "..."}}`, en español y sin detalles internos.

**Decisión:** el dominio devuelve errores tipados; solo el adaptador HTTP conoce códigos de estado. Evita que la capa de aplicación dependa de HTTP.

### D8. Autorización

- Permisos atómicos (`users:manage`, `spaces:manage`, `bookings:create`, `bookings:read:own`, `bookings:read:all`, `bookings:decide`, `bookings:cancel:any`).
- Los roles se mapean a conjuntos de permisos en el dominio; un usuario tiene la unión.
- Un middleware resuelve sesión → usuario → permisos y los inyecta en el contexto de la petición; cada handler declara el permiso requerido.
- Las verificaciones de propiedad (soy autor) y de autoaprobación son reglas de dominio, no del middleware.

**Decisión:** separar permiso grueso (middleware) de regla de negocio (dominio). Así la autoaprobación se prueba en el dominio sin HTTP.

### D9. Frontend

- `src/services/http.ts`: cliente único con `credentials: 'include'`, base URL por variable de entorno y normalización de errores a un tipo `ApiError`.
- `src/auth`: contexto de sesión con usuario y permisos; la interfaz oculta acciones no permitidas, pero la autorización real es del backend.
- Rutas: `/login`, `/espacios`, `/espacios/:id/disponibilidad`, `/reservas`, `/reservas/nueva`, `/aprobaciones`, `/admin/usuarios`, `/admin/espacios`.
- Estado por página con hooks; sin librería de estado global.
- Pruebas con Vitest + RTL sobre los estados de carga, error, vacío y permisos.

**Decisión:** sin gestor de estado externo. El alcance no justifica una dependencia adicional (R-06) y los datos son por página.

### D10. Estrategia de pruebas por nivel

| Nivel | Mecanismo | Aislamiento |
|---|---|---|
| Dominio | `go test` con tablas de casos y `Clock` fijo | Sin I/O |
| Aplicación | Dobles en memoria de los puertos | Sin base de datos |
| Repositorios | PostgreSQL real efímero, migraciones aplicadas al inicio | Base creada y destruida por suite |
| HTTP | `httptest` sobre el router con repositorios reales o dobles | Por suite |
| Concurrencia | N goroutines sobre la misma base, `sync.WaitGroup` | Base real |
| Frontend | Vitest + RTL, `fetch` simulado | Por componente |

**Decisión:** los repositorios se prueban contra PostgreSQL real, nunca contra un doble, porque el objeto de la prueba (restricción de exclusión y códigos de error) es específico del motor.

## Risks / Trade-offs

- **La restricción de exclusión requiere `btree_gist` y falla si la extensión no está disponible** → la migración la crea de forma explícita y el arranque verifica su existencia, fallando con un mensaje claro.
- **Conflictos frecuentes al aprobar cuando muchas solicitudes pendientes compiten por la misma franja** → aceptado por la política S-04: la respuesta 409 informa el rango en conflicto y el solicitante puede reintentar con otro horario. Se documenta como comportamiento esperado, no como error del sistema.
- **Duplicación semántica del solapamiento (Go y SQL)** → pruebas de borde idénticas en ambos niveles (rangos contiguos, contenidos, idénticos, cruce de medianoche).
- **Bloqueo por fila en la aprobación puede serializar decisiones sobre la misma reserva** → deseado: impide doble decisión. El bloqueo se limita a la fila de la reserva, no al espacio, para no serializar aprobaciones de espacios distintos.
- **Cookies `SameSite=Lax` y API JSON** → suficiente para el alcance; en producción se exige HTTPS y `Secure`. No se implementa CSRF token porque no hay formularios cross-site con credenciales y el método de autenticación es cookie de sesión de misma origen.
- **Sin paginación en la bandeja de pendientes en la primera versión** → se añade paginación al historial desde el inicio (requisito) y se deja la bandeja ordenada y acotada por el volumen académico esperado.
- **Dependencia del entorno ausente (Go, Node, PostgreSQL)** → la migración y el arranque no pueden verificarse hasta resolverlo; las tareas lo reflejan como precondición.

## Migration Plan

1. `0001_init.up.sql`: extensiones, tablas, índices y restricciones (incluida la de exclusión).
2. `0002_seed_roles.up.sql`: catálogo de roles y sus permisos.
3. Comando `cmd/api` con subcomando de bootstrap (`-bootstrap-admin`) que crea el primer administrador desde variables de entorno, sin exponer endpoint público.
4. Verificación de migración desde base limpia: `golang-migrate up` → `down` → `up`, y prueba automática que aplica todas las migraciones sobre una base vacía.
5. Reversión: cada migración incluye su `.down.sql`; al ser un sistema nuevo, revertir equivale a eliminar el esquema y volver a migrar.

## Open Questions

- **Calendario de festivos — resuelto (2026-11-24, aprobado por el estudiante).** Se implementa una tabla `holidays` sembrada por migración con los festivos colombianos del año en curso, sin interfaz ni API de administración en el MVP. La actualización anual se realiza mediante una nueva migración. Queda documentado como limitación conocida en el README.
- **Política de intentos de inicio de sesión.** Se aplicará límite simple por cuenta e IP (429) para evitar fuerza bruta; el umbral exacto se ajusta en implementación y no altera los requisitos.
- **Semilla de demostración.** Si se crean usuarios y espacios de demostración, deberán ser explícitamente sintéticos y documentados en el README; su contenido no afecta a las especificaciones.
