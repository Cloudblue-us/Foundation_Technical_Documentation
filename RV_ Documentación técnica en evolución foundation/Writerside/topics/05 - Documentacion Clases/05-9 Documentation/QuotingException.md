# QuotingException

## Descripción General
`QuotingException` es una clase de excepción personalizada que extiende la clase `Exception` estándar de Apex. Esta clase está diseñada para representar errores específicos que ocurren durante los procesos de cotización en el sistema.

## Autor
Soulberto Lorenzo \<soulberto@cloudblue.us\>

## Última Modificación
16 de agosto de 2024

## Estructura
```apex
public class QuotingException extends Exception {

}
```

La clase no define métodos o propiedades adicionales más allá de los heredados de la clase base `Exception`.

## Herencia
- **Clase Base**: `Exception`
- **Paquete**: Estándar de Salesforce

## Propósito
El propósito principal de esta clase es proporcionar un tipo de excepción específico para errores relacionados con el proceso de cotización, permitiendo:

1. **Categorización de Errores**: Distinguir errores de cotización de otros tipos de excepciones en el sistema.
2. **Manejo Específico**: Permitir que el código que captura excepciones pueda identificar y manejar específicamente errores del proceso de cotización.
3. **Claridad en el Código**: Mejorar la legibilidad del código al indicar explícitamente cuando un error está relacionado con el proceso de cotización.
4. **Seguimiento Específico**: Facilitar el seguimiento y análisis de errores específicos del proceso de cotización.

## Métodos Heredados
Al extender la clase `Exception` estándar, `QuotingException` hereda los siguientes métodos y propiedades:

### Constructores
- **`QuotingException()`**: Constructor por defecto.
- **`QuotingException(String message)`**: Constructor que acepta un mensaje de error.
- **`QuotingException(Exception cause)`**: Constructor que acepta otra excepción como causa.
- **`QuotingException(String message, Exception cause)`**: Constructor que acepta un mensaje y otra excepción como causa.

### Métodos
- **`getMessage()`**: Devuelve el mensaje de error asociado con la excepción.
- **`getCause()`**: Devuelve la excepción que causó esta excepción, si existe.
- **`getTypeName()`**: Devuelve el nombre del tipo de excepción.
- **`setMessage(String message)`**: Establece el mensaje de error para la excepción.
- **`getLineNumber()`**: Devuelve el número de línea donde se generó la excepción.
- **`getStackTraceString()`**: Devuelve una representación de texto del seguimiento de la pila.

## Uso en el Sistema
Esta excepción está diseñada para ser utilizada por clases que participan en el proceso de cotización, tales como:

1. **`QuoteProductHandler`**: Puede lanzar esta excepción cuando hay problemas específicos en el proceso de cotización.
2. **`QuoteUtils`**: Puede lanzar esta excepción cuando hay problemas en la validación o manipulación de cotizaciones.
3. **Clases de validación de datos**: Pueden lanzar esta excepción cuando los datos de entrada para una cotización no cumplen con los requisitos.
4. **Procedimientos de integración**: Pueden lanzar esta excepción cuando hay problemas al interactuar con sistemas externos durante el proceso de cotización.

## Ejemplo de Lanzamiento

```apex
public Map<String, Object> calculatePremium(Map<String, Object> quoteData) {
    try {
        // Validar datos de entrada
        if (!quoteData.containsKey('insuredItems') || quoteData.get('insuredItems') == null) {
            throw new QuotingException('Los elementos asegurados son requeridos para calcular la prima');
        }
        
        List<Object> insuredItems = (List<Object>)quoteData.get('insuredItems');
        if (insuredItems.isEmpty()) {
            throw new QuotingException('Debe proporcionar al menos un elemento asegurado');
        }
        
        // Calcular la prima basada en los elementos asegurados
        Decimal totalPremium = 0;
        for (Object item : insuredItems) {
            Map<String, Object> insuredItem = (Map<String, Object>)item;
            
            if (!insuredItem.containsKey('value') || insuredItem.get('value') == null) {
                throw new QuotingException('Cada elemento asegurado debe tener un valor especificado');
            }
            
            Decimal itemValue = (Decimal)insuredItem.get('value');
            Decimal itemPremium = calculateItemPremium(itemValue, insuredItem);
            totalPremium += itemPremium;
        }
        
        // Crear respuesta
        return new Map<String, Object>{
            'totalPremium' => totalPremium,
            'currency' => 'USD',
            'calculationDate' => System.now()
        };
    } catch (QuotingException qe) {
        // Relanzar la excepción específica de cotización
        throw qe;
    } catch (Exception e) {
        // Convertir excepciones genéricas en QuotingException
        throw new QuotingException('Error al calcular la prima: ' + e.getMessage(), e);
    }
}

private Decimal calculateItemPremium(Decimal itemValue, Map<String, Object> itemData) {
    // Lógica de cálculo de prima para un elemento específico
    // ...
    
    // Si hay algún problema específico del cálculo
    if (itemValue <= 0) {
        throw new QuotingException('El valor del elemento asegurado debe ser mayor que cero');
    }
    
    return itemValue * 0.05; // Ejemplo simplificado
}
```

## Ejemplo de Captura

```apex
try {
    // Obtener datos de cotización
    Map<String, Object> quoteData = getQuoteDataFromRequest();
    
    // Calcular la prima
    Map<String, Object> premiumResult = calculatePremium(quoteData);
    
    // Procesar el resultado
    processQuoteResult(premiumResult);
} catch (QuotingException qe) {
    // Manejo específico para errores de cotización
    System.debug('Error en el proceso de cotización: ' + qe.getMessage());
    
    // Registrar el error específico de cotización
    Quoting_Error__c errorRecord = new Quoting_Error__c(
        Error_Message__c = qe.getMessage(),
        Stack_Trace__c = qe.getStackTraceString(),
        Timestamp__c = Datetime.now(),
        Quote_Data__c = JSON.serialize(quoteData)
    );
    insert errorRecord;
    
    // Devolver una respuesta de error específica para cotizaciones
    return createQuotingErrorResponse(qe.getMessage());
} catch (Exception e) {
    // Manejo genérico para otros tipos de excepciones
    System.debug('Error general: ' + e.getMessage());
    
    // Devolver una respuesta de error genérica
    return createGenericErrorResponse(e.getMessage());
}
```

## Extensión Personalizada

Aunque la implementación actual es minimalista, la clase puede ser extendida para incluir información adicional específica del proceso de cotización:

```apex
public class EnhancedQuotingException extends QuotingException {
    private String errorCode;
    private String quoteId;
    private String productCode;
    private Decimal attemptedPremium;
    
    public EnhancedQuotingException(String message) {
        super(message);
    }
    
    public EnhancedQuotingException(String message, String errorCode) {
        super(message);
        this.errorCode = errorCode;
    }
    
    public EnhancedQuotingException(
        String message, 
        String errorCode, 
        String quoteId, 
        String productCode
    ) {
        super(message);
        this.errorCode = errorCode;
        this.quoteId = quoteId;
        this.productCode = productCode;
    }
    
    public EnhancedQuotingException(
        String message, 
        String errorCode, 
        String quoteId, 
        String productCode, 
        Decimal attemptedPremium,
        Exception cause
    ) {
        super(message, cause);
        this.errorCode = errorCode;
        this.quoteId = quoteId;
        this.productCode = productCode;
        this.attemptedPremium = attemptedPremium;
    }
    
    public String getErrorCode() {
        return this.errorCode;
    }
    
    public String getQuoteId() {
        return this.quoteId;
    }
    
    public String getProductCode() {
        return this.productCode;
    }
    
    public Decimal getAttemptedPremium() {
        return this.attemptedPremium;
    }
}
```

## Mejores Prácticas

1. **Mensajes Descriptivos**: Incluir información específica en los mensajes de error, como campos faltantes, valores inválidos o condiciones no cumplidas.
2. **Captura Selectiva**: Capturar esta excepción específicamente cuando se quiera manejar errores de cotización de manera diferenciada.
3. **Información de Contexto**: Al lanzar la excepción, proporcionar información de contexto que pueda ayudar a diagnosticar el problema, como los datos de entrada que causaron el error.
4. **Encadenamiento de Excepciones**: Utilizar el constructor que acepta una excepción causa para preservar el seguimiento de la pila original cuando se convierten excepciones genéricas en `QuotingException`.
5. **Registro Adecuado**: Registrar las excepciones capturadas para su posterior análisis y solución de problemas, posiblemente en un objeto personalizado para errores de cotización.

## Puntos a Considerar

1. **Granularidad**: Considerar si es necesario tener subtipos más específicos de `QuotingException` para diferentes aspectos del proceso de cotización (por ejemplo, `QuoteValidationException`, `PremiumCalculationException`, etc.).
2. **Códigos de Error**: Implementar un sistema de códigos de error para categorizar diferentes tipos de errores de cotización de manera más estructurada.
3. **Información Contextual**: Evaluar la necesidad de incluir información contextual adicional en la excepción, como se muestra en el ejemplo de extensión personalizada.
4. **Integración con Sistemas de Monitoreo**: Asegurar que estas excepciones se integren adecuadamente con los sistemas de monitoreo y alerta existentes.

## Integración con Otras Clases

### Relación con QuoteProductHandler

La clase `QuoteProductHandler` podría utilizar `QuotingException` para manejar errores específicos del proceso de cotización:

```apex
// Ejemplo de uso en QuoteProductHandler
public override Map<String, Object> process(
  Map<String, Object> options,
  Map<String, Object> input,
  Map<String, Object> output
) {
  try {
    // Validar datos de entrada
    Map<Boolean, Object> validation = QuoteUtils.validateInputMapForQuoting(input);
    if (validation.containsKey(false)) {
      throw new QuotingException(validation.get(false).toString());
    }
    
    // Procesar la cotización
    // ...
    
    return output;
  } catch (QuotingException qe) {
    // Manejar o relanzar la excepción específica de cotización
    throw qe;
  } catch (Exception e) {
    // Convertir otras excepciones en QuotingException
    throw new QuotingException('Error en el proceso de cotización: ' + e.getMessage(), e);
  }
}
```

### Relación con QuoteUtils

La clase `QuoteUtils` podría lanzar `QuotingException` en sus métodos de validación:

```apex
// Ejemplo de cómo QuoteUtils podría lanzar QuotingException
public static void validateQuoteData(Map<String, Object> quoteData) {
  if (!quoteData.containsKey(MAIN_NODE) || quoteData.get(MAIN_NODE) == null) {
    throw new QuotingException('Falta el nodo principal requerido: ' + MAIN_NODE);
  }
  
  Map<String, Object> mainNode = (Map<String, Object>)quoteData.get(MAIN_NODE);
  
  for (String requiredParam : REQUIRED_PARAMETERS) {
    if (!mainNode.containsKey(requiredParam) || mainNode.get(requiredParam) == null) {
      throw new QuotingException('Falta el parámetro requerido: ' + requiredParam);
    }
  }
}
```

## Notas Adicionales
1. Esta clase forma parte del ecosistema de cotización de la organización y se utiliza en conjunto con otras clases como `QuoteProductHandler` y `QuoteUtils`.
2. La simplicidad de la implementación sugiere que la clase se utiliza principalmente para categorización de errores más que para transportar información adicional específica.
3. En un entorno de producción, considerar la implementación de mecanismos de registro y notificación automatizados para errores de cotización.
4. Esta excepción podría ser parte de una jerarquía más amplia de excepciones específicas del dominio, posiblemente con una superclase común como `DomainException` o similar.