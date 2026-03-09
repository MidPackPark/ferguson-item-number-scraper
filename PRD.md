# Product Requirements Document

## Project Name
Ferguson Item Number Scraper

## Document Purpose
This PRD defines the purpose, scope, workflow, architecture, assumptions, dependencies, known risks, debugging methods, and future roadmap for Ferguson Item Number Scraper. It is intended to be detailed enough that a future agent or developer can:
- reconstruct the working solution
- diagnose common failures
- safely modify the code
- extend it into price scraping and master database integration

---

## 1. Executive Summary
Ferguson Item Number Scraper is a Python automation that takes manufacturer model numbers from an Excel workbook, searches Ferguson.com for those model numbers, extracts Ferguson’s proprietary item number, and writes that item number back into the workbook. The current working baseline also retrieves account-specific pricing for matched items and writes that value back into the workbook.

The broader business goal is to take sales price Excel documents from manufacturer websites and enrich them with Ferguson item numbers. Those Ferguson item numbers are intended to be used later to scrape prices and populate a master pricing database used for bidding plumbing projects.

---

## 2. Business Context and Problem Statement

### 2.1 Current Business Need
The user works with manufacturer pricing sheets in Excel and needs a scalable way to normalize those parts against Ferguson’s internal catalog. Manufacturer model numbers alone are not enough for the downstream pricing workflow. Ferguson’s internal item number is the key that unlocks account-specific pricing and future catalog integration.

### 2.2 Core Problem
A manufacturer part number does not directly map to the user’s master pricing workflow. The missing bridge is Ferguson’s proprietary item number. Manual lookup is too slow and error-prone for production use.

### 2.3 Why This Project Exists
The project exists to automate this mapping step so that manufacturer catalogs can be enriched with Ferguson item numbers and later with Ferguson pricing.

---

## 3. Product Goal
Build a reliable Python tool that:
1. reads model numbers from a known Excel worksheet
2. authenticates to Ferguson.com
3. searches Ferguson using those model numbers
4. extracts Ferguson item numbers from search results
5. optionally retrieves account-specific price data
6. writes the results back to the workbook
7. is stable enough to become part of a larger production pricing pipeline

---

## 4. Project Scope

### 4.1 In Scope
- read model numbers from Excel
- use an authenticated Ferguson session
- search Ferguson by model number
- extract Ferguson item numbers from search-results page HTML
- retrieve item pricing using Ferguson internal endpoints
- write item number and price back to Excel
- save output workbook periodically
- preserve authenticated session in a dedicated Chrome profile
- provide enough debug output to diagnose misses

### 4.2 Out of Scope for Current Version
- fully headless unattended login
- running reliably on a server without manual authentication bootstrap
- broad ERP integration
- multi-vendor normalization beyond Ferguson
- full product detail extraction beyond item number and price
- automatic session refresh or credential rotation
- CAPTCHA or anti-bot bypass

---

## 5. Users and Stakeholders

### 5.1 Primary User
Ryan Parker

### 5.2 Future Users
- internal assistants/agents helping maintain the automation
- future developers expanding the pricing pipeline
- office/estimating workflows that consume the enriched output workbook

### 5.3 Stakeholder Outcome
The user needs a reliable automation that reduces manual lookup time and forms the first step in a larger pricing database process.

---

## 6. Functional Requirements

### FR-1: Workbook Input
The script shall load an Excel workbook from the same folder as the script. The current canonical implementation expects:
- input workbook: `Ferguson Item Scraper Template.xlsx`
- worksheet: `Scraper data`
- processing starts at row 2
- model number column, Ferguson item number output column, and price output column must remain aligned with the production Python script

### FR-2: Session Authentication
The script shall open Ferguson in Chrome using a dedicated profile folder stored beside the script. The user logs in manually when needed, and later runs reuse the authenticated session. The script shall validate that the current session is truly authenticated rather than assuming success based only on URL changes.

### FR-3: Build Authenticated HTTP Session
After authentication is confirmed, the script shall copy browser cookies into a `requests.Session` so that search and pricing calls can run via HTTP requests instead of visible browser interaction.

### FR-4: Search by Manufacturer Model Number
The script shall generate a Ferguson search URL using query parameter `q` and load the real search results page rather than relying on the autosuggest endpoint.

### FR-5: Sanitize Model Numbers Before Search
The script shall sanitize model numbers before search. This is required because actual search pages captured during debugging showed malformed terms containing stray curly apostrophes/quotes.

### FR-6: Extract Ferguson Item Number from Search Results HTML
The script shall parse the HTML of the search results page and extract the Ferguson item number using product-linked patterns. Confirmed patterns during discovery included:
- `Mfr. Part # {model}`
- `Item # {item}`
- product URL ending in `/{item}.html`
- `preselectedVariant={item}`
- `data-pid="{item}"`

### FR-7: Retrieve Price Through Ferguson Internal Endpoint
Once an item number is found, the script shall call Ferguson’s internal price endpoint:
`/on/demandware.store/Sites-Ferguson-Site/en_US/Search-GetTilePricing`

The request must be made with an authenticated session and the required runtime account parameters.

### FR-8: Runtime Pricing Context
The script shall extract account-dependent pricing context from the logged-in page. The current implementation relies on values such as:
- warehouse location ID
- branch ID
- customer main account number

### FR-9: Write Results to Workbook
For each matched row, the script shall write:
- Ferguson item number to the configured item output column
- price to the configured price output column when available

It shall skip rows that already have an item number and save the workbook periodically during execution.

### FR-10: Output Workbook
The script shall save results to a separate output workbook rather than overwriting the source workbook by default.

---

## 7. Non-Functional Requirements

### NFR-1: Reliability Over Elegance
The working solution should prioritize dependable execution over theoretical purity. Manual login plus profile reuse is acceptable because it is more reliable than the failed login-automation attempts tried so far.

### NFR-2: Production Orientation
The code should be lean, readable, and maintainable. Debugging scaffolding should be limited to misses, major failures, and critical diagnostics.

### NFR-3: Limited User Interaction
The current production baseline requires user interaction only when login must be performed or renewed.

### NFR-4: Traceability
The code should leave enough breadcrumbs to diagnose failures without being cluttered with excessive debug noise.

### NFR-5: Local-First Operation
The script should operate from its own folder using local files and a local Chrome profile.

---

## 8. Confirmed Working Architecture

### 8.1 High-Level Flow
1. launch Chrome with a dedicated local profile
2. reuse Ferguson login session or prompt user to log in manually
3. confirm authenticated state from page data or equivalent page-state signals
4. copy browser cookies into a `requests.Session`
5. read Excel workbook rows with model numbers
6. sanitize model number
7. request Ferguson search page using the authenticated session
8. parse returned HTML for Ferguson item number
9. call `Search-GetTilePricing` with item number plus runtime account parameters
10. write item number and price to workbook
11. periodically save workbook
12. close browser cleanly

### 8.2 Why This Architecture Was Chosen
This architecture is the result of multiple failed approaches and debugging rounds:
- direct Selenium login flow produced false positives and guest redirects
- the Ferguson/Salesforce login handoff was unreliable under automation
- the autosuggest endpoint returned empty fragments and was not authoritative for item lookup
- manual login with a persistent Chrome profile worked reliably
- the search page HTML contained the exact item-number mapping needed
- the price endpoint worked once the Ferguson item number and account context were known

This is the current best-known stable approach.

---

## 9. Rejected or Failed Approaches and Why They Failed

### 9.1 Selenium Form Automation for Login
This approach appeared to type the email correctly, but post-login state still showed guest behavior. The site could visually progress while remaining unauthenticated.

### 9.2 Trusting URL Change as Proof of Login
This was unreliable. The site could redirect to a normal homepage while still treating the session as a guest.

### 9.3 Search Box UI Automation
Driving the visible search box was brittle and unnecessary. The robust solution is authenticated HTTP search-page requests.

### 9.4 Autosuggest Endpoint as Main Lookup Source
The autosuggest endpoint was tied to header search suggestions and did not return useful authoritative item mappings for this workflow.

### 9.5 Anti-Detection / Stealth Login Tricks
Earlier iterations used automation-masking flags and browser tricks. These did not produce a reliable login and may have added complexity without solving the real handoff/session problem.

---

## 10. Key Technical Discoveries from Debugging

### 10.1 Real Search Form Behavior
The page search form submits to Ferguson’s real search results page using the query parameter `q`.

### 10.2 Search Results Page Contains Item Mapping
The returned HTML can embed the item mapping directly and is parseable without needing a dedicated item-lookup API.

### 10.3 Pricing Endpoint Is Separate
Pricing is not embedded in the same deterministic way as the item mapping. Ferguson calls a separate endpoint for tile pricing using internal product IDs and account context.

### 10.4 Manual Login Truly Works
Manual-login verification showed authenticated commercial-account state, while automated login attempts repeatedly produced guest state or misleading UI transitions.

### 10.5 Input Sanitization Is Mandatory
A malformed curly apostrophe was present in a captured search URL and corresponding page output, proving that search input normalization is required.

---

## 11. Current File and Folder Expectations
The script is designed to operate from its own folder. The working folder is the root for:
- the Python script
- the input workbook
- the Chrome profile directory
- optional debug output files
- project documentation files

The user’s permanent working folder is:
`C:\Users\rparker\OneDrive - HillGrp.com\HMS Sales - Documents\Estimating Templates\Data\Automations\Ferguson PVF Item Number Scraper`

---

## 12. Data Contract

### 12.1 Input Data Contract
For each row beginning at row 2:
- the configured model-number column contains the manufacturer model number

### 12.2 Output Data Contract
For each successful match:
- the configured output column receives the Ferguson item number
- the configured price column receives the Ferguson price, when returned

### 12.3 Row Processing Rules
- rows with blank model numbers are ignored
- rows with existing item-number values are skipped
- workbook is periodically autosaved

---

## 13. Search and Parse Logic Requirements

### 13.1 Search Strategy
The script should search using at least the cleaned model number and, where helpful, the original raw model number as fallback.

### 13.2 Strong Parse Patterns
The parser should prioritize model-linked patterns before generic patterns.

### 13.3 Fallback Patterns
Generic fallbacks should be limited because they can raise the risk of false positives.

---

## 14. Runtime Pricing Context Requirements
Pricing retrieval depends on values discovered at runtime from the authenticated page. The script should not hardcode these unless absolutely necessary.

If required pricing-context values are missing, price retrieval should fail gracefully and item-number extraction should still be allowed to succeed.

---

## 15. Logging and Debugging Requirements

### 15.1 Keep in Production
The production baseline should keep lightweight debug support for misses and major failures.

### 15.2 Recommended Debug Files
When a model fails to map, save:
- the returned search-page HTML for that model
- the final URL used
- optionally a screenshot or browser page source if the browser state is relevant

### 15.3 Debug Principles
- save debug only on misses or critical failures
- avoid large-volume logs on successful runs
- preserve enough context to reconstruct the request and HTML response

---

## 16. Known Risks and Constraints

### 16.1 Website Terms and Acceptable Use
Any future use of this project should be evaluated with awareness that the target site may restrict scraping, data mining, or automated extraction.

### 16.2 Session Fragility
The authenticated session may expire, which would require manual re-login.

### 16.3 Layout / Markup Changes
Search-page parsing depends on Ferguson HTML patterns. Site markup changes may break extraction logic.

### 16.4 Endpoint Changes
`Search-GetTilePricing` parameters or response format may change.

### 16.5 Account-Dependent Pricing
Prices are context-specific and can vary by branch, warehouse, account, contract, or related business logic.

### 16.6 Server Automation Is Not Yet Solved
The current working baseline depends on a reusable browser profile. Running in a server or scheduled headless environment remains a future goal, not a solved requirement.

---

## 17. Future Roadmap

### Phase 1: Current Baseline
- manual login via dedicated Chrome profile
- search page HTML for Ferguson item number
- internal price endpoint for account-specific price
- write results to Excel

### Phase 2: Production Hardening
- configurable filenames and sheet mappings
- cleaner structured logging
- retry logic for transient HTTP failures
- better miss-reporting summary
- optional dry-run mode

### Phase 3: Headless / Server Exploration
- test whether profile-based session reuse can be made server-safe
- explore authenticated cookie persistence without manual browser session
- determine whether an enterprise-approved automation path exists
- regress backward carefully toward unattended execution

### Phase 4: Database Integration
- push Ferguson item number and/or price into a master pricing database
- integrate with the larger bidding workflow
- use Ferguson item numbers as durable cross-reference keys in the pricing system

---

## 18. Change Management Guidance for Future Agents
Any future agent modifying this project should preserve the following invariants unless there is a compelling reason to change them:
1. keep manual-login profile reuse as the **current baseline** unless testing a new login method intentionally
2. do not use the autosuggest endpoint as the main lookup path for item numbers
3. keep model-number sanitization in place
4. use authenticated HTTP requests for search and pricing after session establishment
5. treat item-number extraction and price extraction as separate steps
6. preserve local script-folder behavior so deployment remains portable
7. keep output separate from input by default
8. retain lightweight miss-debug support until the script has a long stable production history

Also note: the login/authentication layer is intentionally **not locked down**. Future agents may redesign it substantially, including replacing the current manual-login profile approach, provided they preserve or revalidate the authenticated item lookup workflow and Excel output behavior.

---

## 19. Diagnostic Playbook

### Symptom: Script opens Ferguson but is not actually logged in
Check:
- whether the profile folder is the expected one
- whether the Ferguson session expired
- whether auth state still shows guest behavior
- whether page text still contains login/create-account prompts

### Symptom: Search runs but nothing is found for known-valid model numbers
Check:
- whether the model number contains stray punctuation or quotes
- the exact search URL used
- the saved search-page HTML for the model
- whether Ferguson changed the markup around `Item #`, `preselectedVariant`, `data-pid`, or product URLs

### Symptom: Item number is found but price is blank
Check:
- whether runtime pricing context was extracted
- whether pricing request parameters match the current logged-in account context
- whether Ferguson changed the `Search-GetTilePricing` response shape

### Symptom: Script works locally but not on a server
Check:
- whether the server has a persistent Chrome profile
- whether interactive/manual login is possible on that environment
- whether session cookies are being preserved across runs
- whether headless mode changes response behavior

---

## 20. Suggested Configuration Externalization for Future Refactor
When this project is revisited for production hardening, these values should be candidates for external configuration:
- input workbook filename
- output workbook filename
- worksheet name
- input/output column indexes
- start row
- save interval
- profile directory name
- timeout values
- debug retention policy

---

## 21. Acceptance Criteria
The project is considered successful when:
1. the user can run the script from its permanent folder
2. the script can reuse a dedicated Chrome profile for Ferguson authentication
3. the script successfully reads model numbers from the input workbook
4. for known-valid model numbers, the script writes the correct Ferguson item number to the output workbook
5. when pricing context is available, the script writes the corresponding Ferguson price
6. the script can process multiple rows in one run and autosave progress
7. on misses, the script produces enough debug context to investigate without redoing discovery from scratch

---

## 22. Example Confirmed Mapping
A confirmed working example from debugging:
- model number: `MN-ZMBBU0904`
- Ferguson item number: `1115021`
- price response returned `$14.510`

This example should be used as a regression test case whenever the parser or pricing logic is changed.

---

## 23. Open Questions
1. Can the authenticated session eventually be preserved safely for true scheduled unattended runs?
2. Is there a stable, sanctioned API path for product lookup that would be better than HTML parsing?
3. Can pricing retrieval be expanded to include additional metadata needed for the master pricing database?
4. Should the script eventually write to a database directly rather than to an intermediate workbook?
5. Can retry and throttling rules be added without introducing anti-bot triggers?

---

## 24. Summary
Ferguson Item Number Scraper is now a working bridge between manufacturer model numbers and Ferguson’s internal item/pricing ecosystem. Its stable baseline is:
- manual-profile Ferguson login
- authenticated search-page requests
- HTML parsing for item numbers
- authenticated pricing API calls
- Excel writeback

That baseline should be preserved as the production foundation. Any future refactor should be measured against whether it maintains or improves the reliability of that workflow.

