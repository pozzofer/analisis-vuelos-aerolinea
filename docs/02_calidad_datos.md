# Calidad de datos: diagnóstico y cifras de control

**Dataset:** `data/raw/vuelos_aerolinea.csv` · **Perfilado realizado:** 2026-10-03 (sobre el archivo original, sin modificar)

Este documento tiene dos usos:
1. **Diagnóstico:** qué problemas tiene el dato y cuántas filas afecta cada uno.
2. **Cifras de control:** números de referencia para comprobar que tu limpieza en Power BI quedó bien (si tus conteos no coinciden, algo falló en un paso).

## 1. Problemas detectados

| ID | Problema | Columnas | Filas afectadas | Gravedad | Decisión |
|---|---|---|---|---|---|
| Q-01 | Duplicados exactos (misma fila completa repetida) | Todas | 30 duplicados (60 filas involucradas, 30 IDs) | Alta: infla vuelos, ingresos y costos | D-001 |
| Q-02 | Dos formatos de fecha mezclados | `Fecha` | 874 en `dd/mm/aaaa` · 1.606 en `aaaa-mm-dd` | Alta: Power BI puede interpretar mal o dar error | D-002 |
| Q-03 | Destino con 25 escrituras para 10 ciudades | `Destino` | 245 filas | Alta: parte los totales por ciudad | D-003 |
| Q-04 | Nulos en `Pasajeros` | `Pasajeros` | 14 | Media | D-004 |
| Q-05 | Nulos en `Ingresos` | `Ingresos` | 13 | Media | D-004 |
| Q-06 | Nulos en `Minutos_Demora` | `Minutos_Demora` | 13 | Media | D-005 |
| Q-07 | Ingreso negativo | `Ingresos` | 1 (AP-01172: −6.515.078) | Media | D-006 |
| Q-08 | Ingreso extremo (~10× lo normal por pasajero) | `Ingresos` | 1 (AP-00722: 105.712.830 con 94 pasajeros) | Alta: distorsiona rutas y totales | D-006 |
| Q-09 | Vuelos no cancelados con 0 pasajeros e ingresos 0 | `Pasajeros`, `Ingresos` | 2 (AP-00490, AP-00753) | Media | D-006 |
| Q-10 | Demoras extremas | `Minutos_Demora` | 2 (AP-01508: 688 min · AP-00315: 512 min) | Baja-Media: sesgan el promedio | D-009 |
| Q-11 | Umbral de "Demorado" inconsistente con el estándar A15 | `Estado_Vuelo`, `Minutos_Demora` | 16 vuelos "Demorado" con exactamente 15 min | Baja: define el OTP | D-007 |
| Q-12 | BOM en el archivo (nombre de 1ª columna con carácter invisible) | `ID_Vuelo` | n/a | Baja | D-010 |

## 2. Lo que está bien (no hace falta tocarlo)

- `Origen`, `Tipo_Avion`, `Estado_Vuelo`: sin variantes ni espacios sobrantes.
- `Ruta`: 36 valores consistentes; coincide con `Origen-Destino` normalizado en el 100 % de las filas.
- `Capacidad`: un único valor por tipo de avión.
- `Pasajeros ≤ Capacidad` en todas las filas.
- Coherencia `Estado_Vuelo` ↔ `Minutos_Demora` ↔ `Pasajeros`: sin contradicciones.
- No hay espacios en blanco sobrantes en ninguna columna de texto.
- Las fechas `dd/mm/aaaa` son **inequívocamente día/mes**: el orden de `ID_Vuelo` es cronológico solo si se leen como día/mes (leídas como mes/día, 534 fechas serían inválidas y el orden se rompe).

## 3. Cifras de control

### 3.1 Conteo de filas en cada etapa

| Etapa | Filas esperadas |
|---|---|
| Archivo original | **2.480** |
| Tras eliminar duplicados exactos | **2.450** |
| Cancelados (tras dedup) | 49 |
| Operados (A tiempo + Demorado) | **2.401** |

### 3.2 Distribución de estado (tras eliminar duplicados)

| Estado | Vuelos |
|---|---|
| A tiempo | 1.960 |
| Demorado | 441 |
| Cancelado | 49 |

### 3.3 Nulos tras eliminar duplicados (antes de imputar/tratar)

| Columna | Nulos |
|---|---|
| `Pasajeros` | 14 |
| `Ingresos` | 13 |
| `Minutos_Demora` | 13 |

### 3.4 KPIs de referencia (con datos deduplicados, **sin** tratar nulos ni outliers)

> Sirven para validar el modelo. Cambiarán cuando apliques D-004, D-006 y D-009. Anota en el log cuánto cambia cada uno.

| KPI | Valor | Cómo se calculó |
|---|---|---|
| Tasa de cancelación | 2,00 % | 49 / 2.450 |
| OTP (puntualidad) | 81,63 % | Estado "A tiempo" / operados (2.401) |
| Factor de ocupación | 78,36 % | Σ Pasajeros / Σ Capacidad, operados con `Pasajeros` no nulo |
| Demora promedio (solo demorados) | 49,31 min | Mediana: 38 min. Sin el valor de 688 min: 47,84 min |
| Demora promedio (todos los operados) | 9,04 min | |
| Ingresos totales | 23.823.951.488 | Incluye el valor extremo de AP-00722 y el negativo |
| Costo total | 13.900.875.390 | |
| Margen total | 9.923.076.098 | Ingresos – Costo |
| Rutas con margen total negativo | 10 de 36 | Antes de tratar outliers |
| Vuelos individuales con margen negativo | 359 | De ellos, 48 son cancelados (costo sin ingreso) |

### 3.5 Observación para el análisis

La **mediana** de la demora es 38 min pero la media es 49 min: la distribución tiene cola larga. Conviene reportar mediana junto con el promedio.

## 4. Cómo repetir este perfilado

En Power Query activa *Vista → Calidad de columna / Distribución de columnas / Perfil de columna* con "Basado en el conjunto de datos completo" (no solo las primeras 1.000 filas). Los conteos de la sección 3 deben coincidir.
