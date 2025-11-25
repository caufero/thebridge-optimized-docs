# **CHAPTER 10 — PROCESS (TEMPLATE) MANAGER**

### *The Ontology & Template Engine of The Bridge*

## **10.1 Introduction**

The Process Manager is the **universe generator** — the environment where the organization’s business ontology is defined and compiled into structural templates, attribute definitions, workflows, states, transitions, and triggers.

Through it, managers define the **ASPETTO** of every process (CMP templates), the **NATURA** that governs behavior (workflow + triggers), and the **ENTITÀ** scaffolding (ETY orchestration).
Once saved, these definitions automatically generate the UI, operational logic, and entity lifecycle rules for the entire organization.

---

# **10.2 What the Process Manager Actually Does**

The Process Manager constructs the living ontology of the organization.

When a manager defines a process inside it:

* The structural template is created in **CMP**
* The workflow control layer is created in **ETY**
* Logging patterns are created in **LOG**
* The user interface is generated
* Operational permissions and transitions are enforced

This means:

> **A process defined here = a business capability that now exists.**
> **A process not defined here = a business capability the company does not have.**

In simple terms:

### **The Process Manager defines what the organization is capable of doing.**

---

# **10.3 Multiple-Panel Architecture**

The Process Manager UI is among a number of coordinated panels.
Bootstrap, Attributes, **Templates (Process Manager)**, Instances (Instance Manager), SuperTable, Tests

---

## **10.3.1 Process Hierarchy (TreeView)**

This panel governs the **hierarchical** view of templates.

* Displays all processes in a parent–child tree
* Defines hierarchical and ontological relationships
* Supports:

  * **Add** process
  * **Nest** under parent
  * **Re-order**
  * **Move**
  * **Delete**
* Example hierarchy:
  `COMPANY → TASK → PHONE CALL → FOLLOW-UP`

This is the **ASPETTO vertical axis**.

---

## **10.3.2 Attributes & Meta-Properties**

The details section defines the **internal structure** of a process template.

Here, the following are displayed:

* Attributes (ATR)
* Meta-attributes (MET references)
* Domains
* Validation rules
* Data types
* Default values
* Security & authorization rules
* Searchability
* Temporal rules (versioning, retention)
* Trigger-points inside attributes
* UI representation

Once attributes are defined:

### The Bridge can automatically generate the UI for the instance manager.

No layout-building is necessary.
The template itself **is** the form definition.

---

## **10.3.3 Composition (Workflow, Relationships & Triggers)**

The third sections defines how the process *behaves*.

Here, the following are displayed:

* **Workflow states**
* **Transitions**
* **Conditions**
* **Automatic triggers**
* **Manual triggers**
* **Dependencies between processes**
* **Inter-process orchestration**
* **Operational relationships**

This panel defines the **NATURA** of the process:
its behavior, reactions, and life cycle.

---

# **10.4 What the Process Manager Generates Internally**

When a manager presses **Save**, the system compiles the ontology into four layers:

---

## **10.4.1 CMP (ASPETTO)**

The template is compiled into a structural JSON object:

* Structural definition
* Attribute list
* Domain rules
* Data types
* Allowed operations
* Validation rules
* UI generation map
* Trigger definitions
* Workflow structure

This becomes the **template record** for future instances.

---

## **10.4.2 ETY (ENTITÀ)**

The Process Manager defines:

* Initial workflow state
* Allowed actions
* Controllers and permissions
* Orchestration rules
* Timer rules
* State conditions

This becomes the **operational control layer** for all instances.

---

## **10.4.3 LOG (NATURA)**

Based on the definitions:

* Atomic logs are created for attribute changes
* Activity logs are created for workflow transitions
* Process logs are created for start/end events
* Trigger logs are generated for automation

Every behavioral rule defined in the Process Manager becomes **traceable**.

---

## **10.4.4 Generated UI**

The Bridge automatically creates:

* Forms
* Input components
* Buttons and actions
* State transitions
* Auto-layout structures
* Domain validation mappings
* Trigger execution hooks

This UI becomes what users see in the **Instance Manager**.

---

# **10.5 Example: Defining the “Phone Call Management (PHO)” Process**

The example definition you provided is *exactly* what belongs inside the Process Manager:

```json
{
  "process": {
    "code": "PHO",
    "name": "Phone Call Management",
    "description": "Manages inbound and outbound customer phone calls",
    "version": "1.0",
    "family": "COMMUNICATION",
    "standard_duration": 300,
    "k_target": 1.5,
    "business_value": "high",
    "frequency": "50-100/day"
  }
}
```

Once defined:

* Panel 2 would define attributes like `caller_name`, `phone_number`, `outcome`
* Panel 3 would define workflow states like `NEW → IN_PROGRESS → COMPLETED`
* Triggers would define:
  `"IF outcome = INTERESTED THEN create_offer"`

And once saved:

* The template is created in CMP
* The workflow scaffold is created in ETY
* LOG behavior is configured
* The UI is generated for users to create PHO instances

---

# **10.6 Relationship with Instance Manager**

It is important to distinguish the two roles clearly:

### **Process Manager = Define ontology (Templates)**

### **Instance Manager = Execute ontology (Instances)**

The Process Manager is used **once per process type**.
The Instance Manager is used **every day by operators**.

This separation ensures:

* Organizational stability
* Template consistency
* Low K coefficient
* Predictable UI generation
* Full traceability across CMP–ETY–LOG

---

# **10.7 Summary**

The Process Manager is:

* The **ontology editor**
* The **template engine**
* The **structural designer**
* The **workflow compiler**
* The **trigger configurator**
* The **domain governor**
* The **origin point of CMP / ETY / LOG**
* The **source of all user interfaces**
* The **definer of business capability**

Nothing in the operational system exists unless it is **first defined here**.
