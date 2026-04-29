# Release Notes

## PiPointStatusWatchtower (PPSW)

---

**Producto:** PiPointStatusWatchtower  
**Versión actual:** v4.0.0  
**Fecha de publicación:** 26 de abril de 2026  
**Proveedor:** Contac Ingenieros Ltda.  
**Clasificación:** Confidencial — Uso Interno del Proyecto  

---

## Tabla de Contenidos

1. [Prerrequisitos del Sistema](#1-prerrequisitos-del-sistema)
2. [Historial de Versiones](#2-historial-de-versiones)
   - [v4.0.0 — Versión Actual](#v400----26-de-abril-de-2026)
   - [v3.1.0](#v310----14-de-febrero-de-2025)
   - [v3.0.x — Serie de Correcciones](#v303----11-de-febrero-de-2025)
   - [v3.0.0 — Major Release](#v300----03-de-febrero-de-2025)
   - [v2.x — Serie de Madurez](#v270----31-de-enero-de-2025)
   - [v1.x — Serie Inicial](#v110----10-de-diciembre-de-2024)
3. [Convención de Versiones](#3-convención-de-versiones)

---

## 1. Prerrequisitos del Sistema

Antes de instalar o actualizar **PiPointStatusWatchtower**, verifique que el entorno de destino cumpla con los siguientes requisitos de software y sistema operativo. El incumplimiento de alguno de estos requisitos puede provocar fallos en la instalación o en la ejecución del servicio.

### 1.1 Sistema Operativo

| Requisito | Versión Mínima | Notas |
|:---|:---|:---|
| **Microsoft Windows Server** | Windows Server 2016 (64-bit) | Se recomienda Windows Server 2019 o superior para entornos de producción. |
| **Microsoft Windows (escritorio)** | Windows 10 (64-bit, Build 1607+) | Solo para entornos de desarrollo o pruebas. No recomendado para producción. |

> **Importante:** La arquitectura del instalador y del binario es **x64 exclusivamente** (a partir de v2.6.0). No es compatible con sistemas operativos de 32 bits.

### 1.2 PI System (AVEVA)

| Componente | Versión Mínima | Notas |
|:---|:---|:---|
| **PI Data Archive** | 2018 SP3 (3.4.430) | Compatible con versiones 2018 y superiores (2021, 2023). |
| **PI Asset Framework (PI AF) Server** | 2018 SP3 (2.10.x) | Requerido para la gestión de atributos y estructura de activos. |
| **PI AF SDK (OSIsoft.AF.dll)** | 2.10.x o superior | Debe estar instalado en la máquina donde se ejecuta el servicio. Se distribuye como parte del paquete **PI System Client Tools**. |

> **Nota:** El PI Data Archive de lectura y el de escritura pueden ser el mismo servidor o servidores distintos. Ambos deben ser accesibles desde la máquina donde se instala el servicio mediante autenticación de Windows (Windows Integrated Security). No se requieren credenciales adicionales.

### 1.3 Microsoft .NET Framework

| Requisito | Versión | Notas |
|:---|:---|:---|
| **.NET Framework** | **4.8** | Versión mínima y única soportada. Disponible de forma nativa en Windows Server 2019+. Puede requerir instalación manual en Windows Server 2016. |

> Descarga oficial: [Microsoft .NET Framework 4.8](https://dotnet.microsoft.com/en-us/download/dotnet-framework/net48)

### 1.4 PowerShell

| Requisito | Versión | Notas |
|:---|:---|:---|
| **Windows PowerShell** | 5.1 o superior | Requerido si se utilizan scripts de automatización para la instalación, desinstalación o inicio/parada del servicio. Disponible de forma nativa en Windows Server 2016+. |

### 1.5 Permisos y Red

| Requisito | Descripción |
|:---|:---|
| **Privilegios de instalación** | Cuenta con derechos de administrador local para instalar y registrar el servicio de Windows. |
| **Conectividad de red** | El servidor donde se instala PPSW debe tener acceso TCP/IP a los PI Data Archives de lectura y escritura configurados en `appsettings.yml`. |
| **Cuenta de servicio** | El servicio puede ejecutarse bajo la cuenta `LocalSystem` o una cuenta de dominio con acceso autorizado al PI System. |

---

## 2. Historial de Versiones

---

### v4.0.0 — 26 de abril de 2026

**🏗️ Major Release — Arquitectura Dual: Monitoreo Legacy y Business**

Esta es la versión más significativa del sistema desde su creación. Introduce una reestructuración completa del modelo de monitoreo, separándolo en dos modos de operación independientes y simultáneos, e incorpora tres nuevos módulos de análisis avanzado de calidad de datos.

#### Nuevas Funcionalidades

| Componente | Descripción |
|:---|:---|
| **Modo Legacy (Monitoreo de Infraestructura)** | Nuevo modo de operación que agrupa PI Points por su atributo `PointSource` (ej. `OPC`, `RDBMS`, `UFL`). Permite monitorear la salud de cada interfaz de adquisición de datos de forma independiente. Habilitado mediante `LegacyMonitoring.Enabled = true` en `appsettings.yml`. |
| **Modo Business (Monitoreo de Calidad de Negocio)** | Nuevo modo de operación que agrupa PI Points por contexto funcional usando el atributo `ExDesc`. Permite evaluar la calidad de datos desde la perspectiva del negocio, con perfiles de calidad personalizados por grupo. Habilitado mediante `BusinessMonitoring.Enabled = true`. |
| **Descubrimiento dinámico de grupos (Standard Mode)** | El sistema descubre automáticamente los grupos de negocio buscando PI Points cuyo `ExDesc` contenga el prefijo `PPSW_GROUP=`. No requiere configuración manual de grupos. |
| **Grupos estáticos (Static Mode)** | Modo alternativo al descubrimiento dinámico. Permite definir grupos de negocio explícitamente en `appsettings.yml` mediante la lista `Contexts`, indicando el atributo y valor exacto del `ExDesc`. |
| **Perfiles de Monitoreo (Feature Flags)** | Sistema de perfiles definidos en `MonitoringProfiles` que controla qué módulos de calidad se activan para cada grupo de negocio. Cada perfil tiene flags individuales: `EnableStaleCheck`, `EnableFlatlineCheck`, `EnableDensityCheck`. La asignación de perfiles puede hacerse por grupo o mediante un `DefaultProfile`. |
| **Stale Check — Análisis de Frescura de Datos** | Nuevo módulo que evalúa la latencia de los datos de cada PI Point en estado Good. Clasifica los tags en niveles de retraso configurables (`LatencyBuckets`, ej. 15, 60, 240 minutos) y registra la latencia máxima del grupo (`MaxGroupLatency`). Los tags Bad se excluyen del análisis. |
| **Flatline Check — Detección de Congelamiento** | Nuevo módulo que detecta PI Points numéricos cuyos valores no presentan variación estadística. Utiliza el **algoritmo de Welford** para calcular la desviación estándar poblacional sobre los últimos N eventos configurados (`FlatlineSampleCount`). Tags de tipo Digital, String, Blob y tags con `Step=1` son excluidos automáticamente. Los tags pueden marcarse individualmente con `;IGNORE_FLATLINE;` en su `ExDesc` para ser excluidos. |
| **Density Check — Monitoreo de Densidad de Eventos** | Nuevo módulo que evalúa la tasa de eventos registrados por hora para cada PI Point. Detecta tags con baja densidad (menos eventos que `MinEventsPerHour`) y alta densidad (más eventos que `MaxEventsPerHour`). Los tags pueden marcarse con `;IGNORE_DENSITY;` en su `ExDesc` para ser excluidos. |
| **Nuevos adaptadores de escritura** | Se incorporaron tres nuevos adaptadores especializados de escritura a PI: `PIStaleMetricsAdapter` (escribe `Stale_Level{N}` y `MaxGroupLatency`), `PIFlatlineMetricsAdapter` (escribe `Frozen_Count`) y `PIDensityMetricsAdapter` (escribe `LowDensity_Count` y `HighDensity_Count`). |
| **Orquestador central** | Nueva clase `MonitoringProcessOrchestrator` que coordina ambos modos de operación de forma secuencial en cada ciclo de ejecución, con gestión explícita de conexiones de lectura y escritura a PI. |
| **Servidor PI de escritura dedicado** | Nueva variable `PIServerWriteName` en `appsettings.yml` que permite configurar un PI Data Archive de escritura diferente al de lectura, soportando topologías de infraestructura separadas. |

#### Mejoras

| Componente | Descripción |
|:---|:---|
| **Logging de primera ejecución** | En el primer ciclo de vida del servicio se registra información adicional de diagnóstico: lista de PointSources activos, perfiles de monitoreo resueltos por grupo y PI Points con InstrumentTags duplicados. |
| **Exclusión de Bad Points en análisis de calidad** | Los módulos Stale, Flatline y Density operan exclusivamente sobre PI Points con estado Good, evitando falsos positivos en el análisis de calidad. |
| **Tolerancia a PI Tags inexistentes** | Si un PI Tag de destino no existe en el momento de escritura, el servicio registra una advertencia y continúa con el siguiente sin interrumpir el ciclo. |

#### Cambios en `appsettings.yml`

| Sección | Descripción |
|:---|:---|
| **`LegacyMonitoring`** | Nueva sección que reemplaza la configuración directa de PointSources. Contiene `Enabled`, `PointSources` y `PointSourcesDuplicateEnable`. |
| **`BusinessMonitoring`** | Nueva sección con `Enabled`, `DiscoveryMode` (Standard/Static), y la lista de `Contexts` para el modo Static. |
| **`MonitoringProfiles`** | Nuevo diccionario de perfiles, donde cada perfil define los flags de calidad (`EnableStaleCheck`, `EnableFlatlineCheck`, `EnableDensityCheck`) y sus parámetros (`LatencyBuckets`, `FlatlineSampleCount`, `MinEventsPerHour`, `MaxEventsPerHour`). |
| **`ProfileAssignments`** | Nuevo diccionario que asigna un perfil específico a un grupo de negocio por nombre. Los grupos sin asignación explícita usan el `DefaultProfile`. |

#### Instalador

- Nuevo instalador disponible: **Setup v4.0.0**

---

### v3.1.0 — 14 de febrero de 2025

**🚀 Mejoras de Rendimiento — Procesamiento por Lotes**

Esta versión introduce una optimización fundamental en el motor de consulta de PI Points, habilitando el procesamiento masivo de datos sin degradación del rendimiento.

#### Mejoras

| Componente | Descripción |
|:---|:---|
| **Obtención de valores por lote** | La consulta de valores actuales (snapshot) de PI Points migró de un modelo individual a un modelo de **lotes de 15,000 puntos** por llamada a `PIPointList.CurrentValue()`. Esto permite procesar grupos de hasta 100,000 tags sin provocar timeouts en el PI Data Archive. |
| **Conteo de Bad Points** | Optimización del algoritmo de clasificación de estados (Good/Bad), reduciendo significativamente el tiempo de procesamiento en grupos grandes. |
| **Comparación de estados** | Mejora en la eficiencia del mecanismo de búsqueda y comparación contra la lista `EnabledStates`, utilizando estructuras de datos invertidas para acceso O(1). |
| **Detección de InstrumentTags duplicados** | Optimización del algoritmo de agrupación y comparación de `InstrumentTag` (case-insensitive), reduciendo el tiempo de búsqueda en colecciones grandes. |

#### Instalador

- Nuevo instalador disponible: **Setup v3.1.0**

---

### v3.0.3 — 11 de febrero de 2025

#### Correcciones

| Componente | Descripción |
|:---|:---|
| **Lógica de escritura para `PointSourcesDuplicateEnable`** | Corrección de un defecto en el método de escritura asociado a la lista de PointSources habilitados para el conteo de duplicados. Los contadores de duplicados ahora se registran correctamente en el PI Data Archive de escritura. |

#### Instalador

- Nuevo instalador disponible: **Setup v3.0.3**

---

### v3.0.2 — 06 de febrero de 2025

#### Cambios

| Componente | Descripción |
|:---|:---|
| **Modo de escritura en PI** | Cambio del modo de actualización de PI Tags de `AFUpdateOption.Replace` a `AFUpdateOption.Insert`. Los valores ahora se insertan preservando el historial completo de métricas; si el servicio se ejecuta dos veces en el mismo minuto, ambos registros se conservan. |
| **Eliminación del conteo de Good Points** | El conteo de puntos en estado Good fue suprimido del flujo de procesamiento por estar fuera del alcance funcional del sistema. |

#### Instalador

- Nuevo instalador disponible: **Setup v3.0.2**

---

### v3.0.1 — 05 de febrero de 2025

#### Cambios

| Componente | Descripción |
|:---|:---|
| **Nomenclatura de PI Tags de salida** | Renombrado definitivo de las métricas escritas en PI para alinear con los estándares de nomenclatura del proyecto: `BadPointsCount` → `Bad Points Count`, `GoodPointsCount` → `Good Points Count`, `PointCount` → `Total Point Count`, `DuplicateCount` → `Duplicate Point Count`. |
| **Eliminación del conteo `OthersPoint`** | Se anuló el procesamiento del contador de puntos "Others", simplificando el flujo de escritura. |

#### Instalador

- Nuevo instalador disponible: **Setup v3.0.1**

---

### v3.0.0 — 03 de febrero de 2025

**🆕 Major Release — Control Granular de Duplicados**

#### Nuevas Funcionalidades

| Componente | Descripción |
|:---|:---|
| **Lista `PointSourcesDuplicateEnable`** | Nueva sección en `appsettings.yml` que permite especificar explícitamente qué PointSources participan en el proceso de búsqueda de InstrumentTags duplicados. Los PointSources no incluidos en esta lista omiten el análisis de duplicados, reduciendo el tiempo de procesamiento. |
| **Log de PI Points duplicados** | En la primera ejecución del ciclo de vida del servicio, se registra en los logs de NLog la lista completa de nombres de PI Points que presentan InstrumentTags duplicados, facilitando el diagnóstico. |

#### Instalador

- Nuevo instalador disponible: **Setup v3.0.0**

---

### v2.7.0 — 31 de enero de 2025

#### Cambios

| Componente | Descripción |
|:---|:---|
| **Reducción de verbosidad en logs** | El registro de NLog se ajustó para capturar únicamente los eventos de inicio y fin de cada ejecución del bucle periódico, reduciendo el volumen de archivos de log generados en producción. |

#### Instalador

- Nuevo instalador disponible: **Setup v2.7.0**

---

### v2.6.0 — 29 de enero de 2025

**⚙️ Migración a Arquitectura x64**

#### Cambios

| Componente | Descripción |
|:---|:---|
| **Arquitectura del binario** | Migración completa del servicio de Windows a compilación **x64**. Esta migración es requerida para compatibilidad con PI AF SDK en entornos de producción de 64 bits y para soportar mayores volúmenes de memoria. |
| **Instalador x64** | El instalador (`Setup Project`) fue actualizado para desplegar el binario x64 correctamente en sistemas operativos de 64 bits. |

> **Nota de compatibilidad:** A partir de esta versión, el servicio **no es compatible** con sistemas operativos de 32 bits.

#### Instalador

- Nuevo instalador disponible: **Setup v2.6.0**

---

### v2.5.0 — 24 de enero de 2025

#### Cambios

| Componente | Descripción |
|:---|:---|
| **Comparación case-insensitive de InstrumentTags** | El mecanismo de conteo de duplicados de `InstrumentTag` ahora es insensible a mayúsculas y minúsculas. Tags como `TAG_01`, `tag_01` y `Tag_01` se reconocen como el mismo instrumento. |

#### Instalador

- Nuevo instalador disponible: **Setup v2.5.0**

---

### v2.4.0 — 22 de enero de 2025

#### Cambios

| Componente | Descripción |
|:---|:---|
| **Estructura `EnabledStates` / `DisabledStates`** | Reestructuración de la configuración de estados en `appsettings.yml`. Ahora se gestionan dos diccionarios separados: `EnabledStates` (estados a monitorear y contabilizar) y `DisabledStates` (estados excluidos del análisis), brindando mayor control sobre los filtros de calidad. |

#### Instalador

- Nuevo instalador disponible: **Setup v2.4.0**

---

### v2.3.0 — 21 de enero de 2025

#### Cambios

| Componente | Descripción |
|:---|:---|
| **Manejo de errores de configuración** | Se implementaron estrategias de validación al arranque del servicio. Si la configuración en `appsettings.yml` contiene errores críticos, el servicio registra el error y termina con código de salida controlado (`Environment.Exit(1)`). |
| **Renombrado de variables de configuración** | Variables y valores predeterminados en `appsettings.yml` renombrados para mayor consistencia y claridad. |
| **Manejo de PointSource ausente** | Corrección de comportamiento cuando un PointSource configurado no existe en el PI Data Archive; el servicio ahora continúa con el siguiente sin detenerse. |

#### Instalador

- Nuevo instalador disponible: **Setup v2.3.0**

---

### v2.2.2 — 20 de enero de 2025

#### Correcciones

| Componente | Descripción |
|:---|:---|
| **Precisión del timestamp de escritura** | Se implementó el redondeo del timestamp al minuto más cercano (≥30 seg → minuto superior; <30 seg → minuto inferior) para todos los valores escritos en PI, garantizando alineación temporal con otros sistemas de reporting. |
| **Sintaxis de `appsettings.yml`** | Correcciones de sintaxis YAML para variables cuyos valores contienen caracteres especiales (espacios, puntos, guiones). |

#### Instalador

- Nuevo instalador disponible: **Setup v2.2.2**

---

### v2.2.1 — 17 de enero de 2025

#### Correcciones

| Componente | Descripción |
|:---|:---|
| **Log de tag inexistente** | El log de advertencia al intentar escribir en un PI Tag que no existe ahora incluye el nombre completo del tag, facilitando el diagnóstico. |
| **Renombrado de métricas** | Segunda iteración de renombrado de variables de escritura en PI: `BadState` → `BadPointsCount`, `GoodState` → `GoodPointsCount`, `BadPoints` → `PointCount`, `GoodPoints` → `DuplicateCount`. |

#### Instalador

- Nuevo instalador disponible: **Setup v2.2.1**

---

### v2.2.0 — 17 de enero de 2025

#### Cambios

| Componente | Descripción |
|:---|:---|
| **Configuración dinámica de `StatesConfig`** | La sección `StatesConfig` en `appsettings.yml` migró de una estructura fija a un **diccionario dinámico**, permitiendo agregar o eliminar estados a monitorear sin modificar el código fuente. |

#### Instalador

- Nuevo instalador disponible: **Setup v2.2.0**

---

### v2.1.1 — 17 de enero de 2025

#### Correcciones

| Componente | Descripción |
|:---|:---|
| **Renombrado de métricas de escritura en AF** | Primera iteración de renombrado de variables escritas en el PI Data Archive para alinear con la convención de nomenclatura oficial: `BadState` → `BadPointsCount`, `GoodState` → `GoodPointsCount`, `BadPoints` → `BadPointsPct`, `GoodPoints` → `GoodPointsPct`. |

#### Instalador

- Nuevo instalador disponible: **Setup v2.1.1**

---

### v2.1.0 — 22 de diciembre de 2024

#### Cambios

| Componente | Descripción |
|:---|:---|
| **Flag `serviceDetails`** | Se agregó la variable booleana `serviceDetails` en `appsettings.yml`. Cuando se establece en `true`, el servicio genera un registro detallado y expandido en los logs de NLog, incluyendo información de cada PI Point procesado. Útil durante la fase de diagnóstico. |

#### Instalador

- Nuevo instalador disponible: **Setup v2.1.0**

---

### v2.0.0 — 18 de diciembre de 2024

**🆕 Major Release — Ejecución Cíclica Periódica**

#### Nuevas Funcionalidades

| Componente | Descripción |
|:---|:---|
| **Bucle de ejecución periódica** | Implementación del mecanismo de ejecución cíclica controlada por el parámetro `ExecutionIntervalInMinutes` en `appsettings.yml`. El servicio calcula automáticamente el próximo punto de ejecución alineado al intervalo configurado (ej. cada 10 minutos: :00, :10, :20, ...). |
| **Servicio de procesamiento central** | Nueva clase `ProcessingPointSourceService.cs` con el método `StartProcessingPointSource`, que centraliza el flujo de evaluación y escritura de datos por PointSource. |

#### Cambios

| Componente | Descripción |
|:---|:---|
| **Optimización del inicio del servicio** | Rediseño del proceso de inicialización para reducir el tiempo de arranque y mejorar la estabilidad al inicio. |

#### Instalador

- Nuevo instalador disponible: **Setup v2.0.0**

---

### v1.3.1 — 17 de diciembre de 2024

#### Nuevas Funcionalidades

| Componente | Descripción |
|:---|:---|
| **Validación del atributo `InstrumentTag`** | Se incorporó una metodología específica para validar la presencia y el valor del atributo `InstrumentTag` en los PI Points antes de ejecutar el conteo de duplicados. |
| **Logs de validación de `InstrumentTag`** | Registro informativo en NLog con los resultados de la validación del atributo `InstrumentTag` por cada PI Point evaluado. |

#### Instalador

- Nuevo instalador disponible: **Setup v1.3.1**

---

### v1.3.0 — 16 de diciembre de 2024

#### Nuevas Funcionalidades

| Componente | Descripción |
|:---|:---|
| **Custom Actions en instalador** | La salida compilada del proyecto fue integrada a los `Custom Actions` del instalador, garantizando que los binarios más recientes se despliegan automáticamente durante la instalación. |

#### Correcciones

| Componente | Descripción |
|:---|:---|
| **Validación de conexión a PI System** | Mejora en la lógica de validación de la conexión, con mensajes informativos adicionales en los logs para facilitar el diagnóstico de problemas de conectividad. |
| **Lectura de `appsettings.yml`** | Corrección en el proceso de deserialización del archivo de configuración YAML que causaba fallos en ciertos campos con valores especiales. |
| **Metodología de duplicados** | Actualización del algoritmo de detección de `InstrumentTag` duplicados para mayor precisión. |

#### Instalador

- Nuevo instalador disponible: **Setup v1.3.0**

---

### v1.2.0 — 13 de diciembre de 2024

#### Nuevas Funcionalidades

| Componente | Descripción |
|:---|:---|
| **Detección de InstrumentTags duplicados** | Primera implementación del módulo de búsqueda de PI Points con `InstrumentTag` repetido en la clase `TypeValueCounterService.cs`. Calcula el total de duplicados y los registra en el PI Data Archive bajo la variable `totalDuplicateValuesTagName`. |

#### Instalador

- Nuevo instalador disponible: **Setup v1.2.0**

---

### v1.1.0 — 10 de diciembre de 2024

**🆕 Major Release — Reestructuración Arquitectónica**

#### Nuevas Funcionalidades

| Componente | Descripción |
|:---|:---|
| **Servicio de Windows** | Conversión del aplicativo de consola a Servicio de Windows, habilitando la ejecución en segundo plano con inicio automático gestionado por el SCM (Service Control Manager). |
| **Logging con `StringUtils`** | Nueva función en la clase `StringUtils` para el formateo y registro estandarizado de mensajes en los logs. |
| **Archivo de configuración YAML** | Migración del formato de configuración de `.json` a `.yml` con nuevas variables de operación. |

#### Cambios

| Componente | Descripción |
|:---|:---|
| **Clean Architecture** | Reestructuración completa del proyecto aplicando los principios de Clean Architecture: separación en capas (Presentación, Aplicación, Servicios, Infraestructura, Transversal). |

#### Instalador

- Nuevo instalador disponible: **Setup v1.1.0**

---

### v1.0.0 — 07 de diciembre de 2024

**🎉 Versión Inicial**

Lanzamiento inicial del proyecto **PiPointStatusWatchtower** como prueba de concepto funcional.

| Componente | Descripción |
|:---|:---|
| **Aplicación de consola** | Primera versión operativa del sistema como aplicación de consola. La lógica central se encuentra en el archivo `Program.cs`. |
| **Configuración JSON** | Archivo de configuración en formato `.json` para la parametrización básica del servicio. |

---

## 3. Convención de Versiones

El proyecto sigue el estándar **Semantic Versioning (SemVer)**:

```
vMAJOR.MINOR.PATCH
```

| Segmento | Descripción | Ejemplo |
|:---|:---|:---|
| **MAJOR** | Cambios de arquitectura, rediseño completo o ruptura de compatibilidad. | v1→v2, v2→v3 |
| **MINOR** | Nuevas funcionalidades compatibles con versiones anteriores. | v3.0→v3.1 |
| **PATCH** | Correcciones de errores o ajustes menores sin nuevas funcionalidades. | v3.0.0→v3.0.1 |

---

*Documento generado por Contac Ingenieros Ltda. — Uso interno y confidencial.*  
*Para consultas técnicas, contacte al equipo de desarrollo.*
