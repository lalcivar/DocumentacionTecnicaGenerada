# Documentación técnica: `flask/cli.py`

## 1. Descripción general

Este módulo implementa **la interfaz de línea de comandos (CLI) de Flask**, es decir, toda la maquinaria que da soporte al comando `flask` (`flask run`, `flask shell`, `flask routes`, etc.).

Sus responsabilidades principales son:

1. **Descubrimiento y carga de la aplicación Flask**: localizar una instancia `Flask` o una *application factory* a partir de una cadena de importación (`módulo:nombre`), una ruta de archivo, o por convención (`app.py`, `wsgi.py`, `app`, `application`, `create_app`, `make_app`).
2. **Integración con `click`**: define grupos, tipos de parámetros, decoradores y opciones personalizadas que extienden `click` para que los comandos se ejecuten dentro de un *application context* de Flask.
3. **Gestión del entorno de ejecución**: carga de variables de entorno desde archivos `.env` / `.flaskenv` (vía `python-dotenv`), activación del modo *debug*, y prefijo automático de variables de entorno `FLASK_*`.
4. **Comandos integrados**: `run` (servidor de desarrollo basado en Werkzeug), `shell` (REPL con contexto de aplicación) y `routes` (listado de rutas registradas).
5. **Extensibilidad**: carga de comandos de terceros a través de *entry points* del grupo `flask.commands`, y de comandos definidos por la aplicación o sus *blueprints* (`app.cli`, `blueprint.cli`).

El archivo es también el **punto de entrada ejecutable** del paquete (`main()` / `if __name__ == "__main__"`).

---

## 2. Dependencias relevantes

| Dependencia | Uso |
|---|---|
| `click` | Base de toda la CLI (comandos, grupos, opciones, tipos de parámetros). |
| `werkzeug.run_simple`, `werkzeug.serving.is_running_from_reloader` | Servidor de desarrollo y detección del recargador. |
| `werkzeug.utils.import_string` | Importación dinámica (p. ej. de un `ssl.SSLContext`). |
| `ast`, `inspect`, `importlib.metadata` | Parseo de expresiones de factoría, introspección y lectura de versiones/entry points. |
| `dotenv` (opcional) | Carga de archivos `.env` / `.flaskenv`. |
| `.globals.current_app`, `.helpers.get_debug_flag`, `.helpers.get_load_dotenv` | Integración con el núcleo de Flask. |

---

## 3. Componentes principales

### 3.1 Excepciones

#### `NoAppException(click.UsageError)`
Excepción lanzada cuando no se puede encontrar o cargar la aplicación. Al derivar de `click.UsageError`, se muestra como error de uso amigable (sin *traceback*).

---

### 3.2 Funciones de localización de la aplicación

#### `find_best_app(module) -> Flask`
Dado un módulo ya importado, intenta deducir la aplicación:

1. Busca los atributos `app` y `application`.
2. Si no, busca una **única** instancia de `Flask` en el namespace del módulo (si hay varias, lanza `NoAppException` pidiendo desambiguar).
3. Si no, busca las factorías `create_app` / `make_app` y las invoca sin argumentos.
4. Si todo falla, lanza `NoAppException`.

#### `_called_with_wrong_args(f) -> bool` *(privada)*
Analiza el *traceback* actual para distinguir si un `TypeError` se produjo **al llamar** a la factoría (firma incompatible) o **dentro** de ella. Devuelve `True` en el primer caso. Elimina `tb` en el `finally` para evitar referencias circulares.

#### `find_app_by_string(module, app_name) -> Flask`
Interpreta la parte `name` de `módulo:name` usando `ast.parse(..., mode="eval")`:

- `ast.Name` → se devuelve el atributo directamente.
- `ast.Call` → se exige que la función sea un nombre simple; los argumentos posicionales y de palabra clave se evalúan con `ast.literal_eval` (solo literales, por seguridad) y se invoca la factoría.
- Cualquier otra expresión, error de sintaxis o resultado que no sea una instancia de `Flask` produce `NoAppException`.

#### `prepare_import(path) -> str`
Convierte una **ruta de archivo** en un **nombre de módulo importable**:
- Normaliza la ruta y elimina la extensión `.py`.
- Si es un `__init__.py`, sube al directorio del paquete.
- Asciende por el árbol de directorios mientras existan `__init__.py`, acumulando los segmentos del paquete.
- Inserta el directorio raíz resultante en `sys.path[0]` y devuelve el nombre punteado del módulo.

#### `locate_app(module_name, app_name, raise_if_not_found=True) -> Flask | None`
Importa el módulo y delega en `find_best_app` o `find_app_by_string`. Distingue, inspeccionando la profundidad del *traceback*, entre:
- un `ImportError` del propio módulo solicitado (opcionalmente devuelve `None` si `raise_if_not_found=False`), y
- un `ImportError` producido **dentro** del módulo, que se reporta con el *traceback* completo.

Cuenta con dos `@t.overload` para precisar el tipo de retorno según `raise_if_not_found`.

---

### 3.3 Soporte de versión

#### `get_version(ctx, param, value)` y `version_option`
Callback y `click.Option` *eager* para `--version`; imprime las versiones de Python, Flask y Werkzeug obtenidas con `importlib.metadata.version` y termina la ejecución.

---

### 3.4 `ScriptInfo`

Objeto contenedor que encapsula **cómo obtener la aplicación** y que se propaga como `ctx.obj` de click.

**Constructor:** `ScriptInfo(app_import_path=None, create_app=None, set_debug_flag=True, load_dotenv_defaults=True)`

**Atributos destacados**
| Atributo | Descripción |
|---|---|
| `app_import_path` | Ruta de importación (`módulo:nombre`) de la app. |
| `create_app` | Callback alternativo que construye la app. |
| `data` | Diccionario libre para datos arbitrarios asociados al script. |
| `set_debug_flag` | Si se debe fijar `app.debug` según el entorno. |
| `load_dotenv_defaults` | Indica (solo informativamente) si deben cargarse `.flaskenv`/`.env`. Añadido en 3.1. |
| `_loaded_app` | Caché de la app ya cargada. |

**Método `load_app() -> Flask`**
Carga la app **una sola vez** (memoizada):
1. Usa `create_app` si está definido.
2. Si hay `app_import_path`, lo divide con `re.split(r":(?![\\/])", ..., maxsplit=1)` (evita partir rutas tipo `C:\...`), prepara la importación y llama a `locate_app`.
3. Si no, prueba por convención `wsgi.py` y `app.py`.
4. Si no encuentra nada, lanza `NoAppException` con un mensaje explicativo.
5. Si `set_debug_flag`, asigna `app.debug = get_debug_flag()`.

#### `pass_script_info`
Decorador generado con `click.make_pass_decorator(ScriptInfo, ensure=True)` que inyecta el `ScriptInfo` en el callback del comando.

---

### 3.5 Contexto de aplicación en comandos

#### `with_appcontext(f)`
Decorador que garantiza que el callback (y sus subcomandos) se ejecuten con un *application context* activo. Si no hay `current_app`, carga la app desde el `ScriptInfo` y registra el contexto como recurso del `click.Context` (`ctx.with_resource`), de modo que permanezca activo durante todo el comando.

> No es necesario para comandos registrados en `app.cli` / `blueprint.cli`, que ya reciben contexto desde la versión 2.2.

---

### 3.6 Grupos de comandos

#### `AppGroup(click.Group)`
Grupo de click cuyo decorador `command()` envuelve automáticamente los callbacks con `with_appcontext` (desactivable con `with_appcontext=False`), y cuyo `group()` usa `AppGroup` como clase por defecto.

#### `FlaskGroup(AppGroup)`
Grupo raíz de la CLI de Flask. Añade:

**Parámetros del constructor**
- `add_default_commands`: registra `run`, `shell` y `routes`.
- `create_app`: factoría personalizada.
- `add_version_option`: añade `--version`.
- `load_dotenv`: habilita la carga de `.env`/`.flaskenv`.
- `set_debug_flag`: propaga el flag de debug a la app.

Configura `auto_envvar_prefix="FLASK"` en `context_settings` y añade, en orden, las opciones `--env-file`, `--app` y `--debug/--no-debug` (el orden importa porque son *eager*).

**Métodos sobreescritos**
| Método | Comportamiento |
|---|---|
| `_load_plugin_commands()` | Carga una sola vez los comandos publicados en el *entry point* `flask.commands`. |
| `get_command(ctx, name)` | Busca primero en comandos integrados/plugins; si no existe, carga la app, empuja su contexto si hace falta y delega en `app.cli.get_command`. Si la app no carga, muestra el error en rojo y devuelve `None`. |
| `list_commands(ctx)` | Une comandos integrados/plugins con los de `app.cli`, tolerando fallos de carga (mensaje corto para `NoAppException`, *traceback* completo para otros errores). |
| `make_context(...)` | Fija `FLASK_RUN_FROM_CLI=true` (para que `app.run()` sea un *no-op* al importar) y crea el `ScriptInfo` como `obj` si no se proporcionó. |
| `parse_args(ctx, args)` | Cuando no hay argumentos o solo se pide `--help`, procesa manualmente `--env-file` y `--app` para que se vean los comandos de la app pese a `no_args_is_help`. |

---

### 3.7 Opciones globales (privadas)

| Objeto | Callback | Propósito |
|---|---|---|
| `_app_option` (`-A/--app`) | `_set_app` | Asigna `ScriptInfo.app_import_path`. Es *eager* para que la app esté disponible incluso con `--help`. |
| `_debug_option` (`--debug/--no-debug`) | `_set_debug` | Escribe `FLASK_DEBUG` en `os.environ` (para que esté disponible pronto, incluso dentro de una factoría). Si el valor viene del *default*, devuelve `None` y deja decidir al entorno (usa `ctx.get_parameter_source`). |
| `_env_file_option` (`-e/--env-file`) | `_env_file_callback` | Carga variables de entorno desde un archivo; exige `python-dotenv` si se pasa un valor explícito. Es la opción más *eager*. |

---

### 3.8 Utilidades de entorno y arranque

#### `_path_is_ancestor(path, other) -> bool`
Comprueba si `path` es ancestro de `other` mediante reconstrucción de la ruta.

#### `load_dotenv(path=None, load_defaults=True) -> bool`
Carga variables de entorno con el siguiente orden de precedencia (de mayor a menor): **`os.environ` existente** → **archivo indicado en `path`** → **`.env`** → **`.flaskenv`**. Características:
- *No-op* (devuelve `False`) si `python-dotenv` no está instalado, mostrando un aviso si existen archivos `.env`/`.flaskenv`.
- Lee los archivos con codificación UTF-8 y **no** cambia el directorio de trabajo.
- Nunca sobrescribe variables ya presentes en `os.environ`.
- Devuelve `True` si se cargó al menos una variable.

#### `show_server_banner(debug, app_import_path)`
Imprime los mensajes iniciales (`* Serving Flask app ...`, `* Debug mode: ...`) una sola vez, omitiéndolos en los procesos hijos del recargador.

---

### 3.9 Tipos de parámetros personalizados

#### `CertParamType(click.ParamType)`
Tipo para `--cert`. Acepta:
- la ruta de un archivo de certificado existente,
- la cadena `"adhoc"` (requiere `cryptography`),
- una cadena de importación que resuelva a un `ssl.SSLContext`.

Lanza `click.BadParameter` si Python no tiene soporte SSL o si faltan dependencias.

#### `_validate_key(ctx, param, value)`
Callback de `--key` que valida la coherencia con `--cert`:
- `--key` no se admite con `adhoc` ni con un `SSLContext`.
- `--key` requiere `--cert` y viceversa (cuando `--cert` es un archivo).
- Si ambos son válidos, transforma `ctx.params["cert"]` en la tupla `(cert, key)`.

#### `SeparatedPathType(click.Path)`
Tipo que acepta varias rutas separadas por el separador de rutas del sistema (`:` en POSIX, `;` en Windows) y valida cada una como `click.Path`.

---

### 3.10 Comandos integrados

#### `run_command` → `flask run`
Servidor de desarrollo basado en `werkzeug.run_simple`.

**Opciones:** `--host/-h` (def. `127.0.0.1`), `--port/-p` (def. `5000`), `--cert`, `--key`, `--reload/--no-reload`, `--debugger/--no-debugger`, `--with-threads/--without-threads`, `--extra-files`, `--exclude-patterns`, más `--debug` insertado explícitamente como primer parámetro.

**Lógica clave:**
- Si la carga de la app falla **durante una recarga**, imprime el *traceback* y sustituye la app por una aplicación WSGI que re-lanza el error en cada petición (así el recargador/depurador sigue vivo); si no hay recarga, el comando falla inmediatamente.
- `reload` y `debugger` heredan el valor de `get_debug_flag()` si no se especifican.
- Muestra el banner y arranca `run_simple` con el contexto SSL, hilos y listas de archivos configurados.

> Advertencia documentada: servidor **solo para desarrollo**, no apto para producción.

#### `shell_command` → `flask shell`
Abre un REPL interactivo (`code.interact`) con contexto de aplicación:
- Banner con versión de Python, `import_name` e `instance_path` de la app.
- Ejecuta el script `PYTHONSTARTUP` si existe.
- Pobla el namespace con `current_app.make_shell_context()`.
- Configura `readline` + `rlcompleter` sobre el namespace del shell (en lugar de `__main__`) e invoca `sys.__interactivehook__`.

#### `routes_command` → `flask routes`
Lista las reglas de `current_app.url_map` en forma de tabla alineada.
- `--sort/-s` entre `endpoint`, `methods`, `domain`, `rule`, `match` (donde `match` conserva el orden real de resolución de Flask).
- `--all-methods` incluye `HEAD` y `OPTIONS` (ocultos por defecto).
- Añade la columna `Host` o `Subdomain` solo si alguna regla la usa (según `url_map.host_matching`).
- Calcula los anchos de columna dinámicamente y emite separadores.

---

### 3.11 Punto de entrada

```python
cli = FlaskGroup(name="flask", help="...")

def main() -> None:
    cli.main()

if __name__ == "__main__":
    main()
```

- `cli`: instancia global de `FlaskGroup` usada como *console script* `flask`.
- `main()`: función de arranque, también utilizada al ejecutar `python -m flask`.

---

## 4. Notas de diseño y detalles de implementación

- **Orden de evaluación de opciones**: `--env-file` → `--app` → `--debug` → `--version`/`--help`. Todas son `is_eager` y `expose_value=False`, de forma que actúan como efectos secundarios sobre `ScriptInfo`/`os.environ` antes de resolver subcomandos.
- **Procesamiento vía callbacks de opción** (en lugar de un callback de grupo) para que los usuarios puedan definir su propio callback de grupo sin perder el comportamiento.
- **Seguridad en el parseo de factorías**: solo se admiten literales (`ast.literal_eval`), nunca ejecución arbitraria de expresiones.
- **Tolerancia a fallos**: los comandos integrados y de plugins siguen estando disponibles aunque la aplicación no pueda cargarse.
- **Tipado**: uso extensivo de anotaciones, `from __future__ import annotations`, bloque `if t.TYPE_CHECKING` (para `ssl`, tipos WSGI de `_typeshed` y `Flask`) y `@t.overload` en `locate_app`.
- **Variables de entorno relevantes**: `FLASK_APP`, `FLASK_DEBUG`, `FLASK_RUN_FROM_CLI`, `PYTHONSTARTUP`, además de cualquier `FLASK_*` mapeada automáticamente a opciones por `auto_envvar_prefix`.