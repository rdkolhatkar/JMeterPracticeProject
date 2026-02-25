# 📘 COMPLETE README

# BlazeMeter Chrome Setup + Recording + Troubleshooting Guide

*(Beginner Friendly – Step-by-Step Guide)*

---

# 📌 Objective

In this guide, we will:

* Install BlazeMeter Chrome Extension
* Create a BlazeMeter Account
* Record UI actions from [https://blazedemo.com/](https://blazedemo.com/)
* Generate and Download `.jmx` file
* Import `.jmx` into **Apache JMeter**
* Fix common recording issues

This document is written in **very simple language** so freshers can follow easily.

---

# 🔹 PART 1: Install BlazeMeter Chrome Extension

## Step 1: Open Chrome Browser

Open **Google Chrome**.

---

## Step 2: Search for BlazeMeter Plugin

In Chrome search bar type:

```
BlazeMeter plugin
```

Press Enter.

---

## Step 3: Open Chrome Web Store

Click the first link which opens extension in:

👉 **Chrome Web Store**

---

## Step 4: Install Extension

1. Click **Add to Chrome**
2. Click **Add Extension**
3. Wait for installation
4. BlazeMeter icon will appear at top right

---

# 🔹 PART 2: Create BlazeMeter Account

## Step 5: Sign Up

1. Click BlazeMeter icon
2. Click **Sign Up**
3. Enter:

   * First Name
   * Last Name
   * Email
   * Password
4. Complete registration
5. You will be redirected to BlazeMeter Home Page

---

# 🔹 PART 3: Create a New Test

## Step 6: Choose Test Type

On Home Page:

* Click **Tests**
* Click **Create New Test**

You will see:

👉 **Select a Test Type**

---

## Step 7: Select Recorder

Under **Test Creation Tools**:

* Click **Recorder**

You will now go to Recorder tab.

---

# 🔹 PART 4: Configure Recorder

## Step 8: Enable Full Response

Inside Recorder:

✅ Check **Record full response body**

This ensures complete request/response capture.

---

## Step 9: Create Proxy

Click:

👉 **Create Proxy**

A new Proxy tab will open.

⚠ Do NOT close this tab.

---

# 🔹 PART 5: Start Recording

## Step 10: Start Proxy Recording

In Proxy tab:

🔴 Click the RED button

You will see:

👉 “Recording…”

Recording has started.

---

# 🔹 PART 6: Record BlazeDemo Website

We will record:

👉 [https://blazedemo.com/](https://blazedemo.com/)

---

## Step 11: Open Website

Open new tab and enter:

```
https://blazedemo.com/
```

Press Enter.

---

## Step 12: Perform UI Actions

### Action 1: Search Flights

* Select Departure: Boston
* Select Destination: London
* Click **Find Flights**

---

### Action 2: Choose Flight

* Click **Choose This Flight**

---

### Action 3: Fill Passenger Details

Enter:

* Name
* Address
* City
* State
* Zip Code
* Credit Card
* Name on Card

Click:

👉 **Purchase Flight**

---

### Action 4: Confirmation Page

Confirmation page loads.

Recording is capturing:

* All HTTP Requests
* All Backend API Calls
* Headers
* Cookies

---

# 🔹 PART 7: Stop Recording

## Step 13: Stop Recorder

Go back to Proxy tab.

Click RED button again.

Recording stops.

---

# 🔹 PART 8: Save and Download .jmx File

## Step 14: Save Test

1. Click Save
2. Enter Test Name
3. Click Save Test

BlazeMeter automatically generates `.jmx` file.

---

## Step 15: Download .jmx

1. Open saved test
2. Click **Download JMX**
3. File downloads to system

---

# 🔹 PART 9: Import into JMeter

Open:

👉 **Apache JMeter**

### Import File

1. File → Open
2. Select downloaded `.jmx`
3. Click Open

Your recorded script is now visible.

---

# 🔹 HOW RECORDING WORKS (Important Concept)

BlazeMeter:

* Acts as Proxy
* Captures HTTP/HTTPS traffic
* Converts traffic into JMeter structure:

  * Thread Group
  * HTTP Samplers
  * Listeners
  * Headers
  * Cookies

Then it exports `.jmx` file.

---

# 🔹 TROUBLESHOOTING SECTION

# (If Recording Is Not Visible)

If you are unable to see recording, follow ALL below steps carefully.

---

# ✅ Check 1: Is RED Button Active?

In Recorder tab:

* RED button should show “Recording…”

If not:

👉 Click RED button again.

---

# ✅ Check 2: Is Proxy Tab Open?

When you clicked Create Proxy:

* A new tab opened
* That tab must remain open

⚠ If you close proxy tab → Recording stops.

---

# ✅ Check 3: Disable VPN

BlazeMeter proxy will not work if VPN is active.

Turn OFF:

* System VPN
* Browser VPN extension

Restart Chrome and try again.

---

# ✅ Check 4: Install BlazeMeter Certificate (For HTTPS)

Since [https://blazedemo.com/](https://blazedemo.com/) is HTTPS, certificate must be trusted.

---

## How to Install Certificate (Windows)

1. Start Recorder
2. Download BlazeMeter certificate
3. Double click certificate
4. Click Install Certificate
5. Select:
   👉 Local Machine
6. Choose:
   👉 Place all certificates in the following store
7. Select:
   👉 Trusted Root Certification Authorities
8. Click Finish
9. Restart Chrome

---

# ✅ Check 5: Clear Cache

Press:

```
Ctrl + Shift + Delete
```

Clear:

* Cookies
* Cache

Restart Chrome.

---

# ✅ Check 6: Disable Other Extensions

Go to:

```
chrome://extensions/
```

Disable:

* AdBlock
* Security Extensions
* VPN Extensions

Try recording again.

---

# ✅ Check 7: Check Proxy Settings

In Chrome address bar type:

```
chrome://settings/system
```

Click:

👉 Open your computer’s proxy settings

Make sure:

* No manual proxy configured
* No other proxy running

---

# ✅ Check 8: Try Incognito Mode

1. Open Incognito window
2. Enable BlazeMeter for Incognito
3. Start recording
4. Open website again

---

# ✅ Check 9: Restart Everything

If still not working:

1. Stop recording
2. Close Chrome fully
3. Reopen Chrome
4. Start new recording

---

# ✅ Check 10: Reinstall Extension

1. Remove BlazeMeter extension
2. Restart Chrome
3. Install again from:

👉 **Chrome Web Store**

4. Login again
5. Start recording

---

# 🔹 BEST PRACTICES FOR FRESHERS

✔ Start recording BEFORE opening website
✔ Record one scenario at a time
✔ Don’t open unnecessary tabs
✔ Stop recording immediately after scenario
✔ Save test properly
✔ Always verify requests are visible while recording

---

# 🎯 Final Checklist

| Step                   | Status |
| ---------------------- | ------ |
| Extension Installed    | ✅      |
| Account Created        | ✅      |
| Recorder Selected      | ✅      |
| Proxy Created          | ✅      |
| RED Button Active      | ✅      |
| Certificate Installed  | ✅      |
| VPN Disabled           | ✅      |
| Requests Visible       | ✅      |
| Test Saved             | ✅      |
| JMX Downloaded         | ✅      |
| JMX Imported in JMeter | ✅      |

---

# 🎉 Congratulations!

You have successfully:

* Installed BlazeMeter
* Recorded UI actions
* Generated `.jmx` file
* Imported into **Apache JMeter**
* Learned how to fix recording issues

---