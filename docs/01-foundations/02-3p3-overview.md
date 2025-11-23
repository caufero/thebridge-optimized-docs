## 2.1 Introduction to 3P3

3P3 is the formal structure that expresses how entities exist, behave, and evolve within The Bridge. It provides a unified way to describe identity, state, and time using a single representational framework. The name refers to the three fundamental perspectives through which any entity can be understood.

These perspectives are:

- **Structure** — what the entity is and how it is defined.
- **Being** — the entity's concrete presence or manifestation.
- **Time** — how the entity changes, acts, or participates in processes.

3P3 provides the rules that govern how these dimensions interact.

### 2.1.1 Not separate layers
3P3 is not a three-layer architecture. It does not divide the system into modules representing definitions, data, and logs. Instead, it describes three ways to observe the same object.

The perspectives are complementary, not independent.

### 2.1.2 Alignment with the three-table substrate
Each dimension of 3P3 corresponds to one table in the implementation:

| Perspective | Table | Purpose |
|-------------|-------|----------|
| Structure | CMP | Defines what the entity is |
| Being | ETY | Represents real-world manifestations |
| Time | LOG | Records actions and state transitions |

These perspectives are not implemented separately; they emerge from the ontology.

### 2.1.3 Why 3P3 matters
3P3 gives the system a way to:
- represent definitions without schemas
- instantiate concepts without rigid models
- track history without external logging systems
- reason about behavior without procedural code

It unifies modeling, execution, and traceability under one mechanism.

### 2.1.4 3P3 and self-reference
Because templates and operations are entities, 3P3 applies to them as well. The rules that describe the system are subject to the system's own rules. This allows the ontology to evolve from within rather than being maintained externally.

3P3 is not only how the system represents the world; it is how the system represents itself.

---

## 2.2 The Three Perspectives in Detail

3P3 is built around three complementary perspectives that describe every entity in the system. These perspectives are not conceptual layers or system modules. They are different angles from which the same entity can be observed.

Each perspective highlights a distinct property of existence:

- **Structure** — what the entity is and how it is defined.
- **Being** — the fact that the entity exists in the system.
- **Time** — how the entity changes and interacts.

Together, they form a complete representation of reality inside the substrate.

### 2.2.1 Structure: What the Entity Is
Structure describes the inherent identity of an entity—its attributes, classification, constraints, and conceptual nature. Structure determines what is possible for the entity before it manifests.

In implementation, structure is expressed through **CMP**, where templates define:

- the nature of an entity
- what attributes it may have
- how it relates to other entities
- what operations it can participate in

Structure answers the question:  
**What is this?**

### 2.2.2 Being: The Present Manifestation of the Entity
Being represents the concrete presence of an entity in the system at a point in time. This is where the entity exists in a usable form—values are populated, relationships are active, and the entity is present in the domain.

In implementation, being is expressed through **ETY**, which represents the current manifestations of entities derived from CMP.

Being answers the question:  
**What exists right now?**

### 2.2.3 Time: How the Entity Evolves
Time is the dimension through which entities act, transform, and participate in events. It is not a log or audit trail; it is part of the entity’s identity. Time gives the system memory, causality, and sequence.

In implementation, time is expressed through **LOG**, which records transitions and processes.

Time answers the question:  
**How did this come to be, and what happened next?**

### 2.2.4 One Entity, Three Expressions
These perspectives are not three different data structures. They are three views of the same entity:

- **Structure defines what can exist**
- **Being is that structure manifested**
- **Time expresses how manifestation evolves**

The entity is unified; perspective is what changes.

This eliminates fragmentation between models, data, and history, allowing the system to reason about itself with complete internal coherence.

---

## 2.3 Mapping 3P3 to the Substrate (CMP, ETY, LOG)

The three perspectives of 3P3—Structure, Being, and Time—are expressed in implementation through a single substrate composed of three tables. These tables are not abstractions or conceptual placeholders; they are the operational mechanism through which the ontology becomes a running system.

| Perspective | Table | Purpose |
|-------------|-------|----------|
| Structure | **CMP** | Defines what the entity is, including templates, attributes, classifications, and identity rules |
| Being | **ETY** | Stores concrete manifestations of entities, representing current state |
| Time | **LOG** | Stores events, transitions, and the historical sequence of expression |

This mapping allows the ontology to exist fully inside the system rather than being enforced by external schema or code.

### 2.3.1 CMP — The Layer of Structure
CMP contains entities that define other entities. Templates, attribute definitions, operations, classifications, roles, and domain concepts all live here. CMP does not just describe business objects—it also stores definitions of system behavior.

CMP answers the question:
**What exists?**

### 2.3.2 ETY — The Layer of Manifestation
ETY contains real-world instances derived from CMP. These instances hold values, relationships, and active state. They represent data that currently exists and can be acted upon.

ETY answers the question:
**What exists right now?**

### 2.3.3 LOG — The Layer of Time and Process
LOG records transitions, events, and executions. These records express how entities evolve, interact, and move through processes. Time is part of the entity's identity, not an external audit.

LOG answers the question:
**What happened, and how did we get here?**

### 2.3.4 Why This Mapping Works
- Every concept is represented once.
- The system can describe its own structure.
- No external metadata is required.
- History is a first-class part of the entity.
- Growth does not require more tables.

These mappings make 3P3 not just a conceptual model, but a practical implementation strategy that can scale to entire real-world domains without architectural rewrites.

---

## 2.4 Why the Three Tables Cannot Be More or Less

The design of The Bridge is grounded in ontology, not software conventions. The system models reality through three fundamental dimensions: Structure, Being, and Time. These perspectives are not arbitrary—they represent the minimum required to describe an entity completely.

Because each perspective corresponds to one ontological dimension, the implementation requires exactly three tables:

- **CMP** (Structure)
- **ETY** (Being)
- **LOG** (Time)

These tables are the physical expression of the model. Adding more would introduce duplicate truths; reducing them would remove essential dimensions of existence.

### 2.4.1 Why not more tables?

If additional tables were introduced—such as separate tables for metadata, configurations, operations, workflows, or entity types—they would create new layers outside the ontology. This would:

- duplicate meaning across multiple representations
- break self-reference
- require synchronization
- prevent the system from describing itself

More tables would reintroduce the fragmentation The Bridge was designed to eliminate.

### 2.4.2 Why not fewer tables?

If CMP, ETY, and LOG were collapsed into fewer tables, the following distinctions would disappear:

- **Definition vs manifestation**  
  (A template and its instantiated entity would have no separation.)

- **State vs history**  
  (Past transitions would overwrite current state instead of extending a timeline.)

- **Concept vs existence**  
  (The system could not express an entity that is defined but not yet instantiated.)

Removing any dimension leads to a partial representation of reality.

### 2.4.3 The tables are perspectives, not categories

The three tables do not classify types of entities; they classify dimensions of existence. The same entity appears across all three dimensions:

- defined in CMP
- manifested in ETY
- expressed temporally in LOG

This maintains coherence across structure, state, and time.

### 2.4.4 The substrate is complete by definition

Because all concepts—including templates, attributes, processes, and system rules—are themselves entities, there is no class of object that requires separate storage or privileged schema.

Every part of the system must be expressible through CMP → ETY → LOG.

If something cannot be expressed in this substrate, it does not belong to the ontology.

### 2.4.5 The rule

> **There are exactly three tables because reality expressed through 3P3 has exactly three dimensions.**

The implementation does not shape the ontology; the ontology dictates the implementation.

---

## 2.5 The Lifecycle of an Entity Across CMP, ETY, and LOG

In The Bridge, an entity does not exist in a single location. It is expressed across three dimensions that represent different aspects of its identity. These dimensions do not replace each other; they coexist.

The lifecycle of an entity is the sequence of how these dimensions become visible as the entity comes into existence, manifests, and evolves over time.

### 2.5.1 Origin in CMP (Definition)

Every entity begins as a definition in **CMP**. At this stage, the entity exists as a concept: it has identity, attributes, classifications, and potential behaviors, but it is not yet instantiated.

Examples:
- A *Person* template
- A *Machine* concept
- A *Payment* type
- A *User Role* structure

These are entities, not meta-data. They exist inside the substrate.

**Key outcome:**  
The entity is defined, but no instance exists yet.

### 2.5.2 Manifestation in ETY (Existence)

When the entity becomes real in the system—for example, a specific patient, machine, or invoice—it manifests in **ETY**.

Here, the definition from CMP is applied to real data and relationships. Multiple instances can derive from one definition.

Examples:
- John Mensah as a Person
- Machine #A07 as a Machine
- Invoice #4821 as a Payment

**Key outcome:**  
The entity now exists and can be interacted with.

### 2.5.3 Expression in LOG (Time and Action)

As the entity participates in processes, events are written to **LOG**. These events do not overwrite ETY. They extend the timeline of the entity and describe how it changes.

Examples:
- A patient checked in at 08:41
- Machine A07 changed status from *Idle* to *Running*
- Invoice #4821 marked as Paid

Each LOG entry is part of the entity’s identity. It records how the instance evolves.

**Key outcome:**  
The entity now has history, causality, and motion.

### 2.5.4 The entity is all three dimensions at once

The lifecycle is not a pipeline. The entity does not "move" from CMP to ETY to LOG. Instead:

- **CMP defines what the entity can be**
- **ETY expresses what it is now**
- **LOG records how it became that**

The perspectives coexist and remain linked throughout the entity’s existence.

### 2.5.5 Deleting, retiring, or evolving an entity

Because time is a core dimension:

- Entities are never overwritten.
- State transitions accumulate as history.
- A retired entity is still part of the system’s ontology.

Evolving an entity means extending its timeline—not modifying its past state.

This creates continuity over time rather than state replacement.

---

## 2.6 How Identity Persists Across Dimensions

In traditional systems, an "entity" is often represented by different objects in different layers: a model class in code, a row in a database table, an API payload, and an audit record. These representations must be manually synchronized, and they frequently drift apart.

In The Bridge, identity is unified. One entity expresses itself across the three dimensions of CMP, ETY, and LOG without becoming separate objects. The identity of the entity is the same across all perspectives.

### 2.6.1 A single identity expressed three ways
Every entity has one ontological identity, which is expressed differently depending on perspective:

- **CMP** expresses what the entity is.
- **ETY** expresses that the entity exists now.
- **LOG** expresses how the entity changes over time.

These are not three entities. They are three dimensions of the same entity.

### 2.6.2 Linkage through shared identifiers
An entity's identity is carried through the substrate by a consistent reference, allowing:

- the CMP definition to anchor meaning,
- the ETY instance to anchor current state, and
- the LOG timeline to anchor historical context.

The reference does not represent a database constraint—it is an ontological anchor.

### 2.6.3 No duplication of representation
Unlike traditional systems where structure, instances, and logs are modeled separately, The Bridge ensures that:

- attribute definitions do not reappear in ETY,
- values do not overwrite historical state,
- historical events do not store redundant structure.

Each dimension holds only what belongs to that dimension.

### 2.6.4 Identity survives change
When values change, ETY does not lose meaning because LOG preserves the path that led there. When the structure evolves, CMP retains consistency because new definitions are themselves entities.

Identity is continuous even when:

- attributes are added later in time,
- definitions evolve,
- instances are retired or replaced,
- processes change.

The past remains valid, and the present remains coherent.

### 2.6.5 A unified object across time
Thinking in terms of objects:

- **CMP is the class**
- **ETY is the instantiated object**
- **LOG is the object's timeline**

But unlike object-oriented systems, these are not separate artifacts. They are three simultaneous expressions of one entity.

The entity does not change tables; the perspective does.

---

## 2.7 Representing Templates and Instances Using the Same Ontology

In traditional systems, templates and instances belong to different layers of representation. Templates live in schema definitions, configuration files, class models, or admin panels, while instances live in runtime tables. This separation forces developers to maintain two parallel sources of truth.

In The Bridge, templates and instances share the same representational space. Both are entities. The difference is not structural or stored in different tables—it is conceptual and expressed through role and purpose.

### 2.7.1 Templates are entities
A template defines the conceptual nature of an entity. It describes:

- what attributes can exist
- how the entity can behave
- which processes apply
- how it relates to other entities

Templates exist in **CMP** as first-class entities. They are not metadata. They are part of the ontology and subject to the same rules as the entities they define.

### 2.7.2 Instances are entities
An instance is a real manifestation of a template inside the domain. Instances exist in **ETY**, which represents real data and state.

Multiple instances can derive from one template, and each instance inherits structural meaning from its template.

### 2.7.3 Both are expressed with the same mechanics
Because templates and instances are both entities:

- they share the same identity model
- they have corresponding history in LOG
- they evolve through the same mechanisms

There is no separate code path for “system objects” versus “business objects.”

### 2.7.4 Templates define, instances express
A simple way to understand the relationship is:

- **CMP says what an entity is**
- **ETY shows that the entity exists now**
- **LOG explains how it changed**

Templates define structure, but the definition is not separate from the ontology that stores data.

### 2.7.5 No meta-layer is required
Since templates are entities, the system does not need:

- a schema table
- a configuration layer
- a rule engine
- a workflow engine
- a metadata repository

What would traditionally be “meta” exists as part of the same system.

### 2.7.6 Benefit: the system can describe and modify itself
Because templates live inside the ontology, the system can inspect, adjust, version, and extend itself using its own mechanisms. The platform does not need external migrations or code changes to evolve.

A new domain feature is added by defining a new CMP entity—not by rebuilding the system.

Templates and instances represent two roles, not two systems.

---

## 2.8 The Role of LOG in Process Execution

In most systems, logs are secondary. They exist to track changes, provide audit trails, or support debugging, but they do not affect system behavior. The Bridge treats time as a core dimension of reality. LOG is not a record of what happened after the system runs; it is part of how the system runs.

Processes are executed through LOG. Each event written to LOG expresses a transition that changes the state of an entity. LOG is both the timeline of the entity and the mechanism by which the entity evolves.

### 2.8.1 Events define change, not scripts
Instead of procedural scripts that modify records, The Bridge records transitions as LOG entries. Each entry describes:

- what entity changed
- what action occurred
- what state transition was produced
- why the transition is valid (which definition allowed it)

This means that behavior is driven by recorded events, not external logic.

### 2.8.2 LOG is authoritative, not observational
The timeline stored in LOG is not a derivative view; it is the primary source of truth about how the system reached its current state. Current state (ETY) can be reconstructed at any time by following the LOG timeline.

LOG expresses causality, not commentary.

### 2.8.3 Processes are sequences of transitions
A “process” in The Bridge is not a workflow or flowchart. It is a sequence of LOG events that represent how an entity interacts with other entities over time.

Examples:
- A patient moving from *Checked In* → *Vitals Taken* → *Consultation* → *Dispensed*
- A machine switching states in a production line
- A loan application moving through approval stages

The process is the timeline.

### 2.8.4 State is derived from history
ETY represents the current state, but it is a projection of the LOG timeline, not a standalone source. Changes in ETY are not arbitrary updates—they are the accumulation of recorded events.

If history changes, current state changes accordingly.

### 2.8.5 LOG makes reasoning possible
Because processes are represented as events, the system can answer questions like:

- What caused this state?
- What events led to failure?
- What path is most common?
- What happens after this step historically?

The system gains analytical capability as a natural result of how it operates.

### 2.8.6 LOG enables reversibility and simulation
Since every transition is explicit, the system can:

- replay history
- simulate future states
- roll back to previous states
- branch alternative timelines

These behaviors are impossible in systems where “state” is only the latest value in a table.

LOG is not storytelling. LOG *is* the execution layer.

---

## 2.9 Why Processes Are Not Workflows

In traditional systems, a process is usually defined as a sequence of predefined steps that an entity must move through. These are often represented as workflows, pipelines, or state machines. Each step is executed by procedural logic, and transitions are enforced by code or configuration files.

This approach assumes that the process exists outside the data it manipulates.

In The Bridge, a process is not a workflow. It is a timeline of transitions expressed through LOG events. The system does not execute a predefined path—it records the way entities act in reality, and the pattern of these actions appears through time.

### 2.9.1 Workflows enforce paths; LOG records reality
A workflow says:
> "The entity must follow this route."

LOG says:
> "This is the route the entity took."

Execution emerges from interaction, not from imposed flow constraints.

### 2.9.2 No separate workflow layer exists
There are no workflow tables, rule engines, or flow diagrams stored outside the ontology. What would traditionally be called a workflow is simply a recurring pattern of events inside LOG. The system does not hard-code a specific route; it observes and records transitions.

### 2.9.3 Flexibility replaces enforcement
Workflows break when exceptions arise:

- emergency steps
- manual overrides
- partial progress
- discontinued paths
- out-of-order events

The Bridge handles these naturally because events are logged directly, not constrained to a fixed sequence.

There is no "invalid path"—only paths that have different meanings.

### 2.9.4 The definition of permissible transitions comes from CMP
Instead of hard-coded logic, templates define what transitions are valid. LOG expresses which transitions actually occur.

- **CMP defines potential behavior**
- **LOG records actual behavior**
- **ETY reflects current state based on that history**

### 2.9.5 Processes emerge from data, not code
Over time, the system can recognize:

- common paths
- anomalies
- optimal routes
- observed patterns
- causes of failure

…but these insights are derived from the ontology rather than defined in advance.

### 2.9.6 Implication

> A process is not a flowchart.  
> A process is a timeline of transitions expressed through LOG.

This makes the system adaptive to reality rather than forcing reality to fit predefined flows.

---

## 2.10 The Purpose of 3P3 in System Design

3P3 is not just a theoretical model. It is the guide that ensures the system remains coherent as it grows. It exists to prevent fragmentation across structure, data, and history, and to ensure that every part of the system can be described and evolved using one consistent language.

3P3 gives us three things:

- a unified way to define entities,
- a unified way to represent those entities in reality,
- a unified way to track how those entities evolve over time.

### 2.10.1 The system stays consistent across growth
Instead of adding new tables, new models, and new schemas for each need, 3P3 ensures that all new concepts are expressed through the same substrate. This prevents the architecture from becoming a patchwork of exceptions.

Change happens through definition, not through redesign.

### 2.10.2 Execution is tied to meaning, not code
Processes occur because entities have meaning encoded in CMP. LOG records that meaning being expressed in time. Behavior doesn’t live in separate scripts or workflow engines.

Meaning drives action.

### 2.10.3 Models and runtime are the same language
Traditional systems use different languages for different layers:

- ERDs for structure
- code for logic
- tables for values
- logs for history
- documentation for meaning

3P3 merges these into a single ontology that expresses everything from the inside.

### 2.10.4 The system becomes self-extending
Because the ontology expresses itself, the system can:

- define new types at runtime
- adjust structure without migrations
- evolve processes internally
- generate new entities automatically

The architecture becomes stable even as the domain changes.

### 2.10.5 Summary

> **3P3 makes it possible to build systems that grow without losing coherence.**

It turns software into a continuously evolving model of reality rather than a fixed set of hard-coded behaviors.

---

## 2.11 Summary of the Three Perspectives

3P3 provides a unified way to understand entities across three ontological dimensions: structure, being, and time. These perspectives do not split the entity into separate components. They allow the same entity to be observed from different angles without duplicating representation.

### 2.11.1 Structure

Structure defines what an entity is. It contains identity, classification, attributes, and constraints. Structure exists in CMP, and represents the conceptual truth behind every entity in the domain.

It asks:  
**What is this?**

### 2.11.2 Being

Being expresses the entity’s concrete presence inside the system. It includes values, instance-level relationships, and active state. Being exists in ETY and reflects the current manifestation of the structure.

It asks:  
**What exists right now?**

### 2.11.3 Time

Time expresses how the entity changes, acts, and participates in processes. It records transitions as events that form a narrative of causality. Time exists in LOG and extends the identity of the entity across history.

It asks:  
**How did this become what it is?**

### 2.11.4 The entity is one object

These perspectives describe different dimensions of the same entity:

- CMP defines possible identity
- ETY expresses current identity
- LOG explains identity across time

None of these replace each other; they complete each other.

### 2.11.5 Why this matters

By representing entities in one unified ontology rather than separate system layers:

- meaning and behavior stay aligned
- change accumulates instead of overwriting state
- complexity grows without fragmenting the architecture
- history becomes part of the system, not a side table

This is why 3P3 is foundational to The Bridge—it keeps the entire system coherent as it expands.
