# Documentación Técnica - Módulo `scaffold` (Flask)

## 1. Resumen Ejecutivo

El módulo `scaffold` implementa la clase base `Scaffold`, que centraliza el comportamiento común entre `Flask` y `Blueprint` para el registro de rutas, manejo de errores, hooks del ciclo de vida de la petición y configuración de recursos estáticos y plantillas Jinja2. Provee además el decorador `setupmethod` para controlar el estado de configuración y un conjunto de utilidades a nivel de módulo (`_find_package_path`, `find_package`, `_endpoint_from_view_func`) para la resolución de rutas de paquetes y endpoints.

---

## 2. Especificación de Funciones y Métodos

### 2.1. Función `setupmethod()`

Decorador que protege métodos de configuración para que solo puedan ser invocados antes de que la aplicación finalice su fase de setup. Delega la validación a `Scaffold._check_setup_finished`.

**Docstring (Google Style):**
```python
def setupmethod(f: F) -> F:
    """Decora un método de setup para validar el estado de configuración.

    Args:
        f: Función a decorar. Debe ser un método de instancia de Scaffold.

    Returns:
        Función decorada que invoca `_check_setup_finished` antes de ejecutar `f`.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `f` | `F` (`Callable[..., Any]`) | Sí | Función/método a envolver. Se preserva su metadata vía `update_wrapper`. |

| Retorno | Tipo | Descripción |
| :--- | :--- | :--- |
| `decorated` | `F` | Wrapper que valida el setup y delega la ejecución a `f`. |

| Excepciones | Condición |
| :--- | :--- |
| `NotImplementedError` | Propagada desde `Scaffold._check_setup_finished` si el setup ya finalizó (implementación en subclase). |

#### 2.1.1. Función anidada `wrapper_func()`

Función local definida dentro de `setupmethod()`.

**Docstring (Google Style):**
```python
def wrapper_func(self: Scaffold, *args: Any, **kwargs: Any) -> Any:
    """Wrapper de validación de setup.

    Args:
        self: Instancia de Scaffold.
        *args: Argumentos posicionales del método original.
        **kwargs: Argumentos nominales del método original.

    Returns:
        El valor de retorno del método original `f`.

    Raises:
        NotImplementedError: Si `_check_setup_finished` determina que la configuración ha concluido.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `self` | `Scaffold` | Sí | Instancia sobre la que se invoca el método decorado. |
| `*args` | `Any` | No | Argumentos variables del método original. |
| `**kwargs` | `Any` | No | Argumentos nominales del método original. |

| Retorno | Tipo | Descripción |
| :--- | :--- | :--- |
| `result` | `Any` | Resultado de `f(self, *args, **kwargs)`. |

---

### 2.2. Clase `Scaffold`

Clase base para `Flask` y `Blueprint`. Gestiona registro de vistas, manejadores de error, hooks de petición, procesadores de contexto y resolución de recursos.

#### 2.2.1. Método `Scaffold.__init__()`

```python
def __init__(self, import_name: str, static_folder: str | os.PathLike[str] | None = None, static_url_path: str | None = None, template_folder: str | os.PathLike[str] | None = None, root_path: str | None = None):
    """Inicializa la instancia Scaffold.

    Args:
        import_name: Nombre de importación del paquete/módulo donde se define el objeto. Usualmente `__name__`.
        static_folder: Ruta a la carpeta de archivos estáticos. Si se define, se añade ruta estática.
        static_url_path: Prefijo URL para la ruta estática.
        template_folder: Ruta a la carpeta de plantillas Jinja2.
        root_path: Ruta absoluta base para recursos. Si es None, se descubre vía `get_root_path(import_name)`.

    Attributes:
        import_name (str): Nombre de importación.
        root_path (str): Ruta absoluta del paquete en filesystem.
        view_functions (dict[str, RouteCallable]): Mapeo endpoint -> view func.
        error_handler_spec (dict): Estructura `{scope: {code: {class: handler}}}`.
        before_request_funcs (dict): `{scope: [callables]}`.
        after_request_funcs (dict): `{scope: [callables]}`.
        teardown_request_funcs (dict): `{scope: [callables]}`.
        template_context_processors (dict): `{scope: [callables]}`.
        url_value_preprocessors (dict): `{scope: [callables]}`.
        url_default_functions (dict): `{scope: [callables]}`.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `import_name` | `str` | Sí | Nombre del módulo/paquete. |
| `static_folder` | `str \| os.PathLike[str] \| None` | No | Carpeta estática relativa a `root_path`. Default `None`. |
| `static_url_path` | `str \| None` | No | Prefijo URL estático. Default `None`. |
| `template_folder` | `str \| os.PathLike[str] \| None` | No | Carpeta de plantillas relativa a `root_path`. Default `None`. |
| `root_path` | `str \| None` | No | Ruta raíz absoluta. Si `None`, se infiere. |

| Retorno | Tipo | Descripción |
| :--- | :--- | :--- |
| `None` | `None` | Inicializador, no retorna valor. |

#### 2.2.2. Método `Scaffold.__repr__()`

```python
def __repr__(self) -> str:
    """Retorna representación legible de la instancia.

    Returns:
        String con formato `<ClassName 'name'>`.
    """
```

| Retorno | Tipo | Descripción |
| :--- | :--- | :--- |
| `repr` | `str` | Representación de la instancia. |

#### 2.2.3. Método `Scaffold._check_setup_finished()`

```python
def _check_setup_finished(self, f_name: str) -> None:
    """Valida si la fase de setup ha finalizado.

    Args:
        f_name: Nombre del método que intenta ejecutarse.

    Raises:
        NotImplementedError: Siempre, debe ser implementado por subclases (Flask/Blueprint).
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `f_name` | `str` | Sí | Nombre del método decorado con `@setupmethod`. |

| Excepciones | Condición |
| :--- | :--- |
| `NotImplementedError` | Método abstracto, sin implementación en `Scaffold`. |

#### 2.2.4. Propiedad `Scaffold.static_folder` (getter)

```python
@property
def static_folder(self) -> str | None:
    """Ruta absoluta a la carpeta estática configurada.

    Returns:
        Ruta absoluta si `_static_folder` está definido, `None` en caso contrario.
    """
```

| Retorno | Tipo | Descripción |
| :--- | :--- | :--- |
| `path` | `str \| None` | `os.path.join(root_path, _static_folder)` o `None`. |

#### 2.2.5. Propiedad `Scaffold.static_folder` (setter)

```python
@static_folder.setter
def static_folder(self, value: str | os.PathLike[str] | None) -> None:
    """Establece la carpeta estática.

    Args:
        value: Ruta a la carpeta o None. Se normaliza con `os.fspath` y `rstrip(r"\/")`.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `value` | `str \| os.PathLike[str] \| None` | Sí | Nueva carpeta estática. |

#### 2.2.6. Propiedad `Scaffold.has_static_folder`

```python
@property
def has_static_folder(self) -> bool:
    """Indica si existe carpeta estática configurada.

    Returns:
        True si `static_folder` no es None, False en caso contrario.
    """
```

| Retorno | Tipo | Descripción |
| :--- | :--- | :--- |
| `has_folder` | `bool` | Estado de configuración estática. |

#### 2.2.7. Propiedad `Scaffold.static_url_path` (getter)

```python
@property
def static_url_path(self) -> str | None:
    """Prefijo URL para la ruta estática.

    Returns:
        Valor explícito de `_static_url_path` si existe, derivado de `static_folder` (`/{basename}`) o None.
    """
```

| Retorno | Tipo | Descripción |
| :--- | :--- | :--- |
| `url_path` | `str \| None` | Prefijo URL normalizado. |

#### 2.2.8. Propiedad `Scaffold.static_url_path` (setter)

```python
@static_url_path.setter
def static_url_path(self, value: str | None) -> None:
    """Establece el prefijo URL estático.

    Args:
        value: Prefijo URL o None. Se normaliza con `rstrip("/")`.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `value` | `str \| None` | Sí | Nuevo prefijo URL. |

#### 2.2.9. Propiedad `Scaffold.jinja_loader`

```python
@cached_property
def jinja_loader(self) -> BaseLoader | None:
    """Loader Jinja2 para las plantillas del objeto.

    Returns:
        FileSystemLoader apuntando a `root_path/template_folder` si `template_folder` está definido, None en caso contrario.
    """
```

| Retorno | Tipo | Descripción |
| :--- | :--- | :--- |
| `loader` | `BaseLoader \| None` | Loader cacheado. |

#### 2.2.10. Método `Scaffold._method_route()`

```python
def _method_route(self, method: str, rule: str, options: dict[str, Any]) -> Callable[[T_route], T_route]:
    """Helper interno para atajos de métodos HTTP.

    Args:
        method: Verbo HTTP (GET, POST, etc.).
        rule: Regla URL.
        options: Opciones adicionales para `route`.

    Returns:
        Decorador de ruta configurado con `methods=[method]`.

    Raises:
        TypeError: Si `options` contiene la clave `methods`.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `method` | `str` | Sí | Método HTTP. |
| `rule` | `str` | Sí | Regla URL. |
| `options` | `dict[str, Any]` | Sí | Opciones para `route`. |

| Excepciones | Condición |
| :--- | :--- |
| `TypeError` | Si `methods` ya está en `options`. |

#### 2.2.11. Método `Scaffold.get()`

```python
@setupmethod
def get(self, rule: str, **options: Any) -> Callable[[T_route], T_route]:
    """Atajo para `route` con `methods=["GET"]`.

    Args:
        rule: Regla URL.
        **options: Opciones pasadas a `route`.

    Returns:
        Decorador que registra la vista para GET.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `rule` | `str` | Sí | Regla URL. |
| `**options` | `Any` | No | Opciones de `Rule`. |

#### 2.2.12. Método `Scaffold.query()`

```python
@setupmethod
def query(self, rule: str, **options: Any) -> Callable[[T_route], T_route]:
    """Atajo para `route` con `methods=["QUERY"]`.

    Args:
        rule: Regla URL.
        **options: Opciones pasadas a `route`.

    Returns:
        Decorador que registra la vista para QUERY.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `rule` | `str` | Sí | Regla URL. |
| `**options` | `Any` | No | Opciones de `Rule`. |

#### 2.2.13. Método `Scaffold.post()`

```python
@setupmethod
def post(self, rule: str, **options: Any) -> Callable[[T_route], T_route]:
    """Atajo para `route` con `methods=["POST"]`."""
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `rule` | `str` | Sí | Regla URL. |
| `**options` | `Any` | No | Opciones de `Rule`. |

#### 2.2.14. Método `Scaffold.put()`

```python
@setupmethod
def put(self, rule: str, **options: Any) -> Callable[[T_route], T_route]:
    """Atajo para `route` con `methods=["PUT"]`."""
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `rule` | `str` | Sí | Regla URL. |
| `**options` | `Any` | No | Opciones de `Rule`. |

#### 2.2.15. Método `Scaffold.delete()`

```python
@setupmethod
def delete(self, rule: str, **options: Any) -> Callable[[T_route], T_route]:
    """Atajo para `route` con `methods=["DELETE"]`."""
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `rule` | `str` | Sí | Regla URL. |
| `**options` | `Any` | No | Opciones de `Rule`. |

#### 2.2.16. Método `Scaffold.patch()`

```python
@setupmethod
def patch(self, rule: str, **options: Any) -> Callable[[T_route], T_route]:
    """Atajo para `route` con `methods=["PATCH"]`."""
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `rule` | `str` | Sí | Regla URL. |
| `**options` | `Any` | No | Opciones de `Rule`. |

#### 2.2.17. Método `Scaffold.route()`

```python
@setupmethod
def route(self, rule: str, **options: Any) -> Callable[[T_route], T_route]:
    """Decora una view function para registrarla con una regla URL.

    Args:
        rule: String de la regla URL.
        **options: Opciones extra pasadas a `werkzeug.routing.Rule`. Incluye `endpoint` y `methods`.

    Returns:
        Decorador que registra la función vía `add_url_rule` y retorna la función original.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `rule` | `str` | Sí | Regla URL. |
| `**options` | `Any` | No | Opciones de `Rule` (`endpoint`, `methods`, etc.). |

##### 2.2.17.1. Función anidada `decorator()` (dentro de `Scaffold.route()`)

```python
def decorator(f: T_route) -> T_route:
    """Decorador interno de `route`.

    Args:
        f: View function a registrar.

    Returns:
        La misma view function `f` tras registrarla con `add_url_rule`.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `f` | `T_route` (`RouteCallable`) | Sí | Función de vista. |

#### 2.2.18. Método `Scaffold.add_url_rule()`

```python
@setupmethod
def add_url_rule(self, rule: str, endpoint: str | None = None, view_func: RouteCallable | None = None, provide_automatic_options: bool | None = None, **options: Any) -> None:
    """Registra una regla para routing y construcción de URLs.

    Args:
        rule: String de la regla URL.
        endpoint: Nombre del endpoint. Default `view_func.__name__`.
        view_func: Función de vista asociada al endpoint.
        provide_automatic_options: Si añade método OPTIONS automáticamente.
        **options: Opciones extra para `werkzeug.routing.Rule`.

    Raises:
        NotImplementedError: Método abstracto, implementado por subclases.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `rule` | `str` | Sí | Regla URL. |
| `endpoint` | `str \| None` | No | Nombre del endpoint. |
| `view_func` | `RouteCallable \| None` | No | Función de vista. |
| `provide_automatic_options` | `bool \| None` | No | Control de OPTIONS automático. |
| `**options` | `Any` | No | Opciones de `Rule`. |

#### 2.2.19. Método `Scaffold.endpoint()`

```python
@setupmethod
def endpoint(self, endpoint: str) -> Callable[[F], F]:
    """Decora una view function para registrarla en un endpoint existente.

    Args:
        endpoint: Nombre del endpoint a asociar con la view function.

    Returns:
        Decorador que registra la función en `view_functions[endpoint]`.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `endpoint` | `str` | Sí | Nombre del endpoint. |

##### 2.2.19.1. Función anidada `decorator()` (dentro de `Scaffold.endpoint()`)

```python
def decorator(f: F) -> F:
    """Registra `f` en `view_functions`.

    Args:
        f: View function.

    Returns:
        La misma función `f`.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `f` | `F` (`Callable`) | Sí | Función de vista. |

#### 2.2.20. Método `Scaffold.before_request()`

```python
@setupmethod
def before_request(self, f: T_before_request) -> T_before_request:
    """Registra función a ejecutar antes de cada request.

    Args:
        f: Callable sin argumentos. Si retorna valor no-None, detiene el manejo del request.

    Returns:
        La misma función `f` registrada en `before_request_funcs[None]`.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `f` | `T_before_request` (`BeforeRequestCallable`) | Sí | Hook pre-request. |

#### 2.2.21. Método `Scaffold.after_request()`

```python
@setupmethod
def after_request(self, f: T_after_request) -> T_after_request:
    """Registra función a ejecutar después de cada request.

    Args:
        f: Callable que recibe y debe retornar un objeto response.

    Returns:
        La misma función `f` registrada en `after_request_funcs[None]`.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `f` | `T_after_request` (`AfterRequestCallable`) | Sí | Hook post-request. |

#### 2.2.22. Método `Scaffold.teardown_request()`

```python
@setupmethod
def teardown_request(self, f: T_teardown) -> T_teardown:
    """Registra función a llamar al hacer pop del contexto de request.

    Args:
        f: Callable que recibe opcionalmente un objeto de error. Su retorno es ignorado.

    Returns:
        La misma función `f` registrada en `teardown_request_funcs[None]`.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `f` | `T_teardown` (`TeardownCallable`) | Sí | Hook de teardown. |

#### 2.2.23. Método `Scaffold.context_processor()`

```python
@setupmethod
def context_processor(self, f: T_template_context_processor) -> T_template_context_processor:
    """Registra un procesador de contexto de plantillas.

    Args:
        f: Callable que retorna dict con variables para el template.

    Returns:
        La misma función `f` registrada en `template_context_processors[None]`.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `f` | `T_template_context_processor` | Sí | Procesador de contexto. |

#### 2.2.24. Método `Scaffold.url_value_preprocessor()`

```python
@setupmethod
def url_value_preprocessor(self, f: T_url_value_preprocessor) -> T_url_value_preprocessor:
    """Registra preprocesador de valores de URL.

    Args:
        f: Callable con firma `(endpoint, values)`. Puede modificar `values` in-place. Retorno ignorado.

    Returns:
        La misma función `f` registrada en `url_value_preprocessors[None]`.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `f` | `T_url_value_preprocessor` | Sí | Preprocesador de valores URL. |

#### 2.2.25. Método `Scaffold.url_defaults()`

```python
@setupmethod
def url_defaults(self, f: T_url_defaults) -> T_url_defaults:
    """Registra callback para defaults de URL.

    Args:
        f: Callable con firma `(endpoint, values)` que actualiza `values` in-place.

    Returns:
        La misma función `f` registrada en `url_default_functions[None]`.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `f` | `T_url_defaults` (`URLDefaultCallable`) | Sí | Callback de defaults. |

#### 2.2.26. Método `Scaffold.errorhandler()`

```python
@setupmethod
def errorhandler(self, code_or_exception: type[Exception] | int) -> Callable[[T_error_handler], T_error_handler]:
    """Registra función manejadora de errores por código o clase de excepción.

    Args:
        code_or_exception: Código HTTP entero o clase de Excepción.

    Returns:
        Decorador que registra el handler vía `register_error_handler`.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `code_or_exception` | `type[Exception] \| int` | Sí | Código HTTP o clase de excepción. |

##### 2.2.26.1. Función anidada `decorator()` (dentro de `Scaffold.errorhandler()`)

```python
def decorator(f: T_error_handler) -> T_error_handler:
    """Decorador interno de `errorhandler`.

    Args:
        f: Función manejadora de error.

    Returns:
        La misma función `f` tras registrarla.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `f` | `T_error_handler` (`ErrorHandlerCallable`) | Sí | Handler de error. |

#### 2.2.27. Método `Scaffold.register_error_handler()`

```python
@setupmethod
def register_error_handler(self, code_or_exception: type[Exception] | int, f: ErrorHandlerCallable) -> None:
    """Registra handler de error de forma no decoradora.

    Args:
        code_or_exception: Código HTTP o clase de excepción.
        f: Función manejadora.

    Returns:
        None. Almacena en `error_handler_spec[None][code][exc_class]`.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `code_or_exception` | `type[Exception] \| int` | Sí | Código o excepción. |
| `f` | `ErrorHandlerCallable` | Sí | Handler. |

#### 2.2.28. Método `Scaffold._get_exc_class_and_code()` (staticmethod)

```python
@staticmethod
def _get_exc_class_and_code(exc_class_or_code: type[Exception] | int) -> tuple[type[Exception], int | None]:
    """Obtiene clase de excepción y código HTTP asociado.

    Args:
        exc_class_or_code: Clase de excepción o código HTTP entero.

    Returns:
        Tupla `(exc_class, code)` donde `code` es `exc_class.code` si es HTTPException, None en caso contrario.

    Raises:
        ValueError: Si el entero no es un código HTTP reconocido o si la clase no es subclase de Exception.
        TypeError: Si se pasa una instancia en lugar de una clase.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `exc_class_or_code` | `type[Exception] \| int` | Sí | Código o clase. |

| Retorno | Tipo | Descripción |
| :--- | :--- | :--- |
| `tuple` | `tuple[type[Exception], int \| None]` | Clase y código HTTP. |

| Excepciones | Condición |
| :--- | :--- |
| `ValueError` | Código no está en `werkzeug.exceptions.default_exceptions` o clase no hereda de `Exception`. |
| `TypeError` | `exc_class_or_code` es instancia de `Exception`, no clase. |

---

### 2.3. Funciones a Nivel de Módulo

#### 2.3.1. Función `_endpoint_from_view_func()`

```python
def _endpoint_from_view_func(view_func: RouteCallable) -> str:
    """Retorna el endpoint por defecto para una view function.

    Args:
        view_func: Función de vista. No debe ser None.

    Returns:
        Nombre de la función (`view_func.__name__`).

    Raises:
        AssertionError: Si `view_func` es None.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `view_func` | `RouteCallable` | Sí | Función de vista. |

| Retorno | Tipo | Descripción |
| :--- | :--- | :--- |
| `endpoint` | `str` | `view_func.__name__`. |

#### 2.3.2. Función `_find_package_path()`

```python
def _find_package_path(import_name: str) -> str:
    """Encuentra la ruta que contiene el paquete o módulo.

    Args:
        import_name: Nombre de importación (ej. `myapp.module`).

    Returns:
        Ruta del directorio que contiene el paquete/módulo. Retorna `os.getcwd()` si no se encuentra el spec o es inválido.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `import_name` | `str` | Sí | Nombre de importación. |

| Retorno | Tipo | Descripción |
| :--- | :--- | :--- |
| `path` | `str` | Ruta del paquete. Maneja namespace packages, packages con `__init__.py` y módulos simples. |

#### 2.3.3. Función `find_package()`

```python
def find_package(import_name: str) -> tuple[str | None, str]:
    """Encuentra el prefijo de instalación y la ruta de importación de un paquete.

    Args:
        import_name: Nombre de importación del paquete.

    Returns:
        Tupla `(prefix, package_path)` donde `prefix` es `sys.prefix` o ruta de virtualenv (`site-packages`) o None si no está instalado, y `package_path` es la entrada en `sys.path` o `cwd`.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `import_name` | `str` | Sí | Nombre de importación. |

| Retorno | Tipo | Descripción |
| :--- | :--- | :--- |
| `result` | `tuple[str \| None, str]` | `(prefix, package_path)`. Detecta instalación en sistema, virtualenv (Windows/Unix) o no instalado. |

---

## 3. Ejemplo de Uso

```python
from flask import Flask, g, request, render_template_string

# Scaffold es la base; Flask hereda de ella
app = Flask(__name__, template_folder="templates", static_folder="static")

# 1. Registro de rutas con atajos de método
@app.get("/hello/<name>")
def hello(name: str):
    return f"Hola, {name}!"

@app.post("/data")
def create_data():
    return {"status": "creado"}, 201

# 2. Registro vía add_url_rule + endpoint
def health():
    return "OK"

app.add_url_rule("/health", endpoint="health", view_func=health)

# 3. Hooks del ciclo de vida
@app.before_request
def load_user():
    # Se ejecuta antes de cada request
    g.user = request.headers.get("X-User", "anon")

@app.after_request
def add_header(response):
    # Modifica la respuesta
    response.headers["X-Powered-By"] = "Flask Scaffold"
    return response

@app.teardown_request
def close_db(exc=None):
    # Siempre se ejecuta al finalizar el contexto
    print(f"Teardown con exc={exc}")

# 4. Procesadores de contexto y URL
@app.context_processor
def inject_user():
    return {"current_user": getattr(g, "user", None)}

@app.url_value_preprocessor
def pull_lang(endpoint, values):
    # Extrae lang de la URL y lo pone en g
    g.lang = values.pop("lang", "es")

@app.url_defaults
def add_lang(endpoint, values):
    if "lang" in values or not g.get("lang"):
        return
    if app.url_map.is_endpoint_expecting(endpoint, "lang"):
        values["lang"] = g.lang

# 5. Manejo de errores
@app.errorhandler(404)
def not_found(error):
    return render_template_string("<h1>404 - {{ current_user }} no encontrado</h1>"), 404

@app.errorhandler(ValueError)
def handle_value_error(error):
    return f"Valor inválido: {error}", 400

# 6. Utilidades de paquete
from flask.scaffold import find_package
prefix, path = find_package("flask")
print(f"Flask instalado en: prefix={prefix}, path={path}")

if __name__ == "__main__":
    app.run(debug=True)
```
