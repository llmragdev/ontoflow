# OntoFlow Overview Design

[한국어](../ko/01-overview.md) | English

The Korean document is the primary reference for detailed wording, but this English version is intended to be readable on its own.

> This document defines the product direction and the component boundaries.
> Data models, tables, and validation rules are covered in the [Architecture Spec](02-architecture-spec.md).

---

## 1. Product Goal

The 2026 target is a **production-grade Workflow Manager and a production-grade Ontology Manager**. Everything that attaches to them is treated as a composable component whose contract is fixed early, even when the implementation belongs to a later scope.

OntoFlow is not a chatbot-first architecture. Conversational input is treated as an input adapter that fills the same state slots used by button and form workflows.

```text
Core (production-grade)
  Ontology Manager      Business meaning, validation rules, persistent knowledge
  Workflow Manager      Business process and execution
  Button/Form UI        Explicit execution interface

Front assembly (modules)
  Conversational Input      Natural language -> slots
  Document / API Input      Documents and external systems -> slots

Rear assembly (initial validation scope)
  Gate / WriteBack          Change-candidate approval and controlled persistence

Side assembly (follow-up scope)
  Automate                  Event trigger -> execution
```

Ownership is deliberate:

- The Ontology Manager owns object types, properties, relations, imports, validation rules, and change history.
- The Workflow Manager owns process execution, state slots, action calls, and execution logs.
- Conversational input fills slots and proposes a run. It owns neither the process nor the data.
- Gate adds an approval boundary. It does not execute on behalf of the workflow.
- Automation replaces a human click. It does not bypass the engine.

---

## 2. Why Button/Form Comes First

If conversational automation is built first, slot filling, follow-up questions, and exception handling become dependent on the conversation engine. Separating that logic into an independent workflow core afterwards is difficult.

| Aspect | Button/form core | Conversation only |
|---|---|---|
| Parameter confirmation | Explicit | Requires interpretation |
| Testing | Deterministic | Requires probabilistic evaluation |
| Failure handling | Fall back to manual input | No alternative when recognition fails |
| Permissions | Per button and per action | Depends on utterance intent |
| Field adoption | Fast | Heavy per-customer tuning |

Once the button/form core exists, conversation becomes an adapter that fills the same slots.

```text
Input paths differ            button, form, conversation, document, external API
The target they fill is one   state slots

-> Adding input paths does not grow the core
```

---

## 3. Component Structure

```text
      +------------ Input components -------------+
      |  Button/Form   Conversational             |
      |  Document      External API               |
      +---------------------+---------------------+
                            |  fills slots
                            v
      +------------ Workflow Manager -------------+   Core
      |  Definitions, control edges, bindings     |
      |  Run state, static checks, dry-run        |
      +---------------------+---------------------+
                            |  reads type definitions / evaluates promotion
                            v
      +------------ Ontology Manager -------------+   Core
      |  Type registry, persistent knowledge      |
      |  Bulk import, Unified Validator, history  |
      +---------------------+---------------------+
                            |
                            v
      +----------- Extension components ----------+
      |  Gate -> WriteBack (initial validation)   |
      |  Automate (follow-up scope)               |
      +-------------------------------------------+
```

| Component | 2026 level | Role |
|---|---|---|
| Ontology Manager | **Production-grade** | Type registry, persistent knowledge, bulk import, change procedure, history, validation |
| Workflow Manager | **Production-grade** | Definitions, control edges, bindings, run state, dry-run |
| Button/Form UI | **Production-grade** | Explicit and auditable execution |
| Conversational Input | Module integration | Converts natural language into slot values and run proposals |
| Gate / WriteBack | Initial validation scope | Change-candidate approval and write-back contract validation |
| Automate | Follow-up scope | Event-condition-based automatic execution |

---

## 4. Core Components and Extension Components

A component may be called composable only if all three conditions hold. **If any one of them fails, it is part of the core, not an attachment.**

```text
1. The core is complete without it.
2. Its contract surface converges to a single point.
3. If it fails, execution can fall back to a core path.
```

| Component | 1. Core is complete | 2. Contract surface | 3. Fallback |
|---|---|---|---|
| Conversational Input | Buttons do the same work | **One state slot interface** | Manual input and button execution |
| Gate / WriteBack | Bulk import and direct edit exist | **One change candidate** | Direct edit path |
| Automate | A person can run it | **One run-start call** | Manual execution |

Contract design has a cost, so the current scope and the follow-up scope are separated.

| Component | Contract fixed now | Reason |
|---|---|---|
| Conversational Input | **Yes** | It sits directly on the slot structure |
| Gate / WriteBack | **Yes** | It comes between run state and persistent knowledge |
| Automate | **One line only** | Its only contact point is "start a run" |

Gate and WriteBack are extension components. Their contracts are prepared early, but the 2026 core implementation target remains the Workflow Manager and the Ontology Manager. Automation triggers can start workflows, but they must use the same workflow and state contracts as a button-driven run.

---

## 5. Workflow Manager

| Capability | Description |
|---|---|
| Workflow definition | Stores nodes and control edges |
| Role declaration | Declares which objects a process handles and their types |
| Slot and action binding | Connects the slots a node reads and writes with the action it calls |
| Static check | Catches mismatches between declarations and mappings before save or run |
| Run state | Keeps in-flight values in isolated state slots |
| History | Records slot changes, node entries, and external calls separately |
| **Dry-run** | **Checks parameters, mappings, and validation results without applying anything** |

**Values flow by name, not along edges.** An edge on the canvas means execution order only. Values are written to and read from named state slots, so a value used by three nodes does not add three connections.

Slot names are not free strings. They are derived from properties registered in the type registry and they also name the target role.

```text
subject.tension    With a single target, the role is omitted
source.tension     The role is shown only when two targets are compared
```

Actions consume and produce objects. A produced role is bound after the action succeeds, so a workflow that does not yet have its target object is a normal case rather than a special one.

```text
createAppointment
  required_object_roles   patient, doctor
  produced_object_roles   appointment
```

A dry-run executes to the end, fills slots, calls the validator, and evaluates promotion conditions, but it creates no change candidate and never calls the Apply boundary. Its output is a validation report: the executed path, unmet required slots, per-slot validation results, evidence presence, an action-call summary, whether promotion is possible, and a preview of the values that would have changed.

---

## 6. Ontology Manager

| Area | Content |
|---|---|
| Type registry | Object, property, shared property, relation, action, and workflow types |
| Persistent knowledge | Object instances, property values, relation instances |
| Change procedure | Proposal, impact analysis, approval, apply |
| Change history | What actually changed in definitions and persistent knowledge |
| Bulk import | Large definition and data sets from files or APIs |
| Validation | Type, range, enumeration, pattern, required, referential integrity, cardinality |

A schema table is not enough to claim that an ontology is working. **The type registry must actually be read at the points where decisions are made**: choosing a slot name, saving a node declaration, writing a slot value, importing in bulk, saving a direct edit, and evaluating promotion.

Persistent knowledge has four write paths.

```text
1. ABox bulk import   file, API, UI    -> validate -> apply
2. ABox direct edit   admin screen     -> permission check -> validate -> apply
3. TBox change        proposal -> impact analysis -> approval -> apply
4. Workflow promotion slot -> promotion check -> change candidate -> approval -> apply
```

Three conclusions follow:

- **All four paths use the same Unified Validator and the same Apply boundary.** If the criteria differ per path, the same value passes on one path and fails on another.
- **Any request that changes a definition goes through path 3.** This is enforced by the Apply boundary, not by convention. An ABox edit permission cannot change a definition.
- **Gate only touches path 4.** Because paths 1 to 3 exist, persistent knowledge is manageable without Gate.

There is one validator. Slot writes, dry-run, promotion checks, bulk import, direct edit, and impact analysis all call it. Only the failure policy differs per caller: reject, skip the item, or record it in a report. The result format is also single, so dry-run reports, import reports, and impact-analysis reports share one display component.

ABox and TBox are controlled with different strength.

| | ABox bulk import | TBox bulk change |
|---|---|---|
| Target | Instances, property values, relations | Definitions |
| Apply mode | Full or partial apply | **Full apply only** |
| Approval | **Not required** | **Required** |
| Route | Direct apply | **Proposal, impact analysis, approval** |

A TBox change alters the validation criteria themselves, so the validity of existing data depends on it. Partial apply would leave the criteria in an incomplete state, which is why TBox changes cannot be applied partially.

---

## 7. State and History

Two histories are kept separately.

| History | Meaning | Owner |
|---|---|---|
| Slot change history | How a slot value changed during a run | Workflow Manager |
| Ontology change history | What **actually** changed in a definition or a persistent instance | Ontology Manager |

A workflow computing a value does not by itself change a persistent instance.

| Record | Nature | Created |
|---|---|---|
| Change candidate | **A proposal** awaiting approval | When a workflow promotes a value |
| Change history | **A fact** that already happened | **Every time something is applied** |

A candidate can be rejected, so a candidate is not a history entry. Bulk import and direct edit write history without a candidate. The workflow path writes history when a candidate is applied.

---

## 8. Example Use Cases

Two examples show the same structure with different starting points.

| Example | What starts the run | Document |
|---|---|---|
| Hospital reservation | A person, through a button or form | [Hospital Reservation Example](../../examples/en/hospital-reservation.md) |
| Factory maintenance automation | A sensor event that meets the rule condition | [Factory Maintenance Automation Example](../../examples/en/factory-maintenance-auto.md) |

In the hospital example, an operator selects a patient, a department, and a doctor, checks available slots, and creates an appointment. The same run can be started by an utterance such as "book an orthopedics appointment tomorrow morning": the conversational component fills `patient.name`, `department.name`, and `appointment.requested_time`, asks about the missing doctor, and then hands the run to the same engine a button would have used. **The slots being filled are identical**, so if the conversation fails the user returns to the button screen and fills the same slots directly.

In the factory example, repeated stop events on a conveyor meet a rule condition, and the workflow classifies the symptom, selects a likely cause and a maintenance action, creates a work order, and notifies a technician. No approval step is inserted here.

Both examples share one shape.

```text
Input occurs
  -> Fill state slots
  -> Validate against ontology definitions
  -> Execute actions
  -> Apply business data changes through the Apply boundary
  -> Store history
```

Only the start differs. The hospital workflow starts from a person. The factory workflow starts from an event.

---

## 9. Scope for 2026

The core is built in ten steps.

| Step | Content |
|---|---|
| Core-1 | Basic type registry structure |
| Core-2 | Persistent knowledge store, Apply boundary, change history |
| Core-3 | **Unified Validator, minimum rule set** |
| Core-4 | Bulk import: job and item, ABox load, dry-run |
| Core-5 | TBox change proposal, impact analysis, approval |
| Core-6 | Workflow definitions, roles, nodes, control edges |
| Core-7 | Slot declarations, action bindings, static checks |
| Core-8 | Execution, role binding, state slots |
| Core-9 | Slot change history, action execution logs, evidence |
| Core-10 | Button/form execution, dry-run reports |

**The validator comes before bulk import.** Bulk import is the first caller that validates at scale, so accepting large data sets without the validator would change later validation results.

Extension components follow their own track: conversational input in three phases (intent recognition, follow-up questions, utterance evidence), Gate in three phases (change candidate, approval screen, WriteBack simulation), and automation in a follow-up scope.

The goal is not to reproduce a full large-scale data operations platform. It is narrower and more specific:

```text
A production-grade Ontology Manager
  Type registry, persistent knowledge, bulk import, change procedure, one validator

A production-grade Workflow Manager
  Definitions, bindings, run state, static checks, dry-run

Explicit button/form business automation

A conversational input module in front
A Gate / WriteBack extension behind
An automation extension as follow-up scope
```

---

## 10. Open Items

These items are intentionally left open. They map to the `D-N` items in the [Architecture Spec](02-architecture-spec.md) and are listed as open questions in [Open Questions](03-open-questions.md).

| # | Decision | Options | Spec | Timing |
|---|---|---|---|---|
| O-1 | Scope of shared properties | Global only / project extension allowed | D-2 | Core-1 |
| O-2 | Retention of imported source data | Reference only / keep a copy | D-5 | Core-4 |
| O-3 | Approval unit for TBox bulk changes | Whole job / per item | D-7 | Core-5 |
| O-4 | **Instance coverage of impact analysis** | Full / sample / asynchronous full | **D-8** | **Core-5** |
| O-5 | Retention of dry-run reports | Discard / keep per run / compare over time | D-14 | Core-10 |
| O-6 | Sourcing of the conversational module | In-house / external model / hybrid | - | Conv-P1 |
| O-7 | Ownership of the follow-up question policy | Fixed in declarations / domain override allowed | D-12 | Conv-P2 |
| O-8 | Scope of data source mapping | Manual registration only / pipeline integration | - | After Core-4 |

**O-4 has the widest impact.** A full check is accurate but slows approval on large data sets, while a sample check is fast but surfaces failures at apply time. The choice changes the state set of a change proposal and the screen flow, so it is decided before Core-5 design begins.

---

Three sentences summarize what this structure buys:

> **Every path to persistent knowledge goes through the same validation and the same Apply boundary.**
> **Every request that changes a definition goes through approval, and the Apply boundary enforces it.**
> **Conversation models, approval policies, and automation triggers can change without disturbing the core engine or the type registry.**
