# OntoFlow Architecture Specification

[한국어 정본](../ko/02-architecture-spec.md)

This is an English summary of the canonical Korean architecture specification.

## Layers

| Layer | Description |
|---|---|
| L1 Type Registry | ObjectType, PropertyDefinition, RelationType, ActionType, WorkflowType |
| L2 Runtime State | WorkflowRun, StepRun, RoleBinding, StateSlot, ActionLog |
| L3 Change Candidates | Optional Gate-owned approval candidates |
| L4 Persistent Knowledge | ObjectInstance, PropertyValue, RelationInstance, OntologyChangeHistory |
| L5 Provenance | Document, ontology path, state-change, rule, action, and utterance evidence |

## Persistent Knowledge Write Paths

```text
bulk import
  -> Validator
  -> Apply

direct edit
  -> permission check
  -> Validator
  -> Apply

workflow write-back
  -> promotion check
  -> Gate candidate
  -> approval
  -> Apply
```

## Unified Validator

The same validator is used by:

- slot writes
- workflow dry-runs
- promotion checks
- ABox imports
- direct edits
- TBox impact analysis

Callers may handle failures differently, but validation rules and result shape stay consistent.

## Apply Boundary

Apply is the only write boundary for persistent knowledge. TBox changes require an approved type-change proposal. ABox writes may come from imports, direct edits, or workflow write-back.

## Workflow Runtime

Workflow state is stored in role-qualified state slots rather than data edges. Control edges define execution order; state slots carry values.

For the full specification, see the Korean canonical document.
