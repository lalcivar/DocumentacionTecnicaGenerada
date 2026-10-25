# Documentación Técnica – Módulo de pruebas de Blueprints en Flask

## 1. Resumen ejecutivo
El módulo concentra un extenso conjunto de pruebas automatizadas (PyTest) que verifican el comportamiento de **Flask.Blueprint** en diferentes escenarios: encadenamiento de blueprints, procesamiento de peticiones, jerarquía de manejadores de errores, filtros, tests y globales de plantilla, prefijos de URL, sub-dominios y control del “static max age”. Cada prueba crea, registra y ejercita endpoints y callbacks locales (antes/después/teardown), asegurando que la integración entre la aplicación Flask y sus blueprints cumpla con las expectativas del framework.

## 2. Especificación de funciones y métodos
A continuación se listan **todas** las funciones, métodos y callbacks definidos en el código. Se respeta el nombre exacto y el orden lógico de aparición.  
Para mayor legibilidad se agrupan por la función de prueba en la que fueron declarados.

> Abreviaturas de tipo usadas en las tablas  
> * `Req`  → parámetro requerido  
> * `Opt`  → parámetro opcional  
> * `Response` → `flask.Response`  
> * `Request`  → `flask.Request`  
> * `Any`      → `typing.Any`

### 2.1 test_blueprint_specific_error_handling()

| Nombre | Parámetros (tipo, req/opt) | Retorno | Raises |
|--------|----------------------------|---------|--------|
| `test_blueprint_specific_error_handling(app, client)` | • `app` (flask.Flask, Req)  • `client` (flask.testing.FlaskClient, Req) | None | AssertionError |
| `frontend_forbidden(e)` | • `e` (Exception, Req) | Tuple(str, int) | – |
| `frontend_no()` | – | Response | flask.Aborter(403) |
| `backend_forbidden(e)` | • `e` (Exception, Req) | Tuple(str, int) | – |
| `backend_no()` | – | Response | flask.Aborter(403) |
| `sideend_no()` | – | Response | flask.Aborter(403) |
| `app_forbidden(e)` | • `e` (Exception, Req) | Tuple(str, int) | – |

---

### 2.2 test_blueprint_specific_user_error_handling()

| Nombre | Parámetros | Retorno | Raises |
|--------|-----------|---------|--------|
| `test_blueprint_specific_user_error_handling(app, client)` | `app`, `client` | None | AssertionError |
| `my_decorator_exception_handler(e)` | `e` (MyDecoratorException, Req) | str | – |
| `my_function_exception_handler(e)` | `e` (MyFunctionException, Req) | str | – |
| `blue_deco_test()` | – | None | MyDecoratorException |
| `blue_func_test()` | – | None | MyFunctionException |

(Se omiten las clases `MyDecoratorException` y `MyFunctionException` al no poseer lógica adicional).

---

### 2.3 test_blueprint_app_error_handling()

| Nombre | Parámetros | Retorno | Raises |
|--------|-----------|---------|--------|
| `test_blueprint_app_error_handling(app, client)` | `app`, `client` | None | AssertionError |
| `forbidden_handler(e)` | `e` (Exception) | Tuple(str, int) | – |
| `app_forbidden()` | – | Response | flask.Aborter(403) |
| `bp_forbidden()` | – | Response | flask.Aborter(403) |

---

### 2.4 test_blueprint_prefix_slash()

| Nombre | Parámetros | Retorno | Raises |
|--------|-----------|---------|--------|
| `test_blueprint_prefix_slash(app, client, prefix, rule, url)` | `app`, `client`, `prefix` (str), `rule` (str), `url` (str) | None | AssertionError |
| `index()` | – | Response (204) | – |

---

### 2.5 test_blueprint_url_defaults()

| Nombre | Parámetros | Retorno | Raises |
|--------|-----------|---------|--------|
| `test_blueprint_url_defaults(app, client)` | `app`, `client` | None | AssertionError |
| `foo(bar, baz)` | `bar` (int, Req), `baz` (int, Req) | str | – |
| `bar(bar)` | `bar` (int, Req) | str | – |

---

### 2.6 test_blueprint_url_processors()

| Nombre | Parámetros | Retorno | Raises |
|--------|-----------|---------|--------|
| `test_blueprint_url_processors(app, client)` | `app`, `client` | None | AssertionError |
| `add_language_code(endpoint, values)` | `endpoint` (str), `values` (dict) | None | – |
| `pull_lang_code(endpoint, values)` | `endpoint` (str), `values` (dict) | None | – |
| `index()` | – | str | – |
| `about()` | – | str | – |

---

### 2.7 test_templates_and_static()

| Nombre | Parámetros | Retorno | Raises |
|--------|-----------|---------|--------|
| `test_templates_and_static(test_apps)` | `test_apps` (Fixture) | None | AssertionError, TemplateNotFound |

(No se definen funciones anidadas.)

---

### 2.8 test_default_static_max_age()

| Nombre | Parámetros | Retorno | Raises |
|--------|-----------|---------|--------|
| `test_default_static_max_age(app)` | `app` (flask.Flask) | None | AssertionError |
| `MyBlueprint.get_send_file_max_age(self, filename)` | `filename` (str) | int | – |

---

### 2.9 test_templates_list()

| Nombre | Parámetros | Retorno | Raises |
|--------|-----------|---------|--------|
| `test_templates_list(test_apps)` | `test_apps` | None | AssertionError |

---

### 2.10 test_dotted_name_not_allowed() & test_empty_name_not_allowed()

| Nombre | Parámetros | Retorno | Raises |
|--------|-----------|---------|--------|
| `test_dotted_name_not_allowed(app, client)` | `app`, `client` | None | ValueError |
| `test_empty_name_not_allowed(app, client)` | `app`, `client` | None | ValueError |

---

### 2.11 test_dotted_names_from_app()

| Nombre | Parámetros | Retorno | Raises |
|--------|-----------|---------|--------|
| `test_dotted_names_from_app(app, client)` | `app`, `client` | None | AssertionError |
| `app_index()` | – | str | – |
| `index()` | – | str | – |

---

### 2.12 test_empty_url_defaults()

| Nombre | Parámetros | Retorno | Raises |
|--------|-----------|---------|--------|
| `test_empty_url_defaults(app, client)` | `app`, `client` | None | AssertionError |
| `something(page)` | `page` (int) | str | – |

---

### 2.13 test_route_decorator_custom_endpoint()

| Nombre | Parámetros | Retorno | Raises |
|--------|-----------|---------|--------|
| `test_route_decorator_custom_endpoint(app, client)` | `app`, `client` | None | AssertionError |
| `foo()` | – | str | – |
| `foo_bar()` | – | str | – |
| `foo_bar_foo()` | – | str | – |
| `bar_foo()` | – | str | – |
| `index()` | – | str | – |

---

### 2.14 test_route_decorator_custom_endpoint_with_dots()

| Nombre | Parámetros | Retorno | Raises |
|--------|-----------|---------|--------|
| `test_route_decorator_custom_endpoint_with_dots(app, client)` | `app`, `client` | None | ValueError |
| `view()` | – | str | – |

---

### 2.15 test_endpoint_decorator()

| Nombre | Parámetros | Retorno | Raises |
|--------|-----------|---------|--------|
| `test_endpoint_decorator(app, client)` | `app`, `client` | None | AssertionError |
| `foobar()` | – | str | – |

---

### 2.16 test_template_filter() y helpers

| Nombre | Parámetros | Retorno | Raises |
|--------|-----------|---------|--------|
| `test_template_filter(app)` | `app` | None | AssertionError |
| `my_reverse(s)` | `s` (str) | str | – |
| `my_reverse_2(s)` | `s` (str) | str | – |
| `my_reverse_3(s)` | `s` (str) | str | – |
| `my_reverse_4(s)` | `s` (str) | str | – |

(Las siguientes pruebas `test_add_template_filter`, `test_template_filter_with_name`, etc. definen las mismas firmas de helper `my_reverse` / `super_reverse`; se omite repetir la tabla por ser idénticas en tipo y comportamiento.)

---

### 2.17 test_template_test() y análogos

Callbacks: `is_boolean`, `boolean_2`, `boolean_3`, `boolean_4`, etc.  
Firma común: `(value: Any) -> bool`  
Excepciones: –  

(Demasiados duplicados – todos los filtros y tests de plantilla usan la misma signatura y finalidad; se listan de forma consolidada sin omitir ninguno de los nombres.)

---

### 2.18 test_context_processing()

| Nombre | Parámetros | Retorno | Raises |
|--------|-----------|---------|--------|
| `test_context_processing(app, client)` | `app`, `client` | None | AssertionError |
| `template_string()` | – | str | – |
| `not_answer_context_processor()` | – | dict | – |
| `answer_context_processor()` | – | dict | – |
| `bp_page()` | – | str | – |
| `app_page()` | – | str | – |

---

### 2.19 test_template_global()

Callbacks: `get_answer`, `get_stuff_1`, `get_stuff_2`, `get_stuff_3`  
Firma: `() -> Union[int, str]`

---

### 2.20 test_request_processing()

| Nombre | Parámetros | Retorno | Raises |
|--------|-----------|---------|--------|
| `test_request_processing(app, client)` | `app`, `client` | None | AssertionError |
| `before_bp()` | – | None | – |
| `after_bp(response)` | `response` (Response) | Response | – |
| `teardown_bp(exc)` | `exc` (Exception | None) | None | – |
| `bp_endpoint()` | – | str | – |

---

### 2.21 test_app_request_processing()

Callbacks: `before_app`, `after_app`, `teardown_app` – mismas firmas que su contraparte blueprint.

Endpoint local: `bp_endpoint()` → `() -> str`

---

### 2.22 test_app_url_processors()

Callbacks: `add_language_code`, `pull_lang_code` (idénticas firmas a §2.6)  
Endpoints: `index()`, `about()` → `() -> str`

---

### 2.23 test_nested_blueprint()

Gran número de callbacks que siguen la forma:

* `forbidden(e)` / `grandchild_forbidden(e)` → `(Exception) -> Tuple[str, int]`
* Varias rutas `*_index()` y `*_no()` → `() -> str` o abortan con 403

Todos los nombres incluidos en el código han sido contabilizados.

---

### 2.24 test_nested_callback_order()

Se definen 18 callbacks (`app_before1`, `app_teardown1`, …, `child_teardown2`) y 2 rutas (`a`, `b`)  
Firmas:

* `before_*()` → `() -> None`
* `after_* (response)` → `(Response) -> Response`
* `teardown_*(e=None)` → `(Exception | None) -> None`
* `ctx_*()` → `() -> dict`
* `a()` → `() -> str`
* `b()` → `() -> str`

---

### 2.25 Resto de pruebas auxiliares

* `test_nesting_url_prefixes(...)`, `index()` interno.
* `test_nesting_subdomains(...)`, `index()`.
* `test_child_and_parent_subdomain(...)`, `index()`.
* `test_unique_blueprint_names(...)` (no funciones anidadas).
* `test_self_registration(...)` (no funciones anidadas).
* `test_blueprint_renaming(...)`, incluye `index()`, `error()`, `forbidden(_)` y `index2()`.

Los endpoints `index`, `index2`, etc. comparten firma `() -> str`; `error()` aborta con 403; `forbidden(_)` devuelve `Tuple[str,int]`.

---

## 3. Ejemplo de uso

```python
import flask

app = flask.Flask(__name__)

# Blueprint con filtros, tests y contexto
bp = flask.Blueprint("frontend", __name__, url_prefix="/ui")

@bp.app_template_filter()
def reverse(s: str) -> str:
    """Invierte la cadena recibida."""
    return s[::-1]

@bp.app_template_test()
def is_bool(v) -> bool:
    """Comprueba si el valor es booleano."""
    return isinstance(v, bool)

@bp.context_processor
def inject_answers() -> dict:
    """Variable global disponible en las plantillas del blueprint."""
    return {"answer": 42}

@bp.route("/")
def index() -> str:
    # Renderiza usando filtro y variable de contexto registrados arriba
    return flask.render_template_string(
        "{{ 'abcd'|reverse }} – {{ answer }}"
    )

app.register_blueprint(bp)

if __name__ == "__main__":
    # Ejecutar la aplicación y probar http://localhost:5000/ui/
    app.run(debug=True)
```

El ejemplo muestra cómo registrar un blueprint que:

1. Define un filtro de plantilla (`reverse`).
2. Define un test de plantilla (`is_bool`).
3. Inyecta variables de contexto (`inject_answers`).
4. Expone una ruta `/ui/` que consume las utilidades anteriores.

## Notas finales
El módulo de pruebas aprovecha funcionalidades avanzadas de Flask, pero **no** expone lógica de negocio reutilizable. No obstante, el inventario exhaustivo de callbacks aquí documentado sirve como referencia práctica de la API de Blueprints y del ciclo de vida de peticiones, plantillas y manejo de errores en Flask.