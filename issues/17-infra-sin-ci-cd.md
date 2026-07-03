---
categoria: infraestructura
severidad: media
esfuerzo: bajo
---

# Sin CI/CD

**Archivo:** N/A — no existe `.github/workflows/`, ni `pre-commit`, ni ningún pipeline

## Problema

No hay ninguna verificación automática antes de mergear a `main`: ni `python manage.py check`, ni tests (una vez que existan, [[16-testing-sin-cobertura]]), ni linting. Todo el control de calidad depende de que la persona que hace el cambio se acuerde de correrlo a mano.

## Fix

Agregar un workflow de GitHub Actions mínimo que en cada push/PR corra:
1. `python manage.py check`
2. `python manage.py test`
3. (Opcional) `ruff` o `flake8` para linting básico
4. (Opcional) `python manage.py makemigrations --check --dry-run` para detectar migraciones faltantes

## Verificación

Abrir un PR de prueba con un error introducido a propósito (ej. un test que falla) → el workflow debe fallar y bloquear el merge (si se configura como required check).
