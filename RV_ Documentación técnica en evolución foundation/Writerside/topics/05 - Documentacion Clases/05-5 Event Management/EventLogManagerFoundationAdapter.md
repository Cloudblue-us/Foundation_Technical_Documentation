# EventLogManagerFoundationAdapter

## Descripción General
`EventLogManagerFoundationAdapter` es una clase abstracta global que proporciona una estructura base para la implementación de adaptadores de gestión de registros de eventos. Esta clase define una interfaz común para aplicar la lógica de registro de eventos, permitiendo diferentes implementaciones específicas a través de herencia.

## Estructura

```apex
global abstract class EventLogManagerFoundationAdapter {
    
    global abstract Boolean applyEventLog(
        Map<String, Object> inputMap,
        Map<String, Object> outputMap,
        Map<String, Object> optionMap
    );

    public Boolean invokeMethod(
        String methodName,
        Map<String, Object> inputMap,
        Map<String, Object> outputMap,
        Map<String, Object> optionMap
    ) {
        switch on methodName.toUpperCase() {
            when 'APPLYEVENTLOG' {
                return applyEventLog(inputMap, outputMap, optionMap);
            }
            when else {
                Core.debug('Unsupported method: ' + methodName);
                return false;
            }
        }
    }
}
```

## Métodos

### `applyEventLog(Map<String, Object> inputMap, Map<String, Object> outputMap, Map<String, Object> optionMap)`

#### Descripción
Método abstracto que debe ser implementado por las clases derivadas. Este método aplica la lógica de registro de eventos.

#### Parámetros
- `inputMap` (Map<String, Object>): Mapa con los datos de entrada para el registro de eventos.
- `outputMap` (Map<String, Object>): Mapa donde se pueden guardar resultados o datos de salida.
- `optionMap` (Map<String, Object>): Mapa con opciones adicionales para configurar el comportamiento del registro.

#### Retorno
- `Boolean`: Indica si el registro de eventos se aplicó con éxito (`true`) o no (`false`).

#### Implementación
Por ser un método abstracto, no tiene implementación en esta clase. Las clases que extiendan `EventLogManagerFoundationAdapter` deben proporcionar una implementación concreta.

### `invokeMethod(String methodName, Map<String, Object> inputMap, Map<String, Object> outputMap, Map<String, Object> optionMap)`

#### Descripción
Método público que sirve como punto de entrada para invocar métodos específicos del adaptador. Actualmente, soporta el método 'APPLYEVENTLOG'.

#### Parámetros
- `methodName` (String): Nombre del método a invocar.
- `inputMap` (Map<String, Object>): Mapa con los datos de entrada para el método.
- `outputMap` (Map<String, Object>): Mapa donde se pueden guardar resultados o datos de salida.
- `optionMap` (Map<String, Object>): Mapa con opciones adicionales para configurar el comportamiento del método.

#### Retorno
- `Boolean`: Resultado de la operación, que depende del método invocado:
    - Para 'APPLYEVENTLOG': Devuelve el resultado del método `applyEventLog`.
    - Para cualquier otro método: Devuelve `false` e imprime un mensaje de depuración indicando que el método no está soportado.

#### Proceso
1. Convierte el nombre del método a mayúsculas para una comparación insensible a mayúsculas/minúsculas.
2. Utiliza una declaración `switch` para determinar qué método invocar:
    - Si el método es 'APPLYEVENTLOG', invoca el método abstracto `applyEventLog`.
    - Para cualquier otro método, registra un mensaje de depuración y devuelve `false`.

#### Código
```apex
public Boolean invokeMethod(
    String methodName,
    Map<String, Object> inputMap,
    Map<String, Object> outputMap,
    Map<String, Object> optionMap
) {
    switch on methodName.toUpperCase() {
        when 'APPLYEVENTLOG' {
            return applyEventLog(inputMap, outputMap, optionMap);
        }
        when else {
            Core.debug('Unsupported method: ' + methodName);
            return false;
        }
    }
}
```

## Patrones de Diseño Utilizados

### Adaptador (Adapter)
La clase implementa el patrón de diseño Adaptador, proporcionando una interfaz común (a través de `applyEventLog`) que puede ser implementada por diferentes adaptadores concretos para trabajar con distintos sistemas o mecanismos de registro de eventos.

### Plantilla (Template Method)
El método `invokeMethod` sigue el patrón Plantilla, definiendo la estructura de la operación, pero delegando pasos específicos (como `applyEventLog`) a las subclases.

### Comando (Command)
El método `invokeMethod` implementa un patrón similar al Comando, donde la acción a ejecutar se determina en función del nombre del método pasado como parámetro.

## Dependencias
- `Core`: Clase utilitaria que proporciona el método `debug` para registrar mensajes de depuración.

## Extensibilidad
La clase está diseñada para ser extendida. Las clases derivadas deben implementar el método abstracto `applyEventLog` para proporcionar la lógica específica de registro de eventos.

## Ejemplos de Implementación

### Implementación Básica

```apex
public class SimpleEventLogAdapter extends EventLogManagerFoundationAdapter {
    
    global override Boolean applyEventLog(
        Map<String, Object> inputMap,
        Map<String, Object> outputMap,
        Map<String, Object> optionMap
    ) {
        try {
            // Extraer datos necesarios del inputMap
            String eventType = (String)inputMap.get('eventType');
            String message = (String)inputMap.get('message');
            String userId = UserInfo.getUserId();
            Datetime timestamp = Datetime.now();
            
            // Crear un registro de evento
            Event_Log__c eventLog = new Event_Log__c(
                Event_Type__c = eventType,
                Message__c = message,
                User__c = userId,
                Timestamp__c = timestamp
            );
            
            // Opcionalmente, añadir detalles adicionales si están presentes
            if (inputMap.containsKey('details')) {
                eventLog.Details__c = JSON.serialize(inputMap.get('details'));
            }
            
            // Insertar el registro
            insert eventLog;
            
            // Opcionalmente, guardar información en el outputMap
            outputMap.put('success', true);
            outputMap.put('eventLogId', eventLog.Id);
            
            return true;
        } catch (Exception e) {
            // Registrar el error
            System.debug('Error al aplicar registro de evento: ' + e.getMessage());
            
            // Opcionalmente, guardar información de error en el outputMap
            outputMap.put('success', false);
            outputMap.put('errorMessage', e.getMessage());
            
            return false;
        }
    }
}
```

### Implementación con Plataforma de Eventos

```apex
public class PlatformEventLogAdapter extends EventLogManagerFoundationAdapter {
    
    global override Boolean applyEventLog(
        Map<String, Object> inputMap,
        Map<String, Object> outputMap,
        Map<String, Object> optionMap
    ) {
        try {
            // Extraer datos necesarios del inputMap
            String eventName = (String)inputMap.get('eventName');
            
            // Determinar dinámicamente qué tipo de evento de plataforma publicar
            switch on eventName {
                when 'SystemAudit' {
                    return publishSystemAuditEvent(inputMap, outputMap);
                }
                when 'SecurityAlert' {
                    return publishSecurityAlertEvent(inputMap, outputMap);
                }
                when 'IntegrationLog' {
                    return publishIntegrationLogEvent(inputMap, outputMap);
                }
                when else {
                    outputMap.put('success', false);
                    outputMap.put('errorMessage', 'Tipo de evento no soportado: ' + eventName);
                    return false;
                }
            }
        } catch (Exception e) {
            // Registrar el error
            System.debug('Error al publicar evento de plataforma: ' + e.getMessage());
            
            // Guardar información de error en el outputMap
            outputMap.put('success', false);
            outputMap.put('errorMessage', e.getMessage());
            
            return false;
        }
    }
    
    private Boolean publishSystemAuditEvent(
        Map<String, Object> inputMap,
        Map<String, Object> outputMap
    ) {
        // Crear el evento de plataforma
        System_Audit_Event__e auditEvent = new System_Audit_Event__e(
            User_Id__c = UserInfo.getUserId(),
            Action__c = (String)inputMap.get('action'),
            Record_Id__c = (String)inputMap.get('recordId'),
            Details__c = (String)inputMap.get('details')
        );
        
        // Publicar el evento
        Database.SaveResult result = EventBus.publish(auditEvent);
        
        // Verificar el resultado
        if (result.isSuccess()) {
            outputMap.put('success', true);
            outputMap.put('eventId', result.getId());
            return true;
        } else {
            outputMap.put('success', false);
            outputMap.put('errors', result.getErrors());
            return false;
        }
    }
    
    private Boolean publishSecurityAlertEvent(
        Map<String, Object> inputMap,
        Map<String, Object> outputMap
    ) {
        // Implementación similar para eventos de alerta de seguridad
        // ...
        return true;
    }
    
    private Boolean publishIntegrationLogEvent(
        Map<String, Object> inputMap,
        Map<String, Object> outputMap
    ) {
        // Implementación similar para eventos de log de integración
        // ...
        return true;
    }
}
```

### Implementación con Múltiples Destinos

```apex
public class MultiTargetEventLogAdapter extends EventLogManagerFoundationAdapter {
    
    global override Boolean applyEventLog(
        Map<String, Object> inputMap,
        Map<String, Object> outputMap,
        Map<String, Object> optionMap
    ) {
        // Determinar los destinos de registro basados en opciones
        Set<String> targetSystems = new Set<String>();
        
        if (optionMap.containsKey('targets')) {
            targetSystems = (Set<String>)optionMap.get('targets');
        } else {
            // Destinos predeterminados si no se especifican
            targetSystems.add('DATABASE');
            targetSystems.add('PLATFORM_EVENTS');
        }
        
        Boolean overallSuccess = true;
        Map<String, Object> results = new Map<String, Object>();
        
        // Aplicar el registro a cada destino
        for (String target : targetSystems) {
            Boolean targetSuccess = false;
            
            switch on target.toUpperCase() {
                when 'DATABASE' {
                    targetSuccess = logToDatabase(inputMap);
                }
                when 'PLATFORM_EVENTS' {
                    targetSuccess = logToPlatformEvents(inputMap);
                }
                when 'EXTERNAL_SYSTEM' {
                    targetSuccess = logToExternalSystem(inputMap);
                }
                when else {
                    System.debug('Destino de registro no soportado: ' + target);
                    targetSuccess = false;
                }
            }
            
            results.put(target, targetSuccess);
            overallSuccess = overallSuccess && targetSuccess;
        }
        
        // Guardar resultados detallados
        outputMap.put('targetResults', results);
        outputMap.put('overallSuccess', overallSuccess);
        
        return overallSuccess;
    }
    
    private Boolean logToDatabase(Map<String, Object> inputMap) {
        // Implementación para registrar en la base de datos
        // ...
        return true;
    }
    
    private Boolean logToPlatformEvents(Map<String, Object> inputMap) {
        // Implementación para publicar eventos de plataforma
        // ...
        return true;
    }
    
    private Boolean logToExternalSystem(Map<String, Object> inputMap) {
        // Implementación para enviar a un sistema externo
        // ...
        return true;
    }
}
```

## Ejemplo de Uso

```apex
// Crear una instancia del adaptador concreto
EventLogManagerFoundationAdapter logAdapter = new SimpleEventLogAdapter();

// Preparar los mapas de entrada, salida y opciones
Map<String, Object> inputMap = new Map<String, Object>{
    'eventType' => 'User Activity',
    'message' => 'Usuario actualizó registro',
    'details' => new Map<String, Object>{
        'recordId' => '001xx000003G9abCDE',
        'action' => 'UPDATE',
        'fields' => new List<String>{'Name', 'Phone', 'Email'}
    }
};

Map<String, Object> outputMap = new Map<String, Object>();
Map<String, Object> optionMap = new Map<String, Object>{
    'storeDetails' => true,
    'notifyAdmin' => false
};

// Invocar el método para aplicar el registro de eventos
Boolean success = logAdapter.invokeMethod('applyEventLog', inputMap, outputMap, optionMap);

// Verificar el resultado
if (success) {
    System.debug('Registro de evento aplicado con éxito. ID: ' + outputMap.get('eventLogId'));
} else {
    System.debug('Error al aplicar registro de evento: ' + outputMap.get('errorMessage'));
}
```

## Ejemplo de Uso en Sistema Más Amplio

```apex
public class TransactionManager {
    
    private EventLogManagerFoundationAdapter eventLogAdapter;
    
    public TransactionManager(EventLogManagerFoundationAdapter logAdapter) {
        this.eventLogAdapter = logAdapter;
    }
    
    public Boolean processTransaction(Map<String, Object> transactionData) {
        try {
            // 1. Validar datos de transacción
            validateTransaction(transactionData);
            
            // 2. Procesar la transacción
            String transactionId = executeTransaction(transactionData);
            
            // 3. Registrar evento de éxito
            Map<String, Object> logInput = new Map<String, Object>{
                'eventType' => 'Transaction',
                'message' => 'Transacción completada con éxito',
                'details' => new Map<String, Object>{
                    'transactionId' => transactionId,
                    'amount' => transactionData.get('amount'),
                    'accountId' => transactionData.get('accountId'),
                    'timestamp' => Datetime.now()
                }
            };
            
            Map<String, Object> logOutput = new Map<String, Object>();
            Map<String, Object> logOptions = new Map<String, Object>{
                'priority' => 'High',
                'retainDays' => 90
            };
            
            eventLogAdapter.invokeMethod('applyEventLog', logInput, logOutput, logOptions);
            
            return true;
        } catch (Exception e) {
            // Registrar evento de error
            Map<String, Object> errorLogInput = new Map<String, Object>{
                'eventType' => 'Error',
                'message' => 'Error en transacción: ' + e.getMessage(),
                'details' => new Map<String, Object>{
                    'errorType' => e.getTypeName(),
                    'stackTrace' => e.getStackTraceString(),
                    'transactionData' => transactionData
                }
            };
            
            Map<String, Object> errorLogOutput = new Map<String, Object>();
            Map<String, Object> errorLogOptions = new Map<String, Object>{
                'priority' => 'Critical',
                'notifyAdmin' => true
            };
            
            eventLogAdapter.invokeMethod('applyEventLog', errorLogInput, errorLogOutput, errorLogOptions);
            
            return false;
        }
    }
    
    private void validateTransaction(Map<String, Object> transactionData) {
        // Lógica de validación
        // ...
    }
    
    private String executeTransaction(Map<String, Object> transactionData) {
        // Lógica de ejecución de transacción
        // ...
        return 'TX-' + String.valueOf(Math.random()).substring(2, 10);
    }
}
```

## Consideraciones de Diseño

### Ventajas
1. **Flexibilidad**: Permite diferentes implementaciones de registro de eventos sin cambiar la interfaz.
2. **Extensibilidad**: Facilita la adición de nuevos tipos de adaptadores de registro.
3. **Separación de Responsabilidades**: Separa la lógica de invocación de métodos de la implementación específica de registro.
4. **Reutilización**: La estructura común puede ser reutilizada por múltiples adaptadores.

### Desafíos
1. **Complejidad**: Introduce una capa adicional de abstracción.
2. **Mantenimiento**: Requiere mantener la coherencia entre diferentes implementaciones.
3. **Rendimiento**: Puede tener un impacto en el rendimiento debido a la abstracción adicional.

## Mejores Prácticas

1. **Documentación Clara**: Documentar claramente la estructura esperada de los mapas de entrada, salida y opciones para cada implementación.
2. **Manejo de Errores Robusto**: Implementar un manejo de errores coherente en todas las implementaciones.
3. **Pruebas Unitarias**: Crear pruebas unitarias exhaustivas para cada implementación concreta.
4. **Uso Consistente**: Mantener una estructura consistente para los datos de entrada y salida entre diferentes implementaciones.
5. **Extender según Necesidades**: Considerar extender la funcionalidad añadiendo más métodos soportados en `invokeMethod` si es necesario.

## Notas Adicionales
1. La clase utiliza mapas genéricos (`Map<String, Object>`) para los datos de entrada, salida y opciones, proporcionando flexibilidad pero requiriendo un cuidado adicional para garantizar la consistencia de los datos.
2. El uso de `Core.debug` para los mensajes de depuración sugiere que hay una clase utilitaria `Core` disponible en el sistema.
3. La naturaleza abstracta de la clase asegura que solo se pueden crear instancias de subclases concretas.
4. El modificador `global` indica que la clase está diseñada para ser accesible desde diferentes paquetes o namespaces.
5. Actualmente, el método `invokeMethod` solo soporta 'APPLYEVENTLOG', pero el diseño permite añadir fácilmente soporte para más métodos en el futuro.