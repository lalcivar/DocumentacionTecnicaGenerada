# Documentación Técnica - Suite de Tests de Enrutamiento, Sesiones y Ciclo de Vida de Flask

## 1. Resumen Ejecutivo

Este módulo es una suite de tests de integración para el framework **Flask** que valida el comportamiento del núcleo de la aplicación: enrutamiento y despacho de peticiones, negociación de métodos HTTP y `OPTIONS` automático, gestión de sesiones y cookies, manejo de errores y `teardown`, serialización JSON y generación de URLs. Cubre casos de borde de `APPLICATION_ROOT`, `SERVER_NAME`, `subdomain_matching`/`host_matching`, `Blueprints`, `static_folder` y ciclo de vida `before_request`/`after_request`. Está diseñado para ejecutarse con `pytest` y fixtures `app`, `client`, `req_ctx` y `app_ctx`.

## 2. Especificación de Funciones y Métodos

> **Convención Docstring:** Google Style / PEP 257. Todas las funciones documentadas a continuación mantienen su nombre exacto. Las funciones anidadas se identifican como `funcion_padre.funcion_anidada()`.

### 2.1 Funciones de Nivel Superior (Tests)

#### `test_options_work(app, client)`
Verifica que una ruta `GET, POST` responda correctamente a `OPTIONS` con `Allow` y body vacío.

| Parámetro | Tipo | Requerido | Descripción |
|---|---|---|---|
| `app` | `flask.Flask` | Sí | Fixture de aplicación Flask. |
| `client` | `FlaskClient` | Sí | Cliente de test Werkzeug. |

| Retorna | Tipo | Descripción |
|---|---|---|
| `None` | `NoneType` | No retorna valor, aserciones internas. |

| Lanza | Condición |
|---|---|
| `AssertionError` | Si `Allow` o `data` no coinciden. |

**Docstring Google Style:**
```python
def test_options_work(app, client):
    """Valida respuesta OPTIONS automática.

    Args:
        app (flask.Flask): App de prueba.
        client (FlaskClient): Cliente de test.

    Returns:
        None:

    Raises:
        AssertionError: Si la cabecera Allow es incorrecta.
    """
```

#### `test_options_on_multiple_rules(app, client)`
Valida agregación de métodos `Allow` cuando existen múltiples reglas para la misma URL.

| Parámetro | Tipo | Requerido | Descripción |
|---|---|---|---|
| `app` | `flask.Flask` | Sí | App Flask. |
| `client` | `FlaskClient` | Sí | Cliente de test. |

#### `test_method_route(app, client, method)`
Parametrizado. Verifica decoradores de método `app.get`, `app.post`, `app.put`, etc.

| Parámetro | Tipo | Requerido | Descripción |
|---|---|---|---|
| `app` | `flask.Flask` | Sí | App Flask. |
| `client` | `FlaskClient` | Sí | Cliente. |
| `method` | `str` | Sí | Nombre del método HTTP (`get`, `post`, `put`, `delete`, `patch`, `query`). |

#### `test_method_route_no_methods(app)`
Verifica que `app.get(..., methods=[...])` lance `TypeError`.

| Lanza | Condición |
|---|---|
| `TypeError` | Si se pasa `methods` a un decorador de método específico. |

#### `test_provide_automatic_options_attr_disable(app: flask.Flask, client: FlaskClient) -> None`
Desactiva `OPTIONS` automático vía atributo `view_func.provide_automatic_options = False`. Espera `405`.

#### `test_provide_automatic_options_attr_enable(app: flask.Flask, client: FlaskClient) -> None`
Activa `OPTIONS` vía atributo cuando `PROVIDE_AUTOMATIC_OPTIONS=False` en config. Espera `Allow: GET, HEAD, OPTIONS`.

#### `test_provide_automatic_options_arg_disable(app: flask.Flask, client: FlaskClient) -> None`
Desactiva `OPTIONS` vía argumento `provide_automatic_options=False` en `@app.get`.

#### `test_provide_automatic_options_method_disable(app: flask.Flask, client: FlaskClient) -> None`
Verifica que si la ruta maneja explícitamente `OPTIONS`, no se sobrescribe la respuesta.

#### `test_request_dispatching(app, client)`
Valida despacho por método, `405` con `Allow`, y truncado de `HEAD`.

#### `test_disallow_string_for_allowed_methods(app)`
| Lanza | Condición |
|---|---|
| `TypeError` | Si `methods` es `str` en lugar de `list`. |

#### `test_url_mapping(app, client)`
Valida `add_url_rule` y que `methods=["options"]` en minúsculas no genere `OPTIONS` automático.

#### `test_werkzeug_routing(app, client)`
Valida integración con `werkzeug.routing.Submount` y `Rule`.

#### `test_endpoint_decorator(app, client)`
Valida `@app.endpoint("name")` para mapear endpoints a `Submount`.

#### `test_session_accessed(app: flask.Flask, client: FlaskClient) -> None`
Verifica flags `session.accessed`/`modified` y cabecera `Vary: Cookie`.

#### `test_session_path(app, client)`
Verifica `path=/foo` en `Set-Cookie` cuando `APPLICATION_ROOT="/foo"`.

#### `test_session_using_application_root(app, client)`
Verifica `SCRIPT_NAME` vía middleware `PrefixPathMiddleware` y `APPLICATION_ROOT`.

#### `test_session_using_session_settings(app, client)`
Valida `SESSION_COOKIE_DOMAIN`, `HTTPONLY`, `SECURE`, `PARTITIONED`, `SAMESITE`, `PATH`.

#### `test_session_using_samesite_attribute(app, client)`
| Lanza | Condición |
|---|---|
| `ValueError` | Si `SESSION_COOKIE_SAMESITE="invalid"`. |

#### `test_missing_session(app)`
Valida que sin `secret_key` el acceso a `session` lance `RuntimeError: session is unavailable`.

#### `test_session_secret_key_fallbacks(app, client) -> None`
Valida rotación con `SECRET_KEY_FALLBACKS`.

#### `test_session_expiration(app, client)`
Valida `Expires` para sesión permanente vs no permanente.

#### `test_session_stored_last(app, client)`
Valida que `after_request` que modifica sesión persista para la siguiente petición.

#### `test_session_special_types(app, client)`
Valida serialización de `tuple`, `bytes`, `Markup`, `uuid.UUID`, `datetime`, y claves con prefijos ` t`, ` di`.

#### `test_session_cookie_setting(app)`
Valida `SESSION_REFRESH_EACH_REQUEST` y `permanent` respecto a `Set-Cookie`.

#### `test_session_vary_cookie(app, client)`
Valida cabecera `Vary: Cookie` en accesos `get`, `getitem`, `setdefault`, `clear`.

#### `test_session_refresh_vary(app, client)`
Valida `Vary: Cookie` tras login con sesión permanente.

#### `test_flashes(app, req_ctx)`
Valida `flash()` y `get_flashed_messages()` y flag `session.modified`.

#### `test_extended_flashing(app)`
Valida categorías, `Markup`, y `category_filter`.

#### `test_request_processing(app, client)`
Valida orden `before_request` -> view -> `after_request`.

#### `test_request_preprocessing_early_return(app, client)`
Valida que `before_request` que retorna valor corta la cadena.

#### `test_after_request_processing(app, client)`
Valida `flask.after_this_request`.

#### `test_teardown_request_handler(app, client)` / `test_teardown_request_handler_debug_mode(app, client)`
Valida que `teardown_request` se ejecuta siempre y su retorno es ignorado.

#### `test_teardown_request_handler_error(app, client)`
Valida que `teardown_request` recibe la excepción original (`ZeroDivisionError`) aunque se lance `TypeError` interno. Espera `500`.

#### `test_before_after_request_order(app, client)`
Valida orden LIFO/FIFO: `before: 1,2`, `after: 3,4`, `teardown: 5,6`.

#### `test_error_handling(app, client)`
Valida `errorhandler(404,500,Forbidden)` y `abort()`.

#### `test_error_handling_processing(app, client)`
Valida que `after_request` se aplica también a respuestas de `errorhandler`.

#### `test_baseexception_error_handling(app, client)`
| Lanza | Condición |
|---|---|
| `KeyboardInterrupt` | No es capturado por `errorhandler`, se propaga. |

#### `test_before_request_and_routing_errors(app, client)`
Valida que `before_request` (`flask.g`) está disponible en `errorhandler(404)`.

#### `test_user_error_handling(app, client)`
Valida `errorhandler(MyException)`.

#### `test_http_error_subclass_handling(app, client)`
Valida precedencia de subclase `ForbiddenSubclass` vs `403`.

#### `test_errorhandler_precedence(app, client)`
Valida MRO: `E3(E1,E2)` resuelve a `E2` antes que `Exception`.

#### `test_trap_bad_request_key_error(app, client, debug, trap, expect_key, expect_abort)`
Parametrizado `debug`, `TRAP_BAD_REQUEST_ERRORS`. Valida `BadRequest` vs `KeyError`.

| Parámetro | Tipo | Requerido | Descripción |
|---|---|---|---|
| `debug` | `bool` | Sí | `DEBUG` config. |
| `trap` | `bool\|None` | Sí | `TRAP_BAD_REQUEST_ERRORS`. |
| `expect_key` | `bool` | Sí | Si debe retornar 400. |
| `expect_abort` | `bool` | Sí | Si `abort(400)` debe retornar 400. |

#### `test_trapping_of_all_http_exceptions(app, client)`
Valida `TRAP_HTTP_EXCEPTIONS=True` propaga `NotFound`.

#### `test_error_handler_after_processor_error(app, client)`
Valida `errorhandler(500)` cuando `before_request` o `after_request` lanzan `ZeroDivisionError`.

#### `test_enctype_debug_helper(app, client)`
| Lanza | Condición |
|---|---|
| `DebugFilesKeyError` | Si `request.files["foo"]` sin `enctype=multipart`. |

#### `test_response_types(app, client)`
Valida tipos de retorno: `str`, `bytes`, `tuple(status, headers)`, `Response`, `NotFound`, `dict`, `list` (JSON).

#### `test_response_type_errors()`
Valida `TypeError` para `None`, tupla corta/larga, `bool`, `callable` WSGI inválido.

#### `test_make_response(app, req_ctx)` / `test_make_response_with_response_instance(app, req_ctx)`
Valida `flask.make_response` con `str`, `status`, `generator`, `jsonify`, `Response`.

#### `test_jsonify_no_prettyprint(app, compact)` / `test_jsonify_mimetype(app, req_ctx)` / `test_json_dump_dataclass(app, req_ctx)` / `test_jsonify_args_and_kwargs_check(app, req_ctx)`
Validan `app.json.compact`, `mimetype`, serialización `dataclass`, y `TypeError` si `args` y `kwargs` simultáneos.

#### `test_url_generation(app, req_ctx)` / `test_build_error_handler(app)` / `test_build_error_handler_reraise(app)` / `test_url_for_passes_special_values_to_build_error_handler(app)`
Validan `url_for`, `BuildError` y `url_build_error_handlers`.

#### `test_static_files(app, client)` / `test_static_url_path()` / `test_static_url_path_with_ending_slash()` / `test_static_url_empty_path(app)` / `test_static_url_empty_path_default(app)` / `test_static_folder_with_pathlib_path(app)` / `test_static_folder_with_ending_slash()` / `test_static_route_with_host_matching()`
Validan `static_folder`, `static_url_path`, `Pathlib`, `host_matching` y `static_host`.

| Lanza | Condición |
|---|---|
| `AssertionError` | Si `static_host` sin `host_matching` o viceversa. |

#### `test_request_locals()`
Valida `flask.g` como `LocalProxy` no vinculado.

#### `test_server_name_matching(subdomain_matching: bool, host_matching: bool, expect_subdomain: str, expect_host: str) -> None`
Parametrizado. Valida `SERVER_NAME` con `subdomain_matching` y `host_matching`.

#### `test_server_name_subdomain()`
Valida rutas con `subdomain="foo"` y puertos `:5000`, `:443`.

#### `test_exception_propagation(app, client, key)`
Parametrizado `TESTING`, `PROPAGATE_EXCEPTIONS`, `DEBUG`. Valida propagación de `ZeroDivisionError`.

#### `test_werkzeug_passthrough_errors(monkeypatch, debug, use_debugger, use_reloader, propagate_exceptions, app)`
Valida `passthrough_errors` en `werkzeug.serving.run_simple` vía `monkeypatch`.

#### `test_url_processors(app, client)` / `test_inject_blueprint_url_defaults(app)`
Validan `url_defaults`, `url_value_preprocessor` y `inject_url_defaults`.

#### `test_nonascii_pathinfo(app, client)` / `test_no_setup_after_first_request(app, client)` / `test_routing_redirect_debugging(monkeypatch, app, client)` / `test_route_decorator_custom_endpoint(app, client)`
Validan path no-ASCII, bloqueo de `add_url_rule` tras primera petición, y `RequestRedirect` con `DEBUG`.

| Lanza | Condición |
|---|---|
| `AssertionError` | Si `add_url_rule` tras `first_request` o redirect 301/302 con `follow_redirects`. |

#### `test_get_method_on_g(app_ctx)` / `test_g_iteration_protocol(app_ctx)`
Validan `g.get()`, `in`, e iteración.

#### `test_subdomain_basic_support()` / `test_subdomain_matching()` / `test_subdomain_matching_with_ports()` / `test_subdomain_matching_other_name(matching)`
Validan `subdomain_matching=True` con `SERVER_NAME`.

#### `test_multi_route_rules(app, client)` / `test_multi_route_class_views(app, client)`
Validan múltiples `@app.route` sobre misma función/clase.

#### `test_run_defaults(monkeypatch, app)` / `test_run_server_port(monkeypatch, app)` / `test_run_from_config(monkeypatch, host, port, server_name, expect_host, expect_port, app)`
Validan `app.run()` y resolución de `host`/`port` desde `SERVER_NAME`.

#### `test_max_cookie_size(app, client, recwarn)`
Valida `MAX_COOKIE_SIZE` y `UserWarning: cookie is too large`.

#### `test_app_freed_on_zero_refcount()`
Valida que `Flask` no crea ciclo de referencia que impida GC en CPython. Usa `weakref` y `gc.disable()`.

| Lanza | Condición |
|---|---|
| `AssertionError` | Si `weakref` no es `None` tras `del app`. |

---

### 2.2 Funciones Anidadas, Callbacks y Métodos de Clase

Todas las funciones siguientes son locales y se documentan con su *scope* padre.

| Nombre Completo | Parámetros | Retorna | Descripción |
|---|---|---|---|
| `test_options_work.index()` | - | `str` | View `GET,POST /` retorna `Hello World`. |
| `test_options_on_multiple_rules.index()` / `index_put()` | - | `str` | Views para `GET,POST` y `PUT` en `/`. |
| `test_method_route.hello()` | - | `str` | View genérica para test parametrizado de métodos. |
| `test_provide_automatic_options_* .index()` | - | `str` | View con `provide_automatic_options` True/False. |
| `test_request_dispatching.index()` / `more()` | - | `str` | Retornan `request.method`. |
| `test_url_mapping.index()` / `more()` / `options()` | - | `str` | Views para `/`, `/more`, `/options` (minúsculas). |
| `test_werkzeug_routing.bar()` / `index()` | - | `str` | Views para `Submount /foo`. |
| `test_endpoint_decorator.bar()` / `index()` | - | `str` | Views registradas vía `@app.endpoint`. |
| `test_session_accessed.do_set()` / `do_get()` / `do_nothing() -> str` | - | `str` | Manipulan `session["value"]`. |
| `test_session_path.index()` / `test_session_using_application_root.index()` / `test_session_using_session_settings.index()` / `clear()` / `test_session_using_samesite_attribute.index()` | - | `str` | Setean `session["testing"]=42`. |
| `test_session_using_application_root.PrefixPathMiddleware.__init__(self, app, prefix)` | `app: WSGIApp`, `prefix: str` | `None` | Inicializa middleware. |
| `test_session_using_application_root.PrefixPathMiddleware.__call__(self, environ, start_response)` | `environ: dict`, `start_response: callable` | `iterable` | Inyecta `SCRIPT_NAME`. |
| `test_missing_session.expect_exception(f, *args, **kwargs)` | `f: callable`, `*args`, `**kwargs` | `None` | Helper que espera `RuntimeError: session is unavailable`. |
| `test_session_secret_key_fallbacks.set_session() -> str` / `get_session() -> dict` | - | `str`/`dict` | Set/get `session["a"]`. |
| `test_session_expiration.index()` / `test()` | - | `str` | Setea `session.permanent` y lo lee. |
| `test_session_stored_last.modify_session(response)` | `response: Response` | `Response` | `after_request` que setea `session["foo"]=42`. |
| `test_session_stored_last.dump_session_contents()` / `test_session_special_types.dump_session_contents()` | - | `str`/`tuple` | Escriben tipos especiales en sesión. |
| `test_session_cookie_setting.bump()` / `read()` | - | `str` | Incrementa/lee `session["foo"]`. |
| `test_session_cookie_setting.run_test(expect_header)` | `expect_header: bool` | `None` | Helper que verifica `Set-Cookie` según `SESSION_REFRESH_EACH_REQUEST`. |
| `test_session_vary_cookie.set_session()` / `get()` / `getitem()` / `setdefault()` / `clear()` / `vary_cookie_header_set()` / `vary_header_set()` / `no_vary_header()` | - | `str`/`Response` | Views para validar `Vary`. |
| `test_session_vary_cookie.expect(path, header_value="Cookie")` | `path: str`, `header_value: str\|None` | `None` | Helper de aserción de `Vary`. |
| `test_session_refresh_vary.login()` / `ignored()` | - | `str` | Login con `session.permanent=True`. |
| `test_extended_flashing.index()` / `test()` / `test_with_categories()` / `test_filter()` / `test_filters()` / `test_filters2()` | - | `str` | Flash y `get_flashed_messages` con filtros. |
| `test_request_processing.before_request()` / `after_request(response)` / `index()` | `response: Response` (solo after) | `None`/`Response`/`str` | Hooks de ciclo de vida. |
| `test_request_preprocessing_early_return.before_request1()` / `before_request2()` / `before_request3()` / `index()` | - | `str\|None` | `before_request2` retorna `hello` y corta cadena. |
| `test_after_request_processing.index()` | - | `str` | View que registra `after_this_request`. |
| `test_after_request_processing.index.foo(response)` | `response: Response` | `Response` | Callback `after_this_request` que añade `X-Foo`. |
| `test_teardown_request_handler.teardown_request(exc)` / `root()` | `exc: BaseException\|None` | `str` | `teardown_request` (retorno ignorado). |
| `test_teardown_request_handler_error.teardown_request1(exc)` / `teardown_request2(exc)` / `fails()` | `exc: ZeroDivisionError` | `None` | Validan que `exc` es `ZeroDivisionError` aunque se lance `TypeError` interno. |
| `test_before_after_request_order.before1()` / `before2()` / `after1(response)` / `after2(response)` / `finish1(exc)` / `finish2(exc)` / `index()` | - | `None`/`Response`/`str` | Validan orden 1,2,3,4,5,6. |
| `test_error_handling.not_found(e)` / `internal_server_error(e)` / `forbidden(e)` / `index()` / `error()` / `error2()` | `e: Exception` | `tuple[str,int]` | Handlers y views que hacen `abort`/`raise`. |
| `test_error_handling_processing.internal_server_error(e)` / `broken_func()` / `after_request(resp)` | `e: Exception`, `resp: Response` | `tuple`/`Response` | Handler 500 y `after_request` que cambia `mimetype`. |
| `test_baseexception_error_handling.broken_func()` | - | `None` | Lanza `KeyboardInterrupt`. |
| `test_before_request_and_routing_errors.attach_something()` / `return_something(error)` | `error: 404` | `tuple` | `before_request` setea `g.something`. |
| `test_user_error_handling.handle_my_exception(e)` / `index()` | `e: MyException` | `str` | Handler custom. |
| `test_http_error_subclass_handling.handle_forbidden_subclass(e)` / `handle_403(e)` / `index1()` / `index2()` / `index3()` | `e: Forbidden` | `str` | Handlers para subclase vs base. |
| `test_errorhandler_precedence.handle_e2(e)` / `handle_exception(e)` / `raise_e1()` / `raise_e3()` | `e: Exception` | `str` | Precedencia MRO. |
| `test_trap_bad_request_key_error.fail()` / `allow_abort()` | - | `None` | `fail` accede `request.form["missing_key"]`, `allow_abort` hace `abort(400)`. |
| `test_trapping_of_all_http_exceptions.fail()` | - | `None` | `abort(404)` con `TRAP_HTTP_EXCEPTIONS`. |
| `test_error_handler_after_processor_error.before_request()` / `after_request(response)` / `index()` / `internal_server_error(e)` | - | `str`/`Response` | Simulan error en `before`/`after`. |
| `test_enctype_debug_helper.index()` | - | `str` | Accede `request.files["foo"]`. |
| `test_response_types.from_text()` / `from_bytes()` / `from_full_tuple()` / `from_text_headers()` / `from_text_status()` / `from_response_headers()` / `from_response_status()` / `from_wsgi()` / `from_dict()` / `from_list()` | - | `str`/`bytes`/`tuple`/`Response`/`dict`/`list` | Cubren todos los tipos de retorno soportados por Flask. |
| `test_response_type_errors.from_none()` / `from_small_tuple()` / `from_large_tuple()` / `from_bad_type()` / `from_bad_wsgi()` | - | `None`/`tuple`/`bool`/`callable` | Casos que deben lanzar `TypeError`. |
| `test_url_generation.hello()` | - | `None` | View `POST /hello/<name>` para `url_for`. |
| `test_build_error_handler.handler(error, endpoint, values)` | `error: BuildError`, `endpoint: str`, `values: dict` | `str` | Handler custom que retorna `/test_handler/`. |
| `test_build_error_handler_reraise.handler_raises_build_error(error, endpoint, values)` | `error: BuildError`, `endpoint: str`, `values: dict` | `None` | Re-lanza `BuildError`. |
| `test_url_for_passes_special_values_to_build_error_handler.handler(error, endpoint, values)` | `error: BuildError`, `endpoint: str`, `values: dict` | `str` | Valida `values` contiene `_external`, `_anchor`, `_method`, `_scheme`. |
| `test_static_folder_with_ending_slash.catch_all(path)` | `path: str` | `str` | View catch-all. |
| `test_server_name_matching.index(name: str) -> str` | `name: str` | `str` | View con `host` y `subdomain` dinámicos. |
| `test_server_name_subdomain.index()` / `subdomain()` | - | `str` | Views default y `subdomain="foo"`. |
| `test_exception_propagation.index()` | - | `None` | Lanza `ZeroDivisionError`. |
| `test_werkzeug_passthrough_errors.run_simple_mock(*args, **kwargs)` / `test_run_defaults.run_simple_mock(*args, **kwargs)` / `test_run_server_port.run_simple_mock(hostname, port, application, *args, **kwargs)` / `test_run_from_config.run_simple_mock(hostname, port, *args, **kwargs)` | `*args`, `**kwargs` | `None` | Mocks de `werkzeug.serving.run_simple` que capturan `passthrough_errors`/`hostname`/`port`. |
| `test_url_processors.add_language_code(endpoint, values)` / `pull_lang_code(endpoint, values)` / `index()` / `about()` / `something_else()` | `endpoint: str`, `values: dict` | `None`/`str` | Procesadores de URL para `lang_code`. |
| `test_inject_blueprint_url_defaults.bp_defaults(endpoint, values)` / `view(page)` | `endpoint: str`, `values: dict`, `page: str` | `None` | Defaults de Blueprint. |
| `test_nonascii_pathinfo.index()` / `test_no_setup_after_first_request.index()` / `test_routing_redirect_debugging.user()` / `test_route_decorator_custom_endpoint.foo()` / `for_bar()` / `for_bar_foo()` | - | `str` | Views para paths no-ASCII, redirect y endpoints custom. |
| `test_subdomain_basic_support.normal_index()` / `test_index()` / `test_subdomain_matching.index(user)` / `test_subdomain_matching_with_ports.index(user)` / `test_subdomain_matching_other_name.index()` | `user: str` (cuando aplica) | `str` | Views con `subdomain`. |
| `test_multi_route_rules.index(test="a")` | `test: str` | `str` | View con múltiples reglas. |
| `test_multi_route_class_views.View.__init__(self, app)` | `app: flask.Flask` | `None` | Registra dos `add_url_rule` para `/` y `/<test>/`. |
| `test_multi_route_class_views.View.index(self, test="a")` | `test: str` | `str` | Método de vista de clase. |
| `test_max_cookie_size.index()` | - | `Response` | Setea cookie grande `bar*100`. |

## 3. Ejemplo de Uso

```python
import flask
from flask.testing import FlaskClient

# Crear app y registrar una vista con OPTIONS automático
app = flask.Flask(__name__)
app.config["SECRET_KEY"] = "dev"

@app.route("/", methods=["GET", "POST"])
def index():
    return "Hello World"

# Uso con test client (como en test_options_work y test_session_path)
client: FlaskClient = app.test_client()

# 1. Verificar OPTIONS automático
rv = client.open("/", method="OPTIONS")
assert sorted(rv.allow) == ["GET", "HEAD", "OPTIONS", "POST"]
assert rv.data == b""

# 2. Verificar sesión y Vary: Cookie (como en test_session_accessed)
@app.post("/login")
def login():
    flask.session["user_id"] = 42
    return "ok"

@app.get("/profile")
def profile():
    return str(flask.session.get("user_id", "anon"))

with client:
    rv = client.post("/login")
    assert "cookie" in rv.vary.lower()  # Vary: Cookie
    rv = client.get("/profile")
    assert rv.data == b"42"

# 3. Ejecutar la suite completa con pytest
# $ pytest tests/test_basic.py -k "test_options_work or test_session"
```

> **Nota:** Todas las funciones de test requieren fixtures `app` y `client` provistas por `pytest-flask`. Para reproducir `test_app_freed_on_zero_refcount`, ejecutar con `CPython` y `gc` habilitado.
