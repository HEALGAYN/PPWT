# Documentación Técnica Integral

## PiPointStatusWatchtower (PPSW)

---

**Versión del documento:** 1.0  
**Fecha de elaboración:** 18 de abril de 2026  
**Versión del software:** v3.1.0  
**Elaborado por:** Contac Ingenieros Ltda.  
**Destinatario:** Cliente — CONTAC Ingenieros Ltda.  
**Clasificación:** Confidencial — Uso Interno del Proyecto  

---

## Tabla de Contenidos

1. [Documento de Especificación Funcional](#1-documento-de-especificación-funcional)
   - 1.1 [Propósito del Sistema](#11-propósito-del-sistema)
   - 1.2 [Alcance](#12-alcance)
   - 1.3 [Definiciones y Acrónimos](#13-definiciones-y-acrónimos)
   - 1.4 [Requisitos Funcionales](#14-requisitos-funcionales)
   - 1.5 [Reglas de Negocio](#15-reglas-de-negocio)
   - 1.6 [Modos de Operación](#16-modos-de-operación)
   - 1.7 [Métricas Generadas](#17-métricas-generadas)
   - 1.8 [Nomenclatura de PI Tags de Salida](#18-nomenclatura-de-pi-tags-de-salida)
   - 1.9 [Requisitos No Funcionales](#19-requisitos-no-funcionales)
2. [Documentación de Arquitectura](#2-documentación-de-arquitectura)
   - 2.1 [Visión General de la Arquitectura](#21-visión-general-de-la-arquitectura)
   - 2.2 [Diagrama de Capas](#22-diagrama-de-capas)
   - 2.3 [Componentes del Sistema](#23-componentes-del-sistema)
   - 2.4 [Flujo de Ejecución](#24-flujo-de-ejecución)
   - 2.5 [Dependencias Externas](#25-dependencias-externas)
   - 2.6 [Modelo de Despliegue](#26-modelo-de-despliegue)
   - 2.7 [Decisiones Técnicas](#27-decisiones-técnicas)
3. [Documentación de Código](#3-documentación-de-código)
   - 3.1 [Estructura del Proyecto](#31-estructura-del-proyecto)
   - 3.2 [Catálogo de Clases](#32-catálogo-de-clases)
   - 3.3 [Patrones de Diseño Aplicados](#33-patrones-de-diseño-aplicados)
   - 3.4 [Modelos de Datos](#34-modelos-de-datos)
   - 3.5 [Servicios de Calidad (Quality Checks)](#35-servicios-de-calidad-quality-checks)
   - 3.6 [Adaptadores de Escritura (Metrics Adapters)](#36-adaptadores-de-escritura-metrics-adapters)
   - 3.7 [Buenas Prácticas Implementadas](#37-buenas-prácticas-implementadas)
   - 3.8 [Guía de Mantenimiento del Código](#38-guía-de-mantenimiento-del-código)
4. [Guía del Archivo de Configuración (`appsettings.yml`)](#4-guía-del-archivo-de-configuración-appsettingsyml)
   - 4.1 [Ubicación y Formato](#41-ubicación-y-formato)
   - 4.2 [Sección `AppConfig`](#42-sección-appconfig)
   - 4.3 [Sección `LegacyMonitoring`](#43-sección-legacymonitoring)
   - 4.4 [Sección `BusinessMonitoring`](#44-sección-businessmonitoring)
   - 4.5 [Sección `MonitoringProfiles`](#45-sección-monitoringprofiles)
   - 4.6 [Secciones `EnabledStates` y `DisabledStates`](#46-secciones-enabledstates-y-disabledstates)
   - 4.7 [Ejemplo Completo de Configuración](#47-ejemplo-completo-de-configuración)
   - 4.8 [Validaciones Automáticas](#48-validaciones-automáticas)
5. [Control de Versiones del Documento](#5-control-de-versiones-del-documento)

---

## 1. Documento de Especificación Funcional

### 1.1 Propósito del Sistema

**PiPointStatusWatchtower (PPSW)** es un Servicio de Windows desarrollado en **.NET Framework 4.8** cuya finalidad es el **monitoreo automatizado y continuo de la calidad de datos** en el ecosistema PI System de OSIsoft (actualmente AVEVA). El servicio opera como un proceso en segundo plano (daemon) que periódicamente:

1. Consulta el estado actual de miles de PI Points en uno o más PI Data Archives.
2. Clasifica los estados en categorías configurables (Bad, Good, duplicados).
3. Efectúa análisis de calidad avanzados (frescura, congelamiento, densidad de eventos).
4. Escribe las métricas resultantes como PI Tags en un PI Data Archive de escritura designado.

El sistema permite a los equipos de operaciones e ingeniería de datos disponer de **indicadores cuantificables y automatizados** sobre la salud de sus datos en tiempo real, habilitando la toma de decisiones proactiva y la detección temprana de anomalías en la infraestructura de adquisición de datos.

### 1.2 Alcance

El sistema cubre las siguientes capacidades funcionales:

| ID | Capacidad | Descripción |
|:---|:---|:---|
| **CF-01** | Monitoreo de estados de PI Points | Conteo de puntos en estado Bad por tipo de error del sistema digital de PI. |
| **CF-02** | Detección de duplicados por InstrumentTag | Identificación de PI Points que comparten el mismo atributo `InstrumentTag`. |
| **CF-03** | Agrupación por PointSource (Legacy) | Procesamiento de PI Points agrupados por su interfaz de adquisición. |
| **CF-04** | Agrupación por contexto de negocio (Business) | Procesamiento de PI Points agrupados por atributo `ExDesc` con perfiles de calidad. |
| **CF-05** | Análisis de frescura (Stale Check) | Evaluación de latencia de datos con clasificación en umbrales configurables. |
| **CF-06** | Detección de congelamiento (Flatline Check) | Identificación de tags con valores estáticos mediante análisis de varianza estadística. |
| **CF-07** | Monitoreo de densidad de eventos (Density Check) | Detección de tags con exceso o déficit de eventos por hora. |
| **CF-08** | Escritura de métricas a PI Data Archive | Persistencia de resultados como PI Tags con nomenclatura estandarizada. |
| **CF-09** | Logging centralizado | Registro detallado de operaciones mediante NLog con archivado automático. |

**Fuera de alcance:**

- Interfaz gráfica de usuario (GUI).
- Generación de alertas o notificaciones por correo electrónico.
- Creación automática de PI Tags o estructura AF (las estructuras deben preexistir).
- Modificación o eliminación de datos históricos en PI.

### 1.3 Definiciones y Acrónimos

| Término | Definición |
|:---|:---|
| **PI System** | Plataforma de gestión de datos operativos de AVEVA (antiguamente OSIsoft). |
| **PI Data Archive** | Servidor de almacenamiento de series de tiempo del PI System. |
| **PI Point / PI Tag** | Punto de dato individual en el PI Data Archive que almacena una serie de tiempo. |
| **AF (Asset Framework)** | Capa de modelado jerárquico del PI System para organizar activos y atributos. |
| **AFSDK** | Kit de desarrollo de software del Asset Framework (OSIsoft.AF.dll). |
| **PointSource** | Atributo de un PI Point que identifica la interfaz de adquisición de datos que alimenta el punto. |
| **ExDesc** | Extended Descriptor — atributo de texto libre de un PI Point usado para metadatos adicionales. |
| **Snapshot** | Valor más reciente registrado de un PI Point. |
| **Stale** | Condición en la que un dato no se ha actualizado en un período superior al umbral definido. |
| **Flatline / Frozen** | Condición en la que un tag numérico mantiene el mismo valor sin variación. |
| **Density** | Tasa de eventos por hora registrados por un PI Point. |
| **PPSW** | Acrónimo de PiPointStatusWatchtower. |
| **NLog** | Librería de logging para .NET utilizada para registro de eventos. |

### 1.4 Requisitos Funcionales

#### RF-01: Ciclo de Ejecución Periódica

El servicio ejecutará su lógica de monitoreo de forma cíclica con un intervalo configurable definido por el parámetro `ExecutionIntervalInMinutes`. El ciclo se sincroniza al minuto más cercano para garantizar alineación temporal con otros sistemas de reporting.

**Comportamiento:**
- Al iniciar, calcula el próximo punto de ejecución alineado al intervalo configurado.
- Ejecuta la lógica de monitoreo en el punto temporal calculado.
- Al completar la ejecución, calcula el siguiente punto y espera.
- Los errores en un ciclo no detienen el servicio; se registran y el ciclo continúa.

#### RF-02: Conexión a PI Data Archive

El servicio establece y gestiona dos conexiones simultáneas al PI System:

| Conexión | Propósito | Variable de Configuración |
|:---|:---|:---|
| **Lectura** | Recuperación de PI Points y valores actuales. | `PIServerName` |
| **Escritura** | Escritura de métricas calculadas. | `PIServerWriteName` |

Ambas conexiones se establecen al inicio del servicio y se reutilizan durante todo el ciclo de vida.

#### RF-03: Conteo de Estados (Status Count)

Para cada grupo de PI Points procesado, el servicio:

1. Obtiene el valor actual (snapshot) de todos los PI Points del grupo en lotes de 15,000 puntos.
2. Clasifica cada punto como **Good** (valor válido) o **Bad** (error del sistema digital).
3. Para los puntos Bad, identifica el código de estado específico (ej. `Comm Fail`, `Scan Off`, `I/O Timeout`, entre otros) comparando contra la lista `EnabledStates`.
4. Genera contadores por cada estado habilitado.
5. Calcula el total de puntos Bad y el total general de puntos del grupo.

#### RF-04: Detección de InstrumentTags Duplicados

Para grupos con esta funcionalidad habilitada:

1. Recupera el atributo `InstrumentTag` de cada PI Point.
2. La comparación es **case-insensitive** (insensible a mayúsculas/minúsculas).
3. Identifica PI Points que comparten el mismo `InstrumentTag`.
4. En la primera ejecución del ciclo, registra en el log los nombres de los PI Points duplicados.
5. Escribe el conteo total de duplicados como métrica.

#### RF-05: Perfiles de Monitoreo (Feature Flags)

El sistema implementa un mecanismo de **Feature Flags** mediante perfiles de monitoreo que controlan qué módulos de calidad se activan para cada grupo de negocio.

**Resolución del perfil (orden de prioridad):**

1. Buscar asignación explícita en `ProfileAssignments[nombreGrupo]`.
2. Si no existe, usar el perfil indicado en `DefaultProfile`.
3. Si el perfil referenciado no existe en `MonitoringProfiles`, retornar un perfil vacío (todos los flags en `false`).

#### RF-06: Descubrimiento de Grupos de Negocio

**Modo Standard (Dinámico):**
- Busca PI Points cuyo atributo `ExDesc` contiene el prefijo `PPSW_GROUP=`.
- Parsea el nombre del grupo extrayendo el texto posterior al prefijo (hasta el primer espacio).
- Agrupa los PI Points por nombre de grupo descubierto.
- La búsqueda es case-insensitive.

**Modo Static (Configurado):**
- Itera sobre la lista `Contexts` definida en la configuración.
- Para cada contexto, recupera PI Points cuyo atributo `ExDesc` coincide exactamente con `AttributeValue`.

#### RF-07: Stale Check (Evaluación de Frescura de Datos)

**Condición de activación:** `EnableStaleCheck = true` y `LatencyBuckets` no vacío.

**Algoritmo:**

1. Para cada PI Point del grupo con estado **Good** (los Bad se excluyen):
   - Calcular `Lag = UtcNow - Snapshot.Timestamp` (latencia en minutos).
   - Clasificar en cada bucket configurado (acumulativo): si `Lag > bucket[i]`, incrementar el contador del bucket.
   - Actualizar `MaxGroupLatency` si el Lag actual es mayor al máximo registrado.
2. Escribir un PI Tag por cada nivel de latencia (`Stale_Level1`, `Stale_Level2`, ...) y el valor `MaxGroupLatency`.

**Ejemplo:** Con `LatencyBuckets: [15, 60, 240]`, un tag con Lag = 75 minutos incrementa los contadores de Level1 (>15) y Level2 (>60), pero no Level3 (>240).

#### RF-08: Flatline Check (Detección de Valores Congelados)

**Condición de activación:** `EnableFlatlineCheck = true` y `FlatlineSampleCount >= 2`.

**Algoritmo:**

1. **Exclusiones previas (RF-08.2):** Antes de consultar historial, excluir tags que cumplan:
   - `PointType` es Digital, String o Blob.
   - `Step = 1` (señal escalonada legítima).
   - `ExDesc` contiene la etiqueta `;IGNORE_FLATLINE;`.

2. Para cada PI Point no excluido:
   - Recuperar los últimos `FlatlineSampleCount` eventos registrados.
   - Extraer valores numéricos con estado Good.
   - Calcular la desviación estándar poblacional usando el **algoritmo de Welford** para estabilidad numérica.
   - Si `σ < ε` (donde `ε = 1×10⁻⁵`), marcar el tag como **Frozen**.

3. Escribir el conteo total de tags frozen como PI Tag (`Frozen_Count`).

#### RF-09: Density Check (Monitoreo de Densidad de Eventos)

**Condición de activación:** `EnableDensityCheck = true`.

**Algoritmo:**

1. **Exclusiones (RF-09.2):** Excluir tags cuyo `ExDesc` contenga `;IGNORE_DENSITY;`.

2. Para cada PI Point no excluido:
   - Contar eventos registrados en la última hora (`UtcNow - 1h` a `UtcNow`).
   - Si `MinEventsPerHour > 0` y `eventCount < MinEventsPerHour` → incrementar `LowDensityCount`.
   - Si `MaxEventsPerHour > 0` y `eventCount > MaxEventsPerHour` → incrementar `HighDensityCount`.

3. Escribir `LowDensity_Count` y `HighDensity_Count` como PI Tags.

**Nota:** Si `MinEventsPerHour = 0`, se desactiva la validación de baja densidad. Lo mismo aplica para `MaxEventsPerHour = 0` y alta densidad.

### 1.5 Reglas de Negocio

| ID | Regla | Descripción |
|:---|:---|:---|
| **RN-01** | Timestamp redondeado | Todos los valores escritos a PI usan un timestamp redondeado al minuto más cercano (30 seg → arriba, <30 seg → abajo). |
| **RN-02** | Modo de escritura Insert | Las métricas se escriben con `AFUpdateOption.Insert` y `AFBufferOption.Buffer` para no sobrescribir valores existentes y aprovechar el buffer. |
| **RN-03** | Tolerancia a errores | Si un PI Tag de destino no existe, el servicio registra un warning y continúa con el siguiente sin interrumpir la ejecución. |
| **RN-04** | Ejecución alineada al intervalo | El ciclo no se ejecuta "cada N minutos a partir del inicio", sino que se alinea a intervalos exactos del día (ej. para 10 min: :00, :10, :20...). |
| **RN-05** | Procesamiento por lotes | Todas las operaciones masivas usan lotes de 15,000 PI Points para prevenir timeouts en servidores con alto volumen de datos (hasta 100,000 tags). |
| **RN-06** | Primera ejecución especial | En la primera ejecución del ciclo de vida del servicio, se registra información adicional en los logs (listas de PointSources, perfiles, duplicados). |

### 1.6 Modos de Operación

PPSW soporta dos modos de operación que pueden funcionar **simultáneamente**:

#### 1.6.1 Modo Legacy (Monitoreo de Infraestructura)

Orientado a la **salud de las interfaces de adquisición de datos**. Agrupa PI Points por su `PointSource` (ej. `OPC`, `RDBMS`, `UFL`).

**Funcionalidades disponibles:**
- Conteo de estados (Good/Bad por tipo de error).
- Conteo de duplicados (solo para PointSources incluidos en `PointSourcesDuplicateEnable`).

**Habilitación:** `LegacyMonitoring.Enabled = true`.

#### 1.6.2 Modo Business (Monitoreo de Calidad de Negocio)

Orientado a la **calidad de los datos desde la perspectiva del negocio**. Agrupa PI Points por contexto funcional usando el atributo `ExDesc`.

**Funcionalidades disponibles:**
- Conteo de estados (Good/Bad por tipo de error).
- Stale Check (frescura de datos).
- Flatline Check (detección de congelamiento).
- Density Check (densidad de eventos).
- Cada funcionalidad se activa/desactiva por perfil individual.

**Habilitación:** `BusinessMonitoring.Enabled = true`.

### 1.7 Métricas Generadas

#### 1.7.1 Métricas Base (PIMetricsAdapter — Siempre activas)

| Métrica | Descripción | Tipo |
|:---|:---|:---|
| `Status_{NombreEstado}` | Conteo de PI Points en el estado Bad específico. Se genera un tag por cada estado en `EnabledStates`. | Entero |
| `Bad Points Count` | Total de PI Points con estado Bad (cualquier tipo de error). | Entero |
| `Total Point Count` | Total de PI Points en el grupo evaluado. | Entero |
| `Duplicate Point Count` | Total de PI Points con `InstrumentTag` repetido (solo si habilitado). | Entero |

#### 1.7.2 Métricas de Stale Check (PIStaleMetricsAdapter)

| Métrica | Descripción | Tipo |
|:---|:---|:---|
| `Stale_Level{N}` | Conteo de tags con latencia superior al umbral del nivel N. Un tag por cada elemento en `LatencyBuckets`. | Entero |
| `MaxGroupLatency` | Latencia máxima del grupo en minutos (redondeada a 2 decimales). | Decimal |

#### 1.7.3 Métricas de Flatline Check (PIFlatlineMetricsAdapter)

| Métrica | Descripción | Tipo |
|:---|:---|:---|
| `Frozen_Count` | Conteo de tags con desviación estándar ≈ 0 en sus últimos N eventos. | Entero |

#### 1.7.4 Métricas de Density Check (PIDensityMetricsAdapter)

| Métrica | Descripción | Tipo |
|:---|:---|:---|
| `LowDensity_Count` | Conteo de tags con menos eventos/hora que `MinEventsPerHour`. | Entero |
| `HighDensity_Count` | Conteo de tags con más eventos/hora que `MaxEventsPerHour`. | Entero |

### 1.8 Nomenclatura de PI Tags de Salida

Todos los PI Tags de salida siguen la convención:

```
{PIServerWriteName}:PPSWatchtower.{sourceName}.Total.{nombre_métrica}
```

| Segmento | Descripción | Ejemplo |
|:---|:---|:---|
| `PIServerWriteName` | Servidor PI de escritura. | `PISRV01` |
| `PPSWatchtower` | Nombre fijo de la aplicación. | `PPSWatchtower` |
| `sourceName` | Identificador del grupo (PointSource o nombre de grupo de negocio). | `MAG`, `Reporte_Gerencia` |
| `Total` | Nivel de agregación. | `Total` |
| `nombre_métrica` | Nombre de la métrica específica. | `Status_Comm Fail`, `Frozen_Count` |

**Ejemplo completo:**
```
PISRV01:PPSWatchtower.Reporte_Gerencia.Total.Status_Comm Fail
PISRV01:PPSWatchtower.MAG.Total.Bad Points Count
PISRV01:PPSWatchtower.Informe_Ejecutivo.Total.Stale_Level2
```

### 1.9 Requisitos No Funcionales

| ID | Requisito | Especificación |
|:---|:---|:---|
| **RNF-01** | Rendimiento | El servicio debe procesar grupos de hasta 100,000 PI Points sin provocar timeouts en PI Data Archive. |
| **RNF-02** | Disponibilidad | Ejecutar como Servicio de Windows con inicio automático y recuperación ante fallos. |
| **RNF-03** | Logging | Registro completo de operaciones, errores y tiempos de ejecución en archivos de log diarios. |
| **RNF-04** | Configuración dinámica | Toda la parametrización se gestiona mediante archivo YAML sin necesidad de recompilación. |
| **RNF-05** | Compatibilidad | Compatible con Windows 10/11 y Windows Server. Requiere .NET Framework 4.8 y PI AF SDK instalado en la máquina. |
| **RNF-06** | Seguridad | Utiliza la autenticación integrada de Windows (Windows Integrated Security) para la conexión a PI; no almacena credenciales. |

---

## 2. Documentación de Arquitectura

### 2.1 Visión General de la Arquitectura

PiPointStatusWatchtower implementa una **arquitectura por capas** inspirada en los principios de **Clean Architecture**, adaptada al contexto de un servicio de Windows monolítico de propósito específico. La arquitectura prioriza la separación de responsabilidades, la extensibilidad de los módulos de calidad y la configurabilidad sin recompilación.

```
┌─────────────────────────────────────────────────────┐
│                  CAPA DE PRESENTACIÓN                │
│          (Windows Service / Punto de Entrada)        │
│   Program.cs ←→ WinServicePiPointStatus.cs           │
└─────────────────────┬───────────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────────┐
│                 CAPA DE APLICACIÓN                   │
│     (Orquestadores, Procesos, Operaciones)           │
│   MonitoringProcessOrchestrator                      │
│   ├── ProcessLegacy ←→ OperationPointSource          │
│   └── ProcessBusiness                                │
│       ├── OperationStandard (descubrimiento dinámico)│
│       └── OperationStatic   (contextos configurados) │
└─────────────────────┬───────────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────────┐
│                  CAPA DE SERVICIOS                   │
│     (Lógica de dominio, procesamiento de datos)      │
│   ├── TypeValueCounterService (conteo de estados)    │
│   ├── StaleCheckService      (frescura - RF-07)      │
│   ├── FlatlineCheckService   (congelamiento - RF-08)  │
│   ├── DensityCheckService    (densidad - RF-09)       │
│   ├── ExDescParserService    (parseo de grupos)       │
│   ├── RecoverPIPointsService (recuperación de puntos) │
│   └── ResolveMonitoringProfile (resolución de perfil) │
└─────────────────────┬───────────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────────┐
│              CAPA DE INFRAESTRUCTURA                 │
│     (Adaptadores PI, conexiones, escritura)          │
│   ├── PIServerConnectionManager (conexiones PI)      │
│   ├── WriteValueToPITagService  (escritura a PI)     │
│   ├── PIMetricsAdapter          (métricas base)      │
│   ├── PIStaleMetricsAdapter     (métricas stale)     │
│   ├── PIFlatlineMetricsAdapter  (métricas flatline)  │
│   └── PIDensityMetricsAdapter   (métricas density)   │
└─────────────────────┬───────────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────────┐
│             CAPA TRANSVERSAL (Cross-Cutting)         │
│   ├── ConfigManager  (configuración - Singleton)     │
│   ├── LoggerConfig   (logging NLog)                  │
│   ├── MessageCode    (catálogo de mensajes)          │
│   ├── StringUtils    (formateo de strings)           │
│   └── LoggerUtils    (logging condicional)           │
└─────────────────────────────────────────────────────┘
```

### 2.2 Diagrama de Capas

```
┌─────────────────────────────────────────────────────────────┐
│  Windows Service Host (SCM)                                 │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  WinServicePiPointStatus                              │  │
│  │  ┌─────────────────────────────────────────────────┐  │  │
│  │  │  PiPointStatusWatchtowerOperation               │  │  │
│  │  │  (Bucle principal con CancellationToken)        │  │  │
│  │  │  ┌───────────────────────────────────────────┐  │  │  │
│  │  │  │  MonitoringProcessOrchestrator             │  │  │  │
│  │  │  │  ┌───────────────┐ ┌───────────────────┐  │  │  │  │
│  │  │  │  │ ProcessLegacy │ │ ProcessBusiness    │  │  │  │  │
│  │  │  │  │ (PointSource) │ │ (Standard/Static) │  │  │  │  │
│  │  │  │  └───────┬───────┘ └────────┬──────────┘  │  │  │  │
│  │  │  │          │                  │              │  │  │  │
│  │  │  │  ┌───────▼──────────────────▼──────────┐  │  │  │  │
│  │  │  │  │     Servicios de Calidad             │  │  │  │  │
│  │  │  │  │  Counter │ Stale │ Flatline │ Density│  │  │  │  │
│  │  │  │  └───────┬──────────────────┬──────────┘  │  │  │  │
│  │  │  │          │                  │              │  │  │  │
│  │  │  │  ┌───────▼──────────────────▼──────────┐  │  │  │  │
│  │  │  │  │     Adaptadores de Métricas          │  │  │  │  │
│  │  │  │  │  PI │ Stale │ Flatline │ Density     │  │  │  │  │
│  │  │  │  └───────┬─────────────────────────────┘  │  │  │  │
│  │  │  │          │                                 │  │  │  │
│  │  │  │  ┌───────▼─────────────────────────────┐  │  │  │  │
│  │  │  │  │  WriteValueToPITagService            │  │  │  │  │
│  │  │  │  │  (PI Data Archive Write)             │  │  │  │  │
│  │  │  │  └─────────────────────────────────────┘  │  │  │  │
│  │  │  └───────────────────────────────────────────┘  │  │  │
│  │  └─────────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### 2.3 Componentes del Sistema

#### 2.3.1 Punto de Entrada y Servicio de Windows

| Componente | Archivo | Responsabilidad |
|:---|:---|:---|
| `Program` | `Program.cs` | Detecta si la ejecución es interactiva (consola) o como servicio; instancia `WinServicePiPointStatus`. |
| `WinServicePiPointStatus` | `WinServicePiPointStatus.cs` | Hereda de `ServiceBase`. Gestiona `OnStart` y `OnStop` del servicio de Windows. Delega la lógica a `PiPointStatusWatchtowerOperation` en un hilo asíncrono. |

#### 2.3.2 Capa Core (Operaciones)

| Componente | Archivo | Responsabilidad |
|:---|:---|:---|
| `PiPointStatusWatchtowerOperation` | `core/operations/` | Gestiona el bucle de ejecución periódica con `CancellationTokenSource`. Calcula los puntos de ejecución alineados al intervalo y delega al orquestador. |

#### 2.3.3 Capa de Aplicación (Orquestación)

| Componente | Archivo | Responsabilidad |
|:---|:---|:---|
| `MonitoringProcessOrchestrator` | `application/orchestrators/` | Coordina conexiones PI (lectura/escritura), redondea el timestamp y delega a los procesos Legacy y Business. |
| `ProcessLegacy` | `application/services/legacy/process/` | Controlador de flujo para el modo Legacy. Lee la configuración y delega a `CheckPointSource`. |
| `ProcessBusiness` | `application/services/business/process/` | Controlador de flujo para el modo Business. Determina el modo de descubrimiento (Standard/Static) y delega a la operación correspondiente. |
| `OperationPointSource` | `legacy/features/pointSource/operation/` | Ejecuta la lógica de monitoreo para cada PointSource: recuperar PI Points → contar estados → escribir métricas. |
| `OperationStandard` | `business/features/standard/operation/` | Ejecuta el descubrimiento dinámico de grupos por ExDesc, resuelve perfiles y ejecuta módulos de calidad. |
| `OperationStatic` | `business/features/staticExDesc/operation/` | Itera sobre los contextos configurados estáticamente y ejecuta módulos de calidad por perfil. |
| `ResolveMonitoringProfile` | `business/common/monitoring/` | Resuelve el perfil de monitoreo para un grupo siguiendo la cadena de prioridad (asignación → default → vacío). |

#### 2.3.4 Capa de Servicios (Lógica de Dominio)

| Componente | Archivo | Responsabilidad |
|:---|:---|:---|
| `TypeValueCounterService` | `services/counter/` | Conteo de estados (Good/Bad/Duplicate) por grupo. Procesamiento por lotes. |
| `StaleCheckService` | `services/quality/` | Evaluación de frescura con clasificación en buckets multi-umbral. |
| `FlatlineCheckService` | `services/quality/` | Detección de congelamiento con exclusiones y algoritmo de Welford. |
| `DensityCheckService` | `services/quality/` | Monitoreo de densidad de eventos en ventana de 1 hora. |
| `ExDescParserService` | `services/parser/` | Extracción de nombres de grupo desde ExDesc usando expresiones regulares. |
| `RecoverPIPointsService` | `services/recover/` | Recuperación de PI Points por PointSource, InterfaceID, atributo exacto o comodín. |

#### 2.3.5 Capa de Infraestructura (Persistencia y Conectividad)

| Componente | Archivo | Responsabilidad |
|:---|:---|:---|
| `PIServerConnectionManager` | `infrastructure/connections/` | Pool de conexiones a PI Data Archive con patrón Dictionary. Reutiliza conexiones existentes. |
| `WriteValueToPITagService` | `services/write/` | Escritura de valores individuales a PI Tags. Maneja la inexistencia de tags con warning. |
| `PIMetricsAdapter` | `infrastructure/adapters/` | Formatea y escribe métricas de estado (Status, Bad Count, Total Count, Duplicate Count). |
| `PIStaleMetricsAdapter` | `infrastructure/adapters/` | Formatea y escribe métricas de Stale Check (Stale_Level, MaxGroupLatency). |
| `PIFlatlineMetricsAdapter` | `infrastructure/adapters/` | Formatea y escribe métricas de Flatline Check (Frozen_Count). |
| `PIDensityMetricsAdapter` | `infrastructure/adapters/` | Formatea y escribe métricas de Density Check (LowDensity_Count, HighDensity_Count). |

### 2.4 Flujo de Ejecución

```
[Inicio del Servicio]
       │
       ▼
[1] ConfigManager carga appsettings.yml (Singleton)
       │
       ▼
[2] WinServicePiPointStatus.OnStart()
       │
       ▼
[3] PiPointStatusWatchtowerOperation.StartLoopPointSource()
       │
       ▼
[4] MonitoringProcessOrchestrator instancia conexiones PI (Read + Write)
       │
       ▼
[5] ════╗  BUCLE PERIÓDICO (cada ExecutionIntervalInMinutes) ═══════
       ║                                                           ║
       ║  [5.1] Calcular próximo punto de ejecución alineado       ║
       ║  [5.2] Esperar hasta el punto calculado (Task.Delay)      ║
       ║  [5.3] Redondear AFTime al minuto más cercano             ║
       ║                                                           ║
       ║  [5.4] ── ProcessLegacy ──────────────────────────────    ║
       ║  │   Para cada PointSource en lista configurada:           ║
       ║  │     → RecoverPIPoints (por PointSource)                 ║
       ║  │     → TypeValueCounter (conteo estados + duplicados)    ║
       ║  │     → PIMetricsAdapter.Start() (escribir métricas)     ║
       ║  │                                                         ║
       ║  [5.5] ── ProcessBusiness ────────────────────────────    ║
       ║  │   Según DiscoveryMode (Standard/Static):                ║
       ║  │     → Descubrir/Iterar grupos de negocio                ║
       ║  │     → Para cada grupo:                                  ║
       ║  │       → TypeValueCounter (conteo estados)               ║
       ║  │       → ResolveMonitoringProfile (obtener perfil)       ║
       ║  │       → Si EnableStaleCheck:                            ║
       ║  │           StaleCheckService.Evaluate()                  ║
       ║  │           PIStaleMetricsAdapter.Start()                 ║
       ║  │       → Si EnableFlatlineCheck:                         ║
       ║  │           FlatlineCheckService.Evaluate()               ║
       ║  │           PIFlatlineMetricsAdapter.Start()              ║
       ║  │       → Si EnableDensityCheck:                          ║
       ║  │           DensityCheckService.Evaluate()                ║
       ║  │           PIDensityMetricsAdapter.Start()               ║
       ║  │       → PIMetricsAdapter.Start() (métricas base)       ║
       ║  │                                                         ║
       ║  [5.6] Registrar tiempo transcurrido en log                ║
       ║                                                           ║
       ╚═══════════════════════════════════════════════════════════╝
       │
       ▼
[6] WinServicePiPointStatus.OnStop()
       │
       ▼
[7] CancellationToken señala detención → Desconexión de PI Servers
```

### 2.5 Dependencias Externas

#### 2.5.1 Librerías NuGet

| Librería | Versión | Propósito |
|:---|:---|:---|
| `NLog` | 5.3.4 | Motor de logging estructurado con archivado automático. |
| `NLog.Config` | 4.7.15 | Soporte de configuración XML para NLog. |
| `NLog.Schema` | 5.3.4 | Esquema XSD para validación de NLog.config. |
| `YamlDotNet` | 16.3.0 | Serialización/deserialización de archivos YAML para la configuración. |
| `System.Text.Json` | 9.0.1 | Serialización JSON (soporte auxiliar). |
| `Microsoft.Bcl.AsyncInterfaces` | 9.0.1 | Soporte de interfaces asíncronas para .NET Framework. |

#### 2.5.2 Dependencias del Sistema

| Dependencia | Versión | Propósito |
|:---|:---|:---|
| **.NET Framework** | 4.8 (4.7.2 target) | Runtime de ejecución. |
| **PI AF SDK** | 2.x+ (OSIsoft.AF.dll) | Acceso programático al PI System (PI Data Archive y Asset Framework). |
| **Windows OS** | 10/11 o Server 2016+ | Sistema operativo host para el servicio. |

### 2.6 Modelo de Despliegue

```
┌─────────────────────────────────────────────────┐
│           Servidor de Aplicación                │
│         (Windows Server / Windows 10+)          │
│                                                 │
│  ┌───────────────────────────────────────────┐  │
│  │  Servicio de Windows: PPSWatchtower       │  │
│  │  ├── PiPointStatusWatchtower.exe          │  │
│  │  ├── src/resources/appsettings.yml        │  │
│  │  ├── NLog.config                          │  │
│  │  ├── Logs/                                │  │
│  │  │   ├── log_2026-04-18.txt               │  │
│  │  │   └── archived/                        │  │
│  │  └── [DLLs: OSIsoft.AF, NLog, YamlDotNet] │  │
│  └───────────────────┬───────────────────────┘  │
│                      │ AFSDK (TCP/IP)           │
└──────────────────────┼──────────────────────────┘
                       │
          ┌────────────┼────────────┐
          │            │            │
   ┌──────▼──────┐  ┌──▼─────────┐ │
   │ PI Data     │  │ PI Data    │ │
   │ Archive     │  │ Archive    │ │
   │ (LECTURA)   │  │ (ESCRITURA)│ │
   │ PIServerName│  │PIServerWr..│ │
   └─────────────┘  └────────────┘ │
                                   │
                    ┌──────────────▼┐
                    │ PI AF Server  │
                    │ (Estructura   │
                    │  de Activos)  │
                    └───────────────┘
```

**Nota:** El PI Data Archive de lectura y escritura pueden ser el mismo servidor o servidores diferentes según la topología del cliente.

### 2.7 Decisiones Técnicas

| Decisión | Justificación |
|:---|:---|
| **.NET Framework 4.8** en lugar de .NET 6+ | PI AF SDK (OSIsoft.AF.dll) no ofrece soporte oficial para .NET Core/.NET 6+ al momento del desarrollo. El framework 4.8 garantiza compatibilidad total con AFSDK. |
| **Servicio de Windows** en lugar de Worker Service | Consistencia con la infraestructura existente del cliente y compatibilidad directa con `ServiceBase` de .NET Framework. |
| **YAML** para configuración | Mayor legibilidad para archivos de configuración complejos con listas y diccionarios anidados, comparado con JSON o XML. |
| **Singleton para ConfigManager** | Garantiza una única instancia de configuración cargada en memoria, evitando lecturas repetidas de disco y asegurando consistencia. |
| **Procesamiento por lotes (15,000)** | Equilibrio empírico entre rendimiento y consumo de memoria. Previene timeouts en `PIPointList.CurrentValue()` con volúmenes altos. |
| **Algoritmo de Welford** para desviación estándar | Estabilidad numérica superior al cálculo convencional, especialmente con valores de magnitudes extremas. |
| **Insert en lugar de Replace** | Preserva el historial de métricas. Si el servicio se ejecuta dos veces en el mismo minuto, ambos valores se conservan. |
| **Feature Flags por perfil** | Permite activar/desactivar módulos de calidad granularmente por grupo sin modificar código ni reiniciar el servicio. |

---

## 3. Documentación de Código

### 3.1 Estructura del Proyecto

```
PiPointStatusWatchtower/
├── Program.cs                          # Punto de entrada
├── ProjectInstaller.cs                 # Instalador del servicio Windows
├── packages.config                     # Dependencias NuGet
├── NLog.config                         # Configuración de logging (base)
├── App.config                          # Configuración .NET Framework
├── PiPointStatusWatchtower.csproj      # Archivo de proyecto
│
└── src/
    ├── resources/
    │   └── appsettings.yml             # Configuración principal del servicio
    │
    └── main/
        ├── WinServicePiPointStatus.cs  # Clase del servicio Windows
        │
        ├── core/
        │   └── operations/
        │       └── PiPointStatusWatchtowerOperation.cs  # Bucle principal
        │
        ├── application/
        │   ├── orchestrators/
        │   │   └── MonitoringProcessOrchestrator.cs     # Orquestador central
        │   │
        │   └── services/
        │       ├── legacy/
        │       │   ├── process/
        │       │   │   └── ProcessLegacy.cs             # Controlador Legacy
        │       │   └── features/
        │       │       ├── pointSource/
        │       │       │   ├── check/
        │       │       │   │   └── CheckPointSource.cs  # Verificador PointSource
        │       │       │   └── operation/
        │       │       │       └── OperationPointSource.cs  # Operación PointSource
        │       │       └── interfaceID/
        │       │           ├── check/
        │       │           │   └── CheckInterfaceID.cs   # (Reservado)
        │       │           └── operation/
        │       │               └── OperationInterfaceID.cs  # (Reservado)
        │       │
        │       └── business/
        │           ├── process/
        │           │   └── ProcessBusiness.cs            # Controlador Business
        │           ├── common/
        │           │   └── monitoring/
        │           │       └── ResolveMonitoringProfile.cs  # Resolución de perfiles
        │           └── features/
        │               ├── standard/
        │               │   └── operation/
        │               │       └── OperationStandard.cs  # Descubrimiento dinámico
        │               └── staticExDesc/
        │                   └── operation/
        │                       └── OperationStatic.cs    # Contextos estáticos
        │
        ├── services/
        │   ├── counter/
        │   │   └── TypeValueCounterService.cs  # Conteo de estados
        │   ├── quality/
        │   │   ├── StaleCheckService.cs        # Frescura (RF-07)
        │   │   ├── FlatlineCheckService.cs     # Congelamiento (RF-08)
        │   │   └── DensityCheckService.cs      # Densidad (RF-09)
        │   ├── parser/
        │   │   └── ExDescParserService.cs      # Parseo de ExDesc
        │   ├── recover/
        │   │   └── RecoverPIPointsService.cs   # Recuperación de PI Points
        │   ├── write/
        │   │   └── WriteValueToPITagService.cs # Escritura a PI Tags
        │   └── af/
        │       └── AFStructureService.cs       # Validación AF (comentado)
        │
        ├── infrastructure/
        │   ├── connections/
        │   │   └── PIServerConnectionManager.cs  # Pool de conexiones PI
        │   └── adapters/
        │       ├── PIMetricsAdapter.cs           # Adaptador métricas base
        │       ├── PIStaleMetricsAdapter.cs      # Adaptador métricas Stale
        │       ├── PIFlatlineMetricsAdapter.cs   # Adaptador métricas Flatline
        │       └── PIDensityMetricsAdapter.cs    # Adaptador métricas Density
        │
        ├── models/
        │   ├── AppSettingsModel.cs               # Modelos de configuración
        │   ├── GroupingMode.cs                   # Enum de modos de agrupación
        │   ├── TypesValueAndStateCountsModel.cs  # Modelo conteo de estados
        │   ├── StaleCheckResult.cs               # Modelo resultado Stale
        │   ├── FlatlineCheckResult.cs            # Modelo resultado Flatline
        │   └── DensityCheckResult.cs             # Modelo resultado Density
        │
        ├── config/
        │   ├── ConfigManager.cs                  # Singleton de configuración
        │   └── LoggerConfig.cs                   # Configuración de NLog
        │
        ├── constants/
        │   ├── Constants.cs                      # Constantes globales
        │   └── MessageCode.cs                    # Catálogo de mensajes
        │
        └── utils/
            ├── StringUtils.cs                    # Utilidades de formato
            └── LoggerUtils.cs                    # Logging condicional
```

### 3.2 Catálogo de Clases

#### 3.2.1 `ConfigManager` — Gestión de Configuración

**Patrón:** Singleton (`Lazy<T>` thread-safe).

**Responsabilidades:**
- Carga y deserializa `appsettings.yml` al iniciar usando `YamlDotNet`.
- Valida campos obligatorios (`PIServerName`, `ExecutionIntervalInMinutes`, `PointSources`, `EnabledStates`).
- Si la validación falla, registra el error y detiene el proceso con `Environment.Exit(1)`.
- Provee acceso global a `CurrentConfig` y `Logger`.

**Propiedades públicas:**

| Propiedad | Tipo | Descripción |
|:---|:---|:---|
| `Instance` | `ConfigManager` | Instancia singleton. |
| `CurrentConfig` | `Config` | Configuración deserializada desde YAML. |
| `Logger` | `NLog.Logger` | Instancia del logger para toda la aplicación. |

#### 3.2.2 `PiPointStatusWatchtowerOperation` — Bucle Principal

**Responsabilidades:**
- Gestiona el ciclo de vida del bucle de ejecución con `CancellationTokenSource`.
- Calcula puntos de ejecución alineados al intervalo configurado.
- Delega cada ciclo a `MonitoringProcessOrchestrator.StartAsync()`.
- Captura excepciones por ciclo sin detener el servicio.

**Métodos públicos:**

| Método | Descripción |
|:---|:---|
| `StartLoopPointSource()` | Inicia el bucle asíncrono de monitoreo. |
| `StopLoopPointSource()` | Cancela el bucle y desconecta los servidores PI. |

#### 3.2.3 `MonitoringProcessOrchestrator` — Orquestador Central

**Responsabilidades:**
- Establece conexiones a PI Data Archive (lectura y escritura).
- Instancia `WriteValueToPITagService` para escritura de métricas.
- Redondea el timestamp al minuto más cercano.
- Invoca secuencialmente `ProcessLegacy` y luego `ProcessBusiness`.

#### 3.2.4 `TypeValueCounterService` — Conteo de Estados

**Tipo:** Servicio estático.

**Método principal:** `TypeValueCounter(piPoints, counterDuplicate, pointSource, isFirstExecution)`

**Comportamiento:**
1. Procesa PI Points en lotes de 15,000 usando `PIPointList.CurrentValue()`.
2. Para cada valor Bad, busca el código de estado en el diccionario invertido `ReverseStates` (valor numérico → nombre del estado).
3. Si `counterDuplicate = true`, ejecuta `CounterDuplicate()` que agrupa por `InstrumentTag` (case-insensitive) y retorna el total de duplicados.

#### 3.2.5 `RecoverPIPointsService` — Recuperación de PI Points

**Métodos:**

| Método | Filtro Utilizado | Uso |
|:---|:---|:---|
| `RecoverPIPoints(pointSource, piServer)` | `PointSource = valor` | Modo Legacy. |
| `RecoverPIPointsByInterfaceID(interfaceId, piServer)` | `Location1 = valor` | Reservado. |
| `RecoverPIPointsByAttribute(attrName, attrValue, piServer)` | `atributo = valor exacto` | Modo Static. |
| `RecoverPIPointsByContainsValue(attrName, attrValue, piServer)` | `atributo = *valor*` (comodín) | Modo Standard. |

### 3.3 Patrones de Diseño Aplicados

| Patrón | Implementación | Beneficio |
|:---|:---|:---|
| **Singleton** | `ConfigManager` con `Lazy<T>` | Acceso global thread-safe a la configuración sin cargas repetidas. |
| **Strategy** | `ProcessBusiness` selecciona `OperationStandard` u `OperationStatic` según `DiscoveryMode`. | Extensibilidad para nuevos modos de descubrimiento. |
| **Template Method** | `OperationStandard` y `OperationStatic` comparten la secuencia: recuperar → contar → resolver perfil → ejecutar calidad → escribir. | Consistencia en el flujo de procesamiento. |
| **Adapter** | `PIMetricsAdapter`, `PIStaleMetricsAdapter`, `PIFlatlineMetricsAdapter`, `PIDensityMetricsAdapter`. | Desacopla la lógica de cálculo de la lógica de escritura/nomenclatura. |
| **Connection Pool** (simplificado) | `PIServerConnectionManager` con `Dictionary<string, PIServer>`. | Reutilización de conexiones y prevención de aperturas múltiples. |
| **Feature Flags** | `MonitoringProfile` con propiedades booleanas por módulo. | Activación granular de funcionalidades sin cambios de código. |

### 3.4 Modelos de Datos

#### 3.4.1 `Config` — Raíz de Configuración

```csharp
public class Config
{
    public AppConfig AppConfig { get; set; }
    public LegacyMonitoringConfig LegacyMonitoring { get; set; }
    public BusinessMonitoringConfig BusinessMonitoring { get; set; }
    public Dictionary<string, MonitoringProfile> MonitoringProfiles { get; set; }
    public Dictionary<string, int> EnabledStates { get; set; }
    public Dictionary<string, int> DisabledStates { get; set; }
}
```

#### 3.4.2 `MonitoringProfile` — Perfil de Calidad

```csharp
public class MonitoringProfile
{
    public bool EnableStaleCheck { get; set; }       // RF-07 Frescura
    public bool EnableFlatlineCheck { get; set; }    // RF-08 Congelamiento
    public bool EnableDensityCheck { get; set; }     // RF-09 Densidad
    public List<int> LatencyBuckets { get; set; }    // Umbrales en minutos
    public int FlatlineSampleCount { get; set; }     // Eventos a analizar (min: 2)
    public int MinEventsPerHour { get; set; }        // Umbral baja densidad (0 = off)
    public int MaxEventsPerHour { get; set; }        // Umbral alta densidad (0 = off)
}
```

#### 3.4.3 Modelos de Resultado

| Modelo | Propiedades Clave | Servicio Productor |
|:---|:---|:---|
| `TypesValueAndStateCountsModel` | `BadPointsCount`, `DuplicateCount`, `StateCounts` (diccionario estado→conteo) | `TypeValueCounterService` |
| `StaleCheckResult` | `BucketCounts` (diccionario umbral→conteo), `MaxGroupLatencyMinutes`, `TotalTagsEvaluated`, `TotalTagsExcluded` | `StaleCheckService` |
| `FlatlineCheckResult` | `FrozenCount`, `TotalTagsEvaluated`, `TotalTagsExcluded` | `FlatlineCheckService` |
| `DensityCheckResult` | `LowDensityCount`, `HighDensityCount`, `TotalTagsEvaluated`, `TotalTagsExcluded` | `DensityCheckService` |

### 3.5 Servicios de Calidad (Quality Checks)

#### 3.5.1 `StaleCheckService` — Frescura de Datos

**Algoritmo detallado:**

```
PARA CADA lote de 15,000 PI Points:
  1. Crear PIPointList y obtener CurrentValue() en batch.
  2. PARA CADA AFValue del resultado:
     a. Si !IsGood → TotalTagsExcluded++ → CONTINUAR
     b. TotalTagsEvaluated++
     c. lagMinutes = (UtcNow - Snapshot.Timestamp).TotalMinutes
     d. Si lagMinutes > maxLagMinutes → maxLagMinutes = lagMinutes
     e. PARA CADA bucket en sortedBuckets:
        Si lagMinutes > bucket → BucketCounts[bucket]++
  3. MaxGroupLatencyMinutes = maxLagMinutes
```

**Nota técnica:** Los contadores de bucket son **acumulativos**. Un tag con Lag = 100 min y buckets [15, 60, 240] incrementará Level1 y Level2, pero no Level3.

#### 3.5.2 `FlatlineCheckService` — Detección de Congelamiento

**Reglas de exclusión (RF-08.2):**

| Regla | Condición | Razón |
|:---|:---|:---|
| Regla 1 | `PointType ∈ {Digital, String, Blob}` | Estos tipos no representan valores numéricos continuos. |
| Regla 2 | `Step = 1` | Señal escalonada legítima (no es un "congelamiento"). |
| Regla 3 | `ExDesc` contiene `;IGNORE_FLATLINE;` | Exclusión explícita por el administrador. |

**Algoritmo de Welford (estabilidad numérica):**

```
mean = 0, m2 = 0, n = 0
PARA CADA valor en valores:
    n++
    delta = valor - mean
    mean += delta / n
    delta2 = valor - mean
    m2 += delta × delta2
varianza = m2 / n
σ = √varianza
SI σ < 1×10⁻⁵ → TAG ES FROZEN
```

#### 3.5.3 `DensityCheckService` — Densidad de Eventos

**Regla de exclusión (RF-09.2):** ExDesc contiene `;IGNORE_DENSITY;`.

**Ventana temporal:** Última hora (`UtcNow - 1h` a `UtcNow`).

**Clasificación:**
- `eventCount < MinEventsPerHour` → **Baja densidad** (posible falta de comunicación o muestreo insuficiente).
- `eventCount > MaxEventsPerHour` → **Alta densidad** (posible ruido, scan rate excesivo).

### 3.6 Adaptadores de Escritura (Metrics Adapters)

Todos los adaptadores comparten el mismo patrón de resolución del nombre del servidor de escritura:

```csharp
var piServerName = ConfigManager.Instance.CurrentConfig.AppConfig.PIServerWriteName
                   ?? ConfigManager.Instance.CurrentConfig.AppConfig.PIServerName;
```

Si `PIServerWriteName` no está configurado, se usa `PIServerName` como fallback.

La escritura se realiza a través de `WriteValueToPITagService`, que opera como sigue:

1. Busca el PI Tag por nombre completo: `PIPoint.FindPIPoint()`.
2. Si el tag existe → escribe el valor con `AFUpdateOption.Insert` y `AFBufferOption.Buffer`.
3. Si el tag **no** existe → registra un **Warning** en el log (no falla).

### 3.7 Buenas Prácticas Implementadas

| Práctica | Implementación |
|:---|:---|
| **Procesamiento por lotes** | Todas las operaciones masivas (CurrentValue, LoadAttributes, RecordedValues) usan lotes de 15,000 elementos para prevenir timeouts de PI. |
| **Logging condicional** | Los logs detallados (`ServiceDetails = true`) solo se generan cuando están explícitamente activados, reduciendo I/O en producción. |
| **Tolerancia a fallos** | Cada tag, cada grupo y cada ciclo tiene su propio bloque try/catch. Un error en un elemento no impide el procesamiento de los siguientes. |
| **Separación de lectura/escritura** | Servidores PI independientes para lectura y escritura, evitando contención de recursos en el Data Archive de producción. |
| **Inmutabilidad de configuración** | La configuración se carga una sola vez al inicio y permanece inmutable durante la ejecución del servicio. |
| **Documentación XML** | Las clases y métodos principales incluyen documentación XML (`<summary>`, `<param>`, `<returns>`) para generación automática de documentación. |

### 3.8 Guía de Mantenimiento del Código

#### Agregar un nuevo módulo de calidad

Para agregar un nuevo análisis de calidad (ej. "Jitter Check"):

1. Crear el modelo de resultado en `models/` (ej. `JitterCheckResult.cs`).
2. Crear el servicio de evaluación en `services/quality/` (ej. `JitterCheckService.cs`) con método `Evaluate()`.
3. Crear el adaptador de escritura en `infrastructure/adapters/` (ej. `PIJitterMetricsAdapter.cs`).
4. Agregar el Feature Flag en `MonitoringProfile` (ej. `EnableJitterCheck`).
5. Agregar la invocación condicional en `OperationStandard.cs` y `OperationStatic.cs`.
6. Agregar los parámetros en `appsettings.yml` bajo `MonitoringProfiles`.

#### Agregar un nuevo modo de agrupación

1. Crear un nuevo paquete en `application/services/legacy/features/` o `business/features/`.
2. Implementar las clases `Check*` y `Operation*` siguiendo el patrón existente.
3. Agregar el caso al `switch` en `ProcessLegacy.cs` o `ProcessBusiness.cs`.

---

## 4. Guía del Archivo de Configuración (`appsettings.yml`)

### 4.1 Ubicación y Formato

| Atributo | Valor |
|:---|:---|
| **Ruta relativa** | `src/resources/appsettings.yml` |
| **Ruta absoluta (en ejecución)** | `{DirectorioInstalación}/src/resources/appsettings.yml` |
| **Formato** | YAML (YAML Ain't Markup Language) |
| **Codificación** | UTF-8 |
| **Librería de parseo** | YamlDotNet 16.3.0 |

**Reglas de formato YAML:**
- La indentación debe ser con **espacios** (no tabulaciones).
- Los strings con caracteres especiales deben estar entre comillas simples (`'valor'`) o dobles (`"valor"`).
- Los comentarios se inician con `#`.

---

### 4.2 Sección `AppConfig`

Parámetros globales del servicio.

```yaml
AppConfig:
  PIServerName: 'PIDataArchiveName'
  ExecutionIntervalInMinutes: 10
  ServiceDetails: false
  PIServerWriteName: 'PIDataArchiveWriteName'
```

| Parámetro | Tipo | Obligatorio | Default | Descripción |
|:---|:---|:---|:---|:---|
| `PIServerName` | `string` | **Sí** | — | Nombre del PI Data Archive de **lectura**. Debe coincidir con el nombre registrado en el Known Servers Table (KST) de la máquina. |
| `ExecutionIntervalInMinutes` | `int` | **Sí** | — | Intervalo de ejecución del ciclo de monitoreo en minutos. Debe ser mayor que 0. Valores típicos: 5, 10, 15, 30. |
| `ServiceDetails` | `bool` | No | `false` | Habilita logs detallados (debug mode). En `true`, registra cada tag escrito, conteos individuales y perfiles resueltos. **Recomendación:** desactivar en producción. |
| `PIServerWriteName` | `string` | No | `PIServerName` | Nombre del PI Data Archive de **escritura**. Si no se especifica, se usa el mismo servidor de lectura. Debe coincidir con el KST. |

**Validaciones automáticas:**
- `PIServerName` no puede ser nulo, vacío o solo espacios.
- `ExecutionIntervalInMinutes` debe ser mayor que 0.
- Si la validación falla, el servicio **se detiene inmediatamente** con código de salida 1.

---

### 4.3 Sección `LegacyMonitoring`

Configuración del modo de monitoreo de infraestructura (Legacy).

```yaml
LegacyMonitoring:
  Enabled: true
  GroupingMode: 'PointSource'
  PointSourcesDuplicateEnable:
    - 'PointSource1'
    - 'PointSource2'
  PointSources:
    - 'PointSource1'
    - 'PointSource2'
    - 'PointSource3'
```

| Parámetro | Tipo | Obligatorio | Default | Descripción |
|:---|:---|:---|:---|:---|
| `Enabled` | `bool` | No | `true` | Activa o desactiva el monitoreo Legacy. |
| `GroupingMode` | `string` | No | `PointSource` | Modo de agrupación de PI Points. Valores: `PointSource` (por interfaz de adquisición) o `InterfaceID` (reservado). |
| `PointSourcesDuplicateEnable` | `List<string>` | No | `[]` | Lista de PointSources para los cuales se ejecuta el conteo de `InstrumentTag` duplicados. Estos PointSources **también** se procesan para conteo de estados. |
| `PointSources` | `List<string>` | **Sí*** | — | Lista de PointSources a monitorear (conteo de estados sin duplicados). *Obligatorio si `Enabled = true`. |

**Nota importante:** Los PointSources listados en `PointSourcesDuplicateEnable` se procesan con detección de duplicados habilitada. Los de `PointSources` se procesan sin detección de duplicados. Si un PointSource aparece en ambas listas, se procesará dos veces.

---

### 4.4 Sección `BusinessMonitoring`

Configuración del modo de monitoreo de calidad por contexto de negocio.

```yaml
BusinessMonitoring:
  Enabled: true
  GroupingAttribute: 'ExDesc'
  DiscoveryMode: 'Standard'
  StandardPrefix: 'PPSW_GROUP='
  DefaultProfile: 'Default'
  ProfileAssignments:
    NombreGrupo1: 'NombrePerfil1'
    NombreGrupo2: 'NombrePerfil2'
  Contexts:
    - NameProfile: 'NombrePerfil1'
      AttributeValue: 'VALOR_EXDESC'
      EnableDuplicateCheck: false
```

| Parámetro | Tipo | Obligatorio | Default | Descripción |
|:---|:---|:---|:---|:---|
| `Enabled` | `bool` | No | `false` | Activa o desactiva el monitoreo Business. |
| `GroupingAttribute` | `string` | No | `ExDesc` | Atributo de PI Point usado para agrupar. Típicamente `ExDesc`. |
| `DiscoveryMode` | `string` | No | `Standard` | Modo de descubrimiento de grupos. `Standard`: parseo dinámico de ExDesc. `Static`: lista fija en `Contexts`. |
| `StandardPrefix` | `string` | No | `PPSW_GROUP=` | Prefijo a buscar en el atributo de agrupación para modo Standard. |
| `DefaultProfile` | `string` | No | `Default` | Nombre del perfil de monitoreo a usar cuando un grupo no tiene asignación explícita. |
| `ProfileAssignments` | `Dict<string,string>` | No | `{}` | Mapeo explícito de nombre de grupo → nombre de perfil. Tiene prioridad sobre `DefaultProfile`. |
| `Contexts` | `List<BusinessContext>` | Condicional | `[]` | Lista de contextos de negocio para modo `Static`. Obligatorio si `DiscoveryMode = 'Static'`. |

**Sub-objeto `BusinessContext`:**

| Campo | Tipo | Descripción |
|:---|:---|:---|
| `NameProfile` | `string` | Nombre identificador del contexto (se usa como `sourceName` en la nomenclatura). |
| `AttributeValue` | `string` | Valor exacto del atributo `ExDesc` para filtrar PI Points. |
| `EnableDuplicateCheck` | `bool` | Si `true`, habilita la detección de duplicados por `InstrumentTag` para este contexto. |

---

### 4.5 Sección `MonitoringProfiles`

Define los perfiles de calidad con Feature Flags que controlan qué módulos se activan por grupo.

```yaml
MonitoringProfiles:
  NombrePerfil:
    EnableStaleCheck: true
    EnableFlatlineCheck: false
    EnableDensityCheck: true
    LatencyBuckets: [15, 60, 240]
    FlatlineSampleCount: 10
    MinEventsPerHour: 6
    MaxEventsPerHour: 3600
```

| Parámetro | Tipo | Default | Descripción |
|:---|:---|:---|:---|
| `EnableStaleCheck` | `bool` | `false` | Activa el análisis de frescura de datos (RF-07). |
| `EnableFlatlineCheck` | `bool` | `false` | Activa la detección de valores congelados (RF-08). |
| `EnableDensityCheck` | `bool` | `false` | Activa el monitoreo de densidad de eventos (RF-09). |
| `LatencyBuckets` | `List<int>` | `[]` | Umbrales de latencia en **minutos**. Se ordenan ascendentemente. Cada valor genera un tag `Stale_Level{N}`. Ejemplo: `[15, 60, 240]` → 3 niveles. |
| `FlatlineSampleCount` | `int` | `10` | Cantidad de eventos recientes a analizar para detectar congelamiento. **Mínimo: 2** (requerido para calcular desviación estándar). Rango recomendado: 10–20. |
| `MinEventsPerHour` | `int` | `0` | Umbral inferior de eventos/hora. Tags con menos eventos se clasifican como baja densidad. **Valor 0 desactiva** la validación de baja densidad. |
| `MaxEventsPerHour` | `int` | `0` | Umbral superior de eventos/hora. Tags con más eventos se clasifican como alta densidad (ruido). **Valor 0 desactiva** la validación de alta densidad. |

**Perfil `Default` recomendado:**

```yaml
  Default:
    EnableStaleCheck: false
    EnableFlatlineCheck: false
    EnableDensityCheck: false
    LatencyBuckets: []
    FlatlineSampleCount: 10
    MinEventsPerHour: 0
    MaxEventsPerHour: 0
```

Este perfil se aplica cuando un grupo no tiene asignación explícita. Con todos los flags en `false`, solo se ejecuta el conteo de estados base.

---

### 4.6 Secciones `EnabledStates` y `DisabledStates`

Mapeo de estados del sistema digital de PI que se monitorean.

```yaml
EnabledStates:
  Bad: 307
  Bad Input: 255
  Calc Failed: 249
  Comm Fail: 313
  Configure: 240
  I/O Timeout: 246
  No Data: 248
  No Result: 212
  Not Connect: 314
  Out of Serv: 312
  Pt Created: 253
  Scan Off: 238
  Set to Bad: 247
  Shutdown: 254
  Unit Down: 213

DisabledStates:
```

| Sección | Tipo | Descripción |
|:---|:---|:---|
| `EnabledStates` | `Dict<string, int>` | Mapa de estados Bad a monitorear. **Clave:** Nombre del estado (visible en la métrica). **Valor:** Código numérico del sistema digital de PI. Se genera un PI Tag `Status_{clave}` por cada entrada. |
| `DisabledStates` | `Dict<string, int>` | Estados conocidos pero excluidos del monitoreo. Sirve como documentación de estados ignorados intencionalmente. Puede estar vacío. |

**Cómo funciona internamente:**
- El servicio invierte el diccionario: `{307 → "Bad", 255 → "Bad Input", ...}`.
- Al evaluar un PI Point con estado Bad, obtiene el código numérico del `AFEnumerationValue` y busca su nombre en el diccionario invertido.
- Si el código existe en `EnabledStates`, incrementa el contador del estado correspondiente.

**Importante:** Los códigos numéricos corresponden a los valores del **sistema digital `SYSTEM`** de PI. Estos valores son estándar en PI Data Archive pero podrían variar si se han personalizado sistemas digitales. Verificar con el administrador de PI.

---

### 4.7 Ejemplo Completo de Configuración

```yaml
# ═══════════════════════════════════════════════════════════════
# PiPointStatusWatchtower — Archivo de Configuración Principal
# ═══════════════════════════════════════════════════════════════

# Configuración Global del Servicio
AppConfig:
  PIServerName: 'PISRV_PRODUCCION'        # PI Data Archive de lectura
  ExecutionIntervalInMinutes: 10           # Ejecutar cada 10 minutos
  ServiceDetails: false                    # false = logs mínimos en producción
  PIServerWriteName: 'PISRV_METRICAS'      # PI Data Archive de escritura

# Monitoreo Legacy — Salud de Interfaces
LegacyMonitoring:
  Enabled: true
  GroupingMode: 'PointSource'
  PointSourcesDuplicateEnable:             # Con detección de duplicados
    - 'OPC'
    - 'UFL_P2AH'
  PointSources:                            # Sin detección de duplicados
    - 'RDBMS'
    - 'AF'
    - 'Lab'

# Monitoreo de Negocio — Calidad por Contexto
BusinessMonitoring:
  Enabled: true
  GroupingAttribute: 'ExDesc'
  DiscoveryMode: 'Standard'
  StandardPrefix: 'PPSW_GROUP='
  DefaultProfile: 'Default'
  ProfileAssignments:
    Reporte_Gerencia: 'Perfil_Gerencial'
    Reporte_Operaciones: 'Perfil_Operaciones'

# Perfiles de Monitoreo — Feature Flags por Grupo
MonitoringProfiles:
  Default:
    EnableStaleCheck: false
    EnableFlatlineCheck: false
    EnableDensityCheck: false
    LatencyBuckets: []
    FlatlineSampleCount: 10
    MinEventsPerHour: 0
    MaxEventsPerHour: 0
  Perfil_Gerencial:
    EnableStaleCheck: true
    EnableFlatlineCheck: true
    EnableDensityCheck: false
    LatencyBuckets: [15, 60, 240]          # 3 niveles de latencia
    FlatlineSampleCount: 10
    MinEventsPerHour: 6
    MaxEventsPerHour: 600
  Perfil_Operaciones:
    EnableStaleCheck: true
    EnableFlatlineCheck: true
    EnableDensityCheck: true
    LatencyBuckets: [5, 15, 60]
    FlatlineSampleCount: 15
    MinEventsPerHour: 12
    MaxEventsPerHour: 3600

# Estados del Sistema Digital de PI a Monitorear
EnabledStates:
  Bad: 307
  Bad Input: 255
  Calc Failed: 249
  Comm Fail: 313
  Configure: 240
  I/O Timeout: 246
  No Data: 248
  No Result: 212
  Not Connect: 314
  Out of Serv: 312
  Pt Created: 253
  Scan Off: 238
  Set to Bad: 247
  Shutdown: 254
  Unit Down: 213

DisabledStates:
  # Sin estados deshabilitados
```

### 4.8 Validaciones Automáticas

El `ConfigManager` ejecuta las siguientes validaciones al iniciar el servicio. **Si cualquiera falla, el servicio se detiene inmediatamente.**

| Validación | Condición de Fallo | Mensaje de Error |
|:---|:---|:---|
| Configuración general | `Config` es `null` después de la deserialización. | `Configuration is null after deserialization.` |
| Servidor PI de lectura | `PIServerName` es `null`, vacío o solo espacios. | `PIServerName is missing or invalid in the configuration.` |
| Intervalo de ejecución | `ExecutionIntervalInMinutes <= 0`. | `ExecutionIntervalInMinutes should be greater than 0.` |
| Lista de PointSources | `LegacyMonitoring.PointSources` es `null`. | `PointSources must contain at least one entry.` |
| Estados habilitados | `EnabledStates` es `null`. | `StatesConfig must contain at least one state mapping.` |

**Errores de formato YAML:** Si el archivo no es YAML válido, se captura `YamlException` y el servicio se detiene con el detalle del error de parseo.

**Archivo no encontrado:** Si `appsettings.yml` no existe en la ruta esperada, se captura `FileNotFoundException` y el servicio se detiene.

---

## 5. Control de Versiones del Documento

| Versión | Fecha | Autor | Cambios |
|:---|:---|:---|:---|
| 1.0 | 18/04/2026 | Contac Ingenieros Ltda. | Versión inicial — Consolidación de Especificación Funcional, Arquitectura, Código y Configuración. |

---

*Este documento es propiedad de **Contac Ingenieros Ltda.** y forma parte del entregable técnico del proyecto PiPointStatusWatchtower. Su distribución y uso están restringidos al personal autorizado del proyecto.*
