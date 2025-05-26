# Documentación Técnica: EventLog_Subset_CleanupScheduleClass

## Descripción General
`EventLog_Subset_CleanupScheduleClass` es una clase que implementa la interfaz `Schedulable` de Salesforce. Esta clase está diseñada para programar y ejecutar automáticamente la limpieza de registros antiguos del objeto `EventLog_Subset__c`, eliminando entradas que tienen más de un día de antigüedad.

## Última Modificación
7 de marzo de 2025

## Implementación de Interfaz
- **Interfaz Implementada**: `Schedulable`
- **Propósito**: Permite que la clase sea programada para ejecución utilizando el sistema de trabajos programados de Salesforce.

## Método Principal

### `execute(SchedulableContext sc)`

#### Descripción
Este método es requerido por la interfaz `Schedulable` y se ejecuta automáticamente según la programación establecida. Configura y ejecuta un trabajo por lotes para eliminar registros antiguos de `EventLog_Subset__c`.

#### Parámetros
- `sc` (SchedulableContext): El contexto de programación proporcionado por el sistema de Salesforce.

#### Retorno
- `void`

#### Proceso
1. Obtiene la configuración desde el metadato personalizado `Suratech_Foundation_Schedule_Event__mdt` con el nombre desarrollador 'schedule'.
2. En caso de estar en un contexto de prueba, crea una configuración por defecto.
3. Verifica si la configuración existe, lanzando una excepción si no se encuentra.
4. Comprueba si la programación está activa según la configuración.
5. Si está activa:
    - Calcula la fecha de un día atrás.
    - Formatea la fecha para su uso en una consulta SOQL.
    - Construye una consulta para seleccionar registros de `EventLog_Subset__c` creados antes o en la fecha calculada.
    - Ejecuta un trabajo por lotes (`EventLog_Subset_CleanupBatchClass`) con la consulta construida y un tamaño de lote de 150 registros.

#### Código
```apex
public void execute(SchedulableContext sc) {
  Suratech_Foundation_Schedule_Event__mdt config = Suratech_Foundation_Schedule_Event__mdt.getInstance(
    'schedule'
  );

  if (Test.isRunningTest()) {
    config = new Suratech_Foundation_Schedule_Event__mdt(
      MasterLabel = 'DefaultConfig',
      DeveloperName = 'DefaultConfig',
      Frequency__c = '',
      Is_Active__c = true,
      Schedule_Job_Id__c = ''
    );
  }

  if (config == null) {
    throw new MetadataNotFoundException(
      'No Suratech_Foundation_Schedule_Event__mdt Custom Metadata found'
    );
  }

  if (config.Is_Active__c) {
    Datetime todaySubstracted = dateTime.now().addDays(-1);
    String formattedDatetime = todaySubstracted.format(
      'yyyy-MM-dd\'T\'HH:mm:ss\'Z\''
    );
    String query =
      'SELECT Id, CreatedDate FROM EventLog_Subset__c WHERE CreatedDate<=' +
      formattedDatetime;

    Id batchId = Database.executeBatch(
      new EventLog_Subset_CleanupBatchClass(query),
      150
    );
  }
}
```

## Excepciones

### `MetadataNotFoundException`
Esta excepción personalizada se lanza cuando no se encuentra la configuración de metadatos personalizada necesaria para la ejecución del trabajo programado.

## Dependencias
- `Schedulable`: Interfaz estándar de Salesforce para trabajos programados.
- `Suratech_Foundation_Schedule_Event__mdt`: Tipo de metadatos personalizado que contiene la configuración para el trabajo programado.
- `EventLog_Subset_CleanupBatchClass`: Clase de lotes que realiza la eliminación efectiva de los registros.
- `EventLog_Subset__c`: Objeto personalizado cuyos registros antiguos se eliminarán.
- `MetadataNotFoundException`: Excepción personalizada utilizada cuando no se encuentra la configuración.

## Configuración de Metadatos Personalizada

### `Suratech_Foundation_Schedule_Event__mdt`
La clase depende de un registro de metadatos personalizado con el nombre desarrollador 'schedule' que tiene los siguientes campos:
- `MasterLabel`: Etiqueta principal del registro.
- `DeveloperName`: Nombre de desarrollador único.
- `Frequency__c`: Frecuencia de ejecución (no utilizado directamente en el código).
- `Is_Active__c`: Indicador booleano que determina si la programación está activa.
- `Schedule_Job_Id__c`: ID del trabajo programado (no utilizado directamente en el código).

## Comportamiento en Contexto de Prueba
En un contexto de prueba (`Test.isRunningTest()`), la clase crea una configuración por defecto con valores predefinidos para evitar depender de la existencia de registros de metadatos personalizados durante las pruebas.

## Ejemplo de Uso

### Programación Manual
```apex
// Programar la ejecución diaria a la 1:00 AM
String jobId = System.schedule(
    'EventLog Subset Cleanup - Daily',
    '0 0 1 * * ?',
    new EventLog_Subset_CleanupScheduleClass()
);
```

### Programación Dinámica
```apex
// Obtener la configuración actual
Suratech_Foundation_Schedule_Event__mdt config = Suratech_Foundation_Schedule_Event__mdt.getInstance('schedule');

// Verificar si existe y está activa
if (config != null && config.Is_Active__c) {
    // Utilizar la frecuencia configurada, o usar un valor predeterminado
    String cronExpression = !String.isBlank(config.Frequency__c) ? 
                            config.Frequency__c : '0 0 1 * * ?'; // 1 AM diariamente por defecto
    
    // Abortar cualquier trabajo existente
    if (!String.isBlank(config.Schedule_Job_Id__c)) {
        try {
            System.abortJob(config.Schedule_Job_Id__c);
        } catch (Exception e) {
            // Manejar el caso donde el trabajo ya no existe
        }
    }
    
    // Programar el nuevo trabajo
    String jobId = System.schedule(
        'EventLog Subset Cleanup - ' + System.now(),
        cronExpression,
        new EventLog_Subset_CleanupScheduleClass()
    );
    
    // Actualizar el ID del trabajo en la configuración
    // (Requiere una implementación personalizada para actualizar metadatos)
    updateScheduleJobId(jobId);
}
```

## Clase de Lotes Relacionada (Referencia)

### `EventLog_Subset_CleanupBatchClass`
Esta clase de lotes es la encargada de ejecutar la eliminación real de los registros seleccionados por la consulta.

```apex
public class EventLog_Subset_CleanupBatchClass implements Database.Batchable<sObject> {
    private String query;
    
    public EventLog_Subset_CleanupBatchClass(String query) {
        this.query = query;
    }
    
    public Database.QueryLocator start(Database.BatchableContext bc) {
        return Database.getQueryLocator(query);
    }
    
    public void execute(Database.BatchableContext bc, List<EventLog_Subset__c> scope) {
        if (!scope.isEmpty()) {
            delete scope;
        }
    }
    
    public void finish(Database.BatchableContext bc) {
        // Acciones posteriores a la eliminación, si son necesarias
    }
}
```

## Nota: No incluida en este código, pero referenciada

## Impacto y Consideraciones

### Rendimiento y Gobernadores
- El tamaño del lote está establecido en 150 registros, lo que permite un equilibrio entre velocidad de procesamiento y riesgo de alcanzar límites de gobernadores.
- La eliminación de registros se realiza en lotes para evitar alcanzar el límite de registros DML en una sola transacción.

### Mantenimiento de Datos
- La clase elimina registros que tienen más de un día de antigüedad, lo que ayuda a controlar el tamaño de la base de datos.
- Es importante asegurarse de que los registros de `EventLog_Subset__c` que deben conservarse por más tiempo estén adecuadamente protegidos o archivados antes de que se ejecute esta limpieza.

### Programación
- La frecuencia real de ejecución depende de cómo se programe la clase a través del sistema de trabajos programados de Salesforce.
- Se recomienda programarla para ejecutarse durante horas de baja actividad para minimizar el impacto en el rendimiento del sistema.

## Notas Adicionales
1. La clase utiliza un enfoque configurable a través de metadatos personalizados, lo que permite activar o desactivar la limpieza sin necesidad de modificar el código.
2. La construcción de la consulta SOQL concatena directamente la fecha formateada, lo que es seguro en este contexto ya que la fecha se genera internamente y no proviene de entrada de usuario.
3. La clase no maneja directamente errores durante la ejecución del lote, confiando en el comportamiento predeterminado de los lotes de Salesforce.
4. La configuración en metadatos personalizados permite una fácil adaptación a diferentes entornos (desarrollo, prueba, producción), ya que cada entorno puede tener su propia configuración.