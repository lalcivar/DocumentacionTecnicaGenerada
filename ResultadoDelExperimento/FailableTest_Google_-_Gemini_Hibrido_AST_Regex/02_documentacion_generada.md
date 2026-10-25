Claro, aquí tienes la documentación técnica exhaustiva para el código Java proporcionado, siguiendo la estructura y restricciones solicitadas.

---

### 1. Resumen ejecutivo

La clase `FailableTest` es una suite de pruebas de JUnit diseñada para verificar exhaustivamente el comportamiento de las interfaces funcionales "fallaibles" (`Failable`) del paquete `org.apache.commons.lang3.function`. Su propósito principal es asegurar que estas interfaces y sus métodos de utilidad manejen correctamente las excepciones con chequeo (`checked exceptions`), convirtiéndolas en excepciones de tiempo de ejecución (`runtime exceptions`) o propagándolas según corresponda, garantizando así la interoperabilidad con las interfaces funcionales estándar de Java. Las pruebas cubren una amplia gama de escenarios, incluyendo la ejecución exitosa, el lanzamiento de errores, y la gestión de diferentes tipos de excepciones.

### 2. Especificación de métodos y constructores

A continuación se detalla la especificación de cada clase, constructor y método encontrado en el código fuente.

---

#### **Clase `FailableTest`**

Clase principal que contiene los casos de prueba para las utilidades `Failable`.

##### `beforeEach()`
Prepara el entorno para cada prueba, reiniciando el estado de la clase de utilidad `FailureOnOddInvocations`.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |

##### `testAcceptBiConsumer()`
Verifica el comportamiento del método `Failable.accept()` para un `FailableBiConsumer`. Prueba los casos en que se lanza `IllegalStateException`, `OutOfMemoryError`, `IOException` (envuelta en `UncheckedIOException`) y el caso de ejecución exitosa.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |

##### `testAcceptConsumer()`
Verifica el comportamiento del método `Failable.accept()` para un `FailableConsumer`. Evalúa el manejo de `IllegalStateException`, `OutOfMemoryError`, `IOException` y la ejecución sin errores.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |

##### `testAcceptDoubleConsumer()`
Prueba el método `Failable.accept()` para un `FailableDoubleConsumer`. Comprueba que las excepciones se propagan correctamente y que el valor de entrada se procesa solo en caso de éxito.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |

##### `testAcceptIntConsumer()`
Prueba el método `Failable.accept()` para un `FailableIntConsumer`. Comprueba que las excepciones se propagan correctamente y que el valor de entrada se procesa solo en caso de éxito.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |

##### `testAcceptLongConsumer()`
Prueba el método `Failable.accept()` para un `FailableLongConsumer`. Comprueba que las excepciones se propagan correctamente y que el valor de entrada se procesa solo en caso de éxito.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |

##### `testAcceptObjDoubleConsumer()`
Prueba el método `Failable.accept()` para un `FailableObjDoubleConsumer`. Verifica el manejo de excepciones y asegura que los parámetros se aceptan correctamente en la ejecución exitosa.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |

##### `testAcceptObjIntConsumer()`
Prueba el método `Failable.accept()` para un `FailableObjIntConsumer`. Verifica el manejo de excepciones y asegura que los parámetros se aceptan correctamente en la ejecución exitosa.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |

##### `testAcceptObjLongConsumer()`
Prueba el método `Failable.accept()` para un `FailableObjLongConsumer`. Verifica el manejo de excepciones y asegura que los parámetros se aceptan correctamente en la ejecución exitosa.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |

##### `testApplyBiFunction()`
Verifica el comportamiento del método `Failable.apply()` para una `FailableBiFunction`. Prueba el manejo de excepciones y la correcta devolución de un valor en el caso exitoso.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |

##### `testApplyDoubleBinaryOperator()`
Prueba el método `Failable.applyAsDouble()` para un `FailableDoubleBinaryOperator`. Valida el manejo de excepciones y el resultado de la operación en caso de éxito.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |

##### `testApplyFunction()`
Verifica el comportamiento del método `Failable.apply()` para una `FailableFunction`. Prueba el manejo de excepciones y la correcta devolución de un valor en el caso exitoso.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |

##### `testApplyNonNull()`
Prueba el método `Failable.applyNonNull(T, FailableFunction)`. Verifica que la función se aplica solo a objetos no nulos y maneja correctamente las excepciones con chequeo.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `SomeException` - Puede ser lanzada durante la ejecución de la prueba. |

##### `testApplyNonNull2()`
Prueba el método `Failable.applyNonNull(T, FailableFunction, FailableFunction)`. Verifica la aplicación encadenada de dos funciones a objetos no nulos y el manejo de diferentes tipos de excepciones con chequeo.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `SomeException`, `IOException` - Pueden ser lanzadas durante la ejecución de la prueba. |

##### `testApplyNonNull3()`
Prueba el método `Failable.applyNonNull(T, FailableFunction, FailableFunction, FailableFunction)`. Verifica la aplicación encadenada de tres funciones a objetos no nulos y el manejo de excepciones.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `SomeException`, `IOException`, `ClassNotFoundException` - Pueden ser lanzadas durante la ejecución de la prueba. |

##### `testAsCallable()`
Prueba la conversión de una `FailableCallable` a una `java.util.concurrent.Callable` estándar usando `Failable.asCallable()`. Verifica que las excepciones con chequeo se envuelven en `UndeclaredThrowableException`.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |

##### `testAsConsumer()`
Prueba la conversión de una `FailableConsumer` a una `java.util.function.Consumer` estándar usando `Failable.asConsumer()`. Verifica el correcto encapsulamiento de excepciones.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |

##### `testAsRunnable()`
Prueba la conversión de una `FailableRunnable` a una `java.lang.Runnable` estándar usando `Failable.asRunnable()`. Verifica que las excepciones con chequeo se envuelven en `UndeclaredThrowableException`.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |

##### `testAsSupplier()`
Prueba la conversión de una `FailableSupplier` a una `java.util.function.Supplier` estándar usando `Failable.asSupplier()`. Verifica el encapsulamiento de excepciones.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |

##### `testBiConsumer()`
Prueba la conversión de una `FailableBiConsumer` a una `java.util.function.BiConsumer` estándar. Valida que las excepciones se manejan y propagan correctamente.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `Throwable` - Puede ser lanzada durante la ejecución de la prueba. |

##### `testBiConsumerAndThen()`
Verifica el comportamiento del método `andThen()` de `FailableBiConsumer`. Prueba la composición con un consumidor que falla y uno que no (`nop`), y el caso de borde con un argumento nulo.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `Throwable` - Puede ser lanzada durante la ejecución de la prueba. |

##### `testBiFunction()`
Prueba la conversión de una `FailableBiFunction` a una `java.util.function.BiFunction` estándar. Valida el manejo de excepciones y el retorno del valor correcto.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |

##### `testBiFunctionAndThen()`
Verifica el comportamiento del método `andThen()` de `FailableBiFunction`. Prueba la composición con funciones que fallan y que no, y el caso de borde con un argumento nulo.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `IOException` - Puede ser lanzada durante la ejecución de la prueba. |

##### `testBiPredicate()`
Prueba la conversión de una `FailableBiPredicate` a una `java.util.function.BiPredicate` estándar. Verifica que las excepciones se encapsulan y que el resultado booleano es correcto.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |

##### `testBiPredicateAnd()`
Verifica el comportamiento del método `and()` de `FailableBiPredicate`. Prueba la lógica booleana y el manejo de argumentos nulos.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `Throwable` - Puede ser lanzada durante la ejecución de la prueba. |

##### `testBiPredicateNegate()`
Verifica el comportamiento del método `negate()` de `FailableBiPredicate`.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `Throwable` - Puede ser lanzada durante la ejecución de la prueba. |

##### `testBiPredicateOr()`
Verifica el comportamiento del método `or()` de `FailableBiPredicate`. Prueba la lógica booleana y el manejo de argumentos nulos.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `Throwable` - Puede ser lanzada durante la ejecución de la prueba. |

##### `testByteConsumerAndThen()`
Verifica el método `andThen()` de `FailableByteConsumer`.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `Throwable` - Puede ser lanzada durante la ejecución de la prueba. |

##### `testCallable()`
Prueba los métodos `Failable.run()` y `Failable.call()` con una `FailableCallable`. Verifica el manejo de excepciones y la obtención del resultado.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |

##### `testConsumerAndThen()`
Verifica el método `andThen()` de `FailableConsumer`.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `Throwable` - Puede ser lanzada durante la ejecución de la prueba. |

##### `testDoubleConsumerAndThen()`
Verifica el método `andThen()` de `FailableDoubleConsumer`.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `Throwable` - Puede ser lanzada durante la ejecución de la prueba. |

##### `testDoublePredicate()`
Prueba directamente una `FailableDoublePredicate`.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `Throwable` - Puede ser lanzada durante la ejecución de la prueba. |

##### `testDoublePredicateAnd()`
Verifica el método `and()` de `FailableDoublePredicate`.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `Throwable` - Puede ser lanzada durante la ejecución de la prueba. |

##### `testDoublePredicateNegate()`
Verifica el método `negate()` de `FailableDoublePredicate`.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `Throwable` - Puede ser lanzada durante la ejecución de la prueba. |

##### `testDoublePredicateOr()`
Verifica el método `or()` de `FailableDoublePredicate`.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `Throwable` - Puede ser lanzada durante la ejecución de la prueba. |

##### `testDoubleUnaryOperatorAndThen()`
Verifica el método `andThen()` de `FailableDoubleUnaryOperator`.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `Throwable` - Puede ser lanzada durante la ejecución de la prueba. |

##### `testDoubleUnaryOperatorCompose()`
Verifica el método `compose()` de `FailableDoubleUnaryOperator`.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `Throwable` - Puede ser lanzada durante la ejecución de la prueba. |

##### `testDoubleUnaryOperatorIdentity()`
Verifica el método estático `identity()` de `FailableDoubleUnaryOperator`.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `Throwable` - Puede ser lanzada durante la ejecución de la prueba. |

##### `testFailableBiFunctionNop()`
Prueba que la implementación `nop()` (sin operación) de `FailableBiFunction` retorna `null`.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `Throwable` - Puede ser lanzada durante la ejecución de la prueba. |

... y así sucesivamente para todos los métodos `testFailable...Nop()`. Todos ellos verifican que la implementación "sin operación" de la interfaz correspondiente retorna un valor por defecto (`null`, `0`, `false`) y no lanza excepciones.

##### `testFunction()`
Prueba la conversión de una `FailableFunction` a una `java.util.function.Function` estándar.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |

##### `testFunctionAndThen()`
Verifica el método `andThen()` de `FailableFunction`.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `IOException` - Puede ser lanzada durante la ejecución de la prueba. |

##### `testFunctionCompose()`
Verifica el método `compose()` de `FailableFunction`.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `Throwable` - Puede ser lanzada durante la ejecución de la prueba. |

##### `testFunctionFunction()`
Prueba la conversión de un método de referencia que lanza excepciones a una `FailableFunction` usando `FailableFunction.function()`.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `Exception` - Puede ser lanzada durante la ejecución de la prueba. |

##### `testFunctionIdentity()`
Verifica el método estático `identity()` de `FailableFunction`.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `Throwable` - Puede ser lanzada durante la ejecución de la prueba. |

##### `testGetAsBooleanSupplier()`
Prueba el método `Failable.getAsBoolean()` con un `FailableBooleanSupplier`.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |

##### `testGetAsDoubleSupplier()`
Prueba el método `Failable.getAsDouble()` con un `FailableDoubleSupplier`.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |

##### `testGetAsIntSupplier()`
Prueba el método `Failable.getAsInt()` con un `FailableIntSupplier`.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |

##### `testGetAsLongSupplier()`
Prueba el método `Failable.getAsLong()` con un `FailableLongSupplier`.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |

##### `testGetAsShortSupplier()`
Prueba el método `Failable.getAsShort()` con un `FailableShortSupplier`.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |

##### `testGetFromSupplier()`
Prueba los métodos `Failable.run()` y `Failable.call()` con un `FailableSupplier`.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |

##### `testGetSupplier()`
Prueba el método `Failable.get()` con un `FailableSupplier`.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |

... y así sucesivamente para todos los métodos de prueba restantes (`testIntConsumerAndThen`, `testPredicate`, `testRunnable`, etc.). Cada uno sigue un patrón similar:
1.  Verifica el comportamiento de un método específico de una interfaz `Failable` (p. ej., `andThen`, `negate`, `or`, `compose`).
2.  O verifica un método de utilidad de la clase `Failable` (p. ej., `asPredicate`, `run`, `tryWithResources`).
3.  Cubre casos de éxito, lanzamiento de errores (`Error`), y lanzamiento de excepciones con chequeo (`IOException`, `SomeException`).

##### `testThrows_...()`
Esta serie de métodos (p. ej., `testThrows_FailableBiConsumer_Object_Throwable`) son pruebas de compilación y ejecución que aseguran que las interfaces `Failable` están correctamente definidas para permitir el lanzamiento de cualquier `Throwable`. Lo hacen creando una clase anónima que implementa la interfaz y lanza una `IOException` desde su método principal.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |

##### `testTryWithResources()`
Prueba el método `Failable.tryWithResources()`. Verifica que la acción principal se ejecuta y que el recurso `closeable` se cierra siempre, tanto en casos de éxito como de fallo.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |

##### `throwingFunction(String input)`
Método de ayuda privado que simplemente devuelve su entrada, pero declara que puede lanzar una `Exception`. Se utiliza para probar la adaptación de métodos con excepciones.

| Atributo | Descripción |
| :--- | :--- |
| **@param** | `input` (`String`, requerido): La cadena de entrada. |
| **@return** | `String` - La misma cadena de entrada. |
| **@throws** | `Exception` - Declarado pero no lanzado en la implementación. |

---

#### **Clase `CloseableObject`**

Clase de ayuda estática interna que simula un recurso que puede ser cerrado.

##### `close()`
Marca el objeto como cerrado.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |

##### `isClosed()`
Comprueba si el objeto ha sido cerrado.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `boolean` - `true` si el objeto está cerrado, `false` en caso contrario. |

##### `reset()`
Reinicia el estado del objeto a "no cerrado".

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |

##### `run(Throwable throwable)`
Ejecuta una acción que puede fallar. Lanza el `Throwable` proporcionado si no es nulo.

| Atributo | Descripción |
| :--- | :--- |
| **@param** | `throwable` (`Throwable`, opcional): La excepción a lanzar. Si es `null`, no se lanza nada. |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `Throwable` - El `throwable` pasado como argumento. |

---

#### **Clase `FailureOnInvocationCount`**

Clase de ayuda que falla (lanza una excepción) en un número de invocación específico.

##### `FailureOnInvocationCount(int throwOn)`
Constructor que inicializa la clase con el número de invocación en el que debe fallar.

| Atributo | Descripción |
| :--- | :--- |
| **@param** | `throwOn` (`int`, requerido): El número de invocación (1-based) en el que se lanzará una excepción. |
| **@throws** | `SomeException` - No es lanzado por esta implementación, pero está en la firma. |

##### `inc(int value)`
Incrementa el valor de entrada en 1. Lanza `SomeException` si se alcanza el contador de invocación objetivo.

| Atributo | Descripción |
| :--- | :--- |
| **@param** | `value` (`int`, requerido): El valor a incrementar. |
| **@return** | `int` - El valor incrementado. |
| **@throws** | `SomeException` - Si el contador de invocaciones actual coincide con `throwOn`. |

##### `incCnfe(int value)`
Incrementa el valor de entrada en 1. Lanza `ClassNotFoundException` si se alcanza el contador de invocación objetivo.

| Atributo | Descripción |
| :--- | :--- |
| **@param** | `value` (`int`, requerido): El valor a incrementar. |
| **@return** | `int` - El valor incrementado. |
| **@throws** | `ClassNotFoundException` - Si el contador de invocaciones actual coincide con `throwOn`. |

##### `incIo(int value)`
Incrementa el valor de entrada en 1. Lanza `IOException` si se alcanza el contador de invocación objetivo.

| Atributo | Descripción |
| :--- | :--- |
| **@param** | `value` (`int`, requerido): El valor a incrementar. |
| **@return** | `int` - El valor incrementado. |
| **@throws** | `IOException` - Si el contador de invocaciones actual coincide con `throwOn`. |

##### `reset()`
Reinicia el contador interno de invocaciones a 0.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |

##### `throwCnfeOn()`
Método privado que incrementa el contador y lanza `ClassNotFoundException` si se cumple la condición.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `ClassNotFoundException` - Si el contador de invocaciones actual coincide con `throwOn`. |

##### `throwIoOn()`
Método privado que incrementa el contador y lanza `IOException` si se cumple la condición.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `IOException` - Si el contador de invocaciones actual coincide con `throwOn`. |

##### `throwSeOn()`
Método privado que incrementa el contador y lanza `SomeException` si se cumple la condición.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `SomeException` - Si el contador de invocaciones actual coincide con `throwOn`. |

---

#### **Clase `FailureOnOddInvocations`**

Clase de ayuda cuyos métodos lanzan `SomeException` en invocaciones impares.

##### `reset()`
Método estático que reinicia el contador de invocaciones global a 0.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |

##### `testDouble(double value)`
Método de prueba estático que siempre retorna `true`, pero falla en invocaciones impares.

| Atributo | Descripción |
| :--- | :--- |
| **@param** | `value` (`double`, requerido): Parámetro de entrada (no utilizado). |
| **@return** | `boolean` - Siempre `true`. |
| **@throws** | `SomeException` - Si es una invocación impar. |

... y así sucesivamente para `testGetBool()`, `testInc(int)`, `testInt(int)`, `testLong(long)`. Todos siguen el mismo patrón.

##### `throwOnOdd()`
Método de ayuda estático y privado que incrementa el contador global y lanza `SomeException` si el nuevo valor es impar.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `SomeException` - Si el número de invocación es impar. |

##### `FailureOnOddInvocations()`
Constructor que falla en invocaciones impares.

| Atributo | Descripción |
| :--- | :--- |
| **@throws** | `SomeException` - Si es una invocación impar. |

##### `getAsBoolean()`
Método de instancia que retorna `true`, pero falla en invocaciones impares.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `boolean` - Siempre `true`. |
| **@throws** | `SomeException` - Si es una invocación impar. |

---

#### **Clase `SomeException`**

Excepción personalizada con chequeo para su uso en las pruebas.

##### `SomeException(String message)`
Constructor.

| Atributo | Descripción |
| :--- | :--- |
| **@param** | `message` (`String`, requerido): Mensaje de la excepción. |

##### `SomeException(String message, int value)`
Constructor.

| Atributo | Descripción |
| :--- | :--- |
| **@param** | `message` (`String`, requerido): Mensaje de la excepción. |
| **@param** | `value` (`int`, requerido): Un valor entero asociado a la excepción. |

##### `setThrowable(Throwable throwable)`
Establece un `Throwable` anidado.

| Atributo | Descripción |
| :--- | :--- |
| **@param** | `throwable` (`Throwable`, requerido): El `Throwable` a almacenar. |
| **@return** | `void` - No retorna ningún valor. |

##### `test()`
Lanza el `Throwable` anidado si ha sido establecido.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `void` - No retorna ningún valor. |
| **@throws** | `Throwable` - El `Throwable` previamente establecido. |

---

#### **Clase `Testable<T, P>`**

Clase de arnés de prueba genérica para simular varios tipos de operaciones que pueden fallar.

##### `Testable(Throwable throwable)`
Constructor que configura la excepción que los métodos de esta clase lanzarán.

| Atributo | Descripción |
| :--- | :--- |
| **@param** | `throwable` (`Throwable`, opcional): La excepción a lanzar en las invocaciones de los métodos `test...`. |

##### `getAcceptedObject()`
Obtiene el objeto genérico `T` aceptado por uno de los métodos de prueba.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `T` - El objeto aceptado. |

##### `getAcceptedPrimitiveObject1()`
Obtiene el primer objeto primitivo (envuelto) `P` aceptado por uno de los métodos de prueba.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `P` - El objeto aceptado. |

##### `getAcceptedPrimitiveObject2()`
Obtiene el segundo objeto primitivo (envuelto) `P` aceptado por uno de los métodos de prueba.

| Atributo | Descripción |
| :--- | :--- |
| **@return** | `P` - El objeto aceptado. |

##### `setThrowable(Throwable throwable)`
Actualiza el `Throwable` que será lanzado por los métodos de prueba.

| Atributo | Descripción |
| :--- | :--- |
| **@param** | `throwable` (`Throwable`, opcional): La nueva excepción a lanzar. |
| **@return** | `void` - No retorna ningún valor. |

... y así sucesivamente para todos los métodos `test...` y `testAs...`. Todos siguen el mismo patrón:
1.  Invocan a `test(throwable)` para potencialmente lanzar una excepción.
2.  Si no se lanza excepción, realizan una acción simple como aceptar un parámetro o devolver un valor por defecto (p. ej., `0`, `false`, `null`).

---

### 3. Ejemplo de uso

El siguiente ejemplo demuestra cómo utilizar la clase de utilidad `Failable` para adaptar un método que lanza una excepción con chequeo (`checked exception`) a una interfaz funcional estándar de Java, como `Consumer` o `Function`. Este es el patrón de uso principal que la clase `FailableTest` está diseñada para verificar.

```java
import org.apache.commons.lang3.function.Failable;

import java.io.IOException;
import java.io.UncheckedIOException;
import java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;

public class FailableUsageExample {

    /**
     * Un método que puede lanzar una excepción con chequeo (checked exception).
     * @param data La cadena a procesar.
     * @return La longitud de la cadena.
     * @throws IOException Si la cadena es "error".
     */
    public static int processString(String data) throws IOException {
        if ("error".equalsIgnoreCase(data)) {
            throw new IOException("Error de procesamiento simulado para la entrada: " + data);
        }
        return data.length();
    }

    public static void main(String[] args) {
        List<String> dataList = Arrays.asList("java", "failable", "error", "test");

        // --- Ejemplo 1: Usando Failable.asFunction con Streams ---
        // Se adapta processString a una Function. La IOException se envolverá
        // en una UncheckedIOException, permitiendo su uso dentro de un stream.
        System.out.println("Procesando longitudes de cadenas (puede fallar):");
        try {
            List<Integer> lengths = dataList.stream()
                .map(Failable.asFunction(FailableUsageExample::processString))
                .collect(Collectors.toList());
            System.out.println("Longitudes: " + lengths);
        } catch (UncheckedIOException e) {
            System.err.println("Se capturó una excepción durante el mapeo del stream: " + e.getCause().getMessage());
        }

        System.out.println("\n-----------------------------------\n");

        // --- Ejemplo 2: Usando Failable.accept con forEach ---
        // Se adapta processString a un Consumer. La excepción también se envuelve.
        System.out.println("Procesando cada elemento (puede fallar):");
        try {
            dataList.forEach(Failable.asConsumer(item -> {
                int length = processString(item);
                System.out.println("'" + item + "' procesado correctamente, longitud: " + length);
            }));
        } catch (UncheckedIOException e) {
            System.err.println("Se capturó una excepción durante la iteración: " + e.getCause().getMessage());
        }
    }
}
```