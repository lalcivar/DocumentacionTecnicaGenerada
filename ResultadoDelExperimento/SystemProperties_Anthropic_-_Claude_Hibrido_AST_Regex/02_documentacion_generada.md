# Documentación Técnica — `org.apache.commons.lang3.SystemProperties`

---

## 1. Resumen ejecutivo

`SystemProperties` es una clase utilitaria `final` que centraliza el acceso de solo lectura a las [System Properties](https://docs.oracle.com/en/java/javase/25/docs/api/system-properties.html) de la plataforma Java, exponiendo una constante `String` con el nombre de cada propiedad y un *accessor* estático tipado por propiedad. Todos los accesos se canalizan a través de los métodos internos `getProperty(...)`, que aíslan al llamador de `SecurityException` y del tratamiento de valores vacíos/ausentes, retornando un valor por defecto (habitualmente `null`) en lugar de propagar la excepción.

Adicionalmente, provee conversores de conveniencia (`getBoolean()`, `getInt()`, `getLong()`, `getPath()`) con soporte de valores por defecto mediante `Supplier`/`BooleanSupplier`/`IntSupplier`/`LongSupplier`, y la capacidad de derivar claves con el patrón `"SimpleClassName.key"` a través del helper privado `toKey()`.

> **Nota sobre `@author`:** el código fuente no declara ninguna etiqueta `@author` (convención de Apache Commons: la autoría es colectiva de la Apache Software Foundation y se gestiona vía el archivo `NOTICE`). Se documenta explícitamente como `@author Apache Software Foundation (ASF)` únicamente a nivel informativo; **no existe** dicha etiqueta en el fichero analizado.

---

## 2. Especificación de métodos y constructores

### 2.1 Convenciones aplicadas en esta sección

| Concepto | Significado |
|---|---|
| **Requerido** | `Sí` = el parámetro no admite semántica de omisión; `No` = admite `null` con comportamiento definido. |
| **`@return` nulo** | Salvo indicación contraria, `null` significa “propiedad ausente, vacía o inaccesible por `SecurityException`”. |
| **`@throws`** | Solo se listan excepciones derivables del código provisto (`Boolean/Integer/Long.parse*`, `Paths.get`, desreferencias de `Supplier`). `SecurityException` es **capturada y absorbida** en `getProperty(String, Supplier)`. |

---

### 2.2 Constructor

#### `SystemProperties()`

Constructor público, vacío y **deprecado**. Su existencia se mantiene por compatibilidad binaria; el Javadoc del código indica su conversión a `private` en la versión 4.0. La clase no mantiene estado de instancia: todo su API útil es `static`.

| `@param` | Tipo | Requerido | Descripción |
|---|---|---|---|
| — | — | — | No recibe parámetros. |

| Elemento | Valor |
|---|---|
| `@return` | Nueva instancia sin estado de `SystemProperties`. |
| `@throws` | Ninguna. |
| Modificadores | `public` |
| Anotaciones | `@Deprecated` |

---

### 2.3 Núcleo de acceso a propiedades

#### `getProperty(String property)`

Punto de entrada público de bajo nivel. Delega en `getProperty(String, Supplier)` usando `Suppliers.nul()` como proveedor de valor por defecto, por lo que **nunca** lanza `SecurityException` y retorna `null` ante cualquier fallo.

| `@param` | Tipo | Requerido | Descripción |
|---|---|---|---|
| `property` | `String` | No (admite `null`/vacío) | Nombre de la propiedad del sistema. Si es `null` o vacío (`StringUtils.isEmpty`), se retorna el valor por defecto. |

| Elemento | Valor |
|---|---|
| `@return` | `String` con el valor de la propiedad, o `null` si está ausente, vacía o se produjo un problema de seguridad. |
| `@throws` | Ninguna. |
| Modificadores | `public static` |

---

#### `getProperty(String property, String defaultIfAbsent)`

Sobrecarga **de visibilidad de paquete** (`package-private`). Envuelve el valor literal en una lambda `() -> defaultIfAbsent` y delega en la variante con `Supplier`.

| `@param` | Tipo | Requerido | Descripción |
|---|---|---|---|
| `property` | `String` | No | Nombre de la propiedad del sistema. |
| `defaultIfAbsent` | `String` | No (admite `null`) | Valor a devolver si la propiedad está vacía/ausente o si se captura `SecurityException`. |

| Elemento | Valor |
|---|---|
| `@return` | Valor de la propiedad o `defaultIfAbsent`. |
| `@throws` | Ninguna. |
| Modificadores | `static` (sin modificador de acceso explícito) |

---

#### `getProperty(String property, Supplier<String> defaultIfAbsent)`

Sobrecarga **de visibilidad de paquete** que concentra toda la lógica real de resolución:

1. Si `property` es `null`/vacío → `Suppliers.get(defaultIfAbsent)`.
2. En otro caso → `StringUtils.getIfEmpty(System.getProperty(property), defaultIfAbsent)`.
3. Si `System.getProperty` lanza `SecurityException` → se captura silenciosamente y se retorna `defaultIfAbsent.get()`.

| `@param` | Tipo | Requerido | Descripción |
|---|---|---|---|
| `property` | `String` | No | Nombre de la propiedad del sistema. |
| `defaultIfAbsent` | `Supplier<String>` | Condicional | Proveedor del valor por defecto. En las rutas 1 y 2 se tolera `null` (vía `Suppliers.get`/`StringUtils.getIfEmpty`); en la ruta 3 (`catch`) se invoca **directamente** `defaultIfAbsent.get()`. |

| Elemento | Valor |
|---|---|
| `@return` | Valor de la propiedad o el valor suministrado por `defaultIfAbsent`. |
| `@throws NullPointerException` | Si se captura una `SecurityException` **y** `defaultIfAbsent` es `null` (desreferencia directa en el bloque `catch`). |
| Modificadores | `static` (package-private) |

---

#### `isPropertySet(String property)`

Atajo semántico equivalente a `getProperty(property) != null`.

| `@param` | Tipo | Requerido | Descripción |
|---|---|---|---|
| `property` | `String` | No | Nombre de la propiedad a verificar. |

| Elemento | Valor |
|---|---|
| `@return` | `true` si la propiedad está definida y no vacía; `false` si está ausente, vacía o se produjo `SecurityException`. |
| `@throws` | Ninguna. |
| `@since` | 3.18.0 |
| Modificadores | `public static` |

---

#### `toKey(Class<?> clazz, String key, boolean simpleKey)`

Helper **privado** que compone la clave `"NombreDeClase.key"` mediante `ClassUtils.getName(clazz, StringUtils.EMPTY, simpleKey)` concatenado con `"."` y `key`. Es la base de las sobrecargas `getBoolean/getInt/getLong` que reciben un `Class<?>`.

| `@param` | Tipo | Requerido | Descripción |
|---|---|---|---|
| `clazz` | `Class<?>` | No | Clase de la que se deriva el prefijo. Si es `null`, `ClassUtils.getName` devuelve `StringUtils.EMPTY`, generando una clave con punto inicial (`".key"`). |
| `key` | `String` | Sí | Subclave que se concatena tras el punto separador. |
| `simpleKey` | `boolean` | Sí | `true` → usa el *simple name*; `false` → usa el nombre totalmente cualificado. |

| Elemento | Valor |
|---|---|
| `@return` | `String` con la clave compuesta `prefijo + "." + key`. |
| `@throws` | Ninguna (depende del contrato de `ClassUtils.getName`). |
| Modificadores | `private static` |

---

### 2.4 Accesores tipados con valor por defecto

#### `getBoolean(Class<?> clazz, String key, BooleanSupplier defaultIfAbsent)`

Resuelve la propiedad `"SimpleClassName.key"` (vía `toKey(clazz, key, true)`) y la interpreta como `boolean`.

| `@param` | Tipo | Requerido | Descripción |
|---|---|---|---|
| `clazz` | `Class<?>` | No | Clase usada para el *SimpleClassName*. |
| `key` | `String` | Sí | Subclave. |
| `defaultIfAbsent` | `BooleanSupplier` | No (admite `null`) | Proveedor del valor por defecto; si es `null` y la propiedad está ausente, el resultado es `false`. |

| Elemento | Valor |
|---|---|
| `@return` | `boolean` parseado con `Boolean.parseBoolean(String)` o el valor del `BooleanSupplier`. |
| `@throws` | Ninguna (`Boolean.parseBoolean` no lanza excepciones de formato). |
| `@since` | 3.19.0 |
| Modificadores | `public static` |

---

#### `getBoolean(String key, BooleanSupplier defaultIfAbsent)`

Implementación base: obtiene el `String` con `getProperty(key)`; si es `null`, retorna `defaultIfAbsent != null && defaultIfAbsent.getAsBoolean()`.

| `@param` | Tipo | Requerido | Descripción |
|---|---|---|---|
| `key` | `String` | Sí | Nombre completo de la propiedad. |
| `defaultIfAbsent` | `BooleanSupplier` | No (admite `null`) | Proveedor del valor por defecto; `null` ⇒ `false`. |

| Elemento | Valor |
|---|---|
| `@return` | `boolean` según `Boolean.parseBoolean(String)`, o el valor por defecto. |
| `@throws` | Ninguna. |
| Modificadores | `public static` |

---

#### `getInt(Class<?> clazz, String key, IntSupplier defaultIfAbsent)`

Variante de `getInt` con clave derivada de clase mediante `toKey(clazz, key, true)`.

| `@param` | Tipo | Requerido | Descripción |
|---|---|---|---|
| `clazz` | `Class<?>` | No | Clase usada para el *SimpleClassName*. |
| `key` | `String` | Sí | Subclave. |
| `defaultIfAbsent` | `IntSupplier` | No (admite `null`) | Proveedor del valor por defecto; `null` ⇒ `0`. |

| Elemento | Valor |
|---|---|
| `@return` | `int` parseado con `Integer.parseInt(String)` o el valor por defecto. |
| `@throws NumberFormatException` | Si la propiedad existe pero su valor no es un entero decimal válido. |
| `@since` | 3.19.0 |
| Modificadores | `public static` |

---

#### `getInt(String key, IntSupplier defaultIfAbsent)`

Implementación base: `str == null ? (defaultIfAbsent != null ? defaultIfAbsent.getAsInt() : 0) : Integer.parseInt(str)`.

| `@param` | Tipo | Requerido | Descripción |
|---|---|---|---|
| `key` | `String` | Sí | Nombre completo de la propiedad. |
| `defaultIfAbsent` | `IntSupplier` | No (admite `null`) | Proveedor del valor por defecto; `null` ⇒ `0`. |

| Elemento | Valor |
|---|---|
| `@return` | `int` con el valor parseado o el valor por defecto. |
| `@throws NumberFormatException` | Valor presente no parseable por `Integer.parseInt`. |
| Modificadores | `public static` |

---

#### `getLong(Class<?> clazz, String key, LongSupplier defaultIfAbsent)`

Variante de `getLong` con clave derivada de clase mediante `toKey(clazz, key, true)`.

| `@param` | Tipo | Requerido | Descripción |
|---|---|---|---|
| `clazz` | `Class<?>` | No | Clase usada para el *SimpleClassName*. |
| `key` | `String` | Sí | Subclave. |
| `defaultIfAbsent` | `LongSupplier` | No (admite `null`) | Proveedor del valor por defecto; `null` ⇒ `0`. |

| Elemento | Valor |
|---|---|
| `@return` | `long` parseado con `Long.parseLong(String)` o el valor por defecto. |
| `@throws NumberFormatException` | Valor presente no parseable. |
| `@since` | 3.19.0 |
| Modificadores | `public static` |

---

#### `getLong(String key, LongSupplier defaultIfAbsent)`

Implementación base: `str == null ? (defaultIfAbsent != null ? defaultIfAbsent.getAsLong() : 0) : Long.parseLong(str)`.

| `@param` | Tipo | Requerido | Descripción |
|---|---|---|---|
| `key` | `String` | Sí | Nombre completo de la propiedad. |
| `defaultIfAbsent` | `LongSupplier` | No (admite `null`) | Proveedor del valor por defecto; `null` ⇒ `0`. |

| Elemento | Valor |
|---|---|
| `@return` | `long` con el valor parseado o el valor por defecto. |
| `@throws NumberFormatException` | Valor presente no parseable. |
| Modificadores | `public static` |

---

#### `getPath(String key, Supplier<Path> defaultIfAbsent)`

Convierte el valor de la propiedad a `java.nio.file.Path` mediante `Paths.get(String)`; si la propiedad es `null`, resuelve el defecto con `Suppliers.get(defaultIfAbsent)` (tolerante a `null`).

| `@param` | Tipo | Requerido | Descripción |
|---|---|---|---|
| `key` | `String` | Sí | Nombre completo de la propiedad. |
| `defaultIfAbsent` | `Supplier<Path>` | No (admite `null`) | Proveedor del `Path` por defecto; `null` ⇒ resultado `null`. |

| Elemento | Valor |
|---|---|
| `@return` | `Path` construido desde el valor de la propiedad, o el `Path` del proveedor por defecto. |
| `@throws java.nio.file.InvalidPathException` | Si el valor de la propiedad no es convertible a una ruta válida (`Paths.get`). |
| `@since` | 3.20.0 |
| Modificadores | `public static` |

---

### 2.5 Accesores con sobrecarga de valor por defecto

#### `getJavaSpecificationVersion(String defaultValue)`

Delega en `getProperty(JAVA_SPECIFICATION_VERSION, defaultValue)`.

| `@param` | Tipo | Requerido | Descripción |
|---|---|---|---|
| `defaultValue` | `String` | No (admite `null`) | Valor devuelto si `java.specification.version` está vacía/ausente o hay `SecurityException`. |

| Elemento | Valor |
|---|---|
| `@return` | Valor de `java.specification.version` o `defaultValue`. |
| `@throws` | Ninguna. |
| `@since` | 3.15.0 |
| Modificadores | `public static` |

---

#### `getLineSeparator(Supplier<String> defaultIfAbsent)`

Delega en `getProperty(LINE_SEPARATOR, defaultIfAbsent)`.

| `@param` | Tipo | Requerido | Descripción |
|---|---|---|---|
| `defaultIfAbsent` | `Supplier<String>` | Condicional | Proveedor usado cuando `line.separator` está vacía/ausente. Si se captura `SecurityException` y es `null`, se produce `NullPointerException`. |

| Elemento | Valor |
|---|---|
| `@return` | Valor de `line.separator` o el valor del proveedor. |
| `@throws NullPointerException` | Propagada desde `getProperty(String, Supplier)` en la ruta de `SecurityException` con proveedor `null`. |
| `@since` | 3.15.0 |
| Modificadores | `public static` |

---

#### `getUserName(String defaultValue)`

Delega en `getProperty(USER_NAME, defaultValue)`.

| `@param` | Tipo | Requerido | Descripción |
|---|---|---|---|
| `defaultValue` | `String` | No (admite `null`) | Valor devuelto si `user.name` está vacía/ausente o hay `SecurityException`. |

| Elemento | Valor |
|---|---|
| `@return` | Valor de `user.name` o `defaultValue`. |
| `@throws` | Ninguna. |
| `@since` | 3.15.0 |
| Modificadores | `public static` |

---

### 2.6 Accesores sin argumentos (contrato común)

Todos los métodos de esta sección comparten exactamente la misma firma y contrato Javadoc:

| Elemento | Valor |
|---|---|
| Firma | `public static String <nombreMetodo>()` |
| `@param` | Ninguno. |
| `@return` | Valor de la propiedad del sistema indicada, o `null` si está ausente, vacía o se produjo `SecurityException`. |
| `@throws` | Ninguna. |
| Implementación | `return getProperty(<CONSTANTE>);` |

#### 2.6.1 Plataforma, archivos y codificación

| Método | Propiedad leída | `@since` / Estado |
|---|---|---|
| `getFileEncoding()` | `file.encoding` | 3.13.0 |
| `getFileSeparator()` | `file.separator` | 3.13.0 |
| `getPathSeparator()` | `path.separator` | 3.13.0 |
| `getLineSeparator()` | `line.separator` | 3.13.0 |
| `getNativeEncoding()` | `native.encoding` | 3.15.0 |
| `getStdErrEncoding()` | `stderr.encoding` | 3.15.0 |
| `getStdOutEncoding()` | `stdout.encoding` | 3.15.0 |
| `getOsArch()` | `os.arch` | 3.13.0 |
| `getOsName()` | `os.name` | 3.13.0 |
| `getOsVersion()` | `os.version` | 3.13.0 |

#### 2.6.2 Identidad y entorno del usuario

| Método | Propiedad leída | `@since` / Estado |
|---|---|---|
| `getUserCountry()` | `user.country` | 3.13.0 |
| `getUserDir()` | `user.dir` | 3.13.0 |
| `getUserExtensions()` | `user.extensions` | 3.15.0 |
| `getUserHome()` | `user.home` | 3.13.0 |
| `getUserLanguage()` | `user.language` | 3.13.0 |
| `getUserName()` | `user.name` | 3.13.0 |
| `getUserRegion()` | `user.region` | 3.15.0 |
| `getUserScript()` | `user.script` | 3.15.0 |
| `getUserTimezone()` | `user.timezone` | 3.13.0 |
| `getUserVariant()` | `user.variant` | 3.15.0 |

#### 2.6.3 Runtime, VM y especificación Java

| Método | Propiedad leída | `@since` / Estado |
|---|---|---|
| `getJavaHome()` | `java.home` | 3.13.0 |
| `getJavaIoTmpdir()` | `java.io.tmpdir` | 3.13.0 |
| `getJavaLibraryPath()` | `java.library.path` | 3.13.0 |
| `getJavaClassPath()` | `java.class.path` | 3.13.0 |
| `getJavaClassVersion()` | `java.class.version` | 3.13.0 |
| `getJavaRuntimeName()` | `java.runtime.name` | 3.13.0 |
| `getJavaRuntimeVersion()` | `java.runtime.version` | 3.13.0 |
| `getJavaSpecificationName()` | `java.specification.name` | 3.13.0 |
| `getJavaSpecificationVendor()` | `java.specification.vendor` | 3.13.0 |
| `getJavaSpecificationVersion()` | `java.specification.version` | 3.13.0 |
| `getJavaSpecificationMaintenanceVersion()` | `java.specification.maintenance.version` | 3.15.0 |
| `getJavaVendor()` | `java.vendor` | 3.13.0 |
| `getJavaVendorUrl()` | `java.vendor.url` | 3.13.0 |
| `getJavaVendorVersion()` | `java.vendor.version` | 3.15.0 |
| `getJavaVersion()` | `java.version` | 3.13.0 |
| `getJavaVersionDate()` | `java.version.date` | 3.15.0 |
| `getJavaVmInfo()` | `java.vm.info` | 3.13.0 |
| `getJavaVmName()` | `java.vm.name` | 3.13.0 |
| `getJavaVmSpecificationName()` | `java.vm.specification.name` | 3.13.0 |
| `getJavaVmSpecificationVendor()` | `java.vm.specification.vendor` | 3.13.0 |
| `getJavaVmSpecificationVersion()` | `java.vm.specification.version` | 3.13.0 |
| `getJavaVmVendor()` | `java.vm.vendor` | 3.13.0 |
| `getJavaVmVersion()` | `java.vm.version` | 3.13.0 |
| `getJavaSystemClassLoader()` | `java.system.class.loader` | 3.15.0 |
| `getJavaLocaleProviders()` | `java.locale.providers` | 3.13.0 (Java 9+) |
| `getJavaLocaleUseOldIsoCodes()` | `java.locale.useOldISOCodes` | 3.15.0 |
| `getJavaPropertiesDate()` | `java.properties.date` | 3.15.0 |
| `getJavaTimeZoneDefaultZoneRulesProvider()` | `java.time.zone.DefaultZoneRulesProvider` | 3.15.0 |

#### 2.6.4 Accesores deprecados (propiedades ausentes en Java 17+)

Todos marcados con `@Deprecated` (“*Deprecated without replacement*”).

| Método | Propiedad leída | Estado |
|---|---|---|
| `getAwtToolkit()` | `awt.toolkit` | `@Deprecated` |
| `getJavaAwtFonts()` | `java.awt.fonts` | `@Deprecated` |
| `getJavaAwtGraphicsenv()` | `java.awt.graphicsenv` | `@Deprecated` |
| `getJavaAwtHeadless()` | `java.awt.headless` | `@Deprecated` |
| `getJavaAwtPrinterjob()` | `java.awt.printerjob` | `@Deprecated` |
| `getJavaCompiler()` | `java.compiler` | `@Deprecated` (ver JDK-8305998) |
| `getJavaEndorsedDirs()` | `java.endorsed.dirs` | `@Deprecated` |
| `getJavaExtDirs()` | `java.ext.dirs` | `@Deprecated` |

#### 2.6.5 Red: HTTP, HTTPS, FTP, SOCKS y proxies

| Método | Propiedad leída | `@since` |
|---|---|---|
| `getFtpNonProxyHost()` | `ftp.nonProxyHosts` | 3.15.0 |
| `getFtpProxyHost()` | `ftp.proxyHost` | 3.15.0 |
| `getFtpProxyPort()` | `ftp.proxyPort` | 3.15.0 |
| `getHttpAgent()` | `http.agent` | 3.15.0 |
| `getHttpAuthDigestCnonceRepeat()` | `http.auth.digest.cnonceRepeat` | 3.15.0 |
| `getHttpAuthDigestReenabledAlgorithms()` | `http.auth.digest.reEnabledAlgorithms` | 3.15.0 |
| `getHttpAuthDigestValidateProxy()` | `http.auth.digest.validateProxy` | 3.15.0 |
| `getHttpAuthDigestValidateServer()` | `http.auth.digest.validateServer` | 3.15.0 |
| `getHttpAuthNtlmDomain()` | `http.auth.ntlm.domain` | 3.15.0 |
| `getHttpKeepAlive()` | `http.keepAlive` | 3.15.0 |
| `getHttpKeepAliveTimeProxy()` | `http.keepAlive.time.proxy` | 3.15.0 |
| `getHttpKeepAliveTimeServer()` | `http.keepAlive.time.server` | 3.15.0 |
| `getHttpMaxConnections()` | `http.maxConnections` | 3.15.0 |
| `getHttpMaxRedirects()` | `http.maxRedirects` | 3.15.0 |
| `getHttpNonProxyHosts()` | `http.nonProxyHosts` | 3.15.0 |
| `getHttpProxyHost()` | `http.proxyHost` | 3.15.0 |
| `getHttpProxyPort()` | `http.proxyPort` | 3.15.0 |
| `getHttpsProxyHost()` | `https.proxyHost` | 3.15.0 |
| `getHttpsProxyPort()` | `https.proxyPort` | 3.15.0 |
| `getSocksProxyHost()` | `socksProxyHost` | 3.15.0 |
| `getSocksProxyPort()` | `socksProxyPort` | 3.15.0 |
| `getSocksProxyVersion()` | `socksProxyVersion` | 3.15.0 |
| `getJavaNetPreferIpv4Stack()` | `java.net.preferIPv4Stack` | 3.15.0 |
| `getJavaNetPreferIpv6Addresses()` | `java.net.preferIPv6Addresses` | 3.15.0 |
| `getJavaNetSocksPassword()` | `java.net.socks.password` | 3.15.0 |
| `getJavaNetSocksUserName()` | `java.net.socks.username` | 3.15.0 |
| `getJavaNetUseSystemProxies()` | `java.net.useSystemProxies` | 3.15.0 |
| `getNetworkAddressCacheNegativeTtl()` | `networkaddress.cache.negative.ttl` | 3.15.0 |
| `getNetworkAddressCacheStaleTtl()` | `networkaddress.cache.stale.ttl` | 3.15.0 |
| `getNetworkAddressCacheTtl()` | `networkaddress.cache.ttl` | 3.15.0 |

#### 2.6.6 NIO, RMI, JNDI, JDBC y SPI

| Método | Propiedad leída | `@since` |
|---|---|---|
| `getJavaNioChannelsDefaultThreadPoolInitialSize()` | `java.nio.channels.DefaultThreadPool.initialSize` | 3.15.0 |
| `getJavaNioChannelsDefaultThreadPoolThreadFactory()` | `java.nio.channels.DefaultThreadPool.threadFactory` | 3.15.0 |
| `getJavaNioChannelsSpiAsynchronousChannelProvider()` | `java.nio.channels.spi.AsynchronousChannelProvider` | 3.15.0 |
| `getJavaNioChannelsSpiSelectorProvider()` | `java.nio.channels.spi.SelectorProvider` | 3.15.0 |
| `getJavaNioFileSpiDefaultFileSystemProvider()` | `java.nio.file.spi.DefaultFileSystemProvider` | 3.15.0 |
| `getJavaContentHandlerPkgs()` | `java.content.handler.pkgs` | 3.15.0 |
| `getJavaProtocolHandlerPkgs()` | `java.protocol.handler.pkgs` | 3.15.0 |
| `getJavaRmiServerCodebase()` | `java.rmi.server.codebase` | 3.15.0 |
| `getJavaRmiServerHostName()` | `java.rmi.server.hostname` | 3.15.0 |
| `getJavaRmiServerRandomIds()` | `java.rmi.server.randomIDs` | 3.15.0 |
| `getJavaRmiServerRmiClassLoaderSpi()` | `java.rmi.server.RMIClassLoaderSpi` | 3.15.0 |
| `getComSunJndiLdapObjectTrustSerialData()` | `com.sun.jndi.ldap.object.trustSerialData` | 3.15.0 |
| `getComSunNetHttpServerHttpServerProvider()` | `com.sun.net.httpserver.HttpServerProvider` | 3.15.0 |
| `getJdbcDrivers()` | `jdbc.drivers` | 3.15.0 |

#### 2.6.7 Seguridad, SSL, Kerberos y serialización

| Método | Propiedad leída | `@since` |
|---|---|---|
| `getJavaSecurityAuthLoginConfig()` | `java.security.auth.login.config` | 3.15.0 |
| `getJavaSecurityManager()` | `java.security.manager` | 3.15.0 |
| `getJavaxNetSslSessionCacheSize()` | `javax.net.ssl.sessionCacheSize` | 3.15.0 |
| `getJavaxRmiSslClientEnabledCipherSuites()` | `javax.rmi.ssl.client.enabledCipherSuites` | 3.15.0 |
| `getJavaxRmiSslClientEnabledProtocols()` | `javax.rmi.ssl.client.enabledProtocols` | 3.15.0 |
| `getJavaxSecurityAuthUseSubjectCredsOnly()` | `javax.security.auth.useSubjectCredsOnly` | 3.15.0 |
| `getJavaxSmartCardIoTerminalFactoryDefaultType()` | `javax.smartcardio.TerminalFactory.DefaultType` | 3.15.0 |
| `getSunSecurityKrb5Principal()` | `sun.security.krb5.principal` | 3.15.0 |
| `getJdkSerialFilter()` | `jdk.serialFilter` | 3.15.0 |
| `getJdkSerialFilterFactory()` | `jdk.serialFilterFactory` | 3.15.0 |
| `getJdkTlsClientSignatureSchemes()` | `jdk.tls.client.SignatureSchemes` | 3.15.0 |
| `getJdkTlsNamedGroups()` | `jdk.tls.namedGroups` | 3.15.0 |
| `getJdkTlsServerSignatureSchemes()` | `jdk.tls.server.SignatureSchemes` | 3.15.0 |
| `getJdkIncludeInExceptions()` | `jdk.includeInExceptions` | 3.15.0 |
| `getJdkIoPermissionsUseCanonicalPath()` | `jdk.io.permissionsUseCanonicalPath` | 3.15.0 |
| `getJdkJndiLdapObjectFactoriesFilter()` | `jdk.jndi.ldap.object.factoriesFilter` | 3.15.0 |
| `getJdkJndiObjectFactoriesFilter()` | `jdk.jndi.object.factoriesFilter` | 3.15.0 |
| `getJdkJndiRmiObjectFactoriesFilter()` | `jdk.jndi.rmi.object.factoriesFilter` | 3.15.0 |
| `getOrgJcpXmlDsigSecureValidation()` | `org.jcp.xml.dsig.securevalidation` | 3.15.0 |

#### 2.6.8 Cliente/Servidor HTTP del JDK

| Método | Propiedad leída | `@since` |
|---|---|---|
| `getJdkHttpAuthProxyingDisabledSchemes()` | `jdk.http.auth.proxying.disabledSchemes` | 3.15.0 |
| `getJdkHttpAuthTunnelingDisabledSchemes()` | `jdk.http.auth.tunneling.disabledSchemes` | 3.15.0 |
| `getJdkHttpClientAllowRestrictedHeaders()` | `jdk.httpclient.allowRestrictedHeaders` | 3.15.0 |
| `getJdkHttpClientAuthRetryLimit()` | `jdk.httpclient.auth.retrylimit` | 3.15.0 |
| `getJdkHttpClientBufSize()` | `jdk.httpclient.bufsize` | 3.15.0 |
| `getJdkHttpClientConnectionPoolSize()` | `jdk.httpclient.connectionPoolSize` | 3.15.0 |
| `getJdkHttpClientConnectionWindowSize()` | `jdk.httpclient.connectionWindowSize` | 3.15.0 |
| `getJdkHttpClientDisableRetryConnect()` | `jdk.httpclient.disableRetryConnect` | 3.15.0 |
| `getJdkHttpClientEnableAllMethodRetry()` | `jdk.httpclient.enableAllMethodRetry` | 3.15.0 |
| `getJdkHttpClientEnablePush()` | `jdk.httpclient.enablepush` | 3.15.0 |
| `getJdkHttpClientHpackMaxHeaderTableSize()` | `jdk.httpclient.hpack.maxheadertablesize` | 3.15.0 |
| `getJdkHttpClientHttpClientLog()` | `jdk.httpclient.HttpClient.log` | 3.15.0 |
| `getJdkHttpClientKeepAliveTimeout()` | `jdk.httpclient.keepalive.timeout` | 3.15.0 |
| `getJdkHttpClientKeepAliveTimeoutH2()` | `jdk.httpclient.keepalive.timeout.h2` | 3.15.0 |
| `getJdkHttpClientMaxFrameSize()` | `jdk.httpclient.maxframesize` | 3.15.0 |
| `getJdkHttpClientMaxStreams()` | `jdk.httpclient.maxstreams` | 3.15.0 |
| `getJdkHttpClientReceiveBufferSize()` | `jdk.httpclient.receiveBufferSize` | 3.15.0 |
| `getJdkHttpClientRedirectsRetryLimit()` | `jdk.httpclient.redirects.retrylimit` | 3.15.0 |
| `getJdkHttpClientSendBufferSize()` | `jdk.httpclient.sendBufferSize` | 3.15.0 |
| `getJdkHttpClientWebSocketWriteBufferSize()` | `jdk.httpclient.websocket.writeBufferSize` | 3.15.0 |
| `getJdkHttpClientWindowSize()` | `jdk.httpclient.windowsize` | 3.15.0 |
| `getJdkHttpServerMaxConnections()` | `jdk.httpserver.maxConnections` | 3.15.0 |
| `getJdkHttpsNegotiateCbt()` | `jdk.https.negotiate.cbt` | 3.15.0 |
| `getJdkInternalHttpClientDisableHostNameVerification()` | `jdk.internal.httpclient.disableHostnameVerification` | 3.15.0 |
| `getSunNetHttpServerDrainAmount()` | `sun.net.httpserver.drainAmount` | 3.15.0 |
| `getSunNetHttpServerIdleInterval()` | `sun.net.httpserver.idleInterval` | 3.15.0 |
| `getSunNetHttpServerMaxIdleConnections()` | `sun.net.httpserver.maxIdleConnections` | 3.15.0 |
| `getSunNetHttpServerMaxReqHeaders()` | `sun.net.httpserver.maxReqHeaders` | 3.15.0 |
| `getSunNetHttpServerMaxReqTime()` | `sun.net.httpserver.maxReqTime` | 3.15.0 |
| `getSunNetHttpServerMaxRspTime()` | `sun.net.httpserver.maxRspTime` | 3.15.0 |
| `getSunNetHttpServerNoDelay()` | `sun.net.httpserver.nodelay` | 3.15.0 |

#### 2.6.9 Módulos, concurrencia, logging y utilidades

| Método | Propiedad leída | `@since` |
|---|---|---|
| `getJdkModuleMain()` | `jdk.module.main` | 3.15.0 |
| `getJdkModuleMainClass()` | `jdk.module.main.class` | 3.15.0 |
| `getJdkModulePath()` | `jdk.module.path` | 3.15.0 |
| `getJdkModuleUpgradePath()` | `jdk.module.upgrade.path` | 3.15.0 |
| `getJdkNetUnixDomainTmpDir()` | `jdk.net.unixdomain.tmpdir` | 3.15.0 |
| `getJdkNetUrlClassPathShowIgnoredClassPathEntries()` | `jdk.net.URLClassPath.showIgnoredClassPathEntries` | 3.15.0 |
| `getJdkVirtualThreadSchedulerMaxPoolSize()` | `jdk.virtualThreadScheduler.maxPoolSize` | 3.15.0 |
| `getJdkVirtualThreadSchedulerParallelism()` | `jdk.virtualThreadScheduler.parallelism` | 3.15.0 |
| `getJavaUtilConcurrentForkJoinPoolCommonExceptionHandler()` | `java.util.concurrent.ForkJoinPool.common.exceptionHandler` | 3.15.0 |
| `getJavaUtilConcurrentForkJoinPoolCommonMaximumSpares()` | `java.util.concurrent.ForkJoinPool.common.maximumSpares` | 3.15.0 |
| `getJavaUtilConcurrentForkJoinPoolCommonParallelism()` | `java.util.concurrent.ForkJoinPool.common.parallelism` | 3.15.0 |
| `getJavaUtilConcurrentForkJoinPoolCommonThreadFactory()` | `java.util.concurrent.ForkJoinPool.common.threadFactory` | 3.15.0 |
| `getJavaUtilCurrencyData()` | `java.util.currency.data` | 3.15.0 |
| `getJavaUtilLoggingConfigClass()` | `java.util.logging.config.class` | 3.15.0 |
| `getJavaUtilLoggingConfigFile()` | `java.util.logging.config.file` | 3.15.0 |
| `getJavaUtilLoggingSimpleFormatterFormat()` | `java.util.logging.simpleformatter.format` | 3.15.0 |
| `getJavaUtilPrefsPreferencesFactory()` | `java.util.prefs.PreferencesFactory` | 3.13.0 |
| `getJavaUtilPropertyResourceBundleEncoding()` | `java.util.PropertyResourceBundle.encoding` | 3.15.0 |
| `getOrgOpenJdkJavaUtilStreamTripwire()` | `org.openjdk.java.util.stream.tripwire` | 3.15.0 |
| `getJavaxAccessibilityAssistiveTechnologies()` | `javax.accessibility.assistive_technologies` | 3.15.0 |
| `getAppleAwtEnableTemplateImages()` | `apple.awt.enableTemplateImages` | 3.15.0 |

#### 2.6.10 Subsistema XML (JAXP / `jdk.xml.*`)

| Método | Propiedad leída | `@since` |
|---|---|---|
| `getJavaXmlConfigFile()` | `java.xml.config.file` | 3.15.0 |
| `getJdkXmlCdataChunkSize()` | `jdk.xml.cdataChunkSize` | 3.15.0 |
| `getJdkXmlDtdSupport()` | `jdk.xml.dtd.support` | 3.15.0 |
| `getJdkXmlElementAttributeLimit()` | `jdk.xml.elementAttributeLimit` | 3.15.0 |
| `getJdkXmlEnableExtensionFunctions()` | `jdk.xml.enableExtensionFunctions` | 3.15.0 |
| `getJdkXmlEntityExpansionLimit()` | `jdk.xml.entityExpansionLimit` | 3.15.0 |
| `getJdkXmlEntityReplacementLimit()` | `jdk.xml.entityReplacementLimit` | 3.15.0 |
| `getJdkXmlIsStandalone()` | `jdk.xml.isStandalone` | 3.15.0 |
| `getJdkXmlJdkCatalogResolve()` | `jdk.xml.jdkcatalog.resolve` | 3.15.0 |
| `getJdkXmlMaxElementDepth()` | `jdk.xml.maxElementDepth` | 3.15.0 |
| `getJdkXmlMaxGeneralEntitySizeLimit()` | `jdk.xml.maxGeneralEntitySizeLimit` | 3.15.0 |
| `getJdkXmlMaxOccurLimit()` | `jdk.xml.maxOccurLimit` | 3.15.0 |
| `getJdkXmlMaxParameterEntitySizeLimit()` | `jdk.xml.maxParameterEntitySizeLimit` | 3.15.0 |
| `getJdkXmlMaxXmlNameLimit()` | `jdk.xml.maxXMLNameLimit` | 3.15.0 |
| `getJdkXmlOverrideDefaultParser()` | `jdk.xml.overrideDefaultParser` | 3.15.0 |
| `getJdkXmlResetSymbolTable()` | `jdk.xml.resetSymbolTable` | 3.15.0 |
| `getJdkXmlTotalEntitySizeLimit()` | `jdk.xml.totalEntitySizeLimit` | 3.15.0 |
| `getJdkXmlXsltcIsStandalone()` | `jdk.xml.xsltcIsStandalone` | 3.15.0 |

---

### 2.7 Inventario de verificación de cobertura

| Categoría | Cantidad documentada |
|---|---|
| Constructores | 1 (`SystemProperties()`) |
| Métodos `public static` sin argumentos | 189 |
| Métodos `public static` con argumentos | 12 (`getBoolean` ×2, `getInt` ×2, `getLong` ×2, `getPath`, `getProperty(String)`, `getJavaSpecificationVersion(String)`, `getLineSeparator(Supplier)`, `getUserName(String)`, `isPropertySet(String)`) |
| Métodos `static` package-private | 2 (`getProperty(String, String)`, `getProperty(String, Supplier)`) |
| Métodos `private static` | 1 (`toKey(Class, String, boolean)`) |
| Clases internas / anónimas / lambdas nombradas | 0 (solo lambdas anónimas dentro de `getProperty(String, String)`) |

**Constantes sin accessor dedicado** (solo accesibles mediante `getProperty(String)`): `JAVA_SECURITY_KERBEROS_CONF`, `JAVA_SECURITY_KERBEROS_KDC`, `JAVA_SECURITY_KERBEROS_REALM`, `JAVA_SECURITY_DEBUG`.

---

## 3. Ejemplo de uso

```java
package com.example.runtime;

import java.nio.file.Path;
import java.nio.file.Paths;

import org.apache.commons.lang3.SystemProperties;

/**
 * Resuelve la configuración de ejecución a partir de las System Properties,
 * usando {@link SystemProperties} como única fuente de acceso.
 */
public final class RuntimeConfig {

    /** Clave compuesta: se resolverá como "RuntimeConfig.workers". */
    private static final String WORKERS_SUBKEY = "workers";
    /** Clave compuesta: se resolverá como "RuntimeConfig.verbose". */
    private static final String VERBOSE_SUBKEY = "verbose";
    /** Clave compuesta: se resolverá como "RuntimeConfig.timeoutMillis". */
    private static final String TIMEOUT_SUBKEY = "timeoutMillis";

    private final Path workDir;
    private final int workers;
    private final long timeoutMillis;
    private final boolean verbose;

    public RuntimeConfig() {
        // getPath(): lee "app.work.dir" o cae al directorio temporal de la JVM.
        this.workDir = SystemProperties.getPath("app.work.dir",
                () -> Paths.get(SystemProperties.getJavaIoTmpdir()));

        // getInt(Class, String, IntSupplier): lee "RuntimeConfig.workers".
        this.workers = SystemProperties.getInt(RuntimeConfig.class, WORKERS_SUBKEY, () -> 4);

        // getLong(Class, String, LongSupplier): lee "RuntimeConfig.timeoutMillis".
        this.timeoutMillis = SystemProperties.getLong(RuntimeConfig.class, TIMEOUT_SUBKEY, () -> 30_000L);

        // getBoolean(Class, String, BooleanSupplier): lee "RuntimeConfig.verbose".
        this.verbose = SystemProperties.getBoolean(RuntimeConfig.class, VERBOSE_SUBKEY, () -> false);
    }

    /** Imprime un informe del entorno de ejecución. */
    public void printReport() {
        final String eol = SystemProperties.getLineSeparator(() -> "\n");
        final StringBuilder sb = new StringBuilder();

        sb.append("=== Entorno ===").append(eol);
        sb.append("SO            : ")
          .append(SystemProperties.getOsName()).append(' ')
          .append(SystemProperties.getOsVersion()).append(" (")
          .append(SystemProperties.getOsArch()).append(')').append(eol);
        sb.append("JVM           : ")
          .append(SystemProperties.getJavaVmName()).append(" / ")
          .append(SystemProperties.getJavaVmVersion()).append(eol);
        sb.append("Spec. Java    : ")
          .append(SystemProperties.getJavaSpecificationVersion("desconocida")).append(eol);
        sb.append("Vendor        : ").append(SystemProperties.getJavaVendor()).append(eol);
        sb.append("Usuario       : ").append(SystemProperties.getUserName("anonymous")).append(eol);
        sb.append("Home usuario  : ").append(SystemProperties.getUserHome()).append(eol);
        sb.append("Encoding file : ").append(SystemProperties.getFileEncoding()).append(eol);
        sb.append("Encoding nativo: ").append(SystemProperties.getNativeEncoding()).append(eol);

        sb.append("=== Configuración ===").append(eol);
        sb.append("workDir       : ").append(workDir).append(eol);
        sb.append("workers       : ").append(workers).append(eol);
        sb.append("timeoutMillis : ").append(timeoutMillis).append(eol);
        sb.append("verbose       : ").append(verbose).append(eol);

        sb.append("=== Red ===").append(eol);
        // isPropertySet() evita lógica condicional sobre valores null.
        if (SystemProperties.isPropertySet("http.proxyHost")) {
            sb.append("Proxy HTTP    : ")
              .append(SystemProperties.getHttpProxyHost()).append(':')
              .append(SystemProperties.getHttpProxyPort()).append(eol);
            sb.append("No proxy      : ")
              .append(SystemProperties.getHttpNonProxyHosts()).append(eol);
        } else {
            sb.append("Proxy HTTP    : no configurado").append(eol);
        }

        // Acceso genérico para constantes sin accessor dedicado.
        sb.append("Kerberos conf : ")
          .append(SystemProperties.getProperty(SystemProperties.JAVA_SECURITY_KERBEROS_CONF))
          .append(eol);

        System.out.print(sb);
    }

    public static void main(final String[] args) {
        new RuntimeConfig().printReport();
    }
}
```

**Invocación de ejemplo:**

```bash
java -DRuntimeConfig.workers=8 \
     -DRuntimeConfig.verbose=true \
     -DRuntimeConfig.timeoutMillis=60000 \
     -Dapp.work.dir=/var/tmp/app \
     -cp "commons-lang3.jar:classes" \
     com.example.runtime.RuntimeConfig
```

---

## 4. Observaciones de arquitectura y riesgos documentados

| Observación | Impacto | Recomendación |
|---|---|---|
| `getProperty(String, Supplier)` invoca `defaultIfAbsent.get()` sin verificación de nulidad dentro del bloque `catch (SecurityException)`. | Posible `NullPointerException` en entornos con `SecurityManager` restrictivo. | Usar `Suppliers.get(defaultIfAbsent)` también en la rama de excepción. |
| El constructor `SystemProperties()` es `public` y `@Deprecated`. | Permite instanciación inútil de una clase 100 % estática. | Migrar a `private` en la línea 4.0, tal como anota el propio Javadoc. |
| `getInt()`, `getLong()` y `getPath()` no capturan errores de conversión. | `NumberFormatException` / `InvalidPathException` se propagan al llamador. | Validar el valor o envolver la llamada si la propiedad proviene de entrada no confiable. |
| `getBoolean()` nunca falla por formato (`Boolean.parseBoolean`). | Cualquier valor distinto de `"true"` (ignorando mayúsculas) se interpreta como `false`. | Documentar el comportamiento a los consumidores para evitar falsos negativos. |
| Las constantes `JAVA_SECURITY_KERBEROS_CONF`, `JAVA_SECURITY_KERBEROS_KDC`, `JAVA_SECURITY_KERBEROS_REALM` y `JAVA_SECURITY_DEBUG` carecen de accessor. | Asimetría de API. | Acceder con `getProperty(String)` o incorporar los getters correspondientes. |
| `toKey()` es `private static` y asume `key` no nulo. | Una `key` nula produciría la clave literal `"Clase.null"`. | Validar el argumento en las sobrecargas públicas que lo consumen. |