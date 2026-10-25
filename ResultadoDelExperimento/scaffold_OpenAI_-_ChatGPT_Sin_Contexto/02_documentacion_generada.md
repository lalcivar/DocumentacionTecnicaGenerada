Documentación técnica  
=====================  

Archivo: scaffold.py (nombre tentativo)  

1. Propósito general  
--------------------  
El archivo implementa la “capa de andamiaje” (scaffold) común que comparten los objetos `Flask` y `Blueprint` dentro del micro-framework Flask.  
Su misión principal es proporcionar:

• Mecanismos de registro de vistas (rutas), manipuladores de errores y “hooks” de ciclo de vida de la petición.  
• Soporte para archivos estáticos y plantillas Jinja.  
• Utilidades de descubrimiento de paquetes en disco (útiles para localizar recursos relativos al proyecto).  

El fichero no contiene lógica de servidor HTTP: sólo define la interfaz y la estructura de datos necesarias para que una aplicación o blueprint puedan declararse de forma uniforme.  

2. Dependencias externas  
------------------------  
• Werkzeug: enrutamiento y clases de excepción HTTP.  
• Jinja2: sistema de plantillas.  
• Click (solo para tipado).  
• Módulos estándar: `os`, `pathlib`, `importlib.util`, `functools`, `collections.defaultdict`, `typing`, `sys`.  

3. Componentes principales  
--------------------------  

3.1 Constantes y tipos genéricos  
• `_sentinel`: valor único que sirve como “missing” internamente (no se usa aún en este trozo de código).  
• Serie de `TypeVar` cuyo nombre refleja la familia de callback que acepta (`T_after_request`, `T_route`, etc.). De esta manera se mantiene información de tipos en los decoradores.  

3.2 Decorador `setupmethod`  
Permite marcar métodos que solo son válidos durante la fase de “configuración” (antes de que la aplicación se considere arrancada).  
– Internamente llama a `_check_setup_finished` (método abstracto que el descendiente concreto implementará) para impedir cambios a destiempo.  

3.3 Clase `Scaffold`  
Es una clase base abstracta (contiene varios `raise NotImplementedError`) a partir de la cual se construyen `Flask` y `Blueprint`.  

Atributos destacados creados en `__init__`:

• `import_name`, `root_path`: localización física del paquete.  
• Gestores de rutas y callbacks  
  – `view_functions` – endpoint → función.  
  – `error_handler_spec`, `before_request_funcs`, `after_request_funcs`, `teardown_request_funcs`,  
    `template_context_processors`, `url_value_preprocessors`, `url_default_functions`.  
  Todas son estructuras `defaultdict` que agrupan los handlers por ámbito (None o nombre de blueprint).  

• Soporte estático/plantillas  
  – `static_folder`, `static_url_path` con sus properties auxiliares (`has_static_folder`).  
  – `template_folder` y `jinja_loader` (propiedad con `cached_property` que crea un `FileSystemLoader`).  

Métodos públicos (decorados con `@setupmethod`)  
a) Registro de rutas  
   – `route()` genérico y atajos: `get`, `post`, `put`, `delete`, `patch`, `query`.  
   – `add_url_rule()` – implementación real (abstracta aquí).  
   – `endpoint()` – registra una función para un endpoint ya existente.  

b) Hooks de petición  
   – `before_request`, `after_request`, `teardown_request`.  

c) Procesadores de contexto / URL  
   – `context_processor`, `url_value_preprocessor`, `url_defaults`.  

d) Manejo de errores  
   – `errorhandler()` (decorador) y `register_error_handler()`.  
   – Método estático auxiliar `_get_exc_class_and_code()` normaliza si se recibe un código o clase de excepción.  

e) Métodos auxiliares  
   – `_method_route()` factoriza la construcción de decoradores “GET”, “POST”, etc.  
   – `__repr__` para depuración.  

Privados a implementar por la subclase:  
• `_check_setup_finished`, `add_url_rule`.  

3.4 Funciones de módulo  

a) `_endpoint_from_view_func(view_func)`  
Devuelve el nombre de la función, empleado como endpoint por omisión.  

b) `_find_package_path(import_name)`  
Resuelve la ruta física donde reside el paquete o módulo raíz.  
Considera tres casos:  
  – módulo “namespace package” (PEP-420).  
  – paquete tradicional con `__init__.py`.  
  – módulo simple.  
Si falla, retorna el directorio de trabajo (`os.getcwd()`).  

c) `find_package(import_name)`  
Envuelve a la anterior para, además, determinar si la ruta pertenece:  

  1. Al prefijo del intérprete (instalación de sistema o virtualenv).  
  2. A un `site-packages` de virtualenv.  
  3. Al proyecto local (no instalado).  

Devuelve `(prefix, package_path)` donde `prefix` puede ser `None`.  

4. Flujo de uso típico  
----------------------  
1. Una subclase (`Flask` o `Blueprint`) hereda de `Scaffold` e implementa `add_url_rule` y `_check_setup_finished`.  
2. Durante la fase de configuración:  
   – El desarrollador usa `@app.route`, `@app.before_request`, `@app.errorhandler`, etc.  
   – Cada decorador añade referencias en los `defaultdict` correspondientes.  
3. Cuando la app se “congela” (por ejemplo, cuando se llama `app.finalize()` en Flask), la subclase marca la configuración como cerrada. A partir de ahí, `setupmethod` bloqueará llamadas de mutación para garantizar inmutabilidad en producción.  
4. En tiempo de ejecución, la infraestructura de Flask recorre estas estructuras para:  
   • Resolver la URL hacia la vista correcta.  
   • Ejecutar los hooks en el orden apropiado.  
   • Encontrar handlers de error específicos o genéricos.  

5. Consideraciones de diseño  
----------------------------  
• Uso intensivo de tipado estático (`typing.TypeVar`, “bound”) que facilita a los IDEs la detección de errores.  
• Los diccionarios anidados almacenan información separada por “scope” (aplicación o blueprint) para favorecer la reutilización del código tanto en el núcleo de Flask como en sus blueprints.  
• Se evita importación de Click en tiempo de ejecución excepto para chequeo de tipos (`if TYPE_CHECKING`).  
• El descubrimiento de rutas de paquete está pensado para funcionar en distintos entornos (sistema, venv, namespace packages).  

6. Puntos de extensión / tareas pendientes  
------------------------------------------  
• Implementar la lógica real de `add_url_rule` (en Flask la proporciona la clase `Flask`).  
• Implementar `_check_setup_finished` para marcar la aplicación como “congelada”.  
• Exponer CLI (`cli: Group`) y otros componentes que aquí sólo se tipan.  

Con esta descripción, un desarrollador puede comprender qué responsabilidades cubre el módulo, cómo registra internamente cada tipo de callback y en qué lugar debe extenderlo para integrarse con el resto del framework.