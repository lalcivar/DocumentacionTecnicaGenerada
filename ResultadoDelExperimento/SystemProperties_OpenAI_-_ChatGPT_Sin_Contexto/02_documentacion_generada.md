================================================================================
Documentación técnica
Clase: org.apache.commons.lang3.SystemProperties
Módulo: Apache Commons Lang 3.x
Desde: 3.13.0
================================================================================

1. Propósito general
--------------------
SystemProperties es una clase de utilidades **final** y **no instanciable** que
ofrece:

1. Un catálogo público (`public static final String`) con la gran mayoría de los
   nombres de propiedades del sistema reconocidos por la plataforma Java
   (JDK 8-21).  
2. Métodos auxiliares para obtener de forma **segura** (sin propagar
   `SecurityException`) el valor de dichas propiedades en distintos tipos
   (`String`, `boolean`, `int`, `long`, `Path`, …) o bien determinar si la
   propiedad está definida.  
3. Una pequeña capa de “calidad de vida”:
   • Conversión tipada con reglas idénticas a las clases envoltorio
     (`Boolean.parseBoolean`, `Integer.parseInt`, `Long.parseLong`).  
   • Valores por defecto suministrados mediante lambdas
     (`java.util.function.Supplier`, `BooleanSupplier`, etc.).  
   • Método helper `toKey()` para construir claves con el patrón
     «SimpleClassName.subClave».

La clase sirve como sustituto de literales sueltos dispersos y como envoltorio
seguro de `System.getProperty()` cuando la aplicación se ejecuta bajo
restricciones de *SecurityManager* o *SecurityPolicy*.

2. Estructura interna
---------------------
• **Constantes**:  
  Cientos de campos `public static final String` con el nombre exacto de la
  propiedad.  Algunos están anotados con `@since` o `@deprecated` y enlazan a la
  documentación oficial del JDK. La clase **no** almacena los valores—solo los
  nombres.

• **Constructor**:  
  `private` a partir de la versión 4.0; para compatibilidad actual permanece
  `public` pero marcado `@Deprecated`.

• **Métodos públicos destacados**  

  2.1  Acceso genérico
  --------------------
  String getProperty(String key)  
  String getProperty(String key, String defaultIfAbsent)  
  String getProperty(String key, Supplier<String> defaultIfAbsent)

  – Realiza `System.getProperty(key)` dentro de un bloque *try/catch*.  
  – Si la clave es `null`/vacía o hay `SecurityException` devuelve el valor
    proporcionado por el *supplier* (o `null`).

  2.2  Conversión tipada genérica
  -------------------------------
  boolean getBoolean(String key, BooleanSupplier defaultIfAbsent)  
  int     getInt    (String key, IntSupplier     defaultIfAbsent)  
  long    getLong   (String key, LongSupplier    defaultIfAbsent)  
  Path    getPath   (String key, Supplier<Path>  defaultIfAbsent)

  – Obtienen la cadena con `getProperty()` y la parsean.  
  – Si la propiedad no existe o falla, se ejecuta el *supplier* de
    respaldo.  
  – Existe sobrecarga `(Class<?> clazz, String subKey, …)` que antepone el
    `clazz.getSimpleName()` y un punto antes de la subclave (útil para claves
    “namespaced”, p. ej. *MyClass.debug*).

  2.3  Métodos específicos por propiedad
  --------------------------------------
  Para cada constante existe un «getter» dedicado, p. ej.:

      String getJavaVersion()
      String getOsName()
      String getHttpProxyHost()
      …

  Su implementación es un simple `return getProperty(CONSTANTE);`, lo que aporta
  sintaxis auto-documentada al llamante.

  2.4  Utilidades
  ---------------
  boolean isPropertySet(String key) – atajo de `getProperty(key) != null`.

3. Uso típico
-------------
```java
// Obtener la arquitectura de la JVM, sin preocuparnos de SecurityManager
String arch = SystemProperties.getOsArch();

// Leer «-Dapp.debug=true» con valor por defecto
boolean debug = SystemProperties.getBoolean(
        MyApp.class,           // Prefijo "MyApp."
        "debug",
        () -> false);          // lambda de respaldo

// Construir un Path desde una propiedad (o ~/data si no existe)
Path dataDir = SystemProperties.getPath("app.data.dir",
        () -> Paths.get(SystemProperties.getUserHome(), "data"));
```

4. Consideraciones de diseño
----------------------------
• **Inmutabilidad & Thread safety**:  Todos los miembros son `static final` o
  métodos puros; la clase es inherentemente segura para concurrencia.  
• **Cobertura de JDK**:  Incluye propiedades que han sido retiradas o marcadas
  como *deprecated* en Java 9+, manteniendo `@Deprecated` para reflejar el
  estado oficial.  
• **Dependencias**: Utiliza clases auxiliares de Apache Commons Lang
  (`StringUtils`, `ClassUtils`, `Suppliers`).  
• **Rendimiento**: La llamada real a `System.getProperty()` sólo ocurre cuando
  el cliente invoca un método; la clase no hace *caching*.

5. Extensión y cambios futuros
------------------------------
– El constructor quedará privado en la versión 4.0 (ver anotación
  `@Deprecated`).  
– Pueden añadirse más constantes y *getters* conforme el JDK publique nuevas
  propiedades.

6. Resumen
----------
SystemProperties centraliza la gestión de *System Properties* proporcionando:
• Nombres de propiedades estandarizados.  
• Lectura segura y conversiones tipadas.  
• Compatibilidad con políticas de seguridad y valores por defecto
  diferidos.  

Es una pieza fundamental para escribir código portable y auto-documentado que
necesite interactuar con el entorno de ejecución de la JVM.