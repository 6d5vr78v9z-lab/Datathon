# 🥐 Datathon IV — Análisis y Predicción de Ventas de Panadería

Proyecto completo del **Datathon IV**: análisis de ventas de una panadería, modelado predictivo de series temporales y despliegue con MLflow, desarrollado en Python y visualizado con Power BI.

---

## 📋 Descripción del proyecto

Este datathon aborda el ciclo completo de un proyecto de ciencia de datos sobre datos reales de una panadería:

| Sesión | Descripción |
|--------|-------------|
| **Sesión 1** | Carga de datos y base de datos MySQL (ELT, vistas SQL) |
| **Sesión 2** | EDA completo (16 preguntas) + modelado predictivo |
| **Sesión 3** | MLOps con MLflow: pipeline, registro y despliegue de modelos |
| **Sesión 4** | Dashboard Power BI con mobile layout |

---

## 📁 Estructura del repositorio

```
datathong/
├── Notebooks/
│   ├── 20240319_Datathon_(1)_Carga_de_datos.ipynb     # Sesión 1: BBDD y ELT
│   ├── 20240319_Datathon_(2)_Obtención_de_datos.ipynb  # Sesión 2: Extracción y join
│   └── 20240319_Datathon_(3)_EDA_preguntas.ipynb       # Sesión 2: EDA + modelado
├── Datathon_Edition_IV_mlflow_challenge.ipynb           # Sesión 3: MLOps
├── requirements.txt
├── .gitignore
└── README.md
```

> ⚠️ Los archivos de datos (`.xlsx`, `.csv`) **no están incluidos** en el repositorio para evitar penalizaciones.

---

## 🛠️ Instalación

### Requisitos previos
- Python 3.9+
- MySQL Server (local o remoto)
- MLflow
- (Opcional) Power BI Desktop para la Sesión 4

### Clonar el repositorio

```bash
git clone https://github.com/<usuario>/datathong.git
cd datathong
```

### Instalar dependencias

```bash
pip install -r requirements.txt
```

### Configurar variables de entorno

Crear un archivo `.env` en la raíz del proyecto (no incluido en el repo):

```bash
DB_HOST=your_mysql_host
DB_USER=your_username
DB_PASSWORD=your_password
DB_DATABASE=data
```

---

## 🚀 Ejecución

### Sesión 1 — Carga de datos

Abrir y ejecutar el notebook:

```bash
jupyter notebook Notebooks/20240319_Datathon_(1)_Carga_de_datos.ipynb
```

Asegurarse de que los archivos de datos (`ArticulosPanaderia.xlsx`, `Calendario.xlsx`, `CantidadPedida.xlsx`) estén disponibles localmente.

### Sesión 2 — EDA y Modelado

```bash
jupyter notebook Notebooks/20240319_Datathon_(3)_EDA_preguntas.ipynb
```

### Sesión 3 — MLOps con MLflow

#### 1. Lanzar el servidor MLflow

```bash
mlflow server --host 127.0.0.1 --port 5000
```

#### 2. Ejecutar el notebook principal

```bash
jupyter notebook Datathon_Edition_IV_mlflow_challenge.ipynb
```

El notebook entrena el modelo, lo registra en MLflow y lo promueve a producción.

#### 3. Desplegar el modelo como API REST

```bash
export MLFLOW_TRACKING_URI=http://localhost:5000
mlflow models serve -m models:/datahon_iv@production -p 5001 --env-manager local
```

#### 4. Hacer predicciones via REST

```bash
curl -X POST http://127.0.0.1:5001/invocations \
  -H "Content-Type: application/json" \
  -d '{"dataframe_records": [{"fecha_venta": "2023-05-01", "articulo": 3960}]}'
```

---

## 📊 Datos

Los datos utilizados en este proyecto corresponden a ventas reales de la **Panadería Salvador Echeverría** (Málaga). No están incluidos en el repositorio. Contacta al organizador del datathon para obtener acceso.

| Archivo | Descripción |
|---------|-------------|
| `ArticulosPanaderia.xlsx` | Ventas brutas (familia, artículo, fecha, cantidad, precio) |
| `Calendario.xlsx` | Fechas y festivos |
| `CantidadPedida.xlsx` | Pedidos realizados |
| `ventas_diarias.csv` | Ventas diarias preprocesadas |
| `pedidos.csv` | Pedidos en formato CSV |

---

## 📈 Resultados

- **Artículo estudiado**: `3960` (modalidad sencilla)
- **Familias**: BOLLERÍA, PANADERÍA, PASTELERÍA (modalidad completa: top 5 artículos por familia)
- **Métricas logueadas en MLflow**: `r2_cross_val`, `r2_test`

---

## 🏗️ Workflow de ramas (Git)

```
main         ← rama estable (solo merges)
  └── dev    ← rama de desarrollo principal
       ├── feature/sesion1-bbdd
       ├── feature/sesion2-eda
       ├── feature/sesion3-mlflow
       └── feature/sesion4-powerbi
```

---

## ⚠️ Notas importantes

- **No subir** archivos de datos (`.xlsx`, `.csv`) al repositorio → penalización de -1 pt
- **No subir** contraseñas ni credenciales → penalización de -3 pt
- **No subir** la carpeta `mlruns/` → penalización de -1 pt
- Usar siempre variables de entorno para credenciales

---

## 👥 Autores

Proyecto desarrollado para el **Datathon IV** — Edición 2024/2025.
