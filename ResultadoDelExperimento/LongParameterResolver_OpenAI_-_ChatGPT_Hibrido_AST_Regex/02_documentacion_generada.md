# LongParameterResolver

## 1. Resumen ejecutivo  
`LongParameterResolver` es una implementación de `org.junit.jupiter.api.extension.ParameterResolver` que inyecta siempre el valor `42L` (tipo `Long`) en los parámetros de métodos de prueba.  
El resolver admite tanto parámetros declarados explícitamente como de tipo genérico siempre que el tipo efectivo sea compatible con `java.lang.Long`.

---

## 2. Especificación de constructores y métodos

### 2.1. LongParameterResolver()  
Constructor por defecto.

| @param | Tipo | Obligatorio | Descripción |
|--------|------|-------------|-------------|
| *N/A*  | —    | —           | No recibe parámetros. |

| @return | Descripción |
|---------|-------------|
| —       | Instancia de `LongParameterResolver`. |

| @throws | Condición |
|---------|-----------|
| —       | No declara excepciones. |

---

### 2.2. boolean supportsParameter(ParameterContext parameterContext, ExtensionContext extensionContext)

| @param | Tipo | Obligatorio | Descripción |
|--------|------|-------------|-------------|
| parameterContext | `ParameterContext` | Sí | Contexto que describe el parámetro del método de prueba. |
| extensionContext | `ExtensionContext` | Sí | Contexto de ejecución de la extensión. |

| @return | Descripción |
|---------|-------------|
| `boolean` | `true` si el parámetro es de tipo `Long` o un tipo genérico resoluble a `Long`; en caso contrario, `false`. |

| @throws | Condición |
|---------|-----------|
| —       | No declara excepciones. |

---

### 2.3. Long resolveParameter(ParameterContext parameterContext, ExtensionContext extensionContext)

| @param | Tipo | Obligatorio | Descripción |
|--------|------|-------------|-------------|
| parameterContext | `ParameterContext` | Sí | Contexto que describe el parámetro del método de prueba. |
| extensionContext | `ExtensionContext` | Sí | Contexto de ejecución de la extensión. |

| @return | Descripción |
|---------|-------------|
| `Long`  | El valor constante `42L`. |

| @throws | Condición |
|---------|-----------|
| —       | No declara excepciones. |

---

## 3. Ejemplo de uso

```java
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.engine.execution.injection.sample.LongParameterResolver;

@ExtendWith(LongParameterResolver.class)
class CalculatorTest {

    @Test
    void shouldInjectLong(Long value) {   // Se inyecta 42L
        Assertions.assertEquals(42L, value);
    }
}
```

En el ejemplo, JUnit Jupiter inyecta automáticamente el valor `42L` en el parámetro `value` del método de prueba gracias a `LongParameterResolver`.