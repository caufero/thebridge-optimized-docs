## 4.1 Purpose of the Rules

The rules of The Bridge define how meaning remains consistent across structure, existence, and time. They prevent contradictions, guarantee continuity, and ensure that the system evolves without fragmenting into separate interpretations of reality.

These rules are not implementation details or business logic. They are ontological laws that apply to all domains modeled within the system.

They define what it means for the system to remain true to itself.

---

## 4.2 Rules Govern Meaning, Not Code

Rules in The Bridge do not live in application logic, scripts, workflows, or database constraints. Instead, they exist as definitions inside the ontology itself.

A rule does not describe how code should behave.  
A rule describes what is true in the domain.

Execution conforms to meaning, not the other way around.

---

## 4.3 Rules Apply at All Three Levels

Rules operate across the three perspectives:

| Layer | How Rules Apply |
|-------|----------------|
| CMP | Defines what can exist |
| ETY | Ensures entities exist in valid form |
| LOG | Ensures transitions preserve continuity |

A rule is violated not only when state is invalid, but when the *sequence of events* leading to that state is invalid.

---

## 4.4 Rules Are Expressed as Entities

Rules themselves are stored as entities in the ontology, not embedded in code. They define constraints and logic as part of the same substrate that defines entities, attributes, and operations.

A rule is observed the same way as any other entity:

- defined in CMP
- manifested in ETY when enforced
- reflected in LOG when validated or violated


---

## 4.5 Rules for Existence and Validity of Entities

An entity exists in the system because its identity is defined, not because a record has been stored. Existence begins at the conceptual level and becomes tangible only when manifested through instances and transitions.

An entity is considered valid when its existence aligns with its definition and with the rules that govern its role in the system.

### 4.5.1 Existence originates at definition

An entity must first exist in CMP as a conceptual definition. Without a definition, no instance may be created.

A definition does not require any instances to exist. It represents potential existence.

### 4.5.2 Manifestation does not overwrite identity

When an entity exists in ETY, its current state does not replace its definition. State expresses existence; it does not define it.

If an instance exists without alignment to its definition, the ontology is inconsistent.

### 4.5.3 Existence depends on continuity in time

Even if an entity is structurally valid, a violation in the sequence of events can invalidate it.

An entity can be invalid because:

- its creation was not preceded by a required transition,
- its current state violates prior meaning,
- or its lineage contradicts its definition.

### 4.5.4 Validity is evaluated against meaning

Validity is not enforced by database constraints. It is enforced by ontological truth. The system determines whether an entity is valid based on its alignment with definitions, not based on whether a value exists in a field.

Invalidity is not a mutation error—it is a contradiction of meaning.

### 4.5.5 Violations are resolved through transitions

When an entity violates validity:

- the error becomes part of LOG,
- resolution is expressed as a new transition,
- and meaning is restored through time rather than rewriting data.

The system does not "fix" invalid entities by altering records. It resolves contradictions through further events.

---

## 4.6 Rules for Identity

Identity must remain continuous across all perspectives of the system. Identity cannot be duplicated, rewritten, or split. It can only evolve through transitions that preserve continuity.

### 4.6.1 One identity, many states

An entity may change classification, attributes, or relationships over time. These changes do not create new entities or replace existing ones. They produce a timeline that extends identity.

### 4.6.2 Identity cannot be reassigned

An entity cannot become a fundamentally different entity through mutation. Identity remains bound to its definition and lineage.

### 4.6.3 Identity cannot be destroyed without cause

If an entity ceases to exist, that cessation must be recorded as a meaningful transition, not as deletion. History remains intact.

---

## 4.7 Rules for Transitions

A transition expresses a change in state and must reflect a valid transformation defined by the ontology.

### 4.7.1 No transition without definition

A transition cannot occur unless defined in CMP as a valid operation or event.

### 4.7.2 Transitions are append-only

State changes do not overwrite previous state. They produce new entries in LOG that describe how meaning changed over time.

### 4.7.3 Transitions preserve causality

Each transition must reference the entity it affects and the operation that defines it. A change without causality is invalid.

### 4.7.4 Transitions extend meaning

A transition brings new meaning into existence. It does not mutate identity—it evolves it.

---

## 4.8 Rules for Consistency Across CMP, ETY, and LOG

The three layers of the ontology—definitions, manifestations, and transitions—must always remain aligned. Consistency means that an entity’s identity, current state, and history do not contradict one another.

Each layer expresses a different aspect of the same entity. If these aspects diverge, the ontology becomes invalid.

### 4.8.1 CMP defines what is possible

CMP is the source of truth for:

- what entities are,
- what attributes they may have,
- what transitions they may undergo,
- and what operations apply to them.

If CMP does not define a concept, it cannot exist elsewhere.

### 4.8.2 ETY expresses what currently exists

ETY reflects:

- the current state of each entity,
- the relationships it participates in,
- the values it currently manifests.

ETY must always align with the definitions in CMP.

If ETY expresses a state not permitted by CMP, the system is inconsistent.

### 4.8.3 LOG expresses how the current state came to be

LOG records:

- transitions,
- operations carried out,
- causes of change,
- the exact sequence of events.

LOG must be consistent with both CMP and ETY:

- If LOG records a transition that CMP does not allow, the system is invalid.
- If ETY shows a state that LOG cannot justify, the system is invalid.

### 4.8.4 None of the layers can contradict another

Contradictions arise when:

- ETY shows a relationship not defined by CMP,
- LOG records an operation that ETY does not reflect,
- ETY expresses a state without a supporting LOG history,
- or CMP defines rules that instances violate.

Consistency requires:

- CMP → defines possible truth  
- ETY → manifests current truth  
- LOG → preserves historical truth

These three must never diverge.

### 4.8.5 Consistency is enforced ontologically, not procedurally

No script or algorithm “checks” consistency.  
Consistency is inherent in the ontology.

If the ontology defines rules properly:

- invalid states cannot be represented meaningfully,
- invalid transitions cannot be expressed,
- contradictions naturally surface.

### 4.8.6 Consistency is preserved through time

Even if definitions evolve:

- past transitions remain valid,
- their meaning remains anchored in the definitions that existed at the time,
- ETY continues to derive meaning through LOG.

The past does not mutate to match new rules.

---

## 4.9 Rules for Interpretation and Meaning

Meaning in The Bridge is not derived from data values, UI forms, scripts, or code. Meaning is derived from the ontology itself. Every entity, instance, transition, and attribute must be interpreted according to the definitions that exist in CMP.

Interpretation is the process by which the system understands what something *is* and what it *means* based on the ontology.

### 4.9.1 Meaning is upstream of data

Data does not create meaning.  
Meaning gives data its interpretation.

If the ontology does not define what a value represents, the value is meaningless, even if stored.

### 4.9.2 Interpretation is context-dependent

Entities carry multiple layers of meaning depending on their role, classification, and transitions. Interpretation must respect:

- the entity’s type,
- its historical transitions,
- its current state,
- and its position in the ontology.

Two entities may store similar values but mean different things depending on their definitions.

### 4.9.3 Interpretation must follow lineage

Meaning inherits from higher-level concepts and cannot contradict the definitions it derives from.

Example pattern (conceptual only):

---


Interpretation respects inheritance.

### 4.9.4 Meaning emerges from transitions

An entity’s meaning is not static. As transitions accumulate, meaning expands:

- new operations give new roles,
- new transitions give new states,
- new events give new identity contributions.

The LOG is part of meaning.

### 4.9.5 Interpretation cannot override definitions

Interpretation cannot contradict CMP.  
If a recorded event suggests something impossible by definition, the system is inconsistent.

CMP is the authority over meaning.

### 4.9.6 Meaning must remain continuous

Meaning does not reset when state changes.  
Meaning evolves.

If an operation fundamentally changes identity, it must be expressed as:

- a new entity,
- or a new meaning chain that preserves history.

Interpretation cannot “break” an entity into two unrelated entities.

### 4.9.7 Summary

> Meaning is determined by the ontology.  
> Interpretation is the process of applying that meaning to state and time.  
> Truth must remain consistent across all layers.

---

## 4.10 Rules for Operations and Allowed Behavior

Operations describe how an entity may change, evolve, or participate in processes. These operations do not live in code or workflows; they live in the ontology as definitions. The rules governing operations ensure that all behavior in the system is valid, meaningful, and consistent with the identity of each entity.

Operations define *what may happen*.  
Transitions in LOG define *what did happen*.

### 4.10.1 Operations must be defined before they can be executed

No operation may be applied to an entity unless the operation is explicitly defined in CMP as part of that entity’s structure or lineage.

If CMP does not define an operation:  
- it cannot be executed,  
- no transition may be logged for it,  
- and no state may change because of it.

This preserves integrity.

### 4.10.2 Operations must align with entity type

An operation applies only to the entity types that define or inherit it.  
If an entity does not fall under a type that includes the operation, the operation is invalid.

Example pattern (derived from the uploaded document):

---


Operations do not “float.”  
They are bound to meaning.

### 4.10.3 Operations cannot contradict definitions

An operation cannot produce a state that violates:

- foundational definitions,
- inherited constraints,
- domain rules,
- or other ontological truths.

If an operation leads to contradiction, the ontology is invalid, not the data.

### 4.10.4 Operations produce transitions, not mutations

Executing an operation must generate a transition in LOG.  
The system does **not** mutate existing state; it extends the entity’s timeline.

Every operation must be:

- append-only,
- causal,
- meaningful,
- consistent with definitions.

### 4.10.5 Operations must be temporally valid

Operations must occur in an order that respects the ontology’s logic.

If operation B requires operation A to occur first, the system cannot express B unless LOG shows that A occurred earlier.

Temporal validity is part of ontological truth.

### 4.10.6 Operations may evolve, but their lineage must remain coherent

Operations may be:

- extended,
- specialized,
- refined,
- redefined.

But evolution must not contradict earlier meaning.

Prior transitions remain valid under the historical rules that existed when they occurred.

### 4.10.7 Summary

> Operations define how entities may evolve.  
> Transitions express how they actually evolve.  
> Allowed behavior is whatever remains consistent with the ontology.


---

## 4.11 Rules for Attributes and Their Values

Attributes in The Bridge are not table fields or schema structures. They are entities defined in the ontology. Their values exist as manifestations that must remain consistent with their definitions and with the transitions that produced them.

An attribute defines *what kind of information can exist*.  
A value expresses *that information in a specific instance*.

### 4.11.1 Attributes must exist as definitions before values can exist

A value cannot exist unless the attribute itself is defined in CMP.

If the ontology does not define an attribute, the system cannot meaningfully store or interpret any corresponding value.

The attribute is the source of truth.  
The value is only an expression of it.

### 4.11.2 Values must align with attribute type and meaning

An attribute definition specifies what kind of value is allowed:

- structure,
- format,
- constraints,
- semantic meaning.

A value that contradicts the definition is invalid.

Validity is semantic, not syntactic.

### 4.11.3 Attribute values exist in ETY, not CMP or LOG

Attributes live in CMP.  
Values live in ETY.  
Changes to values are expressed in LOG.

This three-layer flow prevents mutation and ensures the ontology remains consistent.

### 4.11.4 Values cannot be overwritten; they must be transitioned

A change to a value must be expressed as a transition in LOG.

Example conceptual pattern (not domain-specific):

---


Nothing is replaced.  
Everything evolves.

### 4.11.5 Attributes may be inherited or specialized

If an entity type inherits from another type, it inherits all the attributes defined upstream. These attributes remain valid unless explicitly overridden or extended by the specialized type.

Specialization cannot break meaning.

### 4.11.6 Attributes may apply across domains

Attributes do not belong to domains.  
Domains only interpret them.

If an attribute is defined once, it may appear in multiple contexts, depending on how the ontology links it.

This ensures consistency across multi-domain systems.

### 4.11.7 Attribute consistency must hold across time

If values change in time, the LOG must show:

- why the change occurred,
- which operation produced it,
- and how the meaning evolved.

A value with no causal justification in LOG is invalid.

### 4.11.8 Summary

> Attributes define what may be known.  
> Values express what is known.  
> Transitions express how knowledge changes.

---


Nothing is replaced.  
Everything evolves.

### 4.11.5 Attributes may be inherited or specialized

If an entity type inherits from another type, it inherits all the attributes defined upstream. These attributes remain valid unless explicitly overridden or extended by the specialized type.

Specialization cannot break meaning.

### 4.11.6 Attributes may apply across domains

Attributes do not belong to domains.  
Domains only interpret them.

If an attribute is defined once, it may appear in multiple contexts, depending on how the ontology links it.

This ensures consistency across multi-domain systems.

### 4.11.7 Attribute consistency must hold across time

If values change in time, the LOG must show:

- why the change occurred,
- which operation produced it,
- and how the meaning evolved.

A value with no causal justification in LOG is invalid.

### 4.11.8 Summary

> Attributes define what may be known.  
> Values express what is known.  
> Transitions express how knowledge changes.


Relationships are not toggled; they are produced through time.

### 4.12.4 Relationships must remain meaning-consistent

A relationship cannot contradict:

- the identities of the entities involved,
- the constraints defined by their types,
- or their historical transitions.

If a relationship violates meaning, the ontology becomes invalid.

### 4.12.5 Relationships are directional in meaning, not storage

Even if a relationship appears symmetric in data, it may not be symmetric in ontology.

For example (based on Luca’s patterns):

- An entity type may *require* another entity,
- while the reverse relationship is optional or derived.

The meaning defines the direction, not the storage implementation.

### 4.12.6 Relationships may span abstraction levels

An entity may relate to:

- a more abstract concept (template → entity),
- a concrete manifestation (entity ↔ entity),
- or a temporal event (entity ↔ transition).

All of these are valid forms of relationships.

### 4.12.7 Relationship constraints must hold across time

If a relationship becomes invalid due to a transition, it must be dissolved through another transition — not manually removed, mutated, or overwritten.

Meaning must be preserved.

### 4.12.8 Summary

> Relationships express how entities interact in meaning.  
> CMP defines them, ETY manifests them, LOG justifies them.  
> No relationship exists without conceptual and temporal truth.

## 4.13 Rules for Processes and Operational Flow

A process is not a workflow, script, or sequence of procedural steps.  
A process is a pattern of transitions that an entity undergoes through time.  
These transitions must derive from operations defined in CMP and be expressed through LOG.

Processes emerge from meaning, not from code.

### 4.13.1 Processes must be defined conceptually

A process cannot occur unless its structure and possible transitions are defined in CMP.  
Processes are entities that describe:

- what kind of events may occur,
- in what contexts they may occur,
- and what meaning each step represents.

CMP defines **the space of allowed behavior**.

### 4.13.2 Processes are executed through transitions

Execution does not perform steps.  
Execution records transitions.

A process is “running” only because its transitions are being logged over time.  
Nothing is mutated; everything is appended.

### 4.13.3 Processes may be incomplete, branched, or irregular

Since The Bridge does not enforce rigid step-by-step workflow paths, processes may:

- end early,
- skip steps,
- branch,
- loop,
- reverse,
- or remain partially executed.

All these forms are valid as long as they do not violate meaning.

### 4.13.4 Processes apply to entity types, not instances

Instances do not define processes.  
They participate in processes defined upstream at conceptual level.

### 4.13.5 The LOG defines what really happened

The process definition describes **potential behavior**.  
Only the LOG describes **actual behavior**.

The system does not assume ideal flows; it evaluates real flows.

### 4.13.6 Operational flow is determined by causality

The order of transitions in LOG determines:

- how the entity evolved,
- what states are valid,
- what operations are allowed next,
- what contradictions may arise.

Temporal sequence is part of the process.

### 4.13.7 Summary

> Processes define possible transitions.  
> LOG expresses real transitions.  
> The difference between the two defines behavior.

---

## 4.14 Rules for Time and Causality

Time in The Bridge is not a timestamp.  
Time is the ordered sequence of transitions recorded in LOG.  
Causality defines which transitions justify others and how meaning unfolds through this sequence.

Time is the structure of evolution.  
Causality is the reason behind evolution.

### 4.14.1 Time is defined by the order of transitions

The system does not rely on stored timestamps for truth.  
Truth comes from the sequence in which transitions appear in LOG.

If two transitions occur out of order, the meaning becomes inconsistent.

### 4.14.2 Every transition must have a cause

A transition is valid only if it is justified by:

- an operation defined in CMP,  
- and the state preceding it in LOG.

A transition without cause is invalid, even if the data looks correct.

### 4.14.3 Future states cannot contradict past meaning

New transitions may expand meaning but cannot invalidate:

- previous identities,
- previous roles,
- previous definitions,
- or previous states.

The past is immutable.

### 4.14.4 Time does not reset meaning

If an attribute changes, a relationship changes, or a process advances:

- the old state remains part of LOG,
- the new state extends meaning,
- identity remains continuous.

There is no “reset” in The Bridge.

### 4.14.5 Temporal gaps must be meaningful

If a long period passes with no transitions:

- the entity remains in its last valid state,
- absence of transitions is still meaningful,
- and the system must interpret continuity through time.

### 4.14.6 Simultaneous transitions cannot contradict each other

If two transitions occur at the same logical time, they must remain mutually consistent:

- one may extend meaning,
- both may update values,
- but neither may invalidate the other.

### 4.14.7 Time is linear within a single identity

Within one entity:

- transitions cannot fork,
- transitions cannot split identity into two branches,
- and transitions cannot merge unrelated timelines.

Identity has one lineage.

### 4.14.8 Summary

> Time is the order of transitions.  
> Causality is the justification for each transition.  
> Meaning unfolds through both.

---

## 4.15 Rules for Error, Contradiction, and Resolution

Errors in The Bridge are not technical faults.  
They are ontological contradictions — situations where meaning, sequence, or identity no longer aligns with the definitions established in CMP.

Contradictions are part of the system’s reality.  
They are never hidden, overwritten, or corrected by mutation.  
They must be addressed through transitions that restore meaning.

### 4.15.1 Errors arise when meaning and state diverge

An error occurs when:

- ETY expresses a state that CMP does not allow,
- LOG records a transition not defined in CMP,
- relationships contradict identity or lineage,
- or time suggests an impossible sequence.

Errors are deviations from ontological truth.

### 4.15.2 Errors cannot be deleted or corrected by mutation

The Bridge never “fixes data” by rewriting or deleting:

- values,
- transitions,
- identity,
- definitions.

Doing so would destroy causality and falsify the past.

Errors remain part of the timeline.

### 4.15.3 Resolution must occur through transitions, not modifications

A contradiction is resolved by applying a new transition that restores consistency.

Patterns include:

- declaring an invalid state as obsolete,
- redefining meaning through a corrective transition,
- creating a new valid relationship,
- or marking an entity as terminated.

Every resolution is a recorded event — never a rewrite.

### 4.15.4 Errors must remain visible and interpretable

Errors are part of an entity’s meaning.  
They must:

- remain observable,
- remain part of LOG,
- appear in the interpretation of the entity,
- influence how future states are evaluated.

Transparency is a requirement.

### 4.15.5 Contradictions cannot propagate into new meaning

A contradiction must be resolved before transitions that depend on the invalid state can proceed.

If not resolved:

- future transitions become invalid,
- relationships become ambiguous,
- and the meaning chain collapses.

Correct order of resolution preserves identity.

### 4.15.6 Resolution cannot contradict the past

A resolution must:

- maintain historical truth,
- preserve causality,
- acknowledge the invalid transition,
- avoid rewriting earlier meaning.

Past transitions remain untouched.

### 4.15.7 Errors do not invalidate identity

An entity’s identity is not destroyed by:

- a forbidden state,
- an incorrect transition,
- or an invalid relationship.

The identity continues, but must be restored to consistency through time.

### 4.15.8 Summary

> Errors express contradictions.  
> Resolution expresses recovery.  
> Both are part of the entity’s true history.

---

## 4.16 Constraints on Evolution of Definitions (How CMP May Change)

Definitions in CMP form the conceptual backbone of the ontology.  
They describe the identity, structure, attributes, operations, and allowed relationships of every entity.  
When CMP changes, the meaning of the entire system shifts.  
Because of this, CMP evolution must follow strict ontological constraints.

Definitions may evolve — but their evolution must preserve truth.

### 4.16.1 Definitions may evolve, but cannot contradict their origin

When a definition changes, it must remain compatible with the historical meaning of the entity it describes.  
A new definition may extend, clarify, or refine the concept, but cannot negate what the concept *was*.

If a change would invalidate earlier meaning, it is not a refinement — it is a new concept and must be expressed as such.

### 4.16.2 CMP evolution must not invalidate past transitions

Every transition recorded in LOG must remain meaningful under the definitions that existed **at the time** of the transition.

CMP cannot retroactively declare:

- that past operations were impossible,
- that past states were invalid,
- that past relationships were forbidden.

History remains fact.  
Definitions must respect it.

### 4.16.3 CMP evolution cannot fragment identity

A definition change must preserve the continuity of identity.

A single definition cannot be split into two unrelated definitions unless the ontology creates explicit new entities and transitions to separate them.

Identity must remain one continuous lineage.

### 4.16.4 Changes to definitions must be justified by ontology, not implementation

CMP changes cannot originate from:

- database structure limitations,
- workflow convenience,
- UI reorganizations,
- performance needs,
- or implementation shortcuts.

Definition changes must serve ontology — not code.

### 4.16.5 New definitions must integrate into existing meaning

When new attributes, operations, or relationships are introduced, they must:

- fit into the existing hierarchy,
- respect upstream meaning,
- preserve constraints,
- and align with the entity’s existing identity.

A new definition cannot exist in isolation.

### 4.16.6 Removal of definitions is not deletion; it is historical closure

If a definition is deprecated:

- it must not be deleted,
- it must not be erased,
- it must be marked as obsolete.

Deprecated definitions remain part of the ontology’s history and remain meaningful for all transitions that referenced them.

### 4.16.7 Evolution must remain semantically consistent

A definition may change only if:

- it does not contradict upstream meaning,
- it does not invalidate downstream meaning,
- it does not retroactively create contradictions.

CMP evolution must increase clarity, not confusion.

### 4.16.8 Summary

> Definitions may evolve, but not at the cost of identity, truth, or history.  
> CMP changes expand meaning — they do not rewrite it.

---

## 4.17 Invariants of the Ontology (Things That Must Always Hold True)

Invariants are truths that must always remain valid, regardless of how the system evolves, how definitions change, or how entities move through states and transitions.  
These invariants preserve the coherence, continuity, and stability of the entire ontology.

If an invariant is violated, the system ceases to represent meaningful reality.

### 4.17.1 Identity is continuous and non-fragmentable

An entity’s identity must always:

- remain singular,
- remain continuous through time,
- remain tied to its lineage,
- remain consistent with its definition.

Identity cannot break, fork, duplicate, or merge without explicit ontological transitions.

### 4.17.2 Definitions precede existence

Nothing can exist in ETY or LOG unless defined in CMP.  
Entities, operations, attributes, and relationships must originate conceptually before they appear in time.

The system cannot manifest what it does not define.

### 4.17.3 Transitions are append-only

LOG must never:

- rewrite,
- delete,
- reorder,
- or hide transitions.

History is immutable.  
Time moves forward only.  
Meaning evolves through addition, not substitution.

### 4.17.4 Meaning cannot contradict itself

No valid state or transition can introduce a contradiction against:

- definitions,
- roles,
- allowed relationships,
- allowed operations,
- temporal order.

Meaning must remain coherent.

### 4.17.5 State must always be justifiable through LOG

Every value, relationship, or condition in ETY must be explainable by a sequence of transitions in LOG.  
If a current state cannot be traced back to a valid set of transitions, the system is invalid.

### 4.17.6 Transitions must always be justified by definitions

No transition may:

- perform an undefined operation,
- introduce an undefined relationship,
- assign an undefined attribute,
- or produce a state forbidden by the ontology.

Definitions govern what can occur.

### 4.17.7 The past is true even if definitions evolve

Changes to CMP do not retroactively alter:

- the meaning of past transitions,
- the identity of past states,
- or the legality of past operations.

Earlier truth remains true.

### 4.17.8 Entities cannot exist without meaning

An entity must always retain:

- a definition (CMP),
- a state (ETY),
- a lineage (LOG).

If any one of these disappears or contradicts the others, the entity ceases to be meaningful.

### 4.17.9 All three layers (CMP, ETY, LOG) must remain aligned

The ontology requires global consistency:

- CMP defines possibility,
- ETY expresses current reality,
- LOG expresses historical reality.

These layers cannot diverge without producing ontological errors.

### 4.17.10 Summary

> Invariants are the laws of existence in The Bridge.  
> They ensure identity, consistency, and truth across time and meaning.
