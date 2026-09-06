# Canonical Event Model

**Project:** TraceGap
**Version:** 0.1
**Status:** Draft

---

## 1. Purpose

This document defines the canonical representation of an event in TraceGap.

The purpose of the canonical event model is to provide a consistent representation for observations originating from different malware-analysis sources.

TraceGap must be able to consume heterogeneous observations while preserving their original meaning, identity, context, and provenance.

---

## 2. Definition

An **Event** represents an observation recorded during malware analysis.

An event describes something that was observed.

It does not automatically represent a behavioral interpretation, conclusion, or intent.

The fundamental distinction is:

```text
Event = Observation

Behavior = Interpretation derived from observations and relationships
```

Therefore:

```text
Event ≠ Behavior
```

---

## 3. Example

Consider the following observation:

```text
malware.exe created powershell.exe
```

The corresponding event may be represented as:

```text
event_id: E001

actor:
    malware.exe

action:
    process_created

target:
    powershell.exe
```

This event represents the observation that one process created another process.

It does not automatically establish:

```text
PowerShell execution
```

or:

```text
Malicious PowerShell execution
```

Those are analytical interpretations that require additional evidence and reasoning.

---

# 4. Canonical Structure

The initial TraceGap Event model is:

```text
Event
│
├── Identity
│   └── event_id
│
├── Observation
│   ├── timestamp
│   ├── source
│   ├── actor
│   ├── action
│   └── target
│
├── Context
│   └── contextual metadata
│
└── Provenance
    └── evidence reference
```

---

# 5. Identity

## 5.1 event_id

Every event must have a unique identity within the relevant TraceGap dataset.

```text
event_id
```

identifies the individual occurrence represented by the event.

Event identity must not be derived solely from observable content.

For example, these two events may contain identical information:

```text
E001
timestamp: 10:00:00
actor: powershell.exe
action: file_open
target: C:\Temp\a.txt
```

and:

```text
E002
timestamp: 10:00:00
actor: powershell.exe
action: file_open
target: C:\Temp\a.txt
```

They must still be capable of representing two distinct observations.

---

## 5.2 Identity Is Not Ordering

An event identifier must not be confused with event ordering.

For example:

```text
event_id: E104
sequence: 12
```

The sequence number describes position.

The event identifier describes identity.

Therefore:

```text
Identity ≠ Ordering
```

A sequence number must not become the fundamental identity of an event.

---

# 6. Observation

The Observation section describes what was observed.

It contains the initial canonical fields:

```text
timestamp
source
actor
action
target
```

---

## 6.1 Timestamp

The timestamp represents when the observation occurred according to the source that produced it.

Example:

```text
timestamp:
    2026-09-06T18:32:10Z
```

The timestamp describes the observation.

It does not necessarily prove causality.

For example:

```text
E001 occurred before E002
```

does not automatically mean:

```text
E001 caused E002
```

Temporal ordering and causality are separate concepts.

---

## 6.2 Source

The source identifies where the observation originated.

Examples:

```text
sandbox
EDR
network_monitor
memory_analysis
static_analysis
debugger
analyst
```

Example:

```text
source:
    sandbox
```

The source is important because different sources may provide different levels and types of evidence.

---

## 6.3 Actor

The actor identifies the entity associated with performing the observed action.

Example:

```text
actor:
    malware.exe
```

Possible actors may include:

```text
process
thread
user
host
module
network_endpoint
```

The exact representation of actors will be refined as the canonical model evolves.

---

## 6.4 Action

The action identifies what was observed.

Examples:

```text
process_created
file_created
file_opened
module_loaded
registry_modified
network_connection
memory_written
process_terminated
```

The action should describe the observation without embedding an analytical conclusion.

Prefer:

```text
memory_written
```

over:

```text
process_injection
```

when the underlying evidence only establishes a memory-write operation.

---

## 6.5 Target

The target identifies the object affected by the observed action.

Example:

```text
actor:
    malware.exe

action:
    process_created

target:
    powershell.exe
```

The target may represent:

```text
process
file
registry_key
memory_region
network_endpoint
module
host
```

depending on the event.

---

# 7. Context

Context contains additional information necessary to understand the observation.

Examples may include:

```text
process_id
parent_process_id
thread_id
file_hash
command_line
file_path
network_protocol
port
hostname
memory_address
user
integrity_level
```

Context must not be used to silently transform an observation into an interpretation.

For example:

```text
action:
    memory_written

context:
    target_process: explorer.exe
```

does not automatically mean:

```text
behavior:
    process_injection
```

Additional reasoning may be required.

---

# 8. Provenance

Provenance describes where the event came from and how it can be traced back to its original evidence.

Example:

```text
provenance:
    source: sandbox
    evidence_reference: sandbox_event_18472
```

Provenance is fundamental to TraceGap because every analytical result should eventually be traceable back to the observations and evidence that support it.

Conceptually:

```text
Gap
 │
 ▼
Relationship
 │
 ▼
Evidence
 │
 ▼
Event
 │
 ▼
Original Analysis Source
```

---

# 9. Event and Evidence

An Event represents an observation.

Evidence represents the information supporting that observation or a relationship involving that observation.

Therefore:

```text
Event ≠ Evidence
```

A simplified relationship is:

```text
Event
  │
  └── supported by ──► Evidence
```

The detailed Evidence model will be defined separately.

---

# 10. Event and Behavior

TraceGap must maintain a strict distinction between observations and interpretations.

Example:

```text
Event:

actor:
    malware.exe

action:
    memory_written

target:
    explorer.exe
```

Possible interpretation:

```text
Behavior:

Process Injection
```

The behavior cannot be considered proven solely because the event exists.

The behavioral conclusion may require:

* additional events;
* relationships;
* contextual information;
* evidence from another source;
* analyst reasoning.

Therefore:

```text
Observation
    ↓
Evidence
    ↓
Relationships
    ↓
Behavioral Interpretation
```

---

# 11. Example Canonical Event

A complete conceptual event:

```text
Event

Identity:
    event_id: E001

Observation:
    timestamp: 2026-09-06T18:32:10Z
    source: sandbox
    actor: malware.exe
    action: process_created
    target: powershell.exe

Context:
    parent_process_id: 4210
    command_line: powershell.exe -NoProfile

Provenance:
    evidence_reference: sandbox_event_18472
```

This represents an observation.

It does not automatically establish malicious intent.

---

# 12. Multiple Events

TraceGap must support multiple events that contain identical or nearly identical observable information.

Example:

```text
E001:
    powershell.exe
    file_open
    C:\Temp\a.txt

E002:
    powershell.exe
    file_open
    C:\Temp\a.txt
```

These events may represent separate occurrences.

The system must not automatically collapse them into a single event.

If deduplication is performed, it must be an explicit analytical operation.

---

# 13. Event Ordering

Events may be ordered by temporal information or an explicit sequence.

Example:

```text
E001 → E002 → E003
```

However, ordering alone does not establish causality.

For example:

```text
E001:
    process_created

E002:
    network_connection
```

The fact that:

```text
E001 < E002
```

does not automatically establish:

```text
E001 caused E002
```

Causal reasoning belongs to a higher analytical layer.

---

# 14. Uncertainty

The Event model must preserve uncertainty.

If a source cannot determine a field, the system should not invent a value.

For example:

```text
target:
    unknown
```

is preferable to creating an unsupported target.

Similarly:

```text
unknown
```

must remain distinct from:

```text
none
```

and:

```text
false
```

This prevents missing information from being incorrectly interpreted.

---

# 15. Design Rules

The canonical Event model follows these initial rules:

### Rule 1

An Event represents an observation.

### Rule 2

An Event must have an independent identity.

### Rule 3

Event identity must not depend solely on observable content.

### Rule 4

Event identity must not be confused with ordering.

### Rule 5

An Event must not automatically become a behavioral conclusion.

### Rule 6

Missing information must not be silently invented.

### Rule 7

Events should preserve provenance whenever possible.

### Rule 8

Duplicate observations must not be silently destroyed.

### Rule 9

Temporal ordering must not automatically imply causality.

### Rule 10

Analytical interpretations belong to higher-level TraceGap models.

---

# 16. Conceptual Data Flow

The Event model fits into the larger TraceGap architecture:

```text
Analysis Source
      │
      ▼
Raw Observation
      │
      ▼
Canonical Event
      │
      ▼
Evidence
      │
      ▼
Relationship
      │
      ▼
Behavioral Analysis
      │
      ▼
Gap Detection
```

The Event model therefore forms one of the foundational layers of TraceGap.

---

# 17. Future Extensions

The initial model is intentionally small.

Future versions may introduce:

* richer actor types;
* richer target types;
* process and thread identifiers;
* host identity;
* network metadata;
* event confidence;
* source reliability;
* event categories;
* timestamps with uncertainty;
* cross-source correlation;
* event transformations;
* normalized identifiers;
* relationships between events;
* evidence quality metadata.

These extensions must preserve the core principles defined in this document.

---

# 18. Summary

The TraceGap canonical Event model establishes a simple but important rule:

```text
An Event tells us what was observed.
```

It does not automatically tell us:

```text
what it means
```

or:

```text
why it happened
```

or:

```text
whether it was malicious
```

Those questions belong to higher analytical layers.

The foundational model is therefore:

```text
Event
 │
 ├── Identity
 │
 ├── Observation
 │
 ├── Context
 │
 └── Provenance
```

This model will serve as the foundation for the subsequent:

```text
Evidence Model
        ↓
Relationship Model
        ↓
Gap Model
        ↓
TraceGap Analytical Engine
```
