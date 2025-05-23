# Guías de Implementación - SURA Foundation Framework

## Resumen Ejecutivo

Esta guía proporciona instrucciones completas para la implementación, configuración y despliegue de **SURA Foundation Framework**. Incluye desde la instalación inicial hasta la configuración avanzada, mejores prácticas y troubleshooting para garantizar una implementación exitosa.

---

## 🎯 Prerrequisitos

### Requisitos de Ambiente

#### Salesforce Organization
- **Edición**: Enterprise, Unlimited, o Developer Edition
- **API Version**: 58.0 o superior
- **My Domain**: Configurado y desplegado
- **Lightning Experience**: Habilitado

#### Vlocity/OmniStudio
- **OmniStudio**: Versión 2024.1 o superior
- **Industries CPQ**: Instalado y configurado
- **Integration Procedures**: Disponibles
- **Data Raptors**: Configurados

#### Permisos y Licencias
```
┌─────────────────────────────────────────────────────────────────┐
│                    LICENSING REQUIREMENTS                      │
│                                                                 │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐ │
│  │   Salesforce    │  │     Vlocity     │  │    Foundation   │ │
│  │    Licenses     │  │    Licenses     │  │    Permissions  │ │
│  │                 │  │                 │  │                 │ │
│  │ • Platform      │  │ • Industries    │  │ • Process       │ │
│  │ • CRM          │  │   CPQ           │  │   Executor      │ │
│  │ • API Access    │  │ • OmniStudio    │  │ • Config        │ │
│  │                 │  │ • Integration   │  │   Manager       │ │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

### Usuarios y Roles

**Roles Requeridos:**
- **System Administrator**: Instalación y configuración inicial
- **Foundation Administrator**: Gestión de procesos y configuraciones
- **Integration Specialist**: Configuración de servicios externos
- **Developer**: Extensiones y customizaciones

---

## 📦 Instalación del Managed Package

### Paso 1: Descarga e Instalación

#### Desde AppExchange (Recomendado)
```bash
1. Acceder a AppExchange
2. Buscar "SURA Foundation Framework"
3. Hacer clic en "Get It Now"
4. Seleccionar ambiente de destino
5. Confirmar instalación
```

#### Instalación Manual (Sandbox)
```bash
# URL de instalación directa
https://login.salesforce.com/packaging/installPackage.apexp?p0=04t...

# Para Sandbox
https://test.salesforce.com/packaging/installPackage.apexp?p0=04t...
```

#### Package ID por Versión
```
Versión 1.0-1: 04tXXXXXXXXXXXX1
Versión 1.0-3: 04tXXXXXXXXXXXX3
Versión 1.0-4: 04tXXXXXXXXXXXX4
Versión 1.0-5: 04tXXXXXXXXXXXX5
Versión 1.0-6: 04tXXXXXXXXXXXX6 (Actual)
```

### Paso 2: Configuración de Instalación

#### Opciones de Instalación
```
┌─────────────────────────────────────────────────────────────────┐
│                 INSTALLATION OPTIONS                           │
│                                                                 │
│  [x] Install for All Users                                     │
│  [ ] Install for Admins Only                                   │
│  [ ] Install for Specific Profiles                             │
│                                                                 │
│  Security Settings:                                             │
│  [x] Grant access to all data                                  │
│  [x] Enable Apex class access                                  │
│  [x] Enable custom tabs                                        │
│                                                                 │
│  Advanced Options:                                              │
│  [x] Compile all Apex classes                                  │
│  [x] Enable push notifications                                 │
│  [x] Upgrade existing components                               │
└─────────────────────────────────────────────────────────────────┘
```

### Paso 3: Validación de Instalación

#### Script de Validación
```apex
// Ejecutar en Developer Console
System.debug('=== SURA Foundation Installation Validation ===');

// Verificar clases instaladas
Type coreClass = Type.forName('SFSura', 'Core');
System.debug('Core class installed: ' + (coreClass != null));

// Verificar objetos personalizados
Schema.DescribeSObjectResult eventLogDesc = 
    SFFoundation_EventLog__c.SObjectType.getDescribe();
System.debug('EventLog object installed: ' + eventLogDesc.isCreateable());

// Verificar Custom Metadata Types
List<SFFoundation_ProcessConfiguration__mdt> configs = 
    [SELECT Id FROM SFFoundation_ProcessConfiguration__mdt LIMIT 1];
System.debug('Process configurations available: ' + !configs.isEmpty());

System.debug('=== Installation validation complete ===');
```

---

## ⚙️ Configuración Inicial

### Paso 1: Configuración de Metadatos

#### Instalar Configuraciones Básicas
```apex
// Ejecutar desde el Panel de Administración
// Navegación: Setup > Apps > App Manager > SURA Foundation

1. Hacer clic en "Instalar metadatos y configuraciones iniciales"
2. Seleccionar productos a configurar:
   [x] Seguros de Motos
   [x] Seguros de Autos
   [ ] Seguros de Viajes (Beta)
   [ ] Arrendamiento (Beta)
3. Confirmar instalación
```

#### Configuración Manual de Procesos
```sql
-- Custom Metadata Type: SFFoundation_ProcessConfiguration__mdt

INSERT SFFoundation_ProcessConfiguration__mdt (
    DeveloperName = 'Lead_Motos_Conocimiento',
    MasterLabel = 'Lead Motos - Conocimiento',
    ProductFamily__c = 'Seguros de Motos',
    Stage__c = 'Conocimiento',
    HandlerClass__c = 'MT_Kn_123',
    Order__c = 0,
    IsActive__c = true,
    ApiName__c = 'leadMotosKnowing'
);
```

### Paso 2: Feature Flags

#### Configuración Básica de Features
```json
{
  "featureFlags": [
    {
      "developerName": "EnableFoundationLogging",
      "isEnabled": true,
      "environment": "All",
      "scope": "Global",
      "description": "Habilita logging del framework"
    },
    {
      "developerName": "EnableAdvancedValidations",
      "isEnabled": false,
      "environment": "Production",
      "scope": "Global",
      "description": "Validaciones avanzadas de negocio"
    },
    {
      "developerName": "EnableRUNTIntegration",
      "isEnabled": true,
      "environment": "All",
      "scope": "Product",
      "productFamily": "Seguros de Motos",
      "description": "Integración con RUNT para motos"
    }
  ]
}
```

### Paso 3: Configuración de Integraciones

#### RUNT Integration
```apex
// Custom Metadata: SFFoundation_IntegrationConfiguration__mdt

ServiceName__c: 'RUNT_VehicleConsultation'
BaseURL__c: 'https://api.runt.gov.co/v2'
AuthMethod__c: 'API_Key'
TimeoutMs__c: 30000
MaxRetries__c: 3
RetryPattern__c: 'Fibonacci'
RetryIntervalMs__c: 1000
RetryOnHttpCodes__c: '500,502,503,504'
IsActive__c: true
```

#### FASECOLDA Integration
```apex
ServiceName__c: 'FASECOLDA_VehicleData'
BaseURL__c: 'https://api.fasecolda.com/v1'
AuthMethod__c: 'Bearer_Token'
TimeoutMs__c: 20000
MaxRetries__c: 2
RetryPattern__c: 'Incremental'
RetryIntervalMs__c: 500
RetryOnHttpCodes__c: '429,500,502,503'
IsActive__c: true
```

---

## 🔧 Configuración por Ambiente

### Desarrollo (Sandbox)

#### Configuraciones Específicas
```json
{
  "environment": "Development",
  "settings": {
    "logging": {
      "level": "DEBUG",
      "retentionDays": 7
    },
    "integrations": {
      "timeout": 60000,
      "enableMocks": true,
      "bypassValidations": true
    },
    "features": {
      "enableAllFeatures": true,
      "debugMode": true
    }
  }
}
```

#### Mock Services
```apex
// Habilitar servicios simulados para desarrollo
SFFoundation_GlobalSettings__c devSettings = new SFFoundation_GlobalSettings__c();
devSettings.Name = 'Development';
devSettings.IsEnabled__c = true;
devSettings.LogLevel__c = 'DEBUG';
devSettings.DefaultTimeout__c = 60000;
insert devSettings;
```

### Testing (UAT)

#### Configuraciones de Prueba
```json
{
  "environment": "Testing",
  "settings": {
    "logging": {
      "level": "INFO",
      "retentionDays": 30
    },
    "integrations": {
      "timeout": 30000,
      "enableMocks": false,
      "useTestEndpoints": true
    },
    "features": {
      "gradualRollout": true,
      "rolloutPercentage": 50
    }
  }
}
```

### Producción

#### Configuraciones de Producción
```json
{
  "environment": "Production",
  "settings": {
    "logging": {
      "level": "ERROR",
      "retentionDays": 90
    },
    "integrations": {
      "timeout": 30000,
      "enableMocks": false,
      "useProductionEndpoints": true
    },
    "features": {
      "conservativeRollout": true,
      "rolloutPercentage": 10
    }
  }
}
```

---

## 🚀 Despliegue y Migración

### Estrategia de Despliegue

#### Blue-Green Deployment
```
┌─────────────────────────────────────────────────────────────────┐
│                    DEPLOYMENT STRATEGY                         │
│                                                                 │
│  ┌─────────────────┐           ┌─────────────────────────────┐  │
│  │      BLUE       │           │            GREEN            │  │
│  │   (Current)     │           │         (New Version)       │  │
│  │                 │           │                             │  │
│  │ • Version 1.0-5 │    ───▶   │ • Version 1.0-6            │  │
│  │ • 100% Traffic  │           │ • 0% Traffic                │  │
│  │ • Stable        │           │ • Testing                   │  │
│  └─────────────────┘           └─────────────────────────────┘  │
│                                                                 │
│  Phase 1: Deploy Green (0% traffic)                           │
│  Phase 2: Validate Green (smoke tests)                        │
│  Phase 3: Route 10% traffic to Green                          │
│  Phase 4: Monitor and increase to 50%                         │
│  Phase 5: Full cutover to Green (100%)                        │
│  Phase 6: Blue becomes standby                                │
└─────────────────────────────────────────────────────────────────┘
```

#### Rolling Deployment por Productos
```
Week 1: Seguros de Motos (Pilot)
  - Deploy to 10% of users
  - Monitor for 48 hours
  - Scale to 50% if successful

Week 2: Seguros de Autos
  - Same progressive rollout
  - Parallel monitoring

Week 3: Full Production
  - Complete rollout
  - Legacy system sunset
```

### Checklist de Pre-Despliegue

#### Validaciones Técnicas
```
□ Package installation successful
□ All Apex classes compiled
□ Custom objects accessible
□ Metadata types populated
□ Platform events configured
□ Integration endpoints tested
□ Feature flags configured
□ Permissions assigned
□ Test coverage > 75%
□ Performance benchmarks met
```

#### Validaciones de Negocio
```
□ Process configurations validated
□ Business rules tested
□ Integration data validated
□ User acceptance tests passed
□ Training materials updated
□ Support team briefed
□ Rollback plan prepared
□ Communication plan executed
```

---

## 👥 Configuración de Usuarios y Permisos

### Permission Sets

#### SFFoundation_ProcessExecutor
```apex
// Permisos para ejecutar procesos
Custom Permissions:
- SFFoundation_ProcessExecutor: true

Object Permissions:
- SFFoundation_ProcessExecution__c: Create, Read
- SFFoundation_EventLog__c: Create, Read
- Lead: Read, Edit
- Opportunity: Read, Edit
- Quote: Read, Edit

Apex Classes:
- SFSura.Core: Enabled
- SFSura.ProcessManager: Enabled
- SFSura.LeadHandler: Enabled
```

#### SFFoundation_ConfigurationManager
```apex
// Permisos para gestionar configuraciones
Custom Permissions:
- SFFoundation_ConfigurationManager: true

Object Permissions:
- SFFoundation_ProcessConfiguration__mdt: Read
- SFFoundation_FeatureFlag__mdt: Read, Edit
- SFFoundation_IntegrationConfiguration__mdt: Read

Apps:
- SURA Foundation Admin: Visible
```

#### SFFoundation_IntegrationUser
```apex
// Permisos para integraciones
Custom Permissions:
- SFFoundation_IntegrationCaller: true

Object Permissions:
- SFFoundation_IntegrationLog__c: Create, Read
- SFFoundation_EventLog__c: Create, Read

Apex Classes:
- SFSura.ExternalServiceHandler: Enabled
- SFSura.IntegrationManager: Enabled
```

### Asignación de Permisos

#### Por Rol
```apex
// System Administrator
- SFFoundation_ProcessExecutor
- SFFoundation_ConfigurationManager
- SFFoundation_IntegrationUser

// Sales User
- SFFoundation_ProcessExecutor

// Integration User
- SFFoundation_IntegrationUser

// Support User
- SFFoundation_ProcessExecutor (Read-only)
```

---

## 🔗 Configuración de Integraciones

### Named Credentials

#### RUNT Integration
```
Name: RUNT_API_Credential
URL: https://api.runt.gov.co/v2
Identity Type: Named Principal
Authentication Protocol: Password Authentication
Username: ${system_username}
Password: ${system_password}

Custom Headers:
- X-API-Key: ${api_key}
- Content-Type: application/json
```

#### FASECOLDA Integration
```
Name: FASECOLDA_API_Credential
URL: https://api.fasecolda.com/v1
Identity Type: Named Principal
Authentication Protocol: OAuth 2.0
Client ID: ${client_id}
Client Secret: ${client_secret}
Scope: vehicle_data

Custom Headers:
- Accept: application/json
- X-Client-Version: 1.0
```

### Remote Site Settings

```apex
// RUNT
Name: RUNT_API
Remote Site URL: https://api.runt.gov.co
Active: true
Description: RUNT vehicle consultation service

// FASECOLDA
Name: FASECOLDA_API
Remote Site URL: https://api.fasecolda.com
Active: true
Description: FASECOLDA vehicle data service

// Payment Gateway
Name: Payment_Gateway
Remote Site URL: https://payments.sura.com
Active: true
Description: SURA payment processing gateway
```

---

## 📊 Monitoreo y Mantenimiento

### Dashboard de Monitoreo

#### Configuración de Dashboard
```
Dashboard Name: SURA Foundation Monitoring

Components:
1. Process Execution Summary (Last 24h)
   - Total executions
   - Success rate
   - Average duration
   - Error distribution

2. Integration Health
   - Service availability
   - Response times
   - Error rates
   - Retry statistics

3. System Performance
   - CPU usage
   - Memory usage
   - API limits
   - Storage usage

4. Business Metrics
   - Leads processed
   - Quotes generated
   - Policies issued
   - Revenue impact
```

#### Reports de Monitoreo
```sql
-- Process Performance Report
SELECT 
  ProcessName__c,
  COUNT(Id) as TotalExecutions,
  AVG(Duration__c) as AvgDuration,
  COUNT(CASE WHEN Status__c = 'Failed' THEN 1 END) as FailedExecutions
FROM SFFoundation_ProcessExecution__c
WHERE CreatedDate = LAST_N_DAYS:7
GROUP BY ProcessName__c
ORDER BY TotalExecutions DESC

-- Integration Health Report
SELECT 
  ServiceName__c,
  AVG(Duration__c) as AvgResponseTime,
  COUNT(CASE WHEN IsSuccess__c = true THEN 1 END) as SuccessCount,
  COUNT(CASE WHEN IsSuccess__c = false THEN 1 END) as ErrorCount
FROM SFFoundation_IntegrationLog__c
WHERE CreatedDate = TODAY
GROUP BY ServiceName__c
```

### Alertas Automáticas

#### Email Alerts
```apex
// Configurar en Workflow Rules o Process Builder

Alert: High Error Rate
Condition: Error rate > 5% in last hour
Recipients: sysadmin@sura.com, devops@sura.com

Alert: Integration Failure
Condition: Service unavailable > 5 minutes
Recipients: integration-team@sura.com

Alert: Performance Degradation
Condition: Avg response time > 5 seconds
Recipients: performance-team@sura.com
```

### Mantenimiento Regular

#### Tareas Semanales
```
□ Review error logs
□ Check integration health
□ Monitor performance metrics
□ Update feature flag configurations
□ Review capacity utilization
□ Backup configuration metadata
```

#### Tareas Mensuales
```
□ Archive old event logs
□ Review and optimize queries
□ Update integration credentials
□ Performance testing
□ Security review
□ Documentation updates
```

---

## 🔧 Configuración Avanzada

### Custom Settings Avanzadas

#### Performance Tuning
```apex
SFFoundation_GlobalSettings__c perfSettings = new SFFoundation_GlobalSettings__c();
perfSettings.Name = 'PerformanceSettings';
perfSettings.DefaultTimeout__c = 30000;
perfSettings.MaxRetries__c = 3;
perfSettings.CleanupBatchSize__c = 200;
perfSettings.LogLevel__c = 'INFO';

insert perfSettings;
```

#### Configuración de Cache
```json
{
  "cacheSettings": {
    "processConfigurations": {
      "ttl": 3600,
      "maxSize": 1000,
      "enabled": true
    },
    "featureFlags": {
      "ttl": 1800,
      "maxSize": 500,
      "enabled": true
    },
    "integrationConfigs": {
      "ttl": 7200,
      "maxSize": 100,
      "enabled": true
    }
  }
}
```

### Configuración de Batch Jobs

#### Event Log Cleanup
```apex
// Schedule cleanup job
SFFoundation_EventLogCleanupBatch cleanup = new SFFoundation_EventLogCleanupBatch();
String cronExpression = '0 0 2 * * ?'; // Daily at 2 AM
System.schedule('Foundation Event Log Cleanup', cronExpression, cleanup);
```

#### Integration Health Check
```apex
// Schedule health check job
SFFoundation_HealthCheckBatch healthCheck = new SFFoundation_HealthCheckBatch();
String cronExpression = '0 */15 * * * ?'; // Every 15 minutes
System.schedule('Foundation Health Check', cronExpression, healthCheck);
```

---

## 🐛 Troubleshooting

### Problemas Comunes

#### 1. Package Installation Failures
```
Error: "Package installation failed due to missing dependencies"

Solución:
1. Verificar que Vlocity/OmniStudio esté instalado
2. Confirmar versión de API compatible
3. Revisar permisos de usuario instalador
4. Verificar límites de storage
```

#### 2. Process Execution Errors
```
Error: "Handler class not found: MT_Kn_123"

Solución:
1. Verificar configuración en ProcessConfiguration__mdt
2. Confirmar que la clase existe y es accesible
3. Revisar permisos de Apex class access
4. Validar naming convention
```

#### 3. Integration Timeouts
```
Error: "Integration timeout after 30000ms"

Solución:
1. Aumentar timeout en IntegrationConfiguration__mdt
2. Verificar conectividad de red
3. Revisar Named Credentials
4. Contactar proveedor del servicio
```

#### 4. Performance Issues
```
Error: "Process execution taking too long"

Solución:
1. Revisar governor limits usage
2. Optimizar queries SOQL
3. Implementar async processing
4. Revisar bulk operations
```

### Logs de Debug

#### Habilitar Debug Logging
```apex
// Configurar trace flags para debugging
1. Setup > Debug Logs
2. New Trace Flag
3. Traced Entity Type: User
4. Start Date: Today
5. Expiration Date: Tomorrow
6. Debug Level: Foundation_Debug

// Custom Debug Level
Apex Code: DEBUG
Apex Profiling: INFO
Callout: DEBUG
Database: DEBUG
System: DEBUG
Validation: INFO
Visualforce: INFO
Workflow: INFO
```

#### Analizar Logs
```apex
// Buscar patrones en logs
System.debug('=== Foundation Process Start ===');
System.debug('Process: ' + processName);
System.debug('Duration: ' + duration + 'ms');
System.debug('=== Foundation Process End ===');

// Filtrar en Developer Console:
// USER_DEBUG|Foundation
```

---

## 📋 Checklist de Go-Live

### Pre-Go-Live (1 semana antes)

#### Validaciones Técnicas
```
□ All environments synchronized
□ Production data validated
□ Performance benchmarks met
□ Security scan completed
□ Backup procedures tested
□ Rollback plan validated
□ Monitoring alerts configured
□ Load testing completed
```

#### Validaciones de Negocio
```
□ User acceptance testing passed
□ Business process validation
□ Data migration completed
□ Training sessions conducted
□ Support documentation ready
□ Communication plan executed
□ Change management approved
□ Go/No-Go decision made
```

### Go-Live Day

#### Deployment Sequence
```
Hour 0: Deploy to production
Hour 1: Smoke tests
Hour 2: Enable for pilot users (10%)
Hour 4: Monitor and validate
Hour 6: Scale to 50% if successful
Hour 12: Full deployment if stable
Hour 24: Post-deployment review
```

#### Monitoring Checklist
```
□ System health dashboard green
□ All integrations responding
□ Error rates within acceptable limits
□ Performance metrics normal
□ User feedback positive
□ No critical issues reported
□ Support team ready
□ Escalation procedures active
```

### Post-Go-Live (1 semana después)

#### Validation Activities
```
□ Review all metrics and KPIs
□ Collect user feedback
□ Document lessons learned
□ Optimize based on real usage
□ Plan next iteration
□ Update documentation
□ Schedule regular reviews
□ Celebrate success! 🎉
```

---

## 📚 Recursos Adicionales

### Documentación Técnica

- **Arquitectura General**: Diagramas y patrones implementados
- **Modelo de Datos**: Objetos, relaciones y estructura
- **APIs y Endpoints**: Documentación completa de servicios
- **Catálogo de Clases**: Todas las clases documentadas
- **Alcance y Limitaciones**: Cobertura del framework

### Herramientas de Desarrollo

```
- Salesforce CLI (sfdx)
- VS Code con Salesforce Extensions
- Postman Collection para APIs
- Workbench para administración
- Data Loader para migraciones
```

### Contactos de Soporte

```
Arquitectura: Soulberto Lorenzo <soulberto@cloudblue.us>
Desarrollo: Jean Carlos Melendez <jean@cloudblue.us>
Soporte: foundation-support@sura.com
Documentación: docs@sura-foundation.com
```

---

**Fecha de Actualización:** Mayo 2025  
**Versión de Implementación:** 1.0  
**Mantenido por:** Equipo de Implementación SURA Foundation  
**Estado:** Documento Vivo - Actualización Continua