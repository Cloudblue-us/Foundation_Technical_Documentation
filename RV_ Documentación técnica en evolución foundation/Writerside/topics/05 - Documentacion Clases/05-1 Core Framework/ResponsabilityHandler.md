# ResponsabilityHandler

## Descripción General
`ResponsabilityHandler` es una clase abstracta que implementa el patrón de diseño Chain of Responsibility (Cadena de Responsabilidad). Esta clase extiende `Subject`, lo que sugiere que también implementa el patrón Observer. La clase está diseñada para procesar objetos `Lead` según planes seleccionados, permitiendo encadenar múltiples handlers para formar un flujo de procesamiento.

## Autor
Soulberto Lorenzo \<soulberto@cloudblue.us\>

## Última Modificación
2 de septiembre de 2024

## Características de Accesibilidad
La clase y todos sus miembros están anotados con `@namespaceAccessible`, lo que permite que sean accesibles desde otros namespaces en organizaciones con paquetes gestionados.

## Atributos

### `handlerName`
- **Tipo**: `String`
- **Accesibilidad**: `public`
- **Descripción**: Nombre identificativo del handler.

### `nextHandler`
- **Tipo**: `ResponsabilityHandler`
- **Accesibilidad**: `public`
- **Descripción**: Referencia al siguiente handler en la cadena de responsabilidad.

## Constructor

### `ResponsabilityHandler(String handlerName)`

#### Descripción
Constructor que inicializa un nuevo handler con un nombre específico.

#### Parámetros
- `handlerName` (String): Nombre identificativo para el handler.

#### Código
```apex
@namespaceAccessible
public ResponsabilityHandler(String handlerName) {
  this.handlerName = handlerName;
}
```

## Métodos

### `setNext(ResponsabilityHandler nextHandler)`

#### Descripción
Establece el siguiente handler en la cadena de responsabilidad y lo devuelve, permitiendo un encadenamiento fluido de métodos.

#### Parámetros
- `nextHandler` (ResponsabilityHandler): El siguiente handler en la cadena.

#### Retorno
- `ResponsabilityHandler`: El handler proporcionado, permitiendo encadenamiento de métodos.

#### Código
```apex
@namespaceAccessible
public ResponsabilityHandler setNext(ResponsabilityHandler nextHandler) {
  this.nextHandler = nextHandler;
  return nextHandler;
}
```

### `apply(Lead lead, String selectedPlan)`

#### Descripción
Método virtual que debe ser implementado por las clases concretas derivadas. Este método aplicará la lógica de procesamiento específica del handler a un objeto Lead y un plan seleccionado.

#### Parámetros
- `lead` (Lead): El objeto Lead a procesar.
- `selectedPlan` (String): El plan seleccionado que puede determinar el tipo de procesamiento.

#### Retorno
- `void`

#### Comportamiento por Defecto
En la implementación base, este método lanza una excepción `ProductHandlerNotImplementedException`, indicando que las clases derivadas deben sobrescribir este método con su implementación específica.

#### Código
```apex
@namespaceAccessible
public virtual void apply(Lead lead, String selectedPlan) {
  throw new ProductHandlerNotImplementedException(
    'Product Handler must be implemented!'
  );
}
```

## Excepciones

### `ProductHandlerNotImplementedException`
Esta excepción se lanza cuando una clase derivada no ha implementado correctamente el método `apply()`. Indica un error en el diseño o implementación de la cadena de responsabilidad.

## Patrones de Diseño Implementados

### Chain of Responsibility (Cadena de Responsabilidad)
Este patrón permite que múltiples objetos procesen una solicitud. La clase `ResponsabilityHandler` establece la estructura básica para crear una cadena de handlers, donde cada uno puede procesar la solicitud o pasarla al siguiente en la cadena.

### Observer (Observador)
Al extender la clase `Subject`, `ResponsabilityHandler` hereda la capacidad de notificar a los objetos observadores sobre cambios o eventos durante el procesamiento.

## Dependencias
- `Subject`: Clase base que implementa el patrón Observer.
- `ProductHandlerNotImplementedException`: Excepción personalizada lanzada cuando el método `apply()` no está implementado.
- `Lead`: Objeto estándar de Salesforce utilizado en el método `apply()`.

## Ejemplo de Uso
```apex
// Definir handlers concretos
public class LeadQualificationHandler extends ResponsabilityHandler {
    public LeadQualificationHandler() {
        super('LeadQualification');
    }
    
    public override void apply(Lead lead, String selectedPlan) {
        // Lógica para calificar el lead
        lead.Status = 'Qualified';
        lead.Rating = 'Hot';
        
        // Notificar a los observadores
        this.notifyObservers(lead);
        
        // Continuar con el siguiente handler si existe
        if (this.nextHandler != null) {
            this.nextHandler.apply(lead, selectedPlan);
        }
    }
}

public class LeadAssignmentHandler extends ResponsabilityHandler {
    public LeadAssignmentHandler() {
        super('LeadAssignment');
    }
    
    public override void apply(Lead lead, String selectedPlan) {
        // Lógica para asignar el lead según el plan
        if (selectedPlan == 'Premium') {
            lead.OwnerId = UserInfo.getUserId(); // Asignar al usuario actual como ejemplo
        }
        
        // Notificar a los observadores
        this.notifyObservers(lead);
        
        // Continuar con el siguiente handler si existe
        if (this.nextHandler != null) {
            this.nextHandler.apply(lead, selectedPlan);
        }
    }
}

// Configurar y utilizar la cadena de responsabilidad
ResponsabilityHandler qualificationHandler = new LeadQualificationHandler();
ResponsabilityHandler assignmentHandler = new LeadAssignmentHandler();

// Encadenar los handlers
qualificationHandler.setNext(assignmentHandler);

// Procesar un lead
Lead newLead = new Lead(
    FirstName = 'John',
    LastName = 'Doe',
    Company = 'ACME Inc.'
);
qualificationHandler.apply(newLead, 'Premium');
```

## Notas Adicionales
1. Esta clase abstracta está diseñada para ser extendida por clases concretas que implementen la lógica específica de procesamiento.
2. El patrón Chain of Responsibility implementado aquí permite una separación clara de responsabilidades y facilita la adición de nuevos handlers sin modificar el código existente.
3. La combinación con el patrón Observer (a través de la herencia de `Subject`) permite que otros objetos sean notificados durante el procesamiento, facilitando acciones adicionales o registro de eventos.
4. Aunque el método `apply()` lanza una excepción por defecto, se espera que las clases derivadas sobrescriban este método con su propia implementación.
5. La anotación `@namespaceAccessible` en todos los elementos de la clase indica que está diseñada para ser utilizada en un contexto de paquetes gestionados, permitiendo que clases en otros namespaces puedan extenderla y utilizarla.