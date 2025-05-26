# Documentación Técnica: IntegrationProcedureExecutor

## Descripción General
`IntegrationProcedureExecutor` es una interfaz global que define el contrato para la ejecución de procedimientos de integración. Esta interfaz establece un método estándar para invocar procedimientos de integración con parámetros de entrada y obtener resultados estructurados.

## Autor
Jean Carlos Melendez

## Última Modificación
28 de enero de 2025

## Definición de la Interfaz

```apex
global interface IntegrationProcedureExecutor {
  Map<String, Object> execute(String procedureName, Map<String, Object> input);
}
```

## Métodos

### `execute(String procedureName, Map<String, Object> input)`

#### Descripción
Este método ejecuta un procedimiento de integración especificado, pasando los parámetros de entrada proporcionados y devolviendo los resultados del procedimiento.

#### Parámetros
- `procedureName` (String): Nombre del procedimiento de integración a ejecutar.
- `input` (Map<String, Object>): Mapa con los parámetros de entrada para el procedimiento.

#### Retorno
- `Map<String, Object>`: Mapa con los resultados del procedimiento de integración.

## Propósito y Aplicaciones

El propósito principal de esta interfaz es proporcionar una abstracción para la ejecución de procedimientos de integración, ofreciendo:

1. **Desacoplamiento**: Separar el código que necesita invocar procedimientos de integración de la implementación específica de cómo se ejecutan estos procedimientos.
2. **Flexibilidad**: Permitir múltiples implementaciones para diferentes contextos (por ejemplo, producción, pruebas, mock).
3. **Consistencia**: Establecer un contrato estándar para la ejecución de procedimientos de integración en toda la aplicación.
4. **Testabilidad**: Facilitar la creación de mocks para pruebas unitarias.

Esta interfaz es especialmente útil en escenarios como:

- **Sistemas de Integración**: Facilitar la comunicación con sistemas externos a través de procedimientos de integración estandarizados.
- **Orquestación de Procesos**: Coordinar diferentes pasos en flujos de trabajo complejos que pueden involucrar múltiples sistemas.
- **Abstracción de Detalles Técnicos**: Ocultar los detalles específicos de cómo se ejecutan los procedimientos de integración a los componentes que los utilizan.
- **Cambio de Implementaciones**: Permitir cambiar la forma en que se ejecutan los procedimientos de integración sin afectar a los componentes que los invocan.

## Ejemplos de Implementación

### Implementación para OmniScript/Integration Procedures de Vlocity

```apex
global class VlocityIntegrationProcedureExecutor implements IntegrationProcedureExecutor {
    
    global Map<String, Object> execute(String procedureName, Map<String, Object> input) {
        try {
            // Inicializar el resultado
            Map<String, Object> result = new Map<String, Object>();
            
            // Verificar parámetros
            if (String.isBlank(procedureName)) {
                throw new IntegrationProcedureException('El nombre del procedimiento no puede estar vacío');
            }
            
            if (input == null) {
                input = new Map<String, Object>();
            }
            
            // Crear la petición para el procedimiento de integración de Vlocity
            Map<String, Object> request = new Map<String, Object>{
                'procedureName' => procedureName,
                'input' => input
            };
            
            // Invocar el procedimiento de integración a través de la API de Vlocity
            Map<String, Object> response = (Map<String, Object>) vlocity_ins.IntegrationProcedureService.runIntegrationProcedureQueueable(
                procedureName, 
                input
            );
            
            // Verificar la respuesta
            if (response == null) {
                throw new IntegrationProcedureException('No se recibió respuesta del procedimiento de integración');
            }
            
            return response;
        } catch (Exception e) {
            System.debug(LoggingLevel.ERROR, 'Error ejecutando procedimiento de integración: ' + procedureName + '. Error: ' + e.getMessage());
            throw new IntegrationProcedureException('Error ejecutando procedimiento: ' + e.getMessage(), e);
        }
    }
    
    // Clase de excepción personalizada
    global class IntegrationProcedureException extends Exception {}
}
```

### Implementación a través de Remote Actions

```apex
global class RemoteActionIntegrationProcedureExecutor implements IntegrationProcedureExecutor {
    
    global Map<String, Object> execute(String procedureName, Map<String, Object> input) {
        try {
            // Validar parámetros
            if (String.isBlank(procedureName)) {
                throw new IntegrationProcedureException('El nombre del procedimiento no puede estar vacío');
            }
            
            if (input == null) {
                input = new Map<String, Object>();
            }
            
            // Preparar parámetros para Remote Action
            String namespace = 'vlocity_ins';
            String classname = 'IntegrationProcedureService';
            String methodName = 'invokeIntegrationProcedure';
            String options = '{}';
            
            // Convertir el mapa de entrada a JSON
            String inputJson = JSON.serialize(input);
            
            // Invocar Remote Action
            Object result = vlocity_ins.RemoteActionController.invokeMethod(
                namespace, 
                classname, 
                methodName, 
                procedureName, 
                inputJson, 
                options
            );
            
            // Procesar resultado
            if (result == null) {
                return new Map<String, Object>();
            }
            
            // Convertir el resultado a un mapa
            Map<String, Object> resultMap;
            if (result instanceof String) {
                resultMap = (Map<String, Object>) JSON.deserializeUntyped((String) result);
            } else if (result instanceof Map<String, Object>) {
                resultMap = (Map<String, Object>) result;
            } else {
                throw new IntegrationProcedureException('Resultado inesperado del tipo: ' + result.getClass().getName());
            }
            
            return resultMap;
        } catch (Exception e) {
            System.debug(LoggingLevel.ERROR, 'Error ejecutando procedimiento de integración: ' + procedureName + '. Error: ' + e.getMessage());
            throw new IntegrationProcedureException('Error ejecutando procedimiento: ' + e.getMessage(), e);
        }
    }
    
    // Clase de excepción personalizada
    global class IntegrationProcedureException extends Exception {}
}
```

### Implementación Mock para Pruebas

```apex
@IsTest
global class MockIntegrationProcedureExecutor implements IntegrationProcedureExecutor {
    
    // Mapa para almacenar respuestas mockificadas por nombre de procedimiento
    private static Map<String, Map<String, Object>> mockResponses = new Map<String, Map<String, Object>>();
    
    // Bandera para simular errores
    private static Boolean throwError = false;
    private static String errorMessage = 'Error simulado en procedimiento de integración';
    
    global Map<String, Object> execute(String procedureName, Map<String, Object> input) {
        // Registrar la llamada para verificación en las pruebas
        recordExecution(procedureName, input);
        
        // Simular error si está configurado
        if (throwError) {
            throw new IntegrationProcedureException(errorMessage);
        }
        
        // Devolver respuesta mockificada si existe
        if (mockResponses.containsKey(procedureName)) {
            return mockResponses.get(procedureName);
        }
        
        // Respuesta por defecto
        return new Map<String, Object>{
            'success' => true,
            'procedureName' => procedureName,
            'mockInput' => input,
            'result' => 'Mock result for ' + procedureName
        };
    }
    
    // Métodos para configurar el comportamiento del mock
    
    global static void setMockResponse(String procedureName, Map<String, Object> response) {
        mockResponses.put(procedureName, response);
    }
    
    global static void clearMockResponses() {
        mockResponses.clear();
    }
    
    global static void setThrowError(Boolean shouldThrow, String message) {
        throwError = shouldThrow;
        if (String.isNotBlank(message)) {
            errorMessage = message;
        }
    }
    
    // Registro de ejecuciones para verificación en pruebas
    private static List<ExecutionRecord> executionRecords = new List<ExecutionRecord>();
    
    private void recordExecution(String procedureName, Map<String, Object> input) {
        executionRecords.add(new ExecutionRecord(procedureName, input));
    }
    
    global static List<ExecutionRecord> getExecutionRecords() {
        return executionRecords;
    }
    
    global static void clearExecutionRecords() {
        executionRecords.clear();
    }
    
    // Clase interna para registrar ejecuciones
    global class ExecutionRecord {
        public String procedureName;
        public Map<String, Object> input;
        
        public ExecutionRecord(String procedureName, Map<String, Object> input) {
            this.procedureName = procedureName;
            this.input = input;
        }
    }
    
    // Clase de excepción personalizada
    global class IntegrationProcedureException extends Exception {}
}
```

## Ejemplo de Uso

### Uso Básico en un Servicio

```apex
public class QuotingService {
    
    private final IntegrationProcedureExecutor ipExecutor;
    
    // Constructor con inyección de dependencia
    public QuotingService(IntegrationProcedureExecutor ipExecutor) {
        this.ipExecutor = ipExecutor;
    }
    
    // Constructor sin parámetros para uso general
    public QuotingService() {
        this.ipExecutor = new VlocityIntegrationProcedureExecutor();
    }
    
    // Método para calcular prima de un producto
    public Map<String, Object> calculatePremium(String productCode, Map<String, Object> customerData) {
        try {
            // Preparar los datos de entrada
            Map<String, Object> input = new Map<String, Object>{
                'productCode' => productCode,
                'customerData' => customerData,
                'calculationDate' => System.now()
            };
            
            // Ejecutar el procedimiento de integración
            Map<String, Object> result = ipExecutor.execute('CalculatePremium', input);
            
            // Procesar y validar resultados
            if (!result.containsKey('premium')) {
                throw new QuotingException('La respuesta del cálculo de prima no contiene el campo "premium"');
            }
            
            return result;
        } catch (Exception e) {
            throw new QuotingException('Error al calcular prima: ' + e.getMessage(), e);
        }
    }
    
    // Clase de excepción personalizada
    public class QuotingException extends Exception {}
}
```

### Uso con Factory para Obtener la Implementación Adecuada

```apex
public class IntegrationProcedureExecutorFactory {
    
    // Singleton instance
    private static IntegrationProcedureExecutorFactory instance;
    
    // Obtener instancia singleton
    public static IntegrationProcedureExecutorFactory getInstance() {
        if (instance == null) {
            instance = new IntegrationProcedureExecutorFactory();
        }
        return instance;
    }
    
    // Obtener el ejecutor adecuado basado en configuración o contexto
    public IntegrationProcedureExecutor getExecutor() {
        // En un entorno de prueba, usar el mock
        if (Test.isRunningTest()) {
            return new MockIntegrationProcedureExecutor();
        }
        
        // Verificar si se debe usar una implementación alternativa
        SF_IntegrationConfig__mdt config = SF_IntegrationConfig__mdt.getInstance('IPExecutorConfig');
        if (config != null && String.isNotBlank(config.AlternativeImplementation__c)) {
            return (IntegrationProcedureExecutor)Type.forName(config.AlternativeImplementation__c).newInstance();
        }
        
        // Por defecto, usar la implementación estándar
        return new VlocityIntegrationProcedureExecutor();
    }
}

// Uso con factory
public class RatingService {
    
    public Map<String, Object> rateProduct(String productId, Map<String, Object> ratingFactors) {
        IntegrationProcedureExecutor executor = IntegrationProcedureExecutorFactory.getInstance().getExecutor();
        
        Map<String, Object> input = new Map<String, Object>{
            'productId' => productId,
            'ratingFactors' => ratingFactors
        };
        
        return executor.execute('RateProduct', input);
    }
}
```

### Uso en Pruebas Unitarias

```apex
@IsTest
private class QuotingServiceTest {
    
    @IsTest
    static void testCalculatePremium() {
        // Configurar el mock
        MockIntegrationProcedureExecutor mockExecutor = new MockIntegrationProcedureExecutor();
        
        // Configurar respuesta esperada
        Map<String, Object> mockResponse = new Map<String, Object>{
            'premium' => 1200.50,
            'discounts' => new Map<String, Object>{
                'goodDriver' => 100.00,
                'multiPolicy' => 50.00
            },
            'totalPremium' => 1050.50,
            'currency' => 'USD'
        };
        
        MockIntegrationProcedureExecutor.setMockResponse('CalculatePremium', mockResponse);
        
        // Crear instancia del servicio con el mock
        QuotingService service = new QuotingService(mockExecutor);
        
        // Preparar datos de prueba
        String productCode = 'AUTO-001';
        Map<String, Object> customerData = new Map<String, Object>{
            'age' => 35,
            'vehicleValue' => 25000,
            'drivingHistory' => 'Clean'
        };
        
        // Ejecutar el método a probar
        Test.startTest();
        Map<String, Object> result = service.calculatePremium(productCode, customerData);
        Test.stopTest();
        
        // Verificar resultados
        System.assertEquals(1200.50, result.get('premium'), 'Premium should match mock value');
        System.assertEquals(1050.50, result.get('totalPremium'), 'Total premium should match mock value');
        System.assertEquals('USD', result.get('currency'), 'Currency should match mock value');
        
        // Verificar que se llamó al procedimiento correcto con los parámetros esperados
        List<MockIntegrationProcedureExecutor.ExecutionRecord> executions = MockIntegrationProcedureExecutor.getExecutionRecords();
        System.assertEquals(1, executions.size(), 'Should have executed exactly one procedure');
        System.assertEquals('CalculatePremium', executions[0].procedureName, 'Should have called the correct procedure');
        System.assertEquals(productCode, executions[0].input.get('productCode'), 'Should have passed the correct product code');
    }
    
    @IsTest
    static void testCalculatePremiumError() {
        // Configurar el mock para lanzar error
        MockIntegrationProcedureExecutor mockExecutor = new MockIntegrationProcedureExecutor();
        MockIntegrationProcedureExecutor.setThrowError(true, 'Error de cálculo simulado');
        
        // Crear instancia del servicio con el mock
        QuotingService service = new QuotingService(mockExecutor);
        
        // Preparar datos de prueba
        String productCode = 'AUTO-001';
        Map<String, Object> customerData = new Map<String, Object>{
            'age' => 35,
            'vehicleValue' => 25000
        };
        
        // Ejecutar el método y verificar que se lance la excepción esperada
        Test.startTest();
        try {
            service.calculatePremium(productCode, customerData);
            System.assert(false, 'Should have thrown an exception');
        } catch (QuotingService.QuotingException e) {
            System.assert(e.getMessage().contains('Error de cálculo simulado'), 'Exception should contain the mock error message');
        }
        Test.stopTest();
    }
}
```

## Consideraciones de Diseño

### Ventajas
1. **Desacoplamiento**: Los componentes que necesitan ejecutar procedimientos de integración no dependen de una implementación específica.
2. **Flexibilidad**: Se pueden crear diferentes implementaciones para diferentes contextos o requisitos.
3. **Testabilidad**: Facilita la creación de mocks para pruebas unitarias.
4. **Consistencia**: Proporciona un contrato estándar para la ejecución de procedimientos de integración.

### Desafíos
1. **Manejo de Errores**: Las implementaciones deben proporcionar un manejo coherente de errores para mantener un comportamiento predecible.
2. **Tipado Dinámico**: El uso de `Map<String, Object>` proporciona flexibilidad pero requiere verificaciones adicionales de tipo en tiempo de ejecución.
3. **Sincronización**: La interfaz actual es síncrona, lo que podría no ser adecuado para procedimientos de larga duración.

## Patrones de Diseño Relacionados

### Inyección de Dependencias
La interfaz `IntegrationProcedureExecutor` facilita la inyección de dependencias, permitiendo que los componentes reciban la implementación específica a través de constructores o setters.

### Estrategia (Strategy)
Permite definir una familia de algoritmos (diferentes formas de ejecutar procedimientos de integración), encapsularlos y hacerlos intercambiables.

### Fábrica (Factory)
Puede utilizarse junto con un patrón de fábrica para proporcionar la implementación adecuada basada en el contexto.

### Adaptador (Adapter)
Algunas implementaciones pueden actuar como adaptadores, permitiendo que diferentes sistemas de procedimientos de integración se utilicen de manera uniforme.

## Relación con Otras Interfaces y Clases

Esta interfaz está relacionada y es utilizada por varias clases en el sistema, como:

1. **QuoteProductHandler**: Utiliza un `IntegrationProcedureExecutor` para ejecutar procedimientos relacionados con cotizaciones.
2. **IssueProductHandler**: Emplea un ejecutor para gestionar la emisión de productos.
3. **MockIntegrationProcedureExecutor**: Implementación de prueba que simula la ejecución de procedimientos de integración.

## Mejores Prácticas

1. **Manejo Adecuado de Errores**: Implementar un manejo robusto de errores en todas las implementaciones.
2. **Registro de Actividad**: Incluir registro de actividad para facilitar el seguimiento y la depuración.
3. **Validación de Parámetros**: Validar los parámetros de entrada para evitar errores inesperados.
4. **Implementaciones con Caché**: Considerar la implementación de mecanismos de caché para procedimientos frecuentemente utilizados.
5. **Timeout y Reintentos**: Implementar mecanismos de timeout y reintentos para manejar situaciones de fallo transitorio.

## Notas Adicionales
1. Esta interfaz está marcada como `global`, lo que indica que está diseñada para ser accesible desde diferentes paquetes o namespaces.
2. El uso de mapas genéricos (`Map<String, Object>`) proporciona flexibilidad pero requiere un manejo cuidadoso de los tipos en tiempo de ejecución.
3. Esta interfaz forma parte de un sistema más amplio para la integración y orquestación de procesos, trabajando en conjunto con otras interfaces y clases.
4. Para procedimientos de larga duración, podría ser necesario considerar una variante asíncrona de esta interfaz.