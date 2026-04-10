# DemoWebShop JMeter Test Plan – README

**Target Application:** https://demowebshop.tricentis.com/  
**JMeter Version:** 5.6.3+  
**File:** `DemoWebShop_TestPlan.jmx`

---

## Table of Contents

1. [Test Plan Overview](#1-test-plan-overview)
2. [Project Structure](#2-project-structure)
3. [Thread Group Configuration](#3-thread-group-configuration)
4. [What Is Correlation?](#4-what-is-correlation)
5. [What Is Regular Expression Extraction?](#5-what-is-regular-expression-extraction)
6. [All Correlation Points Explained](#6-all-correlation-points-explained)
7. [BeanShell Samplers Explained](#7-beanshell-samplers-explained)
8. [Debug Sampler Explained](#8-debug-sampler-explained)
9. [Listeners Explained](#9-listeners-explained)
10. [How to Run](#10-how-to-run)
11. [Results & Reporting](#11-results--reporting)
12. [Troubleshooting](#12-troubleshooting)

---

## 1. Test Plan Overview

This JMeter test plan simulates a real end-to-end e-commerce user journey on the
DemoWebShop site. The flow covered is:

```
Pre-Test Init (BeanShell)
  │
  ├─► 01 GET Homepage                → Extracts CSRF token + Customer GUID
  ├─► 02 GET Login Page              → Extracts fresh Login token + ReturnUrl
  ├─► 03 POST Login                  → Replays correlated tokens → Authenticated
  ├─► 04 GET Books Category          → Extracts Product URL + all Prices
  ├─► 05 GET Product Detail Page     → Extracts Product ID + Product Name
  │         BeanShell – Validate Correlations
  ├─► 06 POST Add to Cart            → Uses Product ID → Extracts Cart Subtotal
  ├─► 07 GET Shopping Cart           → Extracts Cart Page Token
  ├─► 08 GET Search Results          → Counts search result items
  │
  ├─► DEBUG Sampler                  → Dumps all variables to View Results Tree
  └─► Post-Test Summary (BeanShell)  → Prints summary table to logs
```

---

## 2. Project Structure

```
project-root/
├── DemoWebShop_TestPlan.jmx   ← Main JMeter test plan (this file)
├── README.md                  ← This documentation
└── results/                   ← Auto-created by JMeter at runtime
    ├── view_results_tree.jtl
    ├── summary_report.jtl
    └── aggregate_report.jtl
```

> **Tip:** Create the `results/` folder before running:
> ```bash
> mkdir -p results
> ```

---

## 3. Thread Group Configuration

| Parameter          | Variable        | Default |
|--------------------|-----------------|---------|
| Number of Threads  | `${THREADS}`    | 2       |
| Ramp-Up Period (s) | `${RAMP_UP}`    | 5       |
| Loop Count         | `${LOOP_COUNT}` | 1       |

All values are defined as **User Defined Variables** at the Test Plan level and
can be overridden from the command line:

```bash
jmeter -n -t DemoWebShop_TestPlan.jmx \
       -JTHREADS=10 -JRAMP_UP=30 -JLOOP_COUNT=3 \
       -l results/run1.jtl
```

The **HTTP Cookie Manager** is configured with `Clear cookies each iteration = true`
so every loop starts with a clean session – critical for replay correctness when
combined with correlation.

---

## 4. What Is Correlation?

### The Problem Without Correlation

Modern web applications protect their forms with **dynamic tokens** – values that
the server generates fresh on every page load and embeds invisibly in the HTML.
Examples on DemoWebShop:

* `__RequestVerificationToken` – ASP.NET's CSRF anti-forgery token
* `Nop.customer` cookie value – the user's session GUID
* Product IDs in Add-to-Cart URLs

When you record a JMeter script, these values are **hard-coded** to whatever they
were at record time. The server will **reject** any subsequent run that replays
the old, stale values (HTTP 400 Bad Request or a silent auth failure).

### The Solution: Correlation

Correlation is a two-step technique:

```
Step 1 – EXTRACT:  Use a Regular Expression Extractor to capture the dynamic
                   value from the server's response and store it in a variable.

Step 2 – REPLAY:   Reference that variable (${VAR_NAME}) in the next request
                   where the server expects that dynamic value.
```

This keeps your script valid across every test run because the values are always
fresh from the server.

---

## 5. What Is Regular Expression Extraction?

JMeter's **Regular Expression Extractor** (a Post-Processor) applies a regex
against a sampler's response and saves matching groups into JMeter variables.

### Key Settings

| Setting        | Purpose                                                            |
|----------------|--------------------------------------------------------------------|
| **Reference Name** | The JMeter variable name where the result is stored (e.g., `LOGIN_TOKEN`) |
| **Regular Expression** | The regex pattern with one or more capturing groups `(…)` |
| **Template**   | Which capturing group to store: `$1$` = group 1, `$2$` = group 2 |
| **Match No.**  | `1` = first match; `-1` = ALL matches (stored as an array); `0` = random match |
| **Default Value** | Value written to the variable if the regex finds nothing – used for debugging |
| **Apply To**   | `Main sample only` (HTML body) or `Response Headers` for header extraction |

### Anatomy of a Regex Pattern

```
<input name="__RequestVerificationToken"[^>]*value="([^"]+)"
│                                        │         │       │
│  Literal HTML text to anchor the match │         │       └─ End of captured value
│                                        │         └─ Capturing group 1: any chars except "
│                                        └─ Skip any other attributes between name and value
└─ Match starts here
```

---

## 6. All Correlation Points Explained

### CORR-01 – Extract `__RequestVerificationToken` (Homepage)

**Where:** POST-Processor of sampler `01 – GET Homepage`

**Why needed:** ASP.NET MVC embeds a CSRF token in every HTML form. Without
replaying the exact current token, any form POST will fail.

**HTML it targets:**
```html
<input name="__RequestVerificationToken" type="hidden"
       value="abc123XYZ...longtoken..." />
```

**Regex used:**
```
<input name="__RequestVerificationToken"[^>]*value="([^"]+)"
```

**Breakdown:**
* `<input name="__RequestVerificationToken"` – anchors to the specific input field
* `[^>]*` – matches any other HTML attributes between `name` and `value`
* `value="` – anchors to the value attribute
* `([^"]+)` – **Capturing Group 1**: matches one or more characters that are NOT
  a double-quote, capturing the full token value
* `"` – closing quote after the value

**Variable stored:** `${VERIFICATION_TOKEN}`

---

### CORR-02 – Extract Customer GUID from Cookie Header

**Where:** POST-Processor of sampler `01 – GET Homepage`  
**Apply To:** `Response Headers` (not HTML body)

**Why needed:** NopCommerce assigns every visitor a persistent Customer GUID via
a `Set-Cookie` header. Capturing it enables session-aware testing.

**Header it targets:**
```
Set-Cookie: Nop.customer=3f2504e0-4f89-11d3-9a0c-0305e82c3301; path=/
```

**Regex used:**
```
Nop\.customer=([a-f0-9\-]{36})
```

**Breakdown:**
* `Nop\.customer=` – literal text (`.` is escaped to `\.` to match a dot, not any character)
* `([a-f0-9\-]{36})` – **Capturing Group 1**: exactly 36 characters that are
  lowercase hex digits `a-f`, `0-9`, or a hyphen `\-` – the UUID format

**Variable stored:** `${CUSTOMER_GUID}`

---

### CORR-03 – Extract Login Page Token

**Where:** POST-Processor of sampler `02 – GET Login Page`

**Why needed:** The login page renders its OWN freshly generated token, separate
from the one on the homepage. Reusing the homepage token for the login POST would
cause a mismatch and the login would silently fail.

**Same regex as CORR-01**, applied specifically to the login page response.

**Variable stored:** `${LOGIN_TOKEN}` → **replayed in sampler `03 – POST Login`**

```xml
<!-- How it is replayed in the POST body -->
<Argument.name>__RequestVerificationToken</Argument.name>
<Argument.value>${LOGIN_TOKEN}</Argument.value>
```

---

### CORR-04 – Extract ReturnUrl

**Where:** POST-Processor of sampler `02 – GET Login Page`

**Why needed:** If the user was redirected to the login page from a protected
page, the original URL is embedded as a hidden `ReturnUrl` field. Replaying it
ensures the user lands on the right page after authentication.

**HTML it targets:**
```html
<input name="ReturnUrl" type="hidden" value="/account" />
```

**Regex used:**
```
name="ReturnUrl" value="([^"]*)"
```

**Breakdown:**
* `name="ReturnUrl"` – anchors to the correct hidden input
* `value="` – moves to the value attribute
* `([^"]*)` – **Capturing Group 1**: zero or more non-quote characters (using `*`
  instead of `+` because ReturnUrl can be empty)

**Variable stored:** `${RETURN_URL}`

---

### CORR-05 – Extract Post-Login Token

**Where:** POST-Processor of sampler `03 – POST Login`

**Why needed:** After a successful login, ASP.NET rotates and issues a **new**
anti-forgery token. All authenticated POST requests (checkout, profile update)
must use this newer token, not the pre-login one.

**Variable stored:** `${POST_LOGIN_TOKEN}`

---

### CORR-06 – Extract First Product URL

**Where:** POST-Processor of sampler `04 – GET Books Category Page`

**Why needed:** Product listing URLs contain slugs that change when products are
added or removed from the catalogue. Hard-coding `/fiction-and-literature-book`
would break if the listing order changes; extracting dynamically keeps the script
resilient.

**HTML it targets:**
```html
<h2 class="product-title">
  <a href="/fiction-and-literature-book">Fiction Title</a>
</h2>
```

**Regex used:**
```
class="product-title">\s*<a href="([^"]+)"
```

**Breakdown:**
* `class="product-title">` – anchors to the product title container
* `\s*` – allows optional whitespace/newlines between the `>` and the `<a>`
* `<a href="` – moves to the anchor's href
* `([^"]+)` – **Capturing Group 1**: the relative product URL

**Match No:** `1` (first product on the page)

**Variable stored:** `${PRODUCT_URL}` → **replayed as the path of sampler `05`**

---

### CORR-07 – Extract All Prices (Array Mode)

**Where:** POST-Processor of sampler `04 – GET Books Category Page`

**Why needed:** Demonstrates JMeter's array extraction (Match No = -1) to collect
ALL price values from a listing page for aggregate reporting.

**HTML it targets:**
```html
<span class="price actual-price">45.00</span>
```

**Regex used:**
```
class="price actual-price">([0-9]+\.[0-9]{2})<
```

**Breakdown:**
* `class="price actual-price">` – anchors to the price span
* `([0-9]+\.[0-9]{2})` – **Capturing Group 1**: one or more digits, a literal
  dot `\.`, then exactly two digits (standard decimal price format)
* `<` – the closing `<` that terminates the span's text

**Match No:** `-1` → JMeter stores results as:
```
${PRICE_matchNr}  = 4          (total number of prices found)
${PRICE_1}        = 45.00
${PRICE_2}        = 25.00
${PRICE_3}        = 15.00
${PRICE_4}        = 10.00
```

**Variables stored:** `${PRICE_1}`, `${PRICE_2}`, … `${PRICE_matchNr}`

---

### CORR-08 – Extract Product ID

**Where:** POST-Processor of sampler `05 – GET Product Detail Page`

**Why needed:** The "Add to Cart" endpoint URL is
`/addproducttocart/details/{productId}/1`. The product ID is dynamic and cannot
be hard-coded. It appears in the form's `action` attribute.

**HTML it targets:**
```html
<form action="/addproducttocart/details/13/1" method="post">
```

**Regex used:**
```
/addproducttocart/details/(\d+)/
```

**Breakdown:**
* `/addproducttocart/details/` – literal path prefix as anchor
* `(\d+)` – **Capturing Group 1**: one or more digits (`\d` = digit character class)
  representing the numeric product ID
* `/` – trailing slash

**Variable stored:** `${PRODUCT_ID}` → **replayed in sampler `06 – POST Add to Cart`**

```
Sampler 06 path: /addproducttocart/details/${PRODUCT_ID}/1
Parameter name:  addtocart_${PRODUCT_ID}.EnteredQuantity
```

---

### CORR-09 – Extract Product Name

**Where:** POST-Processor of sampler `05 – GET Product Detail Page`

**Why needed:** Captures the product name for use in assertion messages and the
BeanShell summary report, avoiding hard-coded strings.

**HTML it targets:**
```html
<div class="product-name"><h1>Fiction and Literature Book</h1></div>
```

**Regex used:**
```
class="product-name"><h1>([^<]+)</h1>
```

**Breakdown:**
* `class="product-name"><h1>` – anchors to the exact wrapping elements
* `([^<]+)` – **Capturing Group 1**: one or more characters that are NOT `<`,
  capturing all visible text before the closing tag
* `</h1>` – end of heading

**Variable stored:** `${PRODUCT_NAME}`

---

### CORR-10 – Extract Cart Subtotal from JSON

**Where:** POST-Processor of sampler `06 – POST Add to Cart`

**Why needed:** The Add-to-Cart endpoint returns a JSON response (not HTML).
The cart subtotal value is needed for reporting and assertions.

**JSON it targets:**
```json
{"success":true,"updatetopcartsectionhtml":"(1)","subtotal":"$5.00"}
```

**Regex used:**
```
"subtotal":"([^"]+)"
```

**Breakdown:**
* `"subtotal":"` – JSON key literal as anchor
* `([^"]+)` – **Capturing Group 1**: all characters until the closing double-quote
  (captures `$5.00` including the dollar sign)

**Variable stored:** `${CART_SUBTOTAL}`

---

### CORR-11 – Extract Cart Page Token

**Where:** POST-Processor of sampler `07 – GET Shopping Cart`

**Why needed:** The cart page renders yet another fresh token needed if the user
proceeds to checkout (a POST operation). Captures it ready for the next step.

**Variable stored:** `${CART_TOKEN}`

---

### CORR-12 – Count Search Results (Array Mode)

**Where:** POST-Processor of sampler `08 – GET Search Results`

**Why needed:** Validates that the search feature is returning results and provides
a count for the BeanShell summary.

**Regex used:**
```
class="product-item"
```

**Match No:** `-1` → Every match is captured; `${SEARCH_RESULT_matchNr}` holds
the total count of products on the search results page.

**Variable stored:** `${SEARCH_RESULT_matchNr}` (count of results)

---

## 7. BeanShell Samplers Explained

### BeanShell 1 – Pre-Test Initialisation

Runs **before** any HTTP requests. It:

* Records the test start timestamp into `${TEST_START_TIME}`
* Generates a per-thread unique email (`testuser1@demo.com`, `testuser2@demo.com` …)
  so multiple threads can run without collisions
* Initialises counters: `${REQUEST_COUNT}`, `${PASS_COUNT}`, `${FAIL_COUNT}`
* Writes an `=== Test STARTED ===` line to the JMeter log

### BeanShell 2 – Validate Correlations

Runs **after** the product detail page. It:

* Reads all correlated variables and checks if any are still at their default
  "NOT_FOUND" value (which means the regex failed)
* Increments `${PASS_COUNT}` or `${FAIL_COUNT}` accordingly
* Builds a formatted validation table that appears in **View Results Tree**
* Marks `SampleResult.setSuccessful(false)` if any extraction failed, making the
  failure visible in all JMeter listeners

### BeanShell 3 – Post-Test Summary

Runs **after** all HTTP requests. It:

* Builds a boxed ASCII summary table showing all counters, product name, cart total,
  and search result count
* Writes the summary to the JMeter log (`jmeter.log`)
* Sets the sampler result to PASS so it appears green in View Results Tree

---

## 8. Debug Sampler Explained

The **Debug Sampler** generates a special response that lists every JMeter
variable in scope at the moment it executes. In **View Results Tree**, clicking
the Debug Sampler result and selecting the **Response Data** tab shows output like:

```
JMeterVariables:
BASE_URL=demowebshop.tricentis.com
CART_SUBTOTAL=$5.00
CART_TOKEN=xH7abc...longtoken...
CUSTOMER_GUID=3f2504e0-4f89-11d3-9a0c-0305e82c3301
DYNAMIC_EMAIL=testuser1@demo.com
FAIL_COUNT=0
LOGIN_TOKEN=mno456...longtoken...
PASS_COUNT=1
PRICE_1=45.00
PRICE_2=25.00
PRICE_matchNr=4
PRODUCT_ID=13
PRODUCT_NAME=Fiction and Literature Book
PRODUCT_URL=/fiction-and-literature-book
REQUEST_COUNT=1
RETURN_URL=
SEARCH_RESULT_matchNr=6
TEST_START_TIME=2024-04-10 09:15:22
VERIFICATION_TOKEN=abc123...longtoken...
```

This is the fastest way to **verify that all correlations worked** without adding
print statements or opening a browser.

> **Settings:**
> * `displayJMeterVariables = true` → shows all `${VAR}` values
> * `displayJMeterProperties = false` → hides internal JMeter config
> * `displaySystemProperties = false` → hides OS-level properties

---

## 9. Listeners Explained

| Listener              | File saved to                        | Purpose                                 |
|-----------------------|--------------------------------------|-----------------------------------------|
| **View Results Tree** | `results/view_results_tree.jtl`      | Full request/response per sampler; ideal for debugging |
| **Summary Report**    | `results/summary_report.jtl`         | Per-label totals: avg, min, max, throughput, errors |
| **Aggregate Report**  | `results/aggregate_report.jtl`       | Same as Summary but with percentile columns |

> **Performance Note:** Disable `View Results Tree` for load tests with many
> virtual users. It buffers full response bodies and will consume memory.

---

## 10. How to Run

### Prerequisites

* Apache JMeter 5.6.3 or later ([jmeter.apache.org](https://jmeter.apache.org/))
* Java 11 or 17
* Internet access to reach `demowebshop.tricentis.com`

### GUI Mode (development / debugging)

```bash
# Start JMeter GUI
jmeter

# File → Open → DemoWebShop_TestPlan.jmx
# Press the green ▶ Run button
```

### Non-GUI Mode (CI / load test)

```bash
mkdir -p results

jmeter -n \
  -t DemoWebShop_TestPlan.jmx \
  -l results/run_$(date +%Y%m%d_%H%M%S).jtl \
  -j results/jmeter.log \
  -JTHREADS=5 \
  -JRAMP_UP=10 \
  -JLOOP_COUNT=2
```

### Generate HTML Report

```bash
jmeter -g results/run_*.jtl -o results/html-report/
```

---

## 11. Results & Reporting

After a run you will find:

* **`.jtl` files** – CSV/XML records of every sampler (open in JMeter or Excel)
* **`jmeter.log`** – BeanShell print statements, correlation summaries
* **`html-report/index.html`** – interactive HTML dashboard with charts

### Reading the Debug Sampler in View Results Tree

1. Open View Results Tree listener
2. Click the **"DEBUG – Dump All Variables"** row
3. Select the **"Response Data"** tab
4. Verify all extracted variables have real values (not `*_NOT_FOUND`)

---

## 12. Troubleshooting

| Symptom                                      | Likely Cause                                              | Fix                                                                        |
|----------------------------------------------|-----------------------------------------------------------|----------------------------------------------------------------------------|
| Login POST returns HTTP 400                  | `${LOGIN_TOKEN}` = `LOGIN_TOKEN_NOT_FOUND`               | Inspect login page HTML; update CORR-03 regex to match current HTML        |
| Add-to-Cart returns HTTP 404                 | `${PRODUCT_ID}` = `13` (default fallback)               | Inspect product page HTML; update CORR-08 regex                            |
| All correlated vars show default values      | Site HTML structure changed                              | Re-record the flow and update regexes in each extractor                    |
| BeanShell sampler shows "FAIL"               | One or more correlated variables not extracted           | Check Debug Sampler output for which variable is still at default          |
| Cookie not being sent on POST                | HTTP Cookie Manager absent or session restarted          | Ensure Cookie Manager scope is Thread Group level; check `clearEachIteration` |
| `SEARCH_RESULT_matchNr` = 0                  | HTML class name changed on search page                   | Update CORR-12 regex to match the current product-item CSS class            |
| Regex extracts nothing despite correct HTML  | Angle brackets `<>` not escaped in JMX XML              | Use `&lt;` for `<` and `&gt;` for `>` inside XML attribute values         |

---

## Summary: Correlation + Regular Expression Quick Reference

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  ID      │ Variable              │ Extracted From          │ Replayed In    │
├──────────┼───────────────────────┼─────────────────────────┼────────────────┤
│ CORR-01  │ VERIFICATION_TOKEN    │ Homepage body           │ (reference)    │
│ CORR-02  │ CUSTOMER_GUID         │ Homepage headers        │ Cookie / logs  │
│ CORR-03  │ LOGIN_TOKEN           │ Login page body         │ 03 POST Login  │
│ CORR-04  │ RETURN_URL            │ Login page body         │ 03 POST Login  │
│ CORR-05  │ POST_LOGIN_TOKEN      │ Post-login page body    │ future POSTs   │
│ CORR-06  │ PRODUCT_URL           │ Category page body      │ 05 GET Product │
│ CORR-07  │ PRICE_1…PRICE_n       │ Category page body      │ BeanShell logs │
│ CORR-08  │ PRODUCT_ID            │ Product detail body     │ 06 POST Cart   │
│ CORR-09  │ PRODUCT_NAME          │ Product detail body     │ BeanShell logs │
│ CORR-10  │ CART_SUBTOTAL         │ Add-to-Cart JSON        │ BeanShell logs │
│ CORR-11  │ CART_TOKEN            │ Cart page body          │ checkout POST  │
│ CORR-12  │ SEARCH_RESULT_matchNr │ Search results body     │ BeanShell logs │
└──────────┴───────────────────────┴─────────────────────────┴────────────────┘
```
