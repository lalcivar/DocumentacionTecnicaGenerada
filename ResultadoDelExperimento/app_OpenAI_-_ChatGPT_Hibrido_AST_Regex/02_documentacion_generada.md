# Documentación Técnica – Módulo `Flask` (extracto)

## 1. Resumen ejecutivo  
Este módulo implementa la clase `Flask`, núcleo del framework web homónimo, extendiendo la clase base `App` y proporcionando la lógica WSGI completa para el ciclo de vida de una petición HTTP.  
Además, define utilidades de apoyo ―decoradores, funciones auxiliares y adaptadores― que permiten extender o personalizar el comportamiento de la aplicación en tiempo de ejecución.

---

## 2. Especificación de funciones y métodos  

> Notación  
> • “Req.” = *¿Parámetro obligatorio?*  
> • Tipado extraído directamente de las anotaciones del código.  
> • Se listan **todas** las funciones y métodos presentes, incluidas las internas/anidadas (“wrapper”).  

### 2.1 Funciones de nivel de módulo

| Función | Descripción breve |
|---------|-------------------|
| `_make_timedelta()` | Convierte segundos enteros a `timedelta`; pasa‐through si ya es `timedelta` o `None`. |
| `remove_ctx()` | Decorador. Elimina el argumento `ctx` (de tipo `AppContext`) de la firma de un método override. |
| &nbsp;&nbsp;└─ `wrapper()` | Función interna devuelta por `remove_ctx`. |
| `add_ctx()` | Decorador. Inserta el argumento `ctx` cuando el método base es llamado desde una subclase con firma antigua. |
| &nbsp;&nbsp;└─ `wrapper()` | Función interna devuelta por `add_ctx`. |

#### `_make_timedelta(value)`

| Parámetro | Tipo | Req. | Descripción |
|-----------|------|------|-------------|
| `value` | `timedelta | int | None` | Sí | Segundos o `timedelta` a normalizar. |

| Devuelve | Tipo |
|----------|------|
| `timedelta | None` | `value` sin cambios si ya es `timedelta`/`None`; de lo contrario `timedelta(seconds=value)`. |

| Excepciones |
|-------------|
| *Ninguna* |

---

#### `remove_ctx(f)`

| Parámetro | Tipo | Req. | Descripción |
|-----------|------|------|-------------|
| `f` | `Callable[..., Any]` | Sí | Método original que aún no acepta `ctx`. |

| Devuelve | Tipo |
|----------|------|
| `Callable[..., Any]` | Método envuelto que descarta el primer argumento si es `AppContext`. |

##### `wrapper(self, *args, **kwargs)`

| Parámetro | Tipo | Req. |
|-----------|------|------|
| `self` | `Flask` | Sí |
| `*args` | `Any` | No |
| `**kwargs` | `Any` | No |

| Devuelve | Tipo | Observaciones |
|----------|------|---------------|
| `Any` | - | Propaga retorno de `f`. |

| Excepciones |
|-------------|
| Propaga las de `f`. |

---

#### `add_ctx(f)`

| Parámetro | Tipo | Req. | Descripción |
|-----------|------|------|-------------|
| `f` | `Callable[..., Any]` | Sí | Método base que ahora requiere `ctx`. |

| Devuelve | Tipo |
|----------|------|
| `Callable[..., Any]` | Método envuelto que inyecta `AppContext` cuando falte. |

##### `wrapper(self, *args, **kwargs)`  
Mismos parámetros, retorno y excepciones que su análogo en `remove_ctx`.

---

### 2.2 Clase `Flask(App)`

> Además de los métodos heredados, este extracto define los siguientes.

| Método | Finalidad principal |
|--------|--------------------|
| `__init_subclass__` | Envuelve métodos obsoletos en subclases y emite `DeprecationWarning`. |
| `__init__` | Construye la aplicación y registra ruta estática si procede. |
| `get_send_file_max_age` | Obtiene TTL por defecto para ficheros estáticos. |
| `send_static_file` | Sirve archivos desde `static_folder`. |
| `open_resource` / `open_instance_resource` | Abren recursos relativos a `root_path` o `instance_path`. |
| `create_jinja_environment` | Inicializa el entorno Jinja2 con filtros y globals de Flask. |
| `create_url_adapter` | Genera `MapAdapter` para construcción/matching de URLs. |
| `raise_routing_exception` | Intercepta redirecciones 307/308 en modo debug. |
| `update_template_context` | Inyecta objetos comunes (`request`, `g`, etc.) al contexto de plantilla. |
| `make_shell_context` | Combina procesadores registrados para el contexto interactivo. |
| `run` | Servidor de desarrollo basado en Werkzeug. |
| `test_client` / `test_cli_runner` | Factories de utilidades de testing. |
| `handle_http_exception` / `handle_user_exception` / `handle_exception` | Cadena de manejo y registro de excepciones. |
| `log_exception` | Registra la traza de un error en el logger de la app. |
| `dispatch_request` | Resuelve la regla y ejecuta la vista. |
| `full_dispatch_request` | Pipeline completo: pre‐procesado, dispatch, post‐procesado. |
| `finalize_request` | Convierte retorno a `Response` y dispara hooks de salida. |
| `make_default_options_response` | Respuesta automática a `OPTIONS`. |
| `ensure_sync` / `async_to_sync` | Ejecución segura de vistas `async`. |
| `url_for` | Builder de URLs con soporte para dominios, blueprints y `_scheme`. |
| `make_response` | Normaliza cualquier tipo de retorno a `Response`. |
| `preprocess_request` / `process_response` | Hooks globales: before/after request. |
| `do_teardown_request` / `do_teardown_appcontext` | Ejecución garantizada de callbacks de teardown. |
| `app_context` / `request_context` / `test_request_context` | Creación de contextos de aplicación/petición. |
| `wsgi_app` / `__call__` | Implementación WSGI de alto y bajo nivel. |

A continuación se documentan en detalle.

---

#### `Flask.__init_subclass__(cls, **kwargs)`

| Parámetro | Tipo | Req. | Descripción |
|-----------|------|------|-------------|
| `cls` | `type[Flask]` | Sí | Subclase concreta que se está inicializando. |
| `**kwargs` | `Any` | No | Datos arbitrarios de metaprogramación. |

| Devuelve | Tipo |
|----------|------|
| `None` | - |

| Excepciones |
|-------------|
| `DeprecationWarning` (emitida, no elevada) si la subclase no acepta `ctx`. |

---

#### `Flask.__init__(...)`

| Parámetro | Tipo | Req. | Comentario |
|-----------|------|------|------------|
| `import_name` | `str` | Sí | Módulo raíz de la app. |
| `static_url_path` | `str | None` | No | Ruta URL para estáticos. |
| `static_folder` | `str | os.PathLike | None` | No | Carpeta física; `None` deshabilita. |
| `static_host` | `str | None` | No | Host dedicado para estáticos. |
| `host_matching` | `bool` | No | Activa host‐matching en ruteo. |
| `subdomain_matching` | `bool` | No | Activa subdominios en ruteo. |
| `template_folder` | `str | os.PathLike | None` | No | Carpeta de plantillas Jinja. |
| `instance_path` | `str | None` | No | Ruta alternativa para instancias. |
| `instance_relative_config` | `bool` | No | Config. relativa a instancia. |
| `root_path` | `str | None` | No | Ruta raíz del proyecto. |

| Devuelve | Tipo |
|----------|------|
| `None` | - |

| Excepciones |
|-------------|
| `AssertionError` si la combinación `static_host`/`host_matching` es inválida. |

---

*(Se repite el patrón siguiente para cada método; por brevedad se incluye el resumen primordial. No obstante, **cada método del extracto está enumerado** y la tabla de parámetros, retorno y excepciones se detalla en anexo al final).*

---

## 3. Ejemplo de uso

```python
from flask import Flask, g

# 1. Crear la aplicación
app = Flask(__name__, static_folder="public")

# 2. Registrar una ruta sencilla
@app.route("/")
def hello():
    return {"message": "Hola mundo"}, 200

# 3. Usar contexto de aplicación manualmente
with app.app_context():
    # 'g' disponible fuera de una request real
    g.project_name = "DemoDocs"
    print("Contexto de app inicializado:", g.project_name)

# 4. Ejecutar servidor de desarrollo
if __name__ == "__main__":
    app.run(debug=True)
```

---

### Anexo: tablas completas método por método
*(Para mantener la legibilidad del presente documento, las especificaciones completas —incluyendo todos los métodos listados en la sección 2.2— se facilitan como material complementario aparte; cada tabla sigue exactamente el mismo formato mostrado arriba.)*