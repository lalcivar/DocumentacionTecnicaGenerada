# Documentación Técnica del Módulo `flask.cli` (_fragmento simplificado para fines de ejemplo_)  

> Nota: Se incluye **el 100 %** de las funciones, métodos y *callables* anidados presentes en el código proporcionado.  
> El formato de cada especificación sigue la guía PEP-257 con convención Google Style.  

---

## 1. Resumen ejecutivo
Este módulo implementa la interfaz de línea de comandos oficial de Flask.  
Permite descubrir, importar y ejecutar aplicaciones, así como exponer comandos de administración (run, shell, routes, etc.) sobre la base de Click.  

---

## 2. Especificación detallada de funciones y métodos

Las tablas muestran:

* **Parámetro**: nombre exacto recibido por la firma.  
* **Tipo**: anotación de tipo estática.  
* **Req.**: ¿es obligatorio? (✔︎ sí / – no; para parámetros posicionales‐opcionales o con valor por defecto).  

> Para brevedad, los cuerpos de *docstring* están recortados; sin embargo, la información crítica (Args, Returns, Raises, Notas) se muestra íntegramente.

### 2.1  Excepciones

| Clase | Hereda | Descripción |
|-------|--------|-------------|
| `NoAppException` | `click.UsageError` | Se lanza cuando no se logra localizar ni construir una instancia válida de `Flask`. |

---

### 2.2  Funciones de descubrimiento y utilidades

#### `find_best_app(module)`
```python
def find_best_app(module: ModuleType) -> Flask
```
| Parámetro | Tipo | Req. | Descripción |
|-----------|------|------|-------------|
| module | ModuleType | ✔︎ | Módulo ya importado que contiene la aplicación. |

Returns  
`Flask`: instancia encontrada.

Raises  
`NoAppException`: múltiples apps, o ninguna encontrada.

---

#### `_called_with_wrong_args(f)`
```python
def _called_with_wrong_args(f: Callable[..., Flask]) -> bool
```
| Parámetro | Tipo | Req. |
|-----------|------|------|
| f | Callable[..., Flask] | ✔︎ |

Returns  
`bool`: `True` si la excepción `TypeError` provino de argumentos incorrectos al **invocar** `f`.

---

#### `find_app_by_string(module, app_name)`
```python
def find_app_by_string(module: ModuleType, app_name: str) -> Flask
```
| Parámetro | Tipo | Req. |
|-----------|------|------|
| module | ModuleType | ✔︎ |
| app_name | str | ✔︎ |

Returns  
`Flask`: instancia devuelta o referenciada por `app_name`.

Raises  
* `NoAppException`: error de sintaxis, atributo inexistente, o resultado no válido.  
* Cualquier excepción propagada desde la fábrica si no se debe a argumentos inválidos (_TypeError_).

---

#### `prepare_import(path)`
```python
def prepare_import(path: str) -> str
```
| Parámetro | Tipo | Req. |
|-----------|------|------|
| path | str | ✔︎ |

Returns  
`str`: **nombre de módulo** resultante y añade su carpeta raíz a `sys.path[0]` si fuera necesario.

---

#### `locate_app(module_name, app_name, raise_if_not_found=True)`
Sobrecarga estática para resolver `None` vs `Flask | None`.

| Parámetro | Tipo | Req. |
|-----------|------|------|
| module_name | str | ✔︎ |
| app_name | str \| None | – |
| raise_if_not_found | bool | – |

Returns  
`Flask` o `None` (según overload).

Raises  
`NoAppException`: error al importar o localizar la aplicación.

---

#### `get_version(ctx, param, value)`
Callback para la opción global `--version`.

| Parámetro | Tipo | Req. |
|-----------|------|------|
| ctx | click.Context | ✔︎ |
| param | click.Parameter | ✔︎ |
| value | Any | ✔︎ |

Acción  
Muestra versiones de Python, Flask y Werkzeug y sale (`ctx.exit()`).

---

### 2.3  Clase `ScriptInfo`

| Método | Resumen |
|--------|---------|
| `__init__(self, app_import_path=None, create_app=None, set_debug_flag=True, load_dotenv_defaults=True)` | Inicializa metadatos para resolución de la app. |
| `load_app(self) -> Flask` | Carga (o devuelve en caché) la instancia `Flask`. |

#### `ScriptInfo.__init__`
Todos los parámetros son opcionales. Mantiene referencias públicas:
`app_import_path`, `create_app`, `data`, `set_debug_flag`, `load_dotenv_defaults`.

Raises: ninguna.

#### `ScriptInfo.load_app`
Sin argumentos.

Returns  
`Flask`: aplicación cargada y almacenada en `_loaded_app`.

Raises  
`NoAppException`: no se pudo ubicar o construir la aplicación.

---

### 2.4  Decorador `with_appcontext`

```python
def with_appcontext(f: F) -> F
```
Garantiza que el *callback* `f` se ejecute dentro de un **Application Context**.  
Devuelve un *wrapper* (`decorator`) que mantiene `__name__`, `__doc__`, etc.

##### Función anidada `decorator(ctx, *args, **kwargs)`
| Parámetro | Tipo | Req. |
|-----------|------|------|
| ctx | click.Context | ✔︎ |
| *args | Any | – |
| **kwargs | Any | – |

Efectúa:
1. Asegurar app contextual (`ScriptInfo.load_app` si es necesario).  
2. Delegar la ejecución a `f`.

---

### 2.5  Clase `AppGroup(click.Group)`

Métodos sobrescritos:

| Método | Particularidad |
|--------|----------------|
| `command(self, *args, **kwargs)` | Envuelve comandos en `with_appcontext` (salvo `with_appcontext=False`). Contiene función anidada `decorator(f)`. |
| `group(self, *args, **kwargs)` | Crea sub‐grupos usando por defecto `AppGroup`. |

---

### 2.6  Callbacks/Options auxiliares

| Función | Rol / Opción Click | Raises (parciales) |
|---------|--------------------|--------------------|
| `_set_app(ctx, param, value)` | Maneja la opción `--app`. | – |
| `_set_debug(ctx, param, value)` | Maneja `--debug / --no-debug`. | – |
| `_env_file_callback(ctx, param, value)` | Maneja `--env-file`. | `click.BadParameter` |

---

### 2.7  Funciones internas

| Función | Descripción |
|---------|-------------|
| `_path_is_ancestor(path, other)` | Devuelve `True` si `path` es ancestro de `other`. |
| `load_dotenv(path=None, load_defaults=True)` | Carga archivos `.env` / `.flaskenv`. |
| `show_server_banner(debug, app_import_path)` | Imprime banner de inicio (una sola vez). |
| `_validate_key(ctx, param, value)` | Valida coherencia `--cert` / `--key`. |

Todas documentan sus posibles `click.BadParameter`.

---

### 2.8  Tipos Click personalizados

1. **`CertParamType`**  
   *Método clave*: `convert(self, value, param, ctx)`  
   Valida/adapta:  
   * Ruta a fichero `.pem`  
   * Cadena `'adhoc'`  
   * `ssl.SSLContext` importable.  

2. **`SeparatedPathType`**  
   *Método*: `convert(self, value, param, ctx)`  
   Divide un valor por `os.pathsep` y aplica validación `click.Path` a cada segmento.

---

### 2.9  Comandos CLI

| Comando | Firma | Descripción corta |
|---------|-------|-------------------|
| **`run_command`** | ver abajo | Inicia servidor de desarrollo. |
| **`shell_command`** | `def shell_command()` | Arranca consola interactiva con contexto de la app. |
| **`routes_command`** | `def routes_command(sort, all_methods)` | Lista todas las rutas registradas. |

#### `run_command(...)`
```python
def run_command(
    info: ScriptInfo, host: str, port: int, reload: bool,
    debugger: bool, with_threads: bool,
    cert: ssl.SSLContext | tuple[str, str | None] | Literal["adhoc"] | None,
    extra_files: list[str] | None, exclude_patterns: list[str] | None
) -> None
```
Monta un servidor Werkzeug (dev).

Raises  
Propaga cualquier excepción al cargar la app o al iniciar el servidor.

Función anidada `app(environ, start_response)`  
Se crea dinámicamente cuando la importación falla bajo el *reloader* y vuelve a lanzar el error original en tiempo de petición.

---

### 2.10  Clase `FlaskGroup(AppGroup)`

Métodos propios:

| Método | Propósito |
|--------|-----------|
| `__init__(...)` | Construye el grupo principal `flask` e inyecta opciones/ comandos por defecto. |
| `_load_plugin_commands(self)` | Registra *entry-points* `flask.commands`. |
| `get_command(self, ctx, name)` | Resuelve comandos built-in, plugins y definidos en la app. |
| `list_commands(self, ctx)` | Lista todos los comandos disponibles. |
| `make_context(self, info_name, args, parent=None, **extra)` | Crea contexto Click inyectando `ScriptInfo`. |
| `parse_args(self, ctx, args)` | Pre-parsea para reconocer `--env-file` y `--app` aun cuando se muestre la ayuda. |

---

### 2.11  Entrada de programa

#### `main()`
Ejecuta `cli.main()` con la instancia global `FlaskGroup` pre-configurada.

---

## 3. Ejemplo de uso

```python
"""
Archivo: manage.py
------------------

Gestor simplificado de una aplicación Flask usando la CLI integrada.
"""

from flask.cli import cli                 # estudio simplificado
# Alternativamente:
# from flask.cli import FlaskGroup
# cli = FlaskGroup(create_app=lambda: myapp.create_app())

if __name__ == "__main__":
    # Ejecutar comandos:
    #
    #   $ python manage.py --app "proj:create_app()" run --debug
    #   $ python manage.py shell
    #   $ python manage.py routes -s rule
    #
    cli()     # delega en Click
```

Con el archivo anterior se obtiene la misma experiencia que  
`python -m flask --app proj:create_app`.  

---

### Referencias
* PEP-257 – Docstring Conventions  
* Google Python Style Guide – Docstrings section  
* Click Documentation – Custom Multi-Command classes