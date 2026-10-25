Claro, aquí tienes la documentación técnica para la clase `NumberUtils` de Apache Commons Lang, generada a partir del análisis de su clase de pruebas `NumberUtilsTest`.

---

### **Documentación Técnica: Clase `org.apache.commons.lang3.math.NumberUtils`**

#### **Descripción General**

La clase `NumberUtils` del paquete `org.apache.commons.lang3.math` es una clase de utilidad que proporciona métodos estáticos para trabajar con números en Java. Su objetivo principal es extender la funcionalidad de las clases numéricas estándar de Java (como `Integer`, `Double`, `BigDecimal`, etc.) y facilitar operaciones comunes que de otro modo requerirían código repetitivo o manejo de excepciones.

Entre sus principales capacidades se encuentran la conversión segura de cadenas de texto a tipos numéricos (proporcionando valores por defecto en caso de error), la creación de objetos `Number` a partir de cadenas determinando el tipo más apropiado, la validación de cadenas para comprobar si son números válidos, y la obtención de valores mínimos y máximos de un conjunto de números.

#### **Métodos Principales**

A continuación se describen los grupos de métodos más importantes que ofrece la clase `NumberUtils`, inferidos a partir de su conjunto de pruebas.

##### 1. Creación de Números a partir de Cadenas

*   **`createNumber(String str)`**
    *   **Descripción:** Convierte una cadena de texto (`String`) en un objeto `Number`. Este método es inteligente y determina el tipo de dato numérico más apropiado para la cadena de entrada. Puede devolver un `Integer`, `Long`, `Float`, `Double`, `BigInteger` o `BigDecimal` dependiendo del formato y la magnitud del número en la cadena. Maneja notación decimal, hexadecimal (`0x...`), científica (`e` o `E`) y sufijos de tipo (como 'L', 'F', 'D').
    *   **Ejemplo de uso:** `NumberUtils.createNumber("123.45F")` devolvería un objeto `Float`. `NumberUtils.createNumber("0xFF")` devolvería un `Integer` con el valor 255.

##### 2. Validación de Cadenas Numéricas

*   **`isCreatable(String str)` / `isParsable(String str)`**
    *   **Descripción:** Métodos de validación que comprueban si una cadena de texto representa un número válido que puede ser procesado por Java.
        *   `isCreatable(String str)`: Verifica si se puede crear un número a partir de la cadena de texto utilizando la lógica de `createNumber()`. Devuelve `true` si la cadena es un número Java válido, incluyendo formatos hexadecimales y sufijos de tipo.
        *   `isParsable(String str)`: Una versión más moderna y estricta para verificar si una cadena puede ser analizada como un número. Se alinea más con los métodos `parse...()` de las clases wrapper de Java.
    *   **Ejemplo de uso:** `NumberUtils.isCreatable("123L")` devuelve `true`, mientras que `NumberUtils.isCreatable("abc")` devuelve `false`.

*   **`isDigits(String str)`**
    *   **Descripción:** Comprueba si la cadena contiene únicamente caracteres de dígitos (`0`-`9`). No admite signos, puntos decimales ni exponentes.
    *   **Ejemplo de uso:** `NumberUtils.isDigits("123")` devuelve `true`, pero `NumberUtils.isDigits("12.3")` devuelve `false`.

##### 3. Conversión Segura de Cadenas a Primitivos (con valor por defecto)

*   **`to[Type](String str, [Type] defaultValue)`**
    *   **Descripción:** Una familia de métodos para convertir una cadena de texto a un tipo numérico primitivo de forma segura. Si la conversión falla (por ejemplo, si la cadena no es un número válido, es nula o está vacía), devuelven un valor por defecto proporcionado por el usuario en lugar de lanzar una `NumberFormatException`.
    *   **Métodos disponibles:**
        *   `toInt(String str, int defaultValue)`
        *   `toLong(String str, long defaultValue)`
        *   `toFloat(String str, float defaultValue)`
        *   `toDouble(String str, double defaultValue)`
        *   `toByte(String str, byte defaultValue)`
        *   `toShort(String str, short defaultValue)`
    *   **Ejemplo de uso:** `NumberUtils.toInt("32", 0)` devuelve `32`. `NumberUtils.toInt("hello", 0)` devuelve `0`.

##### 4. Búsqueda de Mínimos y Máximos

*   **`min(...)` / `max(...)`**
    *   **Descripción:** Métodos sobrecargados para encontrar el valor mínimo o máximo dentro de un conjunto de números. Existen versiones para todos los tipos de datos numéricos primitivos (`byte`, `short`, `int`, `long`, `float`, `double`). Aceptan tanto un array de números como una lista de argumentos variables.
    *   **Ejemplo de uso:**
        *   `NumberUtils.max(10, 20, 5)` devuelve `20`.
        *   `int[] nums = {4, 1, 9}; NumberUtils.min(nums)` devuelve `1`.

##### 5. Comparación de Números

*   **`compare(primitive a, primitive b)`**
    *   **Descripción:** Compara dos valores numéricos primitivos (`byte`, `short`, `int`, `long`). Devuelve un valor negativo, cero o positivo si el primer argumento es menor, igual o mayor que el segundo, respectivamente. Es similar a los métodos `compare` introducidos en las clases wrapper de Java.
    *   **Ejemplo de uso:** `NumberUtils.compare(5, 10)` devuelve un valor negativo.

##### 6. Operaciones con `BigDecimal`

*   **`toScaledBigDecimal(...)`**
    *   **Descripción:** Convierte diferentes tipos de números (`Float`, `Double`, `BigDecimal`, `String`) a un `BigDecimal` con una escala y un modo de redondeo específicos. Es útil para realizar cálculos financieros o científicos donde la precisión es crítica.
    *   **Ejemplo de uso:** `NumberUtils.toScaledBigDecimal(123.456f, 2, RoundingMode.HALF_UP)` devolvería un `BigDecimal` con el valor `123.46`.