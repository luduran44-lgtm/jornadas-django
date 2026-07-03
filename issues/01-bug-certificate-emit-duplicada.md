---
categoria: bug
severidad: alta
esfuerzo: bajo
---

# `certificate_emit` duplicada (dead code)

**Archivo:** `charlas/views.py:1033–1078` (primera definición) y `:1082` (segunda, la que realmente corre)

## Problema

Hay dos funciones `def certificate_emit(request):` en el mismo archivo. Python simplemente usa la última definición — la primera (síncrona, sin `EmissionJob`) nunca se ejecuta pero queda ahí confundiendo a cualquiera que lea el archivo o intente modificar el flujo de emisión.

## Fix

Eliminar por completo el bloque de la primera definición (líneas 1033–1078), dejando solo la versión que usa `EmissionJob` para procesar la emisión en background.

## Verificación

- `grep -n "^def certificate_emit" charlas/views.py` debe devolver una sola línea.
- Flujo completo: emitir certificados desde el admin → se crea un `EmissionJob` → se procesa y termina en estado `completado`.
