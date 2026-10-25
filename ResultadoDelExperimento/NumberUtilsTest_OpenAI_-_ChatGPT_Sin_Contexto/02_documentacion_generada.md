Documentación técnica  
Clase: org.apache.commons.lang3.math.NumberUtilsTest  
Versión: Apache Commons Lang 3.x  
Autoría: Proyecto Apache Commons (extracto con fines de documentación)  

1. Propósito general  
   NumberUtilsTest es la batería de pruebas (JUnit 5) que verifica el correcto funcionamiento de la clase utilitaria org.apache.commons.lang3.math.NumberUtils.  
   • Cubre más de 400 casos de prueba entre aserciones positivas y negativas.  
   • Incluye tests unitarios clásicos, pruebas parametrizadas y casos de regresión asociados a incidencias del bug-tracker (LANG-xxx).  
   • Valida tanto lógica aritmética (min/max/compare), conversión y creación dinámica de números (createX, isCreatable, isParsable), como la gestión de valores especiales (NaN, ±Infinity, ±0).  

2. Estructura principal de métodos de prueba  

A. Métodos auxiliares privados  
   • assertCreateNumberZero(...)  – Comprueba cada forma textual del cero (signos, exponentes, sufijos) contra la instancia numérica esperada.  
   • checkCreateNumber(...)       – Lanza createNumber y captura NumberFormatException para reutilizar en otras aserciones.  
   • compareIsCreatableWithCreateNumber(...) / compareIsNumberWithCreateNumber(...) – Contrasta los resultados de isCreatable / isNumber con createNumber.  
   • isApplyNonNull(...)          – Ejecuta un parser específico (Byte::parseByte, etc.) y confirma retorno no nulo.  
   • isNumberFormatParsable(..) & isNumberIntegerOnlyFormatParsable(..) – Apoyos basados en java.text.NumberFormat.  
   • isParsableX(...)             – Familia de helpers que ejercitan NumberUtils.isParsable más el parser del tipo primitivo correspondiente.  

B. Grupos de prueba (métodos @Test / @ParameterizedTest)  

1) Comparaciones de primitivos  
   • compareByte(), compareShort(), compareInt(), compareLong()  
   • testCompareFloat(), testCompareDouble()  
     Verifican la contractilidad de NumberUtils.compare respecto a java.lang.*.compare, incluyendo ordenación de NaN e infinitos.

2) Comprobación de constantes de la clase objetivo  
   • testConstants() – Tipo, visibilidad y valor de LONG_ZERO, INTEGER_ONE, DOUBLE_MINUS_ONE, etc.

3) Verificación de constructor y modificadores de NumberUtils  
   • testConstructor() – La clase debe ser pública y poseer un único constructor público no final.

4) Creación de instancias numéricas a partir de cadenas  
   • testCreateBigDecimal(), testCreateBigDecimalFailure(..)  
   • testCreateBigInteger(), testCreateBigIntegerFailure(..)  
   • testCreateDouble(),   testCreateDoubleFailure(..)  
   • testCreateFloat(),    testCreateFloatFailure(..)  
   • testCreateInteger(),  testCreateIntegerFailure(..)  
   • testCreateLong(),     testCreateLongFailure(..)  
   • testCreateNumber() y serie testCreateNumberFailure_X()  
     Evalúan rutas válidas y excepcionales; contemplan bases 8/10/16, sufijos L l F f D d, notación científica y casos límite de precisión.

5) Métodos de inferencia y validación  
   • testIsCreatable(), testIsNumber() – Cobertura cruzada entre reconocimiento textual y creación real.  
   • testIsParsable*() – Cobertura específica de isParsable para formatos double, float, enteros y dígitos full-width.  

6) Funciones de mínimo/máximo y compareTo variantes array/var-args  
   • testMax*/testMin* (por tipo) – Prueban sobrecargas byte, short, int, long, float, double y arrays; incluyen verificación de ±0 firmado.

7) Conversión segura a primitivos con valores por defecto  
   • testToDouble*, testToFloat*, testToInt*, testToLong*, testToByte*, testToShort*  
     Confirman retorno predeterminado al recibir cadenas no numéricas o null.

8) Escalado a BigDecimal  
   • testToScaledBigDecimal*() – Verifica redondeo HALF_EVEN por defecto y variantes (escala, modo, sobrecarga Double/Float/String).

9) Casos de regresión (issues LANG-xxx, LANG-1xxx, etc.)  
   Diversos métodos reproducen fallos históricos asegurando no reintroducirlos (octales ambiguos, hexadecimales con sufijo, pérdida de precisión, overflow de exponente, etc.).

3. Cobertura funcional de NumberUtils ejercitada  
   • compare, min, max, isDigits, isNumber (deprecated), isCreatable, isParsable  
   • createX (BigDecimal, BigInteger, etc.) y createNumber  
   • toX (primitivos) y toScaledBigDecimal  
   • Constantes precalculadas *_ZERO/ONE/MINUS_ONE  

4. Dependencias externas  
   • JUnit 5 (org.junit.jupiter.*) para estructura de prueba y parametrización.  
   • java.math.*, java.text.NumberFormat, java.lang.reflect.* para apoyo a tests.  
   • org.apache.commons.lang3.* (SystemUtils, JavaVersion, SystemProperties) para adecuar el resultado a la JVM en ejecución y gestionar switches condicionados por versión.

5. Ejecución y resultados esperados  
   Todas las pruebas deben pasar sin fallar en entornos Java 11+; ciertos asertos están condicionados a Java 21 (exponentes > Integer.MAX_VALUE).  
   Contribuye a la garantía de compatibilidad binaria y exactitud matemática de la API NumberUtils en releases futuras.

6. Convenciones de desarrollo  
   • Cada método de prueba describe en su nombre la funcionalidad verificada.  
   • Se utilizan aserciones de Commons Lang (LangAssertions) para validar lanzado de excepciones.  
   • Los valores mágicos provienen de especificaciones JLS y límites de tipo (Float.MAX_VALUE, Double.MIN_VALUE, etc.).  

Resumen  
NumberUtilsTest es un conjunto exhaustivo de pruebas de regresión y validación que asegura la robustez de todas las operaciones numéricas proporcionadas por NumberUtils, cubriendo creación, comparación, parsers, utilidades de conversión y cálculo de extremos, con especial atención a casos límite de precisión, rango y formato.