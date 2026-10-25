# Documentación Técnica - `org.junit.jupiter.api.Tag`

### 1. Resumen Ejecutivo

`Tag` es una anotación repetible de tipo `public @interface` perteneciente al paquete `org.junit.jupiter.api` del framework JUnit 5. Su propósito es declarar una o más etiquetas (tags) a nivel de clase o método de prueba para permitir el filtrado selectivo de la ejecución de tests en un *test plan* (ej. incluir/excluir por `fast`, `slow`, `ci-server`). La anotación es retenida en tiempo de ejecución (`RUNTIME`), es heredable (`@Inherited`) y posee estado de API `STABLE` desde la versión 5.0.

### 2. Especificación de Métodos y Constructores

#### 2.1 Definición de la Anotación

| Atributo | Valor |
| :--- | :--- |
| **Nombre Completo** | `org.junit.jupiter.api.Tag` |
| **Tipo** | `public @interface` `Annotation` |
| **Paquete** | `org.junit.jupiter.api` |
| **Estado API** | `@API(status = STABLE, since = "5.0")` |
| **Meta-anotaciones** | `@Target({ElementType.TYPE, ElementType.METHOD})`, `@Retention(RetentionPolicy.RUNTIME)`, `@Documented`, `@Inherited`, `@Repeatable(Tags.class)` |
| **Desde** | `@since 5.0` |
| **Anotación Contenedora** | `org.junit.jupiter.api.Tags` |
| **Licencia** | Eclipse Public License v2.0 |

> **Análisis de Cobertura al 100%:** Tras el análisis estático exhaustivo del código fuente provisto, se ha detectado **un (1) único elemento de anotación** declarado explícitamente. No se detectaron constructores, métodos `private`, `protected`, `static`, ni clases/métodos anónimos o internos. Al ser un tipo anotación (`@interface`), no posee constructores explícitos y no puede instanciarse directamente.

#### 2.2 Elementos de la Anotación (Métodos Abstractos)

La anotación declara un único miembro abstracto que actúa como método.

##### `value()`

Define el valor textual de la etiqueta. El valor será procesado mediante `String.strip()` antes de su validación. Si tras el *trim* el tag es sintácticamente inválido, se registrará como *warning* y será ignorado.

**Definición Javadoc:**

```java
/**
 * The <em>tag</em>.
 *
 * <p>Note: the tag will first be {@linkplain String#strip() stripped}. If the
 * supplied tag is syntactically invalid after trimming, the error will be
 * logged as a warning, and the invalid tag will be effectively ignored. See
 * {@linkplain Tag Syntax Rules for Tags}.
 *
 * @return el valor del tag ya normalizado (trimmed), nunca {@code null}
 * @since 5.0
 */
String value();
```

**Tabla de Especificación Javadoc para `value()`:**

| Tag Javadoc | Detalle |
| :--- | :--- |
| **Firma** | `String value()` |
| **Modificador** | `public abstract` (implícito en `@interface`) |

**@param - Parámetros de Entrada:**

| Nombre | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| *N/A* | *N/A* | *N/A* | Este elemento no recibe parámetros. Es un accesor del valor de la anotación. |

**@return - Valor de Retorno:**

| Tipo | Descripción |
| :--- | :--- |
| `String` | Cadena que representa el tag. No debe ser vacía (*blank*). El valor retornado es el definido en la declaración de la anotación. |

**@throws - Manejo de Excepciones:**

| Excepción | Condición |
| :--- | :--- |
| *Ninguna* | El método no declara `throws` explícito. La validación de sintaxis no lanza excepción; los tags inválidos son ignorados y logueados como *warning* en tiempo de ejecución por el motor de JUnit Platform. |

**Reglas de Sintaxis Validadas para el valor de `value()`:**

1.  No debe ser vacío o en blanco.
2.  Tras aplicar `strip()`, no debe contener espacios en blanco.
3.  Tras aplicar `strip()`, no debe contener caracteres de control ISO.
4.  Tras aplicar `strip()`, no debe contener caracteres reservados: `,` `(` `)` `&` `|` `!`

#### 2.3 Inventario Completo de Miembros Detectados

| Visibilidad | Tipo | Nombre Exacto | Estado |
| :--- | :--- | :--- | :--- |
| `public` | Elemento de Anotación | `value()` | **Documentado** |
| `private` | - | - | No detectado en el código fuente |
| `protected` | - | - | No detectado en el código fuente |
| `static` | - | - | No detectado en el código fuente |
| `constructor` | - | - | No aplicable para `@interface` |
| `interno / anónimo` | - | - | No detectado en el código fuente |

### 3. Ejemplo de Uso

Caso de uso real: Etiquetado de tests por velocidad y entorno para filtrado en el plan de ejecución con Maven Surefire / Gradle.

```java
package com.ejemplo.proyecto;

import org.junit.jupiter.api.Tag;
import org.junit.jupiter.api.Tags;
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.assertTrue;

// Tag a nivel de clase: todos los tests de esta clase heredan el tag "regression"
@Tag("regression")
@Tag("ci-server")
class CalculadoraTest {

    @Test
    @Tag("fast")
    @Tag("unit")
    void sumaNumerosPositivos() {
        assertTrue(2 + 3 == 5);
    }

    @Test
    @Tag("slow")
    @Tag("integration")
    void operacionLentaConBaseDeDatos() {
        // Test que requiere recursos externos y es excluido en ejecuciones rápidas
        assertTrue(true);
    }

    // Uso de la anotación contenedora @Tags para múltiples tags (equivalente a repetir @Tag)
    @Test
    @Tags({ @Tag("slow"), @Tag("database") })
    void testConTagsContenedora() {
        assertTrue(true);
    }
}
```

**Filtrado en la ejecución (ej. `pom.xml`):**

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <configuration>
        <groups>fast | unit</groups>
        <excludedGroups>slow</excludedGroups>
    </configuration>
</plugin>
```

Con esta configuración, `sumaNumerosPositivos()` se ejecutará, mientras que `operacionLentaConBaseDeDatos()` será excluido del *test plan*.
