# Product Requirements Document

## 1. Project Name

Ferguson Item Number Scraper

## 2. Objective

Build and maintain a stable Python automation that:

1. reads manufacturer model numbers from an Excel workbook
2. finds the corresponding proprietary Ferguson item number
3. retrieves account-specific pricing when available
4. writes the results back into Excel for downstream estimating and pricing workflows

## 3. Business Goal

The business goal is to use Ferguson item numbers as a reliable bridge between manufacturer model numbers and downstream pricing retrieval, helping support a larger plumbing bidding and pricing data workflow.

## 4. Current Accepted Baseline

The current accepted production-style baseline is:

- manual login with a dedicated persistent Chrome profile
- authenticated Ferguson session reused where possible
- item-number lookup from the real Ferguson search results page
- pricing retrieval through Ferguson `Search-GetTilePricing`
- Excel read/write through `openpyxl`

This baseline is considered working and should be preserved unless a change is explicitly requested and revalidated.

## 5. Canonical Files and Workbook Contract

### Canonical script
- `Ferguson Item Number Scraper.py`

### Canonical workbook
- `Ferguson Item Number Scraper Template.xlsx`

### Canonical worksheet
- `Scraper Data`

### Current output workbook
- `Ferguson Item Number Scraper Final.xlsx`

## 6. Functional Requirements

### 6.1 Excel input
The system must:
- open the canonical workbook
- read model numbers from worksheet `Scraper Data`
- begin reading at row 2
- ignore blank model-number rows

### 6.2 Excel output
The system must:
- write Ferguson item numbers to column C
- write pricing to column E when available
- save progress periodically
- save final output as a separate workbook

### 6.3 Authentication
The system must:
- open Ferguson in Chrome using a dedicated persistent profile
- reuse an existing session when available
- allow manual login when no authenticated session exists
- verify authenticated state before proceeding

### 6.4 Item lookup
The system must:
- sanitize model numbers before search
- search Ferguson using the real search results page
- parse the Ferguson item number from returned HTML
- support both a primary parsing path and fallback parsing logic

### 6.5 Pricing lookup
The system must:
- extract runtime pricing context from the logged-in page
- request pricing through `Search-GetTilePricing`
- return account-specific pricing when Ferguson provides it

### 6.6 Diagnostics
The system must:
- save useful debug artifacts when needed
- write those artifacts to `debugging`
- avoid uncontrolled debug-file buildup over time

## 7. Non-Functional Requirements

The solution should be:
- conservative with working code
- small-change friendly
- debuggable
- able to run from a local synced working folder
- resilient to minor Ferguson HTML differences
- safe for iterative maintenance by future agents

## 8. Known Constraints

- Ferguson login flow is brittle for full automation
- storefront state can appear partially advanced while still acting like guest
- Ferguson HTML can vary enough that parser fallback is required
- pricing depends on runtime account context from the authenticated session
- Windows / OneDrive can lock debug folders and files during sync or preview

## 9. Confirmed Technical Discoveries

- autosuggest is not the authoritative item-mapping source for this project
- the real search results page is the validated item-lookup source
- item lookup and pricing lookup are separate workflows
- `sanitize_model_number()` is required for reliable searching and matching
- helper functions such as `wait_for_dom_ready()` and `dismiss_cookie_banner()` must remain present at top level
- deleting the entire debug folder is less reliable than clearing its contents

## 10. Safe Change Boundaries

### Safe to change cautiously
- authentication strategy
- logging format
- retry behavior
- parser hardening
- documentation

### Must remain stable unless deliberately revalidated
- workbook name and sheet contract
- item lookup before pricing lookup
- authenticated search-page-based item extraction
- Excel write-back behavior

## 11. Validation Requirements After Any Meaningful Change

Any meaningful change should be tested by confirming:

1. authenticated session works
2. Ferguson item numbers are found correctly
3. pricing still returns correctly
4. workbook writes back correctly
5. output workbook saves correctly

Start with a small sample batch before running larger sets.
