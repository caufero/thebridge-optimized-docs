# **Chapter 13 — Building the Schema**

## **13.1 Overview**

The schema of THE BRIDGE is minimal, strict, and ontologically grounded.
It is not a domain-driven schema, nor does it resemble traditional software database design.

There are only three core tables:

* **CMP_TABLE** — structural manifestation (ASPETTO)
* **ETY_TABLE** — current state manifestation (ENTITÀ)
* **LOG_TABLE** — historical manifestation (NATURA)

All other tables are optional helpers and must never replace, duplicate, or interfere with the triad.
The schema is intentionally simple so that all complexity is expressed through ontology, not through database structure.

The goal when building the schema is to create the *least amount of physical structure* capable of expressing the *full ontology*.

---

## **13.2 Principles for Schema Construction**

1. **Every entity is represented in three manifestations (CMP–ETY–LOG).**
2. **All attributes must be stored inside JSON containers.**
3. **No domain-specific tables are allowed.**
4. **No fields may be added that bypass attribute definitions (MET/ATR).**
5. **All mutations must be handled by the Process Manager.**
6. **All entity creation must be handled by the Instance Manager.**
7. **Schema must be stable and never evolve with domain needs.**
8. **The only evolving part is the ontology (templates, attributes, operations).**

This creates a universal, domain-free architecture.

---

# **13.3 CMP_TABLE (Structural Manifestation)**

### **13.3.1 Purpose**

CMP_TABLE defines the structural identity of every entity:

* what it *is*
* which MET attributes it supports
* which ATR attributes it supports
* how it behaves
* which operations are valid
* how it appears in the Entity Graph

CMP is the ASPETTO manifestation and must never store dynamic or instance-specific data.

---

### **13.3.2 Recommended Fields**

| Field               | Purpose                                            |
| ------------------- | -------------------------------------------------- |
| **dna_id**          | Identifies the entity across CMP–ETY–LOG           |
| **structure_id**    | Coordinate in structural space                     |
| **breadcrumb_id**   | Ontological trajectory (rarely used for templates) |
| **entity_type**     | MET, OPE, TPL, or domain-level entity              |
| **template_for**    | Indicates which entity the template structures     |
| **json_schema**     | Full definition of entity attributes (MET + ATR)   |
| **json_validation** | Validation rules for attributes                    |
| **default_values**  | Default attribute values for ETY initialization    |
| **json_ui**         | UI configuration descriptors                       |
| **metadata**        | Optional structural metadata                       |

Only structural information is stored in CMP.

---

# **13.4 ETY_TABLE (Current State Manifestation)**

### **13.4.1 Purpose**

ETY_TABLE stores the live state of every entity in the system.

An ETY row is the only place where the “current reality” of an entity exists.

It contains:

* universal attributes (from MET)
* specific attributes (from ATR)
* dynamic changes applied by operations
* links to other entities
* lifecycle information
* timestamps
* state fields

ETY is the ENTITÀ manifestation.

---

### **13.4.2 Recommended Fields**

| Field             | Purpose                                          |
| ----------------- | ------------------------------------------------ |
| **dna_id**        | Identity of the entity                           |
| **structure_id**  | Determines which attributes and operations apply |
| **breadcrumb_id** | Parent/child lineage                             |
| **entity_type**   | Domain-level entity, template, MET, or OPE       |
| **template_id**   | CMP entry that defines structure                 |
| **json_data**     | All attribute values (universal + specific)      |
| **created_at**    | Instance creation timestamp                      |
| **updated_at**    | Last update timestamp                            |
| **status**        | Lifecycle state (optional helper)                |
| **metadata**      | Optional runtime metadata                        |

All attributes live inside `json_data`.
No additional domain fields may be added.

---

# **13.5 LOG_TABLE (Historical Manifestation)**

### **13.5.1 Purpose**

LOG_TABLE stores the immutable history of every change made to any entity via the Process Manager.

A LOG entry:

* records the operation executed
* captures the before and after states
* ensures a complete NATURA record
* enables full reconstruction of any entity’s timeline
* allows navigation of historical depth

LOG is the NATURA manifestation.

---

### **13.5.2 Recommended Fields**

| Field             | Purpose                                |
| ----------------- | -------------------------------------- |
| **log_id**        | Unique identifier for the log entry    |
| **dna_id**        | Which entity this log entry belongs to |
| **structure_id**  | Structural coordinate                  |
| **breadcrumb_id** | Z-axis lineage at time of mutation     |
| **operation_id**  | The OPE executed                       |
| **before_state**  | JSON snapshot before mutation          |
| **after_state**   | JSON snapshot after mutation           |
| **timestamp**     | Moment of execution                    |
| **actor**         | User or system process responsible     |
| **notes**         | Optional natural language description  |

LOG must remain immutable.

---

## **13.6 Index Requirements**

To maintain performance and deterministic queries, the following fields require indexes:

### **CMP_TABLE**

* dna_id
* structure_id

### **ETY_TABLE**

* dna_id
* structure_id
* breadcrumb_id
* entity_type

### **LOG_TABLE**

* dna_id
* timestamp
* operation_id

JSON fields should remain unindexed to avoid overhead.

---

## **13.7 JSON Schema Structure**

Each CMP entry contains a `json_schema` describing:

* universal attributes
* specific attributes
* value types
* validation rules
* allowed operations
* visibility rules
* required fields
* constraints derived from the matrix

This schema determines:

* how ETY rows are initialized
* which operations apply
* which validations run
* what appears in UI components
* how the entity behaves under navigation

---

## **13.8 Domain Extensions (Templates)**

Domain-level structures—PHO, CLI, ORD, EXT—are added by creating **new template entries** in CMP:

* TPL_PHO_001
* TPL_ORD_001
* TPL_CLI_001
* TPL_EXT_001

Each template’s `json_schema` defines:

* which MET apply
* which ATR apply
* structural constraints
* valid lifecycle states
* default values
* UI descriptors

Through templates, entire business domains are added without altering the schema.

---

## **13.9 Schema Immutability**

The schema is designed to remain unchanged forever.
All evolution occurs through template and attribute definitions stored inside CMP and ETY.

Rules:

1. Do not add fields to CMP, ETY, or LOG.
2. Do not add domain-specific tables.
3. Do not modify structure_id values.
4. Do not remove or alter the core triad.
5. Do not bypass ontology-driven storage.

This immutability is what gives THE BRIDGE its universality.

---

## **13.10 Summary**

Building the schema for THE BRIDGE involves:

* creating the three core tables
* configuring essential indexes
* enforcing JSON-based attribute storage
* preparing CMP for structural definitions
* enabling ETY for runtime entity states
* enabling LOG for immutable history

After schema creation, the system is ready for script configuration and navigation engine setup.
