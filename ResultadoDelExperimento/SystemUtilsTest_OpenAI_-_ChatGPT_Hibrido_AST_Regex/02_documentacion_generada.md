# SystemUtilsTest — Documentación Técnica

## 1. Resumen ejecutivo
`SystemUtilsTest` es una batería de pruebas JUnit 5 que valida exhaustivamente la clase utilitaria `SystemUtils` de Apache Commons Lang3.  
Su objetivo es garantizar la correcta detección de versiones de Java, sistemas operativos y propiedades del entorno en tiempo de ejecución, así como la resolución de rutas de archivos y directorios relacionados con el usuario y la JVM.

---

## 2. Especificación de métodos y constructores

> Convención de tablas:  
> • **Obligatorio** = “Sí” cuando el parámetro no puede ser `null`.  
> • Los métodos de prueba no retornan datos de negocio; su valor de retorno es `void`.

### 2.1 Constructores

| Nombre | Descripción |
|--------|-------------|
| `SystemUtilsTest()` | Constructor por defecto; instancia la clase de pruebas. |

### 2.2 Métodos

#### `getIS_JAVA(int version)`

| @param | Tipo | Obligatorio | Descripción |
|--------|------|-------------|-------------|
| version | `int` | Sí | Número mayor o igual que 9 que identifica la versión de Java (`IS_JAVA_<version>`). |

| @return | Descripción |
|---------|-------------|
| `boolean` | `true` si el campo `SystemUtils.IS_JAVA_<version>` es `true`; `false` en caso contrario. |

| @throws | Condición |
|---------|-----------|
| `Exception` | Cualquier error de reflexión al acceder dinámicamente al campo. |

---

#### `getLastSupportedJavaVersion()`

Sin parámetros.

| @return | Descripción |
|---------|-------------|
| `int` | Última versión de Java para la que existe un campo `SystemUtils.IS_JAVA_<X>`. Retorna `0` si no se halla ninguno. |

---

#### `test_IS_JAVA()`

Método de prueba JUnit.

| @throws | Condición |
|---------|-----------|
| `Exception` | Propagada internamente por llamadas a `getIS_JAVA(int)`. |

---

#### `test_IS_OS()`, `test_IS_zOS()`, `test_USER_NAME()`, `testConstructor()`, `testGetEnvironmentVariableAbsent()`, `testGetEnvironmentVariablePresent()`, `testGetHostName()`, `testGetJavaHome()`, `testGetJavaHomePath()`, `testGetJavaIoTmpDir()`, `testGetJavaIoTmpDirPath()`, `testGetUserDir()`, `testGetUserDirPath()`, `testGetUserHome()`, `testGetUserHomePath()`, `testGetUserName()`, `testIsJavaVersionAtLeast()`, `testIsJavaVersionAtMost()`, `testJavaAwtHeadless()`, `testJavaVersionMatches()`, `testOSMatchesName()`, `testOSMatchesNameAndVersion()`, `testOsVersionMatches()`

Todos los métodos listados comparten la misma firma:

| @param | Tipo | Obligatorio | Descripción |
|--------|------|-------------|-------------|
| _N/A_  |      |             | No reciben parámetros. |

| @return | Descripción |
|---------|-------------|
| `void` | Ejecutan aserciones JUnit; no devuelven valor. |

| @throws | Condición |
|---------|-----------|
| — | No declaran excepciones checked. (`AssertionError` puede lanzarse en fallo de prueba). |

---

## 3. Ejemplo de uso

El siguiente fragmento muestra cómo invocar directamente la lógica de utilidades usada por las pruebas; **no** se recomienda ejecutar métodos de prueba fuera del contexto de JUnit, pero es útil para ilustrar la API privada/pública del propio test.

```java
public class SystemUtilsDiagnostics {

    public static void main(String[] args) throws Exception {
        SystemUtilsTest test = new SystemUtilsTest();

        // Obtener la última versión soportada por los flags IS_JAVA_X
        int last = test.getLastSupportedJavaVersion();
        System.out.printf("Última versión de Java soportada por SystemUtils: %d%n", last);

        // Consultar si la JVM actual es, por ejemplo, Java 17
        boolean is17 = test.getIS_JAVA(17);
        System.out.printf("¿Corre sobre Java 17? %s%n", is17);

        // Nota: Los métodos prefijados con 'test' están pensados para JUnit.
        // No deberían invocarse manualmente.
    }
}
```

---

@author Apache Commons contributors  
@since 3.x