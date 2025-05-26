
# `ProductHandler` Apex Class Documentation

**Author**: Jean Carlos Melendez (<jean@cloudblue.us>)  
**Last Modified By**: Jean Carlos Melendez  
**Last Modified On**: 2025-04-23

## 📌 Overview

`ProductHandler` es una clase abstracta que define el comportamiento base para una cadena de responsabilidad (Chain of Responsibility) en el procesamiento de productos dentro de una arquitectura basada en eventos, observadores y banderas de características (`FeatureFlags`).

Su objetivo principal es permitir a los desarrolladores extender la lógica de procesamiento de productos de manera flexible y desacoplada, implementando manejadores específicos que heredan de esta clase.

---

## 🔧 Class Definition

```apex
global abstract class ProductHandler extends Subject
```

Esta clase:

- **Es global**: puede ser accedida desde cualquier contexto.
- **Es abstracta**: no puede ser instanciada directamente.
- **Extiende `Subject`**: lo que implica que implementa el patrón de observador (Observer pattern).

---

## 🧩 Properties

| Propiedad               | Tipo                 | Acceso  | Descripción |
|------------------------|----------------------|---------|-------------|
| `productName`          | `String`             | global  | Nombre del producto que el handler maneja. |
| `processName`          | `String`             | global  | Nombre del proceso que se está ejecutando. |
| `nextHandler`          | `ProductHandler`     | global  | Referencia al siguiente handler en la cadena. |
| `responseProcess`      | `Object`             | public  | Respuesta generada por el proceso del handler. |
| `requestInput`         | `Object`             | global  | Entrada original que recibe el handler. |
| `state`                | `String`             | public  | Estado del procesamiento (`success` o `error`). |
| `FLAG_PLATFORM_NAME`   | `String` (constante) | static  | Bandera para habilitar eventos de plataforma. |

---

## 🏗️ Constructor

### `ProductHandler(String productName)`
Inicializa una nueva instancia del handler con el nombre del producto que procesará.

---

## 🔁 Chain of Responsibility

### `setNext(ProductHandler nextHandler)`
Asigna el siguiente handler en la cadena.

**Retorna**: `ProductHandler` (el handler siguiente).

---

## ⚙️ Métodos de Configuración

### `setProcessName(String processName)`
Define el nombre del proceso actual.

### `setResponseProcess(Object responseProcess)`
Guarda la respuesta generada por el handler.

### `setRequestInput(Object requestInput)`
Guarda la entrada del request en la propiedad del handler.

---

## 👀 Observadores

### `setObservers()`
Agrega observadores al handler. Por defecto, se agrega un `BaseObserver`.

---

## 🎯 Lógica de Ejecución

### `shouldSkipHandler(Map options, Map input, Map output)`

Evalúa si el handler debe ser omitido según:

- Si `productName` está presente en el `input`.
- Si el `productName` coincide con el que maneja este handler.

**Retorna**: `Boolean`  
- `true` si debe ser omitido.
- `false` si debe ejecutar su lógica.

---

## 🔄 Proceso Principal

### `apply(Map options, Map input, Map output)`

Este es el método principal del handler. Ejecuta las siguientes fases:

1. Verifica si debe omitir el handler (`shouldSkipHandler`).
2. Configura los observadores.
3. Evalúa la `FeatureFlag` para eventos de plataforma.
4. Ejecuta el preprocesamiento (`onPre`).
5. Ejecuta el procesamiento principal (`process`).
6. Ejecuta el posprocesamiento (`onPos`).
7. Notifica observadores.
8. Publica log del servicio (`publishServiceEventLog`).
9. Captura y reporta errores personalizados (`ProductHandlerImplementationException`).

---

## 🛠️ Métodos de Fase de Proceso

| Método       | Descripción                                           | Entrada                                     | Retorno                         |
|--------------|-------------------------------------------------------|---------------------------------------------|----------------------------------|
| `process`    | Ejecuta la lógica principal del handler               | `options`, `input`, `output`                | `Map<String, Object>`           |
| `onPre`      | Lógica previa al procesamiento principal              | `options`, `input`, `output`                | `Map<String, Object>`           |
| `onPos`      | Lógica posterior al procesamiento principal           | `options`, `input`, `output`                | `Map<String, Object>`           |

> Estos métodos son virtuales y deben ser sobreescritos en las clases hijas.

---

## 🧾 Publicación de Logs

### `publishServiceEventLog(String executionResult)`

Publica un log asincrónico si la bandera `sura_foundation_event_logs` está habilitada. Incluye detalles del producto, respuesta serializada, proceso y plataforma.

---

## ⚠️ Excepciones

Manejo específico para `ProductHandlerImplementationException`, cuyos datos se añaden al `output`:

- `errorMsg`
- `errorCode`
- `error`

También se registra el estado como `"error"` y se serializa el mensaje de error.

---

## 🔄 Ejemplo de Cadena de Handlers

```apex
ProductHandler handlerA = new SomeProductHandler('ProductoA');
ProductHandler handlerB = new SomeProductHandler('ProductoB');

handlerA.setNext(handlerB);

Map<String, Object> input = new Map<String, Object>{ 'productName' => 'ProductoA' };
Map<String, Object> output = new Map<String, Object>();

handlerA.apply(new Map<String, Object>(), input, output);
```

---

## 📚 Recomendaciones

- Extiende esta clase e implementa `process`, `onPre` y `onPos` en tus propias clases de producto.
- Utiliza `FeatureFlags` para activar funcionalidades específicas sin modificar código productivo.
- Encadena handlers para distribuir la lógica por tipo de producto.
- 


# `LeadProductHandler` Apex Class Documentation

**Author**: Soulberto Lorenzo (<soulberto@cloudblue.us>)  
**Last Modified By**: Jean Carlos Melendez  
**Last Modified On**: 2025-04-25

## 📌 Overview

`LeadProductHandler` es una clase que extiende `ProductHandler` y representa un manejador específico para el producto tipo Lead. Esta clase implementa la lógica de creación de registros `Lead` en Salesforce usando un patrón de configuración flexible a través del constructor `LeadBuilderFoundation`.

También define validaciones específicas y observadores dedicados (`LeadObserver`) para su propio proceso de negocio.

---

## 🔧 Class Definition

```apex
global virtual class LeadProductHandler extends ProductHandler
```

Esta clase:

- **Es global y virtual**: puede ser accedida globalmente y sobreescrita si es necesario.
- **Extiende `ProductHandler`**: hereda todo el comportamiento de la cadena de responsabilidad.

---

## 🧩 Properties

| Propiedad     | Tipo     | Acceso  | Descripción                          |
|---------------|----------|---------|--------------------------------------|
| `PROCESS_NAME`| `String` | static final | Constante que define el nombre del proceso: `"LEAD"` |

---

## 🏗️ Constructor

### `LeadProductHandler(String productName)`

Inicializa una instancia del handler para un producto tipo Lead.

---

## 👀 Observadores

### `setObservers()`

Sobrescribe el método base para asignar el `LeadObserver` como observador del handler.

---

## ⚙️ Métodos de Configuración

Estos métodos sobrescriben los definidos en la clase base para garantizar coherencia:

| Método | Descripción |
|--------|-------------|
| `setProcessName` | Define el nombre del proceso (`LEAD`). |
| `setResponseProcess` | Asigna el objeto de respuesta del handler. |
| `setRequestInput` | Almacena el objeto de entrada que será procesado. |

---

## 🔄 Proceso Principal

### `process(Map options, Map input, Map output)`

Este método ejecuta la lógica principal del handler para crear un `Lead` usando `LeadBuilderFoundation`.

**Flujo de ejecución:**

1. Verifica si se debe omitir el comportamiento por defecto con la opción `skipDefaultBehavior`.
2. Configura el nombre del proceso como `"LEAD"`.
3. Extrae los configuradores (`LeadConfigurator`) del input.
4. Construye el objeto `Lead` mediante el patrón Builder.
5. Agrega el resultado al `output` y lo retorna.

**Manejo de errores:**

- Captura y transforma excepciones `DmlException` y `Exception` en `ProductHandlerImplementationException` con código HTTP 422.

**Retorna**:  
Un `Map<String, Object>` con el resultado del proceso (`resultProcess`).

---

## ✅ Validación de Reglas de Negocio

### `isValidForLeadRequirements(Map<String, Object> data)`

Método virtual para definir las validaciones de negocio específicas que deben cumplirse antes de crear un `Lead`. En esta implementación retorna siempre `true`, pero está diseñado para ser extendido.

**Retorna**: `Boolean`

---

## 📚 Ejemplo de Uso

```apex
LeadProductHandler leadHandler = new LeadProductHandler('ProductoLead');

Map<String, Object> input = new Map<String, Object>{
  'status' => 'Nuevo',
  'firstName' => 'Ana',
  'lastName' => 'García',
  'email' => 'ana.garcia@example.com',
  'company' => 'Empresa XYZ'
};

Map<String, Object> output = new Map<String, Object>();

leadHandler.apply(new Map<String, Object>(), input, output);

System.debug(output.get('resultProcess')); // Muestra el Lead creado
```

---

## 📝 Consideraciones

- `LeadBuilderFoundation` encapsula la lógica de creación del `Lead`.
- `LeadConfigurator` permite extender dinámicamente la configuración del Lead.
- Utiliza excepciones personalizadas para controlar errores del proceso.

---

# `LeadConfigurator` Apex Interface Documentation

**Author**: Jean Carlos Melendez  
**Last Modified By**: Jean Carlos Melendez  
**Last Modified On**: 2025-01-30

## 📌 Overview

`LeadConfigurator` es una interfaz global en Apex que define un contrato para aplicar configuraciones personalizadas a registros de tipo `Lead`. Su propósito es permitir la extensión modular del proceso de creación de Leads, brindando flexibilidad y encapsulamiento para adaptar configuraciones específicas según el contexto o negocio.

---

## 🔧 Interface Definition

```apex
global interface LeadConfigurator {
  void configure(Lead lead);
}
```

---

## 🧩 Method Summary

| Método         | Tipo de Retorno | Descripción |
|----------------|------------------|-------------|
| `configure`    | `void`           | Método obligatorio que implementa la lógica de configuración sobre un objeto `Lead`. |

---

## 🧪 Method Details

### `configure(Lead lead)`

**Descripción**:  
Este método debe ser implementado por cualquier clase que desee proporcionar lógica personalizada de configuración sobre un objeto `Lead`. Puede ser utilizado, por ejemplo, para asignar valores a campos adicionales, ejecutar validaciones o asociar datos relacionados al Lead durante su construcción.

**Parámetros**:

| Nombre | Tipo  | Descripción |
|--------|-------|-------------|
| `lead` | `Lead` | Instancia de `Lead` sobre la cual se aplicarán las configuraciones. |

**Retorno**:  
No retorna ningún valor (`void`), pero modifica directamente el objeto `Lead` recibido por referencia.

---

## 📚 Ejemplo de Implementación

```apex
global class CountryConfigurator implements LeadConfigurator {
  public void configure(Lead lead) {
    lead.Country = 'Colombia';
  }
}
```

---

## 📝 Consideraciones

- Esta interfaz permite desacoplar las configuraciones específicas del proceso de creación de Leads.
- Es útil cuando se construyen múltiples leads con diferentes reglas de negocio.
- Puede ser utilizada en conjunto con patrones como Builder o Chain of Responsibility.

---

## 🧠 Recomendación

Implementa múltiples clases `LeadConfigurator` para cubrir distintos aspectos del negocio y agrúpalas como una lista en el `LeadBuilderFoundation` o clases similares, mejorando así la mantenibilidad y escalabilidad del sistema.

---


# `RateProductHandler` Apex Class Documentation

**Author**: Soulberto Lorenzo (<soulberto@cloudblue.us>)  
**Last Modified By**: Jean Carlos Melendez  
**Last Modified On**: 2025-04-23



## 📌 Overview

`RateProductHandler` es una clase que extiende `ProductHandler` y está diseñada para manejar la tarificación de productos en Salesforce. Utiliza un Integration Procedure (IP) para calcular tarifas basadas en parámetros de entrada, gestionando tanto la validación como la ejecución del procedimiento y el formateo de resultados.

---

## 🔧 Class Definition

```apex
global virtual class RateProductHandler extends ProductHandler
```

---

## 🧩 Properties

| Propiedad               | Tipo                           | Acceso           | Descripción |
|------------------------|--------------------------------|------------------|-------------|
| `PROCESS_NAME`         | `String`                       | `static final`   | Nombre del proceso: `"TARIFICACION"` |
| `OPTION_SKIP_BEHAVIOR` | `String`                       | `static final`   | Opción para omitir la ejecución normal del handler. |
| `RATING_IPROCEDURE`    | `String`                       | `static final`   | Nombre del Integration Procedure utilizado para la tarificación. |
| `EXECUTOR`             | `IntegrationProcedureExecutor` | `global`         | Ejecuta el procedimiento de integración. |
| `REQUIRED_PARAMETERS`  | `Set<String>`                  | `static final`   | Parámetros obligatorios para ejecutar el IP. |

---

## 🏗️ Constructores

### `RateProductHandler(String productName, IntegrationProcedureExecutor executor)`
Crea una instancia del handler con un ejecutor personalizado.

### `RateProductHandler(String productName)`
Crea una instancia con un ejecutor simulado (`MockIntegrationProcedureExecutor`), útil para pruebas.

---

## ⚙️ Métodos de Configuración

| Método                      | Descripción |
|-----------------------------|-------------|
| `setProcessName(String)`    | Asigna el nombre del proceso. |
| `setResponseProcess(Object)`| Guarda la respuesta generada. |
| `setRequestInput(Object)`   | Guarda los datos de entrada procesados. |

---

## ✅ Validación de Parámetros

### `isValidForRatingParameters(Map<String, Object> data)`
Verifica que todos los parámetros definidos en `REQUIRED_PARAMETERS` estén presentes en el `input`.

**Retorna**: `true` si todos están presentes, `false` si falta alguno.

---

## 🔄 Método Principal

### `process(Map options, Map input, Map output)`
Este método es el núcleo de la lógica del handler. Valida, construye y ejecuta el IP.

**Retorno**: `Map<String, Object>` — Resultado del proceso de tarificación.

**Excepción**: Lanza `ProductHandlerImplementationException` si la validación falla.

---

## 🔧 Utilidades

### `buildInputMap(Map input, Map options)`
Combina el `input` original con las `options` para generar el mapa que se enviará al IP.

---

## ⚙️ Ejecución del Integration Procedure

### `executeIntegrationProcedure(Map ipInput)`
Ejecuta el Integration Procedure definido en `RATING_IPROCEDURE` usando el `EXECUTOR`.

---

## 📚 Ejemplo de Uso

```apex
RateProductHandler handler = new RateProductHandler('ProductoX');

Map<String, Object> input = new Map<String, Object>{
  'includeInputKeys' => true,
  'instanceKey' => 'INST001',
  'filters' => new Map<String, Object>{ 'edad' => 35 }
};

Map<String, Object> output = new Map<String, Object>();

handler.apply(new Map<String, Object>(), input, output);

System.debug(output.get('resultProcess'));
```

---

## 📝 Consideraciones

- Este handler se adapta perfectamente a arquitecturas basadas en IP.
- Recomendado sobrescribir `setObservers()` para agregar `RateObserver`.
- Bien estructurado para mantenibilidad y pruebas.

---

# `IntegrationProcedureExecutor` Apex Interface Documentation

**Author**: Jean Carlos Melendez  
**Last Modified By**: Jean Carlos Melendez  
**Last Modified On**: 2025-01-28


## 📌 Overview

`IntegrationProcedureExecutor` es una interfaz global que define el contrato para ejecutar procedimientos de integración en Salesforce.

---

## 🔧 Interface Definition

```apex
global interface IntegrationProcedureExecutor {
  Map<String, Object> execute(String procedureName, Map<String, Object> input);
}
```

---

## 🧩 Method Summary

| Método    | Tipo de Retorno       | Descripción |
|-----------|------------------------|-------------|
| `execute` | `Map<String, Object>`  | Ejecuta un Integration Procedure y retorna los resultados. |

---

## 📥 Method Details

### `execute(String procedureName, Map<String, Object> input)`
Ejecuta un IP con el nombre y mapa de entrada especificado.

**Parámetros**:

| Nombre          | Tipo                | Descripción |
|-----------------|---------------------|-------------|
| `procedureName` | `String`            | Nombre del IP a ejecutar. |
| `input`         | `Map<String, Object>` | Datos de entrada. |

**Retorna**:  
`Map<String, Object>` — Respuesta del IP.

---

## 📚 Ejemplo de Implementación

```apex
goblal class VlocityIntegrationProcedureExecutor implements IntegrationProcedureExecutor {
  public Map<String, Object> execute(
    String procedureName,
    Map<String, Object> input
  ) {
    
    return (Map<String, Object>) vlocity_ins.IntegrationProcedureService.runIntegrationService(
      procedureName,
      input,
      new Map<String, Object>()
    );
  }
}
```

---

## 📝 Consideraciones

- Ideal para pruebas y desacoplamiento de lógica de negocio.
- Facilita el cumplimiento del principio de inversión de dependencias (SOLID).

---

# `QuoteProductHandler` Apex Class Documentation

**Author**: Soulberto Lorenzo (<soulberto@cloudblue.us>)  
**Last Modified By**: Jean Carlos Melendez  
**Last Modified On**: 2025-04-02

## 📌 Overview

`QuoteProductHandler` es una implementación concreta de `ProductHandler` que permite procesar cotizaciones de productos mediante un `IntegrationProcedure` especializado. Esta clase incluye validaciones estrictas de los datos de entrada, ejecuta un procedimiento de cotización y retorna los resultados, integrando observadores y manejando excepciones adecuadamente.

---

## 🔧 Class Definition

```apex
global virtual class QuoteProductHandler extends ProductHandler
```

Esta clase:

- Es global y virtual, permitiendo su extensión.
- Hereda de `ProductHandler`.
- Está diseñada para manejar lógica de cotización de seguros u otros productos complejos.

---

## 🧩 Properties

| Propiedad               | Tipo                           | Acceso         | Descripción |
|------------------------|--------------------------------|----------------|-------------|
| `PROCESS_NAME`         | `String`                       | `static final` | Nombre del proceso: `"COTIZACION"` |
| `MAIN_NODE`            | `String`                       | `static final` | Nodo principal esperado en el input: `"quotepolicyJson"` |
| `OPTION_SKIP_BEHAVIOR` | `String`                       | `static final` | Clave para omitir el procesamiento por defecto. |
| `RATING_IPROCEDURE`    | `String`                       | `static final` | Nombre del Integration Procedure para cotización. |
| `EXECUTOR`             | `IntegrationProcedureExecutor` | `global`       | Ejecuta el procedimiento de cotización. |
| `REQUIRED_PARAMETERS`  | `Set<String>`                  | `static final` | Nodos requeridos dentro de `MAIN_NODE`. |

---

## 🏗️ Constructores

### `QuoteProductHandler(String productName, IntegrationProcedureExecutor executor)`
Crea una instancia del handler utilizando un executor personalizado para ambientes productivos.

### `QuoteProductHandler(String productName)`
Crea una instancia usando un `MockIntegrationProcedureExecutor`, ideal para pruebas.

---

## 👀 Observadores

### `setObservers()`
Registra el observador `QuoteObserver` para recibir notificaciones sobre el proceso.

---

## ⚙️ Métodos de Configuración

| Método | Descripción |
|--------|-------------|
| `setProcessName(String)` | Asigna el nombre del proceso al handler. |
| `setResponseProcess(Object)` | Guarda la respuesta procesada. |
| `setRequestInput(Object)` | Guarda los datos de entrada para el proceso. |

---

## 🔄 Proceso Principal

### `process(Map<String, Object> options, Map<String, Object> input, Map<String, Object> output)`

Este método orquesta el flujo completo del proceso de cotización.

**Pasos**:
1. Omite el procesamiento si se incluye la opción `skipDefaultBehavior`.
2. Define el nombre del proceso como `"COTIZACION"`.
3. Valida que todos los nodos requeridos estén presentes.
4. Ejecuta el `IntegrationProcedure` para generar la cotización.
5. Asigna el resultado al mapa `output` y lo serializa como respuesta.

**Parámetros**:
- `options`: Opciones adicionales del proceso.
- `input`: Datos de entrada (incluye `quotepolicyJson`).
- `output`: Mapa de salida que será completado con el resultado.

**Retorna**: `Map<String, Object>` — Resultado del proceso de cotización.

**Excepciones**: Lanza `ProductHandlerImplementationException` si la validación falla.

---

## 🧪 Validación de Entrada

### `validateInputMapForQuoting(Map inputMap)`

Verifica que el nodo principal y los nodos internos requeridos estén presentes en `quotepolicyJson`.

**Nodos requeridos**:
- `term`
- `productConfigurationDetail`
- `insuredItems`
- `additionalFields`
- `OpportunityDetails`

**Retorna**: `Map<Boolean, Object>` — Resultado estándar de validación con mensaje descriptivo.

---

## 🧱 Utilidades

### `createValidationResult(Boolean isValid, String message)`

Crea un resultado de validación estructurado en forma de mapa.

**Ejemplo de retorno**:
```apex
{ true => 'Todos los parámetros requeridos están presentes.' }
```

---

## ⚙️ Ejecución de Integration Procedure

### `executeIntegrationProcedure(Map ipInput)`

Llama al executor con el nombre definido (`sfcore_quotingprocedure`) y los datos de entrada.

**Retorna**:
```apex
{ 'resultProcess' => <resultado del executor> }
```

---

## 📚 Ejemplo de Uso

```apex
QuoteProductHandler quoteHandler = new QuoteProductHandler('ProductoCotizacion');

Map<String, Object> input = new Map<String, Object>{
  'quotepolicyJson' => new Map<String, Object>{
    'term' => '12M',
    'productConfigurationDetail' => 'PCD001',
    'insuredItems' => new List<Object>{},
    'additionalFields' => new Map<String, Object>(),
    'OpportunityDetails' => new Map<String, Object>()
  }
};

Map<String, Object> output = new Map<String, Object>();

quoteHandler.apply(new Map<String, Object>(), input, output);

System.debug(output.get('resultProcess'));
```

---

## 📝 Consideraciones

- Ideal para procesos de cotización que requieren integración con sistemas externos mediante IPs.
- Es completamente extensible y puede integrarse con distintos observers para monitoreo.
- Utiliza un patrón de validación clara para evitar errores por falta de estructura.


# `IssueProductHandler` Apex Class Documentation

**Author**: Jean Carlos Melendez (<jean@cloudblue.us>)  
**Last Modified By**: Jean Carlos Melendez  
**Last Modified On**: 2025-04-02

---

## 📌 Overview

`IssueProductHandler` es una implementación de `ProductHandler` responsable de manejar el proceso de emisión de productos. Utiliza un Integration Procedure para realizar la lógica de negocio asociada a la emisión, validando los datos de entrada, realizando preprocesamiento, ejecutando el IP y actualizando el estado de la oportunidad relacionada.

---

## 🔧 Class Definition

```apex
global virtual class IssueProductHandler extends ProductHandler
```

---

## 🧩 Properties

| Propiedad                       | Tipo                           | Acceso           | Descripción |
|--------------------------------|--------------------------------|------------------|-------------|
| `FLAG_PLATFORM_NAME`           | `String`                       | `static final`   | Nombre de la Feature Flag para eventos de plataforma. |
| `IP_PREPROCESS`                | `String`                       | `static final`   | Nombre del IP de preprocesamiento. |
| `PROCESS_NAME`                 | `String`                       | `static final`   | Nombre del proceso manejado: `"TARIFICACION"` |
| `OPTION_SKIP_BEHAVIOR`        | `String`                       | `static final`   | Clave para omitir comportamiento por defecto. |
| `ISSUING_IPROCEDURE`          | `String`                       | `static final`   | Nombre del IP utilizado para la emisión. |
| `ERROR_REQUIRED_FIELD`        | `String`                       | `static final`   | Mensaje de error usado en validación de campos. |
| `EXECUTOR`                    | `IntegrationProcedureExecutor` | `global`         | Objeto encargado de ejecutar el Integration Procedure. |
| `REQUIRED_FIELD_FOR_ISSUE_CORE` | `List<String>`               | `static final`   | Lista de campos obligatorios para ejecutar la emisión. |

---

## 🏗️ Constructores

### `IssueProductHandler(String productName, IntegrationProcedureExecutor executor)`
Inicializa el handler con un ejecutor personalizado.

### `IssueProductHandler(String productName)`
Inicializa con un `MockIntegrationProcedureExecutor`.

---

## ⚙️ Métodos de Configuración

| Método                        | Descripción |
|-------------------------------|-------------|
| `setProcessName(String)`      | Define el nombre del proceso. |
| `setResponseProcess(Object)`  | Guarda la respuesta del proceso. |
| `setRequestInput(Object)`     | Guarda los datos de entrada originales. |

---

## 🔄 Métodos del Proceso

### `onPre(Map options, Map input, Map output)`

Realiza validaciones previas y asigna las cuentas aseguradas a la cotización.

### `process(Map options, Map input, Map output)`

Valida los datos requeridos, ejecuta la validación previa (`preIssuingValidation`) y llama al IP de emisión.

### `onPos(Map options, Map input, Map output)`

Actualiza la etapa de la oportunidad relacionada a `'Poliza emitida'`.

### `executeIntegrationProcedure(Map ipInput)`

Ejecuta el IP configurado (`SURAFoundation_IssuingProcedure`) con los datos de entrada validados.

---

## ✅ Validaciones

### `validateInput(Map input, List<String> requiredFields)`

Valida que los campos obligatorios estén presentes en el mapa `input`. Lanza excepción si alguno está ausente.

### `preIssuingValidation(Map input)`

Valida los campos clave para emisión (`quoteId`, `effectiveDate`, `producerId`) y garantiza que la cotización y la oportunidad asociada existan y estén completas.

---

## 🔧 Métodos Auxiliares

### `assignAccountsToQuote(List<Map<String, Object>> parties, Id quoteId)`

Asocia las cuentas aseguradas al objeto Quote y su oportunidad.

### `listInsuredPartyAccounts(Map<String, Object> input)`

Convierte la información del lead a cuentas aseguradas mediante `LeadUtils`.

---

## 📚 Ejemplo de Ejecución

```apex
IssueProductHandler handler = new IssueProductHandler('ProductoX');

Map<String, Object> input = new Map<String, Object>{
  'leadId' => '00Qxx0000001abc',
  'quoteId' => '0Q0xx0000002def',
  'effectiveDate' => '2025-04-15',
  'producerId' => '001xx000003xyzA'
};

Map<String, Object> output = new Map<String, Object>();

handler.apply(new Map<String, Object>(), input, output);

System.debug(output.get('resultProcess'));
```

---

## 📝 Consideraciones

- El método `onPre` asegura que las cuentas estén correctamente asociadas antes de emitir.
- `onPos` actualiza automáticamente el estado de la oportunidad una vez finalizada la emisión.
- Utiliza validaciones sólidas para garantizar que los datos estén completos antes de ejecutar el IP.
- El handler puede ser fácilmente extendido o simulado en pruebas gracias al uso de `IntegrationProcedureExecutor`.




