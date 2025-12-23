# Análisis hidráulico y ML de transientes operacionales (ESVAL) — dataset 1-min

Repositorio académico con el desarrollo completo (notebooks) para:
1) explorar y depurar datos de sensores de **caudal** y **presión manométrica**,
2) derivar variables hidráulicas relevantes (velocidad, presión absoluta, screening de cavitación),
3) detectar y caracterizar **eventos operacionales** (aperturas/cierres; bruscos/suaves; cortos/largos),
4) identificar **anomalías** (Isolation Forest) y
5) entrenar modelos supervisados para **detección** y **predicción probabilística** de eventos bruscos.

> Importante: el repositorio no incluye datos crudos por confidencialidad. El pipeline asume un parquet procesado en `data/processed/` generado desde el Excel original.

---

## Contexto del problema
Se dispone de un registro temporal de presión y caudal en una tubería que sufrió una falla. El objetivo del proyecto es **extraer evidencia desde los datos** (sin asumir causa a priori), conectar los patrones observados con la **física de transientes** y cuantificar si el sistema exhibe:
- cambios bruscos compatibles con maniobras hidráulicas (aperturas/cierres),
- condiciones compatibles con cavitación (screening),
- un régimen de **ciclos repetitivos** potencialmente relevantes para fatiga,
- y si es posible **detectar/anticipar** eventos bruscos mediante ML.

---

## Datos y variables
### Sensores (serie a 1 minuto aprox.)
- Caudal: \( Q(t) \) [L/s]
- Presión manométrica (gauge): \( P_g(t) \) [bar]

### Geometría (tubería)
- Diámetro interno nominal: \( D = 315\,\mathrm{mm} = 0.315\,\mathrm{m} \)
- Área: \( A = \frac{\pi D^2}{4} \)
- Velocidad media: \( V(t) = \frac{Q(t)}{A} \) (conversión de unidades aplicada en notebooks)

### Presión absoluta (screening)
El sensor entrega presión manométrica, por tanto:
\[
P_{abs}(t) \approx P_g(t) + P_{atm}
\]
donde \(P_{atm}\) puede ajustarse por altitud (se usa un valor por defecto en los notebooks).

### Cavitación (screening termodinámico)
Condición necesaria (no suficiente) para cavitación:
\[
P_{abs}(t) < P_v(T)
\]
donde \(P_v(T)\) es la presión de vapor del agua a temperatura \(T\).

También se calcula un indicador hidráulico:
\[
\sigma_{cav} = \frac{P_{abs}-P_v(T)}{\tfrac{1}{2}\rho_f V^2}
\]
Valores bajos de \(\sigma_{cav}\) sugieren mayor susceptibilidad local (especialmente en singularidades), aunque con datos 1-min el análisis es de tipo **screening**.

---

## Notebooks (flujo de trabajo)
Los notebooks están numerados y construyen una historia reproducible:

### `notebooks/00_introduccion.ipynb`
- Carga del dataset, inspección, rango temporal, tasa de missing/ceros.
- Verificación del muestreo (aprox. 1 minuto) y primeras estadísticas.

### `notebooks/01_preparacion_y_segmentacion.ipynb`
- Construcción de grilla regular 1-min.
- Diagnóstico de gaps y preparación “clean” para análisis.
- Segmentación operacional a escala horaria (macro-regímenes).

### `notebooks/02_hidraulica_cavitacion_transientes_ml.ipynb`
- Conversión a variables hidráulicas: \(V(t)\), \(P_{abs}(t)\), margen a vapor \(P_{abs}-P_v\), \(\sigma_{cav}\).
- Screening de cavitación.
- Proxies de transientes a 1-min: \(|\Delta Q|\), \(|\Delta P|\), \(|\Delta V|\), aceleración aparente (proxy).

### `notebooks/03_aperturas_y_cierres_horizontes_4min.ipynb`
- Detección de eventos tipo **apertura** / **cierre** a partir de umbrales en \(\Delta Q\).
- Clasificación de brusquedad por tiempo de subida (bines de 4 min) y duración (bines de 4 min).
- Extracción de “horizontes” \(\{0,4,8,12,16\}\) min respecto a una línea base (baseline = 1 min previo), para comparar respuesta típica del sistema.

### `notebooks/04_interpretacion_fisica_y_iforest.ipynb`
- Interpretación física de familias de eventos (pulso, persistente, intermedio).
- Detección de anomalías con **Isolation Forest** en subconjuntos operacionales.
- Identificación de episodios más atípicos y priorización para inspección.

### `notebooks/05_ml_deteccion_y_prediccion_eventos.ipynb`
Modelos supervisados a 1-min con dos tareas:

**(A) Detección / nowcasting**  
Detectar el minuto de inicio (start) de:
- apertura (ventana ±1 min),
- apertura brusca (ventana ±1 min),
- cierre (start),
- cierre brusco (start).

**(B) Predicción / forecast probabilístico**  
Estimar:
\[
\mathbb{P}(\text{inicio brusco en } (t,t+K])\,,\quad K\in\{10,30,60\}\,\text{min}
\]

**Restricción clave (evitar leakage):**  
features en \(t\) usan solo historia \(\le t\) (rolling hacia atrás). Los labels pueden mirar futuro, pero no “contaminan” features.

**Modelos usados**
- Regresión Logística (LogReg): interpretable (coeficientes), con `Imputer(median)` + `RobustScaler`.
- HistGradientBoosting (HGB): no lineal, robusto, con `Imputer(median)`.

**Métricas**
- Por minuto: PR-AUC (Average Precision) y prevalencia.
- Por evento (operacional):
  - Detección: hit si existe alarma en \([t_0-1,\,t_0+1]\).
  - Forecast: hit si existe alarma en \([t_0-K,\,t_0-1]\), con lead time de anticipación.

**Política operativa de umbral**
- Start: umbral seleccionado sujeto a una cota de FP/día (p.ej. 12).
- Forecast: umbral seleccionado sujeto a cota de alertas/día (p.ej. 50).

### `notebooks/06_conclusiones_finales.ipynb`
- Síntesis física + estadística + ML.
- Interpretación ingenieril de patrones y límites del muestreo 1-min.
- Recomendación de modelos/umbrales según objetivo (detección vs anticipación).

---

## Resultados principales (resumen ejecutivo)
### Calidad y escala
- Registro 2025-01-01 → 2025-11-27, grilla 1-min.
- Se cuantifican gaps, missing y saturaciones; el análisis se hace sobre serie “clean”.

### Hidráulica: cavitación (screening)
- Con presión manométrica y \(P_{atm}\) por defecto, el **screening no detecta** \(P_{abs}<P_v(T)\) en el periodo analizado (a \(T\approx 20^\circ C\)).  
  → Cavitación por “baja de presión bajo vapor” no aparece en estos datos 1-min.

> Nota metodológica: con muestreo 1-min pueden no capturarse mínimos instantáneos de transientes rápidos; por eso se reporta como screening.

### Eventos operacionales (aperturas/cierres)
- Se detectan eventos mediante umbrales en \(\Delta Q\) y se caracterizan por:
  - brusquedad (bines de subida de 4 min),
  - duración (bines de 4 min),
  - respuesta típica de \(Q\) y \(P_g\) en horizontes 0–16 min.
- Se observa asimetría operacional: predominan cierres persistentes y aperturas tipo pulso (familias cuantificadas en los notebooks).

### Anomalías (Isolation Forest)
- Isolation Forest identifica un subconjunto reducido de eventos atípicos (por severidad combinada de \(\Delta Q\), \(\Delta P\), persistencia y proxies).
- Estos eventos se proponen como candidatos para revisar condiciones operacionales/maniobras/sensado.

### ML supervisado (detección/forecast)
- La detección de **cierres** y **cierres bruscos** es más consistente que la de aperturas bruscas (en este dataset y esquema de features).
- Para forecast, el desempeño crece con el horizonte \(K\) (problema menos “instantáneo”).
- Se recomienda seleccionar el modelo y umbral según objetivo operacional:
  - Minimizar falsas alarmas (bajo FP/día o alertas/día),
  - Maximizar recall por evento (capturar la mayor fracción de eventos).

---

## Limitaciones y alcance
- La serie está muestreada a ~1 minuto: transientes de segundos (golpe de ariete clásico) pueden quedar submuestreados.
- Las conclusiones hidráulicas se plantean como:
  - evidencia macroscópica (cambios de régimen, escalones/rampas),
  - screening de cavitación,
  - y cuantificación de repetitividad/ciclos a escala minuto.

---

## Reproducibilidad
Instalación:
```bash
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
