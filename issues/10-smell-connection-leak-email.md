---
categoria: code-smell
severidad: media
esfuerzo: bajo
---

# `_send_certificate_email` puede dejar la conexión SMTP abierta

**Archivo:** `charlas/views.py:344–352`

## Problema

```python
connection.open()
connection.connection.sendmail(...)  # si explota aquí
connection.close()                   # ← nunca se ejecuta
```

Si `sendmail` lanza una excepción (timeout, credenciales inválidas, destinatario rechazado), la conexión SMTP queda abierta indefinidamente. En un job de emisión masiva con cientos de envíos, esto puede agotar conexiones disponibles en el servidor SMTP (Office 365) y afectar envíos posteriores.

## Fix

Usar `try/finally` o, preferentemente, el context manager `with get_connection() as connection:` que Django ya provee y que cierra la conexión automáticamente incluso ante excepción.

## Verificación

Simular un fallo de `sendmail` (mock o credencial inválida) y confirmar que la conexión se cierra igual (no quedan conexiones colgadas en el pool del servidor SMTP).
