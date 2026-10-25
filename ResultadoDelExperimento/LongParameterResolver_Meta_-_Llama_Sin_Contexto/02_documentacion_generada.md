# Documentación Técnica - Clase `LongParameterResolver`

### 1. Información General

| Atributo | Detalle |
| :--- | :--- |
| **Nombre de la Clase** | `LongParameterResolver` |
| **Paquete** | `org.junit.jupiter.engine.execution.injection.sample` |
| **Tipo** | Clase Pública |
| **Interfaz Implementada** | `org.junit.jupiter.api.extension.ParameterResolver` |
| **Licencia** | Eclipse Public License v2.0 |
| **Versión JUnit** | Desde 5.0 |
| **Propósito** | Clase de ejemplo / muestra para la inyección de parámetros en JUnit Jupiter |

### 2. Descripción General

`LongParameterResolver` es una implementación de ejemplo de la interfaz de extensión `ParameterResolver` del motor de JUnit 5 Jupiter.

Su función principal es **resolver automáticamente cualquier parámetro de tipo `Long` en métodos de test o constructores, inyectando siempre el valor constante `42L`**.

Esta clase demuestra el mecanismo de inyección de dependencias de JUnit Jupiter a través de su Extension API. No está pensada para uso en producción, sino como referencia para desarrolladores que deseen crear sus propios `ParameterResolver` personalizados.

Además de la resolución directa, la clase implementa una lógica avanzada para soportar **tipos genéricos** declarados a nivel de clase, siempre que sean compatibles con `Long`.

### 3. Herencia y Jerarquía

```
java.lang.Object
  └── org.junit.jupiter.engine.execution.injection.sample.LongParameterResolver
        └── implements org.junit.jupiter.api.extension.ParameterResolver
```

Al implementar `ParameterResolver`, está obligada a implementar los dos métodos del contrato de la extensión.

### 4. Métodos Principales

La clase sobrescribe los dos métodos definidos por la interfaz `ParameterResolver`:

#### 4.1 `supportsParameter(ParameterContext, ExtensionContext)`

```java
@Override
public boolean supportsParameter(ParameterContext parameterContext, ExtensionContext extensionContext)
```

**Descripción:** Determina si el resolver puede resolver un parámetro específico. JUnit invoca este método antes de intentar la inyección para cada parámetro de un test.

**Lógica de funcionamiento:**

1.  **Coincidencia Exacta:** Verifica si el tipo del parámetro es exactamente `Long.class`. Si es así, retorna `true`.
    ```java
    if (parameterContext.getParameter().getType() == Long.class) return true;
    ```

2.  **Soporte para Genéricos:** Si no hay coincidencia exacta, intenta resolver parámetros genéricos. Itera sobre los `TypeVariable` declarados en la clase que contiene el método de test `getDeclaringClass().getTypeParameters()` y verifica dos condiciones:
    *   `namesMatch`: Que el nombre del tipo del parámetro en el método coincida con el nombre de la variable de tipo genérica.
    *   `typesAreCompatible`: Que la variable genérica tenga un único límite `bounds[0]` y que ese límite sea una `Class` asignable desde `Long` (ej: `T extends Number` o `T extends Long`).

    Si ambas condiciones se cumplen, retorna `true`.

3.  Si ninguna condición se cumple, retorna `false`.

**Parámetros:**
*   `parameterContext`: Contexto que contiene información sobre el parámetro a resolver.
*   `extensionContext`: Contexto de la extensión que provee información sobre el test en ejecución.

**Retorno:** `boolean` - `true` si puede resolver el parámetro, `false` en caso contrario.

#### 4.2 `resolveParameter(ParameterContext, ExtensionContext)`

```java
@Override
public Long resolveParameter(ParameterContext parameterContext, ExtensionContext extensionContext)
```

**Descripción:** Provee el valor real que será inyectado en el parámetro del test. Este método solo es invocado por JUnit si `supportsParameter` retornó `true` previamente.

**Lógica de funcionamiento:** Es una implementación trivial que siempre retorna el valor constante `42L`, sin importar el contexto.

**Parámetros:**
*   `parameterContext`: Contexto del parámetro.
*   `extensionContext`: Contexto de la extensión.

**Retorno:** `Long` - Siempre retorna `42L`.

### 5. Flujo de Ejecución en JUnit 5

1.  El usuario registra la extensión en su clase de test con `@ExtendWith(LongParameterResolver.class)`.
2.  JUnit detecta un método de test con un parámetro de tipo `Long` o genérico compatible.
3.  JUnit invoca `supportsParameter()` para consultar si el resolver es aplicable.
4.  Si retorna `true`, JUnit invoca `resolveParameter()` y obtiene `42L`.
5.  JUnit inyecta `42L` como argumento al invocar el método de test.

### 6. Ejemplo de Uso

```java
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;

@ExtendWith(LongParameterResolver.class)
class MiTest {

    @Test
    void testConInyeccion(Long numero) {
        // numero será siempre 42L
        assertEquals(42L, numero);
    }

    // Ejemplo con genéricos
    abstract class GenericTest<T extends Long> {
        @Test
        void testGenerico(T valorGenerico) {
            // valorGenerico también será resuelto a 42L
            assertEquals(42L, valorGenerico);
        }
    }
}
```

### 7. Consideraciones y Limitaciones

*   **Clase de Ejemplo:** Como indica su Javadoc, es una clase `sample`. No debe usarse como lógica de negocio real.
*   **Valor Hardcodeado:** No es configurable, siempre inyecta `42`.
*   **Soporte Genérico Limitado:** Solo soporta variables de tipo definidas a nivel de clase en una superclase o interfaz, con un único `bound` que sea asignable desde `Long`. No soporta genéricos definidos a nivel de método.
*   **No Maneja `long` Primitivo:** Solo soporta el wrapper `Long.class`, no el tipo primitivo `long`.

