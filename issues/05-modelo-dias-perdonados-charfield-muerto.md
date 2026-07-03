---
categoria: modelo-de-datos
severidad: baja
esfuerzo: bajo
---

# `Reclamo.dias_perdonados` (CharField) es campo muerto

**Archivo:** `charlas/models.py:355–356`

## Problema

El `JSONField dias_perdonados_list` (línea 335) es el que efectivamente se lee y escribe en todo el codebase. El `CharField dias_perdonados` no se usa en ninguna view ni template — es un remanente de una implementación anterior.

## Fix

Crear una migración para eliminar el campo `dias_perdonados` (CharField) del modelo `Reclamo`.

## Verificación

`python manage.py makemigrations && python manage.py migrate` sin errores; `grep -rn "\.dias_perdonados\b" charlas/` no debe encontrar más referencias que al `dias_perdonados_list`.
