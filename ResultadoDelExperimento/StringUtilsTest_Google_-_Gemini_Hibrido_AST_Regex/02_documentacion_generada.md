Claro, aquí tienes la documentación técnica exhaustiva para la clase `StringUtilsTest`, siguiendo la estructura y restricciones solicitadas.

***

# Documentación Técnica: `org.apache.commons.lang3.StringUtilsTest`

*   **@author**: Apache Software Foundation

## 1. Resumen ejecutivo

La clase `StringUtilsTest` es una suite de pruebas exhaustiva diseñada para validar la funcionalidad de la clase de utilidad `org.apache.commons.lang3.StringUtils`. Utilizando el framework de pruebas JUnit 5, verifica el comportamiento de un amplio espectro de métodos de manipulación de cadenas, cubriendo casos de uso comunes, casos límite (edge cases) y entradas nulas. Su objetivo principal es garantizar la robustez, fiabilidad y corrección de cada método expuesto por `StringUtils`.

## 2. Especificación de métodos y constructores

A continuación se detallan todos los constructores, métodos y bloques de inicialización presentes en la clase.

### Bloque de Inicialización Estático (`static {}`)

Este bloque se ejecuta una sola vez cuando la clase es cargada por la JVM. Su propósito es inicializar las constantes estáticas (`WHITESPACE`, `NON_WHITESPACE`, `HARD_SPACE`, `TRIMMABLE`, `NON_TRIMMABLE`) que se utilizan como datos de prueba en varios métodos de la suite.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |

### Constructor

#### `StringUtilsTest()`

Constructor público por defecto de la clase de prueba. Es invocado por el framework JUnit para crear una nueva instancia de la clase antes de ejecutar cada método de prueba.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | No aplica (es un constructor). |

### Métodos Privados

#### `innerTestSplit(final char separator, final String sepStr, final char noMatch)`

Método de ayuda privado que encapsula la lógica de prueba para las diferentes sobrecargas de `StringUtils.split()`. Realiza una serie de aserciones para validar que la división de cadenas funciona como se espera con diferentes separadores y límites.

| Atributo | Nombre | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- | :--- |
| **@param** | `separator` | `char` | Sí | El carácter separador a probar. |
| **@param** | `sepStr` | `String` | Sí | La representación en cadena del separador. |
| **@param** | `noMatch` | `char` | Sí | Un carácter que no coincide con el separador, usado para construir cadenas de prueba. |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `innerTestSplitPreserveAllTokens(final char separator, final String sepStr, final char noMatch)`

Método de ayuda privado similar a `innerTestSplit`, pero enfocado en probar la funcionalidad de `StringUtils.splitPreserveAllTokens()`. Valida que los tokens vacíos generados por separadores adyacentes se conserven en el resultado.

| Atributo | Nombre | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- | :--- |
| **@param** | `separator` | `char` | Sí | El carácter separador a probar. |
| **@param** | `sepStr` | `String` | Sí | La representación en cadena del separador. |
| **@param** | `noMatch` | `char` | Sí | Un carácter que no coincide con el separador, usado para construir cadenas de prueba. |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

### Métodos de Prueba (`@Test`)

A continuación se documentan todos los métodos de prueba unitaria.

<details>
<summary>Expandir para ver todos los métodos de prueba</summary>

#### `testAppendIfMissing()`
Verifica el comportamiento del método `StringUtils.appendIfMissing()`, asegurando que un sufijo se añada a una cadena solo si no termina con él.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testAppendIfMissingIgnoreCase()`
Verifica el comportamiento del método `StringUtils.appendIfMissingIgnoreCase()`, que añade un sufijo ignorando mayúsculas y minúsculas si la cadena no termina con él.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testCapitalize()`
Prueba el método `StringUtils.capitalize()`, que convierte el primer carácter de una cadena a mayúscula (Title Case).

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testCenter_StringInt()`
Valida el método `StringUtils.center(String, int)` para centrar una cadena dentro de un tamaño específico usando espacios.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testCenter_StringIntChar()`
Valida el método `StringUtils.center(String, int, char)` para centrar una cadena usando un carácter de relleno específico.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testCenter_StringIntString()`
Valida el método `StringUtils.center(String, int, String)` para centrar una cadena usando una cadena de relleno específica.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testChomp()`
Prueba el método `StringUtils.chomp()`, que elimina un salto de línea (`\n`, `\r` o `\r\n`) del final de una cadena.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testChop()`
Prueba el método `StringUtils.chop()`, que elimina el último carácter (o par de suplentes) de una cadena.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testConstructor()`
Verifica que la clase `StringUtils` tenga un constructor público y no sea final, permitiendo la instanciación.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testDefault_String()`
Prueba el método `StringUtils.defaultString(String)`, que devuelve una cadena vacía si la entrada es nula.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testDefault_StringString()`
Prueba el método `StringUtils.defaultString(String, String)`, que devuelve una cadena por defecto si la entrada es nula.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testDefaultIfBlank_CharBuffers()`
Prueba `StringUtils.defaultIfBlank()` con `CharBuffer`, devolviendo un valor por defecto si el buffer es nulo, vacío o solo contiene espacios en blanco.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testDefaultIfBlank_StringBuffers()`
Prueba `StringUtils.defaultIfBlank()` con `StringBuffer`, devolviendo un valor por defecto si el buffer es nulo, vacío o solo contiene espacios en blanco.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testDefaultIfBlank_StringBuilders()`
Prueba `StringUtils.defaultIfBlank()` con `StringBuilder`, devolviendo un valor por defecto si el builder es nulo, vacío o solo contiene espacios en blanco.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testDefaultIfBlank_StringString()`
Prueba `StringUtils.defaultIfBlank()` con `String`, devolviendo un valor por defecto si la cadena es nula, vacía o solo contiene espacios en blanco.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testDefaultIfEmpty_CharBuffers()`
Prueba `StringUtils.defaultIfEmpty()` con `CharBuffer`, devolviendo un valor por defecto si el buffer es nulo o vacío.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testDefaultIfEmpty_StringBuffers()`
Prueba `StringUtils.defaultIfEmpty()` con `StringBuffer`, devolviendo un valor por defecto si el buffer es nulo o vacío.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testDefaultIfEmpty_StringBuilders()`
Prueba `StringUtils.defaultIfEmpty()` con `StringBuilder`, devolviendo un valor por defecto si el builder es nulo o vacío.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testDefaultIfEmpty_StringString()`
Prueba `StringUtils.defaultIfEmpty()` con `String`, devolviendo un valor por defecto si la cadena es nula o vacía.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testDeleteWhitespace_String()`
Verifica que `StringUtils.deleteWhitespace()` elimine todos los caracteres de espacio en blanco de una cadena.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testDifference_StringString()`
Prueba el método `StringUtils.difference()`, que devuelve la parte de la segunda cadena que difiere de la primera.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testDifferenceAt_StringArray()`
Prueba el método `StringUtils.indexOfDifference(String[])`, que encuentra el índice en el que un array de cadenas comienza a diferir.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testDifferenceAt_StringString()`
Prueba el método `StringUtils.indexOfDifference(String, String)`, que encuentra el índice en el que dos cadenas comienzan a diferir.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testEMPTY()`
Verifica que la constante `StringUtils.EMPTY` sea una cadena vacía y no nula.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testEscapeSurrogatePairs()`
Prueba que los métodos de escape de `StringEscapeUtils` manejen correctamente los pares de suplentes (surrogate pairs).

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testEscapeSurrogatePairsLang858()`
Prueba específica para el bug LANG-858, asegurando que `escapeJava` y `escapeEcmaScript` manejen correctamente los pares de suplentes.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testGeorgianSample()`
Prueba el comportamiento de los métodos de comparación de cadenas que ignoran mayúsculas y minúsculas con caracteres específicos (georgianos y otros) que tienen mapeos de caso complejos.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testGetBytes_Charset()`
Prueba el método `StringUtils.getBytes(String, Charset)` para convertir una cadena a un array de bytes usando un `Charset` específico.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testGetBytes_String()`
Prueba el método `StringUtils.getBytes(String, String)` para convertir una cadena a un array de bytes usando el nombre de una codificación.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `UnsupportedEncodingException` - Se lanza si la codificación de caracteres no es soportada. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testGetCommonPrefix_StringArray()`
Prueba `StringUtils.getCommonPrefix()` para encontrar el prefijo común más largo entre un array de cadenas.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testGetDigits()`
Verifica que `StringUtils.getDigits()` extraiga correctamente todos los caracteres de dígitos de una cadena.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testGetDigitsKeycaps()`
Prueba `StringUtils.getDigits()` con emojis de "keycaps" para asegurar que los dígitos base se extraen correctamente.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testGetDigitsSupplementary()`
Prueba `StringUtils.getDigits()` con caracteres suplementarios que son dígitos para asegurar que se manejen correctamente.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testGetFuzzyDistance()`
Prueba `StringUtils.getFuzzyDistance()` para calcular la distancia "fuzzy" (basada en coincidencias de acrónimos y secuencias) entre dos cadenas.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testGetFuzzyDistance_NullNullNull()`
Verifica que `StringUtils.getFuzzyDistance()` lance `IllegalArgumentException` si todos sus argumentos son nulos.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si la excepción esperada no es lanzada. |

#### `testGetFuzzyDistance_NullStringLocale()`
Verifica que `StringUtils.getFuzzyDistance()` lance `IllegalArgumentException` si la primera cadena es nula.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si la excepción esperada no es lanzada. |

#### `testGetFuzzyDistance_StringNullLoclae()`
Verifica que `StringUtils.getFuzzyDistance()` lance `IllegalArgumentException` si la segunda cadena es nula.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si la excepción esperada no es lanzada. |

#### `testGetFuzzyDistance_StringStringNull()`
Verifica que `StringUtils.getFuzzyDistance()` lance `IllegalArgumentException` si el `Locale` es nulo.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si la excepción esperada no es lanzada. |

#### `testGetIfBlank_StringStringSupplier()`
Prueba `StringUtils.getIfBlank()` con un `Supplier`, que proporciona un valor por defecto solo si la cadena de entrada está en blanco.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testGetIfEmpty_StringStringSupplier()`
Prueba `StringUtils.getIfEmpty()` con un `Supplier`, que proporciona un valor por defecto solo si la cadena de entrada está vacía.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testGetJaroWinklerDistance_NullNull()`
Verifica que `StringUtils.getJaroWinklerDistance()` lance `IllegalArgumentException` si ambas cadenas son nulas.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si la excepción esperada no es lanzada. |

#### `testGetJaroWinklerDistance_NullString()`
Verifica que `StringUtils.getJaroWinklerDistance()` lance `IllegalArgumentException` si la primera cadena es nula.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si la excepción esperada no es lanzada. |

#### `testGetJaroWinklerDistance_StringNull()`
Verifica que `StringUtils.getJaroWinklerDistance()` lance `IllegalArgumentException` si la segunda cadena es nula.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si la excepción esperada no es lanzada. |

#### `testGetJaroWinklerDistance_StringString()`
Prueba el cálculo de la distancia de Jaro-Winkler entre dos cadenas, una medida de similitud.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testGetLevenshteinDistance_NullString()`
Verifica que `StringUtils.getLevenshteinDistance()` lance `IllegalArgumentException` si la primera cadena es nula.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si la excepción esperada no es lanzada. |

#### `testGetLevenshteinDistance_NullStringInt()`
Verifica que `StringUtils.getLevenshteinDistance()` con umbral lance `IllegalArgumentException` si la primera cadena es nula.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si la excepción esperada no es lanzada. |

#### `testGetLevenshteinDistance_StringNull()`
Verifica que `StringUtils.getLevenshteinDistance()` lance `IllegalArgumentException` si la segunda cadena es nula.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si la excepción esperada no es lanzada. |

#### `testGetLevenshteinDistance_StringNullInt()`
Verifica que `StringUtils.getLevenshteinDistance()` con umbral lance `IllegalArgumentException` si la segunda cadena es nula.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si la excepción esperada no es lanzada. |

#### `testGetLevenshteinDistance_StringString()`
Prueba el cálculo de la distancia de Levenshtein (número de ediciones) entre dos cadenas.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testGetLevenshteinDistance_StringStringInt()`
Prueba la versión optimizada de `getLevenshteinDistance` que acepta un umbral para detener el cálculo si la distancia lo excede.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testGetLevenshteinDistance_StringStringNegativeInt()`
Verifica que `getLevenshteinDistance` con umbral lance `IllegalArgumentException` si el umbral es negativo.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si la excepción esperada no es lanzada. |

#### `testIsAllLowerCase()`
Prueba `StringUtils.isAllLowerCase()` para verificar si una cadena contiene únicamente caracteres en minúsculas.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testIsAllLowerCaseSupplementary()`
Prueba `StringUtils.isAllLowerCase()` con caracteres suplementarios para asegurar el manejo correcto de puntos de código fuera del BMP.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testIsAllUpperCase()`
Prueba `StringUtils.isAllUpperCase()` para verificar si una cadena contiene únicamente caracteres en mayúsculas.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testIsAllUpperCaseSupplementary()`
Prueba `StringUtils.isAllUpperCase()` con caracteres suplementarios para asegurar el manejo correcto de puntos de código fuera del BMP.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testIsMixedCase()`
Prueba `StringUtils.isMixedCase()` para verificar si una cadena contiene tanto caracteres en mayúsculas como en minúsculas.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testIsMixedCaseSupplementary()`
Prueba `StringUtils.isMixedCase()` con caracteres suplementarios para asegurar el manejo correcto de puntos de código fuera del BMP.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testJoin_ArrayCharSeparator()`
Prueba `StringUtils.join()` para unir elementos de un array con un separador de tipo `char`.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testJoin_ArrayOfBooleans()`
Prueba `StringUtils.join()` para unir elementos de un array de `boolean` con un separador.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testJoin_ArrayOfBytes()`
Prueba `StringUtils.join()` para unir elementos de un array de `byte` con un separador.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testJoin_ArrayOfChars()`
Prueba `StringUtils.join()` para unir elementos de un array de `char` con un separador.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testJoin_ArrayOfDoubles()`
Prueba `StringUtils.join()` para unir elementos de un array de `double` con un separador.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testJoin_ArrayOfFloats()`
Prueba `StringUtils.join()` para unir elementos de un array de `float` con un separador.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testJoin_ArrayOfInts()`
Prueba `StringUtils.join()` para unir elementos de un array de `int` con un separador.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testJoin_ArrayOfLongs()`
Prueba `StringUtils.join()` para unir elementos de un array de `long` con un separador.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testJoin_ArrayOfShorts()`
Prueba `StringUtils.join()` para unir elementos de un array de `short` con un separador.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testJoin_ArrayString_EmptyDelimiter()`
Prueba `StringUtils.join()` con un delimitador nulo o vacío, lo que debe concatenar los elementos sin separador.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testJoin_ArrayString_NonEmptyDelimiter(final String delimiter)`
Prueba parametrizada que verifica `StringUtils.join()` con varios delimitadores no vacíos, incluyendo caracteres suplementarios.

| Atributo | Nombre | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- | :--- |
| **@param** | `delimiter` | `String` | Sí | El delimitador a usar en la unión. |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testJoin_IterableChar()`
Prueba `StringUtils.join()` para unir elementos de un `Iterable` con un separador de tipo `char`.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testJoin_IterableString()`
Prueba `StringUtils.join()` para unir elementos de un `Iterable` con un separador de tipo `String`.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testJoin_IteratorChar()`
Prueba `StringUtils.join()` para unir elementos de un `Iterator` con un separador de tipo `char`.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testJoin_IteratorString()`
Prueba `StringUtils.join()` para unir elementos de un `Iterator` con un separador de tipo `String`.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testJoin_List_CharDelimiter()`
Prueba `StringUtils.join()` para unir subsecciones de una `List` con un separador de tipo `char`.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testJoin_List_EmptyDelimiter()`
Prueba `StringUtils.join()` con una `List` y un delimitador nulo o vacío.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testJoin_List_NonEmptyDelimiter(final String delimiter)`
Prueba parametrizada que verifica `StringUtils.join()` con una `List` y varios delimitadores no vacíos.

| Atributo | Nombre | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- | :--- |
| **@param** | `delimiter` | `String` | Sí | El delimitador a usar en la unión. |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testJoin_Objectarray()`
Prueba `StringUtils.join(Object...)` para unir un array de objetos varargs sin separador.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testJoin_Objects()`
Prueba `StringUtils.join(Object...)` con varios objetos, incluyendo nulos y cadenas vacías.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testJoinWith(final String delimiter)`
Prueba parametrizada para `StringUtils.joinWith()`, que une elementos con un delimitador especificado como primer argumento.

| Atributo | Nombre | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- | :--- |
| **@param** | `delimiter` | `String` | Sí | El delimitador a usar en la unión. |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testJoinWithThrowsException()`
Verifica que `StringUtils.joinWith()` lance `IllegalArgumentException` si el array de objetos es nulo.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si la excepción esperada no es lanzada. |

#### `testLang1593()`
Prueba deshabilitada (`@Disabled`) relacionada con el bug LANG-1593.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |

#### `testLang623()`
Prueba específica para el bug LANG-623, verificando el comportamiento de `replaceChars` con caracteres thorn (Þ, þ).

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testLANG666()`
Prueba específica para el bug LANG-666, verificando que `stripEnd` funcione correctamente con cadenas que parecen números decimales.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testLeftPad_StringInt()`
Prueba `StringUtils.leftPad(String, int)` para rellenar una cadena a la izquierda con espacios.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testLeftPad_StringIntChar()`
Prueba `StringUtils.leftPad(String, int, char)` para rellenar una cadena a la izquierda con un carácter específico.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testLeftPad_StringIntString()`
Prueba `StringUtils.leftPad(String, int, String)` para rellenar una cadena a la izquierda con una cadena de relleno.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testLength_CharBuffer()`
Prueba `StringUtils.length()` con un `CharBuffer` para obtener su longitud.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testLengthString()`
Prueba `StringUtils.length()` con un `String` para obtener su longitud, manejando el caso nulo.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testLengthStringBuffer()`
Prueba `StringUtils.length()` con un `StringBuffer` para obtener su longitud.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testLengthStringBuilder()`
Prueba `StringUtils.length()` con un `StringBuilder` para obtener su longitud.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testLowerCase()`
Prueba `StringUtils.lowerCase()` para convertir una cadena a minúsculas, considerando diferentes `Locale`.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testNormalizeSpace()`
Prueba `StringUtils.normalizeSpace()` para reemplazar secuencias de espacios en blanco con un solo espacio y eliminar los espacios en blanco iniciales y finales.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testOverlay_StringStringIntInt()`
Prueba `StringUtils.overlay()` para superponer una cadena sobre otra en una posición específica.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testPrependIfMissing()`
Verifica el comportamiento de `StringUtils.prependIfMissing()`, asegurando que un prefijo se añada a una cadena solo si no comienza con él.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testPrependIfMissingIgnoreCase()`
Verifica el comportamiento de `StringUtils.prependIfMissingIgnoreCase()`, que añade un prefijo ignorando mayúsculas y minúsculas si la cadena no comienza con él.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testReCapitalize()`
Prueba la "reflexividad" de los métodos `capitalize` y `uncapitalize`, asegurando que `uncapitalize(capitalize(str))` devuelva el estado original.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testRemove_char()`
Prueba `StringUtils.remove(String, char)` para eliminar todas las ocurrencias de un carácter de una cadena.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testRemove_String()`
Prueba `StringUtils.remove(String, String)` para eliminar todas las ocurrencias de una subcadena de una cadena.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testRemoveAll_StringString()`
Prueba `StringUtils.removeAll()` para eliminar todas las subcadenas que coincidan con una expresión regular.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `PatternSyntaxException` - Si la expresión regular es inválida. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testRemoveEnd()`
Prueba `StringUtils.removeEnd()` para eliminar una subcadena del final de otra si está presente.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testRemoveEndIgnoreCase()`
Prueba `StringUtils.removeEndIgnoreCase()` para eliminar una subcadena del final de otra, ignorando mayúsculas y minúsculas.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testRemoveFirst_StringString()`
Prueba `StringUtils.removeFirst()` para eliminar la primera subcadena que coincida con una expresión regular.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `PatternSyntaxException` - Si la expresión regular es inválida. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testRemoveIgnoreCase_String()`
Prueba `StringUtils.removeIgnoreCase()` para eliminar todas las ocurrencias de una subcadena, ignorando mayúsculas y minúsculas.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testRemovePattern_StringString()`
Prueba `StringUtils.removePattern()`, que es un alias para `removeAll()`.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testRemoveStartChar()`
Prueba `StringUtils.removeStart()` para eliminar un carácter del inicio de una cadena si está presente.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testRemoveStartIgnoreCase()`
Prueba `StringUtils.removeStartIgnoreCase()` para eliminar una subcadena del inicio de otra, ignorando mayúsculas y minúsculas.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testRemoveStartString()`
Prueba `StringUtils.removeStart()` para eliminar una subcadena del inicio de otra si está presente.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testRepeat_CharInt()`
Prueba `StringUtils.repeat(char, int)` para repetir un carácter un número específico de veces.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testRepeat_StringInt()`
Prueba `StringUtils.repeat(String, int)` para repetir una cadena un número específico de veces.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `IllegalArgumentException` - Si la operación resulta en una cadena demasiado grande. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testRepeat_StringStringInt()`
Prueba `StringUtils.repeat(String, String, int)` para repetir una cadena un número de veces, con un separador entre cada repetición.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `IllegalArgumentException` - Si la operación resulta en una cadena demasiado grande. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testReplace_StringStringArrayStringArray()`
Prueba `StringUtils.replaceEach()` para realizar múltiples reemplazos en una cadena de forma simultánea.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `IllegalArgumentException` - Si los arrays de búsqueda y reemplazo no tienen la misma longitud. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testReplace_StringStringArrayStringArrayBoolean()`
Prueba `StringUtils.replaceEachRepeatedly()` para realizar reemplazos múltiples de forma repetida hasta que no se encuentren más coincidencias.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `IllegalStateException` - Si se detecta un ciclo infinito o se excede el límite de iteraciones. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testReplace_StringStringString()`
Prueba `StringUtils.replace(String, String, String)` para reemplazar todas las ocurrencias de una subcadena por otra.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testReplace_StringStringStringInt()`
Prueba `StringUtils.replace(String, String, String, int)` para reemplazar un número máximo de ocurrencias de una subcadena.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testReplaceAll_StringStringString()`
Prueba `StringUtils.replaceAll()` para reemplazar todas las subcadenas que coincidan con una expresión regular por una cadena de reemplazo.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `PatternSyntaxException` - Si la expresión regular es inválida. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testReplaceChars_StringCharChar()`
Prueba `StringUtils.replaceChars(String, char, char)` para reemplazar todas las ocurrencias de un carácter por otro.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testReplaceChars_StringStringString()`
Prueba `StringUtils.replaceChars(String, String, String)` para reemplazar un conjunto de caracteres por otro conjunto.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testReplaceFirst_StringStringString()`
Prueba `StringUtils.replaceFirst()` para reemplazar la primera subcadena que coincida con una expresión regular.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `PatternSyntaxException` - Si la expresión regular es inválida. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testReplaceIgnoreCase_StringStringString()`
Prueba `StringUtils.replaceIgnoreCase()` para reemplazar todas las ocurrencias de una subcadena por otra, ignorando mayúsculas y minúsculas.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testReplaceIgnoreCase_StringStringStringInt()`
Prueba `StringUtils.replaceIgnoreCase()` con un límite en el número de reemplazos, ignorando mayúsculas y minúsculas.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testReplaceOnce_StringStringString()`
Prueba `StringUtils.replaceOnce()` para reemplazar solo la primera ocurrencia de una subcadena.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testReplaceOnceIgnoreCase_StringStringString()`
Prueba `StringUtils.replaceOnceIgnoreCase()` para reemplazar solo la primera ocurrencia de una subcadena, ignorando mayúsculas y minúsculas.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testReplacePattern_StringStringString()`
Prueba `StringUtils.replacePattern()`, que es un alias para `replaceAll()`.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testReverse_String()`
Prueba `StringUtils.reverse()` para invertir el orden de los caracteres en una cadena.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testReverseDelimited_StringChar()`
Prueba `StringUtils.reverseDelimited()` para invertir el orden de las subcadenas delimitadas por un carácter.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testRightPad_StringInt()`
Prueba `StringUtils.rightPad(String, int)` para rellenar una cadena a la derecha con espacios.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testRightPad_StringIntChar()`
Prueba `StringUtils.rightPad(String, int, char)` para rellenar una cadena a la derecha con un carácter específico.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testRightPad_StringIntString()`
Prueba `StringUtils.rightPad(String, int, String)` para rellenar una cadena a la derecha con una cadena de relleno.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testRotate_StringInt()`
Prueba `StringUtils.rotate()` para rotar los caracteres de una cadena un número determinado de posiciones.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testSplit_String()`
Prueba `StringUtils.split(String)` para dividir una cadena por espacios en blanco, omitiendo tokens vacíos.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testSplit_StringChar()`
Prueba `StringUtils.split(String, char)` para dividir una cadena por un carácter separador, omitiendo tokens vacíos.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testSplit_StringString_StringStringInt()`
Prueba las sobrecargas de `StringUtils.split(String, String)` y `StringUtils.split(String, String, int)` para dividir una cadena por una subcadena separadora.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testSplitByCharacterType()`
Prueba `StringUtils.splitByCharacterType()` para dividir una cadena en función de los cambios en el tipo de carácter (mayúscula, minúscula, dígito, etc.).

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testSplitByCharacterTypeCamelCase()`
Prueba `StringUtils.splitByCharacterTypeCamelCase()` para dividir una cadena siguiendo las convenciones de CamelCase.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testSplitByWholeSeparatorPreserveAllTokens_StringString()`
Prueba `StringUtils.splitByWholeSeparatorPreserveAllTokens()` para dividir una cadena por una subcadena separadora, conservando todos los tokens.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testSplitByWholeSeparatorPreserveAllTokens_StringStringInt()`
Prueba `StringUtils.splitByWholeSeparatorPreserveAllTokens()` con un límite en el número de divisiones.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testSplitByWholeString_StringStringBoolean()`
Prueba `StringUtils.splitByWholeSeparator()` para dividir una cadena por una subcadena separadora, omitiendo tokens vacíos.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testSplitByWholeString_StringStringBooleanInt()`
Prueba `StringUtils.splitByWholeSeparator()` con un límite en el número de divisiones.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testSplitByWholeStringDropsTrailingEmpty(final String str, final String separator, final String expected)`
Prueba parametrizada que verifica que `splitByWholeSeparator` elimine correctamente los tokens vacíos finales.

| Atributo | Nombre | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- | :--- |
| **@param** | `str` | `String` | Sí | La cadena a dividir. |
| **@param** | `separator` | `String` | Sí | El separador a utilizar. |
| **@param** | `expected` | `String` | Sí | El resultado esperado como una cadena separada por comas. |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testSplitPreserveAllTokens_String()`
Prueba `StringUtils.splitPreserveAllTokens(String)` para dividir una cadena por espacios en blanco, conservando todos los tokens (incluidos los vacíos).

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testSplitPreserveAllTokens_StringChar()`
Prueba `StringUtils.splitPreserveAllTokens(String, char)` para dividir una cadena por un carácter separador, conservando todos los tokens.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testSplitPreserveAllTokens_StringString_StringStringInt()`
Prueba las sobrecargas de `StringUtils.splitPreserveAllTokens()` que usan una subcadena como separador y opcionalmente un límite.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testStringUtilsCharSequenceContract()`
Prueba de contrato que verifica que los métodos de `StringUtils` sigan una convención: los métodos "inmutables" (que calculan valores) deben aceptar `CharSequence`, mientras que los métodos "mutables" (que transforman la cadena) deben aceptar y devolver `String`.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testSwapCase_String()`
Prueba `StringUtils.swapCase()` para intercambiar mayúsculas por minúsculas y viceversa en una cadena.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testToCodePoints()`
Prueba `StringUtils.toCodePoints()` para convertir una cadena en un array de puntos de código Unicode, manejando correctamente los pares de suplentes.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testToCodePointsEmoji()`
Prueba `StringUtils.toCodePoints()` con cadenas que contienen emojis complejos para asegurar una conversión correcta a puntos de código.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testToEncodedString()`
Prueba `StringUtils.toEncodedString()` para convertir un array de bytes en una cadena usando un `Charset` específico.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testToRootLowerCase()`
Prueba `StringUtils.toRootLowerCase()` para convertir una cadena a minúsculas usando el `Locale.ROOT`, independientemente del `Locale` por defecto del sistema.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testToRootLowerCaseTurkish()`
Prueba `StringUtils.toRootLowerCase()` específicamente en un entorno con `Locale` turco para asegurar que no se vea afectado por las reglas de mayúsculas/minúsculas de ese idioma.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testToRootUpperCase()`
Prueba `StringUtils.toRootUpperCase()` para convertir una cadena a mayúsculas usando el `Locale.ROOT`.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testToString()`
Prueba `StringUtils.toString(byte[], String)` para convertir un array de bytes en una cadena usando el nombre de una codificación.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `UnsupportedEncodingException` - Si la codificación de caracteres no es soportada. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testTruncate_StringInt()`
Prueba `StringUtils.truncate(String, int)` para truncar una cadena a una longitud máxima.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `IllegalArgumentException` - Si el ancho máximo es negativo. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testTruncate_StringIntInt()`
Prueba `StringUtils.truncate(String, int, int)` para truncar una cadena a una longitud máxima a partir de un desplazamiento (offset).

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `IllegalArgumentException` - Si el desplazamiento o el ancho máximo son negativos. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testTruncate_StringIntInt_surrogatePair()`
Prueba el comportamiento de `StringUtils.truncate()` con cadenas que contienen pares de suplentes para asegurar que no se generen cadenas mal formadas.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testUnCapitalize()`
Prueba `StringUtils.uncapitalize()` para convertir el primer carácter de una cadena a minúscula.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testUnescapeSurrogatePairs()`
Prueba que los métodos de `StringEscapeUtils` para "des-escapar" cadenas manejen correctamente los pares de suplentes.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testUnwrap_StringChar()`
Prueba `StringUtils.unwrap(String, char)` para eliminar un carácter específico del principio y del final de una cadena.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testUnwrap_StringString()`
Prueba `StringUtils.unwrap(String, String)` para eliminar una subcadena específica del principio y del final de una cadena.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testUpperCase()`
Prueba `StringUtils.upperCase()` para convertir una cadena a mayúsculas, considerando diferentes `Locale`.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testWrap_StringChar()`
Prueba `StringUtils.wrap(String, char)` para envolver una cadena con un carácter específico al principio y al final.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testWrap_StringString()`
Prueba `StringUtils.wrap(String, String)` para envolver una cadena con una subcadena específica al principio y al final.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testWrapIfMissing_StringChar()`
Prueba `StringUtils.wrapIfMissing(String, char)` para envolver una cadena con un carácter solo si no está ya envuelta.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

#### `testWrapIfMissing_StringString()`
Prueba `StringUtils.wrapIfMissing(String, String)` para envolver una cadena con una subcadena solo si no está ya envuelta.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `AssertionError` - Se lanza si alguna de las aserciones de la prueba falla. |

</details>

### Clases y Métodos Internos

#### Clase Anónima en `NULL_TO_STRING_LIST`

Se define una clase anónima interna dentro de la inicialización del array `NULL_TO_STRING_LIST`. Su único propósito es proporcionar un objeto cuyo método `toString()` retorne `null`, para ser usado en las pruebas de los métodos `join`.

##### `toString()`

Sobrescribe el método `toString()` de la clase `Object` para que siempre devuelva `null`.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `String` - Siempre retorna `null`. |

## 3. Ejemplo de Uso

Dado que `StringUtilsTest` es una clase de prueba, su uso directo es ejecutarla a través de un corredor de pruebas como parte de un proceso de compilación o integración continua. Sin embargo, un ejemplo más práctico es mostrar cómo se utilizan los métodos de `StringUtils` que esta clase valida.

El siguiente bloque de código demuestra el uso de algunos métodos de `StringUtils` en una aplicación Java real. La correcta ejecución de `StringUtilsTest` garantiza que estos métodos se comporten como se espera.

```java
import org.apache.commons.lang3.StringUtils;
import java.util.Arrays;
import java.util.List;

/**
 * Clase de ejemplo que demuestra el uso de métodos de StringUtils
 * cuya funcionalidad es validada por StringUtilsTest.
 */
public class StringUtilsExample {

    public static void main(String[] args) {
        // El propósito de StringUtilsTest es asegurar que los siguientes métodos
        // funcionen de manera fiable.

        // 1. Unir elementos de una lista (probado en testJoin_*)
        List<String> frameworks = Arrays.asList("Spring", "Hibernate", "JUnit");
        String joinedFrameworks = StringUtils.join(frameworks, ", ");
        System.out.println("Frameworks populares: " + joinedFrameworks);
        // Salida esperada: Frameworks populares: Spring, Hibernate, JUnit

        // 2. Poner en mayúscula la primera letra (probado en testCapitalize)
        String title = "guía de inicio rápido";
        String capitalizedTitle = StringUtils.capitalize(title);
        System.out.println("Título capitalizado: " + capitalizedTitle);
        // Salida esperada: Título capitalizado: Guía de inicio rápido

        // 3. Centrar un texto con un carácter de relleno (probado en testCenter_*)
        String header = StringUtils.center("Reporte Final", 40, "=");
        System.out.println(header);
        // Salida esperada: ==============Reporte Final===============

        // 4. Obtener un valor por defecto si una cadena está en blanco (probado en testDefaultIfBlank_*)
        String userInput = "   ";
        String username = StringUtils.defaultIfBlank(userInput, "invitado");
        System.out.println("Nombre de usuario: " + username);
        // Salida esperada: Nombre de usuario: invitado
    }
}
```