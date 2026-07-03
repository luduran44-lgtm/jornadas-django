---
categoria: feature
severidad: alta
esfuerzo: alto
fase: multi-edicion
orden: 3
depende_de: ["28-02"]
---

# 28-03 — `Talk.date` de texto a `DateField` + `Talk.time_start` real

**Spec:** `docs/superpowers/specs/2026-07-03-multi-edicion-design.md` (sección "Talk.date: de texto libre a DateField" y "Talk: horario de inicio estructurado")

## Objetivo

Convertir `Talk.date` (hoy `CharField` tipo `"Martes 19 de Mayo"`) en un `DateField` real, y agregar `Talk.time_start` (`TimeField`) a partir del `time` existente. Esta es la migración de datos más delicada del backlog — requiere verificación cruzada antes de aplicar.

Absorbe y reemplaza [[11-smell-fechas-hardcodeadas-reclamo]] (no aplicar ese fix por separado).

## Archivos

- Modificar: `charlas/models.py` — `Talk.date` → `DateField`, agregar `Talk.time_start`.
- Modificar: `charlas/forms.py` — `TalkForm` pasa a usar un `DateField`/`DateInput` para `date`, y a guardar `time_start` también en el campo real del modelo (ya recolecta `time_start`/`time_end` como inputs de hora).
- Modificar: todos los templates que muestran `talk.date` como texto (`admin_dashboard.html`, `talk.html`, `talk_form.html`, `index.html`, `admin_talk_details.html`, `cronograma_pdf.html`, y cualquier otro que renderice la fecha) — cambiar a `{{ talk.date|date:"l j \\d\\e F" }}`.
- Crear: migración de esquema (campos nuevos nullable) + migración de datos (parseo y verificación) + migración de esquema (campos finales).

## Pasos

1. Agregar los campos nuevos como nullable en `models.py`, dejando el `CharField` viejo temporalmente con otro nombre para no perder el dato durante la migración:

```python
date = models.CharField('Fecha (texto, deprecado)', max_length=100)  # se elimina al final
date_real = models.DateField('Fecha', null=True, blank=True)
time_start = models.TimeField('Hora de inicio', null=True, blank=True)
```

2. `python manage.py makemigrations charlas`.

3. Migración de datos con verificación de día de la semana antes de aplicar. Los nombres de día en español no son los que devuelve `strftime` con locale por defecto, así que se mapea a mano:

```python
import re
from datetime import date as date_cls
from django.db import migrations

DIAS_SEMANA = {0: 'lunes', 1: 'martes', 2: 'miércoles', 3: 'jueves',
               4: 'viernes', 5: 'sábado', 6: 'domingo'}
MESES = {'enero': 1, 'febrero': 2, 'marzo': 3, 'abril': 4, 'mayo': 5, 'junio': 6,
          'julio': 7, 'agosto': 8, 'septiembre': 9, 'octubre': 10,
          'noviembre': 11, 'diciembre': 12}

def parsear_fecha(texto, anio):
    # "Martes 19 de Mayo" -> (19, 5)
    m = re.match(r'(\w+)\s+(\d{1,2})\s+de\s+(\w+)', texto.strip(), re.IGNORECASE)
    if not m:
        raise ValueError(f'No se pudo parsear la fecha: {texto!r}')
    dia_nombre, dia_num, mes_nombre = m.groups()
    mes = MESES.get(mes_nombre.lower())
    if not mes:
        raise ValueError(f'Mes desconocido: {mes_nombre!r} en {texto!r}')
    fecha = date_cls(anio, mes, int(dia_num))
    dia_calculado = DIAS_SEMANA[fecha.weekday()]
    if dia_calculado != dia_nombre.lower():
        raise ValueError(
            f'Día de la semana no coincide para {texto!r}: '
            f'texto dice {dia_nombre!r}, calculado {dia_calculado!r} para {fecha}'
        )
    return fecha

def parsear_hora_inicio(texto_time):
    # "18:00 a 20:00" o "18 a 20" -> time(18, 0)
    from datetime import time as time_cls
    m = re.match(r'(\d{1,2})(?::(\d{2}))?', texto_time.strip())
    if not m:
        raise ValueError(f'No se pudo parsear la hora: {texto_time!r}')
    hora, minuto = m.groups()
    return time_cls(int(hora), int(minuto or 0))

def migrar_fechas(apps, schema_editor):
    Talk = apps.get_model('charlas', 'Talk')
    errores = []
    for talk in Talk.objects.select_related('edition').all():
        try:
            talk.date_real = parsear_fecha(talk.date, talk.edition.anio)
            talk.time_start = parsear_hora_inicio(talk.time)
            talk.save(update_fields=['date_real', 'time_start'])
        except ValueError as e:
            errores.append(f'Talk id={talk.id} título={talk.title!r}: {e}')
    if errores:
        raise RuntimeError(
            'No se pudieron migrar las siguientes charlas, corregir manualmente '
            'antes de reintentar:\n' + '\n'.join(errores)
        )

def noop_reverse(apps, schema_editor):
    pass

class Migration(migrations.Migration):
    dependencies = [
        ('charlas', 'XXXX_talk_date_real_nullable'),
    ]
    operations = [
        migrations.RunPython(migrar_fechas, noop_reverse),
    ]
```

   Si la migración falla con `RuntimeError`, **no continuar** hasta corregir el dato de origen (`Talk.date`/`Talk.time` de la fila reportada) y volver a correrla.

4. Una vez migrados todos los datos sin errores: eliminar el `CharField date` viejo, renombrar `date_real` a `date` (`python manage.py makemigrations` detecta el rename si se hace en un solo paso con `RenameField` — si Django no lo detecta automáticamente, usar `migrations.RenameField` explícito en la migración generada), y volver `date`/`time_start` `null=False`.

5. Actualizar `TalkForm` en `charlas/forms.py`: cambiar el widget de `date` de `Select` con choices hardcodeadas a un `DateInput` (`type='date'`), y en `save()` asegurarse de que `time_start` se guarde como `TimeField` real además de armar el string `time` para mostrar.

6. Actualizar templates: reemplazar cualquier `{{ talk.date }}` crudo por `{{ talk.date|date:"l j \\d\\e F" }}` (con `LANGUAGE_CODE = es-ar` ya configurado en `settings.py`, esto produce "martes 19 de mayo" — ajustar capitalización en CSS/template si hace falta el estilo "Martes" con mayúscula).

## Verificación

1. La migración de datos corre sin `RuntimeError` sobre el dataset real de 2026.
2. `Talk.objects.filter(date__isnull=True).count() == 0` y lo mismo para `time_start`.
3. Cargar `index.html`, `admin_dashboard.html` y `cronograma_pdf.html` — las fechas se siguen viendo correctas ("martes 19 de mayo") sin el texto hardcodeado del modelo.
4. Crear una charla nueva desde `admin_new_talk` usando el nuevo `DateInput` y confirmar que se guarda correctamente.
