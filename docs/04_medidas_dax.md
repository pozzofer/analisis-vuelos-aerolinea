# Catálogo de medidas DAX

Todas las medidas viven en una tabla propia llamada `_Medidas`, organizadas por *Display Folder*. Cada una tiene:
1. **Descripción** (también cargada en las propiedades de la medida en Power BI, aparece como tooltip).
2. **Regla de negocio** en lenguaje simple.
3. **Código DAX** comentado con el *porqué*, no con la sintaxis.

> Los nombres de tabla/columna asumen el modelo propuesto: tabla `Vuelos` (después de la limpieza) con las columnas originales más `Fecha_Vuelo`, `Ingreso_Invalido` (ver D-006). Ajusta si tus nombres difieren.

## Convenciones

- Prefijo de carpeta: `01 Volumen`, `02 Puntualidad`, `03 Ocupación`, `04 Finanzas`.
- Usar siempre `DIVIDE()` en lugar de `/` (evita errores por división por cero).
- Formato: porcentajes `0,0 %`; minutos `0,0`; montos `#,0`.
- Formatear el código con DAX Formatter antes de guardar.

---

## 01 Volumen

| Medida | Descripción | Decisión relacionada |
|---|---|---|
| `Vuelos Programados` | Cantidad de vuelos (filas únicas), incluye cancelados | D-001 |
| `Vuelos Cancelados` | Vuelos con estado "Cancelado" | |
| `Vuelos Operados` | Programados − cancelados | |
| `Tasa de Cancelación` | Cancelados / Programados | |

```dax
// Cada fila de Vuelos es un vuelo único tras eliminar duplicados (D-001)
Vuelos Programados = COUNTROWS ( Vuelos )

Vuelos Cancelados =
CALCULATE ( COUNTROWS ( Vuelos ), Vuelos[Estado_Vuelo] = "Cancelado" )

// Un vuelo cancelado no operó: no tiene hora de llegada ni pasajeros reales
Vuelos Operados = [Vuelos Programados] - [Vuelos Cancelados]

Tasa de Cancelación = DIVIDE ( [Vuelos Cancelados], [Vuelos Programados] )
```

## 02 Puntualidad

| Medida | Descripción | Decisión relacionada |
|---|---|---|
| `Vuelos Demorados` | Vuelos con estado "Demorado" | |
| `OTP %` | % de vuelos operados con estado "A tiempo" | D-007 |
| `OTP A15 %` | % de vuelos operados con demora ≤ 15 min (estándar industria) | D-007 |
| `Demora Prom. Demorados (min)` | Promedio de minutos solo en vuelos demorados | D-008 |
| `Demora Mediana Demorados (min)` | Mediana, resistente a valores extremos | D-008, D-009 |

```dax
Vuelos Demorados =
CALCULATE ( COUNTROWS ( Vuelos ), Vuelos[Estado_Vuelo] = "Demorado" )

// OTP: se mide sobre vuelos OPERADOS (los cancelados no entran en el denominador)
OTP % =
DIVIDE (
    CALCULATE ( COUNTROWS ( Vuelos ), Vuelos[Estado_Vuelo] = "A tiempo" ),
    [Vuelos Operados]
)

// Variante estándar de la industria (A15): puntual si demora <= 15 min.
// Difiere del OTP % solo por los vuelos "Demorado" con exactamente 15 min (D-007)
OTP A15 % =
DIVIDE (
    CALCULATE (
        COUNTROWS ( Vuelos ),
        Vuelos[Estado_Vuelo] <> "Cancelado",
        Vuelos[Minutos_Demora] <= 15
    ),
    [Vuelos Operados]
)

// Solo vuelos demorados: promediar los "A tiempo" (0 min) diluiría el problema (D-008)
Demora Prom. Demorados (min) =
CALCULATE ( AVERAGE ( Vuelos[Minutos_Demora] ), Vuelos[Estado_Vuelo] = "Demorado" )

// La mediana casi no se mueve con los valores de 512 y 688 min (D-009)
Demora Mediana Demorados (min) =
CALCULATE ( MEDIAN ( Vuelos[Minutos_Demora] ), Vuelos[Estado_Vuelo] = "Demorado" )
```

## 03 Ocupación

| Medida | Descripción | Decisión relacionada |
|---|---|---|
| `Pasajeros` | Total de pasajeros transportados | D-004 |
| `Asientos Ofrecidos` | Capacidad de vuelos operados con pasajeros informados | D-004 |
| `Factor de Ocupación` | Pasajeros / Asientos ofrecidos | D-004 |

```dax
Pasajeros = SUM ( Vuelos[Pasajeros] )

// Solo cuenta la capacidad de los vuelos operados Y con Pasajeros informado.
// Si se sumara la capacidad de una fila con Pasajeros nulo, el factor quedaría subestimado (D-004)
Asientos Ofrecidos =
CALCULATE (
    SUM ( Vuelos[Capacidad] ),
    Vuelos[Estado_Vuelo] <> "Cancelado",
    NOT ISBLANK ( Vuelos[Pasajeros] )
)

Factor de Ocupación = DIVIDE ( [Pasajeros], [Asientos Ofrecidos] )
```

## 04 Finanzas

| Medida | Descripción | Decisión relacionada |
|---|---|---|
| `Ingresos` | Suma de ingresos válidos | D-006 |
| `Costo` | Suma del costo de todos los vuelos (incluye cancelados) | |
| `Margen` | Ingresos − Costo (solo vuelos con ingreso válido) | D-006 |
| `Margen %` | Margen / Ingresos | |
| `Ingreso por Pasajero` | Ingresos / Pasajeros | |
| `Costo por Asiento` | Costo / Asientos ofrecidos (aproxima CASK sin distancia) | |

```dax
// Se excluyen los ingresos inválidos marcados en Ingreso_Invalido (negativo y extremo, ver D-006)
Ingresos =
CALCULATE ( SUM ( Vuelos[Ingresos] ), Vuelos[Ingreso_Invalido] = "No" )

// El costo de los cancelados SÍ se incluye: es plata gastada sin ingreso
Costo = SUM ( Vuelos[Costo_Vuelo] )

// Para que el margen sea comparable, el costo se limita a las mismas filas que tienen ingreso válido
Margen =
VAR _Costo =
    CALCULATE ( SUM ( Vuelos[Costo_Vuelo] ), Vuelos[Ingreso_Invalido] = "No" )
RETURN
    [Ingresos] - _Costo

Margen % = DIVIDE ( [Margen], [Ingresos] )

Ingreso por Pasajero =
DIVIDE ( [Ingresos], CALCULATE ( [Pasajeros], Vuelos[Ingreso_Invalido] = "No" ) )

Costo por Asiento = DIVIDE ( [Costo], [Asientos Ofrecidos] )
```

> ⚠️ **Revisa antes de usar:** estas medidas son un punto de partida. `Ingreso_Invalido` marca solo los 2 vuelos con ingreso negativo o extremo. Los 2 vuelos operados vacíos (AP-00490, AP-00753) tienen su propia marca `Revisar` y **no** se excluyen de las medidas; si decides excluirlos, ajusta los filtros y anótalo en el log. Valida cada medida contra las **cifras de control** de `02_calidad_datos.md`.

---

## Validación (completar a medida que avances)

| Medida | Valor esperado (cifras de control, antes de D-004/D-006/D-009) | Valor en Power BI | ¿Cuadra? |
|---|---|---|---|
| Vuelos Programados | 2.450 | ✏️ | |
| Vuelos Cancelados | 49 | ✏️ | |
| Vuelos Operados | 2.401 | ✏️ | |
| Tasa de Cancelación | 2,00 % | ✏️ | |
| OTP % | 81,63 % | ✏️ | |
| Factor de Ocupación | 78,36 % | ✏️ | |
| Demora Prom. Demorados | 49,31 min | ✏️ | |

Después de aplicar D-006, `Ingresos` y `Margen` **no** deben coincidir con las cifras de control (esa diferencia es justamente el impacto de la decisión: anótala en el log).
