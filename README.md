# PulsePass

Plataforma para descubrir eventos y gestionar entradas de conciertos, festivales, conferencias, eventos universitarios, competencias deportivas y actividades culturales. Es un caso de estudio académico que se construye por fases, desde la persistencia hasta el frontend.

## Autor

- Juan Grau

## Qué permite hacer

| Rol | Puede |
|---|---|
| Visitante anónimo | Ver sedes, artistas y eventos publicados; registrarse e iniciar sesión |
| `USER` | Comprar tickets, consultar y cancelar los propios |
| `STAFF` | Consultar los tickets pagados de un evento y marcarlos como usados en el ingreso |
| `ADMIN` | Crear eventos, publicarlos y asociar artistas |

Las reglas de negocio (edad mínima, capacidad del recinto, estados del evento y del ticket, unicidad de usuarios) viven en el backend. El frontend las muestra, no las decide.

La API expone 20 operaciones de negocio (2 de sedes, 6 de eventos, 3 de artistas, 3 de usuarios y 6 de tickets) y 2 rutas de autenticación (`login` y `me`). El catálogo completo está en [`docs/arquitectura/general.md`](docs/arquitectura/general.md).

## Fases

| Fase | Contenido | Plan |
|---|---|---|
| 1 | Persistencia: modelo relacional, JPA, Flyway, PostgreSQL | 001 |
| 2 | Servicios: reglas de negocio, DTOs, transacciones, pruebas unitarias | 002 |
| 3 | Controladores REST: endpoints, validación, manejo de errores | 003 |
| 4 | Seguridad: Spring Security, JWT, roles y propiedad de recursos | 004 |
| 5 | Frontend: React, TypeScript y Tailwind | 005 |
| 6 | Pruebas end-to-end manuales con Postman | 006 |

Cada fase tiene un documento de requisitos (PRD) y un plan de implementación propio. Los PRD se listan en [`docs/enunciado/`](docs/enunciado/README.md).

## Metodología

El proyecto sigue Specification-Driven Development (SDD):

1. **SPEC** (`docs/features/`): qué se construye, por área funcional.
2. **PLAN** (`docs/planes/`): cómo se construye, con diseño técnico, contratos y tareas.
3. **Implementación**: se ejecuta el plan.
4. **Casos de prueba** (`docs/pruebas/`): cómo se verifica.

Las plantillas están en [`docs/plantillas/`](docs/plantillas/) y la guía en [`docs/proceso/guia-sdd.md`](docs/proceso/guia-sdd.md).

## Arquitectura y tecnologías

Backend con arquitectura limpia por capas (`domain`, `application`, `infrastructure`), descrita en [`docs/arquitectura/general.md`](docs/arquitectura/general.md).

- **Backend:** Java 21, Spring Boot 4, Spring Data JPA, Spring Security y JJWT, PostgreSQL, Flyway, MapStruct y Maven.
- **Frontend:** React 19, TypeScript, Vite, Tailwind CSS v4, React Router v8 y TanStack Query.
- **Pruebas:** JUnit 5, Mockito, AssertJ, MockMvc, Testcontainers, Vitest y Postman.

## Estado

Documentación base creada. El código del backend y del frontend se construye siguiendo los planes de `docs/planes/`. Las instrucciones para ejecutarlo se agregarán cuando exista el esqueleto del proyecto.

## Cómo contribuir

El flujo de ramas, commits y pull requests está en [`CONTRIBUTING.md`](CONTRIBUTING.md). Dónde va cada archivo y cómo se nombra está en [`docs/proceso/estructura-repositorio.md`](docs/proceso/estructura-repositorio.md).

## Licencia

[MIT](LICENSE). La licencia cubre el código y la documentación propia. Los documentos de requisitos del docente no se incluyen en el repositorio.
