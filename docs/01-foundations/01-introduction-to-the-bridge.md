# **CHAPTER 1 — FOUNDATIONS OF THE BRIDGE (3P3)**

## **1.1 The Ontological Premise**

Every system begins with a question:

**“What is an entity?”**

Traditional software answers by dividing reality into disconnected tables, fields, scripts, and modules.
The Bridge answers with ontology.

According to the document, 3P3 — *Tre Prospettive su Tre Realtà* — establishes that **an entity has three simultaneous manifestations**, each one a perspective on the same being:

* **ASPETTO** — what an entity **IS** (its form, structure, definition)
* **ENTITÀ** — what an entity **BECOMES** in existence (its current state, its being)
* **NATURA** — what an entity **DOES** (its processes, actions, and history)

These are not layers, not modules, not categories—they are **three essences of one unified reality**.

Thus The Bridge does *not* say:

> “An entity has these three properties.”

It says:

> **An entity *is* these three properties.**

## **1.2 The Three Tables: CMP — ETY — LOG**

In FileMaker, the three ontological essences are expressed through **three tables**, each one representing a perspective of the same entity:

| Ontology    | Table         | Meaning                                                            |
| ----------- | ------------- | ------------------------------------------------------------------ |
| **ASPETTO** | **CMP_TABLE** | The structure, template, and structural identity of the entity     |
| **ENTITÀ**  | **ETY_TABLE** | The being/instance of the entity (actual data, current state)      |
| **NATURA**  | **LOG_TABLE** | The history, process, and actions the entity performs or undergoes |

This is the first axiom of 3P3:

> **Every entity must exist in all three tables.**
> One record in CMP, one in ETY, and many in LOG.

These three records are not “connected”—they are **one being** observed from three viewpoints.

The unifying force is the **DNA_ID**, the sacred identifier shared across CMP, ETY, LOG.
It binds the three essences into one entity, one movement, one identity.

## **1.3 CMP (ASPETTO): Pure Structure and Instances**

CMP represents *form*, *possibility*, *definition*.
It holds:

* **Templates**
* **Structural instances**
* **Ontological coordinates (structure_id)**
* **Universal and specific attribute composition**
* **Validation rules**
* **UI configuration**

It is where the entity declares:

> “This is what I am allowed to be.”

The sample CMP template for PHO shows MET and ATR unified into one structure JSON.

CMP is also where **autopoiesis** begins:
The system can describe itself structurally using its own language.

## **1.4 ETY (ENTITÀ): Being and Current State**

ETY holds the *living state* of the entity:

* JSON of current attribute values
* lifecycle and temporal attributes
* relational state
* who it is now, at this moment

ETY is where the entity says:

> “This is what I have become right now.”

ETY is the seat of existence.
It maps cleanly to ENTITÀ—where the entity fully manifests as a being.

## **1.5 LOG (NATURA): Process and History**

LOG is the memory of the entity.

It records:

* every operation
* every change of state
* before/after JSON snapshots
* timestamps
* actors
* structural coordinates

LOG is where the entity declares:

> “This is what I have done.”

LOG is NATURA—pure process and chronology.
It is immutable and forms the basis of complete reconstruction from history.

## **1.6 The Sacred Identity: DNA_ID, STRUCTURE_ID, BREADCRUMB_ID**

The ontological identity of an entity is defined by three sacred coordinates:

1. **DNA_ID** — existential identity shared across CMP, ETY, LOG
2. **STRUCTURE_ID** — coordinates in structural space (what template and form the entity belongs to)
3. **BREADCRUMB_ID** — position in ontological depth (graph path, hierarchy)

Together they answer:

* **Who are you?**
* **What are you?**
* **Where are you located in the universal structure?**

## **1.7 Universal Attributes (MET) and Operations (OPE)**

The system defines **56 universal attributes** (MET_TABLE) and **56 universal operations** (OPE_TABLE).
Each OPE is linked 1:1 to a MET.

This yields:

* universal behavior
* universal validation
* universal update rules

The 56×56 matrix defines a total of **3,136 potential behaviors**, of which 971 are active.

These behaviors become the “grammar” of the universe.

## **1.8 ATR: Reusable Specific Attributes**

ATR are not domain-bound.
They are fully reusable and structurally independent.
PHO, CLI, ORD, etc. reuse the same ATR definitions.

This makes the ontology fractal and economical.

## **1.9 Navigators and the Universal Processor**

Navigation replaces traditional scripting:

* **Navigate_X** modifies attributes (SET, UPDATE)
* **Navigate_Y** creates or filters entities
* **Navigate_Z** traverses ontological depth

The Universal Processor orchestrates them into one complete process.

## **1.10 Bootstrap and Autopoiesis**

Bootstrap loads:

* 56 MET (×3 perspectives)
* 56 OPE (×3 perspectives)

Totaling **336 foundational records**.

This is how the system **assembles itself**—an expression of autopoiesis.

## **1.11 The Bridge as a Turing-Complete Ontology**

Because operations (OPE) and attributes (MET) form a closed computational grammar, and because processes recorded in LOG can express any transformation, the ontology is explicitly:

* self-describing
* self-maintaining
* fully generative
* **Turing-complete**

This is why the system can model **any business domain** without schema changes.
