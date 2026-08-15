# Open Questions

[한국어 정본](../ko/03-open-questions.md)

This is an English translation of the canonical Korean document.

This page collects questions where outside review is especially useful.

## 1. State Passing

Should workflow values move through explicit data edges, or should nodes read and write named state slots?

OntoFlow currently favors named state slots with control edges only.

## 2. Ontology Scope

How far should ontology definitions participate in workflow execution?

Current position:

- They define object roles and slot types.
- They validate values.
- They constrain persistent writes.
- They provide explainable references for decisions.

## 3. Conversational Input

Should natural-language input be part of the core workflow engine or a front component?

OntoFlow treats it as a front component that proposes workflow runs and slot values.

## 4. Gate Boundary

Should approval be core behavior or an extension component?

OntoFlow treats Gate as an extension. The core must work without it through imports and direct edits.

## 5. Terminology

Which terms are clearest for public documentation?

- TBox / ABox
- Type Layer / Instance Layer
- Persistent Knowledge
- Object Instance

Feedback on terminology is welcome.
