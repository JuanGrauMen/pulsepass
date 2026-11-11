# Enunciado — Documentos de requisitos (PRD)

Esta carpeta guarda los documentos de requisitos del producto (PRD) que definen cada fase de PulsePass. **No se editan.** Los cambios de alcance o las desviaciones se registran en [`docs/proceso/control-cambios.md`](../proceso/control-cambios.md).

Los archivos PDF no se versionan en el repositorio (están en el `.gitignore`) y se comparten por fuera. Este índice sí se versiona.

| Fase | Documento | Fecha | Plan |
|---|---|---|---|
| 1. Persistencia | No disponible por ahora | — | 001 |
| 2. Servicios | `PRD_PulsePass_Capa_Servicios.pdf` | 2026-09-28 | 002 |
| 3. Controladores REST | `PRD_PulsePass_Capa_Controladores_v2.pdf` | 2026-10-05 | 003 |
| 4. Seguridad | `PRD_PulsePass_Capa_Seguridad_Spring_Security_JJWT.pdf` | 2026-10-11 | 004 |
| 5. Frontend | `PRD_PulsePass_Frontend_React_TypeScript_Tailwind.pdf` | 2026-11-09 | 005 |
| 6. Pruebas E2E con Postman | `PRD_PulsePass_Pruebas_E2E_Postman_Manual.pdf` | 2026-10-11 | 006 |

## Material relacionado que aún no está

- PRD de la fase 1 (persistencia).
- Los 32 diseños de alta fidelidad del frontend (carpeta `imagenes/` que cita el PRD de Frontend).
- Las guías técnicas que cita el PRD de Frontend: React con TypeScript, rutas con React Router v8 y Tailwind v4.

## Precedencia

Ante una divergencia entre documentos, prevalecen los contratos de las fases 3 y 4 sobre el PRD de Postman, según lo indica el propio PRD de pruebas.
