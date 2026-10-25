# Documentación Técnica - Clase `SystemProperties`

**Paquete:** `org.apache.commons.lang3`  
**Librería:** Apache Commons Lang 3  
**Versión:** Desde `3.13.0`  
**Licencia:** Apache License 2.0  
**Tipo:** Clase de utilidad `public final class SystemProperties`

### 1. Descripción General

`SystemProperties` es una clase de utilidad **final y no instanciable** que centraliza el acceso seguro y tipado a las [System Properties de Java](https://docs.oracle.com/en/java/javase/25/docs/api/system-properties.html) `System.getProperty()`.

Su objetivo es resolver tres problemas del acceso directo a `System.getProperty()`:

1.  **Evitar errores de tipeo:** Provee más de 200 constantes `public static final String` con los nombres oficiales de las propiedades del sistema.
2.  **Gestión segura de excepciones:** Todos los accesos están encapsulados en un bloque `try-catch` para `SecurityException`. Si el `SecurityManager` deniega el acceso, el método retorna `null` o el valor por defecto en lugar de lanzar una excepción.
3.  **Acceso tipado y conveniente:** Ofrece métodos de alto nivel para obtener valores ya convertidos a `String`, `boolean`, `int`, `long` y `Path`.

La clase es *thread-safe*, sin estado y compuesta únicamente por métodos y constantes estáticas.

> **Nota de diseño:** El constructor `public SystemProperties()` está marcado como `@Deprecated` y será privado en la versión 4.0. La clase no debe ser instanciada.

### 2. Constantes Principales

La clase define constantes para cada propiedad del sistema documentada por Oracle. Se agrupan por categorías:

| Categoría | Prefijo | Ejemplos de Constantes |
| :--- | :--- | :--- |
| **Java Runtime / VM** | `JAVA_*` | `JAVA_VERSION`, `JAVA_HOME`, `JAVA_VENDOR`, `JAVA_VM_NAME`, `JAVA_CLASS_PATH`, `JAVA_IO_TMPDIR` |
| **Sistema Operativo** | `OS_*` | `OS_NAME`, `OS_ARCH`, `OS_VERSION` |
| **Usuario** | `USER_*` | `USER_NAME`, `USER_HOME`, `USER_DIR`, `USER_LANGUAGE`, `USER_TIMEZONE` |
| **Sistema de Archivos** | `FILE_*`, `PATH_*`, `LINE_*` | `FILE_SEPARATOR`, `PATH_SEPARATOR`, `LINE_SEPARATOR` |
| **Red / Proxy / HTTP** | `HTTP_*`, `HTTPS_*`, `FTP_*` | `HTTP_PROXY_HOST`, `HTTP_PROXY_PORT`, `HTTPS_PROXY_HOST`, `JAVA_NET_USE_SYSTEM_PROXIES` |
| **JDK / JNDI / TLS / XML** | `JDK_*` | `JDK_SERIAL_FILTER`, `JDK_TLS_NAMED_GROUPS`, `JDK_XML_ENTITY_EXPANSION_LIMIT`, `JDK_HTTP_CLIENT_BUF_SIZE` |
| **Obsoletas** | `@Deprecated` | `AWT_TOOLKIT`, `JAVA_AWT_HEADLESS`, `JAVA_ENDORSED_DIRS`, `JAVA_EXT_DIRS` |

> Todas las constantes contienen el valor literal de la propiedad, ej: `public static final String JAVA_VERSION = "java.version";`

### 3. Métodos Principales

Se pueden clasificar en 3 grupos:

#### A. Métodos Base de Acceso Genérico
Son el núcleo sobre el que se construyen todos los demás. Gestionan la lógica de `SecurityException` y valores por defecto.

| Método | Descripción |
| :--- | :--- |
| `static String getProperty(String property)` | Obtiene una propiedad. Retorna `null` si no existe o si hay `SecurityException`. |
| `static String getProperty(String property, String defaultIfAbsent)` | Retorna `defaultIfAbsent` si la propiedad es nula, vacía o hay `SecurityException`. |
| `static String getProperty(String property, Supplier<String> defaultIfAbsent)` | Versión perezosa con `Supplier`. Usa `StringUtils.getIfEmpty()` y `Suppliers`. |
| `static boolean isPropertySet(String property)` | Verifica si una propiedad está definida. Equivalente a `getProperty(property) != null`. Desde `3.18.0`. |

#### B. Getters Específicos Tipados a `String`
Más de 150 métodos con la forma `get<NombrePropiedad>()` que encapsulan `getProperty(CONSTANTE)`. Retornan `String` o `null`.

**Ejemplos:**
* `getJavaVersion()`, `getJavaHome()`, `getJavaVendor()`
* `getOsName()`, `getOsArch()`, `getOsVersion()`
* `getUserName()`, `getUserHome()`, `getUserDir()`
* `getFileEncoding()`, `getFileSeparator()`, `getLineSeparator()`
* `getHttpProxyHost()`, `getHttpsProxyPort()`
* `getJdkSerialFilter()`, `getJdkTlsNamedGroups()`

Sobrecargas con valor por defecto:
* `getJavaSpecificationVersion(String defaultValue)`
* `getUserName(String defaultValue)`
* `getLineSeparator(Supplier<String> defaultIfAbsent)`

#### C. Métodos de Conversión Tipada
Permiten obtener propiedades personalizadas ya convertidas al tipo requerido, con soporte para claves basadas en clases.

| Método | Descripción |
| :--- | :--- |
| `getBoolean(String key, BooleanSupplier defaultIfAbsent)` | Parsea con `Boolean.parseBoolean()`. Retorna el valor del `BooleanSupplier` si la propiedad es `null`. |
| `getBoolean(Class<?> clazz, String key, BooleanSupplier defaultIfAbsent)` | Construye la clave como `SimpleClassName.key` y delega al anterior. Desde `3.19.0`. |
| `getInt(String key, IntSupplier defaultIfAbsent)` | Parsea con `Integer.parseInt()`. Retorna `0` si `defaultIfAbsent` es `null`. |
| `getInt(Class<?> clazz, String key, IntSupplier defaultIfAbsent)` | Variante con clave `SimpleClassName.key`. |
| `getLong(String key, LongSupplier defaultIfAbsent)` | Parsea con `Long.parseLong()`. |
| `getLong(Class<?> clazz, String key, LongSupplier defaultIfAbsent)` | Variante con clave `SimpleClassName.key`. |
| `getPath(String key, Supplier<Path> defaultIfAbsent)` | Convierte el valor a `java.nio.file.Path` vía `Paths.get()`. Desde `3.20.0`. |

Método auxiliar privado:
* `toKey(Class<?> clazz, String key, boolean simpleKey)`: Construye claves del tipo `MiClase.miPropiedad` usando `ClassUtils.getName()`.

### 4. Manejo de Errores y Comportamiento

* Si `property` es `null` o vacío, se retorna directamente el valor por defecto sin consultar al sistema.
* Si `System.getProperty()` lanza `SecurityException`, se captura silenciosamente y se retorna el valor por defecto.
* Los métodos `getInt` / `getLong` lanzarán `NumberFormatException` si el valor existe pero no es parseable. No capturan esta excepción.

### 5. Ejemplo de Uso

```java
// Acceso seguro con constante
String javaVersion = SystemProperties.getJavaVersion();
String osName = SystemProperties.getOsName();

// Acceso con valor por defecto
String user = SystemProperties.getUserName("desconocido");

// Acceso tipado a propiedad custom
boolean debug = SystemProperties.getBoolean("app.debug", () -> false);
int poolSize = SystemProperties.getInt(MyService.class, "pool.size", () -> 10);
Path configPath = SystemProperties.getPath("app.config.path", () -> Paths.get("/tmp/default.conf"));

// Verificación
if (SystemProperties.isPropertySet(SystemProperties.JAVA_VENDOR_URL)) {
    // ...
}
```

