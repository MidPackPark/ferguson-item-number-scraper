# Debugging Guide

## Purpose

This document records the current known failure points, the active debugging strategy, and the lessons learned while stabilizing the Ferguson Item Number Scraper.

## Current Debug Output Behavior

The current baseline writes debug artifacts into a local folder named:

```text
debugging
```

### Current debug file types
- `debug_*.json`
- `debug_*.html`
- `debug_*.png`

### What gets saved
- startup authentication state
- manual login prompt page
- manual login post-check auth state
- runtime pricing context
- limited search-page debug artifacts for misses

### Current cleanup behavior
At startup, the script clears the **contents** of the `debugging` folder but does **not** delete the folder itself.

This was adopted because deleting the whole folder caused Windows / OneDrive permission errors when:
- a debug file was open
- File Explorer was pointed at the folder
- OneDrive was syncing the folder
- Windows held a file/folder handle briefly after a prior run

## Most Common Failure Layers

### 1. Workbook / sheet contract
Symptoms:
- workbook not found
- worksheet key errors
- wrong sheet name
- values not read or written where expected

Current baseline:
- workbook: `Ferguson Item Number Scraper Template.xlsx`
- worksheet: `Scraper Data`

### 2. Authentication / session
Symptoms:
- browser opens but is still guest
- page looks partially logged in but pricing/search acts like guest
- manual login prompt repeats
- pricing context missing

Checks:
- verify login on the actual Ferguson site
- confirm authenticated state by page data, not URL alone
- verify runtime auth JSON if saved

### 3. Search request
Symptoms:
- no item matches
- obvious valid items come back as not found
- search page content does not match browser expectation

Checks:
- inspect saved search-page HTML
- compare browser-rendered page to requests-returned page
- confirm sanitized search term

### 4. HTML parsing
Symptoms:
- Ferguson search page clearly shows the item
- script still reports not found

Checks:
- verify `sanitize_model_number()` exists and is being called
- confirm `pick_item_from_text()` still includes:
  - strong model-linked patterns
  - nearby-window search
  - generic fallback patterns

### 5. Pricing / API
Symptoms:
- item number found
- price missing or wrong
- runtime pricing context incomplete

Checks:
- inspect `debug_runtime_pricing_context.json`
- verify `branch_id`, `warehouse_location_id`, and `customer_main_account_number`
- confirm the item number is passed into pricing lookup

### 6. Debugging layer itself
Symptoms:
- script crashes before main work
- debug cleanup permission errors
- helper function `NameError`

Checks:
- confirm `reset_debug_dir()` clears contents rather than deleting the entire folder
- confirm required top-level helpers still exist

## Required Top-Level Helper Functions

The current code depends on these helpers existing at top level:

- `reset_debug_dir()`
- `save_json_debug()`
- `save_text_debug()`
- `save_page_debug()`
- `wait_for_dom_ready()`
- `dismiss_cookie_banner()`
- `sanitize_model_number()`

If one of these is removed, renamed, or accidentally indented inside another function, the script may fail with `NameError`.

## Known Historical Failures and Fixes

### Failure: browser opened and immediately closed
Cause:
- helper function `wait_for_dom_ready()` was missing

Fix:
- restore the helper at top level above browser/session helpers

### Failure: browser opened and then failed during auth
Cause:
- helper function `dismiss_cookie_banner()` was missing

Fix:
- restore the helper at top level

### Failure: every row failed after successful login
Cause:
- helper function `sanitize_model_number()` was missing

Fix:
- restore the helper and keep it in the utility/helper section

### Failure: search pages showed valid items but parser returned not found
Cause:
- generic parser fallback block had been removed from `pick_item_from_text()`

Fix:
- restore the generic fallback block

### Failure: debug folder cleanup crashed the script
Cause:
- script tried to delete the whole `debugging` folder with `shutil.rmtree()`

Fix:
- keep the folder, remove contents safely, warn on locked files instead of crashing

## Current Recommended Debug Process

When a new issue appears, identify the layer first:

1. workbook / sheet
2. authentication / session
3. search request
4. HTML parsing
5. pricing / API
6. result write-back

Then gather only the minimum useful evidence:
- exact console traceback
- current `.py` file
- relevant debug HTML/JSON files
- screenshot only if it adds something not visible in text

## Current Search-Parsing Notes

The current parser depends on model sanitization before searching and matching.

`sanitize_model_number()` currently:
- unescapes HTML-style text
- removes curly quotes
- removes straight quotes
- normalizes non-breaking spaces
- collapses repeated whitespace
- uppercases the final value

This preprocessing remains important because hidden punctuation and spacing issues can break valid Ferguson searches.

## Current Miss-Artifact Behavior

When an item is not found, the script currently saves only a small number of miss pages for review. This keeps the `debugging` folder useful without becoming too large during a long run.

## What Must Remain Stable

- worksheet name `Scraper Data`
- dedicated profile login baseline
- authenticated search-page lookup flow
- separation between item lookup and pricing lookup
- lightweight but useful debug capture
