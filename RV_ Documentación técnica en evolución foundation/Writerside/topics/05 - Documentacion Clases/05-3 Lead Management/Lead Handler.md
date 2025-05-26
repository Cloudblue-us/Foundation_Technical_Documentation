# LeadProductHandler

Este handler gestiona el procesamiento de leads para diferentes productos, aplicando la lógica de negocio específica y preparando los datos para su procesamiento posterior.

## 📋 Información General

| Atributo | Valor |
|----------|-------|
| **Nombre API** | `LeadProductHandler` |
| **Extiende** | `ProductHandler` |
| **Contexto** | `global virtual` |
| **Proceso** | `LEAD` |
| **Desarrollador original** | Soulberto Lorenzo |
| **Última modificación** | 25-04-2025 (Jean Carlos Melendez) |

## 🧩 Principales características

### 📦 Constantes

- **PROCESS_NAME**: Define el nombre del proceso como "LEAD".

## 🔄 Métodos principales

### 👨‍💼 Constructor

```apex
global LeadProductHandler(String productName)
```

- **Descripción**: Inicializa la instancia del handler con el nombre del producto.
- **Parámetros**:
    - `productName` (String): Nombre del producto asociado al lead.

### 👁️ setObservers()

```apex
public override void setObservers()
```

- **Descripción**: Configura los observadores para este handler.
- **Implementación**: Añade un `LeadObserver` a la lista de observadores.

### 🏷️ setProcessName(String processName)

```apex
global virtual override void setProcessName(String processName)
```

- **Descripción**: Establece el nombre del proceso.
- **Parámetros**:
    - `processName` (String): Nombre del proceso a establecer.

### 📤 setResponseProcess(Object responseProcess)

```apex
global virtual override void setResponseProcess(Object responseProcess)
```

- **Descripción**: Establece la respuesta del proceso.
- **Parámetros**:
    - `responseProcess` (Object): Objeto con la respuesta del proceso.

### 📥 setRequestInput(Object requestInput)

```apex
global virtual override void setRequestInput(Object requestInput)
```

- **Descripción**: Establece los datos de entrada para el proceso.
- **Parámetros**:
    - `requestInput` (Object): Objeto con los datos de entrada.

### ⚙️ process(Map<String, Object> options, Map<String, Object> input, Map<String, Object> output)

```apex
public override Map<String, Object> process(
  Map<String, Object> options,
  Map<String, Object> input,
  Map<String, Object> output
)
```

- **Descripción**: Método principal que ejecuta el procesamiento del lead.
- **Parámetros**:
    - `options` (Map<String, Object>): Opciones de configuración para el proceso.
    - `input` (Map<String, Object>): Datos de entrada para el proceso.
    - `output` (Map<String, Object>): Mapa donde se almacena el resultado.
- **Retorna**:
    - Map<String, Object>: El mapa de salida con el resultado del proceso.
- **Comportamiento**:
    1. Verifica si debe omitirse el comportamiento predeterminado.
    2. Establece el nombre del proceso.
    3. Obtiene los configuradores de lead del mapa de entrada.
    4. Crea un nuevo lead utilizando el patrón Builder.
    5. Registra la finalización del proceso y guarda el resultado.
    6. Maneja excepciones DML y generales, lanzando excepciones personalizadas.

### ✅ isValidForLeadRequirements(Map<String, Object> data)

```apex
global virtual Boolean isValidForLeadRequirements(Map<String, Object> data)
```

- **Descripción**: Valida que los datos cumplan con los requisitos de negocio para la creación de un lead.
- **Parámetros**:
    - `data` (Map<String, Object>): Datos a validar.
- **Retorna**:
    - Boolean: `true` si los datos son válidos, `false` en caso contrario.
- **Notas**: Este método está diseñado para ser sobrescrito en implementaciones específicas.

## 🔄 Flujo de ejecución

1. Se instancia el handler con el nombre del producto.
2. Se configuran los observadores llamando a `setObservers()`.
3. Se invoca el método `process()` con los mapas correspondientes.
4. Se validan los datos y se crea el lead utilizando el builder.
5. Se registra el resultado y se devuelve en el mapa de salida.

## 🧠 Puntos de extensión

Este handler puede ser extendido de las siguientes maneras:

1. **Sobrescribir el método `isValidForLeadRequirements()`**: Para implementar validaciones específicas de negocio.
2. **Crear un handler que extienda `LeadProductHandler`**: Para personalizar comportamientos específicos por producto.
3. **Agregar configuradores adicionales**: Mediante la lista `configurators` para modificar la creación del lead.

## 📊 Ejemplo práctico

```apex
// Creación de un handler para leads de producto Viajes
LeadProductHandler viajesHandler = new LeadProductHandler('Viajes');

// Configuración de los datos de entrada
Map<String, Object> input = new Map<String, Object>{
  'lastName' => 'Pérez',
  'firstName' => 'Juan',
  'email' => 'juan.perez@example.com',
  'mobile_phone' => '3124567890',
  'status' => '0_Abierto'
};

// Procesamiento del lead
Map<String, Object> output = new Map<String, Object>();
Map<String, Object> options = new Map<String, Object>();
output = viajesHandler.process(options, input, output);

// Obtención del resultado
Lead createdLead = (Lead)output.get('resultProcess');
```

## 🧪 Consideraciones técnicas

- Utiliza el patrón Observer para notificar sobre eventos durante el procesamiento.
- Implementa el patrón Builder para la construcción del objeto Lead.
- Utiliza manejo de excepciones personalizadas para informar errores específicos.
- Los errores se devuelven con un código HTTP 422 (UNPROCESSABLE_ENTITY).

## 🔍 Log y trazabilidad

El proceso registra información en el objeto `Subconjunto_Registro_de_Evento__c` para permitir la trazabilidad completa, incluyendo:
- Datos de entrada
- Respuesta del proceso
- Detalles de la ejecución
- Usuario responsable