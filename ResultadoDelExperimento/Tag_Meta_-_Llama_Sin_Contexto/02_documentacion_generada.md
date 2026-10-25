### Documentación Técnica - Anotación `@Tag`

**Proyecto:** JUnit 5 - Jupiter API
**Paquete:** `org.junit.jupiter.api`
**Tipo:** Anotación
**Archivo:** `Tag.java`
**Versión:** Desde 5.0
**Estado API:** `STABLE`

#### 1. Descripción General

`@Tag` es una anotación de tipo `Repeatable` perteneciente a JUnit Jupiter que se utiliza para etiquetar clases y métodos de prueba con fines de filtrado y organización.

No es una clase en sentido estricto, sino una `annotation interface` que actúa como metadato. No contiene lógica de negocio, su función es declarar una o varias etiquetas asociadas a un test para que el motor de ejecución de JUnit Platform pueda incluir o excluir tests durante la creación del `TestPlan`.

Permite a los equipos de desarrollo categorizar las pruebas bajo criterios como: `fast`, `slow`, `integration`, `unit`, `ci-server`, `smoke`, etc., y ejecutar subconjuntos de pruebas según el entorno o la necesidad.

Es heredable y repetible, por lo que una misma clase o método puede tener múltiples tags.

#### 2. Definición de la Anotación

```java
@Target({ ElementType.TYPE, ElementType.METHOD })
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@Repeatable(Tags.class)
@API(status = STABLE, since = "5.0")
public @interface Tag {
    String value();
}
```

**Meta-anotaciones:**

| Meta-anotación | Descripción |
| :--- | :--- |
| `@Target({TYPE, METHOD})` | Indica que solo puede aplicarse a nivel de clase o de método. No puede usarse en campos, parámetros o paquetes. |
| `@Retention(RUNTIME)` | La anotación se conserva en tiempo de ejecución, lo que permite que sea leída por reflexión por el motor de JUnit. |
| `@Documented` | Indica que la anotación debe incluirse en el Javadoc generado. |
| `@Inherited` | Si una clase anotada con `@Tag` es extendida, la subclase hereda automáticamente el tag. |
| `@Repeatable(Tags.class)` | Permite aplicar múltiples anotaciones `@Tag` sobre el mismo elemento. Internamente se agrupan en la anotación contenedora `@Tags`. |
| `@API(status = STABLE)` | Garantiza que la API es estable y no tendrá cambios incompatibles sin previo aviso. |

#### 3. Métodos Principales

La anotación expone un único miembro abstracto obligatorio:

| Método | Tipo de Retorno | Valor por Defecto | Descripción |
| :--- | :--- | :--- | :--- |
| `value()` | `String` | *Ninguno (obligatorio)* | Define el nombre de la etiqueta. Es el único atributo de la anotación. El valor es sometido a un proceso de `strip()` para eliminar espacios al inicio y al final. Si después del recorte el tag resulta sintácticamente inválido, no se lanza una excepción, sino que se registra un `warning` en el log y el tag es ignorado. |

> **Nota:** Al ser una anotación, no posee métodos con lógica implementada, constructores ni atributos de instancia. Su único propósito es almacenar el valor del tag.

#### 4. Reglas de Sintaxis para Tags

Para que un tag sea considerado válido, debe cumplir las siguientes reglas. Un tag inválido será ignorado:

1.  No debe estar vacío o en blanco.
2.  Una vez recortado con `strip()`, no debe contener espacios en blanco.
3.  No debe contener caracteres de control ISO.
4.  No debe contener ninguno de los siguientes caracteres reservados, utilizados para expresiones de filtrado:
    * `,` - coma
    * `(` - paréntesis izquierdo
    * `)` - paréntesis derecho
    * `&` - ampersand
    * `|` - barra vertical
    * `!` - signo de exclamación

#### 5. Ejemplo de Uso

```java
import org.junit.jupiter.api.Tag;
import org.junit.jupiter.api.Test;

@Tag("regression")
@Tag("fast")
class UsuarioServiceTest {

    @Test
    @Tag("unit")
    void testCreacionUsuario() {
        // test rápido y unitario
    }

    @Test
    @Tag("slow")
    @Tag("integration")
    void testIntegracionBaseDeDatos() {
        // test lento de integración
    }
}
```

**Filtrado en la ejecución:**

El filtrado se realiza a nivel de configuración del build tool o del IDE mediante expresiones de tags.

* **Maven Surefire:** `mvn test -Dgroups="fast & unit" -DexcludedGroups="slow"`
* **Gradle:** `useJUnitPlatform { includeTags 'fast'; excludeTags 'slow' }`

#### 6. Anotaciones Relacionadas

* `@Tags`: Anotación contenedora generada automáticamente por el compilador para almacenar múltiples `@Tag` repetidas.
* `@Test`: Anotación principal que marca un método como caso de prueba, comúnmente combinada con `@Tag`.

