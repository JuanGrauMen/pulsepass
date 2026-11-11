# Feature Specification: Venues – Consulta de sedes

**Created**: 2026-11-11  
**Área**: `venues`  
**Rol principal**: Visitante anónimo (también aplica a `USER`, `STAFF` y `ADMIN`)  
**Fuentes**: PRD de Servicios (BR-VENUE-001, BR-VENUE-002, FR-SVC-001, FR-SVC-002), PRD de Controladores (FR-CTRL-VEN-001, FR-CTRL-VEN-002) y PRD de Seguridad (FR-SEC-VEN-01, FR-SEC-VEN-02)

---

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Consultar una sede por código (Priority: P1)

Como persona que explora el catálogo, con o sin sesión iniciada, quiero obtener los datos de una sede a partir de su código de negocio, para saber dónde se realiza un evento y cuál es su capacidad.

**Why this priority**: Toda creación de eventos depende de una sede existente, y la capacidad de la sede decide cuándo un evento se agota. Sin poder consultar una sede, el resto del catálogo no se puede verificar.

**Independent Test**: Se consulta una sede cargada en la semilla por su código y se comprueba que la respuesta trae sus datos. Luego se consulta un código que no existe y se comprueba que el sistema lo informa como no encontrado.

**Acceptance Scenarios**:

1. **Scenario**: Consulta de una sede existente
   - **Given** una sede activa con un código determinado
   - **When** se consulta la sede por ese código
   - **Then** el sistema devuelve el código, el nombre, la ciudad, la capacidad y el estado de la sede

2. **Scenario**: Consulta de una sede inactiva
   - **Given** una sede existente que no está activa
   - **When** se consulta la sede por su código
   - **Then** el sistema devuelve sus datos e indica que la sede no está activa

3. **Scenario**: Consulta de un código inexistente
   - **Given** que no existe ninguna sede con el código consultado
   - **When** se consulta la sede por ese código
   - **Then** el sistema informa que la sede no fue encontrada, identifica el código consultado y usa el contrato de errores común del sistema

4. **Scenario**: Consulta con otra capitalización del código
   - **Given** una sede cuyo código es `VEN-SMR-01`
   - **When** se consulta con el código `ven-smr-01`
   - **Then** el sistema informa que la sede no fue encontrada, porque la coincidencia del código es exacta

5. **Scenario**: Consulta sin sesión iniciada
   - **Given** un visitante que no ha iniciado sesión
   - **When** consulta una sede por su código
   - **Then** el sistema responde con los datos de la sede sin pedir credenciales

---

### User Story 2 - Consultar las sedes activas (Priority: P1)

Como persona que explora el catálogo, o como administrador que va a crear un evento, quiero ver la lista de sedes activas, para saber en cuáles se pueden programar eventos.

**Why this priority**: Un evento solo puede crearse en una sede activa. La lista de sedes activas es la base para elegir dónde programarlo.

**Independent Test**: Con sedes activas e inactivas cargadas en la semilla, se pide la lista de sedes activas y se comprueba que solo aparecen las activas, ordenadas por nombre.

**Acceptance Scenarios**:

1. **Scenario**: Lista de sedes activas
   - **Given** sedes activas e inactivas registradas
   - **When** se consultan las sedes activas
   - **Then** el sistema devuelve únicamente las sedes activas, ordenadas por nombre de forma ascendente

2. **Scenario**: No hay sedes activas
   - **Given** que ninguna sede está activa
   - **When** se consultan las sedes activas
   - **Then** el sistema responde con éxito y una lista vacía, no con un error

3. **Scenario**: Consulta sin sesión iniciada
   - **Given** un visitante que no ha iniciado sesión
   - **When** consulta las sedes activas
   - **Then** el sistema responde con la lista sin pedir credenciales

---

### Edge Cases

- **Sede inactiva**: se puede consultar por código, pero no aparece en la lista de sedes activas. Tampoco admite eventos nuevos (regla del área `eventos`).
- **Capitalización del código**: la coincidencia es exacta; `VEN-SMR-01` y `ven-smr-01` son códigos distintos.
- **Datos de origen**: las sedes se cargan solo con la semilla de datos (decisión D-06 de [`general.md`](../../arquitectura/general.md#puntos-abiertos-de-los-prd-y-cómo-se-resolvieron)). Ninguna operación de esta área crea, modifica ni elimina sedes.
- **Capacidad**: en esta área la capacidad es un dato informativo. La regla que impide vender más entradas que la capacidad pertenece al área `tickets`.

---

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: El sistema MUST permitir consultar una sede por su código de negocio.
- **FR-002**: La coincidencia del código MUST ser exacta, es decir, sensible a mayúsculas y minúsculas.
- **FR-003**: Si no existe una sede con el código consultado, el sistema MUST informar que no fue encontrada, identificar el código y usar el contrato de errores común (BR-VENUE-001).
- **FR-004**: La consulta por código MUST devolver también las sedes inactivas, indicando su estado.
- **FR-005**: El sistema MUST permitir consultar las sedes activas y MUST devolver únicamente las que están activas (BR-VENUE-002).
- **FR-006**: La lista de sedes activas MUST estar ordenada por nombre de forma ascendente.
- **FR-007**: Una lista de sedes activas vacía MUST responderse como éxito y no como error.
- **FR-008**: Ambas consultas MUST ser públicas: no exigen sesión iniciada ni rol.
- **FR-009**: El sistema MUST devolver datos de la sede mediante objetos de transferencia y nunca exponer entidades de persistencia.
- **FR-010**: El sistema MUST NOT ofrecer operaciones para crear, modificar o eliminar sedes. Las sedes se cargan con la semilla de datos.
- **FR-011**: La respuesta de una sede MUST incluir como mínimo código, nombre, ciudad, capacidad y estado activo. [NEEDS CLARIFICATION: la lista exacta de campos de la respuesta de sede depende del PRD de Persistencia, que aún no está disponible]

### Key Entities

- **Venue (sede)**: lugar donde se realizan los eventos. Atributos de negocio: código único, nombre, ciudad, capacidad (cantidad máxima de entradas pagadas por evento) y estado activo o inactivo. Una sede puede tener muchos eventos.

---

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: El 100 % de las sedes existentes se puede consultar por su código exacto y la respuesta coincide con los datos almacenados.
- **SC-002**: 0 % de las sedes inactivas aparece en la lista de sedes activas.
- **SC-003**: El 100 % de las consultas a un código inexistente se responde con el contrato de errores común e identifica el código consultado.
- **SC-004**: El 100 % de las consultas de esta área se resuelve sin credenciales.
