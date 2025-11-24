# **Chapter 7 — The 3 Core Tables**

The architecture of THE BRIDGE is built entirely on **three and only three tables**:

* **CMP** → ASPETTO (Structure)
* **ETY** → ENTITÀ (Being / Current State)
* **LOG** → NATURA (Process / History)

> “Architecture is based on three core tables and a small set of generic engines.”
> *(TheBridge Documentation)*

> “CMP (structure), ETY (entities), LOG (history).”
> *(Implementation Guide)*

These three tables form the full ontological representation of any entity—whether business-level (PHO, CLI, ORD), template-level (TPL_*), or meta-level (MET, OPE).

Everything else in THE BRIDGE (SuperTable, navigators, JSON, React prototype) exists to **serve these three tables**.

---

## **7.1 Why There Are Only Three Tables**

Traditional systems split the world into separate tables:

* customer table
* order table
* call table
* product table
* manufacturing table
* HR table
* scheduling table

This creates fragmentation.
3P3 eliminates this by recognizing:

> **Everything is Entity.
> Whatever exists must appear as ASPETTO, ENTITÀ, and NATURA.**

Therefore:

| Table   | Perspective | Meaning                                        |
| ------- | ----------- | ---------------------------------------------- |
| **CMP** | ASPETTO     | The entity’s structural definition.            |
| **ETY** | ENTITÀ      | The entity’s current state in the world.       |
| **LOG** | NATURA      | The entity’s history – every change or action. |

Every entity—PHO calls, CLI customers, ORD orders, EXT manufacturing batches, MET universal attributes, OPE operations, even template definitions—is represented in *all three* tables.

This is the triadic ontology of 3P3.

---

## **7.2 The Three Sacred Codes**

Every row in CMP, ETY, and LOG is connected by three universal identifiers:

### **1. DNA_ID — Identity Across All Perspectives**

The DNA_ID uniquely identifies the entity across its triad.
Examples:

* `PHO00001`
* `TPL_PHO_001`
* `MET006`
* `OPE009`

The moment an entity exists, its **DNA_ID** becomes the key that binds:

```
CMP (structure)
ETY (current entity)
LOG (history)
```

### **2. STRUCTURE_ID — Coordinate in Structural Space**

Defines the entity’s position in the structural grid:

* which MET apply
* which ATR exist
* which operations (OPE) are valid

This is the entity’s **X-axis coordinate** in structure space.

All three tables share the same STRUCTURE_ID for the same entity.

### **3. BREADCRUMB_ID — Ontological Trajectory**

Tracks:

* parent–child lineage
* navigation path
* depth (Z-axis)
* how the entity was reached through the Universal Navigators

Every LOG entry deepens this trail.

---

## **7.3 Table One: CMP (ASPETTO — Structural Manifestation)**

### **What CMP Actually Contains**

CMP stores the **structural manifestation of every entity**, including:

* Templates (TPL)
* Business entity structures (PHO, CLI, ORD, EXT…)
* Bootstrap manifestations for MET
* Bootstrap manifestations for OPE

> “BOTH (template and instance) are ENTITIES… BOTH live in CMP–ETY–LOG… BOTH follow the same rules.”
> *(Implementation Guide)*

**CMP = ASPETTO**
It is the “what it *is*” perspective of any entity.

### **Fields**

* dna_id
* entity_type (`TPL`, `PHO`, `CLI`, `MET`, `OPE`, etc.)
* template_for (for templates only)
* structure_id
* json_schema (which MET + ATR define this entity)
* validation rules
* default values
* ui_configuration

### **Examples Stored in CMP**

| Entity                   | CMP record                         |
| ------------------------ | ---------------------------------- |
| PHO phone call           | `TPL_PHO_001` + PHO structure rows |
| Customer (CLI)           | structural definition for CLI      |
| MET006 (created_at)      | CMP manifestation for MET006       |
| OPE009 (duration update) | CMP manifestation for OPE009       |

CMP is the **structural spine** of the ontology.

---

## **7.4 Table Two: ETY (ENTITÀ — Current State Manifestation)**

ETY represents the **living entity** at this moment.

Every entity has an ETY row:

* PHO25001 → a real phone call that happened
* CLI03002 → an actual customer in the system
* ORD01923 → a real order
* MET006 → ETY manifestation of the universal attribute
* OPE009 → ETY manifestation of a universal operation
* TPL_PHO_001 → ETY manifestation of a template (yes, templates have state!)

The existence of ETY for templates and meta-entities is explicitly confirmed by bootstrap:

> “Each MET and OPE exists as CMP, ETY, LOG.”

### **ETY Fields (Conceptual)**

* dna_id
* entity_type
* template_id
* structure_id
* breadcrumb_id
* universal attributes (from MET)
* specific attributes (from ATR)
* json_data (entire current state)

ETY is the **current living snapshot** in ENTITÀ perspective.

---

## **7.5 Table Three: LOG (NATURA — Process/History Manifestation)**

LOG stores:

* every change
* every action
* every OPE execution
* every update to MET or ATR
* every workflow step
* every state transition

This is the NATURA manifestation of the entity.

The log contains:

* dna_id
* structure_id
* breadcrumb_id
* operation_id
* before_state
* after_state
* timestamp
* actor

### **Universal Rule**

> “Every mutation of an entity creates a LOG entry.”
> *(Implementation Guide)*

This includes mutations to:

* templates
* MET
* OPE
* PHO, CLI, ORD, EXT, etc.

Everything has NATURA.

---

## **7.6 The Triad Applied to Real Entities**

### **PHO — Phone Call**

| Perspective | Table | Example                                          |
| ----------- | ----- | ------------------------------------------------ |
| ASPETTO     | CMP   | `TPL_PHO_001` structure defines what a call *is* |
| ENTITÀ      | ETY   | Actual calls: `PHO00001`, `PHO00002`             |
| NATURA      | LOG   | Call events: ring → answer → duration → notes    |

### **CLI — Customer**

CMP describes “what a customer is.”
ETY holds each customer’s current data.
LOG holds every change (creation, status update, assigned operator, etc.)

### **MET — Universal Attributes**

CMP describes MET006 (created_at) structurally.
ETY describes its current system-level role.
LOG logs bootstrap creation and later modifications.

### **OPE — Universal Operations**

CMP describes OPE009 (SET_DURATION).
ETY tracks its current registration.
LOG logs when operations are executed.

This is the absolute consistency of the ontology.

---

## **7.7 Summary: The Triad Is the Architecture**

The entire design of THE BRIDGE rests on:

### **Three Manifestations of Every Entity**

1. **CMP** — ASPETTO
2. **ETY** — ENTITÀ
3. **LOG** — NATURA

### **Three Sacred Codes**

* DNA_ID
* STRUCTURE_ID
* BREADCRUMB_ID

### **One Unified Entity Model**

All entities—templates, operations, MET attributes, and business objects—are identical at the ontological level.

This is why the architecture is universal, scalable, deterministic, and self-consistent.

Say **“Continue.”**
