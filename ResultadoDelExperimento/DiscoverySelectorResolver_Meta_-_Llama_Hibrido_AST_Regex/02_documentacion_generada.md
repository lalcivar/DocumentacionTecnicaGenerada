# Documentación Técnica: `DiscoverySelectorResolver`

**Paquete:** `org.junit.platform.suite.engine`  
**Tipo:** `final class`  
**Visibilidad:** package-private  
**Versión:** `@since 1.8`

---

### 1. Resumen Ejecutivo

`DiscoverySelectorResolver` es el componente central de resolución de descubrimiento para el `Suite Engine` de JUnit Platform. Su responsabilidad es traducir un `EngineDiscoveryRequest` en una jerarquía de `TestDescriptor` mediante un `EngineDiscoveryRequestResolver` configurado, delegando el descubrimiento específico de cada suite a `SuiteTestDescriptor::discover` y podando finalmente los descriptores vacíos o no descubiertos.

La clase actúa como orquestador stateless entre la API de descubrimiento de la plataforma (`EngineDiscoveryRequest`, `DiscoveryIssueReporter`) y la lógica específica de suites (`IsSuiteClass`, `ClassSelectorResolver`).

---

### 2. Especificación de Métodos y Constructores

#### 2.1. Información de la Clase

| Atributo | Valor |
| :--- | :--- |
| **Nombre Completo** | `org.junit.platform.suite.engine.DiscoverySelectorResolver` |
| **Modificadores** | `final` |
| **Visibilidad** | package-private (sin modificador `public`) |
| **Superclase** | `java.lang.Object` |
| **Anotación Javadoc** | `@since 1.8` |

```java
/**
 * Resuelve los selectores de descubrimiento para el Suite Engine y orquesta
 * el descubrimiento de suites.
 * <p>
 * Configura un {@code EngineDiscoveryRequestResolver} estático para manejar
 * selectores de tipo ClassContainer y Class, y coordina el post-procesamiento
 * de los descriptores generados.
 * </p>
 *
 * @author JUnit Platform Team
 * @since 1.8
 */
final class DiscoverySelectorResolver { ... }
```

#### 2.2. Campos Estáticos

##### `resolver`

```java
private static final EngineDiscoveryRequestResolver<SuiteEngineDescriptor> resolver
```

Campo estático, inmutable y privado que contiene la instancia configurada del resolver de descubrimiento. Se inicializa mediante el patrón Builder.

**Configuración del Builder:**

| Resolver Registrado | Implementación (Lambda) | Descripción |
| :--- | :--- | :--- |
| `addClassContainerSelectorResolverWithContext` | `context -> new IsSuiteClass(context.getIssueReporter())` | Predicado que determina si una clase contenedora es una clase Suite válida. |
| `addSelectorResolver` | `context -> new ClassSelectorResolver(...)` | Resolver que procesa `ClassSelector` utilizando filtros, parámetros de configuración, creador de directorio de salida y listener de descubrimiento. |

> **Nota:** Las dos lambdas son implementaciones anónimas de interfaces funcionales provistas por `EngineDiscoveryRequestResolver`. No son métodos de `DiscoverySelectorResolver` pero constituyen lógica interna crítica delegada.

#### 2.3. Constructores

##### `DiscoverySelectorResolver()`

Constructor por defecto implícito generado por el compilador al no declararse ningún constructor explícito.

```java
/**
 * Constructor por defecto.
 * <p>
 * Al no declararse explícitamente, el compilador genera un constructor
 * sin argumentos con visibilidad package-private, consistente con la
 * visibilidad de la clase.
 * </p>
 *
 * @author JUnit Platform Team
 * @since 1.8
 */
DiscoverySelectorResolver()
```

| Característica | Detalle |
| :--- | :--- |
| **Visibilidad** | package-private |
| **Parámetros** | Ninguno |
| **Excepciones** | Ninguna (`@throws` no aplica) |

#### 2.4. Métodos

##### `discoverSuites()`

```java
/**
 * Itera sobre todos los hijos del descriptor del engine que son instancias
 * de {@code SuiteTestDescriptor} e invoca su lógica de descubrimiento individual.
 * <p>
 * Este método es la segunda fase del descubrimiento, ejecutada después de
 * que el {@code resolver} ha poblado el {@code engineDescriptor} con los
 * descriptores iniciales.
 * </p>
 *
 * @param engineDescriptor el descriptor raíz del Suite Engine que contiene
 *                         los descriptores de suite descubiertos como hijos
 * @throws ClassCastException si algún hijo del engineDescriptor no es una
 *                            instancia de {@code SuiteTestDescriptor}
 * @author JUnit Platform Team
 * @since 1.8
 */
private static void discoverSuites(SuiteEngineDescriptor engineDescriptor)
```

**Especificación Javadoc:**

| Tag | Detalle |
| :--- | :--- |
| `@param` | `engineDescriptor` |
| `@return` | `void` - No retorna valor |
| `@throws` | `ClassCastException` - Implícito por `SuiteTestDescriptor.class::cast` si la jerarquía es inconsistente. No declarado explícitamente en la firma. |

**Tabla de Parámetros:**

| Nombre | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `engineDescriptor` | `SuiteEngineDescriptor` | Sí | Descriptor raíz del engine. Se utiliza para obtener `getChildren()` y disparar `SuiteTestDescriptor::discover` en cada hijo. |

**Tabla de Retorno:**

| Tipo | Descripción |
| :--- | :--- |
| `void` | No produce valor de retorno. Opera por efecto lateral sobre los hijos del descriptor. |

**Implementación Interna:**
Utiliza Stream API:
```java
engineDescriptor.getChildren().stream()
    .map(SuiteTestDescriptor.class::cast) // Referencia a método anónima
    .forEach(SuiteTestDescriptor::discover); // Referencia a método anónima
```

##### `resolveSelectors()`

```java
/**
 * Resuelve los selectores de la petición de descubrimiento, dispara el
 * descubrimiento de suites y poda los descriptores vacíos.
 * <p>
 * Este es el único punto de entrada público (a nivel de paquete) de la clase.
 * Orquesta tres pasos secuenciales: 1) resolución de selectores, 2) descubrimiento
 * de suites y 3) poda del árbol de descriptores.
 * </p>
 *
 * @param request el requerimiento de descubrimiento de la plataforma que contiene
 *                selectores, filtros, parámetros de configuración y listeners
 * @param engineDescriptor el descriptor raíz del Suite Engine donde se registrarán
 *                         los descriptores descubiertos
 * @throws NullPointerException si {@code request} o {@code engineDescriptor} es {@code null}
 * @author JUnit Platform Team
 * @since 1.8
 */
void resolveSelectors(EngineDiscoveryRequest request, SuiteEngineDescriptor engineDescriptor)
```

**Especificación Javadoc:**

| Tag | Detalle |
| :--- | :--- |
| `@param` | `request` - Petición de descubrimiento |
| `@param` | `engineDescriptor` - Descriptor raíz |
| `@return` | `void` - No retorna valor |
| `@throws` | `NullPointerException` - Si algún parámetro es nulo al invocar `request.getDiscoveryListener()` o `engineDescriptor.getUniqueId()` |

**Tabla de Parámetros:**

| Nombre | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `request` | `EngineDiscoveryRequest` | Sí | Fuente de selectores, `ConfigurationParameters`, `OutputDirectoryCreator` y `DiscoveryListener`. |
| `engineDescriptor` | `SuiteEngineDescriptor` | Sí | Descriptor mutable que actúa como contenedor raíz. Se le añaden hijos durante `resolver.resolve()` y se poda al final. |

**Tabla de Retorno:**

| Tipo | Descripción |
| :--- | :--- |
| `void` | No produce valor de retorno. Modifica el estado de `engineDescriptor`. |

**Tabla de Excepciones:**

| Tipo | Condición |
| :--- | :--- |
| `NullPointerException` | Si `request` es `null` o si `engineDescriptor` es `null`. |
| Ninguna declarada | El método no declara `throws` explícito; los errores de descubrimiento se reportan vía `DiscoveryIssueReporter`. |

**Flujo Interno Detallado:**

1.  **Creación de `issueReporter`:**
    ```java
    DiscoveryIssueReporter issueReporter = DiscoveryIssueReporter.deduplicating(
        DiscoveryIssueReporter.forwarding(request.getDiscoveryListener(), engineDescriptor.getUniqueId()));
    ```
2.  **Resolución:** `resolver.resolve(request, engineDescriptor, issueReporter);`
3.  **Descubrimiento:** `discoverSuites(engineDescriptor);`
4.  **Poda:** `engineDescriptor.accept(TestDescriptor::prune);` // Referencia a método anónima

---

### 3. Ejemplo de Uso

La clase es `package-private` y está diseñada para ser utilizada exclusivamente por `SuiteTestEngine` dentro del mismo paquete. No está destinada a uso externo.

```java
package org.junit.platform.suite.engine;

import org.junit.platform.engine.EngineDiscoveryRequest;
import org.junit.platform.engine.ExecutionRequest;
import org.junit.platform.engine.TestDescriptor;
import org.junit.platform.engine.UniqueId;

/**
 * Ejemplo de uso real dentro del SuiteTestEngine.
 */
public class SuiteTestEngineExample {

    public TestDescriptor discover(EngineDiscoveryRequest discoveryRequest, UniqueId uniqueId) {
        // 1. Creación del descriptor raíz del engine
        SuiteEngineDescriptor engineDescriptor = new SuiteEngineDescriptor(uniqueId, "JUnit Platform Suite");

        // 2. Instanciación del resolver (visibilidad de paquete)
        DiscoverySelectorResolver selectorResolver = new DiscoverySelectorResolver();

        // 3. Resolución de selectores, descubrimiento y poda
        // Este método poblará engineDescriptor con los SuiteTestDescriptor
        // correspondientes a las clases anotadas con @Suite
        selectorResolver.resolveSelectors(discoveryRequest, engineDescriptor);

        // 4. El descriptor ya está listo para la fase de ejecución
        // Los nodos vacíos han sido podados con TestDescriptor::prune
        return engineDescriptor;
    }
}
```

**Caso de uso con `@Suite`:**

```java
import org.junit.platform.suite.api.SelectClasses;
import org.junit.platform.suite.api.Suite;

@Suite
@SelectClasses({ MiClaseDeTest.class, OtraClaseDeTest.class })
public class MiSuiteDePruebas {
    // Esta clase será detectada por IsSuiteClass y procesada
    // por ClassSelectorResolver durante resolveSelectors()
}
```

Cuando la plataforma lanza un `EngineDiscoveryRequest` que contiene un `ClassSelector` para `MiSuiteDePruebas`, `DiscoverySelectorResolver` lo resolverá, creará un `SuiteTestDescriptor` hijo y ejecutará `discover()` para expandir las clases seleccionadas por `@SelectClasses`.
