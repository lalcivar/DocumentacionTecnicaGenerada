# Documentación Técnica — `org.apache.commons.lang3.math.NumberUtilsTest`

---

## 1. Resumen ejecutivo

`NumberUtilsTest` es la suite de pruebas unitarias (JUnit 5 / Jupiter) que valida el contrato funcional completo de la clase de utilidades `org.apache.commons.lang3.math.NumberUtils`. Cubre la creación de números a partir de cadenas (`createNumber`, `createBigDecimal`, `createBigInteger`, `createDouble`, `createFloat`, `createInteger`, `createLong`), los predicados de validación (`isCreatable`, `isNumber`, `isParsable`, `isDigits`), las conversiones tolerantes a fallos (`toInt`, `toLong`, `toByte`, `toShort`, `toFloat`, `toDouble`, `toScaledBigDecimal`), los agregadores `min`/`max` y las utilidades `compare` para tipos primitivos.

La clase extiende `AbstractLangTest` y combina pruebas deterministas (`@Test`) con pruebas de tabla (`@ParameterizedTest` + `@ValueSource`), apoyándose en un conjunto de **métodos auxiliares privados** que establecen invariantes cruzadas: por ejemplo, que `isCreatable(String)` devuelva `true` **si y solo si** `createNumber(String)` produce un número válido, o que `isParsable(String)` concuerde con los parsers nativos del JDK (`Integer::parseInt`, `Double::parseDouble`, etc.) y con `java.text.NumberFormat`.

Adicionalmente, actúa como documentación ejecutable de incidencias históricas del proyecto (LANG-300, LANG-381, LANG-521, LANG-638, LANG-664, LANG-693, LANG-747, LANG-971/972/992, LANG-1018, LANG-1060, LANG-1087, LANG-1187, LANG-1205, LANG-1215, LANG-1252, LANG-1385, LANG-1444, LANG-1613, LANG-1645, LANG-1646, LANG-1729, LANG-1821) y de la discrepancia del JDK `JDK-8326627`.

---

## 2. Vista general de la estructura

| Atributo | Valor |
|---|---|
| Paquete | `org.apache.commons.lang3.math` |
| Nombre | `NumberUtilsTest` |
| Visibilidad | *package-private* (`class NumberUtilsTest`) |
| Superclase | `org.apache.commons.lang3.AbstractLangTest` |
| Clase bajo prueba (SUT) | `org.apache.commons.lang3.math.NumberUtils` |
| Constructores declarados | Ninguno explícito → **constructor por defecto implícito** `NumberUtilsTest()` |
| Clases internas / anónimas | Ninguna clase anónima declarada; sí existen expresiones lambda y referencias a método (ver §3.4) |
| Licencia | Apache License 2.0 (ASF) |

### Javadoc de clase propuesto

```java
/**
 * Tests {@link NumberUtils}.
 *
 * <p>Valida creación de números, predicados de validación, conversiones
 * tolerantes a fallos, agregadores {@code min}/{@code max} y {@code compare}.</p>
 *
 * @see org.apache.commons.lang3.math.NumberUtils
 * @author Apache Commons Lang (la etiqueta {@code @author} NO está presente en el
 *         código fuente analizado; la convención ASF omite la autoría individual)
 */
```

> **Nota sobre `@author`:** el código fuente provisto **no declara** la etiqueta `@author` en ningún elemento. Se documenta explícitamente esta ausencia para no introducir metadatos inventados.

### Convención de excepciones aplicable a toda la clase

| Excepción | Origen | Aplicabilidad |
|---|---|---|
| `AssertionError` (incluidas las subclases de fallo lanzadas por JUnit Jupiter) | `assertEquals`, `assertTrue`, `assertFalse`, `assertNull`, `assertNotNull`, `assertInstanceOf`, `assertThrows`, `assertIllegalArgumentException`, `assertNullPointerException` | **Todos** los métodos `@Test`, `@ParameterizedTest` y los auxiliares que invocan aserciones |
| `NumberFormatException` | Propagada desde el SUT cuando una aserción no la captura | Auxiliares `assertCreateNumberZero()`; verificada (no propagada) en los métodos `*Failure*` mediante `assertThrows` |
| `NullPointerException` | `String.matches`, `NumberFormat.parse`, o el propio SUT | Auxiliares que reciben `null` (ver tablas específicas) |

---

## 3. Especificación de métodos y constructores

### 3.1 Constructor

#### `NumberUtilsTest()`

Constructor por defecto implícito generado por el compilador. No se declara en el código fuente; invoca implícitamente `super()` (`AbstractLangTest()`). Es el constructor que JUnit Jupiter utiliza por reflexión para instanciar la clase una vez por método de prueba (ciclo de vida `PER_METHOD`).

| @param | Tipo | Requerido | Descripción |
|---|---|---|---|
| — | — | — | Sin parámetros |

| @return | Descripción |
|---|---|
| `NumberUtilsTest` | Nueva instancia de la suite de pruebas |

| @throws | Condición |
|---|---|
| — | No declara ni propaga excepciones de forma explícita |

---

### 3.2 Métodos auxiliares (infraestructura de pruebas)

#### `assertCreateNumberZero()`

```java
private static void assertCreateNumberZero(final String number, final Object zero, final Object negativeZero)
```

Verifica el soporte de "ceros tipados" (LANG-1646): comprueba que `NumberUtils.createNumber(number)` y `NumberUtils.createNumber("+" + number)` sean iguales a `zero`, y que `NumberUtils.createNumber("-" + number)` sea igual a `negativeZero`. Los mensajes de fallo se construyen de forma diferida mediante `Supplier<String>` (lambdas `() -> "Input: " + ...`).

| @param | Tipo | Requerido | Descripción |
|---|---|---|---|
| `number` | `String` | Sí | Literal numérico base sin signo (p. ej. `"0."`, `".0"`, `"0e13D"`) |
| `zero` | `Object` | Sí | Valor esperado (cero positivo) con el tipo concreto esperado (`Float`/`Double`) |
| `negativeZero` | `Object` | Sí | Valor esperado para el literal con signo negativo (`-0F` / `-0D`) |

| @return | Descripción |
|---|---|
| `void` | No retorna valor |

| @throws | Condición |
|---|---|
| `AssertionError` | Alguna de las tres igualdades no se cumple |
| `NumberFormatException` | `NumberUtils.createNumber` rechaza alguna de las tres variantes (incluye el caso `number == null`, que produce la cadena `"+null"`) |

---

#### `checkCreateNumber()`

```java
private boolean checkCreateNumber(final String val)
```

Envoltorio defensivo sobre `NumberUtils.createNumber(String)`: normaliza el resultado a un `boolean` que indica "se pudo construir un número". Captura `NumberFormatException` y la traduce a `false`.

| @param | Tipo | Requerido | Descripción |
|---|---|---|---|
| `val` | `String` | No (admite `null`) | Cadena candidata a número |

| @return | Descripción |
|---|---|
| `boolean` | `true` si `NumberUtils.createNumber(val)` devolvió una instancia no nula; `false` si devolvió `null` (caso `val == null`) o si lanzó `NumberFormatException` |

| @throws | Condición |
|---|---|
| `NumberFormatException` | No se propaga: es capturada internamente |

---

#### `compareIsCreatableWithCreateNumber()`

```java
private void compareIsCreatableWithCreateNumber(final String val, final boolean expected)
```

Invariante central de la suite: `NumberUtils.isCreatable(val)` y `checkCreateNumber(val)` deben coincidir **entre sí** y con el valor esperado. El mensaje de fallo incluye la entrada y ambos resultados obtenidos.

| @param | Tipo | Requerido | Descripción |
|---|---|---|---|
| `val` | `String` | No (admite `null`) | Cadena a evaluar |
| `expected` | `boolean` | Sí | Resultado esperado para ambos mecanismos |

| @return | Descripción |
|---|---|
| `void` | No retorna valor |

| @throws | Condición |
|---|---|
| `AssertionError` | `isCreatable(val) != expected` o `checkCreateNumber(val) != expected` |

---

#### `compareIsNumberWithCreateNumber()`

```java
@SuppressWarnings("deprecation")
private void compareIsNumberWithCreateNumber(final String val, final boolean expected)
```

Variante de `compareIsCreatableWithCreateNumber()` que ejercita el método **obsoleto** `NumberUtils.isNumber(String)`; de ahí la anotación `@SuppressWarnings("deprecation")`. Garantiza la retrocompatibilidad del predicado deprecado frente a `createNumber`.

| @param | Tipo | Requerido | Descripción |
|---|---|---|---|
| `val` | `String` | No (admite `null`) | Cadena a evaluar |
| `expected` | `boolean` | Sí | Resultado esperado para `isNumber` y para `createNumber` |

| @return | Descripción |
|---|---|
| `void` | No retorna valor |

| @throws | Condición |
|---|---|
| `AssertionError` | `isNumber(val) != expected` o `checkCreateNumber(val) != expected` |

---

#### `isApplyNonNull()`

```java
private boolean isApplyNonNull(final String s, final Function<String, ?> function)
```

Aplica una función de parseo del JDK (recibida como `Function`, típicamente una referencia a método como `Integer::parseInt`) sobre `s` y determina si el parseo fue exitoso y no nulo. En la rama de captura, imprime la traza únicamente si la entrada **no contiene espacios** (regex `".*\\s.*"`) y si la propiedad de sistema `printStackTrace` asociada a la clase está activada, resuelta vía `SystemProperties.getBoolean(getClass(), "printStackTrace", () -> false)`.

| @param | Tipo | Requerido | Descripción |
|---|---|---|---|
| `s` | `String` | Sí | Entrada a parsear |
| `function` | `Function<String, ?>` | Sí | Función de parseo a aplicar (referencia a método del JDK) |

| @return | Descripción |
|---|---|
| `boolean` | `true` si `function.apply(s)` devolvió un valor no nulo; `false` si lanzó cualquier `Exception` o si la aserción de no nulidad falló |

| @throws | Condición |
|---|---|
| `Exception` | No se propaga: capturada y convertida en `false` |
| `NullPointerException` | Posible si `s == null` y se alcanza la rama `catch` (`s.matches(...)`) |
| `AssertionError` | No se propaga desde el bloque `try`: `assertNotNull` falla dentro del `try` y su error es capturado por `catch (Exception)` sólo si fuese `Exception`; un `AssertionError` **sí** se propagaría al no ser subclase de `Exception` |

> **Observación de diseño:** `AssertionError` no es `Exception`, por lo que un fallo de `assertNotNull(function.apply(s))` escapa del `catch`. En la práctica los parsers del JDK devuelven valores no nulos o lanzan `NumberFormatException`, por lo que la rama es inalcanzable.

---

#### `isNumberFormatParsable()`

```java
private boolean isNumberFormatParsable(final String s)
```

Comprueba si `java.text.NumberFormat.getInstance().parse(s)` produce un objeto. Es relevante porque `NumberFormat` **detiene el parseo** al encontrar un espacio y aun así retorna un resultado parcial (comportamiento documentado en el comentario del código).

| @param | Tipo | Requerido | Descripción |
|---|---|---|---|
| `s` | `String` | Sí | Entrada a parsear con la configuración regional por defecto |

| @return | Descripción |
|---|---|
| `boolean` | `true` si `parse(s)` devolvió objeto no nulo; `false` si lanzó `ParseException` |

| @throws | Condición |
|---|---|
| `ParseException` | No se propaga: capturada internamente |
| `NullPointerException` | Posible si `s == null`, propagada desde `NumberFormat.parse` |

---

#### `isNumberIntegerOnlyFormatParsable()`

```java
private boolean isNumberIntegerOnlyFormatParsable(final String s)
```

Equivalente a `isNumberFormatParsable()` pero con `NumberFormat.setParseIntegerOnly(true)`, de modo que el parseo se detiene en el separador decimal.

| @param | Tipo | Requerido | Descripción |
|---|---|---|---|
| `s` | `String` | Sí | Entrada a parsear en modo "solo enteros" |

| @return | Descripción |
|---|---|
| `boolean` | `true` si `parse(s)` devolvió objeto no nulo; `false` si lanzó `ParseException` |

| @throws | Condición |
|---|---|
| `ParseException` | No se propaga: capturada internamente |
| `NullPointerException` | Posible si `s == null` |

---

#### Familia `isParsable*()`

Los seis métodos siguientes comparten exactamente la misma estructura: (1) obtienen `NumberUtils.isParsable(s)`; (2) afirman que `NumberFormat` (modo normal y modo entero) puede parsear `s`; (3) afirman que `NumberUtils.isParsable(s)` coincide con el resultado del parser nativo correspondiente del JDK; (4) devuelven el valor de `isParsable`.

| Método | Parser del JDK contrastado (`@param function` interno) |
|---|---|
| `isParsableByte()` | `Byte::parseByte` |
| `isParsableDouble()` | `Double::parseDouble` |
| `isParsableFloat()` | `Float::parseFloat` |
| `isParsableInteger()` | `Integer::parseInt` |
| `isParsableLong()` | `Long::parseLong` |
| `isParsableShort()` | `Short::parseShort` |

Firma común: `private boolean isParsableXxx(final String s)`

| @param | Tipo | Requerido | Descripción |
|---|---|---|---|
| `s` | `String` | Sí | Cadena a evaluar |

| @return | Descripción |
|---|---|
| `boolean` | El valor devuelto por `NumberUtils.isParsable(s)` |

| @throws | Condición |
|---|---|
| `AssertionError` | `NumberFormat` no pudo parsear `s` (en cualquiera de sus dos modos) o `NumberUtils.isParsable(s)` discrepa del parser nativo |
| `NullPointerException` | Posible si `s == null` (propagada desde los auxiliares `isNumberFormatParsable()` / `isNumberIntegerOnlyFormatParsable()`) |

---

### 3.3 Métodos de prueba

> En **todas** las tablas de esta sección: `@return` es siempre `void` y `@throws` es siempre `AssertionError` ante el incumplimiento de cualquier aserción del método. Se indica explícitamente cuando existen excepciones adicionales relevantes verificadas mediante `assertThrows`.

#### 3.3.1 Pruebas de `NumberUtils.compare(...)`

| Método | @param | @return | @throws | Propósito |
|---|---|---|---|---|
| `compareByte()` | — | `void` | `AssertionError` | `compare(byte,byte)` es negativo, cero y positivo según orden natural (`-3/0`, `113/113`, `123/32`). Visibilidad `public` |
| `compareInt()` | — | `void` | `AssertionError` | Idem para `compare(int,int)` (`-3/0`, `113/113`, `213/32`). Visibilidad `public` |
| `compareLong()` | — | `void` | `AssertionError` | Idem para `compare(long,long)`. Visibilidad `public` |
| `compareShort()` | — | `void` | `AssertionError` | Idem para `compare(short,short)`. Visibilidad `public` |
| `testCompareDouble()` | — | `void` | `AssertionError` | Matriz exhaustiva 9×9 de `Double.compare` sobre `NaN`, `±Infinity`, `±MAX_VALUE`, `±1.2`, `±0.0`. Verifica el comportamiento del JDK frente a la antigua funcionalidad de Lang |
| `testCompareFloat()` | — | `void` | `AssertionError` | Matriz exhaustiva 9×9 equivalente para `Float.compare` |

#### 3.3.2 Pruebas de constantes y del constructor del SUT

| Método | @param | @return | @throws | Propósito |
|---|---|---|---|---|
| `testConstants()` | — | `void` | `AssertionError` | Verifica tipo (`assertInstanceOf`) y valor de las 18 constantes `LONG_/INTEGER_/SHORT_/BYTE_/DOUBLE_/FLOAT_` × `ZERO/ONE/MINUS_ONE`. Anotado `@SuppressWarnings("cast")` |
| `testConstructor()` | — | `void` | `AssertionError` | Mediante reflexión (`getDeclaredConstructors`, `Modifier`): `NumberUtils` es instanciable, declara exactamente **1** constructor público, la clase es `public` y **no** `final` |

#### 3.3.3 Pruebas de factorías `create*` tipadas

| Método | @param | @return | @throws | Propósito |
|---|---|---|---|---|
| `testCreateBigDecimal()` | — | `void` | `AssertionError` | `createBigDecimal("1234.5")`, `"0.100000001490116121"` y `createBigDecimal(null) == null` |
| `testCreateBigDecimalFailure(String)` | `str` : `String` (requerido, inyectado por `@ValueSource`) | `void` | `AssertionError` | `@ParameterizedTest`. Para cada entrada inválida (vacía, blancos, `\b\t\n\f\r`, espacios Unicode exóticos, `-`, `--`, `--0`, `+`, `++`, `++0`) exige `NumberFormatException` tanto en `NumberUtils.createBigDecimal(str)` como en `new BigDecimal(str)` |
| `testCreateBigInteger()` | — | `void` | `AssertionError` | Decimal, hexadecimal (`0xff`, `0Xff`, `#ff`, `-0xff`), octal (`0377`, `-0377`), ceros con signo y literales LANG-1645 (`+0xFFFFFFFFFFFFFFFF`, `+#FFFFFFFFFFFFFFFF`, `+01234567`); `null → null` |
| `testCreateBigIntegerFailure(String)` | `str` : `String` (requerido) | `void` | `AssertionError` | `@ParameterizedTest`. Entradas inválidas (`#`, `-#`, `0x`, `-0x`, signos duplicados, `-0x-1`, blancos) deben lanzar `NumberFormatException` en el SUT y en `new BigInteger(str)` |
| `testCreateDouble()` | — | `void` | `AssertionError` | `createDouble("1234.5")`, `"0.100000001490116121"`, `null → null` |
| `testCreateDoubleFailure(String)` | `str` : `String` (requerido) | `void` | `AssertionError` | `@ParameterizedTest`. Paridad de `NumberFormatException` con `Double.valueOf(str)` |
| `testCreateFloat()` | — | `void` | `AssertionError` | `createFloat("1234.5")`, `null → null` |
| `testCreateFloatFailure(String)` | `str` : `String` (requerido) | `void` | `AssertionError` | `@ParameterizedTest`. Paridad con `Float.valueOf(str)` |
| `testCreateInteger()` | — | `void` | `AssertionError` | `createInteger("12345")`, `null → null` y LANG-1645 (`"+0xF"` ≡ `Integer.decode("+0xF")`) |
| `testCreateIntegerFailure(String)` | `str` : `String` (requerido) | `void` | `AssertionError` | `@ParameterizedTest`. Paridad con `Integer.valueOf(str)` |
| `testCreateLong()` | — | `void` | `AssertionError` | `createLong("12345")`, `null → null` y LANG-1645 (`"+0xFFFFFFFF"` ≡ `Long.decode(...)`) |
| `testCreateLongFailure(String)` | `str` : `String` (requerido) | `void` | `AssertionError` | `@ParameterizedTest`. Paridad con `Long.valueOf(str)` |

#### 3.3.4 Pruebas de `NumberUtils.createNumber(String)`

| Método | @param | @return | @throws | Propósito |
|---|---|---|---|---|
| `testCreateNumber()` | — | `void` | `AssertionError` | Prueba maestra de inferencia de tipo: sufijos `D/d/F/f/L/l`, hexadecimales, notación exponencial, promoción `Integer→Long→BigInteger` y `Float→Double→BigDecimal`, `null → null`, y regresiones LANG-521, 638, 693, 822, 1018, 1060 (a–n), 1187, 1215, 1645 (a–c), incluido el descarte del sufijo `L` en hexadecimales que desbordan `Long` |
| `testCreateNumberFailure_1()` | — | `void` | `AssertionError` | Exige `NumberFormatException` para `"--1.1E-700F"` (doble signo) |
| `testCreateNumberFailure_2()` | — | `void` | `AssertionError` | Exige `NumberFormatException` para `"-1.1E+0-7e00"` (`e` y `E` con decimal) |
| `testCreateNumberFailure_3()` | — | `void` | `AssertionError` | Exige `NumberFormatException` para `"-11E+0-7e00"` (sin decimal) |
| `testCreateNumberFailure_4()` | — | `void` | `AssertionError` | Exige `NumberFormatException` para `"1eE+00001"` |
| `testCreateNumberFailure_5()` | — | `void` | `AssertionError` | LANG-1205: `"1234.5ff"` inválido |
| `testCreateNumberFailure_6()` | — | `void` | `AssertionError` | LANG-1205: `"1234.5FF"` inválido |
| `testCreateNumberFailure_7()` | — | `void` | `AssertionError` | LANG-1205: `"1234.5dd"` inválido |
| `testCreateNumberFailure_8()` | — | `void` | `AssertionError` | LANG-1205: `"1234.5DD"` inválido |
| `testCreateNumberMagnitude()` | — | `void` | `AssertionError` | Puntos de conmutación de tipo por magnitud: `Float.MAX_VALUE`, `Double.MAX_VALUE`, subnormales (`4.9e-325`, `1e-325`, `0.1e-325`), hexadecimales de 31/32/63/64 bits y octales equivalentes |
| `testCreateNumberZero()` | — | `void` | `AssertionError` | LANG-1646. Itera sobre `{"0.", ".0", "0.0", "0"}` × exponentes `{-2345, 0, 13}` × sufijos `{∅, F, D}` delegando en `assertCreateNumberZero()`; distingue `+0F/-0F` y `+0D/-0D` |
| `testInvalidNumber()` | — | `void` | `AssertionError` | `NumberFormatException` para `"E123e.3"` y `"-"` |
| `testStringCreateNumberEnsureNoPrecisionLoss()` | — | `void` | `AssertionError` | Verifica la clase concreta retornada (`Float`/`Double`/`BigDecimal`) para preservar precisión; cubre LANG-1060, LANG-1613 y LANG-1444 (incluida la vuelta a `Float` cuando la precisión ya se perdió) |
| `testLang1087()` | — | `void` | `AssertionError` | `"0.0"`, `"+0.0"` y `"-0.0"` se mapean a `Float` conservando el signo del cero |
| `testLang300()` | — | `void` | `AssertionError` | `createNumber` acepta `"-1l"`, `"01l"`, `"1l"` sin lanzar excepción (prueba de humo, sin aserciones) |
| `testLang747()` | — | `void` | `AssertionError` | Frontera de hexadecimales: `Integer` hasta `0x7FFFFFFF`, `Long` hasta `0x7FFF...FFFF`, `BigInteger` a partir de `0x8000000000000000`, incluyendo variantes con ceros a la izquierda |
| `testLang1821()` | — | `void` | `AssertionError` | LANG-1821: `isCreatable`/`createNumber` coherentes para sufijos `L/l` y hexadecimales `0x`/`0X` en los límites `Integer.MAX_VALUE` y `Long.MAX_VALUE` |

#### 3.3.5 Pruebas de predicados `isCreatable` / `isNumber` / `isDigits`

| Método | @param | @return | @throws | Propósito |
|---|---|---|---|---|
| `testIsCreatable()` | — | `void` | `AssertionError` | Batería de ~90 casos vía `compareIsCreatableWithCreateNumber()`; incluye LANG-521, 664, 1216, 1645, 1646 y los límites de exponente `1E±2147483647/8/9`, cuyo resultado esperado depende de `SystemUtils.isJavaVersionAtLeast(JavaVersion.JAVA_21)` |
| `testIsNumber()` | — | `void` | `AssertionError` | Batería equivalente sobre el método obsoleto, vía `compareIsNumberWithCreateNumber()` |
| `testIsNumberLANG1252()` | — | `void` | `AssertionError` | `"+2"` y `"+2.0"` válidos para `isNumber` |
| `testIsNumberLANG1385()` | — | `void` | `AssertionError` | `"L"` (sufijo aislado) inválido |
| `testIsNumberLANG971()` | — | `void` | `AssertionError` | Octales: `"0085"`, `"085"`, `"08"` inválidos; `"07"`, `"00"` válidos |
| `testIsNumberLANG972()` | — | `void` | `AssertionError` | `"0xABCD"` y `"0XABCD"` válidos |
| `testIsNumberLANG992()` | — | `void` | `AssertionError` | `"0.0"` y `"0.4790"` válidos |
| `testLANG1252()` | — | `void` | `AssertionError` | Equivalente de `testIsNumberLANG1252()` para `isCreatable` |
| `testLANG971()` | — | `void` | `AssertionError` | Equivalente de `testIsNumberLANG971()` para `isCreatable` |
| `testLANG972()` | — | `void` | `AssertionError` | Equivalente de `testIsNumberLANG972()` para `isCreatable` |
| `testLANG992()` | — | `void` | `AssertionError` | Equivalente de `testIsNumberLANG992()` para `isCreatable` |
| `testIsDigits()` | — | `void` | `AssertionError` | `isDigits`: `null` y `""` → `false`; `"12345"` → `true`; `"1234.5"`, `"1ab"`, `"abc"` → `false` |

#### 3.3.6 Pruebas de `NumberUtils.isParsable(String)`

| Método | @param | @return | @throws | Propósito |
|---|---|---|---|---|
| `testIsParsableDoubleTrue(String)` | `input` : `String` (requerido, `@ValueSource`) | `void` | `AssertionError` | `@ParameterizedTest`. Ejecuta `Double.parseDouble(input)` como control y exige `isParsable(input) == true` para literales decimales, exponenciales, hexadecimales de punto flotante (`0x1.8p3`) y con ceros a la izquierda |
| `testIsParsableFloatTrue(String)` | `input` : `String` (requerido) | `void` | `AssertionError` | `@ParameterizedTest`. Idem con `Float.parseFloat(input)` y sufijos `f/F` |
| `testIsParsableFalse(String)` | `input` : `String` (requerido) | `void` | `AssertionError` | `@ParameterizedTest`. Exige `isParsable(input) == false` para `""`, hexadecimales enteros, texto, notación científica inválida (`e5`, `1e`, `1ee5`) y sufijos aislados (`f`, `d`, `-f`, `-d`) |
| `testIsParsableTrue(String)` | `input` : `String` (requerido) | `void` | `AssertionError` | `@ParameterizedTest`. Exige `true` para `NaN`, `±Infinity`, enteros con/sin signo, decimales truncados (`64.`, `2.`), notación científica y sufijos de tipo |
| `testIsParsableNull()` | — | `void` | `AssertionError` | `isParsable(null) == false` (aislado porque `@ValueSource(strings)` no admite `null`) |
| `testIsParsableNaN()` | — | `void` | `AssertionError` | Verifica `Double.parseDouble("NaN")`, `Float.parseFloat("NaN")` y `isParsable("NaN")` |
| `testIsParsableInfinity()` | — | `void` | `AssertionError` | Idem para `"Infinity"` |
| `testIsParsableInfinityNegative()` | — | `void` | `AssertionError` | Idem para `"-Infinity"` |
| `testIsParsableInfinityPositive()` | — | `void` | `AssertionError` | Idem para `"+Infinity"` |
| `testIsParsableFullWidthUnicodeJDK8326627()` | — | `void` | `AssertionError` | LANG-1729 / JDK-8326627: los dígitos Unicode de ancho completo `"１２３"` hacen fallar `Double.parseDouble`/`Float.parseFloat` pero `isParsable` devuelve `true`; con parte decimal (`"１２３.0"`, `"0.１２３"`) devuelve `false` |
| `testLang1729IsParsableByte()` | — | `void` | `AssertionError` | Delega en `isParsableByte()`: `"1"` y `"１２３"` parseables; con espacios intercalados, no |
| `testLang1729IsParsableDouble()` | — | `void` | `AssertionError` | Delega en `isParsableDouble()`: `"1"`, `"1."`, `"1.f"`, `"1.d"`, `"1.0"` válidos; `"1.0."` y entradas con espacios, inválidos |
| `testLang1729IsParsableFloat()` | — | `void` | `AssertionError` | Delega en `isParsableFloat()` con el mismo conjunto de casos |
| `testLang1729IsParsableInteger()` | — | `void` | `AssertionError` | Delega en `isParsableInteger()` |
| `testLang1729IsParsableLong()` | — | `void` | `AssertionError` | Delega en `isParsableLong()` |
| `testLang1729IsParsableShort()` | — | `void` | `AssertionError` | Delega en `isParsableShort()` |

#### 3.3.7 Pruebas de `min` / `max`

**Sobrecargas varargs (arrays):**

| Método | @param | @return | @throws | Propósito |
|---|---|---|---|---|
| `testMaxByte()` | — | `void` | `AssertionError` | `max(byte...)` con arrays de longitud 1, 2 y 5 |
| `testMaxShort()` | — | `void` | `AssertionError` | `max(short...)` ídem |
| `testMaxInt()` | — | `void` | `AssertionError` | `max(int...)` ídem |
| `testMaxLong()` | — | `void` | `AssertionError` | `max(long...)` ídem |
| `testMaxFloat()` | — | `void` | `AssertionError` | `max(float...)` ídem (con delta `0.0001f`) |
| `testMaxDouble()` | — | `void` | `AssertionError` | `max(double...)` ídem |
| `testMinByte()` | — | `void` | `AssertionError` | `min(byte...)` con arrays de longitud 1, 2 y 5 |
| `testMinShort()` | — | `void` | `AssertionError` | `min(short...)` ídem |
| `testMinInt()` | — | `void` | `AssertionError` | `min(int...)` ídem |
| `testMinLong()` | — | `void` | `AssertionError` | `min(long...)` ídem |
| `testMinFloat()` | — | `void` | `AssertionError` | `min(float...)` ídem, incluido `min(6.8f, Float.NaN) == Float.NaN` |
| `testMinDouble()` | — | `void` | `AssertionError` | `min(double...)` ídem |

**Validación de argumentos (array vacío / nulo):** cada uno usa `assertIllegalArgumentException(NumberUtils::max|min)` (referencia a método con array vacío) o `assertNullPointerException(() -> ...)` con *cast* explícito del `null`.

| Método | @param | @return | @throws | Excepción esperada del SUT |
|---|---|---|---|---|
| `testMaxByte_emptyArray()` | — | `void` | `AssertionError` | `IllegalArgumentException` |
| `testMaxByte_nullArray()` | — | `void` | `AssertionError` | `NullPointerException` (`(byte[]) null`) |
| `testMaxShort_emptyArray()` | — | `void` | `AssertionError` | `IllegalArgumentException` |
| `testMaxShort_nullArray()` | — | `void` | `AssertionError` | `NullPointerException` (`(short[]) null`) |
| `testMaxInt_emptyArray()` | — | `void` | `AssertionError` | `IllegalArgumentException` |
| `testMaxInt_nullArray()` | — | `void` | `AssertionError` | `NullPointerException` (`(int[]) null`) |
| `testMaxLong_emptyArray()` | — | `void` | `AssertionError` | `IllegalArgumentException` |
| `testMaxLong_nullArray()` | — | `void` | `AssertionError` | `NullPointerException` (`(long[]) null`) |
| `testMaxFloat_emptyArray()` | — | `void` | `AssertionError` | `IllegalArgumentException` |
| `testMaxFloat_nullArray()` | — | `void` | `AssertionError` | `NullPointerException` (`(float[]) null`) |
| `testMaxDouble_emptyArray()` | — | `void` | `AssertionError` | `IllegalArgumentException` |
| `testMaxDouble_nullArray()` | — | `void` | `AssertionError` | `NullPointerException` (`(double[]) null`) |
| `testMinByte_emptyArray()` | — | `void` | `AssertionError` | `IllegalArgumentException` |
| `testMinByte_nullArray()` | — | `void` | `AssertionError` | `NullPointerException` (`(byte[]) null`) |
| `testMinShort_emptyArray()` | — | `void` | `AssertionError` | `IllegalArgumentException` |
| `testMinShort_nullArray()` | — | `void` | `AssertionError` | `NullPointerException` (`(short[]) null`) |
| `testMinInt_emptyArray()` | — | `void` | `AssertionError` | `IllegalArgumentException` |
| `testMinInt_nullArray()` | — | `void` | `AssertionError` | `NullPointerException` (`(int[]) null`) |
| `testMinLong_emptyArray()` | — | `void` | `AssertionError` | `IllegalArgumentException` |
| `testMinLong_nullArray()` | — | `void` | `AssertionError` | `NullPointerException` (`(long[]) null`) |
| `testMinFloat_emptyArray()` | — | `void` | `AssertionError` | `IllegalArgumentException` |
| `testMinFloat_nullArray()` | — | `void` | `AssertionError` | `NullPointerException` (`(float[]) null`) |
| `testMinDouble_emptyArray()` | — | `void` | `AssertionError` | `IllegalArgumentException` |
| `testMinDouble_nullArray()` | — | `void` | `AssertionError` | `NullPointerException` (`(double[]) null`) |

**Sobrecargas de tres argumentos y casos especiales:**

| Método | @param | @return | @throws | Propósito |
|---|---|---|---|---|
| `testMaximumByte()` | — | `void` | `AssertionError` | `max(byte,byte,byte)` en las cuatro permutaciones relevantes |
| `testMaximumShort()` | — | `void` | `AssertionError` | `max(short,short,short)` ídem |
| `testMaximumInt()` | — | `void` | `AssertionError` | `max(int,int,int)` en cinco permutaciones, incluidos empates |
| `testMaximumLong()` | — | `void` | `AssertionError` | `max(long,long,long)` ídem |
| `testMaximumFloat()` | — | `void` | `AssertionError` | `max(float,float,float)` con delta `0.0001f` |
| `testMaximumDouble()` | — | `void` | `AssertionError` | `max(double,double,double)` con delta `0.0001` |
| `testMinimumByte()` | — | `void` | `AssertionError` | `min(byte,byte,byte)` |
| `testMinimumShort()` | — | `void` | `AssertionError` | `min(short,short,short)` |
| `testMinimumInt()` | — | `void` | `AssertionError` | `min(int,int,int)` |
| `testMinimumLong()` | — | `void` | `AssertionError` | `min(long,long,long)` |
| `testMinimumFloat()` | — | `void` | `AssertionError` | `min(float,float,float)` con delta |
| `testMinimumDouble()` | — | `void` | `AssertionError` | `min(double,double,double)` con delta |
| `testMinMaxSignedZero()` | — | `void` | `AssertionError` | Contrato del cero con signo: compara bits crudos (`Double.doubleToRawLongBits`, `Float.floatToRawIntBits`) para exigir que varargs y sobrecarga de 3 argumentos coincidan con `Math.min`/`Math.max` |
| `testLang381()` | — | `void` | `AssertionError` | LANG-381: propagación de `NaN` en `min`/`max` para `double`/`float`, tanto en sobrecargas de 3 argumentos como en arrays con `NaN` al inicio, en medio y al final |

#### 3.3.8 Pruebas de conversiones `to*` tolerantes a fallos

| Método | @param | @return | @throws | Propósito |
|---|---|---|---|---|
| `testToByteString()` | — | `void` | `AssertionError` | `toByte(String)`: `"123"` → 123; `"abc"`, `""`, `null` → 0 |
| `testToByteStringI()` | — | `void` | `AssertionError` | `toByte(String, byte)`: valor por defecto para `"12.3"`, `""` y `null` |
| `testToShortString()` | — | `void` | `AssertionError` | `toShort(String)`: `"12345"` → 12345; `"abc"`, `""`, `null` → 0 |
| `testToShortStringI()` | — | `void` | `AssertionError` | `toShort(String, short)` con valor por defecto `5` |
| `testToIntString()` | — | `void` | `AssertionError` | `toInt(String)`: `"12345"` → 12345; entradas inválidas → 0 |
| `testToIntStringI()` | — | `void` | `AssertionError` | `toInt(String, int)` con valor por defecto `5` |
| `testToLongString()` | — | `void` | `AssertionError` | `toLong(String)`: rechaza sufijos `"1L"`/`"1l"` (→0), acepta `Long.MAX_VALUE`/`MIN_VALUE` |
| `testToLongStringL()` | — | `void` | `AssertionError` | `toLong(String, long)` con valor por defecto `5L` |
| `testToFloatString()` | — | `void` | `AssertionError` | `toFloat(String)` incluido LANG-1060 (ceros a la izquierda y signos) y `Float.MAX_VALUE`/`MIN_VALUE`; `""`/`null` → `0.0f` |
| `testToFloatStringF()` | — | `void` | `AssertionError` | `toFloat(String, float)`: valor por defecto ante entradas corruptas (`"-001Z.2345"`, `"+001AB.2345"`), `""` y `null` |
| `testStringToDoubleString()` | — | `void` | `AssertionError` | `toDouble(String)` incluido LANG-1060, `Double.MAX_VALUE`/`MIN_VALUE`; `""`/`null` → `0.0d` |
| `testStringToDoubleStringD()` | — | `void` | `AssertionError` | `toDouble(String, double)` con valor por defecto `5.1d` |
| `testBigIntegerToDoubleBigInteger()` | — | `void` | `AssertionError` | `toDouble(BigDecimal)`: `null` → `0.0d`; `BigDecimal.valueOf(8.5d)` → `8.5d` |
| `testBigIntegerToDoubleBigIntegerD()` | — | `void` | `AssertionError` | `toDouble(BigDecimal, double)`: `null` → valor por defecto `1.1d` |

#### 3.3.9 Pruebas de `toScaledBigDecimal`

| Método | @param | @return | @throws | Propósito |
|---|---|---|---|---|
| `testToScaledBigDecimalBigDecimal()` | — | `void` | `AssertionError` | `toScaledBigDecimal(BigDecimal)`: escala 2 y `RoundingMode.HALF_EVEN` por defecto; `null` → `BigDecimal.ZERO` |
| `testToScaledBigDecimalBigDecimalIRM()` | — | `void` | `AssertionError` | `toScaledBigDecimal(BigDecimal, int, RoundingMode)` con `CEILING`, `FLOOR`, `HALF_UP`, `HALF_EVEN`; `null` → `ZERO` |
| `testToScaledBigDecimalDouble()` | — | `void` | `AssertionError` | `toScaledBigDecimal(Double)` con redondeo por defecto; `null` → `ZERO` |
| `testToScaledBigDecimalDoubleIRM()` | — | `void` | `AssertionError` | `toScaledBigDecimal(Double, int, RoundingMode)`; `null` → `ZERO` |
| `testToScaledBigDecimalFloat()` | — | `void` | `AssertionError` | `toScaledBigDecimal(Float)`; documenta el error de redondeo aceptado (`23.515f` → `23.51`); `null` → `ZERO` |
| `testToScaledBigDecimalFloatIRM()` | — | `void` | `AssertionError` | `toScaledBigDecimal(Float, int, RoundingMode)`; incluye el desvío aceptado con `HALF_UP`; `null` → `ZERO` |
| `testToScaledBigDecimalString()` | — | `void` | `AssertionError` | `toScaledBigDecimal(String)` con redondeo por defecto; `null` → `ZERO` |
| `testToScaledBigDecimalStringIRM()` | — | `void` | `AssertionError` | `toScaledBigDecimal(String, int, RoundingMode)`; `null` → `ZERO` |

---

### 3.4 Construcciones lambda y referencias a método (elementos "anónimos")

No existen clases anónimas ni clases internas declaradas. Sí existen cuerpos de código anónimos relevantes para la trazabilidad:

| Construcción | Ubicación | Tipo funcional | Propósito |
|---|---|---|---|
| `() -> "Input: " + number` (×3) | `assertCreateNumberZero()` | `Supplier<String>` | Construcción diferida del mensaje de fallo |
| `() -> false` | `isApplyNonNull()` | `BooleanSupplier`/proveedor de valor por defecto para `SystemProperties.getBoolean` | Desactiva por defecto la impresión de trazas |
| `Byte::parseByte`, `Short::parseShort`, `Integer::parseInt`, `Long::parseLong`, `Float::parseFloat`, `Double::parseDouble` | familia `isParsable*()` | `Function<String, ?>` | Parser de referencia del JDK contra el que se contrasta `isParsable` |
| `() -> NumberUtils.createBigDecimal(str)`, `() -> new BigDecimal(str)`, `() -> NumberUtils.createBigInteger(str)`, `() -> new BigInteger(str)`, `() -> Double.valueOf(str)`, `() -> Float.valueOf(str)`, `() -> Integer.valueOf(str)`, `() -> Long.valueOf(str)`, `() -> NumberUtils.createNumber(...)`, `() -> Double.parseDouble(...)`, `() -> Float.parseFloat(...)` | métodos `*Failure*`, `testInvalidNumber()`, `testIsParsableFullWidthUnicodeJDK8326627()` | *Executable* de `assertThrows` | Encapsula la invocación que debe lanzar `NumberFormatException` |
| `NumberUtils::max`, `NumberUtils::min` | métodos `*_emptyArray()` | *Executable* / referencia a método varargs | Invoca el agregador con array vacío para provocar `IllegalArgumentException` |
| `() -> NumberUtils.max((byte[]) null)` y análogos por tipo | métodos `*_nullArray()` | *Executable* | Invoca el agregador con array nulo para provocar `NullPointerException` |

---

## 4. Ejemplo de uso

### 4.1 Reutilización del patrón de verificación cruzada

El valor arquitectónico de esta suite reside en su patrón de **verificación cruzada entre predicado y factoría**. El siguiente ejemplo reproduce ese patrón en una clase de prueba propia, utilizando exclusivamente la API ejercitada por el código documentado (`NumberUtils`, JUnit Jupiter, `java.math`):

```java
package com.example.validation;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertFalse;
import static org.junit.jupiter.api.Assertions.assertInstanceOf;
import static org.junit.jupiter.api.Assertions.assertThrows;
import static org.junit.jupiter.api.Assertions.assertTrue;

import java.math.BigDecimal;
import java.math.BigInteger;
import java.math.RoundingMode;

import org.apache.commons.lang3.math.NumberUtils;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.ValueSource;

/**
 * Ejemplo de uso del patrón de verificación cruzada de {@code NumberUtilsTest}
 * aplicado a la validación de importes recibidos desde una integración externa.
 */
class ImporteParserTest {

    /**
     * Réplica del auxiliar {@code checkCreateNumber(String)}: normaliza el
     * resultado de la factoría a un booleano.
     *
     * @param val cadena candidata; admite {@code null}.
     * @return {@code true} si se obtuvo un número no nulo.
     */
    private boolean puedeCrearNumero(final String val) {
        try {
            return NumberUtils.createNumber(val) != null;
        } catch (final NumberFormatException e) {
            return false;
        }
    }

    /**
     * Réplica del auxiliar {@code compareIsCreatableWithCreateNumber(String, boolean)}.
     *
     * @param val      cadena a evaluar.
     * @param esperado resultado esperado para el predicado y para la factoría.
     */
    private void verificarCoherencia(final String val, final boolean esperado) {
        assertEquals(esperado, NumberUtils.isCreatable(val), () -> "isCreatable: " + val);
        assertEquals(esperado, puedeCrearNumero(val), () -> "createNumber: " + val);
    }

    @ParameterizedTest
    @ValueSource(strings = { "12345", "1234.5", "-1234E5", "0x7FFFFFFF", "22338L", "2." })
    void importeValido(final String entrada) {
        verificarCoherencia(entrada, true);
    }

    @ParameterizedTest
    @ValueSource(strings = { "", " ", "--2", "+-2", ".12.3", "1234E5l", "11 11" })
    void importeInvalido(final String entrada) {
        verificarCoherencia(entrada, false);
        assertFalse(NumberUtils.isParsable(entrada));
    }

    @Test
    void inferenciaDeTipoSinPerdidaDePrecision() {
        assertInstanceOf(Float.class, NumberUtils.createNumber("1.23"));
        assertInstanceOf(Double.class, NumberUtils.createNumber("3.40282354e+38"));
        assertInstanceOf(BigDecimal.class, NumberUtils.createNumber("1.797693134862315759e+308"));
        assertInstanceOf(BigInteger.class, NumberUtils.createNumber("0xFFFFFFFFFFFFFFFF"));
    }

    @Test
    void conversionTolerantePorDefecto() {
        // Las conversiones to* no lanzan excepción: devuelven el valor por defecto.
        assertEquals(0, NumberUtils.toInt("importe-corrupto"));
        assertEquals(5, NumberUtils.toInt(null, 5));
        assertEquals(0.0d, NumberUtils.toDouble((String) null));
        // Las factorías create* sí son estrictas.
        assertThrows(NumberFormatException.class, () -> NumberUtils.createNumber("1234.5FF"));
    }

    @Test
    void redondeoContableADosDecimales() {
        // Escala 2 y RoundingMode.HALF_EVEN por defecto.
        assertEquals(BigDecimal.valueOf(123.46), NumberUtils.toScaledBigDecimal("123.456"));
        assertEquals(BigDecimal.valueOf(23.53),
                NumberUtils.toScaledBigDecimal("23.525", 2, RoundingMode.HALF_UP));
        assertEquals(BigDecimal.ZERO, NumberUtils.toScaledBigDecimal((String) null));
    }

    @Test
    void agregacionConPropagacionDeNaN() {
        assertTrue(Double.isNaN(NumberUtils.max(1.2, 2.5, Double.NaN)));
        assertEquals(10, NumberUtils.min(-5, 0, -10, 5, 10) + 0);
    }
}
```

### 4.2 Ejecución de la suite original

```bash
# Ejecuta únicamente NumberUtilsTest
mvn -Dtest=NumberUtilsTest test

# Activa la impresión de trazas en el auxiliar isApplyNonNull(),
# resuelta mediante SystemProperties.getBoolean(getClass(), "printStackTrace", () -> false)
mvn -Dtest=NumberUtilsTest \
    -DargLine="-Dorg.apache.commons.lang3.math.NumberUtilsTest.printStackTrace=true" \
    test
```

> **Advertencia operativa:** `testIsCreatable()` condiciona varios resultados esperados a `SystemUtils.isJavaVersionAtLeast(JavaVersion.JAVA_21)` (límites de exponente `1E2147483648` y variantes). La suite es, por tanto, **sensible a la versión del JDK** de ejecución y debe validarse en la matriz de JDK soportada por el proyecto.