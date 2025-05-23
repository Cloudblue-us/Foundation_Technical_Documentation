# BaseProcessFoundationVlocityAdapter

## Descripción General
Clase abstracta global que actúa como adaptador entre el framework de Vlocity y el sistema de gestión de procesos base (`BaseProcessManager`). Proporciona una interfaz estandarizada para invocar procesos a través de diferentes etapas del flujo de trabajo.

## Información de la Clase
- **Autor:** Jean Carlos Melendez
- **Última modificación:** 03-31-2025
- **Modificado por:** Jean Carlos Melendez
- **Tipo:** Clase abstracta global
- **Propósito:** Adaptador para integración Vlocity-BaseProcessManager

## Métodos

### applyInvoke (Método Estático)
```apex
global static Boolean applyInvoke(
    Map<String, Object> inputMap,
    Map<String, Object> outputMap,
    Map<String, Object> optionMap
)
```

**Descripción:** Método estático global que actúa como punto de entrada principal para invocar procesos a través del BaseProcessManager.

**Parámetros:**
- `inputMap` (Map<String, Object>): Mapa de datos de entrada para el proceso
- `outputMap` (Map<String, Object>): Mapa donde se almacenarán los resultados del proceso
- `optionMap` (Map<String, Object>): Mapa de opciones de configuración para el proceso

**Retorna:**
- `Boolean`: `true` si el proceso se ejecutó exitosamente, `false` en caso contrario

**Comportamiento:**
1. Extrae el valor de 'Stage' del inputMap para determinar la etapa del proceso
2. Registra la etapa en el debug log
3. Delega la ejecución al `BaseProcessManager.invoke()` con los parámetros reorganizados
4. Retorna el resultado de la invocación

**Flujo de Datos:**
```
inputMap['Stage'] → stage → BaseProcessManager.invoke(stage, optionMap, inputMap, outputMap)
```

### invokeMethod
```apex
public Boolean invokeMethod(
    String methodName,
    Map<String, Object> inputMap,
    Map<String, Object> outputMap,
    Map<String, Object> optionMap
)
```

**Descripción:** Método público que proporciona un mecanismo de invocación dinámica basado en el nombre del método.

**Parámetros:**
- `methodName` (String): Nombre del método a ejecutar (case-insensitive)
- `inputMap` (Map<String, Object>): Mapa de datos de entrada
- `outputMap` (Map<String, Object>): Mapa de datos de salida
- `optionMap` (Map<String, Object>): Mapa de opciones de configuración

**Retorna:**
- `Boolean`: Resultado de la ejecución del método solicitado

**Comportamiento:**
- Utiliza una estructura `switch on` para determinar qué método ejecutar
- Convierte el nombre del método a mayúsculas para comparación case-insensitive
- Actualmente soporta únicamente 'APPLYINVOKE'
- Registra métodos no soportados en el debug log y retorna `false`

**Métodos Soportados:**
- `'APPLYINVOKE'`: Invoca el método `applyInvoke()`

## Patrón de Diseño

### Adapter Pattern
Esta clase implementa el **patrón Adapter**, actuando como:
- **Adaptador:** Entre Vlocity framework y BaseProcessManager
- **Interfaz unificada:** Para diferentes tipos de invocaciones de procesos
- **Punto de entrada:** Centralizado para operaciones de proceso

### Command Pattern
El método `invokeMethod` implementa aspectos del **patrón Command**:
- Encapsula la invocación de métodos como objetos
- Permite invocar métodos dinámicamente por nombre
- Facilita la extensión con nuevos comandos

## Integración con Vlocity

### Propósito en el Ecosistema
- **Bridge:** Conecta componentes Vlocity con lógica de negocio personalizada
- **Abstracción:** Oculta la complejidad del BaseProcessManager de Vlocity
- **Estandarización:** Proporciona interfaz consistente para invocaciones

### Flujo de Integración
```
Vlocity Component → BaseProcessFoundationVlocityAdapter → BaseProcessManager → Business Logic
```

## Consideraciones Técnicas

### Visibilidad Global
- Declarada como `global` para permitir acceso desde paquetes Vlocity
- Métodos estáticos globales para invocación directa sin instanciación

### Manejo de Parámetros
- Utiliza casting explícito para asegurar tipos correctos
- Usa operador de navegación segura (`?.`) para prevenir null pointer exceptions

### Extensibilidad
- Estructura switch permite agregar nuevos métodos fácilmente
- Clase abstracta permite herencia para especializaciones

## Dependencias
- **BaseProcessManager:** Clase principal que maneja la lógica de procesos
- **Vlocity Framework:** Para integración con componentes Vlocity
- **System.debug:** Para logging y diagnóstico

## Casos de Uso
1. **Procesos Vlocity:** Ejecutar lógica de negocio desde flows o components Vlocity
2. **Integración de Etapas:** Manejar diferentes etapas de procesos complejos
3. **Invocación Dinámica:** Ejecutar métodos específicos basado en configuración
4. **Punto de Entrada Unificado:** Centralizar invocaciones desde múltiples fuentes

## Mejoras Potenciales
- Implementar manejo de excepciones más robusto
- Agregar validación de parámetros de entrada
- Incluir logging más detallado para auditoría
- Expandir métodos soportados en `invokeMethod`
- Implementar caché para mejorar performance en invocaciones frecuentes