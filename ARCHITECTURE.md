# Architecture

## Purpose
This document explains the current technical design of Ferguson Item Number Scraper, identifies which layers are stable versus intentionally flexible, and records the discoveries that led to the working architecture.

## Design Philosophy
This project should be understood as a layered system rather than a single monolithic script. Future changes should preserve the validated lower-risk layers while allowing controlled experimentation in the problem areas, especially login automation.

## System Layers

### Layer 1: Excel Input/Output Layer
This is the most protected layer.

#### Responsibilities
- locate the input workbook
- open the target worksheet
- read model numbers from the configured input column
- write Ferguson item numbers to the configured output column
- write prices to the configured price column
- autosave progress periodically
- save to a distinct output workbook by default

#### Stability
**Highly stable.** Avoid unnecessary changes here.

#### Why it matters
This layer defines the interface between the scraper and the user's estimating workflow. Breaking this layer breaks the project’s business value immediately.

---

### Layer 2: Lookup Workflow Layer
This is the validated core lookup pipeline.

#### Responsibilities
- sanitize manufacturer model numbers
- request the real Ferguson search results page
- parse the returned HTML for the Ferguson item number
- use the matched item number to request pricing

#### Stability
**Mostly stable.** Changes should be small and regression-tested.

#### Key discovery
The real search results page contained the item-number mapping directly, while the autosuggest endpoint did not provide a reliable authoritative mapping for the project’s needs.

---

### Layer 3: Pricing Layer
This layer is separate from lookup and should stay separate.

#### Responsibilities
- extract runtime pricing context from the logged-in Ferguson session
- call `Search-GetTilePricing`
- parse account-specific price output for the matched Ferguson item number

#### Stability
**Stable but dependent on external markup and endpoint behavior.**

#### Key discovery
Pricing requires:
- `productIDs`
- `shipWhseId`
- `branchId`
- `customerId`

These values are account-dependent and should be extracted at runtime instead of hardcoded.

---

### Layer 4: Authentication / Session Establishment Layer
This is the least stable and the most open to redesign.

#### Current production baseline
- open Chrome with a dedicated persistent profile
- reuse an existing Ferguson session if present
- if not authenticated, let the user log in manually
- verify the session is truly authenticated
- copy browser cookies into a `requests.Session`

#### Stability
**Intentionally flexible.** This layer may be rewritten substantially in the future.

#### Important rule
A future agent may redesign authentication aggressively, but any replacement must preserve or revalidate the downstream lookup and Excel behavior.

---

### Layer 5: Debugging and Diagnostics Layer
This is the safety net for future maintenance.

#### Responsibilities
- save lightweight debug artifacts for misses and major failures
- capture enough context to understand why a row failed
- avoid excessive logging on successful runs

#### Stability
**Moderately stable.** Keep enough to diagnose issues without clutter.

## Current Working Flow
1. Start the script from its local working folder.
2. Launch Chrome with a dedicated persistent profile.
3. Check whether Ferguson is already authenticated.
4. If not, prompt for manual login.
5. Verify authenticated state using page-state signals, not URL alone.
6. Transfer browser cookies into a `requests.Session`.
7. Read rows from the workbook.
8. Sanitize each model number.
9. Request the Ferguson search page for that model.
10. Parse the HTML for the Ferguson item number.
11. Retrieve price using `Search-GetTilePricing` and runtime account context.
12. Write results into the workbook.
13. Autosave periodically.
14. Save final output and close the browser.

## Why the Architecture Looks This Way
The project arrived at this structure through repeated trial and error.

### Earlier ideas that failed or were deprioritized
- visible site-search automation
- Selenium-only login automation
- autosuggest as the main lookup source
- URL-only login validation
- stealth-style anti-detection flags as the main solution

### What was learned
- the login problem and the lookup problem are separate
- authentication must be validated by session state, not by visual progress alone
- item lookup and pricing are separate Ferguson workflows
- the search page is a dependable source of item-number mapping
- the price endpoint becomes usable once the Ferguson item number and runtime account context are known

## Current Canonical Assets
- **Canonical script:** `Ferguson Item Number Scraper.py`
- **Canonical workbook:** `Ferguson Item Scraper Template.xlsx`
- **Canonical sheet:** `Scraper data`
- **Canonical working folder:**
  `C:\Users\rparker\OneDrive - HillGrp.com\HMS Sales - Documents\Estimating Templates\Data\Automations\Ferguson PVF Item Number Scraper`

## Runtime Dependencies
- Python
- `selenium`
- `requests`
- `openpyxl`
- local Chrome installation compatible with Selenium WebDriver

## Data Flow Summary
### Inputs
- Excel workbook rows containing manufacturer model numbers
- authenticated Ferguson browser session
- runtime account context from the logged-in page

### Outputs
- Ferguson proprietary item number written back into Excel
- Ferguson price written back into Excel when available
- limited debug files on misses or major failures

## Architecture Boundaries for Future Agents

### Safe to change aggressively
- authentication and session-establishment strategy
- structured logging format
- config externalization
- retry behavior
- browser automation library choice for login

### Change cautiously
- model-number sanitization
- HTML parsing logic for item-number extraction
- runtime context extraction for pricing
- output workbook naming strategy

### Do not change casually
- workbook/sheet contract without coordinated update
- separation between item lookup and pricing lookup
- the principle of validating authentication before lookup
- the principle of using the real search page as the validated item-number source unless a better source is proven

## Future Architecture Directions

### Direction 1: Automated login
Potential future replacements for the current login layer:
- more reliable Selenium login
- Playwright-based login
- persistent cookie/session serialization
- profile seeding
- enterprise-safe unattended login
- server-safe headless session reuse

### Direction 2: Config externalization
Potentially move these out of the main script:
- workbook name
- sheet name
- columns
- save interval
- output filename
- debug retention policy

### Direction 3: Database integration
Use Ferguson item numbers and prices as keys and values in the future master pricing database for plumbing estimating.

## Architectural Regression Standard
Any major rewrite should still be able to prove the following:
- a known valid model number can be processed successfully
- the correct Ferguson item number is extracted
- the correct price is retrieved when context is available
- the workbook receives the correct values in the expected places

