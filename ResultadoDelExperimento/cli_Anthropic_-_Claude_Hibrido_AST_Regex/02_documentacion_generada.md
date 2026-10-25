# Documentación Técnica — `flask/cli.py`

---

## 1. Resumen ejecutivo

Este módulo implementa la interfaz de línea de comandos (CLI) de Flask sobre `click`. Su responsabilidad principal es **descubrir, importar e instanciar** la aplicación Flask del usuario (mediante variables, factorías o archivos convencionales `wsgi.py` / `app.py`), gestionar la carga de variables de entorno desde archivos *dotenv*, y exponer los comandos integrados `run`, `shell` y `routes`, además de permitir la extensión mediante *entry points* (`flask.commands`) y comandos registrados en `app.cli` / `blueprint.cli`.

El punto de entrada público es el objeto `cli` (instancia de `FlaskGroup`) y la función `main()`, que es la que se vincula al ejecutable `flask`.

---

## 2. Especificación de funciones, clases y métodos

### 2.1 Mapa de símbolos del módulo

| Símbolo | Tipo | Descripción |
|---|---|---|
| `NoAppException` | Clase (`click.UsageError`) | Excepción lanzada cuando no se puede localizar o cargar la aplicación. |
| `find_best_app()` | Función | Heurística de descubrimiento de app dentro de un módulo. |
| `_called_with_wrong_args()` | Función privada | Discrimina el origen de un `TypeError`. |
| `find_app_by_string()` | Función | Resuelve `"name"` o `"name(args)"` contra un módulo. |
| `prepare_import()` | Función | Convierte una ruta de archivo en nombre de módulo importable. |
| `locate_app()` | Función (+2 `@t.overload`) | Importa el módulo y delega el descubrimiento. |
| `get_version()` | Función (callback) | Callback eager de `--version`. |
| `version_option` | `click.Option` | Opción `--version` (eager, `expose_value=False`). |
| `ScriptInfo` | Clase | Contenedor de estado para la carga diferida de la app. |
| `pass_script_info` | Decorador | `click.make_pass_decorator(ScriptInfo, ensure=True)`. |
| `F` | `t.TypeVar` | `TypeVar` acotado a `t.Callable[..., t.Any]`. |
| `with_appcontext()` | Decorador | Garantiza contexto de aplicación en el callback. |
| `AppGroup` | Clase (`click.Group`) | Grupo que envuelve callbacks con `with_appcontext`. |
| `_set_app()` | Función (callback) | Callback de `-A/--app`. |
| `_app_option` | `click.Option` | Opción `-A/--app` (eager). |
| `_set_debug()` | Función (callback) | Callback de `--debug/--no-debug`. |
| `_debug_option` | `click.Option` | Opción `--debug/--no-debug`. |
| `_env_file_callback()` | Función (callback) | Callback eager de `-e/--env-file`. |
| `_env_file_option` | `click.Option` | Opción `-e/--env-file` (eager). |
| `FlaskGroup` | Clase (`AppGroup`) | Grupo raíz con comandos por defecto y plugins. |
| `_path_is_ancestor()` | Función privada | Comprobación de ascendencia de rutas. |
| `load_dotenv()` | Función | Carga de archivos `.env` / `.flaskenv`. |
| `show_server_banner()` | Función | Mensajes de arranque del servidor. |
| `CertParamType` | Clase (`click.ParamType`) | Tipo de parámetro para `--cert`. |
| `_validate_key()` | Función (callback) | Validación cruzada de `--key` / `--cert`. |
| `SeparatedPathType` | Clase (`click.Path`) | Lista de rutas separadas por `os.pathsep`. |
| `run_command` | `click.Command` | Comando `run`. |
| `shell_command` | `click.Command` | Comando `shell`. |
| `routes_command` | `click.Command` | Comando `routes`. |
| `cli` | `FlaskGroup` | Grupo raíz `flask`. |
| `main()` | Función | Punto de entrada del ejecutable. |

---

### 2.2 `NoAppException`

```python
class NoAppException(click.UsageError):
    """Se lanza si una aplicación no puede ser encontrada o cargada.

    Hereda de ``click.UsageError``, por lo que click la formatea como error de
    uso (código de salida 2) y expone ``format_message()``.
    """
```

No define métodos propios.

---

### 2.3 `find_best_app()`

```python
def find_best_app(module: ModuleType) -> Flask:
    """Busca la mejor aplicación posible dentro de una instancia de módulo.

    Estrategia de resolución, en orden:
        1. Atributos ``app`` y ``application`` que sean instancias de ``Flask``.
        2. Única instancia de ``Flask`` presente en ``module.__dict__``.
        3. Factorías ``create_app`` o ``make_app`` invocadas sin argumentos.

    Args:
        module: Módulo ya importado donde se realizará la búsqueda.

    Returns:
        Flask: La instancia de aplicación detectada.

    Raises:
        NoAppException: Si hay múltiples instancias de ``Flask``, si una
            factoría detectada no puede invocarse sin argumentos, o si no se
            encuentra ni aplicación ni factoría.
        TypeError: Se re-propaga el ``TypeError`` original cuando éste se
            originó *dentro* del cuerpo de la factoría (determinado por
            ``_called_with_wrong_args()``).
    """
```

**Parámetros**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `module` | `types.ModuleType` | Sí | Módulo importado a inspeccionar. |

**Retorno**

| Tipo | Descripción |
|---|---|
| `Flask` | Aplicación encontrada directamente o producida por la factoría. |

**Excepciones**

| Excepción | Condición |
|---|---|
| `NoAppException` | Múltiples apps, factoría no invocable sin argumentos, o ninguna app/factoría. |
| `TypeError` | Error originado dentro de la factoría (se re-lanza sin envolver). |

---

### 2.4 `_called_with_wrong_args()`

```python
def _called_with_wrong_args(f: t.Callable[..., Flask]) -> bool:
    """Determina si un ``TypeError`` se debió a una firma incompatible.

    Recorre el traceback activo (``sys.exc_info()[2]``) comparando
    ``tb_frame.f_code`` con ``f.__code__``. Si el frame de ``f`` aparece en la
    pila, la llamada sí se efectuó y el error proviene del interior de ``f``.
    El traceback se elimina en el bloque ``finally`` para evitar una
    referencia circular.

    Args:
        f: La función que fue invocada.

    Returns:
        bool: ``True`` si la llamada falló por argumentos incorrectos (nunca se
            entró en la función); ``False`` si el error ocurrió dentro de ``f``.
    """
```

**Parámetros**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `f` | `t.Callable[..., Flask]` | Sí | Función/factoría invocada que provocó el `TypeError`. |

**Retorno**

| Tipo | Descripción |
|---|---|
| `bool` | `True` = fallo en la invocación; `False` = fallo interno. |

**Excepciones:** no lanza excepciones propias.

---

### 2.5 `find_app_by_string()`

```python
def find_app_by_string(module: ModuleType, app_name: str) -> Flask:
    """Resuelve ``app_name`` como nombre de atributo o como llamada a factoría.

    ``app_name`` se parsea con ``ast.parse(..., mode="eval")``. Se admiten
    únicamente dos formas:
        * ``ast.Name``  -> acceso directo al atributo.
        * ``ast.Call``  -> llamada a una factoría con argumentos literales
          (evaluados con ``ast.literal_eval``). El llamable debe ser un nombre
          simple (no un atributo punteado).

    Args:
        module: Módulo ya importado donde buscar el atributo.
        app_name: Expresión con el nombre o la llamada, p. ej. ``"app"`` o
            ``"create_app('dev')"``.

    Returns:
        Flask: La instancia de aplicación resultante.

    Raises:
        NoAppException: Si la expresión no es parseable, no es un nombre ni una
            llamada, el llamable no es un nombre simple, los argumentos no son
            literales, el atributo no existe en el módulo, la factoría no acepta
            los argumentos dados, o el resultado no es una instancia de ``Flask``.
        TypeError: Se re-propaga cuando el ``TypeError`` se originó dentro de la
            factoría.
    """
```

**Parámetros**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `module` | `types.ModuleType` | Sí | Módulo importado. |
| `app_name` | `str` | Sí | Nombre de atributo o expresión de llamada. |

**Retorno**

| Tipo | Descripción |
|---|---|
| `Flask` | Aplicación obtenida del atributo o de la factoría. |

**Excepciones**

| Excepción | Condición |
|---|---|
| `NoAppException` | `SyntaxError` al parsear, expresión no soportada, llamable no simple, `ValueError` en `literal_eval`, `AttributeError` al resolver, argumentos inválidos, resultado no `Flask`. |
| `TypeError` | Error interno de la factoría. |

---

### 2.6 `prepare_import()`

```python
def prepare_import(path: str) -> str:
    """Calcula el nombre de módulo importable para una ruta de archivo.

    Normaliza la ruta con ``os.path.realpath``, elimina la extensión ``.py``,
    reduce ``__init__`` al directorio del paquete y asciende por el árbol de
    directorios mientras exista ``__init__.py``. El directorio raíz resultante
    se inserta en ``sys.path[0]`` si aún no está allí.

    Args:
        path: Ruta al archivo o paquete que contiene la aplicación.

    Returns:
        str: Nombre de módulo punteado esperado, p. ej. ``"paquete.sub.wsgi"``.

    Note:
        Esta función tiene el efecto colateral de mutar ``sys.path``.
    """
```

**Parámetros**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `path` | `str` | Sí | Ruta de archivo o directorio de paquete. |

**Retorno**

| Tipo | Descripción |
|---|---|
| `str` | Nombre punteado del módulo. |

**Excepciones:** no lanza excepciones propias.

---

### 2.7 `locate_app()`

Incluye dos sobrecargas de tipado y la implementación real.

```python
@t.overload
def locate_app(
    module_name: str, app_name: str | None, raise_if_not_found: t.Literal[True] = True
) -> Flask: ...
```
> Sobrecarga de `locate_app()`: con `raise_if_not_found=True` el retorno se estrecha a `Flask` (nunca `None`).

```python
@t.overload
def locate_app(
    module_name: str, app_name: str | None, raise_if_not_found: t.Literal[False] = ...
) -> Flask | None: ...
```
> Sobrecarga de `locate_app()`: con `raise_if_not_found=False` el retorno puede ser `None`.

```python
def locate_app(
    module_name: str, app_name: str | None, raise_if_not_found: bool = True
) -> Flask | None:
    """Importa ``module_name`` y localiza la aplicación en él.

    Distingue entre "el módulo no existe" y "el módulo falló al importarse":
    si el traceback del ``ImportError`` tiene profundidad > 1
    (``tb_next`` no es ``None``), el error ocurrió dentro del módulo importado y
    se envuelve incluyendo el traceback formateado.

    Args:
        module_name: Nombre punteado del módulo a importar.
        app_name: Nombre o expresión de la app. Si es ``None`` se usa
            ``find_best_app()``; en caso contrario ``find_app_by_string()``.
        raise_if_not_found: Si ``True``, lanza ``NoAppException`` cuando el
            módulo no puede importarse; si ``False``, devuelve ``None``.

    Returns:
        Flask | None: La aplicación localizada, o ``None`` cuando el módulo no
            existe y ``raise_if_not_found`` es ``False``.

    Raises:
        NoAppException: Si el módulo falló internamente al importarse, si no se
            pudo importar (con ``raise_if_not_found=True``), o si falla el
            descubrimiento delegado.
    """
```

**Parámetros**

| Nombre | Tipo | Requerido | Por defecto | Descripción |
|---|---|---|---|---|
| `module_name` | `str` | Sí | — | Módulo a importar. |
| `app_name` | `str \| None` | Sí | — | Nombre/expresión de la app o `None`. |
| `raise_if_not_found` | `bool` | No | `True` | Controla si se lanza o se devuelve `None`. |

**Retorno**

| Tipo | Descripción |
|---|---|
| `Flask \| None` | Aplicación localizada o `None`. |

**Excepciones**

| Excepción | Condición |
|---|---|
| `NoAppException` | `ImportError` interno del módulo, módulo inexistente, o fallo del descubrimiento. |

---

### 2.8 `get_version()`

```python
def get_version(ctx: click.Context, param: click.Parameter, value: t.Any) -> None:
    """Callback eager de la opción ``--version``.

    Imprime la versión de Python, Flask y Werkzeug (obtenidas vía
    ``importlib.metadata.version``) y finaliza el programa. No hace nada si
    ``value`` es falsy o si el contexto está en ``resilient_parsing``
    (autocompletado).

    Args:
        ctx: Contexto de click activo.
        param: Parámetro que disparó el callback.
        value: Valor del flag ``--version``.

    Returns:
        None

    Raises:
        click.exceptions.Exit: Propagada por ``ctx.exit()`` para terminar la
            ejecución tras imprimir la versión.
    """
```

**Parámetros**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `ctx` | `click.Context` | Sí | Contexto de ejecución. |
| `param` | `click.Parameter` | Sí | Parámetro asociado. |
| `value` | `t.Any` | Sí | Valor del flag. |

**Retorno:** `None`.

---

### 2.9 Clase `ScriptInfo`

```python
class ScriptInfo:
    """Objeto auxiliar para la carga diferida de aplicaciones Flask.

    Normalmente lo crea automáticamente :class:`FlaskGroup` y se propaga como
    objeto de contexto de click (``ctx.obj``).

    Attributes:
        app_import_path (str | None): Ruta de importación de la aplicación.
        create_app (t.Callable[..., Flask] | None): Factoría alternativa.
        data (dict[t.Any, t.Any]): Diccionario libre para datos arbitrarios.
        set_debug_flag (bool): Si se debe propagar el flag de depuración a la app.
        load_dotenv_defaults (bool): Si deben cargarse ``.flaskenv`` y ``.env``
            por defecto (valor normalizado con ``get_load_dotenv``).

    .. versionchanged:: 3.1
        Añadidos el parámetro y atributo ``load_dotenv_defaults``.
    """
```

#### 2.9.1 `ScriptInfo.__init__()`

```python
def __init__(
    self,
    app_import_path: str | None = None,
    create_app: t.Callable[..., Flask] | None = None,
    set_debug_flag: bool = True,
    load_dotenv_defaults: bool = True,
) -> None:
    """Inicializa el contenedor de información del script.

    Args:
        app_import_path: Ruta de importación opcional de la app
            (``"modulo:nombre"``).
        create_app: Factoría opcional que devuelve la aplicación.
        set_debug_flag: Si ``True``, ``load_app()`` ajustará ``app.debug``.
        load_dotenv_defaults: Preferencia de carga de archivos dotenv por
            defecto; se normaliza con ``get_load_dotenv()``.

    Returns:
        None
    """
```

| Nombre | Tipo | Requerido | Por defecto |
|---|---|---|---|
| `app_import_path` | `str \| None` | No | `None` |
| `create_app` | `t.Callable[..., Flask] \| None` | No | `None` |
| `set_debug_flag` | `bool` | No | `True` |
| `load_dotenv_defaults` | `bool` | No | `True` |

#### 2.9.2 `ScriptInfo.load_app()`

```python
def load_app(self) -> Flask:
    """Carga (una sola vez) y devuelve la aplicación Flask.

    Orden de resolución:
        1. ``self._loaded_app`` si ya fue cargada (memoización).
        2. ``self.create_app()`` si se proporcionó una factoría.
        3. ``self.app_import_path``, dividido con la expresión
           ``r":(?![\\\\/])"`` en ``(path, name)``, normalizado con
           ``prepare_import()`` y resuelto con ``locate_app()``.
        4. Búsqueda secuencial de ``wsgi.py`` y ``app.py`` con
           ``raise_if_not_found=False``.

    Si ``self.set_debug_flag`` es ``True``, asigna ``app.debug = get_debug_flag()``
    mediante el descriptor para que los valores dependientes se repueblen.

    Returns:
        Flask: La aplicación cargada y memoizada.

    Raises:
        NoAppException: Si no se pudo localizar ninguna aplicación.
    """
```

**Retorno**

| Tipo | Descripción |
|---|---|
| `Flask` | Aplicación cargada (cacheada en `_loaded_app`). |

**Excepciones**

| Excepción | Condición |
|---|---|
| `NoAppException` | No hay factoría, ruta válida ni `wsgi.py` / `app.py`. |

---

### 2.10 `with_appcontext()` y su función anidada `decorator()`

```python
def with_appcontext(f: F) -> F:
    """Envuelve un callback para que se ejecute dentro del contexto de aplicación.

    No es necesario para comandos registrados bajo ``app.cli`` o
    ``blueprint.cli``, que ya disponen de contexto.

    Args:
        f: Callback de click a envolver.

    Returns:
        F: El callback envuelto, con los metadatos copiados vía
            ``update_wrapper``.

    .. versionchanged:: 2.2
        El contexto de aplicación permanece activo también para los subcomandos.
    """
```

**Parámetros**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `f` | `F` (`t.Callable[..., t.Any]`) | Sí | Callback a decorar. |

**Retorno**

| Tipo | Descripción |
|---|---|
| `F` | Wrapper decorado con `@click.pass_context`. |

#### 2.10.1 Función anidada `decorator()` (dentro de `with_appcontext`)

```python
@click.pass_context
def decorator(ctx: click.Context, /, *args: t.Any, **kwargs: t.Any) -> t.Any:
    """Empuja un contexto de aplicación si no hay uno activo e invoca ``f``.

    Si ``current_app`` no está vinculado, obtiene el ``ScriptInfo`` del contexto
    con ``ctx.ensure_object()``, carga la app y registra ``app.app_context()``
    como recurso del contexto (``ctx.with_resource``), de modo que permanezca
    activo durante los subcomandos.

    Args:
        ctx: Contexto de click (posicional-only, inyectado por
            ``click.pass_context``).
        *args: Argumentos posicionales reenviados a ``f``.
        **kwargs: Argumentos nominales reenviados a ``f``.

    Returns:
        t.Any: El valor devuelto por ``ctx.invoke(f, *args, **kwargs)``.

    Raises:
        NoAppException: Propagada por ``ScriptInfo.load_app()``.
    """
```

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `ctx` | `click.Context` | Sí (posicional-only) | Inyectado por `click.pass_context`. |
| `*args` | `t.Any` | No | Reenviados a `f`. |
| `**kwargs` | `t.Any` | No | Reenviados a `f`. |

---

### 2.11 Clase `AppGroup`

```python
class AppGroup(click.Group):
    """Grupo de click cuyo decorador ``command`` aplica ``with_appcontext``.

    No debe confundirse con :class:`FlaskGroup`.
    """
```

#### 2.11.1 `AppGroup.command()`

```python
def command(
    self, *args: t.Any, **kwargs: t.Any
) -> t.Callable[[t.Callable[..., t.Any]], click.Command]:
    """Igual que ``click.Group.command`` pero envuelve el callback.

    El envoltorio con :func:`with_appcontext` puede desactivarse pasando
    ``with_appcontext=False`` (kwarg consumido y no propagado a click).

    Args:
        *args: Argumentos posicionales reenviados a ``click.Group.command``.
        **kwargs: Argumentos nominales reenviados a ``click.Group.command``;
            la clave ``with_appcontext`` (bool, por defecto ``True``) se extrae
            con ``kwargs.pop``.

    Returns:
        t.Callable[[t.Callable[..., t.Any]], click.Command]: Decorador que
            registra el comando en el grupo.
    """
```

| Nombre | Tipo | Requerido | Por defecto | Descripción |
|---|---|---|---|---|
| `*args` | `t.Any` | No | — | Reenviados a `click.Group.command`. |
| `**kwargs` | `t.Any` | No | — | Reenviados a click. |
| `kwargs["with_appcontext"]` | `bool` | No | `True` | Desactiva el envoltorio si es `False`. |

##### Función anidada `decorator()` (dentro de `AppGroup.command`)

```python
def decorator(f: t.Callable[..., t.Any]) -> click.Command:
    """Aplica opcionalmente ``with_appcontext`` y registra el comando.

    Args:
        f: Función callback del comando.

    Returns:
        click.Command: El comando creado por
            ``super(AppGroup, self).command(*args, **kwargs)(f)``.
    """
```

#### 2.11.2 `AppGroup.group()`

```python
def group(
    self, *args: t.Any, **kwargs: t.Any
) -> t.Callable[[t.Callable[..., t.Any]], click.Group]:
    """Igual que ``click.Group.group`` pero con ``cls`` por defecto ``AppGroup``.

    Args:
        *args: Argumentos posicionales reenviados a ``click.Group.group``.
        **kwargs: Argumentos nominales; ``cls`` se fija con
            ``setdefault("cls", AppGroup)``.

    Returns:
        t.Callable[[t.Callable[..., t.Any]], click.Group]: Decorador que
            registra el subgrupo.
    """
```

| Nombre | Tipo | Requerido | Por defecto | Descripción |
|---|---|---|---|---|
| `*args` | `t.Any` | No | — | Reenviados a click. |
| `**kwargs` | `t.Any` | No | — | Reenviados a click. |
| `kwargs["cls"]` | `type` | No | `AppGroup` | Clase del subgrupo. |

---

### 2.12 `_set_app()`

```python
def _set_app(ctx: click.Context, param: click.Option, value: str | None) -> str | None:
    """Callback eager de ``-A/--app``: fija ``ScriptInfo.app_import_path``.

    Args:
        ctx: Contexto de click; se obtiene/crea el ``ScriptInfo`` con
            ``ctx.ensure_object``.
        param: Opción que disparó el callback.
        value: Ruta de importación ``"modulo:nombre"`` o ``None``.

    Returns:
        str | None: El mismo ``value`` recibido (``None`` si no se proporcionó).
    """
```

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `ctx` | `click.Context` | Sí | Contexto activo. |
| `param` | `click.Option` | Sí | Opción `-A/--app`. |
| `value` | `str \| None` | Sí | Valor de la opción. |

**Retorno:** `str | None`.

---

### 2.13 `_set_debug()`

```python
def _set_debug(ctx: click.Context, param: click.Option, value: bool) -> bool | None:
    """Callback de ``--debug/--no-debug``: escribe ``FLASK_DEBUG`` en el entorno.

    Si el origen del parámetro (``ctx.get_parameter_source``) es
    ``ParameterSource.DEFAULT`` o ``DEFAULT_MAP``, no se modifica nada para que
    el modo depuración pueda provenir del entorno.

    Se usa la variable de entorno en lugar de ``ScriptInfo`` para que el valor
    esté disponible de forma temprana dentro de una factoría.

    Args:
        ctx: Contexto de click.
        param: Opción que disparó el callback.
        value: Valor booleano del flag.

    Returns:
        bool | None: El valor recibido, o ``None`` si provino de un valor por
            defecto.
    """
```

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `ctx` | `click.Context` | Sí | Contexto activo. |
| `param` | `click.Option` | Sí | Opción `--debug/--no-debug`. |
| `value` | `bool` | Sí | Estado del flag. |

**Retorno:** `bool | None`. **Efecto colateral:** `os.environ["FLASK_DEBUG"]`.

---

### 2.14 `_env_file_callback()`

```python
def _env_file_callback(
    ctx: click.Context, param: click.Option, value: str | None
) -> str | None:
    """Callback eager de ``-e/--env-file``: carga variables de entorno.

    Intenta importar ``dotenv``. Si no está instalado y se pasó un valor
    explícito, aborta con ``click.BadParameter``; si no se pasó valor, continúa
    para que ``load_dotenv()`` emita únicamente un aviso.

    Invoca ``load_dotenv(value, load_defaults=ctx.obj.load_dotenv_defaults)``
    cuando hay un valor explícito o cuando los defaults están habilitados.

    Args:
        ctx: Contexto de click; ``ctx.obj`` debe ser un ``ScriptInfo``.
        param: Opción que disparó el callback.
        value: Ruta del archivo de entorno o ``None``.

    Returns:
        str | None: El mismo ``value`` recibido.

    Raises:
        click.BadParameter: Si se indicó un archivo pero ``python-dotenv`` no
            está instalado.
    """
```

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `ctx` | `click.Context` | Sí | Contexto; `ctx.obj` → `ScriptInfo`. |
| `param` | `click.Option` | Sí | Opción `-e/--env-file`. |
| `value` | `str \| None` | Sí | Ruta del archivo. |

**Excepciones:** `click.BadParameter`.

---

### 2.15 Clase `FlaskGroup`

```python
class FlaskGroup(AppGroup):
    """Subclase de :class:`AppGroup` que además carga comandos de la app Flask.

    Args (constructor):
        add_default_commands: Añade ``run``, ``shell`` y ``routes``.
        add_version_option: Añade la opción ``--version``.
        create_app: Callback opcional que devuelve la app cargada.
        load_dotenv: Carga los archivos ``.env`` y ``.flaskenv`` más próximos.
        set_debug_flag: Propaga el flag de depuración a la app.

    .. versionchanged:: 3.1
        ``-e path`` tiene precedencia sobre ``.env`` y ``.flaskenv``.
    .. versionchanged:: 2.2
        Añadidas las opciones ``-A/--app``, ``--debug/--no-debug``,
        ``-e/--env-file``; se empuja un contexto de app para comandos de
        ``app.cli``.
    """
```

#### 2.15.1 `FlaskGroup.__init__()`

```python
def __init__(
    self,
    add_default_commands: bool = True,
    create_app: t.Callable[..., Flask] | None = None,
    add_version_option: bool = True,
    load_dotenv: bool = True,
    set_debug_flag: bool = True,
    **extra: t.Any,
) -> None:
    """Construye el grupo raíz de la CLI.

    Inserta en ``params`` (en este orden) ``_env_file_option``, ``_app_option``
    y ``_debug_option`` —``--env-file`` debe evaluarse primero—, y opcionalmente
    ``version_option``. Fija ``context_settings["auto_envvar_prefix"] = "FLASK"``
    mediante ``setdefault``.

    Args:
        add_default_commands: Registra ``run_command``, ``shell_command`` y
            ``routes_command``.
        create_app: Factoría propagada al ``ScriptInfo``.
        add_version_option: Añade ``--version``.
        load_dotenv: Valor propagado a ``ScriptInfo.load_dotenv_defaults``.
        set_debug_flag: Valor propagado al ``ScriptInfo``.
        **extra: Argumentos adicionales para ``click.Group``; se consume
            ``params`` y se completa ``context_settings``.

    Returns:
        None
    """
```

| Nombre | Tipo | Requerido | Por defecto |
|---|---|---|---|
| `add_default_commands` | `bool` | No | `True` |
| `create_app` | `t.Callable[..., Flask] \| None` | No | `None` |
| `add_version_option` | `bool` | No | `True` |
| `load_dotenv` | `bool` | No | `True` |
| `set_debug_flag` | `bool` | No | `True` |
| `**extra` | `t.Any` | No | — |

#### 2.15.2 `FlaskGroup._load_plugin_commands()`

```python
def _load_plugin_commands(self) -> None:
    """Registra los comandos publicados en el entry point ``flask.commands``.

    Idempotente: se protege con el atributo ``self._loaded_plugin_commands``.

    Returns:
        None
    """
```

**Parámetros:** ninguno (aparte de `self`). **Retorno:** `None`.

#### 2.15.3 `FlaskGroup.get_command()`

```python
def get_command(self, ctx: click.Context, name: str) -> click.Command | None:
    """Resuelve un comando: integrados/plugins primero, luego los de la app.

    Si el comando no es integrado ni de plugin, carga la app vía
    ``ScriptInfo.load_app()``. Si la carga falla con ``NoAppException``, imprime
    el mensaje en rojo por ``stderr`` y devuelve ``None``. Si la app se carga,
    empuja su contexto con ``ctx.with_resource(app.app_context())`` salvo que ya
    esté activo, y delega en ``app.cli.get_command``.

    Args:
        ctx: Contexto de click.
        name: Nombre del comando solicitado.

    Returns:
        click.Command | None: El comando encontrado, o ``None`` si no existe o
            la app no pudo cargarse.
    """
```

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `ctx` | `click.Context` | Sí | Contexto activo. |
| `name` | `str` | Sí | Nombre del comando. |

#### 2.15.4 `FlaskGroup.list_commands()`

```python
def list_commands(self, ctx: click.Context) -> list[str]:
    """Lista comandos integrados, de plugins y de la aplicación.

    Los errores de carga se reportan sin interrumpir: ``NoAppException`` se
    muestra con ``format_message()``; cualquier otra excepción se muestra con
    el traceback completo.

    Args:
        ctx: Contexto de click.

    Returns:
        list[str]: Nombres de comando ordenados alfabéticamente y sin duplicados.
    """
```

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `ctx` | `click.Context` | Sí | Contexto activo. |

#### 2.15.5 `FlaskGroup.make_context()`

```python
def make_context(
    self,
    info_name: str | None,
    args: list[str],
    parent: click.Context | None = None,
    **extra: t.Any,
) -> click.Context:
    """Crea el contexto de click e inyecta un ``ScriptInfo`` como ``obj``.

    Fija ``FLASK_RUN_FROM_CLI="true"`` para que ``app.run()`` sea un no-op y no
    arranque el servidor al importar el módulo de la aplicación.

    Args:
        info_name: Nombre con el que se invocó el programa/comando.
        args: Lista de argumentos sin procesar.
        parent: Contexto padre, si existe.
        **extra: Extras para ``click.Group.make_context``; se añade ``obj`` solo
            si no está presente ni en ``extra`` ni en ``self.context_settings``.

    Returns:
        click.Context: El contexto construido.
    """
```

| Nombre | Tipo | Requerido | Por defecto |
|---|---|---|---|
| `info_name` | `str \| None` | Sí | — |
| `args` | `list[str]` | Sí | — |
| `parent` | `click.Context \| None` | No | `None` |
| `**extra` | `t.Any` | No | — |

#### 2.15.6 `FlaskGroup.parse_args()`

```python
def parse_args(self, ctx: click.Context, args: list[str]) -> list[str]:
    """Parsea argumentos forzando la evaluación temprana de ``--env-file``/``--app``.

    Cuando no hay argumentos y ``no_args_is_help`` está activo, o cuando el
    único argumento es una opción de ayuda, el procesamiento eager se omite;
    por ello se invoca manualmente
    ``_env_file_option.handle_parse_result(ctx, {}, [])`` y
    ``_app_option.handle_parse_result(ctx, {}, [])`` para que ``FLASK_APP`` se
    tenga en cuenta y la ayuda incluya los comandos de ``app.cli``.

    Args:
        ctx: Contexto de click.
        args: Argumentos sin procesar.

    Returns:
        list[str]: Argumentos restantes tras el parseo, según ``click.Group``.
    """
```

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `ctx` | `click.Context` | Sí | Contexto activo. |
| `args` | `list[str]` | Sí | Argumentos de entrada. |

---

### 2.16 `_path_is_ancestor()`

```python
def _path_is_ancestor(path: str, other: str) -> bool:
    """Determina si ``path`` es ancestro de ``other``.

    Elimina de ``other`` tantos caracteres como la longitud de ``path``, limpia
    los separadores iniciales y vuelve a unir con ``path``. Si el resultado es
    igual a ``other``, existe relación de ascendencia.

    Args:
        path: Ruta candidata a ancestro.
        other: Ruta descendiente a comprobar.

    Returns:
        bool: ``True`` si ``path`` es ancestro de ``other``.
    """
```

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `path` | `str` | Sí | Ruta base. |
| `other` | `str` | Sí | Ruta a comparar. |

---

### 2.17 `load_dotenv()`

```python
def load_dotenv(
    path: str | os.PathLike[str] | None = None, load_defaults: bool = True
) -> bool:
    """Carga archivos "dotenv" para establecer variables de entorno.

    Precedencia: ``path`` > ``.env`` > ``.flaskenv``. Tras combinar los
    archivos, los valores solo se asignan si la clave no existe ya en
    ``os.environ``. Los archivos por defecto se localizan con
    ``dotenv.find_dotenv(..., usecwd=True)`` y se leen en UTF-8.

    Es un no-op si `python-dotenv` no está instalado; en ese caso, si existen
    archivos dotenv, se emite un aviso informativo por ``stderr``.

    Args:
        path: Ruta del archivo a cargar.
        load_defaults: Buscar y cargar los archivos ``.flaskenv`` y ``.env``.

    Returns:
        bool: ``True`` si se cargó al menos una variable de entorno.

    .. versionchanged:: 3.1
        Añadido ``load_defaults``; una ruta explícita tiene precedencia.
    .. versionchanged:: 2.0
        No se cambia el directorio actual; codificación por defecto UTF-8.
    .. versionchanged:: 1.1.0
        Devuelve ``False`` si python-dotenv no está instalado o la ruta no es
        un archivo.
    .. versionadded:: 1.0
    """
```

| Nombre | Tipo | Requerido | Por defecto | Descripción |
|---|---|---|---|---|
| `path` | `str \| os.PathLike[str] \| None` | No | `None` | Archivo explícito a cargar. |
| `load_defaults` | `bool` | No | `True` | Cargar `.flaskenv` y `.env`. |

**Retorno**

| Tipo | Descripción |
|---|---|
| `bool` | `True` si el diccionario combinado no está vacío. |

**Excepciones:** no lanza excepciones propias (el `ImportError` de `dotenv` se captura).

---

### 2.18 `show_server_banner()`

```python
def show_server_banner(debug: bool, app_import_path: str | None) -> None:
    """Imprime los mensajes de arranque del servidor una sola vez.

    Retorna inmediatamente si el proceso proviene del recargador
    (``is_running_from_reloader()``), para no duplicar la salida.

    Args:
        debug: Estado del modo depuración, usado para el mensaje
            ``Debug mode: on/off``.
        app_import_path: Ruta de importación mostrada como
            ``Serving Flask app '<path>'``; se omite si es ``None``.

    Returns:
        None
    """
```

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `debug` | `bool` | Sí | Estado del modo depuración. |
| `app_import_path` | `str \| None` | Sí | Ruta de importación a mostrar. |

---

### 2.19 Clase `CertParamType`

```python
class CertParamType(click.ParamType):
    """Tipo de parámetro de click para la opción ``--cert``.

    Admite un archivo existente, la cadena ``'adhoc'``, o una ruta de
    importación que resuelva a un objeto :class:`ssl.SSLContext`.

    Attributes:
        name (str): Nombre del tipo expuesto por click (``"path"``).
        path_type (click.Path): Validador interno de rutas.
    """
```

#### 2.19.1 `CertParamType.__init__()`

```python
def __init__(self) -> None:
    """Inicializa el validador interno.

    Crea ``self.path_type`` como
    ``click.Path(exists=True, dir_okay=False, resolve_path=True)``.

    Returns:
        None
    """
```

**Parámetros:** ninguno. **Retorno:** `None`.

#### 2.19.2 `CertParamType.convert()`

```python
def convert(
    self, value: t.Any, param: click.Parameter | None, ctx: click.Context | None
) -> t.Any:
    """Convierte el valor de ``--cert`` a ruta, ``'adhoc'`` o ``SSLContext``.

    Secuencia:
        1. Verifica que Python tenga soporte SSL (``import ssl``).
        2. Intenta validarlo como ruta de archivo existente.
        3. Si falla, lo normaliza a minúsculas; si es ``"adhoc"`` comprueba que
           ``cryptography`` esté instalado y lo devuelve.
        4. En otro caso, intenta ``import_string(value, silent=True)`` y
           devuelve el objeto si es una instancia de ``ssl.SSLContext``.

    Args:
        value: Valor crudo recibido de la línea de comandos.
        param: Parámetro asociado, si existe.
        ctx: Contexto de click, si existe.

    Returns:
        t.Any: Ruta absoluta del certificado, la cadena ``"adhoc"`` o un objeto
            ``ssl.SSLContext``.

    Raises:
        click.BadParameter: Si Python no tiene soporte SSL, o si se solicita
            ``adhoc`` sin la librería ``cryptography``. Además, se re-lanza la
            ``click.BadParameter`` original de validación de ruta cuando el
            valor no es ``adhoc`` ni resuelve a un ``SSLContext``.
    """
```

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `value` | `t.Any` | Sí | Valor de entrada. |
| `param` | `click.Parameter \| None` | Sí | Parámetro asociado. |
| `ctx` | `click.Context \| None` | Sí | Contexto activo. |

---

### 2.20 `_validate_key()`

```python
def _validate_key(ctx: click.Context, param: click.Parameter, value: t.Any) -> t.Any:
    """Valida ``--key`` en función del valor de ``--cert``.

    Cuando ``--cert`` es un archivo, ``--key`` es obligatorio y el parámetro
    ``cert`` del contexto se reescribe como la tupla ``(cert, key)``. Cuando
    ``--cert`` es ``'adhoc'`` o un ``SSLContext``, ``--key`` no debe usarse.

    Args:
        ctx: Contexto de click; se lee y modifica ``ctx.params["cert"]``.
        param: Parámetro ``--key``.
        value: Ruta del archivo de clave o ``None``.

    Returns:
        t.Any: El mismo ``value`` recibido.

    Raises:
        click.BadParameter: Si se pasa ``--key`` con ``--cert adhoc``, con un
            ``SSLContext``, o sin ``--cert``; o si se pasa un ``--cert`` de tipo
            archivo sin ``--key``.
    """
```

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `ctx` | `click.Context` | Sí | Contexto activo. |
| `param` | `click.Parameter` | Sí | Parámetro `--key`. |
| `value` | `t.Any` | Sí | Ruta de la clave privada. |

---

### 2.21 Clase `SeparatedPathType`

```python
class SeparatedPathType(click.Path):
    """Tipo de click que acepta varias rutas separadas por ``os.pathsep``.

    Usa ``:`` en POSIX y ``;`` en Windows. Cada elemento se valida con la
    lógica de :class:`click.Path`.
    """
```

#### 2.21.1 `SeparatedPathType.convert()`

```python
def convert(
    self, value: t.Any, param: click.Parameter | None, ctx: click.Context | None
) -> t.Any:
    """Divide el valor y valida cada elemento como ruta.

    Divide con ``self.split_envvar_value(value)`` y aplica la conversión de la
    clase base a cada ítem. La referencia ``super().convert`` se guarda en una
    variable local porque no puede invocarse ``super()`` sin argumentos dentro
    de una comprensión de lista antes de Python 3.12.

    Args:
        value: Cadena con rutas separadas por el separador del sistema.
        param: Parámetro asociado, si existe.
        ctx: Contexto de click, si existe.

    Returns:
        list[t.Any]: Lista de rutas validadas por ``click.Path.convert``.

    Raises:
        click.BadParameter: Propagada por la clase base si alguna ruta no es
            válida según la configuración del ``click.Path``.
    """
```

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `value` | `t.Any` | Sí | Cadena con rutas concatenadas. |
| `param` | `click.Parameter \| None` | Sí | Parámetro asociado. |
| `ctx` | `click.Context \| None` | Sí | Contexto activo. |

---

### 2.22 `run_command()`

Comando `run` (`@click.command("run")`, decorado con `@pass_script_info`). Tras su definición se ejecuta `run_command.params.insert(0, _debug_option)` para que `--debug` aparezca como primer parámetro del comando.

```python
def run_command(
    info: ScriptInfo,
    host: str,
    port: int,
    reload: bool,
    debugger: bool,
    with_threads: bool,
    cert: ssl.SSLContext | tuple[str, str | None] | t.Literal["adhoc"] | None,
    extra_files: list[str] | None,
    exclude_patterns: list[str] | None,
) -> None:
    """Ejecuta un servidor de desarrollo local.

    Este servidor es solo para desarrollo: no ofrece la estabilidad, seguridad
    ni rendimiento de un servidor WSGI de producción. El recargador y el
    depurador se activan por defecto con la opción ``--debug``.

    Si la carga de la app falla y el proceso proviene del recargador, el error
    se imprime de inmediato y se sustituye la aplicación por un callable WSGI
    que re-lanza la excepción, permitiendo que el depurador/recargador la
    gestione. Si no hay recargador, el error se propaga de inmediato.

    Args:
        info: ``ScriptInfo`` inyectado por ``pass_script_info``.
        host: Interfaz de escucha (``--host/-h``, por defecto ``127.0.0.1``).
        port: Puerto de escucha (``--port/-p``, por defecto ``5000``).
        reload: Activa/desactiva el recargador; ``None`` hereda de ``debug``.
        debugger: Activa/desactiva el depurador; ``None`` hereda de ``debug``.
        with_threads: Activa el multihilo (por defecto ``True``).
        cert: Contexto SSL, par ``(cert, key)``, ``"adhoc"`` o ``None``.
        extra_files: Archivos adicionales que disparan recarga.
        exclude_patterns: Patrones ``fnmatch`` excluidos de la recarga.

    Returns:
        None

    Raises:
        Exception: Se re-lanza cualquier error de carga de la app cuando no se
            está ejecutando bajo el recargador.
        NoAppException: Caso particular del anterior, originado en
            ``ScriptInfo.load_app()``.
    """
```

**Parámetros / opciones CLI**

| Parámetro | Opción CLI | Tipo | Requerido | Por defecto |
|---|---|---|---|---|
| `info` | — (inyectado) | `ScriptInfo` | Sí | `pass_script_info` |
| — | `--debug/--no-debug` | `bool` | No | entorno (`_debug_option`) |
| `host` | `--host`, `-h` | `str` | No | `"127.0.0.1"` |
| `port` | `--port`, `-p` | `int` | No | `5000` |
| `cert` | `--cert` | `CertParamType()` | No | `None` (eager) |
| — | `--key` | `click.Path` | No | `None` (`expose_value=False`, valida con `_validate_key`) |
| `reload` | `--reload/--no-reload` | `bool \| None` | No | `None` |
| `debugger` | `--debugger/--no-debugger` | `bool \| None` | No | `None` |
| `with_threads` | `--with-threads/--without-threads` | `bool` | No | `True` |
| `extra_files` | `--extra-files` | `SeparatedPathType()` | No | `None` |
| `exclude_patterns` | `--exclude-patterns` | `SeparatedPathType()` | No | `None` |

#### 2.22.1 Función anidada `app()` (dentro de `run_command`)

Se define únicamente en la rama de error cuando `is_running_from_reloader()` es verdadero, sombreando la variable `app` y actuando como aplicación WSGI sustituta.

```python
def app(
    environ: WSGIEnvironment, start_response: StartResponse
) -> cabc.Iterable[bytes]:
    """Aplicación WSGI de reemplazo que re-lanza el error de carga.

    Permite que el servidor arranque bajo el recargador aun cuando la
    aplicación real no pudo importarse, de modo que el depurador muestre el
    error en cada petición y la recarga corrija el problema.

    Args:
        environ: Diccionario de entorno WSGI de la petición.
        start_response: Callable WSGI para iniciar la respuesta.

    Returns:
        cabc.Iterable[bytes]: Nunca retorna; la firma se mantiene por contrato
            WSGI.

    Raises:
        Exception: La excepción capturada (``err``) durante la carga de la app,
            re-lanzada con ``raise err from None``.
    """
```

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `environ` | `WSGIEnvironment` | Sí | Entorno WSGI. |
| `start_response` | `StartResponse` | Sí | Callable de inicio de respuesta. |

---

### 2.23 `shell_command()`

Comando `shell` (`@click.command("shell")`, `@with_appcontext`).

```python
def shell_command() -> None:
    """Abre un shell interactivo de Python en el contexto de la aplicación.

    El espacio de nombres se construye con ``current_app.make_shell_context()``.
    Si la variable ``PYTHONSTARTUP`` apunta a un archivo existente, se compila y
    ejecuta previamente sobre ese mismo espacio de nombres. Si existe
    ``sys.__interactivehook__``, se configura ``readline`` con
    ``rlcompleter.Completer(ctx).complete`` —para que el autocompletado use el
    contexto del shell en lugar de ``__main__.__dict__``— y se invoca el hook.
    Finalmente se entra en ``code.interact()`` con un banner informativo
    (versión de Python, plataforma, ``import_name`` e ``instance_path``).

    Returns:
        None
    """
```

**Parámetros:** ninguno (requiere contexto de aplicación activo, garantizado por `with_appcontext`).
**Retorno:** `None`.

---

### 2.24 `routes_command()`

Comando `routes` (`@click.command("routes")`, `@with_appcontext`).

```python
def routes_command(sort: str, all_methods: bool) -> None:
    """Muestra todas las rutas registradas con sus endpoints y métodos.

    Recorre ``current_app.url_map.iter_rules()``. Si ``host_matching`` está
    activo se muestra la columna ``Host``; en caso contrario ``Subdomain``, y
    solo si alguna regla la define. Las columnas se alinean calculando el ancho
    máximo por columna y se separan por dos espacios. Un valor de ``sort`` no
    presente en las columnas disponibles (por ejemplo ``"match"``) provoca un
    ``ValueError`` que se ignora, preservando el orden de coincidencia de Flask.

    Args:
        sort: Criterio de ordenación: ``"endpoint"``, ``"methods"``,
            ``"domain"``, ``"rule"`` o ``"match"``.
        all_methods: Si ``True``, incluye los métodos ``HEAD`` y ``OPTIONS``.

    Returns:
        None
    """
```

| Parámetro | Opción CLI | Tipo | Requerido | Por defecto |
|---|---|---|---|---|
| `sort` | `--sort`, `-s` | `click.Choice(("endpoint","methods","domain","rule","match"))` | No | `"endpoint"` |
| `all_methods` | `--all-methods` | `bool` (flag) | No | `False` |

**Comportamiento especial:** si no hay reglas, imprime `"No routes were registered."` y retorna.

---

### 2.25 `main()`

```python
def main() -> None:
    """Punto de entrada del ejecutable ``flask``.

    Delega en ``cli.main()``, el grupo :class:`FlaskGroup` configurado a nivel
    de módulo. Se invoca también bajo la guarda ``if __name__ == "__main__"``.

    Returns:
        None

    Raises:
        SystemExit: Propagada por click al finalizar el procesamiento de la CLI.
    """
```

**Parámetros:** ninguno. **Retorno:** `None`.

---

## 3. Ejemplo de uso

El siguiente script define una aplicación con patrón *factory*, registra comandos personalizados sobre un `AppGroup`, y construye un punto de entrada propio basado en `FlaskGroup`, utilizando además las utilidades de descubrimiento del módulo.

```python
"""manage.py — CLI personalizada basada en flask.cli."""

from __future__ import annotations

import click
from flask import Flask

from flask.cli import (
    AppGroup,
    FlaskGroup,
    NoAppException,
    ScriptInfo,
    load_dotenv,
    locate_app,
    prepare_import,
    with_appcontext,
)


# --------------------------------------------------------------------------
# 1. Factoría de la aplicación
# --------------------------------------------------------------------------
def create_app(config_name: str = "development") -> Flask:
    app = Flask(__name__)
    app.config["ENV_NAME"] = config_name

    @app.route("/")
    def index() -> str:
        return "OK"

    @app.route("/users/<int:user_id>", methods=["GET", "PUT"])
    def get_user(user_id: int) -> str:
        return f"user {user_id}"

    # Comandos vinculados a la app: ya disponen de contexto de aplicación,
    # por lo que @with_appcontext no es necesario aquí.
    @app.cli.command("init-db")
    def init_db() -> None:
        """Inicializa el esquema de base de datos."""
        click.echo(f"Inicializando DB para entorno {app.config['ENV_NAME']}")

    return app


# --------------------------------------------------------------------------
# 2. Grupo de comandos con contexto automático (AppGroup.group / command)
# --------------------------------------------------------------------------
cache_cli = AppGroup("cache", help="Operaciones de caché.")


@cache_cli.command("clear")  # envuelto automáticamente en with_appcontext
def cache_clear() -> None:
    """Vacía la caché de la aplicación."""
    from flask import current_app

    click.echo(f"Caché limpiada en {current_app.import_name}")


@cache_cli.command("stats", with_appcontext=False)
def cache_stats() -> None:
    """No necesita la aplicación cargada."""
    click.echo("stats: 0 entradas")


# --------------------------------------------------------------------------
# 3. Comando suelto decorado explícitamente con with_appcontext()
# --------------------------------------------------------------------------
@click.command("whoami")
@with_appcontext
def whoami() -> None:
    from flask import current_app

    click.echo(f"App activa: {current_app.name}")


# --------------------------------------------------------------------------
# 4. Punto de entrada: FlaskGroup con la factoría inyectada
# --------------------------------------------------------------------------
cli = FlaskGroup(
    create_app=create_app,          # evita la necesidad de --app / FLASK_APP
    add_default_commands=True,      # run, shell, routes
    add_version_option=True,        # --version -> get_version()
    load_dotenv=True,               # habilita .env / .flaskenv
    set_debug_flag=True,
    help="Utilidad de gestión del proyecto.",
)
cli.add_command(cache_cli)
cli.add_command(whoami)


# --------------------------------------------------------------------------
# 5. Uso programático de las utilidades de descubrimiento
# --------------------------------------------------------------------------
def resolve_app_programmatically() -> Flask | None:
    """Localiza una app sin pasar por la CLI."""
    # load_dotenv() -> True si se exportó al menos una variable de entorno.
    load_dotenv(path=".env.local", load_defaults=True)

    # prepare_import() normaliza la ruta y ajusta sys.path.
    module_name = prepare_import("src/myproject/wsgi.py")

    # locate_app() con raise_if_not_found=False devuelve None si no existe.
    app = locate_app(module_name, "create_app('testing')", raise_if_not_found=False)
    if app is None:
        click.echo("Módulo no encontrado; usando la factoría local.")
        return None
    return app


def resolve_with_script_info() -> Flask:
    """Carga diferida y memoizada mediante ScriptInfo.load_app()."""
    info = ScriptInfo(create_app=lambda: create_app("staging"), set_debug_flag=False)
    try:
        return info.load_app()  # llamadas sucesivas devuelven la misma instancia
    except NoAppException as exc:
        raise SystemExit(exc.format_message()) from exc


if __name__ == "__main__":
    cli()
```

**Invocaciones equivalentes desde la terminal:**

```bash
# Comandos integrados mediante el objeto `cli` del módulo flask.cli (main()).
flask --app "manage:create_app('development')" --debug run --host 0.0.0.0 --port 8000
flask --app manage routes --sort match --all-methods
flask --app manage shell
flask --env-file .env.ci --app manage run --cert adhoc
flask --version

# Comandos del script personalizado (FlaskGroup definido arriba).
python manage.py cache clear
python manage.py whoami
python manage.py init-db          # comando aportado por app.cli
python manage.py run --no-reload --without-threads
```