---
categoria: feature
severidad: media
esfuerzo: bajo
fase: multi-edicion
orden: 5
depende_de: ["28-02"]
---

# 28-05 — `CertificateConfig` por edición

**Spec:** `docs/superpowers/specs/2026-07-03-multi-edicion-design.md` (sección "Nuevo modelo Edition" y "Riesgos y mitigaciones")

## Objetivo

Ahora que `CertificateConfig` tiene `edition` (agregado en `28-02`), actualizar la lógica que hoy asume "una sola config activa en todo el sistema" para que sea "una config activa por edición".

## Archivos

- Modificar: `charlas/signals.py` — `on_config_guardada` y `_limpiar_certificados_no_elegibles`.
- Modificar: `charlas/views.py` — todos los `CertificateConfig.objects.filter(activa=True).first()` (buscar con `grep -n "CertificateConfig.objects.filter(activa=True)" charlas/views.py`) deben además filtrar por la edición correspondiente (la forzada para vistas públicas, la de sesión para vistas admin — ver `28-07` y `28-08`).
- Modificar: `charlas/models.py` — el `save()` de `CertificateConfig` (si no existe, agregarlo) debe garantizar `activa=True` único **por edición**, no global, igual al patrón de `Edition.es_actual`:

```python
def save(self, *args, **kwargs):
    if self.activa:
        CertificateConfig.objects.filter(edition=self.edition).exclude(pk=self.pk).update(activa=False)
    super().save(*args, **kwargs)
```

## Pasos

1. Agregar el `save()` de arriba a `CertificateConfig` en `charlas/models.py`.
2. En `charlas/signals.py`, `_limpiar_certificados_no_elegibles` recibe `config_id`; agregar el filtro por edición al calcular `dnis_con_asistencia`, `dnis_con_reclamo` y `dnis_con_cert`, todos acotados a `config.edition`:

```python
def _limpiar_certificados_no_elegibles(config_id):
    from charlas.models import Certificate, CertificateConfig, Registration, Reclamo, Talk
    from charlas.views import _evaluar_alumno

    config = CertificateConfig.objects.get(id=config_id)
    dias_jornada = set(
        Talk.objects.filter(edition=config.edition).values_list('date', flat=True).distinct()
    )
    dnis_con_asistencia = set(
        Registration.objects.filter(edition=config.edition, attended=True).values_list('dni', flat=True)
    )
    dnis_con_reclamo = set(
        Reclamo.objects.filter(edition=config.edition, estado='aprobado').values_list('dni', flat=True)
    )
    todos_dnis = dnis_con_asistencia | dnis_con_reclamo
    elegibles = {dni for dni in todos_dnis if _evaluar_alumno(dni, config, dias_jornada)}
    dnis_con_cert = set(
        Certificate.objects.filter(edition=config.edition).values_list('dni', flat=True)
    )
    no_califican = dnis_con_cert - elegibles
    if no_califican:
        Certificate.objects.filter(edition=config.edition, dni__in=no_califican).delete()
```

3. Revisar cada `CertificateConfig.objects.filter(activa=True).first()` en `charlas/views.py` y agregar `edition=<edición relevante>` al filtro.

## Verificación

1. Activar una `CertificateConfig` para `jfp-2026` y otra para una edición de prueba `jfp-2027` — ambas quedan `activa=True` simultáneamente (una por edición), sin que activar una desactive la otra.
2. `_limpiar_certificados_no_elegibles` sobre la config de 2027 no borra ni afecta certificados de 2026.
3. `certificate_download`/`certificate_validate` siguen funcionando igual que antes para certificados de 2026 ya emitidos.
