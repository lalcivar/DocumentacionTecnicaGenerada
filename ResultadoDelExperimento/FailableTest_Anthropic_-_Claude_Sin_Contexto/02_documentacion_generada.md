# Documentación técnica — `FailableTest`

**Paquete:** `org.apache.commons.lang3.function`
**Jerarquía:** `AbstractLangTest` → `FailableTest`
**Visibilidad:** package-private (`class FailableTest`)
**Licencia:** Apache License 2.0 (Apache Commons Lang 3)

---

## 1. Descripción general

`FailableTest` es la clase de pruebas unitarias (JUnit 5 / Jupiter) encargada de validar el contrato de las **interfaces funcionales "failable"** definidas en el paquete `org.apache.commons.lang3.function`, así como los métodos estáticos de la clase utilitaria **`Failable`**.

Las interfaces *failable* (p. ej. `FailableConsumer`, `FailableFunction`, `FailableBiPredicate`, `FailableIntUnaryOperator`, etc.) son equivalentes a las interfaces funcionales estándar de `java.util.function`, con la diferencia de que **sus métodos abstractos declaran `throws T extends Throwable`**, lo que permite usar lambdas que lancen excepciones comprobadas (*checked exceptions*).

La clase verifica tres aspectos fundamentales:

1. **Ejecución y propagación de excepciones:** que `Failable.accept(...)`, `Failable.apply(...)`, `Failable.get(...)`, `Failable.run(...)`, `Failable.call(...)`, etc., ejecuten la lambda y, ante un fallo, propaguen la excepción conforme a la política de `Failable.rethrow(...)`:
   - Las `RuntimeException` (ej. `IllegalStateException`) y los `Error` (ej. `OutOfMemoryError`) se relanzan **sin envolver** (se comprueba con `assertSame`).
   - Las `IOException` se envuelven en `UncheckedIOException`.
   - Otras excepciones comprobadas se envuelven en `UndeclaredThrowableException`.
2. **Adaptación a las interfaces del JDK:** que los métodos `Failable.asConsumer`, `asBiConsumer`, `asFunction`, `asBiFunction`, `asPredicate`, `asBiPredicate`, `asSupplier`, `asRunnable` y `asCallable` conviertan correctamente una interfaz *failable* en su contraparte estándar de Java.
3. **Métodos por defecto y constantes de las interfaces:** composición (`andThen`, `compose`), lógica booleana (`and`, `or`, `negate`), constantes (`TRUE`, `FALSE`, `truePredicate()`, `falsePredicate()`), implementaciones neutras (`nop()`) e `identity()`, incluyendo los casos límite documentados en Javadoc que deben lanzar `NullPointerException`.

---

## 2. Clases internas de apoyo (*test fixtures*)

| Clase interna | Propósito |
|---|---|
| `CloseableObject` | Simula un recurso cerrable. Métodos: `close()`, `isClosed()`, `reset()` y `run(Throwable)` (lanza el throwable recibido). Se usa en `testTryWithResources`. |
| `FailureOnInvocationCount` | Falla en la *n-ésima* invocación (`throwOn`). Métodos `inc(int)` (lanza `SomeException`), `incIo(int)` (lanza `IOException`), `incCnfe(int)` (lanza `ClassNotFoundException`) y `reset()`. Se usa en las pruebas de `applyNonNull`. |
| `FailureOnOddInvocations` | Contador estático que lanza `SomeException` en las invocaciones **impares** y tiene éxito en las pares. Permite verificar en una sola prueba el camino de error y el de éxito. Expone `reset()`, `testGetBool()`, `testInt(int)`, `testLong(long)`, `testDouble(double)`, `testInc(int)` y un constructor que también falla en invocaciones impares. |
| `SomeException` | Excepción comprobada propia, con un campo `value` que identifica la invocación que falló y un `Throwable` configurable (`setThrowable`). |
| `Testable<T, P>` | Objeto genérico instrumentado. Almacena un `Throwable` configurable (`setThrowable`) que se lanza al invocar cualquiera de sus métodos `test*`, y registra los argumentos recibidos (`getAcceptedObject()`, `getAcceptedPrimitiveObject1()`, `getAcceptedPrimitiveObject2()`). Ofrece variantes para todas las firmas probadas: `test()`, `test(Object,Object)`, `testInt`, `testLong`, `testDouble`, `testObjInt`, `testObjLong`, `testObjDouble`, `testAsInteger`, `testAsIntPrimitive`, `testAsLongPrimitive`, `testAsDoublePrimitive`, `testAsShortPrimitive`, `testAsBooleanPrimitive`, `testDoubleDouble`. |

### Constantes y preparación

- `ERROR` — instancia compartida de `OutOfMemoryError`.
- `ILLEGAL_STATE_EXCEPTION` — instancia compartida de `IllegalStateException`.
- `@BeforeEach beforeEach()` — reinicia el contador de `FailureOnOddInvocations` antes de cada prueba.
- `throwingFunction(String)` — método privado auxiliar que declara `throws Exception`; se usa para probar `FailableFunction.function(...)`.

---

## 3. Métodos de prueba principales

Todos los métodos están anotados con `@Test` y no reciben parámetros ni devuelven valores. Se agrupan por funcionalidad:

### 3.1 Ejecución de consumidores (`Failable.accept`)

| Método | Qué valida |
|---|---|
| `testAcceptConsumer()` | `Failable.accept(FailableConsumer, T)`: relanza `IllegalStateException`/`OutOfMemoryError` tal cual y envuelve `IOException` en `UncheckedIOException`; no falla si no hay throwable. |
| `testAcceptBiConsumer()` | Igual que el anterior para `FailableBiConsumer` con dos argumentos. |
| `testAcceptIntConsumer()`, `testAcceptLongConsumer()`, `testAcceptDoubleConsumer()` | Variantes primitivas; además comprueban que el valor **solo** se registra cuando la ejecución es exitosa. |
| `testAcceptObjIntConsumer()`, `testAcceptObjLongConsumer()`, `testAcceptObjDoubleConsumer()` | Variantes `Obj*Consumer` (objeto + primitivo); verifican el registro de ambos argumentos. |

### 3.2 Ejecución de funciones (`Failable.apply*`)

| Método | Qué valida |
|---|---|
| `testApplyFunction()` / `testApplyBiFunction()` | Propagación de excepciones y valor de retorno de `Failable.apply` con `FailableFunction` y `FailableBiFunction`. |
| `testApplyDoubleBinaryOperator()` | `Failable.applyAsDouble` con un operador binario de dobles. |
| `testApplyNonNull()` | `Failable.applyNonNull(T, FailableFunction)`: devuelve `null` si la entrada o el resultado es `null`, lanza `NullPointerException` si la función es `null`, y propaga excepciones comprobadas. |
| `testApplyNonNull2()` / `testApplyNonNull3()` | Variantes con 2 y 3 funciones encadenadas; cubren cortocircuito por `null`, funciones nulas y combinaciones de excepciones comprobadas iguales y distintas (`SomeException`, `IOException`). |

### 3.3 Obtención de valores (`Failable.get*`)

`testGetSupplier()`, `testGetFromSupplier()`, `testGetAsIntSupplier()`, `testGetAsLongSupplier()`, `testGetAsDoubleSupplier()`, `testGetAsShortSupplier()`, `testGetAsBooleanSupplier()` — comprueban las sobrecargas `get`, `getAsInt`, `getAsLong`, `getAsDouble`, `getAsShort` y `getAsBoolean`, tanto en la ruta de error como en la de éxito.

### 3.4 Ejecución de código sin retorno

- `testRunnable()` — `Failable.run(FailableRunnable)`: envuelve `SomeException` en `UndeclaredThrowableException` en la invocación impar y funciona en la par; acepta `null` sin fallar.
- `testCallable()` — `Failable.run` y `Failable.call` sobre un `FailableCallable`.
- `testTryWithResources()` — `Failable.tryWithResources(...)`: garantiza que el recurso se cierre (`isClosed()`) **siempre**, con o sin excepción, y que la excepción original se propague según la política habitual.

### 3.5 Conversión a interfaces estándar del JDK (`Failable.as*`)

`testAsConsumer()`, `testBiConsumer()`, `testFunction()`, `testBiFunction()`, `testPredicate()`, `testBiPredicate()`, `testAsSupplier()`, `testAsRunnable()`, `testAsCallable()` — verifican que los adaptadores produzcan instancias válidas de `Consumer`, `BiConsumer`, `Function`, `BiFunction`, `Predicate`, `BiPredicate`, `Supplier`, `Runnable` y `Callable`, y que las excepciones comprobadas aparezcan como causa de `UndeclaredThrowableException` o `UncheckedIOException`.

### 3.6 Métodos por defecto: composición

- **`andThen`:** `testConsumerAndThen`, `testBiConsumerAndThen`, `testByteConsumerAndThen`, `testIntConsumerAndThen`, `testLongConsumerAndThen`, `testDoubleConsumerAndThen`, `testFunctionAndThen`, `testBiFunctionAndThen`, `testIntUnaryOperatorAndThen`, `testLongUnaryOperatorAndThen`, `testDoubleUnaryOperatorAndThen`.
- **`compose`:** `testFunctionCompose`, `testIntUnaryOperatorCompose`, `testLongUnaryOperatorCompose`, `testDoubleUnaryOperatorCompose`.
- **`identity()`:** `testFunctionIdentity`, `testIntUnaryOperatorIdentity`, `testLongUnaryOperatorIdentity`, `testDoubleUnaryOperatorIdentity`.
- **Fábrica:** `testFunctionFunction()` valida `FailableFunction.function(...)` combinado con `andThen`.

En todos los casos se comprueba que la excepción del primer o del segundo eslabón se propague y que pasar `null` lance `NullPointerException` (caso límite documentado en Javadoc).

### 3.7 Predicados *failable*

`testPredicateAnd/Or/Negate`, `testBiPredicateAnd/Or/Negate`, `testIntPredicateAnd/Or/Negate`, `testLongPredicateAnd/Or/Negate`, `testDoublePredicateAnd/Or/Negate`, más `testIntPredicate`, `testLongPredicate`, `testDoublePredicate` — validan la tabla de verdad de `and`/`or`, la inversión con `negate()`, las constantes `TRUE`/`FALSE` y los métodos `truePredicate()`/`falsePredicate()`, además del rechazo de argumentos `null`.

### 3.8 Implementaciones neutras (`nop()`)

Conjunto de pruebas `testFailable*Nop()` (p. ej. `testFailableBiFunctionNop`, `testFailableToIntFunctionNop`, `testFailableObjLongConsumerNop`, `testFailableToBooleanFunctionNop`, …) que verifican el contrato de las implementaciones *no-operation*: los consumidores no lanzan nada, las funciones que devuelven objetos retornan `null`, las que devuelven numéricos retornan `0` y las booleanas retornan `false`.

### 3.9 Pruebas de declaración de excepciones (`testThrows_*`)

Familia muy numerosa de métodos con el patrón:

```
testThrows_<Interfaz>_<TiposGenéricos>()
```

Cada uno crea una implementación anónima de una interfaz *failable* que lanza `IOException` y comprueba con `assertThrows(IOException.class, ...)` que la excepción puede propagarse. Se prueban dos parametrizaciones por interfaz:

- **Caso genérico máximo:** tipos `Object` / `Throwable` (p. ej. `testThrows_FailableFunction_Object_Throwable`).
- **Caso concreto:** tipos `String` / `IOException` (p. ej. `testThrows_FailableFunction_String_IOException`).

Cubre, entre otras: `FailableBiConsumer`, `FailableBiFunction`, `FailableBiPredicate`, `FailableBooleanSupplier`, `FailableByteConsumer`, `FailableByteSupplier`, `FailableCallable`, `FailableConsumer`, `FailableDoubleBinaryOperator`, `FailableDoubleConsumer`, `FailableDoubleFunction`, `FailableDoubleSupplier`, `FailableDoubleToIntFunction`, `FailableDoubleToLongFunction`, `FailableFunction`, `FailableIntBinaryOperator`, `FailableIntConsumer`, `FailableIntFunction`, `FailableIntSupplier`, `FailableIntToDoubleFunction`, `FailableIntToFloatFunction`, `FailableIntToLongFunction`, `FailableLongBinaryOperator`, `FailableLongConsumer`, `FailableLongFunction`, `FailableLongSupplier`, `FailableLongToDoubleFunction`, `FailableLongToIntFunction`, `FailableObjDoubleConsumer`, `FailableObjIntConsumer`, `FailableObjLongConsumer`, `FailablePredicate`, `FailableRunnable`, `FailableShortSupplier`, `FailableSupplier`, `FailableToBooleanFunction`, `FailableToDoubleBiFunction`, `FailableToDoubleFunction`, `FailableToIntBiFunction`, `FailableToIntFunction`, `FailableToLongBiFunction` y `FailableToLongFunction`.

**Objetivo de este grupo:** garantizar (en tiempo de compilación y de ejecución) que la declaración genérica `throws T` está correctamente definida y no impide lanzar excepciones comprobadas arbitrarias.

---

## 4. Dependencias relevantes

- **JUnit 5:** `@Test`, `@BeforeEach` y aserciones estáticas (`assertEquals`, `assertSame`, `assertThrows`, `assertInstanceOf`, `assertNull`, `assertNotNull`, `assertTrue`, `assertFalse`).
- **Commons Lang:** `LangAssertions.assertNullPointerException`, `StringUtils`, `AbstractLangTest`.
- **JDK:** `java.io.IOException`, `java.io.UncheckedIOException`, `java.lang.reflect.UndeclaredThrowableException`, `java.util.concurrent.Callable` y el paquete `java.util.function`.

---

## 5. Notas de uso y mantenimiento

- La clase **no contiene lógica de producción**; cualquier cambio en `Failable` o en las interfaces `Failable*` debe reflejarse aquí añadiendo o ajustando los métodos de prueba correspondientes.
- El patrón recomendado para nuevas interfaces es replicar: (a) una prueba `testThrows_<Interfaz>_Object_Throwable`, (b) una prueba `testThrows_<Interfaz>_String_IOException`, (c) una prueba `testFailable<Interfaz>Nop` y, si procede, pruebas de `andThen`/`compose`/`and`/`or`/`negate`.
- `FailureOnOddInvocations` usa **estado estático**; el `@BeforeEach` lo reinicia, pero las pruebas también lo reinician explícitamente, por lo que no deben ejecutarse en paralelo sin aislamiento.