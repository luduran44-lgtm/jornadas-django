---
categoria: feature
severidad: alta
esfuerzo: medio
fase: multi-edicion
orden: 1
depende_de: []
---

# 28-01 — Modelo `Edition` y edición inicial `jfp-2026`

**Spec:** `docs/superpowers/specs/2026-07-03-multi-edicion-design.md` (sección "Nuevo modelo Edition")

## Objetivo

Crear el modelo `Edition` y una migración de datos que instancie la edición inicial (`jfp-2026`) marcada como actual. Esta issue no toca ningún otro modelo todavía — es la base sobre la que se apoya `28-02`.

## Archivos

- Modificar: `charlas/models.py` — agregar el modelo `Edition`.
- Modificar: `charlas/admin.py` — registrar `Edition` en el admin.
- Crear: `charlas/migrations/00XX_edition.py` (schema).
- Crear: `charlas/migrations/00XX_edition_seed_inicial.py` (data migration).

## Pasos

1. Agregar el modelo en `charlas/models.py`:

```python
class Edition(models.Model):
    slug = models.SlugField('Slug', max_length=50, unique=True)
    nombre = models.CharField('Nombre', max_length=100)
    anio = models.PositiveIntegerField('Año')
    fecha_inicio = models.DateField('Fecha de inicio')
    fecha_fin = models.DateField('Fecha de fin')
    es_actual = models.BooleanField('Edición actual (forzada)', default=False)
    created_at = models.DateTimeField(auto_now_add=True)

    class Meta:
        verbose_name = 'Edición'
        verbose_name_plural = 'Ediciones'
        ordering = ['-anio', '-fecha_inicio']

    def __str__(self):
        return self.nombre

    def save(self, *args, **kwargs):
        if self.es_actual:
            Edition.objects.exclude(pk=self.pk).update(es_actual=False)
        super().save(*args, **kwargs)
```

   El `save()` garantiza que solo una edición tenga `es_actual=True` a la vez, mismo patrón que ya usa `CertificateConfig.activa` en `signals.py`.

2. Registrar en `charlas/admin.py`:

```python
from .models import Edition

@admin.register(Edition)
class EditionAdmin(admin.ModelAdmin):
    list_display = ('nombre', 'slug', 'anio', 'fecha_inicio', 'fecha_fin', 'es_actual')
    list_filter = ('anio', 'es_actual')
    prepopulated_fields = {'slug': ('nombre',)}
    ordering = ('-anio',)
```

3. Generar la migración de esquema: `python manage.py makemigrations charlas`.

4. Crear una migración de datos (`python manage.py makemigrations charlas --empty --name edition_seed_inicial`) que instancie la edición inicial. Ajustar `fecha_inicio`/`fecha_fin` a las fechas reales del cronograma 2026 (los tres días que hoy aparecen hardcodeados en `charlas/views.py:1725`: 19, 20 y 21 de mayo):

```python
from django.db import migrations

def crear_edicion_inicial(apps, schema_editor):
    Edition = apps.get_model('charlas', 'Edition')
    Edition.objects.create(
        slug='jfp-2026',
        nombre='JFP 2026',
        anio=2026,
        fecha_inicio='2026-05-19',
        fecha_fin='2026-05-21',
        es_actual=True,
    )

def eliminar_edicion_inicial(apps, schema_editor):
    Edition = apps.get_model('charlas', 'Edition')
    Edition.objects.filter(slug='jfp-2026').delete()

class Migration(migrations.Migration):
    dependencies = [
        ('charlas', 'XXXX_edition'),  # ajustar al nombre real de la migración de esquema
    ]
    operations = [
        migrations.RunPython(crear_edicion_inicial, eliminar_edicion_inicial),
    ]
```

## Verificación

- `python manage.py migrate` aplica ambas migraciones sin errores.
- `Edition.objects.get(slug='jfp-2026').es_actual` es `True`.
- Crear una segunda `Edition` de prueba con `es_actual=True` vía shell (`python manage.py shell`) y confirmar que la primera pasa a `es_actual=False` automáticamente.
- El admin de Django muestra la sección "Ediciones" con `jfp-2026` cargada.
