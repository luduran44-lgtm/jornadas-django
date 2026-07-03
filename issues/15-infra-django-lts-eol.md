---
categoria: infraestructura
severidad: alta
esfuerzo: medio
---

# Django 4.2 LTS con soporte extendido vencido/por vencer

**Archivo:** `requirements.txt:10` — `Django==4.2.30`

## Problema

Django 4.2 es LTS con soporte de seguridad extendido hasta abril de 2026. A la fecha de esta auditoría (julio 2026) ya no recibe parches de seguridad. Cualquier CVE nuevo sobre Django 4.2 queda sin corrección oficial. El resto del stack (`pillow`, `cryptography`, `requests`, etc.) está en versiones bastante recientes, así que el salto de Django es probablemente el único bloqueante real para actualizar.

## Fix

1. Revisar el [release notes / deprecation timeline](https://docs.djangoproject.com/en/stable/internals/release-process/) de Django 5.x LTS (5.2 es la próxima LTS).
2. Correr `python manage.py check --deploy` y los tests (una vez que existan — ver [[16-testing-sin-cobertura]]) contra Django 5.2 en un branch aparte antes de mergear.
3. Prestar atención a cambios de comportamiento en `FileField`/`ImageField`, `forms`, y el manejo de `USE_TZ`/`zoneinfo` entre versiones.
4. Planificar el upgrade para una ventana sin actividad de jornada activa (no durante el período de inscripciones/certificados).

## Verificación

`pip list --outdated`, `python manage.py check --deploy` sin warnings nuevos, suite de tests en verde contra la nueva versión.
