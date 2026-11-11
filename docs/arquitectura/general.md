# General — Arquitectura y Stack Tecnológico

**Estado**: Referencia técnica transversal de PulsePass  
**Fecha**: 2026-11-11  
**Origen**: PRD de las fases 2 a 6 (ver [`docs/enunciado/README.md`](../enunciado/README.md)). El PRD de la fase 1 (persistencia) no está disponible: el modelo de datos se reconstruye desde el PRD de Servicios y se contrastará cuando se consiga.

## Propósito

Este documento define la arquitectura, las tecnologías y las reglas técnicas comunes de PulsePass. Es la fuente única de verdad para estas decisiones: los planes de implementación enlazan este archivo y documentan solo su alcance, contratos y tareas. Ningún plan repite lo que está aquí.

Las desviaciones respecto a lo que indican los PRD están registradas en [`docs/proceso/control-cambios.md`](../proceso/control-cambios.md) (SC-01).

---

## Stack tecnológico

| Área | Tecnología | Uso |
|---|---|---|
| Lenguaje | Java 21 | Dominio, casos de uso, adaptadores y pruebas |
| Framework | Spring Boot 4.x | Configuración y composición de la aplicación REST |
| Construcción | Maven | Dependencias, compilación y empaquetado (`mvn clean test`) |
| API HTTP | Spring MVC | Controladores REST |
| Validación | Bean Validation | Validación estructural de los `*Request` |
| Persistencia | Spring Data JPA + Hibernate | Entidades y repositorios en infraestructura |
| Base de datos | PostgreSQL | Datos de la aplicación |
| Migraciones | Flyway | Esquema versionado; las migraciones aplicadas no se editan |
| Mapeo | MapStruct | Solo en `infrastructure` (ver reglas de dependencia) |
| Seguridad | Spring Security 7.x + JJWT 0.13.x | JWT HS256 de 15 minutos, roles `USER`, `STAFF`, `ADMIN` |
| Pruebas | JUnit 5, Mockito, AssertJ | Unitarias de dominio y servicios |
| Pruebas HTTP | MockMvc | Contrato HTTP de controladores y seguridad |
| Pruebas de integración | Testcontainers | PostgreSQL real en pruebas de persistencia (`*IT`) |
| Arquitectura | ArchUnit | Verifica las reglas de dependencia entre capas (incorporado por SC-01) |
| Entorno local | Docker Compose | PostgreSQL para desarrollo |
| Frontend | Node 22.22+, Vite, React 19.2.7+, TypeScript estricto, Tailwind CSS v4 | SPA |
| Frontend: ruteo y datos | React Router v8, TanStack Query | Rutas con guardas por rol; estado de servidor |
| Frontend: formularios | React Hook Form + zod | Formularios y validación de lo que entra desde la API |
| Frontend: pruebas | Vitest + Testing Library | Componentes, guardas, formularios y mapeo de errores |
| Pruebas E2E | Postman (ejecución manual) | Fase 6, sin scripts ni ejecución automática |

Pendiente de definir en el plan 001: formateador de código (por ejemplo Spotless), configuración de las pruebas de integración (`mvn verify`) y versión de PostgreSQL.

---

## Arquitectura limpia por capas

El backend se implementa con **arquitectura limpia por capas**. Las dependencias siempre apuntan hacia adentro: la infraestructura depende de la aplicación, y la aplicación depende del dominio. El dominio no conoce nada de las capas externas.

```text
com.pulsepass/
├── PulsepassApplication.java
│
├── domain/                            ← Núcleo. Java puro.
│   ├── model/                         ← Venue, Event, Artist, User, UserProfile, Ticket
│   │                                    y enums: EventCategory, EventStatus, TicketType, TicketStatus, Role
│   ├── exception/                     ← ResourceNotFoundException, BusinessRuleException, DuplicateResourceException
│   ├── port/
│   │   ├── in/                        ← VenueService, EventService, ArtistService, UserService, TicketService
│   │   │   └── dto/                   ← Records de los casos de uso: *Command (entrada) y *Dto (salida)
│   │   └── out/                       ← VenueRepository, EventRepository, ArtistRepository,
│   │                                    UserRepository, TicketRepository y puertos de seguridad
│   └── shared/                        ← Reglas transversales (por ejemplo, cálculo de edad)
│
├── application/                       ← Orquesta el dominio.
│   └── service/                       ← VenueServiceImpl, EventServiceImpl, ArtistServiceImpl,
│                                        UserServiceImpl, TicketServiceImpl; límite transaccional
│
└── infrastructure/                    ← Adaptadores técnicos. Conoce Spring, JPA, JJWT.
    ├── adapter/
    │   ├── rest/                      ← *Controller, *Request, *Response, ErrorResponse
    │   └── persistence/               ← *Entity, *JpaRepository, *PersistenceAdapter, *EntityMapper (MapStruct)
    ├── security/                      ← SecurityFilterChain, filtro JWT, JJWT, AuthenticationEntryPoint, AccessDeniedHandler
    ├── config/                        ← Configuración de Spring, Clock, propiedades
    └── exception/                     ← GlobalExceptionHandler
```

### Reglas de dependencia

1. `domain` **nunca** importa clases de `application`, de `infrastructure` ni de frameworks (Spring, JPA, Jackson, MapStruct, Bean Validation).
2. `application` solo depende de `domain`. Define el límite transaccional (`@Transactional`) y no conoce controladores, entidades JPA ni JWT.
3. `infrastructure` depende de `application` y `domain`. No contiene reglas de negocio.
4. Los `*Request` y `*Response` HTTP viven en `infrastructure` y se traducen a los `*Command` y `*Dto` del dominio. Los nombres y campos del PRD de Controladores se conservan en el contrato HTTP.
5. El paso modelo a DTO se hace con métodos estáticos en los `record` de dominio. MapStruct se usa solo en `infrastructure`: entidad a modelo, request a command y dto a response.
6. ArchUnit verifica estas reglas en una prueba de arquitectura; un PR que las rompa no pasa la revisión.

### Dónde vive cada concepto de los PRD

| Concepto del PRD | Ubicación en PulsePass |
|---|---|
| Interfaces `Service` | `domain/port/in` |
| Implementaciones `@Service` (`*ServiceImpl`) | `application/service` |
| `Repository` con sus consultas (`findByEventCode`...) | Interfaz en `domain/port/out`; Spring Data como `*JpaRepository` en `infrastructure`; el `*PersistenceAdapter` implementa el puerto |
| Entidades JPA | `infrastructure/adapter/persistence` como `*Entity`; el modelo puro está en `domain/model` |
| DTO de entrada y salida de los Service | `*Command` y `*Dto` en `domain/port/in/dto` |
| `CreateEventRequest`, `EventResponse` y demás DTO HTTP | `infrastructure/adapter/rest` |
| MapStruct | `infrastructure` |
| Excepciones personalizadas | `domain/exception` |
| `GlobalExceptionHandler` y `ErrorResponse` | `infrastructure/exception` y `infrastructure/adapter/rest` |
| Spring Security, filtro Bearer y JJWT | `infrastructure/security` |
| Propiedad de recursos (un `USER` solo opera sus tickets) | Política en `application`, apoyada en un puerto que entrega el usuario autenticado; el detalle se fija en el plan de seguridad |

---

## Contrato de errores

Todos los errores HTTP usan `ErrorResponse`, definido en el PRD de Controladores:

```java
public record ErrorResponse(
    LocalDateTime timestamp,
    int status,
    String error,
    String message,
    Map<String, String> details
) {}
```

Ejemplo de error de validación (`400`): `message` es `Validation failed` y `details` mapea cada campo a su mensaje. Los errores producidos por el filtro de seguridad (`401`, `403`) se devuelven con el mismo contrato mediante `AuthenticationEntryPoint` y `AccessDeniedHandler`.

> **Pendiente de aclarar**: el PRD de Seguridad habla de un contrato de "seis campos", pero el PRD de Controladores, el de Postman y el de Frontend muestran cinco (los de arriba). Se adopta el de cinco campos hasta aclararlo.

---

## Catálogo de endpoints

20 operaciones de negocio (PRD de Controladores) más 2 de autenticación (PRD de Seguridad).

| Área | Método y ruta | Operación |
|---|---|---|
| Venues | `GET /api/venues/{code}` | `findByCode` |
| Venues | `GET /api/venues/active` | `findActiveVenues` |
| Eventos | `POST /api/events` (201) | `create` |
| Eventos | `GET /api/events/{eventCode}` | `findByCode` |
| Eventos | `GET /api/events/published` | `findPublishedEvents` |
| Eventos | `PATCH /api/events/{eventCode}/publish` | `publish` |
| Eventos | `POST /api/events/{eventCode}/artists/{artistId}` | `addArtist` |
| Eventos | `GET /api/events/by-artist?stageName=` | `findByArtist` |
| Artistas | `GET /api/artists/{id}` | `findById` |
| Artistas | `GET /api/artists/by-stage-name?stageName=` | `findByStageName` |
| Artistas | `GET /api/artists/active` | `findActiveArtists` |
| Usuarios | `POST /api/users` (201) | `register` |
| Usuarios | `GET /api/users/by-email?email=` | `findByEmail` |
| Usuarios | `GET /api/users/by-username?username=` | `findByUsername` |
| Tickets | `POST /api/tickets` (201) | `purchase` |
| Tickets | `GET /api/tickets/{ticketCode}` | `findByCode` |
| Tickets | `GET /api/tickets/by-user?email=` | `findByUserEmail` |
| Tickets | `GET /api/events/{eventCode}/tickets/paid` | `findPaidTicketsByEvent` |
| Tickets | `PATCH /api/tickets/{ticketCode}/cancel` | `cancel` |
| Tickets | `PATCH /api/tickets/{ticketCode}/use` | `markAsUsed` |
| Auth | `POST /api/auth/login` | Inicio de sesión |
| Auth | `GET /api/auth/me` | Identidad autenticada |

Las políticas de rol y de propiedad de cada ruta se definen en el plan de seguridad.

---

## Estados

```text
Evento:   DRAFT → PUBLISHED → SOLD_OUT → FINISHED
          DRAFT → CANCELLED      PUBLISHED → CANCELLED

Ticket:   PAID → USED
          PAID → CANCELLED
```

Una compra válida genera un ticket `PAID`. Al completarse la capacidad de la sede, el evento pasa a `SOLD_OUT` en la misma transacción.

---

## Puntos abiertos de los PRD y cómo se resolvieron

Los PRD dejan algunas decisiones abiertas. Esta tabla las registra con su resolución. No son solicitudes de cambio, porque no contradicen lo acordado: eligen entre opciones que el PRD permitía. Cada decisión se concreta en un spec, un plan y un caso de prueba; los enlaces se agregan cuando esos documentos existan.

| ID | Punto abierto en el PRD | Decisión | Se concreta en |
|---|---|---|---|
| D-01 | Un `USER` consulta un recurso ajeno: `403` o `404` (Seguridad, secciones 11.1 y 13) | Responde **`404`**, para no revelar si el recurso existe. Aplica a la consulta y la cancelación de tickets ajenos y a la consulta de otros usuarios por un `USER` | Spec `auth`, plan 004 y un caso de prueba |
| D-02 | Alcance de `ADMIN` ("ADMIN autorizado", "con propósito autorizado"; Seguridad, sección 11) | `ADMIN` consulta cualquier usuario, consulta los tickets de cualquier usuario y cancela cualquier ticket. La regla del `404` no aplica a `ADMIN` | Spec `auth`, plan 004 |
| D-03 | Alcance de `STAFF` (rol global, limitación aceptada; Seguridad, sección 11.1) | `STAFF` consulta los tickets pagados de un evento, marca tickets como usados y consulta un ticket por código. No hay asignación de personal por evento | Spec `auth`, spec `tickets`, plan 004 |
| D-04 | Estrategia de precio del ticket (Servicios, sección 28: "una implementación académica puede definir...") | Precio base único y configurable (valor inicial 50.000 COP). Multiplicadores: `GENERAL` 1,00; `STUDENT` 0,75 (descuento del 25 %); `VIP` 2,00; `BACKSTAGE` 3,00. `BigDecimal` con 2 decimales y redondeo `HALF_UP`. La política es pura y vive en el dominio; el evento no lleva precio | Spec `tickets`, plan 002 |
| D-05 | Cómo se crean `STAFF` y `ADMIN` (Seguridad, sección 9.2; Pruebas E2E, sección 5) | Nunca por HTTP. Se cargan con la [semilla de datos](#semilla-de-datos-perfil-dev) del perfil `dev` | Plan 001 y plan 004 |
| D-06 | Sedes y artistas no tienen endpoint de alta (Controladores, sección 8: solo consultas) | Se cargan con la misma semilla; sin ella no se pueden crear eventos | Plan 001 |
| D-07 | Contrato de errores de cinco o de seis campos (ver [Contrato de errores](#contrato-de-errores)) | **Pendiente.** Se adopta el de cinco campos hasta aclararlo | Plan 003 |

---

## Semilla de datos (perfil `dev`)

Las 20 operaciones no crean sedes ni artistas, el registro público solo crea usuarios con rol `USER`, y `STAFF` y `ADMIN` no se crean por HTTP. Por eso el sistema necesita datos iniciales, y se cargan con SQL, no con endpoints.

### Cómo se carga

- Son **migraciones de Flyway** escritas en SQL, en `backend/src/main/resources/db/seed/`.
- Su numeración empieza en `V1000`, para no chocar con las migraciones del esquema de `db/migration/`.
- Solo se aplican con el perfil `dev` (la ubicación `db/seed/` se agrega a Flyway únicamente en `application-dev.yml`). Nunca se cargan en producción.
- Para repetir un escenario se recrea la base de datos y se vuelven a aplicar las migraciones y la semilla. El PRD de pruebas recomienda una base dedicada (`pulsepass_e2e`).

### Qué contiene

| Script | Contenido | Para qué |
|---|---|---|
| `V1000` identidades | Un `ADMIN`, un `STAFF` y un usuario inactivo, cada uno con su credencial y su rol | Probar roles, el acceso privilegiado y la regla de usuario inactivo |
| `V1001` catálogo | Una sede activa de capacidad pequeña y una sede inactiva; tres artistas activos y uno inactivo; tres eventos: uno en borrador, uno publicado y uno por publicar | Fixtures mínimos del PRD de pruebas, y el escenario de capacidad y `SOLD_OUT` del PRD de Servicios |

Los códigos y valores exactos (por ejemplo, la sede `VEN-SMR-01` y los artistas del escenario de aceptación del PRD de Servicios) se fijan en el plan 001.

### Contraseñas y fechas

- Las contraseñas se guardan con BCrypt. **Ninguna contraseña ni hash se sube al repositorio.**
- Como un script SQL no puede leer variables de entorno, el SQL usa **marcadores (placeholders) de Flyway** y el valor real sale de variables de entorno definidas en `backend/.env`. Cada persona que desarrolla genera localmente su hash BCrypt. Los nombres de las variables se definen en el plan 001 y se agregan a `backend/.env.example` sin valores.
- Verificación pendiente: confirmar en el plan 001 que los placeholders funcionan con la versión de Flyway del proyecto.
- Las fechas de los eventos se calculan con la fecha de carga más un intervalo (`now() + intervalo`), para que no queden vencidas con el paso de los días.

### Alcance

La semilla sirve para desarrollo y para las pruebas E2E. Un proceso administrativo real para producción queda fuera del alcance del proyecto.

---

## Estrategia de pruebas

| Tipo | Qué prueba | Herramientas | Sufijo |
|---|---|---|---|
| Unitaria de dominio | Reglas puras (por ejemplo, edad y precio) | JUnit 5, AssertJ | `Test` |
| Unitaria de servicio | `*ServiceImpl` con los puertos `out` simulados; sin Spring ni base de datos | JUnit 5, Mockito, AssertJ | `Test` |
| Contrato HTTP | Controladores y seguridad | MockMvc | `Test` |
| Integración | Adaptadores de persistencia con PostgreSQL real | Testcontainers | `IT` |
| Arquitectura | Reglas de dependencia entre capas | ArchUnit | `ArchitectureTest` |
| Frontend | Componentes, guardas, formularios, errores | Vitest, Testing Library | `.test.tsx` |
| End-to-end | API real con los tres roles | Postman manual | casos en `docs/pruebas/` |

Cada prueba sigue el patrón Arrange, Act, Assert. En las pruebas de servicio, los caminos inválidos verifican con `verify(..., never())` que no se persistió nada.

---

## Seguridad en una frase

API sin sesión: el cliente envía `Authorization: Bearer <JWT>`. El JWT se firma con HS256 con una clave tomada de la variable de entorno `PULSEPASS_JWT_SECRET_BASE64` (la aplicación falla al arrancar si falta). Las contraseñas se guardan con hash BCrypt. Una solicitud autenticada no implica acceso ilimitado: cada operación cumple una política de rol y, cuando corresponde, de propiedad del recurso.
