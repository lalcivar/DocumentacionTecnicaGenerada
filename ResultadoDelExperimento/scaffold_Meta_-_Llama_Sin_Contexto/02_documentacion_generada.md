# Documentación Técnica - `scaffold.py` - Flask

**Archivo:** `flask/scaffold.py`
**Framework:** Flask 2.x / 3.x
**Propósito:** Define la infraestructura base compartida entre la aplicación principal y los Blueprints.

### 1. Descripción General

Este archivo implementa la clase `Scaffold`, que es la clase base abstracta de la que heredan `Flask` y `Blueprint`. Su objetivo es centralizar y reutilizar toda la lógica común de configuración de una aplicación web Flask, evitando duplicación de código.

En términos de arquitectura, `Scaffold` actúa como un **contenedor de registro**. No ejecuta peticiones por sí mismo, sino que proporciona la API declarativa para registrar:

* Rutas y endpoints
* Funciones de ciclo de vida de la petición
* Manejadores de errores
* Procesadores de contexto y plantillas
* Configuración de recursos estáticos y templates

Toda la configuración registrada en `Scaffold` solo puede realizarse durante la fase de *setup* de la aplicación. Una vez que la aplicación ha comenzado a servir peticiones, cualquier intento de registro lanzará un error.

### 2. Dependencias Principales

| Librería | Uso |
| :--- | :--- |
| `jinja2.BaseLoader`, `FileSystemLoader` | Carga de plantillas desde `template_folder` |
| `werkzeug.exceptions.HTTPException` | Manejo de excepciones HTTP para `errorhandler` |
| `werkzeug.utils.cached_property` | Propiedad cacheada para `jinja_loader` |
| `flask.helpers.get_root_path` | Resolución del `root_path` a partir del `import_name` |
| `flask.templating._default_template_ctx_processor` | Procesador de contexto por defecto |
| `collections.defaultdict`, `functools.update_wrapper` | Estructuras de datos y decoradores |

### 3. Componentes Principales

El archivo se compone de 4 bloques lógicos:

1.  **Definiciones de Tipado y Sentinel:** `TypeVars` genéricos para todos los decoradores y un objeto `_sentinel` para valores por defecto.
2.  **Decorador `setupmethod`:** Control de ciclo de vida de la configuración.
3.  **Clase `Scaffold`:** Núcleo del archivo.
4.  **Funciones Auxiliares de Paquete:** `_endpoint_from_view_func`, `_find_package_path` y `find_package` para resolución de rutas en el sistema de archivos.

---

### 4. Decorador `setupmethod(f)`

Es un decorador de métodos que protege la fase de configuración.

**Funcionamiento:**
1. Envuelve la función `f` en `wrapper_func`.
2. Antes de ejecutar `f`, invoca a `self._check_setup_finished(f_name)`.
3. Si la aplicación ya ha finalizado su configuración, este método lanzará una excepción impidiendo el registro tardío.

> Todos los métodos de registro de rutas y hooks están decorados con `@setupmethod`.

### 5. Clase `Scaffold`

Clase central introducida en **Flask 2.0**. Es heredada por `Flask` y `Blueprint`.

#### 5.1 Constructor `__init__`

```python
def __init__(self, import_name, static_folder, static_url_path, template_folder, root_path)
```

| Parámetro | Descripción |
| :--- | :--- |
| `import_name` | Nombre del paquete/módulo donde se define el objeto. Usualmente `__name__`. Se usa para localizar recursos. |
| `static_folder` | Ruta a la carpeta de archivos estáticos. Si se define, se crea una ruta estática. |
| `static_url_path` | Prefijo URL para la ruta estática. |
| `template_folder` | Ruta a la carpeta de plantillas Jinja2. |
| `root_path` | Ruta absoluta base para recursos. Si es `None`, se infiere con `get_root_path(import_name)`. |

Inicializa todas las estructuras internas de registro como `defaultdict` para soportar el scope `None` (global) y el scope por nombre de Blueprint.

#### 5.2 Atributos y Estructuras Internas

| Atributo | Tipo | Descripción |
| :--- | :--- | :--- |
| `view_functions` | `dict[str, RouteCallable]` | Mapeo `endpoint -> view_function`. |
| `error_handler_spec` | `dict[scope, dict[code, dict[Exception, handler]]]` | Especificación de manejadores de error por scope y código HTTP. |
| `before_request_funcs` | `dict[scope, list[callable]]` | Funciones a ejecutar antes de cada request. |
| `after_request_funcs` | `dict[scope, list[callable]]` | Funciones a ejecutar después de cada request. |
| `teardown_request_funcs` | `dict[scope, list[callable]]` | Funciones a ejecutar al destruir el contexto de request, incluso con excepciones. |
| `template_context_processors` | `dict[scope, list[callable]]` | Inyectan variables extra al renderizar plantillas. Inicializado con `_default_template_ctx_processor`. |
| `url_value_preprocessors` | `dict[scope, list[callable]]` | Modifican los valores capturados de la URL antes de pasarlos a la vista. |
| `url_default_functions` | `dict[scope, list[callable]]` | Modifican los argumentos al generar URLs con `url_for`. |

#### 5.3 Propiedades

*   `static_folder -> str | None`: Retorna la ruta absoluta a la carpeta estática. Setter normaliza la ruta con `os.fspath` y elimina barras finales.
*   `has_static_folder -> bool`: Indica si hay una carpeta estática configurada.
*   `static_url_path -> str | None`: Retorna el prefijo URL para estáticos. Si no se configuró, lo deriva del `basename` de `static_folder` como `/{basename}`.
*   `jinja_loader -> BaseLoader | None` (`@cached_property`): Retorna un `FileSystemLoader` apuntando a `template_folder` si está definido. Es cacheado tras el primer acceso.

#### 5.4 Métodos de Ruteo

| Método | Descripción |
| :--- | :--- |
| `route(rule, **options)` | Decorador principal. Registra una view function para una regla URL. Delega en `add_url_rule`. |
| `add_url_rule(rule, endpoint, view_func, ...)` | Método abstracto. Registra una regla en el mapa de ruteo. Lanza `NotImplementedError` y debe ser implementado por `Flask` y `Blueprint`. |
| `endpoint(endpoint)` | Decorador para asociar una view function a un endpoint ya registrado sin `view_func`. |
| `_method_route(method, rule, options)` | Helper interno que valida que no se pase `methods` y llama a `route` con un único método. |
| `get`, `post`, `put`, `delete`, `patch`, `query` | Atajos para `route` con `methods=["GET"]`, `["POST"]`, etc. |

#### 5.5 Métodos de Ciclo de Vida y Hooks

Todos retornan la función decorada para permitir su uso como decorador.

*   `before_request(f)`: Registra función sin argumentos que se ejecuta antes de cada request. Si retorna un valor no nulo, corta el flujo y se usa como respuesta.
*   `after_request(f)`: Registra función que recibe y debe retornar un objeto `response`. Permite modificar la respuesta.
*   `teardown_request(f)`: Registra función que se ejecuta al hacer `pop` del contexto de request. Recibe un objeto de excepción si la hubo. Su valor de retorno es ignorado.
*   `context_processor(f)`: Registra función que retorna un `dict` cuyas claves se inyectan como variables en las plantillas.
*   `url_value_preprocessor(f)`: Registra función `(endpoint, values)` que puede mutar `values` antes del `before_request`.
*   `url_defaults(f)`: Registra función `(endpoint, values)` que inyecta valores por defecto al construir URLs.

#### 5.6 Manejo de Errores

*   `errorhandler(code_or_exception)`: Decorador que registra un handler para un código HTTP `int` o una clase `Exception`.
*   `register_error_handler(code_or_exception, f)`: Versión no decoradora del anterior.
*   `_get_exc_class_and_code(exc_class_or_code) -> tuple[Exception, int | None]`: Método estático que normaliza el parámetro. Si es `int`, lo resuelve a `werkzeug.exceptions.default_exceptions`. Valida que sea una clase y subclase de `Exception`. Si es subclase de `HTTPException`, retorna `(clase, código)`, si no `(clase, None)`.

#### 5.7 Métodos Abstractos

*   `_check_setup_finished(f_name)`: Lanza `NotImplementedError`. Es implementado por las subclases para verificar si la configuración ya finalizó.

### 6. Funciones Auxiliares Externas

#### ` _endpoint_from_view_func(view_func) -> str`
Retorna `view_func.__name__` como nombre de endpoint por defecto. Helper interno usado por `add_url_rule`.

#### `_find_package_path(import_name) -> str`
Resuelve la ruta en el sistema de archivos que contiene el paquete/módulo dado. Maneja casos especiales:
*   Paquetes de tipo *namespace* sin `__init__.py`
*   Paquetes regulares con `__init__.py`
*   Módulos simples
*   Fallback a `os.getcwd()` si el módulo es `__main__` o no se encuentra.

#### `find_package(import_name) -> tuple[str | None, str]`
Determina dónde está instalado un paquete.
*   **Retorna:** `(prefix, package_path)`
*   `prefix`: Directorio base de instalación (`sys.prefix` o `virtualenv`). `None` si no está instalado y se importó desde el directorio de trabajo.
*   `package_path`: Entrada de `sys.path` que contiene el paquete.
*   Detecta instalaciones en `site-packages` tanto en Windows como en Unix.

### 7. Flujo de Uso Típico

```
1. app = Flask(__name__) -> Scaffold.__init__() inicializa estructuras
2. @app.route("/") -> @setupmethod -> _check_setup_finished() -> add_url_rule()
3. @app.before_request -> registra hook en before_request_funcs[None]
4. app.run() -> finaliza fase de setup
5. Cualquier nuevo @app.route() -> _check_setup_finished() lanza AssertionError
```

**En resumen,** `scaffold.py` es el esqueleto arquitectónico de Flask. Abstrae toda la lógica de registro y configuración para que tanto la aplicación como los blueprints compartan una misma API consistente, segura y tipada, delegando solo la implementación específica de ruteo y control de estado a sus subclases.