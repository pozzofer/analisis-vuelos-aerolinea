# Análisis de vuelos de una aerolínea: rentabilidad de rutas

Análisis de la operación de una **aerolínea ficticia de vuelos nacionales** (Argentina) entre enero de 2024 y diciembre de 2025: 10 ciudades, 36 rutas y 3 tipos de avión. El objetivo es identificar qué rutas ganan y cuáles pierden plata, y qué decisiones de planificación se desprenden.

> **Los datos son simulados.** Los hallazgos describen el simulador, no el mercado aéreo real.

## Pregunta de negocio

| | |
|---|---|
| **Pregunta principal** | ¿Qué rutas son rentables y cuáles no, y por qué? |
| **Preguntas secundarias** | ¿Cuándo se concentra la demanda? ¿Dónde y cuándo se concentran las demoras? |
| **Audiencia** | Área de planificación de rutas y destinos |
| **Decisión que apoya** | Qué rutas reforzar, reducir o rediseñar la próxima temporada |

## Resultados principales

- **10 de 36 rutas pierden plata** (−$340 millones en 24 meses). Ninguna pasa por Buenos Aires.
- Esas rutas vuelan con **37–41 % de ocupación**. El punto de equilibrio ronda el **50 %**; con 60 % o más ningún vuelo pierde.
- **No pierden por costo:** su costo por vuelo ($5,0 M) es menor que el de las 10 más rentables ($7,9 M). Pierden por falta de pasajeros.
- Las 18 rutas que tocan Buenos Aires generan todo el margen y vuelan con 87 % de ocupación.
- La demanda es **estacional**: picos en julio, diciembre y enero; valle en marzo y abril.
- Las demoras se concentran en **Ushuaia** (16,6 min) y **Bariloche** (12,9 min), sobre todo entre junio y agosto.

El detalle, con cifras, ubicación en el dashboard y nivel de confianza, está en [`docs/05_hallazgos.md`](docs/05_hallazgos.md).

## Indicadores principales

| Indicador | Valor |
|---|---|
| Margen total (ingresos − costo) | ≈ $9.935 millones |
| Ocupación (pasajeros ÷ asientos, sin cancelados) | 78,33 % |
| Puntualidad (a tiempo ÷ operados) | 81,68 % |

Definiciones exactas en [`docs/00_brief.md`](docs/00_brief.md) y código en [`docs/04_medidas_dax.md`](docs/04_medidas_dax.md).

## Estado del proyecto

| Hito | Estado | Documento |
|---|---|---|
| Perfilado y diagnóstico de calidad | ✅ | `docs/02_calidad_datos.md` |
| Brief y métricas | ✅ | `docs/00_brief.md` |
| Diccionario de datos | ✅ | `docs/01_diccionario_datos.md` |
| Limpieza y transformación (Power Query) | ✅ | `docs/03_log_decisiones.md` |
| Modelo y medidas DAX | ✅ | `docs/04_medidas_dax.md` |
| Dashboard en Power BI | ✅ ✏️ | `powerbi/` |
| Hallazgos y conclusiones | ✅ | `docs/05_hallazgos.md` |

## Cifras de control

Para comprobar que la limpieza se reprodujo bien:

| Paso | Filas |
|---|---|
| Archivo original | 2.480 |
| Sin duplicados exactos (−30) | 2.450 |
| Sin filas con `Pasajeros` o `Ingresos` vacíos (−27) | **2.423** |

Con la tabla final y los filtros del dashboard sin tocar, las medidas deben dar: Margen total ≈ $9.935 M, Ocupación 78,33 %, Puntualidad 81,68 %, Demora promedio 9,04 min. La tabla completa y las diferencias con las cifras de la plantilla original están en `docs/04_medidas_dax.md`.

## Estructura del repositorio

```
├── data/
│   ├── raw/                  Dataset original, sin modificar (vuelos_aerolinea.csv)
│   └── processed/            Dataset limpio (2.423 filas)
├── docs/
│   ├── 00_brief.md           Pregunta, alcance, métricas
│   ├── 01_diccionario_datos.md
│   ├── 02_calidad_datos.md   Diagnóstico de calidad
│   ├── 03_log_decisiones.md  Decisiones de limpieza y análisis (D-001 a D-013)
│   ├── 04_medidas_dax.md     Medidas y columnas calculadas
│   ├── 05_hallazgos.md       Resultados y recomendaciones
│   └── 06_guia_git.md        Guía de uso de Git
├── powerbi/                  Proyecto de Power BI (.pbix)
├── outputs/                  Capturas del dashboard e informes
├── CHANGELOG.md
└── README.md
```

## Datos

- **Archivo:** `data/raw/vuelos_aerolinea.csv`, 2.480 filas × 12 columnas, UTF-8 con BOM.
- **Origen:** generado con un script de Python escrito con ayuda de una IA, con problemas de calidad introducidos a propósito (duplicados, vacíos, fechas en dos formatos, destinos mal escritos y casos atípicos).
- **Columnas:** ver [`docs/01_diccionario_datos.md`](docs/01_diccionario_datos.md).
- **Moneda:** pesos argentinos (supuesto).

## Cómo reproducir el análisis

1. Clonar el repositorio.
2. Abrir el archivo `.pbix` de la carpeta `powerbi/` con Power BI Desktop. ✏️ (indicar el nombre del archivo)
3. Apuntar la consulta al CSV de tu copia local: **Inicio → Transformar datos → Configuración de origen de datos → Cambiar origen**, y elegir `data/raw/vuelos_aerolinea.csv`.
4. Actualizar los datos y verificar que la tabla final tenga **2.423 filas** y que las tarjetas del dashboard coincidan con las cifras de control.

Los pasos de limpieza y su justificación están en `docs/03_log_decisiones.md`. El archivo tiene desactivada la opción *Fecha y hora automáticas* (ver D-013); si se crea un archivo nuevo desde cero, hay que desactivarla para que el gráfico mensual muestre los 24 meses.

## Herramientas

- **Power BI Desktop:** limpieza (Power Query), modelo, medidas (DAX) y dashboard.
- **Python (pandas, numpy):** generación del dataset y verificaciones de cifras.
- **Git y GitHub:** versionado y documentación.

## Limitaciones

- Datos simulados: los patrones (rutas de Buenos Aires con alta ocupación, estacionalidad, más demoras en invierno en Bariloche y Ushuaia) se pidieron al generar el dataset.
- No hay distancia, tarifas, hora del vuelo ni matrícula del avión.
- Las conclusiones sobre demoras estacionales se apoyan en pocos vuelos por origen.

Detalle completo en la sección 4 de `docs/05_hallazgos.md`.

## Uso de IA

Se usó Claude (Anthropic) como asistente para generar el script del dataset, guiar la limpieza, proponer medidas DAX, verificar cifras en Python y redactar la documentación. Las decisiones fueron tomadas y revisadas por el autor. Ver `docs/05_hallazgos.md`, sección 6.

## Autor

Fernando Pozzo · Proyecto iniciado el 2026-10-03 · Versión actual: ver [`CHANGELOG.md`](CHANGELOG.md).
