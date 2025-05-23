# Alcance y Limitaciones - SURA Foundation Framework

## Objetivo del Proyecto

Desarrollar e implementar un framework de procesamiento centralizado (**SURA Foundation**) que permita la separación lógica y arquitectónica de los procesos de venta de seguros, proporcionando una base sólida y reutilizable para múltiples productos de la familia SURA, con especial enfoque en productos de movilidad (Motos, Autos, Viajes, Arrendamiento).

## Contexto del Proyecto

**SURA Foundation** surge como respuesta a los problemas arquitectónicos identificados en el sistema actual, donde múltiples productos comparten código híbrido y artefactos sin separación lógica clara, generando inestabilidad, dificultades de mantenimiento y imposibilidad de medir el rendimiento real de cada proceso.

## Alcance

### Funcionalidades Incluidas

1. **Framework Core de Procesamiento**
    - Implementación del patrón Chain of Responsibility para procesos secuenciales
    - Sistema de observadores (Observer Pattern) para eventos de negocio
    - Adaptadores (Adapter Pattern) para integración con Vlocity y sistemas externos
    - Template Method Pattern para estandarización de procesos

2. **Gestión de Procesos por Etapas**
    - Procesamiento Pre, In y Post para cada etapa del flujo de venta
    - Manejo de eventos mediante Platform Events
    - Service Callouts integrados con lógica de reintentos configurable
    - Panel de administración centralizado para gestión de procesos

3. **Separación Lógica por Producto**
    - Procesos independientes para cada línea de negocio (Motos, Autos, Viajes, etc.)
    - Handlers específicos por producto con nomenclatura estandarizada
    - Responsabilidades claramente definidas por proceso y producto
    - Capacidad de medir desempeño individual por proceso

4. **Artefactos Reutilizables Centralizados**
    - **Conocimiento:** LeadManager y componentes base
    - **Tarificación:** Componentes de cálculo y pricing
    - **Cotización:** SURACrearNumeroPrepoliza, QuoteManager, SURAPaymentManager
    - **Emisión:** InsurancePolicyManager, SURAAccountManager, SURAOpportunityManager
    - **Legalización:** InsurancePolicyTransactionManager, SURAInvoiceListeningServiceV1

5. **Sistema de Feature Flags**
    - Control dinámico de funcionalidades por ambiente
    - Configuración sin deployments mediante Custom Metadata Types
    - Gestión granular de permisos por funcionalidad

6. **Integración con Vlocity**
    - Adaptadores específicos para Integration Procedures
    - Reutilización del modelo estándar de Vlocity
    - Compatibilidad con Omnistudio y Salesforce Core

7. **Empaquetamiento como Managed Package**
    - Código protegido y no modificable
    - Versionado controlado (inestables: 0.x.x-NEXT, estables: 1.x-x)
    - Distribución centralizada a múltiples organizaciones
    - Actualizaciones controladas del framework

8. **Lógica Avanzada de Integraciones**
    - Configuración independiente por integración
    - Sistema de reintentos configurable (Fijo, Incremental, Fibonacci)
    - Manejo de códigos de respuesta específicos (400, 404, 500, etc.)
    - Respeto a límites de gobierno de Salesforce (< 10000ms total)

### Productos Soportados

- **Seguros de Motos** - Implementación completa
- **Seguros de Autos** - Implementación completa
- **Seguros de Viajes** - Base arquitectónica preparada
- **Arrendamiento** - Base arquitectónica preparada

### Procesos de Venta Cubiertos

1. **Conocimiento/Prospección**
    - Gestión de Leads
    - Captura y validación de datos iniciales
    - Integraciones con fuentes externas (RUNT, FASECOLDA)

2. **Tarificación**
    - Cálculos de pricing por producto
    - Configuración flexible de planes y coberturas
    - Validaciones de riesgos consultables

3. **Cotización**
    - Generación de prepolizas
    - Gestión de numeración automática
    - Cálculos de ajustes y descuentos

4. **Emisión**
    - Creación de pólizas definitivas
    - Integración con sistemas de emisión
    - Notificaciones automáticas

5. **Legalización**
    - Gestión de pagos y transacciones
    - Integración con pasarelas de pago
    - Finalización del proceso de venta

## Fuera del Alcance

### Limitaciones Actuales

1. **Procesos Internos Específicos de Productos**
    - El framework no desarrolla la lógica de negocio específica de cada producto
    - Solo proporciona la base arquitectónica para integrarlos

2. **Modificación de Procesos Neurálgicos**
    - Los procesos core de PreVenta no pueden ser duplicados o modificados
    - Implementación obligatoria del modelo estándar de Vlocity

3. **Productos No Incluidos**
    - Seguros de Vida
    - Seguros de Hogar
    - Seguros Empresariales
    - Otros productos fuera de la familia de Movilidad

4. **Mejoras en Lógica Existente**
    - No se incluyen optimizaciones de procesos legacy
    - El enfoque es arquitectónico, no funcional

5. **Interfaces de Usuario**
    - No incluye componentes de frontend específicos
    - Se enfoca en la capa de servicios y lógica de negocio

### Dependencias Externas

1. **Salesforce Platform**
    - Requiere Salesforce Enterprise o superior
    - Dependiente de límites de gobierno de la plataforma

2. **Vlocity/OmniStudio**
    - Integración obligatoria con Vlocity framework
    - Dependiente de versiones compatibles de OmniStudio

3. **Sistemas Externos**
    - RUNT (Registro Único Nacional de Tránsito)
    - FASECOLDA (Federación de Aseguradores Colombianos)
    - Sistemas de pagos y SARLAFT

## Roadmap de Implementación

### Fase 1: Foundation Core (Completada)
- ✅ Implementación de patrones base
- ✅ Managed Package inicial
- ✅ Procesos para Motos y Autos

### Fase 2: Expansión de Productos
- 🔄 Implementación para Seguros de Viajes
- 🔄 Implementación para Arrendamiento
- 🔄 Optimización de performance

### Fase 3: Funcionalidades Avanzadas
- ⏳ Dashboard de monitoreo en tiempo real
- ⏳ Analytics de performance por proceso
- ⏳ Integración con herramientas de CI/CD

### Fase 4: Escalabilidad
- ⏳ Soporte para nuevas familias de productos
- ⏳ Integración con sistemas de terceros
- ⏳ Arquitectura multi-tenant avanzada

## Retos a Afrontar

### Técnicos
1. **Mantenimiento del Core**
    - Evolución continua del framework base
    - Compatibilidad con nuevas versiones de Salesforce/Vlocity

2. **Migración de Productos Existentes**
    - Implementación del nuevo enfoque en productos actuales
    - Minimización del impacto en procesos en producción

3. **Gestión de Versiones**
    - Mantenimiento de múltiples versiones del package
    - Estrategia de deprecación de versiones antiguas

### Organizacionales
1. **Cambio de Mentalidad**
    - Adopción del nuevo modelo de desarrollo
    - Capacitación en patrones de diseño y mejores prácticas

2. **Coordinación entre Equipos**
    - Sincronización entre equipos de diferentes productos
    - Establecimiento de estándares de desarrollo

3. **Puntos de Integración**
    - Mantenimiento de interfaces con sistemas externos
    - Gestión de dependencias entre componentes

## Beneficios Esperados

### Técnicos
- **Separación lógica clara** entre procesos de diferentes productos
- **Facilidad para incluir nuevos productos** y procesos
- **Versionado y obsolescencia controlada** de componentes
- **Mejoras en el manejo de eventos** y notificaciones
- **Independencia lógica** entre productos

### Operacionales
- **Distribución controlada** mediante managed packages
- **Disponibilidad desde Salesforce Core y Omnistudio**
- **Capacidad de medir desempeño** real por proceso
- **Múltiples lógicas simultáneas** de tarificación, cotización y emisión

### Estratégicos
- **Base escalable** para futuros productos
- **Arquitectura enterprise** robusta y mantenible
- **Reducción de riesgos** por acoplamiento de código
- **Velocidad de implementación** de nuevos productos

## Consideraciones de Implementación

### Prerrequisitos
- Salesforce Enterprise Edition o superior
- Vlocity Industries CPQ instalado
- Permisos de instalación de managed packages
- Configuración de Named Credentials para integraciones

### Recursos Requeridos
- Desarrolladores con experiencia en Salesforce y Vlocity
- Arquitectos familiarizados con patrones de diseño
- Administradores para configuración de procesos
- Equipo de QA para validación de integraciones

### Métricas de Éxito
- Reducción en tiempo de implementación de nuevos productos
- Mejora en la estabilidad de procesos existentes
- Disminución de incidentes por acoplamiento de código
- Aumento en la velocidad de resolución de issues

---

**Fecha de Última Actualización:** Mayo 2025  
**Versión del Documento:** 1.0  
**Responsables:** Soulberto Lorenzo, Jean Carlos Melendez