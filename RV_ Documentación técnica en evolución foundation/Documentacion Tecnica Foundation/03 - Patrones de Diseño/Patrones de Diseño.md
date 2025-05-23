# Patrones de Diseño - Framework Foundation

## Descripción General
Este documento describe los patrones de diseño implementados en el Framework Foundation, analizando su propósito, implementación y beneficios dentro de la arquitectura del sistema. El framework utiliza múltiples patrones que trabajan en conjunto para crear una solución robusta, extensible y mantenible.

## Patrones Implementados

### 1. Observer Pattern

#### Implementación
**Clase:** `BaseObserver`
**Interfaz:** `IObserver`

#### Propósito
Permite que múltiples objetos observen y reaccionen a eventos del sistema sin acoplamiento directo entre el emisor y los receptores.

#### Cómo se Utiliza
```apex
public virtual class BaseObserver implements IObserver {
    public virtual void updateObserver(String flag, Object data) {
        Core.debug('flag="' + flag);
    }
    
    public virtual void publishPlatFormEvent(String flag, String productName) {
        Core.debug('Publishing platform event for product= ' + productName);
    }
}
```

#### Beneficios en el Framework
- **Desacoplamiento:** Los componentes pueden comunicarse sin conocerse directamente
- **Extensibilidad:** Nuevos observadores pueden agregarse sin modificar el código existente
- **Event-Driven Architecture:** Facilita una arquitectura basada en eventos
- **Reactividad:** El sistema puede responder automáticamente a cambios de estado

#### Casos de Uso
- Notificaciones de cambios de estado en objetos de negocio
- Logging automático de eventos del sistema
- Integración con Platform Events de Salesforce
- Auditoría de operaciones críticas

---

### 2. Provider Pattern

#### Implementación
**Clase:** `FeatureFlagProvider`
**Interfaz:** `IFeatureFlagProvider`

#### Propósito
Abstrae la obtención de datos y configuraciones, permitiendo diferentes fuentes de información sin cambiar el código cliente.

#### Cómo se Utiliza
```apex
public with sharing class FeatureFlagProvider implements IFeatureFlagProvider {
    public Set<String> getCustomPermissionNames(){
        // Obtiene permisos personalizados desde Salesforce
        List<CustomPermission> perms = [SELECT Id, DeveloperName FROM CustomPermission];
        // ... procesamiento
        return customPermissionNames;
    }

    public Map<String,FeatureFlag__mdt> getFeatureFlags(){
        return FeatureFlag__mdt.getAll();
    }
}
```

#### Beneficios en el Framework
- **Abstracción de Fuentes:** Oculta la complejidad de obtener datos de diferentes fuentes
- **Testabilidad:** Permite inyección de dependencias para testing
- **Flexibilidad:** Diferentes proveedores para diferentes entornos
- **Cacheo:** Centraliza estrategias de cache y performance

#### Casos de Uso
- Obtención de feature flags desde Custom Metadata Types
- Acceso a permisos personalizados
- Configuraciones dinámicas sin deployments
- Testing con datos mock

---

### 3. Strategy Pattern

#### Implementación
**Clases:** `FeatureFlags`, `AccountConfigurator`
**Interfaces:** `IFeatureFlags`, `AccountConfigurator`

#### Propósito
Permite intercambiar algoritmos o comportamientos en tiempo de ejecución sin modificar el código cliente.

#### Cómo se Utiliza

##### En FeatureFlags
```apex
public FeatureEvaluationResult evaluate(String featureName) {
    // Estrategia 1: Mock Values (para testing)
    if (Test.isRunningTest() && mockValues.containsKey(featureName)) {
        return new FeatureEvaluationResult(mockValues.get(featureName), featureName, FeatureReason.MOCK_VALUE);
    }
    
    // Estrategia 2: Custom Permissions
    if (customPermissionNames.contains(featureName)) {
        if (FeatureManagement.checkPermission(featureName)) {
            return new FeatureEvaluationResult(true, featureName, FeatureReason.HAS_CUSTOM_PERMISSION);
        }
    }
    
    // Estrategia 3: Custom Metadata Types
    if (features.containsKey(featureName)) {
        if (features.get(featureName).Is_Active__c) {
            return new FeatureEvaluationResult(true, featureName, FeatureReason.CUSTOM_METADATA_TYPE_ENABLED);
        }
    }
}
```

##### En AccountConfigurator
```apex
global interface AccountConfigurator {
    void configure(Account account);
}

// Diferentes estrategias de configuración
global class IndustryAccountConfigurator implements AccountConfigurator {
    global void configure(Account account) {
        account.Industry = this.industry;
        // Lógica específica por industria
    }
}
```

#### Beneficios en el Framework
- **Intercambiabilidad:** Diferentes algoritmos para el mismo propósito
- **Extensibilidad:** Nuevas estrategias sin modificar código existente
- **Mantenibilidad:** Separación clara de responsabilidades
- **Configurabilidad:** Comportamiento dinámico basado en configuración

#### Casos de Uso
- Evaluación de feature flags con múltiples fuentes
- Configuración modular de objetos Account
- Algoritmos de procesamiento intercambiables
- Validaciones condicionales

---

### 4. Adapter Pattern

#### Implementación
**Clase:** `BaseProcessFoundationVlocityAdapter`

#### Propósito
Permite que interfaces incompatibles trabajen juntas, actuando como puente entre diferentes sistemas o APIs.

#### Cómo se Utiliza
```apex
global abstract class BaseProcessFoundationVlocityAdapter {
    global static Boolean applyInvoke(
        Map<String, Object> inputMap,
        Map<String, Object> outputMap,
        Map<String, Object> optionMap
    ) {
        String stage = (String) inputMap?.get('Stage');
        // Adapta la interfaz de Vlocity a BaseProcessManager
        return BaseProcessManager.invoke(stage, optionMap, inputMap, outputMap);
    }
    
    public Boolean invokeMethod(String methodName, Map<String, Object> inputMap, 
                               Map<String, Object> outputMap, Map<String, Object> optionMap) {
        // Invocación dinámica por nombre de método
        switch on methodName.toUpperCase() {
            when 'APPLYINVOKE' {
                return applyInvoke(inputMap, outputMap, optionMap);
            }
        }
    }
}
```

#### Beneficios en el Framework
- **Integración:** Conecta Vlocity framework con lógica de negocio personalizada
- **Abstracción:** Oculta complejidad del BaseProcessManager
- **Estandarización:** Interfaz consistente para diferentes invocaciones
- **Desacoplamiento:** Vlocity no depende directamente de clases internas

#### Casos de Uso
- Integración entre Vlocity y BaseProcessManager
- Adaptación de APIs externas
- Puente entre diferentes versiones de interfaces
- Normalización de parámetros entre sistemas

---

### 5. Chain of Responsibility Pattern

#### Implementación
**Clase:** `BaseProcessManager`
**Componentes:** `ProductHandler`, `ProcessFactory`, `ReverseIterator`

#### Propósito
Permite pasar solicitudes a lo largo de una cadena de manejadores hasta que uno de ellos procese la solicitud.

#### Cómo se Utiliza
```apex
global static Boolean invoke(String stage, Map<String, Object> options, 
                            Map<String, Object> input, Map<String, Object> output) {
    // 1. Obtener handlers para el stage
    ReverseIterator reverseIterator = new ReverseIterator(
        ((List<ProductHandler>) ProcessFactory.newInstances(
            ProcessFactory.getAllDefinitionsByStage(stage, ignoreCache)
        ))
    );

    // 2. Construir la cadena de responsabilidad
    if (!reverseIterator.isEmpty()) {
        ProductHandler previous = (ProductHandler) reverseIterator.next();
        
        while (reverseIterator.hasNext()) {
            ProductHandler current = (ProductHandler) reverseIterator.next();
            current.setNext(previous);  // Enlazar handlers
            previous = current;
        }

        // 3. Ejecutar la cadena
        ((ProductHandler) reverseIterator.first()).apply(options, input, output);
    }
    
    return true;
}
```

#### Beneficios en el Framework
- **Flexibilidad:** Configuración dinámica de la cadena de procesamiento
- **Desacoplamiento:** Handlers no conocen la estructura completa de la cadena
- **Extensibilidad:** Nuevos handlers sin modificar código existente
- **Configurabilidad:** Orden y composición basada en metadata

#### Casos de Uso
- Procesamiento de órdenes con múltiples validaciones
- Pipeline de transformación de datos
- Workflows configurables por stage
- Procesamiento secuencial con puntos de control

---

### 6. Builder Pattern

#### Implementación
**Clase:** `AccountBuilder`
**Interfaz:** `AccountConfigurator`

#### Propósito
Construye objetos complejos paso a paso, permitiendo diferentes representaciones del mismo tipo de objeto.

#### Cómo se Utiliza
```apex
global class AccountBuilder {
    global Account account;
    global List<AccountConfigurator> configurators = new List<AccountConfigurator>();

    // Métodos fluidos para construcción
    global AccountBuilder setName(String name) {
        account.Name = name;
        return this;
    }
    
    global AccountBuilder setPhone(String phone) {
        account.Phone = phone;
        return this;
    }
    
    global AccountBuilder addConfigurators(List<AccountConfigurator> configuratorsList) {
        this.configurators.addAll(configuratorsList);
        return this;
    }

    // Construcción final
    global Account build() {
        for (AccountConfigurator configurator : configurators) {
            configurator.configure(account);
        }
        return account;
    }
}
```

#### Beneficios en el Framework
- **Fluent Interface:** API intuitiva y legible
- **Flexibilidad:** Construcción opcional e incremental
- **Extensibilidad:** Configuradores personalizados
- **Reutilización:** Mismo builder para diferentes variaciones

#### Casos de Uso
- Construcción de objetos Account complejos
- Configuración modular con Strategy pattern
- APIs de creación user-friendly
- Testing con builders de datos mock

---

### 7. Factory Pattern

#### Implementación
**Clase:** `ProcessFactory` (referenciada en BaseProcessManager)

#### Propósito
Crea objetos sin especificar la clase exacta del objeto que será creado.

#### Cómo se Utiliza
```apex
// Uso implícito en BaseProcessManager
List<ProductHandler> handlers = ProcessFactory.newInstances(
    ProcessFactory.getAllDefinitionsByStage(stage, ignoreCache)
);
```

#### Beneficios en el Framework
- **Abstracción:** Oculta lógica de instanciación
- **Flexibilidad:** Diferentes tipos de handlers basados en configuración
- **Centralización:** Lógica de creación en un solo lugar
- **Cache:** Optimización de instanciación

#### Casos de Uso
- Creación dinámica de ProductHandlers
- Instanciación basada en metadata
- Cache de instancias para performance
- Inyección de dependencias

---

## Interacción Entre Patrones

### Feature Flags Ecosystem
```
Provider Pattern → Strategy Pattern → Observer Pattern
(FeatureFlagProvider) → (FeatureFlags) → (BaseObserver)
```

### Process Management Chain
```
Adapter Pattern → Chain of Responsibility → Factory Pattern
(VlocityAdapter) → (BaseProcessManager) → (ProcessFactory)
```

### Account Building System
```
Builder Pattern ← Strategy Pattern
(AccountBuilder) ← (AccountConfigurator)
```

## Beneficios Arquitectónicos

### Mantenibilidad
- **Separación de responsabilidades:** Cada patrón maneja una preocupación específica
- **Código modular:** Cambios aislados sin efectos secundarios
- **Testing independiente:** Cada componente puede probarse por separado

### Extensibilidad
- **Open/Closed Principle:** Abierto para extensión, cerrado para modificación
- **Configuración sin código:** Metadata-driven configuration
- **Plugin architecture:** Nuevas funcionalidades como plugins

### Performance
- **Cache strategies:** Implementadas en Provider y Factory patterns
- **Lazy loading:** Instanciación bajo demanda
- **Bulk operations:** Optimización para operaciones masivas

### Reusabilidad
- **Componentes intercambiables:** Interfaces bien definidas
- **Configuraciones reutilizables:** Estrategias aplicables en múltiples contextos
- **Abstracción de complejidad:** APIs simples para funcionalidad compleja

## Consideraciones de Implementación

### Performance
- Cache habilitado por feature flags
- Factory pattern para optimización de instanciación
- Configuradores eficientes para operaciones bulk

### Security
- Feature flags basados en permisos de usuario
- Validación en configuradores personalizados
- Respeto a field-level security

### Testing
- Mock strategies en Strategy pattern
- Builder pattern para datos de test
- Provider pattern para inyección de dependencias

## Evolución del Framework

### Patrones Futuros Recomendados
- **Command Pattern:** Para operaciones undoable
- **Memento Pattern:** Para state management
- **Visitor Pattern:** Para operaciones sobre estructuras complejas
- **Template Method:** Para algoritmos con pasos variables

### Mejoras Arquitectónicas
- Event sourcing con Observer pattern
- CQRS con Strategy pattern
- Microservices communication con Adapter pattern
- Domain-driven design con Builder pattern

## Conclusión

El Framework Foundation implementa una arquitectura sólida basada en patrones de diseño probados. La combinación de estos patrones crea un sistema que es:

- **Flexible:** Adaptable a diferentes requerimientos
- **Extensible:** Fácil de expandir con nueva funcionalidad
- **Mantenible:** Código limpio y bien estructurado
- **Testeable:** Componentes independientes y mockeables
- **Performante:** Optimizaciones integradas en el diseño

Esta arquitectura proporciona una base sólida para el desarrollo de aplicaciones empresariales complejas en Salesforce, manteniendo la simplicidad de uso mientras se ofrece la flexibilidad necesaria para casos de uso avanzados.