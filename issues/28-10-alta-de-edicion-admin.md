---
categoria: feature
severidad: media
esfuerzo: bajo
fase: multi-edicion
orden: 10
depende_de: ["28-01"]
---

# 28-10 — Alta de edición nueva desde el admin

**Spec:** `docs/superpowers/specs/2026-07-03-multi-edicion-design.md` (sección "Alta de una edición nueva")

## Objetivo

Permitir crear una edición nueva (ej. `jfp-2027`) y forzarla como actual desde el panel admin, sin pasar por el Django admin genérico (`/admin/charlas/edition/add/` ya funciona desde `28-01`, pero conviene un flujo dedicado dentro del panel propio del proyecto, consistente con el resto de `/admin/...`). La edición nueva arranca **vacía**, sin charlas — no hay clonado (fuera de alcance, ver spec).

## Archivos

- Crear: `charlas/forms.py` — `EditionForm` (ModelForm simple sobre `Edition`).
- Modificar: `charlas/views.py` — vistas `admin_ediciones` (listado), `admin_nueva_edicion` (alta), `admin_forzar_edicion` (marcar `es_actual`).
- Crear: template `charlas/templates/charlas/admin_ediciones.html`.
- Modificar: `charlas/urls.py` — rutas nuevas.
- Modificar: `charlas/templates/charlas/base_admin.html` — link al listado de ediciones en el menú.

## Pasos

1. `EditionForm` en `charlas/forms.py`:

```python
class EditionForm(forms.ModelForm):
    class Meta:
        model = Edition
        fields = ['nombre', 'slug', 'anio', 'fecha_inicio', 'fecha_fin']
        widgets = {
            'nombre': forms.TextInput(attrs={'class': 'form-control'}),
            'slug': forms.TextInput(attrs={'class': 'form-control'}),
            'anio': forms.NumberInput(attrs={'class': 'form-control'}),
            'fecha_inicio': forms.DateInput(attrs={'class': 'form-control', 'type': 'date'}),
            'fecha_fin': forms.DateInput(attrs={'class': 'form-control', 'type': 'date'}),
        }
```

2. Vistas en `charlas/views.py`:

```python
@staff_member_required
def admin_ediciones(request):
    ediciones = Edition.objects.all()
    return render(request, 'charlas/admin_ediciones.html', {'ediciones': ediciones})

@staff_member_required
def admin_nueva_edicion(request):
    form = EditionForm()
    if request.method == 'POST':
        form = EditionForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect('admin_ediciones')
    return render(request, 'charlas/talk_form.html', {
        'form': form,
        'title': 'Nueva Edición',
        'submit_label': 'Crear Edición',
    })

@staff_member_required
@require_POST
def admin_forzar_edicion(request, pk):
    edition = get_object_or_404(Edition, pk=pk)
    edition.es_actual = True
    edition.save()  # el save() de Edition (28-01) desactiva las demás automáticamente
    return redirect('admin_ediciones')
```

3. Rutas en `charlas/urls.py`:

```python
path('admin/ediciones/', views.admin_ediciones, name='admin_ediciones'),
path('admin/ediciones/nueva/', views.admin_nueva_edicion, name='admin_nueva_edicion'),
path('admin/ediciones/<int:pk>/forzar/', views.admin_forzar_edicion, name='admin_forzar_edicion'),
```

4. Template `admin_ediciones.html`: tabla con `nombre`, `anio`, `fecha_inicio`–`fecha_fin`, badge si `es_actual`, botón "Forzar como actual" (POST a `admin_forzar_edicion`) por cada fila que no sea ya la actual, y link a "+ Nueva edición".

5. Agregar el link "Ediciones" al menú de `base_admin.html`.

## Verificación

1. Crear `jfp-2027` desde `/admin/ediciones/nueva/` → aparece en el listado, arranca sin charlas (`Talk.objects.filter(edition=jfp_2027).count() == 0`).
2. Forzar `jfp-2027` como actual → `/` (raíz pública) ahora redirige a `/e/jfp-2027/`, y `jfp-2026` deja de tener `es_actual=True`.
3. El selector de sesión admin (`28-08`) sigue permitiendo consultar `jfp-2026` aunque `jfp-2027` sea la forzada.
