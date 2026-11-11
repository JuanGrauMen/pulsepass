# Feature Specification: Artistas – Consulta de artistas

**Created**: 2026-11-11  
**Área**: `artistas`  
**Rol principal**: Visitante anónimo (también aplica a `USER`, `STAFF` y `ADMIN`)  
**Fuentes**: PRD de Servicios (BR-ARTIST-001, BR-ARTIST-002, FR-SVC-009), PRD de Controladores (FR-CTRL-ART-001, FR-CTRL-ART-002, FR-CTRL-ART-003) y PRD de Seguridad (FR-SEC-ART-01, FR-SEC-ART-02, FR-SEC-ART-03)

---

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Consultar un artista por identificador (Priority: P1)

Como persona que explora el catálogo, con o sin sesión iniciada, quiero obtener los datos de un artista a partir de su identificador, para saber quién participa en un evento.

**Why this priority**: El administrador asocia artistas a los eventos mediante su identificador, y la cartelera muestra a los artistas de cada evento. Sin poder consultar un artista, esa información no se puede verificar.

**Independent Test**: Se consulta un artista cargado en la semilla por su identificador y se comprueba que la respuesta trae sus datos. Luego se consulta un identificador que no existe y se comprueba que el sistema lo informa como no encontrado.

**Acceptance Scenarios**:

1. **Scenario**: Consulta de un artista existente
   - **Given** un artista activo con un identificador determinado
   - **When** se consulta el artista por ese identificador
   - **Then** el sistema devuelve su identificador, su nombre artístico y su estado

2. **Scenario**: Consulta de un artista inactivo
   - **Given** un artista existente que no está activo
   - **When** se consulta por su identificador
   - **Then** el sistema devuelve sus datos e indica que el artista no está activo

3. **Scenario**: Consulta de un identificador inexistente
   - **Given** que no existe ningún artista con el identificador consultado
   - **When** se consulta el artista por ese identificador
   - **Then** el sistema informa que el artista no fue encontrado, identifica el identificador consultado y usa el contrato de errores común del sistema

4. **Scenario**: Consulta con un identificador no numérico
   - **Given** un identificador que no es un número, por ejemplo `abc`
   - **When** se consulta el artista por ese identificador
   - **Then** el sistema rechaza la consulta como solicitud inválida con el contrato de errores común, y no la trata como un artista no encontrado

5. **Scenario**: Consulta sin sesión iniciada
   - **Given** un visitante que no ha iniciado sesión
   - **When** consulta un artista por su identificador
   - **Then** el sistema responde con los datos del artista sin pedir credenciales

---

### User Story 2 - Consultar un artista por nombre artístico (Priority: P1)

Como persona que explora el catálogo, quiero encontrar a un artista escribiendo su nombre artístico, para ver sus datos sin conocer su identificador.

**Why this priority**: El público conoce a los artistas por su nombre, no por un identificador. Además, la búsqueda de eventos por artista se apoya en ese nombre.

**Independent Test**: Se consulta un artista de la semilla por su nombre artístico, una vez con la capitalización original y otra con otra distinta, y se comprueba que ambas devuelven el mismo artista.

**Acceptance Scenarios**:

1. **Scenario**: Consulta de un artista existente
   - **Given** un artista con un nombre artístico determinado
   - **When** se consulta por ese nombre artístico
   - **Then** el sistema devuelve su identificador, su nombre artístico y su estado

2. **Scenario**: Consulta con otra capitalización del nombre
   - **Given** un artista cuyo nombre artístico es `Solar Beat`
   - **When** se consulta con el nombre `solar beat`
   - **Then** el sistema devuelve el mismo artista, porque la búsqueda ignora mayúsculas y minúsculas

3. **Scenario**: Consulta de un artista inactivo
   - **Given** un artista existente que no está activo
   - **When** se consulta por su nombre artístico
   - **Then** el sistema devuelve sus datos e indica que el artista no está activo

4. **Scenario**: Consulta de un nombre inexistente
   - **Given** que no existe ningún artista con el nombre consultado
   - **When** se consulta el artista por ese nombre
   - **Then** el sistema informa que el artista no fue encontrado, identifica el nombre consultado y usa el contrato de errores común

5. **Scenario**: Consulta sin sesión iniciada
   - **Given** un visitante que no ha iniciado sesión
   - **When** consulta un artista por su nombre artístico
   - **Then** el sistema responde con los datos del artista sin pedir credenciales

---

### User Story 3 - Consultar los artistas activos (Priority: P1)

Como persona que explora el catálogo, o como administrador que va a armar la programación de un evento, quiero ver la lista de artistas activos, para saber con quiénes se puede contar.

**Why this priority**: Es la forma de descubrir qué artistas hay disponibles y la base para elegir a quién asociar a un evento.

**Independent Test**: Con artistas activos e inactivos cargados en la semilla, se pide la lista de artistas activos y se comprueba que solo aparecen los activos, ordenados por nombre artístico.

**Acceptance Scenarios**:

1. **Scenario**: Lista de artistas activos
   - **Given** artistas activos e inactivos registrados
   - **When** se consultan los artistas activos
   - **Then** el sistema devuelve únicamente los artistas activos, ordenados por nombre artístico de forma ascendente

2. **Scenario**: No hay artistas activos
   - **Given** que ningún artista está activo
   - **When** se consultan los artistas activos
   - **Then** el sistema responde con éxito y una lista vacía, no con un error

3. **Scenario**: Consulta sin sesión iniciada
   - **Given** un visitante que no ha iniciado sesión
   - **When** consulta los artistas activos
   - **Then** el sistema responde con la lista sin pedir credenciales

---

### Edge Cases

- **Artista inactivo**: se puede consultar por identificador y por nombre artístico, pero no aparece en la lista de artistas activos.
- **Capitalización**: el nombre artístico se busca ignorando mayúsculas y minúsculas. En cambio, el identificador es numérico y no tiene capitalización.
- **Nombre artístico repetido**: la búsqueda por nombre asume que identifica a un solo artista. [NEEDS CLARIFICATION: si el nombre artístico es único ignorando mayúsculas depende del PRD de Persistencia, que aún no está disponible]
- **Datos de origen**: los artistas se cargan solo con la semilla de datos (decisión D-06 de [`general.md`](../../arquitectura/general.md#puntos-abiertos-de-los-prd-y-cómo-se-resolvieron)). Ninguna operación de esta área crea, modifica ni elimina artistas.
- **Relación con eventos**: un artista puede participar en muchos eventos, y un evento puede tener muchos artistas. La asociación de artistas a eventos y la búsqueda de eventos por artista pertenecen al área `eventos`.

---

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: El sistema MUST permitir consultar un artista por su identificador.
- **FR-002**: El identificador MUST ser numérico. Un identificador que no lo sea MUST rechazarse como solicitud inválida usando el contrato de errores común.
- **FR-003**: Si no existe un artista con el identificador consultado, el sistema MUST informar que no fue encontrado, identificar el identificador y usar el contrato de errores común (BR-ARTIST-001).
- **FR-004**: El sistema MUST permitir consultar un artista por su nombre artístico, ignorando mayúsculas y minúsculas.
- **FR-005**: Si no existe un artista con el nombre consultado, el sistema MUST informar que no fue encontrado, identificar el nombre y usar el contrato de errores común (BR-ARTIST-001).
- **FR-006**: Las consultas por identificador y por nombre MUST devolver también los artistas inactivos, indicando su estado.
- **FR-007**: El sistema MUST permitir consultar los artistas activos y MUST devolver únicamente los que están activos (BR-ARTIST-002).
- **FR-008**: La lista de artistas activos MUST estar ordenada por nombre artístico de forma ascendente.
- **FR-009**: Una lista de artistas activos vacía MUST responderse como éxito y no como error.
- **FR-010**: Las tres consultas MUST ser públicas: no exigen sesión iniciada ni rol.
- **FR-011**: El sistema MUST devolver datos del artista mediante objetos de transferencia y nunca exponer entidades de persistencia.
- **FR-012**: El sistema MUST NOT ofrecer operaciones para crear, modificar o eliminar artistas. Los artistas se cargan con la semilla de datos.
- **FR-013**: La respuesta de un artista MUST incluir como mínimo su identificador, su nombre artístico y su estado activo. [NEEDS CLARIFICATION: la lista exacta de campos de la respuesta de artista depende del PRD de Persistencia, que aún no está disponible]

### Key Entities

- **Artist (artista)**: persona o grupo que se presenta en los eventos. Atributos de negocio: identificador numérico, nombre artístico y estado activo o inactivo. Se relaciona con muchos eventos y un evento se relaciona con muchos artistas.

---

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: El 100 % de los artistas existentes se puede consultar por su identificador y la respuesta coincide con los datos almacenados.
- **SC-002**: El 100 % de las búsquedas por nombre artístico devuelve el mismo artista sin importar las mayúsculas y minúsculas escritas.
- **SC-003**: 0 % de los artistas inactivos aparece en la lista de artistas activos.
- **SC-004**: El 100 % de las consultas a un identificador o nombre inexistente se responde con el contrato de errores común e identifica el valor consultado.
- **SC-005**: El 100 % de los identificadores no numéricos se rechaza como solicitud inválida.
- **SC-006**: El 100 % de las consultas de esta área se resuelve sin credenciales.
