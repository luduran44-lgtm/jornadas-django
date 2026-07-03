---
categoria: seguridad
severidad: media
esfuerzo: bajo
---

# Validación insuficiente de archivos subidos

**Archivos:** `charlas/forms.py` (`AttendanceImportForm.csv_file`, `TalkForm.image`), `charlas/models.py` (`Reclamo.archivo`)

## Problema

- `AttendanceImportForm.csv_file` solo restringe la extensión desde el atributo HTML `accept='.csv'`, que es una sugerencia del navegador y no una validación real — nada impide subir un archivo con otra extensión o contenido renombrado a `.csv` vía POST directo.
- `Talk.image` (`ImageField`) y `Reclamo.archivo` (`FileField`) no tienen límite de tamaño explícito ni validación de tipo MIME real en el servidor — solo lo que Pillow valida implícitamente para `ImageField` (que sí verifica que sea una imagen válida, pero no el tamaño).
- No hay límite de tamaño de archivo configurado a nivel Django (`DATA_UPLOAD_MAX_MEMORY_SIZE`/`FILE_UPLOAD_MAX_MEMORY_SIZE` quedan en default), lo que permite subir archivos grandes y consumir memoria/disco del servidor.

## Fix

1. Agregar un `clean_csv_file` en `AttendanceImportForm` que valide extensión real y, si es posible, intente parsear la primera línea antes de aceptar el archivo.
2. Agregar validadores de tamaño (`django.core.validators.FileExtensionValidator` + un validador de tamaño custom) a `Reclamo.archivo` y `Talk.image`.
3. Configurar `DATA_UPLOAD_MAX_MEMORY_SIZE` y `FILE_UPLOAD_MAX_MEMORY_SIZE` en `settings.py` con un límite razonable (ej. 5-10 MB).

## Verificación

Subir un archivo de 50MB al formulario de reclamo → debe rechazarse con un error de validación claro, no consumir recursos del servidor sin límite.
