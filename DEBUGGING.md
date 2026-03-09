# Debugging Guide

## Purpose
This guide helps future agents and developers diagnose failures in Ferguson Item Number Scraper without repeating the full discovery process.

## First Rule
Always determine **which layer is failing** before changing code:
1. Excel input/output
2. authentication/session establishment
3. search-page lookup
4. item-number parsing
5. runtime pricing context extraction
6. pricing endpoint call

Do not assume the visible symptom points to the real cause.

## Fast Triage Checklist
When the script fails, answer these first:
- Did Chrome open and reuse the correct profile?
- Was Ferguson truly authenticated?
- Did the script search for the expected sanitized model number?
- Did the returned search page contain the model and item mapping?
- Was the runtime pricing context present?
- Did the pricing endpoint return the expected structure?
- Did Excel writeback succeed?

## Known Good Regression Example
Use this whenever possible for quick validation:
- **Model number:** `MN-ZMBBU0904`
- **Expected Ferguson item number:** `1115021`
- **Expected price:** `$14.510`

If this known case fails, the problem is likely systemic rather than row-specific.

## Layer-by-Layer Troubleshooting

### 1. Excel Layer Problems
#### Symptoms
- workbook not found
- worksheet not found
- no rows processed
- values not written back
- output workbook missing

#### Checks
- verify the script folder is correct
- verify `Ferguson Item Scraper Template.xlsx` exists beside the script
- verify the worksheet is `Scraper data`
- verify the configured columns match the workbook layout
- verify the output workbook path is writable

#### Likely causes
- workbook renamed without updating code
- worksheet renamed without updating code
- column mapping drift
- workbook open/locked by another process

---

### 2. Authentication Problems
#### Symptoms
- Ferguson opens but is not actually logged in
- homepage loads as guest
- script appears to proceed but later calls fail or behave like guest traffic
- login works manually in Chrome but not in automated form-submission attempts

#### Checks
- verify the correct dedicated profile folder is being used
- inspect auth-state output if available
- confirm Ferguson is actually logged in inside the opened browser window
- verify the site is not still showing `LOGIN` / `CREATE ACCOUNT`
- verify the script is not using a fresh empty profile unintentionally

#### What we learned historically
- URL change alone is **not** reliable proof of login
- earlier form-based login attempts could look partially successful while still ending in guest state
- manual login with a persistent profile worked reliably

#### Recommended response
For production stability, prefer the manual-login profile baseline unless explicitly testing a new login method.

---

### 3. Search Request Problems
#### Symptoms
- script says it searched but nothing is found
- known good models come back blank
- requests complete without errors but no item is extracted

#### Checks
- inspect the exact search URL used
- verify the sanitized model number
- verify the raw model number if fallback search is used
- save and inspect the returned search-page HTML
- search inside the HTML for the model number and `Item #`

#### Important historical discovery
The autosuggest endpoint was not the right authoritative source for item-number mapping. The real search results page was the successful path.

---

### 4. Parsing Problems
#### Symptoms
- search page loads correctly but item number is blank
- parsing works for some items but not others
- item extracted appears wrong or inconsistent

#### Checks
Search the returned HTML for these patterns near the model number:
- `Mfr. Part #`
- `Item #`
- `preselectedVariant=`
- `data-pid=`
- product URLs ending in `/1234567.html`

#### Likely causes
- Ferguson changed markup
- parser is too narrow
- parser is too broad and catches the wrong value
- input model contains stray punctuation or formatting characters

#### Important historical discovery
A captured search URL and page included a trailing curly apostrophe in the model string. Sanitization was required.

---

### 5. Pricing Context Problems
#### Symptoms
- item number is found but price is blank
- pricing endpoint call fails or returns no matching product pricing

#### Checks
Verify the script has all required runtime context:
- `warehouse_location_id`
- `branch_id`
- `customer_main_account_number`

#### Likely causes
- auth state changed or was not fully loaded
- parser failed to pull one of the context values
- page markup changed
- account/session is different from the one used during discovery

---

### 6. Pricing Endpoint Problems
#### Symptoms
- item number is known but price API returns nothing useful
- JSON structure changed
- returned pricing does not include the expected item id

#### Known endpoint
`Search-GetTilePricing`

#### Required parameters
- `productIDs`
- `shipWhseId`
- `branchId`
- `customerId`

#### Checks
- verify item number is correct
- verify request parameters match the authenticated account/session context
- verify the referer is a valid Ferguson search URL for that item/model lookup
- inspect the JSON response shape

#### Historical discovery
Pricing was solved once the item number and account context were known. The missing link was item-number extraction from the search-results page.

---

## Historical Login Experiments and What Was Learned

### Automated Selenium login
#### Result
Failed as a dependable production method.

#### Symptoms seen
- email visually entered correctly
- site moved forward visually but still ended in guest state
- false positives when using URL changes as the success check
- misleading UI states like account-not-found behavior or guest homepage redirects

#### Lesson
Visual progression is not enough. Authentication must be validated through page state.

### Hidden-character / spacing theory
#### Result
Plausible early on, but not the root cause of the login failure.

#### Lesson
Input normalization matters, but the deeper problem was the Ferguson/Salesforce handoff under automation.

### Stealth and anti-detection tricks
#### Result
Did not produce a dependable login solution.

#### Lesson
Could add complexity without solving the underlying handoff/session problem.

### Manual login with persistent profile
#### Result
Worked and became the current baseline.

#### Lesson
Use the real browser session for stability, then shift the data lookup work to HTTP requests.

## Recommended Debug Artifacts to Keep
For misses or major failures, it is useful to save:
- search-page HTML for the model
- final search URL
- auth-state snapshot
- runtime pricing context snapshot
- screenshot only when the browser state is relevant

Avoid generating heavy debug output for every successful row.

## Safe Debugging Workflow
1. Reproduce the failure on one or two rows only.
2. Use a known-good regression item first.
3. Save the exact returned HTML or JSON.
4. Confirm which layer failed.
5. Make one small code change.
6. Re-test the same known example.
7. Only then broaden the run.

## When Exploring New Login Automation
If future work resumes automated login experimentation, always record:
- method tried
- why it was tried
- exact observed result
- whether the site was truly authenticated afterward
- whether item lookup still worked afterward
- whether the method looked promising or should be abandoned

## Good Debugging Questions for Future Agents
- Is the script actually authenticated, or only appearing to progress?
- Is the model string clean before search?
- Is the search HTML the real issue rather than the request itself?
- Did Ferguson change HTML markup or endpoint shape?
- Did the workbook contract drift from the production code?
- Is this a login problem, a parsing problem, or a pricing-context problem?

