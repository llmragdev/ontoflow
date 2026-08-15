# OntoFlow Overview Design

[한국어 정본](../ko/01-overview.md)

This is an English summary of the canonical Korean overview document.

## Goal

OntoFlow combines a commercial-grade Ontology Manager with a commercial-grade Workflow Manager.

The core product is button/form-first deterministic workflow automation. Conversational input, Gate, WriteBack, and automation triggers are composable components around the core.

## Core Components

| Component | Role |
|---|---|
| Ontology Manager | Type registry, persistent knowledge, imports, validation, change history |
| Workflow Manager | Workflow definitions, role bindings, state slots, action execution |
| Button/Form UI | Explicit and auditable execution path |
| Conversational Input | Front component that proposes workflow runs and slot values |
| Gate / WriteBack | Extension for approval and controlled persistence |
| Automate | Later trigger-based execution extension |

## Main Principles

- The core must work without conversational input.
- The core must work without Gate.
- Persistent knowledge has three write paths: bulk import, direct edit, and workflow write-back.
- All write paths use the same Unified Validator and Apply boundary.
- TBox bulk changes require proposal, impact analysis, and approval.
- Change candidates and actual change history are different records.

For the full overview, see the Korean canonical document.
