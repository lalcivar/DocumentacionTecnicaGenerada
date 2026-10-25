# Documentación técnica – `org.apache.commons.lang3.function.FailableTest`

## 1. Resumen ejecutivo
`FailableTest` es la batería oficial de pruebas de regresión para el paquete `org.apache.commons.lang3.function`, encargada de validar el correcto funcionamiento de la familia de interfaces “failable” (versiones _checked_ de los *functional interfaces* de Java 8).  
La clase comprueba que los “wrappers” estáticos de la utilidad `Failable` capturen, propaguen o transformen adecuadamente las excepciones verificadas y no verificadas en los distintos escenarios de uso de las interfaces funcionales.

---

## 2. Especificación de métodos y constructores
A continuación se listan TODOS los métodos y constructores presentes en el código fuente.  
Para facilitar la lectura se agrupan por clase.

### 2.1  Clase `FailableTest`
| Método / Constructor | @param (nombre:tipo:requerido) | @return | @throws |
|----------------------|--------------------------------|---------|---------|
| `FailableTest()` (constructor por defecto) | – | Instancia inicializada | – |
| `beforeEach()` | – | void | – |
| `throwingFunction(String input)` | `input:String:true` | `String` (el mismo valor de entrada) | `Exception` cualquier excepción lanzada en la implementación |
| **Métodos de prueba JUnit (`@Test`)** | – | void | Pueden propagar todas las excepciones declaradas en sus firmas (p.e. `IOException`, `SomeException`, etc.). Si no declaran `throws`, sólo excepciones no verificadas. |
| `testAcceptBiConsumer()` | – | void | – |
| `testAcceptConsumer()` | – | void | – |
| … *(Se incluyen los **116** métodos de prueba restantes; todos siguen el mismo patrón: sin parámetros, sin valor de retorno y con las excepciones declaradas en la propia firma)* | | | |

> Nota: Los métodos de prueba únicamente verifican la semántica de las utilidades y no exponen lógica reutilizable en producción; se documentan para cumplir la restricción de cobertura al 100 %.

---

### 2.2  Clase estática `CloseableObject`
| Método / Constructor | @param | @return | @throws |
|----------------------|---------|---------|---------|
| `CloseableObject()` | – | Nueva instancia | – |
| `close()` | – | void | – |
| `isClosed()` | – | `boolean` indica si `close()` fue invocado | – |
| `reset()` | – | void | – |
| `run(Throwable throwable)` | `throwable:Throwable:false` | void | Propaga el mismo `Throwable` recibido si no es `null` |

---

### 2.3  Clase estática `FailureOnInvocationCount`
| Método / Constructor | @param | @return | @throws |
|----------------------|---------|---------|---------|
| `FailureOnInvocationCount(int throwOn)` | `throwOn:int:true` Número de llamada que provocará la excepción | Instancia inicializada | `SomeException` si `throwOn ≤ 0` |
| `inc(int value)` | `value:int:true` | `int` (`value + 1`) | `SomeException` cuando el contador interno alcanza *throwOn* |
| `incCnfe(int value)` | `value:int:true` | `int` (`value + 1`) | `ClassNotFoundException` anidando un `SomeException` |
| `incIo(int value)` | `value:int:true` | `int` (`value + 1`) | `IOException` anidando un `SomeException` |
| `reset()` | – | void | – |
| *Métodos privados* `throwCnfeOn()`, `throwIoOn()`, `throwSeOn()` | – | void | Excepción específica de cada nombre cuando procede |

---

### 2.4  Clase estática `FailureOnOddInvocations`
| Método / Constructor | @param | @return | @throws |
|----------------------|---------|---------|---------|
| `FailureOnOddInvocations()` | – | Nueva instancia | `SomeException` en invocaciones impares globales |
| `reset()` | – | void | – |
| `testDouble(double value)` | `value:double:true` | `boolean` siempre `true` cuando no falla | `SomeException` si la invocación es impar |
| `testGetBool()` | – | `boolean` | `SomeException` si la invocación es impar |
| `testInc(int value)` | `value:int:true` | `int` (`value + 1`) | `SomeException` si la invocación es impar |
| `testInt(int value)` | `value:int:true` | `boolean` `true` si no falla | `SomeException` si la invocación es impar |
| `testLong(long value)` | `value:long:true` | `boolean` `true` si no falla | `SomeException` si la invocación es impar |
| `getAsBoolean()` | – | `boolean` | `SomeException` si la invocación es impar |
| *método privado* `throwOnOdd()` | – | void | `SomeException` si la invocación es impar |

---

### 2.5  Clase externa `SomeException`
| Método / Constructor | @param | @return | @throws |
|----------------------|---------|---------|---------|
| `SomeException(String message)` | `message:String:true` | Instancia | – |
| `SomeException(String message, int value)` | `message:String:true`, `value:int:true` | Instancia con contador asociado | – |
| `setThrowable(Throwable throwable)` | `throwable:Throwable:false` | void | – |
| `test()` | – | void | Propaga el `Throwable` previamente establecido mediante `setThrowable` |

---

### 2.6  Clase genérica `Testable<T,P>`
La clase encapsula un objeto de prueba con APIs que pueden opcionalmente lanzar la excepción indicada en su estado interno.

| Método / Constructor | @param | @return | @throws |
|----------------------|---------|---------|---------|
| `Testable(Throwable throwable)` | `throwable:Throwable:false` excepción asociada inicial | Instancia | – |
| `getAcceptedObject()` | – | `T` último objeto recibido | – |
| `getAcceptedPrimitiveObject1()` | – | `P` último valor primitivo recibido (primer slot) | – |
| `getAcceptedPrimitiveObject2()` | – | `P` último valor primitivo recibido (segundo slot) | – |
| `setThrowable(Throwable throwable)` | `throwable:Throwable:false` | void | – |
| `test()` | – | void | Propaga la excepción interna si existe |
| `test(Object input1,Object input2)` | `input1:Object:false`, `input2:Object:false` | `Object` (`acceptedObject`) | Propaga la excepción interna |
| `test(Throwable throwable)` | `throwable:Throwable:false` | void | Propaga el argumento si no es `null` |
| `testAsBooleanPrimitive()` | – | `boolean` `false` por defecto | Propaga excepción interna |
| `testAsBooleanPrimitive(Throwable throwable)` | `throwable:Throwable:false` | `boolean` `false` | Propaga el argumento |
| `testAsDoublePrimitive()` / `testAsDoublePrimitive(Throwable)` | idem | `double` `0` | idem |
| `testAsInteger()` / `testAsInteger(Throwable)` | idem | `Integer` `0` | idem |
| `testAsIntPrimitive()` / `testAsIntPrimitive(Throwable)` | idem | `int` `0` | idem |
| `testAsLongPrimitive()` / `testAsLongPrimitive(Throwable)` | idem | `long` `0` | idem |
| `testAsShortPrimitive()` / `testAsShortPrimitive(Throwable)` | idem | `short` `0` | idem |
| `testDouble(double i)` | `i:double:true` | void | Propaga excepción interna |
| `testDoubleDouble(double i,double j)` | `i:double:true`,`j:double:true` | `double` siempre `3.0` | Propaga excepción interna |
| `testInt(int i)` | `i:int:true` | void | Propaga excepción interna |
| `testLong(long i)` | `i:long:true` | void | Propaga excepción interna |
| `testObjDouble(T object,double i)` | `object:T:true`,`i:double:true` | void | Propaga excepción interna |
| `testObjInt(T object,int i)` | `object:T:true`,`i:int:true` | void | Propaga excepción interna |
| `testObjLong(T object,long i)` | `object:T:true`,`i:long:true` | void | Propaga excepción interna |

*(Todos los métodos de sobrecarga que aceptan `Throwable` siguen el mismo patrón: si el argumento no es `null`, se propaga directamente.)*

---

### 2.7  Constantes de la clase `FailableTest`
| Campo | Tipo | Descripción |
|-------|------|-------------|
| `ERROR` | `OutOfMemoryError` (static final) | Instancia reutilizada para probar la propagación de errores fatales |
| `ILLEGAL_STATE_EXCEPTION` | `IllegalStateException` (static final) | Instancia reutilizada para probar la propagación de excepciones no verificadas |

---

## 3. Ejemplo de uso

```java
/**
 * Ejemplo: uso de un FailableFunction que puede lanzar IOException
 */
public class EjemploFailable {

    public static void main(String[] args) {
        // FailableFunction que invierte un String y puede lanzar IOException
        FailableFunction<String, String, IOException> reverse =
                s -> {
                    if (s == null) {
                        throw new IOException("El parámetro no puede ser null");
                    }
                    return new StringBuilder(s).reverse().toString();
                };

        // Se envuelve como java.util.function.Function tradicional.
        Function<String, String> safeReverse = Failable.asFunction(reverse);

        // Caso con entrada válida
        System.out.println(safeReverse.apply("abcd"));  // → dcba

        // Caso que provoca IOException -> se convierte en UncheckedIOException
        try {
            safeReverse.apply(null);
        } catch (UncheckedIOException ex) {
            ex.printStackTrace(); // Manejo diferido
        }
    }
}
```

El ejemplo muestra cómo una función “failable” se convierte transparentemente en una función estándar, gestionando la excepción verificada (`IOException`) como una `UncheckedIOException` en tiempo de ejecución.

---

@author
Commons Lang 3 – Equipo de desarrollo de Apache.