# Documentación Técnica – `LauncherSessionListener`

## 1. Resumen Ejecutivo
`LauncherSessionListener` es una interfaz de extensión dentro del paquete `org.junit.platform.launcher` diseñada para recibir notificaciones cuando un `LauncherSession` se abre o se cierra.  Permite a bibliotecas y herramientas externas engancharse al ciclo de vida de la sesión del *launcher* de JUnit Platform mediante implementación directa, registro programático a través de `LauncherConfig.Builder#addLauncherSessionListeners`, o descubrimiento automático con `ServiceLoader`.

---

## 2. Especificación de Constructores, Métodos y Clases Internas/Anónimas

### 2.1. Miembros estáticos

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `NOOP` | `LauncherSessionListener` | Implementación vacía (no-op) que puede reutilizarse cuando no se requiere lógica adicional. Implementada mediante una clase anónima interna sin miembros adicionales. |

> Nota: la clase anónima que respalda a `NOOP` posee únicamente el constructor implícito sin argumentos generado por el compilador (`<init>()`), por lo que no expone API pública adicional.

### 2.2. Métodos de instancia (todos `default`)

#### 2.2.1. `launcherSessionOpened(LauncherSession session)`

| @param | Tipo | Requerido | Descripción |
|--------|------|-----------|-------------|
| `session` | `LauncherSession` | Sí | Instancia recién abierta del *launcher session* que dispara la notificación. |

| @return | Descripción |
|---------|-------------|
| `void`  | No retorna valor; método puramente de notificación. |

| @throws | Condición |
|---------|-----------|
| — | El contrato no especifica excepciones; la implementación por defecto es *no-op*. |

#### 2.2.2. `launcherSessionClosed(LauncherSession session)`

| @param | Tipo | Requerido | Descripción |
|--------|------|-----------|-------------|
| `session` | `LauncherSession` | Sí | Instancia que acaba de cerrarse y que origina la notificación. |

| @return | Descripción |
|---------|-------------|
| `void`  | No retorna valor; método de notificación. |

| @throws | Condición |
|---------|-----------|
| — | El contrato no especifica excepciones; la implementación por defecto es *no-op*. |

---

## 3. Ejemplo de uso

```java
import org.junit.platform.launcher.*;
import org.junit.platform.launcher.core.LauncherConfig;
import org.junit.platform.launcher.core.LauncherFactory;

public class LauncherSessionDemo {

    // Implementación concreta del listener
    static class LoggingSessionListener implements LauncherSessionListener {

        @Override
        public void launcherSessionOpened(LauncherSession session) {
            System.out.println("Launcher session OPENED: " + session);
        }

        @Override
        public void launcherSessionClosed(LauncherSession session) {
            System.out.println("Launcher session CLOSED: " + session);
        }
    }

    public static void main(String[] args) {
        LauncherConfig config = LauncherConfig.builder()
                .addLauncherSessionListeners(new LoggingSessionListener())
                .build();

        try (LauncherSession session = LauncherFactory.openSession(config)) {
            Launcher launcher = session.getLauncher();

            // Aquí se podrían registrar test plans, ejecutar tests, etc.
            // Por brevedad el ejemplo se limita a abrir y cerrar la sesión.
        }
    }
}
```

En este escenario:
1. Se crea un `LauncherConfig` que registra una instancia de `LoggingSessionListener`.
2. Se abre una sesión mediante `LauncherFactory.openSession(config)`. Al abrirse, se invoca `launcherSessionOpened(...)`.
3. Al cerrar el bloque *try-with-resources*, se llama automáticamente a `launcherSessionClosed(...)`.

Con este patrón se obtiene un punto único y coherente para instrumentar o auditar el ciclo de vida del *launcher* de JUnit Platform.