# AccountBuilder

## Descripción General
Clase global que implementa el patrón Builder para la construcción flexible y configuración de objetos `Account`. Proporciona una interfaz fluida para crear cuentas con diferentes configuraciones, soporte para configuradores personalizados, y métodos de conveniencia para conversión de Leads a Accounts.

## Información de la Clase
- **Autor:** Jean Carlos Melendez
- **Última modificación:** 01-30-2025
- **Modificado por:** Jean Carlos Melendez
- **Tipo:** Clase global
- **Propósito:** Constructor flexible de objetos Account con patrón Builder

## Propiedades Globales

### account
```apex
global Account account
```
Instancia del objeto `Account` que se está construyendo.

### configurators
```apex
global List<AccountConfigurator> configurators
```
Lista de configuradores personalizados que se aplicarán durante la construcción del Account.

## Constructor

### AccountBuilder()
```apex
global AccountBuilder()
```
**Descripción:** Constructor por defecto que inicializa un nuevo objeto Account vacío.

**Comportamiento:**
- Crea una nueva instancia de `Account`
- Inicializa la lista de configuradores vacía

## Métodos Builder (Fluent Interface)

### setId
```apex
global AccountBuilder setId(String Id)
```
**Descripción:** Establece el ID del Account (típicamente para updates).

**Parámetros:**
- `Id` (String): ID del Account

**Retorna:** `AccountBuilder` para encadenamiento de métodos

### setName
```apex
global AccountBuilder setName(String name)
```
**Descripción:** Establece el nombre del Account.

**Parámetros:**
- `name` (String): Nombre del Account

**Retorna:** `AccountBuilder` para encadenamiento de métodos

### setPhone
```apex
global AccountBuilder setPhone(String phone)
```
**Descripción:** Establece el teléfono del Account.

**Parámetros:**
- `phone` (String): Número de teléfono

**Retorna:** `AccountBuilder` para encadenamiento de métodos

### setBillingAddress
```apex
global AccountBuilder setBillingAddress(String city, String street)
```
**Descripción:** Establece la dirección de facturación del Account.

**Parámetros:**
- `city` (String): Ciudad de facturación
- `street` (String): Calle de facturación

**Retorna:** `AccountBuilder` para encadenamiento de métodos

**Nota:** Solo configura ciudad y calle; otros campos de dirección pueden necesitar métodos adicionales.

### addConfigurators
```apex
global AccountBuilder addConfigurators(List<AccountConfigurator> configuratorsList)
```
**Descripción:** Agrega una lista de configuradores personalizados que se aplicarán durante la construcción.

**Parámetros:**
- `configuratorsList` (List<AccountConfigurator>): Lista de configuradores a agregar

**Retorna:** `AccountBuilder` para encadenamiento de métodos

**Comportamiento:**
- Verifica que la lista no esté vacía antes de agregar
- Utiliza `addAll()` para agregar todos los configuradores de una vez

## Métodos de Construcción

### build
```apex
global Account build()
```
**Descripción:** Construye el objeto Account aplicando todos los configuradores sin persistirlo en la base de datos.

**Retorna:** `Account` configurado pero no insertado

**Proceso:**
1. Verifica si hay configuradores en la lista
2. Itera sobre cada configurador y llama a `configure(account)`
3. Retorna el Account configurado

### buildAndInsert
```apex
global Account buildAndInsert()
```
**Descripción:** Construye el objeto Account, aplica configuradores, y lo inserta en la base de datos.

**Retorna:** `Account` configurado e insertado con ID asignado

**Proceso:**
1. Aplica todos los configuradores (mismo proceso que `build()`)
2. Ejecuta `insert account`
3. Retorna el Account con ID asignado

## Métodos de Servicio

### AccountService (con Record Type)
```apex
global Account AccountService(Lead foundLead, String recordTypeAPIName)
```
**Descripción:** Método de servicio para crear un Account basado en un Lead con un Record Type específico.

**Parámetros:**
- `foundLead` (Lead): Lead fuente para extraer datos
- `recordTypeAPIName` (String): API Name del Record Type a asignar

**Retorna:** `Account` creado e insertado

**Proceso:**
1. Consulta el Record Type por `DeveloperName`
2. Crea un nuevo AccountBuilder
3. Configura nombre (concatenando LastName + FirstName del Lead)
4. Configura teléfono desde `MobilePhone` del Lead
5. Ejecuta `buildAndInsert()`

**Nota:** El código tiene una consulta SOQL para Record Type pero no se asigna al Account.

### AccountService (Record Type por Defecto)
```apex
global Account AccountService(Lead foundLead)
```
**Descripción:** Sobrecarga del método anterior que usa 'PersonAccount' como Record Type por defecto.

**Parámetros:**
- `foundLead` (Lead): Lead fuente para extraer datos

**Retorna:** `Account` creado e insertado

**Comportamiento:** Llama a `AccountService(foundLead, 'PersonAccount')`

## Patrón de Diseño Implementado

### Builder Pattern
**Ventajas:**
- **Fluent Interface:** Métodos encadenables para construcción intuitiva
- **Flexibilidad:** Configuración opcional de diferentes campos
- **Extensibilidad:** Soporte para configuradores personalizados
- **Reutilización:** Mismo builder puede crear múltiples variaciones

**Estructura:**
```apex
Account acc = new AccountBuilder()
    .setName("Test Company")
    .setPhone("555-1234")
    .setBillingAddress("Miami", "123 Main St")
    .addConfigurators(customConfigurators)
    .buildAndInsert();
```

## Patrón Strategy (con Configuradores)

### AccountConfigurator Interface
Los configuradores implementan el patrón Strategy para aplicar configuraciones específicas:

```apex
public interface AccountConfigurator {
    void configure(Account account);
}
```

**Ejemplos de Configuradores:**
- Configurador de campos personalizados
- Configurador de integración externa
- Configurador específico por industria

## Casos de Uso

### Construcción Básica
```apex
Account basicAccount = new AccountBuilder()
    .setName("ACME Corp")
    .setPhone("555-0123")
    .build();
```

### Conversión de Lead a Account
```apex
Lead myLead = [SELECT LastName, FirstName, MobilePhone FROM Lead LIMIT 1];
Account convertedAccount = new AccountBuilder().AccountService(myLead);
```

### Configuración Avanzada con Configuradores
```apex
List<AccountConfigurator> configs = new List<AccountConfigurator>{
    new IndustryConfigurator('Technology'),
    new CustomFieldConfigurator()
};

Account complexAccount = new AccountBuilder()
    .setName("Tech Solutions Inc")
    .addConfigurators(configs)
    .buildAndInsert();
```

### Account con Record Type Específico
```apex
Account personAccount = new AccountBuilder()
    .AccountService(leadRecord, 'PersonAccount');
```

## Ventajas del Diseño

### Flexibilidad
- Configuración incremental de campos
- Soporte para extensiones a través de configuradores
- Métodos de conveniencia para casos comunes

### Mantenibilidad
- Separación clara entre construcción y configuración
- Código reutilizable para diferentes tipos de Account
- Fácil testing a través de builders

### Usabilidad
- Interfaz fluida y legible
- Métodos de servicio para casos específicos
- Soporte tanto para build como insert

## Consideraciones y Mejoras

### Problemas Identificados
1. **Record Type no asignado:** El método `AccountService` consulta Record Type pero no lo asigna
2. **Validación limitada:** Falta validación de campos requeridos
3. **Manejo de errores:** No hay manejo de excepciones en inserts

### Mejoras Sugeridas

#### Corrección de Record Type
```apex
global Account AccountService(Lead foundLead, String recordTypeAPIName) {
    Id recordTypeId = [SELECT Id FROM RecordType 
                      WHERE DeveloperName = :recordTypeAPIName LIMIT 1].Id;
    
    return new AccountBuilder()
        .setId(null) // Nuevo método para Record Type
        .setRecordTypeId(recordTypeId) // Método faltante
        .setName(foundLead?.LastName + ' ' + foundLead?.FirstName)
        .setPhone(foundLead?.MobilePhone)
        .buildAndInsert();
}
```

#### Métodos Adicionales Recomendados
```apex
global AccountBuilder setRecordTypeId(Id recordTypeId)
global AccountBuilder setIndustry(String industry)
global AccountBuilder setWebsite(String website)
global AccountBuilder setShippingAddress(String city, String street)
```

#### Validación
```apex
global Account build() {
    validateRequiredFields();
    applyConfigurators();
    return account;
}

private void validateRequiredFields() {
    if (String.isBlank(account.Name)) {
        throw new BuilderException('Account Name is required');
    }
}
```

## Dependencias
- **Account:** Objeto estándar de Salesforce
- **Lead:** Para métodos de conversión
- **RecordType:** Para configuración de tipos de cuenta
- **AccountConfigurator:** Interfaz para configuradores personalizados (debe ser implementada)

## Testing Considerations
- Crear builders de test con datos mock
- Probar encadenamiento de métodos
- Validar configuradores personalizados
- Testing de métodos de servicio con Leads reales