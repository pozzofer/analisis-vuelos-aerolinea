# Brief del proyecto

> **Cómo usar este archivo:** complétalo ANTES de seguir limpiando. Los puntos marcados con ✏️ los tienes que decidir tú; los demás ya están propuestos a partir del dataset. Si cambias algo más adelante, no borres: anótalo en `03_log_decisiones.md`.

## 1. Contexto

Una aerolínea nacional (ficticia) necesita optimizar su operación de vuelos. Se dispone de 2 años de operación simulada en 36 rutas entre 10 ciudades, con 3 tipos de avión.

## 2. Pregunta de negocio 

- [ ] ¿Qué rutas son rentables y cuáles no? (ingresos vs. costo por ruta)
- [ ] ¿Está el tipo de avión asignado correctamente a cada ruta? (ocupación según capacidad)
- [ ] ¿Dónde y cuándo se concentran las demoras y cancelaciones?
- [ ] ¿Hay estacionalidad en la demanda (pasajeros por mes)?


**Decisión que se tomaría con la respuesta:** 
- que rutas reforzar, reducir o rediseñar para la proxima temporada


## 3. Alcance

| | |
|---|---|
| Período | 2024-01-01 a 2025-12-31 (24 meses) |
| Rutas | 36 rutas dirigidas (A→B y B→A son rutas distintas), 10 ciudades |
| Granularidad | 1 fila = 1 vuelo |
| Fuera de alcance | Combustible por separado, tarifas por clase, tripulación, mantenimiento, datos externos (clima, feriados) — el dataset no los incluye |


## 4. Métricas y definiciones exactas

Estas son las definiciones implementadas en Power BI (ver `04_medidas_dax.md`).
Moneda: pesos argentinos (supuesto, dataset simulado).

### KPIs principales 

| Métrica | Definición | Tratamiento de casos especiales | Valor de control |
|---|---|---|---|
| **Margen total** | Σ Ingresos – Σ Costo_Vuelo. Se calcula por vuelo (columna `Margen`) y se suma. | Los cancelados **sí entran**: tienen costo y no ingreso, por lo que son una pérdida real. Los casos atípicos de ingresos (chárter, ingreso negativo) se conservan; ver D-006. | ≈ $9.935 millones |
| **Ocupación %** | Σ Pasajeros / Σ Capacidad | Excluye cancelados: un vuelo que no despegó no estuvo "vacío". Se pondera por asientos (no es el promedio de porcentajes por vuelo). | 78,33 % |
| **Puntualidad %** | Vuelos con `Estado_Vuelo = "A tiempo"` / vuelos operados | Excluye cancelados del denominador. Se usa `Estado_Vuelo` como fuente (ver D-007). | 81,68 % |

### Métricas de apoyo 

| Métrica | Definición | Tratamiento de casos especiales |
|---|---|---|
| **Demora promedio (min)** | Promedio de `Minutos_Demora` | Sobre **todos los vuelos operados** (incluye los puntuales, con 0 min); excluye cancelados (ver D-008). Conserva las demoras extremas de 512 y 688 min (D-009). |
| **Costo promedio por vuelo** | Promedio de `Costo_Vuelo` | Se usa para comprobar si las rutas que pierden plata tienen un costo mayor o si la causa es la baja ocupación. |

### Conceptos auxiliares

| Concepto | Definición |
|---|---|
| **Vuelo programado** | Fila única tras eliminar duplicados. Incluye cancelados. |
| **Vuelo operado** | Programado menos cancelado. |
| **Ruta dirigida** | A→B y B→A son rutas distintas (36 rutas). |

### Criterios transversales

- **Cancelados:** entran al margen (son pérdida) y salen de ocupación, puntualidad y demora (miden vuelos que despegaron).
- **Nulos en `Pasajeros` e `Ingresos`:** se eliminaron 27 filas (1,1 % de las 2.450 únicas). Queda un dataset de **2.423 filas**. Diferencia respecto de D-004, registrada en `03_log_decisiones.md`.
- **Nulos en `Minutos_Demora`:** 10 vuelos "A tiempo" completados con 0; 3 vuelos "Demorado" completados con 38 min (mediana de los demorados). Diferencia respecto de D-005.


## 5. Supuestos y limitaciones iniciales

- Dataset **simulado**: las conclusiones aplican al simulador, no al mercado real.
- No hay distancia (km), por lo que no se pueden calcular RASK/CASK reales.
- No hay hora del día, solo fecha: no se puede analizar demora por franja horaria.
- No hay matrícula de avión: no se puede medir utilización por aeronave.
- Los vuelos de una misma ruta son "iguales" salvo avión y fecha; no se conoce la frecuencia diaria programada.

## 6. Entregables

- [ ] Dashboard en Power BI (`powerbi/`)
- [ ] Documentación completa (esta carpeta `docs/`)
- [ ] Informe/resumen ejecutivo (`outputs/informes/`)

