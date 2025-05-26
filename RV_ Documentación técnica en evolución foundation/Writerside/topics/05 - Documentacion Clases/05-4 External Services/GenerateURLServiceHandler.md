# GenerateURLServiceHandler

## Descripción General
`GenerateURLServiceHandler` es una clase virtual global que extiende `PlatformEventIntegrationObserver` y está diseñada para manejar la integración con servicios externos para la generación de URLs. Esta clase implementa un patrón de procesamiento en tres fases (pre-proceso, proceso principal y post-proceso) y proporciona capacidades de publicación de eventos de plataforma para monitoreo y auditoría.

## Autor
Felipe Correa \<felipe.correa@nespon.com\>

## Proyecto
Foundation Salesforce - Suratech - UH-17594

## Última Modificación
20 de febrero de 2025 por "ChangeMeIn@UserSettingsUnder.SFDoc"

## Estructura de la Clase

```apex
global virtual class GenerateURLServiceHandler extends PlatformEventIntegrationObserver {
  // Constantes de configuración
  private static final String SERVICE_METHOD = 'POST';
  private static final String SURATECH_MDT_CONFIG_NAME = 'url_endpoint';
  
  // Métodos de procesamiento
  global virtual Map<String, Object> preProcess(Map<String, Object> input, Map<String, Object> options);
  global Map<String, Object> invokeProcess(Map<String, Object> input, Map<String, Object> output, Map<String, Object> options);
  global virtual Map<String, Object> postProcess(Map<String, Object> response, Map<String, Object> options);
  global void publishPlatformEventIntegration(String moment, String data);
}
```

## Herencia
- **Clase Base**: `PlatformEventIntegrationObserver`
- **Modificadores**: `global virtual`

## Constantes

| Nombre | Valor | Descripción |
|--------|-------|-------------|
| `SERVICE_METHOD` | `'POST'` | Método HTTP utilizado para las llamadas al servicio externo |
| `SURATECH_MDT_CONFIG_NAME` | `'url_endpoint'` | Nombre del registro de metadatos personalizados que contiene la configuración del servicio |

## Métodos

### `preProcess(Map<String, Object> input, Map<String, Object> options)`

#### Descripción
Método virtual que se ejecuta antes del procesamiento principal. Realiza la preparación inicial de los datos y opcionalmente publica eventos de plataforma para el monitoreo del proceso.

#### Parámetros
- `input` (Map<String, Object>): Datos de entrada para el procesamiento.
- `options` (Map<String, Object>): Opciones de configuración para el procesamiento.

#### Retorno
- `Map<String, Object>`: Los datos de entrada, potencialmente modificados durante el pre-procesamiento.

#### Proceso
1. Registra el inicio del pre-procesamiento.
2. Verifica si los eventos de plataforma están habilitados en las opciones.
3. Si están habilitados, publica un evento de plataforma con los datos de entrada.
4. Devuelve los datos de entrada sin modificaciones.

#### Código
```apex
global virtual Map<String, Object> preProcess(
  Map<String, Object> input,
  Map<String, Object> options
) {
  Core.debug('Starting Generate URL Integration Service Pre Process');
  if (
    options.containsKey('PlatformEventEnabled') &&
    ((Boolean) options.get('PlatformEventEnabled')) == true
  )
    publishPlatformEventIntegration('pre', JSON.serialize(input));
  return input;
}
```

### `invokeProcess(Map<String, Object> input, Map<String, Object> output, Map<String, Object> options)`

#### Descripción
Método principal que ejecuta la lógica de integración con el servicio externo para la generación de URLs. Maneja la configuración, validación, comunicación con el servicio externo y procesamiento de respuestas.

#### Parámetros
- `input` (Map<String, Object>): Datos de entrada para el procesamiento.
- `output` (Map<String, Object>): Mapa de salida (no utilizado en la implementación actual).
- `options` (Map<String, Object>): Opciones de configuración para el procesamiento.

#### Retorno
- `Map<String, Object>`: Respuesta procesada del servicio externo, o `null` en caso de error.

#### Proceso
1. Obtiene la configuración desde metadatos personalizados.
2. Configura las opciones de eventos de plataforma.
3. Ejecuta el pre-procesamiento.
4. Realiza la llamada HTTP al servicio externo.
5. Procesa la respuesta y la estructura en un formato estándar.
6. Publica eventos de plataforma con los resultados.
7. Ejecuta el post-procesamiento.
8. Maneja excepciones y devuelve resultados.

#### Código
```apex
global Map<String, Object> invokeProcess(
  Map<String, Object> input,
  Map<String, Object> output,
  Map<String, Object> options
) {
  // Obtener configuración de metadatos
  SF_IntegrationSetting__mdt config = Test.isRunningTest()
    ? ExternalServiceMetadataIntegrationMock.getIntegrationSettings(SURATECH_MDT_CONFIG_NAME)
    : SF_IntegrationSetting__mdt.getInstance(SURATECH_MDT_CONFIG_NAME);
    
  if (config == null) {
    throw new ExternalServiceException(
      'No existing "' + SURATECH_MDT_CONFIG_NAME + 
      '" custom metadata type record at Integration Setting'
    );
  }

  // Configurar opciones
  options.put('PlatformEventEnabled', config.Platform_Events_Enabled__c);

  ExternalNotificationHandler enh = new ExternalNotificationHandler();
  Map<String, Object> payload = new Map<String, Object>();

  Boolean ignoreHash = options.containsKey('ignoreHash')
    ? (Boolean) options.get('ignoreHash')
    : false;
    
  try {
    payload = this.preProcess(input, options);
    Core.debug('Starting URL Generation Integration Service In Process');
    publishPlatformEventIntegration('in', JSON.serialize(input));

    // Realizar llamada HTTP
    System.HTTPResponse response = enh.emit(
      SERVICE_METHOD,
      config,
      '',
      (Object) payload
    );

    // Procesar respuesta
    Map<String, Object> mapResponse = new Map<String, Object>{
      'body' => (Map<String, Object>) JSON.deserializeUntyped(response.getBody()),
      'status' => response.getStatus(),
      'statusCode' => response.getStatusCode()
    };
    
    String responseString = response.getBody();
    System.debug('Response String: ' + responseString);

    this.publishPlatFormEvent(
      response.getStatusCode(),
      JSON.serialize(payload),
      JSON.serialize(response.getBody())
    );
    
    return this.postProcess(mapResponse, options);
  } catch (Exception e) {
    enh.controlException(
      'Generate URL remote service',
      e,
      JSON.serialize(payload)
    );
    return null;
  }
}
```

### `postProcess(Map<String, Object> response, Map<String, Object> options)`

#### Descripción
Método virtual que se ejecuta después del procesamiento principal. Realiza tareas de finalización y opcionalmente publica eventos de plataforma con la respuesta final.

#### Parámetros
- `response` (Map<String, Object>): Respuesta del procesamiento principal.
- `options` (Map<String, Object>): Opciones de configuración.

#### Retorno
- `Map<String, Object>`: La respuesta, potencialmente modificada durante el post-procesamiento.

#### Código
```apex
global virtual Map<String, Object> postProcess(
  Map<String, Object> response,
  Map<String, Object> options
) {
  Core.debug('Starting Generate URL Integration Service Post Process');
  if (
    options.containsKey('PlatformEventEnabled') &&
    ((Boolean) options.get('PlatformEventEnabled')) == true
  )
    publishPlatformEventIntegration('post', JSON.serialize(response));
  return response;
}
```

### `publishPlatformEventIntegration(String moment, String data)`

#### Descripción
Método que publica eventos de plataforma en diferentes momentos del procesamiento para proporcionar visibilidad y auditoría del proceso de integración.

#### Parámetros
- `moment` (String): Momento del procesamiento ('pre', 'in', 'post').
- `data` (String): Datos a incluir en el evento de plataforma.

#### Retorno
- `void`: Este método no devuelve ningún valor.

#### Proceso
1. Evalúa el momento especificado usando un switch.
2. Llama al método de publicación de evento apropiado según el momento.
3. Incluye información contextual específica para cada momento.

#### Código
```apex
global void publishPlatformEventIntegration(String moment, String data) {
  switch on moment.toLowerCase() {
    when 'pre' {
      publishInitPrePlatformEvent(data, 'Generate URL Integration Service');
    }
    when 'post' {
      publishInitPosPlatformEvent(data, 'Generate URL Integration Service');
    }
    when 'in' {
      publishPlatFormEvent(200, data, 'Starting Generate URL In process');
    }
  }
}
```

## Dependencias

- **`PlatformEventIntegrationObserver`**: Clase base que proporciona funcionalidad de eventos de plataforma.
- **`SF_IntegrationSetting__mdt`**: Tipo de metadatos personalizados que almacena la configuración de integración.
- **`ExternalNotificationHandler`**: Clase que maneja las comunicaciones HTTP con servicios externos.
- **`ExternalServiceException`**: Excepción personalizada para errores de servicios externos.
- **`ExternalServiceMetadataIntegrationMock`**: Clase mock para proporcionar configuración durante las pruebas.
- **`Core`**: Clase utilitaria que proporciona funcionalidad de depuración.

## Patrón de Diseño Implementado

### Template Method
La clase implementa el patrón Template Method a través de los métodos `preProcess`, `invokeProcess` y `postProcess`, definiendo un algoritmo de procesamiento en pasos donde algunos pasos pueden ser sobrescritos por subclases.

### Observer
A través de la herencia de `PlatformEventIntegrationObserver`, la clase implementa el patrón Observer para la publicación de eventos de plataforma.

## Ejemplos de Uso

### Ejemplo 1: Uso Básico del Servicio

```apex
public class URLGenerationService {
    
    public static Map<String, Object> generatePaymentURL(Map<String, Object> paymentData) {
        try {
            // Crear instancia del handler
            GenerateURLServiceHandler handler = new GenerateURLServiceHandler();
            
            // Preparar opciones
            Map<String, Object> options = new Map<String, Object>{
                'PlatformEventEnabled' => true,
                'ignoreHash' => false
            };
            
            // Preparar datos de entrada
            Map<String, Object> input = new Map<String, Object>{
                'amount' => paymentData.get('amount'),
                'currency' => paymentData.get('currency'),
                'customerInfo' => paymentData.get('customerInfo'),
                'returnUrl' => paymentData.get('returnUrl')
            };
            
            // Ejecutar el procesamiento
            Map<String, Object> result = handler.invokeProcess(input, new Map<String, Object>(), options);
            
            if (result != null && result.containsKey('body')) {
                Map<String, Object> responseBody = (Map<String, Object>) result.get('body');
                return new Map<String, Object>{
                    'success' => true,
                    'paymentUrl' => responseBody.get('url'),
                    'transactionId' => responseBody.get('transactionId')
                };
            } else {
                return new Map<String, Object>{
                    'success' => false,
                    'error' => 'No se pudo generar la URL de pago'
                };
            }
        } catch (Exception e) {
            System.debug('Error generando URL de pago: ' + e.getMessage());
            return new Map<String, Object>{
                'success' => false,
                'error' => e.getMessage()
            };
        }
    }
}

// Uso del servicio
Map<String, Object> paymentData = new Map<String, Object>{
    'amount' => 150000,
    'currency' => 'COP',
    'customerInfo' => new Map<String, Object>{
        'name' => 'Juan Pérez',
        'email' => 'juan@example.com',
        'phone' => '3001234567'
    },
    'returnUrl' => 'https://mysite.com/payment/return'
};

Map<String, Object> result = URLGenerationService.generatePaymentURL(paymentData);

if ((Boolean) result.get('success')) {
    String paymentUrl = (String) result.get('paymentUrl');
    System.debug('URL de pago generada: ' + paymentUrl);
} else {
    System.debug('Error: ' + result.get('error'));
}
```

### Ejemplo 2: Extensión de la Clase para Funcionalidad Específica

```apex
public class EnhancedURLServiceHandler extends GenerateURLServiceHandler {
    
    // Sobrescribir pre-procesamiento para validaciones adicionales
    global override Map<String, Object> preProcess(
        Map<String, Object> input,
        Map<String, Object> options
    ) {
        Core.debug('Starting Enhanced URL Service Pre Process');
        
        // Ejecutar validaciones personalizadas
        validateInputData(input);
        
        // Enriquecer datos de entrada
        input = enrichInputData(input);
        
        // Llamar al método padre para mantener funcionalidad base
        return super.preProcess(input, options);
    }
    
    // Sobrescribir post-procesamiento para transformaciones adicionales
    global override Map<String, Object> postProcess(
        Map<String, Object> response,
        Map<String, Object> options
    ) {
        Core.debug('Starting Enhanced URL Service Post Process');
        
        // Procesar respuesta
        response = transformResponse(response);
        
        // Crear registros de auditoría
        createAuditRecord(response, options);
        
        // Llamar al método padre
        return super.postProcess(response, options);
    }
    
    private void validateInputData(Map<String, Object> input) {
        if (!input.containsKey('amount') || input.get('amount') == null) {
            throw new CumuloMissingParameterException('El monto es requerido para generar la URL de pago');
        }
        
        Decimal amount = (Decimal) input.get('amount');
        if (amount <= 0) {
            throw new IllegalArgumentException('El monto debe ser mayor que cero');
        }
        
        if (!input.containsKey('customerInfo')) {
            throw new CumuloMissingParameterException('La información del cliente es requerida');
        }
    }
    
    private Map<String, Object> enrichInputData(Map<String, Object> input) {
        // Añadir timestamp
        input.put('timestamp', System.now().getTime());
        
        // Añadir información de la organización
        input.put('organizationId', UserInfo.getOrganizationId());
        
        // Generar ID de transacción único
        input.put('transactionRef', generateTransactionReference());
        
        return input;
    }
    
    private Map<String, Object> transformResponse(Map<String, Object> response) {
        if (response.containsKey('body')) {
            Map<String, Object> body = (Map<String, Object>) response.get('body');
            
            // Añadir información adicional a la respuesta
            body.put('processedAt', System.now());
            body.put('processedBy', UserInfo.getUserId());
            
            response.put('body', body);
        }
        
        return response;
    }
    
    private void createAuditRecord(Map<String, Object> response, Map<String, Object> options) {
        try {
            // Crear registro de auditoría en objeto personalizado
            URL_Generation_Audit__c audit = new URL_Generation_Audit__c(
                Response_Status_Code__c = (Integer) response.get('statusCode'),
                Response_Status__c = (String) response.get('status'),
                Platform_Events_Enabled__c = (Boolean) options.get('PlatformEventEnabled'),
                Processing_Date__c = System.now()
            );
            
            insert audit;
        } catch (Exception e) {
            System.debug('Error creando registro de auditoría: ' + e.getMessage());
        }
    }
    
    private String generateTransactionReference() {
        return 'TXN_' + String.valueOf(System.currentTimeMillis()) + '_' + 
               String.valueOf(Math.random()).substring(2, 8);
    }
}
```

### Ejemplo 3: Uso en Procedimiento de Integración

```apex
public class PaymentIntegrationProcedure {
    
    public static Map<String, Object> generatePaymentURL(Map<String, Object> input) {
        try {
            // Extraer datos del input del procedimiento de integración
            Map<String, Object> paymentInfo = (Map<String, Object>) input.get('paymentInfo');
            Map<String, Object> options = (Map<String, Object>) input.get('options');
            
            if (options == null) {
                options = new Map<String, Object>{
                    'PlatformEventEnabled' => true
                };
            }
            
            // Crear handler y procesar
            GenerateURLServiceHandler handler = new GenerateURLServiceHandler();
            Map<String, Object> result = handler.invokeProcess(
                paymentInfo, 
                new Map<String, Object>(), 
                options
            );
            
            // Estructurar respuesta para el procedimiento de integración
            if (result != null) {
                return new Map<String, Object>{
                    'success' => true,
                    'data' => result,
                    'message' => 'URL de pago generada exitosamente'
                };
            } else {
                return new Map<String, Object>{
                    'success' => false,
                    'data' => null,
                    'message' => 'Error al generar URL de pago'
                };
            }
        } catch (ExternalServiceException e) {
            return new Map<String, Object>{
                'success' => false,
                'data' => null,
                'message' => 'Error de servicio externo: ' + e.getMessage(),
                'errorType' => 'EXTERNAL_SERVICE_ERROR'
            };
        } catch (Exception e) {
            return new Map<String, Object>{
                'success' => false,
                'data' => null,
                'message' => 'Error inesperado: ' + e.getMessage(),
                'errorType' => 'UNEXPECTED_ERROR'
            };
        }
    }
}
```

## Configuración de Metadatos Personalizados

La clase utiliza el registro de metadatos personalizados `SF_IntegrationSetting__mdt` con el nombre `url_endpoint`. Este registro debe contener:

| Campo | Descripción |
|-------|-------------|
| `Platform_Events_Enabled__c` | Boolean que indica si los eventos de plataforma están habilitados |
| Campos de endpoint | URL del servicio externo, credenciales, timeouts, etc. |

Ejemplo de configuración:
```
Label: URL Endpoint Configuration
Developer Name: url_endpoint
Platform_Events_Enabled__c: true
Endpoint__c: https://api.payment-provider.com/v1/generate-url
Timeout__c: 30000
API_Key__c: [encrypted]
```

## Mejores Prácticas Implementadas

1. **Separación de Responsabilidades**: La clase separa claramente las fases de pre-procesamiento, procesamiento principal y post-procesamiento.

2. **Configuración Externalizada**: Utiliza metadatos personalizados para la configuración, permitiendo cambios sin modificar código.

3. **Manejo de Errores**: Implementa manejo centralizado de excepciones.

4. **Observabilidad**: Proporciona eventos de plataforma para monitoreo y auditoría.

5. **Testabilidad**: Incluye soporte para mocks durante las pruebas.

## Consideraciones de Seguridad

1. **Datos Sensibles**: La clase serializa datos de entrada y respuesta para eventos de plataforma. Asegurar que no se expongan datos sensibles.

2. **Configuración de Metadatos**: Las credenciales y endpoints deben estar adecuadamente protegidos en los metadatos personalizados.

3. **Validación de Entrada**: Aunque hay comentarios sobre validación de esquema, la implementación actual no incluye validaciones robustas.

## Consideraciones de Rendimiento

1. **Llamadas HTTP**: Las operaciones son síncronas y pueden afectar el tiempo de respuesta total.

2. **Eventos de Plataforma**: La publicación de múltiples eventos puede tener un impacto en el rendimiento.

3. **Serialización JSON**: La serialización repetida de objetos grandes puede ser costosa.

## Notas Adicionales

1. **Código Comentado**: La clase contiene código comentado que sugiere funcionalidades planeadas o alternativas (como `INTEGRATION_PROCEDURE_NAME` y `SERVICE_NAME`).

2. **Parámetro Ignorado**: El parámetro `ignoreHash` se lee pero no se utiliza en la lógica actual.

3. **Métodos Virtuales**: Los métodos `preProcess` y `postProcess` son virtuales, permitiendo su sobrescritura en subclases.

4. **Autor Modificado**: El campo "last modified by" muestra un placeholder, sugiriendo un proceso de configuración incompleto.

5. **Flexibilidad**: La naturaleza virtual de la clase permite extensión y personalización para diferentes tipos de servicios de generación de URL.