---
categoria: code-smell
severidad: media
esfuerzo: bajo
---

# `print()` en vez de `logging`

**Archivos:** `charlas/views.py` (líneas 66, 88, 305, 355, 1066, 1864, 1893, 1926, etc.), `charlas/signals.py` (líneas 33, 35)

## Problema

Todos los `print(f'[CERT]...')` / `print(f'[SIGNAL]...')` son invisibles en producción — bajo WSGI, stdout no va a ningún log persistente. Cuando algo falla en emisión de certificados o en la limpieza automática de certificados no elegibles, no queda ningún rastro.

## Fix

Al inicio de `views.py` y `signals.py` agregar:

```python
import logging
logger = logging.getLogger(__name__)
```

Reemplazar todos los `print(f'[X] ...')` por `logger.info(...)` / `logger.warning(...)` / `logger.error(...)` según corresponda, y agregar un handler de logging a archivo (o al servicio de logging del hosting) en `settings.py` para el logger `charlas` — hoy `LOGGING` solo configura el logger `send_reminders`.

## Verificación

`grep -n "print(" charlas/views.py charlas/signals.py` no debe devolver líneas de debug de negocio (dejar solo prints de management commands si corresponde). Forzar un error en emisión de certificado y verificar que aparece en el log configurado.
