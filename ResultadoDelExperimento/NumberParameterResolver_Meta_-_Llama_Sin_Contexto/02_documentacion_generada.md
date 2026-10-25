# Documentación Técnica - Clase `NumberParameterResolver`

### 1. Información General

| Atributo | Detalle |
| :--- | :--- |
| **Nombre de la Clase** | `NumberParameterResolver` |
| **Paquete** | `org.junit.jupiter.engine.execution.injection.sample` |
| **Tipo** | Clase pública |
| **Interfaz Implementada** | `org.junit.jupiter.api.extension.ParameterResolver` |
| **Licencia** | Eclipse Public License v2.0 |
| **Versión / Desde** | JUnit 5.0 |
| **Propósito** | Clase de ejemplo / sample para pruebas del motor de inyección de dependencias de JUnit Jupiter |

### 2. Descripción General

`NumberParameterResolver` es una implementación de la interfaz `ParameterResolver` de JUnit 5 cuya función es resolver parámetros de tipo `Number` en métodos de prueba, clases de test y constructores.

Se trata de una clase **no realista y con fines didácticos/de prueba**. Su particularidad es que declara ser capaz de resolver *cualquier* subtipo de `Number` - `Integer`, `Double`, `Long`, `BigDecimal`, etc. - pero en la práctica **siempre resuelve y retorna un objeto de tipo `Integer` con el valor `42`**.

Esta inconsistencia es intencional. Sirve para simular y probar el escenario donde un `ParameterResolver` afirma que puede soportar un parámetro pero el valor resuelto finalmente no es compatible por asignación con el tipo declarado. Esto provoca un `IllegalArgumentException: argument type mismatch` al momento de invocar el método de prueba.

> **Caso de uso real que simula:** La documentación de la clase cita el ejemplo de `SpringExtension`, que puede resolver cualquier `ApplicationContext`, pero si el test requiere un `WebApplicationContext` y no se ha configurado `@WebAppConfiguration`, el contexto resuelto no será asignable al tipo requerido, generando el mismo error.

Esta clase se utiliza internamente en el módulo `junit-jupiter-engine` para validar el manejo de errores del motor de ejecución `execution.injection`.

### 3. Métodos Principales

La clase implementa los dos métodos definidos por la interfaz `ParameterResolver`:

#### 3.1 `supportsParameter`

```java
@Override
public boolean supportsParameter(ParameterContext parameterContext, ExtensionContext extensionContext)
```

Determina si el resolver puede manejar un parámetro específico.

*   **Función:** Verifica si el tipo del parámetro es `Number` o alguno de sus subtipos.
*   **Lógica:** Utiliza `Number.class.isAssignableFrom(parameterContext.getParameter().getType())`.
*   **Parámetros:**
    *   `parameterContext`: Contexto que contiene información sobre el parámetro a resolver.
    *   `extensionContext`: Contexto de la extensión de JUnit.
*   **Retorno:** `true` si el tipo del parámetro es asignable a `Number`; `false` en caso contrario.

#### 3.2 `resolveParameter`

```java
@Override
public Object resolveParameter(ParameterContext parameterContext, ExtensionContext extensionContext)
```

Resuelve y provee el valor que será inyectado en el parámetro del test.

*   **Función:** Retorna el valor concreto que JUnit inyectará.
*   **Lógica:** No realiza ninguna validación adicional, retorna directamente un literal.
*   **Parámetros:** Los mismos que `supportsParameter`.
*   **Retorno:** `Object` - Siempre retorna un `Integer` con valor `42` autoboxeado. Aunque el tipo de retorno es `Object`, el valor real es `java.lang.Integer`.

### 4. Flujo de Funcionamiento

1.  JUnit Jupiter detecta un método de test con un parámetro de tipo `Number`, por ejemplo: `void test(Double numero)`.
2.  El motor consulta a todos los `ParameterResolver` registrados llamando a `supportsParameter`. `NumberParameterResolver` responde `true` porque `Double` es un subtipo de `Number`.
3.  JUnit invoca `resolveParameter` para obtener el valor a inyectar. El resolver retorna `42` como `Integer`.
4.  El motor intenta invocar el método `test` con el valor `Integer`. Como `Integer` no es asignable a `Double`, la invocación por reflexión falla con `IllegalArgumentException`.

Este flujo permite verificar que el motor de JUnit maneja correctamente los errores de incompatibilidad de tipos.

### 5. Ejemplo de Uso

```java
@ExtendWith(NumberParameterResolver.class)
class MiTest {

    @Test
    void testConIntegerFunciona(Integer numero) {
        // OK: Integer es compatible con el valor resuelto Integer(42)
        assertEquals(42, numero);
    }

    @Test
    void testConDoubleFalla(Double numero) {
        // FALLA: supportsParameter retorna true, pero Integer(42) no es asignable a Double
        // Se lanzará IllegalArgumentException: argument type mismatch
    }
}
```

### 6. Consideraciones Técnicas

*   **No usar en producción:** Es una clase de ejemplo ubicada en el paquete `...sample`. No está diseñada para ser utilizada en tests reales.
*   **Dependencia:** Requiere JUnit Jupiter API `org.junit.jupiter.api:junit-jupiter-api`.
*   **Thread-Safety:** La clase no mantiene estado, por lo que es thread-safe.