# Documentación técnica – Clase `StringUtilsTest`

## 1. Resumen ejecutivo
`StringUtilsTest` es una suite de pruebas JUnit 5 que comprueba de forma exhaustiva el correcto funcionamiento de la clase utilitaria `org.apache.commons.lang3.StringUtils` y de clases auxiliares asociadas (`StringEscapeUtils`, `ArrayUtils`, etc.).  
El objetivo de la clase es validar **todos** los escenarios límite, comportamientos previstos y regresiones detectadas previamente en la biblioteca *Apache Commons Lang* relativos al tratamiento de cadenas de caracteres.

---

## 2. Especificación de constructores y métodos

### 2.1 Constructores

| Constructor | Descripción | @throws |
|-------------|-------------|---------|
| `public StringUtilsTest()` | Construye la clase de pruebas. No realiza inicialización adicional. | – |

### 2.2 Métodos auxiliares (internos)

| Método | @param (nombre - tipo - requerido) | @return | @throws |
|--------|-------------------------------------|---------|---------|
| `private void innerTestSplit(char separator, String sepStr, char noMatch)` | 1. `separator` - `char` - Sí<br>2. `sepStr` - `String` - Sí (puede ser `null`)<br>3. `noMatch` - `char` - Sí | `void` – Verifica distintos comportamientos de `StringUtils.split` usando los valores recibidos. | – |
| `private void innerTestSplitPreserveAllTokens(char separator, String sepStr, char noMatch)` | 1. `separator` - `char` - Sí<br>2. `sepStr` - `String` - Sí (puede ser `null`)<br>3. `noMatch` - `char` - Sí | `void` – Verifica los mismos casos que el método anterior pero usando `StringUtils.splitPreserveAllTokens`. | – |

### 2.3 Métodos de prueba JUnit

Los métodos de prueba validan múltiples aspectos de `StringUtils`. Todos ellos son de la forma:

```java
@Test // ó @ParameterizedTest
void testXxx...(...)
```

* Visibilidad: `package-private`  
* Retorno: `void`  
* Excepciones esperadas: se controlan mediante `assertThrows`, por lo que no propagan checked exceptions.

A continuación se listan **todos** los métodos presentes en el código fuente, indicando sus parámetros cuando corresponda:

| Método | Parámetros (@param) |
|--------|---------------------|
| `testAppendIfMissing()` | – |
| `testAppendIfMissingIgnoreCase()` | – |
| `testCapitalize()` | – |
| `testCenter_StringInt()` | – |
| `testCenter_StringIntChar()` | – |
| `testCenter_StringIntString()` | – |
| `testChomp()` | – |
| `testChop()` | – |
| `testConstructor()` | – |
| `testDefault_String()` | – |
| `testDefault_StringString()` | – |
| `testDefaultIfBlank_CharBuffers()` | – |
| `testDefaultIfBlank_StringBuffers()` | – |
| `testDefaultIfBlank_StringBuilders()` | – |
| `testDefaultIfBlank_StringString()` | – |
| `testDefaultIfEmpty_CharBuffers()` | – |
| `testDefaultIfEmpty_StringBuffers()` | – |
| `testDefaultIfEmpty_StringBuilders()` | – |
| `testDefaultIfEmpty_StringString()` | – |
| `testDeleteWhitespace_String()` | – |
| `testDifference_StringString()` | – |
| `testDifferenceAt_StringArray()` | – |
| `testDifferenceAt_StringString()` | – |
| `testEMPTY()` | – |
| `testEscapeSurrogatePairs()` | – |
| `testEscapeSurrogatePairsLang858()` | – |
| `testGeorgianSample()` | – |
| `testGetBytes_Charset()` | – |
| `testGetBytes_String()` | `throws UnsupportedEncodingException` *(manejada internamente)* |
| `testGetCommonPrefix_StringArray()` | – |
| `testGetDigits()` | – |
| `testGetDigitsKeycaps()` | – |
| `testGetDigitsSupplementary()` | – |
| `testGetFuzzyDistance()` | – |
| `testGetFuzzyDistance_NullNullNull()` | – |
| `testGetFuzzyDistance_NullStringLocale()` | – |
| `testGetFuzzyDistance_StringNullLoclae()` | – |
| `testGetFuzzyDistance_StringStringNull()` | – |
| `testGetIfBlank_StringStringSupplier()` | – |
| `testGetIfEmpty_StringStringSupplier()` | – |
| `testGetJaroWinklerDistance_NullNull()` | – |
| `testGetJaroWinklerDistance_NullString()` | – |
| `testGetJaroWinklerDistance_StringNull()` | – |
| `testGetJaroWinklerDistance_StringString()` | – |
| `testGetLevenshteinDistance_NullString()` | – |
| `testGetLevenshteinDistance_NullStringInt()` | – |
| `testGetLevenshteinDistance_StringNull()` | – |
| `testGetLevenshteinDistance_StringNullInt()` | – |
| `testGetLevenshteinDistance_StringString()` | – |
| `testGetLevenshteinDistance_StringStringInt()` | – |
| `testGetLevenshteinDistance_StringStringNegativeInt()` | – |
| `testIsAllLowerCase()` | – |
| `testIsAllLowerCaseSupplementary()` | – |
| `testIsAllUpperCase()` | – |
| `testIsAllUpperCaseSupplementary()` | – |
| `testIsMixedCase()` | – |
| `testIsMixedCaseSupplementary()` | – |
| `testJoin_ArrayCharSeparator()` | – |
| `testJoin_ArrayOfBooleans()` | – |
| `testJoin_ArrayOfBytes()` | – |
| `testJoin_ArrayOfChars()` | – |
| `testJoin_ArrayOfDoubles()` | – |
| `testJoin_ArrayOfFloats()` | – |
| `testJoin_ArrayOfInts()` | – |
| `testJoin_ArrayOfLongs()` | – |
| `testJoin_ArrayOfShorts()` | – |
| `testJoin_ArrayString_EmptyDelimiter()` | – |
| `testJoin_ArrayString_NonEmptyDelimiter(String delimiter)` | `delimiter` – `String` |
| `testJoin_IterableChar()` | – |
| `testJoin_IterableString()` | – |
| `testJoin_IteratorChar()` | – |
| `testJoin_IteratorString()` | – |
| `testJoin_List_CharDelimiter()` | – |
| `testJoin_List_EmptyDelimiter()` | – |
| `testJoin_List_NonEmptyDelimiter(String delimiter)` | `delimiter` – `String` |
| `testJoin_Objectarray()` | – |
| `testJoin_Objects()` | – |
| `testJoinWith(String delimiter)` | `delimiter` – `String` |
| `testJoinWithThrowsException()` | – |
| `testLang1593()` *(deshabilitada)* | – |
| `testLang623()` | – |
| `testLANG666()` | – |
| `testLeftPad_StringInt()` | – |
| `testLeftPad_StringIntChar()` | – |
| `testLeftPad_StringIntString()` | – |
| `testLength_CharBuffer()` | – |
| `testLengthString()` | – |
| `testLengthStringBuffer()` | – |
| `testLengthStringBuilder()` | – |
| `testLowerCase()` | – |
| `testNormalizeSpace()` | – |
| `testOverlay_StringStringIntInt()` | – |
| `testPrependIfMissing()` | – |
| `testPrependIfMissingIgnoreCase()` | – |
| `testReCapitalize()` | – |
| `testRemove_char()` | – |
| `testRemove_String()` | – |
| `testRemoveAll_StringString()` | – |
| `testRemoveEnd()` | – |
| `testRemoveEndIgnoreCase()` | – |
| `testRemoveFirst_StringString()` | – |
| `testRemoveIgnoreCase_String()` | – |
| `testRemovePattern_StringString()` | – |
| `testRemoveStartChar()` | – |
| `testRemoveStartIgnoreCase()` | – |
| `testRemoveStartString()` | – |
| `testRepeat_CharInt()` | – |
| `testRepeat_StringInt()` | – |
| `testRepeat_StringStringInt()` | – |
| `testReplace_StringStringArrayStringArray()` | – |
| `testReplace_StringStringArrayStringArrayBoolean()` | – |
| `testReplace_StringStringString()` | – |
| `testReplace_StringStringStringInt()` | – |
| `testReplaceAll_StringStringString()` | – |
| `testReplaceChars_StringCharChar()` | – |
| `testReplaceChars_StringStringString()` | – |
| `testReplaceFirst_StringStringString()` | – |
| `testReplaceIgnoreCase_StringStringString()` | – |
| `testReplaceIgnoreCase_StringStringStringInt()` | – |
| `testReplaceOnce_StringStringString()` | – |
| `testReplaceOnceIgnoreCase_StringStringString()` | – |
| `testReplacePattern_StringStringString()` | – |
| `testReverse_String()` | – |
| `testReverseDelimited_StringChar()` | – |
| `testRightPad_StringInt()` | – |
| `testRightPad_StringIntChar()` | – |
| `testRightPad_StringIntString()` | – |
| `testRotate_StringInt()` | – |
| `testSplit_String()` | – |
| `testSplit_StringChar()` | – |
| `testSplit_StringString_StringStringInt()` | – |
| `testSplitByCharacterType()` | – |
| `testSplitByCharacterTypeCamelCase()` | – |
| `testSplitByWholeSeparatorPreserveAllTokens_StringString()` | – |
| `testSplitByWholeSeparatorPreserveAllTokens_StringStringInt()` | – |
| `testSplitByWholeString_StringStringBoolean()` | – |
| `testSplitByWholeString_StringStringBooleanInt()` | – |
| `testSplitByWholeStringDropsTrailingEmpty(String str,String separator,String expected)` | `str` – `String`<br>`separator` – `String`<br>`expected` – `String` |
| `testSplitPreserveAllTokens_String()` | – |
| `testSplitPreserveAllTokens_StringChar()` | – |
| `testSplitPreserveAllTokens_StringString_StringStringInt()` | – |
| `testStringUtilsCharSequenceContract()` | – |
| `testSwapCase_String()` | – |
| `testToCodePoints()` | – |
| `testToCodePointsEmoji()` | – |
| `testToEncodedString()` | – |
| `testToRootLowerCase()` | – |
| `testToRootLowerCaseTurkish()` | – |
| `testToRootUpperCase()` | – |
| `testToString()` | `throws UnsupportedEncodingException` *(manejada internamente)* |
| `testTruncate_StringInt()` | – |
| `testTruncate_StringIntInt()` | – |
| `testTruncate_StringIntInt_surrogatePair()` | – |
| `testUnCapitalize()` | – |
| `testUnescapeSurrogatePairs()` | – |
| `testUnwrap_StringChar()` | – |
| `testUnwrap_StringString()` | – |
| `testUpperCase()` | – |
| `testWrap_StringChar()` | – |
| `testWrap_StringString()` | – |
| `testWrapIfMissing_StringChar()` | – |
| `testWrapIfMissing_StringString()` | – |

> **Nota:** En los métodos parametrizados (`@ParameterizedTest`) la columna Parámetros refleja las variables de método inyectadas por JUnit.

---

## 3. Ejemplo de uso

A continuación se muestra un caso simplificado que reutiliza los métodos auxiliares para validar un escenario concreto fuera del contexto de JUnit:

```java
import org.apache.commons.lang3.StringUtils;

/**
 * Ejemplo de reutilización de la lógica de test sin JUnit.
 */
public class SplitChecker {

    public static void main(String[] args) {
        // Caso equivalente a lo comprobado en innerTestSplit
        String sample = "a,b,,c";
        String[] tokens = StringUtils.split(sample, ",");
        if (tokens.length == 3
                && "a".equals(tokens[0])
                && "b".equals(tokens[1])
                && "c".equals(tokens[2])) {
            System.out.println("Validación manual superada.");
        } else {
            throw new IllegalStateException("Resultado inesperado en split.");
        }
    }
}
```

Este fragmento demuestra de forma lúdica cómo aprovechar la lógica verificada por `StringUtilsTest` en un programa autónomo.

---

### © Apache Commons Lang – Equipo de desarrollo  
@author Apache Commons Lang contributors