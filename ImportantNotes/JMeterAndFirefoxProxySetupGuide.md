# 📘 Apache JMeter + Firefox Proxy Setup Guide (HTTP & HTTPS)

## 📌 Objective

This guide explains step-by-step:

* How to configure Apache JMeter
* How to configure Firefox Manual Proxy
* How to record HTTP traffic
* How to record HTTPS traffic
* Which fields to fill
* Which fields to keep blank
* How to install SSL certificate for HTTPS

This guide is written in **simple language for beginners**.

---

# 🖥️ System Setup

* OS: Windows
* Browser: Firefox
* Tool: Apache JMeter
* Proxy Port: 8888
* JMeter and Firefox running on the **same machine**

---

# 🔹 PART 1: JMeter Configuration

---

## Step 1: Open JMeter

1. Go to JMeter `bin` folder
2. Double click:

```
jmeter.bat
```

---

## Step 2: Add HTTP(S) Test Script Recorder

1. Right click on **Test Plan**
2. Click:

```
Add → Non-Test Elements → HTTP(S) Test Script Recorder
```

---

## Step 3: Configure HTTP(S) Test Script Recorder

Now configure these settings:

### 🔹 Global Settings

| Field             | Value                | Notes                         |
| ----------------- | -------------------- | ----------------------------- |
| Port              | 8888                 | Must match Firefox proxy port |
| Target Controller | Recording Controller | Create if not present         |

Leave everything else as default.

---

## Step 4: Create Recording Controller (If Not Present)

1. Right click on **Test Plan**
2. Add → Threads (Users) → Thread Group
3. Right click **Thread Group**
4. Add → Logic Controller → Recording Controller

---

## Step 5: Start the Proxy

Click:

```
Start
```

If successful, JMeter will show:

```
Proxy started...
```

---

# 🔹 PART 2: Firefox Manual Proxy Configuration (for JMeter on same machine)

Now we configure Firefox to send traffic to JMeter.

---

## Step 1: Open Firefox Network Settings

1. Open Firefox
2. Go to:

```
Settings → Network Settings → Settings
```

---

## Step 2: Select Manual Proxy Configuration

Select:

```
Manual proxy configuration
```

Now fill fields as below:

---

# ✅ Firefox Manual Proxy Configuration (for JMeter on same machine)

| Setting                           | Value           | Explanation                      |
| --------------------------------- | --------------- | -------------------------------- |
| HTTP Proxy                        | 127.0.0.1       | Local machine                    |
| Port                              | 8888            | Same as JMeter                   |
| ☑ Also use this proxy for HTTPS   | ✅ CHECKED       | Required for HTTPS recording     |
| HTTPS Proxy                       | Leave Blank     | Auto filled if checkbox selected |
| Port (HTTPS)                      | Leave Blank     | Not needed                       |
| SOCKS Host                        | Leave Blank     | Not needed                       |
| SOCKS Port                        | Leave Blank     | Not needed                       |
| SOCKS v4                          | ❌ Do not select | Not required                     |
| SOCKS v5                          | ❌ Do not select | Not required                     |
| Automatic proxy configuration URL | Leave Blank     | Not required                     |

Click:

```
OK
```

---

# 🔹 PART 3: Recording HTTP Traffic

If the website starts with:

```
http://
```

Then:

✅ No certificate required
✅ JMeter will record immediately

Steps:

1. Start Proxy in JMeter
2. Browse website in Firefox
3. Requests will appear inside Recording Controller

---

# 🔹 PART 4: Recording HTTPS Traffic (IMPORTANT)

If website starts with:

```
https://
```

Then you MUST install JMeter SSL certificate.

Otherwise you will get:

```
SSL Error
Secure Connection Failed
```

---

## Step 1: Start JMeter Proxy

Click:

```
Start
```

---

## Step 2: Download Certificate

Open Firefox and go to:

```
http://127.0.0.1:8888
```

Download:

```
ApacheJMeterTemporaryRootCA.crt
```

---

## Step 3: Install Certificate in Firefox

Go to:

```
Settings → Privacy & Security → Certificates → View Certificates
```

Click:

```
Authorities → Import
```

Select:

```
ApacheJMeterTemporaryRootCA.crt
```

Check:

```
☑ Trust this CA to identify websites
```

Click:

```
OK
```

---

# 🔹 PART 5: Verify Recording

Now:

1. Start Proxy in JMeter
2. Open any HTTPS website in Firefox
3. Requests should appear in Recording Controller

If working correctly:

✅ No SSL error
✅ Requests visible in JMeter

---

# 🔹 PART 6: After Recording (Very Important)

After recording, remove proxy from Firefox.

Go to:

```
Settings → Network Settings
```

Select:

```
No Proxy
```

Otherwise internet will not work when JMeter is closed.

---

# 🔎 Common Problems & Solutions

---

## ❌ Problem: Nothing is recording

Check:

* Is proxy started?
* Is port 8888 same in both?
* Is firewall blocking port?
* Is another app using 8888?

---

## ❌ Problem: SSL Error

Solution:

* Install JMeter certificate properly
* Restart Firefox
* Make sure certificate is in Authorities section

---

## ❌ Problem: Internet not working after closing JMeter

Solution:

Set Firefox back to:

```
No Proxy
```

---

# 🔹 Quick Final Checklist

✅ JMeter opened
✅ HTTP(S) Test Script Recorder added
✅ Port set to 8888
✅ Firefox proxy set to 127.0.0.1:8888
✅ “Also use this proxy for HTTPS” checked
✅ Certificate installed (for HTTPS)
✅ Proxy removed after recording

---

# 🎯 Summary (In Simple Words)

* JMeter acts like a middleman.
* Firefox sends traffic to JMeter.
* JMeter records the requests.
* For HTTPS, certificate is required.
* After recording, remove proxy.
