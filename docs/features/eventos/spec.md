# Feature Specification: Eventos – Creación, publicación y consulta

**Created**: 2026-11-11  
**Área**: `eventos`  
**Rol principal**: `ADMIN` (gestión de eventos) y visitante anónimo (consulta)  
**Fuentes**: PRD de Servicios (BR-EVENT-001 a BR-EVENT-011, FR-SVC-003 a FR-SVC-008), PRD de Controladores (FR-CTRL-EVT-001 a FR-CTRL-EVT-006), PRD de Seguridad (FR-SEC-EVT-01 a FR-SEC-EVT-06) y PRD de Frontend (estados visibles para el público)  
**Solicitudes de cambio**: SC-02 (no se asocian artistas inactivos a eventos)

---

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Crear un evento (Priority: P1)

Como administrador, quiero crear un evento en una sede activa, para empezar a prepararlo antes de ponerlo a la venta.

**Why this priority**: Sin eventos no hay entradas que vender. Es el punto de partida de todo el flujo del producto.

**Independent Test**: Un administrador crea un evento con datos válidos sobre una sede activa de la semilla y se comprueba que queda en estado borrador, sin artistas y con los datos de su sede.

**Acceptance Scenarios**:

1. **Scenario**: Creación exitosa
   - **Given** un administrador con sesión iniciada y una sede activa
   - **When** crea un evento con código, nombre, descripción, categoría, fecha futura, edad mínima y código de la sede
   - **Then** el sistema crea el evento en estado borrador, sin artistas, y devuelve sus datos junto con el nombre y el código de la sede

2. **Scenario**: El estado inicial no se puede elegir
   - **Given** un administrador que intenta indicar un estado distinto de borrador al crear el evento
   - **When** envía la solicitud
   - **Then** el evento se crea siempre en estado borrador

3. **Scenario**: Código de evento repetido
   - **Given** un evento existente con un código determinado
   - **When** el administrador crea otro evento con el mismo código
   - **Then** el sistema rechaza la creación como conflicto, identifica el código y no crea nada

4. **Scenario**: Sede inexistente
   - **Given** que el código de sede indicado no corresponde a ninguna sede
   - **When** el administrador crea el evento
   - **Then** el sistema informa que la sede no fue encontrada y no crea el evento

5. **Scenario**: Sede inactiva
   - **Given** una sede existente que no está activa
   - **When** el administrador crea un evento en ella
   - **Then** el sistema rechaza la creación por regla de negocio y no crea el evento

6. **Scenario**: Fecha que no es futura
   - **Given** una fecha igual o anterior al momento actual
   - **When** el administrador crea el evento con esa fecha
   - **Then** el sistema rechaza la creación por regla de negocio y no crea el evento

7. **Scenario**: Edad mínima cero
   - **Given** un evento con edad mínima 0
   - **When** el administrador lo crea
   - **Then** el sistema lo acepta y entiende que no hay restricción de edad

8. **Scenario**: Datos faltantes o inválidos
   - **Given** una solicitud con campos obligatorios vacíos o con una edad mínima negativa
   - **When** el administrador la envía
   - **Then** el sistema la rechaza como solicitud inválida e indica, campo por campo, qué debe corregirse

9. **Scenario**: Usuario sin permiso
   - **Given** una persona sin sesión o con sesión de un rol distinto de `ADMIN`
   - **When** intenta crear un evento
   - **Then** el sistema no lo permite y no crea nada, según la política del área `auth`

---

### User Story 2 - Consultar la cartelera (Priority: P1)

Como persona que busca planes, con o sin sesión iniciada, quiero ver la lista de eventos disponibles, para elegir a cuál asistir.

**Why this priority**: La cartelera es la puerta de entrada al producto y de ella sale la compra de entradas.

**Independent Test**: Con eventos en distintos estados cargados en la semilla, se consulta la cartelera y se comprueba que solo aparecen los publicados y los agotados, ordenados por fecha.

**Acceptance Scenarios**:

1. **Scenario**: Cartelera con eventos
   - **Given** eventos publicados y agotados
   - **When** se consulta la cartelera
   - **Then** el sistema devuelve un resumen de cada uno, con código, nombre, categoría, fecha, estado y datos de la sede, ordenados por fecha ascendente

2. **Scenario**: Otros estados no aparecen
   - **Given** eventos en borrador, cancelados o finalizados
   - **When** se consulta la cartelera
   - **Then** el sistema no incluye esos eventos

3. **Scenario**: Cartelera vacía
   - **Given** que no hay eventos publicados ni agotados
   - **When** se consulta la cartelera
   - **Then** el sistema responde con éxito y una lista vacía

4. **Scenario**: Consulta sin sesión iniciada
   - **Given** un visitante que no ha iniciado sesión
   - **When** consulta la cartelera
   - **Then** el sistema responde con la lista sin pedir credenciales

---

### User Story 3 - Consultar un evento por código (Priority: P1)

Como persona interesada en un evento, quiero ver su detalle completo, para decidir si compro una entrada.

**Why this priority**: El detalle es lo que la persona necesita para decidir, y es donde se inicia la compra.

**Independent Test**: Se consulta por código un evento publicado y uno en borrador, sin sesión y con sesión de `ADMIN`, y se comprueba que el borrador solo lo ve el personal autorizado.

**Acceptance Scenarios**:

1. **Scenario**: Evento publicado
   - **Given** un evento publicado con artistas asociados
   - **When** cualquier persona lo consulta por su código
   - **Then** el sistema devuelve su identificador, código, nombre, descripción, categoría, estado, fecha, edad mínima, sede (código y nombre) y sus artistas

2. **Scenario**: Evento agotado, o cancelado o finalizado que estuvo publicado
   - **Given** un evento agotado, o un evento cancelado o finalizado que estuvo publicado alguna vez
   - **When** cualquier persona lo consulta por su código
   - **Then** el sistema devuelve su detalle indicando el estado, para que la persona entienda por qué no se puede comprar

3. **Scenario**: Borrador, o cancelado sin haberse publicado, consultado por el público
   - **Given** un evento en borrador, o un evento cancelado que nunca fue publicado
   - **When** lo consulta una persona sin sesión o un `USER`
   - **Then** el sistema responde como si el evento no existiera (no encontrado) y no revela que existe ni cuál es su estado

4. **Scenario**: Borrador, o cancelado sin haberse publicado, consultado por personal autorizado
   - **Given** un evento en borrador, o un evento cancelado que nunca fue publicado
   - **When** lo consulta un `STAFF` o un `ADMIN`
   - **Then** el sistema devuelve su detalle completo

5. **Scenario**: Código inexistente
   - **Given** que no existe un evento con el código consultado
   - **When** se consulta por ese código
   - **Then** el sistema informa que el evento no fue encontrado, identifica el código y usa el contrato de errores común

---

### User Story 4 - Publicar un evento (Priority: P1)

Como administrador, quiero publicar un evento que ya preparé, para que el público lo vea y pueda comprar entradas.

**Why this priority**: Publicar es el paso que abre la venta. Sin él, los eventos nunca llegan al público.

**Independent Test**: Se publica un evento en borrador de la semilla y se comprueba que aparece en la cartelera.

**Acceptance Scenarios**:

1. **Scenario**: Publicación exitosa
   - **Given** un evento en borrador, con fecha futura y en una sede activa
   - **When** un administrador lo publica
   - **Then** el sistema cambia su estado a publicado y devuelve sus datos

2. **Scenario**: Evento que no está en borrador
   - **Given** un evento publicado, agotado, cancelado o finalizado
   - **When** un administrador intenta publicarlo
   - **Then** el sistema rechaza la operación como conflicto y no modifica el evento

3. **Scenario**: La fecha dejó de ser futura
   - **Given** un evento en borrador cuya fecha ya pasó
   - **When** un administrador intenta publicarlo
   - **Then** el sistema rechaza la operación por regla de negocio y no modifica el evento

4. **Scenario**: La sede dejó de estar activa
   - **Given** un evento en borrador cuya sede ya no está activa
   - **When** un administrador intenta publicarlo
   - **Then** el sistema rechaza la operación por regla de negocio y no modifica el evento

5. **Scenario**: Evento inexistente
   - **Given** que no existe un evento con el código indicado
   - **When** un administrador intenta publicarlo
   - **Then** el sistema informa que el evento no fue encontrado

6. **Scenario**: Usuario sin permiso
   - **Given** una persona sin sesión o con sesión de un rol distinto de `ADMIN`
   - **When** intenta publicar un evento
   - **Then** el sistema no lo permite y no modifica el evento, según la política del área `auth`

---

### User Story 5 - Asociar un artista a un evento (Priority: P2)

Como administrador, quiero indicar qué artistas participan en un evento, para que el público sepa quién se presenta.

**Why this priority**: Enriquece la cartelera y permite buscar eventos por artista, pero el evento existe y se vende sin ello.

**Independent Test**: Se asocian a un evento de la semilla dos artistas distintos y se comprueba que el detalle del evento los muestra.

**Acceptance Scenarios**:

1. **Scenario**: Asociación exitosa
   - **Given** un evento que no está cancelado ni finalizado y un artista existente y activo
   - **When** un administrador asocia el artista al evento
   - **Then** el sistema devuelve el evento actualizado con el artista incluido

2. **Scenario**: Varios artistas en un mismo evento
   - **Given** un evento al que ya se asoció un artista
   - **When** un administrador asocia otro artista distinto
   - **Then** el evento queda con ambos artistas

3. **Scenario**: Artista ya asociado
   - **Given** un evento al que ya está asociado un artista
   - **When** un administrador intenta asociar de nuevo al mismo artista
   - **Then** el sistema rechaza la operación como conflicto y no duplica la asociación

4. **Scenario**: Evento cancelado o finalizado
   - **Given** un evento cancelado o finalizado
   - **When** un administrador intenta asociarle un artista
   - **Then** el sistema rechaza la operación como conflicto y no modifica el evento

5. **Scenario**: Artista inactivo
   - **Given** un evento que admite artistas y un artista que no está activo
   - **When** un administrador intenta asociar el artista al evento
   - **Then** el sistema rechaza la operación como conflicto por regla de negocio y no modifica el evento

6. **Scenario**: Evento o artista inexistente
   - **Given** un código de evento o un identificador de artista que no existe
   - **When** un administrador intenta asociarlos
   - **Then** el sistema informa cuál de los dos no fue encontrado y no modifica nada

7. **Scenario**: Usuario sin permiso
   - **Given** una persona sin sesión o con sesión de un rol distinto de `ADMIN`
   - **When** intenta asociar un artista
   - **Then** el sistema no lo permite y no modifica el evento, según la política del área `auth`

---

### User Story 6 - Consultar eventos por artista (Priority: P2)

Como persona que sigue a un artista, quiero ver en qué eventos se presenta, para no perderme sus presentaciones.

**Why this priority**: Es una forma alternativa de descubrir eventos. La cartelera ya cubre el descubrimiento principal.

**Independent Test**: Se consultan los eventos de un artista de la semilla con y sin sesión de `ADMIN`, y se comprueba que solo el personal autorizado ve los borradores.

**Acceptance Scenarios**:

1. **Scenario**: Artista con eventos
   - **Given** un artista asociado a eventos publicados y a un evento en borrador
   - **When** una persona sin sesión o un `USER` consulta los eventos de ese artista por su nombre artístico
   - **Then** el sistema devuelve el resumen de los eventos visibles para el público, ordenados por fecha ascendente

2. **Scenario**: Personal autorizado
   - **Given** el mismo artista
   - **When** un `STAFF` o un `ADMIN` consulta sus eventos
   - **Then** el sistema devuelve todos, incluidos los borradores

3. **Scenario**: Artista sin eventos visibles
   - **Given** un artista existente sin eventos visibles para quien consulta
   - **When** se consultan sus eventos
   - **Then** el sistema responde con éxito y una lista vacía

4. **Scenario**: Artista inexistente
   - **Given** que no existe un artista con el nombre indicado
   - **When** se consultan sus eventos
   - **Then** el sistema informa que el artista no fue encontrado, identifica el nombre y usa el contrato de errores común

5. **Scenario**: Nombre con otra capitalización
   - **Given** un artista llamado `Solar Beat`
   - **When** se consultan los eventos con el nombre `solar beat`
   - **Then** el sistema devuelve los eventos de ese mismo artista

6. **Scenario**: Consulta sin sesión iniciada
   - **Given** un visitante que no ha iniciado sesión
   - **When** consulta los eventos de un artista
   - **Then** el sistema responde sin pedir credenciales, con los eventos visibles para el público

---

### Edge Cases

- **Estados sin operación**: en esta versión no existe una operación para cancelar ni para finalizar un evento. Esos estados existen y sus reglas se aplican, pero solo se cargan con un script SQL de pruebas E2E, que incluye un evento cancelado que estuvo publicado y otro cancelado sin haberse publicado nunca.
- **Estado agotado**: lo asigna el área `tickets` cuando una compra completa la capacidad de la sede. Esta área solo lo consulta.
- **Fechas y zona horaria**: las fechas de los eventos se interpretan en la zona `America/Bogota`. Una fecha es futura si es posterior al instante actual en esa zona, medido con un reloj inyectado y configurable para poder fijarlo en las pruebas.
- **Visibilidad**: el público ve los eventos publicados y agotados, y los cancelados o finalizados que estuvieron publicados alguna vez. Un borrador, o un evento cancelado sin haberse publicado nunca, solo lo ven `STAFF` y `ADMIN`. Para distinguirlos, el sistema registra internamente cuándo se publicó cada evento.
- **Código de evento**: la coincidencia es exacta; `CMF-2026` y `cmf-2026` son códigos distintos.
- **Artista inactivo**: no se puede asociar a un evento (solicitud de cambio SC-02). Si un artista ya asociado se desactiva después, el evento conserva la asociación.
- **Cartelera sin paginación**: devuelve todos los eventos visibles, igual que el cliente espera.
- **Edad mínima**: 0 significa sin restricción; el valor negativo no es válido.
- **Una sede inactiva con eventos existentes**: los eventos ya creados siguen consultables. La sede inactiva impide crear y publicar eventos nuevos, y bloquea la compra de entradas de sus eventos (regla del área `tickets`, solicitud de cambio SC-02). Las entradas ya vendidas siguen siendo válidas.

---

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: El sistema MUST permitir a un `ADMIN` crear un evento con código, nombre, descripción, categoría, fecha, edad mínima y código de sede.
- **FR-002**: Los campos código, nombre, categoría, fecha, edad mínima y código de sede MUST ser obligatorios; la descripción es opcional. Un campo obligatorio vacío o una edad mínima negativa MUST rechazarse como solicitud inválida, indicando el motivo por campo.
- **FR-003**: El código del evento MUST ser único y su coincidencia MUST ser exacta (BR-EVENT-001). Un código repetido MUST rechazarse como conflicto.
- **FR-004**: La sede indicada MUST existir; si no existe, el sistema MUST informar que no fue encontrada (BR-EVENT-002).
- **FR-005**: La sede MUST estar activa; una sede inactiva MUST rechazar la creación del evento por regla de negocio (BR-EVENT-003).
- **FR-006**: La fecha del evento MUST ser futura al momento de la creación (BR-EVENT-004).
- **FR-007**: Todo evento nuevo MUST iniciar en estado borrador, sin importar lo que envíe la solicitud (BR-EVENT-005).
- **FR-008**: La edad mínima MUST ser mayor o igual que 0; 0 significa sin restricción (BR-EVENT-006).
- **FR-009**: Las categorías válidas MUST ser música, deportes, tecnología, educación, cultura y entretenimiento.
- **FR-010**: El sistema MUST permitir a un `ADMIN` publicar un evento, cambiando su estado de borrador a publicado.
- **FR-011**: Solo un evento en borrador MUST poder publicarse (BR-EVENT-007).
- **FR-012**: Al publicar, la fecha del evento MUST seguir siendo futura (BR-EVENT-008) y su sede MUST seguir activa (BR-EVENT-009).
- **FR-013**: El sistema MUST permitir consultar la cartelera y MUST incluir únicamente los eventos publicados y agotados, ordenados por fecha ascendente.
- **FR-014**: Una cartelera sin eventos MUST responderse como éxito con una lista vacía.
- **FR-015**: El sistema MUST permitir consultar un evento por su código. Cualquier persona MUST poder ver un evento publicado o agotado, y un evento cancelado o finalizado que haya estado publicado alguna vez.
- **FR-016**: Un evento en borrador, o cancelado sin haber sido publicado nunca, MUST ser visible solo para `STAFF` y `ADMIN`. Para cualquier otra persona MUST responderse como no encontrado, sin revelar que existe.
- **FR-017**: El sistema MUST permitir a un `ADMIN` asociar un artista existente a un evento y MUST devolver el evento actualizado.
- **FR-018**: El sistema MUST NOT permitir asociar dos veces el mismo artista a un evento (BR-EVENT-010).
- **FR-019**: El sistema MUST NOT permitir asociar artistas a un evento cancelado o finalizado (BR-EVENT-011).
- **FR-020**: Un evento o un artista inexistente MUST informarse como no encontrado, indicando cuál de los dos falta.
- **FR-021**: El sistema MUST permitir consultar los eventos de un artista por su nombre artístico, ignorando mayúsculas y minúsculas, ordenados por fecha ascendente.
- **FR-022**: Quien no sea `STAFF` ni `ADMIN` MUST ver únicamente los eventos visibles para el público (FR-015 y FR-016); `STAFF` y `ADMIN` MUST ver todos.
- **FR-023**: Si el artista no existe, el sistema MUST informar que no fue encontrado usando el contrato de errores común. Si existe pero no tiene eventos visibles, MUST responder con éxito y una lista vacía.
- **FR-024**: Las operaciones de crear, publicar y asociar artistas MUST estar reservadas a `ADMIN`. La consulta de cartelera, de detalle de un evento visible para el público y de eventos por artista MUST ser pública. Las respuestas de "no autenticado" y "sin permiso" siguen la política del área `auth`.
- **FR-025**: La respuesta de un evento MUST incluir identificador, código, nombre, descripción, categoría, estado, fecha, edad mínima, código y nombre de la sede, y sus artistas. El resumen de un evento MUST incluir código, nombre, categoría, fecha, estado y los datos de su sede (nombre y ciudad).
- **FR-026**: El sistema MUST devolver datos mediante objetos de transferencia y nunca exponer entidades de persistencia.
- **FR-027**: Los textos del evento MUST respetar una longitud máxima. [NEEDS CLARIFICATION: las longitudes máximas de código, nombre y descripción dependen del PRD de Persistencia, que aún no está disponible]
- **FR-028**: El artista que se asocia a un evento MUST estar activo; asociar un artista inactivo MUST rechazarse por regla de negocio (solicitud de cambio SC-02).
- **FR-029**: El sistema MUST registrar internamente la fecha y hora en que un evento se publica. Ese dato decide la visibilidad pública de los eventos cancelados y finalizados y MUST NOT incluirse en las respuestas.

### Key Entities

- **Event (evento)**: actividad con fecha, que se realiza en una sede. Atributos de negocio: identificador, código único, nombre, descripción, categoría (música, deportes, tecnología, educación, cultura, entretenimiento), estado, fecha y hora, edad mínima, sede y fecha de publicación (dato interno que no se devuelve en las respuestas). Se relaciona con una sede, con muchos artistas y con muchos tickets.
- **Estado del evento**: borrador, publicado, agotado, cancelado o finalizado. Transiciones: borrador a publicado; publicado a agotado; agotado a finalizado; borrador o publicado a cancelado.

---

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: El 100 % de los eventos creados con datos válidos queda en estado borrador.
- **SC-002**: 0 % de los eventos se crea con un código repetido, una sede inexistente o inactiva, o una fecha no futura.
- **SC-003**: 0 % de los borradores, y de los eventos cancelados que nunca se publicaron, es visible para una persona sin sesión o con rol `USER`, y el 100 % de las consultas a esos eventos por esas personas se responde como no encontrado.
- **SC-004**: El 100 % de las publicaciones de un evento que no está en borrador se rechaza sin modificar el evento.
- **SC-005**: El 100 % de las operaciones de crear, publicar y asociar artistas ejecutadas sin rol `ADMIN` se rechaza sin modificar datos.
- **SC-006**: 0 % de las asociaciones de artista queda duplicada.
- **SC-007**: El 100 % de las consultas a un evento o artista inexistente se responde con el contrato de errores común e identifica el valor consultado.
- **SC-008**: 0 % de las asociaciones de un artista inactivo a un evento se completa.
