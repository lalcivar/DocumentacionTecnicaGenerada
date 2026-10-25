# Documentación Técnica - `NumberUtils`

> **Aclaración Importante:** El código proporcionado no corresponde a la clase de implementación `NumberUtils`, sino a su clase de pruebas unitarias **`NumberUtilsTest`** ubicada en `org.apache.commons.lang3.math`. Esta clase valida el comportamiento de `NumberUtils` de **Apache Commons Lang 3**.
>
> La siguiente documentación describe la clase real bajo prueba: **`org.apache.commons.lang3.math.NumberUtils`**, inferida a partir del análisis de `NumberUtilsTest`.

### 1. Información General

| Atributo | Detalle |
| :--- | :--- |
| **Clase** | `org.apache.commons.lang3.math.NumberUtils` |
| **Tipo** | Clase de Utilidad `public` - No instanciable |
| **Paquete** | `org.apache.commons.lang3.math` |
| **Herencia** | `java.lang.Object` |
| **Licencia** | Apache License, Version 2.0 |
| **Dependencia** | Apache Commons Lang 3 |
| **Propósito** | Proporcionar métodos estáticos de ayuda para el manejo, conversión, validación y comparación de tipos numéricos primitivos y objetos `Number`. |

**Descripción General:**
`NumberUtils` es una clase utilitaria `final` con constructor público pero sin estado. Todos sus métodos son `static` y `thread-safe`. Centraliza la lógica para parsear `Strings` a números, validar si una cadena es numérica, realizar conversiones seguras con valores por defecto y encontrar valores mínimos/máximos, manejando correctamente casos especiales como `null`, cadenas vacías, notación hexadecimal, octal, científica, sufijos de tipo y valores `NaN` / `+/- Infinity` / `+/-0.0`.

La clase `NumberUtilsTest extends AbstractLangTest` es la suite de pruebas JUnit 5 que garantiza la cobertura de todos estos comportamientos.

### 2. Constantes Principales

La clase expone constantes reutilizables para evitar el autoboxing innecesario:

*   **Enteros:** `LONG_ZERO`, `LONG_ONE`, `LONG_MINUS_ONE`, `INTEGER_ZERO`, `INTEGER_ONE`, `INTEGER_MINUS_ONE`, `SHORT_ZERO`, `SHORT_ONE`, `SHORT_MINUS_ONE`, `BYTE_ZERO`, `BYTE_ONE`, `BYTE_MINUS_ONE`
*   **Decimales:** `DOUBLE_ZERO`, `DOUBLE_ONE`, `DOUBLE_MINUS_ONE`, `FLOAT_ZERO`, `FLOAT_ONE`, `FLOAT_MINUS_ONE`

Todas son instancias `final` de sus respectivos wrappers `Long`, `Integer`, `Short`, `Byte`, `Double`, `Float`.

### 3. Métodos Principales

#### 3.1. Creación y Parsing de Números

Convierten un `String` en un objeto `Number` específico. Retornan `null` si el input es `null` y lanzan `NumberFormatException` si el formato es inválido.

| Método | Descripción |
| :--- | :--- |
| `createNumber(String str)` | **Método central.** Analiza la cadena y retorna el tipo más apropiado: `Integer`, `Long`, `BigInteger`, `Float`, `Double` o `BigDecimal`. Soporta decimales, notación científica `1.1E200`, hexadecimales `0xFADE`, `#FF`, octales `0377`, y sufijos `L/l`, `F/f`, `D/d`. Si el número excede `Long` o `Double`, hace fallback a `BigInteger`/`BigDecimal`. |
| `createBigDecimal(String str)` | Crea un `BigDecimal`. Retorna `null` si es `null`. |
| `createBigInteger(String str)` | Crea un `BigInteger`. Soporta hex `0x`, `0X`, `#` y octal. |
| `createDouble(String str)` | Crea un `Double` vía `Double.valueOf()`. |
| `createFloat(String str)` | Crea un `Float` vía `Float.valueOf()`. |
| `createInteger(String str)` | Crea un `Integer` vía `Integer.decode()`. Soporta `+0xF`. |
| `createLong(String str)` | Crea un `Long` vía `Long.decode()`. |

#### 3.2. Validación de Cadenas Numéricas

| Método | Descripción |
| :--- | :--- |
| `isCreatable(String str)` | Verifica si `createNumber(str)` podría crear un número válido. Retorna `false` para `null`, vacío, espacios o formatos inválidos como `--2`, `11a`, `1.1L`. Es el reemplazo del método deprecado `isNumber`. |
| `isParsable(String str)` | Verifica si la cadena es parseable por `Float.parseFloat` / `Double.parseDouble`. Es más permisivo que `isCreatable`. Acepta `NaN`, `Infinity`, `+Infinity`, `-Infinity`, `64.`, `2.f`, `1.2e-5f`. Retorna `false` para hexadecimales `0xC1AB` o sufijos `L`. |
| `isDigits(String str)` | Retorna `true` solo si la cadena contiene únicamente dígitos Unicode `0-9`. `null` o `""` retornan `false`. |
| `isNumber(String str)` | **@Deprecated.** Alias de `isCreatable`. Mantenido por compatibilidad. |

#### 3.3. Conversión Segura con Valor por Defecto

No lanzan excepción. Si el parseo falla o el input es `null`/vacío, retornan `0` o el valor por defecto especificado.

| Método | Descripción |
| :--- | :--- |
| `toInt(String str)`, `toInt(String str, int defaultValue)` | Convierte a `int`. |
| `toLong(String str)`, `toLong(String str, long defaultValue)` | Convierte a `long`. |
| `toFloat(String str)`, `toFloat(String str, float defaultValue)` | Convierte a `float`. |
| `toDouble(String str)`, `toDouble(String str, double defaultValue)` | Convierte a `double`. |
| `toByte(String str)`, `toByte(String str, byte defaultValue)` | Convierte a `byte`. |
| `toShort(String str)`, `toShort(String str, short defaultValue)` | Convierte a `short`. |
| `toDouble(BigDecimal value)`, `toDouble(BigDecimal value, double defaultValue)` | Convierte `BigDecimal` a `double`. Retorna `0.0` o el default si es `null`. |
| `toScaledBigDecimal(String/BigDecimal/Double/Float value)`, `toScaledBigDecimal(..., int scale, RoundingMode roundingMode)` | Convierte a `BigDecimal` con escala de 2 decimales y `RoundingMode.HALF_EVEN` por defecto. Si es `null` retorna `BigDecimal.ZERO`. Permite especificar escala y modo de redondeo. |

#### 3.4. Comparación y Cálculo de Mínimo/Máximo

| Método | Descripción |
| :--- | :--- |
| `compare(byte/short/int/long x, y)` | Compara dos primitivos. Retorna `<0`, `0`, `>0`. |
| `compare(double/float)` | Delega a `Double.compare` / `Float.compare` manejando correctamente `NaN` y `-0.0` vs `0.0`. |
| `min(byte[]/short[]/int[]/long[]/float[]/double[] array)` | Retorna el valor mínimo de un array. Lanza `NullPointerException` si es `null` e `IllegalArgumentException` si está vacío. Propaga `NaN` si algún elemento es `NaN`. |
| `max(byte[]/short[]/int[]/long[]/float[]/double[] array)` | Retorna el valor máximo de un array. Mismo manejo de errores que `min`. |
| `min/max(byte/short/int/long/float/double a, b, c)` | Sobrecarga para 3 valores. Respeta el signo de cero: `max(-0.0, 0.0)` es `0.0` y `min(-0.0, 0.0)` es `-0.0`. |

### 4. Manejo de Errores y Casos Borde

*   **Null Safety:** Todos los métodos `create*` retornan `null` si la entrada es `null`. Los métodos `is*` retornan `false`. Los métodos `to*` retornan `0` o el default.
*   **Excepciones:** `NumberFormatException` para formatos inválidos. `NullPointerException` e `IllegalArgumentException` para `min`/`max` con arrays nulos o vacíos.
*   **Precisión:** `createNumber` evita pérdida de precisión: elige `Float` vs `Double` vs `BigDecimal` según la magnitud y cantidad de decimales. Ej: `1.0000001` -> `Float`, `1.00000001` -> `Double`.
*   **Formatos Soportados:** Signos `+/-`, ceros a la izquierda `001234.56`, punto decimal al inicio/final `.123` / `2.`, exponente `e/E` y sufijos de tipo.

### 5. Ejemplo de Uso

```java
// Creación inteligente
Number n1 = NumberUtils.createNumber("0xFF"); // Integer 255
Number n2 = NumberUtils.createNumber("1.1E200"); // Double
Number n3 = NumberUtils.createNumber("12345678901234567890L"); // BigInteger

// Validación
NumberUtils.isCreatable("123.4E21D"); // true
NumberUtils.isParsable("Infinity"); // true
NumberUtils.isDigits("12345"); // true

// Conversión segura
int i = NumberUtils.toInt("abc", 5); // retorna 5
double d = NumberUtils.toDouble("001.2345"); // retorna 1.2345

// Min/Max
long max = NumberUtils.max(5L, 10L, 3L); // 10
double min = NumberUtils.min(new double[]{1.2, Double.NaN, 3.7}); // NaN
```

