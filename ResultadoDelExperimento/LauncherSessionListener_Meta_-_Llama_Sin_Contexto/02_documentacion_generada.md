# Documentación Técnica - `LauncherSessionListener`

### 1. Información General

| Atributo | Detalle |
| :--- | :--- |
| **Nombre Completo** | `org.junit.platform.launcher.LauncherSessionListener` |
| **Tipo** | Interfaz |
| **Paquete** | `org.junit.platform.launcher` |
| **Plataforma** | JUnit Platform - Launcher API |
| **Estado de API** | `STABLE` desde `1.10` |
| **Disponible desde** | `1.8` |
| **Licencia** | Eclipse Public License v2.0 |
| **Dependencias** | `org.apiguardian:apiguardian-api`, `org.junit.platform.launcher.core.LauncherFactory`, `org.junit.platform.launcher.core.LauncherConfig` |

### 2. Descripción General

`LauncherSessionListener` es una interfaz del tipo *Listener* que forma parte de la API de JUnit Platform Launcher. Su propósito es permitir la intercepción y observación del **ciclo de vida de una sesión de ejecución** `LauncherSession`.

Actúa como un mecanismo de extensión y observabilidad. Permite ejecutar lógica personalizada en dos momentos clave: inmediatamente después de que una sesión es abierta y justo después de que es cerrada. Esto es fundamental para la gestión de recursos globales, inicialización de infraestructura de pruebas, logging, métricas o integración con herramientas externas.

La interfaz está diseñada bajo el principio de no intrusividad: todos sus métodos poseen implementaciones por defecto vacías, por lo que las clases que la implementen solo necesitan sobrescribir los eventos que les interesan.

### 3. Características Principales

*   **Patrón Observer / Listener:** Desacopla la lógica de monitoreo de la lógica de ejecución del Launcher.
*   **Implementación Opcional:** Gracias a los métodos `default`, no es obligatorio implementar todos los eventos.
*   **Doble Mecanismo de Registro:** Soporta registro programático y descubrimiento automático.
*   **API Estable:** Marcada con `@API(status = STABLE)`, garantiza compatibilidad a largo plazo.
*   **Invocada por el Framework:** Los eventos son disparados exclusivamente por las instancias de `Launcher` o `LauncherSession` creadas a través de `LauncherFactory`.

### 4. Miembros Principales

#### 4.1 Constante

##### `LauncherSessionListener NOOP`
Implementación *No-Operation* de la interfaz. Es una instancia anónima que no realiza ninguna acción en ninguno de los métodos. Se utiliza como objeto nulo para evitar comprobaciones de `null` y como valor por defecto seguro.

```java
LauncherSessionListener NOOP = new LauncherSessionListener() {};
```

#### 4.2 Métodos

| Método | Firma | Descripción |
| :--- | :--- | :--- |
| **launcherSessionOpened** | `default void launcherSessionOpened(LauncherSession session)` | Se invoca cuando una `LauncherSession` ha sido abierta exitosamente. Es el punto ideal para inicializar recursos, configurar contextos, iniciar temporizadores o registrar información de la sesión. Recibe como parámetro la sesión abierta. |
| **launcherSessionClosed** | `default void launcherSessionClosed(LauncherSession session)` | Se invoca cuando una `LauncherSession` ha sido cerrada. Es el punto ideal para liberar recursos, realizar limpieza, volcar reportes, cerrar conexiones o finalizar métricas. Recibe como parámetro la sesión cerrada. |

> **Nota:** Ambos métodos tienen una implementación por defecto vacía. No es necesario sobrescribir ambos si solo se requiere uno.

### 5. Mecanismos de Registro

Existen dos formas de registrar un listener para que sea notificado por el `LauncherFactory`:

**1. Registro Programático:**
A través del builder de `LauncherConfig` al momento de abrir la sesión. Es el método recomendado para configuración explícita y controlada.

```java
LauncherConfig config = LauncherConfig.builder()
    .addLauncherSessionListeners(new MiCustomListener())
    .build();

try (LauncherSession session = LauncherFactory.openSession(config)) {
    // ... ejecución de pruebas
}
```

**2. Registro Automático vía ServiceLoader:**
Mediante el mecanismo estándar de Java `ServiceLoader`. Permite el auto-descubrimiento sin modificar código. Para ello, se debe crear un archivo en:
`META-INF/services/org.junit.platform.launcher.LauncherSessionListener`
que contenga el nombre cualificado de la clase que implementa la interfaz.

### 6. Ejemplo de Implementación

```java
import org.junit.platform.launcher.LauncherSession;
import org.junit.platform.launcher.LauncherSessionListener;

public class LoggingSessionListener implements LauncherSessionListener {

    @Override
    public void launcherSessionOpened(LauncherSession session) {
        System.out.println("Sesión iniciada: " + session);
        // Inicializar recursos globales
    }

    @Override
    public void launcherSessionClosed(LauncherSession session) {
        System.out.println("Sesión finalizada: " + session);
        // Liberar recursos y generar reporte
    }
}
```

### 7. Ciclo de Vida

```
LauncherFactory.openSession(config) 
      |
      +--> launcherSessionOpened(session)  // Notifica a todos los listeners registrados
      |
      +--> [ Ejecución de Test Discovery y Execution ]
      |
      +--> session.close()
              |
              +--> launcherSessionClosed(session) // Notifica a todos los listeners registrados
```

### 8. Véase También

*   `org.junit.platform.launcher.LauncherSession`
*   `org.junit.platform.launcher.Launcher`
*   `org.junit.platform.launcher.core.LauncherFactory`
*   `org.junit.platform.launcher.core.LauncherConfig.Builder#addLauncherSessionListeners`