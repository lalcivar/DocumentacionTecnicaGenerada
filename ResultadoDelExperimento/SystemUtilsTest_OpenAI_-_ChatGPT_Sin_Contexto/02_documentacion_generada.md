Documentación técnica  
Clase: org.apache.commons.lang3.SystemUtilsTest  
Módulo: commons-lang3 (Apache Commons Lang)

-------------------------------------------------------------------------------
1. Propósito general
-------------------------------------------------------------------------------
SystemUtilsTest es la suite de pruebas JUnit 5 que valida el correcto
funcionamiento de la clase utilitaria SystemUtils.  SystemUtils concentra
métodos y constantes de conveniencia para:

• Identificar la versión de Java que se está ejecutando.  
• Detectar el sistema operativo y su versión.  
• Recuperar información básica del entorno de ejecución (HOME, JAVA_HOME, TMP,
  hostname, variables de entorno, etc.).  
• Proporcionar utilidades de comparación de versión de Java y del sistema
  operativo.

SystemUtilsTest verifica cada uno de estos contratos ejecutando pruebas
unitarias exhaustivas sobre:
– Constantes booleanas IS_JAVA_x e IS_OS_x.  
– Métodos getXXX (JavaHome, UserHome, TmpDir, HostName, etc.).  
– Comparadores de versión isJavaVersionAtLeast / isJavaVersionAtMost.  
– Predicados de coincidencia de versiones y nombres
  (isJavaVersionMatch, isOsMatch, isOsNameMatch, isOsVersionMatch).  

-------------------------------------------------------------------------------
2. Métodos principales de la clase de prueba
-------------------------------------------------------------------------------
Aunque la clase contiene más de 200 métodos de prueba generados de forma
sistemática, los puntos de entrada clave son los siguientes:

Métodos auxiliares (no anotados con @Test)
------------------------------------------
1. boolean getIS_JAVA(int version)
   • Recupera por reflexión el valor de la constante
     SystemUtils.IS_JAVA_<versión> para Java 9 o superior.
   • Parámetros: versión — número de versión mayor (9,10,11,…).
   • Excepciones: IllegalAccessException | NoSuchFieldException.

2. int getLastSupportedJavaVersion()
   • Recorre los campos públicos de SystemUtils y obtiene la última constante
     IS_JAVA_X disponible; se emplea para escalar dinámicamente las aserciones
     a futuras versiones de Java.

Métodos de prueba (anotados con @Test)
--------------------------------------
1. test_IS_JAVA()  
   • Valida que exactamente una de las constantes IS_JAVA_* sea true según la
     versión real de la JVM.

2. test_IS_OS() / test_IS_zOS()  
   • Comprueban la detección del sistema operativo mediante las banderas
     IS_OS_* incluidas las variantes específicas de macOS y z/OS.

3. testConstructor()  
   • Verifica los modificadores (public non-final) y la existencia de un único
     constructor sin argumentos en SystemUtils.

4. testGetEnvironmentVariableAbsent() / testGetEnvironmentVariablePresent()  
   • Aseguran la correcta lectura de variables de entorno y el uso de valores
     por defecto.

5. testGetHostName(), testGetUserName(), test_USER_NAME()  
   • Evalúan la recuperación de información de usuario y equipo.

6. testGetJavaHome*, testGetJavaIoTmpDir*, testGetUserDir*, testGetUserHome*  
   • Comprueban los métodos que exponen rutas de interés como File y como Path
     asegurando que existen en el sistema.

7. testIsJavaVersionAtLeast() / testIsJavaVersionAtMost()  
   • Ejercitan los predicados de comparación con todas las constantes de la
     enumeración JavaVersion (JAVA_1_1 … JAVA_27).

8. testJavaAwtHeadless()  
   • Valida la correcta lectura de la propiedad «java.awt.headless».

9. testJavaVersionMatches()  
   • Verifica el algoritmo isJavaVersionMatch para múltiples strings de
     versión simuladas.

10. testOSMatchesName() / testOSMatchesNameAndVersion() / testOsVersionMatches()  
    • Verifican los métodos de coincidencia de nombre y versión de sistema
      operativo.

-------------------------------------------------------------------------------
3. Dependencias externas
-------------------------------------------------------------------------------
• JUnit 5 (jupiter-api y jupiter-engine).  
• Apache Commons Lang 3 (la propia clase SystemUtils, BooleanUtils, etc.).  
• java.nio.file.Files para validar la existencia de rutas.

-------------------------------------------------------------------------------
4. Consideraciones relevantes
-------------------------------------------------------------------------------
• Cobertura futura: getLastSupportedJavaVersion() y getIS_JAVA() permiten que
  la batería de pruebas se adapte automáticamente cuando Apache Commons Lang
  añada nuevas constantes IS_JAVA_X sin necesidad de modificar el código de
  prueba.  
• El conjunto de pruebas asume que NO existe SecurityManager; si se ejecutan
  bajo entornos con permisos restringidos podrían fallar los accesos a
  propiedades de sistema o al sistema de archivos.  
• Algunas aserciones se delegan al propio entorno de ejecución; si el sistema
  operativo o la JVM retornan valores inesperados las pruebas emitirán
  mensajes informativos en lugar de fallar (véase los bloques «Can't test …»).

-------------------------------------------------------------------------------
5. Ejecución
-------------------------------------------------------------------------------
Las pruebas se ejecutan con cualquier runner JUnit 5 estándar:

$ mvn test
ó
$ gradle test

No requieren configuración adicional más allá de la definición de la
dependencia commons-lang3 en el classpath de prueba.

-------------------------------------------------------------------------------
6. Historial
-------------------------------------------------------------------------------
Versión original incluida en Apache Commons Lang 3.x para garantizar la
retro-compatibilidad de SystemUtils con nuevas versiones de Java y con
diversas familias de sistemas operativos.

-------------------------------------------------------------------------------
Fin del documento