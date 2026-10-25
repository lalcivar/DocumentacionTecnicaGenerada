# Documentación técnica: `NumberUtilsTest`

**Paquete:** `org.apache.commons.lang3.math`
**Jerarquía:** `AbstractLangTest` → `NumberUtilsTest`
**Visibilidad:** package-private (`class`)
**Framework de pruebas:** JUnit 5 (Jupiter), con uso de `@Test` y `@ParameterizedTest` + `@ValueSource`

---

## 1. Descripción general

`NumberUtilsTest` es la clase de pruebas unitarias de la clase de utilidades `org.apache.commons.lang3.math.NumberUtils` de Apache Commons Lang. No contiene lógica de negocio: su única responsabilidad es **verificar, mediante aserciones, el contrato público de `NumberUtils`**, cubriendo tanto los casos válidos como los casos límite y de error.

Las áreas funcionales cubiertas son:

| Área | Métodos de `NumberUtils` validados |
|---|---|
| Constantes numéricas | `LONG_ZERO`, `INTEGER_ONE`, `BYTE_MINUS_ONE`, `DOUBLE_ZERO`, `FLOAT_ONE`, etc. |
| Creación de objetos `Number` | `createNumber`, `createInteger`, `createLong`, `createFloat`, `createDouble`, `createBigInteger`, `createBigDecimal` |
| Validación de cadenas | `isCreatable`, `isNumber` (obsoleto), `isDigits`, `isParsable` |
| Conversión con valor por defecto | `toInt`, `toLong`, `toShort`, `toByte`, `toFloat`, `toDouble` (y sus sobrecargas con valor por defecto) |
| Redondeo / escala | `toScaledBigDecimal(BigDecimal|Double|Float|String[, int, RoundingMode])` |
| Comparación | `compare(byte|short|int|long)` |
| Mínimos y máximos | `min` / `max` para arrays y para tres argumentos de cada tipo primitivo |
| Estructura de la clase | Constructor público, clase pública no `final` (vía reflexión) |

Además, muchos tests están etiquetados con el identificador del *issue* de JIRA que motivó su creación (LANG-521, LANG-693, LANG-971, LANG-1060, LANG-1645, LANG-1729, LANG-1821, etc.), actuando como **pruebas de regresión documentadas**.

---

## 2. Dependencias relevantes

- `org.junit.jupiter.api.Assertions.*` — aserciones estándar.
- `org.apache.commons.lang3.LangAssertions.assertIllegalArgumentException` / `assertNullPointerException` — helpers propios del proyecto para validar excepciones.
- `java.text.NumberFormat` — usado como "oráculo" de referencia frente a `isParsable`.
- `SystemUtils` / `JavaVersion` — permiten que ciertas expectativas varíen según la versión del JDK (p. ej. el comportamiento de exponentes muy grandes cambió en Java 21).
- `SystemProperties` — habilita la impresión opcional de *stack traces* en los helpers de diagnóstico.

---

## 3. Métodos auxiliares (privados)

Estos métodos no son casos de prueba; dan soporte a los tests reduciendo duplicación.

| Método | Descripción |
|---|---|
| `assertCreateNumberZero(String number, Object zero, Object negativeZero)` | *static.* Comprueba que `createNumber` devuelve el cero esperado para la cadena sin signo, con `+` y con `-` (validando además la distinción entre `0` y `-0`). |
| `checkCreateNumber(String val)` | Invoca `NumberUtils.createNumber(val)` y devuelve `true` si produce un valor no nulo; devuelve `false` si lanza `NumberFormatException`. Encapsula el manejo de la excepción. |
| `compareIsCreatableWithCreateNumber(String val, boolean expected)` | Verifica la **coherencia** entre `isCreatable(val)` y el resultado real de `createNumber(val)`: ambos deben coincidir con `expected`. |
| `compareIsNumberWithCreateNumber(String val, boolean expected)` | Equivalente al anterior pero para el método obsoleto `isNumber(val)`. Anotado con `@SuppressWarnings("deprecation")`. |
| `isApplyNonNull(String s, Function<String,?> function)` | Aplica una función de parseo (`Integer::parseInt`, `Double::parseDouble`, …) y devuelve `true` si retorna un valor no nulo sin excepción. Opcionalmente imprime la traza según la propiedad de sistema `printStackTrace`. |
| `isNumberFormatParsable(String s)` | Indica si `NumberFormat.getInstance().parse(s)` tiene éxito. Sirve de contraste con el parseo estricto de `NumberUtils`. |
| `isNumberIntegerOnlyFormatParsable(String s)` | Igual que el anterior, pero con `setParseIntegerOnly(true)`. |
| `isParsableByte/Short/Integer/Long/Float/Double(String s)` | Familia de helpers que, para una cadena dada, comprueban que `NumberUtils.isParsable` coincide con el resultado del `parseXxx` correspondiente del JDK, y adicionalmente que `NumberFormat` la acepta. Devuelven el valor de `isParsable`. |

---

## 4. Casos de prueba principales

### 4.1 Constantes y estructura de la clase
- **`testConstants()`** — Verifica el tipo en tiempo de ejecución (`assertInstanceOf`) y el valor numérico de todas las constantes `*_ZERO`, `*_ONE` y `*_MINUS_ONE`.
- **`testConstructor()`** — Mediante reflexión comprueba que `NumberUtils` expone exactamente un constructor público y que la clase es pública y **no** `final` (requisito de compatibilidad histórica de Commons Lang).

### 4.2 Creación de números
- **`testCreateBigDecimal`, `testCreateBigInteger`, `testCreateDouble`, `testCreateFloat`, `testCreateInteger`, `testCreateLong`** — Casos positivos y tratamiento de `null` (debe devolver `null`, no lanzar).
- **`testCreateXxxFailure(String)`** (paramétricos) — Para entradas inválidas (cadena vacía, espacios en blanco "exóticos" como `\u00A0`, `\uFEFF`, signos duplicados `--`, `++`, prefijos hexadecimales incompletos `0x`, `#`) se exige `NumberFormatException`, **y además** que el constructor/factoría equivalente del JDK falle igualmente (coherencia con la plataforma).
- **`testCreateNumber()`** — El test más extenso. Valida la heurística de inferencia de tipo de `createNumber`: sufijos `L/l`, `F/f`, `D/d`, notación científica, literales hexadecimales (`0x`, `0X`, `#`) y octales, promoción automática `Integer → Long → BigInteger` y `Float → Double → BigDecimal`, y numerosas regresiones de JIRA.
- **`testCreateNumberMagnitude()`** — Comprueba los puntos exactos de cambio de tipo al superar `Float.MAX_VALUE`, `Double.MAX_VALUE`, y con valores subnormales (`4.9e-325`), así como los límites de 31/32/63/64 bits en hexadecimal y octal.
- **`testCreateNumberZero()`** — Regresión LANG-1646: el cero con todas sus variantes sintácticas (`0.`, `.0`, `0.0`, `0`), con signo, exponente y sufijo de tipo, respetando `-0.0f` / `-0.0d`.
- **`testCreateNumberFailure_1` … `_8`** y **`testInvalidNumber`** — Entradas malformadas que deben lanzar `NumberFormatException` (doble signo, `e` y `E` combinadas, sufijos repetidos `ff`, `DD`, etc.).
- **`testStringCreateNumberEnsureNoPrecisionLoss()`** — Verifica que el tipo elegido por `createNumber` nunca implique pérdida de precisión frente a la cadena original (Float vs Double vs BigDecimal).
- **`testLang300`, `testLang747`, `testLang1087`, `testLang1821`** — Regresiones específicas sobre sufijos `l`, hexadecimales con ceros a la izquierda, ceros con signo y hexadecimales con sufijo de tipo.

### 4.3 Validación de cadenas
- **`testIsCreatable()` / `testIsNumber()`** — Baterías de ~70 entradas cada una que garantizan la invariante: `isCreatable(s)` es `true` si y solo si `createNumber(s)` produce un número. Incluye dependencia de la versión del JDK para exponentes del orden de `Integer.MAX_VALUE`.
- **`testIsNumberLANG971/972/992/1252/1385`, `testLANG971/972/992`, `testLANG1252`** — Regresiones puntuales (octales inválidos como `085`, hexadecimales mayúsculos, signo `+` explícito, la cadena `"L"` sola).
- **`testIsDigits()`** — Solo dígitos decimales; `null` y cadena vacía devuelven `false`.
- **`testIsParsableTrue/False`, `testIsParsableDoubleTrue`, `testIsParsableFloatTrue`** (paramétricos) — Cobertura de literales decimales, científicos y hexadecimales de punto flotante (`0x1.8p3`), sufijos de tipo y formas inválidas.
- **`testIsParsableNaN/Infinity/InfinityNegative/InfinityPositive`, `testIsParsableNull`** — Casos especiales de IEEE-754 y `null`.
- **`testIsParsableFullWidthUnicodeJDK8326627()`** — Documenta la divergencia conocida (LANG-1729 / JDK-8326627) con dígitos Unicode de ancho completo.
- **`testLang1729IsParsableXxx()`** — Comparan `isParsable` con los `parseXxx` del JDK y con `NumberFormat`, incluyendo dígitos de ancho completo y cadenas con espacios.

### 4.4 Conversiones con valor por defecto
- **`testToByteString[I]`, `testToShortString[I]`, `testToIntString[I]`, `testToLongString[L]`, `testToFloatString[F]`, `testStringToDoubleString[D]`, `testBigIntegerToDoubleBigInteger[D]`** — Verifican el valor convertido para entradas válidas y la devolución del valor por defecto (0 o el indicado) para `null`, cadena vacía o texto no numérico. Incluyen regresiones LANG-1060 (ceros y signos a la izquierda) y los extremos `MAX_VALUE` / `MIN_VALUE`.

### 4.5 `toScaledBigDecimal`
- **`testToScaledBigDecimalBigDecimal[IRM]`, `...Double[IRM]`, `...Float[IRM]`, `...String[IRM]`** — Comprueban la escala por defecto (2 decimales) con `RoundingMode.HALF_EVEN`, el uso de escala y modo de redondeo explícitos, la conservación de la escala tras una multiplicación (`toString()`) y que `null` se traduce en `BigDecimal.ZERO`. Se documentan explícitamente las desviaciones esperadas por error de redondeo en `Float`.

### 4.6 Comparación, mínimos y máximos
- **`compareByte`, `compareShort`, `compareInt`, `compareLong`** — Signo del resultado de `NumberUtils.compare`.
- **`testCompareDouble`, `testCompareFloat`** — Matriz exhaustiva del orden total de `Double.compare` / `Float.compare` sobre `NaN`, infinitos, `±MAX_VALUE` y `±0.0`; sirven como contraste del comportamiento del JDK frente a la antigua implementación de Lang.
- **`testMaxXxx` / `testMinXxx`** (`byte`, `short`, `int`, `long`, `float`, `double`) — Variantes con arrays de longitud 1, 2 y 5, y con tres argumentos (`testMaximumXxx` / `testMinimumXxx`).
- **`testMaxXxx_emptyArray` / `_nullArray`** — Exigen `IllegalArgumentException` para arrays vacíos y `NullPointerException` para arrays `null`, usando los helpers de `LangAssertions`.
- **`testLang381()`** — Propagación de `NaN` en `min`/`max`.
- **`testMinMaxSignedZero()`** — Comprueba a nivel de bits (`doubleToRawLongBits` / `floatToRawIntBits`) que las sobrecargas varargs y de tres argumentos coinciden con `Math.min`/`Math.max` en el signo del cero.

---

## 5. Notas de diseño y buenas prácticas observadas

1. **Verificación cruzada con el JDK.** Casi todos los tests negativos no solo exigen la excepción de `NumberUtils`, sino que el método equivalente del JDK (`new BigDecimal(String)`, `Double.valueOf`, …) falle igualmente. Esto evita que la utilidad divergía de la semántica de la plataforma.
2. **Invariantes en lugar de valores aislados.** `compareIsCreatableWithCreateNumber` y `compareIsNumberWithCreateNumber` codifican la invariante `isCreatable(s) ⇔ createNumber(s) != null`, que es más robusta que comprobar cada método por separado.
3. **Pruebas paramétricas.** Las entradas inválidas se agrupan en `@ValueSource`, lo que produce un informe de fallos por entrada concreta y facilita la ampliación.
4. **Trazabilidad con JIRA.** Los comentarios `// LANG-xxxx` y los nombres `testLangNNNN` vinculan cada aserción con el defecto que la originó.
5. **Tolerancia a variaciones del entorno.** `SystemUtils.isJavaVersionAtLeast(JavaVersion.JAVA_21)` se usa como valor esperado para que la suite sea válida en múltiples JDK.
6. **Mensajes de aserción descriptivos**, frecuentemente mediante `Supplier<String>` para evitar construir la cadena cuando la aserción pasa.

---

## 6. Ejecución

```bash
# Toda la clase
mvn test -Dtest=NumberUtilsTest

# Un caso concreto
mvn test -Dtest=NumberUtilsTest#testCreateNumber

# Habilitar la impresión de trazas en los helpers isApplyNonNull
mvn test -Dtest=NumberUtilsTest \
  -Dorg.apache.commons.lang3.math.NumberUtilsTest.printStackTrace=true
```