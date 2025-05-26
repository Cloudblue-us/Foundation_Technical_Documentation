# LeadBuilderFoundation

## Descripción General
`LeadBuilderFoundation` es una clase global que implementa el patrón de diseño Builder para crear y configurar objetos Lead de manera fluida en Salesforce. Esta clase permite encadenar métodos para establecer diferentes atributos del Lead y aplicar configuradores personalizados antes de construir e insertar el registro.

## Autor
Felipe Correa \<felipe.correa@nespon.com\>

## Última Modificación
30 de enero de 2025 por Jean Carlos Melendez

## Atributos

### `lead`
- **Tipo**: `Lead`
- **Accesibilidad**: `global`
- **Descripción**: El objeto Lead que se está construyendo.

### `configurators`
- **Tipo**: `List<LeadConfigurator>`
- **Accesibilidad**: `global`
- **Descripción**: Lista de configuradores que aplicarán personalizaciones adicionales al Lead.

## Constructor

### `LeadBuilderFoundation()`

#### Descripción
Constructor que inicializa un nuevo objeto Lead vacío.

#### Parámetros
Ninguno.

#### Código
```apex
global LeadBuilderFoundation() {
  lead = new Lead();
}
```

## Métodos de Configuración Fluida

Estos métodos permiten establecer valores para campos específicos del Lead y devuelven la instancia del builder, permitiendo el encadenamiento de métodos.

### `setId(Id id)`

#### Descripción
Establece el Id del Lead.

#### Parámetros
- `id` (Id): Id del Lead.

#### Retorno
- `LeadBuilderFoundation`: La instancia actual del builder.

#### Código
```apex
global LeadBuilderFoundation setId(Id id) {
  lead.Id = id;
  return this;
}
```

### `setStatus(String status)`

#### Descripción
Establece el estado del Lead.

#### Parámetros
- `status` (String): Estado del Lead.

#### Retorno
- `LeadBuilderFoundation`: La instancia actual del builder.

#### Código
```apex
global LeadBuilderFoundation setStatus(String status) {
  lead.Status = status;
  return this;
}
```

### `setLastName(String lastName)`

#### Descripción
Establece el apellido del Lead.

#### Parámetros
- `lastName` (String): Apellido del Lead.

#### Retorno
- `LeadBuilderFoundation`: La instancia actual del builder.

#### Código
```apex
global LeadBuilderFoundation setLastName(String lastName) {
  lead.LastName = lastName;
  return this;
}
```

### `setFirstName(String firstName)`

#### Descripción
Establece el nombre del Lead.

#### Parámetros
- `firstName` (String): Nombre del Lead.

#### Retorno
- `LeadBuilderFoundation`: La instancia actual del builder.

#### Código
```apex
global LeadBuilderFoundation setFirstName(String firstName) {
  lead.FirstName = firstName;
  return this;
}
```

### `setEmail(String email)`

#### Descripción
Establece el correo electrónico del Lead.

#### Parámetros
- `email` (String): Correo electrónico del Lead.

#### Retorno
- `LeadBuilderFoundation`: La instancia actual del builder.

#### Código
```apex
global LeadBuilderFoundation setEmail(String email) {
  lead.Email = email;
  return this;
}
```

### `setMobile(String mobile)`

#### Descripción
Establece el número de teléfono móvil del Lead.

#### Parámetros
- `mobile` (String): Número de teléfono móvil del Lead.

#### Retorno
- `LeadBuilderFoundation`: La instancia actual del builder.

#### Código
```apex
global LeadBuilderFoundation setMobile(String mobile) {
  lead.MobilePhone = mobile;
  return this;
}
```

### `setPhone(String phone)`

#### Descripción
Establece el número de teléfono del Lead.

#### Parámetros
- `phone` (String): Número de teléfono del Lead.

#### Retorno
- `LeadBuilderFoundation`: La instancia actual del builder.

#### Código
```apex
global LeadBuilderFoundation setPhone(String phone) {
  lead.Phone = phone;
  return this;
}
```

### `addConfigurators(List<LeadConfigurator> configuratorsList)`

#### Descripción
Añade una lista de configuradores para personalizar el Lead.

#### Parámetros
- `configuratorsList` (List<LeadConfigurator>): Lista de configuradores a añadir.

#### Retorno
- `LeadBuilderFoundation`: La instancia actual del builder.

#### Código
```apex
global LeadBuilderFoundation addConfigurators(
  List<LeadConfigurator> configuratorsList
) {
  if (!configuratorsList.isEmpty()) {
    this.configurators.addAll(configuratorsList);
  }
  return this;
}
```

## Métodos de Construcción

### `build()`

#### Descripción
Construye el objeto Lead aplicando todos los configuradores añadidos.

#### Retorno
- `Lead`: El objeto Lead configurado.

#### Proceso
1. Verifica si hay configuradores en la lista.
2. Si existen, aplica cada configurador al objeto Lead.
3. Devuelve el objeto Lead configurado.

#### Código
```apex
global Lead build() {
  if (!this.configurators.isEmpty()) {
    for (LeadConfigurator configurator : configurators) {
      configurator.configure(lead);
    }
  }
  return lead;
}
```

### `buildAndInsert()`

#### Descripción
Construye el objeto Lead aplicando todos los configuradores y lo inserta en la base de datos.

#### Retorno
- `Lead`: El objeto Lead configurado e insertado.

#### Proceso
1. Verifica si hay configuradores en la lista.
2. Si existen, aplica cada configurador al objeto Lead.
3. Realiza un upsert del Lead en la base de datos.
4. Devuelve el objeto Lead insertado.

#### Código
```apex
global Lead buildAndInsert() {
  if (!this.configurators.isEmpty()) {
    for (LeadConfigurator configurator : configurators) {
      configurator.configure(lead);
    }
  }
  upsert lead;
  return lead;
}
```

## Métodos Estáticos de Servicio

### `createLead(Lead lead)`

#### Descripción
Método estático que inserta un Lead utilizando reglas de asignación.

#### Parámetros
- `lead` (Lead): El objeto Lead a insertar.

#### Retorno
- `Lead`: El objeto Lead insertado.

#### Código
```apex
global static Lead createLead(Lead lead) {
  return insertLeadWithAssignmentRules(lead);
}
```

### `upsertLead(Lead lead)`

#### Descripción
Método estático que realiza un upsert de un Lead utilizando reglas de asignación.

#### Parámetros
- `lead` (Lead): El objeto Lead para realizar upsert.

#### Retorno
- `Lead`: El objeto Lead después del upsert.

#### Proceso
1. Configura las opciones DML para utilizar la regla de asignación predeterminada.
2. Establece estas opciones en el objeto Lead.
3. Realiza un upsert del Lead en la base de datos.
4. Devuelve el objeto Lead actualizado.

#### Código
```apex
global static Lead upsertLead(Lead lead) {
  Database.DMLOptions dmlOptn = new Database.DMLOptions();
  dmlOptn.assignmentRuleHeader.useDefaultRule = true;
  lead.setOptions(dmlOptn);
  Database.upsert(lead);
  return lead;
}
```

## Métodos Privados

### `insertLeadWithAssignmentRules(Lead lead)`

#### Descripción
Método auxiliar privado que inserta un Lead utilizando reglas de asignación predeterminadas.

#### Parámetros
- `lead` (Lead): El objeto Lead a insertar.

#### Retorno
- `Lead`: El objeto Lead insertado.

#### Proceso
1. Configura las opciones DML para utilizar la regla de asignación predeterminada.
2. Establece estas opciones en el objeto Lead.
3. Realiza un upsert del Lead en la base de datos.
4. Devuelve el objeto Lead insertado.

#### Código
```apex
private static Lead insertLeadWithAssignmentRules(Lead lead) {
  Database.DMLOptions dmlOptn = new Database.DMLOptions();
  dmlOptn.assignmentRuleHeader.useDefaultRule = true;
  lead.setOptions(dmlOptn);
  Database.upsert(lead);
  return lead;
}
```

## Dependencias
- `Lead`: Objeto estándar de Salesforce para almacenar prospectos.
- `LeadConfigurator`: Interfaz o clase que define el método `configure(Lead)` para personalizar Leads.
- `Database.DMLOptions`: Clase de Salesforce para configurar opciones de operaciones DML.

## Patrón de Diseño Implementado

### Builder
La clase implementa el patrón de diseño Builder, que permite construir objetos complejos paso a paso. Las principales características de este patrón en la implementación son:

1. **Métodos de configuración fluida**: Todos los métodos `set*` devuelven la instancia del builder, permitiendo el encadenamiento de métodos.
2. **Separación de construcción y representación**: El proceso de construcción está separado de la representación final del objeto Lead.
3. **Proceso paso a paso**: El objeto Lead se construye gradualmente añadiendo propiedades específicas.
4. **Configuración extensible**: El método `addConfigurators` permite añadir comportamientos personalizados a través de implementaciones de `LeadConfigurator`.

## Ejemplo de Uso

### Uso Básico
```apex
// Crear un Lead con el Builder
Lead newLead = new LeadBuilderFoundation()
    .setFirstName('John')
    .setLastName('Doe')
    .setEmail('john.doe@example.com')
    .setPhone('555-1234')
    .setMobile('555-5678')
    .setStatus('Open')
    .build();

// Insertar el Lead creado usando el servicio estático
Lead insertedLead = LeadBuilderFoundation.createLead(newLead);
```

### Uso con Configuradores Personalizados
```apex
// Definir un configurador personalizado
public class CompanyNameConfigurator implements LeadConfigurator {
    private String companyName;
    
    public CompanyNameConfigurator(String companyName) {
        this.companyName = companyName;
    }
    
    public void configure(Lead lead) {
        lead.Company = this.companyName;
    }
}

// Definir otro configurador
public class LeadSourceConfigurator implements LeadConfigurator {
    private String source;
    
    public LeadSourceConfigurator(String source) {
        this.source = source;
    }
    
    public void configure(Lead lead) {
        lead.LeadSource = this.source;
    }
}

// Crear y usar una lista de configuradores
List<LeadConfigurator> configurators = new List<LeadConfigurator>{
    new CompanyNameConfigurator('ACME Inc.'),
    new LeadSourceConfigurator('Web')
};

// Crear y insertar un Lead con configuradores personalizados
Lead lead = new LeadBuilderFoundation()
    .setFirstName('Jane')
    .setLastName('Smith')
    .setEmail('jane.smith@example.com')
    .addConfigurators(configurators)
    .buildAndInsert();
```

### Uso con ID Existente (Actualización)
```apex
// Actualizar un Lead existente
Id existingLeadId = '00Q0x000000XXXXX';
Lead updatedLead = new LeadBuilderFoundation()
    .setId(existingLeadId)
    .setStatus('Qualified')
    .build();

// Realizar upsert del Lead
Lead result = LeadBuilderFoundation.upsertLead(updatedLead);
```

## Notas Adicionales
1. La clase utiliza el método `upsert` para permitir tanto la inserción como la actualización de registros de Lead, dependiendo de si se proporciona un Id.
2. Los configuradores personalizados (`LeadConfigurator`) permiten extender la funcionalidad del builder sin modificar su código base.
3. El uso de reglas de asignación asegura que los Leads se asignen correctamente según las reglas configuradas en Salesforce.
4. El patrón builder implementado mejora la legibilidad del código y simplifica la creación de objetos Lead con múltiples atributos.
5. La clase está marcada como `global`, lo que permite su uso en paquetes gestionados y su acceso desde otros namespaces.
6. Hay una duplicación de código entre `insertLeadWithAssignmentRules` y `upsertLead` que podría refactorizarse para mejorar la mantenibilidad.