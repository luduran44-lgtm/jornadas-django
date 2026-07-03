---
categoria: seguridad
severidad: alta
esfuerzo: bajo
---

# Borrado de `Talk` sin paso de confirmación ni soft-delete

**Archivo:** `charlas/views.py` — `admin_delete_talk` (línea ~643)

## Problema

`admin_delete_talk` borra la charla inmediatamente al recibir el POST, sin pantalla de confirmación intermedia server-side (depende de que el frontend muestre un `confirm()` de JS, que es trivialmente bypasseable con un POST directo). Como `Registration.talk` tiene `on_delete=models.CASCADE`, borrar una charla borra en cascada **todas** las inscripciones asociadas — de forma permanente e irreversible. No hay soft-delete ni papelera de reciclaje. Un click accidental (o un bug de frontend que dispare el POST sin querer) puede destruir datos de inscripción que no se pueden recuperar sin un backup de base de datos.

## Fix

1. Corto plazo: agregar una vista de confirmación intermedia (GET muestra "¿confirmás borrar esta charla y sus N inscripciones?", solo el POST desde esa página borra).
2. Medio plazo: evaluar soft-delete (campo `deleted_at` + manager que filtra por default) para `Talk` y `Registration`, dado que son los datos más costosos de perder (certificados y reclamos dependen indirectamente de las inscripciones).
3. Asegurar que existan backups automáticos de la base de datos independientemente de este fix (ver [[27-infra-media-sin-backup]] para el caso análogo de archivos media).

## Verificación

Intentar borrar una charla con inscripciones vía POST directo sin pasar por la pantalla de confirmación → debe rechazarse o exigir un token/paso adicional.
