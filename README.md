# datascience_esval

Proyecto de análisis de sensores (caudal y presión) con narrativa paso a paso:

- 00: carga desde `data/raw/DATOS SENSORES.xlsx`, QA temporal y variables, primer test |ΔP| y export canónico.
- 01: grilla 1-min, imputación controlada de gaps cortos, segmentación operacional interpretable y export.

## Requisitos
- Python 3.10+
- Instala dependencias:
  ```bash
  pip install -r requirements.txt

