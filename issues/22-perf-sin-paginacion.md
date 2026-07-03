---
categoria: rendimiento
severidad: media
esfuerzo: medio
estado: absorbido-por-28
---

# Sin paginación en dashboards y listados admin

> **Se resuelve dentro de [[28-08-selector-admin-y-paginacion]]**, en el mismo trabajo que agrega el selector de edición en sesión admin — la edición de sesión es el filtro natural que además reduce el tamaño de cada página. No implementar la paginación por separado antes de esa issue para no duplicar el trabajo sobre las mismas vistas.

**Archivos:** `charlas/views.py` — `admin_dashboard` (`Talk.objects.all()`), `reclamos_dashboard` (`Reclamo.objects.all()`), `admin_talk_details` (`talk.registrations.all()`)

## Problema

Ningún listado usa `Paginator`. Con el volumen actual (una sola edición, algunas decenas de charlas, cientos de inscriptos) no se nota, pero el problema se agrava directamente con [[28-00-feature-multi-edicion-selector]]: si cada año se acumulan más charlas y reclamos en las mismas tablas, estos listados van a crecer sin límite y sin filtro por edición, degradando cada vez más el tiempo de carga del panel admin.

## Fix

Ver pasos concretos en [[28-08-selector-admin-y-paginacion]].

## Verificación

Con un dataset de prueba de 500+ registros en `Reclamo`, `reclamos_dashboard` debe cargar en tiempo similar al de 20 registros (paginado), no degradarse linealmente.
