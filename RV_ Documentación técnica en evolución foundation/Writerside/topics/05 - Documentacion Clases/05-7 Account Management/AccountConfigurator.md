# AccountConfigurator

## Descripción General
Interfaz global que define el contrato para configuradores de objetos `Account`. Implementa el patrón Strategy permitiendo aplicar diferentes configuraciones a instancias de Account de manera flexible y extensible. Esta interfaz trabaja en conjunto con `AccountBuilder` para proporcionar un sistema de configuración modular.

## Información de la Interfaz
- **Autor:** Jean Carlos Melendez
- **Última modificación:** 01-30-2025
- **Modificado por:** Jean Carlos Melendez
- **Tipo:** Interfaz global
- **Propósito:** Contrato para configuradores de Account en patrón Strategy

## Método de la Interfaz

### configure
```apex
void configure(Account account)
```

**Descripción:** Método que debe ser implementado por todas las clases configuradoras para aplicar configuraciones específicas a un objeto Account.

**Parámetros:**
- `account` (Account): Instancia del objeto Account a configurar

**Retorna:** `void` - Modifica el Account por referencia

**Responsabilidades de la Implementación:**
- Aplicar configuraciones específicas al Account recibido
- Modificar campos, establecer valores por defecto, o aplicar lógica de negocio
- Manejar excepciones de configuración si es necesario

## Patrón de Diseño

### Strategy Pattern
Esta interfaz implementa el **patrón Strategy**, que permite:

- **Intercambiabilidad:** Diferentes estrategias de configuración sin cambiar el código cliente
- **Extensibilidad:** Agregar nuevas configuraciones sin modificar código existente
- **Separación de responsabilidades:** Cada configurador maneja una preocupación específica
- **Reutilización:** Configuradores pueden ser compartidos entre diferentes builders

### Integración con Builder Pattern
Trabaja en conjunto con `AccountBuilder` para crear un sistema flexible:

```apex
// El Builder utiliza la interfaz
global AccountBuilder addConfigurators(List<AccountConfigurator> configuratorsList)

// Durante build(), aplica todos los configuradores
for (AccountConfigurator configurator : configurators) {
    configurator.configure(account);
}
```

## Implementaciones Recomendadas

### Configurador de Industria
```apex
global class IndustryAccountConfigurator implements AccountConfigurator {
    private String industry;
    
    global IndustryAccountConfigurator(String industry) {
        this.industry = industry;
    }
    
    global void configure(Account account) {
        account.Industry = this.industry;
        
        // Configuración específica por industria
        switch on industry {
            when 'Technology' {
                account.Type = 'Customer - Direct';
                account.Rating = 'Hot';
            }
            when 'Healthcare' {
                account.Type = 'Customer - Channel';
                account.Rating = 'Warm';
            }
        }
    }
}
```

### Configurador de Dirección Completa
```apex
global class AddressAccountConfigurator implements AccountConfigurator {
    private String street, city, state, country, postalCode;
    
    global AddressAccountConfigurator(String street, String city, String state, String country, String postalCode) {
        this.street = street;
        this.city = city;
        this.state = state;
        this.country = country;
        this.postalCode = postalCode;
    }
    
    global void configure(Account account) {
        // Billing Address
        account.BillingStreet = this.street;
        account.BillingCity = this.city;
        account.BillingState = this.state;
        account.BillingCountry = this.country;
        account.BillingPostalCode = this.postalCode;
        
        // Shipping Address (mismo que billing por defecto)
        account.ShippingStreet = this.street;
        account.ShippingCity = this.city;
        account.ShippingState = this.state;
        account.ShippingCountry = this.country;
        account.ShippingPostalCode = this.postalCode;
    }
}
```

### Configurador de Campos Personalizados
```apex
global class CustomFieldAccountConfigurator implements AccountConfigurator {
    private Map<String, Object> customFields;
    
    global CustomFieldAccountConfigurator(Map<String, Object> customFields) {
        this.customFields = customFields;
    }
    
    global void configure(Account account) {
        for (String fieldName : customFields.keySet()) {
            try {
                account.put(fieldName, customFields.get(fieldName));
            } catch (Exception e) {
                System.debug('Error setting custom field ' + fieldName + ': ' + e.getMessage());
            }
        }
    }
}
```

### Configurador de Integración Externa
```apex
global class ExternalSystemAccountConfigurator implements AccountConfigurator {
    private String externalSystemId;
    private String systemName;
    
    global ExternalSystemAccountConfigurator(String externalSystemId, String systemName) {
        this.externalSystemId = externalSystemId;
        this.systemName = systemName;
    }
    
    global void configure(Account account) {
        // Configurar campos de integración
        account.put('External_System_ID__c', this.externalSystemId);
        account.put('Source_System__c', this.systemName);
        account.put('Integration_Status__c', 'Pending');
        account.put('Last_Sync_Date__c', System.now());
    }
}
```

### Configurador Condicional
```apex
global class ConditionalAccountConfigurator implements AccountConfigurator {
    private Map<String, Object> conditions;
    private List<AccountConfigurator> configurators;
    
    global ConditionalAccountConfigurator(Map<String, Object> conditions, List<AccountConfigurator> configurators) {
        this.conditions = conditions;
        this.configurators = configurators;
    }
    
    global void configure(Account account) {
        if (shouldApplyConfiguration(account)) {
            for (AccountConfigurator configurator : configurators) {
                configurator.configure(account);
            }
        }
    }
    
    private Boolean shouldApplyConfiguration(Account account) {
        // Lógica para evaluar condiciones
        for (String field : conditions.keySet()) {
            if (account.get(field) != conditions.get(field)) {
                return false;
            }
        }
        return true;
    }
}
```

## Casos de Uso Completos

### Configuración Empresarial Completa
```apex
List<AccountConfigurator> enterpriseConfigurators = new List<AccountConfigurator>{
    new IndustryAccountConfigurator('Technology'),
    new AddressAccountConfigurator('123 Tech St', 'San Francisco', 'CA', 'USA', '94105'),
    new CustomFieldAccountConfigurator(new Map<String, Object>{
        'Annual_Revenue__c' => 1000000,
        'Employee_Count__c' => 50,
        'Support_Level__c' => 'Premium'
    }),
    new ExternalSystemAccountConfigurator('EXT-12345', 'CRM_Legacy')
};

Account enterpriseAccount = new AccountBuilder()
    .setName('TechCorp Solutions')
    .setPhone('555-123-4567')
    .addConfigurators(enterpriseConfigurators)
    .buildAndInsert();
```

### Configuración con Validación
```apex
global class ValidatedAccountConfigurator implements AccountConfigurator {
    private List<AccountConfigurator> configurators;
    
    global void configure(Account account) {
        // Pre-validación
        validateAccount(account);
        
        // Aplicar configuradores
        for (AccountConfigurator configurator : configurators) {
            configurator.configure(account);
        }
        
        // Post-validación
        validateConfiguredAccount(account);
    }
    
    private void validateAccount(Account account) {
        if (String.isBlank(account.Name)) {
            throw new ConfigurationException('Account name is required before configuration');
        }
    }
    
    private void validateConfiguredAccount(Account account) {
        // Validaciones post-configuración
    }
}
```

## Ventajas del Diseño

### Flexibilidad
- **Composición dinámica:** Diferentes combinaciones de configuradores para diferentes tipos de Account
- **Configuración condicional:** Aplicar configuraciones basadas en condiciones específicas
- **Reutilización:** Configuradores pueden usarse en múltiples contextos

### Mantenibilidad
- **Separación de responsabilidades:** Cada configurador maneja una preocupación específica
- **Fácil testing:** Configuradores pueden probarse independientemente
- **Código modular:** Cambios en configuraciones no afectan otras partes del sistema

### Extensibilidad
- **Nuevas configuraciones:** Agregar configuradores sin modificar código existente
- **Configuraciones complejas:** Combinar múltiples configuradores simples
- **Integración con sistemas externos:** Configuradores específicos para integraciones

## Testing Strategies

### Test de Configurador Individual
```apex
@isTest
private class IndustryAccountConfiguratorTest {
    @isTest
    static void testTechnologyIndustryConfiguration() {
        Account testAccount = new Account(Name = 'Test Account');
        AccountConfigurator configurator = new IndustryAccountConfigurator('Technology');
        
        configurator.configure(testAccount);
        
        System.assertEquals('Technology', testAccount.Industry);
        System.assertEquals('Customer - Direct', testAccount.Type);
        System.assertEquals('Hot', testAccount.Rating);
    }
}
```

### Test de Múltiples Configuradores
```apex
@isTest
static void testMultipleConfigurators() {
    Account testAccount = new Account(Name = 'Test Account');
    List<AccountConfigurator> configurators = new List<AccountConfigurator>{
        new IndustryAccountConfigurator('Healthcare'),
        new AddressAccountConfigurator('456 Health Ave', 'Boston', 'MA', 'USA', '02101')
    };
    
    for (AccountConfigurator configurator : configurators) {
        configurator.configure(testAccount);
    }
    
    System.assertEquals('Healthcare', testAccount.Industry);
    System.assertEquals('456 Health Ave', testAccount.BillingStreet);
    System.assertEquals('Boston', testAccount.BillingCity);
}
```

## Consideraciones de Implementación

### Performance
- Configuradores deben ser eficientes, especialmente en operaciones bulk
- Evitar consultas SOQL dentro de configuradores cuando sea posible
- Considerar cache para configuraciones complejas

### Error Handling
- Configuradores deben manejar errores gracefully
- Logging apropiado para debugging
- Validación de datos de entrada

### Security
- Validar permisos de campo antes de configurar
- Sanitizar datos de entrada
- Respetar field-level security

## Mejores Prácticas

### Naming Convention
- Sufijo `AccountConfigurator` para todas las implementaciones
- Nombres descriptivos que indiquen el propósito: `IndustryAccountConfigurator`

### Constructor Pattern
- Configuradores deben recibir datos necesarios en el constructor
- Evitar configuradores con estado mutable

### Validation
- Validar parámetros en constructores
- Manejar casos edge apropiadamente
- Documentar precondiciones y postcondiciones

### Logging
- Log configuraciones aplicadas para auditoría
- Debug level apropiado para diferentes entornos

## Dependencias
- **Account:** Objeto estándar de Salesforce a configurar
- **AccountBuilder:** Clase que utiliza esta interfaz
- **System.debug:** Para logging (recomendado en implementaciones)

Esta interfaz es fundamental para crear un sistema flexible y extensible de configuración de Accounts, permitiendo que el código sea mantenible y fácil de extender con nuevas funcionalidades.