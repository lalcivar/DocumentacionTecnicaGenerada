# Documentación Técnica: `NumberParameterResolver`

**Paquete:** `org.junit.jupiter.engine.execution.injection.sample`  
**Tipo:** Clase Pública  
**Implementa:** `org.junit.jupiter.api.extension.ParameterResolver`  
**Licencia:** Eclipse Public License v2.0  
**Desde:** 5.0

---

### 1. Resumen Ejecutivo

`NumberParameterResolver` es una implementación de la interfaz `ParameterResolver` de JUnit Jupiter 5 diseñada como caso de prueba para la inyección de parámetros. Su propósito es declarar soporte para cualquier parámetro de tipo `Number` o subtipo mediante `supportsParameter()` y resolverlo incondicionalmente a un valor `Integer` con valor `42` mediante `resolveParameter()`.

Esta clase simula un escenario de incompatibilidad de tipos en tiempo de ejecución, donde el resolutor afirma poder resolver un tipo genérico pero entrega un subtipo específico no asignable, provocando un `IllegalArgumentException: argument type mismatch` si el método de prueba espera un subtipo distinto a `Integer` (ej. `Double`).

### 2. Especificación de Métodos y Constructores

Análisis estático del código fuente provisto. Se detectaron **3 miembros ejecutables**: 1 constructor implícito por defecto y 2 métodos públicos de instancia sobrescritos. No se detectaron métodos `private`, `protected`, `static` ni clases/métodos anónimos o internos.

#### 2.1. Información a Nivel de Clase

```java
/**
 * This is a non-realistic {@link ParameterResolver} that claims to
 * resolve any {@link Number}, when in fact it always resolves an {@link Integer}.
 *
 * <p>For example, consider the case with Spring: the {@code SpringExtension} can
 * theoretically resolve any type of {@code ApplicationContext}, but if the
 * required parameter type is {@code WebApplicationContext} and the user
 * neglects to annotate the test class with {@code @WebAppConfiguration} then
 * the {@code ApplicationContext} loaded will not be assignment compatible.
 *
 * @author original author or authors
 * @since 5.0
 * @see org.junit.jupiter.api.extension.ParameterResolver
 * @see java.lang.Number
 * @see java.lang.Integer
 */
public class NumberParameterResolver implements ParameterResolver
```

#### 2.2. Constructor `NumberParameterResolver()`

Constructor por defecto implícito generado por el compilador al no declararse ningún otro constructor.

> **Firma:** `public NumberParameterResolver()`

| Atributo | Detalle |
| :--- | :--- |
| **Visibilidad** | `public` |
| **Tipo** | Constructor |
| **Descripción** | Crea una nueva instancia de `NumberParameterResolver`. No realiza inicialización adicional. |

**Documentación Javadoc:**

```java
/**
 * Construye una nueva instancia de {@code NumberParameterResolver}.
 */
public NumberParameterResolver()
```

| Tag | Descripción |
| :--- | :--- |
| `@param` | No aplica. No recibe parámetros. |
| `@return` | No aplica. Constructor. |
| `@throws` | No lanza excepciones verificadas. |

#### 2.3. Método `supportsParameter()`

Determina si el resolutor soporta la inyección para un parámetro dado.

> **Firma:** `public boolean supportsParameter(ParameterContext parameterContext, ExtensionContext extensionContext)`

**Documentación Javadoc:**

```java
/**
 * Determina si este resolutor soporta el parámetro proporcionado.
 * <p>Retorna {@code true} si el tipo del parámetro es asignable desde
 * {@link Number}, es decir, si es {@code Number} o cualquier subtipo
 * como {@code Integer}, {@code Double}, {@code Long}, etc.
 *
 * @param parameterContext el contexto del parámetro que contiene la información
 *                         del parámetro a resolver, no debe ser {@code null}
 * @param extensionContext el contexto de la extensión de JUnit Jupiter, no debe ser {@code null}
 * @return {@code true} si {@code Number.class.isAssignableFrom(parameterContext.getParameter().getType())}, 
 *         de lo contrario {@code false}
 * @throws NullPointerException si {@code parameterContext} es {@code null}
 */
@Override
public boolean supportsParameter(ParameterContext parameterContext, ExtensionContext extensionContext)
```

**Tabla de Parámetros (`@param`):**

| Nombre | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `parameterContext` | `ParameterContext` | Sí | Contexto que encapsula el `java.lang.reflect.Parameter` del método/constructor de prueba. Se utiliza para obtener `getParameter().getType()`. |
| `extensionContext` | `ExtensionContext` | Sí | Contexto de ejecución de la extensión. No es utilizado en la implementación actual, pero es requerido por el contrato de la interfaz. |

**Tabla de Retorno (`@return`):**

| Tipo | Descripción |
| :--- | :--- |
| `boolean` | `true` si el tipo del parámetro es `Number` o un subtipo. `false` en caso contrario. |

**Tabla de Excepciones (`@throws`):**

| Excepción | Condición |
| :--- | :--- |
| `NullPointerException` | Si `parameterContext` es `null` al invocar `parameterContext.getParameter()`. No declarada explícitamente pero posible por desreferenciación. |

#### 2.4. Método `resolveParameter()`

Resuelve y provee el valor a inyectar para el parámetro soportado.

> **Firma:** `public Object resolveParameter(ParameterContext parameterContext, ExtensionContext extensionContext)`

**Documentación Javadoc:**

```java
/**
 * Resuelve el valor del parámetro a inyectar.
 * <p>Esta implementación ignora el contexto y siempre retorna un valor
 * constante de tipo {@link Integer} con valor {@code 42}, independientemente
 * del subtipo de {@link Number} solicitado. Esto puede provocar un
 * {@link IllegalArgumentException} por incompatibilidad de tipos si el
 * parámetro declarado no es asignable desde {@code Integer} (ej. {@code Double}).
 *
 * @param parameterContext el contexto del parámetro a resolver, no debe ser {@code null}
 * @param extensionContext el contexto de la extensión de JUnit Jupiter, no debe ser {@code null}
 * @return un objeto {@link Integer} con valor {@code 42}
 * @throws IllegalArgumentException si JUnit intenta inyectar el {@code Integer} retornado
 *         en un parámetro de tipo incompatible (ej. {@code Double}) durante la invocación del método
 */
@Override
public Object resolveParameter(ParameterContext parameterContext, ExtensionContext extensionContext)
```

**Tabla de Parámetros (`@param`):**

| Nombre | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `parameterContext` | `ParameterContext` | Sí | Contexto del parámetro. No es utilizado en la lógica actual. |
| `extensionContext` | `ExtensionContext` | Sí | Contexto de ejecución de la extensión. No es utilizado en la lógica actual. |

**Tabla de Retorno (`@return`):**

| Tipo | Descripción |
| :--- | :--- |
| `Object` | Instancia de `java.lang.Integer` con valor `42`. Autoboxing de `int` a `Integer`. |

**Tabla de Excepciones (`@throws`):**

| Excepción | Condición |
| :--- | :--- |
| `IllegalArgumentException` | No lanzada directamente por este método, pero provocada por el motor de JUnit (`org.junit.jupiter.engine`) al intentar invocar el método de prueba si el tipo retornado (`Integer`) no es asignable al tipo del parámetro declarado (ej. `Double`, `Long`). Mensaje típico: `argument type mismatch`. |

> **Nota de Cobertura Total:** Se han documentado el 100% de los métodos presentes en el código fuente: `NumberParameterResolver()`, `supportsParameter()` y `resolveParameter()`. No existen métodos `private`, `protected`, `static` o clases internas que documentar.

---

### 3. Ejemplo de Uso

Caso de uso real que demuestra el comportamiento esperado y el caso de fallo por incompatibilidad de tipos.

```java
package org.junit.jupiter.engine.execution.injection.sample;

import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertTrue;

// Registro del resolutor a nivel de clase
@ExtendWith(NumberParameterResolver.class)
class NumberParameterResolverTest {

    /**
     * Caso de éxito: El parámetro es de tipo Number (supertipo de Integer).
     * supportsParameter() retorna true y resolveParameter() inyecta 42 correctamente.
     */
    @Test
    void testConNumberGenerico(Number numero) {
        assertTrue(numero instanceof Integer);
        assertEquals(42, numero.intValue());
    }

    /**
     * Caso de éxito: El parámetro es de tipo Integer (tipo exacto retornado).
     */
    @Test
    void testConIntegerEspecifico(Integer numero) {
        assertEquals(42, numero);
    }

    /**
     * Caso de fallo: El parámetro es de tipo Double.
     * supportsParameter() retorna true (porque Double es un Number),
     * pero resolveParameter() retorna un Integer (42).
     * JUnit lanzará IllegalArgumentException: argument type mismatch
     * al intentar invocar este método, ya que Integer no es asignable a Double.
     */
    @Test
    void testConDoubleFallaraPorIncompatibilidad(Double numero) {
        // Este test nunca se ejecutará correctamente.
        // Fallará en la fase de inyección de parámetros.
    }
}
```

**Explicación del flujo:**
1. JUnit detecta la anotación `@ExtendWith(NumberParameterResolver.class)`.
2. Antes de invocar `testConNumberGenerico`, invoca `supportsParameter()` -> `Number.class.isAssignableFrom(Number.class)` -> `true`.
3. Invoca `resolveParameter()` -> retorna `42` (Integer).
4. Inyecta `42` como argumento `numero` y ejecuta el test con éxito.
5. En `testConDoubleFallaraPorIncompatibilidad`, el paso 2 también retorna `true`, pero la inyección falla por incompatibilidad de tipos.
