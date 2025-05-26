# Documentación Técnica: QuoteUtils

## Descripción General
`QuoteUtils` es una clase virtual global que proporciona utilidades para trabajar con cotizaciones (Quote) en Salesforce. Esta clase ofrece métodos para validar, actualizar y consultar cotizaciones, así como para establecer relaciones entre cotizaciones, cuentas y oportunidades.

## Autor
Jean Carlos Melendez \<jean@cloudblue.us\>

## Última Modificación
31 de enero de 2025 por Esneyder Zabala

## Constantes

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `MAIN_NODE` | `String` | Clave del nodo principal en los datos de entrada para cotizaciones: 'quotepolicyJson' |
| `REQUIRED_PARAMETERS` | `Set<String>` | Conjunto de parámetros requeridos para la cotización: 'term', 'productConfigurationDetail', 'insuredItems', 'additionalFields', 'OpportunityDetails' |

## Métodos

### `isValidSalesforceId(String objectId)`

#### Descripción
Comprueba si un ID proporcionado es un ID de Salesforce válido que corresponde a algún objeto.

#### Parámetros
- `objectId` (String): El ID de Salesforce a validar.

#### Retorno
- `Boolean`: `true` si el ID es válido y corresponde a un objeto, `false` en caso contrario.

#### Proceso
1. Intenta convertir la cadena a un ID de Salesforce.
2. Si la conversión es exitosa, obtiene el tipo de objeto asociado al ID.
3. Verifica que el tipo de objeto no esté en blanco.
4. Si ocurre alguna excepción durante el proceso, retorna `false`.

#### Código
```apex
global static Boolean isValidSalesforceId(String objectId) {
  try {
    Id validId = Id.valueOf(objectId); // Lanza excepción si el ID no es válido
    String sObjectType = validId.getSObjectType().getDescribe().getName();
    return String.isNotBlank(sObjectType);
  } catch (Exception e) {
    return false; // El ID no es válido o no pertenece a un objeto conocido
  }
}
```

### `validateInputMapForQuoting(Map<String, Object> inputMap)`

#### Descripción
Verifica si todos los nodos principales requeridos están presentes en el mapa de entrada para un proceso de cotización.

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
global static Map<Boolean, Object> validateInputMapForQuoting(
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

### `updateQuoteState(Id quoteId, String status)`

#### Descripción
Actualiza el estado de una cotización, verificando primero que el valor del estado sea válido.

#### Parámetros
- `quoteId` (Id): Id de la cotización a actualizar.
- `status` (String): Nuevo valor para el campo Status.

#### Retorno
- `SObject`: La cotización actualizada.

#### Proceso
1. Obtiene los metadatos del objeto Quote para acceder a la descripción del campo Status.
2. Verifica que el valor de estado proporcionado sea válido según los valores de la lista de selección (picklist).
3. Si el valor no es válido, lanza una excepción.
4. Consulta la cotización por su ID.
5. Actualiza el campo Status con el nuevo valor.
6. Guarda los cambios.
7. Devuelve la cotización actualizada.

#### Código
```apex
global static SObject updateQuoteState(Id quoteId, String status) {
  // Obtener la metadata del objeto Quote
  Schema.SObjectType quoteSchema = Schema.getGlobalDescribe().get('Quote');
  Schema.DescribeSObjectResult quoteDescribe = quoteSchema.getDescribe();
  Map<String, Schema.SObjectField> fieldMap = quoteDescribe.fields.getMap();
  Schema.DescribeFieldResult statusFieldDescribe = fieldMap.get('Status')
    .getDescribe();

  if (!CorePicklistUtils.containsVal(fieldMap.get('Status'), status)) {
    throw new ProductHandlerImplementationException(
      'invalid opportunity stage',
      'UNPROCESSABLE_ENTITY',
      422
    );
  }
  String query = 'SELECT Id, Status FROM Quote WHERE Id = :quoteId';
  List<SObject> results = Database.query(query);
  SObject quote = results[0];
  quote.put('Status', status);
  System.debug('quote utils: ' + quote);
  update quote;
  System.debug('quote utils update ok: ' + quote);
  return quote;
}
```

### `getQuoteById(Id quoteId)`

#### Descripción
Obtiene una cotización por su ID, incluyendo campos específicos.

#### Parámetros
- `quoteId` (Id): Id de la cotización a consultar.

#### Retorno
- `SObject`: La cotización encontrada.

#### Código
```apex
global static SObject getQuoteById(Id quoteId) {
  String query = 'SELECT Id, Name, AccountId, OpportunityId, Status, OwnerId FROM Quote WHERE Id = :quoteId LIMIT 1';
  List<SObject> quotes = Database.query(query);
  return quotes[0];
}
```

### `setInsuredPartyToQuoteAndOpp(List<Map<String, Object>> insuredPeople, SObject quote)`

#### Descripción
Método virtual que establece las partes aseguradas en una cotización y actualiza la cuenta asociada a la oportunidad relacionada.

#### Parámetros
- `insuredPeople` (List<Map<String, Object>>): Lista de mapas que representan las personas aseguradas.
- `quote` (SObject): La cotización a la que se asociarán las partes aseguradas.

#### Retorno
- `void`

#### Proceso
1. Obtiene el ID de la oportunidad asociada a la cotización.
2. Consulta la oportunidad para obtener sus detalles.
3. Recorre la lista de personas aseguradas.
4. Para cada persona, si contiene una clave 'Asegurado' con una cuenta asociada, actualiza la cuenta de la oportunidad.
5. Actualiza tanto la cotización como la oportunidad para guardar los cambios.

#### Código
```apex
global virtual void setInsuredPartyToQuoteAndOpp(
  List<Map<String, Object>> insuredPeople,
  SObject quote
) {
  // Obtener OpportunityId de la Quote
  Id opportunityId = (Id) quote.get('OpportunityId');

  Opportunity opp = [
    SELECT Id, AccountId
    FROM Opportunity
    WHERE Id = :opportunityId
  ];

  for (Map<String, Object> person : insuredPeople) {
    System.debug(person);
    if (person.containsKey('Asegurado')) {
      Account account = (Account) person.get('Asegurado');
      System.debug(account);
      if (account != null) {
        opp.AccountId = account.Id;
      }
    }
  }
  update quote;
  update opp;
}
```

## Excepciones

### `ProductHandlerImplementationException`
Esta excepción se lanza en el método `updateQuoteState` cuando se proporciona un valor de estado no válido para la cotización.

## Dependencias
- `CorePicklistUtils`: Clase utilitaria que contiene el método `containsVal` para verificar si un valor está presente en una lista de selección.
- `ProductHandlerImplementationException`: Excepción personalizada lanzada cuando hay errores en la implementación del manejador de productos.
- `Quote`: Objeto estándar de Salesforce para cotizaciones.
- `Opportunity`: Objeto estándar de Salesforce para oportunidades.
- `Account`: Objeto estándar de Salesforce para cuentas.

## Estructura de Datos Esperada

La clase espera que los datos de entrada para la validación de cotizaciones tengan la siguiente estructura:

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

### Validar datos de entrada para cotización

```apex
// Preparar datos de entrada para validación
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

// Validar los datos de entrada
Map<Boolean, Object> validationResult = QuoteUtils.validateInputMapForQuoting(input);
if (validationResult.containsKey(false)) {
  System.debug('Error de validación: ' + validationResult.get(false));
  // Manejar el error
} else {
  System.debug('Validación exitosa: ' + validationResult.get(true));
  // Continuar con el proceso
}
```

### Actualizar el estado de una cotización

```apex
// ID de la cotización a actualizar
Id quoteId = '0Q00x000000XXXXX';

try {
  // Actualizar el estado a "Approved"
  SObject updatedQuote = QuoteUtils.updateQuoteState(quoteId, 'Approved');
  System.debug('Cotización actualizada: ' + updatedQuote);
} catch (ProductHandlerImplementationException e) {
  System.debug('Error al actualizar el estado: ' + e.getMessage());
}
```

### Obtener una cotización por ID

```apex
// ID de la cotización a consultar
Id quoteId = '0Q00x000000XXXXX';

// Obtener la cotización
SObject quote = QuoteUtils.getQuoteById(quoteId);
System.debug('Cotización obtenida: ' + quote);
```

### Establecer partes aseguradas

```apex
// ID de la cotización
Id quoteId = '0Q00x000000XXXXX';

// Obtener la cotización
SObject quote = QuoteUtils.getQuoteById(quoteId);

// Crear cuenta para el asegurado
Account insuredAccount = new Account(
  Name = 'Asegurado Principal',
  Phone = '555-1234',
  BillingCity = 'Ciudad Ejemplo'
);
insert insuredAccount;

// Crear lista de partes aseguradas
List<Map<String, Object>> insuredParties = new List<Map<String, Object>>{
  new Map<String, Object>{
    'Asegurado' => insuredAccount,
    'TipoAsegurado' => 'Principal'
  }
};

// Establecer las partes aseguradas
QuoteUtils quoteUtils = new QuoteUtils();
quoteUtils.setInsuredPartyToQuoteAndOpp(insuredParties, quote);
```

## Extensión de la Clase

Siendo una clase virtual, `QuoteUtils` puede ser extendida para personalizar comportamientos específicos:

```apex
public class CustomQuoteUtils extends QuoteUtils {
    
    // Sobrescribir el método para añadir lógica personalizada
    global override void setInsuredPartyToQuoteAndOpp(
        List<Map<String, Object>> insuredPeople,
        SObject quote
    ) {
        // Obtener OpportunityId de la Quote
        Id opportunityId = (Id) quote.get('OpportunityId');

        Opportunity opp = [
            SELECT Id, AccountId, StageName, Amount
            FROM Opportunity
            WHERE Id = :opportunityId
        ];

        // Lógica personalizada para determinar la cuenta principal
        Account primaryAccount = null;
        Map<String, Account> accountsByType = new Map<String, Account>();
        
        // Clasificar cuentas por tipo
        for (Map<String, Object> person : insuredPeople) {
            if (person.containsKey('Asegurado')) {
                Account account = (Account) person.get('Asegurado');
                String tipoAsegurado = (String) person.get('TipoAsegurado');
                
                if (account != null) {
                    accountsByType.put(tipoAsegurado, account);
                    
                    if (tipoAsegurado == 'Principal' || primaryAccount == null) {
                        primaryAccount = account;
                    }
                }
            }
        }
        
        // Establecer la cuenta principal en la oportunidad
        if (primaryAccount != null) {
            opp.AccountId = primaryAccount.Id;
        }
        
        // Actualizar campos adicionales en la oportunidad basados en la cotización
        quote.put('Insured_Party_Count__c', insuredPeople.size());
        
        // Llamar al método personalizado para crear contactos de roles
        createContactRoles(opp.Id, accountsByType);
        
        // Guardar cambios
        update quote;
        update opp;
    }
    
    // Método adicional para crear roles de contacto
    private void createContactRoles(Id opportunityId, Map<String, Account> accountsByType) {
        List<OpportunityContactRole> roles = new List<OpportunityContactRole>();
        
        for (String type : accountsByType.keySet()) {
            Account acc = accountsByType.get(type);
            
            // Obtener el contacto primario de la cuenta
            Contact primaryContact = [
                SELECT Id 
                FROM Contact 
                WHERE AccountId = :acc.Id
                LIMIT 1
            ];
            
            if (primaryContact != null) {
                OpportunityContactRole role = new OpportunityContactRole(
                    OpportunityId = opportunityId,
                    ContactId = primaryContact.Id,
                    Role = mapTypeToRole(type)
                );
                roles.add(role);
            }
        }
        
        if (!roles.isEmpty()) {
            insert roles;
        }
    }
    
    // Método para mapear tipo de asegurado a rol de contacto
    private String mapTypeToRole(String type) {
        switch on type {
            when 'Principal' {
                return 'Decision Maker';
            }
            when 'Adicional' {
                return 'Influencer';
            }
            when else {
                return 'Other';
            }
        }
    }
}
```

## Consideraciones y Mejores Prácticas

1. **Manejo de Excepciones**: La clase maneja excepciones en algunos métodos pero no en todos. Considerar implementar un manejo consistente de excepciones en todos los métodos.

2. **Métodos Virtuales**: El método `setInsuredPartyToQuoteAndOpp` está marcado como virtual, permitiendo su personalización en clases derivadas. Al extender la clase, asegurarse de mantener la funcionalidad básica o llamar al método base según sea necesario.

3. **DML en Bucles**: El método `setInsuredPartyToQuoteAndOpp` realiza operaciones DML fuera de los bucles, lo cual es una buena práctica para evitar alcanzar límites de gobernador.

4. **Validación de Entradas**: La clase incluye métodos para validar la estructura de los datos de entrada, lo que es una buena práctica para garantizar la integridad de los datos.

5. **Uso de Consultas Dinámicas**: La clase utiliza consultas dinámicas para obtener objetos. Cuando sea posible, considerar el uso de consultas estáticas para aprovechar la verificación en tiempo de compilación.

## Notas Adicionales
1. La clase utiliza el operador de navegación segura (`?.`) para evitar excepciones de referencia nula al acceder a valores en mapas.
2. El mensaje de excepción en `updateQuoteState` menciona "invalid opportunity stage" aunque está validando el estado de una cotización, lo que podría indicar un error o reutilización de código.
3. La clase parece formar parte de un sistema más amplio para la gestión de cotizaciones y productos, interactuando con otras clases como `CorePicklistUtils` y `ProductHandlerImplementationException`.
4. El uso de la palabra clave `global` indica que esta clase está diseñada para ser accesible desde paquetes gestionados o diferentes namespaces.