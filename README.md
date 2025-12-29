#  Análisis Forense de Falla en Tubería HDPE (Ruta E-89)

![Status](https://img.shields.io/badge/Estado-Completado-success?style=for-the-badge)
![Python](https://img.shields.io/badge/Python-3.10%2B-blue?style=for-the-badge&logo=python&logoColor=white)
![Focus](https://img.shields.io/badge/Enfoque-Ingeniería%20Forense-orange?style=for-the-badge)

> **Resumen Ejecutivo:** Investigación de causa raíz de una falla catastrófica en una aducción de HDPE tras 11 años de operación. Se utilizó Ciencia de Datos, Simulación Estocástica y Modelos de Fatiga para reconstruir la historia hidráulica del sistema.

---

##  Descripción del Proyecto

Este repositorio contiene el flujo de trabajo completo (*pipeline*) para procesar telemetría SCADA (1 minuto de muestreo) y transformarla en evidencia física. 

El proyecto resuelve la incertidumbre de la "ceguera temporal" del SCADA utilizando **Simulación de Monte Carlo** para reconstruir transientes hidráulicos y la **Regla de Miner** para cuantificar el daño acumulado en el material. El resultado final son las condiciones de borde precisas para una validación por Elementos Finitos / CFD.

---

##  Flujo de Trabajo (Pipeline)

El análisis se estructura en 6 etapas secuenciales. Cada notebook consume los datos procesados del anterior.

| Notebook | Fase | Descripción Técnica |
| :--- | :--- | :--- |
| **00 Auditoría** |  Limpieza | QA/QC de datos crudos (`.xlsx`). Sincronización de sensores, imputación de gaps y generación de la **Serie Maestra**. |
| **01 Detección** |  Física | Identificación de anomalías. Clasificación taxonómica: `FAST_STOP` (Ariete), `FAST_START` (Onda) y `LOW_PRESSURE` (Vacío). |
| **02 Monte Carlo** |  Simulación | Reconstrucción estocástica de $P_{max}$ y $P_{min}$ (Joukowsky Probabilístico). **5,000 iteraciones por evento**. |
| **02b Visualización** | Evidencia | Diagnóstico visual. Mapas de calor de fatiga ($P_{max}$ vs $P_{min}$) y curvas de excedencia de presión. |
| **03 Daño Acumulado** |  Vida Útil | Cálculo del Índice de Daño Relativo (Miner, $m=4$). Proyección de fatiga a 11 años de historia. |
| **04 Selección CFD** |  Ingeniería | Selección de los 3 "Killer Events" para exportar condiciones de borde ($P_0, Q_0$) a simulación **CFD**. |

---
##  Hallazgos Principales

### 1. Mecanismo de Falla Identificado
La evidencia descarta la sobrepresión estática. La falla se debe a **Fatiga de Alta Amplitud asistida por Vacío**.
> *La tubería no "explotó" por presión alta; falló por el ciclo violento de aplastamiento (vacío) y re-inflado.*

### 2. El "Asesino Silencioso"
Aunque los eventos de **Separación de Columna (`LOW_PRESSURE`)** son menos del **5%** de los eventos detectados, son responsables del **>90% del daño estructural** acumulado.

### 3. La "Zona de la Muerte"
El mapa de fatiga revela que el sistema operó repetidamente en una zona de **Vacío Profundo ($P < -5$ mca)** seguido de rebotes de presión de alta frecuencia, sometiendo al HDPE a esfuerzos de ovalización para los cuales no fue diseñado.

---

##  Instalación y Uso

Este proyecto utiliza **Python 3.10+**. Sigue estos comandos en tu terminal para replicar el entorno.

### 1. Clonar el repositorio
```bash
git clone [https://github.com/maxilovesbjj/datascience_esval.git](https://github.com/maxilovesbjj/datascience_esval.git)
cd datascience_esval