---
categoria: feature
severidad: media
esfuerzo: medio
fase: multi-edicion
orden: 4
depende_de: ["28-03"]
---

# 28-04 — `Reclamo.dia` y `dias_perdonados_list` a fechas ISO

**Spec:** `docs/superpowers/specs/2026-07-03-multi-edicion-design.md` (sección "Talk.date: de texto libre a DateField")

## Objetivo

Migrar `Reclamo.dia` (hoy texto libre) y `Reclamo.dias_perdonados_list` (JSONField con lista de strings de texto) para que usen el mismo formato de fecha ISO que `Talk.date` después de `28-03`, reutilizando el mapeo texto→fecha ya validado en esa migración.

## Archivos

- Modificar: `charlas/models.py` — `Reclamo.dia` pasa de `CharField` a `DateField` (`null=True, blank=True`, porque no todo reclamo tiene un día asociado). `dias_perdonados_list` sigue siendo `JSONField` pero ahora almacena strings ISO (`"2026-05-19"`) en vez de texto libre.
- Modificar: `charlas/views.py` — cualquier lugar que compare `reclamo.dia` o elementos de `dias_perdonados_list` contra `talk.date` como string (buscar con `grep -n "dias_perdonados_list\|reclamo.dia\|\.dia ==" charlas/views.py`) debe comparar contra el nuevo `Talk.date` (objeto `date`) o su `.isoformat()`.
- Modificar: `charlas/forms.py` y el template `reclamo_nuevo.html` — el select de "día" ahora se puebla con `Talk.objects.values_list('date', flat=True).distinct().order_by('date')` (fechas reales, no texto), mostrando el label formateado pero enviando el valor ISO.
- Crear: migración de datos que reescriba `dia` y `dias_perdonados_list` de cada `Reclamo` existente usando la misma función `parsear_fecha` de `28-03` (o su resultado ya aplicado: cruzar por texto contra `Talk.date` ya migrado, en vez de volver a parsear).

## Pasos

1. Agregar campo temporal `dia_real = models.DateField(null=True, blank=True)` en `Reclamo`, migrar.
2. Migración de datos: para cada `Reclamo`, si `dia` no está vacío, buscar el `Talk` de esa edición cuyo `date` coincida con el texto (cruzando por el texto formateado, ya que después de `28-03` `Talk.date` es un `DateField` real):

```python
def migrar_dia_reclamo(apps, schema_editor):
    Reclamo = apps.get_model('charlas', 'Reclamo')
    Talk = apps.get_model('charlas', 'Talk')
    for reclamo in Reclamo.objects.exclude(dia='').select_related('edition'):
        # Construir el mapa texto-formateado -> fecha real para la edición de este reclamo
        talks = Talk.objects.filter(edition=reclamo.edition)
        mapa = {}
        for t in talks:
            # Debe generarse con el mismo formato que se usaba antes de 28-03
            # (ver el texto original conservado en fixtures/backup si hace falta)
            mapa[t.date.isoformat()] = t.date
        # Si el texto de reclamo.dia coincide con alguna fecha ya migrada, asignar
        # (ajustar el matching según el formato real encontrado en los datos)
        for fecha_iso, fecha in mapa.items():
            if reclamo.dia.strip().lower() in fecha_iso:  # placeholder de matching, ajustar con datos reales
                reclamo.dia_real = fecha
                break
        reclamo.save(update_fields=['dia_real'])
```

   **Nota:** el matching exacto depende de qué formato de texto tenía `Reclamo.dia` en los datos reales (puede diferir levemente del de `Talk.date` original). Antes de escribir la versión final de esta migración, correr `Reclamo.objects.exclude(dia='').values_list('dia', flat=True).distinct()` en producción/staging para confirmar los valores reales y ajustar el mapeo exacto.

3. Migrar `dias_perdonados_list` con la misma lógica, elemento por elemento de la lista, guardando `date.isoformat()` en vez del texto.
4. Eliminar `dia` (texto) y renombrar `dia_real` a `dia`; volver el campo su tipo final.
5. Actualizar todas las comparaciones en `views.py` (`_evaluar_alumno`, `reclamo_nuevo`, `reclamo_resolver`) que hoy comparan strings de día, para que comparen objetos `date`.

## Verificación

1. `python manage.py migrate` sin errores ni reclamos con `dia_real` no resuelto cuando `dia` original no estaba vacío (loguear cualquier caso no resuelto para revisión manual).
2. Flujo completo de reclamo: crear un reclamo nuevo eligiendo un día del select → se guarda como fecha ISO, no texto.
3. `_evaluar_alumno` con perdones (modo `por_dia`) sigue funcionando igual que antes de la migración (correr los tests de [[16-testing-sin-cobertura]] si ya existen para este momento del backlog).
