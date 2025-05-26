# CumuloMissingParameterException

## Descripción General
`CumuloMissingParameterException` es una clase de excepción personalizada que extiende la clase `Exception` estándar de Apex. Esta clase está diseñada específicamente para representar errores que ocurren cuando faltan parámetros requeridos en operaciones relacionadas con el framework Cumulo o procesos específicos que requieren validación de parámetros obligatorios.

## Autor
Felipe Correa \<felipe.correa@nespon.com\>

## Proyecto
Foundation Salesforce - Suratech - UH-17589

## Última Modificación
21 de noviembre de 2024

## Estructura

```apex
public with sharing class CumuloMissingParameterException extends Exception {
}
```

La clase tiene una implementación minimalista, sin agregar métodos o propiedades adicionales más allá de los heredados de la clase base `Exception`.

## Herencia
- **Clase Base**: `Exception`
- **Modificadores**: `public with sharing`

## Propósito
El propósito principal de esta clase es proporcionar un tipo de excepción específico para errores relacionados con parámetros faltantes en el contexto del framework Cumulo, permitiendo:

1. **Especificidad del Error**: Distinguir claramente los errores de parámetros faltantes de otros tipos de excepciones en el sistema.
2. **Validación de Entrada**: Facilitar la validación de parámetros requeridos en métodos y procesos críticos.
3. **Manejo Específico**: Permitir que el código que captura excepciones pueda identificar y manejar específicamente casos donde faltan parámetros obligatorios.
4. **Claridad en el Código**: Mejorar la legibilidad del código al indicar explícitamente cuando se está tratando con un error de parámetro faltante.
5. **Debugging y Mantenimiento**: Facilitar la identificación de problemas relacionados con configuración o llamadas incorrectas a métodos.

## Contexto del Framework Cumulo
El nombre "Cumulo" sugiere que esta excepción forma parte de un framework o sistema específico dentro de la organización Suratech. Los frameworks suelen requerir configuraciones específicas y parámetros obligatorios para funcionar correctamente, por lo que tener una excepción específica para parámetros faltantes es una práctica común y recomendada.

## Métodos Heredados
Al extender la clase `Exception` estándar, `CumuloMissingParameterException` hereda los siguientes métodos y propiedades:

### Constructores
- **`CumuloMissingParameterException()`**: Constructor por defecto.
- **`CumuloMissingParameterException(String message)`**: Constructor que acepta un mensaje de error.
- **`CumuloMissingParameterException(Exception cause)`**: Constructor que acepta otra excepción como causa.
- **`CumuloMissingParameterException(String message, Exception cause)`**: Constructor que acepta un mensaje y otra excepción como causa.

### Métodos
- **`getMessage()`**: Devuelve el mensaje de error asociado con la excepción.
- **`getCause()`**: Devuelve la excepción que causó esta excepción, si existe.
- **`getTypeName()`**: Devuelve el nombre del tipo de excepción.
- **`setMessage(String message)`**: Establece el mensaje de error para la excepción.
- **`getLineNumber()`**: Devuelve el número de línea donde se generó la excepción.
- **`getStackTraceString()`**: Devuelve una representación de texto del seguimiento de la pila.

## Ejemplos de Uso

### Ejemplo 1: Validación de Parámetros de Configuración

```apex
public class CumuloConfigurationManager {
    
    public static void initializeFramework(Map<String, Object> config) {
        // Validar parámetros requeridos
        validateRequiredParameters(config);
        
        // Proceder con la inicialización
        String endpoint = (String) config.get('endpoint');
        String apiKey = (String) config.get('apiKey');
        Integer timeout = (Integer) config.get('timeout');
        
        // Lógica de inicialización del framework
        // ...
    }
    
    private static void validateRequiredParameters(Map<String, Object> config) {
        if (config == null) {
            throw new CumuloMissingParameterException('La configuración no puede ser nula');
        }
        
        List<String> requiredParams = new List<String>{'endpoint', 'apiKey', 'timeout'};
        
        for (String param : requiredParams) {
            if (!config.containsKey(param) || config.get(param) == null) {
                throw new CumuloMissingParameterException(
                    'Parámetro requerido faltante: ' + param
                );
            }
            
            if (config.get(param) instanceof String && String.isBlank((String)config.get(param))) {
                throw new CumuloMissingParameterException(
                    'El parámetro ' + param + ' no puede estar vacío'
                );
            }
        }
    }
}

// Uso
try {
    Map<String, Object> config = new Map<String, Object>{
        'endpoint' => 'https://api.cumulo.com',
        'apiKey' => 'sk_test_123456'
        // Falta el parámetro 'timeout'
    };
    
    CumuloConfigurationManager.initializeFramework(config);
} catch (CumuloMissingParameterException e) {
    System.debug('Error de configuración: ' + e.getMessage());
    // Manejar el error específico de parámetro faltante
}
```

### Ejemplo 2: Validación en Métodos de Servicio

```apex
public class CumuloDataService {
    
    public static Map<String, Object> processData(
        String recordId, 
        String operation, 
        Map<String, Object> data
    ) {
        // Validar parámetros de entrada
        validateInputParameters(recordId, operation, data);
        
        // Procesar los datos
        Map<String, Object> result = new Map<String, Object>();
        
        switch on operation.toUpperCase() {
            when 'CREATE' {
                result = createRecord(data);
            }
            when 'UPDATE' {
                result = updateRecord(recordId, data);
            }
            when 'DELETE' {
                result = deleteRecord(recordId);
            }
            when else {
                throw new IllegalArgumentException('Operación no soportada: ' + operation);
            }
        }
        
        return result;
    }
    
    private static void validateInputParameters(
        String recordId, 
        String operation, 
        Map<String, Object> data
    ) {
        if (String.isBlank(operation)) {
            throw new CumuloMissingParameterException('El parámetro "operation" es requerido');
        }
        
        Set<String> operationsRequiringId = new Set<String>{'UPDATE', 'DELETE'};
        if (operationsRequiringId.contains(operation.toUpperCase()) && String.isBlank(recordId)) {
            throw new CumuloMissingParameterException(
                'El parámetro "recordId" es requerido para la operación ' + operation
            );
        }
        
        Set<String> operationsRequiringData = new Set<String>{'CREATE', 'UPDATE'};
        if (operationsRequiringData.contains(operation.toUpperCase())) {
            if (data == null || data.isEmpty()) {
                throw new CumuloMissingParameterException(
                    'El parámetro "data" es requerido para la operación ' + operation
                );
            }
        }
    }
    
    private static Map<String, Object> createRecord(Map<String, Object> data) {
        // Lógica de creación
        return new Map<String, Object>{'success' => true, 'id' => 'new_record_id'};
    }
    
    private static Map<String, Object> updateRecord(String recordId, Map<String, Object> data) {
        // Lógica de actualización
        return new Map<String, Object>{'success' => true, 'id' => recordId};
    }
    
    private static Map<String, Object> deleteRecord(String recordId) {
        // Lógica de eliminación
        return new Map<String, Object>{'success' => true, 'deleted' => true};
    }
}

// Uso
try {
    // Esto lanzará CumuloMissingParameterException porque falta recordId para UPDATE
    CumuloDataService.processData(null, 'UPDATE', new Map<String, Object>{'name' => 'Test'});
} catch (CumuloMissingParameterException e) {
    System.debug('Error: ' + e.getMessage());
    // Manejar error de parámetro faltante
}
```

### Ejemplo 3: Validación en Procedimientos de Integración

```apex
public class CumuloIntegrationProcedure {
    
    public static Map<String, Object> executeIntegration(Map<String, Object> input) {
        try {
            // Validar estructura de entrada
            validateIntegrationInput(input);
            
            // Extraer parámetros validados
            String integrationPoint = (String) input.get('integrationPoint');
            Map<String, Object> payload = (Map<String, Object>) input.get('payload');
            Map<String, Object> headers = (Map<String, Object>) input.get('headers');
            
            // Ejecutar la integración
            return performIntegration(integrationPoint, payload, headers);
            
        } catch (CumuloMissingParameterException e) {
            // Devolver error estructurado
            return new Map<String, Object>{
                'success' => false,
                'errorType' => 'MISSING_PARAMETER',
                'errorMessage' => e.getMessage()
            };
        } catch (Exception e) {
            // Manejar otros errores
            return new Map<String, Object>{
                'success' => false,
                'errorType' => 'INTEGRATION_ERROR',
                'errorMessage' => e.getMessage()
            };
        }
    }
    
    private static void validateIntegrationInput(Map<String, Object> input) {
        if (input == null) {
            throw new CumuloMissingParameterException('Los datos de entrada no pueden ser nulos');
        }
        
        // Validar integrationPoint
        if (!input.containsKey('integrationPoint') || 
            String.isBlank((String)input.get('integrationPoint'))) {
            throw new CumuloMissingParameterException(
                'El parámetro "integrationPoint" es requerido'
            );
        }
        
        // Validar payload
        if (!input.containsKey('payload') || input.get('payload') == null) {
            throw new CumuloMissingParameterException(
                'El parámetro "payload" es requerido'
            );
        }
        
        // Validar que el payload sea un mapa
        if (!(input.get('payload') instanceof Map<String, Object>)) {
            throw new CumuloMissingParameterException(
                'El parámetro "payload" debe ser un objeto válido'
            );
        }
        
        // Headers es opcional, pero si está presente debe ser un mapa
        if (input.containsKey('headers') && input.get('headers') != null) {
            if (!(input.get('headers') instanceof Map<String, Object>)) {
                throw new CumuloMissingParameterException(
                    'El parámetro "headers" debe ser un objeto válido si se proporciona'
                );
            }
        }
    }
    
    private static Map<String, Object> performIntegration(
        String integrationPoint, 
        Map<String, Object> payload, 
        Map<String, Object> headers
    ) {
        // Lógica de integración
        return new Map<String, Object>{
            'success' => true,
            'integrationPoint' => integrationPoint,
            'processedAt' => System.now()
        };
    }
}
```

### Ejemplo 4: Validación con Información Detallada

```apex
public class CumuloValidator {
    
    public static void validateBusinessObject(Map<String, Object> businessObject, String objectType) {
        if (String.isBlank(objectType)) {
            throw new CumuloMissingParameterException('El tipo de objeto es requerido');
        }
        
        // Definir campos requeridos por tipo de objeto
        Map<String, List<String>> requiredFieldsByType = new Map<String, List<String>>{
            'CUSTOMER' => new List<String>{'name', 'email', 'phone'},
            'PRODUCT' => new List<String>{'name', 'sku', 'price'},
            'ORDER' => new List<String>{'customerId', 'productIds', 'totalAmount'}
        };
        
        if (!requiredFieldsByType.containsKey(objectType.toUpperCase())) {
            throw new IllegalArgumentException('Tipo de objeto no soportado: ' + objectType);
        }
        
        List<String> requiredFields = requiredFieldsByType.get(objectType.toUpperCase());
        List<String> missingFields = new List<String>();
        
        for (String field : requiredFields) {
            if (!businessObject.containsKey(field) || 
                businessObject.get(field) == null ||
                (businessObject.get(field) instanceof String && 
                 String.isBlank((String)businessObject.get(field)))) {
                missingFields.add(field);
            }
        }
        
        if (!missingFields.isEmpty()) {
            throw new CumuloMissingParameterException(
                'Faltan los siguientes campos requeridos para ' + objectType + ': ' + 
                String.join(missingFields, ', ')
            );
        }
    }
}

// Uso
try {
    Map<String, Object> customer = new Map<String, Object>{
        'name' => 'Juan Pérez',
        'email' => 'juan@example.com'
        // Falta 'phone'
    };
    
    CumuloValidator.validateBusinessObject(customer, 'CUSTOMER');
} catch (CumuloMissingParameterException e) {
    System.debug('Error de validación: ' + e.getMessage());
    // Resultado: "Faltan los siguientes campos requeridos para CUSTOMER: phone"
}
```

## Implementación Extendida Propuesta

Aunque la implementación actual es minimalista, podría extenderse para incluir información más específica:

```apex
/**
 * @description       : Excepción para parámetros faltantes en el framework Cumulo
 * @author            : felipe.correa@nespon.com
 * @project           : Foundation Salesforce - Suratech - UH-17589
 * @last modified on  : 11-21-2024
 * @last modified by  : felipe.correa@nespon.com
 **/
public with sharing class CumuloMissingParameterException extends Exception {
    
    private String parameterName;
    private String parameterType;
    private String contextMethod;
    private List<String> missingParameters;
    
    /**
     * Constructor por defecto
     */
    public CumuloMissingParameterException() {
        super();
    }
    
    /**
     * Constructor con mensaje de error
     * @param message Mensaje descriptivo del error
     */
    public CumuloMissingParameterException(String message) {
        super(message);
    }
    
    /**
     * Constructor con información específica del parámetro
     * @param message Mensaje descriptivo del error
     * @param parameterName Nombre del parámetro faltante
     * @param contextMethod Método donde ocurrió el error
     */
    public CumuloMissingParameterException(String message, String parameterName, String contextMethod) {
        super(message);
        this.parameterName = parameterName;
        this.contextMethod = contextMethod;
    }
    
    /**
     * Constructor para múltiples parámetros faltantes
     * @param message Mensaje descriptivo del error
     * @param missingParameters Lista de parámetros faltantes
     * @param contextMethod Método donde ocurrió el error
     */
    public CumuloMissingParameterException(
        String message, 
        List<String> missingParameters, 
        String contextMethod
    ) {
        super(message);
        this.missingParameters = missingParameters;
        this.contextMethod = contextMethod;
    }
    
    /**
     * Constructor con excepción causa
     * @param message Mensaje descriptivo del error
     * @param cause Excepción que causó esta excepción
     */
    public CumuloMissingParameterException(String message, Exception cause) {
        super(message, cause);
    }
    
    /**
     * Obtiene el nombre del parámetro faltante
     * @return Nombre del parámetro
     */
    public String getParameterName() {
        return this.parameterName;
    }
    
    /**
     * Obtiene el tipo del parámetro faltante
     * @return Tipo del parámetro
     */
    public String getParameterType() {
        return this.parameterType;
    }
    
    /**
     * Obtiene el método donde ocurrió el error
     * @return Nombre del método
     */
    public String getContextMethod() {
        return this.contextMethod;
    }
    
    /**
     * Obtiene la lista de parámetros faltantes
     * @return Lista de parámetros faltantes
     */
    public List<String> getMissingParameters() {
        return this.missingParameters != null ? this.missingParameters : new List<String>();
    }
    
    /**
     * Genera un mensaje completo con toda la información disponible
     * @return Mensaje completo
     */
    public String getDetailedMessage() {
        String detailedMessage = getMessage();
        
        if (String.isNotBlank(contextMethod)) {
            detailedMessage += '\nMétodo: ' + contextMethod;
        }
        
        if (String.isNotBlank(parameterName)) {
            detailedMessage += '\nParámetro faltante: ' + parameterName;
        }
        
        if (missingParameters != null && !missingParameters.isEmpty()) {
            detailedMessage += '\nParámetros faltantes: ' + String.join(missingParameters, ', ');
        }
        
        return detailedMessage;
    }
}
```

## Mejores Prácticas para su Uso

1. **Mensajes Específicos**: Proporcionar mensajes de error claros que indiquen exactamente qué parámetro falta y en qué contexto.

2. **Validación Temprana**: Utilizar esta excepción al inicio de los métodos para validar parámetros de entrada.

3. **Agrupación de Validaciones**: Cuando sea posible, validar todos los parámetros requeridos de una vez y reportar todos los faltantes.

4. **Documentación de Parámetros**: Asegurar que la documentación de los métodos indique claramente qué parámetros son requeridos.

5. **Manejo Diferenciado**: Capturar y manejar esta excepción específicamente para proporcionar mensajes de error más útiles a los usuarios.

## Consideraciones de Diseño

### Ventajas de la Implementación Actual
1. **Simplicidad**: La implementación minimalista es fácil de entender y utilizar.
2. **Enfoque Específico**: Se centra exclusivamente en representar errores de parámetros faltantes.
3. **Respeto a la Seguridad**: El modificador `with sharing` asegura que se respeten las reglas de compartición.

### Posibles Mejoras
1. **Información Detallada**: Podría extenderse para incluir información más específica sobre los parámetros faltantes.
2. **Métodos de Utilidad**: Podrían añadirse métodos estáticos para facilitar la validación común de parámetros.
3. **Integración con Logging**: Podría incluir funcionalidad para registrar automáticamente los errores de parámetros faltantes.

## Integración con Pruebas Unitarias

```apex
@IsTest
private class CumuloMissingParameterExceptionTest {
    
    @IsTest
    static void testBasicException() {
        String expectedMessage = 'Parámetro requerido faltante: testParam';
        
        try {
            throw new CumuloMissingParameterException(expectedMessage);
        } catch (CumuloMissingParameterException e) {
            System.assertEquals(expectedMessage, e.getMessage());
            System.assertEquals('CumuloMissingParameterException', e.getTypeName());
        }
    }
    
    @IsTest
    static void testValidationMethod() {
        Map<String, Object> incompleteConfig = new Map<String, Object>{
            'endpoint' => 'https://test.com'
            // Falta apiKey
        };
        
        try {
            CumuloConfigurationManager.initializeFramework(incompleteConfig);
            System.assert(false, 'Debería haber lanzado CumuloMissingParameterException');
        } catch (CumuloMissingParameterException e) {
            System.assert(e.getMessage().contains('apiKey'));
        }
    }
}
```

## Notas Adicionales
1. El nombre "Cumulo" sugiere que forma parte de un framework específico dentro de la organización Suratech.
2. El modificador `with sharing` indica que respeta las reglas de compartición de Salesforce, lo cual es apropiado para validaciones que pueden involucrar datos sensibles.
3. Esta excepción complementa otras excepciones del sistema como `ProductHandlerImplementationException`, proporcionando un ecosistema completo de manejo de errores específicos.
4. La referencia al proyecto "UH-17589" indica que forma parte de un desarrollo específico y estructurado dentro del framework Foundation Salesforce.
5. Es una práctica excelente tener excepciones específicas para diferentes tipos de errores de validación, mejorando tanto la experiencia del desarrollador como la capacidad de depuración del sistema.