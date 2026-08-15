# OntoFlow

Ontology-native workflow automation for explainable business operations.

OntoFlow is a public architecture project for combining an ontology manager with a deterministic workflow manager. The goal is to make business automation easier to inspect, validate, and evolve.

## Why

Many automation systems start from a conversational interface or a chain of tool calls. That is useful, but enterprise workflows also need a stable model for answering practical questions:

- Which business object is being changed?
- Which workflow step produced the value?
- Which API, MCP tool, or internal service was called?
- Which validation rule passed or failed?
- When does a candidate change become persistent knowledge?

OntoFlow treats these as core architecture concerns.

## Core Ideas

- **Ontology Manager**: owns object types, properties, relations, action types, persistent objects, import jobs, and change history.
- **Workflow Manager**: owns workflow definitions, role bindings, state slots, step runs, action logs, and deterministic execution.
- **Unified Validator**: provides one validation path for slot writes, imports, direct edits, dry-runs, and promotion checks.
- **Apply Boundary**: the only path that changes persistent ontology knowledge.
- **Composable Components**: conversational input, approval gates, write-back, and automation triggers attach around the core instead of replacing it.

## Current Release

This repository starts with design documents. Implementation code will be added only after the architecture and public contribution model are stable.

Recommended reading order:

1. [Overview Design](docs/01-overview.md)
2. [Architecture Specification](docs/02-architecture-spec.md)
3. [Glossary](GLOSSARY.md)
4. [Hospital Reservation Example](examples/hospital-reservation.md)
5. [Open Questions](docs/03-open-questions.md)

## Repositories

| Repository | Visibility | Role |
|---|---|---|
| `ontoflow` | Public | Public design, examples, discussions, releases |
| `ontoflow-lab` | Private | Drafts, deeper review, release curation for invited contributors |

Public discussion happens in this repository. Private lab access is invite-only and is considered after public discussion or the contributor survey.

## Participate

You can participate by:

- Asking design questions in GitHub Discussions.
- Suggesting use cases.
- Reviewing the ontology/workflow boundaries.
- Proposing clearer terminology.
- Improving documentation.
- Sharing experience from enterprise workflow automation, data modeling, or governance systems.

Contributor survey:

```text
Survey link will be added here.
```

## Project Scope

2026 core scope:

- Commercial-grade Workflow Manager design.
- Commercial-grade Ontology Manager design.
- Button/form-first workflow execution.
- REST, MCP, and internal service action contracts.
- Import, validation, apply, and change-history contracts.

Extension scope:

- Conversational input module.
- Gate and WriteBack prototype.
- Later automation triggers.

## License

Documents are intended to be shared under CC BY 4.0. Example code, when added, is intended to use Apache License 2.0.

See [NOTICE](NOTICE) for the current repository-level license note.
