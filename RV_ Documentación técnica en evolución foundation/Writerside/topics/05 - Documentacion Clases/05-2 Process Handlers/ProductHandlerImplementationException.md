# ProductHandlerImplementationException

## Descripción General
`ProductHandlerImplementationException` es una clase global de excepción personalizada que extiende la clase `Exception` estándar de Apex. Esta clase está diseñada específicamente para representar errores que ocurren durante la implementación y ejecución de manejadores de productos, proporcionando información adicional como códigos de error y códigos de estado HTTP.

## Autor
Jean Carlos Melendez

## Última Modificación
4 de febrero de 2025

## Estructura

```apex
global class ProductHandlerImplementationException extends Exception {
   String msg;
   String errorCode;
   Integer errorStatusCode;

   global ProductHandlerImplementationException(String msg, String errorCode, Integer errorStatusCode) {
      this.msg = msg;
      this.errorCode = errorCode;
      this.errorStatusCode = errorStatusCode;
   }

   global override String getMessage() {
      return this.msg;
   }

   global String getErrorCode() {
      return this.errorCode;
   }

   global Integer getStatusCode() {
      return this.errorStatusCode;
   }
}
```

## Propiedades

### `msg`
- **Tipo**: `String`
- **Descripción**: Mensaje que describe el error.

### `errorCode`
- **Tipo**: `String`
- **Descripción**: Código de error específico que categoriza el tipo de error.

### `errorStatusCode`
- **Tipo**: `Integer`
- **Descripción**: Código de estado HTTP correspondiente al error, útil para mapear excepciones a respuestas HTTP.

## Constructor

### `ProductHandlerImplementationException(String msg, String errorCode, Integer errorStatusCode)`

#### Descripción
Constructor que inicializa una nueva instancia de la excepción con un mensaje, código de error y código de estado.

#### Parámetros
- `msg` (String): Mensaje descriptivo del error.
- `errorCode` (String): Código de error específico.
- `errorStatusCode` (Integer): Código de estado HTTP correspondiente.

#### Código
```apex
global ProductHandlerImplementationException(String msg, String errorCode, Integer errorStatusCode) {
   this.msg = msg;
   this.errorCode = errorCode;
   this.errorStatusCode = errorStatusCode;
}
```

## Métodos

### `getMessage()`

#### Descripción
Método sobrescrito que devuelve el mensaje de error asociado con la excepción.

#### Parámetros
Ninguno.

#### Retorno
- `String`: El mensaje de error.

#### Código
```apex
global override String getMessage() {
   return this.msg;
}
```

### `getErrorCode()`

#### Descripción
Método que devuelve el código de error específico asociado con la excepción.

#### Parámetros
Ninguno.

#### Retorno
- `String`: El código de error.

#### Código
```apex
global String getErrorCode() {
   return this.errorCode;
}
```

### `getStatusCode()`

#### Descripción
Método que devuelve el código de estado HTTP asociado con la excepción.

#### Parámetros
Ninguno.

#### Retorno
- `Integer`: El código de estado HTTP.

#### Código
```apex
global Integer getStatusCode() {
   return this.errorStatusCode;
}
```

## Códigos de Error Comunes

A continuación se presenta una tabla de códigos de error comunes que se utilizan con esta excepción:

| Código de Error | Descripción | Código de Estado Típico |
|----------------|-------------|------------------------|
| `BAD_REQUEST` | La solicitud es inválida o carece de parámetros requeridos. | 400 |
| `UNAUTHORIZED` | La solicitud requiere autenticación o la autenticación proporcionada es insuficiente. | 401 |
| `FORBIDDEN` | La autenticación es válida pero no tiene permisos para el recurso. | 403 |
| `NOT_FOUND` | El recurso o producto solicitado no se encuentra. | 404 |
| `UNPROCESSABLE_ENTITY` | La solicitud es válida pero no se puede procesar debido a errores semánticos. | 422 |
| `INTERNAL_ERROR` | Ocurrió un error interno inesperado. | 500 |
| `SERVICE_UNAVAILABLE` | El servicio externo requerido no está disponible. | 503 |

## Ejemplos de Uso

### Ejemplo 1: Validación de Campos Requeridos

```apex
public class ProductValidator {
    public static void validateRequiredFields(Map<String, Object> productData) {
        if (productData == null) {
            throw new ProductHandlerImplementationException(
                'Datos del producto no proporcionados',
                'BAD_REQUEST',
                400
            );
        }
        
        if (!productData.containsKey('productCode') || String.isBlank((String)productData.get('productCode'))) {
            throw new ProductHandlerImplementationException(
                'El código del producto es requerido',
                'UNPROCESSABLE_ENTITY',
                422
            );
        }
        
        if (!productData.containsKey('price') || productData.get('price') == null) {
            throw new ProductHandlerImplementationException(
                'El precio del producto es requerido',
                'UNPROCESSABLE_ENTITY',
                422
            );
        }
    }
}
```

### Ejemplo 2: Verificación de Existencia de Producto

```apex
public class ProductFinder {
    public static Product2 findProductById(Id productId) {
        List<Product2> products = [SELECT Id, Name, ProductCode, IsActive FROM Product2 WHERE Id = :productId LIMIT 1];
        
        if (products.isEmpty()) {
            throw new ProductHandlerImplementationException(
                'Producto no encontrado con ID: ' + productId,
                'NOT_FOUND',
                404
            );
        }
        
        Product2 product = products[0];
        
        if (!product.IsActive) {
            throw new ProductHandlerImplementationException(
                'El producto está inactivo: ' + product.Name,
                'UNPROCESSABLE_ENTITY',
                422
            );
        }
        
        return product;
    }
}
```

### Ejemplo 3: Manejo de Errores en un Controlador REST

```apex
@RestResource(urlMapping='/products/*')
global class ProductRESTService {
    
    @HttpGet
    global static void getProduct() {
        RestRequest req = RestContext.request;
        RestResponse res = RestContext.response;
        
        try {
            String productId = req.requestURI.substring(req.requestURI.lastIndexOf('/') + 1);
            
            if (String.isBlank(productId)) {
                throw new ProductHandlerImplementationException(
                    'ID de producto no proporcionado',
                    'BAD_REQUEST',
                    400
                );
            }
            
            Product2 product = ProductFinder.findProductById(productId);
            
            Map<String, Object> result = new Map<String, Object>{
                'success' => true,
                'product' => new Map<String, Object>{
                    'id' => product.Id,
                    'name' => product.Name,
                    'code' => product.ProductCode
                }
            };
            
            res.responseBody = Blob.valueOf(JSON.serialize(result));
            res.statusCode = 200;
        } catch (ProductHandlerImplementationException e) {
            handleException(e, res);
        } catch (Exception e) {
            // Convertir excepciones estándar a nuestra excepción personalizada
            ProductHandlerImplementationException customEx = new ProductHandlerImplementationException(
                'Error inesperado: ' + e.getMessage(),
                'INTERNAL_ERROR',
                500
            );
            handleException(customEx, res);
        }
    }
    
    private static void handleException(ProductHandlerImplementationException e, RestResponse res) {
        Map<String, Object> errorResponse = new Map<String, Object>{
            'success' => false,
            'errorCode' => e.getErrorCode(),
            'message' => e.getMessage()
        };
        
        res.responseBody = Blob.valueOf(JSON.serialize(errorResponse));
        res.statusCode = e.getStatusCode();
    }
}
```

### Ejemplo 4: Uso en Manejadores de Productos

```apex
public class AutoInsuranceProductHandler extends ProductHandler {
    
    public override Map<String, Object> process(
        Map<String, Object> options,
        Map<String, Object> input,
        Map<String, Object> output
    ) {
        try {
            // Validar datos de entrada
            validateInput(input);
            
            // Procesar el producto
            // ...
            
            return output;
        } catch (Exception e) {
            if (e instanceof ProductHandlerImplementationException) {
                throw e;
            } else {
                throw new ProductHandlerImplementationException(
                    'Error procesando el producto Auto Insurance: ' + e.getMessage(),
                    'INTERNAL_ERROR',
                    500
                );
            }
        }
    }
    
    private void validateInput(Map<String, Object> input) {
        if (!input.containsKey('vehicleData')) {
            throw new ProductHandlerImplementationException(
                'Datos del vehículo no proporcionados',
                'UNPROCESSABLE_ENTITY',
                422
            );
        }
        
        Map<String, Object> vehicleData = (Map<String, Object>)input.get('vehicleData');
        
        if (!vehicleData.containsKey('make') || String.isBlank((String)vehicleData.get('make'))) {
            throw new ProductHandlerImplementationException(
                'La marca del vehículo es requerida',
                'UNPROCESSABLE_ENTITY',
                422
            );
        }
        
        if (!vehicleData.containsKey('model') || String.isBlank((String)vehicleData.get('model'))) {
            throw new ProductHandlerImplementationException(
                'El modelo del vehículo es requerido',
                'UNPROCESSABLE_ENTITY',
                422
            );
        }
        
        if (!vehicleData.containsKey('year') || vehicleData.get('year') == null) {
            throw new ProductHandlerImplementationException(
                'El año del vehículo es requerido',
                'UNPROCESSABLE_ENTITY',
                422
            );
        }
    }
}
```

## Captura y Manejo de la Excepción

```apex
try {
    // Código que podría lanzar la excepción
    Map<String, Object> productData = new Map<String, Object>();
    ProductValidator.validateRequiredFields(productData);
} catch (ProductHandlerImplementationException e) {
    System.debug('Error: ' + e.getMessage());
    System.debug('Código de error: ' + e.getErrorCode());
    System.debug('Código de estado: ' + e.getStatusCode());
    
    // Manejar el error según su código
    switch on e.getErrorCode() {
        when 'BAD_REQUEST' {
            // Lógica para manejar errores de solicitud incorrecta
        }
        when 'UNPROCESSABLE_ENTITY' {
            // Lógica para manejar errores de validación
        }
        when 'NOT_FOUND' {
            // Lógica para manejar recursos no encontrados
        }
        when else {
            // Lógica para manejar otros errores
        }
    }
}
```

## Uso en Pruebas Unitarias

```apex
@IsTest
private class ProductHandlerImplementationExceptionTest {
    
    @IsTest
    static void testConstructorAndGetters() {
        // Configurar
        String message = 'Error de prueba';
        String errorCode = 'TEST_ERROR';
        Integer statusCode = 418; // I'm a teapot
        
        // Ejecutar
        ProductHandlerImplementationException ex = new ProductHandlerImplementationException(
            message, errorCode, statusCode
        );
        
        // Verificar
        System.assertEquals(message, ex.getMessage(), 'El mensaje debe coincidir');
        System.assertEquals(errorCode, ex.getErrorCode(), 'El código de error debe coincidir');
        System.assertEquals(statusCode, ex.getStatusCode(), 'El código de estado debe coincidir');
    }
    
    @IsTest
    static void testExceptionInProductValidator() {
        // Configurar
        Map<String, Object> emptyData = new Map<String, Object>();
        
        // Ejecutar y verificar
        try {
            ProductValidator.validateRequiredFields(emptyData);
            System.assert(false, 'Debería haber lanzado una excepción');
        } catch (ProductHandlerImplementationException e) {
            System.assertEquals('UNPROCESSABLE_ENTITY', e.getErrorCode(), 'Debería ser un error de entidad no procesable');
            System.assertEquals(422, e.getStatusCode(), 'El código de estado debería ser 422');
        }
    }
}
```

## Mejores Prácticas

1. **Códigos de Error Consistentes**: Utilizar un conjunto consistente de códigos de error en toda la aplicación.
2. **Mensajes Descriptivos**: Proporcionar mensajes de error claros y descriptivos.
3. **Códigos de Estado HTTP Apropiados**: Usar códigos de estado HTTP estándar que se alineen con el tipo de error.
4. **Manejo Centralizado**: Implementar un manejo centralizado de excepciones en puntos de entrada como servicios REST.
5. **Registro de Errores**: Registrar los errores para análisis y depuración futuros.

## Integración con Sistemas más Amplios

La clase `ProductHandlerImplementationException` está diseñada para integrarse con el sistema más amplio de manejo de productos:

1. **ProductHandler y Derivados**: Clases como `QuoteProductHandler`, `IssueProductHandler`, etc., utilizan esta excepción para gestionar errores.
2. **Servicios REST**: Los servicios REST pueden capturar y transformar estas excepciones en respuestas HTTP estructuradas.
3. **Procedimientos de Integración**: Los procedimientos de integración pueden capturar estas excepciones y manejarlas adecuadamente.

## Consideraciones de Rendimiento

1. **Creación de Instancias**: La creación de objetos de excepción tiene un costo en términos de rendimiento y debe utilizarse para condiciones de error genuinas, no para control de flujo normal.
2. **Propagación de Excepciones**: Propagar excepciones a través de múltiples niveles de la pila de llamadas puede tener un impacto en el rendimiento. Considerar capturar y manejar excepciones lo más cerca posible de su origen cuando sea apropiado.

## Notas Adicionales
1. La clase está marcada como `global`, lo que indica que está diseñada para ser accesible desde diferentes paquetes o namespaces.
2. La implementación actual no guarda la excepción causa, lo que podría ser una mejora para considerar en futuras versiones.
3. Los códigos de estado HTTP utilizados (como 422 para `UNPROCESSABLE_ENTITY`) se alinean con los estándares HTTP, lo que facilita la integración con servicios web.
4. Esta excepción proporciona una estructura estandarizada para manejo de errores, lo que mejora la consistencia en toda la aplicación.