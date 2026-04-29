# Solicitud de Especificación de Requerimientos - PPSW v4.0.0

## Resumen del requerimiento 
Evolucionar la herramienta de monitoreo PiPointStatusWatchtower (PPSW) desde una aplicación exclusiva de monitoreo de infraestructura hacia un sistema dual que incorpore un modelo de validación de calidad de datos para contextos de negocio. Se requiere la implementación de tres nuevos módulos de análisis avanzado: Frescura (Stale), Congelamiento estadístico (Flatline) y Densidad de eventos (Density), orquestados a través de un sistema de Perfiles de Monitoreo configurables.

## Contexto y Objetivo 
En versiones previas (hasta v3.1.0), el sistema evaluaba exclusivamente los estados (Good/Bad) y duplicados de los PI Points enfocándose en la salud de las interfaces de adquisición (PointSource). El cliente requiere expandir esta capacidad para analizar la calidad de los datos generados desde una perspectiva de negocio.
**Objetivo:** Implementar un Modo de Negocio (Business Monitoring) que analice de manera granular, y mediante Feature Flags, la sanidad de la información temporal, permitiendo identificar latencias anómalas, sensores congelados y tasas de actualización erróneas.

## Alcance 

**Incluye:** 
- Orquestación de un sistema dual: Modo Legacy (Infraestructura) y Modo Business (Negocio).
- Módulos de cálculo avanzado: Stale Check (latencias), Flatline Check (congelamiento mediante desviación estándar), y Density Check (frecuencia de eventos).
- Descubrimiento de contextos de negocio de manera estática y dinámica (por prefijos en PI Tags).
- Sistema de asignación de Feature Flags a través de Perfiles de Monitoreo.
- Soporte para un PI Data Archive de lectura y otro servidor independiente de escritura de métricas.

**No incluye:** 
- Generación de reportes visuales nativos, frontends de administración de configuración o editores visuales de reglas.
- Generación y envío directo de alarmas/notificaciones (e-mails, SMS).
- Intervención y/o eliminación de eventos históricos en PI Data Archive.

## Definiciones 
- **Legacy Monitoring:** Evaluación y agrupamiento tradicional mediante el atributo `PointSource` (salud de la interfaz PI).
- **Business Monitoring:** Agrupamiento basado en el atributo `ExDesc`, diseñado para evaluar contextos y procesos de la planta.
- **Stale Check:** Medición de latencia de datos, clasificándolos en diferentes niveles de retraso (Buckets).
- **Flatline Check:** Evaluación algorítmica (Algoritmo de Welford) de la variación de valores continuos para determinar si un sensor/tag está "congelado".
- **Density Check:** Medición del recuento de eventos registrados en una ventana de tiempo (ej. 1 hora) en comparación con rangos esperados.
- **Perfil de Monitoreo:** Colección de configuraciones activables/desactivables (Feature Flags) asignables a los distintos grupos.

## Reglas del Negocio  
- Los módulos de calidad avanzada (Stale, Flatline, Density) se ejecutarán **exclusivamente** sobre aquellos PI Points cuyo estado más reciente (snapshot) sea *Good*. Los puntos con estado *Bad* quedan excluidos de esta capa para no alterar las métricas estadísticas.
- Un evento o PI Point será excluido del análisis de congelamiento (Flatline) si su tipo es Digital, String, Blob, o posee propiedad escalonada (`Step=1`).
- Soporte para exclusiones explícitas administradas por el usuario, detectando las etiquetas `;IGNORE_FLATLINE;` y `;IGNORE_DENSITY;` configuradas directamente en el `ExDesc` del tag en PI System.
- Cualquier operación masiva que implique análisis histórico de datos o consulta a miles de puntos, se particionará de manera mandatoria en lotes de 15,000 puntos para evadir bloqueos por timeout y prevenir saturación de RAM.

## Requerimientos Funcionales 
*(Detalles que definen qué debe hacer el sistema para satisfacer los objetivos)*

- **RF-1: Orquestación Dual de Monitoreo** 
  El sistema debe soportar y procesar de forma secuencial los modos de monitoreo de Infraestructura (Legacy) y Negocio (Business) dentro del mismo intervalo o ciclo, sin que existan solapamientos destructivos.
- **RF-2: Descubrimiento Dinámico y Estático de Contextos**
  El sistema debe permitir definir grupos de PI Points leyendo contextos fijos (`Static`) en el archivo YAML de configuración, o descubriendo automáticamente los grupos (`Standard`) a través de la búsqueda del prefijo `PPSW_GROUP=` dentro de las propiedades de los tags en AF/PI.
- **RF-3: Gestión de Perfiles (Feature Flags)**
  El sistema debe posibilitar la activación independiente de los módulos de evaluación (Stale, Flatline, Density) asociando un perfil configurable a los distintos grupos de negocio. En caso de no contar con una asignación, debe aplicarse un `DefaultProfile`.
- **RF-4: Stale Check (Métrica de Frescura)**
  El sistema debe poder clasificar la latencia individual de los PI points dentro de umbrales en minutos preconfigurados, publicando el conteo de rezagos y calculando la "Latencia Máxima" por grupo.
- **RF-5: Flatline Check (Detección de Congelamiento)**
  El sistema evaluará un número predefinido de eventos históricos recientes mediante el uso de cálculos de desviación estándar, debiendo poder marcar como "congelados" numéricamente a aquellos PI points cuya varianza sea menor o igual a una tolerancia mínima (≈ 0).
- **RF-6: Density Check (Densidad de Recepción)**
  El sistema deberá registrar y comparar la cantidad de eventos arribados en la última hora, exponiendo dos métricas: deficiencia de datos respecto al límite inferior (LowDensity), y saturación de eventos respecto al límite superior (HighDensity).
- **RF-7: Escritura Desacoplada en Servidores PI**
  El sistema debe permitir definir vía configuración un PI Data Archive designado a las lecturas (monitoreo) y uno distinto (`PIServerWriteName`) asignado a las escrituras de métricas.

## Requerimientos No Funcionales 

- **Performance Lotes:** El servicio debe ser capaz de operar con volúmenes de hasta 100,000 PI Points segmentando internamente en lotes (15,000), garantizando que los cálculos estadísticos en memoria, como el algoritmo de Welford, no aumenten el tiempo de carga y procesamiento por sobre el umbral general del periodo cíclico asignado.
- **Resiliencia (Manejo de Errores):** La inexistencia de un servidor de escritura o desconexiones en las transacciones no debe provocar una detención (Crash) total del ciclo; en su lugar, se emitirán mensajes tipo "Warning" en NLog y se intentará el reprocesamiento en el periodo siguiente.
- **Configurabilidad en Caliente:** El comportamiento de los análisis de calidad debe ser regido íntegramente por `appsettings.yml`, no requiriendo la re-compilación del código fuente.

## Implicancia en recursos existentes – Backend

- **(RF-1)** Incorporación de `MonitoringProcessOrchestrator` como coordinador central unificando lógicas separadas preexistentes.
- **(RF-2)(RF-3)** Creación e implementación de componentes orquestadores de dominio en las sub-rutas de `/business/features`, así como las rutinas de resolución de perfiles.
- **(RF-4)(RF-5)(RF-6)** Implementación de servicios estadísticos (`StaleCheckService`, `FlatlineCheckService`, `DensityCheckService`) junto con la inyección de sus correspondientes adaptadores de salida (`PIStaleMetricsAdapter`, etc.).
- **(RF-7)** Refactorización del controlador de conexión `PIServerConnectionManager` para administrar credenciales y pool de sockets por separado.

## Implicancia en recursos existentes – Frontend
*(Dado que el servicio funciona como un background daemon sin interfaz propia, la implicancia de usuario radica en las herramientas corporativas de OSIsoft/AVEVA).*

- **(RF-4)(RF-5)(RF-6)** Los clientes y analistas de confiabilidad visualizarán nuevas series temporales bajo la convención: `{PIServerWriteName}:PPSWatchtower.{GroupName}.Total.{Stale_Level_N | Frozen_Count | LowDensity_Count}` dentro de PI Vision/PI System Explorer, disponibles para la creación de reportes nativos.

## Criterios de Aceptación 

- **CA-1 (Ejecución y Orquestación) (RF-1):** 
  Dado que el servicio se inicia correctamente, se evidenciará mediante NLog la corrida completa, agrupando y procesando satisfactoriamente tanto los "PointSources" del modo Legacy como los de la estrategia de "Negocio", uno tras otro, sin conflictos.
- **CA-2 (Exclusiones Paramétricas y de Bad States) (RF-5) (RF-6):** 
  Dados PI Points que cuenten con estados "Bad", tipo "Digital", o cuyo `ExDesc` posea `;IGNORE_FLATLINE;` o `;IGNORE_DENSITY;`, el sistema deberá ignorarlos en su evaluación estadística en el ciclo correspondiente.
- **CA-3 (Evaluación por Perfil Default) (RF-3):** 
  A partir de un grupo de negocio que no posea un perfil explícito, el sistema aplicará automáticamente el `DefaultProfile`. Si este define que el cálculo Flatline y Stale están inactivos, no se consumirán recursos para ellos.
- **CA-4 (Rendimiento Lote) (RNF):** 
  Ante un entorno de prueba donde un grupo específico posee 45,000 tags, el procesamiento de historial (Density/Flatline) se ejecutará fragmentado transparentemente en al menos 3 iteraciones de máximo 15,000 puntos, previniendo timeout.

## Pruebas Funcionales mínimas (QA) 

1. Validar la lectura y validación de las nuevas variables YAML.
2. Inyección de valores controlados simulando latencias por encima de 400 minutos para observar incrementos correspondientes en los buckets de `Stale_LevelN`.
3. Prueba de carga (Welford): Inyectar 100 valores matemáticamente idénticos para forzar varianza cero en un tag continuo. Validar el crecimiento del contador `Frozen_Count`.
4. Creación de tag dinámico con `ExDesc` en formato `PPSW_GROUP=QA_Test` y observar la creación correcta del grupo en las métricas resultantes tras 1 ejecución.
5. Inyección en tags configurados bajo `;IGNORE_FLATLINE;` comprobando que su valor de estado no altere ninguna métrica.

## Riesgos y decisiones explícitas 

- **Decisión sobre Cálculo de Desviación Estándar:** Validado el uso del **Algoritmo estadístico de Welford**. Previene errores numéricos de imprecisión y desbordamiento sobre sensores en ambientes inestables.
- **Exclusiones por default:** Se asume que tags de tipo `Step=1` o strings son intencionalmente estables, reduciendo enormemente los falsos positivos en el análisis de Flatline.
- **Separación de infraestructura vs negocio:** En lugar de reemplazar y romper las rutinas operacionales anteriores (Legacy), se decidió que corrieran en modo dual permitiendo al cliente mantener compatibilidad histórica total de métricas preexistentes mientras se migra paulatinamente.
