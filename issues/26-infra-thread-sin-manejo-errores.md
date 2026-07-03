---
categoria: infraestructura
severidad: media
esfuerzo: medio
---

# `threading.Thread` sin manejo de errores para limpieza de certificados

**Archivo:** `charlas/signals.py:38-43`

## Problema

Al guardar una `CertificateConfig` activa, se dispara un `threading.Thread` daemon que corre `_limpiar_certificados_no_elegibles` fuera del ciclo request/response. Si esa función lanza una excepción, no hay ningún manejo — el thread muere silenciosamente y lo único que queda es lo que haya hecho `print()` antes de la falla (ver también [[09-smell-print-vs-logging]]). Además, bajo un servidor WSGI con múltiples workers/procesos, un `Thread` de Python vive y muere con el proceso worker que lo creó; si ese worker se recicla (deploy, restart, timeout) antes de que el thread termine, la limpieza queda a medio hacer sin ningún registro de que pasó.

## Fix

Corto plazo: envolver `_limpiar_certificados_no_elegibles` en un `try/except` que loguee cualquier excepción con `logger.exception(...)` antes de que el thread termine.

Medio/largo plazo: si el volumen de certificados a limpiar crece (multi-edición, ver [[28-00-feature-multi-edicion-selector]]), evaluar reemplazar el `Thread` ad-hoc por una cola de tareas real (`django-rq`, Celery, o incluso un management command encolado) que sí tenga reintentos y visibilidad de fallos — el mismo patrón que ya usa `EmissionJob` para emisión de certificados, pero aplicado a esta limpieza.

## Verificación

Forzar una excepción dentro de `_limpiar_certificados_no_elegibles` (ej. mockear `_evaluar_alumno` para que falle) → debe quedar un log de error visible, no un fallo silencioso.
