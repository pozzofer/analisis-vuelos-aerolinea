# Guía de Git para este proyecto

Git guarda el historial de **qué cambió, cuándo y por qué**. Combinado con la documentación, te permite volver atrás ante un error y demostrar cómo llegaste a cada resultado.

## 1. Primeros pasos (una sola vez)

```bash
# Identidad (ya configurada en este repo, pero en tu equipo hazlo una vez global)
git config --global user.name "Tu Nombre"
git config --global user.email "tu@correo.com"

# Entrar al proyecto (el repo ya está inicializado y tiene el primer commit)
cd analisis-vuelos-aerolinea
git status
git log --oneline
```

### Subirlo a GitHub (opcional pero recomendado: es tu copia de seguridad y tu portafolio)

1. Crea un repositorio **vacío** en GitHub (sin README ni .gitignore).
2. Conéctalo y súbelo:
   ```bash
   git remote add origin https://github.com/<tu-usuario>/analisis-vuelos-aerolinea.git
   git push -u origin main
   ```

## 2. El ciclo diario de trabajo

```bash
git status                     # ¿qué cambió?
git diff                       # ver los cambios línea por línea
git add docs/03_log_decisiones.md   # elegir qué guardar (mejor archivo por archivo)
git commit -m "Aplica D-003: normaliza Destino desde Ruta (245 filas)"
git push                       # (si tienes remoto)
```

**Hábito clave:** un commit = un cambio con sentido. No mezcles "limpié fechas" con "armé el gráfico de rutas".

## 3. Cómo escribir buenos mensajes de commit

Formato: **verbo en presente + qué + dato de impacto**.

| ✅ Bien | ❌ Mal |
|---|---|
| `Aplica D-001: elimina 30 duplicados (2.480 → 2.450 filas)` | `cambios` |
| `Agrega medidas de puntualidad OTP y OTP A15` | `update` |
| `Documenta nulos de Pasajeros en diccionario` | `listo` |

Conviene referenciar la decisión (`D-00X`) cuando el commit corresponde a una decisión del log.

## 4. Qué se versiona y qué no

| Se versiona | No se versiona |
|---|---|
| Todo `docs/` | Archivos `.pbix` (binarios; ver abajo) |
| `data/raw/` (el CSV es chico, 2.480 filas) | Carpetas `.pbi/` y cachés locales |
| Proyecto Power BI en formato `.pbip` | Archivos temporales del sistema |
| Capturas finales en `outputs/` | |

El `.gitignore` ya está configurado para esto.

## 5. Power BI + Git: usa el formato de proyecto (.pbip)

Un `.pbix` es un archivo binario: Git no puede mostrarte qué cambió. El formato **Power BI Project (.pbip)** guarda el modelo y el informe como carpetas de texto (TMDL y JSON), así que `git diff` muestra qué medida o columna se modificó.

Para activarlo en Power BI Desktop:
1. *Archivo → Opciones y configuración → Opciones → Características en vista previa* y habilita **Guardar con formato de proyecto de Power BI (.pbip)** (el nombre exacto puede variar según la versión de Power BI Desktop).
2. *Archivo → Guardar como* y elige `.pbip`, guardando dentro de la carpeta `powerbi/` de este repo.
3. Cierra, reabre desde el `.pbip` y haz el primer commit.

Verifica estas opciones en tu versión de Power BI Desktop, porque Microsoft cambia la ubicación y el estado (vista previa o general) de esta función con frecuencia.

## 6. Ramas (cuando quieras experimentar sin riesgo)

```bash
git switch -c experimento-ocupacion     # crea y cambia a una rama nueva
# ...trabajas, haces commits...
git switch main
git merge experimento-ocupacion         # si salió bien, lo incorporas
```

Para un proyecto individual, trabajar en `main` está bien. Usa ramas cuando vayas a probar algo que podría salir mal.

## 7. Etiquetas para versiones entregables

```bash
git tag -a v1.0 -m "Entrega final del análisis"
git push --tags
```

Registra también la versión en `CHANGELOG.md`.

## 8. Errores comunes

- **Commits gigantes** al final del proyecto → commitea cada vez que cierres un paso.
- **Subir datos sensibles:** aquí el dataset es simulado, pero en proyectos reales no subas datos personales ni confidenciales a un repo público.
- **No documentar el porqué:** el commit dice *qué*; el log de decisiones dice *por qué*.
- **Editar `data/raw/`:** si lo modificas, pierdes el punto de partida. Toda transformación va en Power Query.

## 9. Rutina recomendada por sesión de trabajo

1. `git status` (¿dónde dejé las cosas?)
2. Trabajar un paso → anotar la decisión en `03_log_decisiones.md`
3. Actualizar documentación afectada (diccionario, medidas, README)
4. `git add` + `git commit` con mensaje claro
5. Al cerrar: actualizar la tabla de estado del README y `CHANGELOG.md` si hubo hito
