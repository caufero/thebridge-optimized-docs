## 3.1 What is an Ontological Structure?

An ontological structure defines how concepts relate to one another inside the system. It is the framework that establishes meaning, identity, and classification before those concepts manifest as data or processes.

In The Bridge, the ontology is not a reference document. It is the living foundation of the system itself. The ontology defines the entities that exist, what roles they play, how they relate to each other, and how they participate in time.

### 3.1.1 The ontology precedes implementation

Before a system can operate, it must understand what exists in its domain. The ontology provides that understanding, not by defining storage mechanisms, but by defining the entities conceptually.

The ontology answers questions such as:

- What kinds of entities exist in this universe?
- How are these entities classified?
- What properties define them?
- How do they behave and interact?

Only after these questions are answered can the system manifest data or execute processes.

### 3.1.2 The ontology defines meaning, not structure in code

Traditional systems store meaning externally—in documentation, developer knowledge, or static schema. The Bridge stores meaning internally as entities, so the system understands itself using the same ontology that describes the business domain.

Meaning is not upstream from implementation; meaning *is* the implementation.

### 3.1.3 Why ontology matters in The Bridge

The ontology ensures:

- definitions are consistent across time
- structure and behavior are aligned
- new concepts can be added without redesign
- the system evolves through meaning, not refactoring

The system becomes coherent because everything that exists is grounded in the same conceptual structure.

### 3.1.4 A structure of entities, not tables

The ontology describes relationships between entities—not database tables, classes, or modules. Entities exist conceptually in CMP, are manifested in ETY, and evolve through LOG.

The ontology defines:

- **what entities exist**
- **how they relate**
- **how meaning flows between them**

The tables simply express that ontology.

### 3.1.5 Summary

> The ontological structure is the conceptual blueprint that defines reality inside the system.  
> It is not documentation; it is the foundation of execution.

---

## 3.2 The Ontological Hierarchy

The ontology in The Bridge is not a flat list of concepts. It is structured hierarchically, where different entities occupy different levels of abstraction. These levels describe how concepts move from pure definition to real-world manifestation.

The hierarchy does not represent privilege or precedence. Instead, it represents how meaning flows from higher-level definitions into concrete reality.

### 3.2.1 Higher levels define lower levels
The top of the hierarchy contains definitions that have no dependency on runtime entities. These definitions establish the conceptual landscape of the system:

- fundamental entity types
- universal classifications
- domain primitives

Lower levels represent instantiated or derived entities that depend on these definitions.

This creates a cascade:

> **Meaning originates at higher levels and becomes concrete at lower levels.**

### 3.2.2 Templates sit above instances
Templates are conceptual. They describe nature, identity, and possible attributes. Instances depend on templates for meaning.

| Level | Description | Example |
|-------|-------------|---------|
| Template | Conceptual definition | "Patient" as a kind of entity |
| Instance | Concrete manifestation | "Patient #18204: Ama Owusu" |

Instances cannot exist without templates, but templates can exist without instances.

### 3.2.3 State emerges from processes
At the lowest level of manifestation, the entity’s lifecycle is expressed through events in time. These events do not redefine the entity; they express its evolution.

Thus:

- **Definition → Manifestation → Timeline**

### 3.2.4 The hierarchy is not structural in the database
The hierarchy is ontological, not physical:

- Everything is stored in the same substrate
- No separate tables exist for different layers of abstraction
- Higher- and lower-level entities share the same representational mechanics

The hierarchy is conceptual, not architectural.

### 3.2.5 Why hierarchy matters
The hierarchy keeps meaning coherent when the system grows. It makes it possible to:

- add new entity types without rewriting logic
- extend domains through definitions
- reason about behavior at different levels of abstraction
- allow specialization without losing generality

### 3.2.6 Summary

> Higher levels define meaning.  
> Lower levels express that meaning in reality.  
> The system remains unified because all levels exist in the same ontology.

---

## 3.3 Entity Types and Roles

In The Bridge, different kinds of entities exist, but they do not require different tables or separate schema. All entities share the same representational substrate and differ only in their role within the ontology.

An “entity type” defines the role an entity plays in the system. It does not define where the entity is stored or how it is represented technically—those are constant across all types.

Different types exist to express different conceptual functions, not different storage mechanisms.

### 3.3.1 Types describe meaning, not structure

Traditional systems categorize entities based on how they are stored:

- core tables
- lookup tables
- transaction tables
- metadata tables
- configuration tables

In The Bridge, this categorization does not exist. All entities are stored the same way and participate in the same logic. The system distinguishes entities based on what they mean, not how they are stored.

### 3.3.2 Examples of entity types by role

Some entities define structure:

- templates
- conceptual types
- classifications

Others express concrete reality:

- people
- machines
- artifacts
- records

Others express behavior over time:

- events
- transitions
- actions
- observations

These types describe **function**, not implementation.

### 3.3.3 Types are emergent from CMP

Entity types live in **CMP** as definitions. They are entities that define how other entities behave. But unlike traditional schema, these definitions are not external:

- a template is an entity
- a classification is an entity
- an operation is an entity
- even an attribute definition is an entity

The ontology is composed of entities defining entities.

### 3.3.4 Instances derive from types

While types define meaning, instances express that meaning concretely. Instances exist in **ETY**, where structure becomes real.

Examples:

| CMP (Type) | ETY (Instance) |
|------------|---------------|
| *Medication* | Amoxicillin 500mg Capsule |
| *Person* | Patient #21804: Ama Owusu |
| *Transaction* | Invoice #4221 paid at 13:45 |

Types are upstream; instances flow from them.

### 3.3.5 Types do not create schema

Even though entity types describe structure, they do not create new tables. They are not equivalent to schema classes or database models.

There is **no one-to-one correspondence** between type definitions and storage architecture.

There is only the ontology.

### 3.3.6 Why this matters

This approach allows the system to:

- define new concepts without migrations
- add complexity without altering architecture
- store meaning and behavior as entities
- expand domains simply by inserting CMP entries

The difference between types is conceptual, not structural.

### 3.3.7 Summary

> Entity types define meaning.  
> Instances express that meaning.  
> Both exist as entities.  
> The difference is role, not structure, and never tables.

---


The attribute is universal; its manifestation is contextual.

### 3.4.3 Values exist independently of schema

Because attributes are entities, recording an attribute does not require altering schema. The system stores attribute values in a uniform representational structure, not in dedicated fields.

This allows:

- new attributes added dynamically
- changing attribute definitions without migrations
- consistency across domains

Meaning evolves without altering architecture.

### 3.4.4 Attributes are part of identity

Attributes contribute to what an entity *is*, not just what data it stores. They help define identity by specifying qualities that persist across time.

For example:

- A product's "Weight" attribute contributes to classification in shipping.
- A patient's "Blood Type" attribute contributes to medical treatment.

Attributes are part of ontological truth.

### 3.4.5 Attributes influence time through LOG

When values change, those changes are recorded in LOG as transitions. The transition is not "field updated"; it is **a change in an ontological property of an entity.**

Example conceptual transition:

> Patient blood type updated due to corrected lab result.

The LOG entry expresses meaning, not database mutation.

### 3.4.6 Summary

> Attributes are entities that define meaning.  
> Values express that meaning in time.  
> Schema does not define attributes; attributes define schema.

---

## 3.5 Operations as Ontological Entities

Operations describe how an entity can change over time. They define permitted transformations, conditions, and consequences, but they are not implemented as code. Operations exist as entities in the ontology and act as the conceptual source of behavior.

Execution occurs when a transition is recorded in LOG that expresses the operation being carried out on a concrete instance.

Operations define possible change. LOG expresses actual change.

### 3.5.1 Operations define transitions, not functions

An operation is not a function call or script. It is a definition of how an entity's state may evolve.

Examples:

- "Approve Order"
- "Assign Bed"
- "Dispense Medication"
- "Record Inspection Result"

Each operation has meaning at the level of CMP, not at the level of runtime code.

### 3.5.2 Operations apply to templates, manifest in instances

Operations apply upstream to templates or entity types. Instances inherit operations from the templates that define them.

| Layer | Expression |
|-------|------------|
| CMP | Operation defined: *"A Patient can be admitted"* |
| ETY | Patient #18204 is admitted |
| LOG | Admission recorded at 14:22:03 with details |

The ontology defines what "admission" *means* before a patient ever exists.

### 3.5.3 Execution is recording transitions in LOG

Instead of running code that mutates records, the system performs an operation by writing a transition into LOG. That transition updates ETY, forming a causal chain.

Execution = meaning + time

The system knows why something happened because the ontology defines the operation that caused it.

### 3.5.4 Operations can evolve without code changes

Because operations exist as entities:

- new operations can be added dynamically
- workflows can change through definitions
- transitions can be analyzed historically
- behavior is controlled by meaning, not codebase

The system can gain new capabilities by adding definitions rather than writing code.

### 3.5.5 Operations influence identity through time

Operations build the narrative of an entity. They explain the chain of meaning that led to its current state.

For a patient:

- admitted
- triaged
- diagnosed
- treated
- discharged

Each step is a meaningful transition, not a mutation of fields.

### 3.5.6 Summary

> Operations are entities that define how meaning evolves into action.  
> The system runs operations by recording transitions, not by executing procedural code.

---

## 3.6 Classifications & Taxonomy

Entities in The Bridge are organized through classifications and taxonomies. These structures define how concepts relate to broader categories or derive from more fundamental types. Classification is a conceptual operation, not a schema relationship. It defines meaning, not storage.

A classification states what kind of thing an entity is. A taxonomy defines how classifications relate to one another in a structured hierarchy.

### 3.6.1 Classification defines identity

A classification connects an entity to a higher-level concept. It determines what the entity *is* in relation to the ontology.

Examples:

- A *Patient* is a classification of *Person*.
- A *Medication* is a classification of *Product*.
- A *Visa Application* is a classification of *Process Instance*.

Classification answers the question:  
**What role does this entity play in the domain?**

### 3.6.2 Taxonomy defines relationships between classifications

Classifications can form hierarchies. These relationships define conceptual lineage, not data structure.

Examples of taxonomic relationships:

- Person → Patient → Inpatient
- Artifact → Document → Medical Report
- Event → Medical Event → Lab Result Logged

These chains do not create tables or storage layers—they express meaning across levels of abstraction.

### 3.6.3 Classification applies through definitions, not schema

To classify an entity, the system does not:

- add a column like `Category`
- assign a class through code
- create new child tables

Instead, classification is stored as an ontological relationship between entities in CMP.

The system understands classification because classification exists as meaning.

### 3.6.4 Instances inherit classification from templates

Classification happens at the definition level. When an instance is created, it inherits classification from the template that defines it.

| Level | Example | Meaning |
|--------|---------|---------|
| CMP | Entity Type: Patient | Defines what a Patient *is* |
| ETY | Patient #18204 | Is a Patient because CMP defines that type |
| LOG | "Admitted at 14:22" | Expresses how that entity changed in time |

No separate structure defines classification—it is inherent in ontology.

### 3.6.5 Multiple classifications may apply

An entity may belong to multiple classifications if meaning requires it.

Example:

A medical facility may be:

- a *Company*
- a *Location*
- a *Care Provider*
- a *Business Entity*

All of these classifications exist as conceptual relationships, not structural constraints.

### 3.6.6 Taxonomy supports specialization and generalization

Higher-level concepts define shared characteristics. Lower-level concepts refine or extend them.

This enables:

- universal behaviors
- domain-specific extensions
- consistent reasoning across levels

### 3.6.7 Summary

> Classifications define what an entity is.  
> Taxonomy defines how concepts relate across abstraction.  
> Both exist as relationships inside the ontology, not schema constructs.

---

## 3.7 Ontological Composition (Entities Made of Entities)

Entities in The Bridge are not defined by internal schema structures. Instead, they are composed of relationships with other entities. Composition expresses how meaning is built from parts without introducing structural fragmentation.

Composition describes how an entity is formed conceptually, not which fields it contains.

### 3.7.1 Composition is conceptual, not structural

In traditional system design, composition often leads to:

- join tables
- nested models
- subrecords
- polymorphic associations

These approaches tie meaning to storage.

In The Bridge, composition exists only as ontology. An entity is “made of” other entities if meaning requires it, not because of schema.

Example:

A *Medical Report* may be composed of:

- observations
- measurements
- interpretations
- signatures

Each of these is an entity that contributes to meaning.

No extra schema exists for "report parts."

### 3.7.2 Composition flows through relationships

Composition is expressed via ontological links that define how entities relate. These links exist uniformly across CMP, ETY, and LOG.

Types of compositional relationships include:

- **part-of**
- **contains**
- **derived-from**
- **extends**
- **depends-on**
- **references**

These are not structural joins. They are relationships that carry meaning.

### 3.7.3 Composition exists at multiple abstraction levels

Composition can occur at different levels of ontology:

| Level | Example |
|-------|---------|
| Definition (CMP) | A "Patient Profile" is composed of "Demographics," "Vitals," "History" |
| Manifestation (ETY) | Patient #18204 has 5 vitals records and 12 history entries |
| Time (LOG) | Each change in the profile is logged as events |

Composition remains true across all layers without redefining structure.

### 3.7.4 Composition is dynamic

Entities can gain or lose components over time. A composed structure is not fixed at creation.

Example:

A shipment may gain items as it progresses through logistics events. Each addition is logged as a transition, not as a restructuring of data.

Composition evolves through LOG.

### 3.7.5 No “subtables” or physical nesting

Even though entities may contain other entities conceptually, this does not justify:

- parent-child tables
- nested schemas
- embedded documents
- subtype-specific storage

Composition does not change the substrate.

Everything remains an entity stored in the same representational layer.

### 3.7.6 Composition supports reusable meaning

Entities can be composed using existing components rather than duplicating structure.

For example:

- The concept *Address* can be composed into Person, Hospital, Warehouse, etc.
- The attribute *Location Coordinates* can apply across domains.

Meaning is assembled, not rebuilt.

### 3.7.7 Summary

> Composition defines how entities form meaningful wholes.  
> It is expressed through relationships, not schema.  
> Structure is built from meaning, not tables.


---


The entity does not need multiple representations. It is one entity with layered meaning.

### 3.8.4 Specialization may add capabilities

A more specialized entity may:

- gain new attributes
- support additional operations
- participate in new workflows
- impose constraints not found higher up

These extensions are defined as additional entities linked to the specialized type.

No structural fork occurs in storage.

### 3.8.5 Multiple specialization paths can coexist

An entity may be a specialization of multiple upstream concepts simultaneously.

Example:

A medical drone may be:

- a Vehicle
- a Medical Device
- a Delivery Asset
- a Controlled Substance Container

Meaning is multifaceted and layered. There is no rigid single-parent inheritance.

### 3.8.6 The lineage must remain consistent

Because specialization defines identity, inheritance chains must:

- preserve meaning across levels
- avoid contradictions at lower levels
- ensure transitions remain logical

If a specialized type conflicts with its parent, the ontology is invalid.

### 3.8.7 Summary

> Specialization refines meaning without altering representation.  
> Inheritance exists as conceptual lineage, not schema hierarchy.  
> Instances carry the full meaning of their type lineage.

---

## 3.9 Ontological Constraints & Validity Rules

Ontological constraints define what can truthfully exist in the domain and under what conditions. They prevent contradictions, enforce meaning, and ensure that a system's state remains logically consistent over time.

These constraints are not implemented as validation rules in code or database schema. Instead, they are stored as entities that define what transitions and manifestations are valid.

Constraints live in CMP. Violations surface in ETY and are recorded in LOG.

### 3.9.1 Constraints define what is allowed to exist

A constraint is a rule that describes logical boundaries of meaning.

Examples:

- A *Patient* must be a *Person*.
- A *Medication* must have a dosage unit.
- A *Payment* must refer to a *Transaction*.

These constraints describe semantic truth, not UI or database requirements.

### 3.9.2 Constraints apply to both definitions and instances

Constraints apply at multiple levels:

| Level | Meaning |
|-------|---------|
| Conceptual (CMP) | Defines truth for all instances |
| Manifestation (ETY) | Ensures entities exist in valid form |
| Time (LOG) | Ensures transitions preserve logical continuity |

If a constraint is violated, the ontology—not the database—detects the inconsistency.

### 3.9.3 Constraints govern transitions, not just state

A system can be invalid **not because of its current data**, but because of the path that led to that state.

Example:

> A patient cannot be discharged if they were never admitted.

This is enforced at the level of LOG, not ETY.

Meaning persists across time.

### 3.9.4 Constraints evolve through definitions

Since constraints are entities, the rules governing the system can change:

- new constraints can be added
- old constraints can be deprecated
- constraints can specialize across taxonomies
- domain logic evolves without schema changes

This enables systematic evolution of behavior.

### 3.9.5 No constraints live in code

Constraints are not:

- `if` statements
- SQL validation rules
- data-type checks
- form validations

Even if such mechanisms exist at the UI level, they *mirror* ontology—they do not define it.

Meaning must exist upstream.

### 3.9.6 Detecting and resolving violations

When constraints are violated:

- the violation is recorded in LOG
- the invalid state is surfaced through ontology
- resolution occurs through further transitions

Example:

> System corrects medication dose through a new transition, not by overwriting values.

Time is preserved.

### 3.9.7 Summary

> Constraints define the boundaries of truth.  
> Validity is enforced by meaning, not schema.  
> Violations are resolved through transitions, not data rewrites.

---

## 3.10 Hierarchical vs Networked Ontologies

The ontology in The Bridge supports multiple ways of structuring meaning. While classification often appears hierarchical, entities can also participate in multiple overlapping structures that form networks of meaning rather than strict trees.

Hierarchy helps us reason about abstraction. Networks help us reason about relationships. Both coexist in the same ontology.

### 3.10.1 Hierarchies define abstraction

Hierarchies allow the system to model general–to–specific meaning. They show how concepts refine their parent concepts.

Examples:

- Person → Patient → Inpatient
- Product → Medication → Antibiotic

These express semantic lineage:

> “This entity is a more specific form of that entity.”

Hierarchy answers:  
**Where does this concept sit in the chain of meaning?**

### 3.10.2 Networks define relationships

While hierarchy organizes meaning vertically, networks organize meaning laterally. They describe how entities relate to each other through roles, dependencies, and participation.

Examples:

- A Person is linked to a Hospital as *a worker*, *a patient*, or *a visitor*
- A Vehicle participates in Processes such as *delivery*, *inspection*, *maintenance*
- A Document may belong to multiple domains simultaneously

Networks answer:  
**How does this concept interact with others?**

### 3.10.3 The ontology allows both simultaneously

A Patient may be:

- a Person (hierarchy)
- assigned to a Ward (network)
- admitted through a Process (network)
- categorized as Inpatient (hierarchy)

Meaning is multi-dimensional.

No single structure must dominate.

### 3.10.4 No structural divergence in storage

Even though meaning is multi-layered, all entities:

- share the same storage model
- store relationships uniformly
- participate in the same logic

There is no structural difference between "parent," "child," or "linking" entities.

Hierarchy is conceptual.  
Networks are relational.  
Storage is uniform.

### 3.10.5 Networks prevent rigid domain modeling

A system modeled purely as a tree becomes brittle:

- new relationships break hierarchy
- real-world entities fit poorly into rigid levels
- cross-domain meaning becomes impossible

The Bridge supports ambiguity and multi-parent identity, which matches reality more closely.

### 3.10.6 When to use hierarchy vs network

Use hierarchy when defining:

- lineage of identity
- specialization
- categorization
- abstraction

Use networks when defining:

- collaboration
- dependency
- participation
- composition

### 3.10.7 Summary

> The ontology is hierarchical where meaning demands structure, and networked where meaning demands relationships.  
> Both coexist in a single representational layer without altering schema.

---

## 3.11 Identity & Persistence Across Time

Identity in The Bridge is the continuity of meaning across structure, existence, and time. An entity remains itself even as its state changes. Identity does not come from schema or IDs—it comes from its definition in the ontology.

### 3.11.1 Identity originates in CMP

An entity is considered to exist because its identity is defined conceptually. Identity begins at the level of definition, not manifestation.

Even if no instance exists yet, the ontology defines the type of identity that can manifest.

Example:

- "Patient" exists conceptually before Patient #18204 is created.

### 3.11.2 Identity persists as ETY changes

When an entity exists in ETY, its values, attributes, and relationships may change. These changes do not replace identity; they express it in new states.

Identity persists because:

- state changes do not overwrite meaning
- new states are derived from previous states
- the core definition remains consistent

### 3.11.3 LOG extends identity across time

Time is not something applied after the fact; it is part of identity. Every change produces a transition that becomes part of the entity’s narrative.

A person is not a static record—they are a history of becoming.

Example conceptual timeline:

> Patient admitted → triaged → diagnosed → treated → discharged

Each step is not overwriting the previous; it adds layers to identity.

### 3.11.4 Identity does not depend on storage keys

Traditional systems treat identity as:

- a primary key
- an auto-incremented number
- a UUID
- a record location

These are implementation artifacts.

In The Bridge, identifiers are references to identity, not identity itself.

Meaning defines identity. Keys only help locate manifestations.

### 3.11.5 Identity persists through transformation

Identity remains continuous even when:

- attributes change
- classifications change
- roles change
- states change
- processes produce new forms

A Patient may later become an Employee.  
A Shipment may become an Asset on arrival.  
Identity persists; context evolves.

### 3.11.6 Violations of identity are ontological errors

An operation that contradicts identity is not just a failed transaction—it is a breach of meaning and must be resolved through transitions, not overwrites.

Examples of invalid transitions:

- Discharging a patient who was never admitted
- Treating a non-existent order as fulfilled
- Changing a person into a non-person entity

Such cases must be resolved in LOG, not patched in data.

### 3.11.7 Summary

> Identity is meaning carried across time.  
> CMP defines identity, ETY manifests it, LOG preserves its continuity.  
> The entity remains itself because its narrative is preserved, not overwritten.

---


An entity can belong to zero, one, or many domains without replication.

### 3.12.5 Cross-domain meaning is a core feature

Real-world entities rarely remain isolated inside a single conceptual system. The Bridge enables:

- healthcare + finance + logistics + compliance
- agriculture + sensor networks + weather + exports
- corporate structure + legal + payroll + auditing
- visas + travel + HR + immigration workflow

Because domains share entities, not copy them.

### 3.12.6 Multi-domain consistency is enforced by ontology

Domains may impose different constraints on the same entity. The ontology ensures these constraints remain consistent.

Example:

A company cannot be:

- "inactive" in compliance records
- "billing clients" in finance
- "processing shipments" in logistics

If domain meanings conflict, the ontology signals inconsistency.

### 3.12.7 Summary

> Domains are not separate systems.  
> They are semantic layers that interpret the same entities through different roles.  
> Meaning flows across domains because identity is unified.
