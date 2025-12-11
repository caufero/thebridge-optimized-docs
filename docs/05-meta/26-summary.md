# **Phase 2 Proposal – THE BRIDGE 3P3 Implementation in FileMaker**

Prepared for: **KOOLTOOL SRL (Luca Meggiolaro)**

Prepared by: **Caufero (Cyril Amegah & Osbert Vulor)**

---

## 1. Context and Objective

Phase 1 delivered the ontological and technical foundation of **THE BRIDGE – 3P3 Ontological System** in FileMaker, including:

* A full description of the **3P3 ontology**: ASPETTO (Structure), NATURA (Process), ENTITÀ (Being) and their implementation as **CMP, LOG, ETY** tables. 
* The **sacred identity system**: `DNA_ID`, `STRUCTURE_ID`, `BREADCRUMB_ID`. 
* The **universal attribute and operation system**: 56 MET, 56 OPE, ATR as reusable specific attributes. 
* The **minimal architecture**: 3 tables, 4 Universal Navigators, 1 SuperTable layout. 

The objective of **Phase 2** is to turn this architecture into a working FileMaker implementation that:

* Uses **3 tables only** (CMP, ETY, LOG) as the universal model for all domains. 
* Implements the **Universal Navigators** and the **SuperTable UI**. 
* Delivers at least one full **reference domain** (PHO) end to end, from bootstrap to attributes to templates to business data, to validate the 3P3 system in a real scenario.

---

## 2. Scope of Development Work

### 2.1 Core Triadic Engine (CMP / ETY / LOG)

We will implement the three core tables exactly as defined in Phase 1:

* **CMP (ASPETTO – Structure)**

  * Store every template, structural instance and business instance.
  * Store MET and ATR composition, allowed OPE, STRUCTURE_ID, and other structural coordinates. 

* **ETY (ENTITÀ – State)**

  * Store the current JSON state of each entity, lifecycle status, current relations and operator context. 

* **LOG (NATURA – Process and History)**

  * Store every operation, before and after JSON, actor, timestamp and structural coordinates in an immutable way. 

Implementation tasks:

* Implement `DNA_ID`, `STRUCTURE_ID`, `BREADCRUMB_ID` as the **only identity system** for entities. 
* Implement core JSON fields for CMP, ETY and LOG in line with the spec (structure JSON in CMP, state JSON in ETY, snapshots in LOG). 
* Implement the append-only rule for LOG and internal consistency checks between CMP, ETY and LOG.

### 2.2 Universal Attribute and Operation System (MET, ATR, OPE)

We will operationalise the universal grammar:

* Create **MET table** and **OPE table** with 56 meta-attributes and 56 operations, respecting the 1:1 correspondence. 
* Implement **ATR** as reusable attributes that can be composed into templates in CMP. 
* Implement the **MET–OPE symmetry rule** in the engine so that every MET always has exactly one OPE and vice versa. 

Implementation tasks:

* Build bootstrap routines to load the 56 MET × 3 perspectives and 56 OPE × 3 perspectives (336 records) as described in the Bootstrap section. 
* Implement validation rules that prevent any MET or OPE from being created or used outside the symmetry rules.

### 2.3 Template and Instance Layer

This layer turns the ontology into real working structures:

* **Templates in CMP**

  * Implement templates as CMP records that define MET + ATR composition, structural shape and allowed operations. 

* **Entities as CMP + ETY**

  * Implement entity creation as a coordinated creation of CMP structural instance and ETY state instance for each DNA_ID. 

* **Instance generation logic**

  * Use STRUCTURE_ID, MET and ATR to automatically build ETY JSON skeletons consistent with the template definition. 

Implementation tasks:

* Implement **template designer UI** (Process Manager part) that lets a user define or refine templates by composing MET and ATR and storing them into CMP.
* Implement **instance generation routines** that create initial ETY JSON from a chosen template in CMP, respecting all structural rules.

### 2.4 Universal Navigators and Universal Processor

We will implement the four scripts described in the architecture chapter:

* **Navigate_X** – performs attribute mutations on MET and ATR. 
* **Navigate_Y** – handles creation and filtering of entities. 
* **Navigate_Z** – traverses hierarchy along BREADCRUMB_ID. 
* **Universal_Processor** – orchestrates sequences of X, Y, Z navigations to implement full processes. 

Implementation tasks:

* Implement these navigators as FileMaker scripts that read CMP and ETY JSON, perform transformations consistent with MET/OPE rules, and write to ETY and LOG.
* Implement a **process definition pattern** where processes are declared as sequences of Navigator calls, rather than ad hoc scripts per domain.

### 2.5 SuperTable UI

We will implement the single universal layout described as **SuperTable**:

* A single layout that can display any entity, template or structural form using fixed column coordinates and JSON driven rendering. 

Implementation tasks:

* Implement the SuperTable as one FileMaker layout with Web Viewer based UI that reads CMP and ETY JSON to decide columns, fields, and actions.
* Implement integration with the Universal Navigators so that all entity operations are triggered via X/Y/Z scripts from the SuperTable context.
* Add basic filters, sort and navigation that rely on Navigate_Y and Navigate_Z.

### 2.6 PHO Reference Domain Implementation

To validate the architecture, we will fully implement at least one complete domain:

* Use **PHO** as the primary reference domain, following the PHO Bridge and PHO Process documents from Phase 1.
* Model PHO templates in CMP, PHO instances in ETY, and PHO process steps in LOG, using the same MET / ATR / OPE and Navigator system used for everything else.

Implementation tasks:

* Define PHO templates with their MET and ATR composition.
* Implement PHO process flows using Navigator sequences.
* Use PHO as a showcase for demonstration, debugging and refinement of the architecture.

### 2.7 Documentation and Knowledge Transfer

For every component above we will:

* Update the **Technical Specification Document** so it becomes the living reference for the implementation.
* Update implementation notes for CMP / ETY / LOG, Navigators, SuperTable and PHO domain.
* Prepare a short **Developer Onboarding Guide** that explains how to model a new domain using templates, attributes and navigators instead of traditional tables and scripts.

---

## 3. Estimated Timeline and Milestones

The following is a realistic outline:

### Month 1 – Core engine foundation

* Implement CMP / ETY / LOG tables and identity system (DNA_ID, STRUCTURE_ID, BREADCRUMB_ID).
* Implement MET and OPE tables and Bootstrap loader for 56 MET and 56 OPE.
* Implement basic consistency checks between CMP, ETY and LOG.

**Milestone 1:**
Core triadic engine and MET / OPE loaded and verified in a test file.

---

### Month 2 – Templates and instance generation

* Implement template management in CMP with MET + ATR composition.
* Implement instance generation routines that create ETY JSON from CMP templates.
* Implement basic LOG creation for state transitions.

**Milestone 2:**
Templates can be created and changed in CMP, and ETY instances can be generated and updated with LOG entries.

---

### Month 3 – Universal Navigators

* Implement Navigate_X, Navigate_Y and Navigate_Z as working FileMaker scripts.
* Implement Universal_Processor and define at least one process as a sequence of navigator calls.
* Connect navigators to CMP / ETY / LOG and MET / OPE symmetry rules.

**Milestone 3:**
Universal Navigators and Universal_Processor working in a test domain with basic operations.

---

### Month 4 – SuperTable UI

* Build the SuperTable layout that lists and edits entities of any type using CMP and ETY JSON.
* Integrate navigator triggers into the SuperTable (buttons, actions, filters).
* Add basic user experience refinements for performance and clarity.

**Milestone 4:**
Working SuperTable prototype capable of listing and managing different entity types through the navigator engine.

---

### Month 5 – PHO reference implementation

* Model PHO templates in CMP fully.
* Implement PHO processes using Universal_Processor and navigators.
* Connect PHO entities to SuperTable for real-world navigation and editing.

**Milestone 5:**
PHO domain fully operational in THE BRIDGE, used as primary demo and validation case.

---

### Month 6 – Stabilisation and documentation

* Refinement of engine behaviour, performance and error handling.
* Completion of updated Technical Specification and Developer Guide.
* Final tuning and preparation for potential Phase 3 evolution.

**Milestone 6:**
Phase 2 completion and full mode testing.

---

## 4. Resource Allocation

**Roles**

* **Luca** – Ontologist and Product Owner

  * Provide ontological guidance, validate that implementation respects 3P3 axioms, review templates and processes.

* **Caufero (Cyril and Osbert)** – FileMaker architecture and implementation

  * Design, implement and develop CMP / ETY / LOG, MET / ATR / OPE, Navigators, SuperTable and PHO domain.
  * Maintain and update the technical documentation as the system grows.

We expect:

* Regular alignment calls (for example weekly or biweekly) to validate decisions and adjust scope if needed.

---

## 5. Budget Proposal

We propose to **keep the same commercial terms** as Phase 1:

* **Rate:** €35 per hour
* **Monthly cap:** 100 hours per month (invoicing based on actual hours up to that cap)

With the current scope and timeline, Phase 2 is estimated around:

* **Duration:** 5 to 6 months

(Speed of development will be employed to achieve this duration)

---

## 6. Risk Assessment and Mitigation

### 6.1 Performance and scalability

**Risk:**
The universal architecture relies heavily on JSON and a small number of tables, which can create performance challenges at scale.

**Mitigation:**

* Introduce performance testing early with realistic data volumes.
* Optimise SuperTable queries and Navigator calls as part of Month 4 and Month 5 work.

### 6.2 UI and user adoption

**Risk:**
The SuperTable and Navigator-based interaction model is very different from traditional FileMaker systems, which may require more training for users and other developers.

**Mitigation:**

* Deliver PHO domain as a clear reference implementation.
* Prepare a simple user guide and training session focused on how to “think in 3P3” rather than table-based systems.

### 6.3 Scope drift

**Risk:**
As the universality of the system becomes visible, there may be pressure to add more domains or features inside Phase 2.

**Mitigation:**

* Keep Phase 2 scope focused on the engine, the SuperTable and one reference domain (PHO).
* Defer additional domains and advanced tools to Phase 3.
