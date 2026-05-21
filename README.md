# DATATHON IV — Predicción de Ventas: Panadería Salvador Echeverría

Proyecto de análisis y predicción de ventas para el DATATHON. Se analizan los datos históricos de ventas de una panadería en Málaga (2021–2023) y se construyen modelos predictivos para los 15 artículos principales de las familias BOLLERÍA, PANADERÍA y PASTELERÍA.

---

## Estructura del proyecto

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
