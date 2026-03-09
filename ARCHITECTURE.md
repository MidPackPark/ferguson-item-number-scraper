# Architecture

## Purpose

This document describes the current technical architecture of the Ferguson Item Number Scraper, the boundaries that should remain stable, and the parts of the system that may be redesigned in the future.

## System Design Philosophy

Treat this project as a layered system, not a single script.

The most important rule is that the **item lookup and Excel pipeline must remain stable** unless a change is explicitly intended and revalidated. Authentication may evolve, but downstream behavior must continue to work.

## Current Working Architecture

### Layer 1: Excel Input / Output

**Responsibilities**
- locate the input workbook
- open worksheet `Scraper Data`
- read manufacturer model numbers from column A
- write Ferguson item numbers to column C
- write pricing to column E
- autosave progress during the run
- save final output to a separate workbook

**Current settings in code**
- workbook: `Ferguson Item Number Scraper Template.xlsx`
- sheet: `Scraper Data`
- start row: `2`

**Stability**
- highly stable
- should not be changed casually

---

### Layer 2: Authentication / Session Establishment

**Current working baseline**
- launch Chrome with dedicated profile folder `Ferguson Item Number Scraper Profile`
- attempt to reuse existing authenticated Ferguson session
- if session is not authenticated, prompt user for manual login
- verify authentication from page-state signals
- copy cookies from Selenium browser into a `requests.Session`

**Important helper functions**
- `wait_for_dom_ready()`
- `dismiss_cookie_banner()`
- `ensure_authenticated_session()`
- `build_authenticated_session()`

**Stability**
- flexible
- may be redesigned later
- any redesign must revalidate downstream lookup and Excel output

---

### Layer 3: Search and Item Extraction

**Responsibilities**
- sanitize the manufacturer model number before searching
- request the Ferguson search results page
- parse the returned HTML for the Ferguson item number

**Current lookup strategy**
1. sanitize model number
2. search Ferguson using the sanitized model
3. if needed, try the original raw model number
4. parse the search results HTML for the Ferguson item number

**Current parser behavior**
- strong model-linked patterns first
- nearby-window search around the matched model text
- generic fallback patterns over the page HTML

**Why the generic fallback remains**
Recent debugging confirmed that some valid Ferguson result pages only resolved correctly when the generic fallback block was present. It is therefore part of the current stable baseline.

**Important helper functions**
- `sanitize_model_number()`
- `search_page_request()`
- `search_page_selenium()`
- `pick_item_from_text()`

**Stability**
- mostly stable
- changes should be small and tested on a sample batch first

---

### Layer 4: Pricing / API Layer

**Responsibilities**
- extract pricing context from the authenticated session/page
- call Ferguson `Search-GetTilePricing`
- parse and return the account-specific price

**Runtime pricing context currently required**
- `productIDs`
- `shipWhseId`
- `branchId`
- `customerId`

In the current script, these values are derived from:
- `warehouse_location_id`
- `branch_id`
- `customer_main_account_number`

**Important helper functions**
- `extract_runtime_context()`
- `extract_runtime_context_from_html()`
- `get_tile_pricing()`

**Stability**
- stable but dependent on Ferguson markup and endpoint behavior

---

### Layer 5: Debugging and Diagnostics

**Responsibilities**
- save enough information to diagnose misses and failures
- avoid overwhelming the working folder with loose debug files
- preserve a lightweight support trail for login, parsing, and runtime context issues

**Current behavior**
- debug output goes into `debugging`
- saved file types include `.json`, `.html`, and `.png`
- only a limited number of miss pages are saved per run
- startup cleanup clears the contents of `debugging` without deleting the folder itself

**Why the cleanup works this way**
Deleting the entire folder caused Windows / OneDrive permission errors when files were open, syncing, or locked. Content-only cleanup is the current safe baseline.

**Stability**
- moderately stable
- can be improved, but should remain lightweight and useful

## Current End-to-End Flow

1. start script from working folder
2. open workbook
3. gather model numbers from worksheet
4. launch Chrome with dedicated profile
5. verify existing authenticated Ferguson session or prompt for manual login
6. transfer cookies into `requests.Session`
7. extract runtime pricing context
8. sanitize model number
9. search Ferguson results page
10. parse Ferguson item number from search HTML
11. request price from `Search-GetTilePricing`
12. write item number and price back to workbook
13. autosave periodically
14. save final workbook
15. close browser

## Confirmed Historical Discoveries

- visible site-search automation was not the reliable core solution
- autosuggest was not the authoritative item-mapping source
- the real Ferguson search results page contained usable item-number mapping
- a storefront can look partially advanced while still behaving like guest state
- login validation must be based on authenticated page state, not URL alone
- pricing became reliable only after the Ferguson item number and runtime account context were known
- model-number sanitization is required for stable searching

## Current Canonical Assets

- **Canonical script:** `Ferguson Item Number Scraper.py`
- **Canonical workbook:** `Ferguson Item Number Scraper Template.xlsx`
- **Canonical worksheet:** `Scraper Data`
- **Dedicated profile folder:** `Ferguson Item Number Scraper Profile`
- **Debug folder:** `debugging`

## Safe vs Risky Changes

### Safer changes
- documentation updates
- debug-file organization
- small parser hardening changes
- improved warnings and diagnostics
- retry tuning

### Higher-risk changes
- changing workbook column contract
- rewriting authentication
- changing the search source of truth
- changing pricing context extraction
- broad refactors without sample validation

## Required Revalidation After Login Changes

Any authentication rewrite must revalidate all of the following:

1. authenticated session is truly established
2. Ferguson item numbers are still extracted correctly
3. prices are still returned correctly
4. Excel writes still land in the intended workbook/sheet/columns
