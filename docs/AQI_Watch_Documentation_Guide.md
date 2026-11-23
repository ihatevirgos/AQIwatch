# AQI Watch — Documentation Guide for the Group

How to add information to the project's documentation, and where it goes.

- **GitHub** ([ihatevirgos/AQIwatch](https://github.com/ihatevirgos/AQIwatch)) holds the notebook, the Data Dictionary and this guide: anything that should have version history.
- **Google Drive** (`AQI_Watch` folder) holds the data files, figures, and the documents the group edits together (Decision Log, EDA Findings, Meeting Notes).

---

## 1. Where things live

**GitHub repo**

```
notebooks/AQI_Watch_Ortigas_EDA_Merge.ipynb
docs/AQI_Watch_Data_Dictionary.md
docs/AQI_Watch_Documentation_Guide.md
README.md
```

**Google Drive**

```
AQI_Watch/
├── data/                                      ← written by the notebook only; never edit by hand
│   ├── table1a_openaq_stations.csv
│   ├── table1_openaq_pm25_hourly_raw.csv
│   ├── table1b_openaq_neighbor_hourly_raw.csv
│   ├── table2_openmeteo_weather_hourly.csv
│   ├── table3_osm_environment.csv
│   ├── table4_time_features_hourly.csv
│   ├── data_quality_scorecard.csv
│   ├── merged_hourly_full.csv
│   ├── model_ready_hourly.csv
│   └── run_metadata.json                      ← retrieval dates, station, weather grid point
├── docs/
│   ├── Decision Log                           ← every decision and why
│   ├── EDA Findings                           ← what the results show, per data-quality dimension
│   └── Meeting Notes
├── figures/                                   ← exported charts for the report and slides
└── report/                                    ← the final paper and presentation
```

Create the Decision Log, EDA Findings and Meeting Notes as **Google Docs** in `docs/`, so everyone can edit, comment and use *Suggesting* mode. The Data Dictionary stays in GitHub because it must change together with the notebook.

---

## 2. One-time setup (each member)

1. **Get access.** The folder owner shares `AQI_Watch` with everyone as **Editor**.
2. **Add a shortcut to My Drive.** Open *Shared with me*, right-click `AQI_Watch` → *Organize → Add shortcut* → *My Drive* (top level, not inside another folder).
   This matters: the notebook looks for the data at `/content/drive/MyDrive/AQI_Watch/data`. Without the shortcut, Colab can't find the shared folder and will save into a new private folder instead.
3. **Decide who retrieves the data.** Only **one person** runs the notebook with `USE_SAVED_TABLES = False` (needs an OpenAQ key in Colab Secrets). Everyone else sets `USE_SAVED_TABLES = True`, so the group analyses the same snapshot. The APIs can return slightly different numbers on different days.

---

## 3. Working rules

| Rule | Why |
|---|---|
| Say in the group chat before editing the notebook, and again when done | Two people editing the same Colab notebook at once can overwrite each other's cells |
| Never edit files in `data/` by hand | They must always match what the notebook produces; hand edits can't be traced |
| Change thresholds only in the **Settings** cell (Part 0) | One place to look; every later step uses them |
| After any change: *Runtime → Run all*, then update the 📝 cells it affects | Numbers in the write-up must match the current run |
| Save notebook changes to GitHub: *File → Save a copy in GitHub*, clear outputs first, write a commit message saying what changed (e.g. "Gap threshold 3 h → 6 h, D21") | Every change is recorded and can be undone |
| Open the notebook from GitHub each time (*File → Open notebook → GitHub*), not from an old copy in your Drive | Otherwise you edit an outdated version |
| Every change that affects results gets a **Decision Log** entry | The report has to justify every decision |

---

## 4. Where each kind of information goes

| You want to record… | Put it in | Section |
|---|---|---|
| What a chart or check shows | The 📝 cell under it in the notebook, then **EDA Findings** | 4.1, 4.4 |
| A choice the group made (threshold, method, dropped column) | **Decision Log** + the notebook's Part 6 table | 4.2 |
| A new, renamed or removed column | **Data Dictionary** (GitHub, `docs/`) | 4.3 |
| A chart for the report or slides | `figures/` | 4.5 |
| Discussion, who does what, deadlines | **Meeting Notes** | 4.6 |

### 4.1 Filling in the 📝 cells in the notebook

Every **📝 Fill in after running** cell lists what to write. Replace the placeholder text with your findings. Double-click the cell to edit it.

Each finding has three parts: **the number, what it means, and what we do about it.** (The numbers below are made up, for illustration.)

> ❌ *Completeness is low.*
>
> ✅ *PM2.5 is present for 81.4% of hours (14,260 of 17,520). Five months fall below 75% (Nov 2024, Feb 2025, Aug 2025, Sep 2025, Nov 2025). The longest gap is 480 hours (12–31 Oct 2026), when the station stopped reporting. Gaps of ≤ 3 h cover only 1.9% of missing hours, so interpolation recovers little; the missing data is mainly long outages.*

Write your initials and the date at the end of the cell, e.g. `— JD, 2026-11-25`.

### 4.2 Adding a Decision Log entry

Add an entry whenever the group **chooses** something that changes the data or the results: a threshold, a method, a dropped column, a changed date range, a fix for an error.

**Template** (copy one row per decision):

| Field | What to write |
|---|---|
| **ID** | Next number (D21, D22, …) |
| **Date** | YYYY-MM-DD |
| **Author** | Who proposed it |
| **CRISP-DM phase** | Data Understanding / Data Preparation / Modeling / Evaluation / Deployment |
| **Decision** | One sentence: what we do |
| **Alternatives considered** | What else we could have done |
| **Justification** | Why this option; cite a number from the notebook, the proposal, or a source |
| **Evidence** | Notebook section or figure (e.g. "Part 2.2 gap table") |
| **Impact** | What changes (rows lost, classes affected, etc.) |
| **Status** | Proposed / Agreed / Replaced by D## |

**Example** (numbers are illustrative):

| ID | Date | Author | Phase | Decision | Alternatives | Justification | Evidence | Impact | Status |
|---|---|---|---|---|---|---|---|---|---|
| D13 | 2026-11-23 | Group | Data Preparation | Interpolate PM2.5 gaps of ≤ 3 hours only | Fill all gaps; fill none; fill from neighbor station | Short gaps follow the daily traffic cycle; longer ones would erase rush-hour peaks. Neighbor filling mixes locations (out of scope). | Part 2.2 gap table; Part 4.2 | 81 hours filled; 4,201 still missing | Agreed |

**Starting point:** copy the 20 decisions from the notebook's **Part 6 — Decision summary** into the Decision Log as D1–D20. New decisions continue from D21.

**Changing an earlier decision:** don't delete the old entry. Set its status to *Replaced by D##* and add a new entry explaining why it changed. The report may need to explain the change.

**Keep the notebook in step:** also add or edit the matching row in the notebook's Part 6 table, so the notebook and the log agree.

### 4.3 Updating the Data Dictionary

Update it whenever a column is **added, renamed, removed or redefined**, in any table.

For each column, record:

| Field | Example |
|---|---|
| Attribute | `rain_24h_mm` |
| Table | Merged |
| Description | Total rain over the past 24 hours (mm) |
| Unit / allowed values | mm, ≥ 0 |
| Source / how it's made | Sum of `rain_mm` over the current and previous 23 hours (needs ≥ 18 values) |
| Role | Feature / Target / Join key / Label (reporting only) / Quality check only |

When a column is removed, move it to the **Removed columns** list at the end of the Data Dictionary with the Decision Log ID that removed it. Don't delete it.

### 4.4 Writing up the EDA Findings

The EDA Findings doc becomes the *Data Understanding* section of the report. Organize it by the six data-quality dimensions, in the same order as the notebook:

```
1. Uniqueness
2. Completeness
3. Validity
4. Consistency
5. Accuracy
6. Timeliness
7. Outliers
8. Merged table (class balance, daily pattern, correlations)
9. Summary of data limitations
```

Under each heading:

1. **What was checked**: one sentence (copy from the notebook's table at the top of Part 2).
2. **Result**: the numbers, copied from the scorecard (`data/data_quality_scorecard.csv`) or the cell output.
3. **What it means and what we did**: the interpretation, plus the Decision Log ID(s).
4. **Figure**: file name from `figures/`, if one applies.

Every `FLAG` in the scorecard needs an explanation here, even if the answer is "acceptable because…".

### 4.5 Saving figures

- To save a chart, add this line before `plt.show()` in that cell:
  `plt.savefig(f"{FIG_DIR}/fig_2-2_completeness_heatmap.png", dpi=200, bbox_inches="tight")`
  (`FIG_DIR` is the `figures/` folder next to `data/`; the notebook creates it.)
- Name files `fig_<notebook section>_<topic>.png`, e.g. `fig_3-2_pm25_extremes.png`, `fig_5_class_balance.png`.
- Every figure used in the report needs a caption that says what it shows and where the data comes from, e.g. *"Figure 3. Daily mean PM2.5 at the Ortigas station, Aug 2024–Oct 2026, with extreme hours (orange). Source: OpenAQ."*

### 4.6 Meeting Notes

One heading per meeting: **date, who attended, what was decided (with Decision Log IDs), who does what by when.** Decisions are only "Agreed" once they appear in a meeting note or the group chat.

---

## 5. Writing conventions

- **Dates** as YYYY-MM-DD; **times** in Philippine time (Asia/Manila), 24-hour clock.
- **Units** always: µg/m³ for PM2.5, °C, %, km/h, mm.
- **Column names** in `code format`, exactly as in the notebook.
- **Category names** exactly as in the notebook: Good, Fair, Unhealthy, Very Unhealthy, Acutely Unhealthy, Emergency. (DENR's official name for "Unhealthy" is *Unhealthy for Sensitive Groups*; say so once in the report.)
- **Sources**: cite OpenAQ, Open-Meteo, OpenStreetMap, DENR DAO 2020-14 (AQI breakpoints) and the proposal wherever you use them.
- **Plain language.** Write for a reader who hasn't seen the notebook.

---

## 6. Checklist before a submission or checkpoint

- [ ] Notebook runs top to bottom with *Run all* and no errors
- [ ] All 📝 cells are filled in and match the current run
- [ ] Every `FLAG` in the scorecard is explained in EDA Findings
- [ ] Decision Log and notebook Part 6 agree
- [ ] Data Dictionary lists every column in `merged_hourly_full.csv`
- [ ] Figures in `figures/` match the current run and have captions
- [ ] Notebook and Data Dictionary changes are committed to GitHub

---

## 7. Where each document feeds the report (CRISP-DM)

| Report section | Comes from |
|---|---|
| Business Understanding | Proposal (objectives, problem statement) |
| Data Understanding | Data Dictionary (sources, tables), EDA Findings 1–7, figures |
| Data Preparation | Decision Log (cleaning, outliers, merge, features), notebook Part 3–4 |
| Modeling | Modeling notebook + Decision Log |
| Evaluation | Modeling notebook; results with and without event periods (`event` column) |
| Limitations | EDA Findings 9, Decision Log entries marked as limitations |
