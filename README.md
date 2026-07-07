# telemetry-data-analysis
A desktop (Tkinter) application for loading, validating, browsing, filtering, and
analyzing 1553/telemetry-style bus-log CSV exports. It decodes 16-bit hex "words,"
supports bit-level queries, subsystem filtering/export, live plotting, and PDF
acceptance reports.


1. What This Tool Does (Workflow Overview)

The app follows one continuous pipeline, from raw CSV to analyzed output:

 ┌──────────────┐   ┌───────────────┐   ┌────────────────┐   ┌──────────────────┐
 │ 1. Load CSV   │→  │ 2. Normalize  │→  │ 3. Validate     │→  │ 4. Browse Table  │
 │ (file dialog) │   │  & split Data │   │ required cols   │   │ + Row Detail     │
 └──────────────┘   └───────────────┘   └────────────────┘   └──────────────────┘
                                                                        │
        ┌───────────────────────────────────────────────────────────────┘
        ▼
 ┌───────────────┐   ┌────────────────┐   ┌───────────────┐   ┌──────────────────┐
 │ 5. Filter /   │→  │ 6. Bit / Hex   │→  │ 7. Plot Words │→  │ 8. Export /       │
 │ Search        │   │  Analysis      │   │  Over Time    │   │  Generate Report  │
 └───────────────┘   └────────────────┘   └───────────────┘   └──────────────────┘

Step-by-step


Load a CSV — User clicks "Upload," a file dialog opens, DataManager.load()
reads it with pandas.
Normalize columns — Raw exports use inconsistent header names
(Time, Rx_Cmd, Tx_Cmd, …). COLUMN_ALIASES renames them to canonical
names (DateTime, Rx_cmd, Tx_cmd, …). The single Data column
(32 space-separated hex words) is split into word0 … word31.
Validate — DataManager._validate() checks that all REQUIRED_COLUMNS
(DateTime, DeltaT, BlockStatus, MessageType, Bus, Rx_cmd, Tx_cmd, Tx_status, Rx_status) are present and the file isn't empty. Invalid files raise a
ValueError shown to the user in a message box — nothing loads silently.
Browse — Valid data populates the TableFrame (a tksheet.Sheet grid).
Selecting any row populates the DetailPanel on the right with a
field : value breakdown of that row.
Filter / Search

Tx_cmd prefix filter — highlights/filters rows whose Tx_cmd starts
with a given prefix.
Hex search — searches all word0..word31 cells for a matching hex
value and highlights matches.
Subsystem export — _export_filtered_by_subsystem() splits filtered
rows into per-subsystem folders/files (SubSys_comWordFiltered/SS<N>/…).



Bit / Hex analysis — BitAnalysisPanel lets a user pick a word and
query individual bits or bit ranges, converting between binary, hex, octal,
and decimal (Word, query_bit, query_bits_combined).
Plot — _open_plot_window() opens a subsystem-selection dialog, then a
PlotWindow (matplotlib, embedded via FigureCanvasTkAgg) plots selected
word columns over time. A user-defined formula (safely parsed via ast,
see safe_eval_formula) can transform values before plotting (e.g. x*0.125).
Export / Report — AcceptanceEngine + _build_pdf_report() (via
reportlab) generate a formatted PDF acceptance report with tables, plots,
and pass/fail styling; _save_acceptance_output() writes it to disk and can
open the containing folder (_open_in_explorer).



2. Required CSV Structure

Minimum required columns (case-sensitive, aliases auto-corrected):

Required columnCommon raw aliasDateTimeTimeDeltaT—BlockStatus—MessageType—Bus—Rx_cmdRx_CmdTx_cmdTx_CmdTx_statusTx_StatusRx_statusRx_StatusData (optional but expected)32 space-separated hex words, e.g. 0X0 0X67bc 0Xb22e ...

See Sample_Data.csv in this repo for a working example.


3. Repository Structure (Recommended)

telemetry-csv-viewer/
├── README.md                     ← this file
├── LICENSE
├── requirements.txt
├── .gitignore
├── src/
│   └── telemetry_viewer.py       ← the main app (rename test_3.py → this)
├── sample_data/
│   └── Sample_Data.csv
├── docs/
│   └── screenshots/              ← optional UI screenshots for the README
└── tests/
    └── test_data_manager.py      ← optional unit tests for DataManager, Word, etc.


Rename test_3.py to something descriptive like telemetry_viewer.py before
committing — the file's own docstring already refers to it that way.




4. Setting Up the GitHub Repository

4.1 Create the repo

bash# On GitHub.com: create a new empty repository, e.g. "telemetry-csv-viewer"
# Then locally:
git init telemetry-csv-viewer
cd telemetry-csv-viewer

4.2 Add the project files

bashmkdir -p src sample_data docs/screenshots tests
cp /path/to/test_3.py        src/telemetry_viewer.py
cp /path/to/Sample_Data.csv  sample_data/Sample_Data.csv

4.3 Create requirements.txt

pandas
tksheet
matplotlib
reportlab


tkinter ships with the standard Python installer on Windows/macOS. On Linux,
install it via the system package manager (see §5).



4.4 Create .gitignore

__pycache__/
*.pyc
.venv/
venv/
SubSys_comWordFiltered/
*.pdf
.DS_Store

4.5 Commit and push

bashgit add .
git commit -m "Initial commit: telemetry CSV viewer"
git branch -M main
git remote add origin https://github.com/<your-org>/telemetry-csv-viewer.git
git push -u origin main


5. Instructions for Other People to Run It

Give collaborators this exact sequence:

bash# 1. Clone
git clone https://github.com/<your-org>/telemetry-csv-viewer.git
cd telemetry-csv-viewer

# 2. Create and activate a virtual environment
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate

# 3. Install dependencies
pip install -r requirements.txt

# 4. (Linux only) install Tk if missing
sudo apt-get install python3-tk    # Debian/Ubuntu
# or: sudo dnf install python3-tkinter   # Fedora

# 5. Run the app
python src/telemetry_viewer.py

Then, inside the app:


Click Upload CSV and select sample_data/Sample_Data.csv (or their own
export) to confirm the install works end-to-end.
Confirm the metadata bar shows the correct row/column counts.
Select a row and confirm the detail panel populates.
Try a Tx_cmd filter and a hex search to confirm both highlight correctly.
Open the plot window and generate a PDF report to confirm matplotlib and
reportlab are working.



6. Contribution Guidelines (suggested CONTRIBUTING.md content)


Branch per feature: git checkout -b feature/<short-name>.
Keep DataManager free of any tkinter imports (already true today) — it
should stay independently testable.
If you add new required columns, update both REQUIRED_COLUMNS and this
README's table in §2.
Run a manual smoke test against sample_data/Sample_Data.csv before opening
a PR.
Open a PR into main; include a short description and, if UI-affecting, a
screenshot dropped into docs/screenshots/.



7. Known Dependencies Recap

PackagePurposepandasCSV loading, column manipulationtksheetFast, Excel-like table widget for TkintermatplotlibEmbedded plotting (PlotWindow)reportlabPDF acceptance report generation (optional — app degrades gracefully if missing, per the try/except ImportError at the top of the script)tkinter (stdlib)
