# Documentación Técnica — Módulo `flask/app.py`

---

## 1. Resumen ejecutivo

Este módulo define la clase `Flask`, la implementación concreta y *síncrona* (WSGI) del framework, que hereda la lógica agnóstica de transporte desde `App` (`flask.sansio.app`). Contiene el ciclo de vida completo de una petición HTTP: creación del contexto, despacho de rutas, pre/post-procesamiento, manejo de errores, generación de respuestas, construcción de URLs y *teardown* de contextos.

Adicionalmente provee dos decoradores de compatibilidad (`remove_ctx()` y `add_ctx()`) y un hook `__init_subclass__()` que detecta subclases que aún usan las firmas antiguas de los métodos del ciclo de vida (sin el parámetro `ctx: AppContext`), emitiendo un `DeprecationWarning` y adaptando automáticamente las llamadas.

---

## 2. Elementos de nivel de módulo

### 2.1 Variables de tipo (`TypeVar`)

| Nombre | Vinculación (`bound`) | Propósito |
|---|---|---|
| `T_shell_context_processor` | `ft.ShellContextProcessorCallable` | Tipado de decoradores de procesadores de contexto de shell. |
| `T_teardown` | `ft.TeardownCallable` | Tipado de callbacks de *teardown*. |
| `T_template_filter` | `ft.TemplateFilterCallable` | Tipado de filtros Jinja. |
| `T_template_global` | `ft.TemplateGlobalCallable` | Tipado de globales Jinja. |
| `T_template_test` | `ft.TemplateTestCallable` | Tipado de tests Jinja. |
| `F` | `t.Callable[..., t.Any]` | Tipado genérico usado por `remove_ctx()` y `add_ctx()`. |

---

### 2.2 `_make_timedelta()`

```python
def _make_timedelta(value: timedelta | int | None) -> timedelta | None
```

Normaliza un valor de configuración temporal a un objeto `datetime.timedelta`. Función auxiliar privada; si el valor ya es `timedelta` o `None`, se devuelve sin modificar; en otro caso se interpreta como segundos.

**Parámetros**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `value` | `timedelta | int | None` | Sí | Valor a normalizar. Un `int` se interpreta como segundos. |

**Retorno**

| Tipo | Descripción |
|---|---|
| `timedelta | None` | `None` si la entrada es `None`; el mismo objeto si ya era `timedelta`; en otro caso `timedelta(seconds=value)`. |

**Raises**

| Excepción | Condición |
|---|---|
| `TypeError` | Propagada por `timedelta()` si `value` no es un numérico válido. |

---

### 2.3 `remove_ctx()`

```python
def remove_ctx(f: F) -> F
```

Decorador de compatibilidad retroactiva. Envuelve un método **sobrescrito por el usuario** que conserva la firma antigua (sin `ctx`). Si el primer argumento posicional recibido es una instancia de `AppContext`, lo descarta antes de invocar la función original. Los metadatos se preservan mediante `functools.update_wrapper`.

**Parámetros**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `f` | `F` (`Callable[..., Any]`) | Sí | Método con firma legada que será envuelto. |

**Retorno**

| Tipo | Descripción |
|---|---|
| `F` | La función `wrapper()` con los metadatos de `f` aplicados. |

**Raises**

| Excepción | Condición |
|---|---|
| — | No lanza excepciones propias; propaga las de `f`. |

#### Función anidada: `wrapper()` (dentro de `remove_ctx()`)

```python
def wrapper(self: Flask, *args: t.Any, **kwargs: t.Any) -> t.Any
```

**Parámetros**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `self` | `Flask` | Sí | Instancia de aplicación sobre la que se invoca el método. |
| `*args` | `t.Any` | No | Argumentos posicionales. Si `args[0]` es `AppContext`, se elimina. |
| `**kwargs` | `t.Any` | No | Argumentos por palabra clave, reenviados sin cambios. |

**Retorno**

| Tipo | Descripción |
|---|---|
| `t.Any` | Resultado de `f(self, *args, **kwargs)`. |

---

### 2.4 `add_ctx()`

```python
def add_ctx(f: F) -> F
```

Decorador de compatibilidad inverso a `remove_ctx()`. Se aplica al **método base de `Flask`** para el caso en que una subclase legada invoque `super().metodo(...)` sin pasar el contexto. Inyecta el `AppContext` activo (`app_ctx._get_current_object()`) como primer argumento posicional cuando falta.

**Parámetros**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `f` | `F` (`Callable[..., Any]`) | Sí | Método base de `Flask` que recibirá el contexto inyectado. |

**Retorno**

| Tipo | Descripción |
|---|---|
| `F` | La función `wrapper()` con los metadatos de `f` aplicados. |

**Raises**

| Excepción | Condición |
|---|---|
| `RuntimeError` | Propagada por `app_ctx._get_current_object()` si no hay contexto de aplicación activo. |

#### Función anidada: `wrapper()` (dentro de `add_ctx()`)

```python
def wrapper(self: Flask, *args: t.Any, **kwargs: t.Any) -> t.Any
```

**Parámetros**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `self` | `Flask` | Sí | Instancia de aplicación. |
| `*args` | `t.Any` | No | Si está vacío o `args[0]` no es `AppContext`, se prefija el contexto activo. |
| `**kwargs` | `t.Any` | No | Argumentos por palabra clave, reenviados sin cambios. |

**Retorno**

| Tipo | Descripción |
|---|---|
| `t.Any` | Resultado de `f(self, *args, **kwargs)`. |

---

## 3. Clase `Flask(App)`

Implementa una aplicación WSGI y actúa como registro central de vistas, reglas de URL, configuración de plantillas, comandos CLI y manejadores de error.

### 3.1 Atributos de clase

| Atributo | Tipo | Valor por defecto | Descripción |
|---|---|---|---|
| `default_config` | `ImmutableDict` | *(ver código)* | Configuración base inmutable (`DEBUG`, `SECRET_KEY`, `SESSION_COOKIE_*`, `MAX_CONTENT_LENGTH`, etc.). |
| `request_class` | `type[Request]` | `Request` | Clase usada para objetos de petición. |
| `response_class` | `type[Response]` | `Response` | Clase usada para objetos de respuesta. |
| `session_interface` | `SessionInterface` | `SecureCookieSessionInterface()` | Interfaz de sesión empleada por la aplicación. |

---

### 3.2 `__init_subclass__()`

```python
def __init_subclass__(cls, **kwargs: t.Any) -> None
```

Hook de creación de subclases. Inspecciona los doce métodos del ciclo de vida que migraron a la firma `(self, ctx: AppContext, ...)`:
`handle_http_exception`, `handle_user_exception`, `handle_exception`, `log_exception`, `dispatch_request`, `full_dispatch_request`, `finalize_request`, `make_default_options_response`, `preprocess_request`, `process_response`, `do_teardown_request`, `do_teardown_appcontext`.

Para cada método sobrescrito, valida mediante `inspect.signature` que el segundo parámetro se llame `ctx` (sin anotación) o esté anotado como `AppContext` (anotación en cadena o clase real). Si la validación falla, emite un `DeprecationWarning` y reemplaza:
- el método de la subclase por `remove_ctx(method)`,
- el método base de `Flask` por `add_ctx(base_method)`.

**Parámetros**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `cls` | `type[Flask]` | Sí (implícito) | Subclase en construcción. |
| `**kwargs` | `t.Any` | No | Argumentos de clase adicionales (aceptados y no utilizados). |

**Retorno**

| Tipo | Descripción |
|---|---|
| `None` | Opera por efecto lateral sobre `cls` y sobre `Flask`. |

**Raises / Warnings**

| Excepción / Aviso | Condición |
|---|---|
| `DeprecationWarning` (emitido, no lanzado) | Un método sobrescrito no declara el parámetro `ctx: AppContext`. |

---

### 3.3 `__init__()`

```python
def __init__(
    self,
    import_name: str,
    static_url_path: str | None = None,
    static_folder: str | os.PathLike[str] | None = "static",
    static_host: str | None = None,
    host_matching: bool = False,
    subdomain_matching: bool = False,
    template_folder: str | os.PathLike[str] | None = "templates",
    instance_path: str | None = None,
    instance_relative_config: bool = False,
    root_path: str | None = None,
)
```

Construye la aplicación: delega en `App.__init__`, crea el grupo de comandos Click (`self.cli = cli.AppGroup()`), lo nombra con `self.name` y, si existe carpeta estática, registra la regla de URL `"{static_url_path}/<path:filename>"` con endpoint `"static"`. La vista se registra mediante una lambda que utiliza una `weakref.ref` a la app para evitar ciclos de referencia.

**Parámetros**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `import_name` | `str` | Sí | Nombre del paquete/módulo de la aplicación. |
| `static_url_path` | `str | None` | No | Ruta web de los archivos estáticos. Por defecto, el nombre de `static_folder`. |
| `static_folder` | `str | os.PathLike[str] | None` | No | Carpeta de archivos estáticos (relativa a `root_path` o absoluta). |
| `static_host` | `str | None` | No | Host de la ruta estática. Obligatorio si `host_matching=True` con `static_folder`. |
| `host_matching` | `bool` | No | Activa `url_map.host_matching`. |
| `subdomain_matching` | `bool` | No | Considera el subdominio relativo a `SERVER_NAME` al enrutar. |
| `template_folder` | `str | os.PathLike[str] | None` | No | Carpeta de plantillas Jinja. |
| `instance_path` | `str | None` | No | Ruta alternativa de la carpeta *instance*. |
| `instance_relative_config` | `bool` | No | Si es `True`, los nombres relativos de configuración se resuelven respecto a `instance_path`. |
| `root_path` | `str | None` | No | Raíz de los archivos de la aplicación (solo si no se autodetecta). |

**Retorno**

| Tipo | Descripción |
|---|---|
| `None` | Constructor. |

**Raises**

| Excepción | Condición |
|---|---|
| `AssertionError` | `bool(static_host) != host_matching` cuando hay carpeta estática configurada. |

#### Función anidada (lambda) de vista estática

```python
lambda **kw: self_ref().send_static_file(**kw)
```

Callback registrado como `view_func` del endpoint `"static"`. Resuelve la referencia débil a la aplicación y delega en `send_static_file()`.

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `**kw` | `t.Any` | No | Variables de ruta capturadas; en la práctica `filename`. |

**Retorno:** `Response` — resultado de `send_static_file()`.

---

### 3.4 `get_send_file_max_age()`

```python
def get_send_file_max_age(self, filename: str | None) -> int | None
```

Determina el valor `max_age` de caché para un archivo servido por `send_file`, a partir de `SEND_FILE_MAX_AGE_DEFAULT`. Si el valor configurado es un `timedelta`, se convierte a segundos enteros.

**Parámetros**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `filename` | `str | None` | Sí | Ruta del archivo evaluado (no se usa en la implementación por defecto). |

**Retorno**

| Tipo | Descripción |
|---|---|
| `int | None` | Segundos de caché, o `None` para usar peticiones condicionales. |

**Raises**

| Excepción | Condición |
|---|---|
| — | No lanza excepciones. |

---

### 3.5 `send_static_file()`

```python
def send_static_file(self, filename: str) -> Response
```

Vista que sirve archivos desde `static_folder`, calculando `max_age` mediante `get_send_file_max_age()` y delegando en `send_from_directory()`.

**Parámetros**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `filename` | `str` | Sí | Ruta del archivo relativa a `static_folder`. |

**Retorno**

| Tipo | Descripción |
|---|---|
| `Response` | Respuesta con el archivo solicitado. |

**Raises**

| Excepción | Condición |
|---|---|
| `RuntimeError` | `static_folder` no está configurada. |
| `NotFound` (HTTP 404) | Propagada por `send_from_directory()` si el archivo no existe. |

---

### 3.6 `open_resource()`

```python
def open_resource(
    self, resource: str, mode: str = "rb", encoding: str | None = None
) -> t.IO[t.AnyStr]
```

Abre, **solo para lectura**, un archivo relativo a `root_path`.

**Parámetros**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `resource` | `str` | Sí | Ruta del recurso relativa a `root_path`. |
| `mode` | `str` | No | Modo de apertura. Valores válidos: `"r"`, `"rt"`, `"rb"`. |
| `encoding` | `str | None` | No | Codificación en modo texto; ignorada en modo binario. |

**Retorno**

| Tipo | Descripción |
|---|---|
| `t.IO[t.AnyStr]` | Objeto archivo abierto. |

**Raises**

| Excepción | Condición |
|---|---|
| `ValueError` | `mode` no pertenece a `{"r", "rt", "rb"}`. |
| `OSError` / `FileNotFoundError` | Propagada por `open()`. |

---

### 3.7 `open_instance_resource()`

```python
def open_instance_resource(
    self, resource: str, mode: str = "rb", encoding: str | None = "utf-8"
) -> t.IO[t.AnyStr]
```

Abre un archivo relativo a `instance_path`. A diferencia de `open_resource()`, admite modos de escritura.

**Parámetros**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `resource` | `str` | Sí | Ruta relativa a `instance_path`. |
| `mode` | `str` | No | Cualquier modo válido de `open()`. Por defecto `"rb"`. |
| `encoding` | `str | None` | No | Codificación en modo texto (por defecto `"utf-8"`); ignorada si `"b"` está en `mode`. |

**Retorno**

| Tipo | Descripción |
|---|---|
| `t.IO[t.AnyStr]` | Objeto archivo abierto. |

**Raises**

| Excepción | Condición |
|---|---|
| `OSError` / `FileNotFoundError` | Propagada por `open()`. |

---

### 3.8 `create_jinja_environment()`

```python
def create_jinja_environment(self) -> Environment
```

Crea el entorno Jinja a partir de `jinja_options`, estableciendo `autoescape` (vía `select_jinja_autoescape`) y `auto_reload` (según `TEMPLATES_AUTO_RELOAD` o `self.debug`). Inyecta los globales `url_for`, `get_flashed_messages`, `config`, `request`, `session`, `g`, y fija la política `json.dumps_function`.

**Parámetros:** ninguno (aparte de `self`).

**Retorno**

| Tipo | Descripción |
|---|---|
| `Environment` | Entorno Jinja configurado. |

**Raises**

| Excepción | Condición |
|---|---|
| — | No lanza excepciones propias. |

---

### 3.9 `create_url_adapter()`

```python
def create_url_adapter(self, request: Request | None) -> MapAdapter | None
```

Crea un `MapAdapter` para la petición dada o, si `request is None`, para un contexto de aplicación usando `SERVER_NAME`, `APPLICATION_ROOT` y `PREFERRED_URL_SCHEME`. Aplica `TRUSTED_HOSTS` y resuelve el host con `get_host()`. Omite `server_name` cuando `host_matching` está activo y fuerza el subdominio por defecto cuando `subdomain_matching` está desactivado.

**Parámetros**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `request` | `Request | None` | Sí | Petición para la que se construye el adaptador; `None` para contexto de aplicación. |

**Retorno**

| Tipo | Descripción |
|---|---|
| `MapAdapter | None` | Adaptador de URLs, o `None` si no hay petición ni `SERVER_NAME` configurado. |

**Raises**

| Excepción | Condición |
|---|---|
| `SecurityError` | Propagada por `get_host()` si el host no está en `trusted_hosts`. |

---

### 3.10 `raise_routing_exception()`

```python
def raise_routing_exception(self, request: Request) -> t.NoReturn
```

Relanza la excepción de enrutamiento almacenada en la petición. En modo depuración, si se trata de un `RequestRedirect` con código distinto de 307/308 y el método no es `GET`/`HEAD`/`OPTIONS`, la sustituye por `FormDataRoutingRedirect` para advertir de la pérdida del cuerpo.

**Parámetros**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `request` | `Request` | Sí | Petición con `routing_exception` establecida. |

**Retorno**

| Tipo | Descripción |
|---|---|
| `t.NoReturn` | Nunca retorna; siempre lanza. |

**Raises**

| Excepción | Condición |
|---|---|
| `request.routing_exception` | Caso general (no depuración, redirecciones 307/308 o métodos seguros). |
| `FormDataRoutingRedirect` | Modo depuración con redirección que descartaría el cuerpo de la petición. |

---

### 3.11 `update_template_context()`

```python
def update_template_context(self, ctx: AppContext, context: dict[str, t.Any]) -> None
```

Actualiza *in place* el contexto de plantilla ejecutando los `template_context_processors` de la app y de los blueprints implicados (en orden inverso). Los valores originales pasados a `render_template` se reaplican al final, de modo que tienen precedencia.

**Parámetros**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `ctx` | `AppContext` | Sí | Contexto activo; si `ctx.has_request`, se incluyen los blueprints de la petición. |
| `context` | `dict[str, t.Any]` | Sí | Diccionario de contexto modificado en el sitio. |

**Retorno**

| Tipo | Descripción |
|---|---|
| `None` | Mutación directa de `context`. |

**Raises**

| Excepción | Condición |
|---|---|
| `Exception` | Propagada desde cualquier procesador de contexto. |

---

### 3.12 `make_shell_context()`

```python
def make_shell_context(self) -> dict[str, t.Any]
```

Construye el espacio de nombres para el shell interactivo (`flask shell`), partiendo de `{"app": self, "g": g}` y aplicando todos los `shell_context_processors`.

**Parámetros:** ninguno.

**Retorno**

| Tipo | Descripción |
|---|---|
| `dict[str, t.Any]` | Diccionario de nombres disponibles en el shell. |

**Raises**

| Excepción | Condición |
|---|---|
| `Exception` | Propagada desde cualquier procesador registrado. |

---

### 3.13 `run()`

```python
def run(
    self,
    host: str | None = None,
    port: int | None = None,
    debug: bool | None = None,
    load_dotenv: bool = True,
    **options: t.Any,
) -> None
```

Arranca el servidor de desarrollo de Werkzeug (`run_simple`). No apto para producción. Si la variable `FLASK_RUN_FROM_CLI` es `"true"`, la llamada se ignora (mostrando un aviso en rojo salvo que se esté en el recargador). Resuelve `host`/`port` desde los argumentos, `SERVER_NAME` o los valores por defecto `127.0.0.1:5000`, y fija `use_reloader`, `use_debugger` y `threaded`. En el bloque `finally` restablece `self._got_first_request = False`.

**Parámetros**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `host` | `str | None` | No | Interfaz de escucha. `"0.0.0.0"` para exposición externa. |
| `port` | `int | None` | No | Puerto de escucha. Por defecto `5000` o el de `SERVER_NAME`. |
| `debug` | `bool | None` | No | Activa/desactiva el modo depuración; tiene precedencia sobre otras fuentes. |
| `load_dotenv` | `bool` | No | Carga los archivos `.env` y `.flaskenv` más cercanos. |
| `**options` | `t.Any` | No | Opciones reenviadas a `werkzeug.serving.run_simple`. |

**Retorno**

| Tipo | Descripción |
|---|---|
| `None` | Bloquea hasta la detención del servidor, o retorna de inmediato si se invoca desde la CLI. |

**Raises**

| Excepción | Condición |
|---|---|
| `OSError` | Propagada por `run_simple` (p. ej. puerto en uso). |
| `ValueError` | `port` no convertible a entero. |

---

### 3.14 `test_client()`

```python
def test_client(self, use_cookies: bool = True, **kwargs: t.Any) -> FlaskClient
```

Crea un cliente de pruebas usando `test_client_class` o, si es `None`, `flask.testing.FlaskClient`.

**Parámetros**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `use_cookies` | `bool` | No | Habilita la persistencia de cookies entre peticiones. |
| `**kwargs` | `t.Any` | No | Argumentos adicionales para el constructor de `test_client_class`. |

**Retorno**

| Tipo | Descripción |
|---|---|
| `FlaskClient` | Cliente de pruebas, utilizable como gestor de contexto. |

**Raises**

| Excepción | Condición |
|---|---|
| `TypeError` | Argumentos incompatibles con el constructor de la clase de cliente. |

---

### 3.15 `test_cli_runner()`

```python
def test_cli_runner(self, **kwargs: t.Any) -> FlaskCliRunner
```

Crea un *runner* para probar comandos CLI, instanciando `test_cli_runner_class` o `flask.testing.FlaskCliRunner`. La aplicación se pasa como primer argumento.

**Parámetros**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `**kwargs` | `t.Any` | No | Argumentos adicionales para el constructor del *runner*. |

**Retorno**

| Tipo | Descripción |
|---|---|
| `FlaskCliRunner` | Runner de pruebas de CLI. |

**Raises**

| Excepción | Condición |
|---|---|
| `TypeError` | Argumentos incompatibles con el constructor. |

---

### 3.16 `handle_http_exception()`

```python
def handle_http_exception(
    self, ctx: AppContext, e: HTTPException
) -> HTTPException | ft.ResponseReturnValue
```

Gestiona una `HTTPException`. Devuelve la excepción sin cambios si `e.code is None` (excepciones proxy) o si es una `RoutingException`. En otro caso busca un manejador con `_find_error_handler()` según los blueprints de la petición y lo ejecuta vía `ensure_sync()`.

**Parámetros**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `ctx` | `AppContext` | Sí | Contexto activo (se usa `ctx.request.blueprints`). |
| `e` | `HTTPException` | Sí | Excepción HTTP a manejar. |

**Retorno**

| Tipo | Descripción |
|---|---|
| `HTTPException | ft.ResponseReturnValue` | La propia excepción, o el valor devuelto por el manejador registrado. |

**Raises**

| Excepción | Condición |
|---|---|
| `Exception` | Propagada si el manejador de error falla. |

---

### 3.17 `handle_user_exception()`

```python
def handle_user_exception(
    self, ctx: AppContext, e: Exception
) -> HTTPException | ft.ResponseReturnValue
```

Punto central de manejo de excepciones de usuario. Activa `show_exception` en `BadRequestKeyError` bajo depuración o `TRAP_BAD_REQUEST_ERRORS`; delega en `handle_http_exception()` cuando `e` es `HTTPException` y no está "atrapada"; en otro caso busca un manejador específico.

**Parámetros**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `ctx` | `AppContext` | Sí | Contexto activo. |
| `e` | `Exception` | Sí | Excepción capturada durante el despacho. |

**Retorno**

| Tipo | Descripción |
|---|---|
| `HTTPException | ft.ResponseReturnValue` | Valor de respuesta producido por el manejador correspondiente. |

**Raises**

| Excepción | Condición |
|---|---|
| *(re-raise desnudo)* | No existe manejador registrado para `e`; se relanza preservando el traceback original. |

---

### 3.18 `handle_exception()`

```python
def handle_exception(self, ctx: AppContext, e: Exception) -> Response
```

Maneja excepciones sin manejador asociado (o lanzadas dentro de un manejador), resultando siempre en un *500 Internal Server Error*. Envía la señal `got_request_exception`. Si `PROPAGATE_EXCEPTIONS` (o, por defecto, `testing`/`debug`) es verdadero, relanza. En caso contrario registra el error con `log_exception()`, construye un `InternalServerError(original_exception=e)`, aplica el manejador de 500 si existe y finaliza la petición con `finalize_request(..., from_error_handler=True)`.

**Parámetros**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `ctx` | `AppContext` | Sí | Contexto activo. |
| `e` | `Exception` | Sí | Excepción no manejada. |

**Retorno**

| Tipo | Descripción |
|---|---|
| `Response` | Respuesta 500 finalizada (o la del manejador registrado). |

**Raises**

| Excepción | Condición |
|---|---|
| `Exception` (`e` o la activa) | `PROPAGATE_EXCEPTIONS` evaluado como verdadero. |

---

### 3.19 `log_exception()`

```python
def log_exception(
    self,
    ctx: AppContext,
    exc_info: tuple[type, BaseException, TracebackType] | tuple[None, None, None],
) -> None
```

Registra la excepción como `ERROR` en `self.logger`, incluyendo ruta y método de la petición.

**Parámetros**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `ctx` | `AppContext` | Sí | Contexto activo; se usan `ctx.request.path` y `ctx.request.method`. |
| `exc_info` | `tuple[type, BaseException, TracebackType] | tuple[None, None, None]` | Sí | Tupla al estilo `sys.exc_info()`. |

**Retorno**

| Tipo | Descripción |
|---|---|
| `None` | Efecto lateral sobre el logger. |

**Raises**

| Excepción | Condición |
|---|---|
| — | No lanza excepciones. |

---

### 3.20 `dispatch_request()`

```python
def dispatch_request(self, ctx: AppContext) -> ft.ResponseReturnValue
```

Realiza el despacho: si hay `routing_exception` invoca `raise_routing_exception()`; si la regla declara `provide_automatic_options` y el método es `OPTIONS`, retorna `make_default_options_response()`; en otro caso invoca la función de vista del endpoint con `**view_args`, envuelta en `ensure_sync()`.

**Parámetros**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `ctx` | `AppContext` | Sí | Contexto activo con la petición enrutada. |

**Retorno**

| Tipo | Descripción |
|---|---|
| `ft.ResponseReturnValue` | Valor crudo devuelto por la vista (no necesariamente un `Response`). |

**Raises**

| Excepción | Condición |
|---|---|
| `HTTPException` / `RoutingException` | Fallo de enrutamiento vía `raise_routing_exception()`. |
| `Exception` | Propagada desde la función de vista. |

---

### 3.21 `full_dispatch_request()`

```python
def full_dispatch_request(self, ctx: AppContext) -> Response
```

Orquesta el ciclo completo: avisa (`DeprecationWarning`) si `should_ignore_error` está definido, marca `_got_first_request`, envía `request_started`, ejecuta `preprocess_request()` y, si no cortocircuita, `dispatch_request()`. Cualquier `Exception` se canaliza a `handle_user_exception()`. El resultado se pasa a `finalize_request()`.

**Parámetros**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `ctx` | `AppContext` | Sí | Contexto de la petición en curso. |

**Retorno**

| Tipo | Descripción |
|---|---|
| `Response` | Respuesta completamente procesada. |

**Raises / Warnings**

| Excepción / Aviso | Condición |
|---|---|
| `DeprecationWarning` (emitido) | `should_ignore_error` definido en la primera petición. |
| `Exception` | Relanzada desde `handle_user_exception()` cuando no hay manejador. |

---

### 3.22 `finalize_request()`

```python
def finalize_request(
    self,
    ctx: AppContext,
    rv: ft.ResponseReturnValue | HTTPException,
    from_error_handler: bool = False,
) -> Response
```

Convierte el valor de retorno en `Response` con `make_response()`, aplica `process_response()` y emite `request_finished`. Con `from_error_handler=True` opera en modo seguro: los fallos del post-procesamiento se registran con `logger.exception` en lugar de propagarse.

**Parámetros**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `ctx` | `AppContext` | Sí | Contexto activo. |
| `rv` | `ft.ResponseReturnValue | HTTPException` | Sí | Valor devuelto por la vista o por un manejador de error. |
| `from_error_handler` | `bool` | No | Activa el modo tolerante a fallos. |

**Retorno**

| Tipo | Descripción |
|---|---|
| `Response` | Respuesta final lista para el servidor WSGI. |

**Raises**

| Excepción | Condición |
|---|---|
| `TypeError` | Propagada por `make_response()` ante un valor de retorno inválido. |
| `Exception` | Fallo en `process_response()`/señales con `from_error_handler=False`. |

---

### 3.23 `make_default_options_response()`

```python
def make_default_options_response(self, ctx: AppContext) -> Response
```

Construye la respuesta automática a `OPTIONS`, poblando el encabezado `Allow` con `ctx.url_adapter.allowed_methods()`.

**Parámetros**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `ctx` | `AppContext` | Sí | Contexto activo; requiere `url_adapter` no nulo. |

**Retorno**

| Tipo | Descripción |
|---|---|
| `Response` | Respuesta vacía con el encabezado `Allow` establecido. |

**Raises**

| Excepción | Condición |
|---|---|
| `AttributeError` | `ctx.url_adapter` es `None`. |

---

### 3.24 `ensure_sync()`

```python
def ensure_sync(self, func: t.Callable[..., t.Any]) -> t.Callable[..., t.Any]
```

Garantiza que el callable sea síncrono para workers WSGI: las funciones `async def` se envuelven con `async_to_sync()`; las demás se devuelven tal cual.

**Parámetros**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `func` | `t.Callable[..., t.Any]` | Sí | Vista, manejador o callback a normalizar. |

**Retorno**

| Tipo | Descripción |
|---|---|
| `t.Callable[..., t.Any]` | Callable síncrono equivalente. |

**Raises**

| Excepción | Condición |
|---|---|
| `RuntimeError` | Propagada por `async_to_sync()` si falta el extra `async`. |

---

### 3.25 `async_to_sync()`

```python
def async_to_sync(
    self, func: t.Callable[..., t.Coroutine[t.Any, t.Any, t.Any]]
) -> t.Callable[..., t.Any]
```

Convierte una función corrutina en una función síncrona mediante `asgiref.sync.async_to_sync`.

**Parámetros**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `func` | `t.Callable[..., t.Coroutine[t.Any, t.Any, t.Any]]` | Sí | Función corrutina a adaptar. |

**Retorno**

| Tipo | Descripción |
|---|---|
| `t.Callable[..., t.Any]` | Wrapper síncrono que ejecuta y espera la corrutina. |

**Raises**

| Excepción | Condición |
|---|---|
| `RuntimeError` | `asgiref` no está instalado (`pip install flask[async]`). |

---

### 3.26 `url_for()`

```python
def url_for(
    self,
    /,
    endpoint: str,
    *,
    _anchor: str | None = None,
    _method: str | None = None,
    _scheme: str | None = None,
    _external: bool | None = None,
    **values: t.Any,
) -> str
```

Genera la URL de un endpoint. Dentro de una petición activa usa `ctx.url_adapter` y resuelve endpoints relativos con prefijo `"."` respecto al blueprint actual; fuera de petición, construye un adaptador y fuerza URLs externas por defecto. Aplica `inject_url_defaults()`, delega en `url_adapter.build()` y, ante `BuildError`, deriva a `handle_url_build_error()`. El ancla se codifica con `urllib.parse.quote`.

**Parámetros**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `endpoint` | `str` | Sí | Nombre del endpoint (posicional-only). Un `"."` inicial lo hace relativo al blueprint actual. |
| `_anchor` | `str | None` | No | Fragmento `#anchor` añadido al final. |
| `_method` | `str | None` | No | Método HTTP para seleccionar la regla. |
| `_scheme` | `str | None` | No | Esquema a usar en URLs externas. |
| `_external` | `bool | None` | No | Fuerza URL externa (`True`) o interna (`False`). |
| `**values` | `t.Any` | No | Valores de las partes variables; las claves desconocidas se añaden como *query string*. |

**Retorno**

| Tipo | Descripción |
|---|---|
| `str` | URL construida (o el valor de `handle_url_build_error()`). |

**Raises**

| Excepción | Condición |
|---|---|
| `RuntimeError` | Fuera de petición activa y sin `SERVER_NAME` configurado. |
| `ValueError` | Se indicó `_scheme` con `_external=False`. |
| `BuildError` | Relanzada por `handle_url_build_error()` cuando no se puede construir la URL. |

---

### 3.27 `make_response()`

```python
def make_response(self, rv: ft.ResponseReturnValue) -> Response
```

Normaliza el valor de retorno de una vista a una instancia de `response_class`. Desempaqueta tuplas de 2 o 3 elementos `(body, status, headers)`; convierte `str`/`bytes`/iteradores en respuestas (estas últimas en *streaming*), `dict`/`list` en JSON, instancias de otras clases `Response` o callables WSGI mediante `force_type()`. Finalmente aplica `status` y extiende `headers`.

**Parámetros**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `rv` | `ft.ResponseReturnValue` | Sí | Valor de retorno de la vista: `str`, `bytes`, `dict`, `list`, generador/iterador, tupla, `Response` o callable WSGI. |

**Retorno**

| Tipo | Descripción |
|---|---|
| `Response` | Instancia de `response_class` con estado y encabezados aplicados. |

**Raises**

| Excepción | Condición |
|---|---|
| `TypeError` | Tupla con longitud distinta de 2 o 3. |
| `TypeError` | `rv is None` (la vista no devolvió nada). |
| `TypeError` | `force_type()` falla, o el tipo devuelto no es soportado. |

---

### 3.28 `preprocess_request()`

```python
def preprocess_request(self, ctx: AppContext) -> ft.ResponseReturnValue | None
```

Ejecuta los `url_value_preprocessors` y posteriormente los `before_request_funcs` de la app y de los blueprints (en orden inverso de anidamiento). Si algún `before_request` devuelve un valor distinto de `None`, se detiene el procesamiento y ese valor se usa como respuesta.

**Parámetros**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `ctx` | `AppContext` | Sí | Contexto de la petición. |

**Retorno**

| Tipo | Descripción |
|---|---|
| `ft.ResponseReturnValue | None` | Valor de cortocircuito, o `None` para continuar al despacho. |

**Raises**

| Excepción | Condición |
|---|---|
| `Exception` | Propagada desde cualquier preprocesador o `before_request`. |

---

### 3.29 `process_response()`

```python
def process_response(self, ctx: AppContext, response: Response) -> Response
```

Aplica los callbacks `ctx._after_request_functions` y los `after_request_funcs` de blueprints y app (en orden inverso de registro). Si la sesión no es nula, la persiste mediante `session_interface.save_session()`.

**Parámetros**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `ctx` | `AppContext` | Sí | Contexto activo. |
| `response` | `Response` | Sí | Respuesta a modificar. |

**Retorno**

| Tipo | Descripción |
|---|---|
| `Response` | Respuesta resultante (misma instancia u otra de `response_class`). |

**Raises**

| Excepción | Condición |
|---|---|
| `Exception` | Propagada desde cualquier `after_request` o al guardar la sesión. |

---

### 3.30 `do_teardown_request()`

```python
def do_teardown_request(
    self, ctx: AppContext, exc: BaseException | None = None
) -> None
```

Ejecuta todos los `teardown_request_funcs` (blueprints primero, luego app, en orden inverso de registro) y envía la señal `request_tearing_down`. Utiliza `_CollectErrors` como gestor de contexto para ejecutar **todos** los callbacks aunque alguno falle, agregando los errores al final con `raise_any()`.

**Parámetros**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `ctx` | `AppContext` | Sí | Contexto de la petición que se está cerrando. |
| `exc` | `BaseException | None` | No | Excepción no manejada durante el despacho; se pasa a cada callback. |

**Retorno**

| Tipo | Descripción |
|---|---|
| `None` | Efecto lateral. |

**Raises**

| Excepción | Condición |
|---|---|
| Error agregado (`raise_any("Errors during request teardown")`) | Uno o más callbacks o receptores de señal fallaron. |

---

### 3.31 `do_teardown_appcontext()`

```python
def do_teardown_appcontext(
    self, ctx: AppContext, exc: BaseException | None = None
) -> None
```

Ejecuta los `teardown_appcontext_funcs` en orden inverso y envía `appcontext_tearing_down`, con la misma estrategia de recolección de errores que `do_teardown_request()`.

**Parámetros**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `ctx` | `AppContext` | Sí | Contexto de aplicación que se está cerrando. |
| `exc` | `BaseException | None` | No | Excepción no manejada mientras el contexto estuvo activo. |

**Retorno**

| Tipo | Descripción |
|---|---|
| `None` | Efecto lateral. |

**Raises**

| Excepción | Condición |
|---|---|
| Error agregado (`raise_any("Errors during app teardown")`) | Uno o más callbacks o receptores de señal fallaron. |

---

### 3.32 `app_context()`

```python
def app_context(self) -> AppContext
```

Crea un `AppContext` no asociado a petición. Al hacer *push*, `current_app` y `g` quedan disponibles.

**Parámetros:** ninguno.

**Retorno**

| Tipo | Descripción |
|---|---|
| `AppContext` | Contexto utilizable como bloque `with`. |

**Raises**

| Excepción | Condición |
|---|---|
| — | No lanza excepciones. |

---

### 3.33 `request_context()`

```python
def request_context(self, environ: WSGIEnvironment) -> AppContext
```

Crea un `AppContext` con información de petición a partir de un entorno WSGI, vía `AppContext.from_environ()`. Uso interno; para pruebas se recomienda `test_request_context()`.

**Parámetros**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `environ` | `WSGIEnvironment` | Sí | Diccionario de entorno WSGI válido. |

**Retorno**

| Tipo | Descripción |
|---|---|
| `AppContext` | Contexto con petición asociada. |

**Raises**

| Excepción | Condición |
|---|---|
| `KeyError` / `ValueError` | Entorno WSGI incompleto o inválido. |

---

### 3.34 `test_request_context()`

```python
def test_request_context(self, *args: t.Any, **kwargs: t.Any) -> AppContext
```

Construye un entorno WSGI con `flask.testing.EnvironBuilder` (cerrándolo en `finally`) y devuelve el `AppContext` correspondiente. Acepta los mismos argumentos que `werkzeug.test.EnvironBuilder`, con valores por defecto derivados de la configuración de la app.

**Parámetros**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `path` | `str` | No | Ruta URL solicitada. |
| `base_url` | `str` | No | URL base; si se omite, se deriva de `PREFERRED_URL_SCHEME`, `subdomain`, `SERVER_NAME` y `APPLICATION_ROOT`. |
| `subdomain` | `str` | No | Subdominio a prefijar a `SERVER_NAME`. |
| `url_scheme` | `str` | No | Esquema alternativo a `PREFERRED_URL_SCHEME`. |
| `data` | `str | bytes | dict` | No | Cuerpo de la petición o datos de formulario. |
| `json` | `t.Any` | No | Objeto serializado como JSON; fija `content_type` a `application/json`. |
| `*args` | `t.Any` | No | Argumentos posicionales adicionales para `EnvironBuilder`. |
| `**kwargs` | `t.Any` | No | Argumentos por palabra clave adicionales para `EnvironBuilder`. |

**Retorno**

| Tipo | Descripción |
|---|---|
| `AppContext` | Contexto de petición simulada, usable como bloque `with`. |

**Raises**

| Excepción | Condición |
|---|---|
| `TypeError` / `ValueError` | Argumentos inválidos para `EnvironBuilder`. |

---

### 3.35 `wsgi_app()`

```python
def wsgi_app(
    self, environ: WSGIEnvironment, start_response: StartResponse
) -> cabc.Iterable[bytes]
```

Implementación real de la aplicación WSGI, separada de `__call__()` para permitir envolver middlewares sin perder la referencia a la app (`app.wsgi_app = MyMiddleware(app.wsgi_app)`).

Flujo: crea y empuja el contexto de petición, ejecuta `full_dispatch_request()`, deriva cualquier `Exception` a `handle_exception()` y captura `BaseException` para registrar el error y relanzarlo. En `finally` preserva el contexto para el depurador (`werkzeug.debug.preserve_context`), aplica `should_ignore_error` si está definido y ejecuta `ctx.pop(error)`.

**Parámetros**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `environ` | `WSGIEnvironment` | Sí | Entorno WSGI de la petición. |
| `start_response` | `StartResponse` | Sí | Callable WSGI que inicia la respuesta. |

**Retorno**

| Tipo | Descripción |
|---|---|
| `cabc.Iterable[bytes]` | Iterable del cuerpo de la respuesta. |

**Raises**

| Excepción | Condición |
|---|---|
| `BaseException` | Excepciones que no derivan de `Exception` (p. ej. `KeyboardInterrupt`, `SystemExit`) se relanzan tras registrarse. |
| `Exception` | Relanzada por `handle_exception()` si `PROPAGATE_EXCEPTIONS` está activo. |

---

### 3.36 `__call__()`

```python
def __call__(
    self, environ: WSGIEnvironment, start_response: StartResponse
) -> cabc.Iterable[bytes]
```

Hace de la instancia `Flask` un callable WSGI; delega íntegramente en `wsgi_app()`.

**Parámetros**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `environ` | `WSGIEnvironment` | Sí | Entorno WSGI. |
| `start_response` | `StartResponse` | Sí | Callable de inicio de respuesta. |

**Retorno**

| Tipo | Descripción |
|---|---|
| `cabc.Iterable[bytes]` | Resultado de `wsgi_app(environ, start_response)`. |

**Raises**

| Excepción | Condición |
|---|---|
| *(ver `wsgi_app()`)* | Propaga las excepciones de `wsgi_app()`. |

---

## 4. Ejemplo de uso

```python
from __future__ import annotations

import typing as t

from werkzeug.exceptions import HTTPException, NotFound

from flask import Flask
from flask.ctx import AppContext


class AuditedFlask(Flask):
    """Subclase que ilustra la firma moderna con 'ctx: AppContext'."""

    def dispatch_request(self, ctx: AppContext) -> t.Any:
        # Acceso al contexto explícito en lugar de los proxies globales.
        self.logger.info("Despachando %s %s", ctx.request.method, ctx.request.path)
        return super().dispatch_request(ctx)

    def handle_http_exception(
        self, ctx: AppContext, e: HTTPException
    ) -> t.Any:
        if e.code == 404:
            # make_response() normaliza la tupla (body, status, headers).
            return self.make_response(({"error": "not found"}, 404))
        return super().handle_http_exception(ctx, e)

    def log_exception(self, ctx: AppContext, exc_info: t.Any) -> None:
        self.logger.error("Fallo en %s", ctx.request.full_path, exc_info=exc_info)


# --- Construcción de la aplicación ---------------------------------------
app = AuditedFlask(__name__, static_folder="static", template_folder="templates")
app.config.update(SERVER_NAME="example.com", PREFERRED_URL_SCHEME="https")


@app.route("/")
def index() -> str:
    # url_for() se resuelve usando el adaptador del contexto activo.
    return f'<a href="{app.url_for("profile", user_id=7, _anchor="bio")}">perfil</a>'


@app.route("/users/<int:user_id>")
def profile(user_id: int) -> dict[str, int]:
    if user_id <= 0:
        raise NotFound()
    return {"user_id": user_id}  # dict -> respuesta JSON vía make_response()


@app.teardown_appcontext
def close_resources(exc: BaseException | None) -> None:
    # Invocado por do_teardown_appcontext(); todos los callbacks se ejecutan.
    app.logger.debug("Teardown de contexto de app (exc=%r)", exc)


# --- Uso de contextos y utilidades --------------------------------------
with app.app_context():
    # URL externa fuera de petición: requiere SERVER_NAME configurado.
    print(app.url_for("index", _external=True))
    print(app.make_shell_context().keys())

with app.test_request_context("/users/7", method="GET"):
    print(app.url_for(".profile", user_id=7))  # endpoint relativo

# --- Cliente de pruebas --------------------------------------------------
app.testing = True
with app.test_client() as client:
    assert client.get("/users/7").get_json() == {"user_id": 7}
    assert client.get("/users/0").status_code == 404  # manejador personalizado

# --- Lectura de recursos -------------------------------------------------
try:
    with app.open_resource("schema.sql", mode="r", encoding="utf-8") as fh:
        schema = fh.read()
except FileNotFoundError:
    schema = ""

# --- Middleware sobre wsgi_app ------------------------------------------
original_wsgi_app = app.wsgi_app


def timing_middleware(environ, start_response):
    return original_wsgi_app(environ, start_response)


app.wsgi_app = timing_middleware

if __name__ == "__main__":
    # run() es solo para desarrollo; se ignora si se ejecuta vía 'flask run'.
    app.run(host="127.0.0.1", port=5000, debug=True, use_reloader=False)
```