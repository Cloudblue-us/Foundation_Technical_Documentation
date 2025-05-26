# FeatureFlagProvider

## Descripción General
Clase que implementa la interfaz `IFeatureFlagProvider` para proporcionar funcionalidad de gestión de feature flags y permisos personalizados en Salesforce. Permite obtener información sobre permisos personalizados y feature flags configurados en el sistema.

## Información de la Clase
- **Tipo:** Clase pública con sharing
- **Implementa:** IFeatureFlagProvider
- **Propósito:** Proveedor de feature flags y permisos personalizados

## Métodos

### getCustomPermissionNames
```apex
public Set<String> getCustomPermissionNames()
```

**Descripción:** Obtiene todos los nombres de desarrollador de los permisos personalizados disponibles en la organización.

**Parámetros:** Ninguno

**Retorna:**
- `Set<String>`: Conjunto de nombres de desarrollador de permisos personalizados

**Comportamiento:**
1. Crea un conjunto vacío para almacenar los nombres de permisos
2. Ejecuta una consulta SOQL para obtener todos los permisos personalizados
3. Itera sobre los resultados y agrega el `DeveloperName` de cada permiso al conjunto
4. Retorna el conjunto completo de nombres

**Consulta SOQL utilizada:**
```sql
SELECT Id, DeveloperName FROM CustomPermission
```

### getFeatureFlags
```apex
public Map<String,FeatureFlag__mdt> getFeatureFlags()
```

**Descripción:** Obtiene todos los feature flags configurados como Custom Metadata Types.

**Parámetros:** Ninguno

**Retorna:**
- `Map<String,FeatureFlag__mdt>`: Mapa con todos los feature flags donde la clave es el nombre del registro y el valor es el objeto metadata

**Comportamiento:**
- Utiliza el método estático `getAll()` de Custom Metadata Types para obtener todos los registros de `FeatureFlag__mdt`
- Retorna directamente el mapa completo sin filtrado

## Funcionalidad Principal

### Gestión de Permisos Personalizados
- Proporciona acceso programático a todos los permisos personalizados de la organización
- Facilita la validación de permisos en tiempo de ejecución
- Permite implementar lógica condicional basada en permisos

### Gestión de Feature Flags
- Acceso centralizado a los feature flags configurados
- Utiliza Custom Metadata Types para configuración sin código
- Permite activar/desactivar funcionalidades dinámicamente

## Consideraciones de Diseño

### Sharing Settings
- Declarada como `with sharing` para respetar las reglas de seguridad del usuario actual
- Asegura que las consultas respeten los permisos de acceso a datos

### Performance
- `getCustomPermissionNames()`: Ejecuta consulta SOQL, considerar caché si se llama frecuentemente
- `getFeatureFlags()`: Utiliza Custom Metadata Types que están cacheados automáticamente

### Dependencias
- Requiere Custom Metadata Type `FeatureFlag__mdt`
- Depende de la existencia de Custom Permissions en la organización
- Implementa interfaz `IFeatureFlagProvider`

## Casos de Uso
1. **Validación de Permisos:** Verificar si un usuario tiene permisos específicos antes de ejecutar operaciones
2. **Feature Toggling:** Activar/desactivar funcionalidades basado en feature flags
3. **Configuración Dinámica:** Cambiar comportamiento de la aplicación sin despliegues de código
4. **Control de Acceso:** Implementar lógica de autorización granular

## Mejoras Potenciales
- Implementar caché para `getCustomPermissionNames()` si se requiere alto rendimiento
- Agregar métodos para verificar permisos específicos de usuarios
- Incluir logging para auditoría de acceso a feature flags