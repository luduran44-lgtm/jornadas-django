---
categoria: mantenibilidad
severidad: alta
esfuerzo: medio
estado: absorbido-por-28
---

# Hardcoding del año 2026 en código, textos y nombres de archivo

> **Absorbida por el conjunto [[28-01-modelo-edition-y-migracion-inicial]] a [[28-10-alta-de-edicion-admin]].** No implementar el "fix parcial" descripto más abajo por separado — quedaría en conflicto con la migración a `Edition`. Se deja este archivo como registro del problema original y de la motivación del feature completo.

**Archivos:**
- `charlas/views.py:44-46` — `TEMPLATES_CERT = {'diploma': 'JFP2026_DIPLOMA.pdf', ...}`
- `charlas/views.py:147,155` — texto de constancia: `"...de Mayo de 2026..."`
- `charlas/views.py:222` — fallback `'JFP2026_DIPLOMA.pdf'`
- `charlas/views.py:335` — asunto de email: `"Tu certificado — Jornadas de Formación Profesional 2026"`
- `charlas/views.py:825,870` — `PDF_CRONOGRAMA_PATH` y nombre de descarga `cronograma_jfp2026.pdf`
- `charlas/forms.py` — choices de fecha hardcodeadas (ver [[11-smell-fechas-hardcodeadas-reclamo]])

## Problema

El sistema fue construido asumiendo una sola edición (JFP 2026) que nunca cambia. El año, las fechas del cronograma y hasta los nombres de los templates PDF de certificado están escritos literalmente en el código. Para reusar el sistema en 2027 hoy habría que:
- Editar `views.py` a mano en al menos 6 lugares distintos.
- Renombrar/reemplazar los templates PDF de certificado.
- Revisar cada template HTML por si el año está también escrito ahí (pendiente de auditar templates en detalle).

Esto es exactamente la brecha que hace falta cerrar para soportar múltiples ediciones/años. Es la motivación central de [[28-00-feature-multi-edicion-selector]].

## Fix (parcial, sin esperar al modelo de Edición completo)

Como paso intermedio de bajo riesgo, extraer todos estos valores a `CertificateConfig` (que ya existe y ya se usa) o a variables de configuración por edición: nombre de la jornada, año, texto de fechas, rutas de templates PDF. Esto además sienta la base de datos necesaria para [[28-00-feature-multi-edicion-selector]], porque separa "dato de la edición" de "código".

## Verificación

`grep -rn "2026" charlas/` no debe devolver literales de negocio (dejar solo referencias históricas en migraciones/seeds si corresponde) — todo el texto/año visible al usuario debe salir de un modelo, no de un string literal en `views.py`.
