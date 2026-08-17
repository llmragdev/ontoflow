# Public Review Topics

[한국어](../ko/03-open-questions.md)

This English page is a summary of the Korean reference document.

This page lists design topics where outside review is useful. Each topic first states OntoFlow's current position, then explains what kind of feedback would help. The questions are **example prompts for discussion**, not unsupported open-ended questions.

## 1. Workflow State Passing

### Background

Workflow steps need to pass values from one step to another. In a hospital reservation workflow, this may include the selected patient, department, doctor, requested date, and available appointment slot.

### Current Position

OntoFlow favors named `State Slot` values over direct data wires between nodes.

```text
Control flow   Node A -> Node B -> Node C
Data values    selected_patient, requested_date, available_slot
```

This makes it easier to trace when a value changed and to record external API or MCP tool results in the same state history.

### Feedback We Want

- Whether named state slots are intuitive enough for real workflow automation
- Whether some UI or developer scenarios still need visible data wires
- How to display state names and roles in parallel branches or repeated runs

### Example Prompt

Should workflow values move through explicit data wires, or should nodes read and write named state slots?

## 2. Ontology Scope

### Background

The ontology manages business object types, properties, relationships, and action definitions. The design question is how far these definitions should participate during workflow execution.

### Current Position

OntoFlow expects ontology definitions to:

- define object roles and slot types,
- validate incoming slot values,
- constrain persistent writes,
- provide explainable references for data changes.

### Feedback We Want

- Whether ontology checks could slow early workflow design too much
- How users should select object types and properties in the UI
- How temporary values should be handled before a type is finalized

### Example Prompt

How far should ontology definitions participate in workflow execution?

## 3. Conversational Input

### Background

A user may say, "Book patient Kim for next Tuesday morning" instead of filling out a form. The question is whether conversational input belongs inside the workflow engine or in front of it.

### Current Position

OntoFlow treats conversational input as a front component. It proposes workflow runs and slot values, but `Workflow Manager` remains responsible for execution.

```text
User message -> Conversational Input -> run/value candidates -> Workflow Manager
```

### Feedback We Want

- How to show confidence and evidence for values produced by conversation
- Whether a confirm-and-run button flow is enough
- How to fall back to form input when conversation fails

### Example Prompt

Should natural-language input be part of the workflow engine or a front input component?

## 4. Gate and Approval Boundary

### Background

Workflow results may change real business data. Some changes should be reviewed before they are applied, especially external write-back, customer data updates, or definition changes.

### Current Position

OntoFlow treats `Gate` as an extension component. The core must still work through bulk import and direct edit paths. Gate reviews workflow-generated change candidates before they pass through the Apply Boundary.

```text
Workflow Manager -> Change Candidate -> Gate approval -> Apply Boundary -> actual data change
```

### Feedback We Want

- Which changes should apply immediately and which should require approval
- How an approval screen should show previous value, candidate value, evidence, and impact
- What minimum controls are needed even without Gate

### Example Prompt

Should approval be core behavior or an extension component?

## 5. Public Terminology

### Background

OntoFlow combines ontology, workflow, state history, validation, and approval boundaries. If terms are too hard or look product-specific, new contributors may lose context.

### Current Position

Public documents use plain language first and technical terms where needed.

```text
TBox = class/type layer
ABox = instance/data layer
ObjectType = class/type of business object
ObjectInstance = actual business object instance
ActionType = executable business operation definition
```

### Feedback We Want

- Clear Korean wording for general readers
- Stable English identifiers for implementation documents
- Consistent terminology between Korean reference documents and English summaries

### Example Prompt

Which terms are clearest for public documentation?

## How to Give Feedback

GitHub Discussions are preferred for topic-level feedback.

Useful feedback format:

```text
Topic: Workflow state passing
Scenario: A hospital reservation workflow refers to the same patient and appointment candidates across multiple steps.
Suggestion: State slots are good for traceability, but the UI should also show where values were created and consumed.
Reason: Operators first want to see "where this value came from and where it went."
```
