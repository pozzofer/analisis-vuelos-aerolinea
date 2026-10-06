# Diccionario de datos – Vuelos Aerolínea

> Describe el dataset **original** (`data/raw/vuelos_aerolinea.csv`), los problemas de calidad encontrados, las columnas que se agregaron en el modelo y las cifras de control. Los tratamientos aplicados se justifican en `03_log_decisiones.md`; las medidas, en `04_medidas_dax.md`.

## 1. Descripción general

| | |
|---|---|
| Archivo | `data/raw/vuelos_aerolinea.csv` |
| Contenido | Operación simulada de una aerolínea ficticia de vuelos nacionales (Argentina) |
| Granularidad | 1 fila = 1 vuelo |
| Período | 2024-01-01 a 2025-12-31 (24 meses) |
| Tamaño original | 2.480 filas × 12 columnas |
| Cobertura | 10 ciudades, 36 rutas dirigidas (A→B y B→A son rutas distintas), 3 tipos de avión |
| Codificación | UTF-8 con BOM (puede alterar el nombre de la primera columna, `ID_Vuelo`, al importar) |
| Moneda | Pesos argentinos (supuesto: el archivo no la indica) |

## 2. Origen del dataset

- **Generación:** script de Python (pandas y numpy) escrito con ayuda de una IA a partir de un prompt propio y ejecutado en Google Colab.
- **Semilla aleatoria:** el prompt pidió `np.random.seed(42)`. ✏️ Confirmar en el script generado que efectivamente se usó.
- **Patrones pedidos:** estacionalidad (verano y vacaciones de invierno altos; marzo-abril y octubre-noviembre bajos), tendencia levemente creciente 2024→2025, rutas desde y hacia Buenos Aires con alta ocupación, rutas transversales del interior con baja ocupación, más demoras en Bariloche y Ushuaia en invierno, ATR 72 en rutas cortas y Boeing 737 en las largas.
- **Problemas introducidos a propósito** (para practicar limpieza): duplicados exactos, valores vacíos, fechas en dos formatos, destinos escritos de varias formas y casos atípicos aislados (ver sección 6).
- **Responsable:** Fernando Pozzo. **Fecha de entrega:** ✏️

> Los datos son simulados. Los hallazgos describen el simulador, no el mercado aéreo real.

## 3. Columnas del dataset original

| # | Columna | Tipo original | Tipo final | Descripción | Valores / rango | Problemas detectados |
|---|---|---|---|---|---|---|
| 1 | `ID_Vuelo` | Texto | Texto | Identificador del vuelo, formato `AP-#####` | 2.450 IDs distintos | 30 filas duplicadas exactas (mismo ID y mismos valores) |
| 2 | `Fecha` | Texto | Fecha | Fecha del vuelo | 2024-01-01 a 2025-12-31 | **Dos formatos mezclados:** `aaaa-mm-dd` (1.606 filas) y `dd/mm/aaaa` (874 filas) |
| 3 | `Origen` | Texto | Texto | Ciudad de salida | 10 ciudades: Bariloche, Buenos Aires, Córdoba, Iguazú, Mendoza, Neuquén, Rosario, Salta, Tucumán, Ushuaia | Ninguno |
| 4 | `Destino` | Texto | Texto | Ciudad de llegada | 10 ciudades (las mismas que `Origen`) | **25 variantes de escritura** en lugar de 10 valores; 245 filas distintas del valor correcto (ej. "Bs As", "BUENOS AIRES", "Cordoba", "San Miguel de Tucumán") |
| 5 | `Ruta` | Texto | Texto | Origen y destino unidos por guion, ej. `Buenos Aires-Salta` | 36 rutas | Ninguno: es consistente en las 2.480 filas y sirve para reconstruir `Destino` |
| 6 | `Tipo_Avion` | Texto | Texto | Modelo de avión usado | ATR 72, Embraer 190, Boeing 737-700 | Ninguno |
| 7 | `Capacidad` | Entero | Entero | Asientos del avión | 70 (ATR 72), 100 (Embraer 190), 126 (Boeing 737-700) | Ninguno; depende del tipo de avión |
| 8 | `Pasajeros` | Entero | Entero | Pasajeros transportados | 0 a 123; nunca supera la capacidad | **14 vacíos**; 2 vuelos "A tiempo" con 0 pasajeros (casos atípicos) |
| 9 | `Ingresos` | Decimal | Decimal | Ingresos del vuelo en pesos | −6,5 M a 105,7 M | **13 vacíos**; 1 valor negativo; 1 valor extremo (≈10 veces lo normal) |
| 10 | `Costo_Vuelo` | Entero | Decimal | Costo del vuelo en pesos | 405 mil a 14,4 M | Ninguno. Los vuelos cancelados **sí tienen costo** |
| 11 | `Estado_Vuelo` | Texto | Texto | Resultado operativo del vuelo | A tiempo (1.984), Demorado (446), Cancelado (50) | Ninguno (conteos del archivo original) |
| 12 | `Minutos_Demora` | Entero | Entero | Minutos de demora | 0 a 688; todo vuelo "Demorado" tiene al menos 15 | **13 vacíos** (10 de vuelos "A tiempo" y 3 de "Demorado"); 2 valores extremos |

### Reglas de coherencia observadas

- Los vuelos **cancelados** registran 0 pasajeros e ingresos 0 (salvo los vacíos), pero mantienen su costo.
- `Capacidad` queda determinada por `Tipo_Avion`.
- `Ruta` = `Origen` + "-" + destino correcto.
- Los vuelos "A tiempo" tienen 0 minutos de demora cuando el dato no falta.

## 4. Columnas agregadas en el modelo

| Columna | Dónde se crea | Cálculo | Para qué sirve |
|---|---|---|---|
| `Destino` (reconstruido) | Power Query | Texto posterior al guion de `Ruta` | Dejar 10 valores consistentes (ver D-003) |
| `Fecha` (estandarizada) | Power Query | Conversión de los dos formatos a tipo Fecha, interpretando `dd/mm/aaaa` como día/mes/año | Poder ordenar y agrupar por tiempo (ver D-002) |
| `Minutos_Demora` (completada) | Power Query | Vacíos reemplazados: 0 si el vuelo está "A tiempo", 38 si está "Demorado" (mediana de los demorados) | Evitar vacíos en las medidas de demora (ver D-005) |
| `Margen` | Power BI (columna calculada) | `Ingresos − Costo_Vuelo` | Rentabilidad por vuelo, ruta y mes |
| `Mes_Inicio` | Power BI (columna calculada) | Primer día del mes de `Fecha` | Eje continuo del gráfico mensual |

## 5. Cifras de control

| Paso | Filas | Diferencia |
|---|---|---|
| Archivo original | 2.480 | |
| Tras quitar duplicados exactos | 2.450 | −30 |
| Tras quitar filas con `Pasajeros` o `Ingresos` vacíos | **2.423** | −27 (14 + 13, sin superposición) |

Composición final (2.423 filas): 1.940 "A tiempo", 435 "Demorado", 48 "Cancelado".

Valores que deben verse en el dashboard con la tabla final: Margen total ≈ $9.935 millones, ocupación 78,33 %, puntualidad 81,68 %.

## 6. Casos atípicos conocidos

Se conservaron en la limpieza porque forman parte del análisis de valores extremos 

| ID_Vuelo | Tipo de caso | Detalle |
|---|---|---|
| AP-00490 | Vuelo "A tiempo" sin pasajeros | 0 pasajeros, Buenos Aires–Ushuaia |
| AP-00753 | Vuelo "A tiempo" sin pasajeros | 0 pasajeros, Buenos Aires–Neuquén |
| AP-00315 | Demora extrema | 512 minutos, Buenos Aires–Salta |
| AP-01508 | Demora extrema | 688 minutos, Buenos Aires–Mendoza |
| AP-00722 | Ingreso extremo (tipo chárter) | ≈ $105,7 millones, Buenos Aires–Iguazú |
| AP-01172 | Ingreso negativo (error de carga) | −$6,5 millones, Bariloche–Córdoba |


## 8. Limitaciones del dataset

- Es simulado: las conclusiones aplican al simulador.
- No incluye distancia en kilómetros, por lo que no se pueden calcular indicadores por kilómetro.
- Tiene solo fecha, sin hora: no permite analizar demoras por franja horaria.
- No tiene matrícula de avión: no permite medir la utilización por aeronave.
- Los vuelos de una misma ruta solo se distinguen por fecha y avión; no se conoce la frecuencia diaria programada.
