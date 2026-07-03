---
categoria: feature
severidad: alta
esfuerzo: alto
estado: diseno-aprobado
---

# Modelo de Edición + selector, con edición actual forzada desde el admin

**Spec de diseño:** `docs/superpowers/specs/2026-07-03-multi-edicion-design.md` (aprobado)

## Estado

El diseño ya fue discutido y aprobado. Esta issue quedó como **índice**: la implementación real está partida en sub-issues secuenciales (`28-01` a `28-10`) más dos oportunistas (`28-11`, `28-12`) que aprovechan que ya se está tocando `views.py`/dashboards admin para el resto del backlog.

**Prerrequisito de secuencia:** hacer primero todo lo de la sección "Seguridad" de `PLAN.md` (issues 12, 20, 21, 23, 24) — en particular [[20-seguridad-permisos-login-required-vs-staff]], porque `28-08` agrega superficie nueva al panel admin y conviene que ya use el decorador correcto desde el día uno.

## Orden de implementación

| # | Sub-issue | Depende de |
|---|---|---|
| 1 | [28-01 — Modelo Edition y edición inicial](28-01-modelo-edition-y-migracion-inicial.md) | — |
| 2 | [28-02 — FK Edition en modelos existentes](28-02-fk-edition-en-modelos-existentes.md) | 28-01 |
| 3 | [28-03 — Talk.date → DateField + time_start](28-03-talk-date-datefield-y-time-start.md) | 28-02 |
| 4 | [28-04 — Reclamo: fechas a ISO](28-04-reclamo-fechas-iso.md) | 28-03 |
| 5 | [28-05 — CertificateConfig por edición](28-05-certificateconfig-por-edicion.md) | 28-02 |
| 6 | [28-06 — Cierre automático de inscripción/cancelación](28-06-cierre-automatico-inscripcion.md) | 28-03 |
| 7 | [28-07 — URL /e/&lt;slug&gt;/ y selector público](28-07-url-scheme-selector-publico.md) | 28-01, 28-02 |
| 8 | [28-08 — Selector de sesión en admin + paginación](28-08-selector-admin-y-paginacion.md) | 28-02, [[20-seguridad-permisos-login-required-vs-staff]] |
| 9 | [28-09 — DashboardToken por edición](28-09-dashboardtoken-por-edicion.md) | 28-02 |
| 10 | [28-10 — Alta de edición nueva desde el admin](28-10-alta-de-edicion-admin.md) | 28-01 |

**Oportunistas (en paralelo, no bloquean la secuencia principal):**

| # | Sub-issue | Relacionada con |
|---|---|---|
| 11 | [28-11 — Dividir views.py en módulos (opcional)](28-11-opcional-split-views.md) | [[19-arquitectura-views-god-file]] |
| 12 | (fusionada en 28-08) Paginación de listados admin | [[22-perf-sin-paginacion]] |

## Issues antiguas reacomodadas por este diseño

- [[11-smell-fechas-hardcodeadas-reclamo]] — absorbida por `28-03`/`28-04`, no implementar por separado.
- [[14-hardcoding-anio-2026]] — absorbida por el conjunto `28-01`…`28-10`, no implementar por separado.
- [[19-arquitectura-views-god-file]] — sigue siendo válida como issue independiente, pero se ofrece como bundle oportunista en `28-11`.
- [[22-perf-sin-paginacion]] — se resuelve como parte de `28-08` en vez de como fix aislado.
