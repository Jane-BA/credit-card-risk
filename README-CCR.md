# Credit Card Risk — Análisis y modelado de riesgo crediticio

**Pipeline end-to-end de Machine Learning** sobre 30,000 clientes de tarjeta de crédito: extracción de datos, EDA, ingeniería de características financieras, clasificación, regresión y segmentación de clientes, con interpretación de resultados orientada al negocio.

**Stack:** Python · pandas · scikit-learn · UMAP · matplotlib/seaborn · SQL Server (SQLAlchemy/pyodbc) · Power BI

---

## 🎯 Problema de negocio

¿Podemos identificar **con anticipación** qué clientes van a caer en default y segmentar los perfiles de riesgo de la cartera?

Para una institución financiera, no detectar a un cliente que incumplirá cuesta mucho más que revisar de más a uno que sí pagará. Este proyecto construye y compara modelos de scoring, y traduce los resultados en reglas de alerta temprana accionables.

**Dataset:** [Default of Credit Card Clients (UCI)](https://www.kaggle.com/datasets/uciml/default-of-credit-card-clients-dataset) — 30,000 clientes de un banco de Taiwán, 25 variables (límite de crédito, demografía, historial de pagos, facturación y pagos de 6 meses).
**Variable objetivo:** `default.payment.next.month` (1 = el cliente incumple el mes siguiente · 22.1% de la cartera).

---

## 🔍 Resumen del pipeline

1. **Extracción de datos** — descarga reproducible desde Kaggle vía `kagglehub` (con respaldo local) y bloque alternativo de extracción desde **SQL Server** con SQLAlchemy.
2. **Limpieza** — consolidación de categorías no documentadas en `EDUCATION` y `MARRIAGE`; verificación de nulos y duplicados.
3. **Feature engineering** — 5 variables financieras creadas: `deb_ratio` (utilización del crédito), `avg_bill`, `avg_pay`, `pay_to_bill_ratio` (proporción de deuda pagada) y `num_delays` (meses con retraso).
4. **EDA** — univariado, bivariado contra el target, correlaciones (alta colinealidad entre `BILL_AMT1–6`) y detección de outliers vía IQR.
5. **Preparación** — split 80/20 estratificado, escalado ajustado solo con train (StandardScaler vs RobustScaler), prevención explícita de *data leakage* en los targets de regresión.
6. **Reducción de dimensionalidad** — PCA (9 componentes ≈ 90% de varianza) y UMAP 2D sobre datos escalados para clustering.
7. **Modelado** — comparación de 3 algoritmos × 4 variantes de preprocesamiento en cada tarea:
   - *Clasificación:* Logistic Regression y Random Forest con `class_weight='balanced'`, KNN.
   - *Regresión:* Linear Regression, Random Forest, KNN sobre dos targets (`avg_bill` y `LIMIT_BAL`).
   - *Clustering:* K-Means (k=6), DBSCAN y jerárquico sobre el embedding UMAP.
8. **Interpretación de negocio** — tasas de default por segmento, perfil del cliente riesgoso y recomendación final.

---

## 📊 Resultados clave

### Clasificación (predicción de default)

| Modelo | Variante | ROC-AUC | Recall (default) | Precision (default) |
|---|---|---|---|---|
| **Random Forest** | Standard | **0.760** | 0.34 | 0.65 |
| Logistic Regression (balanced) | Standard | 0.71 | **0.62** | 0.37 |
| KNN (k=15) | Standard | 0.73 | — | — |

Dos estrategias según la prioridad del negocio: **Random Forest** como mejor motor de ranking de riesgo (candidato a ajuste de umbral), o **Regresión Logística balanceada** si se prioriza detectar al mayor número de morosos (6 de cada 10) con un modelo interpretable.

### Regresión

- La **deuda promedio** (`avg_bill`) es altamente predecible: **R² = 0.87** (Random Forest).
- El **límite de crédito** (`LIMIT_BAL`) no: **R² = 0.43** — depende de información externa al dataset (ingresos, buró), hallazgo en sí mismo relevante.

### Clustering

K-Means sobre UMAP identificó **un segmento con ~55% de tasa de default** (2.5× la tasa base del 22%), construido **sin ver la etiqueta** — evidencia de que el comportamiento financiero por sí solo define perfiles de riesgo.

---

## 💡 3 insights de negocio

1. **El primer retraso casi triplica el riesgo.** La tasa de default pasa de 11.7% (clientes sin retrasos) a 29.8% con un solo mes de retraso, y escala hasta 70.3% con seis. Es el predictor más fuerte del dataset y la base natural de un sistema de alerta temprana.

2. **Lo que importa no es cuánto debe el cliente, sino qué proporción paga.** Quienes cubren menos del 5% de su deuda promedio incumplen al 27.9%; quienes pagan el 100% o más, al 14.8%. El ratio pago/deuda duplica la discriminación de riesgo.

3. **El perfil del cliente riesgoso es medible con tres señales:** utilización del crédito al 49% del límite (vs 27% en cumplidos), pagos del 5.6% de la deuda (vs 10.9%) y al menos un retraso registrado. Las variables demográficas (sexo, educación, estado civil) aportan poco.

*(Gráficas completas en `images/` y análisis detallado en el reporte técnico.)*

---

## 🏦 Recomendación al negocio

- **Scoring:** Random Forest + StandardScaler con umbral calibrado por matriz de costos; Regresión Logística balanceada como alternativa interpretable.
- **Alerta temprana:** activar gestión preventiva (recordatorios, reestructura, congelar aumentos de línea) ante el primer mes de retraso o un ratio pago/deuda < 5%.
- **Gestión de cartera:** monitoreo intensivo del segmento de alto riesgo identificado por clustering; mayor apetito comercial en los segmentos con tasas de 12–15%.

---

## 📈 Dashboard (Power BI)

> 🚧 *En construcción — carpeta `powerbi/`.*

Dashboard interactivo con los KPIs del proyecto: tasa de default por segmento (retrasos, utilización, límite), perfil comparativo del cliente riesgoso, y vista de la segmentación de clientes con su nivel de riesgo.

---

## 📁 Estructura del repo

```
credit-card-risk/
├── README-CCR.md                              ← este archivo
├── notebook/
│   └── 01_credit_risk_analysis.ipynb      ← pipeline completo (EDA → modelos → interpretación)
├── reporte/
│   └── Reporte_credit_risk.docx           ← reporte técnico con figuras e interpretaciones
├── powerbi/
│   └── credit_risk_dashboard.pbix         ← dashboard interactivo (en construcción)
├── images/                                ← gráficas exportadas para el README
├── data/                                  
│   └── UCI_Credit_Card.csv                ← no se sube: el notebook descarga desde Kaggle
├── requirements.txt
└── .gitignore
```

---

## ▶️ Cómo reproducir

```bash
git clone https://github.com/<usuario>/credit-card-risk.git
cd credit-card-risk
pip install -r requirements.txt
jupyter notebook notebook/01_credit_risk_analysis.ipynb
```

El notebook descarga el dataset automáticamente desde Kaggle con `kagglehub` (no requiere API key). Opcionalmente, incluye un bloque comentado para extraer los datos desde una instancia local de SQL Server.

**requirements.txt:**

```
pandas
numpy
matplotlib
seaborn
scikit-learn
umap-learn
kagglehub
sqlalchemy
pyodbc
jupyter
```
