---
categoria: bug
severidad: media
esfuerzo: bajo
---

# `_evaluar_alumno` en modo `por_dia` ignora días perdonados si ya existe asistencia real

**Archivo:** `charlas/views.py:387–396`

## Problema

```python
for dia in dias_perdonados:
    if dia not in dias:           # ← solo agrega si NO existe
        dias[dia] = config.minimo
```

Si el alumno tiene 1 asistencia real en un día y ese mismo día también está perdonado, el perdón no suma al count — se pierde. En modo `total` sí suma correctamente. Es una inconsistencia de comportamiento entre las dos modalidades de `CertificateConfig`.

## Fix

En modo `por_dia`, para los días perdonados que ya existen en el dict, incrementar el count en lugar de ignorarlo:

```python
for dia in dias_perdonados:
    dias[dia] = dias.get(dia, 0) + config.minimo
```

## Verificación

`_evaluar_alumno` con modo `por_dia`, un día con 1 asistencia real + ese mismo día perdonado → el alumno debe calificar si la suma alcanza el mínimo.
