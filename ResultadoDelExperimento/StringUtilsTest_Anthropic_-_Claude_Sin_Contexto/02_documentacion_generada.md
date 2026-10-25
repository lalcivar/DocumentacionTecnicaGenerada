# Documentación Técnica: `StringUtilsTest`

**Paquete:** `org.apache.commons.lang3`
**Tipo:** Clase de pruebas unitarias (JUnit 5 / Jupiter)
**Jerarquía:** `StringUtilsTest extends AbstractLangTest`
**Visibilidad:** Package-private (`class`)
**Anotaciones de clase:** `@SuppressWarnings("deprecation")` — se usan deliberadamente APIs marcadas como obsoletas para garantizar su compatibilidad retroactiva.

---

## 1. Descripción general

`StringUtilsTest` es la suite de pruebas unitarias principal de la clase utilitaria `org.apache.commons.lang3.StringUtils` de Apache Commons Lang 3.

Su objetivo es **verificar exhaustivamente el contrato funcional** de los métodos estáticos de manipulación de cadenas que ofrece `StringUtils`, cubriendo:

- **Casos normales** (entradas típicas).
- **Casos límite / frontera** (cadenas vacías, índices negativos, `Integer.MIN_VALUE`, `Integer.MAX_VALUE`, longitudes mayores al búfer de relleno).
- **Seguridad frente a `null`** (`null-safety`), una de las garantías centrales de `StringUtils`.
- **Soporte Unicode avanzado**: pares subrogados (*surrogate pairs*), puntos de código suplementarios (planos astrales), emojis, secuencias ZWJ, modificadores de tono de piel y caracteres *TitleCase*.
- **Sensibilidad al `Locale`** (problema clásico del idioma turco con la letra `I`).
- **Expresiones regulares** y lanzamiento de `PatternSyntaxException`.
- **Excepciones esperadas**: `IllegalArgumentException`, `IllegalStateException`.

> **Nota:** El Javadoc original indica que esta clase agrupa los métodos de `StringUtils` **que no han sido migrados** a clases de prueba independientes (existen suites separadas para `trim`, `strip`, `isEmpty`, `contains`, `indexOf`, etc.).

---

## 2. Dependencias relevantes

| Dependencia | Uso |
|---|---|
| `org.junit.jupiter.api.*` | Framework de pruebas (`@Test`, `@Disabled`, aserciones estáticas). |
| `org.junit.jupiter.params.*` | Pruebas parametrizadas (`@ParameterizedTest`, `@ValueSource`, `@CsvSource`). |
| `org.junitpioneer.jupiter.*` | Control del `Locale` por defecto (`@DefaultLocale`, `@ReadsDefaultLocale`, `@WritesDefaultLocale`). |
| `LangAssertions.assertIllegalArgumentException` | Aserción auxiliar propia del proyecto. |
| `org.apache.commons.lang3.mutable.MutableInt` | Contador para verificar evaluación diferida (*lazy*) de `Supplier`. |
| `org.apache.commons.lang3.function.Suppliers` | Proveedores utilitarios (`Suppliers.nul()`). |
| `java.lang.reflect.*` | Introspección para validar el contrato de la API. |

---

## 3. Campos y datos de prueba (*fixtures*)

### 3.1 Constantes de espaciado (inicializadas en bloque `static`)

Un bloque estático recorre todo el rango `[0, Character.MAX_VALUE)` y clasifica los caracteres:

| Constante | Contenido |
|---|---|
| `WHITESPACE` | Todos los caracteres para los que `Character.isWhitespace()` es `true`. |
| `NON_WHITESPACE` | Caracteres `< 40` que no son espacios en blanco. |
| `HARD_SPACE` | Espacio duro / no separable (`char` 160, `\u00A0`). |
| `TRIMMABLE` | Caracteres `[0, 32]` (recortables por `String.trim()`). |
| `NON_TRIMMABLE` | Espacios en blanco con código `> 32` (no recortables por `trim()`). |

### 3.2 Arreglos y listas de prueba

Conjuntos predefinidos para validar las sobrecargas de `join`:

- **Arreglos de objetos:** `ARRAY_LIST`, `EMPTY_ARRAY_LIST`, `NULL_ARRAY_LIST`, `MIXED_ARRAY_LIST`, `MIXED_TYPE_LIST`, `NULL_TO_STRING_LIST` (objeto anónimo cuyo `toString()` devuelve `null`).
- **Arreglos primitivos:** `LONG_PRIM_LIST`, `INT_PRIM_LIST`, `BYTE_PRIM_LIST`, `SHORT_PRIM_LIST`, `CHAR_PRIM_LIST`, `FLOAT_PRIM_LIST`, `DOUBLE_PRIM_LIST`, `EMPTY`, `ARRAY_FALSE_*`.
- **Colecciones:** `STRING_LIST`, `EMPTY_STRING_LIST`, `NULL_STRING_LIST`, `MIXED_STRING_LIST`, `MIXED_TYPE_OBJECT_LIST`.
- **Separadores y resultados esperados:** `SEPARATOR` (`","`), `SEPARATOR_CHAR` (`';'`), `COMMA_SEPARATOR_CHAR`, `TEXT_LIST`, `TEXT_LIST_CHAR`, `TEXT_LIST_NOSEP`.
- **Capitalización:** `FOO_UNCAP`/`FOO_CAP`, `SENTENCE_UNCAP`/`SENTENCE_CAP`.

---

## 4. Métodos auxiliares (privados)

### `private void innerTestSplit(char separator, String sepStr, char noMatch)`
Rutina reutilizable que valida `StringUtils.split(String, String)` y `split(String, String, int)` sobre una cadena sintética `"a" + sep + "b" + sep + sep + noMatch + "c"`.
Verifica:
- Colapso de separadores adyacentes.
- Eliminación de tokens vacíos iniciales/finales.
- Comportamiento del límite `max`: `-1`, `0` (sin límite), `1`, `2`.

Se invoca en bucle con todos los pares (espacio en blanco × no-espacio) para cobertura exhaustiva.

### `private void innerTestSplitPreserveAllTokens(char separator, String sepStr, char noMatch)`
Equivalente al anterior, pero para `splitPreserveAllTokens(...)`: comprueba que **sí** se conservan los tokens vacíos derivados de separadores contiguos, iniciales y finales.

---

## 5. Métodos de prueba principales, agrupados por funcionalidad

### 5.1 Añadir / anteponer sufijos y prefijos
| Método | Cubre |
|---|---|
| `testAppendIfMissing()` | `appendIfMissing` (con y sin lista de sufijos alternativos). |
| `testAppendIfMissingIgnoreCase()` | Variante insensible a mayúsculas/minúsculas. |
| `testPrependIfMissing()` | `prependIfMissing`. |
| `testPrependIfMissingIgnoreCase()` | Variante insensible a caso. |

### 5.2 Capitalización y cambio de caso
| Método | Cubre |
|---|---|
| `testCapitalize()` / `testUnCapitalize()` | `capitalize` / `uncapitalize`, incluido el uso de *TitleCase* (`\u01C9` → `\u01C8`). |
| `testReCapitalize()` | Propiedad de reflexividad: `uncapitalize(capitalize(s)) == s`. |
| `testLowerCase()` / `testUpperCase()` | Con y sin `Locale`. |
| `testToRootLowerCase()` / `testToRootUpperCase()` / `testToRootLowerCaseTurkish()` | Independencia del `Locale` por defecto (`Locale.ROOT`), con el caso turco como control. |
| `testSwapCase_String()` | `swapCase`, comparado contra `WordUtils.swapCase`. |
| `testIsAllLowerCase()`, `testIsAllUpperCase()`, `testIsMixedCase()` y sus variantes `...Supplementary()` | Predicados de caso, incluidos puntos de código suplementarios (alfabeto Deseret `U+10400` / `U+10428`). |

### 5.3 Relleno, centrado y recorte estructural
- `testCenter_StringInt`, `testCenter_StringIntChar`, `testCenter_StringIntString`
- `testLeftPad_StringInt|StringIntChar|StringIntString`
- `testRightPad_StringInt|StringIntChar|StringIntString`
- `testChomp()`, `testChop()` — `chop` verifica que un par subrogado final se elimine completo y nunca quede un subrogado huérfano.
- `testTruncate_StringInt`, `testTruncate_StringIntInt`, `testTruncate_StringIntInt_surrogatePair()`
- `testRotate_StringInt()`, `testReverse_String()`, `testReverseDelimited_StringChar()`

### 5.4 Valores por defecto y proveedores diferidos
- `testDefault_String`, `testDefault_StringString`
- `testDefaultIfEmpty_*` y `testDefaultIfBlank_*` para `String`, `StringBuilder`, `StringBuffer` y `CharBuffer` (verifican además la **covarianza del tipo de retorno** genérico).
- `testGetIfEmpty_StringStringSupplier()` y `testGetIfBlank_StringStringSupplier()`: usan un `MutableInt` para confirmar que el `Supplier` se invoca **solo cuando es necesario** (evaluación perezosa).

### 5.5 Unión (`join`)
Batería muy extensa:
- Por tipo de contenedor: `testJoin_Objectarray`, `testJoin_Objects`, `testJoin_IterableString`, `testJoin_IterableChar`, `testJoin_IteratorString`, `testJoin_IteratorChar`, `testJoin_List_*`.
- Por tipo primitivo: `testJoin_ArrayOfBooleans|Bytes|Chars|Shorts|Ints|Longs|Floats|Doubles`.
- Con delimitadores: `testJoin_ArrayCharSeparator`, `testJoin_ArrayString_EmptyDelimiter`, y los parametrizados `testJoin_ArrayString_NonEmptyDelimiter(String)` y `testJoin_List_NonEmptyDelimiter(String)` que incluyen delimitadores **suplementarios** (`Supplementary.CharU20000`).
- `testJoinWith(String)` y `testJoinWithThrowsException()`.

### 5.6 División (`split`)
- `testSplit_String`, `testSplit_StringChar`, `testSplit_StringString_StringStringInt`
- `testSplitPreserveAllTokens_*`
- `testSplitByWholeString_*`, `testSplitByWholeSeparatorPreserveAllTokens_*`
- `testSplitByWholeStringDropsTrailingEmpty(...)` — parametrizado con `@CsvSource`; valida que el separador final no genere un token vacío en la variante normal, pero sí en la variante *preserve all tokens*.
- `testSplitByCharacterType()` y `testSplitByCharacterTypeCamelCase()` — incluyen letras y dígitos matemáticos suplementarios (`U+1D400`, `U+1D7D3`).

### 5.7 Reemplazo y eliminación
| Método | Cubre |
|---|---|
| `testReplace_StringStringString`, `testReplace_StringStringStringInt` | Reemplazo literal, con límite de ocurrencias. |
| `testReplaceIgnoreCase_*`, `testReplaceOnce_*`, `testReplaceOnceIgnoreCase_*` | Variantes insensibles a caso y de una sola ocurrencia. |
| `testReplace_StringStringArrayStringArray` | `replaceEach`, incluida validación de arreglos de longitud dispar (`IllegalArgumentException`). |
| `testReplace_StringStringArrayStringArrayBoolean` | `replaceEachRepeatedly`: detección de **referencias circulares** y agotamiento del TTL → `IllegalStateException` (LANG-1528, LANG-1753). |
| `testReplaceAll_*`, `testReplaceFirst_*`, `testReplacePattern_*` | Reemplazo basado en regex. |
| `testReplaceChars_StringCharChar`, `testReplaceChars_StringStringString` | Traducción carácter a carácter. |
| `testRemove_char`, `testRemove_String`, `testRemoveIgnoreCase_String` | Eliminación literal. |
| `testRemoveAll_*`, `testRemoveFirst_*`, `testRemovePattern_*` | Eliminación por regex (verifica `PatternSyntaxException`). |
| `testRemoveStartString`, `testRemoveStartChar`, `testRemoveStartIgnoreCase`, `testRemoveEnd`, `testRemoveEndIgnoreCase` | Eliminación de prefijos/sufijos. |

### 5.8 Comparación y métricas de distancia
- `testDifference_StringString`, `testDifferenceAt_StringString`, `testDifferenceAt_StringArray`, `testGetCommonPrefix_StringArray` — todos incluyen aserciones sobre pares subrogados compartidos para garantizar que nunca se devuelva un subrogado aislado.
- `testGetFuzzyDistance()` y sus cuatro variantes de validación de argumentos nulos.
- `testGetJaroWinklerDistance_StringString` + variantes con `null`.
- `testGetLevenshteinDistance_StringString`, `..._StringStringInt` (umbral), `..._StringStringNegativeInt` y variantes con `null`.
- `testGeorgianSample()` — compara `startsWithIgnoreCase`, `endsWithIgnoreCase`, `compareIgnoreCase`, `indexOfIgnoreCase` y `lastIndexOfIgnoreCase` contra `String.equalsIgnoreCase` para caracteres problemáticos (I sin punto, Theta griego).

### 5.9 Conversión, codificación y longitud
- `testGetBytes_Charset()`, `testGetBytes_String()`
- `testToEncodedString()`, `testToString()`
- `testToCodePoints()`, `testToCodePointsEmoji()` (emojis compuestos con ZWJ y modificadores Fitzpatrick)
- `testLengthString`, `testLength_CharBuffer`, `testLengthStringBuffer`, `testLengthStringBuilder`
- `testGetDigits()`, `testGetDigitsKeycaps()`, `testGetDigitsSupplementary()`

### 5.10 Envoltura y desenvoltura
- `testWrap_StringChar`, `testWrap_StringString`
- `testWrapIfMissing_StringChar`, `testWrapIfMissing_StringString` (verifican además identidad de referencia con `assertSame` cuando no hay cambios)
- `testUnwrap_StringChar`, `testUnwrap_StringString`

### 5.11 Pruebas estructurales y de regresión

| Método | Propósito |
|---|---|
| `testConstructor()` | Vía reflexión, confirma que `StringUtils` tiene **un único constructor público** y que la clase es pública y **no final** (requisito de compatibilidad histórica). |
| `testStringUtilsCharSequenceContract()` | **Prueba de contrato de API por reflexión**: recorre todos los métodos públicos y exige que los métodos "inmutables en espíritu" (que no devuelven `String`/`String[]`) acepten `CharSequence` como primer parámetro, y que los "mutables en espíritu" (que devuelven `String`/`String[]`) **no** acepten `CharSequence`. Mantiene una lista blanca `excludeMethods` con excepciones legítimas. |
| `testEMPTY()` | Comprobación de cordura de la constante `StringUtils.EMPTY`. |
| `testNormalizeSpace()` | `normalizeSpace`, incluido el tratamiento del espacio duro y de caracteres de control. |
| `testDeleteWhitespace_String()` | `deleteWhitespace` con espacios Unicode especiales (`\u00A0`, `\u202F`). |
| `testOverlay_StringStringIntInt()` | `overlay` con índices invertidos, negativos y fuera de rango. |
| `testRepeat_CharInt`, `testRepeat_StringInt`, `testRepeat_StringStringInt` | Repetición, incluido desbordamiento (`IllegalArgumentException` con `1_073_741_824`). |
| `testEscapeSurrogatePairs()`, `testEscapeSurrogatePairsLang858()`, `testUnescapeSurrogatePairs()` | Regresiones de `StringEscapeUtils` relacionadas con pares subrogados (LANG-858). |
| `testLang623()`, `testLANG666()`, `testLang1593()` | Pruebas de regresión por número de incidencia. `testLang1593` está marcada con `@Disabled` (inconsistencia conocida entre `join(int[], char)` y `join(int[], String)`). |

---

## 6. Convenciones y observaciones de diseño

1. **Nomenclatura:** `test<Método>_<FirmaDeParámetros>`, p. ej. `testCenter_StringIntChar`. Las pruebas de regresión usan `testLANG<nnn>`.
2. **Mensajes de aserción descriptivos:** la mayoría de aserciones incluyen un tercer argumento con la invocación literal probada, lo que facilita el diagnóstico de fallos.
3. **Cobertura Unicode sistemática:** cada operación que manipula índices de `char` tiene al menos una prueba con pares subrogados para evitar la emisión de subrogados huérfanos.
4. **Pruebas parametrizadas** para delimitadores, de modo que se cubren separadores BMP y suplementarios con el mismo cuerpo de prueba.
5. **Aislamiento de `Locale`:** uso de JUnit Pioneer (`@DefaultLocale`, `@WritesDefaultLocale`) y restauración manual con `try/finally` para no contaminar otras pruebas.
6. **Verificación de identidad referencial:** se usa `assertSame` donde el contrato exige devolver la misma instancia si no hay modificación (optimización de memoria).

---

## 7. Ejecución

```bash
# Toda la clase
mvn test -Dtest=StringUtilsTest

# Un método concreto
mvn test -Dtest=StringUtilsTest#testCapitalize
```

> La prueba `testLang1593` no se ejecuta por estar anotada con `@Disabled`.