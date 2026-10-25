# Documentación Técnica — `org.apache.commons.lang3.SystemUtilsTest`

---

## 1. Resumen ejecutivo

`SystemUtilsTest` es la suite de pruebas unitarias (JUnit 5 / Jupiter) que valida el comportamiento de la clase de utilidades `SystemUtils` de Apache Commons Lang 3. Su propósito es verificar, de forma condicional al entorno de ejecución real (JVM y sistema operativo activos), la coherencia de las constantes `IS_JAVA_*` / `IS_OS_*`, de los *accessors* de propiedades del sistema (`getJavaHome()`, `getUserDir()`, `getHostName()`, etc.) y de los algoritmos de comparación de versiones (`isJavaVersionAtLeast`, `isJavaVersionAtMost`, `isJavaVersionMatch`, `isOsMatch`, `isOsVersionMatch`, `isOsNameMatch`).

Dado que se trata de pruebas dependientes del entorno ("*Only limited testing can be performed*"), la clase emplea reflexión (`java.lang.reflect.Field`, `Constructor`, `Modifier`) para descubrir dinámicamente las constantes de versión soportadas y así evitar que la suite quede obsoleta al liberarse nuevas versiones de Java.

### Metadatos de la clase

| Atributo | Valor |
|---|---|
| Paquete | `org.apache.commons.lang3` |
| Nombre | `SystemUtilsTest` |
| Modificador | *package-private* (`class`) |
| Superclase | `AbstractLangTest` |
| Framework de pruebas | JUnit 5 (`org.junit.jupiter.api.Test`, `Assertions.*` estáticos) |
| Clase bajo prueba (SUT) | `SystemUtils` |
| Colaboradores internos del proyecto | `JavaVersion`, `BooleanUtils`, `AbstractLangTest` |
| `@author` | No declarado en el código fuente (la cabecera corresponde a la licencia Apache 2.0 / ASF) |
| Licencia | Apache License, Version 2.0 |

> **Nota sobre `@author`:** el archivo original no incluye la etiqueta `@author`; conforme a la convención de la ASF, la autoría es colectiva («*contributor license agreements*»). No se infiere ningún autor.

---

## 2. Especificación de métodos y constructores

### 2.0 Constructor implícito — `SystemUtilsTest()`

```java
/**
 * Constructor por defecto implícito generado por el compilador.
 * JUnit 5 crea una nueva instancia de la clase de prueba por cada método @Test
 * (ciclo de vida por defecto PER_METHOD).
 */
SystemUtilsTest()
```

| Elemento | Detalle |
|---|---|
| Declaración | No existe constructor explícito en el código fuente; el compilador genera el constructor *package-private* sin argumentos. |
| Parámetros | Ninguno |
| `@return` | N/A (constructor) |
| `@throws` | Ninguna excepción declarada |
| Observación | Delega implícitamente en el constructor de `AbstractLangTest` vía `super()`. |

---

### 2.1 `getIS_JAVA(int)`

```java
/**
 * Returns the value of the SystemUtils.IS_JAVA_X field for the versions >= 9.
 *
 * @param version número mayor de versión de Java (>= 9) cuyo campo constante se desea leer.
 * @return el valor booleano del campo {@code SystemUtils.IS_JAVA_<version>}.
 * @throws Exception si el campo no existe, no es accesible o no es de tipo boolean.
 */
private boolean getIS_JAVA(final int version) throws Exception
```

**Propósito.** Lee por reflexión el campo público estático `SystemUtils.IS_JAVA_<version>` (p. ej. `IS_JAVA_21`) y devuelve su valor booleano. Permite verificar las constantes de versión sin acoplar el test a un listado estático de identificadores.

**Parámetros de entrada (`@param`)**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| `version` | `int` (final) | Sí | Número de versión mayor de Java (uso previsto: `>= 9`). Se concatena al prefijo `"IS_JAVA_"` para construir el nombre del campo. |

**Valor de retorno (`@return`)**

| Tipo | Descripción |
|---|---|
| `boolean` | Valor del campo estático `SystemUtils.IS_JAVA_<version>`. `true` si la JVM en ejecución corresponde a esa versión mayor. |

**Excepciones (`@throws`)**

| Excepción | Condición |
|---|---|
| `java.lang.NoSuchFieldException` | No existe el campo `IS_JAVA_<version>` en `SystemUtils` (versión no soportada por la librería). |
| `java.lang.SecurityException` | Un gestor de seguridad impide el acceso reflexivo al campo. |
| `java.lang.IllegalAccessException` | El campo no es accesible desde el contexto de llamada. |
| `java.lang.IllegalArgumentException` | El campo no es de tipo `boolean` (contrato de `Field#getBoolean`). |
| `java.lang.Exception` | Declarada genéricamente en la firma; engloba todas las anteriores. |

**Detalles de implementación.** Invoca `SystemUtils.class.getField("IS_JAVA_" + version).getBoolean(null)`; el argumento `null` indica acceso a campo estático.

---

### 2.2 `getLastSupportedJavaVersion()`

```java
/**
 * Returns the last supported version with the SystemUtils.IS_JAVA_X fields.
 *
 * @return el mayor número de versión N para el que existe el campo público
 *         {@code SystemUtils.IS_JAVA_N}; 0 si no se encuentra ninguno.
 */
public int getLastSupportedJavaVersion()
```

**Propósito.** Determina dinámicamente la última versión mayor de Java soportada por `SystemUtils`, inspeccionando los campos públicos cuyo nombre cumple la expresión regular `IS_JAVA_\d+` (es decir, se excluyen las constantes históricas con punto, tipo `IS_JAVA_1_8`).

**Parámetros de entrada (`@param`)**

| Nombre | Tipo | Requerido | Descripción |
|---|---|---|---|
| — | — | — | El método no recibe parámetros. |

**Valor de retorno (`@return`)**

| Tipo | Descripción |
|---|---|
| `int` | Máximo `N` extraído de los nombres `IS_JAVA_N` mediante `name.substring(8)` (longitud del prefijo `"IS_JAVA_"`). Devuelve `0` si no se detecta ningún campo coincidente. |

**Excepciones (`@throws`)**

| Excepción | Condición |
|---|---|
| `java.lang.SecurityException` | Un gestor de seguridad impide `Class#getFields()`. |
| `java.lang.NumberFormatException` | Teóricamente, si el sufijo numérico excede el rango de `int`. No se declara en la firma (*unchecked*). |

**Detalles de implementación.** Acumulador `lastSupportedVersion` inicializado a `0`, actualizado con `Math.max(...)` por cada campo coincidente obtenido de `SystemUtils.class.getFields()`.

---

### 2.3 `test_IS_JAVA()`

```java
/**
 * Verifica la coherencia de las constantes SystemUtils.IS_JAVA_* respecto a
 * SystemUtils.JAVA_VERSION.
 *
 * @throws Exception propagada desde {@link #getIS_JAVA(int)} al resolver campos por reflexión.
 */
@Test
@SuppressWarnings("deprecation")
void test_IS_JAVA() throws Exception
```

**Propósito y lógica de ramificación.** Obtiene `SystemUtils.JAVA_VERSION` y `getLastSupportedJavaVersion()`, y aplica tres escenarios mutuamente excluyentes:

| Escenario (`javaVersion`) | Aserciones aplicadas |
|---|---|
| `null` | Todas las constantes `IS_JAVA_1_1` … `IS_JAVA_26` deben ser `false`, y `getIS_JAVA(v)` debe ser `false` para `v` en `[9, lastSupportedVersion]`. |
| Comienza por `"1.8"` | Solo `IS_JAVA_1_8` es `true`; el resto (`IS_JAVA_1_1`…`IS_JAVA_1_7`, `IS_JAVA_1_9`, `IS_JAVA_10`…`IS_JAVA_26`) es `false`, al igual que todos los `getIS_JAVA(v)` para `v >= 9`. |
| **No** comienza por `"1."` (Java 9+) | `IS_JAVA_1_1`…`IS_JAVA_1_8` deben ser `false`; `IS_JAVA_1_9` se compara con `javaVersion.startsWith("9")`; para cada `v` en `[9, lastSupportedVersion]`, `getIS_JAVA(v)` se compara con `javaVersion.startsWith("" + v)`. |
| Resto (`"1.x"` con `x != 8`) | No se afirma nada: se imprime en `System.out` el mensaje `"Can't test IS_JAVA value: " + javaVersion`. |

**Parámetros / retorno / excepciones**

| Elemento | Detalle |
|---|---|
| `@param` | Ninguno |
| `@return` | `void` |
| `@throws Exception` | Propagada por `getIS_JAVA(int)` (reflexión). |
| Anotaciones | `@Test`, `@SuppressWarnings("deprecation")` (uso de constantes `IS_JAVA_1_x` marcadas como obsoletas). |

---

### 2.4 `test_IS_OS()`

```java
/**
 * Verifica la consistencia de las constantes SystemUtils.IS_OS_* frente a la
 * propiedad del sistema "os.name".
 */
@Test
void test_IS_OS()
```

**Propósito.** Lee `System.getProperty("os.name")` y valida la familia de banderas de sistema operativo según la plataforma detectada.

| Valor de `os.name` | Aserciones principales |
|---|---|
| `null` | `IS_OS_WINDOWS`, `IS_OS_UNIX`, `IS_OS_SOLARIS`, `IS_OS_LINUX`, `IS_OS_MAC_OSX`, `IS_OS_NETWARE` = `false`. |
| `Windows*` | `IS_OS_WINDOWS` = `true`; `IS_OS_ANDROID`, `IS_OS_UNIX`, `IS_OS_NETWARE` = `false`. |
| `Solaris*` | `IS_OS_SOLARIS` e `IS_OS_UNIX` = `true`; `IS_OS_ANDROID`, `IS_OS_WINDOWS`, `IS_OS_NETWARE` = `false`. |
| `linux*` (comparación en minúsculas con `Locale.ENGLISH`) | `IS_OS_LINUX`, `IS_OS_UNIX` = `true`; `IS_OS_WINDOWS`, `IS_OS_NETWARE` = `false`. |
| `Mac OS X*` | `IS_OS_MAC_OSX`, `IS_OS_UNIX` = `true`; `IS_OS_ANDROID`, `IS_OS_WINDOWS`, `IS_OS_NETWARE` = `false`. Además, construye el array `macOsValues` con las 21 constantes de versión de macOS (Cheetah … Sequoia) y, si `BooleanUtils.or(macOsValues)` es `true`, exige exclusividad mutua mediante `assertTrue(BooleanUtils.xor(macOsValues))`. |
| `OS/2*` | `IS_OS_OS2` = `true`; `IS_OS_UNIX`, `IS_OS_ANDROID`, `IS_OS_WINDOWS`, `IS_OS_NETWARE` = `false`. |
| `SunOS*` | `IS_OS_SUN_OS`, `IS_OS_UNIX` = `true`; resto `false`. |
| `FreeBSD*` | `IS_OS_FREE_BSD`, `IS_OS_UNIX` = `true`; resto `false`. |
| Otro | Mensaje informativo en `System.err`: `"Can't test IS_OS_ value: " + osName`. |

| Elemento | Detalle |
|---|---|
| `@param` | Ninguno |
| `@return` | `void` |
| `@throws` | No declara excepciones. Puede fallar con `AssertionFailedError` si una aserción no se cumple. |

---

### 2.5 `test_IS_zOS()`

```java
/**
 * Verifica la bandera SystemUtils.IS_OS_ZOS para la plataforma z/OS.
 */
@Test
void test_IS_zOS()
```

| Elemento | Detalle |
|---|---|
| Lógica | Si `os.name` es `null`, `IS_OS_ZOS` debe ser `false`. Si `os.name` contiene `"z/OS"`, entonces `IS_OS_WINDOWS` = `false` e `IS_OS_ZOS` = `true`. En cualquier otro caso no se afirma nada. |
| `@param` | Ninguno |
| `@return` | `void` |
| `@throws` | Ninguna declarada; `AssertionFailedError` en caso de fallo. |

---

### 2.6 `test_USER_NAME()`

```java
/**
 * Assumes no security manager exists.
 * Comprueba que la constante SystemUtils.USER_NAME coincide con la propiedad "user.name".
 */
@Test
void test_USER_NAME()
```

| Elemento | Detalle |
|---|---|
| Aserción | `assertEquals(System.getProperty("user.name"), SystemUtils.USER_NAME)` |
| Precondición | No debe existir un `SecurityManager` que restrinja la lectura de propiedades. |
| `@param` | Ninguno |
| `@return` | `void` |
| `@throws` | Ninguna declarada. |

---

### 2.7 `testConstructor()`

```java
/**
 * Verifica el contrato público de la clase SystemUtils: instanciabilidad, número y
 * visibilidad de constructores y modificadores de la clase.
 */
@Test
void testConstructor()
```

| Verificación | Aserción |
|---|---|
| Instanciable | `assertNotNull(new SystemUtils())` |
| Un único constructor declarado | `assertEquals(1, cons.length)` sobre `SystemUtils.class.getDeclaredConstructors()` |
| Constructor público | `assertTrue(Modifier.isPublic(cons[0].getModifiers()))` |
| Clase pública | `assertTrue(Modifier.isPublic(SystemUtils.class.getModifiers()))` |
| Clase **no** final (extensible) | `assertFalse(Modifier.isFinal(SystemUtils.class.getModifiers()))` |

| Elemento | Detalle |
|---|---|
| `@param` | Ninguno |
| `@return` | `void` |
| `@throws` | Ninguna declarada; puede lanzar `SecurityException` (*unchecked* en este contexto reflexivo). |

---

### 2.8 `testGetEnvironmentVariableAbsent()`

```java
/**
 * Verifica que SystemUtils.getEnvironmentVariable(String, String) devuelve el valor por
 * defecto cuando la variable de entorno no existe.
 */
@Test
void testGetEnvironmentVariableAbsent()
```

| Elemento | Detalle |
|---|---|
| Datos de prueba | `name = "THIS_ENV_VAR_SHOULD_NOT_EXIST_FOR_THIS_TEST_TO_PASS"` |
| Precondición verificada | `assertNull(System.getenv(name))` |
| Aserción objetivo | `assertEquals("DEFAULT", SystemUtils.getEnvironmentVariable(name, "DEFAULT"))` |
| `@param` / `@return` | Ninguno / `void` |
| `@throws` | Ninguna declarada. |

---

### 2.9 `testGetEnvironmentVariablePresent()`

```java
/**
 * Verifica que SystemUtils.getEnvironmentVariable(String, String) devuelve el valor real
 * de la variable de entorno cuando esta existe (caso "PATH").
 */
@Test
void testGetEnvironmentVariablePresent()
```

| Elemento | Detalle |
|---|---|
| Datos de prueba | `name = "PATH"`, valor por defecto `null` |
| Aserción | `assertEquals(System.getenv("PATH"), SystemUtils.getEnvironmentVariable("PATH", null))` |
| `@param` / `@return` | Ninguno / `void` |
| `@throws` | Ninguna declarada. |

---

### 2.10 `testGetHostName()`

```java
/**
 * Verifica que SystemUtils.getHostName() resuelve el nombre de host usando la variable de
 * entorno dependiente del SO: COMPUTERNAME en Windows, HOSTNAME en el resto.
 */
@Test
void testGetHostName()
```

| Elemento | Detalle |
|---|---|
| Valor esperado | `SystemUtils.IS_OS_WINDOWS ? System.getenv("COMPUTERNAME") : System.getenv("HOSTNAME")` |
| Aserción | `assertEquals(expected, SystemUtils.getHostName())` |
| `@param` / `@return` | Ninguno / `void` |
| `@throws` | Ninguna declarada. |

---

### 2.11 `testGetJavaHome()`

```java
/**
 * Assumes no security manager exists.
 * Verifica que SystemUtils.getJavaHome() devuelve un File no nulo y existente.
 */
@Test
void testGetJavaHome()
```

| Elemento | Detalle |
|---|---|
| Aserciones | `assertNotNull(dir)`; `assertTrue(dir.exists())` sobre `File dir = SystemUtils.getJavaHome()` |
| Precondición | Ausencia de `SecurityManager`. |
| `@param` / `@return` | Ninguno / `void` |
| `@throws` | Ninguna declarada. |

---

### 2.12 `testGetJavaHomePath()`

```java
/**
 * Assumes no security manager exists.
 * Verifica que SystemUtils.getJavaHomePath() devuelve un Path no nulo y existente.
 */
@Test
void testGetJavaHomePath()
```

| Elemento | Detalle |
|---|---|
| Aserciones | `assertNotNull(dir)`; `assertTrue(Files.exists(dir))` sobre `Path dir = SystemUtils.getJavaHomePath()` |
| `@param` / `@return` | Ninguno / `void` |
| `@throws` | Ninguna declarada. |

---

### 2.13 `testGetJavaIoTmpDir()`

```java
/**
 * Assumes no security manager exists.
 * Verifica que SystemUtils.getJavaIoTmpDir() devuelve un File no nulo y existente.
 */
@Test
void testGetJavaIoTmpDir()
```

| Elemento | Detalle |
|---|---|
| Aserciones | `assertNotNull(dir)`; `assertTrue(dir.exists())` |
| `@param` / `@return` | Ninguno / `void` |
| `@throws` | Ninguna declarada. |

---

### 2.14 `testGetJavaIoTmpDirPath()`

```java
/**
 * Assumes no security manager exists.
 * Verifica que SystemUtils.getJavaIoTmpDirPath() devuelve un Path no nulo y existente.
 */
@Test
void testGetJavaIoTmpDirPath()
```

| Elemento | Detalle |
|---|---|
| Aserciones | `assertNotNull(dir)`; `assertTrue(Files.exists(dir))` |
| `@param` / `@return` | Ninguno / `void` |
| `@throws` | Ninguna declarada. |

---

### 2.15 `testGetUserDir()`

```java
/**
 * Assumes no security manager exists.
 * Verifica que SystemUtils.getUserDir() devuelve un File no nulo y existente.
 */
@Test
void testGetUserDir()
```

| Elemento | Detalle |
|---|---|
| Aserciones | `assertNotNull(dir)`; `assertTrue(dir.exists())` |
| `@param` / `@return` | Ninguno / `void` |
| `@throws` | Ninguna declarada. |

---

### 2.16 `testGetUserDirPath()`

```java
/**
 * Assumes no security manager exists.
 * Verifica que SystemUtils.getUserDirPath() devuelve un Path no nulo y existente.
 */
@Test
void testGetUserDirPath()
```

| Elemento | Detalle |
|---|---|
| Aserciones | `assertNotNull(dir)`; `assertTrue(Files.exists(dir))` |
| `@param` / `@return` | Ninguno / `void` |
| `@throws` | Ninguna declarada. |

---

### 2.17 `testGetUserHome()`

```java
/**
 * Assumes no security manager exists.
 * Verifica que SystemUtils.getUserHome() devuelve un File no nulo y existente.
 */
@Test
void testGetUserHome()
```

| Elemento | Detalle |
|---|---|
| Aserciones | `assertNotNull(dir)`; `assertTrue(dir.exists())` |
| `@param` / `@return` | Ninguno / `void` |
| `@throws` | Ninguna declarada. |

---

### 2.18 `testGetUserHomePath()`

```java
/**
 * Assumes no security manager exists.
 * Verifica que SystemUtils.getUserHomePath() devuelve un Path no nulo y existente.
 */
@Test
void testGetUserHomePath()
```

| Elemento | Detalle |
|---|---|
| Aserciones | `assertNotNull(dir)`; `assertTrue(Files.exists(dir))` |
| `@param` / `@return` | Ninguno / `void` |
| `@throws` | Ninguna declarada. |

---

### 2.19 `testGetUserName()`

```java
/**
 * Assumes no security manager exists.
 * Verifica las dos sobrecargas de SystemUtils.getUserName().
 */
@Test
void testGetUserName()
```

| Verificación | Aserción |
|---|---|
| Sobrecarga sin argumentos | `assertEquals(System.getProperty("user.name"), SystemUtils.getUserName())` |
| Sobrecarga con valor por defecto | `assertEquals(System.getProperty("user.name", "foo"), SystemUtils.getUserName("foo"))` |

| Elemento | Detalle |
|---|---|
| Comentario de diseño | El test evita deliberadamente sobrescribir la propiedad del sistema («*Don't overwrite the system property in this test…*»), preservando el aislamiento del entorno. |
| `@param` / `@return` | Ninguno / `void` |
| `@throws` | Ninguna declarada. |

---

### 2.20 `testIsJavaVersionAtLeast()`

```java
/**
 * Verifica SystemUtils.isJavaVersionAtLeast(JavaVersion) para la JVM en ejecución,
 * cubriendo el rango JAVA_1_1 .. JAVA_27.
 */
@Test
void testIsJavaVersionAtLeast()
```

**Propósito.** Estructura de `if / else if` sobre las constantes `IS_JAVA_1_8`, `IS_JAVA_9` … `IS_JAVA_27`. Para la rama que corresponda a la JVM activa, afirma:

- `assertTrue(...)` para todas las `JavaVersion` **iguales o inferiores** a la versión detectada.
- `assertFalse(...)` para todas las `JavaVersion` **superiores**, hasta `JavaVersion.JAVA_27` inclusive.

Si ninguna constante `IS_JAVA_*` es `true` (versión desconocida), el método no ejecuta aserciones y termina con éxito (*no-op*).

| Elemento | Detalle |
|---|---|
| Ramas cubiertas | `IS_JAVA_1_8`, `IS_JAVA_9`, `IS_JAVA_10` … `IS_JAVA_27` (20 ramas) |
| `@param` | Ninguno |
| `@return` | `void` |
| `@throws` | Ninguna declarada; `AssertionFailedError` en caso de fallo. |

---

### 2.21 `testIsJavaVersionAtMost()`

```java
/**
 * Verifica SystemUtils.isJavaVersionAtMost(JavaVersion) para la JVM en ejecución,
 * cubriendo el rango JAVA_1_1 .. JAVA_27.
 */
@Test
void testIsJavaVersionAtMost()
```

**Propósito.** Simétrico a `testIsJavaVersionAtLeast()`. En la rama activa afirma:

- `assertFalse(...)` para toda `JavaVersion` **inferior** a la detectada.
- `assertTrue(...)` para la versión detectada y todas las **superiores**.

| Elemento | Detalle |
|---|---|
| Ramas cubiertas | `IS_JAVA_1_8`, `IS_JAVA_9` … `IS_JAVA_27` |
| `@param` | Ninguno |
| `@return` | `void` |
| `@throws` | Ninguna declarada. |
| Observaciones de calidad | (a) La rama `IS_JAVA_9` contiene una aserción duplicada: `assertTrue(... JAVA_27)` aparece dos veces (redundante, inocua). (b) La rama `IS_JAVA_20` termina en `JAVA_26`, **omitiendo** la aserción para `JavaVersion.JAVA_27`, a diferencia del resto de ramas. Ambos puntos son candidatos a refactorización (p. ej. `@ParameterizedTest` iterando `JavaVersion.values()`). |

---

### 2.22 `testJavaAwtHeadless()`

```java
/**
 * Verifica la constante SystemUtils.JAVA_AWT_HEADLESS y el método
 * SystemUtils.isJavaAwtHeadless() frente a la propiedad "java.awt.headless".
 */
@Test
void testJavaAwtHeadless()
```

| Paso | Detalle |
|---|---|
| 1 | `expectedStringValue = System.getProperty("java.awt.headless")` |
| 2 | `expectedStringValueWithDefault = System.getProperty("java.awt.headless", "false")`; se exige `assertNotNull(...)`. |
| 3 | `expectedValue = Boolean.parseBoolean(expectedStringValue)` |
| 4 | Si la propiedad está definida (`!= null`): `assertEquals(expectedStringValue, SystemUtils.JAVA_AWT_HEADLESS)`. |
| 5 | `assertEquals(expectedValue, SystemUtils.isJavaAwtHeadless())` |
| 6 | `assertEquals(expectedStringValueWithDefault, "" + SystemUtils.isJavaAwtHeadless())` — compara la representación textual del booleano con el valor de la propiedad (o `"false"` por defecto). |

| Elemento | Detalle |
|---|---|
| `@param` / `@return` | Ninguno / `void` |
| `@throws` | Ninguna declarada. |

---

### 2.23 `testJavaVersionMatches()`

```java
/**
 * Verifica el algoritmo SystemUtils.isJavaVersionMatch(String, String) con una matriz
 * exhaustiva de versiones de entrada y prefijos de versión esperados.
 */
@Test
void testJavaVersionMatches()
```

**Propósito.** Prueba de tabla (*table-driven*) sobre la variable local `javaVersion`, evaluada contra los 25 prefijos `"1.0"`, `"1.1"` … `"1.8"`, `"9"`, `"10"` … `"25"`. Para cada valor de entrada se espera exactamente una coincidencia `true` (o ninguna en los casos degenerados).

| Valor de `javaVersion` | Resultado esperado |
|---|---|
| `null` | `false` para los 25 prefijos. |
| `""` (cadena vacía) | `false` para los 25 prefijos. |
| `"1.0"`, `"1.1"`, `"1.2"` | `true` solo para su prefijo homónimo. |
| `"1.3.0"`, `"1.3.1"` | `true` solo para `"1.3"`. |
| `"1.4.0"`, `"1.4.1"`, `"1.4.2"` | `true` solo para `"1.4"`. |
| `"1.5.0"`, `"1.6.0"`, `"1.7.0"`, `"1.8.0"` | `true` solo para `"1.5"`, `"1.6"`, `"1.7"`, `"1.8"` respectivamente. |
| `"9"`, `"10"`, `"11"` (LTS), `"17"` (LTS), `"21"` (LTS), `"22"`, `"23"`, `"24"`, `"25"` | `true` solo para su prefijo homónimo. |

| Elemento | Detalle |
|---|---|
| `@param` / `@return` | Ninguno / `void` |
| `@throws` | Ninguna declarada. |

---

### 2.24 `testOSMatchesName()`

```java
/**
 * Verifica SystemUtils.isOsNameMatch(String, String) con distintos valores de "os.name".
 */
@Test
void testOSMatchesName()
```

| `osName` | Prefijo esperado | Resultado |
|---|---|---|
| `null` | `"Windows"` | `false` |
| `""` | `"Windows"` | `false` |
| `"Windows 95"` | `"Windows"` | `true` |
| `"Windows NT"` | `"Windows"` | `true` |
| `"OS/2"` | `"Windows"` | `false` |

| Elemento | Detalle |
|---|---|
| `@param` / `@return` | Ninguno / `void` |
| `@throws` | Ninguna declarada. |

---

### 2.25 `testOSMatchesNameAndVersion()`

```java
/**
 * Verifica SystemUtils.isOsMatch(String, String, String, String), que combina
 * coincidencia de nombre de SO y de versión.
 */
@Test
void testOSMatchesNameAndVersion()
```

| `osName` | `osVersion` | Patrón nombre | Patrón versión | Resultado |
|---|---|---|---|---|
| `null` | `null` | `"Windows 9"` | `"4.1"` | `false` |
| `""` | `""` | `"Windows 9"` | `"4.1"` | `false` |
| `"Windows 95"` | `"4.0"` | `"Windows 9"` | `"4.1"` | `false` |
| `"Windows 95"` | `"4.1"` | `"Windows 9"` | `"4.1"` | `true` |
| `"Windows 98"` | `"4.1"` | `"Windows 9"` | `"4.1"` | `true` |
| `"Windows NT"` | `"4.0"` | `"Windows 9"` | `"4.1"` | `false` |
| `"OS/2"` | `"4.0"` | `"Windows 9"` | `"4.1"` | `false` |

| Elemento | Detalle |
|---|---|
| `@param` / `@return` | Ninguno / `void` |
| `@throws` | Ninguna declarada. |

---

### 2.26 `testOsVersionMatches()`

```java
/**
 * Verifica SystemUtils.isOsVersionMatch(String, String), validando la comparación
 * segmentada por puntos (evitando falsos positivos entre "10.1" y "10.10").
 */
@Test
void testOsVersionMatches()
```

| `osVersion` (patrón base) | Versión evaluada | Resultado |
|---|---|---|
| `null` | `"10.1"` | `false` |
| `""` | `"10.1"` | `false` |
| `"10"` | `"10.1"`, `"10.1.1"`, `"10.10"`, `"10.10.1"` | `true` en los cuatro casos |
| `"10.1"` | `"10.1"`, `"10.1.1"` / `"10.10"`, `"10.10.1"` | `true` / `false` |
| `"10.1.1"` | `"10.1"`, `"10.1.1"` / `"10.10"`, `"10.10.1"` | `true` / `false` |
| `"10.10"` | `"10.1"`, `"10.1.1"` / `"10.10"`, `"10.10.1"` | `false` / `true` |
| `"10.10.1"` | `"10.1"`, `"10.1.1"` / `"10.10"`, `"10.10.1"` | `false` / `true` |

| Elemento | Detalle |
|---|---|
| Valor de la prueba | Garantiza que la comparación es por segmentos numéricos y no por prefijo de cadena (`"10.10"` no debe coincidir con `"10.1"`). |
| `@param` / `@return` | Ninguno / `void` |
| `@throws` | Ninguna declarada. |

---

### 2.27 Clases y métodos anónimos / internos

| Elemento | Presencia |
|---|---|
| Clases internas (`static` o de instancia) | **Ninguna** declarada. |
| Clases anónimas | **Ninguna** declarada. |
| Lambdas / referencias a método | **Ninguna** utilizada. |
| Métodos estáticos propios | **Ninguno**; los métodos `assertEquals`, `assertTrue`, `assertFalse`, `assertNull`, `assertNotNull` provienen de importaciones estáticas de `org.junit.jupiter.api.Assertions` (clase externa, no definida en este archivo). |
| Variables locales destacables | `macOsValues` (`boolean[]`) en `test_IS_OS()`, `cons` (`Constructor<?>[]`) en `testConstructor()`. |

---

## 3. Ejemplo de uso

El siguiente ejemplo muestra el uso de la parte reutilizable de la clase —el método público `getLastSupportedJavaVersion()` y el auxiliar `getIS_JAVA(int)` accedido desde una subclase del mismo paquete— junto con la ejecución estándar de la suite mediante JUnit 5.

```java
package org.apache.commons.lang3;

import org.junit.jupiter.api.Test;

/**
 * Caso de uso real: extensión de SystemUtilsTest para auditar, en tiempo de ejecución,
 * qué constantes IS_JAVA_* están soportadas por la versión de Commons Lang del classpath
 * y cuál de ellas corresponde a la JVM activa.
 */
class SystemUtilsVersionAuditTest extends SystemUtilsTest {

    @Test
    void auditarConstantesDeVersionSoportadas() throws Exception {
        // 1) Descubrimiento dinámico de la última versión soportada (API pública del test).
        final int ultimaSoportada = getLastSupportedJavaVersion();
        System.out.printf("Última versión soportada por SystemUtils: IS_JAVA_%d%n", ultimaSoportada);

        // 2) Versión reportada por la propiedad del sistema.
        System.out.println("java.version = " + SystemUtils.JAVA_VERSION);

        // 3) Reutilización de la suite base: valida IS_JAVA_* contra JAVA_VERSION.
        test_IS_JAVA();

        // 4) Comparaciones de rango sobre la JVM en ejecución.
        System.out.println("¿Al menos Java 17? " + SystemUtils.isJavaVersionAtLeast(JavaVersion.JAVA_17));
        System.out.println("¿Como máximo Java 21? " + SystemUtils.isJavaVersionAtMost(JavaVersion.JAVA_21));

        // 5) Coincidencia explícita de versión (algoritmo probado en testJavaVersionMatches()).
        System.out.println("¿Coincide con 21? " + SystemUtils.isJavaVersionMatch(SystemUtils.JAVA_VERSION, "21"));

        // 6) Inspección del entorno de sistema operativo (cubierto por test_IS_OS()).
        System.out.println("Windows: " + SystemUtils.IS_OS_WINDOWS + " | Unix: " + SystemUtils.IS_OS_UNIX);
        System.out.println("Host: " + SystemUtils.getHostName());
        System.out.println("Java home: " + SystemUtils.getJavaHomePath());
        System.out.println("Tmp dir: " + SystemUtils.getJavaIoTmpDirPath());
        System.out.println("Usuario: " + SystemUtils.getUserName("desconocido"));
        System.out.println("Variable PATH: " + SystemUtils.getEnvironmentVariable("PATH", "<vacío>"));
    }
}
```

Ejecución de la suite original desde la línea de comandos (Maven Surefire):

```bash
# Ejecuta todos los métodos @Test de SystemUtilsTest
mvn test -Dtest=SystemUtilsTest

# Ejecuta un método concreto
mvn test -Dtest='SystemUtilsTest#testOsVersionMatches'
```

---

## 4. Notas de arquitectura y observaciones técnicas

1. **Pruebas condicionales al entorno.** Prácticamente todas las pruebas de constantes (`test_IS_JAVA`, `test_IS_OS`, `test_IS_zOS`, `testIsJavaVersionAtLeast`, `testIsJavaVersionAtMost`) contienen ramas que no ejecutan ninguna aserción si la plataforma no es reconocida; en tales casos el test pasa sin verificar nada (en `test_IS_JAVA` y `test_IS_OS` se emite un mensaje por `System.out` / `System.err`). Esto es una decisión explícita de diseño acorde al comentario de clase «*Only limited testing can be performed*».
2. **Reflexión como mecanismo de extensibilidad.** `getLastSupportedJavaVersion()` y `getIS_JAVA(int)` desacoplan la suite del conjunto estático de constantes, de modo que `test_IS_JAVA()` sigue siendo válido al añadirse nuevos campos `IS_JAVA_N`. En contraste, `testIsJavaVersionAtLeast()` / `testIsJavaVersionAtMost()` siguen un enfoque enumerado manualmente, con el consiguiente coste de mantenimiento.
3. **Dependencia de `SecurityManager`.** Los métodos anotados con el comentario «*Assumes no security manager exists*» (`test_USER_NAME`, `testGetJavaHome`, `testGetJavaHomePath`, `testGetJavaIoTmpDir`, `testGetJavaIoTmpDirPath`, `testGetUserDir`, `testGetUserDirPath`, `testGetUserHome`, `testGetUserHomePath`, `testGetUserName`) asumen acceso libre a propiedades del sistema.
4. **Defectos menores detectados** (ver §2.21): aserción duplicada en la rama `IS_JAVA_9` y falta de la aserción `JAVA_27` en la rama `IS_JAVA_20` de `testIsJavaVersionAtMost()`. Asimismo, `test_IS_JAVA()` enumera constantes hasta `IS_JAVA_26`, mientras que `testIsJavaVersionAtLeast()`/`testIsJavaVersionAtMost()` ya referencian `IS_JAVA_27` y `JavaVersion.JAVA_27`.
5. **Dependencia de `testGetEnvironmentVariablePresent()`** respecto a la existencia de la variable `PATH`: la aserción es tautológica si `PATH` no existiese (ambos lados serían `null`), por lo que la prueba es robusta aunque menos informativa en entornos sin dicha variable.