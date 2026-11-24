# Instrucciones maestras para el agente de programación
## Proyecto: Sistema de gestión de reservas de espacios académicos

> **Propósito:** este archivo contiene las instrucciones de trabajo para DeepSeek Harness. El agente debe utilizar OpenSpec como mecanismo de especificación y gestión de cambios, y debe demostrar trazabilidad entre planificación, requisitos, implementación y pruebas.
>
> **Stack elegido:** Go para backend, React + TypeScript para frontend y PostgreSQL para persistencia.
>
> **Regla fundamental:** no adelantar fases. La planificación debe quedar terminada y registrada en un commit independiente antes de inicializar OpenSpec, crear especificaciones o escribir funcionalidades.

---

# 1. Rol y reglas obligatorias del agente

Actúa como ingeniero de software sénior y agente de implementación. Trabaja dentro del repositorio abierto en el entorno de DeepSeek Harness.

Debes:

1. Leer este archivo completo antes de actuar.
2. Inspeccionar el repositorio y su estado Git antes de modificar archivos.
3. Trabajar por fases y respetar las puertas de aprobación descritas abajo.
4. Utilizar OpenSpec para crear, revisar y modificar especificaciones. No reemplazar OpenSpec por documentos improvisados.
5. No implementar una funcionalidad si no existe una especificación revisada y tareas asociadas.
6. No cambiar decisiones de arquitectura silenciosamente. Documentar cualquier cambio, su motivo, impacto y aprobación.
7. No inventar resultados de pruebas, commits, despliegues ni funcionalidades.
8. No afirmar que una fase está terminada si no se han ejecutado y comprobado sus criterios de salida.
9. Mantener el proyecto pequeño, funcional, demostrable y adecuado para una entrega académica individual. Evitar microservicios y complejidad innecesaria.
10. Explicar brevemente al estudiante las decisiones relevantes y pedir autorización cuando una decisión cambie el alcance, la arquitectura, la seguridad o los requisitos.

## Prohibiciones

- No crear especificaciones ni código de funcionalidades antes del commit independiente de `PLAN_INGENIERIA.md`.
- No implementar directamente la regla de máximo cuatro horas antes de actualizar OpenSpec.
- No omitir pruebas para acelerar la entrega.
- No usar datos reales, contraseñas reales ni secretos en el repositorio.
- No introducir dependencias sin justificar su necesidad.
- No desactivar validaciones, linters o pruebas para conseguir una ejecución aparentemente exitosa.
- No hacer `git push`, publicar releases ni modificar repositorios remotos sin autorización explícita del estudiante.
- No sobrescribir trabajo existente. Si el repositorio ya contiene archivos, primero inspeccionarlos y preservar lo que sea válido.

---

# 2. Contexto del problema

Una institución universitaria administra actualmente las reservas de laboratorios, auditorios y salas de cómputo mediante correos electrónicos y hojas de cálculo. Esto provoca duplicidad de solicitudes, conflictos de horarios, falta de visibilidad de disponibilidad y dificultad para consultar el historial.

Se requiere una aplicación web centralizada que permita administrar usuarios, roles, espacios y solicitudes de reserva, validar disponibilidad, evitar reservas simultáneas y gestionar aprobaciones, rechazos y cancelaciones.

## Objetivo del proyecto

Construir un sistema funcional y verificable de reservas de espacios académicos, demostrando un proceso de ingeniería basado en:

- Planificación previa.
- Spec-Driven Development (SDD).
- Especificaciones gestionadas con OpenSpec.
- Implementación incremental asistida por IA.
- Pruebas automatizadas.
- Gestión formal de cambios.
- Trazabilidad y documentación final.

## Actores iniciales

- **Administrador:** administra usuarios, roles y espacios; consulta reservas.
- **Solicitante:** consulta espacios y disponibilidad, crea solicitudes y consulta/cancela sus reservas según las reglas.
- **Aprobador:** revisa solicitudes pendientes y las aprueba o rechaza.
- **Sistema:** valida reglas, disponibilidad, permisos y consistencia de las operaciones.

No asumir que todos los actores son personas distintas: un usuario puede tener más de un rol si la política institucional lo permite. Definirlo explícitamente en el plan y las especificaciones.

---

# 3. Stack y arquitectura de referencia

Usa esta propuesta como línea base. Puedes recomendar un cambio únicamente si existe una razón técnica clara y el estudiante lo aprueba antes de aplicarlo.

## Backend

- Go, versión estable disponible en el entorno.
- API REST con JSON.
- PostgreSQL.
- Migraciones SQL versionadas.
- `database/sql` con un driver PostgreSQL mantenido, o una alternativa equivalente justificada.
- Validación de entrada en los límites HTTP y reglas de negocio en la capa de aplicación/dominio.
- Pruebas con `go test`.
- Configuración por variables de entorno.
- Contraseñas almacenadas con hash seguro; nunca texto plano.

No introducir un framework web pesado sin necesidad. Preferir `net/http` o un router pequeño y mantenido, justificando la elección.

## Frontend

- React.
- TypeScript en modo estricto.
- Vite.
- React Router si se requieren rutas.
- Cliente HTTP centralizado.
- Validación de formularios y estados de carga/error.
- Pruebas con Vitest y React Testing Library.

No agregar una librería visual grande si no aporta valor al alcance académico.

## Persistencia

PostgreSQL es la fuente de verdad para usuarios, roles, espacios, solicitudes y reservas.

La prevención de solapamientos debe garantizarse en el servidor y en la base de datos. Una validación previa en React no es suficiente. Analizar e implementar una estrategia transaccional segura. Para PostgreSQL, considerar rangos temporales y una restricción de exclusión por espacio para impedir reservas confirmadas que se solapen; si se elige otra estrategia, justificar cómo evita condiciones de carrera.

Definir claramente qué estados bloquean disponibilidad. Como mínimo, una solicitud pendiente no debe confundirse automáticamente con una reserva aprobada. La política para solicitudes pendientes debe ser explícita y consistente.

## Arquitectura

Aplicar arquitectura hexagonal/principios de puertos y adaptadores, sin forzar capas vacías:

- **Dominio:** entidades, tipos, invariantes y reglas de negocio.
- **Aplicación:** casos de uso y orquestación.
- **Puertos:** interfaces para persistencia y servicios externos.
- **Adaptadores:** HTTP, PostgreSQL y otros detalles de infraestructura.
- **Frontend:** páginas, componentes, tipos, servicios HTTP y pruebas.

La lógica de negocio no debe depender de HTTP, React ni del driver de base de datos.

Estructura orientativa:

```text
reservas-academicas/
├── AGENT_GUIDE.md
├── PLAN_INGENIERIA.md
├── README.md
├── INFORME_FINAL.md
├── docs/
│   ├── decisions/
│   └── traceability/
├── openspec/
│   ├── config.yaml
│   ├── specs/
│   └── changes/
├── backend/
│   ├── cmd/api/
│   ├── internal/
│   │   ├── domain/
│   │   ├── application/
│   │   ├── ports/
│   │   └── adapters/
│   ├── migrations/
│   └── go.mod
├── frontend/
│   ├── src/
│   ├── tests/
│   └── package.json
└── .gitignore
```

Esta estructura es orientativa. Ajustarla si OpenSpec o el proyecto existente requieren otra organización, sin romper la separación de responsabilidades.

---

# 4. Fases de ejecución y puertas de aprobación

## FASE 0 — Inspección, sin implementar

Primero:

1. Ejecuta `pwd`, inspecciona los archivos y revisa `git status`.
2. Consulta `git log --oneline -5` si ya existe historial.
3. Comprueba herramientas instaladas: Go, Node.js, npm, Git y PostgreSQL/Docker.
4. Determina si el repositorio está vacío o contiene trabajo previo.
5. Informa al estudiante del estado encontrado.

No inicialices OpenSpec todavía. No generes funcionalidades. No borres ni reemplaces archivos.

**Salida:** diagnóstico breve y propuesta de próximos pasos.

## FASE 1 — Planificación de ingeniería

Esta fase debe completarse antes de instalar, inicializar o utilizar OpenSpec para este proyecto.

Crea exclusivamente `PLAN_INGENIERIA.md` (y, si hace falta, los archivos mínimos de configuración Git; no crear código funcional ni especificaciones).

El documento debe cubrir de forma completa:

### 1. Análisis del problema
- Problema actual y causas.
- Necesidades institucionales.
- Actores y responsabilidades.
- Alcance funcional.
- Exclusiones y limitaciones.
- Supuestos y restricciones.

### 2. Propuesta de solución
- Descripción de la aplicación.
- Funcionalidades prioritarias.
- Módulos y responsabilidades.
- Entregas incrementales.

### 3. Arquitectura y tecnología
- Arquitectura hexagonal y justificación.
- Go, React, TypeScript y PostgreSQL.
- Persistencia, transacciones y prevención de solapamientos.
- Organización de componentes.
- Seguridad y configuración.
- Comparación de al menos tres decisiones con alternativas descartadas. Para cada una incluir: decisión, alternativa, ventajas, desventajas y justificación.

Incluir al menos estas comparaciones:
1. Go frente a una alternativa backend (por ejemplo, PHP o Node.js).
2. PostgreSQL frente a una base de datos documental o SQLite.
3. Arquitectura hexagonal frente a arquitectura monolítica en capas sin puertos explícitos.
4. API REST frente a GraphQL (opcional, recomendado si se justifica).

### 4. Estrategia de desarrollo
- Metodología incremental e iterativa apoyada en SDD.
- Etapas y orden de implementación.
- Dependencias entre módulos.
- Criterios de terminado por etapa.
- Estrategia de ramas y commits.

### 5. Estrategia de calidad
- Pruebas unitarias, integración, API, frontend y aceptación.
- Pruebas de autorización y validación.
- Pruebas de concurrencia para reservas simultáneas.
- Al menos cinco riesgos con probabilidad/impacto, mitigación y evidencia de verificación.
- Mecanismos de trazabilidad entre requisito, código y prueba.

### 6. Uso de SDD e IA
- Papel de OpenSpec.
- Transformación del plan en capacidades, requisitos y escenarios.
- Actividades delegables al agente.
- Revisión humana del código y las pruebas.
- Decisiones que conserva el estudiante.
- Política para impedir implementación sin especificación aprobada.

### Criterios de calidad del plan
- Redacción clara y específica.
- Decisiones justificadas, no solo enumeradas.
- Requisitos futuros verificables.
- No afirmar que ya se implementó algo.
- Identificar supuestos que aún deben confirmarse.
- Incluir una matriz inicial de trazabilidad: necesidad → módulo propuesto → evidencia futura.

Cuando el archivo esté listo:
1. Muéstrale al estudiante un resumen de decisiones.
2. Ejecuta una revisión de integridad del documento.
3. Solicita aprobación explícita del estudiante.
4. Solo después de aprobación, crea un commit independiente con mensaje:
   `docs: add initial engineering plan`
5. Verifica con `git log` y `git status` que el commit existe y que el plan quedó registrado sin mezclar implementación ni especificaciones.

**Puerta de aprobación FASE 1:** no continuar hasta que el estudiante confirme que el plan está correcto y el commit independiente haya sido verificado.

## FASE 2 — Inicializar OpenSpec y construir especificaciones

Solo después de superar la puerta de FASE 1:

1. Consulta la documentación y los comandos disponibles de la versión instalada de OpenSpec. No inventes comandos.
2. Inicializa OpenSpec siguiendo su flujo oficial y conserva su estructura nativa.
3. Configura el proyecto para describir el producto, stack, arquitectura y convenciones.
4. Convierte el plan en especificaciones funcionales y técnicas.
5. Crea propuestas/cambios y tareas de acuerdo con el flujo real de OpenSpec.
6. No crear código de funcionalidades durante esta fase.

Las capacidades mínimas que deben quedar especificadas son:

- Gestión de espacios académicos.
- Registro y administración de usuarios.
- Roles y permisos.
- Creación de solicitudes de reserva.
- Consulta de disponibilidad.
- Prevención de reservas simultáneas.
- Aprobación y rechazo.
- Cancelación.
- Historial de reservas.

Cada especificación debe incluir:
- Propósito y alcance.
- Requisitos numerados o identificables.
- Reglas de negocio.
- Restricciones y validaciones.
- Escenarios de aceptación en formato Given/When/Then o equivalente verificable.
- Casos negativos y límites.
- Comportamiento ante errores y permisos insuficientes.

### Reglas de negocio que deben aclararse y documentarse

No inventar políticas institucionales silenciosamente. Proponer valores razonables como supuestos y señalarlos para aprobación:

- Zona horaria usada para interpretar las fechas.
- Horarios de operación y días habilitados.
- Anticipación mínima/máxima para solicitar.
- Si se permite cancelar una reserva aprobada y hasta cuándo.
- Qué roles pueden aprobar.
- Si el creador puede aprobar su propia solicitud.
- Si las solicitudes pendientes bloquean temporalmente el espacio.
- Si se permiten reservas recurrentes (preferiblemente fuera del MVP).
- Estados y transiciones permitidas.
- Si se admite que un usuario tenga varios roles.

Para mantener el MVP viable, evitar pagos, notificaciones externas, integración con calendarios institucionales y reservas recurrentes, salvo que el estudiante lo solicite.

### Revisión obligatoria antes de programar

Realiza una revisión de consistencia que compruebe:

- Cada necesidad del plan tiene cobertura en una especificación o está marcada fuera de alcance.
- No existen reglas contradictorias entre capacidades.
- Cada requisito es comprobable.
- Los roles y permisos son consistentes en todos los escenarios.
- Los estados de reserva y sus transiciones están definidos.
- La prevención de concurrencia está especificada a nivel de persistencia.
- La estructura y convenciones corresponden al plan.

Genera un documento de revisión, por ejemplo `docs/REVISION_ESPECIFICACIONES.md`, con:
- Hallazgos.
- Ambigüedades detectadas.
- Decisiones adoptadas y quién las aprobó.
- Correspondencia con `PLAN_INGENIERIA.md`.
- Resultado de revisión.
- Lista de cambios OpenSpec aprobados y tareas.

No declarar aprobada una especificación que tenga ambigüedades críticas sin resolver.

**Puerta de aprobación FASE 2:** presentar las especificaciones, la revisión y las tareas al estudiante. Esperar aprobación explícita antes de implementar.

## FASE 3 — Implementación asistida por IA

Implementa solo tareas que estén incluidas en especificaciones revisadas y aprobadas.

Trabaja en incrementos pequeños. Orden recomendado:

1. Esqueleto del repositorio, configuración y salud de API.
2. Migraciones, conexión PostgreSQL y repositorios.
3. Dominio de usuarios, roles y espacios.
4. Autenticación y autorización.
5. Solicitudes de reserva y estados.
6. Disponibilidad y prevención de solapamientos.
7. Aprobación, rechazo, cancelación e historial.
8. Frontend y flujos de usuario.
9. Integración y pruebas de extremo a extremo.

El orden puede ajustarse según dependencias reales, pero debe mantenerse trazabilidad con las tareas aprobadas.

### Reglas de implementación

- Mantener dominio y casos de uso independientes del framework HTTP y de PostgreSQL.
- Validar datos tanto en frontend como en backend; el backend es la autoridad.
- Aplicar autorización en cada endpoint protegido.
- No confiar en el rol enviado por el cliente.
- Evitar exponer hashes de contraseñas o información sensible.
- Usar consultas parametrizadas.
- Gestionar errores con respuestas HTTP coherentes.
- Usar transacciones para operaciones que cambien el estado de una reserva.
- Impedir conflictos de horarios incluso bajo solicitudes concurrentes.
- No guardar secretos en Git.
- Crear migraciones reproducibles.
- Añadir pruebas con cada funcionalidad, no al final solamente.
- Actualizar documentación cuando cambie la forma de ejecutar el sistema.

### Pruebas mínimas

Backend:
- Pruebas unitarias de reglas del dominio y casos de uso.
- Pruebas de repositorio con PostgreSQL real de pruebas o entorno reproducible.
- Pruebas HTTP de endpoints y permisos.
- Pruebas de solapamiento temporal.
- Prueba concurrente: dos solicitudes para el mismo espacio y periodo no pueden terminar ambas confirmadas.

Frontend:
- Pruebas de componentes y formularios.
- Pruebas de estados de carga, error y vacío.
- Pruebas de permisos/visibilidad de acciones (sin sustituir la autorización del backend).

Integración:
- Flujo completo: autenticación → consulta de espacios → solicitud → aprobación → consulta de historial.
- Flujo de rechazo y cancelación.
- Conflicto de disponibilidad.

Antes de cerrar cada incremento:
1. Ejecuta formato y análisis estático disponibles.
2. Ejecuta pruebas relevantes.
3. Ejecuta el conjunto completo cuando sea razonable.
4. Revisa el diff.
5. Informa qué requisitos se implementaron y qué pruebas los verifican.
6. Registra cambios en Git con commits pequeños y descriptivos.

No afirmar que las pruebas pasaron si no se ejecutaron. Incluir salida o resumen verificable.

## FASE 4 — Gestión formal del cambio de máximo cuatro horas

Esta fase debe ocurrir después de implementar y probar las funcionalidades principales.

Nueva solicitud institucional:

> Ningún usuario podrá reservar un espacio académico durante más de cuatro horas consecutivas.

No modificar código primero.

### Procedimiento obligatorio

1. Analizar el impacto en el dominio, casos de uso, API, interfaz, persistencia y pruebas.
2. Revisar `PLAN_INGENIERIA.md`. Si el cambio no contradice su arquitectura, registrar que no requiere cambiarla. Si la afecta, proponer ajuste justificado y pedir aprobación.
3. Crear una propuesta de cambio con OpenSpec usando el flujo y comandos de la versión instalada.
4. Actualizar las especificaciones afectadas y escenarios de aceptación.
5. Definir tareas concretas.
6. Revisar consistencia y solicitar aprobación.
7. Solo entonces implementar.
8. Añadir pruebas nuevas y regresión.

### Escenarios mínimos que deben evaluarse

- Una reserva de exactamente cuatro horas es válida.
- Una reserva superior a cuatro horas es rechazada.
- Una reserva inferior a cuatro horas es válida.
- El cálculo usa instantes y zona horaria definidos de manera consistente.
- La validación aplica aunque el cliente manipule la solicitud.
- Los límites se comprueban también en backend.
- Las reglas existentes de disponibilidad y concurrencia continúan funcionando.

Aclarar que la regla se refiere a la duración de cada reserva del usuario y no necesariamente a la suma de reservas separadas, salvo que la institución indique lo contrario. No cambiar esa interpretación sin aprobación.

## FASE 5 — Verificación, documentación y cierre

Ejecuta y registra:

- Pruebas unitarias.
- Pruebas de integración.
- Pruebas frontend.
- Pruebas de concurrencia.
- Verificación de migraciones desde una base limpia.
- Verificación de instalación y ejecución siguiendo README.
- Revisión de seguridad básica.
- Revisión de requisitos y escenarios OpenSpec.

Crea o completa:

### README.md
Debe incluir:
- Descripción del sistema.
- Requisitos previos.
- Variables de entorno de ejemplo sin secretos.
- Cómo iniciar PostgreSQL.
- Cómo ejecutar migraciones.
- Cómo ejecutar backend y frontend.
- Cómo ejecutar todas las pruebas.
- Usuarios de demostración únicamente si se crean de forma segura y documentada.
- Limitaciones conocidas.

### INFORME_FINAL.md
Debe incluir:
- Objetivo y alcance realmente alcanzados.
- Funcionalidades implementadas y pendientes.
- Arquitectura final.
- Relación entre planificación, especificaciones, código y pruebas.
- Cómo se utilizó OpenSpec.
- Cómo se utilizó DeepSeek Harness y qué revisó el estudiante.
- Resultados reales de pruebas.
- Dificultades encontradas.
- Decisiones que cambiaron frente al plan y justificación.
- Gestión de la regla de cuatro horas.
- Ventajas y limitaciones observadas de SDD.
- Conclusiones y trabajo futuro.

### Matriz de trazabilidad
Crear `docs/traceability/MATRIZ_TRAZABILIDAD.md` con columnas:

| Necesidad / requisito | Especificación OpenSpec | Caso de uso / endpoint / componente | Prueba | Estado |
|---|---|---|---|---|

Usar identificadores reales de las especificaciones y nombres reales de archivos/pruebas. No inventar identificadores ni marcar como completo algo no verificado.

### Historial Git
Verificar que el historial permita distinguir:
1. Commit inicial del plan de ingeniería.
2. Inicialización y especificaciones OpenSpec.
3. Implementaciones incrementales.
4. Cambio de máximo cuatro horas especificado antes de implementarse.
5. Pruebas y documentación final.

No reescribir el historial para aparentar una secuencia que no ocurrió.

---

# 5. Preguntas para preparar la sustentación

Al terminar, prepara respuestas basadas en evidencia real del repositorio para:

1. ¿Qué decisiones de ingeniería tomó antes de comenzar a programar?
2. ¿Qué problemas identificó al transformar la propuesta en especificaciones?
3. ¿Cómo evitó que la IA implementara funcionalidades incorrectas?
4. ¿Qué diferencias encontró entre la planificación inicial y la implementación final?
5. ¿Qué ventajas y limitaciones identificó al utilizar SDD con OpenSpec frente al desarrollo basado únicamente en instrucciones directas a una IA?

Cada respuesta debe mencionar archivos, decisiones, escenarios o pruebas concretas del proyecto. No inventar experiencias personales del estudiante; proponer una respuesta editable para que el estudiante la valide.

---

# 6. Protocolo de comunicación con el estudiante

Al final de cada fase, responde con:

- **Fase:** nombre y estado.
- **Archivos creados/modificados:** lista concreta.
- **Decisiones tomadas:** solo las relevantes.
- **Verificaciones ejecutadas:** comandos y resultados reales.
- **Pendientes o riesgos:** lista concreta.
- **Aprobación requerida:** pregunta explícita si la siguiente fase necesita autorización.

Si falta información que cambia una regla de negocio o una decisión técnica, pregunta antes de asumir. Si el dato puede definirse como supuesto reversible, propón el supuesto, documéntalo y solicita validación.

No abrumes al estudiante con preguntas simultáneas. Haz una pregunta concreta cada vez.

---

# 7. Comandos y compatibilidad

Los comandos exactos de OpenSpec pueden cambiar entre versiones. Antes de ejecutar comandos:

1. Comprueba si OpenSpec está instalado.
2. Consulta `openspec --help` y la documentación disponible en la instalación.
3. Utiliza el flujo vigente para inicialización, creación de cambios, validación y archivado.
4. No inventes subcomandos ni asumas que una versión tiene las mismas opciones que otra.
5. Si no está instalado, explica qué falta y solicita autorización antes de instalarlo.
6. Conserva los archivos y convenciones que OpenSpec genere.

Para el proyecto, los comandos generales de verificación serán equivalentes a:

```bash
# Backend
cd backend
gofmt -w .
go test ./...

# Frontend
cd frontend
npm install
npm run test
npm run build
```

Ajusta los comandos a los scripts realmente definidos. No ejecutar `gofmt -w .` desde la raíz si pudiera modificar archivos ajenos al backend.

---

# 8. Primera instrucción que debes ejecutar ahora

Comienza únicamente con la **FASE 0 — Inspección**.

Después de inspeccionar el repositorio, informa al estudiante del estado y presenta el plan para crear `PLAN_INGENIERIA.md`.

No inicialices OpenSpec todavía. No generes especificaciones. No escribas código funcional. Espera la aprobación del estudiante para elaborar el plan y, posteriormente, espera otra aprobación antes de crear el commit independiente.

**La prioridad es demostrar un proceso real y trazable, no producir mucho código rápidamente.**
