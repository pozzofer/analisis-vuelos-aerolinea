# Carpeta Power BI

Guarda aquí el proyecto de Power BI en formato **.pbip** (ver `docs/06_guia_git.md`, sección 5).

## Organización sugerida de consultas en Power Query

Agrupa las consultas en carpetas para que el flujo se lea de arriba hacia abajo:

| Carpeta | Consulta | Qué hace | ¿Se carga al modelo? |
|---|---|---|---|
| `00_Parametros` | `RutaDatos` | Carpeta local donde está el repositorio (cada persona la ajusta) | No |
| `01_Raw` | `Vuelos_Raw` | Lee `data/raw/vuelos_aerolinea.csv` sin ningún cambio | No |
| `02_Staging` | `Vuelos_Staging` | Aplica D-001 a D-010 en orden, un paso por decisión | No |
| `03_Modelo` | `Vuelos` | Tabla de hechos final (referencia a `Vuelos_Staging`) | Sí |
| `03_Modelo` | `Calendario` | Dimensión de fechas (2024-01-01 a 2025-12-31) | Sí |
| `03_Modelo` | `Ciudades` | Dimensión de 10 ciudades (relación con Origen y Destino) | Sí |
| `03_Modelo` | `Rutas` | Dimensión de 36 rutas | Sí |
| `03_Modelo` | `Aviones` | Dimensión: `Tipo_Avion` y `Capacidad` | Sí |

## Cómo documentar dentro de Power Query

- **Renombra cada paso aplicado** con el ID de la decisión: `D-001 Eliminar duplicados`, `D-002 Fecha_Vuelo`, `D-003 Destino desde Ruta`…
- **Pon una descripción** en cada paso (clic derecho → Propiedades) con el motivo y las filas afectadas.
- **Descripción de cada consulta** en una frase (qué contiene, de dónde viene).
- Deja `Vuelos_Raw` **intacta**: es tu punto de comparación.

## Orden recomendado de los pasos en `Vuelos_Staging`

1. Importar con UTF-8 y arreglar nombre de la 1ª columna (D-010)
2. Eliminar duplicados exactos (D-001)
3. Crear `Fecha_Vuelo` y quitar `Fecha` (D-002)
4. Reconstruir `Destino` desde `Ruta` (D-003)
5. Tipos de datos (D-010)
6. Rellenar con 0 las demoras nulas de vuelos "A tiempo" (D-005)
7. Crear `Ingreso_Invalido` y `Revisar` (D-006)

Después de **cada** paso, anota cuántas filas quedaron y compara con `docs/02_calidad_datos.md` (sección 3).

## Pruebas rápidas del modelo

Cuando cargues el modelo, valida estos números (si no coinciden, revisa la limpieza antes de seguir):

- `Vuelos` tiene **2.450** filas
- `Destino` tiene **10** valores únicos
- `Fecha_Vuelo` va de **2024-01-01** a **2025-12-31** sin errores
