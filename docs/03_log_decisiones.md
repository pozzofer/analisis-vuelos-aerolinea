# Log de decisiones

> **Regla:** si cambia un número del informe, tiene que tener una entrada acá. Las decisiones no se borran: si una se revierte, se agrega una nueva que la reemplace.

## Leyenda de estados

| Estado | Significado |
|---|---|
| 🟢 Aceptada | Aplicada tal como se propuso |
| 🟠 Aceptada con cambios | Aplicada, pero distinta de la propuesta original (se explica la diferencia) |
| 🔴 No aplicada | La propuesta se descartó (se explica por qué) |
| ✏️ Por confirmar | Falta verificar un dato antes de cerrarla |

## Resumen

| ID | Tema | Estado | Efecto en el dataset |
|---|---|---|---|
| D-001 | Duplicados exactos | 🟢 | 2.480 → 2.450 filas |
| D-002 | Formato de fechas | 🟠 | Una sola columna de tipo Fecha |
| D-003 | Destino desde `Ruta` | 🟢 | 25 variantes → 10 valores |
| D-004 | Vacíos en `Pasajeros` e `Ingresos` | 🟠 | 2.450 → 2.423 filas |
| D-005 | Vacíos en `Minutos_Demora` | 🟠 | 13 vacíos completados |
| D-006 | Ingresos atípicos | 🔴 | Se conservan; efecto de 0,86 % en el margen |
| D-007 | Definición de puntualidad | 🟠 | Se usa `Estado_Vuelo` |
| D-008 | Demora promedio | 🟠 | Una sola versión (todos los operados) |
| D-009 | Demoras extremas | 🟢 | Se conservan |
| D-010 | Importación del CSV | 🟢 | Columnas importadas correctamente |
| D-011 | Tratamiento de cancelados | 🟢 | Decisión propia |
| D-012 | Cálculo de la ocupación | 🟢 | Decisión propia |
| D-013 | Eje de fechas del gráfico mensual | 🟢 | Decisión técnica propia |

---

## D-001 · Duplicados exactos

- **Problema:** 30 filas idénticas a otras (mismo `ID_Vuelo` y mismos valores en todas las columnas). Ninguna tenía vacíos.
- **Decisión:** eliminar los duplicados usando `ID_Vuelo` como clave y conservar la primera aparición. Como los 2.450 `ID_Vuelo` distintos coinciden con las 2.450 filas únicas, la clave elegida es equivalente a comparar la fila completa.
- **Motivo:** un mismo vuelo contado dos veces infla vuelos, pasajeros, ingresos y costos.
- **Alternativa descartada:** conservar los duplicados. Distorsiona todas las sumas.
- **Impacto:** 2.480 → 2.450 filas (−1,21 %).
- **Estado:** 🟢 Aceptada

## D-002 · Formato de fechas

- **Problema:** la columna `Fecha` mezcla `aaaa-mm-dd` (1.606 filas) y `dd/mm/aaaa` (874 filas). Con configuración regional de EE. UU., una fecha como `02/01/2024` se leería como 1 de febrero sin dar error.
- **Decisión:** convertir la columna con la transformación **Analizar** de Power Query, en lugar de la fórmula condicional por formato que proponía la plantilla.
- **Motivo:** resuelve ambos formatos en un solo paso, sin código adicional.
- **Riesgo asumido:** la interpretación depende de la configuración regional. Se mitiga con verificaciones puntuales sobre vuelos cuya fecha es ambigua (día menor o igual a 12).
- **Verificación realizada:** se comprobaron vuelos con fecha ambigua (día menor o igual a 12), como AP-00007 (02-01-2024) y AP-00333 (05-04-2024), y la columna quedó con las fechas correctas, sin errores.
- **Alternativa descartada:** fórmula condicional con `Date.FromText` y formato explícito por cada caso (independiente de la configuración regional; más robusta, pero más larga).
- **Estado:** 🟠 Aceptada con cambios

## D-003 · Destino reconstruido desde `Ruta`

- **Problema:** `Destino` tiene 25 variantes de escritura para 10 ciudades (por ejemplo "Bs As", "BUENOS AIRES", "Cordoba", "San Miguel de Tucumán"), en 245 filas.
- **Decisión:** duplicar `Ruta`, dividir la copia por el guion, descartar la parte de origen, reemplazar la columna `Destino` original por la parte de destino y mantener `Ruta` intacta.
- **Motivo:** `Ruta` es consistente en las 2.480 filas, así que es una fuente confiable y evita corregir variante por variante.
- **Alternativa descartada:** reemplazar valores a mano. Funciona, pero es más largo y deja margen para olvidar variantes.
- **Impacto:** `Destino` queda con exactamente 10 valores. Las 245 filas afectadas se corrigen sin perder ningún vuelo.
- **Estado:** 🟢 Aceptada

## D-004 · Vacíos en `Pasajeros` e `Ingresos`

- **Problema:** `Pasajeros` tiene 14 vacíos e `Ingresos` tiene 13. No coinciden en ninguna fila: son 27 filas distintas (1,10 % de las 2.450).
- **Propuesta original:** no imputar y excluir cada fila solo de la métrica donde falta el dato.
- **Decisión aplicada:** eliminar las 27 filas.
- **Motivo:** todas las métricas (ocupación, ingresos y margen) se calculan sobre exactamente el mismo conjunto de 2.423 vuelos, lo que las hace comparables entre sí. Al ser solo el 1,1 % del total, no se justifica inventar un valor.
- **Alternativas descartadas:** (a) la propuesta original, que obliga a cada medida a filtrar sus propios vacíos y trabaja con conjuntos distintos según la métrica; (b) rellenar con el promedio de la ruta, que introduce un dato inventado.
- **Impacto (comparación con la alternativa de conservar las filas):**

| Métrica | Con las 27 filas eliminadas (aplicado) | Excluyendo vacíos por métrica |
|---|---|---|
| Margen total | $9.935,0 millones (2.423 vuelos) | $9.993,4 millones (2.437 vuelos con ingreso) |
| Ocupación | 78,33 % | 78,36 % |

  El margen es 0,58 % menor porque 14 de las filas eliminadas tenían ingresos válidos. No altera ninguna conclusión.
- **Estado:** 🟠 Aceptada con cambios

## D-005 · Vacíos en `Minutos_Demora`

- **Problema:** 13 vacíos: 10 en vuelos "A tiempo" y 3 en vuelos "Demorado".
- **Propuesta original:** completar con 0 los de vuelos "A tiempo" y dejar vacíos los 3 de vuelos "Demorado".
- **Decisión aplicada:** completar con 0 los 10 vuelos "A tiempo" (se deduce del estado) y con **38 minutos** los 3 vuelos "Demorado". El valor 38 es la mediana de los vuelos demorados.
- **Motivo:** un vuelo demorado no puede tener 0 minutos, y se prefiere la mediana a la media (49,3 min) porque la media está inflada por las demoras extremas de 512 y 688 minutos.
- **Alternativas descartadas:** dejarlos vacíos (la propuesta, que complica las medidas de demora); usar la media (sesgada por los extremos); eliminarlos (se perdería un dato válido de vuelo demorado).
- **Impacto:** la demora promedio de los vuelos operados es 9,04 min con los 3 valores estimados y 9,00 min sin ellos (diferencia de 0,04 min). Los 3 valores son estimaciones, no datos observados.
- **Estado:** 🟠 Aceptada con cambios

## D-006 · Ingresos atípicos

- **Problema:** hay dos vuelos con ingresos fuera de lo normal: AP-01172, con ingreso negativo de −$6,5 millones (probable error de carga), y AP-00722, con $105,7 millones (≈10 veces lo habitual en su ruta, tipo chárter). Además hay dos vuelos "A tiempo" con 0 pasajeros (AP-00490 y AP-00753).
- **Propuesta original:** marcar los registros sospechosos con un indicador y excluirlos de las métricas de ingresos y margen sin borrarlos.
- **Decisión aplicada:** no marcarlos ni excluirlos. Se conservan tal como vienen.
- **Motivo:** la consigna de análisis estadístico pide detectarlos y decidir caso por caso (media vs. mediana, valores atípicos). Excluirlos antes del análisis eliminaría el material de estudio. Se tratan en la sección de valores atípicos del informe.
- **Impacto:** si se excluyeran los dos vuelos de ingresos atípicos, el margen total bajaría de $9.935,0 millones a $9.849,6 millones (−$85,4 millones; −0,86 %). Las conclusiones por ruta no cambian.
- **Alternativa descartada:** indicador `Caso_Atipico` con exclusión selectiva (propuesta). Queda como mejora posible.
- **Estado:** 🔴 No aplicada

## D-007 · Definición de puntualidad

- **Decisión aplicada:** la puntualidad se calcula como vuelos con `Estado_Vuelo = "A tiempo"` sobre vuelos operados (excluye cancelados). No se construyó la variante paralela de la propuesta (OTP A15).
- **Nota sobre el umbral:** en este dataset todo vuelo "Demorado" tiene al menos 15 minutos. Por eso, definir "puntual" como demora menor a 15 min coincide con el estado (81,68 %). Si se definiera como demora de 15 min o menos, subiría a 82,36 %.
- **Estado:** 🟠 Aceptada con cambios

## D-008 · Demora promedio

- **Propuesta original:** reportar dos versiones, una solo sobre vuelos demorados y otra sobre todos los operados, con etiquetas claras.
- **Decisión aplicada:** se reporta una sola versión, sobre **todos los vuelos operados** (incluye los puntuales con 0 min y excluye cancelados). Es la que usa el gráfico de demora por origen. Valor general: 9,04 min.
- **Referencia de la otra versión:** entre los vuelos demorados, la media es 49,3 min y la mediana 38 min.
- **Estado:** 🟠 Aceptada con cambios

## D-009 · Demoras extremas

- **Problema:** dos vuelos con demoras de 512 min (AP-00315) y 688 min (AP-01508), ambos con origen en Buenos Aires.
- **Decisión:** conservarlos y usar la mediana como control.
- **Motivo:** son datos válidos del dataset y se analizan como valores atípicos. Su efecto es limitado.
- **Impacto:** la demora promedio de los operados baja de 9,04 a 8,54 min si se excluyen. El ranking de orígenes con más demora (Ushuaia y Bariloche primero) no cambia. La mediana de demora de los vuelos operados es 0 min, porque la mayoría sale a tiempo.
- **Estado:** 🟢 Aceptada

## D-010 · Importación del CSV

- **Propuesta:** importar en UTF-8, verificar que el nombre de la primera columna sea `ID_Vuelo` (el archivo tiene BOM, que puede alterarlo) y convertir las columnas numéricas a entero.
- **Aplicado:** el CSV se importó en UTF-8 y los nombres de las columnas, incluida la primera (`ID_Vuelo`), quedaron correctos. Los tipos finales de cada columna figuran en `01_diccionario_datos.md`.
- **Estado:** 🟢 Aceptada

## D-011 · Tratamiento de los vuelos cancelados

- **Decisión:** los cancelados **entran al margen** y **salen de ocupación, puntualidad y demora**.
- **Motivo:** un vuelo cancelado tiene costo y no genera ingresos, por lo que es una pérdida real (48 cancelados en la tabla final). En cambio, no puede estar "vacío" ni "a tiempo", porque no despegó; incluirlo bajaría la ocupación y distorsionaría la medición de cómo se llenan los aviones.
- **Alternativa descartada:** incluirlos en todas las métricas. Mezcla dos fenómenos distintos (cuán llenos van los aviones y cuántos vuelos se cancelan).
- **Impacto:** la ocupación sería 76,78 % con cancelados y 78,33 % sin ellos. La puntualidad sería 80,07 % con cancelados y 81,68 % sin ellos.
- **Estado:** 🟢 Aceptada

## D-012 · Cálculo de la ocupación

- **Decisión:** la ocupación se calcula como Σ Pasajeros ÷ Σ Capacidad, no como el promedio de los porcentajes de cada vuelo.
- **Motivo:** pondera cada vuelo por sus asientos, de modo que un avión grande pesa más que uno chico. Es la forma habitual en el sector.
- **Impacto:** 78,33 % con esta definición, contra 77,23 % si se promediaran los porcentajes por vuelo.
- **Estado:** 🟢 Aceptada

## D-013 · Eje de fechas del gráfico mensual

- **Problema:** Power BI agrega automáticamente una jerarquía de fechas (año, trimestre, mes, día) al campo `Mes_inicio`, y el gráfico agrupaba por nombre de mes, mezclando 2024 y 2025 en 12 puntos.
- **Decisión:** desactivar la opción "Fecha y hora automáticas" del archivo y usar `Mes_inicio` como eje continuo, con interpolación recta (sin suavizado).
- **Motivo:** el gráfico debe mostrar los 24 meses por separado para ver la estacionalidad en cada año, y el forecast se calcula sobre los puntos reales.
- **Estado:** 🟢 Aceptada
