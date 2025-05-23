# Arquitectura General - SURA Foundation Framework

## Resumen Ejecutivo

**SURA Foundation** es un framework de procesamiento centralizado diseñado como un **Managed Package** de Salesforce que proporciona una arquitectura modular y escalable para los procesos de venta de seguros. El framework implementa patrones de diseño enterprise para garantizar la separación lógica, reutilización de componentes y facilitar el mantenimiento de múltiples productos de seguros.

---

## 🏗️ Visión Arquitectónica

### Concepto Central: Suratech Core

El corazón del framework es **Suratech Core**, un managed package que encapsula:
- Lógica de negocio común a todos los productos
- Patrones de procesamiento estandarizados
- Interfaces de integración con sistemas externos
- Componentes reutilizables para el ciclo de vida de ventas

```
┌─────────────────────────────────────────────────────────────┐
│                    SURATECH CORE                           │
│                 (Managed Package)                          │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌────────┐ │
│  │   MOTOS     │ │    AUTOS    │ │   VIAJES    │ │  LEASE │ │
│  │   Handler   │ │   Handler   │ │   Handler   │ │Handler │ │
│  └─────────────┘ └─────────────┘ └─────────────┘ └────────┘ │
└─────────────────────────────────────────────────────────────┘
```

---

## 🔄 Evolución Arquitectónica

### Arquitectura Actual (Problemática)

**Problemas Identificados:**
- Código híbrido compartido entre múltiples productos
- Lógica mezclada sin separación clara
- Modificaciones por diferentes células sin control
- Imposibilidad de medir desempeño individual
- Difícil mantenimiento y extensibilidad
- Responsabilidades no definidas por producto

```
┌───────────────────────────────────────────────────────────┐
│        ARQUITECTURA MONOLÍTICA ACTUAL                    │
│                                                           │
│  ◆────▢────◆────▢────◆────▢                             │
│   │    │    │    │    │    │                             │
│   ▼    ▲    ▼    ▲    ▼    ▲                             │
│  ▢────▢────▢────▢────▢────▢                              │
│                                                           │
│  ⚠️ Código compartido y mezclado                          │
│  ⚠️ Sin separación de responsabilidades                   │
└───────────────────────────────────────────────────────────┘
```

### Nueva Arquitectura (Solución)

**Beneficios del Nuevo Enfoque:**
- Separación lógica clara por proceso y producto
- Componentes encapsulados y reutilizables
- Facilidad para agregar nuevos productos
- Medición individual de desempeño
- Versionado y distribución controlada

```
┌─────────────────────────────────────────────────────────────┐
│                 NUEVA ARQUITECTURA                          │
│                                                             │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐     │
│  │   PROCESO   │    │   PROCESO   │    │   PROCESO   │     │
│  │   MOTOS     │    │    AUTOS    │    │   VIAJES    │     │
│  │             │    │             │    │             │     │
│  └─────────────┘    └─────────────┘    └─────────────┘     │
│          │                  │                  │           │
│          └──────────────────┼──────────────────┘           │
│                             │                              │
│           ┌─────────────────▼─────────────────┐            │
│           │         SURATECH CORE             │            │
│           │      (Managed Package)            │            │
│           └───────────────────────────────────┘            │
└─────────────────────────────────────────────────────────────┘
```

---

## 🔧 Estructura de Procesamiento

### Modelo Pre-In-Pos

El framework implementa un modelo de procesamiento en tres fases para cada etapa del flujo de ventas:

```
┌─────────────────────────────────────────────────────────────────┐
│                    FLUJO DE PROCESAMIENTO                       │
│                                                                 │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐                     │
│  │   PRE   │    │   IN    │    │   POS   │                     │
│  │         │    │         │    │         │                     │
│  │ ◆────▢  │    │ ◆────▢  │    │ ◆────▢  │                     │
│  │  │   │  │    │  │   │  │    │  │   │  │                     │
│  │  ▼   ▲  │    │  ▼   ▲  │    │  ▼   ▲  │                     │
│  │ ▢────▢  │    │ ▢────▢  │    │ ▢────▢  │                     │
│  └─────────┘    └─────────┘    └─────────┘                     │
│       │              │              │                         │
│       ▼              ▼              ▼                         │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐                     │
│  │Observer │    │Observer │    │Observer │                     │
│  └─────────┘    └─────────┘    └─────────┘                     │
│       │              │              │                         │
│       ▼              ▼              ▼                         │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐                     │
│  │Platform │    │Platform │    │Platform │                     │
│  │ Event   │    │ Event   │    │ Event   │                     │
│  └─────────┘    └─────────┘    └─────────┘                     │
│                       │                                        │
│                       ▼                                        │
│                ┌─────────────┐                                 │
│                │Service      │                                 │
│                │Callout      │                                 │
│                └─────────────┘                                 │
└─────────────────────────────────────────────────────────────────┘
```

#### Fases de Procesamiento:

1. **PRE**: Validaciones y preparación de datos
2. **IN**: Lógica de negocio principal
3. **POS**: Notificaciones y procesos posteriores

#### Componentes de Integración:

- **Observer**: Monitoreo de eventos en cada fase
- **Platform Events**: Comunicación asíncrona entre componentes
- **Service Callout**: Integración con servicios externos

---

## 📦 Arquitectura de Componentes

### Artefactos Principales por Proceso

#### 1. Conocimiento (Lead Management)
```
┌─────────────────────────┐
│     CONOCIMIENTO        │
├─────────────────────────┤
│ • LeadManager           │
│ • SURA_ConsultarVehiculo│
│ • SURA_ConsultaFasecolda│
│ • SURA_RiesgosConsultables│
└─────────────────────────┘
```

#### 2. Tarificación (Pricing)
```
┌─────────────────────────┐
│     TARIFICACIÓN        │
├─────────────────────────┤
│ • RateProductHandler    │
│ • PricingEngine         │
│ • ValidationComponents  │
└─────────────────────────┘
```

#### 3. Cotización (Quoting)
```
┌─────────────────────────┐
│      COTIZACIÓN         │
├─────────────────────────┤
│ • SURACrearNumeroPrepoliza│
│ • SURAQuotePricingAdjustmentTrigger│
│ • SURAPaymentManager    │
│ • QuoteManager          │
│ • PAGOS_GenerarUrlPago  │
│ • CCM_EnviarBienvenida  │
│ • PAGOS_ValidacionSARLAFT│
└─────────────────────────┘
```

#### 4. Emisión (Issuance)
```
┌─────────────────────────┐
│       EMISIÓN           │
├─────────────────────────┤
│ • SURANotificatorToExternalServices│
│ • SURAHomologacionPreEmision│
│ • InsurancePolicyManager│
│ • SURAAccountManager    │
│ • SURAOpportunityManager│
└─────────────────────────┘
```

#### 5. Legalización (Legalization)
```
┌─────────────────────────┐
│     LEGALIZACIÓN        │
├─────────────────────────┤
│ • SURAInvoiceListeningServiceV1│
│ • InsurancePolicyTransactionServicesV1│
│ • InsurancePolicyTransactionManager│
└─────────────────────────┘
```

---

## 🎯 Patrones de Diseño Implementados

### 1. Chain of Responsibility (Cadena de Responsabilidad)
```
Request ──┬──> [Motos Handler] ──┬──> [Autos Handler] ──┬──> [Viajes Handler]
          │                     │                     │
          ▼                     ▼                     ▼
       ☹️ Reject              ☹️ Reject              ✅ Process
```

**Aplicación:**
- Procesamiento secuencial por tipo de producto
- Cada handler determina si puede procesar la solicitud
- Si no puede, la pasa al siguiente en la cadena

### 2. Observer Pattern (Observador)
```
┌─────────────┐    notify    ┌─────────────┐
│   Subject   │ ─────────────>│  Observer   │
│ (Process)   │              │ (Listener)  │
└─────────────┘              └─────────────┘
      │                            │
      │                            ▼
      │                    ┌─────────────┐
      │                    │Platform     │
      └────────────────────>│Event        │
                           └─────────────┘
```

**Aplicación:**
- Monitoreo de cambios en procesos de negocio
- Desacoplamiento entre emisores y receptores de eventos
- Notificaciones automáticas de estados

### 3. Adapter Pattern (Adaptador)
```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Client    │───>│   Adapter   │───>│   Vlocity   │
│ (Foundation)│    │  Interface  │    │   Service   │
└─────────────┘    └─────────────┘    └─────────────┘
```

**Aplicación:**
- Integración transparente con Vlocity/OmniStudio
- Abstracción de servicios externos
- Reutilización de componentes existentes

### 4. Template Method (Método Plantilla)
```
┌─────────────────────────┐
│   ProductHandler        │
│                         │
│ ┌─────────────────────┐ │
│ │    OnPre()          │ │ ◄── Virtual
│ │    (Virtual)        │ │
│ └─────────────────────┘ │
│ ┌─────────────────────┐ │
│ │    Process()        │ │ ◄── Encapsulated
│ │  (Encapsulated)     │ │
│ └─────────────────────┘ │
│ ┌─────────────────────┐ │
│ │    OnPos()          │ │ ◄── Virtual
│ │    (Virtual)        │ │
│ └─────────────────────┘ │
└─────────────────────────┘
```

**Aplicación:**
- Estructura común para todos los handlers de producto
- Puntos de extensión predefinidos (OnPre, OnPos)
- Lógica core protegida y reutilizable

---

## 🔗 Integración con Ecosistema Salesforce

### Vlocity/OmniStudio Integration

```
┌─────────────────────────────────────────────────────────────┐
│                    SALESFORCE ORG                          │
│                                                             │
│  ┌─────────────────┐    ┌─────────────────────────────────┐ │
│  │   VLOCITY/      │    │      SURA FOUNDATION            │ │
│  │   OMNISTUDIO    │◄──►│     (Managed Package)           │ │
│  │                 │    │                                 │ │
│  │ • Integration   │    │ • Adapters                      │ │
│  │   Procedures    │    │ • Process Handlers              │ │
│  │ • DataRaptors   │    │ • Event Management              │ │
│  │ • FlexCards     │    │ • External Integrations        │ │
│  └─────────────────┘    └─────────────────────────────────┘ │
│           │                           │                     │
│           └───────────────┬───────────┘                     │
│                           │                                 │
│  ┌─────────────────────────▼─────────────────────────────┐   │
│  │            SALESFORCE CORE                           │   │
│  │                                                      │   │
│  │ • Custom Objects     • Platform Events              │   │
│  │ • Custom Metadata    • Integration APIs              │   │
│  │ • Triggers           • Security Model                │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### Puntos de Integración:

1. **Integration Procedures**: Llamadas desde Vlocity hacia Foundation
2. **Platform Events**: Comunicación asíncrona bidireccional
3. **Custom Metadata Types**: Configuración compartida
4. **Apex Classes**: Lógica de negocio expuesta via interfaces

---

## 📊 Arquitectura de Datos

### Modelo de Metadatos

```
┌─────────────────────────────────────────────────────────────┐
│                 FOUNDATION METADATA                         │
│                                                             │
│  ┌─────────────────┐  ┌─────────────────┐                  │
│  │  Process Config │  │  Feature Flags  │                  │
│  │                 │  │                 │                  │
│  │ • Product       │  │ • Feature Name  │                  │
│  │ • Stage         │  │ • Enabled       │                  │
│  │ • Handler Class │  │ • Environment   │                  │
│  │ • Active        │  │ • Scope         │                  │
│  └─────────────────┘  └─────────────────┘                  │
│           │                     │                          │
│           └─────────────────────┼──────────────────────────┘
│                                 │                          
│  ┌─────────────────┐  ┌─────────▼───────┐                  │
│  │ Integration     │  │  Event Log      │                  │
│  │ Settings        │  │  Configuration  │                  │
│  │                 │  │                 │                  │
│  │ • Endpoint      │  │ • Log Level     │                  │
│  │ • Timeout       │  │ • Retention     │                  │
│  │ • Retry Policy  │  │ • Categories    │                  │
│  │ • Auth Method   │  │ • Cleanup Rules │                  │
│  └─────────────────┘  └─────────────────┘                  │
└─────────────────────────────────────────────────────────────┘
```

---

## 🚀 Empaquetamiento y Distribución

### Managed Package Strategy

#### Estructura del Paquete:
```
┌─────────────────────────────────────────────────────────────┐
│                 SURA FOUNDATION PACKAGE                     │
│                                                             │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │                 UPGRADEABLE COMPONENTS                  │ │
│  │                                                         │ │
│  │ • Custom Metadata Types                                 │ │
│  │ • Platform Events                                       │ │
│  │ • Custom Settings                                       │ │
│  │ • Subscriber-Editable Attributes                        │ │
│  └─────────────────────────────────────────────────────────┘ │
│                                                             │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │                NON-UPGRADEABLE COMPONENTS               │ │
│  │                                                         │ │
│  │ • Apex Classes (Core Logic)                             │ │
│  │ • Triggers                                              │ │
│  │ • Flows                                                 │ │
│  │ • Locked Attributes                                     │ │
│  └─────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

#### Ventajas del Managed Package:

1. **Código Protegido**: Lógica core no visible ni modificable
2. **Versionado Controlado**: Gestión de versiones y actualizaciones
3. **Distribución Centralizada**: Instalación en múltiples orgs
4. **Compatibilidad**: Asegura funcionamiento correcto
5. **Soporte**: Mantenimiento centralizado del framework

#### Estrategia de Versionado:

- **Versiones Inestables**: 0.1.0-NEXT, 0.1.0-2, etc.
- **Versiones Estables**: 1.0-1, 1.0-3, etc.
- **Actualizaciones**: Automáticas para componentes upgradeables

---

## 🔧 Arquitectura de Integración Externa

### Sistema de Reintentos

```
┌─────────────────────────────────────────────────────────────┐
│                 INTEGRATION RETRY LOGIC                     │
│                                                             │
│  Caller ──┬──> Business Process ──┬──> External Service     │
│           │                       │                        │
│           │    ┌─────────────┐    │    ┌─────────────┐     │
│           │    │ Callout 1   │    │    │ Response    │     │
│           │    │ Te1 + Tr1   │    │    │ Handler     │     │
│           │    └─────────────┘    │    └─────────────┘     │
│           │                       │                        │
│           │    ┌─────────────┐    │                        │
│           │    │ Callout 2   │    │                        │
│           │    │ Te2 + Tr2   │    │                        │
│           │    └─────────────┘    │                        │
│           │                       │                        │
│           └──> Total < 10000ms ───┘                        │
│                                                             │
│  Retry Patterns:                                           │
│  • Fixed: 100ms, 100ms, 100ms...                          │
│  • Incremental: 100ms, 200ms, 300ms...                    │
│  • Fibonacci: 100ms, 200ms, 300ms, 500ms, 800ms...       │
└─────────────────────────────────────────────────────────────┘
```

### Configuración de Integraciones:

- **Por Integración**: Configuración independiente
- **Códigos de Respuesta**: Definir qué códigos activan reintentos (400, 404, 500, etc.)
- **Cantidad de Reintentos**: Mínimo 1, máximo 5
- **Intervalos de Espera**: Configurables por patrón
- **Límites de Gobierno**: Respeto a los 10 segundos de Salesforce

---

## 📋 Panel de Administración

### Características del Panel:

```
┌─────────────────────────────────────────────────────────────┐
│              FOUNDATION ADMIN PANEL                         │
│                                                             │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │                 PROCESS MANAGEMENT                      │ │
│  │                                                         │ │
│  │ ┌─────────┬──────────┬─────────┬────────┬──────────────┐ │ │
│  │ │ Nombre  │ Producto │ Etapa   │ Orden  │ Estado       │ │ │
│  │ ├─────────┼──────────┼─────────┼────────┼──────────────┤ │ │
│  │ │ Lead    │ Viajes   │ Gestión │   0    │ ✅ Activo    │ │ │
│  │ │ Rate    │ Viajes   │ Tarif.  │   0    │ ✅ Activo    │ │ │
│  │ │ Quote   │ Viajes   │ Cotiz.  │   0    │ ✅ Activo    │ │ │
│  │ │ Emision │ Viajes   │ Emisión │   0    │ ✅ Activo    │ │ │
│  │ └─────────┴──────────┴─────────┴────────┴──────────────┘ │ │
│  └─────────────────────────────────────────────────────────┘ │
│                                                             │
│  [Instalar Metadatos] [Configuraciones] [Refrescar] [Nuevo] │
└─────────────────────────────────────────────────────────────┘
```

**Funcionalidades:**
- Activar/Desactivar procesos dinámicamente
- Configurar orden de ejecución
- Monitorear estado de integraciones
- Gestionar feature flags por ambiente

---

## 🎯 Beneficios Arquitectónicos

### Técnicos
- **Modularidad**: Componentes independientes y reutilizables
- **Escalabilidad**: Fácil adición de nuevos productos y procesos
- **Mantenibilidad**: Código organizado con responsabilidades claras
- **Testabilidad**: Componentes aislados facilitan pruebas unitarias
- **Performance**: Medición individual de cada proceso

### Operacionales
- **Disponibilidad**: Framework disponible desde Salesforce Core y OmniStudio
- **Configurabilidad**: Feature flags y metadata configurable
- **Monitoreo**: Eventos y logs centralizados
- **Distribución**: Managed package para múltiples organizaciones

### Estratégicos
- **Tiempo al Mercado**: Reducción en tiempo de implementación
- **Consistencia**: Estándares unificados para todos los productos
- **Evolución**: Base sólida para futuras expansiones
- **Riesgo**: Reducción de errores por código acoplado

---

## 🔮 Roadmap Arquitectónico

### Versión Actual (1.0)
- ✅ Framework core implementado
- ✅ Patrones de diseño establecidos
- ✅ Integración con Vlocity
- ✅ Productos Motos y Autos

### Versión 2.0 (Próxima)
- 🔄 Expansión a Viajes y Arrendamiento
- 🔄 Dashboard de monitoreo avanzado
- 🔄 APIs REST para integraciones externas
- 🔄 Optimizaciones de performance

### Versión 3.0 (Futuro)
- ⏳ Soporte multi-tenant avanzado
- ⏳ Machine Learning para optimización
- ⏳ Integración con herramientas DevOps
- ⏳ Expansión a otras líneas de negocio

---

**Fecha de Actualización:** Mayo 2025  
**Versión del Documento:** 1.0  
**Arquitectos:** Soulberto Lorenzo, Jean Carlos Melendez  
**Estado:** Documento Vivo - Actualización Continua