# Flood Detection con Sentinel-1 SAR — Emilia-Romagna, Maggio 2023

## Descrizione del progetto

Questo progetto implementa una pipeline completa di **flood detection** 
usando immagini radar del satellite Sentinel-1 dell'ESA per rilevare 
automaticamente le zone allagate durante l'alluvione che ha colpito 
l'Emilia-Romagna nel maggio 2023 — uno degli eventi alluvionali più 
gravi della storia recente italiana, con circa 500-600 km² di territorio 
sommerso tra le province di Bologna, Forlì-Cesena e Ravenna.

L'analisi si concentra sulla **provincia di Ravenna**, una delle aree 
più colpite, identificando circa **69 km²** di zone allagate attraverso 
tecniche di change detection su dati SAR.

---

## Perché il radar e non le fotografie satellitari?

Durante un'alluvione il cielo è coperto di nuvole — i satelliti ottici 
come Sentinel-2 non riescono a fotografare il suolo. Sentinel-1 invece 
usa onde radar che attraversano le nuvole senza problemi, funzionando 
24 ore su 24 in qualsiasi condizione meteo.

Il principio fisico è semplice: l'acqua ferma riflette le onde radar 
lontano dal satellite (basso backscatter, ~-20/-25 dB), mentre il suolo 
asciutto le diffonde in tutte le direzioni (backscatter più alto, 
~-10/-15 dB). Confrontando un'immagine pre-alluvione con una 
post-alluvione, le zone allagate emergono come aree dove il backscatter 
è diminuito significativamente.

---

## Pipeline di analisi
```
Immagini Sentinel-1 (GEE)
        ↓
Preprocessing (filtro speckle, conversione dB)
        ↓
Change detection (soglia assoluta backscatter)
        ↓
Pulizia morfologica (opening + closing)
        ↓
Logical masking (esclusione acqua permanente)
        ↓
Mappa zone allagate + statistiche
```

---

## Struttura del progetto
```
FloodingMap_project/
├── data/
│   ├── raw/              # immagini Sentinel-1 originali (non incluse)
│   ├── processed/        # immagini preprocessate (non incluse)
│   └── aoi/              # area di interesse (GeoJSON)
├── notebooks/
│   ├── 01_data_download.ipynb      # download da Google Earth Engine
│   ├── 02_preprocessing.ipynb      # filtro speckle, normalizzazione
│   └── 03_change_detection.ipynb   # analisi, classificazione, output
└── outputs/
    ├── flood_mask_final.tif         # maschera binaria georeferenziata
    ├── flood_map_emilia_romagna_2023.png   # mappa tecnica Python
    ├── flood_map_qgis_final.png     # mappa cartografica QGIS
    ├── flood_map_qgis_final.pdf     # versione stampabile
    └── flood_statistics.csv         # statistiche quantitative
```

---

## Risultati

| Parametro | Valore |
|---|---|
| Sensore | Sentinel-1 SAR (banda C) |
| Polarizzazione | VV |
| Risoluzione | 30 metri per pixel |
| Periodo pre-alluvione | Aprile 2023 (media 5 immagini) |
| Periodo post-alluvione | Maggio 2023 (media 4 immagini) |
| Soglia backscatter | -15 dB |
| Area allagata identificata | 69 km² |
| Area di interesse | Provincia di Ravenna |

![Mappa flood](outputs/flood_map_qgis_final.png)

---

## Nota metodologica

Durante l'analisi è emerso un problema di **stagionalità della 
vegetazione** — tra aprile e maggio la crescita delle piante aumenta 
il backscatter medio dell'area, mascherando il segnale dell'acqua 
nella differenza pre/post. Il problema è stato risolto adottando una 
soglia assoluta sul backscatter post-alluvione invece della 
differenza relativa, combinata con un logical masking per escludere 
l'acqua permanente (mare, laghi, canali):
```python
acqua_permanente  = pre < -15.0
flood_mask_finale = flood_mask & ~acqua_permanente
```

Questo approccio è più robusto in presenza di bias stagionali ed è 
documentato nella letteratura SAR come alternativa alla change 
detection classica.

---

## Come riprodurre l'analisi

1. Clona il repository
```bash
   git clone https://github.com/tuo-username/FloodingMap_project.git
```

2. Installa le dipendenze
```bash
   pip install -r requirements.txt
```

3. Autentica Google Earth Engine
```python
   import ee
   ee.Authenticate(auth_mode='notebook')
   ee.Initialize(project='il-tuo-progetto-gee')
```

4. Esegui i notebook in ordine
```
   01_data_download.ipynb → 02_preprocessing.ipynb → 03_change_detection.ipynb
```

---

## Stack tecnologico

![Python](https://img.shields.io/badge/Python-3.12-blue)
![GEE](https://img.shields.io/badge/Google_Earth_Engine-API-green)
![QGIS](https://img.shields.io/badge/QGIS-3.x-brightgreen)

- **Python 3.12** — linguaggio principale
- **Google Earth Engine** — accesso e download dati Sentinel-1
- **rasterio** — lettura e scrittura file GeoTIFF
- **NumPy / SciPy** — elaborazione array e morfologia binaria
- **matplotlib** — visualizzazione e mappa tecnica
- **QGIS** — layout cartografico finale

---

## Autore

**Gabriele Fraterrigo**  
Progetto sviluppato come portfolio personale per l'apprendimento 
di Python applicato al GIS e al telerilevamento satellitare.