---
categoria: seguridad
severidad: media
esfuerzo: medio
---

# Sin rate-limiting en endpoints públicos sensibles

**Archivos:** `charlas/views.py` — `talk_register`, `reclamo_nuevo`, `certificate_validate`, `certificate_download`, `api_scan`

## Problema

Varios endpoints públicos, sin autenticación, permiten búsquedas o intentos por DNI/código sin ningún límite de tasa:
- `certificate_validate` y `certificate_download` aceptan `POST` con `dni` + `codigo`/datos, sin límite de intentos — permiten fuerza bruta para enumerar DNIs válidos o códigos de certificado, y scraping masivo de datos personales (nombre, apellido, charlas a las que asistió).
- `talk_register` y `reclamo_nuevo` no tienen protección contra spam/bots (sin CAPTCHA, sin límite por IP).
- `api_scan` requiere estar autenticado (`login_required_json`) así que el riesgo ahí es menor, pero sigue sin límite de tasa contra fuerza bruta de tokens si una sesión de admin quedara comprometida.

## Fix

Agregar rate-limiting básico por IP en estos endpoints. Opciones sin agregar infraestructura nueva:
- `django-ratelimit` (liviano, decorador sobre la view).
- O un middleware simple basado en cache (Django ya soporta cache en DB/local-memory) que cuente requests por IP/ventana de tiempo.

Priorizar `certificate_validate` y `certificate_download` por ser los que exponen más datos personales ante fuerza bruta.

## Verificación

Enviar 50+ requests en un minuto desde la misma IP a `certificate_validate` → a partir de cierto umbral debe responder 429 en vez de seguir procesando.
