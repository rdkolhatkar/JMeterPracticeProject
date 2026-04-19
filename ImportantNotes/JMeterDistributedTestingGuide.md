# 🚀 JMeter Distributed (Master-Slave) Testing — Complete Guide

> A step-by-step guide for freshers to set up and run JMeter in **Distributed Mode** across Windows, macOS, and Linux machines.

---

## 📖 Table of Contents

1. [What is JMeter Distributed Testing?](#1-what-is-jmeter-distributed-testing)
2. [Architecture Overview](#2-architecture-overview)
3. [Prerequisites](#3-prerequisites)
4. [Network & Firewall Requirements](#4-network--firewall-requirements)
5. [Java Installation (All Machines)](#5-java-installation-all-machines)
6. [JMeter Installation (All Machines)](#6-jmeter-installation-all-machines)
7. [Slave Machine Configuration](#7-slave-machine-configuration)
   - [Windows Slave Setup](#71-windows-slave-setup)
   - [macOS Slave Setup](#72-macos-slave-setup)
   - [Linux Slave Setup](#73-linux-slave-setup)
8. [Master Machine Configuration](#8-master-machine-configuration)
9. [JMeter Properties Configuration (Master & Slave)](#9-jmeter-properties-configuration-master--slave)
10. [Running Distributed Tests](#10-running-distributed-tests)
11. [Collecting Results on Master](#11-collecting-results-on-master)
12. [Troubleshooting Common Issues](#12-troubleshooting-common-issues)
13. [Quick Reference Cheat Sheet](#13-quick-reference-cheat-sheet)

---

## 1. What is JMeter Distributed Testing?

When you run a **single JMeter instance**, there is a limit to how many virtual users (threads) your machine can generate before it runs out of CPU or memory. **Distributed Testing** solves this by using **multiple machines** to generate load together.

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│   WHY DISTRIBUTED TESTING?                                      │
│                                                                 │
│   Single Machine  →  500 users  ──►  Limited Load              │
│                                                                 │
│   Distributed     →  5 Machines × 500 users = 2500 users ──►   │
│                       Massive Realistic Load!                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Concepts:

| Term       | Role                                                                 |
|------------|----------------------------------------------------------------------|
| **Master** | The controller machine. You create and run test plans from here.     |
| **Slave**  | Worker machines. They receive instructions and generate the actual load. |
| **RMI**    | Remote Method Invocation — Java protocol used for Master-Slave communication. |

---

## 2. Architecture Overview

```
╔══════════════════════════════════════════════════════════════════════════╗
║                    JMETER DISTRIBUTED ARCHITECTURE                       ║
╠══════════════════════════════════════════════════════════════════════════╣
║                                                                          ║
║   ┌─────────────────────────────────────────────┐                       ║
║   │           MASTER MACHINE (Controller)        │                       ║
║   │                                              │                       ║
║   │   • Creates Test Plan (.jmx)                 │                       ║
║   │   • Sends test to all Slaves                 │                       ║
║   │   • Collects & aggregates results            │                       ║
║   │   • JMeter GUI runs here                     │                       ║
║   │                                              │                       ║
║   │    [IP: 192.168.1.100]                       │                       ║
║   └──────────────────┬──────────────────────────┘                       ║
║                      │  TCP Port 1099 (RMI)                              ║
║          ┌───────────┼────────────┐                                      ║
║          │           │            │                                       ║
║          ▼           ▼            ▼                                       ║
║   ┌──────────┐ ┌──────────┐ ┌──────────┐                                ║
║   │  SLAVE 1 │ │  SLAVE 2 │ │  SLAVE 3 │                                ║
║   │          │ │          │ │          │                                ║
║   │ Windows  │ │  macOS   │ │  Linux   │                                ║
║   │          │ │          │ │          │                                ║
║   │192.168   │ │192.168   │ │192.168   │                                ║
║   │.1.101    │ │.1.102    │ │.1.103    │                                ║
║   └────┬─────┘ └────┬─────┘ └────┬─────┘                               ║
║        │             │             │                                      ║
║        └─────────────┴─────────────┘                                    ║
║                       │                                                  ║
║                       ▼                                                  ║
║             ┌──────────────────┐                                         ║
║             │   TARGET SERVER  │                                         ║
║             │  (Your Web App)  │                                         ║
║             │  e.g. myapp.com  │                                         ║
║             └──────────────────┘                                         ║
║                                                                          ║
╚══════════════════════════════════════════════════════════════════════════╝
```

### How the Flow Works (Step by Step):

```
STEP 1: Master sends Test Plan ──────────────────────────────────────────►
         Master ──────[JMX file + config]──────► Slave 1, Slave 2, Slave N

STEP 2: Each Slave executes the test independently ──────────────────────►
         Slave 1 ──[500 users]──► Target Server
         Slave 2 ──[500 users]──► Target Server
         Slave 3 ──[500 users]──► Target Server

STEP 3: Slaves send results back to Master ──────────────────────────────►
         Slave 1 ──[Results]──┐
         Slave 2 ──[Results]──┼──► Master (Aggregates everything)
         Slave 3 ──[Results]──┘
```

---

## 3. Prerequisites

Before you start, make sure every machine (Master + all Slaves) meets the following requirements:

### ✅ Checklist for ALL Machines

| Requirement | Details |
|---|---|
| **Java** | JDK 8 or higher (JDK 11 recommended) |
| **JMeter** | Same version on ALL machines (very important!) |
| **OS** | Windows 10/11, macOS 10.13+, or Linux (Ubuntu/CentOS/RHEL) |
| **RAM** | Minimum 4 GB (8 GB recommended per Slave) |
| **Network** | All machines must be on the **same network** or VPN |
| **Firewall** | Port 1099 (RMI) must be open on Slave machines |
| **Same JMeter version** | Master and all Slaves MUST have identical JMeter versions |
| **Same Plugin versions** | All plugins must match between Master and Slaves |

> ⚠️ **IMPORTANT FOR FRESHERS:** The single most common mistake is having **different JMeter versions** on Master and Slave. Always use the exact same version everywhere!

---

## 4. Network & Firewall Requirements

```
╔══════════════════════════════════════════════════════════╗
║              PORTS THAT MUST BE OPEN                     ║
╠══════════════════════════════════════════════════════════╣
║                                                          ║
║   Master ──► Slave   :  Port 1099  (RMI Registry)       ║
║   Master ──► Slave   :  Port 4000  (RMI Server, custom) ║
║   Slave  ──► Master  :  Random Port (Result callbacks)   ║
║                                                          ║
╚══════════════════════════════════════════════════════════╝
```

### Firewall Rules to Open on Each Slave Machine:

**Windows (PowerShell — Run as Administrator):**
```powershell
netsh advfirewall firewall add rule name="JMeter RMI" dir=in action=allow protocol=TCP localport=1099
netsh advfirewall firewall add rule name="JMeter Server" dir=in action=allow protocol=TCP localport=4000
```

**macOS (Terminal):**
```bash
# macOS uses pf (Packet Filter). Check System Preferences > Security & Privacy > Firewall
# For simple cases, just turn off the firewall during testing or allow JMeter specifically
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --setglobalstate off
```

**Linux (Ubuntu — Terminal):**
```bash
sudo ufw allow 1099/tcp
sudo ufw allow 4000/tcp
sudo ufw reload
sudo ufw status   # Verify rules are applied
```

**Linux (CentOS/RHEL — Terminal):**
```bash
sudo firewall-cmd --permanent --add-port=1099/tcp
sudo firewall-cmd --permanent --add-port=4000/tcp
sudo firewall-cmd --reload
```

---

## 5. Java Installation (All Machines)

### 5.1 Check if Java is Already Installed

Open a terminal / command prompt on every machine and run:

```bash
java -version
```

Expected output (example):
```
openjdk version "11.0.20" 2023-07-18
OpenJDK Runtime Environment (build 11.0.20+8)
```

If you see an error, follow the steps below for your OS.

---

### 5.2 Install Java on Windows

```
STEP 1: Go to https://adoptium.net/
STEP 2: Click "Latest LTS Release" → Download the .msi installer
STEP 3: Run the installer → Click Next → Accept License → Install
STEP 4: Set JAVA_HOME environment variable:
        - Right-click "This PC" → Properties
        - Click "Advanced system settings"
        - Click "Environment Variables"
        - Under "System Variables" → Click "New"
          Variable name:  JAVA_HOME
          Variable value: C:\Program Files\Eclipse Adoptium\jdk-11.x.x
        - Find "Path" → Edit → New → Add:  %JAVA_HOME%\bin
STEP 5: Open new Command Prompt → Run: java -version
```

### 5.3 Install Java on macOS

```bash
# Using Homebrew (recommended)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
brew install openjdk@11

# Link it so system finds it
sudo ln -sfn /opt/homebrew/opt/openjdk@11/libexec/openjdk.jdk /Library/Java/JavaVirtualMachines/openjdk-11.jdk

# Add to shell profile
echo 'export JAVA_HOME=$(/usr/libexec/java_home -v 11)' >> ~/.zshrc
echo 'export PATH=$JAVA_HOME/bin:$PATH' >> ~/.zshrc
source ~/.zshrc

# Verify
java -version
```

### 5.4 Install Java on Linux (Ubuntu/Debian)

```bash
# Update package list
sudo apt update

# Install OpenJDK 11
sudo apt install openjdk-11-jdk -y

# Set JAVA_HOME
echo 'export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64' >> ~/.bashrc
echo 'export PATH=$JAVA_HOME/bin:$PATH' >> ~/.bashrc
source ~/.bashrc

# Verify
java -version
```

### 5.5 Install Java on Linux (CentOS/RHEL)

```bash
sudo yum install java-11-openjdk-devel -y

# Set JAVA_HOME
echo 'export JAVA_HOME=/usr/lib/jvm/java-11-openjdk' >> ~/.bashrc
source ~/.bashrc
java -version
```

---

## 6. JMeter Installation (All Machines)

> 🔔 **Repeat these steps on EVERY machine** (Master + all Slaves). Use the **exact same JMeter version**.

### 6.1 Download JMeter

```
STEP 1: Visit: https://jmeter.apache.org/download_jmeter.cgi
STEP 2: Under "Binaries", download: apache-jmeter-5.6.3.tgz (or latest .zip for Windows)
```

---

### 6.2 Install JMeter on Windows

```
STEP 1: Download apache-jmeter-5.6.3.zip from the JMeter website

STEP 2: Right-click the zip file → "Extract All"
        Extract to: C:\JMeter\apache-jmeter-5.6.3\

STEP 3: Set Environment Variables:
        - System Variables → New:
          Variable name:  JMETER_HOME
          Variable value: C:\JMeter\apache-jmeter-5.6.3
        - Edit "Path" → Add: %JMETER_HOME%\bin

STEP 4: Verify Installation:
        Open Command Prompt and run:
        > jmeter --version

        Expected output:
          _    Apache JMeter
         / \  /
        / _ \/    Version 5.6.3

STEP 5: (GUI Launch — Master only)
        > jmeter.bat
```

**Folder Structure after installation:**
```
C:\JMeter\apache-jmeter-5.6.3\
│
├── bin\                  ← Executables & config files (most important!)
│   ├── jmeter.bat        ← Launch GUI (Windows)
│   ├── jmeter-server.bat ← Start as Slave (Windows)
│   ├── jmeter.properties ← Main config file ⭐
│   └── user.properties   ← Your custom overrides ⭐
│
├── lib\                  ← Libraries & plugins
├── extras\               ← Sample files
└── docs\                 ← Documentation
```

---

### 6.3 Install JMeter on macOS

```bash
# STEP 1: Download JMeter
curl -O https://downloads.apache.org/jmeter/binaries/apache-jmeter-5.6.3.tgz

# STEP 2: Extract the archive
tar -xzf apache-jmeter-5.6.3.tgz
sudo mv apache-jmeter-5.6.3 /opt/apache-jmeter

# STEP 3: Set environment variables
echo 'export JMETER_HOME=/opt/apache-jmeter' >> ~/.zshrc
echo 'export PATH=$JMETER_HOME/bin:$PATH' >> ~/.zshrc
source ~/.zshrc

# STEP 4: Verify
jmeter --version

# STEP 5: Launch GUI (Master only)
jmeter
```

---

### 6.4 Install JMeter on Linux

```bash
# STEP 1: Download JMeter
wget https://downloads.apache.org/jmeter/binaries/apache-jmeter-5.6.3.tgz

# STEP 2: Extract
tar -xzf apache-jmeter-5.6.3.tgz
sudo mv apache-jmeter-5.6.3 /opt/apache-jmeter

# STEP 3: Set environment variables
echo 'export JMETER_HOME=/opt/apache-jmeter' >> ~/.bashrc
echo 'export PATH=$JMETER_HOME/bin:$PATH' >> ~/.bashrc
source ~/.bashrc

# STEP 4: Verify
jmeter --version

# STEP 5: Launch GUI (Master only, if GUI is available)
jmeter
```

---

## 7. Slave Machine Configuration

The **Slave** machine runs `jmeter-server` — it waits for instructions from the Master. Here is how to configure and start the slave on each OS.

```
╔════════════════════════════════════════════════════════════╗
║  SLAVE MACHINE FLOW                                        ║
║                                                            ║
║  1. Configure jmeter.properties                           ║
║  2. Start jmeter-server process                           ║
║  3. Wait for Master to connect...                         ║
║  4. Receive test plan & run it                            ║
║  5. Send results back to Master                           ║
╚════════════════════════════════════════════════════════════╝
```

---

### 7.1 Windows Slave Setup

#### Step 1 — Edit jmeter.properties on Slave

Navigate to: `C:\JMeter\apache-jmeter-5.6.3\bin\`
Open `jmeter.properties` in Notepad++ or any text editor.

Find and update these lines:

```properties
# ─────────────────────────────────────────────────────────────
# SLAVE MACHINE — jmeter.properties changes
# File: C:\JMeter\apache-jmeter-5.6.3\bin\jmeter.properties
# ─────────────────────────────────────────────────────────────

# 1. Set the IP of THIS slave machine (NOT the master!)
#    Replace with your actual Slave machine IP
server.rmi.localport=4000

# 2. Set the RMI registry port (default 1099, keep as is)
server_port=1099

# 3. Disable SSL for RMI (use only in internal/safe networks)
server.rmi.ssl.disable=true
```

#### Step 2 — Edit user.properties on Slave (Recommended)

Open `user.properties` file (same `bin\` folder):

```properties
# ─────────────────────────────────────────────────────────────
# user.properties on SLAVE (Windows)
# ─────────────────────────────────────────────────────────────

# Set the slave's own IP address explicitly
# Replace 192.168.1.101 with actual Slave IP
java.rmi.server.hostname=192.168.1.101

# Fix the RMI server port so firewall rules work predictably
server.rmi.localport=4000

# Disable SSL (use in trusted LAN environments)
server.rmi.ssl.disable=true
```

#### Step 3 — Start the Slave Server (Windows)

```cmd
# Open Command Prompt as Administrator
# Navigate to JMeter bin folder
cd C:\JMeter\apache-jmeter-5.6.3\bin

# Start JMeter in Server (Slave) Mode
jmeter-server.bat
```

**Expected Output (Slave started successfully):**
```
Created remote object: UnicastServerRef2 [liveRef: [endpoint:[192.168.1.101:4000](local),objID:[-6a5d...]]]
Found ApacheJMeter_core.jar
Starting the JMeter Non-GUI resource usage collector thread...

              _    ___
             / \  /   \
            / _ \/     \     Apache JMeter
           / _ / |\     \
          /_/ \_| |______\

Copyright (c) 1999-2023 The Apache Software Foundation

Configured for SSL
Starting server...
Created remote object: UnicastServerRef...
Server started
```

> ✅ If you see "Server started", your Slave is running and ready!

---

### 7.2 macOS Slave Setup

#### Step 1 — Edit jmeter.properties on Slave (macOS)

```bash
# Open the properties file
nano /opt/apache-jmeter/bin/jmeter.properties
```

Find and update or add these lines:
```properties
# ─────────────────────────────────────────────────────────────
# SLAVE MACHINE — jmeter.properties (macOS)
# ─────────────────────────────────────────────────────────────
server.rmi.localport=4000
server_port=1099
server.rmi.ssl.disable=true
```

Save: `Ctrl+O` → `Enter` → `Ctrl+X`

#### Step 2 — Edit user.properties on Slave (macOS)

```bash
nano /opt/apache-jmeter/bin/user.properties
```

Add these lines:
```properties
# ─────────────────────────────────────────────────────────────
# user.properties on SLAVE (macOS)
# Replace 192.168.1.102 with your actual macOS Slave IP
# ─────────────────────────────────────────────────────────────
java.rmi.server.hostname=192.168.1.102
server.rmi.localport=4000
server.rmi.ssl.disable=true
```

#### Step 3 — Find Your macOS Machine's IP

```bash
# Get your machine's local IP
ifconfig | grep "inet " | grep -v 127.0.0.1
# or
ipconfig getifaddr en0    # for WiFi
ipconfig getifaddr en1    # for Ethernet
```

#### Step 4 — Start the Slave Server (macOS)

```bash
# Navigate to JMeter bin
cd /opt/apache-jmeter/bin

# Make the script executable (first time only)
chmod +x jmeter-server

# Start Slave
./jmeter-server
```

**Expected Output:**
```
Created remote object: UnicastServerRef2 [liveRef: [endpoint:[192.168.1.102:4000]...]]
Server started
```

#### Step 5 — Run Slave as a Background Process (macOS)

```bash
# Start in background so terminal stays free
nohup ./jmeter-server &

# Check if it's running
ps aux | grep jmeter-server
```

---

### 7.3 Linux Slave Setup

#### Step 1 — Edit jmeter.properties on Slave (Linux)

```bash
# Open file with nano or vi
nano /opt/apache-jmeter/bin/jmeter.properties
```

Add/update these lines:
```properties
# ─────────────────────────────────────────────────────────────
# SLAVE MACHINE — jmeter.properties (Linux)
# ─────────────────────────────────────────────────────────────
server.rmi.localport=4000
server_port=1099
server.rmi.ssl.disable=true
```

#### Step 2 — Edit user.properties on Slave (Linux)

```bash
nano /opt/apache-jmeter/bin/user.properties
```

```properties
# ─────────────────────────────────────────────────────────────
# user.properties on SLAVE (Linux)
# Replace 192.168.1.103 with your actual Linux Slave IP
# ─────────────────────────────────────────────────────────────
java.rmi.server.hostname=192.168.1.103
server.rmi.localport=4000
server.rmi.ssl.disable=true
```

#### Step 3 — Find Your Linux Machine's IP

```bash
# Method 1
hostname -I

# Method 2
ip addr show | grep "inet " | grep -v "127.0.0.1"

# Method 3
ifconfig eth0 | grep 'inet '   # replace eth0 with your interface
```

#### Step 4 — Start the Slave Server (Linux)

```bash
# Navigate to JMeter
cd /opt/apache-jmeter/bin

# Make executable
chmod +x jmeter-server

# Start Slave (foreground)
./jmeter-server

# OR start as background process (recommended for servers)
nohup ./jmeter-server > /tmp/jmeter-slave.log 2>&1 &

# Verify it started
tail -f /tmp/jmeter-slave.log
```

#### Step 5 — Create a systemd Service (Linux — Optional but Recommended)

This makes the Slave start automatically when the Linux server reboots.

```bash
# Create the service file
sudo nano /etc/systemd/system/jmeter-slave.service
```

Paste the following content:

```ini
[Unit]
Description=JMeter Slave Server
After=network.target

[Service]
Type=simple
User=root
ExecStart=/opt/apache-jmeter/bin/jmeter-server
Restart=on-failure
RestartSec=10
StandardOutput=append:/var/log/jmeter-slave.log
StandardError=append:/var/log/jmeter-slave-error.log

[Install]
WantedBy=multi-user.target
```

Enable and start the service:

```bash
sudo systemctl daemon-reload
sudo systemctl enable jmeter-slave
sudo systemctl start jmeter-slave

# Check status
sudo systemctl status jmeter-slave
```

---

## 8. Master Machine Configuration

The Master is where you create your test plan and trigger the distributed test. It sends the test plan to all Slaves and collects results.

```
╔══════════════════════════════════════════════════════════════════╗
║  MASTER MACHINE FLOW                                             ║
║                                                                  ║
║  1. List all Slave IPs in jmeter.properties                     ║
║  2. Open JMeter GUI                                             ║
║  3. Create or load Test Plan (.jmx)                             ║
║  4. Run → Remote Start All  (or use CLI for real tests)         ║
║  5. Watch results stream in from all Slaves                     ║
╚══════════════════════════════════════════════════════════════════╝
```

### 8.1 Edit jmeter.properties on Master

Open the `jmeter.properties` file on the **Master** machine:

- **Windows:** `C:\JMeter\apache-jmeter-5.6.3\bin\jmeter.properties`
- **macOS/Linux:** `/opt/apache-jmeter/bin/jmeter.properties`

Find the `remote_hosts` property and add all your Slave IPs:

```properties
# ─────────────────────────────────────────────────────────────
# MASTER MACHINE — jmeter.properties
# This is the MOST IMPORTANT setting on the Master!
# ─────────────────────────────────────────────────────────────

# List ALL slave machine IPs and their RMI ports
# Format: ip:port,ip:port,ip:port
remote_hosts=192.168.1.101:1099,192.168.1.102:1099,192.168.1.103:1099

# If your slaves are all on default port 1099, this also works:
# remote_hosts=192.168.1.101,192.168.1.102,192.168.1.103

# RMI SSL (disable for internal networks)
server.rmi.ssl.disable=true
```

### 8.2 Edit user.properties on Master

Open `user.properties` on the **Master** machine:

```properties
# ─────────────────────────────────────────────────────────────
# user.properties on MASTER
# Replace 192.168.1.100 with your actual Master IP
# ─────────────────────────────────────────────────────────────

# Master's own IP (important when machine has multiple network cards)
java.rmi.server.hostname=192.168.1.100

# Disable RMI SSL for LAN (re-enable for public networks!)
server.rmi.ssl.disable=true
```

### 8.3 Open JMeter GUI on Master

- **Windows:** Double-click `jmeter.bat` OR run `jmeter.bat` from Command Prompt
- **macOS/Linux:** Run `jmeter` from Terminal

### 8.4 Verify Slaves Are Connected in GUI

```
In JMeter GUI:
  Menu → Run → Remote Start All

OR go to:
  Menu → Run → Choose Slaves (you'll see the list of connected slave IPs)
```

If slaves show up in the list ✅ — you are ready to run!

---

## 9. JMeter Properties Configuration (Master & Slave)

This section is a complete reference for all the important properties you need to understand.

### 9.1 jmeter.properties — Complete Reference

```properties
# ═══════════════════════════════════════════════════════════════
#  FILE: jmeter.properties
#  LOCATION: <JMETER_HOME>/bin/jmeter.properties
#  APPLIES TO: Master AND Slave (different values per machine)
# ═══════════════════════════════════════════════════════════════

# ── MASTER ONLY ────────────────────────────────────────────────
# List of remote slave hosts (IP:port)
# remote_hosts=192.168.1.101:1099,192.168.1.102:1099

# ── SLAVE ONLY ─────────────────────────────────────────────────
# Port that THIS slave listens on for RMI registry
server_port=1099

# ── BOTH MASTER AND SLAVE ──────────────────────────────────────

# Fixed port for RMI communication (avoids random port issues)
server.rmi.localport=4000

# Disable SSL — only for trusted internal networks
server.rmi.ssl.disable=true

# Batch size for sending results from Slave → Master
mode=StrippedBatch

# Number of samples per batch (default 100)
# num_sample_threshold=100

# Interval (ms) to send batches of results to master
# time_threshold=60000

# Heap size adjustments (set in jmeter script/bat, not here)
# See: jmeter.bat → set HEAP=-Xms1g -Xmx4g
```

---

### 9.2 user.properties — Complete Reference

```properties
# ═══════════════════════════════════════════════════════════════
#  FILE: user.properties
#  LOCATION: <JMETER_HOME>/bin/user.properties
#  NOTE: Settings here OVERRIDE jmeter.properties
#        Put your custom settings here to avoid editing
#        the main jmeter.properties file.
# ═══════════════════════════════════════════════════════════════

# ── ON MASTER ──────────────────────────────────────────────────
# java.rmi.server.hostname=192.168.1.100   ← Master's IP
# remote_hosts=192.168.1.101,192.168.1.102,192.168.1.103

# ── ON SLAVE ───────────────────────────────────────────────────
# java.rmi.server.hostname=192.168.1.101   ← This Slave's IP
# server.rmi.localport=4000
# server.rmi.ssl.disable=true

# ── BOTH ───────────────────────────────────────────────────────

# Summariser output interval (seconds)
summariser.interval=30

# Enable detailed logging (useful for debugging)
# log_level.jmeter=DEBUG

# Results file format
jmeter.save.saveservice.output_format=csv
jmeter.save.saveservice.response_data=false
jmeter.save.saveservice.samplerData=false
jmeter.save.saveservice.requestHeaders=false
jmeter.save.saveservice.url=true
jmeter.save.saveservice.filename=true
jmeter.save.saveservice.hostname=true
jmeter.save.saveservice.thread_counts=true
jmeter.save.saveservice.sample_count=true
jmeter.save.saveservice.response_time=true
```

---

### 9.3 Increase JVM Heap Memory (For High Load Tests)

When running tests with thousands of users, you need to increase Java heap memory.

**Windows** — Edit `C:\JMeter\apache-jmeter-5.6.3\bin\jmeter.bat`:
```batch
rem Find this line and update:
set HEAP=-Xms1g -Xmx4g -XX:MaxMetaspaceSize=256m
```

**macOS / Linux** — Edit `/opt/apache-jmeter/bin/jmeter`:
```bash
# Find this line and update:
HEAP="-Xms1g -Xmx4g -XX:MaxMetaspaceSize=256m"
```

**On Slave machines** — Edit `jmeter-server` (same bin folder):
```bash
# Add/update heap settings:
HEAP="-Xms2g -Xmx8g -XX:MaxMetaspaceSize=512m"
```

> 💡 **Rule of thumb:** Set `-Xmx` to about 80% of available RAM. E.g., 8 GB RAM → use `-Xmx6g`

---

## 10. Running Distributed Tests

### 10.1 Method 1 — Using JMeter GUI (Good for beginners)

```
┌──────────────────────────────────────────────────────────────┐
│                  STEPS IN JMETER GUI                         │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. Open JMeter on Master machine                           │
│     Windows: jmeter.bat                                     │
│     Mac/Linux: ./jmeter                                     │
│                                                             │
│  2. Create or open your Test Plan (.jmx file)               │
│                                                             │
│  3. Go to: Run → Remote Start All                           │
│     (This starts the test on ALL slaves simultaneously)     │
│                                                             │
│  4. OR go to: Run → Remote Start                            │
│     (Choose specific slave IPs to use)                      │
│                                                             │
│  5. Watch the results in:                                   │
│     - View Results Tree listener                            │
│     - Summary Report listener                               │
│     - Aggregate Report listener                             │
│                                                             │
│  6. To stop: Run → Remote Stop All                          │
│                                                             │
└──────────────────────────────────────────────────────────────┘
```

> ⚠️ **IMPORTANT:** Never use JMeter GUI for actual performance tests! GUI consumes resources. Use CLI (below) for real load testing and only use GUI to build the test plan.

---

### 10.2 Method 2 — Using CLI / Command Line (Recommended for Real Tests)

**Run from the Master machine:**

```bash
# ─────────────────────────────────────────────────────────────────
# BASIC CLI COMMAND FOR DISTRIBUTED TEST
# Run this on the MASTER machine
# ─────────────────────────────────────────────────────────────────

jmeter -n \
  -t /path/to/your/TestPlan.jmx \
  -r \
  -l /path/to/results/results.jtl \
  -e \
  -o /path/to/html-report/

# ─────────────────────────────────────────────────────────────────
# PARAMETER BREAKDOWN:
# ─────────────────────────────────────────────────────────────────
# -n          = Non-GUI mode (no window, just terminal output)
# -t          = Path to your Test Plan (.jmx file)
# -r          = Run test on ALL remote slaves (from remote_hosts)
# -l          = Log results to this .jtl file
# -e          = Generate HTML report after test
# -o          = Output folder for the HTML report
```

**With specific slave IPs (override remote_hosts):**

```bash
jmeter -n \
  -t /path/to/TestPlan.jmx \
  -R 192.168.1.101,192.168.1.102,192.168.1.103 \
  -l results.jtl \
  -e \
  -o html-report/

# -R (capital) = Specify slave IPs directly (overrides remote_hosts property)
```

**Windows Example:**
```cmd
cd C:\JMeter\apache-jmeter-5.6.3\bin

jmeter.bat -n -t C:\Tests\MyTestPlan.jmx -r -l C:\Results\results.jtl -e -o C:\Results\report
```

**macOS / Linux Example:**
```bash
cd /opt/apache-jmeter/bin

./jmeter -n -t ~/tests/MyTestPlan.jmx -r -l ~/results/results.jtl -e -o ~/results/report
```

---

### 10.3 What You Should See When Test Runs (CLI Output)

```
Creating summariser <summary>
Created the tree successfully using /tests/MyTestPlan.jmx
Configuring remote engine: 192.168.1.101:1099
Configuring remote engine: 192.168.1.102:1099
Configuring remote engine: 192.168.1.103:1099
Starting distributed test with remote engines: [192.168.1.101:1099, 192.168.1.102:1099, 192.168.1.103:1099]
Remote engines have been started:[192.168.1.101:1099, 192.168.1.102:1099, 192.168.1.103:1099]
Waiting for possible Shutdown/StopTestNow/HeapDump/ThreadDump message on port 4445

summary =   1500 in 00:00:30 =   50.0/s Avg:  200 Min:  101 Max:  850 Err:     0 (0.00%)
summary =   3000 in 00:00:30 =  100.0/s Avg:  210 Min:   98 Max:  920 Err:     2 (0.07%)
```

```
╔═══════════════════════════════════════════════════════════════╗
║  HOW TO READ THE SUMMARY OUTPUT                               ║
╠═══════════════════════════════════════════════════════════════╣
║                                                               ║
║  summary = 1500    → Total samples sent so far               ║
║  in 00:00:30       → Time elapsed                            ║
║  = 50.0/s          → Throughput (requests per second)        ║
║  Avg: 200          → Average response time (ms)              ║
║  Min: 101          → Fastest response (ms)                   ║
║  Max: 850          → Slowest response (ms)                   ║
║  Err: 0 (0.00%)    → Error count and % ← keep this low!     ║
║                                                               ║
╚═══════════════════════════════════════════════════════════════╝
```

---

## 11. Collecting Results on Master

After the test completes, all results are automatically collected from Slaves and aggregated on the Master.

### 11.1 Results File (.jtl)

The `.jtl` file is a CSV containing every request's details:

```csv
timeStamp,elapsed,label,responseCode,responseMessage,threadName,dataType,success,failureMessage,bytes,sentBytes,grpThreads,allThreads,URL,Latency,IdleTime,Connect
1699523401000,245,HTTP Request,200,OK,Thread Group 1-1,text,true,,12340,432,50,150,https://myapp.com/api,230,0,15
```

### 11.2 Generate HTML Report

```bash
# Generate HTML report from .jtl file (if not done automatically)
jmeter -g results.jtl -o /path/to/html-report/
```

The HTML report contains:
- 📊 Response time graphs
- 📈 Throughput over time
- 🥧 Error percentage pie charts
- 📋 Statistics table (90th, 95th, 99th percentile)

### 11.3 View Results in JMeter GUI (Post-Run)

```
1. Open JMeter GUI (without running a test)
2. Add a listener: Right-click Test Plan → Add → Listener → Aggregate Report
3. In the listener: Browse → Select your results.jtl file
4. JMeter will load and display all the results
```

---

## 12. Troubleshooting Common Issues

```
╔══════════════════════════════════════════════════════════════════════╗
║                  COMMON PROBLEMS & FIXES                             ║
╠═══════════════════════════════╦══════════════════════════════════════╣
║ ERROR                         ║ SOLUTION                             ║
╠═══════════════════════════════╬══════════════════════════════════════╣
║ Connection refused to slave   ║ • Check slave IP in remote_hosts     ║
║                               ║ • Ensure jmeter-server is running    ║
║                               ║ • Check firewall ports 1099 & 4000   ║
╠═══════════════════════════════╬══════════════════════════════════════╣
║ Slave connects but no results ║ • Check server.rmi.localport matches ║
║                               ║ • Verify no firewall blocks callback  ║
╠═══════════════════════════════╬══════════════════════════════════════╣
║ java.io.NotSerializable       ║ • Plugins on Master ≠ Plugins on     ║
║                               ║   Slaves. Sync all JMeter libs!      ║
╠═══════════════════════════════╬══════════════════════════════════════╣
║ OutOfMemoryError              ║ • Increase HEAP in jmeter/            ║
║                               ║   jmeter-server script               ║
╠═══════════════════════════════╬══════════════════════════════════════╣
║ Slaves not in Remote Start    ║ • Wrong IP in remote_hosts           ║
║ list                          ║ • Slave server not started           ║
║                               ║ • Different JMeter versions          ║
╠═══════════════════════════════╬══════════════════════════════════════╣
║ RMI ClassNotFoundException    ║ • JMeter version mismatch            ║
║                               ║ • Missing JAR on Slave               ║
╚═══════════════════════════════╩══════════════════════════════════════╝
```

### Debug Commands

**Ping all slaves from Master:**
```bash
ping 192.168.1.101
ping 192.168.1.102
ping 192.168.1.103
```

**Check if Slave RMI port is open:**
```bash
# From Master, test if slave port is reachable
telnet 192.168.1.101 1099   # Should connect
telnet 192.168.1.101 4000   # Should connect

# Or use nc (netcat)
nc -zv 192.168.1.101 1099
```

**Check JMeter Slave logs:**
```bash
# Linux
tail -f /tmp/jmeter-slave.log

# Or look in JMeter bin folder
cat /opt/apache-jmeter/bin/jmeter-server.log
```

**Kill stale JMeter slave process:**
```bash
# Linux/macOS
pkill -f jmeter-server
# or
ps aux | grep jmeter | awk '{print $2}' | xargs kill -9

# Windows (Command Prompt)
taskkill /f /im java.exe
```

---

## 13. Quick Reference Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════════╗
║                     JMETER DISTRIBUTED — QUICK CHEAT SHEET              ║
╠══════════════════════════════════════════════════════════════════════════╣
║                                                                          ║
║  ➤ START SLAVE                                                           ║
║    Windows :  jmeter-server.bat                                          ║
║    Mac/Linux: ./jmeter-server                                            ║
║                                                                          ║
║  ➤ RUN TEST (CLI from Master)                                            ║
║    jmeter -n -t test.jmx -r -l results.jtl -e -o report/               ║
║                                                                          ║
║  ➤ RUN TEST WITH SPECIFIC SLAVES                                         ║
║    jmeter -n -t test.jmx -R ip1,ip2,ip3 -l results.jtl                 ║
║                                                                          ║
║  ➤ GENERATE HTML REPORT FROM EXISTING RESULTS                           ║
║    jmeter -g results.jtl -o report/                                     ║
║                                                                          ║
║  ➤ KEY PROPERTIES (jmeter.properties / user.properties)                 ║
║    MASTER: remote_hosts=ip1:1099,ip2:1099                               ║
║    SLAVE:  java.rmi.server.hostname=<THIS_SLAVE_IP>                     ║
║    BOTH:   server.rmi.localport=4000                                    ║
║    BOTH:   server.rmi.ssl.disable=true                                  ║
║                                                                          ║
║  ➤ PORTS TO OPEN ON SLAVE FIREWALL                                       ║
║    TCP 1099  (RMI Registry)                                              ║
║    TCP 4000  (RMI Server — server.rmi.localport)                        ║
║                                                                          ║
╚══════════════════════════════════════════════════════════════════════════╝
```

---

### Summary of Files Changed on Each Machine

| File | Master | Slave |
|------|--------|-------|
| `jmeter.properties` | Add `remote_hosts` with all Slave IPs | Add `server_port`, `server.rmi.localport` |
| `user.properties` | Add Master's own IP as `java.rmi.server.hostname` | Add Slave's own IP as `java.rmi.server.hostname` |
| `jmeter.bat` or `jmeter` script | Increase HEAP for collecting results | Increase HEAP for generating load |
| `jmeter-server.bat` / `jmeter-server` | Not applicable | Start this to run as Slave |

---

### Complete Setup Verification Checklist

```
Before running your first distributed test, verify:

  SLAVE MACHINES (do for each slave):
  [ ] Java installed and java -version works
  [ ] JMeter installed (SAME version as Master!)
  [ ] user.properties: java.rmi.server.hostname = <slave IP>
  [ ] user.properties: server.rmi.localport = 4000
  [ ] user.properties: server.rmi.ssl.disable = true
  [ ] Firewall: Port 1099 open (inbound)
  [ ] Firewall: Port 4000 open (inbound)
  [ ] jmeter-server is running (see "Server started" in output)

  MASTER MACHINE:
  [ ] Java installed and java -version works
  [ ] JMeter installed (SAME version as Slaves!)
  [ ] jmeter.properties: remote_hosts = all slave IPs listed
  [ ] user.properties: java.rmi.server.hostname = <master IP>
  [ ] user.properties: server.rmi.ssl.disable = true
  [ ] Can ping all slave machines successfully
  [ ] Telnet/nc to slave:1099 succeeds

  NETWORK:
  [ ] All machines on same network or VPN
  [ ] No proxy between Master and Slaves
  [ ] DNS or /etc/hosts entries if using hostnames instead of IPs
```

---

*📝 Created for JMeter Distributed Testing — Beginner's Complete Reference Guide*  
*📦 JMeter Version: 5.6.x | ☕ Java Version: 11 LTS Recommended*  
*🖥️ Supported OS: Windows 10/11, macOS 12+, Ubuntu 20.04/22.04, CentOS/RHEL 7/8*
