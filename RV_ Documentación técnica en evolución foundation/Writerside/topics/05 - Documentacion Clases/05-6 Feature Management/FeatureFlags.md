# FeatureFlags

## Descripción General
Clase principal para la evaluación y gestión de feature flags en el sistema. Implementa la interfaz `IFeatureFlags` y proporciona un mecanismo robusto para activar/desactivar funcionalidades basado en Custom Permissions y Custom Metadata Types.

## Información de la Clase
- **Autor:** felipe.correa@nespon.com
- **Proyecto:** Foundation Salesforce - Suratech - UH-17589
- **Última modificación:** 04-11-2025
- **Modificado por:** fabian.lopez@nespon.com
- **Tipo:** Clase pública con sharing
- **Implementa:** IFeatureFlags

## Propiedades Privadas

### features
```apex
private Map<String, FeatureFlag__mdt> features
```
Mapa que contiene todos los feature flags definidos como Custom Metadata Types.

### customPermissionNames
```apex
private Set<String> customPermissionNames
```
Conjunto de nombres de Custom Permissions disponibles en la organización.

### mockValues
```apex
private static Map<String, Boolean> mockValues
```
Mapa estático para almacenar valores mock durante la ejecución de tests.

## Constructores

### Constructor con Provider
```apex
public FeatureFlags(IFeatureFlagProvider provider)
```
**Descripción:** Constructor que permite inyección de dependencias para el proveedor de feature flags.

**Parámetros:**
- `provider` (IFeatureFlagProvider): Implementación del proveedor de feature flags

**Comportamiento:**
- Inicializa `features` usando `provider.getFeatureFlags()`
- Inicializa `customPermissionNames` usando `provider.getCustomPermissionNames()`

### Constructor por Defecto
```apex
public FeatureFlags()
```
**Descripción:** Constructor por defecto que utiliza `FeatureFlagProvider` como implementación estándar.

**Comportamiento:**
- Llama al constructor principal con una nueva instancia de `FeatureFlagProvider`

## Métodos Principales

### evaluate
```apex
public FeatureEvaluationResult evaluate(String featureName)
```

**Descripción:** Método principal para evaluar el estado de un feature flag específico.

**Parámetros:**
- `featureName` (String): Nombre del feature flag a evaluar

**Retorna:**
- `FeatureEvaluationResult`: Objeto que contiene el resultado, nombre del feature y razón de la evaluación

**Lógica de Evaluación:**

1. **Contexto de Test:**
    - Si `Test.isRunningTest()` es true, configura valores mock para features específicos
    - `sura_foundation_platform_events` = true (para tests de PlatformEventIntegrationObserver)
    - `sura_foundation_event_logs` = true (para tests de EventLogManager)
    - Retorna valor mock si existe

2. **Evaluación por Custom Permission:**
    - Si el feature name existe en `customPermissionNames`
    - Usa `FeatureManagement.checkPermission()` para verificar el permiso
    - Retorna `HAS_CUSTOM_PERMISSION` o `MISSING_CUSTOM_PERMISSION`

3. **Evaluación por Custom Metadata Type:**
    - Si el feature existe en el mapa `features`
    - Verifica el campo `Is_Active__c`
    - Retorna `CUSTOM_METADATA_TYPE_ENABLED` o `CUSTOM_METADATA_TYPE_DISABLED`

4. **Feature No Encontrado:**
    - Si el feature no existe en ninguna fuente
    - Retorna `FLAG_NOT_FOUND`

### lwcEvaluate (Método Estático para LWC)
```apex
@AuraEnabled(cacheable=true)
public static Boolean lwcEvaluate(String featureName)
```

**Descripción:** Método estático optimizado para componentes Lightning Web Components.

**Parámetros:**
- `featureName` (String): Nombre del feature flag a evaluar

**Retorna:**
- `Boolean`: Estado del feature flag (true/false)

**Características:**
- Anotado con `@AuraEnabled(cacheable=true)` para optimización de cache
- Crea una nueva instancia de `FeatureFlags` para cada evaluación
- Simplifica el resultado a un boolean

### setMockValue (Método de Test)
```apex
@TestVisible
private static void setMockValue(String featureName, Boolean value)
```

**Descripción:** Método utilitario para configurar valores mock durante tests.

**Parámetros:**
- `featureName` (String): Nombre del feature flag
- `value` (Boolean): Valor mock a asignar

**Visibilidad:** Solo visible durante tests (`@TestVisible`)

## Clases Internas

### FeatureEvaluationResult
```apex
public class FeatureEvaluationResult
```

**Descripción:** Clase interna que encapsula el resultado de una evaluación de feature flag.

**Propiedades:**
- `result` (Boolean): Estado del feature flag
- `featureName` (String): Nombre del feature evaluado
- `reason` (FeatureReason): Razón de la evaluación

**Métodos:**
- `isEnabled()`: Retorna el estado boolean del feature
- `getFeatureName()`: Retorna el nombre del feature
- `getReason()`: Retorna la razón de la evaluación

**Constructor:**
```apex
public FeatureEvaluationResult(Boolean result, String featureName, FeatureReason reason)
```

### FeatureReason (Enum)
```apex
public enum FeatureReason
```

**Valores:**
- `HAS_CUSTOM_PERMISSION`: Feature habilitado por Custom Permission
- `MISSING_CUSTOM_PERMISSION`: Feature deshabilitado por falta de Custom Permission
- `CUSTOM_METADATA_TYPE_ENABLED`: Feature habilitado por Custom Metadata Type
- `CUSTOM_METADATA_TYPE_DISABLED`: Feature deshabilitado por Custom Metadata Type
- `FLAG_NOT_FOUND`: Feature flag no encontrado en el sistema
- `MOCK_VALUE`: Valor configurado para testing

## Estrategias de Evaluación

### Jerarquía de Evaluación
1. **Mock Values** (solo en tests)
2. **Custom Permissions** (basado en usuario/perfil)
3. **Custom Metadata Types** (configuración global)
4. **Default** (feature no encontrado = false)

### Ventajas del Diseño
- **Flexibilidad:** Soporta múltiples fuentes de configuración
- **Granularidad:** Control a nivel de usuario (permissions) y global (metadata)
- **Testabilidad:** Sistema de mocking integrado
- **Performance:** Cache habilitado para LWC
- **Trazabilidad:** Razones detalladas de evaluación

## Integración con Lightning Web Components

### Uso en LWC
```javascript
import { wire } from 'lwc';
import lwcEvaluate from '@salesforce/apex/FeatureFlags.lwcEvaluate';

export default class MyComponent extends LightningElement {
    @wire(lwcEvaluate, { featureName: 'my_feature' })
    featureResult;
    
    get isFeatureEnabled() {
        return this.featureResult.data === true;
    }
}
```

## Casos de Uso

### Desarrollo
- Control de features en desarrollo vs producción
- Activación gradual de funcionalidades (rollout progresivo)
- A/B testing basado en perfiles de usuario

### Administración
- Habilitar/deshabilitar features sin deployments
- Control granular por usuario o perfil
- Configuración centralizada de comportamientos

### Testing
- Simulación de diferentes estados de features
- Tests independientes del estado real de configuración
- Validación de comportamiento con features activadas/desactivadas

## Dependencias
- **IFeatureFlags:** Interfaz implementada
- **IFeatureFlagProvider:** Para obtención de datos
- **FeatureFlagProvider:** Implementación por defecto del proveedor
- **FeatureFlag__mdt:** Custom Metadata Type para configuración
- **FeatureManagement:** API de Salesforce para Custom Permissions

## Consideraciones de Performance
- Custom Metadata Types están cacheados automáticamente
- Custom Permissions se evalúan en tiempo real
- Método LWC con cache habilitado para mejor performance en UI
- Mock values solo se procesan durante tests

## Mejoras Potenciales
- Implementar cache local para Custom Permissions
- Agregar logging de evaluaciones para auditoría
- Soporte para feature flags con valores complejos (no solo boolean)
- Integración con herramientas de monitoreo para uso de features
- Implementar feature flags con fecha de expiración