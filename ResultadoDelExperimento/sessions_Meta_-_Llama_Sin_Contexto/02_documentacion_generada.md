# Documentación Técnica - `requests.sessions`

**Módulo:** `requests.sessions`  
**Archivo:** `sessions.py`  
**Librería:** `requests`  
**Versión de referencia:** Compatible con Python 3.x

### 1. Descripción General

El archivo `requests.sessions` implementa el núcleo de la persistencia y gestión de estado de la librería `requests`. Su objetivo principal es proporcionar el objeto `Session`, que permite **persistir parámetros y configuraciones a través de múltiples peticiones HTTP**.

A diferencia de usar `requests.get()` de forma aislada, una `Session` permite:
* Reutilizar conexiones mediante *connection pooling* a través de adaptadores `HTTPAdapter`.
* Persistir `cookies`, `headers`, `autenticación` y `proxies` entre peticiones.
* Gestionar de forma centralizada el manejo de redirecciones, verificación SSL, proxies y hooks.
* Optimizar el rendimiento al reutilizar conexiones TCP subyacentes.

El archivo actúa como orquestador entre los modelos `Request` / `PreparedRequest` / `Response` y la capa de transporte `adapters`.

### 2. Dependencias Principales

| Módulo | Uso |
| :--- | :--- |
| `requests.adapters.HTTPAdapter` | Adaptador por defecto para `http://` y `https://`. Gestiona el pool de conexiones `urllib3`. |
| `requests.models` | Importa `Request`, `PreparedRequest`, `Response`, `DEFAULT_REDIRECT_LIMIT` y `REDIRECT_STATI`. |
| `requests.cookies` | Gestión de `RequestsCookieJar`, fusión y extracción de cookies. |
| `requests.auth` | Construcción de cabecera `Authorization`. |
| `requests.utils` | Utilidades como `default_headers`, `requote_uri`, `resolve_proxies`, `get_environ_proxies`, `rewind_body`. |
| `collections.OrderedDict` | Mantiene el orden de los adaptadores montados por longitud de prefijo. |
| `time.perf_counter / time.time` | Medición de `elapsed` de la petición. Definido como `preferred_clock` según el SO. |

### 3. Componentes Principales

El archivo se compone de 2 funciones de utilidad, 1 Mixin, 1 clase principal y 1 función factory.

#### 3.1 Funciones de Utilidad

##### `merge_setting(request_setting, session_setting, dict_class=OrderedDict)`
Fusiona la configuración a nivel de petición con la configuración a nivel de sesión. Es la lógica central de persistencia.

* **Lógica:** Si uno de los dos es `None`, retorna el otro. Si ambos son diccionarios `Mapping`, los fusiona dando prioridad a `request_setting`. Elimina claves con valor `None` para permitir anular un setting de sesión.
* **Uso:** Para `headers`, `params`, `auth`, `proxies`, `verify`, `cert`.

##### `merge_hooks(request_hooks, session_hooks, dict_class=OrderedDict)`
Fusión especializada para el sistema de hooks. Evita que un hook vacío `{'response': []}` a nivel de petición sobrescriba y anule los hooks definidos en la sesión.

#### 3.2 Clase `SessionRedirectMixin`

Clase Mixin que encapsula toda la lógica de redirecciones HTTP. No se instancia directamente, es heredada por `Session`.

| Método | Descripción |
| :--- | :--- |
| `get_redirect_target(resp)` | Analiza una `Response`. Si `is_redirect` es `True`, extrae y decodifica correctamente el header `Location` de `latin1` a `utf8`. Retorna `str` o `None`. |
| `should_strip_auth(old_url, new_url)` | Determina si se debe eliminar la cabecera `Authorization` al redirigir. Retorna `True` si cambia el `hostname`, `port` o `scheme`. Contiene una excepción para permitir redirección `http -> https` en puertos estándar sin perder el auth. |
| `resolve_redirects(resp, req, ...)` | **Generador** que resuelve la cadena de redirecciones. Controla el historial, valida `max_redirects` y lanza `TooManyRedirects`, normaliza URLs relativas, preserva fragmentos, reconstruye método/headers/cookies/auth/proxies y hace `rewind` del body si es necesario. Puede ceder `PreparedRequest` si `yield_requests=True` o `Response` en caso contrario. |
| `rebuild_auth(prepared_request, response)` | Elimina `Authorization` si `should_strip_auth` es `True` y re-aplica autenticación vía `.netrc` si `trust_env` está activo. |
| `rebuild_proxies(prepared_request, proxies)` | Re-evalúa la configuración de proxies considerando `NO_PROXY` y variables de entorno. Elimina y regenera `Proxy-Authorization` si es necesario. |
| `rebuild_method(prepared_request, response)` | Ajusta el método HTTP según el código de redirección y el comportamiento de los navegadores: `303 See Other -> GET`, `302 Found -> GET` y `301 Moved Permanently + POST -> GET`. |

#### 3.3 Clase `Session(SessionRedirectMixin)`

Clase principal del módulo. Gestiona el ciclo de vida completo de una petición.

**Atributos de Configuración `__attrs__`:**
`headers` (CaseInsensitiveDict), `cookies` (RequestsCookieJar), `auth`, `proxies`, `hooks`, `params`, `verify` (bool/str), `cert`, `adapters` (OrderedDict), `stream`, `trust_env`, `max_redirects` (default 30).

**Métodos Principales:**

| Método | Descripción |
| :--- | :--- |
| `__init__()` | Inicializa la sesión con valores por defecto: `default_headers()`, `verify=True`, `trust_env=True`, `max_redirects=30`, `cookiejar_from_dict({})` y monta dos `HTTPAdapter` para `https://` y `http://`. |
| `__enter__ / __exit__` | Permite uso como context manager `with requests.Session() as s:` y asegura el cierre de conexiones. |
| `prepare_request(request)` | Fusiona un objeto `Request` con la configuración de la `Session` usando `merge_setting` y `merge_hooks`. Gestiona cookies y autenticación `netrc`. Retorna un `PreparedRequest` listo para enviar. |
| `request(method, url, ...)` | Método central. Crea un `Request`, lo prepara con `prepare_request()`, fusiona la configuración de entorno con `merge_environment_settings()` y delega el envío a `send()`. |
| `get(), post(), put(), patch(), delete(), head(), options()` | Wrappers de conveniencia sobre `request()`. Definen `allow_redirects` por defecto: `True` para `GET`/`OPTIONS` y `False` para `HEAD`. |
| `send(request, **kwargs)` | Envía un `PreparedRequest` a través del `BaseAdapter` correspondiente. Mide `elapsed`, ejecuta `dispatch_hook('response')`, persiste cookies, resuelve redirecciones si `allow_redirects=True` y gestiona `Response.history` y `Response._next`. Valida que no se envíe un objeto `Request` sin preparar. |
| `merge_environment_settings(url, proxies, stream, verify, cert)` | Fusiona la configuración de la sesión con variables de entorno `REQUESTS_CA_BUNDLE`, `CURL_CA_BUNDLE` y `get_environ_proxies()` si `trust_env=True`. |
| `get_adapter(url)` | Retorna el `BaseAdapter` apropiado buscando el prefijo más largo que coincida con la URL. Lanza `InvalidSchema` si no hay coincidencia. |
| `mount(prefix, adapter)` | Registra un adaptador para un prefijo. Reordena `self.adapters` de forma descendente por longitud de prefijo para asegurar el matching más específico primero. |
| `close()` | Cierra todos los adaptadores y sus pools de conexiones. |
| `__getstate__ / __setstate__` | Soporte para serialización `pickle` de la sesión. |

#### 3.4 Función `session()`

```python
def session() -> Session
```
Factory function marcada como **deprecated desde la v1.0.0**. Retorna una nueva instancia de `Session`. Se mantiene solo por compatibilidad hacia atrás. Se recomienda instanciar directamente `requests.Session()`.

### 4. Flujo de Ejecución Típico

1.  **Instanciación:** `s = requests.Session()` -> `__init__` monta adaptadores.
2.  **Petición:** `s.get('https://api.example.com')` -> llama a `request('GET', ...)`
3.  **Preparación:** `request()` crea un `Request` y llama a `prepare_request()` que fusiona headers/cookies/auth de la sesión.
4.  **Entorno:** `merge_environment_settings()` incorpora proxies y CA bundle del sistema.
5.  **Envío:** `send()` obtiene el adaptador con `get_adapter()`, envía la petición y calcula `r.elapsed`.
6.  **Redirección:** Si la respuesta es 3xx y `allow_redirects=True`, `resolve_redirects()` itera generando nuevas peticiones hasta obtener la respuesta final.
7.  **Retorno:** Se retorna el objeto `Response` final con su `history` poblado.

