---
categoria: feature
severidad: media
esfuerzo: alto
fase: multi-edicion
orden: 8
depende_de: ["28-02", "20-seguridad-permisos-login-required-vs-staff"]
---

# 28-08 — Selector de edición en sesión admin + filtrado de dashboards + paginación

**Spec:** `docs/superpowers/specs/2026-07-03-multi-edicion-design.md` (sección "Panel admin")

## Prerrequisito

Hacer primero [[20-seguridad-permisos-login-required-vs-staff]]: esta issue agrega superficie nueva al panel admin (el selector) y conviene que toda esa superficie ya use `@staff_member_required` de forma consistente desde el inicio, no `@login_required`.

## Objetivo

Agregar un selector de edición en la sesión del admin (no cambia las URLs `/admin/...`), y usar esa edición de sesión como filtro por defecto en todos los dashboards y listados admin. Se aprovecha esta issue para agregar paginación (resolviendo [[22-perf-sin-paginacion]] en el mismo trabajo, en vez de como fix aislado) ya que la edición de sesión es el filtro natural que reduce el tamaño de cada listado.

## Archivos

- Crear: `charlas/context_processors.py` — agregar `edicion_admin_actual` (edición de sesión, con fallback a la forzada).
- Modificar: `charlas/views.py` — nueva vista `admin_cambiar_edicion`, y `admin_dashboard`, `reclamos_dashboard`, `admin_talk_details` (y cualquier otro listado admin) filtran por la edición de sesión + agregan `Paginator`.
- Modificar: `charlas/templates/charlas/base_admin.html` — agregar el dropdown "Viendo: <edición> ▾".
- Modificar: `charlas/urls.py` — nueva ruta para cambiar la edición de sesión.

## Pasos

1. Nueva vista para cambiar la edición de sesión:

```python
@staff_member_required
@require_POST
def admin_cambiar_edicion(request):
    edition_id = request.POST.get('edition_id')
    edition = get_object_or_404(Edition, pk=edition_id)
    request.session['admin_edition_id'] = edition.id
    return redirect(request.META.get('HTTP_REFERER', 'admin_dashboard'))
```

   Ruta en `urls.py`: `path('admin/cambiar-edicion/', views.admin_cambiar_edicion, name='admin_cambiar_edicion')`.

2. Helper reutilizable para resolver la edición admin actual (usar en cada vista, o centralizarlo en un context processor):

```python
def _edicion_admin_actual(request):
    edition_id = request.session.get('admin_edition_id')
    if edition_id:
        edition = Edition.objects.filter(pk=edition_id).first()
        if edition:
            return edition
    return Edition.objects.filter(es_actual=True).first()
```

3. Context processor en `charlas/context_processors.py` para que el dropdown esté disponible en todos los templates admin sin pasarlo manualmente:

```python
def edicion_admin_actual(request):
    if not request.user.is_authenticated:
        return {}
    from .views import _edicion_admin_actual
    from .models import Edition
    return {
        'edicion_admin_actual': _edicion_admin_actual(request),
        'todas_las_ediciones': Edition.objects.all(),
    }
```

   Agregarlo a `TEMPLATES[0]['OPTIONS']['context_processors']` en `jornadas/settings.py`.

4. Actualizar `admin_dashboard`, `reclamos_dashboard`, `admin_talk_details` (y el resto de listados/dashboards admin) para filtrar por `_edicion_admin_actual(request)` y agregar paginación:

```python
from django.core.paginator import Paginator

@staff_member_required
def admin_dashboard(request):
    edition = _edicion_admin_actual(request)
    talks_qs = Talk.objects.filter(edition=edition)
    paginator = Paginator(talks_qs, 25)
    page = paginator.get_page(request.GET.get('page'))
    return render(request, 'charlas/admin_dashboard.html', {'talks': page})
```

   Aplicar el mismo patrón (filtro por edición + `Paginator`) a `reclamos_dashboard` (sobre `Reclamo.objects.filter(edition=edition)`) y a `admin_talk_details` (sobre `talk.registrations.all()`, ya acotado porque `talk` ya pertenece a una edición).

5. Agregar el dropdown a `base_admin.html`:

```html
<form method="post" action="{% url 'admin_cambiar_edicion' %}" class="edition-switcher">
  {% csrf_token %}
  <select name="edition_id" onchange="this.form.submit()">
    {% for ed in todas_las_ediciones %}
      <option value="{{ ed.id }}" {% if ed == edicion_admin_actual %}selected{% endif %}>{{ ed.nombre }}</option>
    {% endfor %}
  </select>
</form>
```

## Verificación

1. Cambiar la edición desde el dropdown persiste en la sesión y los dashboards admin muestran solo datos de esa edición.
2. La edición de sesión del admin es independiente de `Edition.es_actual` — cambiarla en el admin no afecta lo que ve el público en `/e/<slug>/`.
3. Con un dataset de prueba de 500+ reclamos en una edición, `reclamos_dashboard` pagina en vez de listar todo de una vez (verificar con `?page=2`).
4. Todas las vistas tocadas en esta issue usan `@staff_member_required`, no `@login_required`.
