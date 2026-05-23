# Credit Card Risk — Análisis y modelado de riesgo crediticio

> **Proyecto bandera del portafolio.** Conecta la experiencia previa en banca (HSBC) con el diplomado de IA aplicado a un caso real de la industria financiera.

**Dataset:** UCI Credit Card (Kaggle) · 30 000 clientes de un banco de Taiwán.
**Variable objetivo:** `default_payment_next_month` (1 = el cliente no paga el siguiente mes).
**Pregunta de negocio:** ¿podemos identificar con anticipación qué clientes van a caer en default y segmentar perfiles de riesgo?

---

## Estado actual del proyecto (revisión 2026-05-08)

### ✅ Lo que YA está hecho en el notebook (PROYECTO1.ipynb — 94 celdas)

#### 1. Entendimiento de negocio
- Diccionario completo de variables (LIMIT_BAL, SEX, EDUCATION, MARRIAGE, PAY_X, BILL_AMTX, PAY_AMTX, default).

#### 2. Extracción de datos desde **SQL Server** (pyodbc)
- Conexión a base local con manejo manual.
- Nota interna: hay que mejorar a `try/except`, `conn.close()` y evitar `SELECT *`. *(ya identificado en el propio notebook)*

#### 3. Limpieza
- Análisis y consolidación de categorías irregulares en `EDUCATION` (0, 5, 6 → "others").
- Eliminación de duplicados.
- Eliminación de columna `ID`.

#### 4. Feature engineering
- `deb_ratio`: ratio deuda / límite — variable clave en riesgo.
- Promedio de deuda mensual.
- `pay_to_bill_ratio` (con manejo de NaN).

#### 5. Análisis Exploratorio (EDA)
- Estadísticas por SEX, distribución de la variable objetivo.
- Histogramas, boxplots, IQR para outliers.
- Matriz de correlación con heatmap.
- Pairplot de variables relevantes.

#### 6. Train/test split
- Buena práctica documentada: split antes de escalar (evita data leakage).

#### 7. Escalado
- StandardScaler y RobustScaler comparados.
- Categóricas binarias dejadas sin escalar (decisión correcta).

#### 8. Reducción de dimensionalidad
- **PCA**: 2 componentes capturan 91% de la varianza (PC1: 61%, PC2: 30%).
- **UMAP**: configuración estándar (n_neighbors=10, min_dist=0.1).

#### 9. Modelos
- **Clasificación:** Logistic Regression × 4 versiones (sin escalar / Standard / Robust / PCA).
- **Regresión:** Linear Regression para predecir `avg_bill`.
- **Clustering:** K-Means, DBSCAN, Agglomerative Clustering.

#### 10. Evaluación
- Clasificación: accuracy comparativo + ROC AUC.
- Regresión: RMSE.
- Clustering: Silhouette + Davies-Bouldin + Calinski-Harabasz.

#### 11. Interpretación de clusters
- Marco de preguntas de negocio identificadas (qué cliente, riesgo, uso bancario).

---

### 🟡 Lo que FALTA (polish para portafolio)

Estas son las mejoras que llevan el proyecto de "tarea de diplomado" a "proyecto de portafolio para reclutador":

#### A. Calidad técnica
- [ ] **Corregir el data leakage en regresión:** la variable objetivo `avg_bill` se construye desde `BILL_AMTX`, pero esas mismas variables están en X. Hay que removerlas o cambiar la pregunta de regresión. *(ya marcado en el notebook)*
- [ ] **Métricas adecuadas para clase desbalanceada:** la variable de default está alrededor de 22% / 78%. Reportar **Precision, Recall, F1, ROC-AUC y PR-AUC**, no solo accuracy.
- [ ] **Modelos más fuertes:** añadir **Random Forest** y **XGBoost** al benchmark de clasificación. Logistic regression sola no es suficiente para "vender" el proyecto.
- [ ] **Manejo de imbalance:** probar `class_weight='balanced'` y/o SMOTE, comparar.
- [ ] **Hyperparameter tuning:** GridSearchCV o RandomizedSearchCV en al menos un modelo.
- [ ] **Pipeline de sklearn** completo (limpieza → escalado → modelo) para reproducibilidad. *(la celda de pipeline está como comentario)*
- [ ] **SQL extraction** con buenas prácticas (try/except, columnas explícitas, conn.close()).

#### B. Storytelling y presentación
- [ ] **Llenar el `README.md`** con problema de negocio, dataset, 3 insights clave y conclusión empresarial *(este archivo)*.
- [ ] **Visualizaciones de negocio** (no solo técnicas):
  - ¿Qué % de cada grupo de edad incumple?
  - ¿La relación PAY_0 (último retraso) → default es lineal o brutal a partir de N meses de retraso?
  - Distribución de límite de crédito vs default.
- [ ] **Interpretación de los 4 clusters** en lenguaje de negocio (qué cliente es cada uno, cuál es el más riesgoso, recomendación al banco).
- [ ] **Dashboard en PowerBI** (carpeta `powerbi/` ya creada y vacía).

#### C. Estructura del repo (antes de subir a GitHub)
- [ ] Renombrar `notebook/PROYECTO1.ipynb` → `notebook/01_credit_risk_analysis.ipynb`.
- [ ] Crear `src/` con funciones reutilizables (preprocessing, models, metrics).
- [ ] `requirements.txt` poblado (actualmente vacío).
- [ ] No subir el archivo `risk_analytics_lab.db` (167 MB) a GitHub — añadir a `.gitignore`.
- [ ] No subir `UCI_Credit_Card.csv` (2.8 MB es OK pero mejor enlazar a Kaggle).

---

## Plan de ejecución — semana del 11 al 15 de mayo

| Día | Foco principal | Entregable |
|---|---|---|
| **Lun 11** | Refactor del notebook: separar EDA, feature engineering, modelos en notebooks distintos. Limpiar SQL extraction. | `01_eda.ipynb` y `02_feature_engineering.ipynb` |
| **Mar 12** | Corregir data leakage en regresión + añadir Random Forest y XGBoost al benchmark de clasificación. Imbalance handling. | `03_classification.ipynb` con métricas correctas |
| **Mié 13** | Hyperparameter tuning + análisis profundo del mejor modelo (feature importance, errores típicos). | Tabla comparativa final + insights |
| **Jue 14** | Cerrar regresión + clustering: interpretar los 4 clusters en lenguaje de negocio. | `04_regression.ipynb` y `05_clustering.ipynb` |
| **Vie 15** | Storytelling: redactar este README con problema, insights y conclusión. Push a GitHub. PowerBI dashboard. | Repo público + post en LinkedIn |

---

## Cómo se va a "vender" en CV / LinkedIn

> **Credit Card Default Risk — Modeling & Customer Segmentation** · Python, scikit-learn, SQL Server, PowerBI.
> Pipeline end-to-end sobre 30K clientes: extracción desde SQL, EDA, feature engineering financiero (debt ratio, pay-to-bill), comparación de Logistic Regression / Random Forest / XGBoost para predicción de default. Clustering (K-Means + DBSCAN) para segmentación de riesgo. Modelo final con ROC-AUC = X.XX. Dashboard interactivo en PowerBI.

---

## Estructura del repo (objetivo)

```
credit-card-risk/
├── README.md                          ← este archivo (storytelling de negocio)
├── notebook/
│   ├── 01_eda.ipynb
│   ├── 02_feature_engineering.ipynb
│   ├── 03_classification.ipynb
│   ├── 04_regression.ipynb
│   └── 05_clustering.ipynb
├── src/
│   ├── sql_extraction.py
│   ├── preprocessing.py
│   ├── models.py
│   └── metrics.py
├── data/
│   └── (no se sube — link a Kaggle)
├── powerbi/
│   └── credit_risk_dashboard.pbix
├── images/
│   └── (gráficas exportadas para el README)
├── requirements.txt
└── .gitignore
```
