# OntoFlow

[한국어](README.md) | English

Business process automation that is easy to understand, validate, and trace.

The Korean documents are the source documents. English documents are translations or summaries.

OntoFlow is a public architecture project that connects business data models with workflow execution. The goal is to make business automation easier for people to inspect, validate, trace, and evolve.

## Why

Many automation systems start from a conversational interface or a chain of tool calls. That is useful, but real business workflows also need clear answers to practical questions:

- Which business data changed?
- Which workflow step produced the value?
- Which API, MCP tool, or internal service was called?
- Which validation rule passed or failed?
- When does a proposed change become actual business data?

OntoFlow treats these as core architecture concerns.

## Core Ideas

- **Business Data Model Manager (Ontology Manager)**: manages business objects, properties, relations, bulk imports, and change history.
- **Business Process Runner (Workflow Manager)**: manages workflow steps, input values, state changes, external calls, and execution logs.
- **Shared Validator (Unified Validator)**: makes inputs, imports, direct edits, and dry-runs use the same validation rules.
- **Apply Boundary**: keeps all actual data changes behind one controlled write path.
- **Composable Components**: conversational input, approval screens, write-back, and automation triggers attach around the core.

## Current Release

This repository starts with design documents. Implementation code will be added only after the architecture and public contribution model are stable.

Recommended reading order:

1. [Overview Design](docs/en/01-overview.md)
2. [Architecture Specification](docs/en/02-architecture-spec.md)
3. [Glossary](GLOSSARY.en.md)
4. [Hospital Reservation Example](examples/en/hospital-reservation.md)
5. [Open Questions](docs/en/03-open-questions.md)

Canonical Korean documents:

1. [개괄 설계](docs/ko/01-overview.md)
2. [상세 설계](docs/ko/02-architecture-spec.md)
3. [용어집](GLOSSARY.md)

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

- Commercial-grade business process runner design.
- Commercial-grade business data model manager design.
- Button/form-first workflow execution.
- REST, MCP, and internal service action contracts.
- Bulk import, validation, apply, and change-history contracts.

Extension scope:

- Conversational input module.
- Gate and WriteBack prototype.
- Later automation triggers.

## License

Documents are intended to be shared under CC BY 4.0. Example code, when added, is intended to use Apache License 2.0.

See [NOTICE](NOTICE) for the current repository-level license note.
