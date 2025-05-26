# EventLogManagerProvider

## Descripción General
`EventLogManagerProvider` es una clase que proporciona funcionalidad para generar registros de eventos de prueba o datos de muestra en el objeto Big Object `EventLog__b`. Esta clase está diseñada para facilitar la carga masiva de datos de eventos simulados con valores aleatorios dentro de un conjunto predefinido de opciones.

## Autor
Soulberto Lorenzo \<soulberto@cloudblue.us\>

## Última Modificación
30 de octubre de 2024

## Estructura de la Clase

```apex
public with sharing class EventLogManagerProvider {
  public static void generate() {
    // Implementación del método para generar registros de eventos
  }
}
```

## Métodos

### `generate()`

#### Descripción
Método estático que genera y guarda un conjunto de registros de eventos simulados (hasta 5000) en el objeto Big Object `EventLog__b`. Los registros se generan con valores aleatorios para diversos campos, seleccionados de listas predefinidas.

#### Parámetros
Ninguno.

#### Retorno
- `void`: Este método no devuelve ningún valor.

#### Proceso
1. Inicializa un array para almacenar los resultados de las operaciones de guardado y una lista para acumular los registros de eventos.
2. Define listas constantes con valores predefinidos para diferentes campos:
    - `PRODUCTS`: Lista de productos ('Moto', 'Auto', 'Arrendmiento', 'Viaje').
    - `LEVELS`: Lista de niveles de registro ('Info', 'Warning', 'Error', 'Alert').
    - `PLATFORMS`: Lista de plataformas ('Salesforce', 'Mulesoft').
    - `TYPES`: Lista de tipos de evento ('INFO', 'ERROR').
    - `PROCESSES`: Lista de procesos ('Knowing', 'Rating', 'Quoting', 'Issuing', 'Notifying').
3. Itera hasta 5000 veces, generando en cada iteración:
    - Selecciona valores aleatorios de cada lista para crear un registro de evento único.
    - Crea un nuevo registro `EventLog__b` con los valores seleccionados y la fecha actual.
    - Añade el registro a la lista de eventos.
    - Cuando la lista alcanza 200 registros, los inserta en la base de datos en modo inmediato y limpia la lista.
4. Después del bucle, inserta cualquier registro restante (menos de 200) en la base de datos.

#### Código
```apex
public static void generate() {
  Database.SaveResult[] results;
  List<EventLog__b> eventLogs = new List<EventLog__b>();

  final List<String> PRODUCTS = new List<String>{
    'Moto',
    'Auto',
    'Arrendmiento',
    'Viaje'
  };
  final List<String> LEVELS = new List<String>{
    'Info',
    'Warning',
    'Error',
    'Alert'
  };
  final List<String> PLATFORMS = new List<String>{ 'Salesforce', 'Mulesoft' };
  final List<String> TYPES = new List<String>{ 'INFO', 'ERROR' };
  final List<String> PROCESSES = new List<String>{
    'Knowing',
    'Rating',
    'Quoting',
    'Issuing',
    'Notifying'
  };

  for (Integer i = 0; i < 5000; i++) {
    String product = PRODUCTS.get(
      (Math.random() * PRODUCTS.size()).intValue()
    );
    String platform = PLATFORMS.get(
      (Math.random() * PLATFORMS.size()).intValue()
    );
    String level = LEVELS.get((Math.random() * LEVELS.size()).intValue());
    String type = TYPES.get((Math.random() * TYPES.size()).intValue());
    String process = PROCESSES.get(
      (Math.random() * PROCESSES.size()).intValue()
    );

    EventLog__b eventLog = new EventLog__b(
      Date__c = System.now(),
      Product__c = product,
      Level__c = level,
      Platform__c = Platform,
      Process__c = process,
      Type__c = type
    );
    eventLogs.add(eventLog);

    // Insert records in batches
    if (eventLogs.size() == 200) {
      Database.insertImmediate(eventLogs);
      eventLogs.clear();
    }
  }

  // Insert any remaining records
  if (!eventLogs.isEmpty()) {
    results = Database.insertImmediate(eventLogs);
  }
}
```

## Dependencias
- `EventLog__b`: Objeto Big Object que almacena los registros de eventos. Este objeto debe estar definido en la organización con los siguientes campos:
    - `Date__c`: Campo de fecha y hora para el momento del evento.
    - `Product__c`: Campo de texto para el producto relacionado.
    - `Level__c`: Campo de texto para el nivel de importancia del evento.
    - `Platform__c`: Campo de texto para la plataforma donde ocurrió el evento.
    - `Process__c`: Campo de texto para el proceso relacionado.
    - `Type__c`: Campo de texto para el tipo de evento.

## Observaciones sobre el Código

### Puntos Fuertes
1. **Eficiencia en inserción masiva**: Utiliza un enfoque por lotes (batch) para insertar registros, lo que es más eficiente que insertar registros individualmente.
2. **Versatilidad en datos generados**: Crea una variedad de combinaciones de datos utilizando selección aleatoria de valores predefinidos.
3. **Uso de Big Objects**: Aprovecha la capacidad de Big Objects para almacenar grandes volúmenes de datos de eventos.
4. **Método inmediato de inserción**: Utiliza `Database.insertImmediate()`, que es el método adecuado para insertar registros en Big Objects.

### Posibles Errores y Mejoras
1. **Error de variable**: En la asignación `Platform__c = Platform`, se usa `Platform` (mayúscula) como valor, pero la variable es `platform` (minúscula). Esto provocará un error en tiempo de ejecución.
2. **Manejo de errores**: No hay manejo de errores para las operaciones de inserción. Sería recomendable añadir código para verificar los resultados de `Database.insertImmediate()`.
3. **Número fijo de registros**: La generación de 5000 registros está codificada de forma fija. Podría ser beneficioso hacer este número configurable.
4. **Distribución aleatoria simple**: Utiliza una distribución uniforme simple para seleccionar valores, lo que podría no reflejar patrones reales de eventos en un sistema de producción.
5. **No verifica límites del sistema**: No verifica los límites del sistema (como límites de API, DML, etc.) antes de intentar insertar registros.

### Corrección del Error de Variable
El error en `Platform__c = Platform` debería corregirse a `Platform__c = platform` para utilizar la variable correctamente:

```apex
EventLog__b eventLog = new EventLog__b(
  Date__c = System.now(),
  Product__c = product,
  Level__c = level,
  Platform__c = platform,  // Corregido: platform en minúscula
  Process__c = process,
  Type__c = type
);
```

## Mejoras Propuestas

### Versión Mejorada con Manejo de Errores y Parámetros Configurables

```apex
public with sharing class EventLogManagerProvider {
  /**
   * @description Genera registros de eventos simulados en el objeto Big Object EventLog__b
   * @param count Número de registros a generar (predeterminado: 5000, máximo: 10000)
   * @param batchSize Tamaño del lote para inserción (predeterminado: 200, máximo: 200)
   * @return ResultInfo Información sobre el resultado de la operación
   */
  public static ResultInfo generate(Integer count, Integer batchSize) {
    // Validar y establecer valores predeterminados para los parámetros
    count = (count != null && count > 0) ? Math.min(count, 10000) : 5000;
    batchSize = (batchSize != null && batchSize > 0) ? Math.min(batchSize, 200) : 200;
    
    ResultInfo result = new ResultInfo();
    List<EventLog__b> eventLogs = new List<EventLog__b>();

    // Listas predefinidas de valores
    final List<String> PRODUCTS = new List<String>{
      'Moto', 'Auto', 'Arrendmiento', 'Viaje'
    };
    final List<String> LEVELS = new List<String>{
      'Info', 'Warning', 'Error', 'Alert'
    };
    final List<String> PLATFORMS = new List<String>{ 
      'Salesforce', 'Mulesoft' 
    };
    final List<String> TYPES = new List<String>{ 
      'INFO', 'ERROR' 
    };
    final List<String> PROCESSES = new List<String>{
      'Knowing', 'Rating', 'Quoting', 'Issuing', 'Notifying'
    };

    try {
      for (Integer i = 0; i < count; i++) {
        // Generar valores aleatorios
        String product = getRandomValue(PRODUCTS);
        String platform = getRandomValue(PLATFORMS);
        String level = getRandomValue(LEVELS);
        String type = getRandomValue(TYPES);
        String process = getRandomValue(PROCESSES);

        // Crear registro de evento
        EventLog__b eventLog = new EventLog__b(
          Date__c = System.now(),
          Product__c = product,
          Level__c = level,
          Platform__c = platform,
          Process__c = process,
          Type__c = type
        );
        eventLogs.add(eventLog);

        // Insertar registros en lotes
        if (eventLogs.size() == batchSize) {
          insertAndProcessResults(eventLogs, result);
          eventLogs.clear();
        }
      }

      // Insertar registros restantes
      if (!eventLogs.isEmpty()) {
        insertAndProcessResults(eventLogs, result);
      }
      
      result.success = true;
      result.message = 'Generación completada: ' + result.totalRecords + ' registros insertados con ' + 
                      result.failedRecords + ' fallos.';
    } catch (Exception e) {
      result.success = false;
      result.message = 'Error en la generación: ' + e.getMessage() + ' en la línea ' + e.getLineNumber();
      result.exception = e;
    }
    
    return result;
  }
  
  // Método sobrecargado para usar valores predeterminados
  public static ResultInfo generate() {
    return generate(5000, 200);
  }
  
  // Método ayudante para seleccionar un valor aleatorio de una lista
  private static String getRandomValue(List<String> values) {
    return values.get((Math.random() * values.size()).intValue());
  }
  
  // Método auxiliar para insertar registros y procesar resultados
  private static void insertAndProcessResults(List<EventLog__b> eventLogs, ResultInfo result) {
    Database.SaveResult[] saveResults = Database.insertImmediate(eventLogs);
    
    for (Database.SaveResult sr : saveResults) {
      result.totalRecords++;
      if (!sr.isSuccess()) {
        result.failedRecords++;
        for (Database.Error err : sr.getErrors()) {
          result.errors.add(err.getMessage());
        }
      }
    }
  }
  
  // Clase interna para devolver información sobre el resultado
  public class ResultInfo {
    public Boolean success = false;
    public String message = '';
    public Integer totalRecords = 0;
    public Integer failedRecords = 0;
    public List<String> errors = new List<String>();
    public Exception exception;
  }
}
```

## Ejemplo de Uso

### Uso Básico

```apex
// Generar 5000 registros usando el método predeterminado
EventLogManagerProvider.generate();
```

### Uso Avanzado

```apex
// Generar 1000 registros con un tamaño de lote de 150
EventLogManagerProvider.ResultInfo result = EventLogManagerProvider.generate(1000, 150);

// Verificar el resultado
if (result.success) {
    System.debug('Generación exitosa: ' + result.message);
} else {
    System.debug('Error en la generación: ' + result.message);
    for (String error : result.errors) {
        System.debug('Error: ' + error);
    }
}
```

### Uso en un Trabajo por Lotes

```apex
public class EventLogGeneratorBatch implements Database.Batchable<Integer>, Database.Stateful {
    private Integer totalRecords;
    private Integer batchSize;
    private List<Integer> iterations;
    private EventLogManagerProvider.ResultInfo finalResult;
    
    public EventLogGeneratorBatch(Integer totalRecords, Integer batchSize) {
        this.totalRecords = totalRecords;
        this.batchSize = batchSize;
        this.finalResult = new EventLogManagerProvider.ResultInfo();
        
        // Crear una lista de iteraciones para procesar
        this.iterations = new List<Integer>();
        Integer remainingRecords = totalRecords;
        Integer maxPerBatch = 5000; // Máximo de registros por ejecución de generate()
        
        while (remainingRecords > 0) {
            Integer recordsThisBatch = Math.min(remainingRecords, maxPerBatch);
            iterations.add(recordsThisBatch);
            remainingRecords -= recordsThisBatch;
        }
    }
    
    public List<Integer> start(Database.BatchableContext bc) {
        return iterations;
    }
    
    public void execute(Database.BatchableContext bc, List<Integer> scope) {
        for (Integer recordCount : scope) {
            EventLogManagerProvider.ResultInfo result = EventLogManagerProvider.generate(recordCount, batchSize);
            
            // Acumular resultados
            finalResult.totalRecords += result.totalRecords;
            finalResult.failedRecords += result.failedRecords;
            finalResult.errors.addAll(result.errors);
            
            if (!result.success) {
                finalResult.success = false;
                finalResult.message += ' ' + result.message;
            }
        }
    }
    
    public void finish(Database.BatchableContext bc) {
        finalResult.message = 'Generación de registros completada. Total: ' + 
                             finalResult.totalRecords + ', Fallidos: ' + 
                             finalResult.failedRecords;
        
        // Registrar resultado final
        System.debug('Resultado final: ' + finalResult.message);
    }
}

// Uso del trabajo por lotes para generar 100,000 registros
Database.executeBatch(new EventLogGeneratorBatch(100000, 200), 1);
```

## Consideraciones de Rendimiento

1. **Volumen de Datos**: Generar 5000 registros puede ser intensivo en recursos. Para volúmenes más grandes, considerar un enfoque por lotes o asíncrono.
2. **Límites de Big Objects**: BigObjects tienen límites diferentes a los objetos estándar. Verificar la documentación de Salesforce para conocer los límites actuales.
3. **Límites de Transacción**: El método `Database.insertImmediate()` cuenta para los límites de la transacción actual.
4. **Aleatoriedad**: El método `Math.random()` proporciona una distribución uniforme simple, que puede no ser óptima para todos los casos de prueba.

## Uso Típico

Esta clase es útil en varios escenarios:

1. **Configuración de Entornos**: Cargar datos de muestra en entornos de desarrollo o sandbox.
2. **Pruebas de Rendimiento**: Generar volúmenes grandes de datos para probar el rendimiento de componentes que procesan eventos.
3. **Demos y Presentaciones**: Crear datos realistas para demostraciones o presentaciones.
4. **Pruebas de Integración**: Simular eventos para probar integración con sistemas externos.

## Notas Adicionales
1. El uso de Big Objects (`EventLog__b`) es una buena elección para almacenar grandes volúmenes de datos de registro, ya que están optimizados para este propósito.
2. La clase utiliza una distribución de valores aleatoria uniforme, lo que significa que cada opción tiene la misma probabilidad de ser seleccionada. En un sistema real, algunos tipos de eventos podrían ser más comunes que otros.
3. La inserción por lotes de 200 registros es una buena práctica para evitar alcanzar los límites de Salesforce para operaciones DML.
4. La clase no incluye la generación de datos relacionados específicos del negocio o contenido detallado de mensajes de error, que podrían hacer los datos más realistas.
5. La fecha utilizada para todos los registros es la misma (`System.now()`). Para simular datos históricos, podría ser útil generar fechas variadas dentro de un rango determinado.