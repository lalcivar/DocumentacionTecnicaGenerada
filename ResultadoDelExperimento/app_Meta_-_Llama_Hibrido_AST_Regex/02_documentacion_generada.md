# Documentación Técnica - Módulo `flask.app` (Clase `Flask`)

## 1. Resumen Ejecutivo

El módulo implementa la clase central `Flask`, que extiende `sansio.app.App` para proveer una aplicación WSGI completa. Actúa como registro central para rutas, vistas, manejo de errores, contextos de aplicación/request, gestión de sesiones, integración con Jinja2 y Werkzeug, y utilidades para desarrollo y testing. El archivo incluye además funciones auxiliares de compatibilidad (`_make_timedelta`, `remove_ctx`, `add_ctx`) para la transición de firmas con `AppContext` y el ciclo de vida WSGI.

---

## 2. Especificación de Funciones y Métodos

### 2.1 Funciones a Nivel de Módulo

#### ` _make_timedelta()`

Convierte un valor entero (segundos) o `timedelta` en `timedelta`.

```python
def _make_timedelta(value: timedelta | int | None) -> timedelta | None:
    """Convierte un valor a timedelta.

    Args:
        value: Valor a convertir. Puede ser timedelta, entero en segundos o None.

    Returns:
        El mismo timedelta si ya lo es, un nuevo timedelta si es int, o None.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
|---|---|---|---|
| `value` | `timedelta \| int \| None` | Sí | Valor a normalizar. `int` se interpreta como segundos. |

| Retorno | Tipo | Descripción |
|---|---|---|
| `timedelta \| None` | `timedelta \| None` | `timedelta` normalizado o `None` si la entrada fue `None`. |

**Raises:** No lanza excepciones.

---

#### `remove_ctx()`

Decorador de compatibilidad para subclases que aún usan la firma antigua sin `ctx: AppContext`. Elimina el primer argumento si es `AppContext` antes de delegar a la función original.

```python
def remove_ctx(f: F) -> F:
    """Envuelve una función para eliminar el argumento ctx si está presente.

    Args:
        f: Función objetivo con firma antigua.

    Returns:
        Función envuelta que ignora el AppContext inicial si se pasa.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
|---|---|---|---|
| `f` | `Callable[..., Any]` | Sí | Función/método a envolver. |

| Retorno | Tipo | Descripción |
|---|---|---|
| `F` | `Callable` | Wrapper con `update_wrapper` aplicado. |

---

#### `remove_ctx.wrapper()`

Función anidada/local dentro de `remove_ctx`.

```python
def wrapper(self: Flask, *args: Any, **kwargs: Any) -> Any:
    """Wrapper interno que filtra AppContext.

    Args:
        self: Instancia de Flask.
        *args: Argumentos posicionales, posiblemente con AppContext al inicio.
        **kwargs: Argumentos nominales.

    Returns:
        Resultado de invocar f sin el AppContext.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
|---|---|---|---|
| `self` | `Flask` | Sí | Instancia receptora. |
| `*args` | `Any` | No | Argumentos posicionales. |
| `**kwargs` | `Any` | No | Argumentos nominales. |

| Retorno | Tipo | Descripción |
|---|---|---|
| `Any` | `Any` | Retorno de `f`. |

---

#### `add_ctx()`

Decorador inverso a `remove_ctx`. Inyecta el `AppContext` actual (`app_ctx._get_current_object()`) como primer argumento si no está presente, para permitir que el método base llame a `super()` sin `ctx`.

```python
def add_ctx(f: F) -> F:
    """Envuelve una función para inyectar AppContext automáticamente.

    Args:
        f: Función objetivo con firma nueva (requiere ctx).

    Returns:
        Función envuelta que antepone el AppContext actual.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
|---|---|---|---|
| `f` | `Callable[..., Any]` | Sí | Función/método con firma nueva. |

| Retorno | Tipo | Descripción |
|---|---|---|
| `F` | `Callable` | Wrapper con `update_wrapper` aplicado. |

---

#### `add_ctx.wrapper()`

Función anidada/local dentro de `add_ctx`.

```python
def wrapper(self: Flask, *args: Any, **kwargs: Any) -> Any:
    """Wrapper interno que inyecta AppContext.

    Args:
        self: Instancia de Flask.
        *args: Argumentos posicionales.
        **kwargs: Argumentos nominales.

    Returns:
        Resultado de invocar f con AppContext antepuesto.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
|---|---|---|---|
| `self` | `Flask` | Sí | Instancia receptora. |
| `*args` | `Any` | No | Argumentos posicionales. |
| `**kwargs` | `Any` | No | Argumentos nominales. |

| Retorno | Tipo | Descripción |
|---|---|---|
| `Any` | `Any` | Retorno de `f`. |

---

### 2.2 Métodos de la Clase `Flask`

#### `Flask.__init_subclass__()`

Hook de creación de subclases que detecta métodos sobreescritos con firma antigua (sin `ctx: AppContext`), emite `DeprecationWarning` y aplica `remove_ctx`/`add_ctx` para compatibilidad hasta Flask 4.0.

```python
def __init_subclass__(cls, **kwargs: Any) -> None:
    """Detecta firmas deprecadas y aplica wrappers de compatibilidad.

    Args:
        **kwargs: Argumentos de creación de subclase.

    Returns:
        None
    """
```

| Parámetro | Tipo | Requerido | Descripción |
|---|---|---|---|
| `cls` | `type` | Sí | Subclase de `Flask` en creación. |
| `**kwargs` | `Any` | No | Argumentos adicionales de `type.__new__`. |

| Retorno | Tipo | Descripción |
|---|---|---|
| `None` | `None` | No retorna valor. |

**Raises:**
| Excepción | Condición |
|---|---|
| `DeprecationWarning` | Cuando un método sobreescrito no declara `ctx: AppContext` como segundo parámetro. |

Métodos inspeccionados: `handle_http_exception`, `handle_user_exception`, `handle_exception`, `log_exception`, `dispatch_request`, `full_dispatch_request`, `finalize_request`, `make_default_options_response`, `preprocess_request`, `process_response`, `do_teardown_request`, `do_teardown_appcontext`.

---

#### `Flask.__init__()`

Inicializa la aplicación Flask, configura `cli.AppGroup` y registra la ruta estática si `has_static_folder` es verdadero.

```python
def __init__(self, import_name: str, static_url_path: str | None = None, static_folder: str | os.PathLike[str] | None = "static", static_host: str | None = None, host_matching: bool = False, subdomain_matching: bool = False, template_folder: str | os.PathLike[str] | None = "templates", instance_path: str | None = None, instance_relative_config: bool = False, root_path: str | None = None) -> None:
    """Inicializa la aplicación Flask.

    Args:
        import_name: Nombre del paquete/módulo de la aplicación.
        static_url_path: Ruta URL para archivos estáticos.
        static_folder: Carpeta de archivos estáticos.
        static_host: Host para la ruta estática (requerido si host_matching=True).
        host_matching: Habilita matching por host en url_map.
        subdomain_matching: Habilita matching por subdominio relativo a SERVER_NAME.
        template_folder: Carpeta de plantillas.
        instance_path: Ruta alternativa para la carpeta instance.
        instance_relative_config: Si True, config relativa a instance_path.
        root_path: Ruta raíz de archivos de la app.

    Returns:
        None
    """
```

| Parámetro | Tipo | Requerido | Descripción |
|---|---|---|---|
| `import_name` | `str` | Sí | Nombre de importación para resolución de recursos. |
| `static_url_path` | `str \| None` | No | Path URL estático. Default `None` (derivado de `static_folder`). |
| `static_folder` | `str \| os.PathLike[str] \| None` | No | Carpeta estática. Default `"static"`. |
| `static_host` | `str \| None` | No | Host estático. Default `None`. |
| `host_matching` | `bool` | No | Default `False`. |
| `subdomain_matching` | `bool` | No | Default `False`. |
| `template_folder` | `str \| os.PathLike[str] \| None` | No | Default `"templates"`. |
| `instance_path` | `str \| None` | No | Default `None` (auto-detectado). |
| `instance_relative_config` | `bool` | No | Default `False`. |
| `root_path` | `str \| None` | No | Default `None` (auto-detectado). |

**Raises:**
| Excepción | Condición |
|---|---|
| `AssertionError` | Si `bool(static_host) != host_matching` cuando hay `static_folder`. |

**Función anidada:** `Flask.__init__.<lambda>` (`view_func=lambda **kw: self_ref().send_static_file(**kw)`)
| Parámetro | Tipo | Requerido | Descripción |
|---|---|---|---|
| `**kw` | `Any` | No | `filename` y otros kwargs de la ruta. |
| Retorno | `Response` | Resultado de `send_static_file`. |

---

#### `Flask.get_send_file_max_age()`

Determina el valor `max_age` para `send_file`.

```python
def get_send_file_max_age(self, filename: str | None) -> int | None:
    """Obtiene max_age para cache de archivos.

    Args:
        filename: Nombre del archivo solicitado.

    Returns:
        Segundos de max_age, o None para usar peticiones condicionales.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
|---|---|---|---|
| `filename` | `str \| None` | Sí | Nombre del archivo. |

| Retorno | Tipo | Descripción |
|---|---|---|
| `int \| None` | `int \| None` | Segundos o `None`. |

---

#### `Flask.send_static_file()`

View function para servir archivos de `static_folder`.

```python
def send_static_file(self, filename: str) -> Response:
    """Sirve un archivo estático.

    Args:
        filename: Ruta relativa dentro de static_folder.

    Returns:
        Response con el archivo.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
|---|---|---|---|
| `filename` | `str` | Sí | Nombre/ruta del archivo. |

| Retorno | Tipo | Descripción |
|---|---|---|
| `Response` | `Response` | Respuesta con el archivo. |

**Raises:**
| Excepción | Condición |
|---|---|
| `RuntimeError` | Si `has_static_folder` es `False`. |
| `NotFound` | Si el archivo no existe (vía `send_from_directory`). |

---

#### `Flask.open_resource()`

Abre un recurso relativo a `root_path` solo en modo lectura.

```python
def open_resource(self, resource: str, mode: str = "rb", encoding: str | None = None) -> IO[AnyStr]:
    """Abre un recurso del paquete.

    Args:
        resource: Ruta relativa a root_path.
        mode: Modo de apertura. Solo "r", "rt", "rb".
        encoding: Codificación para modo texto.

    Returns:
        Handle de archivo abierto.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
|---|---|---|---|
| `resource` | `str` | Sí | Ruta relativa. |
| `mode` | `str` | No | Default `"rb"`. |
| `encoding` | `str \| None` | No | Default `None`. |

**Raises:**
| Excepción | Condición |
|---|---|
| `ValueError` | Si `mode` no es `r`, `rt` o `rb`. |
| `FileNotFoundError` | Si el recurso no existe. |

---

#### `Flask.open_instance_resource()`

Abre un recurso relativo a `instance_path`, permite lectura y escritura.

```python
def open_instance_resource(self, resource: str, mode: str = "rb", encoding: str | None = "utf-8") -> IO[AnyStr]:
    """Abre un recurso de la carpeta instance.

    Args:
        resource: Ruta relativa a instance_path.
        mode: Modo de apertura.
        encoding: Codificación para modo texto. Default "utf-8".

    Returns:
        Handle de archivo abierto.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
|---|---|---|---|
| `resource` | `str` | Sí | Ruta relativa. |
| `mode` | `str` | No | Default `"rb"`. |
| `encoding` | `str \| None` | No | Default `"utf-8"`. |

**Raises:**
| Excepción | Condición |
|---|---|
| `FileNotFoundError` | Si el recurso no existe y se abre en lectura. |

---

#### `Flask.create_jinja_environment()`

Crea el `Environment` de Jinja2 configurado con `jinja_options`, `select_jinja_autoescape` y globales de Flask.

```python
def create_jinja_environment(self) -> Environment:
    """Crea el entorno Jinja2.

    Returns:
        Environment configurado con url_for, get_flashed_messages, config, request, session, g.
    """
```

| Retorno | Tipo | Descripción |
|---|---|---|
| `Environment` | `Environment` | Entorno Jinja listo para renderizar. |

---

#### `Flask.create_url_adapter()`

Crea un `MapAdapter` para construir/resolver URLs.

```python
def create_url_adapter(self, request: Request | None) -> MapAdapter | None:
    """Crea adaptador de URLs.

    Args:
        request: Request actual o None si se llama fuera de request.

    Returns:
        MapAdapter vinculado al environ o a SERVER_NAME, o None si no hay SERVER_NAME fuera de request.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
|---|---|---|---|
| `request` | `Request \| None` | Sí | Request actual o `None`. |

| Retorno | Tipo | Descripción |
|---|---|---|
| `MapAdapter \| None` | `MapAdapter \| None` | Adaptador o `None`. |

---

#### `Flask.raise_routing_exception()`

Intercepta `RequestRedirect` en modo debug para evitar pérdida de datos de formulario.

```python
def raise_routing_exception(self, request: Request) -> NoReturn:
    """Intercepta excepciones de routing.

    Args:
        request: Request con routing_exception.

    Raises:
        RequestRedirect | FormDataRoutingRedirect | RoutingException
    """
```

| Parámetro | Tipo | Requerido | Descripción |
|---|---|---|---|
| `request` | `Request` | Sí | Request con `routing_exception`. |

**Raises:**
| Excepción | Condición |
|---|---|
| `RequestRedirect` | Si no está en debug o método es GET/HEAD/OPTIONS o código 307/308. |
| `FormDataRoutingRedirect` | Si está en debug y la redirección descartaría body. |

---

#### `Flask.update_template_context()`

Inyecta variables comunes y context processors al contexto de plantillas.

```python
def update_template_context(self, ctx: AppContext, context: dict[str, Any]) -> None:
    """Actualiza el contexto de plantilla in-place.

    Args:
        ctx: AppContext actual.
        context: Diccionario de contexto a actualizar.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
|---|---|---|---|
| `ctx` | `AppContext` | Sí | Contexto de aplicación. |
| `context` | `dict[str, Any]` | Sí | Diccionario mutable. |

| Retorno | Tipo | Descripción |
|---|---|---|
| `None` | `None` | Modifica `context` in-place. |

---

#### `Flask.make_shell_context()`

Ejecuta todos los `shell_context_processors` registrados.

```python
def make_shell_context(self) -> dict[str, Any]:
    """Retorna contexto para shell interactivo.

    Returns:
        Diccionario con app, g y valores de processors.
    """
```

| Retorno | Tipo | Descripción |
|---|---|---|
| `dict[str, Any]` | `dict` | Contexto combinado. |

---

#### `Flask.run()`

Ejecuta servidor de desarrollo Werkzeug.

```python
def run(self, host: str | None = None, port: int | None = None, debug: bool | None = None, load_dotenv: bool = True, **options: Any) -> None:
    """Inicia servidor de desarrollo.

    Args:
        host: Hostname. Default 127.0.0.1 o SERVER_NAME.
        port: Puerto. Default 5000 o SERVER_NAME.
        debug: Habilita debug. Sobrescribe config.
        load_dotenv: Carga .env/.flaskenv.
        **options: Opciones para werkzeug.serving.run_simple.

    Returns:
        None
    """
```

| Parámetro | Tipo | Requerido | Descripción |
|---|---|---|---|
| `host` | `str \| None` | No | Default `None`. |
| `port` | `int \| None` | No | Default `None`. |
| `debug` | `bool \| None` | No | Default `None`. |
| `load_dotenv` | `bool` | No | Default `True`. |
| `**options` | `Any` | No | `use_reloader`, `use_debugger`, `threaded`, etc. |

---

#### `Flask.test_client()`

Crea cliente de pruebas WSGI.

```python
def test_client(self, use_cookies: bool = True, **kwargs: Any) -> FlaskClient:
    """Crea test client.

    Args:
        use_cookies: Si True mantiene cookies entre requests.
        **kwargs: Argumentos para test_client_class.

    Returns:
        Instancia de FlaskClient.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
|---|---|---|---|
| `use_cookies` | `bool` | No | Default `True`. |
| `**kwargs` | `Any` | No | Args adicionales. |

| Retorno | Tipo | Descripción |
|---|---|---|
| `FlaskClient` | `FlaskClient` | Cliente de pruebas. |

---

#### `Flask.test_cli_runner()`

Crea runner para comandos CLI.

```python
def test_cli_runner(self, **kwargs: Any) -> FlaskCliRunner:
    """Crea CLI runner.

    Args:
        **kwargs: Argumentos para test_cli_runner_class.

    Returns:
        Instancia de FlaskCliRunner.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
|---|---|---|---|
| `**kwargs` | `Any` | No | Args adicionales. |

| Retorno | Tipo | Descripción |
|---|---|---|
| `FlaskCliRunner` | `FlaskCliRunner` | Runner CLI. |

---

#### `Flask.handle_http_exception()`

Maneja `HTTPException` invocando error handlers registrados.

```python
def handle_http_exception(self, ctx: AppContext, e: HTTPException) -> HTTPException | ResponseReturnValue:
    """Maneja excepción HTTP.

    Args:
        ctx: AppContext actual.
        e: Excepción HTTP.

    Returns:
        Respuesta del handler o la misma excepción si no hay handler.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
|---|---|---|---|
| `ctx` | `AppContext` | Sí | Contexto. |
| `e` | `HTTPException` | Sí | Excepción. |

| Retorno | Tipo | Descripción |
|---|---|---|
| `HTTPException \| ResponseReturnValue` | `HTTPException \| ResponseReturnValue` | Handler result o excepción original. |

---

#### `Flask.handle_user_exception()`

Punto central para excepciones de usuario; delega a `handle_http_exception` si es `HTTPException` y no debe atraparse.

```python
def handle_user_exception(self, ctx: AppContext, e: Exception) -> HTTPException | ResponseReturnValue:
    """Maneja excepción de usuario.

    Args:
        ctx: AppContext actual.
        e: Excepción capturada.

    Returns:
        Valor de respuesta del handler.

    Raises:
        Exception: Si no hay handler registrado, re-lanza e.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
|---|---|---|---|
| `ctx` | `AppContext` | Sí | Contexto. |
| `e` | `Exception` | Sí | Excepción. |

**Raises:**
| Excepción | Condición |
|---|---|
| `Exception` | Si no existe handler para `e`. |

---

#### `Flask.handle_exception()`

Maneja excepciones no capturadas, genera `InternalServerError` 500, envía señal `got_request_exception` y respeta `PROPAGATE_EXCEPTIONS`.

```python
def handle_exception(self, ctx: AppContext, e: Exception) -> Response:
    """Maneja excepción no controlada.

    Args:
        ctx: AppContext actual.
        e: Excepción original.

    Returns:
        Response finalizada (500 o handler de 500).
    """
```

| Parámetro | Tipo | Requerido | Descripción |
|---|---|---|---|
| `ctx` | `AppContext` | Sí | Contexto. |
| `e` | `Exception` | Sí | Excepción. |

**Raises:**
| Excepción | Condición |
|---|---|
| `Exception` | Si `PROPAGATE_EXCEPTIONS` es `True` (testing/debug). |

---

#### `Flask.log_exception()`

Registra excepción en `self.logger`.

```python
def log_exception(self, ctx: AppContext, exc_info: tuple[type, BaseException, TracebackType] | tuple[None, None, None]) -> None:
    """Loguea excepción.

    Args:
        ctx: AppContext actual.
        exc_info: Tupla exc_info de sys.exc_info().
    """
```

| Parámetro | Tipo | Requerido | Descripción |
|---|---|---|---|
| `ctx` | `AppContext` | Sí | Contexto. |
| `exc_info` | `tuple` | Sí | Info de excepción. |

| Retorno | Tipo | Descripción |
|---|---|---|
| `None` | `None` | Solo loguea. |

---

#### `Flask.dispatch_request()`

Resuelve la URL y despacha a la view function.

```python
def dispatch_request(self, ctx: AppContext) -> ResponseReturnValue:
    """Despacha request a la vista.

    Args:
        ctx: AppContext con request.

    Returns:
        Valor de retorno de la vista o respuesta OPTIONS automática.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
|---|---|---|---|
| `ctx` | `AppContext` | Sí | Contexto. |

**Raises:**
| Excepción | Condición |
|---|---|
| `RequestRedirect` | Si `req.routing_exception` existe. |
| `KeyError` | Si `rule.endpoint` no está en `view_functions`. |

---

#### `Flask.full_dispatch_request()`

Orquesta `preprocess_request`, `dispatch_request`, manejo de excepciones y `finalize_request`. Emite `request_started`.

```python
def full_dispatch_request(self, ctx: AppContext) -> Response:
    """Despacho completo con pre/post-procesamiento.

    Args:
        ctx: AppContext actual.

    Returns:
        Response finalizada.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
|---|---|---|---|
| `ctx` | `AppContext` | Sí | Contexto. |

| Retorno | Tipo | Descripción |
|---|---|---|
| `Response` | `Response` | Respuesta final. |

---

#### `Flask.finalize_request()`

Convierte `rv` en `Response` vía `make_response`, aplica `process_response` y emite `request_finished`.

```python
def finalize_request(self, ctx: AppContext, rv: ResponseReturnValue | HTTPException, from_error_handler: bool = False) -> Response:
    """Finaliza request convirtiendo rv en Response.

    Args:
        ctx: AppContext actual.
        rv: Valor de retorno de vista o excepción HTTP.
        from_error_handler: Si True, suprime errores en post-procesamiento y solo loguea.

    Returns:
        Response procesada.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
|---|---|---|---|
| `ctx` | `AppContext` | Sí | Contexto. |
| `rv` | `ResponseReturnValue \| HTTPException` | Sí | Valor a convertir. |
| `from_error_handler` | `bool` | No | Default `False`. |

| Retorno | Tipo | Descripción |
|---|---|---|
| `Response` | `Response` | Respuesta final. |

**Raises:**
| Excepción | Condición |
|---|---|
| `Exception` | Si `process_response` falla y `from_error_handler` es `False`. |

---

#### `Flask.make_default_options_response()`

Genera respuesta automática para método `OPTIONS`.

```python
def make_default_options_response(self, ctx: AppContext) -> Response:
    """Crea respuesta OPTIONS por defecto.

    Args:
        ctx: AppContext con url_adapter.

    Returns:
        Response con header Allow poblado.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
|---|---|---|---|
| `ctx` | `AppContext` | Sí | Contexto. |

| Retorno | Tipo | Descripción |
|---|---|---|
| `Response` | `Response` | Respuesta OPTIONS. |

---

#### `Flask.ensure_sync()`

Garantiza que una función sea síncrona para workers WSGI.

```python
def ensure_sync(self, func: Callable[..., Any]) -> Callable[..., Any]:
    """Asegura ejecución síncrona.

    Args:
        func: Función posiblemente async.

    Returns:
        Función síncrona (original o envuelta vía async_to_sync).
    """
```

| Parámetro | Tipo | Requerido | Descripción |
|---|---|---|---|
| `func` | `Callable[..., Any]` | Sí | Función a normalizar. |

| Retorno | Tipo | Descripción |
|---|---|---|
| `Callable[..., Any]` | `Callable` | Función síncrona. |

---

#### `Flask.async_to_sync()`

Convierte corrutina en función síncrona usando `asgiref.sync.async_to_sync`.

```python
def async_to_sync(self, func: Callable[..., Coroutine[Any, Any, Any]]) -> Callable[..., Any]:
    """Convierte async def a sync.

    Args:
        func: Función corrutina.

    Returns:
        Función síncrona equivalente.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
|---|---|---|---|
| `func` | `Callable[..., Coroutine]` | Sí | Corrutina. |

**Raises:**
| Excepción | Condición |
|---|---|
| `RuntimeError` | Si `asgiref` no está instalado y se requiere soporte async. |

---

#### `Flask.url_for()`

Genera URL para un endpoint.

```python
def url_for(self, endpoint: str, *, _anchor: str | None = None, _method: str | None = None, _scheme: str | None = None, _external: bool | None = None, **values: Any) -> str:
    """Genera URL para endpoint.

    Args:
        endpoint: Nombre del endpoint (prefijo blueprint con ".").
        _anchor: Ancla a añadir como #anchor.
        _method: Método para seleccionar regla.
        _scheme: Esquema si es externa.
        _external: Si True fuerza URL externa.
        **values: Valores para variables de URL y query string.

    Returns:
        URL generada como string.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
|---|---|---|---|
| `endpoint` | `str` | Sí | Nombre del endpoint. |
| `_anchor` | `str \| None` | No | Default `None`. |
| `_method` | `str \| None` | No | Default `None`. |
| `_scheme` | `str \| None` | No | Default `None`. |
| `_external` | `bool \| None` | No | Default `None` (auto según contexto). |
| `**values` | `Any` | No | Variables de URL. |

**Raises:**
| Excepción | Condición |
|---|---|
| `RuntimeError` | Si no hay request/app context y falta `SERVER_NAME`. |
| `ValueError` | Si `_scheme` se especifica con `_external=False`. |
| `BuildError` | Si `handle_url_build_error` no retorna string y la construcción falla. |

---

#### `Flask.make_response()`

Convierte valor de retorno de vista en instancia de `response_class`.

```python
def make_response(self, rv: ResponseReturnValue) -> Response:
    """Convierte rv en Response.

    Args:
        rv: Valor de retorno (str, bytes, dict, list, tuple, Response, WSGI callable, iterator).

    Returns:
        Instancia de response_class.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
|---|---|---|---|
| `rv` | `ResponseReturnValue` | Sí | Valor de vista. |

**Raises:**
| Excepción | Condición |
|---|---|
| `TypeError` | Si `rv` es `None`, tupla de tamaño inválido o tipo no soportado. |

---

#### `Flask.preprocess_request()`

Ejecuta `url_value_preprocessors` y `before_request_funcs`.

```python
def preprocess_request(self, ctx: AppContext) -> ResponseReturnValue | None:
    """Pre-procesa request.

    Args:
        ctx: AppContext actual.

    Returns:
        Valor de respuesta si un before_request lo retorna, o None para continuar.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
|---|---|---|---|
| `ctx` | `AppContext` | Sí | Contexto. |

| Retorno | Tipo | Descripción |
|---|---|---|
| `ResponseReturnValue \| None` | `Any \| None` | Respuesta temprana o `None`. |

---

#### `Flask.process_response()`

Aplica `after_request_funcs` y guarda sesión.

```python
def process_response(self, ctx: AppContext, response: Response) -> Response:
    """Post-procesa response.

    Args:
        ctx: AppContext actual.
        response: Response a procesar.

    Returns:
        Response modificada.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
|---|---|---|---|
| `ctx` | `AppContext` | Sí | Contexto. |
| `response` | `Response` | Sí | Respuesta. |

| Retorno | Tipo | Descripción |
|---|---|---|
| `Response` | `Response` | Respuesta procesada. |

---

#### `Flask.do_teardown_request()`

Ejecuta `teardown_request_funcs` y señal `request_tearing_down`.

```python
def do_teardown_request(self, ctx: AppContext, exc: BaseException | None = None) -> None:
    """Ejecuta teardown de request.

    Args:
        ctx: AppContext actual.
        exc: Excepción no manejada durante dispatch, o None.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
|---|---|---|---|
| `ctx` | `AppContext` | Sí | Contexto. |
| `exc` | `BaseException \| None` | No | Default `None`. |

**Raises:**
| Excepción | Condición |
|---|---|
| `Exception` | Agregada vía `_CollectErrors` si algún teardown falla. |

---

#### `Flask.do_teardown_appcontext()`

Ejecuta `teardown_appcontext_funcs` y señal `appcontext_tearing_down`.

```python
def do_teardown_appcontext(self, ctx: AppContext, exc: BaseException | None = None) -> None:
    """Ejecuta teardown de appcontext.

    Args:
        ctx: AppContext actual.
        exc: Excepción no manejada durante el contexto, o None.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
|---|---|---|---|
| `ctx` | `AppContext` | Sí | Contexto. |
| `exc` | `BaseException \| None` | No | Default `None`. |

**Raises:**
| Excepción | Condición |
|---|---|
| `Exception` | Agregada si algún teardown falla. |

---

#### `Flask.app_context()`

Crea `AppContext` sin request.

```python
def app_context(self) -> AppContext:
    """Crea contexto de aplicación.

    Returns:
        AppContext asociado a self.
    """
```

| Retorno | Tipo | Descripción |
|---|---|---|
| `AppContext` | `AppContext` | Contexto de app. |

---

#### `Flask.request_context()`

Crea `AppContext` con información de request a partir de un `environ` WSGI.

```python
def request_context(self, environ: WSGIEnvironment) -> AppContext:
    """Crea contexto de request.

    Args:
        environ: Diccionario WSGI environ.

    Returns:
        AppContext con request poblado.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
|---|---|---|---|
| `environ` | `WSGIEnvironment` | Sí | Environ WSGI. |

| Retorno | Tipo | Descripción |
|---|---|---|
| `AppContext` | `AppContext` | Contexto con request. |

---

#### `Flask.test_request_context()`

Helper para tests que construye `environ` vía `EnvironBuilder`.

```python
def test_request_context(self, *args: Any, **kwargs: Any) -> AppContext:
    """Crea contexto de request para testing.

    Args:
        *args: Args para EnvironBuilder.
        **kwargs: Kwargs para EnvironBuilder (path, base_url, subdomain, url_scheme, data, json, etc.).

    Returns:
        AppContext de prueba.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
|---|---|---|---|
| `*args` | `Any` | No | Args posicionales para `EnvironBuilder`. |
| `**kwargs` | `Any` | No | Kwargs para `EnvironBuilder`. |

| Retorno | Tipo | Descripción |
|---|---|---|
| `AppContext` | `AppContext` | Contexto de prueba. |

---

#### `Flask.wsgi_app()`

Aplicación WSGI real. Gestiona push/pop de contexto, dispatch y manejo de errores.

```python
def wsgi_app(self, environ: WSGIEnvironment, start_response: StartResponse) -> Iterable[bytes]:
    """Aplicación WSGI.

    Args:
        environ: Environ WSGI.
        start_response: Callable start_response(status, headers, exc_info).

    Returns:
        Iterable de bytes de la respuesta.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
|---|---|---|---|
| `environ` | `WSGIEnvironment` | Sí | Environ WSGI. |
| `start_response` | `StartResponse` | Sí | Callable WSGI. |

| Retorno | Tipo | Descripción |
|---|---|---|
| `Iterable[bytes]` | `Iterable[bytes]` | Cuerpo de la respuesta. |

---

#### `Flask.__call__()`

Entry-point WSGI que delega a `wsgi_app` (permite envolver `wsgi_app` con middleware sin perder referencia a `app`).

```python
def __call__(self, environ: WSGIEnvironment, start_response: StartResponse) -> Iterable[bytes]:
    """Invocación WSGI del objeto app.

    Args:
        environ: Environ WSGI.
        start_response: Callable start_response.

    Returns:
        Iterable de bytes.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
|---|---|---|---|
| `environ` | `WSGIEnvironment` | Sí | Environ WSGI. |
| `start_response` | `StartResponse` | Sí | Callable WSGI. |

| Retorno | Tipo | Descripción |
|---|---|---|
| `Iterable[bytes]` | `Iterable[bytes]` | Resultado de `wsgi_app`. |

---

## 3. Ejemplo de Uso

```python
from flask import Flask, request, jsonify, url_for

# 1. Creación de la aplicación
app = Flask(__name__, static_folder="static", template_folder="templates")
app.config["SERVER_NAME"] = "example.com"
app.config["SECRET_KEY"] = "dev-secret"

# 2. Registro de ruta y uso de url_for / make_response
@app.route("/hello/<name>", methods=["GET"])
def hello(name: str):
    # Flask convierte dict -> JSON response vía make_response
    return {"message": f"Hola, {name}!"}

@app.route("/login", methods=["POST"])
def login():
    data = request.get_json()
    if not data or "user" not in data:
        # Retorno tupla (body, status)
        return {"error": "user requerido"}, 400
    return jsonify(user=data["user"]), 200

# 3. Contextos manuales y generación de URLs fuera de request
with app.app_context():
    # url_for fuera de request requiere SERVER_NAME
    print(app.url_for("hello", name="Mundo", _external=True))
    # open_resource para leer archivos del paquete
    # with app.open_resource("schema.sql") as f:
    #     print(f.read())

# 4. Testing con test_client y test_request_context
with app.test_client() as client:
    rv = client.get("/hello/Mundo")
    assert rv.status_code == 200
    assert rv.get_json()["message"] == "Hola, Mundo!"

with app.test_request_context("/hello/Test?x=1"):
    assert request.args["x"] == "1"
    assert request.endpoint == "hello"

# 5. Servidor de desarrollo (solo en guard __main__)
if __name__ == "__main__":
    # Respeta FLASK_RUN_FROM_CLI y carga .env si existe
    app.run(host="127.0.0.1", port=5000, debug=True)

# 6. WSGI middleware wrapping (preserva app)
# from werkzeug.middleware.proxy_fix import ProxyFix
# app.wsgi_app = ProxyFix(app.wsgi_app, x_for=1)
```

