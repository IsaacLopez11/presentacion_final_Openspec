# Revisión de especificaciones

| Campo | Valor |
|---|---|
| Documento | `docs/REVISION_ESPECIFICACIONES.md` |
| Fecha | 2026-11-24 |
| Fase | FASE 2 — Especificaciones y revisión de consistencia |
| Cambio revisado | `add-reservation-system-mvp` (OpenSpec, esquema `spec-driven`) |
| Referencia de ingeniería | `PLAN_INGENIERIA.md` v1.0 (commit `e1231eb`) |
| Validador | `openspec validate add-reservation-system-mvp --strict` → **válido** |
| Resultado | **Aprobado con observaciones menores**, pendiente de aprobación explícita del estudiante |

## Alcance de la revisión

Se revisaron los cuatro artefactos del cambio (`proposal.md`, `design.md`, `tasks.md` y diez especificaciones de capacidad) contra: las necesidades N-01…N-10 del plan, los supuestos S-01…S-10, las reglas de negocio institucionales, y los seis criterios de revisión obligatoria que exige `AGENT_GUIDE.md` (FASE 2).

## Inventario verificado

| Capacidad | Requisitos | Escenarios |
|---|---|---|
| `gestion-espacios` | 4 | 12 |
| `gestion-usuarios` | 4 | 11 |
| `roles-permisos` | 5 | 12 |
| `solicitudes-reserva` | 6 | 19 |
| `consulta-disponibilidad` | 4 | 10 |
| `prevencion-solapamientos` | 4 | 10 |
| `aprobacion-rechazo` | 5 | 15 |
| `cancelacion-reservas` | 6 | 14 |
| `historial-reservas` | 6 | 14 |
| `auditoria-estados` | 5 | 12 |
| **Total** | **49** | **129** |

Todos los requisitos tienen al menos un escenario verificable; no hay escenarios con formato incorrecto (cero con tres almohadillas). Las tareas se distribuyen en **10 grupos con 62 tareas**, cada una con su criterio de verificación.

## Hallazgos

### H-01 — Autoaprobación no cubría explícitamente al administrador (corregido)

**Detectado en:** `aprobacion-rechazo`, requisito *Prohibición de autoaprobación*.
**Descripción:** el requisito prohibía la autoaprobación "incluso si tiene el rol de aprobador", pero `aprobacion-rechazo` también autoriza al administrador a decidir sobre solicitudes. Quedaba ambiguo si la prohibición alcanzaba al administrador.
**Corrección aplicada:** se añadió el escenario *Administrador autor de la solicitud*, que exige rechazar la operación por autoaprobación.
**Estado:** corregido en la especificación antes de la aprobación.

### H-02 — Festivos sin fuente de datos definida (resuelto por decisión)

**Detectado en:** `solicitudes-reserva`, requisito de horario operativo, que exige rechazar solicitudes en días festivos.
**Descripción:** ni el plan ni las especificaciones definían cómo se conoce el calendario de festivos.
**Decisión adoptada:** tabla `holidays` sembrada por migración con los festivos colombianos del año en curso, **sin interfaz ni API de administración en el MVP**; la actualización anual se hace por migración.
**Aprobado por:** el estudiante, en esta sesión (2026-11-24).
**Impacto:** tarea 2.4 y tarea 5.3; limitación documentada en el README.

### H-03 — Umbral de intentos de inicio de sesión sin cuantificar (diferido)

**Descripción:** el diseño prevé un límite de intentos (429) pero no fija el número. No altera ningún requisito de las especificaciones.
**Decisión:** se ajusta en implementación; se registra como pregunta abierta en `design.md`. No bloquea la aprobación.

### H-04 — Política S-04 verificada como consistente en las tres especificaciones afectadas

**Descripción:** se comprobó que `solicitudes-reserva`, `consulta-disponibilidad` y `prevencion-solapamientos` coinciden: solo `aprobada` bloquea; `pendiente` no bloquea y se revalida al aprobar. No se encontró contradicción.
**Estado:** conforme a la decisión S-04 aprobada por el estudiante (2026-11-24).

### H-05 — Estados y transiciones definidos de forma única

**Descripción:** se verificó que los cuatro estados (`pendiente`, `aprobada`, `rechazada`, `cancelada`) y sus transiciones son idénticos en `solicitudes-reserva`, `aprobacion-rechazo`, `cancelacion-reservas`, `historial-reservas` y `auditoria-estados`. `rechazada` y `cancelada` son terminales en todas ellas; ninguna especificación reintroduce un estado `finalizada` almacenado.

### H-06 — Concurrencia especificada a nivel de persistencia

**Descripción:** `prevencion-solapamientos` exige la garantía en la capa de persistencia y comportamiento determinista bajo concurrencia (exactamente una aprobación exitosa). El diseño la concreta con `EXCLUDE USING gist` y la traducción del código `23P01`. Se cumple el criterio de que la prevención no dependa de una validación previa. Tareas 2.6, 7.2 y 7.6 lo verifican.

### H-07 — Ambigüedad menor: qué ve un aprobador del historial

**Descripción:** el plan no precisaba si el aprobador accede al historial global.
**Decisión:** el aprobador solo consulta las solicitudes sobre las que emitió decisión; el historial global queda reservado al administrador. Coherente con el principio de mínimo privilegio y con `historial-reservas`.
**Estado:** resuelto en la especificación; requiere confirmación del estudiante junto con el resto del paquete.

### H-08 — Riesgo de entorno no resuelto (RG-03)

**Descripción:** la máquina no tiene Go, Node.js, npm ni PostgreSQL. No afecta a la validez de las especificaciones, pero impide verificar cualquier tarea de compilación o prueba.
**Estado:** abierto. No bloquea la aprobación de FASE 2, sí la ejecución de FASE 3.

## Correspondencia con PLAN_INGENIERIA.md

| Necesidad del plan | Especificación que la cubre | Estado |
|---|---|---|
| N-01 Catálogo de espacios | `gestion-espacios` | Cubierta |
| N-02 Gestión de usuarios | `gestion-usuarios` | Cubierta |
| N-03 Roles y permisos en servidor | `roles-permisos` | Cubierta |
| N-04 Creación de solicitudes | `solicitudes-reserva` | Cubierta |
| N-05 Consulta de disponibilidad | `consulta-disponibilidad` | Cubierta |
| N-06 No solapamiento | `prevencion-solapamientos` | Cubierta |
| N-07 Aprobación y rechazo | `aprobacion-rechazo` | Cubierta |
| N-08 Cancelación | `cancelacion-reservas` | Cubierta |
| N-09 Historial | `historial-reservas` | Cubierta |
| N-10 Trazabilidad de estados | `auditoria-estados` | Cubierta |

Las diez capacidades propuestas coinciden exactamente con las de la sección 6.2 del plan; no se introdujo ninguna capacidad nueva ni se omitió ninguna.

## Supuestos frente a las especificaciones

| Supuesto | Requisito que lo materializa | Estado de confirmación |
|---|---|---|
| S-01 Zona horaria `America/Bogota` en UTC | `solicitudes-reserva`, req. de zona horaria | Aprobado por inferencia institucional; confirmar |
| S-02 Horario operativo y festivos | `solicitudes-reserva`, req. de horario | **Pendiente de confirmación** |
| S-03 Anticipación 24 h / 90 días | `solicitudes-reserva`, req. de anticipación | **Pendiente de confirmación** |
| S-04 Solo `aprobada` bloquea | `consulta-disponibilidad`, `prevencion-solapamientos` | **Aprobado** (2026-11-24) |
| S-05 Sin autoaprobación | `aprobacion-rechazo` | **Pendiente de confirmación** |
| S-06 Plazos de cancelación | `cancelacion-reservas` | **Pendiente de confirmación** |
| S-07 Motivo obligatorio al rechazar | `aprobacion-rechazo` | **Pendiente de confirmación** |
| S-08 Multi-rol | `roles-permisos` | **Pendiente de confirmación** |
| S-09 Espacio inactivo | `gestion-espacios`, `solicitudes-reserva` | **Pendiente de confirmación** |
| S-10 Duración mínima y granularidad | `solicitudes-reserva`, `design.md` (CHECK) | **Pendiente de confirmación** |

Los supuestos pendientes están materializados como requisitos explícitos, no como constantes ocultas: si la institución cambia un valor, se modifica un requisito con su escenario, no un número disperso en el código.

## Lista de cambios y tareas aprobadas

- **Cambio OpenSpec:** `add-reservation-system-mvp`, esquema `spec-driven`, con `proposal.md`, `design.md`, `tasks.md` y diez deltas de especificación.
- **Tareas:** organizadas en diez grupos (esqueleto, base de datos, identidad y espacios, autenticación, solicitudes, disponibilidad, aprobación y solapamiento, cancelación/historial/auditoría, frontend e integración), cada una con su criterio de verificación y sus pruebas dentro del grupo.
- **Cumplimiento del límite de fases:** no se creó código de funcionalidad, ni migraciones, ni archivos de `backend/` o `frontend/`.

## Resultado de la revisión

- Criterio «cada necesidad tiene cobertura o está fuera de alcance»: **cumplido** (10/10 cubiertas).
- Criterio «sin reglas contradictorias»: **cumplido**, con la corrección H-01 aplicada.
- Criterio «cada requisito es comprobable»: **cumplido** (49/49 con escenarios; validación estricta sin avisos).
- Criterio «roles y permisos consistentes»: **cumplido** tras resolver H-07.
- Criterio «estados y transiciones definidos»: **cumplido** (H-05).
- Criterio «prevención de concurrencia en la persistencia»: **cumplido** (H-06).
- Criterio «estructura y convenciones del plan»: **cumplido**; `design.md` respeta la arquitectura hexagonal y el stack aprobado, sin añadir dependencias no justificadas.

**No se declara aprobada ninguna especificación con ambigüedad crítica sin resolver.** Las ambigüedades encontradas quedaron cerradas en H-01, H-02 y H-07; H-03 se difiere al no afectar requisitos; H-08 es un riesgo de entorno, no una ambigüedad de especificación.
