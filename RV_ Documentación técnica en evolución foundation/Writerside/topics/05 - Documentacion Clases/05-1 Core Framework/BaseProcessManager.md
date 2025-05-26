# BaseProcessManager

## Descripción General
Clase global que actúa como el gestor principal de procesos en el sistema Foundation. Implementa el patrón Chain of Responsibility para ejecutar secuencias de `ProductHandler` organizados por etapas (stages), con soporte opcional para cache y manejo dinámico de configuraciones.

## Información de la Clase
- **Autor:** Jean Carlos Melendez
- **Última modificación:** 04-01-2025
- **Modificado por:** fabian.lopez@nespon.com
- **Tipo:** Clase global con sharing
- **Propósito:** Gestor centralizado de procesos y cadenas de responsabilidad

## Método Principal

### invoke (Método Estático Global)
```apex
global static Boolean invoke(
    String stage,
    Map<String, Object> options,
    Map<String, Object> input,
    Map<String, Object> output
)
```

**Descripción:** Método principal que ejecuta una cadena de procesos para una etapa específica, construyendo dinámicamente la cadena de responsabilidad y ejecutándola.

**Parámetros:**
- `stage` (String): Etapa del proceso a ejecutar, debe existir en `SF_MainProcess__mdt.Stage__c`
- `options` (Map<String, Object>): Opciones de configuración para el proceso
- `input` (Map<String, Object>): Datos de entrada para el proceso
- `output` (Map<String, Object>): Mapa donde se almacenarán los resultados del proceso

**Retorna:**
- `Boolean`: Siempre retorna `true` si la ejecución es exitosa

**Excepciones:**
- `ProcessManagerNotFoundException`: Se lanza cuando el stage no existe en los metadatos

## Flujo de Ejecución Detallado

### 1. Validación de Stage
```apex
if (!CorePicklistUtils.containsVal(SF_MainProcess__mdt.Stage__c, stage))
```
- Valida que el stage exista en el Custom Metadata Type `SF_MainProcess__mdt`
- Utiliza `CorePicklistUtils` para verificar valores válidos
- Lanza excepción si el stage no es válido

### 2. Evaluación de Cache
```apex
FeatureFlag__mdt flag = [
    SELECT Is_Active__c
    FROM FeatureFlag__mdt
    WHERE DeveloperName = 'sura_foundation_cache'
    LIMIT 1
];
cacheEnabled = flag.Is_Active__c;
```

**Comportamiento:**
- Consulta el feature flag `sura_foundation_cache`
- Si existe y está activo, habilita el cache
- En caso de excepción (flag no existe), deshabilita el cache
- Establece `ignoreCache = !cacheEnabled`

### 3. Construcción de la Cadena de Responsabilidad
```apex
ReverseIterator reverseIterator = new ReverseIterator(
    ((List<ProductHandler>) ProcessFactory.newInstances(
        ProcessFactory.getAllDefinitionsByStage(stage, ignoreCache)
    ))
);
```

**Proceso:**
1. `ProcessFactory.getAllDefinitionsByStage(stage, ignoreCache)` obtiene definiciones
2. `ProcessFactory.newInstances()` crea instancias de `ProductHandler`
3. `ReverseIterator` invierte el orden para construcción correcta de la cadena

### 4. Enlazado de Handlers (Chain Building)
```apex
ProductHandler previous = (ProductHandler) reverseIterator.next();

while (reverseIterator.hasNext()) {
    ProductHandler current = (ProductHandler) reverseIterator.next();
    current.setNext(previous);
    previous = current;
}
```

**Algoritmo:**
- Itera en reversa para construir la cadena correctamente
- Cada handler actual apunta al anterior como `next`
- El resultado es una cadena donde el primer handler procesará toda la secuencia

### 5. Ejecución de la Cadena
```apex
((ProductHandler) reverseIterator.first()).apply(options, input, output);
```
- Ejecuta el primer handler de la cadena
- Cada handler decide si procesar y/o pasar al siguiente
- Los datos fluyen a través de `options`, `input` y `output`

### 6. Finalización y Logging
```apex
System.debug(output);
System.debug('finalizar proceso baseprocess');
Core.getSymmaryLimits();
```
- Registra el output final en debug
- Ejecuta `Core.getSymmaryLimits()` para monitoreo de límites
- Retorna `true` indicando ejecución exitosa

## Patrones de Diseño Implementados

### Chain of Responsibility
- **Propósito:** Permite que múltiples objetos manejen una solicitud sin acoplar el emisor con los receptores
- **Implementación:** Cada `ProductHandler` puede procesar la solicitud y/o pasarla al siguiente
- **Ventajas:** Flexibilidad para agregar/remover handlers dinámicamente

### Factory Pattern
- **Uso:** `ProcessFactory` para crear instancias de handlers
- **Beneficio:** Centraliza la lógica de creación e instanciación

### Iterator Pattern
- **Implementación:** `ReverseIterator` para navegación controlada
- **Propósito:** Facilita la construcción correcta de la cadena

## Componentes y Dependencias

### Dependencias Principales
- **SF_MainProcess__mdt:** Custom Metadata Type para definición de stages
- **FeatureFlag__mdt:** Para control de cache (`sura_foundation_cache`)
- **ProcessFactory:** Factory para creación de handlers
- **ProductHandler:** Clase base para handlers de la cadena
- **ReverseIterator:** Iterador personalizado para construcción de cadena
- **CorePicklistUtils:** Utilidades para validación de picklist values
- **Core:** Utilidades del sistema (límites, debugging)

### Excepciones Personalizadas
- **ProcessManagerNotFoundException:** Excepción específica para stages no encontrados

## Configuración del Sistema

### Feature Flags Utilizados
- `sura_foundation_cache`: Controla si el cache está habilitado para el sistema

### Custom Metadata Types
- `SF_MainProcess__mdt`: Define los stages disponibles y sus configuraciones
- `FeatureFlag__mdt`: Configuración de feature flags del sistema

## Casos de Uso

### Procesamiento de Órdenes
```apex
Map<String, Object> input = new Map<String, Object>{'orderId' => '123'};
Map<String, Object> output = new Map<String, Object>();
Map<String, Object> options = new Map<String, Object>{'validateOnly' => false};

BaseProcessManager.invoke('ORDER_PROCESSING', options, input, output);
```

### Validación de Datos
```apex
BaseProcessManager.invoke('DATA_VALIDATION', options, inputData, results);
```

### Integración con Sistemas Externos
```apex
BaseProcessManager.invoke('EXTERNAL_INTEGRATION', config, requestData, responseData);
```

## Ventajas del Diseño

### Flexibilidad
- Configuración dinámica de handlers por stage
- Soporte para cache configurable
- Fácil adición/remoción de pasos del proceso

### Mantenibilidad
- Separación clara de responsabilidades
- Configuración basada en metadata
- Logging integrado para debugging

### Performance
- Cache opcional para mejorar rendimiento
- Factory pattern para optimización de instanciación
- Monitoreo de límites integrado

### Escalabilidad
- Soporte para múltiples stages
- Handlers reutilizables entre diferentes procesos
- Configuración sin código a través de metadata

## Consideraciones de Performance

### Cache Management
- Feature flag `sura_foundation_cache` controla el uso de cache
- Cache puede mejorar significativamente la performance en procesos frecuentes
- Considerar el impacto de memory vs. processing time

### Límites de Salesforce
- `Core.getSymmaryLimits()` monitorea el uso de límites
- Importante para procesos que consumen muchos recursos
- Permite identificar cuellos de botella

## Mejoras Potenciales
- Implementar retry logic para procesos fallidos
- Agregar métricas de performance por handler
- Soporte para procesamiento asíncrono (Queueable/Batch)
- Implementar circuit breaker pattern para handlers problemáticos
- Agregar validación de tipos para input/output maps
- Soporte para handlers condicionales basados en datos de entrada