# Documentación Técnica: IFeatureFlags

## Descripción General
`IFeatureFlags` es una interfaz que define el contrato para la evaluación de banderas de características (feature flags) en el sistema. Esta interfaz establece el método necesario para evaluar si una característica específica está habilitada o no, devolviendo un resultado detallado de la evaluación.

## Autor
Fabian Lopez \<fabian.lopez@nespon.com\>

## Última Modificación
11 de abril de 2025

## Definición de la Interfaz

```apex
public interface IFeatureFlags {
    FeatureFlags.FeatureEvaluationResult evaluate(String featureName);
}
```

## Métodos

### `evaluate(String featureName)`

#### Descripción
Este método evalúa si una característica específica está habilitada, basándose en el nombre de la característica proporcionado.

#### Parámetros
- `featureName` (String): Nombre de la característica a evaluar.

#### Retorno
- `FeatureFlags.FeatureEvaluationResult`: Objeto que contiene el resultado detallado de la evaluación de la característica, incluyendo si está habilitada y posiblemente información adicional sobre el motivo de la decisión.

## Dependencias
- `FeatureFlags.FeatureEvaluationResult`: Clase anidada o interna que define la estructura del resultado de evaluación de una característica.

## Propósito y Aplicaciones

El propósito principal de esta interfaz es proporcionar una abstracción para el mecanismo de evaluación de banderas de características, permitiendo:

1. **Decisiones Dinámicas**: Tomar decisiones en tiempo de ejecución sobre qué características están disponibles.
2. **Flexibilidad de Implementación**: Permitir diferentes estrategias de evaluación que pueden ser intercambiadas o modificadas sin afectar al código que las utiliza.
3. **Pruebas Simplificadas**: Facilitar la creación de implementaciones mock para pruebas unitarias.
4. **Desacoplamiento**: Separar la lógica de decisión sobre características de la lógica de negocio que depende de esas decisiones.

Esta interfaz es especialmente útil en escenarios como:

- **Despliegue Progresivo**: Habilitar nuevas características gradualmente para diferentes segmentos de usuarios.
- **Pruebas A/B**: Mostrar diferentes variantes de una característica a diferentes usuarios para medir su eficacia.
- **Switches de Emergencia**: Desactivar rápidamente características problemáticas sin necesidad de redeployer.
- **Personalización por Usuario/Perfil**: Adaptar la experiencia del usuario basándose en su perfil, permisos o preferencias.

## Clase FeatureEvaluationResult Inferida

Aunque no se muestra en el código proporcionado, podemos inferir que existe una clase `FeatureEvaluationResult` dentro de una clase principal `FeatureFlags`. Esta clase probablemente tendría una estructura similar a la siguiente:

```apex
public class FeatureFlags {
    
    public class FeatureEvaluationResult {
        private final Boolean isEnabled;
        private final String reason;
        private final Map<String, Object> metadata;
        
        public FeatureEvaluationResult(Boolean isEnabled, String reason) {
            this.isEnabled = isEnabled;
            this.reason = reason;
            this.metadata = new Map<String, Object>();
        }
        
        public FeatureEvaluationResult(Boolean isEnabled, String reason, Map<String, Object> metadata) {
            this.isEnabled = isEnabled;
            this.reason = reason;
            this.metadata = metadata != null ? metadata : new Map<String, Object>();
        }
        
        public Boolean isEnabled() {
            return isEnabled;
        }
        
        public String getReason() {
            return reason;
        }
        
        public Map<String, Object> getMetadata() {
            return metadata;
        }
        
        public Object getMetadataValue(String key) {
            return metadata.containsKey(key) ? metadata.get(key) : null;
        }
    }
    
    // Otros métodos y clases de la clase principal FeatureFlags...
}
```

## Ejemplos de Implementación

### Implementación Básica

```apex
public class BasicFeatureFlags implements IFeatureFlags {
    
    private Map<String, Boolean> featuresMap;
    
    public BasicFeatureFlags() {
        // Inicializar con algunas características predeterminadas
        featuresMap = new Map<String, Boolean>{
            'NewUserInterface' => true,
            'AdvancedReporting' => false,
            'MultiLanguageSupport' => true
        };
    }
    
    public FeatureFlags.FeatureEvaluationResult evaluate(String featureName) {
        if (String.isBlank(featureName)) {
            return new FeatureFlags.FeatureEvaluationResult(false, 'Feature name is blank');
        }
        
        Boolean isEnabled = featuresMap.containsKey(featureName) ? featuresMap.get(featureName) : false;
        String reason = isEnabled ? 'Feature explicitly enabled' : 'Feature not enabled or not found';
        
        return new FeatureFlags.FeatureEvaluationResult(isEnabled, reason);
    }
}
```

### Implementación Basada en Metadatos Personalizados

```apex
public class MetadataFeatureFlags implements IFeatureFlags {
    
    private IFeatureFlagProvider provider;
    
    public MetadataFeatureFlags() {
        this.provider = new DefaultFeatureFlagProvider();
    }
    
    public MetadataFeatureFlags(IFeatureFlagProvider provider) {
        this.provider = provider;
    }
    
    public FeatureFlags.FeatureEvaluationResult evaluate(String featureName) {
        if (String.isBlank(featureName)) {
            return new FeatureFlags.FeatureEvaluationResult(false, 'Feature name is blank');
        }
        
        Map<String, FeatureFlag__mdt> featureFlags = provider.getFeatureFlags();
        
        // Si la bandera no existe, la característica está deshabilitada
        if (!featureFlags.containsKey(featureName)) {
            return new FeatureFlags.FeatureEvaluationResult(
                false, 
                'Feature not found in metadata'
            );
        }
        
        FeatureFlag__mdt flag = featureFlags.get(featureName);
        
        // Si la bandera no está activa, la característica está deshabilitada
        if (!flag.Is_Active__c) {
            return new FeatureFlags.FeatureEvaluationResult(
                false, 
                'Feature is defined but not active'
            );
        }
        
        // Si la bandera requiere un permiso personalizado, verificar si el usuario actual lo tiene
        if (String.isNotBlank(flag.Custom_Permission_Name__c)) {
            Boolean hasPermission = FeatureManagement.checkPermission(flag.Custom_Permission_Name__c);
            
            if (!hasPermission) {
                return new FeatureFlags.FeatureEvaluationResult(
                    false, 
                    'User lacks required permission: ' + flag.Custom_Permission_Name__c
                );
            }
        }
        
        // Crear un mapa con metadatos adicionales que podrían ser útiles
        Map<String, Object> metadata = new Map<String, Object>{
            'flagId' => flag.DeveloperName,
            'lastModified' => flag.LastModifiedDate,
            'description' => flag.Description__c
        };
        
        // Si no se requiere permiso personalizado y la bandera está activa, la característica está habilitada
        return new FeatureFlags.FeatureEvaluationResult(
            true, 
            'Feature is active' + (String.isNotBlank(flag.Custom_Permission_Name__c) ? 
                                ' and user has required permission' : ''),
            metadata
        );
    }
}
```

### Implementación Basada en Usuarios

```apex
public class UserBasedFeatureFlags implements IFeatureFlags {
    
    private Set<String> betaUserEmails;
    private Set<String> betaFeatures;
    
    public UserBasedFeatureFlags() {
        // Inicializar conjuntos de usuarios beta y características beta
        betaUserEmails = new Set<String>{
            'test.user@example.com',
            'beta.tester@example.com'
        };
        
        betaFeatures = new Set<String>{
            'NewSearchAlgorithm',
            'RedesignedProfile',
            'EnhancedDashboards'
        };
    }
    
    public FeatureFlags.FeatureEvaluationResult evaluate(String featureName) {
        if (String.isBlank(featureName)) {
            return new FeatureFlags.FeatureEvaluationResult(false, 'Feature name is blank');
        }
        
        // Si no es una característica beta, siempre está habilitada
        if (!betaFeatures.contains(featureName)) {
            return new FeatureFlags.FeatureEvaluationResult(true, 'Not a beta feature');
        }
        
        // Obtener el email del usuario actual
        String userEmail = UserInfo.getUserEmail();
        
        // Verificar si el usuario es un beta tester
        Boolean isBetaUser = betaUserEmails.contains(userEmail);
        
        // Crear metadatos con información adicional
        Map<String, Object> metadata = new Map<String, Object>{
            'isBetaFeature' => true,
            'userEmail' => userEmail
        };
        
        if (isBetaUser) {
            return new FeatureFlags.FeatureEvaluationResult(
                true, 
                'User is a beta tester with access to this feature',
                metadata
            );
        } else {
            return new FeatureFlags.FeatureEvaluationResult(
                false, 
                'User is not a beta tester',
                metadata
            );
        }
    }
}
```

### Implementación Compuesta

```apex
public class CompositeFeatureFlags implements IFeatureFlags {
    
    private List<IFeatureFlags> evaluators;
    
    public CompositeFeatureFlags() {
        // Inicializar con evaluadores predeterminados
        evaluators = new List<IFeatureFlags>{
            new MetadataFeatureFlags(),
            new UserBasedFeatureFlags()
        };
    }
    
    public CompositeFeatureFlags(List<IFeatureFlags> evaluators) {
        this.evaluators = evaluators != null ? evaluators : new List<IFeatureFlags>();
    }
    
    public FeatureFlags.FeatureEvaluationResult evaluate(String featureName) {
        if (String.isBlank(featureName)) {
            return new FeatureFlags.FeatureEvaluationResult(false, 'Feature name is blank');
        }
        
        // Si cualquier evaluador habilita la característica, se considera habilitada
        for (IFeatureFlags evaluator : evaluators) {
            FeatureFlags.FeatureEvaluationResult result = evaluator.evaluate(featureName);
            if (result.isEnabled()) {
                return result;
            }
        }
        
        // Si ningún evaluador habilitó la característica, devolver el resultado del último evaluador
        // o un resultado predeterminado si no hay evaluadores
        if (!evaluators.isEmpty()) {
            return evaluators[evaluators.size() - 1].evaluate(featureName);
        } else {
            return new FeatureFlags.FeatureEvaluationResult(false, 'No feature flag evaluators available');
        }
    }
    
    public void addEvaluator(IFeatureFlags evaluator) {
        if (evaluator != null) {
            evaluators.add(evaluator);
        }
    }
}
```

### Implementación Mock para Pruebas

```apex
@IsTest
public class MockFeatureFlags implements IFeatureFlags {
    
    private Map<String, Boolean> featuresMap;
    private Map<String, String> reasonsMap;
    
    public MockFeatureFlags() {
        featuresMap = new Map<String, Boolean>();
        reasonsMap = new Map<String, String>();
    }
    
    public FeatureFlags.FeatureEvaluationResult evaluate(String featureName) {
        Boolean isEnabled = featuresMap.containsKey(featureName) ? featuresMap.get(featureName) : false;
        String reason = reasonsMap.containsKey(featureName) ? reasonsMap.get(featureName) : 'Default mock reason';
        
        return new FeatureFlags.FeatureEvaluationResult(isEnabled, reason);
    }
    
    // Métodos adicionales para configurar el mock
    
    public void setFeatureEnabled(String featureName, Boolean isEnabled) {
        featuresMap.put(featureName, isEnabled);
    }
    
    public void setFeatureReason(String featureName, String reason) {
        reasonsMap.put(featureName, reason);
    }
    
    public void reset() {
        featuresMap.clear();
        reasonsMap.clear();
    }
}
```

## Ejemplo de Uso

```apex
public class ProductController {
    
    private final IFeatureFlags featureFlags;
    
    public ProductController() {
        // Usar la implementación predeterminada
        this.featureFlags = new MetadataFeatureFlags();
    }
    
    // Constructor con inyección de dependencias para pruebas
    @TestVisible
    private ProductController(IFeatureFlags featureFlags) {
        this.featureFlags = featureFlags;
    }
    
    public List<Product2> getProductsForDisplay() {
        List<Product2> products = [SELECT Id, Name, Description, Family, IsActive FROM Product2 WHERE IsActive = true];
        
        // Verificar si la característica de filtrado avanzado está habilitada
        FeatureFlags.FeatureEvaluationResult advancedFilteringResult = 
            featureFlags.evaluate('AdvancedProductFiltering');
        
        if (advancedFilteringResult.isEnabled()) {
            // Aplicar filtrado avanzado
            return applyAdvancedFiltering(products);
        } else {
            // Aplicar filtrado básico
            return applyBasicFiltering(products);
        }
    }
    
    private List<Product2> applyAdvancedFiltering(List<Product2> products) {
        // Implementación del filtrado avanzado
        // ...
        return products;
    }
    
    private List<Product2> applyBasicFiltering(List<Product2> products) {
        // Implementación del filtrado básico
        // ...
        return products;
    }
}
```

## Ejemplo de Uso en Controlador Lightning

```javascript
// productController.js
import { LightningElement, wire } from 'lwc';
import checkFeatureEnabled from '@salesforce/apex/FeatureFlagController.checkFeatureEnabled';
import getProducts from '@salesforce/apex/ProductController.getProducts';

export default class ProductList extends LightningElement {
    products;
    error;
    showAdvancedSearch = false;
    
    connectedCallback() {
        // Verificar si la característica de búsqueda avanzada está habilitada
        checkFeatureEnabled({ featureName: 'AdvancedProductSearch' })
            .then(result => {
                this.showAdvancedSearch = result.isEnabled;
                if (result.isEnabled) {
                    console.log('Advanced search enabled: ' + result.reason);
                }
            })
            .catch(error => {
                console.error('Error checking feature flag:', error);
            });
    }
    
    @wire(getProducts)
    wiredProducts({ error, data }) {
        if (data) {
            this.products = data;
            this.error = undefined;
        } else if (error) {
            this.error = error;
            this.products = undefined;
        }
    }
    
    // Resto del controlador...
}
```

```apex
// FeatureFlagController.cls
public with sharing class FeatureFlagController {
    
    private static IFeatureFlags featureFlags = new MetadataFeatureFlags();
    
    @AuraEnabled(cacheable=true)
    public static Map<String, Object> checkFeatureEnabled(String featureName) {
        FeatureFlags.FeatureEvaluationResult result = featureFlags.evaluate(featureName);
        
        // Convertir el resultado a un mapa que pueda ser serializado para JavaScript
        return new Map<String, Object>{
            'isEnabled' => result.isEnabled(),
            'reason' => result.getReason()
        };
    }
}
```

## Ejemplo de Uso en Pruebas

```apex
@IsTest
private class ProductControllerTest {
    
    @IsTest
    static void testGetProductsWithAdvancedFilteringEnabled() {
        // Configurar datos de prueba
        createTestProducts();
        
        // Crear un mock de IFeatureFlags
        MockFeatureFlags mockFeatureFlags = new MockFeatureFlags();
        mockFeatureFlags.setFeatureEnabled('AdvancedProductFiltering', true);
        mockFeatureFlags.setFeatureReason('AdvancedProductFiltering', 'Enabled for test');
        
        // Crear una instancia del controlador con el mock
        ProductController controller = new ProductController(mockFeatureFlags);
        
        // Ejecutar el método a probar
        Test.startTest();
        List<Product2> products = controller.getProductsForDisplay();
        Test.stopTest();
        
        // Verificar que se aplicó el filtrado avanzado
        // (Aquí se agregarían las aserciones específicas según la implementación del filtrado)
        System.assertEquals(5, products.size(), 'Should return 5 products with advanced filtering');
    }
    
    @IsTest
    static void testGetProductsWithAdvancedFilteringDisabled() {
        // Configurar datos de prueba
        createTestProducts();
        
        // Crear un mock de IFeatureFlags
        MockFeatureFlags mockFeatureFlags = new MockFeatureFlags();
        mockFeatureFlags.setFeatureEnabled('AdvancedProductFiltering', false);
        mockFeatureFlags.setFeatureReason('AdvancedProductFiltering', 'Disabled for test');
        
        // Crear una instancia del controlador con el mock
        ProductController controller = new ProductController(mockFeatureFlags);
        
        // Ejecutar el método a probar
        Test.startTest();
        List<Product2> products = controller.getProductsForDisplay();
        Test.stopTest();
        
        // Verificar que se aplicó el filtrado básico
        // (Aquí se agregarían las aserciones específicas según la implementación del filtrado)
        System.assertEquals(3, products.size(), 'Should return 3 products with basic filtering');
    }
    
    private static void createTestProducts() {
        // Lógica para crear productos de prueba
        // ...
    }
}
```

## Relación con IFeatureFlagProvider

La interfaz `IFeatureFlags` probablemente trabaja en conjunto con la interfaz `IFeatureFlagProvider` en un sistema de gestión de banderas de características más amplio:

- `IFeatureFlagProvider`: Se enfoca en obtener y proporcionar acceso a la configuración de las banderas de características.
- `IFeatureFlags`: Se enfoca en la evaluación lógica de si una característica está habilitada o no para un contexto particular.

Una implementación típica de `IFeatureFlags` (como la clase `MetadataFeatureFlags` en los ejemplos) podría utilizar un `IFeatureFlagProvider` para obtener la configuración de las banderas y luego aplicar la lógica de evaluación.

## Mejores Prácticas

1. **Inyección de Dependencias**: Utilizar inyección de dependencias para proporcionar implementaciones específicas de `IFeatureFlags` a las clases que lo necesitan, facilitando las pruebas y la flexibilidad.
2. **Resultados Informativos**: Proporcionar razones claras y detalladas en los resultados de evaluación para facilitar la depuración y el entendimiento.
3. **Caché Inteligente**: Implementar mecanismos de caché para evitar evaluaciones repetitivas de las mismas características en una misma transacción.
4. **Auditoría**: Considerar registrar evaluaciones importantes para análisis y depuración.
5. **Comportamiento Predeterminado Seguro**: Si una característica no está definida, normalmente debería considerarse como deshabilitada.

## Consideraciones de Diseño

### Ventajas
1. **Flexibilidad**: Permite intercambiar diferentes estrategias de evaluación sin modificar el código cliente.
2. **Testabilidad**: Facilita la prueba de código que depende de banderas de características.
3. **Separación de Responsabilidades**: Separa la lógica de evaluación de características de la lógica de negocio.
4. **Extensibilidad**: Permite añadir nuevas estrategias de evaluación sin modificar el código existente.

### Desafíos
1. **Complejidad**: Introduce una capa adicional de abstracción.
2. **Overhead de Performance**: Puede añadir cierto overhead de rendimiento, especialmente si la evaluación es compleja.
3. **Mantenimiento**: Requiere mantener la configuración de banderas actualizada y relevante.

## Notas Adicionales
1. La interfaz `IFeatureFlags` es parte de un sistema más amplio de gestión de características que probablemente incluye otros componentes como proveedores de configuración, servicios de administración, etc.
2. La clase `FeatureEvaluationResult` proporciona más información que un simple booleano, permitiendo entender por qué una característica está habilitada o deshabilitada.
3. Este enfoque a menudo se utiliza en conjunto con estrategias de despliegue continuo, permitiendo liberar código que contiene características incompletas o experimentales sin exponerlas a todos los usuarios.
4. Las implementaciones concretas pueden variar significativamente según los requisitos específicos del sistema, desde simples verificaciones de configuración hasta algoritmos complejos que consideran múltiples factores como perfiles de usuario, regiones geográficas, horarios, etc.