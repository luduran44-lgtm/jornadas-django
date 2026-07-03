---
categoria: rendimiento
severidad: media
esfuerzo: bajo
---

# N+1 saves de `EmissionJob` en `_run_emission`

**Archivo:** `charlas/views.py:295`

## Problema

`job.save()` se llama una vez por cada certificado emitido dentro del loop. Con 500 alumnos elegibles, eso son 500 writes a la base de datos solo para actualizar contadores de progreso — innecesario y ralentiza la emisión masiva.

## Fix

Usar `update_fields` con expresiones `F()` para acumular los contadores sin necesidad de traer y guardar el objeto completo en cada iteración:

```python
EmissionJob.objects.filter(pk=job.id).update(
    enviados=F('enviados') + (1 if ok else 0),
    errores=F('errores') + (0 if ok else 1),
)
```

Guardar el objeto completo (`job.save()`) solo una vez al final, para cambiar el `status`.

## Verificación

Emitir certificados para un lote de prueba (50+ alumnos) y confirmar, con el query log de Django (`django.db.backends` en `DEBUG`), que la cantidad de `UPDATE` sobre `EmissionJob` baja de N a ~N (updates atómicos, pero sin el overhead de un `SELECT` completo por vuelta) o, mejor, medir el tiempo total de la emisión antes/después.
