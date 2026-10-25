¡Claro! Aquí tienes la documentación técnica para la clase `StringUtils` de Apache Commons Lang, generada a partir del análisis de su clase de prueba `StringUtilsTest`.

---

### Documentación Técnica: Clase `StringUtils` de Apache Commons Lang

### Descripción General

La clase `StringUtils` de la biblioteca Apache Commons Lang 3 proporciona un conjunto de métodos de utilidad para la manipulación de cadenas de caracteres (`String`). Su principal objetivo es ofrecer funcionalidades que no están disponibles en la clase `java.lang.String` nativa, con un enfoque especial en el **manejo seguro de valores `null`**.

En lugar de lanzar una `NullPointerException`, los métodos de esta clase suelen tratar las cadenas `null` como si fueran cadenas vacías o devuelven `null` de manera predecible. Esto simplifica el código al reducir la necesidad de comprobaciones explícitas de nulidad.

Las operaciones que ofrece incluyen, entre otras:
*   **Comprobaciones**: Verificar si una cadena está vacía, en blanco, es numérica, etc.
*   **Manipulación**: Recortar, rellenar (padding), invertir, truncar.
*   **División y Unión**: Métodos `split` y `join` más potentes y flexibles.
*   **Reemplazo**: Búsqueda y reemplazo de subcadenas.
*   **Capitalización**: Cambiar mayúsculas/minúsculas de forma controlada.
*   **Comparación**: Comparaciones seguras frente a `null`.

### Métodos Principales

A continuación se describen algunos de los métodos más importantes y utilizados de la clase `StringUtils`, inferidos a partir de los casos de prueba.

#### 1. Comprobación de Cadenas (`isEmpty`, `isBlank`, etc.)

Estos métodos permiten verificar el estado de una cadena de forma segura.

*   **`isEmpty(CharSequence cs)` / `isNotEmpty(CharSequence cs)`**
    *   **Descripción**: `isEmpty` devuelve `true` si la cadena es `null` o si su longitud es cero (`""`). `isNotEmpty` es su negación.
    *   **Ejemplo**:
        ```java
        StringUtils.isEmpty(null);     // true
        StringUtils.isEmpty("");       // true
        StringUtils.isEmpty(" ");      // false
        StringUtils.isEmpty("abc");    // false
        ```

*   **`isBlank(CharSequence cs)` / `isNotBlank(CharSequence cs)`**
    *   **Descripción**: `isBlank` es más estricto. Devuelve `true` si la cadena es `null`, vacía o si **solo contiene caracteres de espacio en blanco** (espacios, tabuladores, saltos de línea, etc.). `isNotBlank` es su negación.
    *   **Ejemplo**:
        ```java
        StringUtils.isBlank(null);     // true
        StringUtils.isBlank("");       // true
        StringUtils.isBlank("   ");    // true
        StringUtils.isBlank("abc");    // false
        ```

#### 2. Unión de Cadenas (`join`)

Permite concatenar los elementos de un array, `Iterable` o `Iterator` en una sola cadena, usando un separador.

*   **`join(Object[] array, String separator)`**
    *   **Descripción**: Une los elementos de un array en una cadena. Si un elemento es `null`, se trata como una cadena vacía.
    *   **Ejemplo**:
        ```java
        String[] array = {"manzana", "pera", "naranja"};
        StringUtils.join(array, ", "); // retorna "manzana, pera, naranja"

        Object[] mixedArray = {"a", null, "b"};
        StringUtils.join(mixedArray, "-"); // retorna "a--b"
        ```

#### 3. División de Cadenas (`split`)

Ofrece métodos más flexibles para dividir una cadena que el `String.split()` nativo.

*   **`split(String str, String separatorChars)`**
    *   **Descripción**: Divide la cadena usando los caracteres separadores. A diferencia del método nativo, no utiliza expresiones regulares y agrupa los separadores consecutivos, ignorando los tokens vacíos.
    *   **Ejemplo**:
        ```java
        StringUtils.split("a..b.c", "."); // retorna {"a", "b", "c"}
        ```

*   **`splitPreserveAllTokens(String str, String separatorChars)`**
    *   **Descripción**: Similar a `split`, pero **conserva los tokens vacíos** que resultan de separadores adyacentes.
    *   **Ejemplo**:
        ```java
        StringUtils.splitPreserveAllTokens("a..b.c", "."); // retorna {"a", "", "b", "c"}
        ```

#### 4. Valores por Defecto (`defaultString`, `defaultIfBlank`, etc.)

Estos métodos son fundamentales para evitar `NullPointerException`.

*   **`defaultString(String str, String defaultStr)`**
    *   **Descripción**: Devuelve la cadena original si no es `null`; de lo contrario, devuelve la cadena por defecto.
    *   **Ejemplo**:
        ```java
        StringUtils.defaultString(null, "N/A"); // retorna "N/A"
        StringUtils.defaultString("hola", "N/A"); // retorna "hola"
        ```

*   **`defaultIfBlank(CharSequence cs, CharSequence defaultCs)`**
    *   **Descripción**: Devuelve la cadena original si no está "en blanco" (`isNotBlank`); de lo contrario, devuelve la cadena por defecto.
    *   **Ejemplo**:
        ```java
        StringUtils.defaultIfBlank(null, "default");  // retorna "default"
        StringUtils.defaultIfBlank("  ", "default");  // retorna "default"
        StringUtils.defaultIfBlank("text", "default"); // retorna "text"
        ```

#### 5. Relleno de Cadenas (`leftPad`, `rightPad`, `center`)

Útiles para formatear cadenas a una longitud fija.

*   **`leftPad(String str, int size, char padChar)`**
    *   **Descripción**: Rellena la cadena por la izquierda con el carácter `padChar` hasta que alcance la longitud `size`.
    *   **Ejemplo**:
        ```java
        StringUtils.leftPad("123", 5, '0'); // retorna "00123"
        ```

*   **`rightPad(String str, int size, String padStr)`**
    *   **Descripción**: Rellena la cadena por la derecha con la cadena `padStr` hasta alcanzar la longitud `size`.
    *   **Ejemplo**:
        ```java
        StringUtils.rightPad("abc", 7, "-+"); // retorna "abc-+-+"
        ```

#### 6. Manipulación de Mayúsculas/Minúsculas (`capitalize`, `uncapitalize`, `swapCase`)

*   **`capitalize(String str)`**
    *   **Descripción**: Convierte el primer carácter de la cadena a mayúscula (Title Case) y el resto a minúscula.
    *   **Ejemplo**:
        ```java
        StringUtils.capitalize("gato"); // retorna "Gato"
        StringUtils.capitalize("GATO"); // retorna "Gato"
        ```

*   **`uncapitalize(String str)`**
    *   **Descripción**: Convierte el primer carácter de la cadena a minúscula.
    *   **Ejemplo**:
        ```java
        StringUtils.uncapitalize("Gato"); // retorna "gato"
        ```

#### 7. Reemplazo de Cadenas (`replace`, `replaceOnce`, `overlay`)

*   **`replace(String text, String searchString, String replacement)`**
    *   **Descripción**: Reemplaza **todas** las ocurrencias de `searchString` en `text` con `replacement`. Es seguro para `null`.
    *   **Ejemplo**:
        ```java
        StringUtils.replace("hola mundo hola", "hola", "adiós"); // retorna "adiós mundo adiós"
        ```

*   **`replaceOnce(String text, String searchString, String replacement)`**
    *   **Descripción**: Reemplaza solo la **primera** ocurrencia de `searchString`.
    *   **Ejemplo**:
        ```java
        StringUtils.replaceOnce("hola mundo hola", "hola", "adiós"); // retorna "adiós mundo hola"
        ```

#### 8. Eliminación de Caracteres (`strip`, `trim`, `remove`)

*   **`strip(String str)` / `trim(String str)`**
    *   **Descripción**: Ambos eliminan los espacios en blanco de los extremos de una cadena. Son seguros para `null` (devuelven `null` si la entrada es `null`). `strip` utiliza una definición más moderna de espacio en blanco que `trim`.
    *   **Ejemplo**:
        ```java
        StringUtils.strip("  abc  "); // retorna "abc"
        StringUtils.strip(null);     // retorna null
        ```

*   **`remove(String str, String remove)`**
    *   **Descripción**: Elimina todas las ocurrencias de la subcadena `remove` de la cadena principal.
    *   **Ejemplo**:
        ```java
        StringUtils.remove("queued", "ue"); // retorna "qd"
        ```