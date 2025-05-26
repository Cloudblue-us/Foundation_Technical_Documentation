# ProductHandlerNotImplementedException

## Descripción General
`ProductHandlerNotImplementedException` es una clase de excepción personalizada que extiende la clase `Exception` estándar de Apex. Esta clase está diseñada para representar errores específicos que ocurren cuando se intenta utilizar un manejador de productos que no ha sido implementado correctamente o cuando se intenta acceder a funcionalidades no implementadas dentro de un manejador de productos.

## Autor
Soulberto Lorenzo \<soulberto@cloudblue.us\>

## Última Modificación
20 de agosto de 2024

## Estructura

```apex
public with sharing class ProductHandlerNotImplementedException extends Exception {
}
```

La clase tiene una implementación minimalista, sin agregar métodos o propiedades adicionales más allá de los heredados de la clase base `Exception`.

## Herencia
- **Clase Base**: `Exception`
- **Modificadores**: `public with sharing`

## Propósito
El propósito principal de esta clase es proporcionar un tipo de excepción específico para errores relacionados con implementaciones faltantes o incompletas en manejadores de productos, permitiendo:

1. **Especificidad del Error**: Distinguir claramente los errores de "no implementado" de otros tipos de excepciones en el sistema de manejo de productos.
2. **Manejo Específico**: Permitir que el código que captura excepciones pueda identificar y manejar específicamente casos donde una funcionalidad requerida no está implementada.
3. **Claridad en el Código**: Mejorar la legibilidad del código al indicar explícitamente cuando se está tratando con un caso de funcionalidad no implementada.
4. **Documentación Implícita**: Servir como recordatorio para los desarrolladores de que ciertas funcionalidades aún necesitan ser implementadas.

## Métodos Heredados
Al extender la clase `Exception` estándar, `ProductHandlerNotImplementedException` hereda los siguientes métodos y propiedades:

### Constructores
- **`ProductHandlerNotImplementedException()`**: Constructor por defecto.
- **`ProductHandlerNotImplementedException(String message)`**: Constructor que acepta un mensaje de error.
- **`ProductHandlerNotImplementedException(Exception cause)`**: Constructor que acepta otra excepción como causa.
- **`ProductHandlerNotImplementedException(String message, Exception cause)`**: Constructor que acepta un mensaje y otra excepción como causa.

### Métodos
- **`getMessage()`**: Devuelve el mensaje de error asociado con la excepción.
- **`getCause()`**: Devuelve la excepción que causó esta excepción, si existe.
- **`getTypeName()`**: Devuelve el nombre del tipo de excepción.
- **`setMessage(String message)`**: Establece el mensaje de error para la excepción.
- **`getLineNumber()`**: Devuelve el número de línea donde se generó la excepción.
- **`getStackTraceString()`**: Devuelve una representación de texto del seguimiento de la pila.

## Ejemplos de Uso

### Ejemplo 1: Método Abstracto No Implementado

```apex
public abstract class AbstractProductHandler {
    
    public abstract void processProduct(Product2 product);
    
    public void validateAndProcess(Product2 product) {
        // Validar el producto
        if (product == null) {
            throw new IllegalArgumentException('El producto no puede ser nulo');
        }
        
        // Ejecutar el procesamiento específico del producto
        try {
            processProduct(product);
        } catch (System.MethodNotImplementedException e) {
            // Capturar la excepción estándar y lanzar nuestra excepción personalizada
            throw new ProductHandlerNotImplementedException(
                'El método processProduct no está implementado para el manejador ' + 
                this.getClass().getName(), e
            );
        }
    }
}

public class IncompleteProductHandler extends AbstractProductHandler {
    // Nota: No se implementa el método abstracto processProduct
    // Esto generará un error en tiempo de ejecución
}

// Uso
try {
    AbstractProductHandler handler = new IncompleteProductHandler();
    Product2 product = [SELECT Id, Name FROM Product2 LIMIT 1];
    handler.validateAndProcess(product);
} catch (ProductHandlerNotImplementedException e) {
    System.debug('Error: ' + e.getMessage());
    // Manejar el caso específico de método no implementado
}
```

### Ejemplo 2: Funcionalidad No Implementada

```apex
public class GenericProductHandler {
    
    public void processProduct(String productType, Map<String, Object> productData) {
        switch on productType.toUpperCase() {
            when 'HARDWARE' {
                processHardwareProduct(productData);
            }
            when 'SOFTWARE' {
                processSoftwareProduct(productData);
            }
            when 'SERVICE' {
                // Esta funcionalidad aún no está implementada
                throw new ProductHandlerNotImplementedException(
                    'El procesamiento de productos de tipo SERVICE no está implementado todavía'
                );
            }
            when else {
                throw new IllegalArgumentException('Tipo de producto no reconocido: ' + productType);
            }
        }
    }
    
    private void processHardwareProduct(Map<String, Object> productData) {
        // Implementación para productos de hardware
    }
    
    private void processSoftwareProduct(Map<String, Object> productData) {
        // Implementación para productos de software
    }
}

// Uso
try {
    GenericProductHandler handler = new GenericProductHandler();
    Map<String, Object> productData = new Map<String, Object>{ 
        'name' => 'Servicio de Mantenimiento',
        'price' => 1200.00
    };
    handler.processProduct('SERVICE', productData);
} catch (ProductHandlerNotImplementedException e) {
    System.debug('Error: ' + e.getMessage());
    // Informar al usuario que la funcionalidad no está disponible
    ApexPages.addMessage(new ApexPages.Message(
        ApexPages.Severity.WARNING,
        'Esta funcionalidad estará disponible próximamente. Error: ' + e.getMessage()
    ));
}
```

### Ejemplo 3: Implementación Parcial con Método Placeholder

```apex
public interface ProductHandler {
    void initialize();
    void process();
    void finalize();
}

public abstract class BaseProductHandler implements ProductHandler {
    
    public void initialize() {
        // Implementación común de inicialización
    }
    
    public abstract void process();
    
    public void finalize() {
        // Implementación común de finalización
    }
}

public class PartialProductHandler extends BaseProductHandler {
    
    // Implementación del método abstracto requerido, pero no completamente funcional
    public override void process() {
        // Esto es solo un placeholder, no una implementación completa
        throw new ProductHandlerNotImplementedException(
            'La implementación de process() está en desarrollo. Por favor, consulte la documentación para alternativas.'
        );
    }
}

// Uso
try {
    ProductHandler handler = new PartialProductHandler();
    handler.initialize();
    handler.process(); // Esto lanzará ProductHandlerNotImplementedException
    handler.finalize();
} catch (ProductHandlerNotImplementedException e) {
    System.debug('Error: ' + e.getMessage());
    // Redirigir a una implementación alternativa o documentación
}
```

## Relación con Otras Clases en el Sistema

Esta excepción está diseñada para trabajar en conjunto con:

1. **ProductHandler**: Interfaz o clase base que define el contrato para manejadores de productos.
2. **Implementaciones Concretas de ProductHandler**: Clases que implementan `ProductHandler` para tipos específicos de productos.
3. **QuoteProductHandler**: Manejador especializado para productos en cotizaciones.
4. **ProductHandlerImplementationException**: Otra excepción personalizada que puede ser utilizada para errores diferentes a "no implementado".

## Mejores Prácticas para su Uso

1. **Mensajes Descriptivos**: Proporcionar mensajes de error claros y descriptivos que indiquen qué funcionalidad específica no está implementada.
2. **Documentación Adicional**: Cuando sea posible, incluir en el mensaje información sobre cuándo estará disponible la funcionalidad o alternativas.
3. **Uso Selectivo**: Utilizar esta excepción específicamente para casos de "no implementado", no para errores generales de implementación.
4. **Captura Específica**: Al capturar excepciones, manejar esta excepción específicamente para proporcionar información adecuada al usuario.

## Implementación Extendida Propuesta

Aunque la implementación actual es minimalista, podría extenderse para incluir información adicional:

```apex
public with sharing class ProductHandlerNotImplementedException extends Exception {
    
    private String featureName;
    private String expectedAvailability;
    private String alternativePath;
    
    /**
     * Constructor por defecto
     */
    public ProductHandlerNotImplementedException() {
        super();
    }
    
    /**
     * Constructor con mensaje de error
     * @param message Mensaje descriptivo del error
     */
    public ProductHandlerNotImplementedException(String message) {
        super(message);
    }
    
    /**
     * Constructor con información detallada
     * @param message Mensaje descriptivo del error
     * @param featureName Nombre de la característica no implementada
     * @param expectedAvailability Fecha o versión esperada para la disponibilidad
     * @param alternativePath Ruta o método alternativo a utilizar mientras tanto
     */
    public ProductHandlerNotImplementedException(
        String message, 
        String featureName, 
        String expectedAvailability, 
        String alternativePath
    ) {
        super(message);
        this.featureName = featureName;
        this.expectedAvailability = expectedAvailability;
        this.alternativePath = alternativePath;
    }
    
    /**
     * Constructor con excepción causa
     * @param message Mensaje descriptivo del error
     * @param cause Excepción que causó esta excepción
     */
    public ProductHandlerNotImplementedException(String message, Exception cause) {
        super(message, cause);
    }
    
    /**
     * Obtiene el nombre de la característica no implementada
     * @return Nombre de la característica
     */
    public String getFeatureName() {
        return this.featureName;
    }
    
    /**
     * Obtiene la fecha o versión esperada para la disponibilidad
     * @return Fecha o versión esperada
     */
    public String getExpectedAvailability() {
        return this.expectedAvailability;
    }
    
    /**
     * Obtiene la ruta o método alternativo sugerido
     * @return Ruta o método alternativo
     */
    public String getAlternativePath() {
        return this.alternativePath;
    }
    
    /**
     * Genera un mensaje completo con toda la información disponible
     * @return Mensaje completo
     */
    public String getFullMessage() {
        String fullMessage = getMessage();
        
        if (String.isNotBlank(featureName)) {
            fullMessage += '\nCaracterística: ' + featureName;
        }
        
        if (String.isNotBlank(expectedAvailability)) {
            fullMessage += '\nDisponibilidad esperada: ' + expectedAvailability;
        }
        
        if (String.isNotBlank(alternativePath)) {
            fullMessage += '\nAlternativa: ' + alternativePath;
        }
        
        return fullMessage;
    }
}
```

## Ejemplo de Uso de la Implementación Extendida

```apex
try {
    throw new ProductHandlerNotImplementedException(
        'La generación automática de cotizaciones para productos de servicio no está implementada',
        'AutoQuoteService',
        'Q3 2023',
        'Utilizar el proceso manual a través de QuoteWizardController.generateManualQuote()'
    );
} catch (ProductHandlerNotImplementedException e) {
    // Registrar el error completo para depuración
    System.debug(e.getFullMessage());
    
    // Mostrar un mensaje amigable al usuario
    String userMessage = 'La funcionalidad solicitada aún no está disponible';
    
    if (String.isNotBlank(e.getExpectedAvailability())) {
        userMessage += ' (esperada para ' + e.getExpectedAvailability() + ')';
    }
    
    if (String.isNotBlank(e.getAlternativePath())) {
        userMessage += '. Mientras tanto, puede ' + e.getAlternativePath();
    }
    
    ApexPages.addMessage(new ApexPages.Message(
        ApexPages.Severity.INFO,
        userMessage
    ));
}
```

## Consideraciones de Diseño

### Ventajas de la Implementación Actual
1. **Simplicidad**: La implementación minimalista es fácil de entender y utilizar.
2. **Enfoque Específico**: Se centra exclusivamente en representar un error de "no implementado".
3. **Integración con el Manejo de Excepciones Existente**: Se integra perfectamente con los mecanismos estándar de manejo de excepciones de Apex.

### Posibles Mejoras
1. **Información Adicional**: Podría extenderse para incluir información más detallada como se muestra en la implementación extendida propuesta.
2. **Métodos de Utilidad**: Podrían añadirse métodos de utilidad para facilitar la creación y el manejo de esta excepción.
3. **Integración con Sistemas de Registro**: Podría añadirse funcionalidad para registrar automáticamente estas excepciones en un sistema de seguimiento.

## Notas Adicionales
1. La clase utiliza el modificador `with sharing`, lo que significa que respetará las reglas de compartición de Salesforce durante su ejecución. Esto es apropiado para excepciones que se utilizan en contextos donde la seguridad es importante.
2. La excepción es parte de un ecosistema más amplio de manejo de productos, trabajando en conjunto con otras clases relacionadas.
3. El nombre de la clase indica claramente su propósito, lo que es una buena práctica para mejorar la legibilidad y mantenibilidad del código.
4. Es una práctica común en desarrollo de software crear excepciones específicas para diferentes tipos de errores, como se hace aquí para el caso de funcionalidades no implementadas.