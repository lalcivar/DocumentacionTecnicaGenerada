# Documentación Técnica - `NumberUtilsTest`

**Paquete:** `org.apache.commons.lang3.math`  
**Clase:** `NumberUtilsTest extends AbstractLangTest`  
**Tipo:** Clase de pruebas unitarias JUnit 5 (Jupiter)  
**Licencia:** Apache License 2.0  
**Dependencia bajo prueba:** `org.apache.commons.lang3.math.NumberUtils`

---

## 1. Resumen Ejecutivo

`NumberUtilsTest` es la suite de pruebas unitarias para la clase utilitaria `NumberUtils` de Apache Commons Lang 3. Su propósito es validar de forma exhaustiva la creación, conversión, validación y comparación de tipos numéricos (`Integer`, `Long`, `BigDecimal`, `BigInteger`, `Float`, `Double`, etc.), incluyendo el manejo de casos límite, formatos hexadecimales, notación científica y valores especiales como `NaN` e `Infinity`. La clase extiende `AbstractLangTest` y utiliza JUnit 5, `ParameterizedTest` y aserciones de `LangAssertions` para garantizar la conformidad con la especificación de `NumberUtils`.

---

## 2. Especificación de Métodos y Constructores

### 2.1 Información de Clase

```java
/**
 * Tests {@link NumberUtils}.
 * @author Apache Software Foundation
 */
class NumberUtilsTest extends AbstractLangTest
```

| Atributo | Valor |
|---|---|
| **Visibilidad** | package-private (default) |
| **Modificadores** | `class` no final |
| **Superclase** | `org.apache.commons.lang3.AbstractLangTest` |
| **Anotaciones de clase** | Ninguna |

### 2.2 Constructor

#### `NumberUtilsTest()`
Constructor por defecto implícito generado por el compilador. No declarado explícitamente en el código fuente.

| Elemento | Descripción |
|---|---|
| **@param** | Ninguno |
| **@return** | Instancia de `NumberUtilsTest` |
| **@throws** | Ninguno |
| **Visibilidad** | package-private (default) |
| **Validación** | Verificada por `testConstructor()` mediante reflexión: `NumberUtils.class.getDeclaredConstructors()` debe tener longitud 1 y ser `public`. |

> **Javadoc:**
> ```java
> /**
>  * Constructor por defecto para NumberUtilsTest.
>  * Crea una nueva instancia de la suite de pruebas.
>  */
> ```

### 2.3 Métodos de Utilidad Privados y de Soporte

#### `assertCreateNumberZero()`
```java
private static void assertCreateNumberZero(final String number, final Object zero, final Object negativeZero)
```

| @param | Tipo | Requerido | Descripción |
|---|---|---|---|
| `number` | `String` | Sí | Representación base del cero (ej: `"0."`, `".0"`, `"0.0"`) sin signo. |
| `zero` | `Object` | Sí | Valor esperado para `NumberUtils.createNumber(number)` y `+"+number"`. Típicamente `Float(0)` o `Double(0)`. |
| `negativeZero` | `Object` | Sí | Valor esperado para `NumberUtils.createNumber("-"+number)`. Típicamente `Float(-0.0)` o `Double(-0.0)`. |

| @return | `void` |
|---|---|
| **@throws** | `AssertionError` si `NumberUtils.createNumber` no retorna el valor esperado. |
| **Visibilidad** | `private static` |

#### `checkCreateNumber()`
```java
private boolean checkCreateNumber(final String val)
```

| @param | Tipo | Requerido | Descripción |
|---|---|---|---|
| `val` | `String` | Sí | Cadena a evaluar con `NumberUtils.createNumber(val)`. Puede ser `null`. |

| @return | `boolean` - `true` si `createNumber` retorna no-nulo sin lanzar excepción, `false` si lanza `NumberFormatException`. |
|---|---|
| **@throws** | Ninguno (captura `NumberFormatException` internamente). |

#### `compareIsCreatableWithCreateNumber()`
```java
private void compareIsCreatableWithCreateNumber(final String val, final boolean expected)
```

| @param | Tipo | Requerido | Descripción |
|---|---|---|---|
| `val` | `String` | No (acepta `null`) | Valor a validar. |
| `expected` | `boolean` | Sí | Resultado esperado tanto para `NumberUtils.isCreatable(val)` como para `checkCreateNumber(val)`. |

| @return | `void` |
|---|---|
| **@throws** | `AssertionError` si `isCreatable` y `createNumber` no coinciden con `expected`. |

#### `compareIsNumberWithCreateNumber()`
```java
@SuppressWarnings("deprecation")
private void compareIsNumberWithCreateNumber(final String val, final boolean expected)
```

| @param | Tipo | Requerido | Descripción |
|---|---|---|---|
| `val` | `String` | No | Valor a validar (método deprecado `isNumber`). |
| `expected` | `boolean` | Sí | Resultado esperado. |

| @return | `void` |
|---|---|
| **@throws** | `AssertionError` |

#### `isApplyNonNull()`
```java
private boolean isApplyNonNull(final String s, final Function<String, ?> function)
```

| @param | Tipo | Requerido | Descripción |
|---|---|---|---|
| `s` | `String` | Sí | Cadena de entrada. |
| `function` | `Function<String, ?>` | Sí | Función de parseo (ej: `Byte::parseByte`, `Double::parseDouble`). |

| @return | `boolean` - `true` si `function.apply(s)` retorna no-nulo, `false` si lanza `Exception`. |
|---|---|
| **@throws** | Ninguno (captura `Exception`). Efecto secundario: imprime stacktrace si `s` no contiene espacios y `SystemProperties.getBoolean(..., "printStackTrace")` es `true`. |

#### `isNumberFormatParsable()`
```java
private boolean isNumberFormatParsable(final String s)
```

| @param | Tipo | Requerido | Descripción |
|---|---|---|---|
| `s` | `String` | Sí | Cadena a parsear con `NumberFormat.getInstance().parse(s)`. |

| @return | `boolean` - `true` si el parseo es exitoso, `false` si lanza `ParseException`. |
|---|---|
| **@throws** | Ninguno |

#### `isNumberIntegerOnlyFormatParsable()`
```java
private boolean isNumberIntegerOnlyFormatParsable(final String s)
```

| @param | Tipo | Requerido | Descripción |
|---|---|---|---|
| `s` | `String` | Sí | Cadena a parsear con `NumberFormat` en modo `setParseIntegerOnly(true)`. |

| @return | `boolean` |
|---|---|
| **@throws** | Ninguno |

#### `isParsableByte()`, `isParsableDouble()`, `isParsableFloat()`, `isParsableInteger()`, `isParsableLong()`, `isParsableShort()`
Seis métodos con firma idéntica, varían solo el tipo de parseo interno:

```java
private boolean isParsableByte(final String s)
private boolean isParsableDouble(final String s)
private boolean isParsableFloat(final String s)
private boolean isParsableInteger(final String s)
private boolean isParsableLong(final String s)
private boolean isParsableShort(final String s)
```

| @param | Tipo | Requerido | Descripción |
|---|---|---|---|
| `s` | `String` | Sí | Cadena a validar. |

| @return | `boolean` - valor de `NumberUtils.isParsable(s)`. |
|---|---|
| **@throws** | `AssertionError` si `isParsable` no coincide con `isApplyNonNull` o con `NumberFormat` parsable. Valida consistencia entre `NumberUtils.isParsable` y `Byte::parseByte` / `Double::parseDouble` / etc. |

---

### 2.4 Métodos de Prueba Públicos (`@Test` / `@ParameterizedTest`)

> Todos los métodos siguientes son `void`, no retornan valor y no declaran `throws` explícito. Lanzan `AssertionError` si la aserción falla. Los anotados con `@ParameterizedTest` reciben inyección de parámetros vía `@ValueSource`.

| Método | Visibilidad | @param | @return | @throws | Descripción |
|---|---|---|---|---|---|
| `compareByte()` | `public` | - | `void` | `AssertionError` | Valida `NumberUtils.compare(byte, byte)` con valores -3, 0, 113, 123, 32. |
| `compareInt()` | `public` | - | `void` | `AssertionError` | Valida `NumberUtils.compare(int, int)`. |
| `compareLong()` | `public` | - | `void` | `AssertionError` | Valida `NumberUtils.compare(long, long)`. |
| `compareShort()` | `public` | - | `void` | `AssertionError` | Valida `NumberUtils.compare(short, short)`. |
| `testBigIntegerToDoubleBigInteger()` | package-private | - | `void` | `AssertionError` | Test para `NumberUtils.toDouble(BigDecimal)` con `null` y `8.5d`. |
| `testBigIntegerToDoubleBigIntegerD()` | package-private | - | `void` | `AssertionError` | Test para `NumberUtils.toDouble(BigDecimal, double)` con valor por defecto `1.1d`. |
| `testCompareDouble()` | package-private | - | `void` | `AssertionError` | Valida `Double.compare` exhaustivo con `NaN`, `POSITIVE_INFINITY`, `MAX_VALUE`, `0.0`, `-0.0`. |
| `testCompareFloat()` | package-private | - | `void` | `AssertionError` | Equivalente a `testCompareDouble` para `Float`. |
| `testConstants()` | package-private | - | `void` | `AssertionError` | Verifica constantes `LONG_ZERO`, `INTEGER_ONE`, `BYTE_MINUS_ONE`, `DOUBLE_ZERO`, `FLOAT_ONE`, etc. tipo y valor. |
| `testConstructor()` | package-private | - | `void` | `AssertionError` | Verifica que `NumberUtils` tiene 1 constructor público y no es `final`. |
| `testCreateBigDecimal()` | package-private | - | `void` | `AssertionError` | Valida `createBigDecimal(String)` con `"1234.5"` y `null`. |
| `testCreateBigDecimalFailure(String str)` | package-private | `str:String` Sí | `void` | `AssertionError`, `NumberFormatException` esperado | `@ParameterizedTest` con strings vacíos, espacios, signos solos. Debe lanzar `NumberFormatException`. |
| `testCreateBigInteger()` | package-private | - | `void` | `AssertionError` | Valida `createBigInteger` con decimales, hex `0xff`, `#ff`, octal `0377`, signo `+`. |
| `testCreateBigIntegerFailure(String str)` | package-private | `str:String` Sí | `void` | `AssertionError` | `@ParameterizedTest` con `""`, `"#"`, `"0x"`, `"--1"`, `"-0x-1"`. |
| `testCreateDouble()` | package-private | - | `void` | `AssertionError` | Valida `createDouble`. |
| `testCreateDoubleFailure(String str)` | package-private | `str:String` Sí | `void` | `AssertionError` | `@ParameterizedTest` strings vacíos/blancos. |
| `testCreateFloat()` | package-private | - | `void` | `AssertionError` | Valida `createFloat`. |
| `testCreateFloatFailure(String str)` | package-private | `str:String` Sí | `void` | `AssertionError` | Idem. |
| `testCreateInteger()` | package-private | - | `void` | `AssertionError` | Valida `createInteger` con `"+0xF"`. |
| `testCreateIntegerFailure(String str)` | package-private | `str:String` Sí | `void` | `AssertionError` | Idem. |
| `testCreateLong()` | package-private | - | `void` | `AssertionError` | Valida `createLong` con `"+0xFFFFFFFF"`. |
| `testCreateLongFailure(String str)` | package-private | `str:String` Sí | `void` | `AssertionError` | Idem. |
| `testCreateNumber()` | package-private | - | `void` | `AssertionError` | Suite principal de `createNumber`: decimales, sufijos `D/F/L`, hex, exponentes, `BigInteger`/`BigDecimal`. |
| `testCreateNumberFailure_1()` | package-private | - | `void` | `AssertionError` | Debe lanzar `NumberFormatException` para `"--1.1E-700F"`. |
| `testCreateNumberFailure_2()` | package-private | - | `void` | `AssertionError` | Para `"-1.1E+0-7e00"` (doble exponente). |
| `testCreateNumberFailure_3()` | package-private | - | `void` | `AssertionError` | Para `"-11E+0-7e00"`. |
| `testCreateNumberFailure_4()` | package-private | - | `void` | `AssertionError` | Para `"1eE+00001"`. |
| `testCreateNumberFailure_5()` | package-private | - | `void` | `AssertionError` | Para `"1234.5ff"` (LANG-1205). |
| `testCreateNumberFailure_6()` | package-private | - | `void` | `AssertionError` | Para `"1234.5FF"`. |
| `testCreateNumberFailure_7()` | package-private | - | `void` | `AssertionError` | Para `"1234.5dd"`. |
| `testCreateNumberFailure_8()` | package-private | - | `void` | `AssertionError` | Para `"1234.5DD"`. |
| `testCreateNumberMagnitude()` | package-private | - | `void` | `AssertionError` | Valida cambio de tipo por magnitud: `Float.MAX_VALUE` -> `Double` -> `BigDecimal`, hex y octal límites. |
| `testCreateNumberZero()` | package-private | - | `void` | `AssertionError` | LANG-1646: ceros con exponente y sufijos `F/D`, verifica `0.`, `.0`, `0e1`. Usa `assertCreateNumberZero()`. |
| `testInvalidNumber()` | package-private | - | `void` | `AssertionError` | Verifica `NumberFormatException` para `"E123e.3"` y `"-"`. |
| `testIsCreatable()` | package-private | - | `void` | `AssertionError` | Valida `isCreatable` vs `createNumber` con 60+ casos incluyendo `null`, `"--2.3"`, `"0xGF"`, `"1E2147483647"`. |
| `testIsDigits()` | package-private | - | `void` | `AssertionError` | Valida `isDigits` con `null`, `""`, `"12345"`, `"1ab"`. |
| `testIsNumber()` | package-private | - | `void` | `AssertionError` | Valida método deprecado `isNumber` (LANG-521, LANG-1645, LANG-1646). |
| `testIsNumberLANG1252()` | package-private | - | `void` | `AssertionError` | Valida `"+2"`, `"+2.0"` para `isNumber`. |
| `testIsNumberLANG1385()` | package-private | - | `void` | `AssertionError` | Valida `"L"` -> `false`. |
| `testIsNumberLANG971()` | package-private | - | `void` | `AssertionError` | Valida octales `"0085"` `false`, `"07"` `true`. |
| `testIsNumberLANG972()` | package-private | - | `void` | `AssertionError` | Valida `"0xABCD"` `true`. |
| `testIsNumberLANG992()` | package-private | - | `void` | `AssertionError` | Valida `"0.0"`, `"0.4790"`. |
| `testIsParsableDoubleTrue(String input)` | package-private | `input:String` Sí | `void` | `AssertionError` | `@ParameterizedTest` con literales decimales, exponenciales y hex float `0x1.8p3`. Verifica `Double.parseDouble` y `isParsable`. |
| `testIsParsableFalse(String input)` | package-private | `input:String` Sí | `void` | `AssertionError` | `@ParameterizedTest` con `""`, `"0xC1AB"`, `"64, 2"`, `"e5"`, `"f"`. Debe ser `false`. |
| `testIsParsableFloatTrue(String input)` | package-private | `input:String` Sí | `void` | `AssertionError` | `@ParameterizedTest` con sufijo `f` y hex float. |
| `testIsParsableFullWidthUnicodeJDK8326627()` | package-private | - | `void` | `AssertionError` | LANG-1729 / JDK-8326627: dígitos fullwidth `\uFF10\uFF11\uFF12` deben ser `isParsable=true` pero `Double.parseDouble` lanza `NumberFormatException`. |
| `testIsParsableInfinity()` | package-private | - | `void` | `AssertionError` | Valida `"Infinity"` -> `isParsable true`. |
| `testIsParsableInfinityNegative()` | package-private | - | `void` | `AssertionError` | Valida `"-Infinity"`. |
| `testIsParsableInfinityPositive()` | package-private | - | `void` | `AssertionError` | Valida `"+Infinity"`. |
| `testIsParsableNaN()` | package-private | - | `void` | `AssertionError` | Valida `"NaN"`. |
| `testIsParsableNull()` | package-private | - | `void` | `AssertionError` | Valida `isParsable(null)` -> `false`. |
| `testIsParsableTrue(String input)` | package-private | `input:String` Sí | `void` | `AssertionError` | `@ParameterizedTest` con 30+ casos válidos incluyendo `"NaN"`, `"64."`, `"1.2e-5f"`, `"2f"`. |
| `testLang1087()` | package-private | - | `void` | `AssertionError` | Valida `createNumber("0.0")` con signos `+`/`-` retorna `Float`. |
| `testLANG1252()` | package-private | - | `void` | `AssertionError` | Valida `isCreatable` con `"+2"`. |
| `testLang1729IsParsableByte()` | package-private | - | `void` | `AssertionError` | Valida `isParsableByte` con `"1"`, `"1 2 3"`, `"１２３"`. |
| `testLang1729IsParsableDouble()` | package-private | - | `void` | `AssertionError` | Valida `isParsableDouble` con `"1."`, `"1.f"`, `"1 2 3"`. |
| `testLang1729IsParsableFloat()` | package-private | - | `void` | `AssertionError` | Idem para Float. |
| `testLang1729IsParsableInteger()` | package-private | - | `void` | `AssertionError` | Idem para Integer. |
| `testLang1729IsParsableLong()` | package-private | - | `void` | `AssertionError` | Idem para Long. |
| `testLang1729IsParsableShort()` | package-private | - | `void` | `AssertionError` | Idem para Short. |
| `testLang1821()` | package-private | - | `void` | `AssertionError` | Valida hex con sufijo `L` y límites `Integer.MAX_VALUE`/`Long.MAX_VALUE`. |
| `testLang300()` | package-private | - | `void` | `AssertionError` | Valida `createNumber("-1l")`, `"01l"`, `"1l"` no lanza excepción. |
| `testLang381()` | package-private | - | `void` | `AssertionError` | Valida `min`/`max` con `NaN` retorna `NaN` para `double[]` y `float[]`. |
| `testLang747()` | package-private | - | `void` | `AssertionError` | Valida hex límites `0x8000` -> `Integer`, `0x80000000` -> `Long`, `0x8000000000000000` -> `BigInteger`. |
| `testLANG971()` | package-private | - | `void` | `AssertionError` | Duplicado de `testIsNumberLANG971` pero para `isCreatable`. |
| `testLANG972()` | package-private | - | `void` | `AssertionError` | Duplicado para `isCreatable` con hex. |
| `testLANG992()` | package-private | - | `void` | `AssertionError` | Duplicado para `isCreatable` con `"0.0"`. |
| `testMaxByte()` | package-private | - | `void` | `AssertionError` | Valida `NumberUtils.max(byte[])` con 1,2,5 elementos. |
| `testMaxByte_emptyArray()` | package-private | - | `void` | `AssertionError` | Verifica `IllegalArgumentException` para array vacío vía `assertIllegalArgumentException`. |
| `testMaxByte_nullArray()` | package-private | - | `void` | `AssertionError` | Verifica `NullPointerException` para `null`. |
| `testMaxDouble()` | package-private | - | `void` | `AssertionError` | Valida `max(double[])`. |
| `testMaxDouble_emptyArray()` | package-private | - | `void` | `AssertionError` | `IllegalArgumentException`. |
| `testMaxDouble_nullArray()` | package-private | - | `void` | `AssertionError` | `NullPointerException`. |
| `testMaxFloat()` | package-private | - | `void` | `AssertionError` | Valida `max(float[])`. |
| `testMaxFloat_emptyArray()` | package-private | - | `void` | `AssertionError` | `IllegalArgumentException`. |
| `testMaxFloat_nullArray()` | package-private | - | `void` | `AssertionError` | `NullPointerException`. |
| `testMaximumByte()` | package-private | - | `void` | `AssertionError` | Valida `max(byte,byte,byte)` 3 args. |
| `testMaximumDouble()` | package-private | - | `void` | `AssertionError` | Valida `max(double,double,double)`. |
| `testMaximumFloat()` | package-private | - | `void` | `AssertionError` | Valida `max(float,float,float)`. |
| `testMaximumInt()` | package-private | - | `void` | `AssertionError` | Valida `max(int,int,int)`. |
| `testMaximumLong()` | package-private | - | `void` | `AssertionError` | Valida `max(long,long,long)`. |
| `testMaximumShort()` | package-private | - | `void` | `AssertionError` | Valida `max(short,short,short)`. |
| `testMaxInt()` | package-private | - | `void` | `AssertionError` | Valida `max(int[])`. |
| `testMaxInt_emptyArray()` | package-private | - | `void` | `AssertionError` | `IllegalArgumentException`. |
| `testMaxInt_nullArray()` | package-private | - | `void` | `AssertionError` | `NullPointerException`. |
| `testMaxLong()` | package-private | - | `void` | `AssertionError` | Valida `max(long[])`. |
| `testMaxLong_emptyArray()` | package-private | - | `void` | `AssertionError` | `IllegalArgumentException`. |
| `testMaxLong_nullArray()` | package-private | - | `void` | `AssertionError` | `NullPointerException`. |
| `testMaxShort()` | package-private | - | `void` | `AssertionError` | Valida `max(short[])`. |
| `testMaxShort_emptyArray()` | package-private | - | `void` | `AssertionError` | `IllegalArgumentException`. |
| `testMaxShort_nullArray()` | package-private | - | `void` | `AssertionError` | `NullPointerException`. |
| `testMinByte()` | package-private | - | `void` | `AssertionError` | Valida `min(byte[])`. |
| `testMinByte_emptyArray()` | package-private | - | `void` | `AssertionError` | `IllegalArgumentException`. |
| `testMinByte_nullArray()` | package-private | - | `void` | `AssertionError` | `NullPointerException`. |
| `testMinDouble()` | package-private | - | `void` | `AssertionError` | Valida `min(double[])`. |
| `testMinDouble_emptyArray()` | package-private | - | `void` | `AssertionError` | `IllegalArgumentException`. |
| `testMinDouble_nullArray()` | package-private | - | `void` | `AssertionError` | `NullPointerException`. |
| `testMinFloat()` | package-private | - | `void` | `AssertionError` | Valida `min(float[])` incluyendo `NaN`. |
| `testMinFloat_emptyArray()` | package-private | - | `void` | `AssertionError` | `IllegalArgumentException`. |
| `testMinFloat_nullArray()` | package-private | - | `void` | `AssertionError` | `NullPointerException`. |
| `testMinimumByte()` | package-private | - | `void` | `AssertionError` | Valida `min(byte,byte,byte)`. |
| `testMinimumDouble()` | package-private | - | `void` | `AssertionError` | Valida `min(double,double,double)`. |
| `testMinimumFloat()` | package-private | - | `void` | `AssertionError` | Valida `min(float,float,float)`. |
| `testMinimumInt()` | package-private | - | `void` | `AssertionError` | Valida `min(int,int,int)`. |
| `testMinimumLong()` | package-private | - | `void` | `AssertionError` | Valida `min(long,long,long)`. |
| `testMinimumShort()` | package-private | - | `void` | `AssertionError` | Valida `min(short,short,short)`. |
| `testMinInt()` | package-private | - | `void` | `AssertionError` | Valida `min(int[])`. |
| `testMinInt_emptyArray()` | package-private | - | `void` | `AssertionError` | `IllegalArgumentException`. |
| `testMinInt_nullArray()` | package-private | - | `void` | `AssertionError` | `NullPointerException`. |
| `testMinLong()` | package-private | - | `void` | `AssertionError` | Valida `min(long[])`. |
| `testMinLong_emptyArray()` | package-private | - | `void` | `AssertionError` | `IllegalArgumentException`. |
| `testMinLong_nullArray()` | package-private | - | `void` | `AssertionError` | `NullPointerException`. |
| `testMinMaxSignedZero()` | package-private | - | `void` | `AssertionError` | Valida signo de cero: `max(-0.0,0.0)` debe ser `0.0` y `min(-0.0,0.0)` debe ser `-0.0` vía `doubleToRawLongBits`. |
| `testMinShort()` | package-private | - | `void` | `AssertionError` | Valida `min(short[])`. |
| `testMinShort_emptyArray()` | package-private | - | `void` | `AssertionError` | `IllegalArgumentException`. |
| `testMinShort_nullArray()` | package-private | - | `void` | `AssertionError` | `NullPointerException`. |
| `testStringCreateNumberEnsureNoPrecisionLoss()` | package-private | - | `void` | `AssertionError` | Valida que `createNumber` elige `Float`/`Double`/`BigDecimal` sin pérdida de precisión (LANG-1060, LANG-1444, LANG-1613). |
| `testStringToDoubleString()` | package-private | - | `void` | `AssertionError` | Valida `toDouble(String)` con `"-1.2345"`, `"abc"`->`0.0`, `null`->`0.0`. |
| `testStringToDoubleStringD()` | package-private | - | `void` | `AssertionError` | Valida `toDouble(String, double)` con valor por defecto. |
| `testToByteString()` | package-private | - | `void` | `AssertionError` | Valida `toByte(String)` con `"123"`, `"abc"`->`0`. |
| `testToByteStringI()` | package-private | - | `void` | `AssertionError` | Valida `toByte(String, byte)` con default `5`. |
| `testToFloatString()` | package-private | - | `void` | `AssertionError` | Valida `toFloat(String)`. |
| `testToFloatStringF()` | package-private | - | `void` | `AssertionError` | Valida `toFloat(String, float)`. |
| `testToIntString()` | package-private | - | `void` | `AssertionError` | Valida `toInt(String)`. |
| `testToIntStringI()` | package-private | - | `void` | `AssertionError` | Valida `toInt(String, int)`. |
| `testToLongString()` | package-private | - | `void` | `AssertionError` | Valida `toLong(String)` con `"1L"`->`0`. |
| `testToLongStringL()` | package-private | - | `void` | `AssertionError` | Valida `toLong(String, long)`. |
| `testToScaledBigDecimalBigDecimal()` | package-private | - | `void` | `AssertionError` | Valida `toScaledBigDecimal(BigDecimal)` con escala 2 y `HALF_EVEN`, `null`->`ZERO`. |
| `testToScaledBigDecimalBigDecimalIRM()` | package-private | - | `void` | `AssertionError` | Valida `toScaledBigDecimal(BigDecimal,int,RoundingMode)` con `CEILING`, `FLOOR`, `HALF_UP`. |
| `testToScaledBigDecimalDouble()` | package-private | - | `void` | `AssertionError` | Valida `toScaledBigDecimal(Double)`. |
| `testToScaledBigDecimalDoubleIRM()` | package-private | - | `void` | `AssertionError` | Valida `toScaledBigDecimal(Double,int,RoundingMode)`. |
| `testToScaledBigDecimalFloat()` | package-private | - | `void` | `AssertionError` | Valida `toScaledBigDecimal(Float)` (nota error de redondeo con `23.515f`). |
| `testToScaledBigDecimalFloatIRM()` | package-private | - | `void` | `AssertionError` | Valida `toScaledBigDecimal(Float,int,RoundingMode)`. |
| `testToScaledBigDecimalString()` | package-private | - | `void` | `AssertionError` | Valida `toScaledBigDecimal(String)`. |
| `testToScaledBigDecimalStringIRM()` | package-private | - | `void` | `AssertionError` | Valida `toScaledBigDecimal(String,int,RoundingMode)`. |
| `testToShortString()` | package-private | - | `void` | `AssertionError` | Valida `toShort(String)`. |
| `testToShortStringI()` | package-private | - | `void` | `AssertionError` | Valida `toShort(String, short)`. |

### 2.5 Métodos y Clases Anónimas / Lambdas Internas

El código no declara clases internas con nombre, pero utiliza extensivamente lambdas y referencias a métodos como implementaciones anónimas de interfaces funcionales. Se documentan por completitud:

| Lambda / Referencia | Tipo Funcional | Ubicación | Propósito |
|---|---|---|---|
| `() -> "Input: " + number` | `Supplier<String>` | `assertCreateNumberZero()` | Mensaje lazy para `assertEquals`. |
| `() -> false` | `BooleanSupplier` | `isApplyNonNull()` | Supplier para `SystemProperties.getBoolean(..., "printStackTrace", () -> false)` |
| `Byte::parseByte`, `Double::parseDouble`, `Float::parseFloat`, `Integer::parseInt`, `Long::parseLong`, `Short::parseShort` | `Function<String, ?>` | `isParsableByte()` etc. | Pasadas a `isApplyNonNull()` para validar parsabilidad. |
| `() -> NumberUtils.createBigDecimal(str)` | `Executable` | `testCreateBigDecimalFailure()` | Lambda para `assertThrows(NumberFormatException.class, ...)`. |
| `() -> new BigDecimal(str)` | `Executable` | `testCreateBigDecimalFailure()` | Comparación con JDK. |
| `() -> NumberUtils.createBigInteger(str)` / `() -> new BigInteger(str)` | `Executable` | `testCreateBigIntegerFailure()` | Idem. |
| `() -> NumberUtils.createDouble(str)` / `() -> Double.valueOf(str)` | `Executable` | `testCreateDoubleFailure()` | Idem. |
| `() -> NumberUtils.createFloat(str)` / `() -> Float.valueOf(str)` | `Executable` | `testCreateFloatFailure()` | Idem. |
| `() -> NumberUtils.createInteger(str)` / `() -> Integer.valueOf(str)` | `Executable` | `testCreateIntegerFailure()` | Idem. |
| `() -> NumberUtils.createLong(str)` / `() -> Long.valueOf(str)` | `Executable` | `testCreateLongFailure()` | Idem. |
| `() -> NumberUtils.createNumber(...)` | `Executable` | `testCreateNumberFailure_1` a `_8`, `testInvalidNumber()` | Verifica lanzamiento de `NumberFormatException`. |
| `() -> NumberUtils.max((byte[]) null)` etc. | `Executable` | `testMax*_*_nullArray()` / `testMin*_*_nullArray()` | Verifica `NullPointerException` vía `assertNullPointerException`. |
| `NumberUtils::max` / `NumberUtils::min` | `Supplier` / `Executable` | `testMax*_*_emptyArray()` / `testMin*_*_emptyArray()` | Verifica `IllegalArgumentException` vía `assertIllegalArgumentException`. |
| `() -> Double.parseDouble(fullWidth123)` | `Executable` | `testIsParsableFullWidthUnicodeJDK8326627()` | Verifica que JDK lanza `NumberFormatException` con dígitos fullwidth. |

---

## 3. Ejemplo de Uso

El siguiente bloque muestra un caso de uso real de la clase como suite JUnit 5. No se instancia directamente en producción, sino que se ejecuta vía el motor de pruebas.

```java
package org.apache.commons.lang3.math;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertTrue;
import org.junit.jupiter.api.Test;

/**
 * Ejemplo de uso de NumberUtils validado por NumberUtilsTest.
 * Demuestra cómo NumberUtilsTest verifica el comportamiento de NumberUtils.
 */
public class EjemploUsoNumberUtils {

    @Test
    void ejemploCreateNumber() {
        // Caso validado por testCreateNumber() y testIsCreatable()
        assertTrue(NumberUtils.isCreatable("1234.5"));
        assertEquals(Float.valueOf("1234.5"), NumberUtils.createNumber("1234.5"));
        
        // Hexadecimal con signo - validado por testLang747() y testIsCreatable()
        assertEquals(Integer.valueOf(0xFADE), NumberUtils.createNumber("0xFADE"));
        assertEquals(Integer.valueOf(0x8000), NumberUtils.createNumber("0x8000"));

        // Cero con sufijo - validado por testCreateNumberZero()
        assertEquals(Float.valueOf(0.0f), NumberUtils.createNumber("0.0F"));
        assertEquals(Double.valueOf(-0.0d), NumberUtils.createNumber("-0.0D"));
    }

    @Test
    void ejemploIsParsable() {
        // Validado por testIsParsableTrue() y testIsParsableDoubleTrue()
        assertTrue(NumberUtils.isParsable("1.23e10"));
        assertTrue(NumberUtils.isParsable("0x1.8p3")); // hex float
        assertTrue(NumberUtils.isParsable("NaN"));
        assertTrue(NumberUtils.isParsable("-Infinity"));
    }

    @Test
    void ejemploMinMax() {
        // Validado por testMaxInt(), testMinDouble(), testMinMaxSignedZero()
        assertEquals(10, NumberUtils.max(-10, -5, 0, 5, 10));
        assertEquals(-10, NumberUtils.min(-10, -5, 0, 5, 10));
        
        // Manejo de -0.0 vs 0.0 - validado por testMinMaxSignedZero()
        assertEquals(0.0d, NumberUtils.max(-0.0d, 0.0d));
        assertEquals(-0.0d, NumberUtils.min(-0.0d, 0.0d));
    }

    // Ejecución con Maven:
    // mvn -Dtest=org.apache.commons.lang3.math.NumberUtilsTest test
}
```

**Notas de ejecución:**
- Requiere JUnit Jupiter 5.x, `commons-lang3` y `AbstractLangTest` en el classpath.
- Los métodos `isParsable*` internos de `NumberUtilsTest` demuestran cómo validar consistencia entre `NumberUtils.isParsable` y `NumberFormat`/`parseXxx` del JDK.
- Para reproducir la validación de `testConstructor()`, usar reflexión:
  ```java
  Constructor<?>[] cons = NumberUtils.class.getDeclaredConstructors();
  assertEquals(1, cons.length);
  assertTrue(Modifier.isPublic(cons[0].getModifiers()));
  ```

