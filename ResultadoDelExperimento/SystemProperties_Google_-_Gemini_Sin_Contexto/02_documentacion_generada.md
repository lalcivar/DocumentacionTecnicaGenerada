Claro, aquí tienes la documentación técnica para la clase `SystemProperties` de Apache Commons Lang.

---

### Documentación Técnica: Clase `SystemProperties` de Apache Commons Lang

### Descripción General

La clase `org.apache.commons.lang3.SystemProperties` es una clase de utilidad (`utility class`) final que proporciona un acceso seguro y conveniente a las [propiedades del sistema de Java](https://docs.oracle.com/en/java/javase/25/docs/api/system-properties.html) (`System Properties`). Su objetivo es simplificar y robustecer el trabajo con estas propiedades en comparación con el uso directo de `System.getProperty()`.

Las principales ventajas de usar esta clase son:

1.  **Constantes Predefinidas:** Proporciona un conjunto exhaustivo de constantes `public static final String` para las claves de propiedades del sistema estándar (ej. `JAVA_HOME`, `OS_NAME`, `USER_DIR`), lo que ayuda a evitar errores tipográficos y mejora la legibilidad del código.
2.  **Acceso Seguro:** Ofrece métodos de acceso (getters) que manejan de forma segura las `SecurityException`. En lugar de lanzar una excepción si el acceso a una propiedad está restringido por el `SecurityManager`, estos métodos devuelven `null` o un valor predeterminado, haciendo el código más robusto.
3.  **Getters Tipados:** Proporciona métodos para obtener propiedades ya convertidas a tipos comunes como `boolean`, `int`, `long` y `Path`, simplificando el código al evitar la necesidad de conversiones manuales y manejo de valores por defecto.
4.  **API Fluida y Legible:** Ofrece métodos específicos para cada constante (ej. `getJavaHome()`), lo que resulta en un código más limpio y auto-documentado.

Al ser una clase `final` con métodos exclusivamente estáticos, no está diseñada para ser instanciada.

### Métodos Principales

La funcionalidad de la clase se centra en sus métodos estáticos para obtener propiedades. Se pueden agrupar en las siguientes categorías:

#### 1. Getters Genéricos (`getProperty`)

Estos son los métodos base para obtener cualquier propiedad del sistema como una cadena de texto (`String`).

*   `public static String getProperty(final String property)`
    *   **Descripción:** Obtiene el valor de una propiedad del sistema a partir de su clave. Es la forma más segura de llamar a `System.getProperty()`.
    *   **Comportamiento:** Devuelve el valor de la propiedad como un `String`. Si la propiedad no existe o si se produce una `SecurityException` al intentar acceder a ella, devuelve `null`.

*   `static String getProperty(final String property, final Supplier<String> defaultIfAbsent)`
    *   **Descripción:** Similar al anterior, pero permite especificar un valor por defecto a través de un `Supplier`.
    *   **Comportamiento:** Si la propiedad no está definida o no se puede acceder a ella, se invoca al `Supplier` y se devuelve su resultado. El uso de un `Supplier` permite que el valor por defecto se genere de forma "perezosa" (solo cuando es necesario).

#### 2. Getters Tipados (`getBoolean`, `getInt`, `getLong`, `getPath`)

Estos métodos obtienen el valor de una propiedad y lo convierten al tipo de dato correspondiente, proporcionando una forma de definir un valor por defecto si la propiedad no existe.

*   `public static boolean getBoolean(final String key, final BooleanSupplier defaultIfAbsent)`
    *   **Descripción:** Obtiene una propiedad y la convierte a un `boolean` usando las reglas de `Boolean.parseBoolean()`.
    *   **Comportamiento:** Si la propiedad no existe, devuelve el valor proporcionado por el `BooleanSupplier`.

*   `public static int getInt(final String key, final IntSupplier defaultIfAbsent)`
    *   **Descripción:** Obtiene una propiedad y la convierte a un `int` usando `Integer.parseInt()`.
    *   **Comportamiento:** Si la propiedad no existe, devuelve el valor del `IntSupplier`. Si la propiedad existe pero no es un entero válido, se lanzará una `NumberFormatException`.

*   `public static long getLong(final String key, final LongSupplier defaultIfAbsent)`
    *   **Descripción:** Obtiene una propiedad y la convierte a un `long` usando `Long.parseLong()`.
    *   **Comportamiento:** Similar al método `getInt`, pero para el tipo `long`.

*   `public static Path getPath(final String key, final Supplier<Path> defaultIfAbsent)`
    *   **Descripción:** Obtiene una propiedad que representa una ruta de archivo y la convierte a un objeto `java.nio.file.Path`.
    *   **Comportamiento:** Si la propiedad no existe, devuelve el valor del `Supplier<Path>`.

#### 3. Getters Específicos (ej. `getJavaHome()`, `getOsName()`)

La clase proporciona un método getter específico para la mayoría de las constantes que define. Estos métodos son atajos convenientes que mejoran la legibilidad y la seguridad del código.

*   `public static String getJavaHome()`
    *   **Descripción:** Devuelve el valor de la propiedad del sistema `java.home`. Es un atajo para `getProperty(JAVA_HOME)`.

*   `public static String getOsName()`
    *   **Descripción:** Devuelve el valor de la propiedad del sistema `os.name`. Es un atajo para `getProperty(OS_NAME)`.

*   `public static String getUserDir()`
    *   **Descripción:** Devuelve el valor de la propiedad del sistema `user.dir` (directorio de trabajo actual). Es un atajo para `getProperty(USER_DIR)`.

*   `public static String getJavaVersion()`
    *   **Descripción:** Devuelve el valor de la propiedad del sistema `java.version`. Es un atajo para `getProperty(JAVA_VERSION)`.

Existen docenas de métodos similares para casi todas las propiedades estándar, como `getJavaClassPath()`, `getFileSeparator()`, `getUserName()`, etc.

#### 4. Método de Verificación (`isPropertySet`)

*   `public static boolean isPropertySet(final String property)`
    *   **Descripción:** Un método de utilidad simple para comprobar si una propiedad del sistema está definida.
    *   **Comportamiento:** Devuelve `true` si el valor de la propiedad no es `null`, y `false` en caso contrario (incluyendo si el acceso está denegado por seguridad). Es un atajo para `getProperty(property) != null`.