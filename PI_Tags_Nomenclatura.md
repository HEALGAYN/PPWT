# Nomenclatura de Escritura de PI Tags — PPSWatchtower

## Formato General

```
{PIServerWriteName}:PPSWatchtower.{sourceName}.Total.{nombre_métrica}
```

| Segmento | Descripción |
|---|---|
| `PIServerWriteName` | Servidor PI de escritura configurado en `appsettings.yml` |
| `PPSWatchtower` | Nombre fijo de la aplicación |
| `sourceName` | Identificador del grupo (PointSource en Legacy, groupName en Business) |
| `Total` | Nivel de agregación |
| `nombre_métrica` | Nombre de la métrica específica |

---

## 1. Modo Legacy (por PointSource)

`sourceName` = el **PointSource** configurado (ej: `MAG`, `AF`, `UFL_P2AH`).

### Ejemplo: PointSource = `MAG`

#### PIMetricsAdapter (métricas de estado)

```
CMPSERVMPISGODV:PPSWatchtower.MAG.Total.Status_Bad
CMPSERVMPISGODV:PPSWatchtower.MAG.Total.Status_Bad Input
CMPSERVMPISGODV:PPSWatchtower.MAG.Total.Status_Calc Failed
CMPSERVMPISGODV:PPSWatchtower.MAG.Total.Status_Comm Fail
CMPSERVMPISGODV:PPSWatchtower.MAG.Total.Status_Configure
CMPSERVMPISGODV:PPSWatchtower.MAG.Total.Status_I/O Timeout
CMPSERVMPISGODV:PPSWatchtower.MAG.Total.Status_No Data
CMPSERVMPISGODV:PPSWatchtower.MAG.Total.Status_No Result
CMPSERVMPISGODV:PPSWatchtower.MAG.Total.Status_Not Connect
CMPSERVMPISGODV:PPSWatchtower.MAG.Total.Status_Out of Serv
CMPSERVMPISGODV:PPSWatchtower.MAG.Total.Status_Pt Created
CMPSERVMPISGODV:PPSWatchtower.MAG.Total.Status_Scan Off
CMPSERVMPISGODV:PPSWatchtower.MAG.Total.Status_Set to Bad
CMPSERVMPISGODV:PPSWatchtower.MAG.Total.Status_Shutdown
CMPSERVMPISGODV:PPSWatchtower.MAG.Total.Status_Unit Down
CMPSERVMPISGODV:PPSWatchtower.MAG.Total.Bad Points Count
CMPSERVMPISGODV:PPSWatchtower.MAG.Total.Total Point Count
CMPSERVMPISGODV:PPSWatchtower.MAG.Total.Duplicate Point Count   ← Solo si MAG está en PointSourcesDuplicateEnable
```

### Ejemplo: PointSource = `AF` (sin duplicados)

```
CMPSERVMPISGODV:PPSWatchtower.AF.Total.Status_Bad
CMPSERVMPISGODV:PPSWatchtower.AF.Total.Status_Comm Fail
... (un tag por cada estado habilitado)
CMPSERVMPISGODV:PPSWatchtower.AF.Total.Bad Points Count
CMPSERVMPISGODV:PPSWatchtower.AF.Total.Total Point Count
```

> **Nota:** `AF` no está en `PointSourcesDuplicateEnable`, por lo tanto no se escribe `Duplicate Point Count`.

---

## 2. Modo Business (por grupo de negocio)

`sourceName` = el **nombre del grupo** descubierto o configurado.

- **Modo Standard:** Se parsea del ExDesc del PIPoint (lo que viene después del prefijo `PPSW_G=`).
- **Modo Static:** Se usa el `NameProfile` configurado en `Contexts`.

Los módulos de calidad (Stale, Flatline, Density) se activan según el perfil asignado al grupo.

---

### Ejemplo: Grupo `Informe_Ejecutivo` → Perfil `Informe_Ejecutivo_default`

Perfil: `EnableStaleCheck: false` | `EnableFlatlineCheck: true` | `EnableDensityCheck: false`

#### PIMetricsAdapter (siempre se ejecuta)

```
CMPSERVMPISGODV:PPSWatchtower.Informe_Ejecutivo.Total.Status_Bad
CMPSERVMPISGODV:PPSWatchtower.Informe_Ejecutivo.Total.Status_Bad Input
CMPSERVMPISGODV:PPSWatchtower.Informe_Ejecutivo.Total.Status_Calc Failed
CMPSERVMPISGODV:PPSWatchtower.Informe_Ejecutivo.Total.Status_Comm Fail
... (un tag por cada estado habilitado)
CMPSERVMPISGODV:PPSWatchtower.Informe_Ejecutivo.Total.Bad Points Count
CMPSERVMPISGODV:PPSWatchtower.Informe_Ejecutivo.Total.Total Point Count
```

#### PIFlatlineMetricsAdapter ✅ (habilitado)

```
CMPSERVMPISGODV:PPSWatchtower.Informe_Ejecutivo.Total.Frozen_Count
```

#### PIStaleMetricsAdapter ❌ — No se ejecuta (`EnableStaleCheck: false`)

#### PIDensityMetricsAdapter ❌ — No se ejecuta (`EnableDensityCheck: false`)

---

### Ejemplo: Grupo `Informe_Ejecutivo_CMP_Preliminar` → Perfil `Informe_Ejecutivo_CMP_Preliminar`

Perfil: `EnableStaleCheck: true` | `EnableFlatlineCheck: false` | `EnableDensityCheck: false`  
`LatencyBuckets: [1441, 1471, 1531]` (3 niveles)

#### PIMetricsAdapter (siempre se ejecuta)

```
CMPSERVMPISGODV:PPSWatchtower.Informe_Ejecutivo_CMP_Preliminar.Total.Status_Bad
CMPSERVMPISGODV:PPSWatchtower.Informe_Ejecutivo_CMP_Preliminar.Total.Status_Comm Fail
... (un tag por cada estado habilitado)
CMPSERVMPISGODV:PPSWatchtower.Informe_Ejecutivo_CMP_Preliminar.Total.Bad Points Count
CMPSERVMPISGODV:PPSWatchtower.Informe_Ejecutivo_CMP_Preliminar.Total.Total Point Count
```

#### PIStaleMetricsAdapter ✅ (habilitado, 3 buckets + MaxLatency)

```
CMPSERVMPISGODV:PPSWatchtower.Informe_Ejecutivo_CMP_Preliminar.Total.Stale_Level1
CMPSERVMPISGODV:PPSWatchtower.Informe_Ejecutivo_CMP_Preliminar.Total.Stale_Level2
CMPSERVMPISGODV:PPSWatchtower.Informe_Ejecutivo_CMP_Preliminar.Total.Stale_Level3
CMPSERVMPISGODV:PPSWatchtower.Informe_Ejecutivo_CMP_Preliminar.Total.MaxGroupLatency
```

#### PIFlatlineMetricsAdapter ❌ — No se ejecuta (`EnableFlatlineCheck: false`)

#### PIDensityMetricsAdapter ❌ — No se ejecuta (`EnableDensityCheck: false`)

---

### Ejemplo: Grupo `Informe_Embarque` → Perfil `Informe_Embarque`

Perfil: `EnableStaleCheck: true` | `EnableFlatlineCheck: false` | `EnableDensityCheck: false`  
`LatencyBuckets: [1441, 1471, 1531]`

#### PIStaleMetricsAdapter ✅

```
CMPSERVMPISGODV:PPSWatchtower.Informe_Embarque.Total.Stale_Level1
CMPSERVMPISGODV:PPSWatchtower.Informe_Embarque.Total.Stale_Level2
CMPSERVMPISGODV:PPSWatchtower.Informe_Embarque.Total.Stale_Level3
CMPSERVMPISGODV:PPSWatchtower.Informe_Embarque.Total.MaxGroupLatency
```

---

## Resumen de Métricas por Adapter

| Adapter | Métricas escritas | Condición |
|---|---|---|
| **PIMetricsAdapter** | `Status_{estado}`, `Bad Points Count`, `Total Point Count`, `Duplicate Point Count` | Siempre (Duplicate solo si está habilitado) |
| **PIStaleMetricsAdapter** | `Stale_Level{N}`, `MaxGroupLatency` | `EnableStaleCheck: true` + `LatencyBuckets` no vacío |
| **PIFlatlineMetricsAdapter** | `Frozen_Count` | `EnableFlatlineCheck: true` + `FlatlineSampleCount >= 2` |
| **PIDensityMetricsAdapter** | `LowDensity_Count`, `HighDensity_Count` | `EnableDensityCheck: true` |
