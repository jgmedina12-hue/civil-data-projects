# Project 01: Pedestrian crash risk on Bucaramanga's roads (design)

*Written 2026-11-09. Status: draft for your approval.*

## 1. Intent

`civil-data-projects` is a personal, public portfolio: one data analysis project per civil engineering field (transportation, structures, geotechnics, …). The goal is less the results themselves than **learning the process**: how to turn an engineering question into a measurable one, choose and check statistical methods, and judge what the data does and does not support.

Project 01 is the worked example that produces **version 1 of a reusable framework**, a playbook you'll apply to and refine in every later project.

## 2. The question

> **Which roads in Bucaramanga are riskiest for pedestrians in terms of crashes, and why?**

Broken into answerable parts:

- **Q1. Where and when** do pedestrian crashes concentrate?
- **Q2. Which intersections and main corridors rank highest**, and how stable is that ranking when the definition of "risk" changes or random variation is taken into account?
- **Q3. Which road characteristics** (lanes, widths, slope, pavement, Metrolínea, bus routes, bike lane, road class…) are associated with more pedestrian crashes on main corridors?

## 3. Goals, non-goals, anti-goals

**Goals**
- G1. Answer Q1–Q3, with uncertainty and limitations stated.
- G2. Keep a written **decision log**: every important choice, the alternatives, why, and how it was checked.
- G3. Produce **framework v1** (`framework/playbook.md`), generic enough to reuse in geotechnics or structures.
- G4. A **reproducible** public repository: someone can clone it and rebuild the results from the original sources (within what the licenses allow).

**Non-goals**
- Causal claims ("a crosswalk reduces crashes by X%"). The data is observational: we measure associations.
- Pedestrian exposure modeling (no pedestrian counts exist). It's treated as an explicit limitation.
- Dashboards, apps, or machine learning as the core method (at most a final comparison).
- The whole metro area: the crash data only covers the municipality of Bucaramanga.

**Anti-goals** (failure even if "it works")
- A polished map whose numbers don't hold up under scrutiny.
- A ranking that comes out of a black box without explaining why.
- Publishing personal data.
- Dropping data silently (the crashes that can't be located must be counted and analyzed, not hidden).

## 4. Data (verified 2026-11-09)

| ID | Source | Contents | Coverage | Role |
|---|---|---|---|---|
| **D1** | Thesis 45034 (Noesis UIS), `Apéndices.zip` → `Datos de accidentes.xlsx`, sheet `DatosProcesados` | 3,651 pedestrian crashes, 31 columns: date, time, severity (3,446 injury / 201 fatal / 4 property damage only), vehicles involved, **intersection address (100%: `tipo_via_1` + `identificador_via_1` × `tipo_via_2` + `identificador_via_2`)**, neighborhood, comuna, lat/lon (**only 183 = 5%**, all highway km-marker crashes) | 2012–2023 (2023 partial; 2020–21 COVID drop) | Primary source |
| **D2** | datos.gov.co `7cci-nqqb` "Accidentes de Tránsito ocurridos en el Municipio de Bucaramanga" (Alcaldía) | 39,193 crashes (3,813 involving pedestrians), neighborhood/comuna, no address or coordinates | 2012–2023 | Cross-check of D1 (CC BY-SA 4.0) |
| **D3** | Thesis 42458 → `BASE_BUCARAMANGA_DTB.mdb` (ArcGIS personal geodatabase), already at `C:\Users\Usuario\Tesis\datos\42458_senalizacion\extraido\` | `RedVial_Primaria_terciaria`: 1,289 segments / 105 named main roads, ~58 attributes (type, class, pavement, lanes, carriageways, slope, widths, IRI, layers, Metrolínea, bike lane, transit routes, freight…). `BASE_Nomenclatura_Vial`: 8,139 street-name lines (1,430 names, ~292 km). `BASE_Barrios` (218), `COMUNAS` (17, 2005 population), `MANZANA_DANE` (dwellings per block) | Bucaramanga | Road map, address matching, attributes |

**Coordinate system of D3:** local Transverse Mercator "BUCARAMANGA" (MAGNA). Proj string:
`+proj=tmerc +lat_0=7.07888714167 +lon_0=-73.1973432 +k=1 +x_0=1097241.305 +y_0=1274642.278 +a=6379068 +rf=298.257222101 +units=m +no_defs`
Work in these meters for distances; use EPSG:4326 only for web maps.

**Feasibility already tested:** matching addresses against `BASE_Nomenclatura_Vial` + `RedVial` (closest point between the two named streets, tolerance 60 m) locates **2,821 crashes (77%)**, plus 183 with given coordinates → **~82%**. The accuracy of that matching **has not been validated yet** (Phase 2). You'll write your own version. Once you have it, I can show you the test approach so you can compare the two.

**Access links:**
- D1: https://noesis.uis.edu.co/handle/20.500.14071/45034 → file `Apéndices.zip` (10 MB). The site shows a "Security Verification" first: move the mouse over the box.
- D2: API `https://www.datos.gov.co/resource/7cci-nqqb.json?$limit=50000` (or export CSV from the dataset page).
- D3: already downloaded and verified (MD5) in your Tesis folder.

## 5. Outputs

| # | Output | Where |
|---|---|---|
| O1 | Repository structure (§7) | `civil-data-projects/` |
| O2 | Notebooks per phase, each ending with "what I learned / what I decided" | `01-pedestrian-crash-risk/notebooks/` |
| O3 | Derived datasets: located crashes (with location method and match quality), intersections table, corridors table with attributes and counts | `01-…/data/processed/` |
| O4 | Decision log. One entry per decision: **context → options considered → choice → why → how I checked it → what would change my mind** | `01-…/decisions.md` |
| O5 | Final report: findings, maps, ranking with uncertainty, limitations, recommendations | `01-…/README.md` |
| O6 | Framework v1 (playbook) | `framework/playbook.md` |
| O7 | Data sheet: sources, licenses, what is not committed and why | `01-…/data/README.md` |

## 6. Phases

Each phase ends at a **checkpoint** (what must exist to move on). Sessions = ~1 hour.

### Phase 0: Setup *(2–3 sessions)*
- Python virtual environment, libraries, git, public GitHub repo, `.gitignore` that excludes raw data.
- **Checkpoint:** a `00_setup_check.ipynb` notebook imports every library; first commit pushed.
- **Concepts:** reproducibility, environments.

### Phase 1: Data and provenance *(5–7 sessions)*
- Get D1 (zip from Noesis), D2 (portal API) and export the D3 layers to GeoPackage (see §8, export script).
- Profile each source: missing values, duplicates, categories, date ranges.
- **Cross-check D1 vs D2** by year and comuna: does the thesis contain all of the official pedestrian crashes? Where are the differences?
- Check the license of each source → `data/README.md`.
- **Checkpoint:** profiling notebook + list of "known data problems" + data sheet.
- **Concepts:** provenance, completeness, missing data and its types.

### Phase 2: Locating crashes *(8–12 sessions, built by you)*
- Normalize addresses (types, spellings, `35a` vs `35 A`, named roads).
- Matching: find where the two streets meet; use the given coordinates for km-marker crashes.
- **Validation:** (a) does the located point fall inside the neighborhood polygon recorded for the crash? (neighborhood names in D1 and in `BASE_Barrios` will need normalizing as well); (b) manual check of ~30 random crashes on a map.
- **Bias of the unlocated ones:** compare the located rate by year, comuna and severity (**chi-square test**, your first hypothesis test). If the unlocated ones aren't random, the conclusions are qualified.
- **Checkpoint:** located-crashes table with `metodo_ubicacion` and `calidad` fields; match rate and validation result in the decision log.
- **Concepts:** measurement error, selection bias, chi-square test of independence.

### Phase 3: Exploration *(5–7 sessions)*
- Time: year (COVID drop, partial 2023), month, weekday, hour, day/night.
- Severity and vehicles involved.
- Space: maps, crashes by comuna **in counts and per 10,000 residents** (population from `COMUNAS`; note it's from 2005).
- **Checkpoint:** EDA notebook with ~8–10 figures, each with a one-sentence conclusion.
- **Concepts:** distributions, **counts vs rates**, time trends as confounders, describing vs inferring.

### Phase 4: Ranking risk *(6–9 sessions)*
- Units: **intersections** (crashes grouped within a radius you'll choose and justify) and **corridors** (`RedVial` main roads, crashes within a buffer).
- Metrics: frequency; per km (corridors); severity-weighted (choosing the weight is a judgement call to document).
- **Stability:** compare the top 10 across metrics (rank correlation); split the period into 2012–2017 vs 2018–2023 (**regression to the mean**); confidence intervals for counts (Poisson) and ranks (bootstrap).
- *Stretch:* Empirical Bayes (the standard road safety method for correcting regression to the mean).
- **Checkpoint:** ranking with intervals + list of "robust hotspots" (high under every definition).
- **Concepts:** Poisson distribution, confidence intervals, bootstrap, regression to the mean.

### Phase 5: Explaining risk *(8–12 sessions)*
- Unit: `RedVial` segments, with crashes assigned and segment length as an **offset**.
- Model: crashes ~ lanes + widths + slope + pavement + class + Metrolínea + transit routes + bike lane + …
- Start with **Poisson**, detect **overdispersion**, switch to **negative binomial**; compare with AIC.
- Diagnostics: residuals, multicollinearity (VIF), excess zeros; *stretch:* spatial autocorrelation of residuals (Moran's I).
- Interpretation: incidence rate ratios with intervals, in plain words, and **what the model can't say** (without pedestrian exposure, "busy road" and "dangerous road" get mixed up).
- *Optional:* severity model (fatal vs injury) with logistic regression.
- **Checkpoint:** model table + plain-language interpretation + limitations.
- **Concepts:** GLMs, offset, overdispersion, IRR, multicollinearity, confounding, model comparison.

### Phase 6: Judgement and write-up *(4–6 sessions)*
- Recommendations: which places to look at first and what to check on site; what further data would change the conclusions.
- Final report (O5), cleaned-up decision log, **framework v1** (O6).
- **Checkpoint:** a non-specialist understands the README; the playbook is generic (it doesn't mention crashes in its steps).

**Estimated total:** ~38–56 sessions (≈ 8–11 weeks at 1 h/day; fewer during vacations).

## 7. Repository structure

```
civil-data-projects/
├── README.md                      # what the portfolio is, list of projects
├── framework/
│   └── playbook.md                # framework v1 (grows with each project)
├── docs/
│   └── 2026-11-09-project-01-pedestrian-crash-risk-design.md
├── requirements.txt
├── .gitignore                     # .venv/, data/raw/, data/interim/, *.mdb
└── 01-pedestrian-crash-risk/
    ├── README.md                  # final report
    ├── decisions.md               # decision log
    ├── notebooks/                 # 00_setup_check, 01_profiling, 02_geocoding, …
    ├── src/                       # reusable functions (addresses, matching)
    └── data/
        ├── README.md              # data sheet
        ├── raw/                   # not committed
        ├── interim/               # not committed
        └── processed/             # only derived data without personal info, if the license allows
```

## 8. Constraints and rules

- **Personal data:** `Registro1` and `Registro_simplificado` (D3) contain names, ID numbers and addresses of property owners. **They are never exported or read.** Only the listed layers are exported.
- **Public repo:** raw data doesn't go into git; scripts download or rebuild it. Derived data is published only if the license of each source allows it (D2 is CC BY-SA: share-alike; D1 and D3 still need checking in Phase 1).
- **Windows / Access:** the only Access driver installed is 32-bit. Exporting D3 requires a one-time script run with 32-bit PowerShell (`C:\Windows\SysWOW64\WindowsPowerShell\v1.0\powershell.exe`) that writes the geometries to a text file, which Python then converts to GeoPackage. I'll provide that script at the start of Phase 1 (to `01-…/tools/`), with the personal-data tables excluded in the code itself.
- **Statistics:** each method is introduced when the problem needs it, never as a theory block up front.

## 9. Who decided what

**Decided by you:** personal project; public portfolio on GitHub; Python; learning focus (process and judgement); the question (riskiest roads and why); you write the code and I explain and review; ~1 h/day with no deadline; you build the crash-locating step; the design as played back.

**My calls (check them):**
1. The phases and their order, and the session estimates.
2. Two units of analysis (intersections + corridors) instead of one.
3. The method list per phase (chi-square, Poisson, bootstrap, negative binomial; Empirical Bayes as a stretch).
4. Data rules (§8) and the repo structure (§7).
5. **The D3 export script is provided as a utility** (it's Windows/Access plumbing, not part of what you want to learn). Address matching is yours.
6. Language: English for code and documentation, keeping the original Spanish field names.

**Left for you to decide along the way** (each one goes in the decision log; they're the judgement exercises): matching tolerance, intersection grouping radius, corridor buffer, severity weights, which variables go into the model, what counts as a "robust hotspot".

## 10. Known risks

- The license of D1 (thesis) is unverified → it determines what can be published.
- Matching accuracy is unvalidated → Phase 2 measures it.
- Road attributes describe a single moment in time; the crashes cover 12 years.
- No pedestrian exposure → every comparison is "per crash", not "per pedestrian".
- 2020–2021 (COVID) and partial 2023 distort the trends.
- **Phase 5 sample:** only the crashes that fall on the 105 main roads in `RedVial` go into the model. If there are few (to be measured in Phase 2), the model will have wide intervals or few usable variables. That's a result in itself, not a failure.

## 11. First session (Phase 0, ~1 h)

1. Open a terminal in `C:\Users\Usuario\civil-data-projects`.
2. Create the environment: `python -m venv .venv`
3. Install: `.venv\Scripts\python -m pip install pandas geopandas pyogrio shapely pyproj statsmodels scipy matplotlib jupyterlab openpyxl` *(backslashes: in Command Prompt, `/` is read as an option and the command fails)*
4. `git init`, create `.gitignore` (see §7) and `README.md` with two paragraphs on what the portfolio is. Save the versions: `.venv\Scripts\python -m pip freeze > requirements.txt`.
5. Create the public repo on GitHub and push the first commit (this document included).
6. `.venv\Scripts\python -m jupyter lab` → notebook `01-pedestrian-crash-risk/notebooks/00_setup_check.ipynb` that imports each library and prints its version.
7. First entry in `decisions.md`: why a virtual environment and those libraries.

**Checkpoint:** the notebook runs with no errors and the commit is on GitHub.
