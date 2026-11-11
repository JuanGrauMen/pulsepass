# Guía de contribución — PulsePass

Este documento define el flujo de trabajo con Git del proyecto: ramas, nombres, commits, pull requests, entregas y control de cambios. Todo integrante del equipo debe seguirlo.

PulsePass es una plataforma de eventos, artistas y entradas que se construye por fases a partir de los documentos de requisitos listados en [`docs/enunciado/README.md`](docs/enunciado/README.md). La organización de carpetas y las reglas de nombres de archivos están en [`docs/proceso/estructura-repositorio.md`](docs/proceso/estructura-repositorio.md).

## Índice

1. [Ramas](#1-ramas)
2. [Nombres de rama](#2-nombres-de-rama)
3. [Commits](#3-commits)
4. [Qué rama y qué tipo usar](#4-qué-rama-y-qué-tipo-usar)
5. [Control de cambios](#5-control-de-cambios)
6. [Flujo diario](#6-flujo-diario)
7. [Pull requests](#7-pull-requests)
8. [Entregas y hotfix](#8-entregas-y-hotfix)
9. [Versionado](#9-versionado)
10. [Reglas de oro](#10-reglas-de-oro)

---

## 1. Ramas

```
main ─────●─────────────────────────●──────────────●─────►  versiones entregadas (tags)
           \                       / \            /
release     \                ●──●─┘   \          /
             \              /          \        /
develop ──────●──●──●──●──●──●──────────●──●───●─────────►  integración
                \  /  \  /                \   /
feature/*        ●●    \/                  hotfix/* (sale de main)
docs/*                 ●●
```

| Rama | Sale de | Se une a | Tipo de merge | Propósito |
|---|---|---|---|---|
| `main` | — | — | — | Código entregado y estable. Cada merge lleva un tag `vX.Y.Z` |
| `develop` | `main` | `release/*` | — | Integración del trabajo terminado |
| `feature/*` | `develop` | `develop` | **Squash** | Implementar una funcionalidad |
| `docs/*` | `develop` | `develop` | **Squash** | Cambios **solo** de documentación (plan, requisitos, diseño, control de cambios) |
| `release/*` | `develop` | `main` y `develop` | **Merge commit** | Preparar una entrega |
| `hotfix/*` | `main` | `main` y `develop` | **Merge commit** | Arreglo urgente sobre lo ya entregado |

`main` y `develop` están **protegidas**: no se permite push directo y todo entra por pull request con al menos 1 aprobación de otro integrante. La integración continua (CI) se agrega junto con el esqueleto del proyecto; cuando exista, también debe estar en verde.

### Qué pasa con la rama después del merge

| Rama | Después del merge |
|---|---|
| `main`, `develop` | Permanentes |
| `feature/*`, `docs/*` | Se borra. El trabajo queda en `develop` y el PR conserva el cambio. Si el equipo decide conservar una rama como registro, no se vuelve a trabajar sobre ella |
| `release/*`, `hotfix/*` | Se conserva. Su punto de entrega queda marcado con un tag |

---

## 2. Nombres de rama

```
feature/PULSE-<id>-<descripcion>     feature/PULSE-12-compra-ticket
docs/PULSE-<id>-<descripcion>        docs/PULSE-27-precio-por-tipo-ticket
release/v<X.Y.Z>                     release/v1.1.0
hotfix/v<X.Y.Z>-<descripcion>        hotfix/v1.1.1-edad-minima
```

| Regla | Correcto | Incorrecto |
|---|---|---|
| Usar el prefijo correspondiente | `feature/PULSE-12-...` | `PULSE-12-...`, `feat-...`, `Feature/...` |
| Incluir el ID de la tarea o issue | `feature/PULSE-12-compra-ticket` | `feature/compra-ticket` |
| Todo en minúsculas | `compra-ticket` | `Compra-Ticket` |
| Separar palabras con guiones | `evento-publicado` | `evento_publicado`, `eventoPublicado` |
| Sin tildes, ñ ni caracteres especiales | `edad-minima` | `edad-mínima`, `tickets#2` |
| Descripción de 2 a 5 palabras | `cancelacion-ticket` | `cancelar-un-ticket-pagado-antes-de-la-fecha-del-evento` |
| Describir qué se hace, no quién lo hace | `login-jwt` | `cambios-juan` |
| Una rama por tarea | `feature/PULSE-12-...` | `feature/PULSE-12-13-14-varios` |
| Nada genérico | `filtro-cartelera` | `fix`, `cambios`, `prueba` |

Si una tarea no tiene ID, primero se crea el issue y luego la rama.

Expresión regular de validación:

```
^(feature|docs)/PULSE-[0-9]+-[a-z0-9]+(-[a-z0-9]+){0,4}$|^release/v[0-9]+\.[0-9]+\.[0-9]+$|^hotfix/v[0-9]+\.[0-9]+\.[0-9]+-[a-z0-9-]+$
```

---

## 3. Commits

Se usa la convención [Conventional Commits](https://www.conventionalcommits.org/es/).

### Formato

```
<tipo>(<alcance>): <descripción>

[cuerpo opcional: por qué se hizo el cambio]

[pie opcional: Closes #n · Refs #n · Refs: SC-n · BREAKING CHANGE: ...]
```

### Tipos

| Tipo | Uso | Efecto en la versión |
|---|---|---|
| `feat` | Funcionalidad nueva o cambio de comportamiento | MENOR |
| `fix` | Corrección de un bug | PARCHE |
| `docs` | Solo documentación | — |
| `style` | Formato, sin cambios de lógica | — |
| `refactor` | Reestructura sin cambiar el comportamiento | — |
| `perf` | Mejora de rendimiento | PARCHE |
| `test` | Pruebas | — |
| `build` | Dependencias o empaquetado | — |
| `ci` | Pipelines de integración continua | — |
| `chore` | Mantenimiento general | — |
| `revert` | Revierte un commit anterior | Según lo revertido |

Para indicar un cambio incompatible se agrega `!` después del alcance (`feat(api)!: ...`) y un pie `BREAKING CHANGE: <detalle>`. Esto sube la versión MAYOR.

### Alcances

**Funcionales:**

| Alcance | Área |
|---|---|
| `venues` | Consulta de sedes: por código y sedes activas |
| `artistas` | Consulta de artistas: por id, por nombre artístico y activos |
| `eventos` | Creación, publicación, asociación de artistas y cartelera |
| `usuarios` | Registro con perfil y consulta de usuarios |
| `tickets` | Compra, cancelación, uso, capacidad, edad mínima y precio |
| `auth` | Inicio de sesión, JWT, roles y propiedad de recursos |

Un alcance funcional cubre backend y frontend: `feat(tickets)` puede tocar las clases de tickets del backend y `frontend/src/features/tickets/` en el mismo commit. El backend se organiza por capas, no por área (ver [estructura del repositorio](docs/proceso/estructura-repositorio.md#áreas-funcionales-y-alcances-de-commit)).

**Técnicos:**

| Alcance | Área |
|---|---|
| `api` | Capa de endpoints (`backend/.../infrastructure/adapter/rest/`) y cliente HTTP del frontend (`frontend/src/shared/api/`) |
| `db` | Esquema, migraciones y seeds (`backend/src/main/resources/db/`) |
| `ui` | Componentes, layout y estilos compartidos del frontend (`frontend/src/app/`, `shared/`, `styles/`) |
| `config` | Configuración del proyecto (Maven, Vite, Spring, compose, CI) |

**Documentación:**

| Alcance | Documento |
|---|---|
| `plan` | Planes de implementación: `docs/planes/` |
| `requisitos` | Specs de cada área (`docs/features/*/spec.md`) |
| `diseno` | Arquitectura y diagramas: `docs/arquitectura/` |
| `pruebas` | Casos de prueba: `docs/pruebas/` |
| `cambios` | [`docs/proceso/control-cambios.md`](docs/proceso/control-cambios.md) |

Los alcances van en minúsculas, son una sola palabra y no llevan tildes. No se agregan alcances nuevos sin acordarlo con el equipo. Si el cambio afecta muchas áreas, se omite el alcance.

### Descripción

| Regla | Correcto | Incorrecto |
|---|---|---|
| Verbo en presente, 3.ª persona | `agrega`, `corrige`, `elimina` | `agregué`, `agregar`, `agregando` |
| Empieza en minúscula | `agrega filtro` | `Agrega filtro` |
| Sin punto final | `agrega filtro` | `agrega filtro.` |
| Máximo 72 caracteres en la primera línea | — | — |
| En español y específica | `corrige cálculo de edad en la fecha del evento` | `fix`, `cambios`, `ya funciona` |
| Dice qué cambia, no cómo | `valida capacidad antes de vender` | `agrega un if en la línea 40` |

Verbos sugeridos:

- `feat`: agrega, implementa, calcula, permite, incorpora
- `fix`: corrige, soluciona, evita, valida
- `refactor`: extrae, separa, simplifica, renombra, mueve
- `docs`: documenta, actualiza, registra, describe, modifica
- `test`: agrega pruebas de, cubre
- `chore` / `build`: actualiza, configura, elimina

### Pies

| Pie | Uso |
|---|---|
| `Closes #12` | Cierra el issue al hacer merge |
| `Refs #12` | Menciona el issue sin cerrarlo |
| `Refs: SC-03` | Referencia una solicitud de cambio (ver [sección 5](#5-control-de-cambios)) |
| `BREAKING CHANGE: <detalle>` | Cambio incompatible |
| `Co-authored-by: Nombre <email>` | Trabajo en pareja |

### Ejemplos

```
feat(tickets): calcula precio según el tipo de ticket
feat(eventos): crea el evento en estado DRAFT
fix(tickets): corrige cálculo de edad en la fecha del evento
feat(auth): emite JWT firmado al iniciar sesión
test(tickets): cubre compra con capacidad agotada
docs(plan): detalla la estrategia de precio por tipo de ticket
docs(cambios): registra SC-02 por nuevo descuento STUDENT
chore: configura prettier y oxlint
```

```
fix(tickets): corrige cálculo de edad en la fecha del evento

La edad se calculaba contra la fecha de compra y no contra la fecha
del evento, así que un usuario que cumple años antes del evento era
rechazado por error.

Closes #18
```

### Buenas prácticas

- Un cambio lógico por commit.
- El proyecto debe compilar en cada commit.
- Commits pequeños y frecuentes.
- Revisar con `git diff --staged` antes de hacer commit.
- Nunca commitear `.env`, credenciales, el secreto del JWT (`PULSEPASS_JWT_SECRET_BASE64`), exportaciones de entornos de Postman ni `node_modules`.

---

## 4. Qué rama y qué tipo usar

| Situación | Rama | Tipo de commit |
|---|---|---|
| Se agrega una funcionalidad | `feature/` | `feat` |
| Cambia el comportamiento porque cambió el requisito | `feature/` | `feat` |
| Cambio incompatible (API, base de datos, flujo) | `feature/` | `feat!` + `BREAKING CHANGE` |
| El código no cumplía lo ya definido | `feature/` | `fix` |
| Bug urgente en una versión entregada | `hotfix/` | `fix` |
| Se modifica el plan pero aún no se implementa | `docs/` | `docs(plan)` + `docs(cambios)` |
| Reestructura sin cambiar el comportamiento | `feature/` | `refactor` |

Si el requisito **cambió**, es `feat`. Si el requisito **era el mismo** y el código no lo cumplía, es `fix`.

---

## 5. Control de cambios

El control de cambios registra las decisiones que modifican lo acordado en los documentos de requisitos, los specs y los planes. El registro está en [`docs/proceso/control-cambios.md`](docs/proceso/control-cambios.md). Cada solicitud tiene un ID `SC-<n>`.

### Cuándo se necesita una solicitud de cambio

| Necesita SC | No necesita SC |
|---|---|
| Cambiar una regla de negocio (`BR-*`) o la estrategia de precio | Corregir un bug (`fix`) |
| Agregar o quitar una funcionalidad | Refactorizar código |
| Cambiar un contrato HTTP o un DTO público | Mejorar estilos o textos de la interfaz |
| Desviarse de lo indicado en un documento de requisitos (PRD) | Corregir errores de redacción |
| Cambiar el alcance o una fecha de entrega | |

### Flujo

```
1. docs/PULSE-27-...    →  docs(cambios) + docs(plan)   Refs: SC-03 · Refs #27    → PR aprobado por el líder
2. feature/PULSE-27-... →  feat(tickets)!: ...          Refs: SC-03 · Closes #27  → PR revisado como implementación
```

**Paso 1: registrar y documentar el cambio**

1. Crear el issue y la rama `docs/PULSE-<id>-<descripcion>`.
2. Agregar la fila `SC-<n>` en `docs/proceso/control-cambios.md` con estado **Propuesto** y su análisis de impacto.
3. Actualizar el plan o los specs afectados.
4. Abrir el PR. Al aprobarlo, el estado de la SC pasa a **Aprobado** (o **Rechazado**, y el PR se cierra sin tocar el plan).

```
docs(plan): agrega descuento STUDENT al cálculo de precio

Se actualiza el plan para que el ticket STUDENT aplique un descuento
sobre el precio base. Pendiente de implementación.

Refs: SC-02
Refs #27
```

- Sin `!` ni `BREAKING CHANGE`, porque el documento no rompe el código.
- Con `Refs` en lugar de `Closes`, porque el issue sigue abierto.
- El PR de `docs/` es la **aprobación formal** de la solicitud y lo debe aprobar el líder del equipo.

**Paso 2: implementar el cambio**

```
feat(tickets)!: aplica descuento STUDENT al precio del ticket

BREAKING CHANGE: el precio de un ticket STUDENT deja de ser el precio base.
Refs: SC-02
Closes #27
```

Después del merge, la SC pasa a **Implementado**, y a **Verificado** cuando se confirme en la siguiente release.

Si hay una `release/*` abierta y el cambio de documentación debe ir en esa entrega, se hace directamente en esa `release/*`.

---

## 6. Flujo diario

```bash
# 1. Partir de develop actualizado
git switch develop
git pull
git switch -c feature/PULSE-12-compra-ticket

# 2. Trabajar con commits pequeños
git add -p
git commit -m "feat(tickets): calcula precio según el tipo de ticket"

# 3. Mantener la rama al día con develop
git fetch origin
git rebase origin/develop

# 4. Subir la rama y abrir un PR hacia develop
git push -u origin feature/PULSE-12-compra-ticket
```

Para cambios de documentación el flujo es igual, con una rama `docs/`.

### Corregir commits

| Situación | Comando | Condición |
|---|---|---|
| Corregir el mensaje del último commit | `git commit --amend` | Solo si no se ha subido |
| Agregar un archivo olvidado al último commit | `git add <archivo> && git commit --amend --no-edit` | Solo si no se ha subido |
| Deshacer el último commit conservando los cambios | `git reset --soft HEAD~1` | Solo si no se ha subido |
| Deshacer un commit ya subido | `git revert <hash>` | Siempre seguro |

---

## 7. Pull requests

- **Título:** con el mismo formato de un commit (`<tipo>(<alcance>): <descripción>`), por ejemplo `feat(tickets): calcula precio según el tipo de ticket`. Al hacer squash, GitHub usa el título del PR como mensaje del commit en `develop`. El ID de la tarea ya va en el nombre de la rama.
- **Destino:** `develop` para `feature/` y `docs/`; `main` para `release/` y `hotfix/`.
- **Requisitos:** al menos 1 aprobación de otro integrante. Los PR de control de cambios los aprueba el líder.
- **Descripción:** qué cambia, cómo probarlo, el issue relacionado (`Closes #12` o `Refs #12`) y la SC si aplica (`Refs: SC-03`).
- **Merge:**
  - `feature/` y `docs/` → **Squash and merge**. GitHub toma el título del PR y le agrega el número: `feat(tickets): calcula precio según el tipo de ticket (#12)`.
  - `release/` y `hotfix/` → **Create a merge commit**.
- **Después del merge:** borrar las ramas `feature/` y `docs/` (ver [sección 1](#qué-pasa-con-la-rama-después-del-merge)). Si hace falta más trabajo, crear una rama nueva desde `develop`.

---

## 8. Entregas y hotfix

### Release

Al cierre de una fase o antes de una entrega:

1. Crear `release/vX.Y.0` desde `develop`.
2. En esa rama solo se permiten correcciones, la actualización de la versión, el CHANGELOG y la documentación. No entran funcionalidades nuevas.
3. Abrir un PR hacia `main` y unir con **merge commit**.
4. Crear el tag `vX.Y.0` sobre `main`.
5. Unir `main` de vuelta a `develop`.
6. Marcar como **Verificado** en `docs/proceso/control-cambios.md` las SC incluidas en la entrega.

### Hotfix

1. Crear `hotfix/vX.Y.Z-<descripcion>` desde `main`.
2. Corregir y abrir un PR hacia `main`; unir con **merge commit**.
3. Crear el tag `vX.Y.Z`.
4. Unir también a `develop` (o a la `release/*` abierta, si existe).

Las ramas `release/*` y `hotfix/*` no se borran después del merge.

---

## 9. Versionado

Se usa [Versionado Semántico](https://semver.org/lang/es/) `MAYOR.MENOR.PARCHE`.

| Cambio | Versión |
|---|---|
| `BREAKING CHANGE` / `!` | MAYOR (`1.x.x` → `2.0.0`) |
| `feat` | MENOR (`1.1.x` → `1.2.0`) |
| `fix` / `perf` | PARCHE (`1.1.0` → `1.1.1`) |
| `docs`, `style`, `refactor`, `test`, `chore`, `build`, `ci` | No cambia la versión |

Durante el desarrollo se usa `v0.x`. La primera entrega formal es `v1.0.0`.

---

## 10. Reglas de oro

1. Nunca hacer push directo a `main` ni a `develop`.
2. Una tarea = una rama = un PR = un commit con squash en `develop`.
3. Squash en ramas cortas (`feature/`, `docs/`); merge commit en `release/` y `hotfix/`.
4. No reescribir (`amend`, `reset`, `rebase`) commits que otros ya descargaron. En ramas compartidas se usa `git revert`.
5. Borrar las ramas `feature/` y `docs/` después del merge; conservar `release/` y `hotfix/`.
6. Todo cambio debe poder rastrearse: rama → ID de tarea, commit → `Closes`/`Refs`, cambio de lo acordado → `SC-n`.
