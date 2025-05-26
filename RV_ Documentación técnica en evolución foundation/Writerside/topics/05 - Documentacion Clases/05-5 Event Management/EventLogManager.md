# EventLogManager

## Descripción General
`EventLogManager` es una clase global con sharing que proporciona funcionalidad completa para la gestión de registros de eventos en el sistema. Esta clase maneja tanto Big Objects (`EventLog__b`) como objetos personalizados estándar (`EventLog_Subset__c`), ofreciendo capacidades de consulta, guardado, análisis y limpieza de logs de eventos.

## Autor
Soulberto Lorenzo \<soulberto@cloudblue.us\>

## Última Modificación
17 de marzo de 2025 por Esneyder Zabala

## Estructura de la Clase

```apex
global with sharing class EventLogManager {
  public static List<EventLog__b> fetchAll(Datetime startAt, Datetime endAt);
  public static Boolean save(List<Object> inputMapList);
  @future(callout=true) global static void saveAsync(String serializeInputMapList);
  public static Boolean refreshAnalytics(Datetime startAt, Datetime endAt, Boolean reset);
  @future public static void refreshAnalyticsAsync(Datetime startAt, Datetime endAt, Boolean reset);
  public static Boolean clearAnalytics();
  @future public static void clearLogsAsync();
}
```

## Métodos

### `fetchAll(Datetime startAt, Datetime endAt)`

#### Descripción
Recupera registros de eventos del Big Object `EventLog__b` dentro de un rango de fechas específico. Incluye soporte para testing a través de mocks.

#### Parámetros
- `startAt` (Datetime): Fecha y hora de inicio del rango de consulta.
- `endAt` (Datetime): Fecha y hora de fin del rango de consulta.

#### Retorno
- `List<EventLog__b>`: Lista de registros de eventos encontrados en el rango especificado.

#### Proceso
1. Registra información de depuración sobre el rango de consulta.
2. Ejecuta una consulta SOQL al Big Object `EventLog__b` con límite de 50,000 registros.
3. Registra el número de registros encontrados.
4. Si se ejecuta en contexto de prueba, devuelve datos mock.
5. Retorna los registros encontrados.

#### Código
```apex
public static List<EventLog__b> fetchAll(Datetime startAt, Datetime endAt) {
  Core.debug('Querying Event Logs from ' + startAt + ' to ' + endAt);
  List<EventLog__b> records = [
    SELECT
      Id,
      Date__c,
      Product__c,
      Level__c,
      Platform__c,
      Process__c,
      Details__c,
      Data__c,
      Type__c,
      UserId__c
    FROM EventLog__b
    WHERE Date__c >= :startAt AND Date__c <= :endAt
    LIMIT 50000
  ];
  Core.debug('Found ' + records.size() + ' records on the specified range...');

  if (Test.isRunningTest()) {
    return EventLogManagerMock.getMockLogs();
  }

  return records;
}
```

### `save(List<Object> inputMapList)`

#### Descripción
Guarda una lista de registros de eventos tanto en el Big Object `EventLog__b` como en el objeto personalizado `EventLog_Subset__c`. Incluye validación de campos obligatorios y procesamiento por lotes.

#### Parámetros
- `inputMapList` (List<Object>): Lista de mapas que contienen los datos de eventos a guardar.

#### Retorno
- `Boolean`: `true` si el guardado fue exitoso, `false` en caso de error.

#### Proceso
1. Verifica si la feature flag 'sura_foundation_event_logs' está habilitada.
2. Itera sobre la lista de mapas de entrada.
3. Valida que cada mapa contenga los campos obligatorios (Level, Type, Product).
4. Crea registros para ambos objetos: `EventLog__b` y `EventLog_Subset__c`.
5. Procesa en lotes de 200 registros para optimizar el rendimiento.
6. Ejecuta las operaciones de inserción correspondientes.
7. Maneja excepciones y retorna el resultado.

#### Campos Procesados
- **Date__c**: Fecha y hora actual
- **Product__c**: Producto (obligatorio)
- **Level__c**: Nivel del evento (obligatorio)
- **Platform__c**: Plataforma de origen
- **Process__c**: Proceso relacionado
- **Details__c**: Detalles del evento
- **Data__c**: Datos adicionales
- **Type__c**: Tipo de evento (obligatorio)
- **UserId__c**: ID del usuario
- **Response__c**: Respuesta del evento

### `saveAsync(String serializeInputMapList)`

#### Descripción
Versión asíncrona del método `save` que utiliza el decorador `@future(callout=true)` para procesar el guardado de eventos de forma asíncrona.

#### Parámetros
- `serializeInputMapList` (String): Lista de mapas serializada en JSON que contiene los datos de eventos a guardar.

#### Retorno
- `void`: Al ser un método future, no devuelve valor.

#### Proceso
1. Deserializa la cadena JSON a una lista de objetos.
2. Llama al método `save` con la lista deserializada.

#### Código
```apex
@future(callout=true)
global static void saveAsync(String serializeInputMapList) {
  List<Object> inputMapList = (List<Object>) JSON.deserializeUntyped(
    serializeInputMapList
  );
  EventLogManager.save(inputMapList);
}
```

### `refreshAnalytics(Datetime startAt, Datetime endAt, Boolean reset)`

#### Descripción
Actualiza los datos analíticos poblando el objeto `EventLog_Subset__c` con datos del Big Object `EventLog__b` dentro de un rango de fechas específico.

#### Parámetros
- `startAt` (Datetime): Fecha de inicio del rango para el análisis.
- `endAt` (Datetime): Fecha de fin del rango para el análisis.
- `reset` (Boolean): Indica si se deben limpiar los datos analíticos existentes antes de la actualización.

#### Retorno
- `Boolean`: `true` si la actualización fue exitosa.

#### Proceso
1. Si `reset` es true, limpia los datos analíticos existentes.
2. Consulta eventos del Big Object usando `fetchAll`.
3. Crea registros correspondientes en `EventLog_Subset__c`.
4. Inserta los registros del subset si no se está ejecutando en pruebas.

### `refreshAnalyticsAsync(Datetime startAt, Datetime endAt, Boolean reset)`

#### Descripción
Versión asíncrona del método `refreshAnalytics` que utiliza el decorador `@future`.

#### Parámetros
- `startAt` (Datetime): Fecha de inicio del rango.
- `endAt` (Datetime): Fecha de fin del rango.
- `reset` (Boolean): Indica si se deben limpiar los datos existentes.

#### Retorno
- `void`: Al ser un método future, no devuelve valor.

### `clearAnalytics()`

#### Descripción
Elimina todos los registros del objeto `EventLog_Subset__c` para limpiar los datos analíticos.

#### Parámetros
Ninguno.

#### Retorno
- `Boolean`: `true` si la limpieza fue exitosa, `false` en caso de error.

#### Proceso
1. Consulta hasta 50,000 registros de `EventLog_Subset__c`.
2. Si existen registros, los elimina.
3. Maneja excepciones y retorna el resultado.

### `clearLogsAsync()`

#### Descripción
Método asíncrono que elimina registros del Big Object `EventLog__b` para limpiar logs antiguos.

#### Parámetros
Ninguno.

#### Retorno
- `void`: Al ser un método future, no devuelve valor.

#### Proceso
1. Consulta hasta 5,000 registros de `EventLog__b`.
2. Si existen registros, los elimina usando `Database.deleteImmediate`.
3. Maneja excepciones internamente.

## Dependencias

- **`EventLog__b`**: Big Object que almacena los registros de eventos principales.
- **`EventLog_Subset__c`**: Objeto personalizado que almacena un subconjunto de eventos para análisis.
- **`FeatureFlags`**: Clase que maneja feature flags para controlar la funcionalidad.
- **`Core`**: Clase utilitaria para registro de depuración.
- **`EventLogManagerMock`**: Clase mock para proporcionar datos de prueba.

## Arquitectura de Datos

### EventLog__b (Big Object)
Big Object optimizado para grandes volúmenes de datos de eventos:

| Campo | Tipo | Descripción |
|-------|------|-------------|
| Id | Text | Identificador único |
| Date__c | DateTime | Fecha y hora del evento |
| Product__c | Text | Producto relacionado |
| Level__c | Text | Nivel del evento (Info, Warning, Error, etc.) |
| Platform__c | Text | Plataforma de origen |
| Process__c | Text | Proceso que generó el evento |
| Details__c | LongTextArea | Detalles del evento |
| Data__c | LongTextArea | Datos adicionales |
| Type__c | Text | Tipo de evento |
| UserId__c | Text | ID del usuario |
| Response__c | LongTextArea | Respuesta del evento |

### EventLog_Subset__c (Objeto Personalizado)
Objeto estándar para análisis y reportes:

| Campo | Tipo | Descripción |
|-------|------|-------------|
| Name | Text | Nombre del registro |
| Date__c | DateTime | Fecha y hora del evento |
| Product__c | Text | Producto relacionado |
| Level__c | Text | Nivel del evento |
| Platform__c | Text | Plataforma de origen |
| Process__c | Text | Proceso que generó el evento |
| Details__c | LongTextArea | Detalles del evento |
| Type__c | Text | Tipo de evento |
| Response__c | LongTextArea | Respuesta del evento |
| Data__c | LongTextArea | Datos adicionales |

## Ejemplos de Uso

### Ejemplo 1: Guardar Eventos de Forma Síncrona

```apex
public class EventLogger {
    
    public static void logUserActivity(String userId, String activity, String details) {
        List<Object> events = new List<Object>();
        
        Map<String, Object> eventData = new Map<String, Object>{
            'Level' => 'Info',
            'Type' => 'USER_ACTIVITY',
            'Product' => 'CRM',
            'Platform' => 'Salesforce',
            'Process' => 'User Management',
            'Details' => details,
            'UserId' => userId,
            'Data' => JSON.serialize(new Map<String, Object>{
                'activity' => activity,
                'timestamp' => System.now(),
                'sessionId' => UserInfo.getSessionId()
            })
        };
        
        events.add(eventData);
        
        Boolean success = EventLogManager.save(events);
        
        if (success) {
            System.debug('Evento de actividad de usuario registrado exitosamente');
        } else {
            System.debug('Error al registrar evento de actividad de usuario');
        }
    }
}

// Uso
EventLogger.logUserActivity(UserInfo.getUserId(), 'LOGIN', 'Usuario inició sesión exitosamente');
```

### Ejemplo 2: Guardar Múltiples Eventos Asincrónicamente

```apex
public class BatchEventLogger {
    
    public static void logIntegrationEvents(List<Map<String, Object>> integrationResults) {
        List<Object> events = new List<Object>();
        
        for (Map<String, Object> result : integrationResults) {
            String level = (Boolean) result.get('success') ? 'Info' : 'Error';
            
            Map<String, Object> eventData = new Map<String, Object>{
                'Level' => level,
                'Type' => 'INTEGRATION',
                'Product' => (String) result.get('product'),
                'Platform' => 'External API',
                'Process' => 'Data Integration',
                'Details' => (String) result.get('message'),
                'UserId' => UserInfo.getUserId(),
                'Response' => JSON.serialize(result.get('response')),
                'Data' => JSON.serialize(new Map<String, Object>{
                    'endpoint' => result.get('endpoint'),
                    'duration' => result.get('duration'),
                    'statusCode' => result.get('statusCode')
                })
            };
            
            events.add(eventData);
        }
        
        // Guardar asincrónicamente
        String serializedEvents = JSON.serialize(events);
        EventLogManager.saveAsync(serializedEvents);
        
        System.debug('Enviados ' + events.size() + ' eventos para procesamiento asíncrono');
    }
}

// Uso
List<Map<String, Object>> integrationResults = new List<Map<String, Object>>{
    new Map<String, Object>{
        'success' => true,
        'product' => 'Insurance',
        'message' => 'Póliza creada exitosamente',
        'endpoint' => '/api/v1/policies',
        'duration' => 1250,
        'statusCode' => 200,
        'response' => new Map<String, Object>{'policyId' => 'POL-123456'}
    },
    new Map<String, Object>{
        'success' => false,
        'product' => 'Insurance',
        'message' => 'Error al validar datos del cliente',
        'endpoint' => '/api/v1/customers/validate',
        'duration' => 500,
        'statusCode' => 400,
        'response' => new Map<String, Object>{'error' => 'Invalid customer data'}
    }
};

BatchEventLogger.logIntegrationEvents(integrationResults);
```

### Ejemplo 3: Análisis y Consulta de Eventos

```apex
public class EventAnalytics {
    
    public static Map<String, Object> generateDailyReport(Date reportDate) {
        Datetime startOfDay = Datetime.newInstance(reportDate, Time.newInstance(0, 0, 0, 0));
        Datetime endOfDay = Datetime.newInstance(reportDate, Time.newInstance(23, 59, 59, 999));
        
        // Obtener eventos del día
        List<EventLog__b> dailyEvents = EventLogManager.fetchAll(startOfDay, endOfDay);
        
        // Análisis básico
        Map<String, Integer> eventsByLevel = new Map<String, Integer>();
        Map<String, Integer> eventsByProduct = new Map<String, Integer>();
        Map<String, Integer> eventsByType = new Map<String, Integer>();
        
        for (EventLog__b event : dailyEvents) {
            // Contar por nivel
            String level = event.Level__c;
            eventsByLevel.put(level, eventsByLevel.containsKey(level) ? 
                             eventsByLevel.get(level) + 1 : 1);
            
            // Contar por producto
            String product = event.Product__c;
            eventsByProduct.put(product, eventsByProduct.containsKey(product) ? 
                               eventsByProduct.get(product) + 1 : 1);
            
            // Contar por tipo
            String type = event.Type__c;
            eventsByType.put(type, eventsByType.containsKey(type) ? 
                            eventsByType.get(type) + 1 : 1);
        }
        
        return new Map<String, Object>{
            'date' => reportDate,
            'totalEvents' => dailyEvents.size(),
            'eventsByLevel' => eventsByLevel,
            'eventsByProduct' => eventsByProduct,
            'eventsByType' => eventsByType,
            'generatedAt' => System.now()
        };
    }
    
    public static void refreshAnalyticsForPeriod(Date startDate, Date endDate) {
        Datetime startDatetime = Datetime.newInstance(startDate, Time.newInstance(0, 0, 0, 0));
        Datetime endDatetime = Datetime.newInstance(endDate, Time.newInstance(23, 59, 59, 999));
        
        // Refrescar análisis asincrónicamente
        EventLogManager.refreshAnalyticsAsync(startDatetime, endDatetime, true);
        
        System.debug('Actualización de análisis iniciada para el período: ' + 
                    startDate + ' - ' + endDate);
    }
}

// Uso
Map<String, Object> dailyReport = EventAnalytics.generateDailyReport(Date.today());
System.debug('Reporte diario: ' + JSON.serializePretty(dailyReport));

// Refrescar análisis para la última semana
Date lastWeek = Date.today().addDays(-7);
EventAnalytics.refreshAnalyticsForPeriod(lastWeek, Date.today());
```

### Ejemplo 4: Mantenimiento y Limpieza de Logs

```apex
public class EventLogMaintenance {
    
    @InvocableMethod(label='Limpiar Logs Antiguos' description='Elimina logs de eventos antiguos')
    public static void cleanupOldLogs() {
        // Limpiar logs del Big Object asincrónicamente
        EventLogManager.clearLogsAsync();
        
        System.debug('Proceso de limpieza de logs iniciado');
    }
    
    public static void scheduleAnalyticsRefresh() {
        // Refrescar análisis para los últimos 30 días
        Datetime thirtyDaysAgo = Datetime.now().addDays(-30);
        Datetime now = Datetime.now();
        
        EventLogManager.refreshAnalyticsAsync(thirtyDaysAgo, now, false);
        
        System.debug('Actualización programada de análisis iniciada');
    }
    
    public static void clearAnalyticsData() {
        Boolean success = EventLogManager.clearAnalytics();
        
        if (success) {
            System.debug('Datos analíticos limpiados exitosamente');
        } else {
            System.debug('Error al limpiar datos analíticos');
        }
    }
}

// Programar limpieza semanal
public class EventLogMaintenanceSchedule implements Schedulable {
    public void execute(SchedulableContext sc) {
        EventLogMaintenance.cleanupOldLogs();
        EventLogMaintenance.scheduleAnalyticsRefresh();
    }
}

// Programar la limpieza para ejecutarse cada domingo a las 2 AM
String cronExp = '0 0 2 ? * SUN';
String jobId = System.schedule('Weekly Event Log Cleanup', cronExp, new EventLogMaintenanceSchedule());
```

## Mejores Prácticas Implementadas

1. **Procesamiento por Lotes**: La clase procesa registros en lotes de 200 para optimizar el rendimiento.

2. **Validación de Datos**: Valida campos obligatorios antes de procesar los eventos.

3. **Feature Flags**: Utiliza feature flags para controlar la funcionalidad de logging.

4. **Procesamiento Asíncrono**: Proporciona métodos asíncronos para operaciones que pueden tomar tiempo.

5. **Manejo de Excepciones**: Incluye manejo robusto de excepciones con logging de errores.

6. **Soporte para Testing**: Incluye verificaciones para contextos de prueba y usa mocks.

## Consideraciones de Rendimiento

1. **Límites de Consulta**: Las consultas están limitadas (50,000 para fetch, 5,000 para limpieza) para evitar timeouts.

2. **Big Objects**: Utiliza Big Objects para almacenar grandes volúmenes de datos de eventos de forma eficiente.

3. **Procesamiento Asíncrono**: Los métodos future permiten procesar operaciones pesadas sin bloquear la transacción principal.

4. **Lotes de Inserción**: Procesa inserciones en lotes para optimizar el uso de recursos.

## Observaciones sobre el Código

### Código Comentado
La clase contiene código comentado que sugiere implementaciones alternativas o funcionalidades no activas:

- Limitaciones de longitud de campos comentadas
- Lógica de procesamiento por lotes comentada en algunos métodos
- Diferentes enfoques para eliminar registros

### Mejoras Potenciales

1. **Validación de Entrada**: Podría beneficiarse de validaciones más robustas de los datos de entrada.

2. **Gestión de Errores**: Mejorar el reporte de errores específicos durante las operaciones batch.

3. **Configuración Externalizada**: Hacer configurables los límites de registros y tamaños de lote.

4. **Métricas de Rendimiento**: Añadir métricas para monitorear el rendimiento de las operaciones.

## Notas Adicionales

1. **Seguridad**: La clase utiliza `WITH SECURITY_ENFORCED` en algunas consultas para respetar las reglas de seguridad.

2. **Dual Storage**: Mantiene datos tanto en Big Objects (para volumen) como en objetos estándar (para análisis).

3. **Feature Flag Integration**: La funcionalidad de guardado depende de la feature flag 'sura_foundation_event_logs'.

4. **Testing Support**: Incluye soporte completo para testing con mocks y verificaciones de contexto.

5. **Flexibilidad**: El diseño permite diferentes tipos de eventos y productos, siendo flexible para diversos casos de uso.