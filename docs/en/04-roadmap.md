# Roadmap

[한국어 기준 문서](../ko/04-roadmap.md)

This English page is a summary of the Korean source document.

This document explains the implementation order for OntoFlow.

Names such as `Core-1` and `Core-2` are internal development phase identifiers. They are not product version numbers. They show which foundation should be built first and which capabilities are layered on top.

## 2026 Core Scope

The 2026 goal is to make **Workflow Manager** and **Ontology Manager** production-grade. Conversational input, Gate, and automation are extension components around the core.

## Implementation Order

| Phase | What It Builds | Why It Comes First |
|---|---|---|
| Core-1 | Type registry foundations | Object, property, relationship, and action definitions are needed before validation and execution |
| Core-2 | Persistent knowledge store and Apply boundary | Actual business data and change history need one shared write path |
| Core-3 | Unified Validator minimum rule set | Inputs, imports, direct edits, and workflow results should use the same validation rules |
| Core-4 | Bulk import and dry-run | Files and APIs need a way to validate and apply many records |
| Core-5 | Type change proposals and approval | Changes to business definitions need impact analysis and approval |
| Core-6 | Workflow definition structure | Business processes, roles, nodes, and control flow need to be stored |
| Core-7 | Slot declarations and action bindings | Each node's reads, writes, and action calls need static checks |
| Core-8 | Workflow runtime state | Runs need role bindings, state slots, and step progress |
| Core-9 | State history and execution evidence | Data changes need traceable step, value, and external-call history |
| Core-10 | Button/form execution and reports | Users need explicit execution and pre-run validation reports |

## Term Notes

| Term | Meaning |
|---|---|
| Type registry | Definitions of object types, properties, relationships, and actions |
| Persistent knowledge | Stored business data: object instances, property values, relationships, and change history |
| Apply boundary | The shared path for actual business data changes |
| Unified Validator | A shared validator for all inputs and writes |
| ImportJob | A tracked unit for validating and applying many records from a file or API |
| ImportItem | One item inside an ImportJob, such as one row or one change item |
| ABox bulk import | Bulk loading of actual business data instances |
| TBox change | A change to business definitions such as object types, properties, or relationships |
| dry-run | A preview mode that validates and reports expected results without applying changes |
| State Slot | A named temporary value used during workflow execution |

## Extension Components

Extension components are attached before or after the core. They are not required for the core to run, but they use the same contracts.

| Phase | What It Builds | Description |
|---|---|---|
| Conv-P1 | Intent detection, sessions, candidate buffer | Turns user messages into run candidates and value candidates |
| Conv-P2 | Clarification, normalization, confidence | Asks for missing values and normalizes terms |
| Conv-P3 | Utterance evidence and execution suggestion | Records where proposed values came from and suggests execution |
| Gate-P1 | Change candidate storage | Stores workflow results for approval instead of applying them immediately |
| Gate-P2 | Minimal approval screen | Shows previous value, candidate value, and evidence for approval or rejection |
| Gate-P3 | WriteBack simulation | Validates external write-back flow before actual writes |

## Follow-up Scope

These items are expanded after the core stabilizes.

- Event triggers
- Automated execution
- Governance dashboards
- Domain-specific templates
