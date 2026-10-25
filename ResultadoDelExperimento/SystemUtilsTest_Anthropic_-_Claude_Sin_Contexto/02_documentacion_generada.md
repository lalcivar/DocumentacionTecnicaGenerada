# Documentación técnica: `SystemUtilsTest`

**Paquete:** `org.apache.commons.lang3`
**Clase padre:** `AbstractLangTest`
**Framework de pruebas:** JUnit 5 (Jupiter)
**Licencia:** Apache License 2.0 (Apache Software Foundation)

---

## 1. Descripción general

`SystemUtilsTest` es la clase de pruebas unitarias de la utilidad `SystemUtils` de Apache Commons Lang 3. Su objetivo es verificar que las constantes y los métodos de `SystemUtils` reflejen correctamente la información del entorno de ejecución: versión de Java, sistema operativo, propiedades del sistema (`user.name`, `java.home`, `java.io.tmpdir`, etc.) y variables de entorno.

Dado que la clase bajo prueba depende del entorno real donde se ejecuta la JVM, las verificaciones son necesariamente **condicionales**: los tests detectan primero el entorno (leyendo `System.getProperty("os.name")`, `SystemUtils.JAVA_VERSION`, etc.) y luego comparan los valores esperados con los expuestos por `SystemUtils`. Por ello el propio Javadoc original advierte que *"solo puede realizarse una verificación limitada"*. Cuando el entorno no es reconocido, el test no falla: simplemente imprime un mensaje informativo por consola (`System.out` / `System.err`).

### Supuestos y limitaciones

- Varios tests asumen que **no existe un `SecurityManager`** activo, ya que leen propiedades del sistema y acceden al sistema de archivos.
- Algunos tests usan **reflexión** (`java.lang.reflect.Field`, `Constructor`, `Modifier`) para descubrir dinámicamente las constantes `IS_JAVA_X` y validar la estructura de la clase.
- Se emplea `@SuppressWarnings("deprecation")` donde se accede a constantes obsoletas (por ejemplo `IS_JAVA_1_1` … `IS_JAVA_1_9`).

---

## 2. Métodos auxiliares (helpers)

| Método | Visibilidad | Descripción |
|---|---|---|
| `getIS_JAVA(int version)` | `private` | Obtiene por reflexión el valor booleano del campo estático `SystemUtils.IS_JAVA_<version>` para versiones ≥ 9. Permite escribir aserciones en bucle sin enumerar cada constante. Lanza `Exception` si el campo no existe. |
| `getLastSupportedJavaVersion()` | `public` | Recorre todos los campos públicos de `SystemUtils` y, filtrando con la expresión regular `IS_JAVA_\d+`, devuelve el número de versión más alto soportado. Hace que los tests se adapten automáticamente cuando se añaden nuevas versiones de Java a la librería. |

---

## 3. Métodos de prueba principales

### 3.1 Detección de versión de Java

| Método | Qué valida |
|---|---|
| `test_IS_JAVA()` | Comprueba la coherencia de todas las constantes `IS_JAVA_*` frente a `SystemUtils.JAVA_VERSION`. Cubre tres escenarios: versión nula (todas deben ser `false`), versiones `1.8.x` (solo `IS_JAVA_1_8` en `true`) y versiones modernas (`9+`), donde compara cada constante con `javaVersion.startsWith(...)` usando el helper `getIS_JAVA`. |
| `testIsJavaVersionAtLeast()` | Verifica `SystemUtils.isJavaVersionAtLeast(JavaVersion)`. Según la versión detectada en tiempo de ejecución, todas las versiones iguales o anteriores deben devolver `true` y las posteriores `false`. Se implementa como una cadena de `if/else if` por versión (de 1.8 a 27). |
| `testIsJavaVersionAtMost()` | Equivalente al anterior pero para `isJavaVersionAtMost(JavaVersion)`: las versiones iguales o superiores deben devolver `true` y las anteriores `false`. |
| `testJavaVersionMatches()` | Prueba el método estático `isJavaVersionMatch(String javaVersion, String versionPrefix)` con valores de entrada controlados (`null`, cadena vacía, `"1.0"`, `"1.3.1"`, `"9"`, `"17"`, `"21"`, `"25"`, …), asegurando que solo haya coincidencia exacta de la línea de versión. Este test es **independiente del entorno**. |

### 3.2 Detección del sistema operativo

| Método | Qué valida |
|---|---|
| `test_IS_OS()` | Lee `os.name` y comprueba la consistencia de las constantes `IS_OS_WINDOWS`, `IS_OS_UNIX`, `IS_OS_LINUX`, `IS_OS_MAC_OSX`, `IS_OS_SOLARIS`, `IS_OS_SUN_OS`, `IS_OS_FREE_BSD`, `IS_OS_OS2`, `IS_OS_NETWARE`, `IS_OS_ANDROID`. En macOS, además verifica mediante `BooleanUtils.or` / `BooleanUtils.xor` que **como máximo una** de las constantes de versión (Cheetah … Sequoia) sea `true` (exclusión mutua). |
| `test_IS_zOS()` | Comprueba que `IS_OS_ZOS` sea `true` si `os.name` contiene `"z/OS"` y que en ese caso `IS_OS_WINDOWS` sea `false`. |
| `testOSMatchesName()` | Prueba `isOsNameMatch(String osName, String prefix)` con entradas fijas (`null`, `""`, `"Windows 95"`, `"Windows NT"`, `"OS/2"`). |
| `testOSMatchesNameAndVersion()` | Prueba `isOsMatch(osName, osVersion, namePrefix, versionPrefix)` combinando nombre y versión del SO. |
| `testOsVersionMatches()` | Prueba `isOsVersionMatch(osVersion, versionPrefix)` verificando la comparación por segmentos separados por puntos (p. ej. `"10.1"` **no** coincide con `"10.10"`). |

### 3.3 Propiedades del sistema y directorios

| Método | Qué valida |
|---|---|
| `test_USER_NAME()` | `SystemUtils.USER_NAME` debe coincidir con `System.getProperty("user.name")`. |
| `testGetUserName()` | Valida `getUserName()` y la sobrecarga `getUserName(String defaultValue)`. |
| `testGetJavaHome()` / `testGetJavaHomePath()` | El directorio devuelto (`File` / `Path`) no debe ser `null` y debe existir. |
| `testGetJavaIoTmpDir()` / `testGetJavaIoTmpDirPath()` | Ídem para el directorio temporal. |
| `testGetUserDir()` / `testGetUserDirPath()` | Ídem para el directorio de trabajo actual. |
| `testGetUserHome()` / `testGetUserHomePath()` | Ídem para el directorio *home* del usuario. |
| `testJavaAwtHeadless()` | Verifica la coherencia entre la propiedad `java.awt.headless`, la constante `JAVA_AWT_HEADLESS` y el método `isJavaAwtHeadless()`, incluyendo el valor por defecto `"false"`. |

### 3.4 Variables de entorno y nombre de host

| Método | Qué valida |
|---|---|
| `testGetEnvironmentVariablePresent()` | `getEnvironmentVariable("PATH", null)` debe devolver el mismo valor que `System.getenv("PATH")`. |
| `testGetEnvironmentVariableAbsent()` | Para una variable inexistente, debe devolverse el valor por defecto (`"DEFAULT"`). |
| `testGetHostName()` | `getHostName()` debe coincidir con `COMPUTERNAME` en Windows o `HOSTNAME` en el resto de plataformas. |

### 3.5 Contrato estructural de la clase

| Método | Qué valida |
|---|---|
| `testConstructor()` | Mediante reflexión comprueba que `SystemUtils` es instanciable (`new SystemUtils()` no nulo), que declara **exactamente un** constructor y que este es `public`, y que la clase es `public` pero **no** `final`. Esto documenta y protege el contrato de compatibilidad binaria histórico de la clase. |

---

## 4. Dependencias relevantes

- `org.junit.jupiter.api.Test` y `Assertions` (`assertEquals`, `assertTrue`, `assertFalse`, `assertNull`, `assertNotNull`).
- `org.apache.commons.lang3.SystemUtils`, `JavaVersion`, `BooleanUtils`.
- API estándar: `java.io.File`, `java.nio.file.Path`/`Files`, `java.lang.reflect.*`, `java.util.Locale`.

---

## 5. Notas de mantenimiento

- Los tests `testIsJavaVersionAtLeast` y `testIsJavaVersionAtMost` están escritos de forma explícita y repetitiva: al añadir soporte para una nueva versión de Java en `SystemUtils`/`JavaVersion` es necesario ampliar manualmente estas cadenas de condiciones.
- En cambio, `test_IS_JAVA` sí es extensible automáticamente gracias a `getLastSupportedJavaVersion()` y `getIS_JAVA(int)`, por lo que es el patrón recomendado para nuevas aserciones.
- El uso de `Locale.ENGLISH` en la comparación de `os.name` (caso Linux) evita problemas de localización al pasar a minúsculas.