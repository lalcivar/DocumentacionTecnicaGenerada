DOCUMENTACIÓN TÉCNICA  
Módulo: flask.app (fragmento simplificado)  
Objeto principal: clase Flask

--------------------------------------------------------------------
1. Propósito general del archivo
--------------------------------------------------------------------
El archivo define la implementación completa del objeto ​Flask,​ la clase que convierte a un proyecto en una aplicación WSGI compatible con el micro-framework Flask.  
Sus responsabilidades principales son:

• Mantener el registro central de configuración, rutas, filtros de plantilla, señales, sesiones, etc.  
• Gestionar el ciclo de vida de las peticiones HTTP de principio a fin.  
• Exponer utilidades de servidor, CLI, pruebas y contexto para facilitar el desarrollo.  
• Proveer puntos de extensión para que terceros integren nuevas capacidades o reemplacen componentes internos.

--------------------------------------------------------------------
2. Componentes principales
--------------------------------------------------------------------

2.1  Importaciones
    – Funciones estándar (os, sys, datetime, inspect…), tipado (typing) y utilidades de compatibilidad.  
    – Werkzeug: wrappers, routing, excepciones, servidor de desarrollo.  
    – Paquetes internos de Flask (ctx, signals, helpers, sessions, wrappers, cli, etc.).

2.2  Tipos genéricos y ayudas internas
    • _make_timedelta(): Normaliza enteros o timedelta.  
    • remove_ctx() / add_ctx(): Wrappers retro-compatibles que agregan o quitan el nuevo parámetro ctx en métodos sobreescritos por subclasses.  
    • Varios TypeVar usados para reforzar type-checking en callbacks de usuario.

2.3  Clase Flask (hereda de sansio.app.App)
    a. Atributos y configuración
       – default_config: diccionario inmutable con más de 30 claves (DEBUG, SECRET_KEY, SESSION_COOKIE_* …).  
       – request_class, response_class: wrappers personalizados.  
       – session_interface: por omisión SecureCookieSessionInterface.

    b. Fase de construcción (__init__)  
       – Recibe parámetros para “descubrir” recursos (import_name, root_path), carpeta de estáticos, plantillas, instance_path, etc.  
       – Crea el grupo CLI (self.cli).  
       – Si hay carpeta estática, registra la ruta /static y su view send_static_file.

    c. Gestión de recursos (open_resource, open_instance_resource).  
       Facilitan abrir archivos relativos al root_path o instance_path respetando modos binarios / texto.

    d. Motor de plantillas
       – create_jinja_environment(): construye un Environment pre-configurado con filtros y globals propios de Flask.  
       – update_template_context(): ejecuta los context-processors registrados.

    e. Adaptador de URL
       – create_url_adapter(): instancia MapAdapter según la request o, fuera de ella, con SERVER_NAME.

    f. Ciclo de vida de la petición
       1. __call__ → wsgi_app(): punto de entrada WSGI.  
       2. request_context(): crea AppContext + Request.  
       3. push() de contexto.  
       4. full_dispatch_request():  
          • preprocess_request() → before_request  
          • dispatch_request() → routing y vista  
          • handle_*_exception() si hubo error  
          • finalize_request()  
       5. process_response() → after_request, sesión.  
       6. do_teardown_request() y do_teardown_appcontext() → teardown_* y señales.  
       7. pop() de contexto.

    g. Gestión de errores
       – handle_http_exception(), handle_user_exception(), handle_exception():  
         resuelven HTTPException, excepciones de usuario o no atrapadas; consultan los manejadores registrados y emiten señales.  
       – log_exception(): log a .logger con traceback.

    h. Generación de respuestas
       – make_response(): convierte el retorno de la vista a Response; soporta str, bytes, dict / list (→ JSON), tuple (body, status, headers), generadores, WSGI callables.

    i. Soporte asíncrono
       – ensure_sync(): detecta si la vista es async y la envuelve con asgiref.async_to_sync.  
       – async_to_sync(): punto de extensión si se quiere cambiar la estrategia.

    j. Utilidades de URL
       – url_for(): versión de método que implementa flask.url_for. Maneja _external, _scheme, _anchor, valores variables y errores BuildError.

    k. Hooks de pruebas
       – test_client(), test_cli_runner(), test_request_context(): construyen clientes y contextos simulados.

    l. Servidor de desarrollo
       – run(): envoltura sobre werkzeug.serving.run_simple con soporte para .env, FLASK_DEBUG, reloader, banner, etc. Bloquea el thread excepto si se invoca desde “flask run”.

--------------------------------------------------------------------
3. Flujo de extensión típico
--------------------------------------------------------------------
• Routing: @app.route() registra funciones en self.view_functions; url_map se actualiza.  
• Middleware: reasignar app.wsgi_app.  
• Señales: conectar a request_started, got_request_exception, etc.  
• Callbacks: before_request, after_request, teardown_request, teardown_appcontext, shell_context_processor.  
• Error handlers: @app.errorhandler(404) o números HTTP / clases de excepción.  
• CLI: @app.cli.command().  
• Sesiones: reemplazar session_interface.  
• Async: sobreescribir async_to_sync() para usar otro event-loop.

--------------------------------------------------------------------
4. Relación con otras piezas del framework
--------------------------------------------------------------------
El archivo presupone la existencia de:

• Modulo sansio.app.App: capa mínima sin dependencias WSGI usada como base.  
• Contenedores de contexto (AppContext, RequestContext) y proxies (request, session, g, current_app).  
• helpers, signals, templating y wrappers que complementan la funcionalidad.

--------------------------------------------------------------------
5. Resumen de señales emitidas
--------------------------------------------------------------------
request_started           – antes de procesar la petición.  
got_request_exception     – cuando se captura una excepción.  
request_finished          – después de enviar la respuesta.  
request_tearing_down      – al iniciar teardown_request.  
appcontext_tearing_down   – al iniciar teardown_appcontext.

--------------------------------------------------------------------
6. Consideraciones de compatibilidad y deprecaciones
--------------------------------------------------------------------
• remove_ctx / add_ctx permiten que subclases que no actualicen firmas con el nuevo parámetro ctx sigan funcionando hasta Flask 4.0 (se emite DeprecationWarning).  
• should_ignore_error() está deprecado (aviso en full_dispatch_request).  
• SEND_FILE_MAX_AGE_DEFAULT cambió de 12 h a None desde Flask 2.0.

--------------------------------------------------------------------
7. Puntos clave de seguridad
--------------------------------------------------------------------
• SECRET_KEY y SECRET_KEY_FALLBACKS son necesarios para firmar cookies y otros tokens.  
• TRAP_BAD_REQUEST_ERRORS y PROPAGATE_EXCEPTIONS controlan filtrado de errores en producción.  
• TRUSTED_HOSTS evita Host header injection.  
• SESSION_COOKIE_SECURE, HTTPONLY, SAMESITE y PARTITIONED endurecen cookies.

--------------------------------------------------------------------
8. Conclusión
--------------------------------------------------------------------
Este módulo constituye el núcleo operativo del micro-framework Flask. Define la clase central «Flask», inicializa configuración por defecto, implementa el ciclo WSGI completo, la gestión de contexto, plantillas, rutas, sesiones, CLI, pruebas, ejecución del servidor y los mecanismos de extensión necesarios para que desarrolladores y extensiones alteren o amplíen su comportamiento.