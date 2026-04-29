# PIPointSatusWatchtower

# Changelog

## [v1.0.0] - 07-12-2024

#### Satus
- Diseñado como aplicacion de consola.
- La lógica del proyecto se encuentra exclusivamente en el archivo `Program.cs`.
- No se ha aplicado ninguna arquitectura definida en el proyecto.
- Se ha añadido una extensión del archivo de configuración en formato `.json`.

## [v1.1.0] - 10-12-2024

#### Added
- Nueva funcionalidad para imprimir logs a través de una función en `StringUtils`.
- Nuevas variables agregadas al archivo de configuración `appsettings.yml`.
- Nuevo instalador del proyecto: **Setup v1.1.0**.

#### Changed
- Diseñado como servico de windows.
- Cambio de extensión del archivo de configuración de `.json` a `.yml`.
- Reestructuración general del proyecto, implementando la **Clean Architecture**.

## [v1.2.0] - 13-12-2024

### Added
- Se añadió la funcionalidad para obtener la cantidad de valores repetidos de `InstrumentTag` en el servicio `Counter`, específicamente en la clase `TypeValueCounterService.cs`.
- Implementación de la escritura en la base de datos AF bajo el nombre `totalDuplicateValuesTagName`.
- Nuevo instalador del proyecto: **Setup v1.2.0**.

## [v1.3.0] - 16-12-2024

### Added
- Se integro la salida de la solución al los `Custome Actions` del instalador.
- Nuevo instalador del proyecto: **Setup v1.3.0**.

### Changed
- Se cambió la forma en que se valida la conexión a **PI SYSTEM**, agregando *loggers* informativos.
- Se corrigió la lectura de los datos del archivo `appsettings.yml`.
- Se actualizó la metodología para obtener valores repetidos de `InstrumentTag`.


## [v1.3.1] - 17-12-2024

### Added
- Se agrego un metodología para validar el atributo `InstrumentTag` de los PIPoints.
- Se agregaron *loggers* informativos que validan el atributo `InstrumentTag`.
- Nuevo instalador del proyecto: **Setup v1.3.1**.


## [v2.0.0] - 18-12-2024

### Added
- Se implementó un bucle en el servicio que permite su ejecución según el valor definido en la variable `ExecutionIntervalInMinutes` ubicada en `appsettings.yml`.
- Se agregó el servicio de procesamiento, donde la clase `ProcessingPointSourceService.cs` y su método `StartProcessingPointSource` realizan la evaluación y el registro de datos.
- Nuevo instalador del proyecto: **Setup v2.0.0**.

### Changed
- Se modificó el diseño del software para optimizar el proceso de inicio del servicio.


## [v2.1.0] - 22-12-2024

### Added
- Nuevo instalador del proyecto: **Setup v2.1.0**.

### Changed
- Se agrego un variable boleana `serviceDetails` en `appsettings.yml`,  la cual al establecerse en true, se genera un registro detallado y completo en los logs.

## [v2.1.1] - 17-01-2025

### Added
- Nuevo instalador del proyecto: **Setup v2.1.1**.

### Changed
- Se modificó los nombres de las variables que escriben a `AFServer`.
- BadState -> BadPointsCount
- GoodState -> GoodPointsCount
- BadPoints -> BadPointsPct
- GoodPoints -> GoodPointsPct


## [v2.2.0] - 17-01-2025

### Added
- Nuevo instalador del proyecto: **Setup v2.2.0**.

### Changed
- `StatesConfig` en `appsettings.yml` ahora utiliza un diccionario dinámico en lugar de una estructura fija, permitiendo configuraciones más flexibles.

## [v2.2.1] - 17-01-2025

### Added
- Nuevo instalador del proyecto: **Setup v2.2.1**.

### Changed
- Modificado el log para mostrar el valor del tag cuando este no existe.
- Actualizados los nombres de las variables que escriben en el servidor AF para mejorar la claridad y alinearlos con las convenciones de nomenclatura.
- BadState -> BadPointsCount
- GoodState -> GoodPointsCount
- BadPoints -> PointCount
- GoodPoints -> DuplicateCount  

## [v2.2.2] - 20-01-2025

### Added
- Nuevo instalador del proyecto: **Setup v2.2.2**.

### Changed
- Redondear la precion del timestamp que escribe a AFserver.
- Correcciones de sintaxis en `appsettings.yml` para variables con caracteres especiales.

## [v2.3.0] - 21-01-2025

### Added
- Nuevo instalador del proyecto: **Setup v2.3.0**.

### Changed
- Renombradas variables y valores predeterminados en `appsetting.yml`.
- Implementadas estrategias para manejar errores de configuración al iniciar el servicio.
- Modificada lógica del código para manejar la ausencia de un PointSource.

## [v2.4.0] - 22-01-2025

### Added
- Nuevo instalador del proyecto: **Setup v2.4.0**.

### Changed
- Cambio estructura en appsetting.yml para gestionar dos listas: EnabledStates (PIPoints que se ejecutan) e DisabledStates (PIPoints que no se ejecutan).

## [v2.5.0] - 24-01-2025

### Added
- Nuevo instalador del proyecto: **Setup v2.5.0**.

### Changed
- Se habilito la insensibilidad a mayúsculas/minúsculas en el mecanismo de conteo de duplicados de InstrumentTags.

## [v2.6.0] - 29-01-2025

### Added
- Nuevo instalador del proyecto: **Setup v2.6.0**.

### Changed
- Migración completa del servicio de Windows a **arquitectura x64**.
- Se actualizó el instalador (`Setup Project`) para soportar `x64`.

## [v2.7.0] - 31-01-2025

### Added
- Nuevo instalador del proyecto: **Setup v2.7.0**.

### Changed
- Ahora se registran en los logs de `NLOG` solo el inicio y el fin de cada ejecución del bucle periódico.


## [v3.0.0] - 03-02-2025

### Added
- Se agregó una nueva lista denominada `PointSourcesDuplicateEnable`, en la cual únicamente se realizará el proceso de búsqueda de duplicados.
- Nuevo instalador del proyecto: **Setup v3.0.0**.

### Changed
- Ahora se registran en los logs de `NLOG` los tagName de los **PIPoint** duplicados.

## [v3.0.1] - 05-02-2025

### Added
- Nuevo instalador del proyecto: **Setup v3.0.1**.

### Changed
- Se anulo el proceso de conteo de `OthersPoint`.
- Actualizados los nombres de las variables que escriben en el servidor AF para mejorar la claridad y alinearlos con las convenciones de nomenclatura.
- BadPointsCount -> Bad Points Count
- GoodPointsCount -> Good Points Count
- PointCount -> Total Point Count
- DuplicateCount -> Duplicate Point Count

## [v3.0.2] - 06-02-2025

### Added
- Nuevo instalador del proyecto: **Setup v3.0.2**.

### Changed
- Se cambió la forma de escritura en AF de `AFUpdateOption.Replace` a `AFUpdateOption.Insert`.
- Se anulo el proceso de conteo de `GoodPoints`.

## [v3.0.3] - 11-02-2025

### Added
- Nuevo instalador del proyecto: **Setup v3.0.3**.

### Changed
- Correcion de logica y metodo de escrituta para lista de `PointSourcesDuplicateEnable`.


## [v3.1.0] - 14-02-2025

### Added
- Nuevo instalador del proyecto: **Setup v3.1.0**.

### Changed
- Actualizar la obtención de valores de `PIPoints` de individual a por lote.
- Optimización y reducción del tiempo de conteo de `BadPoints`.
- Optimización y reducción del tiempo de búsqueda y comparación de `estados`.
- Optimización y reducción del tiempo de búsqueda y comparación de `instrumentag` duplicados.


## [v4.0.0] - 26-04-2026

### Added
- Se implementó el **Modo Legacy** (`LegacyMonitoring`): agrupa PI Points por `PointSource` para monitorear la salud de las interfaces de adquisición de datos. Habilitado mediante `LegacyMonitoring.Enabled = true` en `appsettings.yml`.
- Se implementó el **Modo Business** (`BusinessMonitoring`): agrupa PI Points por contexto funcional usando el atributo `ExDesc`. Habilitado mediante `BusinessMonitoring.Enabled = true` en `appsettings.yml`.
- Se agregó el **Modo de descubrimiento dinámico (Standard)**: el sistema descubre automáticamente grupos de negocio buscando PI Points cuyo `ExDesc` contiene el prefijo `PPSW_GROUP=`.
- Se agregó el **Modo de contextos estáticos (Static)**: permite definir grupos de negocio explícitamente en `appsettings.yml` mediante la lista `Contexts`.
- Se implementó el sistema de **Perfiles de Monitoreo** (`MonitoringProfiles`) con Feature Flags por módulo de calidad (`EnableStaleCheck`, `EnableFlatlineCheck`, `EnableDensityCheck`). Los perfiles se asignan por grupo en `ProfileAssignments` o mediante `DefaultProfile`.
- **Stale Check**: nuevo módulo de análisis de frescura de datos. Evalúa la latencia de cada PI Point con estado Good y la clasifica en niveles configurables (`LatencyBuckets`). Escribe `Stale_Level{N}` y `MaxGroupLatency` como PI Tags.
- **Flatline Check**: nuevo módulo de detección de congelamiento de valores. Utiliza el algoritmo de Welford para calcular la desviación estándar sobre los últimos `FlatlineSampleCount` eventos. Tags Digital, String, Blob y con `Step=1` son excluidos automáticamente. Soporte de exclusión manual mediante `;IGNORE_FLATLINE;` en `ExDesc`. Escribe `Frozen_Count` como PI Tag.
- **Density Check**: nuevo módulo de monitoreo de densidad de eventos por hora. Detecta tags con baja densidad (< `MinEventsPerHour`) y alta densidad (> `MaxEventsPerHour`). Soporte de exclusión manual mediante `;IGNORE_DENSITY;` en `ExDesc`. Escribe `LowDensity_Count` y `HighDensity_Count` como PI Tags.
- Nuevos adaptadores de escritura: `PIStaleMetricsAdapter`, `PIFlatlineMetricsAdapter`, `PIDensityMetricsAdapter`.
- Nueva clase `MonitoringProcessOrchestrator` que coordina ambos modos de operación de forma secuencial por ciclo de ejecución.
- Nueva variable `PIServerWriteName` en `appsettings.yml` para configurar un PI Data Archive de escritura dedicado, separado del servidor de lectura.
- Nuevo instalador del proyecto: **Setup v4.0.0**.

### Changed
- La estructura de `appsettings.yml` fue reorganizada completamente: las secciones `LegacyMonitoring` y `BusinessMonitoring` reemplazan la configuración directa de PointSources anterior.
- Los módulos Stale, Flatline y Density operan exclusivamente sobre PI Points con estado Good, excluyendo automáticamente los puntos Bad del análisis de calidad.
- En la primera ejecución del ciclo de vida del servicio se registra información adicional de diagnóstico: PointSources activos, perfiles de monitoreo resueltos y PI Points con InstrumentTags duplicados.