# Project: Log File Analysis with SPlunk

## 📋 Executive Summary
Built a simulated Security Information and Event Management (SIEM) pipeline on Linux and Windows environments using Splunk Enterprise to ingest, parse, query, and visualize Linux system logs (`Linux_2k.log`). Combined manual log analysis, custom Python scripting, and Splunk Search Processing Language (SPL) to detect unauthorized access attempts, identify brute-force patterns, and map attack timelines targeting administrative accounts.

---

## 🛠️ Architecture & Tech Stack
* **Operating System:** Windows 10 / Linux (Ubuntu Environment)
* **Log Sources:** `Linux_2k.log` (from LogHub)
* **Automation/Scripting:** Python (`log.py` for extraction and Excel export)
* **Visualization/SIEM Tool:** Splunk Enterprise (v10.6.1.1)
* **Supporting Tools:** VS Code, Microsoft Excel

---

## ⚙️ Step-by-Step Implementation

### Step 1: Environment Setup & Configuration
* Registered and downloaded **Splunk Enterprise** (v10.6.1.1 Windows 64-bit installer) from the official Splunk trial page https://www.splunk.com/en_us/download.html .
* When selecting the installation package, choose the one that suits your setup; in this case, I chose the Windows version.
* After downloading, begin installing the downloaded file. You may be prompted to create an admin account before installation. Choose a username and password that are easy to remember.
* Installed Splunk Enterprise and configured administrative credentials (`admin`).
* Open Splunk, which will direct you to the login page for your existing account. After you enter your username and password, you will be taken to the administrator dashboard.

---

### Step 2: Log Collection & Manual Analysis
* Downloaded the raw log file `Linux_2k.log` from the open-source [LogHub repository](https://github.com/logpai/loghub/tree/master/Linux).

---

### Step 3: Splunk Data Ingestion & Source Configuration
1. In Splunk Enterprise, navigated to **Settings → Add Data → Upload**.
2. Uploaded `Linux_2k.log` (renamed to `Linux2k.log` if path/string concatenation issues arose due to filenames).
3. Set Source Type parameters:
   * **Source Type:** Custom (`Linux`)
   * **Description:** Log Analysis
   * **Category:** Custom
   * **App:** Search & Reporting (`search`)
4. Configured **Input Settings**:
   * **Host:** Constant value (in my case, host field value is `LAPTOP-494NK9CQ` -- Splunk will recognize your machine).
   * **Index:** `Default`
5. Reviewed configuration and submitted data for indexing.

---

### Step 4: Search & Querying in Splunk
Executed targeted searches using Splunk Search Processing Language (SPL) to filter security and authentication events:

```spl
source="Linux_2k.log" host="LAPTOP-494NK9CQ" sourcetype="Linux" ("Failed password" OR "authentication failure" OR "invalid user" OR "user unknown")
```

**Query Results:**
* Successfully isolated **608 events** indicating brute-force or unauthorized access attempts out of the initial indexed dataset.

---

### Step 5: SIEM Visualization & Dashboards

Using Splunk's native analytical tabs (**Events**, **Patterns**, **Statistics**, and **Visualization**), key attack vectors were isolated:

#### 1. Events View (Source Identification)
* Flagged repeated authentication failures for high-value targets (specifically `user=root`).
* Identified primary malicious actor IP address: `207.243.167.114`.
* Observed high-frequency login attempts occurring within seconds of each other (e.g., multiple failures at `07:04:02`, `07:04:05`, `07:04:07`, `07:04:12`).

#### 2. Patterns View (Cluster Analysis)
* Splunk event clustering revealed dominant pattern groups:
  * `authentication failure; logname= uid=0 euid=0 ... user=root` (representing ~80.59% of flagged events).
  * `check pass; user unknown` (representing ~19.24% of flagged events).
* Confirmed systematic, non-random attack signatures.

#### 3. Statistics & Visual Timeline
* Generated structured tables and line charts mapping authentication failures over time using **Table View** and **Pivot Line Charts**.
* **Chart Insights:**
  * Displays distinct vertical spikes in failure rates during specific time windows.
  * Highlights bursty, automated brute-force behavior characteristic of botnets or scripted attacks attempting privilege escalation.

---

## 🧪 Testing & Validation
* **Ingestion Validation:** Verified that 2,000 raw log events were fully ingested and indexed without timestamp parsing errors.
* **Search Accuracy:** Cross-checked Python keyword extraction (652 results) against Splunk SPL filtering (608 strict authentication failure events) to validate low false-positive rates.
* **Threat Identification:** Successfully isolated key compromised indicators:
  * Top Targeted Account: `root`
  * Primary Attacking IP: `207.243.167.114` (along with scattered secondary IPs like `150.183.249.110`).

---

## 💡 Key Learnings & Challenges

* **Challenge:** Encountered `Upload failed with ERROR: can only concatenate str (not 'NoneType') to str` during data ingestion in Splunk.
  * **Solution:** Resolved by stripping special characters and spaces from the filename (`Linux_2k.log` → `Linux2k.log`) and re-uploading via standard file stream.
* **Challenge:** Differentiating between background system alerts and active external attack attempts in raw logs.
  * **Solution:** Used targeted multi-keyword boolean queries (`OR`) and visual event clustering in Splunk to group attack signatures.
* **Key Takeaway:** Gained hands-on experience bridging manual Python script-based parsing with enterprise SIEM platforms like Splunk to scale log investigation in a Security Operations Center (SOC).

---

## 🚀 Future Improvements
1. **Automated Alerting:** Configure real-time Splunk alert triggers (e.g., send email/webhook notifications when failed login attempts exceed 10 within 1 minute).
2. **GeoIP Mapping:** Integrate Splunk IP location lookups to map foreign malicious IP locations visually on a world map.
3. **Live Forwarding:** Setup Splunk Universal Forwarder on Linux virtual machines to stream `/var/log/auth.log` live rather than uploading static batch files.