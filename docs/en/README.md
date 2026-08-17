# OntoFlow English Documentation

[한국어](../ko/README.md) | English

This folder contains English translations or summaries. The Korean documentation is the primary reference.

## Reading Order

| Order | Document | Description |
|---:|---|---|
| 1 | [Overview](01-overview.md) | The problem OntoFlow addresses and the overall architecture |
| 2 | [Architecture Spec](02-architecture-spec.md) | Core contracts for types, instances, workflows, validation, and history |
| 3 | [Open Questions](03-open-questions.md) | Design topics that are not finalized yet |
| 4 | [Roadmap](04-roadmap.md) | Implementation priorities and future extensions |

Some detailed component documents are currently available only in Korean.

## Examples

| Example | Description |
|---|---|
| [Hospital Reservation Example](../../examples/en/hospital-reservation.md) | A human-started button/form workflow |
| [Factory Maintenance Automation Example](../../examples/en/factory-maintenance-auto.md) | An event-started automation workflow |

## Language Structure

```text
docs/ko  Korean reference documentation
docs/en  English translations or summaries
```

GitHub directory pages do not provide a built-in website-style language switcher. Each language folder uses a `README.md` index to provide navigation between Korean and English documents.

## Terminology

See the root [Glossary](../../GLOSSARY.en.md) first.

```text
TBox = class/type layer
ABox = instance/data layer
ObjectType = class/type of business object
ObjectInstance = actual business object instance
ActionType = executable business operation definition
```
