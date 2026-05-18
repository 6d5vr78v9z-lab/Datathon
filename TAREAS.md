# Tareas - DATATHON IV: Análisis y Predicción de Ventas

El datathon está dividido en **4 sesiones** con evaluación independiente. La entrega final es un `.zip` con todos los notebooks + Power BI + Word con link al dashboard publicado.

---

## 🗄️ DATOS DISPONIBLES

Los datos de la panadería están disponibles **localmente** (ya NO en la BBDD antigua). Usar los archivos de la carpeta:

| Archivo | Contenido |
|---|---|
| `ArticulosPanaderia.xlsx` | Ventas brutas (FAMILIA, Tipo, FechaVenta, HoraVenta, Articulo, Cantidad, Precio, Importe) |
| `Calendario.xlsx` | Fechas y festivos |
| `CantidadPedida.xlsx` | Pedidos (Tipo, Fecha, Articulo, Cantidad, Precio, Importe) |
| `Datathon IV/ventas_diarias.csv` | Ventas diarias ya procesadas |
| `Datathon IV/pedidos.csv` | Pedidos en CSV |

**⚠️ BBDD nueva (para classicmodels de otros módulos, NO tiene datos de panadería):**
- Host: `relational.fel.cvut.cz`
- Database: `classicmodels`
- Usuario: `guest`
- Contraseña: `ctu-relational`

---

## SESIÓN 1 — Carga de datos y BBDD (10 puntos)

**Notebook**: `Notebooks/20240319_Datathon_(1)_Carga_de_datos.ipynb`

### Parte BBDD (4 puntos)

- **[2 pt]** Script(s) completo con el proceso de creación de tablas y carga desde los archivos fuente:
  - Crear tablas en MySQL: `raw_ventas`, `raw_calendario`, `raw_pedidos`
  - Cargar los datos de `ArticulosPanaderia.xlsx` → `raw_ventas`
  - Cargar los datos de `Calendario.xlsx` → `raw_calendario`
  - Cargar los datos de `CantidadPedida.xlsx` → `raw_pedidos`
  - Ejecutar las queries SQL que crean las vistas procesadas:
    - `articulos_top` (top 5 artículos por familia según importe)
    - `calendario_dias` (calendario recursivo 2017-2023)
    - `calendario_completo` (calendario + festivos)
    - `ventas_diarias` (ventas agregadas por día con calendario y artículos top)
    - Vista `ventas_diarias_estudio` (filtro: solo ventas, desde mayo 2021, hasta abril 2023)
    - Vista `ventas_diarias_estudio_completo` (igual pero incluyendo mayo 2023)

- **[1 pt]** Mostrar **alternativas** de librerías/métodos para la carga (e.g., `mysql.connector` vs `SQLAlchemy` vs `pandas.read_excel`)

- **[1 pt]** **Justificación y benchmark** comparando las alternativas para estos orígenes de datos concretos

### Parte EDA — Conexión y extracción (6 puntos)

- **[1 pt]** Conectarse a la BBDD y extraer los datos correctamente usando `DatabaseConnection` y la query sobre `ventas_diarias_estudio`
- **[1 pt]** Llamar a la **API de Meteostat** correctamente para obtener datos meteorológicos del rango de fechas del dataset:
  - Variables: `tavg`, `tmin`, `tmax`, `prcp`, `wdir`, `wspd`, `pres`
  - Localización: Panadería Salvador Echeverría → `Point(36.721477644071705, -4.363132134392174)`
- **Hacer el JOIN** de ventas + meteorología por `fecha_venta`
- **[2 pt]** Exploración inicial del DataFrame resultante:
  - Descripción de qué significa cada fila
  - Cardinalidad y granularidad de cada variable
  - Análisis de valores nulos (% de nulos por columna, visualización con `missingno`)
  - Detección de duplicados (`duplicated(['fecha_venta', 'articulo'])`)
- **[2 pt]** Exploración más profunda (se valora originalidad y relevancia para el objetivo):
  - Ver más abajo las preguntas del EDA

---

## SESIÓN 2 — EDA completo y Modelado (10 puntos)

**Notebook**: `Notebooks/20240319_Datathon_(2)_Obtención_de_datos.ipynb` y/o `Notebooks/20240319_Datathon_(3)_EDA_preguntas.ipynb`

**Notebook de apoyo**: https://colab.research.google.com/drive/12LGFGasEi98TzfhBeeEKHeEDQsAaZ7WH?usp=sharing

### EDA — Preguntas (continúa desde Sesión 1)

**Grupo 1 — Inspección inicial:**
1. Describe qué significa cada fila del conjunto de datos
2. ¿Cuántos valores únicos hay en cada variable? ¿Qué insight se observa al comparar los únicos de `articulo` con los de `precio`?
3. ¿Cuántos valores nulos hay en cada variable?
4. ¿Hay duplicados?

**Grupo 2 — Análisis temporal:**
5. ¿Cuál es el rango de fechas? Dividiendo por producto, ¿hay fechas faltantes? Crear gráfico temporal de `cantidad` para el producto `6549`
6. Separando por producto, ¿hay outliers en la variable `cantidad`? (método IQR)

**Grupo 3 — Evolución de la variable objetivo:**
7. Crear gráfico de la evolución temporal **general** de `cantidad` (agrupada por mes)
8. Crear gráfico de la evolución temporal **por familia** de `cantidad` (agrupada)
9. Crear gráfico de la evolución temporal **por artículo** de `cantidad` (agrupada)
10. ¿A simple vista, hay tendencia y/o estacionalidad en las series temporales anteriores?

**Grupo 4 — Análisis estadístico de estacionalidad:**
11. Aplicar técnica estadística para detectar estacionalidad: tomar la **primera diferencia** y hacer un **análisis de autocorrelación** (`plot_acf`). Usar test ADF para verificar estacionariedad
12. Sin primera diferencia, crear columnas de fecha (`weekofyear`, `monthofyear`, `dayofweek`, `dayofmonth`, `dayofyear`) y hacer agrupaciones/gráficos para confirmar el patrón semanal detectado en autocorrelación

**Grupo 5 — Variables exógenas vs cantidad:**
13. ¿El comportamiento de compra (`cantidad`) cuando es festivo es superior a cuando no lo es?
14. ¿El comportamiento de compra cuando llueve es superior a cuando no llueve?
15. Dividir `tavg_w` en quintiles y mostrar con gráfico de barras si `cantidad` es superior en algún quintil

**Grupo 6 — Precio y consumo:**
16. ¿Un incremento en el precio reduce la propensión a consumir de un artículo?

### Modelado (10 puntos)

- **[3 pt]** Describir y **justificar qué variables** (días pasados, semanas pasadas y exógenas) son más representativas del comportamiento de ventas del artículo estudiado
- **[4 pt]** **Desarrollar, modelar, entrenar y evaluar** un modelo predictivo para el artículo `3960` (modalidad sencilla) o para todos los artículos (modalidad completa)
- **[3 pt]** Repetir los pasos anteriores para **el resto de artículos** de la base de datos

**Modalidad sencilla**: solo producto `3960`

**Modalidad completa**: top 5 productos de cada una de las 3 familias (BOLLERIA, PANADERIA, PASTELERIA) → 15 artículos en total

---

## SESIÓN 3 — MLOps con MLFlow (10 puntos)

**Notebook**: `Datathon_Edition_IV_mlflow_challenge.ipynb`

### Pipeline y MLFlow (7 puntos)

- **[1 pt]** Crear un **Pipeline de Scikit-Learn** con los transformadores (`CalendarTransformer`, `WeatherTransformer`, `ToSupervisedTransformer`) + `ColumnTransformer` + `SimpleImputer` + modelo
- **[0.5 pt]** Lanzar un **servidor MLflow local**: `mlflow server --host 127.0.0.1 --port 5000`
- **[1 pt]** Entrenar y evaluar el modelo **dentro de un MLflow Run** (`with mlflow.start_run()`)
- **[1 pt]** **Logear modelo y métricas** en MLflow (métricas `r2_cross_val`, `r2_test` + tags `product_id`, `product_family`)
- **[1 pt]** **Registrar el modelo** en el Model Registry de MLflow: `mlflow.register_model(...)`
- **[1 pt]** **Desplegar el modelo** a API REST:
  ```bash
  export MLFLOW_TRACKING_URI=http://localhost:5000
  mlflow models serve -m models:/datahon_iv@production -p 5001 --env-manager local
  ```
- **[1 pt]** **Hacer predicciones** via REST (POST a `http://127.0.0.1:5001/invocations`)
- **[0.5 pt]** **Subir predicciones a la BBDD**: tabla `Materials_Prediction_Group_{Nombre}` con columnas `fecha`, `cantidad`, `articulo`, `familia`

### Código y Documentación (1 punto)
- **[0.5 pt]** Legibilidad del código
- **[0.5 pt]** Documentación del notebook (títulos, subtítulos, celdas de texto)

### GIT (2 puntos — repositorio en **privado**)
- **[1 pt]** `README.md` completo (descripción del proyecto, instalación, ejecución, deploy MLflow)
- **[0.5 pt]** `.gitignore` + `requirements.txt`
- **[0.5 pt]** Usar **branches** para desarrollo (no trabajar en `main` directamente)

### Bonus
- GridSearch para optimizar hiperparámetros
- Probar con más productos/artículos

### ⚠️ Penalizaciones
- **[-1 pt]** Push de datos (CSVs, Excels) al repo
- **[-3 pt]** Push de contraseñas o credenciales (borrar antes de commitear)
- **[-1 pt]** Push de carpeta `mlruns/`
- **[-0.5 pt]** Faltar `requirements.txt` o `.gitignore`
- **[-1 pt]** README.md de mala calidad

---

## SESIÓN 4 — Power BI Dashboard (10 puntos)

**Archivo a entregar**: `.pbix`

**Datos disponibles para el dashboard** (misma carpeta `DATATHONG`):
- `ArticulosPanaderia.xlsx`
- `Calendario.xlsx`
- `CantidadPedida.xlsx`
- `ventas_diarias.csv`
- `pedidos.csv`

### Tareas (10 puntos)
- **[3 pt]** Creación del modelo de datos y **gráficos básicos** (ventas por familia, por artículo, evolución temporal, etc.)
- **[2 pt]** Crear **Tooltips** personalizados en los gráficos
- **[1 pt]** **Diseño** visual del dashboard (colores, layout, coherencia visual)
- **[1 pt]** **Storytelling**: que el dashboard cuente una historia, con conclusiones claras
- **[3 pt]** Crear **Mobile Layout** (vista adaptada para móvil)

> **Importante**: Publicar el dashboard en Power BI Service y compartir el link en un Word. Todos los que tengan el link deben poder verlo sin necesidad de cuenta.

---

## 📦 ENTREGABLES — ZIP final

El archivo `.zip` de entrega debe contener:

1. **Notebook Sesión 1** (`Datathon_(1)_Carga_de_datos.ipynb`) — BBDD y carga
2. **Notebook Sesión 2** (`Datathon_(2/3)_EDA_y_Modelado.ipynb`) — EDA + modelo predictivo
3. **Notebook Sesión 3** (`Datathon_Edition_IV_mlflow_challenge.ipynb`) — MLOps
4. **Fichero Power BI** (`.pbix`) — dashboard
5. **Word** con el link al dashboard publicado (accesible para cualquiera con el link)

---

## Material de referencia

- `Notebooks/20240319_Datathon_(1)_Carga_de_datos.ipynb` — notebook sesión 1 (base)
- `Notebooks/20240319_Datathon_(2)_Obtención_de_datos.ipynb` — notebook sesión 2 (base)
- `Notebooks/20240319_Datathon_(3)_EDA_preguntas.ipynb` — notebook sesión 3/EDA (base)
- `Datathon_Edition_IV_mlflow_challenge.ipynb` — notebook sesión 3/MLFlow (base)
- `Datathon IV/Evaluación Datathon.pdf` — rúbrica oficial de evaluación
- `20241112 Aclaraciones Datathon y enlace a carpeta con Dataset (1).pdf` — aclaraciones
- `20241112 Datathon (1) - Intro, carga datos y EDA.pdf` — introducción y contexto
