# 📘 Apache JMeter Setup Guide

This README walks you through downloading, installing, and running **Apache JMeter** from scratch.
It’s written for **freshers / beginners** and contains official documentation links for deeper learning.

---

## 🧾 Table of Contents

1. What is Apache JMeter?
2. System Requirements
3. Pre-requisites
4. Step-by-Step Installation
5. First Time Running JMeter
6. Official Resources & Documentation
7. Troubleshooting
8. Useful Tips

---

## 🧠 1. What is Apache JMeter?

Apache JMeter is an open-source tool used for **performance testing** and **load testing** web applications, APIs, databases, and more.
It simulates multiple users sending requests to servers to measure performance.

Official Site: [https://jmeter.apache.org/](https://jmeter.apache.org/)

---

## 💻 2. System Requirements

Before installing JMeter, make sure your system meets the following:

✔️ **Operating System** – Windows / macOS / Linux
✔️ **Java Runtime Environment (JRE)** installed (version 8 or above)

---

## ☑️ 3. Pre-requisites

Apache JMeter runs on Java, so you must install Java first.

### 📌 A. Check if Java is Installed

Open your terminal or command prompt and run:

```bash
java -version
```

If Java is installed, you will see output like:

```
java version "1.8.0_281"
```

If Java is **not installed** or version is below 8, follow the next step.

---

### 📌 B. Install Java (if not installed)

#### ✅ For Windows

1. Go to official Java download page:
   [https://www.oracle.com/java/technologies/javase/jdk11-archivedownloads.html](https://www.oracle.com/java/technologies/javase/jdk11-archivedownloads.html)

2. Download the **JDK** installer for Windows.

3. Run the installer and follow the steps.

4. Set `JAVA_HOME` in Environment Variables.

---

#### ✅ For macOS

You can install Java via Homebrew:

```bash
brew install openjdk@17
```

Then add it to your path:

```bash
export JAVA_HOME="/usr/local/opt/openjdk@17"
```

---

#### ✅ For Linux (Ubuntu Example)

```bash
sudo apt update
sudo apt install openjdk-17-jdk
```

---

## 📥 4. Step-by-Step Installation of Apache JMeter

Follow these steps:

---

### 📌 Step-1: Go to JMeter Download Page

Official download page:
[https://jmeter.apache.org/download_jmeter.cgi](https://jmeter.apache.org/download_jmeter.cgi)

---

### 📌 Step-2: Download Binary

Under “**Binaries**” section click the ZIP or TGZ link:

Look for:
👉 **apache-jmeter-X.X.zip** (Windows)
👉 **apache-jmeter-X.X.tgz** (macOS / Linux)

---

### 📌 Step-3: Extract the Archive

After downloading:

✔️ Windows – Right-click ZIP ➜ Extract
✔️ macOS / Linux – Use terminal:

```bash
tar -xzf apache-jmeter-X.X.tgz
```

This creates a folder named like:

```
apache-jmeter-X.X
```

---

### 📌 Step-4: Run JMeter

#### 🔹 On Windows

Double-click:

```
apache-jmeter-X.X\bin\jmeter.bat
```

#### 🔹 On macOS / Linux

Open terminal and run:

```bash
cd apache-jmeter-X.X/bin
./jmeter
```

---

🎉 If everything works correctly, the JMeter GUI should open!

---

## ▶️ 5. Your First Test in JMeter

Once JMeter opens:

1. **File > New** – Create a new test plan
2. **Right-click Test Plan > Add > Threads (Users) > Thread Group**
3. **Right-click Thread Group > Add > Sampler > HTTP Request**
   • Put a server name like `example.com`
4. **Right-click Thread Group > Add > Listener > View Results Tree**
5. **Click the green ▶ (Start)**

You will see the results in the listener!

---

## 📚 6. Official Documentation & Helpful Links

| Resource                    | Link                                                                                               |
| --------------------------- | -------------------------------------------------------------------------------------------------- |
| Apache JMeter Official Site | [https://jmeter.apache.org/](https://jmeter.apache.org/)                                           |
| JMeter User Manual          | [https://jmeter.apache.org/usermanual/index.html](https://jmeter.apache.org/usermanual/index.html) |
| JMeter Download Page        | [https://jmeter.apache.org/download_jmeter.cgi](https://jmeter.apache.org/download_jmeter.cgi)     |
| JMeter FAQ                  | [https://jmeter.apache.org/faq.html](https://jmeter.apache.org/faq.html)                           |
| JMeter Plugins              | [https://jmeter-plugins.org/](https://jmeter-plugins.org/)                                         |

👉 Official Getting Started Guide:
[https://jmeter.apache.org/usermanual/get-started.html](https://jmeter.apache.org/usermanual/get-started.html)

---

## 🛠 7. Troubleshooting Common Issues

### ❗ Java Not Found Error

Fix → Set Java environment variables:

#### Windows

```
JAVA_HOME = C:\Program Files\Java\jdk-17
```

Add to PATH:

```
%JAVA_HOME%\bin
```

---

### ❗ JMeter GUI Not Opening

✔️ Check if Java is installed
✔️ Use correct JMeter folder
✔️ Try running from terminal to see errors

---

## 💡 8. Useful Tips for Beginners

✅ Learn core components:
✔️ Thread Group
✔️ Samplers
✔️ Listeners
✔️ Controllers

✅ Save your Test Plan often (`.jmx` file)
✅ Use “View Results Tree” carefully (heavy memory usage)
✅ Use non-GUI mode for large tests:

```
jmeter -n -t testplan.jmx -l result.jtl
```

---

## 🎯 Summary

You now know:

✔️ How to install Java
✔️ How to download JMeter
✔️ How to run JMeter first time
✔️ Where to find official documentation

---