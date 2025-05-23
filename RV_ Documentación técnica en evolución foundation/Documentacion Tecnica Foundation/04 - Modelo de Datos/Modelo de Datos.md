# Modelo de Datos - SURA Foundation Framework

## Resumen Ejecutivo

El modelo de datos de **SURA Foundation Framework** está diseñado para soportar la arquitectura modular y escalable del sistema. Se basa en una combinación de **Custom Objects** para datos transaccionales, **Custom Metadata Types** para configuración, y **Platform Events** para comunicación asíncrona entre componentes.

---

## 🏗️ Arquitectura de Datos

### Principios de Diseño

1. **Separación de Responsabilidades**: Datos transaccionales vs configuración
2. **Flexibilidad**: Configuración sin deployments mediante metadata
3. **Trazabilidad**: Log completo de eventos y transacciones
4. **Escalabilidad**: Modelo preparado para múltiples productos
5. **Integración**: Compatibilidad con objetos estándar de Salesforce/Vlocity

```
┌─────────────────────────────────────────────────────────────────┐
│                    FOUNDATION DATA MODEL                       │
│                                                                 │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐ │
│  │   TRANSACTION   │  │  CONFIGURATION  │  │     EVENTS      │ │
│  │     DATA        │  │     DATA        │  │     DATA        │ │
│  │                 │  │                 │  │                 │ │
│  │ • Custom Objects│  │ • Custom        │  │ • Platform      │ │
│  │ • Standard      │  │   Metadata      │  │   Events        │ │
│  │   Objects       │  │ • Custom        │  │ • Event Logs    │ │
│  │ • Vlocity       │  │   Settings      │  │                 │ │
│  │   Objects       │  │                 │  │                 │ │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📊 Custom Objects Principales

### 1. SFFoundation_EventLog__c

**Propósito**: Registro centralizado de eventos del framework

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `Name` | Auto Number | Identificador único del evento (EL-{0000}) |
| `EventType__c` | Picklist | Tipo de evento (Process, Integration, Error, etc.) |
| `ProcessName__c` | Text(255) | Nombre del proceso que generó el evento |
| `ProductFamily__c` | Picklist | Familia de producto (Motos, Autos, Viajes, etc.) |
| `Stage__c` | Picklist | Etapa del proceso (Conocimiento, Tarificación, etc.) |
| `Status__c` | Picklist | Estado del evento (Success, Error, Warning, Info) |
| `Message__c` | Long Text | Mensaje detallado del evento |
| `StackTrace__c` | Long Text | Stack trace en caso de errores |
| `RecordId__c` | Text(18) | ID del registro relacionado |
| `UserId__c` | Lookup(User) | Usuario que ejecutó la acción |
| `SessionId__c` | Text(255) | ID de sesión para trazabilidad |
| `ExecutionTime__c` | Number(10,2) | Tiempo de ejecución en milisegundos |
| `Payload__c` | Long Text | Datos de entrada/salida (JSON) |
| `CreatedDate` | DateTime | Fecha y hora de creación |

**Índices:**
- `EventType__c, CreatedDate`
- `ProcessName__c, Status__c`
- `ProductFamily__c, Stage__c`

### 2. SFFoundation_ProcessExecution__c

**Propósito**: Seguimiento de ejecuciones de procesos

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `Name` | Auto Number | Número de ejecución (PE-{0000}) |
| `ProcessName__c` | Text(255) | Nombre del proceso ejecutado |
| `ProductFamily__c` | Picklist | Familia de producto |
| `HandlerClass__c` | Text(255) | Clase handler ejecutada |
| `Stage__c` | Picklist | Etapa del proceso |
| `Phase__c` | Picklist | Fase (Pre, In, Pos) |
| `Status__c` | Picklist | Estado (Running, Completed, Failed) |
| `StartTime__c` | DateTime | Hora de inicio |
| `EndTime__c` | DateTime | Hora de finalización |
| `Duration__c` | Number(10,2) | Duración en milisegundos |
| `InputParameters__c` | Long Text | Parámetros de entrada (JSON) |
| `OutputParameters__c` | Long Text | Parámetros de salida (JSON) |
| `ErrorMessage__c` | Long Text | Mensaje de error si aplica |
| `ParentExecution__c` | Lookup(SFFoundation_ProcessExecution__c) | Ejecución padre |
| `LeadId__c` | Lookup(Lead) | Lead relacionado |
| `OpportunityId__c` | Lookup(Opportunity) | Oportunidad relacionada |
| `QuoteId__c` | Lookup(Quote) | Cotización relacionada |
| `PolicyId__c` | Text(255) | ID de póliza externa |

### 3. SFFoundation_IntegrationLog__c

**Propósito**: Log de integraciones con servicios externos

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `Name` | Auto Number | Número de integración (IL-{0000}) |
| `ServiceName__c` | Text(255) | Nombre del servicio integrado |
| `Endpoint__c` | Text(255) | URL del endpoint |
| `HttpMethod__c` | Picklist | Método HTTP (GET, POST, PUT, DELETE) |
| `RequestHeaders__c` | Long Text | Headers de la request (JSON) |
| `RequestBody__c` | Long Text | Cuerpo de la request |
| `ResponseStatus__c` | Number(3,0) | Código de estado HTTP |
| `ResponseHeaders__c` | Long Text | Headers de la response (JSON) |
| `ResponseBody__c` | Long Text | Cuerpo de la response |
| `RequestTime__c` | DateTime | Hora de envío |
| `ResponseTime__c` | DateTime | Hora de respuesta |
| `Duration__c` | Number(10,2) | Duración en milisegundos |
| `RetryAttempt__c` | Number(2,0) | Número de intento |
| `MaxRetries__c` | Number(2,0) | Máximo número de reintentos |
| `IsSuccess__c` | Checkbox | Indica si fue exitosa |
| `ErrorCode__c` | Text(50) | Código de error interno |
| `ErrorMessage__c` | Text(255) | Mensaje de error |
| `ProcessExecutionId__c` | Lookup(SFFoundation_ProcessExecution__c) | Ejecución relacionada |

---

## ⚙️ Custom Metadata Types

### 1. SFFoundation_ProcessConfiguration__mdt

**Propósito**: Configuración de procesos por producto y etapa

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `DeveloperName` | Text(40) | Nombre único del proceso |
| `MasterLabel` | Text(40) | Etiqueta del proceso |
| `ProductFamily__c` | Text(255) | Familia de producto |
| `Stage__c` | Text(255) | Etapa del proceso |
| `HandlerClass__c` | Text(255) | Clase Apex del handler |
| `Order__c` | Number(3,0) | Orden de ejecución |
| `IsActive__c` | Checkbox | Indica si está activo |
| `Description__c` | Text(255) | Descripción del proceso |
| `ApiName__c` | Text(255) | Nombre de API para referencias |
| `ConfigurationJSON__c` | Long Text | Configuración adicional (JSON) |

**Ejemplo de registros:**
```
Lead_Motos_Conocimiento:
  ProductFamily__c: "Seguros de Motos"
  Stage__c: "Conocimiento"
  HandlerClass__c: "MT_Kn_123"
  Order__c: 0
  IsActive__c: true

Rate_Autos_Tarificacion:
  ProductFamily__c: "Seguros de Autos"
  Stage__c: "Tarificación"
  HandlerClass__c: "AU_Rt_456"
  Order__c: 0
  IsActive__c: true
```

### 2. SFFoundation_FeatureFlag__mdt

**Propósito**: Control de funcionalidades por ambiente

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `DeveloperName` | Text(40) | Nombre único del feature |
| `MasterLabel` | Text(40) | Etiqueta del feature |
| `IsEnabled__c` | Checkbox | Indica si está habilitado |
| `Environment__c` | Text(50) | Ambiente (Sandbox, Production, etc.) |
| `Scope__c` | Picklist | Alcance (Global, Product, Process) |
| `ProductFamily__c` | Text(255) | Familia de producto específica |
| `Description__c` | Text(255) | Descripción de la funcionalidad |
| `EffectiveDate__c` | Date | Fecha de activación |
| `ExpirationDate__c` | Date | Fecha de expiración |
| `ConfigurationJSON__c` | Long Text | Configuración específica (JSON) |

### 3. SFFoundation_IntegrationConfiguration__mdt

**Propósito**: Configuración de integraciones externas

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `DeveloperName` | Text(40) | Nombre único de la integración |
| `MasterLabel` | Text(40) | Etiqueta de la integración |
| `ServiceName__c` | Text(255) | Nombre del servicio |
| `BaseURL__c` | Text(255) | URL base del servicio |
| `AuthMethod__c` | Picklist | Método de autenticación |
| `TimeoutMs__c` | Number(5,0) | Timeout en milisegundos |
| `MaxRetries__c` | Number(2,0) | Máximo número de reintentos |
| `RetryPattern__c` | Picklist | Patrón de reintento (Fixed, Incremental, Fibonacci) |
| `RetryIntervalMs__c` | Number(4,0) | Intervalo base de reintento |
| `RetryOnHttpCodes__c` | Text(255) | Códigos HTTP que activan reintento |
| `IsActive__c` | Checkbox | Indica si está activa |
| `Description__c` | Text(255) | Descripción de la integración |

### 4. SFFoundation_EventLogConfiguration__mdt

**Propósito**: Configuración del sistema de logging

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `DeveloperName` | Text(40) | Nombre único de configuración |
| `MasterLabel` | Text(40) | Etiqueta de configuración |
| `LogLevel__c` | Picklist | Nivel de log (ERROR, WARN, INFO, DEBUG) |
| `Category__c` | Text(255) | Categoría de eventos |
| `IsEnabled__c` | Checkbox | Indica si está habilitado |
| `RetentionDays__c` | Number(3,0) | Días de retención de logs |
| `MaxRecordsPerBatch__c` | Number(4,0) | Máximo registros por lote |
| `CleanupSchedule__c` | Text(255) | Programación de limpieza (Cron) |
| `NotificationEmail__c` | Email | Email para notificaciones críticas |

---

## 📡 Platform Events

### 1. SFFoundation_ProcessEvent__e

**Propósito**: Comunicación de eventos entre procesos

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `EventType__c` | Text(255) | Tipo de evento |
| `ProcessName__c` | Text(255) | Nombre del proceso |
| `ProductFamily__c` | Text(255) | Familia de producto |
| `Stage__c` | Text(255) | Etapa del proceso |
| `Phase__c` | Text(50) | Fase (Pre, In, Pos) |
| `Status__c` | Text(50) | Estado del evento |
| `RecordId__c` | Text(18) | ID del registro relacionado |
| `UserId__c` | Text(18) | ID del usuario |
| `SessionId__c` | Text(255) | ID de sesión |
| `Payload__c` | Long Text | Datos del evento (JSON) |
| `Timestamp__c` | DateTime | Timestamp del evento |

### 2. SFFoundation_IntegrationEvent__e

**Propósito**: Eventos de integraciones con servicios externos

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `ServiceName__c` | Text(255) | Nombre del servicio |
| `EventType__c` | Text(100) | Tipo de evento (Request, Response, Error) |
| `Status__c` | Text(50) | Estado de la integración |
| `HttpStatus__c` | Number(3,0) | Código de estado HTTP |
| `Duration__c` | Number(10,2) | Duración en milisegundos |
| `RecordId__c` | Text(18) | ID del registro relacionado |
| `RequestId__c` | Text(255) | ID único de la request |
| `ErrorCode__c` | Text(50) | Código de error |
| `ErrorMessage__c` | Text(255) | Mensaje de error |
| `Payload__c` | Long Text | Datos de la integración (JSON) |

### 3. SFFoundation_NotificationEvent__e

**Propósito**: Notificaciones del sistema

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `NotificationType__c` | Text(100) | Tipo de notificación |
| `Severity__c` | Text(50) | Severidad (Critical, High, Medium, Low) |
| `Title__c` | Text(255) | Título de la notificación |
| `Message__c` | Long Text | Mensaje de la notificación |
| `RecipientId__c` | Text(18) | ID del destinatario |
| `RecipientType__c` | Text(50) | Tipo de destinatario (User, Queue, Group) |
| `RecordId__c` | Text(18) | ID del registro relacionado |
| `Category__c` | Text(100) | Categoría de la notificación |
| `ExpirationDate__c` | DateTime | Fecha de expiración |
| `IsRead__c` | Checkbox | Indica si fue leída |

---

## 🔗 Relaciones y Dependencias

### Diagrama de Entidades Principales

```
┌─────────────────────────────────────────────────────────────────┐
│                    ENTITY RELATIONSHIP DIAGRAM                 │
│                                                                 │
│  ┌─────────────┐     1:N     ┌─────────────────────────────┐   │
│  │    Lead     │────────────▶│  SFFoundation_ProcessExecution│   │
│  │ (Standard)  │             │           __c               │   │
│  └─────────────┘             └─────────────────────────────┘   │
│         │                                   │                  │
│         │                                   │ 1:N              │
│         │                                   ▼                  │
│         │                     ┌─────────────────────────────┐   │
│         │                     │   SFFoundation_EventLog     │   │
│         │                     │           __c               │   │
│         │                     └─────────────────────────────┘   │
│         │                                   ▲                  │
│         │                                   │ 1:N              │
│         │                     ┌─────────────────────────────┐   │
│         │                     │ SFFoundation_IntegrationLog │   │
│         │                     │           __c               │   │
│         │                     └─────────────────────────────┘   │
│         │                                                      │
│         │ 1:N                                                  │
│         ▼                                                      │
│  ┌─────────────┐     1:N     ┌─────────────────────────────┐   │
│  │Opportunity  │────────────▶│          Quote              │   │
│  │ (Standard)  │             │       (Standard)            │   │
│  └─────────────┘             └─────────────────────────────┘   │
│         │                                   │                  │
│         │ 1:N                               │ 1:N              │
│         ▼                                   ▼                  │
│  ┌─────────────┐                   ┌─────────────────────────┐ │
│  │InsurancePolicy                  │    Asset                │ │
│  │ (Vlocity)   │                   │  (Standard)             │ │
│  └─────────────┘                   └─────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

### Flujo de Datos por Proceso

#### 1. Conocimiento (Lead Management)
```
Lead Creation ──┬──> ProcessExecution__c
                │
                ├──> EventLog__c (Lead Created)
                │
                ├──> IntegrationLog__c (RUNT/FASECOLDA)
                │
                └──> Platform Event (Lead_Process_Complete)
```

#### 2. Tarificación (Rating)
```
Rate Request ───┬──> ProcessExecution__c
                │
                ├──> EventLog__c (Rating Started)
                │
                ├──> IntegrationLog__c (External Rating Service)
                │
                ├──> EventLog__c (Rating Completed)
                │
                └──> Platform Event (Rating_Complete)
```

#### 3. Cotización (Quoting)
```
Quote Creation ─┬──> ProcessExecution__c
                │
                ├──> EventLog__c (Quote Generated)
                │
                ├──> IntegrationLog__c (Payment Gateway)
                │
                └──> Platform Event (Quote_Ready)
```

#### 4. Emisión (Issuance)
```
Policy Issuance ┬──> ProcessExecution__c
                │
                ├──> EventLog__c (Policy Issued)
                │
                ├──> IntegrationLog__c (Core Insurance System)
                │
                └──> Platform Event (Policy_Issued)
```

---

## 📋 Custom Settings

### 1. SFFoundation_GlobalSettings__c

**Propósito**: Configuraciones globales del framework

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `Name` | Text(38) | Nombre del setting |
| `IsEnabled__c` | Checkbox | Indica si está habilitado |
| `LogLevel__c` | Text(10) | Nivel de log global |
| `DefaultTimeout__c` | Number(5,0) | Timeout por defecto |
| `MaxRetries__c` | Number(2,0) | Reintentos por defecto |
| `CleanupBatchSize__c` | Number(4,0) | Tamaño de lote para limpieza |
| `NotificationEmail__c` | Email | Email para notificaciones |

### 2. SFFoundation_ProductSettings__c

**Propósito**: Configuraciones específicas por producto

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `Name` | Text(38) | Nombre del producto |
| `IsActive__c` | Checkbox | Indica si está activo |
| `DefaultHandler__c` | Text(255) | Handler por defecto |
| `ConfigurationJSON__c` | Long Text | Configuración específica (JSON) |
| `ValidationRules__c` | Long Text | Reglas de validación (JSON) |

---

## 🔍 Índices y Performance

### Estrategia de Indexación

#### Objetos de Alto Volumen

**SFFoundation_EventLog__c:**
- Índice compuesto: `(EventType__c, CreatedDate)`
- Índice compuesto: `(ProcessName__c, Status__c)`
- Índice simple: `CreatedDate` (para cleanup)

**SFFoundation_ProcessExecution__c:**
- Índice compuesto: `(ProcessName__c, Status__c, StartTime__c)`
- Índice simple: `LeadId__c`
- Índice simple: `OpportunityId__c`

**SFFoundation_IntegrationLog__c:**
- Índice compuesto: `(ServiceName__c, RequestTime__c)`
- Índice simple: `IsSuccess__c`
- Índice simple: `ProcessExecutionId__c`

### Estrategias de Archivado

```
┌─────────────────────────────────────────────────────────────────┐
│                    DATA LIFECYCLE MANAGEMENT                   │
│                                                                 │
│  ┌─────────────┐    30 days    ┌─────────────┐    90 days      │
│  │   Active    │──────────────▶│   Archive   │─────────────▶   │
│  │    Data     │               │    Data     │               │  │
│  └─────────────┘               └─────────────┘               │  │
│         │                             │                     │  │
│         │ Real-time                   │ Batch               │  │
│         │ Access                      │ Access              │  │
│         ▼                             ▼                     ▼  │
│  ┌─────────────┐               ┌─────────────┐      ┌─────────┐ │
│  │ Dashboard   │               │  Reports    │      │ Purge   │ │
│  │   & APIs    │               │ & Analytics │      │         │ │
│  └─────────────┘               └─────────────┘      └─────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📊 Ejemplo de Datos de Configuración

### Configuración de Procesos

```json
{
  "processes": [
    {
      "developerName": "Lead_Motos_Conocimiento",
      "productFamily": "Seguros de Motos",
      "stage": "Conocimiento",
      "handlerClass": "MT_Kn_123",
      "order": 0,
      "isActive": true,
      "configuration": {
        "validateRUNT": true,
        "requireFasecolda": true,
        "mandatoryFields": ["VehicleType__c", "Model__c", "Year__c"]
      }
    },
    {
      "developerName": "Rate_Motos_Tarificacion",
      "productFamily": "Seguros de Motos",
      "stage": "Tarificación",
      "handlerClass": "MT_Rt_456",
      "order": 0,
      "isActive": true,
      "configuration": {
        "externalRatingService": "SURA_Rating_API",
        "fallbackMethod": "internal",
        "cacheResults": true
      }
    }
  ]
}
```

### Configuración de Feature Flags

```json
{
  "featureFlags": [
    {
      "developerName": "EnableNewRatingEngine",
      "isEnabled": false,
      "environment": "Production",
      "scope": "Global",
      "effectiveDate": "2025-06-01",
      "description": "Habilita el nuevo motor de tarificación"
    },
    {
      "developerName": "EnableAdvancedLogging",
      "isEnabled": true,
      "environment": "Sandbox",
      "scope": "Product",
      "productFamily": "Seguros de Motos",
      "description": "Logging avanzado para debugging"
    }
  ]
}
```

---

## 🚀 Consideraciones de Implementación

### Mejores Prácticas

1. **Naming Conventions**:
    - Custom Objects: `SFFoundation_[ObjectName]__c`
    - Custom Fields: `[FieldName]__c`
    - Metadata Types: `SFFoundation_[TypeName]__mdt`
    - Platform Events: `SFFoundation_[EventName]__e`

2. **Data Governance**:
    - Configuración centralizada via Custom Metadata
    - Logs con retención automática
    - Validaciones a nivel de campo y objeto

3. **Security**:
    - Field-level security apropiada
    - Sharing rules para data isolation
    - Encryption para campos sensibles

4. **Monitoring**:
    - Dashboards para métricas key
    - Alertas automáticas para errores
    - Reports de performance

### Límites y Consideraciones

- **Governor Limits**: Considerar límites de DML, SOQL y heap
- **Storage**: Monitorear uso de data storage
- **API Limits**: Gestionar callouts hacia servicios externos
- **Platform Events**: Respetar límites de publicación

---

**Fecha de Actualización:** Mayo 2025  
**Versión del Documento:** 1.0  
**Arquitectos de Datos:** Soulberto Lorenzo, Jean Carlos Melendez  
**Estado:** Documento Vivo - Actualización Continua