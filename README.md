# Ferguson Item Number Scraper

Python automation for reading manufacturer model numbers from Excel, searching Ferguson.com for the corresponding Ferguson item number, retrieving account-specific pricing, and writing results back to Excel.

## Current Working Baseline

This project is currently considered working under the following baseline:

- manual Ferguson login using a dedicated persistent Chrome profile
- authenticated session reuse when possible
- Ferguson item lookup from the real search results page HTML
- pricing lookup through Ferguson's `Search-GetTilePricing` endpoint
- Excel read/write using `openpyxl`
- debug artifacts written into a `debugging` subfolder

## Canonical Files

- `Ferguson Item Number Scraper.py`
- `Ferguson Item Number Scraper Template.xlsx`
- `ARCHITECTURE.md`
- `DEBUGGING.md`
- `CHANGELOG.md`
- `PRD.md`
- `PROMPT.md`
- `requirements.txt`

## Canonical Workbook Contract

- **Workbook name:** `Ferguson Item Number Scraper Template.xlsx`
- **Worksheet name:** `Scraper Data`

## Current Script Behavior

The script:

1. opens the workbook
2. reads manufacturer model numbers from column A starting at row 2
3. launches Chrome with a dedicated profile
4. reuses an authenticated Ferguson session if available
5. otherwise prompts the user to log in manually
6. transfers browser cookies into a `requests.Session`
7. extracts runtime pricing context from the authenticated page
8. searches Ferguson using the sanitized model number first, then the original model if needed
9. parses the Ferguson item number from the search results HTML
10. requests pricing using the Ferguson item number plus runtime account context
11. writes the item number to column C and price to column E
12. saves output to `Ferguson Item Number Scraper Final.xlsx`

## Current Runtime Paths

The script expects to run from its own folder and uses these local paths relative to the script:

- input workbook
- output workbook
- dedicated Chrome profile folder
- `debugging` folder for HTML, PNG, and JSON debug artifacts

## Debugging Output

Current debug behavior:

- debug files are written to the `debugging` folder
- the script clears contents of the folder at startup without deleting the folder itself
- this content-only cleanup avoids Windows / OneDrive folder-lock issues
- only a limited number of miss pages are saved during a run

## Important Technical Notes

- **Model sanitization matters.** Curly quotes, straight quotes, non-breaking spaces, and inconsistent spacing can break Ferguson search.
- **The search page is authoritative for item mapping.** Autosuggest is not treated as the primary lookup source.
- **Pricing is separate from item lookup.** Item number extraction must succeed first.
- **Authentication must be verified by page state.** URL alone is not enough.
- **The generic parser fallback is part of the current stable baseline.** Ferguson search-result HTML varies enough that the fallback remains necessary.

## Safe Change Boundary

Changes are safest when limited to one layer at a time:

- Excel I/O
- authentication/session
- search/item extraction
- pricing/API
- documentation/debugging

Avoid broad rewrites unless each downstream layer is revalidated.

## Recommended Test Path For Changes

For any meaningful code change:

1. run a very small batch first
2. confirm authentication works
3. confirm item numbers are found correctly
4. confirm prices populate correctly
5. confirm Excel writes back correctly
6. then expand to a larger batch

## Typical Run Command

```bash
python "Ferguson Item Number Scraper.py"
```
