# Documentación Técnica - `test_utils.py`

### 1. Descripción General

**Archivo:** `test_utils.py` (Suite de pruebas para `requests.utils`)  
**Tipo:** Pruebas unitarias y de integración  
**Framework:** `pytest`  
**Módulo bajo prueba:** `requests.utils`, `requests._internal_utils`, `requests.cookies`, `requests.structures`

Este archivo contiene la suite de pruebas automatizadas para validar el correcto funcionamiento de las **funciones utilitarias de la librería `requests`**. No contiene lógica de negocio en sí misma, sino que verifica mediante aserciones el comportamiento esperado de más de 25 funciones auxiliares encargadas de manejo de URLs, proxies, codificación, cabeceras HTTP, cookies, archivos y redes.

Su objetivo es garantizar la robustez, compatibilidad y manejo de casos borde de las utilidades internas que utiliza `requests` para realizar peticiones HTTP.

### 2. Dependencias Principales

| Dependencia | Propósito |
| :--- | :--- |
| `pytest` | Framework de testing, parametrización y fixtures |
| `unittest.mock` | Mocking para simular `proxy_bypass` y registro de Windows |
| `requests.utils` | Módulo principal bajo prueba (25+ funciones) |
| `requests.compat` | Capa de compatibilidad Python 2/3 |
| `requests.cookies` | Validación de `RequestsCookieJar` |
| `requests.structures` | Validación de `CaseInsensitiveDict` |
| `tarfile`, `zipfile`, `filecmp`, `os` | Pruebas de manejo de archivos y archivos comprimidos |
| `winreg` | Pruebas específicas para bypass de proxy en Windows |

### 3. Componentes Principales

El archivo está estructurado en **12 clases de prueba** y **18 funciones de prueba independientes**. Cada componente valida una o varias funciones utilitarias de forma aislada.

#### 3.1 Clases de Prueba

| Clase | Función Validada | Descripción |
| :--- | :--- | :--- |
| `TestSuperLen` | `super_len()` | Verifica el cálculo correcto de la longitud de objetos tipo stream, archivos, `BytesIO`, `StringIO`, `tarfile` y objetos con `__len__`, `tell()` y `fileno()`. Incluye manejo de errores `IOError`/`OSError`. |
| `TestGetNetrcAuth` | `get_netrc_auth()` | Valida la autenticación vía archivo `.netrc`. Prueba extracción de credenciales, protección contra parsing malicioso de URLs y manejo de credenciales por defecto vacías. |
| `TestToKeyValList` | `to_key_val_list()` | Comprueba la conversión de diccionarios, tuplas y listas a formato `list[tuple]`. Valida el lanzamiento de `ValueError` con entradas inválidas. |
| `TestUnquoteHeaderValue` | `unquote_header_value()` | Valida el desentrecomillado de valores de cabeceras HTTP, incluyendo manejo de escapes `\\` y el flag `is_filename`. |
| `TestGetEnvironProxies` | `get_environ_proxies()` | Verifica la lógica de `no_proxy` / `NO_PROXY` para decidir si una URL debe bypasear el proxy. Prueba rangos CIDR, IPs y dominios. |
| `TestIsIPv4Address` | `is_ipv4_address()` | Valida el reconocimiento de direcciones IPv4 válidas e inválidas. |
| `TestIsValidCIDR` | `is_valid_cidr()` | Valida el formato de notación CIDR (ej. `192.168.1.0/24`). |
| `TestAddressInNetwork` | `address_in_network()` | Comprueba si una IP pertenece a una red CIDR dada. |
| `TestGuessFilename` | `guess_filename()` | Valida la extracción del nombre de archivo desde objetos con atributo `name`, manejando tipos `bytes` y `str`. |
| `TestExtractZippedPaths` | `extract_zipped_paths()` | Verifica la extracción de rutas dentro de archivos `.zip` y `.egg` a una ruta temporal, y que rutas no comprimidas permanezcan intactas. |
| `TestContentEncodingDetection` | `get_encodings_from_content()` | Valida la detección de codificación a partir de etiquetas HTML `<meta charset>` y `<?xml encoding>`, y su orden de precedencia. |
| `TestGuessJSONUTF` | `guess_json_utf()` | Valida la detección de la codificación UTF de un payload JSON (utf-8, utf-16, utf-32, con y sin BOM). |

#### 3.2 Funciones de Prueba Independientes

Funciones parametrizadas con `@pytest.mark.parametrize` que validan utilidades de forma directa:

| Función de Prueba | Función Validada | Aspecto Validado |
| :--- | :--- | :--- |
| `test_get_auth_from_url` | `get_auth_from_url()` | Extracción y decodificación de `user:password` desde la URL, incluyendo caracteres especiales y URL-encoding. |
| `test_requote_uri_with_unquoted_percents` | `requote_uri()` | Re-codificación correcta de URIs con signos `%` sin codificar. |
| `test_unquote_unreserved` | `unquote_unreserved()` | Decodificación solo de caracteres no reservados según RFC 3986. |
| `test_dotted_netmask` | `dotted_netmask()` | Conversión de máscara CIDR (ej. 24) a máscara punteada (ej. 255.255.255.0). |
| `test_select_proxies` | `select_proxy()` | Selección de proxy por esquema y por host específico, incluyendo proxy `all://`. |
| `test_parse_dict_header` | `parse_dict_header()` | Parsing de cabeceras tipo `foo="bar", key=value`. |
| `test__parse_content_type_header` | `_parse_content_type_header()` | Parsing de `Content-Type` y sus parámetros `charset` y `boundary`. |
| `test_get_encoding_from_headers` | `get_encoding_from_headers()` | Obtención de codificación desde `CaseInsensitiveDict` de cabeceras. |
| `test_iter_slices` | `iter_slices()` | Iteración por bloques de un string/bytes según longitud. |
| `test_parse_header_links` | `parse_header_links()` | Parsing de cabecera `Link` con atributos `rel` y `type`. |
| `test_prepend_scheme_if_needed` | `prepend_scheme_if_needed()` | Añade `http://` si la URL no tiene esquema. |
| `test_to_native_string` | `to_native_string()` | Conversión de `bytes` a `str` nativo. |
| `test_urldefragauth` | `urldefragauth()` | Elimina credenciales y fragmento de la URL. |
| `test_should_bypass_proxies` (5 variantes) | `should_bypass_proxies()` | Lógica completa de bypass de proxy por `no_proxy`, CIDR, dominio, puerto y registro de Windows (`winreg`). Incluye validación de límites de dominio. |
| `test_add_dict_to_cookiejar` | `add_dict_to_cookiejar()` | Añadir diccionario de cookies a `CookieJar` y `RequestsCookieJar`. |
| `test_unicode_is_ascii` | `unicode_is_ascii()` | Verifica si un string contiene solo caracteres ASCII. |
| `test_set_environ` | `set_environ()` | Context manager que modifica y restaura variables de entorno de forma segura, incluso ante excepciones. |

### 4. Características Técnicas Destacadas

*   **Parametrización Intensiva:** Uso extensivo de `@pytest.mark.parametrize` para probar múltiples casos de entrada/salida con una sola función.
*   **Fixtures y Monkeypatch:** Uso de `tmp_path`, `tmpdir`, `monkeypatch` y `recwarn` para aislar pruebas de sistema de archivos, variables de entorno y warnings.
*   **Compatibilidad Multiplataforma:** Pruebas condicionales con `@pytest.mark.skipif(os.name != "nt")` para validar comportamiento específico de Windows Registry.
*   **Manejo de Casos Borde:** Cobertura de errores de I/O, archivos parcialmente leídos, URLs malformadas, BOMs y codificaciones inválidas.
*   **Pruebas de Seguridad:** Validación explícita contra vulnerabilidades de parsing de URL en `get_netrc_auth`.

### 5. Ejecución

```bash
pytest tests/test_utils.py -v
pytest tests/test_utils.py::TestSuperLen -v
pytest tests/test_utils.py -k "proxy"
```

> **Conclusión:** El archivo actúa como contrato de calidad para el módulo `requests.utils`. Asegura que todas las funciones auxiliares, críticas para el funcionamiento de la librería, se comporten de forma predecible, segura y compatible en diferentes entornos y sistemas operativos.
