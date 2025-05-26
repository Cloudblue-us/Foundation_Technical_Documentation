# Documentación Técnica: ExternalNotificationHandler

## Descripción General
`ExternalNotificationHandler` es una clase virtual que extiende `PlatformEventIntegrationObserver` y está diseñada para manejar notificaciones a servicios externos. Esta clase implementa un mecanismo sofisticado de llamadas HTTP a servicios externos con capacidades de reintento, control de tiempos de espera, y registro de eventos de plataforma para monitoreo y depuración.

## Autor
Soulberto Lorenzo \<soulberto@cloudblue.us\>

## Última Modificación
9 de mayo de 2025 por Esneyder Zabala

## Constantes y Variables Estáticas

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `NAMED_CREDENTIAL` | `String` | Nombre de la credencial con nombre utilizada para autenticación: 'Suratech_Integration_Bus_Credentials' |
| `skipLogDefault` | `Boolean` | Controla si se deben registrar logs de eventos. Valor por defecto: `false` |
| `retryQuantity` | `Integer` | Número de reintentos realizados. Valor inicial: `1` |
| `familyProductControl` | `String` | Familia de producto para el control de logs. Valor por defecto: 'All' |
| `accumulatedRetryTime` | `Long` | Tiempo acumulado en milisegundos para los reintentos |
| `MAX_RETRY_TIME` | `Long` | Tiempo máximo permitido para los reintentos, obtenido desde un metadata personalizado |
| `bodyInput` | `String` | Almacena el cuerpo de la solicitud para su uso en logs |

## Variables de Instancia

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `startTimeGlobal` | `Long` | Tiempo de inicio global para medir el tiempo total de ejecución |

## Inicialización Estática

La clase inicializa `MAX_RETRY_TIME` a través de un bloque estático que obtiene el valor desde un registro de metadatos personalizado `SF_ConfigTimeRetryIntegrations__mdt`.

```apex
static {
  MAX_RETRY_TIME = getMaxRetryTimeFromMetadata();
}
```

## Métodos de Acceso (Getters y Setters)

| Método | Descripción |
|--------|-------------|
| `setBodyInput(String value)` | Establece el valor del cuerpo de la solicitud para su uso en logs |
| `getBodyInput()` | Obtiene el valor del cuerpo de la solicitud |
| `setFamilyProduct(String value)` | Establece la familia de producto para control de logs |
| `getFamilyProduct()` | Obtiene la familia de producto actual |
| `setretryQuantity(Integer value)` | Establece el número de reintentos (método estático) |
| `getretryQuantity()` | Obtiene el número actual de reintentos (método estático) |
| `setskipLogDefault(Boolean value)` | Establece si se deben omitir los logs por defecto |
| `getskipLogDefault()` | Obtiene el valor actual de omisión de logs |

## Métodos Principales

### `emit(String methodType, SF_IntegrationSetting__mdt config, String serviceParameters, Object payload, String family)`

#### Descripción
Emite una solicitud HTTP a un servicio externo utilizando la configuración proporcionada.

#### Parámetros
- `methodType` (String): Tipo de método HTTP (GET, POST, etc.).
- `config` (SF_IntegrationSetting__mdt): Configuración de integración desde metadatos personalizados.
- `serviceParameters` (String): Parámetros adicionales para la URL del servicio.
- `payload` (Object): Carga útil para enviar en la solicitud (para método POST).
- `family` (String): Familia de producto para los logs.

#### Retorno
- `System.HTTPResponse`: Respuesta HTTP del servicio externo.

#### Proceso
1. Almacena la carga útil para el registro.
2. Construye la URL del servicio combinando la ruta base con los parámetros.
3. Crea y configura una solicitud HTTP con credenciales nombradas.
4. Establece encabezados y cuerpo para solicitudes POST.
5. Configura el tiempo de espera según la configuración.
6. Crea un wrapper con toda la información necesaria para gestionar el envío.
7. Llama al método `send` para ejecutar la solicitud con lógica de reintento.

#### Código
```apex
public System.HTTPResponse emit(
  String methodType,
  SF_IntegrationSetting__mdt config,
  String serviceParameters,
  Object payload,
  String family
) {
  setBodyInput(String.valueOf(payload));
  String service = (config.Path__c ?? '') + serviceParameters;
  
  Core.debug('Using Named Credentials="' + NAMED_CREDENTIAL + '"');
  HttpRequest req = new HttpRequest();
  req.setEndpoint('callout:' + NAMED_CREDENTIAL + service);
  req.setMethod(methodType);
  if (methodType == 'POST') {
    Core.debug(
      'Emitting ' +
        config.MasterLabel +
        ' to external service="' +
        service +
        '" with payload=' +
        JSON.serialize(payload)
    );
    req.setHeader('Content-Type', 'application/json');
    req.setBody(JSON.serialize(payload));
    req.setTimeout(Integer.valueOf(config.TimeOutLimit__c) ?? 120000);
  }

  SendMethodResourceWrapper sendWrapper = new SendMethodResourceWrapper(
    req,
    (config.RetryOnResponseCode__c ?? '').split(';'),
    config.TimeoutType__c,
    Integer.valueOf(config.RetryQuantity__c)
  );
  String familyProduct = family != null ? family : '';
  System.HttpResponse res = send(
    sendWrapper,
    String.valueOf(config.MasterLabel),
    familyProduct
  );

  return res;
}
```

### `send(SendMethodResourceWrapper resourceWrapper, String process, String family)`

#### Descripción
Método estático que realiza la solicitud HTTP con lógica de reintento basada en la configuración proporcionada.

#### Parámetros
- `resourceWrapper` (SendMethodResourceWrapper): Wrapper que contiene la solicitud y la configuración de reintentos.
- `process` (String): Nombre del proceso para incluir en los logs.
- `family` (String): Familia de producto para los logs.

#### Retorno
- `System.HTTPResponse`: Respuesta HTTP del servicio externo.

#### Proceso
1. Realiza la solicitud HTTP y mide el tiempo de ejecución.
2. Acumula el tiempo de reintento para control de límites.
3. Verifica si el tiempo de CPU consumido excede el límite máximo.
4. Determina si se debe realizar un reintento basado en el código de respuesta y el número de reintentos restantes.
5. Registra eventos de éxito o error según corresponda.
6. Si es necesario un reintento, aplica un tiempo de espera según la estrategia configurada y llama recursivamente al método.

#### Código
```apex
public static System.HttpResponse send(
  SendMethodResourceWrapper resourceWrapper,
  String process,
  String family
) {
  Http http = new Http();
  Long startTime = System.currentTimeMillis();
  Core.debug('Waiting response...');
  System.HttpResponse res = http.send(resourceWrapper.request);

  Long endTime = System.currentTimeMillis();
  Long retryTime = endTime - startTime;
  accumulatedRetryTime += retryTime;

  Integer activeTime = Limits.getCpuTime();
  if (activeTime > MAX_RETRY_TIME) {
    ExternalServiceValidationUtil.publishServiceEventLog(
      'Error',
      'Error trying connect to remote ' +
        resourceWrapper.request.getEndpoint() +
        ' failed with response= tiempo de cpu consumido supera el limite' +
        ' reintento numero = ' +
        getretryQuantity() +
        ' | Tiempo acumulado de CPU= ' +
        activeTime,
      process,
      family,
      getBodyInput(),
      JSON.serialize(res.getBody())
    );
    return res;
  }

  if (
    resourceWrapper.retriesLeft == 0 ||
    !resourceWrapper.retriesCodes.contains(
      String.valueOf(res.getStatusCode())
    )
  ) {
    if (!skipLogDefault) {
      switch on res.getStatusCode() {
        when 200 {
          ExternalServiceValidationUtil.publishServiceEventLog(
            'Success',
            'Success connecting to remote ' +
              resourceWrapper.request.getEndpoint() +
              ' reintento numero = ' +
              getretryQuantity() +
              ' | Tiempo acumulado = ' +
              accumulatedRetryTime +
              ' milisegundos',
            process,
            family,
            getBodyInput(),
            JSON.serialize(res.getBody())
          );
        }
        when else {
          ExternalServiceValidationUtil.publishServiceEventLog(
            'Error',
            'Error trying connect to remote ' +
              resourceWrapper.request.getEndpoint() +
              ' reintento numero = ' +
              getretryQuantity() +
              ' | Tiempo acumulado = ' +
              accumulatedRetryTime +
              ' milisegundos',
            process,
            family,
            getBodyInput(),
            JSON.serialize(res.getBody())
          );
        }
      }
    }
    return res;
  }

  Core.debug('Retry Code found, retrying...');
  applyTimeOut(
    resourceWrapper.timeOutType,
    resourceWrapper.retriesCurrentNumber
  );
  resourceWrapper.substractRetryLeft();
  resourceWrapper.addRetry();
  return send(resourceWrapper, process, family);
}
```

### `controlException(String integration, Exception e, String payload)`

#### Descripción
Maneja excepciones relacionadas con llamadas a servicios externos, registrando eventos de plataforma y relanzando excepciones apropiadas.

#### Parámetros
- `integration` (String): Nombre de la integración donde ocurrió la excepción.
- `e` (Exception): La excepción capturada.
- `payload` (String): Carga útil que se estaba enviando cuando ocurrió la excepción.

#### Proceso
1. Determina el tipo de excepción (CalloutException, ExternalServiceValidationException, u otra).
2. Publica un evento de plataforma con código 400 o 500 según el tipo de excepción.
3. Lanza una ExternalServiceException o relanza la excepción original según corresponda.

#### Código
```apex
public void controlException(
  String integration,
  Exception e,
  String payload
) {
  if (
    e instanceof CalloutException ||
    e instanceof ExternalServiceValidationException
  ) {
    if (
      e.getMessage().contains('400') ||
      e instanceof ExternalServiceValidationException
    )
      this.publishPlatformEvent(
        400,
        JSON.serialize(payload),
        'A  has ocurred during ' +
          integration +
          ' Execution: ' +
          e.getMessage()
      );
    else
      this.publishPlatformEvent(
        500,
        JSON.serialize(payload),
        'A callout Exception has ocurred during ' +
          integration +
          ' Execution: ' +
          e.getMessage()
      );
    throw new ExternalServiceException(
      integration + ' has failed. Exception: ' + e.getMessage()
    );
  } else {
    this.publishPlatformEvent(
      500,
      JSON.serialize(payload),
      'An Exception has ocurred during ' +
        integration +
        ' Execution: ' +
        e.getMessage()
    );
    throw e;
  }
}
```

### `controlExceptionSavingLog(String integration, String messaje, String process)`

#### Descripción
Registra información de excepciones en logs de eventos si el registro está habilitado.

#### Parámetros
- `integration` (String): Nombre de la integración donde ocurrió la excepción.
- `messaje` (String): Mensaje de error.
- `process` (String): Nombre del proceso para incluir en los logs.

#### Proceso
1. Verifica si el registro de logs está habilitado.
2. Calcula el tiempo transcurrido desde el inicio global.
3. Formatea el mensaje según el tipo de error.
4. Publica un evento de log con la información recopilada.

#### Código
```apex
public void controlExceptionSavingLog(
  String integration,
  String messaje,
  String process
) {
  if (!skipLogDefault) {
    System.debug('Generating eventlog');
    Long endTime = System.currentTimeMillis();
    Long retryTime = endTime - startTimeGlobal;

    String setMensaje = messaje.equals('Read timed out')
      ? 'failed with response= tiempo de espera del callout a superado el limite' +
        ' reintento numero = ' +
        getretryQuantity() +
        ' | Tiempo acumulado de cpu= ' +
        Limits.getCpuTime() +
        ' y tiempo acumulado de logica: ' +
        retryTime
      : 'Error trying connect to ' + integration + '.... ' + messaje;
    ExternalServiceValidationUtil.publishServiceEventLog(
      'ERROR',
      setMensaje,
      process,
      familyProductControl,
      getBodyInput(),
      ''
    );
  }
}
```

## Métodos de Gestión de Tiempo de Espera

### `applyTimeOut(String timeOutType, Integer retriesCurrentNumber)`

#### Descripción
Aplica una estrategia de tiempo de espera según el tipo configurado.

#### Parámetros
- `timeOutType` (String): Tipo de estrategia de tiempo de espera ('FIXED', 'INCREMENTAL', 'FIBONACCI').
- `retriesCurrentNumber` (Integer): Número actual de reintentos.

#### Proceso
Aplica la estrategia de tiempo de espera correspondiente según el tipo especificado.

#### Código
```apex
private static void applyTimeOut(
  String timeOutType,
  Integer retriesCurrentNumber
) {
  Integer miliseconds = 100;
  switch on timeOutType.toUpperCase() {
    when 'FIXED' {
      fixedTimeOut(miliseconds);
    }
    when 'INCREMENTAL' {
      incrementalTimeOut(retriesCurrentNumber, miliseconds);
    }
    when 'FIBONACCI' {
      waitMethod(miliseconds * fibonacciTimeOut(retriesCurrentNumber));
    }
  }
}
```

### Métodos de Estrategia de Tiempo de Espera

- **`fixedTimeOut(Integer miliseconds)`**: Aplica un tiempo de espera fijo.
- **`incrementalTimeOut(Integer numberOfRetry, Integer miliseconds)`**: Aplica un tiempo de espera que incrementa linealmente con el número de reintentos.
- **`fibonacciTimeOut(Integer numberOfRetry)`**: Calcula el término de Fibonacci correspondiente al número de reintento.
- **`waitMethod(Integer miliseconds)`**: Implementa la espera activa durante el número de milisegundos especificado.

## Clase Interna

### `SendMethodResourceWrapper`

#### Descripción
Clase interna privada que encapsula la información necesaria para gestionar solicitudes HTTP con lógica de reintento.

#### Atributos
- `request` (HttpRequest): La solicitud HTTP a enviar.
- `retriesCodes` (List<String>): Lista de códigos de respuesta que deberían desencadenar un reintento.
- `timeOutType` (String): Tipo de estrategia de tiempo de espera.
- `retriesCurrentNumber` (Integer): Número actual de reintento.
- `retriesLeft` (Integer): Número de reintentos restantes.

#### Métodos
- **Constructor**: Inicializa los atributos con los valores proporcionados.
- **`substractRetryLeft()`**: Decrementa el contador de reintentos restantes.
- **`addRetry()`**: Incrementa el contador de reintentos actuales y actualiza el contador global.

## Dependencias
- `PlatformEventIntegrationObserver`: Clase base que proporciona métodos para publicar eventos de plataforma.
- `SF_IntegrationSetting__mdt`: Tipo de metadatos personalizado que contiene la configuración para la integración.
- `SF_ConfigTimeRetryIntegrations__mdt`: Tipo de metadatos personalizado que contiene la configuración de tiempo máximo de reintento.
- `Core`: Clase de utilidad que proporciona el método `debug` para registro.
- `ExternalServiceValidationUtil`: Clase que proporciona métodos para validar y registrar eventos de servicio.
- `ExternalServiceException`: Excepción personalizada lanzada cuando hay problemas con el servicio externo.
- `ExternalServiceValidationException`: Excepción personalizada para errores de validación.

## Estrategias de Reintento

La clase implementa tres estrategias diferentes de tiempo de espera para reintentos:

### 1. FIXED
Espera un tiempo fijo (100 ms) entre cada reintento.

### 2. INCREMENTAL
Espera un tiempo que aumenta linealmente con el número de reintentos (n * 100 ms).

### 3. FIBONACCI
Espera un tiempo basado en la secuencia de Fibonacci, multiplicado por un factor base (100 ms).

## Ejemplo de Uso

```apex
try {
    // Obtener la configuración de integración
    SF_IntegrationSetting__mdt config = SF_IntegrationSetting__mdt.getInstance('mi_servicio');
    
    // Crear el manejador de notificaciones
    ExternalNotificationHandler handler = new ExternalNotificationHandler();
    
    // Establecer configuraciones específicas (opcional)
    handler.setFamilyProduct('MiProducto');
    handler.setskipLogDefault(false);
    
    // Preparar la carga útil
    Map<String, Object> payload = new Map<String, Object>{
        'id' => '12345',
        'name' => 'Test Request',
        'date' => System.now().format()
    };
    
    // Emitir la solicitud
    System.HTTPResponse response = handler.emit(
        'POST',
        config,
        '/resource/endpoint',
        payload,
        'MiProducto'
    );
    
    // Procesar la respuesta
    if (response.getStatusCode() == 200) {
        Map<String, Object> responseBody = (Map<String, Object>)JSON.deserializeUntyped(response.getBody());
        System.debug('Respuesta exitosa: ' + responseBody);
    } else {
        System.debug('Error en la respuesta: ' + response.getStatus());
    }
} catch (Exception e) {
    // Manejar la excepción
    System.debug('Error: ' + e.getMessage());
}
```

## Notas Adicionales
1. La clase utiliza un mecanismo sofisticado de reintentos con diferentes estrategias de tiempo de espera.
2. Implementa un control de tiempo máximo para evitar que los reintentos consuman demasiados recursos.
3. Registra detalladamente los eventos de integración para facilitar la depuración y el monitoreo.
4. Está diseñada para ser extendida, permitiendo personalizar comportamientos específicos a través de herencia.
5. Utiliza metadatos personalizados para configurar aspectos como tiempos de espera, estrategias de reintento y códigos de respuesta que deben desencadenar reintentos.
6. Implementa un método de espera activa (`waitMethod`) que podría consumir significativamente recursos de CPU.
7. La clase contiene código comentado que podría representar versiones anteriores o alternativas de implementación.