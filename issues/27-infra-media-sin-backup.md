---
categoria: infraestructura
severidad: media
esfuerzo: medio
---

# Archivos media sin estrategia de backup/almacenamiento externo

**Archivo:** `jornadas/settings.py:74-75` — `MEDIA_ROOT = BASE_DIR / 'media'` (almacenamiento local en disco)

## Problema

Certificados PDF emitidos, adjuntos de reclamos, PDFs de disertantes e imágenes de charlas se guardan en el filesystem local del servidor, sin backup automático evidente ni almacenamiento redundante (no S3/GCS/Azure Blob). Si el servidor se pierde, se resetea, o hay un deploy que no preserva `MEDIA_ROOT`, se pierden certificados de años anteriores de forma irrecuperable — esto se vuelve más grave a medida que se acumulan ediciones ([[28-00-feature-multi-edicion-selector]]) y crece el volumen de certificados históricos que la gente puede necesitar re-descargar.

## Fix

1. Corto plazo: confirmar que existe un backup periódico (a nivel infraestructura/hosting) de la carpeta `media/`, no solo de la base de datos.
2. Medio plazo: evaluar migrar `MEDIA_ROOT` a almacenamiento de objetos (S3-compatible) usando `django-storages`, que además simplifica escalar horizontalmente si el hosting deja de ser un único servidor con disco persistente.

## Verificación

Confirmar (con quien gestiona el hosting) que existe y se prueba periódicamente un restore de `media/` desde backup. Si no existe, este issue queda bloqueado hasta que se defina la estrategia de hosting.
