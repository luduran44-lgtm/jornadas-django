# Diseño: modelo de Edición + selector multi-año

- **Fecha:** 2026-07-03
- **Estado:** aprobado para pasar a plan de implementación
- **Relacionado:** `issues/28-00-feature-multi-edicion-selector.md`, `issues/14-hardcoding-anio-2026.md`, `issues/11-smell-fechas-hardcodeadas-reclamo.md`, `issues/22-perf-sin-paginacion.md`, `issues/19-arquitectura-views-god-file.md`

## Contexto y objetivo

El sistema fue construido asumiendo una sola edición para siempre (JFP 2026): el año y las fechas del cronograma están hardcodeados en varios puntos de `views.py`, y no existe ningún modelo que represente "una edición de la jornada". Para reutilizar el sistema en 2027 y años siguientes — conservando el historial de ediciones pasadas consultable (certificados, encuestas, estadísticas) — se introduce un modelo `Edition` de primera clase, del cual dependen todas las entidades operativas del sistema.

Requisito de producto explícito: debe existir un selector público de edición (para navegar ediciones pasadas), pero el administrador puede **forzar cuál es la "edición actual"** — la que ve el público por defecto y la única donde se puede operar en vivo — independientemente de qué edición esté navegando cada visitante.

Sumado a esto: una vez que una charla concreta ya empezó (+1 hora de margen), nadie puede autogestionar su inscripción o cancelación a esa charla — solo un administrador puede hacerlo manualmente desde ese punto en adelante.

## Modelo de datos

### Nuevo modelo `Edition`

| Campo | Tipo | Notas |
|---|---|---|
| `slug` | `SlugField`, único | Identificador propio, no solo el año — permite más de una edición en el mismo año calendario (ej. `jfp-2026`). Usado en la URL pública. |
| `nombre` | `CharField` | Nombre para mostrar (ej. "JFP 2026"). |
| `anio` | `PositiveIntegerField` | Año calendario de la edición. |
| `fecha_inicio` / `fecha_fin` | `DateField` | Rango de fechas del evento. |
| `es_actual` | `BooleanField`, default `False` | La edición forzada. Se garantiza unicidad (una sola `True` a la vez) sobreescribiendo `save()` para desactivar cualquier otra edición al activar una, igual que el patrón ya usado en `CertificateConfig.activa`. |

### FKs nuevas (directas, no heredadas vía Talk)

Se agrega `edition = models.ForeignKey(Edition, on_delete=models.PROTECT)` a:
- `Talk`
- `Registration`
- `Certificate`
- `Reclamo`
- `CertificateConfig`
- `DashboardToken`

`on_delete=PROTECT` para evitar borrar una edición con datos históricos asociados por accidente — borrar una edición completa, si alguna vez hace falta, debe ser una acción explícita y consciente, no un efecto secundario en cascada.

`CertificateConfig` deja de tener un único registro "activo" global: pasa a haber una config por edición, ligada 1:1 conceptualmente (una edición activa tiene una `CertificateConfig` propia con sus propios `modalidad`, `minimo`, `descarga_habilitada`, `dias_reclamo`, etc.). Esto permite que 2027 tenga reglas de elegibilidad distintas a 2026 sin tocar la config histórica ya usada para emitir certificados pasados.

### `Talk.date`: de texto libre a `DateField`

Hoy `Talk.date` es `CharField` con valores como `"Martes 19 de Mayo"`, sin año, usado también como identificador de "día" en `Reclamo.dia` y `Reclamo.dias_perdonados_list`. Se convierte a `DateField` real:
- El texto para mostrar ("Martes 19 de Mayo") se genera con formateo de fecha localizado en el template (`{{ talk.date|date:"l j \\d\\e F" }}` con `LANGUAGE_CODE = es-ar`), no se guarda como string.
- `Reclamo.dia` y `dias_perdonados_list` pasan a almacenar fechas ISO (`date`) en vez de texto libre, consistente con `Talk.date`.
- Esto resuelve de raíz `issues/11` (choices hardcodeadas del select de día) y buena parte de `issues/14` (hardcoding de fechas).

### `Talk`: horario de inicio estructurado

Se agrega `time_start = models.TimeField()` (hora de inicio real). El campo `time` (texto libre, ej. `"18:00 a 20:00"`) se mantiene como texto para mostrar el rango horario completo, pero ya no es la única fuente de verdad: `TalkForm` ya recolecta `time_start`/`time_end` como inputs de tipo hora por separado (ver `charlas/forms.py`), así que este cambio es guardar `time_start` también como campo real del modelo en vez de solo concatenarlo al string `time`.

Con `Talk.date` (`DateField`) + `Talk.time_start` (`TimeField`) se puede calcular `inicio_datetime = datetime.combine(talk.date, talk.time_start)` de forma exacta.

## Reglas de negocio nuevas

### 1. Edición forzada (`Edition.es_actual=True`)

- Define qué charlas ve el público por defecto en la navegación sin edición explícita en la URL.
- Es la **única** edición donde `talk_register` acepta inscripciones nuevas.
- Es el filtro por defecto de todos los dashboards admin (asistencia, reclamos, certificados, encuestas) — el admin puede cambiar la edición que está consultando sin afectar lo que ve el público.
- Es la única edición donde se pueden cargar reclamos nuevos (`reclamo_nuevo`) o completar encuestas de certificado (`survey`).

### 2. Cierre automático de inscripción/cancelación por charla

Pasada 1 hora desde `inicio_datetime` de una charla puntual:
- `talk_register` deja de aceptar inscripciones nuevas a esa charla.
- `cancel_registration` deja de aceptar cancelaciones de inscripciones a esa charla.
- Mismo criterio para ambas acciones — no hay ventanas distintas para inscribirse vs. desinscribirse.
- **El administrador queda exento**: `admin_register_student`, `admin_delete_registration` y cualquier acción equivalente desde `/admin/...` siguen funcionando sin este chequeo, en cualquier momento.
- No se agrega un estado global "edición finalizada": como cada charla se cierra individualmente, para cuando termina la jornada completa todas sus charlas ya están cerradas por esta misma regla — un flag adicional sería redundante.
- Implementación: un chequeo compartido (ej. `talk.inscripcion_abierta` como `@property`, o una función `_inscripcion_abierta(talk)`) reutilizado por ambas vistas públicas, comparando `timezone.now()` contra `inicio_datetime + timedelta(hours=1)`.

## Navegación y URLs

- Prefijo `/e/<slug>/` para las rutas públicas de navegación: `/e/jfp-2026/`, `/e/jfp-2026/talk/12/`.
- La raíz `/` redirige siempre a `/e/<slug-de-la-edición-forzada>/`.
- Los links con token o pk propio ya distribuidos **no** llevan el prefijo — la edición se resuelve a partir del objeto referenciado, no de la URL:
  - `/cancel/<token>/`
  - `/reclamo/ampliar/<pk>/<token>/`
  - `/admin/scan/<pk>/`
  - Cualquier otro link con token/pk único (validación/descarga de certificado, encuesta).
  - Esto evita romper QRs ya impresos y emails ya enviados para JFP 2026.

## Panel admin

- Selector de sesión ("Viendo: JFP 2026 ▾") agregado a `base_admin.html`, que guarda la edición elegida en la sesión del usuario admin.
- Las URLs `/admin/...` no cambian de forma ni de prefijo.
- Todas las vistas admin que listan/filtran datos (dashboards, listados de charlas, reclamos, certificados) usan la edición de la sesión admin como filtro por defecto — separado por completo de cuál sea la edición forzada para el público.

## Tokens de dashboard

`DashboardToken` gana FK a `Edition`, elegida por el admin al crear el token. Un token creado para "JFP 2026" sigue mostrando siempre esa edición, incluso después de que `es_actual` pase a otra edición — el link ya compartido con un tercero (ej. Rectorado) no cambia de contenido solo.

## Alta de una edición nueva

La edición nueva arranca **vacía**, sin charlas — el admin las carga una por una como ya hace hoy. Clonar charlas desde una edición anterior queda **fuera de alcance** de este diseño; puede evaluarse como mejora incremental posterior sin bloquear esta implementación.

## Plan de migración de datos existentes

1. Crear una migración de datos que instancie la edición inicial: `slug="jfp-2026"`, `nombre="JFP 2026"`, `anio=2026`, `es_actual=True`, con `fecha_inicio`/`fecha_fin` tomadas del rango real de `Talk.date` existente.
2. Backfill: asignar esa edición a todos los registros existentes de `Talk`, `Registration`, `Certificate`, `Reclamo`, `CertificateConfig` y `DashboardToken` antes de que el FK se vuelva `NOT NULL`.
3. Reconstruir `Talk.date` (texto → `DateField`): parsear cada valor existente (ej. `"Martes 19 de Mayo"`) asumiendo el año de la edición (2026), y **verificar que el día de la semana calculado coincida con el texto** antes de aplicar — si no coincide, la migración debe fallar de forma visible en vez de guardar una fecha incorrecta silenciosamente.
4. Poblar `Talk.time_start` parseando la parte inicial del `time` existente (ej. `"18:00 a 20:00"` → `18:00`). Si algún registro no matchea el formato esperado, listarlo para corrección manual antes de continuar — no asumir un valor por defecto.
5. Migrar `Reclamo.dia` y `dias_perdonados_list` de texto a fechas ISO usando el mismo mapeo texto→fecha ya construido en el paso 3.
6. Recién con el backfill completo, aplicar la migración de esquema que vuelve los FKs `NOT NULL` y `Talk.date`/`Talk.time_start` sus tipos finales.

## Fuera de alcance

- Clonado de charlas entre ediciones.
- Roles o permisos granulares por edición (ej. "organizador solo de 2027") — sigue siendo todo-o-nada a nivel staff, ver `issues/20`.
- Conversión de `Talk.time` completo (fin del horario) a tipo estructurado — solo se estructura el inicio, necesario para la regla de cierre automático.
- Estado explícito "edición finalizada" a nivel `Edition` — cubierto indirectamente por el cierre automático por charla.
- Selector visual/UI final del componente de navegación por edición en el frontend público (queda para la etapa de implementación de UI, no es una decisión de arquitectura).

## Riesgos y mitigaciones

- **Riesgo:** la reconstrucción de `Talk.date` desde texto libre puede fallar silenciosamente si el formato real en base tiene variantes no previstas. *Mitigación:* la migración de datos verifica día-de-semana calculado vs. texto original y aborta con un reporte claro si no coincide, en vez de continuar con datos incorrectos.
- **Riesgo:** cambiar `CertificateConfig` de "una activa global" a "una por edición" puede romper `signals.py` (`on_config_guardada`), que hoy asume una sola config activa en todo el sistema. *Mitigación:* el signal debe filtrar por `instance.edition` al limpiar certificados no elegibles, no por todo el sistema.
- **Riesgo:** este trabajo toca gran parte de `views.py` de todos modos — buen momento para abordar en paralelo `issues/19` (dividir el archivo) y `issues/22` (paginación, ahora naturalmente filtrable por edición), pero eso se decide en el plan de implementación, no en este spec, para no mezclar objetivos.
- **Riesgo:** requiere tests mínimos (`issues/16`) antes de encarar esta migración de datos — sin cobertura previa, verificar que nada se rompió depende de pruebas manuales exclusivamente.

## Criterios de aceptación

1. Existen al menos dos ediciones activas en el sistema de prueba (`jfp-2026` archivada, `jfp-2027` como forzada) sin que se mezclen datos entre ellas en ningún dashboard o listado público.
2. Cambiar `es_actual` a una nueva edición redirige la raíz pública (`/`) a esa edición sin romper ningún link con token existente de la edición anterior.
3. Una charla con `inicio_datetime` de hace más de 1 hora rechaza inscripción y cancelación pública, pero sigue permitiendo `admin_register_student`/`admin_delete_registration`.
4. Todos los certificados, encuestas y reclamos de JFP 2026 emitidos antes de esta migración siguen siendo accesibles y correctos después de aplicarla.
