# RateProductHandler

## Descripción General
`RateProductHandler` es una clase virtual global que extiende `ProductHandler` y se encarga de gestionar el proceso de calificación (rating) de un producto utilizando Integration Procedures.

## Autor y Proyecto
- **Autor**: Soulberto Lorenzo <soulberto@cloudblue.us>
- **Última modificación**: 21 de febrero de 2025
- **Último modificador**: Jean Carlos Melendez

## Constantes y Variables
```apex
public static final String PROCESS_NAME = 'Rating';
public static final String OPTION_SKIP_BEHAVIOR = 'skipDefaultBehavior';
public static final String RATING_IPROCEDURE = 'sfcore_ratingprocedure';
global IntegrationProcedureExecutor EXECUTOR;
public static final Set<String> REQUIRED_PARAMETERS = new Set<String>{
  'includeInputKeys',
  'instanceKey',
  'filters'
};
```

## Constructores
```apex
global RateProductHandler(String productName, IntegrationProcedureExecutor executor)
```
Constructor que recibe el nombre del producto y un ejecutor de Integration Procedure.

```apex
global RateProductHandler(String productName)
```
Constructor que recibe solo el nombre del producto e inicializa un ejecutor mock.

## Métodos Principales

### setObservers
```apex
public override void setObservers()
```
Sobrescribe el método de la clase padre para añadir un observador de tipo `RateObserver`.

### isValidForRatingParameters
```apex
global Boolean isValidForRatingParameters(Map<String, Object> data)
```
Verifica que todos los parámetros requeridos estén presentes en el mapa de datos:
- Recorre los parámetros requeridos definidos en `REQUIRED_PARAMETERS`
- Devuelve `false` si falta algún parámetro
- Devuelve `true` si todos los parámetros están presentes

### process
```apex
public override Map<String, Object> process(
  Map<String, Object> options,
  Map<String, Object> input,
  Map<String, Object> output
)
```
Método principal que procesa el producto:
- Verifica si debe omitir el comportamiento predeterminado
- Establece el nombre del proceso como 'Rating'
- Valida los parámetros de entrada
- Construye el mapa de entrada para el Integration Procedure
- Ejecuta el Integration Procedure
- Actualiza el mapa de salida con los resultados
- Devuelve el mapa de salida actualizado

### buildInputMap
```apex
public Map<String, Object> buildInputMap(
  Map<String, Object> input,
  Map<String, Object> options
)
```
Construye el mapa de entrada para el Integration Procedure:
- Crea una copia del mapa de entrada
- Agrega todas las opciones si no están vacías
- Devuelve el mapa combinado

### executeIntegrationProcedure
```apex
public Map<String, Object> executeIntegrationProcedure(
  Map<String, Object> ipInput
)
```
Ejecuta el Integration Procedure definido en `RATING_IPROCEDURE`:
- Utiliza el ejecutor para invocar el procedimiento
- Devuelve un mapa con la clave 'resultProcess' conteniendo el resultado

## Dependencias
- `ProductHandler`: Clase padre que extiende
- `IntegrationProcedureExecutor`: Interfaz o clase para ejecutar Integration Procedures
- `MockIntegrationProcedureExecutor`: Implementación mock del ejecutor
- `RateObserver`: Clase observadora para el proceso de rating
- `ProductHandlerImplementationException`: Excepción personalizada para errores de implementación

## Notas Técnicas
- La clase utiliza el patrón Observer con `addObserver`
- Implementa validación de parámetros requeridos
- Utiliza el patrón Strategy para la ejecución de procedimientos de integración
- Gestiona opciones para omitir comportamientos predeterminados
- Proporciona manejo de errores con excepciones personalizadas