# **Chapter 17 — Deployment**

## **17.1 Overview**

Deployment in THE BRIDGE is the process of taking the triadic CMP–ETY–LOG architecture, the navigators, and the UI/WebViewer framework, and placing them into a **stable, secure, and high-performance FileMaker Server environment**.

Unlike traditional deployments, THE BRIDGE does **not** require domain-specific configuration, custom tables, or migration scripts.
Deployment focuses on:

* ensuring ontological integrity
* ensuring performance of JSON operations
* ensuring the navigators run consistently
* ensuring WebViewer components load correctly
* ensuring PSOS executes without bottlenecks
* ensuring LOG is immutable and stored safely
* ensuring backups preserve the triad

This chapter describes the recommended deployment pipeline, server configuration, performance tuning, backups, and post-deployment checks.

---

# **17.2 Deployment Pipeline**

Deployment occurs in five stages:

1. **Prepare the FileMaker Server environment**
2. **Upload the core file(s)**
3. **Run bootstrap integrity checks**
4. **Publish domain templates (CMP)**
5. **Enable live operations (PTY, OPE, UI)**

### **17.2.1 Step 1 — Environment Preparation**

Ensure the server meets required conditions:

* FileMaker Server latest stable version
* SSL enabled
* Container streaming enabled
* Scripting engine active
* PSOS allowed
* Schedules enabled

If using cloud hosting (e.g., FM Cloud, AWS, macOS server), match specs described in Chapter 12.

---

## **17.3 Uploading the Core Files**

THE BRIDGE normally deploys as:

* one core file containing CMP, ETY, LOG, Instance Manager, Process Manager, navigators, JSON utilities, and UI layouts
* optional secondary UI files (if any)
* optional domain-specific modular extensions (templates only, not tables)

### **17.3.1 Recommended Deployment Structure**

```
TheBridge.fmp12 (File Name)
```

### **17.3.2 Upload Procedure**

1. Open FileMaker Server Admin Console
2. Go to **Databases > Upload Database**
3. Upload TheBridge.fmp12
4. Ensure it opens without errors
5. Activate Web Direct so it can also be accessed through the browser
6. Set **Auto-Open** ON

### **17.3.3 Validate Database Encryption (If Applied)**

If the file is encrypted:

* verify the encryption key loads correctly
* ensure the Admin Console unlocks it on server start
* verify no decryption failures in schedule logs

---

# **17.4 Initial Server Script Configuration**

THE BRIDGE requires several server-side configurations to run efficiently.

### **17.4.1 Required Server Scripts**

1. **LOG Backup Consolidation**
2. **CMP / ETY Consistency Scan**
3. **Orphan Detection (optional)**
4. **Cache Cleaning (optional)**
5. **Rebuild UI View Models (if used for dashboards)**

Create schedules in FileMaker Server Admin Console:

| Script                   | Frequency | Purpose                           |
| ------------------------ | --------- | --------------------------------- |
| LOG_Consolidate          | Daily     | Archive logs for analytics        |
| System_Integrity_Check   | Daily     | Validate CMP/ETY structures       |
| Refresh_Dashboard_Caches | Hourly    | Optional performance optimization |
| Backup_Database          | Daily     | Core triad backup                 |

---

# **17.5 WebViewer Deployment Considerations**

### **17.5.1 CDN Asset Loading**

If your UI loads external JS/CSS (like React, Tailwind, Chart.js), you must ensure:

* no corporate firewall blocks CDN
* HTTPS is enforced
* Fallback local versions exist if needed

### **17.5.2 Data URLs vs Embedded HTML**

For reliability, the recommended strategy is:

* **Embed HTML internally in global fields** (UI::g_HTML)
* Inject VIEW_MODEL via `data:` URL or substitution

This keeps deployments self-contained.

---

# **17.6 JSON Performance Tuning**

Large deployments of THE BRIDGE use heavy JSON manipulation.
Recommendations:

### **17.6.1 Enable “WiredTiger” (macOS/Linux)**

If using FM Server on Linux/macOS with Mongo microservice extensions:
(not standard FM feature, optional)

### **17.6.2 Limit Deep ETY Queries**

Use indexes:

* dna_id
* structure_id
* breadcrumb_id

### **17.6.3 Precompute Partial JSON (Optional)**

Sometimes a “VIEW_CACHED” JSON field can store UI-specific precomputations to avoid repetitive deep parsing.

---

# **17.7 PSOS Optimization**

PSOS (Perform Script on Server) is central to how THE BRIDGE executes complex OPE chains.

### **17.7.1 Recommendations**

1. Keep PSOS scripts **stateless**
2. Avoid opening many related records
3. Send only JSON
4. Return only JSON
5. Do not manipulate layouts unnecessarily
6. Remove unnecessary window operations
7. Never store PSOS results in global fields

### **17.7.2 Example PSOS Execution Snippet**

```
// Client-side script
Set Variable [ $payload ; JSONSetElement ( "{}" ; "dna_id" ; $dna ; JSONString ) ]

Perform Script on Server [ “Universal_Processor” ; Parameter: $payload ]

Set Variable [ $result ; Value: Get ( ScriptResult ) ]

If [ JSONGetElement ( $result ; "status" ) = "ERROR" ]
  Show Custom Dialog [ "Process Error" ; JSONGetElement ( $result ; "message" ) ]
End If
```

---

# **17.8 Security Configuration**

Security is critical because CMP, ETY, and LOG contain the ontology and entire system history.

### **17.8.1 Required Security Controls**

* Require **Admin Console** password
* Use **external authentication** for users
* Restrict privilege sets:

  * Users should never see CMP_TABLE or LOG_TABLE
  * Users should only interact through UI
  * Developers may have full access
* Disable Guest account
* Enforce SSL
* Use encryption-at-rest for the database file

### **17.8.2 Recommended Privilege Sets**

| Role       | Access                                    |
| ---------- | ----------------------------------------- |
| Admin      | Full access                               |
| Developer  | Full access but cannot delete CMP entries |
| Operator   | Read/write ETY, navigate only             |
| Viewer     | Read-only UI                              |
| API / PSOS | Restricted script access                  |

---

# **17.9 Backup Strategy**

The triad must be preserved:

* CMP Table
* ETY Table
* LOG Table

### **17.9.1 Backup Intervals**

* **Daily full backup**
* **Weekly offsite backup**

### **17.9.2 Example Backup Folder Structure**

```
/backups/daily/TheBridge_2025-11-23.fmp12
/backups/offsite/weekly/TheBridge_2025-W47.fmp12
```

### **17.9.3 Post-Backup Validation Script**

After backup completes:

```
// Backup_Verify
If [ Get (LastError) ≠ 0 ]
  Send Mail [ “admin@domain.com” ; “Backup Failed” ; Get (LastError) ]
Else
  Perform Script [ “System_Integrity_Check” ]
End If
```

---

# **17.10 Deployment Checklist**

### **Server**

* [ ] Latest FileMaker Server
* [ ] SSL enabled
* [ ] PSOS enabled
* [ ] Schedules enabled
* [ ] External authentication configured
* [ ] Backups configured

### **File(s)**

* [ ] Core file uploaded
* [ ] File auto-open enabled
* [ ] Encryption key stored
* [ ] UI file uploaded (if applicable)
* [ ] Template definitions loaded

### **Application**

* [ ] CMP integrity validated
* [ ] ETY integrity validated
* [ ] LOG integrity validated
* [ ] UI rendering tested
* [ ] Navigation tested (X, Y, Z)
* [ ] Operations tested (OPE)
* [ ] PSOS load verified

---

# **17.11 Summary**

Deployment of THE BRIDGE requires:

* preparing a clean and secure server environment
* uploading the core architecture
* verifying CMP–ETY–LOG integrity
* enabling PSOS and JSON tooling
* configuring backups and schedules
* validating UI and Data API behavior
* running performance and reliability checks

Once deployed, the system becomes a universal engine ready to support any domain via templates, attributes, and operations — without changing the architecture.
