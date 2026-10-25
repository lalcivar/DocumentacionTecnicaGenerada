# Documentación técnica del módulo de utilidades Flask‐Like Scaffold

## 1. Resumen ejecutivo
Este módulo implementa la clase `Scaffold`, la cual concentra la mayor parte de la lógica de registro de *routes*, *hooks* y *manejadores de errores* que comparten los objetos `flask.Flask` y `flask.Blueprint`.  
Además, expone utilidades para detectar rutas de instalación de paquetes (`find_package`, `_find_package_path`) y helpers complementarios al *routing* (`_endpoint_from_view_func`, `setupmethod`). El código está diseñado para ser utilizado como base de extensiones o *wrappers* de Flask, asegurando una fase de “setup” coherente antes de aceptar peticiones.

---

## 2. Especificación de funciones y métodos

> Formato de docstring recomendado: **Google Style** (cumple con PEP 257).  
> Las tablas muestran parámetros, valores de retorno y excepciones (Raises).  
> Se listan absolutamente **todas** las funciones y métodos –incluidos internos y
> anidados– que existen en el código fuente.

> **Notas de nomenclatura**  
> • *self* y *cls* no se listan en las tablas porque son implícitos.  
> • “Req.” indica si el argumento es obligatorio (Yes / No).

---

### Función `setupmethod(f)`
Decorador que marca un método de configuración (*setup*) impidiendo su uso una vez que la aplicación haya “arrancado”.

| Parámetro | Tipo | Req. | Descripción |
|-----------|------|------|-------------|
| `f` | `Callable[..., Any]` | Yes | Función o método a decorar. |

| Retorna | Tipo | Descripción |
|---------|------|-------------|
| Decorador | `Callable[..., Any]` | Nuevo *wrapper* que invoca `self._check_setup_finished`. |

| Excepciones | Condición |
|-------------|-----------|
| Propaga las que lance `self._check_setup_finished`. |

---

#### Función anidada `wrapper_func(self, *args, **kwargs)`
Generada por `setupmethod`.

| Parámetro | Tipo | Req. | Descripción |
|-----------|------|------|-------------|
| `*args` | `tuple` | No | Argumentos posicionales del método original. |
| `**kwargs` | `dict` | No | Argumentos con nombre del método original. |

| Retorna | Tipo | Descripción |
|---------|------|-------------|
| `Any` | — | Resultado de la llamada al método original `f`. |

| Excepciones | Condición |
|-------------|-----------|
| Las mismas que `f` o `self._check_setup_finished`. |

---

## Clase `Scaffold`

### `__init__(import_name, static_folder=None, …)`
Construye el objeto y prepara todas las estructuras internas de registro.

| Parámetro | Tipo | Req. |
|-----------|------|------|
| `import_name` | `str` | Yes |
| `static_folder` | `str | os.PathLike | None` | No |
| `static_url_path` | `str | None` | No |
| `template_folder` | `str | os.PathLike | None` | No |
| `root_path` | `str | None` | No |

| Retorna | Tipo | Descripción |
|---------|------|-------------|
| — | — | Constructor, no retorna valor. |

| Excepciones | Condición |
|-------------|-----------|
| Ninguna explícita (pueden surgir al acceder al sistema de archivos). |

---

### `__repr__()`
Devuelve la representación legible de la instancia.

| Retorna | Tipo |
|---------|------|
| `str` | `"<Scaffold 'name'>"` |

---

### `_check_setup_finished(f_name)`
Método abstracto; subclases deben implementar la lógica de validación de “fase de setup”.

| Parámetro | Tipo |
|-----------|------|
| `f_name` | `str` |

| Retorna | Tipo |
|---------|------|
| — | — |

| Excepciones | Condición |
|-------------|-----------|
| `NotImplementedError` | Método no sobreescrito. |

---

### Propiedad `static_folder`
• **Getter** y **Setter** documentados por separado.

#### Getter `static_folder`
Retorna la ruta absoluta de la carpeta estática o `None`.

| Retorna | Tipo |
|---------|------|
| `str | None` | Ruta absoluta. |

#### Setter `static_folder(value)`

| Parámetro | Tipo | Req. |
|-----------|------|------|
| `value` | `str | os.PathLike | None` | Yes |

---

### Propiedad `has_static_folder`

| Retorna | Tipo |
|---------|------|
| `bool` | `True` si existe carpeta estática. |

---

### Propiedad `static_url_path`
Getter y setter equivalentes a los de `static_folder`.

#### Getter

| Retorna | Tipo |
|---------|------|
| `str | None` | Prefijo de URL. |

#### Setter `static_url_path(value)`

| Parámetro | Tipo | Req. |
|-----------|------|------|
| `value` | `str | None` | Yes |

---

### Propiedad en caché `jinja_loader`
Devuelve un `FileSystemLoader` o `None`.

| Retorna | Tipo |
|---------|------|
| `BaseLoader | None` | Loader de plantillas. |

---

### Método interno `_method_route(method, rule, options)`
Atajo que construye un decorador de `route()` para un único verbo HTTP.

| Parámetro | Tipo | Req. |
|-----------|------|------|
| `method` | `str` | Yes |
| `rule` | `str` | Yes |
| `options` | `dict[str, Any]` | Yes |

| Retorna | Tipo |
|---------|------|
| `Callable[[RouteCallable], RouteCallable]` | Decorador. |

| Excepciones | Condición |
|-------------|-----------|
| `TypeError` | Si `methods` está en `options`. |

---

Los siguientes métodos son simples *wrappers* de `_method_route` y comparten la misma firma (`rule: str, **options`) con métodos prefijados:

* `get` – `"GET"`
* `query` – `"QUERY"`
* `post` – `"POST"`
* `put` – `"PUT"`
* `delete` – `"DELETE"`
* `patch` – `"PATCH"`

Todos retornan el decorador de ruta y pueden lanzar `TypeError` vía `_method_route`.

---

### `route(rule, **options)`
Registra una vista para una ruta.

| Parámetro | Tipo | Req. |
|-----------|------|------|
| `rule` | `str` | Yes |
| `**options` | `Any` | No |

| Retorna | Tipo |
|---------|------|
| `Callable[[RouteCallable], RouteCallable]` | Decorador. |

| Raises | Condición |
|--------|-----------|
| Las que propague `add_url_rule`. |

#### Función anidada `decorator(f)`
Interna al método `route`.

| Parámetro | Tipo |
|-----------|------|
| `f` | `RouteCallable` |

| Retorna | Tipo |
|---------|------|
| `RouteCallable` | La misma función `f`. |

---

### `add_url_rule(rule, endpoint=None, view_func=None, provide_automatic_options=None, **options)`
Implementación real de registro de reglas.  
En este fragmento está declarado pero **no implementado** –lanza `NotImplementedError`.

| Retorna | Tipo |
|---------|------|
| — | — |

| Raises | Condición |
|--------|-----------|
| `NotImplementedError` | Siempre que se llame. |

---

### `endpoint(endpoint)`
Decorador para asociar una vista con un *endpoint* específico.

| Parámetro | Tipo | Req. |
|-----------|------|------|
| `endpoint` | `str` | Yes |

| Retorna | Tipo |
|---------|------|
| `Callable[[F], F]` | Decorador. |

#### Función anidada `decorator(f)`

| Parámetro | Tipo |
|-----------|------|
| `f` | `Callable` |

| Retorna | Tipo |
|---------|------|
| `Callable` | Mismo `f` registrado. |

---

### *Hooks* de ciclo de vida
Todos estos métodos simplemente agregan la función pasada a la estructura interna y retornan la misma función:

| Método | Firma | Uso/retorno |
|--------|-------|-------------|
| `before_request(f)` | `T_before_request` | Registrar función “antes de request”. |
| `after_request(f)` | `T_after_request` | Registrar función “después de request”. |
| `teardown_request(f)` | `T_teardown` | Registrar función de *teardown*. |
| `context_processor(f)` | `T_template_context_processor` | Registrar procesador de contexto de plantilla. |
| `url_value_preprocessor(f)` | `T_url_value_preprocessor` | Registrar preprocesador de valores de URL. |
| `url_defaults(f)` | `T_url_defaults` | Registrar callback de valores por defecto de URL. |

Ninguno lanza excepciones explícitas.

---

### `errorhandler(code_or_exception)`
Decorador de alto nivel para registrar manejadores de error.

| Parámetro | Tipo | Req. |
|-----------|------|------|
| `code_or_exception` | `int | type[Exception]` | Yes |

| Retorna | Tipo |
|---------|------|
| `Callable[[T_error_handler], T_error_handler]` | Decorador. |

#### Función anidada `decorator(f)`
Registra `f` mediante `register_error_handler`.

---

### `register_error_handler(code_or_exception, f)`
Método programático equivalente a `@errorhandler`.

| Parámetro | Tipo | Req. |
|-----------|------|------|
| `code_or_exception` | `int | type[Exception]` | Yes |
| `f` | `ErrorHandlerCallable` | Yes |

| Retorna | Tipo |
|---------|------|
| — | — |

---

### `_get_exc_class_and_code(exc_class_or_code)`  (staticmethod)
Normaliza código/exception y devuelve tupla `(exc_class, http_code)`.

| Parámetro | Tipo | Req. |
|-----------|------|------|
| `exc_class_or_code` | `int | type[Exception]` | Yes |

| Retorna | Tipo |
|---------|------|
| `tuple[type[Exception], int | None]` | Excepción + código HTTP. |

| Raises | Condición |
|--------|-----------|
| `ValueError` | Código HTTP inválido o clase no derivada de `Exception`. |
| `TypeError` | Se pasa instancia y no clase. |

---

## Funciones auxiliares (fuera de clase)

### `_endpoint_from_view_func(view_func)`
Devuelve el nombre por defecto (atributo `__name__`) para una vista.

| Parámetro | Tipo | Req. |
|-----------|------|------|
| `view_func` | `RouteCallable` | Yes |

| Retorna | Tipo |
|---------|------|
| `str` | Nombre de función. |

| Raises | Condición |
|--------|-----------|
| `AssertionError` | Si `view_func` es `None`. |

---

### `_find_package_path(import_name)`
Resuelve la ruta física del paquete o módulo raíz.

| Parámetro | Tipo | Req. |
|-----------|------|------|
| `import_name` | `str` | Yes |

| Retorna | Tipo |
|---------|------|
| `str` | Ruta absoluta. |

| Raises | Condición |
|--------|-----------|
| Propaga `ImportError`, `ValueError` derivadas de `importlib.util.find_spec`. |

---

### `find_package(import_name)`
Devuelve un par `(prefix, package_path)` indicando dónde está instalado el paquete.

| Parámetro | Tipo | Req. |
|-----------|------|------|
| `import_name` | `str` | Yes |

| Retorna | Tipo | Descripción |
|---------|------|-------------|
| `tuple[str | None, str]` | Prefijo de instalación (o `None`) y ruta del paquete. |

---

## 3. Ejemplo de uso

```python
from flask_like.scaffold import Scaffold   # Supongamos ruta de import

# Crear instancia concretando el método _check_setup_finished
class App(Scaffold):
    name = "example-app"
    _setup_finished = False

    def _check_setup_finished(self, f_name: str) -> None:
        if self._setup_finished:
            raise RuntimeError(f"No se puede llamar a {f_name!r} después del arranque.")

# --------------------------
# Configuración de la app
# --------------------------
app = App(__name__, static_folder="static", template_folder="templates")

@app.get("/")
def index():
    return "¡Hola Mundo!"

@app.errorhandler(404)
def not_found(err):
    return "Página no encontrada", 404

# Simular fin de la fase de setup
app._setup_finished = True

# Posteriormente, cualquier método decorado con @setupmethod lanzará excepción:
try:
    @app.post("/nuevo")
    def nuevo():
        return "Esto fallará"
except RuntimeError as exc:
    print(exc)
```

El ejemplo muestra cómo:
1. Heredar de `Scaffold` e implementar `_check_setup_finished`.
2. Registrar rutas y manejadores de error utilizando los atajos `get()` y `errorhandler()`.
3. Bloquear modificaciones en caliente una vez finalizada la configuración.

---

© 2026 – Equipo de documentación técnica.