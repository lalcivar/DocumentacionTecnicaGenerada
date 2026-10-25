# Documentación Técnica: `LauncherSessionListener`

**Paquete:** `org.junit.platform.launcher`  
**Tipo:** `public interface`  
**Anotación:** `@API(status = STABLE, since = "1.10")`  
**Desde:** `1.8`  
**Licencia:** Eclipse Public License v2.0

---

### 1. Resumen Ejecutivo

`LauncherSessionListener` es una interfaz del JUnit Platform Launcher diseñada para implementar el patrón Observer sobre el ciclo de vida de una `LauncherSession`. Su propósito es permitir que componentes externos sean notificados de forma desacoplada cuando una sesión del launcher es abierta y cerrada, facilitando tareas de inicialización, limpieza de recursos, logging o integración con herramientas de terceros.

La interfaz puede ser registrada de forma programática mediante `LauncherConfig.Builder#addLauncherSessionListeners` al invocar `LauncherFactory#openSession(LauncherConfig)` o de forma automática mediante el mecanismo `java.util.ServiceLoader`.

### 2. Especificación de Métodos y Constructores

#### 2.1 Información General de la Interfaz

| Atributo | Valor |
| :--- | :--- |
| **Nombre Completo** | `org.junit.platform.launcher.LauncherSessionListener` |
| **Modificador** | `public interface` |
| **Anotaciones** | `@API(status = STABLE, since = "1.10")` |
| **Super-tipo** | Ninguno (interfaz base) |
| **Constructores** | No aplica. Al ser una interfaz, no declara constructores explícitos. El compilador no genera constructor. |
| **Métodos Totales Detectados** | 2 métodos de instancia `default` + 1 constante con clase anónima interna |

#### 2.2 Constantes y Campos

##### `NOOP`

Implementación No-Op (No Operation) de la interfaz. Se utiliza como objeto nulo para evitar comprobaciones de `null`.

```java
LauncherSessionListener NOOP = new LauncherSessionListener() {
};
```

| Atributo | Valor |
| :--- | :--- |
| **Firma** | `public static final LauncherSessionListener NOOP` |
| **Tipo** | `LauncherSessionListener` |
| **Modificador** | `public static final` (implícito en interfaces) |
| **Inicialización** | Clase anónima interna que implementa `LauncherSessionListener` sin sobreescribir ningún método, heredando las implementaciones `default` vacías. |
| **Propósito** | Proveer una instancia segura que no realiza ninguna acción al recibir eventos. |

> **Clase Anónima Interna asociada a `NOOP`:** Clase anónima generada por el compilador (`org.junit.platform.launcher.LauncherSessionListener$1`). No declara métodos propios. Hereda por defecto `launcherSessionOpened()` y `launcherSessionClosed()` con comportamiento vacío. No lanza excepciones.

#### 2.3 Detalle de Métodos

Se han detectado y documentado el **100%** de los métodos declarados en el código fuente.

##### 2.3.1 `launcherSessionOpened()`

Notifica al listener que una sesión del launcher ha sido abierta. Invocado por `Launcher` o `LauncherSession` creado vía `LauncherFactory`.

**Firma:**
```java
default void launcherSessionOpened(LauncherSession session)
```

**Documentación Javadoc:**
```java
/**
 * Called when a launcher session was opened.
 *
 * @param session the opened session
 */
```

**Especificación:**

| @param | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `session` | `LauncherSession` | Sí | La sesión que ha sido abierta. No debe ser `null`. Contiene el `Launcher` asociado y su configuración. |

| @return | Tipo | Descripción |
| :--- | :--- | :--- |
| `void` | `void` | No retorna valor. Método de notificación. |

| @throws | Condición |
| :--- | :--- |
| *Ninguna declarada* | La implementación por defecto es vacía y no lanza excepciones. Las implementaciones personalizadas no deben lanzar excepciones checked; cualquier excepción no capturada podría interrumpir el flujo del `LauncherFactory`. |

**Modificadores:** `public default`

##### 2.3.2 `launcherSessionClosed()`

Notifica al listener que una sesión del launcher ha sido cerrada. Invocado al finalizar el ciclo de vida de la sesión, ideal para liberar recursos.

**Firma:**
```java
default void launcherSessionClosed(LauncherSession session)
```

**Documentación Javadoc:**
```java
/**
 * Called when a launcher session was closed.
 *
 * @param session the closed session
 */
```

**Especificación:**

| @param | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `session` | `LauncherSession` | Sí | La sesión que ha sido cerrada. No debe ser `null`. |

| @return | Tipo | Descripción |
| :--- | :--- | :--- |
| `void` | `void` | No retorna valor. Método de notificación. |

| @throws | Condición |
| :--- | :--- |
| *Ninguna declarada* | La implementación por defecto es vacía y no lanza excepciones. |

**Modificadores:** `public default`

#### 2.4 Resumen Tabular de Métodos

| Método | Modificador | Parámetros | Retorno | @throws | Descripción |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `launcherSessionOpened()` | `public default` | `session: LauncherSession` | `void` | - | Invocado al abrir una `LauncherSession`. |
| `launcherSessionClosed()` | `public default` | `session: LauncherSession` | `void` | - | Invocado al cerrar una `LauncherSession`. |

---

### 3. Ejemplo de Uso

Caso de uso real: Implementación de un listener personalizado para medir la duración de la sesión y registrarlo programáticamente.

```java
package com.example.junit.listener;

import org.junit.platform.launcher.LauncherSession;
import org.junit.platform.launcher.LauncherSessionListener;
import org.junit.platform.launcher.core.LauncherConfig;
import org.junit.platform.launcher.core.LauncherFactory;
import org.junit.platform.launcher.core.LauncherDiscoveryRequestBuilder;
import static org.junit.platform.engine.discovery.DiscoverySelectors.selectClass;

// 1. Implementación del listener
public class TimingLauncherSessionListener implements LauncherSessionListener {

    private long startTime;

    @Override
    public void launcherSessionOpened(LauncherSession session) {
        startTime = System.currentTimeMillis();
        System.out.println("Sesión abierta: " + session.hashCode());
        // Inicializar recursos, p.ej., conexión a reporte
    }

    @Override
    public void launcherSessionClosed(LauncherSession session) {
        long duration = System.currentTimeMillis() - startTime;
        System.out.println("Sesión cerrada. Duración: " + duration + " ms");
        // Liberar recursos
    }
}

// 2. Registro y uso programático
public class LauncherExample {
    public static void main(String[] args) {
        LauncherConfig config = LauncherConfig.builder()
                .addLauncherSessionListeners(new TimingLauncherSessionListener())
                .enableTestEngineAutoRegistration(true)
                .enableTestExecutionListenerAutoRegistration(true)
                .build();

        try (LauncherSession session = LauncherFactory.openSession(config)) {
            var launcher = session.getLauncher();
            var request = LauncherDiscoveryRequestBuilder.request()
                    .selectors(selectClass(MyTestClass.class))
                    .build();
            launcher.execute(request);
        } // launcherSessionClosed() se invoca automáticamente al cerrar el try-with-resources
    }
}
```

**Registro alternativo vía ServiceLoader:**
Crear el archivo `META-INF/services/org.junit.platform.launcher.LauncherSessionListener` con el contenido:
```
com.example.junit.listener.TimingLauncherSessionListener
```
En este caso no es necesario el registro programático; JUnit lo descubrirá automáticamente.

