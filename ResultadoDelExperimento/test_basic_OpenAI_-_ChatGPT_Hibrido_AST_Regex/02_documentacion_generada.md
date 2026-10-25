# Documentación Técnica – Suite de Pruebas de Flask  
*(Formato PEP-257 / Google Style)*  

---

## 1. Resumen ejecutivo
Este módulo contiene una batería exhaustiva de pruebas **pytest** que valida el comportamiento del micro-framework **Flask** en situaciones de ruteo, sesiones, generación de URL, manejo de errores, `jsonify`, archivos estáticos y utilidades de servidor.  
A través de un cliente de prueba (`flask.testing.FlaskClient`) se crean aplicaciones aisladas, se inyectan rutas en tiempo de ejecución y se afirman las respuestas HTTP y los efectos colaterales sobre el contexto de aplicación.

---

## 2. Especificación de funciones y métodos  
Todas las definiciones de nivel superior disponibles en el código fuente se listan a continuación. Cada prueba acepta objetos **fixture** de `pytest` (por ejemplo, `app`, `client`, `req_ctx`, `monkeypatch`, `recwarn`) provistos por la configuración de pruebas; dichos argumentos se marcan como “requerido” (pytest los inyecta automáticamente).  
Las funciones internas creadas dentro de cada prueba (por ejemplo `index()`, `hello()`, `bump()`, etc.) son _callbacks_ que implementan vistas, procesadores, manejadores de error o middlewares. Estas siguen siempre el contrato estándar de Flask (`Callable[..., flask.Response | str | tuple]`) y se omiten en la tabla por su gran número y mínima variabilidad; no obstante, cada una cumple:

```
def <internal>(...) -> flask.typing.ResponseReturnValue
```

Sin efectos laterales fuera del alcance de la prueba que la contiene.

### Leyenda de tablas
• **Req.** – ¿El parámetro es obligatorio para el llamador?  
• **Tipo** – Hint de tipo según `typing` o clases de Flask/Werkzeug.  
• **Devuelve** – Valor o estructura devuelta por la función.  
• **Raises** – Excepciones esperadas que la prueba valida.

> Nota: **Todas** las pruebas devuelven `None` (el valor implícito de Python) y su objetivo es hacer aserciones; sin embargo, el framework de pruebas puede propagar o atrapar excepciones según el caso de uso.

---

### Funciones de prueba

| Función | Parámetros (nombre / tipo / Req.) | Devuelve | Raises validada / esperada |
|---------|-----------------------------------|----------|----------------------------|
| `test_options_work` | `app: flask.Flask ✓`<br>`client: FlaskClient ✓` | `None` | – |
| `test_options_on_multiple_rules` | idem | `None` | – |
| `test_method_route` | `app ✓`, `client ✓`, `method: str ✓` | `None` | – |
| `test_method_route_no_methods` | `app ✓` | `None` | `TypeError` |
| `test_provide_automatic_options_attr_disable` | `app ✓`, `client ✓` | `None` | – |
| `test_provide_automatic_options_attr_enable` | `app ✓`, `client ✓` | `None` | – |
| `test_provide_automatic_options_arg_disable` | `app ✓`, `client ✓` | `None` | – |
| `test_provide_automatic_options_method_disable` | `app ✓`, `client ✓` | `None` | – |
| `test_request_dispatching` | `app ✓`, `client ✓` | `None` | – |
| `test_disallow_string_for_allowed_methods` | `app ✓` | `None` | `TypeError` |
| `test_url_mapping` | `app ✓`, `client ✓` | `None` | – |
| `test_werkzeug_routing` | `app ✓`, `client ✓` | `None` | – |
| `test_endpoint_decorator` | `app ✓`, `client ✓` | `None` | – |
| `test_session_accessed` | `app ✓`, `client ✓` | `None` | – |
| `test_session_path` | `app ✓`, `client ✓` | `None` | – |
| `test_session_using_application_root` | `app ✓`, `client ✓` | `None` | – |
| `test_session_using_session_settings` | `app ✓`, `client ✓` | `None` | – |
| `test_session_using_samesite_attribute` | `app ✓`, `client ✓` | `None` | `ValueError` |
| `test_missing_session` | `app ✓` | `None` | `RuntimeError` |
| `test_session_secret_key_fallbacks` | `app ✓`, `client ✓` | `None` | – |
| `test_session_expiration` | `app ✓`, `client ✓` | `None` | – |
| `test_session_stored_last` | `app ✓`, `client ✓` | `None` | – |
| `test_session_special_types` | `app ✓`, `client ✓` | `None` | – |
| `test_session_cookie_setting` | `app ✓` | `None` | – |
| `test_session_vary_cookie` | `app ✓`, `client ✓` | `None` | – |
| `test_session_refresh_vary` | `app ✓`, `client ✓` | `None` | – |
| `test_flashes` | `app ✓`, `req_ctx ✓` | `None` | – |
| `test_extended_flashing` | `app ✓` | `None` | – |
| `test_request_processing` | `app ✓`, `client ✓` | `None` | – |
| `test_request_preprocessing_early_return` | `app ✓`, `client ✓` | `None` | – |
| `test_after_request_processing` | `app ✓`, `client ✓` | `None` | – |
| `test_teardown_request_handler` | `app ✓`, `client ✓` | `None` | – |
| `test_teardown_request_handler_debug_mode` | idem | `None` | – |
| `test_teardown_request_handler_error` | idem | `None` | – |
| `test_before_after_request_order` | idem | `None` | – |
| `test_error_handling` | idem | `None` | – |
| `test_error_handling_processing` | idem | `None` | – |
| `test_baseexception_error_handling` | idem | `None` | `KeyboardInterrupt` |
| `test_before_request_and_routing_errors` | idem | `None` | – |
| `test_user_error_handling` | idem | `None` | – |
| `test_http_error_subclass_handling` | idem | `None` | – |
| `test_errorhandler_precedence` | idem | `None` | – |
| `test_trap_bad_request_key_error` | `app ✓`, `client ✓`, `debug: bool ✓`, `trap: bool|None ✓`, `expect_key: bool ✓`, `expect_abort: bool ✓` | `None` | `KeyError`, `BadRequest` |
| `test_trapping_of_all_http_exceptions` | `app ✓`, `client ✓` | `None` | `NotFound` |
| `test_error_handler_after_processor_error` | idem | `None` | – |
| `test_enctype_debug_helper` | idem | `None` | `flask.debughelpers.DebugFilesKeyError` |
| `test_response_types` | idem | `None` | – |
| `test_response_type_errors` | (*crea su propio app y client*) | `None` | `TypeError` |
| `test_make_response` | `app ✓`, `req_ctx ✓` | `None` | – |
| `test_make_response_with_response_instance` | idem | `None` | – |
| `test_jsonify_no_prettyprint` | `app ✓`, `compact: bool ✓` | `None` | – |
| `test_jsonify_mimetype` | `app ✓`, `req_ctx ✓` | `None` | – |
| `test_json_dump_dataclass` | idem | `None` | – |
| `test_jsonify_args_and_kwargs_check` | idem | `None` | `TypeError` |
| `test_url_generation` | idem | `None` | – |
| `test_build_error_handler` | `app ✓` | `None` | `BuildError` |
| `test_build_error_handler_reraise` | `app ✓` | `None` | `BuildError` |
| `test_url_for_passes_special_values_to_build_error_handler` | `app ✓` | `None` | – |
| `test_static_files` | `app ✓`, `client ✓` | `None` | – |
| `test_static_url_path` | – | `None` | – |
| `test_static_url_path_with_ending_slash` | – | `None` | – |
| `test_static_url_empty_path` | – | `None` | – |
| `test_static_url_empty_path_default` | – | `None` | – |
| `test_static_folder_with_pathlib_path` | – | `None` | – |
| `test_static_folder_with_ending_slash` | – | `None` | – |
| `test_static_route_with_host_matching` | – | `None` | `AssertionError` |
| `test_request_locals` | – | `None` | – |
| `test_server_name_matching` | (*parámetros parametrizados*) | `None` | – |
| `test_server_name_subdomain` | – | `None` | – |
| `test_exception_propagation` | `app ✓`, `client ✓`, `key: str|None ✓` | `None` | `ZeroDivisionError` |
| `test_werkzeug_passthrough_errors` | `monkeypatch ✓`, `debug: bool ✓`, `use_debugger: bool ✓`, `use_reloader: bool ✓`, `propagate_exceptions: bool|None ✓`, `app ✓` | `None` | – |
| `test_url_processors` | `app ✓`, `client ✓` | `None` | – |
| `test_inject_blueprint_url_defaults` | `app ✓` | `None` | – |
| `test_nonascii_pathinfo` | `app ✓`, `client ✓` | `None` | – |
| `test_no_setup_after_first_request` | `app ✓`, `client ✓` | `None` | `AssertionError` |
| `test_routing_redirect_debugging` | `monkeypatch ✓`, `app ✓`, `client ✓` | `None` | `AssertionError` |
| `test_route_decorator_custom_endpoint` | `app ✓`, `client ✓` | `None` | – |
| `test_get_method_on_g` | `app_ctx ✓` | `None` | – |
| `test_g_iteration_protocol` | `app_ctx ✓` | `None` | – |
| `test_subdomain_basic_support` | – | `None` | – |
| `test_subdomain_matching` | – | `None` | – |
| `test_subdomain_matching_with_ports` | – | `None` | – |
| `test_subdomain_matching_other_name` | `matching: bool ✓` | `None` | – |
| `test_multi_route_rules` | `app ✓`, `client ✓` | `None` | – |
| `test_multi_route_class_views` | idem | `None` | – |
| `test_run_defaults` | `monkeypatch ✓`, `app ✓` | `None` | – |
| `test_run_server_port` | `monkeypatch ✓`, `app ✓` | `None` | – |
| `test_run_from_config` | `monkeypatch ✓`, `host: str|None ✓`, `port: int|None ✓`, `server_name: str ✓`, `expect_host: str ✓`, `expect_port: int ✓`, `app ✓` | `None` | – |
| `test_max_cookie_size` | `app ✓`, `client ✓`, `recwarn ✓` | `None` | – |
| `test_app_freed_on_zero_refcount` | – | `None` | – |

*(Las pruebas marcadas con “–” en parámetros crean su propia aplicación local o no requieren fixtures explícitos —el `pytest` las inyecta automáticamente).*

---

## 3. Ejemplo de uso  

El siguiente fragmento demuestra cómo un desarrollador podría reproducir, fuera del entorno de pruebas, parte de la lógica cubierta por `test_response_types()` para construir diversas respuestas:

```python
import flask
from markupsafe import Markup
from werkzeug.exceptions import NotFound

app = flask.Flask(__name__)

@app.route("/text")
def as_plain_text() -> str:
    return "¡Hola, mundo!"

@app.route("/bytes")
def as_bytes() -> bytes:
    return b"Byte-payload"

@app.route("/tuple_full")
def tuple_full() -> tuple[str, int, dict[str, str]]:
    return (
        Markup("<h1>Mensaje</h1>"),      # cuerpo
        202,                             # código HTTP
        {"X-Custom": "Demo"}             # cabeceras
    )

@app.route("/wsgi")
def raises_wsgi() -> NotFound:
    # Retornar directamente una instancia de HTTPException es
    # totalmente válido y será convertida en una respuesta.
    return NotFound(description="Recurso no encontrado")

if __name__ == "__main__":
    # Ejecución de desarrollo sobre 127.0.0.1:5000
    app.run(debug=True)
```

Al solicitar cada ruta:
```
$ curl http://localhost:5000/text
¡Hola, mundo!

$ curl -i http://localhost:5000/tuple_full
HTTP/1.0 202 ACCEPTED
X-Custom: Demo
Content-Type: text/html; charset=utf-8
...
<h1>Mensaje</h1>

$ curl -i http://localhost:5000/wsgi
HTTP/1.0 404 NOT FOUND
Content-Type: text/html; charset=utf-8
...
<!doctype html>...
```

Este pequeño servicio reproduce variantes de retornos que las pruebas validan:  
• cadenas Unicode → respuesta `text/html` por defecto.  
• bytes → cuerpo binario sin recodificar.  
• tupla completa *(body, status, headers)* → control granular.  
• instancia `werkzeug.exceptions.HTTPException` → respuesta WSGI lista.

---

### Conclusión
La batería de pruebas aquí documentada cubre el espectro completo de funcionalidades esenciales de Flask y puede servir como referencia de comportamiento esperado al extender el framework o al implementar aplicaciones sobre él.