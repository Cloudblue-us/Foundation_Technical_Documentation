# Documentación Técnica: LocalEmisionNotificationHandlerTest.cls

## Descripción General
`LocalEmisionNotificationHandlerTest` es una clase de prueba de Apex diseñada para validar el funcionamiento de la clase `LocalEmisionNotificationHandler`, que gestiona notificaciones de emisión local para productos.

## Autor y Proyecto
- **Autor**: felipe.correa@nespon.com
- **Proyecto**: Foundation Salesforce - Suratech - UH-17589
- **Última modificación**: 31 de enero de 2025

## Estructura de la Clase

### Método @testSetup
```apex
@testSetup
static void makeData() {
  TestUtils.insertTestUsers();
}
```
Este método configura los datos de prueba necesarios, específicamente insertando usuarios de prueba mediante la clase utilitaria `TestUtils`.

### Métodos de Prueba

#### 1. testLocalEmision_Success
```apex
@isTest
static void testLocalEmision_Success()
```
Valida el flujo exitoso de la notificación de emisión local:
- Configura un mock HTTP que devuelve código 200 (OK)
- Ejecuta la prueba en el contexto de un usuario de prueba
- Crea un mapa de entrada con el producto 'viajes'
- Encola un trabajo asíncrono con `LocalEmisionNotificationHandler`
- Verifica que el estado del trabajo sea 'Completed'

#### 2. testLocalEmision_Exception_Failure
```apex
@isTest
static void testLocalEmision_Exception_Failure()
```
Prueba el manejo de errores cuando el servicio HTTP falla:
- Configura un mock HTTP que devuelve código 400 con un error
- Ejecuta la prueba en el contexto de un usuario de prueba
- Intenta encolar un trabajo con `LocalEmisionNotificationHandler`
- Verifica que se lance una excepción con el mensaje 'Error en el servicio'

#### 3. testLocalEmision_NoProduct_Failure
```apex
@isTest
static void testLocalEmision_NoProduct_Failure()
```
Prueba el escenario de fallo cuando no se proporciona un producto:
- Ejecuta la prueba en el contexto de un usuario de prueba
- Crea un mapa de entrada vacío (sin producto)
- Intenta encolar un trabajo con `LocalEmisionNotificationHandler`
- Verifica que se lance una excepción con el mensaje 'No se encontro el producto'

## Dependencias
- `TestUtils`: Clase utilitaria para crear datos de prueba
- `LocalEmisionNotificationHandler`: Clase principal que se está probando
- `LocalEmisionNotificationHandlerMock`: Mock para simular respuestas HTTP

## Notas Técnicas
- La clase utiliza el framework de pruebas de Apex con anotaciones `@isTest`
- Implementa pruebas para escenarios positivos y negativos
- Utiliza mocks HTTP para simular llamadas a servicios externos
- Emplea trabajos asíncronos con `System.enqueueJob()`
- Verifica el estado de los trabajos asíncronos después de su ejecución