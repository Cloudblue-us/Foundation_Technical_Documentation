# Documentación Técnica: IFeatureFlagProvider

## Descripción General
`IFeatureFlagProvider` es una interfaz que define el contrato para proveedores de banderas de características (feature flags) en el sistema. Esta interfaz establece los métodos necesarios para acceder tanto a los nombres de permisos personalizados como a las configuraciones de banderas de características almacenadas en metadatos personalizados.

## Definición de la Interfaz

```apex
public interface IFeatureFlagProvider {
    
    Set<String> getCustomPermissionNames();
    Map<String,FeatureFlag__mdt> getFeatureFlags();

}
```

## Métodos

### `getCustomPermissionNames()`

#### Descripción
Este método debe devolver un conjunto de nombres de permisos personalizados disponibles en el sistema. Los permisos personalizados suelen utilizarse como uno de los mecanismos para controlar la activación de características específicas para diferentes usuarios o perfiles.

#### Parámetros
Ninguno.

#### Retorno
- `Set<String>`: Conjunto de nombres de permisos personalizados.

### `getFeatureFlags()`

#### Descripción
Este método debe devolver un mapa de banderas de características definidas en el sistema. Las banderas de características están almacenadas en registros de metadatos personalizados (`FeatureFlag__mdt`) y permiten configurar características específicas que pueden ser activadas o desactivadas de forma dinámica.

#### Parámetros
Ninguno.

#### Retorno
- `Map<String, FeatureFlag__mdt>`: Mapa donde la clave es el nombre de la bandera de característica y el valor es el registro de metadatos personalizado correspondiente.

## Dependencias
- `FeatureFlag__mdt`: Tipo de metadatos personalizado que contiene la configuración de las banderas de características.

## Propósito y Aplicaciones

El propósito principal de esta interfaz es proporcionar una abstracción para el acceso a la configuración de banderas de características. Esto permite:

1. **Gestión Centralizada**: Centralizar la lógica de acceso a banderas de características.
2. **Flexibilidad de Implementación**: Permitir diferentes implementaciones según el entorno o las necesidades.
3. **Facilidad de Pruebas**: Facilitar la creación de implementaciones mock para pruebas unitarias.
4. **Desacoplamiento**: Desacoplar la gestión de características del código que las utiliza.

Esta interfaz es particularmente útil en escenarios como:

- **Despliegue Progresivo**: Liberar características a grupos selectos de usuarios antes de un lanzamiento general.
- **Test A/B**: Probar diferentes variantes de una característica con diferentes grupos de usuarios.
- **Configuración por Entorno**: Activar o desactivar características específicas según el entorno (desarrollo, prueba, producción).
- **Interruptores de Emergencia**: Proporcionar la capacidad de desactivar rápidamente características problemáticas sin necesidad de desplegar nuevo código.

## Ejemplos de Implementación

### Implementación Básica

```apex
public class DefaultFeatureFlagProvider implements IFeatureFlagProvider {
    
    public Set<String> getCustomPermissionNames() {
        // Obtener todos los permisos personalizados disponibles en el sistema
        Set<String> permissionNames = new Set<String>();
        
        for (CustomPermission cp : [SELECT Id, DeveloperName FROM CustomPermission]) {
            permissionNames.add(cp.DeveloperName);
        }
        
        return permissionNames;
    }
    
    public Map<String, FeatureFlag__mdt> getFeatureFlags() {
        // Obtener todas las banderas de características desde los metadatos personalizados
        Map<String, FeatureFlag__mdt> featureFlags = new Map<String, FeatureFlag__mdt>();
        
        for (FeatureFlag__mdt flag : [SELECT DeveloperName, MasterLabel, Is_Active__c, 
                                     Description__c, Custom_Permission_Name__c, 
                                     Min_Api_Version__c, Max_Api_Version__c
                                     FROM FeatureFlag__mdt]) {
            featureFlags.put(flag.DeveloperName, flag);
        }
        
        return featureFlags;
    }
}
```

### Implementación con Caché

```apex
public class CachedFeatureFlagProvider implements IFeatureFlagProvider {
    
    private static Set<String> cachedPermissionNames;
    private static Map<String, FeatureFlag__mdt> cachedFeatureFlags;
    private static Datetime lastCacheRefresh;
    private static final Integer CACHE_EXPIRY_MINS = 15;
    
    public Set<String> getCustomPermissionNames() {
        if (shouldRefreshCache()) {
            refreshCache();
        }
        return cachedPermissionNames;
    }
    
    public Map<String, FeatureFlag__mdt> getFeatureFlags() {
        if (shouldRefreshCache()) {
            refreshCache();
        }
        return cachedFeatureFlags;
    }
    
    private Boolean shouldRefreshCache() {
        if (cachedPermissionNames == null || cachedFeatureFlags == null || lastCacheRefresh == null) {
            return true;
        }
        
        Datetime now = Datetime.now();
        Long millisSinceRefresh = now.getTime() - lastCacheRefresh.getTime();
        Long millisToExpiry = CACHE_EXPIRY_MINS * 60 * 1000;
        
        return millisSinceRefresh > millisToExpiry;
    }
    
    private void refreshCache() {
        // Obtener permisos personalizados
        cachedPermissionNames = new Set<String>();
        for (CustomPermission cp : [SELECT DeveloperName FROM CustomPermission]) {
            cachedPermissionNames.add(cp.DeveloperName);
        }
        
        // Obtener banderas de características
        cachedFeatureFlags = new Map<String, FeatureFlag__mdt>();
        for (FeatureFlag__mdt flag : [SELECT DeveloperName, MasterLabel, Is_Active__c, 
                                     Description__c, Custom_Permission_Name__c, 
                                     Min_Api_Version__c, Max_Api_Version__c
                                     FROM FeatureFlag__mdt]) {
            cachedFeatureFlags.put(flag.DeveloperName, flag);
        }
        
        // Actualizar timestamp de refresco
        lastCacheRefresh = Datetime.now();
    }
}
```

### Mock para Pruebas

```apex
@IsTest
public class MockFeatureFlagProvider implements IFeatureFlagProvider {
    
    private Set<String> mockPermissions;
    private Map<String, FeatureFlag__mdt> mockFeatureFlags;
    
    public MockFeatureFlagProvider() {
        // Inicializar con valores predeterminados vacíos
        mockPermissions = new Set<String>();
        mockFeatureFlags = new Map<String, FeatureFlag__mdt>();
    }
    
    public Set<String> getCustomPermissionNames() {
        return mockPermissions;
    }
    
    public Map<String, FeatureFlag__mdt> getFeatureFlags() {
        return mockFeatureFlags;
    }
    
    // Métodos adicionales para configurar el mock para pruebas
    
    public void addCustomPermission(String permissionName) {
        mockPermissions.add(permissionName);
    }
    
    public void setCustomPermissions(Set<String> permissions) {
        mockPermissions = permissions;
    }
    
    public void addFeatureFlag(String flagName, Boolean isActive, String permissionName) {
        FeatureFlag__mdt flag = createMockFeatureFlag(flagName, isActive, permissionName);
        mockFeatureFlags.put(flagName, flag);
    }
    
    public void setFeatureFlags(Map<String, FeatureFlag__mdt> featureFlags) {
        mockFeatureFlags = featureFlags;
    }
    
    private FeatureFlag__mdt createMockFeatureFlag(String name, Boolean isActive, String permissionName) {
        // Crear una instancia de FeatureFlag__mdt utilizando técnicas de prueba
        // Nota: Como los registros de metadatos personalizados no se pueden crear en tiempo de ejecución,
        // se necesitaría utilizar técnicas como la reflexión o una clase wrapper
        
        // Ejemplo simplificado (no funcionará directamente, es ilustrativo)
        FeatureFlag__mdt mockFlag = new FeatureFlag__mdt();
        mockFlag.DeveloperName = name;
        mockFlag.MasterLabel = name;
        mockFlag.Is_Active__c = isActive;
        mockFlag.Custom_Permission_Name__c = permissionName;
        
        return mockFlag;
    }
}
```

## Clase de Servicio de Feature Flags

Esta interfaz normalmente sería utilizada por una clase de servicio que proporciona la lógica para verificar si una característica está habilitada:

```apex
public class FeatureFlagService {
    
    private static FeatureFlagService instance;
    private IFeatureFlagProvider provider;
    
    // Constructor privado para patrón Singleton
    private FeatureFlagService() {
        this.provider = new DefaultFeatureFlagProvider();
    }
    
    // Método para obtener la instancia singleton
    public static FeatureFlagService getInstance() {
        if (instance == null) {
            instance = new FeatureFlagService();
        }
        return instance;
    }
    
    // Método para inyectar un proveedor personalizado (útil para pruebas)
    public void setProvider(IFeatureFlagProvider customProvider) {
        this.provider = customProvider;
    }
    
    // Verificar si una característica está habilitada
    public Boolean isFeatureEnabled(String featureName) {
        Map<String, FeatureFlag__mdt> featureFlags = provider.getFeatureFlags();
        
        // Si la bandera no existe, la característica está deshabilitada
        if (!featureFlags.containsKey(featureName)) {
            return false;
        }
        
        FeatureFlag__mdt flag = featureFlags.get(featureName);
        
        // Si la bandera no está activa, la característica está deshabilitada
        if (!flag.Is_Active__c) {
            return false;
        }
        
        // Si la bandera requiere un permiso personalizado, verificar si el usuario actual lo tiene
        if (String.isNotBlank(flag.Custom_Permission_Name__c)) {
            return FeatureManagement.checkPermission(flag.Custom_Permission_Name__c);
        }
        
        // Si no se requiere permiso personalizado y la bandera está activa, la característica está habilitada
        return true;
    }
    
    // Verificar si una característica está habilitada y cumple con restricciones de versión API
    public Boolean isFeatureEnabledForApiVersion(String featureName, Decimal apiVersion) {
        Map<String, FeatureFlag__mdt> featureFlags = provider.getFeatureFlags();
        
        // Si la bandera no existe, la característica está deshabilitada
        if (!featureFlags.containsKey(featureName)) {
            return false;
        }
        
        FeatureFlag__mdt flag = featureFlags.get(featureName);
        
        // Verificar si la bandera está habilitada
        if (!isFeatureEnabled(featureName)) {
            return false;
        }
        
        // Verificar restricciones de versión API
        if (flag.Min_Api_Version__c != null && apiVersion < flag.Min_Api_Version__c) {
            return false;
        }
        
        if (flag.Max_Api_Version__c != null && apiVersion > flag.Max_Api_Version__c) {
            return false;
        }
        
        return true;
    }
}
```

## Ejemplo de Uso

```apex
// Obtener instancia del servicio
FeatureFlagService featureFlagService = FeatureFlagService.getInstance();

// Verificar si una característica está habilitada
if (featureFlagService.isFeatureEnabled('NewUserInterface')) {
    // Usar la nueva interfaz de usuario
    showNewUI();
} else {
    // Usar la interfaz de usuario clásica
    showClassicUI();
}

// Ejemplo con versión API
Decimal currentApiVersion = 54.0;
if (featureFlagService.isFeatureEnabledForApiVersion('AdvancedSearch', currentApiVersion)) {
    // Usar funcionalidad de búsqueda avanzada
    performAdvancedSearch();
} else {
    // Usar búsqueda estándar
    performStandardSearch();
}
```

## Ejemplo de uso en Pruebas

```apex
@IsTest
private class FeatureFlagServiceTest {
    
    @IsTest
    static void testFeatureEnabled() {
        // Crear un mock del proveedor
        MockFeatureFlagProvider mockProvider = new MockFeatureFlagProvider();
        
        // Configurar el mock para la prueba
        mockProvider.addFeatureFlag('TestFeature', true, null);
        
        // Inyectar el mock en el servicio
        FeatureFlagService service = FeatureFlagService.getInstance();
        service.setProvider(mockProvider);
        
        // Verificar que la característica está habilitada
        System.assertEquals(true, service.isFeatureEnabled('TestFeature'));
    }
    
    @IsTest
    static void testFeatureDisabled() {
        // Crear un mock del proveedor
        MockFeatureFlagProvider mockProvider = new MockFeatureFlagProvider();
        
        // Configurar el mock para la prueba
        mockProvider.addFeatureFlag('TestFeature', false, null);
        
        // Inyectar el mock en el servicio
        FeatureFlagService service = FeatureFlagService.getInstance();
        service.setProvider(mockProvider);
        
        // Verificar que la característica está deshabilitada
        System.assertEquals(false, service.isFeatureEnabled('TestFeature'));
    }
    
    // Otros casos de prueba...
}
```

## Estructura del Objeto Personalizado FeatureFlag__mdt

El tipo de metadatos personalizado `FeatureFlag__mdt` probablemente tendría la siguiente estructura:

| Campo | Tipo | Descripción |
|-------|------|-------------|
| DeveloperName | String | Nombre único de la bandera de característica |
| MasterLabel | String | Etiqueta visible para la bandera |
| Is_Active__c | Boolean | Indica si la característica está activa |
| Description__c | Text | Descripción de la característica |
| Custom_Permission_Name__c | String | Nombre del permiso personalizado requerido (si aplica) |
| Min_Api_Version__c | Number | Versión mínima de API compatible (si aplica) |
| Max_Api_Version__c | Number | Versión máxima de API compatible (si aplica) |

## Mejores Prácticas

1. **Caché Eficiente**: Implementar estrategias de caché para evitar consultas repetitivas a los metadatos.
2. **Manejo de Errores**: Incluir manejo de errores robusto en las implementaciones para evitar que fallos en la configuración de banderas afecten la funcionalidad principal.
3. **Despliegue Controlado**: Utilizar las banderas de características para implementar un despliegue controlado de nuevas funcionalidades.
4. **Documentación**: Mantener documentación clara sobre cada bandera de característica y su propósito.
5. **Limpieza**: Implementar un proceso regular para revisar y eliminar banderas de características obsoletas.

## Consideraciones de Rendimiento

1. **Consultas Optimizadas**: Minimizar el número de consultas SOQL cargando todos los datos necesarios de una vez.
2. **Uso de Caché**: Implementar mecanismos de caché para reducir las consultas a la base de datos.
3. **Tamaño de los Datos**: Ser consciente del número de banderas de características y permisos personalizados, ya que un número grande podría afectar el rendimiento.

## Consideraciones de Seguridad

1. **Control de Acceso**: Asegurar que los permisos personalizados estén configurados correctamente para controlar el acceso a las características.
2. **Validación de Entradas**: Validar los nombres de características antes de utilizarlos en las consultas.
3. **Exposición de Características**: Tener cuidado con la exposición de información sobre características en desarrollo a usuarios no autorizados.

## Notas Adicionales
1. Esta interfaz es parte de un sistema más amplio de gestión de características que probablemente incluye otros componentes como servicios de administración, páginas de configuración, etc.
2. La implementación concreta podría necesitar ajustes dependiendo de los requisitos específicos y la arquitectura del sistema.
3. En sistemas con un gran número de banderas de características, podría ser beneficioso implementar categorías o grupos para una mejor organización.
4. La interfaz podría extenderse para incluir métodos adicionales, como la capacidad de filtrar banderas por categoría o buscar banderas que coincidan con ciertos criterios.