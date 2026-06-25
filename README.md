# VA-WHPC Survey Data Analysis

Virginia Women in High Performance Computing (VA-WHPC) event registration data analysis and visualization project. Covers 24 events from 2021 to 2026, with ~1700 registration records.

## Project Structure

```
va-whpc/
├── scripts/
│   ├── 01_data_cleaning.ipynb               # Step 1: Data cleaning pipeline
│   ├── 02_overall_visualization.ipynb       # Step 2a: Pie charts + interactive timelines
│   └── 03_grouped_visualization.ipynb       # Step 2b: Bar charts by year/group
│
├── data/
│   ├── cleaned/                             # All cleaned CSVs (24 files): ORG + Gender + Role
│   ├── 2021/ ~ 2026/                       # Cleaned CSVs grouped by year
│   ├── 2025_2026/                           # Combined 2025-2026 view
│   ├── 2024mid_2026/                        # Combined mid-2024 to 2026 view
│   └── StudentLightningTalks/               # Student Lightning Talks across years
│
├── output/
│   ├── overall/                             # Charts from 02_overall_visualization (HTML/SVG/PNG)
│   └── bygroup/                             # Charts from 03_grouped_visualization (PNG by year)
│       ├── 2021/
│       ├── 2022/
│       ├── ...
│       └── 2021-2026/
│
├── mailing list/                            # Mailing list export results
├── icon.png                                 # VA-WHPC logo (used as watermark in charts)
├── archive/                                 # Old notebook versions (for reference only)
├── requirements.txt
└── .gitignore
```

## How to Run

### Option A: HPC (Rivanna)

1. Create and activate the conda environment:
   ```bash
   conda create -n whpc python=3.11 jupyterlab notebook pandas matplotlib seaborn plotly ipywidgets -y
   pip install kaleido anywidget
   conda activate whpc
   ```
2. Register the Jupyter kernel:
   ```bash
   python -m ipykernel install --user --name whpc --display-name "whpc"
   ```
3. Launch Jupyter and select the `whpc` kernel:
   ```bash
   cd /path/to/va-whpc
   jupyter notebook
   ```

### Option B: Google Colab

1. Upload this folder to Google Drive at: `My Drive/WHPC/`
2. Open the notebook in Colab and uncomment `drive.mount()` in the first cell

### Step 1: Data Cleaning (`scripts/01_data_cleaning.ipynb`)

Run this notebook when you need to process new raw survey data.

**Pipeline:**

```
New CSV (raw export)
  │
  ├─ ① Clean ORG names (standardize abbreviations: UVA → University of Virginia)
  ├─ ② Cross-fill missing ORG using data/cleaned/ as reference
  ├─ ③ Fill ORG by email domain (virginia.edu → University of Virginia)
  │        │
  │        ▼
  │   Manually add cleaned file to data/cleaned/
  │        │
  ├─ ④ Unify ORG name variants (fix typos, merge duplicates)
  ├─ ⑤ Backfill Gender & Role from other events (same person)
  └─ ⑥ Infer Role from Job Title (e.g., "PhD Candidate" → Student)
        │
        ▼
  data/cleaned/  (final output)
```

**Key utilities in this notebook:**
- Search a person's records across all events by name
- Search all records from a specific organization
- View all unique organization names

### Step 2a: Overall Visualization (`scripts/02_overall_visualization.ipynb`)

Reads all cleaned CSVs from `data/cleaned/` and generates:
- **Pie charts**: Organization distribution per event, overall distribution
- **Interactive Plotly timelines**: Participation trends by organization and category
- **Participant analysis**: Attendance frequency, retention/drop-off analysis
- **Mailing list**: Extract people who opted in

Output saved to `output/overall/`.

### Step 2b: Grouped Visualization (`scripts/03_grouped_visualization.ipynb`)

Reads CSVs from year subfolders under `data/` and generates bar charts per year:

| Chart | Description |
|---|---|
| Event Counts | Registrant counts per event (chronological) |
| Gender Distribution | Unique / total registrations by gender |
| Role Distribution | Unique / total registrations by role |
| Organization (Detailed) | Top 8 orgs + grouped remainder |
| Organization (Macro) | Broad categories: Academia, Industry, Government, etc. |
| Female Event Counts | Female registrations per event |
| Stacked Gender | Female / Male / Choose-not-to-answer per event |
| Female by Role | Female registrations broken down by role |

Charts are generated in both **Unique** (deduplicated by person, `_Unique.png`) and **Total** (all registrations, `_Total_Registrations.png`) modes. Output saved to `output/bygroup/{year}/`.

## Adding New Event Data

When a new event registration CSV arrives:

1. **Rename** the file as `MMDDYYYY-Event Name.csv` (e.g., `03182026-AI in HPC Panel.csv`)
2. **Run `scripts/01_data_cleaning.ipynb`** — set the new file as `source_folder`, use `data/cleaned/` as `history_path`
   - This cleans ORG names, cross-fills missing ORG from historical data, and fills by email domain
   - Review the ORG column manually — if any organization name is non-standard or misspelled, correct it in the CSV and add the mapping to the `official_map` / `unification_map` dictionaries in the corresponding cells
3. **Re-run** the unification + Gender/Role backfill + Role inference steps
4. **Copy** the final CSV into `data/cleaned/`
5. **Copy** the final CSV to the appropriate `data/{year}/` folder
6. **Run `scripts/02_overall_visualization.ipynb`** to update overall charts
7. **Run `scripts/03_grouped_visualization.ipynb`** to update grouped charts
8. If the new CSV has organizations not seen before, add them to `ORG_CATEGORY_MAPPING` in the visualization notebooks

## Data Notes

- **CSV format**: Files exported after 2025-06-24 have 5 header rows to skip (handled automatically by `01_data_cleaning.ipynb`)
- **ORG mapping**: Organization names are standardized via `ORG_CATEGORY_MAPPING` dictionaries in the notebooks. New organizations must be added manually
- **PII**: CSV files contain personal information (names, emails). Do not publish to public repositories
