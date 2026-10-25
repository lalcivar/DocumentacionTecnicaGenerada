# Documentación Técnica - Suite de Tests para `flask.Blueprint`

## 1. Resumen Ejecutivo

Este módulo es una suite de tests de integración para el sistema de `Blueprints` de Flask. Valida de forma exhaustiva el aislamiento y la composición de blueprints, incluyendo el manejo de errores específico por blueprint (`errorhandler` vs `app_errorhandler`), prefijos de URL, procesadores de contexto, filtros/tests/globals de Jinja2, hooks del ciclo de vida de la petición (`before_request`, `after_request`, `teardown_request`) y el anidamiento jerárquico con herencia de subdominios y prefijos.

El código no expone lógica de negocio propia, sino que sirve como especificación ejecutable del contrato de `flask.Blueprint` y su interacción con la instancia `Flask` mediante `pytest` y el cliente de pruebas de Flask.

---

## 2. Especificación de Funciones y Métodos

> Todas las funciones siguen el estándar **PEP 257 / Google Style Docstring**. Se documentan funciones de nivel superior y **todas las funciones anidadas/locales**, callbacks y métodos de clases internas.

### 2.1 `test_blueprint_specific_error_handling()`

```python
def test_blueprint_specific_error_handling(app, client):
    """Verifica el manejo de errores 403 aislado por Blueprint.

    Cada Blueprint puede definir su propio handler para un código HTTP
    sin afectar a otros Blueprints ni a la app global.

    Args:
        app (flask.Flask): Fixture de la aplicación Flask.
        client (flask.testing.FlaskClient): Cliente de pruebas.

    Returns:
        None: Las aserciones validan el comportamiento. No retorna valor.

    Raises:
        AssertionError: Si la respuesta no coincide con el handler esperado.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
|---|---|---|---|
| `app` | `flask.Flask` | Sí | Instancia de la aplicación donde se registran los blueprints |
| `client` | `FlaskClient` | Sí | Cliente para realizar peticiones GET |

#### 2.1.1 `frontend_forbidden()`
```python
def frontend_forbidden(e):
    """Handler 403 específico del Blueprint frontend.

    Args:
        e (HTTPException): Excepción 403 capturada.

    Returns:
        tuple[str, int]: Cuerpo "frontend says no" y código 403.
    """
```
| Parámetro | Tipo | Requerido | Descripción |
|---|---|---|---|
| `e` | `Exception` | Sí | Error 403 |

#### 2.1.2 `frontend_no()`
```python
def frontend_no():
    """Ruta /frontend-no que fuerza un abort(403).

    Returns:
        None: Aborta la petición con 403.
    Raises:
        HTTPException: 403 vía flask.abort.
    """
```

#### 2.1.3 `backend_forbidden()`
Idéntica a `frontend_forbidden()` pero retorna `b"backend says no"`, 403.

#### 2.1.4 `backend_no()`
Idéntica a `frontend_no()` para la ruta `/backend-no`.

#### 2.1.5 `sideend_no()`
```python
def sideend_no():
    """Ruta /what-is-a-sideend sin handler propio. Debe caer al handler global."""
```

#### 2.1.6 `app_forbidden()`
```python
def app_forbidden(e):
    """Handler 403 global de la aplicación.

    Args:
        e (HTTPException): Error 403.
    Returns:
        tuple[str, int]: "application itself says no", 403.
    """
```

### 2.2 `test_blueprint_specific_user_error_handling()`

```python
def test_blueprint_specific_user_error_handling(app, client):
    """Valida el registro de handlers para excepciones custom vía decorador y vía método.

    Args:
        app (flask.Flask): App fixture.
        client (FlaskClient): Cliente de pruebas.
    Returns:
        None
    """
```

#### 2.2.1 `my_decorator_exception_handler()`
```python
def my_decorator_exception_handler(e):
    """Handler registrado con @blue.errorhandler para MyDecoratorException.

    Args:
        e (MyDecoratorException): Excepción custom.
    Returns:
        str: "boom"
    """
```

#### 2.2.2 `my_function_exception_handler()`
```python
def my_function_exception_handler(e):
    """Handler registrado con blue.register_error_handler para MyFunctionException.

    Args:
        e (MyFunctionException): Excepción custom.
    Returns:
        str: "bam"
    """
```

#### 2.2.3 `blue_deco_test()`
```python
def blue_deco_test():
    """Ruta /decorator que lanza MyDecoratorException. Raises: MyDecoratorException"""
```

#### 2.2.4 `blue_func_test()`
```python
def blue_func_test():
    """Ruta /function que lanza MyFunctionException. Raises: MyFunctionException"""
```

### 2.3 `test_blueprint_app_error_handling()`

```python
def test_blueprint_app_error_handling(app, client):
    """Verifica que @errors.app_errorhandler registra un handler global desde un Blueprint.

    Args:
        app (flask.Flask): App fixture.
        client (FlaskClient): Cliente.
    Returns:
        None
    """
```

| Anidada | Firma | Descripción |
|---|---|---|
| `forbidden_handler(e)` | `(e: Exception) -> tuple[str, int]` | Retorna `"you shall not pass", 403` para toda la app |
| `app_forbidden()` | `() -> None` | Ruta `/forbidden` que hace `abort(403)` |
| `bp_forbidden()` | `() -> None` | Ruta `/nope` en `forbidden_bp` que hace `abort(403)` |

### 2.4 `test_blueprint_prefix_slash()`

```python
@pytest.mark.parametrize(("prefix", "rule", "url"), (...))
def test_blueprint_prefix_slash(app, client, prefix, rule, url):
    """Valida la normalización de slashes al combinar url_prefix y rule.

    Args:
        app (flask.Flask): App fixture.
        client (FlaskClient): Cliente.
        prefix (str): Prefijo del Blueprint.
        rule (str): Regla de la ruta.
        url (str): URL final esperada que debe responder 204.
    Returns:
        None
    """
```

#### 2.4.1 `index()`
```python
def index():
    """Endpoint de prueba. Returns: tuple[str, int]: ("", 204)"""
```

### 2.5 `test_blueprint_url_defaults()`

```python
def test_blueprint_url_defaults(app, client):
    """Valida url_defaults y defaults de ruta en registros múltiples del mismo Blueprint.

    Args:
        app (flask.Flask): App fixture.
        client (FlaskClient): Cliente.
    Returns:
        None
    """
```

| Anidada | Firma | Descripción |
|---|---|---|
| `foo(bar, baz)` | `(bar: Any, baz: int) -> str` | Ruta `/foo` con `defaults={"baz": 42}`. Retorna `f"{bar}/{baz}"` |
| `bar(bar)` | `(bar: Any) -> str` | Ruta `/bar`. Retorna `str(bar)` |

### 2.6 `test_blueprint_url_processors()`

```python
def test_blueprint_url_processors(app, client):
    """Valida url_defaults y url_value_preprocessor a nivel Blueprint con prefijo <lang_code>.

    Args:
        app (flask.Flask): App fixture.
        client (FlaskClient): Cliente.
    Returns:
        None
    """
```

| Anidada | Firma | Descripción |
|---|---|---|
| `add_language_code(endpoint, values)` | `(endpoint: str, values: dict) -> None` | Inyecta `lang_code` desde `g.lang_code` si no existe |
| `pull_lang_code(endpoint, values)` | `(endpoint: str, values: dict) -> None` | Extrae `lang_code` de `values` y lo guarda en `g.lang_code` |
| `index()` | `() -> str` | Ruta `/` retorna `url_for(".about")` |
| `about()` | `() -> str` | Ruta `/about` retorna `url_for(".index")` |

### 2.7 `test_templates_and_static()`

```python
def test_templates_and_static(test_apps):
    """Valida resolución de templates, static files y cache_control por Blueprint.

    Args:
        test_apps (fixture): Fixture que expone blueprintapp.app.
    Returns:
        None
    Raises:
        TemplateNotFound: Si se intenta renderizar missing.html.
        AssertionError: Si el contenido estático o templates no coinciden.
    """
```
*No contiene funciones anidadas con nombre; usa `client.get` y `test_request_context`.*

### 2.8 `test_default_static_max_age()`

```python
def test_default_static_max_age(app: flask.Flask) -> None:
    """Valida override de get_send_file_max_age en subclase de Blueprint.

    Args:
        app (flask.Flask): App tipada.
    Returns:
        None
    """
```

#### 2.8.1 `MyBlueprint.get_send_file_max_age()`
```python
def get_send_file_max_age(self, filename):
    """Retorna max_age custom para archivos estáticos.

    Args:
        filename (str | None): Nombre del archivo.
    Returns:
        int: 100 segundos.
    """
```
| Parámetro | Tipo | Requerido | Descripción |
|---|---|---|---|
| `self` | `MyBlueprint` | Sí | Instancia del Blueprint |
| `filename` | `str` | No | Nombre del archivo estático |

### 2.9 `test_templates_list()`

```python
def test_templates_list(test_apps):
    """Verifica que app.jinja_env.list_templates() lista templates de blueprints.

    Args:
        test_apps: Fixture de apps.
    Returns:
        None
    """
```

### 2.10 `test_dotted_name_not_allowed()`

```python
def test_dotted_name_not_allowed(app, client):
    """Verifica que Blueprint con nombre con punto lanza ValueError.

    Args:
        app (flask.Flask): App fixture.
        client (FlaskClient): Cliente.
    Raises:
        ValueError: Esperado al crear Blueprint("app.ui", ...).
    Returns:
        None
    """
```

### 2.11 `test_empty_name_not_allowed()`

```python
def test_empty_name_not_allowed(app, client):
    """Verifica que Blueprint con nombre vacío lanza ValueError.

    Args:
        app (flask.Flask): App fixture.
        client (FlaskClient): Cliente.
    Raises:
        ValueError: Nombre vacío no permitido.
    Returns:
        None
    """
```

### 2.12 `test_dotted_names_from_app()`

```python
def test_dotted_names_from_app(app, client):
    """Valida url_for cruzado entre app y blueprint.

    Args:
        app (flask.Flask): App fixture.
        client (FlaskClient): Cliente.
    Returns:
        None
    """
```

| Anidada | Firma | Descripción |
|---|---|---|
| `app_index()` | `() -> str` | Ruta `/` retorna `url_for("test.index")` |
| `index()` | `() -> str` | Ruta `/test/` en blueprint retorna `url_for("app_index")` |

### 2.13 `test_empty_url_defaults()`

```python
def test_empty_url_defaults(app, client):
    """Valida defaults en rutas con y sin parámetro.

    Args:
        app (flask.Flask): App fixture.
        client (FlaskClient): Cliente.
    Returns:
        None
    """
```

#### 2.13.1 `something()`
```python
def something(page):
    """Endpoint con defaults page=1.

    Args:
        page (int): Número de página.
    Returns:
        str: str(page)
    """
```

### 2.14 `test_route_decorator_custom_endpoint()`

```python
def test_route_decorator_custom_endpoint(app, client):
    """Valida el parámetro endpoint custom en @bp.route.

    Args:
        app (flask.Flask): App fixture.
        client (FlaskClient): Cliente.
    Returns:
        None
    """
```

| Anidada | Firma | Descripción |
|---|---|---|
| `foo()` | `() -> str` | Ruta `/foo` retorna `request.endpoint` (`bp.foo`) |
| `foo_bar()` | `() -> str` | Ruta `/bar` con `endpoint="bar"` |
| `foo_bar_foo()` | `() -> str` | Ruta `/bar/123` con `endpoint="123"` |
| `bar_foo()` | `() -> str` | Ruta `/bar/foo` |
| `index()` | `() -> str` | Ruta `/` de app retorna `request.endpoint` |

### 2.15 `test_route_decorator_custom_endpoint_with_dots()`

```python
def test_route_decorator_custom_endpoint_with_dots(app, client):
    """Verifica que endpoints con punto lanzan ValueError.

    Args:
        app (flask.Flask): App fixture.
        client (FlaskClient): Cliente.
    Raises:
        ValueError: Si endpoint contiene ".".
    Returns:
        None
    """
```

#### 2.15.1 `view()`
```python
def view():
    """Vista dummy con __name__ = "a.b" para probar validación. Returns: str"""
```

### 2.16 `test_endpoint_decorator()`

```python
def test_endpoint_decorator(app, client):
    """Valida @bp.endpoint para asociar función a endpoint existente en url_map.

    Args:
        app (flask.Flask): App fixture.
        client (FlaskClient): Cliente.
    Returns:
        None
    """
```

#### 2.16.1 `foobar()`
```python
def foobar():
    """Handler para endpoint "bar". Returns: str: request.endpoint"""
```

### 2.17 `test_template_filter()`

```python
def test_template_filter(app):
    """Valida registro de filtros Jinja vía @bp.app_template_filter con variantes de nombre.

    Args:
        app (flask.Flask): App fixture.
    Returns:
        None
    """
```

| Anidada | Firma | Descripción |
|---|---|---|
| `my_reverse(s)` | `(s: str) -> str` | Retorna `s[::-1]`, registrado como `my_reverse` |
| `my_reverse_2(s)` | `(s: str) -> str` | Variante con decorador sin paréntesis |
| `my_reverse_3(s)` | `(s: str) -> str` | Registrado como `my_reverse_custom_name_3` |
| `my_reverse_4(s)` | `(s: str) -> str` | Registrado como `my_reverse_custom_name_4` vía `name=` |

### 2.18 `test_add_template_filter()`

```python
def test_add_template_filter(app):
    """Valida bp.add_app_template_filter sin nombre custom.

    Args:
        app (flask.Flask): App fixture.
    Returns:
        None
    """
```

#### 2.18.1 `my_reverse(s)` - `(s: str) -> str` retorna `s[::-1]`.

### 2.19 `test_template_filter_with_name()`

```python
def test_template_filter_with_name(app):
    """Valida @bp.app_template_filter("strrev").

    Args:
        app (flask.Flask): App fixture.
    Returns:
        None
    """
```

#### 2.19.1 `my_reverse(s)` - `(s: str) -> str` registrado como `strrev`.

### 2.20 `test_add_template_filter_with_name()`

```python
def test_add_template_filter_with_name(app):
    """Valida bp.add_app_template_filter(my_reverse, "strrev").

    Args:
        app (flask.Flask): App fixture.
    Returns:
        None
    """
```

#### 2.20.1 `my_reverse(s)` - `(s: str) -> str`.

### 2.21 `test_template_filter_with_template()`

```python
def test_template_filter_with_template(app, client):
    """Valida uso real del filtro en template_filter.html.

    Args:
        app (flask.Flask): App fixture.
        client (FlaskClient): Cliente.
    Returns:
        None
    """
```

| Anidada | Firma | Descripción |
|---|---|---|
| `super_reverse(s)` | `(s: str) -> str` | Filtro `super_reverse` retorna `s[::-1]` |
| `index()` | `() -> str` | Ruta `/` renderiza `template_filter.html` con `value="abcd"` |

### 2.22 `test_template_filter_after_route_with_template()`

```python
def test_template_filter_after_route_with_template(app, client):
    """Valida que el orden registro ruta -> filtro no afecta.

    Args:
        app (flask.Flask): App fixture.
        client (FlaskClient): Cliente.
    Returns:
        None
    """
```

| Anidada | Firma | Descripción |
|---|---|---|
| `index()` | `() -> str` | Ruta `/` definida antes del filtro |
| `super_reverse(s)` | `(s: str) -> str` | Filtro registrado después |

### 2.23 `test_add_template_filter_with_template()`

Análoga a 2.21 pero usando `bp.add_app_template_filter(super_reverse)`.

| Anidada | Firma | Descripción |
|---|---|---|
| `super_reverse(s)` | `(s: str) -> str` |  |
| `index()` | `() -> str` |  |

### 2.24 `test_template_filter_with_name_and_template()`

| Anidada | Firma | Descripción |
|---|---|---|
| `my_reverse(s)` | `(s: str) -> str` | Registrado como `super_reverse` |
| `index()` | `() -> str` | Renderiza template |

### 2.25 `test_add_template_filter_with_name_and_template()`

| Anidada | Firma | Descripción |
|---|---|---|
| `my_reverse(s)` | `(s: str) -> str` | Registrado vía `add_app_template_filter(..., "super_reverse")` |
| `index()` | `() -> str` |  |

### 2.26 `test_template_test()`

```python
def test_template_test(app):
    """Valida registro de tests Jinja vía @bp.app_template_test.

    Args:
        app (flask.Flask): App fixture.
    Returns:
        None
    """
```

| Anidada | Firma | Descripción |
|---|---|---|
| `is_boolean(value)` | `(value: Any) -> bool` | `isinstance(value, bool)` registrado como `is_boolean` |
| `boolean_2(value)` | `(value: Any) -> bool` | Decorador sin paréntesis |
| `boolean_3(value)` | `(value: Any) -> bool` | Nombre custom `my_boolean_custom_name` |
| `boolean_4(value)` | `(value: Any) -> bool` | Nombre custom `my_boolean_custom_name_2` vía `name=` |

### 2.27 `test_add_template_test()`

```python
def test_add_template_test(app):
    """Valida bp.add_app_template_test(is_boolean).

    Args:
        app (flask.Flask): App fixture.
    Returns:
        None
    """
```

#### 2.27.1 `is_boolean(value)` - `(value: Any) -> bool`.

### 2.28 `test_template_test_with_name()`

#### 2.28.1 `is_boolean(value)` - Registrado como `boolean`.

### 2.29 `test_add_template_test_with_name()`

#### 2.29.1 `is_boolean(value)` - Registrado vía `add_app_template_test(..., "boolean")`.

### 2.30 `test_template_test_with_template()`

| Anidada | Firma | Descripción |
|---|---|---|
| `boolean(value)` | `(value: Any) -> bool` | Test `boolean` |
| `index()` | `() -> str` | Ruta `/` renderiza `template_test.html` con `value=False` |

### 2.31 `test_template_test_after_route_with_template()`

| Anidada | Firma | Descripción |
|---|---|---|
| `index()` | `() -> str` | Definida antes del test |
| `boolean(value)` | `(value: Any) -> bool` |  |

### 2.32 `test_add_template_test_with_template()`

| Anidada | Firma | Descripción |
|---|---|---|
| `boolean(value)` | `(value: Any) -> bool` | Registrado vía `add_app_template_test` |
| `index()` | `() -> str` |  |

### 2.33 `test_template_test_with_name_and_template()`

| Anidada | Firma | Descripción |
|---|---|---|
| `is_boolean(value)` | `(value: Any) -> bool` | Registrado como `boolean` |
| `index()` | `() -> str` |  |

### 2.34 `test_add_template_test_with_name_and_template()`

| Anidada | Firma | Descripción |
|---|---|---|
| `is_boolean(value)` | `(value: Any) -> bool` |  |
| `index()` | `() -> str` |  |

### 2.35 `test_context_processing()`

```python
def test_context_processing(app, client):
    """Valida context_processor local vs app_context_processor.

    Args:
        app (flask.Flask): App fixture.
        client (FlaskClient): Cliente.
    Returns:
        None
    """
```

| Anidada | Firma | Descripción |
|---|---|---|
| `template_string()` | `() -> str` | Renderiza string con `notanswer` y `answer` |
| `not_answer_context_processor()` | `() -> dict` | `@app_context_processor` retorna `{"notanswer": 43}` global |
| `answer_context_processor()` | `() -> dict` | `@context_processor` retorna `{"answer": 42}` solo para blueprint |
| `bp_page()` | `() -> str` | Ruta `/bp` retorna `template_string()` |
| `app_page()` | `() -> str` | Ruta `/` retorna `template_string()` |

### 2.36 `test_template_global()`

```python
def test_template_global(app):
    """Valida registro de globales Jinja vía @bp.app_template_global.

    Args:
        app (flask.Flask): App fixture.
    Returns:
        None
    """
```

| Anidada | Firma | Descripción |
|---|---|---|
| `get_answer()` | `() -> int` | Retorna `42`, registrado como `get_answer` |
| `get_stuff_1()` | `() -> str` | Retorna `"get_stuff_1"` |
| `get_stuff_2()` | `() -> str` | Registrado como `my_get_stuff_custom_name_2` |
| `get_stuff_3()` | `() -> str` | Registrado como `my_get_stuff_custom_name_3` vía `name=` |

### 2.37 `test_request_processing()`

```python
def test_request_processing(app, client):
    """Valida hooks before/after/teardown a nivel Blueprint.

    Args:
        app (flask.Flask): App fixture.
        client (FlaskClient): Cliente.
    Returns:
        None
    """
```

| Anidada | Firma | Descripción |
|---|---|---|
| `before_bp()` | `() -> None` | `@bp.before_request` añade `"before"` a `evts` |
| `after_bp(response)` | `(response: Response) -> Response` | Añade `b"|after"` a `response.data` y `"after"` a `evts` |
| `teardown_bp(exc)` | `(exc: BaseException | None) -> None` | Añade `"teardown"` a `evts` |
| `bp_endpoint()` | `() -> str` | Ruta `/bp` retorna `"request"` |

### 2.38 `test_app_request_processing()`

```python
def test_app_request_processing(app, client):
    """Valida hooks before_app/after_app/teardown_app registrados desde Blueprint.

    Args:
        app (flask.Flask): App fixture.
        client (FlaskClient): Cliente.
    Returns:
        None
    """
```

| Anidada | Firma | Descripción |
|---|---|---|
| `before_app()` | `() -> None` | `@bp.before_app_request` |
| `after_app(response)` | `(response: Response) -> Response` | `@bp.after_app_request` |
| `teardown_app(exc)` | `(exc: BaseException | None) -> None` | `@bp.teardown_app_request` |
| `bp_endpoint()` | `() -> str` | Ruta `/` retorna `"request"` |

### 2.39 `test_app_url_processors()`

```python
def test_app_url_processors(app, client):
    """Valida app_url_defaults y app_url_value_preprocessor registrados desde Blueprint.

    Args:
        app (flask.Flask): App fixture.
        client (FlaskClient): Cliente.
    Returns:
        None
    """
```

| Anidada | Firma | Descripción |
|---|---|---|
| `add_language_code(endpoint, values)` | `(endpoint: str, values: dict) -> None` | `@bp.app_url_defaults` |
| `pull_lang_code(endpoint, values)` | `(endpoint: str, values: dict) -> None` | `@bp.app_url_value_preprocessor` |
| `index()` | `() -> str` | Ruta `/<lang_code>/` retorna `url_for("about")` |
| `about()` | `() -> str` | Ruta `/<lang_code>/about` retorna `url_for("index")` |

### 2.40 `test_nested_blueprint()`

```python
def test_nested_blueprint(app, client):
    """Valida anidamiento de 3 niveles y herencia de errorhandlers.

    Args:
        app (flask.Flask): App fixture.
        client (FlaskClient): Cliente.
    Returns:
        None
    """
```

| Anidada | Firma | Descripción |
|---|---|---|
| `forbidden(e)` | `(e: Exception) -> tuple[str, int]` | Handler 403 de `parent` |
| `parent_index()` | `() -> str` | Ruta `/` de parent |
| `parent_no()` | `() -> None` | Ruta `/no` hace `abort(403)` |
| `child_index()` | `() -> str` | Ruta `/` de child |
| `child_no()` | `() -> None` | Ruta `/no` de child |
| `grandchild_forbidden(e)` | `(e: Exception) -> tuple[str, int]` | Handler 403 de grandchild |
| `grandchild_index()` | `() -> str` | Ruta `/` de grandchild |
| `grandchild_no()` | `() -> None` | Ruta `/no` de grandchild |

### 2.41 `test_nested_callback_order()`

```python
def test_nested_callback_order(app, client):
    """Valida orden de ejecución de before_request/teardown y context_processor anidados.

    Args:
        app (flask.Flask): App fixture.
        client (FlaskClient): Cliente.
    Returns:
        None
    """
```

| Anidada | Firma | Descripción |
|---|---|---|
| `app_before1()` | `() -> None` | `@app.before_request` añade `app_1` a `g.seen` |
| `app_teardown1(e)` | `(e: BaseException | None) -> None` | Verifica y hace pop `app_1` |
| `app_before2()` | `() -> None` | Añade `app_2` |
| `app_teardown2(e)` | `(e: BaseException | None) -> None` | Pop `app_2` |
| `app_ctx()` | `() -> dict` | `@app.context_processor` retorna `{"key": "app"}` |
| `parent_before1()` | `() -> None` | Añade `parent_1` |
| `parent_teardown1(e)` | `(e: BaseException | None) -> None` | Pop `parent_1` |
| `parent_before2()` | `() -> None` | Añade `parent_2` |
| `parent_teardown2(e)` | `(e: BaseException | None) -> None` | Pop `parent_2` |
| `parent_ctx()` | `() -> dict` | Retorna `{"key": "parent"}` |
| `child_before1()` | `() -> None` | Añade `child_1` |
| `child_teardown1(e)` | `(e: BaseException | None) -> None` | Pop `child_1` |
| `child_before2()` | `() -> None` | Añade `child_2` |
| `child_teardown2(e)` | `(e: BaseException | None) -> None` | Pop `child_2` |
| `child_ctx()` | `() -> dict` | Retorna `{"key": "child"}` |
| `a()` | `() -> str` | Ruta `/a` retorna `", ".join(g.seen)` |
| `b()` | `() -> str` | Ruta `/b` renderiza `{{ key }}` |

### 2.42 `test_nesting_url_prefixes()`

```python
def test_nesting_url_prefixes(parent_init, child_init, parent_registration, child_registration, app, client) -> None:
    """Valida combinación de url_prefix en init vs registro para blueprints anidados.

    Args:
        parent_init (str | None): url_prefix en constructor del parent.
        child_init (str | None): url_prefix en constructor del child.
        parent_registration (str | None): url_prefix al registrar parent en app.
        child_registration (str | None): url_prefix al registrar child en parent.
        app (flask.Flask): App fixture.
        client (FlaskClient): Cliente.
    Returns:
        None
    """
```

#### 2.42.1 `index()`
```python
def index():
    """Ruta / del child. Returns: str: "index" """
```

### 2.43 `test_nesting_subdomains()`

```python
def test_nesting_subdomains(app, client) -> None:
    """Valida anidamiento con subdomain_matching.

    Args:
        app (flask.Flask): App fixture.
        client (FlaskClient): Cliente.
    Returns:
        None
    """
```

#### 2.43.1 `index()` - `() -> str` Ruta `/child/` retorna `"child"`.

### 2.44 `test_child_and_parent_subdomain()`

```python
def test_child_and_parent_subdomain(app, client) -> None:
    """Valida composición de subdominios parent + child (api.parent.example.test).

    Args:
        app (flask.Flask): App fixture.
        client (FlaskClient): Cliente.
    Returns:
        None
    """
```

#### 2.44.1 `index()` - `() -> str` Ruta `/` retorna `"child"`.

### 2.45 `test_unique_blueprint_names()`

```python
def test_unique_blueprint_names(app, client) -> None:
    """Valida unicidad de nombres de Blueprint y parámetro name en registro.

    Args:
        app (flask.Flask): App fixture.
        client (FlaskClient): Cliente.
    Raises:
        ValueError: Si se registra mismo nombre o mismo objeto sin alias.
    Returns:
        None
    """
```

### 2.46 `test_self_registration()`

```python
def test_self_registration(app, client) -> None:
    """Verifica que un Blueprint no puede registrarse a sí mismo.

    Args:
        app (flask.Flask): App fixture.
        client (FlaskClient): Cliente.
    Raises:
        ValueError: Auto-registro no permitido.
    Returns:
        None
    """
```

### 2.47 `test_blueprint_renaming()`

```python
def test_blueprint_renaming(app, client) -> None:
    """Valida renombrado de Blueprint vía parámetro name y aislamiento de endpoints/errorhandlers.

    Args:
        app (flask.Flask): App fixture.
        client (FlaskClient): Cliente.
    Returns:
        None
    """
```

| Anidada | Firma | Descripción |
|---|---|---|
| `index()` | `() -> str` | `@bp.get("/")` retorna `request.endpoint` |
| `error()` | `() -> None` | `@bp.get("/error")` hace `abort(403)` |
| `forbidden(_)` | `(_: Exception) -> tuple[str, int]` | `@bp.errorhandler(403)` retorna `"Error", 403` |
| `index2()` | `() -> str` | `@bp2.get("/")` retorna `request.endpoint` |

---

## 3. Ejemplo de Uso

```python
import flask

# 1. Creación de la app y blueprints con manejo de errores aislado
app = flask.Flask(__name__)
frontend = flask.Blueprint("frontend", __name__)
backend = flask.Blueprint("backend", __name__)

@frontend.errorhandler(403)
def frontend_forbidden(e):
    """Handler específico para frontend."""
    return "frontend says no", 403

@frontend.route("/frontend-no")
def frontend_no():
    flask.abort(403)

@backend.errorhandler(403)
def backend_forbidden(e):
    """Handler específico para backend."""
    return "backend says no", 403

@backend.route("/backend-no")
def backend_no():
    flask.abort(403)

# 2. Procesadores de URL a nivel blueprint (i18n)
bp = flask.Blueprint("shop", __name__, url_prefix="/<lang_code>")

@bp.url_value_preprocessor
def pull_lang_code(endpoint, values):
    flask.g.lang_code = values.pop("lang_code")

@bp.url_defaults
def add_language_code(endpoint, values):
    values.setdefault("lang_code", flask.g.lang_code)

@bp.route("/")
def index():
    return flask.url_for(".about")

# 3. Filtros y contexto
@bp.app_template_filter("reverse")
def my_reverse(s: str) -> str:
    return s[::-1]

@bp.context_processor
def inject_answer():
    return {"answer": 42}

# 4. Anidamiento y registro con prefijos
parent = flask.Blueprint("parent", __name__)
child = flask.Blueprint("child", __name__)

@child.route("/")
def child_index():
    return "Child yes"

parent.register_blueprint(child, url_prefix="/child")
app.register_blueprint(parent, url_prefix="/parent")
app.register_blueprint(frontend)
app.register_blueprint(backend)
app.register_blueprint(bp)

# 5. Uso del cliente de pruebas
with app.test_client() as client:
    assert client.get("/frontend-no").data == b"frontend says no"
    assert client.get("/backend-no").data == b"backend says no"
    assert client.get("/de/").data == b"/de/about"
    assert client.get("/parent/child/").data == b"Child yes"

# 6. Renombrado para reutilizar el mismo blueprint con distinto prefijo
app2 = flask.Flask(__name__)
bp_reusable = flask.Blueprint("bp", __name__)

@bp_reusable.get("/")
def reusable_index():
    return flask.request.endpoint

app2.register_blueprint(bp_reusable, url_prefix="/a")
app2.register_blueprint(bp_reusable, url_prefix="/b", name="alt")
# /a/ -> bp.reusable_index, /b/ -> alt.reusable_index
```
