# LeadProductHandler

## Descripción General
`LeadProductHandler` es una clase virtual global que extiende `ProductHandler`. Esta clase está diseñada para manejar el procesamiento de leads como productos, siguiendo un patrón de diseño que facilita la creación y configuración flexible de objetos Lead en Salesforce.

## Autor
Soulberto Lorenzo \<soulberto@cloudblue.us\>

## Última Modificación
21 de febrero de 2025 por Jean Carlos Melendez

## Constantes

### `PROCESS_NAME`
- **Valor**: `"KNOWING"`
- **Descripción**: Constante que define el nombre del proceso asociado a este handler.

## Constructor

### `LeadProductHandler(String productName)`

#### Descripción
Constructor que inicializa un nuevo handler con un nombre de producto específico, llamando al constructor de la clase base.

#### Parámetros
- `productName` (String): Nombre del producto asociado a este handler.

#### Código
```apex
global LeadProductHandler(String productName) {
  super(productName);
}
```

## Métodos

### `setObservers()`

#### Descripción
Método sobrescrito que configura los observadores para el handler, específicamente añadiendo un `LeadObserver`.

#### Retorno
- `void`

#### Código
```apex
public override void setObservers() {
  this.addObserver(new LeadObserver());
}
```

### `process(Map<String, Object> options, Map<String, Object> input, Map<String, Object> output)`

#### Descripción
Método principal sobrescrito que procesa la creación de un Lead utilizando los datos de entrada proporcionados, aplicando opciones adicionales y almacenando el resultado en el mapa de salida.

#### Parámetros
- `options` (Map<String, Object>): Opciones adicionales para el proceso.
- `input` (Map<String, Object>): Datos de entrada para la creación del Lead.
- `output` (Map<String, Object>): Mapa de salida donde se almacenará el resultado.

#### Retorno
- `Map<String, Object>`: Mapa con los datos del Lead creado.

#### Proceso
1. Verifica si se debe omitir el comportamiento predeterminado según las opciones.
2. Valida que ningún valor en el mapa de entrada sea nulo.
3. Establece el nombre del proceso utilizando la constante `PROCESS_NAME`.
4. Obtiene los configuradores de Lead del mapa de entrada.
5. Utiliza `LeadBuilderFoundation` con un patrón builder para crear un objeto Lead.
6. Guarda el Lead creado en el mapa de salida.
7. Establece el proceso de respuesta.
8. Maneja excepciones de DML y generales, lanzando `ProductHandlerImplementationException` con el código 422.

#### Código
```apex
public override Map<String, Object> process(
  Map<String, Object> options,
  Map<String, Object> input,
  Map<String, Object> output
) {
  if (options?.containsKey('skipDefaultBehavior')) {
    return input;
  }

  for (String key : input.keySet()) {
    if (input?.get(key) == null) {
      Core.debug(
        key +
        ' must not be empty, throwing product handler implementation custom exception'
      );
      throw new ProductHandlerImplementationException(
        key + ' must not be empty',
        'UNPROCESSABLE_ENTITY',
        422
      );
    }
  }
  setProcessName(PROCESS_NAME);

  List<LeadConfigurator> configurators = new List<LeadConfigurator>();
  configurators = (List<LeadConfigurator>) input?.get('configurators');
  try {
    Lead newLead = LeadBuilderFoundation.createLead(
      new LeadBuilderFoundation()
        .setStatus((String) input?.get('status') ?? '0_Abierto')
        .setId((Id) input?.get('leadId'))
        .setLastName((String) input?.get('lastName'))
        .setFirstName((String) input?.get('firstName'))
        .setEmail((String) input?.get('email'))
        .setMobile((String) input?.get('mobile_phone'))
        .setPhone((String) input?.get('phone'))
        .addConfigurators(configurators ?? new List<LeadConfigurator>())
        .build()
    );

    Core.debug('Finishing lead trip process...' + newLead);
    output.put('resultProcess', newLead);
    this.setResponseProcess((newLead));
    return output;

    // TODO: Implement exception handling Pipe
  } catch (DmlException dmlEx) {
    System.debug(
      'DML Exception while processing lead: ' + dmlEx.getDmlMessage(0)
    );
    throw new ProductHandlerImplementationException(
      dmlEx.getMessage(),
      'UNPROCESSABLE_ENTITY',
      422
    );
  } catch (Exception e) {
    //System.debug(e.getCause().toString() + ' ' + e.getMessage());
    throw new ProductHandlerImplementationException(
      e.getMessage(),
      'UNPROCESSABLE_ENTITY',
      422
    );
  }
}
```

### `isValidForLeadRequirements(Map<String, Object> data)`

#### Descripción
Método virtual que valida los requerimientos de negocio para la creación de un Lead. Este método está diseñado para ser sobrescrito por clases derivadas concretas que implementarán lógicas específicas de validación.

#### Parámetros
- `data` (Map<String, Object>): Datos a validar.

#### Retorno
- `Boolean`: `true` si los datos cumplen con los requisitos, `false` en caso contrario. La implementación base siempre retorna `true`.

#### Código
```apex
global virtual Boolean isValidForLeadRequirements(Map<String, Object> data) {
  /**
   * Se sobreescribira el codigo que evaluará la logica
   * y los requerimientos de negocios para creación de Lead...
   **/
  return true;
}
```

## Excepciones

### `ProductHandlerImplementationException`
Esta excepción se lanza en las siguientes situaciones:
1. Cuando un valor requerido en el mapa de entrada es nulo.
2. Cuando ocurre una excepción de DML durante la creación del Lead.
3. Cuando ocurre cualquier otra excepción durante el proceso.

En todos los casos, la excepción se lanza con un código HTTP 422 (UNPROCESSABLE_ENTITY).

## Dependencias
- `ProductHandler`: Clase base que `LeadProductHandler` extiende.
- `LeadObserver`: Clase observadora añadida en el método `setObservers()`.
- `Core`: Clase utilitaria que proporciona el método `debug`.
- `ProductHandlerImplementationException`: Excepción personalizada lanzada cuando hay errores en el proceso.
- `LeadBuilderFoundation`: Clase que implementa el patrón Builder para crear objetos Lead.
- `LeadConfigurator`: Interfaz o clase que define configuradores para personalizar objetos Lead.
- `Lead`: Objeto estándar de Salesforce utilizado en el proceso.

## Patrones de Diseño Implementados

### Observer
A través del método `setObservers()`, la clase implementa el patrón Observer, permitiendo que otros objetos (específicamente, `LeadObserver`) sean notificados de eventos durante el procesamiento.

### Builder
Aunque no implementado directamente en esta clase, `LeadProductHandler` utiliza el patrón Builder a través de `LeadBuilderFoundation` para crear objetos Lead de manera flexible.

### Template Method
Como clase virtual con métodos que pueden ser sobrescritos, `LeadProductHandler` implementa el patrón Template Method, definiendo el esqueleto de un algoritmo pero permitiendo que las subclases sobrescriban pasos específicos.

## Ejemplo de Uso
```apex
// Crear una instancia del handler
LeadProductHandler handler = new LeadProductHandler('LeadGeneration');

// Preparar los datos de entrada
Map<String, Object> input = new Map<String, Object>{
  'lastName' => 'Pérez',
  'firstName' => 'Juan',
  'email' => 'juan.perez@example.com',
  'phone' => '555-1234',
  'mobile_phone' => '555-5678',
  'status' => '0_Abierto'
};

// Configuradores personalizados (opcional)
List<LeadConfigurator> configurators = new List<LeadConfigurator>{
  new CompanyNameConfigurator('ACME Inc.'),
  new LeadSourceConfigurator('Web')
};
input.put('configurators', configurators);

// Mapas para opciones y salida
Map<String, Object> options = new Map<String, Object>();
Map<String, Object> output = new Map<String, Object>();

try {
  // Procesar la creación del Lead
  Map<String, Object> result = handler.process(options, input, output);
  
  // Obtener el Lead creado
  Lead newLead = (Lead)result.get('resultProcess');
  System.debug('Lead creado: ' + newLead.Id);
} catch (ProductHandlerImplementationException e) {
  System.debug('Error: ' + e.getMessage());
}
```

## Extensión de la Clase
```apex
public class PremiumLeadHandler extends LeadProductHandler {
    
    public PremiumLeadHandler() {
        super('PremiumLead');
    }
    
    // Sobrescribir para implementar validaciones específicas
    global override Boolean isValidForLeadRequirements(Map<String, Object> data) {
        // Validar que sea un lead premium (ejemplo: debe tener email y teléfono)
        String email = (String)data.get('email');
        String phone = (String)data.get('phone');
        
        return !String.isBlank(email) && !String.isBlank(phone);
    }
    
    // Opcionalmente sobrescribir process para añadir lógica específica
    public override Map<String, Object> process(
        Map<String, Object> options,
        Map<String, Object> input,
        Map<String, Object> output
    ) {
        // Validar requisitos específicos antes de procesar
        if (!isValidForLeadRequirements(input)) {
            throw new ProductHandlerImplementationException(
                'Este lead no cumple con los requisitos para ser premium',
                'UNPROCESSABLE_ENTITY',
                422
            );
        }
        
        // Añadir configurador específico para leads premium
        List<LeadConfigurator> configurators = (List<LeadConfigurator>)input.get('configurators');
        if (configurators == null) {
            configurators = new List<LeadConfigurator>();
            input.put('configurators', configurators);
        }
        
        configurators.add(new PremiumLeadConfigurator());
        
        // Llamar al proceso base
        return super.process(options, input, output);
    }
}
```

## Notas Adicionales
1. La clase usa el operador de navegación segura (`?.`) y el operador de coalescencia nula (`??`) para evitar excepciones de referencia nula, lo que indica que está destinada a ejecutarse en versiones recientes de Apex.
2. Hay un comentario TODO para implementar un manejo de excepciones con "Pipe", lo que sugiere que podría haber planes para mejorar el manejo de errores en el futuro.
3. El método `process()` valida que todos los valores de entrada no sean nulos, lo que podría ser restrictivo en algunos casos. Considere implementar una lista de campos requeridos específicos en lugar de exigir que todos los campos tengan un valor.
4. La clase implementa un mecanismo para omitir su comportamiento predeterminado si la opción `skipDefaultBehavior` está presente, lo que proporciona flexibilidad para casos de uso especiales.