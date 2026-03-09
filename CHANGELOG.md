# Changelog

## 2026-03-09

### Confirmed working baseline
- confirmed workbook name as `Ferguson Item Number Scraper Template.xlsx`
- confirmed worksheet tab name as `Scraper Data`
- confirmed current accepted login baseline is manual Ferguson login using a dedicated persistent Chrome profile
- confirmed current item lookup baseline is Ferguson search-results-page parsing followed by pricing lookup through `Search-GetTilePricing`

### Script stability fixes
- restored missing helper `wait_for_dom_ready()`
- restored missing helper `dismiss_cookie_banner()`
- restored missing helper `sanitize_model_number()`
- preserved helper functions at top level so they are available to the authentication and lookup flow

### Parsing fixes
- restored generic fallback logic in `pick_item_from_text()`
- retained model sanitization as a required preprocessing step before Ferguson search and parsing
- kept the nearby-window parser logic in place for model-linked extraction

### Debugging changes
- moved debug artifacts into a dedicated `debugging` folder
- updated debug cleanup to clear folder contents safely instead of deleting the folder itself
- avoided Windows / OneDrive permission failures caused by deleting the entire debug folder
- kept JSON, HTML, and PNG debug outputs available for login, runtime context, and miss analysis

### Documentation alignment
- aligned documentation to the actual workbook and worksheet names
- aligned documentation to the current working login baseline
- aligned documentation to current debug-folder behavior
- aligned documentation to the restored parser fallback and helper-function requirements
