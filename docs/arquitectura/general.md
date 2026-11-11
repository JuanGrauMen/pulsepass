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
