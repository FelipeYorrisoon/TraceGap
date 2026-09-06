# TraceGap

**An open-source framework for identifying, explaining, and prioritizing potential gaps in malware analysis.**

> **TraceGap asks not only *"What did the malware do?"*, but also *"What can we actually prove from the available evidence, and what remains unresolved?"***

---

## Overview

Malware analysis produces a large amount of heterogeneous information.

A sandbox may generate process events, file operations, registry modifications, network connections, API calls, memory artifacts, and other observations.

The problem is that collecting observations does not necessarily mean that we can reconstruct the malware's behavior with confidence.

Important relationships may remain:

* missing;
* ambiguous;
* incomplete;
* unsupported by sufficient evidence;
* or difficult to explain and reproduce.

**TraceGap** is designed to identify these gaps in behavioral reconstruction.

Instead of focusing primarily on malware detection, TraceGap focuses on a different question:

> **Where is the available evidence insufficient to confidently reconstruct the observed behavior?**

---

## The Problem

Existing malware-analysis workflows can generate thousands of observations.

For example:

```text
process_created
file_created
module_loaded
network_connection
registry_modified
memory_written
process_terminated
```

These observations are valuable, but observations alone do not necessarily explain behavior.

Consider:

```text
E1: malware.exe creates powershell.exe

E2: powershell.exe loads payload.dll

E3: powershell.exe establishes a network connection
```

We may observe all three events.

But do we have enough evidence to establish that:

```text
payload.dll → network connection
```

is actually a meaningful behavioral relationship?

Maybe.

Maybe not.

The available evidence may be insufficient to establish the connection.

A traditional analysis workflow may still lead an analyst toward an interpretation.

TraceGap takes a different approach:

```text
Observation
     ↓
Evidence
     ↓
Relationship
     ↓
Can the relationship be supported?
     ↓
       NO
     ↓
Potential Gap
```

The goal is not to invent an explanation.

The goal is to **identify where the explanation is not sufficiently supported**.

---

## Core Idea

TraceGap introduces an analytical layer between raw observations and behavioral conclusions.

```text
             Malware
                │
                ▼
          Analysis Tools
                │
                ▼
           Observations
                │
                ▼
             Evidence
                │
                ▼
          Relationships
                │
                ▼
            TraceGap
                │
        ┌───────┴────────┐
        ▼                ▼
      Gaps            Supported
        │             Relations
        ▼
    Priorities
        │
        ▼
   Investigation
```

TraceGap does not replace malware-analysis tools.

Instead, it consumes their observations and helps determine whether those observations provide enough support for behavioral reconstruction.

---

# What TraceGap Does

TraceGap is intended to provide a framework for:

* representing observations;
* preserving evidence;
* representing relationships between observations;
* identifying unsupported or incomplete relationships;
* explicitly representing uncertainty;
* explaining why a potential gap exists;
* associating gaps with the observations and evidence that produced them;
* prioritizing gaps for further investigation.

The central objective is **evidence-aware behavioral reconstruction**.

---

# What TraceGap Does NOT Do

TraceGap is not intended to initially replace:

* antivirus software;
* EDR platforms;
* malware sandboxes;
* packet capture systems;
* debuggers;
* disassemblers;
* reverse-engineering frameworks;
* memory forensics tools;
* malware detection engines.

TraceGap is an **analytical framework**, not a malware execution or detection platform.

Existing tools can generate observations.

TraceGap analyzes whether those observations are sufficient to support relationships and behavioral conclusions.

---

# A Different Question

Traditional malware analysis often asks:

> **What did the malware do?**

TraceGap adds another question:

> **How much of that behavior can we actually support with evidence?**

This distinction is important.

For example:

```text
Observed:
    WriteProcessMemory()

Possible interpretation:
    Process Injection
```

The observation does not automatically prove the interpretation.

There may be additional evidence required.

TraceGap therefore separates:

```text
Observation ≠ Interpretation
```

This principle is fundamental to the project.

---

# Core Concepts

TraceGap is initially built around a small set of concepts.

```text
Event
  │
  ▼
Evidence
  │
  ▼
Relationship
  │
  ▼
Gap
  │
  ▼
Priority
```

## Event

An **Event** represents an observation produced during analysis.

Examples:

```text
process_created
file_created
module_loaded
network_connection
registry_modified
memory_written
```

An event describes **what was observed**.

It should not automatically describe **what the observation means**.

---

## Evidence

**Evidence** represents the information that supports an observation or relationship.

Evidence may originate from sources such as:

* sandbox telemetry;
* system monitoring;
* network monitoring;
* memory analysis;
* static analysis;
* dynamic analysis;
* debugger observations;
* analyst annotations.

The purpose is to preserve the basis on which an analytical decision was made.

---

## Relationship

A **Relationship** represents a connection between observations.

For example:

```text
E1 ───────► E2
```

might represent:

```text
process_created
       │
       ▼
module_loaded
```

However, relationships should not be created merely because two events appear related.

They should be supported by available evidence.

---

## Gap

A **Gap** represents a potentially important deficiency in the available evidence or reconstruction.

Examples may include:

```text
Missing Relationship
Ambiguous Relationship
Incomplete Evidence
Unsupported Interpretation
Missing Context
```

The first gap class targeted by the project is:

### Missing Relationship

A **Missing Relationship** exists when two or more observations appear relevant to the same behavioral reconstruction, but the available evidence is insufficient to establish their relationship.

Example:

```text
E1: payload.dll loaded
E2: network connection established

Available evidence:

E1 ─────── ? ─────── E2
```

TraceGap may represent this as:

```text
G1

Type:
    Missing Relationship

Affected Events:
    E1
    E2

Status:
    Open
```

The system does not automatically claim that the events are causally related.

It records that the relationship remains insufficiently supported.

---

## Priority

Not every gap has the same importance.

A future TraceGap component will allow gaps to be prioritized according to factors such as:

* behavioral importance;
* evidence quality;
* uncertainty;
* potential impact on the analysis;
* number of dependent conclusions;
* analyst-defined severity.

The objective is to help analysts answer:

> **Which unresolved gap should I investigate first?**

---

# Design Principles

## 1. Observation Is Not Interpretation

An observed event must not automatically become a behavioral conclusion.

```text
Event
  ≠
Behavior
```

Example:

```text
WriteProcessMemory()
```

does not automatically mean:

```text
Process Injection
```

Additional evidence may be necessary.

---

## 2. Absence of Evidence Is Not Evidence of Absence

If TraceGap cannot establish a relationship, that does not mean the relationship does not exist.

Therefore:

```text
Unknown
```

must remain different from:

```text
False
```

This is essential for malware analysis.

---

## 3. Preserve Uncertainty

The framework should avoid forcing incomplete evidence into definitive conclusions.

Instead of:

```text
YES → Process Injection
```

TraceGap should be capable of representing:

```text
Potential relationship
Evidence insufficient
Further investigation required
```

---

## 4. Auditability

Every identified gap should be explainable.

An analyst should be able to move from:

```text
Gap
 ↓
Reason
 ↓
Relationship
 ↓
Evidence
 ↓
Original Observations
```

This creates an auditable analytical chain.

---

## 5. Preserve Original Observations

Original observations should not be destroyed simply because they appear duplicated, contradictory, or irrelevant.

Two observations can contain identical information and still represent separate occurrences.

Therefore:

```text
Observation A
```

and

```text
Observation B
```

must be capable of remaining distinct.

Analytical operations such as deduplication should be explicit rather than silently modifying the original evidence.

---

# Initial MVP

The first version of TraceGap will intentionally remain small.

### MVP goals

The initial implementation should support:

* normalized events;
* unique event identity;
* evidence representation;
* relationship representation;
* gap representation;
* at least one gap class;
* gap explanations;
* references from gaps back to affected observations/evidence;
* human-readable output.

### First Gap Type

The initial gap type is:

```text
Missing Relationship
```

### Example

Input:

```text
E1:
    process_created
    actor: malware.exe
    target: powershell.exe

E2:
    file_created
    actor: powershell.exe
    target: C:\Temp\payload.dll

E3:
    module_loaded
    actor: powershell.exe
    target: payload.dll

E4:
    network_connection
    actor: powershell.exe
    target: 192.0.2.10:443
```

If the available evidence cannot sufficiently establish the relationship between `E3` and `E4`, TraceGap can represent:

```text
G1:
    type: Missing Relationship

    affected_events:
        E3
        E4

    status:
        Open

    reason:
        The available evidence does not sufficiently
        establish the relationship between the loaded
        module and the observed network connection.
```

The important point is that TraceGap **does not invent the missing relationship**.

It identifies the gap.

---

# Architecture

The initial conceptual architecture is:

```text
                ┌─────────────────────┐
                │   Analysis Sources  │
                └──────────┬──────────┘
                           │
                           ▼
                ┌─────────────────────┐
                │   Event Ingestion   │
                └──────────┬──────────┘
                           │
                           ▼
                ┌─────────────────────┐
                │  Canonical Events   │
                └──────────┬──────────┘
                           │
                           ▼
                ┌─────────────────────┐
                │ Evidence & Relations│
                └──────────┬──────────┘
                           │
                           ▼
                ┌─────────────────────┐
                │    Gap Analysis     │
                └──────────┬──────────┘
                           │
                           ▼
                ┌─────────────────────┐
                │   Gap Prioritizer   │
                └──────────┬──────────┘
                           │
                           ▼
                ┌─────────────────────┐
                │ Analyst Investigation│
                └─────────────────────┘
```

The architecture is intentionally modular so that future versions can support different analysis sources without changing the core analytical model.

---

# Planned Data Model

The initial conceptual model is:

```text
Event
 ├── Identity
 ├── Observation
 ├── Context
 └── Provenance

Evidence
 ├── Source
 ├── Reference
 └── Quality

Relationship
 ├── Source Events
 ├── Target Events
 ├── Evidence
 └── Status

Gap
 ├── Type
 ├── Affected Events
 ├── Supporting Evidence
 ├── Reason
 └── Status

Priority
 ├── Severity
 ├── Confidence
 └── Investigation Value
```

The exact schema is still under development.

---

# Example Workflow

A simplified TraceGap workflow:

```text
1. Collect observations

2. Normalize observations

3. Assign unique identities

4. Preserve provenance

5. Represent available evidence

6. Establish supported relationships

7. Identify unsupported relationships

8. Create gaps

9. Explain each gap

10. Prioritize investigation
```

This allows an analyst to move from raw observations toward an explicit understanding of what is known and what remains unresolved.

---

# Why This Matters

Malware behavior is often reconstructed from incomplete and heterogeneous evidence.

A conclusion may depend on relationships between events that are not directly observed.

For example:

```text
Process A
    │
    ├── creates Process B
    │
    ├── loads Module C
    │
    └── connects to Network D
```

The important analytical question is not simply whether these events occurred.

It is:

```text
What evidence connects them?
```

If the connection is weak or missing, the analyst should know that.

TraceGap is intended to make these weaknesses explicit.

---

# Project Status

**Current status: Early-stage research and architecture definition.**

The project is currently focused on defining:

* the problem;
* the analytical model;
* the core entities;
* evidence preservation;
* relationship modeling;
* the first gap class;
* the architecture for the MVP.

The project does **not** currently claim to provide a production-ready malware-analysis platform.

The initial goal is to establish a rigorous foundation before implementing more advanced functionality.

---

# Roadmap

## Phase 0 — Foundation

* [x] Define project problem
* [x] Define project objective
* [x] Define initial concepts
* [x] Define initial gap type
* [ ] Formalize data model
* [ ] Define event schema
* [ ] Define evidence schema
* [ ] Define relationship schema
* [ ] Define gap schema

## Phase 1 — MVP

* [ ] Implement canonical events
* [ ] Implement event identity
* [ ] Implement evidence representation
* [ ] Implement relationships
* [ ] Implement `Missing Relationship`
* [ ] Generate human-readable gap explanations
* [ ] Create basic test dataset

## Phase 2 — Analytical Engine

* [ ] Additional gap types
* [ ] Relationship validation
* [ ] Temporal reasoning
* [ ] Process-tree reasoning
* [ ] Evidence quality evaluation
* [ ] Gap prioritization
* [ ] Analyst annotations

## Phase 3 — Reconstruction

* [ ] Behavioral graphs
* [ ] Sequence reconstruction
* [ ] Cross-source evidence correlation
* [ ] Static + dynamic evidence correlation
* [ ] Memory evidence integration
* [ ] Causal hypotheses
* [ ] Explainable inference

## Phase 4 — Ecosystem

* [ ] Sandbox integrations
* [ ] Analysis-tool adapters
* [ ] Visualization
* [ ] API
* [ ] CLI improvements
* [ ] Documentation
* [ ] Research datasets

---

# Research Direction

TraceGap is intended to explore an important question in malware analysis:

> **Can we systematically measure where a behavioral reconstruction is incomplete or insufficiently supported?**

The project therefore sits at the intersection of:

* Malware Analysis
* Reverse Engineering
* Digital Forensics
* Behavioral Analysis
* Evidence Modeling
* Graph Analysis
* Explainable Security
* Security Data Engineering

The long-term goal is to make analytical uncertainty **explicit, traceable, and actionable**.

---

# Repository Structure

The project structure will evolve as implementation begins.

The planned structure is approximately:

```text
TraceGap/
│
├── README.md
├── LICENSE
├── CONTRIBUTING.md
├── SECURITY.md
├── CHANGELOG.md
├── ROADMAP.md
│
├── docs/
│   ├── architecture/
│   │   ├── problem-statement.md
│   │   ├── canonical-event-model.md
│   │   └── overview.md
│   │
│   ├── research/
│   │   ├── related-work.md
│   │   └── gap-analysis.md
│   │
│   └── decisions/
│       └── ADR-0001-*.md
│
├── src/
│
├── tests/
│
└── examples/
```

Only components that are actually implemented will be considered part of a stable release.

---

# Documentation

Architecture and research documentation will live under:

```text
docs/
```

Important documents include:

* `problem-statement.md`
* `canonical-event-model.md`
* `overview.md`
* research and gap-analysis documents
* Architecture Decision Records (ADRs)

The documentation is considered part of the project itself because the analytical model must be understandable and auditable.

---

# Contributing

TraceGap is an open-source research project.

Contributions related to the project's core problem are welcome, including:

* architecture discussions;
* data-model proposals;
* malware-analysis research;
* evidence modeling;
* relationship modeling;
* gap-detection strategies;
* test datasets;
* documentation;
* implementation;
* reproducible experiments.

Before implementing a major feature, contributors should first discuss how it fits into the analytical model.

See `CONTRIBUTING.md` for project contribution guidelines.

---

# Security

TraceGap is designed for security research and defensive analysis.

Malware samples should only be analyzed in controlled environments and with appropriate authorization.

See `SECURITY.md` for security-related project guidance.

---

# License

The project license will be defined as the implementation and contribution structure are finalized.

---

# Philosophy

TraceGap is built around a simple idea:

> **A good malware analysis should not only explain what happened. It should also make clear what cannot yet be proven.**

The project therefore prioritizes:

```text
Evidence
   ↓
Traceability
   ↓
Relationships
   ↓
Uncertainty
   ↓
Explainability
```

rather than treating every observed event as an unquestionable conclusion.

---

# Long-Term Vision

The long-term vision is to create a framework capable of transforming large amounts of heterogeneous malware-analysis data into an explainable reconstruction of behavior while explicitly identifying the places where evidence is insufficient.

Conceptually:

```text
Raw Analysis Data
        │
        ▼
     Events
        │
        ▼
    Evidence
        │
        ▼
  Relationships
        │
        ▼
 Behavioral Model
        │
        ├───────────────┐
        ▼               ▼
   Supported        Unsupported
   Behavior         Relationships
                        │
                        ▼
                      Gaps
                        │
                        ▼
                    Priority
                        │
                        ▼
                  Investigation
```

The ultimate objective is not to make the analyst trust the system blindly.

It is the opposite:

> **Make every important conclusion traceable back to evidence—and make every important uncertainty visible.**

---

## Status

**TraceGap is an early-stage open-source research project.**

The architecture and analytical model are being developed before the implementation of the full engine.

---

## Core Question

If there is one question that defines TraceGap, it is:

> **What important part of the malware's behavior can we not yet adequately support with the evidence we have?**
An open-source framework for identifying, explaining, and prioritizing potential gaps in malware analysi