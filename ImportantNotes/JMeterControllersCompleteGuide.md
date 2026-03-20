# 🚀 JMeter Controllers – Complete Step-by-Step Guide
### Using OrangeHRM Demo Site: https://opensource-demo.orangehrmlive.com/

---

## 📋 Table of Contents

1. [What is a JMeter Controller?](#1-what-is-a-jmeter-controller)
2. [Types of JMeter Controllers](#2-types-of-jmeter-controllers)
3. [Logic Controllers – Detailed Guide](#3-logic-controllers--detailed-guide)
   - [3.01 Simple Controller](#301-simple-controller)
   - [3.02 Loop Controller](#302-loop-controller)
   - [3.03 Once Only Controller](#303-once-only-controller)
   - [3.04 Interleave Controller](#304-interleave-controller)
   - [3.05 Random Controller](#305-random-controller)
   - [3.06 Random Order Controller](#306-random-order-controller)
   - [3.07 Throughput Controller](#307-throughput-controller)
   - [3.08 Runtime Controller](#308-runtime-controller)
   - [3.09 If Controller](#309-if-controller)
   - [3.10 While Controller](#310-while-controller)
   - [3.11 Switch Controller](#311-switch-controller)
   - [3.12 ForEach Controller](#312-foreach-controller)
   - [3.13 Transaction Controller](#313-transaction-controller)
   - [3.14 Module Controller](#314-module-controller)
   - [3.15 Recording Controller](#315-recording-controller)
4. [OrangeHRM – Scenario Summary Table](#4-orangehrm--scenario-summary-table)
5. [JMX File – Detailed Explanation](#5-jmx-file--detailed-explanation)
6. [Assertions & Validations – How They Were Added](#6-assertions--validations--how-they-were-added)
7. [Controller Comparison Table](#7-controller-comparison-table)

---

## 1. What is a JMeter Controller?

A **JMeter Controller** is a **test element** that controls the **flow and execution logic** of samplers (HTTP requests) within a Thread Group. Controllers decide:

- **When** a sampler should run
- **How many times** it should run
- **Under what conditions** it should run
- **In what order** the samplers should execute

Think of controllers as the **brain** of your test plan. Without controllers, all requests would simply execute top-to-bottom in sequence. Controllers allow you to build realistic, dynamic, and complex test flows that mirror how real users interact with an application.

### 🔑 Key Points About Controllers

| Aspect | Description |
|--------|-------------|
| **Location** | Inside a Thread Group |
| **Can be nested** | Yes — controllers can contain other controllers |
| **Can contain** | Samplers, Timers, Assertions, Extractors, other Controllers |
| **Purpose** | Execution flow logic, looping, branching, grouping |
| **Two main families** | Logic Controllers and Samplers (samplers are technically their own category) |

### 🏗️ Where Controllers Sit in a JMeter Test Plan

```
Test Plan
└── Thread Group
    ├── Config Elements (Cookie Manager, HTTP Defaults...)
    ├── [Controllers]        ← This is where controllers live
    │   ├── Samplers         ← HTTP Requests inside controllers
    │   ├── Assertions       ← Validation rules
    │   ├── Extractors       ← Capture dynamic values
    │   └── Timers           ← Think time between requests
    └── Listeners            ← Results collectors
```

---

## 2. Types of JMeter Controllers

JMeter has **two broad categories** of controllers:

### 🅰️ Logic Controllers
These control the **flow** of execution — they determine **which child elements run and when**.

| # | Controller | Purpose |
|---|-----------|---------|
| 1 | Simple Controller | Logical grouping of requests |
| 2 | Loop Controller | Repeat requests N times |
| 3 | Once Only Controller | Run children only once per thread |
| 4 | Interleave Controller | Alternate between child samplers |
| 5 | Random Controller | Pick one child at random |
| 6 | Random Order Controller | Run all children but in random order |
| 7 | Throughput Controller | Control percentage/rate of execution |
| 8 | Runtime Controller | Run children for a fixed time duration |
| 9 | If Controller | Conditional execution (if/else logic) |
| 10 | While Controller | Loop while a condition is true |
| 11 | Switch Controller | Select child by index/name |
| 12 | ForEach Controller | Iterate over a list of values |
| 13 | Transaction Controller | Group requests as a single transaction |
| 14 | Module Controller | Reference and reuse test fragments |
| 15 | Recording Controller | Placeholder for recorded scripts |

### 🅱️ Sampler Controllers
These **generate the actual requests** (HTTP, FTP, JDBC, etc.). While they are technically in the "sampler" category, they are children of logic controllers.

---

## 3. Logic Controllers – Detailed Guide

---

### 3.01 Simple Controller

#### 📌 What Is It?
The **Simple Controller** is the most basic organizational unit in JMeter. It does **not change execution behavior** — it simply acts as a **folder or container** to group related samplers together for better readability and organization.

#### ⚙️ Key Features
- Zero impact on test execution logic
- Purely organizational — acts like a folder
- Can hold samplers, timers, assertions, other controllers
- Makes test plans readable and maintainable
- Frequently used to group requests by feature/module

#### 🎯 Why We Use It
- **Organization**: Group all "Employee Management" requests separately from "Leave Management" requests
- **Readability**: Large test plans become unmanageable without grouping
- **Reusability**: Easy to enable/disable an entire feature section
- **Debugging**: Easier to isolate a failing module

#### 🔄 How It Differs from Others
Unlike Transaction Controller (which measures combined response time), the Simple Controller does **not record a combined result**. Unlike Loop or If controllers, it has **no execution logic**. It is purely a structural container.

#### 🌐 OrangeHRM Scenario

**Scenario**: Group all Employee Management operations (View Employee List, Search Employee, View Employee Profile) under one logical container called "Employee Management Module."

**Why this controller for this scenario**: The OrangeHRM PIM module has multiple related operations. Using Simple Controller, we group `GET /pim/viewEmployeeList`, `GET /pim/searchEmployee`, and `GET /pim/viewPimModule` together. This provides no functional change but makes it immediately clear to any team member what those 3 requests are testing.

**Steps in test plan**:
```
Thread Group: Employee Management
└── Simple Controller: "PIM - Employee Management"
    ├── HTTP Request: GET Employee List
    │   └── Response Assertion: Status 200
    ├── HTTP Request: GET Search Employee  
    │   └── Response Assertion: Contains "Employee Name"
    └── HTTP Request: GET Employee Profile
        └── Duration Assertion: < 3000ms
```

**How to add in JMeter GUI**:
1. Right-click Thread Group → Add → Logic Controller → Simple Controller
2. Rename to "PIM - Employee Management"
3. Drag HTTP requests into the controller

---

### 3.02 Loop Controller

#### 📌 What Is It?
The **Loop Controller** repeats all its child elements a **specified number of times** within a single thread iteration. It is one of the most fundamental flow control elements in JMeter.

#### ⚙️ Key Features
- Set a fixed loop count (integer) or infinite loop (checkbox)
- Can be nested inside other controllers
- Loop count is independent of thread iteration count
- Useful for simulating repeated actions within one user session
- Supports JMeter variables for dynamic loop counts: `${LOOP_COUNT}`

#### 🎯 Why We Use It
- Simulate a user who performs the same action multiple times (e.g., searching 5 different employees)
- Stress test a specific endpoint by hitting it repeatedly within one session
- Simulate batch operations (submit 3 leave requests back-to-back)
- Data-driven looping when combined with CSV Data Set Config

#### 🔄 How It Differs from Others
- **vs Thread Group loops**: Thread Group loops iterate the entire thread group. Loop Controller only repeats its own children.
- **vs While Controller**: Loop Controller has a fixed count; While Controller runs based on a dynamic condition.
- **vs ForEach Controller**: ForEach iterates over specific variable values; Loop simply repeats N times.

#### 🌐 OrangeHRM Scenario

**Scenario**: Simulate a manager who searches for 3 different employees back-to-back in the same session.

**Why this controller for this scenario**: The HR Manager doesn't log out and log back in between each search. They stay in the same session and repeat the search action. Loop Controller perfectly models this — 3 iterations of the same search request within one user session.

**Steps in test plan**:
```
Thread Group: Manager Search Session
└── HTTP Request: POST Login (Admin/admin123)
└── Loop Controller: "Repeat Employee Search x3"  [Loop Count = 3]
    ├── HTTP Request: GET Employee List
    │   └── Response Assertion: Status 200
    └── HTTP Request: GET Search Employee (name=${EMPLOYEE_NAME})
        └── Response Assertion: Contains "employeeId"
```

**Configuration**:
- Loop Count: `3` (or use `${LOOP_COUNT}` for dynamic)
- To loop forever: Check "Forever" checkbox

---

### 3.03 Once Only Controller

#### 📌 What Is It?
The **Once Only Controller** ensures its child elements execute **exactly once per thread** (per virtual user), regardless of how many times the Thread Group iterates. This is perfect for **one-time setup actions** like login.

#### ⚙️ Key Features
- Executes children only on the **first iteration** of the thread
- Skipped completely on all subsequent iterations (2nd, 3rd, ...)
- No configuration parameters needed — it just works
- Often used for login, setup data, or initializing sessions
- Thread-scoped: each virtual user runs it once independently

#### 🎯 Why We Use It
- **Login**: Users log in once and maintain session across iterations
- **Setup**: Load a homepage, get CSRF token — done once per user
- **Data initialization**: Fetch user preferences on first visit
- **Cost efficiency**: Avoid redundant requests that inflate test results

#### 🔄 How It Differs from Others
- **vs Loop Controller**: Loop repeats children N times; Once Only restricts to 1 time regardless of N iterations
- **vs If Controller**: If Controller re-evaluates condition every iteration; Once Only never evaluates — it simply skips after iteration 1
- **vs Transaction Controller**: Transaction Controller groups for timing; Once Only is about execution frequency

#### 🌐 OrangeHRM Scenario

**Scenario**: Run 5 iterations of browsing HR modules (Dashboard → Leave → Time). But the user should only log in **once** — not re-login on every iteration.

**Why this controller for this scenario**: Real users don't log out and back in between browsing pages. The session stays alive. With 5 Thread Group iterations and no Once Only Controller, JMeter would POST to `/auth/validate` 5 times. With Once Only Controller, login happens only on iteration 1, and the cookie/session is reused for iterations 2–5.

**Steps in test plan**:
```
Thread Group: Browse HR Modules [Loops = 5]
├── Once Only Controller: "Login Once"
│   ├── HTTP Request: GET Login Page (/web/index.php/auth/login)
│   └── HTTP Request: POST Login (/web/index.php/auth/validate)
│       └── Response Assertion: Redirect to /dashboard
├── HTTP Request: GET Dashboard
│   └── Response Assertion: Contains "Dashboard"
├── HTTP Request: GET Leave Module
└── HTTP Request: GET Time Module
```

**Execution flow**:
- Iteration 1: Login → Dashboard → Leave → Time
- Iteration 2: [skip login] → Dashboard → Leave → Time
- Iteration 3: [skip login] → Dashboard → Leave → Time

---

### 3.04 Interleave Controller

#### 📌 What Is It?
The **Interleave Controller** alternates through its child samplers — executing **one child per iteration**, cycling through them in order. On the 1st iteration it runs child #1, on the 2nd iteration child #2, and so on. After exhausting all children, it loops back to child #1.

#### ⚙️ Key Features
- One child executed per iteration (round-robin style)
- Option to **"Ignore sub-controller blocks"** — treats nested controllers as single units
- Option to **"Interleave across threads"** — coordinates execution order across all virtual users
- Cycles back to the beginning after the last child
- Useful for simulating varied user behavior over time

#### 🎯 Why We Use It
- Simulate different users doing different tasks across iterations
- Balance load evenly across multiple HR modules
- Avoid hammering one endpoint repeatedly when multiple are available
- Model realistic browsing: user visits different pages on different sessions

#### 🔄 How It Differs from Others
- **vs Random Controller**: Random picks any child unpredictably; Interleave cycles sequentially
- **vs Random Order Controller**: Random Order runs all children per iteration in random order; Interleave runs only one per iteration
- **vs Loop Controller**: Loop repeats all children N times; Interleave runs one at a time

#### 🌐 OrangeHRM Scenario

**Scenario**: Simulate users who alternately check the Leave module and the Time module across multiple test iterations — mimicking how different sessions access different HR sections.

**Why this controller for this scenario**: If we run 6 iterations, the Leave module gets tested on iterations 1, 3, 5 and the Time module on iterations 2, 4, 6. This creates a realistic mixed workload without overwhelming a single endpoint.

**Steps in test plan**:
```
Thread Group: HR Module Alternation [Loops = 6]
├── Once Only Controller: "Login"
│   └── HTTP Request: POST Login
└── Interleave Controller: "Alternate Leave & Time"
    ├── HTTP Request: GET Leave List (/web/index.php/leave/viewLeaveList)
    │   └── Response Assertion: Status 200
    └── HTTP Request: GET Time Module (/web/index.php/time/viewTimeModule)
        └── Response Assertion: Status 200
```

**Execution pattern**:
```
Iter 1 → GET Leave List
Iter 2 → GET Time Module
Iter 3 → GET Leave List
Iter 4 → GET Time Module
...
```

---

### 3.05 Random Controller

#### 📌 What Is It?
The **Random Controller** picks **one child at random** each time it is executed. Every iteration, it randomly selects a single child sampler or controller from its children and runs only that one.

#### ⚙️ Key Features
- Truly random selection (not round-robin)
- Only one child is executed per controller execution
- Every child has an equal chance of being selected
- Can be seeded or run unpredictably
- Useful for simulating realistic, non-deterministic user behavior

#### 🎯 Why We Use It
- Model unpredictable user navigation (users don't follow scripts)
- Test that all endpoints perform equally well under random load
- Simulate a pool of possible user actions where one is chosen each session
- Chaos/exploratory testing of different application paths

#### 🔄 How It Differs from Others
- **vs Interleave Controller**: Interleave cycles sequentially; Random is non-deterministic
- **vs Random Order Controller**: Random Order runs ALL children but in random sequence; Random runs only ONE child
- **vs Switch Controller**: Switch is deterministic (based on a variable); Random is probabilistic

#### 🌐 OrangeHRM Scenario

**Scenario**: Simulate users who, after logging in, randomly navigate to one of the four main HR modules: PIM (Employee List), Leave, Recruitment, or Performance.

**Why this controller for this scenario**: Real users don't follow a predictable path. One user goes to Leave, another goes to PIM, another to Recruitment. Random Controller simulates this stochastic navigation, creating a realistic mixed load across all OrangeHRM modules simultaneously.

**Steps in test plan**:
```
Thread Group: Random Module Navigation [Threads = 10]
├── Once Only Controller: "Login"
│   └── HTTP Request: POST Login
└── Random Controller: "Navigate to Random Module"
    ├── HTTP Request: GET Employee List (PIM)
    │   └── Response Assertion: Status 200
    ├── HTTP Request: GET Leave Module
    │   └── Response Assertion: Status 200
    ├── HTTP Request: GET Recruitment
    │   └── Response Assertion: Status 200
    └── HTTP Request: GET Performance
        └── Response Assertion: Status 200
```

Each thread randomly picks one of the 4 modules on each iteration.

---

### 3.06 Random Order Controller

#### 📌 What Is It?
The **Random Order Controller** executes **all** of its child elements, but in a **randomized order** each time it runs. Unlike Random Controller (which picks one), this controller runs everything — just shuffled.

#### ⚙️ Key Features
- All children are executed (none skipped)
- Order is randomized on every iteration
- Each iteration may have a completely different execution sequence
- No configuration required — it automatically shuffles
- Useful when order of operations shouldn't matter but all must be tested

#### 🎯 Why We Use It
- Test that your application handles requests in any order correctly
- Simulate users who navigate pages in different sequences
- Validate that there are no hidden order-of-execution dependencies in the system
- Realistic modeling of multi-tab browsing behavior

#### 🔄 How It Differs from Others
- **vs Random Controller**: Random Controller runs only ONE child; Random Order runs ALL children
- **vs Interleave Controller**: Interleave runs one per iteration in sequence; Random Order runs all per iteration in random sequence
- **vs Simple Controller**: Simple Controller preserves order; Random Order shuffles it

#### 🌐 OrangeHRM Scenario

**Scenario**: Test that all four HR modules (PIM, Leave, Time, Recruitment) are accessible in any order — validating there are no session or state dependencies between modules.

**Why this controller for this scenario**: A QA engineer wants to verify that the OrangeHRM application doesn't break if users access modules in an unconventional order. Random Order Controller ensures every permutation is eventually tested across multiple test runs.

**Steps in test plan**:
```
Thread Group: Module Access Order Test [Loops = 5]
├── Once Only Controller: "Login"
│   └── HTTP Request: POST Login
└── Random Order Controller: "All Modules - Random Order"
    ├── HTTP Request: GET PIM Module
    │   └── Response Assertion: Contains "Employee Information"
    ├── HTTP Request: GET Leave Module
    │   └── Response Assertion: Contains "Leave"
    ├── HTTP Request: GET Time Module
    │   └── Response Assertion: Contains "Time"
    └── HTTP Request: GET Recruitment Module
        └── Response Assertion: Contains "Recruitment"
```

---

### 3.07 Throughput Controller

#### 📌 What Is It?
The **Throughput Controller** limits how often its children execute relative to other elements in the same Thread Group. It can work in two modes:
- **Percent Executions**: Children execute X% of the time
- **Total Executions**: Children execute exactly N times total

#### ⚙️ Key Features
- Two modes: Percentage-based and Count-based
- Percentage mode: Values between 0.0 and 100.0
- Count mode: Exact number of total executions
- "Per User" checkbox: Count is per-thread or global
- Allows realistic traffic distribution (e.g., 70% browse, 30% transact)

#### 🎯 Why We Use It
- Model realistic traffic distribution (most users browse, fewer transact)
- Simulate A/B test traffic splits
- Control the ratio of read vs write operations
- Test system behavior under a specific load mix
- Validate performance SLAs at different usage tiers

#### 🔄 How It Differs from Others
- **vs Runtime Controller**: Runtime controls duration; Throughput controls frequency
- **vs Loop Controller**: Loop repeats N times per iteration; Throughput controls global execution probability
- **vs If Controller**: If checks conditions; Throughput uses statistical probability

#### 🌐 OrangeHRM Scenario

**Scenario**: Model realistic OrangeHRM usage — 70% of users just view the Dashboard and Employee List (read-only), while 30% of users create or modify leave requests (write operations).

**Why this controller for this scenario**: In production HR systems, the majority of traffic is read operations (viewing dashboards, reports) while a minority is write operations (submitting leaves, updating records). Throughput Controller lets us accurately model this 70/30 split without writing complex logic.

**Steps in test plan**:
```
Thread Group: Realistic HR Traffic [Threads = 10, Loops = 10]
├── Once Only Controller: "Login"
│   └── HTTP Request: POST Login
├── Throughput Controller: "Dashboard Read - 70%"  [Mode: Percent, Value: 70.0]
│   ├── HTTP Request: GET Dashboard
│   └── HTTP Request: GET Employee List
│       └── Response Assertion: Status 200
└── Throughput Controller: "Leave Management Write - 30%"  [Mode: Percent, Value: 30.0]
    ├── HTTP Request: GET Leave Apply Form
    └── HTTP Request: POST Apply Leave
        └── Response Assertion: Contains "Successfully Saved"
```

---

### 3.08 Runtime Controller

#### 📌 What Is It?
The **Runtime Controller** executes its child elements for a **specified duration in seconds**, then stops — regardless of how many iterations have been completed. It introduces time-based execution instead of count-based.

#### ⚙️ Key Features
- Duration is specified in seconds
- Children run repeatedly until the time expires
- After expiry, no more children are executed for that iteration
- Can be nested — inner controller runs for its own duration
- Useful for timed scenarios and soak tests

#### 🎯 Why We Use It
- Soak testing: "Run login stress for exactly 60 seconds"
- Time-boxed performance tests within a larger test plan
- Simulate users who spend a fixed amount of time in a specific section
- Combine with other controllers for "run during X seconds, then switch"

#### 🔄 How It Differs from Others
- **vs Loop Controller**: Loop is count-based; Runtime is time-based
- **vs While Controller**: While checks a boolean condition; Runtime uses wall-clock time
- **vs Thread Group Scheduler**: Thread Group scheduler controls the whole group; Runtime controls only its children

#### 🌐 OrangeHRM Scenario

**Scenario**: Simulate users who browse the OrangeHRM directory and employee list for exactly 30 seconds — representing a user who spends half a minute searching before giving up or finding what they need.

**Why this controller for this scenario**: This is useful for measuring throughput: "how many directory searches can OrangeHRM handle in a 30-second burst?" Runtime Controller makes this time-boxed test precise without needing to calculate loop counts.

**Steps in test plan**:
```
Thread Group: Timed Directory Browse [Threads = 5]
├── Once Only Controller: "Login"
│   └── HTTP Request: POST Login
└── Runtime Controller: "Browse Directory for 30s"  [Runtime = 30 seconds]
    ├── HTTP Request: GET Directory (/web/index.php/directory/viewDirectory)
    │   └── Response Assertion: Status 200
    │   └── Duration Assertion: < 3000ms
    └── HTTP Request: GET Employee Search
        └── Response Assertion: Status 200
```

---

### 3.09 If Controller

#### 📌 What Is It?
The **If Controller** is JMeter's conditional branching element. It evaluates a **condition expression** and only executes its children if the condition evaluates to **true**. This enables true if/else branching in test scripts.

#### ⚙️ Key Features
- Condition can use JMeter variables: `"${USER_ROLE}" == "Admin"`
- Supports Groovy expressions (via `__groovy` function)
- Can use `${JMeterThread.last_sample_ok}` to check last request status
- "Evaluate for all children" option: re-evaluates condition for each child
- "Use Sun Javascript Engine": legacy option for JavaScript evaluation
- Can be combined with Beanshell/Groovy for complex logic

#### 🎯 Why We Use It
- Conditional navigation based on user role
- Handle error paths (if login failed, skip remaining steps)
- Simulate different user types in the same Thread Group
- Dynamic test logic based on extracted response values
- Branch based on random values for probabilistic tests

#### 🔄 How It Differs from Others
- **vs Switch Controller**: Switch selects from multiple options by index/name; If is a binary true/false branch
- **vs Throughput Controller**: Throughput is statistical; If is deterministic based on actual conditions
- **vs While Controller**: While loops while condition is true; If executes once when true

#### 🌐 OrangeHRM Scenario

**Scenario**: After login, check if the logged-in user is an Admin. If they are Admin, navigate to the Admin panel and manage users. If they are not Admin, navigate only to the Employee Self-Service portal.

**Why this controller for this scenario**: OrangeHRM has role-based access control. Admin users have different navigation paths than ESS users. The If Controller lets us handle both user types in a single Thread Group using a variable `${USER_ROLE}`.

**Steps in test plan**:
```
Thread Group: Role-Based Navigation [Threads = 5]
├── User Defined Variables: USER_ROLE = Admin
├── HTTP Request: POST Login
│   └── Response Assertion: Redirect to dashboard
├── If Controller: "Admin Path"  [Condition: "${USER_ROLE}" == "Admin"]
│   ├── HTTP Request: GET Admin Panel (/web/index.php/admin/viewSystemUsers)
│   │   └── Response Assertion: Contains "System Users"
│   └── HTTP Request: GET Admin Config
│       └── Response Assertion: Status 200
└── If Controller: "ESS Path"  [Condition: "${USER_ROLE}" == "ESS"]
    └── HTTP Request: GET My Info (/web/index.php/pim/viewMyDetails)
        └── Response Assertion: Status 200
```

---

### 3.10 While Controller

#### 📌 What Is It?
The **While Controller** repeatedly executes its child elements **as long as a specified condition remains true**. It is JMeter's equivalent of a `while` loop — ideal for polling, retrying, or waiting for a condition to change.

#### ⚙️ Key Features
- Condition evaluated before each iteration of children
- Supports `LAST` keyword: `"${LAST}" != "false"` (loop while last sample was OK)
- Can use Groovy/BeanShell expressions for complex conditions
- Can use JMeter variables updated by child extractors
- No hard iteration limit — must be careful to avoid infinite loops
- Combine with a counter/extractor to prevent runaway loops

#### 🎯 Why We Use It
- Poll an endpoint until a record/status changes
- Retry a failed request until it succeeds
- Simulate a user who keeps clicking "Refresh" until data appears
- Wait for background job completion (async processes)
- Implement retry logic for flaky test environments

#### 🔄 How It Differs from Others
- **vs Loop Controller**: Loop is count-based (fixed N); While is condition-based (dynamic)
- **vs If Controller**: If runs children once if true; While runs children repeatedly while true
- **vs Runtime Controller**: Runtime is time-based; While is condition-based

#### 🌐 OrangeHRM Scenario

**Scenario**: Simulate a user who repeatedly checks the Leave Application Status page, refreshing until the leave status changes from "Pending" to "Approved" (up to a maximum of 5 attempts).

**Why this controller for this scenario**: HR processes (leave approvals, job applications) are asynchronous. The While Controller lets us model the realistic polling behavior of an employee checking their leave status, with a counter-based stop condition to prevent infinite loops.

**Steps in test plan**:
```
Thread Group: Leave Status Polling [Threads = 3]
├── Once Only Controller: "Login"
│   └── HTTP Request: POST Login
├── HTTP Request: POST Apply Leave
├── Counter Config: counter_val, start=0, increment=1
└── While Controller: "Poll Until Approved"
    [Condition: "${__groovy(vars.get('counter_val').toInteger() < 5 && vars.get('LEAVE_STATUS') != 'Approved')}"]
    ├── HTTP Request: GET Leave Status
    │   └── JSON Extractor: LEAVE_STATUS from $.data.status
    ├── Response Assertion: Status 200
    └── Constant Timer: 2000ms (wait before retry)
```

---

### 3.11 Switch Controller

#### 📌 What Is It?
The **Switch Controller** is JMeter's equivalent of a `switch/case` statement. It selects **one child to execute** based on the value of a variable. The selection is **deterministic** — you control which child runs by setting a variable.

#### ⚙️ Key Features
- Switch value can be a number (index) or a string (child name)
- If value is a number: selects child by 0-based index
- If value is a string: selects child whose name matches the string
- If value is blank or doesn't match: first child (index 0) is executed
- Combined with User Defined Variables or CSV data for data-driven switching

#### 🎯 Why We Use It
- Route different user types to different test paths
- Data-driven test execution based on CSV input
- Implement menu-driven test scenarios
- Replace multiple If Controllers with cleaner switch logic
- Test multiple distinct workflows in a single thread group

#### 🔄 How It Differs from Others
- **vs If Controller**: If is binary (true/false); Switch handles multiple discrete cases
- **vs Random Controller**: Random is probabilistic; Switch is fully deterministic
- **vs Interleave Controller**: Interleave cycles; Switch chooses based on a variable value

#### 🌐 OrangeHRM Scenario

**Scenario**: Based on the department of the logged-in employee, route them to the relevant HR module: HR Department → Admin Panel, Finance Department → Reports, Operations → Time Module.

**Why this controller for this scenario**: Different employees in OrangeHRM have different primary workflows. Rather than writing three If Controllers, Switch Controller provides a cleaner way to route each user type to their respective module using a `${DEPARTMENT}` variable.

**Steps in test plan**:
```
User Defined Variables: DEPARTMENT = HR  (or Finance, Operations)

Thread Group: Department-Based Navigation [Threads = 3]
├── HTTP Request: POST Login
└── Switch Controller: "Route by Department"  [Switch Value: ${DEPARTMENT}]
    ├── HTTP Request: GET Admin Panel          [Name: "HR"]
    │   └── Response Assertion: Contains "System Users"
    ├── HTTP Request: GET Reports Module       [Name: "Finance"]
    │   └── Response Assertion: Status 200
    └── HTTP Request: GET Time Module          [Name: "Operations"]
        └── Response Assertion: Status 200
```

If `${DEPARTMENT}` = "HR", the Admin Panel request runs.
If `${DEPARTMENT}` = "Finance", the Reports request runs.

---

### 3.12 ForEach Controller

#### 📌 What Is It?
The **ForEach Controller** iterates over a **set of related JMeter variables** that share a common prefix. For each variable in the set, it executes its child elements once, making the current variable's value available inside the loop.

#### ⚙️ Key Features
- Reads variables named `prefix_1`, `prefix_2`, `prefix_3`, ... (1-indexed by default)
- Exports the current value into an output variable for use in samplers
- Start and end index can be configured
- Option to add underscore separator between prefix and index
- Works seamlessly with CSV Data Set Config or extracted arrays
- Can be nested with other controllers

#### 🎯 Why We Use It
- Process a list of employee IDs extracted from a previous response
- Iterate through multiple test data records from CSV
- Submit multiple similar records (e.g., apply leave for a list of employees)
- Create parameterized tests where the input list is dynamic

#### 🔄 How It Differs from Others
- **vs Loop Controller**: Loop repeats N times with no variable context; ForEach iterates over actual data values
- **vs While Controller**: While checks a boolean; ForEach is data-driven iteration
- **vs CSV Data Set Config**: CSV feeds new data per thread iteration; ForEach iterates over pre-loaded variables within one iteration

#### 🌐 OrangeHRM Scenario

**Scenario**: Given a list of Employee IDs (1, 2, 3, 4, 5), visit each employee's profile page to verify all profiles load successfully.

**Why this controller for this scenario**: HR admins frequently bulk-review employee profiles. ForEach Controller lets us define `EMP_ID_1=1`, `EMP_ID_2=2`, etc. and iterate through each, sending a GET request for each employee profile URL dynamically.

**Steps in test plan**:
```
User Defined Variables:
  EMP_ID_1 = 1
  EMP_ID_2 = 2
  EMP_ID_3 = 3
  EMP_ID_4 = 4
  EMP_ID_5 = 5

Thread Group: Employee Profile Verification
├── HTTP Request: POST Login
└── ForEach Controller: "Iterate Employee IDs"
    [Input Variable Prefix: EMP_ID]
    [Output Variable: CURRENT_EMP_ID]
    [Start Index: 1, End Index: 5]
    └── HTTP Request: GET Employee Profile
        Path: /web/index.php/pim/viewPersonalDetails/empNumber/${CURRENT_EMP_ID}
        └── Response Assertion: Status 200
        └── Response Assertion: Contains "Personal Details"
        └── Duration Assertion: < 3000ms
```

Each iteration fetches a different employee profile based on `${CURRENT_EMP_ID}`.

---

### 3.13 Transaction Controller

#### 📌 What Is It?
The **Transaction Controller** groups multiple HTTP requests into a **single logical transaction** and records the **combined elapsed time** of all child requests as one measurement. It is essential for measuring end-to-end business process performance.

#### ⚙️ Key Features
- Records a combined "transaction" sample in results
- "Generate Parent Sample" option: shows rolled-up parent metric in results
- "Include timer duration in generated sample": optionally includes think time
- Children still produce individual samples (unless Generate Parent Sample is on)
- Critical for SLA reporting on multi-step business flows
- Widely used in performance test reports

#### 🎯 Why We Use It
- Measure complete business process duration (e.g., full login flow time)
- Report against SLAs like "Login must complete within 3 seconds"
- Group atomic actions that business stakeholders understand
- Correlate test results to business KPIs
- Standard in enterprise performance testing methodologies (e.g., APDEX)

#### 🔄 How It Differs from Others
- **vs Simple Controller**: Simple Controller is organizational only; Transaction Controller records timing
- **vs If Controller**: If is for conditional logic; Transaction is for time measurement
- **vs Module Controller**: Module Controller reuses fragments; Transaction Controller measures duration

#### 🌐 OrangeHRM Scenario

**Scenario**: Measure the total time it takes for a user to complete the "Full Login Flow" — from landing on the login page to seeing the dashboard — as a single business transaction.

**Why this controller for this scenario**: Performance SLAs for OrangeHRM would be defined at the business level: "The login process must complete in under 3 seconds." Transaction Controller captures this holistically. Without it, you'd see individual request times but not the total perceived user experience time.

**Steps in test plan**:
```
Thread Group: Login Performance Measurement [Threads = 10]
└── Transaction Controller: "TC: Full Login Flow"  [Generate Parent Sample = true]
    ├── HTTP Request: GET Login Page (/web/index.php/auth/login)
    │   └── Response Assertion: Contains "OrangeHRM"
    │   └── Response Assertion: HTTP 200
    ├── HTTP Request: POST Login (/web/index.php/auth/validate)
    │   └── Response Assertion: Redirect 302
    └── HTTP Request: GET Dashboard (/web/index.php/dashboard/index)
        └── Response Assertion: Contains "Dashboard"
        └── Duration Assertion: Whole transaction < 3000ms
```

Results show one "TC: Full Login Flow" sample with the combined time of all 3 requests.

---

### 3.14 Module Controller

#### 📌 What Is It?
The **Module Controller** allows you to **reference and reuse** a test fragment defined elsewhere in the test plan. Instead of duplicating the same sequence of requests (like login) in multiple thread groups, you define it once as a "Test Fragment" and reference it via Module Controller.

#### ⚙️ Key Features
- References any named controller or Test Fragment in the test plan
- Changes to the referenced module propagate everywhere it's used
- Reduces duplication — single source of truth
- Works with Simple Controller, Transaction Controller, or dedicated Test Fragment elements
- Referenced module must be a direct child of the Test Plan or Thread Group

#### 🎯 Why We Use It
- DRY principle (Don't Repeat Yourself) in test plans
- Centralize common flows (login, logout, setup) for reuse
- When the login flow changes, update it in one place, not everywhere
- Modular, maintainable test architecture
- Enable test component libraries

#### 🔄 How It Differs from Others
- **vs Simple Controller**: Simple Controller is a local container; Module Controller references external fragments
- **vs Include Controller**: Include Controller imports from an external .jmx file; Module Controller references within the same plan
- **vs Transaction Controller**: Transaction measures time; Module Controller enables reuse

#### 🌐 OrangeHRM Scenario

**Scenario**: Multiple test scenarios (Employee Test, Leave Test, Admin Test) all need to log in first. Define the login flow once as a Test Fragment and reference it in all three thread groups using Module Controller.

**Why this controller for this scenario**: OrangeHRM's login flow involves 2 requests (GET login page + POST credentials). Without Module Controller, we'd copy this into every thread group. With Module Controller, we define it once and reuse it — when credentials change, we update one place.

**Steps in test plan**:
```
Test Plan
├── Test Fragment: "Login Fragment"  [Disabled - won't run directly]
│   ├── HTTP Request: GET Login Page
│   └── HTTP Request: POST Login
│
├── Thread Group: Employee Tests
│   ├── Module Controller → references "Login Fragment"
│   └── HTTP Request: GET Employee List
│
├── Thread Group: Leave Tests
│   ├── Module Controller → references "Login Fragment"
│   └── HTTP Request: GET Leave Module
│
└── Thread Group: Admin Tests
    ├── Module Controller → references "Login Fragment"
    └── HTTP Request: GET Admin Panel
```

---

### 3.15 Recording Controller

#### 📌 What Is It?
The **Recording Controller** is a **target placeholder** used during JMeter's HTTP(S) Test Script Recorder session. When you record browser traffic through JMeter's built-in proxy, all captured requests are automatically placed inside the Recording Controller.

#### ⚙️ Key Features
- Acts as target container for the HTTP(S) Test Script Recorder
- Does not change execution behavior when tests run
- Automatically populated during recording sessions
- Best practice: move recorded requests out and organize them after recording
- Can be converted to a Simple Controller or Transaction Controller post-recording
- JMeter's proxy records all HTTP/HTTPS traffic passing through it

#### 🎯 Why We Use It
- Quickly capture real user interactions with OrangeHRM as a baseline test
- Reduce manual scripting effort for complex multi-step flows
- Capture exact request payloads, headers, and parameters
- Good starting point for parameterized performance tests

#### 🔄 How It Differs from Others
- **vs Simple Controller**: Recording Controller is for capturing; Simple Controller is for organizing
- **vs Transaction Controller**: Transaction Controller measures time; Recording Controller captures requests
- The Recording Controller is unique — it is the only controller that serves a creation/tooling purpose rather than an execution-flow purpose

#### 🌐 OrangeHRM Scenario

**Scenario**: Record a complete HR workflow by browsing OrangeHRM manually through JMeter's proxy — recording: Login → Navigate to Leave Module → Apply for Leave → Logout.

**Why this controller for this scenario**: For complex workflows with CSRF tokens, form submissions, and session cookies, manual scripting is error-prone. Recording Controller + JMeter Proxy captures all exact HTTP requests with correct headers, payloads, and parameters automatically.

**Steps to use Recording Controller**:
```
1. Add Recording Controller to Thread Group
2. Go to: Test Plan → Add → Non-Test Elements → HTTP(S) Test Script Recorder
3. Set Target Controller = "Recording Controller"  
4. Set Port = 8888
5. Configure browser proxy to localhost:8888
6. Click Start on the Recorder
7. Navigate OrangeHRM in browser:
   - Go to https://opensource-demo.orangehrmlive.com/
   - Login as Admin/admin123
   - Navigate to Leave → Apply
   - Fill form and submit
   - Logout
8. Click Stop on Recorder
9. All requests appear in Recording Controller
10. Post-process: Add assertions, parameterize values, organize into proper controllers
```

---

## 4. OrangeHRM – Scenario Summary Table

| # | Controller | OrangeHRM Scenario | Key Variable/Config |
|---|-----------|-------------------|---------------------|
| 1 | Simple Controller | Group PIM/Employee Management requests | None |
| 2 | Loop Controller | Manager searches 3 employees per session | Loop Count = 3 |
| 3 | Once Only Controller | Login only once across 5 browse iterations | Thread Group Loops = 5 |
| 4 | Interleave Controller | Alternate Leave / Time module access | 2 children, 6 iterations |
| 5 | Random Controller | Random navigation to any HR module | 4 module children |
| 6 | Random Order Controller | All modules accessed in random sequence | 4 module children |
| 7 | Throughput Controller | 70% browse / 30% write transactions | Mode: Percent |
| 8 | Runtime Controller | Browse Directory for exactly 30 seconds | Runtime = 30s |
| 9 | If Controller | Admin → Admin Panel, ESS → My Info | `${USER_ROLE}` |
| 10 | While Controller | Poll leave status until Approved (max 5x) | Counter + JSON Extractor |
| 11 | Switch Controller | Route by department (HR/Finance/Operations) | `${DEPARTMENT}` |
| 12 | ForEach Controller | View 5 employee profiles by ID | `EMP_ID_1..5` variables |
| 13 | Transaction Controller | Measure complete login flow as one metric | Generate Parent Sample = true |
| 14 | Module Controller | Reuse login fragment across thread groups | Test Fragment referenced |
| 15 | Recording Controller | Record live OrangeHRM browsing session | JMeter Proxy port 8888 |

---

## 5. JMX File – Detailed Explanation

The `OrangeHRM.jmx` file is a complete JMeter test plan containing all 13 actively-executable controllers demonstrated above (Module Controller uses a Test Fragment, Recording Controller is structural). Here is the full architecture:

### 🏗️ Test Plan Structure

```
OrangeHRM Controllers Demo (TestPlan)
├── 🌐 HTTP Cookie Manager          ← Global: handles session cookies
├── 🔧 HTTP Request Defaults        ← Global: base URL and protocol settings
├── 📝 User Defined Variables       ← Global: BASE_URL, USERNAME, PASSWORD, etc.
│
├── Thread Group 1: TG1 - Transaction Controller Demo
│   └── Transaction Controller: "TC: Complete Login Flow"
│       ├── GET Login Page + Response Assertion
│       ├── POST Login (credentials) + Response Assertion
│       └── GET Dashboard + Duration Assertion
│
├── Thread Group 2: TG2 - Loop Controller Demo
│   └── Once Only: Login
│   └── Loop Controller x3: Employee Search
│       ├── GET Employee List + Response Assertion
│       └── GET Search Employee + Response Assertion
│
├── Thread Group 3: TG3 - Once Only Controller Demo
│   ├── Once Only Controller: Login (runs once)
│   ├── GET Dashboard + Assertion
│   ├── GET Leave Module + Assertion
│   └── GET Time Module + Assertion
│
├── Thread Group 4: TG4 - If Controller Demo
│   ├── POST Login + Assertion
│   ├── If Controller [Admin Path]: GET Admin Panel
│   └── If Controller [ESS Path]: GET My Info
│
├── Thread Group 5: TG5 - While Controller Demo
│   ├── Once Only: Login
│   ├── POST Apply Leave (mock)
│   └── While Controller: Poll Leave Status (max 5 iterations)
│       ├── GET Leave Status + JSON Extractor
│       └── Constant Timer: 1000ms
│
├── Thread Group 6: TG6 - ForEach Controller Demo
│   ├── Once Only: Login
│   └── ForEach Controller [EMP_ID_1..5]
│       └── GET Employee Profile/${CURRENT_EMP_ID} + Assertions
│
├── Thread Group 7: TG7 - Switch Controller Demo
│   ├── POST Login
│   └── Switch Controller [${DEPARTMENT}]
│       ├── GET Admin Panel [name=HR]
│       ├── GET Reports [name=Finance]
│       └── GET Time Module [name=Operations]
│
├── Thread Group 8: TG8 - Random Controller Demo
│   ├── Once Only: Login
│   └── Random Controller: Navigate to Random Module
│       ├── GET PIM + Assertion
│       ├── GET Leave + Assertion
│       ├── GET Recruitment + Assertion
│       └── GET Performance + Assertion
│
├── Thread Group 9: TG9 - Random Order Controller Demo
│   ├── Once Only: Login
│   └── Random Order Controller: All Modules Random Seq
│       ├── GET PIM + Assertion
│       ├── GET Leave + Assertion
│       ├── GET Time + Assertion
│       └── GET Recruitment + Assertion
│
├── Thread Group 10: TG10 - Throughput Controller Demo
│   ├── Once Only: Login
│   ├── Throughput Controller 70%: Dashboard + Employee List
│   └── Throughput Controller 30%: Leave Application
│
├── Thread Group 11: TG11 - Runtime Controller Demo
│   ├── Once Only: Login
│   └── Runtime Controller [30s]: Directory Browse
│       ├── GET Directory + Assertion
│       └── GET Employee Search + Assertion
│
├── Thread Group 12: TG12 - Interleave Controller Demo
│   ├── Once Only: Login
│   └── Interleave Controller: Alternate Leave & Time
│       ├── GET Leave List + Assertion
│       └── GET Time Module + Assertion
│
├── Thread Group 13: TG13 - Simple Controller Demo
│   ├── Once Only: Login
│   └── Simple Controller: "PIM - Employee Management"
│       ├── GET Employee List + Assertions
│       ├── GET Search Employee + Assertions
│       └── GET Employee Profile + Assertions
│
├── Test Fragment: "Login Fragment"  ← For Module Controller reuse
│   ├── GET Login Page
│   └── POST Login
│
├── Thread Group 14: TG14 - Module Controller Demo
│   ├── Module Controller → "Login Fragment"
│   └── GET Dashboard + Assertion
│
└── 📊 Listeners (at TestPlan level)
    ├── View Results Tree
    ├── Summary Report
    └── Aggregate Report
```

### ⚙️ Global Configuration Elements

#### HTTP Cookie Manager
```
Purpose: Automatically manages session cookies across all requests
Setting: Clear cookies on each iteration = true
Why: OrangeHRM uses session-based authentication. The Cookie Manager
     stores the session cookie after login and sends it with all
     subsequent requests automatically.
```

#### HTTP Request Defaults
```
Server: opensource-demo.orangehrmlive.com
Port: 443
Protocol: https
Encoding: UTF-8
Purpose: Eliminates the need to specify domain/port on every HTTP sampler.
         All Thread Groups inherit these settings.
```

#### User Defined Variables
```
BASE_URL   = opensource-demo.orangehrmlive.com
USERNAME   = Admin
PASSWORD   = admin123
USER_ROLE  = Admin
DEPARTMENT = HR
EMP_ID_1   = 1
EMP_ID_2   = 2
EMP_ID_3   = 3
EMP_ID_4   = 4
EMP_ID_5   = 5
```

---

## 6. Assertions & Validations – How They Were Added

Assertions are validation rules attached to HTTP samplers to verify that the response meets expected criteria. Here is a detailed breakdown of every assertion type used in the JMX file and why:

### 🔵 Type 1: Response Assertion

**What it does**: Validates the response code, response body, response headers, or response message against an expected pattern.

**How it was added in JMX**:
```xml
<ResponseAssertion guiclass="AssertionGui" testclass="ResponseAssertion" 
                   testname="Assert - OrangeHRM Title" enabled="true">
  <collectionProp name="Asserion.test_strings">
    <stringProp name="49586">OrangeHRM</stringProp>
  </collectionProp>
  <stringProp name="Assertion.custom_message">
    Login page must contain OrangeHRM branding
  </stringProp>
  <stringProp name="Assertion.test_field">Assertion.response_data</stringProp>
  <boolProp name="Assertion.assume_success">false</boolProp>
  <intProp name="Assertion.test_type">2</intProp>
</ResponseAssertion>
```

**test_type values**:
| Value | Type |
|-------|------|
| 1 | Matches (regex) |
| 2 | Contains (substring) |
| 4 | Equals |
| 8 | Substring |
| 16 | NOT |

**Where used in OrangeHRM JMX**:
- GET Login Page → Assert response contains "OrangeHRM"
- GET Dashboard → Assert response contains "Dashboard"
- GET Employee List → Assert response body contains "employeeId" or "Employee Information"
- POST Login → Assert HTTP response code is 302 (redirect) or 200
- GET Admin Panel → Assert response contains "System Users"
- GET Leave Module → Assert response contains "Leave"

**In JMeter GUI**:
```
Right-click on HTTP Sampler → Add → Assertions → Response Assertion
→ Pattern: "OrangeHRM"
→ Test Field: Response Body
→ Pattern Matching Rules: Contains
→ Custom Failure Message: "Expected OrangeHRM branding not found"
```

---

### 🟡 Type 2: Duration Assertion

**What it does**: Fails the sample if the response takes longer than the specified duration in milliseconds. Enforces performance SLAs.

**How it was added in JMX**:
```xml
<DurationAssertion guiclass="DurationAssertionGui" testclass="DurationAssertion" 
                   testname="Assert - Response Within 3s" enabled="true">
  <longProp name="DurationAssertion.duration">3000</longProp>
</DurationAssertion>
```

**Where used in OrangeHRM JMX**:
- GET Dashboard → Duration < 3000ms (3 seconds)
- GET Employee List → Duration < 3000ms
- GET Admin Panel → Duration < 5000ms
- ForEach employee profiles → Duration < 3000ms each
- Transaction Controller children → Individual and combined timing

**In JMeter GUI**:
```
Right-click on HTTP Sampler → Add → Assertions → Duration Assertion
→ Duration in milliseconds: 3000
```

---

### 🟠 Type 3: Size Assertion

**What it does**: Validates the response size (in bytes) is within expected bounds. Helps detect truncated responses, empty responses, or unexpectedly large payloads.

**How it was added in JMX**:
```xml
<SizeAssertion guiclass="SizeAssertionGui" testclass="SizeAssertion" 
               testname="Assert - Response Not Empty" enabled="true">
  <stringProp name="Assertion.test_field">SizeAssertion.response_network_size</stringProp>
  <intProp name="SizeAssertion.operator">4</intProp>
  <longProp name="SizeAssertion.size">500</longProp>
</SizeAssertion>
```

**Operator values**:
| Value | Meaning |
|-------|---------|
| 1 | = Equal |
| 2 | != Not equal |
| 3 | > Greater than |
| 4 | < Less than |
| 5 | >= Greater than or equal |
| 6 | <= Less than or equal |

**Where used in OrangeHRM JMX**:
- GET Login Page → Size > 1000 bytes (page must have content)
- GET Employee List → Size > 500 bytes (not an empty response)
- GET Dashboard → Size > 1000 bytes

---

### 🟣 Type 4: JSON Assertion (JSONPath Assertion)

**What it does**: Validates specific JSON fields in the response body using JSONPath expressions. Essential for API testing.

**How it was added in JMX**:
```xml
<JSONPathAssertion guiclass="JSONPathAssertionGui" testclass="JSONPathAssertion" 
                   testname="Assert - Leave Status Field Exists" enabled="true">
  <stringProp name="JSON_PATH">$.data.status</stringProp>
  <stringProp name="EXPECTED_VALUE">Pending</stringProp>
  <boolProp name="JSONVALIDATION">true</boolProp>
  <boolProp name="EXPECT_NULL">false</boolProp>
  <boolProp name="INVERT">false</boolProp>
  <boolProp name="ISREGEX">false</boolProp>
</JSONPathAssertion>
```

**Where used in OrangeHRM JMX**:
- While Controller scenario → Assert `$.data.status` exists in leave status API response
- ForEach employee profiles → Assert response has expected JSON structure

---

### 📐 Assertion Placement Strategy

Each assertion in the JMX was placed **directly inside the hashTree of the HTTP sampler it validates**:

```xml
<HTTPSamplerProxy ... testname="GET Dashboard">
  ...
</HTTPSamplerProxy>
<hashTree>
  <!-- Assertion 1: Content check -->
  <ResponseAssertion ... testname="Assert - Dashboard Content">
    <collectionProp name="Asserion.test_strings">
      <stringProp>Dashboard</stringProp>
    </collectionProp>
  </ResponseAssertion>
  <hashTree/>
  
  <!-- Assertion 2: Performance check -->
  <DurationAssertion ... testname="Assert - Response Time < 3s">
    <longProp name="DurationAssertion.duration">3000</longProp>
  </DurationAssertion>
  <hashTree/>
</hashTree>
```

### 📊 Assertion Coverage Summary

| Thread Group | Assertion Types Used | Purpose |
|-------------|---------------------|---------|
| TG1 - Transaction | Response (text) + Response (status) + Duration | Verify login flow content & timing |
| TG2 - Loop | Response (status) + Response (text) | Verify search returns results |
| TG3 - Once Only | Response (status) + Duration | Verify module pages load |
| TG4 - If | Response (text) + Response (status) | Verify role-based content |
| TG5 - While | Response (status) + JSON Path | Verify status field in API |
| TG6 - ForEach | Response (status) + Duration + Size | Verify profiles load fully |
| TG7 - Switch | Response (text) + Response (status) | Verify correct module loaded |
| TG8 - Random | Response (status) | Verify each module accessible |
| TG9 - Random Order | Response (status) + Duration | Verify all modules load timely |
| TG10 - Throughput | Response (status) + Response (text) | Verify correct page content |
| TG11 - Runtime | Response (status) + Duration | Verify performance during burst |
| TG12 - Interleave | Response (status) | Verify each module accessible |
| TG13 - Simple | Response (text) + Duration + Size | Verify employee data completeness |
| TG14 - Module | Response (status) + Response (text) | Verify login fragment works |

---

## 7. Controller Comparison Table

| Feature | Simple | Loop | Once Only | Interleave | Random | Random Order | Throughput | Runtime | If | While | Switch | ForEach | Transaction | Module | Recording |
|---------|--------|------|-----------|------------|--------|--------------|------------|---------|-----|-------|--------|---------|-------------|--------|-----------|
| **Execution Logic** | None (grouping) | Repeat N times | Once per thread | One per iteration (round-robin) | One random per execution | All in random order | % or count-based | Time-based | Conditional | Condition loop | Index/name select | Data iteration | Groups as transaction | Reuse fragment | Capture tool |
| **Changes Execution?** | ❌ No | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes | ❌ No |
| **Records Timing?** | ❌ No | ❌ No | ❌ No | ❌ No | ❌ No | ❌ No | ❌ No | ❌ No | ❌ No | ❌ No | ❌ No | ❌ No | ✅ Yes | ❌ No | ❌ No |
| **Uses Variables?** | ❌ No | Optional | ❌ No | ❌ No | ❌ No | ❌ No | Optional | ❌ No | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes | ❌ No | ❌ No | ❌ No |
| **Can be Nested?** | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes | ❌ No | ❌ No |
| **Iteration Type** | N/A | Fixed count | Once | Sequential cycle | Random single | Random all | Statistical | Duration | Conditional | Conditional | Deterministic | Data-driven | N/A | Reference | N/A |
| **Runs All Children?** | ✅ Yes | ✅ Yes | ✅ Yes | ❌ One only | ❌ One only | ✅ All (shuffled) | Conditional | ✅ While active | Conditional | Conditional | ❌ One only | ✅ Yes (per item) | ✅ Yes | ✅ Yes | ✅ Yes |
| **Primary Use Case** | Organization | Repeat actions | One-time setup | Balanced alternation | Random load | Order independence | Traffic split | Timed burst | Branching | Polling/retry | Multi-path routing | Data iteration | SLA measurement | Code reuse | Script capture |
| **Configuration** | None | Count | None | Flags | None | None | % or count | Seconds | Expression | Expression | Switch value | Prefix+Output var | Parent sample | Target ref | None |
| **Complexity** | ⭐ Low | ⭐ Low | ⭐ Low | ⭐⭐ Medium | ⭐ Low | ⭐ Low | ⭐⭐ Medium | ⭐ Low | ⭐⭐⭐ High | ⭐⭐⭐ High | ⭐⭐ Medium | ⭐⭐ Medium | ⭐ Low | ⭐⭐ Medium | ⭐ Low |
| **Best For** | Large test plans | Stress testing | Login/setup | Mixed workload | Real-user simulation | Order-independent testing | Realistic traffic | Soak tests | Role-based flows | Async polling | Multi-role routing | Bulk data testing | Business SLAs | DRY test plans | Getting started |
| **Risk of Infinite Loop?** | ❌ No | ❌ No | ❌ No | ❌ No | ❌ No | ❌ No | ❌ No | ❌ No | ❌ No | ⚠️ Yes | ❌ No | ❌ No | ❌ No | ❌ No | ❌ No |

---

### 🎯 Quick Selection Guide

**Use Simple Controller when**: You just want to organize requests into logical groups.

**Use Loop Controller when**: A user needs to repeat the same action N times in one session.

**Use Once Only Controller when**: Login or setup should happen exactly once per virtual user.

**Use Interleave Controller when**: You want to balance load across multiple endpoints over time.

**Use Random Controller when**: You want to simulate unpredictable user navigation.

**Use Random Order Controller when**: All requests must run but the sequence should vary.

**Use Throughput Controller when**: You need a specific % split of user behavior (read vs write).

**Use Runtime Controller when**: You need a time-boxed test of a specific feature.

**Use If Controller when**: Test flow must branch based on a variable or response value.

**Use While Controller when**: You need to poll or retry until a condition changes.

**Use Switch Controller when**: You have multiple distinct paths and need deterministic selection.

**Use ForEach Controller when**: You have a list of data items and need to process each one.

**Use Transaction Controller when**: You need to measure end-to-end business process time.

**Use Module Controller when**: The same sequence (login, logout) is used in multiple test groups.

**Use Recording Controller when**: You want to capture a live browser session as a starting point.

---

*Test Plan File*: `OrangeHRM.jmx`  
*JMeter Version*: 5.6.3+  
*Target Application*: https://opensource-demo.orangehrmlive.com/  
*Default Credentials*: Admin / admin123
