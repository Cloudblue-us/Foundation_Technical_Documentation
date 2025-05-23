# Documentación Técnica: MetaDataUtils

## Descripción General
`MetaDataUtils` es una clase Apex que proporciona utilidades para trabajar con metadatos personalizados en Salesforce. Esta clase facilita la creación y actualización (upsert) de registros de metadatos personalizados mediante la API de Metadata.

## Autor
Soulberto Lorenzo \<soulberto@cloudblue.us\>

## Última Modificación
28 de octubre de 2024

## Métodos Públicos

### `upsertMetadata(List<sObject> customMetadataList)`

#### Descripción
Este método permite crear o actualizar (upsert) múltiples registros de metadatos personalizados en una sola operación.

#### Parámetros
- `customMetadataList` (List\<sObject\>): Lista de objetos SObject que representan los registros de metadatos personalizados a insertar o actualizar.

#### Retorno
- `String`: ID del despliegue de metadatos. Este ID puede utilizarse para rastrear el estado del despliegue.

#### Proceso
1. Crea un contenedor de despliegue (`Metadata.DeployContainer`) para los metadatos personalizados.
2. Para cada objeto en la lista de metadatos:
    - Obtiene el nombre del objeto y sus detalles.
    - Crea una instancia de metadatos personalizados.
    - Establece el nombre completo y la etiqueta del registro.
    - Obtiene todos los campos del objeto.
    - Define un conjunto de campos para omitir ('developername', 'masterlabel', etc.).
    - Itera sobre los campos populados en el objeto.
    - Para cada campo no omitido y con valor no nulo:
        - Crea una instancia de valor de metadatos personalizados.
        - Asigna el nombre del campo y su valor.
        - Añade el campo al objeto de metadatos.
    - Añade el objeto de metadatos al contenedor de despliegue.
3. Crea una instancia de la clase callback (`CustomMetadataCallback`).
4. Encola el despliegue y retorna el ID de despliegue o un ID fijo ('04tO9000000ScNJ') si está ejecutándose en un contexto de prueba.

#### Código
```apex
public static String upsertMetadata(List<sObject> customMetadataList) {
  //Create Deployment container for custom Metadata
  Metadata.DeployContainer mdContainer = new Metadata.DeployContainer();
  for (sobject sObMD : customMetadataList) {
    //Get metadata object name and details
    String sObjectname = sObMD.getSObjectType().getDescribe().getName();

    //Create custom Metadata instance
    Metadata.CustomMetadata customMetadata = new Metadata.CustomMetadata();
    String recordName = String.valueOf(sObMD.get('DeveloperName'))
      .replaceAll(' ', '_');
    customMetadata.fullName = sObjectname + '.' + recordName;
    customMetadata.label = (String) sObMD.get('MasterLabel');

    //Get all fields
    schema.SObjectType sobjType = Schema.getGlobalDescribe().get(sObjectname);

    Map<String, Schema.sObjectField> sObjectFields = sobjType.getDescribe()
      .fields.getMap();
    Set<String> skipFieldSet = new Set<String>{
      'developername',
      'masterlabel',
      'language',
      'namespaceprefix',
      'label',
      'qualifiedapiname',
      'id'
    };

    // Use getPopulatedFieldsAsMap to get the populate field and iterate over them
    for (String fieldName : sObMD.getPopulatedFieldsAsMap().keySet()) {
      if (
        skipFieldSet.contains(fieldName.toLowerCase()) ||
        sObMD.get(fieldName) == null
      ) {
        continue;
      }

      Object value = sObMD.get(fieldName);
      //create field instance and populate it with field API name and value
      Metadata.CustomMetadataValue customField = new Metadata.CustomMetadataValue();
      customField.field = fieldName;

      Schema.DisplayType valueType = sObjectFields.get(fieldName)
        .getDescribe()
        .getType();
      customField.value = value;
      //Add fields in the object, similar to creating sObject instance
      customMetadata.values.add(customField);
    }
    //Add metadata in container
    mdContainer.addMetadata(customMetadata);
  }

  // Callback class instance
  CustomMetadataCallback callback = new CustomMetadataCallback();
  return Test.isRunningTest()
    ? '04tO9000000ScNJ'
    : Metadata.Operations.enqueueDeployment(mdContainer, callback);
}
```

### `upsertMetadata(sObject customMetadata)`

#### Descripción
Sobrecarga del método `upsertMetadata` que acepta un solo objeto SObject en lugar de una lista. Internamente, convierte el objeto en una lista y llama al método principal.

#### Parámetros
- `customMetadata` (sObject): Objeto SObject que representa el registro de metadatos personalizado a insertar o actualizar.

#### Retorno
- `String`: ID del despliegue de metadatos.

#### Código
```apex
public static String upsertMetadata(sObject customMetadata) {
  return upsertMetadata(new List<sObject>{ customMetadata });
}
```

## Observaciones
1. La clase incluye código comentado relacionado con la conversión de tipos de datos, particularmente para fechas, horas, porcentajes, monedas, etc. Este código está actualmente inactivo pero podría ser útil para manejar conversiones de tipos especiales si es necesario.
2. El método utiliza `getPopulatedFieldsAsMap()` para obtener solo los campos que han sido explícitamente asignados en el objeto, lo que mejora la eficiencia.
3. Se utiliza una clase de callback (`CustomMetadataCallback`) para manejar las respuestas del proceso de despliegue, aunque esta clase no está definida en el código proporcionado.
4. En entornos de prueba (`Test.isRunningTest()`), el método retorna un ID fijo ('04tO9000000ScNJ') en lugar de realizar el despliegue real.

## Dependencias
- `CustomMetadataCallback`: Una clase que implementa la interfaz `Metadata.DeployCallback` para manejar las respuestas del proceso de despliegue. Esta clase no está incluida en el código proporcionado.

## Ejemplo de Uso
```apex
// Crear un registro de metadatos personalizado
My_Custom_Metadata__mdt customMD = new My_Custom_Metadata__mdt();
customMD.DeveloperName = 'Test_Record';
customMD.MasterLabel = 'Test Record';
customMD.Field_1__c = 'Value 1';
customMD.Field_2__c = 100;

// Realizar el upsert
String deploymentId = MetaDataUtils.upsertMetadata(customMD);
System.debug('Deployment ID: ' + deploymentId);
```