# Documentación Técnica — `flask/sansio/scaffold.py`

## 1. Descripción general

Este módulo define la clase base **`Scaffold`**, que encapsula el comportamiento común compartido entre los dos objetos centrales de Flask: la aplicación (`flask.Flask`) y los blueprints (`flask.Blueprint`).

`Scaffold` no es utilizable por sí misma (es una clase semi-abstracta): define la estructura de datos para registrar rutas, manejadores de errores, hooks de ciclo de vida de la petición y procesadores de contexto de plantillas, además de resolver rutas del sistema de archivos (carpeta estática, carpeta de plantillas y `root_path`). Las subclases deben implementar `add_url_rule()` y `_check_setup_finished()`.

El módulo pertenece al paquete `sansio` ("sans I/O"), es decir, contiene lógica **independiente del transporte/WSGI**: solo registro y configuración, sin manejo real de peticiones.

Adicionalmente, el archivo expone utilidades a nivel de módulo para descubrir la ubicación en disco de un paquete/módulo importado (`find_package`, `_find_package_path`) y un helper interno para derivar el nombre de un *endpoint* a partir de una función de vista.

---

## 2. Dependencias principales

| Dependencia | Uso |
|---|---|
| `importlib.util`, `pathlib`, `os`, `sys` | Resolución de rutas de paquetes y archivos. |
| `collections.defaultdict` | Estructuras de registro anidadas por *scope*. |
| `functools.update_wrapper` | Preservación de metadatos en el decorador `setupmethod`. |
| `jinja2.BaseLoader`, `FileSystemLoader` | Cargador de plantillas. |
| `werkzeug.exceptions.default_exceptions`, `HTTPException` | Mapeo de códigos HTTP a clases de excepción. |
| `werkzeug.utils.cached_property` | Cacheo del `jinja_loader`. |
| `flask.typing` (`ft`), `flask.helpers.get_root_path`, `flask.templating._default_template_ctx_processor` | Tipos internos y utilidades de Flask. |

---

## 3. Componentes a nivel de módulo

### 3.1 Constantes y variables de tipo

- **`_sentinel`**: objeto centinela único usado como valor por defecto de parámetros, para distinguir "no pasado" de `None`.
- **`F`**: `TypeVar` genérico acotado a cualquier callable.
- **`T_after_request`, `T_before_request`, `T_error_handler`, `T_teardown`, `T_template_context_processor`, `T_url_defaults`, `T_url_value_preprocessor`, `T_route`**: `TypeVar`s acotados a los protocolos de callables definidos en `flask.typing`. Permiten que los decoradores devuelvan **exactamente** el mismo tipo que reciben, preservando la firma de la función decorada para los verificadores estáticos.

### 3.2 `setupmethod(f: F) -> F`

Decorador que envuelve métodos de configuración (`route`, `before_request`, `errorhandler`, etc.). Antes de ejecutar el método, invoca `self._check_setup_finished(f_name)`, lo que permite a las subclases **impedir registros tardíos** (por ejemplo, registrar una ruta después de que la aplicación ya atendió su primera petición). Usa `update_wrapper` para conservar nombre, docstring y anotaciones del método original.

---

## 4. Clase `Scaffold`

### 4.1 Propósito

Base común para `Flask` y `Blueprint`. Centraliza:
1. Metadatos de localización (`import_name`, `root_path`, `static_folder`, `template_folder`).
2. Registros internos de vistas, hooks y manejadores de error.
3. Los decoradores públicos de registro.

### 4.2 Constructor

```python
Scaffold(
    import_name: str,
    static_folder: str | os.PathLike[str] | None = None,
    static_url_path: str | None = None,
    template_folder: str | os.PathLike[str] | None = None,
    root_path: str | None = None,
)
```

| Parámetro | Descripción |
|---|---|
| `import_name` | Nombre del módulo/paquete donde se define el objeto (normalmente `__name__`). Se usa para descubrir `root_path`. |
| `static_folder` | Carpeta de archivos estáticos a servir. Si se define, se añade una ruta estática. |
| `static_url_path` | Prefijo de URL de la ruta estática. Si se omite, se deriva de `static_folder`. |
| `template_folder` | Carpeta de plantillas; si se define, se crea un cargador Jinja. |
| `root_path` | Ruta absoluta base para estáticos, plantillas y recursos. Si es `None`, se calcula con `get_root_path(import_name)`. |

### 4.3 Atributos de clase

| Atributo | Tipo | Descripción |
|---|---|---|
| `cli` | `click.Group` | Grupo de comandos CLI asociado (poblado por las subclases). |
| `name` | `str` | Nombre del objeto (app o blueprint). Definido por las subclases. |
| `_static_folder`, `_static_url_path` | `str \| None` | Almacenamiento interno de las propiedades correspondientes. |

### 4.4 Estructuras de registro (atributos de instancia)

Todas son **internas**: no deben modificarse directamente y su formato puede cambiar.

| Atributo | Formato | Descripción |
|---|---|---|
| `view_functions` | `{endpoint: view}` | Mapa de nombre de endpoint a función de vista. |
| `error_handler_spec` | `{scope: {code: {clase_exc: handler}}}` | Manejadores de error. `scope` = nombre del blueprint o `None` (global); `code` = código HTTP o `None` para excepciones no-HTTP. |
| `before_request_funcs` | `{scope: [funcs]}` | Funciones ejecutadas antes de cada petición. |
| `after_request_funcs` | `{scope: [funcs]}` | Funciones ejecutadas tras cada petición (reciben y devuelven la respuesta). |
| `teardown_request_funcs` | `{scope: [funcs]}` | Funciones ejecutadas al destruir el contexto de petición, incluso ante excepciones. |
| `template_context_processors` | `{scope: [funcs]}` | Procesadores de contexto de plantillas. Se inicializa con `{None: [_default_template_ctx_processor]}`. |
| `url_value_preprocessors` | `{scope: [funcs]}` | Funciones que modifican los valores extraídos de la URL antes de llamar a la vista. |
| `url_default_functions` | `{scope: [funcs]}` | Funciones que inyectan valores por defecto al construir URLs. |

### 4.5 Propiedades

| Miembro | Tipo | Descripción |
|---|---|---|
| `static_folder` (getter/setter) | `str \| None` | Devuelve la ruta **absoluta** (`root_path` + valor configurado). El setter normaliza con `os.fspath` y elimina separadores finales. |
| `has_static_folder` | `bool` | `True` si hay carpeta estática configurada. *(desde 0.5)* |
| `static_url_path` (getter/setter) | `str \| None` | Prefijo de URL de la ruta estática. Si no se configuró, se deriva del nombre base de `static_folder`. El setter elimina la `/` final. |
| `jinja_loader` | `BaseLoader \| None` | `cached_property` que devuelve un `FileSystemLoader` apuntando a `root_path/template_folder`, o `None` si no hay plantillas. *(desde 0.5)* |

### 4.6 Métodos de registro de rutas

| Método | Descripción |
|---|---|
| `_method_route(method, rule, options)` | Helper interno: delega en `route()` fijando `methods=[method]`. Lanza `TypeError` si se pasa `methods` explícitamente. |
| `get`, `post`, `put`, `delete`, `patch` | Atajos de `route()` con el método HTTP correspondiente. *(desde 2.0)* |
| `query` | Atajo de `route()` con `methods=["QUERY"]`. *(desde 3.2)* |
| `route(rule, **options)` | Decorador que registra la función de vista llamando a `add_url_rule`. El endpoint por defecto es el nombre de la función; `methods` por defecto `["GET"]` (+ `HEAD`/`OPTIONS` automáticos). |
| `add_url_rule(rule, endpoint=None, view_func=None, provide_automatic_options=None, **options)` | **Abstracto** (`raise NotImplementedError`). Registra la regla de enrutamiento; implementado por `Flask` y `Blueprint`. |
| `endpoint(endpoint)` | Decorador que asocia una función de vista a un nombre de endpoint ya registrado sin `view_func`. |

### 4.7 Hooks del ciclo de vida de la petición

Todos son decoradores `@setupmethod` que registran la función en el *scope* `None` y la devuelven sin modificar.

| Método | Momento de ejecución / Semántica |
|---|---|
| `before_request(f)` | Antes de cada petición. Sin argumentos; si devuelve un valor no-`None`, se usa como respuesta y se detiene el procesamiento. |
| `after_request(f)` | Después de cada petición. Recibe y debe devolver la respuesta. No garantizado si otra función lanza excepción. |
| `teardown_request(f)` | Al hacer *pop* del contexto de petición, incluso con excepciones (recibe el error si no fue manejado). No debe lanzar excepciones; su valor de retorno se ignora. |
| `context_processor(f)` | Antes de renderizar plantillas; el dict devuelto se añade al contexto. |
| `url_value_preprocessor(f)` | Antes de los `before_request`; recibe `(endpoint, values)` y puede mutar `values`. |
| `url_defaults(f)` | Al generar URLs; recibe `(endpoint, values)` y actualiza `values` in-place. |

> En blueprints, estos hooks solo afectan a las peticiones que el blueprint maneja. Para alcance global desde un blueprint existen las variantes `before_app_request`, `after_app_request`, `teardown_app_request`, `app_context_processor`, `app_url_value_preprocessor`, `app_url_defaults`.

### 4.8 Manejo de errores

| Método | Descripción |
|---|---|
| `errorhandler(code_or_exception)` | Decorador que registra un manejador para un código HTTP (`404`) o una clase de excepción arbitraria. Delega en `register_error_handler`. *(desde 0.7)* |
| `register_error_handler(code_or_exception, f)` | Versión no-decorador. Resuelve clase/código con `_get_exc_class_and_code` y almacena en `error_handler_spec[None][code][exc_class]`. *(desde 0.7)* |
| `_get_exc_class_and_code(exc_class_or_code)` *(staticmethod)* | Normaliza la entrada a `(clase_excepción, código \| None)`. Traduce enteros mediante `werkzeug.exceptions.default_exceptions`. Errores: `ValueError` si el código no es un error HTTP reconocido o la clase no deriva de `Exception`; `TypeError` si se pasa una *instancia* en lugar de una clase. Devuelve `code = exc_class.code` solo para subclases de `HTTPException`. |

### 4.9 Otros métodos

| Método | Descripción |
|---|---|
| `__repr__()` | Representación del tipo `<Flask 'mi_app'>`. |
| `_check_setup_finished(f_name)` | **Abstracto**. Gancho invocado por `setupmethod`; las subclases lanzan excepción si la fase de configuración ya terminó. |

---

## 5. Funciones a nivel de módulo

### 5.1 `_endpoint_from_view_func(view_func) -> str`
Helper interno que devuelve el nombre por defecto del endpoint: siempre `view_func.__name__`. Afirma (`assert`) que la función no sea `None`.

### 5.2 `_find_package_path(import_name) -> str`
Determina el directorio que **contiene** el paquete o módulo indicado. Lógica:

1. Toma el módulo raíz (`import_name.partition(".")`).
2. Busca su `ModuleSpec` con `importlib.util.find_spec`. Si falla (`ImportError`/`ValueError`, nombre inválido, `__main__`, o spec `None`), devuelve `os.getcwd()`.
3. Si el spec tiene `submodule_search_locations`:
   - **Namespace package** (`origin` es `None` o `"namespace"`): busca el spec del `import_name` completo y elige la ubicación del namespace que contiene el submódulo (vía `os.path.commonpath` + `Path.is_relative_to`); si no se encuentra, toma la primera ubicación. Devuelve su directorio padre.
   - **Paquete regular** (con `__init__.py`): devuelve `dirname(dirname(origin))`.
4. Si no es paquete (módulo simple): devuelve `dirname(origin)`.

### 5.3 `find_package(import_name) -> tuple[str | None, str]`
Devuelve la tupla **`(prefix, package_path)`**:

- **`package_path`**: entrada de `sys.path` desde la que se importaría el paquete (obtenida con `_find_package_path`). Si el paquete no está instalado, se asume el directorio de trabajo actual.
- **`prefix`**: directorio que contiene la jerarquía estándar (`lib`, `bin`, …), o `None` si el paquete no está instalado en el sistema ni en un virtualenv.

Casos contemplados:

| Caso | Resultado del `prefix` |
|---|---|
| Ruta relativa a `sys.prefix` (instalación de sistema) | `os.path.abspath(sys.prefix)` |
| `prefix/lib/site-packages` (Windows) | `prefix` |
| `prefix/lib/pythonX.Y/site-packages` (Unix) | `prefix` |
| `prefix/site-packages` (otro layout) | `site_parent` |
| No instalado | `None` |

---

## 6. Notas de diseño y extensión

- **Patrón plantilla (Template Method)**: `Scaffold` define la API pública y el almacenamiento; las subclases implementan `add_url_rule` y `_check_setup_finished`.
- **Scope `None`**: todos los decoradores de `Scaffold` registran en la clave `None` del diccionario; el proceso de registro de blueprints re-mapea posteriormente esas entradas al nombre del blueprint.
- **Tipado estricto**: el uso de `TypeVar`s acotados garantiza que los decoradores sean transparentes para el análisis estático (la función decorada conserva su tipo exacto).
- **`from __future__ import annotations`**: permite sintaxis de tipos moderna (`str | None`, `dict[...]`) en versiones antiguas de Python al diferir la evaluación de anotaciones.