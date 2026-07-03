---
categoria: arquitectura
severidad: media
esfuerzo: alto
---

# `views.py` de 2082 líneas — god file

**Archivo:** `charlas/views.py` (2082 líneas, mezcla inscripción pública, asistencia/QR, certificados, encuestas, reclamos, tokens de dashboard y varias funciones auxiliares privadas)

## Problema

Un solo archivo concentra siete dominios funcionales distintos. Esto hace difícil navegar el código, aumenta la probabilidad de colisiones de merge cuando dos cambios tocan el mismo archivo por razones no relacionadas, y ya produjo al menos un bug directo ([[01-bug-certificate-emit-duplicada]], una función duplicada que pasó desapercibida por el tamaño del archivo).

## Fix

Dividir en un paquete `charlas/views/` con módulos por dominio, siguiendo los mismos nombres que ya usa `urls.py` para agrupar:
- `views/public.py` — index, inscripción, cancelación
- `views/attendance.py` — QR scan, import/export de asistencia
- `views/certificates.py` — emisión, validación, descarga, config
- `views/surveys.py` — encuesta y sus dashboards
- `views/reclamos.py` — reclamos y su dashboard
- `views/admin_talks.py` — CRUD de charlas
- `views/tokens.py` — tokens de dashboard

Mantener las funciones privadas compartidas (`_evaluar_alumno`, `_duplicate_exists`, etc.) en un `views/_shared.py` o en `services.py`/`selectors.py` si se quiere ir un paso más allá. Este refactor **debe hacerse después de tener tests mínimos** ([[16-testing-sin-cobertura]]) para no romper nada a ciegas.

> **Nota:** se ofrece como bundle oportunista en [[28-11-opcional-split-views]] (opcional, no bloquea la secuencia principal de la feature de multi-edición), porque las issues 28-06/28-07/28-08 ya tocan la mayoría de estas vistas de todos modos. Si no se hace ahí, sigue siendo válida como issue independiente en cualquier momento posterior.

## Verificación

`python manage.py check`, `python manage.py test` en verde, y todas las URLs en `urls.py` siguen resolviendo (`python manage.py show_urls` si está disponible, o smoke test manual de cada ruta).
