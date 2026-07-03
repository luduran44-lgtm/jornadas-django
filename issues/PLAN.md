# Plan de mejoras — jornadas-django

> **¿Instancia nueva / sin contexto previo?** Leer primero [`CONTEXTO.md`](CONTEXTO.md) en esta misma carpeta.

Auditoría completa realizada el 2026-07-03 sobre el sistema de Jornadas de Formación Profesional (inscripciones, asistencia, certificados, encuestas y reclamos). Cada mejora vive en su propio archivo en esta carpeta para que un agente la pueda tomar de forma independiente. Este archivo es solo el índice priorizado.

Ver también el informe de auditoría en HTML (compartido aparte) para el panorama ejecutivo y la recomendación de continuar vs. reescribir el proyecto.

## Cómo usar este backlog

- Cada issue tiene frontmatter con `categoria`, `severidad` (`alta`/`media`/`baja`) y `esfuerzo` (`bajo`/`medio`/`alto`).
- Los links `[[nombre-archivo]]` conectan issues relacionadas.
- El orden sugerido de ejecución es: **Tier 1 → Tier 2 → Tier 3 → Seguridad → Testing/CI → Arquitectura → Feature multi-edición**. Las issues de Tier 1 son las únicas que corrigen comportamiento activamente incorrecto hoy; el resto es prevención y mantenibilidad.
- **28-00-feature-multi-edicion-selector** es el índice de la feature grande — el diseño ya está aprobado (`docs/superpowers/specs/2026-07-03-multi-edicion-design.md`) y partido en sub-issues `28-01`…`28-11` para implementar en orden.

## Tier 1 — Bugs con comportamiento incorrecto (hacer primero)

| # | Issue | Severidad |
|---|---|---|
| 01 | [certificate_emit duplicada](01-bug-certificate-emit-duplicada.md) | Alta |
| 02 | [attended_reclamo no se setea](02-bug-attended-reclamo-no-seteado.md) | Alta |
| 03 | [reclamo_resolver no idempotente](03-bug-reclamo-resolver-no-idempotente.md) | Alta |
| 04 | [_evaluar_alumno por_dia ignora perdones](04-bug-evaluar-alumno-por-dia-perdones.md) | Media |

## Tier 2 — Modelo de datos

| # | Issue | Severidad |
|---|---|---|
| 05 | [dias_perdonados CharField muerto](05-modelo-dias-perdonados-charfield-muerto.md) | Baja |
| 06 | [charlas_perdonadas M2M muerto](06-modelo-charlas-perdonadas-m2m-muerto.md) | Baja |
| 07 | [Falta db_index en dni/legajo](07-modelo-falta-db-index.md) | Media |
| 08 | [Falta unique_together Registration](08-modelo-falta-unique-together-registration.md) | Alta |

## Tier 3 — Code smells y mantenibilidad

| # | Issue | Severidad |
|---|---|---|
| 09 | [print() en vez de logging](09-smell-print-vs-logging.md) | Media |
| 10 | [Connection leak en email](10-smell-connection-leak-email.md) | Media |
| 11 | [Fechas hardcodeadas en reclamo](11-smell-fechas-hardcodeadas-reclamo.md) | Media |
| 13 | [N+1 saves en EmissionJob](13-perf-n1-saves-emissionjob.md) | Media |
| 25 | [Código muerto comentado](25-smell-codigo-muerto-comentado.md) | Baja |

## Seguridad

| # | Issue | Severidad |
|---|---|---|
| 12 | [Validación motivo→tipo server-side](12-seguridad-validacion-motivo-tipo-server-side.md) | Media |
| 20 | [Permisos: login_required vs staff_member_required](20-seguridad-permisos-login-required-vs-staff.md) | Media |
| 21 | [Sin rate-limiting en endpoints públicos](21-seguridad-sin-rate-limiting.md) | Media |
| 23 | [Validación de archivos subidos](23-seguridad-validacion-archivos-subidos.md) | Media |
| 24 | [Borrado de Talk sin confirmación](24-seguridad-borrado-sin-confirmacion.md) | Alta |

## Rendimiento y escalabilidad

| # | Issue | Severidad |
|---|---|---|
| 22 | [Sin paginación en dashboards](22-perf-sin-paginacion.md) | Media |

## Testing y CI/CD

| # | Issue | Severidad |
|---|---|---|
| 16 | [Cobertura de tests inexistente](16-testing-sin-cobertura.md) | Alta |
| 17 | [Sin CI/CD](17-infra-sin-ci-cd.md) | Media |

## Infraestructura y operación

| # | Issue | Severidad |
|---|---|---|
| 15 | [Django LTS por vencer/vencido](15-infra-django-lts-eol.md) | Alta |
| 26 | [Thread sin manejo de errores](26-infra-thread-sin-manejo-errores.md) | Media |
| 27 | [Media sin backup](27-infra-media-sin-backup.md) | Media |
| 18 | [Sin README](18-docs-sin-readme.md) | Baja |

## Arquitectura

| # | Issue | Severidad |
|---|---|---|
| 19 | [views.py god file](19-arquitectura-views-god-file.md) | Media |

## Mantenibilidad / preparación multi-edición

| # | Issue | Severidad | Estado |
|---|---|---|---|
| 14 | [Hardcoding del año 2026](14-hardcoding-anio-2026.md) | Alta | Absorbida por 28-01…28-10 |

## Feature grande — diseño aprobado, implementación partida en sub-issues

Spec completo: `docs/superpowers/specs/2026-07-03-multi-edicion-design.md`.

| # | Issue | Severidad | Depende de |
|---|---|---|---|
| 28 | [Índice — Modelo de Edición + selector](28-00-feature-multi-edicion-selector.md) | Alta | Toda la sección **Seguridad** de este documento (en particular 20) |
| 28-01 | [Modelo Edition y edición inicial](28-01-modelo-edition-y-migracion-inicial.md) | Alta | — |
| 28-02 | [FK Edition en modelos existentes](28-02-fk-edition-en-modelos-existentes.md) | Alta | 28-01 |
| 28-03 | [Talk.date → DateField + time_start](28-03-talk-date-datefield-y-time-start.md) | Alta | 28-02 |
| 28-04 | [Reclamo: fechas a ISO](28-04-reclamo-fechas-iso.md) | Media | 28-03 |
| 28-05 | [CertificateConfig por edición](28-05-certificateconfig-por-edicion.md) | Media | 28-02 |
| 28-06 | [Cierre automático de inscripción/cancelación (+1h)](28-06-cierre-automatico-inscripcion.md) | Alta | 28-03 |
| 28-07 | [URL /e/&lt;slug&gt;/ y selector público](28-07-url-scheme-selector-publico.md) | Alta | 28-01, 28-02 |
| 28-08 | [Selector de sesión admin + paginación](28-08-selector-admin-y-paginacion.md) | Media | 28-02, **20** |
| 28-09 | [DashboardToken por edición](28-09-dashboardtoken-por-edicion.md) | Baja | 28-02 |
| 28-10 | [Alta de edición nueva desde el admin](28-10-alta-de-edicion-admin.md) | Media | 28-01 |
| 28-11 | [(opcional) Dividir views.py](28-11-opcional-split-views.md) | Baja | — (bundle con 19) |

---

## Recomendación general

Continuar sobre este proyecto (no reescribir de cero) — el detalle y las razones están en el informe de auditoría HTML. Orden sugerido de trabajo real:

1. **Tier 1** (bugs activos) — bajo esfuerzo, alto impacto, hacerlo ya.
2. **16** (tests mínimos) antes de tocar nada más grande — da red de seguridad para todo lo que sigue.
3. **Tier 2 + Tier 3 + Seguridad completa** (12, 20, 21, 23, 24) — la mayoría son cambios acotados y de bajo riesgo con tests en verde. Esto va **antes** de la feature de multi-edición: en particular 20 (`@staff_member_required` consistente) es prerrequisito directo de 28-08, que agrega superficie nueva al panel admin.
4. **15** (upgrade Django) en una ventana sin actividad de jornada.
5. **28-01 → 28-10 en orden** (ver tabla arriba) — modelo de Edición completo. 14 y 11 quedan absorbidas acá, no se implementan por separado. 28-11 (split de `views.py`, bundle con 19) y la paginación de 22 (bundle con 28-08) se hacen oportunistamente en el mismo tramo de trabajo, no antes.
