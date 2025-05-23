# Documentación Técnica: Adapter

## Descripción General
`Adapter` es una interfaz pública que define el contrato fundamental para el patrón de diseño Adaptador en el framework Foundation de Suratech. Esta interfaz establece un método estándar que permite adaptar diferentes tipos de servicios, APIs o sistemas externos a una estructura común de datos, facilitando la integración y el intercambio de información entre componentes heterogéneos.

## Autor
Soulberto Lorenzo \<soulberto@cloudblue.us\>

## Última Modificación
2 de septiembre de 2024

## Definición de la Interfaz

```apex
@namespaceAccessible
public interface Adapter {
  Map<String, Object> adaptee();
}
```

## Decoradores y Accesibilidad

### `@namespaceAccessible`
La interfaz está marcada con `@namespaceAccessible`, lo que indica que está diseñada para ser accesible desde diferentes espacios de nombres (namespaces). Esto es especialmente relevante en contextos de paquetes gestionados donde se necesita que la interfaz sea accesible desde fuera del namespace del paquete.

## Métodos

### `adaptee()`

#### Descripción
Método que debe ser implementado por todas las clases que implementen esta interfaz. Su propósito es adaptar la funcionalidad específica de un servicio, API o sistema a un formato estándar de mapa de datos.

#### Parámetros
Ninguno.

#### Retorno
- `Map<String, Object>`: Mapa que contiene los datos adaptados en un formato estándar que puede ser procesado de manera uniforme por otros componentes del sistema.

## Propósito y Aplicaciones

El propósito principal de esta interfaz es implementar el patrón de diseño Adaptador, que permite:

1. **Integración de Sistemas Heterogéneos**: Adaptar diferentes APIs, servicios o sistemas a una interfaz común.
2. **Desacoplamiento**: Separar la lógica específica de integración de la lógica de negocio principal.
3. **Flexibilidad**: Permitir el intercambio de diferentes implementaciones sin afectar el código cliente.
4. **Estandarización**: Proporcionar un formato común de datos para sistemas diversos.
5. **Extensibilidad**: Facilitar la adición de nuevos tipos de adaptadores sin modificar código existente.

Esta interfaz es especialmente útil en escenarios como:

- **Integraciones con APIs Externas**: Adaptar respuestas de diferentes APIs a un formato común.
- **Migración de Sistemas**: Facilitar la transición entre diferentes sistemas o versiones.
- **Compatibilidad con Múltiples Proveedores**: Permitir trabajar con diferentes proveedores de servicios usando la misma interfaz.
- **Transformación de Datos**: Convertir datos de un formato a otro de manera consistente.

## Patrón de Diseño: Adapter

### Estructura del Patrón
```
Client -> Target Interface (Adapter) -> Adaptee (Specific Implementation)
```

En este caso:
- **Client**: Código que utiliza el adaptador
- **Target Interface**: La interfaz `Adapter`
- **Adaptee**: Las implementaciones específicas que adaptan diferentes servicios

### Beneficios del Patrón
1. **Reutilización de Código**: Permite reutilizar clases existentes con interfaces incompatibles.
2. **Separación de Responsabilidades**: Separa la lógica de adaptación de la lógica de negocio.
3. **Flexibilidad**: Facilita el cambio de implementaciones sin afectar el código cliente.

## Implementaciones Existentes

### `InsServiceAdapter`
Una implementación conocida de esta interfaz que se utiliza para adaptar servicios de Vlocity Insurance:

```apex
@namespaceAccessible
public class InsServiceAdapter implements Adapter {
  @namespaceAccessible
  public Map<String, Object> adaptee() {
    // return new URLGenerationNotificationHandler();
    return new Map<String, Object>();
  }
}
```

## Ejemplos de Implementación

### Ejemplo 1: Adaptador para Servicios de Pago

```apex
public class PaymentServiceAdapter implements Adapter {
    
    private String paymentProvider;
    private Map<String, Object> configuration;
    
    public PaymentServiceAdapter(String paymentProvider, Map<String, Object> configuration) {
        this.paymentProvider = paymentProvider;
        this.configuration = configuration;
    }
    
    public Map<String, Object> adaptee() {
        try {
            switch on paymentProvider.toLowerCase() {
                when 'stripe' {
                    return adaptStripeService();
                }
                when 'paypal' {
                    return adaptPayPalService();
                }
                when 'square' {
                    return adaptSquareService();
                }
                when else {
                    throw new AdapterException('Unsupported payment provider: ' + paymentProvider);
                }
            }
        } catch (Exception e) {
            return new Map<String, Object>{
                'success' => false,
                'error' => e.getMessage(),
                'provider' => paymentProvider
            };
        }
    }
    
    private Map<String, Object> adaptStripeService() {
        // Lógica específica para adaptar Stripe
        return new Map<String, Object>{
            'provider' => 'stripe',
            'apiVersion' => '2023-10-16',
            'supportedMethods' => new List<String>{'card', 'bank_transfer', 'apple_pay'},
            'configuration' => new Map<String, Object>{
                'publicKey' => configuration.get('stripe_public_key'),
                'webhookEndpoint' => configuration.get('stripe_webhook_url')
            },
            'capabilities' => new Map<String, Object>{
                'subscriptions' => true,
                'refunds' => true,
                'disputes' => true,
                'payouts' => true
            }
        };
    }
    
    private Map<String, Object> adaptPayPalService() {
        // Lógica específica para adaptar PayPal
        return new Map<String, Object>{
            'provider' => 'paypal',
            'apiVersion' => 'v2',
            'supportedMethods' => new List<String>{'paypal', 'card', 'venmo'},
            'configuration' => new Map<String, Object>{
                'clientId' => configuration.get('paypal_client_id'),
                'webhookId' => configuration.get('paypal_webhook_id')
            },
            'capabilities' => new Map<String, Object>{
                'subscriptions' => true,
                'refunds' => true,
                'disputes' => true,
                'payouts' => false
            }
        };
    }
    
    private Map<String, Object> adaptSquareService() {
        // Lógica específica para adaptar Square
        return new Map<String, Object>{
            'provider' => 'square',
            'apiVersion' => '2023-12-13',
            'supportedMethods' => new List<String>{'card', 'cash_app', 'afterpay'},
            'configuration' => new Map<String, Object>{
                'applicationId' => configuration.get('square_application_id'),
                'locationId' => configuration.get('square_location_id')
            },
            'capabilities' => new Map<String, Object>{
                'subscriptions' => false,
                'refunds' => true,
                'disputes' => true,
                'payouts' => true
            }
        };
    }
    
    public class AdapterException extends Exception {}
}
```

### Ejemplo 2: Adaptador para APIs de Notificación

```apex
public class NotificationServiceAdapter implements Adapter {
    
    private String notificationService;
    private Map<String, Object> serviceConfig;
    
    public NotificationServiceAdapter(String notificationService, Map<String, Object> serviceConfig) {
        this.notificationService = notificationService;
        this.serviceConfig = serviceConfig;
    }
    
    public Map<String, Object> adaptee() {
        Map<String, Object> adapterInfo = new Map<String, Object>{
            'service' => notificationService,
            'timestamp' => System.now(),
            'isAvailable' => checkServiceAvailability()
        };
        
        switch on notificationService.toLowerCase() {
            when 'sendgrid' {
                adapterInfo.putAll(adaptSendGridService());
            }
            when 'twilio' {
                adapterInfo.putAll(adaptTwilioService());
            }
            when 'slack' {
                adapterInfo.putAll(adaptSlackService());
            }
            when 'teams' {
                adapterInfo.putAll(adaptTeamsService());
            }
            when else {
                adapterInfo.put('error', 'Unsupported notification service: ' + notificationService);
                adapterInfo.put('isAvailable', false);
            }
        }
        
        return adapterInfo;
    }
    
    private Map<String, Object> adaptSendGridService() {
        return new Map<String, Object>{
            'type' => 'email',
            'endpoint' => 'https://api.sendgrid.com/v3/mail/send',
            'authentication' => 'api_key',
            'maxBatchSize' => 1000,
            'supportedFeatures' => new List<String>{
                'html_content', 'attachments', 'templates', 'scheduling'
            },
            'rateLimits' => new Map<String, Object>{
                'requests_per_second' => 600,
                'emails_per_month' => getEmailQuota()
            }
        };
    }
    
    private Map<String, Object> adaptTwilioService() {
        return new Map<String, Object>{
            'type' => 'sms',
            'endpoint' => 'https://api.twilio.com/2010-04-01/Accounts/' + serviceConfig.get('account_sid') + '/Messages.json',
            'authentication' => 'basic_auth',
            'maxBatchSize' => 1,
            'supportedFeatures' => new List<String>{
                'sms', 'mms', 'whatsapp', 'voice_calls'
            },
            'rateLimits' => new Map<String, Object>{
                'requests_per_second' => 10,
                'messages_per_day' => getSMSQuota()
            }
        };
    }
    
    private Map<String, Object> adaptSlackService() {
        return new Map<String, Object>{
            'type' => 'chat',
            'endpoint' => 'https://slack.com/api/chat.postMessage',
            'authentication' => 'bearer_token',
            'maxBatchSize' => 1,
            'supportedFeatures' => new List<String>{
                'rich_text', 'attachments', 'blocks', 'threads'
            },
            'rateLimits' => new Map<String, Object>{
                'requests_per_minute' => 50,
                'messages_per_channel_per_second' => 1
            }
        };
    }
    
    private Map<String, Object> adaptTeamsService() {
        return new Map<String, Object>{
            'type' => 'chat',
            'endpoint' => serviceConfig.get('webhook_url'),
            'authentication' => 'webhook',
            'maxBatchSize' => 1,
            'supportedFeatures' => new List<String>{
                'adaptive_cards', 'mentions', 'attachments'
            },
            'rateLimits' => new Map<String, Object>{
                'requests_per_second' => 4,
                'requests_per_app_per_second' => 4
            }
        };
    }
    
    private Boolean checkServiceAvailability() {
        // Lógica para verificar disponibilidad del servicio
        // Podría hacer una llamada de ping o verificar configuración
        return serviceConfig != null && !serviceConfig.isEmpty();
    }
    
    private Integer getEmailQuota() {
        // Obtener cuota de emails basada en el plan
        return (Integer) serviceConfig.get('email_quota') ?? 40000;
    }
    
    private Integer getSMSQuota() {
        // Obtener cuota de SMS basada en el plan
        return (Integer) serviceConfig.get('sms_quota') ?? 1000;
    }
}
```

### Ejemplo 3: Adaptador para Servicios de Almacenamiento

```apex
public class StorageServiceAdapter implements Adapter {
    
    private String storageProvider;
    private Map<String, Object> credentials;
    
    public StorageServiceAdapter(String storageProvider, Map<String, Object> credentials) {
        this.storageProvider = storageProvider;
        this.credentials = credentials;
    }
    
    public Map<String, Object> adaptee() {
        Map<String, Object> storageInfo = new Map<String, Object>{
            'provider' => storageProvider,
            'adaptedAt' => System.now()
        };
        
        try {
            switch on storageProvider.toLowerCase() {
                when 'aws_s3' {
                    storageInfo.putAll(adaptAWSS3());
                }
                when 'azure_blob' {
                    storageInfo.putAll(adaptAzureBlob());
                }
                when 'google_cloud' {
                    storageInfo.putAll(adaptGoogleCloud());
                }
                when 'salesforce_files' {
                    storageInfo.putAll(adaptSalesforceFiles());
                }
                when else {
                    throw new UnsupportedStorageException('Provider not supported: ' + storageProvider);
                }
            }
            
            storageInfo.put('isConfigured', true);
            
        } catch (Exception e) {
            storageInfo.put('isConfigured', false);
            storageInfo.put('error', e.getMessage());
        }
        
        return storageInfo;
    }
    
    private Map<String, Object> adaptAWSS3() {
        return new Map<String, Object>{
            'baseUrl' => 'https://s3.amazonaws.com',
            'region' => credentials.get('aws_region') ?? 'us-east-1',
            'bucketName' => credentials.get('bucket_name'),
            'accessKeyId' => credentials.get('access_key_id'),
            'supportedOperations' => new List<String>{
                'upload', 'download', 'delete', 'list', 'copy'
            },
            'maxFileSize' => 5368709120, // 5GB
            'supportedFormats' => new List<String>{
                'pdf', 'jpg', 'png', 'docx', 'xlsx', 'zip'
            },
            'encryption' => new Map<String, Object>{
                'supported' => true,
                'types' => new List<String>{'AES256', 'aws:kms'}
            }
        };
    }
    
    private Map<String, Object> adaptAzureBlob() {
        return new Map<String, Object>{
            'baseUrl' => 'https://' + credentials.get('account_name') + '.blob.core.windows.net',
            'containerName' => credentials.get('container_name'),
            'accountName' => credentials.get('account_name'),
            'supportedOperations' => new List<String>{
                'upload', 'download', 'delete', 'list', 'snapshot'
            },
            'maxFileSize' => 4398046511104, // 4TB
            'supportedFormats' => new List<String>{
                'pdf', 'jpg', 'png', 'docx', 'xlsx', 'zip', 'mp4'
            },
            'encryption' => new Map<String, Object>{
                'supported' => true,
                'types' => new List<String>{'Microsoft Managed', 'Customer Managed'}
            }
        };
    }
    
    private Map<String, Object> adaptGoogleCloud() {
        return new Map<String, Object>{
            'baseUrl' => 'https://storage.googleapis.com',
            'bucketName' => credentials.get('bucket_name'),
            'projectId' => credentials.get('project_id'),
            'supportedOperations' => new List<String>{
                'upload', 'download', 'delete', 'list', 'versioning'
            },
            'maxFileSize' => 5497558138880, // 5TB
            'supportedFormats' => new List<String>{
                'pdf', 'jpg', 'png', 'docx', 'xlsx', 'zip', 'mp4', 'csv'
            },
            'encryption' => new Map<String, Object>{
                'supported' => true,
                'types' => new List<String>{'Google-managed', 'Customer-managed', 'Customer-supplied'}
            }
        };
    }
    
    private Map<String, Object> adaptSalesforceFiles() {
        return new Map<String, Object>{
            'baseUrl' => URL.getSalesforceBaseUrl().toExternalForm(),
            'supportedOperations' => new List<String>{
                'upload', 'download', 'delete', 'share', 'version'
            },
            'maxFileSize' => 2147483648, // 2GB
            'supportedFormats' => new List<String>{
                'pdf', 'jpg', 'png', 'docx', 'xlsx', 'pptx', 'txt'
            },
            'encryption' => new Map<String, Object>{
                'supported' => true,
                'types' => new List<String>{'Platform Encryption'}
            },
            'sharingModel' => 'Salesforce Security Model'
        };
    }
    
    public class UnsupportedStorageException extends Exception {}
}
```

## Uso con Factory Pattern

```apex
public class AdapterFactory {
    
    public static Adapter createAdapter(String adapterType, Map<String, Object> configuration) {
        switch on adapterType.toLowerCase() {
            when 'payment' {
                String provider = (String) configuration.get('provider');
                return new PaymentServiceAdapter(provider, configuration);
            }
            when 'notification' {
                String service = (String) configuration.get('service');
                return new NotificationServiceAdapter(service, configuration);
            }
            when 'storage' {
                String provider = (String) configuration.get('provider');
                return new StorageServiceAdapter(provider, configuration);
            }
            when 'insurance' {
                return new InsServiceAdapter();
            }
            when else {
                throw new UnsupportedAdapterException('Unsupported adapter type: ' + adapterType);
            }
        }
    }
    
    public class UnsupportedAdapterException extends Exception {}
}

// Uso del factory
Map<String, Object> paymentConfig = new Map<String, Object>{
    'provider' => 'stripe',
    'stripe_public_key' => 'pk_test_...',
    'stripe_webhook_url' => 'https://myapp.com/webhooks/stripe'
};

Adapter paymentAdapter = AdapterFactory.createAdapter('payment', paymentConfig);
Map<String, Object> paymentInfo = paymentAdapter.adaptee();
```

## Ejemplo de Uso en Servicios de Integración

```apex
public class IntegrationOrchestrator {
    
    private List<Adapter> adapters;
    
    public IntegrationOrchestrator() {
        this.adapters = new List<Adapter>();
    }
    
    public void addAdapter(Adapter adapter) {
        adapters.add(adapter);
    }
    
    public Map<String, Object> executeIntegration() {
        Map<String, Object> integrationResults = new Map<String, Object>{
            'totalAdapters' => adapters.size(),
            'results' => new List<Map<String, Object>>(),
            'executedAt' => System.now()
        };
        
        List<Map<String, Object>> results = new List<Map<String, Object>>();
        
        for (Integer i = 0; i < adapters.size(); i++) {
            try {
                Adapter adapter = adapters[i];
                Map<String, Object> adapterResult = adapter.adaptee();
                
                results.add(new Map<String, Object>{
                    'adapterIndex' => i,
                    'success' => true,
                    'data' => adapterResult
                });
                
            } catch (Exception e) {
                results.add(new Map<String, Object>{
                    'adapterIndex' => i,
                    'success' => false,
                    'error' => e.getMessage()
                });
            }
        }
        
        integrationResults.put('results', results);
        return integrationResults;
    }
}

// Uso del orquestador
IntegrationOrchestrator orchestrator = new IntegrationOrchestrator();

// Añadir múltiples adaptadores
orchestrator.addAdapter(AdapterFactory.createAdapter('payment', paymentConfig));
orchestrator.addAdapter(AdapterFactory.createAdapter('notification', notificationConfig));
orchestrator.addAdapter(AdapterFactory.createAdapter('storage', storageConfig));

// Ejecutar todas las integraciones
Map<String, Object> results = orchestrator.executeIntegration();
```

## Mejores Prácticas para Implementación

1. **Consistencia en el Formato de Retorno**: Mantener una estructura consistente en el mapa de retorno para facilitar el procesamiento.

2. **Manejo de Errores**: Incluir información de errores en el mapa de retorno en lugar de lanzar excepciones cuando sea posible.

3. **Información de Metadatos**: Incluir metadatos útiles como timestamps, versiones de API, y información de configuración.

4. **Validación de Configuración**: Validar la configuración requerida antes de intentar adaptar el servicio.

5. **Documentación Clara**: Documentar claramente qué estructura de datos devuelve cada implementación.

## Consideraciones de Diseño

### Ventajas
1. **Flexibilidad**: Permite trabajar con múltiples servicios usando la misma interfaz.
2. **Mantenibilidad**: Facilita el mantenimiento al encapsular la lógica específica de cada servicio.
3. **Testabilidad**: Permite crear implementaciones mock fácilmente para pruebas.
4. **Extensibilidad**: Nuevos adaptadores pueden añadirse sin modificar código existente.

### Consideraciones
1. **Complejidad**: Puede añadir complejidad al sistema si se abusa del patrón.
2. **Overhead**: Puede introducir overhead si la adaptación es muy simple.
3. **Consistencia**: Requiere disciplina para mantener consistencia entre implementaciones.

## Notas Adicionales
1. La anotación `@namespaceAccessible` indica que esta interfaz está diseñada para ser utilizada en contextos de paquetes gestionados.
2. El comentario en la documentación del método muestra un parámetro vacío, lo que sugiere que originalmente se planeó que el método tuviera parámetros.
3. Esta interfaz forma parte de un sistema más amplio de adaptadores en el framework Foundation de Suratech.
4. La simplicidad de la interfaz es intencional, proporcionando máxima flexibilidad para las implementaciones.