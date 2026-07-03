---
categoria: feature
severidad: baja
esfuerzo: alto
fase: multi-edicion
orden: 11
opcional: true
depende_de: []
---

# 28-11 (opcional) — Dividir `views.py` en módulos aprovechando el trabajo de multi-edición

**Relacionada con:** [[19-arquitectura-views-god-file]]

## Objetivo

No es parte obligatoria de la secuencia `28-01`…`28-10`. Se deja registrada porque `28-06`, `28-07` y `28-08` ya van a tocar la mayoría de las vistas de `charlas/views.py` de todos modos (inscripción, dashboards, admin) — es un buen momento para, en el mismo esfuerzo, dividir el archivo en el paquete `charlas/views/` descripto en [[19-arquitectura-views-god-file]], en vez de seguir agrandando un único archivo de 2000+ líneas con más lógica de edición.

## Cuándo hacerla

Después de `28-08` (una vez que ya están definidas las vistas nuevas/modificadas de esta feature), y **solo si ya existen tests mínimos** ([[16-testing-sin-cobertura]]) que permitan verificar que el split no rompió nada — dividir un archivo de este tamaño a ciegas es arriesgado.

## Alcance

Igual al descripto en [[19-arquitectura-views-god-file]]: `views/public.py`, `views/attendance.py`, `views/certificates.py`, `views/surveys.py`, `views/reclamos.py`, `views/admin_talks.py`, `views/tokens.py`, más un nuevo `views/editions.py` para las vistas agregadas en `28-07`/`28-08`/`28-10`.

## Verificación

`python manage.py check`, suite de tests en verde, y todas las URLs de `urls.py` siguen resolviendo sin cambios de comportamiento.
