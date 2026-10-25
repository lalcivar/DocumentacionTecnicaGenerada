Documentación técnica  
Interface: org.junit.platform.launcher.LauncherSessionListener  
Versión incorporada: 1.8 (API estable a partir de 1.10)  

1. Propósito general  
LauncherSessionListener define el contrato para recibir notificaciones sobre el ciclo de vida de una LauncherSession, es decir, los momentos en que la sesión de lanzamiento de pruebas se abre y se cierra.  
Su objetivo principal es permitir que componentes externos (p. ej. herramientas de IDE, sistemas de CI, extensiones de logging o métricas) reaccionen a dichos eventos sin acoplarse directamente a la implementación interna del launcher de JUnit Platform.

2. Registro del listener  
Existen dos formas de dar de alta un listener:  
• Programáticamente:  
  LauncherConfig config = LauncherConfig.builder()  
      .addLauncherSessionListeners(new MiListener())  
      .build();  
  try (LauncherSession session = LauncherFactory.openSession(config)) { … }  

• Automáticamente mediante java.util.ServiceLoader:  
  Crear el archivo META-INF/services/org.junit.platform.launcher.LauncherSessionListener que contenga la(s) implementación(es) concretas.

3. Flujo de eventos típico  
a) LauncherFactory.openSession(...) crea la sesión y llama, en el orden de registro, a launcherSessionOpened(session).  
b) Cuando la sesión finaliza (cierre implícito al salir del try-with-resources o explícito vía close()), se dispara launcherSessionClosed(session).  

Ambos métodos se ejecutan en el hilo que gestiona la sesión; si realiza operaciones bloqueantes o costosas es responsabilidad del implementador delegarlas a otro hilo.

4. Principales miembros

4.1 Constantes  
• NOOP: Instancia singleton “vacía” que implementa la interfaz sin añadir lógica. Útil como objeto por defecto cuando no se requiere procesamiento adicional.

4.2 Métodos

a) default void launcherSessionOpened(LauncherSession session)  
   – Descripción: Notificado inmediatamente después de crearse y ponerse activa una LauncherSession.  
   – Parámetro:  
     • session – instancia recién abierta. Proporciona acceso a un Launcher configurado y a su Store (para datos de usuario).  
   – Implementación por defecto: no hace nada. Sobrescribir para:  
     • Registrar listeners de TestExecution, Discovery, etc.  
     • Inicializar recursos (p. ej. starting log, contadores, telemetría).  

b) default void launcherSessionClosed(LauncherSession session)  
   – Descripción: Invocado justo antes de que la sesión sea cerrada y liberados sus recursos.  
   – Parámetro:  
     • session – la sesión que está a punto de cerrarse.  
   – Implementación por defecto: no hace nada. Sobrescribir para:  
     • Liberar recursos, vaciar buffers, detener hilos auxiliares.  
     • Generar reportes o métricas finales.

5. Extensión e implementación  
• La interfaz utiliza métodos default vacíos para evitar obligar al usuario a implementar ambos callbacks; basta con sobrescribir los que interesen.  
• Se recomienda que las implementaciones sean thread-safe si comparten estado con otros listeners o con código de pruebas.  
• No arrojar excepciones no comprobadas desde las callbacks; un fallo podría propagarse y abortar la ejecución de pruebas.  

6. Ejemplo mínimo

public class LoggingSessionListener implements LauncherSessionListener {
    @Override
    public void launcherSessionOpened(LauncherSession session) {
        System.out.println("Launcher session iniciada: " + session);
    }
    @Override
    public void launcherSessionClosed(LauncherSession session) {
        System.out.println("Launcher session finalizada: " + session);
    }
}

Registrado por ServiceLoader, cualquier ejecución de pruebas sobre JUnit Platform mostrará las trazas anteriores al inicio y cierre de la sesión.

7. Notas de compatibilidad  
• Anotado con @API(status = STABLE, since = "1.10"): la firma se considera estable, pero podrían añadirse nuevos métodos por defecto en versiones futuras.  
• Disponible desde JUnit Platform 1.8; verificar dependencia junit-platform-launcher ≥ 1.8.0.