# Brief del proyecto

> **Cómo usar este archivo:** complétalo ANTES de seguir limpiando. Los puntos marcados con ✏️ los tienes que decidir tú; los demás ya están propuestos a partir del dataset. Si cambias algo más adelante, no borres: anótalo en `03_log_decisiones.md`.

## 1. Contexto

Una aerolínea nacional (ficticia) necesita optimizar su operación de vuelos. Se dispone de 2 años de operación simulada en 36 rutas entre 10 ciudades, con 3 tipos de avión.

## 2. Pregunta de negocio ✏️

"Optimizar vuelos" es demasiado amplio. Elige **una pregunta principal** y hasta dos secundarias. Candidatas que el dataset permite responder:

- [ ] ¿Qué rutas son rentables y cuáles no? (ingresos vs. costo por ruta)
- [ ] ¿Está el tipo de avión asignado correctamente a cada ruta? (ocupación según capacidad)
- [ ] ¿Dónde y cuándo se concentran las demoras y cancelaciones?
- [ ] ¿Hay estacionalidad en la demanda (pasajeros por mes)?

**Pregunta principal elegida:** ✏️ _______________________________________________

**Decisión que se tomaría con la respuesta:** ✏️ (ej. "recortar frecuencias en rutas con margen negativo", "cambiar Boeing por Embraer en rutas con ocupación < 60 %")

**¿Quién la pregunta?** ✏️ (ej. Gerencia de Planificación de Red)

## 3. Alcance

| | |
|---|---|
| Período | 2024-01-01 a 2025-12-31 (24 meses) |
| Rutas | 36 rutas dirigidas (A→B y B→A son rutas distintas), 10 ciudades |
| Granularidad | 1 fila = 1 vuelo |
| Fuera de alcance | Combustible por separado, tarifas por clase, tripulación, mantenimiento, datos externos (clima, feriados) — el dataset no los incluye |

## 4. Métricas y definiciones exactas

Una métrica sin definición exacta no es comparable. Estas son las propuestas; **confírmalas o ajústalas** y mantenlas idénticas en Power BI (`04_medidas_dax.md`).

| Métrica | Definición propuesta | Tratamiento de casos especiales |
|---|---|---|
| **Vuelos programados** | Cantidad de filas únicas (tras eliminar duplicados) | Incluye cancelados |
| **Vuelos operados** | Programados – cancelados | |
| **Tasa de cancelación** | Cancelados / Programados | |
| **Puntualidad (OTP)** | Operados con `Estado_Vuelo = "A tiempo"` / Operados | Excluye cancelados del denominador. Ver D-007 sobre el umbral de 15 min |
| **Demora promedio (min)** | Promedio de `Minutos_Demora` | Decidir: ¿solo vuelos demorados o todos los operados? Ver D-008 |
| **Factor de ocupación** | Σ Pasajeros / Σ Capacidad | Solo vuelos operados con `Pasajeros` no nulo |
| **Ingresos** | Σ `Ingresos` | Moneda: ✏️ confirmar (supuesto: pesos argentinos) |
| **Costo** | Σ `Costo_Vuelo` | Los cancelados sí tienen costo (ver diccionario) |
| **Margen** | Ingresos – Costo | Puede ser negativo |
| **Margen %** | Margen / Ingresos | |
| **Ingreso por pasajero** | Σ Ingresos / Σ Pasajeros | |
| **Costo por asiento ofrecido** | Σ Costo / Σ Capacidad | Aproxima el CASK sin distancia (el dataset no tiene km) |

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

## 7. Plan

| Hito | Fecha objetivo |
|---|---|
| Brief cerrado | ✏️ |
| Limpieza terminada y cuadrada con cifras de control | ✏️ |
| Modelo + medidas DAX | ✏️ |
| Dashboard v1 | ✏️ |
| Conclusiones y entrega | ✏️ |
