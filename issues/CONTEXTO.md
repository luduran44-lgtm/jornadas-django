# Contexto para retomar este backlog

Este archivo es para que una instancia nueva de Claude Code (u otro agente) entienda de qué se trata esta carpeta sin tener que releer toda la conversación original. Leer esto primero, antes de tocar cualquier issue.

## Qué es esta carpeta

`issues/` es el backlog de mejoras de una auditoría técnica completa hecha el 2026-07-03 sobre **jornadas-django**: el sistema de inscripciones, asistencia, certificados, encuestas y reclamos de las Jornadas de Formación Profesional (UTN FRLP). Cada archivo es una mejora independiente, pensada para que un agente la tome sin más contexto que el archivo mismo + este documento.

`PLAN.md` es el índice — **siempre empezar por ahí**, no por listar archivos a ciegas.

Esta carpeta se llamó `issus/` originalmente (por un archivo previo del usuario, `issus.md` en la raíz, ya migrado y eliminado) y se renombró a `issues/` después.

## Veredicto de la auditoría

**Continuar sobre este proyecto, no reescribirlo.** La lógica de dominio difícil (cupos, asistencia por QR, elegibilidad de certificados con perdones, reclamos con estados, emisión masiva, encuestas) ya está resuelta y probada en producción para la edición 2026. Los problemas relevados son deuda técnica localizada y corregible de forma incremental, no fallas de arquitectura. El informe visual completo (HTML) se generó como Artifact en la conversación original — si no está a mano, esta carpeta + `PLAN.md` son la fuente de verdad durable, el HTML era solo la vista ejecutiva.

## Orden de trabajo obligatorio

No paralelizar ni saltear este orden sin acuerdo explícito del usuario:

1. **Tier 1** (`01`–`04`): bugs de comportamiento activo. Bajo esfuerzo, alto impacto, no tienen dependencias.
2. **`16`** (tests mínimos): antes de tocar cualquier cosa más grande. Sin esto, el resto del backlog se hace "a ciegas".
3. **Tier 2 + Tier 3 + Seguridad completa** (`05`-`13`, `20`, `21`, `23`, `24`, `25`, etc.): cambios acotados, bajo riesgo. **Esto va antes que la feature de multi-edición** — decisión explícita del usuario, no una sugerencia mía. En particular `20` (`@staff_member_required` consistente) es prerrequisito duro de `28-08`.
4. **`15`** (upgrade de Django): en una ventana sin actividad de jornada en curso.
5. **`28-00` → `28-11`** (modelo de Edición multi-año): recién después de lo anterior. Ver sección siguiente.

## La feature grande: modelo de Edición (multi-año)

Motivación: el sistema asumía una sola edición para siempre (todo hardcodeado a "2026"). El usuario quiere reutilizarlo para 2027 y años siguientes, con un selector público de ediciones pero con el admin pudiendo **forzar cuál es la edición actual**.

- **Diseño ya discutido y aprobado** — no volver a preguntar lo que ya está decidido acá. Spec completo con todas las decisiones y su razonamiento: `docs/superpowers/specs/2026-07-03-multi-edicion-design.md`.
- `28-00-feature-multi-edicion-selector.md` es el **índice** de la feature — lista las sub-issues `28-01`...`28-10` en orden de dependencia, más `28-11` (opcional).
- Cada sub-issue (`28-01`, `28-02`, ...) trae archivos concretos, pasos, snippets de código para las partes no triviales (migraciones de datos con verificación cruzada, `save()` con unicidad, etc.) y criterios de verificación — están escritas para implementarse directamente, no son solo descripción del problema.
- Decisiones clave ya cerradas (para no volver a preguntarlas): `Edition` se identifica por `slug` (no solo año, permite +1 edición por año) · FK directa a `Edition` en cada modelo (no heredada vía `Talk`) · `CertificateConfig` pasa a ser una config por edición · URLs públicas bajo `/e/<slug>/`, la raíz redirige a la edición forzada · links con token/pk (cancelación, ampliación de reclamo, QR) **no** llevan el prefijo de edición · panel admin usa un selector de sesión, no cambia sus URLs · `DashboardToken` queda atado a una edición fija al crearse · edición nueva arranca vacía, sin clonado · `Talk.date` pasa de texto libre a `DateField` real, y se agrega `Talk.time_start` (`TimeField`) para poder calcular el cierre automático de inscripción/cancelación **1 hora después de que empieza la charla** (regla agregada después del diseño inicial, a pedido del usuario — el admin queda exento de esta regla).

## Issues viejas que quedaron reacomodadas por este diseño

No implementar estas por separado — ya están cubiertas dentro de la feature de Edición:

- `11` (fechas hardcodeadas en reclamo) → absorbida por `28-03`/`28-04`.
- `14` (hardcoding del año 2026) → absorbida por `28-01`...`28-10` completo.
- `19` (views.py god file) → sigue siendo válida sola, pero se ofrece como bundle oportunista en `28-11`.
- `22` (sin paginación) → se resuelve dentro de `28-08`, no aparte.

Cada una de estas cuatro tiene una nota en su propio archivo (`estado: absorbido-por-28` en el frontmatter) apuntando a la sub-issue que la reemplaza.

## Convenciones de los archivos

- Frontmatter YAML: `categoria`, `severidad` (`alta`/`media`/`baja`), `esfuerzo` (`bajo`/`medio`/`alto`); las sub-issues de `28-*` suman `fase`, `orden`, `depende_de`.
- `[[nombre-de-archivo]]` es un link interno a otra issue de esta misma carpeta (sin `.md`).
- Confirmar el estado real de git (`git status`/`git log`) antes de asumir qué está commiteado y qué no — no dar por sentado lo que diga este documento a medida que pase el tiempo.
