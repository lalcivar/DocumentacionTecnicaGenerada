# Documentación Técnica — `org.apache.commons.lang3.function.FailableTest`

---

## 1. Resumen ejecutivo

`FailableTest` es la clase de pruebas unitarias (JUnit 5) que valida el contrato de las interfaces funcionales *"failable"* del paquete `org.apache.commons.lang3.function` (`FailableConsumer`, `FailableFunction`, `FailablePredicate`, `FailableBiFunction`, etc.) y de la clase utilitaria `Failable`. Su objetivo es verificar tres aspectos: (a) que las excepciones comprobadas (`checked`) declaradas en los genéricos realmente se propagan, (b) que `Failable` reenvía (`rethrow`) correctamente `RuntimeException`, `Error`, `IOException` (envuelta en `UncheckedIOException`) y el resto de `Throwable` (envueltos en `UndeclaredThrowableException`), y (c) que los métodos de composición (`andThen`, `compose`, `and`, `or`, `negate`, `nop`, `identity`) cumplen sus contratos, incluidos los casos borde con `null`.

La clase extiende `AbstractLangTest` y contiene cinco clases anidadas estáticas auxiliares (`CloseableObject`, `FailureOnInvocationCount`, `FailureOnOddInvocations`, `SomeException`, `Testable`) que actúan como dobles de prueba configurables para forzar fallos deterministas.

**Visibilidad de la clase:** `class FailableTest extends AbstractLangTest` (nivel de paquete).

### 1.1 Constantes de estado (campos estáticos)

| Campo | Tipo | Modificadores | Propósito |
|---|---|---|---|
| `ERROR` | `OutOfMemoryError` | `private static final` | Instancia reutilizable de `Error` para validar que los `Error` se propagan sin envolver. |
| `ILLEGAL_STATE_EXCEPTION` | `IllegalStateException` | `private static final` | Instancia reutilizable de `RuntimeException` para validar propagación directa. |

---

## 2. Especificación de métodos y constructores

### 2.1 Clase anidada `FailableTest.CloseableObject`

Doble de prueba que simula un recurso cerrable, usado en `testTryWithResources()`. Dispone del constructor por defecto implícito `CloseableObject()`.

| Campo | Tipo | Modificadores |
|---|---|---|
| `closed` | `boolean` | `private` |

#### `close()`

```java
/**
 * Marca el recurso como cerrado estableciendo el indicador interno {@code closed} a {@code true}.
 */
public void close()
```

| @param | Tipo | Requerido |
|---|---|---|
| *(ninguno)* | — | — |

| @return | @throws |
|---|---|
| `void` | *(ninguna)* |

#### `isClosed()`

```java
/**
 * Indica si {@link #close()} ya fue invocado desde la última llamada a {@link #reset()}.
 *
 * @return {@code true} si el recurso está marcado como cerrado.
 */
public boolean isClosed()
```

| @param | Tipo | Requerido |
|---|---|---|
| *(ninguno)* | — | — |

| @return | Descripción | @throws |
|---|---|---|
| `boolean` | Valor del indicador `closed`. | *(ninguna)* |

#### `reset()`

```java
/**
 * Restablece el indicador {@code closed} a {@code false} para reutilizar la instancia entre aserciones.
 */
public void reset()
```

| @param | Tipo | Requerido |
|---|---|---|
| *(ninguno)* | — | — |

| @return | @throws |
|---|---|
| `void` | *(ninguna)* |

#### `run(Throwable)`

```java
/**
 * Lanza el {@code Throwable} recibido, o no hace nada si es {@code null}.
 * Se usa como referencia de método ({@code closeable::run}) para construir un FailableConsumer.
 *
 * @param throwable excepción a lanzar; puede ser {@code null}.
 * @throws Throwable el argumento recibido, cuando no es {@code null}.
 */
public void run(final Throwable throwable) throws Throwable
```

| @param | Tipo | Requerido | Descripción |
|---|---|---|---|
| `throwable` | `Throwable` | No (admite `null`) | Excepción a relanzar. |

| @return | @throws |
|---|---|
| `void` | `Throwable` — el propio argumento si no es `null`. |

---

### 2.2 Clase anidada `FailableTest.FailureOnInvocationCount`

Doble de prueba que falla exactamente en la *n*-ésima invocación. Se emplea en `testApplyNonNull()`, `testApplyNonNull2()` y `testApplyNonNull3()`.

| Campo | Tipo | Modificadores |
|---|---|---|
| `current` | `int` | `private` |
| `throwOn` | `int` | `private final` |

#### Constructor `FailureOnInvocationCount(int)`

```java
/**
 * Crea un contador que fallará cuando el número de invocaciones alcance {@code throwOn}.
 *
 * @param throwOn número de invocación (base 1) en la que se lanzará la excepción.
 * @throws SomeException declarada en la firma para forzar el uso de interfaces failable;
 *         en la implementación actual nunca se lanza.
 */
FailureOnInvocationCount(final int throwOn) throws SomeException
```

| @param | Tipo | Requerido | Descripción |
|---|---|---|---|
| `throwOn` | `int` | Sí | Invocación en la que se produce el fallo. |

| @return | @throws |
|---|---|
| *(constructor)* | `SomeException` — declarada, no lanzada. |

#### `inc(int)`

```java
/**
 * Incrementa el valor recibido, fallando con SomeException en la invocación configurada.
 *
 * @param value valor base.
 * @return {@code value + 1}.
 * @throws SomeException si el contador interno alcanza {@code throwOn}.
 */
int inc(final int value) throws SomeException
```

| @param | Tipo | Requerido |
|---|---|---|
| `value` | `int` | Sí |

| @return | Descripción | @throws |
|---|---|---|
| `int` | `value + 1`. | `SomeException` (vía `throwSeOn()`). |

#### `incCnfe(int)`

```java
/**
 * Variante de {@link #inc(int)} que falla con ClassNotFoundException.
 *
 * @param value valor base.
 * @return {@code value + 1}.
 * @throws ClassNotFoundException si el contador interno alcanza {@code throwOn}.
 */
int incCnfe(final int value) throws ClassNotFoundException
```

| @param | Tipo | Requerido |
|---|---|---|
| `value` | `int` | Sí |

| @return | Descripción | @throws |
|---|---|---|
| `int` | `value + 1`. | `ClassNotFoundException` (vía `throwCnfeOn()`). |

> **Observación:** `incCnfe(int)` no es referenciado por ninguna prueba del código provisto; está disponible para escenarios de excepciones heterogéneas.

#### `incIo(int)`

```java
/**
 * Variante de {@link #inc(int)} que falla con IOException cuya causa es una SomeException.
 *
 * @param value valor base.
 * @return {@code value + 1}.
 * @throws IOException si el contador interno alcanza {@code throwOn}.
 */
int incIo(final int value) throws IOException
```

| @param | Tipo | Requerido |
|---|---|---|
| `value` | `int` | Sí |

| @return | Descripción | @throws |
|---|---|---|
| `int` | `value + 1`. | `IOException` (vía `throwIoOn()`). |

#### `reset()`

```java
/**
 * Reinicia el contador de invocaciones a cero, permitiendo repetir el ciclo de fallo.
 */
void reset()
```

| @param | Tipo | Requerido |
|---|---|---|
| *(ninguno)* | — | — |

| @return | @throws |
|---|---|
| `void` | *(ninguna)* |

#### `throwCnfeOn()`

```java
/**
 * Incrementa el contador y lanza ClassNotFoundException (con SomeException como causa)
 * cuando la cuenta coincide con {@code throwOn}.
 *
 * @throws ClassNotFoundException cuando {@code ++current == throwOn}.
 */
private void throwCnfeOn() throws ClassNotFoundException
```

| @param | Tipo | Requerido |
|---|---|---|
| *(ninguno)* | — | — |

| @return | @throws |
|---|---|
| `void` | `ClassNotFoundException` con mensaje `"Count: n"` y causa `SomeException`. |

#### `throwIoOn()`

```java
/**
 * Incrementa el contador y lanza IOException (con SomeException como causa)
 * cuando la cuenta coincide con {@code throwOn}.
 *
 * @throws IOException cuando {@code ++current == throwOn}.
 */
private void throwIoOn() throws IOException
```

| @param | Tipo | Requerido |
|---|---|---|
| *(ninguno)* | — | — |

| @return | @throws |
|---|---|
| `void` | `IOException` cuya causa es `SomeException("Count: n", n)`. |

#### `throwSeOn()`

```java
/**
 * Incrementa el contador y lanza SomeException cuando la cuenta coincide con {@code throwOn}.
 *
 * @throws SomeException cuando {@code ++current == throwOn}.
 */
private void throwSeOn() throws SomeException
```

| @param | Tipo | Requerido |
|---|---|---|
| *(ninguno)* | — | — |

| @return | @throws |
|---|---|
| `void` | `SomeException("Count: n", n)`. |

---

### 2.3 Clase anidada `FailableTest.FailureOnOddInvocations`

Doble de prueba con contador **estático**: falla en todas las invocaciones impares y tiene éxito en las pares. Permite verificar el patrón "primera llamada lanza, segunda retorna".

| Campo | Tipo | Modificadores |
|---|---|---|
| `invocations` | `int` | `private static` (manipulado directamente por la clase externa: `FailureOnOddInvocations.invocations = 0`) |

#### Constructor `FailureOnOddInvocations()`

```java
/**
 * Construye la instancia delegando en {@link #throwOnOdd()}; por tanto falla en
 * construcciones impares. Se usa como referencia {@code FailureOnOddInvocations::new}
 * para FailableRunnable, FailableCallable y FailableSupplier.
 *
 * @throws SomeException si el número global de invocaciones resultante es impar.
 */
FailureOnOddInvocations() throws SomeException
```

| @param | Tipo | Requerido |
|---|---|---|
| *(ninguno)* | — | — |

| @return | @throws |
|---|---|
| *(constructor)* | `SomeException` con mensaje `"Odd Invocation: n"`. |

#### `reset()`

```java
/**
 * Restablece el contador estático global de invocaciones a cero.
 */
static void reset()
```

| @param | Tipo | Requerido |
|---|---|---|
| *(ninguno)* | — | — |

| @return | @throws |
|---|---|
| `void` | *(ninguna)* |

#### `testDouble(double)`

```java
/**
 * Predicado failable sobre un double; falla en invocaciones impares.
 *
 * @param value valor evaluado (ignorado por la lógica).
 * @return siempre {@code true} cuando no se lanza excepción.
 * @throws SomeException en invocaciones impares.
 */
static boolean testDouble(final double value) throws SomeException
```

| @param | Tipo | Requerido |
|---|---|---|
| `value` | `double` | Sí (no se usa) |

| @return | Descripción | @throws |
|---|---|---|
| `boolean` | `true`. | `SomeException`. |

#### `testGetBool()`

```java
/**
 * Proveedor booleano failable; falla en invocaciones impares.
 *
 * @return siempre {@code true} cuando no se lanza excepción.
 * @throws SomeException en invocaciones impares.
 */
static boolean testGetBool() throws SomeException
```

| @param | Tipo | Requerido |
|---|---|---|
| *(ninguno)* | — | — |

| @return | Descripción | @throws |
|---|---|---|
| `boolean` | `true`. | `SomeException`. |

#### `testInc(int)`

```java
/**
 * Operador unario failable sobre int; falla en invocaciones impares.
 *
 * @param value valor base.
 * @return {@code value + 1}.
 * @throws SomeException en invocaciones impares.
 */
static int testInc(final int value) throws SomeException
```

| @param | Tipo | Requerido |
|---|---|---|
| `value` | `int` | Sí |

| @return | Descripción | @throws |
|---|---|---|
| `int` | `value + 1`. | `SomeException`. |

> **Observación:** `testInc(int)` no es invocado por ninguna prueba del código provisto.

#### `testInt(int)`

```java
/**
 * Predicado failable sobre un int; falla en invocaciones impares.
 *
 * @param value valor evaluado (ignorado por la lógica).
 * @return siempre {@code true} cuando no se lanza excepción.
 * @throws SomeException en invocaciones impares.
 */
static boolean testInt(final int value) throws SomeException
```

| @param | Tipo | Requerido |
|---|---|---|
| `value` | `int` | Sí (no se usa) |

| @return | Descripción | @throws |
|---|---|---|
| `boolean` | `true`. | `SomeException`. |

#### `testLong(long)`

```java
/**
 * Predicado failable sobre un long; falla en invocaciones impares.
 *
 * @param value valor evaluado (ignorado por la lógica).
 * @return siempre {@code true} cuando no se lanza excepción.
 * @throws SomeException en invocaciones impares.
 */
static boolean testLong(final long value) throws SomeException
```

| @param | Tipo | Requerido |
|---|---|---|
| `value` | `long` | Sí (no se usa) |

| @return | Descripción | @throws |
|---|---|---|
| `boolean` | `true`. | `SomeException`. |

#### `throwOnOdd()`

```java
/**
 * Incrementa el contador estático y lanza SomeException si el valor resultante es impar.
 *
 * @throws SomeException con mensaje "Odd Invocation: n" y {@code value = n}.
 */
private static void throwOnOdd() throws SomeException
```

| @param | Tipo | Requerido |
|---|---|---|
| *(ninguno)* | — | — |

| @return | @throws |
|---|---|
| `void` | `SomeException`. |

#### `getAsBoolean()`

```java
/**
 * Variante de instancia del proveedor booleano failable; falla en invocaciones impares.
 *
 * @return siempre {@code true} cuando no se lanza excepción.
 * @throws SomeException en invocaciones impares.
 */
boolean getAsBoolean() throws SomeException
```

| @param | Tipo | Requerido |
|---|---|---|
| *(ninguno)* | — | — |

| @return | Descripción | @throws |
|---|---|---|
| `boolean` | `true`. | `SomeException`. |

---

### 2.4 Clase anidada `FailableTest.SomeException extends Exception`

Excepción comprobada propia de las pruebas. Transporta un valor entero (`value`) que identifica la invocación que provocó el fallo, y un `Throwable` opcional (`t`) reemitible.

| Campo | Tipo | Modificadores |
|---|---|---|
| `serialVersionUID` | `long` | `private static final` (`-4965704778119283411L`) |
| `t` | `Throwable` | `private` |
| `value` | `int` | `final` (nivel de paquete) |

#### Constructor `SomeException(String)`

```java
/**
 * Crea la excepción con el mensaje indicado y valor cero.
 *
 * @param message mensaje descriptivo.
 */
SomeException(final String message)
```

| @param | Tipo | Requerido | Descripción |
|---|---|---|---|
| `message` | `String` | Sí | Mensaje delegado a `super(message)` vía `this(message, 0)`. |

| @return | @throws |
|---|---|
| *(constructor)* | *(ninguna)* |

> **Observación:** este constructor no es utilizado por las pruebas del código provisto.

#### Constructor `SomeException(String, int)`

```java
/**
 * Crea la excepción con mensaje y el número de invocación asociado.
 *
 * @param message mensaje descriptivo.
 * @param value   número de invocación / discriminante usado en las aserciones.
 */
SomeException(final String message, final int value)
```

| @param | Tipo | Requerido |
|---|---|---|
| `message` | `String` | Sí |
| `value` | `int` | Sí |

| @return | @throws |
|---|---|
| *(constructor)* | *(ninguna)* |

#### `setThrowable(Throwable)`

```java
/**
 * Establece el Throwable que {@link #test()} relanzará.
 *
 * @param throwable excepción a almacenar; puede ser {@code null}.
 */
public void setThrowable(final Throwable throwable)
```

| @param | Tipo | Requerido |
|---|---|---|
| `throwable` | `Throwable` | No (admite `null`) |

| @return | @throws |
|---|---|
| `void` | *(ninguna)* |

#### `test()`

```java
/**
 * Relanza el Throwable configurado mediante {@link #setThrowable(Throwable)}, si existe.
 *
 * @throws Throwable el Throwable almacenado, cuando no es {@code null}.
 */
void test() throws Throwable
```

| @param | Tipo | Requerido |
|---|---|---|
| *(ninguno)* | — | — |

| @return | @throws |
|---|---|
| `void` | `Throwable`. |

---

### 2.5 Clase anidada genérica `FailableTest.Testable<T, P>`

Doble de prueba multipropósito: almacena un `Throwable` configurable y expone métodos con todas las formas (objeto, primitivo, bi-argumento) requeridas por las interfaces failable. Los métodos registran los argumentos recibidos en `acceptedObject`, `acceptedPrimitiveObject1` y `acceptedPrimitiveObject2`, lo que permite verificar que **no** se produjo escritura cuando la invocación falla.

| Parámetro de tipo | Significado |
|---|---|
| `T` | Tipo del objeto aceptado (`acceptedObject`). |
| `P` | Tipo del valor primitivo envuelto aceptado (`Double`, `Integer`, `Long`...). |

| Campo | Tipo | Modificadores |
|---|---|---|
| `acceptedObject` | `T` | `private` |
| `acceptedPrimitiveObject1` | `P` | `private` |
| `acceptedPrimitiveObject2` | `P` | `private` |
| `throwable` | `Throwable` | `private` |

#### Constructor `Testable(Throwable)`

```java
/**
 * Crea un doble de prueba que fallará con el Throwable indicado.
 *
 * @param throwable excepción/error a lanzar en los métodos de prueba; {@code null} para
 *        comportamiento exitoso.
 */
Testable(final Throwable throwable)
```

| @param | Tipo | Requerido |
|---|---|---|
| `throwable` | `Throwable` | No (admite `null`) |

| @return | @throws |
|---|---|
| *(constructor)* | *(ninguna)* |

#### Getters y configuración

| Método | @param | @return | @throws |
|---|---|---|---|
| `getAcceptedObject()` | *(ninguno)* | `T` — último objeto aceptado; `null` si nunca se aceptó. | *(ninguna)* |
| `getAcceptedPrimitiveObject1()` | *(ninguno)* | `P` — primer primitivo aceptado (envuelto); `null` si no hubo. | *(ninguna)* |
| `getAcceptedPrimitiveObject2()` | *(ninguno)* | `P` — segundo primitivo aceptado (envuelto); `null` si no hubo. | *(ninguna)* |
| `setThrowable(Throwable throwable)` | `throwable` : `Throwable` (opcional, admite `null`) | `void` | *(ninguna)* |

#### Métodos de ejecución y sobrecargas

| Método | @param (nombre : tipo, requerido) | @return | @throws |
|---|---|---|---|
| `test()` | *(ninguno)* | `void` | `Throwable` — delega en `test(throwable)`. |
| `test(Object, Object)` | `input1` : `Object` (sí, ignorado); `input2` : `Object` (sí, ignorado) | `Object` — devuelve `acceptedObject`. | `Throwable` — el configurado. *(Método no usado por las pruebas provistas.)* |
| `test(Throwable)` | `throwable` : `Throwable` (opcional) | `void` | `Throwable` — el argumento, si no es `null`. |
| `testAsBooleanPrimitive()` | *(ninguno)* | `boolean` — delega en `testAsBooleanPrimitive(throwable)`. | `Throwable` |
| `testAsBooleanPrimitive(Throwable)` | `throwable` : `Throwable` (opcional) | `boolean` — `false` si no falla. | `Throwable` |
| `testAsDoublePrimitive()` | *(ninguno)* | `double` — delega en la sobrecarga. | `Throwable` |
| `testAsDoublePrimitive(Throwable)` | `throwable` : `Throwable` (opcional) | `double` — `0` si no falla. | `Throwable` |
| `testAsInteger()` | *(ninguno)* | `Integer` — delega en la sobrecarga. | `Throwable` |
| `testAsInteger(Throwable)` | `throwable` : `Throwable` (opcional) | `Integer` — `0` si no falla. | `Throwable` |
| `testAsIntPrimitive()` | *(ninguno)* | `int` — delega en la sobrecarga. | `Throwable` |
| `testAsIntPrimitive(Throwable)` | `throwable` : `Throwable` (opcional) | `int` — `0` si no falla. | `Throwable` |
| `testAsLongPrimitive()` | *(ninguno)* | `long` — delega en la sobrecarga. | `Throwable` |
| `testAsLongPrimitive(Throwable)` | `throwable` : `Throwable` (opcional) | `long` — `0` si no falla. | `Throwable` |
| `testAsShortPrimitive()` | *(ninguno)* | `short` — delega en la sobrecarga. | `Throwable` |
| `testAsShortPrimitive(Throwable)` | `throwable` : `Throwable` (opcional) | `short` — `0` si no falla. | `Throwable` |
| `testDouble(double i)` | `i` : `double` (sí) | `void` — almacena `i` en `acceptedPrimitiveObject1` (cast no comprobado a `P`). | `Throwable` |
| `testDoubleDouble(double i, double j)` | `i` : `double` (sí); `j` : `double` (sí) | `double` — constante `3d`; almacena `i` y `j`. | `Throwable` |
| `testInt(int i)` | `i` : `int` (sí) | `void` — almacena `i` en `acceptedPrimitiveObject1`. | `Throwable` |
| `testLong(long i)` | `i` : `long` (sí) | `void` — almacena `i` en `acceptedPrimitiveObject1`. | `Throwable` |
| `testObjDouble(T object, double i)` | `object` : `T` (sí); `i` : `double` (sí) | `void` — almacena ambos. | `Throwable` |
| `testObjInt(T object, int i)` | `object` : `T` (sí); `i` : `int` (sí) | `void` — almacena ambos. | `Throwable` |
| `testObjLong(T object, long i)` | `object` : `T` (sí); `i` : `long` (sí) | `void` — almacena ambos. | `Throwable` |

> **Nota técnica:** todos los métodos que almacenan primitivos realizan `(P) (Double|Integer|Long) i`, un *unchecked cast* seguro solo porque las pruebas instancian `Testable<?, Double>`, `Testable<?, Integer>` o `Testable<?, Long>` coherentemente.

---

### 2.6 Método de ciclo de vida de la prueba

#### `beforeEach()`

```java
/**
 * Reinicia el contador estático global de FailureOnOddInvocations antes de cada prueba,
 * garantizando el aislamiento entre casos de test.
 */
@BeforeEach
void beforeEach()
```

| @param | Tipo | Requerido |
|---|---|---|
| *(ninguno)* | — | — |

| @return | @throws |
|---|---|
| `void` | *(ninguna)* |

---

### 2.7 Métodos de prueba — `Failable.accept*` / `Failable.apply*`

Todos son `void`, sin parámetros, anotados con `@Test`.

| Método | @param | @return | @throws (declaradas) | Comportamiento verificado |
|---|---|---|---|---|
| `testAcceptBiConsumer()` | — | `void` | — | `Failable.accept(FailableBiConsumer, T, U)` propaga `IllegalStateException` y `OutOfMemoryError` sin envolver, envuelve `IOException` en `UncheckedIOException` (misma causa) y no lanza con `null`. |
| `testAcceptConsumer()` | — | `void` | — | Idéntico al anterior para `FailableConsumer` (un solo argumento). |
| `testAcceptDoubleConsumer()` | — | `void` | — | `Failable.accept(FailableDoubleConsumer, double)`: ante fallo `acceptedPrimitiveObject1` permanece `null`; en éxito vale `1`. |
| `testAcceptIntConsumer()` | — | `void` | — | Igual que el anterior con `FailableIntConsumer` y valor `1`. |
| `testAcceptLongConsumer()` | — | `void` | — | Igual con `FailableLongConsumer` y valor `1L`. |
| `testAcceptObjDoubleConsumer()` | — | `void` | — | `FailableObjDoubleConsumer`: verifica que ni `acceptedObject` ni el primitivo se escriben ante fallo; en éxito `"X"` y `1d`. |
| `testAcceptObjIntConsumer()` | — | `void` | — | `FailableObjIntConsumer`: `"X"` y `1`. |
| `testAcceptObjLongConsumer()` | — | `void` | — | `FailableObjLongConsumer`: `"X"` y `1L`. |
| `testApplyBiFunction()` | — | `void` | — | `Failable.apply(FailableBiFunction, T, U)` propaga/envuelve igual que `accept` y retorna `0` en el camino exitoso. |
| `testApplyDoubleBinaryOperator()` | — | `void` | — | `Failable.applyAsDouble(FailableDoubleBinaryOperator, 1d, 2d)` lanza `IllegalStateException` o retorna `3d`. |
| `testApplyFunction()` | — | `void` | — | `Failable.apply(FailableFunction, T)`: propagación y retorno `0`. |
| `testApplyNonNull()` | — | `void` | `SomeException` | `Failable.applyNonNull(T, FailableFunction)`: mapea `"a"→"A"`, propaga `null` de entrada y de salida, `NullPointerException` si la función es `null`, y propaga `SomeException` (con `value == 1`) desde `FailureOnInvocationCount#inc`. |
| `testApplyNonNull2()` | — | `void` | `SomeException`, `IOException` | Variante de dos funciones encadenadas: cortocircuito con `null`, `NullPointerException` por función nula y propagación de excepciones iguales o distintas (`SomeException` directa, `IOException` con causa `SomeException`). |
| `testApplyNonNull3()` | — | `void` | `SomeException`, `IOException`, `ClassNotFoundException` | Variante de tres funciones (`" abc " → "CBA"`), mismos escenarios de `null`, `NullPointerException` y excepciones homogéneas/heterogéneas; usa `reset()` de los contadores. |

### 2.8 Métodos de prueba — adaptadores `Failable.as*`

| Método | @param | @return | @throws (declaradas) | Comportamiento verificado |
|---|---|---|---|---|
| `testAsCallable()` | — | `void` | — | `Failable.asCallable(FailableCallable)` envuelve `SomeException` en `UndeclaredThrowableException` (`"Odd Invocation: 1"`); la segunda llamada tiene éxito. Usa `Failable.rethrow(ex)` en el `catch`. |
| `testAsConsumer()` | — | `void` | — | `Failable.asConsumer(FailableConsumer)` genera un `java.util.function.Consumer` que propaga/envuelve según el tipo de `Throwable`. |
| `testAsRunnable()` | — | `void` | — | `Failable.asRunnable(FailableRunnable)`: primera ejecución lanza `UndeclaredThrowableException`; la segunda (par) no. |
| `testAsSupplier()` | — | `void` | — | `Failable.asSupplier(FailableSupplier)`: primera `get()` lanza `UndeclaredThrowableException`, la segunda retorna instancia no nula. |

### 2.9 Métodos de prueba — interfaces `Bi*`

| Método | @param | @return | @throws (declaradas) | Comportamiento verificado |
|---|---|---|---|---|
| `testBiConsumer()` | — | `void` | `Throwable` | `Failable.asBiConsumer`: propagación directa de `IllegalStateException`/`OutOfMemoryError`, envoltura de `IOException`, invocación directa del `FailableBiConsumer` y caso `null` sin excepción. |
| `testBiConsumerAndThen()` | — | `void` | `Throwable` | `FailableBiConsumer#andThen` ejecuta ambos consumidores (el error se propaga en cualquier orden), `nop().andThen(nop())` no lanza y `andThen(null)` lanza `NullPointerException`. |
| `testBiFunction()` | — | `void` | — | `Failable.asBiFunction`: propagación/envoltura y retorno `0`. |
| `testBiFunctionAndThen()` | — | `void` | `IOException` | `FailableBiFunction#andThen` con combinaciones *failing/nop*; `andThen(null)` → `NullPointerException`. Incluye el patrón JRE no comprobado como referencia. |
| `testBiPredicate()` | — | `void` | — | `Failable.asBiPredicate` produce un `BiPredicate` que lanza `UndeclaredThrowableException` en la invocación impar y `true` en la par. |
| `testBiPredicateAnd()` | — | `void` | `Throwable` | Tabla de verdad de `and()` sobre `FailableBiPredicate.TRUE/FALSE` y `NullPointerException` con argumento `null` (incluso en `falsePredicate()`). |
| `testBiPredicateNegate()` | — | `void` | `Throwable` | `negate()` sobre constantes y métodos factoría `truePredicate()` / `falsePredicate()`. |
| `testBiPredicateOr()` | — | `void` | `Throwable` | Tabla de verdad de `or()` y casos `null`. |

### 2.10 Métodos de prueba — consumidores y operadores primitivos

| Método | @param | @return | @throws (declaradas) | Comportamiento verificado |
|---|---|---|---|---|
| `testByteConsumerAndThen()` | — | `void` | `Throwable` | `FailableByteConsumer#andThen` en ambos órdenes con `accept((byte) 0)`; `andThen(null)` → `NullPointerException`. |
| `testCallable()` | — | `void` | — | `Failable.run(FailableRunnable)` envuelve en `UndeclaredThrowableException`; `Failable.call(FailableCallable)` retorna instancia en la invocación par. |
| `testConsumerAndThen()` | — | `void` | `Throwable` | `FailableConsumer#andThen` propaga el `Error`; `nop().andThen(nop())` no lanza; `andThen(null)` → `NullPointerException`. |
| `testDoubleConsumerAndThen()` | — | `void` | `Throwable` | Equivalente para `FailableDoubleConsumer` con `accept(0d)`. |
| `testDoublePredicate()` | — | `void` | `Throwable` | `FailableDoublePredicate` creado desde `FailureOnOddInvocations::testDouble`: lanza `SomeException` (impar) y luego funciona. |
| `testDoublePredicateAnd()` | — | `void` | `Throwable` | Tabla de verdad de `and()` + casos `null`. |
| `testDoublePredicateNegate()` | — | `void` | `Throwable` | `negate()` sobre constantes y factorías. |
| `testDoublePredicateOr()` | — | `void` | `Throwable` | Tabla de verdad de `or()` + casos `null`. |
| `testDoubleUnaryOperatorAndThen()` | — | `void` | `Throwable` | `FailableDoubleUnaryOperator#andThen` con `applyAsDouble(0d)`; `andThen(null)` → `NullPointerException`. |
| `testDoubleUnaryOperatorCompose()` | — | `void` | `Throwable` | Igual para `compose`; `compose(null)` → `NullPointerException`. |
| `testDoubleUnaryOperatorIdentity()` | — | `void` | `Throwable` | `identity()` admite `compose(identity())` y rechaza `compose(null)`. |
| `testIntConsumerAndThen()` | — | `void` | `Throwable` | `FailableIntConsumer#andThen` con `accept(0)`. |
| `testIntPredicate()` | — | `void` | `Throwable` | `FailableIntPredicate` desde `FailureOnOddInvocations::testInt`. |
| `testIntPredicateAnd()` | — | `void` | `Throwable` | Tabla de verdad de `and()` + `null`. |
| `testIntPredicateNegate()` | — | `void` | `Throwable` | `negate()` sobre constantes y factorías. |
| `testIntPredicateOr()` | — | `void` | `Throwable` | Tabla de verdad de `or()` + `null`. |
| `testIntUnaryOperatorAndThen()` | — | `void` | `Throwable` | `FailableIntUnaryOperator#andThen` con `applyAsInt(0)`. |
| `testIntUnaryOperatorCompose()` | — | `void` | `Throwable` | `compose` y caso `null`. |
| `testIntUnaryOperatorIdentity()` | — | `void` | `Throwable` | `identity()` y caso `null`. |
| `testLongConsumerAndThen()` | — | `void` | `Throwable` | `FailableLongConsumer#andThen` con `accept(0L)`. |
| `testLongPredicate()` | — | `void` | `Throwable` | `FailableLongPredicate` desde `FailureOnOddInvocations::testLong`. |
| `testLongPredicateAnd()` | — | `void` | `Throwable` | Tabla de verdad de `and()` + `null`. |
| `testLongPredicateNegate()` | — | `void` | `Throwable` | `negate()` sobre constantes y factorías. |
| `testLongPredicateOr()` | — | `void` | `Throwable` | Tabla de verdad de `or()` + `null`. |
| `testLongUnaryOperatorAndThen()` | — | `void` | `Throwable` | `FailableLongUnaryOperator#andThen` con `applyAsLong(0L)`. |
| `testLongUnaryOperatorCompose()` | — | `void` | `Throwable` | `compose` y caso `null`. |
| `testLongUnaryOperatorIdentity()` | — | `void` | `Throwable` | `identity()` y caso `null`. |

### 2.11 Métodos de prueba — implementaciones `nop()`

Todos declaran `throws Throwable`, no reciben parámetros y retornan `void`.

| Método | Elemento verificado | Resultado esperado |
|---|---|---|
| `testFailableBiFunctionNop()` | `FailableBiFunction.nop().apply("Foo", "Bar")` | `null` |
| `testFailableConsumerNop()` | `FailableConsumer.nop().accept("Foo")` | No lanza nada |
| `testFailableDoubleFunctionNop()` | `FailableDoubleFunction.nop().apply(Double.MAX_VALUE)` | `null` |
| `testFailableDoubleToIntFunctionNop()` | `applyAsInt(Double.MAX_VALUE)` | `0` |
| `testFailableDoubleToLongFunctionNop()` | `applyAsLong(Double.MAX_VALUE)` | `0` |
| `testFailableIntFunctionNop()` | `apply(Integer.MAX_VALUE)` | `null` |
| `testFailableIntToDoubleFunctionNop()` | `applyAsDouble(Integer.MAX_VALUE)` | `0` |
| `testFailableIntToFloatFunctionNop()` | `applyAsFloat(Integer.MAX_VALUE)` | `0` |
| `testFailableIntToLongFunctionNop()` | `applyAsLong(Integer.MAX_VALUE)` | `0` |
| `testFailableLongFunctionNop()` | `apply(Long.MAX_VALUE)` | `null` |
| `testFailableLongToDoubleFunctionNop()` | `applyAsDouble(Long.MAX_VALUE)` | `0` |
| `testFailableLongToIntFunctionNop()` | `applyAsInt(Long.MAX_VALUE)` | `0` |
| `testFailableObjDoubleConsumerNop()` | `accept("Foo", Double.MAX_VALUE)` | No lanza nada |
| `testFailableObjIntConsumerNop()` | `accept("Foo", Integer.MAX_VALUE)` | No lanza nada |
| `testFailableObjLongConsumerNop()` | `accept("Foo", Long.MAX_VALUE)` | No lanza nada |
| `testFailableToBooleanFunctionNop()` | `applyAsBoolean("Foo")` | `false` |
| `testFailableToDoubleBiFunctionNop()` | `applyAsDouble("Foo", "Bar")` | `0` |
| `testFailableToDoubleFunctionNop()` | `applyAsDouble("Foo")` | `0` |
| `testFailableToIntBiFunctionNop()` | `applyAsInt("Foo", "Bar")` | `0` |
| `testFailableToIntFunctionNop()` | `applyAsInt("Foo")` | `0` |
| `testFailableToLongBiFunctionNop()` | `applyAsLong("Foo", "Bar")` | `0` |
| `testFailableToLongFunctionNop()` | `applyAsLong("Foo")` | `0` |

### 2.12 Métodos de prueba — `FailableFunction`, proveedores y `tryWithResources`

| Método | @param | @return | @throws (declaradas) | Comportamiento verificado |
|---|---|---|---|---|
| `testFunction()` | — | `void` | — | `Failable.asFunction(FailableFunction)` propaga `IllegalStateException`/`OutOfMemoryError`, envuelve `IOException` y retorna `0`. |
| `testFunctionAndThen()` | — | `void` | `IOException` | `FailableFunction#andThen` con combinaciones *failing/nop* y `andThen(null)` → `NullPointerException`. |
| `testFunctionCompose()` | — | `void` | `Throwable` | `FailableFunction#compose` en ambos órdenes y `compose(null)` → `NullPointerException`. |
| `testFunctionFunction()` | — | `void` | `Exception` | `FailableFunction.function(this::throwingFunction).andThen(this::throwingFunction).apply("foo")` devuelve `"foo"`. |
| `testFunctionIdentity()` | — | `void` | `Throwable` | `FailableFunction.identity()` admite `compose(identity())` y rechaza `compose(null)`. |
| `testGetAsBooleanSupplier()` | — | `void` | — | `Failable.getAsBoolean(FailableBooleanSupplier)`: propagación/envoltura y `false` final. |
| `testGetAsDoubleSupplier()` | — | `void` | — | `Failable.getAsDouble(...)`: propagación/envoltura y `0`. |
| `testGetAsIntSupplier()` | — | `void` | — | `Failable.getAsInt(...)` con `testAsIntPrimitive` para los fallos y `testAsInteger` (auto-unboxing) para el éxito; resultado `0`. |
| `testGetAsLongSupplier()` | — | `void` | — | `Failable.getAsLong(...)`: propagación/envoltura y `0`. |
| `testGetAsShortSupplier()` | — | `void` | — | `Failable.getAsShort(...)`: propagación/envoltura y `0`. |
| `testGetFromSupplier()` | — | `void` | — | Mismo escenario que `testCallable()`: `Failable.run` envuelve en `UndeclaredThrowableException` y `Failable.call` tiene éxito en la invocación par. |
| `testGetSupplier()` | — | `void` | — | `Failable.get(FailableSupplier)`: propagación/envoltura; el valor final se obtiene vía `Failable.apply(Testable::testAsInteger, testable)` y es `0`. |
| `testPredicate()` | — | `void` | — | `Failable.asPredicate(FailablePredicate)`: `UndeclaredThrowableException` en impar, `true` en par. |
| `testPredicateAnd()` | — | `void` | `Throwable` | Tabla de verdad de `and()` sobre `FailablePredicate.TRUE/FALSE` y `NullPointerException` con `null`. |
| `testPredicateNegate()` | — | `void` | `Throwable` | `negate()` sobre constantes y factorías. |
| `testPredicateOr()` | — | `void` | `Throwable` | Tabla de verdad de `or()` y casos `null`. |
| `testRunnable()` | — | `void` | — | `Failable.run(FailureOnOddInvocations::new)` lanza `UndeclaredThrowableException` en impar; la par no lanza; `Failable.run(null)` se tolera. |
| `testTryWithResources()` | — | `void` | — | `Failable.tryWithResources(action, closeable::close)` cierra siempre el recurso (`isClosed()` verdadero), propaga `IllegalStateException`/`OutOfMemoryError` tal cual y envuelve `IOException` en `UncheckedIOException` conservando la causa. |

### 2.13 Método de apoyo privado

#### `throwingFunction(String)`

```java
/**
 * Función identidad cuya firma declara una excepción comprobada, usada como referencia de
 * método en {@code testFunctionFunction()} para validar FailableFunction.function(...).
 *
 * @param input cadena de entrada.
 * @return la misma cadena recibida.
 * @throws Exception declarada en la firma; nunca se lanza en la implementación actual.
 */
private String throwingFunction(final String input) throws Exception
```

| @param | Tipo | Requerido |
|---|---|---|
| `input` | `String` | Sí (admite `null` por contrato de identidad) |

| @return | Descripción | @throws |
|---|---|---|
| `String` | El argumento `input` sin modificar. | `Exception` (declarada, no lanzada). |

---

### 2.14 Métodos de prueba `testThrows_*` y sus clases anónimas

Cada método `testThrows_*` es `@Test`, `void`, **sin parámetros** y **sin excepciones declaradas**. Todos siguen el mismo patrón: instancian una clase anónima que implementa la interfaz failable correspondiente, lanzan `new IOException("test")` dentro del método abstracto y comprueban con `assertThrows(IOException.class, ...)` que la excepción atraviesa la abstracción sin envolverse.

| Método de prueba (`@Test`, `void`, sin `@param`, sin `@throws` declaradas) | Tipo anónimo instanciado | Método sobrescrito (`@Override`) | @param del método anónimo | @return | @throws |
|---|---|---|---|---|---|
| `testThrows_FailableBiConsumer_Object_Throwable()` | `FailableBiConsumer<Object,Object,Throwable>` | `accept(Object object1, Object object2)` | `object1`:`Object`; `object2`:`Object` | `void` | `Throwable` (`IOException`) |
| `testThrows_FailableBiConsumer_String_IOException()` | `FailableBiConsumer<String,String,IOException>` | `accept(String object1, String object2)` | `object1`:`String`; `object2`:`String` | `void` | `IOException` |
| `testThrows_FailableBiFunction_Object_Throwable()` | `FailableBiFunction<Object,Object,Object,Throwable>` | `apply(Object input1, Object input2)` | `input1`:`Object`; `input2`:`Object` | `Object` | `Throwable` |
| `testThrows_FailableBiFunction_String_IOException()` | `FailableBiFunction<String,String,String,IOException>` | `apply(String input1, String input2)` | `input1`:`String`; `input2`:`String` | `String` | `IOException` |
| `testThrows_FailableBiPredicate_Object_Throwable()` | `FailableBiPredicate<Object,Object,Throwable>` | `test(Object object1, Object object2)` | `object1`:`Object`; `object2`:`Object` | `boolean` | `Throwable` |
| `testThrows_FailableBiPredicate_String_IOException()` | `FailableBiPredicate<String,String,IOException>` | `test(String object1, String object2)` | `object1`:`String`; `object2`:`String` | `boolean` | `IOException` |
| `testThrows_FailableBooleanSupplier_IOException()` | `FailableBooleanSupplier<IOException>` | `getAsBoolean()` | — | `boolean` | `IOException` |
| `testThrows_FailableBooleanSupplier_Throwable()` | `FailableBooleanSupplier<Throwable>` | `getAsBoolean()` | — | `boolean` | `Throwable` |
| `testThrows_FailableByteConsumer_IOException()` | `FailableByteConsumer<IOException>` | `accept(byte value)` | `value`:`byte` | `void` | `IOException` |
| `testThrows_FailableByteConsumer_Throwable()` | `FailableByteConsumer<Throwable>` | `accept(byte value)` | `value`:`byte` | `void` | `Throwable` |
| `testThrows_FailableByteSupplier_IOException()` | `FailableByteSupplier<IOException>` | `getAsByte()` | — | `byte` | `IOException` |
| `testThrows_FailableByteSupplier_Throwable()` | `FailableByteSupplier<Throwable>` | `getAsByte()` | — | `byte` | `Throwable` |
| `testThrows_FailableCallable_Object_Throwable()` | `FailableCallable<Object,Throwable>` | `call()` | — | `Object` | `Throwable` |
| `testThrows_FailableCallable_String_IOException()` | `FailableCallable<String,IOException>` | `call()` | — | `String` | `IOException` |
| `testThrows_FailableConsumer_Object_Throwable()` | `FailableConsumer<Object,Throwable>` | `accept(Object object)` | `object`:`Object` | `void` | `Throwable` |
| `testThrows_FailableConsumer_String_IOException()` | `FailableConsumer<String,IOException>` | `accept(String object)` | `object`:`String` | `void` | `IOException` |
| `testThrows_FailableDoubleBinaryOperator_IOException()` | `FailableDoubleBinaryOperator<IOException>` | `applyAsDouble(double left, double right)` | `left`:`double`; `right`:`double` | `double` | `IOException` |
| `testThrows_FailableDoubleBinaryOperator_Throwable()` | `FailableDoubleBinaryOperator<Throwable>` | `applyAsDouble(double left, double right)` | `left`:`double`; `right`:`double` | `double` | `Throwable` |
| `testThrows_FailableDoubleConsumer_IOException()` | `FailableDoubleConsumer<IOException>` | `accept(double value)` | `value`:`double` | `void` | `IOException` |
| `testThrows_FailableDoubleConsumer_Throwable()` | `FailableDoubleConsumer<Throwable>` | `accept(double value)` | `value`:`double` | `void` | `Throwable` |
| `testThrows_FailableDoubleFunction_IOException()` | `FailableDoubleFunction<String,IOException>` | `apply(double input)` | `input`:`double` | `String` | `IOException` |
| `testThrows_FailableDoubleFunction_Throwable()` | `FailableDoubleFunction<Object,Throwable>` | `apply(double input)` | `input`:`double` | `Object` | `Throwable` |
| `testThrows_FailableDoubleSupplier_IOException()` | `FailableDoubleSupplier<IOException>` | `getAsDouble()` | — | `double` | `IOException` |
| `testThrows_FailableDoubleSupplier_Throwable()` | `FailableDoubleSupplier<Throwable>` | `getAsDouble()` | — | `double` | `Throwable` |
| `testThrows_FailableDoubleToIntFunction_IOException()` | `FailableDoubleToIntFunction<IOException>` | `applyAsInt(double value)` | `value`:`double` | `int` | `IOException` |
| `testThrows_FailableDoubleToIntFunction_Throwable()` | `FailableDoubleToIntFunction<Throwable>` | `applyAsInt(double value)` | `value`:`double` | `int` | `Throwable` |
| `testThrows_FailableDoubleToLongFunction_IOException()` | `FailableDoubleToLongFunction<IOException>` | `applyAsLong(double value)` | `value`:`double` | `int` *(ver nota)* | `IOException` |
| `testThrows_FailableDoubleToLongFunction_Throwable()` | `FailableDoubleToLongFunction<Throwable>` | `applyAsLong(double value)` | `value`:`double` | `int` *(ver nota)* | `Throwable` |
| `testThrows_FailableFunction_Object_Throwable()` | `FailableFunction<Object,Object,Throwable>` | `apply(Object input)` | `input`:`Object` | `Object` | `Throwable` |
| `testThrows_FailableFunction_String_IOException()` | `FailableFunction<String,String,IOException>` | `apply(String input)` | `input`:`String` | `String` | `IOException` |
| `testThrows_FailableIntBinaryOperator_IOException()` | `FailableIntBinaryOperator<IOException>` | `applyAsInt(int left, int right)` | `left`:`int`; `right`:`int` | `int` | `IOException` |
| `testThrows_FailableIntBinaryOperator_Throwable()` | `FailableIntBinaryOperator<Throwable>` | `applyAsInt(int left, int right)` | `left`:`int`; `right`:`int` | `int` | `Throwable` |
| `testThrows_FailableIntConsumer_IOException()` | `FailableIntConsumer<IOException>` | `accept(int value)` | `value`:`int` | `void` | `IOException` |
| `testThrows_FailableIntConsumer_Throwable()` | `FailableIntConsumer<Throwable>` | `accept(int value)` | `value`:`int` | `void` | `Throwable` |
| `testThrows_FailableIntFunction_Object_Throwable()` | `FailableIntFunction<Object,Throwable>` | `apply(int input)` | `input`:`int` | `Object` | `Throwable` |
| `testThrows_FailableIntFunction_String_IOException()` | `FailableIntFunction<String,IOException>` | `apply(int input)` | `input`:`int` | `String` | `IOException` |
| `testThrows_FailableIntSupplier_IOException()` | `FailableIntSupplier<IOException>` | `getAsInt()` | — | `int` | `IOException` |
| `testThrows_FailableIntSupplier_Throwable()` | `FailableIntSupplier<Throwable>` | `getAsInt()` | — | `int` | `Throwable` |
| `testThrows_FailableIntToDoubleFunction_IOException()` | `FailableIntToDoubleFunction<IOException>` | `applyAsDouble(int value)` | `value`:`int` | `double` | `IOException` |
| `testThrows_FailableIntToDoubleFunction_Throwable()` | `FailableIntToDoubleFunction<Throwable>` | `applyAsDouble(int value)` | `value`:`int` | `double` | `Throwable` |
| `testThrows_FailableIntToFloatFunction_IOException()` | `FailableIntToFloatFunction<IOException>` | `applyAsFloat(int value)` | `value`:`int` | `float` | `IOException` |
| `testThrows_FailableIntToFloatFunction_Throwable()` | `FailableIntToFloatFunction<Throwable>` | `applyAsFloat(int value)` | `value`:`int` | `float` | `Throwable` |
| `testThrows_FailableIntToLongFunction_IOException()` | `FailableIntToLongFunction<IOException>` | `applyAsLong(int value)` | `value`:`int` | `long` | `IOException` |
| `testThrows_FailableIntToLongFunction_Throwable()` | `FailableIntToLongFunction<Throwable>` | `applyAsLong(int value)` | `value`:`int` | `long` | `Throwable` |
| `testThrows_FailableLongBinaryOperator_IOException()` | `FailableLongBinaryOperator<IOException>` | `applyAsLong(long left, long right)` | `left`:`long`; `right`:`long` | `long` | `IOException` |
| `testThrows_FailableLongBinaryOperator_Throwable()` | `FailableLongBinaryOperator<Throwable>` | `applyAsLong(long left, long right)` | `left`:`long`; `right`:`long` | `long` | `Throwable` |
| `testThrows_FailableLongConsumer_IOException()` | `FailableLongConsumer<IOException>` | `accept(long object)` | `object`:`long` | `void` | `IOException` |
| `testThrows_FailableLongConsumer_Throwable()` | `FailableLongConsumer<Throwable>` | `accept(long object)` | `object`:`long` | `void` | `Throwable` |
| `testThrows_FailableLongFunction_IOException()` | `FailableLongFunction<String,IOException>` | `apply(long input)` | `input`:`long` | `String` | `IOException` |
| `testThrows_FailableLongFunction_Throwable()` | `FailableLongFunction<Object,Throwable>` | `apply(long input)` | `input`:`long` | `Object` | `Throwable` |
| `testThrows_FailableLongSupplier_IOException()` | `FailableLongSupplier<IOException>` | `getAsLong()` | — | `long` | `IOException` |
| `testThrows_FailableLongSupplier_Throwable()` | `FailableLongSupplier<Throwable>` | `getAsLong()` | — | `long` | `Throwable` |
| `testThrows_FailableLongToDoubleFunction_IOException()` | `FailableLongToDoubleFunction<IOException>` | `applyAsDouble(long value)` | `value`:`long` | `double` | `IOException` |
| `testThrows_FailableLongToDoubleFunction_Throwable()` | `FailableLongToDoubleFunction<Throwable>` | `applyAsDouble(long value)` | `value`:`long` | `double` | `Throwable` |
| `testThrows_FailableLongToIntFunction_IOException()` | `FailableLongToIntFunction<IOException>` | `applyAsInt(long value)` | `value`:`long` | `int` | `IOException` |
| `testThrows_FailableLongToIntFunction_Throwable()` | `FailableLongToIntFunction<Throwable>` | `applyAsInt(long value)` | `value`:`long` | `int` | `Throwable` |
| `testThrows_FailableObjDoubleConsumer_Object_Throwable()` | `FailableObjDoubleConsumer<Object,Throwable>` | `accept(Object object, double value)` | `object`:`Object`; `value`:`double` | `void` | `Throwable` |
| `testThrows_FailableObjDoubleConsumer_String_IOException()` | `FailableObjDoubleConsumer<String,IOException>` | `accept(String object, double value)` | `object`:`String`; `value`:`double` | `void` | `IOException` |
| `testThrows_FailableObjIntConsumer_Object_Throwable()` | `FailableObjIntConsumer<Object,Throwable>` | `accept(Object object, int value)` | `object`:`Object`; `value`:`int` | `void` | `Throwable` |
| `testThrows_FailableObjIntConsumer_String_IOException()` | `FailableObjIntConsumer<String,IOException>` | `accept(String object, int value)` | `object`:`String`; `value`:`int` | `void` | `IOException` |
| `testThrows_FailableObjLongConsumer_Object_Throwable()` | `FailableObjLongConsumer<Object,Throwable>` | `accept(Object object, long value)` | `object`:`Object`; `value`:`long` | `void` | `Throwable` |
| `testThrows_FailableObjLongConsumer_String_IOException()` | `FailableObjLongConsumer<String,IOException>` + lambda `f` | `accept(String object, long value)` y lambda `(o, l) -> { throw new IOException("test"); }` | `object`/`o`:`String`; `value`/`l`:`long` | `void` | `IOException` |
| `testThrows_FailablePredicate_Object_Throwable()` | `FailablePredicate<Object,Throwable>` | `test(Object object)` | `object`:`Object` | `boolean` | `Throwable` |
| `testThrows_FailablePredicate_String_IOException()` | `FailablePredicate<String,IOException>` | `test(String object)` | `object`:`String` | `boolean` | `IOException` |
| `testThrows_FailableRunnable_IOException()` | `FailableRunnable<IOException>` | `run()` | — | `void` | `IOException` |
| `testThrows_FailableRunnable_Throwable()` | `FailableRunnable<Throwable>` | `run()` | — | `void` | `Throwable` |
| `testThrows_FailableShortSupplier_IOException()` | `FailableShortSupplier<IOException>` | `getAsShort()` | — | `short` | `IOException` |
| `testThrows_FailableShortSupplier_Throwable()` | `FailableShortSupplier<Throwable>` | `getAsShort()` | — | `short` | `Throwable` |
| `testThrows_FailableSupplier_Object_Throwable()` | `FailableSupplier<Object,Throwable>` | `get()` | — | `Object` | `Throwable` |
| `testThrows_FailableSupplier_String_IOException()` | `FailableSupplier<String,IOException>` | `get()` | — | `String` | `IOException` |
| `testThrows_FailableToBooleanFunction_Object_Throwable()` | `FailableToBooleanFunction<Object,Throwable>` | `applyAsBoolean(Object t)` | `t`:`Object` | `boolean` | `Throwable` |
| `testThrows_FailableToBooleanFunction_String_IOException()` | `FailableToBooleanFunction<String,IOException>` | `applyAsBoolean(String t)` | `t`:`String` | `boolean` | `IOException` |
| `testThrows_FailableToDoubleBiFunction_Object_Throwable()` | `FailableToDoubleBiFunction<Object,Object,Throwable>` | `applyAsDouble(Object t, Object u)` | `t`:`Object`; `u`:`Object` | `double` | `Throwable` |
| `testThrows_FailableToDoubleBiFunction_String_IOException()` | `FailableToDoubleBiFunction<String,String,IOException>` | `applyAsDouble(String t, String u)` | `t`:`String`; `u`:`String` | `double` | `IOException` |
| `testThrows_FailableToDoubleFunction_Object_Throwable()` | `FailableToDoubleFunction<Object,Throwable>` | `applyAsDouble(Object t)` | `t`:`Object` | `double` | `Throwable` |
| `testThrows_FailableToDoubleFunction_String_IOException()` | `FailableToDoubleFunction<String,IOException>` | `applyAsDouble(String t)` | `t`:`String` | `double` | `IOException` |
| `testThrows_FailableToIntBiFunction_Object_Throwable()` | `FailableToIntBiFunction<Object,Object,Throwable>` | `applyAsInt(Object t, Object u)` | `t`:`Object`; `u`:`Object` | `int` | `Throwable` |
| `testThrows_FailableToIntBiFunction_String_IOException()` | `FailableToIntBiFunction<String,String,IOException>` | `applyAsInt(String t, String u)` | `t`:`String`; `u`:`String` | `int` | `IOException` |
| `testThrows_FailableToIntFunction_Object_Throwable()` | `FailableToIntFunction<Object,Throwable>` | `applyAsInt(Object t)` | `t`:`Object` | `int` | `Throwable` |
| `testThrows_FailableToIntFunction_String_IOException()` | `FailableToIntFunction<String,IOException>` | `applyAsInt(String t)` | `t`:`String` | `int` | `IOException` |
| `testThrows_FailableToLongBiFunction_Object_Throwable()` | `FailableToLongBiFunction<Object,Object,Throwable>` | `applyAsLong(Object t, Object u)` | `t`:`Object`; `u`:`Object` | `long` | `Throwable` |
| `testThrows_FailableToLongBiFunction_String_IOException()` | `FailableToLongBiFunction<String,String,IOException>` | `applyAsLong(String t, String u)` | `t`:`String`; `u`:`String` | `long` | `IOException` |
| `testThrows_FailableToLongFunction_Object_Throwable()` | `FailableToLongFunction<Object,Throwable>` | `applyAsLong(Object t)` | `t`:`Object` | `long` | `Throwable` |
| `testThrows_FailableToLongFunction_String_IOException()` | `FailableToLongFunction<String,IOException>` | `applyAsLong(String t)` | `t`:`String` | `long` | `IOException` |

> **Nota sobre `applyAsLong(double)`:** en `testThrows_FailableDoubleToLongFunction_IOException()` y `testThrows_FailableDoubleToLongFunction_Throwable()` el método sobrescrito se declara con tipo de retorno `int`. Dado que Java exige retorno idéntico para primitivos en una sobrescritura, esto implica que la interfaz `FailableDoubleToLongFunction` del paquete declara `int applyAsLong(double)`. Se documenta tal como aparece en el código fuente.

### 2.15 Implementaciones funcionales locales (lambdas y referencias a método)

Elementos funcionales declarados dentro de los métodos de prueba; se documentan por completitud, ya que constituyen implementaciones anónimas del método abstracto de cada interfaz.

| Método contenedor | Variable local | Interfaz implementada | Comportamiento |
|---|---|---|---|
| `testAsCallable()` | `failableCallable` | `FailableCallable<FailureOnOddInvocations,SomeException>` | Referencia `FailureOnOddInvocations::new`. |
| `testAsSupplier()` | `failableSupplier` | `FailableSupplier<FailureOnOddInvocations,Throwable>` | Referencia `FailureOnOddInvocations::new`. |
| `testBiConsumer()` / `testBiConsumerAndThen()` | `failableBiConsumer` / `failing` | `FailableBiConsumer<Testable<?,?>,Throwable,Throwable>` | `(t, th) -> { t.setThrowable(th); t.test(); }`. |
| `testBiFunction()` | `failableBiFunction` | `FailableBiFunction<Testable<?,?>,Throwable,Integer,Throwable>` | `(t, th) -> { t.setThrowable(th); return t.testAsInteger(); }`. |
| `testBiFunctionAndThen()` | `nopBiFunction`, `nopFunction`, `failingBiFunctionTest`, `failingFunction` | `BiFunction`, `Function`, `FailableBiFunction`, `FailableFunction` | Retornan `null` o lanzan `IOException`. |
| `testBiPredicate()` | `failableBiPredicate` | `FailableBiPredicate<Object,Object,Throwable>` | `(t1, t2) -> FailureOnOddInvocations.testGetBool()`. |
| `testByteConsumerAndThen()`, `testDoubleConsumerAndThen()`, `testIntConsumerAndThen()`, `testLongConsumerAndThen()` | `failing` | `FailableByte/Double/Int/LongConsumer<Throwable>` | Configura `ERROR` en el `Testable` y lo lanza vía `test()`. |
| `testConsumerAndThen()` | `failableConsumer` | `FailableConsumer<Throwable,Throwable>` | `th -> { testable.setThrowable(th); testable.test(); }`. |
| `testDoublePredicate()`, `testIntPredicate()`, `testLongPredicate()` | `failablePredicate` | `FailableDouble/Int/LongPredicate<Throwable>` | Referencias a `FailureOnOddInvocations::testDouble/testInt/testLong`. |
| `testDoubleUnaryOperatorAndThen()/Compose()`, `testIntUnaryOperator*()`, `testLongUnaryOperator*()` | `failing` | `FailableDouble/Int/LongUnaryOperator<Throwable>` | Lanza `ERROR` y retorna `0`/`0d`/`0L`. |
| `testFunction()` | `failableFunction` | `FailableFunction<Throwable,Integer,Throwable>` | `th -> { testable.setThrowable(th); return testable.testAsInteger(); }`. |
| `testFunctionAndThen()` | `nopFunction`, `failingFunction` | `Function`, `FailableFunction<Object,Integer,IOException>` | Retorna `null` / lanza `IOException`. |
| `testFunctionCompose()` | `failing` | `FailableFunction<Object,Integer,Throwable>` | Lanza `ERROR` y retorna `0`. |
| `testApplyNonNull*()` | lambdas `s -> null` | `FailableFunction` | Simulan resultado nulo intermedio. |
| `testTryWithResources()` | `consumer` | `FailableConsumer<Throwable, ? extends Throwable>` | Referencia `closeable::run`. |

---

## 3. Ejemplo de uso

El siguiente ejemplo muestra cómo reutilizar los dobles de prueba de esta clase (`Testable`, `FailureOnOddInvocations`, `CloseableObject`, `SomeException`) para escribir un caso real que valide la política de reenvío de excepciones de `Failable`.

```java
package org.apache.commons.lang3.function;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertSame;
import static org.junit.jupiter.api.Assertions.assertThrows;
import static org.junit.jupiter.api.Assertions.assertTrue;

import java.io.IOException;
import java.io.UncheckedIOException;
import java.lang.reflect.UndeclaredThrowableException;

import org.apache.commons.lang3.function.FailableTest.CloseableObject;
import org.apache.commons.lang3.function.FailableTest.FailureOnOddInvocations;
import org.apache.commons.lang3.function.FailableTest.SomeException;
import org.apache.commons.lang3.function.FailableTest.Testable;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

class FailableUsageExampleTest {

    private static final IllegalStateException RUNTIME = new IllegalStateException();

    @BeforeEach
    void setUp() {
        // Equivalente a FailableTest#beforeEach()
        FailureOnOddInvocations.reset();
    }

    @Test
    void unaRuntimeExceptionSePropagaSinEnvolver() {
        // Testable(Throwable) configura el fallo; Testable#test() lo relanza.
        final Testable<?, ?> testable = new Testable<>(RUNTIME);
        final Throwable actual = assertThrows(IllegalStateException.class,
            () -> Failable.accept(Testable::test, testable));
        assertSame(RUNTIME, actual);
    }

    @Test
    void unaIOExceptionSeEnvuelveEnUncheckedIOException() {
        final IOException ioe = new IOException("disco lleno");
        final Testable<?, Integer> testable = new Testable<>(ioe);

        // Testable#setThrowable(Throwable) permite reconfigurar el doble.
        final UncheckedIOException wrapper = assertThrows(UncheckedIOException.class,
            () -> Failable.accept(testable::testInt, 42));
        assertSame(ioe, wrapper.getCause());

        // Camino exitoso: Testable#getAcceptedPrimitiveObject1() registra el argumento.
        testable.setThrowable(null);
        Failable.accept(testable::testInt, 42);
        assertEquals(42, testable.getAcceptedPrimitiveObject1());
    }

    @Test
    void unaCheckedExceptionArbitrariaSeEnvuelveEnUndeclaredThrowableException() {
        // FailureOnOddInvocations() falla en la invocación impar con SomeException.
        final UndeclaredThrowableException e = assertThrows(UndeclaredThrowableException.class,
            () -> Failable.run(FailureOnOddInvocations::new));
        final SomeException cause = (SomeException) e.getCause();
        assertEquals("Odd Invocation: 1", cause.getMessage());

        // La invocación par (segunda) no lanza.
        Failable.run(FailureOnOddInvocations::new);
    }

    @Test
    void elRecursoSeCierraAunqueLaAccionFalle() {
        final CloseableObject closeable = new CloseableObject();
        // CloseableObject#run(Throwable) lanza el argumento recibido.
        assertThrows(IllegalStateException.class,
            () -> Failable.tryWithResources(() -> closeable.run(RUNTIME), closeable::close));
        assertTrue(closeable.isClosed());

        // CloseableObject#reset() reinicia el indicador para una nueva verificación.
        closeable.reset();
        Failable.tryWithResources(() -> closeable.run(null), closeable::close);
        assertTrue(closeable.isClosed());
    }
}
```

---

## 4. Observaciones de arquitectura

1. **Política de reenvío validada:** las pruebas documentan de forma ejecutable el contrato de `Failable.rethrow`: `RuntimeException` y `Error` se propagan intactos (`assertSame`), `IOException` se envuelve en `UncheckedIOException` preservando la causa, y cualquier otra `checked exception` se envuelve en `UndeclaredThrowableException`.
2. **Casos borde con `null` documentados:** el patrón repetido `assertNullPointerException(() -> x.andThen(null))` fija el comportamiento declarado en el Javadoc de las interfaces (el `NullPointerException` se produce al **componer**, no al **invocar**).
3. **Aislamiento de estado:** `FailureOnOddInvocations` utiliza un contador `static`; el método `beforeEach()` y las asignaciones explícitas `FailureOnOddInvocations.invocations = 0` dentro de varios tests son indispensables para la reproducibilidad. Esto hace que la clase **no sea segura** para ejecución paralela de pruebas.
4. **Miembros no referenciados:** `FailureOnInvocationCount#incCnfe(int)` (y su auxiliar `throwCnfeOn()`), `FailureOnOddInvocations#testInc(int)`, `FailureOnOddInvocations#getAsBoolean()`, `SomeException(String)`, `SomeException#test()` y `Testable#test(Object, Object)` no son utilizados por los casos de prueba del fuente provisto; constituyen puntos de extensión para futuras pruebas.
5. **Firmas con `throws` declarados y no lanzados:** el constructor `FailureOnInvocationCount(int)` y el método `throwingFunction(String)` declaran excepciones comprobadas que nunca lanzan; esto es intencional para que puedan adaptarse a las interfaces *failable* y verificar la inferencia del parámetro genérico de excepción.