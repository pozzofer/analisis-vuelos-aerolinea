# Diccionario de datos

**Tabla:** `vuelos_aerolinea.csv` (ubicación: `data/raw/`)
**Filas:** 2.480 (2.450 únicas tras eliminar 30 duplicados exactos) · **Columnas:** 12
**Codificación:** UTF-8 con BOM (el BOM hace que la primera columna aparezca como `﻿ID_Vuelo`; Power Query suele manejarlo, pero verifica el nombre de la columna).

> Las columnas "valores / rango" y "nulos" están medidas sobre el archivo **original (2.480 filas)**, salvo indicación contraria. Las columnas ✏️ deben confirmarse con quien generó el dataset.

## Columnas

| # | Columna | Tipo original | Tipo objetivo en Power BI | Descripción | Valores / rango observado | Nulos | Problemas conocidos |
|---|---|---|---|---|---|---|---|
| 1 | `ID_Vuelo` | Texto | Texto | Identificador del vuelo, formato `AP-#####` | `AP-00001` … | 0 | 30 IDs repetidos; las filas repetidas son idénticas en todas las columnas (duplicado exacto) |
| 2 | `Fecha` | Texto | **Fecha** | Fecha del vuelo | 2024-01-01 a 2025-12-31 | 0 | **Dos formatos mezclados:** `aaaa-mm-dd` (1.606 filas) y `dd/mm/aaaa` (874 filas). Ver D-002 |
| 3 | `Origen` | Texto | Texto | Ciudad de origen | 10 valores: Bariloche, Buenos Aires, Córdoba, Iguazú, Mendoza, Neuquén, Rosario, Salta, Tucumán, Ushuaia | 0 | Limpio |
| 4 | `Destino` | Texto | Texto | Ciudad de destino | 10 valores "reales", pero aparecen **25 variantes** de escritura | 0 | Mayúsculas (`BUENOS AIRES`), abreviaturas (`Bs As`), sin tilde (`Cordoba`), nombres largos (`San Carlos de Bariloche`, `San Miguel de Tucumán`, `Puerto Iguazú`). Afecta 245 filas. Ver D-003 |
| 5 | `Ruta` | Texto | Texto | Origen y destino unidos: `Origen-Destino` | 36 rutas dirigidas | 0 | **Limpia y consistente:** coincide con `Origen` + `-` + `Destino` normalizado en el 100 % de las filas. Es la referencia para corregir `Destino` |
| 6 | `Tipo_Avion` | Texto | Texto | Modelo de aeronave | `ATR 72`, `Embraer 190`, `Boeing 737-700` | 0 | Limpio |
| 7 | `Capacidad` | Entero | Entero | Asientos del avión | ATR 72 = 70 · Embraer 190 = 100 · Boeing 737-700 = 126 | 0 | Limpio. Es función exacta del tipo de avión (1 valor por modelo) |
| 8 | `Pasajeros` | Entero (decimal al importar) | Entero | Pasajeros transportados | 0 a 123; nunca supera la capacidad | **14** | Nulos en vuelos no cancelados. 2 vuelos "A tiempo" con 0 pasajeros (ver D-006) |
| 9 | `Ingresos` | Entero (decimal al importar) | Moneda / Decimal | Ingresos del vuelo ✏️ (moneda no indicada; supuesto: ARS) | −6.515.078 a 105.712.830; mediana ≈ 9,3 M | **13** | 1 valor negativo (AP-01172), 1 valor extremo (AP-00722), 49 ceros en cancelados y 2 ceros en no cancelados |
| 10 | `Costo_Vuelo` | Entero | Moneda / Decimal | Costo operativo del vuelo ✏️ (misma moneda que Ingresos) | 405.063 a 14.421.388 | 0 | Cancelados **sí tienen costo** (media ≈ 1,16 M): costo parcial incurrido. Escala con el avión (ATR ≈ 2,6 M · Embraer ≈ 5,2 M · Boeing ≈ 9,0 M de media) |
| 11 | `Estado_Vuelo` | Texto | Texto | Resultado operativo | `A tiempo` (1.984), `Demorado` (446), `Cancelado` (50) | 0 | Coherente con `Minutos_Demora` (ver abajo) |
| 12 | `Minutos_Demora` | Entero (decimal al importar) | Entero | Minutos de retraso | 0 a 688 | **13** | Mínimo en `Demorado` = 15 (16 filas exactas en 15). 2 valores extremos: 688 min (AP-01508) y 512 min (AP-00315); el resto de las demoras no pasa de 300 min. Ver D-009 |

## Reglas de coherencia verificadas

| Regla | Resultado |
|---|---|
| `Estado_Vuelo = "A tiempo"` → `Minutos_Demora = 0` | ✅ Se cumple siempre (cuando no es nulo) |
| `Estado_Vuelo = "Cancelado"` → `Pasajeros = 0`, `Ingresos = 0`, `Minutos_Demora = 0` | ✅ Se cumple siempre (1 cancelado con `Ingresos` nulo) |
| `Estado_Vuelo = "Demorado"` → `Minutos_Demora ≥ 15` | ✅ Se cumple (mínimo 15) |
| `Pasajeros ≤ Capacidad` | ✅ Se cumple siempre |
| `Ruta = Origen-Destino` | ✅ Tras normalizar `Destino` |
| `Origen ≠ Destino` | ✅ Se cumple siempre |

## Relaciones útiles para el modelo (estrella)

- **Hecho:** `Vuelos` (1 fila = 1 vuelo).
- **Dimensiones sugeridas:** `Calendario` (por `Fecha`), `Aeropuertos`/`Ciudades` (Origen y Destino, 2 relaciones: una activa y otra inactiva), `Rutas`, `Aviones` (`Tipo_Avion` + `Capacidad`).

## Procedencia ✏️ (completar)

| Pregunta | Respuesta |
|---|---|
| ¿Cómo se generó el dataset? (script, herramienta, IA, plantilla) | |
| ¿Quién lo entregó y cuándo? | |
| ¿Hay semilla/parámetros de generación? | |
| ¿Qué problemas de calidad se introdujeron a propósito para practicar limpieza? | Los duplicados, fechas mixtas, destinos con variantes, nulos, negativo y outliers parecen intencionales; confirmar |
| Moneda de `Ingresos` y `Costo_Vuelo` | |
