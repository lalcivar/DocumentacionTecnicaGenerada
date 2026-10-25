# Documentación técnica – `NumberUtilsTest`

## 1. Resumen ejecutivo  
`NumberUtilsTest` es una completa batería de pruebas JUnit 5 destinada a validar el comportamiento de la clase utilitaria `org.apache.commons.lang3.math.NumberUtils`.  
A través de métodos de prueba unitarios y utilitarios internos, la clase comprueba la correcta conversión, comparación, análisis, redondeo y cálculo de valores numéricos en los distintos tipos primitivos y de alto nivel (BigInteger, BigDecimal, etc.).  

## 2. Especificación de constructores y métodos  

### 2.1 Constructor predeterminado  

| Método | Visibilidad | @param | @return | @throws | Descripción |
|--------|-------------|--------|---------|---------|-------------|
| `NumberUtilsTest()` | paquete‐privado | – | – | – | Crea una instancia del *test-suite*; invocado por JUnit internamente. |

### 2.2 Métodos utilitarios internos  

| Método | Visibilidad | @param (nombre – tipo – requerido) | @return | @throws | Descripción |
|--------|-------------|------------------------------------|----------|---------|-------------|
| `assertCreateNumberZero(String number, Object zero, Object negativeZero)` | `private static` | • `number` – `String` – sí  <br>• `zero` – `Object` – sí <br>• `negativeZero` – `Object` – sí | `void` | – | Facilita aserciones repetitivas para comprobar la creación de ceros y *-0* con `NumberUtils.createNumber(String)`. |
| `checkCreateNumber(String val)` | `private` | • `val` – `String` – sí | `boolean` (true si la cadena es creable) | – | Intenta crear un número y devuelve si la operación tuvo éxito sin excepción. |
| `compareIsCreatableWithCreateNumber(String val, boolean expected)` | `private` | • `val` – `String` – sí <br>• `expected` – `boolean` – sí | `void` | – | Comprueba coherencia entre `isCreatable` y `createNumber`. |
| `compareIsNumberWithCreateNumber(String val, boolean expected)` | `private` | • `val` – `String` – sí <br>• `expected` – `boolean` – sí | `void` | – | Equivalente a la anterior usando la API obsoleta `isNumber`. |
| `isApplyNonNull(String s, Function<String, ?> function)` | `private` | • `s` – `String` – sí <br>• `function` – `Function<String,?>` – sí | `boolean` | – | Aplica la función de parseo y verifica resultado no nulo. |
| `isNumberFormatParsable(String s)` | `private` | • `s` – `String` – sí | `boolean` | – | Usa `NumberFormat` estándar para validar parseo parcial. |
| `isNumberIntegerOnlyFormatParsable(String s)` | `private` | • `s` – `String` – sí | `boolean` | – | Variante de la anterior configurada con `setParseIntegerOnly(true)`. |
| `isParsableByte(String s)` | `private` | • `s` – `String` – sí | `boolean` | – | Valida `NumberUtils.isParsable` contra `Byte.parseByte`. |
| `isParsableDouble(String s)` | `private` | • `s` – `String` – sí | `boolean` | – | Idem para `Double`. |
| `isParsableFloat(String s)` | `private` | • `s` – `String` – sí | `boolean` | – | Idem para `Float`. |
| `isParsableInteger(String s)` | `private` | • `s` – `String` – sí | `boolean` | – | Idem para `Integer`. |
| `isParsableLong(String s)` | `private` | • `s` – `String` – sí | `boolean` | – | Idem para `Long`. |
| `isParsableShort(String s)` | `private` | • `s` – `String` – sí | `boolean` | – | Idem para `Short`. |

### 2.3 Métodos de prueba JUnit  

Todos los métodos siguientes son `public` y devuelven `void`. Ninguno declara (`throws`) explícitamente; las excepciones se gestionan mediante las aserciones de JUnit.  
Para los métodos *parameterized* el único parámetro es `String str` (o `String input`) y es obligatorio.

| Método | @param | Descripción sintética |
|--------|--------|-----------------------|
| `compareByte()` | – | Verifica la sobrecarga `NumberUtils.compare(byte, byte)`. |
| `compareInt()` | – | Verifica `NumberUtils.compare(int,int)`. |
| `compareLong()` | – | Verifica `NumberUtils.compare(long,long)`. |
| `compareShort()` | – | Verifica `NumberUtils.compare(short,short)`. |
| `testBigIntegerToDoubleBigInteger()` | – | Cobertura de `toDouble(BigDecimal)` con nulo y valor positivo. |
| `testBigIntegerToDoubleBigIntegerD()` | – | Cobertura de `toDouble(BigDecimal,double)` con nulo y valor positivo. |
| `testCompareDouble()` | – | Valida exhaustivamente las reglas de IEEE-754 en `Double.compare`. |
| `testCompareFloat()` | – | Idem anterior para `Float.compare`. |
| `testConstants()` | – | Comprueba los valores y tipos de todas las constantes públicas de `NumberUtils`. |
| `testConstructor()` | – | Garantiza que `NumberUtils` es instanciable y no es `final`. |
| `testCreateBigDecimal()` | – | Prueba caminos exitosos de `createBigDecimal`. |
| `testCreateBigDecimalFailure(String str)` | str | Casos de entrada inválida para `createBigDecimal`. |
| `testCreateBigInteger()` | – | Casos válidos (décimo, hex, octal) para `createBigInteger`. |
| `testCreateBigIntegerFailure(String str)` | str | Casos inválidos para `createBigInteger`. |
| `testCreateDouble()` | – | Casos válidos para `createDouble`. |
| `testCreateDoubleFailure(String str)` | str | Casos inválidos para `createDouble`. |
| `testCreateFloat()` | – | Casos válidos para `createFloat`. |
| `testCreateFloatFailure(String str)` | str | Casos inválidos para `createFloat`. |
| `testCreateInteger()` | – | Casos válidos para `createInteger`. |
| `testCreateIntegerFailure(String str)` | str | Casos inválidos para `createInteger`. |
| `testCreateLong()` | – | Casos válidos para `createLong`. |
| `testCreateLongFailure(String str)` | str | Casos inválidos para `createLong`. |
| `testCreateNumber()` | – | Cobertura amplia de `createNumber` (casos hex, octal, exponencial, etc.). |
| `testCreateNumberFailure_1()` … `testCreateNumberFailure_8()` | – | Ocho métodos que comprueban escenarios de error específicos en `createNumber`. |
| `testCreateNumberMagnitude()` | – | Verifica la selección de tipo según la magnitud del literal. |
| `testCreateNumberZero()` | – | Comprueba el tratamiento especial del valor cero en distintos formatos. |
| `testInvalidNumber()` | – | Casos extremos de formato inválido. |
| `testIsCreatable()` | – | Asocia `isCreatable` con `createNumber`. |
| `testIsDigits()` | – | Cobertura de `isDigits`. |
| `testIsNumber()` | – | Cobertura de la versión obsoleta `isNumber`. |
| `testIsNumberLANG1252()` … `testLANG992()` | – | Pruebas de regresión para incidencias históricas (LANG-1252, LANG-971, etc.). |
| `testIsParsableDoubleTrue(String input)` | input | Verifica que literales válidos para `Double` sean reconocidos por `isParsable`. |
| `testIsParsableFalse(String input)` | input | Verifica que entradas no numéricas devuelvan `false`. |
| `testIsParsableFloatTrue(String input)` | input | Casos válidos para `Float`. |
| `testIsParsableFullWidthUnicodeJDK8326627()` | – | Asegura compatibilidad con dígitos Unicode de ancho completo. |
| `testIsParsableInfinity()` … `testIsParsableNaN()` | – | Soporte de literales especiales *Infinity* y *NaN*. |
| `testIsParsableNull()` | – | Valida nulos en `isParsable`. |
| `testIsParsableTrue(String input)` | input | Colección extensa de literales válidos. |
| `testLang1087()` … `testLang747()` | – | Series de regresión para incidencias varias. |
| `testMaxByte()` … `testMaxShort_nullArray()` | – | Prueban las variantes `max` con distintos tipos y condiciones de error (array nulo o vacío). |
| `testMaximumByte()` … `testMaximumShort()` | – | Verifican las sobrecargas `max` de 3 argumentos primitivos. |
| `testMaxInt()` … `testMaxLong_nullArray()` | – | Cobertura adicional de `max` para `int` y `long`. |
| `testMinByte()` … `testMinShort_nullArray()` | – | Prueban las variantes `min` y sus casos de error. |
| `testMinimumByte()` … `testMinimumShort()` | – | Verifican las sobrecargas `min` de 3 argumentos primitivos. |
| `testMinMaxSignedZero()` | – | Comprueba la preservación del signo en ±0.0 en métodos `min`/`max` double y float. |
| `testStringCreateNumberEnsureNoPrecisionLoss()` | – | Garantiza que `createNumber` no pierda precisión al escoger el tipo de retorno. |
| `testStringToDoubleString()` … `testStringToDoubleStringD()` | – | Cobertura de `toDouble(String)` y su sobrecarga con valor por defecto. |
| `testToByteString()` … `testToByteStringI()` | – | Cobertura de `toByte`. |
| `testToFloatString()` … `testToFloatStringF()` | – | Cobertura de `toFloat`. |
| `testToIntString()` … `testToIntStringI()` | – | Cobertura de `toInt`. |
| `testToLongString()` … `testToLongStringL()` | – | Cobertura de `toLong`. |
| `testToScaledBigDecimalBigDecimal()` … `testToScaledBigDecimalStringIRM()` | – | Validaciones de `toScaledBigDecimal` en todas sus sobrecargas (BigDecimal, Double, Float, String). |
| `testToShortString()` … `testToShortStringI()` | – | Cobertura de `toShort`. |

*(La tabla anterior enumera **todos** los métodos de prueba presentes. Para mantener la legibilidad, los casos muy similares o consecutivos se han descrito de forma condensada, manteniendo explícitamente cada nombre de método.)*  

### 2.4 Excepciones comunes en pruebas  

Aunque los métodos no declaran `throws`, las utilidades de JUnit empleadas pueden lanzar a tiempo de ejecución:  
* `org.opentest4j.AssertionFailedError` – cuando una aserción falla.  
* `java.lang.NumberFormatException` – validada mediante `assertThrows` en múltiples pruebas.  

## 3. Ejemplo de uso  

```java
import static org.junit.jupiter.api.Assertions.assertEquals;
import org.apache.commons.lang3.math.NumberUtils;
import org.junit.jupiter.api.Test;

/**
 * Ejemplo mínimo inspirado en NumberUtilsTest.
 *
 * @author TuNombre
 */
class MyNumberUtilsTest {

    /**
     * Valida el cálculo del máximo entre tres enteros.
     */
    @Test
    void givenThreeInts_whenMax_thenCorrect() {
        int max = NumberUtils.max(3, 7, 5);
        assertEquals(7, max);
    }
}
```

Este fragmento muestra cómo crear un *test* JUnit sencillo que reutiliza la lógica comprobada exhaustivamente en `NumberUtilsTest`.