# LocalEmisionNotificationHandler

## Descripción General
`LocalEmisionNotificationHandler` es una clase que gestiona la integración para notificaciones de emisión local. Extiende `PlatformEventIntegrationObserver` e implementa las interfaces `Queueable` y `Database.AllowsCallouts`, lo que le permite ser ejecutada como trabajo en cola asíncrono y realizar llamadas HTTP a servicios externos.

## Autor y Modificaciones
- **Autor Original:** felipe.correa@nespon.com
- **Proyecto:** Foundation Salesforce - Suratech - UH-17589
- **Última modificación:** 03-03-2025
- **Modificado por:** fabian.lopez@nespon.com

## Anotaciones
- `global virtual`: La clase es accesible desde cualquier namespace y puede ser extendida por otras clases.

## Constantes
| Constante | Valor | Descripción |
|-----------|-------|-------------|
| `SERVICE_METHOD` | `'POST'` | Método HTTP utilizado para las llamadas al servicio externo |
| `SURATECH_MDT_CONFIG_NAME` | `'emisionlocal_endpoint'` | Nombre del registro de metadatos personalizado que contiene la configuración de integración |

## Propiedades
| Propiedad | Tipo | Descripción |
|-----------|------|-------------|
| `input` | `Map<String, Object>` | Datos de entrada para el proceso de integración |
| `output` | `Map<String, Object>` | Datos de salida del proceso de integración |
| `options` | `Map<String, Object>` | Opciones de configuración para el proceso |
| `policyId` | `String` | Identificador de la póliza relacionada (declarado pero no utilizado en el código) |

## Constructor
```apex
global LocalEmisionNotificationHandler(
  Map<String, Object> input,
  Map<String, Object> output,
  Map<String, Object> options
)
```

**Descripción:** Inicializa una nueva instancia de la clase con los parámetros proporcionados.

**Parámetros:**
- `input`: Datos de entrada para el proceso
- `output`: Contenedor para los datos de salida
- `options`: Opciones de configuración

## Métodos

### execute
```apex
global void execute(QueueableContext context)
```

**Descripción:** Implementación del método requerido por la interfaz `Queueable`. Inicia el proceso de integración cuando el trabajo en cola es ejecutado.

**Parámetros:**
- `context`: Contexto de la ejecución en cola

### preProcess
```apex
global virtual Map<String, Object> preProcess(
  Map<String, Object> input,
  Map<String, Object> options
)
```

**Descripción:** Método virtual que realiza el pre-procesamiento de los datos de entrada antes de la integración.

**Parámetros:**
- `input`: Datos de entrada para procesar
- `options`: Opciones de configuración

**Retorna:** Los datos de entrada posiblemente modificados

**Funcionalidad:**
1. Registra un mensaje de depuración
2. Si las opciones incluyen 'PlatformEventEnabled' establecido como verdadero, publica un evento de plataforma con los datos de entrada
3. Devuelve los datos de entrada (potencialmente modificados)

### invokeProcess
```apex
global Map<String, Object> invokeProcess(
  Map<String, Object> input,
  Map<String, Object> output,
  Map<String, Object> options
)
```

**Descripción:** Método principal que orquesta todo el proceso de integración.

**Parámetros:**
- `input`: Datos de entrada para el proceso
- `output`: Contenedor para los datos de salida
- `options`: Opciones de configuración

**Retorna:** Mapa con la respuesta procesada del servicio externo o null en caso de error

**Funcionalidad:**
1. Obtiene la configuración de integración desde metadatos personalizados
2. Establece la configuración para eventos de plataforma
3. Ejecuta el pre-procesamiento de los datos
4. Valida que el producto esté incluido en los datos de entrada
5. Añade la dirección IP externa de Salesforce a los datos de entrada
6. Estructura la carga útil en el formato esperado por el servicio
7. Invoca el servicio externo a través de `ExternalNotificationHandler`
8. Procesa la respuesta y publica un evento de plataforma con los resultados
9. Ejecuta el post-procesamiento de la respuesta
10. Maneja cualquier excepción que ocurra durante el proceso

### postProcess
```apex
global virtual Map<String, Object> postProcess(
  Map<String, Object> response,
  Map<String, Object> options
)
```

**Descripción:** Método virtual que realiza el post-procesamiento de la respuesta del servicio.

**Parámetros:**
- `response`: Respuesta del servicio externo
- `options`: Opciones de configuración

**Retorna:** La respuesta posiblemente modificada

**Funcionalidad:**
1. Registra un mensaje de depuración
2. Si las opciones incluyen 'PlatformEventEnabled' establecido como verdadero, publica un evento de plataforma con los datos de la respuesta
3. Devuelve la respuesta (potencialmente modificada)

### getExternalIPAddress
```apex
public static String getExternalIPAddress()
```

**Descripción:** Método estático que obtiene la dirección IP externa del servidor de Salesforce.

**Retorna:** String con la dirección IP externa

**Funcionalidad:**
1. Registra los límites actuales a través de `Core.getSymmaryLimits()`
2. Realiza una llamada HTTP GET al servicio ipify.org
3. Registra y devuelve la respuesta que contiene la dirección IP

### publishPlatformEventIntegration
```apex
global void publishPlatformEventIntegration(String moment, String data)
```

**Descripción:** Publica diferentes tipos de eventos de plataforma según el momento del proceso.

**Parámetros:**
- `moment`: Momento del proceso ('pre', 'post', 'in')
- `data`: Datos a incluir en el evento

**Funcionalidad:**
Utiliza un switch para determinar qué tipo de evento publicar:
- 'pre': Publica un evento de pre-proceso inicial
- 'post': Publica un evento de post-proceso inicial
- 'in': Publica un evento general con código de estado 200

## Flujo de Trabajo Principal
1. La clase puede ser instanciada y ejecutada como un trabajo en cola (Queueable)
2. Cuando se ejecuta, sigue un patrón de pre-proceso → proceso principal → post-proceso
3. Durante el proceso principal:
    - Obtiene configuración desde metadatos personalizados
    - Valida los datos de entrada
    - Enriquece los datos con información adicional (como la IP externa)
    - Realiza llamadas a servicios externos
    - Maneja las respuestas y las excepciones

## Integración con Eventos de Plataforma
La clase utiliza eventos de plataforma para registrar y comunicar diferentes etapas del proceso de integración:
- Eventos de pre-proceso ('pre')
- Eventos durante el proceso ('in')
- Eventos de post-proceso ('post')

## Dependencias
- `PlatformEventIntegrationObserver`: Clase base que proporciona funcionalidad para eventos de plataforma
- `ExternalNotificationHandler`: Clase para interactuar con servicios externos
- `Core`: Clase de utilidad para depuración y registro
- `SF_IntegrationSetting__mdt`: Metadatos personalizados para configuración de integración

## Casos de Uso
Esta clase se utiliza principalmente para:
1. Enviar notificaciones sobre emisiones locales a servicios externos
2. Integrar Salesforce con sistemas externos para la emisión de pólizas o documentos
3. Proporcionar un flujo de trabajo asíncrono para operaciones de integración que pueden tomar tiempo

## Consideraciones y Mejores Prácticas
- La clase está diseñada para ser extendida (virtual), permitiendo personalizar el comportamiento en clases derivadas
- Utiliza un patrón asíncrono para evitar problemas de límites de tiempo en Salesforce
- Implementa manejo de excepciones para garantizar que los errores sean registrados adecuadamente
- Usa eventos de plataforma para proporcionar visibilidad sobre el proceso de integración
- El nombre de configuración de metadatos sugiere variantes para diferentes productos (comentario indica "_Arrendamientos, _Viajes, _Autos, _Motos")