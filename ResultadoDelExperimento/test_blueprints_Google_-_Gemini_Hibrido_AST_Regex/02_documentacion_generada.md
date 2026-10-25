Claro, aquí tienes la documentación técnica exhaustiva del código fuente proporcionado, siguiendo el estándar de Docstrings de Google y las especificaciones requeridas.

***

## 1. Resumen ejecutivo

Este módulo Python es una suite de pruebas exhaustiva para la clase `flask.Blueprint` del framework Flask. Utiliza el framework `pytest` para verificar sistemáticamente el comportamiento de los Blueprints, incluyendo el manejo de errores, el enrutamiento de URL, la gestión de plantillas y archivos estáticos, los procesadores de contexto y el anidamiento de Blueprints. El propósito es asegurar la modularidad, reusabilidad y correcta integración de los Blueprints dentro de una aplicación Flask.

## 2. Especificación de funciones y métodos

A continuación se detalla cada función de prueba y las funciones, métodos y clases anidadas que contiene.

### `test_blueprint_specific_error_handling(app, client)`

Verifica que los manejadores de errores registrados en un Blueprint son específicos para las rutas de ese Blueprint. Comprueba que diferentes Blueprints pueden tener manejadores distintos para el mismo código de error y que se utiliza un manejador global de la aplicación como fallback si un Blueprint no tiene uno específico.

**Parámetros de Entrada**

| Nombre   | Tipo                          | Requerido | Descripción                                                              |
| :------- | :---------------------------- | :-------- | :----------------------------------------------------------------------- |
| `app`    | `flask.Flask`                 | Sí        | Fixture de Pytest que proporciona una instancia de la aplicación Flask.  |
| `client` | `flask.testing.FlaskClient`   | Sí        | Fixture de Pytest que proporciona un cliente de prueba para la aplicación. |

**Valores de Retorno**

| Tipo     | Descripción                                         |
| :------- | :-------------------------------------------------- |
| `None`   | Esta función es una prueba y no retorna ningún valor. |

#### Funciones Anidadas

##### `frontend_forbidden(e)`

Manejador de errores para el código 403, específico del Blueprint `frontend`.

| Parámetro | Tipo                                  | Requerido | Descripción                                    |
| :-------- | :------------------------------------ | :-------- | :--------------------------------------------- |
| `e`       | `werkzeug.exceptions.HTTPException`   | Sí        | La excepción HTTP que activó el manejador.     |
| **Retorno** | `tuple[str, int]`                     | -         | Tupla con el mensaje de error y el código 403. |

##### `frontend_no()`

Ruta del Blueprint `frontend` que aborta la petición con un error 403.

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| **Raises** | `werkzeug.exceptions.Forbidden` | - | Siempre aborta la petición con un error 403. |

##### `backend_forbidden(e)`

Manejador de errores para el código 403, específico del Blueprint `backend`.

| Parámetro | Tipo                                  | Requerido | Descripción                                    |
| :-------- | :------------------------------------ | :-------- | :--------------------------------------------- |
| `e`       | `werkzeug.exceptions.HTTPException`   | Sí        | La excepción HTTP que activó el manejador.     |
| **Retorno** | `tuple[str, int]`                     | -         | Tupla con el mensaje de error y el código 403. |

##### `backend_no()`

Ruta del Blueprint `backend` que aborta la petición con un error 403.

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| **Raises** | `werkzeug.exceptions.Forbidden` | - | Siempre aborta la petición con un error 403. |

##### `sideend_no()`

Ruta del Blueprint `sideend` que aborta la petición con un error 403. Este Blueprint no tiene un manejador de errores propio.

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| **Raises** | `werkzeug.exceptions.Forbidden` | - | Siempre aborta la petición con un error 403. |

##### `app_forbidden(e)`

Manejador de errores global a nivel de aplicación para el código 403.

| Parámetro | Tipo                                  | Requerido | Descripción                                    |
| :-------- | :------------------------------------ | :-------- | :--------------------------------------------- |
| `e`       | `werkzeug.exceptions.HTTPException`   | Sí        | La excepción HTTP que activó el manejador.     |
| **Retorno** | `tuple[str, int]`                     | -         | Tupla con el mensaje de error y el código 403. |

---

### `test_blueprint_specific_user_error_handling(app, client)`

Valida que un Blueprint puede registrar y manejar excepciones personalizadas definidas por el usuario, tanto mediante el decorador `@errorhandler` como con el método `register_error_handler`.

**Parámetros de Entrada**

| Nombre   | Tipo                          | Requerido | Descripción                                                              |
| :------- | :---------------------------- | :-------- | :----------------------------------------------------------------------- |
| `app`    | `flask.Flask`                 | Sí        | Fixture de Pytest que proporciona una instancia de la aplicación Flask.  |
| `client` | `flask.testing.FlaskClient`   | Sí        | Fixture de Pytest que proporciona un cliente de prueba para la aplicación. |

**Valores de Retorno**

| Tipo     | Descripción                                         |
| :------- | :-------------------------------------------------- |
| `None`   | Esta función es una prueba y no retorna ningún valor. |

#### Funciones y Clases Anidadas

##### `class MyDecoratorException(Exception)`

Excepción personalizada para ser capturada por un manejador registrado con decorador.

##### `class MyFunctionException(Exception)`

Excepción personalizada para ser capturada por un manejador registrado con un método.

##### `my_decorator_exception_handler(e)`

Manejador de errores para la excepción `MyDecoratorException`.

| Parámetro | Tipo                    | Requerido | Descripción                                    |
| :-------- | :---------------------- | :-------- | :--------------------------------------------- |
| `e`       | `MyDecoratorException`  | Sí        | La instancia de la excepción capturada.        |
| **Retorno** | `str`                   | -         | El cuerpo de la respuesta.                     |

##### `my_function_exception_handler(e)`

Manejador de errores para la excepción `MyFunctionException`.

| Parámetro | Tipo                  | Requerido | Descripción                                    |
| :-------- | :-------------------- | :-------- | :--------------------------------------------- |
| `e`       | `MyFunctionException` | Sí        | La instancia de la excepción capturada.        |
| **Retorno** | `str`                 | -         | El cuerpo de la respuesta.                     |

##### `blue_deco_test()`

Ruta que lanza una `MyDecoratorException`.

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| **Raises** | `MyDecoratorException` | - | Siempre lanza esta excepción. |

##### `blue_func_test()`

Ruta que lanza una `MyFunctionException`.

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| **Raises** | `MyFunctionException` | - | Siempre lanza esta excepción. |

---

### `test_blueprint_app_error_handling(app, client)`

Verifica que un manejador de errores registrado en un Blueprint con `app_errorhandler` se aplica globalmente a toda la aplicación, incluyendo las rutas de la propia aplicación y las de otros Blueprints.

**Parámetros de Entrada**

| Nombre   | Tipo                          | Requerido | Descripción                                                              |
| :------- | :---------------------------- | :-------- | :----------------------------------------------------------------------- |
| `app`    | `flask.Flask`                 | Sí        | Fixture de Pytest que proporciona una instancia de la aplicación Flask.  |
| `client` | `flask.testing.FlaskClient`   | Sí        | Fixture de Pytest que proporciona un cliente de prueba para la aplicación. |

**Valores de Retorno**

| Tipo     | Descripción                                         |
| :------- | :-------------------------------------------------- |
| `None`   | Esta función es una prueba y no retorna ningún valor. |

#### Funciones Anidadas

##### `forbidden_handler(e)`

Manejador de errores global para el código 403, registrado a través de un Blueprint.

| Parámetro | Tipo                                  | Requerido | Descripción                                    |
| :-------- | :------------------------------------ | :-------- | :--------------------------------------------- |
| `e`       | `werkzeug.exceptions.HTTPException`   | Sí        | La excepción HTTP que activó el manejador.     |
| **Retorno** | `tuple[str, int]`                     | -         | Tupla con el mensaje de error y el código 403. |

##### `app_forbidden()`

Ruta a nivel de aplicación que aborta la petición con un error 403.

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| **Raises** | `werkzeug.exceptions.Forbidden` | - | Siempre aborta la petición con un error 403. |

##### `bp_forbidden()`

Ruta de un Blueprint que aborta la petición con un error 403.

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| **Raises** | `werkzeug.exceptions.Forbidden` | - | Siempre aborta la petición con un error 403. |

---

### `test_blueprint_prefix_slash(app, client, prefix, rule, url)`

Prueba parametrizada que verifica la correcta construcción de URLs cuando se combinan diferentes `url_prefix` en el Blueprint y reglas de ruta, especialmente en lo que respecta al manejo de las barras (`/`).

**Parámetros de Entrada**

| Nombre   | Tipo                          | Requerido | Descripción                                                              |
| :------- | :---------------------------- | :-------- | :----------------------------------------------------------------------- |
| `app`    | `flask.Flask`                 | Sí        | Fixture de Pytest que proporciona una instancia de la aplicación Flask.  |
| `client` | `flask.testing.FlaskClient`   | Sí        | Fixture de Pytest que proporciona un cliente de prueba para la aplicación. |
| `prefix` | `str`                         | Sí        | El `url_prefix` a usar en el Blueprint (parametrizado).                  |
| `rule`   | `str`                         | Sí        | La regla de URL a usar en la ruta (parametrizado).                       |
| `url`    | `str`                         | Sí        | La URL final esperada que se probará (parametrizado).                    |

**Valores de Retorno**

| Tipo     | Descripción                                         |
| :------- | :-------------------------------------------------- |
| `None`   | Esta función es una prueba y no retorna ningún valor. |

#### Funciones Anidadas

##### `index()`

Ruta simple que devuelve una respuesta vacía con estado 204.

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| **Retorno** | `tuple[str, int]` | - | Tupla con cuerpo vacío y código de estado 204. |

---

### `test_blueprint_url_defaults(app, client)`

Comprueba que los valores por defecto para los parámetros de una URL (`url_defaults`) se pueden establecer tanto al registrar el Blueprint como en la propia definición de la ruta, y que se aplican correctamente.

**Parámetros de Entrada**

| Nombre   | Tipo                          | Requerido | Descripción                                                              |
| :------- | :---------------------------- | :-------- | :----------------------------------------------------------------------- |
| `app`    | `flask.Flask`                 | Sí        | Fixture de Pytest que proporciona una instancia de la aplicación Flask.  |
| `client` | `flask.testing.FlaskClient`   | Sí        | Fixture de Pytest que proporciona un cliente de prueba para la aplicación. |

**Valores de Retorno**

| Tipo     | Descripción                                         |
| :------- | :-------------------------------------------------- |
| `None`   | Esta función es una prueba y no retorna ningún valor. |

#### Funciones Anidadas

##### `foo(bar, baz)`

Ruta que acepta dos parámetros y devuelve una cadena formateada con sus valores.

| Parámetro | Tipo  | Requerido | Descripción                                    |
| :-------- | :---- | :-------- | :--------------------------------------------- |
| `bar`     | `any` | Sí        | Parámetro de URL, proporcionado por `url_defaults`. |
| `baz`     | `int` | Sí        | Parámetro de URL, proporcionado por `defaults`.    |
| **Retorno** | `str` | -         | Cadena con los valores de `bar` y `baz`.       |

##### `bar(bar)`

Ruta que acepta un parámetro y devuelve su valor como cadena.

| Parámetro | Tipo  | Requerido | Descripción                                    |
| :-------- | :---- | :-------- | :--------------------------------------------- |
| `bar`     | `any` | Sí        | Parámetro de URL, proporcionado por `url_defaults`. |
| **Retorno** | `str` | -         | Cadena con el valor de `bar`.                  |

---

### `test_blueprint_url_processors(app, client)`

Verifica el funcionamiento de los procesadores de URL de un Blueprint: `url_defaults` para inyectar valores al generar URLs y `url_value_preprocessor` para procesar valores de la URL antes de que lleguen a la vista.

**Parámetros de Entrada**

| Nombre   | Tipo                          | Requerido | Descripción                                                              |
| :------- | :---------------------------- | :-------- | :----------------------------------------------------------------------- |
| `app`    | `flask.Flask`                 | Sí        | Fixture de Pytest que proporciona una instancia de la aplicación Flask.  |
| `client` | `flask.testing.FlaskClient`   | Sí        | Fixture de Pytest que proporciona un cliente de prueba para la aplicación. |

**Valores de Retorno**

| Tipo     | Descripción                                         |
| :------- | :-------------------------------------------------- |
| `None`   | Esta función es una prueba y no retorna ningún valor. |

#### Funciones Anidadas

##### `add_language_code(endpoint, values)`

Procesador `url_defaults` que añade un código de idioma por defecto al generar URLs si no está presente.

| Parámetro | Tipo   | Requerido | Descripción                                    |
| :-------- | :----- | :-------- | :--------------------------------------------- |
| `endpoint`| `str`  | Sí        | El endpoint para el que se genera la URL.      |
| `values`  | `dict` | Sí        | Diccionario de valores para la URL.            |

##### `pull_lang_code(endpoint, values)`

Procesador `url_value_preprocessor` que extrae el código de idioma de los valores de la URL y lo almacena en el objeto global `flask.g`.

| Parámetro | Tipo   | Requerido | Descripción                                    |
| :-------- | :----- | :-------- | :--------------------------------------------- |
| `endpoint`| `str`  | Sí        | El endpoint que se está procesando.            |
| `values`  | `dict` | Sí        | Diccionario de valores extraídos de la URL.    |

##### `index()`

Ruta que devuelve la URL generada para la vista `about` del mismo Blueprint.

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| **Retorno** | `str` | - | La URL generada para la vista `.about`. |

##### `about()`

Ruta que devuelve la URL generada para la vista `index` del mismo Blueprint.

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| **Retorno** | `str` | - | La URL generada para la vista `.index`. |

---

### `test_templates_and_static(test_apps)`

Prueba la capacidad de un Blueprint para servir plantillas y archivos estáticos desde sus propias carpetas (`template_folder` y `static_folder`). Verifica el acceso a los archivos, la correcta generación de URLs estáticas y el manejo de la caché.

**Parámetros de Entrada**

| Nombre      | Tipo   | Requerido | Descripción                                                                                             |
| :---------- | :----- | :-------- | :------------------------------------------------------------------------------------------------------ |
| `test_apps` | `Any`  | Sí        | Fixture de Pytest que proporciona acceso a aplicaciones Flask preconfiguradas para pruebas de Blueprints. |

**Valores de Retorno**

| Tipo     | Descripción                                         |
| :------- | :-------------------------------------------------- |
| `None`   | Esta función es una prueba y no retorna ningún valor. |

**Excepciones (Raises)**

| Tipo               | Descripción                                                              |
| :----------------- | :----------------------------------------------------------------------- |
| `TemplateNotFound` | Se espera que se lance al intentar renderizar una plantilla no existente. |

---

### `test_default_static_max_age(app: flask.Flask)`

Verifica que se puede personalizar la directiva `max-age` de la caché para archivos estáticos de un Blueprint sobrescribiendo el método `get_send_file_max_age`.

**Parámetros de Entrada**

| Nombre | Tipo          | Requerido | Descripción                                                              |
| :----- | :------------ | :-------- | :----------------------------------------------------------------------- |
| `app`  | `flask.Flask` | Sí        | Fixture de Pytest que proporciona una instancia de la aplicación Flask.  |

**Valores de Retorno**

| Tipo     | Descripción                                         |
| :------- | :-------------------------------------------------- |
| `None`   | Esta función es una prueba y no retorna ningún valor. |

#### Clases y Métodos Anidados

##### `class MyBlueprint(flask.Blueprint)`

Clase Blueprint personalizada que sobrescribe un método para controlar la caché de archivos estáticos.

##### `get_send_file_max_age(self, filename)`

Método que devuelve un valor `max-age` fijo (100 segundos) para los archivos estáticos servidos por este Blueprint.

| Parámetro | Tipo  | Requerido | Descripción                                    |
| :-------- | :---- | :-------- | :--------------------------------------------- |
| `self`    | `MyBlueprint` | Sí | La instancia del Blueprint. |
| `filename`| `str` | Sí | El nombre del archivo estático solicitado. |
| **Retorno** | `int` | - | El valor `max-age` en segundos. |

---

### `test_templates_list(test_apps)`

Comprueba que el entorno de Jinja2 de la aplicación reconoce y lista correctamente las plantillas provenientes de los Blueprints registrados.

**Parámetros de Entrada**

| Nombre      | Tipo   | Requerido | Descripción                                                                                             |
| :---------- | :----- | :-------- | :------------------------------------------------------------------------------------------------------ |
| `test_apps` | `Any`  | Sí        | Fixture de Pytest que proporciona acceso a aplicaciones Flask preconfiguradas para pruebas de Blueprints. |

**Valores de Retorno**

| Tipo     | Descripción                                         |
| :------- | :-------------------------------------------------- |
| `None`   | Esta función es una prueba y no retorna ningún valor. |

---

### `test_dotted_name_not_allowed(app, client)`

Asegura que la creación de un `flask.Blueprint` con un nombre que contiene puntos (`.`) lanza un `ValueError`, ya que esto está prohibido para evitar conflictos de nomenclatura.

**Parámetros de Entrada**

| Nombre   | Tipo                          | Requerido | Descripción                                                              |
| :------- | :---------------------------- | :-------- | :----------------------------------------------------------------------- |
| `app`    | `flask.Flask`                 | Sí        | Fixture de Pytest que proporciona una instancia de la aplicación Flask.  |
| `client` | `flask.testing.FlaskClient`   | Sí        | Fixture de Pytest que proporciona un cliente de prueba para la aplicación. |

**Excepciones (Raises)**

| Tipo         | Descripción                                                              |
| :----------- | :----------------------------------------------------------------------- |
| `ValueError` | Se espera que se lance al intentar crear un Blueprint con un nombre punteado. |

---

### `test_empty_name_not_allowed(app, client)`

Asegura que la creación de un `flask.Blueprint` con un nombre vacío lanza un `ValueError`.

**Parámetros de Entrada**

| Nombre   | Tipo                          | Requerido | Descripción                                                              |
| :------- | :---------------------------- | :-------- | :----------------------------------------------------------------------- |
| `app`    | `flask.Flask`                 | Sí        | Fixture de Pytest que proporciona una instancia de la aplicación Flask.  |
| `client` | `flask.testing.FlaskClient`   | Sí        | Fixture de Pytest que proporciona un cliente de prueba para la aplicación. |

**Excepciones (Raises)**

| Tipo         | Descripción                                                              |
| :----------- | :----------------------------------------------------------------------- |
| `ValueError` | Se espera que se lance al intentar crear un Blueprint con un nombre vacío. |

---

### `test_dotted_names_from_app(app, client)`

Verifica que es posible generar URLs para endpoints de un Blueprint desde una ruta de la aplicación principal y viceversa, utilizando la notación de punto (`blueprint_name.endpoint_name`).

**Parámetros de Entrada**

| Nombre   | Tipo                          | Requerido | Descripción                                                              |
| :------- | :---------------------------- | :-------- | :----------------------------------------------------------------------- |
| `app`    | `flask.Flask`                 | Sí        | Fixture de Pytest que proporciona una instancia de la aplicación Flask.  |
| `client` | `flask.testing.FlaskClient`   | Sí        | Fixture de Pytest que proporciona un cliente de prueba para la aplicación. |

**Valores de Retorno**

| Tipo     | Descripción                                         |
| :------- | :-------------------------------------------------- |
| `None`   | Esta función es una prueba y no retorna ningún valor. |

#### Funciones Anidadas

##### `app_index()`

Ruta de la aplicación que genera y devuelve la URL para la vista `index` del Blueprint `test`.

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| **Retorno** | `str` | - | La URL generada para `test.index`. |

##### `index()`

Ruta del Blueprint `test` que genera y devuelve la URL para la vista `app_index` de la aplicación.

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| **Retorno** | `str` | - | La URL generada para `app_index`. |

---

### `test_empty_url_defaults(app, client)`

Comprueba que se pueden definir rutas con valores por defecto para parámetros de URL, permitiendo que una misma función de vista maneje múltiples patrones de URL.

**Parámetros de Entrada**

| Nombre   | Tipo                          | Requerido | Descripción                                                              |
| :------- | :---------------------------- | :-------- | :----------------------------------------------------------------------- |
| `app`    | `flask.Flask`                 | Sí        | Fixture de Pytest que proporciona una instancia de la aplicación Flask.  |
| `client` | `flask.testing.FlaskClient`   | Sí        | Fixture de Pytest que proporciona un cliente de prueba para la aplicación. |

**Valores de Retorno**

| Tipo     | Descripción                                         |
| :------- | :-------------------------------------------------- |
| `None`   | Esta función es una prueba y no retorna ningún valor. |

#### Funciones Anidadas

##### `something(page)`

Vista que maneja dos rutas: una con un valor por defecto para `page` y otra que lo captura de la URL.

| Parámetro | Tipo  | Requerido | Descripción                                    |
| :-------- | :---- | :-------- | :--------------------------------------------- |
| `page`    | `int` | Sí        | El número de página, ya sea por defecto o de la URL. |
| **Retorno** | `str` | -         | El número de página como cadena.               |

---

### `test_route_decorator_custom_endpoint(app, client)`

Verifica que el decorador `@bp.route` permite especificar un `endpoint` personalizado. Si no se especifica, el endpoint se deriva del nombre de la función de vista.

**Parámetros de Entrada**

| Nombre   | Tipo                          | Requerido | Descripción                                                              |
| :------- | :---------------------------- | :-------- | :----------------------------------------------------------------------- |
| `app`    | `flask.Flask`                 | Sí        | Fixture de Pytest que proporciona una instancia de la aplicación Flask.  |
| `client` | `flask.testing.FlaskClient`   | Sí        | Fixture de Pytest que proporciona un cliente de prueba para la aplicación. |

**Valores de Retorno**

| Tipo     | Descripción                                         |
| :------- | :-------------------------------------------------- |
| `None`   | Esta función es una prueba y no retorna ningún valor. |

#### Funciones Anidadas

Todas las funciones de vista (`foo`, `foo_bar`, `foo_bar_foo`, `bar_foo`, `index`) devuelven el nombre del endpoint actual (`flask.request.endpoint`) para verificar que se ha registrado correctamente.

---

### `test_route_decorator_custom_endpoint_with_dots(app, client)`

Asegura que el uso de puntos (`.`) en el nombre de un endpoint al definir una ruta en un Blueprint (ya sea con `@bp.route` o `bp.add_url_rule`) lanza un `ValueError`.

**Parámetros de Entrada**

| Nombre   | Tipo                          | Requerido | Descripción                                                              |
| :------- | :---------------------------- | :-------- | :----------------------------------------------------------------------- |
| `app`    | `flask.Flask`                 | Sí        | Fixture de Pytest que proporciona una instancia de la aplicación Flask.  |
| `client` | `flask.testing.FlaskClient`   | Sí        | Fixture de Pytest que proporciona un cliente de prueba para la aplicación. |

**Excepciones (Raises)**

| Tipo         | Descripción                                                              |
| :----------- | :----------------------------------------------------------------------- |
| `ValueError` | Se espera que se lance al intentar usar un punto en un nombre de endpoint. |

#### Funciones Anidadas

##### `view()`

Función de vista simple utilizada para la prueba con `add_url_rule`.

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| **Retorno** | `str` | - | Una cadena vacía. |

---

### `test_endpoint_decorator(app, client)`

Verifica que el decorador `@bp.endpoint` puede asociar una función de vista a un endpoint que ya ha sido definido en el mapa de URLs de la aplicación, pero que esta asociación es local al Blueprint y no afecta a la regla original.

**Parámetros de Entrada**

| Nombre   | Tipo                          | Requerido | Descripción                                                              |
| :------- | :---------------------------- | :-------- | :----------------------------------------------------------------------- |
| `app`    | `flask.Flask`                 | Sí        | Fixture de Pytest que proporciona una instancia de la aplicación Flask.  |
| `client` | `flask.testing.FlaskClient`   | Sí        | Fixture de Pytest que proporciona un cliente de prueba para la aplicación. |

**Valores de Retorno**

| Tipo     | Descripción                                         |
| :------- | :-------------------------------------------------- |
| `None`   | Esta función es una prueba y no retorna ningún valor. |

#### Funciones Anidadas

##### `foobar()`

Función de vista asociada al endpoint `bar` dentro del Blueprint.

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| **Retorno** | `str` | - | El nombre del endpoint de la petición (`flask.request.endpoint`). |

---

### `test_template_filter(app)` y funciones relacionadas

Estas funciones (`test_template_filter`, `test_add_template_filter`, `test_template_filter_with_name`, etc.) verifican la capacidad de un Blueprint para registrar filtros de plantillas Jinja2 a nivel de aplicación. Prueban el registro mediante el decorador `@bp.app_template_filter` y el método `bp.add_app_template_filter`, con y sin nombres personalizados, y su correcta ejecución en plantillas.

**Parámetros de Entrada**

| Nombre   | Tipo                          | Requerido | Descripción                                                              |
| :------- | :---------------------------- | :-------- | :----------------------------------------------------------------------- |
| `app`    | `flask.Flask`                 | Sí        | Fixture de Pytest que proporciona una instancia de la aplicación Flask.  |
| `client` | `flask.testing.FlaskClient`   | Sí        | Fixture de Pytest que proporciona un cliente de prueba para la aplicación. |

#### Funciones Anidadas Comunes

##### `my_reverse(s)` / `super_reverse(s)`

Filtro de plantilla que invierte una cadena.

| Parámetro | Tipo  | Requerido | Descripción             |
| :-------- | :---- | :-------- | :---------------------- |
| `s`       | `str` | Sí        | La cadena a invertir.   |
| **Retorno** | `str` | -         | La cadena invertida.    |

##### `index()`

Ruta que renderiza una plantilla para probar el filtro.

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| **Retorno** | `str` | - | El contenido HTML renderizado. |

---

### `test_template_test(app)` y funciones relacionadas

Estas funciones (`test_template_test`, `test_add_template_test`, `test_template_test_with_name`, etc.) verifican la capacidad de un Blueprint para registrar "tests" de plantillas Jinja2 a nivel de aplicación. Prueban el registro mediante el decorador `@bp.app_template_test` y el método `bp.add_app_template_test`, con y sin nombres personalizados, y su correcta evaluación en plantillas.

**Parámetros de Entrada**

| Nombre   | Tipo                          | Requerido | Descripción                                                              |
| :------- | :---------------------------- | :-------- | :----------------------------------------------------------------------- |
| `app`    | `flask.Flask`                 | Sí        | Fixture de Pytest que proporciona una instancia de la aplicación Flask.  |
| `client` | `flask.testing.FlaskClient`   | Sí        | Fixture de Pytest que proporciona un cliente de prueba para la aplicación. |

#### Funciones Anidadas Comunes

##### `is_boolean(value)` / `boolean(value)`

Test de plantilla que comprueba si un valor es de tipo booleano.

| Parámetro | Tipo  | Requerido | Descripción             |
| :-------- | :---- | :-------- | :---------------------- |
| `value`   | `Any` | Sí        | El valor a comprobar.   |
| **Retorno** | `bool`| -         | `True` si es booleano, `False` en caso contrario. |

##### `index()`

Ruta que renderiza una plantilla para probar el test.

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| **Retorno** | `str` | - | El contenido HTML renderizado. |

---

### `test_context_processing(app, client)`

Valida el funcionamiento de los procesadores de contexto. Comprueba que un `app_context_processor` de un Blueprint inyecta variables en el contexto de todas las plantillas de la aplicación, mientras que un `context_processor` solo lo hace para las plantillas renderizadas en las rutas de su propio Blueprint.

**Parámetros de Entrada**

| Nombre   | Tipo                          | Requerido | Descripción                                                              |
| :------- | :---------------------------- | :-------- | :----------------------------------------------------------------------- |
| `app`    | `flask.Flask`                 | Sí        | Fixture de Pytest que proporciona una instancia de la aplicación Flask.  |
| `client` | `flask.testing.FlaskClient`   | Sí        | Fixture de Pytest que proporciona un cliente de prueba para la aplicación. |

**Valores de Retorno**

| Tipo     | Descripción                                         |
| :------- | :-------------------------------------------------- |
| `None`   | Esta función es una prueba y no retorna ningún valor. |

#### Funciones Anidadas

##### `template_string()`

Función auxiliar que renderiza una cadena de plantilla para mostrar las variables de contexto.

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| **Retorno** | `str` | - | La cadena de plantilla renderizada. |

##### `not_answer_context_processor()`

Procesador de contexto a nivel de aplicación que inyecta la variable `notanswer`.

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| **Retorno** | `dict` | - | Diccionario con la variable de contexto. |

##### `answer_context_processor()`

Procesador de contexto a nivel de Blueprint que inyecta la variable `answer`.

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| **Retorno** | `dict` | - | Diccionario con la variable de contexto. |

##### `bp_page()` y `app_page()`

Rutas que llaman a `template_string()` para probar los contextos.

---

### `test_template_global(app)`

Verifica que un Blueprint puede registrar funciones globales para plantillas Jinja2 usando el decorador `@bp.app_template_global`, tanto con el nombre de la función como con un nombre personalizado.

**Parámetros de Entrada**

| Nombre | Tipo          | Requerido | Descripción                                                              |
| :----- | :------------ | :-------- | :----------------------------------------------------------------------- |
| `app`  | `flask.Flask` | Sí        | Fixture de Pytest que proporciona una instancia de la aplicación Flask.  |

**Valores de Retorno**

| Tipo     | Descripción                                         |
| :------- | :-------------------------------------------------- |
| `None`   | Esta función es una prueba y no retorna ningún valor. |

#### Funciones Anidadas

##### `get_answer()`, `get_stuff_1()`, `get_stuff_2()`, `get_stuff_3()`

Funciones globales de plantilla que devuelven valores fijos para ser probados.

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| **Retorno** | `int` o `str` | - | Un valor constante. |

---

### `test_request_processing(app, client)`

Comprueba el ciclo de vida de las funciones de procesamiento de peticiones de un Blueprint: `before_request`, `after_request` y `teardown_request`. Verifica que se ejecutan en el orden correcto y solo para las peticiones dirigidas a ese Blueprint.

**Parámetros de Entrada**

| Nombre   | Tipo                          | Requerido | Descripción                                                              |
| :------- | :---------------------------- | :-------- | :----------------------------------------------------------------------- |
| `app`    | `flask.Flask`                 | Sí        | Fixture de Pytest que proporciona una instancia de la aplicación Flask.  |
| `client` | `flask.testing.FlaskClient`   | Sí        | Fixture de Pytest que proporciona un cliente de prueba para la aplicación. |

**Valores de Retorno**

| Tipo     | Descripción                                         |
| :------- | :-------------------------------------------------- |
| `None`   | Esta función es una prueba y no retorna ningún valor. |

#### Funciones Anidadas

##### `before_bp()`

Función `before_request` que registra su ejecución.

##### `after_bp(response)`

Función `after_request` que modifica la respuesta y registra su ejecución.

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `response` | `flask.Response` | Sí | El objeto de respuesta a punto de ser enviado. |
| **Retorno** | `flask.Response` | - | El objeto de respuesta modificado. |

##### `teardown_bp(exc)`

Función `teardown_request` que registra su ejecución.

| Parámetro | Tipo | Requerido | Descripción |
| :--- | :--- | :--- | :--- |
| `exc` | `Exception` o `None` | Sí | La excepción que ocurrió durante la petición, si la hubo. |

##### `bp_endpoint()`

Ruta del Blueprint para activar los callbacks de petición.

---

### `test_app_request_processing(app, client)`

Comprueba el ciclo de vida de las funciones de procesamiento de peticiones a nivel de aplicación registradas en un Blueprint: `before_app_request`, `after_app_request` y `teardown_app_request`. Verifica que se ejecutan para todas las peticiones de la aplicación.

**Parámetros de Entrada**

| Nombre   | Tipo                          | Requerido | Descripción                                                              |
| :------- | :---------------------------- | :-------- | :----------------------------------------------------------------------- |
| `app`    | `flask.Flask`                 | Sí        | Fixture de Pytest que proporciona una instancia de la aplicación Flask.  |
| `client` | `flask.testing.FlaskClient`   | Sí        | Fixture de Pytest que proporciona un cliente de prueba para la aplicación. |

**Valores de Retorno**

| Tipo     | Descripción                                         |
| :------- | :-------------------------------------------------- |
| `None`   | Esta función es una prueba y no retorna ningún valor. |

#### Funciones Anidadas

Similares a `test_request_processing`, pero registradas con decoradores a nivel de aplicación (`@bp.before_app_request`, etc.).

---

### `test_app_url_processors(app, client)`

Verifica que los procesadores de URL a nivel de aplicación (`app_url_defaults` y `app_url_value_preprocessor`) registrados en un Blueprint se aplican a todas las rutas de la aplicación, no solo a las del Blueprint.

**Parámetros de Entrada**

| Nombre   | Tipo                          | Requerido | Descripción                                                              |
| :------- | :---------------------------- | :-------- | :----------------------------------------------------------------------- |
| `app`    | `flask.Flask`                 | Sí        | Fixture de Pytest que proporciona una instancia de la aplicación Flask.  |
| `client` | `flask.testing.FlaskClient`   | Sí        | Fixture de Pytest que proporciona un cliente de prueba para la aplicación. |

**Valores de Retorno**

| Tipo     | Descripción                                         |
| :------- | :-------------------------------------------------- |
| `None`   | Esta función es una prueba y no retorna ningún valor. |

#### Funciones Anidadas

Similares a `test_blueprint_url_processors`, pero registradas con decoradores a nivel de aplicación (`@bp.app_url_defaults`, etc.) y probadas con rutas de la aplicación principal.

---

### `test_nested_blueprint(app, client)`

Prueba el anidamiento de Blueprints (un Blueprint registrado dentro de otro). Verifica que las rutas se construyen correctamente concatenando los `url_prefix` y que la herencia de manejadores de errores funciona como se espera (el manejador del ancestro más cercano se utiliza).

**Parámetros de Entrada**

| Nombre   | Tipo                          | Requerido | Descripción                                                              |
| :------- | :---------------------------- | :-------- | :----------------------------------------------------------------------- |
| `app`    | `flask.Flask`                 | Sí        | Fixture de Pytest que proporciona una instancia de la aplicación Flask.  |
| `client` | `flask.testing.FlaskClient`   | Sí        | Fixture de Pytest que proporciona un cliente de prueba para la aplicación. |

**Valores de Retorno**

| Tipo     | Descripción                                         |
| :------- | :-------------------------------------------------- |
| `None`   | Esta función es una prueba y no retorna ningún valor. |

#### Funciones Anidadas

Se definen tres Blueprints (`parent`, `child`, `grandchild`) con sus respectivas rutas y manejadores de errores para probar la jerarquía.

---

### `test_nested_callback_order(app, client)`

Verifica el orden de ejecución de los callbacks (`before_request`, `teardown_request`, `context_processor`) en una estructura de Blueprints anidados. El orden esperado es: aplicación -> padre -> hijo. Para los procesadores de contexto, el del hijo tiene precedencia.

**Parámetros de Entrada**

| Nombre   | Tipo                          | Requerido | Descripción                                                              |
| :------- | :---------------------------- | :-------- | :----------------------------------------------------------------------- |
| `app`    | `flask.Flask`                 | Sí        | Fixture de Pytest que proporciona una instancia de la aplicación Flask.  |
| `client` | `flask.testing.FlaskClient`   | Sí        | Fixture de Pytest que proporciona un cliente de prueba para la aplicación. |

**Valores de Retorno**

| Tipo     | Descripción                                         |
| :------- | :-------------------------------------------------- |
| `None`   | Esta función es una prueba y no retorna ningún valor. |

#### Funciones Anidadas

Se definen múltiples callbacks en la aplicación y en los Blueprints anidados para registrar su orden de ejecución en `flask.g.seen`.

---

### `test_nesting_url_prefixes(...)`

Prueba parametrizada que verifica todas las combinaciones posibles de definir `url_prefix` (en la inicialización del Blueprint y/o en su registro) para Blueprints anidados, asegurando que la URL final se construye correctamente.

**Parámetros de Entrada**

| Nombre                | Tipo                          | Requerido | Descripción                                                              |
| :-------------------- | :---------------------------- | :-------- | :----------------------------------------------------------------------- |
| `parent_init`         | `str` o `None`                | Sí        | `url_prefix` en la creación del Blueprint padre.                         |
| `child_init`          | `str` o `None`                | Sí        | `url_prefix` en la creación del Blueprint hijo.                          |
| `parent_registration` | `str` o `None`                | Sí        | `url_prefix` en el registro del Blueprint padre en la app.               |
| `child_registration`  | `str` o `None`                | Sí        | `url_prefix` en el registro del Blueprint hijo en el padre.              |
| `app`                 | `flask.Flask`                 | Sí        | Fixture de Pytest.                                                       |
| `client`              | `flask.testing.FlaskClient`   | Sí        | Fixture de Pytest.                                                       |

**Valores de Retorno**

| Tipo     | Descripción                                         |
| :------- | :-------------------------------------------------- |
| `None`   | Esta función es una prueba y no retorna ningún valor. |

---

### `test_nesting_subdomains(app, client)`

Verifica que un Blueprint padre registrado con un subdominio pasa ese subdominio a sus Blueprints hijos anidados.

**Parámetros de Entrada**

| Nombre   | Tipo                          | Requerido | Descripción                                                              |
| :------- | :---------------------------- | :-------- | :----------------------------------------------------------------------- |
| `app`    | `flask.Flask`                 | Sí        | Fixture de Pytest.                                                       |
| `client` | `flask.testing.FlaskClient`   | Sí        | Fixture de Pytest.                                                       |

**Valores de Retorno**

| Tipo     | Descripción                                         |
| :------- | :-------------------------------------------------- |
| `None`   | Esta función es una prueba y no retorna ningún valor. |

---

### `test_child_and_parent_subdomain(app, client)`

Comprueba que si tanto un Blueprint padre como uno hijo definen un subdominio, estos se concatenan para formar el subdominio final (ej: `api.parent.example.test`).

**Parámetros de Entrada**

| Nombre   | Tipo                          | Requerido | Descripción                                                              |
| :------- | :---------------------------- | :-------- | :----------------------------------------------------------------------- |
| `app`    | `flask.Flask`                 | Sí        | Fixture de Pytest.                                                       |
| `client` | `flask.testing.FlaskClient`   | Sí        | Fixture de Pytest.                                                       |

**Valores de Retorno**

| Tipo     | Descripción                                         |
| :------- | :-------------------------------------------------- |
| `None`   | Esta función es una prueba y no retorna ningún valor. |

---

### `test_unique_blueprint_names(app, client)`

Asegura que los nombres de los Blueprints registrados en una aplicación deben ser únicos. Permite registrar el mismo Blueprint varias veces si se le asigna un nombre diferente cada vez con el parámetro `name`.

**Parámetros de Entrada**

| Nombre   | Tipo                          | Requerido | Descripción                                                              |
| :------- | :---------------------------- | :-------- | :----------------------------------------------------------------------- |
| `app`    | `flask.Flask`                 | Sí        | Fixture de Pytest.                                                       |
| `client` | `flask.testing.FlaskClient`   | Sí        | Fixture de Pytest.                                                       |

**Excepciones (Raises)**

| Tipo         | Descripción                                                              |
| :----------- | :----------------------------------------------------------------------- |
| `ValueError` | Se espera que se lance al registrar un Blueprint con un nombre ya en uso. |

---

### `test_self_registration(app, client)`

Verifica que un Blueprint no puede registrarse a sí mismo como hijo, lo que causaría una recursión infinita.

**Parámetros de Entrada**

| Nombre   | Tipo                          | Requerido | Descripción                                                              |
| :------- | :---------------------------- | :-------- | :----------------------------------------------------------------------- |
| `app`    | `flask.Flask`                 | Sí        | Fixture de Pytest.                                                       |
| `client` | `flask.testing.FlaskClient`   | Sí        | Fixture de Pytest.                                                       |

**Excepciones (Raises)**

| Tipo         | Descripción                                                              |
| :----------- | :----------------------------------------------------------------------- |
| `ValueError` | Se espera que se lance al intentar registrar un Blueprint en sí mismo.   |

---

### `test_blueprint_renaming(app, client)`

Prueba el renombramiento de Blueprints durante el registro. Verifica que al registrar un Blueprint con un `name` diferente, los endpoints, rutas y manejadores de errores se asocian correctamente al nuevo nombre.

**Parámetros de Entrada**

| Nombre   | Tipo                          | Requerido | Descripción                                                              |
| :------- | :---------------------------- | :-------- | :----------------------------------------------------------------------- |
| `app`    | `flask.Flask`                 | Sí        | Fixture de Pytest.                                                       |
| `client` | `flask.testing.FlaskClient`   | Sí        | Fixture de Pytest.                                                       |

**Valores de Retorno**

| Tipo     | Descripción                                         |
| :------- | :-------------------------------------------------- |
| `None`   | Esta función es una prueba y no retorna ningún valor. |

#### Funciones Anidadas

Se definen varias rutas y un manejador de errores en los Blueprints para verificar que el renombramiento afecta a todos los aspectos del Blueprint.

## 3. Ejemplo de uso

El siguiente bloque de código muestra un caso de uso práctico de `flask.Blueprint` para modularizar una aplicación web simple. Se crea un Blueprint para gestionar un blog, con su propia ruta y un filtro de plantilla personalizado.

```python
import flask

# 1. Crear una instancia de la aplicación principal
app = flask.Flask(__name__)

# 2. Crear un Blueprint para la sección del blog
# Se especifica una carpeta de plantillas y un prefijo de URL
blog_bp = flask.Blueprint(
    'blog',
    __name__,
    template_folder='templates',
    url_prefix='/blog'
)

# 3. Definir un filtro de plantilla personalizado en el Blueprint
@blog_bp.app_template_filter('uppercase')
def to_uppercase(s: str) -> str:
    """Filtro de plantilla que convierte una cadena a mayúsculas."""
    return s.upper()

# 4. Definir una ruta dentro del Blueprint
@blog_bp.route('/')
def index():
    """
    Renderiza la página principal del blog.
    La URL final será /blog/
    """
    # El nombre de la plantilla se resuelve desde la 'template_folder' del Blueprint
    return flask.render_template('blog_index.html', title="Mi Blog con Blueprints")

# 5. Registrar el Blueprint en la aplicación principal
app.register_blueprint(blog_bp)

# Ruta en la aplicación principal para demostrar la modularidad
@app.route('/')
def home():
    """Página de inicio de la aplicación."""
    return "<h1>Página Principal</h1><p>Visita el <a href='/blog/'>Blog</a>.</p>"

if __name__ == '__main__':
    # Para ejecutar este ejemplo:
    # 1. Crea una carpeta 'templates' en el mismo directorio.
    # 2. Dentro de 'templates', crea un archivo 'blog_index.html' con el contenido:
    #    <!DOCTYPE html>
    #    <html>
    #    <head><title>{{ title }}</title></head>
    #    <body>
    #      <h1>{{ "Bienvenido al blog" | uppercase }}</h1>
    #    </body>
    #    </html>
    # 3. Ejecuta el script y visita http://127.0.0.1:5000/ y http://127.0.0.1:5000/blog/
    app.run(debug=True)

```