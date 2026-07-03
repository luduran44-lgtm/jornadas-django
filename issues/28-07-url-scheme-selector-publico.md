---
categoria: feature
severidad: alta
esfuerzo: medio
fase: multi-edicion
orden: 7
depende_de: ["28-01", "28-02"]
---

# 28-07 — Prefijo `/e/<slug>/` y redirección de la raíz a la edición forzada

**Spec:** `docs/superpowers/specs/2026-07-03-multi-edicion-design.md` (sección "Navegación y URLs")

## Objetivo

Las rutas públicas de navegación (index, detalle/inscripción de charla) quedan bajo `/e/<slug>/`. La raíz `/` redirige siempre a la edición con `es_actual=True`. Los links con token o pk propio (cancelación, ampliación de reclamo, escaneo QR, validación/descarga de certificado, encuesta) **no** se tocan — siguen resolviendo la edición desde el objeto referenciado, sin prefijo.

## Archivos

- Modificar: `charlas/urls.py` — agregar el grupo de rutas bajo `/e/<slug:edition_slug>/` y la vista de redirección en `/`.
- Modificar: `charlas/views.py` — `index` y `talk_register` (y `talk_detail` si existe como vista separada) pasan a recibir `edition_slug` y resolver `Edition.objects.get(slug=edition_slug)` en vez de operar implícitamente sobre "todas las charlas".
- Modificar: templates que arman URLs con `{% url %}` hacia `index`/`talk_register` — agregar `edition_slug` como parámetro donde corresponda.

## Pasos

1. En `charlas/urls.py`, envolver las rutas públicas de navegación bajo el prefijo, y agregar una vista de redirección en la raíz:

```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.index_redirect, name='index_redirect'),
    path('e/<slug:edition_slug>/', views.index, name='index'),
    path('e/<slug:edition_slug>/talk/<int:pk>/', views.talk_register, name='talk_register'),

    # Sin prefijo — resuelven la edición desde el objeto (token/pk propio)
    path('cancel/<str:token>/', views.cancel_registration, name='cancel_registration'),
    # ... el resto de las rutas con token/pk (reclamo/ampliar, admin/scan, certificado/*, encuesta) sin cambios ...
]
```

2. Nueva vista `index_redirect` en `charlas/views.py`:

```python
def index_redirect(request):
    edicion_actual = get_object_or_404(Edition, es_actual=True)
    return redirect('index', edition_slug=edicion_actual.slug)
```

3. Actualizar `index` y `talk_register` para recibir `edition_slug` y resolver la edición, filtrando las charlas mostradas por esa edición:

```python
def index(request, edition_slug):
    edition = get_object_or_404(Edition, slug=edition_slug)
    talks = Talk.objects.filter(edition=edition)
    # ... resto de la vista usa `talks` ya filtradas y pasa `edition` al contexto ...
```

4. En `talk_register`, agregar `edition_slug` a la firma y validar coherencia (la charla pedida por `pk` debe pertenecer a esa edición, si no, 404):

```python
def talk_register(request, edition_slug, pk):
    talk = get_object_or_404(Talk, pk=pk, edition__slug=edition_slug)
    # ... resto sin cambios ...
```

5. Revisar todos los `{% url 'index' %}` y `{% url 'talk_register' pk=... %}` en templates (`index.html`, `talk.html`, `base.html`, emails) y agregar `edition_slug=talk.edition.slug` (o `edition.slug` según el contexto disponible).

6. Selector visual (dropdown simple en `base.html`, listando `Edition.objects.all()` ordenadas por año, cada una linkeando a `{% url 'index' edition_slug=ed.slug %}`) — implementación de UI a criterio de quien ejecute esta issue, no hay una decisión de arquitectura pendiente acá.

## Verificación

1. `/` redirige a `/e/jfp-2026/` (o la edición que tenga `es_actual=True` en ese momento).
2. `/e/jfp-2026/talk/12/` funciona si la charla 12 pertenece a esa edición; devuelve 404 si pertenece a otra edición.
3. Los links existentes `/cancel/<token>/`, `/reclamo/ampliar/<pk>/<token>/`, `/admin/scan/<pk>/`, `/certificado/validar/`, `/certificado/descarga/` siguen funcionando exactamente igual que antes (regresión manual sobre cada uno).
4. Cambiar `es_actual` a otra edición de prueba (`jfp-2027`) y confirmar que `/` redirige ahora a `/e/jfp-2027/`.
