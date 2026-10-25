# Documentación Técnica - Clase `SystemUtilsTest`

### 1. Información General

| Atributo | Valor |
| :--- | :--- |
| **Nombre de la Clase** | `SystemUtilsTest` |
| **Paquete** | `org.apache.commons.lang3` |
| **Clase Probada** | `org.apache.commons.lang3.SystemUtils` |
| **Herencia** | `extends AbstractLangTest` |
| **Framework de Pruebas** | JUnit 5 `Jupiter` |
| **Licencia** | Apache License, Version 2.0 |
| **Tipo de Clase** | Clase de Pruebas Unitarias |
| **Visibilidad** | package-private `class` |

> **Nota Importante:** El código proporcionado no corresponde a la clase de utilidad `SystemUtils` en sí, sino a su **clase de pruebas unitarias `SystemUtilsTest`**. Su objetivo es validar el correcto funcionamiento de `SystemUtils`, que es una clase de utilidad de `Apache Commons Lang3` para interactuar con el sistema operativo y el entorno de ejecución de Java.

### 2. Descripción General

`SystemUtilsTest` es una suite de pruebas unitarias que verifica el comportamiento de la clase `SystemUtils`. 

La clase `SystemUtils` es una clase de utilidad `helper` que abstrae el acceso a propiedades del sistema `System.getProperty()`, variables de entorno `System.getenv()` y proporciona constantes y métodos estáticos para:

1.  **Detección del Sistema Operativo:** Identificar si la aplicación se ejecuta en Windows, Linux, macOS, Solaris, Unix, etc.
2.  **Detección de la Versión de Java:** Identificar la versión de la JVM en ejecución mediante constantes `IS_JAVA_*` y métodos de comparación.
3.  **Acceso a directorios del sistema:** Obtener de forma segura rutas como `java.home`, `user.home`, `user.dir` y `java.io.tmpdir` tanto como `File` como `Path`.
4.  **Gestión de entorno:** Obtener variables de entorno y nombre del host.

`SystemUtilsTest` garantiza que esta detección sea correcta y multiplataforma, asumiendo que no existe un `SecurityManager` que bloquee el acceso a las propiedades del sistema.

### 3. Dependencias

*   `org.apache.commons.lang3.SystemUtils` - Clase bajo prueba.
*   `org.apache.commons.lang3.JavaVersion` - Enum para comparación de versiones de Java.
*   `org.apache.commons.lang3.BooleanUtils` - Para operaciones lógicas sobre arrays de booleanos.
*   `org.junit.jupiter.api` - Anotaciones y aserciones de JUnit 5.
*   `java.lang.reflect` - Para acceso reflexivo a campos y constructores.
*   `java.nio.file` - Para validación de `Path`.

### 4. Métodos Auxiliares Privados

Métodos de soporte interno para la lógica de las pruebas, no son casos de prueba en sí.

| Método | Descripción |
| :--- | :--- |
| `getIS_JAVA(int version)` | Obtiene mediante reflexión el valor del campo estático `SystemUtils.IS_JAVA_X` para versiones >= 9. Permite testear dinámicamente sin hardcodear cada constante. |
| `getLastSupportedJavaVersion()` | Itera todos los campos de `SystemUtils` que coinciden con el patrón `IS_JAVA_\d+` y retorna el número de versión más alto soportado. Hace que las pruebas sean extensibles a futuras versiones de Java. |

### 5. Métodos Principales - Casos de Prueba

#### 5.1 Validación de Constantes de Versión y Sistema Operativo

| Método | Descripción |
| :--- | :--- |
| `test_IS_JAVA()` | Valida que solo una de las constantes `IS_JAVA_1_8`, `IS_JAVA_9` ... `IS_JAVA_26` sea `true` según el valor de `SystemUtils.JAVA_VERSION`. Cubre los casos `null`, `1.8` y `>=9`. |
| `test_IS_OS()` | Valida las constantes `IS_OS_WINDOWS`, `IS_OS_LINUX`, `IS_OS_MAC_OSX`, `IS_OS_UNIX`, etc., comparándolas contra `os.name`. Incluye validación exclusiva para las versiones de macOS `IS_OS_MAC_OSX_CATALINA`, `BIG_SUR`, `VENTURA`, etc., asegurando que solo una sea verdadera a la vez. |
| `test_IS_zOS()` | Caso específico para validar la detección del sistema `z/OS` cuando `os.name` contiene `z/OS`. |
| `test_USER_NAME()` | Verifica que la constante `SystemUtils.USER_NAME` coincida con `System.getProperty("user.name")`. |

#### 5.2 Validación de Estructura de la Clase

| Método | Descripción |
| :--- | :--- |
| `testConstructor()` | Verifica que `SystemUtils` tenga un constructor público, que la clase sea pública y no sea final. Es una prueba estándar para clases de utilidad en Commons Lang. |

#### 5.3 Validación de Métodos de Acceso a Sistema y Entorno

| Método | Descripción |
| :--- | :--- |
| `testGetEnvironmentVariableAbsent()` | Verifica que `getEnvironmentVariable(name, defaultValue)` retorne el valor por defecto cuando la variable no existe. |
| `testGetEnvironmentVariablePresent()` | Verifica que retorne el valor real de `System.getenv()` cuando la variable sí existe. Usa `PATH` como caso de prueba. |
| `testGetHostName()` | Valida `getHostName()` comparándolo con `COMPUTERNAME` en Windows o `HOSTNAME` en Unix. |
| `testGetJavaHome()` / `testGetJavaHomePath()` | Verifica que `getJavaHome()` y `getJavaHomePath()` retornen un directorio existente y no nulo. |
| `testGetJavaIoTmpDir()` / `testGetJavaIoTmpDirPath()` | Valida el directorio temporal de Java `java.io.tmpdir`. |
| `testGetUserDir()` / `testGetUserDirPath()` | Valida el directorio de trabajo del usuario `user.dir`. |
| `testGetUserHome()` / `testGetUserHomePath()` | Valida el directorio home del usuario `user.home`. |
| `testGetUserName()` | Valida `getUserName()` y su sobrecarga `getUserName(defaultValue)`. |

#### 5.4 Validación de Lógica de Comparación de Versiones

| Método | Descripción |
| :--- | :--- |
| `testIsJavaVersionAtLeast(JavaVersion)` | Prueba exhaustiva que verifica que `isJavaVersionAtLeast()` retorne `true` para todas las versiones menores o iguales a la actual y `false` para las superiores. Contiene una rama `if/else` para cada versión desde Java 1.8 hasta Java 27. |
| `testIsJavaVersionAtMost(JavaVersion)` | Lógica inversa a la anterior. Verifica que `isJavaVersionAtMost()` retorne `true` para versiones mayores o iguales a la actual y `false` para las inferiores. |
| `testJavaVersionMatches()` | Valida `isJavaVersionMatch(String version, String prefix)`. Prueba con valores `null`, vacío, `1.0` hasta `25` y versiones con patch como `1.3.0`, asegurando que el matching sea por prefijo. |
| `testJavaAwtHeadless()` | Valida la constante `JAVA_AWT_HEADLESS` y el método `isJavaAwtHeadless()` contra `System.getProperty("java.awt.headless")`. |

#### 5.5 Validación de Lógica de Comparación de Sistema Operativo

| Método | Descripción |
| :--- | :--- |
| `testOSMatchesName()` | Valida `isOsNameMatch(osName, prefix)` con casos como `Windows 95`, `Windows NT` y `OS/2`. |
| `testOSMatchesNameAndVersion()` | Valida `isOsMatch(osName, osVersion, prefix, versionPrefix)` para combinaciones de nombre y versión. Ej: `Windows 95` + `4.1`. |
| `testOsVersionMatches()` | Valida `isOsVersionMatch(osVersion, prefix)` con lógica de prefijo de versión. Ej: `10` debe hacer match con `10.1` y `10.10.1`, pero `10.1` no debe hacer match con `10.10`. |

### 6. Conclusión

`SystemUtilsTest` es una clase de pruebas de alta cobertura y de naturaleza regresiva. No contiene lógica de negocio propia, sino que actúa como garante de calidad para `SystemUtils`. Su diseño es altamente parametrizado y extensible, permitiendo añadir soporte para nuevas versiones de Java y macOS sin reescribir la lógica de validación, gracias al uso de reflexión y bucles dinámicos.