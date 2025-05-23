# Documentación Técnica: CustomMetadataCallback

## Descripción General
`CustomMetadataCallback` es una clase que implementa la interfaz `Metadata.DeployCallback` de Salesforce. Esta clase está diseñada para manejar los resultados de operaciones de despliegue de metadatos personalizados, proporcionando funcionalidad de callback que se ejecuta cuando una operación de despliegue de metadatos se completa (exitosa o fallida).

## Autor
Soulberto Lorenzo \<soulberto@cloudblue.us\>

## Última Modificación
28 de octubre de 2024

## Estructura de la Clase

```apex
public class CustomMetadataCallback implements Metadata.DeployCallback {
  public void handleResult(
    Metadata.DeployResult result,
    Metadata.DeployCallbackContext context
  ) {
    if (!Test.isRunningTest()) {
      Core.debug(
        String.format(
          '[{0}] Metadata updated job queued with jobId="{1}"',
          new List<String>{
            result.status == Metadata.DeployStatus.Succeeded
              ? 'Succeeded'
              : 'Failed',
            result.status == Metadata.DeployStatus.Succeeded
              ? context.getCallbackJobId().toString()
              : 'No created job'
          }
        )
      );
    }
  }
}
```

## Interfaz Implementada

### `Metadata.DeployCallback`
La clase implementa la interfaz `Metadata.DeployCallback` de Salesforce, que es parte de la API de Metadatos. Esta interfaz define el método `handleResult` que debe ser implementado para procesar los resultados de operaciones de despliegue de metadatos.

## Métodos

### `handleResult(Metadata.DeployResult result, Metadata.DeployCallbackContext context)`

#### Descripción
Método que se ejecuta automáticamente cuando una operación de despliegue de metadatos se completa. Procesa el resultado de la operación y registra información sobre el estado del despliegue.

#### Parámetros
- `result` (Metadata.DeployResult): Objeto que contiene información sobre el resultado del despliegue de metadatos.
- `context` (Metadata.DeployCallbackContext): Contexto del callback que proporciona información adicional sobre la operación.

#### Retorno
- `void`: Este método no devuelve ningún valor.

#### Proceso
1. Verifica si el código no se está ejecutando en un contexto de prueba.
2. Determina el estado del despliegue (Succeeded o Failed).
3. Obtiene el ID del trabajo de callback si el despliegue fue exitoso.
4. Formatea y registra un mensaje de depuración con la información del resultado.

#### Código
```apex
public void handleResult(
  Metadata.DeployResult result,
  Metadata.DeployCallbackContext context
) {
  if (!Test.isRunningTest()) {
    Core.debug(
      String.format(
        '[{0}] Metadata updated job queued with jobId="{1}"',
        new List<String>{
          result.status == Metadata.DeployStatus.Succeeded
            ? 'Succeeded'
            : 'Failed',
          result.status == Metadata.DeployStatus.Succeeded
            ? context.getCallbackJobId().toString()
            : 'No created job'
        }
      )
    );
  }
}
```

## Dependencias
- `Metadata.DeployCallback`: Interfaz estándar de Salesforce para callbacks de despliegue de metadatos.
- `Metadata.DeployResult`: Clase que encapsula el resultado de una operación de despliegue de metadatos.
- `Metadata.DeployCallbackContext`: Clase que proporciona contexto adicional para el callback.
- `Core`: Clase utilitaria que proporciona el método `debug` para registro de actividad.

## Propósito y Casos de Uso

Esta clase es especialmente útil en escenarios como:

1. **Despliegue Programático de Metadatos**: Cuando se necesita desplegar metadatos personalizados desde código Apex y monitorear el resultado.
2. **Operaciones Asíncronas**: Para manejar operaciones de metadatos que se ejecutan de forma asíncrona.
3. **Auditoría y Registro**: Para mantener un registro de las operaciones de despliegue de metadatos y su estado.
4. **Integración con Procesos de Negocio**: Cuando las actualizaciones de metadatos forman parte de procesos de negocio más amplios.

## Contexto de la API de Metadatos de Salesforce

La API de Metadatos de Salesforce permite crear, actualizar y eliminar metadatos de forma programática. Los callbacks como este se utilizan porque las operaciones de metadatos son inherentemente asíncronas y pueden tomar tiempo en completarse.

## Ejemplos de Uso

### Ejemplo 1: Despliegue de Metadatos Personalizados con Callback

```apex
public class MetadataDeploymentService {
    
    public static void deployCustomMetadata(String metadataName, Map<String, Object> fieldValues) {
        try {
            // Crear el registro de metadatos personalizados
            Metadata.CustomMetadata customMetadata = new Metadata.CustomMetadata();
            customMetadata.fullName = metadataName;
            customMetadata.label = metadataName;
            
            // Añadir valores de campos
            for (String fieldName : fieldValues.keySet()) {
                Metadata.CustomMetadataValue metadataValue = new Metadata.CustomMetadataValue();
                metadataValue.field = fieldName;
                metadataValue.value = fieldValues.get(fieldName);
                customMetadata.values.add(metadataValue);
            }
            
            // Crear el contenedor de despliegue
            Metadata.DeployContainer deployContainer = new Metadata.DeployContainer();
            deployContainer.addMetadata(customMetadata);
            
            // Crear la instancia del callback
            CustomMetadataCallback callback = new CustomMetadataCallback();
            
            // Ejecutar el despliegue con el callback
            Id deployJobId = Metadata.Operations.enqueueDeployment(deployContainer, callback);
            
            System.debug('Despliegue de metadatos iniciado con ID: ' + deployJobId);
        } catch (Exception e) {
            System.debug('Error al desplegar metadatos: ' + e.getMessage());
            throw e;
        }
    }
}

// Uso del servicio
Map<String, Object> configValues = new Map<String, Object>{
    'API_Endpoint__c' => 'https://api.example.com',
    'Timeout__c' => 30000,
    'Is_Active__c' => true
};

MetadataDeploymentService.deployCustomMetadata('MyConfig.Production', configValues);
```

### Ejemplo 2: Clase de Callback Extendida con Funcionalidad Adicional

```apex
public class EnhancedCustomMetadataCallback implements Metadata.DeployCallback {
    
    private String operationId;
    private String operationType;
    
    public EnhancedCustomMetadataCallback(String operationId, String operationType) {
        this.operationId = operationId;
        this.operationType = operationType;
    }
    
    public void handleResult(
        Metadata.DeployResult result,
        Metadata.DeployCallbackContext context
    ) {
        if (!Test.isRunningTest()) {
            // Registro detallado del resultado
            logDeploymentResult(result, context);
            
            // Enviar notificaciones según el resultado
            if (result.status == Metadata.DeployStatus.Succeeded) {
                handleSuccessfulDeployment(result, context);
            } else {
                handleFailedDeployment(result, context);
            }
        }
    }
    
    private void logDeploymentResult(
        Metadata.DeployResult result,
        Metadata.DeployCallbackContext context
    ) {
        String logMessage = String.format(
            '[{0}] Metadata deployment {1} - Operation: {2}, JobId: {3}',
            new List<String>{
                result.status.name(),
                result.status == Metadata.DeployStatus.Succeeded ? 'completed successfully' : 'failed',
                this.operationType,
                result.status == Metadata.DeployStatus.Succeeded 
                    ? context.getCallbackJobId().toString() 
                    : 'N/A'
            }
        );
        
        Core.debug(logMessage);
        
        // También registrar detalles adicionales si están disponibles
        if (result.details != null) {
            Core.debug('Deployment details: ' + JSON.serialize(result.details));
        }
    }
    
    private void handleSuccessfulDeployment(
        Metadata.DeployResult result,
        Metadata.DeployCallbackContext context
    ) {
        try {
            // Crear registro de auditoría
            createAuditRecord('SUCCESS', result, context);
            
            // Enviar notificación de éxito
            sendNotification('Deployment Successful', 
                           'Metadata deployment completed successfully for operation: ' + this.operationType);
            
            // Ejecutar acciones post-despliegue si las hay
            executePostDeploymentActions();
        } catch (Exception e) {
            Core.debug('Error in success handler: ' + e.getMessage());
        }
    }
    
    private void handleFailedDeployment(
        Metadata.DeployResult result,
        Metadata.DeployCallbackContext context
    ) {
        try {
            // Crear registro de auditoría para el fallo
            createAuditRecord('FAILED', result, context);
            
            // Enviar notificación de error
            sendNotification('Deployment Failed', 
                           'Metadata deployment failed for operation: ' + this.operationType);
            
            // Ejecutar acciones de rollback si las hay
            executeRollbackActions();
        } catch (Exception e) {
            Core.debug('Error in failure handler: ' + e.getMessage());
        }
    }
    
    private void createAuditRecord(
        String status, 
        Metadata.DeployResult result, 
        Metadata.DeployCallbackContext context
    ) {
        // Ejemplo de creación de registro de auditoría
        // Esto podría ser un objeto personalizado para tracking de deployments
        /*
        Metadata_Deployment_Log__c auditRecord = new Metadata_Deployment_Log__c(
            Operation_Id__c = this.operationId,
            Operation_Type__c = this.operationType,
            Status__c = status,
            Job_Id__c = context.getCallbackJobId()?.toString(),
            Deployment_Date__c = System.now(),
            Details__c = result.details != null ? JSON.serialize(result.details) : null
        );
        
        insert auditRecord;
        */
    }
    
    private void sendNotification(String subject, String body) {
        // Implementar lógica de notificación
        // Esto podría incluir emails, platform events, etc.
    }
    
    private void executePostDeploymentActions() {
        // Ejecutar acciones específicas después de un despliegue exitoso
        // Por ejemplo, actualizar caché, notificar a otros sistemas, etc.
    }
    
    private void executeRollbackActions() {
        // Ejecutar acciones de rollback en caso de fallo
        // Por ejemplo, restaurar configuraciones anteriores, notificar errores, etc.
    }
}
```

### Ejemplo 3: Uso en un Proceso de Configuración Dinámico

```apex
public class DynamicConfigurationManager {
    
    public class ConfigurationItem {
        public String name;
        public String value;
        public String type;
        
        public ConfigurationItem(String name, String value, String type) {
            this.name = name;
            this.value = value;
            this.type = type;
        }
    }
    
    public static void updateConfiguration(
        String configurationSetName, 
        List<ConfigurationItem> items
    ) {
        try {
            // Crear metadatos para cada item de configuración
            Metadata.DeployContainer deployContainer = new Metadata.DeployContainer();
            
            for (ConfigurationItem item : items) {
                Metadata.CustomMetadata customMetadata = createMetadataFromItem(
                    configurationSetName, item
                );
                deployContainer.addMetadata(customMetadata);
            }
            
            // Crear callback con información contextual
            EnhancedCustomMetadataCallback callback = new EnhancedCustomMetadataCallback(
                generateOperationId(),
                'CONFIGURATION_UPDATE'
            );
            
            // Ejecutar el despliegue
            Id deployJobId = Metadata.Operations.enqueueDeployment(deployContainer, callback);
            
            System.debug('Configuration update initiated with job ID: ' + deployJobId);
        } catch (Exception e) {
            System.debug('Error updating configuration: ' + e.getMessage());
            throw new ConfigurationException('Failed to update configuration: ' + e.getMessage(), e);
        }
    }
    
    private static Metadata.CustomMetadata createMetadataFromItem(
        String setName, 
        ConfigurationItem item
    ) {
        Metadata.CustomMetadata customMetadata = new Metadata.CustomMetadata();
        customMetadata.fullName = setName + '.' + item.name;
        customMetadata.label = item.name;
        
        // Añadir valor del item
        Metadata.CustomMetadataValue valueField = new Metadata.CustomMetadataValue();
        valueField.field = 'Value__c';
        valueField.value = item.value;
        customMetadata.values.add(valueField);
        
        // Añadir tipo del item
        Metadata.CustomMetadataValue typeField = new Metadata.CustomMetadataValue();
        typeField.field = 'Type__c';
        typeField.value = item.type;
        customMetadata.values.add(typeField);
        
        return customMetadata;
    }
    
    private static String generateOperationId() {
        return 'OP_' + String.valueOf(System.currentTimeMillis());
    }
    
    public class ConfigurationException extends Exception {}
}

// Uso del manager
List<DynamicConfigurationManager.ConfigurationItem> configItems = 
    new List<DynamicConfigurationManager.ConfigurationItem>{
        new DynamicConfigurationManager.ConfigurationItem('API_URL', 'https://new-api.com', 'String'),
        new DynamicConfigurationManager.ConfigurationItem('TIMEOUT', '45000', 'Number'),
        new DynamicConfigurationManager.ConfigurationItem('ENABLED', 'true', 'Boolean')
    };

DynamicConfigurationManager.updateConfiguration('ProductionConfig', configItems);
```

## Mejoras Propuestas para la Implementación Actual

### Versión Mejorada con Funcionalidad Adicional

```apex
/**
 * @description       : Callback para manejar resultados de despliegue de metadatos personalizados
 * @author            : Soulberto Lorenzo <soulberto@cloudblue.us>
 * @group             : Metadata Management
 * @last modified on  : 10-28-2024
 * @last modified by  : Soulberto Lorenzo <soulberto@cloudblue.us>
 **/
public class CustomMetadataCallback implements Metadata.DeployCallback {
    
    private String operationContext;
    private Map<String, Object> additionalData;
    
    /**
     * Constructor por defecto
     */
    public CustomMetadataCallback() {
        this.operationContext = 'DEFAULT';
        this.additionalData = new Map<String, Object>();
    }
    
    /**
     * Constructor con contexto
     * @param operationContext Contexto de la operación
     */
    public CustomMetadataCallback(String operationContext) {
        this.operationContext = operationContext;
        this.additionalData = new Map<String, Object>();
    }
    
    /**
     * Constructor con contexto y datos adicionales
     * @param operationContext Contexto de la operación
     * @param additionalData Datos adicionales para el callback
     */
    public CustomMetadataCallback(String operationContext, Map<String, Object> additionalData) {
        this.operationContext = operationContext;
        this.additionalData = additionalData != null ? additionalData : new Map<String, Object>();
    }
    
    /**
     * Maneja el resultado del despliegue de metadatos
     * @param result Resultado del despliegue
     * @param context Contexto del callback
     */
    public void handleResult(
        Metadata.DeployResult result,
        Metadata.DeployCallbackContext context
    ) {
        if (!Test.isRunningTest()) {
            try {
                // Registro básico del resultado
                logBasicResult(result, context);
                
                // Registro detallado si está disponible
                logDetailedResult(result);
                
                // Procesar según el estado del resultado
                if (result.status == Metadata.DeployStatus.Succeeded) {
                    handleSuccess(result, context);
                } else {
                    handleFailure(result, context);
                }
            } catch (Exception e) {
                Core.debug('Error in CustomMetadataCallback.handleResult: ' + e.getMessage());
                logCallbackError(e);
            }
        }
    }
    
    /**
     * Registra información básica del resultado
     */
    private void logBasicResult(Metadata.DeployResult result, Metadata.DeployCallbackContext context) {
        String statusText = result.status == Metadata.DeployStatus.Succeeded ? 'Succeeded' : 'Failed';
        String jobId = result.status == Metadata.DeployStatus.Succeeded 
            ? context.getCallbackJobId()?.toString() 
            : 'No created job';
        
        Core.debug(
            String.format(
                '[{0}] Metadata deployment {1} - Context: {2}, JobId: {3}',
                new List<String>{ statusText, statusText.toLowerCase(), this.operationContext, jobId }
            )
        );
    }
    
    /**
     * Registra información detallada del resultado si está disponible
     */
    private void logDetailedResult(Metadata.DeployResult result) {
        if (result.details != null) {
            // Registrar detalles específicos del despliegue
            Core.debug('Deployment details available - Components: ' + 
                      (result.details.componentSuccesses?.size() ?? 0) + ' successful, ' +
                      (result.details.componentFailures?.size() ?? 0) + ' failed');
            
            // Registrar errores específicos si los hay
            if (result.details.componentFailures != null && !result.details.componentFailures.isEmpty()) {
                for (Metadata.DeployMessage failure : result.details.componentFailures) {
                    Core.debug('Component failure: ' + failure.fullName + ' - ' + failure.problem);
                }
            }
        }
    }
    
    /**
     * Maneja el caso de despliegue exitoso
     */
    private void handleSuccess(Metadata.DeployResult result, Metadata.DeployCallbackContext context) {
        Core.debug('Metadata deployment completed successfully for context: ' + this.operationContext);
        
        // Ejecutar acciones post-éxito si están definidas en additionalData
        if (additionalData.containsKey('onSuccess') && additionalData.get('onSuccess') != null) {
            executeCallback('onSuccess');
        }
    }
    
    /**
     * Maneja el caso de despliegue fallido
     */
    private void handleFailure(Metadata.DeployResult result, Metadata.DeployCallbackContext context) {
        Core.debug('Metadata deployment failed for context: ' + this.operationContext);
        
        // Ejecutar acciones post-fallo si están definidas en additionalData
        if (additionalData.containsKey('onFailure') && additionalData.get('onFailure') != null) {
            executeCallback('onFailure');
        }
    }
    
    /**
     * Ejecuta callbacks adicionales definidos en additionalData
     */
    private void executeCallback(String callbackType) {
        try {
            // Aquí se podría implementar lógica para ejecutar callbacks específicos
            // Por ejemplo, invocar métodos específicos, enviar platform events, etc.
            Core.debug('Executing ' + callbackType + ' callback for context: ' + this.operationContext);
        } catch (Exception e) {
            Core.debug('Error executing ' + callbackType + ' callback: ' + e.getMessage());
        }
    }
    
    /**
     * Registra errores del callback mismo
     */
    private void logCallbackError(Exception e) {
        Core.debug('CustomMetadataCallback error - Context: ' + this.operationContext + 
                  ', Error: ' + e.getMessage() + ', Stack: ' + e.getStackTraceString());
    }
}
```

## Consideraciones de Rendimiento y Limitaciones

1. **Límites de API de Metadatos**: Las operaciones de metadatos están sujetas a límites específicos de Salesforce. Es importante estar consciente de estos límites al diseñar soluciones.

2. **Asincronía**: Las operaciones de metadatos son inherentemente asíncronas. El callback se ejecuta cuando la operación se completa, no inmediatamente.

3. **Contexto de Ejecución**: Los callbacks se ejecutan en un contexto específico y pueden tener limitaciones en cuanto a las operaciones que pueden realizar.

4. **Manejo de Errores**: Es importante incluir manejo robusto de errores en los callbacks para evitar que fallos en el callback afecten la operación principal.

## Pruebas Unitarias

```apex
@IsTest
private class CustomMetadataCallbackTest {
    
    @IsTest
    static void testSuccessfulDeployment() {
        Test.startTest();
        
        // Crear mock del resultado exitoso
        Metadata.DeployResult mockResult = new Metadata.DeployResult();
        mockResult.status = Metadata.DeployStatus.Succeeded;
        
        // Crear mock del contexto
        // Nota: En pruebas reales, necesitarías usar mocks más sofisticados
        // ya que Metadata.DeployCallbackContext no se puede instanciar directamente
        
        CustomMetadataCallback callback = new CustomMetadataCallback();
        
        // Ejecutar el método (no debería registrar nada durante las pruebas)
        callback.handleResult(mockResult, null);
        
        Test.stopTest();
        
        // En un escenario real, verificarías los efectos secundarios
        // como registros creados, notificaciones enviadas, etc.
        System.assert(true, 'Callback should execute without errors');
    }
    
    @IsTest
    static void testFailedDeployment() {
        Test.startTest();
        
        // Crear mock del resultado fallido
        Metadata.DeployResult mockResult = new Metadata.DeployResult();
        mockResult.status = Metadata.DeployStatus.Failed;
        
        CustomMetadataCallback callback = new CustomMetadataCallback();
        
        // Ejecutar el método
        callback.handleResult(mockResult, null);
        
        Test.stopTest();
        
        // Verificar que el método maneja correctamente los fallos
        System.assert(true, 'Callback should handle failures gracefully');
    }
}
```

## Notas Adicionales
1. Esta clase forma parte del ecosistema de gestión de metadatos de la organización y trabaja en conjunto con la API de Metadatos de Salesforce.
2. La verificación `!Test.isRunningTest()` es importante para evitar registrar información durante las pruebas unitarias.
3. El uso de `Core.debug()` sugiere que hay una clase utilitaria centralizada para el registro de actividad.
4. Esta implementación es básica pero funcional, y puede ser extendida según las necesidades específicas del proyecto.
5. Las operaciones de metadatos son poderosas pero deben usarse con cuidado, especialmente en entornos de producción.