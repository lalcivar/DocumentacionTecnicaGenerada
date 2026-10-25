# Documentación Técnica - `org.apache.commons.lang3.function.FailableTest`

> **Paquete:** `org.apache.commons.lang3.function`  
> **Clase:** `FailableTest extends AbstractLangTest`  
> **Tipo:** Clase de pruebas unitarias JUnit 5 (Jupiter)  
> **Dependencias:** `org.apache.commons.lang3.function.Failable`, `Failable*` interfaces, `org.junit.jupiter.api`, `java.io`, `java.util.function`

### 1. Resumen Ejecutivo

`FailableTest` es la suite de pruebas unitarias para el conjunto de interfaces funcionales `Failable*` de Apache Commons Lang3. Su propósito es validar el comportamiento de los wrappers de la clase utilitaria `Failable` que permiten propagar excepciones comprobadas (`checked exceptions`) dentro del API funcional de Java 8+, verificando la conversión correcta a `UncheckedIOException`, `UndeclaredThrowableException` y la preservación de `RuntimeException`/`Error`. La clase contiene 5 clases internas de soporte (`CloseableObject`, `FailureOnInvocationCount`, `FailureOnOddInvocations`, `SomeException`, `Testable`) y más de 150 métodos de prueba que cubren todos los operadores, consumidores, funciones, predicados y suppliers failable.

### 2. Especificación de Métodos y Constructores

#### 2.1 Clase Principal `FailableTest`

| Método | Visibilidad | Estático | @param (nombre: tipo, requerido) | @return | @throws | Descripción |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `beforeEach()` | package-private | No | - | `void` | - | Inicializa el estado antes de cada test. Invoca `FailureOnOddInvocations.reset()`. Anotado con `@BeforeEach`. |
| `testAcceptBiConsumer()` | package-private | No | - | `void` | - | Verifica `Failable.accept(FailableBiConsumer, ...)` con `IllegalStateException`, `OutOfMemoryError` e `IOException` -> `UncheckedIOException`. |
| `testAcceptConsumer()` | package-private | No | - | `void` | - | Verifica `Failable.accept(FailableConsumer, ...)` con los 3 tipos de `Throwable`. |
| `testAcceptDoubleConsumer()` | package-private | No | - | `void` | - | Verifica `Failable.accept(FailableDoubleConsumer, double)` y estado de `Testable.getAcceptedPrimitiveObject1()`. |
| `testAcceptIntConsumer()` | package-private | No | - | `void` | - | Verifica `Failable.accept(FailableIntConsumer, int)`. |
| `testAcceptLongConsumer()` | package-private | No | - | `void` | - | Verifica `Failable.accept(FailableLongConsumer, long)`. |
| `testAcceptObjDoubleConsumer()` | package-private | No | - | `void` | - | Verifica `Failable.accept(FailableObjDoubleConsumer, T, double)`. |
| `testAcceptObjIntConsumer()` | package-private | No | - | `void` | - | Verifica `Failable.accept(FailableObjIntConsumer, T, int)`. |
| `testAcceptObjLongConsumer()` | package-private | No | - | `void` | - | Verifica `Failable.accept(FailableObjLongConsumer, T, long)`. |
| `testApplyBiFunction()` | package-private | No | - | `void` | - | Verifica `Failable.apply(FailableBiFunction, ...)` con manejo de excepciones. |
| `testApplyDoubleBinaryOperator()` | package-private | No | - | `void` | - | Verifica `Failable.applyAsDouble(FailableDoubleBinaryOperator, double, double)`. |
| `testApplyFunction()` | package-private | No | - | `void` | - | Verifica `Failable.apply(FailableFunction, ...)`. |
| `testApplyNonNull()` | package-private | No | - | `void` | `SomeException` | Verifica `Failable.applyNonNull(T, FailableFunction)` con y sin excepciones. |
| `testApplyNonNull2()` | package-private | No | - | `void` | `SomeException, IOException` | Verifica sobrecarga de 2 funciones `Failable.applyNonNull(T, F1, F2)`. |
| `testApplyNonNull3()` | package-private | No | - | `void` | `SomeException, IOException, ClassNotFoundException` | Verifica sobrecarga de 3 funciones `Failable.applyNonNull(T, F1, F2, F3)`. |
| `testAsCallable()` | package-private | No | - | `void` | - | Verifica `Failable.asCallable(FailableCallable)` -> `Callable` y `UndeclaredThrowableException`. |
| `testAsConsumer()` | package-private | No | - | `void` | - | Verifica `Failable.asConsumer(FailableConsumer)` -> `Consumer`. |
| `testAsRunnable()` | package-private | No | - | `void` | - | Verifica `Failable.asRunnable(FailableRunnable)` -> `Runnable`. |
| `testAsSupplier()` | package-private | No | - | `void` | - | Verifica `Failable.asSupplier(FailableSupplier)` -> `Supplier`. |
| `testBiConsumer()` | package-private | No | - | `void` | `Throwable` | Verifica `Failable.asBiConsumer(FailableBiConsumer)` -> `BiConsumer`. |
| `testBiConsumerAndThen()` | package-private | No | - | `void` | `Throwable` | Verifica `FailableBiConsumer.andThen()` y `nop()`, caso `null` -> `NullPointerException`. |
| `testBiFunction()` | package-private | No | - | `void` | - | Verifica `Failable.asBiFunction(FailableBiFunction)` -> `BiFunction`. |
| `testBiFunctionAndThen()` | package-private | No | - | `void` | `IOException` | Verifica `FailableBiFunction.andThen(FailableFunction)`. |
| `testBiPredicate()` | package-private | No | - | `void` | - | Verifica `Failable.asBiPredicate(FailableBiPredicate)` -> `BiPredicate`. |
| `testBiPredicateAnd()` | package-private | No | - | `void` | `Throwable` | Verifica `FailableBiPredicate.and()`. |
| `testBiPredicateNegate()` | package-private | No | - | `void` | `Throwable` | Verifica `FailableBiPredicate.negate()`. |
| `testBiPredicateOr()` | package-private | No | - | `void` | `Throwable` | Verifica `FailableBiPredicate.or()`. |
| `testByteConsumerAndThen()` | package-private | No | - | `void` | `Throwable` | Verifica `FailableByteConsumer.andThen()`. |
| `testCallable()` | package-private | No | - | `void` | - | Verifica `Failable.call(FailableCallable)` y `Failable.run(FailableCallable)`. |
| `testConsumerAndThen()` | package-private | No | - | `void` | `Throwable` | Verifica `FailableConsumer.andThen()`. |
| `testDoubleConsumerAndThen()` | package-private | No | - | `void` | `Throwable` | Verifica `FailableDoubleConsumer.andThen()`. |
| `testDoublePredicate()` | package-private | No | - | `void` | `Throwable` | Verifica `FailableDoublePredicate.test(double)`. |
| `testDoublePredicateAnd()` | package-private | No | - | `void` | `Throwable` | Verifica `FailableDoublePredicate.and()`. |
| `testDoublePredicateNegate()` | package-private | No | - | `void` | `Throwable` | Verifica `FailableDoublePredicate.negate()`. |
| `testDoublePredicateOr()` | package-private | No | - | `void` | `Throwable` | Verifica `FailableDoublePredicate.or()`. |
| `testDoubleUnaryOperatorAndThen()` | package-private | No | - | `void` | `Throwable` | Verifica `FailableDoubleUnaryOperator.andThen()`. |
| `testDoubleUnaryOperatorCompose()` | package-private | No | - | `void` | `Throwable` | Verifica `FailableDoubleUnaryOperator.compose()`. |
| `testDoubleUnaryOperatorIdentity()` | package-private | No | - | `void` | `Throwable` | Verifica `FailableDoubleUnaryOperator.identity()`. |
| `testFailableBiFunctionNop()` | package-private | No | - | `void` | `Throwable` | Verifica `FailableBiFunction.nop()` retorna `null`. |
| `testFailableConsumerNop()` | package-private | No | - | `void` | `Throwable` | Verifica `FailableConsumer.nop()` no lanza excepción. |
| `testFailableDoubleFunctionNop()` | package-private | No | - | `void` | `Throwable` | Verifica `FailableDoubleFunction.nop()` retorna `null`. |
| `testFailableDoubleToIntFunctionNop()` | package-private | No | - | `void` | `Throwable` | Verifica `FailableDoubleToIntFunction.nop()` retorna `0`. |
| `testFailableDoubleToLongFunctionNop()` | package-private | No | - | `void` | `Throwable` | Verifica `FailableDoubleToLongFunction.nop()` retorna `0`. |
| `testFailableIntFunctionNop()` | package-private | No | - | `void` | `Throwable` | Verifica `FailableIntFunction.nop()` retorna `null`. |
| `testFailableIntToDoubleFunctionNop()` | package-private | No | - | `void` | `Throwable` | Verifica `FailableIntToDoubleFunction.nop()` retorna `0`. |
| `testFailableIntToFloatFunctionNop()` | package-private | No | - | `void` | `Throwable` | Verifica `FailableIntToFloatFunction.nop()` retorna `0`. |
| `testFailableIntToLongFunctionNop()` | package-private | No | - | `void` | `Throwable` | Verifica `FailableIntToLongFunction.nop()` retorna `0`. |
| `testFailableLongFunctionNop()` | package-private | No | - | `void` | `Throwable` | Verifica `FailableLongFunction.nop()` retorna `null`. |
| `testFailableLongToDoubleFunctionNop()` | package-private | No | - | `void` | `Throwable` | Verifica `FailableLongToDoubleFunction.nop()` retorna `0`. |
| `testFailableLongToIntFunctionNop()` | package-private | No | - | `void` | `Throwable` | Verifica `FailableLongToIntFunction.nop()` retorna `0`. |
| `testFailableObjDoubleConsumerNop()` | package-private | No | - | `void` | `Throwable` | Verifica `FailableObjDoubleConsumer.nop()`. |
| `testFailableObjIntConsumerNop()` | package-private | No | - | `void` | `Throwable` | Verifica `FailableObjIntConsumer.nop()`. |
| `testFailableObjLongConsumerNop()` | package-private | No | - | `void` | `Throwable` | Verifica `FailableObjLongConsumer.nop()`. |
| `testFailableToBooleanFunctionNop()` | package-private | No | - | `void` | `Throwable` | Verifica `FailableToBooleanFunction.nop()` retorna `false`. |
| `testFailableToDoubleBiFunctionNop()` | package-private | No | - | `void` | `Throwable` | Verifica `FailableToDoubleBiFunction.nop()` retorna `0`. |
| `testFailableToDoubleFunctionNop()` | package-private | No | - | `void` | `Throwable` | Verifica `FailableToDoubleFunction.nop()` retorna `0`. |
| `testFailableToIntBiFunctionNop()` | package-private | No | - | `void` | `Throwable` | Verifica `FailableToIntBiFunction.nop()` retorna `0`. |
| `testFailableToIntFunctionNop()` | package-private | No | - | `void` | `Throwable` | Verifica `FailableToIntFunction.nop()` retorna `0`. |
| `testFailableToLongBiFunctionNop()` | package-private | No | - | `void` | `Throwable` | Verifica `FailableToLongBiFunction.nop()` retorna `0`. |
| `testFailableToLongFunctionNop()` | package-private | No | - | `void` | `Throwable` | Verifica `FailableToLongFunction.nop()` retorna `0`. |
| `testFunction()` | package-private | No | - | `void` | - | Verifica `Failable.asFunction(FailableFunction)` -> `Function`. |
| `testFunctionAndThen()` | package-private | No | - | `void` | `IOException` | Verifica `FailableFunction.andThen()`. |
| `testFunctionCompose()` | package-private | No | - | `void` | `Throwable` | Verifica `FailableFunction.compose()`. |
| `testFunctionFunction()` | package-private | No | - | `void` | `Exception` | Verifica `FailableFunction.function()` con `throwingFunction()`. |
| `testFunctionIdentity()` | package-private | No | - | `void` | `Throwable` | Verifica `FailableFunction.identity()`. |
| `testGetAsBooleanSupplier()` | package-private | No | - | `void` | - | Verifica `Failable.getAsBoolean(FailableBooleanSupplier)`. |
| `testGetAsDoubleSupplier()` | package-private | No | - | `void` | - | Verifica `Failable.getAsDouble(FailableDoubleSupplier)`. |
| `testGetAsIntSupplier()` | package-private | No | - | `void` | - | Verifica `Failable.getAsInt(FailableIntSupplier)`. |
| `testGetAsLongSupplier()` | package-private | No | - | `void` | - | Verifica `Failable.getAsLong(FailableLongSupplier)`. |
| `testGetAsShortSupplier()` | package-private | No | - | `void` | - | Verifica `Failable.getAsShort(FailableShortSupplier)`. |
| `testGetFromSupplier()` | package-private | No | - | `void` | - | Verifica `Failable.call/run` con `FailableSupplier`. |
| `testGetSupplier()` | package-private | No | - | `void` | - | Verifica `Failable.get(FailableSupplier)`. |
| `testIntConsumerAndThen()` | package-private | No | - | `void` | `Throwable` | Verifica `FailableIntConsumer.andThen()`. |
| `testIntPredicate()` | package-private | No | - | `void` | `Throwable` | Verifica `FailableIntPredicate.test(int)`. |
| `testIntPredicateAnd()` | package-private | No | - | `void` | `Throwable` | Verifica `FailableIntPredicate.and()`. |
| `testIntPredicateNegate()` | package-private | No | - | `void` | `Throwable` | Verifica `FailableIntPredicate.negate()`. |
| `testIntPredicateOr()` | package-private | No | - | `void` | `Throwable` | Verifica `FailableIntPredicate.or()`. |
| `testIntUnaryOperatorAndThen()` | package-private | No | - | `void` | `Throwable` | Verifica `FailableIntUnaryOperator.andThen()`. |
| `testIntUnaryOperatorCompose()` | package-private | No | - | `void` | `Throwable` | Verifica `FailableIntUnaryOperator.compose()`. |
| `testIntUnaryOperatorIdentity()` | package-private | No | - | `void` | `Throwable` | Verifica `FailableIntUnaryOperator.identity()`. |
| `testLongConsumerAndThen()` | package-private | No | - | `void` | `Throwable` | Verifica `FailableLongConsumer.andThen()`. |
| `testLongPredicate()` | package-private | No | - | `void` | `Throwable` | Verifica `FailableLongPredicate.test(long)`. |
| `testLongPredicateAnd()` | package-private | No | - | `void` | `Throwable` | Verifica `FailableLongPredicate.and()`. |
| `testLongPredicateNegate()` | package-private | No | - | `void` | `Throwable` | Verifica `FailableLongPredicate.negate()`. |
| `testLongPredicateOr()` | package-private | No | - | `void` | `Throwable` | Verifica `FailableLongPredicate.or()`. |
| `testLongUnaryOperatorAndThen()` | package-private | No | - | `void` | `Throwable` | Verifica `FailableLongUnaryOperator.andThen()`. |
| `testLongUnaryOperatorCompose()` | package-private | No | - | `void` | `Throwable` | Verifica `FailableLongUnaryOperator.compose()`. |
| `testLongUnaryOperatorIdentity()` | package-private | No | - | `void` | `Throwable` | Verifica `FailableLongUnaryOperator.identity()`. |
| `testPredicate()` | package-private | No | - | `void` | - | Verifica `Failable.asPredicate(FailablePredicate)` -> `Predicate`. |
| `testPredicateAnd()` | package-private | No | - | `void` | `Throwable` | Verifica `FailablePredicate.and()`. |
| `testPredicateNegate()` | package-private | No | - | `void` | `Throwable` | Verifica `FailablePredicate.negate()`. |
| `testPredicateOr()` | package-private | No | - | `void` | `Throwable` | Verifica `FailablePredicate.or()`. |
| `testRunnable()` | package-private | No | - | `void` | - | Verifica `Failable.run(FailableRunnable)` y `null` handling. |
| `testTryWithResources()` | package-private | No | - | `void` | - | Verifica `Failable.tryWithResources(FailableRunnable, FailableRunnable)`. |
| `throwingFunction(String input)` | private | No | `input: String, Sí` | `String` | `Exception` | Función auxiliar que retorna `input` sin transformación; usada para testear `FailableFunction.function()`. |

#### 2.2 Clase Interna `CloseableObject` (`public static`)

| Método | @param | @return | @throws | Descripción |
| :--- | :--- | :--- | :--- | :--- |
| `CloseableObject()` | - | - | - | Constructor por defecto implícito. |
| `close()` | - | `void` | - | Marca `closed = true`. Simula `AutoCloseable`. |
| `isClosed()` | - | `boolean` | - | Retorna estado `closed`. |
| `reset()` | - | `void` | - | Resetea `closed = false`. |
| `run(Throwable throwable)` | `throwable: Throwable, No` | `void` | `Throwable` | Lanza `throwable` si no es `null`. Usado para probar `tryWithResources`. |

#### 2.3 Clase Interna `FailureOnInvocationCount` (`public static`)

| Método | @param | @return | @throws | Descripción |
| :--- | :--- | :--- | :--- | :--- |
| `FailureOnInvocationCount(int throwOn)` | `throwOn: int, Sí` | - | `SomeException` | Constructor. Inicializa contador `throwOn` donde debe fallar. |
| `inc(int value)` | `value: int, Sí` | `int` | `SomeException` | Incrementa `value+1` o lanza `SomeException` si `++current == throwOn`. |
| `incCnfe(int value)` | `value: int, Sí` | `int` | `ClassNotFoundException` | Variante que lanza `ClassNotFoundException` envolviendo `SomeException`. |
| `incIo(int value)` | `value: int, Sí` | `int` | `IOException` | Variante que lanza `IOException` envolviendo `SomeException`. |
| `reset()` | - | `void` | - | Resetea `current = 0`. |
| `throwCnfeOn()` | - | `void` | `ClassNotFoundException` | **private**. Lanza `ClassNotFoundException` si se alcanza `throwOn`. |
| `throwIoOn()` | - | `void` | `IOException` | **private**. Lanza `IOException` si se alcanza `throwOn`. |
| `throwSeOn()` | - | `void` | `SomeException` | **private**. Lanza `SomeException` si se alcanza `throwOn`. |

#### 2.4 Clase Interna `FailureOnOddInvocations` (`public static`)

| Método | @param | @return | @throws | Descripción |
| :--- | :--- | :--- | :--- | :--- |
| `FailureOnOddInvocations()` | - | - | `SomeException` | Constructor. Invoca `throwOnOdd()` y falla en invocaciones impares. |
| `reset()` | - | `void` | - | **static**. Resetea `invocations = 0`. |
| `testDouble(double value)` | `value: double, Sí` | `boolean` | `SomeException` | **static**. Retorna `true` o lanza en invocación impar. |
| `testGetBool()` | - | `boolean` | `SomeException` | **static**. Retorna `true` o lanza en invocación impar. |
| `testInc(int value)` | `value: int, Sí` | `int` | `SomeException` | **static**. Retorna `value+1` o lanza en invocación impar. |
| `testInt(int value)` | `value: int, Sí` | `boolean` | `SomeException` | **static**. Retorna `true` o lanza en invocación impar. |
| `testLong(long value)` | `value: long, Sí` | `boolean` | `SomeException` | **static**. Retorna `true` o lanza en invocación impar. |
| `throwOnOdd()` | - | `void` | `SomeException` | **private static**. Incrementa `invocations` y lanza `SomeException` si es impar. |
| `getAsBoolean()` | - | `boolean` | `SomeException` | Retorna `true` o lanza en invocación impar (instancia). |

#### 2.5 Clase Interna `SomeException extends Exception` (`public static`)

| Método | @param | @return | @throws | Descripción |
| :--- | :--- | :--- | :--- | :--- |
| `SomeException(String message)` | `message: String, Sí` | - | - | Constructor delega a `SomeException(message, 0)`. |
| `SomeException(String message, int value)` | `message: String, Sí` / `value: int, Sí` | - | - | Constructor principal. Almacena `value`. |
| `setThrowable(Throwable throwable)` | `throwable: Throwable, Sí` | `void` | - | Asigna `t` interno. |
| `test()` | - | `void` | `Throwable` | Lanza `t` si no es `null`. |

#### 2.6 Clase Interna `Testable<T, P>` (`public static`)

| Método | @param | @return | @throws | Descripción |
| :--- | :--- | :--- | :--- | :--- |
| `Testable(Throwable throwable)` | `throwable: Throwable, No` | - | - | Constructor. Inicializa `throwable` interno. |
| `getAcceptedObject()` | - | `T` | - | Retorna `acceptedObject`. |
| `getAcceptedPrimitiveObject1()` | - | `P` | - | Retorna primer primitivo boxeado aceptado. |
| `getAcceptedPrimitiveObject2()` | - | `P` | - | Retorna segundo primitivo boxeado aceptado. |
| `setThrowable(Throwable throwable)` | `throwable: Throwable, No` | `void` | - | Setter para `throwable`. |
| `test()` | - | `void` | `Throwable` | Delega a `test(throwable)`. |
| `test(Object input1, Object input2)` | `input1: Object, Sí` / `input2: Object, Sí` | `Object` | `Throwable` | Lanza `throwable` o retorna `acceptedObject`. |
| `test(Throwable throwable)` | `throwable: Throwable, No` | `void` | `Throwable` | Lanza `throwable` si no es `null`. |
| `testAsBooleanPrimitive()` | - | `boolean` | `Throwable` | Delega a `testAsBooleanPrimitive(throwable)`. |
| `testAsBooleanPrimitive(Throwable throwable)` | `throwable: Throwable, No` | `boolean` | `Throwable` | Lanza o retorna `false`. |
| `testAsDoublePrimitive()` | - | `double` | `Throwable` | Delega a `testAsDoublePrimitive(throwable)`. |
| `testAsDoublePrimitive(Throwable throwable)` | `throwable: Throwable, No` | `double` | `Throwable` | Lanza o retorna `0`. |
| `testAsInteger()` | - | `Integer` | `Throwable` | Delega a `testAsInteger(throwable)`. |
| `testAsInteger(Throwable throwable)` | `throwable: Throwable, No` | `Integer` | `Throwable` | Lanza o retorna `0`. |
| `testAsIntPrimitive()` | - | `int` | `Throwable` | Delega a `testAsIntPrimitive(throwable)`. |
| `testAsIntPrimitive(Throwable throwable)` | `throwable: Throwable, No` | `int` | `Throwable` | Lanza o retorna `0`. |
| `testAsLongPrimitive()` | - | `long` | `Throwable` | Delega a `testAsLongPrimitive(throwable)`. |
| `testAsLongPrimitive(Throwable throwable)` | `throwable: Throwable, No` | `long` | `Throwable` | Lanza o retorna `0`. |
| `testAsShortPrimitive()` | - | `short` | `Throwable` | Delega a `testAsShortPrimitive(throwable)`. |
| `testAsShortPrimitive(Throwable throwable)` | `throwable: Throwable, No` | `short` | `Throwable` | Lanza o retorna `0`. |
| `testDouble(double i)` | `i: double, Sí` | `void` | `Throwable` | Almacena `i` en `acceptedPrimitiveObject1` o lanza. |
| `testDoubleDouble(double i, double j)` | `i: double, Sí` / `j: double, Sí` | `double` | `Throwable` | Almacena `i,j` y retorna `3d` o lanza. |
| `testInt(int i)` | `i: int, Sí` | `void` | `Throwable` | Almacena `i` o lanza. |
| `testLong(long i)` | `i: long, Sí` | `void` | `Throwable` | Almacena `i` o lanza. |
| `testObjDouble(T object, double i)` | `object: T, Sí` / `i: double, Sí` | `void` | `Throwable` | Almacena `object` e `i` o lanza. |
| `testObjInt(T object, int i)` | `object: T, Sí` / `i: int, Sí` | `void` | `Throwable` | Almacena `object` e `i` o lanza. |
| `testObjLong(T object, long i)` | `object: T, Sí` / `i: long, Sí` | `void` | `Throwable` | Almacena `object` e `i` o lanza. |

#### 2.7 Métodos Anónimos Internos (`testThrows_*`)

Cada método `testThrows_*` instancia una clase anónima que implementa una interfaz `Failable*` y sobrescribe su método abstracto único para validar que la firma genérica permite lanzar cualquier `Throwable`. Todos siguen el patrón:

```java
/**
 * @param param tipo genérico (Object/String, primitivo)
 * @throws IOException si se fuerza el fallo de prueba
 * @return según interfaz (void, boolean, int, long, double, Object)
 */
@Override
public <tipoRetorno> <nombreMetodo>(<params>) throws E { throw new IOException("test"); }
```

Listado exhaustivo de métodos anónimos documentados (todos con `@Override`):

| Interfaz Anónima | Método Anónimo `nombreMetodo()` | Firma | @throws |
| :--- | :--- | :--- | :--- |
| `FailableBiConsumer<Object,Object,Throwable>` | `accept()` | `accept(Object object1, Object object2)` | `Throwable` |
| `FailableBiConsumer<String,String,IOException>` | `accept()` | `accept(String object1, String object2)` | `IOException` |
| `FailableBiFunction<Object,Object,Object,Throwable>` | `apply()` | `apply(Object input1, Object input2)` | `Throwable` |
| `FailableBiFunction<String,String,String,IOException>` | `apply()` | `apply(String input1, String input2)` | `IOException` |
| `FailableBiPredicate<Object,Object,Throwable>` | `test()` | `test(Object object1, Object object2)` | `Throwable` |
| `FailableBiPredicate<String,String,IOException>` | `test()` | `test(String object1, String object2)` | `IOException` |
| `FailableBooleanSupplier<IOException>` | `getAsBoolean()` | `getAsBoolean()` | `IOException` |
| `FailableBooleanSupplier<Throwable>` | `getAsBoolean()` | `getAsBoolean()` | `Throwable` |
| `FailableByteConsumer<IOException>` | `accept()` | `accept(byte value)` | `IOException` |
| `FailableByteConsumer<Throwable>` | `accept()` | `accept(byte value)` | `Throwable` |
| `FailableByteSupplier<IOException>` | `getAsByte()` | `getAsByte()` | `IOException` |
| `FailableByteSupplier<Throwable>` | `getAsByte()` | `getAsByte()` | `Throwable` |
| `FailableCallable<Object,Throwable>` | `call()` | `call()` | `Throwable` |
| `FailableCallable<String,IOException>` | `call()` | `call()` | `IOException` |
| `FailableConsumer<Object,Throwable>` | `accept()` | `accept(Object object)` | `Throwable` |
| `FailableConsumer<String,IOException>` | `accept()` | `accept(String object)` | `IOException` |
| `FailableDoubleBinaryOperator<IOException>` | `applyAsDouble()` | `applyAsDouble(double left, double right)` | `IOException` |
| `FailableDoubleBinaryOperator<Throwable>` | `applyAsDouble()` | `applyAsDouble(double left, double right)` | `Throwable` |
| `FailableDoubleConsumer<IOException>` | `accept()` | `accept(double value)` | `IOException` |
| `FailableDoubleConsumer<Throwable>` | `accept()` | `accept(double value)` | `Throwable` |
| `FailableDoubleFunction<String,IOException>` | `apply()` | `apply(double input)` | `IOException` |
| `FailableDoubleFunction<Object,Throwable>` | `apply()` | `apply(double input)` | `Throwable` |
| `FailableDoubleSupplier<IOException>` | `getAsDouble()` | `getAsDouble()` | `IOException` |
| `FailableDoubleSupplier<Throwable>` | `getAsDouble()` | `getAsDouble()` | `Throwable` |
| `FailableDoubleToIntFunction<IOException>` | `applyAsInt()` | `applyAsInt(double value)` | `IOException` |
| `FailableDoubleToIntFunction<Throwable>` | `applyAsInt()` | `applyAsInt(double value)` | `Throwable` |
| `FailableDoubleToLongFunction<IOException>` | `applyAsLong()` | `applyAsLong(double value)` | `IOException` |
| `FailableDoubleToLongFunction<Throwable>` | `applyAsLong()` | `applyAsLong(double value)` | `Throwable` |
| `FailableFunction<Object,Object,Throwable>` | `apply()` | `apply(Object input)` | `Throwable` |
| `FailableFunction<String,String,IOException>` | `apply()` | `apply(String input)` | `IOException` |
| `FailableIntBinaryOperator<IOException>` | `applyAsInt()` | `applyAsInt(int left, int right)` | `IOException` |
| `FailableIntBinaryOperator<Throwable>` | `applyAsInt()` | `applyAsInt(int left, int right)` | `Throwable` |
| `FailableIntConsumer<IOException>` | `accept()` | `accept(int value)` | `IOException` |
| `FailableIntConsumer<Throwable>` | `accept()` | `accept(int value)` | `Throwable` |
| `FailableIntFunction<Object,Throwable>` | `apply()` | `apply(int input)` | `Throwable` |
| `FailableIntFunction<String,IOException>` | `apply()` | `apply(int input)` | `IOException` |
| `FailableIntSupplier<IOException>` | `getAsInt()` | `getAsInt()` | `IOException` |
| `FailableIntSupplier<Throwable>` | `getAsInt()` | `getAsInt()` | `Throwable` |
| `FailableIntToDoubleFunction<IOException>` | `applyAsDouble()` | `applyAsDouble(int value)` | `IOException` |
| `FailableIntToDoubleFunction<Throwable>` | `applyAsDouble()` | `applyAsDouble(int value)` | `Throwable` |
| `FailableIntToFloatFunction<IOException>` | `applyAsFloat()` | `applyAsFloat(int value)` | `IOException` |
| `FailableIntToFloatFunction<Throwable>` | `applyAsFloat()` | `applyAsFloat(int value)` | `Throwable` |
| `FailableIntToLongFunction<IOException>` | `applyAsLong()` | `applyAsLong(int value)` | `IOException` |
| `FailableIntToLongFunction<Throwable>` | `applyAsLong()` | `applyAsLong(int value)` | `Throwable` |
| `FailableLongBinaryOperator<IOException>` | `applyAsLong()` | `applyAsLong(long left, long right)` | `IOException` |
| `FailableLongBinaryOperator<Throwable>` | `applyAsLong()` | `applyAsLong(long left, long right)` | `Throwable` |
| `FailableLongConsumer<IOException>` | `accept()` | `accept(long object)` | `IOException` |
| `FailableLongConsumer<Throwable>` | `accept()` | `accept(long object)` | `Throwable` |
| `FailableLongFunction<String,IOException>` | `apply()` | `apply(long input)` | `IOException` |
| `FailableLongFunction<Object,Throwable>` | `apply()` | `apply(long input)` | `Throwable` |
| `FailableLongSupplier<IOException>` | `getAsLong()` | `getAsLong()` | `IOException` |
| `FailableLongSupplier<Throwable>` | `getAsLong()` | `getAsLong()` | `Throwable` |
| `FailableLongToDoubleFunction<IOException>` | `applyAsDouble()` | `applyAsDouble(long value)` | `IOException` |
| `FailableLongToDoubleFunction<Throwable>` | `applyAsDouble()` | `applyAsDouble(long value)` | `Throwable` |
| `FailableLongToIntFunction<IOException>` | `applyAsInt()` | `applyAsInt(long value)` | `IOException` |
| `FailableLongToIntFunction<Throwable>` | `applyAsInt()` | `applyAsInt(long value)` | `Throwable` |
| `FailableObjDoubleConsumer<Object,Throwable>` | `accept()` | `accept(Object object, double value)` | `Throwable` |
| `FailableObjDoubleConsumer<String,IOException>` | `accept()` | `accept(String object, double value)` | `IOException` |
| `FailableObjIntConsumer<Object,Throwable>` | `accept()` | `accept(Object object, int value)` | `Throwable` |
| `FailableObjIntConsumer<String,IOException>` | `accept()` | `accept(String object, int value)` | `IOException` |
| `FailableObjLongConsumer<Object,Throwable>` | `accept()` | `accept(Object object, long value)` | `Throwable` |
| `FailableObjLongConsumer<String,IOException>` | `accept()` | `accept(String object, long value)` | `IOException` |
| `FailablePredicate<Object,Throwable>` | `test()` | `test(Object object)` | `Throwable` |
| `FailablePredicate<String,IOException>` | `test()` | `test(String object)` | `IOException` |
| `FailableRunnable<IOException>` | `run()` | `run()` | `IOException` |
| `FailableRunnable<Throwable>` | `run()` | `run()` | `Throwable` |
| `FailableShortSupplier<IOException>` | `getAsShort()` | `getAsShort()` | `IOException` |
| `FailableShortSupplier<Throwable>` | `getAsShort()` | `getAsShort()` | `Throwable` |
| `FailableSupplier<Object,Throwable>` | `get()` | `get()` | `Throwable` |
| `FailableSupplier<String,IOException>` | `get()` | `get()` | `IOException` |
| `FailableToBooleanFunction<Object,Throwable>` | `applyAsBoolean()` | `applyAsBoolean(Object t)` | `Throwable` |
| `FailableToBooleanFunction<String,IOException>` | `applyAsBoolean()` | `applyAsBoolean(String t)` | `IOException` |
| `FailableToDoubleBiFunction<Object,Object,Throwable>` | `applyAsDouble()` | `applyAsDouble(Object t, Object u)` | `Throwable` |
| `FailableToDoubleBiFunction<String,String,IOException>` | `applyAsDouble()` | `applyAsDouble(String t, String u)` | `IOException` |
| `FailableToDoubleFunction<Object,Throwable>` | `applyAsDouble()` | `applyAsDouble(Object t)` | `Throwable` |
| `FailableToDoubleFunction<String,IOException>` | `applyAsDouble()` | `applyAsDouble(String t)` | `IOException` |
| `FailableToIntBiFunction<Object,Object,Throwable>` | `applyAsInt()` | `applyAsInt(Object t, Object u)` | `Throwable` |
| `FailableToIntBiFunction<String,String,IOException>` | `applyAsInt()` | `applyAsInt(String t, String u)` | `IOException` |
| `FailableToIntFunction<Object,Throwable>` | `applyAsInt()` | `applyAsInt(Object t)` | `Throwable` |
| `FailableToIntFunction<String,IOException>` | `applyAsInt()` | `applyAsInt(String t)` | `IOException` |
| `FailableToLongBiFunction<Object,Object,Throwable>` | `applyAsLong()` | `applyAsLong(Object t, Object u)` | `Throwable` |
| `FailableToLongBiFunction<String,String,IOException>` | `applyAsLong()` | `applyAsLong(String t, String u)` | `IOException` |
| `FailableToLongFunction<Object,Throwable>` | `applyAsLong()` | `applyAsLong(Object t)` | `Throwable` |
| `FailableToLongFunction<String,IOException>` | `applyAsLong()` | `applyAsLong(String t)` | `IOException` |

> Todos los métodos anónimos lanzan `new IOException("test")` para verificar que la interfaz failable propaga correctamente la excepción comprobada.

### 3. Ejemplo de Uso

Ejemplo real que combina los patrones validados por `FailableTest`: uso de `Failable` para integrar lambdas con excepciones comprobadas en Streams, `tryWithResources` y conversión a interfaces estándar de `java.util.function`.

```java
package org.apache.commons.lang3.function.example;

import org.apache.commons.lang3.function.Failable;
import org.apache.commons.lang3.function.FailableConsumer;
import org.apache.commons.lang3.function.FailableFunction;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.util.Arrays;
import java.util.List;
import java.util.function.Consumer;
import java.util.function.Function;
import java.util.stream.Collectors;

public class FailableExample {

    // Función failable que puede lanzar IOException
    private static String readFirstLine(Path path) throws IOException {
        return Files.readAllLines(path).get(0);
    }

    public static void main(String[] args) {
        List<Path> paths = Arrays.asList(Path.of("/tmp/a.txt"), Path.of("/tmp/b.txt"));

        // 1. Uso de Failable.apply() - propaga IOException como UncheckedIOException
        FailableFunction<Path, String, IOException> failableFn = FailableExample::readFirstLine;
        // Conversión a Function estándar para uso en Stream
        Function<Path, String> safeFunction = Failable.asFunction(failableFn);
        List<String> lines = paths.stream()
                .map(safeFunction)
                .collect(Collectors.toList());

        // 2. Uso de Failable.accept() con BiConsumer
        FailableConsumer<Path, IOException> writer = p -> Files.writeString(p, "hello");
        Failable.accept(writer, Path.of("/tmp/out.txt")); // lanza UncheckedIOException si falla

        // 3. Uso de Failable.tryWithResources() - equivalente a try-with-resources con excepciones failable
        CloseableResource resource = new CloseableResource();
        Failable.tryWithResources(
            () -> writer.accept(Path.of("/tmp/out.txt")),
            resource::close
        );

        // 4. Uso de applyNonNull - encadenamiento null-safe con excepciones
        String result = Failable.applyNonNull("  hello world  ", String::trim, String::toUpperCase);
        System.out.println(result); // HELLO WORLD

        // 5. Uso de asRunnable / asCallable con manejo de checked exceptions
        Consumer<String> consumer = Failable.asConsumer(s -> {
            if (s == null) throw new IOException("null value");
            System.out.println(s);
        });
        consumer.accept("test"); // IOException se convierte en UncheckedIOException
    }

    static class CloseableResource {
        boolean closed = false;
        void close() { closed = true; }
    }
}
```

**Notas de manejo de excepciones validadas por `FailableTest`:**
- `IOException` -> `UncheckedIOException` (preserva `cause`)
- `RuntimeException` / `IllegalStateException` -> se propaga sin envolver
- `Error` / `OutOfMemoryError` -> se propaga sin envolver
- Otras `checked exceptions` -> `UndeclaredThrowableException`

---
*Documentación generada bajo estándar Javadoc 17. @author Apache Commons Lang3 Team. @since 3.11*
