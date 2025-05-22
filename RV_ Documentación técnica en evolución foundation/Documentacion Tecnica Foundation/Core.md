# Documentación Técnica: Core

## Descripción General
`Core` es una clase global utilitaria fundamental que proporciona funcionalidades básicas de debugging y monitoreo de límites del sistema Salesforce. Esta clase actúa como una biblioteca central de utilidades que es ampliamente utilizada en todo el framework Foundation de Suratech para registro de depuración y seguimiento del rendimiento del sistema.

## Autor
Soulberto Lorenzo \<soulberto@cloudblue.us\>

## Última Modificación
4 de febrero de 2025 por Jean Carlos Melendez

## Estructura de la Clase

```apex
global without sharing class Core {
  global static void debug(String message);
  global static String getCalloutStatistics();
  global static String getCPUTimeStatistics();
  global static String getSOQLStatistics();
  global static String getSOSLStatistics();
  global static void getSymmaryLimits();
}
```

## Características Clave
- **Modificador `without sharing`**: La clase no hereda reglas de compartición, permitiendo acceso completo a datos del sistema para funciones de monitoreo.
- **Métodos estáticos globales**: Todos los métodos son estáticos y globales, permitiendo su uso desde cualquier contexto sin necesidad de instanciación.

## Métodos

### `debug(String message)`

#### Descripción
Método estático global que proporciona funcionalidad de debugging estandarizada para todo el framework. Registra mensajes con un nivel de logging específico y un prefijo identificador.

#### Parámetros
- `message` (String): Mensaje a registrar en los logs de depuración.

#### Retorno
- `void`: Este método no devuelve ningún valor.

#### Características
- **Nivel de Logging**: Utiliza `LoggingLevel.FINEST` para registros de depuración detallados.
- **Prefijo Estándar**: Añade el prefijo "Core: " a todos los mensajes para facilitar la identificación.
- **Uso Universal**: Es utilizado por múltiples clases del framework como método central de debugging.

#### Código
```apex
global static void debug(String message) {
  System.debug(LoggingLevel.FINEST, 'Core: ' + message);
}
```

### `getCalloutStatistics()`

#### Descripción
Método que retorna estadísticas de uso de callouts HTTP, mostrando cuántos callouts se han realizado versus el límite permitido.

#### Parámetros
Ninguno.

#### Retorno
- `String`: Cadena formateada que muestra "Callouts statistics=X/Y" donde X es el número actual de callouts y Y es el límite máximo.

#### Uso
Útil para monitorear el consumo de callouts y prevenir que se alcancen los límites del sistema.

#### Código
```apex
global static String getCalloutStatistics() {
  return 'Callouts statistics=' +
    System.Limits.getCallouts() +
    '/' +
    System.Limits.getLimitCallouts();
}
```

### `getCPUTimeStatistics()`

#### Descripción
Método que retorna estadísticas de uso de tiempo de CPU, mostrando el tiempo consumido versus el límite permitido.

#### Parámetros
Ninguno.

#### Retorno
- `String`: Cadena formateada que muestra "CPU time usage=X/Y" donde X es el tiempo de CPU utilizado en milisegundos y Y es el límite máximo.

#### Uso
Crítico para monitorear el rendimiento y prevenir timeouts por exceso de procesamiento.

#### Código
```apex
global static String getCPUTimeStatistics() {
  return 'CPU time usage=' +
    System.Limits.getCpuTime() +
    '/' +
    System.Limits.getLimitCpuTime();
}
```

### `getSOQLStatistics()`

#### Descripción
Método que retorna estadísticas de uso de consultas SOQL, mostrando cuántas consultas se han ejecutado versus el límite permitido.

#### Parámetros
Ninguno.

#### Retorno
- `String`: Cadena formateada que muestra "SOQL queries usage=X/Y" donde X es el número de consultas SOQL ejecutadas y Y es el límite máximo.

#### Uso
Esencial para prevenir excepciones por exceso de consultas SOQL en una transacción.

#### Código
```apex
global static String getSOQLStatistics() {
  return 'SOQL queries usage=' +
    System.Limits.getQueries() +
    '/' +
    System.Limits.getLimitQueries();
}
```

### `getSOSLStatistics()`

#### Descripción
Método que retorna estadísticas de uso de consultas SOSL, mostrando cuántas búsquedas se han ejecutado versus el límite permitido.

#### Parámetros
Ninguno.

#### Retorno
- `String`: Cadena formateada que muestra "SOSL queries usage=X/Y" donde X es el número de consultas SOSL ejecutadas y Y es el límite máximo.

#### Uso
Útil para monitorear el uso de búsquedas SOSL y evitar alcanzar los límites del sistema.

#### Código
```apex
global static String getSOSLStatistics() {
  return 'SOSL queries usage=' +
    System.Limits.getSoslQueries() +
    '/' +
    System.Limits.getLimitSoslQueries();
}
```

### `getSymmaryLimits()`

#### Descripción
Método que registra un resumen completo de todas las estadísticas de límites del sistema de forma formateada.

**Nota**: Existe un error tipográfico en el nombre del método (debería ser `getSummaryLimits`).

#### Parámetros
Ninguno.

#### Retorno
- `void`: Este método no devuelve ningún valor, sino que registra la información.

#### Proceso
1. Obtiene estadísticas de CPU, SOQL, SOSL y Callouts.
2. Formatea toda la información en una sola cadena.
3. Registra el resumen usando el método `debug`.

#### Código
```apex
global static void getSymmaryLimits() {
  Core.debug(
    '===== Statistics(' +
      Core.getCPUTimeStatistics() +
      ',' +
      Core.getSOQLStatistics() +
      ',' +
      Core.getSOSLStatistics() +
      ',' +
      Core.getCalloutStatistics() +
      ') ====='
  );
}
```

## Límites del Sistema Salesforce Monitoreados

| Tipo de Límite | Método de Acceso | Límite Típico | Descripción |
|----------------|------------------|---------------|-------------|
| **Callouts HTTP** | `getCallouts()` | 100 por transacción | Llamadas a servicios externos |
| **Tiempo de CPU** | `getCpuTime()` | 10,000ms (síncr.) / 60,000ms (asíncr.) | Tiempo de procesamiento |
| **Consultas SOQL** | `getQueries()` | 100 por transacción | Consultas a la base de datos |
| **Consultas SOSL** | `getSoslQueries()` | 20 por transacción | Búsquedas de texto |

## Ejemplos de Uso

### Ejemplo 1: Debugging Básico

```apex
public class ExampleService {
    
    public static void processData(List<Account> accounts) {
        Core.debug('Starting data processing for ' + accounts.size() + ' accounts');
        
        try {
            for (Account acc : accounts) {
                // Procesar cada cuenta
                processAccount(acc);
                Core.debug('Processed account: ' + acc.Name);
            }
            
            Core.debug('Data processing completed successfully');
        } catch (Exception e) {
            Core.debug('Error in data processing: ' + e.getMessage());
            throw e;
        }
    }
    
    private static void processAccount(Account acc) {
        // Lógica de procesamiento
        Core.debug('Processing account with ID: ' + acc.Id);
    }
}

// Uso
List<Account> accounts = [SELECT Id, Name FROM Account LIMIT 10];
ExampleService.processData(accounts);
```

### Ejemplo 2: Monitoreo de Límites en Procesamiento por Lotes

```apex
public class BatchProcessorWithMonitoring implements Database.Batchable<SObject> {
    
    public Database.QueryLocator start(Database.BatchableContext bc) {
        Core.debug('Starting batch processing');
        Core.getSymmaryLimits(); // Registrar límites iniciales
        
        return Database.getQueryLocator('SELECT Id, Name FROM Account WHERE IsActive__c = true');
    }
    
    public void execute(Database.BatchableContext bc, List<Account> scope) {
        Core.debug('Processing batch of ' + scope.size() + ' records');
        
        // Registrar límites antes del procesamiento
        Core.debug('Limits before processing:');
        Core.debug(Core.getCPUTimeStatistics());
        Core.debug(Core.getSOQLStatistics());
        
        try {
            // Procesar registros
            for (Account acc : scope) {
                processAccount(acc);
                
                // Verificar límites periódicamente
                if (System.Limits.getCpuTime() > 8000) { // 80% del límite
                    Core.debug('CPU time approaching limit: ' + Core.getCPUTimeStatistics());
                }
            }
            
            // Actualizar registros
            update scope;
            
        } catch (Exception e) {
            Core.debug('Error in batch execution: ' + e.getMessage());
            
            // Registrar límites en caso de error
            Core.getSymmaryLimits();
            throw e;
        }
        
        // Registrar límites después del procesamiento
        Core.debug('Limits after processing:');
        Core.getSymmaryLimits();
    }
    
    public void finish(Database.BatchableContext bc) {
        Core.debug('Batch processing completed');
        Core.getSymmaryLimits(); // Registrar límites finales
    }
    
    private void processAccount(Account acc) {
        // Simular procesamiento
        acc.Description = 'Processed at ' + System.now();
    }
}

// Ejecutar el batch
Database.executeBatch(new BatchProcessorWithMonitoring(), 200);
```

### Ejemplo 3: Monitoreo en Integraciones

```apex
public class IntegrationServiceWithMonitoring {
    
    public static Map<String, Object> callExternalService(String endpoint, Map<String, Object> payload) {
        Core.debug('Starting external service call to: ' + endpoint);
        
        // Verificar límites antes de la llamada
        Core.debug('Pre-callout limits: ' + Core.getCalloutStatistics());
        
        try {
            // Configurar la solicitud HTTP
            HttpRequest request = new HttpRequest();
            request.setEndpoint(endpoint);
            request.setMethod('POST');
            request.setBody(JSON.serialize(payload));
            request.setHeader('Content-Type', 'application/json');
            
            // Realizar la llamada
            Http http = new Http();
            HttpResponse response = http.send(request);
            
            Core.debug('External service response received. Status: ' + response.getStatusCode());
            
            // Verificar límites después de la llamada
            Core.debug('Post-callout limits: ' + Core.getCalloutStatistics());
            
            // Procesar respuesta
            Map<String, Object> result = new Map<String, Object>{
                'statusCode' => response.getStatusCode(),
                'body' => response.getBody(),
                'success' => response.getStatusCode() == 200
            };
            
            return result;
            
        } catch (Exception e) {
            Core.debug('Error in external service call: ' + e.getMessage());
            
            // Registrar todos los límites en caso de error
            Core.getSymmaryLimits();
            
            throw new IntegrationException('Failed to call external service: ' + e.getMessage(), e);
        }
    }
    
    public class IntegrationException extends Exception {}
}

// Uso
Map<String, Object> payload = new Map<String, Object>{
    'action' => 'create',
    'data' => new Map<String, Object>{'name' => 'Test', 'value' => 123}
};

Map<String, Object> result = IntegrationServiceWithMonitoring.callExternalService(
    'https://api.example.com/data', 
    payload
);
```

### Ejemplo 4: Utility Class para Monitoreo Avanzado

```apex
public class PerformanceMonitor {
    
    private static Map<String, Long> startTimes = new Map<String, Long>();
    
    // Iniciar monitoreo de una operación
    public static void startOperation(String operationName) {
        startTimes.put(operationName, System.currentTimeMillis());
        Core.debug('Started operation: ' + operationName);
        Core.getSymmaryLimits();
    }
    
    // Finalizar monitoreo de una operación
    public static void endOperation(String operationName) {
        Long startTime = startTimes.get(operationName);
        if (startTime != null) {
            Long duration = System.currentTimeMillis() - startTime;
            Core.debug('Completed operation: ' + operationName + ' in ' + duration + 'ms');
            startTimes.remove(operationName);
        }
        Core.getSymmaryLimits();
    }
    
    // Verificar si estamos cerca de los límites
    public static Boolean isApproachingLimits() {
        Boolean approaching = false;
        
        // Verificar CPU (80% del límite)
        if (System.Limits.getCpuTime() > (System.Limits.getLimitCpuTime() * 0.8)) {
            Core.debug('WARNING: Approaching CPU time limit');
            approaching = true;
        }
        
        // Verificar SOQL (80% del límite)
        if (System.Limits.getQueries() > (System.Limits.getLimitQueries() * 0.8)) {
            Core.debug('WARNING: Approaching SOQL queries limit');
            approaching = true;
        }
        
        // Verificar Callouts (80% del límite)
        if (System.Limits.getCallouts() > (System.Limits.getLimitCallouts() * 0.8)) {
            Core.debug('WARNING: Approaching callouts limit');
            approaching = true;
        }
        
        return approaching;
    }
    
    // Generar reporte detallado de uso
    public static String generateUsageReport() {
        List<String> report = new List<String>();
        
        report.add('=== PERFORMANCE USAGE REPORT ===');
        report.add('CPU Time: ' + Core.getCPUTimeStatistics());
        report.add('SOQL Queries: ' + Core.getSOQLStatistics());
        report.add('SOSL Queries: ' + Core.getSOSLStatistics());
        report.add('Callouts: ' + Core.getCalloutStatistics());
        
        // Calcular porcentajes de uso
        Decimal cpuUsagePercent = (Decimal.valueOf(System.Limits.getCpuTime()) / 
                                  System.Limits.getLimitCpuTime() * 100).setScale(2);
        Decimal soqlUsagePercent = (Decimal.valueOf(System.Limits.getQueries()) / 
                                   System.Limits.getLimitQueries() * 100).setScale(2);
        
        report.add('CPU Usage: ' + cpuUsagePercent + '%');
        report.add('SOQL Usage: ' + soqlUsagePercent + '%');
        report.add('================================');
        
        String fullReport = String.join(report, '\n');
        Core.debug(fullReport);
        
        return fullReport;
    }
}

// Uso del monitor de rendimiento
PerformanceMonitor.startOperation('DataMigration');

try {
    // Realizar operaciones
    List<Account> accounts = [SELECT Id, Name FROM Account LIMIT 1000];
    
    for (Account acc : accounts) {
        acc.Description = 'Updated at ' + System.now();
        
        // Verificar límites periódicamente
        if (PerformanceMonitor.isApproachingLimits()) {
            Core.debug('Pausing operation due to approaching limits');
            break;
        }
    }
    
    update accounts;
    
} finally {
    PerformanceMonitor.endOperation('DataMigration');
    PerformanceMonitor.generateUsageReport();
}
```

### Ejemplo 5: Debugging en Triggers

```apex
public class AccountTriggerHandler {
    
    public static void handleAfterInsert(List<Account> newAccounts) {
        Core.debug('AccountTriggerHandler.handleAfterInsert started for ' + newAccounts.size() + ' accounts');
        
        try {
            // Verificar límites antes del procesamiento
            if (System.Limits.getQueries() > 80) { // Cerca del límite de 100
                Core.debug('WARNING: High SOQL usage detected: ' + Core.getSOQLStatistics());
            }
            
            // Procesar cuentas
            for (Account acc : newAccounts) {
                processNewAccount(acc);
            }
            
            Core.debug('AccountTriggerHandler.handleAfterInsert completed successfully');
            
        } catch (Exception e) {
            Core.debug('Error in AccountTriggerHandler.handleAfterInsert: ' + e.getMessage());
            
            // Registrar límites en caso de error para debugging
            Core.getSymmaryLimits();
            
            throw e;
        }
    }
    
    private static void processNewAccount(Account acc) {
        Core.debug('Processing new account: ' + acc.Name + ' (ID: ' + acc.Id + ')');
        
        // Lógica de procesamiento específica
        // ...
    }
}
```

## Uso en el Framework Foundation

La clase `Core` es utilizada extensivamente en todo el framework Foundation de Suratech:

- **EventLogManager**: Para debugging de operaciones de logs
- **GenerateURLServiceHandler**: Para debugging de procesos de integración
- **CCMServiceHandler**: Para debugging de comunicaciones CCM
- **FoundationEventLogController**: Para debugging de operaciones del controlador
- **QuoteUtils**: Para debugging de operaciones de cotización

## Mejores Prácticas para su Uso

1. **Debugging Consistente**: Utilizar `Core.debug()` en lugar de `System.debug()` directo para mantener consistencia en el formato de logs.

2. **Monitoreo Proactivo**: Usar los métodos de estadísticas para monitorear límites antes de que se alcancen.

3. **Logging en Puntos Críticos**: Registrar estadísticas al inicio y fin de operaciones complejas.

4. **Manejo de Errores**: Incluir `Core.getSymmaryLimits()` en bloques catch para facilitar el debugging.

5. **Verificación Periódica**: En bucles largos, verificar límites periódicamente para evitar excepciones.

## Consideraciones de Rendimiento

1. **Overhead Mínimo**: Los métodos de la clase `Core` tienen overhead mínimo y pueden usarse liberalmente.

2. **Frecuencia de Llamadas**: Los métodos de estadísticas pueden llamarse frecuentemente sin impacto significativo.

3. **Nivel de Logging**: `LoggingLevel.FINEST` puede ser filtrado en producción según la configuración de logging.

## Posibles Mejoras

### Corrección del Error Tipográfico

```apex
// Método corregido
global static void getSummaryLimits() {
  Core.debug(
    '===== Statistics(' +
      Core.getCPUTimeStatistics() +
      ',' +
      Core.getSOQLStatistics() +
      ',' +
      Core.getSOSLStatistics() +
      ',' +
      Core.getCalloutStatistics() +
      ') ====='
  );
}
```

### Extensión con Métodos Adicionales

```apex
// Métodos adicionales que podrían añadirse
global static String getDMLStatistics() {
  return 'DML statements usage=' +
    System.Limits.getDMLStatements() +
    '/' +
    System.Limits.getLimitDMLStatements();
}

global static String getHeapSizeStatistics() {
  return 'Heap size usage=' +
    System.Limits.getHeapSize() +
    '/' +
    System.Limits.getLimitHeapSize();
}

global static Boolean isApproachingAnyLimit(Integer thresholdPercent) {
  // Lógica para verificar si algún límite está cerca del threshold
  return false; // Implementación simplificada
}
```

## Notas Adicionales

1. **Clase Fundamental**: Esta es una clase fundamental del framework que es utilizada por prácticamente todas las demás clases del sistema.

2. **Logging Centralizado**: Proporciona un punto central para el logging, facilitando cambios futuros en la estrategia de logging.

3. **Monitoreo de Rendimiento**: Esencial para el monitoreo de rendimiento y la prevención de errores por límites del sistema.

4. **Simplicidad Intencional**: La simplicidad de la clase es intencional, proporcionando funcionalidad básica pero esencial de forma confiable.

5. **Sin Dependencias**: No depende de otras clases del framework, asegurando que siempre esté disponible para uso.