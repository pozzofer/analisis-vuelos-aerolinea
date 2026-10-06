# Hallazgos

> **Regla:** cada hallazgo = afirmación + cifra + dónde verlo + qué se recomienda. Solo se reporta lo cuantificable y accionable. Las cifras corresponden a la tabla final de 2.423 vuelos (ver `01_diccionario_datos.md`) y a las medidas de `04_medidas_dax.md`.

## 1. Pregunta principal y respuesta

**Pregunta:** ¿Qué rutas son rentables y cuáles no, y por qué? ✏️ (confirmar que coincide con la elegida en `00_brief.md`)

**Respuesta sintetizada:** 10 de las 36 rutas pierden plata (−$340 millones en 24 meses). Ninguna pasa por Buenos Aires, todas salen con el 37–41 % de los asientos ocupados y su costo por vuelo es *menor* que el de las rutas rentables. La causa es la baja ocupación, no el costo. Las 18 rutas que tocan Buenos Aires generan todo el margen de la aerolínea ($9.941 millones, más que el total de $9.935 millones, porque las otras 18 rutas suman −$6 millones) y vuelan con 87 % de ocupación.

**Decisión que habilita:** reducir frecuencias o rediseñar las 10 rutas deficitarias, y reforzar las rutas de Buenos Aires que superan el 88 % de ocupación.

## 2. Tabla de hallazgos

| # | Afirmación | Cifra | Dónde verlo | Confianza | Recomendación |
|---|---|---|---|---|---|
| H-01 | 10 de 36 rutas pierden plata, y ninguna pasa por Buenos Aires. | Margen conjunto de −$340 M (3,4 % del margen total). Peor ruta: Bariloche–Córdoba, −$64,3 M. | Dashboard: gráfico de margen por ruta (Inferior 10). | Alta (se repite en todos los escenarios de la sección 3) | Reducir frecuencias o rediseñar esas 10 rutas. |
| H-02 | Esas 10 rutas vuelan casi vacías, y por debajo de ~50 % de ocupación un vuelo pierde plata. | Ocupación de 37,4 % a 40,9 %. Entre los vuelos con menos de 50 % de ocupación, el 93 % pierde plata; con 60 % o más ninguno pierde. Punto de equilibrio estimado: ≈ 50 %. | Dashboard: gráfico de ocupación por ruta. | Media (el equilibrio sale de una regresión lineal simple, es una estimación) | Usar 60 % como umbral de alerta en la planificación de rutas. |
| H-03 | Las rutas que pierden no son las más caras de operar: pierden por falta de ingresos. | Costo promedio por vuelo: $5,0 M en las 10 peores contra $7,9 M en las 10 mejores. Margen por vuelo: −$1,07 M contra +$7,3 M. | Medida `Costo Promedio` filtrada por ruta. | Alta | No intentar resolver el problema bajando costos: hay que ajustar la oferta (asientos y frecuencias). |
| H-04 | Todo el margen depende de las rutas que tocan Buenos Aires. | Las 18 rutas con Buenos Aires suman $9.941 M; las otras 18 suman −$6 M. Las 10 mejores concentran el 75,6 % del margen. Ocupación: 87,1 % contra 51,9 %. | Dashboard: gráfico de margen por ruta (Superior 10). | Media (la estructura de hub es un patrón del simulador) | Proteger esas rutas y reforzar las de mayor ocupación (Mendoza–Buenos Aires 90,5 %, Buenos Aires–Mendoza 89,3 %, Neuquén–Buenos Aires 89,2 %). |
| H-05 | Hay una zona intermedia del interior que hoy es rentable pero con poco margen de seguridad. | 8 rutas del interior con ocupación de 59,5 % a 69,8 % suman +$334 M. La más frágil es Rosario–Córdoba (59,5 %, +$16 M). | Power BI: matriz de ruta con margen y ocupación (a crear) | Media | Mantenerlas y vigilar las que bajen de 60 %. |
| H-06 | El tipo de avión no explica la baja ocupación: es baja con los tres modelos. | En las 10 peores rutas: ATR 72 39,4 %, Embraer 190 38,3 %, Boeing 737 38,0 %. El Boeing cuesta $8,3 M por vuelo contra $2,5 M del ATR, y hace 88 de los 319 vuelos de esas rutas. | Power BI: segmentador de `Tipo_Avion` sobre el gráfico de ocupación | Media | Antes de cambiar de avión, reducir frecuencias. Evaluar un avión menor solo donde hoy opera el Boeing. |
| H-07 | La demanda es estacional: pico en julio, diciembre y enero; valle en marzo y abril. | Margen de los 2 años por mes: julio $1.099 M, diciembre $1.040 M, enero $993 M; marzo $530 M, abril $597 M. Ocupación: 86,6 % en julio y 69,6 % en marzo. El patrón se repite en 2024 y 2025. | Dashboard: gráfico de margen por mes. | Alta | Reforzar oferta en vacaciones de invierno, verano y fiestas; ajustar la de marzo–abril. |
| H-08 | El margen creció en 2025, pero no por más pasajeros. | Margen +21,7 % ($4.482 M a $5.453 M). Pasajeros −0,7 % y vuelos −3,6 %. El ingreso por vuelo subió 20,6 % y el costo por vuelo 16,6 %. Ocupación de 77,0 % a 79,7 %. | Dashboard: gráfico de margen por mes con segmentador de año. | Media-baja (la tendencia creciente fue un parámetro de generación del dataset) | No atribuir el crecimiento a la demanda; revisar la política de precios. |
| H-09 | Las demoras se concentran en Ushuaia y Bariloche, y sobre todo en invierno. | Demora promedio: Ushuaia 16,6 min, Bariloche 12,9 min, resto 8,4 min. De junio a agosto: Ushuaia 49,2 min contra 4,8 el resto del año; Bariloche 24,1 contra 9,5. | Dashboard: gráfico de demora por origen. | Media (muestras chicas: 22 y 47 vuelos en invierno; patrón pedido al generar el dataset) | Prever un margen operativo en esos aeropuertos entre junio y agosto. |
| H-10 | Las cancelaciones son poco frecuentes y de impacto menor. | 48 vuelos cancelados (2,0 %), con un costo de $56,1 M sin ingresos (0,6 % del margen total). | Power BI: filtro de `Estado_Vuelo` = Cancelado sobre `Margen Total` | Alta | Monitorear; no es prioridad frente a H-01. |

## 3. Análisis de sensibilidad

¿Cambian las conclusiones si se hubiera limpiado de otra manera?

| Escenario | Margen total | Ocupación | ¿Cambian las 10 rutas que pierden? |
|---|---|---|---|
| **A. Aplicado** (2.423 filas) | $9.935,0 M | 78,33 % | — |
| B. Excluir vacíos por métrica en lugar de borrar filas (propuesta de D-004) | $9.993,4 M | 78,36 % | No |
| C. Excluir los 2 ingresos atípicos (D-006) | $9.849,6 M | — | No |
| D. Incluir cancelados en la ocupación (D-011) | $9.935,0 M | 76,78 % | No |
| E. Excluir las 2 demoras extremas (D-009) | — | — | La demora promedio pasa de 9,04 a 8,54 min; Ushuaia y Bariloche siguen primeros |

**Conclusión:** ninguna decisión de limpieza altera los hallazgos. Las diferencias de cifras son de menos del 1 %.

