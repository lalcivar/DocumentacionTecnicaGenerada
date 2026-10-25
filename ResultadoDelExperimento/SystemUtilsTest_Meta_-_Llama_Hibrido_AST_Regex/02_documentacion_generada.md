# Documentación Técnica - `SystemUtilsTest`

**Paquete:** `org.apache.commons.lang3`  
**Clase:** `SystemUtilsTest extends AbstractLangTest`  
**Licencia:** Apache License, Version 2.0  
**Dependencias:** JUnit Jupiter API (`org.junit.jupiter.api`), `org.apache.commons.lang3.SystemUtils`, `org.apache.commons.lang3.JavaVersion`, `org.apache.commons.lang3.BooleanUtils`

---

### 1. Resumen Ejecutivo

`SystemUtilsTest` es la clase de pruebas unitarias para `org.apache.commons.lang3.SystemUtils` dentro de Apache Commons Lang 3. Su propósito es validar de forma exhaustiva la correcta detección del entorno de ejecución, incluyendo la versión de Java (`IS_JAVA_*`), el sistema operativo (`IS_OS_*`), variables de entorno, directorios del sistema (`java.home`, `user.home`, `java.io.tmpdir`) y utilidades de comparación de versiones. La clase extiende `AbstractLangTest` y utiliza JUnit 5 para garantizar que las constantes y métodos de `SystemUtils` reflejen fielmente las propiedades del sistema (`System.getProperty`, `System.getenv`).

### 2. Especificación de Métodos y Constructores

#### 2.1 Definición de Clase

```java
/**
 * Tests {@link SystemUtils}.
 * Only limited testing can be performed.
 *
 * @author Apache Software Foundation
 * @since 3.x
 * @see SystemUtils
 * @see AbstractLangTest
 */
class SystemUtilsTest extends AbstractLangTest
```

Clase de visibilidad de paquete (`package-private`), no `final`, no `abstract`. Hereda de `AbstractLangTest`. No declara campos ni clases internas.

#### 2.2 Constructor

##### `SystemUtilsTest()`
Constructor por defecto implícito generado por el compilador. No declarado explícitamente en el código fuente.

| Elemento | Descripción |
| :--- | :--- |
| **Firma** | `SystemUtilsTest()` |
| **Visibilidad** | package-private (default) |
| **@return** | Nueva instancia de `SystemUtilsTest` |
| **@throws** | No lanza excepciones |

> **Nota:** El método `testConstructor()` valida mediante reflexión que `SystemUtils` posee un único constructor `public` y que la clase no es `final`.

#### 2.3 Métodos Auxiliares

##### `getIS_JAVA()`

```java
/**
 * Returns the value of the SystemUtils.IS_JAVA_X field for the versions >= 9.
 *
 * @param version el número de versión de Java a consultar (ej. 9, 11, 17)
 * @return true si SystemUtils.IS_JAVA_{version} es true, false en caso contrario
 * @throws Exception si el campo no existe o no es accesible vía reflexión
 */
private boolean getIS_JAVA(final int version) throws Exception
```

| @param | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `version` | `int` | Sí | Número de versión Java >= 9. Se concatena a `"IS_JAVA_"` para obtener el campo vía reflexión. |

| @return | Tipo | Descripción |
| :--- | :--- | :--- |
| `boolean` | `boolean` | Valor del campo estático `SystemUtils.IS_JAVA_{version}` |

| @throws | Tipo | Condición |
| :--- | :--- | :--- |
| `Exception` | `NoSuchFieldException`, `IllegalAccessException` | Si el campo `IS_JAVA_{version}` no existe en `SystemUtils` o no es accesible. |

##### `getLastSupportedJavaVersion()`

```java
/**
 * Returns the last supported version with the SystemUtils.IS_JAVA_X fields.
 *
 * @return el número entero más alto encontrado en los campos que coinciden con IS_JAVA_\d+
 * @throws NumberFormatException implícito si el sufijo del campo no es parseable (no ocurre en el código actual)
 */
public int getLastSupportedJavaVersion()
```

| @param | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| *Ninguno* | - | - | Método sin parámetros. Itera sobre `SystemUtils.class.getFields()` |

| @return | Tipo | Descripción |
| :--- | :--- | :--- |
| `int` | `int` | Última versión soportada detectada (ej. 26, 27). Retorna 0 si no se encuentra ningún campo. |

| @throws | Tipo | Condición |
| :--- | :--- | :--- |
| *Ninguno* | - | No declara excepciones comprobadas. |

#### 2.4 Métodos de Prueba (JUnit 5)

Todos los métodos de prueba son de visibilidad package-private, anotados con `@Test`, retornan `void` y no reciben parámetros.

##### `test_IS_JAVA()`

```java
/**
 * Valida las constantes IS_JAVA_1_1 hasta IS_JAVA_26 y las dinámicas IS_JAVA_X >=9
 * contra SystemUtils.JAVA_VERSION.
 *
 * @return void
 * @throws Exception propagada por getIS_JAVA(version) vía reflexión
 */
@Test
@SuppressWarnings("deprecation")
void test_IS_JAVA() throws Exception
```

| @param | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| *Ninguno* | - | - | - |

| @return | Tipo | Descripción |
| :--- | :--- | :--- |
| `void` | `void` | No retorna valor. Valida con `assertTrue`/`assertFalse`/`assertEquals`. |

| @throws | Tipo | Condición |
| :--- | :--- | :--- |
| `Exception` | `Exception` | Propagada por `getIS_JAVA()` si falla la reflexión. |

##### `test_IS_OS()`

```java
/**
 * Valida las constantes IS_OS_* (WINDOWS, UNIX, SOLARIS, LINUX, MAC_OSX, etc.)
 * contra System.getProperty("os.name").
 *
 * @return void
 * @throws AssertionError si la detección de OS no coincide con el valor esperado
 */
@Test
void test_IS_OS()
```

| @param | - | - | - |
| @return | `void` | - | - |
| @throws | `AssertionError` | Fallo de aserción JUnit | - |

##### `test_IS_zOS()`

```java
/**
 * Valida específicamente la constante IS_OS_ZOS cuando os.name contiene "z/OS".
 *
 * @return void
 */
@Test
void test_IS_zOS()
```

| @param | - | - | - |
| @return | `void` | - | - |
| @throws | `AssertionError` | Si `IS_OS_ZOS` no coincide con el estado esperado | - |

##### `test_USER_NAME()`

```java
/**
 * Assumes no security manager exists.
 * Valida que SystemUtils.USER_NAME coincida con System.getProperty("user.name").
 *
 * @return void
 */
@Test
void test_USER_NAME()
```

| @param | - | - | - |
| @return | `void` | - | - |
| @throws | `AssertionError` | Si los valores no coinciden | - |

##### `testConstructor()`

```java
/**
 * Valida mediante reflexión que SystemUtils tiene un único constructor público
 * y que la clase es pública y no final.
 *
 * @return void
 */
@Test
void testConstructor()
```

| @param | - | - | - |
| @return | `void` | - | - |
| @throws | `AssertionError` | Si el número de constructores o modificadores no es el esperado | - |

##### `testGetEnvironmentVariableAbsent()`

```java
/**
 * Valida SystemUtils.getEnvironmentVariable(name, defaultValue) cuando la variable no existe.
 *
 * @return void
 */
@Test
void testGetEnvironmentVariableAbsent()
```

| @param | - | - | - |
| @return | `void` | - | - |
| @throws | `AssertionError` | Si no retorna el valor por defecto `"DEFAULT"` | - |

##### `testGetEnvironmentVariablePresent()`

```java
/**
 * Valida SystemUtils.getEnvironmentVariable(name, defaultValue) cuando la variable existe (PATH).
 *
 * @return void
 */
@Test
void testGetEnvironmentVariablePresent()
```

| @param | - | - | - |
| @return | `void` | - | - |
| @throws | `AssertionError` | Si no retorna `System.getenv("PATH")` | - |

##### `testGetHostName()`

```java
/**
 * Valida SystemUtils.getHostName() contra System.getenv("COMPUTERNAME") en Windows
 * o System.getenv("HOSTNAME") en Unix.
 *
 * @return void
 */
@Test
void testGetHostName()
```

| @param | - | - | - |
| @return | `void` | - | - |
| @throws | `AssertionError` | Si el hostname no coincide | - |

##### `testGetJavaHome()`

```java
/**
 * Assumes no security manager exists.
 * Valida que SystemUtils.getJavaHome() retorne un File existente y no nulo.
 *
 * @return void
 */
@Test
void testGetJavaHome()
```

| @param | - | - | - |
| @return | `void` | - | - |
| @throws | `AssertionError` | Si el directorio es nulo o no existe | - |

##### `testGetJavaHomePath()`

```java
/**
 * Assumes no security manager exists.
 * Valida que SystemUtils.getJavaHomePath() retorne un Path existente y no nulo.
 *
 * @return void
 */
@Test
void testGetJavaHomePath()
```

| @param | - | - | - |
| @return | `void` | - | - |
| @throws | `AssertionError` | Si el Path es nulo o no existe (`Files.exists`) | - |

##### `testGetJavaIoTmpDir()`

```java
/**
 * Assumes no security manager exists.
 * Valida SystemUtils.getJavaIoTmpDir().
 *
 * @return void
 */
@Test
void testGetJavaIoTmpDir()
```

| @param | - | - | - |
| @return | `void` | - | - |
| @throws | `AssertionError` | Si el File es nulo o no existe | - |

##### `testGetJavaIoTmpDirPath()`

```java
/**
 * Assumes no security manager exists.
 * Valida SystemUtils.getJavaIoTmpDirPath().
 *
 * @return void
 */
@Test
void testGetJavaIoTmpDirPath()
```

| @param | - | - | - |
| @return | `void` | - | - |
| @throws | `AssertionError` | Si el Path es nulo o no existe | - |

##### `testGetUserDir()`

```java
/**
 * Assumes no security manager exists.
 * Valida SystemUtils.getUserDir().
 *
 * @return void
 */
@Test
void testGetUserDir()
```

| @param | - | - | - |
| @return | `void` | - | - |
| @throws | `AssertionError` | Si el File es nulo o no existe | - |

##### `testGetUserDirPath()`

```java
/**
 * Assumes no security manager exists.
 * Valida SystemUtils.getUserDirPath().
 *
 * @return void
 */
@Test
void testGetUserDirPath()
```

| @param | - | - | - |
| @return | `void` | - | - |
| @throws | `AssertionError` | Si el Path es nulo o no existe | - |

##### `testGetUserHome()`

```java
/**
 * Assumes no security manager exists.
 * Valida SystemUtils.getUserHome().
 *
 * @return void
 */
@Test
void testGetUserHome()
```

| @param | - | - | - |
| @return | `void` | - | - |
| @throws | `AssertionError` | Si el File es nulo o no existe | - |

##### `testGetUserHomePath()`

```java
/**
 * Assumes no security manager exists.
 * Valida SystemUtils.getUserHomePath().
 *
 * @return void
 */
@Test
void testGetUserHomePath()
```

| @param | - | - | - |
| @return | `void` | - | - |
| @throws | `AssertionError` | Si el Path es nulo o no existe | - |

##### `testGetUserName()`

```java
/**
 * Assumes no security manager exists.
 * Valida SystemUtils.getUserName() y SystemUtils.getUserName(defaultValue).
 *
 * @return void
 */
@Test
void testGetUserName()
```

| @param | - | - | - |
| @return | `void` | - | - |
| @throws | `AssertionError` | Si no coincide con `System.getProperty("user.name")` | - |

##### `testIsJavaVersionAtLeast()`

```java
/**
 * Valida SystemUtils.isJavaVersionAtLeast(JavaVersion) para todas las versiones
 * desde JAVA_1_1 hasta JAVA_27 según la versión actual en ejecución.
 *
 * @return void
 */
@Test
void testIsJavaVersionAtLeast()
```

| @param | - | - | - |
| @return | `void` | - | - |
| @throws | `AssertionError` | Si la lógica de comparación `atLeast` es incorrecta | - |

##### `testIsJavaVersionAtMost()`

```java
/**
 * Valida SystemUtils.isJavaVersionAtMost(JavaVersion) para todas las versiones
 * desde JAVA_1_1 hasta JAVA_27 según la versión actual en ejecución.
 *
 * @return void
 */
@Test
void testIsJavaVersionAtMost()
```

| @param | - | - | - |
| @return | `void` | - | - |
| @throws | `AssertionError` | Si la lógica de comparación `atMost` es incorrecta | - |

##### `testJavaAwtHeadless()`

```java
/**
 * Valida SystemUtils.JAVA_AWT_HEADLESS y SystemUtils.isJavaAwtHeadless()
 * contra System.getProperty("java.awt.headless").
 *
 * @return void
 */
@Test
void testJavaAwtHeadless()
```

| @param | - | - | - |
| @return | `void` | - | - |
| @throws | `AssertionError` | Si el valor booleano/string no coincide con la propiedad del sistema | - |

##### `testJavaVersionMatches()`

```java
/**
 * Valida SystemUtils.isJavaVersionMatch(javaVersion, versionPrefix) con múltiples
 * combinaciones de javaVersion (null, "", "1.0"..."25") y prefijos.
 *
 * @return void
 */
@Test
void testJavaVersionMatches()
```

| @param | - | - | - |
| @return | `void` | - | - |
| @throws | `AssertionError` | Si el matching de prefijo de versión es incorrecto | - |

##### `testOSMatchesName()`

```java
/**
 * Valida SystemUtils.isOsNameMatch(osName, prefix) con casos null, "", "Windows 95", etc.
 *
 * @return void
 */
@Test
void testOSMatchesName()
```

| @param | - | - | - |
| @return | `void` | - | - |
| @throws | `AssertionError` | Si el matching de nombre de OS falla | - |

##### `testOSMatchesNameAndVersion()`

```java
/**
 * Valida SystemUtils.isOsMatch(osName, osVersion, prefixName, prefixVersion).
 *
 * @return void
 */
@Test
void testOSMatchesNameAndVersion()
```

| @param | - | - | - |
| @return | `void` | - | - |
| @throws | `AssertionError` | Si el matching combinado nombre+versión falla | - |

##### `testOsVersionMatches()`

```java
/**
 * Valida SystemUtils.isOsVersionMatch(osVersion, prefix) con versiones como "10", "10.1", "10.10.1".
 *
 * @return void
 */
@Test
void testOsVersionMatches()
```

| @param | - | - | - |
| @return | `void` | - | - |
| @throws | `AssertionError` | Si el matching de versión de OS falla | - |

> **Cobertura total:** 26 métodos documentados (2 auxiliares + 24 de prueba + 1 constructor implícito). No existen métodos `static`, `protected`, clases internas ni anónimas en el código fuente provisto.

---

### 3. Ejemplo de Uso

Ejemplo real de ejecución y reutilización de la lógica auxiliar de la clase de prueba:

```java
package org.apache.commons.lang3;

import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

public class EjemploUsoSystemUtilsTest {

    /**
     * Ejemplo de cómo reutilizar la lógica de detección de versión
     * y cómo ejecutar las validaciones de SystemUtils.
     */
    @Test
    void ejemploValidacionEntorno() throws Exception {
        // 1. Instanciar la clase de prueba para acceder a helpers
        SystemUtilsTest testHelper = new SystemUtilsTest();
        
        // 2. Obtener la última versión Java soportada por SystemUtils vía reflexión
        int lastVersion = testHelper.getLastSupportedJavaVersion();
        System.out.println("Última versión soportada: " + lastVersion); // ej. 26

        // 3. Validar directamente SystemUtils en código de aplicación
        if (SystemUtils.IS_OS_WINDOWS) {
            System.out.println("Ejecutando en Windows: " + System.getProperty("os.name"));
        }

        if (SystemUtils.isJavaVersionAtLeast(JavaVersion.JAVA_17)) {
            System.out.println("Java 17+ detectado: " + SystemUtils.JAVA_VERSION);
        }

        // 4. Validar variable de entorno con fallback
        String path = SystemUtils.getEnvironmentVariable("PATH", "/usr/bin");
        assertNotNull(path);

        // 5. Validar directorios del sistema
        assertTrue(SystemUtils.getJavaHome().exists());
        assertTrue(SystemUtils.getUserHome().exists());
    }

    // Ejecución vía Maven:
    // mvn -Dtest=org.apache.commons.lang3.SystemUtilsTest test
}
```

