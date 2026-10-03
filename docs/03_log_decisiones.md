# Log de decisiones

Registro de cada decisión que afecta los resultados. Regla de oro: **si cambia un número del informe, tiene que tener una entrada acá.**

**Estados:** 🟡 Propuesta (hay que confirmarla) · 🟢 Aceptada (ya aplicada en Power Query) · 🔴 Descartada/Reemplazada

> Las decisiones D-001 a D-010 vienen **propuestas** a partir del diagnóstico (`02_calidad_datos.md`). Léelas, cambia las que no compartas y, cuando las apliques en Power Query, pasa el estado a 🟢 y completa "Impacto real".

---

## D-001 · Eliminar duplicados exactos 🟡
- **Fecha:** 2026-10-03
- **Problema:** Q-01. 30 filas repetidas idénticas en las 12 columnas (mismo `ID_Vuelo`, mismos datos).
- **Decisión:** Eliminar duplicados considerando **todas las columnas**; conservar la primera aparición.
- **Motivo:** Un vuelo no puede ocurrir dos veces con el mismo ID. Mantenerlos inflaría vuelos, ingresos y costos.
- **Alternativas descartadas:** Deduplicar solo por `ID_Vuelo` (equivalente aquí, porque no hay IDs repetidos con datos distintos, pero menos seguro si el dato cambia).
- **Impacto esperado:** 2.480 → 2.450 filas (−1,2 %).
- **Impacto real:** ✏️
- **Dónde se aplica:** Power Query, consulta `Vuelos_Staging`, paso `Eliminar duplicados exactos`.

## D-002 · Unificar el formato de `Fecha` 🟡
- **Problema:** Q-02. Dos formatos: `aaaa-mm-dd` (1.606) y `dd/mm/aaaa` (874).
- **Decisión:** Convertir a tipo Fecha leyendo cada fila con su formato. Las fechas con `/` se interpretan como **día/mes/año**.
- **Motivo:** Verificado que el orden cronológico de `ID_Vuelo` solo se cumple con día/mes (como mes/día, 534 fechas serían inválidas).
- **Alternativas descartadas:** Cambiar la configuración regional de toda la consulta (arriesga convertir mal las fechas ISO).
- **Cómo aplicarlo (columna personalizada en Power Query):**
  ```m
  // Fecha_Vuelo: ISO (aaaa-mm-dd) o dd/mm/aaaa; cualquier otra cosa queda en error visible
  = try Date.FromText([Fecha], [Format="yyyy-MM-dd", Culture="es-AR"])
    otherwise Date.FromText([Fecha], [Format="dd/MM/yyyy", Culture="es-AR"])
  ```
- **Control:** 0 errores; mín = 2024-01-01; máx = 2025-12-31.
- **Impacto real:** ✏️

## D-003 · Normalizar `Destino` usando `Ruta` como referencia 🟡
- **Problema:** Q-03. 25 escrituras para 10 ciudades (`Bs As`, `BUENOS AIRES`, `Cordoba`, `San Miguel de Tucumán`, `Puerto Iguazú`…), 245 filas.
- **Decisión:** Reconstruir `Destino` a partir de `Ruta` (parte posterior al guion).
- **Motivo:** `Ruta` está limpia y es 100 % consistente con `Origen` + `Destino` normalizado. Evita mantener una tabla de reemplazos que se desactualiza.
- **Alternativas descartadas:** Lista de "Reemplazar valores" (válida, pero hay que mantenerla a mano si aparecen variantes nuevas).
- **Cómo aplicarlo:**
  ```m
  // Los nombres de ciudad no contienen guion, así que es seguro cortar en el primer "-"
  = Text.AfterDelimiter([Ruta], "-")
  ```
- **Control:** `Destino` queda con exactamente 10 valores únicos.
- **Impacto real:** ✏️

## D-004 · Nulos en `Pasajeros` e `Ingresos`: no imputar 🟡
- **Problema:** Q-04 y Q-05. 14 y 13 nulos, en vuelos no cancelados.
- **Decisión:** Dejarlos como nulos. Excluir de cada ratio **solo la fila afectada** (ej. el factor de ocupación no cuenta esa fila ni en numerador ni en denominador).
- **Motivo:** Son < 0,6 % de las filas; imputar inventaría datos y puede sesgar rutas con pocos vuelos.
- **Alternativas descartadas:** Imputar con la media/mediana por ruta (introduce supuestos); eliminar la fila entera (pierde costo y estado válidos).
- **Riesgo:** Si Power BI suma nulos como 0 en un cociente, sesga el resultado. Cuidar el DAX (ver `04_medidas_dax.md`).
- **Impacto real:** ✏️

## D-005 · Nulos en `Minutos_Demora`: inferir 0 solo si `Estado_Vuelo = "A tiempo"` 🟡
- **Problema:** Q-06. 13 nulos: 10 en vuelos "A tiempo", 3 en "Demorado".
- **Decisión:** Rellenar con 0 los nulos de vuelos "A tiempo". Dejar nulos los 3 de "Demorado".
- **Motivo:** En todo el dataset, "A tiempo" siempre tiene demora 0 (regla verificada), así que es una inferencia determinista, no una estimación. En "Demorado" no se puede inferir el valor.
- **Alternativas descartadas:** Imputar los 3 "Demorado" con la mediana (38 min): afectaría el promedio sin base.
- **Impacto real:** ✏️

## D-006 · Ingresos anómalos: marcar y excluir de métricas de ingresos 🟡
- **Problema:** Q-07, Q-08, Q-09.
  - AP-01172: ingreso **negativo** (−6.515.078), vuelo "A tiempo".
  - AP-00722: ingreso **105.712.830** con 94 pasajeros → ≈ 1,12 M por pasajero, ~10× la mediana (≈ 113 mil). Sospecha: dígito de más.
  - AP-00490 y AP-00753: "A tiempo" con **0 pasajeros e ingresos 0** y costo normal. Un vuelo operado vacío es raro.
- **Decisión:** Crear dos columnas indicadoras: `Ingreso_Invalido` (Sí/No) y `Revisar` (Sí/No). Los dos casos con ingreso inválido (el negativo AP-01172 y el extremo AP-00722) se marcan en `Ingreso_Invalido` y se **excluyen de las métricas de ingresos y margen**, pero el vuelo se mantiene para conteos, puntualidad y costo. Los 2 vuelos vacíos se mantienen en todas las métricas y se marcan en `Revisar` para consultarlo con quien generó el dataset.
- **Motivo:** Eliminar el vuelo perdería información válida; corregir el valor (dividir por 10) sería un supuesto sin respaldo.
- **Alternativas descartadas:** Corregir AP-00722 dividiéndolo por 10; tomar el valor absoluto del negativo.
- **Impacto esperado:** El valor extremo por sí solo suma ~95 M de más en `Buenos Aires-Iguazú`. Medir el margen total antes y después.
- **Impacto real:** ✏️

## D-007 · Definición de puntualidad: respetar `Estado_Vuelo` 🟡
- **Problema:** Q-11. El estándar de la industria (A15) considera puntual hasta 15 min incluidos. En este dataset hay 16 vuelos "Demorado" con exactamente 15 min.
- **Decisión:** Para el OTP principal usar `Estado_Vuelo = "A tiempo"` tal como viene. Dejar una medida alternativa `OTP A15` (demora ≤ 15) para comparación.
- **Motivo:** El dato fuente ya clasifica; cambiarlo sin saber cómo lo generó el simulador sería arbitrario. La diferencia es chica (alrededor de 0,7 puntos de OTP).
- **Alternativas descartadas:** Reclasificar los 16 vuelos como "A tiempo".
- **Impacto real:** ✏️ (reportar ambos OTP)

## D-008 · Demora promedio: reportar dos versiones 🟡
- **Problema:** "Demora promedio" es ambigua. Con solo demorados = 49,31 min; con todos los operados = 9,04 min.
- **Decisión:** Mostrar "Demora promedio de vuelos demorados" como métrica principal y la mediana al lado. Etiquetar siempre cuál es.
- **Motivo:** Es más informativo para decidir qué hacer con las demoras; el promedio sobre todos los operados diluye el problema.
- **Impacto real:** n/a (decisión de definición)

## D-009 · Demoras extremas: mantener, no eliminar 🟡
- **Problema:** Q-10. AP-01508 (688 min) y AP-00315 (512 min); el resto no pasa de 300.
- **Decisión:** Mantenerlas (son demoras posibles, no errores evidentes) y apoyarse en la mediana. Mostrar el promedio sin ellas como control de sensibilidad (49,31 → 47,84 min con solo quitar la de 688).
- **Alternativas descartadas:** Eliminarlas como outliers.
- **Impacto real:** ✏️

## D-010 · Importación del CSV (BOM y tipos) 🟡
- **Problema:** Q-12. El archivo es UTF-8 con BOM; la 1ª columna puede importarse como `﻿ID_Vuelo`.
- **Decisión:** Importar con origen de archivo **UTF-8 (65001)**, verificar que la 1ª columna se llame exactamente `ID_Vuelo` y renombrarla si no. `Pasajeros`, `Ingresos`, `Minutos_Demora` pasan a **Número entero**.
- **Impacto real:** n/a

---

## Plantilla para nuevas decisiones

```
## D-0XX · Título corto en forma de acción
- **Fecha:**
- **Problema:** qué dato/situación, cuántas filas.
- **Decisión:** qué se hace exactamente.
- **Motivo:** por qué esta opción.
- **Alternativas descartadas:** y por qué.
- **Impacto real:** filas afectadas y cómo cambió una métrica (antes → después).
- **Dónde se aplica:** consulta y paso de Power Query / medida DAX.
```
