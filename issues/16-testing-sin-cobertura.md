---
categoria: testing
severidad: alta
esfuerzo: alto
---

# Cobertura de tests inexistente

**Archivo:** `charlas/tests.py` (3 líneas, vacío)

## Problema

No hay ningún test automatizado sobre lógica que es genuinamente compleja y con casos borde ya conocidos: evaluación de elegibilidad para certificado (`_evaluar_alumno`, con dos modalidades y perdones), flujo de reclamos (estados, idempotencia, ampliaciones), emisión masiva de certificados, e importación de asistencia por CSV. Cada fix de este backlog (empezando por los bugs de Tier 1) se aplica "a ciegas", sin forma de confirmar que no rompe otro caso, y cualquier refactor futuro (incluyendo introducir el modelo de Edición) es mucho más riesgoso sin esta red de seguridad.

## Fix

Priorizar tests sobre las funciones puras/críticas primero (dan más cobertura por esfuerzo):
1. `_evaluar_alumno` — todas las combinaciones de modalidad (`total`/`por_dia`) × perdones × asistencia real.
2. `reclamo_resolver` — idempotencia, cada `resolucion` posible.
3. `_duplicate_exists` / inscripción — duplicados por DNI/legajo.
4. Flujo de `certificate_download` / `certificate_validate` — casos de config inactiva, encuesta obligatoria pendiente, certificado no elegible.
5. Import de asistencia por CSV — filas inválidas, DNIs no encontrados, duplicados.

Usar `django.test.TestCase` con fixtures mínimas por test (no depender de `db.sqlite3` con datos reales).

## Verificación

`python manage.py test` corre y pasa; medir cobertura con `coverage run manage.py test && coverage report` como línea de base para ir subiendo.
