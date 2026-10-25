# Documentación Técnica - `org.apache.commons.lang3.SystemProperties`

> **Paquete:** `org.apache.commons.lang3`
> **Clase:** `public final class SystemProperties`
> **Versión:** `3.13.0` (constantes y métodos adicionales desde `3.15.0` hasta `3.20.0`)
> **Dependencias internas:** `org.apache.commons.lang3.function.Suppliers`, `org.apache.commons.lang3.ClassUtils`, `org.apache.commons.lang3.StringUtils`
> **Dependencias JDK:** `java.nio.file.Path`, `java.nio.file.Paths`, `java.util.function.*`

---

### 1. Resumen Ejecutivo

`SystemProperties` es una clase final de utilidad que centraliza el acceso tipado y seguro a las *System Properties* de la JVM. Expone más de 200 constantes `public static final String` con los nombres canónicos de propiedades documentadas por Oracle (Java 17/25) y provee métodos estáticos `get*()` que encapsulan `System.getProperty(String)` capturando `SecurityException` y retornando `null` o un valor por defecto suministrado vía `Supplier`. La clase es no instanciable por diseño (constructor público deprecado para ser privado en 4.0) y actúa como fachada de lectura segura para configuración de red, I/O, seguridad, XML, HTTP Client y entorno del sistema.

---

### 2. Especificación de Métodos y Constructores

#### 2.1 Constructor

##### `SystemProperties()`
Constructor público por defecto. No realiza inicialización. Marcado como `@Deprecated` con indicación de volverse `private` en la versión 4.0. La clase está diseñada para uso exclusivamente estático.

| Elemento | Descripción |
| :--- | :--- |
| **Firma** | `public SystemProperties()` |
| **Visibilidad** | `public` |
| **@param** | Ninguno |
| **@return** | Instancia de `SystemProperties` (uso desaconsejado) |
| **@throws** | Ninguno |
| **@deprecated** | TODO Make private in 4.0 |

```java
/**
 * @deprecated TODO Make private in 4.0.
 */
```

---

#### 2.2 Métodos de Infraestructura y Acceso Genérico

##### `getProperty(String property)`
Método base de lectura segura.

| @param | Nombre | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- | :--- |
| `@param` | `property` | `String` | Sí | Nombre de la propiedad del sistema. Si es `null` o vacío, se delega al `defaultIfAbsent`. |

| @return | Tipo | Descripción |
| :--- | :--- | :--- |
| `@return` | `String` | Valor de `System.getProperty(property)` o `null` si no existe o si se captura `SecurityException`. |

| @throws | Descripción |
| :--- | :--- |
| `SecurityException` | **Capturada internamente.** No se propaga; se retorna `null` (vía `Suppliers.nul()`). |

##### `getProperty(String property, String defaultIfAbsent)` - *package-private static*
Sobrecarga que permite valor por defecto `String`.

| @param | Nombre | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- | :--- |
| `@param` | `property` | `String` | Sí | Nombre de la propiedad. |
| `@param` | `defaultIfAbsent` | `String` | No | Valor retornado si la propiedad es `null`, vacía o hay `SecurityException`. |

| @return | Tipo | Descripción |
| :--- | :--- | :--- |
| `@return` | `String` | Valor de la propiedad o `defaultIfAbsent`. |

| @throws | Descripción |
| :--- | :--- |
| `SecurityException` | Capturada internamente, retorna `defaultIfAbsent`. |

##### `getProperty(String property, Supplier<String> defaultIfAbsent)` - *package-private static*
Sobrecarga genérica con `Supplier` perezoso. Es el método central invocado por todos los demás.

| @param | Nombre | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- | :--- |
| `@param` | `property` | `String` | Sí | Nombre de la propiedad. |
| `@param` | `defaultIfAbsent` | `Supplier<String>` | No | Proveedor invocado vía `StringUtils.getIfEmpty()` y `Suppliers.get()` si la propiedad está vacía o hay excepción. |

| @return | Tipo | Descripción |
| :--- | :--- | :--- |
| `@return` | `String` | Valor de la propiedad o resultado del `Supplier`. |

| @throws | Descripción |
| :--- | :--- |
| `SecurityException` | Capturada internamente, retorna `defaultIfAbsent.get()`. |

##### `getBoolean(String key, BooleanSupplier defaultIfAbsent)`
Parsea la propiedad como `boolean` según `Boolean.parseBoolean(String)`.

| @param | Nombre | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- | :--- |
| `@param` | `key` | `String` | Sí | Clave de la propiedad. |
| `@param` | `defaultIfAbsent` | `BooleanSupplier` | No | Proveedor del valor por defecto si `getProperty(key) == null`. Si es `null`, se retorna `false`. |

| @return | Tipo | Descripción |
| :--- | :--- | :--- |
| `@return` | `boolean` | `Boolean.parseBoolean(str)` si existe, o `defaultIfAbsent.getAsBoolean()` en caso contrario. |

| @throws | Descripción |
| :--- | :--- |
| `SecurityException` | Capturada en `getProperty`, tratada como propiedad ausente. |

##### `getBoolean(Class<?> clazz, String key, BooleanSupplier defaultIfAbsent)`
Variante con prefijo `SimpleClassName.Key`.

| @param | Nombre | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- | :--- |
| `@param` | `clazz` | `Class<?>` | Sí | Clase cuyo `getSimpleName()` se usa como prefijo. |
| `@param` | `key` | `String` | Sí | Sub-clave. |
| `@param` | `defaultIfAbsent` | `BooleanSupplier` | No | Valor por defecto. |

| @return | Tipo | Descripción |
| :--- | :--- | :--- |
| `@return` | `boolean` | Delega a `getBoolean(toKey(clazz, key, true), defaultIfAbsent)`. |

##### `getInt(String key, IntSupplier defaultIfAbsent)`
Parsea la propiedad como `int`.

| @param | Nombre | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- | :--- |
| `@param` | `key` | `String` | Sí | Clave. |
| `@param` | `defaultIfAbsent` | `IntSupplier` | No | Proveedor por defecto. Si es `null` y la propiedad no existe, retorna `0`. |

| @return | Tipo | Descripción |
| :--- | :--- | :--- |
| `@return` | `int` | `Integer.parseInt(str)` o valor del supplier. |

| @throws | Descripción |
| :--- | :--- |
| `NumberFormatException` | Si `str != null` y no es un entero válido. No capturada. |
| `SecurityException` | Capturada como ausencia. |

##### `getInt(Class<?> clazz, String key, IntSupplier defaultIfAbsent)`
| @param | Nombre | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- | :--- |
| `@param` | `clazz` | `Class<?>` | Sí | Clase para prefijo. |
| `@param` | `key` | `String` | Sí | Sub-clave. |
| `@param` | `defaultIfAbsent` | `IntSupplier` | No | Valor por defecto. |
| **@return** | `int` | Delega a `getInt(toKey(...), defaultIfAbsent)` |
| **@throws** | `NumberFormatException`, `SecurityException` (capturada) | |

##### `getLong(String key, LongSupplier defaultIfAbsent)`
| @param | Nombre | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- | :--- |
| `@param` | `key` | `String` | Sí | Clave. |
| `@param` | `defaultIfAbsent` | `LongSupplier` | No | Proveedor por defecto. Si es `null`, retorna `0L`. |
| **@return** | `long` | `Long.parseLong(str)` o valor del supplier. |
| **@throws** | `NumberFormatException` si el formato es inválido. `SecurityException` capturada. |

##### `getLong(Class<?> clazz, String key, LongSupplier defaultIfAbsent)`
| @param | Nombre | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- | :--- |
| `@param` | `clazz` | `Class<?>` | Sí | Clase para prefijo. |
| `@param` | `key` | `String` | Sí | Sub-clave. |
| `@param` | `defaultIfAbsent` | `LongSupplier` | No | Valor por defecto. |
| **@return** | `long` | Delega a `getLong(toKey(...), defaultIfAbsent)` |

##### `getPath(String key, Supplier<Path> defaultIfAbsent)` - *since 3.20.0*
Convierte la propiedad a `java.nio.file.Path` vía `Paths.get(String)`.

| @param | Nombre | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- | :--- |
| `@param` | `key` | `String` | Sí | Clave. |
| `@param` | `defaultIfAbsent` | `Supplier<Path>` | No | Proveedor si la propiedad es `null`. Evaluado vía `Suppliers.get()`. |

| @return | Tipo | Descripción |
| :--- | :--- | :--- |
| `@return` | `Path` | `Paths.get(str)` o valor del supplier. `null` si ambos son `null`. |

| @throws | Descripción |
| :--- | :--- |
| `InvalidPathException` | Si `str` no es una ruta válida. |
| `SecurityException` | Capturada como ausencia. |

##### `isPropertySet(String property)` - *since 3.18.0*
| @param | Nombre | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- | :--- |
| `@param` | `property` | `String` | Sí | Nombre de la propiedad. |
| **@return** | `boolean` | `true` si `getProperty(property) != null`, `false` en caso contrario o `SecurityException`. |
| **@throws** | Ninguno (SecurityException capturada) | |

##### `toKey(Class<?> clazz, String key, boolean simpleKey)` - *private static*
Construye clave compuesta `ClassName.key`.

| @param | Nombre | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- | :--- |
| `@param` | `clazz` | `Class<?>` | Sí | Clase base. Se usa `ClassUtils.getName(clazz, StringUtils.EMPTY, simpleKey)`. |
| `@param` | `key` | `String` | Sí | Sub-clave. |
| `@param` | `simpleKey` | `boolean` | Sí | `true` usa `getSimpleName()`, `false` usa nombre canónico. |

| @return | Tipo | Descripción |
| :--- | :--- | :--- |
| `@return` | `String` | Concatenación `className + "." + key`. |

| @throws | Descripción |
| :--- | :--- |
| Ninguno | |

---

#### 2.3 Métodos Getters Específicos (Sin Parámetros)

> **Patrón común para todos los métodos listados a continuación:**
> - **Firma:** `public static String getXxx()`
> - **@param:** Ninguno
> - **@return:** `String` - Valor de `getProperty(CONSTANTE_ASOCIADA)` o `null` si la propiedad está ausente o se captura `SecurityException`.
> - **@throws:** Ninguno propagado (`SecurityException` capturada internamente).
> - **@since:** `3.15.0` salvo que se indique otra versión o sin anotación (API base).

Se documentan **TODOS** los métodos detectados, manteniendo el nombre exacto:

| Método `get*()` | Constante Asociada | Notas |
| :--- | :--- | :--- |
| `getAppleAwtEnableTemplateImages()` | `APPLE_AWT_ENABLE_TEMPLATE_IMAGES` |  |
| `getAwtToolkit()` | `AWT_TOOLKIT` | `@Deprecated` |
| `getComSunJndiLdapObjectTrustSerialData()` | `COM_SUN_JNDI_LDAP_OBJECT_TRUST_SERIAL_DATA` |  |
| `getComSunNetHttpServerHttpServerProvider()` | `COM_SUN_NET_HTTP_SERVER_HTTP_SERVER_PROVIDER` |  |
| `getFileEncoding()` | `FILE_ENCODING` |  |
| `getFileSeparator()` | `FILE_SEPARATOR` |  |
| `getFtpNonProxyHost()` | `FTP_NON_PROXY_HOST` |  |
| `getFtpProxyHost()` | `FTP_PROXY_HOST` |  |
| `getFtpProxyPort()` | `FTP_PROXY_PORT` |  |
| `getHttpAgent()` | `HTTP_AGENT` |  |
| `getHttpAuthDigestCnonceRepeat()` | `HTTP_AUTH_DIGEST_CNONCE_REPEAT` |  |
| `getHttpAuthDigestReenabledAlgorithms()` | `HTTP_AUTH_DIGEST_RE_ENABLED_ALGORITHMS` |  |
| `getHttpAuthDigestValidateProxy()` | `HTTP_AUTH_DIGEST_VALIDATE_PROXY` |  |
| `getHttpAuthDigestValidateServer()` | `HTTP_AUTH_DIGEST_VALIDATE_SERVER` |  |
| `getHttpAuthNtlmDomain()` | `HTTP_AUTH_NTLM_DOMAIN` |  |
| `getHttpKeepAlive()` | `HTTP_KEEP_ALIVE` |  |
| `getHttpKeepAliveTimeProxy()` | `HTTP_KEEP_ALIVE_TIME_PROXY` |  |
| `getHttpKeepAliveTimeServer()` | `HTTP_KEEP_ALIVE_TIME_SERVER` |  |
| `getHttpMaxConnections()` | `HTTP_MAX_CONNECTIONS` |  |
| `getHttpMaxRedirects()` | `HTTP_MAX_REDIRECTS` |  |
| `getHttpNonProxyHosts()` | `HTTP_NON_PROXY_HOSTS` |  |
| `getHttpProxyHost()` | `HTTP_PROXY_HOST` |  |
| `getHttpProxyPort()` | `HTTP_PROXY_PORT` |  |
| `getHttpsProxyHost()` | `HTTPS_PROXY_HOST` |  |
| `getHttpsProxyPort()` | `HTTPS_PROXY_PORT` |  |
| `getJavaAwtFonts()` | `JAVA_AWT_FONTS` | `@Deprecated` |
| `getJavaAwtGraphicsenv()` | `JAVA_AWT_GRAPHICSENV` | `@Deprecated` |
| `getJavaAwtHeadless()` | `JAVA_AWT_HEADLESS` | `@Deprecated` |
| `getJavaAwtPrinterjob()` | `JAVA_AWT_PRINTERJOB` | `@Deprecated` |
| `getJavaClassPath()` | `JAVA_CLASS_PATH` |  |
| `getJavaClassVersion()` | `JAVA_CLASS_VERSION` |  |
| `getJavaCompiler()` | `JAVA_COMPILER` | `@Deprecated` |
| `getJavaContentHandlerPkgs()` | `JAVA_CONTENT_HANDLER_PKGS` |  |
| `getJavaEndorsedDirs()` | `JAVA_ENDORSED_DIRS` | `@Deprecated` |
| `getJavaExtDirs()` | `JAVA_EXT_DIRS` | `@Deprecated` |
| `getJavaHome()` | `JAVA_HOME` |  |
| `getJavaIoTmpdir()` | `JAVA_IO_TMPDIR` |  |
| `getJavaLibraryPath()` | `JAVA_LIBRARY_PATH` |  |
| `getJavaLocaleProviders()` | `JAVA_LOCALE_PROVIDERS` |  |
| `getJavaLocaleUseOldIsoCodes()` | `JAVA_LOCALE_USE_OLD_ISO_CODES` |  |
| `getJavaNetPreferIpv4Stack()` | `JAVA_NET_PREFER_IPV4_STACK` |  |
| `getJavaNetPreferIpv6Addresses()` | `JAVA_NET_PREFER_IPV6_ADDRESSES` |  |
| `getJavaNetSocksPassword()` | `JAVA_NET_SOCKS_PASSWORD` |  |
| `getJavaNetSocksUserName()` | `JAVA_NET_SOCKS_USER_NAME` |  |
| `getJavaNetUseSystemProxies()` | `JAVA_NET_USE_SYSTEM_PROXIES` |  |
| `getJavaNioChannelsDefaultThreadPoolInitialSize()` | `JAVA_NIO_CHANNELS_DEFAULT_THREAD_POOL_INITIAL_SIZE` |  |
| `getJavaNioChannelsDefaultThreadPoolThreadFactory()` | `JAVA_NIO_CHANNELS_DEFAULT_THREAD_POOL_THREAD_FACTORY` |  |
| `getJavaNioChannelsSpiAsynchronousChannelProvider()` | `JAVA_NIO_CHANNELS_SPI_ASYNCHRONOUS_CHANNEL_PROVIDER` |  |
| `getJavaNioChannelsSpiSelectorProvider()` | `JAVA_NIO_CHANNELS_SPI_SELECTOR_PROVIDER` |  |
| `getJavaNioFileSpiDefaultFileSystemProvider()` | `JAVA_NIO_FILE_SPI_DEFAULT_FILE_SYSTEM_PROVIDER` |  |
| `getJavaPropertiesDate()` | `JAVA_PROPERTIES_DATE` |  |
| `getJavaProtocolHandlerPkgs()` | `JAVA_PROTOCOL_HANDLER_PKGS` |  |
| `getJavaRmiServerCodebase()` | `JAVA_RMI_SERVER_CODEBASE` |  |
| `getJavaRmiServerHostName()` | `JAVA_RMI_SERVER_HOST_NAME` |  |
| `getJavaRmiServerRandomIds()` | `JAVA_RMI_SERVER_RANDOM_IDS` |  |
| `getJavaRmiServerRmiClassLoaderSpi()` | `JAVA_RMI_SERVER_RMI_CLASS_LOADER_SPI` |  |
| `getJavaRuntimeName()` | `JAVA_RUNTIME_NAME` |  |
| `getJavaRuntimeVersion()` | `JAVA_RUNTIME_VERSION` |  |
| `getJavaSecurityAuthLoginConfig()` | `JAVA_SECURITY_AUTH_LOGIN_CONFIG` |  |
| `getJavaSecurityManager()` | `JAVA_SECURITY_MANAGER` |  |
| `getJavaSpecificationMaintenanceVersion()` | `JAVA_SPECIFICATION_MAINTENANCE_VERSION` |  |
| `getJavaSpecificationName()` | `JAVA_SPECIFICATION_NAME` |  |
| `getJavaSpecificationVendor()` | `JAVA_SPECIFICATION_VENDOR` |  |
| `getJavaSpecificationVersion()` | `JAVA_SPECIFICATION_VERSION` | Sobrecarga sin parámetros |
| `getJavaSystemClassLoader()` | `JAVA_SYSTEM_CLASS_LOADER` |  |
| `getJavaTimeZoneDefaultZoneRulesProvider()` | `JAVA_TIME_ZONE_DEFAULT_ZONE_RULES_PROVIDER` |  |
| `getJavaUtilConcurrentForkJoinPoolCommonExceptionHandler()` | `JAVA_UTIL_CONCURRENT_FORK_JOIN_POOL_COMMON_EXCEPTION_HANDLER` |  |
| `getJavaUtilConcurrentForkJoinPoolCommonMaximumSpares()` | `JAVA_UTIL_CONCURRENT_FORK_JOIN_POOL_COMMON_MAXIMUM_SPARES` |  |
| `getJavaUtilConcurrentForkJoinPoolCommonParallelism()` | `JAVA_UTIL_CONCURRENT_FORK_JOIN_POOL_COMMON_PARALLELISM` |  |
| `getJavaUtilConcurrentForkJoinPoolCommonThreadFactory()` | `JAVA_UTIL_CONCURRENT_FORK_JOIN_POOL_COMMON_THREAD_FACTORY` |  |
| `getJavaUtilCurrencyData()` | `JAVA_UTIL_CURRENCY_DATA` |  |
| `getJavaUtilLoggingConfigClass()` | `JAVA_UTIL_LOGGING_CONFIG_CLASS` |  |
| `getJavaUtilLoggingConfigFile()` | `JAVA_UTIL_LOGGING_CONFIG_FILE` |  |
| `getJavaUtilLoggingSimpleFormatterFormat()` | `JAVA_UTIL_LOGGING_SIMPLE_FORMATTER_FORMAT` |  |
| `getJavaUtilPrefsPreferencesFactory()` | `JAVA_UTIL_PREFS_PREFERENCES_FACTORY` |  |
| `getJavaUtilPropertyResourceBundleEncoding()` | `JAVA_UTIL_PROPERTY_RESOURCE_BUNDLE_ENCODING` |  |
| `getJavaVendor()` | `JAVA_VENDOR` |  |
| `getJavaVendorUrl()` | `JAVA_VENDOR_URL` |  |
| `getJavaVendorVersion()` | `JAVA_VENDOR_VERSION` |  |
| `getJavaVersion()` | `JAVA_VERSION` |  |
| `getJavaVersionDate()` | `JAVA_VERSION_DATE` |  |
| `getJavaVmInfo()` | `JAVA_VM_INFO` |  |
| `getJavaVmName()` | `JAVA_VM_NAME` |  |
| `getJavaVmSpecificationName()` | `JAVA_VM_SPECIFICATION_NAME` |  |
| `getJavaVmSpecificationVendor()` | `JAVA_VM_SPECIFICATION_VENDOR` |  |
| `getJavaVmSpecificationVersion()` | `JAVA_VM_SPECIFICATION_VERSION` |  |
| `getJavaVmVendor()` | `JAVA_VM_VENDOR` |  |
| `getJavaVmVersion()` | `JAVA_VM_VERSION` |  |
| `getJavaXmlConfigFile()` | `JAVA_XML_CONFIG_FILE` |  |
| `getJavaxAccessibilityAssistiveTechnologies()` | `JAVAX_ACCESSIBILITY_ASSISTIVE_TECHNOLOGIES` |  |
| `getJavaxNetSslSessionCacheSize()` | `JAVAX_NET_SSL_SESSION_CACHE_SIZE` |  |
| `getJavaxRmiSslClientEnabledCipherSuites()` | `JAVAX_RMI_SSL_CLIENT_ENABLED_CIPHER_SUITES` |  |
| `getJavaxRmiSslClientEnabledProtocols()` | `JAVAX_RMI_SSL_CLIENT_ENABLED_PROTOCOLS` |  |
| `getJavaxSecurityAuthUseSubjectCredsOnly()` | `JAVAX_SECURITY_AUTH_USE_SUBJECT_CREDS_ONLY` |  |
| `getJavaxSmartCardIoTerminalFactoryDefaultType()` | `JAVAX_SMART_CARD_IO_TERMINAL_FACTORY_DEFAULT_TYPE` |  |
| `getJdbcDrivers()` | `JDBC_DRIVERS` |  |
| `getJdkHttpAuthProxyingDisabledSchemes()` | `JDK_HTTP_AUTH_PROXYING_DISABLED_SCHEMES` |  |
| `getJdkHttpAuthTunnelingDisabledSchemes()` | `JDK_HTTP_AUTH_TUNNELING_DISABLED_SCHEMES` |  |
| `getJdkHttpClientAllowRestrictedHeaders()` | `JDK_HTTP_CLIENT_ALLOW_RESTRICTED_HEADERS` |  |
| `getJdkHttpClientAuthRetryLimit()` | `JDK_HTTP_CLIENT_AUTH_RETRY_LIMIT` |  |
| `getJdkHttpClientBufSize()` | `JDK_HTTP_CLIENT_BUF_SIZE` |  |
| `getJdkHttpClientConnectionPoolSize()` | `JDK_HTTP_CLIENT_CONNECTION_POOL_SIZE` |  |
| `getJdkHttpClientConnectionWindowSize()` | `JDK_HTTP_CLIENT_CONNECTION_WINDOW_SIZE` |  |
| `getJdkHttpClientDisableRetryConnect()` | `JDK_HTTP_CLIENT_DISABLE_RETRY_CONNECT` |  |
| `getJdkHttpClientEnableAllMethodRetry()` | `JDK_HTTP_CLIENT_ENABLE_ALL_METHOD_RETRY` |  |
| `getJdkHttpClientEnablePush()` | `JDK_HTTP_CLIENT_ENABLE_PUSH` |  |
| `getJdkHttpClientHpackMaxHeaderTableSize()` | `JDK_HTTP_CLIENT_HPACK_MAX_HEADER_TABLE_SIZE` |  |
| `getJdkHttpClientHttpClientLog()` | `JDK_HTTP_CLIENT_HTTP_CLIENT_LOG` |  |
| `getJdkHttpClientKeepAliveTimeout()` | `JDK_HTTP_CLIENT_KEEP_ALIVE_TIMEOUT` |  |
| `getJdkHttpClientKeepAliveTimeoutH2()` | `JDK_HTTP_CLIENT_KEEP_ALIVE_TIMEOUT_H2` |  |
| `getJdkHttpClientMaxFrameSize()` | `JDK_HTTP_CLIENT_MAX_FRAME_SIZE` |  |
| `getJdkHttpClientMaxStreams()` | `JDK_HTTP_CLIENT_MAX_STREAMS` |  |
| `getJdkHttpClientReceiveBufferSize()` | `JDK_HTTP_CLIENT_RECEIVE_BUFFER_SIZE` |  |
| `getJdkHttpClientRedirectsRetryLimit()` | `JDK_HTTP_CLIENT_REDIRECTS_RETRY_LIMIT` |  |
| `getJdkHttpClientSendBufferSize()` | `JDK_HTTP_CLIENT_SEND_BUFFER_SIZE` |  |
| `getJdkHttpClientWebSocketWriteBufferSize()` | `JDK_HTTP_CLIENT_WEB_SOCKET_WRITE_BUFFER_SIZE` |  |
| `getJdkHttpClientWindowSize()` | `JDK_HTTP_CLIENT_WINDOW_SIZE` |  |
| `getJdkHttpServerMaxConnections()` | `JDK_HTTP_SERVER_MAX_CONNECTIONS` |  |
| `getJdkHttpsNegotiateCbt()` | `JDK_HTTPS_NEGOTIATE_CBT` |  |
| `getJdkIncludeInExceptions()` | `JDK_INCLUDE_IN_EXCEPTIONS` |  |
| `getJdkInternalHttpClientDisableHostNameVerification()` | `JDK_INTERNAL_HTTP_CLIENT_DISABLE_HOST_NAME_VERIFICATION` |  |
| `getJdkIoPermissionsUseCanonicalPath()` | `JDK_IO_PERMISSIONS_USE_CANONICAL_PATH` |  |
| `getJdkJndiLdapObjectFactoriesFilter()` | `JDK_JNDI_LDAP_OBJECT_FACTORIES_FILTER` |  |
| `getJdkJndiObjectFactoriesFilter()` | `JDK_JNDI_OBJECT_FACTORIES_FILTER` |  |
| `getJdkJndiRmiObjectFactoriesFilter()` | `JDK_JNDI_RMI_OBJECT_FACTORIES_FILTER` |  |
| `getJdkModuleMain()` | `JDK_MODULE_MAIN` |  |
| `getJdkModuleMainClass()` | `JDK_MODULE_MAIN_CLASS` |  |
| `getJdkModulePath()` | `JDK_MODULE_PATH` |  |
| `getJdkModuleUpgradePath()` | `JDK_MODULE_UPGRADE_PATH` |  |
| `getJdkNetUnixDomainTmpDir()` | `JDK_NET_UNIX_DOMAIN_TMPDIR` |  |
| `getJdkNetUrlClassPathShowIgnoredClassPathEntries()` | `JDK_NET_URL_CLASS_PATH_SHOW_IGNORED_CLASS_PATH_ENTRIES` |  |
| `getJdkSerialFilter()` | `JDK_SERIAL_FILTER` |  |
| `getJdkSerialFilterFactory()` | `JDK_SERIAL_FILTER_FACTORY` |  |
| `getJdkTlsClientSignatureSchemes()` | `JDK_TLS_CLIENT_SIGNATURE_SCHEMES` |  |
| `getJdkTlsNamedGroups()` | `JDK_TLS_NAMED_GROUPS` |  |
| `getJdkTlsServerSignatureSchemes()` | `JDK_TLS_SERVER_SIGNATURE_SCHEMES` |  |
| `getJdkVirtualThreadSchedulerMaxPoolSize()` | `JDK_VIRTUAL_THREAD_SCHEDULER_MAXPOOLSIZE` |  |
| `getJdkVirtualThreadSchedulerParallelism()` | `JDK_VIRTUAL_THREAD_SCHEDULER_PARALLELISM` |  |
| `getJdkXmlCdataChunkSize()` | `JDK_XML_CDATA_CHUNK_SIZE` |  |
| `getJdkXmlDtdSupport()` | `JDK_XML_DTD_SUPPORT` |  |
| `getJdkXmlElementAttributeLimit()` | `JDK_XML_ELEMENT_ATTRIBUTE_LIMIT` |  |
| `getJdkXmlEnableExtensionFunctions()` | `JDK_XML_ENABLE_EXTENSION_FUNCTIONS` |  |
| `getJdkXmlEntityExpansionLimit()` | `JDK_XML_ENTITY_EXPANSION_LIMIT` |  |
| `getJdkXmlEntityReplacementLimit()` | `JDK_XML_ENTITY_REPLACEMENT_LIMIT` |  |
| `getJdkXmlIsStandalone()` | `JDK_XML_IS_STANDALONE` |  |
| `getJdkXmlJdkCatalogResolve()` | `JDK_XML_JDK_CATALOG_RESOLVE` |  |
| `getJdkXmlMaxElementDepth()` | `JDK_XML_MAX_ELEMENT_DEPTH` |  |
| `getJdkXmlMaxGeneralEntitySizeLimit()` | `JDK_XML_MAX_GENERAL_ENTITY_SIZE_LIMIT` |  |
| `getJdkXmlMaxOccurLimit()` | `JDK_XML_MAX_OCCUR_LIMIT` |  |
| `getJdkXmlMaxParameterEntitySizeLimit()` | `JDK_XML_MAX_PARAMETER_ENTITY_SIZE_LIMIT` |  |
| `getJdkXmlMaxXmlNameLimit()` | `JDK_XML_MAX_XML_NAME_LIMIT` |  |
| `getJdkXmlOverrideDefaultParser()` | `JDK_XML_OVERRIDE_DEFAULT_PARSER` |  |
| `getJdkXmlResetSymbolTable()` | `JDK_XML_RESET_SYMBOL_TABLE` |  |
| `getJdkXmlTotalEntitySizeLimit()` | `JDK_XML_TOTAL_ENTITY_SIZE_LIMIT` |  |
| `getJdkXmlXsltcIsStandalone()` | `JDK_XML_XSLTC_IS_STANDALONE` |  |
| `getLineSeparator()` | `LINE_SEPARATOR` | Sobrecarga sin parámetros |
| `getNativeEncoding()` | `NATIVE_ENCODING` |  |
| `getNetworkAddressCacheNegativeTtl()` | `NETWORK_ADDRESS_CACHE_NEGATIVE_TTL` |  |
| `getNetworkAddressCacheStaleTtl()` | `NETWORK_ADDRESS_CACHE_STALE_TTL` |  |
| `getNetworkAddressCacheTtl()` | `NETWORK_ADDRESS_CACHE_TTL` |  |
| `getOrgJcpXmlDsigSecureValidation()` | `ORG_JCP_XML_DSIG_SECURE_VALIDATION` |  |
| `getOrgOpenJdkJavaUtilStreamTripwire()` | `ORG_OPENJDK_JAVA_UTIL_STREAM_TRIPWIRE` |  |
| `getOsArch()` | `OS_ARCH` |  |
| `getOsName()` | `OS_NAME` |  |
| `getOsVersion()` | `OS_VERSION` |  |
| `getPathSeparator()` | `PATH_SEPARATOR` |  |
| `getSocksProxyHost()` | `SOCKS_PROXY_HOST` |  |
| `getSocksProxyPort()` | `SOCKS_PROXY_PORT` |  |
| `getSocksProxyVersion()` | `SOCKS_PROXY_VERSION` |  |
| `getStdErrEncoding()` | `STDERR_ENCODING` |  |
| `getStdOutEncoding()` | `STDOUT_ENCODING` |  |
| `getSunNetHttpServerDrainAmount()` | `SUN_NET_HTTP_SERVER_DRAIN_AMOUNT` |  |
| `getSunNetHttpServerIdleInterval()` | `SUN_NET_HTTP_SERVER_IDLE_INTERVAL` |  |
| `getSunNetHttpServerMaxIdleConnections()` | `SUN_NET_HTTP_SERVER_MAX_IDLE_CONNECTIONS` |  |
| `getSunNetHttpServerMaxReqHeaders()` | `SUN_NET_HTTP_SERVER_MAX_REQ_HEADERS` |  |
| `getSunNetHttpServerMaxReqTime()` | `SUN_NET_HTTP_SERVER_MAX_REQ_TIME` |  |
| `getSunNetHttpServerMaxRspTime()` | `SUN_NET_HTTP_SERVER_MAX_RSP_TIME` |  |
| `getSunNetHttpServerNoDelay()` | `SUN_NET_HTTP_SERVER_NO_DELAY` |  |
| `getSunSecurityKrb5Principal()` | `SUN_SECURITY_KRB5_PRINCIPAL` |  |
| `getUserCountry()` | `USER_COUNTRY` |  |
| `getUserDir()` | `USER_DIR` |  |
| `getUserExtensions()` | `USER_EXTENSIONS` |  |
| `getUserHome()` | `USER_HOME` |  |
| `getUserLanguage()` | `USER_LANGUAGE` |  |
| `getUserName()` | `USER_NAME` | Sobrecarga sin parámetros |
| `getUserRegion()` | `USER_REGION` |  |
| `getUserScript()` | `USER_SCRIPT` |  |
| `getUserTimezone()` | `USER_TIMEZONE` |  |
| `getUserVariant()` | `USER_VARIANT` |  |

#### 2.4 Métodos Sobrecargados con Parámetro por Defecto

##### `getJavaSpecificationVersion(String defaultValue)`
| @param | Nombre | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- | :--- |
| `@param` | `defaultValue` | `String` | No | Valor retornado si `JAVA_SPECIFICATION_VERSION` está vacía o hay `SecurityException`. |
| **@return** | `String` | Valor de la propiedad o `defaultValue`. |
| **@throws** | Ninguno propagado | |

##### `getLineSeparator(Supplier<String> defaultIfAbsent)` - *since 3.15.0*
| @param | Nombre | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- | :--- |
| `@param` | `defaultIfAbsent` | `Supplier<String>` | No | Proveedor si la propiedad está vacía o hay `SecurityException`. |
| **@return** | `String` | Valor de `LINE_SEPARATOR` o resultado del supplier. |

##### `getUserName(String defaultValue)` - *since 3.15.0*
| @param | Nombre | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- | :--- |
| `@param` | `defaultValue` | `String` | No | Valor por defecto. |
| **@return** | `String` | Valor de `USER_NAME` o `defaultValue`. |

> **Nota sobre clases/métodos anónimos o internos:** El código fuente analizado no declara clases internas, anónimas ni métodos `protected`/`private` adicionales más allá de `toKey()` y las sobrecargas `getProperty()` ya documentadas. No se detectaron lambdas con cuerpo de método documentable.

---

### 3. Ejemplo de Uso

```java
import org.apache.commons.lang3.SystemProperties;
import java.nio.file.Path;

public class SystemPropertiesExample {

    // Clase de ejemplo para demostrar getBoolean/getInt con prefijo
    static class MyService {}

    public static void main(String[] args) {
        // 1. Lectura directa tipada - getters sin parámetros (retornan null si hay SecurityException)
        String javaHome = SystemProperties.getJavaHome();
        String osName = SystemProperties.getOsName();
        String fileEncoding = SystemProperties.getFileEncoding();
        String lineSep = SystemProperties.getLineSeparator(() -> System.lineSeparator());

        System.out.printf("Java Home: %s%nOS: %s%nEncoding: %s%n", javaHome, osName, fileEncoding);

        // 2. Lectura con valor por defecto perezoso
        String specVersion = SystemProperties.getJavaSpecificationVersion("17");
        String user = SystemProperties.getUserName("unknown-user");

        // 3. Lectura tipada boolean/int/long con clave genérica
        // Propiedad: -DMyService.enableCache=true
        boolean enableCache = SystemProperties.getBoolean(MyService.class, "enableCache", () -> true);
        
        // Propiedad: -Dapp.maxRetries=5  (o -DMyService.maxRetries=5)
        int maxRetries = SystemProperties.getInt("app.maxRetries", () -> 3);
        long timeout = SystemProperties.getLong(MyService.class, "timeoutMs", () -> 5000L);

        // 4. Lectura como Path (since 3.20.0)
        Path tmpDir = SystemProperties.getPath(SystemProperties.JAVA_IO_TMPDIR, () -> Path.of("/tmp"));

        // 5. Verificación de existencia sin lanzar excepción
        if (SystemProperties.isPropertySet(SystemProperties.JDK_SERIAL_FILTER)) {
            String filter = SystemProperties.getJdkSerialFilter();
            System.out.println("Serial Filter: " + filter);
        }

        // 6. Acceso a propiedades de red/proxy
        String proxyHost = SystemProperties.getHttpProxyHost();
        String proxyPort = SystemProperties.getHttpProxyPort();
        if (proxyHost != null) {
            System.out.printf("Proxy HTTP: %s:%s%n", proxyHost, proxyPort);
        }
    }
}
```

**Buenas prácticas:**
- Preferir los getters específicos (`getJavaHome()`, `getOsName()`) sobre `getProperty(String)` para evitar errores de tipeo en el nombre de la propiedad.
- Usar las sobrecargas con `Supplier`/`IntSupplier`/`LongSupplier` para valores por defecto costosos o que requieren cómputo perezoso.
- Validar `null` en todos los getters sin defecto, ya que `SecurityManager` puede bloquear el acceso.
- Para propiedades custom de aplicación, usar `getBoolean(Class, String, ...)` / `getInt(Class, String, ...)` que generan claves del tipo `MyService.enableCache`.

