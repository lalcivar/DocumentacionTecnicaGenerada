# Documentación Técnica - `flask/cli.py`

## 1. Descripción General

**Archivo:** `cli.py`  
**Framework:** Flask 3.x  
**Propósito:** Implementar la interfaz de línea de comandos (CLI) de Flask basada en `Click`.

Este archivo es el núcleo del sistema de comandos `flask`. Se encarga de:

1.  **Localizar y cargar la aplicación Flask** a partir de un import string, una ruta de archivo o por convención.
2.  **Gestionar el contexto de la aplicación** para la ejecución de comandos.
3.  **Definir el grupo principal de comandos `FlaskGroup`** y registrar los comandos por defecto: `run`, `shell` y `routes`.
4.  **Gestionar variables de entorno y archivos `.env` / `.flaskenv`** mediante `python-dotenv`.
5.  **Proveer el servidor de desarrollo** con soporte para reloader, debugger, SSL y multithreading.

Es el punto de entrada que se ejecuta al invocar `flask --app app run` en la terminal.

### Dependencias Principales
*   `click`: Creación de la CLI, grupos, comandos y opciones.
*   `werkzeug`: Servidor de desarrollo `run_simple`, utilidades `import_string` y `is_running_from_reloader`.
*   `importlib.metadata`: Obtención de versiones de Flask/Werkzeug.
*   `python-dotenv` (opcional): Carga de variables de entorno.

---

## 2. Componentes Principales

### 2.1. Manejo de Excepciones

| Componente | Descripción |
| :--- | :--- |
| `NoAppException(click.UsageError)` | Excepción personalizada que se lanza cuando no se puede encontrar, importar o cargar una aplicación Flask. Hereda de `click.UsageError` para mostrar un error amigable en la CLI. |

### 2.2. Localización y Carga de la Aplicación

Este es el subsistema más crítico. Permite la flexibilidad de `flask --app`.

| Función | Descripción |
| :--- | :--- |
| `find_best_app(module: ModuleType) -> Flask` | Intenta encontrar la mejor aplicación dentro de un módulo ya importado. Sigue una heurística en orden: 1. Busca atributos `app` o `application`. 2. Si hay una única instancia de `Flask` en el módulo, la retorna. 3. Busca factories `create_app` o `make_app` y las intenta ejecutar sin argumentos. Si encuentra múltiples apps, lanza `NoAppException`. |
| `find_app_by_string(module, app_name) -> Flask` | Resuelve un string del tipo `nombre` o `factory(arg, kwarg='val')`. Usa `ast.parse` para validar la sintaxis de forma segura y `ast.literal_eval` para evaluar solo argumentos literales. Si el atributo es una función, la invoca con los argumentos parseados. |
| `prepare_import(path: str) -> str` | Convierte una ruta de archivo (ej. `src/app.py` o `src/app/__init__.py`) en un nombre de módulo importable (ej. `app`). Calcula el `sys.path` correcto subiendo directorios hasta que no encuentra un `__init__.py` y lo inserta en `sys.path[0]`. |
| `locate_app(module_name, app_name, raise_if_not_found) -> Flask | None` | Orquestador principal. Hace `__import__(module_name)`, maneja `ImportError` diferenciando si el error es del módulo buscado o de una dependencia interna, y delega a `find_best_app` o `find_app_by_string` según si `app_name` es `None`. Tiene sobrecargas con `typing.overload`. |
| `_called_with_wrong_args(f) -> bool` | Función auxiliar que inspecciona el `traceback` para determinar si un `TypeError` al llamar a una factory se debe a argumentos incorrectos en la llamada o a un error interno de la propia factory. |

### 2.3. Gestión de Contexto y Estado

| Componente | Descripción |
| :--- | :--- |
| `class ScriptInfo` | Objeto contenedor que mantiene el estado de la CLI. Almacena `app_import_path`, `create_app` (callback), `data` (dict arbitrario), `set_debug_flag` y `load_dotenv_defaults`. Su método clave es `load_app() -> Flask`, que implementa la lógica de carga lazy y cacheada (`_loaded_app`). Busca la app en: 1. `create_app` callback, 2. `app_import_path` especificado, 3. Archivos por convención `wsgi.py` / `app.py`. También sincroniza el flag `app.debug` con `get_debug_flag()`. |
| `with_appcontext(f)` | Decorador que garantiza que la función decorada se ejecute dentro de un `app_context`. Si no hay `current_app` activo, carga la app vía `ScriptInfo` y hace `ctx.with_resource(app.app_context())`. Esencial para comandos que necesitan acceder a la configuración o base de datos. |
| `pass_script_info` | Decorador `click.make_pass_decorator(ScriptInfo, ensure=True)` para inyectar el objeto `ScriptInfo` en los comandos. |

### 2.4. Grupos de Comandos Click

| Componente | Descripción |
| :--- | :--- |
| `class AppGroup(click.Group)` | Subclase de `click.Group` que modifica el comportamiento de `@group.command()` y `@group.group()` para que todos los comandos se envuelvan automáticamente con `@with_appcontext` (desactivable con `with_appcontext=False`). |
| `class FlaskGroup(AppGroup)` | Grupo principal de la CLI `flask`. Es el corazón de la extensibilidad. **Responsabilidades:** 1. Define las opciones globales eager: `-A/--app`, `--debug/--no-debug`, `-e/--env-file` y `--version`. 2. Carga comandos de plugins vía `importlib.metadata.entry_points(group="flask.commands")`. 3. Sobrescribe `get_command()` y `list_commands()` para combinar comandos built-in, de plugins y los registrados en `app.cli`. 4. Sobrescribe `make_context()` para inyectar `ScriptInfo` y setear `FLASK_RUN_FROM_CLI=true`. 5. Maneja `no_args_is_help` cargando anticipadamente `--app` y `--env-file`. |

### 2.5. Opciones Globales y Callbacks

| Opción / Callback | Descripción |
| :--- | :--- |
| `_app_option (-A, --app)` | Opción eager que guarda el valor en `ScriptInfo.app_import_path` vía callback `_set_app`. |
| `_debug_option (--debug/--no-debug)` | Modifica `os.environ["FLASK_DEBUG"]` directamente para que esté disponible incluso durante la ejecución de una factory. Callback `_set_debug`. |
| `_env_file_option (-e, --env-file)` | Opción eager que carga variables de entorno. Callback `_env_file_callback` que verifica `python-dotenv` e invoca a `load_dotenv()`. |
| `version_option (--version)` | Muestra `Python`, `Flask` y `Werkzeug` y hace `ctx.exit()`. Callback `get_version`. |

### 2.6. Comandos por Defecto

| Comando | Descripción |
| :--- | :--- |
| `run_command` (`flask run`) | Inicia el servidor de desarrollo `werkzeug.run_simple`. Opciones: `--host`, `--port`, `--cert`/`--key` (HTTPS), `--reload/--no-reload`, `--debugger/--no-debugger`, `--with-threads`, `--extra-files`, `--exclude-patterns`. Maneja el caso de error durante el reload creando una WSGI app dummy que relanza la excepción. |
| `shell_command` (`flask shell`) | Inicia un shell interactivo de Python dentro del contexto de la aplicación. Carga `PYTHONSTARTUP`, configura `readline` con autocompletado del contexto de `app.make_shell_context()` y usa `code.interact`. Decorado con `@with_appcontext`. |
| `routes_command` (`flask routes`) | Muestra la tabla de rutas registradas en `current_app.url_map`. Opciones: `--sort` (endpoint, methods, domain, rule, match) y `--all-methods`. Calcula anchos de columna dinámicamente y formatea la salida. |

### 2.7. Utilidades

| Componente | Descripción |
| :--- | :--- |
| `load_dotenv(path, load_defaults) -> bool` | Carga archivos dotenv. Prioridad: `path` explícito > `.env` > `.flaskenv`. Solo setea variables que no existen en `os.environ`. Es no-op si `python-dotenv` no está instalado. |
| `show_server_banner(debug, app_import_path)` | Muestra el banner inicial `* Serving Flask app...` y `* Debug mode...` solo si no se está ejecutando desde el reloader. |
| `CertParamType(click.ParamType)` | Tipo para `--cert`. Acepta una ruta a archivo, el string `adhoc` (requiere `cryptography`) o un import string a un objeto `ssl.SSLContext`. |
| `SeparatedPathType(click.Path)` | Tipo que acepta múltiples rutas separadas por `os.pathsep` (`:` en Unix, `;` en Windows) y las valida individualmente. Usado en `--extra-files`. |
| `_validate_key` | Callback que valida la coherencia entre `--cert` y `--key`. |
| `cli = FlaskGroup(...)` | Instancia global del grupo principal. Es el entry point registrado como `flask` en `pyproject.toml`. |
| `main()` | Función que ejecuta `cli.main()` si el archivo se invoca como `__main__`. |

---

## 3. Flujo de Ejecución General

1.  El usuario ejecuta `flask --app myproject:create_app('prod') run --port 8000`.
2.  `FlaskGroup.make_context()` crea un `ScriptInfo` y setea `FLASK_RUN_FROM_CLI=true`.
3.  Los callbacks eager se ejecutan en orden: `_env_file_callback` carga `.env`, `_set_app` guarda `myproject:create_app('prod')` en `ScriptInfo`.
4.  Click resuelve el comando `run` vía `FlaskGroup.get_command()`, que intenta `info.load_app()`.
5.  `ScriptInfo.load_app()` llama a `prepare_import("myproject")` y luego a `locate_app("myproject", "create_app('prod')")`.
6.  `locate_app` importa `myproject` y delega a `find_app_by_string`, que parsea el AST, obtiene `create_app` y lo llama con `app = create_app('prod')`.
7.  Se establece `app.debug` según `FLASK_DEBUG` y se cachea la app.
8.  `FlaskGroup.get_command` hace push del `app_context` para que esté disponible en los callbacks de parámetros.
9.  Se ejecuta `run_command`, que obtiene la app ya cargada y llama a `werkzeug.run_simple()` con los parámetros de host, puerto y reloader.

