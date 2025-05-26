# BaseObserver

## Descripción General
Clase base virtual que implementa la interfaz `IObserver` para el patrón Observer. Proporciona implementaciones base para la observación de eventos y publicación de eventos de plataforma.

## Información de la Clase
- **Autor:** Soulberto Lorenzo <soulberto@cloudblue.us>
- **Última modificación:** 10-11-2024
- **Modificado por:** Jean Carlos Melendez
- **Tipo:** Clase virtual pública
- **Implementa:** IObserver

## Métodos

### updateObserver
```apex
public virtual void updateObserver(String flag, Object data)
```

**Descripción:** Método virtual para actualizar el observador cuando se produce un evento.

**Parámetros:**
- `flag` (String): Bandera que identifica el tipo de evento o acción
- `data` (Object): Datos asociados al evento

**Comportamiento:**
- Registra un mensaje de debug con el valor de la bandera
- Al ser virtual, puede ser sobrescrito en clases herederas

**Autor:** Soulberto Lorenzo <soulberto@cloudblue.us> | 08-16-2024

### publishPlatFormEvent
```apex
public virtual void publishPlatFormEvent(String flag, String productName)
```

**Descripción:** Método virtual para publicar eventos de plataforma relacionados con productos.

**Parámetros:**
- `flag` (String): Bandera que identifica el tipo de evento
- `productName` (String): Nombre del producto asociado al evento

**Comportamiento:**
- Registra un mensaje de debug indicando la publicación del evento para el producto especificado
- Al ser virtual, puede ser sobrescrito en clases herederas para implementar lógica específica

**Autor:** Soulberto Lorenzo <soulberto@cloudblue.us> | 08-22-2024

## Patrón de Diseño
Esta clase implementa el **patrón Observer**, donde:
- Actúa como observador base que puede recibir notificaciones
- Proporciona métodos virtuales que permiten a las clases herederas personalizar el comportamiento
- Facilita la publicación de eventos de plataforma

## Uso Recomendado
1. Extender esta clase para crear observadores específicos
2. Sobrescribir los métodos virtuales según las necesidades del negocio
3. Utilizar para implementar comunicación desacoplada entre componentes

## Consideraciones
- Los métodos son virtuales, por lo que deben ser sobrescritos para funcionalidad completa
- La implementación actual solo incluye logging para propósitos de debug
- Requiere la implementación completa de la interfaz `IObserver`