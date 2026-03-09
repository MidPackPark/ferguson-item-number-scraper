# Ferguson Item Number Scraper

## Overview
Ferguson Item Number Scraper is a Python automation that reads manufacturer model numbers from an Excel workbook, looks up the corresponding Ferguson proprietary item numbers on Ferguson.com, and writes those item numbers back into the workbook. The current working baseline also retrieves account-specific Ferguson pricing for matched items and writes that value back into the workbook.

This project exists to support a larger plumbing estimating workflow. The immediate purpose is to take sales-price Excel documents from manufacturer websites and enrich them with Ferguson item numbers. Those Ferguson item numbers are intended to become durable cross-reference keys for later price scraping and population of a master pricing database used for bidding plumbing projects.

## Current Status
The project is currently **working** with the following baseline:
- manual login to Ferguson in a real Chrome window
- persistent local Chrome profile reuse across runs
- authenticated HTTP requests for lookup after login
- Ferguson search-results page HTML parsing for item-number extraction
- Ferguson internal `Search-GetTilePricing` endpoint for price retrieval
- Excel writeback to the configured workbook and worksheet

The login layer is intentionally **not final**. It works today through manual login plus session reuse. Future work may redesign authentication completely, but any login rewrite must preserve or revalidate the downstream item-lookup and Excel-output behavior.

## Canonical Project Identity
- **Project name:** Ferguson Item Number Scraper
- **Canonical script:** `Ferguson Item Number Scraper.py`
- **Canonical working folder:**
  `C:\Users\rparker\OneDrive - HillGrp.com\HMS Sales - Documents\Estimating Templates\Data\Automations\Ferguson PVF Item Number Scraper`

## Canonical Workbook Assumptions
These names are now the canonical baseline unless deliberately changed:
- **Input workbook:** `Ferguson Item Scraper Template.xlsx`
- **Worksheet:** `Scraper data`

The precise column mapping should stay aligned with the production Python script. If the workbook layout changes, update the code and the documentation together.

## High-Level Workflow
1. Launch Chrome using a dedicated local profile stored beside the script.
2. Reuse an existing Ferguson login session if present.
3. If needed, prompt the user to log in manually.
4. Confirm the session is actually authenticated.
5. Copy browser cookies into a `requests.Session`.
6. Read model numbers from the Excel workbook.
7. Sanitize each model number before lookup.
8. Request the real Ferguson search results page.
9. Parse the HTML for the Ferguson item number.
10. Call Ferguson's pricing endpoint with the matched item number and runtime account context.
11. Write the Ferguson item number and price back into Excel.
12. Periodically autosave and write a final output workbook.

## Why This Architecture Was Chosen
This architecture came from repeated debugging and failed alternatives.

### What failed
- Selenium form automation for Ferguson/Salesforce login
- trusting a URL change as proof of authentication
- driving the visible site search box as the primary lookup method
- using the autosuggest endpoint as the main item-number lookup source
- stealth-style login tricks as the main approach

### What worked
- manual login in a real Chrome profile
- explicit authenticated-state verification
- authenticated HTTP requests after session establishment
- parsing the real search-results page for the Ferguson item number
- calling `Search-GetTilePricing` separately for price

## Key Confirmed Example
A confirmed regression example from discovery and testing:
- **Manufacturer model number:** `MN-ZMBBU0904`
- **Ferguson item number:** `1115021`
- **Price endpoint result:** `$14.510`

This example should remain a standard regression test whenever login, parsing, or pricing logic is changed.

## Important Project Principles
- Make **small, testable edits** before merging changes into production code.
- Treat the login/authentication layer as replaceable.
- Treat the item-lookup pipeline as validated and more stable.
- Treat the Excel input/output contract as the most protected layer.
- Keep enough debugging artifacts to diagnose misses without recreating discovery work from scratch.

## Required Python Packages
Install these packages in the Python environment used by the script:

```bash
pip install openpyxl selenium requests
```

## How to Run
Run the canonical script from Windows Command Prompt or PowerShell:

```bat
python "C:\Users\rparker\OneDrive - HillGrp.com\HMS Sales - Documents\Estimating Templates\Data\Automations\Ferguson PVF Item Number Scraper\Ferguson Item Number Scraper.py"
```

## Expected Local Files Beside the Script
The working folder should contain at least:
- `Ferguson Item Number Scraper.py`
- `Ferguson Item Scraper Template.xlsx`
- the project documentation files
- the dedicated Chrome profile folder created by the script
- optional debug output files created on misses/failures

## Output Behavior
The script should write results to a separate output workbook by default rather than overwriting the source workbook. The exact output filename may evolve, but the output should remain clearly distinct from the input workbook unless explicitly changed.

## Project Documents
This project uses the following supporting documents:
- `README.md` – quick entry point for humans and agents
- `PRD.md` – business and product requirements
- `prompt.md` – structured operating prompt for future AI agents
- `ARCHITECTURE.md` – system layers and technical design
- `DEBUGGING.md` – known failure modes and troubleshooting steps
- `CHANGELOG.md` – meaningful project history

## Future Roadmap
### Short term
- keep the working manual-login baseline stable
- harden parsing and error reporting
- maintain the workbook and output behavior

### Medium term
- refactor configuration into external settings if helpful
- improve structured logging and summary reporting
- expand price enrichment carefully

### Long term
- attempt automated login again in a controlled manner
- explore server-safe or headless session reuse
- integrate Ferguson IDs and pricing into a larger master pricing database

## Important Warning About Future Changes
A future agent or developer should feel free to redesign the **login/authentication layer**, including replacing the current manual-login approach. However, any such rewrite must preserve or explicitly revalidate:
- authenticated item lookup
- correct Ferguson item extraction
- correct pricing retrieval
- correct Excel output behavior

