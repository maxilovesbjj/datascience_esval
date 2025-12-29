# Análisis Forense de Falla en Tubería HDPE (Ruta E-89)

**Proyecto:** Ciencia de Datos aplicada a Ingeniería Forense
**Estado:** Completado (Fase de Definición CFD)
**Fecha:** Diciembre 2025

## Descripción del Proyecto

Este repositorio contiene el flujo de trabajo completo para investigar la causa raíz de una falla catastrófica en una tubería de HDPE con 11 años de operación. Utilizando un año de telemetría SCADA (muestreo de 1 minuto), se reconstruyó la historia hidráulica del sistema mediante simulación estocástica y modelos de daño acumulado.

El objetivo final fue determinar si la falla se debió a operación normal, eventos extremos aislados o fatiga acumulada, y generar las condiciones de borde precisas para una validación mediante Dinámica de Fluidos Computacional (CFD).

## Metodología y Flujo de Trabajo

El análisis se estructura en seis etapas secuenciales documentadas en Jupyter Notebooks. Cada etapa consume los datos procesados de la anterior, asegurando la trazabilidad de la evidencia.

### 00_auditoria_datos
* **Fase:** Auditoría y Preprocesamiento
* **Alcance:** Carga de datos crudos, limpieza de valores atípicos físicos (outliers), sincronización temporal de sensores y tratamiento de brechas de información (gaps). Generación de la Serie Maestra de datos validados.

### 01_fisica_transientes
* **Fase:** Detección y Clasificación
* **Alcance:** Identificación estadística de anomalías hidráulicas basadas en gradientes de caudal y presión. Implementación de una taxonomía de eventos para clasificar las maniobras en tres categorías críticas:
    * **FAST_STOP:** Cierres rápidos de válvula o paradas de bomba (Riesgo de Ariete).
    * **FAST_START:** Arranques bruscos de bomba (Riesgo de Onda de Depresión).
    * **LOW_PRESSURE:** Eventos de separación de columna y cavitación (Riesgo de Colapso).

### 02_simulacion_monte_carlo
* **Fase:** Reconstrucción Física Estocástica
* **Alcance:** Dado que la frecuencia de muestreo del SCADA (1 min) no captura los transientes de alta frecuencia, se utilizó una simulación de Monte Carlo (5,000 iteraciones por evento) para reconstruir probabilísticamente los picos de presión máxima (P_max) y mínima (P_min). Se utilizaron las ecuaciones de Joukowsky y modelos físicos de colapso de cavidad.

### 02b_visualizacion_forense
* **Fase:** Diagnóstico Visual
* **Alcance:** Generación de mapas de calor de fatiga y curvas de excedencia. Identificación de la "Zona de Fatiga", correlacionando eventos de vacío profundo con rebotes de alta presión. Confirmación visual de la hipótesis de daño por ciclos de histéresis no controlados.

### 03_dano_acumulado_miner
* **Fase:** Evaluación de Vida Útil
* **Alcance:** Cálculo del Índice de Daño Relativo acumulado utilizando la Regla de Palmgren-Miner modificada para materiales viscoelásticos (exponente m=4). Proyección de la carga histórica de fatiga a los 11 años de operación de la tubería para determinar el agotamiento estructural.

### 04_seleccion_escenarios_cfd
* **Fase:** Ingeniería de Detalle
* **Alcance:** Selección determinista de los tres escenarios más representativos ("Killer Events") basada en el daño acumulado y la severidad instantánea. Exportación de las condiciones de borde reales (Presión Inicial, Caudal Inicial, Delta Q) para alimentar simulaciones de Dinámica de Fluidos Computacional (CFD) y Análisis de Elementos Finitos (FEA).

## Hallazgos Principales

1.  **Mecanismo de Falla:** La evidencia descarta una rotura por sobrepresión estática simple. El mecanismo dominante identificado es Fatiga de Alta Amplitud asistida por Vacío.

2.  **Impacto del Vacío:** Aunque los eventos de Separación de Columna (LOW_PRESSURE) representan una fracción minoritaria de la frecuencia operativa, son responsables de más del 90% del daño estructural acumulado debido a la severidad del colapso de cavidad.

3.  **Causa Raíz:** La falta de sistemas de control de transientes (ventosas de triple función o tanques unidireccionales) permitió la formación repetitiva de vacío, sometiendo al material a ciclos de deformación para los cuales no fue diseñado.

## Requisitos e Instalación

Este proyecto utiliza Python 3.10+. Las dependencias principales son pandas, numpy, scipy (para distribuciones LogNormales), matplotlib y seaborn.

1.  Clonar el repositorio:
    `git clone https://github.com/maxilovesbjj/datascience_esval.git`

2.  Instalar dependencias:
    `pip install -r requirements.txt`

3.  Ejecución:
    Se recomienda ejecutar los notebooks en orden numérico estricto (00 a 04) para garantizar la integridad de los datos procesados.

## Estructura del Repositorio

* **data/**: Directorios para datos crudos (raw) y procesados (processed).
* **notebooks/**: Jupyter Notebooks con el análisis secuencial.
* **src/**: Scripts de soporte y funciones auxiliares.
* **reports/**: Gráficos vectoriales y reportes generados.
* **README.md**: Documentación técnica del proyecto.

## Bibliografía y Referencias Técnicas

La metodología implementada en este estudio se sustenta en literatura estándar de ingeniería hidráulica y ciencia de materiales:

1.  **Transientes Hidráulicos & Golpe de Ariete**
    * Wylie, E. B., & Streeter, V. L. (1993). *Fluid Transients in Systems*. Prentice Hall. (Referencia base para ecuaciones de Joukowsky y métodos de características).
    * Bergant, A., Simpson, A. R., & Tijsseling, A. S. (2006). "Water hammer with column separation: A historical review". *Journal of Fluids and Structures*, 22(2), 135-171. (Base para la física de colapso de cavidad).

2.  **Fatiga en Materiales y HDPE**
    * Miner, M. A. (1945). "Cumulative damage in fatigue". *Journal of Applied Mechanics*, 12, A159-A164. (Regla de Miner para daño acumulado).
    * ISO 13479:2009. "Polyolefin pipes for the conveyance of fluids - Determination of resistance to crack propagation - Test method for slow crack growth on notched pipes". (Base para el exponente de fatiga m=4).
    * Martins, N. M., et al. (2018). "Failure analysis of HDPE pipes due to pressure transients". *Engineering Failure Analysis*.

3.  **Análisis Forense y CFD**
    * Tullis, J. P. (1989). *Hydraulics of Pipelines: Pumps, Valves, Cavitation, Transients*. Wiley-Interscience.