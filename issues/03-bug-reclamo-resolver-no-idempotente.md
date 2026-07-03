---
categoria: bug
severidad: alta
esfuerzo: bajo
---

# `reclamo_resolver` no es idempotente

**Archivo:** `charlas/views.py:1931–1946`

## Problema

Si un admin hace clic en "aprobar" dos veces (doble submit, doble click, o refresh accidental de un POST), `_aplicar_resolucion` corre dos veces: puede crear dos certificados, marcar dos veces la asistencia, o duplicar efectos secundarios del reclamo.

## Fix

Al inicio del bloque `if accion == 'aprobar':`, agregar un guard:

```python
if reclamo.estado == 'aprobado':
    return redirect('reclamo_detalle', pk=pk)
```

## Verificación

Aprobar el mismo reclamo dos veces seguidas (doble POST) → la segunda vez redirige sin crear un certificado/efecto duplicado.
