# Documentación Técnica - Clase `DiscoverySelectorResolver`

### 1. Información General

| Atributo | Detalle |
| :--- | :--- |
| **Nombre de la Clase** | `DiscoverySelectorResolver` |
| **Paquete** | `org.junit.platform.suite.engine` |
| **Tipo** | Clase final, de visibilidad de paquete |
| **Versión** | `@since 1.8` |
| **Licencia** | Eclipse Public License v2.0 |
| **Módulo** | JUnit Platform Suite Engine |

#### 1.1 Descripción General

`DiscoverySelectorResolver` es una clase interna y de infraestructura del **JUnit Platform Suite Engine**. Su responsabilidad principal es **resolver y procesar las solicitudes de descubrimiento de tests** `EngineDiscoveryRequest` para el motor de suites.

Actúa como un orquestador que delega la lógica de descubrimiento a un `EngineDiscoveryRequestResolver` configurado, descubre las suites anidadas y finalmente poda el árbol de descriptores de test para eliminar nodos vacíos o no válidos.

Esta clase no está diseñada para ser usada directamente por el usuario final, sino que es utilizada internamente por el `SuiteTestEngine` para construir el `TestDescriptor` jerárquico que representa las suites de pruebas.

---

### 2. Dependencias Principales

*   `SuiteEngineDescriptor`: Descriptor raíz del motor de suites. Representa el engine completo.
*   `SuiteTestDescriptor`: Descriptor que representa una clase anotada con `@Suite`.
*   `EngineDiscoveryRequest`: Contiene los selectores y filtros enviados por el launcher de JUnit.
*   `EngineDiscoveryRequestResolver`: API de JUnit Platform para resolver selectores de forma declarativa.
*   `DiscoveryIssueReporter`: Mecanismo para reportar problemas durante el descubrimiento de forma deduplicada.
*   `IsSuiteClass`: Predicado que verifica si una clase es una Suite válida.
*   `ClassSelectorResolver`: Resolver específico para selectores de tipo `ClassSelector`.

### 3. Atributos

#### `private static final EngineDiscoveryRequestResolver<SuiteEngineDescriptor> resolver`

Resolver estático, inmutable y thread-safe que se construye una sola vez al cargar la clase mediante el patrón Builder.

Se configura con dos estrategias de resolución:

1.  `addClassContainerSelectorResolverWithContext(...)`: Registra un resolver para selectores de tipo contenedor de clase. Utiliza el predicado `IsSuiteClass` para filtrar y aceptar únicamente aquellas clases que son suites válidas. Recibe el `DiscoveryIssueReporter` del contexto.
2.  `addSelectorResolver(...)`: Registra un `ClassSelectorResolver` para selectores de tipo `ClassSelector`. Este resolver se encarga de crear los `SuiteTestDescriptor` correspondientes, utilizando el filtro de nombres de clase, los parámetros de configuración, el creador de directorio de salida y el listener de descubrimiento.

> Este atributo centraliza toda la lógica de mapeo entre los selectores solicitados y los descriptores del engine.

### 4. Métodos Principales

#### `void resolveSelectors(EngineDiscoveryRequest request, SuiteEngineDescriptor engineDescriptor)`

**Visibilidad:** de paquete  
**Descripción:** Es el método principal y punto de entrada de la clase. Orquesta todo el proceso de descubrimiento de suites.

**Flujo de ejecución:**
1.  Crea un `DiscoveryIssueReporter` deduplicado que reenvía los problemas al `DiscoveryListener` de la solicitud, asociado al `UniqueId` del engine.
2.  Invoca a `resolver.resolve(request, engineDescriptor, issueReporter)` para que los resolvers registrados procesen los selectores y pueblen el `engineDescriptor` con los hijos correspondientes.
3.  Llama a `discoverSuites(engineDescriptor)` para realizar el descubrimiento de segundo nivel dentro de cada suite encontrada.
4.  Ejecuta `engineDescriptor.accept(TestDescriptor::prune)` para podar el árbol y eliminar descriptores que quedaron vacíos, sin hijos o que no contienen tests ejecutables.

**Parámetros:**
*   `request`: La solicitud de descubrimiento que contiene selectores, filtros y configuración.
*   `engineDescriptor`: El descriptor raíz del engine donde se agregarán los descriptores descubiertos.

#### `private static void discoverSuites(SuiteEngineDescriptor engineDescriptor)`

**Visibilidad:** `private static`  
**Descripción:** Método auxiliar encargado del descubrimiento anidado.

Itera sobre todos los hijos del `engineDescriptor`, los convierte a `SuiteTestDescriptor` mediante `SuiteTestDescriptor.class::cast` y para cada uno invoca su método `discover()`. Este método permite que cada suite resuelva sus propias clases y métodos contenidos según las anotaciones `@SelectClasses`, `@SelectPackages`, etc.

Utiliza la API de Streams de Java 8 para realizar la operación de forma declarativa.

### 5. Flujo de Trabajo Resumido

```
EngineDiscoveryRequest
        |
        v
resolveSelectors() -> Crea DiscoveryIssueReporter
        |
        v
resolver.resolve() -> [IsSuiteClass + ClassSelectorResolver] -> Pobla SuiteEngineDescriptor
        |
        v
discoverSuites() -> Para cada SuiteTestDescriptor.discover()
        |
        v
TestDescriptor.prune() -> Elimina nodos vacíos del árbol
```

### 6. Consideraciones Técnicas

*   **Clase Final y No Extensible:** Al ser `final` y con visibilidad de paquete, no puede ser heredada ni accedida fuera de `org.junit.platform.suite.engine`.
*   **Inmutabilidad:** El `resolver` es `static final`, lo que garantiza que su configuración es inmutable y segura para uso concurrente.
*   **Manejo de Errores:** No lanza excepciones directamente; los problemas de descubrimiento se reportan de forma centralizada a través del `DiscoveryIssueReporter` deduplicado para evitar reportes duplicados.
*   **Poda del Árbol:** La llamada final a `prune` es crítica para que el motor no reporte suites vacías como tests ejecutables, optimizando la fase de ejecución.

