-------------------------------------------------------------
Documentación técnica  
Clase: org.apache.commons.lang3.StringUtilsTest  
-------------------------------------------------------------

1. Propósito general
--------------------
StringUtilsTest es la batería de pruebas de regresión que valida el 100 % de la API pública de
org.apache.commons.lang3.StringUtils (y parte de StringEscapeUtils).  
Su misión es garantizar que:

* Todos los escenarios contemplados en la documentación oficial se cumplan.
* Los bugs reportados en el pasado (LANG-xxx) no reaparezcan.
* El contrato “CharSequence in / String out” exigido por la librería se respete
  (véase testStringUtilsCharSequenceContract).

El código está escrito con JUnit 5 e incluye pruebas parametrizadas, de límites,
de comportamiento con Unicode suplementario y de compatibilidad binaria.

2. Dependencias clave
---------------------
* JUnit 5 (org.junit.jupiter).  
* JUnit Pioneer (lectura/escritura del Locale por prueba).  
* Apache Commons Lang 3.x (clases bajo prueba).  
* Commons Lang helpers: ArrayUtils, CharUtils, WordUtils, Suppliers, MutableInt.  

3. Datos de prueba (constantes)
-------------------------------
El fichero define una gran cantidad de fixtures reutilizables, entre los que
destacan:

* WHITESPACE / NON_WHITESPACE → Todos los caracteres que cumplen –o no–  
  Character.isWhitespace.
* Arrays con valores primitivos y objetos (ARRAY_LIST, INT_PRIM_LIST, etc.)
  usados en las pruebas de join().
* Cadenas de ejemplo pre-capitalizadas, sentencias con/ sin mayúsculas, etc.
* SEPARATOR y variantes para los tests de división y concatenación.

Disponer de estas constantes evita duplicación y asegura la cobertura de casos
especiales (espacios duros, caracteres nulos, pares sustitutos, …).

4. Métodos utilitarios privados
-------------------------------
• innerTestSplit(...)  
• innerTestSplitPreserveAllTokens(...)

Ambos encapsulan la lógica repetitiva necesaria para probar
StringUtils.split(…) y splitPreserveAllTokens(…) con distintas combinaciones
de separadores, límites y caracteres “no-match”.

5. Organización de los métodos de prueba
----------------------------------------
El nombre de cada @Test refleja exactamente el bloque funcional que cubre.
Los métodos se pueden agrupar de la siguiente forma:

A. Manipulación de sufijos/prefijos  
   – testAppendIfMissing(), testPrependIfMissing() y sus variantes ignoreCase.

B. Capitalización y cambios de caso  
   – testCapitalize(), testReCapitalize(), testSwapCase(), testLowerCase(),  
     testUpperCase(), testToRootLowerCase(), testToRootUpperCase().

C. Relleno y alineación  
   – testCenter_*(), testLeftPad_*(), testRightPad_*().

D. Valores por defecto y utilidades “isBlank/Empty”  
   – testDefault*, testGetIfBlank/Empty(), testIsAllLowerCase/UpperCase/MixedCase.

E. Métricas de similitud  
   – testGetLevenshteinDistance*, testGetJaroWinklerDistance*,  
     testGetFuzzyDistance*.

F. Comparación, diferencia y prefijos comunes  
   – testDifference_*, testDifferenceAt_*, testGetCommonPrefix().

G. Unión y división de cadenas  
   – testJoin_*(), testSplit_*, testSplitByCharacterType*,  
     testSplitByWholeSeparator*(), etc.

H. Reemplazo, eliminación y regex  
   – testReplace*, testRemove*, testStrip/LANG-related issues.

I. Truncado, rotación, padding variable  
   – testTruncate_*, testRotate_*(), testRepeat_*(), testOverlay_*().

J. Conversión de bytes/códigos unicode  
   – testGetBytes_*(), testToEncodedString(), testToString(), testToCodePoints().

K. Validación de contratos de diseño  
   – testConstructor() asegura que StringUtils no sea final y que su
     constructor sea público;  
   – testStringUtilsCharSequenceContract() valida la coherencia de tipos
     (String vs CharSequence) mediante reflexión.

6. Cobertura de incidencias históricas
--------------------------------------
Cada vez que se resuelve un bug en Commons Lang, se añade una prueba con la
referencia LANG-nnn (p.e. testLANG666(), testEscapeSurrogatePairsLang858()).
Esto ofrece regresión automática ante futuras modificaciones.

7. Ejecución
------------
Las pruebas se ejecutan con cualquier runner de JUnit 5.  
Algunas utilizan @DefaultLocale, @WritesDefaultLocale o @Disabled para aislar
condiciones de entorno concretas.

8. Conclusión
-------------
StringUtilsTest constituye la principal salvaguarda de la estabilidad de
StringUtils.  Cubre:

* Casuísticas con caracteres BMP y no-BMP.
* Parámetros nulos, vacíos y valores extremos.
* Comportamiento sensible a Locale y a la plataforma (encodings).

Mantener esta clase actualizada es esencial cuando se añadan nuevas API o se
modifique la semántica existente en Apache Commons Lang.