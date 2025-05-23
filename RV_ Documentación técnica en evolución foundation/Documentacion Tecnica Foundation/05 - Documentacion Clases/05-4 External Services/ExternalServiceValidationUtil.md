# Documentación Técnica: ExternalServiceValidationUtil

## Descripción General
`ExternalServiceValidationUtil` es una clase de utilidad que proporciona métodos para validar datos utilizados en servicios externos, específicamente enfocada en la validación de placas de vehículos y el registro de eventos de integración.

## Autor
Felipe Correa \<felipe.correa@nespon.com\>

## Proyecto
Foundation Salesforce - Suratech - UH-17585

## Última Modificación
17 de enero de 2025

## Estructura y Métodos

### Validación de Placas

#### `plateValidation(String plate)`

##### Descripción
Método público que procesa y valida una placa de vehículo. Convierte la placa a mayúsculas y luego realiza la validación.

##### Parámetros
- `plate` (String): La placa del vehículo a validar.

##### Excepciones
- `ExternalServiceValidationException`: Si la placa no cumple con los criterios de validación.

##### Código
```apex
public static void plateValidation(String plate) {
  plate = parseUpperCase(plate);
  validatePlate(plate);
}
```

#### `parseUpperCase(String param)`

##### Descripción
Convierte una cadena a mayúsculas.

##### Parámetros
- `param` (String): La cadena a convertir.

##### Retorno
- `String`: La cadena convertida a mayúsculas.

##### Código
```apex
public static String parseUpperCase(String param) {
  return param.toUpperCase();
}
```

#### `validatePlate(String plate)`

##### Descripción
Valida una placa de vehículo según criterios específicos:
- No puede ser nula o vacía
- Debe tener exactamente 6 caracteres
- Debe seguir un patrón específico para automóviles o motocicletas

##### Parámetros
- `plate` (String): La placa del vehículo a validar.

##### Excepciones
- `ExternalServiceValidationException`: Si la placa no cumple con alguno de los criterios.

##### Código
```apex
public static void validatePlate(String plate) {
  if (plate == null || String.isEmpty(plate))
    throw new ExternalServiceValidationException(
      'The plate cannot be empty or Null!'
    );
  else if (plate.length() != 6) {
    throw new ExternalServiceValidationException(
      'The plate must contain 6 characters! Actual Length = ' + plate.length()
    );
  } else if (!platePattern(plate)) {
    throw new ExternalServiceValidationException(
      'The plate format must be correct!'
    );
  }
}
```

#### `platePattern(String plate)`

##### Descripción
Método privado que verifica si una placa sigue uno de los patrones definidos para automóviles o motocicletas utilizando expresiones regulares.

##### Parámetros
- `plate` (String): La placa a validar.

##### Retorno
- `Boolean`: `true` si la placa sigue alguno de los patrones definidos, `false` en caso contrario.

##### Patrones
- **Automóviles**: `(^[A-Z]([A-Z0-9][0-9]|[A-Z]{2})[0-9]{2}[A-Z0-9]$)`
    - Comienza con una letra mayúscula
    - Seguida por una letra mayúscula o número y un número, o dos letras mayúsculas
    - Seguida por dos números
    - Termina con una letra mayúscula o número
- **Motocicletas**: `(^[A-Z][0-9]{4}[A-Z0-9]$)`
    - Comienza con una letra mayúscula
    - Seguida por cuatro números
    - Termina con una letra mayúscula o número

##### Código
```apex
private static boolean platePattern(String plate) {
  String carPattern = '(^[A-Z]([A-Z0-9][0-9]|[A-Z]{2})[0-9]{2}[A-Z0-9]$)';
  String motoPattern = '(^[A-Z][0-9]{4}[A-Z0-9]$)';
  // return validatePattern(plate, carPattern+'|'+motoPattern);
  return Pattern.matches(carPattern + '|' + motoPattern, plate);
}
```

### Registro de Eventos

#### `publishServiceEventLog(String type, String details)`

##### Descripción
Publica un registro de evento relacionado con servicios externos si la función de registros de eventos está habilitada a través de `FeatureFlags`.

##### Parámetros
- `type` (String): El tipo o nivel del evento (por ejemplo, 'ERROR', 'INFO', etc.).
- `details` (String): Detalles del evento a registrar.

##### Proceso
1. Verifica si la función `sura_foundation_event_logs` está habilitada.
2. Si está habilitada, registra un evento asincrónico con información sobre el servicio externo.

##### Código
```apex
public static void publishServiceEventLog(String type, String details) {
  if (new FeatureFlags().evaluate('sura_foundation_event_logs').isEnabled()) {
    Core.debug('Saving ' + type + ' Log...');
    EventLogManager.saveAsync(
      JSON.serialize(
        new List<Map<String, Object>>{
          new Map<String, Object>{
            'Product' => 'All',
            'Level' => type,
            'Platform' => 'Salesforce',
            'Details' => details,
            'Process' => 'External Service',
            'Type' => 'Integración'
          }
        }
      )
    );
  }
}
```

## Dependencias
- `ExternalServiceValidationException`: Excepción personalizada lanzada cuando se incumplen los criterios de validación.
- `FeatureFlags`: Clase que gestiona la habilitación de características a través de indicadores.
- `Core`: Clase de utilidad que proporciona el método `debug` para registro.
- `EventLogManager`: Gestor que proporciona funcionalidades para guardar registros de eventos.

## Ejemplo de Uso

### Validación de Placa

```apex
try {
    // Validar una placa de automóvil
    ExternalServiceValidationUtil.plateValidation('ABC123');
    System.debug('La placa es válida.');
    
    // Proceder con operaciones relacionadas con la placa
    // ...
} catch (ExternalServiceValidationException e) {
    // Manejar error de validación
    System.debug('Error de validación: ' + e.getMessage());
    
    // Registrar el error
    ExternalServiceValidationUtil.publishServiceEventLog('ERROR', e.getMessage());
    
    // Informar al usuario
    ApexPages.addMessage(new ApexPages.Message(
        ApexPages.Severity.ERROR,
        'La placa proporcionada no es válida: ' + e.getMessage()
    ));
}
```

### Publicación de Evento de Servicio

```apex
// Registrar un evento informativo
ExternalServiceValidationUtil.publishServiceEventLog(
    'INFO', 
    'Consulta exitosa a servicio externo de validación de placas. Placa: ABC123'
);

// Registrar un evento de error
try {
    // Operación del servicio externo
    // ...
} catch (Exception e) {
    ExternalServiceValidationUtil.publishServiceEventLog(
        'ERROR', 
        'Error en servicio externo: ' + e.getMessage()
    );
}
```

## Patrones Válidos de Placas

### Automóviles (carPattern)
- **Patrón**: `(^[A-Z]([A-Z0-9][0-9]|[A-Z]{2})[0-9]{2}[A-Z0-9]$)`
- **Ejemplos válidos**:
    - `ABC123`: Primera letra (A) + dos letras (BC) + dos números (12) + letra/número (3)
    - `A1D234`: Primera letra (A) + letra/número y número (1D) + dos números (23) + letra/número (4)

### Motocicletas (motoPattern)
- **Patrón**: `(^[A-Z][0-9]{4}[A-Z0-9]$)`
- **Ejemplos válidos**:
    - `A1234B`: Primera letra (A) + cuatro números (1234) + letra/número (B)
    - `Z9876X`: Primera letra (Z) + cuatro números (9876) + letra/número (X)

## Notas Adicionales
1. La clase contiene código comentado que sugiere una implementación alternativa para la validación de patrones y el guardado sincrónico de eventos. Estas líneas están actualmente inactivas.
2. El método `publishServiceEventLog` utiliza `saveAsync` para guardar los eventos de manera asincrónica, lo que optimiza el rendimiento al no bloquear la transacción actual.
3. La validación de placas está diseñada específicamente para un formato que tiene 6 caracteres, siguiendo patrones diferentes para automóviles y motocicletas.
4. La clase utiliza regiones de código (`#region`) para organizar lógicamente los métodos relacionados, mejorando la legibilidad del código.
5. El uso de `FeatureFlags` permite habilitar o deshabilitar el registro de eventos sin modificar el código, facilitando los despliegues y pruebas.

## Posibles Mejoras
1. Añadir documentación JavaDoc completa a todos los métodos.
2. Implementar validaciones adicionales para otros tipos de datos utilizados en servicios externos.
3. Proporcionar métodos de utilidad para validar y formatear otros tipos comunes de información (como números de identificación, correos electrónicos, etc.).
4. Implementar métodos para validar placas de diferentes longitudes o formatos, para adaptarse a posibles cambios en los requisitos de formato.
5. Añadir tests unitarios exhaustivos para cubrir todos los casos posibles de validación.