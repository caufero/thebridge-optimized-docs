# 5. Terminology and Concepts

This chapter defines the formal vocabulary of The Bridge.  
Every definition, every operation, every state, and every transition depends on the language established here.  
The goal is to ensure that all parts of the ontology use precise, unambiguous terminology.

The terms below are aligned with the definitions, structure, and patterns provided in the source document.

---

## 5.1 Purpose of the Terminology

The Bridge uses a controlled conceptual vocabulary.  
Terms are not descriptive or informal; they carry strict ontological meaning.  
A term expresses:

- what something **is** in the conceptual layer (CMP),
- how it **manifests** in the existence layer (ETY),
- how it **evolves** across time (LOG).

This chapter ensures every later section uses terms consistently and precisely.

---

## 5.2 Core Conceptual Terms (High-Level)

### 5.2.1 Ontology
The complete structure that defines meaning, entities, relationships, operations, transitions, and the rules governing them.  
The ontology is the foundation of The Bridge.

### 5.2.2 Entity
A meaningful conceptual element defined in CMP.  
An entity can represent a type, attribute, operation, relationship, or any other conceptual structure.

### 5.2.3 Identity
The unique essence of an entity or instance that persists through time and transitions.  
Identity cannot be rewritten, duplicated, split, or merged without explicit ontological transitions.

### 5.2.4 Definition
The conceptual description of what an entity is.  
Definitions live in CMP and determine:

- allowed attributes  
- possible operations  
- allowed relationships  
- the role and meaning of an entity  

### 5.2.5 Manifestation
The existence of an entity in the ETY layer.  
Manifestation represents the current state of an entity, derived from its definition.

### 5.2.6 Transition
A recorded event in LOG that expresses a change of state.  
Transitions are append-only and form the timeline of meaning.

### 5.2.7 Process
A meaningful sequence of transitions.  
A process is a conceptual pattern of evolution, not a workflow.

---

## 5.3 Structural Terms (CMP Layer)

### 5.3.1 Concept
The highest-level abstraction that defines a category of meaning.  
Concepts may expand, specialize, or structure the ontology.

### 5.3.2 Attribute (Definition)
An element defined in CMP that expresses a property of an entity.  
Attributes define what values may be manifested in ETY.

### 5.3.3 Operation (Definition)
A conceptual action defined in CMP that expresses how an entity may change through transitions.

### 5.3.4 Relationship (Definition)
A conceptual link that specifies how two entities may interact or connect.  
Relationships originate exclusively in CMP.

### 5.3.5 Typology / Inheritance
A hierarchical structure where entities derive meaning from parent concepts.  
Inheritance ensures meaning remains coherent across levels.

---

## 5.4 Manifestation Terms (ETY Layer)

### 5.4.1 Instance
The manifestation of a definition in ETY.  
Instances express the current state of entities.

### 5.4.2 Attribute Value
The manifestation of an attribute in ETY.  
Values must always align with definitions.

### 5.4.3 State
The total set of attribute values, relationships, and manifestations for an instance at a given time.

### 5.4.4 Relationship Instance
A manifested connection between two entities, derived from a defined relationship.

---

## 5.5 Temporal Terms (LOG Layer)

### 5.5.1 Transition
A time-ordered event expressing a change.  
Transitions cannot be deleted, mutated, or reordered.

### 5.5.2 Event
A transition with additional contextual meaning.  
Events express why something changed.

### 5.5.3 Cause
The operation or rule that justifies a transition.

### 5.5.4 Sequence
The chronological ordering of transitions, forming the entity’s timeline.

### 5.5.5 Timeline
The complete historical chain of transitions associated with an entity.

---

## 5.6 Cross-Layer Terms

### 5.6.1 Consistency
The alignment of CMP, ETY, and LOG such that no contradictions exist.  
If one layer diverges, the ontology becomes invalid.

### 5.6.2 Meaning
The interpretation of an entity based on:

- its definition,
- its manifestation,
- its transitions.

Meaning emerges from ontology, not from stored values.

### 5.6.3 Validity
A state is valid only when consistent with the entity’s definition and its historical transitions.

### 5.6.4 Evolution
The expansion of meaning through transitions.  
Evolution is append-only and never overwrites previous truth.

### 5.6.5 Resolution
The restoration of consistency after a contradiction, accomplished through new transitions.

---

## 5.7 Patterns and Classifications

### 5.7.1 Roles
Semantic positions an entity can hold within a relationship or process.

### 5.7.2 Types
Entities classified under higher abstractions through inheritance.

### 5.7.3 Families
Groups of related concepts sharing structural patterns.

### 5.7.4 Categories
Named collections of entities organized by meaningful criteria.

### 5.7.5 Abstract vs. Concrete
- **Abstract**: conceptual definitions in CMP  
- **Concrete**: manifested instances in ETY  

---

## 5.8 Reserved Terms Used in the Document

This section lists terms that appear explicitly in the original source document and must retain their original meaning.

- **CMP** — conceptual layer of definitions  
- **ETY** — manifestation layer  
- **LOG** — temporal layer  
- **TYPE** — structural classification of entity  
- **ATTR** — attribute definition  
- **REL** — relationship definition  
- **OP** — operation definition  
- **INST** — an entity’s instance  
- **TRN** — a transition or event  
- **LIN** — lineage or chain of identity  
- **TMP / Template** — an entity’s conceptual blueprint  

(Only terms that appear in the original source are included.)

---

## 5.9 Summary

This chapter defines the vocabulary used throughout The Bridge.  
Each term expresses a precise and controlled meaning across the three layers of the ontology.

- CMP defines possibility.  
- ETY expresses current reality.  
- LOG expresses historical truth.  

With these terms established, all remaining chapters can now describe architecture, implementation, and behavior with full clarity and accuracy.
