---
categoria: code-smell
severidad: media
esfuerzo: bajo
estado: absorbido-por-28
---

# Fechas hardcodeadas en `reclamo_nuevo`

> **Absorbida por [[28-04-reclamo-fechas-iso]] y [[28-03-talk-date-datefield-y-time-start]].** No implementar este fix por separado: al convertir `Talk.date` a `DateField` real y `Reclamo.dia`/`dias_perdonados_list` a fechas ISO, el select de día pasa a poblarse dinámicamente como parte de esa migración. Se deja este archivo como registro del problema original.

**Archivo:** `charlas/views.py:1725`, `charlas/forms.py` (choices del select de días)

## Problema

```python
dias = ['Martes 19 de Mayo', 'Miércoles 20 de Mayo', 'Jueves 21 de Mayo']
```

Si se agrega una charla con otra fecha (o si cambia el cronograma de una edición a otra), el formulario de reclamo no la muestra como opción. Esto es un caso puntual del problema más amplio de fechas/año hardcodeados — ver [[14-hardcoding-anio-2026]] y [[28-00-feature-multi-edicion-selector]].

## Fix

Derivar los días dinámicamente desde la base de datos:

```python
dias = list(Talk.objects.values_list('date', flat=True).distinct().order_by('date'))
```

Esto funciona porque `Talk.date` es un CharField que ya contiene el texto legible ("Martes 19 de Mayo"). Aplicar el mismo criterio al `forms.py`.

## Verificación

Agregar una charla con una fecha nueva → el select de "día" en el formulario de reclamo la incluye sin tocar código.
