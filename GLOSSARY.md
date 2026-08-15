# Glossary

## OntoFlow

An ontology-native workflow automation architecture. OntoFlow separates business meaning, workflow execution, validation, persistent knowledge, and optional approval components.

## Ontology Manager

The component that owns type definitions, persistent objects, property values, relation instances, import jobs, and ontology change history.

## Workflow Manager

The component that owns workflow definitions, role declarations, node declarations, state slots, step runs, and action execution logs.

## ObjectType

A type of business object, such as `Patient`, `Doctor`, `Appointment`, `Equipment`, or `WorkOrder`.

## PropertyDefinition

A typed property attached to an ObjectType. It defines value type, requirement, enum values, ranges, patterns, and related validation rules.

## RelationType

A typed relationship between two ObjectTypes.

## ActionType

A typed execution contract for an operation. It defines required object roles, produced object roles, input schema, output schema, connector references, retry policy, and dry-run behavior.

## State Slot

A workflow runtime value identified by run, object role, and slot name. It stores the current value used by workflow steps.

## State Slot History

The workflow-level history of state slot changes. It answers which step, action, or input changed a runtime value.

## Persistent Knowledge

The durable ontology data layer: object instances, property values, relation instances, and ontology change history.

## Apply Boundary

The single write boundary for persistent knowledge. Imports, direct edits, and workflow write-back all pass through this boundary.

## Unified Validator

The shared validation component used by slot writes, imports, direct edits, dry-runs, promotion checks, and type-change impact analysis.

## Import Job

A tracked unit for bulk ingestion. A job contains normalized import items and validation results.

## TBox

The type layer: object types, properties, relations, action types, and workflow types.

## ABox

The instance layer: object instances, property values, and relation instances.

## Gate

An optional approval component for promoted workflow changes. Gate owns candidates and approval state, not persistent facts.

## Change Candidate

A proposed change awaiting approval. It is not the same as change history because it can be rejected.

## Ontology Change History

The immutable history of actual changes to TBox or ABox data.

## Conversational Input

An optional front component that turns utterances into workflow candidates and slot fill candidates. It does not directly edit ontology data or execute actions.

## Dry-Run

A non-persistent execution or validation mode used to inspect mappings, missing values, validation failures, and expected changes before committing.
