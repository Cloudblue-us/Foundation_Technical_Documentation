# Clase: LeadUtils

Esta clase proporciona utilidades para la gestión y manipulación de objetos Lead, facilitando la búsqueda, recuperación y conversión de datos relacionados con leads.

## 📋 Información General

| Atributo | Valor |
|----------|-------|
| **Nombre API** | `LeadUtils` |
| **Contexto** | `global virtual` |
| **Grupo** | `utils` |
| **Desarrollador original** | Jean Carlos Melendez |
| **Última modificación** | 30-01-2025 (Jean Carlos Melendez) |

## 🧩 Principales características

Esta clase proporciona métodos utilitarios para:
- Búsqueda de leads por criterios específicos (email y teléfono móvil)
- Recuperación de leads por ID
- Conversión de leads a cuentas para partes aseguradas

## 🔄 Métodos principales

### 🔍 searchLeadIdByEmailAndMobilePhone(String email, String mobilePhone)

```apex
global static Id searchLeadIdByEmailAndMobilePhone(String email, String mobilePhone)
```

- **Descripción**: Busca un lead por su correo electrónico y número de teléfono móvil.
- **Parámetros**:
    - `email` (String): Correo electrónico del lead.
    - `mobilePhone` (String): Número de teléfono móvil del lead.
- **Retorna**:
    - `Id`: ID del lead encontrado, o null si no se encuentra ningún lead.
- **Comportamiento**:
    1. Realiza una consulta SOQL para buscar leads que coincidan con el correo electrónico y teléfono móvil proporcionados.
    2. Si encuentra al menos un lead, devuelve el ID del primer lead encontrado.
    3. Si no encuentra ningún lead, devuelve null.
    4. Registra en el log de depuración el ID del lead encontrado.

### 📄 getLeadById(Id leadId)

```apex
global static Lead getLeadById(Id leadId)
```

- **Descripción**: Recupera un objeto Lead por su ID.
- **Parámetros**:
    - `leadId` (Id): ID del lead a recuperar.
- **Retorna**:
    - `Lead`: Objeto Lead correspondiente al ID proporcionado.
- **Comportamiento**:
    1. Realiza una consulta SOQL para buscar el lead con el ID proporcionado.
    2. Devuelve el objeto Lead encontrado.
    3. La consulta está limitada a 1 registro para optimizar el rendimiento.

### 🔄 convertInsuredPartyFromLeadToAccount(Map<String, Object> input)

```apex
global virtual List<Map<String, Object>> convertInsuredPartyFromLeadToAccount(Map<String, Object> input)
```

- **Descripción**: Convierte un lead en una estructura de datos para una parte asegurada (cuenta).
- **Parámetros**:
    - `input` (Map<String, Object>): Mapa con los datos de entrada, debe contener una clave 'leadId' con el ID del lead a convertir.
- **Retorna**:
    - `List<Map<String, Object>>`: Lista de mapas con la estructura de datos para las partes aseguradas.
- **Comportamiento**:
    1. Extrae el ID del lead del mapa de entrada.
    2. Consulta los datos del lead, incluyendo nombre, apellido, correo electrónico, teléfono móvil, ciudad y calle.
    3. Crea una estructura de datos para la parte asegurada utilizando el patrón Builder (AccountBuilder).
    4. Registra en el log de depuración la estructura de datos creada.
    5. Devuelve la lista de partes aseguradas.
- **Notas**:
    - Este método es virtual, por lo que puede ser sobrescrito en clases que extiendan LeadUtils.
    - Utiliza el patrón Builder para la construcción del objeto Account.

## 📊 Ejemplo práctico

```apex
// Búsqueda de un lead por correo electrónico y teléfono
Id leadId = LeadUtils.searchLeadIdByEmailAndMobilePhone('ejemplo@email.com', '5551234567');
if (leadId != null) {
    // Recuperación del lead completo
    Lead leadEncontrado = LeadUtils.getLeadById(leadId);
    
    // Conversión del lead a estructura de asegurado
    Map<String, Object> entrada = new Map<String, Object>{'leadId' => leadId};
    List<Map<String, Object>> partesAseguradas = new LeadUtils().convertInsuredPartyFromLeadToAccount(entrada);
    
    // Procesar las partes aseguradas
    for (Map<String, Object> parte : partesAseguradas) {
        Account cuentaAsegurado = (Account)parte.get('Asegurado');
        // Hacer algo con la cuenta...
    }
}
```

## 🔍 Dependencias

- **AccountBuilder**: Patrón Builder utilizado para construir objetos Account a partir de datos de Lead.
- **Objeto Lead**: Interactúa directamente con el objeto estándar Lead de Salesforce.

## 🧪 Consideraciones técnicas

- Los métodos `searchLeadIdByEmailAndMobilePhone` y `getLeadById` son estáticos, por lo que pueden ser llamados directamente desde la clase sin necesidad de instanciarla.
- El método `convertInsuredPartyFromLeadToAccount` es virtual y no estático, lo que permite sobrescribirlo en clases que extiendan LeadUtils.
- Las consultas SOQL están optimizadas con límites para evitar problemas de rendimiento.
- La clase utiliza el patrón Builder para la construcción de objetos Account, lo que facilita la creación de objetos complejos.