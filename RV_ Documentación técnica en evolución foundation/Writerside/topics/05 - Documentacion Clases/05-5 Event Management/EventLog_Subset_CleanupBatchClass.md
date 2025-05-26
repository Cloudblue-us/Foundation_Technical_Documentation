# EventLog_Subset_CleanupBatchClass

## Descripción General
`EventLog_Subset_CleanupBatchClass` es una clase que implementa la interfaz `Database.Batchable<sObject>` de Salesforce. Esta clase está diseñada para ejecutar operaciones de eliminación en lotes sobre registros del objeto `EventLog_Subset__c` (inferido por el nombre de la clase), permitiendo procesar grandes volúmenes de datos de manera eficiente y dentro de los límites de gobernabilidad de la plataforma.

## Autor
No especificado (aparece como "ChangeMeIn@UserSettingsUnder.SFDoc")

## Última Modificación
25 de febrero de 2025

## Estructura de la Clase

```apex
public class EventLog_Subset_CleanupBatchClass implements Database.Batchable<sObject> {
  private String query;

  public EventLog_Subset_CleanupBatchClass(String query) {
    this.query = query;
  }

  public Database.QueryLocator start(Database.BatchableContext bc) {
    return Database.getQueryLocator(this.query);
  }

  public void execute(Database.BatchableContext bc, List<sObject> scope) {
    delete scope;
  }

  public void finish(Database.BatchableContext bc) {
    System.debug('Finish Method');
  }
}
```

## Interfaz Implementada

### `Database.Batchable<sObject>`
La clase implementa la interfaz `Database.Batchable<sObject>` de Salesforce, que define tres métodos que deben ser implementados:
- `start`: Método que inicia el proceso por lotes y define el conjunto de registros a procesar.
- `execute`: Método que se ejecuta para cada lote de registros.
- `finish`: Método que se ejecuta después de que todos los lotes han sido procesados.

## Propiedades

### `query`
- **Tipo**: `String`
- **Descripción**: Almacena la consulta SOQL que determina qué registros serán procesados por el trabajo por lotes.
- **Visibilidad**: Privada

## Constructor

### `EventLog_Subset_CleanupBatchClass(String query)`

#### Descripción
Constructor que inicializa la clase con la consulta SOQL que determinará qué registros serán procesados.

#### Parámetros
- `query` (String): Consulta SOQL que especifica los registros a procesar.

#### Código
```apex
public EventLog_Subset_CleanupBatchClass(String query) {
  this.query = query;
}
```

## Métodos

### `start(Database.BatchableContext bc)`

#### Descripción
Método que se ejecuta al inicio del proceso por lotes. Define el conjunto de registros que serán procesados utilizando un QueryLocator basado en la consulta SOQL proporcionada.

#### Parámetros
- `bc` (Database.BatchableContext): Contexto del trabajo por lotes, proporcionado por la plataforma Salesforce.

#### Retorno
- `Database.QueryLocator`: Un QueryLocator que define el conjunto de registros a procesar.

#### Código
```apex
public Database.QueryLocator start(Database.BatchableContext bc) {
  return Database.getQueryLocator(this.query);
}
```

### `execute(Database.BatchableContext bc, List<sObject> scope)`

#### Descripción
Método que se ejecuta para cada lote de registros definido por el QueryLocator. Elimina todos los registros en el lote actual.

#### Parámetros
- `bc` (Database.BatchableContext): Contexto del trabajo por lotes, proporcionado por la plataforma Salesforce.
- `scope` (List<sObject>): Lista de registros a procesar en el lote actual.

#### Retorno
- `void`: Este método no devuelve ningún valor.

#### Proceso
1. Elimina todos los registros en el lote actual utilizando la operación DML `delete`.

#### Código
```apex
public void execute(Database.BatchableContext bc, List<sObject> scope) {
  delete scope;
}
```

### `finish(Database.BatchableContext bc)`

#### Descripción
Método que se ejecuta después de que todos los lotes han sido procesados. Actualmente solo registra un mensaje de depuración.

#### Parámetros
- `bc` (Database.BatchableContext): Contexto del trabajo por lotes, proporcionado por la plataforma Salesforce.

#### Retorno
- `void`: Este método no devuelve ningún valor.

#### Proceso
1. Registra un mensaje de depuración indicando que el método de finalización ha sido ejecutado.

#### Código
```apex
public void finish(Database.BatchableContext bc) {
  System.debug('Finish Method');
}
```

## Relación con EventLog_Subset_CleanupScheduleClass

Esta clase está diseñada para ser utilizada en conjunto con `EventLog_Subset_CleanupScheduleClass`, que es una clase programable que configura y ejecuta este trabajo por lotes. La clase programable:

1. Verifica si la limpieza debe ejecutarse basándose en la configuración de metadatos personalizados.
2. Construye una consulta SOQL para seleccionar registros `EventLog_Subset__c` que sean más antiguos que un día.
3. Inicia este trabajo por lotes pasándole la consulta y un tamaño de lote de 150 registros.

## Flujo de Trabajo

1. Una clase programable o algún otro proceso construye una consulta SOQL para seleccionar los registros de `EventLog_Subset__c` que deben ser eliminados.
2. Se crea una instancia de `EventLog_Subset_CleanupBatchClass` pasándole la consulta.
3. Se ejecuta el trabajo por lotes utilizando `Database.executeBatch()`, especificando opcionalmente un tamaño de lote.
4. El método `start` se ejecuta, definiendo el conjunto completo de registros a procesar.
5. El método `execute` se ejecuta para cada lote de registros, eliminándolos.
6. Después de que todos los lotes han sido procesados, se ejecuta el método `finish`, que actualmente solo registra un mensaje de depuración.

## Mejores Prácticas Implementadas

1. **Procesamiento por Lotes**: La clase utiliza el framework de procesamiento por lotes de Salesforce para manejar grandes volúmenes de datos de manera eficiente.
2. **Configurabilidad**: La consulta SOQL que determina qué registros se procesan se proporciona externamente, lo que permite flexibilidad en la selección de registros.
3. **Simplicidad**: El código es conciso y se enfoca en una única responsabilidad: eliminar los registros especificados.

## Mejoras Posibles

1. **Manejo de Errores**: La implementación actual no incluye manejo de errores durante la eliminación. Podría mejorar incluyendo bloques try-catch para manejar y registrar posibles errores.
2. **Registro de Actividad**: Podría añadir más registro de actividad para facilitar el seguimiento y la depuración.
3. **Notificaciones**: Podría implementar notificaciones para informar sobre el resultado del proceso de limpieza.
4. **Información de Resumen**: El método `finish` podría proporcionar información resumida sobre cuántos registros se procesaron y eliminaron.

## Implementación Mejorada Propuesta

```apex
/**
 * @description       : Clase por lotes para eliminar registros antiguos de EventLog_Subset__c
 * @author            : [Nombre del Autor]
 * @group             : Mantenimiento
 * @last modified on  : 02-25-2025
 * @last modified by  : [Nombre del Modificador]
 **/
public class EventLog_Subset_CleanupBatchClass implements Database.Batchable<sObject>, Database.Stateful {
  private String query;
  private Integer totalRecordsProcessed = 0;
  private Integer totalRecordsDeleted = 0;
  private List<String> errors = new List<String>();
  
  /**
   * @description Constructor que inicializa la clase con la consulta SOQL para seleccionar los registros a eliminar
   * @param query Consulta SOQL que define los registros a procesar
   */
  public EventLog_Subset_CleanupBatchClass(String query) {
    this.query = query;
    System.debug(LoggingLevel.INFO, 'Iniciando trabajo de limpieza con consulta: ' + query);
  }
  
  /**
   * @description Define el conjunto de registros que serán procesados
   * @param bc Contexto del trabajo por lotes
   * @return QueryLocator que define el conjunto de registros a procesar
   */
  public Database.QueryLocator start(Database.BatchableContext bc) {
    try {
      return Database.getQueryLocator(this.query);
    } catch (Exception e) {
      System.debug(LoggingLevel.ERROR, 'Error en método start: ' + e.getMessage());
      errors.add('Error en método start: ' + e.getMessage());
      // Retornar un QueryLocator vacío en caso de error
      return Database.getQueryLocator('SELECT Id FROM EventLog_Subset__c WHERE Id = null');
    }
  }
  
  /**
   * @description Procesa cada lote de registros, eliminándolos
   * @param bc Contexto del trabajo por lotes
   * @param scope Lista de registros a procesar en el lote actual
   */
  public void execute(Database.BatchableContext bc, List<sObject> scope) {
    if (scope == null || scope.isEmpty()) {
      return;
    }
    
    totalRecordsProcessed += scope.size();
    System.debug(LoggingLevel.INFO, 'Procesando lote de ' + scope.size() + ' registros');
    
    try {
      // Opcionalmente, registrar algunos detalles de los registros antes de eliminarlos
      if (System.isBatch()) {
        for (SObject record : scope) {
          System.debug(LoggingLevel.FINE, 'Eliminando registro: ' + record.Id);
        }
      }
      
      // Eliminar los registros
      Database.DeleteResult[] results = Database.delete(scope, false); // Allownos some partial success
      
      // Procesar resultados
      for (Database.DeleteResult result : results) {
        if (result.isSuccess()) {
          totalRecordsDeleted++;
        } else {
          for (Database.Error error : result.getErrors()) {
            String errorMsg = 'Error al eliminar registro: ' + error.getStatusCode() + ' - ' + error.getMessage();
            System.debug(LoggingLevel.ERROR, errorMsg);
            errors.add(errorMsg);
          }
        }
      }
    } catch (Exception e) {
      String errorMsg = 'Error en ejecución de lote: ' + e.getMessage() + ' ' + e.getStackTraceString();
      System.debug(LoggingLevel.ERROR, errorMsg);
      errors.add(errorMsg);
    }
  }
  
  /**
   * @description Se ejecuta después de que todos los lotes han sido procesados
   * @param bc Contexto del trabajo por lotes
   */
  public void finish(Database.BatchableContext bc) {
    // Generar mensaje de resumen
    String summary = 'Proceso de limpieza de EventLog_Subset__c completado. ' +
                    'Registros procesados: ' + totalRecordsProcessed + '. ' +
                    'Registros eliminados: ' + totalRecordsDeleted + '. ' +
                    'Errores: ' + errors.size();
    
    System.debug(LoggingLevel.INFO, summary);
    
    // Registrar errores si los hay
    if (!errors.isEmpty()) {
      System.debug(LoggingLevel.ERROR, 'Errores encontrados durante el proceso:');
      for (String error : errors) {
        System.debug(LoggingLevel.ERROR, error);
      }
    }
    
    // Aquí se podría añadir código para enviar notificaciones, crear registros de auditoría, etc.
    try {
      sendCompletionNotification(summary, errors);
    } catch (Exception e) {
      System.debug(LoggingLevel.ERROR, 'Error al enviar notificación: ' + e.getMessage());
    }
  }
  
  /**
   * @description Envía una notificación con el resumen de la ejecución del trabajo
   * @param summary Resumen de la ejecución
   * @param errors Lista de errores encontrados
   */
  private void sendCompletionNotification(String summary, List<String> errors) {
    // Implementación del envío de notificación
    // Esto podría incluir envío de email, creación de registro de log, etc.
    
    // Ejemplo: Crear un registro de log
    /*
    Event_Log__c log = new Event_Log__c(
      Type__c = 'Batch Execution',
      Level__c = errors.isEmpty() ? 'Info' : 'Error',
      Message__c = summary,
      Details__c = String.join(errors, '\n').left(32000) // Límite de caracteres para campo de texto largo
    );
    
    insert log;
    */
  }
}
```

## Ejemplos de Uso

### Uso Básico Directamente

```apex
// Construir la consulta para seleccionar registros de más de un día de antigüedad
Datetime cutoffDate = Datetime.now().addDays(-1);
String formattedCutoffDate = cutoffDate.format('yyyy-MM-dd\'T\'HH:mm:ss\'Z\'');
String query = 'SELECT Id, CreatedDate FROM EventLog_Subset__c WHERE CreatedDate <= ' + formattedCutoffDate;

// Ejecutar el trabajo por lotes con un tamaño de lote de 150
Id batchId = Database.executeBatch(new EventLog_Subset_CleanupBatchClass(query), 150);

System.debug('Trabajo por lotes iniciado con ID: ' + batchId);
```

### Uso desde una Clase Programable

```apex
public class EventLog_Subset_CleanupScheduleClass implements Schedulable {
  public void execute(SchedulableContext sc) {
    // Verificar configuración (ejemplo simplificado)
    Boolean shouldRun = true;
    
    if (shouldRun) {
      // Construir consulta para registros de más de un día
      Datetime cutoffDate = Datetime.now().addDays(-1);
      String formattedCutoffDate = cutoffDate.format('yyyy-MM-dd\'T\'HH:mm:ss\'Z\'');
      String query = 'SELECT Id, CreatedDate FROM EventLog_Subset__c WHERE CreatedDate <= ' + formattedCutoffDate;
      
      // Ejecutar el trabajo por lotes
      Id batchId = Database.executeBatch(new EventLog_Subset_CleanupBatchClass(query), 150);
      
      System.debug('Trabajo de limpieza programado iniciado con ID: ' + batchId);
    }
  }
}
```

### Programación del Trabajo

```apex
// Programar el trabajo para ejecutarse todos los días a la 1 AM
String jobId = System.schedule(
    'EventLog Subset Cleanup - Daily',
    '0 0 1 * * ?',
    new EventLog_Subset_CleanupScheduleClass()
);

System.debug('Trabajo programado con ID: ' + jobId);
```

## Consideraciones de Rendimiento

1. **Tamaño del Lote**: La clase está diseñada para ser ejecutada con un tamaño de lote específico (150 registros según se infiere de la clase programable relacionada). Este valor puede ajustarse según las necesidades específicas y las características de los registros.
2. **Consulta Eficiente**: La eficiencia de la consulta SOQL proporcionada afectará el rendimiento del trabajo. Es importante asegurarse de que la consulta esté optimizada y utilice índices cuando sea posible.
3. **Operaciones DML**: La clase realiza operaciones de eliminación directamente, lo que consume límites de DML. Asegúrese de que el tamaño de lote sea apropiado para no exceder estos límites.
4. **Datos en Cascada**: Si los registros que se eliminan tienen relaciones y están configurados para eliminación en cascada, esto podría aumentar el consumo de recursos y afectar el rendimiento.

## Pruebas Unitarias

A continuación se presenta un ejemplo de cómo podrían ser las pruebas unitarias para esta clase:

```apex
@IsTest
private class EventLog_Subset_CleanupBatchClassTest {
    
    @TestSetup
    static void setupTestData() {
        // Crear registros de prueba
        List<EventLog_Subset__c> testLogs = new List<EventLog_Subset__c>();
        for (Integer i = 0; i < 200; i++) {
            testLogs.add(new EventLog_Subset__c());
        }
        
        insert testLogs;
    }
    
    @IsTest
    static void testBatchExecution() {
        // Verificar que existen registros antes de la prueba
        System.assertEquals(200, [SELECT COUNT() FROM EventLog_Subset__c], 'Deberían existir 200 registros antes de la prueba');
        
        // Construir consulta para seleccionar todos los registros
        String query = 'SELECT Id FROM EventLog_Subset__c';
        
        Test.startTest();
        // Ejecutar el trabajo por lotes
        Id batchId = Database.executeBatch(new EventLog_Subset_CleanupBatchClass(query), 50);
        Test.stopTest();
        
        // Verificar que todos los registros han sido eliminados
        System.assertEquals(0, [SELECT COUNT() FROM EventLog_Subset__c], 'Todos los registros deberían haber sido eliminados');
    }
    
    @IsTest
    static void testBatchWithNoRecords() {
        // Eliminar todos los registros creados en setup
        delete [SELECT Id FROM EventLog_Subset__c];
        
        // Verificar que no hay registros antes de la prueba
        System.assertEquals(0, [SELECT COUNT() FROM EventLog_Subset__c], 'No deberían existir registros antes de la prueba');
        
        // Construir consulta para seleccionar todos los registros (que serán ninguno)
        String query = 'SELECT Id FROM EventLog_Subset__c';
        
        Test.startTest();
        // Ejecutar el trabajo por lotes
        Id batchId = Database.executeBatch(new EventLog_Subset_CleanupBatchClass(query), 50);
        Test.stopTest();
        
        // Verificar que no hay errores (el trabajo debería completarse sin problemas)
        // No hay aserciones específicas aquí ya que solo estamos verificando que no hay excepciones
    }
    
    @IsTest
    static void testBatchWithInvalidQuery() {
        // Construir una consulta inválida
        String invalidQuery = 'SELECT InvalidField FROM EventLog_Subset__c';
        
        Test.startTest();
        
        // Capturar la excepción que debería ocurrir
        try {
            Id batchId = Database.executeBatch(new EventLog_Subset_CleanupBatchClass(invalidQuery), 50);
            System.assert(false, 'Debería haber lanzado una excepción');
        } catch (Exception e) {
            // Verificar que se lanzó la excepción esperada
            System.assert(e.getMessage().contains('InvalidField'), 'La excepción debería estar relacionada con el campo inválido');
        }
        
        Test.stopTest();
    }
}
```

## Notas Adicionales
1. La clase está diseñada con un propósito específico: eliminar registros de `EventLog_Subset__c` según una consulta proporcionada.
2. La implementación actual es muy simple y directa, sin manejo de errores o registro detallado de actividad.
3. El nombre de la clase sugiere que es parte de un sistema de mantenimiento para limpiar registros antiguos del objeto `EventLog_Subset__c`.
4. Esta clase trabaja en conjunto con `EventLog_Subset_CleanupScheduleClass`, que es la clase programable que la configura y ejecuta.
5. La simplicidad de la implementación actual puede ser adecuada si los volúmenes de datos son pequeños y el riesgo de errores es bajo, pero para entornos de producción con grandes volúmenes de datos, se recomendaría una implementación más robusta como la propuesta en la sección "Implementación Mejorada".