# Documentación Técnica — `org.apache.commons.lang3.StringUtilsTest`

---

## 1. Resumen ejecutivo

`StringUtilsTest` es la clase de pruebas unitarias (JUnit 5 / Jupiter) que valida el contrato funcional de `org.apache.commons.lang3.StringUtils`, cubriendo familias de operaciones sobre cadenas: *padding*, *split*, *join*, *replace*, *remove*, *trim/chomp/chop*, conversión de mayúsculas/minúsculas, métricas de similitud (Levenshtein, Jaro‑Winkler, *fuzzy distance*) y manejo de pares subrogados (code points suplementarios).

Además de las aserciones funcionales, la clase incorpora pruebas estructurales por reflexión (`testConstructor()`, `testStringUtilsCharSequenceContract()`) que garantizan invariantes de diseño del API: visibilidad del constructor y la regla «métodos inmutables en espíritu reciben `CharSequence`; métodos mutables reciben `String`».

La clase extiende `AbstractLangTest`, declara constantes compartidas con otras clases de test del paquete (`WHITESPACE`, `NON_WHITESPACE`, `HARD_SPACE`, `TRIMMABLE`, `NON_TRIMMABLE`) y está anotada con `@SuppressWarnings("deprecation")` por el uso deliberado de API obsoleta (`getLevenshteinDistance`, `getJaroWinklerDistance`, `getFuzzyDistance`, `WordUtils`, `StringEscapeUtils`).

> **Nota sobre `@author`**: el código fuente provisto no declara la etiqueta `@author` (convención de Apache Commons: autoría gestionada vía `NOTICE`/SCM). No se infiere autoría.

---

## 2. Especificación de métodos y constructores

### 2.1 Metadatos estructurales de la clase

| Elemento | Modificadores | Descripción |
|---|---|---|
| `StringUtilsTest` | `class` (package‑private), `extends AbstractLangTest` | Clase de pruebas. Anotada `@SuppressWarnings("deprecation")`. |
| `StringUtilsTest()` | implícito, package‑private | **Constructor por defecto generado por el compilador.** No está declarado en el código fuente; JUnit 5 lo invoca por reflexión una vez por método de prueba (ciclo de vida `PER_METHOD`). No recibe parámetros, no retorna valor y no declara excepciones comprobadas. |
| Bloque `static { ... }` | `static` initializer | Inicializador estático (no es un método, pero es código ejecutable detectado). Ver §2.2. |

#### Constantes de instrumentación (visibilidad de paquete, reutilizadas por otros tests)

| Constante | Tipo | Contenido |
|---|---|---|
| `WHITESPACE` | `static final String` | Todos los caracteres `c < Character.MAX_VALUE` con `Character.isWhitespace(c) == true`. |
| `NON_WHITESPACE` | `static final String` | Caracteres con código `< 40` que **no** son whitespace. |
| `HARD_SPACE` | `static final String` | Carácter 160 (espacio duro / NBSP). |
| `TRIMMABLE` | `static final String` | Caracteres `0..32` inclusive (recortables por `String.trim()`). |
| `NON_TRIMMABLE` | `static final String` | Whitespace con código `> 32` (no recortable por `trim()`). |

#### Constantes privadas de datos de prueba (resumen)

`ARRAY_LIST`, `EMPTY_ARRAY_LIST`, `NULL_ARRAY_LIST`, `NULL_TO_STRING_LIST`, `MIXED_ARRAY_LIST`, `MIXED_TYPE_LIST`, `LONG_PRIM_LIST`, `INT_PRIM_LIST`, `BYTE_PRIM_LIST`, `SHORT_PRIM_LIST`, `CHAR_PRIM_LIST`, `FLOAT_PRIM_LIST`, `DOUBLE_PRIM_LIST`, `MIXED_STRING_LIST`, `MIXED_TYPE_OBJECT_LIST`, `STRING_LIST`, `EMPTY_STRING_LIST`, `NULL_STRING_LIST`, `SEPARATOR`, `SEPARATOR_CHAR`, `COMMA_SEPARATOR_CHAR`, `TEXT_LIST`, `TEXT_LIST_CHAR`, `TEXT_LIST_NOSEP`, `FOO_UNCAP`, `FOO_CAP`, `SENTENCE_UNCAP`, `SENTENCE_CAP`, `EMPTY`, `ARRAY_FALSE_FALSE`, `ARRAY_FALSE_TRUE`, `ARRAY_FALSE_TRUE_FALSE`.

---

### 2.2 Inicializador estático — `static { ... }`

**Propósito:** construir en tiempo de carga de clase las cadenas `WHITESPACE`, `NON_WHITESPACE`, `HARD_SPACE`, `TRIMMABLE` y `NON_TRIMMABLE` recorriendo el rango `[0, Character.MAX_VALUE)` y clasificando cada `char` mediante `Character.isWhitespace(char)`.

| Aspecto | Detalle |
|---|---|
| `@param` | No aplica (bloque de inicialización). |
| `@return` | No aplica. Efecto lateral: asignación de los 5 campos `static final`. |
| `@throws` | `ExceptionInInitializerError` únicamente si falla la inicialización (no esperado en operación normal). |

**Algoritmo:**
1. Para `i` en `[0, Character.MAX_VALUE)`: si `isWhitespace(i)` → se añade a `ws`; y si además `i > 32` → se añade a `ntr`. En caso contrario, si `i < 40` → se añade a `nws`.
2. Para `i` en `[0, 32]` → se añade a `tr`.

---

### 2.3 Clase anónima interna — `toString()` en `NULL_TO_STRING_LIST`

Declarada dentro del inicializador del campo `private static final Object[] NULL_TO_STRING_LIST`.

```java
new Object() {
    @Override
    public String toString() { return null; }
}
```

| Firma | `public String toString()` |
|---|---|
| **Tipo** | Método sobrescrito de una clase anónima que extiende `java.lang.Object`. |
| `@param` | *(ninguno)* |
| `@return` | `String` — **siempre `null`**. Simula un objeto cuyo `toString()` viola el contrato habitual. |
| `@throws` | *(ninguna)* |

**Uso:** permite verificar que `StringUtils.join(...)` y `StringUtils.joinWith(...)` producen el literal `"null"` cuando el elemento devuelve `null` desde `toString()` (consumido por `testJoin_ArrayCharSeparator()`, `testJoin_IteratorChar()`, `testJoin_IteratorString()`, `testJoin_Objectarray()` y `testJoinWith(String)`).

---

### 2.4 Lambdas y expresiones funcionales internas

| Ubicación | Firma funcional | `@param` | `@return` | `@throws` |
|---|---|---|---|---|
| `testGetIfBlank_StringStringSupplier()` → `countingDefaultSupplier` | `Supplier<String>` (`get()`) | *(ninguno)* | `"NULL"` | — |
| `testGetIfEmpty_StringStringSupplier()` → `countingDefaultSupplier` | `Supplier<String>` (`get()`) | *(ninguno)* | `"NULL"` | — |
| Lambdas `() -> "NULL"` (varias) | `Supplier<String>` | *(ninguno)* | `"NULL"` | — |
| Lambdas pasadas a `assertIllegalArgumentException(...)` / `assertThrows(...)` | `Executable` (`execute()`) | *(ninguno)* | `void` | Propaga la excepción esperada (`IllegalArgumentException`, `IllegalStateException`, `PatternSyntaxException`). |

> Los dos `countingDefaultSupplier` incrementan un `MutableInt` (`numberOfCalls`) como efecto lateral; se usan para verificar **evaluación perezosa** del proveedor de valor por defecto.

---

### 2.5 Métodos auxiliares privados

#### `innerTestSplit(char, String, char)`

```java
private void innerTestSplit(final char separator, final String sepStr, final char noMatch)
```

**Descripción:** rutina de verificación parametrizada que ejercita `StringUtils.split(String, String)` y `StringUtils.split(String, String, int)` sobre la cadena sintética `"a" + separator + "b" + separator + separator + noMatch + "c"`. Comprueba que los separadores adyacentes se colapsan y que el parámetro `max` recorta correctamente (`-1`, `0`, `1`, `2`).

| `@param` | Tipo | Requerido | Descripción |
|---|---|---|---|
| `separator` | `char` | Sí | Carácter separador que se inserta físicamente en la cadena de entrada. |
| `sepStr` | `String` | No (admite `null`) | Cadena de separadores entregada a `StringUtils.split`. `null` ⇒ semántica de whitespace. |
| `noMatch` | `char` | Sí | Carácter que **no** debe actuar como separador; valida que no se fragmente el token final. |

| `@return` | `void` — la verificación se realiza mediante aserciones. |
|---|---|
| `@throws` | `AssertionError` si cualquier aserción sobre longitud o contenido del array resultante falla. El mensaje incluye los valores hexadecimales de `separator` y `noMatch` y el `sepStr` utilizado. |

**Invariantes verificados:**
- `split(str, sepStr)` → `3` tokens: `"a"`, `"b"`, `noMatch + "c"`.
- `split(separator + "a" + separator, sepStr)` → `1` token: `"a"`.
- `max = -1` y `max = 0` → equivalentes a sin límite (3 tokens).
- `max = 1` → la cadena completa como único token.
- `max = 2` → `"a"` y `str.substring(2)`.

---

#### `innerTestSplitPreserveAllTokens(char, String, char)`

```java
private void innerTestSplitPreserveAllTokens(final char separator, final String sepStr, final char noMatch)
```

**Descripción:** homólogo de `innerTestSplit()` para `StringUtils.splitPreserveAllTokens(String, String)` y `splitPreserveAllTokens(String, String, int)`. Verifica que los separadores adyacentes y los extremos **sí** generan tokens vacíos.

| `@param` | Tipo | Requerido | Descripción |
|---|---|---|---|
| `separator` | `char` | Sí | Carácter separador insertado en la cadena de entrada. |
| `sepStr` | `String` | No (admite `null`) | Conjunto de separadores pasado al método bajo prueba; `null` ⇒ whitespace. |
| `noMatch` | `char` | Sí | Carácter no separador para validar la integridad del último token. |

| `@return` | `void` — verificación por aserciones. |
|---|---|
| `@throws` | `AssertionError` con mensaje diagnóstico (hex de `separator`/`noMatch` y `sepStr`) si alguna aserción falla. |

**Invariantes verificados:**
- `splitPreserveAllTokens(str, sepStr)` → `4` tokens: `"a"`, `"b"`, `""`, `noMatch + "c"`.
- `splitPreserveAllTokens(separator + "a" + separator, sepStr)` → `3` tokens: `""`, `"a"`, `""`.
- `max = -1` / `max = 0` → sin límite (4 tokens); `max = 1` → cadena completa; `max = 2` → `"a"` + resto.

---

### 2.6 Métodos de prueba — especificación completa

> **Contrato común a todos los métodos `@Test` sin parámetros:**
> `@param` → *(ninguno)*; `@return` → `void`; `@throws AssertionError` si falla una aserción de JUnit (`assertEquals`, `assertNull`, `assertTrue`, `assertArrayEquals`, `assertSame`, `assertNotEquals`, `assertFalse`, `assertNotNull`, `assertThrows`, `assertIllegalArgumentException`). Las excepciones comprobadas adicionales se indican explícitamente en la columna `@throws`.

#### Grupo A — Append / Prepend / Wrap / Unwrap

| Método | `@param` | `@return` | `@throws` | Propósito |
|---|---|---|---|---|
| `testAppendIfMissing()` | — | `void` | `AssertionError` | Valida `appendIfMissing(String, CharSequence)` y la sobrecarga con varargs `CharSequence...`: propagación de `null`, sufijo vacío, sufijos alternativos y sensibilidad a mayúsculas (`"aXYZ"` → `"aXYZxyz"`). |
| `testAppendIfMissingIgnoreCase()` | — | `void` | `AssertionError` | Valida `appendIfMissingIgnoreCase(...)`: `"abcXYZ"` no recibe sufijo; el varargs `null` y `""` se tratan como casos límite. |
| `testPrependIfMissing()` | — | `void` | `AssertionError` | Valida `prependIfMissing(...)` con y sin varargs; comportamiento sensible al caso (`"XYZabc"` → `"xyzXYZabc"`). |
| `testPrependIfMissingIgnoreCase()` | — | `void` | `AssertionError` | Valida `prependIfMissingIgnoreCase(...)`: `"XYZabc"` y `"MNOabc"` permanecen inalterados. |
| `testWrap_StringChar()` | — | `void` | `AssertionError` | `wrap(String, char)`: envuelve siempre, incluso si ya está envuelta (`"'abcd'"` → `"''abcd''"`). `null` → `null`. |
| `testWrap_StringString()` | — | `void` | `AssertionError` | `wrap(String, String)`: token `null`/`""` deja la cadena intacta. |
| `testWrapIfMissing_StringChar()` | — | `void` | `AssertionError` | `wrapIfMissing(String, char)`: idempotencia y verificación de **identidad de referencia** (`assertSame`) para `"/"` y `"/x/"`. |
| `testWrapIfMissing_StringString()` | — | `void` | `AssertionError` | `wrapIfMissing(String, String)`: idempotencia; `assertSame` para `"ab/ab"` y `"//x//"`. |
| `testUnwrap_StringChar()` | — | `void` | `AssertionError` | `unwrap(String, char)`: desenvuelve sólo un nivel; `"#A"`/`"A#"` sin cambios. |
| `testUnwrap_StringString()` | — | `void` | `AssertionError` | `unwrap(String, String)`: tokens multicarácter (`"AA"`, `"123"`), desenvoltura asimétrica no aplicada. |

#### Grupo B — Capitalización y cambio de caso

| Método | `@param` | `@return` | `@throws` | Propósito |
|---|---|---|---|---|
| `testCapitalize()` | — | `void` | `AssertionError` | `capitalize(String)`: `null`, vacío, un carácter y uso de **TitleCase** (`\u01C9` → `\u01C8`); incluye los ejemplos del Javadoc. |
| `testUnCapitalize()` | — | `void` | `AssertionError` | `uncapitalize(String)` incluyendo ejemplos del Javadoc (`"CAT"` → `"cAT"`). |
| `testReCapitalize()` | — | `void` | `AssertionError` | Prueba de reflexividad: `uncapitalize(capitalize(x)) == x` y `capitalize(uncapitalize(x)) == x` para frases y palabras simples. |
| `testLowerCase()` | — | `void` | `AssertionError` | `lowerCase(String)` y `lowerCase(String, Locale)`. |
| `testUpperCase()` | — | `void` | `AssertionError` | `upperCase(String)` y `upperCase(String, Locale)`. |
| `testSwapCase_String()` | — | `void` | `AssertionError` | `swapCase(String)` contrastado contra `WordUtils.swapCase(String)`, incluyendo carácter TitleCase `\u01C8`/`\u01C9`. |
| `testToRootLowerCase()` | — | `void` | `AssertionError` | `toRootLowerCase(String)`. Anotado `@ReadsDefaultLocale` + `@WritesDefaultLocale`; comprueba independencia del `Locale` turco. |
| `testToRootLowerCaseTurkish()` | — | `void` | `AssertionError` | Igual al anterior, ejecutado con `@DefaultLocale("tr")` + `@ReadsDefaultLocale`. |
| `testToRootUpperCase()` | — | `void` | `AssertionError` | `toRootUpperCase(String)`; fija temporalmente `Locale.setDefault(TURKISH)` dentro de `try/finally` y restaura el `Locale` original. |
| `testIsAllLowerCase()` | — | `void` | `AssertionError` | `isAllLowerCase(CharSequence)`: `null`, vacío, espacios, dígitos y símbolos ⇒ `false`. |
| `testIsAllLowerCaseSupplementary()` | — | `void` | `AssertionError` | `isAllLowerCase` con code points suplementarios `U+10428` (true) y `U+10400` (false). |
| `testIsAllUpperCase()` | — | `void` | `AssertionError` | `isAllUpperCase(CharSequence)`. |
| `testIsAllUpperCaseSupplementary()` | — | `void` | `AssertionError` | `isAllUpperCase` con `U+10400` (true) y `U+10428` (false). |
| `testIsMixedCase()` | — | `void` | `AssertionError` | `isMixedCase(CharSequence)`: requiere al menos una minúscula y una mayúscula. |
| `testIsMixedCaseSupplementary()` | — | `void` | `AssertionError` | `isMixedCase` combinando `'a'` con `U+10400`. |

#### Grupo C — Padding, centrado, repetición, rotación y truncado

| Método | `@param` | `@return` | `@throws` | Propósito |
|---|---|---|---|---|
| `testCenter_StringInt()` | — | `void` | `AssertionError` | `center(String, int)` con tamaños negativos, nulos y menores que la longitud. |
| `testCenter_StringIntChar()` | — | `void` | `AssertionError` | `center(String, int, char)` con relleno `'x'`. |
| `testCenter_StringIntString()` | — | `void` | `AssertionError` | `center(String, int, String)`; `null`/`""` ⇒ relleno con espacio; relleno multicarácter `"yz"`. |
| `testLeftPad_StringInt()` | — | `void` | `AssertionError` | `leftPad(String, int)` incluido `Integer.MIN_VALUE`. |
| `testLeftPad_StringIntChar()` | — | `void` | `AssertionError` | `leftPad(String, int, char)`; caso de 10 000 caracteres (supera el límite interno de *pad*) verificado con `containsOnly`. |
| `testLeftPad_StringIntString()` | — | `void` | `AssertionError` | `leftPad(String, int, String)` con relleno truncado (`"-+~"`). |
| `testRightPad_StringInt()` | — | `void` | `AssertionError` | `rightPad(String, int)`. |
| `testRightPad_StringIntChar()` | — | `void` | `AssertionError` | `rightPad(String, int, char)`; caso de 10 000 caracteres. |
| `testRightPad_StringIntString()` | — | `void` | `AssertionError` | `rightPad(String, int, String)`. |
| `testRepeat_CharInt()` | — | `void` | `AssertionError` | `repeat(char, int)` con conteo `0` y negativo. |
| `testRepeat_StringInt()` | — | `void` | `AssertionError` | `repeat(String, int)`; espera `IllegalArgumentException` para `repeat("aa", 1_073_741_824)` (desbordamiento de capacidad). |
| `testRepeat_StringStringInt()` | — | `void` | `AssertionError` | `repeat(String, String, int)` (repetición con separador); espera `IllegalArgumentException` en `1_073_741_824`. |
| `testRotate_StringInt()` | — | `void` | `AssertionError` | `rotate(String, int)` con desplazamientos positivos, negativos y mayores que la longitud. |
| `testTruncate_StringInt()` | — | `void` | `AssertionError` | `truncate(String, int)`; `maxWidth` negativo ⇒ `IllegalArgumentException`. |
| `testTruncate_StringIntInt()` | — | `void` | `AssertionError` | `truncate(String, int, int)`; `offset` y `maxWidth` negativos ⇒ `IllegalArgumentException`. |
| `testTruncate_StringIntInt_surrogatePair()` | — | `void` | `AssertionError` | Garantiza que `truncate` nunca emite un *lone surrogate* al cortar sobre `U+1F600`. |

#### Grupo D — Chomp, Chop, Trim y normalización

| Método | `@param` | `@return` | `@throws` | Propósito |
|---|---|---|---|---|
| `testChomp()` | — | `void` | `AssertionError` | `chomp(String)` mediante matriz de casos `String[][]`, y `chomp(String, String)`. |
| `testChop()` | — | `void` | `AssertionError` | `chop(String)` vía matriz de casos; verifica que un par subrogado final se elimina completo. |
| `testDeleteWhitespace_String()` | — | `void` | `AssertionError` | `deleteWhitespace(String)`; conserva `\u00A0` y `\u202F`. |
| `testNormalizeSpace()` | — | `void` | `AssertionError` | `normalizeSpace(String)` sobre controles `\u0009`–`\u001F`, NBSP y `\u0000`. |
| `testLANG666()` | — | `void` | `AssertionError` | Regresión LANG‑666: `stripEnd("120.00", ".0")` → `"12"`. |

#### Grupo E — Valores por defecto y longitud

| Método | `@param` | `@return` | `@throws` | Propósito |
|---|---|---|---|---|
| `testDefault_String()` | — | `void` | `AssertionError` | `defaultString(String)`. |
| `testDefault_StringString()` | — | `void` | `AssertionError` | `defaultString(String, String)`. |
| `testDefaultIfBlank_CharBuffers()` | — | `void` | `AssertionError` | `defaultIfBlank` con `CharBuffer`; verifica compatibilidad del tipo de retorno genérico. |
| `testDefaultIfBlank_StringBuffers()` | — | `void` | `AssertionError` | `defaultIfBlank` con `StringBuffer`. |
| `testDefaultIfBlank_StringBuilders()` | — | `void` | `AssertionError` | `defaultIfBlank` con `StringBuilder`. |
| `testDefaultIfBlank_StringString()` | — | `void` | `AssertionError` | `defaultIfBlank(String, String)`. |
| `testDefaultIfEmpty_CharBuffers()` | — | `void` | `AssertionError` | `defaultIfEmpty` con `CharBuffer`. |
| `testDefaultIfEmpty_StringBuffers()` | — | `void` | `AssertionError` | `defaultIfEmpty` con `StringBuffer`. |
| `testDefaultIfEmpty_StringBuilders()` | — | `void` | `AssertionError` | `defaultIfEmpty` con `StringBuilder`. |
| `testDefaultIfEmpty_StringString()` | — | `void` | `AssertionError` | `defaultIfEmpty(String, String)` y contraste con `getIfEmpty("", null)`. |
| `testGetIfBlank_StringStringSupplier()` | — | `void` | `AssertionError` | `getIfBlank(String, Supplier<String>)`; usa `MutableInt` + lambda contadora para probar **invocación perezosa** (0→1→2→3 llamadas). |
| `testGetIfEmpty_StringStringSupplier()` | — | `void` | `AssertionError` | `getIfEmpty(String, Supplier<String>)`; misma técnica (0→1→2). Usa `Suppliers.nul()`. |
| `testLength_CharBuffer()` | — | `void` | `AssertionError` | `length(CharSequence)` con `CharBuffer`. |
| `testLengthString()` | — | `void` | `AssertionError` | `length(CharSequence)` con `String` y `null`. |
| `testLengthStringBuffer()` | — | `void` | `AssertionError` | `length(CharSequence)` con `StringBuffer`. |
| `testLengthStringBuilder()` | — | `void` | `AssertionError` | `length(CharSequence)` con `StringBuilder`. |
| `testEMPTY()` | — | `void` | `AssertionError` | *Sanity check* de la constante `StringUtils.EMPTY` (no nula, `""`, longitud 0). |

#### Grupo F — Diferencias, prefijos y métricas de similitud

| Método | `@param` | `@return` | `@throws` | Propósito |
|---|---|---|---|---|
| `testDifference_StringString()` | — | `void` | `AssertionError` | `difference(String, String)`; la diferencia nunca inicia con un *lone low surrogate* (`U+10400` vs `U+10401`). |
| `testDifferenceAt_StringArray()` | — | `void` | `AssertionError` | `indexOfDifference(String...)`; el índice se reporta al inicio del par subrogado. |
| `testDifferenceAt_StringString()` | — | `void` | `AssertionError` | `indexOfDifference(CharSequence, CharSequence)`. |
| `testGetCommonPrefix_StringArray()` | — | `void` | `AssertionError` | `getCommonPrefix(String...)`; el prefijo común no puede ser un *lone high surrogate*. |
| `testGetFuzzyDistance()` | — | `void` | `AssertionError` | `getFuzzyDistance(CharSequence, CharSequence, Locale)` con `Locale.ENGLISH`. |
| `testGetFuzzyDistance_NullNullNull()` | — | `void` | `AssertionError` | Espera `IllegalArgumentException` con los tres argumentos `null`. |
| `testGetFuzzyDistance_NullStringLocale()` | — | `void` | `AssertionError` | Espera `IllegalArgumentException` con término `null`. |
| `testGetFuzzyDistance_StringNullLoclae()` | — | `void` | `AssertionError` | Espera `IllegalArgumentException` con query `null`. *(El nombre contiene la errata original «Loclae», se conserva tal cual.)* |
| `testGetFuzzyDistance_StringStringNull()` | — | `void` | `AssertionError` | Espera `IllegalArgumentException` con `Locale` `null`. |
| `testGetJaroWinklerDistance_NullNull()` | — | `void` | `AssertionError` | Espera `IllegalArgumentException`. |
| `testGetJaroWinklerDistance_NullString()` | — | `void` | `AssertionError` | Espera `IllegalArgumentException`. |
| `testGetJaroWinklerDistance_StringNull()` | — | `void` | `AssertionError` | Espera `IllegalArgumentException`. |
| `testGetJaroWinklerDistance_StringString()` | — | `void` | `AssertionError` | Valores de referencia (0.93, 0.0, 0.44, 0.84, 0.95, 0.92, 0.88, 0.63). |
| `testGetLevenshteinDistance_NullString()` | — | `void` | `AssertionError` | `getLevenshteinDistance("a", null)` ⇒ `IllegalArgumentException`. |
| `testGetLevenshteinDistance_StringNull()` | — | `void` | `AssertionError` | `getLevenshteinDistance(null, "a")` ⇒ `IllegalArgumentException`. |
| `testGetLevenshteinDistance_NullStringInt()` | — | `void` | `AssertionError` | Variante con umbral y primer argumento `null`. |
| `testGetLevenshteinDistance_StringNullInt()` | — | `void` | `AssertionError` | Variante con umbral y segundo argumento `null`. |
| `testGetLevenshteinDistance_StringString()` | — | `void` | `AssertionError` | Distancias canónicas (`frog/fog`=1, `elephant/hippo`=7, …). |
| `testGetLevenshteinDistance_StringStringInt()` | — | `void` | `AssertionError` | Variante con `threshold`: retorna `-1` cuando la distancia lo excede; cubre *stripe* fuera de rango y `Integer.MAX_VALUE`. |
| `testGetLevenshteinDistance_StringStringNegativeInt()` | — | `void` | `AssertionError` | Umbral `-1` ⇒ `IllegalArgumentException`. |
| `testGeorgianSample()` | — | `void` | `AssertionError` | Contrasta `startsWithIgnoreCase`, `endsWithIgnoreCase`, `compareIgnoreCase`, `indexOfIgnoreCase` y `lastIndexOfIgnoreCase` contra `String.equalsIgnoreCase` para caracteres turcos/griegos (`U+0131`, `U+03F4`, `U+0130`, `U+03D1`). |

#### Grupo G — `join` / `joinWith`

| Método | `@param` | `@return` | `@throws` | Propósito |
|---|---|---|---|---|
| `testJoin_ArrayCharSeparator()` | — | `void` | `AssertionError` | `join(Object[], char)` y `join(Object[], char, int, int)` incluyendo rango invertido (`2, 1` → `""`). |
| `testJoin_ArrayOfBooleans()` | — | `void` | `AssertionError` | `join(boolean[], char[, int, int])`. |
| `testJoin_ArrayOfBytes()` | — | `void` | `AssertionError` | `join(byte[], char[, int, int])`. |
| `testJoin_ArrayOfChars()` | — | `void` | `AssertionError` | `join(char[], char[, int, int])`. |
| `testJoin_ArrayOfDoubles()` | — | `void` | `AssertionError` | `join(double[], char[, int, int])`. |
| `testJoin_ArrayOfFloats()` | — | `void` | `AssertionError` | `join(float[], char[, int, int])`. |
| `testJoin_ArrayOfInts()` | — | `void` | `AssertionError` | `join(int[], char[, int, int])`. |
| `testJoin_ArrayOfLongs()` | — | `void` | `AssertionError` | `join(long[], char[, int, int])`. |
| `testJoin_ArrayOfShorts()` | — | `void` | `AssertionError` | `join(short[], char[, int, int])`. |
| `testJoin_ArrayString_EmptyDelimiter()` | — | `void` | `AssertionError` | `join(Object[], String)` con delimitador `null` o `""`. |
| `testJoin_IterableChar()` | — | `void` | `AssertionError` | `join(Iterable<?>, char)`. |
| `testJoin_IterableString()` | — | `void` | `AssertionError` | `join(Iterable<?>, String)`. |
| `testJoin_IteratorChar()` | — | `void` | `AssertionError` | `join(Iterator<?>, char)` incluido `NULL_TO_STRING_LIST`. |
| `testJoin_IteratorString()` | — | `void` | `AssertionError` | `join(Iterator<?>, String)`. |
| `testJoin_List_CharDelimiter()` | — | `void` | `AssertionError` | `join(List<?>, char, int, int)`. |
| `testJoin_List_EmptyDelimiter()` | — | `void` | `AssertionError` | `join(List<?>, String)` con delimitador vacío/`null`. |
| `testJoin_Objectarray()` | — | `void` | `AssertionError` | `join(Object...)` con *varargs* explícitos y cast a `(Object[]) null`. |
| `testJoin_Objects()` | — | `void` | `AssertionError` | `join(Object...)` — variante reducida. |
| `testJoinWithThrowsException()` | — | `void` | `AssertionError` | `joinWith(",", (Object[]) null)` ⇒ `IllegalArgumentException`. |
| `testLang1593()` | — | `void` | `AssertionError` | **`@Disabled`.** Regresión LANG‑1593: compara `join(int[], char)` con `join(int[], String)`. No se ejecuta en la suite. |

##### Métodos de prueba parametrizados (`@ParameterizedTest`)

**`testJoin_ArrayString_NonEmptyDelimiter(String delimiter)`**
`@ValueSource(strings = {",", ";", Supplementary.CharU20000, Supplementary.CharU20001})`

| `@param` | Tipo | Requerido | Descripción |
|---|---|---|---|
| `delimiter` | `String` | Sí (inyectado por JUnit) | Delimitador no vacío; incluye code points suplementarios. |

| `@return` | `void` |
|---|---|
| `@throws` | `AssertionError` si `StringUtils.join(Object[], String[, int, int])` difiere de `String.join`. |

**`testJoin_List_NonEmptyDelimiter(String delimiter)`** — mismo `@ValueSource`.

| `@param` | Tipo | Requerido | Descripción |
|---|---|---|---|
| `delimiter` | `String` | Sí | Delimitador no vacío inyectado. |

| `@return` | `void` |
|---|---|
| `@throws` | `AssertionError` al comparar `join(List<?>, String[, int, int])` con `String.join`. |

**`testJoinWith(String delimiter)`** — mismo `@ValueSource`.

| `@param` | Tipo | Requerido | Descripción |
|---|---|---|---|
| `delimiter` | `String` | Sí | Delimitador inyectado; también se prueba `joinWith(null, "a", "b")` → `"ab"`. |

| `@return` | `void` |
|---|---|
| `@throws` | `AssertionError` ante divergencias con `String.join`. |

#### Grupo H — `remove*`

| Método | `@param` | `@return` | `@throws` | Propósito |
|---|---|---|---|---|
| `testRemove_char()` | — | `void` | `AssertionError` | `remove(String, char)`. |
| `testRemove_String()` | — | `void` | `AssertionError` | `remove(String, String)`. |
| `testRemoveIgnoreCase_String()` | — | `void` | `AssertionError` | `removeIgnoreCase(String, String)` incluida regresión LANG‑1453 y `\u0130`. |
| `testRemoveAll_StringString()` | — | `void` | `AssertionError` | `removeAll(String, String)` (regex); espera `PatternSyntaxException` con `"{badRegexSyntax}"`. |
| `testRemoveFirst_StringString()` | — | `void` | `AssertionError` | `removeFirst(String, String)` (regex); espera `PatternSyntaxException`. |
| `testRemovePattern_StringString()` | — | `void` | `AssertionError` | `removePattern(String, String)` con flag `DOTALL` implícito. |
| `testRemoveEnd()` | — | `void` | `AssertionError` | `removeEnd(String, String)`. |
| `testRemoveEndIgnoreCase()` | — | `void` | `AssertionError` | `removeEndIgnoreCase(String, String)`. |
| `testRemoveStartString()` | — | `void` | `AssertionError` | `removeStart(String, String)`. |
| `testRemoveStartChar()` | — | `void` | `AssertionError` | `removeStart(String, char)` incluido `CharUtils.NUL` (`'\0'`). |
| `testRemoveStartIgnoreCase()` | — | `void` | `AssertionError` | `removeStartIgnoreCase(String, String)`. |

#### Grupo I — `replace*`

| Método | `@param` | `@return` | `@throws` | Propósito |
|---|---|---|---|---|
| `testReplace_StringStringString()` | — | `void` | `AssertionError` | `replace(String, String, String)`. |
| `testReplace_StringStringStringInt()` | — | `void` | `AssertionError` | `replace(String, String, String, int)`; verifica identidad de referencia con `assertSame` cuando no hay coincidencias. |
| `testReplace_StringStringArrayStringArray()` | — | `void` | `AssertionError` | `replaceEach(String, String[], String[])`; incluye LANG‑552 (`null` dentro de arrays) y `IllegalArgumentException` por arrays de longitud desigual. |
| `testReplace_StringStringArrayStringArrayBoolean()` | — | `void` | `AssertionError` | `replaceEachRepeatedly(...)`; cubre LANG‑1528/LANG‑1753, el límite TTL por defecto (32→16→8→4→2→1) y detección de referencias circulares mediante `IllegalStateException`. |
| `testReplaceAll_StringStringString()` | — | `void` | `AssertionError` | `replaceAll(String, String, String)` (regex) con grupos `$2`; espera `PatternSyntaxException`. |
| `testReplaceFirst_StringStringString()` | — | `void` | `AssertionError` | `replaceFirst(String, String, String)`; espera `PatternSyntaxException`. |
| `testReplacePattern_StringStringString()` | — | `void` | `AssertionError` | `replacePattern(String, String, String)` con semántica `DOTALL`. |
| `testReplaceChars_StringCharChar()` | — | `void` | `AssertionError` | `replaceChars(String, char, char)`. |
| `testReplaceChars_StringStringString()` | — | `void` | `AssertionError` | `replaceChars(String, String, String)`; `assertSame` cuando no hay cambios; caso Bugzilla #25454. |
| `testReplaceIgnoreCase_StringStringString()` | — | `void` | `AssertionError` | `replaceIgnoreCase(String, String, String)`. |
| `testReplaceIgnoreCase_StringStringStringInt()` | — | `void` | `AssertionError` | `replaceIgnoreCase(String, String, String, int)` con `max` negativo, `0` y > longitud. |
| `testReplaceOnce_StringStringString()` | — | `void` | `AssertionError` | `replaceOnce(String, String, String)`. |
| `testReplaceOnceIgnoreCase_StringStringString()` | — | `void` | `AssertionError` | `replaceOnceIgnoreCase(String, String, String)`. |
| `testLang623()` | — | `void` | `AssertionError` | Regresión LANG‑623: `replaceChars` con `\u00DE` y `\u00FE`. |
| `testOverlay_StringStringIntInt()` | — | `void` | `AssertionError` | `overlay(String, String, int, int)` con índices negativos, invertidos y fuera de rango. |

#### Grupo J — `split*`

| Método | `@param` | `@return` | `@throws` | Propósito |
|---|---|---|---|---|
| `testSplit_String()` | — | `void` | `AssertionError` | `split(String)` (whitespace). |
| `testSplit_StringChar()` | — | `void` | `AssertionError` | `split(String, char)`. |
| `testSplit_StringString_StringStringInt()` | — | `void` | `AssertionError` | `split(String, String)` y `split(String, String, int)`; delega en `innerTestSplit()` iterando el producto cartesiano `WHITESPACE × NON_WHITESPACE`. |
| `testSplitPreserveAllTokens_String()` | — | `void` | `AssertionError` | `splitPreserveAllTokens(String)`. |
| `testSplitPreserveAllTokens_StringChar()` | — | `void` | `AssertionError` | `splitPreserveAllTokens(String, char)` incluido el ejemplo del Javadoc `"a..b.c"`. |
| `testSplitPreserveAllTokens_StringString_StringStringInt()` | — | `void` | `AssertionError` | Variantes con `sepStr` y `max`; delega en `innerTestSplitPreserveAllTokens()`. |
| `testSplitByCharacterType()` | — | `void` | `AssertionError` | `splitByCharacterType(String)`; clasifica code points suplementarios (`U+1D400`, `U+1D7D3`) por su tipo propio. |
| `testSplitByCharacterTypeCamelCase()` | — | `void` | `AssertionError` | `splitByCharacterTypeCamelCase(String)`; una mayúscula suplementaria seguida de minúsculas se agrupa igual que una BMP. |
| `testSplitByWholeString_StringStringBoolean()` | — | `void` | `AssertionError` | `splitByWholeSeparator(String, String)`. |
| `testSplitByWholeString_StringStringBooleanInt()` | — | `void` | `AssertionError` | `splitByWholeSeparator(String, String, int)`. |
| `testSplitByWholeSeparatorPreserveAllTokens_StringString()` | — | `void` | `AssertionError` | `splitByWholeSeparatorPreserveAllTokens(String, String)`. |
| `testSplitByWholeSeparatorPreserveAllTokens_StringStringInt()` | — | `void` | `AssertionError` | Variante con `max`. |

**`testSplitByWholeStringDropsTrailingEmpty(String str, String separator, String expected)`**
`@ParameterizedTest` + `@CsvSource({"a:b:, :, 'a,b'", "a:, :, a", "ab-!-cd-!-, -!-, 'ab,cd'"})`

| `@param` | Tipo | Requerido | Descripción |
|---|---|---|---|
| `str` | `String` | Sí | Cadena a dividir, con separador final. |
| `separator` | `String` | Sí | Separador completo (uno o varios caracteres). |
| `expected` | `String` | Sí | Tokens esperados codificados como CSV (se dividen internamente con `expected.split(",")`). |

| `@return` | `void` |
|---|---|
| `@throws` | `AssertionError` si `splitByWholeSeparator` no descarta el token vacío final, o si `splitByWholeSeparatorPreserveAllTokens` no lo conserva (comparación con `ArrayUtils.add(expectedTokens, "")`). |

#### Grupo K — Conversiones, bytes y code points

| Método | `@param` | `@return` | `@throws` | Propósito |
|---|---|---|---|---|
| `testGetBytes_Charset()` | — | `void` | `AssertionError` | `getBytes(String, Charset)`; `null` ⇒ `ArrayUtils.EMPTY_BYTE_ARRAY`. |
| `testGetBytes_String()` | — | `void` | **`UnsupportedEncodingException`** (declarada; lanzada por el método bajo prueba), `AssertionError` | `getBytes(String, String)` con nombre de charset. |
| `testToEncodedString()` | — | `void` | `AssertionError` | `toEncodedString(byte[], Charset)` con `Charset.defaultCharset()`, `SystemUtils.FILE_ENCODING` y `UTF-16`. |
| `testToString()` | — | `void` | **`UnsupportedEncodingException`** (declarada), `AssertionError` | `toString(byte[], String)` con encoding `null`, por defecto y `UTF-16`. |
| `testToCodePoints()` | — | `void` | `AssertionError` | `toCodePoints(CharSequence)` con subrogados huérfanos y un code point suplementario (`U+2070E`). |
| `testToCodePointsEmoji()` | — | `void` | `AssertionError` | `toCodePoints` sobre secuencias emoji con ZWJ (`U+200D`) y modificadores de tono de piel; usa `ArrayFill.fill`. |
| `testGetDigits()` | — | `void` | `AssertionError` | `getDigits(String)` incluidos dígitos devanagari. |
| `testGetDigitsKeycaps()` | — | `void` | `AssertionError` | `getDigits` sobre secuencias *keycap* (`0️⃣`…`9️⃣`, `#️⃣`). |
| `testGetDigitsSupplementary()` | — | `void` | `AssertionError` | `getDigits` con `U+1D7CF` (dígito suplementario, par subrogado). |
| `testReverse_String()` | — | `void` | `AssertionError` | `reverse(String)`. |
| `testReverseDelimited_StringChar()` | — | `void` | `AssertionError` | `reverseDelimited(String, char)`. |

#### Grupo L — Pruebas estructurales y de escape

| Método | `@param` | `@return` | `@throws` | Propósito |
|---|---|---|---|---|
| `testConstructor()` | — | `void` | `AssertionError` | Verifica por reflexión (`getDeclaredConstructors`, `Modifier`) que `StringUtils` tiene **un único constructor público**, que la clase es `public` y **no** `final`. |
| `testStringUtilsCharSequenceContract()` | — | `void` | `AssertionError` | Recorre `StringUtils.class.getMethods()` y aplica la regla de diseño: si el retorno es `String`/`String[]` (mutable en espíritu), el primer parámetro no debe ser `CharSequence`/`CharSequence[]`; en caso contrario no debe ser `String`/`String[]`. Mantiene una lista blanca `excludeMethods` (6 firmas: `compare`, `compareIgnoreCase`, `getBytes`). |
| `testEscapeSurrogatePairs()` | — | `void` | `AssertionError` | `StringEscapeUtils.escapeCsv/escapeHtml3/escapeHtml4/escapeXml` preservan pares subrogados. |
| `testEscapeSurrogatePairsLang858()` | — | `void` | `AssertionError` | Regresión LANG‑858: `escapeJava` y `escapeEcmaScript` sobre `\uDBFF\uDFFD`. |
| `testUnescapeSurrogatePairs()` | — | `void` | `AssertionError` | `unescapeCsv/unescapeHtml3/unescapeHtml4` preservan pares subrogados. |

---

## 3. Ejemplo de uso

### 3.1 Ejecución de la suite

```bash
# Ejecuta únicamente esta clase de pruebas
mvn -Dtest=StringUtilsTest test

# Ejecuta un único método (incluida su variante parametrizada)
mvn -Dtest='StringUtilsTest#testJoinWith*' test
```

### 3.2 Reutilización del patrón en una clase de prueba propia

El siguiente ejemplo muestra un caso de uso real: una clase de test que **reutiliza las constantes de visibilidad de paquete** publicadas por `StringUtilsTest` y replica sus patrones (helper privado parametrizado, `@ParameterizedTest`, verificación de excepciones y de pereza del `Supplier`).

```java
package org.apache.commons.lang3;

import static org.junit.jupiter.api.Assertions.assertArrayEquals;
import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertThrows;

import java.util.function.Supplier;

import org.apache.commons.lang3.mutable.MutableInt;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.ValueSource;

class StringUtilsUsageExampleTest extends AbstractLangTest {

    /** Helper al estilo de innerTestSplit(char, String, char) de StringUtilsTest. */
    private void assertSplitsInto(final String input, final String sepStr, final String... expected) {
        final String msg = "input(" + input + "), sepStr(" + sepStr + ")";
        assertArrayEquals(expected, StringUtils.split(input, sepStr), msg);
    }

    @Test
    void testSplitUsesPackageConstants() {
        // WHITESPACE y NON_WHITESPACE provienen del inicializador estático de StringUtilsTest.
        final String str = "a" + StringUtilsTest.WHITESPACE + "b" + StringUtilsTest.NON_WHITESPACE + "c";
        final String[] tokens = StringUtils.split(str);
        assertEquals(2, tokens.length);
        assertEquals("a", tokens[0]);
        assertEquals("b" + StringUtilsTest.NON_WHITESPACE + "c", tokens[1]);

        assertSplitsInto("a.b..c", ".", "a", "b", "c");
    }

    @ParameterizedTest
    @ValueSource(strings = {",", ";", "-!-"})
    void testJoinWithSeveralDelimiters(final String delimiter) {
        assertEquals(String.join(delimiter, "a", "b", "c"),
                     StringUtils.joinWith(delimiter, "a", "b", "c"));
    }

    @Test
    void testSupplierIsEvaluatedLazily() {
        final MutableInt calls = new MutableInt(0);
        final Supplier<String> defaultSupplier = () -> {
            calls.increment();
            return "DEFAULT";
        };

        assertEquals("abc", StringUtils.getIfBlank("abc", defaultSupplier));
        assertEquals(0, calls.get(), "el proveedor no debe invocarse si la entrada no está en blanco");

        assertEquals("DEFAULT", StringUtils.getIfBlank("   ", defaultSupplier));
        assertEquals(1, calls.get());
    }

    @Test
    void testTruncateRejectsNegativeWidth() {
        assertThrows(IllegalArgumentException.class,
                     () -> StringUtils.truncate("abcdefghij", -1),
                     "maxWidth cannot be negative");
    }
}
```

---

## 4. Notas de mantenimiento

- **Dependencias internas del proyecto utilizadas:** `StringUtils`, `StringEscapeUtils`, `ArrayUtils`, `ArrayFill`, `CharUtils`, `SystemUtils`, `Supplementary`, `AbstractLangTest`, `LangAssertions.assertIllegalArgumentException`, `org.apache.commons.lang3.function.Suppliers`, `org.apache.commons.lang3.mutable.MutableInt`, `org.apache.commons.lang3.text.WordUtils`.
- **Dependencias de test:** JUnit Jupiter (`@Test`, `@Disabled`, `@ParameterizedTest`, `@ValueSource`, `@CsvSource`) y JUnit Pioneer (`@DefaultLocale`, `@ReadsDefaultLocale`, `@WritesDefaultLocale`).
- `testLang1593()` está desactivado (`@Disabled`); cualquier corrección de LANG‑1593 debe reactivarlo.
- Los métodos `testToRootLowerCase()`, `testToRootUpperCase()` y `testToRootLowerCaseTurkish()` manipulan el `Locale` por defecto; las anotaciones de JUnit Pioneer son obligatorias para preservar el aislamiento en ejecución paralela.
- `testStringUtilsCharSequenceContract()` fallará automáticamente al añadir a `StringUtils` un método que rompa la convención `CharSequence`/`String`; ampliar `excludeMethods` sólo con justificación documentada.