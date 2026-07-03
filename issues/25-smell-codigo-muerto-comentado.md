---
categoria: code-smell
severidad: baja
esfuerzo: bajo
---

# Bloques de código muerto comentados con docstrings

**Archivo:** `charlas/views.py:601-606` (función `scanner_dashboard` completa, comentada con `""" ... """`)

## Problema

Usar strings triple-comillados para "comentar" código en vez de borrarlo dificulta la lectura y confunde sobre si esa función está en uso o no (más aún dado que hay una `admin_scan` activa con nombre parecido). Git ya guarda el historial — no hace falta dejar código muerto in-line.

## Fix

Eliminar el bloque comentado. Si `scanner_dashboard` hiciera falta en el futuro, está en el historial de git.

## Verificación

`grep -n '"""' charlas/views.py` no debe encontrar bloques de código comentado (solo docstrings reales de documentación, si las hay).
