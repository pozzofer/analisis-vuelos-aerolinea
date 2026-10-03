# Hallazgos y conclusiones

> Completar al final del análisis. Regla: **cada hallazgo = afirmación + cifra + dónde verlo + qué se recomienda**. Si no tiene cifra, no es un hallazgo.

⚠️ **Recordatorio:** el dataset es simulado. Estos hallazgos describen el simulador, no el mercado real.

## Pregunta principal (copiada de `00_brief.md`)

✏️

## Respuesta en una frase

✏️ (si solo se lee esto, ¿qué debería saber quien decide?)

## Hallazgos

| # | Hallazgo | Cifra | Página del dashboard | Confianza (alta / media / baja) | Recomendación |
|---|---|---|---|---|---|
| H-1 | ✏️ | | | | |
| H-2 | ✏️ | | | | |
| H-3 | ✏️ | | | | |

**Ejemplo de redacción** (no es un resultado real, solo el formato): *"10 de las 36 rutas tienen margen total negativo en los 24 meses (antes de tratar outliers). Verificar tras D-006."*

## Cómo cambian los resultados según mis decisiones (análisis de sensibilidad)

Esta tabla muestra qué tanto dependen las conclusiones de las decisiones de limpieza. Es lo que más genera confianza.

| Métrica | Sin tratar (cifras de control) | Con limpieza completa | Diferencia | Decisión responsable |
|---|---|---|---|---|
| Ingresos totales | 23.823.951.488 | ✏️ | | D-006 |
| Margen total | 9.923.076.098 | ✏️ | | D-006 |
| Rutas con margen negativo | 10 de 36 | ✏️ | | D-006 |
| OTP | 81,63 % | ✏️ | | D-007 (alternativa A15) |
| Demora promedio (demorados) | 49,31 min | ✏️ | | D-009 |

## Limitaciones

- Dataset simulado; no se pueden extrapolar conclusiones al mercado real.
- Sin distancia, hora del día, matrícula de avión ni tarifas: se omiten análisis de RASK/CASK reales, franjas horarias y utilización por aeronave.
- Nulos y valores anómalos tratados según `03_log_decisiones.md`; otras decisiones razonables darían números distintos (ver sensibilidad).
- ✏️ Agregar las que surjan durante el análisis.

## Próximos pasos sugeridos

- ✏️ (qué datos adicionales pedirías, qué análisis harías con más tiempo)

## Uso de IA en este proyecto

- ✏️ Anotar qué partes se apoyaron en un asistente de IA (ej. borradores de documentación y medidas DAX) y cómo se validaron contra las cifras de control.
