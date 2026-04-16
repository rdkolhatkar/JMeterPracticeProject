# Steps to Convert JMeter Non-GUI (.jtl) Results into Reports

---

## METHOD 1 : Generate HTML Dashboard Report (Built-in JMeter Feature)
> **This is the BEST and most recommended method.** It gives you graphs, charts, response time trends, throughput, error %, percentiles and much more.

1) Make sure your test has already been executed in Non-GUI mode and the results.jtl file has been generated at your reports path.
   Example: `D:\JMeterPracticeProject\Reports\results.jtl`

2) Open the command prompt (administrator mode) or git bash and navigate to the JMeter bin directory by running the command:
   ```
   cd D:\JMeterPracticeProject\apache-jmeter-5.6.3\apache-jmeter-5.6.3\bin
   ```

3) Run the following command to generate the HTML Dashboard Report from the existing .jtl results file:
   ```
   jmeter -g <Path to .jtl file> -o <Path to output HTML report folder>
   ```
   Example:
   ```
   jmeter -g D:\JMeterPracticeProject\Reports\results.jtl -o D:\JMeterPracticeProject\Reports\HTMLReport
   ```

4) **Important Note:** The output folder (HTMLReport) must be **EMPTY** or must **NOT exist** before running the command. If it already exists and has content, JMeter will throw an error. Either delete the folder or give a new folder name each time.

5) After the command runs successfully, go to the output folder:
   `D:\JMeterPracticeProject\Reports\HTMLReport`

6) Open the `index.html` file in any browser (Chrome, Edge, Firefox).

7) The HTML Dashboard will show you the following sections and graphs:
   - Test and Report Information
   - APDEX (Application Performance Index) Table
   - Requests Summary (Pass/Fail Pie Chart)
   - Statistics Table (Samples, Average, Min, Max, 90th/95th/99th Percentile, Error %, Throughput, Received KB/s, Sent KB/s)
   - Response Times Over Time (Graph)
   - Response Time Percentiles (Graph)
   - Active Threads Over Time (Graph)
   - Bytes Throughput Over Time (Graph)
   - Latencies Over Time (Graph)
   - Connect Time Over Time (Graph)
   - Hits Per Second (Graph)
   - Codes Per Second (Graph)
   - Transactions Per Second (Graph)
   - Response Time vs Request (Graph)
   - Latency vs Request (Graph)
   - Response Time Distribution (Graph)

---

## METHOD 2 : Generate HTML Report DURING the Test Run (Single Command)
> Use this method when you want to run the test AND generate the HTML report in a single command.

1) Open command prompt (administrator mode) or git bash and navigate to the JMeter bin directory:
   ```
   cd D:\JMeterPracticeProject\apache-jmeter-5.6.3\apache-jmeter-5.6.3\bin
   ```

2) Run the following combined command:
   ```
   jmeter -n -t <TestFile.jmx> -l <Path to .jtl file> -e -o <Path to HTML report folder>
   ```
   Example:
   ```
   jmeter -n -t BlazeDemoWebsiteProject.jmx -l D:\JMeterPracticeProject\Reports\results.jtl -e -o D:\JMeterPracticeProject\Reports\HTMLReport
   ```

3) Explanation of the flags used in the command:

   | Flag | Description |
   |------|-------------|
   | `-n` | Run JMeter in Non-GUI (headless) mode |
   | `-t` | Specifies the Test Plan (.jmx) file to run |
   | `-l` | Specifies the path where the .jtl result file will be saved |
   | `-e` | Tells JMeter to generate the HTML Dashboard Report after the test |
   | `-o` | Specifies the output folder where the HTML report will be saved |

4) **Important Note:** Same as Method 1, the output folder specified after `-o` must be **EMPTY** or must **NOT exist** before running the command.

5) After the test completes, open the `index.html` file from the output folder in any browser to view the full graphical report.

---

## METHOD 3 : Convert .jtl Results to EXCEL (Tabular Sheet)
> Use this method if you want the raw results data in Excel format for custom analysis, filtering or sharing.

### OPTION A : Open .jtl Directly in Excel

1) Navigate to the location of your .jtl file:
   `D:\JMeterPracticeProject\Reports\results.jtl`

2) The .jtl file is actually a CSV (Comma Separated Values) file in plain text format. You can open it directly in Microsoft Excel.

3) Right-click on the `results.jtl` file.

4) Select "Open With" and then choose "Microsoft Excel".

5) If Excel does not auto-detect columns, do the following:
   - a) Open Excel manually.
   - b) Go to File > Open and browse to your results.jtl file. (Make sure to change the file type filter to "All Files (*.*)" to see .jtl files)
   - c) The "Text Import Wizard" will open.
   - d) Select "Delimited" and click Next.
   - e) Check "Comma" as the delimiter and click Next.
   - f) Click Finish.

6) The .jtl file will now open in Excel as a tabular sheet with the following columns (default JMeter .jtl columns):

   | Column | Description |
   |--------|-------------|
   | `timeStamp` | Time of the sample in milliseconds (epoch) |
   | `elapsed` | Time taken for the request (in ms) |
   | `label` | Name of the Sampler / Request |
   | `responseCode` | HTTP Response Code (e.g. 200, 404, 500) |
   | `responseMessage` | HTTP Response Message (e.g. OK) |
   | `threadName` | Name of the Thread Group and thread number |
   | `dataType` | Type of data returned |
   | `success` | true or false (whether request passed or failed) |
   | `failureMessage` | Error message if the request failed |
   | `bytes` | Number of bytes in the response |
   | `sentBytes` | Number of bytes sent in the request |
   | `grpThreads` | Number of active threads in the thread group |
   | `allThreads` | Total number of active threads |
   | `URL` | The URL that was hit |
   | `Latency` | Time to first byte received (in ms) |
   | `IdleTime` | Time spent idle (in ms) |
   | `Connect` | TCP connection time (in ms) |

7) Save the file as an Excel Workbook (.xlsx) by going to:
   File > Save As > Choose ".xlsx" format.

8) You can now apply Excel features like:
   - Filters and Sorting
   - Pivot Tables for summary by label/request name
   - Conditional Formatting to highlight failures
   - Charts and Graphs (Bar, Line) using the elapsed/responseCode columns

---

### OPTION B : Use a Simple Python Script to Convert .jtl to .xlsx

1) Make sure Python is installed on your machine. Check by running:
   ```
   python --version
   ```

2) Install the required library by running:
   ```
   pip install openpyxl pandas
   ```

3) Create a Python file (e.g. `convert_jtl.py`) and paste the following code:
   ```python
   import pandas as pd

   jtl_file  = r"D:\JMeterPracticeProject\Reports\results.jtl"
   xlsx_file = r"D:\JMeterPracticeProject\Reports\results.xlsx"

   df = pd.read_csv(jtl_file)
   df.to_excel(xlsx_file, index=False)
   print("Conversion complete! File saved at:", xlsx_file)
   ```

4) Run the script from command prompt:
   ```
   python convert_jtl.py
   ```

5) Open the generated `results.xlsx` file in Excel. All your .jtl results will now be in a clean, properly formatted Excel tabular sheet.

---

## METHOD 4 : View Results Using JMeter GUI (Listeners)
> Use this method if you want to load the .jtl file back into the JMeter UI and see results through the built-in Listeners.

1) Open JMeter UI by running the command in the bin directory:
   ```
   jmeter
   ```

2) Open your existing Test Plan:
   `File > Open > Select "BlazeDemoWebsiteProject.jmx"`

3) Add a Listener to your Test Plan. Right-click on the Thread Group or Test Plan > Add > Listener and choose any of the following:
   - **View Results Tree** : Shows each request and response in detail
   - **View Results in Table** : Shows results in a tabular format
   - **Summary Report** : Shows aggregate stats per label
   - **Aggregate Report** : Shows min, max, avg, 90%, 95%, 99% percentile
   - **Graph Results** : Shows a basic response time graph
   - **Response Time Graph** : Shows response time over time per label
   - **jp@gc - Transactions per Second** (if JMeter Plugins installed)

4) In the Listener, look for the "Browse" button or "Filename" input field at the top of the Listener panel.

5) Click "Browse" and select your existing .jtl results file:
   `D:\JMeterPracticeProject\Reports\results.jtl`

6) Click the small folder/load icon or press Enter. JMeter will load all the results from the .jtl file into the Listener automatically.

7) You can now view all past test results in graphical and tabular format inside the JMeter UI without re-running the test.

---

## METHOD 5 : Use JMeter Plugins for Advanced Graphs
> Install JMeter Plugins Manager for extra graphs and listeners.

1) Download the JMeter Plugins Manager .jar file from:
   https://jmeter-plugins.org/install/Install/

2) Copy the downloaded `jmeter-plugins-manager-X.X.jar` file to the lib/ext directory of JMeter:
   `D:\JMeterPracticeProject\apache-jmeter-5.6.3\apache-jmeter-5.6.3\lib\ext`

3) Restart JMeter UI.

4) Go to Options > Plugins Manager.

5) Search for and install the plugin set called "JMeter Plugins - Standard Set" or "3 Basic Graphs". This includes:
   - jp@gc - Transactions per Second
   - jp@gc - Response Times Over Time
   - jp@gc - Active Threads Over Time

6) After installation, restart JMeter and add these new Listeners to your Test Plan. Load your .jtl file into them as described in Method 4 Step 5.

---

## QUICK REFERENCE COMMAND SUMMARY

Run test (Non-GUI) and save .jtl only:
```
jmeter -n -t BlazeDemoWebsiteProject.jmx -l D:\JMeterPracticeProject\Reports\results.jtl
```

Generate HTML report from existing .jtl:
```
jmeter -g D:\JMeterPracticeProject\Reports\results.jtl -o D:\JMeterPracticeProject\Reports\HTMLReport
```

Run test + Generate HTML report in one command:
```
jmeter -n -t BlazeDemoWebsiteProject.jmx -l D:\JMeterPracticeProject\Reports\results.jtl -e -o D:\JMeterPracticeProject\Reports\HTMLReport
```

---

## IMPORTANT NOTES / TIPS

1) The HTML report output folder must always be **EMPTY** or **NON-EXISTENT** before running the `-o` or `-e -o` command. Delete the old folder and create a fresh one, or use a new folder name with a timestamp to avoid conflicts.
   Example folder names: `HTMLReport_Run1`, `HTMLReport_Run2`, `HTMLReport_20240416`

2) If you get an error saying the output directory is not empty, delete all files inside the output folder and re-run the command.

3) The .jtl file is essentially a CSV file. Any tool that can read CSVs (Excel, Google Sheets, Power BI, Tableau) can open and analyze it.

4) For large .jtl files with thousands of rows, using the Python script (Method 3, Option B) or the HTML report (Method 1 or 2) is preferred over opening directly in Excel.

5) You can open the HTML report `index.html` in any browser. No internet connection is required as all assets are stored locally in the report folder.

6) The JMeter bin directory path used in all examples:
   `D:\JMeterPracticeProject\apache-jmeter-5.6.3\apache-jmeter-5.6.3\bin`

7) Always run command prompt in **Administrator mode** to avoid permission issues while writing output files and folders.
