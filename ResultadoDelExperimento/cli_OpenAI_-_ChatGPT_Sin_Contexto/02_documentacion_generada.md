========================================================
Documentación técnica – Utilidades de línea de comandos Flask
========================================================

Visión general
--------------

El archivo implementa la mayor parte de la funcionalidad que el comando
“flask” ofrece desde la consola.  Proporciona un *CLI* (Command Line
Interface) extensible que permite:

* Localizar y cargar una instancia de aplicación Flask o una *app
  factory* especificada con **--app / FLASK_APP**.
* Cargar variables de entorno desde ficheros *.flaskenv* y *.env*.
* Exponer comandos de desarrollo integrados:
  - ``flask run`` : servidor de desarrollo.
  - ``flask shell`` : intérprete interactivo con contexto de app.
  - ``flask routes`` : listado de reglas de enrutamiento.
* Registrar comandos añadidos por extensiones o por la propia
  aplicación (``app.cli``).
* Mostrar información de versión y gestionar flags de *debug*.

Componentes principales
-----------------------

1. Manejo de errores
   • **NoAppException**  
     Excepción propia que hereda de *click.UsageError*.  Se lanza cuando
     no se puede localizar o instanciar la aplicación Flask solicitada.

2. Localización de la aplicación
   • **find_best_app(mod)**  
     Busca objetos ``Flask`` comunes: *app*, *application* o la única
     instancia presente en el módulo.  
   • **find_app_by_string(mod, expr)**  
     Interpreta cadenas del tipo ``"my_app"`` o ``"factory(arg=1)"``:
     si es función la llama, si es variable la devuelve. Usa *ast* para
     validar que sólo se emplean literales seguros.  
   • **prepare_import(path)**  
     Convierte una ruta de archivo Python en un import dotted-path y la
     añade a *sys.path*.  
   • **locate_app(mod_name, app_name)**  
     Importa el módulo, captura *ImportError* internas y delega en las
     funciones anteriores.  Opcionalmente devuelve *None* en vez de
     fallar.

3. Información compartida
   • **ScriptInfo**  
     Objeto que viaja dentro del contexto *click*; almacena
     ``app_import_path``, ``create_app`` y banderas como
     ``set_debug_flag`` o ``load_dotenv_defaults``.  
     Su método **load_app()** contiene la lógica definitiva para:
       - resolver la aplicación;
       - aplicar ``FLASK_DEBUG`` si procede;
       - cachear el resultado.

4. Decoradores y Grupos *click*
   • **with_appcontext**  
     Garantiza que la función se ejecute dentro de un *app context*.  
   • **AppGroup**  
     Sub-clase de *click.Group* que envuelve automáticamente los
     callbacks con *with_appcontext*.  
   • **FlaskGroup**  
     Grupo principal que implementa el comando “flask”.  Añade opciones
     globales (`--app`, `--debug`, `--env-file`, `--version`), carga
     comandos por *entry points* (“flask.commands”) y delega en la app
     para exponer sus propios comandos.

5. Tipos y validadores personalizados
   • **CertParamType**  
     Gestiona ``--cert`` admitiendo: ruta a fichero, ``adhoc`` o un
     objeto ``ssl.SSLContext`` importable.  
   • **SeparatedPathType**  
     Permite pasar listas separadas por el separador de rutas del SO.  
   • Callbacks auxiliares: ``_validate_key`` (relaciona --key con
     --cert), ``_set_app``, ``_set_debug``, etc.

6. Utilidades varias
   • **load_dotenv(path, load_defaults=True)**  
     Abstracción sobre *python-dotenv* para combinar *.flaskenv*,
     *.env* y un path explícito sin pisar variables ya definidas.  
   • **show_server_banner**  
     Muestra texto “*Serving Flask app ...*” sólo una vez (ignora
     recarga).

7. Comandos incorporados
   a) **run_command**  
      Implementa ``flask run``.  Usa *werkzeug.run_simple*, controla
      reloader, debugger, TLS, hilos, archivos extra, etc.  Si la app
      falla al cargar y estamos dentro del reloader, captura la
      excepción y expone un WSGI que la relanza para que el debugger la
      muestre.  

   b) **shell_command**  
      Lanza un intérprete interactivo (*code.interact*) con un espacio
      de nombres inicial proveniente de
      ``current_app.make_shell_context()`` e integra *readline* si
      existe.

   c) **routes_command**  
      Muestra tabla con endpoint, métodos, dominio/subdominio y regla.
      Permite ordenar con ``--sort`` y ocultar HEAD/OPTIONS.

8. Punto de entrada
   • **cli = FlaskGroup(...)**  
     Instancia del grupo con nombre “flask”.  
   • **main()**  
     Llama a ``cli.main()`` para delegar en *click*.  Si el archivo se
     ejecuta directamente, se dispara este *entry point*.

Flujo típico de ejecución
-------------------------

1. El usuario ejecuta `flask <cmd> [opciones]`.
2. *click* crea el contexto y construye **ScriptInfo**.
3. El callback de ``--env-file`` puede cargar variables antes de que se
   procese el resto.
4. Cuando algún comando requiere la aplicación (por ej. *run*, *shell*),
   invoca ``info.load_app()``:
     - resuelve o importa la app/factory indicada;
     - ajusta *debug* si corresponde;
     - devuelve la instancia y la cachea.
5. Commands decorados con *with_appcontext* aseguran que existe un app
   context activo.
6. Cada sub-comando realiza su lógica (levantar servidor, abrir shell,
   etc.).

Dependencias externas usadas
----------------------------

• click – construcción del CLI.  
• werkzeug – servidor de desarrollo y utilidades.  
• python-dotenv (opcional) – carga de ficheros *.env*.  
• cryptography (opcional) – certificados “adhoc” para TLS.  

Extensibilidad
--------------

Extensiones o proyectos pueden registrar comandos adicionales mediante
el *entry point* “flask.commands” o mediante ``app.cli.add_command``;
*FlaskGroup* los descubre en tiempo de ejecución.

Seguridad y buenas prácticas
----------------------------

El servidor que se arranca con ``flask run`` está pensado sólo para
desarrollo.  Para producción debe usarse un servidor WSGI robusto
(gunicorn, uWSGI, etc.).  Además, las funcionalidades de *debugger* y
*reloader* no deben activarse en entornos productivos.

Conclusión
----------

El módulo encapsula todo lo necesario para que “flask” sea un comando
autónomo, capaz de localizar la aplicación, exponer utilidades comunes
y facilitar la extensión tanto por la propia app como por terceros.