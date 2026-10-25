DOCUMENTACIÓN TÉCNICA  
Archivo: test_flask_core.py  (nombre sugerido)

1. Propósito general  
El archivo es una batería de pruebas automatizadas escrita con pytest que valida el comportamiento público de Flask frente a Werkzeug. Cubre prácticamente todo el ciclo de vida de una petición HTTP, la configuración de la aplicación, el enrutamiento, la gestión de sesiones, el sistema de mensajes flash, la generación de URLs, la entrega de ficheros estáticos y las utilidades de JSON, así como la integración con el servidor de desarrollo de Werkzeug.  

2. Dependencias principales  
• Librería estándar – gc, re, uuid, datetime, functools, weakref, contextlib, platform, typing.  
• Paquetes externos – pytest, flask, werkzeug, markupsafe.  
• El marcador require_cpython_gc omite una prueba si la VM no es CPython (por su política de GC).  

3. Fixtures y organización de pytest  
Se asume la existencia de los fixtures app, client, req_ctx y app_ctx (definidos en conftest.py u otro módulo).  
El archivo usa parametrize para explotar combinatorias de casos (métodos HTTP, configuraciones de debug, etc.) y monkeypatch cuando es necesario interceptar llamadas a werkzeug.serving.run_simple.

4. Estructura de las pruebas (componentes principales)

4.1 Enrutamiento y métodos HTTP  
   • test_options_* Verifica la generación automática y la desactivación explícita de la respuesta OPTIONS.  
   • test_method_route* Comprueba los helpers app.get, app.post, etc.  
   • test_url_mapping / test_werkzeug_routing / test_endpoint_decorator Validan la interacción con reglas manuales de Werkzeug, sub-mounts y el decorador @app.endpoint.  
   • test_multi_route_* Demuestra que un mismo view puede estar registrado bajo varias reglas.

4.2 Sesiones  
   • Pruebas de creación, modificación, expiración y flags “permanent”.  
   • test_session_secret_key_fallbacks Rotación de claves para permitir descifrar cookies antiguas.  
   • test_session_* Dominios, Application Root, atributos SameSite, tamaño máximo de la cookie, cabecera Vary y refresco condicional.  

4.3 Ciclo de petición  
   • before_request, after_request, after_this_request, teardown_request y su orden de ejecución.  
   • Simulación de errores en cada fase y verificación de los handlers 500 personalizados.

4.4 Gestión de errores  
   • test_error_handling* Maneja errores HTTP estándar y excepciones de usuario.  
   • test_trap_bad_request_key_error / test_trapping_of_all_http_exceptions Evalúan la configuración DEBUG y TRAP\_* para decidir si se propaga la excepción o se convierte en respuesta.  
   • test_errorhandler_precedence Comprueba la precedencia de handlers cuando hay herencia múltiple de excepciones.

4.5 Respuestas  
   • test_response_types / test_response_type_errors Aceptación de distintos tipos de retorno (str, bytes, tuple, dict, list, Response, WSGI) y mensajes de error cuando el tipo no es válido.  
   • test_make_response* Cobertura directa de flask.make_response.

4.6 JSON  
   • test_jsonify_* y test_json_dump_dataclass Validan compactación, mimetype personalizado y soporte para dataclasses.

4.7 Generación de URL  
   • test_url_generation, test_build_error_handler*, test_url_processors, inject_blueprint_url_defaults.  
   • Casos con subdominios, host matching, SERVER_NAME y rutas estáticas.

4.8 Ficheros estáticos  
   • test_static_* Distintas combinaciones de static_folder, static_url_path, host_matching y pathlib.Path.

4.9 Servidor de desarrollo  
   • test_run_* y test_werkzeug_passthrough_errors Comprueban que app.run delega en Werkzeug con los parámetros correctos (host, puerto, flags de depuración/reload y passthrough_errors).

4.10 Utilidades varias  
   • Flash messages (test_flashes, test_extended_flashing).  
   • LocalProxy g (test_get_method_on_g, test_g_iteration_protocol).  
   • Garbage-collection de la instancia Flask (test_app_freed_on_zero_refcount).  
   • Decodificación de rutas Unicode (test_nonascii_pathinfo).

5. Convenciones y buenas prácticas ilustradas  
• Uso de context managers del cliente de prueba para mantener cookies y contexto de aplicación.  
• Separación de concerns: cada test comprueba un comportamiento atómico.  
• Empleo intensivo de asserts para garantizar tanto estado interno (session.modified) como externo (códigos y cabeceras HTTP).  
• Parametrización sistemática para reducir redundancia y aumentar cobertura.  

6. Ejecución de la suite  
$ pytest test_flask_core.py  

Todas las pruebas son unitarias; no se requiere ningún servicio externo ni conexión a red. Algunas dependencias de versión se resuelven en tiempo de ejecución (ej. werkzeug ≥ 3.2).  

7. Posibles puntos de extensión  
• Añadir más combinaciones de SameSite (“None” con Secure) o nuevos tipos serializables en la sesión.  
• Incluir pruebas ASGI en caso de integrar Flask con wrappers asincrónicos.  
• Medir cobertura de logs (caplog) para asegurar que los warnings esperados realmente se emiten.

En resumen, este archivo funciona como especificación viviente de la API pública de Flask: cada test documenta, valida y protege contra regresiones futuras en las áreas clave del framework.