# Documentación Técnica - Suite de Pruebas para `Blueprints` de Flask

### 1. Información General

| Atributo | Detalle |
| :--- | :--- |
| **Tipo de Archivo** | Suite de pruebas unitarias / integración |
| **Framework** | `pytest` + `Flask` |
| **Lenguaje** | Python 3 |
| **Dependencias Principales** | `flask`, `pytest`, `jinja2.TemplateNotFound`, `werkzeug.routing.Rule` |
| **Fixtures Requeridas** | `app` (instancia de `Flask`), `client` (test client), `test_apps` (app de prueba `blueprintapp`) |

### 2. Descripción General

Este archivo valida de forma exhaustiva el comportamiento del objeto `flask.Blueprint`, componente central de Flask para la modularización de aplicaciones.

Su objetivo no es implementar lógica de negocio, sino **garantizar que el sistema de Blueprints funciona correctamente** en todos sus aspectos: aislamiento, enrutamiento, manejo de errores, integración con Jinja2, gestión de plantillas/archivos estáticos y composición jerárquica.

Actúa como una prueba de regresión para asegurar que cualquier cambio en el core de Flask no rompa el contrato de los Blueprints.

### 3. Componentes Principales

El archivo se estructura en 10 bloques funcionales que agrupan más de 35 casos de prueba:

#### 3.1. Manejo de Errores
Verifica la jerarquía y el alcance de los manejadores de excepciones.

*   `test_blueprint_specific_error_handling`: Comprueba que un `errorhandler(403)` definido en un Blueprint solo captura errores de sus propias rutas. Si la ruta no tiene handler propio, hace fallback al handler de la `app`.
*   `test_blueprint_specific_user_error_handling`: Valida el manejo de excepciones personalizadas mediante dos mecanismos: decorador `@blue.errorhandler(Exception)` y método `blue.register_error_handler()`.
*   `test_blueprint_app_error_handling`: Valida `@errors.app_errorhandler()` que permite a un Blueprint registrar un manejador de errores a nivel global de la aplicación, afectando tanto a rutas de la app como de otros Blueprints.

#### 3.2. Enrutamiento y Gestión de URLs
Valida la construcción y resolución de URLs.

*   `test_blueprint_prefix_slash`: Prueba parametrizada que asegura la correcta normalización de `url_prefix` y `rule` evitando dobles barras `//` o barras faltantes. Ej: `/foo/` + `/bar` = `/foo/bar`.
*   `test_blueprint_url_defaults`: Verifica `url_prefix` y `url_defaults` al registrar el mismo Blueprint dos veces con nombres y prefijos distintos `/1` y `/2`.
*   `test_empty_url_defaults`: Valida el uso de `defaults` en el decorador `@bp.route`.
*   `test_route_decorator_custom_endpoint` y `test_route_decorator_custom_endpoint_with_dots`: Comprueban la personalización del `endpoint` y la validación que impide usar puntos `.` en el nombre del endpoint.
*   `test_endpoint_decorator`: Valida el decorador `@bp.endpoint()` para asociar una función a un endpoint ya existente en `url_map`.
*   `test_dotted_names_from_app`: Verifica la generación de URLs con `url_for` entre la app y el Blueprint.

#### 3.3. Procesadores de URL
Valida la inyección y extracción de parámetros de la URL.

*   `test_blueprint_url_processors`: Usa `@bp.url_defaults` y `@bp.url_value_preprocessor` para implementar URLs multilenguaje `/<lang_code>/`. El preprocesador extrae `lang_code` a `flask.g` y el defaults lo reinyecta en `url_for`.
*   `test_app_url_processors`: Mismo caso pero con `@bp.app_url_defaults` y `@bp.app_url_value_preprocessor`, que aplican a nivel global de la aplicación.

#### 3.4. Plantillas y Archivos Estáticos
*   `test_templates_and_static`: Prueba de integración completa con la app `blueprintapp`. Valida renderizado de plantillas por Blueprint `frontend`/`admin`, servicio de archivos estáticos `/admin/static/`, control de caché `SEND_FILE_MAX_AGE_DEFAULT` y manejo de `TemplateNotFound`.
*   `test_templates_list`: Verifica que `app.jinja_env.list_templates()` lista correctamente las plantillas de todos los Blueprints.
*   `test_default_static_max_age`: Comprueba la personalización del caché de archivos estáticos sobreescribiendo `get_send_file_max_age()` en una subclase de Blueprint.

#### 3.5. Integración con Jinja2
Valida la extensión del entorno de plantillas desde un Blueprint.

*   **Filtros:** `test_template_filter`, `test_add_template_filter` y 6 variantes más. Prueban `@bp.app_template_filter()` y `bp.add_app_template_filter()` con y sin nombre personalizado, y su uso real dentro de una plantilla `template_filter.html`.
*   **Tests:** `test_template_test`, `test_add_template_test` y 6 variantes más. Prueban `@bp.app_template_test()` para registrar tests personalizados `is boolean` en Jinja2.
*   **Globales:** `test_template_global`: Prueba `@bp.app_template_global()` para exponer funciones Python como globales en todas las plantillas.
*   **Context Processors:** `test_context_processing`: Diferencia entre `@bp.app_context_processor` (global) y `@bp.context_processor` (solo para plantillas del Blueprint).

#### 3.6. Ciclo de Vida de la Petición
*   `test_request_processing`: Valida hooks a nivel de Blueprint: `@bp.before_request`, `@bp.after_request` y `@bp.teardown_request`.
*   `test_app_request_processing`: Valida hooks a nivel de aplicación registrados desde un Blueprint: `@bp.before_app_request`, `@bp.after_app_request` y `@bp.teardown_app_request`.

#### 3.7. Blueprints Anidados y Composición
*   `test_nested_blueprint`: Valida el registro jerárquico `grandchild -> child -> parent -> app` y la herencia de manejadores de errores.
*   `test_nested_callback_order`: Verifica el orden de ejecución de `before_request`, `teardown_request` y `context_processor` en una jerarquía anidada. El orden es `app -> parent -> child`.
*   `test_nesting_url_prefixes`: Prueba parametrizada que asegura que los `url_prefix` se concatenan correctamente sin importar si se definen en la creación del Blueprint o en `register_blueprint()`.
*   `test_nesting_subdomains` y `test_child_and_parent_subdomain`: Validan el enrutamiento por subdominios `subdomain_matching` y `SERVER_NAME` en Blueprints anidados.

#### 3.8. Validaciones y Restricciones
*   `test_dotted_name_not_allowed` y `test_empty_name_not_allowed`: Aseguran que el nombre del Blueprint no puede estar vacío ni contener puntos.
*   `test_unique_blueprint_names`: Valida que no se pueden registrar dos Blueprints con el mismo nombre, a menos que se use el parámetro `name` para renombrar.
*   `test_self_registration`: Impide que un Blueprint se registre a sí mismo.
*   `test_blueprint_renaming`: Valida el renombrado completo de un Blueprint y sus hijos al registrarlo con `name="alt"`, verificando que endpoints y handlers se renombran correctamente `bp.index` -> `alt.index`.

### 4. Conclusión

Este archivo es una **suite de pruebas de contrato** para el sistema de Blueprints de Flask. No contiene lógica de aplicación, sino que documenta de forma ejecutable el comportamiento esperado del framework.

Su mantenimiento es crítico para garantizar:
1.  **Aislamiento:** Cada Blueprint mantiene sus errores, plantillas y hooks sin contaminar a otros.
2.  **Reutilización:** Un mismo Blueprint puede registrarse múltiples veces con distintos prefijos y configuraciones.
3.  **Escalabilidad:** Soporte para arquitecturas modulares complejas mediante anidamiento y subdominios.

**Ejecución:**
```bash
pytest tests/test_blueprints.py -v
```