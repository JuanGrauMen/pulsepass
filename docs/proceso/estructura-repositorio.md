# Estructura del repositorio — PulsePass

Este documento define dónde va cada archivo, cómo se agrupan las carpetas y cómo se nombran. La arquitectura y el stack se describen en [`general.md`](../arquitectura/general.md); aquí solo se aterrizan en carpetas. El flujo de Git está en [`CONTRIBUTING.md`](../../CONTRIBUTING.md).

Las carpetas `backend/` y `frontend/` se llenan a medida que avanzan los planes. Lo descrito para ellas es la estructura objetivo.

## Índice

1. [Principios](#1-principios)
2. [Vista general](#2-vista-general)
3. [Raíz del repositorio](#3-raíz-del-repositorio)
4. [Documentación (`docs/`)](#4-documentación-docs)
5. [Backend: código](#5-backend-código)
6. [Backend: recursos](#6-backend-recursos)
7. [Backend: pruebas](#7-backend-pruebas)
8. [Frontend](#8-frontend)
9. [Dónde se guarda cada dato](#9-dónde-se-guarda-cada-dato)
10. [Reglas de nombres](#10-reglas-de-nombres)
11. [Reglas de agrupación](#11-reglas-de-agrupación)
12. [Cambiar la estructura](#12-cambiar-la-estructura)

---

## 1. Principios

1. **Cada carpeta responde una sola pregunta.** Si un archivo no responde la pregunta de la carpeta, no va ahí.
2. **Una carpeta contiene archivos del mismo nivel o carpetas del mismo tipo, no ambas mezcladas.** Por ejemplo, `docs/features/` solo contiene carpetas de áreas funcionales.
3. **Cada documento existe en un solo lugar.** No hay copias del mismo archivo en dos rutas.
4. **Sin anidamiento redundante.** El repositorio es solo de PulsePass, así que no hay carpetas `pulsepass/` dentro.
5. **El alcance de commit indica el área funcional.** Un `feat(tickets)` toca clases de tickets del backend y la carpeta `features/tickets/` del frontend; un `docs(diseno)` toca `docs/arquitectura/`.
6. **Backend y frontend en el mismo repositorio, cada uno en su carpeta.** Un solo flujo de Git, una sola versión, y una funcionalidad completa (endpoint y pantalla) cabe en un solo PR. Ninguno importa archivos del otro: se comunican solo por la API REST.

---

## 2. Vista general

```
pulsepass/
├── .github/                    plantillas de PR e issues (y CI cuando exista)
├── docs/
│   ├── enunciado/              PRD de cada fase (no se editan)
│   ├── features/               un área funcional por carpeta (spec)
│   ├── planes/                 planes de implementación, uno por fase
│   ├── arquitectura/           stack, capas y diagramas
│   ├── pruebas/                casos de prueba
│   ├── plantillas/             moldes para specs, planes y casos de prueba
│   └── proceso/                forma de trabajo
├── backend/                    API REST (Java + Spring Boot + Maven)
│   ├── src/
│   │   ├── main/
│   │   │   ├── java/com/pulsepass/
│   │   │   │   ├── domain/         modelo, puertos y reglas puras
│   │   │   │   ├── application/    servicios que implementan los casos de uso
│   │   │   │   └── infrastructure/ Spring, REST, JPA, seguridad
│   │   │   └── resources/          configuración y migraciones
│   │   └── test/                   espejo de main
│   ├── .env.example
│   └── pom.xml
├── frontend/                   interfaz (React + TypeScript + Vite)
│   ├── public/
│   ├── src/
│   │   ├── app/                arranque, rutas, layout
│   │   ├── features/           una carpeta por área de pantallas
│   │   ├── shared/             lo que usan varias features
│   │   └── styles/             estilos globales
│   ├── .env.example
│   ├── index.html
│   ├── package.json
│   └── vite.config.ts
├── .editorconfig
├── .gitattributes
├── .gitignore
├── compose.yaml
├── CONTRIBUTING.md
├── LICENSE
└── README.md
```

---

## 3. Raíz del repositorio

En la raíz solo van archivos que aplican a **todo el repositorio** o que GitHub espera encontrar ahí. Lo que es solo del backend va en `backend/` y lo que es solo del frontend va en `frontend/`.

| Archivo / carpeta | Contenido |
|---|---|
| `README.md` | Presentación: alcance, fases, metodología y estado |
| `CONTRIBUTING.md` | Flujo de Git y forma de trabajo |
| `LICENSE` | Licencia MIT |
| `compose.yaml` | Entorno local: PostgreSQL |
| `.gitignore`, `.gitattributes`, `.editorconfig` | Configuración de Git y del editor para ambos proyectos |
| `.github/pull_request_template.md` | Plantilla de PR |
| `.github/ISSUE_TEMPLATE/` | Plantillas de issues |
| `.github/workflows/` | Integración continua (se agrega cuando exista código que compilar) |

No van en la raíz: `pom.xml`, `package.json`, `node_modules/`, otros documentos `.md`, diagramas, scripts sueltos, exportaciones.

---

## 4. Documentación (`docs/`)

```
docs/
├── enunciado/
│   └── README.md                         índice de los PRD (los PDF no se versionan)
├── features/                             SOLO carpetas de áreas funcionales
│   ├── venues/spec.md
│   ├── artistas/spec.md
│   ├── eventos/spec.md
│   ├── usuarios/spec.md
│   ├── tickets/spec.md
│   └── auth/spec.md
├── planes/                               cómo se construye cada fase
│   ├── 001-persistencia.md
│   ├── 002-servicios.md
│   ├── 003-controladores.md
│   ├── 004-seguridad.md
│   ├── 005-frontend.md
│   └── 006-pruebas-e2e.md
├── arquitectura/
│   ├── general.md                        stack, capas, reglas
│   └── diagramas/                        fuentes editables
├── pruebas/                              casos de prueba
├── plantillas/
│   ├── spec.md
│   ├── plan.md
│   └── test-case.md
└── proceso/
    ├── control-cambios.md                registro SC-<n>
    ├── estructura-repositorio.md         este documento
    └── guia-sdd.md                       cómo usar spec y plan (SDD)
```

| Carpeta | Pregunta que responde | Alcance de commit |
|---|---|---|
| `enunciado/` | ¿Qué pidieron los PRD? | `docs` |
| `features/<area>/spec.md` | ¿Qué debe hacer esta área? | `docs(requisitos)` |
| `planes/` | ¿Cómo se implementa esta fase? | `docs(plan)` |
| `arquitectura/` | ¿Con qué y cómo se construye todo el sistema? | `docs(diseno)` |
| `pruebas/` | ¿Cómo se verifica cada comportamiento? | `docs(pruebas)` |
| `plantillas/` | ¿Qué molde copio para un spec, plan o caso de prueba nuevo? | `docs` |
| `proceso/` | ¿Cómo trabaja el equipo? | `docs(cambios)` para el registro, `docs` para el resto |

Reglas:

- **Un área funcional = una carpeta** en `features/`, con su `spec.md` y nada más. Si necesita un diagrama propio, va en `features/<area>/diagramas/`.
- **Un plan por fase.** Cada plan corresponde a un PRD y lista al inicio, en `**Specs**:`, los specs que cubre con enlace. Todo spec debe estar cubierto por los planes que lo implementan.
- **Nombre de un plan:** `<nnn>-<descripcion>.md`, con número de tres dígitos consecutivo que no se reutiliza.
- **Nombre de un caso de prueba:** `tc-<area>-<nnn>-<descripcion>.md`, por ejemplo `tc-tickets-001-compra-valida.md`.
- **Las plantillas no se modifican.** Se copian a su carpeta y se completan.
- **Nada suelto en `features/`.** Un documento que hable de varias áreas va en `arquitectura/` (si es cómo) o en la sección de requisitos de cada spec.
- **Los diagramas se suben solo como fuente editable** (`.drawio`, `.puml`), sin exportaciones `.png` o `.svg`.
- **`enunciado/` no se edita.** Los cambios de alcance se registran en `proceso/control-cambios.md`.
- **Las entregas se identifican con tags de Git** (`v1.0.0`), no con carpetas de copias.

---

## 5. Backend: código

Paquete raíz: `com.pulsepass`. La organización es **por capa y, dentro de cada capa, por tipo de pieza**, como la define [`general.md`](../arquitectura/general.md#arquitectura-limpia-por-capas), que es la referencia oficial del árbol completo. No hay subpaquetes por área funcional: el área de una clase se reconoce por su nombre (`TicketServiceImpl`, `EventController`).

### Qué va en cada capa

| Capa | Puede contener | No puede contener | Puede depender de |
|---|---|---|---|
| `domain` | Modelo, enums, excepciones de negocio, interfaces de servicio (`port/in`) con sus DTO (`port/in/dto`), interfaces de repositorio y de seguridad (`port/out`), reglas puras | Anotaciones de Spring, `@Entity`, Bean Validation, MapStruct, `*Request`/`*Response` HTTP | Nada |
| `application` | `*ServiceImpl`, `@Service`, `@Transactional`, políticas de acceso | Controladores, entidades JPA, clases de JJWT | `domain` |
| `infrastructure` | Controladores, `*Request`/`*Response`, entidades JPA, mappers, seguridad, configuración | Reglas de negocio | `application`, `domain` |

### Áreas funcionales y alcances de commit

Las áreas no son paquetes, pero sí deciden el alcance del commit (ver [`CONTRIBUTING.md`](../../CONTRIBUTING.md#alcances)):

| Alcance | Clases típicas |
|---|---|
| `venues` | `Venue`, `VenueService`, `VenueServiceImpl`, `VenueController` |
| `artistas` | `Artist`, `ArtistService`, `ArtistServiceImpl`, `ArtistController` |
| `eventos` | `Event`, `EventService`, `EventServiceImpl`, `EventController` |
| `usuarios` | `User`, `UserProfile`, `UserService`, `UserServiceImpl`, `UserController` |
| `tickets` | `Ticket`, `TicketService`, `TicketServiceImpl`, `TicketController`, estrategia de precio |
| `auth` | `infrastructure/security/`, `AuthController`, políticas de propiedad |

---

## 6. Backend: recursos

```
resources/
├── application.yml                       configuración común
├── application-dev.yml                   PostgreSQL de compose.yaml
├── application-test.yml
├── application-prod.yml                  solo referencias a variables de entorno
└── db/
    └── migration/                        esquema (Flyway)
        ├── V1__create_<...>.sql
        └── ...
```

Las migraciones son incrementales: una migración aplicada no se edita. La numeración y el contenido se fijan en el plan 001. Si se agregan datos de demostración, van en `db/seed/` con numeración desde `V1000` para no chocar con el esquema.

---

## 7. Backend: pruebas

`backend/src/test/java` replica **exactamente** los paquetes de `backend/src/main/java`. La prueba de una clase está en el mismo paquete que la clase.

| Tipo de prueba | Sufijo | Ejemplo |
|---|---|---|
| Unitaria (dominio o servicio con Mockito) | `Test` | `TicketServiceImplTest` |
| Contrato HTTP con MockMvc | `Test` | `TicketControllerTest` |
| Integración (Testcontainers con PostgreSQL) | `IT` | `TicketPersistenceAdapterIT` |
| Arquitectura (ArchUnit) | `ArchitectureTest` | `ArchitectureTest` |

Los datos de prueba compartidos van en `backend/src/test/resources/`.

---

## 8. Frontend

React con TypeScript estricto y Vite. Estilos con Tailwind CSS v4: clases utilitarias en el propio componente y tokens del tema en `styles/global.css`. Pruebas con Vitest y Testing Library.

```
frontend/
├── public/                               archivos estáticos sin procesar
├── src/
│   ├── main.tsx                          punto de entrada
│   ├── app/                              App, router, Layout, proveedores (sesión)
│   ├── features/
│   │   ├── eventos/                      cartelera y detalle de evento
│   │   ├── tickets/                      compra y mis tickets
│   │   ├── auth/                         login y registro
│   │   ├── admin/                        crear y publicar eventos, asociar artistas
│   │   └── staff/                        validación de ingreso
│   ├── shared/
│   │   ├── api/                          cliente HTTP y manejo de errores
│   │   ├── components/
│   │   ├── hooks/
│   │   └── format/
│   └── styles/
│       └── global.css
├── .env.example
├── index.html
├── package.json
└── vite.config.ts
```

La organización definitiva de `features/` se fija en el plan 005, a partir de los diseños del PRD de Frontend.

### Reglas del frontend

1. **Las features no se importan entre sí.** Si dos features necesitan lo mismo, se mueve a `shared/`.
2. **Las llamadas HTTP solo van en archivos `<area>Api.ts`**, que usan el cliente de `shared/api/`. Los componentes nunca llaman a `fetch` directamente.
3. **Sin reglas de negocio en el frontend.** Edad mínima, capacidad, estados y permisos los decide el backend; las guardas de ruta son solo experiencia de usuario.
4. **Todo dato externo** (API, almacenamiento, parámetros de URL) se valida con zod.
5. **Las rutas se centralizan** en un solo módulo; no se escriben rutas sueltas.
6. **La prueba va junto al archivo que prueba:** `TicketCard.test.tsx` al lado de `TicketCard.tsx`.
7. **Los tipos de la API** reflejan los `Request`/`Response` del backend con los mismos nombres de campo.

---

## 9. Dónde se guarda cada dato

| Dato | Ruta | ¿Se sube a Git? |
|---|---|---|
| Configuración común del backend | `backend/src/main/resources/application.yml` | Sí |
| Configuración por entorno del backend | `backend/src/main/resources/application-<perfil>.yml` | Sí, sin secretos |
| Secretos (contraseña de la base de datos, clave del JWT) | Variables de entorno; en local, `backend/.env` | **No** |
| Lista de variables requeridas del backend | `backend/.env.example` | Sí, con valores de ejemplo |
| URL de la API para el frontend | `frontend/.env` (`VITE_API_BASE_URL`) | **No**; se sube `frontend/.env.example` |
| Esquema de base de datos | `backend/src/main/resources/db/migration/` | Sí |
| Colección de Postman | `docs/pruebas/postman/` | Sí, **sin** tokens ni valores de variables secretas |
| Exportaciones de entornos de Postman (`*.postman_environment.json`) | — | **No** |
| PRD en PDF | `docs/enunciado/` | **No** (se sube solo el índice `README.md`) |
| Diagramas | `docs/arquitectura/diagramas/` | Sí, solo la fuente editable |
| Compilados (`backend/target/`, `frontend/dist/`), `node_modules/`, IDE (`.idea/`, `.vscode/`), logs | — | **No** |
| Dependencias exactas del frontend | `frontend/package-lock.json` | **Sí** |

---

## 10. Reglas de nombres

### Carpetas y archivos de documentación

| Regla | Correcto | Incorrecto |
|---|---|---|
| Minúsculas y guiones (`kebab-case`) | `control-cambios.md` | `Control_Cambios.md`, `controlCambios.md` |
| Sin tildes, ñ, espacios ni caracteres especiales | `diseno-base-datos.drawio` | `diseño base datos.drawio` |
| Extensión en minúsculas | `guia.md` | `guia.MD` |
| Sin versión ni fecha en el nombre (Git guarda el historial) | `casos-uso.drawio` | `casos_uso_v2.drawio` |
| Nombres de carpeta en español | `arquitectura`, `plantillas` | `architecture`, `templates` |
| Excepción: archivos que GitHub nombra así | `README.md`, `CONTRIBUTING.md`, `LICENSE` | — |

Estas reglas aplican a `docs/`. El código sigue las convenciones de su lenguaje.

### Backend (Java)

| Elemento | Regla | Ejemplo |
|---|---|---|
| Paquete | Minúsculas, una palabra, sin tildes | `persistence`, `security` |
| Clase | `PascalCase`, en inglés como en los PRD | `EventServiceImpl` |
| Método y variable | `camelCase` | `findPublishedEvents` |
| Constante | `MAYUSCULAS_CON_GUION_BAJO` | `MAX_AGE` |

### Sufijos por rol

| Rol | Sufijo | Capa | Ejemplo |
|---|---|---|---|
| Entidad de dominio | sin sufijo | `domain/model` | `Ticket` |
| Interfaz de servicio (puerto de entrada) | `Service` | `domain/port/in` | `TicketService` |
| DTO de entrada de un caso de uso | `Command` | `domain/port/in/dto` | `PurchaseTicketCommand` |
| DTO de salida de un caso de uso | `Dto` | `domain/port/in/dto` | `TicketDto` |
| Puerto de salida de datos | `Repository` | `domain/port/out` | `TicketRepository` |
| Excepción de negocio | `Exception` | `domain/exception` | `BusinessRuleException` |
| Implementación del servicio | `ServiceImpl` | `application/service` | `TicketServiceImpl` |
| Controlador REST | `Controller` | `infrastructure/adapter/rest` | `TicketController` |
| DTO HTTP de entrada / salida | `Request` / `Response` | `infrastructure/adapter/rest` | `PurchaseTicketRequest`, `TicketResponse` |
| Entidad JPA | `Entity` | `infrastructure/adapter/persistence` | `TicketEntity` |
| Repositorio Spring Data | `JpaRepository` | `infrastructure/adapter/persistence` | `TicketJpaRepository` |
| Implementación de un puerto de datos | `PersistenceAdapter` | `infrastructure/adapter/persistence` | `TicketPersistenceAdapter` |
| Conversión entre modelos | `Mapper` | `infrastructure` | `TicketEntityMapper` |
| Configuración | `Config` | `infrastructure/config` | `SecurityConfig` |

### Frontend (React)

| Elemento | Regla | Ejemplo |
|---|---|---|
| Carpeta de feature | Minúsculas, una palabra | `tickets`, `eventos` |
| Carpetas técnicas | Inglés | `app`, `shared`, `components`, `hooks` |
| Componente | `PascalCase.tsx`, un componente exportado por archivo | `TicketCard.tsx` |
| Página (una ruta) | `PascalCase` + `Page` | `CarteleraPage.tsx` |
| Hook | `use` + `PascalCase`, `.ts` | `useTickets.ts` |
| Llamadas a la API | `<area>Api.ts` | `ticketsApi.ts` |
| Otros módulos TypeScript | `camelCase.ts` | `formatPrice.ts` |
| Prueba | Mismo nombre + `.test.tsx` / `.test.ts` | `formatPrice.test.ts` |
| Variable de entorno | `VITE_` + `MAYUSCULAS` | `VITE_API_BASE_URL` |

### Migraciones y recursos (backend)

| Elemento | Regla | Ejemplo |
|---|---|---|
| Migración Flyway | `V<n>__<verbo>_<objeto>.sql`, en inglés y `snake_case`; numeración consecutiva; una migración aplicada no se edita | `V4__add_auth_columns.sql` |
| Tablas y columnas | `snake_case`, en singular | `ticket`, `event_code` |
| Perfil de configuración | `application-<perfil>.yml` | `application-dev.yml` |

---

## 11. Reglas de agrupación

Estas reglas son del backend; las del frontend están en la [sección 8](#reglas-del-frontend).

1. **Capa primero, tipo de pieza después.** Una clase nueva se ubica respondiendo: ¿a qué capa pertenece? y ¿qué pieza es (modelo, puerto, servicio, adaptador)? El árbol oficial está en `general.md`.
2. **No se crean paquetes por área funcional** (`tickets/`, `eventos/`) ni paquetes de tipo fuera de los definidos (`models/`, `dtos/`, `utils/`). El sufijo y el nombre de la clase indican rol y área.
3. **Lo que usan varias áreas** va en `domain/shared/`. No se crean paquetes `util`, `common` ni `helpers`.
4. **Las pruebas siguen al código.** Si una clase se mueve de paquete, su prueba se mueve en el mismo commit.
5. **Una clase pública por archivo**, con el mismo nombre del archivo.
6. **Profundidad máxima:** la de `general.md`. Si hace falta un nivel más, se discute antes.
7. **`.gitkeep` solo mientras la carpeta esté vacía.** Se borra en el mismo commit que agrega el primer archivo real.

---

## 12. Cambiar la estructura

Agregar, renombrar o mover carpetas de primer o segundo nivel, crear un área funcional nueva o cambiar una regla de este documento:

1. Se acuerda con el equipo (igual que un alcance de commit nuevo).
2. Se actualiza este documento en una rama `docs/` con `docs(diseno)`.
3. Los movimientos de archivos se hacen con un commit `refactor` que **solo mueve**, sin cambiar contenido, para que Git conserve el historial.

```
refactor: mueve los puertos de salida a domain/port/out
```
