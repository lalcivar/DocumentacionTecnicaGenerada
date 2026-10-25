# Documentación Técnica - Módulo de Tests `requests.utils`

## 1. Resumen Ejecutivo

Este módulo corresponde a la suite de tests unitarios para `requests.utils` y `requests._internal_utils` de la librería `requests`. Valida de forma exhaustiva el comportamiento de utilidades críticas para el manejo de HTTP: cálculo de longitud de streams (`super_len`), gestión de proxies y `no_proxy`, parsing de headers, detección de encoding, manejo de cookies, autenticación `netrc` y normalización de URLs. Está implementado con `pytest` y utiliza fixtures, parametrización y mocks para cubrir casos de borde, errores de I/O y compatibilidad específica de plataforma (Windows Registry).

## 2. Especificación de Funciones y Métodos

> Todas las funciones siguen el estándar **PEP 257 / Google Style Docstring**. Cada entrada mantiene el nombre exacto del código fuente y documenta funciones principales, auxiliares, métodos de clase y funciones/clases anidadas locales.

### 2.1 Clase `TestSuperLen`

Suite para validar `requests.utils.super_len`.

#### `TestSuperLen.test_io_streams(self, stream, value)`
```python
def test_io_streams(self, stream, value):
    """Verifica el cálculo de longitud para diferentes tipos de streams de I/O.

    Args:
        stream (type): Clase de stream a instanciar (StringIO.StringIO, BytesIO, cStringIO).
        value (str|bytes): Contenido inicial del stream.

    Returns:
        None

    Raises:
        AssertionError: Si super_len no retorna 0 para stream vacío o 4 para stream con "Test".
    """
```
| Parámetro | Tipo | Requerido | Descripción |
|---|---|---|---|
| `stream` | `type` | Sí | Factory del stream. Parametrizado vía `@pytest.mark.parametrize` |
| `value` | `str` \| `bytes` | Sí | Datos para inicializar el stream |
| `self` | `TestSuperLen` | Sí | Instancia del test |

| Retorna | Tipo | Descripción |
|---|---|---|
| `None` | `NoneType` | Test de aserción, no retorna valor |

#### `TestSuperLen.test_super_len_correctly_calculates_len_of_partially_read_file(self)`
```python
def test_super_len_correctly_calculates_len_of_partially_read_file(self):
    """Asegura que objetos file-like parcialmente consumidos se manejen correctamente.

    Args:
        self (TestSuperLen): Instancia del test.

    Returns:
        None
    """
```
| Parámetro | Tipo | Requerido | Descripción |
|---|---|---|---|
| `self` | `TestSuperLen` | Sí | Instancia |

#### `TestSuperLen.test_super_len_handles_files_raising_weird_errors_in_tell(self, error)`
```python
def test_super_len_handles_files_raising_weird_errors_in_tell(self, error):
    """Valida que si tell() lanza IOError/OSError se asuma cursor en posición 0.

    Args:
        self (TestSuperLen): Instancia.
        error (type): Excepción a simular (IOError, OSError). Parametrizado.

    Returns:
        None
    """
```
**Clase anidada `BoomFile` (definida dentro de `test_super_len_handles_files_raising_weird_errors_in_tell`)**

##### `BoomFile.__len__(self)`
```python
def __len__(self):
    """Retorna longitud fija para simular objeto con __len__.

    Returns:
        int: Valor 5.
    """
```
| Retorna | Tipo | Descripción |
|---|---|---|
| `5` | `int` | Longitud simulada |

##### `BoomFile.tell(self)`
```python
def tell(self):
    """Simula fallo en tell() lanzando la excepción parametrizada.

    Raises:
        IOError | OSError: Siempre lanza `error()`.
    """
```
| Lanza | Condición |
|---|---|
| `IOError` / `OSError` | Invocación incondicional para probar fallback de `super_len` |

#### `TestSuperLen.test_super_len_tell_ioerror(self, error)`
```python
def test_super_len_tell_ioerror(self, error):
    """Asegura que super_len no falle si tell() lanza IOError sin __len__ disponible.

    Args:
        self (TestSuperLen): Instancia.
        error (type): IOError u OSError parametrizado.

    Returns:
        None
    """
```
**Clase anidada `NoLenBoomFile`**

##### `NoLenBoomFile.tell(self)`
```python
def tell(self):
    """Lanza error simulado en tell().

    Raises:
        IOError | OSError: Excepción parametrizada.
    """
```

##### `NoLenBoomFile.seek(self, offset, whence)`
```python
def seek(self, offset, whence):
    """Implementación no-op de seek para cumplir interfaz file-like.

    Args:
        offset (int): Offset.
        whence (int): Modo de posicionamiento.

    Returns:
        None
    """
```
| Parámetro | Tipo | Requerido | Descripción |
|---|---|---|---|
| `offset` | `int` | Sí | Desplazamiento |
| `whence` | `int` | Sí | Referencia (0,1,2) |

#### `TestSuperLen.test_string(self)`
```python
def test_string(self):
    """Valida super_len sobre str plano.

    Returns:
        None
    Raises:
        AssertionError: Si super_len("Test") != 4
    """
```

#### `TestSuperLen.test_file(self, tmpdir, mode, warnings_num, recwarn)`
```python
def test_file(self, tmpdir, mode, warnings_num, recwarn):
    """Valida super_len sobre descriptores de archivo reales y conteo de warnings.

    Args:
        tmpdir (py.path.local): Fixture pytest para directorio temporal.
        mode (str): Modo de apertura ("r" o "rb"). Parametrizado.
        warnings_num (int): Número esperado de warnings (1 para "r", 0 para "rb").
        recwarn (WarningsRecorder): Fixture pytest para capturar warnings.

    Returns:
        None
    """
```
| Parámetro | Tipo | Requerido | Descripción |
|---|---|---|---|
| `tmpdir` | `py.path.local` | Sí | Directorio temporal |
| `mode` | `str` | Sí | Modo apertura |
| `warnings_num` | `int` | Sí | Warnings esperados |
| `recwarn` | `WarningsRecorder` | Sí | Capturador de warnings |

#### `TestSuperLen.test_tarfile_member(self, tmpdir)`
```python
def test_tarfile_member(self, tmpdir):
    """Valida super_len sobre miembros extraídos de tarfile.

    Args:
        tmpdir (py.path.local): Directorio temporal.

    Returns:
        None
    """
```

#### `TestSuperLen.test_super_len_with__len__(self)`
```python
def test_super_len_with__len__(self):
    """Valida super_len usando __len__ de una lista.

    Returns:
        None
    """
```

#### `TestSuperLen.test_super_len_with_no__len__(self)`
```python
def test_super_len_with_no__len__(self):
    """Valida fallback a atributo .len cuando __len__ no existe.

    Returns:
        None
    """
```
**Clase anidada `LenFile`**

##### `LenFile.__init__(self)`
```python
def __init__(self):
    """Inicializa objeto con atributo len=5 sin método __len__.

    Args:
        self (LenFile): Instancia.

    Returns:
        None
    """
```

#### `TestSuperLen.test_super_len_with_tell(self)`
```python
def test_super_len_with_tell(self):
    """Valida cálculo basado en tell()/seek() para StringIO.

    Returns:
        None
    """
```

#### `TestSuperLen.test_super_len_with_fileno(self)`
```python
def test_super_len_with_fileno(self):
    """Valida super_len usando fileno() sobre archivo real (__file__).

    Returns:
        None
    """
```

#### `TestSuperLen.test_super_len_with_no_matches(self)`
```python
def test_super_len_with_no_matches(self):
    """Asegura que objetos sin métodos de longitud retornen 0.

    Returns:
        None
    """
```

### 2.2 Clase `TestGetNetrcAuth`

#### `TestGetNetrcAuth.test_works(self, tmp_path, monkeypatch)`
```python
def test_works(self, tmp_path, monkeypatch):
    """Verifica obtención exitosa de credenciales desde .netrc.

    Args:
        tmp_path (pathlib.Path): Fixture directorio temporal.
        monkeypatch (MonkeyPatch): Fixture para setear env NETRC.

    Returns:
        None
    """
```

#### `TestGetNetrcAuth.test_not_vulnerable_to_bad_url_parsing(self, tmp_path, monkeypatch)`
```python
def test_not_vulnerable_to_bad_url_parsing(self, tmp_path, monkeypatch):
    """Valida que URLs maliciosas con @ no extraigan credenciales incorrectas.

    Args:
        tmp_path (pathlib.Path): Path temporal.
        monkeypatch (MonkeyPatch): Mock de entorno.

    Returns:
        None
    """
```

#### `TestGetNetrcAuth.test_empty_default_credentials_ignored(self, tmp_path, monkeypatch)`
```python
def test_empty_default_credentials_ignored(self, tmp_path, monkeypatch):
    """Verifica que credenciales default vacías en .netrc sean ignoradas.

    Args:
        tmp_path (pathlib.Path): Path temporal.
        monkeypatch (MonkeyPatch): Mock de entorno.

    Returns:
        None
    """
```

### 2.3 Clase `TestToKeyValList`

#### `TestToKeyValList.test_valid(self, value, expected)`
```python
def test_valid(self, value, expected):
    """Valida conversión de dict/tupla/lista a lista de pares clave-valor.

    Args:
        value (list|tuple|dict|None): Entrada a convertir.
        expected (list|None): Salida esperada.

    Returns:
        None
    """
```

#### `TestToKeyValList.test_invalid(self)`
```python
def test_invalid(self):
    """Verifica que entrada string lance ValueError.

    Raises:
        ValueError: Cuando se pasa "string" a to_key_val_list.
    Returns:
        None
    """
```
| Lanza | Condición |
|---|---|
| `ValueError` | `to_key_val_list("string")` |

### 2.4 Clase `TestUnquoteHeaderValue`

#### `TestUnquoteHeaderValue.test_valid(self, value, expected)`
```python
def test_valid(self, value, expected):
    """Valida unquoting de valores de header.

    Args:
        value (str|None): Valor con comillas y escapes.
        expected (str|None): Valor esperado sin comillas.

    Returns:
        None
    """
```

#### `TestUnquoteHeaderValue.test_is_filename(self)`
```python
def test_is_filename(self):
    """Valida rama is_filename=True preserva escapes.

    Returns:
        None
    """
```

### 2.5 Clase `TestGetEnvironProxies`

#### `TestGetEnvironProxies.no_proxy(self, request, monkeypatch)`
```python
@pytest.fixture(autouse=True, params=["no_proxy", "NO_PROXY"])
def no_proxy(self, request, monkeypatch):
    """Fixture autouse que setea no_proxy con rangos CIDR y hostnames.

    Args:
        request (FixtureRequest): Request con param "no_proxy" o "NO_PROXY".
        monkeypatch (MonkeyPatch): Para setear env.

    Returns:
        None
    """
```
| Parámetro | Tipo | Requerido | Descripción |
|---|---|---|---|
| `request` | `FixtureRequest` | Sí | Contiene `request.param` |
| `monkeypatch` | `MonkeyPatch` | Sí | Mock entorno |

#### `TestGetEnvironProxies.test_bypass(self, url)`
```python
def test_bypass(self, url):
    """Verifica que URLs en no_proxy retornen dict vacío (bypass).

    Args:
        url (str): URL parametrizada dentro de rango no_proxy.

    Returns:
        None
    """
```

#### `TestGetEnvironProxies.test_not_bypass(self, url)`
```python
def test_not_bypass(self, url):
    """Verifica que URLs fuera de no_proxy no hagan bypass.

    Args:
        url (str): URL fuera de rango.

    Returns:
        None
    """
```

#### `TestGetEnvironProxies.test_bypass_no_proxy_keyword(self, url)`
```python
def test_bypass_no_proxy_keyword(self, url):
    """Valida bypass usando argumento no_proxy explícito.

    Args:
        url (str): URL parametrizada.

    Returns:
        None
    """
```

#### `TestGetEnvironProxies.test_not_bypass_no_proxy_keyword(self, url, monkeypatch)`
```python
def test_not_bypass_no_proxy_keyword(self, url, monkeypatch):
    """Valida que argumento no_proxy sobrescriba variable de entorno.

    Args:
        url (str): URL parametrizada.
        monkeypatch (MonkeyPatch): Setea http_proxy.

    Returns:
        None
    """
```

### 2.6 Clase `TestIsIPv4Address`

#### `TestIsIPv4Address.test_valid(self)`
```python
def test_valid(self):
    """Valida que 8.8.8.8 sea reconocida como IPv4 válida.

    Returns:
        None
    """
```

#### `TestIsIPv4Address.test_invalid(self, value)`
```python
def test_invalid(self, value):
    """Valida rechazo de IPv4 inválidas.

    Args:
        value (str): "8.8.8.8.8" o "localhost.localdomain".

    Returns:
        None
    """
```

### 2.7 Clase `TestIsValidCIDR`

#### `TestIsValidCIDR.test_valid(self)`
```python
def test_valid(self):
    """Valida CIDR 192.168.1.0/24 como válido.

    Returns:
        None
    """
```

#### `TestIsValidCIDR.test_invalid(self, value)`
```python
def test_invalid(self, value):
    """Valida rechazo de CIDR malformados.

    Args:
        value (str): CIDR inválido parametrizado.

    Returns:
        None
    """
```

### 2.8 Clase `TestAddressInNetwork`

#### `TestAddressInNetwork.test_valid(self)`
```python
def test_valid(self):
    """Valida que 192.168.1.1 pertenece a 192.168.1.0/24.

    Returns:
        None
    """
```

#### `TestAddressInNetwork.test_invalid(self)`
```python
def test_invalid(self):
    """Valida que 172.16.0.1 no pertenece a 192.168.1.0/24.

    Returns:
        None
    """
```

### 2.9 Clase `TestGuessFilename`

#### `TestGuessFilename.test_guess_filename_invalid(self, value)`
```python
def test_guess_filename_invalid(self, value):
    """Valida que guess_filename retorne None para entradas inválidas.

    Args:
        value (int|object): Objeto sin name válido o name no str/bytes.

    Returns:
        None
    """
```

#### `TestGuessFilename.test_guess_filename_valid(self, value, expected_type)`
```python
def test_guess_filename_valid(self, value, expected_type):
    """Valida extracción de filename preservando tipo bytes/str.

    Args:
        value (bytes|str): Valor de atributo name.
        expected_type (type): Tipo esperado (compat.bytes o compat.str).

    Returns:
        None
    """
```

### 2.10 Clase `TestExtractZippedPaths`

#### `TestExtractZippedPaths.test_unzipped_paths_unchanged(self, path)`
```python
def test_unzipped_paths_unchanged(self, path):
    """Verifica que paths no zipeados se retornen sin cambios.

    Args:
        path (str): Path parametrizado (/, __file__, etc.).

    Returns:
        None
    """
```

#### `TestExtractZippedPaths.test_zipped_paths_extracted(self, tmpdir)`
```python
def test_zipped_paths_extracted(self, tmpdir):
    """Valida extracción de archivo dentro de zip a path temporal.

    Args:
        tmpdir (py.path.local): Directorio temporal.

    Returns:
        None
    """
```

#### `TestExtractZippedPaths.test_invalid_unc_path(self)`
```python
def test_invalid_unc_path(self):
    """Valida que UNC path inválido se retorne sin cambios.

    Returns:
        None
    """
```

### 2.11 Clase `TestContentEncodingDetection`

#### `TestContentEncodingDetection.test_none(self)`
```python
def test_none(self):
    """Valida que contenido vacío retorne lista vacía de encodings.

    Returns:
        None
    """
```

#### `TestContentEncodingDetection.test_pragmas(self, content)`
```python
def test_pragmas(self, content):
    """Valida detección de charset vía meta charset, http-equiv y xml.

    Args:
        content (str): Fragmento HTML/XML parametrizado.

    Returns:
        None
    """
```

#### `TestContentEncodingDetection.test_precedence(self)`
```python
def test_precedence(self):
    """Valida orden de precedencia HTML5 > HTML4 > XML.

    Returns:
        None
    """
```

### 2.12 Clase `TestGuessJSONUTF`

#### `TestGuessJSONUTF.test_encoded(self, encoding)`
```python
def test_encoded(self, encoding):
    """Valida detección de encoding JSON para múltiples UTF.

    Args:
        encoding (str): Encoding parametrizado (utf-32, utf-8-sig, etc.).

    Returns:
        None
    """
```

#### `TestGuessJSONUTF.test_bad_utf_like_encoding(self)`
```python
def test_bad_utf_like_encoding(self):
    """Valida que bytes nulos retornen None.

    Returns:
        None
    """
```

#### `TestGuessJSONUTF.test_guess_by_bom(self, encoding, expected)`
```python
def test_guess_by_bom(self, encoding, expected):
    """Valida detección vía BOM.

    Args:
        encoding (str): Encoding con BOM (utf-16-be, etc.).
        expected (str): Encoding normalizado (utf-16, utf-32).

    Returns:
        None
    """
```

### 2.13 Funciones Standalone (Nivel Módulo)

#### `test_get_auth_from_url(url, auth)`
```python
def test_get_auth_from_url(url, auth):
    """Valida extracción y unquoting de credenciales desde URL.

    Args:
        url (str): URL con user:pass parametrizada.
        auth (tuple[str,str]): Tupla esperada (user, password).

    Returns:
        None
    """
```
| Parámetro | Tipo | Requerido | Descripción |
|---|---|---|---|
| `url` | `str` | Sí | URL con auth embebida |
| `auth` | `tuple` | Sí | Credenciales esperadas |

#### `test_requote_uri_with_unquoted_percents(uri, expected)`
```python
def test_requote_uri_with_unquoted_percents(uri, expected):
    """Valida requoteo de URIs con signos % sin escapar (issue #2356).

    Args:
        uri (str): URI de entrada.
        expected (str): URI requoteada.

    Returns:
        None
    """
```

#### `test_unquote_unreserved(uri, expected)`
```python
def test_unquote_unreserved(uri, expected):
    """Valida unquoting solo de caracteres no reservados.

    Args:
        uri (str): URI con secuencias %XX.
        expected (str): URI con unquoting selectivo.

    Returns:
        None
    """
```

#### `test_dotted_netmask(mask, expected)`
```python
def test_dotted_netmask(mask, expected):
    """Valida conversión de prefijo CIDR a máscara dotted.

    Args:
        mask (int): Prefijo (8,24,25).
        expected (str): Máscara dotted (255.0.0.0, etc.).

    Returns:
        None
    """
```

#### `test_select_proxies(url, expected, proxies)`
```python
def test_select_proxies(url, expected, proxies):
    """Valida selección de proxy por host y esquema (http/all).

    Args:
        url (str): URL a evaluar.
        expected (str|None): Proxy esperado.
        proxies (dict): Diccionario de proxies (http_proxies/all_proxies/mixed).

    Returns:
        None
    """
```

#### `test_parse_dict_header(value, expected)`
```python
def test_parse_dict_header(value, expected):
    """Valida parsing de header tipo dict.

    Args:
        value (str): Header crudo.
        expected (dict): Diccionario esperado.

    Returns:
        None
    """
```

#### `test__parse_content_type_header(value, expected)`
```python
def test__parse_content_type_header(value, expected):
    """Valida parsing de Content-Type en (media_type, params).

    Args:
        value (str): Header Content-Type.
        expected (tuple[str, dict]): Tupla (tipo, dict params).

    Returns:
        None
    """
```

#### `test_get_encoding_from_headers(value, expected)`
```python
def test_get_encoding_from_headers(value, expected):
    """Valida extracción de charset desde CaseInsensitiveDict.

    Args:
        value (CaseInsensitiveDict): Headers.
        expected (str|None): Encoding esperado o None.

    Returns:
        None
    """
```

#### `test_iter_slices(value, length)`
```python
def test_iter_slices(value, length):
    """Valida generador iter_slices con diferentes longitudes de slice.

    Args:
        value (str): Cadena a slicear.
        length (int|None): Tamaño de slice.

    Returns:
        None
    """
```

#### `test_parse_header_links(value, expected)`
```python
def test_parse_header_links(value, expected):
    """Valida parsing de header Link (RFC 5988).

    Args:
        value (str): Header Link crudo.
        expected (list[dict]): Lista de dicts con url, rel, type.

    Returns:
        None
    """
```

#### `test_prepend_scheme_if_needed(value, expected)`
```python
def test_prepend_scheme_if_needed(value, expected):
    """Valida adición de esquema http:// si falta.

    Args:
        value (str): URL sin esquema o con esquema.
        expected (str): URL con esquema.

    Returns:
        None
    """
```

#### `test_to_native_string(value, expected)`
```python
def test_to_native_string(value, expected):
    """Valida conversión a str nativo (py2/py3 compat).

    Args:
        value (str|bytes): Valor de entrada.
        expected (str): Valor nativo esperado.

    Returns:
        None
    """
```

#### `test_urldefragauth(url, expected)`
```python
def test_urldefragauth(url, expected):
    """Valida eliminación de fragmento y credenciales de URL.

    Args:
        url (str): URL con auth y fragmento.
        expected (str): URL limpia.

    Returns:
        None
    """
```

#### `test_should_bypass_proxies(url, expected, monkeypatch)`
```python
def test_should_bypass_proxies(url, expected, monkeypatch):
    """Valida should_bypass_proxies con env no_proxy/NO_PROXY.

    Args:
        url (str): URL parametrizada.
        expected (bool): True si debe bypassear proxy.
        monkeypatch (MonkeyPatch): Setea no_proxy.

    Returns:
        None
    """
```

#### `test_should_bypass_proxies_pass_only_hostname(url, expected)`
```python
def test_should_bypass_proxies_pass_only_hostname(url, expected):
    """Verifica que proxy_bypass sea llamado solo con hostname/IP sin puerto ni auth.

    Args:
        url (str): URL con puerto y credenciales.
        expected (str): Hostname esperado pasado a proxy_bypass.

    Returns:
        None
    """
```

#### `test_add_dict_to_cookiejar(cookiejar)`
```python
def test_add_dict_to_cookiejar(cookiejar):
    """Valida adición de dict a CookieJar (compat CookieJar y RequestsCookieJar).

    Args:
        cookiejar (CookieJar|RequestsCookieJar): Jar parametrizado.

    Returns:
        None
    """
```

#### `test_unicode_is_ascii(value, expected)`
```python
def test_unicode_is_ascii(value, expected):
    """Valida detección de strings ASCII puros.

    Args:
        value (str): Cadena a evaluar.
        expected (bool): True si es ASCII.

    Returns:
        None
    """
```

#### `test_should_bypass_proxies_no_proxy(url, expected, monkeypatch)`
```python
def test_should_bypass_proxies_no_proxy(url, expected, monkeypatch):
    """Valida bypass usando argumento no_proxy explícito (no env).

    Args:
        url (str): URL parametrizada.
        expected (bool): Resultado esperado.
        monkeypatch (MonkeyPatch): Fixture (no usado directamente).

    Returns:
        None
    """
```

#### `test_should_bypass_proxies_no_proxy_domain_boundary(url, expected)`
```python
def test_should_bypass_proxies_no_proxy_domain_boundary(url, expected):
    """Valida que matching de no_proxy respete límites de dominio (bpo-39057).

    Args:
        url (str): URL parametrizada.
        expected (bool): True si debe bypassear.

    Returns:
        None
    """
```

#### `test_should_bypass_proxies_win_registry(url, expected, override, monkeypatch)`
```python
@pytest.mark.skipif(os.name != "nt", reason="Test only on Windows")
def test_should_bypass_proxies_win_registry(url, expected, override, monkeypatch):
    """Valida bypass leyendo ProxyOverride del registro de Windows.

    Args:
        url (str): URL parametrizada.
        expected (bool): Resultado esperado.
        override (str|None): Valor ProxyOverride a mockear.
        monkeypatch (MonkeyPatch): Mockea winreg y env.

    Returns:
        None
    """
```
**Clases/Funciones anidadas en `test_should_bypass_proxies_win_registry`**

##### `RegHandle.Close(self)` (win_registry)
```python
def Close(self):
    """No-op para cerrar handle de registro mockeado.

    Returns:
        None
    """
```

##### `OpenKey(key, subkey)` (win_registry)
```python
def OpenKey(key, subkey):
    """Mock de winreg.OpenKey que retorna handle IE Settings.

    Args:
        key (int): Hive del registro.
        subkey (str): Subclave.

    Returns:
        RegHandle: Handle mockeado.
    """
```

##### `QueryValueEx(key, value_name)` (win_registry)
```python
def QueryValueEx(key, value_name):
    """Mock de winreg.QueryValueEx para ProxyEnable y ProxyOverride.

    Args:
        key (RegHandle): Handle.
        value_name (str): Nombre del valor ("ProxyEnable"/"ProxyOverride").

    Returns:
        list: Lista con valor mockeado (int/str).
    """
```

#### `test_should_bypass_proxies_win_registry_bad_values(monkeypatch)`
```python
@pytest.mark.skipif(os.name != "nt", reason="Test only on Windows")
def test_should_bypass_proxies_win_registry_bad_values(monkeypatch):
    """Valida manejo de valores inválidos en registro (ProxyEnable="").

    Args:
        monkeypatch (MonkeyPatch): Mockea winreg.

    Returns:
        None
    """
```
**Anidadas `RegHandle.Close`, `OpenKey`, `QueryValueEx` (bad_values)** - Mismas firmas que arriba, con `QueryValueEx` retornando `[""]` para `ProxyEnable` para simular valor inválido.

#### `test_set_environ(env_name, value)`
```python
def test_set_environ(env_name, value):
    """Valida context manager set_environ setea y restaura environ.

    Args:
        env_name (str): Nombre de variable de entorno.
        value (str|None): Valor a setear.

    Returns:
        None
    """
```

#### `test_set_environ_raises_exception()`
```python
def test_set_environ_raises_exception():
    """Valida que set_environ propague excepciones internas y restaure environ.

    Raises:
        Exception: "Expected exception" lanzada dentro del contexto.

    Returns:
        None
    """
```
| Lanza | Condición |
|---|---|
| `Exception` | Bloque `with set_environ` lanza excepción interna |

#### `test_should_bypass_proxies_win_registry_ProxyOverride_value(monkeypatch)`
```python
@pytest.mark.skipif(os.name != "nt", reason="Test only on Windows")
def test_should_bypass_proxies_win_registry_ProxyOverride_value(monkeypatch):
    """Valida manejo de ProxyOverride terminado en ; y con <-loopback>.

    Args:
        monkeypatch (MonkeyPatch): Mockea winreg.

    Returns:
        None
    """
```
**Anidadas `RegHandle.Close`, `OpenKey`, `QueryValueEx` (ProxyOverride)** - `QueryValueEx` retorna `"192.168.*;127.0.0.1;localhost.localdomain;172.16.1.1;<-loopback>;"` para `ProxyOverride` y `[1]` para `ProxyEnable`.

---

## 3. Ejemplo de Uso

```python
# Ejemplo 1: Ejecución de la suite completa con pytest
# $ pytest tests/test_utils.py -v

# Ejemplo 2: Uso directo de las utilidades validadas por los tests
from requests.utils import (
    super_len, get_netrc_auth, select_proxy, 
    should_bypass_proxies, parse_header_links
)
from io import BytesIO

# 1. Cálculo de longitud de stream (cubierto por TestSuperLen)
stream = BytesIO(b"Test payload")
print(super_len(stream))  # 12
stream.read(4)
print(super_len(stream))  # 8 (restante)

# 2. Selección de proxy por host (cubierto por test_select_proxies)
proxies = {
    "http": "http://proxy.global:3128",
    "http://api.internal": "http://proxy.internal:8080"
}
print(select_proxy("http://api.internal/v1/data", proxies))
# http://proxy.internal:8080

# 3. Bypass de proxy por no_proxy (cubierto por TestGetEnvironProxies)
# con no_proxy="192.168.0.0/24,localhost"
print(should_bypass_proxies("http://192.168.0.10/api", no_proxy="192.168.0.0/24"))
# True

# 4. Parsing de header Link (cubierto por test_parse_header_links)
links = parse_header_links('<https://api.github.com/next>; rel="next"; type="application/json"')
print(links)
# [{'url': 'https://api.github.com/next', 'rel': 'next', 'type': 'application/json'}]

# 5. Context manager para variables de entorno (cubierto por test_set_environ)
from requests.utils import set_environ
import os

with set_environ("NO_PROXY", "example.com"):
    assert os.environ["NO_PROXY"] == "example.com"
# Al salir, NO_PROXY se restaura a su valor original
```
