# Control de cambios — PulsePass

Registro de las solicitudes de cambio (SC) sobre lo acordado: reglas de negocio, funcionalidades, contratos, alcance, fechas y desviaciones respecto a los documentos de requisitos (PRD). El proceso se describe en [`CONTRIBUTING.md`](../../CONTRIBUTING.md#5-control-de-cambios).

## Estados

| Estado | Significado |
|---|---|
| **Propuesto** | Registrada en un PR de `docs/`, pendiente de aprobación |
| **Aprobado** | PR de `docs/` aprobado por el líder; plan actualizado |
| **Rechazado** | No se aplicará; se deja registrado el motivo |
| **Implementado** | El código del cambio está en `develop` |
| **Verificado** | El cambio está incluido en una versión entregada |

## Registro

Los ID de solicitud llevan dos dígitos (`SC-03`) y no deben confundirse con los criterios de éxito de las specs, que llevan tres (`SC-003`). Fuera de este archivo se citan como "solicitud SC-03" o con `Refs: SC-03`.

| ID | Fecha | Solicitante | Descripción | Motivo | Impacto (alcances / esfuerzo) | Estado | Aprobó | Issue | Versión |
|---|---|---|---|---|---|---|---|---|---|
| SC-01 | 2026-11-11 | Equipo | Implementar el backend con arquitectura limpia por capas en lugar de la estructura plana de paquetes que sugieren los PRD (ver detalle) | Aislar las reglas de negocio de Spring y JPA y poder verificar las dependencias con ArchUnit | `api`, `db`, `config`, `eventos`, `tickets`, `usuarios`, `auth` · arquitectura general, planes 002 a 004 · transversal a todas las fases | Propuesto | | — | |

## Detalle de solicitudes

### SC-01 — Arquitectura limpia por capas frente a los PRD

- **Situación en los PRD:** el PRD de Servicios sugiere los paquetes `domain`, `repository`, `dto`, `mapper`, `exception` y `service`, con entidades JPA como modelo de dominio, MapStruct de entidad a DTO y pruebas de servicio con un `Mapper` simulado.
- **Cambio propuesto:**
  1. Capas `domain`, `application` e `infrastructure`, con la regla de dependencias de [`general.md`](../arquitectura/general.md).
  2. Las interfaces `Service` y sus nombres del PRD se conservan, ubicadas como puertos de entrada en `domain/port/in`. Las `*ServiceImpl` van en `application/service`.
  3. Las entidades JPA pasan a `infrastructure` y son distintas del modelo de dominio; un `*PersistenceAdapter` implementa los puertos de salida (`Repository`).
  4. Los Service reciben y devuelven `*Command` y `*Dto` de dominio. Los `*Request` y `*Response` del PRD se mantienen como contrato HTTP en `infrastructure`.
  5. MapStruct se usa solo en `infrastructure`. El paso modelo a DTO usa métodos estáticos en los `record`. Por eso las pruebas de servicio no simulan un `Mapper`.
  6. Se incorpora ArchUnit para verificar las dependencias.
- **No cambia:** los 20 endpoints y sus respuestas, las reglas de negocio `BR-*`, el uso de Maven, Java 21, Spring Boot 4, JJWT y los criterios de aceptación de cada PRD.
- **Impacto:** más clases y mapeos que en la estructura plana; a cambio, un dominio sin frameworks y dependencias verificables.
