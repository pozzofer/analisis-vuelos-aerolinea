# Optimización de vuelos – Aerolínea nacional (simulación)

> ⚠️ **Dataset simulado.** Los hallazgos describen los datos simulados, no el mercado aéreo real.

## ¿Qué es este proyecto?

Análisis de 2 años de operación (2024-01-01 a 2025-12-31) de una aerolínea nacional ficticia con 10 ciudades, 36 rutas y 3 tipos de avión, con el objetivo de identificar oportunidades para optimizar la red de vuelos. Limpieza y visualización en **Power BI**; documentación y versionado en **Git**.

## Estado del proyecto

| Etapa | Estado |
|---|---|
| Brief y definición de métricas | ⬜ Pendiente de completar (`docs/00_brief.md`) |
| Perfilado y diagnóstico de calidad | ✅ Hecho (`docs/02_calidad_datos.md`) |
| Limpieza en Power Query | ⬜ Pendiente |
| Modelo y medidas DAX | ⬜ Pendiente |
| Informe / dashboard | ⬜ Pendiente |
| Conclusiones y recomendaciones | ⬜ Pendiente |

*Actualiza esta tabla cada vez que avances.*

## Estructura del repositorio

```
analisis-vuelos-aerolinea/
├── README.md                       ← estás aquí
├── CHANGELOG.md                    ← historial de versiones del análisis
├── .gitignore
├── docs/
│   ├── 00_brief.md                 ← pregunta de negocio, alcance, métricas
│   ├── 01_diccionario_datos.md     ← qué significa cada columna
│   ├── 02_calidad_datos.md         ← problemas detectados + cifras de control
│   ├── 03_log_decisiones.md        ← qué decidí, por qué y con qué impacto
│   ├── 04_medidas_dax.md           ← catálogo de medidas y su regla de negocio
│   ├── 05_hallazgos.md             ← conclusiones y limitaciones
│   ├── 06_guia_git.md              ← flujo de trabajo con Git
│   └── imagenes/                   ← capturas (modelo, informe)
├── data/
│   ├── raw/                        ← dato original. NO SE EDITA NUNCA
│   │   └── vuelos_aerolinea.csv
│   └── processed/                  ← dato limpio exportado (opcional)
├── powerbi/                        ← proyecto Power BI (.pbip)
└── outputs/
    ├── capturas/                   ← imágenes del dashboard
    └── informes/                   ← PDF / presentación final
```

## Cómo reproducir el análisis

1. Clona el repositorio: `git clone <URL-del-repo>`
2. Abre `powerbi/<nombre>.pbip` con Power BI Desktop (activa *Power BI Project* en Opciones → Vista previa si no aparece).
3. En Power Query, edita el parámetro `RutaDatos` (carpeta donde clonaste el repo) y actualiza.
4. Los pasos de limpieza están explicados en `docs/03_log_decisiones.md` y deben coincidir con las **cifras de control** de `docs/02_calidad_datos.md`.

## Fuente de los datos

- Archivo: `data/raw/vuelos_aerolinea.csv` (2.480 filas × 12 columnas, codificación UTF-8 con BOM).
- Origen: dataset simulado entregado para el ejercicio. **Cómo fue generado: pendiente de documentar** (ver `docs/01_diccionario_datos.md`, sección "Procedencia").

## Resultados principales

*(Completar al final. Máximo 3-5 puntos, cada uno con su cifra.)*

## Autoría

- Autor: Fernando Pozzo
- Creado: 2026-10-03
- Última actualización: 2026-10-03
