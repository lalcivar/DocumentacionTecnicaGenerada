# Documentación Técnica - Módulo `flask.cli`

## 1. Resumen Ejecutivo

El módulo `flask.cli` implementa la interfaz de línea de comandos (CLI) oficial de Flask basada en `Click`. Gestiona el descubrimiento y carga de instancias `Flask` (`find_best_app`, `locate_app`, `prepare_import`), la administración del contexto de aplicación (`ScriptInfo`, `with_appcontext`), la carga de variables de entorno (`load_dotenv`) y el grupo de comandos principal `FlaskGroup` que expone los comandos `run`, `shell` y `routes` para desarrollo y diagnóstico.

---

## 2. Especificación de Funciones y Métodos

### 2.1 `NoAppException`

Excepción personalizada que hereda de `click.UsageError`. Se lanza cuando no se puede encontrar, importar o instanciar una aplicación Flask.

```python
class NoAppException(click.UsageError):
    """Raised if an application cannot be found or loaded."""
```

### 2.2 `find_best_app()`

```python
def find_best_app(module: ModuleType) -> Flask:
    """Intenta encontrar la mejor instancia de aplicación Flask en un módulo dado.

    Busca en orden: atributos `app`/`application`, única instancia de `Flask` en el
    módulo, y factories `create_app`/`make_app` sin argumentos.

    Args:
        module: Módulo de Python donde buscar la aplicación.

    Returns:
        La instancia de `Flask` encontrada.

    Raises:
        NoAppException: Si se detectan múltiples aplicaciones, si la factory requiere
            argumentos o si no se encuentra ninguna aplicación.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `module` | `ModuleType` | Sí | Módulo inspeccionado para localizar la app |

| Retorna | Tipo | Descripción |
| :--- | :--- | :--- |
| `app` | `Flask` | Instancia de la aplicación Flask encontrada |

| Excepción | Condición |
| :--- | :--- |
| `NoAppException` | Múltiples instancias `Flask` detectadas |
| `NoAppException` | Factory `create_app`/`make_app` requiere argumentos |
| `NoAppException` | No se encontró ninguna app o factory válida |

### 2.3 `_called_with_wrong_args()`

```python
def _called_with_wrong_args(f: t.Callable[..., Flask]) -> bool:
    """Verifica si un TypeError se debe a una llamada incorrecta a la factory.

    Inspecciona el traceback para determinar si el error ocurrió antes de entrar
    a la función `f` (argumentos incorrectos) o dentro de ella (error interno).

    Args:
        f: Función factory que se intentó llamar.

    Returns:
        True si la llamada falló por argumentos incorrectos, False si el error
        se originó dentro de la función.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `f` | `t.Callable[..., Flask]` | Sí | Función factory a inspeccionar |

| Retorna | Tipo | Descripción |
| :--- | :--- | :--- |
| `bool` | `bool` | `True` si no se alcanzó el código de `f` |

### 2.4 `find_app_by_string()`

```python
def find_app_by_string(module: ModuleType, app_name: str) -> Flask:
    """Obtiene una app Flask a partir de un string de atributo o llamada a factory.

    Parsea `app_name` como expresión AST. Soporta `nombre` y `nombre(args, kwargs)`
    donde los argumentos deben ser literales evaluables con `ast.literal_eval`.

    Args:
        module: Módulo donde buscar el atributo.
        app_name: Nombre del atributo o llamada a factory (ej: "create_app('prod')").

    Returns:
        Instancia de `Flask` obtenida.

    Raises:
        NoAppException: Si el parsing falla, el atributo no existe, los argumentos
            no son literales, la factory falla o el resultado no es una app Flask.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `module` | `ModuleType` | Sí | Módulo contenedor |
| `app_name` | `str` | Sí | Expresión de atributo o factory |

| Retorna | Tipo | Descripción |
| :--- | :--- | :--- |
| `app` | `Flask` | Aplicación Flask resuelta |

| Excepción | Condición |
| :--- | :--- |
| `NoAppException` | Error de sintaxis en `app_name` |
| `NoAppException` | Referencia no es `Name` ni `Call` simple |
| `NoAppException` | Argumentos no son valores literales |
| `NoAppException` | Atributo no encontrado en el módulo |
| `NoAppException` | Factory llamada con argumentos incorrectos |
| `NoAppException` | El objeto obtenido no es instancia `Flask` |

### 2.5 `prepare_import()`

```python
def prepare_import(path: str) -> str:
    """Calcula el nombre del módulo a partir de una ruta de archivo y lo añade a sys.path.

    Normaliza la ruta, elimina extensión .py, maneja paquetes __init__.py y asciende
    hasta encontrar el directorio que no es paquete para insertarlo en sys.path[0].

    Args:
        path: Ruta de archivo o módulo (ej: "myapp/app.py" o "myapp/__init__.py").

    Returns:
        Nombre de módulo punteado resultante (ej: "myapp.app").
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `path` | `str` | Sí | Ruta del archivo a importar |

| Retorna | Tipo | Descripción |
| :--- | :--- | :--- |
| `module_name` | `str` | Nombre de importación punteado |

### 2.6 `locate_app()`

```python
@t.overload
def locate_app(module_name: str, app_name: str | None, raise_if_not_found: t.Literal[True] = True) -> Flask: ...

@t.overload
def locate_app(module_name: str, app_name: str | None, raise_if_not_found: t.Literal[False] = ...) -> Flask | None: ...

def locate_app(module_name: str, app_name: str | None, raise_if_not_found: bool = True) -> Flask | None:
    """Importa un módulo y localiza la aplicación Flask dentro de él.

    Args:
        module_name: Nombre punteado del módulo a importar.
        app_name: Nombre de atributo/factory dentro del módulo. Si es None, usa find_best_app.
        raise_if_not_found: Si es True lanza NoAppException cuando el import falla; si es False retorna None.

    Returns:
        Instancia Flask si se encuentra, None si raise_if_not_found es False y no se encuentra.

    Raises:
        NoAppException: Si el import falla y raise_if_not_found es True, o si el ImportError
            ocurrió dentro del módulo importado (re-lanzado con traceback).
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `module_name` | `str` | Sí | Nombre del módulo a importar con `__import__` |
| `app_name` | `str \| None` | Sí | Atributo/factory o `None` para auto-detección |
| `raise_if_not_found` | `bool` | No (default `True`) | Controla si se lanza excepción o se retorna `None` |

| Retorna | Tipo | Descripción |
| :--- | :--- | :--- |
| `Flask \| None` | `Flask \| None` | App encontrada o `None` |

| Excepción | Condición |
| :--- | :--- |
| `NoAppException` | `ImportError` dentro del módulo importado |
| `NoAppException` | No se pudo importar `module_name` |

### 2.7 `get_version()`

```python
def get_version(ctx: click.Context, param: click.Parameter, value: t.Any) -> None:
    """Callback eager para la opción --version. Muestra versiones y sale.

    Args:
        ctx: Contexto de Click.
        param: Parámetro que disparó el callback.
        value: Valor de la opción --version.

    Returns:
        None. Hace ctx.exit() si value es True.

    Raises:
        SystemExit: Vía ctx.exit() tras imprimir versiones.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `ctx` | `click.Context` | Sí | Contexto Click |
| `param` | `click.Parameter` | Sí | Definición del parámetro |
| `value` | `t.Any` | Sí | Valor booleano de `--version` |

### 2.8 `ScriptInfo.__init__()`

```python
class ScriptInfo:
    def __init__(self, app_import_path: str | None = None, create_app: t.Callable[..., Flask] | None = None, set_debug_flag: bool = True, load_dotenv_defaults: bool = True) -> None:
        """Inicializa el helper para la carga de la aplicación Flask.

        Args:
            app_import_path: Ruta de importación tipo "module:name(args)".
            create_app: Callable que recibe ScriptInfo y retorna Flask.
            set_debug_flag: Si True, sincroniza app.debug con get_debug_flag().
            load_dotenv_defaults: Si True, indica que se deben cargar .env/.flaskenv por defecto.

        Attributes:
            app_import_path: Ruta de importación configurada.
            create_app: Factory opcional.
            data: Diccionario para datos arbitrarios asociados.
            set_debug_flag: Flag para actualizar debug.
            load_dotenv_defaults: Resultado de get_load_dotenv().
            _loaded_app: Cache interno de la app cargada.
        """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `app_import_path` | `str \| None` | No | Ruta de importación de la app |
| `create_app` | `t.Callable[..., Flask] \| None` | No | Factory personalizada |
| `set_debug_flag` | `bool` | No (default `True`) | Si se debe setear `app.debug` |
| `load_dotenv_defaults` | `bool` | No (default `True`) | Si se cargan dotenv por defecto |

### 2.9 `ScriptInfo.load_app()`

```python
def load_app(self) -> Flask:
    """Carga y cachea la aplicación Flask.

    Orden de resolución: 1) create_app si está definido, 2) app_import_path vía
    prepare_import + locate_app, 3) auto-descubrimiento de wsgi.py / app.py.

    Returns:
        Instancia Flask cargada y cacheada.

    Raises:
        NoAppException: Si no se localiza ninguna aplicación.
    """
```

| Retorna | Tipo | Descripción |
| :--- | :--- | :--- |
| `Flask` | `Flask` | Aplicación cargada |

| Excepción | Condición |
| :--- | :--- |
| `NoAppException` | No se encontró app en ninguna de las fuentes |

### 2.10 `with_appcontext()`

```python
def with_appcontext(f: F) -> F:
    """Decorador que garantiza la ejecución dentro del app_context.

    Args:
        f: Función callback a envolver.

    Returns:
        Función envuelta que asegura app_context activo antes de invocar f.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `f` | `F` (`t.Callable[..., t.Any]`) | Sí | Función a decorar |

| Retorna | Tipo | Descripción |
| :--- | :--- | :--- |
| `F` | `F` | Función decorada con `update_wrapper` |

#### 2.10.1 `with_appcontext.decorator()`

Función anidada/local generada dentro de `with_appcontext`.

```python
@click.pass_context
def decorator(ctx: click.Context, /, *args: t.Any, **kwargs: t.Any) -> t.Any:
    """Wrapper interno que inyecta app_context si no existe.

    Args:
        ctx: Contexto Click inyectado.
        *args: Argumentos posicionales del comando.
        **kwargs: Argumentos nombrados del comando.

    Returns:
        Resultado de ctx.invoke(f, *args, **kwargs).
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `ctx` | `click.Context` | Sí | Contexto Click |
| `*args` | `t.Any` | No | Args del comando original |
| `**kwargs` | `t.Any` | No | Kwargs del comando original |

### 2.11 `AppGroup.command()`

```python
def command(self, *args: t.Any, **kwargs: t.Any) -> t.Callable[[t.Callable[..., t.Any]], click.Command]:
    """Registra un comando envolviéndolo en with_appcontext por defecto.

    Args:
        *args: Argumentos para click.Group.command.
        **kwargs: Argumentos para click.Group.command. Soporta `with_appcontext` (bool, default True).

    Returns:
        Decorador que registra y retorna un click.Command.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `*args` | `t.Any` | No | Args para `click.Group.command` |
| `**kwargs` | `t.Any` | No | Kwargs para `click.Group.command`, `with_appcontext` controla el wrapping |

#### 2.11.1 `AppGroup.command.decorator()`

```python
def decorator(f: t.Callable[..., t.Any]) -> click.Command:
    """Decorador interno que aplica with_appcontext y registra el comando.

    Args:
        f: Función del comando.

    Returns:
        Instancia click.Command registrada.
    """
```

### 2.12 `AppGroup.group()`

```python
def group(self, *args: t.Any, **kwargs: t.Any) -> t.Callable[[t.Callable[..., t.Any]], click.Group]:
    """Crea un subgrupo cuyo cls por defecto es AppGroup.

    Args:
        *args: Argumentos para click.Group.group.
        **kwargs: Argumentos para click.Group.group.

    Returns:
        Decorador que registra y retorna un click.Group de tipo AppGroup.
    """
```

### 2.13 `_set_app()`

```python
def _set_app(ctx: click.Context, param: click.Option, value: str | None) -> str | None:
    """Callback eager para -A/--app. Almacena la ruta en ScriptInfo.

    Args:
        ctx: Contexto Click.
        param: Opción --app.
        value: Valor de importación "module:name".

    Returns:
        El mismo value recibido, o None si no se proveyó.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `ctx` | `click.Context` | Sí | Contexto |
| `param` | `click.Option` | Sí | Parámetro --app |
| `value` | `str \| None` | Sí | Ruta de importación |

### 2.14 `_set_debug()`

```python
def _set_debug(ctx: click.Context, param: click.Option, value: bool) -> bool | None:
    """Callback para --debug/--no-debug. Sincroniza FLASK_DEBUG en os.environ.

    Ignora valores por defecto (DEFAULT/DEFAULT_MAP) para permitir que env vars prevalezcan.

    Args:
        ctx: Contexto Click.
        param: Opción --debug.
        value: Booleano del flag.

    Returns:
        bool si el flag fue provisto explícitamente, None si fue default.
    """
```

### 2.15 `_env_file_callback()`

```python
def _env_file_callback(ctx: click.Context, param: click.Option, value: str | None) -> str | None:
    """Callback eager para -e/--env-file. Carga dotenv si está disponible.

    Args:
        ctx: Contexto Click (requiere ctx.obj.load_dotenv_defaults).
        param: Opción --env-file.
        value: Ruta al archivo env o None.

    Returns:
        El value recibido.

    Raises:
        click.BadParameter: Si se pasa value pero python-dotenv no está instalado.
    """
```

### 2.16 `FlaskGroup.__init__()`

```python
def __init__(self, add_default_commands: bool = True, create_app: t.Callable[..., Flask] | None = None, add_version_option: bool = True, load_dotenv: bool = True, set_debug_flag: bool = True, **extra: t.Any) -> None:
    """Inicializa el grupo CLI principal de Flask.

    Args:
        add_default_commands: Si True añade run, shell y routes.
        create_app: Factory opcional para ScriptInfo.
        add_version_option: Si True añade --version.
        load_dotenv: Si True habilita carga de .env/.flaskenv.
        set_debug_flag: Si True propaga flag debug a la app.
        **extra: Argumentos extra para click.Group (params, context_settings, etc.).

    Returns:
        None.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `add_default_commands` | `bool` | No (default `True`) | Añadir comandos por defecto |
| `create_app` | `t.Callable[..., Flask] \| None` | No | Factory para ScriptInfo |
| `add_version_option` | `bool` | No (default `True`) | Añadir `--version` |
| `load_dotenv` | `bool` | No (default `True`) | Cargar dotenv |
| `set_debug_flag` | `bool` | No (default `True`) | Sincronizar debug |
| `**extra` | `t.Any` | No | Args extra para `AppGroup` |

### 2.17 `FlaskGroup._load_plugin_commands()`

```python
def _load_plugin_commands(self) -> None:
    """Carga comandos de entry_points group='flask.commands' una sola vez.

    Returns:
        None.
    """
```

### 2.18 `FlaskGroup.get_command()`

```python
def get_command(self, ctx: click.Context, name: str) -> click.Command | None:
    """Resuelve un comando: primero built-ins/plugins, luego app.cli.

    Si la app no carga, imprime error en rojo y retorna None. Si hay app_context
    inactivo, lo empuja vía ctx.with_resource.

    Args:
        ctx: Contexto Click.
        name: Nombre del comando.

    Returns:
        Comando encontrado o None.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `ctx` | `click.Context` | Sí | Contexto |
| `name` | `str` | Sí | Nombre del comando |

### 2.19 `FlaskGroup.list_commands()`

```python
def list_commands(self, ctx: click.Context) -> list[str]:
    """Lista comandos combinando built-ins, plugins y app.cli.

    Args:
        ctx: Contexto Click.

    Returns:
        Lista ordenada de nombres de comandos.
    """
```

### 2.20 `FlaskGroup.make_context()`

```python
def make_context(self, info_name: str | None, args: list[str], parent: click.Context | None = None, **extra: t.Any) -> click.Context:
    """Crea el contexto Click e inyecta ScriptInfo y FLASK_RUN_FROM_CLI.

    Args:
        info_name: Nombre del programa.
        args: Argumentos CLI.
        parent: Contexto padre.
        **extra: Args extra, si no hay obj crea ScriptInfo.

    Returns:
        Contexto Click configurado.
    """
```

### 2.21 `FlaskGroup.parse_args()`

```python
def parse_args(self, ctx: click.Context, args: list[str]) -> list[str]:
    """Parsea args manejando no_args_is_help y carga temprana de --env-file/--app.

    Args:
        ctx: Contexto Click.
        args: Lista de argumentos.

    Returns:
        Lista de argumentos restantes tras el parseo.
    """
```

### 2.22 `_path_is_ancestor()`

```python
def _path_is_ancestor(path: str, other: str) -> bool:
    """Determina si path es ancestro de other.

    Args:
        path: Ruta candidata a ancestro.
        other: Ruta a verificar.

    Returns:
        True si path es ancestro de other.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `path` | `str` | Sí | Ruta base |
| `other` | `str` | Sí | Ruta hija |

### 2.23 `load_dotenv()`

```python
def load_dotenv(path: str | os.PathLike[str] | None = None, load_defaults: bool = True) -> bool:
    """Carga archivos dotenv sin sobrescribir variables ya existentes en os.environ.

    Precedencia: path explícito > .env > .flaskenv. No-op si python-dotenv no está instalado.

    Args:
        path: Ruta explícita a archivo env.
        load_defaults: Si True busca y carga .flaskenv y .env con find_dotenv.

    Returns:
        True si al menos una variable fue cargada, False en caso contrario.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `path` | `str \| os.PathLike[str] \| None` | No | Archivo env explícito |
| `load_defaults` | `bool` | No (default `True`) | Cargar defaults |

### 2.24 `show_server_banner()`

```python
def show_server_banner(debug: bool, app_import_path: str | None) -> None:
    """Muestra banner de inicio del servidor (ignora reloader).

    Args:
        debug: Estado del modo debug.
        app_import_path: Ruta de importación de la app a mostrar.

    Returns:
        None.
    """
```

### 2.25 `CertParamType.__init__()`

```python
def __init__(self) -> None:
    """Inicializa el tipo Click para --cert con validación de path existente."""
```

### 2.26 `CertParamType.convert()`

```python
def convert(self, value: t.Any, param: click.Parameter | None, ctx: click.Context | None) -> t.Any:
    """Convierte el valor de --cert a path, 'adhoc' o SSLContext.

    Args:
        value: Valor crudo de la opción.
        param: Parámetro Click.
        ctx: Contexto Click.

    Returns:
        Path resuelto, string "adhoc" o instancia ssl.SSLContext.

    Raises:
        click.BadParameter: Si Python sin SSL, si falta cryptography para adhoc,
            o si el valor no es path/adhoc/SSLContext importable.
    """
```

### 2.27 `_validate_key()`

```python
def _validate_key(ctx: click.Context, param: click.Parameter, value: t.Any) -> t.Any:
    """Valida coherencia entre --cert y --key. Convierte cert a tupla (cert, key) si aplica.

    Args:
        ctx: Contexto Click (usa ctx.params["cert"]).
        param: Parámetro --key.
        value: Ruta del key o None.

    Returns:
        El value recibido.

    Raises:
        click.BadParameter: Si --key se usa con adhoc/SSLContext, si --cert falta,
            o si --cert requiere --key y no se proveyó.
    """
```

### 2.28 `SeparatedPathType.convert()`

```python
def convert(self, value: t.Any, param: click.Parameter | None, ctx: click.Context | None) -> t.Any:
    """Convierte string separado por os.pathsep en lista de paths validados.

    Args:
        value: String con paths separados por ":" o ";" en Windows.
        param: Parámetro Click.
        ctx: Contexto Click.

    Returns:
        Lista de paths validados.
    """
```

### 2.29 `run_command()`

```python
@click.command("run")
@pass_script_info
def run_command(info: ScriptInfo, host: str, port: int, reload: bool, debugger: bool, with_threads: bool, cert: ssl.SSLContext | tuple[str, str | None] | t.Literal["adhoc"] | None, extra_files: list[str] | None, exclude_patterns: list[str] | None) -> None:
    """Ejecuta el servidor de desarrollo Werkzeug.

    Args:
        info: ScriptInfo inyectado vía pass_script_info.
        host: Interfaz de bind (default 127.0.0.1).
        port: Puerto de bind (default 5000).
        reload: Habilita reloader (default None -> sigue debug).
        debugger: Habilita debugger (default None -> sigue debug).
        with_threads: Habilita multithreading.
        cert: Certificado SSL (path, tupla, SSLContext o "adhoc").
        extra_files: Archivos extra que disparan reload.
        exclude_patterns: Patrones fnmatch a excluir del reload.

    Returns:
        None. Llama a werkzeug.run_simple de forma bloqueante.

    Raises:
        Exception: Re-lanza error de load_app si no está en reloader; en reloader
            imprime traceback y sirve app que siempre falla.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `info` | `ScriptInfo` | Sí | Info del script (inyectado) |
| `host` | `str` | No (default `127.0.0.1`) | Host |
| `port` | `int` | No (default `5000`) | Puerto |
| `reload` | `bool` | No (default `None`) | Reloader |
| `debugger` | `bool` | No (default `None`) | Debugger |
| `with_threads` | `bool` | No (default `True`) | Threads |
| `cert` | `ssl.SSLContext \| tuple[str, str \| None] \| Literal["adhoc"] \| None` | No | Certificado |
| `extra_files` | `list[str] \| None` | No | Archivos extra |
| `exclude_patterns` | `list[str] \| None` | No | Patrones a excluir |

#### 2.29.1 `run_command.app()`

Función anidada/local definida cuando `info.load_app()` falla bajo reloader.

```python
def app(environ: WSGIEnvironment, start_response: StartResponse) -> cabc.Iterable[bytes]:
    """WSGI app fallback que siempre lanza el error original de carga.

    Args:
        environ: Entorno WSGI.
        start_response: Callable start_response.

    Returns:
        Nunca retorna normalmente.

    Raises:
        Exception: El error capturado de info.load_app().
    """
```

### 2.30 `shell_command()`

```python
@click.command("shell")
@with_appcontext
def shell_command() -> None:
    """Ejecuta un shell interactivo dentro del app_context.

    Carga PYTHONSTARTUP, contexto de app.make_shell_context(), y hook
    interactivo con readline/rlcompleter. Usa code.interact.

    Returns:
        None.
    """
```

### 2.31 `routes_command()`

```python
@click.command("routes")
@with_appcontext
def routes_command(sort: str, all_methods: bool) -> None:
    """Muestra las rutas registradas en current_app.url_map.

    Args:
        sort: Campo de ordenamiento ("endpoint", "methods", "domain", "rule", "match").
        all_methods: Si True incluye HEAD y OPTIONS.

    Returns:
        None. Imprime tabla formateada vía click.echo.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `sort` | `str` | No (default `endpoint`) | Criterio de orden |
| `all_methods` | `bool` | No (default `False`) | Mostrar HEAD/OPTIONS |

### 2.32 `main()`

```python
def main() -> None:
    """Punto de entrada CLI. Invoca cli.main() del FlaskGroup global.

    Returns:
        None.
    """
```

---

## 3. Ejemplo de Uso

```python
# app.py
from flask import Flask

def create_app(env="development"):
    app = Flask(__name__)
    app.config["ENV"] = env

    @app.cli.command("hello")
    def hello():
        print(f"Hola desde {app.import_name} en {env}")

    @app.route("/")
    def index():
        return "OK"

    return app

# Uso programático del módulo flask.cli
from flask.cli import locate_app, ScriptInfo, FlaskGroup

# 1. Localizar app vía string con argumentos (equivale a --app "app:create_app('prod')")
app = locate_app("app", "create_app('prod')")
print(app.config["ENV"])  # prod

# 2. Usar ScriptInfo para carga lazy (como lo hace FlaskGroup internamente)
info = ScriptInfo(app_import_path="app:create_app('testing')")
app2 = info.load_app()

# 3. Crear CLI personalizado con factory
def make_app():
    return create_app(env="cli")

cli = FlaskGroup(create_app=make_app, add_default_commands=True)
# Ejecutar como: python -m flask --app app:create_app run --debug
# O programáticamente:
if __name__ == "__main__":
    cli.main(args=["routes", "--sort", "rule"])
    cli.main(args=["hello"])
```
