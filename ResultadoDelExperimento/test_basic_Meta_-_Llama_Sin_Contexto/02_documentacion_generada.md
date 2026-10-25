# Documentación Técnica - Suite de Pruebas del Core de Flask

**Archivo:** `test_basic.py` / `test_flask_core.py`
**Framework:** `pytest` + `Flask` + `Werkzeug`
**Tipo:** Suite de pruebas unitarias y de integración
**Propósito:** Validar el comportamiento central del framework Flask

### 1. Descripción General

Este archivo constituye la suite de pruebas principal del núcleo de Flask. Su objetivo es verificar de forma exhaustiva el correcto funcionamiento de los componentes fundamentales del framework: sistema de enrutamiento, gestión de peticiones y respuestas, manejo de sesiones y cookies, ciclo de vida de la petición, manejo de errores, generación de URLs y servicio de archivos estáticos.

No es código de aplicación, sino **código de validación**. Cada función `test_*` verifica un contrato específico de la API de Flask utilizando un cliente de pruebas `FlaskClient` y contextos de aplicación/petición.

El archivo contiene **más de 70 casos de prueba** y cubre tanto el comportamiento esperado como los casos de error y condiciones límite.

### 2. Dependencias Principales

| Librería | Uso en el archivo |
| :--- | :--- |
| `flask` | Framework bajo prueba. Se importan `Flask`, `request`, `session`, `g`, `flash`, `Response` |
| `pytest` | Runner de tests, aserciones y fixtures `app`, `client`, `req_ctx`, `app_ctx` |
| `werkzeug` | Dependencia base de Flask. Se prueban `routing.Rule`, `routing.Submount`, `exceptions`, `http.parse_date`, `serving.run_simple` |
| `markupsafe.Markup` | Validación de tipos seguros para HTML en sesión y flash |
| `gc`, `weakref`, `platform` | Prueba de gestión de memoria y recolección de basura en CPython |

### 3. Configuración Global

```python
require_cpython_gc = pytest.mark.skipif(...)
werkzeug_3_2 = importlib.metadata.version("werkzeug") >= "3.2."
```
*   `require_cpython_gc`: Marcador para ejecutar tests que dependen del recolector de basura de CPython solo en dicho intérprete.
*   `werkzeug_3_2`: Flag de compatibilidad para adaptar aserciones según cambios de comportamiento en Werkzeug 3.2+.

Los fixtures `app` y `client` inyectados en cada test proveen una instancia aislada de `Flask(__name__)` y su cliente de pruebas `app.test_client()`.

### 4. Componentes Principales

El archivo se organiza en 9 módulos funcionales:

#### 4.1. Enrutamiento y Métodos HTTP
Verifica el sistema de despacho de URLs y la negociación de métodos.

*   `test_options_work`, `test_options_on_multiple_rules`: Manejo automático del método `OPTIONS` y cabecera `Allow`.
*   `test_method_route`: Decoradores de atajo `@app.get`, `@app.post`, `@app.put`, etc.
*   `test_provide_automatic_options_*` (4 tests): Control fino de `provide_automatic_options` vía atributo de función, argumento de ruta y configuración `PROVIDE_AUTOMATIC_OPTIONS`.
*   `test_request_dispatching`, `test_url_mapping`: Despacho correcto, códigos `405 Method Not Allowed` y `HEAD` truncado.
*   `test_werkzeug_routing`, `test_endpoint_decorator`: Integración directa con `werkzeug.routing.Rule/Submount` y `@app.endpoint`.
*   `test_disallow_string_for_allowed_methods`, `test_no_setup_after_first_request`: Validación de errores de configuración.

#### 4.2. Gestión de Sesiones y Cookies
El bloque más extenso. Valida el sistema de sesiones firmadas del lado del cliente.

*   `test_session_accessed`, `test_session_cookie_setting`, `test_session_vary_cookie`, `test_session_refresh_vary`: Control de cabeceras `Vary: Cookie` y flags `accessed`/`modified`.
*   `test_session_path`, `test_session_using_application_root`, `test_session_using_session_settings`, `test_session_using_samesite_attribute`: Configuración de cookies `SESSION_COOKIE_DOMAIN`, `PATH`, `SECURE`, `HTTPONLY`, `SAMESITE`, `PARTITIONED` y `APPLICATION_ROOT`.
*   `test_session_expiration`, `test_session_stored_last`: Manejo de `permanent_session_lifetime` y persistencia al final del ciclo `after_request`.
*   `test_session_special_types`: Serialización de tipos complejos en sesión: `tuple`, `bytes`, `Markup`, `uuid`, `datetime`.
*   `test_session_secret_key_fallbacks`: Rotación de claves con `SECRET_KEY_FALLBACKS`.
*   `test_missing_session`, `test_max_cookie_size`: Casos de error sin `secret_key` y advertencia por `MAX_COOKIE_SIZE`.

#### 4.3. Sistema de Mensajes Flash
*   `test_flashes`, `test_extended_flashing`: Pruebas de `flask.flash()` y `get_flashed_messages()` con categorías, filtros `category_filter` y preservación de `Markup`.

#### 4.4. Ciclo de Vida de la Petición
Valida los hooks del pipeline de Flask.

*   `test_request_processing`, `test_before_after_request_order`: Orden de ejecución `before_request` -> vista -> `after_request` -> `teardown_request`.
*   `test_request_preprocessing_early_return`: Retorno anticipado desde `before_request` que interrumpe la cadena.
*   `test_after_request_processing`: Decorador `after_this_request` a nivel de vista.
*   `test_teardown_request_handler*` (3 tests): Ejecución garantizada de `teardown_request` incluso con excepciones y en modo `debug`.
*   `test_url_processors`, `test_inject_blueprint_url_defaults`: Preprocesadores `url_value_preprocessor` y `url_defaults` para internacionalización y Blueprints.

#### 4.5. Manejo de Errores y Excepciones
*   `test_error_handling`, `test_error_handling_processing`, `test_user_error_handling`: Registro de `@app.errorhandler` para códigos HTTP y excepciones personalizadas.
*   `test_http_error_subclass_handling`, `test_errorhandler_precedence`: Resolución por MRO y precedencia de subclases de `HTTPException`.
*   `test_trap_bad_request_key_error`, `test_trapping_of_all_http_exceptions`: Configuraciones `TRAP_BAD_REQUEST_ERRORS` y `TRAP_HTTP_EXCEPTIONS` vs `DEBUG`.
*   `test_exception_propagation`, `test_baseexception_error_handling`: Propagación de excepciones según `TESTING`, `PROPAGATE_EXCEPTIONS` y `BaseException`.
*   `test_before_request_and_routing_errors`, `test_error_handler_after_processor_error`: Interacción entre errores en hooks y manejadores.

#### 4.6. Tipos de Respuesta y Serialización
*   `test_response_types`, `test_response_type_errors`: Conversión automática de valores de retorno: `str`, `bytes`, `tuple (body, status, headers)`, `Response`, `dict/list` -> JSON, `WSGI app`. Validación de `TypeError` para retornos inválidos.
*   `test_make_response*`, `test_jsonify_*`, `test_json_dump_dataclass`: Helpers `make_response`, `jsonify`, configuración `app.json.compact` y `mimetype`.

#### 4.7. Generación de URLs y Archivos Estáticos
*   `test_url_generation`, `test_build_error_handler*`: `url_for`, manejo de `BuildError` y `url_build_error_handlers`.
*   `test_static_files`, `test_static_url_path*` (6 tests): Servicio de `/static/<path>`, configuración `static_url_path`, `static_folder` con `pathlib.Path` y `host_matching` + `static_host`.

#### 4.8. Subdominios y Host Matching
*   `test_server_name_matching`, `test_server_name_subdomain`, `test_subdomain_*` (4 tests): Validación de `SERVER_NAME`, `subdomain_matching=True` y `host_matching=True` con diferentes puertos y esquemas.

#### 4.9. Configuración y Ejecución del Servidor
*   `test_run_defaults`, `test_run_server_port`, `test_run_from_config`: Mock de `werkzeug.serving.run_simple` para verificar parámetros `host`, `port` y `SERVER_NAME`.
*   `test_werkzeug_passthrough_errors`: Propagación de `passthrough_errors` según `debug`, `use_debugger`, `use_reloader`.
*   `test_app_freed_on_zero_refcount`: Prueba de ausencia de ciclos de referencia que impidan liberar la app en CPython.

### 5. Conclusión

Este archivo es una **prueba de regresión integral** del core de Flask. Garantiza que cualquier cambio en el framework no rompa los contratos fundamentales de enrutamiento, sesiones, ciclo de petición y manejo de errores. Su ejecución exitosa es requisito para validar la estabilidad de una nueva versión de Flask.
