# DATATHON IV — Predicción de Ventas: Panadería Salvador Echeverría

Proyecto de análisis y predicción de ventas para el DATATHON IV. Se analizan los datos históricos de ventas de una panadería en Málaga (2021–2023) y se construyen modelos predictivos para los 15 artículos principales de las familias BOLLERÍA, PANADERÍA y PASTELERÍA.

---

## Estructura del proyecto

```
DATATHONG/
├── Sesión_1_Carga_de_datos.ipynb          # Carga Excel → SQLite, SQL views, Meteostat, EDA inicial
├── Sesión_2_EDA_y_Modelado.ipynb          # 16 preguntas EDA + modelos RF para 15 artículos
├── Sesión_3_MLFlow.ipynb                  # Pipeline sklearn + MLFlow + deploy REST + predicciones BBDD
├── Sesión_4_Dashboard.pbix                # Dashboard Power BI — diseño profesional Salvador 1905
├── Salvador1905_Theme.json                # Tema corporativo Power BI (rojo #C8191E)
├── salvador_logo.png                      # Logo marca Salvador 1905
├── requirements.txt                        # Dependencias Python
├── .gitignore                              # Archivos excluidos del repo
├── README.md                               # Este archivo
├── ArticulosPanaderia.xlsx                 # Datos fuente (NO en repo)
├── Calendario.xlsx                         # Datos fuente (NO en repo)
├── CantidadPedida.xlsx                     # Datos fuente (NO en repo)
└── Datathon IV/
    ├── ventas_diarias.csv                  # Datos procesados (NO en repo)
    ├── pedidos.csv                         # Pedidos CSV (NO en repo)
    └── df_ventas_meteo.csv                 # Generado en Sesión 1 (NO en repo)
```

---

## Instalación

```bash
# Clonar el repositorio (privado)
git clone https://github.com/<usuario>/datathon-iv.git
cd datathon-iv

# Crear entorno virtual
python -m venv venv
source venv/bin/activate        # Linux/Mac
venv\Scripts\activate           # Windows

# Instalar dependencias
pip install -r requirements.txt
```

---

## Ejecución

Ejecutar los notebooks **en orden**:

1. `Sesión_1_Carga_de_datos.ipynb` — carga de datos, BBDD SQLite, Meteostat API
2. `Sesión_2_EDA_y_Modelado.ipynb` — EDA completo + modelos predictivos
3. `Sesión_3_MLFlow.ipynb` — MLOps pipeline
4. `Sesión_4_Dashboard.pbix` — abrir con **Power BI Desktop** (no requiere Python)

> Los archivos de datos (`.xlsx`, `.csv`) deben estar en la carpeta `DATATHONG/` y `DATATHONG/Datathon IV/` y **NO** se suben al repositorio.

---

## Dashboard Power BI — Sesión 4

**Archivo**: `Sesión_4_Dashboard.pbix`  
**Requisito**: Power BI Desktop (descarga gratuita en microsoft.com/power-bi)

### Páginas del informe

| Página | Contenido |
|---|---|
| **Portada** | Logo Salvador 1905, título del proyecto, branding corporativo rojo |
| **Dashboard de Ventas** | 3 gráficos interactivos + 2 segmentadores (AñoTrimestre, Artículo) |
| **Conclusiones** | 5 hallazgos clave del análisis + gráfico por familia |

### Columnas DAX creadas (tabla `ventas_diarias`)

```dax
AñoMes = FORMAT(ventas_diarias[FechaVenta], "YYYY-MM")
AñoTrimestre = FORMAT(ventas_diarias[FechaVenta], "YYYY") & "-T" & QUARTER(ventas_diarias[FechaVenta])
Año = YEAR(ventas_diarias[FechaVenta])
```

### Diseño

- **Tema corporativo**: `Salvador1905_Theme.json` — rojo `#C8191E` como color primario
- **Logo**: `salvador_logo.png` integrado en la portada
- **Mobile Layout**: activado en Dashboard de Ventas y Conclusiones
- **Gráficos**: barras verticales (familia), barras horizontales (artículos), línea temporal (AñoMes)

### Publicación (pendiente)

Para publicar en Power BI Service:
1. Abrir `Sesión_4_Dashboard.pbix` en Power BI Desktop
2. Clic en **Publicar** (esquina superior derecha)
3. Seleccionar espacio de trabajo en Power BI Service
4. Copiar el link público y añadirlo al documento Word de entrega

---

## Deploy MLflow

### 1. Lanzar el servidor de tracking

```bash
mlflow server --host 127.0.0.1 --port 5000
```

### 2. Ejecutar el notebook de MLflow

Abrir `Sesión_3_MLFlow.ipynb` y ejecutar todas las celdas.  
El notebook entrenará y registrará el modelo automáticamente.

### 3. Desplegar el modelo como API REST

```bash
export MLFLOW_TRACKING_URI=http://localhost:5000
mlflow models serve -m models:/datahon_iv@production -p 5001 --env-manager local
```

### 4. Hacer predicciones vía REST

```bash
curl -X POST http://127.0.0.1:5001/invocations \
  -H "Content-Type: application/json" \
  -d '{"dataframe_records": [{"dayofweek": 5, "monthofyear": 12, ...}]}'
```

---

## Datos

| Archivo | Descripción |
|---|---|
| `ArticulosPanaderia.xlsx` | Ventas brutas por hora (2017–2023) |
| `Calendario.xlsx` | Fechas y festivos |
| `CantidadPedida.xlsx` | Pedidos de abastecimiento |
| `Datathon IV/ventas_diarias.csv` | Ventas diarias procesadas |

---

## Modelo

- **Algoritmo**: RandomForestRegressor  
- **Features**: lags 1–7, lags 14/21/28 días, día de semana, mes, festivo, temperatura, lluvia, precio  
- **Familias**: BOLLERÍA, PANADERÍA, PASTELERÍA (top 5 artículos por familia)  
- **Modalidad**: Completa (15 artículos)  
- **Evaluación**: R² test + TimeSeriesSplit (5 folds)

---

## Ramas de desarrollo

```
main          ← releases estables
develop       ← integración
feature/eda   ← análisis exploratorio
feature/model ← modelado predictivo
feature/mlops ← MLFlow y deploy
```

---

## Penalizaciones a evitar

- ❌ NO subir datos al repo (`.xlsx`, `.csv`)  
- ❌ NO subir credenciales o contraseñas  
- ❌ NO subir la carpeta `mlruns/`  
- ✅ `.gitignore` configurado para excluir todo lo anterior
