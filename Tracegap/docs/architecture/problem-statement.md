# TraceGap — Problem Statement

**Version:** 0.1
**Status:** Draft

## 1. Problem

Malware analysis produces a large amount of heterogeneous information.

A single analysis may generate process events, file-system operations, registry modifications, network connections, memory observations, API calls, system logs, static-analysis findings, and analyst observations.

These observations are useful individually, but an important problem remains:

> **What important parts of the malware's behavior are still unsupported, ambiguous, or incomplete?**

Existing analysis workflows commonly focus on collecting observations and identifying suspicious behavior. However, identifying what is **missing from the available evidence** is a different problem.

An analyst may observe:

```text
Process A
    ↓
creates File B
    ↓
connects to Network C
```

but the available evidence may not be sufficient to establish whether these observations belong to the same behavioral chain.

The problem is therefore not simply:

> "What did the malware do?"

It is also:

> "What can we actually prove from the available evidence, and what remains unresolved?"

TraceGap addresses this second problem.

---

## 2. Hypothesis

If heterogeneous malware-analysis observations are represented in a structured model and their relationships are explicitly tracked, it is possible to identify gaps in behavioral reconstruction.

A **gap** represents a part of the analytical explanation that is:

* missing;
* weakly supported;
* ambiguous;
* disconnected from related observations; or
* dependent on evidence that has not yet been collected.

By explicitly representing these gaps, an analysis system can help analysts determine what should be investigated next.

---

## 3. Objective

The objective of TraceGap is to provide an open-source framework capable of:

1. representing observations produced during malware analysis;
2. associating observations with their supporting evidence;
3. representing relationships between observations;
4. identifying incomplete or weakly supported behavioral chains;
5. representing these deficiencies as explicit gaps;
6. assigning analytical priority to gaps; and
7. providing an auditable explanation for why a gap was identified.

TraceGap is therefore intended to answer:

> **"Where is the evidence insufficient to confidently reconstruct the malware's behavior?"**

---

## 4. Core Concept

The fundamental object of TraceGap is the **Gap**.

A Gap is not necessarily an indication that malicious behavior occurred.

Instead, it represents a deficiency in the current analytical knowledge.

For example:

```text
Observation 1
powershell.exe started

Observation 2
powershell.exe created payload.dll

Observation 3
payload.dll was loaded

Observation 4
network connection established

             ↓

Potential behavioral chain:

PowerShell
    ↓
payload.dll
    ↓
execution
    ↓
network communication
```

Suppose the available evidence does not establish the relationship between the DLL execution and the network connection.

TraceGap should represent:

```text
GAP-001

Type:
Missing Relationship

Description:
Insufficient evidence connecting payload.dll
execution with the observed network connection.

Impact:
Behavioral reconstruction incomplete.

Priority:
High
```

The system should not invent the missing relationship.

It should explicitly represent the uncertainty.

---

## 5. What TraceGap Does

TraceGap focuses on **analytical gaps**.

The initial version should support:

### 5.1 Observation Representation

Represent individual observations such as:

* process creation;
* process termination;
* file creation;
* file modification;
* registry modification;
* network connection;
* memory-related observation;
* API call;
* command execution.

An observation describes something that was observed.

It does not automatically describe intent.

---

### 5.2 Evidence Representation

Every important analytical conclusion should be traceable to evidence.

Conceptually:

```text
Conclusion
     ↓
Behavior
     ↓
Relationship
     ↓
Observation
     ↓
Evidence
```

This allows an analyst to navigate from an identified gap back to the evidence that caused the gap to exist.

---

### 5.3 Relationship Representation

TraceGap must explicitly represent relationships between observations.

For example:

```text
Event A ──creates──> Event B
Event B ──loaded by──> Event C
Event C ──connected to──> Event D
```

A relationship should contain enough information to explain:

* what is connected;
* why it is connected;
* what evidence supports the connection;
* how confident the relationship is.

---

### 5.4 Gap Detection

TraceGap should identify situations such as:

```text
Missing evidence
Missing relationship
Ambiguous relationship
Incomplete chain
Unsupported conclusion
Conflicting observations
Unresolved dependency
```

The initial implementation should not attempt to detect every possible type automatically.

The first objective is to establish a precise and auditable model.

---

### 5.5 Gap Prioritization

Not every gap has the same importance.

A future version should therefore allow gaps to be prioritized.

For example:

```text
HIGH
Missing relationship affecting execution chain

MEDIUM
Unclear relationship between process and file

LOW
Incomplete contextual metadata
```

Priority should be explainable rather than arbitrary.

---

## 6. What TraceGap Does Not Do

TraceGap is **not** initially intended to be:

* an antivirus;
* a malware detector;
* a sandbox;
* an EDR;
* a packet capture system;
* a reverse-engineering disassembler;
* a debugger;
* a replacement for existing malware-analysis platforms.

TraceGap should instead consume information produced by these systems and analyze the **completeness and evidentiary quality of the resulting behavioral reconstruction**.

Conceptually:

```text
Existing Tools
      │
      ▼
Observations
      │
      ▼
   TraceGap
      │
      ▼
Behavioral Gaps
      │
      ▼
Analyst Investigation
```

---

## 7. Initial Use Case

The first use case is intentionally small.

### Scenario

An analyst executes a malware sample in a controlled environment.

The analysis produces:

```text
E1:
process_created
actor = malware.exe
target = powershell.exe

E2:
file_created
actor = powershell.exe
target = C:\Temp\payload.dll

E3:
module_loaded
actor = powershell.exe
target = payload.dll

E4:
network_connection
actor = powershell.exe
target = 192.0.2.10:443
```

The observations exist.

However, the system cannot automatically assume:

```text
E3 → E4
```

represents a causal behavioral relationship.

If no supporting evidence establishes that relationship, TraceGap should identify:

```text
G1

Gap Type:
Missing Relationship

Affected Observations:
E3
E4

Description:
The available evidence does not sufficiently establish
the relationship between payload.dll execution and
the observed network connection.

Status:
Open
```

The analyst can then investigate the gap.

---

## 8. Core Data Model

The initial conceptual model is:

```text
┌──────────────┐
│    Event     │
│ observation  │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│   Evidence   │
│  provenance  │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ Relationship  │
│ event ↔ event │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│     Gap      │
│ deficiency   │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│   Priority   │
│ investigation│
└──────────────┘
```

These objects form the initial foundation of TraceGap.

---

## 9. Design Principles

### 9.1 Observation ≠ Interpretation

An observation must not automatically become a behavioral conclusion.

Example:

```text
Observation:
WriteProcessMemory() called

NOT automatically:

Conclusion:
Process Injection
```

The relationship requires additional evidence and analysis.

---

### 9.2 Absence of Evidence ≠ Evidence of Absence

If TraceGap does not observe an event, it must not automatically conclude that the event did not occur.

Instead:

```text
No observation
      ≠
Event did not happen
```

This distinction is fundamental to malware analysis.

---

### 9.3 Preserve Uncertainty

When evidence is insufficient, TraceGap should preserve the uncertainty instead of forcing a conclusion.

```text
Known
  ↓
Supported

Unknown
  ↓
Gap

Ambiguous
  ↓
Gap

Contradictory
  ↓
Gap
```

---

### 9.4 Auditability

Every generated gap should eventually be explainable.

An analyst should be able to ask:

```text
Why was this gap created?
       ↓
Which observations caused it?
       ↓
Which evidence was considered?
       ↓
Which relationship is missing?
       ↓
Why was this priority assigned?
```

---

### 9.5 Evidence Preservation

TraceGap should never destroy the original observations merely because they appear duplicated, contradictory, or irrelevant.

Analytical operations should remain distinguishable from raw observations.

---

## 10. MVP Definition

TraceGap v0.1 will be considered successful if it can:

1. ingest a small set of normalized events;
2. assign each event a unique identity;
3. represent evidence associated with events;
4. represent relationships between events;
5. identify at least one class of analytical gap;
6. store the gap and its reason;
7. associate the gap with affected evidence/events; and
8. produce a human-readable explanation.

The first MVP does **not** need machine learning.

It does **not** need automatic malware classification.

It does **not** need a graphical interface.

It does **not** need to support every malware-analysis source.

The priority is establishing a correct and auditable analytical model.

---

## 11. First Gap Type

The first gap type selected for TraceGap v0.1 is:

> **Missing Relationship**

Definition:

> A Missing Relationship exists when two or more observations appear relevant to the same behavioral reconstruction, but the available evidence is insufficient to establish the relationship between them.

Example:

```text
E1 ──?──> E2

E1:
payload.dll loaded

E2:
network connection

Relationship:
UNKNOWN

Gap:
MISSING_RELATIONSHIP
```

This intentionally narrow scope will allow the first implementation to be small enough to understand and test.

---

## 12. Future Direction

Future versions may introduce:

* additional gap types;
* confidence scoring;
* temporal reasoning;
* process-tree reasoning;
* causal hypotheses;
* graph-based reconstruction;
* static-analysis evidence;
* dynamic-analysis evidence;
* memory-analysis evidence;
* analyst annotations;
* automated gap discovery;
* gap prioritization algorithms;
* visualization;
* integration with malware sandboxes and analysis frameworks.

These capabilities are outside the initial v0.1 scope.

---

## 13. Definition of Success

TraceGap succeeds when an analyst can look at an incomplete behavioral reconstruction and receive an explicit answer to:

> **"What part of this explanation is currently unsupported, and what evidence would help resolve it?"**

That is the central problem TraceGap exists to solve.
