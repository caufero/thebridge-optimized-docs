# **CHAPTER 3 — ONTOLOGICAL STRUCTURE**

### *ASPETTO — NATURA — ENTITÀ in Depth*

## **3.1 Introduction to Ontological Structure**

The Bridge does not begin from tables, fields, or scripts.
It begins from **Being**.

Ontology governs everything.
Every entity in the system exists simultaneously in three intertwined realities:

* **ASPETTO** — Structure (what it *is*)
* **NATURA** — Process (what it *does*)
* **ENTITÀ** — Being (what it *becomes*)

These realities are not “parts” of an entity.
They are the entity.
They are the three angles by which the same being becomes intelligible.

```
ENTITY = ASPETTO  ∩  NATURA  ∩  ENTITÀ
```

## **3.2 ASPETTO — Structure (CMP Table)**

ASPETTO is the realm of form.
It is where the entity declares:

> “This is what I fundamentally am, regardless of time.”

ASPETTO determines:

* the **template** the entity belongs to
* the **structural definition** of that template
* which universal attributes (MET) apply
* which specific attributes (ATR) apply
* which operations (OPE) may transform it
* the shape of its JSON structure
* its position in the structural coordinate system (STRUCTURE_ID)
* defaults, invariants, and compositional truth

ASPETTO corresponds to **CMP** table, whose function is:

* Store every **template**
* Store every **structural instance** of each entity
* Store every **business instance** of each entity
* Store the **identity of the entity in structural space**

ASPETTO is always static, even when the entity is dynamic.
It is the *possibility* of the entity.
It defines the *rules of its existence*.

### **3.2.1 Structural Coordinates (STRUCTURE_ID)**

STRUCTURE_ID is the “address” of the entity within the ontological structure.
It tells:

* which parent template it derives from
* which composition rules bind it
* which structural siblings exist
* which MET and ATR apply
* what it is allowed to *become*

STRUCTURE_ID is the blueprint of the entity.

### **3.2.2 Templates and Structural Instances**

The Bridge removes any distinction between “template” and “schema of an instance.”

Templates are simply **structural CMP records**, and each real entity also has its **own CMP record**, which is the structural view of itself.

Thus:

* **Templates ≠ fixed types**
* **Templates = structural definitions that can themselves evolve**

Every entity has two “faces” in CMP:

1. *Template Definition Face* — structure it inherits
2. *Instance Structural Face* — structure it embodies

This duality is natural because ASPETTO always expresses **structure**, whether structural form is abstract (template) or concrete (instance).

## **3.3 ENTITÀ — Being (ETY Table)**

ENTITÀ is the realm of manifestation.
What the entity *is now*, in its current moment of existence.

This is **ETY** table.

ETY contains:

* JSON of all current attribute values
* current lifecycle & status
* current dynamic relations
* current operator or user responsible
* every mutable aspect of the entity

It is the **living dimension** of the system.

### **3.3.1 ENTITÀ as Integration**

ENTITÀ integrates:

* the structural truth of ASPETTO
* the historical truth of NATURA
* the existential truth of the entity right now

Anything that changes does so in ENTITÀ.

It is the seat of becoming.

### **3.3.2 The ETY JSON State**

The ETY_JSON expresses:

* states of universal MET attributes
* states of domain-specific ATR attributes
* states of any operationally introduced attributes
* derived calculations regarding an entity

ENTITÀ is described as “integration and becoming,” and the ETY table holds this integrated state.

The ETY JSON is always the translation of the structure (ASPETTO) and the operations (NATURA) into a single unified substance.

## **3.4 NATURA — Actions (LOG Table)**

NATURA is the dimension of **action**.

Where ASPETTO is form, and ENTITÀ is existence, NATURA is:

* motion
* causality
* change
* transformation

This is **LOD** table.

LOG records:

* every operation (OPE) applied
* before/after JSON states
* who performed the action
* timestamps
* structural coordinates
* the semantic meaning of each transition

NATURA is history, verbs, doing, and transformations.

### **3.4.1 LOG as Immutable Reality**

LOG is append-only.
Nothing is deleted.
Nothing is overwritten.

Every action in NATURA is sacred and permanent.

This gives The Bridge:

* full replay
* full reconstruction
* full traceability
* mathematical reversibility

### **3.4.2 NATURA and OPE**

Each action in LOG is an expression of an OPE (Operation).
Since OPE are linked one-to-one with MET, NATURA expresses **the active behavior** of the entity in perfect alignment with its structure.

## **3.5 The Three Sacred Identifiers**

Every entity is defined by three identifiers:

### **1. DNA_ID** — existential identity

Shared across CMP, ETY, LOG.

### **2. STRUCTURE_ID** — structural identity

Defines the structural space and template inheritance.

### **3. BREADCRUMB_ID** — hierarchical identity

Defines the entity’s nesting level and path within any hierarchical sequence.

Together they answer the absolute ontological questions:

* **WHO** are you? → DNA_ID
* **WHAT** are you structurally? → STRUCTURE_ID
* **WHERE** do you exist in relation to others? → BREADCRUMB_ID

## **3.6 The Unity of ASPETTO–NATURA–ENTITÀ**

The Bridge rejects the Western technical dualism between:

* schema and instance
* model and behavior
* metadata and data
* structure and execution

Instead, it declares:

> **One entity exists simultaneously in ASPETTO, NATURA, and ENTITÀ.**
> The difference is only perspective—not substance.

In formal notation:

```
ENTITY = (ASPETTO, NATURA, ENTITÀ)
```

And in FileMaker notation:

```
ENTITY = CMP + ETY + LOG
```

CMP is the *form*,
ETY is the *manifestation*,
LOG is the *action*.

This unity of structure, becoming, and process is the foundation of the entire architecture.

## **3.7 MET, ATR, and OPE Inside the Ontological Structure**

### **MET — Universal Attributes (56)**

Present across ASPETTO (CMP), ENTITÀ (ETY), and NATURA (LOG).
One MET = one universal property of all entities.

### **ATR — Reusable Specific Attributes**

Not tied to templates.
Compositional building blocks to enrich structural definitions.

### **OPE — Universal Operations (56)**

One OPE per MET.
Defines how NATURA can transform ENTITÀ based on ASPETTO.

Together, these produce:

* a universal grammar
* a universal state machine
* a universal semantic engine

## **3.8 Ontological Structure in FileMaker Terms**

The system’s depth is philosophical, but its implementation is brutally simple:

### **CMP** Table — structural reality**

### **ETY** Table — existential reality**

### **LOG** Table — process reality**

Nothing else is needed.

The entire The Bridge ecosystem unfolds from these three tables.
