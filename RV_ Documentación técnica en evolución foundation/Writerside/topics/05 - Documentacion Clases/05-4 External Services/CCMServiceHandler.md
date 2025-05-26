# CCMServiceHandler

## Descripción General
`CCMServiceHandler` es una clase virtual global que extiende `PlatformEventIntegrationObserver` y está diseñada para manejar la integración con servicios CCM (Customer Communication Management). Esta clase implementa un patrón de procesamiento en tres fases y proporciona capacidades especializadas para el envío de documentos a través de un sistema de mensajería con enrutamiento específico.

## Autor
Felipe Correa \<felipe.correa@nespon.com\>

## Proyecto
Foundation Salesforce - Suratech - UH-17589

## Última Modificación
20 de febrero de 2025 por "ChangeMeIn@UserSettingsUnder.SFDoc"

## Estructura de la Clase

```apex
global virtual class CCMServiceHandler extends PlatformEventIntegrationObserver {
  // Constantes de configuración
  private static final String SERVICE_METHOD = 'POST';
  private static final String SURATECH_MDT_CONFIG_NAME = 'ccm_endpoint';
  
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
| `SERVICE_METHOD` | `'POST'` | Método HTTP utilizado para las llamadas al servicio CCM |
| `SURATECH_MDT_CONFIG_NAME` | `'ccm_endpoint'` | Nombre del registro de metadatos personalizados que contiene la configuración del servicio CCM |

## Métodos

### `preProcess(Map<String, Object> input, Map<String, Object> options)`

#### Descripción
Método virtual que se ejecuta antes del procesamiento principal. Realiza la preparación inicial de los datos y opcionalmente publica eventos de plataforma para el monitoreo del proceso CCM.

#### Parámetros
- `input` (Map<String, Object>): Datos de entrada para el procesamiento CCM.
- `options` (Map<String, Object>): Opciones de configuración para el procesamiento.

#### Retorno
- `Map<String, Object>`: Los datos de entrada, potencialmente modificados durante el pre-procesamiento.

#### Proceso
1. Registra el inicio del pre-procesamiento CCM.
2. Verifica si los eventos de plataforma están habilitados en las opciones.
3. Si están habilitados, publica un evento de plataforma con los datos de entrada.
4. Devuelve los datos de entrada sin modificaciones.

#### Código
```apex
global virtual Map<String, Object> preProcess(
  Map<String, Object> input,
  Map<String, Object> options
) {
  Core.debug('Starting CCM Integration Service Pre Process');
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
Método principal que ejecuta la lógica de integración con el servicio CCM. Maneja la configuración, codificación especial de datos, comunicación con el sistema de mensajería y procesamiento de respuestas.

#### Parámetros
- `input` (Map<String, Object>): Datos de entrada para el procesamiento CCM.
- `output` (Map<String, Object>): Mapa de salida (no utilizado en la implementación actual).
- `options` (Map<String, Object>): Opciones de configuración para el procesamiento.

#### Retorno
- `Map<String, Object>`: Respuesta procesada del servicio CCM, o `null` en caso de error.

#### Proceso
1. Obtiene la configuración desde metadatos personalizados.
2. Configura las opciones de eventos de plataforma.
3. Ejecuta el pre-procesamiento.
4. Codifica el payload en Base64 para transmisión segura.
5. Estructura el mensaje con propiedades específicas para el sistema de mensajería.
6. Realiza la llamada HTTP al servicio CCM.
7. Procesa la respuesta y la estructura en un formato estándar.
8. Publica eventos de plataforma con los resultados.
9. Ejecuta el post-procesamiento.

#### Características Específicas de CCM
- **Codificación Base64**: Los datos se codifican en Base64 para transmisión segura.
- **Sistema de Mensajería**: Utiliza un routing key específico ('CDD.ccm.document').
- **Metadata de Mensaje**: Incluye propiedades como content-type y payload_encoding.

#### Código
```apex
global Map<String, Object> invokeProcess(
  Map<String, Object> input,
  Map<String, Object> output,
  Map<String, Object> options
) {
  // Obtener configuración
  SF_IntegrationSetting__mdt config = Test.isRunningTest()
    ? ExternalServiceMetadataIntegrationMock.getIntegrationSettings(SURATECH_MDT_CONFIG_NAME)
    : SF_IntegrationSetting__mdt.getInstance(SURATECH_MDT_CONFIG_NAME);
    
  if (config == null) {
    throw new ExternalServiceException(
      'No existing "' + SURATECH_MDT_CONFIG_NAME + 
      '" custom metadata type record at Integration Setting'
    );
  }

  options.put('PlatformEventEnabled', config.Platform_Events_Enabled__c);

  ExternalNotificationHandler enh = new ExternalNotificationHandler();
  Map<String, Object> payload = new Map<String, Object>();
  
  try {
    input = this.preProcess(input, options);
    Core.debug('Starting CCM Integration Service In Process');
    publishPlatformEventIntegration('in', JSON.serialize(input));

    // Codificación especial para CCM
    String payloadString = JSON.serialize(input);
    String encoded = EncodingUtil.base64Encode(Blob.valueOf(JSON.serialize(payloadString)));

    // Estructura del mensaje para CCM
    payload.put('payload_encoding', 'base64');
    payload.put('properties', new Map<String, Object>{ 'content-type' => 'application/json' });
    payload.put('routing_key', 'CDD.ccm.document');
    payload.put('payload', '{"Content":"' + encoded + '"}');

    // Enviar al servicio
    System.HTTPResponse response = enh.emit(SERVICE_METHOD, config, '', (Object) payload);

    // Procesar respuesta
    Map<String, Object> responseMap = new Map<String, Object>{
      'body' => (Map<String, Object>) JSON.deserializeUntyped(response.getBody()),
      'status' => response.getStatus(),
      'statusCode' => response.getStatusCode()
    };

    this.publishPlatFormEvent(
      response.getStatusCode(),
      JSON.serialize(payload),
      JSON.serialize(response.getBody())
    );

    System.debug('Response Wrapper: ' + responseMap);
    return this.postProcess(responseMap, options);
  } catch (Exception e) {
    enh.controlException('CCM remote service', e, JSON.serialize(payload));
    return null;
  }
}
```

### `postProcess(Map<String, Object> response, Map<String, Object> options)`

#### Descripción
Método virtual que se ejecuta después del procesamiento principal. Realiza tareas de finalización específicas de CCM y opcionalmente publica eventos de plataforma con la respuesta final.

#### Parámetros
- `response` (Map<String, Object>): Respuesta del procesamiento principal CCM.
- `options` (Map<String, Object>): Opciones de configuración.

#### Retorno
- `Map<String, Object>`: La respuesta, potencialmente modificada durante el post-procesamiento.

#### Código
```apex
global virtual Map<String, Object> postProcess(
  Map<String, Object> response,
  Map<String, Object> options
) {
  Core.debug('Starting CCM Integration Service Post Process');
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
Método que publica eventos de plataforma específicos para CCM en diferentes momentos del procesamiento para proporcionar visibilidad y auditoría del proceso de integración CCM.

#### Parámetros
- `moment` (String): Momento del procesamiento ('pre', 'in', 'post').
- `data` (String): Datos a incluir en el evento de plataforma.

#### Retorno
- `void`: Este método no devuelve ningún valor.

#### Proceso
1. Evalúa el momento especificado usando un switch.
2. Llama al método de publicación de evento apropiado según el momento.
3. Incluye información contextual específica para CCM en cada momento.

#### Código
```apex
global void publishPlatformEventIntegration(String moment, String data) {
  switch on moment.toLowerCase() {
    when 'pre' {
      publishInitPrePlatformEvent(data, 'CCM Integration Service');
    }
    when 'post' {
      publishInitPosPlatformEvent(data, 'CCM Integration Service');
    }
    when 'in' {
      publishPlatFormEvent(200, data, 'Starting CCM In process');
    }
  }
}
```

## Dependencias

- **`PlatformEventIntegrationObserver`**: Clase base que proporciona funcionalidad de eventos de plataforma.
- **`SF_IntegrationSetting__mdt`**: Tipo de metadatos personalizados que almacena la configuración de integración CCM.
- **`ExternalNotificationHandler`**: Clase que maneja las comunicaciones HTTP con servicios externos.
- **`ExternalServiceException`**: Excepción personalizada para errores de servicios externos.
- **`ExternalServiceMetadataIntegrationMock`**: Clase mock para proporcionar configuración durante las pruebas.
- **`Core`**: Clase utilitaria que proporciona funcionalidad de depuración.

## Características Específicas de CCM

### Sistema de Mensajería
La clase está diseñada para trabajar con un sistema de mensajería que utiliza:
- **Routing Key**: `'CDD.ccm.document'` para enrutar mensajes CCM
- **Codificación Base64**: Para transmisión segura de documentos
- **Propiedades de Mensaje**: Metadatos específicos del mensaje

### Estructura del Payload
```json
{
  "payload_encoding": "base64",
  "properties": {
    "content-type": "application/json"
  },
  "routing_key": "CDD.ccm.document",
  "payload": "{\"Content\":\"<base64_encoded_data>\"}"
}
```

## Ejemplos de Uso

### Ejemplo 1: Envío de Documento CCM Básico

```apex
public class CCMDocumentService {
    
    public static Map<String, Object> sendDocument(Map<String, Object> documentData) {
        try {
            // Crear instancia del handler CCM
            CCMServiceHandler handler = new CCMServiceHandler();
            
            // Preparar opciones
            Map<String, Object> options = new Map<String, Object>{
                'PlatformEventEnabled' => true
            };
            
            // Preparar datos del documento
            Map<String, Object> input = new Map<String, Object>{
                'documentType' => documentData.get('documentType'),
                'recipientEmail' => documentData.get('recipientEmail'),
                'templateId' => documentData.get('templateId'),
                'documentData' => documentData.get('documentData'),
                'metadata' => new Map<String, Object>{
                    'senderId' => UserInfo.getUserId(),
                    'organizationId' => UserInfo.getOrganizationId(),
                    'timestamp' => System.now()
                }
            };
            
            // Ejecutar el procesamiento CCM
            Map<String, Object> result = handler.invokeProcess(input, new Map<String, Object>(), options);
            
            if (result != null && result.containsKey('body')) {
                Map<String, Object> responseBody = (Map<String, Object>) result.get('body');
                return new Map<String, Object>{
                    'success' => true,
                    'messageId' => responseBody.get('messageId'),
                    'status' => result.get('status'),
                    'statusCode' => result.get('statusCode')
                };
            } else {
                return new Map<String, Object>{
                    'success' => false,
                    'error' => 'No se pudo enviar el documento CCM'
                };
            }
        } catch (Exception e) {
            System.debug('Error enviando documento CCM: ' + e.getMessage());
            return new Map<String, Object>{
                'success' => false,
                'error' => e.getMessage()
            };
        }
    }
}

// Uso del servicio
Map<String, Object> documentData = new Map<String, Object>{
    'documentType' => 'POLICY_CERTIFICATE',
    'recipientEmail' => 'customer@example.com',
    'templateId' => 'TEMPLATE_001',
    'documentData' => new Map<String, Object>{
        'policyNumber' => 'POL-123456',
        'customerName' => 'Juan Pérez',
        'effectiveDate' => Date.today(),
        'premium' => 150000
    }
};

Map<String, Object> result = CCMDocumentService.sendDocument(documentData);

if ((Boolean) result.get('success')) {
    String messageId = (String) result.get('messageId');
    System.debug('Documento CCM enviado exitosamente. Message ID: ' + messageId);
} else {
    System.debug('Error: ' + result.get('error'));
}
```

### Ejemplo 2: Extensión Especializada para Comunicaciones de Seguros

```apex
public class InsuranceCCMHandler extends CCMServiceHandler {
    
    // Sobrescribir pre-procesamiento para validaciones específicas de seguros
    global override Map<String, Object> preProcess(
        Map<String, Object> input,
        Map<String, Object> options
    ) {
        Core.debug('Starting Insurance CCM Service Pre Process');
        
        // Validar datos específicos de seguros
        validateInsuranceDocument(input);
        
        // Enriquecer con datos de la póliza
        input = enrichWithPolicyData(input);
        
        // Llamar al método padre
        return super.preProcess(input, options);
    }
    
    // Sobrescribir post-procesamiento para acciones específicas
    global override Map<String, Object> postProcess(
        Map<String, Object> response,
        Map<String, Object> options
    ) {
        Core.debug('Starting Insurance CCM Service Post Process');
        
        // Crear registro de comunicación
        createCommunicationRecord(response, options);
        
        // Actualizar estado de la póliza si es necesario
        updatePolicyStatus(response);
        
        // Llamar al método padre
        return super.postProcess(response, options);
    }
    
    private void validateInsuranceDocument(Map<String, Object> input) {
        if (!input.containsKey('policyNumber')) {
            throw new CumuloMissingParameterException('El número de póliza es requerido para documentos de seguros');
        }
        
        if (!input.containsKey('documentType')) {
            throw new CumuloMissingParameterException('El tipo de documento es requerido');
        }
        
        String documentType = (String) input.get('documentType');
        Set<String> validTypes = new Set<String>{'POLICY_CERTIFICATE', 'CLAIM_NOTICE', 'PAYMENT_RECEIPT', 'RENEWAL_NOTICE'};
        
        if (!validTypes.contains(documentType)) {
            throw new IllegalArgumentException('Tipo de documento no válido para seguros: ' + documentType);
        }
    }
    
    private Map<String, Object> enrichWithPolicyData(Map<String, Object> input) {
        String policyNumber = (String) input.get('policyNumber');
        
        // Consultar información adicional de la póliza
        List<Policy__c> policies = [
            SELECT Id, PolicyNumber, Status, Premium, EffectiveDate, ExpirationDate, Account__r.Name, Account__r.Email
            FROM Policy__c 
            WHERE PolicyNumber = :policyNumber 
            LIMIT 1
        ];
        
        if (!policies.isEmpty()) {
            Policy__c policy = policies[0];
            input.put('policyId', policy.Id);
            input.put('policyStatus', policy.Status);
            input.put('customerName', policy.Account__r.Name);
            input.put('customerEmail', policy.Account__r.Email);
            input.put('effectiveDate', policy.EffectiveDate);
            input.put('expirationDate', policy.ExpirationDate);
        }
        
        return input;
    }
    
    private void createCommunicationRecord(Map<String, Object> response, Map<String, Object> options) {
        try {
            // Crear registro de comunicación en objeto personalizado
            Communication_Log__c commLog = new Communication_Log__c(
                Type__c = 'CCM_DOCUMENT',
                Status__c = (Integer) response.get('statusCode') == 200 ? 'Sent' : 'Failed',
                Response_Status_Code__c = (Integer) response.get('statusCode'),
                Sent_Date__c = System.now(),
                Platform_Events_Enabled__c = (Boolean) options.get('PlatformEventEnabled')
            );
            
            insert commLog;
        } catch (Exception e) {
            System.debug('Error creando registro de comunicación: ' + e.getMessage());
        }
    }
    
    private void updatePolicyStatus(Map<String, Object> response) {
        // Lógica para actualizar estado de póliza basada en la respuesta CCM
        // Por ejemplo, marcar como "Notificada" después de enviar certificado
    }
}
```

### Ejemplo 3: Uso en Procedimiento de Integración

```apex
public class CCMIntegrationProcedure {
    
    public static Map<String, Object> sendCCMDocument(Map<String, Object> input) {
        try {
            // Extraer datos del input del procedimiento
            Map<String, Object> documentInfo = (Map<String, Object>) input.get('documentInfo');
            Map<String, Object> options = (Map<String, Object>) input.get('options');
            
            if (options == null) {
                options = new Map<String, Object>{
                    'PlatformEventEnabled' => true
                };
            }
            
            // Determinar el tipo de handler a usar
            CCMServiceHandler handler;
            String documentType = (String) documentInfo.get('documentType');
            
            if (documentType != null && documentType.startsWith('INSURANCE_')) {
                handler = new InsuranceCCMHandler();
            } else {
                handler = new CCMServiceHandler();
            }
            
            // Procesar con CCM
            Map<String, Object> result = handler.invokeProcess(
                documentInfo, 
                new Map<String, Object>(), 
                options
            );
            
            // Estructurar respuesta para el procedimiento de integración
            if (result != null) {
                return new Map<String, Object>{
                    'success' => true,
                    'data' => result,
                    'message' => 'Documento CCM enviado exitosamente'
                };
            } else {
                return new Map<String, Object>{
                    'success' => false,
                    'data' => null,
                    'message' => 'Error al enviar documento CCM'
                };
            }
        } catch (ExternalServiceException e) {
            return new Map<String, Object>{
                'success' => false,
                'data' => null,
                'message' => 'Error de servicio CCM: ' + e.getMessage(),
                'errorType' => 'CCM_SERVICE_ERROR'
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

### Ejemplo 4: Envío Masivo de Documentos CCM

```apex
public class BulkCCMService {
    
    @future(callout=true)
    public static void sendBulkDocuments(String serializedDocuments) {
        List<Map<String, Object>> documents = 
            (List<Map<String, Object>>) JSON.deserializeUntyped(serializedDocuments);
        
        CCMServiceHandler handler = new CCMServiceHandler();
        List<Map<String, Object>> results = new List<Map<String, Object>>();
        
        Map<String, Object> options = new Map<String, Object>{
            'PlatformEventEnabled' => true
        };
        
        for (Map<String, Object> doc : documents) {
            try {
                Map<String, Object> result = handler.invokeProcess(
                    doc, 
                    new Map<String, Object>(), 
                    options
                );
                
                results.add(new Map<String, Object>{
                    'documentId' => doc.get('documentId'),
                    'success' => result != null,
                    'result' => result
                });
                
                // Pausa breve entre llamadas para evitar throttling
                if (!Test.isRunningTest()) {
                    System.debug('Documento procesado, continuando...');
                }
            } catch (Exception e) {
                results.add(new Map<String, Object>{
                    'documentId' => doc.get('documentId'),
                    'success' => false,
                    'error' => e.getMessage()
                });
            }
        }
        
        // Procesar resultados (guardar en base de datos, enviar notificaciones, etc.)
        processBulkResults(results);
    }
    
    private static void processBulkResults(List<Map<String, Object>> results) {
        Integer successCount = 0;
        Integer failureCount = 0;
        
        for (Map<String, Object> result : results) {
            if ((Boolean) result.get('success')) {
                successCount++;
            } else {
                failureCount++;
                System.debug('Error en documento ' + result.get('documentId') + ': ' + result.get('error'));
            }
        }
        
        System.debug('Procesamiento masivo CCM completado. Éxitos: ' + successCount + ', Fallos: ' + failureCount);
        
        // Crear registro de resumen
        createBulkProcessingSummary(successCount, failureCount, results.size());
    }
    
    private static void createBulkProcessingSummary(Integer success, Integer failures, Integer total) {
        try {
            Bulk_Processing_Log__c summary = new Bulk_Processing_Log__c(
                Process_Type__c = 'CCM_BULK_SEND',
                Total_Records__c = total,
                Successful_Records__c = success,
                Failed_Records__c = failures,
                Processing_Date__c = System.now(),
                Success_Rate__c = total > 0 ? (Decimal.valueOf(success) / total * 100).setScale(2) : 0
            );
            
            insert summary;
        } catch (Exception e) {
            System.debug('Error creando resumen de procesamiento masivo: ' + e.getMessage());
        }
    }
}
```

## Configuración de Metadatos Personalizados

La clase utiliza el registro de metadatos personalizados `SF_IntegrationSetting__mdt` con el nombre `ccm_endpoint`. Este registro debe contener:

| Campo | Descripción |
|-------|-------------|
| `Platform_Events_Enabled__c` | Boolean que indica si los eventos de plataforma están habilitados |
| Campos de endpoint CCM | URL del servicio CCM, credenciales, timeouts, etc. |

Ejemplo de configuración:
```
Label: CCM Endpoint Configuration
Developer Name: ccm_endpoint
Platform_Events_Enabled__c: true
Endpoint__c: https://ccm.suratech.com/api/v1/messages
Timeout__c: 45000
API_Key__c: [encrypted]
```

## Consideraciones de Seguridad CCM

1. **Codificación Base64**: Los datos se codifican en Base64 para transmisión segura.
2. **Datos Sensibles**: El contenido del documento puede contener información personal sensible.
3. **Enrutamiento**: El routing key específico asegura que los documentos lleguen al destino correcto.
4. **Configuración Segura**: Las credenciales deben estar protegidas en los metadatos personalizados.

## Consideraciones de Rendimiento

1. **Codificación**: La doble serialización y codificación Base64 puede impactar el rendimiento con documentos grandes.
2. **Tamaño de Payload**: Los documentos grandes pueden exceder límites de tamaño de HTTP.
3. **Latencia de Red**: Las comunicaciones con CCM pueden tener latencia variable.
4. **Throttling**: El servicio CCM puede tener límites de velocidad que deben considerarse.

## Notas Adicionales

1. **Doble Serialización**: El código realiza una doble serialización del payload, lo que puede ser optimizado.
2. **Routing Key Específico**: El routing key 'CDD.ccm.document' sugiere un sistema de mensajería empresarial específico.
3. **Métodos Virtuales**: Permite extensión para casos de uso específicos como seguros o otros dominios.
4. **Sistema de Colas**: La estructura del payload sugiere integración con un sistema de colas o mensaje broker.
5. **Validación Comentada**: Existe código comentado para validación de esquema CCM que podría implementarse.