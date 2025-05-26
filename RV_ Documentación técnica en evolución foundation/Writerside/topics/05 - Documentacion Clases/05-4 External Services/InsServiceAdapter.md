# InsServiceAdapter

## Descripción General
`InsServiceAdapter` es una clase que implementa la interfaz `Adapter` y proporciona funcionalidad para interactuar con servicios de Vlocity Insurance. Esta clase actúa como un adaptador que facilita la comunicación entre el sistema Salesforce y los servicios de seguros externos, específicamente para acceder a productos calificados (rated products).

## Autor
Soulberto Lorenzo \<soulberto@cloudblue.us\>

## Última Modificación
2 de septiembre de 2024

## Decoradores y Accesibilidad

La clase está marcada con `@namespaceAccessible`, lo que indica que está diseñada para ser accesible desde diferentes espacios de nombres (namespaces). Esto es especialmente relevante en contextos de paquetes gestionados donde se necesita que componentes específicos sean accesibles desde fuera del namespace del paquete.

## Estructura de la Clase

```apex
@namespaceAccessible
public class InsServiceAdapter implements Adapter {
  @namespaceAccessible
  public Map<String, Object> adaptee() {
    return new Map<String, Object>();
  }

  @namespaceAccessible
  public static Boolean useInsAdapter() {
    // Implementación del método
  }
}
```

## Interfaz Implementada

### `Adapter`
La clase implementa la interfaz `Adapter`. Aunque no se muestra la definición de esta interfaz en el código proporcionado, se puede inferir que requiere al menos un método `adaptee()` que devuelve un `Map<String, Object>`.

## Métodos

### `adaptee()`

#### Descripción
Método que implementa la interfaz `Adapter` y devuelve un mapa vacío. Según el comentario comentado, estaba previsto que devolviera una instancia de `URLGenerationNotificationHandler`, pero actualmente devuelve un mapa vacío.

#### Parámetros
Ninguno.

#### Retorno
- `Map<String, Object>`: Un mapa vacío.

#### Código
```apex
@namespaceAccessible
public Map<String, Object> adaptee() {
  // return new URLGenerationNotificationHandler();
  return new Map<String, Object>();
}
```

### `useInsAdapter()`

#### Descripción
Método estático que intenta realizar una llamada HTTP a un servicio de Vlocity Insurance para obtener productos calificados. Determina si el adaptador de seguros debe ser utilizado basándose en el éxito de esta llamada.

#### Parámetros
Ninguno.

#### Retorno
- `Boolean`: `true` si la llamada al servicio fue exitosa (estado 200), `false` en caso contrario.

#### Proceso
1. Inicializa una variable booleana `isSuccess` como `false`.
2. Crea una nueva solicitud HTTP.
3. Consulta el nombre de la instancia de la organización actual.
4. Configura el endpoint de la solicitud usando una credencial nombrada para el servicio de integración de Vlocity.
5. Establece el método HTTP como 'GET'.
6. Crea una instancia de Http y envía la solicitud.
7. Analiza la respuesta:
    - Si el código de estado es 200, registra el cuerpo de la respuesta y establece `isSuccess` como `true`.
    - Si el código de estado no es 200, registra el estado de error y mantiene `isSuccess` como `false`.
8. Devuelve el valor de `isSuccess`.

#### Código
```apex
@namespaceAccessible
public static Boolean useInsAdapter() {
  Boolean isSuccess = false;
  HttpRequest insServiceRequest = new HttpRequest();

  String instanceName = [SELECT InstanceName FROM Organization LIMIT 1]
  .InstanceName;
  System.debug(instanceName);

  // insServiceRequest.setEndpoint(
  //   'callout:salesforce_ins_service_adaptar/services/apexrest/vlocity_insurance/ratedproducts'
  // );
  insServiceRequest.setEndpoint(
    'callout:Suratech_Salesforce_Vlocity_Integration_Credential/services/apexrest/vlocity_insurance/ratedproducts'
  );
  insServiceRequest.setMethod('GET');

  Http http = new Http();
  HttpResponse res = http.send(insServiceRequest);

  if (res.getStatusCode() == 200) {
    System.debug('Response: ' + res.getBody());
    isSuccess = true;
  } else {
    System.debug('Error: ' + res.getStatus());
    isSuccess = false;
  }

  return isSuccess;
}
```

## Observaciones sobre el Código

### Puntos a Destacar
1. **Implementación Parcial o Transitoria**: El método `adaptee()` parece estar en un estado transitorio, ya que contiene código comentado y devuelve un mapa vacío.
2. **Uso de Credencial Nombrada**: La clase utiliza una credencial nombrada (`Suratech_Salesforce_Vlocity_Integration_Credential`) para la autenticación segura con el servicio externo.
3. **Consulta de Organización**: La clase consulta la información de la organización actual, aunque actualmente solo utiliza esta información para registro de depuración.

### Posibles Mejoras
1. **Implementación Completa de `adaptee()`**: Completar la implementación del método `adaptee()` para que devuelva la instancia apropiada o un mapa con datos relevantes.
2. **Manejo de Excepciones**: Añadir manejo de excepciones para capturar y gestionar adecuadamente posibles errores durante la llamada HTTP.
3. **Parametrización**: Considerar hacer que el endpoint sea configurable, posiblemente a través de metadatos personalizados, en lugar de codificarlo directamente.
4. **Timeout y Reintentos**: Implementar lógica de timeout y reintentos para manejar situaciones donde el servicio externo no responde o lo hace lentamente.
5. **Utilización de la Información de Instancia**: Hacer un uso más significativo de la información de instancia que se está consultando.

## Versión Mejorada Propuesta

```apex
/**
 * @description       : Adaptador para servicios de seguros de Vlocity
 * @author            : Soulberto Lorenzo <soulberto@cloudblue.us>
 * @group             : Adapters
 * @last modified on  : 09-02-2024
 * @last modified by  : Soulberto Lorenzo <soulberto@cloudblue.us>
 **/
@namespaceAccessible
public class InsServiceAdapter implements Adapter {
  // Constantes para configuración
  private static final String VLOCITY_ENDPOINT_PATH = '/services/apexrest/vlocity_insurance/ratedproducts';
  private static final String NAMED_CREDENTIAL = 'Suratech_Salesforce_Vlocity_Integration_Credential';
  private static final Integer DEFAULT_TIMEOUT = 30000; // 30 segundos
  
  /**
   * @description Implementa la interfaz Adapter devolviendo un adaptador para servicios de seguros
   * @author Soulberto Lorenzo <soulberto@cloudblue.us> | 09-02-2024
   * @return Map<String, Object> Mapa con información del adaptador
   **/
  @namespaceAccessible
  public Map<String, Object> adaptee() {
    Map<String, Object> adapterInfo = new Map<String, Object>();
    
    try {
      // Verificar disponibilidad del servicio
      Boolean serviceAvailable = checkServiceAvailability();
      
      // Añadir información relevante al mapa
      adapterInfo.put('type', 'InsuranceServiceAdapter');
      adapterInfo.put('serviceAvailable', serviceAvailable);
      adapterInfo.put('instanceName', getOrganizationInstanceName());
      
      // Si el servicio está disponible, añadir información adicional
      if (serviceAvailable) {
        // Aquí se podría añadir más información específica del servicio
        // Por ejemplo, versión de API, capacidades disponibles, etc.
      }
    } catch (Exception e) {
      // Registrar error y añadir información de error al mapa
      System.debug(LoggingLevel.ERROR, 'Error en adaptee(): ' + e.getMessage());
      adapterInfo.put('error', e.getMessage());
      adapterInfo.put('errorType', e.getTypeName());
    }
    
    return adapterInfo;
  }

  /**
   * @description Verifica si se debe utilizar el adaptador de seguros basándose en la disponibilidad del servicio
   * @author Soulberto Lorenzo <soulberto@cloudblue.us> | 09-02-2024
   * @return Boolean true si el servicio está disponible, false en caso contrario
   **/
  @namespaceAccessible
  public static Boolean useInsAdapter() {
    return checkServiceAvailability();
  }
  
  /**
   * @description Verifica la disponibilidad del servicio de seguros
   * @return Boolean true si el servicio está disponible, false en caso contrario
   **/
  private static Boolean checkServiceAvailability() {
    try {
      HttpRequest insServiceRequest = new HttpRequest();
      
      String endpoint = 'callout:' + NAMED_CREDENTIAL + VLOCITY_ENDPOINT_PATH;
      System.debug('Verificando disponibilidad del servicio: ' + endpoint);
      
      insServiceRequest.setEndpoint(endpoint);
      insServiceRequest.setMethod('GET');
      insServiceRequest.setTimeout(DEFAULT_TIMEOUT);
      
      Http http = new Http();
      HttpResponse res = http.send(insServiceRequest);
      
      Integer statusCode = res.getStatusCode();
      System.debug('Código de estado de respuesta: ' + statusCode);
      
      if (statusCode == 200) {
        System.debug('Servicio disponible. Respuesta: ' + res.getBody());
        return true;
      } else {
        System.debug('Servicio no disponible. Estado: ' + res.getStatus());
        logServiceError(statusCode, res.getStatus(), res.getBody());
        return false;
      }
    } catch (Exception e) {
      System.debug(LoggingLevel.ERROR, 'Error al verificar disponibilidad del servicio: ' + e.getMessage());
      return false;
    }
  }
  
  /**
   * @description Obtiene el nombre de la instancia de la organización actual
   * @return String Nombre de la instancia
   **/
  private static String getOrganizationInstanceName() {
    try {
      Organization org = [SELECT InstanceName FROM Organization LIMIT 1];
      return org.InstanceName;
    } catch (Exception e) {
      System.debug(LoggingLevel.ERROR, 'Error al obtener el nombre de la instancia: ' + e.getMessage());
      return '';
    }
  }
  
  /**
   * @description Registra información detallada sobre errores del servicio
   * @param statusCode Código de estado HTTP
   * @param status Descripción del estado
   * @param responseBody Cuerpo de la respuesta
   **/
  private static void logServiceError(Integer statusCode, String status, String responseBody) {
    // Aquí se podría implementar lógica para registrar errores en un objeto personalizado,
    // enviar notificaciones, o realizar otras acciones según la política de gestión de errores
    
    System.debug(LoggingLevel.ERROR, 'Error en servicio de seguros - Código: ' + statusCode + 
                 ', Estado: ' + status + ', Respuesta: ' + responseBody);
  }
  
  /**
   * @description Realiza una llamada al servicio de seguros para obtener productos calificados
   * @param parameters Parámetros de la solicitud
   * @return Map<String, Object> Respuesta del servicio
   * @throws InsServiceException Si hay un error en la llamada al servicio
   **/
  @namespaceAccessible
  public static Map<String, Object> getRatedProducts(Map<String, Object> parameters) {
    try {
      // Verificar disponibilidad del servicio
      if (!checkServiceAvailability()) {
        throw new InsServiceException('El servicio de seguros no está disponible');
      }
      
      // Implementar lógica para obtener productos calificados
      // ...
      
      return new Map<String, Object>(); // Placeholder
    } catch (Exception e) {
      throw new InsServiceException('Error al obtener productos calificados: ' + e.getMessage(), e);
    }
  }
  
  /**
   * Clase de excepción personalizada para errores del servicio de seguros
   */
  public class InsServiceException extends Exception {}
}
```

## Ejemplo de Uso

### Uso Básico para Verificar Disponibilidad

```apex
// Verificar si el adaptador de seguros debe ser utilizado
Boolean useInsAdapter = InsServiceAdapter.useInsAdapter();

if (useInsAdapter) {
    System.debug('El servicio de seguros está disponible, se utilizará el adaptador');
    // Lógica adicional cuando el servicio está disponible
} else {
    System.debug('El servicio de seguros no está disponible, se utilizará una alternativa');
    // Lógica alternativa cuando el servicio no está disponible
}
```

### Uso del Adaptador

```apex
// Crear una instancia del adaptador
InsServiceAdapter adapter = new InsServiceAdapter();

// Obtener información del adaptador
Map<String, Object> adapterInfo = adapter.adaptee();

// Verificar si el servicio está disponible
Boolean serviceAvailable = (Boolean)adapterInfo.get('serviceAvailable');

if (serviceAvailable) {
    // Utilizar el adaptador para obtener productos calificados
    Map<String, Object> parameters = new Map<String, Object>{
        'productType' => 'Auto',
        'clientId' => '12345'
    };
    
    try {
        Map<String, Object> ratedProducts = InsServiceAdapter.getRatedProducts(parameters);
        // Procesar los productos calificados
        // ...
    } catch (InsServiceAdapter.InsServiceException e) {
        System.debug('Error al obtener productos calificados: ' + e.getMessage());
        // Manejar el error
    }
} else {
    System.debug('El servicio no está disponible: ' + adapterInfo.get('error'));
    // Manejar la falta de disponibilidad del servicio
}
```

## Contexto y Uso Típico

### Vlocity Insurance y Salesforce

Esta clase probablemente forma parte de una integración entre Salesforce y Vlocity, una solución específica para la industria de seguros que ahora es parte de Salesforce Industries. Vlocity proporciona componentes y funcionalidad específica para varias industrias, incluyendo seguros.

### Casos de Uso Típicos

1. **Cotización de Seguros**: Obtener productos calificados con precios y coberturas específicas para un cliente.
2. **Configuración de Productos**: Acceder a reglas de configuración y opciones para productos de seguros.
3. **Verificación de Elegibilidad**: Determinar si un cliente es elegible para ciertos productos o coberturas.
4. **Cálculo de Primas**: Obtener cálculos de primas basados en información específica del cliente.

## Patrones de Diseño Utilizados

### Patrón Adaptador (Adapter)
La clase implementa el patrón Adaptador, que permite que interfaces incompatibles trabajen juntas. En este caso, adapta la comunicación con el servicio de Vlocity Insurance a una interfaz común definida por `Adapter`.

### Patrón de Comprobación de Disponibilidad (Availability Check)
El método `useInsAdapter()` implementa un patrón de comprobación de disponibilidad, que verifica si un servicio externo está disponible antes de intentar utilizarlo para operaciones críticas.

## Consideraciones de Seguridad

1. **Credenciales Nombradas**: El uso de credenciales nombradas es una buena práctica para manejar la autenticación, ya que abstrae las credenciales del código.
2. **Registro de Datos Sensibles**: El código actual registra el cuerpo completo de la respuesta, lo que podría contener información sensible. Considerar filtrar o anonimizar datos sensibles en los registros.

## Consideraciones de Rendimiento

1. **Timeout**: El código original no establece un timeout específico para la llamada HTTP, lo que podría resultar en esperas prolongadas si el servicio externo no responde.
2. **Consulta de Organización**: La consulta a la tabla Organization se realiza en cada llamada al método `useInsAdapter()`. Para uso frecuente, considerar cachear esta información.

## Notas Adicionales
1. La clase parece estar en desarrollo o en transición, como lo indican los comentarios y la implementación incompleta de `adaptee()`.
2. La anotación `@namespaceAccessible` sugiere que esta clase está diseñada para ser parte de un paquete gestionado que necesita exponer funcionalidad a otros paquetes o al código de la organización.
3. La clase no incluye manejo de excepciones, lo que podría llevar a comportamientos inesperados si la llamada HTTP falla por razones distintas a un código de estado no 200 (por ejemplo, timeout, problemas de DNS, etc.).
4. El código actual no incluye documentación completa ni pruebas unitarias, que serían recomendables para una clase de esta naturaleza.