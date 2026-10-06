# Catálogo de medidas DAX

> Documenta las columnas calculadas y medidas del modelo en Power BI (`powerbi/`). Cada una tiene: descripción, regla de negocio en lenguaje simple, código DAX, formato y valor de control. Las definiciones de negocio están en `00_brief.md` (sección 4) y las justificaciones en `03_log_decisiones.md`.

## Convenciones

- Tabla del modelo: `vuelos_aerolinea` (tabla plana, ver `01_diccionario_datos.md`).
- Las divisiones se hacen con `DIVIDE()` para evitar errores de división por cero.
- Formato de porcentajes: `0,00 %`. Formato de montos: moneda en pesos, mostrada en millones en las tarjetas.
- **Cancelados:** entran en las medidas financieras y quedan fuera de ocupación, puntualidad y demora (ver D-011).
- Los casos atípicos de ingresos no se excluyen de ninguna medida (ver D-006).

## Resumen

| Carpeta | Elemento | Tipo | Se usa en |
|---|---|---|---|
| Finanzas | `Margen` | Columna calculada | Gráfico de margen por ruta, margen total |
| Finanzas | `Margen Total` | Medida | Tarjeta de KPI, gráficos de margen por ruta y por mes |
| Finanzas | `Costo Promedio` | Medida | Comprobación de la causa de las pérdidas |
| Ocupación | `Ocupacion %` | Medida | Tarjeta de KPI, gráfico de ocupación por ruta |
| Puntualidad | `Puntualidad %` | Medida | Tarjeta de KPI |
| Demoras | `Demora Promedio` | Medida | Gráfico de demora por origen |
| Tiempo | `Mes_inicio` | Columna calculada | Eje del gráfico mensual |

---

## Columnas calculadas

### `Margen`

- **Descripción:** resultado económico de cada vuelo.
- **Regla de negocio:** ingresos menos costo del vuelo. Puede ser negativo. Un vuelo cancelado tiene ingresos 0 y costo positivo, por lo que su margen es una pérdida.
- **Código:**

```dax
Margen = vuelos_aerolinea[Ingresos] - vuelos_aerolinea[Costo_Vuelo]
```

### `Mes_inicio`

- **Descripción:** primer día del mes en que ocurrió el vuelo.
- **Regla de negocio:** agrupa todos los vuelos de un mes bajo una misma fecha, para armar un eje temporal continuo de 24 puntos (enero 2024 a diciembre 2025).
- **Código:**

```dax
Mes_inicio =
DATE(
    YEAR(vuelos_aerolinea[Fecha]),
    MONTH(vuelos_aerolinea[Fecha]),
    1
)
```

- **Nota:** requiere tener desactivada la opción "Fecha y hora automáticas" para que no se agregue una jerarquía (ver D-013).

---

## Medidas

### Finanzas

#### `Margen Total`

- **Descripción:** rentabilidad acumulada del conjunto de vuelos filtrado.
- **Regla de negocio:** suma de la columna `Margen` de todos los vuelos, incluidos los cancelados.
- **Código:**

```dax
Margen Total = SUM(vuelos_aerolinea[Margen])
```

- **Formato:** moneda, en millones.
- **Valor de control:** ≈ $9.935 millones (2.423 vuelos).

#### `Costo Promedio`

- **Descripción:** costo medio por vuelo.
- **Regla de negocio:** promedio de `Costo_Vuelo`, incluidos los cancelados (que sí tienen costo). Sirve para comparar el costo de las rutas que pierden plata con el de las que ganan.
- **Código:**

```dax
Costo Promedio = AVERAGE(vuelos_aerolinea[Costo_Vuelo])
```

- **Formato:** moneda, en millones.
- **Valor de control:** ≈ $5,68 millones por vuelo en todo el dataset.
- **Uso:** en las 10 rutas de menor margen el costo promedio es ≈ $5,0 millones por vuelo; en las 10 de mayor margen, ≈ $7,9 millones. Las rutas que pierden **no** son las de costo más alto: pierden porque sus ingresos no alcanzan, es decir, porque salen con pocos pasajeros.

### Ocupación

#### `Ocupacion %`

- **Descripción:** proporción de asientos ocupados en los vuelos que despegaron.
- **Regla de negocio:** total de pasajeros dividido por el total de asientos ofrecidos, solo sobre vuelos operados (excluye cancelados). Pondera por asientos: un avión grande pesa más que uno chico (ver D-012).
- **Código:**

```dax
Ocupacion % =
CALCULATE(
    DIVIDE(
        SUM(vuelos_aerolinea[Pasajeros]),
        SUM(vuelos_aerolinea[Capacidad])
    ),
    vuelos_aerolinea[Estado_Vuelo] <> "Cancelado"
)
```

- **Formato:** porcentaje, `0,00 %`.
- **Valor de control:** 78,33 %.

### Puntualidad

#### `Puntualidad %`

- **Descripción:** proporción de vuelos operados que salieron a tiempo.
- **Regla de negocio:** vuelos con estado "A tiempo" divididos por los vuelos operados. Los cancelados quedan fuera del denominador (ver D-007 y D-011).
- **Código:**

```dax
Puntualidad % =
DIVIDE(
    CALCULATE(
        COUNTROWS(vuelos_aerolinea),
        vuelos_aerolinea[Estado_Vuelo] = "A tiempo"
    ),
    CALCULATE(
        COUNTROWS(vuelos_aerolinea),
        vuelos_aerolinea[Estado_Vuelo] <> "Cancelado"
    )
)
```

- **Formato:** porcentaje, `0,00 %`.
- **Valor de control:** 81,68 %.

### Demoras

#### `Demora Promedio`

- **Descripción:** minutos de demora promedio por vuelo operado.
- **Regla de negocio:** promedio de `Minutos_Demora` sobre todos los vuelos operados, incluidos los puntuales (con 0 minutos) y excluyendo cancelados (ver D-008). Conserva las demoras extremas (ver D-009).
- **Código:**

```dax
Demora Promedio =
CALCULATE(
    AVERAGE(vuelos_aerolinea[Minutos_Demora]),
    vuelos_aerolinea[Estado_Vuelo] <> "Cancelado"
)
```

- **Formato:** número con un decimal.
- **Valor de control:** 9,04 minutos en todo el dataset. Por origen: Ushuaia ≈ 16,1; Bariloche ≈ 12,9; el resto entre 6 y 9.

---

## Validación: cifras de control

Con los filtros del dashboard sin tocar (todos los años, orígenes y aviones), las medidas deben dar:

| Medida | Valor esperado |
|---|---|
| Vuelos en la tabla (filas) | 2.423 |
| `Margen Total` | ≈ $9.935 millones |
| `Ocupacion %` | 78,33 % |
| `Puntualidad %` | 81,68 % |
| `Demora Promedio` | 9,04 min |
| `Costo Promedio` | ≈ $5,68 millones |

