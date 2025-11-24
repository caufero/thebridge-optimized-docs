# **Chapter 8 — Relationships & The Entity Graph**

## **8.1 Overview**

The Bridge uses a single ontological principle to govern all relationships:

> **Entities relate to each other through their manifestations — CMP, ETY, LOG — using the three sacred codes: DNA_ID, STRUCTURE_ID, and BREADCRUMB_ID.**

Because everything is Entity, relationships are universal and do not depend on domain-specific schemas.
There is no difference between a relationship linking two “business objects,” two templates, or two meta-entities.
All forms of linkage follow the same structural rules.

The relationships form the **Entity Graph**, a deterministic, navigable structure connecting all entities across all perspectives.

---

## **8.2 The Entity Graph**

The Entity Graph is a multi-level network in which:

* **CMP nodes** represent structural relationships
* **ETY nodes** represent real-world connections
* **LOG nodes** represent process-driven links

These three layers together form the complete ontology of connectivity.

### **8.2.1 CMP-Level Links (Structural Relationships)**

Structural links in CMP express:

* parent → child structures
* allowed compositions
* inheritance of universal and specific attributes
* constraints defined by MET and ATR
* applicability of operations (OPE)

CMP relationships define what the world *can be*, not what it currently is.

Examples:

* A PHO template may structurally include a “related customer” attribute.
* An ORD template may require a link to a CLI customer.
* A MET or OPE entity may be structurally related through the 56×56 matrix.

CMP relationships are **purely definitional** and drive:

* attribute availability
* valid operations
* structural inheritance across depth levels
* UI configuration
* validation rules

All CMP links contribute to the **structural backbone** of the Entity Graph.

---

## **8.2.2 ETY-Level Links (Instance Relationships)**

At the ETY level, links represent actual relationships between instantiated entities.

Examples:

* A phone call ETY links to the actual customer ETY involved.
* An order ETY links to the production batch ETY fulfilling it.
* A production run ETY links to its parent manufacturing request ETY.

These are not foreign-key relationships in the traditional sense.
They are **entity-to-entity connections** preserved through:

* `parent_dna`
* `breadcrumb_id`
* structural binding inherited from CMP
* explicit associations written into ETY JSON data
* implicit associations triggered by OPE actions

ETY relationships express **what the world currently is**.

They support:

* navigation flows
* cascading attribute updates
* graph traversal
* instance-level filtering
* contextual UI behavior
* process sequencing

Because ETY is always aligned with CMP and LOG, instance relationships remain coherent regardless of depth, scale, or domain.

---

## **8.2.3 LOG-Level Links (Process Relationships)**

LOG links represent sequential, temporal, and causal relationships between actions.

Every LOG entry is connected to:

* the **entity** (via DNA_ID)
* the **structural coordinate** (via STRUCTURE_ID)
* the **trajectory** (via BREADCRUMB_ID)
* the **operation** executed (via OPE ID)
* the **before/after** entity states
* the **actor** responsible
* the **preceding and following** LOG entries

LOG-level relationships define **what happened**, and in what order, across the entire Entity Graph.

They support:

* auditing
* undo/rollback
* timeline generation
* analytical reporting
* compliance tracking
* historical reconstruction
* behavior pattern detection
* interpreting navigation depth through Z-axis traversal

LOG relationships are immutable and preserve the complete process history of the ontology.

---

## **8.3 Relationship Binding Through the Sacred Codes**

All relationships in the system, regardless of layer, are anchored by the three sacred identifiers:

### **8.3.1 DNA_ID — Identity Link**

Ensures absolute continuity between CMP, ETY, and LOG manifestations of the same entity.

A relationship referencing DNA_ID is a relationship to the **entity itself**, not merely a table row.

### **8.3.2 STRUCTURE_ID — Coordinate Link**

Links entities structurally and ensures that instance relationships remain consistent with their structure definitions.

A relationship referencing STRUCTURE_ID ensures:

* structural compatibility
* correct inheritance
* valid OPE execution
* deterministic filtering in the SuperTable

### **8.3.3 BREADCRUMB_ID — Trajectory Link**

Links entities and LOG entries along the ontological depth axis.

Breadcrumb relationships allow:

* parent/child navigation
* hierarchy discovery
* depth-based filtering
* Z-axis traversal through Navigate_Z
* reconstruction of how an entity was reached
* full history mapping of complex workflows

Together, these three identifiers make relationships **self-maintaining**, removing the need for traditional referential logic.

---

## **8.4 Universal Relationship Rules**

The Bridge enforces four universal rules for all relationships:

### **Rule 1: All Relationships Are Entity-to-Entity**

No relationship is table-specific.
CMP, ETY, and LOG merely provide different perspectives of the entity.

### **Rule 2: Relationships Must Not Break Structural Consistency**

Instance relationships must always reflect structural definitions bound in CMP.

### **Rule 3: Process Relationships Are Immutable**

All LOG relationships remain permanently preserved and must never be modified.

### **Rule 4: Traversal Must Follow Ontological Logic**

All navigation across the Entity Graph must occur through:

* Navigate_X (attribute navigation)
* Navigate_Y (instance creation/filtering)
* Navigate_Z (depth navigation)
* Universal_Processor (orchestration)

This ensures deterministic behavior.

---

## **8.5 Visual Outline of the Entity Graph**

```
               ┌───────────┐
               │   CMP     │   (Structure)
               └─────┬─────┘
                     │ structure_id
                     ▼
               ┌───────────┐
               │   ETY     │   (Current State)
               └─────┬─────┘
         dna_id       │       dna_id
                     ▼
               ┌───────────┐
               │   LOG     │   (History)
               └───────────┘

CMP ↔ ETY ↔ LOG together form a single entity.
Each can link outward to other entities on their layer,
creating the full Entity Graph.
```

Across this triad, additional relationships exist between entities, forming the complete graph:

```
PHO entity  →  CLI entity
ORD entity  →  EXT entity
TPL entities → domain-level CMP entities
MET entities → OPE entities (matrix-driven)
```

The Entity Graph is uniform across all domains because the ontology is uniform.

---

## **8.6 Summary**

The Bridge does not rely on traditional relational modeling.
All relationships emerge from the **triadic ontology** and the **three sacred codes**, forming a unified Entity Graph that:

* preserves structure
* maintains current state
* records all processes
* handles depth navigation
* supports universal traversal
* remains consistent regardless of domain
* scales to any number of entity types
