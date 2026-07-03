---
categoria: modelo-de-datos
severidad: alta
esfuerzo: bajo
---

# Sin `unique_together` en `Registration(talk, dni)`

**Archivo:** `charlas/models.py` (`Registration.Meta`)

## Problema

Un alumno puede inscribirse dos veces a la misma charla si hace doble submit en el formulario web, o si un import de datos lo agrega de nuevo. Existe lógica en `_procesar_fila` (import) que intenta prevenir duplicados, pero la inscripción web (`talk_register`, `admin_register_student`) no tiene esa protección a nivel de base de datos — solo el chequeo de aplicación `_duplicate_exists`, que tiene una condición de carrera entre el check y el `create`.

## Fix

Agregar `unique_together = ('talk', 'dni')` en `Registration.Meta` y generar la migración. **Antes de aplicar la migración**, correr una query de detección de duplicados existentes y resolverlos manualmente (la migración fallará si ya hay filas duplicadas).

## Verificación

1. Detectar duplicados existentes: `Registration.objects.values('talk', 'dni').annotate(c=Count('id')).filter(c__gt=1)`.
2. Resolver duplicados si los hay.
3. Migrar.
4. Doble submit del formulario de inscripción → la segunda inserción debe fallar de forma controlada (capturar `IntegrityError` y mostrar el mismo mensaje de "ya se encuentra inscripto").
