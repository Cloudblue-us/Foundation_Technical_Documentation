# ExternalServiceException

## Descripción General
`ExternalServiceException` es una clase de excepción personalizada que extiende la clase `Exception` estándar de Apex. Esta clase está diseñada para representar errores específicos que ocurren durante la interacción con servicios externos.

## Estructura
```apex
public class ExternalServiceException extends Exception {

}
```

La clase no define métodos o propiedades adicionales más allá de los heredados de la clase base `Exception`.

## Herencia
- **Clase Base**: `Exception`
- **Paquete**: Estándar de Salesforce

## Propósito
El propósito principal de esta clase es proporcionar un tipo de excepción específico para errores relacionados con servicios externos, permitiendo:

1. **Categorización de Errores**: Distinguir errores de servicios externos de otros tipos de excepciones en el sistema.
2. **Manejo Específico**: Permitir que el código que captura excepciones pueda identificar y manejar específicamente errores de servicios externos.
3. **Claridad en el Código**: Mejorar la legibilidad del código al indicar explícitamente cuando un error está relacionado con un servicio externo.

## Métodos Heredados
Al extender la clase `Exception` estándar, `ExternalServiceException` hereda los siguientes métodos y propiedades:

### Constructores
- **`ExternalServiceException()`**: Constructor por defecto.
- **`ExternalServiceException(String message)`**: Constructor que acepta un mensaje de error.
- **`ExternalServiceException(Exception cause)`**: Constructor que acepta otra excepción como causa.
- **`ExternalServiceException(String message, Exception cause)`**: Constructor que acepta un mensaje y otra excepción como causa.

### Métodos
- **`getMessage()`**: Devuelve el mensaje de error asociado con la excepción.
- **`getCause()`**: Devuelve la excepción que causó esta excepción, si existe.
- **`getTypeName()`**: Devuelve el nombre del tipo de excepción.
- **`setMessage(String message)`**: Establece el mensaje de error para la excepción.
- **`getLineNumber()`**: Devuelve el número de línea donde se generó la excepción.
- **`getStackTraceString()`**: Devuelve una representación de texto del seguimiento de la pila.

## Uso en el Sistema
Esta excepción es utilizada por varias clases en el sistema que interactúan con servicios externos, tales como:

1. **`ExternalNotificationHandler`**: La utiliza para envolver excepciones específicas durante las llamadas a servicios externos.
2. **`RUNTServiceHandler`**: La lanza cuando hay problemas para obtener la configuración del servicio RUNT.
3. **Otras clases de integración**: Cualquier clase que realice llamadas a servicios web, APIs externas o sistemas integrados.

## Ejemplo de Lanzamiento

```apex
try {
    // Intentar realizar una operación con un servicio externo
    HttpRequest req = new HttpRequest();
    req.setEndpoint('https://api.example.com/resource');
    req.setMethod('GET');
    
    Http http = new Http();
    HttpResponse res = http.send(req);
    
    if (res.getStatusCode() != 200) {
        throw new ExternalServiceException(
            'Error en el servicio externo. Código: ' + res.getStatusCode() + 
            '. Respuesta: ' + res.getBody()
        );
    }
    
    // Procesar la respuesta exitosa
} catch (CalloutException e) {
    throw new ExternalServiceException(
        'Error de comunicación con el servicio externo: ' + e.getMessage(), 
        e
    );
} catch (Exception e) {
    // Manejar otras excepciones
    throw new ExternalServiceException(
        'Error inesperado al interactuar con el servicio externo: ' + e.getMessage(), 
        e
    );
}
```

## Ejemplo de Captura

```apex
try {
    // Llamar a un método que podría lanzar diferentes tipos de excepciones
    processExternalServiceRequest();
} catch (ExternalServiceException e) {
    // Manejo específico para errores de servicios externos
    System.debug('Error de servicio externo: ' + e.getMessage());
    
    // Registrar el error en un objeto personalizado
    External_Service_Error__c errorRecord = new External_Service_Error__c(
        Error_Message__c = e.getMessage(),
        Stack_Trace__c = e.getStackTraceString(),
        Timestamp__c = Datetime.now()
    );
    insert errorRecord;
    
    // Notificar al administrador
    sendErrorNotification('Error de servicio externo', e.getMessage());
} catch (Exception e) {
    // Manejo genérico para otros tipos de excepciones
    System.debug('Error general: ' + e.getMessage());
}
```

## Extensión Personalizada

Aunque la implementación actual es minimalista, la clase puede ser extendida para incluir información adicional específica de servicios externos:

```apex
public class EnhancedExternalServiceException extends ExternalServiceException {
    private Integer statusCode;
    private String endpoint;
    private String requestBody;
    private String responseBody;
    
    public EnhancedExternalServiceException(String message) {
        super(message);
    }
    
    public EnhancedExternalServiceException(String message, Integer statusCode, String endpoint) {
        super(message);
        this.statusCode = statusCode;
        this.endpoint = endpoint;
    }
    
    public EnhancedExternalServiceException(
        String message, 
        Integer statusCode, 
        String endpoint, 
        String requestBody, 
        String responseBody
    ) {
        super(message);
        this.statusCode = statusCode;
        this.endpoint = endpoint;
        this.requestBody = requestBody;
        this.responseBody = responseBody;
    }
    
    public Integer getStatusCode() {
        return this.statusCode;
    }
    
    public String getEndpoint() {
        return this.endpoint;
    }
    
    public String getRequestBody() {
        return this.requestBody;
    }
    
    public String getResponseBody() {
        return this.responseBody;
    }
}
```

## Mejores Prácticas

1. **Mensajes Descriptivos**: Incluir información específica en los mensajes de error, como códigos de estado, endpoints y respuestas.
2. **Captura Selectiva**: Capturar esta excepción específicamente cuando se quiera manejar errores de servicios externos de manera diferenciada.
3. **Información de Contexto**: Al lanzar la excepción, proporcionar información de contexto que pueda ayudar a diagnosticar el problema.
4. **Encadenamiento de Excepciones**: Utilizar el constructor que acepta una excepción causa para preservar el seguimiento de la pila original.
5. **Registro Adecuado**: Registrar las excepciones capturadas para su posterior análisis y solución de problemas.

## Puntos a Considerar

1. **Simplicidad vs. Especialización**: La implementación actual es muy simple, lo que la hace flexible pero menos especializada. Considerar si se necesita una versión más especializada con campos adicionales.
2. **Seguridad**: Tener cuidado con la información sensible que se incluye en los mensajes de excepción, especialmente si estos se muestran a los usuarios finales.
3. **Rendimiento**: La creación de objetos de excepción tiene un costo, por lo que deben utilizarse apropiadamente y no como mecanismo de control de flujo normal.

## Notas Adicionales
1. Esta clase forma parte del ecosistema de integración de la organización y se utiliza en conjunto con otras clases como `ExternalNotificationHandler` y `RUNTServiceHandler`.
2. La simplicidad de la implementación sugiere que la clase se utiliza principalmente para categorización de errores más que para transportar información adicional específica.
3. En un entorno de producción, considerar la implementación de mecanismos de registro y notificación automatizados para errores de servicios externos.