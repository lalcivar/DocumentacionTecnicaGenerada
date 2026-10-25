==============================
Documentación técnica de clase
==============================

Nombre completo  
‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾  
org.junit.jupiter.engine.execution.injection.sample.LongParameterResolver

Responsabilidad  
‾‾‾‾‾‾‾‾‾‾‾‾‾‾  
LongParameterResolver implementa la interfaz org.junit.jupiter.api.extension.ParameterResolver con el objetivo de suministrar, de forma automática, valores de tipo java.lang.Long a los parámetros de métodos de prueba ejecutados por JUnit Jupiter (JUnit 5).  

Su comportamiento es deliberadamente simple y determinista: siempre inyecta el valor 42 L en cualquier parámetro que:

1. Sea exactamente de tipo java.lang.Long, o  
2. Haya sido declarado como un tipo genérico cuyo límite (upper bound) es asignable desde Long y que esté definido en la firma de la clase (no en el método).  

De esta manera, los desarrolladores pueden escribir pruebas más concisas sin preocuparse por la creación explícita del objeto Long.

Contexto de uso típico  
‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾  
```
class MyTest {

    @ParameterizedTest
    void testWithLong(Long value) {      // Se inyecta 42L
        ...
    }

    // Ejemplo con genérico a nivel de clase
    static class GenericTest<T extends Number> {

        @Test
        void testWithGeneric(T number) { // Se inyecta 42L si T == Long
            ...
        }
    }
}
```

Principales métodos públicos  
‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾

1. boolean supportsParameter(ParameterContext parameterContext,
                              ExtensionContext extensionContext)

   • Propósito: Indica a JUnit si este resolver es capaz de manejar el parámetro solicitado.  
   • Lógica detallada:  
     a. Comprueba si el tipo del parámetro es exactamente Long.class.  
     b. De no ser así, analiza el tipo parametrizado del método.  
        - Recorre todas las variables de tipo (TypeVariable) declaradas en la clase que contiene al método.  
        - Para cada variable verifica:  
          i.  Coincidencia de nombre con el tipo encontrado en el parámetro.  
          ii. Que el límite superior de la variable sea un único tipo Class<?> y que dicho tipo sea asignable desde Long (p.ej. Number, Object o Long).  
        - Si ambos criterios se cumplen, devuelve true.  
   • Retorno: true si este resolver puede aportar un valor al parámetro; false en caso contrario.

2. Long resolveParameter(ParameterContext parameterContext,
                         ExtensionContext extensionContext)

   • Propósito: Devuelve el valor que será inyectado en tiempo de ejecución cuando supportsParameter haya devuelto true.  
   • Implementación: Retorna siempre el literal 42L.  
   • Motivo de diseño: Mantener un ejemplo claro y reproducible, a la vez que demuestra la capacidad de inyectar objetos no triviales (wrappers).

Detalles de implementación y consideraciones  
‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾  
• No se realiza ningún uso del parámetro ExtensionContext; la firma se mantiene para cumplir la interfaz.  
• El algoritmo de compatibilidad genérica es intencionalmente restrictivo: sólo cubre el caso de un único límite superior y no examina jerarquías más profundas ni wildcards.  
• El valor hard-coded (42) tiene fines demostrativos y puede ser sustituido por cualquier lógica de negocio si se reutiliza el resolver.  
• Thread-safety: La clase es stateless; por lo tanto, es segura para uso concurrente por el motor de pruebas.

Desde  
‾‾‾‾  
JUnit 5.0