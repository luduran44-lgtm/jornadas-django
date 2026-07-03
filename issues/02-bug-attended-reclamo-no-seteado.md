---
categoria: bug
severidad: alta
esfuerzo: bajo
---

# `attended_reclamo` no se setea cuando `tipo='asistencia'`

**Archivo:** `charlas/views.py:1835–1842`

## Problema

Cuando se aprueba un reclamo con `tipo='asistencia'`, el código pone `reg.attended = True` pero **no** `reg.attended_reclamo = True`. La rama de `resolucion in ('charla', 'dia')` sí lo setea (línea 1872). Los templates usan `attended_reclamo` para mostrar el badge "Presente por reclamo", así que queda una inconsistencia silenciosa: el alumno aparece como presente pero sin la marca de que fue por reclamo.

## Fix

En el bloque `if reclamo.tipo == 'asistencia':`, agregar `reg.attended_reclamo = True` junto con `reg.attended = True`.

## Verificación

Crear un reclamo con `tipo=asistencia`, aprobarlo, y verificar que `Registration.attended_reclamo == True` para el DNI correspondiente.
