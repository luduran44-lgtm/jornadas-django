---
categoria: modelo-de-datos
severidad: media
esfuerzo: bajo
---

# Falta `db_index` en campos muy filtrados

**Archivo:** `charlas/models.py` (`Registration.dni`, `Registration.legajo`, `Reclamo.dni`, `Certificate.dni`)

## Problema

Estos campos se usan en `.filter(dni=...)` / `.filter(legajo=...)` en decenas de lugares de `views.py` (validación de certificados, dashboards, reclamos, encuestas). Sin índice, cada una de esas queries hace un table scan completo, y el problema empeora a medida que se acumulan inscripciones de múltiples ediciones/años (ver [[28-00-feature-multi-edicion-selector]]).

## Fix

Agregar `db_index=True` a `Registration.dni`, `Registration.legajo`, `Reclamo.dni` y `Certificate.dni`, y generar la migración correspondiente.

## Verificación

`python manage.py makemigrations` genera una migración de tipo `AlterField` para los cuatro campos; `EXPLAIN QUERY PLAN` (sqlite) o `EXPLAIN ANALYZE` (postgres) sobre un `filter(dni=...)` muestra uso de índice en vez de scan.
