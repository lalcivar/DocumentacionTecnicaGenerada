# Documentación Técnica - `requests.models`

## 1. Resumen Ejecutivo

El módulo `requests.models` contiene los objetos primarios que impulsan la librería `Requests`. Implementa las clases `Request`, `PreparedRequest` y `Response`, junto con los mixins `RequestEncodingMixin` y `RequestHooksMixin`, encargados de la codificación de parámetros, construcción de cuerpos multipart, gestión de hooks y el ciclo de vida completo de una petición/respuesta HTTP.

Este módulo abstrae la preparación, serialización y envío de peticiones HTTP, así como el consumo, decodificación y validación de respuestas, incluyendo manejo de redirecciones, cookies, autenticación y streaming.

---

## 2. Especificación de Funciones y Métodos

### 2.1 Clase `RequestEncodingMixin`

Mixin responsable de la codificación de URL, parámetros y archivos para peticiones HTTP.

#### 2.1.1 `RequestEncodingMixin.path_url` (property)

```python
@property
def path_url(self) -> str:
    """Construye la URL de ruta (path + query) a utilizar.

    Returns:
        str: Ruta y query string extraídos de `self.url`. Retorna "/" si no hay path.
    """
```

| Atributo | Tipo | Descripción |
| :--- | :--- | :--- |
| `self.url` | `str \| None` | URL completa de la petición. Se parsea con `urlsplit`. |

| Retorno | Tipo | Descripción |
| :--- | :--- | :--- |
| `path_url` | `str` | Concatenación de `path` y `query` (ej. `/get?foo=bar`). |

**Raises:** No lanza excepciones propias. Depende de `urlsplit`.

#### 2.1.2 `RequestEncodingMixin._encode_params()`

```python
@staticmethod
def _encode_params(data: _t.EncodableDataType) -> str | bytes | _t.SupportsRead[str | bytes]:
    """Codifica parámetros en un fragmento de datos.

    Codifica correctamente parámetros cuando se pasan como dict o lista de
    tuplas de 2 elementos. El orden se conserva si es lista de tuplas.

    Args:
        data: Datos a codificar. Puede ser str, bytes, objeto con método read(),
              dict o lista de tuplas [(key, value)].

    Returns:
        str | bytes | SupportsRead: Datos codificados como query string (urlencode),
            o el mismo objeto si ya es str/bytes/file-like.

    Raises:
        TypeError: Si el tipo de dato no es iterable de forma esperada.
    """
```

Sobrecargas (`@overload`):
* `_encode_params(data: str) -> str`
* `_encode_params(data: bytes) -> bytes`
* `_encode_params(data: SupportsRead[str | bytes]) -> SupportsRead[str | bytes]`
* `_encode_params(data: KVDataType) -> str`

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `data` | `str \| bytes \| SupportsRead \| KVDataType` | Sí | Datos a codificar. |

| Retorno | Tipo | Descripción |
| :--- | :--- | :--- |
| `encoded` | `str \| bytes \| SupportsRead` | Parámetros codificados con `urlencode(doseq=True)` o dato original. |

#### 2.1.3 `RequestEncodingMixin._encode_files()`

```python
@staticmethod
def _encode_files(files: _t.FilesType, data: _t.RawDataType | None) -> tuple[bytes, str]:
    """Construye el cuerpo para una petición multipart/form-data.

    Args:
        files: Diccionario o lista de tuplas de archivos. Cada entrada puede ser
            2-tupla (filename, fileobj), 3-tupla (filename, fileobj, content_type)
            o 4-tupla (filename, fileobj, content_type, custom_headers).
        data: Datos de formulario adicionales a incluir junto a los archivos.

    Returns:
        tuple[bytes, str]: Tupla (body, content_type) donde body son los bytes
            codificados y content_type es el header multipart con boundary.

    Raises:
        ValueError: Si `files` está vacío o si `data` es un string.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `files` | `_t.FilesType` | Sí | Archivos a codificar. |
| `data` | `_t.RawDataType \| None` | No | Datos de formulario adicionales. |

| Retorno | Tipo | Descripción |
| :--- | :--- | :--- |
| `body` | `bytes` | Cuerpo multipart codificado. |
| `content_type` | `str` | Valor para el header `Content-Type`. |

| Excepción | Condición |
| :--- | :--- |
| `ValueError` | `files` es falsy. |
| `ValueError` | `data` es `basestring`. |

---

### 2.2 Clase `RequestHooksMixin`

Mixin para el sistema de eventos/hooks de Requests.

#### 2.2.1 `RequestHooksMixin.register_hook()`

```python
def register_hook(self, event: str, hook: Iterable[_t.HookType] | _t.HookType) -> None:
    """Registra correctamente un hook.

    Args:
        event: Nombre del evento (ej. "response").
        hook: Callable o iterable de callables a registrar.

    Returns:
        None

    Raises:
        ValueError: Si `event` no existe en `self.hooks`.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `event` | `str` | Sí | Nombre del evento. |
| `hook` | `Iterable[HookType] \| HookType` | Sí | Hook o lista de hooks. |

| Excepción | Condición |
| :--- | :--- |
| `ValueError` | `event` no soportado. |

#### 2.2.2 `RequestHooksMixin.deregister_hook()`

```python
def deregister_hook(self, event: str, hook: _t.HookType) -> bool:
    """Elimina un hook previamente registrado.

    Args:
        event: Nombre del evento.
        hook: Callable a eliminar.

    Returns:
        bool: True si el hook existía y fue eliminado, False si no se encontró.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `event` | `str` | Sí | Nombre del evento. |
| `hook` | `_t.HookType` | Sí | Hook a eliminar. |

| Retorno | Tipo | Descripción |
| :--- | :--- | :--- |
| `result` | `bool` | Éxito de la operación. |

---

### 2.3 Clase `Request`

Objeto creado por el usuario para preparar una `PreparedRequest`.

#### 2.3.1 `Request.__init__()`

```python
def __init__(self, method: str | None = None, url: _t.UriType | None = None, headers: _t.HeadersType = None, files: _t.FilesType = None, data: _t.DataType = None, params: _t.ParamsType = None, auth: _t.AuthType = None, cookies: RequestsCookieJar | CookieJar | dict[str, str] | None = None, hooks: _t.HooksInputType | None = None, json: _t.JsonType = None) -> None:
    """Inicializa una petición de usuario.

    Args:
        method: Método HTTP a usar (GET, POST, etc.).
        url: URL a la que enviar la petición.
        headers: Diccionario de headers.
        files: Diccionario {filename: fileobject} para upload multipart.
        data: Cuerpo a adjuntar. Si es dict o lista de tuplas, se hace form-encoding.
        params: Parámetros URL a añadir a la URL.
        auth: Manejador Auth o tupla (user, pass).
        cookies: Diccionario o CookieJar de cookies.
        hooks: Diccionario de hooks de callback.
        json: Objeto JSON para el cuerpo (si files o data no están especificados).

    Returns:
        None
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `method` | `str \| None` | No | Verbo HTTP. |
| `url` | `_t.UriType \| None` | No | URL destino. |
| `headers` | `_t.HeadersType` | No | Headers iniciales. Default `{}`. |
| `files` | `_t.FilesType` | No | Archivos. Default `[]`. |
| `data` | `_t.DataType` | No | Body. Default `[]`. |
| `params` | `_t.ParamsType` | No | Query params. Default `{}`. |
| `auth` | `_t.AuthType` | No | Autenticación. |
| `cookies` | `RequestsCookieJar \| CookieJar \| dict \| None` | No | Cookies. |
| `hooks` | `_t.HooksInputType \| None` | No | Hooks. Default `{}`. |
| `json` | `_t.JsonType` | No | Payload JSON. |

#### 2.3.2 `Request.__repr__()`

```python
def __repr__(self) -> str:
    """Retorna representación textual del objeto.

    Returns:
        str: Formato "<Request [METHOD]>".
    """
```

#### 2.3.3 `Request.prepare()`

```python
def prepare(self) -> PreparedRequest:
    """Construye una PreparedRequest para transmisión y la retorna.

    Returns:
        PreparedRequest: Petición totalmente preparada lista para ser enviada
            vía `Session.send()`.

    Raises:
        InvalidURL: Si la URL es inválida (propagado desde PreparedRequest).
        MissingSchema: Si falta el esquema en la URL.
    """
```

| Retorno | Tipo | Descripción |
| :--- | :--- | :--- |
| `p` | `PreparedRequest` | Instancia preparada. |

---

### 2.4 Clase `PreparedRequest`

Objeto totalmente mutable que contiene los bytes exactos que se enviarán al servidor. Hereda de `RequestEncodingMixin` y `RequestHooksMixin`.

#### 2.4.1 `PreparedRequest.__init__()`

```python
def __init__(self) -> None:
    """Inicializa una PreparedRequest vacía.

    Attributes:
        method: Verbo HTTP.
        url: URL HTTP.
        headers: Diccionario de headers (CaseInsensitiveDict).
        _cookies: CookieJar interno.
        body: Cuerpo de la petición.
        hooks: Diccionario de hooks.
        _body_position: Posición inicial de un body file-like.
    """
```

#### 2.4.2 `PreparedRequest.prepare()`

```python
def prepare(self, method: str | None = None, url: _t.UriType | None = None, headers: Mapping[str, str | bytes] | None = None, files: _t.FilesType = None, data: _t.DataType = None, params: _t.ParamsType = None, auth: _t.AuthType = None, cookies: RequestsCookieJar | CookieJar | dict[str, str] | None = None, hooks: _t.HooksInputType | None = None, json: _t.JsonType = None) -> None:
    """Prepara la petición completa con los parámetros dados.

    Orquesta secuencialmente: prepare_method, prepare_url, prepare_headers,
    prepare_cookies, prepare_body, prepare_auth y prepare_hooks.

    Args:
        method: Método HTTP.
        url: URL destino.
        headers: Headers.
        files: Archivos.
        data: Datos del body.
        params: Parámetros URL.
        auth: Autenticación.
        cookies: Cookies.
        hooks: Hooks.
        json: Payload JSON.

    Returns:
        None
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `method` | `str \| None` | No | Método HTTP. |
| `url` | `_t.UriType \| None` | No | URL. |
| `headers` | `Mapping[str, str \| bytes] \| None` | No | Headers. |
| `files` | `_t.FilesType` | No | Archivos. |
| `data` | `_t.DataType` | No | Datos. |
| `params` | `_t.ParamsType` | No | Params. |
| `auth` | `_t.AuthType` | No | Auth. |
| `cookies` | `RequestsCookieJar \| CookieJar \| dict \| None` | No | Cookies. |
| `hooks` | `_t.HooksInputType \| None` | No | Hooks. |
| `json` | `_t.JsonType` | No | JSON. |

#### 2.4.3 `PreparedRequest.__repr__()`

```python
def __repr__(self) -> str:
    """Retorna representación textual.

    Returns:
        str: "<PreparedRequest [METHOD]>".
    """
```

#### 2.4.4 `PreparedRequest.copy()`

```python
def copy(self) -> PreparedRequest:
    """Crea una copia superficial de la PreparedRequest.

    Returns:
        PreparedRequest: Nueva instancia con method, url, headers, _cookies,
            body, hooks y _body_position copiados.
    """
```

| Retorno | Tipo | Descripción |
| :--- | :--- | :--- |
| `p` | `PreparedRequest` | Copia del objeto actual. |

#### 2.4.5 `PreparedRequest.prepare_method()`

```python
def prepare_method(self, method: str | None) -> None:
    """Prepara el método HTTP dado.

    Args:
        method: Método HTTP. Se convierte a mayúsculas y a native string.

    Returns:
        None
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `method` | `str \| None` | No | Método a preparar. |

#### 2.4.6 `PreparedRequest._get_idna_encoded_host()`

```python
@staticmethod
def _get_idna_encoded_host(host: str) -> str:
    """Codifica un host Unicode a IDNA.

    Args:
        host: Hostname con caracteres no-ASCII.

    Returns:
        str: Host codificado con idna.encode(uts46=True).

    Raises:
        UnicodeError: Si idna lanza IDNAError (label inválido).
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `host` | `str` | Sí | Host a codificar. |

| Excepción | Condición |
| :--- | :--- |
| `UnicodeError` | `idna.IDNAError` durante la codificación. |

#### 2.4.7 `PreparedRequest.prepare_url()`

```python
def prepare_url(self, url: _t.UriType, params: _t.ParamsType) -> None:
    """Prepara la URL HTTP dada.

    Args:
        url: URL a preparar. Acepta str o bytes. Se hace lstrip y requote.
        params: Parámetros a codificar y añadir al query string.

    Returns:
        None

    Raises:
        InvalidURL: Si parse_url falla o host está vacío o label inválido.
        MissingSchema: Si no hay esquema en la URL.
        UnicodeError: Propagado si el host IDNA es inválido (convertido a InvalidURL).
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `url` | `_t.UriType` | Sí | URL base. |
| `params` | `_t.ParamsType` | Sí | Parámetros query. |

| Excepción | Condición |
| :--- | :--- |
| `InvalidURL` | `LocationParseError`, host vacío, host `*.` o label IDNA inválido. |
| `MissingSchema` | URL sin esquema. |

#### 2.4.8 `PreparedRequest.prepare_headers()`

```python
def prepare_headers(self, headers: Mapping[str, str | bytes] | None) -> None:
    """Prepara los headers HTTP dados.

    Args:
        headers: Mapeo de headers.

    Returns:
        None

    Raises:
        InvalidHeader: Si check_header_validity detecta header inválido (propagado).
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `headers` | `Mapping[str, str \| bytes] \| None` | No | Headers a preparar. |

#### 2.4.9 `PreparedRequest.prepare_body()`

```python
def prepare_body(self, data: _t.DataType, files: _t.FilesType, json: _t.JsonType = None) -> None:
    """Prepara el body HTTP dado.

    Maneja 3 casos: JSON, body streameable (Iterable/file-like) y
    body raw/multipart.

    Args:
        data: Datos del body. Puede ser Iterable, file-like, dict, str, bytes.
        files: Archivos para multipart.
        json: Objeto JSON a serializar.

    Returns:
        None

    Raises:
        InvalidJSONError: Si json.dumps falla con ValueError.
        NotImplementedError: Si se intenta enviar body streameado y files simultáneamente.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `data` | `_t.DataType` | Sí | Datos del body. |
| `files` | `_t.FilesType` | Sí | Archivos. |
| `json` | `_t.JsonType` | No | Payload JSON. |

| Excepción | Condición |
| :--- | :--- |
| `InvalidJSONError` | `json.dumps` lanza `ValueError` (ej. NaN con `allow_nan=False`). |
| `NotImplementedError` | `files` y body streameado son mutuamente excluyentes. |

#### 2.4.10 `PreparedRequest.prepare_content_length()`

```python
def prepare_content_length(self, body: _t.BodyType) -> None:
    """Prepara el header Content-Length basado en el método y body.

    Args:
        body: Cuerpo de la petición.

    Returns:
        None
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `body` | `_t.BodyType` | Sí | Body del que calcular longitud vía `super_len`. |

Comportamiento: Si `body` no es None y `super_len(body)` > 0, setea `Content-Length`. Si `body` es None y método no es `GET`/`HEAD` y no hay `Content-Length`, setea `0`.

#### 2.4.11 `PreparedRequest.prepare_auth()`

```python
def prepare_auth(self, auth: _t.AuthType, url: _t.UriType = "") -> None:
    """Prepara los datos de autenticación HTTP.

    Si auth es None, extrae credenciales de la URL. Si es tupla de 2 elementos,
    usa HTTPBasicAuth. Caso contrario, lo trata como callable.

    Args:
        auth: Manejador de autenticación o tupla (user, pass).
        url: URL usada para extraer auth si es necesario (default "").

    Returns:
        None
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `auth` | `_t.AuthType` | Sí | Auth handler. |
| `url` | `_t.UriType` | No | URL para fallback. Default `""`. |

#### 2.4.12 `PreparedRequest.prepare_cookies()`

```python
def prepare_cookies(self, cookies: RequestsCookieJar | CookieJar | dict[str, str] | None) -> None:
    """Prepara los datos de cookies HTTP.

    Genera el header Cookie usando cookielib. Solo tiene efecto la primera vez,
    a menos que se elimine el header Cookie previamente.

    Args:
        cookies: CookieJar o dict de cookies.

    Returns:
        None
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `cookies` | `RequestsCookieJar \| CookieJar \| dict \| None` | Sí | Cookies a preparar. |

#### 2.4.13 `PreparedRequest.prepare_hooks()`

```python
def prepare_hooks(self, hooks: _t.HooksInputType | None) -> None:
    """Prepara los hooks dados.

    Args:
        hooks: Diccionario de hooks por evento. Si es None o falsy, se usa {}.

    Returns:
        None
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `hooks` | `_t.HooksInputType \| None` | No | Hooks a registrar. |

---

### 2.5 Clase `Response`

Objeto que contiene la respuesta del servidor a una petición HTTP.

#### 2.5.1 `Response.__init__()`

```python
def __init__(self) -> None:
    """Inicializa una Response vacía.

    Attributes:
        _content: Bytes del contenido o False si no se ha leído.
        _content_consumed: Flag de consumo.
        _next: PreparedRequest para la siguiente redirección.
        status_code: Código de estado HTTP.
        headers: CaseInsensitiveDict de headers.
        raw: Objeto file-like de la respuesta.
        url: URL final.
        encoding: Codificación para decodificar text.
        history: Lista de Responses de redirecciones.
        reason: Razón textual del estado.
        cookies: CookieJar.
        elapsed: Timedelta del tiempo transcurrido.
        request: PreparedRequest asociada.
    """
```

#### 2.5.2 `Response.__enter__()`

```python
def __enter__(self) -> Self:
    """Soporte para context manager.

    Returns:
        Self: La propia instancia de Response.
    """
```

#### 2.5.3 `Response.__exit__()`

```python
def __exit__(self, *args: Any) -> None:
    """Sale del context manager y cierra la conexión.

    Args:
        *args: Argumentos del context manager (exc_type, exc_val, exc_tb).

    Returns:
        None
    """
```

#### 2.5.4 `Response.__getstate__()`

```python
def __getstate__(self) -> dict[str, Any]:
    """Prepara el estado para pickle.

    Consume el contenido si no ha sido consumido.

    Returns:
        dict[str, Any]: Diccionario con atributos definidos en __attrs__.
    """
```

#### 2.5.5 `Response.__setstate__()`

```python
def __setstate__(self, state: dict[str, Any]) -> None:
    """Restaura el estado desde pickle.

    Args:
        state: Diccionario de estado.

    Returns:
        None
    """
```

Efecto: Restaura atributos y setea `_content_consumed=True` y `raw=None`.

#### 2.5.6 `Response.__repr__()`

```python
def __repr__(self) -> str:
    """Retorna representación textual.

    Returns:
        str: "<Response [status_code]>".
    """
```

#### 2.5.7 `Response.__bool__()`

```python
def __bool__(self) -> bool:
    """Retorna True si status_code < 400.

    Returns:
        bool: True si ok, False si error cliente/servidor.
    """
```

#### 2.5.8 `Response.__nonzero__()`

```python
def __nonzero__(self) -> bool:
    """Compatibilidad Python 2 para __bool__.

    Returns:
        bool: Igual que __bool__.
    """
```

#### 2.5.9 `Response.__iter__()`

```python
def __iter__(self) -> Iterator[bytes]:
    """Permite usar la response como iterador.

    Returns:
        Iterator[bytes]: Iterador sobre iter_content(128).
    """
```

#### 2.5.10 `Response.ok` (property)

```python
@property
def ok(self) -> bool:
    """Retorna True si status_code < 400.

    Returns:
        bool: True si no hay HTTPError al llamar raise_for_status().
    """
```

#### 2.5.11 `Response.is_redirect` (property)

```python
@property
def is_redirect(self) -> bool:
    """Indica si es una redirección procesable automáticamente.

    Returns:
        bool: True si tiene header 'location' y status_code en REDIRECT_STATI
            (301, 302, 303, 307, 308).
    """
```

#### 2.5.12 `Response.is_permanent_redirect` (property)

```python
@property
def is_permanent_redirect(self) -> bool:
    """Indica si es una redirección permanente.

    Returns:
        bool: True si tiene 'location' y status_code es 301 o 308.
    """
```

#### 2.5.13 `Response.next` (property)

```python
@property
def next(self) -> PreparedRequest | None:
    """Retorna la PreparedRequest para la siguiente petición en la cadena de redirección.

    Returns:
        PreparedRequest | None: Siguiente petición o None.
    """
```

#### 2.5.14 `Response.apparent_encoding` (property)

```python
@property
def apparent_encoding(self) -> str | None:
    """La codificación aparente detectada por chardet/charset_normalizer.

    Returns:
        str | None: Encoding detectado o "utf-8" si no hay librería disponible.
    """
```

#### 2.5.15 `Response.iter_content()`

```python
@overload
def iter_content(self, chunk_size: int | None = 1, decode_unicode: Literal[False] = False) -> Iterator[bytes]: ...
@overload
def iter_content(self, chunk_size: int | None = 1, *, decode_unicode: Literal[True]) -> Iterator[str | bytes]: ...
def iter_content(self, chunk_size: int | None = 1, decode_unicode: bool = False) -> Iterator[str | bytes]:
    """Itera sobre los datos de la respuesta.

    Evita leer todo el contenido en memoria cuando stream=True.

    Args:
        chunk_size: Número de bytes a leer en memoria. None lee según llegan los chunks.
            Debe ser int o None.
        decode_unicode: Si True, decodifica usando la codificación de la respuesta.

    Returns:
        Iterator[str | bytes]: Iterador de chunks.

    Raises:
        StreamConsumedError: Si _content_consumed es True y _content es bool.
        TypeError: Si chunk_size no es int ni None.
        ChunkedEncodingError: Si urllib3 lanza ProtocolError.
        ContentDecodingError: Si urllib3 lanza DecodeError.
        ConnectionError: Si urllib3 lanza ReadTimeoutError.
        RequestsSSLError: Si urllib3 lanza SSLError.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `chunk_size` | `int \| None` | No | Tamaño de chunk. Default `1`. |
| `decode_unicode` | `bool` | No | Decodificar a unicode. Default `False`. |

##### 2.5.15.1 `Response.iter_content.<locals>.generate()` (función anidada)

```python
def generate() -> Generator[bytes, None, None]:
    """Generador interno que produce chunks desde self.raw.

    Yields:
        bytes: Chunk de datos leídos de self.raw.stream() o self.raw.read().

    Raises:
        ChunkedEncodingError: En ProtocolError.
        ContentDecodingError: En DecodeError.
        ConnectionError: En ReadTimeoutError.
        RequestsSSLError: En SSLError.
    """
```

Función local definida dentro de `iter_content`. Maneja dos ramas: si `self.raw` tiene método `stream` (urllib3) itera con `raw.stream(chunk_size, decode_content=True)`, sino lee con `raw.read(chunk_size)` en bucle. Al finalizar setea `self._content_consumed = True`.

#### 2.5.16 `Response.iter_lines()`

```python
@overload
def iter_lines(self, chunk_size: int = ITER_CHUNK_SIZE, decode_unicode: Literal[False] = False, delimiter: bytes | None = None) -> Iterator[bytes]: ...
@overload
def iter_lines(self, chunk_size: int = ITER_CHUNK_SIZE, *, decode_unicode: Literal[True], delimiter: str | bytes | None = None) -> Iterator[str | bytes]: ...
def iter_lines(self, chunk_size: int = ITER_CHUNK_SIZE, decode_unicode: bool = False, delimiter: str | bytes | None = None) -> Iterator[str | bytes]:
    """Itera sobre los datos de la respuesta, una línea a la vez.

    Args:
        chunk_size: Tamaño de chunk para iter_content. Default 512.
        decode_unicode: Si True, decodifica el contenido.
        delimiter: Delimitador para dividir líneas. Si es None, usa splitlines().

    Returns:
        Iterator[str | bytes]: Iterador de líneas. No es reentrante.

    Yields:
        str | bytes: Cada línea de la respuesta.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `chunk_size` | `int` | No | Tamaño de chunk. Default `512`. |
| `decode_unicode` | `bool` | No | Decodificar unicode. Default `False`. |
| `delimiter` | `str \| bytes \| None` | No | Delimitador. Default `None`. |

#### 2.5.17 `Response.content` (property)

```python
@property
def content(self) -> bytes:
    """Contenido de la respuesta, en bytes.

    Returns:
        bytes: Contenido completo. Vacío si status_code == 0 o raw is None.

    Raises:
        RuntimeError: Si el contenido ya fue consumido (_content_consumed True y _content False).
    """
```

#### 2.5.18 `Response.text` (property)

```python
@property
def text(self) -> str:
    """Contenido de la respuesta, en unicode.

    Si encoding es None, usa apparent_encoding.

    Returns:
        str: Contenido decodificado con errors="replace". Retorna "" si no hay contenido.

    Raises:
        LookupError: Capturado internamente, hace fallback a decode sin encoding.
        TypeError: Capturado internamente si encoding es None.
    """
```

#### 2.5.19 `Response.json()`

```python
def json(self, **kwargs: Any) -> Any:
    """Decodifica el cuerpo JSON de la respuesta como objeto Python.

    Args:
        **kwargs: Argumentos opcionales pasados a json.loads.

    Returns:
        Any: Objeto Python (dict, list, etc.) resultante del JSON.

    Raises:
        RequestsJSONDecodeError: Si el cuerpo no contiene JSON válido.
            Encapsula JSONDecodeError original.
    """
```

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `**kwargs` | `Any` | No | Argumentos para `complexjson.loads`. |

Comportamiento: Si no hay `encoding` y `content` > 3 bytes, intenta detectar UTF-8/16/32 con `guess_json_utf` y decodificar directamente. Si falla, usa `self.text`.

#### 2.5.20 `Response.links` (property)

```python
@property
def links(self) -> dict[str, dict[str, str]]:
    """Retorna los header links parseados de la respuesta.

    Returns:
        dict[str, dict[str, str]]: Diccionario indexado por rel o url,
            con cada link parseado por parse_header_links.
    """
```

#### 2.5.21 `Response.raise_for_status()`

```python
def raise_for_status(self) -> None:
    """Lanza HTTPError si ocurrió un error HTTP.

    Returns:
        None

    Raises:
        HTTPError: Si status_code está entre 400-499 (Client Error) o
            500-599 (Server Error). Incluye reason y url en el mensaje.
    """
```

| Excepción | Condición |
| :--- | :--- |
| `HTTPError` | `400 <= status_code < 500` o `500 <= status_code < 600` |

#### 2.5.22 `Response.close()`

```python
def close(self) -> None:
    """Libera la conexión de vuelta al pool.

    Cierra raw si el contenido no ha sido consumido y llama a
    raw.release_conn() si existe.

    Returns:
        None
    """
```

---

## 3. Ejemplo de Uso

```python
import requests
from requests import Request, Session
from requests.models import PreparedRequest, Response

# 1. Uso básico con Request y PreparedRequest
req = Request(
    method='GET',
    url='https://httpbin.org/get',
    params={'q': 'requests', 'lang': 'es'},
    headers={'Accept': 'application/json'},
    hooks={'response': lambda r, *args, **kwargs: print(f"Hook: {r.status_code}")}
)
prepared: PreparedRequest = req.prepare()
print(prepared.path_url)  # /get?q=requests&lang=es
print(prepared.url)       # https://httpbin.org/get?q=requests&lang=es

# Envío con Session (permite persistir cookies, auth y usar hooks)
with Session() as s:
    # 2. POST con JSON y manejo de Response
    resp: Response = s.send(prepared)
    resp.raise_for_status()  # Lanza HTTPError si 4xx/5xx
    print(resp.text[:200])
    print(resp.json())
    print(resp.links)
    print(resp.is_redirect, resp.ok)

    # 3. POST multipart con archivos
    files = {'file': ('report.csv', open('report.csv', 'rb'), 'text/csv')}
    r2 = s.post('https://httpbin.org/post', data={'user': 'admin'}, files=files)
    print(r2.status_code)

    # 4. Streaming y consumo iterativo
    with s.get('https://httpbin.org/stream/5', stream=True) as r_stream:
        for chunk in r_stream.iter_content(chunk_size=1024, decode_unicode=False):
            print(f"Chunk {len(chunk)} bytes")
        for line in r_stream.iter_lines(decode_unicode=True):
            print(line)

    # 5. Manejo de hooks y autenticación
    def log_hook(response, *args, **kwargs):
        print(f"URL final: {response.url} - Elapsed: {response.elapsed}")
        return response

    req_auth = Request('GET', 'https://httpbin.org/basic-auth/user/pass', auth=('user', 'pass'))
    req_auth.register_hook('response', log_hook)
    prepped_auth = req_auth.prepare()
    resp_auth = s.send(prepped_auth)
    print(resp_auth.ok)  # True si 200

    # 6. Uso como context manager y cierre explícito
    resp2 = s.get('https://httpbin.org/get')
    try:
        print(resp2.content)
    finally:
        resp2.close()
```
