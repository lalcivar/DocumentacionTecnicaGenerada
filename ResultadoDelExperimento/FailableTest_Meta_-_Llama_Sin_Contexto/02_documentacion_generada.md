# Documentación Técnica - Clase `FailableTest`

### 1. Ficha Técnica

| Atributo | Detalle |
| :--- | :--- |
| **Nombre de la Clase** | `FailableTest` |
| **Paquete** | `org.apache.commons.lang3.function` |
| **Herencia** | `extends AbstractLangTest` |
| **Tipo** | Clase de Pruebas Unitarias - JUnit 5 (Jupiter) |
| **Licencia** | Apache License, Version 2.0 |
| **Dependencia Principal** | `org.apache.commons.lang3.function.Failable` |
| **Framework de Pruebas** | JUnit 5, Mockito implícito |

> **Nota Importante:** El código proporcionado no corresponde a la clase productiva `Failable`, sino a su **suite de pruebas unitarias** `FailableTest`. Su objetivo es validar el correcto funcionamiento de la clase utilitaria `Failable` y de todas las interfaces `Failable*` del paquete `org.apache.commons.lang3.function`.

### 2. Descripción General

`FailableTest` es una clase de test integral que verifica el comportamiento de la utilidad `Failable` de Apache Commons Lang 3.

La clase `Failable` es una utilidad que permite trabajar con interfaces funcionales de Java 8+ (`Consumer`, `Function`, `Predicate`, `Supplier`, `BiConsumer`, etc.) que declaran excepciones comprobadas `checked exceptions`. Normalmente las interfaces funcionales estándar de `java.util.function` no permiten lanzar `IOException` o `Exception`, lo que obliga a capturarlas. `Failable` soluciona esto mediante wrappers que convierten excepciones comprobadas en no comprobadas `UncheckedIOException`, `UndeclaredThrowableException` o las relanzan directamente.

`FailableTest` garantiza que:
1.  Las excepciones se propaguen correctamente según su tipo.
2.  La conversión de interfaces `Failable` a interfaces estándar de Java sea correcta.
3.  Los métodos de composición funcional `andThen`, `compose`, `and`, `or`, `negate` funcionen con excepciones.
4.  Los casos nulos y de error se manejen según la especificación.

### 3. Clases Auxiliares Internas

Para simular diferentes escenarios de fallo, la clase define 5 clases estáticas internas:

*   **`CloseableObject`**: Simula un recurso cerrable para probar `Failable.tryWithResources()`. Contiene métodos `close()`, `isClosed()` y `run(Throwable)`.
*   **`FailureOnInvocationCount`**: Simula fallos en una invocación específica `throwOn`. Permite probar métodos `inc()`, `incIo()`, `incCnfe()` que lanzan `SomeException`, `IOException` y `ClassNotFoundException`.
*   **`FailureOnOddInvocations`**: Lanza `SomeException` en invocaciones impares. Usada para probar `FailableCallable`, `FailableRunnable` y `FailableSupplier`.
*   **`SomeException extends Exception`**: Excepción comprobada personalizada para las pruebas.
*   **`Testable<T, P>`**: Clase genérica principal para pruebas. Contiene múltiples métodos `test*` que pueden lanzar un `Throwable` configurable. Permite verificar `accept`, `apply`, `getAs*` con diferentes tipos primitivos y objetos.

Constantes de prueba:
*   `ERROR: OutOfMemoryError` - Para probar propagación de `Error`.
*   `ILLEGAL_STATE_EXCEPTION: IllegalStateException` - Para probar propagación de `RuntimeException`.

### 4. Métodos Principales

La clase contiene más de 80 métodos de prueba agrupados en 6 categorías funcionales:

#### 4.1 Inicialización
*   `beforeEach()`: Anotado con `@BeforeEach`. Resetea el contador de `FailureOnOddInvocations` antes de cada test para garantizar aislamiento.

#### 4.2 Pruebas de Ejecución Directa - `Failable.*`
Verifican los métodos estáticos que ejecutan una interfaz failable y gestionan la excepción.

| Método | Descripción |
| :--- | :--- |
| `testAccept*()` | Prueba `Failable.accept()` para `Consumer`, `BiConsumer`, `IntConsumer`, `LongConsumer`, `DoubleConsumer`, `ObjIntConsumer`, `ObjLongConsumer`, `ObjDoubleConsumer`. Verifica que `IOException` se convierta en `UncheckedIOException` y que `RuntimeException`/`Error` se relancen sin envolver. |
| `testApply*()` | Prueba `Failable.apply()` y `Failable.applyAsDouble()` para `Function` y `BiFunction`. |
| `testGet*()` | Prueba `Failable.get()`, `getAsInt()`, `getAsLong()`, `getAsDouble()`, `getAsBoolean()`, `getAsShort()` para `Supplier` y variantes primitivas. |
| `testCallable()` / `testRunnable()` / `testGetFromSupplier()` | Prueba `Failable.call()` y `Failable.run()` que envuelven `FailableCallable`/`FailableRunnable` en `UndeclaredThrowableException`. |
| `testApplyNonNull()` / `testApplyNonNull2()` / `testApplyNonNull3()` | Prueba la ejecución encadenada de 1, 2 y 3 funciones solo si el valor no es nulo. Valida el manejo de `null` y la propagación de excepciones mixtas. |
| `testTryWithResources()` | Prueba `Failable.tryWithResources()` como alternativa a try-with-resources que permite lanzar excepciones comprobadas. |

#### 4.3 Pruebas de Conversión - `Failable.as*()`
Verifican la conversión de una interfaz failable a su equivalente estándar de Java.

*   `testAsConsumer()`, `testAsCallable()`, `testAsRunnable()`, `testAsSupplier()`
*   `testBiConsumer()`, `testBiFunction()`, `testBiPredicate()`, `testPredicate()`
*   `testFunction()`

Todas validan que al invocar la interfaz estándar resultante, la excepción comprobada se transforme correctamente.

#### 4.4 Pruebas de Composición Funcional
Verifican los métodos por defecto de las interfaces failable `andThen`, `compose`, `and`, `or`, `negate`, `identity`, `nop`.

*   `testConsumerAndThen()`, `testBiConsumerAndThen()`, `testFunctionAndThen()`, `testBiFunctionAndThen()`, `testFunctionCompose()`
*   `testIntConsumerAndThen()`, `testLongConsumerAndThen()`, `testDoubleConsumerAndThen()`, `testByteConsumerAndThen()`
*   `testIntUnaryOperatorAndThen/Compose/Identity()`, `testLongUnaryOperator*`, `testDoubleUnaryOperator*`
*   `testPredicateAnd/Or/Negate()`, `testIntPredicate*`, `testLongPredicate*`, `testDoublePredicate*`, `testBiPredicate*`
*   `testFailable* Nop()`: Verifica que los métodos `nop()` retornen implementaciones nulas que no hacen nada y retornan `null` o `0`/`false`.

#### 4.5 Pruebas de Definición de Interfaces - `testThrows_*`
Conjunto de más de 40 tests que validan que todas las interfaces `Failable*` estén correctamente definidas para poder lanzar cualquier `Throwable`.

Ejemplo: `testThrows_FailableConsumer_Object_Throwable()`, `testThrows_FailableBiFunction_String_IOException()`, `testThrows_FailableSupplier_Throwable()`, etc. Cubren: `FailableConsumer`, `FailableFunction`, `FailablePredicate`, `FailableSupplier`, `FailableRunnable`, `FailableCallable`, `FailableIntConsumer`, `FailableLongFunction`, `FailableDoubleSupplier`, entre otras.

#### 4.6 Manejo de Excepciones Esperado
Todos los tests siguen el mismo patrón de validación:
1.  Si se lanza `RuntimeException` o `Error` -> se relanza idéntico `assertSame`.
2.  Si se lanza `IOException` -> se envuelve en `UncheckedIOException`.
3.  Si se lanza otra `Exception` comprobada -> se envuelve en `UndeclaredThrowableException`.
4.  Si no hay excepción -> la ejecución continúa normalmente.

### 5. Dependencias

*   `org.junit.jupiter.api.*` - JUnit 5
*   `org.apache.commons.lang3.function.*` - Todas las interfaces `Failable*`
*   `java.util.function.*` - Interfaces estándar de Java
*   `java.io.IOException`, `java.io.UncheckedIOException`

### 6. Conclusión

`FailableTest` no es una clase de lógica de negocio, sino una **clase de aseguramiento de calidad**. Su propósito es garantizar con una cobertura del 100% que la utilidad `Failable` permita usar programación funcional con excepciones comprobadas de forma segura, sin perder información de la excepción original y manteniendo compatibilidad con el API de Streams y Lambdas de Java 8+.

> Si requieres la documentación de la clase productiva `Failable.java` y no de su test, indícalo y se generará su documentación con sus métodos `rethrow()`, `get()`, `accept()`, `apply()`, `asFunction()`, etc.