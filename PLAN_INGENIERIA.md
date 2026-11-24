# Plan de Ingeniería

## Sistema de gestión de reservas de espacios académicos

| Campo | Valor |
|---|---|
| Documento | `PLAN_INGENIERIA.md` |
| Versión | 1.0 (borrador para aprobación) |
| Fecha | 2026-11-24 |
| Estado | **Pendiente de aprobación del estudiante** |
| Alcance del documento | Planificación de ingeniería. No describe funcionalidades implementadas. |
| Autor del proceso | Estudiante (responsable de decisiones) asistido por DeepSeek Harness |
| Repositorio | `exposicion_uninpahu` |

> **Naturaleza de este documento.** Todo lo que aparece aquí en tiempo futuro o condicional es un plan, no una afirmación de que exista implementación. Al momento de redactarlo, el repositorio contiene únicamente `README.md` (título del proyecto) y este archivo. No hay backend, frontend, migraciones ni pruebas.
>
> **Regla de proceso.** Este plan se somete a aprobación explícita antes de cualquier trabajo de especificación (FASE 2) e implementación (FASE 3). La regla institucional de duración máxima de reserva se gestionará como cambio formal posterior (FASE 4) y **no** se incorpora como requisito del MVP.

---

# 1. Análisis del problema

## 1.1 Situación actual y causas

La institución administra hoy las reservas de laboratorios, auditorios y salas de cómputo mediante correo electrónico y hojas de cálculo. El proceso funciona porque hay personas que lo sostienen manualmente, pero no porque el sistema lo garantice.

**Causas del problema actual**

| Causa | Efecto observable |
|---|---|
| No existe un registro único de solicitudes | Dos personas reservan el mismo espacio y hora; el conflicto se descubre tarde. |
| La disponibilidad no es consultable | El solicitante pregunta por correo y espera respuesta; se duplican consultas. |
| La validación de choques es manual | Depende de la atención y memoria de quien revisa la hoja de cálculo. |
| Los estados no están formalizados | No se distingue con claridad una solicitud pendiente de una reserva aprobada. |
| No hay historial confiable | Al auditar o reclamar, no se puede reconstruir qué se aprobó, cuándo y por quién. |
| Permisos informales | Cualquier persona con acceso al archivo puede modificar filas ajenas. |
| Falta de trazabilidad del cambio | No hay registro de ni un cambio de estado ni de su responsable. |

**Consecuencia central:** el problema no es solo de comodidad, sino de **consistencia**. El estado de ocupación de un espacio no tiene una fuente de verdad, y por tanto no es verificable.

## 1.2 Necesidades institucionales

Se identifican como necesidades del producto (identificadores usados luego en la matriz de trazabilidad y en las especificaciones de FASE 2):

| ID | Necesidad |
|---|---|
| N-01 | Un catálogo único de espacios académicos con sus atributos (tipo, capacidad, ubicación, estado). |
| N-02 | Administración de usuarios con registro, consulta, actualización y desactivación. |
| N-03 | Roles y permisos verificados en el servidor, no en el cliente. |
| N-04 | Creación de solicitudes de reserva con espacio, fecha, hora de inicio y hora de fin. |
| N-05 | Consulta de disponibilidad real de un espacio para un rango de tiempo. |
| N-06 | Imposibilidad de que dos reservas aprobadas se solapen en el mismo espacio. |
| N-07 | Revisión de solicitudes pendientes con decisión de aprobación o rechazo motivado. |
| N-08 | Cancelación de reservas conforme a una política explícita. |
| N-09 | Historial consultable de solicitudes y reservas, propio y global según rol. |
| N-10 | Trazabilidad de los cambios de estado (quién, cuándo, de qué estado a cuál). |

## 1.3 Actores y responsabilidades

| Actor | Responsabilidades | Permisos principales |
|---|---|---|
| **Administrador** | Mantener usuarios, roles y espacios; consultar reservas e historial global. | CRUD de usuarios, asignación de roles, CRUD de espacios, lectura global de reservas. |
| **Solicitante** | Consultar espacios y disponibilidad; crear solicitudes; consultar y cancelar sus propias reservas según política. | Lectura de espacios y disponibilidad; creación de solicitudes; cancelación de las propias; lectura de su historial. |
| **Aprobador** | Revisar solicitudes pendientes; aprobar o rechazar con motivo. | Lectura de solicitudes pendientes y de disponibilidad; aprobar/rechazar. |
| **Sistema** | Validar reglas, disponibilidad, permisos y consistencia de cada operación. | No es un actor humano: actúa como garante de invariantes. |

**Sobre la multiplicidad de roles.** El documento fuente indica que no debe asumirse que los actores son personas distintas. Este plan **admite que un usuario acumule varios roles** (por ejemplo, Aprobador y Solicitante) mediante una relación N:M entre usuarios y roles, y concentra la autorización en permisos derivados de roles, no en un único campo de rol. La versión sin acumulación sería más simple, pero menos fiel a la política institucional; ver supuesto **S-08** para confirmación.

## 1.4 Alcance funcional

**Dentro del MVP**

1. Autenticación por credenciales y sesión revocable.
2. Gestión de usuarios y asignación de roles (Administrador).
3. Gestión de espacios académicos con estado activo/inactivo (Administrador).
4. Consulta de espacios y de disponibilidad por rango de tiempo (autenticados).
5. Creación de solicitudes de reserva en estado *pendiente*.
6. Bandeja de solicitudes pendientes y decisión de aprobación o rechazo con motivo (Aprobador).
7. Prevención de solapamientos garantizada en la base de datos para reservas aprobadas.
8. Cancelación de solicitudes y reservas conforme a política.
9. Historial de reservas por usuario y global para Administrador.
10. Registro de transiciones de estado para trazabilidad.

**Prioridad** — Must: 1, 2, 3, 5, 6, 7, 8. Should: 4, 9, 10. La consulta de disponibilidad (4) se marca *Should* solo porque su valor depende de datos reales; su ausencia haría el producto poco útil, por lo que se implementa en la misma entrega que la creación de solicitudes.

## 1.5 Exclusiones y limitaciones

**Fuera de alcance del MVP** (no se implementa; si se solicita, entra como cambio formal):

- Pagos, facturación o cualquier transacción económica.
- Notificaciones por correo, SMS o push.
- Integración con calendarios institucionales (Google Calendar, Outlook, ICS).
- Reservas recurrentes o periódicas.
- Aplicación móvil nativa.
- Carga masiva de usuarios o espacios desde archivos.
- Recuperación de contraseña por correo o autoservicio de registro público.
- Firma electrónica, aprobaciones multinivel o comités.
- Panel de analítica avanzada o reportería exportable.

**Limitaciones conocidas del alcance**

- El MVP no impone un límite a la duración de una reserva individual. Cualquier límite futuro se gestionará como cambio formal con propuesta, especificación y pruebas, no como ajuste silencioso.
- La aprobación es de un solo nivel: un Aprobador basta.
- No se modela jerarquía organizacional (facultades, programas) más allá de los atributos del usuario.
- La detección de solapamientos se define por espacio y por rango temporal; no considera recursos adicionales (proyectores, equipos) como entidades reservables.
- El sistema es de una sola institución: no es multi-tenant.

## 1.6 Supuestos y restricciones

Estos supuestos se proponen como valores razonables y **requieren confirmación**. Ninguno pretende inventar política institucional.

| ID | Supuesto propuesto | Impacto si es incorrecto | Estado |
|---|---|---|---|
| **S-01** | Zona horaria de operación: `America/Bogota` (UTC-5, sin horario de verano). El servidor almacena instantes en UTC y muestra en hora local. | Alto: cambia el cálculo de rangos y solapamientos. | Por confirmar |
| **S-02** | Horario operativo: lunes a viernes 06:00–22:00, sábados 07:00–18:00, domingos y festivos cerrado. | Medio: solo afecta validaciones de conveniencia, no la integridad. | Por confirmar |
| **S-03** | Anticipación mínima para solicitar: 24 horas; máxima: 90 días. | Bajo: regla de negocio ajustable. | Por confirmar |
| **S-04** | Solo las reservas **aprobadas** bloquean disponibilidad. Una solicitud pendiente **no** bloquea, y su disponibilidad se revalida al aprobar. | Alto: define la semántica de N-06 y el riesgo de doble aprobación. Ver D-05. | Por confirmar |
| **S-05** | Un Aprobador no puede aprobar su propia solicitud. | Medio: afecta la segregación de funciones. | Por confirmar |
| **S-06** | Cancelación: el solicitante puede cancelar una solicitud pendiente en cualquier momento; una reserva aprobada solo hasta 24 horas antes del inicio. El Administrador puede cancelar sin esa restricción. | Medio: reglas de N-08. | Por confirmar |
| **S-07** | Motivo obligatorio al rechazar; opcional al cancelar por parte del solicitante. | Bajo. | Por confirmar |
| **S-08** | Un usuario puede tener más de un rol simultáneamente. | Medio: simplifica a un rol por usuario si se rechaza. | Por confirmar |
| **S-09** | Espacios desactivados no admiten nuevas solicitudes, pero conservan sus reservas históricas. | Bajo. | Por confirmar |
| **S-10** | Duración mínima de una reserva: 30 minutos; granularidad de 15 minutos. | Bajo. | Por confirmar |

**Restricciones del proyecto**

| ID | Restricción |
|---|---|
| R-01 | Proyecto académico individual: debe ser pequeño, funcional y demostrable. Sin microservicios. |
| R-02 | Stack obligatorio: Go (backend), React + TypeScript (frontend), PostgreSQL (persistencia). |
| R-03 | Sin datos reales, contraseñas reales ni secretos en el repositorio. |
| R-04 | Sin `git push` ni publicación de releases sin autorización explícita del estudiante. |
| R-05 | Prohibido desactivar validaciones, linters o pruebas para obtener una ejecución aparentemente exitosa. |
| R-06 | Ninguna dependencia se introduce sin justificación escrita (ver sección 3.6). |
| R-07 | Cada fase respeta su puerta de aprobación; no se adelanta trabajo de fases posteriores. |

**Nota de entorno (hallazgo de FASE 0).** En la máquina de desarrollo están disponibles `git` 2.51.2 y `openspec` 2.1.1. **No** están instalados Go, Node.js, npm, Docker ni `psql`. Esto no bloquea la planificación ni la especificación, pero **bloquea la FASE 3** hasta resolverlo. Es una restricción de entorno, no una decisión de arquitectura.

---

# 2. Propuesta de solución

## 2.1 Descripción de la aplicación

Una aplicación web centralizada, de una sola institución, que reemplaza el circuito de correo y hojas de cálculo por un flujo con estado explícito y fuente de verdad única en PostgreSQL:

1. El **Administrador** mantiene usuarios, roles y espacios académicos.
2. El **Solicitante** consulta qué espacios existen y su disponibilidad real para un rango, y crea una solicitud, que nace **pendiente**.
3. El **Aprobador** revisa la bandeja de pendientes y aprueba o rechaza con motivo.
4. Al aprobar, el sistema verifica de nuevo la disponibilidad y persiste la reserva de forma que **la base de datos**, no solo el código de aplicación, impida el solapamiento.
5. Cualquier actor autorizado consulta el historial; cada transición queda registrada con autor y fecha.

El valor demostrable no es el volumen de funcionalidades, sino que **las invariantes críticas (permisos y no-solapamiento) se sostienen aunque el cliente mienta o dos peticiones lleguen a la vez**.

## 2.2 Funcionalidades prioritarias

| Prioridad | Funcionalidad | Justificación |
|---|---|---|
| 1 | Autenticación y sesión | Sin identidad no hay autorización ni historial atribuible. |
| 2 | Espacios (CRUD + estado) | Es la entidad que se reserva; sin ella no hay producto. |
| 3 | Consulta de disponibilidad | Es la necesidad que hoy provoca más correos. |
| 4 | Solicitudes (crear, listar propias) | Corazón del flujo. |
| 5 | Aprobación/rechazo | Convierte solicitud en reserva de forma controlada. |
| 6 | No-solapamiento en base de datos | Invariante central; es el criterio de calidad más exigente. |
| 7 | Cancelación | Cierra el ciclo de vida y libera disponibilidad. |
| 8 | Historial y auditoría de transiciones | Trazabilidad exigida por la sustentación. |

## 2.3 Módulos y responsabilidades

**Backend (hexagonal)**

| Módulo | Responsabilidad | Depende de |
|---|---|---|
| `domain/identity` | Usuario, Rol, permisos, invariantes de credenciales. | — |
| `domain/spaces` | Espacio académico, tipo, capacidad, estado. | — |
| `domain/booking` | Solicitud/Reserva, estados, transiciones válidas, reglas de duración mínima y anticipación. | `domain/identity`, `domain/spaces` (por identificadores) |
| `domain/availability` | Cálculo de solapamiento y de franjas disponibles; función pura sobre intervalos. | `domain/booking` |
| `application/*` | Casos de uso: autenticar, gestionar usuarios/espacios, crear solicitud, aprobar, rechazar, cancelar, consultar historial. Orquestan transacciones. | Dominio + puertos |
| `ports` | Interfaces de repositorios (`UserRepository`, `SpaceRepository`, `BookingRepository`, `SessionRepository`) y de servicios (`Clock`, `PasswordHasher`, `IDGenerator`). | — |
| `adapters/http` | Rutas, DTOs, autenticación, mapeo de errores a códigos HTTP, middleware de autorización. | Aplicación |
| `adapters/postgres` | Implementación de repositorios con `pgx`; consultas parametrizadas; uso de transacciones y de la restricción de exclusión. | Puertos |
| `cmd/api` | Composición (inyección manual de dependencias), configuración, arranque y apagado ordenado. | Adaptadores |

**Frontend**

| Módulo | Responsabilidad |
|---|---|
| `pages` | Login, espacios, disponibilidad, mis reservas, bandeja de aprobación, administración de usuarios y espacios. |
| `components` | Formularios, tablas, indicadores de estado, mensajes de error y estados de carga/vacío. |
| `services/http` | Cliente HTTP centralizado: base URL, credenciales, manejo uniforme de errores. |
| `types` | Tipos TypeScript del contrato de API. |
| `auth` | Contexto de sesión y visibilidad de acciones por permiso (solo presentación, nunca autorización real). |
| `tests` | Pruebas de componentes y flujos con Vitest + React Testing Library. |

**Criterio de frontera.** La lógica de negocio (estados, permisos, solapamiento, anticipación) vive en `domain` y `application`. Ni HTTP, ni React, ni `pgx` pueden contener reglas de negocio.

## 2.4 Entregas incrementales

| Entrega | Contenido | Resultado demostrable |
|---|---|---|
| **E1** | Esqueleto del repositorio, `go.mod`, configuración por entorno, endpoint de salud, CI mínima local. | La API arranca y responde salud; el frontend compila. |
| **E2** | Migraciones SQL, conexión a PostgreSQL, repositorios base, verificación de migración desde base limpia. | Base de datos reproducible y repositorios probados contra PostgreSQL real. |
| **E3** | Dominio y casos de uso de usuarios, roles y espacios. | Reglas de identidad y espacios probadas sin HTTP ni base de datos. |
| **E4** | Autenticación, sesión y autorización por permiso aplicada en cada endpoint protegido. | Acceso denegado correctamente probado por rol. |
| **E5** | Solicitudes de reserva y máquina de estados. | Crear, listar y transicionar con validación de transiciones ilegales. |
| **E6** | Disponibilidad y no-solapamiento (restricción de exclusión + transacción). | Prueba concurrente: dos aprobaciones simultáneas no terminan ambas confirmadas. |
| **E7** | Aprobación, rechazo, cancelación e historial con auditoría de transiciones. | Flujo completo de negocio por API. |
| **E8** | Frontend: páginas, formularios, estados de carga/error/vacío, visibilidad por permiso. | Un usuario recorre el flujo completo desde la interfaz. |
| **E9** | Integración y pruebas de extremo a extremo; documentación de ejecución. | `README.md` reproducible y suites verdes. |

Cada entrega termina con pruebas propias; ninguna entrega se cierra sin pruebas.

---

# 3. Arquitectura y tecnología

## 3.1 Arquitectura hexagonal: capas y regla de dependencia

Se adopta arquitectura hexagonal (puertos y adaptadores) con cuatro zonas y una única dirección de dependencia:

```
adapters (http, postgres)  ──▶  application  ──▶  domain
                                    │                 ▲
                                    └──▶ ports ───────┘   (implementados por adapters)
```

- **Dominio:** entidades, tipos, invariantes. Sin imports de infraestructura. Funciones puras donde sea posible (intervalos, estados, permisos).
- **Aplicación:** casos de uso, transacciones, orquestación, decisiones de autorización sobre operaciones.
- **Puertos:** interfaces que el dominio/aplicación necesitan y que la infraestructura satisface. Permiten sustituir PostgreSQL por dobles de prueba.
- **Adaptadores:** HTTP y PostgreSQL. Traducen; no deciden.

**Por qué no se fuerzan capas vacías:** no habrá entidades sin comportamiento, ni un `service` que solo reenvíe al repositorio, ni mapeos ceremoniales. Si una capa no aporta, no se crea.

## 3.2 Estructura de carpetas propuesta

```text
exposicion_uninpahu/
├── AGENT_GUIDE.md
├── PLAN_INGENIERIA.md
├── README.md
├── INFORME_FINAL.md
├── docs/
│   ├── decisions/
│   └── traceability/
├── openspec/          # FASE 2, no antes
├── backend/
│   ├── cmd/api/
│   ├── internal/
│   │   ├── domain/{identity,spaces,booking,availability}/
│   │   ├── application/
│   │   ├── ports/
│   │   └── adapters/{http,postgres}/
│   ├── migrations/
│   └── go.mod
├── frontend/
│   ├── src/
│   ├── tests/
│   └── package.json
└── .gitignore
```

## 3.3 Backend: decisiones técnicas

| Aspecto | Decisión | Motivo breve |
|---|---|---|
| Lenguaje | Go (versión estable disponible en el entorno) | Ver D-01. |
| HTTP | Biblioteca estándar `net/http` con el enrutador de patrones por método y ruta (`GET /api/spaces/{id}`) introducido en Go 1.22 | Evita dependencia de framework; suficiente para ~15 endpoints. Ver D-07. |
| Driver PostgreSQL | `github.com/jackc/pgx/v5` (`pgxpool`) | Driver mantenido, soporte nativo de rangos y de `EXCLUDE` vía SQL, mejor manejo de tipos y de contexto. Ver D-08. |
| Migraciones | Archivos SQL versionados y aplicados con `golang-migrate` como herramienta de desarrollo | Reproducibles y auditables; ver D-09. |
| Configuración | Variables de entorno validadas al arranque; sin valores por defecto para secretos | Falla rápido y evita secretos en Git. |
| Contraseñas | `bcrypt` (coste ≥ 12) mediante `golang.org/x/crypto/bcrypt` | Algoritmo estándar, resistente a fuerza bruta; nunca texto plano. |
| Validación | En el borde HTTP (forma/tipos/tamaños) y en dominio (reglas) | El backend es la autoridad; el cliente nunca se confía. |
| Errores | Taxonomía de dominio → mapeo explícito a HTTP (400/401/403/404/409/422/500); cuerpo JSON uniforme | Coherencia y no filtración de detalles internos. |
| Registro | `log/slog` estructurado; nunca registrar contraseñas ni tokens | Observabilidad sin filtrar datos sensibles. |

## 3.4 Persistencia, transacciones y prevención de solapamientos

**Modelo de datos propuesto**

| Tabla | Propósito |
|---|---|
| `users` | Identidad, credenciales (hash), estado activo, datos de contacto institucional. |
| `roles` | Catálogo de roles (administrador, solicitante, aprobador). |
| `user_roles` | Relación N:M usuario-rol (soporta S-08). |
| `spaces` | Espacio académico: nombre, tipo, capacidad, ubicación, estado activo. |
| `bookings` | Solicitud y reserva en una sola entidad con `status` y rango temporal `[starts_at, ends_at)`. |
| `booking_events` | Bitácora inmutable: transición, autor, fecha, motivo. |
| `sessions` | Sesión opaca revocable: token hasheado, usuario, expiración, revocación. |

**Una sola entidad con estado, no dos tablas.** Una solicitud y una reserva aprobada son el mismo registro en dos momentos de su ciclo de vida. Modelarlas como entidades separadas duplicaría datos del intervalo y abriría la posibilidad de inconsistencia entre ambas; el estado explícito evita además confundir *pendiente* con *aprobada* (N-04, S-04).

**Estados y transiciones**

```
                 ┌────────────── aprobar ──────────────┐
                 │                                     ▼
  (crear) ──▶ PENDIENTE ──── rechazar ──▶ RECHAZADA            APROBADA
                 │                                     │  ▲
                 └────────── cancelar ──▶ CANCELADA ◀──┘  └── cancelar ──┘
```

| Estado | Bloquea disponibilidad | Transiciones permitidas |
|---|---|---|
| `pendiente` | **No** (S-04) | → `aprobada`, `rechazada`, `cancelada` |
| `aprobada` | **Sí** | → `cancelada` |
| `rechazada` | No | (terminal) |
| `cancelada` | No | (terminal) |

`rechazada` y `cancelada` son terminales. Una reserva cuyo fin ya pasó no cambia de estado almacenado: su carácter de "finalizada" es derivado por fecha, no un quinto estado que alguien deba mantener.

**Prevención de solapamientos (la invariante central)**

La garantía se implementa en PostgreSQL mediante una restricción de exclusión sobre rangos temporales, no solo con una comprobación previa en código:

```sql
CREATE EXTENSION IF NOT EXISTS btree_gist;

ALTER TABLE bookings
  ADD CONSTRAINT bookings_no_overlap
  EXCLUDE USING gist (
    space_id WITH =,
    tstzrange(starts_at, ends_at, '[)') WITH &&
  ) WHERE (status = 'aprobada');
```

Propiedades que esto aporta:

1. **Atomicidad.** Si dos transacciones intentan aprobar intervalos solapados del mismo espacio, una de ellas falla con violación de restricción; no existe ventana de carrera.
2. **Doble defensa.** La aplicación valida antes para dar un mensaje amigable (409 con el conflicto concreto) y la base de datos garantiza el invariante si esa validación se equivoca o dos peticiones llegan juntas.
3. **Consistencia con S-04.** El predicado parcial `WHERE status = 'aprobada'` hace que solo las aprobadas bloqueen, tal como exige la política.
4. **Rangos semiabiertos `[)`.** Una reserva que termina a las 10:00 y otra que empieza a las 10:00 no se solapan, que es el comportamiento correcto.

**Transacciones.** Aprobar, rechazar y cancelar son operaciones de escritura sobre el estado más inserción en `booking_events`; se ejecutan en una transacción. La aprobación revalida disponibilidad y actualiza estado dentro de la misma transacción, de modo que la decisión y su evidencia histórica nunca se separan.

**Condición de carrera residual.** Con la restricción de exclusión activa, el peor caso de concurrencia es que una segunda aprobación falle y deba reintentarse o informarse al usuario; nunca es que dos reservas aprobadas coexistan. Esto debe demostrarse con una prueba concurrente (sección 5.3).

## 3.5 Seguridad y configuración

| Control | Medida |
|---|---|
| Contraseñas | `bcrypt` con coste ≥ 12; nunca texto plano ni hashes expuestos en respuestas o logs. |
| Sesión | Token opaco aleatorio (≥ 128 bits), almacenado **hasheado** en `sessions`, enviado en cookie `HttpOnly`, `Secure` en producción, `SameSite=Lax`; expiración absoluta y por inactividad; revocable. Ver D-06. |
| Autorización | Verificación por permiso en cada endpoint protegido, derivada de roles en el servidor; el rol enviado por el cliente se ignora siempre. |
| Inyección SQL | Exclusivamente consultas parametrizadas; sin concatenación de SQL. |
| Entrada | Validación de forma y tamaño; límite de tamaño de cuerpo; rechazo de campos desconocidos en DTOs sensibles. |
| Transporte | HTTPS en despliegue; en desarrollo local, HTTP en `localhost`. |
| Secretos | Solo por variables de entorno; `.env.example` sin valores reales; `.env` en `.gitignore`. |
| CORS | Origen del frontend explícito; credenciales permitidas solo para ese origen. |
| Errores | Mensajes genéricos hacia el cliente para fallos internos; detalle solo en logs. |
| Auditoría | `booking_events` registra autor y fecha de cada transición. |

**Variables de entorno previstas** (`APP_ENV`, `HTTP_ADDR`, `DATABASE_URL`, `SESSION_TTL`, `COOKIE_SECURE`, `FRONTEND_ORIGIN`, `BCRYPT_COST`, `LOG_LEVEL`).

## 3.6 Comparación de decisiones y alternativas descartadas

### D-01 — Lenguaje del backend: Go frente a Node.js y PHP

| Aspecto | Go (elegido) | Node.js + Express | PHP + Laravel |
|---|---|---|---|
| Seguridad de tipos | Tipado estático, compilado | Dinámico (TS ayuda, no garantiza en runtime) | Dinámico |
| Concurrencia | Goroutines y contexto nativos; ideal para validación y bloqueo por transacción | Modelo asíncrono de un solo hilo; carga cognitiva | Modelo por proceso/petición |
| Despliegue | Un binario único, sin runtime | Requiere runtime y `node_modules` | Requiere PHP-FPM y servidor web |
| Ecosistema web | Menor, pero suficiente con biblioteca estándar | Muy amplio | Muy amplio (Laravel) |
| Rendimiento | Alto con bajo consumo de memoria | Bueno | Adecuado |

**Ventajas de Go:** binario único, arranque rápido, tipado estricto, concurrencia idiomática, excelente biblioteca estándar para HTTP y SQL, y una curva que favorece demostrar separación de capas sin magia de framework.
**Desventajas de Go:** menos "baterías incluidas" (validación, ORM) y ecosistema de frontend nulo; obliga a escribir más código explícito.
**Justificación:** el proyecto exige demostrar reglas de dominio aisladas y concurrencia correcta. Go hace explícitas las dependencias y no oculta transacciones ni errores detrás de un ORM, lo cual conviene a un ejercicio de trazabilidad. Node.js y PHP habrían sido válidos, pero aportan más magia y menos garantías de tipo.

### D-02 — Persistencia: PostgreSQL frente a una base documental y frente a SQLite

| Aspecto | PostgreSQL (elegido) | MongoDB (documental) | SQLite |
|---|---|---|---|
| Integridad referencial | Sólida, con claves foráneas | Limitada; se resuelve en aplicación | Sólida |
| Concurrencia | MVCC, aislamiento configurable | Depende de configuración y del modelo | Un solo escritor; bloqueo global |
| Restricciones de exclusión sobre rangos | **Nativas** (`EXCLUDE USING gist` + `btree_gist`) | No existen; hay que implementarlas en código | No existen |
| Tipos de rango y zona horaria | `tstzrange`, `timestamptz` con zona | Fechas como tipo simple | Fechas como texto/número |
| Uso académico | Instalable localmente o por contenedor | Requiere servicio | Sin servidor |

**Ventajas de PostgreSQL:** la invariante "dos reservas aprobadas no se solapan" puede expresarse como una restricción declarativa de la base de datos, que es exactamente el nivel que el proyecto considera obligatorio. Aporta además `timestamptz` correcto y transacciones robustas.
**Desventajas:** requiere un servicio en ejecución; algo más de operación que SQLite.
**Por qué se descarta MongoDB:** la prevención de solapamientos quedaría solo en código de aplicación, sin garantía ante concurrencia; sería una decisión técnicamente más débil para la necesidad N-06.
**Por qué se descarta SQLite:** cómodo, pero su modelo de un escritor y la ausencia de restricciones de exclusión impedirían demostrar la invariante en el nivel exigido, y no representa el entorno institucional.

### D-03 — Arquitectura: hexagonal frente a monolito en capas sin puertos

| Aspecto | Hexagonal (elegido) | Capas clásicas (controller → service → repository) |
|---|---|---|
| Dirección de dependencias | Hacia el dominio; infraestructura al borde | La lógica suele acabar dependiendo del framework o del ORM |
| Testabilidad | Reglas puras probables sin base de datos | Requiere dobles sobre clases concretas |
| Sustitución de infraestructura | Interfaz explícita | Acoplamiento implícito |
| Coste inicial | Mayor: definir puertos y composición | Menor |
| Riesgo de sobreingeniería | Controlable no creando capas vacías | Bajo, pero invita a servicios anémicos |

**Ventajas:** las reglas de negocio (estados, permisos, solapamiento) quedan aisladas y verificables; el adaptador HTTP y el de PostgreSQL son reemplazables; se puede demostrar que el dominio no conoce ni HTTP ni SQL.
**Desventajas:** más archivos y ceremonia de composición; exige disciplina para no crear capas inútiles.
**Justificación:** el objetivo académico incluye *demostrar* que la lógica no depende de la infraestructura, y la prueba de solapamiento requiere probar dominio y adaptador por separado. Un monolito en capas sin puertos dificultaría ambas demostraciones.

### D-04 — Estilo de API: REST frente a GraphQL

| Aspecto | REST/JSON (elegido) | GraphQL |
|---|---|---|
| Coste de implementación | Bajo con `net/http` | Alto: esquema, resolvers, herramientas |
| Consultas flexibles | No; endpoints específicos | Sí |
| Caché y simplicidad de depuración | Natural con HTTP | Requiere estrategia propia |
| Superficie de autorización | Por endpoint, explícita | Por campo/resolver, más fácil de omitir |
| Ajuste al alcance | Bueno: recursos claros y flujos acotados | Sobredimensionado |

**Ventajas de REST:** alineado con recursos del dominio (usuarios, roles, espacios, reservas), autorización explícita por endpoint, fácil de probar con `httptest` y de consumir desde React.
**Desventajas:** sobre- o infra-obtención de datos; más endpoints para vistas compuestas.
**Justificación:** el MVP no tiene clientes heterogéneos que justifiquen GraphQL, y su coste reduce el tiempo disponible para lo que sí se evalúa: reglas, concurrencia y pruebas. GraphQL añadiría superficie de autorización difícil de verificar por campo.

### D-05 — Estrategia anti-solapamiento: restricción de exclusión frente a validación en aplicación y bloqueos

| Estrategia | Garantía bajo concurrencia | Complejidad | Veredicto |
|---|---|---|---|
| **Restricción de exclusión `EXCLUDE` con predicado sobre estado** | Total: la base de datos rechaza el solapamiento | Baja (una migración) | **Elegida** |
| Validar `SELECT` y luego `INSERT`/`UPDATE` sin bloqueo | Ninguna: dos transacciones pueden pasar la validación | Baja | Descartada: no cumple N-06 |
| `SELECT ... FOR UPDATE` sobre el espacio o sobre filas solapadas | Alta, si se bloquea de forma consistente | Media-alta: risk de deadlocks y de olvidar el bloqueo | Descartada como mecanismo principal |
| Bloqueo por asesor (`pg_advisory_xact_lock` por espacio) | Alta | Media: serializa por espacio | Descartada: innecesaria teniendo `EXCLUDE` |
| Comprobación solo en React | Ninguna | Trivial | Descartada explícitamente |

**Justificación:** la restricción de exclusión expresa el invariante de negocio como propiedad de los datos. La validación previa en la aplicación se conserva, pero solo como mejora de mensaje al usuario (409 con el conflicto), no como garantía. El bloqueo asesor se evita porque introduce serialización y riesgo de deadlocks sin aportar más que la restricción.

### D-06 — Autenticación: sesión opaca en base de datos frente a JWT

| Aspecto | Sesión opaca revocable (elegida) | JWT |
|---|---|---|
| Revocación inmediata | Sí, borrando o marcando la sesión | No sin lista negra |
| Estado en servidor | Sí (tabla `sessions`) | Sin estado |
| Exposición en cliente | Cookie `HttpOnly`, no accesible a JS | Habitualmente en memoria/localStorage (riesgoso) |
| Escalado horizontal | Requiere leer estado compartido | Trivial |
| Complejidad | Baja | Media (firma, rotación, expiración) |

**Justificación:** se prioriza poder revocar acceso (desactivar usuario, cerrar sesión) y no exponer el token a JavaScript. La escala horizontal no es un requisito de un proyecto individual, por lo que la principal ventaja de JWT no aplica.

### D-07 — Enrutador HTTP: `net/http` frente a frameworks

- **Elegido:** `net/http` con patrones de método y ruta (Go ≥ 1.22).
- **Alternativas:** `chi` (router idiomático, buen middleware) y Gin (framework con más utilidades y más acoplamiento).
- **Ventajas de la elección:** cero dependencias, suficiente para ~15 endpoints, sin capa oculta.
- **Desventajas:** middleware propio (autenticación, recuperación de pánicos, CORS) escrito a mano.
- **Justificación:** R-06 exige justificar dependencias; aquí no hace falta ninguna. Si el número de rutas creciera, `chi` sería el siguiente paso razonable.

### D-08 — Driver de PostgreSQL: `pgx` frente a `lib/pq`

- **Elegido:** `pgx/v5` con `pgxpool`.
- **Ventajas:** mantenimiento activo, mejor rendimiento, soporte de tipos ricos, `pgxpool` con contexto y salud del pool.
- **Desventajas:** API más amplia que la de `database/sql` puro. *Nota:* la línea base del guide menciona `database/sql`; `pgx` puede usarse a través de él si se prefiere, y esta elección debe quedar aprobada explícitamente (ver supuesto de decisión en la sección de cierre).
- **Justificación:** el proyecto depende de transacciones correctas y de errores tipados de PostgreSQL (violación de exclusión). `pgx` facilita detectar el código de error de la restricción y responder con 409.

### D-09 — Migraciones: SQL versionado con `golang-migrate` frente a ORM con migraciones automáticas

- **Elegido:** archivos `NNNN_descripcion.up.sql` / `.down.sql` aplicados por `golang-migrate`.
- **Ventajas:** el SQL de la restricción de exclusión es visible y revisable; reproducible desde base limpia; independiente del lenguaje.
- **Desventajas:** paso manual adicional en el flujo de desarrollo.
- **Por qué se evita un ORM con `AutoMigrate`:** ocultaría precisamente la parte más crítica del modelo (rangos, predicados, exclusiones) y dificultaría demostrar FASE 5 (migración desde base limpia).

---

# 4. Estrategia de desarrollo

## 4.1 Metodología: incremental, iterativa y guiada por especificaciones (SDD)

Se trabaja en incrementos pequeños, cada uno con: especificación aprobada previa (OpenSpec), tareas, implementación, pruebas y verificación. El ciclo de cada incremento es:

1. Leer las especificaciones y tareas aprobadas del cambio vigente.
2. Implementar únicamente lo que esas tareas describen.
3. Escribir o ampliar pruebas junto con el código (no al final).
4. Ejecutar formato, análisis estático y pruebas relevantes.
5. Revisar el diff.
6. Registrar el avance con un commit descriptivo y pequeño.
7. Informar qué requisito quedó cubierto y con qué prueba.

Ningún incremento avanza sin especificación aprobada. Si durante la implementación aparece una necesidad no especificada, se detiene el trabajo y se abre un cambio formal en lugar de decidirlo en el código.

## 4.2 Etapas y orden de implementación

| Etapa | Alcance | Depende de | Criterio de terminado |
|---|---|---|---|
| 1 | Esqueleto, configuración, salud | — | La API arranca, `/health` responde 200, `go vet` y `gofmt` limpios; existe `.gitignore` y `.env.example`. |
| 2 | Migraciones, conexión, repositorios | 1 | Migraciones aplican y revierten desde base limpia; repositorios probados contra PostgreSQL real. |
| 3 | Dominio de usuarios, roles y espacios | 2 | Pruebas unitarias del dominio sin base de datos ni HTTP; invariantes cubiertas. |
| 4 | Autenticación y autorización | 3 | Login, logout, sesión revocable; pruebas de permisos por endpoint (403 en denegados). |
| 5 | Solicitudes y máquina de estados | 4 | Transiciones válidas aceptadas; transiciones ilegales rechazadas con prueba. |
| 6 | Disponibilidad y no-solapamiento | 5 | Prueba concurrente demuestra que no pueden coexistir dos aprobadas solapadas. |
| 7 | Aprobación, rechazo, cancelación, historial | 6 | Flujo completo por API con auditoría de transiciones verificada. |
| 8 | Frontend y flujos | 7 | Recorrido completo en interfaz; estados de carga, error y vacío cubiertos por pruebas. |
| 9 | Integración y documentación | 8 | Flujos E2E verdes; `README.md` reproducible; matriz de trazabilidad completa. |

## 4.3 Dependencias entre módulos

```
config ─▶ database ─▶ repositories ─▶ use cases ─▶ http handlers
domain (identity, spaces) ─▶ domain/booking ─▶ domain/availability
identity/roles ─▶ autorización ─▶ todos los endpoints protegidos
```

Dependencias duras a respetar: no hay solicitudes sin usuarios y espacios; no hay aprobación sin control de disponibilidad; no hay frontend funcional sin contrato HTTP estable. Las dependencias **no** son asimétricas: el dominio nunca depende de adaptadores.

## 4.4 Criterios de terminado (Definition of Done) por etapa

Una etapa se considera terminada solo si:

- [ ] El código compila y `gofmt`/`go vet` (backend) o `tsc --noEmit`/lint (frontend) no reportan problemas.
- [ ] Las pruebas nuevas de la etapa pasan y las previas siguen pasando.
- [ ] Cada requisito tocado tiene al menos una prueba que lo verifica.
- [ ] Se revisó el diff antes de commitear.
- [ ] La documentación de ejecución se actualizó si cambió (README o variables de entorno).
- [ ] El commit es pequeño, descriptivo y no mezcla etapas.
- [ ] El informe de avance indica requirement cubierto y prueba que lo evidencia.

No se acepta "terminado" por apariencia: si no hay salida de pruebas, no está terminado.

## 4.5 Estrategia de ramas y commits

- Rama principal `main` protegida conceptualmente por convención: los incrementos se integran solo cuando las pruebas pasan.
- Ramas cortas por incremento con prefijo: `feat/E<n>-<tema>`, `fix/<tema>`, `docs/<tema>`, `test/<tema>`.
- **Commits atómicos**, en imperativo, con prefijo de tipo y referencia de la tarea o requisito cuando exista:
  - `docs: add initial engineering plan` (FASE 1, **commit independiente**)
  - `feat(booking): create pending booking request`
  - `test(availability): prove exclusion constraint blocks overlapping approvals`
- Prohibido mezclar en un mismo commit el plan, las especificaciones y la implementación: el historial debe permitir distinguir las fases.
- Sin `git push` ni reescritura de historial sin autorización explícita (R-04).
- Un commit no se corrige "ocultando" trabajo posterior: si algo se implementó por error, se documenta y se corrige en un commit visible.

---

# 5. Estrategia de calidad

## 5.1 Niveles de prueba

| Nivel | Alcance | Herramienta | Cuándo |
|---|---|---|---|
| Unitaria de dominio | Estados, transiciones, intervalos, permisos, reglas de anticipación y duración mínima | `go test` (tablas de casos) | Con cada regla de negocio |
| Unitaria de aplicación | Casos de uso con dobles de puertos | `go test` | Con cada caso de uso |
| Repositorio | SQL real contra PostgreSQL de pruebas | `go test` + contenedor/base de pruebas | Etapa 2 en adelante |
| HTTP/API | Rutas, validación, códigos, autenticación y permisos | `httptest` | Con cada endpoint |
| Concurrencia | Aprobaciones simultáneas del mismo espacio y rango | `go test` con goroutines y `sync.WaitGroup` | Etapa 6 |
| Frontend de componentes | Formularios, tablas, estados de carga/error/vacío, visibilidad por permiso | Vitest + React Testing Library | Con cada componente relevante |
| Integración/E2E | Autenticación → espacios → solicitud → aprobación → historial; rechazo; cancelación; conflicto | Suite de integración (API real + frontend o cliente HTTP) | Etapa 9 |

**Prohibiciones de calidad.** No se omite ni desactiva ninguna prueba para conseguir una ejecución verde. No se declara una prueba "pasada" sin haberla ejecutado y visto su salida.

## 5.2 Pruebas de autorización y validación

- Cada endpoint protegido tiene al menos: una prueba con usuario autorizado (éxito) y una con usuario sin permiso (403).
- Se prueba que un Solicitante no puede aprobar, que un Aprobador no administra usuarios y que nadie accede a recursos de otro usuario sin permiso.
- Se prueba que el rol enviado por el cliente se ignora: una petición con un rol falsificado en el cuerpo no eleva privilegios.
- Se prueban entradas inválidas: campos faltantes, tipos erróneos, fechas invertidas (fin ≤ inicio), texto excesivo, identificadores inexistentes.
- Se prueba que una reserva con inicio en el pasado o fuera de la anticipación permitida se rechaza.

## 5.3 Pruebas de concurrencia

Objetivo: demostrar que la invariante **N-06** se sostiene sin sincronización cooperativa.

Diseño de la prueba: N goroutines (por ejemplo 8) intentan aprobar el mismo espacio con el mismo rango, o rangos parcialmente solapados, contra la misma base de datos.

Criterios de aceptación de la prueba:

1. Exactamente **una** aprobación tiene éxito (o el número que corresponda a rangos no solapados).
2. Las demás fallan con conflicto (409 en HTTP; violación de restricción en el repositorio).
3. Al final, una consulta de conteo sobre aprobadas solapadas devuelve **cero** filas solapadas.
4. La prueba es repetible: se ejecuta varias veces y no es "verde por suerte".

## 5.4 Riesgos

| ID | Riesgo | Probabilidad | Impacto | Mitigación | Evidencia de verificación |
|---|---|---|---|---|---|
| **RG-01** | Condición de carrera en aprobaciones simultáneas que produzca doble reserva | Media | Crítico | Restricción de exclusión `EXCLUDE` (D-05) + transacción + revalidación al aprobar | Prueba concurrente de 5.3; consulta de solapados = 0 |
| **RG-02** | Escalada de privilegios por confiar en el rol del cliente o por omitir autorización en un endpoint | Media | Crítico | Autorización por permiso en middleware + prueba por endpoint; prohibición de leer rol del cliente | Suite de permisos: 403 por cada caso denegado |
| **RG-03** | Entorno incompleto (sin Go, Node, PostgreSQL en la máquina actual) que impida ejecutar y demostrar | **Alta** | Alto | Resolver instalación antes de FASE 3; documentar requisitos; no compensar con simulaciones | FASE 0 documentada; `go test ./...` y `npm run test` ejecutados realmente |
| **RG-04** | Desviación de alcance (añadir notificaciones, recurrentes, pagos) que rompa el cronograma | Media | Medio | Lista explícita de exclusiones (1.5); todo añadido pasa por cambio OpenSpec | Revisión de alcance contra especificaciones aprobadas |
| **RG-05** | Ambigüedad de política institucional (S-01…S-10) resuelta silenciosamente en el código | Media | Alto | Supuestos listados y aprobados antes de especificar; reglas en un solo lugar del dominio | Correspondencia supuesto → requisito → prueba |
| **RG-06** | Manejo inconsistente de zona horaria (UTC vs Bogotá) que desplace reservas | Media | Medio | `timestamptz` + cálculo en UTC + conversión solo en presentación | Pruebas de borde horario y de rangos que cruzan medianoche |
| **RG-07** | Documentación insuficiente para reproducir el sistema en FASE 5 | Media | Medio | README vivo actualizado en cada etapa que cambie la ejecución | Verificación de instalación desde limpio siguiendo el README |
| **RG-08** | Dependencia no justificada o pesada (framework/ORM) que oculte reglas | Baja | Medio | R-06: justificar cada dependencia; D-07/D-08/D-09 | Lista de dependencias de `go.mod`/`package.json` justificada |

## 5.5 Mecanismos de trazabilidad

- Cada requisito tendrá un identificador estable en las especificaciones OpenSpec (FASE 2).
- Cada prueba citará en su nombre o comentario el requisito que verifica (convención `TestApprove_RejectsOverlap_ReqN06`).
- `docs/traceability/MATRIZ_TRAZABILIDAD.md` relacionará necesidad → especificación → caso de uso/endpoint/componente → prueba → estado.
- `booking_events` aporta trazabilidad en tiempo de ejecución; la matriz aporta trazabilidad en tiempo de desarrollo.
- Cada commit referenciará el requisito o la tarea que avanza.

---

# 6. Uso de SDD e IA

## 6.1 Papel de OpenSpec

OpenSpec es el mecanismo formal de especificación y gestión de cambios. Concretamente:

- Guarda las especificaciones de capacidad en `openspec/specs/` como fuente de verdad de comportamiento esperado.
- Agrupa el trabajo en cambios (`openspec/changes/`) con propuesta, especificaciones delta, diseño y tareas.
- Permite que un cambio institucional (por ejemplo, una nueva regla de negocio) se documente, revise y apruebe **antes** de tocar código.
- Proporciona verificación de estructura y estado mediante sus propios comandos, de modo que la trazabilidad no dependa de documentos improvisados.

OpenSpec **no** sustituye a `PLAN_INGENIERIA.md`: el plan decide el "qué y por qué" de la ingeniería; OpenSpec lo descompone en capacidades, requisitos y escenarios verificables.

Según el guide, OpenSpec se inicializa en **FASE 2**, después de aprobado y commiteado este plan. En el entorno ya existe una carpeta `openspec/` sin seguimiento y sin especificaciones, creada antes de que este guide existiera; se tratará en FASE 2 mediante el flujo oficial de la versión instalada, conservando su estructura nativa y sin inventar comandos.

## 6.2 Del plan a capacidades, requisitos y escenarios

Capacidades previstas para FASE 2 (una especificación por capacidad, con propósito, requisitos numerados, reglas, validaciones, escenarios Given/When/Then, casos negativos y comportamiento ante errores y permisos insuficientes):

| Capacidad propuesta | Necesidades cubiertas |
|---|---|
| `gestion-espacios` | N-01 |
| `gestion-usuarios` | N-02 |
| `roles-permisos` | N-03 |
| `solicitudes-reserva` | N-04 |
| `consulta-disponibilidad` | N-05 |
| `prevencion-solapamientos` | N-06 |
| `aprobacion-rechazo` | N-07 |
| `cancelacion-reservas` | N-08 |
| `historial-reservas` | N-09 |
| `auditoria-estados` | N-10 |

Cada requisito del plan se convertirá en requisitos con identificador y en escenarios verificables; las reglas de negocio (S-01…S-10) deberán quedar explícitas como requisitos de política, no como constantes ocultas.

## 6.3 Actividades delegables al agente

- Inspección del repositorio y del entorno; diagnóstico.
- Redacción de borradores de documentos de planificación, especificaciones, diseño y tareas.
- Traducción de requisitos a escenarios Given/When/Then.
- Implementación de tareas ya aprobadas.
- Redacción de pruebas y ejecución de las suites.
- Refactorizaciones locales que no cambien comportamiento ni arquitectura.
- Actualización de documentación de ejecución.
- Verificación mecánica: formato, análisis estático, migraciones desde limpia, revisión de diff.

## 6.4 Revisión humana del código y las pruebas

El estudiante revisa antes de aceptar cualquier incremento:

- Que el diff corresponda exactamente a las tareas aprobadas y no incluya cambios colaterales.
- Que cada requisito tocado tenga pruebas que realmente lo verifiquen y que no estén vacías o triviales.
- Que no se hayan desactivado pruebas, linters o validaciones.
- Que no haya secretos, datos reales ni credenciales en el repositorio.
- Que las decisiones de arquitectura documentadas se respeten.
- Que las salidas de pruebas mostradas sean reales y reproducibles.

**El agente no aprueba su propio trabajo.** La aprobación de una fase o incremento es del estudiante.

## 6.5 Decisiones que conserva el estudiante

1. Aprobar o rechazar este plan y sus supuestos S-01…S-10.
2. Aprobar la arquitectura y las dependencias propuestas (D-07, D-08, D-09).
3. Aprobar el contenido de las especificaciones y sus escenarios (FASE 2).
4. Autorizar cualquier cambio de alcance, de arquitectura o de seguridad.
5. Definir políticas institucionales (horarios, anticipación, cancelación, quién aprueba).
6. Autorizar `git push`, despliegues y publicación de releases.
7. Decidir el orden y el momento de cada incremento.
8. Aprobar el cambio formal de la regla institucional de FASE 4.

## 6.6 Política para impedir implementación sin especificación aprobada

1. Ninguna tarea se implementa si no está en un cambio OpenSpec con especificación y tareas revisadas.
2. Si aparece una necesidad nueva durante la implementación, el agente se detiene y propone un cambio formal; no la resuelve "de paso".
3. Toda desviación entre plan, especificación y código se registra en `docs/decisions/` con motivo, impacto y aprobación.
4. Las pruebas del incremento se escriben contra los escenarios aprobados, no contra el comportamiento accidental del código.
5. Un incremento sin escenario verificable asociado no se acepta como terminado.

---

# 7. Matriz inicial de trazabilidad

Correspondencia entre necesidad, módulo propuesto y evidencia futura. Los nombres de archivos y pruebas son **propuestas**, no existentes.

| Necesidad | Módulo / componente propuesto | Evidencia futura prevista |
|---|---|---|
| N-01 Gestión de espacios | `domain/spaces`, `adapters/http` (CRUD `/api/spaces`), página *Espacios* | Pruebas de dominio + pruebas HTTP de CRUD y autorización; escenario de espacio desactivado |
| N-02 Gestión de usuarios | `domain/identity`, `/api/users`, página *Administración* | Pruebas de alta/baja lógica y de acceso restringido a Administrador |
| N-03 Roles y permisos | `domain/identity`, middleware de autorización | Suite de permisos por endpoint (403) y prueba de rol falsificado |
| N-04 Solicitudes de reserva | `domain/booking`, `/api/bookings`, página *Nueva solicitud* | Pruebas de creación, validación de fechas y estado inicial `pendiente` |
| N-05 Consulta de disponibilidad | `domain/availability`, `/api/spaces/{id}/availability` | Pruebas de intervalos, bordes y franjas cerradas |
| N-06 Prevención de solapamientos | Restricción `EXCLUDE` + repositorio + caso de uso *aprobar* | Prueba concurrente y de violación de restricción; solapados = 0 |
| N-07 Aprobación y rechazo | `application` (aprobar/rechazar), `/api/bookings/{id}/approve` | Pruebas de transiciones, motivo obligatorio y 409 por conflicto |
| N-08 Cancelación | Caso de uso *cancelar*, `/api/bookings/{id}/cancel` | Pruebas de política de 24 h y de liberación de disponibilidad |
| N-09 Historial de reservas | Consultas de `bookings` por usuario/global, página *Historial* | Pruebas de filtrado por rol y de aislamiento entre usuarios |
| N-10 Auditoría de estados | `booking_events` | Prueba de que cada transición registra autor, fecha y motivo |

---

# 8. Supuestos pendientes de confirmación

Consolidado de lo que requiere decisión del estudiante antes de especificar (FASE 2) o implementar (FASE 3):

| ID | Supuesto | Valor propuesto | Bloquea | Riesgo si se asume mal |
|---|---|---|---|---|
| S-01 | Zona horaria | `America/Bogota` | FASE 2 | Alto |
| S-02 | Horario operativo | L-V 06–22, Sáb 07–18, Dom cerrado | FASE 2 | Medio |
| S-03 | Anticipación | Mín. 24 h, máx. 90 días | FASE 2 | Bajo |
| S-04 | Qué bloquea disponibilidad | Solo `aprobada`; pendientes no bloquean y se revalidan al aprobar | FASE 2 | Alto |
| S-05 | Autoaprobación | Prohibida | FASE 2 | Medio |
| S-06 | Cancelación | Pendiente: siempre; aprobada: hasta 24 h antes; Administrador sin límite | FASE 2 | Medio |
| S-07 | Motivo obligatorio | Obligatorio al rechazar | FASE 2 | Bajo |
| S-08 | Multi-rol | Permitido | FASE 2 | Medio |
| S-09 | Espacio desactivado | No admite nuevas solicitudes; conserva historial | FASE 2 | Bajo |
| S-10 | Duración mínima / granularidad | 30 min / 15 min | FASE 2 | Bajo |

---

# 9. Control de cambios del documento

| Versión | Fecha | Cambio | Estado |
|---|---|---|---|
| 1.0 | 2026-11-24 | Creación inicial del plan de ingeniería | Pendiente de aprobación |

Cualquier modificación posterior a la aprobación se registrará aquí con motivo, impacto y aprobación explícita, y no se aplicará silenciosamente.
