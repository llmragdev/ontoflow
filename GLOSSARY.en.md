# Glossary

[한국어](GLOSSARY.md) | English

The Korean glossary is the source document. This English glossary is a translation summary.

## OntoFlow

A workflow automation architecture that connects business data models with business process execution. OntoFlow makes data changes easier to understand, validate, and trace.

## Ontology Manager

The business data model manager. It manages business objects, properties, relationships, bulk imports, and change history.

## Workflow Manager

The business process runner. It manages workflow steps, input values, state changes, external calls, and execution logs.

## ObjectType

A kind of business thing, such as `Patient`, `Doctor`, `Appointment`, `Equipment`, or `WorkOrder`.

## PropertyDefinition

A defined field on a business object, such as patient name, appointment time, or equipment status. It defines value format, requirement, allowed values, and validation rules.

## RelationType

A relationship between two business things. For example, a patient has an appointment, or a doctor handles a visit.

## ActionType

A definition of an executable operation. It states what input is needed, what output is produced, and which external system may be called.

## State Slot

A temporary value used while a workflow is running, such as the selected patient, requested date, or available appointment time.

## State Slot History

A record of how temporary workflow values changed: when, where, and by which step or input.

## Persistent Knowledge

The actual stored business data, including business objects, values, relationships, and change history.

## Apply Boundary

The shared write path for actual business data changes. Bulk imports, direct edits, and workflow results all pass through this path.

## Unified Validator

The shared validation component. It makes inputs, imports, direct edits, dry-runs, and impact checks use the same rules.

## Import Job

A tracked unit for validating and applying many records from a file or API.

## TBox

The definition layer. It describes which business object types, fields, relationships, actions, and workflow types exist.

## ABox

The actual data layer. It contains real patients, appointments, equipment, work orders, values, and relationships.

## Gate

An optional approval component. It lets people review proposed workflow changes before they become actual data changes.

## Change Candidate

A proposed change that has not been applied yet. It may be approved or rejected.

## Ontology Change History

The record of actual changes to business data definitions or actual business data.

## Conversational Input

An optional input component that turns user messages into workflow candidates and input-value candidates. It does not directly change data or execute actions.

## Dry-Run

A preview mode. It checks missing values, validation failures, and expected changes without applying them.
