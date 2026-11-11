# Predicción de Cancelaciones Hoteleras en Portugal y Optimización de Ingresos

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/SebaGu1n1/Cancelaciones_hotel_DS/blob/main/notebooks/cancelaciones_hoteleras.ipynb)
![Python](https://img.shields.io/badge/Python-3.10+-blue.svg)
![Scikit-Learn](https://img.shields.io/badge/Scikit--Learn-1.3+-orange.svg)
![XGBoost](https://img.shields.io/badge/XGBoost-2.0+-green.svg)
![SHAP](https://img.shields.io/badge/SHAP-Explainability-purple.svg)

## 1. Presentación del Proyecto
En hotelería, una reserva cancelada sobre la fecha casi siempre termina siendo una habitación vacía que ya no se llega a revender. El objetivo de este trabajo fue armar un modelo de clasificación que permita anticipar qué reservas tienen mayor riesgo de caerse antes del *check-in*, para poder ajustar los cupos de sobreventa (*overbooking*) y activar gestiones tempranas sobre las reservas más inestables.

* **Fuente de datos:** *Hotel Booking Demand Datasets* (Antonio, de Almeida & Nunes, 2019).
* **Datos originales:** **119.390 reservas reales** tomadas entre julio de 2015 y agosto de 2017 en dos hoteles de **Portugal**: uno urbano en Lisboa (*City Hotel*) y un resort vacacional en la costa del Algarve (*Resort Hotel*).
* **Base final de modelado:** Después de limpiar duplicados exactos, descartar registros inconsistentes (reservas con 0 huéspedes), eliminar columnas con demasiados faltantes (`company`) y quitar las variables que generaban fuga de información del futuro (`reservation_status` y `reservation_status_date`), trabajamos sobre **87.230 reservas únicas**.

## 2. Comparativa de Modelos y Validación
Armamos un `Pipeline` completo en Scikit-Learn para evitar cualquier filtrado de datos entre entrenamiento y testeo (imputación por mediana, escalado estándar, codificación *One-Hot* y variables creadas a medida como `total_stay`, `total_guests`, `is_family`, `room_type_changed` y `cancellation_rate_history`).

Resultados sobre el conjunto de prueba (**17.446 reservas**):

| Modelo | Accuracy | Precision | Recall | F1-Score | ROC-AUC |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **Dummy Classifier (Base)** | 0.7248 | 0.0000 | 0.0000 | 0.0000 | 0.5000 |
| **Logistic Regression** | 0.8020 | 0.6962 | 0.4977 | 0.5804 | 0.8536 |
| **Random Forest** | 0.8372 | 0.7828 | 0.5652 | 0.6564 | 0.9029 |
| **XGBoost (Elegido)** | **0.8439** | **0.7524** | **0.6449** | **0.6946** | **0.9077** |

* **Validación Cruzada y Ajuste de Hiperparámetros:** Con `RandomizedSearchCV` (5 folds estratificados) comprobamos que el modelo es muy estable (`ROC-AUC promedio = 0.9088`) y que al llevarlo a 200 árboles con `subsample=0.8` sube a **`0.9148` en Test**. Igual decidimos hacer la simulación económica con el modelo base (`0.9077`) para trabajar sobre un escenario conservador.
* **Qué mira el modelo (`SHAP`):** El análisis de explicabilidad muestra que comprar con mucha anticipación (`lead_time`), reservar por agencias online (`Online TA`), tarifas altas (`adr`) y tener historial de cancelaciones empujan fuerte hacia la baja. En cambio, pedir cochera, hacer solicitudes especiales o modificar la reserva son señales claras de que el huésped efectivamente va a viajar.

## 3. Ajuste del Umbral e Impacto Económico
Quedarse con el corte estándar de `0.50` no tiene mucho sentido comercial porque los errores no cuestan lo mismo:
* **No detectar una cancelación a tiempo (Falso Negativo):** habitación vacía difícil de reubicar, con una pérdida media estimada en **353,60 €** (`ADR = 104 €` × `3,4 noches`).
* **Llamar o pedir reconfirmación a alguien que sí iba a venir (Falso Positivo):** costo operativo estimado en **5,00 €**.

Al bajar el umbral de decisión a **`p = 0.36`** (asumiendo que logramos revender o retener el 70% de las bajas detectadas a tiempo), los números sobre la muestra de testeo cambian bastante:

| Escenario | Bajas Detectadas | Bajas Sorpresa (FN) | Pérdida por Hab. Vacía | Costo Gestión Alertas | Beneficio Neto Recuperado |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **1. Sin Modelo (Operación actual)** | 0 | 4.802 | 1.697.987,20 € | 0,00 € | **0,00 €** |
| **2. XGBoost Estándar (`p = 0.50`)** | 3.097 | 1.705 | 602.888,00 € | 5.095,00 € | **+761.474,44 €** |
| **3. XGBoost Calibrado (`p = 0.36`)** | **3.817** | **985** | **348.296,00 €** | **9.650,00 €** | **+935.133,84 €** |

Con este ajuste detectamos casi **8 de cada 10 cancelaciones reales (Recall: 79,49%)** y sumamos **+173.659,40 € adicionales** respecto de usar el modelo sin calibrar.

## 4. Qué pasó con las hipótesis iniciales
* **H1 — Anticipación de compra (`lead_time`) [Confirmada]:** Mientras en reservas de último momento (≤ 7 días) solo cae el **8,4%**, en las que se sacan con más de 148 días de anticipación las cancelaciones saltan al **38,2%**.
* **H2 — Tipo de depósito (`deposit_type`) [Hallazgo particular del dataset]:** Contra lo intuitivo, las tarifas *Non Refund* tuvieron un **94,7%** de cancelación. En estos dos hoteles esa cláusula se usaba defensivamente para bloqueos de agencias o grupos especulativos que terminaban cayendo por falta de pago. La recomendación operativa es pasar a señas parciales (20%–30%) con validación temprana de tarjeta en reservas con score `>= 0.36`.
* **H3 — Clientes recurrentes (`is_repeated_guest`) [Confirmada]:** Quienes ya se alojaron antes cancelan apenas el **7,7%**, frente al **28,3%** de los huéspedes nuevos.

## 5. Organización del repositorio
```text
Cancelaciones_hotel_DS/
├── data/
│   ├── raw/
│   │   └── hotel_bookings.csv            # Dataset original crudo (119.390 filas)
│   └── processed/
│       └── hotel_bookings_cleaned.csv    # Dataset limpio sin duplicados ni leakage (87.230 filas)
├── models/                               # Carpeta para modelos serializados
├── notebooks/
│   └── cancelaciones_hoteleras.ipynb     # Desarrollo completo paso a paso
├── README.md                             # Resumen del proyecto
└── requirements.txt                      # Librerías utilizadas
├── notebooks/
│   └── cancelaciones_hoteleras.ipynb     # Notebook completo (EDA, Pipeline, ML y Simulación ROI)
├── README.md                             # Resumen ejecutivo del proyecto
└── requirements.txt                      # Dependencias del entorno

```

## 6. Cómo correr el notebook
* **Directo en Google Colab (más práctico):** Hacé clic en el botón **Open in Colab** arriba de todo y seleccioná `Entorno de ejecución -> Ejecutar todas`. El notebook levanta el CSV crudo directamente desde este repositorio, así que corre de punta a punta sin tener que montar Google Drive ni subir archivos a mano.
