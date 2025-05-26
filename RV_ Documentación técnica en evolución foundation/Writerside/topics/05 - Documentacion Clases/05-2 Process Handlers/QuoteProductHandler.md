# Documentación Técnica: QuoteProductHandler

## Descripción General
`QuoteProductHandler` es una clase virtual global que extiende `ProductHandler`. Esta clase está diseñada para gestionar el proceso de cotización de productos mediante la ejecución de procedimientos de integración, validando la estructura de los datos de entrada y gestionando los resultados.

## Autor
Soulberto Lorenzo \<soulberto@cloudblue.us\>

## Última Modificación
21 de febrero de 2025 por Jean Carlos Melendez

## Constantes

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `PROCESS_NAME` | `String` | Nombre del proceso: 'QUOTING' |
| `MAIN_NODE` | `String` | Clave del nodo principal en los datos de entrada: 'quotepolicyJson' |
| `OPTION_SKIP_BEHAVIOR` | `String` | Opción para omitir el comportamiento predeterminado: 'skipDefaultBehavior' |
| `RATING_IPROCEDURE` | `String` | Nombre del procedimiento de integración para cotización: 'sfcore_quotingprocedure' |
| `REQUIRED_PARAMETERS` | `Set<String>` | Conjunto de parámetros requeridos para la cotización: 'term', 'productConfigurationDetail', 'insuredItems', 'additionalFields', 'OpportunityDetails' |

## Variables Globales

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `EXECUTOR` | `IntegrationProcedureExecutor` | Ejecutor de procedimientos de integración |

## Constructores

### `QuoteProductHandler(String productName, IntegrationProcedureExecutor executor)`

#### Descripción
Constructor que inicializa el handler con un nombre de producto específico y un ejecutor de procedimientos de integración personalizado.

#### Parámetros
- `productName` (String): Nombre del producto a gestionar.
- `executor` (IntegrationProcedureExecutor): Ejecutor personalizado para los procedimientos de integración.

#### Código
```apex
global QuoteProductHandler(
  String productName,
  IntegrationProcedureExecutor executor
) {
  super(productName);
  this.EXECUTOR = executor;
}
```

### `QuoteProductHandler(String productName)`

#### Descripción
Constructor alternativo que inicializa el handler con un nombre de producto específico y un ejecutor mock por defecto.

#### Parámetros
- `productName` (String): Nombre del producto a gestionar.

#### Código
```apex
global QuoteProductHandler(String productName) {
  super(productName);
  this.EXECUTOR = new MockIntegrationProcedureExecutor();
}
```

## Métodos

### `setObservers()`

#### Descripción
Método sobrescrito que configura los observadores para el handler, específicamente añadiendo un `QuoteObserver`.

#### Retorno
- `void`

#### Código
```apex
public override void setObservers() {
  this.addObserver(new QuoteObserver());
}
```

### `process(Map<String, Object> options, Map<String, Object> input, Map<String, Object> output)`

#### Descripción
Método principal sobrescrito que procesa la cotización del producto utilizando un procedimiento de integración y gestiona los resultados.

#### Parámetros
- `options` (Map<String, Object>): Opciones adicionales para el proceso.
- `input` (Map<String, Object>): Datos de entrada para el proceso.
- `output` (Map<String, Object>): Mapa de salida donde se guardarán los resultados.

#### Retorno
- `Map<String, Object>`: Mapa con los datos de la cotización creada.

#### Proceso
1. Verifica si se debe omitir el comportamiento predeterminado según las opciones.
2. Valida la estructura de los datos de entrada utilizando `validateInputMapForQuoting`.
3. Si la validación falla, lanza una excepción con el mensaje de error.
4. Establece el nombre del proceso como 'Quoting'.
5. Ejecuta el procedimiento de integración.
6. Actualiza el mapa de salida con los resultados.
7. Establece el proceso de respuesta.
8. Añade los datos de entrada originales al mapa de salida.
9. Devuelve el mapa de salida completo.

#### Código
```apex
public override Map<String, Object> process(
  Map<String, Object> options,
  Map<String, Object> input,
  Map<String, Object> output
) {
  if (options?.containsKey(OPTION_SKIP_BEHAVIOR)) {
    return input;
  }

  Map<Boolean, Object> validation = QuoteProductHandler.validateInputMapForQuoting(
    input
  );
  if (validation.containsKey(false)) {
    throw new ProductHandlerImplementationException(
      validation.get(false).toString(),
      'UNPROCESSABLE_ENTITY',
      422
    );
  }

  this.setProcessName('Quoting');

  Map<String, Object> resultProcess = executeIntegrationProcedure(input);
  Core.getSymmaryLimits();
  output.clear();
  output.putAll(resultProcess);
  this.setResponseProcess(output);
  output.putAll(input);

  return output;
}
```

### `executeIntegrationProcedure(Map<String, Object> ipInput)`

#### Descripción
Ejecuta el procedimiento de integración para la cotización del producto.

#### Parámetros
- `ipInput` (Map<String, Object>): Datos de entrada para el procedimiento de integración.

#### Retorno
- `Map<String, Object>`: Mapa con los resultados del procedimiento de integración.

#### Código
```apex
public Map<String, Object> executeIntegrationProcedure(
  Map<String, Object> ipInput
) {
  return new Map<String, Object>{
    'resultProcess' => EXECUTOR.execute(RATING_IPROCEDURE, ipInput)
  };
}
```

### `validateInputMapForQuoting(Map<String, Object> inputMap)`

#### Descripción
Método estático que verifica si todos los nodos principales requeridos están presentes en el mapa de entrada.

#### Parámetros
- `inputMap` (Map<String, Object>): Mapa de entrada que contiene los datos a validar.

#### Retorno
- `Map<Boolean, Object>`: Mapa con el resultado de la validación. La clave es un booleano que indica si la validación fue exitosa, y el valor es un mensaje descriptivo.

#### Proceso
1. Verifica la existencia del nodo principal (`MAIN_NODE`).
2. Obtiene el mapa correspondiente al nodo principal.
3. Verifica la existencia de cada uno de los parámetros requeridos definidos en `REQUIRED_PARAMETERS`.
4. Devuelve un mapa con el resultado de la validación y un mensaje descriptivo.

#### Código
```apex
public static Map<Boolean, Object> validateInputMapForQuoting(
  Map<String, Object> inputMap
) {
  if (!inputMap.containsKey(MAIN_NODE) || inputMap?.get(MAIN_NODE) == null) {
    return createValidationResult(
      false,
      'Falta el nodo principal requerido: ' + MAIN_NODE
    );
  }
  Map<String, Object> mainNode = (Map<String, Object>) inputMap.get(
    MAIN_NODE
  );

  for (String node : REQUIRED_PARAMETERS) {
    if (!mainNode.containsKey(node)) {
      return createValidationResult(
        false,
        'Falta el nodo requerido: ' + node
      );
    }
  }
  System.debug('Todos los nodos requeridos están presentes.');
  return createValidationResult(
    true,
    'Todos los parámetros requeridos están presentes.'
  );
}
```

### `createValidationResult(Boolean isValid, String message)`

#### Descripción
Método privado estático que crea un mapa estándar para representar el resultado de una validación.

#### Parámetros
- `isValid` (Boolean): Indicador de si la validación fue exitosa.
- `message` (String): Mensaje descriptivo del resultado.

#### Retorno
- `Map<Boolean, Object>`: Mapa con el estado de validación y el mensaje.

#### Código
```apex
private static Map<Boolean, Object> createValidationResult(
  Boolean isValid,
  String message
) {
  return new Map<Boolean, Object>{ isValid => message };
}
```

## Excepciones

### `ProductHandlerImplementationException`
Esta excepción se lanza cuando la validación de los datos de entrada falla, indicando qué nodo requerido está faltando. Se lanza con el código HTTP 422 (UNPROCESSABLE_ENTITY).

## Dependencias
- `ProductHandler`: Clase base que `QuoteProductHandler` extiende.
- `IntegrationProcedureExecutor`: Interfaz o clase que define el método `execute` para ejecutar procedimientos de integración.
- `MockIntegrationProcedureExecutor`: Implementación mock de `IntegrationProcedureExecutor` utilizada en el constructor alternativo.
- `QuoteObserver`: Clase observadora añadida en el método `setObservers()`.
- `Core`: Clase utilitaria que proporciona métodos como `getSymmaryLimits()`.
- `ProductHandlerImplementationException`: Excepción personalizada lanzada cuando hay errores en el proceso.

## Patrones de Diseño Implementados

### Observer
A través del método `setObservers()`, la clase implementa el patrón Observer, permitiendo que otros objetos (específicamente, `QuoteObserver`) sean notificados de eventos durante el procesamiento.

### Strategy
La clase utiliza el patrón Strategy al recibir un `IntegrationProcedureExecutor` que puede tener diferentes implementaciones, permitiendo cambiar el comportamiento de ejecución de procedimientos de integración sin modificar el código de la clase.

## Flujo de Proceso

El proceso de cotización sigue el siguiente flujo:

1. **Validación**: Verifica que todos los nodos y parámetros requeridos estén presentes en los datos de entrada.
2. **Preparación**: Establece el nombre del proceso como 'Quoting'.
3. **Ejecución**: Llama al procedimiento de integración a través del executor.
4. **Gestión de Resultados**: Actualiza el mapa de salida con los resultados y añade los datos de entrada originales.
5. **Notificación**: Notifica a los observadores registrados a través del método heredado `setResponseProcess`.

## Estructura de Datos de Entrada

Para que el proceso de cotización funcione correctamente, los datos de entrada deben tener la siguiente estructura:

```json
{
  "quotepolicyJson": {
    "term": { /* ... */ },
    "productConfigurationDetail": { /* ... */ },
    "insuredItems": [ /* ... */ ],
    "additionalFields": { /* ... */ },
    "OpportunityDetails": { /* ... */ }
  },
  /* Otros campos opcionales */
}
```

## Ejemplo de Uso

```apex
// Crear el ejecutor de procedimientos de integración
IntegrationProcedureExecutor executor = new SomeIntegrationProcedureExecutor();

// Instanciar el handler
QuoteProductHandler handler = new QuoteProductHandler('AutoInsurance', executor);

// Preparar los datos de entrada
Map<String, Object> term = new Map<String, Object>{
  'startDate' => Date.today(),
  'endDate' => Date.today().addYears(1)
};

Map<String, Object> productConfigDetail = new Map<String, Object>{
  'productCode' => 'AUTO-001',
  'coverageLevel' => 'Premium'
};

List<Map<String, Object>> insuredItems = new List<Map<String, Object>>{
  new Map<String, Object>{
    'type' => 'Vehicle',
    'make' => 'Toyota',
    'model' => 'Corolla',
    'year' => 2020
  }
};

Map<String, Object> additionalFields = new Map<String, Object>{
  'discountCode' => 'NEWCUST10'
};

Map<String, Object> opportunityDetails = new Map<String, Object>{
  'opportunityId' => '0061x00000AbCdEf',
  'accountId' => '0011x00000GhIjKl'
};

Map<String, Object> quotepolicyJson = new Map<String, Object>{
  'term' => term,
  'productConfigurationDetail' => productConfigDetail,
  'insuredItems' => insuredItems,
  'additionalFields' => additionalFields,
  'OpportunityDetails' => opportunityDetails
};

Map<String, Object> input = new Map<String, Object>{
  'quotepolicyJson' => quotepolicyJson
};

// Mapas para opciones y salida
Map<String, Object> options = new Map<String, Object>();
Map<String, Object> output = new Map<String, Object>();

try {
  // Procesar la cotización
  Map<String, Object> result = handler.process(options, input, output);
  
  // Obtener los resultados
  Map<String, Object> resultProcess = (Map<String, Object>)result.get('resultProcess');
  System.debug('Cotización completada. Resultado: ' + resultProcess);
} catch (ProductHandlerImplementationException e) {
  System.debug('Error en el proceso de cotización: ' + e.getMessage());
}
```

## Extensión de la Clase

```apex
public class SpecializedQuoteHandler extends QuoteProductHandler {
    
    public SpecializedQuoteHandler(String productName) {
        super(productName);
    }
    
    // Sobrescribir el método para añadir validaciones adicionales
    public override Map<String, Object> process(
        Map<String, Object> options,
        Map<String, Object> input,
        Map<String, Object> output
    ) {
        // Realizar validaciones adicionales específicas
        Map<String, Object> quotepolicyJson = (Map<String, Object>)input.get(MAIN_NODE);
        if (quotepolicyJson != null) {
            Map<String, Object> term = (Map<String, Object>)quotepolicyJson.get('term');
            if (term != null) {
                Date startDate = (Date)term.get('startDate');
                Date endDate = (Date)term.get('endDate');
                
                if (startDate != null && endDate != null && startDate > endDate) {
                    throw new ProductHandlerImplementationException(
                        'La fecha de inicio no puede ser posterior a la fecha de fin',
                        'UNPROCESSABLE_ENTITY',
                        422
                    );
                }
            }
        }
        
        // Llamar al método base para continuar con el procesamiento estándar
        return super.process(options, input, output);
    }
    
    // Sobrescribir el método para personalizar la ejecución del procedimiento
    public override Map<String, Object> executeIntegrationProcedure(
        Map<String, Object> ipInput
    ) {
        // Agregar datos adicionales al input antes de ejecutar
        Map<String, Object> quotepolicyJson = (Map<String, Object>)ipInput.get(MAIN_NODE);
        Map<String, Object> additionalFields = (Map<String, Object>)quotepolicyJson.get('additionalFields');
        
        // Añadir un campo adicional
        additionalFields.put('handlerType', 'Specialized');
        
        // Ejecutar el procedimiento y procesar los resultados
        Map<String, Object> results = super.executeIntegrationProcedure(ipInput);
        
        // Realizar procesamientos adicionales en los resultados si es necesario
        
        return results;
    }
}
```

## Notas Adicionales
1. La clase utiliza el operador de navegación segura (`?.`) para evitar excepciones de referencia nula al acceder a valores en mapas.
2. La clase implementa un mecanismo para omitir su comportamiento predeterminado si la opción `skipDefaultBehavior` está presente.
3. El método `executeIntegrationProcedure` encapsula la ejecución del procedimiento de integración, facilitando la prueba unitaria y la extensibilidad.
4. La validación de la estructura de datos de entrada se realiza a través de un método estático, lo que permite su reutilización en otras partes del código.
5. La salida del proceso incluye tanto los resultados del procedimiento de integración como los datos de entrada originales, proporcionando un contexto completo.
6. La llamada a `Core.getSymmaryLimits()` sugiere que se está realizando algún tipo de monitoreo o registro de los límites de recursos durante el proceso.