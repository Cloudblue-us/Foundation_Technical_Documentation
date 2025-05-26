# RUNTServiceHandler

## Descripción General
`RUNTServiceHandler` es una clase virtual global que extiende `PlatformEventIntegrationObserver`. Esta clase está diseñada para manejar la integración con el servicio RUNT (Registro Único Nacional de Tránsito), permitiendo consultar información de vehículos a través de un servicio externo.

## Autor
Felipe Correa \<felipe.correa@nespon.com\>

## Proyecto
Foundation Salesforce - Suratech - UH-17752

## Última Modificación
20 de febrero de 2025

## Constantes

### `SERVICE_METHOD`
- **Valor**: `"GET"`
- **Descripción**: Define el método HTTP utilizado para la comunicación con el servicio externo (GET).

### `SURATECH_MDT_CONFIG_NAME`
- **Valor**: `"runt_endpoint"`
- **Descripción**: Nombre del registro de metadatos personalizado que contiene la configuración de conexión al servicio RUNT.

## Métodos

### `preProcess(Map<String, Object> input, Map<String, Object> options)`

#### Descripción
Método virtual que realiza el pre-procesamiento de la solicitud antes de invocar el servicio externo. Publica un evento de plataforma si está habilitado.

#### Parámetros
- `input` (Map<String, Object>): Datos de entrada para la solicitud.
- `options` (Map<String, Object>): Opciones adicionales para el procesamiento.

#### Retorno
- `Map<String, Object>`: Los datos de entrada, posiblemente modificados durante el pre-procesamiento.

#### Código
```apex
global virtual Map<String, Object> preProcess(
  Map<String, Object> input,
  Map<String, Object> options
) {
  Core.debug('Starting RUNT Integration Service Pre Process');
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
Método principal que orquesta el flujo completo de integración con el servicio RUNT. Obtiene la configuración de metadatos, realiza validaciones, emite la solicitud al servicio externo, procesa la respuesta y maneja excepciones.

#### Parámetros
- `input` (Map<String, Object>): Datos de entrada para la solicitud.
- `output` (Map<String, Object>): Mapa donde se almacenará el resultado de la operación.
- `options` (Map<String, Object>): Opciones adicionales para el procesamiento.

#### Retorno
- `Map<String, Object>`: Mapa con la respuesta procesada o `null` en caso de error.

#### Proceso
1. Obtiene la configuración del servicio desde metadatos personalizados.
2. Marca si los eventos de plataforma están habilitados para este servicio.
3. Crea un manejador de notificaciones externas.
4. Ejecuta el pre-procesamiento.
5. Publica un evento de plataforma para el inicio del proceso.
6. Emite la solicitud al servicio externo mediante el manejador de notificaciones.
7. Procesa la respuesta HTTP.
8. Publica un evento de plataforma con el resultado.
9. Ejecuta el post-procesamiento.
10. Maneja excepciones si ocurren durante el proceso.

#### Código
```apex
global Map<String, Object> invokeProcess(
  Map<String, Object> input,
  Map<String, Object> output,
  Map<String, Object> options
) {
  SF_IntegrationSetting__mdt config = Test.isRunningTest()
    ? ExternalServiceMetadataIntegrationMock.getIntegrationSettings(
        SURATECH_MDT_CONFIG_NAME
      )
    : SF_IntegrationSetting__mdt.getInstance(SURATECH_MDT_CONFIG_NAME);
  if (config == null) {
    throw new ExternalServiceException(
      'No existing "' +
        SURATECH_MDT_CONFIG_NAME +
        '" custom metadata type record at Integration Setting'
    );
  }

  //Marcamos la activacion si debemos ejecutar Eventos de Plataformas para este servicio.
  options.put('PlatformEventEnabled', config.Platform_Events_Enabled__c);

  ExternalNotificationHandler enh = new ExternalNotificationHandler();
  Map<String, Object> payload = new Map<String, Object>();

  try {
    payload = this.preProcess(input, options);
    Core.debug('Starting RUNT Integration Service In Process');
    publishPlatformEventIntegration('in', JSON.serialize(input));
    //Validations
    // ExternalServiceValidationUtil.plateValidation(
    //   (String) input.get('plate')
    // );

    System.HTTPResponse response = enh.emit(
      SERVICE_METHOD,
      config,
      String.valueOf(input.get('plate')),
      payload
    );

    Map<String, Object> responseMap = new Map<String, Object>{
      'body' => (Map<String, Object>) JSON.deserializeUntyped(
        response.getBody()
      ),
      'status' => response.getStatus(),
      'statusCode' => response.getStatusCode()
    };

    this.publishPlatFormEvent(
      response.getStatusCode(),
      JSON.serialize(payload),
      JSON.serialize(response.getBody())
    );
    return this.postProcess(responseMap, options);
  } catch (Exception e) {
    enh.controlException(
      'RUNT Service remote service',
      e,
      JSON.serialize(payload)
    );
    return null;
  }
}
```

### `postProcess(Map<String, Object> response, Map<String, Object> options)`

#### Descripción
Método virtual que realiza el post-procesamiento de la respuesta después de invocar el servicio externo. Publica un evento de plataforma si está habilitado.

#### Parámetros
- `response` (Map<String, Object>): Datos de respuesta del servicio externo.
- `options` (Map<String, Object>): Opciones adicionales para el procesamiento.

#### Retorno
- `Map<String, Object>`: La respuesta, posiblemente modificada durante el post-procesamiento.

#### Código
```apex
global virtual Map<String, Object> postProcess(
  Map<String, Object> response,
  Map<String, Object> options
) {
  Core.debug('Starting RUNT Integration Service Post Process');
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
Publica diferentes tipos de eventos de plataforma según el momento del proceso de integración.

#### Parámetros
- `moment` (String): Momento del proceso ('pre', 'post', 'in').
- `data` (String): Datos serializados para incluir en el evento.

#### Proceso
Utiliza una declaración switch para determinar qué tipo de evento de plataforma publicar:
- 'pre': Evento de pre-procesamiento.
- 'post': Evento de post-procesamiento.
- 'in': Evento durante el procesamiento.

#### Código
```apex
global void publishPlatformEventIntegration(String moment, String data) {
  switch on moment.toLowerCase() {
    when 'pre' {
      publishInitPrePlatformEvent(data, 'RUNT Integration Service');
    }
    when 'post' {
      publishInitPosPlatformEvent(data, 'RUNT Integration Service');
    }
    when 'in' {
      publishPlatFormEvent(200, data, 'Starting RUNT In process');
    }
  }
}
```

## Excepciones

### `ExternalServiceException`
Esta excepción se lanza cuando no se encuentra la configuración del servicio RUNT en los metadatos personalizados.

## Dependencias
- `PlatformEventIntegrationObserver`: Clase base que proporciona funcionalidades para publicar eventos de plataforma.
- `SF_IntegrationSetting__mdt`: Tipo de metadatos personalizado que contiene la configuración para la integración.
- `ExternalServiceMetadataIntegrationMock`: Clase mock utilizada durante las pruebas para simular la configuración de integración.
- `ExternalNotificationHandler`: Clase que maneja la emisión de solicitudes a servicios externos y el control de excepciones.
- `Core`: Clase de utilidad que proporciona el método `debug` para registro.
- `ExternalServiceException`: Excepción personalizada lanzada cuando hay problemas con la configuración del servicio.
- `ExternalServiceValidationUtil`: Clase de utilidad para validar datos (actualmente comentada).

## Características Notables

### Eventos de Plataforma
La clase implementa una integración completa con eventos de plataforma de Salesforce, permitiendo el seguimiento y monitoreo del proceso de integración en diferentes etapas:
- Pre-procesamiento
- Durante el procesamiento
- Post-procesamiento

### Manejo de Configuración
La configuración del servicio se obtiene dinámicamente de un registro de metadatos personalizado (`SF_IntegrationSetting__mdt`), facilitando cambios en la configuración sin modificar el código.

### Manejo de Pruebas
Incorpora lógica especial para entornos de prueba, utilizando una clase mock (`ExternalServiceMetadataIntegrationMock`) para simular la configuración durante las pruebas.

### Virtualidad
Los métodos principales (`preProcess` y `postProcess`) son declarados como virtuales, permitiendo que las clases derivadas personalicen el comportamiento sin modificar el flujo general.

## Ejemplo de Uso

```apex
// Crear una instancia del handler
RUNTServiceHandler handler = new RUNTServiceHandler();

// Preparar los datos de entrada
Map<String, Object> input = new Map<String, Object>{
  'plate' => 'ABC123'  // Placa del vehículo a consultar
};

// Mapas para opciones y salida
Map<String, Object> options = new Map<String, Object>();
Map<String, Object> output = new Map<String, Object>();

try {
  // Invocar el proceso de integración
  Map<String, Object> result = handler.invokeProcess(input, output, options);
  
  if (result != null) {
    // Procesar el resultado
    Map<String, Object> body = (Map<String, Object>)result.get('body');
    Integer statusCode = (Integer)result.get('statusCode');
    
    if (statusCode == 200) {
      // Extraer información del vehículo del resultado
      System.debug('Información del vehículo: ' + body);
      
      // Realizar operaciones con la información obtenida
      // ...
    } else {
      System.debug('Error en la consulta. Código: ' + statusCode);
    }
  }
} catch (Exception e) {
  System.debug('Error en la integración: ' + e.getMessage());
}
```

## Extensión de la Clase

```apex
public class CustomRUNTServiceHandler extends RUNTServiceHandler {
    
    // Personalizar el pre-procesamiento
    global override Map<String, Object> preProcess(
        Map<String, Object> input,
        Map<String, Object> options
    ) {
        // Realizar validaciones adicionales
        String plate = (String)input.get('plate');
        if (plate != null) {
            // Convertir la placa a mayúsculas
            input.put('plate', plate.toUpperCase());
            
            // Validar el formato de la placa
            ExternalServiceValidationUtil.plateValidation(plate);
        }
        
        // Agregar información adicional
        input.put('requestTimestamp', System.now().getTime());
        input.put('requestUser', UserInfo.getUserId());
        
        // Llamar al método base
        return super.preProcess(input, options);
    }
    
    // Personalizar el post-procesamiento
    global override Map<String, Object> postProcess(
        Map<String, Object> response,
        Map<String, Object> options
    ) {
        // Transformar o enriquecer la respuesta
        if (response != null && response.containsKey('body')) {
            Map<String, Object> body = (Map<String, Object>)response.get('body');
            
            // Agregar información adicional
            body.put('processedTimestamp', System.now().getTime());
            
            // Transformar datos específicos si es necesario
            // ...
        }
        
        // Llamar al método base
        return super.postProcess(response, options);
    }
}
```

## Notas Adicionales
1. La clase contiene código comentado que sugiere una implementación anterior o alternativa, incluyendo constantes para el nombre del procedimiento de integración y el nombre del servicio, así como una validación de placa de vehículo.
2. La clase está diseñada siguiendo un patrón de tres fases (pre-proceso, proceso, post-proceso), permitiendo personalizar cada fase a través de la herencia.
3. La integración con eventos de plataforma facilita el monitoreo y depuración del proceso de integración.
4. La clase es consciente del contexto de prueba (`Test.isRunningTest()`) y utiliza mocks para simular la configuración durante las pruebas.
5. La clase no realiza actualmente la validación de la placa del vehículo, aunque hay código comentado que sugiere que esta validación podría ser necesaria en algún momento.