# PIPointStatusWatchtower

**PIPointStatusWatchtower** es un **Servicio de Windows** desarrollado para monitorear y gestionar el estado de puntos en el sistema PI de OSIsoft. Utiliza **AFSDK** (Asset Framework SDK) para interactuar con el PI System y **NLog** para registrar información de logs y eventos.

## Descripción

El objetivo de este proyecto es proporcionar un servicio automatizado que pueda rastrear y reportar el estado de puntos en el sistema PI, permitiendo a los usuarios gestionar y responder a eventos críticos de manera eficiente. A través de su integración con AFSDK, el servicio puede acceder a los datos de infraestructura del PI System. Además, los logs generados con NLog permiten un seguimiento detallado de las operaciones realizadas.

## Características

- **Monitoreo en tiempo real** del estado de puntos del sistema PI.
- **Integración con PI System** a través de AFSDK.
- **Logs centralizados y detallados** mediante NLog.
- **Configuración dinámica** a través de archivos YAML o JSON.
- **Arquitectura extensible y modular** para futuras mejoras.
- **Ejecución como un Servicio de Windows**, con soporte para instalación/desinstalación.

## Requisitos

Antes de ejecutar o desarrollar con este proyecto, asegúrate de cumplir con los siguientes requisitos:

- **.NET Framework** v4.8.
- **PI AF SDK** (Asset Framework SDK de OSIsoft).
- **NLog** (librería de logging).
- Acceso a un **sistema PI** configurado.
- **Windows 10/11** o **Windows Server** con privilegios administrativos.

## Configuración

Configura el archivo `appsettings.yml` (o appsettings.json) con los detalles de conexión y configuración de tu `PI System`. Aquí puedes establecer las conexiones a las bases de datos de PI, configuraciones de eventos y otros parámetros importantes para la operación del servicio.

## Uso 

- El servicio se ejecutará en segundo plano, monitoreando eventos y generando logs de diagnóstico y de eventos.
- Los logs generados se almacenan utilizando NLog y pueden ser configurados en el archivo `NLog.config` según las preferencias (por ejemplo, registrar en archivos, bases de datos o sistemas de monitoreo externos).

## Log

El servicio usa NLog para registrar eventos y errores. Los logs se almacenan según la configuración definida en el archivo `NLog.config`. Puedes personalizar el nivel de log y los destinos (por ejemplo, archivo de texto, base de datos, etc.).

## Contribuciones

Este proyecto es desarrollado y mantenido internamente por **Contac Ingenieros Ltda**. Si deseas contribuir o tienes preguntas, por favor contacta al equipo de desarrollo o a los administradores del sistema.

## Licencia

Este proyecto es privado y es propiedad exclusiva de **Contac Ingenieros Ltda**. No tiene una licencia pública y su uso está restringido a los empleados de la empresa o personas autorizadas.

## Soporte

Si necesitas soporte o tienes preguntas sobre el uso o configuración del servicio, por favor contacta al equipo de soporte interno de **Contac Ingenieros Ltda**.

## Instalación

### 1. Clonar el Repositorio
Clona el repositorio desde GitLab o GitHub a tu máquina local:

```bash
git clone https://gitlab.com/contac/pipointstatuswatchtower.git
