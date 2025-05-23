# Documentación Técnica: ExternalServiceValidationException

## Descripción General
`ExternalServiceValidationException` es una clase de excepción personalizada que extiende la clase base `Exception` de Apex. Esta excepción está diseñada específicamente para representar errores de validación que ocurren durante la interacción con servicios externos.

## Estructura de la Clase

```apex
public class ExternalServiceValidationException extends Exception {
}
```

## Características

### Herencia
La clase `ExternalServiceValidationException` hereda de la clase base `Exception` de Apex, lo que significa que hereda todos los métodos y propiedades estándar de las excepciones de Apex.

### Accesibilidad
- **Modificador de Acceso**: `public`
- **Descripción**: La excepción es accesible desde cualquier contexto dentro de la organización.

## Funcionalidad

Esta excepción personalizada proporciona una forma específica de indicar errores relacionados con la validación de servicios externos. Al utilizar una clase de excepción dedicada, el código que interactúa con servicios externos puede capturar y manejar específicamente los errores de validación, separándolos de otros tipos de excepciones.

## Métodos Heredados

Al extender la clase base `Exception`, esta clase hereda los siguientes métodos importantes:

### `getMessage()`
- **Retorno**: `String`
- **Descripción**: Devuelve el mensaje de error asociado con la excepción.

### `getTypeName()`
- **Retorno**: `String`
- **Descripción**: Devuelve el nombre completo de la clase de excepción.

### `getCause()`
- **Retorno**: `Exception`
- **Descripción**: Devuelve la excepción causante, si existe.

### `getLineNumber()`
- **Retorno**: `Integer`
- **Descripción**: Devuelve el número de línea donde se produjo la excepción.

### `getStackTraceString()`
- **Retorno**: `String`
- **Descripción**: Devuelve la traza de la pila como una cadena.

## Constructores Implícitos

Aunque no están explícitamente definidos en el código, la clase hereda los siguientes constructores de la clase base `Exception`:

### `ExternalServiceValidationException()`
- **Descripción**: Constructor por defecto que crea una instancia de la excepción sin mensaje.

### `ExternalServiceValidationException(String message)`
- **Descripción**: Constructor que crea una instancia de la excepción con un mensaje específico.
- **Parámetros**:
    - `message` (String): Mensaje descriptivo de la excepción.

### `ExternalServiceValidationException(Exception cause)`
- **Descripción**: Constructor que crea una instancia de la excepción con una excepción causante.
- **Parámetros**:
    - `cause` (Exception): Excepción causante.

### `ExternalServiceValidationException(String message, Exception cause)`
- **Descripción**: Constructor que crea una instancia de la excepción con un mensaje y una excepción causante.
- **Parámetros**:
    - `message` (String): Mensaje descriptivo de la excepción.
    - `cause` (Exception): Excepción causante.

## Ejemplo de Uso

```apex
public class ExternalServiceConnector {
    
    /**
     * @description Valida y procesa datos con un servicio externo
     * @param data Los datos a procesar
     * @return Los datos procesados
     * @throws ExternalServiceValidationException Si los datos no pasan la validación
     */
    public Map<String, Object> processWithExternalService(Map<String, Object> data) {
        // Validar los datos antes de enviarlos al servicio externo
        if (!isValidForExternalService(data)) {
            throw new ExternalServiceValidationException(
                'Los datos proporcionados no cumplen con los requisitos del servicio externo');
        }
        
        try {
            // Código para interactuar con el servicio externo
            HttpRequest req = new HttpRequest();
            req.setEndpoint('https://api.ejemplo.com/procesar');
            req.setMethod('POST');
            req.setHeader('Content-Type', 'application/json');
            req.setBody(JSON.serialize(data));
            
            Http http = new Http();
            HttpResponse res = http.send(req);
            
            // Validar la respuesta
            if (res.getStatusCode() != 200) {
                Map<String, Object> errorResponse = (Map<String, Object>)JSON.deserializeUntyped(res.getBody());
                throw new ExternalServiceValidationException(
                    'Error del servicio externo: ' + errorResponse.get('error_message'));
            }
            
            return (Map<String, Object>)JSON.deserializeUntyped(res.getBody());
        } catch (Exception e) {
            if (e instanceof ExternalServiceValidationException) {
                throw e; // Relanzar excepciones de validación
            }
            throw new ExternalServiceValidationException(
                'Error al procesar con el servicio externo: ' + e.getMessage(), e);
        }
    }
    
    /**
     * @description Valida si los datos cumplen con los requisitos del servicio externo
     * @param data Los datos a validar
     * @return true si los datos son válidos, false en caso contrario
     */
    private Boolean isValidForExternalService(Map<String, Object> data) {
        // Implementación de la lógica de validación
        if (data == null || data.isEmpty()) {
            return false;
        }
        
        // Verificar campos requeridos
        String[] requiredFields = new String[] {'id', 'nombre', 'tipo'};
        for (String field : requiredFields) {
            if (!data.containsKey(field) || data.get(field) == null) {
                return false;
            }
        }
        
        return true;
    }
}
```

## Manejo de la Excepción

```apex
try {
    Map<String, Object> data = new Map<String, Object> {
        'id' => '12345',
        'nombre' => 'Ejemplo',
        // Falta el campo 'tipo'
    };
    
    ExternalServiceConnector connector = new ExternalServiceConnector();
    Map<String, Object> result = connector.processWithExternalService(data);
    
    // Procesar el resultado exitoso
    System.debug('Procesamiento exitoso: ' + result);
} catch (ExternalServiceValidationException e) {
    // Manejar específicamente los errores de validación
    System.debug('Error de validación: ' + e.getMessage());
    
    // Registrar el error
    LoggerUtility.logError('ExternalServiceValidation', e.getMessage(), e.getStackTraceString());
    
    // Notificar al usuario con un mensaje amigable
    ApexPages.addMessage(new ApexPages.Message(
        ApexPages.Severity.ERROR,
        'No se pudo completar la operación debido a un problema de validación con el servicio externo. ' +
        'Por favor, verifique los datos proporcionados.'));
} catch (Exception e) {
    // Manejar otros tipos de excepciones
    System.debug('Error general: ' + e.getMessage());
    LoggerUtility.logError('ExternalServiceGeneral', e.getMessage(), e.getStackTraceString());
}
```

## Buenas Prácticas

1. **Mensajes descriptivos**: Al lanzar una `ExternalServiceValidationException`, proporcione mensajes descriptivos que ayuden a identificar la causa exacta del error de validación.

2. **Captura específica**: Capture específicamente esta excepción cuando sea relevante para el contexto de validación de servicios externos.

3. **Jerarquía de excepciones**: Considere crear una jerarquía de excepciones más detallada si necesita distinguir entre diferentes tipos de errores de validación de servicios externos.

4. **Encapsular excepciones originales**: Cuando sea apropiado, encapsule la excepción original como causa al lanzar una `ExternalServiceValidationException`, para mantener la información completa del error.

5. **Registro adecuado**: Asegúrese de registrar adecuadamente las excepciones para facilitar la depuración y el análisis de problemas.

## Notas Adicionales

1. Esta excepción personalizada ayuda a mejorar la legibilidad y mantenibilidad del código al proporcionar un tipo específico para los errores de validación de servicios externos.

2. Aunque la implementación actual es mínima, la clase podría ampliarse en el futuro para incluir propiedades adicionales específicas de validación de servicios externos, como códigos de error, detalles de campo, o información sobre el servicio externo específico.

3. En una arquitectura compleja que interactúe con múltiples servicios externos, podría ser beneficioso extender esta excepción base para crear excepciones más específicas para cada servicio.