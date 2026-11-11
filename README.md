# Predicción de Cancelaciones Hoteleras en Portugal y Optimización de Revenue Management

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/SebaGu1n1/Cancelaciones_hotel_DS/blob/main/notebooks/cancelaciones_hoteleras.ipynb)
![Python](https://img.shields.io/badge/Python-3.10+-blue.svg)
![Scikit-Learn](https://img.shields.io/badge/Scikit--Learn-1.3+-orange.svg)
![XGBoost](https://img.shields.io/badge/XGBoost-2.0+-green.svg)

## 1. Contexto y Origen del Dataset
Este proyecto desarrolla un sistema de clasificación binaria supervisada orientado a anticipar qué reservas tienen alta probabilidad de cancelarse antes del *check-in*, permitiendo optimizar cupos de sobreventa controlada (*smart overbooking*), proteger el inventario y focalizar acciones tempranas de retención.

* **Fuente de datos:** *Hotel Booking Demand Datasets* (Antonio, de Almeida & Nunes, 2019).
* **Alcance:** **119.390 reservas reales** registradas entre julio de 2015 y agosto de 2017 en dos establecimientos de **Portugal**: un hotel urbano en Lisboa (*City Hotel*) y un hotel vacacional en la región del Algarve (*Resort Hotel*).
* **Dataset procesado:** Tras auditoría de calidad, remoción de duplicados exactos, filtrado de registros inconsistentes (0 huéspedes) y exclusión preventiva de variables con fuga de datos (*Data Leakage*: `reservation_status` y `reservation_status_date`), se modeló sobre **87.230 reservas únicas**.

## 2. Desempeño Comparativo de Modelos (Conjunto de Test: 17.446 reservas)
Se construyó un `Pipeline` reproducible integrando imputación sin fuga de datos, estandarización numérica, codificación *One-Hot* e ingeniería de atributos (`total_stay`, `total_guests`, `is_family`, `room_type_changed` y `cancellation_rate_history`):

| Modelo | Accuracy | Precision | Recall | F1-Score | ROC-AUC |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **Dummy Classifier (Baseline)** | 0.7248 | 0.0000 | 0.0000 | 0.0000 | 0.5000 |
| **Logistic Regression** | 0.8020 | 0.6962 | 0.4977 | 0.5804 | 0.8536 |
| **Random Forest** | 0.8372 | 0.7828 | 0.5652 | 0.6564 | 0.9029 |
| **XGBoost (Modelo Ganador)** | **0.8439** | **0.7524** | **0.6449** | **0.6946** | **0.9077** |

## 3. Calibración de Umbral e Impacto Económico (Threshold Tuning)
En hotelería el costo del error es fuertemente asimétrico: un **Falso Negativo** implica una habitación vacía difícil de revender (pérdida promedio de **353,60 €**, considerando `ADR = 104 €` y estadía de `3,4 noches`), mientras que un **Falso Positivo** solo conlleva una gestión preventiva de bajo costo (**5,00 €**).

Al optimizar la decisión comercial desplazando el umbral de corte de `p = 0.50` a **`p = 0.36`** (tasa de recupero conservadora del 70%):

| Escenario | Bajas Detectadas | Bajas Sorpresa (FN) | Pérdida por Hab. Vacía | Costo Gestión Alertas | Beneficio Neto Recuperado |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **1. Sin Modelo (Actual)** | 0 | 4.802 | 1.697.987,20 € | 0,00 € | **0,00 €** |
| **2. XGBoost Estándar (`p = 0.50`)** | 3.097 | 1.705 | 602.888,00 € | 5.095,00 € | **+761.474,44 €** |
| **3. XGBoost Calibrado (`p = 0.36`)** | **3.817** | **985** | **348.296,00 €** | **9.650,00 €** | **+935.133,84 €** |

* **Impacto operativo:** Se anticipa casi **8 de cada 10 cancelaciones reales (Recall: 79,49%)**, reduciendo un **79,5%** las pérdidas por habitaciones vacías imprevistas frente a operar sin modelo y sumando **+173.659,40 € netos adicionales** exclusivamente gracias a la calibración del umbral.

## 4. Validación de Hipótesis de Negocio
* **H1 — Anticipación (`lead_time`) [Confirmada]:** La tasa de cancelación crece de forma sostenida desde el **8,4%** en reservas de último momento (≤ 7 días) hasta el **38,2%** en compras con más de 148 días de antelación.
* **H2 — Política de Garantía (`deposit_type`) [Confirmada con matiz operativo]:** Las tarifas *Non Refund* concentraron un **94,7%** de bajas vinculadas a bloqueos especulativos de intermediarios y fallos de cobro, motivando migrar hacia anticipos parciales (20%–30%) y preautorización temprana en reservas con probabilidad `>= 0.36`.
* **H3 — Historial y Compromiso (`is_repeated_guest`) [Confirmada]:** Los huéspedes recurrentes reducen su tasa de baja al **7,7%** (frente al **28,3%** en nuevos clientes), mientras que señales de compromiso como pedidos de cochera, solicitudes especiales y cambios de habitación lideran la retención en el modelo.

## 5. Estructura del Repositorio
```text
Cancelaciones_hotel_DS/
├── data/                  # Dataset crudo y procesado (hotel_bookings_cleaned.csv)
├── models/                # Espacio reservado para artefactos serializados
├── notebooks/
│   └── cancelaciones_hoteleras.ipynb   # Notebook completo (EDA, Pipeline, ML y ROI)
├── README.md              # Resumen ejecutivo del proyecto
└── requirements.txt       # Dependencias del entorno
