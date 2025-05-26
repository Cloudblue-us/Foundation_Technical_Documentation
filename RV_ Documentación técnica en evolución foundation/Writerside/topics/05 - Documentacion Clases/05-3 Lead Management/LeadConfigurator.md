# Documentación Técnica: LeadConfigurator

## Descripción General
`LeadConfigurator` es una interfaz global que define un contrato para clases que configuran objetos Lead en Salesforce. Esta interfaz forma parte del patrón de diseño Strategy, permitiendo encapsular diferentes algoritmos de configuración para Leads que pueden ser utilizados intercambiablemente.

## Autor
Jean Carlos Melendez

## Última Modificación
30 de enero de 2025

## Definición de la Interfaz

```apex
global interface LeadConfigurator {
  void configure(Lead lead);
}
```

## Métodos

### `configure(Lead lead)`

#### Descripción
Método que debe ser implementado por las clases que implementan esta interfaz. Su propósito es aplicar una configuración específica a un objeto Lead.

#### Parámetros
- `lead` (Lead): El objeto Lead que será configurado.

#### Retorno
- `void`: Este método no devuelve ningún valor.

## Propósito y Aplicaciones

El propósito principal de esta interfaz es proporcionar un mecanismo flexible para configurar objetos Lead de diferentes maneras sin modificar el código que utiliza estas configuraciones. Esto es particularmente útil en escenarios como:

1. **Configuración de Leads por canal**: Aplicar configuraciones específicas según el canal de origen del Lead (web, teléfono, evento, etc.).
2. **Configuración por campañas**: Configurar Leads con información relacionada a campañas específicas.
3. **Configuración por producto de interés**: Aplicar lógica personalizada según el producto o servicio de interés del Lead.
4. **Normalización de datos**: Implementar diferentes estrategias para limpiar y normalizar datos de Leads.
5. **Asignación de equipos o territorios**: Configurar asignaciones basadas en reglas personalizadas.

## Patrones de Diseño Relacionados

### Strategy
La interfaz `LeadConfigurator` es un ejemplo clásico del patrón de diseño Strategy, que permite definir una familia de algoritmos (en este caso, algoritmos de configuración de Leads), encapsularlos y hacerlos intercambiables. Este patrón permite que el algoritmo varíe independientemente de los clientes que lo utilizan.

### Composite
Cuando se utilizan múltiples implementaciones de `LeadConfigurator` juntas para configurar un Lead, se está empleando indirectamente el patrón Composite, permitiendo tratar tanto configuraciones individuales como grupos de configuraciones de manera uniforme.

### Decorator
Las implementaciones de esta interfaz pueden actuar como decoradores, añadiendo funcionalidades adicionales o modificando propiedades existentes de un objeto Lead sin cambiar su estructura básica.

## Relación con LeadBuilderFoundation

Esta interfaz está diseñada para trabajar en conjunto con la clase `LeadBuilderFoundation`, que implementa el patrón Builder para construir objetos Lead. La clase `LeadBuilderFoundation` tiene un método `addConfigurators` que acepta una lista de objetos que implementan esta interfaz, y luego los aplica secuencialmente en su método `build`.

```apex
// Fragmento de código de LeadBuilderFoundation
global Lead build() {
  if (!this.configurators.isEmpty()) {
    for (LeadConfigurator configurator : configurators) {
      configurator.configure(lead);
    }
  }
  return lead;
}
```

## Ejemplos de Implementación

### Configurador de Fuente de Lead

```apex
public class LeadSourceConfigurator implements LeadConfigurator {
    private String source;
    private String sourceDetail;
    
    public LeadSourceConfigurator(String source, String sourceDetail) {
        this.source = source;
        this.sourceDetail = sourceDetail;
    }
    
    public void configure(Lead lead) {
        lead.LeadSource = this.source;
        lead.Source_Detail__c = this.sourceDetail;
    }
}
```

### Configurador de Datos de Contacto

```apex
public class ContactInfoConfigurator implements LeadConfigurator {
    private String email;
    private String phone;
    private String country;
    
    public ContactInfoConfigurator(String email, String phone, String country) {
        this.email = email;
        this.phone = phone;
        this.country = country;
    }
    
    public void configure(Lead lead) {
        lead.Email = this.email;
        lead.Phone = this.phone;
        lead.Country = this.country;
        
        // Normalizar el número de teléfono según el país
        if (this.country == 'USA' || this.country == 'US') {
            lead.Phone = normalizeUSPhone(this.phone);
        }
    }
    
    private String normalizeUSPhone(String phone) {
        // Implementación de normalización de número telefónico de EE.UU.
        // ...
        return phone;
    }
}
```

### Configurador para Asignación de Propietario

```apex
public class OwnerAssignmentConfigurator implements LeadConfigurator {
    private String productInterest;
    
    public OwnerAssignmentConfigurator(String productInterest) {
        this.productInterest = productInterest;
    }
    
    public void configure(Lead lead) {
        lead.Product_Interest__c = this.productInterest;
        
        // Asignar propietario basado en el producto de interés
        Id ownerId = getOwnerIdForProduct(this.productInterest);
        if (ownerId != null) {
            lead.OwnerId = ownerId;
        }
    }
    
    private Id getOwnerIdForProduct(String product) {
        // Lógica para determinar el propietario adecuado basado en el producto
        // Podría consultar una configuración personalizada, una jerarquía de roles, etc.
        // ...
        return null; // Devolver el ID del propietario apropiado
    }
}
```

### Configurador Compuesto

```apex
public class CompositeLeadConfigurator implements LeadConfigurator {
    private List<LeadConfigurator> configurators;
    
    public CompositeLeadConfigurator() {
        this.configurators = new List<LeadConfigurator>();
    }
    
    public CompositeLeadConfigurator addConfigurator(LeadConfigurator configurator) {
        this.configurators.add(configurator);
        return this;
    }
    
    public void configure(Lead lead) {
        for (LeadConfigurator configurator : configurators) {
            configurator.configure(lead);
        }
    }
}
```

## Ejemplo de Uso con LeadBuilderFoundation

```apex
// Crear configuradores específicos
LeadSourceConfigurator sourceConfig = new LeadSourceConfigurator('Web', 'Landing Page');
ContactInfoConfigurator contactConfig = new ContactInfoConfigurator('john.doe@example.com', '555-1234', 'US');
OwnerAssignmentConfigurator ownerConfig = new OwnerAssignmentConfigurator('Cloud Services');

// Añadir configuradores al builder
List<LeadConfigurator> configurators = new List<LeadConfigurator>{
    sourceConfig,
    contactConfig,
    ownerConfig
};

// Crear y configurar un Lead usando el builder
Lead newLead = new LeadBuilderFoundation()
    .setFirstName('John')
    .setLastName('Doe')
    .addConfigurators(configurators)
    .build();

// Alternativamente, crear y configurar y insertar un Lead
Lead insertedLead = new LeadBuilderFoundation()
    .setFirstName('Jane')
    .setLastName('Smith')
    .addConfigurators(configurators)
    .buildAndInsert();
```

## Ejemplo Avanzado con Lógica Personalizada

```apex
public class CampaignResponderConfigurator implements LeadConfigurator {
    private Id campaignId;
    private Boolean attended;
    private Date responseDate;
    
    public CampaignResponderConfigurator(Id campaignId, Boolean attended, Date responseDate) {
        this.campaignId = campaignId;
        this.attended = attended;
        this.responseDate = responseDate;
    }
    
    public void configure(Lead lead) {
        // Configurar campos relacionados con la campaña
        lead.Campaign_Source__c = this.campaignId;
        
        // Después de insertar el Lead, crear un CampaignMember
        if (lead.Id != null) {
            try {
                CampaignMember member = new CampaignMember(
                    CampaignId = this.campaignId,
                    LeadId = lead.Id,
                    Status = this.attended ? 'Attended' : 'Responded',
                    HasResponded = true
                );
                
                if (this.responseDate != null) {
                    member.FirstRespondedDate = this.responseDate;
                }
                
                insert member;
            } catch (Exception e) {
                // Manejar errores o registrar excepciones
                System.debug('Error al crear CampaignMember: ' + e.getMessage());
            }
        }
    }
}
```

## Consideraciones para Testing

Al probar clases que implementan la interfaz `LeadConfigurator`, es importante:

1. **Verificar campos individuales**: Comprobar que cada campo afectado por el configurador tenga el valor esperado.
2. **Probar combinaciones**: Verificar que múltiples configuradores funcionen correctamente juntos sin conflictos.
3. **Simular dependencias**: Utilizar mocks o datos de prueba para cualquier consulta o lógica externa.
4. **Verificar efectos secundarios**: Si el configurador realiza DML u otras operaciones, asegurarse de que se ejecuten correctamente.

Ejemplo de una clase de prueba:

```apex
@isTest
private class LeadSourceConfiguratorTest {
    @isTest
    static void testConfigureLead() {
        // Preparar datos
        String source = 'Web';
        String detail = 'Social Media';
        LeadSourceConfigurator configurator = new LeadSourceConfigurator(source, detail);
        
        Lead testLead = new Lead(
            FirstName = 'Test',
            LastName = 'User'
        );
        
        // Ejecutar la configuración
        Test.startTest();
        configurator.configure(testLead);
        Test.stopTest();
        
        // Verificar resultados
        System.assertEquals(source, testLead.LeadSource, 'Lead source should be set correctly');
        System.assertEquals(detail, testLead.Source_Detail__c, 'Source detail should be set correctly');
    }
}
```

## Mejores Prácticas

1. **Principio de Responsabilidad Única**: Cada implementación de `LeadConfigurator` debe tener una responsabilidad única y específica.
2. **Inmutabilidad**: Las implementaciones deberían ser inmutables cuando sea posible, recibiendo todos los datos necesarios en el constructor.
3. **Manejo de errores robusto**: Implementar manejo de excepciones adecuado para evitar fallos en cascada.
4. **Documentación clara**: Documentar qué campos modifica cada configurador para facilitar su uso y mantenimiento.
5. **Evitar efectos secundarios**: Minimizar las operaciones DML u otros efectos secundarios dentro de los configuradores.
6. **Parametrización**: Diseñar configuradores parametrizados en lugar de codificar valores fijos.

## Notas Adicionales
1. La naturaleza global de la interfaz sugiere que está diseñada para ser accesible desde diferentes paquetes o namespaces.
2. Esta interfaz facilita la aplicación del principio abierto/cerrado, permitiendo extender la funcionalidad sin modificar el código existente.
3. Al ser parte de un sistema más amplio que incluye `LeadBuilderFoundation`, esta interfaz contribuye a una arquitectura más modular y mantenible para la gestión de Leads.