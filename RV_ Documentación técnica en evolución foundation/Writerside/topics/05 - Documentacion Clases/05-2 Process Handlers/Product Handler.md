---
title: Documentación Técnica - Objeto `Subconjunto Registro de Evento`
description: Definición técnica y funcional del objeto personalizado `Subconjunto_Registro_de_Evento__c` en Salesforce, incluyendo sus campos y características clave.
author: Salesforce Architecture Team
date: 2025-05-16
---

# Objeto `Subconjunto Registro de Evento`
Este objeto está destinado a registrar subconjuntos o trazabilidad de eventos internos, permitiendo una integración ordenada, monitoreo estructurado y separación de responsabilidades entre plataformas o procesos involucrados.

##  Información General

| Atributo                        | Valor                              |
|---------------------------------|------------------------------------|
| **Nombre API**                  | `Subconjunto_Registro_de_Evento__c`|
| **Etiqueta(Label)**             | Subconjunto Registro de Evento     |
| **Etiqueta(Label) Plural**      | Subconjunto Registros de Eventos   |
| **Estado de implementación**    | `Deployed`                         |
| **Namespace de acceso**         | `SFCore`                           |
| **Modelo de uso compartido interno** | `ReadWrite`                   |
| **Modelo de uso compartido externo** | `Private`                     |

---

## 🧾 Campos Personalizados

### 🟦 `Data__c`
- **Etiqueta(Label):** Dato
- **Tipo:** Área de texto larga (`LongTextArea`)
- **Longitud máxima:** 32.768 caracteres
- **Líneas visibles:** 3
- **Uso:** Almacena los datos de entrada para cada Handler (proceso de manejo) o integración (proceso de integración).
- **Requiere valor:** No

---

### 📅 `Date__c`
- **Etiqueta(Label):** Fecha
- **Tipo:** Fecha y hora (`DateTime`)
- **Valor predeterminado:** `NOW()`
- **Uso:** Registra la fecha y hora del almacenamiento del subconjunto.
- **Requiere valor:** Sí

---

### 📋 `Details__c`
- **Etiqueta(Label):** Detalle
- **Tipo:** Área de texto larga (`LongTextArea`)
- **Longitud máxima:** 131.072 caracteres
- **Líneas visibles:** 3
- **Uso:** Almacena descripciones detalladas o trazabilidad del evento.
- **Requiere valor:** No

---

### 🧭 `Level__c`
- **Etiqueta(Label):** Nivel
- **Tipo:** Texto (`Text`)
- **Longitud máxima:** 30 caracteres
- **Uso:** Nivel o severidad del evento (ej. INFO, WARNING, ERROR, SUCCESS).
- **Requiere valor:** Sí

---

### 🖥️ `Platform__c`
- **Etiqueta(Label):** Plataforma
- **Tipo:** Texto (`Text`)
- **Longitud máxima:** 50 caracteres
- **Uso:** Plataforma tecnológica desde la cual se origina el evento. (ej, Salesfoce, Formulario placa, etc)
- **Requiere valor:** No

---

### ⚙️ `Process__c`
- **Etiqueta(Label):** Proceso
- **Tipo:** Texto (`Text`)
- **Longitud máxima:** 80 caracteres
- **Uso:** Identificador del proceso responsable del evento (Proceso de Lead, Tarifa, Emision, Fasecolda, etc).
- **Requiere valor:** No

---

### 🛒 `Product__c`
- **Etiqueta(Label):** Producto
- **Tipo:** Texto (`Text`)
- **Longitud máxima:** 30 caracteres
- **Uso:** Producto asociado al evento (ej. Motos, Autos, Viajes).
- **Requiere valor:** No

---

### 💬 `Response__c`
- **Etiqueta(Label):** Response
- **Tipo:** Área de texto larga (`LongTextArea`)
- **Longitud máxima:** 32.768 caracteres
- **Líneas visibles:** 3
- **Uso:** Guarda la respuesta de un servicio externo o proceso interno.
- **Requiere valor:** No

---

### 🏷️ `Type__c`
- **Etiqueta(Label):** Tipo de Registro
- **Tipo:** Texto (`Text`)
- **Longitud máxima:** 30 caracteres
- **Uso:** Define el tipo específico del registro (Handler, Integración).
- **Requiere valor:** No

---

### 👤 `UserId__c`
- **Etiqueta(Label):** ID de Usuario
- **Tipo:** Texto (`Text`)
- **Longitud máxima:** 30 caracteres
- **Uso:** Identificador del usuario responsable del evento.
- **Requiere valor:** No

---

## Ejemplo Practico
Al Crear un lead de viajes y dejar el registro de Log para su trazabilidad, tenemos dentro del registro del objeto lo siguiente:
- **Registro de Evento Name:** Evento de LEAD
- **Fecha:** Fecha de creación
- **ID de Usuario:** ID de Usuario
- **Producto:** Viajes
- **Plataforma:** Salesforce
- **Dato:** --Datos de entrada del servicio -- "{\"Stage\":\"KNOWING\",\"options\":{},\"SESION_ID\":\"1747677805101\",\"assessorCode\":\"4999\",\"productName\":\"Viajes\",\"accept_tos\":true,\"email\":\"jo@test.us\",\"mobile_phone\":\"3124567887\",\"lastName\":\"johan\",\"first_name\":\"johan\"}" 
- **Response:** --Respuesta del servicio -- {\"type\":\"Lead\",\"url\":\"/services/data/v64.0/sobjects/Lead/00QU900000Cl3LKMAZ\"},\"Status\":\"0_Abierto\",\"Id\":\"00QU900000Cl3LKMAZ\",\"LastName\":\"johan\",\"FirstName\":null,\"Email\":\"jo@test.us\",\"MobilePhone\":\"3124567887\",\"Phone\":\"3124567887\",\"Company\":null,\"SURAAuthForPersonalDataProcess__c\":true,\"SURAUpgPersonDataProccess__c\":\"2020-11-01\",\"SURALastUpgPersonDataProccess__c\":\"2020-10-29\",\"RecordTypeId\":\"012D60000045fd1IAA\",\"SURAProducerId__c\":\"0YxD60000008UqJKAU\"}"
- **Detalle:** manejador de producto Viajes para proceso LEAD
- **Proceso:** Lead
- **Tipo de registro:** Handler(Proceso)
- **Nivel:** success





## 🧠 Observaciones Técnicas

- Utiliza diseño compacto del sistema (`SYSTEM`).
- Puede utilizarse en reportes y vía API o Streaming API.
- No está indexado para búsqueda global.

---



