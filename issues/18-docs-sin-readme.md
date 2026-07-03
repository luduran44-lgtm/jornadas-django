---
categoria: documentacion
severidad: baja
esfuerzo: bajo
---

# Sin README ni documentación de setup/deploy

**Archivo:** N/A — no existe `README.md` en la raíz

## Problema

No hay documentación de: cómo levantar el proyecto localmente, qué variables de entorno son obligatorias más allá de `.env.example`, cómo correr los management commands existentes (`seed`, `import_data`, `send_reminders`, `bulk_attendance`, etc.), ni cómo se despliega a producción. Esto hace que cualquier persona nueva (incluido un agente que retome este backlog) tenga que releer todo `views.py` para entender el flujo operativo.

## Fix

Crear `README.md` con, como mínimo:
- Requisitos y setup local (`venv`, `pip install -r requirements.txt`, `.env`, `migrate`, `runserver`).
- Descripción breve de cada management command en `charlas/management/commands/`.
- Cómo generar certificados (WeasyPrint necesita dependencias de sistema — documentarlas).
- Flujo de deploy (dónde corre hoy, qué variables de entorno usa producción).

## Verificación

Una persona sin contexto previo puede levantar el proyecto local siguiendo solo el README, sin preguntar nada.
