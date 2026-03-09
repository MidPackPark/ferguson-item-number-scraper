## Working notes
Repository initialized and baseline v1.0 created on GitHub.

# Changelog

All meaningful project changes should be recorded here. This file is intended to help future agents and developers understand what changed, why it changed, and what was learned.

## Change Log Conventions
For each important change, try to record:
- date
- summary of change
- reason for change
- impact on login, lookup, pricing, or Excel behavior
- whether it was tested
- what was learned

---

## 2026-03-09
### Added working production baseline documentation
- Created `README.md`, `ARCHITECTURE.md`, and `DEBUGGING.md`.
- Updated `PRD.md` and `prompt.md` to align with the current production direction.
- Standardized project identity around **Ferguson Item Number Scraper**.

### Why
The project needed durable context for future agents and maintainers so they can safely build on the working baseline without repeating the full discovery process.

### Impact
- No code logic change.
- Documentation now explicitly distinguishes the flexible login layer from the more stable lookup and Excel layers.

### Tested
Documentation only.

---

## 2026-03-09
### Canonical workbook naming updated
- Canonical workbook name changed to `Ferguson Item Scraper Template.xlsx`.
- Canonical worksheet name changed to `Scraper data`.

### Why
The user updated the workbook and worksheet naming in the production code and wanted the documentation to match the new baseline.

### Impact
Future agents should treat these names as canonical unless deliberately changed.

### Tested
User-confirmed naming change.

---

## 2026-03-09
### Standardized project name and canonical script name
- Project standardized as **Ferguson Item Number Scraper**.
- Canonical script standardized as `Ferguson Item Number Scraper.py`.

### Why
The project needed a single durable identity for code, docs, and future maintenance.

### Impact
Reduces drift across future rewrites, prompts, and documentation.

### Tested
Documentation and file naming update.

---

## 2026-03-09
### Production script cleaned up from debug-heavy working versions
- Removed a large amount of exploratory scaffolding from the working script.
- Kept focused debug output for misses and critical failures.
- Preserved manual-login profile baseline, item lookup, and pricing flow.

### Why
The project had reached a working state and needed a leaner production-ready code path.

### Impact
Cleaner production baseline with reduced noise while preserving essential troubleshooting hooks.

### Tested
Working baseline confirmed by user before cleanup request.

---

## 2026-03-09
### Confirmed working item-number and pricing flow
- Confirmed that the real Ferguson search results page contains the item-number mapping.
- Confirmed that `Search-GetTilePricing` returns account-specific pricing when called with the correct context.
- Confirmed example mapping:
  - model `MN-ZMBBU0904`
  - item number `1115021`
  - price `$14.510`

### Why
This discovery established the production lookup pipeline.

### Impact
The project no longer depends on the autosuggest endpoint or visible site-search automation for the main workflow.

### Tested
Yes. User confirmed the rewritten script worked.

---

## 2026-03-09
### Adopted manual login with persistent Chrome profile as the current authentication baseline
- Stopped trying to rely on form-based automated login as the primary production method.
- Switched to a dedicated Chrome profile and manual one-time login with session reuse.

### Why
Automated login attempts repeatedly failed or produced false guest states even when the UI suggested partial progress.

### Impact
Current local workflow is stable enough for production use.

### Tested
Yes. User confirmed manual login works.

---

## 2026-03-09
### Failed login approaches documented
Attempted and rejected or deferred as primary production methods:
- Selenium-driven login form submission
- trusting URL changes as proof of authentication
- hidden-character / stray-space theories as the primary cause
- stealth/anti-detection browser tricks as the main solution
- autosuggest endpoint as authoritative lookup source
- visible search-box automation as the main lookup path

### Why
These attempts either failed outright, produced misleading results, or were less stable than the eventual production baseline.

### Impact
Future agents should use this history to avoid repeating dead ends without a good reason.

### Tested
Yes during discovery/debugging.

---

## Future Entries Template
## YYYY-MM-DD
### Summary
- Change made
- Change made

### Why
Reason for the change.

### Impact
Describe whether login, parsing, pricing, workbook behavior, or deployment changed.

### Tested
Yes/No and how.

### Notes
Any discoveries, caveats, or rollback concerns.

