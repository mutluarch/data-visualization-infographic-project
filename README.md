# Hagia Sophia — Spatial Memory
### MBL549E Special Topics in Architectural Design Computing | Final Project
**Student:** Muhammed Furkan Mutlu | **Institution:** Istanbul Technical University | **Date:** June 2026

---

## Research Question

> **How have historical events shaped the spatial memory and emotional identity of Hagia Sophia across 1,494 years (532–2026 CE)?**

This project maps 337 historical events onto a 3D model of Hagia Sophia using a custom-built interactive infographic. Each event is positioned according to its real spatial location in the building (X, Y axes) and its date in chronological order (Z axis), encoding emotional intensity through color, impact category through shape, and historical period through network color.

---

## Dataset Description

| Field | Value |
|---|---| 
| **Source** | Manually curated from academic and historical references |
| **File** | `ayasofya_olaylar_v3_sinif.csv` |
| **Total Records** | 337 events |
| **Time Span** | 532 CE → 2025 CE (1,493 years) |
| **Columns** | 15 |
| **Unique Spatial Locations** | 75 distinct `mekan_bolumu` values |

### Column Overview

| Column | Type | Description |
|---|---|---|
| `olay_id` | string | Unique identifier (EVT001–EVT337) |
| `tarih` | string | Historical date (YYYY or YYYY-MM-DD) |
| `yil` | integer | Year as integer for sorting (532–2025) |
| `yy` | integer | Century (6–21), computed as `(yil-1)//100+1` |
| `donem` | categorical | Historical period (Bizans / Latin_Isgali / Osmanli / Cumhuriyet) |
| `olay_tipi` | string | Raw event type code |
| `etki` | categorical | Computed impact factor (Dini / Insan / Yapisal / Doga / Kulturel / Saglik / Diger) |
| `olay_adi` | string | Full event name/title |
| `mekan_bolumu` | string | Specific spatial location within or near Hagia Sophia |
| `mekan_sinif` | categorical | Spatial layer (Kubbe & Apsis / Ic Mekan / Tum Yapi / Cevre) |
| `kisaca` | string | Short event description |
| `duygu_yogunluk` | categorical | Emotional intensity label (9 levels) |
| `duygu_skor` | integer | Emotional score −5 to +5 |
| `referans_kaynak` | string | Academic source |

### Distribution by Historical Period

| Period | Events | Avg. Emotional Score |
|---|---|---|
| Byzantine (532–1453) | 132 (39.2%) | −0.77 |
| Latin Occupation (1204–1261) | 7 (2.1%) | −4.71 |
| Ottoman (1453–1923) | 124 (36.8%) | +0.06 |
| Republic (1923–2026) | 74 (22.0%) | +0.89 |

### Distribution by Spatial Classification

| Spatial Class | Events | Layer in 3D Model |
|---|---|---|
| Tüm Yapı (Whole Building) | 126 (37.4%) | Tüm_Yapı (teal) |
| Çevre (Surroundings) | 108 (32.0%) | Çevre (green) |
| İç Mekan (Interior) | 90 (26.7%) | İç_Mekan (blue) |
| Kubbe & Apsis (Dome & Apse) | 13 (3.9%) | Dome (gold) |

### Distribution by Impact Factor

| Impact Factor | Events | Visual Shape |
|---|---|---|
| Religious / Ritual | 82 (24.3%) | Mosque silhouette |
| Human / Political | 69 (20.5%) | Human figure |
| Structural / Construction | 45 (13.4%) | Building outline |
| Natural Events | 43 (12.8%) | Tree |
| Other | 68 (20.2%) | Diamond |
| Cultural / Artistic | 25 (7.4%) | Brush |
| Health / Epidemic | 5 (1.5%) | Heart |

---

## Data Quality Analyses

### 1. Data Consistency Check

**Three inconsistencies** were identified and corrected:

| Event ID | Field | Original Value | Corrected Value | Reason |
|---|---|---|---|---|
| EVT306 | `donem` | `Bizans` | `Osmanli` | Event date is 1869 — well within Ottoman period; UNESCO scientific body founding misattributed |
| EVT160 | `mekan_sinif` | `Cevre` | `Tum Yapi` | Event concerns minarets (WWII MG-08 placement) — minarets are part of the whole building, not the urban surroundings |
| EVT171 | `mekan_sinif` | `Cevre` | `Tum Yapi` | Event concerns minarets (regular ezan/adhan) — same reasoning as EVT160 |

**Z-axis consistency:** All 337 events are confirmed within their correct century band (Z-axis). Each century occupies 80 canvas units; events are sorted chronologically within each band and distributed with a 2-unit margin at each edge. Verified via Python simulation: **0 / 337 violations**.

**Spatial coordinate consistency:** All 337 events have unique XY coordinates (verified post-jitter correction). Spatial positions are derived from `mekan_bolumu` mapped to the AyasofyaLayered2.obj model geometry (75 predefined coordinate pairs + spiral distribution for generic locations).

---

### 2. Missing Data Analysis

```
Column-level missing value scan (337 records):

olay_id           → 0 missing  ✓
tarih             → 0 missing  ✓
donem             → 0 missing  ✓
olay_tipi         → 0 missing  ✓
olay_adi          → 0 missing  ✓
mekan_bolumu      → 0 missing  ✓
kisaca            → 0 missing  ✓
duygu_yogunluk    → 0 missing  ✓
duygu_skor        → 0 missing  ✓
mekan_sinif       → 0 missing  ✓
yil               → 0 missing  ✓

referans_kaynak   → partial (some events lack academic citations)
                    Not treated as missing — reflects genuine source gaps
                    for well-documented but source-unattributed events.

RESULT: 0 structurally missing values across all analytical columns.
```

**Note:** The `referans_kaynak` field has partial entries for 23 events. These are events drawn from multiple cross-referenced sources where a single citation was not applicable. They are not imputed or removed, as the event itself is historically verified.

---

### 3. Outlier Analysis

**Emotional Score Distribution (duygu_skor, scale: −5 to +5):**

```
Mean:    −0.18
Median:  −1
Std Dev:  3.70
Min:     −5
Max:     +5
Q1:      −4
Q3:      +4
IQR:      8
```

**IQR-based outlier detection:**
- Lower fence: Q1 − 1.5×IQR = −4 − 12 = **−16** (below possible range)
- Upper fence: Q3 + 1.5×IQR = +4 + 12 = **+16** (above possible range)
- **Result: 0 statistical outliers** — the scoring scale is bounded by design (−5 to +5)

**Score distribution:**
```
Positive events (score > 0):  143  (42.4%)
Negative events (score < 0):  145  (43.0%)
Mixed/Neutral (score = 0):     49  (14.5%)
```

The near-equal split between positive and negative events reflects the turbulent history of a building that served both as a symbol of glory and as a site of conquest, disaster, and ideological conflict. The slight negative tilt (mean = −0.18) is influenced by the Latin Occupation period's extreme negative scores (avg = −4.71).

---

### 4. Distribution Plots & Statistical Summary

**Emotional Score by Century (mean):**

```
 6c (532–600)   │████████████████░░░░│  +0.64   (14 events)
 7c (601–700)   │░░░░░░░░████████████│  −2.10   (11 events)
 8c (701–800)   │░░░░░░░░░░░░████████│  −2.40    (5 events)
 9c (801–900)   │░░░░░░░░████████████│  −1.93   (15 events)
10c (901–1000)  │░░░░░░░░████████████│  −2.00    (8 events)
11c (1001–1100) │░░░░░░░░████████████│  −2.14    (7 events)
12c (1101–1200) │░░░░░░░░████████████│  −2.14   (14 events)
13c (1201–1300) │░░░░░░░░░░░░░░░░████│  −4.22   (23 events)  ← Latin Occupation
14c (1301–1400) │░░░░░░░░░░░░████████│  −2.44   (18 events)
15c (1401–1500) │░░░░░░░░████████████│  −1.09   (56 events)  ← Ottoman transition
16c (1501–1600) │████████████████████│  +0.32   (22 events)
17c (1601–1700) │████████████████░░░░│  +0.14   (14 events)
18c (1701–1800) │████████████████░░░░│  +0.08   (24 events)
19c (1801–1900) │████████████████████│  +0.72   (25 events)
20c (1901–2000) │████████████████████│  +0.49   (41 events)
21c (2001–2026) │████████████████░░░░│  +0.08   (40 events)
```

**Key observation:** The 13th century shows the most negative average (−4.22), corresponding to the Crusader sack of Constantinople (1204) and the Latin Occupation. The early Byzantine and later Republican periods show the most positive scores.

**Emotional Intensity Distribution:**

| Category | Count | % |
|---|---|---|
| Çok Yüksek Negatif (−5) | 62 | 18.4% |
| Yüksek Negatif (−4) | 48 | 14.2% |
| Orta Negatif (−3) | 35 | 10.4% |
| Karışık (0/±2) | 49 | 14.5% |
| Orta Pozitif (+3) | 38 | 11.3% |
| Yüksek Pozitif (+4) | 55 | 16.3% |
| Çok Yüksek Pozitif (+5) | 50 | 14.8% |

---

### 5. Repeated Data Check

**Potential duplicates scan (by event name):**

```
"Buyuk deprem"          → 2 occurrences
"Buyuk Istanbul depremi" → 3 occurrences
```

**Verdict: Not true duplicates.** These are distinct earthquake events occurring in different years (e.g., 740 CE, 869 CE, 1509 CE earthquakes). The repetition of the generic name "Büyük deprem" is a naming convention, not a data entry error. Each record has a distinct `olay_id`, `yil`, and `tarih`. **No records removed.**

**Duplicate record check by olay_id:** No duplicate IDs found across all 337 records.

---

### 6. Bias Check

**Temporal Bias:**
- 15th century has the most events (56 / 337 = 16.6%) due to the high historical significance of the 1453 Ottoman conquest, which attracted multiple simultaneous events.
- Early centuries (8c: 5 events, 11c: 7 events) are underrepresented — reflects the historical record gap, not a curation bias.

**Spatial Bias:**
- "Tüm Yapı" (whole building) has the most events (126), which is expected: many historical events affect the entire structure without being specific to one space.
- The Dome & Apse layer has the fewest specific events (13), reflecting the difficulty of attributing precise spatial location to historical sources.

**Emotional Scoring Bias:**
- Scores were assigned by the researcher based on historical interpretation; they are inherently subjective.
- The near-equal positive/negative split (42.4% vs 43.0%) suggests reasonable balance in curation.
- Latin Occupation (avg −4.71) was scored consistently negative, which aligns with historical consensus on the destructive nature of that period.
- **No systematic positive or negative bias was detected** — the dataset includes events across the full −5 to +5 range.

**Recency Bias:**
- The 21st century has 40 events (11.9%), the highest absolute count for a sub-century period (2001–2025, only 25 years).
- This reflects the intense contemporary political debate around Hagia Sophia's status (museum → mosque conversion 2020), not a curation artifact.

---

## Project Documentation & Reflection

### Narrative

The visualization communicates: **Hagia Sophia is not just a building — it is a layered spatial memory accumulating 1,494 years of human emotion, conflict, faith, and resilience.** Each historical event has left a spatial imprint at a specific location within the building. By mapping these events in 3D space-time, the infographic makes visible the invisible: the accumulation of meaning across centuries.

### Key Message

> The most contested spaces in history carry the heaviest emotional weight. The spaces where conquests were declared, disasters struck, and rituals transformed are not neutral — they are charged with the memory of human experience.

The visualization reveals that the dome and apse area, while architecturally central, accumulates the densest concentration of events with extreme emotional valence (both −5 and +5), reflecting its role as the symbolic heart of the building.

### Intended Audience

**Primary:** Architecture students and researchers in architectural history, cultural heritage, and spatial humanities.

**Secondary:** Digital humanities scholars, historians of Byzantine and Ottoman studies, and general audiences interested in the history of Istanbul.

**Data literacy level assumed:** Moderate. Users are expected to understand 3D spatial navigation and basic data visualization principles. The filter analysis panel provides statistical summaries for less data-literate users.

### Design Decisions

| Decision | Rationale |
|---|---|
| **3D point cloud on architectural model** | Places events in their actual spatial context; avoids abstraction that would sever the spatial-temporal relationship |
| **Z-axis = time (bottom=past, top=present)** | Intuitive for architectural sections; matches the geological metaphor of strata |
| **Century bands with dashed grid lines** | Allows immediate temporal reading without overwhelming the spatial data |
| **Color = emotional intensity** | Universal mapping (green=positive, red=negative); works across cultural literacy levels |
| **Shape = impact factor** | Adds a second data dimension without adding visual clutter; shapes are legible at small sizes |
| **4-layer wireframe model** | Preserves architectural legibility while maintaining point cloud visibility; layer-based filtering allows spatial focus |
| **Network lines to period color** | Connects events across time by shared historical context without requiring text labels |
| **Filter Analysis panel** | Provides statistical context on demand; keeps the main view clean |

### Beyond the Original Narrative

Several unexpected patterns emerged:

1. **The 15th century concentration:** The density of events in the 1400s — spanning the late Byzantine decline, the Latin aftermath, and the early Ottoman consolidation — creates a visible "traffic jam" in the Z-axis around z=720–800. This is not noise; it reflects the genuine historical intensity of that period.

2. **Surroundings as spatial memory:** The Çevre (surroundings) class has 108 events — nearly as many as the interior. This suggests Hagia Sophia's spatial memory extends far beyond its walls: to the Hippodrome, the Patriarchate, Yedikule fortress, and even Nicaea (İznik). The building's influence is metropolitan, not just architectural.

3. **The Ottoman emotional complexity:** Ottoman events cluster near score 0 (average: +0.06), suggesting a period of institutional maintenance rather than dramatic transformation. This challenges the binary narrative of "Byzantine glory vs Ottoman conquest."

4. **Republican period ambiguity:** The Republic scores the highest average (+0.89), yet includes the most politically contentious event in recent memory (the 2020 mosque reconversion, scoring −5). The positive average is driven by extensive restoration work and UNESCO recognition — a tension between preservation and ideology.

5. **Health events as spatial outliers:** The 5 epidemic events (plague, pandemic) show no spatial specificity — they affect the whole building or the city. This reflects the nature of epidemics: they have no respect for spatial boundaries, unlike structural or political events.

---

## File Structure

```
repository/
├── README.md                          ← This file
├── dataset.pkl                        ← Python pickle of cleaned dataset (337 records)
├── dataDictionary.json                ← Field definitions, value descriptions, corrections
├── metadata.json                      ← Project metadata, statistics, visualization specs
├── requirements.txt                   ← Python libraries used
└── M_Furkan_Mutlu_Infographics.html ← Interactive 3D infographic (final submission)
```

---

## How to Run the Infographic

The infographic is a **self-contained single HTML file** — no server, no dependencies, no installation required.

1. Download `M_Furkan_Mutlu_Infographics.html`
2. Open in any modern browser (Chrome, Firefox, Edge, Safari)
3. Use mouse to navigate:
   - **Left drag** → Rotate
   - **Right drag** → Pan
   - **Scroll** → Zoom
   - **Click a dot** → Inspect event details
   - **Click again / empty area** → Close
4. Use **Classification Filters** (bottom-left) to filter by Period, Spatial Type, Emotion, Impact Factor, or Century
5. Press **P** to open the Admin Panel for advanced settings

---

*Hagia Sophia — Spatial Memory | MBL549E Final | M. Furkan Mutlu | ITU 2026*
