# RUNTWrapper

## Descripción General
`RUNTWrapper` es una clase diseñada para manejar y estructurar los datos provenientes del Registro Único Nacional de Tránsito (RUNT) de Colombia. Esta clase encapsula información detallada sobre vehículos, sus propietarios y características técnicas.

## Autor
- **Autor:** felipe.correa@nespon.com
- **Proyecto:** Foundation Salesforce - Suratech - UH-17585
- **Última modificación:** 11-06-2024

## Anotaciones
- `@NamespaceAccessible`: Permite que la clase sea accesible desde otros namespaces, facilitando su uso en paquetes gestionados o en integraciones.

## Estructura de la Clase

### Propiedades Principales
La clase contiene numerosas propiedades que representan los diferentes atributos de un vehículo registrado en el RUNT:

| Propiedad | Descripción |
|-----------|-------------|
| `codigoResultado` | Código del resultado de la consulta |
| `fecha` | Fecha de la consulta o registro |
| `idUsuario` | Identificador del usuario |
| `noRegistro` | Número de registro |
| `noLicenciaTransito` | Número de licencia de tránsito |
| `fechaExpedicionLicTransito` | Fecha de expedición de la licencia de tránsito |
| `estadoDelVehiculo` | Estado actual del vehículo |
| `idTipoServicio` | ID del tipo de servicio del vehículo |
| `tipoServicio` | Descripción del tipo de servicio |
| `idClaseVehiculo` | ID de la clase del vehículo |
| `claseVehiculo` | Descripción de la clase del vehículo |
| `idMarca` | ID de la marca del vehículo |
| `marca` | Nombre de la marca del vehículo |
| `idLinea` | ID de la línea del vehículo |
| `linea` | Descripción de la línea del vehículo |
| `modelo` | Año modelo del vehículo |
| `idColor` | ID del color del vehículo |
| `color` | Descripción del color del vehículo |
| `noMotor` | Número de motor |
| `noChasis` | Número de chasis |
| `noVin` | Número VIN (Vehicle Identification Number) |
| `cilindraje` | Cilindraje del vehículo |
| `divipola` | Código de División Político-Administrativa |
| `idTipoCarroceria` | ID del tipo de carrocería |
| `tipoCarroceria` | Descripción del tipo de carrocería |
| `fechaMatricula` | Fecha de matrícula del vehículo |
| `tieneGravamenes` | Indica si el vehículo tiene gravámenes |
| `organismoTransito` | Organismo de tránsito donde está registrado |
| `prendas` | Información sobre prendas asociadas al vehículo |
| `prendario` | Información adicional sobre estado prendario |
| `clasificacion` | Clasificación del vehículo |
| `esRegrabadoMotor` | Indica si el motor ha sido regrabado |
| `esRegrabadoChasis` | Indica si el chasis ha sido regrabado |
| `esRegrabadoSerie` | Indica si la serie ha sido regrabada |
| `esRegrabadoVin` | Indica si el VIN ha sido regrabado |
| `tipoServicioNombre` | Nombre del tipo de servicio |
| `nroMotor` | Número de motor (duplicado de `noMotor`) |
| `capacidadCarga` | Capacidad de carga del vehículo |
| `pesoBrutoVehicular` | Peso bruto vehicular |
| `noEjes` | Número de ejes del vehículo |
| `noPlaca` | Número de placa del vehículo |
| `repotenciado` | Indica si el vehículo ha sido repotenciado |
| `diasMatriculado` | Días transcurridos desde la matrícula |
| `idPaisOrigen` | ID del país de origen del vehículo |
| `paisOrigen` | Nombre del país de origen |
| `capacidadPasajerosSentados` | Capacidad de pasajeros sentados |
| `idTipoImportacion` | ID del tipo de importación |
| `tipoImportacion` | Descripción del tipo de importación |
| `idTipoCombustible` | ID del tipo de combustible |
| `tipoCombustible` | Descripción del tipo de combustible |
| `nombreAcreedor` | Nombre del acreedor en caso de prenda |
| `datosTecnicos` | Objeto que contiene datos técnicos adicionales |
| `normalizacion` | Objeto con información de normalización |
| `propietarios` | Lista de propietarios generales |
| `propietariosActuales` | Lista de propietarios actuales con información detallada |
| `codigoUUID` | Código UUID de la consulta |

### Clases Internas

#### DatosTecnicos
Contiene información técnica adicional del vehículo:

| Propiedad | Descripción |
|-----------|-------------|
| `capacidadCarga` | Capacidad de carga en kilogramos |
| `pesoBrutoVehicular` | Peso bruto del vehículo en kilogramos |
| `noEjes` | Número de ejes |
| `puertas` | Número de puertas |
| `idTipoCombustible` | ID del tipo de combustible |
| `tipoCombustible` | Descripción del tipo de combustible |

#### Normalizacion
Contiene información sobre el estado de normalización del vehículo:

| Propiedad | Descripción |
|-----------|-------------|
| `deficienciaMatriculaInicial` | Indica deficiencias en la matrícula inicial |
| `vehiculoNormalizado` | Indica si el vehículo ha sido normalizado |

#### Propietario
Representa información general sobre los propietarios:

| Propiedad | Descripción |
|-----------|-------------|
| `cantidad` | Cantidad de propietarios |
| `tipoPropietario` | Tipo de propietario |

#### PropietarioActual
Contiene información detallada sobre los propietarios actuales:

| Propiedad | Descripción |
|-----------|-------------|
| `id` | ID del propietario |
| `idTipoDocumento` | ID del tipo de documento |
| `tipoDocumento` | Tipo de documento de identidad |
| `noDocumento` | Número de documento |
| `nombreCompleto` | Nombre completo del propietario |
| `primerNombre` | Primer nombre |
| `segundoNombre` | Segundo nombre |
| `primerApellido` | Primer apellido |
| `segundoApellido` | Segundo apellido |
| `tipoPropiedad` | Tipo de propiedad (numérico) |
| `detallePropiedad` | Detalle del tipo de propiedad |
| `fechaNacimiento` | Fecha de nacimiento |
| `aux` | Campo auxiliar |

## Métodos

### newInstance
```apex
public static RUNTWrapper newInstance(Map<String, Object> response)
```

**Descripción:** Método estático que crea una nueva instancia de `RUNTWrapper` a partir de una respuesta recibida, típicamente de una llamada API.

**Parámetros:**
- `response` (Map<String, Object>): Mapa que contiene la respuesta, donde se espera que haya una clave 'body' con los datos a deserializar.

**Retorna:** Una instancia de `RUNTWrapper` con los datos deserializados de la respuesta.

**Funcionalidad:**
1. Serializa el objeto almacenado en la clave 'body' del mapa de respuesta
2. Deserializa el JSON resultante a la clase `RUNTWrapper`
3. Devuelve la instancia creada

## Uso Típico
Esta clase se utiliza principalmente para:
1. Recibir y estructurar datos de consultas al RUNT
2. Facilitar el acceso a información detallada de vehículos colombianos
3. Servir como modelo de datos para integraciones con el sistema RUNT

## Consideraciones
- La clase contiene algunas propiedades que parecen duplicadas (como `noMotor` y `nroMotor`), lo que podría indicar variaciones en la nomenclatura del sistema de origen.
- Al ser `@NamespaceAccessible`, esta clase está diseñada para ser utilizada en contextos de paquetes gestionados o integraciones externas.