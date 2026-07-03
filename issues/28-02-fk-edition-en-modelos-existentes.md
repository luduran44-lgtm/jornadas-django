---
categoria: feature
severidad: alta
esfuerzo: alto
fase: multi-edicion
orden: 2
depende_de: ["28-01"]
---

# 28-02 — FK `edition` en `Talk`, `Registration`, `Certificate`, `Reclamo`, `CertificateConfig`, `DashboardToken`

**Spec:** `docs/superpowers/specs/2026-07-03-multi-edicion-design.md` (sección "FKs nuevas")

## Objetivo

Agregar `edition = models.ForeignKey(Edition, on_delete=models.PROTECT)` a los seis modelos, con backfill de todos los registros existentes a `jfp-2026` (creada en `28-01`) antes de volver el campo obligatorio.

## Archivos

- Modificar: `charlas/models.py` — agregar el campo a `Talk`, `Registration`, `Certificate`, `Reclamo`, `CertificateConfig`, `DashboardToken`.
- Modificar: `charlas/admin.py` — agregar `edition` a `list_filter` en `TalkAdmin` y `RegistrationAdmin`.
- Crear: migración de esquema (FK nullable), migración de datos (backfill), migración de esquema (FK `NOT NULL`).

## Pasos

1. En cada uno de los seis modelos, agregar el campo como `null=True` primero (no se puede agregar un FK obligatorio a una tabla con filas existentes sin un valor por defecto):

```python
edition = models.ForeignKey('Edition', on_delete=models.PROTECT, null=True, blank=True, verbose_name='Edición')
```

2. `python manage.py makemigrations charlas` → genera la migración de esquema con el campo nullable.

3. Crear una migración de datos (`--empty --name backfill_edition`) que asigne `jfp-2026` a todo lo existente:

```python
from django.db import migrations

def backfill_edition(apps, schema_editor):
    Edition = apps.get_model('charlas', 'Edition')
    edicion_2026 = Edition.objects.get(slug='jfp-2026')
    for model_name in ('Talk', 'Registration', 'Certificate', 'Reclamo', 'CertificateConfig', 'DashboardToken'):
        Model = apps.get_model('charlas', model_name)
        Model.objects.filter(edition__isnull=True).update(edition=edicion_2026)

def noop_reverse(apps, schema_editor):
    pass

class Migration(migrations.Migration):
    dependencies = [
        ('charlas', 'XXXX_fk_edition_nullable'),  # ajustar al nombre real
    ]
    operations = [
        migrations.RunPython(backfill_edition, noop_reverse),
    ]
```

4. Cambiar los seis campos a `null=False, blank=False` en `models.py` y correr `makemigrations` de nuevo → genera la migración de esquema final que vuelve el FK obligatorio. Django va a preguntar por un default para filas existentes: como ya no debería haber ninguna con `edition=NULL` después del paso 3, se puede confirmar sin default (o usar `--fake` si hiciera falta, pero no debería).

5. Actualizar `TalkAdmin.list_filter` y `RegistrationAdmin.list_filter` en `charlas/admin.py` agregando `'edition'` a la tupla existente.

## Verificación

1. `python manage.py migrate` aplica las tres migraciones en orden sin error.
2. `Talk.objects.filter(edition__isnull=True).count() == 0` y lo mismo para los otros cinco modelos.
3. `python manage.py check` sin warnings.
4. El admin de `Talk` y `Registration` permite filtrar por edición.
