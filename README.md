# AQI Watch

**A 72-hour Air Quality Risk Levels Classifier in Ortigas Using Historical PM2.5, Weather (Temperature, Humidity, Wind speed, and Rainfall), Time-of-Day, and Urban Environmental Factors**

Predicts the DENR air quality risk category (Good, Fair, Unhealthy, Very Unhealthy, Acutely Unhealthy, Emergency) for the Ortigas monitoring station in Mandaluyong City, 24, 48 and 72 hours ahead.

Study period: 14 Aug 2024 – 26 Oct 2026, hourly (19,296 hours).

## Data sources

| Source | What | Access |
|---|---|---|
| [OpenAQ v3](https://openaq.org) | Hourly PM2.5 at the Ortigas station | API key (free) |
| [Open-Meteo archive](https://open-meteo.com) | Temperature, humidity, wind, rainfall | No key |
| [OpenStreetMap / Overpass](https://overpass-api.de) | Roads, industrial areas, fuel stations, bus stops near the station | No key |
| Generated | Hour, weekday, rush-hour period, PH holidays, season | — |

Risk categories use the DENR PM2.5 breakpoints (DAO 2020-14, 24-hour average).

## Repository layout

```
notebooks/
  AQI_Watch_Ortigas_EDA_Merge.ipynb   Data retrieval, data-quality EDA, outliers, merge (CRISP-DM: Data Understanding & Preparation)
docs/
  AQI_Watch_Data_Dictionary.md        Every table and column, with its role
  AQI_Watch_Documentation_Guide.md    How the group adds documentation
```

Data files are **not** stored here. The notebook saves them to the shared Google Drive folder `AQI_Watch/data`.

## Running the notebook

1. Open it in Colab: *File → Open notebook → GitHub*, paste this repo's URL, pick the notebook.
2. Add a shortcut to the shared `AQI_Watch` Drive folder in your *My Drive* (see the Documentation Guide, section 2).
3. Only the person retrieving data needs an OpenAQ key (Colab 🔑 Secrets → `OPENAQ_API_KEY`). Everyone else sets `USE_SAVED_TABLES = True`.
4. *Runtime → Run all*.

## Working with this repo

- Save changes back with *File → Save a copy in GitHub* and a short commit message saying what changed.
- Clear outputs before saving (*Edit → Clear all outputs*) to keep the file small and the history readable.
- Tell the group before editing the notebook; two people saving the same notebook will overwrite each other.
- Never commit API keys or data files (`.gitignore` blocks the usual ones).
