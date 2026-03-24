# JMeter Script Guide: FakeStoreAPI Login with Data-Driven Testing

## Overview

This guide walks you through recording and configuring a JMeter test script for the **FakeStoreAPI Login** endpoint using:
- **Regular Expression Extractor** – to capture dynamic tokens from responses
- **Data-Driven Testing** – feeding login credentials from an external **CSV file** and **JSON file**

---

## API Under Test

| Property | Details |
|---|---|
| **API Name** | FakeStoreAPI Login |
| **URL** | `https://fakestoreapi.com/auth/login` |
| **Method** | `POST` |
| **Content-Type** | `application/json` |

**Sample Request Body:**
```json
{
  "username": "mor_2314",
  "password": "83r5^_"
}
```

**Sample Response:**
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

---

## Prerequisites

- Apache JMeter **5.6+** installed → [Download here](https://jmeter.apache.org/download_jmeter.cgi)
- Java **JDK 8 or 11+** installed and `JAVA_HOME` configured
- Internet access to reach `https://fakestoreapi.com`
- A text editor (VS Code, Notepad++, etc.) to prepare data files

---

## Part 1 – Prepare External Data Files

### 1.1 Create the CSV File

Create a file named **`login_data.csv`** in a known folder (e.g., `C:/JMeter_TestData/`):

```
username,password
mor_2314,83r5^_
johnd,m38rmF$
kevinryan,kev02937@
donero,ewedon
derek,jklg*_56
david_r,3478*#54
snyder,f238*#4
kate_h,I_Hunting_77
Melodia45,25cQqA1p6
```

> **Rules for the CSV file:**
> - First row must be the **header row** (column names).
> - No spaces around the commas.
> - Save with **UTF-8** encoding.
> - Each row represents one unique login attempt.

---

### 1.2 Create the JSON File

Create a file named **`login_data.json`** in the same folder:

```json
[
  { "username": "mor_2314",  "password": "83r5^_"     },
  { "username": "johnd",     "password": "m38rmF$"    },
  { "username": "kevinryan", "password": "kev02937@"  },
  { "username": "donero",    "password": "ewedon"      },
  { "username": "derek",     "password": "jklg*_56"   },
  { "username": "david_r",   "password": "3478*#54"   },
  { "username": "snyder",    "password": "f238*#4"    },
  { "username": "kate_h",    "password": "I_Hunting_77"},
  { "username": "Melodia45", "password": "25cQqA1p6"  }
]
```

> **Note:** JMeter does not natively read JSON files for parameterization. A JSR223 PreProcessor (Groovy script) will be used to read this file — covered in Part 4.

---

## Part 2 – Set Up the JMeter Test Plan

### Step 1 – Launch JMeter

Open a terminal/command prompt and run:

```bash
# Windows
C:\apache-jmeter-5.6\bin\jmeter.bat

# macOS/Linux
/opt/apache-jmeter-5.6/bin/jmeter.sh
```

---

### Step 2 – Create a New Test Plan

1. JMeter opens with a default **Test Plan** node in the left panel.
2. Click on **Test Plan**, and rename it to:
   ```
   FakeStoreAPI - Login Data-Driven Test
   ```
3. Go to **File → Save Test Plan As** and save as `FakeStoreAPI_Login.jmx`.

---

### Step 3 – Add a Thread Group

A Thread Group defines how many virtual users (threads) will run the test.

1. Right-click **Test Plan** → **Add** → **Threads (Users)** → **Thread Group**
2. Configure the Thread Group:

| Setting | Value | Description |
|---|---|---|
| **Name** | `Login Thread Group` | Label for this group |
| **Number of Threads (users)** | `9` | One per row in the data file |
| **Ramp-Up Period (seconds)** | `5` | Time to start all threads |
| **Loop Count** | `1` | Each user runs once |

> Set **Number of Threads** to match the number of data rows in your CSV/JSON file.

---

## Part 3 – Data-Driven Testing Using CSV File

### Step 4 – Add a CSV Data Set Config

This element reads the CSV file and feeds each row's values to JMeter variables.

1. Right-click **Login Thread Group** → **Add** → **Config Element** → **CSV Data Set Config**
2. Configure it as follows:

| Property | Value |
|---|---|
| **Name** | `CSV Login Data` |
| **Filename** | `C:/JMeter_TestData/login_data.csv` |
| **Variable Names** | `username,password` |
| **Ignore first line** | `True` (because row 1 is the header) |
| **Delimiter** | `,` |
| **Allow quoted data?** | `False` |
| **Recycle on EOF?** | `False` |
| **Stop thread on EOF?** | `True` |
| **Sharing mode** | `All Threads` |

> JMeter will now expose `${username}` and `${password}` as variables for each virtual user.

---

### Step 5 – Add an HTTP Request Sampler (CSV Version)

1. Right-click **Login Thread Group** → **Add** → **Sampler** → **HTTP Request**
2. Configure it:

| Property | Value |
|---|---|
| **Name** | `POST - Login (CSV)` |
| **Protocol** | `https` |
| **Server Name or IP** | `fakestoreapi.com` |
| **HTTP Method** | `POST` |
| **Path** | `/auth/login` |

3. In the **Body Data** tab, enter:

```json
{
  "username": "${username}",
  "password": "${password}"
}
```

4. Click **Add** under the **HTTP Header Manager** (or add one as below):

---

### Step 6 – Add HTTP Header Manager

1. Right-click **POST - Login (CSV)** → **Add** → **Config Element** → **HTTP Header Manager**
2. Click **Add** and enter:

| Name | Value |
|---|---|
| `Content-Type` | `application/json` |

---

## Part 4 – Data-Driven Testing Using JSON File

This approach reads the JSON file at runtime using a **JSR223 PreProcessor**.

### Step 7 – Add a Second HTTP Request Sampler (JSON Version)

1. Right-click **Login Thread Group** → **Add** → **Sampler** → **HTTP Request**
2. Configure it exactly like Step 5, but name it:
   ```
   POST - Login (JSON)
   ```
3. In the **Body Data** tab enter:

```json
{
  "username": "${json_username}",
  "password": "${json_password}"
}
```

---

### Step 8 – Add JSR223 PreProcessor to Read JSON File

1. Right-click **POST - Login (JSON)** → **Add** → **Pre Processors** → **JSR223 PreProcessor**
2. Set **Language** to `groovy`
3. Paste the following Groovy script in the **Script** area:

```groovy
import groovy.json.JsonSlurper

// Path to your JSON file
def filePath = "C:/JMeter_TestData/login_data.json"

// Read and parse the JSON file
def jsonContent = new File(filePath).text
def jsonSlurper = new JsonSlurper()
def userList = jsonSlurper.parseText(jsonContent)

// Get the current thread number (0-indexed)
int threadIndex = (ctx.getThreadNum()) % userList.size()

// Extract credentials for this thread
def currentUser = userList[threadIndex]

// Set JMeter variables
vars.put("json_username", currentUser.username)
vars.put("json_password", currentUser.password)

log.info("Thread ${ctx.getThreadNum()} using username: ${currentUser.username}")
```

> This script assigns a unique JSON entry to each thread based on its thread index. The `${json_username}` and `${json_password}` variables are now available to the HTTP Request sampler.

---

## Part 5 – Add Regular Expression Extractor

The Regular Expression Extractor captures the **token** from the login response and stores it in a JMeter variable for use in subsequent requests.

### Step 9 – Add Regular Expression Extractor

1. Right-click **POST - Login (CSV)** → **Add** → **Post Processors** → **Regular Expression Extractor**
2. Configure it as follows:

| Property | Value | Description |
|---|---|---|
| **Name** | `Extract Auth Token` | Label |
| **Apply to** | `Main sample only` | Scope |
| **Field to check** | `Body` | Checks the response body |
| **Reference Name** | `auth_token` | Variable name to store extracted value |
| **Regular Expression** | `"token"\s*:\s*"([^"]+)"` | Captures the token value |
| **Template** | `$1$` | `$1$` means the first capturing group |
| **Match No.** | `1` | Extract the first match |
| **Default Value** | `TOKEN_NOT_FOUND` | Fallback if extraction fails |

> **Regex Breakdown:**
> - `"token"` → Matches the literal key
> - `\s*:\s*` → Matches colon with optional spaces
> - `"([^"]+)"` → Captures everything inside the quotes after the colon
>
> After extraction, use `${auth_token}` in any subsequent HTTP Request (e.g., Authorization header).

---

### Step 10 – Repeat Regular Expression Extractor for JSON Sampler

Repeat Step 9 by right-clicking **POST - Login (JSON)** and adding the same **Regular Expression Extractor** with:
- **Reference Name:** `auth_token_json`
- All other settings identical to Step 9

---

## Part 6 – Add Listeners to View Results

### Step 11 – Add Listeners

Right-click **Login Thread Group** → **Add** → **Listener** and add the following:

1. **View Results Tree**
   - Shows each request and response in detail.
   - Use for debugging during development.

2. **Summary Report**
   - Shows aggregate statistics (throughput, error %, avg response time).

3. **Aggregate Report**
   - Similar to Summary Report but with percentile data.

> **Tip:** Disable listeners when running performance tests in bulk — they consume memory and slow down JMeter.

---

## Part 7 – Full Test Plan Structure

After completing all steps, your JMeter Test Plan tree should look like this:

```
📁 Test Plan: FakeStoreAPI - Login Data-Driven Test
 └── 📁 Thread Group: Login Thread Group
      ├── ⚙️  CSV Data Set Config: CSV Login Data
      │
      ├── 🌐 HTTP Request: POST - Login (CSV)
      │    ├── ⚙️  HTTP Header Manager
      │    └── 🔍 Regular Expression Extractor: Extract Auth Token
      │
      ├── 🌐 HTTP Request: POST - Login (JSON)
      │    ├── 🔧 JSR223 PreProcessor: Read JSON Credentials
      │    ├── ⚙️  HTTP Header Manager
      │    └── 🔍 Regular Expression Extractor: Extract Auth Token (JSON)
      │
      ├── 📊 Listener: View Results Tree
      ├── 📊 Listener: Summary Report
      └── 📊 Listener: Aggregate Report
```

---

## Part 8 – Recording the Script Using JMeter Proxy (Optional)

If you prefer to **record** the API call instead of manually creating the HTTP Request sampler, follow these steps:

### Step 12 – Configure JMeter HTTP(S) Test Script Recorder

1. Right-click **Test Plan** → **Add** → **Non-Test Elements** → **HTTP(S) Test Script Recorder**
2. Set **Port** to `8888`
3. Set **Target Controller** to your **Login Thread Group**
4. Click **Start** — JMeter will prompt you to install a certificate (accept it)

### Step 13 – Configure Your Browser/Tool as Proxy

In your browser (e.g., Firefox) or API tool (Postman):
- Set **HTTP Proxy** to `localhost` and **Port** to `8888`

### Step 14 – Send the Login Request

Using Postman or curl, send:

```bash
curl -x http://localhost:8888 \
  -X POST https://fakestoreapi.com/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username": "mor_2314", "password": "83r5^_"}'
```

JMeter will capture this request and add it as an HTTP Request sampler automatically.

### Step 15 – Stop the Recording

Click **Stop** in the HTTP(S) Test Script Recorder. The recorded request will appear in your Thread Group.

> After recording, manually replace hardcoded values with `${username}` and `${password}` variables as described in Step 5.

---

## Part 9 – Run the Test

### Step 16 – Save and Execute

1. Press `Ctrl + S` to save the test plan.
2. Click the **green Play button (▶)** in the toolbar or go to **Run → Start**.
3. Watch the **View Results Tree** to verify each request uses a different username/password.

### Step 17 – Verify Results

In **View Results Tree**, for each request verify:

- ✅ **Request Tab:** Confirm `username` and `password` values are different per thread
- ✅ **Response Tab:** Should return HTTP **200 OK** with a `token` in the JSON body
- ✅ **RegEx Extractor:** Check that `auth_token` variable is populated (non-empty, not `TOKEN_NOT_FOUND`)

---

## Part 10 – Running from Command Line (Non-GUI Mode)

For actual performance testing, always run JMeter in **non-GUI mode**:

```bash
jmeter -n \
  -t C:/JMeter_Scripts/FakeStoreAPI_Login.jmx \
  -l C:/JMeter_Results/results.jtl \
  -e \
  -o C:/JMeter_Results/HTMLReport/
```

| Flag | Description |
|---|---|
| `-n` | Non-GUI (headless) mode |
| `-t` | Path to your `.jmx` test plan |
| `-l` | Path to save raw results `.jtl` file |
| `-e` | Generate HTML report after test |
| `-o` | Output folder for the HTML report |

---

## Common Issues & Troubleshooting

| Issue | Likely Cause | Fix |
|---|---|---|
| `TOKEN_NOT_FOUND` in variable | Regex doesn't match response body | Check response body in **View Results Tree** and adjust regex |
| All threads use same credentials | Sharing mode is wrong | Set CSV sharing mode to `All Threads` |
| `Connection refused` error | Proxy not configured correctly | Ensure browser/curl uses `localhost:8888` as proxy |
| `SSLHandshakeException` | JMeter SSL cert not installed | Install the JMeter certificate in your browser/system trust store |
| JSON file not found | Wrong file path | Use absolute path; avoid backslashes on Mac/Linux — use `/` |
| `groovy.lang.MissingMethodException` | Groovy script error | Check syntax in JSR223 PreProcessor; verify JSON file structure |

---

## Quick Reference – JMeter Variables

| Variable | Source | Usage |
|---|---|---|
| `${username}` | CSV Data Set Config | Username from CSV row |
| `${password}` | CSV Data Set Config | Password from CSV row |
| `${json_username}` | JSR223 PreProcessor (Groovy) | Username from JSON entry |
| `${json_password}` | JSR223 PreProcessor (Groovy) | Password from JSON entry |
| `${auth_token}` | Regular Expression Extractor (CSV sampler) | JWT token from login response |
| `${auth_token_json}` | Regular Expression Extractor (JSON sampler) | JWT token from JSON login response |

---

## Summary

| Step | Action |
|---|---|
| 1 | Prepared `login_data.csv` with 9 user credentials |
| 2 | Prepared `login_data.json` with the same credentials |
| 3 | Created Thread Group with 9 virtual users |
| 4 | Added CSV Data Set Config for CSV-driven testing |
| 5 | Added HTTP Request sampler using `${username}` and `${password}` |
| 6 | Added JSR223 PreProcessor to read JSON file using Groovy |
| 7 | Added second HTTP Request sampler using JSON-derived variables |
| 8 | Added Regular Expression Extractor to capture auth token |
| 9 | Added listeners to view and validate results |
| 10 | Executed test and verified unique credentials per thread |

---

*Guide prepared for Apache JMeter 5.6 | FakeStoreAPI v1 | Last updated: March 2026*
