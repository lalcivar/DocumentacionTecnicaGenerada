# Documentación Técnica - Clase `StringUtilsTest`

### 1. Información General

| Atributo | Valor |
| :--- | :--- |
| **Nombre de la Clase** | `StringUtilsTest` |
| **Paquete** | `org.apache.commons.lang3` |
| **Tipo** | Clase de Pruebas Unitarias - JUnit 5 (Jupiter) |
| **Herencia** | `extends AbstractLangTest` |
| **Clase Probada** | `org.apache.commons.lang3.StringUtils` |
| **Licencia** | Apache License, Version 2.0 |
| **Framework** | JUnit 5, JUnit Pioneer, Apache Commons Lang 3 |
| **Visibilidad** | package-private `class` |

> **Nota Importante:** El código proporcionado **no corresponde a la clase productiva `StringUtils`**, sino a su **clase de pruebas asociada `StringUtilsTest`**. Su único propósito es validar, mediante pruebas unitarias, el correcto funcionamiento de todos los métodos de `StringUtils`.

### 2. Descripción General

`StringUtilsTest` es una suite de pruebas exhaustiva que verifica el comportamiento de la clase de utilidades `StringUtils` de Apache Commons Lang. `StringUtils` es una clase utilitaria `final` que proporciona más de 100 métodos estáticos para la manipulación segura de `String` y `CharSequence` con manejo de `null`.

Esta clase de test garantiza que las operaciones sobre cadenas sean **null-safe**, manejen correctamente caracteres Unicode suplementarios (pares sustitutos / surrogate pairs), espacios en blanco y casos límite.

No contiene lógica de negocio y no debe ser utilizada en código de producción. Su ejecución se realiza únicamente durante la fase de `test` con Maven/Gradle.

### 3. Estructura Interna

#### 3.1 Constantes y Datos de Prueba
La clase define un conjunto de constantes estáticas para reutilizar en todas las pruebas:

*   `WHITESPACE`, `NON_WHITESPACE`, `HARD_SPACE`, `TRIMMABLE`, `NON_TRIMMABLE`: Cadenas generadas dinámicamente en bloque `static` que contienen todos los caracteres considerados espacios en blanco por `Character.isWhitespace()`.
*   `ARRAY_LIST`, `EMPTY_ARRAY_LIST`, `NULL_ARRAY_LIST`, `MIXED_ARRAY_LIST`: Arrays de `String` y `Object` para probar métodos `join` y `split`.
*   `SEPARATOR`, `SEPARATOR_CHAR`, `TEXT_LIST`: Valores de referencia para validar resultados de unión/separación.
*   `FOO_UNCAP`, `FOO_CAP`, `SENTENCE_UNCAP`, `SENTENCE_CAP`: Cadenas para pruebas de capitalización.

#### 3.2 Métodos Auxiliares Privados

*   `innerTestSplit(char separator, String sepStr, char noMatch)`: Método helper que centraliza la lógica para probar `StringUtils.split()` con diferentes separadores y límites.
*   `innerTestSplitPreserveAllTokens(...)`: Equivalente para `splitPreserveAllTokens()`, verificando que se preserven los tokens vacíos.

### 4. Métodos Principales - Categorías de Pruebas

La clase contiene más de 120 métodos anotados con `@Test` y `@ParameterizedTest`. Se agrupan por funcionalidad de `StringUtils`:

#### A. Validación y Propiedades de la Cadena
| Método de Prueba | Método Validado en `StringUtils` | Descripción |
| :--- | :--- | :--- |
| `testIsAllLowerCase` / `testIsAllUpperCase` / `testIsMixedCase` | `isAllLowerCase`, `isAllUpperCase`, `isMixedCase` | Verifica si la cadena contiene solo minúsculas, mayúsculas o mezcla. Incluye pruebas con code points suplementarios. |
| `testLengthString` / `testLengthStringBuffer` | `length(CharSequence)` | Valida el cálculo de longitud con manejo de `null` retornando 0. |
| `testEMPTY` | `StringUtils.EMPTY` | Verifica la constante `EMPTY = ""`. |
| `testConstructor` | `StringUtils()` | Asegura que el constructor sea público y la clase no sea final. |

#### B. Manipulación de Mayúsculas/Minúsculas y Capitalización
| Método de Prueba | Método Validado | Descripción |
| :--- | :--- | :--- |
| `testCapitalize` / `testUnCapitalize` / `testReCapitalize` | `capitalize()`, `uncapitalize()` | Convierte el primer carácter a TitleCase / minúscula. |
| `testSwapCase_String` | `swapCase()` | Invierte mayúsculas y minúsculas. |
| `testLowerCase` / `testUpperCase` | `lowerCase()`, `upperCase()` | Conversión con soporte para `Locale`. |
| `testToRootLowerCase` / `testToRootUpperCase` | `toRootLowerCase()`, `toRootUpperCase()` | Conversión usando `Locale.ROOT` para evitar problemas con locale turco. |

#### C. Relleno, Centrado y Truncado
| Método de Prueba | Método Validado | Descripción |
| :--- | :--- | :--- |
| `testCenter_StringInt` | `center(String, int)` | Centra la cadena rellenando con espacios. |
| `testLeftPad_StringInt` / `testRightPad_StringInt` | `leftPad()`, `rightPad()` | Rellena a la izquierda/derecha con un carácter o cadena. |
| `testTruncate_StringInt` | `truncate(String, int)` | Trunca la cadena a un ancho máximo, manejando correctamente pares sustitutos para no dejar surrogates huérfanos. |
| `testChop` / `testChomp` | `chop()`, `chomp()` | Elimina el último carácter o el separador de línea final `\r\n`. |

#### D. Limpieza y Normalización
| Método de Prueba | Método Validado | Descripción |
| :--- | :--- | :--- |
| `testDeleteWhitespace_String` | `deleteWhitespace()` | Elimina todos los caracteres de espacio en blanco. |
| `testNormalizeSpace` | `normalizeSpace()` | Elimina espacios al inicio/final y normaliza espacios múltiples a uno solo. |

#### E. Búsqueda, Comparación y Distancia
| Método de Prueba | Método Validado | Descripción |
| :--- | :--- | :--- |
| `testDifference_StringString` | `difference()`, `indexOfDifference()` | Obtiene la parte diferente entre dos cadenas y el índice donde difieren. |
| `testGetCommonPrefix_StringArray` | `getCommonPrefix()` | Obtiene el prefijo común de un array de cadenas. |
| `testGetLevenshteinDistance` / `testGetJaroWinklerDistance` / `testGetFuzzyDistance` | `getLevenshteinDistance()`, `getJaroWinklerDistance()`, `getFuzzyDistance()` | Calcula distancias de edición para algoritmos de búsqueda difusa. |

#### F. Unión - Join
| Método de Prueba | Método Validado | Descripción |
| :--- | :--- | :--- |
| `testJoin_ArrayCharSeparator` / `testJoin_IterableString` | `join(Object[], char)`, `join(Iterable, String)` | Une elementos de arrays, `Iterable` e `Iterator` con un separador. Sobrecargas para tipos primitivos `int[]`, `long[]`, `byte[]`, etc. |
| `testJoinWith` | `joinWith(String, Object...)` | Une usando un delimitador, tratando `null` como cadena vacía. |

#### G. División - Split
| Método de Prueba | Método Validado | Descripción |
| :--- | :--- | :--- |
| `testSplit_String` / `testSplit_StringChar` | `split(String)`, `split(String, char)` | Divide por espacios en blanco o por un carácter separador, ignorando tokens vacíos. |
| `testSplitPreserveAllTokens_String` | `splitPreserveAllTokens()` | Variante que preserva tokens vacíos entre separadores consecutivos. |
| `testSplitByCharacterType` / `testSplitByCharacterTypeCamelCase` | `splitByCharacterType()` | Divide según el tipo de carácter Unicode - letra, dígito, etc. Útil para separar `camelCase`. |
| `testSplitByWholeSeparator` | `splitByWholeSeparator()` | Divide usando una cadena completa como separador, no un conjunto de caracteres. |

#### H. Reemplazo y Eliminación
| Método de Prueba | Método Validado | Descripción |
| :--- | :--- | :--- |
| `testReplace_StringStringString` / `testReplaceChars` | `replace()`, `replaceChars()`, `replaceEach()` | Reemplaza ocurrencias de subcadenas o conjuntos de caracteres. |
| `testRemove_String` / `testRemoveEnd` / `testRemoveStart` | `remove()`, `removeEnd()`, `removeStart()` | Elimina subcadenas, prefijos o sufijos. Incluye variantes `IgnoreCase`. |
| `testRemovePattern` / `testReplaceAll` | `removePattern()`, `replaceAll()` | Operaciones basadas en expresiones regulares. |

#### I. Otras Utilidades
| Método de Prueba | Método Validado | Descripción |
| :--- | :--- | :--- |
| `testReverse_String` / `testReverseDelimited` | `reverse()`, `reverseDelimited()` | Invierte la cadena completa o el orden de tokens delimitados. |
| `testRotate_StringInt` | `rotate()` | Rota los caracteres de la cadena N posiciones. |
| `testOverlay_StringStringIntInt` | `overlay()` | Sobrescribe una porción de la cadena con otra. |
| `testWrap_StringString` / `testUnwrap` | `wrap()`, `unwrap()`, `wrapIfMissing()` | Envuelve la cadena con un carácter o cadena delimitadora. |
| `testGetDigits` | `getDigits()` | Extrae solo los caracteres de dígito, incluyendo dígitos suplementarios. |
| `testToCodePoints` | `toCodePoints()` | Convierte la cadena a un array de code points Unicode. |

### 5. Dependencias y Anotaciones

*   **JUnit 5:** `@Test`, `@ParameterizedTest`, `@ValueSource`, `@CsvSource`, `@Disabled`
*   **JUnit Pioneer:** `@DefaultLocale`, `@ReadsDefaultLocale`, `@WritesDefaultLocale` para pruebas sensibles a `Locale`.
*   **Assertions:** `assertEquals`, `assertNull`, `assertThrows`, `assertArrayEquals` de `org.junit.jupiter.api.Assertions`.

### 6. Conclusión

`StringUtilsTest` no aporta funcionalidad al sistema, pero es crítica para la **calidad y regresión** del proyecto Apache Commons Lang. Actúa como documentación ejecutable y especificación del comportamiento esperado de `StringUtils`, asegurando robustez frente a valores `null`, cadenas vacías, caracteres Unicode complejos y casos límite.