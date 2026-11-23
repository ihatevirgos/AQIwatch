# AQI Watch — Data Dictionary (Ortigas Station, Mandaluyong)

Every table and column produced by `AQI_Watch_Ortigas_EDA_Merge.ipynb`. Update this file whenever a column is added, renamed, removed or redefined (see the Documentation Guide, section 4.3).

**Role** values: *Feature* (model input) · *Target* (what the model predicts) · *Join key* · *Label* (for reporting/evaluation only, never a model input) · *Quality check* (used in EDA/cleaning, not merged) · *Info* (identification).

---

## 1. Source tables

### Table 1a — OpenAQ stations near Ortigas
Lookup table, used once to pick the Ortigas PM2.5 sensor. File: `table1a_openaq_stations.csv`

| Attribute | Description | Role |
|---|---|---|
| `location_id` | OpenAQ's ID for the monitoring site | Info |
| `location_name` | Site name (used to find "Ortigas") | Info |
| `locality` | City/area the site is listed under | Info |
| `provider` | Who supplies the data (e.g. DENR-EMB, AirGradient); used in the accuracy check | Quality check |
| `lat`, `lon` | Station coordinates | Info |
| `sensor_id` | ID of the PM2.5 sensor at that site; used to download readings | Info |
| `units` | Unit of measurement (µg/m³); used in the consistency check | Quality check |
| `first_reading_utc`, `last_reading_utc` | Date range the sensor has data for; used in the timeliness check | Quality check |
| `distance_km` | Distance from the study point (computed in the notebook) | Info |

### Table 1 — PM2.5 readings (raw)
Source: OpenAQ v3 · hourly, **has gaps** · File: `table1_openaq_pm25_hourly_raw.csv` (saved exactly as received)

| Attribute | Description | Unit / allowed values | Role |
|---|---|---|---|
| `datetime_utc` | Start of the hour, in UTC | — | Quality check (source of `datetime_local`) |
| `datetime_local` | Same hour in Philippine time (UTC+8) | — | **Join key** |
| `pm25` | Average PM2.5 concentration for the hour | µg/m³, 0–1000 | Feature / basis of target |
| `pm25_min`, `pm25_max` | Lowest and highest raw reading within the hour | µg/m³ | Quality check |
| `observed_count` | How many raw readings went into the hourly average | count | Quality check (duplicate resolution) |
| `percent_coverage` | Share of expected readings received that hour | %, 0–100 | Quality check (< 25% → missing; see decision 5) |

### Table 1b — Nearest other PM2.5 station
Same columns as Table 1. Used **only** for the accuracy cross-check (daily correlation); never merged. File: `table1b_openaq_neighbor_hourly_raw.csv`

### Table 2 — Weather
Source: Open-Meteo historical archive · hourly, complete · File: `table2_openmeteo_weather_hourly.csv`

| Attribute | Description | Unit / allowed values | Role |
|---|---|---|---|
| `datetime_local` | Hour in Philippine time | — | **Join key** |
| `temp_c` | Air temperature 2 m above ground, at that moment | °C, 10–45 | Feature |
| `humidity_pct` | Relative humidity at 2 m, at that moment | %, 0–100 | Feature |
| `wind_speed_kmh` | Wind speed 10 m above ground, at that moment | km/h, 0–250 | Feature |
| `wind_dir_deg` | Direction the wind blows from (0° = north, 90° = east) | degrees, 0–360 | Converted to `wind_dir_sin/cos` |
| `precip_mm` | Total precipitation in the preceding hour (rain + showers) | mm, 0–200 | Quality check; **not merged** (≈ identical to `rain_mm`) |
| `rain_mm` | Rain in the preceding hour | mm, 0–200 | Feature |

### Table 3 — Urban environment
Source: OpenStreetMap (Overpass API) · one static row · File: `table3_osm_environment.csv`

| Attribute | Description | Role |
|---|---|---|
| `station_name` | Station the counts are centred on | Info (copied to every merged row) |
| `lat`, `lon` | Station coordinates | Info; not merged |
| `radius_m` | Search radius around the station (1,000 m) | Info; not merged |
| `industrial_areas` | Areas tagged as industrial land use | Feature (**constant**, see note) |
| `major_road_segments` | Segments of motorways, trunk and primary roads (EDSA, Ortigas Ave, Shaw Blvd) | Feature (**constant**) |
| `secondary_road_segments` | Segments of secondary and tertiary roads | Feature (**constant**) |
| `fuel_stations` | Gas stations | Feature (**constant**) |
| `bus_stops` | Bus stops | Feature (**constant**) |

> With one station these values are identical on every row, so they can't affect predictions and will show zero importance. Reported as site description.

### Table 4 — Time features
Source: generated in the notebook · hourly, complete · File: `table4_time_features_hourly.csv`

| Attribute | Description | Allowed values | Role |
|---|---|---|---|
| `datetime_local` | Hour in Philippine time | — | **Join key** / merge spine |
| `date` | Calendar date | — | Info (needed only for daily aggregation) |
| `hour` | Hour of day | 0–23 | Feature |
| `day_of_week` | Day of week | 0 = Monday … 6 = Sunday | Feature |
| `day_name` | Day of week as text | Monday … Sunday | Info; **not merged** (repeats `day_of_week`) |
| `is_weekend` | Saturday or Sunday | 0 / 1 | Feature |
| `month` | Month | 1–12 | Feature |
| `day_period` | `morning_rush` (6–9), `midday` (10–16), `evening_rush` (17–20), `night` | 4 categories | Feature |
| `is_holiday` | Philippine public holiday | 0 / 1 | Feature |
| `season` | `wet` (Jun–Nov) or `dry` (Dec–May) | 2 categories | Feature |

### Timing notes
- **PM2.5** at 13:00 is the average for 13:00–13:59.
- **Temperature, humidity, wind** at 13:00 are the values at that moment.
- **Rain** at 13:00 is the total from 12:00 to 13:00.
- The one-hour offset is left as is (negligible for 24–72 h forecasts) and documented.

---

## 2. Merged table

Files: `merged_hourly_full.csv` (every hour of the study period) and `model_ready_hourly.csv` (only rows with all features and all three targets present).
One row per hour. Built from Table 4 (spine) ← left join Table 1 (cleaned) ← left join Table 2 ← cross join Table 3, on `datetime_local`.

### Columns carried over from the source tables
`datetime_local`, `date`, `hour`, `day_of_week`, `is_weekend`, `month`, `day_period`, `is_holiday`, `season` (Table 4) · `pm25` (Table 1, cleaned and gap-filled) · `temp_c`, `humidity_pct`, `wind_speed_kmh`, `wind_dir_deg`, `rain_mm` (Table 2) · `station_name`, `industrial_areas`, `major_road_segments`, `secondary_road_segments`, `fuel_stations`, `bus_stops` (Table 3)

### Cleaning and outlier labels (Part 3–4)

| Attribute | Description | Values | Role |
|---|---|---|---|
| `invalid_reason` | Why the original PM2.5 value was set to missing | `out_of_range`, `outside_own_min_max`, `low_coverage`, `stuck_sensor`, or empty | Label |
| `is_extreme_iqr` | PM2.5 above Q3 + 3 × IQR of the whole study period | 0 / 1 | Label |
| `is_isolated_spike` | One-hour jump (robust z > 5) with normal hours on both sides | 0 / 1 | Label |
| `event` | Inside a known event window (`new_year_YYYY`, `haze_sep_2026`) | text or empty | Label (never a feature: events can't be known ahead) |
| `pm25_interpolated` | `pm25` was filled by interpolation (gap ≤ 3 h) | 0 / 1 | Label |

### Derived features (Part 4.3) — all use only current or past hours

| Attribute | Description | Unit | Role |
|---|---|---|---|
| `pm25_lag1` | PM2.5 one hour earlier | µg/m³ | Feature |
| `pm25_lag24` | PM2.5 24 hours earlier (same hour yesterday) | µg/m³ | Feature |
| `pm25_24h` | Average PM2.5 of the current and previous 23 hours (needs ≥ 18 valid hours) | µg/m³ | Feature; basis of the target |
| `rain_24h_mm` | Total rain over the current and previous 23 hours | mm | Feature |
| `wind_24h_kmh` | Average wind speed over the current and previous 23 hours | km/h | Feature |
| `wind_dir_sin`, `wind_dir_cos` | Wind direction as sine and cosine (so 359° and 1° are close) | −1 to 1 | Feature |

### Target columns

| Attribute | Description | Values | Role |
|---|---|---|---|
| `aqi_category` | DENR category of `pm25_24h` at this hour | Good (≤ 25), Fair (25.1–35), Unhealthy (35.1–45), Very Unhealthy (45.1–55), Acutely Unhealthy (55.1–90), Emergency (> 90) µg/m³ | Basis of targets |
| `target_24h` | `aqi_category` 24 hours later | same 6 categories | **Target** |
| `target_48h` | `aqi_category` 48 hours later | same | **Target** |
| `target_72h` | `aqi_category` 72 hours later | same | **Target** |

Breakpoints: DENR DAO 2020-14, PM2.5 24-hour average. DENR's official name for "Unhealthy" is *Unhealthy for Sensitive Groups*.

---

## 3. Other output files

| File | Contents |
|---|---|
| `data_quality_scorecard.csv` | One row per data-quality check: dimension, table, check, result, OK/FLAG |
| `run_metadata.json` | Retrieval date of each table, chosen station and sensor, neighbor station, Open-Meteo grid point |

---

## 4. Removed columns

| Attribute | Table | Removed at | Reason | Decision |
|---|---|---|---|---|
| `pm25_min`, `pm25_max`, `observed_count`, `percent_coverage` | 1 | Merge | Used only for quality checks and cleaning | D12 |
| `datetime_utc` | 1 | Merge | Replaced by `datetime_local` | D12 |
| `precip_mm` | 2 | Merge | Nearly identical to `rain_mm` | D12 |
| `day_name` | 4 | Merge | Repeats `day_of_week` | D12 |
| `lat`, `lon`, `radius_m` | 3 | Merge | Identification only | D12 |
