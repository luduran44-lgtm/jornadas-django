---
categoria: feature
severidad: alta
esfuerzo: bajo
fase: multi-edicion
orden: 6
depende_de: ["28-03"]
---

# 28-06 — Cierre automático de inscripción/cancelación 1h después del inicio de la charla

**Spec:** `docs/superpowers/specs/2026-07-03-multi-edicion-design.md` (sección "Reglas de negocio nuevas → 2. Cierre automático")

## Objetivo

Una vez que una charla puntual empezó hace más de 1 hora, bloquear autogestión pública de inscripción (`talk_register`) y cancelación (`cancel_registration`) a esa charla. Las vistas admin (`admin_register_student`, `admin_delete_registration`) quedan exentas de esta regla.

## Archivos

- Modificar: `charlas/models.py` — agregar la property `Talk.inscripcion_abierta`.
- Modificar: `charlas/views.py` — `talk_register` y `cancel_registration`.
- Modificar: `charlas/templates/charlas/talk.html` — mostrar mensaje si la inscripción ya cerró en vez de mostrar el formulario.

## Pasos

1. Agregar la property al modelo `Talk` en `charlas/models.py`:

```python
from datetime import timedelta
from django.utils import timezone

class Talk(models.Model):
    # ... campos existentes ...

    @property
    def inicio_datetime(self):
        return timezone.make_aware(
            timezone.datetime.combine(self.date, self.time_start)
        )

    @property
    def inscripcion_abierta(self):
        return timezone.now() < self.inicio_datetime + timedelta(hours=1)
```

2. En `talk_register` (charlas/views.py), agregar el chequeo al principio, antes de procesar el formulario:

```python
def talk_register(request, pk):
    talk = get_object_or_404(Talk, pk=pk)
    if not talk.inscripcion_abierta:
        return render(request, 'charlas/talk.html', {
            'talk': talk,
            'form': None,
            'errors': ['La inscripción a esta charla ya cerró.'],
        })
    # ... resto de la vista sin cambios ...
```

3. En `cancel_registration` (charlas/views.py), agregar el mismo chequeo contra `registration.talk.inscripcion_abierta` antes de eliminar/cancelar:

```python
def cancel_registration(request, token):
    reg = get_object_or_404(Registration, token=token)
    if not reg.talk.inscripcion_abierta:
        return render(request, 'charlas/cancel_success.html', {
            'error': 'Ya no se puede cancelar esta inscripción: la charla ya comenzó hace más de una hora. Contactate con la organización si necesitás ayuda.',
        })
    # ... resto de la vista sin cambios ...
```

4. **No** agregar este chequeo en `admin_register_student` ni `admin_delete_registration` — quedan exentos por diseño.

5. Actualizar `talk.html` para mostrar el mensaje de cierre de forma clara cuando `form` es `None` o cuando `talk.inscripcion_abierta` es `False`.

## Verificación

1. Test manual: crear una charla de prueba con `date`/`time_start` de hace más de 1 hora → `talk_register` y `cancel_registration` públicos la rechazan con mensaje claro.
2. La misma charla sigue permitiendo `admin_register_student` y `admin_delete_registration` desde el panel admin sin restricción.
3. Una charla con `date`/`time_start` futuro (o dentro de la primera hora) permite inscripción y cancelación normalmente.
4. Si ya existen tests de [[16-testing-sin-cobertura]] para este momento del backlog, agregar un caso de test unitario para `Talk.inscripcion_abierta` con los tres escenarios (antes, durante la primera hora, después).
