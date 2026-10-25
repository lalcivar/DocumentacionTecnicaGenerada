# Documentación Técnica — `requests.sessions`

## 1. Resumen Ejecutivo

El módulo `requests.sessions` implementa el núcleo de persistencia y gestión de estado de la librería `requests`. Provee la clase `Session` y el mixin `SessionRedirectMixin` para reutilizar conexiones mediante *connection pooling*, persistir cookies, cabeceras, autenticación y proxies entre múltiples peticiones, y resolver redirecciones HTTP de forma conforme a RFC 7231/3986. Expone además utilidades de fusión de configuración (`merge_setting`, `merge_hooks`) y gestión de adaptadores de transporte.

## 2. Especificación de Funciones y Métodos

> **Nota de cobertura:** Se documenta el 100% de las funciones y métodos detectados en el código fuente provisto. No se detectaron funciones anidadas/locales, decoradores ni *callbacks* internos definidos dentro de otras funciones. Todas las entidades listadas conservan su nombre exacto.

### 2.1 Funciones a Nivel de Módulo

#### `merge_setting()`

```python
def merge_setting(
    request_setting: Any, 
    session_setting: Any, 
    dict_class: type = OrderedDict
) -> Any:
    """Determines appropriate setting for a given request, taking into account
    the explicit setting on that request, and the setting in the session. If a
    setting is a dictionary, they will be merged together using `dict_class`
    """
```

Fusiona la configuración a nivel de petición con la configuración a nivel de sesión. Si ambos valores son `Mapping`, los combina con `dict_class`; si alguno es `None`, retorna el otro; si no son diccionarios, prioriza `request_setting`. Elimina claves con valor `None`.

**Docstring Google Style:**
```python
def merge_setting(request_setting, session_setting, dict_class=OrderedDict):
    """Fusiona la configuración de petición y sesión.

    Args:
        request_setting (Any): Valor explícito de la petición. Puede ser
            Mapping, escalar o None.
        session_setting (Any): Valor por defecto de la sesión. Puede ser
            Mapping, escalar o None.
        dict_class (type): Clase de diccionario a usar para la fusión.
            Por defecto OrderedDict.

    Returns:
        Any: Configuración resultante. Si ambos son Mapping, retorna un
            dict fusionado sin claves None. En otro caso, retorna
            request_setting si no es None, o session_setting.

    Raises:
        TypeError: Si dict_class no es invocable como constructor de mapping.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `request_setting` | `Any` | Sí | Valor definido en el objeto `Request`. |
| `session_setting` | `Any` | Sí | Valor definido en el objeto `Session`. |
| `dict_class` | `type` | No (default `OrderedDict`) | Clase para construir el diccionario fusionado. |

| Retorno | Tipo | Descripción |
| :--- | :--- | :--- |
| `merged_setting` | `Any` | Diccionario fusionado o valor escalar priorizado. |

| Excepciones | Condición |
| :--- | :--- |
| `Ninguna explícita` | No lanza excepciones propias; propaga errores de `to_key_val_list`. |

---

#### `merge_hooks()`

```python
def merge_hooks(
    request_hooks: _t.HooksType,
    session_hooks: _t.HooksType,
    dict_class: type = OrderedDict,
) -> _t.HooksType:
    """Properly merges both requests and session hooks.

    This is necessary because when request_hooks == {'response': []}, the
    merge breaks Session hooks entirely.
    """
```

Fusión especializada para hooks que corrige el caso borde donde `{'response': []}` sobrescribiría los hooks de sesión. Delega en `merge_setting` cuando ambos contienen hooks válidos.

**Docstring Google Style:**
```python
def merge_hooks(request_hooks, session_hooks, dict_class=OrderedDict):
    """Fusiona correctamente los hooks de petición y sesión.

    Args:
        request_hooks (_t.HooksType): Hooks definidos en la petición.
            Ej: {'response': [callable]}.
        session_hooks (_t.HooksType): Hooks definidos en la sesión.
        dict_class (type): Clase de diccionario para la fusión.

    Returns:
        _t.HooksType: Diccionario de hooks resultante, priorizando
            request_hooks si session_hooks está vacío o es None y viceversa.

    Raises:
        AttributeError: Si alguno de los argumentos no implementa .get().
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `request_hooks` | `_t.HooksType` | Sí | Hooks de la petición (`dict[str, list[callable]]` o `None`). |
| `session_hooks` | `_t.HooksType` | Sí | Hooks de la sesión. |
| `dict_class` | `type` | No (default `OrderedDict`) | Clase para la fusión subyacente. |

| Retorno | Tipo | Descripción |
| :--- | :--- | :--- |
| `hooks` | `_t.HooksType` | Hooks fusionados. |

| Excepciones | Condición |
| :--- | :--- |
| `Ninguna explícita` | - |

---

#### `session()`

```python
def session() -> Session:
    """
    Returns a :class:`Session` for context-management.

    .. deprecated:: 1.0.0
    ...
    :rtype: Session
    """
```

Factoría de compatibilidad histórica. Retorna una nueva instancia de `Session`.

**Docstring Google Style:**
```python
def session():
    """Retorna una sesión para gestión de contexto.

    .. deprecated:: 1.0.0
        Usar directamente `requests.sessions.Session()`.

    Returns:
        Session: Nueva instancia de Session sin configuración adicional.

    Raises:
        Ninguna.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| *Ninguno* | - | - | - |

| Retorno | Tipo | Descripción |
| :--- | :--- | :--- |
| `Session` | `Session` | Nueva sesión. |

| Excepciones | Condición |
| :--- | :--- |
| `Ninguna` | - |

---

### 2.2 Clase `SessionRedirectMixin`

Mixin que encapsula toda la lógica de redirecciones HTTP.

#### `SessionRedirectMixin.send()`

```python
def send(self, request: PreparedRequest, **kwargs: Any) -> Response: ...
```

Método abstracto/stub. Define la interfaz que debe implementar la clase concreta (`Session`) para enviar una `PreparedRequest`. Es invocado por `resolve_redirects`.

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `request` | `PreparedRequest` | Sí | Petición preparada a enviar. |
| `**kwargs` | `Any` | No | Argumentos del adaptador (`stream`, `verify`, `proxies`, etc.). |

| Retorno | Tipo | Descripción |
| :--- | :--- | :--- |
| `Response` | `Response` | Respuesta del servidor. |

| Excepciones | Condición |
| :--- | :--- |
| `NotImplementedError` | Si no es sobrescrito por la subclase. |

---

#### `SessionRedirectMixin.get_redirect_target()`

```python
def get_redirect_target(self, resp: Response) -> str | None:
    """Receives a Response. Returns a redirect URI or ``None``"""
```

Determina si una respuesta es una redirección y extrae el header `Location` re-codificándolo de `latin1` a `utf8` para soportar URIs con caracteres no-ASCII.

**Docstring Google Style:**
```python
def get_redirect_target(self, resp):
    """Obtiene el objetivo de redirección de una respuesta.

    Args:
        resp (Response): Respuesta a evaluar. Debe exponer `is_redirect`
            y `headers`.

    Returns:
        str | None: URI de redirección normalizada como string nativo,
            o None si no es una redirección.

    Raises:
        KeyError: Si `is_redirect` es True pero falta el header 'location'.
        UnicodeDecodeError: Si la re-codificación latin1 -> utf8 falla.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `resp` | `Response` | Sí | Respuesta HTTP a inspeccionar. |

| Retorno | Tipo | Descripción |
| :--- | :--- | :--- |
| `location` | `str | None` | URI de redirección o `None`. |

| Excepciones | Condición |
| :--- | :--- |
| `KeyError` | `resp.is_redirect == True` y no existe `headers["location"]`. |

---

#### `SessionRedirectMixin.should_strip_auth()`

```python
def should_strip_auth(self, old_url: str, new_url: str) -> bool:
    """Decide whether Authorization header should be removed when redirecting"""
```

Decide si se debe eliminar la cabecera `Authorization` al redirigir, comparando host, esquema y puerto. Permite `http -> https` en puertos estándar (80/443) sin *strip* por compatibilidad histórica.

**Docstring Google Style:**
```python
def should_strip_auth(self, old_url, new_url):
    """Decide si eliminar la cabecera Authorization en una redirección.

    Args:
        old_url (str): URL origen de la petición.
        new_url (str): URL destino de la redirección.

    Returns:
        bool: True si debe eliminarse la autenticación, False en caso contrario.

    Raises:
        ValueError: Si alguna URL no es parseable por urlparse.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `old_url` | `str` | Sí | URL original. |
| `new_url` | `str` | Sí | URL de destino. |

| Retorno | Tipo | Descripción |
| :--- | :--- | :--- |
| `bool` | `bool` | `True` si cambia host/puerto/esquema (excepto caso http->https estándar). |

| Excepciones | Condición |
| :--- | :--- |
| `Ninguna explícita` | - |

---

#### `SessionRedirectMixin.resolve_redirects()`

```python
def resolve_redirects(
    self,
    resp: Response,
    req: PreparedRequest,
    stream: bool = False,
    timeout: _t.TimeoutType = None,
    verify: _t.VerifyType = True,
    cert: _t.CertType = None,
    proxies: dict[str, str] | None = None,
    yield_requests: bool = False,
    **adapter_kwargs: Any,
) -> Generator[Response, None, None]:
    """Receives a Response. Returns a generator of Responses or Requests."""
```

Generador que resuelve la cadena de redirecciones. Gestiona historial, consumo/liberación de conexiones, normalización de URL (RFC 1808, RFC 7231, RFC 3986), reconstrucción de método/cabeceras/cookies/auth/proxies y rebobinado del body.

**Docstring Google Style:**
```python
def resolve_redirects(self, resp, req, stream=False, timeout=None, verify=True, cert=None, proxies=None, yield_requests=False, **adapter_kwargs):
    """Resuelve redirecciones de forma iterativa.

    Args:
        resp (Response): Respuesta inicial que puede contener redirección.
        req (PreparedRequest): Petición original que generó resp.
        stream (bool): Si es True, no precarga el contenido.
        timeout (_t.TimeoutType): Timeout para peticiones redirigidas.
        verify (_t.VerifyType): Verificación SSL.
        cert (_t.CertType): Certificado cliente SSL.
        proxies (dict[str, str] | None): Diccionario de proxies.
        yield_requests (bool): Si es True, genera PreparedRequest en lugar de Response.
        **adapter_kwargs (Any): Argumentos adicionales para el adaptador.

    Yields:
        Response | PreparedRequest: Siguiente respuesta o petición en la cadena.

    Raises:
        TooManyRedirects: Si len(history) >= self.max_redirects.
        ChunkedEncodingError: Al consumir resp.content con encoding chunked inválido.
        ContentDecodingError: Al decodificar contenido fallido.
        RuntimeError: Errores de lectura del socket.
        InvalidSchema: Si la URL redirigida no tiene adaptador.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `resp` | `Response` | Sí | Respuesta actual. |
| `req` | `PreparedRequest` | Sí | Petición que originó `resp`. |
| `stream` | `bool` | No (default `False`) | Control de streaming. |
| `timeout` | `_t.TimeoutType` | No (default `None`) | Timeout. |
| `verify` | `_t.VerifyType` | No (default `True`) | Verificación TLS. |
| `cert` | `_t.CertType` | No (default `None`) | Cert cliente. |
| `proxies` | `dict[str, str] | None` | No (default `None`) | Proxies. |
| `yield_requests` | `bool` | No (default `False`) | Si `True`, genera `PreparedRequest`. |
| `**adapter_kwargs` | `Any` | No | Args extra para `send()`. |

| Retorno | Tipo | Descripción |
| :--- | :--- | :--- |
| `Generator` | `Generator[Response, None, None]` | Generador de respuestas/peticiones redirigidas. |

| Excepciones | Condición |
| :--- | :--- |
| `TooManyRedirects` | Se excede `self.max_redirects`. |

---

#### `SessionRedirectMixin.rebuild_auth()`

```python
def rebuild_auth(self, prepared_request: PreparedRequest, response: Response) -> None:
    """When being redirected we may want to strip authentication from the
    request to avoid leaking credentials. This method intelligently removes
    and reapplies authentication where possible to avoid credential loss.
    """
```

Elimina `Authorization` si `should_strip_auth` es `True` y reaplica autenticación vía `.netrc` si `trust_env` está activo.

**Docstring Google Style:**
```python
def rebuild_auth(self, prepared_request, response):
    """Reconstruye la autenticación tras una redirección.

    Args:
        prepared_request (PreparedRequest): Petición redirigida a modificar.
        response (Response): Respuesta que causó la redirección.

    Returns:
        None: Modifica prepared_request in-place.

    Raises:
        AssertionError: Si request/response no son PreparedRequest válidos.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `prepared_request` | `PreparedRequest` | Sí | Petición destino. |
| `response` | `Response` | Sí | Respuesta origen con `response.request`. |

| Retorno | Tipo | Descripción |
| :--- | :--- | :--- |
| `None` | `None` | Mutación in-place. |

| Excepciones | Condición |
| :--- | :--- |
| `AssertionError` | Si `_is_prepared` falla. |

---

#### `SessionRedirectMixin.rebuild_proxies()`

```python
def rebuild_proxies(
    self,
    prepared_request: PreparedRequest,
    proxies: dict[str, str] | None,
) -> dict[str, str]:
    """This method re-evaluates the proxy configuration by considering the
    environment variables. If we are redirected to a URL covered by
    NO_PROXY, we strip the proxy configuration. Otherwise, we set missing
    proxy keys for this URL (in case they were stripped by a previous
    redirect).
    ...
    :rtype: dict
    """
```

Re-evalúa proxies considerando `NO_PROXY` y variables de entorno, elimina `Proxy-Authorization` previo y lo regenera si corresponde (no-HTTPS con credenciales en URL del proxy).

**Docstring Google Style:**
```python
def rebuild_proxies(self, prepared_request, proxies):
    """Re-evalúa la configuración de proxy tras redirección.

    Args:
        prepared_request (PreparedRequest): Petición redirigida.
        proxies (dict[str, str] | None): Proxies actuales.

    Returns:
        dict[str, str]: Nuevo diccionario de proxies resuelto.

    Raises:
        AssertionError: Si prepared_request no es PreparedRequest válido.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `prepared_request` | `PreparedRequest` | Sí | Petición con nueva URL. |
| `proxies` | `dict[str, str] | None` | Sí | Proxies previos. |

| Retorno | Tipo | Descripción |
| :--- | :--- | :--- |
| `new_proxies` | `dict[str, str]` | Proxies actualizados. |

| Excepciones | Condición |
| :--- | :--- |
| `AssertionError` | Si `_is_prepared` falla. |

---

#### `SessionRedirectMixin.rebuild_method()`

```python
def rebuild_method(self, prepared_request: PreparedRequest, response: Response) -> None:
    """When being redirected we may want to change the method of the request
    based on certain specs or browser behavior.
    """
```

Ajusta el método HTTP según el código de estado: `303 -> GET` (excepto HEAD), `302 -> GET` (excepto HEAD), `301 + POST -> GET`.

**Docstring Google Style:**
```python
def rebuild_method(self, prepared_request, response):
    """Reconstruye el método HTTP tras redirección.

    Args:
        prepared_request (PreparedRequest): Petición a mutar.
        response (Response): Respuesta con status_code de redirección.

    Returns:
        None: Modifica prepared_request.method in-place.

    Raises:
        Ninguna.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `prepared_request` | `PreparedRequest` | Sí | Petición a ajustar. |
| `response` | `Response` | Sí | Respuesta con `status_code`. |

| Retorno | Tipo | Descripción |
| :--- | :--- | :--- |
| `None` | `None` | Mutación in-place. |

| Excepciones | Condición |
| :--- | :--- |
| `Ninguna` | - |

---

### 2.3 Clase `Session`

#### `Session.__init__()`

```python
def __init__(self) -> None:
```

Inicializa una sesión con valores por defecto: `headers` (via `default_headers()`), `auth=None`, `proxies={}`, `hooks=default_hooks()`, `params={}`, `stream=False`, `verify=True`, `cert=None`, `max_redirects=30`, `trust_env=True`, `cookies=RequestsCookieJar()`, `adapters` con `HTTPAdapter` para `http://` y `https://`.

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| *Ninguno* | - | - | - |

| Retorno | Tipo | Descripción |
| :--- | :--- | :--- |
| `None` | `None` | Inicialización in-place. |

| Excepciones | Condición |
| :--- | :--- |
| `Ninguna` | - |

---

#### `Session.__enter__()`

```python
def __enter__(self) -> Self:
```

Soporte para *context manager*. Retorna `self`.

| Retorno | Tipo | Descripción |
| :--- | :--- | :--- |
| `Self` | `Session` | La propia sesión. |

---

#### `Session.__exit__()`

```python
def __exit__(self, *args: Any) -> None:
```

Cierra la sesión al salir del contexto, invocando `self.close()`.

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `*args` | `Any` | No | `(exc_type, exc_val, exc_tb)` ignorados. |

| Retorno | Tipo | Descripción |
| :--- | :--- | :--- |
| `None` | `None` | - |

---

#### `Session.prepare_request()`

```python
def prepare_request(self, request: Request) -> PreparedRequest:
    """Constructs a :class:`PreparedRequest <PreparedRequest>` for
    transmission and returns it. ...
    :param request: :class:`Request` instance to prepare with this
        session's settings.
    :rtype: requests.PreparedRequest
    """
```

Fusiona `Request` con la configuración de `Session` (cookies, headers, params, auth, hooks) y retorna un `PreparedRequest` listo para `send()`.

**Docstring Google Style:**
```python
def prepare_request(self, request):
    """Construye un PreparedRequest fusionando Request y Session.

    Args:
        request (Request): Petición sin preparar.

    Returns:
        PreparedRequest: Petición preparada y lista para enviar.

    Raises:
        ValueError: Si faltan url o method en la petición.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `request` | `Request` | Sí | Petición a preparar. |

| Retorno | Tipo | Descripción |
| :--- | :--- | :--- |
| `PreparedRequest` | `PreparedRequest` | Petición preparada. |

---

#### `Session.request()`

```python
def request(
    self,
    method: str,
    url: _t.UriType,
    params: _t.ParamsType = None,
    data: _t.DataType = None,
    headers: _t.HeadersType = None,
    cookies: RequestsCookieJar | CookieJar | dict[str, str] | None = None,
    files: _t.FilesType = None,
    auth: _t.AuthType = None,
    timeout: _t.TimeoutType = None,
    allow_redirects: bool = True,
    proxies: dict[str, str] | None = None,
    hooks: _t.HooksInputType | None = None,
    stream: bool | None = None,
    verify: _t.VerifyType | None = None,
    cert: _t.CertType = None,
    json: _t.JsonType = None,
) -> Response:
    """Constructs a :class:`Request <Request>`, prepares it and sends it.
    Returns :class:`Response <Response>` object.
    ...
    :rtype: requests.Response
    """
```

Método central. Crea un `Request`, lo prepara con `prepare_request`, fusiona configuración de entorno con `merge_environment_settings` y lo envía con `send`.

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `method` | `str` | Sí | Verbo HTTP (`GET`, `POST`, etc.). |
| `url` | `_t.UriType` | Sí | URL (`str` o `bytes`). |
| `params` | `_t.ParamsType` | No | Querystring. |
| `data` | `_t.DataType` | No | Body form-encoded / bytes / file-like. |
| `headers` | `_t.HeadersType` | No | Cabeceras HTTP. |
| `cookies` | `RequestsCookieJar | CookieJar | dict | None` | No | Cookies. |
| `files` | `_t.FilesType` | No | Archivos para multipart. |
| `auth` | `_t.AuthType` | No | Tupla o callable de autenticación. |
| `timeout` | `_t.TimeoutType` | No | Timeout en segundos o tupla `(connect, read)`. |
| `allow_redirects` | `bool` | No (default `True`) | Seguir redirecciones. |
| `proxies` | `dict[str, str] | None` | No | Proxies. |
| `hooks` | `_t.HooksInputType | None` | No | Hooks de evento. |
| `stream` | `bool | None` | No | Streaming de respuesta. |
| `verify` | `_t.VerifyType | None` | No | Verificación TLS. |
| `cert` | `_t.CertType` | No | Cert cliente. |
| `json` | `_t.JsonType` | No | Payload JSON serializable. |

| Retorno | Tipo | Descripción |
| :--- | :--- | :--- |
| `Response` | `Response` | Respuesta final tras redirecciones. |

| Excepciones | Condición |
| :--- | :--- |
| `InvalidSchema` | URL sin adaptador montado. |
| `TooManyRedirects` | Exceso de redirecciones. |
| `RequestException` y subclases | Errores de red, timeout, SSL. |

---

#### `Session.get()`

```python
def get(self, url: _t.UriType, params: _t.ParamsType = None, **kwargs: Unpack[_t.GetKwargs]) -> Response:
    r"""Sends a GET request. Returns :class:`Response` object.
    ...
    :rtype: requests.Response
    """
```

Wrapper sobre `request("GET", ...)`. Fuerza `allow_redirects=True` por defecto.

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `url` | `_t.UriType` | Sí | URL destino. |
| `params` | `_t.ParamsType` | No | Querystring. |
| `**kwargs` | `Unpack[_t.GetKwargs]` | No | Argumentos pasados a `request`. |

| Retorno | Tipo | Descripción |
| :--- | :--- | :--- |
| `Response` | `Response` | Respuesta. |

---

#### `Session.options()`

```python
def options(self, url: _t.UriType, **kwargs: Unpack[_t.RequestKwargs]) -> Response:
    r"""Sends a OPTIONS request. Returns :class:`Response` object."""
```

Wrapper sobre `request("OPTIONS", ...)`. `allow_redirects=True` por defecto.

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `url` | `_t.UriType` | Sí | URL destino. |
| `**kwargs` | `Unpack[_t.RequestKwargs]` | No | Args de `request`. |

| Retorno | Tipo | Descripción |
| :--- | :--- | :--- |
| `Response` | `Response` | Respuesta. |

---

#### `Session.head()`

```python
def head(self, url: _t.UriType, **kwargs: Unpack[_t.RequestKwargs]) -> Response:
    r"""Sends a HEAD request. Returns :class:`Response` object."""
```

Wrapper sobre `request("HEAD", ...)`. `allow_redirects=False` por defecto (semántica HEAD).

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `url` | `_t.UriType` | Sí | URL destino. |
| `**kwargs` | `Unpack[_t.RequestKwargs]` | No | Args de `request`. |

| Retorno | Tipo | Descripción |
| :--- | :--- | :--- |
| `Response` | `Response` | Respuesta sin body. |

---

#### `Session.post()`

```python
def post(self, url: _t.UriType, data: _t.DataType = None, json: _t.JsonType = None, **kwargs: Unpack[_t.PostKwargs]) -> Response:
    r"""Sends a POST request. Returns :class:`Response` object."""
```

Wrapper sobre `request("POST", ...)`.

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `url` | `_t.UriType` | Sí | URL destino. |
| `data` | `_t.DataType` | No | Body. |
| `json` | `_t.JsonType` | No | Payload JSON. |
| `**kwargs` | `Unpack[_t.PostKwargs]` | No | Args de `request`. |

| Retorno | Tipo | Descripción |
| :--- | :--- | :--- |
| `Response` | `Response` | Respuesta. |

---

#### `Session.put()`

```python
def put(self, url: _t.UriType, data: _t.DataType = None, **kwargs: Unpack[_t.DataKwargs]) -> Response:
    r"""Sends a PUT request. Returns :class:`Response` object."""
```

Wrapper sobre `request("PUT", ...)`.

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `url` | `_t.UriType` | Sí | URL destino. |
| `data` | `_t.DataType` | No | Body. |
| `**kwargs` | `Unpack[_t.DataKwargs]` | No | Args de `request`. |

| Retorno | Tipo | Descripción |
| :--- | :--- | :--- |
| `Response` | `Response` | Respuesta. |

---

#### `Session.patch()`

```python
def patch(self, url: _t.UriType, data: _t.DataType = None, **kwargs: Unpack[_t.DataKwargs]) -> Response:
    r"""Sends a PATCH request. Returns :class:`Response` object."""
```

Wrapper sobre `request("PATCH", ...)`.

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `url` | `_t.UriType` | Sí | URL destino. |
| `data` | `_t.DataType` | No | Body. |
| `**kwargs` | `Unpack[_t.DataKwargs]` | No | Args de `request`. |

| Retorno | Tipo | Descripción |
| :--- | :--- | :--- |
| `Response` | `Response` | Respuesta. |

---

#### `Session.delete()`

```python
def delete(self, url: _t.UriType, **kwargs: Unpack[_t.RequestKwargs]) -> Response:
    r"""Sends a DELETE request. Returns :class:`Response` object."""
```

Wrapper sobre `request("DELETE", ...)`.

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `url` | `_t.UriType` | Sí | URL destino. |
| `**kwargs` | `Unpack[_t.RequestKwargs]` | No | Args de `request`. |

| Retorno | Tipo | Descripción |
| :--- | :--- | :--- |
| `Response` | `Response` | Respuesta. |

---

#### `Session.send()`

```python
def send(self, request: PreparedRequest, **kwargs: Any) -> Response:
    """Send a given PreparedRequest.
    :rtype: requests.Response
    """
```

Envía una `PreparedRequest` usando el adaptador correspondiente, gestiona hooks, persistencia de cookies, medición de `elapsed`, resolución de redirecciones y precarga de contenido si `stream=False`.

**Docstring Google Style:**
```python
def send(self, request, **kwargs):
    """Envía una PreparedRequest.

    Args:
        request (PreparedRequest): Petición preparada. No se acepta Request sin preparar.
        **kwargs (Any): stream, verify, cert, proxies, allow_redirects, timeout, etc.

    Returns:
        Response: Respuesta final con historial y cookies persistidas.

    Raises:
        ValueError: Si request es una instancia de Request sin preparar.
        InvalidSchema: Si no hay adaptador para la URL.
        TooManyRedirects: Si se excede max_redirects.
        AssertionError: Si request no pasa _is_prepared.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `request` | `PreparedRequest` | Sí | Petición preparada. |
| `**kwargs` | `Any` | No | `stream`, `verify`, `cert`, `proxies`, `allow_redirects`, `timeout`. |

| Retorno | Tipo | Descripción |
| :--- | :--- | :--- |
| `Response` | `Response` | Respuesta con `history` y `elapsed` poblados. |

| Excepciones | Condición |
| :--- | :--- |
| `ValueError` | `isinstance(request, Request)` es `True`. |
| `InvalidSchema` | `get_adapter` no encuentra prefijo. |
| `TooManyRedirects` | Propagada desde `resolve_redirects`. |

---

#### `Session.merge_environment_settings()`

```python
def merge_environment_settings(
    self,
    url: str,
    proxies: dict[str, str] | None,
    stream: bool | None,
    verify: _t.VerifyType | None,
    cert: _t.CertType,
) -> dict[str, Any]:
    """
    Check the environment and merge it with some settings.
    :rtype: dict
    """
```

Combina configuración de entorno (`HTTP_PROXY`, `NO_PROXY`, `REQUESTS_CA_BUNDLE`, `CURL_CA_BUNDLE`) con la configuración de sesión y de la petición, respetando `trust_env`.

**Docstring Google Style:**
```python
def merge_environment_settings(self, url, proxies, stream, verify, cert):
    """Fusiona configuración de entorno con la de sesión.

    Args:
        url (str): URL destino para resolver proxies.
        proxies (dict[str, str] | None): Proxies explícitos de la petición.
        stream (bool | None): Flag de streaming.
        verify (_t.VerifyType | None): Verificación TLS.
        cert (_t.CertType): Certificado cliente.

    Returns:
        dict[str, Any]: Diccionario con claves proxies, stream, verify, cert.

    Raises:
        Ninguna.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `url` | `str` | Sí | URL para evaluar proxies. |
| `proxies` | `dict[str, str] | None` | Sí | Proxies de la petición. |
| `stream` | `bool | None` | Sí | Streaming solicitado. |
| `verify` | `_t.VerifyType | None` | Sí | Verificación TLS solicitada. |
| `cert` | `_t.CertType` | Sí | Cert cliente solicitado. |

| Retorno | Tipo | Descripción |
| :--- | :--- | :--- |
| `dict` | `dict[str, Any]` | `{"proxies": ..., "stream": ..., "verify": ..., "cert": ...}` |

---

#### `Session.get_adapter()`

```python
def get_adapter(self, url: str) -> BaseAdapter:
    """
    Returns the appropriate connection adapter for the given URL.
    :rtype: requests.adapters.BaseAdapter
    """
```

Retorna el adaptador cuyo prefijo coincide con la URL (comparación case-insensitive). Itera `self.adapters` ordenados por longitud de prefijo descendente.

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `url` | `str` | Sí | URL a resolver. |

| Retorno | Tipo | Descripción |
| :--- | :--- | :--- |
| `BaseAdapter` | `BaseAdapter` | Adaptador correspondiente. |

| Excepciones | Condición |
| :--- | :--- |
| `InvalidSchema` | Ningún prefijo coincide con `url`. |

---

#### `Session.close()`

```python
def close(self) -> None:
    """Closes all adapters and as such the session"""
```

Cierra todos los adaptadores y sus pools de conexiones.

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| *Ninguno* | - | - | - |

| Retorno | Tipo | Descripción |
| :--- | :--- | :--- |
| `None` | `None` | - |

---

#### `Session.mount()`

```python
def mount(self, prefix: str, adapter: BaseAdapter) -> None:
    """Registers a connection adapter to a prefix.
    Adapters are sorted in descending order by prefix length.
    """
```

Registra un adaptador para un prefijo y reordena `self.adapters` para que los prefijos más largos tengan prioridad.

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `prefix` | `str` | Sí | Prefijo URL (`"https://"`, `"http://example.com"`). |
| `adapter` | `BaseAdapter` | Sí | Adaptador a montar. |

| Retorno | Tipo | Descripción |
| :--- | :--- | :--- |
| `None` | `None` | Mutación in-place de `self.adapters`. |

---

#### `Session.__getstate__()`

```python
def __getstate__(self) -> dict[str, Any]:
```

Soporte para `pickle`. Retorna diccionario con los atributos listados en `__attrs__`.

| Retorno | Tipo | Descripción |
| :--- | :--- | :--- |
| `dict` | `dict[str, Any]` | Estado serializable de la sesión. |

---

#### `Session.__setstate__()`

```python
def __setstate__(self, state: dict[str, Any]) -> None:
```

Restaura el estado desde `pickle` asignando cada atributo del diccionario.

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `state` | `dict[str, Any]` | Sí | Estado previamente obtenido de `__getstate__`. |

| Retorno | Tipo | Descripción |
| :--- | :--- | :--- |
| `None` | `None` | Restauración in-place. |

---

## 3. Ejemplo de Uso

```python
import requests
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry

# 1. Uso básico con context manager y persistencia de cookies/headers
with requests.Session() as s:
    s.headers.update({"User-Agent": "MiApp/1.0"})
    s.auth = ("usuario", "password")
    s.max_redirects = 5

    # Montar adaptador con reintentos para mayor resiliencia
    retry = Retry(total=3, backoff_factor=0.5, status_forcelist=[500, 502, 503, 504])
    adapter = HTTPAdapter(max_retries=retry, pool_connections=10, pool_maxsize=10)
    s.mount("https://", adapter)
    s.mount("http://", adapter)

    # 2. Preparar petición manualmente para inspeccionar antes de enviar
    req = requests.Request("GET", "https://httpbin.org/cookies/set", params={"session": "abc123"})
    prepped = s.prepare_request(req)
    print(f"URL preparada: {prepped.url}")
    print(f"Headers: {prepped.headers}")

    # 3. Enviar y manejar redirecciones automáticamente
    resp = s.send(prepped, timeout=(3.05, 10), allow_redirects=True)
    print(f"Status: {resp.status_code}, Historial: {len(resp.history)}")
    print(f"Cookies de sesión: {s.cookies.get_dict()}")
    print(f"Tiempo transcurrido: {resp.elapsed.total_seconds()}s")

    # 4. Métodos de conveniencia (GET, POST) con fusión de settings
    # merge_setting y merge_hooks se usan internamente aquí
    r_get = s.get(
        "https://httpbin.org/get",
        params={"q": "python"},
        headers={"X-Custom": "valor"},
        verify=True
    )
    r_get.raise_for_status()
    print(r_get.json()["args"])

    r_post = s.post(
        "https://httpbin.org/post",
        json={"nombre": "Ada", "rol": "dev"},
        hooks={"response": [lambda r, *a, **kw: print(f"Hook: {r.status_code}")]}
    )
    print(f"POST {r_post.status_code}")

    # 5. Desactivar redirecciones y obtener el siguiente destino manualmente
    r_no_redirect = s.get("https://httpbin.org/redirect/1", allow_redirects=False)
    if r_no_redirect.is_redirect:
        print(f"Redirige a: {r_no_redirect.headers['Location']}")
        # Acceso al PreparedRequest de la siguiente redirección
        print(f"Siguiente request: {r_no_redirect.next.url}")

# Sesión cerrada automáticamente por __exit__
```
