---
categoria: feature
severidad: baja
esfuerzo: bajo
fase: multi-edicion
orden: 9
depende_de: ["28-02"]
---

# 28-09 — `DashboardToken` atado a una edición fija

**Spec:** `docs/superpowers/specs/2026-07-03-multi-edicion-design.md` (sección "Tokens de dashboard")

## Objetivo

`DashboardToken` ya tiene `edition` (agregado en `28-02`, backfillado a `jfp-2026`). Falta que el flujo de creación del token deje elegir la edición, y que las vistas de dashboard accedidas por token usen esa edición fija en vez de la de sesión/forzada.

## Archivos

- Modificar: `charlas/views.py` — `dashboard_token_crear`, y las vistas de dashboard que aceptan acceso por token (`attendance_dashboard`, `survey_dashboard`, y cualquier otra que chequee `es_token_user`).
- Modificar: `charlas/templates/charlas/dashboard_tokens.html` — agregar el select de edición al formulario de creación.

## Pasos

1. En el formulario/vista `dashboard_token_crear`, agregar un campo para elegir `edition` (select con `Edition.objects.all()`), y guardarlo en el `DashboardToken` creado.
2. En las vistas que se acceden vía `DashboardToken` (revisar `charlas/context_processors.py::es_token_user` y cómo se resuelve el token en `views.py` para ubicar el middleware/vista que valida el token), usar `token.edition` como filtro fijo en vez de la edición de sesión o forzada — es decir, estas vistas dejan de depender de `_edicion_admin_actual(request)` cuando el acceso es por token, y usan directamente `request.dashboard_token.edition`.
3. Actualizar `dashboard_tokens.html` para mostrar a qué edición está atado cada token en el listado.

## Verificación

1. Crear un `DashboardToken` para `jfp-2026` → el dashboard accedido con ese token muestra datos de 2026 incluso después de forzar `jfp-2027` como edición actual.
2. El listado de tokens en `dashboard_tokens.html` muestra la edición de cada uno.
