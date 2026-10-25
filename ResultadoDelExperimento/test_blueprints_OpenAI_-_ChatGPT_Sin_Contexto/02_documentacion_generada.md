============================================================
Documentación técnica  
Archivo: tests/test_blueprints.py   (nombre sugerido)
============================================================

1. Objetivo general
-------------------
El archivo contiene una batería de pruebas automatizadas (pytest) que validan el
comportamiento de *Blueprints* en Flask.  Cada prueba cubre una faceta
distinta: manejo de errores, construcción de URLs, ciclo de vida de la
petición, plantillas Jinja, archivos estáticos, anidación de blueprints,
subdominios y restricciones sobre nombres o registros.  
El propósito es garantizar que la implementación de Flask (o una extensión
propia) respete las reglas oficiales y los edge-cases que los usuarios suelen
encontrar al trabajar con blueprints.

2. Dependencias
---------------
• Python ≥ 3.8  
• pytest  
• Flask (incluye Werkzeug y Jinja2)  

Algunas pruebas importan «blueprintapp», un paquete de ejemplo que contiene dos
blueprints (frontend y admin) con sus plantillas y carpeta estática.

3. Estructura de alto nivel
---------------------------
Las pruebas se agrupan de manera implícita por tema:

A. Manejo de errores
   - test_blueprint_specific_error_handling  
   - test_blueprint_specific_user_error_handling  
   - test_blueprint_app_error_handling  

B. Enrutamiento y prefijos
   - test_blueprint_prefix_slash   (parámetro @pytest.mark.parametrize)  
   - test_blueprint_url_defaults  
   - test_blueprint_url_processors  
   - test_blueprint_app_url_processors  

C. Plantillas, filtros y estáticos
   - test_templates_and_static  
   - test_default_static_max_age  
   - test_templates_list  
   - test_template_filter* (8 variantes)  
   - test_template_test*   (8 variantes)  
   - test_template_global  

D. Contextos y ciclo de petición
   - test_context_processing  
   - test_request_processing  
   - test_app_request_processing  

E. Blueprints anidados y subdominios
   - test_nested_blueprint  
   - test_nested_callback_order  
   - test_nesting_url_prefixes (parametrizado)  
   - test_nesting_subdomains  
   - test_child_and_parent_subdomain  

F. Validaciones y renombrado
   - test_dotted_name_not_allowed  
   - test_empty_name_not_allowed  
   - test_dotted_names_from_app  
   - test_unique_blueprint_names  
   - test_self_registration  
   - test_blueprint_renaming  

4. Descripción de los componentes principales
---------------------------------------------

4.1 Manejo de errores
    • Se crean blueprints independientes con manejadores de error 403
      específicos; se asegura que cada blueprint atienda su propia excepción y
      que, de no existir, la aplicación principal asuma el control.  
    • Se prueban manejadores para excepciones de usuario registradas mediante
      el decorador @errorhandler o blue.register_error_handler().  
    • Se valida @errors.app_errorhandler para registrar un manejador “global”
      desde un blueprint.

4.2 Prefijos, reglas de ruta y procesadores de URL
    • Combinaciones de url_prefix y reglas (‘/’, ‘’, ‘//bar’, …) demuestran
      cómo Flask normaliza las barras.  
    • url_defaults/url_value_preprocessor a nivel blueprint y a nivel app
      garantizan que los parámetros (p.e. <lang_code>) se inyecten o extraigan
      correctamente.  
    • Se muestra la posibilidad de pasar url_prefix y url_defaults distintos en
      varios registros del mismo blueprint.

4.3 Plantillas y archivos estáticos
    • blueprintapp.frontend y blueprintapp.admin sirven como ejemplo real:
      plantillas “index.html” y archivos en “static/”.  
    • Se comprueba la propagación de la configuración SEND_FILE_MAX_AGE_DEFAULT
      y la sobre-escritura mediante una subclase de Blueprint
      (get_send_file_max_age).  
    • list_templates() confirma que Jinja2 combina los directorios “templates”
      de todos los blueprints registrados.

4.4 Filtros, tests y globales de Jinja2
    • Cada tipo (@app_template_filter, add_app_template_filter, etc.) se prueba
      con y sin nombre explícito, y antes o después de que exista una view que
      renderice la plantilla.  
    • Se verifican tanto la disponibilidad en app.jinja_env como la ejecución
      correcta al renderizar.

4.5 Context processors
    • @app_context_processor (scope global) versus .context_processor (scope
      blueprint) demuestran la precedencia de variables para distintas rutas.

4.6 Hooks de petición
    • before_request, after_request, teardown_request y sus equivalentes
      “before_app_request”, etc. se usan para registrar eventos y modificar la
      respuesta.  
    • Se confirma el orden de evaluación en ambientes anidados (app → parent →
      child).

4.7 Blueprints anidados y subdominios
    • register_blueprint permite anidar arbitrariamente (padre/hijo/nieto) y
      concatenar url_prefix.  
    • Se cubren casos con SERVER_NAME y subdomain_matching para que las rutas
      funcionen bajo api.example.test, etc.

4.8 Restricciones sobre nombres y registro
    • Nombres con “.” o vacíos levantan ValueError.  
    • El registro duplicado del mismo nombre o la auto-referencia están
      prohibidos.  
    • register_blueprint acepta el parámetro name para clonar un blueprint con
      un endpoint namespace distinto.

5. Parámetros y datos relevantes
--------------------------------
• Fixture «app» – instancia de flask.Flask creada en conftest.py
  (no mostrada).  
• Fixture «client» – instancia de app.test_client().  
• Fixture «test_apps» – importa blueprintapp y expone su app.  

6. Ejecución de las pruebas
---------------------------
Dentro de un entorno virtual con Flask y pytest instalados:

    pytest tests/test_blueprints.py

Opcionalmente se puede ejecutar sólo un subconjunto, por ejemplo:

    pytest -k "error_handling and not nested"

7. Puntos de extensión o referencia
-----------------------------------
• Cómo crear subclases de Blueprint para modificar comportamiento estático.  
• Ejemplo completo de arquitectura modular (frontend, admin, api) mediante la
  anidación de blueprints.  
• Demostraciones de todas las APIs públicas de Blueprint:  
  - route(), add_url_rule(), errorhandler(), register_error_handler()  
  - endpoint(), app_*(context|template|url)_(processor|filter|test|global)  
  - before/after/teardown (_request / _app_request)

8. Conclusión
-------------
Este archivo sirve tanto como batería de regresión para el núcleo de Flask
como guía práctica de las capacidades avanzadas de Blueprint.  Cada prueba
es, en esencia, un ejemplo mínimo y reproducible que los desarrolladores
pueden reutilizar para entender o depurar las interacciones entre módulos en
aplicaciones Flask de mediana y gran escala.