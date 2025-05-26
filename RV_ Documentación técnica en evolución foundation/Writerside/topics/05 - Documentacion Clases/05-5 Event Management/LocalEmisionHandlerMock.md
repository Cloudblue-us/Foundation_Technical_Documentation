# LocalEmisionNotificationHandlerMock

## Descripción
Clase mock para simular respuestas HTTP durante pruebas del manejador de notificaciones de emisión local. Implementa la interfaz HttpCalloutMock para interceptar llamadas HTTP salientes durante pruebas.

## Autor
ChangeMeIn@UserSettingsUnder.SFDoc

## Última modificación
31 de enero de 2025 por ChangeMeIn@UserSettingsUnder.SFDoc

## Detalles
Esta clase de prueba permite simular diferentes respuestas del servicio de notificaciones de emisión local, incluyendo respuestas exitosas y escenarios de error.

## Anotaciones
- `@isTest`: Indica que esta clase es exclusivamente para pruebas.
- `global`: Visibilidad máxima para permitir su uso en cualquier contexto de prueba.

## Implementación
Implementa la interfaz `HttpCalloutMock` para simular respuestas HTTP durante pruebas.

## Propiedades
| Nombre | Tipo | Visibilidad | Descripción |
|--------|------|-------------|-------------|
| statusCode | Integer | private | Código de estado HTTP a devolver en la respuesta simulada |
| status | String | private | Mensaje de estado HTTP a devolver |
| cException | Boolean | private | Bandera para indicar si se debe lanzar una excepción |

## Constructor
```apex
global LocalEmisionNotificationHandlerMock(Integer statusCode, String status, boolean cException)
```
Configura la instancia mock con los parámetros especificados para controlar su comportamiento.

### Parámetros
- `statusCode`: Código de estado HTTP que devolverá la respuesta simulada
- `status`: Mensaje de estado HTTP que devolverá la respuesta
- `cException`: Si es `true`, el mock lanzará una excepción de callout en lugar de devolver una respuesta

## Métodos

### respond
```apex
global HTTPResponse respond(HTTPRequest req)
```
Método principal que implementa la interfaz HttpCalloutMock. Genera y devuelve una respuesta HTTP simulada basada en la configuración del mock.

#### Parámetros
- `req`: La solicitud HTTP interceptada

#### Retorno
- Objeto `HTTPResponse` con la respuesta simulada según la configuración

#### Comportamiento
- Si `cException` es verdadero, lanza una excepción `CalloutException` con el mensaje "Error en el servicio"
- De lo contrario, construye una respuesta HTTP con el código y estado configurados
- El cuerpo de la respuesta depende del endpoint:
    - Para requests a "https://api.ipify.org", devuelve una IP simulada
    - Para otros endpoints, devuelve un objeto JSON vacío

### localEmisionBodyResponse (private)
```apex
private static String localEmisionBodyResponse()
```
Genera el cuerpo de respuesta estándar para el servicio de emisión local.

#### Retorno
- Un objeto JSON vacío como string: `"{}"`

### salesforceExternalIP (private)
```apex
private static String salesforceExternalIP()
```
Proporciona una dirección IP simulada para las solicitudes al servicio de identificación de IP externa.

#### Retorno
- Una dirección IP simulada como string: `"155.226.129.250"`