¡Excelente! Procedo a actuar como Arquitecto de Software Senior y Redactor Técnico para documentar el código Java proporcionado.

A continuación, se presenta la documentación técnica exhaustiva de la clase `org.apache.commons.lang3.SystemProperties`.

***

### Documentación Técnica: `org.apache.commons.lang3.SystemProperties`

**Autor:** Apache Software Foundation (análisis y documentación por un Arquitecto de Software Senior)

---

### 1. Resumen Ejecutivo

La clase `SystemProperties` es una utilidad final que centraliza y facilita el acceso a las propiedades del sistema de Java (`System.getProperty`). Proporciona un conjunto exhaustivo de constantes `public static final` para los nombres de propiedades estándar, evitando errores de escritura y mejorando la legibilidad del código. Adicionalmente, ofrece métodos `get` que recuperan los valores de estas propiedades de forma segura, manejando internamente las `SecurityException` y devolviendo `null` o un valor por defecto en lugar de lanzar una excepción.

---

### 2. Especificación de Métodos y Constructores

A continuación se detallan todos los constructores y métodos de la clase `SystemProperties`.

#### **Constructor**

##### `SystemProperties()`

Constructor público deprecado. La clase está diseñada como una utilidad con métodos estáticos y no para ser instanciada. Se planea que este constructor sea privado en futuras versiones.

| Atributo | Descripción |
| :--- | :--- |
| **Visibilidad** | `public` |
| **Deprecado** | Sí. Se recomienda no instanciar la clase. |

<br/>

#### **Métodos**

##### `getBoolean(final Class<?> clazz, final String key, final BooleanSupplier defaultIfAbsent)`

Obtiene el valor de una propiedad del sistema, construida como `"NombreSimpleDeClase.key"`, y la convierte a un `boolean`.

*   **Parámetros de Entrada (`@param`)**
| Nombre | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `clazz` | `Class<?>` | Sí | La clase cuyo nombre simple se usará como prefijo para la clave de la propiedad. |
| `key` | `String` | Sí | La sub-clave que se añadirá al nombre de la clase para formar la clave completa. |
| `defaultIfAbsent` | `BooleanSupplier` | Sí | Proveedor del valor por defecto a devolver si la propiedad no existe o no se puede acceder. |

*   **Valor de Retorno (`@return`)**
| Tipo | Descripción |
| :--- | :--- |
| `boolean` | El valor de la propiedad parseado como `boolean` (según `Boolean.parseBoolean(String)`), o el valor del proveedor por defecto si la propiedad no se encuentra. |

<br/>

##### `getBoolean(final String key, final BooleanSupplier defaultIfAbsent)`

Obtiene el valor de una propiedad del sistema y la convierte a un `boolean`.

*   **Parámetros de Entrada (`@param`)**
| Nombre | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `key` | `String` | Sí | La clave de la propiedad del sistema. |
| `defaultIfAbsent` | `BooleanSupplier` | Sí | Proveedor del valor por defecto a devolver si la propiedad no existe o no se puede acceder. |

*   **Valor de Retorno (`@return`)**
| Tipo | Descripción |
| :--- | :--- |
| `boolean` | El valor de la propiedad parseado como `boolean` (según `Boolean.parseBoolean(String)`), o el valor del proveedor por defecto si la propiedad no se encuentra. |

<br/>

##### `getInt(final Class<?> clazz, final String key, final IntSupplier defaultIfAbsent)`

Obtiene el valor de una propiedad del sistema, construida como `"NombreSimpleDeClase.key"`, y la convierte a un `int`.

*   **Parámetros de Entrada (`@param`)**
| Nombre | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `clazz` | `Class<?>` | Sí | La clase cuyo nombre simple se usará como prefijo para la clave de la propiedad. |
| `key` | `String` | Sí | La sub-clave que se añadirá al nombre de la clase para formar la clave completa. |
| `defaultIfAbsent` | `IntSupplier` | Sí | Proveedor del valor por defecto a devolver si la propiedad no existe o no se puede acceder. |

*   **Valor de Retorno (`@return`)**
| Tipo | Descripción |
| :--- | :--- |
| `int` | El valor de la propiedad parseado como `int`, o el valor del proveedor por defecto si la propiedad no se encuentra. |

*   **Excepciones (`@throws`)**
| Tipo | Condición |
| :--- | :--- |
| `NumberFormatException` | Si el valor de la propiedad existe pero no es un entero parseable. |

<br/>

##### `getInt(final String key, final IntSupplier defaultIfAbsent)`

Obtiene el valor de una propiedad del sistema y la convierte a un `int`.

*   **Parámetros de Entrada (`@param`)**
| Nombre | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `key` | `String` | Sí | La clave de la propiedad del sistema. |
| `defaultIfAbsent` | `IntSupplier` | Sí | Proveedor del valor por defecto a devolver si la propiedad no existe o no se puede acceder. |

*   **Valor de Retorno (`@return`)**
| Tipo | Descripción |
| :--- | :--- |
| `int` | El valor de la propiedad parseado como `int`, o el valor del proveedor por defecto si la propiedad no se encuentra. |

*   **Excepciones (`@throws`)**
| Tipo | Condición |
| :--- | :--- |
| `NumberFormatException` | Si el valor de la propiedad existe pero no es un entero parseable. |

<br/>

##### `getLong(final Class<?> clazz, final String key, final LongSupplier defaultIfAbsent)`

Obtiene el valor de una propiedad del sistema, construida como `"NombreSimpleDeClase.key"`, y la convierte a un `long`.

*   **Parámetros de Entrada (`@param`)**
| Nombre | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `clazz` | `Class<?>` | Sí | La clase cuyo nombre simple se usará como prefijo para la clave de la propiedad. |
| `key` | `String` | Sí | La sub-clave que se añadirá al nombre de la clase para formar la clave completa. |
| `defaultIfAbsent` | `LongSupplier` | Sí | Proveedor del valor por defecto a devolver si la propiedad no existe o no se puede acceder. |

*   **Valor de Retorno (`@return`)**
| Tipo | Descripción |
| :--- | :--- |
| `long` | El valor de la propiedad parseado como `long`, o el valor del proveedor por defecto si la propiedad no se encuentra. |

*   **Excepciones (`@throws`)**
| Tipo | Condición |
| :--- | :--- |
| `NumberFormatException` | Si el valor de la propiedad existe pero no es un `long` parseable. |

<br/>

##### `getLong(final String key, final LongSupplier defaultIfAbsent)`

Obtiene el valor de una propiedad del sistema y la convierte a un `long`.

*   **Parámetros de Entrada (`@param`)**
| Nombre | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `key` | `String` | Sí | La clave de la propiedad del sistema. |
| `defaultIfAbsent` | `LongSupplier` | Sí | Proveedor del valor por defecto a devolver si la propiedad no existe o no se puede acceder. |

*   **Valor de Retorno (`@return`)**
| Tipo | Descripción |
| :--- | :--- |
| `long` | El valor de la propiedad parseado como `long`, o el valor del proveedor por defecto si la propiedad no se encuentra. |

*   **Excepciones (`@throws`)**
| Tipo | Condición |
| :--- | :--- |
| `NumberFormatException` | Si el valor de la propiedad existe pero no es un `long` parseable. |

<br/>

##### `getPath(final String key, final Supplier<Path> defaultIfAbsent)`

Obtiene el valor de una propiedad del sistema y la convierte a un objeto `java.nio.file.Path`.

*   **Parámetros de Entrada (`@param`)**
| Nombre | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `key` | `String` | Sí | La clave de la propiedad del sistema. |
| `defaultIfAbsent` | `Supplier<Path>` | Sí | Proveedor del valor por defecto a devolver si la propiedad no existe o no se puede acceder. |

*   **Valor de Retorno (`@return`)**
| Tipo | Descripción |
| :--- | :--- |
| `Path` | El objeto `Path` creado a partir del valor de la propiedad, o el valor del proveedor por defecto. |

<br/>

##### `getProperty(final String property)`

Obtiene el valor de una propiedad del sistema. Si la propiedad no se puede leer debido a una `SecurityException` o no existe, devuelve `null`.

*   **Parámetros de Entrada (`@param`)**
| Nombre | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `property` | `String` | Sí | El nombre de la propiedad del sistema a obtener. |

*   **Valor de Retorno (`@return`)**
| Tipo | Descripción |
| :--- | :--- |
| `String` | El valor de la propiedad, o `null` si no existe o no se puede acceder. |

<br/>

##### `getProperty(final String property, final String defaultIfAbsent)`

Obtiene el valor de una propiedad del sistema. Si la propiedad no se puede leer, está vacía o no existe, devuelve el valor por defecto proporcionado.

*   **Parámetros de Entrada (`@param`)**
| Nombre | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `property` | `String` | Sí | El nombre de la propiedad del sistema a obtener. |
| `defaultIfAbsent` | `String` | Sí | Valor a devolver si la propiedad no se encuentra o no es accesible. |

*   **Valor de Retorno (`@return`)**
| Tipo | Descripción |
| :--- | :--- |
| `String` | El valor de la propiedad, o el valor por defecto. |

<br/>

##### `getProperty(final String property, final Supplier<String> defaultIfAbsent)`

Obtiene el valor de una propiedad del sistema. Si la propiedad no se puede leer, está vacía o no existe, invoca al proveedor para obtener un valor por defecto.

*   **Parámetros de Entrada (`@param`)**
| Nombre | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `property` | `String` | Sí | El nombre de la propiedad del sistema a obtener. |
| `defaultIfAbsent` | `Supplier<String>` | Sí | Proveedor del valor por defecto a devolver si la propiedad no se encuentra o no es accesible. |

*   **Valor de Retorno (`@return`)**
| Tipo | Descripción |
| :--- | :--- |
| `String` | El valor de la propiedad, o el valor obtenido del proveedor por defecto. |

<br/>

##### `isPropertySet(final String property)`

Verifica si una propiedad del sistema está definida (no es `null`).

*   **Parámetros de Entrada (`@param`)**
| Nombre | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `property` | `String` | Sí | El nombre de la propiedad del sistema a verificar. |

*   **Valor de Retorno (`@return`)**
| Tipo | Descripción |
| :--- | :--- |
| `boolean` | `true` si la propiedad está definida y es accesible, `false` en caso contrario. |

<br/>

##### `toKey(final Class<?> clazz, final String key, final boolean simpleKey)`

Método de utilidad privado para construir una clave de propiedad a partir del nombre de una clase y una sub-clave.

*   **Parámetros de Entrada (`@param`)**
| Nombre | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `clazz` | `Class<?>` | Sí | La clase base para el nombre de la clave. |
| `key` | `String` | Sí | La sub-clave a concatenar. |
| `simpleKey` | `boolean` | Sí | Si es `true`, usa `clazz.getSimpleName()`; de lo contrario, usa el nombre completo. |

*   **Valor de Retorno (`@return`)**
| Tipo | Descripción |
| :--- | :--- |
| `String` | La clave de propiedad generada con el formato `NombreClase.key`. |

<br/>

#### **Métodos `get` Específicos de Propiedades**

Estos métodos son atajos para obtener propiedades específicas. Todos son `public static`, no reciben parámetros y devuelven un `String` con el valor de la propiedad o `null` si no existe o no es accesible.

| Nombre del Método | Propiedad Obtenida (`System.getProperty(...)`) |
| :--- | :--- |
| `getAppleAwtEnableTemplateImages()` | `apple.awt.enableTemplateImages` |
| `getAwtToolkit()` (deprecado) | `awt.toolkit` |
| `getComSunJndiLdapObjectTrustSerialData()` | `com.sun.jndi.ldap.object.trustSerialData` |
| `getComSunNetHttpServerHttpServerProvider()` | `com.sun.net.httpserver.HttpServerProvider` |
| `getFileEncoding()` | `file.encoding` |
| `getFileSeparator()` | `file.separator` |
| `getFtpNonProxyHost()` | `ftp.nonProxyHosts` |
| `getFtpProxyHost()` | `ftp.proxyHost` |
| `getFtpProxyPort()` | `ftp.proxyPort` |
| `getHttpAgent()` | `http.agent` |
| `getHttpAuthDigestCnonceRepeat()` | `http.auth.digest.cnonceRepeat` |
| `getHttpAuthDigestReenabledAlgorithms()` | `http.auth.digest.reEnabledAlgorithms` |
| `getHttpAuthDigestValidateProxy()` | `http.auth.digest.validateProxy` |
| `getHttpAuthDigestValidateServer()` | `http.auth.digest.validateServer` |
| `getHttpAuthNtlmDomain()` | `http.auth.ntlm.domain` |
| `getHttpKeepAlive()` | `http.keepAlive` |
| `getHttpKeepAliveTimeProxy()` | `http.keepAlive.time.proxy` |
| `getHttpKeepAliveTimeServer()` | `http.keepAlive.time.server` |
| `getHttpMaxConnections()` | `http.maxConnections` |
| `getHttpMaxRedirects()` | `http.maxRedirects` |
| `getHttpNonProxyHosts()` | `http.nonProxyHosts` |
| `getHttpProxyHost()` | `http.proxyHost` |
| `getHttpProxyPort()` | `http.proxyPort` |
| `getHttpsProxyHost()` | `https.proxyHost` |
| `getHttpsProxyPort()` | `https.proxyPort` |
| `getJavaAwtFonts()` (deprecado) | `java.awt.fonts` |
| `getJavaAwtGraphicsenv()` (deprecado) | `java.awt.graphicsenv` |
| `getJavaAwtHeadless()` (deprecado) | `java.awt.headless` |
| `getJavaAwtPrinterjob()` (deprecado) | `java.awt.printerjob` |
| `getJavaClassPath()` | `java.class.path` |
| `getJavaClassVersion()` | `java.class.version` |
| `getJavaCompiler()` (deprecado) | `java.compiler` |
| `getJavaContentHandlerPkgs()` | `java.content.handler.pkgs` |
| `getJavaEndorsedDirs()` (deprecado) | `java.endorsed.dirs` |
| `getJavaExtDirs()` (deprecado) | `java.ext.dirs` |
| `getJavaHome()` | `java.home` |
| `getJavaIoTmpdir()` | `java.io.tmpdir` |
| `getJavaLibraryPath()` | `java.library.path` |
| `getJavaLocaleProviders()` | `java.locale.providers` |
| `getJavaLocaleUseOldIsoCodes()` | `java.locale.useOldISOCodes` |
| `getJavaNetPreferIpv4Stack()` | `java.net.preferIPv4Stack` |
| `getJavaNetPreferIpv6Addresses()` | `java.net.preferIPv6Addresses` |
| `getJavaNetSocksPassword()` | `java.net.socks.password` |
| `getJavaNetSocksUserName()` | `java.net.socks.username` |
| `getJavaNetUseSystemProxies()` | `java.net.useSystemProxies` |
| `getJavaNioChannelsDefaultThreadPoolInitialSize()` | `java.nio.channels.DefaultThreadPool.initialSize` |
| `getJavaNioChannelsDefaultThreadPoolThreadFactory()` | `java.nio.channels.DefaultThreadPool.threadFactory` |
| `getJavaNioChannelsSpiAsynchronousChannelProvider()` | `java.nio.channels.spi.AsynchronousChannelProvider` |
| `getJavaNioChannelsSpiSelectorProvider()` | `java.nio.channels.spi.SelectorProvider` |
| `getJavaNioFileSpiDefaultFileSystemProvider()` | `java.nio.file.spi.DefaultFileSystemProvider` |
| `getJavaPropertiesDate()` | `java.properties.date` |
| `getJavaProtocolHandlerPkgs()` | `java.protocol.handler.pkgs` |
| `getJavaRmiServerCodebase()` | `java.rmi.server.codebase` |
| `getJavaRmiServerHostName()` | `java.rmi.server.hostname` |
| `getJavaRmiServerRandomIds()` | `java.rmi.server.randomIDs` |
| `getJavaRmiServerRmiClassLoaderSpi()` | `java.rmi.server.RMIClassLoaderSpi` |
| `getJavaRuntimeName()` | `java.runtime.name` |
| `getJavaRuntimeVersion()` | `java.runtime.version` |
| `getJavaSecurityAuthLoginConfig()` | `java.security.auth.login.config` |
| `getJavaSecurityManager()` | `java.security.manager` |
| `getJavaSpecificationMaintenanceVersion()` | `java.specification.maintenance.version` |
| `getJavaSpecificationName()` | `java.specification.name` |
| `getJavaSpecificationVendor()` | `java.specification.vendor` |
| `getJavaSpecificationVersion()` | `java.specification.version` |
| `getJavaSpecificationVersion(String)` | `java.specification.version` (con valor por defecto) |
| `getJavaSystemClassLoader()` | `java.system.class.loader` |
| `getJavaTimeZoneDefaultZoneRulesProvider()` | `java.time.zone.DefaultZoneRulesProvider` |
| `getJavaUtilConcurrentForkJoinPoolCommonExceptionHandler()` | `java.util.concurrent.ForkJoinPool.common.exceptionHandler` |
| `getJavaUtilConcurrentForkJoinPoolCommonMaximumSpares()` | `java.util.concurrent.ForkJoinPool.common.maximumSpares` |
| `getJavaUtilConcurrentForkJoinPoolCommonParallelism()` | `java.util.concurrent.ForkJoinPool.common.parallelism` |
| `getJavaUtilConcurrentForkJoinPoolCommonThreadFactory()` | `java.util.concurrent.ForkJoinPool.common.threadFactory` |
| `getJavaUtilCurrencyData()` | `java.util.currency.data` |
| `getJavaUtilLoggingConfigClass()` | `java.util.logging.config.class` |
| `getJavaUtilLoggingConfigFile()` | `java.util.logging.config.file` |
| `getJavaUtilLoggingSimpleFormatterFormat()` | `java.util.logging.simpleformatter.format` |
| `getJavaUtilPrefsPreferencesFactory()` | `java.util.prefs.PreferencesFactory` |
| `getJavaUtilPropertyResourceBundleEncoding()` | `java.util.PropertyResourceBundle.encoding` |
| `getJavaVendor()` | `java.vendor` |
| `getJavaVendorUrl()` | `java.vendor.url` |
| `getJavaVendorVersion()` | `java.vendor.version` |
| `getJavaVersion()` | `java.version` |
| `getJavaVersionDate()` | `java.version.date` |
| `getJavaVmInfo()` | `java.vm.info` |
| `getJavaVmName()` | `java.vm.name` |
| `getJavaVmSpecificationName()` | `java.vm.specification.name` |
| `getJavaVmSpecificationVendor()` | `java.vm.specification.vendor` |
| `getJavaVmSpecificationVersion()` | `java.vm.specification.version` |
| `getJavaVmVendor()` | `java.vm.vendor` |
| `getJavaVmVersion()` | `java.vm.version` |
| `getJavaxAccessibilityAssistiveTechnologies()` | `javax.accessibility.assistive_technologies` |
| `getJavaXmlConfigFile()` | `java.xml.config.file` |
| `getJavaxNetSslSessionCacheSize()` | `javax.net.ssl.sessionCacheSize` |
| `getJavaxRmiSslClientEnabledCipherSuites()` | `javax.rmi.ssl.client.enabledCipherSuites` |
| `getJavaxRmiSslClientEnabledProtocols()` | `javax.rmi.ssl.client.enabledProtocols` |
| `getJavaxSecurityAuthUseSubjectCredsOnly()` | `javax.security.auth.useSubjectCredsOnly` |
| `getJavaxSmartCardIoTerminalFactoryDefaultType()` | `javax.smartcardio.TerminalFactory.DefaultType` |
| `getJdbcDrivers()` | `jdbc.drivers` |
| `getJdkHttpAuthProxyingDisabledSchemes()` | `jdk.http.auth.proxying.disabledSchemes` |
| `getJdkHttpAuthTunnelingDisabledSchemes()` | `jdk.http.auth.tunneling.disabledSchemes` |
| `getJdkHttpClientAllowRestrictedHeaders()` | `jdk.httpclient.allowRestrictedHeaders` |
| `getJdkHttpClientAuthRetryLimit()` | `jdk.httpclient.auth.retrylimit` |
| `getJdkHttpClientBufSize()` | `jdk.httpclient.bufsize` |
| `getJdkHttpClientConnectionPoolSize()` | `jdk.httpclient.connectionPoolSize` |
| `getJdkHttpClientConnectionWindowSize()` | `jdk.httpclient.connectionWindowSize` |
| `getJdkHttpClientDisableRetryConnect()` | `jdk.httpclient.disableRetryConnect` |
| `getJdkHttpClientEnableAllMethodRetry()` | `jdk.httpclient.enableAllMethodRetry` |
| `getJdkHttpClientEnablePush()` | `jdk.httpclient.enablepush` |
| `getJdkHttpClientHpackMaxHeaderTableSize()` | `jdk.httpclient.hpack.maxheadertablesize` |
| `getJdkHttpClientHttpClientLog()` | `jdk.httpclient.HttpClient.log` |
| `getJdkHttpClientKeepAliveTimeout()` | `jdk.httpclient.keepalive.timeout` |
| `getJdkHttpClientKeepAliveTimeoutH2()` | `jdk.httpclient.keepalive.timeout.h2` |
| `getJdkHttpClientMaxFrameSize()` | `jdk.httpclient.maxframesize` |
| `getJdkHttpClientMaxStreams()` | `jdk.httpclient.maxstreams` |
| `getJdkHttpClientReceiveBufferSize()` | `jdk.httpclient.receiveBufferSize` |
| `getJdkHttpClientRedirectsRetryLimit()` | `jdk.httpclient.redirects.retrylimit` |
| `getJdkHttpClientSendBufferSize()` | `jdk.httpclient.sendBufferSize` |
| `getJdkHttpClientWebSocketWriteBufferSize()` | `jdk.httpclient.websocket.writeBufferSize` |
| `getJdkHttpClientWindowSize()` | `jdk.httpclient.windowsize` |
| `getJdkHttpServerMaxConnections()` | `jdk.httpserver.maxConnections` |
| `getJdkHttpsNegotiateCbt()` | `jdk.https.negotiate.cbt` |
| `getJdkIncludeInExceptions()` | `jdk.includeInExceptions` |
| `getJdkInternalHttpClientDisableHostNameVerification()` | `jdk.internal.httpclient.disableHostnameVerification` |
| `getJdkIoPermissionsUseCanonicalPath()` | `jdk.io.permissionsUseCanonicalPath` |
| `getJdkJndiLdapObjectFactoriesFilter()` | `jdk.jndi.ldap.object.factoriesFilter` |
| `getJdkJndiObjectFactoriesFilter()` | `jdk.jndi.object.factoriesFilter` |
| `getJdkJndiRmiObjectFactoriesFilter()` | `jdk.jndi.rmi.object.factoriesFilter` |
| `getJdkModuleMain()` | `jdk.module.main` |
| `getJdkModuleMainClass()` | `jdk.module.main.class` |
| `getJdkModulePath()` | `jdk.module.path` |
| `getJdkModuleUpgradePath()` | `jdk.module.upgrade.path` |
| `getJdkNetUnixDomainTmpDir()` | `jdk.net.unixdomain.tmpdir` |
| `getJdkNetUrlClassPathShowIgnoredClassPathEntries()` | `jdk.net.URLClassPath.showIgnoredClassPathEntries` |
| `getJdkSerialFilter()` | `jdk.serialFilter` |
| `getJdkSerialFilterFactory()` | `jdk.serialFilterFactory` |
| `getJdkTlsClientSignatureSchemes()` | `jdk.tls.client.SignatureSchemes` |
| `getJdkTlsNamedGroups()` | `jdk.tls.namedGroups` |
| `getJdkTlsServerSignatureSchemes()` | `jdk.tls.server.SignatureSchemes` |
| `getJdkVirtualThreadSchedulerMaxPoolSize()` | `jdk.virtualThreadScheduler.maxPoolSize` |
| `getJdkVirtualThreadSchedulerParallelism()` | `jdk.virtualThreadScheduler.parallelism` |
| `getJdkXmlCdataChunkSize()` | `jdk.xml.cdataChunkSize` |
| `getJdkXmlDtdSupport()` | `jdk.xml.dtd.support` |
| `getJdkXmlElementAttributeLimit()` | `jdk.xml.elementAttributeLimit` |
| `getJdkXmlEnableExtensionFunctions()` | `jdk.xml.enableExtensionFunctions` |
| `getJdkXmlEntityExpansionLimit()` | `jdk.xml.entityExpansionLimit` |
| `getJdkXmlEntityReplacementLimit()` | `jdk.xml.entityReplacementLimit` |
| `getJdkXmlIsStandalone()` | `jdk.xml.isStandalone` |
| `getJdkXmlJdkCatalogResolve()` | `jdk.xml.jdkcatalog.resolve` |
| `getJdkXmlMaxElementDepth()` | `jdk.xml.maxElementDepth` |
| `getJdkXmlMaxGeneralEntitySizeLimit()` | `jdk.xml.maxGeneralEntitySizeLimit` |
| `getJdkXmlMaxOccurLimit()` | `jdk.xml.maxOccurLimit` |
| `getJdkXmlMaxParameterEntitySizeLimit()` | `jdk.xml.maxParameterEntitySizeLimit` |
| `getJdkXmlMaxXmlNameLimit()` | `jdk.xml.maxXMLNameLimit` |
| `getJdkXmlOverrideDefaultParser()` | `jdk.xml.overrideDefaultParser` |
| `getJdkXmlResetSymbolTable()` | `jdk.xml.resetSymbolTable` |
| `getJdkXmlTotalEntitySizeLimit()` | `jdk.xml.totalEntitySizeLimit` |
| `getJdkXmlXsltcIsStandalone()` | `jdk.xml.xsltcIsStandalone` |
| `getLineSeparator()` | `line.separator` |
| `getLineSeparator(Supplier<String>)` | `line.separator` (con proveedor por defecto) |
| `getNativeEncoding()` | `native.encoding` |
| `getNetworkAddressCacheNegativeTtl()` | `networkaddress.cache.negative.ttl` |
| `getNetworkAddressCacheStaleTtl()` | `networkaddress.cache.stale.ttl` |
| `getNetworkAddressCacheTtl()` | `networkaddress.cache.ttl` |
| `getOrgJcpXmlDsigSecureValidation()` | `org.jcp.xml.dsig.securevalidation` |
| `getOrgOpenJdkJavaUtilStreamTripwire()` | `org.openjdk.java.util.stream.tripwire` |
| `getOsArch()` | `os.arch` |
| `getOsName()` | `os.name` |
| `getOsVersion()` | `os.version` |
| `getPathSeparator()` | `path.separator` |
| `getSocksProxyHost()` | `socksProxyHost` |
| `getSocksProxyPort()` | `socksProxyPort` |
| `getSocksProxyVersion()` | `socksProxyVersion` |
| `getStdErrEncoding()` | `stderr.encoding` |
| `getStdOutEncoding()` | `stdout.encoding` |
| `getSunNetHttpServerDrainAmount()` | `sun.net.httpserver.drainAmount` |
| `getSunNetHttpServerIdleInterval()` | `sun.net.httpserver.idleInterval` |
| `getSunNetHttpServerMaxIdleConnections()` | `sun.net.httpserver.maxIdleConnections` |
| `getSunNetHttpServerMaxReqHeaders()` | `sun.net.httpserver.maxReqHeaders` |
| `getSunNetHttpServerMaxReqTime()` | `sun.net.httpserver.maxReqTime` |
| `getSunNetHttpServerMaxRspTime()` | `sun.net.httpserver.maxRspTime` |
| `getSunNetHttpServerNoDelay()` | `sun.net.httpserver.nodelay` |
| `getSunSecurityKrb5Principal()` | `sun.security.krb5.principal` |
| `getUserCountry()` | `user.country` |
| `getUserDir()` | `user.dir` |
| `getUserExtensions()` | `user.extensions` |
| `getUserHome()` | `user.home` |
| `getUserLanguage()` | `user.language` |
| `getUserName()` | `user.name` |
| `getUserName(String)` | `user.name` (con valor por defecto) |
| `getUserRegion()` | `user.region` |
| `getUserScript()` | `user.script` |
| `getUserTimezone()` | `user.timezone` |
| `getUserVariant()` | `user.variant` |

---

### 3. Ejemplo de Uso

El siguiente bloque de código muestra cómo utilizar la clase `SystemProperties` para acceder a diferentes propiedades del sistema de forma segura y conveniente.

```java
import org.apache.commons.lang3.SystemProperties;
import java.nio.file.Path;
import java.nio.file.Paths;

public class SystemPropertiesExample {

    public static void main(String[] args) {
        // 1. Obtener una propiedad estándar usando un método de acceso directo
        String javaVersion = SystemProperties.getJavaVersion();
        System.out.println("Versión de Java: " + javaVersion);

        // 2. Obtener el directorio home del usuario
        String userHome = SystemProperties.getUserHome();
        System.out.println("Directorio Home del Usuario: " + userHome);

        // 3. Intentar obtener una propiedad que no existe (devuelve null de forma segura)
        String nonExistentProp = SystemProperties.getProperty("my.app.non.existent.property");
        System.out.println("Propiedad inexistente: " + nonExistentProp);

        // 4. Usar un método genérico con un valor por defecto
        String osName = SystemProperties.getProperty(SystemProperties.OS_NAME, "Desconocido");
        System.out.println("Nombre del SO: " + osName);

        // 5. Obtener un valor numérico con un valor por defecto usando un Supplier
        // Para probar, se puede ejecutar con: -Dmyapp.pool.size=50
        int poolSize = SystemProperties.getInt("myapp.pool.size", () -> 30); // 30 es el valor por defecto
        System.out.println("Tamaño del Pool de Conexiones: " + poolSize);

        // 6. Obtener una propiedad booleana
        // Para probar, se puede ejecutar con: -Dmyapp.feature.enabled=true
        boolean isFeatureEnabled = SystemProperties.getBoolean("myapp.feature.enabled", () -> false);
        System.out.println("¿Funcionalidad activada?: " + isFeatureEnabled);
        
        // 7. Obtener una propiedad de tipo Path
        Path tempDir = SystemProperties.getPath(SystemProperties.JAVA_IO_TMPDIR, () -> Paths.get("/tmp"));
        System.out.println("Directorio temporal: " + tempDir.toAbsolutePath());

        // 8. Verificar si una propiedad está definida
        boolean isUserDirSet = SystemProperties.isPropertySet(SystemProperties.USER_DIR);
        System.out.println("¿La propiedad 'user.dir' está definida?: " + isUserDirSet);
    }
}
```