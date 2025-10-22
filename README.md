# ShareVolume-78ca5b3a

A lightweight Flask app that fetches Air Products (APD) stock-ownership data, renders a clean HTML view, and can export a static version of the site. It also supports dynamic updates via a query parameter-based fetch to SEC data and a deterministic fallback for offline scenarios.

Live demo: https://pthangachi-1609.github.io/ShareVolume-78ca5b3a/

Note: This README and the code in this repo were generated with AI tooling for transparency.

---

## Overview

- The app targets Air Products (APD), with CIK 0000002969 by default.
- On first run, it attempts to read data.json. If missing, it fetches data from the SEC endpoint:
  - URL: https://data.sec.gov/api/xbrl/companyconcept/CIK0000002969/dei/EntityCommonStockSharesOutstanding.json
  - It uses a descriptive User-Agent in compliance with SEC guidance.
- The data structure saved to data.json is:
  ```
  {
    "entityName": "Air Products",
    "max": { "val": ..., "fy": ... },
    "min": { "val": ..., "fy": ... }
  }
  ```
  where max/min are the highest/lowest share counts (val) for fiscal years (fy > 2020).
- The app renders a visually appealing index.html with IDs for dynamic values:
  - share-entity-name
  - share-max-value, share-max-fy
  - share-min-value, share-min-fy
- If you open the page with a CIK parameter (e.g., index.html?CIK=0001018724), the client-side script can fetch alternate SEC data and replace the title and IDs without reloading.
- Attachments with data URIs in data.json are decoded and written to the output folder on export.

---

## Features

- Dynamic rendering at / with live entityName, max, and min values.
- Optional client-side override via CIK query parameter to fetch SEC data from a browser.
- Static export mode to generate a self-contained site (index.html, data.json, uid.txt) under output/.
- Attachments (data URIs) support: decoded and saved to output/.
- UID preservation: uid.txt is copied to the static export, if present.
- Simple, readable codebase suitable for quick customization.

---

## Prerequisites

- Python 3.8+ (recommended) or any modern Python 3.x environment.
- Optional: internet access for SEC data fetch (during initial data.json creation).
- Optional Python dependencies (see Setup below):
  - Flask
  - requests (used if available; otherwise the code falls back to deterministic data)

Notes:
- The code gracefully handles the absence of the requests package by falling back to a deterministic stash if remote fetch fails.
- The project uses a local data.json, uid.txt, and an output/ directory for static export.

---

## Setup and Installation

1. Clone the repository:
   - git clone https://github.com/<your-username>/ShareVolume-78ca5b3a.git
2. Create and activate a virtual environment (recommended):
   - python3 -m venv venv
   - source venv/bin/activate  # on macOS/Linux
   - venv\Scripts\activate     # on Windows
3. Install dependencies (Flask is required; requests is optional):
   - pip install Flask
   - Optional: pip install requests
4. Prepare the data (automatically handled by the app):
   - On first run, the app will try to load data.json. If missing, it will fetch from SEC (or fall back to deterministic data).
   - UID and attachments handling are performed during static export.

---

## How to Run

Development (dynamic, live rendering)

- Run the app:
  - python app.py
  - By default, it serves on port 5000 at 0.0.0.0, so you can access http://localhost:5000/
- Optional port:
  - python app.py --port 8000
- Access the live data page:
  - http://localhost:5000/
  - You can pass a CIK as a query param to the static fetch feature:
    - http://localhost:5000/?CIK=0001018724

Static export (static site generation)

- Generate static site:
  - python app.py --export
- After running, you’ll find the static site under the output/ directory:
  - output/index.html
  - output/data.json
  - output/uid.txt (if uid.txt existed in the repo)
  - Any decoded attachments from data.json will also be written to output/ with their original names

Notes:
- The root index.html is also created during export to mirror the dynamic page content.
- The data.json in output/ mirrors the structure described above.

---

## Data and Files

- data.json: Core data store. Created by _ensure_data_json or reset during export.
- uid.txt: Committed file; copied to static export if present.
- output/: Directory created by --export, containing:
  - index.html
  - data.json
  - uid.txt (if present)
  - Decoded attachments from data.json (data URIs decoded to files)

Data flow summary:
1. Try to read data.json.
2. If missing, attempt to fetch from SEC with a descriptive User-Agent.
3. Extract entityName and shares data; determine max/min values for fy > 2020.
4. Save the result as:
   {"entityName": "...", "max": {"val": ..., "fy": ...}, "min": {"val": ..., "fy": ...}}
5. Render index.html with the live values.
6. If --export is used, write index.html, data.json, uid.txt to output/ and create a root index.html for parity.

---

## Code Structure and How It Works (Overview)

- app.py is the single source of truth for behavior:
  - Data handling helpers:
    - _read_json, _write_json: read/write JSON with error tolerance.
    - _decode_data_uri: decode data: URIs to binary data.
    - _ensure_data_json: fetches data from SEC (if possible) or uses a deterministic fallback; writes to data.json.
    - _load_or_build_data: ensures a valid data structure is present with required keys.
    - _copy_attachments_to_output: decodes and writes data URI attachments to output/.
  - Export logic:
    - _export_site: creates output/; decodes attachments; renders index.html using the INDEX_TEMPLATE; writes output/data.json and copies uid.txt if present.
  - Web server:
    - Routes:
      - /: renders the dynamic HTML using data from _read_current_data().
      - /data.json: serves the current data as JSON.
      - /uid.txt: serves uid.txt if present.
  - UTF-8 safety:
    - _ensure_utf8_env attempts to enforce UTF-8 for prints (defensive, mostly compatible with Python 3+ environments).
  - Entry point:
    - main() parses --export and --port. In export mode, it runs _export_site() and exits. Otherwise, it boots a dev server and ensures data exists.

Data sources and behavior:
- SEC fetch:
  - URL format: https://data.sec.gov/api/xbrl/companyconcept/CIK{CIK}/dei/EntityCommonStockSharesOutstanding.json
  - A descriptive User-Agent is included (per SEC guidance) to prevent blocking.
- Fallbacks:
  - If SEC fetch fails, or requests is not available, the code falls back to deterministic values to ensure the UI still renders.
  - This makes the app robust for offline development or restricted environments.
- Query-param-based override:
  - If you open the page with ?CIK=..., the client-side JS in INDEX_TEMPLATE will attempt to fetch data from the SEC API for that CIK and update the title and the various IDs on the page without reloading.

---

## Licensing

- LICENSE: MIT License
  - Grants broad rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the software.
  - Disclaims warranty; limits liability; requires attribution in distributions.

---

## Accessibility and UX

- The HTML view uses a clean card layout with labeled rows for:
  - Max value and max fiscal year
  - Min value and min fiscal year
- The page title and header incorporate the live entityName.
- The client-side CIK fetch, when used, updates the same IDs and title for a seamless experience without a full page reload.

---

## Customization and Extensibility

- You can customize:
  - The default base CIK by changing the base_ci_k argument in _ensure_data_json.
  - The UI text by editing the INDEX_TEMPLATE.
  - The export behavior to include additional assets or extra attachments in data.json.
- If your data.json includes attachments with data URIs:
  - They’ll be decoded and saved to output/ during export so the static site remains self-contained.

---

## Security & Privacy

- The SEC fetch uses a User-Agent header to comply with SEC guidance and to avoid being blocked.
- The app does not embed external assets on the dynamic page beyond what’s rendered; the static export mirrors the data.json and uid.txt as present.
- The CIK query parameter in the browser triggers a fetch to SEC for a richer user experience, but it happens client-side in the browser and is not persisted on the server.

---

## Live Demo

- Live demo page (GitHub Pages): https://pthangachi-1609.github.io/ShareVolume-78ca5b3a/

---

## AI Generation Notice

- This README is AI-assisted. It aims to accurately reflect the codebase and usage. If you notice any discrepancies, please refer to the app.py source for exact behavior.

---

## Quick Start Commands (recap)

- Initialize and run (development):
  - python3 -m venv venv
  - source venv/bin/activate  # macOS/Linux
  - venv\Scripts\activate     # Windows
  - pip install Flask
  - python app.py
  - Visit http://localhost:5000/

- Static export:
  - python app.py --export
  - Inspect output/ for:
    - output/index.html
    - output/data.json
    - output/uid.txt (if available)

---

If you’d like adjustments or a more concise version, tell me your target audience (general developers, data scientists, or product managers) and I can tailor the README accordingly.