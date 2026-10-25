# Documentación Técnica — Módulo `flask.sansio.scaffold`

## 1. Resumen ejecutivo

Este módulo define la clase base `Scaffold`, que encapsula el comportamiento común compartido entre `flask.Flask` y `flask.blueprints.Blueprint`: resolución de rutas de recursos (`root_path`, `static_folder`, `template_folder`), registro de vistas, manejadores de error, hooks de ciclo de vida de petición y procesadores de contexto de plantillas.

Adicionalmente, provee el decorador `setupmethod()` —que bloquea mutaciones de configuración una vez finalizada la fase de *setup*— y utilidades de descubrimiento de rutas del sistema de archivos (`find_package()`, `_find_package_path()`) empleadas para localizar el prefijo de instalación de un paquete.

El módulo es *sans-io*: no contiene lógica de transporte HTTP; los métodos `add_url_rule()` y `_check_setup_finished()` son abstractos y deben implementarse en las subclases.

---

## 2. Especificación de funciones y métodos

### 2.1 Constantes y variables de tipo a nivel de módulo

| Nombre | Tipo | Descripción |
|---|---|---|
| `_sentinel` | `object` | Valor centinela *singleton* usado como default de parámetros, para distinguir "no provisto" de `None`. |
| `F` | `TypeVar` (bound `Callable[..., Any]`) | Callable genérico; preserva la firma al decorar. |
| `T_after_request` | `TypeVar` (bound `ft.AfterRequestCallable[Any]`) | Tipo del callback de `after_request()`. |
| `T_before_request` | `TypeVar` (bound `ft.BeforeRequestCallable`) | Tipo del callback de `before_request()`. |
| `T_error_handler` | `TypeVar` (bound `ft.ErrorHandlerCallable`) | Tipo del manejador de errores. |
| `T_teardown` | `TypeVar` (bound `ft.TeardownCallable`) | Tipo del callback de `teardown_request()`. |
| `T_template_context_processor` | `TypeVar` (bound `ft.TemplateContextProcessorCallable`) | Tipo del procesador de contexto de plantillas. |
| `T_url_defaults` | `TypeVar` (bound `ft.URLDefaultCallable`) | Tipo del callback de `url_defaults()`. |
| `T_url_value_preprocessor` | `TypeVar` (bound `ft.URLValuePreprocessorCallable`) | Tipo del callback de `url_value_preprocessor()`. |
| `T_route` | `TypeVar` (bound `ft.RouteCallable`) | Tipo de la función de vista. |

---

### 2.2 Funciones a nivel de módulo

#### `setupmethod()`

```python
def setupmethod(f: F) -> F
```

> **Docstring propuesto (Google Style)**
>
> ```
> Decora un método de `Scaffold` para que valide el estado de setup.
>
> Envuelve el método `f` de forma que, antes de ejecutarlo, se invoque
> `self._check_setup_finished(f.__name__)`. Esto impide registrar rutas,
> hooks o manejadores después de que la aplicación haya comenzado a
> atender peticiones. Los metadatos del original se preservan mediante
> `functools.update_wrapper`.
>
> Args:
>     f (F): Método de instancia de `Scaffold` a decorar.
>
> Returns:
>     F: El wrapper, casteado al tipo original para preservar la firma.
> ```

**Parámetros**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `f` | `F` (`Callable[..., Any]`) | Sí | Método de instancia a proteger. |

**Retorno**

| Tipo | Descripción |
|---|---|
| `F` | `wrapper_func` envuelto con `update_wrapper` y `t.cast` al tipo de `f`. |

**Raises**

| Excepción | Condición |
|---|---|
| — | No lanza excepciones propias. Las excepciones se propagan desde `_check_setup_finished()` o desde `f`. |

**Variables locales relevantes**

| Nombre | Descripción |
|---|---|
| `f_name` | `f.__name__`, capturado por *closure* y pasado a `_check_setup_finished()`. |

---

##### `wrapper_func()` *(función anidada en `setupmethod()`)*

```python
def wrapper_func(self: Scaffold, *args: t.Any, **kwargs: t.Any) -> t.Any
```

> **Docstring propuesto**
>
> ```
> Ejecuta la validación de setup y delega en el método original.
>
> Args:
>     self (Scaffold): Instancia propietaria del método decorado.
>     *args (Any): Argumentos posicionales reenviados a `f`.
>     **kwargs (Any): Argumentos nombrados reenviados a `f`.
>
> Returns:
>     Any: El valor retornado por `f(self, *args, **kwargs)`.
>
> Raises:
>     NotImplementedError: Si la subclase no implementa
>         `_check_setup_finished()`.
>     AssertionError: Si la subclase concreta determina que la fase de
>         setup ya finalizó (comportamiento definido por la subclase).
> ```

**Parámetros**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `self` | `Scaffold` | Sí | Instancia sobre la que se invoca el método. |
| `*args` | `Any` | No | Argumentos posicionales reenviados. |
| `**kwargs` | `Any` | No | Argumentos nombrados reenviados. |

**Retorno**

| Tipo | Descripción |
|---|---|
| `Any` | Resultado del método original `f`. |

---

#### `_endpoint_from_view_func()`

```python
def _endpoint_from_view_func(view_func: ft.RouteCallable) -> str
```

> **Docstring original/propuesto**
>
> ```
> Helper interno que devuelve el endpoint por defecto de una función.
>
> El endpoint por defecto siempre es el nombre de la función de vista.
>
> Args:
>     view_func (ft.RouteCallable): Función de vista. No puede ser None.
>
> Returns:
>     str: `view_func.__name__`.
>
> Raises:
>     AssertionError: Si `view_func` es None.
> ```

**Parámetros**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `view_func` | `ft.RouteCallable` | Sí | Función de vista registrada. |

**Retorno**

| Tipo | Descripción |
|---|---|
| `str` | Nombre de la función (`__name__`). |

**Raises**

| Excepción | Condición |
|---|---|
| `AssertionError` | `view_func is None` (mensaje: `"expected view func if endpoint is not provided."`). |

---

#### `_find_package_path()`

```python
def _find_package_path(import_name: str) -> str
```

> **Docstring propuesto**
>
> ```
> Encuentra el directorio que contiene el paquete o módulo indicado.
>
> Resuelve el *spec* del módulo raíz mediante `importlib.util.find_spec`.
> Soporta tres escenarios:
>   * Paquete *namespace*: localiza la ruta del namespace que contiene el
>     submódulo solicitado (vía `os.path.commonpath` y
>     `pathlib.Path.is_relative_to`), o la primera ruta disponible.
>   * Paquete regular (con `__init__.py`): dos niveles por encima de
>     `origin`.
>   * Módulo simple: el directorio de `origin`.
>
> Args:
>     import_name (str): Nombre de importación, p. ej. `"mi_app.web"`.
>
> Returns:
>     str: Ruta del directorio contenedor. Si el módulo no puede
>         resolverse, devuelve `os.getcwd()`.
> ```

**Parámetros**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `import_name` | `str` | Sí | Nombre de importación; solo se usa su componente raíz (`partition(".")`) para el *spec* inicial. |

**Retorno**

| Tipo | Descripción |
|---|---|
| `str` | Directorio contenedor del paquete/módulo, o CWD si no se resuelve. |

**Raises**

| Excepción | Condición |
|---|---|
| — | No propaga excepciones. `ImportError` y `ValueError` (nombre inválido, `__main__`, o `root_spec is None`) se capturan y derivan en el retorno de `os.getcwd()`. |

> **Nota de implementación:** el `ValueError("not found")` se lanza internamente solo como mecanismo de control de flujo y es capturado en el mismo bloque `try/except`.

---

#### `find_package()`

```python
def find_package(import_name: str) -> tuple[str | None, str]
```

> **Docstring original/propuesto**
>
> ```
> Encuentra el prefijo de instalación de un paquete y su ruta de import.
>
> El prefijo es el directorio que contiene la jerarquía estándar
> (lib, bin, etc.). Si el paquete no está instalado en el sistema
> (`sys.prefix`) ni en un virtualenv (`site-packages`), el prefijo es
> `None`.
>
> La ruta es la entrada de `sys.path` desde la que se importaría el
> paquete. Si el paquete no está instalado, se asume el directorio de
> trabajo actual.
>
> Args:
>     import_name (str): Nombre de importación del paquete.
>
> Returns:
>     tuple[str | None, str]: Par `(prefix, package_path)`.
> ```

**Parámetros**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `import_name` | `str` | Sí | Nombre de importación del paquete a localizar. |

**Retorno**

| Escenario detectado | Valor retornado |
|---|---|
| Instalado en el sistema (`package_path` relativo a `sys.prefix`) | `(py_prefix, package_path)` |
| Virtualenv Windows (`prefix/lib/site-packages`) | `(parent, package_path)` |
| Virtualenv Unix (`prefix/lib/pythonX.Y/site-packages`) | `(os.path.dirname(parent), package_path)` |
| Otro layout con `site-packages` (`prefix/site-packages`) | `(site_parent, package_path)` |
| No instalado | `(None, package_path)` |

**Raises**

| Excepción | Condición |
|---|---|
| — | No lanza excepciones propias. |

---

### 2.3 Clase `Scaffold`

> **Docstring (original)**
>
> ```
> Comportamiento común compartido entre `flask.Flask` y
> `flask.blueprints.Blueprint`.
>
> .. versionadded:: 2.0
> ```

**Atributos de clase**

| Nombre | Tipo | Descripción |
|---|---|---|
| `cli` | `Group` (anotación, `TYPE_CHECKING`) | Grupo de comandos Click asociado. |
| `name` | `str` | Nombre del objeto; definido por la subclase. |
| `_static_folder` | `str \| None` | Almacenamiento interno del *setter* `static_folder`. Default `None`. |
| `_static_url_path` | `str \| None` | Almacenamiento interno del *setter* `static_url_path`. Default `None`. |

---

#### `Scaffold.__init__()`

```python
def __init__(
    self,
    import_name: str,
    static_folder: str | os.PathLike[str] | None = None,
    static_url_path: str | None = None,
    template_folder: str | os.PathLike[str] | None = None,
    root_path: str | None = None,
) -> None
```

**Parámetros**

| Nombre | Tipo | Requerido | Default | Descripción |
|---|---|---|---|---|
| `import_name` | `str` | Sí | — | Nombre de importación del módulo donde se define el objeto; normalmente `__name__`. |
| `static_folder` | `str \| os.PathLike[str] \| None` | No | `None` | Carpeta de archivos estáticos a servir. Si se define, se añade una ruta estática. |
| `static_url_path` | `str \| None` | No | `None` | Prefijo URL para la ruta estática. |
| `template_folder` | `str \| os.PathLike[str] \| None` | No | `None` | Carpeta de plantillas; si se define, se añade un *loader* Jinja. |
| `root_path` | `str \| None` | No | `None` | Ruta base para estáticos, plantillas y recursos. Si es `None`, se descubre con `get_root_path(import_name)`. |

**Retorno**

| Tipo | Descripción |
|---|---|
| `None` | Constructor. |

**Atributos de instancia inicializados**

| Atributo | Tipo | Estructura / Default |
|---|---|---|
| `import_name` | `str` | Valor del parámetro. Inmutable tras construcción. |
| `static_folder` | *property* | Delegado al *setter*. |
| `static_url_path` | *property* | Delegado al *setter*. |
| `template_folder` | `str \| os.PathLike[str] \| None` | Valor del parámetro. |
| `root_path` | `str` | Parámetro o `get_root_path(import_name)`. |
| `view_functions` | `dict[str, ft.RouteCallable]` | `{}` — mapea endpoint → vista. **Interno.** |
| `error_handler_spec` | `dict[AppOrBlueprintKey, dict[int \| None, dict[type[Exception], ErrorHandlerCallable]]]` | `defaultdict(lambda: defaultdict(dict))` — `{scope: {code: {class: handler}}}`. **Interno.** |
| `before_request_funcs` | `dict[AppOrBlueprintKey, list[BeforeRequestCallable]]` | `defaultdict(list)`. **Interno.** |
| `after_request_funcs` | `dict[AppOrBlueprintKey, list[AfterRequestCallable[Any]]]` | `defaultdict(list)`. **Interno.** |
| `teardown_request_funcs` | `dict[AppOrBlueprintKey, list[TeardownCallable]]` | `defaultdict(list)`. **Interno.** |
| `template_context_processors` | `dict[AppOrBlueprintKey, list[TemplateContextProcessorCallable]]` | `defaultdict(list, {None: [_default_template_ctx_processor]})`. **Interno.** |
| `url_value_preprocessors` | `dict[AppOrBlueprintKey, list[URLValuePreprocessorCallable]]` | `defaultdict(list)`. **Interno.** |
| `url_default_functions` | `dict[AppOrBlueprintKey, list[URLDefaultCallable]]` | `defaultdict(list)`. **Interno.** |

**Raises**

| Excepción | Condición |
|---|---|
| Propagadas de `get_root_path()` | Si `root_path is None` y la resolución del `import_name` falla. |

---

#### `Scaffold.__repr__()`

```python
def __repr__(self) -> str
```

| Aspecto | Detalle |
|---|---|
| **Parámetros** | Ninguno (solo `self`). |
| **Retorno** | `str` — `f"<{type(self).__name__} {self.name!r}>"`. |
| **Raises** | `AttributeError` si `name` no ha sido definido por la subclase. |

---

#### `Scaffold._check_setup_finished()`

```python
def _check_setup_finished(self, f_name: str) -> None
```

> **Docstring propuesto**
>
> ```
> Hook abstracto invocado por `setupmethod()` antes de cada mutación.
>
> Las subclases deben implementarlo y abortar si la fase de setup ya
> terminó.
>
> Args:
>     f_name (str): Nombre del método de setup que se intenta ejecutar.
>
> Returns:
>     None
>
> Raises:
>     NotImplementedError: Siempre, en la clase base.
> ```

**Parámetros**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `f_name` | `str` | Sí | Nombre del método decorado con `setupmethod()`. |

**Raises**

| Excepción | Condición |
|---|---|
| `NotImplementedError` | Siempre en `Scaffold`; método abstracto. |

---

#### `Scaffold.static_folder` *(property — getter)*

```python
@property
def static_folder(self) -> str | None
```

| Aspecto | Detalle |
|---|---|
| **Docstring** | Ruta absoluta a la carpeta estática configurada; `None` si no hay carpeta estática. |
| **Parámetros** | Ninguno. |
| **Retorno** | `str \| None` — `os.path.join(self.root_path, self._static_folder)` o `None`. |
| **Raises** | — |

---

#### `Scaffold.static_folder` *(setter)*

```python
@static_folder.setter
def static_folder(self, value: str | os.PathLike[str] | None) -> None
```

**Parámetros**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `value` | `str \| os.PathLike[str] \| None` | Sí | Ruta a normalizar. Si no es `None`, se convierte con `os.fspath()` y se eliminan separadores finales (`\` y `/`). |

| Aspecto | Detalle |
|---|---|
| **Retorno** | `None`. Asigna `self._static_folder`. |
| **Raises** | `TypeError` desde `os.fspath()` si `value` no es `str` ni implementa `__fspath__`. |

---

#### `Scaffold.has_static_folder` *(property)*

```python
@property
def has_static_folder(self) -> bool
```

| Aspecto | Detalle |
|---|---|
| **Docstring** | `True` si `static_folder` está configurado. *(versionadded 0.5)* |
| **Parámetros** | Ninguno. |
| **Retorno** | `bool` — `self.static_folder is not None`. |
| **Raises** | — |

---

#### `Scaffold.static_url_path` *(property — getter)*

```python
@property
def static_url_path(self) -> str | None
```

| Aspecto | Detalle |
|---|---|
| **Docstring** | Prefijo URL desde el que será accesible la ruta estática. Si no se configuró en el init, se deriva de `static_folder`. |
| **Parámetros** | Ninguno. |
| **Retorno** | `str \| None` — `self._static_url_path` si está definido; en su defecto `f"/{os.path.basename(static_folder)}".rstrip("/")`; `None` si no hay carpeta estática. |
| **Raises** | — |

---

#### `Scaffold.static_url_path` *(setter)*

```python
@static_url_path.setter
def static_url_path(self, value: str | None) -> None
```

**Parámetros**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `value` | `str \| None` | Sí | Prefijo URL. Si no es `None`, se le aplica `rstrip("/")`. |

| Aspecto | Detalle |
|---|---|
| **Retorno** | `None`. Asigna `self._static_url_path`. |
| **Raises** | `AttributeError` si `value` no es `str` ni `None`. |

---

#### `Scaffold.jinja_loader` *(cached_property)*

```python
@cached_property
def jinja_loader(self) -> BaseLoader | None
```

| Aspecto | Detalle |
|---|---|
| **Docstring** | Loader Jinja para las plantillas del objeto. Por defecto, un `jinja2.loaders.FileSystemLoader` apuntando a `template_folder` si está definido. *(versionadded 0.5)* |
| **Parámetros** | Ninguno. |
| **Retorno** | `BaseLoader \| None` — `FileSystemLoader(os.path.join(root_path, template_folder))` o `None`. |
| **Raises** | — |
| **Nota** | Resultado memoizado por `werkzeug.utils.cached_property`; cambios posteriores en `template_folder` no se reflejan salvo invalidación del atributo en `__dict__`. |

---

#### `Scaffold._method_route()`

```python
def _method_route(
    self,
    method: str,
    rule: str,
    options: dict[str, t.Any],
) -> t.Callable[[T_route], T_route]
```

> **Docstring propuesto**
>
> ```
> Helper interno que construye un decorador de ruta para un único
> método HTTP.
>
> Args:
>     method (str): Método HTTP (p. ej. "GET", "POST").
>     rule (str): Regla de URL.
>     options (dict[str, Any]): Opciones extra para `Rule`. No debe
>         contener la clave "methods".
>
> Returns:
>     Callable[[T_route], T_route]: Decorador equivalente a
>         `self.route(rule, methods=[method], **options)`.
>
> Raises:
>     TypeError: Si `options` contiene la clave "methods".
> ```

**Parámetros**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `method` | `str` | Sí | Verbo HTTP a asociar. |
| `rule` | `str` | Sí | Regla de URL. |
| `options` | `dict[str, Any]` | Sí | Opciones adicionales de `werkzeug.routing.Rule`. |

**Retorno**

| Tipo | Descripción |
|---|---|
| `Callable[[T_route], T_route]` | Decorador producido por `route()`. |

**Raises**

| Excepción | Condición |
|---|---|
| `TypeError` | `"methods" in options` → `"Use the 'route' decorator to use the 'methods' argument."` |

---

#### Atajos por verbo HTTP: `get()`, `query()`, `post()`, `put()`, `delete()`, `patch()`

Todos comparten firma y semántica, decorados con `@setupmethod`:

```python
@setupmethod
def get(self, rule: str, **options: t.Any) -> t.Callable[[T_route], T_route]
```

| Método | Verbo HTTP delegado | Disponible desde |
|---|---|---|
| `get()` | `"GET"` | 2.0 |
| `query()` | `"QUERY"` | 3.2 |
| `post()` | `"POST"` | 2.0 |
| `put()` | `"PUT"` | 2.0 |
| `delete()` | `"DELETE"` | 2.0 |
| `patch()` | `"PATCH"` | 2.0 |

**Parámetros (comunes)**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `rule` | `str` | Sí | Regla de URL. |
| `**options` | `Any` | No | Opciones extra para `Rule`. No admite `methods`. |

**Retorno (común)**

| Tipo | Descripción |
|---|---|
| `Callable[[T_route], T_route]` | Decorador de registro de vista. |

**Raises (comunes)**

| Excepción | Condición |
|---|---|
| `TypeError` | Se pasó `methods` en `**options` (desde `_method_route()`). |
| `NotImplementedError` / error de setup | Desde `_check_setup_finished()` inyectado por `setupmethod()`. |

---

#### `Scaffold.route()`

```python
@setupmethod
def route(self, rule: str, **options: t.Any) -> t.Callable[[T_route], T_route]
```

> **Docstring (original)**
>
> Decora una función de vista para registrarla con la regla URL y opciones dadas. Invoca `add_url_rule()`. El nombre de endpoint por defecto es el nombre de la función de vista si no se pasa `endpoint`. `methods` por defecto es `["GET"]`; `HEAD` y `OPTIONS` se añaden automáticamente.

**Parámetros**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `rule` | `str` | Sí | Cadena de regla de URL. |
| `**options` | `Any` | No | Opciones extra para `werkzeug.routing.Rule`. Se extrae `endpoint` si está presente. |

**Retorno**

| Tipo | Descripción |
|---|---|
| `Callable[[T_route], T_route]` | Función `decorator` que registra la vista y la devuelve sin modificar. |

**Raises**

| Excepción | Condición |
|---|---|
| Propagadas de `add_url_rule()` | P. ej. `NotImplementedError` en `Scaffold` base, o `AssertionError` por endpoint duplicado en subclases concretas. |

---

##### `decorator()` *(función anidada en `route()`)*

```python
def decorator(f: T_route) -> T_route
```

| Aspecto | Detalle |
|---|---|
| **Propósito** | Extrae `endpoint` de `options` mediante `options.pop("endpoint", None)` e invoca `self.add_url_rule(rule, endpoint, f, **options)`. |
| **Parámetros** | `f` (`T_route`, requerido): función de vista a registrar. |
| **Retorno** | `T_route` — la misma función `f`, sin envolver. |
| **Raises** | Las propagadas por `add_url_rule()`. |

---

#### `Scaffold.add_url_rule()`

```python
@setupmethod
def add_url_rule(
    self,
    rule: str,
    endpoint: str | None = None,
    view_func: ft.RouteCallable | None = None,
    provide_automatic_options: bool | None = None,
    **options: t.Any,
) -> None
```

> **Docstring (original, resumido)**
>
> Registra una regla para enrutar peticiones entrantes y construir URLs. `route()` es un atajo que invoca este método con `view_func`. Si `view_func` tiene atributo `required_methods`, esos métodos se añaden a los pasados y automáticos; si tiene `provide_automatic_methods`, se usa como default cuando el parámetro no se pasa. **Método abstracto en `Scaffold`.**

**Parámetros**

| Nombre | Tipo | Requerido | Default | Descripción |
|---|---|---|---|---|
| `rule` | `str` | Sí | — | Cadena de regla de URL. |
| `endpoint` | `str \| None` | No | `None` | Nombre de endpoint a asociar. Por defecto `view_func.__name__`. |
| `view_func` | `ft.RouteCallable \| None` | No | `None` | Función de vista asociada al endpoint. |
| `provide_automatic_options` | `bool \| None` | No | `None` | Añade el método `OPTIONS` y responde automáticamente a peticiones `OPTIONS`. |
| `**options` | `Any` | No | — | Opciones extra para `werkzeug.routing.Rule` (`methods`, `defaults`, etc.). |

**Retorno**

| Tipo | Descripción |
|---|---|
| `None` | Registro con efecto lateral. |

**Raises**

| Excepción | Condición |
|---|---|
| `NotImplementedError` | Siempre en `Scaffold`; debe sobrescribirse en subclases. |

---

#### `Scaffold.endpoint()`

```python
@setupmethod
def endpoint(self, endpoint: str) -> t.Callable[[F], F]
```

> **Docstring (original)**
>
> Decora una función de vista para registrarla en el endpoint dado. Se usa cuando una regla fue añadida sin `view_func` mediante `add_url_rule()`.

**Parámetros**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `endpoint` | `str` | Sí | Nombre de endpoint a asociar con la función de vista. |

**Retorno**

| Tipo | Descripción |
|---|---|
| `Callable[[F], F]` | Función `decorator`. |

**Raises**

| Excepción | Condición |
|---|---|
| Error de setup | Desde `_check_setup_finished()`. |

---

##### `decorator()` *(función anidada en `endpoint()`)*

```python
def decorator(f: F) -> F
```

| Aspecto | Detalle |
|---|---|
| **Propósito** | Asigna `self.view_functions[endpoint] = f`. |
| **Parámetros** | `f` (`F`, requerido): función de vista. |
| **Retorno** | `F` — la misma función `f`. |
| **Raises** | — |

---

#### Hooks de ciclo de vida: tabla comparativa

Todos están decorados con `@setupmethod`, reciben un único callable y lo devuelven sin modificar (patrón decorador idempotente).

| Método | Firma | Estructura mutada | Operación |
|---|---|---|---|
| `before_request()` | `(f: T_before_request) -> T_before_request` | `before_request_funcs` | `setdefault(None, []).append(f)` |
| `after_request()` | `(f: T_after_request) -> T_after_request` | `after_request_funcs` | `setdefault(None, []).append(f)` |
| `teardown_request()` | `(f: T_teardown) -> T_teardown` | `teardown_request_funcs` | `setdefault(None, []).append(f)` |
| `context_processor()` | `(f: T_template_context_processor) -> T_template_context_processor` | `template_context_processors` | `[None].append(f)` |
| `url_value_preprocessor()` | `(f: T_url_value_preprocessor) -> T_url_value_preprocessor` | `url_value_preprocessors` | `[None].append(f)` |
| `url_defaults()` | `(f: T_url_defaults) -> T_url_defaults` | `url_default_functions` | `[None].append(f)` |

**Parámetro común**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `f` | `TypeVar` acotado al callable correspondiente | Sí | Callback a registrar en el ámbito `None` (ámbito propio del objeto). |

**Raises comunes**

| Excepción | Condición |
|---|---|
| Error de setup | Desde `_check_setup_finished()` (`NotImplementedError` en la clase base). |

**Semántica específica de cada hook**

##### `before_request()`
Registra una función a ejecutar antes de cada petición. Se invoca **sin argumentos**. Si retorna un valor distinto de `None`, dicho valor se trata como el retorno de la vista y se detiene el procesamiento posterior. En un *blueprint*, solo aplica a las peticiones que este maneja; para alcance global desde un blueprint usar `Blueprint.before_app_request`.

##### `after_request()`
Registra una función a ejecutar después de cada petición. Recibe el objeto *response* y **debe retornar** un objeto *response*. Si una función lanza excepción, las restantes `after_request` no se ejecutan; para acciones garantizadas usar `teardown_request()`. Equivalente global en blueprints: `Blueprint.after_app_request`.

##### `teardown_request()`
Registra una función invocada al hacer *pop* del contexto de petición (fin de petición o salida del bloque `with app.test_request_context()`). Si hubo una excepción no manejada, recibe el objeto de error; si existe un `errorhandler()` registrado, este consume la excepción y el teardown no la recibe. **Debe evitar lanzar excepciones**; sus valores de retorno se ignoran. Equivalente global: `Blueprint.teardown_app_request`.

##### `context_processor()`
Registra una función ejecutada antes de renderizar una plantilla; las claves del `dict` retornado se inyectan como variables de plantilla. Equivalente global: `Blueprint.app_context_processor`.

##### `url_value_preprocessor()`
Registra una función ejecutada **antes** de las funciones `before_request()`. Recibe el nombre del endpoint y el `dict` de valores capturados de la URL, pudiendo modificarlos *in place* (p. ej. extraer un código de idioma hacia `g`). El valor de retorno se ignora. Equivalente global: `Blueprint.app_url_value_preprocessor`.

##### `url_defaults()`
Registra un callback de valores por defecto para la construcción de URLs. Recibe el endpoint y el `dict` de valores, que debe actualizar *in place*. Equivalente global: `Blueprint.app_url_defaults`.

---

#### `Scaffold.errorhandler()`

```python
@setupmethod
def errorhandler(
    self, code_or_exception: type[Exception] | int
) -> t.Callable[[T_error_handler], T_error_handler]
```

> **Docstring (original, resumido)**
>
> Registra una función para manejar errores por código HTTP o por clase de excepción. *(versionadded 0.7: usar `register_error_handler()` en lugar de modificar `error_handler_spec` directamente; se admiten tipos de excepción arbitrarios no derivados de `HTTPException`.)*

**Parámetros**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `code_or_exception` | `type[Exception] \| int` | Sí | Código HTTP entero o clase de excepción arbitraria. |

**Retorno**

| Tipo | Descripción |
|---|---|
| `Callable[[T_error_handler], T_error_handler]` | Función `decorator`. |

**Raises**

| Excepción | Condición |
|---|---|
| `ValueError` / `TypeError` | Al aplicar el decorador, propagadas desde `_get_exc_class_and_code()`. |
| Error de setup | Desde `_check_setup_finished()`. |

---

##### `decorator()` *(función anidada en `errorhandler()`)*

```python
def decorator(f: T_error_handler) -> T_error_handler
```

| Aspecto | Detalle |
|---|---|
| **Propósito** | Invoca `self.register_error_handler(code_or_exception, f)`. |
| **Parámetros** | `f` (`T_error_handler`, requerido): manejador que recibe el objeto de error. |
| **Retorno** | `T_error_handler` — la misma función `f`. |
| **Raises** | `ValueError`, `TypeError` propagadas desde `register_error_handler()` → `_get_exc_class_and_code()`. |

---

#### `Scaffold.register_error_handler()`

```python
@setupmethod
def register_error_handler(
    self,
    code_or_exception: type[Exception] | int,
    f: ft.ErrorHandlerCallable,
) -> None
```

> **Docstring (original)**
>
> Alternativa no-decorador a `errorhandler()`, más directa para uso imperativo. *(versionadded 0.7)*

**Parámetros**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `code_or_exception` | `type[Exception] \| int` | Sí | Código HTTP o clase de excepción. |
| `f` | `ft.ErrorHandlerCallable` | Sí | Manejador a registrar. |

**Retorno**

| Tipo | Descripción |
|---|---|
| `None` | Escribe en `self.error_handler_spec[None][code][exc_class] = f`. |

**Raises**

| Excepción | Condición |
|---|---|
| `ValueError` | Código HTTP no reconocido, o clase no subclase de `Exception`. |
| `TypeError` | Se pasó una *instancia* de excepción en lugar de una clase. |
| Error de setup | Desde `_check_setup_finished()`. |

---

#### `Scaffold._get_exc_class_and_code()` *(staticmethod)*

```python
@staticmethod
def _get_exc_class_and_code(
    exc_class_or_code: type[Exception] | int,
) -> tuple[type[Exception], int | None]
```

> **Docstring (original)**
>
> ```
> Obtiene la clase de excepción manejada. Para códigos de estado HTTP o
> subclases de `HTTPException`, devuelve tanto la excepción como el
> código de estado.
>
> Args:
>     exc_class_or_code (type[Exception] | int): Cualquier clase de
>         excepción, o un código de estado HTTP como entero.
>
> Returns:
>     tuple[type[Exception], int | None]: Par `(exc_class, code)`. `code`
>         es `None` para excepciones que no derivan de `HTTPException`.
> ```

**Parámetros**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `exc_class_or_code` | `type[Exception] \| int` | Sí | Clase de excepción o código HTTP. Si es `int`, se resuelve vía `werkzeug.exceptions.default_exceptions`. |

**Retorno**

| Caso | Valor |
|---|---|
| Subclase de `HTTPException` (o `int` válido) | `(exc_class, exc_class.code)` |
| Otra subclase de `Exception` | `(exc_class, None)` |

**Raises**

| Excepción | Condición | Mensaje |
|---|---|---|
| `ValueError` | `int` no presente en `default_exceptions` (`KeyError` re-lanzado con `from None`). | `"'<code>' is not a recognized HTTP error code. Use a subclass of HTTPException with that code instead."` |
| `TypeError` | `isinstance(exc_class, Exception)` → se pasó una instancia. | `"<obj> is an instance, not a class. Handlers can only be registered for Exception classes or HTTP error codes."` |
| `ValueError` | `not issubclass(exc_class, Exception)`. | `"'<name>' is not a subclass of Exception. Handlers can only be registered for Exception classes or HTTP error codes."` |

---

## 3. Ejemplo de uso

El siguiente ejemplo implementa una subclase concreta mínima de `Scaffold` (ya que `add_url_rule()` y `_check_setup_finished()` son abstractos), ejercita los decoradores de registro y las utilidades de descubrimiento de rutas.

```python
from __future__ import annotations

import typing as t

from werkzeug.exceptions import NotFound

from flask.sansio.scaffold import (
    Scaffold,
    find_package,
    setupmethod,
    _endpoint_from_view_func,
)


class MiniApp(Scaffold):
    """Implementación concreta mínima de `Scaffold` para demostración."""

    def __init__(self, import_name: str, **kwargs: t.Any) -> None:
        super().__init__(import_name, **kwargs)
        self.name = import_name
        self.url_map: list[tuple[str, str, list[str]]] = []
        self._setup_finished = False

    # --- Implementación de los hooks abstractos -------------------------
    def _check_setup_finished(self, f_name: str) -> None:
        if self._setup_finished:
            raise AssertionError(
                f"No se puede llamar a '{f_name}' tras finalizar el setup."
            )

    @setupmethod
    def add_url_rule(
        self,
        rule: str,
        endpoint: str | None = None,
        view_func=None,
        provide_automatic_options: bool | None = None,
        **options: t.Any,
    ) -> None:
        if endpoint is None:
            endpoint = _endpoint_from_view_func(view_func)

        if endpoint in self.view_functions and self.view_functions[endpoint] is not view_func:
            raise AssertionError(f"Endpoint duplicado: {endpoint!r}")

        methods = list(options.pop("methods", ["GET"]))
        self.url_map.append((rule, endpoint, methods))

        if view_func is not None:
            self.view_functions[endpoint] = view_func

    def finalize(self) -> None:
        """Cierra la fase de setup; `setupmethod()` bloqueará mutaciones."""
        self._setup_finished = True


# --------------------------------------------------------------------------
# 1. Construcción y configuración de rutas de recursos
# --------------------------------------------------------------------------
app = MiniApp(__name__, static_folder="static/", template_folder="templates")

print(repr(app))                     # <MiniApp '__main__'>
print(app.has_static_folder)         # True
print(app.static_url_path)           # /static  (derivado de static_folder)
print(type(app.jinja_loader))        # <class 'jinja2.loaders.FileSystemLoader'>

# Reasignación vía setters (normaliza separadores finales)
app.static_url_path = "/assets/"
print(app.static_url_path)           # /assets


# --------------------------------------------------------------------------
# 2. Registro de vistas: route() y atajos por verbo HTTP
# --------------------------------------------------------------------------
@app.route("/")
def index():
    return "Hello, World!"


@app.get("/users")
def list_users():
    return "lista de usuarios"


@app.post("/users")
def create_user():
    return "usuario creado", 201


@app.put("/users/<int:uid>")
def replace_user(uid: int):
    return f"usuario {uid} reemplazado"


@app.patch("/users/<int:uid>")
def update_user(uid: int):
    return f"usuario {uid} actualizado"


@app.delete("/users/<int:uid>")
def delete_user(uid: int):
    return "", 204


@app.query("/search")
def search():
    return "resultados"


# Regla sin view_func + endpoint() para asociarla después
app.add_url_rule("/about", endpoint="about")


@app.endpoint("about")
def about():
    return "Acerca de"


# --------------------------------------------------------------------------
# 3. Hooks de ciclo de vida y procesadores de contexto
# --------------------------------------------------------------------------
@app.before_request
def open_resources():
    print("-> before_request")


@app.after_request
def add_header(response):
    print("-> after_request")
    return response


@app.teardown_request
def close_resources(exc: BaseException | None = None):
    print(f"-> teardown_request (exc={exc!r})")


@app.context_processor
def inject_globals() -> dict[str, t.Any]:
    return {"site_name": "MiniApp"}


@app.url_value_preprocessor
def pull_lang(endpoint: str | None, values: dict[str, t.Any] | None) -> None:
    if values is not None:
        values.pop("lang_code", None)


@app.url_defaults
def set_lang(endpoint: str, values: dict[str, t.Any]) -> None:
    values.setdefault("lang_code", "es")


# --------------------------------------------------------------------------
# 4. Manejo de errores: errorhandler(), register_error_handler()
# --------------------------------------------------------------------------
@app.errorhandler(404)
def page_not_found(error):
    return "Esta página no existe", 404


class DatabaseError(Exception):
    pass


def db_failure(error):
    return "Fallo de conexión a la base de datos", 500


app.register_error_handler(DatabaseError, db_failure)

# Inspección de la estructura interna `error_handler_spec`
print(app.error_handler_spec[None][404][NotFound].__name__)   # page_not_found
print(app.error_handler_spec[None][None][DatabaseError].__name__)  # db_failure

# `_get_exc_class_and_code()` resuelve códigos y clases
print(Scaffold._get_exc_class_and_code(404))            # (<class NotFound>, 404)
print(Scaffold._get_exc_class_and_code(DatabaseError))  # (<class DatabaseError>, None)

# Errores de validación esperados
try:
    Scaffold._get_exc_class_and_code(999)
except ValueError as exc:
    print("ValueError:", exc)

try:
    Scaffold._get_exc_class_and_code(DatabaseError())  # instancia, no clase
except TypeError as exc:
    print("TypeError:", exc)


# --------------------------------------------------------------------------
# 5. Restricciones: `methods` en atajos y cierre del setup
# --------------------------------------------------------------------------
try:
    app.get("/bad", methods=["POST"])
except TypeError as exc:
    print("TypeError:", exc)  # Use the 'route' decorator to use the 'methods' argument.

app.finalize()

try:
    @app.route("/tarde")
    def too_late():
        return "no registrado"
except AssertionError as exc:
    print("AssertionError:", exc)


# --------------------------------------------------------------------------
# 6. Descubrimiento de rutas de paquetes
# --------------------------------------------------------------------------
prefix, path = find_package("json")
print("prefix:", prefix)
print("path:  ", path)

prefix, path = find_package("paquete_inexistente_xyz")
print(prefix, path)  # None <cwd>
```

### Salida esperada (fragmento representativo)

```
<MiniApp '__main__'>
True
/static
<class 'jinja2.loaders.FileSystemLoader'>
/assets
page_not_found
db_failure
(<class 'werkzeug.exceptions.NotFound'>, 404)
(<class '__main__.DatabaseError'>, None)
ValueError: '999' is not a recognized HTTP error code. Use a subclass of HTTPException with that code instead.
TypeError: DatabaseError() is an instance, not a class. Handlers can only be registered for Exception classes or HTTP error codes.
TypeError: Use the 'route' decorator to use the 'methods' argument.
AssertionError: No se puede llamar a 'route' tras finalizar el setup.
```

---

## Apéndice: Dependencias externas utilizadas

| Importación | Uso en el módulo |
|---|---|
| `importlib.util.find_spec` | Resolución de *specs* en `_find_package_path()`. |
| `os`, `pathlib`, `sys` | Manipulación y normalización de rutas; `sys.prefix` en `find_package()`. |
| `collections.defaultdict` | Estructuras de registro en `__init__()`. |
| `functools.update_wrapper` | Preservación de metadatos en `setupmethod()`. |
| `jinja2.BaseLoader`, `jinja2.FileSystemLoader` | Tipo de retorno y construcción de `jinja_loader`. |
| `werkzeug.exceptions.default_exceptions`, `HTTPException` | Resolución de códigos en `_get_exc_class_and_code()`. |
| `werkzeug.utils.cached_property` | Memoización de `jinja_loader`. |
| `..typing as ft` | Alias de tipos de callables y claves de ámbito. |
| `..helpers.get_root_path` | Descubrimiento de `root_path` en `__init__()`. |
| `..templating._default_template_ctx_processor` | Procesador de contexto preinstalado en el ámbito `None`. |
| `click.Group` | Anotación de `cli` (solo bajo `TYPE_CHECKING`). |