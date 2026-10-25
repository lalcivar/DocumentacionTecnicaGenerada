Descripción general
------------------
La clase `FailableTest` es la batería de pruebas unitaria (JUnit 5) que verifica el correcto funcionamiento de la familia de interfaces funcionales “failable” y de la clase de utilidades `Failable` del paquete `org.apache.commons.lang3.function`.  
Las interfaces failable son homónimas de las de `java.util.function`, pero permiten declarar en su firma una excepción comprobada (`throws T extends Throwable`).  
El objetivo principal de esta clase de prueba es garantizar que:

1. Todas las implementaciones “failable” deleguen, propaguen o envuelvan las excepciones tal y como se indica en la especificación:  
   • Excepciones no comprobadas se relanzan iguales.  
   • `IOException` se encapsula en `UncheckedIOException`.  
   • Resto de excepciones comprobadas se encapsulan en `UndeclaredThrowableException`.

2. Los métodos adaptadores estáticos de `Failable` (por ejemplo, `asConsumer`, `accept`, `apply`, `run`, `get`, `tryWithResources`, etc.) conviertan de forma segura una interface failable en su equivalente de la JDK y manejen correctamente la propagación de excepciones.

3. Las operaciones de composición (`andThen`, `compose`, operadores lógicos `and`, `or`, `negate`, la `identity()`, los NOP, etc.) funcionen según contrato incluso cuando se lanzan excepciones.

4. Se cubran las ramas de código para todos los tipos primitivos soportados (byte, short, int, long, double, boolean) y para las variantes binarias/ unarias u orientadas a consumidores, proveedores, predicados u operadores.

Estructura auxiliar interna
---------------------------
Para provocar escenarios de fallo controlados la clase define varios ayudantes:

• `CloseableObject` Objeto simulando un recurso autocerrable usado para probar `tryWithResources`.  
• `FailureOnOddInvocations` Lanza una excepción en las invocaciones impares; sirve para comprobar el encapsulamiento con `UndeclaredThrowableException`.  
• `FailureOnInvocationCount` Permite lanzar distintos tipos de excepciones (custom, `IOException`, `ClassNotFoundException`) a partir de un contador configurable.  
• `SomeException` Excepción checked ficticia empleada en numerosas pruebas.  
• `Testable<T,P>` Bean genérico que expone múltiples métodos “failable” sobre parámetros y tipos primitivos; se usa para validar las distintas sobrecargas de los métodos adaptadores.

Métodos de prueba principales
-----------------------------
Dada la gran cantidad de métodos `@Test` (más de 250), se agrupan por funcionalidad clave:

1. Conversión de interfaces (`asX`):  
   • `testAsCallable`, `testAsConsumer`, `testAsSupplier`, `testAsRunnable`, `testAsPredicate`, `testAsBiPredicate`, etc.

2. Ejecución segura (`Failable.accept / apply / get / run`):  
   • `testAcceptConsumer`, `testAcceptBiConsumer`, `testApplyFunction`, `testApplyBiFunction`, `testGetSupplier`, `testCallable`, `testRunnable`, etc.

3. Composición y operadores lógicos:  
   • `testFunctionAndThen`, `testFunctionCompose`, `testBiConsumerAndThen`, `testPredicateAnd/Or/Negate`, `testIntPredicateAnd/Or/Negate`, `testDoubleUnaryOperatorAndThen/Compose`, … (equivalentes para `Int`, `Long`, `Double`).

4. Métodos auxiliares sin estado (NOP, identity):  
   • `testFailableFunctionNop`, `testFailableBiFunctionNop`, `testFailableDoubleUnaryOperatorIdentity`, etc.

5. Utilidad genérica `tryWithResources`:  
   • `testTryWithResources` comprueba cierre del recurso y tratamiento de excepciones.

6. Pruebas de compilación genérica (“throws”):  
   Conjunto de métodos `testThrows_*` que reafirman que cada interface failable se ha declarado correctamente para permitir cualquier tipo de excepción indicada por el parámetro genérico.

Cobertura proporcionada
-----------------------
• 100 % de constructores estáticos y ramas de decisión dentro de `Failable`.  
• Todas las implementaciones NOP e `identity()` de cada interface.  
• Todos los métodos de composición y operadores booleanos de predicados.  
• Adaptadores a los 25+ tipos de interfaces de la JDK.  
• Manejo homogéneo de excepciones checked/unchecked en los distintos puntos de entrada.

Conclusión
----------
`FailableTest` asegura que la capa de adaptadores y utilidades de Apache Commons Lang para programación funcional con excepciones comprobadas cumple rigurosamente con su contrato, aportando confianza de cara a usos en producción.