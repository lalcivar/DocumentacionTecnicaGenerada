# Documentación Técnica: `flask/app.py`

## 1. Descripción general

Este módulo contiene la implementación de la clase **`Flask`**, el objeto central del framework Flask. Dicha clase implementa una **aplicación WSGI** completa y actúa como registro central para:

- funciones de vista (*view functions*) y reglas de URL,
- configuración de plantillas (Jinja2),
- manejadores de errores,
- hooks del ciclo de vida de la petición (`before_request`, `after_request`, `teardown_*`),
- comandos CLI (Click),
- sesiones e interfaz de sesión.

El archivo corresponde a la parte **"con I/O" (síncrona / WSGI)** de Flask: hereda de `App` (definida en `flask.sansio.app`), que contiene toda la lógica agnóstica del protocolo (registro de rutas, blueprints, configuración), y añade aquí todo lo que requiere entorno WSGI real: despacho de peticiones, contextos, servidor de desarrollo, cliente de pruebas, etc.

### Responsabilidades principales

| Área | Métodos relevantes |
|---|---|
| Punto de entrada WSGI | `wsgi_app`, `__call__` |
| Ciclo de vida de la petición | `full_dispatch_request`, `dispatch_request`, `preprocess_request`, `process_response`, `finalize_request` |
| Manejo de errores | `handle_http_exception`, `handle_user_exception`, `handle_exception`, `log_exception`, `raise_routing_exception` |
| Contextos | `app_context`, `request_context`, `test_request_context`, `do_teardown_request`, `do_teardown_appcontext` |
| Generación de respuestas | `make_response`, `make_default_options_response` |
| URLs | `url_for`, `create_url_adapter` |
| Plantillas | `create_jinja_environment`, `update_template_context` |
| Recursos estáticos y ficheros | `send_static_file`, `get_send_file_max_age`, `open_resource`, `open_instance_resource` |
| Desarrollo y pruebas | `run`, `test_client`, `test_cli_runner`, `make_shell_context` |
| Soporte async | `ensure_sync`, `async_to_sync` |

---

## 2. Dependencias

- **Werkzeug**: enrutamiento (`Rule`, `MapAdapter`, `BuildError`, `RequestRedirect`), excepciones HTTP (`HTTPException`, `InternalServerError`, `BadRequestKeyError`), estructuras de datos (`Headers`, `ImmutableDict`), servidor de desarrollo (`run_simple`, `is_running_from_reloader`) y utilidades WSGI (`get_host`).
- **Click**: grupo de comandos CLI de la aplicación (`cli.AppGroup`).
- **asgiref** (opcional, extra `async`): conversión de vistas `async def` a llamadas síncronas.
- **Módulos internos de Flask**: `ctx.AppContext`, `globals` (`request`, `session`, `g`, `app_ctx`, `_cv_app`), `sessions`, `signals`, `templating`, `wrappers`, `helpers`, `sansio.app.App`.

---

## 3. Componentes de nivel de módulo

### 3.1 Variables de tipo (TypeVars)

```python
T_shell_context_processor, T_teardown, T_template_filter,
T_template_global, T_template_test, F
```

Se usan para preservar las firmas en decoradores y utilidades genéricas (`F` está acotado a cualquier callable).

### 3.2 `_make_timedelta(value) -> timedelta | None`

Función auxiliar privada. Normaliza un valor de configuración a `timedelta`:
- `None` → `None`
- `timedelta` → se devuelve sin cambios
- `int` → se interpreta como segundos.

### 3.3 `remove_ctx(f) -> F`

Decorador de compatibilidad. Envuelve un método que **todavía usa la firma antigua** (sin el parámetro `ctx`): si el primer argumento posicional es un `AppContext`, lo descarta antes de invocar la función original.

### 3.4 `add_ctx(f) -> F`

Decorador complementario. Envuelve un método base que **sí espera `ctx`**: si quien lo invoca (p. ej. un `super()` de código legado) no proporciona el `AppContext`, lo inyecta obteniéndolo del proxy `app_ctx`.

> Ambos decoradores existen únicamente para dar soporte transitorio a subclases antiguas de `Flask`, junto con el mecanismo de aviso de deprecación en `__init_subclass__`.

---

## 4. Clase `Flask(App)`

Implementa una aplicación WSGI. Recibe el *import name* del módulo o paquete, que Flask usa para resolver rutas de recursos (`root_path`, `static_folder`, `template_folder`, `instance_path`).

```python
from flask import Flask
app = Flask(__name__)
```

### 4.1 Atributos de clase

| Atributo | Tipo | Descripción |
|---|---|---|
| `default_config` | `ImmutableDict` | Valores por defecto de configuración: `DEBUG`, `TESTING`, `SECRET_KEY`, `PERMANENT_SESSION_LIFETIME`, `SERVER_NAME`, `APPLICATION_ROOT`, familia `SESSION_COOKIE_*`, `MAX_CONTENT_LENGTH`, `MAX_FORM_MEMORY_SIZE`, `MAX_FORM_PARTS`, `TRUSTED_HOSTS`, `PREFERRED_URL_SCHEME`, `MAX_COOKIE_SIZE`, etc. |
| `request_class` | `type[Request]` | Clase usada para los objetos de petición. |
| `response_class` | `type[Response]` | Clase usada para los objetos de respuesta. |
| `session_interface` | `SessionInterface` | Por defecto `SecureCookieSessionInterface()`. |

### 4.2 `__init_subclass__(cls, **kwargs)`

Hook de compatibilidad hacia atrás. Al definirse una subclase de `Flask`, inspecciona un conjunto de métodos cuya firma cambió para aceptar `ctx: AppContext` como primer parámetro:

`handle_http_exception`, `handle_user_exception`, `handle_exception`, `log_exception`, `dispatch_request`, `full_dispatch_request`, `finalize_request`, `make_default_options_response`, `preprocess_request`, `process_response`, `do_teardown_request`, `do_teardown_appcontext`.

Para cada método sobrescrito, comprueba mediante `inspect.signature` si el segundo parámetro (tras `self`) se llama `ctx` o está anotado como `AppContext`. Si no lo está:

1. Emite un `DeprecationWarning` (soporte eliminado en Flask 4.0).
2. Envuelve el método de la subclase con `remove_ctx`.
3. Envuelve el método base de `Flask` con `add_ctx`.

### 4.3 `__init__(...)`

Parámetros principales: `import_name`, `static_url_path`, `static_folder`, `static_host`, `host_matching`, `subdomain_matching`, `template_folder`, `instance_path`, `instance_relative_config`, `root_path`.

Acciones:
1. Delega en `App.__init__` toda la inicialización agnóstica.
2. Crea `self.cli = cli.AppGroup()` y le asigna el nombre de la app.
3. Si hay `static_folder` configurado, valida la coherencia `static_host`/`host_matching` y registra automáticamente la regla `"<static_url_path>/<path:filename>"` con endpoint `"static"`. La vista se registra a través de un **`weakref`** a la app para evitar ciclos de referencia (issue #3761).

---

## 5. Métodos agrupados por responsabilidad

### 5.1 Archivos estáticos y recursos

| Método | Descripción |
|---|---|
| `get_send_file_max_age(filename)` | Devuelve el valor `max_age` de caché para `send_file`, tomado de `SEND_FILE_MAX_AGE_DEFAULT` (acepta `timedelta` o segundos). `None` ⇒ peticiones condicionales. |
| `send_static_file(filename)` | Vista que sirve ficheros desde `static_folder` mediante `send_from_directory`. Lanza `RuntimeError` si no hay carpeta estática. |
| `open_resource(resource, mode="rb", encoding=None)` | Abre un fichero relativo a `root_path` **solo en lectura** (`"r"`, `"rt"`, `"rb"`); otros modos provocan `ValueError`. |
| `open_instance_resource(resource, mode="rb", encoding="utf-8")` | Abre un fichero relativo a `instance_path`; permite escritura. |

### 5.2 Plantillas

- **`create_jinja_environment() -> Environment`**  
  Construye el entorno Jinja a partir de `jinja_options`, estableciendo `autoescape` (vía `select_jinja_autoescape`) y `auto_reload` (según `TEMPLATES_AUTO_RELOAD` o el modo debug). Registra los globales `url_for`, `get_flashed_messages`, `config`, `request`, `session`, `g` y la política `json.dumps_function`.

- **`update_template_context(ctx, context)`**  
  Inyecta en el contexto de plantilla el resultado de los *context processors* registrados en la app y en los blueprints de la petición actual (si hay petición). Los valores originales pasados a `render_template` se reaplican al final para que tengan precedencia.

### 5.3 Enrutamiento y URLs

- **`create_url_adapter(request) -> MapAdapter | None`**  
  Crea el `MapAdapter` de Werkzeug. Con petición activa: aplica `TRUSTED_HOSTS`, resuelve el host real, y ajusta `server_name`/`subdomain` según `host_matching` y `subdomain_matching`. Sin petición: requiere `SERVER_NAME` y usa `APPLICATION_ROOT` y `PREFERRED_URL_SCHEME`; devuelve `None` si no es posible.

- **`raise_routing_exception(request) -> NoReturn`** *(interno)*  
  Relanza la excepción de enrutamiento. En modo debug, convierte un redirect que descartaría el cuerpo de la petición en un `FormDataRoutingRedirect` informativo. No intercepta 307/308 ni métodos `GET`/`HEAD`/`OPTIONS`.

- **`url_for(endpoint, *, _anchor, _method, _scheme, _external, **values) -> str`**  
  Construye una URL para un endpoint. Resuelve endpoints relativos (`".name"`) respecto al blueprint actual; dentro de una petición genera URLs internas por defecto y externas fuera de ella. Aplica `inject_url_defaults`, delega en `url_adapter.build` y, ante `BuildError`, invoca `handle_url_build_error`. Valida que `_scheme` solo se use con `_external=True` y añade el ancla con *quoting*.

### 5.4 Manejo de errores

| Método | Descripción |
|---|---|
| `handle_http_exception(ctx, e)` | Busca un manejador para la `HTTPException` (por código y por MRO, considerando blueprints). Devuelve la excepción tal cual si no tiene `code` o si es una `RoutingException`. |
| `handle_user_exception(ctx, e)` | Punto de entrada para excepciones durante el despacho. Activa `show_exception` en `BadRequestKeyError` bajo debug/`TRAP_BAD_REQUEST_ERRORS`; redirige `HTTPException` a `handle_http_exception`; si no hay manejador, relanza. |
| `handle_exception(ctx, e)` | Trata excepciones no manejadas. Emite la señal `got_request_exception`; si `PROPAGATE_EXCEPTIONS` (o testing/debug) relanza; en otro caso registra el error, crea un `InternalServerError(original_exception=e)`, aplica el manejador de 500 si existe y finaliza la petición con `from_error_handler=True`. |
| `log_exception(ctx, exc_info)` | Registra la excepción como `error` en `self.logger`, incluyendo ruta y método. |

### 5.5 Ciclo de vida de la petición

```
__call__ → wsgi_app
            ├── request_context(environ) → ctx.push()
            ├── full_dispatch_request(ctx)
            │     ├── signal request_started
            │     ├── preprocess_request(ctx)
            │     ├── dispatch_request(ctx)
            │     ├── [handle_user_exception]
            │     └── finalize_request(ctx, rv)
            │            ├── make_response(rv)
            │            ├── process_response(ctx, response)
            │            └── signal request_finished
            ├── [handle_exception]
            └── ctx.pop(error) → do_teardown_request / do_teardown_appcontext
```

- **`dispatch_request(ctx)`**: relanza excepciones de routing, responde automáticamente a `OPTIONS` si la regla lo permite, y en caso contrario invoca la función de vista del endpoint con `view_args` (pasando por `ensure_sync`).
- **`full_dispatch_request(ctx)`**: añade pre/post-procesado, señales y captura de errores. Marca `_got_first_request` y avisa si `should_ignore_error` está definido (deprecado, eliminado en Flask 3.3).
- **`finalize_request(ctx, rv, from_error_handler=False)`**: convierte el valor de retorno en respuesta y ejecuta el post-procesado. En modo "seguro" (`from_error_handler=True`) los fallos se registran y se ignoran.
- **`preprocess_request(ctx)`**: ejecuta `url_value_preprocessors` y después `before_request_funcs` (app y blueprints, del más específico al menos). Si un handler devuelve algo distinto de `None`, ese valor cortocircuita el despacho.
- **`process_response(ctx, response)`**: aplica las funciones `after_request` del contexto y de app/blueprints (en orden inverso al de registro) y guarda la sesión mediante `session_interface.save_session` si no es una sesión nula.
- **`make_default_options_response(ctx)`**: construye la respuesta `OPTIONS` por defecto con la cabecera `Allow` poblada desde `url_adapter.allowed_methods()`.

### 5.6 `make_response(rv) -> Response`

Normaliza el valor devuelto por una vista a una instancia de `response_class`. Tipos admitidos:

- `str` / `bytes` / `bytearray` → cuerpo de la respuesta.
- `dict` / `list` → respuesta JSON.
- generador o iterador → respuesta en *streaming*.
- `tuple` → `(body, status, headers)`, `(body, status)` o `(body, headers)`.
- instancia de `response_class` → sin cambios.
- otra `Response` de Werkzeug o callable WSGI → coercionada con `force_type`.

Lanza `TypeError` si el valor es `None`, si la tupla tiene un tamaño inválido o si el tipo no es soportado. Finalmente aplica `status` y extiende `headers`.

### 5.7 Contextos de aplicación y petición

| Método | Descripción |
|---|---|
| `app_context()` | Crea un `AppContext`; al hacer *push* habilita `current_app` y `g`. |
| `request_context(environ)` | Crea un `AppContext` con información de petición desde un entorno WSGI (`AppContext.from_environ`). Uso interno. |
| `test_request_context(*args, **kwargs)` | Construye un entorno WSGI con `testing.EnvironBuilder` (defaults tomados de la config) y devuelve el contexto correspondiente. Ideal para pruebas. |
| `do_teardown_request(ctx, exc=None)` | Ejecuta todas las funciones `teardown_request` (app y blueprints, orden inverso) y envía `request_tearing_down`. Usa `_CollectErrors` para ejecutar **todos** los callbacks y agregar los errores. |
| `do_teardown_appcontext(ctx, exc=None)` | Ejecuta `teardown_appcontext_funcs` y envía `appcontext_tearing_down`, con la misma recolección de errores. |

### 5.8 Interfaz WSGI

- **`wsgi_app(environ, start_response)`**  
  Implementación real de la aplicación WSGI. Crea y hace *push* del contexto, despacha la petición, convierte las excepciones no manejadas vía `handle_exception`, devuelve la respuesta y, en el bloque `finally`, preserva el contexto para el depurador de Werkzeug si procede, aplica `should_ignore_error` y hace `ctx.pop(error)`.  
  Se mantiene separada de `__call__` para permitir envolver middlewares sin perder la referencia a la app:
  ```python
  app.wsgi_app = MyMiddleware(app.wsgi_app)
  ```

- **`__call__(environ, start_response)`**  
  Delega en `wsgi_app`; es el punto que invoca el servidor WSGI.

### 5.9 Soporte asíncrono

- **`ensure_sync(func)`**: devuelve `func` si es síncrona; si es `async def`, la envuelve con `async_to_sync`. Se usa en todos los puntos donde se invocan callbacks de usuario (vistas, hooks, manejadores de error, *context processors*).
- **`async_to_sync(func)`**: usa `asgiref.sync.async_to_sync`; lanza `RuntimeError` indicando instalar `Flask[async]` si `asgiref` no está disponible.

### 5.10 Desarrollo, CLI y pruebas

- **`run(host=None, port=None, debug=None, load_dotenv=True, **options)`**  
  Arranca el servidor de desarrollo de Werkzeug (`run_simple`). **No apto para producción.** Comportamiento:
  - Si se está ejecutando desde `flask run` (`FLASK_RUN_FROM_CLI`), ignora la llamada y avisa.
  - Carga `.env`/`.flaskenv` (si `load_dotenv` y python-dotenv disponible) y aplica `FLASK_DEBUG`; el parámetro `debug` tiene prioridad máxima.
  - Deriva host/puerto de `SERVER_NAME` si no se indican (por defecto `127.0.0.1:5000`).
  - Defaults: `use_reloader` y `use_debugger` según debug, `threaded=True`.
  - Al terminar, resetea `_got_first_request`.

- **`test_client(use_cookies=True, **kwargs) -> FlaskClient`**  
  Devuelve un cliente de pruebas (`test_client_class` o `flask.testing.FlaskClient`). Soporta uso como gestor de contexto para acceder a los *context locals*.

- **`test_cli_runner(**kwargs) -> FlaskCliRunner`**  
  Devuelve un runner para probar comandos CLI (`test_cli_runner_class` o `FlaskCliRunner`).

- **`make_shell_context() -> dict`**  
  Construye el espacio de nombres del shell interactivo: `{"app": self, "g": g}` más lo aportado por los `shell_context_processors`.

---

## 6. Señales emitidas

| Señal | Momento |
|---|---|
| `request_started` | Inicio de `full_dispatch_request` |
| `request_finished` | Tras `process_response` en `finalize_request` |
| `got_request_exception` | Al entrar en `handle_exception` |
| `request_tearing_down` | Final de `do_teardown_request` |
| `appcontext_tearing_down` | Final de `do_teardown_appcontext` |

---

## 7. Notas de compatibilidad y deprecaciones

- Los métodos del ciclo de vida ahora reciben **`ctx: AppContext`** como primer parámetro. Las subclases con la firma antigua siguen funcionando mediante `__init_subclass__` + `remove_ctx`/`add_ctx`, pero emiten `DeprecationWarning` (soporte retirado en **Flask 4.0**).
- `should_ignore_error` está deprecado y se eliminará en **Flask 3.3**; la recomendación es gestionar los errores en los handlers de *teardown*.
- Desde 3.2, `do_teardown_request` y `do_teardown_appcontext` ejecutan **todos** los callbacks y agregan los errores (`_CollectErrors.raise_any`) en lugar de detenerse en el primero.
- Desde 3.1, `SERVER_NAME` ya no restringe las peticiones a ese dominio, y `open_resource`/`open_instance_resource` aceptan `encoding`.