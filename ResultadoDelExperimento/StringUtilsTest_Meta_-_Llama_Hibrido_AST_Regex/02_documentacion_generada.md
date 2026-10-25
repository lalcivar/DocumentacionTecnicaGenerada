# Documentación Técnica - `StringUtilsTest`

**Paquete:** `org.apache.commons.lang3`  
**Clase:** `StringUtilsTest extends AbstractLangTest`  
**Tipo:** Clase de pruebas unitarias JUnit 5 (Jupiter)  
**Dependencias directas:** `org.apache.commons.lang3.StringUtils`, `org.apache.commons.lang3.StringEscapeUtils`, `org.apache.commons.lang3.ArrayUtils`, `org.junit.jupiter.api`, `org.junitpioneer.jupiter`

---

## 1. Resumen Ejecutivo

`StringUtilsTest` es la suite de pruebas unitarias para la clase utilitaria `org.apache.commons.lang3.StringUtils` de Apache Commons Lang 3. Su propósito es validar de forma exhaustiva el comportamiento de los métodos de manipulación de `String` y `CharSequence` (trim, split, join, pad, case, distance, etc.), incluyendo casos límite, manejo de `null`, pares sustitutos (surrogate pairs) y compatibilidad de contrato `CharSequence` vs `String`.

La clase centraliza más de 150 casos de prueba organizados por funcionalidad, utiliza constantes estáticas pre-calculadas para espacios y separadores, y emplea helpers privados para parametrizar pruebas de `split`. No contiene lógica de negocio; su único rol es garantizar la corrección y regresión de `StringUtils`.

---

## 2. Especificación de Métodos y Constructores

### 2.1 Información de Clase

| Elemento | Valor |
|---|---|
| **Nombre completo** | `org.apache.commons.lang3.StringUtilsTest` |
| **Modificadores** | `class` (package-private), `@SuppressWarnings("deprecation")` |
| **Superclase** | `AbstractLangTest` |
| **Anotaciones de clase** | Ninguna directa; métodos anotados con `@Test`, `@ParameterizedTest`, `@Disabled`, `@DefaultLocale`, `@ReadsDefaultLocale`, `@WritesDefaultLocale` |
| **Constructor implícito** | `StringUtilsTest()` - constructor por defecto package-private generado por el compilador |

> **Bloque de inicialización estático `static { ... }`**
> Inicializa `WHITESPACE`, `NON_WHITESPACE`, `HARD_SPACE` (char 160), `TRIMMABLE` (0-32) y `NON_TRIMMABLE` iterando `Character.MAX_VALUE` y clasificando con `Character.isWhitespace()`. No recibe parámetros, no retorna valor, no lanza excepciones comprobadas.

### 2.2 Clase Anónima Interna

Definida en la constante `NULL_TO_STRING_LIST`:

```java
new Object() {
    @Override
    public String toString() { return null; }
}
```

| Método | Descripción Javadoc |
|---|---|
| `toString()` | Sobrescribe `Object.toString()`. **@return** `String` siempre `null`. **@throws** ninguno. Usado para probar `StringUtils.join` con objetos cuyo `toString()` retorna `null`. |

### 2.3 Constructores

#### `StringUtilsTest()`
Constructor por defecto implícito.

| @param | Tipo | Requerido | Descripción |
|---|---|---|---|
| - | - | - | No recibe parámetros |

| @return | Descripción |
|---|---|
| `StringUtilsTest` | Nueva instancia de la suite de pruebas |

| @throws | Condición |
|---|---|
| - | No lanza excepciones |

---

### 2.4 Métodos Privados de Utilidad

#### `innerTestSplit()`
```java
private void innerTestSplit(char separator, String sepStr, char noMatch)
```

| @param | Tipo | Requerido | Descripción |
|---|---|---|---|
| `separator` | `char` | Sí | Carácter separador usado para construir la cadena de prueba |
| `sepStr` | `String` | No | Representación String del separador (puede ser `null` para whitespace) |
| `noMatch` | `char` | Sí | Carácter que no debe coincidir como separador |

| @return | Descripción |
|---|---|
| `void` | No retorna valor. Ejecuta aserciones `assertEquals` sobre `StringUtils.split(String, String)` y `split(String, String, int)` |

| @throws | Condición |
|---|---|
| `AssertionFailedError` | Si alguna aserción de JUnit falla |

#### `innerTestSplitPreserveAllTokens()`
```java
private void innerTestSplitPreserveAllTokens(char separator, String sepStr, char noMatch)
```

| @param | Tipo | Requerido | Descripción |
|---|---|---|---|
| `separator` | `char` | Sí | Carácter separador |
| `sepStr` | `String` | No | String separador |
| `noMatch` | `char` | Sí | Carácter no separador |

| @return | Descripción |
|---|---|
| `void` | Valida `StringUtils.splitPreserveAllTokens` con y sin límite `max` |

| @throws | Condición |
|---|---|
| `AssertionFailedError` | Fallo de aserción |

---

### 2.5 Métodos de Prueba Públicos (JUnit)

> Todos los métodos siguientes son `package-private` (visibilidad por defecto en JUnit 5), anotados con `@Test` salvo indicación contraria, retornan `void` y no declaran parámetros salvo los parametrizados. Se documenta `@throws` solo cuando existe declaración explícita.

#### `testAppendIfMissing()`
Valida `StringUtils.appendIfMissing(String, CharSequence, CharSequence...)` con y sin sufijos alternativos.
| @return | `void` |
| @throws | `AssertionFailedError` |

#### `testAppendIfMissingIgnoreCase()`
Valida variante insensible a mayúsculas `appendIfMissingIgnoreCase`.
| @return | `void` |
| @throws | `AssertionFailedError` |

#### `testCapitalize()`
Valida `StringUtils.capitalize(String)` incluyendo `null`, vacío, TitleCase `\u01C9` -> `\u01C8`.
| @return | `void` |

#### `testCenter_StringInt()`
Valida `StringUtils.center(String, int)`.
| @return | `void` |

#### `testCenter_StringIntChar()`
Valida `StringUtils.center(String, int, char)`.
| @return | `void` |

#### `testCenter_StringIntString()`
Valida `StringUtils.center(String, int, String)` con pad `null` y vacío.
| @return | `void` |

#### `testChomp()`
Valida `StringUtils.chomp(String)` y `chomp(String, String)` con `\r`, `\n`, `\r\n`.
| @return | `void` |

#### `testChop()`
Valida `StringUtils.chop(String)` incluyendo par sustituto `\uD83D\uDE00`.
| @return | `void` |

#### `testConstructor()`
Verifica que `StringUtils` tiene constructor público único y no es `final`.
| @return | `void` |

#### `testDefault_String()`
Valida `StringUtils.defaultString(String)`.
| @return | `void` |

#### `testDefault_StringString()`
Valida `StringUtils.defaultString(String, String)`.
| @return | `void` |

#### `testDefaultIfBlank_CharBuffers()`
Valida `StringUtils.defaultIfBlank(CharBuffer, CharBuffer)`.
| @return | `void` |

#### `testDefaultIfBlank_StringBuffers()`
Valida `StringUtils.defaultIfBlank(StringBuffer, StringBuffer)`.
| @return | `void` |

#### `testDefaultIfBlank_StringBuilders()`
Valida `StringUtils.defaultIfBlank(StringBuilder, StringBuilder)`.
| @return | `void` |

#### `testDefaultIfBlank_StringString()`
Valida `StringUtils.defaultIfBlank(String, String)`.
| @return | `void` |

#### `testDefaultIfEmpty_CharBuffers()`
Valida `StringUtils.defaultIfEmpty(CharBuffer, CharBuffer)`.
| @return | `void` |

#### `testDefaultIfEmpty_StringBuffers()`
Valida `StringUtils.defaultIfEmpty(StringBuffer, StringBuffer)`.
| @return | `void` |

#### `testDefaultIfEmpty_StringBuilders()`
Valida `StringUtils.defaultIfEmpty(StringBuilder, StringBuilder)`.
| @return | `void` |

#### `testDefaultIfEmpty_StringString()`
Valida `StringUtils.defaultIfEmpty(String, String)` y `getIfEmpty`.
| @return | `void` |

#### `testDeleteWhitespace_String()`
Valida `StringUtils.deleteWhitespace(String)` con `WHITESPACE` y `\u00A0`.
| @return | `void` |

#### `testDifference_StringString()`
Valida `StringUtils.difference(String, String)` con pares sustitutos `0x10400`.
| @return | `void` |

#### `testDifferenceAt_StringArray()`
Valida `StringUtils.indexOfDifference(String...)` (varargs).
| @return | `void` |

#### `testDifferenceAt_StringString()`
Valida `StringUtils.indexOfDifference(CharSequence, CharSequence)`.
| @return | `void` |

#### `testEMPTY()`
Valida constante `StringUtils.EMPTY`.
| @return | `void` |

#### `testEscapeSurrogatePairs()`
Valida `StringEscapeUtils.escapeCsv/Html3/Html4/Xml` con pares sustitutos.
| @return | `void` |

#### `testEscapeSurrogatePairsLang858()`
Valida `escapeJava` y `escapeEcmaScript` con `\uDBFF\uDFFD` (LANG-858).
| @return | `void` |

#### `testGeorgianSample()`
Valida `startsWithIgnoreCase`, `endsWithIgnoreCase`, `compareIgnoreCase`, `indexOfIgnoreCase` con caracteres georgianos `\u0131`, `\u03F4`.
| @return | `void` |

#### `testGetBytes_Charset()`
Valida `StringUtils.getBytes(String, Charset)` con `null` y `StandardCharsets.US_ASCII`.
| @return | `void` |

#### `testGetBytes_String()`
```java
void testGetBytes_String() throws UnsupportedEncodingException
```
| @param | Tipo | Requerido | Descripción |
|---|---|---|---|
| - | - | - | No recibe parámetros |

| @return | `void` |
| @throws | `UnsupportedEncodingException` si el charset String no es soportado |

#### `testGetCommonPrefix_StringArray()`
Valida `StringUtils.getCommonPrefix(String...)` con pares sustitutos.
| @return | `void` |

#### `testGetDigits()`
Valida `StringUtils.getDigits(String)` con dígitos Unicode `\u0967`.
| @return | `void` |

#### `testGetDigitsKeycaps()`
Valida `getDigits` con emojis keycap `0️⃣`.
| @return | `void` |

#### `testGetDigitsSupplementary()`
Valida `getDigits` con dígito suplementario `U+1D7CF`.
| @return | `void` |

#### `testGetFuzzyDistance()`
Valida `StringUtils.getFuzzyDistance(CharSequence, CharSequence, Locale)`.
| @return | `void` |

#### `testGetFuzzyDistance_NullNullNull()`
Valida que `getFuzzyDistance(null,null,null)` lanza `IllegalArgumentException`.
| @return | `void` |

#### `testGetFuzzyDistance_NullStringLocale()`
Valida `getFuzzyDistance(null, "clear", Locale.ENGLISH)` lanza `IllegalArgumentException`.
| @return | `void` |

#### `testGetFuzzyDistance_StringNullLoclae()`
Valida `getFuzzyDistance(" ", null, Locale.ENGLISH)` lanza `IllegalArgumentException`. Nota: nombre con typo `Loclae`.
| @return | `void` |

#### `testGetFuzzyDistance_StringStringNull()`
Valida `getFuzzyDistance(" ", "clear", null)` lanza `IllegalArgumentException`.
| @return | `void` |

#### `testGetIfBlank_StringStringSupplier()`
Valida `StringUtils.getIfBlank(T, Supplier<T>)` con evaluación perezosa y `MutableInt`.
| @return | `void` |

#### `testGetIfEmpty_StringStringSupplier()`
Valida `StringUtils.getIfEmpty(T, Supplier<T>)`.
| @return | `void` |

#### `testGetJaroWinklerDistance_NullNull()`
Valida excepción con ambos `null`.
| @return | `void` |

#### `testGetJaroWinklerDistance_NullString()`
Valida excepción con primer `null`.
| @return | `void` |

#### `testGetJaroWinklerDistance_StringNull()`
Valida excepción con segundo `null`.
| @return | `void` |

#### `testGetJaroWinklerDistance_StringString()`
Valida distancias Jaro-Winkler conocidas (`frog`/`fog` = 0.93).
| @return | `void` |

#### `testGetLevenshteinDistance_NullString()`
Valida `getLevenshteinDistance("a", null)` lanza `IllegalArgumentException`.
| @return | `void` |

#### `testGetLevenshteinDistance_NullStringInt()`
Valida `getLevenshteinDistance(null, "a", 0)` lanza excepción.
| @return | `void` |

#### `testGetLevenshteinDistance_StringNull()`
Valida `getLevenshteinDistance(null, "a")` lanza excepción.
| @return | `void` |

#### `testGetLevenshteinDistance_StringNullInt()`
Valida `getLevenshteinDistance("a", null, 0)` lanza excepción.
| @return | `void` |

#### `testGetLevenshteinDistance_StringString()`
Valida distancias Levenshtein sin umbral.
| @return | `void` |

#### `testGetLevenshteinDistance_StringStringInt()`
Valida `getLevenshteinDistance(String, String, int threshold)` con casos de stripe y `Integer.MAX_VALUE`.
| @return | `void` |

#### `testGetLevenshteinDistance_StringStringNegativeInt()`
Valida umbral negativo lanza `IllegalArgumentException`.
| @return | `void` |

#### `testIsAllLowerCase()`
Valida `StringUtils.isAllLowerCase(CharSequence)`.
| @return | `void` |

#### `testIsAllLowerCaseSupplementary()`
Valida con letra suplementaria `U+10428`.
| @return | `void` |

#### `testIsAllUpperCase()`
Valida `isAllUpperCase`.
| @return | `void` |

#### `testIsAllUpperCaseSupplementary()`
Valida con `U+10400`.
| @return | `void` |

#### `testIsMixedCase()`
Valida `isMixedCase`.
| @return | `void` |

#### `testIsMixedCaseSupplementary()`
Valida `isMixedCase` con mezcla `a` + `U+10400`.
| @return | `void` |

#### `testJoin_ArrayCharSeparator()`
Valida `StringUtils.join(Object[], char)` y `join(Object[], char, int, int)`.
| @return | `void` |

#### `testJoin_ArrayOfBooleans()`
Valida `join(boolean[], char, int, int)`.
| @return | `void` |

#### `testJoin_ArrayOfBytes()`
Valida `join(byte[], char, int, int)`.
| @return | `void` |

#### `testJoin_ArrayOfChars()`
Valida `join(char[], char, int, int)`.
| @return | `void` |

#### `testJoin_ArrayOfDoubles()`
Valida `join(double[], char, int, int)`.
| @return | `void` |

#### `testJoin_ArrayOfFloats()`
Valida `join(float[], char, int, int)`.
| @return | `void` |

#### `testJoin_ArrayOfInts()`
Valida `join(int[], char, int, int)`.
| @return | `void` |

#### `testJoin_ArrayOfLongs()`
Valida `join(long[], char, int, int)`.
| @return | `void` |

#### `testJoin_ArrayOfShorts()`
Valida `join(short[], char, int, int)`.
| @return | `void` |

#### `testJoin_ArrayString_EmptyDelimiter()`
Valida `join(Object[], String)` con delimitador `null`/`""`.
| @return | `void` |

#### `testJoin_ArrayString_NonEmptyDelimiter()`
```java
@ParameterizedTest @ValueSource(strings = {",", ";", Supplementary.CharU20000, Supplementary.CharU20001})
void testJoin_ArrayString_NonEmptyDelimiter(String delimiter)
```
| @param | Tipo | Requerido | Descripción |
|---|---|---|---|
| `delimiter` | `String` | Sí | Delimitador parametrizado incluyendo caracteres suplementarios |

| @return | `void` |

#### `testJoin_IterableChar()`
Valida `join(Iterable, char)`.
| @return | `void` |

#### `testJoin_IterableString()`
Valida `join(Iterable, String)`.
| @return | `void` |

#### `testJoin_IteratorChar()`
Valida `join(Iterator, char)`.
| @return | `void` |

#### `testJoin_IteratorString()`
Valida `join(Iterator, String)`.
| @return | `void` |

#### `testJoin_List_CharDelimiter()`
Valida `join(List, char, int, int)`.
| @return | `void` |

#### `testJoin_List_EmptyDelimiter()`
Valida `join(List, String)` con delimitador vacío.
| @return | `void` |

#### `testJoin_List_NonEmptyDelimiter()`
```java
@ParameterizedTest @ValueSource(...)
void testJoin_List_NonEmptyDelimiter(String delimiter)
```
| @param | Tipo | Requerido | Descripción |
|---|---|---|---|
| `delimiter` | `String` | Sí | Delimitador |

| @return | `void` |

#### `testJoin_Objectarray()`
Valida `join(Object...)` varargs.
| @return | `void` |

#### `testJoin_Objects()`
Valida `join(Object...)` con `null`.
| @return | `void` |

#### `testJoinWith()`
```java
@ParameterizedTest @ValueSource(...)
void testJoinWith(String delimiter)
```
Valida `StringUtils.joinWith(String, Object...)`.
| @param | Tipo | Requerido | Descripción |
|---|---|---|---|
| `delimiter` | `String` | Sí | Delimitador |

| @return | `void` |

#### `testJoinWithThrowsException()`
Valida que `joinWith(",", (Object[]) null)` lanza `IllegalArgumentException`.
| @return | `void` |

#### `testLang1593()`
`@Disabled` - Valida consistencia `join(int[], char)` vs `join(int[], String)`.
| @return | `void` |

#### `testLang623()`
Valida `replaceChars` con `\u00DE` y `\u00FE`.
| @return | `void` |

#### `testLANG666()`
Valida `stripEnd("120.00", ".0")`.
| @return | `void` |

#### `testLeftPad_StringInt()`
Valida `leftPad(String, int)`.
| @return | `void` |

#### `testLeftPad_StringIntChar()`
Valida `leftPad(String, int, char)` con pad grande 10000.
| @return | `void` |

#### `testLeftPad_StringIntString()`
Valida `leftPad(String, int, String)`.
| @return | `void` |

#### `testLength_CharBuffer()`
Valida `StringUtils.length(CharBuffer)`.
| @return | `void` |

#### `testLengthString()`
Valida `length(CharSequence)` con `null`.
| @return | `void` |

#### `testLengthStringBuffer()`
Valida `length(StringBuffer)`.
| @return | `void` |

#### `testLengthStringBuilder()`
Valida `length(StringBuilder)`.
| @return | `void` |

#### `testLowerCase()`
Valida `lowerCase(String)` y `lowerCase(String, Locale)`.
| @return | `void` |

#### `testNormalizeSpace()`
Valida `normalizeSpace(String)` con `\u00A0` y caracteres de control.
| @return | `void` |

#### `testOverlay_StringStringIntInt()`
Valida `overlay(String, String, int, int)` con índices negativos y fuera de rango.
| @return | `void` |

#### `testPrependIfMissing()`
Valida `prependIfMissing`.
| @return | `void` |

#### `testPrependIfMissingIgnoreCase()`
Valida `prependIfMissingIgnoreCase`.
| @return | `void` |

#### `testReCapitalize()`
Valida idempotencia `capitalize`/`uncapitalize`.
| @return | `void` |

#### `testRemove_char()`
Valida `remove(String, char)`.
| @return | `void` |

#### `testRemove_String()`
Valida `remove(String, String)`.
| @return | `void` |

#### `testRemoveAll_StringString()`
Valida `removeAll(String, String regex)` y `PatternSyntaxException`.
| @return | `void` |

#### `testRemoveEnd()`
Valida `removeEnd(String, String)`.
| @return | `void` |

#### `testRemoveEndIgnoreCase()`
Valida `removeEndIgnoreCase`.
| @return | `void` |

#### `testRemoveFirst_StringString()`
Valida `removeFirst(String, String)`.
| @return | `void` |

#### `testRemoveIgnoreCase_String()`
Valida `removeIgnoreCase`.
| @return | `void` |

#### `testRemovePattern_StringString()`
Valida `removePattern`.
| @return | `void` |

#### `testRemoveStartChar()`
Valida `removeStart(String, char)`.
| @return | `void` |

#### `testRemoveStartIgnoreCase()`
Valida `removeStartIgnoreCase`.
| @return | `void` |

#### `testRemoveStartString()`
Valida `removeStart(String, String)`.
| @return | `void` |

#### `testRepeat_CharInt()`
Valida `repeat(char, int)`.
| @return | `void` |

#### `testRepeat_StringInt()`
Valida `repeat(String, int)` y `IllegalArgumentException` por overflow.
| @return | `void` |

#### `testRepeat_StringStringInt()`
Valida `repeat(String, String, int)`.
| @return | `void` |

#### `testReplace_StringStringArrayStringArray()`
Valida `replaceEach(String, String[], String[])` y `IllegalArgumentException` por longitudes distintas.
| @return | `void` |

#### `testReplace_StringStringArrayStringArrayBoolean()`
Valida `replaceEachRepeatedly` con TTL y detección de referencia circular `IllegalStateException`.
| @return | `void` |

#### `testReplace_StringStringString()`
Valida `replace(String, String, String)`.
| @return | `void` |

#### `testReplace_StringStringStringInt()`
Valida `replace(String, String, String, int max)`.
| @return | `void` |

#### `testReplaceAll_StringStringString()`
Valida `replaceAll(String, String, String)`.
| @return | `void` |

#### `testReplaceChars_StringCharChar()`
Valida `replaceChars(String, char, char)`.
| @return | `void` |

#### `testReplaceChars_StringStringString()`
Valida `replaceChars(String, String, String)`.
| @return | `void` |

#### `testReplaceFirst_StringStringString()`
Valida `replaceFirst`.
| @return | `void` |

#### `testReplaceIgnoreCase_StringStringString()`
Valida `replaceIgnoreCase`.
| @return | `void` |

#### `testReplaceIgnoreCase_StringStringStringInt()`
Valida `replaceIgnoreCase` con `max`.
| @return | `void` |

#### `testReplaceOnce_StringStringString()`
Valida `replaceOnce`.
| @return | `void` |

#### `testReplaceOnceIgnoreCase_StringStringString()`
Valida `replaceOnceIgnoreCase`.
| @return | `void` |

#### `testReplacePattern_StringStringString()`
Valida `replacePattern`.
| @return | `void` |

#### `testReverse_String()`
Valida `reverse(String)`.
| @return | `void` |

#### `testReverseDelimited_StringChar()`
Valida `reverseDelimited(String, char)`.
| @return | `void` |

#### `testRightPad_StringInt()`
Valida `rightPad(String, int)`.
| @return | `void` |

#### `testRightPad_StringIntChar()`
Valida `rightPad(String, int, char)`.
| @return | `void` |

#### `testRightPad_StringIntString()`
Valida `rightPad(String, int, String)`.
| @return | `void` |

#### `testRotate_StringInt()`
Valida `rotate(String, int)` con rotaciones positivas/negativas y wrap.
| @return | `void` |

#### `testSplit_String()`
Valida `split(String)` por whitespace.
| @return | `void` |

#### `testSplit_StringChar()`
Valida `split(String, char)`.
| @return | `void` |

#### `testSplit_StringString_StringStringInt()`
Valida `split(String, String)` y `split(String, String, int)` delegando a `innerTestSplit`.
| @return | `void` |

#### `testSplitByCharacterType()`
Valida `splitByCharacterType(String)` con suplementarios `U+1D400`, `U+1D7D3`.
| @return | `void` |

#### `testSplitByCharacterTypeCamelCase()`
Valida `splitByCharacterTypeCamelCase`.
| @return | `void` |

#### `testSplitByWholeSeparatorPreserveAllTokens_StringString()`
Valida `splitByWholeSeparatorPreserveAllTokens(String, String)`.
| @return | `void` |

#### `testSplitByWholeSeparatorPreserveAllTokens_StringStringInt()`
Valida `splitByWholeSeparatorPreserveAllTokens(String, String, int)`.
| @return | `void` |

#### `testSplitByWholeString_StringStringBoolean()`
Valida `splitByWholeSeparator(String, String)`.
| @return | `void` |

#### `testSplitByWholeString_StringStringBooleanInt()`
Valida `splitByWholeSeparator(String, String, int)`.
| @return | `void` |

#### `testSplitByWholeStringDropsTrailingEmpty()`
```java
@ParameterizedTest @CsvSource({...})
void testSplitByWholeStringDropsTrailingEmpty(String str, String separator, String expected)
```
| @param | Tipo | Requerido | Descripción |
|---|---|---|---|
| `str` | `String` | Sí | Cadena a dividir |
| `separator` | `String` | Sí | Separador completo |
| `expected` | `String` | Sí | Tokens esperados separados por coma |

| @return | `void` |

#### `testSplitPreserveAllTokens_String()`
Valida `splitPreserveAllTokens(String)`.
| @return | `void` |

#### `testSplitPreserveAllTokens_StringChar()`
Valida `splitPreserveAllTokens(String, char)`.
| @return | `void` |

#### `testSplitPreserveAllTokens_StringString_StringStringInt()`
Valida `splitPreserveAllTokens(String, String)` y `splitPreserveAllTokens(String, String, int)`.
| @return | `void` |

#### `testStringUtilsCharSequenceContract()`
Valida por reflexión que métodos inmutables acepten `CharSequence` y mutables `String`.
| @return | `void` |

#### `testSwapCase_String()`
Valida `swapCase(String)` y `WordUtils.swapCase`.
| @return | `void` |

#### `testToCodePoints()`
Valida `toCodePoints(String)` con surrogates huérfanos.
| @return | `void` |

#### `testToCodePointsEmoji()`
Valida `toCodePoints` con emojis `🦊` y familia `👩🏻‍👨🏻‍👦🏻`.
| @return | `void` |

#### `testToEncodedString()`
Valida `toEncodedString(byte[], Charset)`.
| @return | `void` |

#### `testToRootLowerCase()`
Valida `toRootLowerCase(String)` con `Locale.ROOT` vs `tr`.
| @return | `void` |

#### `testToRootLowerCaseTurkish()`
Anotado `@DefaultLocale("tr")`, valida `toRootLowerCase` bajo locale turco.
| @return | `void` |

#### `testToRootUpperCase()`
Valida `toRootUpperCase(String)` y que ignora `Locale.getDefault()`.
| @return | `void` |

#### `testToString()`
```java
void testToString() throws UnsupportedEncodingException
```
Valida `StringUtils.toString(byte[], String)`.
| @return | `void` |
| @throws | `UnsupportedEncodingException` si el encoding no es soportado |

#### `testTruncate_StringInt()`
Valida `truncate(String, int maxWidth)` y `IllegalArgumentException` si `maxWidth < 0`.
| @return | `void` |

#### `testTruncate_StringIntInt()`
Valida `truncate(String, int offset, int maxWidth)`.
| @return | `void` |

#### `testTruncate_StringIntInt_surrogatePair()`
Valida `truncate` con emoji `😀` evitando surrogates huérfanos.
| @return | `void` |

#### `testUnCapitalize()`
Valida `uncapitalize(String)`.
| @return | `void` |

#### `testUnescapeSurrogatePairs()`
Valida `unescapeCsv/Html3/Html4` con pares sustitutos.
| @return | `void` |

#### `testUnwrap_StringChar()`
Valida `unwrap(String, char)`.
| @return | `void` |

#### `testUnwrap_StringString()`
Valida `unwrap(String, String)`.
| @return | `void` |

#### `testUpperCase()`
Valida `upperCase(String)` y `upperCase(String, Locale)`.
| @return | `void` |

#### `testWrap_StringChar()`
Valida `wrap(String, char)`.
| @return | `void` |

#### `testWrap_StringString()`
Valida `wrap(String, String)`.
| @return | `void` |

#### `testWrapIfMissing_StringChar()`
Valida `wrapIfMissing(String, char)` con `assertSame` para casos ya envueltos.
| @return | `void` |

#### `testWrapIfMissing_StringString()`
Valida `wrapIfMissing(String, String)`.
| @return | `void` |

---

## 3. Ejemplo de Uso

Ejemplo real de ejecución y extensión de la suite con JUnit 5:

```java
package org.apache.commons.lang3;

import static org.junit.jupiter.api.Assertions.assertEquals;
import org.junit.jupiter.api.Test;

/**
 * Ejemplo de uso de StringUtilsTest como referencia
 * y de StringUtils en código cliente.
 */
public class EjemploUsoStringUtils {

    // Uso directo de StringUtils validado por StringUtilsTest
    public void ejemploCliente() {
        // 1. Capitalize / Uncapitalize
        String cap = StringUtils.capitalize("hello world"); // "Hello world"
        String uncap = StringUtils.uncapitalize("Hello");   // "hello"

        // 2. Join con manejo de null y tipos mixtos
        String joined = StringUtils.join(new Object[]{"foo", null, "bar"}, ';'); // "foo;;bar"
        String with = StringUtils.joinWith(",", "a", "b", "c"); // "a,b,c"

        // 3. Truncate seguro con surrogate pairs (emoji)
        String grin = "😀";
        String trunc = StringUtils.truncate("a" + grin + "b", 0, 2); // "a" - no deja surrogate huérfano

        // 4. Split preservando tokens
        String[] tokens = StringUtils.splitPreserveAllTokens("a..b.c", '.'); // ["a", "", "b", "c"]

        System.out.println(cap + " | " + joined + " | " + trunc);
    }

    // Extender la suite para un caso propio
    @Test
    void testMiCasoPersonalizado() {
        // Reutiliza el patrón de StringUtilsTest
        assertEquals("ab", StringUtils.truncate("abcd", 2));
        assertEquals("  ab  ", StringUtils.center("ab", 6));
        assertEquals("a,b,c", StringUtils.join(new String[]{"a","b","c"}, ","));
    }

    // Ejecución vía Maven:
    // mvn -Dtest=org.apache.commons.lang3.StringUtilsTest test
    // Ejecución vía Gradle:
    // gradle test --tests "org.apache.commons.lang3.StringUtilsTest"
}
```

**Notas de ejecución:**
* Requiere JUnit Jupiter 5.8+ y `junit-pioneer` para `@DefaultLocale`.
* La clase no debe instanciarse en producción; se ejecuta mediante el runner de JUnit.
* Para depurar un método específico: `mvn -Dtest=StringUtilsTest#testTruncate_StringIntInt_surrogatePair test`.

