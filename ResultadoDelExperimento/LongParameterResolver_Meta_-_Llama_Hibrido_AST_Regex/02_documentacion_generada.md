# Documentación Técnica - `LongParameterResolver`

### 1. Resumen Ejecutivo

`LongParameterResolver` es una implementación de ejemplo de la interfaz `org.junit.jupiter.api.extension.ParameterResolver` del motor JUnit Jupiter (JUnit 5). Su propósito es resolver automáticamente cualquier parámetro de tipo `Long` en constructores o métodos de prueba, inyectando el valor constante `42L`. Incluye soporte extendido para tipos genéricos declarados a nivel de clase en superclases o interfaces, siempre que el tipo genérico sea asignable desde `Long`.

**Paquete:** `org.junit.jupiter.engine.execution.injection.sample`  
**Licencia:** Eclipse Public License v2.0  
**Desde:** 5.0

---

### 2. Especificación de Métodos y Constructores

#### 2.1 Definición de la Clase

```java
public class LongParameterResolver implements ParameterResolver
```

| Elemento | Detalle |
| :--- | :--- |
| **Modificador** | `public` |
| **Superclase** | `java.lang.Object` |
| **Interfaces Implementadas** | `org.junit.jupiter.api.extension.ParameterResolver` |
| **Anotaciones** | Ninguna a nivel de clase |

**Documentación Javadoc de Clase:**

```java
/**
 * Example {@link ParameterResolver} that always resolves a {@link Long}
 * parameter to {@code 42}.
 * <p>This resolver also <em>attempts</em> to support generic parameter type
 * declarations if the generic type is defined at the class level in a
 * superclass or interface (extended or implemented by the test class) and
 * is assignable from {@link Long}.
 *
 * @since 5.0
 */
```

#### 2.2 Constructor

##### `LongParameterResolver()`

Constructor por defecto implícito provisto por el compilador al no declararse ningún otro constructor.

```java
/**
 * Construye una nueva instancia de {@code LongParameterResolver}.
 * <p>Constructor por defecto sin argumentos requerido por la plataforma
 * JUnit Jupiter para la instanciación vía ServiceLoader y la API
 * de extensiones {@code @ExtendWith}.
 *
 * @since 5.0
 */
public LongParameterResolver()
```

| Atributo | Valor |
| :--- | :--- |
| **Visibilidad** | `public` |
| **Parámetros** | Ninguno |
| **Excepciones** | Ninguna (`@throws` no aplica) |

#### 2.3 Métodos Detectados

Se han detectado **2 métodos** en el código fuente. No se detectaron métodos `private`, `protected`, `static` ni clases/métodos anónimos o internos adicionales.

---

##### `supportsParameter()`

```java
@Override
public boolean supportsParameter(ParameterContext parameterContext, ExtensionContext extensionContext)
```

Determina si este resolver soporta la inyección para el parámetro dado. Implementa dos estrategias de validación: coincidencia exacta de tipo `Long.class` y coincidencia de variable de tipo genérico (`TypeVariable`) declarada en la clase del ejecutable.

**Javadoc Completo:**

```java
/**
 * Determina si este {@code ParameterResolver} soporta la resolución del parámetro
 * proporcionado.
 * <p>Retorna {@code true} si se cumple alguna de las siguientes condiciones:
 * <ol>
 *   <li>El tipo del parámetro es exactamente {@link Long}.</li>
 *   <li>El tipo parametrizado del método coincide por nombre con una variable de tipo
 *       genérico declarada en la clase del ejecutable y el bound de dicha variable
 *       es una {@code Class} asignable desde {@link Long}.</li>
 * </ol>
 *
 * @param parameterContext contexto del parámetro a resolver, provee acceso a
 *                         {@link java.lang.reflect.Parameter} y al tipo parametrizado
 * @param extensionContext contexto de la extensión de JUnit Jupiter para la invocación actual
 * @return {@code true} si el parámetro es de tipo {@code Long} o es una variable genérica
 *         compatible con {@code Long}; {@code false} en caso contrario
 * @throws NullPointerException si {@code parameterContext} o {@code extensionContext} es {@code null}
 *                              (comportamiento heredado de la API de JUnit)
 */
```

**Tabla de Especificación:**

| Tag Javadoc | Nombre | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- | :--- |
| `@param` | `parameterContext` | `ParameterContext` | Sí | Contexto que contiene el `Parameter` y el `Type` parametrizado del método/constructor. |
| `@param` | `extensionContext` | `ExtensionContext` | Sí | Contexto de ejecución de la extensión. No es utilizado en la lógica actual pero es requerido por la interfaz. |
| `@return` | - | `boolean` | - | `true` si el resolver puede inyectar el parámetro, `false` si no. |
| `@throws` | `NullPointerException` | - | - | Si alguno de los contextos es `null`. No lanzada explícitamente en el código, pero posible por invocación a `getParameter()`. |

**Lógica Interna Detallada:**
1.  `parameterContext.getParameter().getType() == Long.class`: Verificación de igualdad por referencia.
2.  `parameterContext.getParameter().getParameterizedType()`: Obtiene el `Type` genérico.
3.  `parameterContext.getDeclaringExecutable().getDeclaringClass().getTypeParameters()`: Itera sobre las `TypeVariable<?>` de la clase declarante.
4.  Valida `namesMatch` y `typesAreCompatible` (bound único de tipo `Class` y `isAssignableFrom(Long.class)`).

---

##### `resolveParameter()`

```java
@Override
public Long resolveParameter(ParameterContext parameterContext, ExtensionContext extensionContext)
```

Resuelve y provee el valor a inyectar para el parámetro soportado.

**Javadoc Completo:**

```java
/**
 * Resuelve el valor del parámetro para la invocación de la prueba.
 * <p>Este método es invocado por el motor de JUnit Jupiter únicamente si una
 * invocación previa a {@link #supportsParameter(ParameterContext, ExtensionContext)}
 * retornó {@code true} para el mismo parámetro.
 *
 * @param parameterContext contexto del parámetro a resolver
 * @param extensionContext contexto de la extensión de JUnit Jupiter para la invocación actual
 * @return un objeto {@link Long} con el valor constante {@code 42L}, nunca {@code null}
 * @throws org.junit.jupiter.api.extension.ParameterResolutionException si ocurre un error
 *         durante la resolución (definido en la interfaz, no lanzado explícitamente aquí)
 */
```

**Tabla de Especificación:**

| Tag Javadoc | Nombre | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- | :--- |
| `@param` | `parameterContext` | `ParameterContext` | Sí | Contexto del parámetro a resolver. No es utilizado en la implementación actual. |
| `@param` | `extensionContext` | `ExtensionContext` | Sí | Contexto de ejecución de la extensión. No es utilizado en la implementación actual. |
| `@return` | - | `Long` | - | Instancia `Long` con valor `42L`. Autoboxing de `long` primitivo. |
| `@throws` | `ParameterResolutionException` | - | - | Definida en la firma de la interfaz `ParameterResolver`. Esta implementación no la lanza. |

---

### 3. Ejemplo de Uso

#### Caso 1: Inyección Directa de `Long`

```java
package org.example;

import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.junit.jupiter.engine.execution.injection.sample.LongParameterResolver;

import static org.junit.jupiter.api.Assertions.assertEquals;

@ExtendWith(LongParameterResolver.class)
class LongInjectionTest {

    @Test
    void testConInyeccionDirecta(Long numero) {
        // El resolver inyecta automáticamente 42L
        assertEquals(42L, numero);
    }

    @Test
    void testConMultiplesParametros(Long numero, String otroParametro) {
        // Solo los parámetros Long son resueltos por LongParameterResolver
        // 'otroParametro' deberá ser resuelto por otro resolver o fallará
        assertEquals(42L, numero);
    }
}
```

#### Caso 2: Soporte para Tipo Genérico a Nivel de Clase

```java
package org.example;

import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.junit.jupiter.engine.execution.injection.sample.LongParameterResolver;

import static org.junit.jupiter.api.Assertions.assertEquals;

// El tipo genérico T está acotado a Number, por lo que Long es asignable
abstract class GenericBase<T extends Number> {}

@ExtendWith(LongParameterResolver.class)
class GenericInjectionTest extends GenericBase<Long> {

    @Test
    void testConParametroGenerico(T valorGenerico) {
        // supportsParameter() detectará que T coincide con la TypeVariable
        // de GenericBase y que su bound (Number) es asignable desde Long
        assertEquals(42L, valorGenerico);
    }
}
```

