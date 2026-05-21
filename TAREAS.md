# Tareas - DATATHON IV: Análisis y Predicción de Ventas

El datathon está dividido en **4 sesiones** con evaluación independiente. La entrega final es un `.zip` con todos los notebooks + Power BI + Word con link al dashboard publicado.

---

## 🗄️ DATOS DISPONIBLES

Los datos de la panadería están disponibles **localmente** (ya NO en la BBDD antigua). Usar los archivos de la carpeta:

| Archivo | Contenido |
|---|---|
| `ArticulosPanaderia.xlsx` | Ventas brutas (FAMILIA, Tipo, FechaVenta, HoraVenta, Articulo, Cantidad, Precio, Importe) |
| `Calendario.xlsx` | Fechas y festivos (columna `festivo` contiene nombres de festivos, no S/N) |
| `CantidadPedida.xlsx` | Pedidos (Tipo, Fecha, Articulo, Cantidad, Precio, Importe) |
| `Datathon IV/ventas_diarias.csv` | Ventas diarias ya procesadas |
| `Datathon IV/pedidos.csv` | Pedidos en CSV |
| `Datathon IV/df_ventas_meteo.csv` | ✅ GENERADO — ventas + meteo combinadas (output Sesión 1) |
| `Datathon IV/modelos_rf.pkl` | ✅ GENERADO — modelos RandomForest de los 15 artículos |
| `Datathon IV/metricas_modelos.csv` | ✅ GENERADO — métricas R², MAE, RMSE por artículo |
| `panaderia.db` | ✅ GENERADO — base de datos SQLite local |

**⚠️ BBDD nueva (para classicmodels de otros módulos, NO tiene datos de panadería):**
- Host: `relational.fel.cvut.cz`
- Database: `classicmodels`
- Usuario: `guest`
- Contraseña: `ctu-relational`

**⚠️ NOTA TÉCNICA:** La columna `festivo` de `Calendario.xlsx` contiene el **nombre** del festivo (ej. "Día de la Madre") o "N" si no es festivo. Para binarizar usar: `df['es_festivo'] = df['festivo'].apply(lambda x: 'Festivo' if str(x).strip() != 'N' else 'No festivo')`

---

## ✅ SESIÓN 1 — Carga de datos y BBDD (10 puntos) — COMPLETADA

**Notebook entregado**: `Sesión_1_Carga_de_datos.ipynb`

### Lo que hace el notebook:
- Carga `ArticulosPanaderia.xlsx` → tabla `raw_ventas` en SQLite
- Carga `Calendario.xlsx` → tabla `raw_calendario`
- Carga `CantidadPedida.xlsx` → tabla `raw_pedidos`
- Crea tablas procesadas: `articulos_top`, `calendario_completo`, `ventas_diarias`
- Crea vistas: `ventas_diarias_estudio` (hasta abr 2023) y `ventas_diarias_estudio_completo`
- Benchmark de 3 métodos de carga (pandas+to_sql vs openpyxl+sqlite3 vs SQLAlchemy)
- Clase `DatabaseConnection` + extracción via query SQL
- Llamada a API Meteostat (`Point(36.721477, -4.363132)`) → variables `tavg`, `tmin`, `tmax`, `prcp`, `wdir`, `wspd`, `pres`
- JOIN ventas + meteorología → guarda `Datathon IV/df_ventas_meteo.csv`
- EDA inicial: cardinalidad, nulos (missingno), duplicados, evolución temporal

### Artículos del estudio (top 5 por familia según importe desde may 2021):
| Familia | Artículos |
|---|---|
| BOLLERIA | 3960, 6286, 3880, 5803, 6425 |
| PANADERIA | 968, 900, 417, 1043, 1084 |
| PASTELERIA | 5404, 6523, 5403, 6451, 6549 |

---

## ✅ SESIÓN 2 — EDA completo y Modelado (10 puntos) — COMPLETADA

**Notebook entregado**: `Sesión_2_EDA_y_Modelado.ipynb`

### Lo que hace el notebook:
- **Grupo 1** (P1-P4): descripción de filas, cardinalidad, nulos, duplicados
- **Grupo 2** (P5-P6): rango de fechas, fechas faltantes por artículo, gráfico art.6549, outliers IQR
- **Grupo 3** (P7-P10): evolución temporal general/por familia/por artículo, tendencia y estacionalidad
- **Grupo 4** (P11-P12): test ADF, primera diferencia, autocorrelación (ACF), columnas de fecha para confirmar patrón semanal
- **Grupo 5** (P13-P15): festivo vs cantidad, lluvia vs cantidad, quintiles de temperatura
- **Grupo 6** (P16): correlación precio-cantidad, scatter plots por artículo
- **Modelado Modalidad Completa**: 15 artículos (top 5 × 3 familias)
  - Transformadores: `CalendarTransformer`, `WeatherTransformer`, `ToSupervisedTransformer`
  - Lags: 1-7 días + 14, 21, 28 días
  - Modelo: RandomForestRegressor (200 árboles, max_depth=12)
  - Evaluación: R² test + TimeSeriesSplit (5 folds)
  - Guarda `modelos_rf.pkl` y `metricas_modelos.csv`

---

## ✅ SESIÓN 3 — MLOps con MLFlow (10 puntos) — COMPLETADA

**Notebook entregado**: `Sesión_3_MLFlow.ipynb`

### Lo que hace el notebook:
- Pipeline sklearn completo: `ColumnTransformer` + `SimpleImputer` + `MinMaxScaler` + `RandomForestRegressor`
- Servidor MLflow en `http://127.0.0.1:5000` (lanzar manualmente con `mlflow server --host 127.0.0.1 --port 5000`)
- Entrena dentro de `with mlflow.start_run()` para el artículo principal (3960)
- Logea: parámetros, métricas (`r2_test`, `r2_cross_val`, `mae_test`, `rmse_test`), tags (`product_id`, `product_family`), gráfico como artefacto
- Registra modelo en Model Registry como `datahon_iv`
- Entrena y registra los 15 artículos en MLflow
- Sube predicciones a tabla SQLite `Materials_Prediction_Group_Victor`
- Deploy REST (instrucción manual): `mlflow models serve -m models:/datahon_iv/1 -p 5001 --env-manager local`

### GIT (archivos creados):
- `README.md` — descripción completa del proyecto, instalación, ejecución, deploy MLflow
- `.gitignore` — excluye datos, mlruns/, credenciales, __pycache__
- `requirements.txt` — todas las dependencias con versiones mínimas

### ⚠️ Pendiente GIT:
- [ ] Crear repositorio privado en GitHub
- [ ] Crear branches (`develop`, `feature/eda`, `feature/model`, `feature/mlops`)
- [ ] Push sin datos ni carpeta `mlruns/`

---

## ✅ SESIÓN 4 — Power BI Dashboard (10 puntos) — COMPLETADA

**Archivo entregado**: `Sesión_4_Dashboard.pbix`

**Datos usados en Power BI**:
- `ArticulosPanaderia.xlsx` — ventas brutas
- `Calendario.xlsx` — fechas y festivos
- `CantidadPedida.xlsx` — pedidos
- `Datathon IV/ventas_diarias.csv` — ventas diarias procesadas
- `Datathon IV/pedidos.csv` — pedidos CSV

### Lo que hace el dashboard:

**Estructura del informe** (3 páginas):
1. **Portada** — Imagen de marca Salvador 1905, título y subtítulo en rojo corporativo (#C8191E)
2. **Página 1 — Dashboard de Ventas** — Vista principal con KPIs y gráficos interactivos
3. **Conclusiones** — Hallazgos clave del análisis y gráfico comparativo por familia

**Gráficos implementados**:
- Gráfico de barras verticales: Suma de Importe por FAMILIA (Bollería, Panadería, Pastelería)
- Gráfico de barras horizontales: Top artículos por importe
- Gráfico de líneas temporal: Evolución Mensual de Ventas (eje X = AñoMes `YYYY-MM`)
- Segmentadores (slicers): AñoTrimestre (`YYYY-TQ`) y Artículo

**Columnas DAX calculadas** (tabla `ventas_diarias`):
```
AñoMes = FORMAT(ventas_diarias[FechaVenta], "YYYY-MM")
AñoTrimestre = FORMAT(ventas_diarias[FechaVenta], "YYYY") & "-T" & QUARTER(ventas_diarias[FechaVenta])
Año = YEAR(ventas_diarias[FechaVenta])
```

**Diseño visual**:
- Tema corporativo aplicado: `Salvador1905_Theme.json` (rojo `#C8191E` como color primario)
- Logo de la empresa en portada: `salvador_logo.png`
- Tipografía y paleta coherentes con la marca Salvador 1905
- Títulos de segmentadores y gráficos en rojo corporativo

### Checklist:
- [x] **[3 pt]** Modelo de datos + gráficos básicos (ventas por familia, artículo, evolución temporal)
- [x] **[2 pt]** Tooltips sobre barras y líneas (datos al hover)
- [x] **[1 pt]** Diseño visual (tema rojo Salvador 1905, layout profesional)
- [x] **[1 pt]** Storytelling (página Conclusiones con 5 hallazgos clave)
- [x] **[3 pt]** Mobile Layout (diseño móvil activado en Página 1 y Conclusiones)
- [x] Publicar en Power BI Service y obtener link público
- [x] Crear Word con el link (`Entrega_Sesion4_Dashboard_PowerBI.docx`)

---

## 📦 ENTREGABLES — ZIP final

| # | Archivo | Estado |
|---|---|---|
| 1 | `Sesión_1_Carga_de_datos.ipynb` | ✅ Listo |
| 2 | `Sesión_2_EDA_y_Modelado.ipynb` | ✅ Listo |
| 3 | `Sesión_3_MLFlow.ipynb` | ✅ Listo |
| 4 | `Sesión_4_Dashboard.pbix` Power BI | ✅ Listo |
| 5 | `Entrega_Sesion4_Dashboard_PowerBI.docx` (Word con link publicado) | ✅ Listo |

---

## Material de referencia

- `Notebooks/20240319_Datathon_(1)_Carga_de_datos.ipynb` — notebook sesión 1 (base original)
- `Notebooks/20240319_Datathon_(2)_Obtención_de_datos.ipynb` — notebook sesión 2 (base original)
- `Notebooks/20240319_Datathon_(3)_EDA_preguntas.ipynb` — notebook sesión 3/EDA (base original)
- `Datathon_Edition_IV_mlflow_challenge.ipynb` — notebook sesión 3/MLFlow (base original)
- `Datathon IV/Evaluación Datathon.pdf` — rúbrica oficial de evaluación
- `20241112 Aclaraciones Datathon y enlace a carpeta con Dataset (1).pdf` — aclaraciones
- `20241112 Datathon (1) - Intro, carga datos y EDA.pdf` — introducción y contexto
