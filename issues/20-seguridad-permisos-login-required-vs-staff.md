---
categoria: seguridad
severidad: media
esfuerzo: medio
---

# Vistas admin protegidas con `@login_required` en vez de `@staff_member_required`, sin roles

**Archivo:** `charlas/views.py` (la mayoría de las vistas bajo `/admin/...` usan `@login_required`; solo un grupo reducido al final usa `@staff_member_required`)

## Problema

`@login_required` solo exige estar autenticado — no exige `is_staff`. Hoy esto no es explotable porque no hay registro público de usuarios (las únicas cuentas se crean vía `createsuperuser` o el admin de Django), pero es un supuesto implícito y frágil: si en algún momento se crea una cuenta de usuario no-staff (por ejemplo, para un rol futuro tipo "organizador de un solo departamento"), esa cuenta tendría acceso completo a todo el panel admin — crear/borrar charlas, emitir certificados, resolver reclamos, generar tokens de dashboard — sin ninguna verificación adicional. Tampoco hay ningún concepto de rol/permiso granular: es todo-o-nada.

## Fix

1. Reemplazar `@login_required` por `@staff_member_required` en todas las vistas bajo `/admin/...` para que el requisito mínimo sea explícito y consistente.
2. Si en el futuro se necesitan roles (ej. "solo puede administrar su departamento", relevante si cada edición futura suma organizadores distintos — ver [[28-00-feature-multi-edicion-selector]]), evaluar `django.contrib.auth` groups + permissions en vez de un check binario.

## Verificación

Crear un usuario con `is_staff=False` autenticado → debe recibir 403/redirect a login en cualquier URL `/admin/...`, no un 200.
