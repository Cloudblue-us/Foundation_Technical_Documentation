# IssueProductHandler

## Descripción General
`IssueProductHandler` es una clase virtual global que extiende `ProductHandler` y está diseñada para manejar la emisión de productos de seguros a través de procedimientos de integración. Esta clase implementa la lógica necesaria para validar datos, asignar cuentas a cotizaciones y gestionar el proceso de emisión de pólizas.

## Autor
Jean Carlos Melendez \<jean@cloudblue.us\>

## Última Modificación
21 de febrero de 2025

## Constantes

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `FLAG_PLATFORM_NAME` | `String` | Nombre del indicador de plataforma: 'sura_foundation_platform_events' |
| `IP_PREPROCESS` | `String` | Nombre del procedimiento de preprocesamiento: 'sfcore_preandpostquotingprocess' |
| `PROCESS_NAME` | `String` | Nombre del proceso: 'Issuing' |
| `OPTION_SKIP_BEHAVIOR` | `String` | Opción para omitir el comportamiento predeterminado: 'skipDefaultBehavior' |
| `ISSUING_IPROCEDURE` | `String` | Nombre del procedimiento de integración para emisión: 'SURAFoundation_IssuingProcedure' |
| `ERROR_REQUIRED_FIELD` | `String` | Mensaje de error para campos requeridos: ' es requerido' |
| `REQUIRED_FIELD_FOR_ISSUE_CORE` | `List<String>` | Lista de campos requeridos para la emisión: 'quoteId', 'effectiveDate', 'producerId' |

## Variables Globales

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `EXECUTOR` | `IntegrationProcedureExecutor` | Ejecutor de procedimientos de integración |

## Constructores

### `IssueProductHandler(String productName, IntegrationProcedureExecutor executor)`

#### Descripción
Constructor que inicializa el handler con un nombre de producto específico y un ejecutor de procedimientos de integración personalizado.

#### Parámetros
- `productName` (String): Nombre del producto a gestionar.
- `executor` (IntegrationProcedureExecutor): Ejecutor personalizado para los procedimientos de integración.

#### Código
```apex
global IssueProductHandler(
  String productName,
  IntegrationProcedureExecutor executor
) {
  super(productName);
  this.EXECUTOR = executor;
}
```

### `IssueProductHandler(String productName)`

#### Descripción
Constructor alternativo que inicializa el handler con un nombre de producto específico y un ejecutor mock por defecto.

#### Parámetros
- `productName` (String): Nombre del producto a gestionar.

#### Código
```apex
global IssueProductHandler(String productName) {
  super(productName);
  this.EXECUTOR = new MockIntegrationProcedureExecutor();
}
```

## Métodos Públicos y Globales

### `onPre(Map<String, Object> options, Map<String, Object> input, Map<String, Object> output)`

#### Descripción
Método virtual sobrescrito que se ejecuta antes del proceso principal. Realiza validaciones iniciales y asigna cuentas a la cotización.

#### Parámetros
- `options` (Map<String, Object>): Opciones adicionales para el proceso.
- `input` (Map<String, Object>): Datos de entrada para el proceso.
- `output` (Map<String, Object>): Mapa de salida donde se guardarán los resultados.

#### Retorno
- `Map<String, Object>`: Los datos de entrada, posiblemente modificados durante el pre-procesamiento.

#### Código
```apex
global virtual override Map<String, Object> onPre(
  Map<String, Object> options,
  Map<String, Object> input,
  Map<String, Object> output
) {
  validateInput(input, new List<String>{ 'leadId', 'quoteId' });
  assignAccountsToQuote(
    listInsuredPartyAccounts(input),
    (Id) input.get('quoteId')
  );
  return input;
}
```

### `assignAccountsToQuote(List<Map<String, Object>> parties, Id quoteId)`

#### Descripción
Método virtual que asigna cuentas de partes aseguradas a una cotización.

#### Parámetros
- `parties` (List<Map<String, Object>>): Lista de partes aseguradas a asignar.
- `quoteId` (Id): ID de la cotización a la que se asignarán las cuentas.

#### Retorno
- `void`

#### Código
```apex
global virtual void assignAccountsToQuote(
  List<Map<String, Object>> parties,
  Id quoteId
) {
  SObject quote = QuoteUtils.getQuoteById(quoteId);
  new QuoteUtils().setInsuredPartyToQuoteAndOpp(parties, quote);
}
```

### `listInsuredPartyAccounts(Map<String, Object> input)`

#### Descripción
Método virtual que obtiene la lista de cuentas de partes aseguradas a partir de los datos de entrada.

#### Parámetros
- `input` (Map<String, Object>): Datos de entrada que contienen información sobre las partes aseguradas.

#### Retorno
- `List<Map<String, Object>>`: Lista de mapas que representan las cuentas de partes aseguradas.

#### Código
```apex
global virtual List<Map<String, Object>> listInsuredPartyAccounts(
  Map<String, Object> input
) {
  return new LeadUtils().convertInsuredPartyFromLeadToAccount(input);
}
```

### `onPos(Map<String, Object> options, Map<String, Object> input, Map<String, Object> output)`

#### Descripción
Método virtual sobrescrito que se ejecuta después del proceso principal. Actualiza el estado de la oportunidad a "Poliza emitida".

#### Parámetros
- `options` (Map<String, Object>): Opciones adicionales para el proceso.
- `input` (Map<String, Object>): Datos de entrada para el proceso.
- `output` (Map<String, Object>): Mapa de salida con los resultados del proceso principal.

#### Retorno
- `Map<String, Object>`: Los datos de entrada, posiblemente modificados durante el post-procesamiento.

#### Código
```apex
global virtual override Map<String, Object> onPos(
  Map<String, Object> options,
  Map<String, Object> input,
  Map<String, Object> output
) {
  String opportunityId = (String) ((Map<String, Object>) input
      ?.get('resultProcess'))
    ?.get('OpportunityId');

  if (opportunityId == null) {
    throw new ProductHandlerImplementationException(
      'OpportunityId is null',
      'UNPROCESSABLE_ENTITY',
      422
    );
  }
  Opportunity opp = [
    SELECT Id, StageName
    FROM Opportunity
    WHERE Id = :opportunityId
  ];

  opp.StageName = 'Poliza emitida';
  update opp;
  return input;
}
```

### `process(Map<String, Object> options, Map<String, Object> input, Map<String, Object> output)`

#### Descripción
Método sobrescrito que orquesta el proceso principal de emisión de producto.

#### Parámetros
- `options` (Map<String, Object>): Opciones adicionales para el proceso.
- `input` (Map<String, Object>): Datos de entrada para el proceso.
- `output` (Map<String, Object>): Mapa de salida donde se guardarán los resultados.

#### Retorno
- `Map<String, Object>`: Mapa con los datos del producto emitido.

#### Código
```apex
global override Map<String, Object> process(
  Map<String, Object> options,
  Map<String, Object> input,
  Map<String, Object> output
) {
  if (options?.containsKey(OPTION_SKIP_BEHAVIOR)) {
    return input;
  }

  return executeIntegrationProcedure(this.preIssuingValidation(input));
}
```

### `executeIntegrationProcedure(Map<String, Object> ipInput)`

#### Descripción
Ejecuta el procedimiento de integración para la emisión del producto.

#### Parámetros
- `ipInput` (Map<String, Object>): Datos de entrada para el procedimiento de integración.

#### Retorno
- `Map<String, Object>`: Mapa con los resultados del procedimiento de integración.

#### Código
```apex
global Map<String, Object> executeIntegrationProcedure(
  Map<String, Object> ipInput
) {
  Map<String, Object> output = new Map<String, Object>{
    'resultProcess' => EXECUTOR.execute(ISSUING_IPROCEDURE, ipInput)
  };
  this.setResponseProcess(output);
  return output;
}
```

## Métodos Privados

### `validateInput(Map<String, Object> input, List<String> requiredFields)`

#### Descripción
Valida que los campos requeridos estén presentes en el mapa de entrada.

#### Parámetros
- `input` (Map<String, Object>): Datos de entrada a validar.
- `requiredFields` (List<String>): Lista de nombres de campos requeridos.

#### Retorno
- `void`

#### Excepciones
- `ProductHandlerImplementationException`: Si algún campo requerido está ausente o vacío.

#### Código
```apex
private void validateInput(
  Map<String, Object> input,
  List<String> requiredFields
) {
  for (String field : requiredFields) {
    if (String.isBlank(String.valueOf(input?.get(field)))) {
      throw new ProductHandlerImplementationException(
        field + ERROR_REQUIRED_FIELD,
        'UNPROCESSABLE_ENTITY',
        422
      );
    }
  }
}
```

### `preIssuingValidation(Map<String, Object> input)`

#### Descripción
Realiza validaciones específicas para el preprocesamiento de emisión, verificando la presencia de campos requeridos y las relaciones entre cotización, oportunidad y cuenta.

#### Parámetros
- `input` (Map<String, Object>): Datos de entrada a validar.

#### Retorno
- `Map<String, Object>`: Los datos de entrada validados.

#### Excepciones
- `ProductHandlerImplementationException`: Si no se encuentra el ID de oportunidad en la cotización o el ID de cuenta en la oportunidad.

#### Código
```apex
private Map<String, Object> preIssuingValidation(Map<String, Object> input) {
  validateInput(input, REQUIRED_FIELD_FOR_ISSUE_CORE);
  SObject quote = QuoteUtils.getQuoteById((Id) input.get('quoteId'));
  Id opportunityId = (Id) quote.get('OpportunityId');

  if (opportunityId == null) {
    throw new ProductHandlerImplementationException(
      'No OpportunityId found in Quote',
      'UNPROCESSABLE_ENTITY',
      422
    );
  }

  Opportunity opp = [
    SELECT Id, AccountId
    FROM Opportunity
    WHERE Id = :opportunityId
    LIMIT 1
  ];

  if (opp.AccountId == null) {
    throw new ProductHandlerImplementationException(
      'No AccountId found in Opportunity',
      'UNPROCESSABLE_ENTITY',
      422
    );
  }

  return input;
}
```

## Excepciones

### `ProductHandlerImplementationException`
Esta excepción personalizada se lanza en las siguientes situaciones:
1. Cuando un campo requerido está ausente o vacío.
2. Cuando no se encuentra el ID de oportunidad en la cotización.
3. Cuando no se encuentra el ID de cuenta en la oportunidad.
4. Cuando el ID de oportunidad es nulo en los resultados del proceso.

## Dependencias
- `ProductHandler`: Clase base que `IssueProductHandler` extiende.
- `IntegrationProcedureExecutor`: Interfaz o clase que define el método `execute` para ejecutar procedimientos de integración.
- `MockIntegrationProcedureExecutor`: Implementación mock de `IntegrationProcedureExecutor` utilizada en el constructor alternativo.
- `QuoteUtils`: Clase utilitaria para operaciones con cotizaciones.
- `LeadUtils`: Clase utilitaria para operaciones con leads.
- `ProductHandlerImplementationException`: Excepción personalizada lanzada cuando hay errores en el proceso.

## Flujo de Proceso

El proceso de emisión de producto sigue el siguiente flujo:

1. **Pre-procesamiento (`onPre`)**:
    - Validación de campos requeridos ('leadId', 'quoteId').
    - Obtención de la lista de cuentas de partes aseguradas.
    - Asignación de cuentas a la cotización.

2. **Proceso principal (`process`)**:
    - Validación específica para emisión (verificación de campos y relaciones).
    - Ejecución del procedimiento de integración para emisión.

3. **Post-procesamiento (`onPos`)**:
    - Actualización del estado de la oportunidad a "Poliza emitida".

## Ejemplo de Uso

```apex
// Crear el ejecutor de procedimientos de integración
IntegrationProcedureExecutor executor = new SomeIntegrationProcedureExecutor();

// Instanciar el handler
IssueProductHandler handler = new IssueProductHandler('AutoInsurance', executor);

// Preparar los datos de entrada
Map<String, Object> input = new Map<String, Object>{
  'leadId' => '00Q0x000000XXXXX',
  'quoteId' => '0Q00x000000XXXXX',
  'effectiveDate' => Date.today(),
  'producerId' => '0010x000000XXXXX'
};

// Mapas para opciones y salida
Map<String, Object> options = new Map<String, Object>();
Map<String, Object> output = new Map<String, Object>();

try {
  // Iniciar el flujo de emisión
  Map<String, Object> result = handler.onPre(options, input, output);
  result = handler.process(options, result, output);
  result = handler.onPos(options, result, output);
  
  // Verificar el resultado
  System.debug('Emisión completada. Resultado: ' + result);
} catch (ProductHandlerImplementationException e) {
  System.debug('Error en el proceso de emisión: ' + e.getMessage());
}
```

## Extensión de la Clase

```apex
public class SpecializedIssueHandler extends IssueProductHandler {
    
    public SpecializedIssueHandler(String productName) {
        super(productName);
    }
    
    // Personalizar el comportamiento previo a la emisión
    global override Map<String, Object> onPre(
        Map<String, Object> options,
        Map<String, Object> input,
        Map<String, Object> output
    ) {
        // Realizar validaciones adicionales específicas del producto
        if (!String.valueOf(input.get('productLine')).equals('SpecialLine')) {
            throw new ProductHandlerImplementationException(
                'Este handler solo maneja la línea de producto "SpecialLine"',
                'UNPROCESSABLE_ENTITY',
                422
            );
        }
        
        // Llamar al método base
        return super.onPre(options, input, output);
    }
    
    // Personalizar el comportamiento posterior a la emisión
    global override Map<String, Object> onPos(
        Map<String, Object> options,
        Map<String, Object> input,
        Map<String, Object> output
    ) {
        // Llamar al método base primero
        Map<String, Object> result = super.onPos(options, input, output);
        
        // Realizar acciones adicionales después de la emisión
        String opportunityId = (String) ((Map<String, Object>) input.get('resultProcess')).get('OpportunityId');
        if (opportunityId != null) {
            // Crear una tarea de seguimiento, por ejemplo
            Task followUpTask = new Task(
                WhatId = opportunityId,
                Subject = 'Seguimiento post-emisión',
                ActivityDate = Date.today().addDays(7),
                Status = 'Pendiente',
                Priority = 'Normal'
            );
            insert followUpTask;
        }
        
        return result;
    }
}
```

## Notas Adicionales
1. La clase utiliza el operador de navegación segura (`?.`) para evitar excepciones de referencia nula al acceder a valores en mapas.
2. La clase implementa un mecanismo para omitir su comportamiento predeterminado si la opción `skipDefaultBehavior` está presente.
3. El método `executeIntegrationProcedure` encapsula la ejecución del procedimiento de integración, facilitando la prueba unitaria y la extensibilidad.
4. La clase realiza validaciones exhaustivas en diferentes etapas del proceso para garantizar la integridad de los datos.
5. La estructura de la clase facilita la personalización mediante herencia, permitiendo sobrescribir los métodos virtuales para comportamientos específicos.