---
categoria: seguridad
severidad: media
esfuerzo: bajo
---

# Falta validar server-side que `motivo` → `tipo` sean coherentes

**Archivo:** `charlas/views.py:1727–1770`

## Problema

El campo `tipo` ('asistencia' / 'justificacion') lo determina JavaScript en el cliente en base al `motivo` elegido. No hay validación equivalente en el servidor. Un usuario puede manipular el POST y enviar `tipo='asistencia'` con `motivo='trabajo'`, y el reclamo se guarda sin error, generando datos inconsistentes que después confunden al admin al resolver.

## Fix

Agregar la misma lógica de mapeo en la view antes del `Reclamo.objects.create`, ignorando el valor que mande el cliente:

```python
MOTIVO_TIPO_MAP = {
    'no_registrado': 'asistencia',
    'superposicion': 'justificacion',
    'trabajo': 'justificacion',
    'no_cursa': 'justificacion',
}
tipo = MOTIVO_TIPO_MAP.get(motivo, tipo)  # override del valor del cliente
```

## Verificación

Enviar un POST manual (curl/Postman) con `motivo='trabajo'` y `tipo='asistencia'` → el reclamo creado debe quedar con `tipo='justificacion'`, no con lo que mandó el cliente.
