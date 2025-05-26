# FoundationEventLogController

## Descripción General
`FoundationEventLogController` es una clase que actúa como controlador para componentes Lightning Web Components (LWC) o Aura Components, proporcionando funcionalidad de gestión de registros de eventos y trabajos programados. Esta clase expone métodos anotados con `@AuraEnabled` para permitir la interacción desde el frontend con las funcionalidades de análisis de eventos y programación de tareas de limpieza.

## Autor
Soulberto Lorenzo \<soulberto@cloudblue.us\>

## Última Modificación
7 de marzo de 2025 por "ChangeMeIn@UserSettingsUnder.SFDoc"

## Estructura de la Clase

```apex
public class FoundationEventLogController {
  @AuraEnabled public static Boolean refreshAnalytics(Datetime startAt, Datetime endAt);
  @AuraEnabled public static Boolean cleanAnalytics();
  @AuraEnabled public static eventInfoWrapper loadScheduleInfo();
  @AuraEnabled public static void refreshScheduleInfo(String cronExp, List<String> timeParts);
  @AuraEnabled public static boolean enabledScheduleInfo(Boolean enabled);
  
  public class eventInfoWrapper {
    @AuraEnabled public Boolean isActive;
    @AuraEnabled public Datetime nextFireTime;
    @AuraEnabled public Datetime previousFireTime;
    @AuraEnabled public String cronExp;
    @AuraEnabled public Integer numberOfExecutions;
  }
}
```

## Métodos

### `refreshAnalytics(Datetime startAt, Datetime endAt)`

#### Descripción
Método que inicia el proceso de actualización de análisis de eventos de forma asíncrona para un rango de fechas específico.

#### Parámetros
- `startAt` (Datetime): Fecha y hora de inicio del rango para el análisis.
- `endAt` (Datetime): Fecha y hora de fin del rango para el análisis.

#### Retorno
- `Boolean`: `true` si el proceso se inició exitosamente, `false` en caso de error.

#### Proceso
1. Valida que las fechas de inicio y fin no sean nulas.
2. Si alguna fecha es nula, registra un mensaje de depuración y usa la fecha/hora actual como valor por defecto.
3. Llama al método `refreshAnalyticsAsync` de `EventLogManager`.
4. Maneja excepciones y retorna el resultado.

#### Código
```apex
@AuraEnabled
public static Boolean refreshAnalytics(Datetime startAt, Datetime endAt) {
  try {
    if (startAt == null || endAt == null)
      Core.debug('start Date or End Date is coming null');
    EventLogManager.refreshAnalyticsAsync(
      startAt ?? Datetime.now(),
      endAt ?? Datetime.now(),
      false
    );
  } catch (Exception e) {
    Core.debug('Something has failed refreshing Analytics' + e.getMessage());
    return false;
  }
  return true;
}
```

### `cleanAnalytics()`

#### Descripción
Método que ejecuta la limpieza de datos analíticos eliminando registros del objeto `EventLog_Subset__c`.

#### Parámetros
Ninguno.

#### Retorno
- `Boolean`: `true` si la limpieza fue exitosa, `false` en caso contrario.

#### Proceso
1. Llama al método `clearAnalytics` de `EventLogManager`.
2. Retorna el resultado de la operación directamente.

#### Código
```apex
@AuraEnabled
public static Boolean cleanAnalytics() {
  Boolean result = false;
  result = EventLogManager.clearAnalytics();
  return result;
}
```

### `loadScheduleInfo()`

#### Descripción
Método que carga y retorna información sobre la configuración del trabajo programado de limpieza de eventos, incluyendo detalles del cronograma y estado de ejecución.

#### Parámetros
Ninguno.

#### Retorno
- `eventInfoWrapper`: Objeto wrapper que contiene información completa sobre el trabajo programado.

#### Proceso
1. Crea una instancia del wrapper de información de eventos.
2. Si se ejecuta en contexto de prueba, crea configuración mock.
3. Si no es prueba, consulta la configuración desde metadatos personalizados.
4. Extrae información básica (estado activo, expresión cron).
5. Si existe un ID de trabajo programado, consulta detalles del CronTrigger.
6. Retorna toda la información encapsulada en el wrapper.

#### Código
```apex
@AuraEnabled
public static eventInfoWrapper loadScheduleInfo() {
  eventInfoWrapper info = new eventInfoWrapper();

  Suratech_Foundation_Schedule_Event__mdt config;
  if (Test.isRunningTest()) {
    Id testJobId = System.schedule(
      'Test Event Log Cleanup',
      '0 00 02 ? * WED,THU,FRI *',
      new EventLog_Subset_CleanupScheduleClass()
    );
    config = new Suratech_Foundation_Schedule_Event__mdt(
      MasterLabel = 'DefaultConfig',
      DeveloperName = 'DefaultConfig',
      Frequency__c = '0 00 02 ? * WED,THU,FRI *',
      Is_Active__c = true,
      Schedule_Job_Id__c = testJobId
    );
  } else {
    config = [
      SELECT
        Id,
        Frequency__c,
        Schedule_Job_Id__c,
        MasterLabel,
        DeveloperName,
        Is_Active__c
      FROM Suratech_Foundation_Schedule_Event__mdt
      WHERE DeveloperName = 'DefaultConfig'
      LIMIT 1
    ];
  }

  info.isActive = config.Is_Active__c;
  info.cronExp = config.Frequency__c;

  if (!Test.isRunningTest() && config.Id == null) {
    throw new MetadataNotFoundException(
      'No Suratech_Foundation_Schedule_Event__mdt Custom Metadata found for "config"'
    );
  }

  if (config.Schedule_Job_Id__c != null) {
    CronTrigger ct = [
      SELECT
        Id,
        CronExpression,
        TimesTriggered,
        NextFireTime,
        PreviousFireTime
      FROM CronTrigger
      WHERE Id = :config.Schedule_Job_Id__c
    ];
    info.previousFireTime = ct.PreviousFireTime;
    info.nextFireTime = ct.NextFireTime;
    info.numberOfExecutions = ct.TimesTriggered;
  }
  return info;
}
```

### `refreshScheduleInfo(String cronExp, List<String> timeParts)`

#### Descripción
Método que actualiza la configuración del trabajo programado, incluyendo la reprogramación con una nueva expresión cron.

#### Parámetros
- `cronExp` (String): Nueva expresión cron para la programación del trabajo.
- `timeParts` (List<String>): Partes de tiempo (no utilizado en la implementación actual).

#### Retorno
- `void`: Este método no devuelve ningún valor.

#### Proceso
1. Obtiene la configuración actual de metadatos personalizados.
2. Actualiza la frecuencia con la nueva expresión cron.
3. Si existe un trabajo programado anterior, lo cancela.
4. Programa un nuevo trabajo con la nueva expresión cron.
5. Actualiza la configuración con el nuevo ID de trabajo.
6. Guarda los cambios en metadatos (excepto en pruebas).

#### Código
```apex
@AuraEnabled
public static void refreshScheduleInfo(
  String cronExp,
  List<String> timeParts
) {
  System.debug('Eventos');

  Suratech_Foundation_Schedule_Event__mdt config;
  if (Test.isRunningTest()) {
    Id testJobId = System.schedule(
      'Test Event Log Cleanup',
      cronExp,
      new EventLog_Subset_CleanupScheduleClass()
    );
    config = new Suratech_Foundation_Schedule_Event__mdt(
      MasterLabel = 'DefaultConfig',
      DeveloperName = 'DefaultConfig',
      Frequency__c = '',
      Is_Active__c = true,
      Schedule_Job_Id__c = ''
    );
  } else {
    config = [
      SELECT
        Id,
        Frequency__c,
        Schedule_Job_Id__c,
        MasterLabel,
        DeveloperName,
        Is_Active__c
      FROM Suratech_Foundation_Schedule_Event__mdt
      WHERE DeveloperName = 'DefaultConfig'
      LIMIT 1
    ];
  }

  config.Frequency__c = cronExp;

  if (config.Id == null) {
    config.MasterLabel = 'DefaultConfig';
    config.DeveloperName = 'DefaultConfig';
  }

  if (config.Schedule_Job_Id__c != null || config.Schedule_Job_Id__c == '') {
    try {
      System.abortJob(config.Schedule_Job_Id__c);
    } catch (Exception e) {
      System.debug('ERROR with Abbort Job: ' + e.getMessage());
    }
  }

  String jobName = Test.isRunningTest()
    ? 'Old Event Log Cleanup'
    : 'Test Event Log Cleanup';
  String jobId = System.schedule(
    jobName,
    cronExp,
    new EventLog_Subset_CleanupScheduleClass()
  );
  config.Is_Active__c = true;
  config.Schedule_Job_Id__c = jobId;

  if (!Test.isRunningTest())
    MetadataUtils.upsertMetadata(config);

  System.debug('Job rescheduled with ID: ' + jobId);
}
```

### `enabledScheduleInfo(Boolean enabled)`

#### Descripción
Método que habilita o deshabilita la configuración del trabajo programado (método incompleto en la implementación actual).

#### Parámetros
- `enabled` (Boolean): Indica si el trabajo debe estar habilitado o deshabilitado.

#### Retorno
- `Boolean`: `true` si la operación fue exitosa, `false` en caso de error.

#### Proceso
1. Obtiene la configuración de metadatos personalizados.
2. Intenta actualizar la configuración (implementación incompleta).
3. Maneja excepciones y retorna el resultado.

**Nota**: Este método parece estar incompleto, ya que no actualiza realmente el estado habilitado/deshabilitado.

## Clase Wrapper

### `eventInfoWrapper`

#### Descripción
Clase interna que encapsula información sobre el trabajo programado de eventos para ser utilizada por componentes del frontend.

#### Propiedades
- `isActive` (Boolean): Indica si el trabajo programado está activo.
- `nextFireTime` (Datetime): Próxima fecha y hora de ejecución del trabajo.
- `previousFireTime` (Datetime): Última fecha y hora de ejecución del trabajo.
- `cronExp` (String): Expresión cron que define la programación.
- `numberOfExecutions` (Integer): Número de veces que el trabajo ha sido ejecutado.

## Dependencias

- **`EventLogManager`**: Clase que proporciona la funcionalidad principal de gestión de eventos.
- **`EventLog_Subset_CleanupScheduleClass`**: Clase que implementa la lógica del trabajo programado.
- **`Suratech_Foundation_Schedule_Event__mdt`**: Tipo de metadatos personalizados para configuración.
- **`MetadataUtils`**: Clase utilitaria para operaciones con metadatos.
- **`Core`**: Clase utilitaria para registro de depuración.
- **`MetadataNotFoundException`**: Excepción personalizada para casos donde no se encuentra la configuración.

## Ejemplos de Uso

### Ejemplo 1: Uso desde Lightning Web Component

```javascript
// foundationEventLogManager.js
import { LightningElement, track, wire } from 'lwc';
import refreshAnalytics from '@salesforce/apex/FoundationEventLogController.refreshAnalytics';
import cleanAnalytics from '@salesforce/apex/FoundationEventLogController.cleanAnalytics';
import loadScheduleInfo from '@salesforce/apex/FoundationEventLogController.loadScheduleInfo';
import refreshScheduleInfo from '@salesforce/apex/FoundationEventLogController.refreshScheduleInfo';

export default class FoundationEventLogManager extends LightningElement {
    @track scheduleInfo = {};
    @track isLoading = false;
    @track startDate;
    @track endDate;
    @track cronExpression;

    connectedCallback() {
        this.loadScheduleInformation();
    }

    async loadScheduleInformation() {
        try {
            this.isLoading = true;
            this.scheduleInfo = await loadScheduleInfo();
            this.cronExpression = this.scheduleInfo.cronExp;
        } catch (error) {
            console.error('Error loading schedule info:', error);
        } finally {
            this.isLoading = false;
        }
    }

    async handleRefreshAnalytics() {
        if (!this.startDate || !this.endDate) {
            // Mostrar error de validación
            return;
        }

        try {
            this.isLoading = true;
            const result = await refreshAnalytics({
                startAt: new Date(this.startDate),
                endAt: new Date(this.endDate)
            });

            if (result) {
                // Mostrar mensaje de éxito
                this.showToast('Success', 'Analytics refresh started successfully', 'success');
            } else {
                // Mostrar mensaje de error
                this.showToast('Error', 'Failed to start analytics refresh', 'error');
            }
        } catch (error) {
            console.error('Error refreshing analytics:', error);
            this.showToast('Error', error.body.message, 'error');
        } finally {
            this.isLoading = false;
        }
    }

    async handleCleanAnalytics() {
        try {
            this.isLoading = true;
            const result = await cleanAnalytics();

            if (result) {
                this.showToast('Success', 'Analytics cleaned successfully', 'success');
            } else {
                this.showToast('Error', 'Failed to clean analytics', 'error');
            }
        } catch (error) {
            console.error('Error cleaning analytics:', error);
            this.showToast('Error', error.body.message, 'error');
        } finally {
            this.isLoading = false;
        }
    }

    async handleUpdateSchedule() {
        if (!this.cronExpression) {
            this.showToast('Error', 'Please provide a valid cron expression', 'error');
            return;
        }

        try {
            this.isLoading = true;
            await refreshScheduleInfo({
                cronExp: this.cronExpression,
                timeParts: []
            });

            this.showToast('Success', 'Schedule updated successfully', 'success');
            await this.loadScheduleInformation(); // Reload info
        } catch (error) {
            console.error('Error updating schedule:', error);
            this.showToast('Error', error.body.message, 'error');
        } finally {
            this.isLoading = false;
        }
    }

    handleStartDateChange(event) {
        this.startDate = event.target.value;
    }

    handleEndDateChange(event) {
        this.endDate = event.target.value;
    }

    handleCronExpressionChange(event) {
        this.cronExpression = event.target.value;
    }

    showToast(title, message, variant) {
        const event = new ShowToastEvent({
            title: title,
            message: message,
            variant: variant
        });
        this.dispatchEvent(event);
    }
}
```

```html
<!-- foundationEventLogManager.html -->
<template>
    <lightning-card title="Foundation Event Log Manager" icon-name="standard:logging">
        <div class="slds-p-horizontal_small">
            <!-- Analytics Section -->
            <div class="slds-section slds-is-open">
                <h3 class="slds-section__title">
                    <span class="slds-truncate slds-p-horizontal_small" title="Analytics Management">
                        Analytics Management
                    </span>
                </h3>
                <div class="slds-section__content slds-p-around_medium">
                    <div class="slds-grid slds-gutters">
                        <div class="slds-col slds-size_1-of-2">
                            <lightning-input
                                type="datetime-local"
                                label="Start Date"
                                value={startDate}
                                onchange={handleStartDateChange}>
                            </lightning-input>
                        </div>
                        <div class="slds-col slds-size_1-of-2">
                            <lightning-input
                                type="datetime-local"
                                label="End Date"
                                value={endDate}
                                onchange={handleEndDateChange}>
                            </lightning-input>
                        </div>
                    </div>
                    
                    <div class="slds-m-top_medium">
                        <lightning-button
                            variant="brand"
                            label="Refresh Analytics"
                            onclick={handleRefreshAnalytics}
                            disabled={isLoading}>
                        </lightning-button>
                        
                        <lightning-button
                            variant="destructive"
                            label="Clean Analytics"
                            onclick={handleCleanAnalytics}
                            disabled={isLoading}
                            class="slds-m-left_small">
                        </lightning-button>
                    </div>
                </div>
            </div>

            <!-- Schedule Section -->
            <div class="slds-section slds-is-open slds-m-top_large">
                <h3 class="slds-section__title">
                    <span class="slds-truncate slds-p-horizontal_small" title="Schedule Management">
                        Schedule Management
                    </span>
                </h3>
                <div class="slds-section__content slds-p-around_medium">
                    <div class="slds-grid slds-gutters">
                        <div class="slds-col slds-size_1-of-2">
                            <lightning-input
                                label="Cron Expression"
                                value={cronExpression}
                                onchange={handleCronExpressionChange}
                                help="Example: 0 0 2 * * ? (Daily at 2 AM)">
                            </lightning-input>
                        </div>
                        <div class="slds-col slds-size_1-of-2">
                            <div class="slds-form-element">
                                <label class="slds-form-element__label">Schedule Status</label>
                                <div class="slds-form-element__control">
                                    <lightning-badge
                                        label={scheduleInfo.isActive ? 'Active' : 'Inactive'}
                                        variant={scheduleInfo.isActive ? 'success' : 'error'}>
                                    </lightning-badge>
                                </div>
                            </div>
                        </div>
                    </div>

                    <template if:true={scheduleInfo.nextFireTime}>
                        <div class="slds-grid slds-gutters slds-m-top_medium">
                            <div class="slds-col slds-size_1-of-3">
                                <div class="slds-form-element">
                                    <label class="slds-form-element__label">Next Execution</label>
                                    <div class="slds-form-element__control">
                                        <lightning-formatted-date-time
                                            value={scheduleInfo.nextFireTime}>
                                        </lightning-formatted-date-time>
                                    </div>
                                </div>
                            </div>
                            <div class="slds-col slds-size_1-of-3">
                                <div class="slds-form-element">
                                    <label class="slds-form-element__label">Last Execution</label>
                                    <div class="slds-form-element__control">
                                        <lightning-formatted-date-time
                                            value={scheduleInfo.previousFireTime}>
                                        </lightning-formatted-date-time>
                                    </div>
                                </div>
                            </div>
                            <div class="slds-col slds-size_1-of-3">
                                <div class="slds-form-element">
                                    <label class="slds-form-element__label">Executions Count</label>
                                    <div class="slds-form-element__control">
                                        <span class="slds-text-heading_small">
                                            {scheduleInfo.numberOfExecutions}
                                        </span>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </template>

                    <div class="slds-m-top_medium">
                        <lightning-button
                            variant="brand"
                            label="Update Schedule"
                            onclick={handleUpdateSchedule}
                            disabled={isLoading}>
                        </lightning-button>
                    </div>
                </div>
            </div>

            <!-- Loading Spinner -->
            <template if:true={isLoading}>
                <lightning-spinner
                    alternative-text="Processing..."
                    size="medium">
                </lightning-spinner>
            </template>
        </div>
    </lightning-card>
</template>
```

### Ejemplo 2: Uso desde Apex para Administración Programática

```apex
public class EventLogAdministration {
    
    // Método para refrescar análisis para el último mes
    public static void refreshLastMonthAnalytics() {
        Datetime lastMonth = Datetime.now().addDays(-30);
        Datetime now = Datetime.now();
        
        Boolean success = FoundationEventLogController.refreshAnalytics(lastMonth, now);
        
        if (success) {
            System.debug('Analytics refresh for last month initiated successfully');
        } else {
            System.debug('Failed to initiate analytics refresh');
        }
    }
    
    // Método para programar limpieza diaria
    public static void scheduleDailyCleanup() {
        String dailyCronExp = '0 0 2 * * ?'; // Todos los días a las 2 AM
        
        try {
            FoundationEventLogController.refreshScheduleInfo(dailyCronExp, new List<String>());
            System.debug('Daily cleanup scheduled successfully');
        } catch (Exception e) {
            System.debug('Error scheduling daily cleanup: ' + e.getMessage());
        }
    }
    
    // Método para obtener información del cronograma
    public static void displayScheduleInfo() {
        try {
            FoundationEventLogController.eventInfoWrapper info = 
                FoundationEventLogController.loadScheduleInfo();
            
            System.debug('Schedule Status: ' + (info.isActive ? 'Active' : 'Inactive'));
            System.debug('Cron Expression: ' + info.cronExp);
            System.debug('Next Fire Time: ' + info.nextFireTime);
            System.debug('Previous Fire Time: ' + info.previousFireTime);
            System.debug('Number of Executions: ' + info.numberOfExecutions);
        } catch (Exception e) {
            System.debug('Error loading schedule info: ' + e.getMessage());
        }
    }
    
    // Método para limpiar análisis
    public static void performAnalyticsCleanup() {
        Boolean success = FoundationEventLogController.cleanAnalytics();
        
        if (success) {
            System.debug('Analytics cleaned successfully');
        } else {
            System.debug('Failed to clean analytics');
        }
    }
}

// Uso de los métodos
EventLogAdministration.refreshLastMonthAnalytics();
EventLogAdministration.scheduleDailyCleanup();
EventLogAdministration.displayScheduleInfo();
EventLogAdministration.performAnalyticsCleanup();
```

### Ejemplo 3: Configuración de Expresiones Cron Comunes

```apex
public class CronExpressionHelper {
    
    // Expresiones cron comunes para la limpieza de eventos
    public static final Map<String, String> COMMON_SCHEDULES = new Map<String, String>{
        'DAILY_2AM' => '0 0 2 * * ?',           // Todos los días a las 2 AM
        'WEEKLY_SUNDAY' => '0 0 2 ? * SUN',     // Domingos a las 2 AM
        'MONTHLY_FIRST' => '0 0 2 1 * ?',       // Primer día del mes a las 2 AM
        'WEEKDAYS_ONLY' => '0 0 2 ? * MON-FRI', // Días laborables a las 2 AM
        'TWICE_DAILY' => '0 0 2,14 * * ?'       // Dos veces al día (2 AM y 2 PM)
    };
    
    public static void setupWeeklyCleanup() {
        FoundationEventLogController.refreshScheduleInfo(
            COMMON_SCHEDULES.get('WEEKLY_SUNDAY'), 
            new List<String>()
        );
    }
    
    public static void setupDailyCleanup() {
        FoundationEventLogController.refreshScheduleInfo(
            COMMON_SCHEDULES.get('DAILY_2AM'), 
            new List<String>()
        );
    }
    
    public static void setupMonthlyCleanup() {
        FoundationEventLogController.refreshScheduleInfo(
            COMMON_SCHEDULES.get('MONTHLY_FIRST'), 
            new List<String>()
        );
    }
}
```

## Mejores Prácticas Implementadas

1. **Separación de Responsabilidades**: El controlador actúa como una capa de interfaz entre el frontend y la lógica de negocio.

2. **Manejo de Contexto de Pruebas**: Incluye lógica específica para contextos de prueba.

3. **Validación de Datos**: Valida parámetros de entrada y maneja casos nulos.

4. **Manejo de Excepciones**: Incluye bloques try-catch para manejar errores.

5. **Uso de Wrapper Classes**: Utiliza clases wrapper para estructurar datos complejos para el frontend.

## Consideraciones y Mejoras Potenciales

### Observaciones sobre el Código

1. **Método Incompleto**: El método `enabledScheduleInfo` parece estar incompleto y no actualiza realmente el estado del trabajo programado.

2. **Parámetro No Utilizado**: El parámetro `timeParts` en `refreshScheduleInfo` no se utiliza en la implementación.

3. **Inconsistencia en Nombres de Trabajos**: Los nombres de trabajos en las pruebas están intercambiados.

4. **Lógica de Cancelación**: La lógica para cancelar trabajos programados podría mejorarse.

### Mejoras Sugeridas

```apex
// Versión mejorada del método enabledScheduleInfo
@AuraEnabled
public static boolean enabledScheduleInfo(Boolean enabled) {
    try {
        Suratech_Foundation_Schedule_Event__mdt config;
        if (Test.isRunningTest()) {
            config = new Suratech_Foundation_Schedule_Event__mdt(
                MasterLabel = 'DefaultConfig',
                DeveloperName = 'DefaultConfig',
                Frequency__c = '0 0 2 * * ?',
                Is_Active__c = enabled,
                Schedule_Job_Id__c = ''
            );
        } else {
            config = [
                SELECT Id, Frequency__c, Schedule_Job_Id__c, MasterLabel, DeveloperName, Is_Active__c
                FROM Suratech_Foundation_Schedule_Event__mdt
                WHERE DeveloperName = 'DefaultConfig'
                LIMIT 1
            ];
        }

        // Actualizar el estado
        config.Is_Active__c = enabled;
        
        // Si se está deshabilitando y hay un trabajo programado, cancelarlo
        if (!enabled && String.isNotBlank(config.Schedule_Job_Id__c)) {
            try {
                System.abortJob(config.Schedule_Job_Id__c);
                config.Schedule_Job_Id__c = null;
            } catch (Exception e) {
                System.debug('Error aborting job: ' + e.getMessage());
            }
        }
        
        if (!Test.isRunningTest()) {
            MetadataUtils.upsertMetadata(config);
        }
        
        return true;
    } catch (Exception e) {
        System.debug('Error in enabledScheduleInfo: ' + e.getMessage());
        return false;
    }
}
```

## Notas Adicionales

1. **Interfaz de Usuario**: Esta clase está diseñada específicamente para proporcionar una interfaz entre componentes Lightning y la funcionalidad de gestión de eventos.

2. **Configuración Centralizada**: Utiliza metadatos personalizados para mantener la configuración centralizada y modificable sin cambios de código.

3. **Soporte para Testing**: Incluye lógica robusta para manejar contextos de prueba.

4. **Flexibilidad de Programación**: Permite programación flexible de trabajos usando expresiones cron.

5. **Monitoreo de Trabajos**: Proporciona información detallada sobre el estado y historial de ejecución de trabajos programados.