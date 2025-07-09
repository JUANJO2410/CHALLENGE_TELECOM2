# Predicción de Churn en **TelecomX** 📉🔌

**Autor:** Juan José Ramos  
**Fecha:** julio 2025  

---

## 🎯 Objetivo
Construir un pipeline de ciencia de datos — desde la ingesta del JSON original hasta la evaluación de tres modelos — para predecir qué clientes tienen mayor probabilidad de cancelar sus servicios.

---

## 1. Fuente de datos

| Archivo | Descripción |
|---------|-------------|
| `TelecomX_Data.json` | Dataset crudo publicado por Alura (≈ 7 K registros). |
| `telecomx_flat_renombrado.parquet` | *Snapshot* limpio y aplanado (7 043 filas, 25 columnas), listo para modelado. |

---

## 2. Pipeline de preparación

1. **Aplanado del JSON**  
   Se explotan listas y se normalizan diccionarios anidados.

2. **Limpieza e imputación**  
   `CARGO_MENSUAL` y `CARGO_TOTAL` se convierten a `float`; nulos imputados con la **mediana**.

3. **Renombrado en español**  
   `customer_gender → GENERO_CLIENTE`, `account_Contract → CONTRATO`, etc.

4. **Binarización**  
   - Todas las columnas `Yes/No` → `1/0` (incluye *“No phone service”*, *“No internet service”*).  
   - Género: `Female → 0`, `Male → 1`.

5. **Eliminación de identificador**  
   `ID_CLIENTE` removido para evitar fugas de información.

6. **Snapshot Parquet**  
   Se genera **siempre**: `telecomx_flat_renombrado.parquet`.

---

## 3. Ingeniería de variables y modelos

| Paso | Implementación |
|------|----------------|
| Preprocesamiento | `ColumnTransformer`: <br>• `StandardScaler` en numéricas <br>• `OneHotEncoder(drop="first")` en categóricas |
| Modelos | Logistic Regression (`max_iter=1000`), Random Forest (`n_estimators=300`), XGBoost (`n_estimators=300`, `lr=0.05`, `max_depth=4`) |
| División | 70 % entrenamiento / 30 % prueba, estratificada |

---

## 4. Resultados (set de prueba)

| Métrica | **LogReg** | **Random Forest** | **XGBoost** |
|---------|-----------:|------------------:|------------:|
| **Accuracy** | **0.80** | 0.79 | **0.80** |
| **ROC‑AUC**  | **0.84** | 0.82 | **0.84** |
| **Recall** (clase 1) | **0.60** | 0.50 | 0.55 |
| **Precision** (clase 1) | 0.66 | 0.62 | **0.64** |
| **F1‑Score** (clase 1) | **0.62** | 0.55 | 0.59 |

> Distribución real: **73 % activos · 27 % cancelados**.

### Curvas ROC

| LogReg | Random Forest | XGBoost |
|:---:|:---:|:---:|
| ![ROC LogReg](./img/roc_logreg.png) | ![ROC RF](./img/roc_rf.png) | ![ROC XGB](./img/roc_xgb.png) |

---

## 5. Análisis comparativo

| Aspecto | LogReg | RF | XGB |
|---------|--------|----|-----|
| **AUC** | 🟢 | 🟠 | 🟢 |
| **Cobertura (recall)** | 🟢 60 % | 🔴 50 % | 🟠 55 % |
| **Precisión** | 🟢 66 % | 🟠 62 % | 🟢 64 % |
| **Interpretabilidad** | 🟢 Alta | 🟠 Media | 🔴 Baja |
| **Velocidad** | 🟢 Muy rápida | 🟢 Rápida | 🟠 Media |

---

## 6. Recomendaciones

1. **Umbral según negocio**  
   Bajar threshold de LogReg a ≈ 0.35 aumenta el recall a ~75 %.

2. **Tuning de XGBoost**  
   Ajustar `scale_pos_weight` para mejorar la clase minoritaria.

3. **Ensemble**  
   Promediar probabilidades LogReg + XGB suele sumar ~2 p.p. de AUC.

4. **Despliegue piloto**  
   Gatillar alertas semanales al equipo de fidelización para ofertas proactivas.


