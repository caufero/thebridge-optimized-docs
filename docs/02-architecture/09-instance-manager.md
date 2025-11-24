# **Chapter 9 — Instance Manager**

## **9.1 Purpose of the Instance Manager**

The **Instance Manager** is the engine responsible for the creation, initialization, structural alignment, and state coherence of all entity instances in THE BRIDGE.

It ensures that every new entity — regardless of domain, purpose, or depth — is:

1. **Created correctly** according to its structural definition (CMP).
2. **Assigned a valid identity** through all three sacred codes.
3. **Initialized with the correct attributes**, both universal (MET) and specific (ATR).
4. **Registered in the triad** (CMP → ETY → LOG) in a consistent manner.
5. **Inserted into the Entity Graph** at the correct coordinate and depth level.

The Instance Manager removes the need for domain-specific creation scripts.
All entity births follow the same process, making instance creation universal, deterministic, and self-consistent across the system.

---

## **9.2 The Lifecyle of Entity Creation**

Every entity undergoes a standardized creation sequence:

```
(1) Select template (CMP)
(2) Generate sacred codes (DNA, STRUCTURE, BREADCRUMB)
(3) Initialize ETY record (current state)
(4) Apply defaults (CMP.default_values)
(5) Apply MET universal attributes
(6) Apply ATR specific attributes
(7) Insert LOG entry for creation
(8) Return ready-to-use entity to navigation layer
```

This lifecycle applies to:

* Business entities (PHO, ORD, CLI, EXT…)
* Template entities (TPL_PHO_001, TPL_ORD_001…)
* Meta-entities (MET006, OPE009…)
* Hierarchical entities created through Navigate_Z
* Any new entity type introduced in the future

The Instance Manager is domain-agnostic.

---

## **9.3 Template Selection (CMP → ETY)**

The first step is identifying which CMP definition will produce the new entity.

Templates contain:

* allowed MET
* allowed ATR
* default values
* structure_id
* UI configuration
* validation constraints

When creating PHO00001, for example, the Instance Manager selects:

```
CMP = "TPL_PHO_001"
```

From this CMP entry, it derives the structure of the new ETY instance.

The template is optional only for bootstrap entities (MET and OPE), as they originate from matrix definitions.

---

## **9.4 Assigning the Sacred Codes**

### **9.4.1 DNA_ID — The Ontological Identity**

The Instance Manager generates or receives a unique DNA_ID that identifies the new entity across:

* CMP (structure)
* ETY (current state)
* LOG (history)

Examples:

* `PHO00001`
* `CLI00042`
* `ORD00139`
* `EXT00501`
* `TPL_PHO_001`

DNA_ID ensures the triad is unified.

---

### **9.4.2 STRUCTURE_ID — Coordinate of Existence**

The STRUCTURE_ID is copied directly from the template:

```
ETY.structure_id = CMP.structure_id
LOG.structure_id = CMP.structure_id
```

This ensures:

* valid operations
* valid attribute set
* correct positioning in the structure grid
* predictable behavior under navigation

The STRUCTURE_ID never changes during entity lifespan.

---

### **9.4.3 BREADCRUMB_ID — Depth and Trajectory**

If the entity is created as part of a hierarchical process (e.g., via Navigate_Z), the Instance Manager assigns a breadcrumb:

```
parent_dna + “>” + new_dna
```

This forms the Z-axis lineage.

Example for a child task under an order:

```
ORD00031 > TSK00001
```

Breadcrumbs enable hierarchical views, depth navigation, and inherited filtering.

---

## **9.5 Initializing the ETY Record**

Once the sacred codes are in place, the Instance Manager constructs the ETY record.

### **9.5.1 Load Universal Attributes (MET)**

MET universal attributes are always applied:

* created_at
* deadline
* lifecycle_state
* name
* category
* cost
* assigned_to

Defaults may be provided through template definitions.

### **9.5.2 Load Specific Attributes (ATR)**

The template's JSON schema defines which ATR values apply.
The Instance Manager loads these attributes and sets:

* default values from CMP
* initial user-provided values
* empty placeholders if not provided

All attributes are stored in ETY.json_data.

---

## **9.6 Logging the Creation (LOG Entry)**

Every entity creation must be registered in LOG:

```
LOG:
- dna_id
- structure_id
- breadcrumb_id
- action_type = "CREATED"
- operation_id = "OPE_CREATE_ENTITY"
- before_state = {}
- after_state = full initial JSON
- timestamp
- created_by
```

This ensures NATURA receives the first entry in the process timeline.

Entity creation is not complete until the LOG record exists.

---

## **9.7 Instance Manager and the Entity Graph**

The Instance Manager inserts the new entity into the Entity Graph by:

1. Binding structural constraints from CMP.
2. Creating the entity’s present form in ETY.
3. Recording its first history entry in LOG.
4. Linking it to parent entities via breadcrumb.
5. Ensuring all future navigation flows can locate it deterministically.

Relationships between entities — such as “phone call → customer” or “order → production batch” — are not created by scripts but by structural and process rules inherited through the triad.

Once created, the new entity is:

* visible in the SuperTable
* addressable via Navigate_X
* creatable and filterable via Navigate_Y
* reachable through Navigate_Z
* orchestratable through Universal_Processor

This makes the entity fully alive within the ontology.

---

## **9.8 Special Case: Bootstrap Entities**

MET and OPE are produced through a dedicated bootstrap process, but they still pass through the Instance Manager logic in principle:

* CMP manifestation created → structural definition of the MET/OPE
* ETY manifestation created → their present state
* LOG manifestation created → their NATURA origin

Bootstrap is simply a batch instance creation with fixed structural inputs.

---

## **9.9 Guarantees of the Instance Manager**

The Instance Manager ensures that every new entity:

* has correct and complete structure
* is fully aligned with its template
* inherits correct MET and ATR
* is inserted into the Entity Graph
* has valid navigation coordinates
* is immediately compatible with all universal navigators
* is deterministically retrievable and reversible
* is fully logged
* is ontologically consistent across CMP–ETY–LOG

Through the Instance Manager, creation becomes a uniform, domain-independent operation.

All business behavior emerges naturally from the ontology, rather than custom coding.
