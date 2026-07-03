---
categoria: modelo-de-datos
severidad: baja
esfuerzo: bajo
---

# `Reclamo.charlas_perdonadas` (M2M) es campo muerto

**Archivo:** `charlas/models.py:353–354`

## Problema

El M2M `charlas_perdonadas` nunca se popula en ninguna parte del código. `_evaluar_alumno` usa el `PositiveSmallIntegerField charlas_perdonadas_count` en su lugar. El M2M es confusión pura para cualquiera que lea el modelo.

## Fix

Crear una migración para eliminar el M2M `charlas_perdonadas`. Mantener solo `charlas_perdonadas_count`.

## Verificación

`python manage.py makemigrations && python manage.py migrate` sin errores; el admin de `Reclamo` ya no debe mostrar el widget M2M.
