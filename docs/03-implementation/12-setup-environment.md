# **Chapter 12 — Setup & Environment Requirements**

## **12.1 Overview**

This chapter describes the technical requirements and environment configuration necessary to run THE BRIDGE.
Because the system is built on a universal, ontology-driven architecture with CMP–ETY–LOG as its core, the environment must guarantee:

* consistency
* determinism
* stable JSON handling
* correct execution of the navigators
* reliable integration with the Process Manager and Instance Manager

The environment is not domain-specific; it must support the ontology without altering or constraining it.

---

## **12.2 Software Requirements**

### **12.2.1 FileMaker Platform**

THE BRIDGE requires a modern FileMaker platform with:

* **FileMaker Server** (latest stable version recommended)
* **FileMaker Pro** for development
* **FileMaker Go** optionally for mobile execution
* **FileMaker WebDirect** optionally for browser access

The system relies on:

* JSON support (native)
* ExecuteSQL (native)
* Layout object manipulation
* Script parameter and result JSON handling
* PSOS (Perform Script on Server)
* Scheduled scripts
* Container fields (for internal assets)
* WebViewer (for UI rendering and React-based graphs)

Any environment must support these features.

---

### **12.2.2 JavaScript Runtime (for WebViewer Components)**

THE BRIDGE uses JavaScript inside WebViewers to:

* render dynamic UI
* display navigation outputs
* represent the Entity Graph
* host responsive components
* execute JSON-driven rendering

Requirements:

* WebKit-based WebViewer (macOS / iOS)
* Chromium-based WebViewer (Windows)
* ES6 support
* Fetch API for internal calls
* Consistent DOM behavior across platforms

React components (from the appendices) can be embedded if desired, but are optional.

---

### **12.2.3 Server Hardware**

Recommended specifications for FileMaker Server:

* **CPU:** 4+ cores
* **RAM:** 16 GB minimum
* **Storage:** SSD recommended
* **Network:** stable, low-latency connectivity
* **OS:**

  * macOS Server
  * Windows Server
  * Linux (if supported by installed FileMaker Server version)

For large deployments, 32–64 GB RAM is preferable.

---

## **12.3 File Structure Requirements**

Three core tables must exist and remain immutable in name and structure:

* **CMP** Table
* **ETY** Table
* **LOG** Table

All structures must follow the definitions established during bootstrap.

---

## **12.4 JSON Infrastructure**

THE BRIDGE relies heavily on JSON for:

* attribute storage
* operation definitions
* process execution results
* navigation requests
* navigation responses
* UI rendering
* deep entity reconstruction

The environment must support JSON operations consistently.

Required FileMaker JSON functions:

* JSONSetElement
* JSONGetElement
* JSONDeleteElement
* JSONListKeys
* JSONFormatElements

JSON structures must not be altered outside the navigation and process engines.

---

## **12.5 Server Configuration**

### **12.5.1 PSOS (Perform Script on Server)**

Many key operations—especially Process Manager executions—should be run using PSOS for:

* performance
* consistency
* integrity
* avoiding race conditions

The server must allow:

* PSOS execution
* script parameter passing
* JSON results

### **12.5.2 Scheduled Scripts**

Required for:

* automated maintenance
* cleanup routines
* system integrity verification
* optional logging aggregation
* optional ETY compaction processes

Schedules must avoid modifying core tables directly.

---

## **12.6 Optional External Integrations**

External integrations are not required for THE BRIDGE to function, but common optional enhancements include:

* Web APIs (REST endpoints)
* Dashboard rendering services
* Cloud storage for binary assets
* Notification services (email/SMS)

All external systems should communicate via JSON where possible.

---

## **12.7 Environment Safety Rules**

To maintain ontological integrity:

1. **Do not modify CMP table structure after bootstrap.**
2. **Do not manually edit ETY table records.**
3. **Do not delete LOG table entries.**
4. **Do not bypass the Bootstrap.**
5. **Do not bypass the Specific Attributes Manager.**
6. **Do not bypass the Templates / Process Manager.**
7. **Do not bypass the Instance Manager.**
8. **Do not create entities without proper structural definitions.**
9. **Do not alter the structure_id of any record.**
10. **Do not create domain-specific tables.**
11. **Do not implement custom workflows outside OPE logic.**
12. **Do not run scripts that mutate entities without generating LOG entries.**

Violating these rules breaks the ontology.

---

## **12.8 Deployment-Level Requirements**

### **12.8.1 File Hosting**

The system must be hosted in an environment that guarantees:

* uninterrupted uptime
* strong backup strategy
* transactional consistency
* secure transmission (SSL)
* authenticated access

### **12.8.2 Backup Strategy**

Backups must include:

* all three core tables
* MET and OPE definitions
* navigation script folder
* configuration tables
* any embedded JSON assets

Recommended retention: 7–30 days depending on deployment scale.

---

## **12.9 Summary**

THE BRIDGE requires:

* a modern FileMaker environment
* stable JSON handling
* guaranteed consistency
* correct execution of the navigation engines
* secure and stable hosting
* preservation of the three core tables
* an environment free from domain-specific structural drift

Once these requirements are met, the system becomes ready for the next implementation phase.
