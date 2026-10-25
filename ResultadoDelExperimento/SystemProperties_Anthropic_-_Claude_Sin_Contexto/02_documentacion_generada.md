# Documentación técnica: `org.apache.commons.lang3.SystemProperties`

## 1. Descripción general

`SystemProperties` es una clase **utilitaria final** (`public final class`) perteneciente a **Apache Commons Lang 3** (disponible desde la versión **3.13.0**) cuyo propósito es ofrecer un **acceso centralizado, tipado y seguro a las propiedades del sistema de la JVM** (las que normalmente se consultan mediante `System.getProperty(...)`).

La clase cumple dos funciones complementarias:

1. **Catálogo de nombres de propiedades**: expone más de 200 constantes `public static final String` con los nombres canónicos de las propiedades documentadas por Oracle (p. ej. `JAVA_VERSION = "java.version"`, `OS_NAME = "os.name"`, `JDK_XML_MAX_ELEMENT_DEPTH = "jdk.xml.maxElementDepth"`). Esto evita errores por *strings* literales mal escritos ("magic strings").
2. **Accesores tolerantes a fallos**: proporciona un método *getter* dedicado por cada propiedad conocida (`getJavaVersion()`, `getOsName()`, `getUserHome()`, …) más un conjunto de métodos genéricos de conversión de tipos (`getBoolean`, `getInt`, `getLong`, `getPath`, `getProperty`).

### Característica clave: ausencia de excepciones

Todos los accesores delegan en el método interno `getProperty(String, Supplier<String>)`, que **captura e ignora `SecurityException`**. Si la propiedad no existe, está vacía o el `SecurityManager` impide su lectura, el resultado es `null` (o el valor por defecto suministrado), nunca una excepción. Esto hace que la clase sea segura de usar en entornos restringidos (applets históricos, contenedores, *sandboxes*).

### Convenciones de diseño

| Aspecto | Detalle |
|---|---|
| Instanciación | No debe instanciarse. El constructor público está **`@Deprecated`** y se documenta que pasará a `private` en la versión 4.0. |
| Estado | Clase sin estado (*stateless*); todos los métodos de consulta son `static`. |
| Valores "en vivo" | No hay caché: cada llamada lee el mapa de propiedades actual, por lo que refleja cambios hechos con `System.setProperty(...)`. |
| Elementos obsoletos | Las propiedades eliminadas a partir de Java 17 (`awt.toolkit`, `java.awt.fonts`, `java.awt.headless`, `java.compiler`, `java.endorsed.dirs`, `java.ext.dirs`, …) y sus *getters* están marcados como `@Deprecated`. |
| Dependencias internas | `StringUtils` (comprobación de cadenas vacías), `ClassUtils` (construcción de claves), `org.apache.commons.lang3.function.Suppliers` (evaluación segura de `Supplier`). |

---

## 2. Agrupación de las constantes

Las constantes se organizan por el área funcional de la JVM a la que pertenecen:

| Grupo | Ejemplos de constantes |
|---|---|
| **Entorno Java / JVM** | `JAVA_VERSION`, `JAVA_VENDOR`, `JAVA_HOME`, `JAVA_CLASS_PATH`, `JAVA_VM_NAME`, `JAVA_SPECIFICATION_VERSION`, `JAVA_RUNTIME_NAME` |
| **Sistema operativo y rutas** | `OS_NAME`, `OS_ARCH`, `OS_VERSION`, `FILE_SEPARATOR`, `PATH_SEPARATOR`, `LINE_SEPARATOR`, `JAVA_IO_TMPDIR` |
| **Usuario y localización** | `USER_NAME`, `USER_HOME`, `USER_DIR`, `USER_LANGUAGE`, `USER_COUNTRY`, `USER_TIMEZONE`, `USER_VARIANT` |
| **Codificaciones** | `FILE_ENCODING`, `NATIVE_ENCODING`, `STDOUT_ENCODING`, `STDERR_ENCODING` |
| **Red / proxies** | `HTTP_PROXY_HOST`, `HTTP_PROXY_PORT`, `HTTPS_PROXY_HOST`, `FTP_PROXY_HOST`, `SOCKS_PROXY_HOST`, `HTTP_NON_PROXY_HOSTS`, `JAVA_NET_PREFER_IPV4_STACK` |
| **Cliente/servidor HTTP del JDK** | `JDK_HTTP_CLIENT_*` (más de 20 constantes), `JDK_HTTP_SERVER_MAX_CONNECTIONS`, `SUN_NET_HTTP_SERVER_*` |
| **Seguridad / criptografía** | `JAVA_SECURITY_MANAGER`, `JAVA_SECURITY_DEBUG`, `JAVA_SECURITY_KERBEROS_CONF/KDC/REALM`, `JDK_TLS_NAMED_GROUPS`, `JDK_SERIAL_FILTER` |
| **XML (JAXP)** | `JDK_XML_ENTITY_EXPANSION_LIMIT`, `JDK_XML_MAX_ELEMENT_DEPTH`, `JDK_XML_ENABLE_EXTENSION_FUNCTIONS`, `JAVA_XML_CONFIG_FILE` |
| **Concurrencia / NIO** | `JAVA_UTIL_CONCURRENT_FORK_JOIN_POOL_COMMON_PARALLELISM`, `JDK_VIRTUAL_THREAD_SCHEDULER_PARALLELISM`, `JAVA_NIO_CHANNELS_SPI_SELECTOR_PROVIDER` |
| **Módulos (JPMS)** | `JDK_MODULE_MAIN`, `JDK_MODULE_MAIN_CLASS`, `JDK_MODULE_PATH`, `JDK_MODULE_UPGRADE_PATH` |

---

## 3. Métodos principales

### 3.1 Acceso genérico

#### `public static String getProperty(String property)`
Devuelve el valor de la propiedad indicada.
- **Parámetros**: `property` — nombre de la propiedad del sistema.
- **Retorno**: el valor, o `null` si la propiedad es inexistente, su valor está vacío, el nombre es vacío/nulo, o se produce una `SecurityException`.
- Es el método base del que dependen todos los *getters* específicos.

#### `static String getProperty(String property, String defaultIfAbsent)` *(visibilidad de paquete)*
Variante con valor por defecto literal.

#### `static String getProperty(String property, Supplier<String> defaultIfAbsent)` *(visibilidad de paquete)*
Núcleo de la implementación: valida que el nombre no esté vacío, consulta `System.getProperty`, sustituye valores vacíos por el resultado del `Supplier` y captura `SecurityException`. La evaluación diferida del `Supplier` evita calcular el valor por defecto cuando no es necesario.

#### `public static boolean isPropertySet(String property)` *(desde 3.18.0)*
Indica si la propiedad está definida. Equivale a `getProperty(property) != null`; devuelve `false` ante `SecurityException`.

---

### 3.2 Accesores tipados

Permiten obtener el valor convertido a un tipo primitivo o a `Path`, aplicando un valor por defecto perezoso cuando la propiedad no existe.

| Método | Conversión | Comportamiento si está ausente |
|---|---|---|
| `getBoolean(String key, BooleanSupplier defaultIfAbsent)` | `Boolean.parseBoolean` | `defaultIfAbsent.getAsBoolean()`; `false` si el supplier es `null` |
| `getInt(String key, IntSupplier defaultIfAbsent)` | `Integer.parseInt` | `defaultIfAbsent.getAsInt()`; `0` si el supplier es `null` |
| `getLong(String key, LongSupplier defaultIfAbsent)` | `Long.parseLong` | `defaultIfAbsent.getAsLong()`; `0` si el supplier es `null` |
| `getPath(String key, Supplier<Path> defaultIfAbsent)` *(3.20.0)* | `Paths.get` | `Suppliers.get(defaultIfAbsent)` (seguro ante `null`) |

> **Nota**: si la propiedad existe pero su contenido no es numérico, `getInt`/`getLong` propagan `NumberFormatException`; la tolerancia a fallos sólo cubre la ausencia del valor y las restricciones de seguridad.

---

### 3.3 Accesores tipados con clave derivada de una clase *(desde 3.19.0)*

```java
public static boolean getBoolean(Class<?> clazz, String key, BooleanSupplier defaultIfAbsent)
public static int     getInt    (Class<?> clazz, String key, IntSupplier     defaultIfAbsent)
public static long    getLong   (Class<?> clazz, String key, LongSupplier    defaultIfAbsent)
```

Construyen la clave efectiva con el patrón **`"NombreSimpleDeClase.key"`** mediante el método auxiliar privado:

```java
private static String toKey(Class<?> clazz, String key, boolean simpleKey)
```

que delega en `ClassUtils.getName(clazz, "", true)`. Por ejemplo, `getInt(MiCache.class, "size", () -> 128)` consulta la propiedad `MiCache.size`. Resulta útil para definir propiedades de configuración con espacio de nombres propio de cada componente.

---

### 3.4 Accesores específicos por propiedad

Constituyen la mayor parte de la API pública. Siguen un patrón uniforme de una sola línea:

```java
public static String getJavaVersion() {
    return getProperty(JAVA_VERSION);
}
```

Ejemplos representativos:

| Método | Propiedad consultada |
|---|---|
| `getJavaHome()`, `getJavaVersion()`, `getJavaVendor()`, `getJavaClassPath()` | entorno Java |
| `getOsName()`, `getOsArch()`, `getOsVersion()` | sistema operativo |
| `getFileSeparator()`, `getPathSeparator()`, `getLineSeparator()` | separadores |
| `getUserName()`, `getUserHome()`, `getUserDir()`, `getUserLanguage()` | usuario |
| `getFileEncoding()`, `getNativeEncoding()`, `getStdOutEncoding()`, `getStdErrEncoding()` | codificaciones |
| `getHttpProxyHost()`, `getHttpsProxyPort()`, `getSocksProxyHost()` | red |
| `getJdkXmlMaxElementDepth()`, `getJdkXmlEntityExpansionLimit()` | XML |
| `getJdkVirtualThreadSchedulerParallelism()` | hilos virtuales |

**Sobrecargas con valor por defecto** (añadidas en 3.15.0):

- `getJavaSpecificationVersion(String defaultValue)`
- `getUserName(String defaultValue)`
- `getLineSeparator(Supplier<String> defaultIfAbsent)`

**Accesores obsoletos** (`@Deprecated`, sin reemplazo): `getAwtToolkit()`, `getJavaAwtFonts()`, `getJavaAwtGraphicsenv()`, `getJavaAwtHeadless()`, `getJavaAwtPrinterjob()`, `getJavaCompiler()`, `getJavaEndorsedDirs()`, `getJavaExtDirs()`.

---

## 4. Ejemplos de uso

```java
// 1. Lectura directa mediante getter específico
String version = SystemProperties.getJavaVersion();      // p. ej. "21.0.3"
String tmp     = SystemProperties.getJavaIoTmpdir();

// 2. Valor por defecto cuando la propiedad no está disponible
String user = SystemProperties.getUserName("desconocido");
String eol  = SystemProperties.getLineSeparator(() -> "\n");

// 3. Comprobación de existencia
if (SystemProperties.isPropertySet(SystemProperties.JAVA_SECURITY_MANAGER)) {
    // ...
}

// 4. Conversión tipada con valor por defecto perezoso
int  poolSize = SystemProperties.getInt("app.pool.size", () -> 10);
long timeout  = SystemProperties.getLong("app.timeout.ms", () -> 30_000L);
boolean debug = SystemProperties.getBoolean("app.debug", () -> false);
Path  config  = SystemProperties.getPath("app.config", () -> Paths.get("conf/app.properties"));

// 5. Clave con espacio de nombres de clase: lee "MiServicio.retries"
int retries = SystemProperties.getInt(MiServicio.class, "retries", () -> 3);

// 6. Uso de las constantes para evitar literales
String path = SystemProperties.getProperty(SystemProperties.JAVA_CLASS_PATH);
```

---

## 5. Consideraciones y buenas prácticas

- **No instanciar**: utilizar siempre los métodos estáticos. El constructor existe sólo por compatibilidad binaria y desaparecerá como `public` en 4.0.
- **Semántica de "vacío"**: una propiedad definida con cadena vacía se trata como ausente (se aplica el valor por defecto), por el uso de `StringUtils.getIfEmpty`.
- **Thread-safety**: la clase es inherentemente *thread-safe* al no mantener estado mutable; la seguridad última depende de `System.getProperties()`.
- **Portabilidad entre versiones de Java**: muchas constantes corresponden a propiedades introducidas en versiones recientes (Java 9, 11, 17, 19+). Consultarlas en una JVM anterior simplemente devuelve `null`.
- **Relación con `SystemUtils`**: `SystemUtils` expone constantes evaluadas una única vez al cargar la clase; `SystemProperties` lee el valor en cada invocación, por lo que es la opción adecuada cuando las propiedades pueden cambiar en tiempo de ejecución.