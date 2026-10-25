# Documentación Técnica - `requests.models`

**Módulo:** `requests.models`  
**Librería:** Requests - HTTP for Humans  
**Descripción:** Módulo central que contiene los objetos primarios que potencian la librería Requests. Gestiona el ciclo de vida completo de una petición HTTP: desde su creación, preparación y codificación, hasta el procesamiento de la respuesta del servidor.

---

### 1. Descripción General

El archivo `requests.models` es el núcleo de la librería `Requests`. Su responsabilidad es abstraer la complejidad del protocolo HTTP y de `urllib3` para ofrecer una API orientada a objetos, simple y pythonica.

El flujo principal que implementa es:

`Request` (objeto creado por el usuario) -> `PreparedRequest` (petición serializada y lista para ser enviada por el socket) -> `Response` (respuesta del servidor procesada y enriquecida)

El módulo se encarga de:
* Normalización y validación de URLs, headers y parámetros.
* Codificación de cuerpos `application/x-www-form-urlencoded` y `multipart/form-data`.
* Gestión de autenticación, cookies y hooks.
* Manejo de internacionalización de dominios [IDNA].
* Consumo, decodificación y streaming de la respuesta.

### 2. Dependencias Principales

| Dependencia | Uso |
| :--- | :--- |
| `urllib3` | Codificación multipart, parsing de URLs y manejo de excepciones de bajo nivel (`ProtocolError`, `SSLError`) |
| `encodings.idna` | Soporte para dominios internacionalizados |
| `chardet` / `charset_normalizer` | Detección automática de encoding en `Response.apparent_encoding` |
| `http.cookiejar` | Gestión de cookies |
| Módulos internos `requests.*` | `auth`, `cookies`, `exceptions`, `structures.CaseInsensitiveDict`, `utils`, `hooks` |

### 3. Constantes Globales

| Constante | Valor | Descripción |
| :--- | :--- | :--- |
| `REDIRECT_STATI` | `(301, 302, 303, 307, 308)` | Códigos de estado que indican una redirección procesable automáticamente. |
| `DEFAULT_REDIRECT_LIMIT` | `30` | Límite máximo de redirecciones para evitar bucles infinitos. |
| `CONTENT_CHUNK_SIZE` | `10 * 1024` | Tamaño de bloque en bytes para consumir `Response.content`. |
| `ITER_CHUNK_SIZE` | `512` | Tamaño de bloque por defecto para `Response.iter_lines()`. |

### 4. Componentes Principales

#### 4.1. `RequestEncodingMixin`
Clase Mixin que proporciona utilidades de codificación para las peticiones.

* **`path_url` (property):** Construye el `path` + `query string` a partir de `self.url`. Devuelve `/` si no hay path.
* **`_encode_params(data)` (staticmethod):** Codifica parámetros. Acepta `str`, `bytes`, `dict` o `list[tuple]`. Si es un diccionario/lista, lo convierte a `application/x-www-form-urlencoded` usando `urlencode`.
* **`_encode_files(files, data)` (staticmethod):** Construye el cuerpo para peticiones `multipart/form-data`. Utiliza `urllib3.filepost.encode_multipart_formdata` y soporta tuplas de 2, 3 y 4 elementos: `(filename, fileobj, content_type, custom_headers)`.

#### 4.2. `RequestHooksMixin`
Clase Mixin para el sistema de eventos [hooks] de Requests.

* **`register_hook(event, hook)`:** Registra una función callback para un evento. Valida que el evento exista.
* **`deregister_hook(event, hook)`:** Elimina un hook previamente registrado. Retorna `True` si existía, `False` si no.

#### 4.3. `Request` - Clase de Alto Nivel
> Hereda de `RequestHooksMixin`

Es el objeto que crea el usuario final. Es una representación abstracta y no preparada de la petición.

**Atributos principales:** `method`, `url`, `headers`, `files`, `data`, `json`, `params`, `auth`, `cookies`, `hooks`.

**Métodos:**
* `__init__(...)`: Inicializa la petición con valores por defecto vacíos y registra los hooks.
* `prepare() -> PreparedRequest`: Orquesta la creación de un objeto `PreparedRequest` llamando a `PreparedRequest.prepare()` con todos sus atributos. Es el puente entre la API de usuario y la capa de transporte.

#### 4.4. `PreparedRequest` - Clase de Bajo Nivel
> Hereda de `RequestEncodingMixin`, `RequestHooksMixin`

Es la representación totalmente mutable y serializada de la petición. Contiene los bytes exactos que se enviarán al servidor. No debe instanciarse manualmente.

**Atributos principales:** `method`, `url`, `headers: CaseInsensitiveDict`, `body`, `_cookies`, `_body_position`.

**Métodos de Preparación (llamados en orden por `prepare()`):**

| Método | Función |
| :--- | :--- |
| `prepare_method(method)` | Normaliza el método a mayúsculas. |
| `prepare_url(url, params)` | Valida, limpia y reconstruye la URL. Maneja codificación IDNA para dominios unicode, valida esquema y host, y codifica los `params`. Lanza `MissingSchema` e `InvalidURL`. |
| `prepare_headers(headers)` | Valida los headers con `check_header_validity` y los almacena en un `CaseInsensitiveDict`. |
| `prepare_cookies(cookies)` | Genera el header `Cookie` a partir de un `dict` o `CookieJar`. |
| `prepare_body(data, files, json)` | Lógica central del cuerpo. Prioriza `json` > `data` streameable > `files` multipart > `data` form-encoded. Gestiona `Content-Length` vs `Transfer-Encoding: chunked` y `Content-Type`. |
| `prepare_content_length(body)` | Calcula y asigna el header `Content-Length` usando `super_len`. |
| `prepare_auth(auth, url)` | Resuelve la autenticación. Extrae credenciales de la URL si no se provee `auth` y delega a `HTTPBasicAuth` o a un handler custom. |
| `prepare_hooks(hooks)` | Registra los hooks finales. |
| `copy() -> PreparedRequest` | Crea una copia superficial de la petición, incluyendo copia del `CookieJar`. |
| `_get_idna_encoded_host(host)` | Codifica hosts unicode a formato IDNA compatible con ASCII. |

#### 4.5. `Response` - Objeto de Respuesta
Contiene la respuesta del servidor a una petición HTTP. Implementa el protocolo de context manager y es iterable.

**Atributos principales:** `status_code`, `headers`, `raw`, `url`, `encoding`, `history`, `reason`, `cookies`, `elapsed`, `request`, `_content`, `_content_consumed`.

**Propiedades:**

| Propiedad | Descripción |
| :--- | :--- |
| `ok` | `True` si `status_code < 400`. |
| `is_redirect` | `True` si es una redirección válida con header `Location`. |
| `is_permanent_redirect` | `True` si es `301` o `308`. |
| `next` | Retorna el `PreparedRequest` para la siguiente redirección en la cadena. |
| `apparent_encoding` | Encoding detectado por `chardet`/`charset_normalizer`. |
| `content: bytes` | Contenido de la respuesta en bytes. Consume el stream una sola vez. |
| `text: str` | Contenido decodificado a `str` usando `encoding` o `apparent_encoding`. |
| `links: dict` | Parsea el header `Link` para paginación. |

**Métodos principales:**

* `iter_content(chunk_size, decode_unicode) -> Iterator[bytes|str]`: Itera sobre la respuesta en bloques sin cargarla toda en memoria. Ideal para `stream=True`. Maneja `ChunkedEncodingError` y `ContentDecodingError`.
* `iter_lines(chunk_size, decode_unicode, delimiter) -> Iterator[bytes|str]`: Itera línea por línea.
* `json(**kwargs) -> Any`: Decodifica el cuerpo como JSON. Detecta automáticamente UTF-8/16/32 si no hay encoding. Lanza `RequestsJSONDecodeError`.
* `raise_for_status()`: Lanza `HTTPError` si el código es 4xx o 5xx.
* `close()`: Libera la conexión de vuelta al pool de `urllib3`. Cierra `raw`.

### 5. Flujo de Ejecución

```
Usuario: requests.Request('GET', 'https://api.com', params={...})
    |
    v
Request.prepare() -> Crea PreparedRequest()
    |
    v
PreparedRequest.prepare() -> Ejecuta en secuencia:
  prepare_method -> prepare_url -> prepare_headers -> prepare_cookies -> prepare_body -> prepare_auth -> prepare_hooks
    |
    v
Session.send(PreparedRequest) -> Envía por red vía urllib3
    |
    v
Response -> El usuario consume .content, .text, .json() o .iter_content()
```

Esta arquitectura separa la **definición** de la petición `Request` de su **serialización** `PreparedRequest`, permitiendo inspeccionar, modificar y reutilizar peticiones antes de enviarlas, y facilita el manejo de redirecciones y reintentos.

