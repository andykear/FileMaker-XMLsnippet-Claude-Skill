---
name: filemaker-ai-vocabulary
description: Use this skill whenever Claude needs to name, call, or validate a FileMaker function or script step — writing a calculation, writing a script, checking whether a function or script step exists, checking parameter order or count, or checking whether a script step is supported on a given platform (Pro, Go, WebDirect, Server, Cloud, Data API, Custom Web Publishing). Trigger any time FileMaker calculation syntax or script step syntax is being written or reviewed. Do not name or call a FileMaker function or script step from memory alone — check the reference files first.
---

# FileMaker AI Vocabulary Skill

The authoritative, compressed vocabulary of every FileMaker Pro function and script step at FileMaker 26.1.1, with 26.0 as the floor. Consult it before naming or calling any function or step, so nothing invented, misspelled, or wrongly-signatured reaches the output.

Created by Andrew Kear of Clockwork Creative Technology. Licence: CC BY 4.0.

## Critical rules (enforced before generating)

1. **Only use functions and script steps that appear in the reference files.** Never invent one. If a plausible-sounding name is not listed, state that no such FileMaker function or step exists rather than guessing.
2. **Function parameter order is authoritative.** Braces `{ }` mark optional parameters; an ellipsis marks a repeatable group. Do not reorder, drop, or add parameters. Three functions carry a second signature line for an alternate call form: `Substitute` and `JSONSetElement` (variadic bracket-pair forms) and `ComputeModel` (a distinct image-model parameter list).
3. **Aggregate argument shapes.** `Sum`, `Count`, `Average`, `Max`, `Min`, `StDev`, `StDevP`, `Variance`, `VarianceP` show `{; field...}` but each field slot also accepts a repeating field, a related field (`table::field`), a semicolon-separated list of several fields, or cross-table match lists. The signature shows the canonical form; see the function file header.
4. **Target is FileMaker 26.1.1, with 26.0 as the floor.** Every entry exists from 26.0 unless tagged `[26.1+]`, and a platform value of `26.1+` means No before 26.1 and Yes from 26.1. Use a `[26.1+]` function or step only when every client and host that will run it is on 26.1 or later; if the version is unknown, assume 26.0.x and use an alternative. Versions before 26 are not tracked — supporting them is a separate check this skill does not make.
5. **The script step file is an index, not an option spec.** It carries each step's canonical name, internal ID, and platform support only. It deliberately does not list step options. For a step's options, element structure, or paste-ready XML, defer to the companion FileMaker Script XML and Field XML skills — do not reconstruct options from this file.
6. **⚠ marks a platform exception, not the norm.** No ⚠ means the step is fully supported (Yes) on Pro, Go, WebDirect, Server, Cloud, Data API, and Custom Web Publishing. Where ⚠ appears, only the listed platforms deviate (No = skipped, returns error 3; Partial = runs but differs); treat all unlisted platforms as Yes. Check these flags before using a step in a server-scheduled, WebDirect, or Go workflow.

## Reference files

Before producing any FileMaker function or script step in output, verify it against the relevant reference file. Each file carries its own header restating these rules.

- `references/clockwork_functions.md` — 375 functions (378 signature lines): `FunctionName ( params ) → return type`, with `[26.1+]` on functions new in 26.1.
- `references/clockwork_scriptsteps.md` — 218 script steps: `Step Name   #internalID`, with `[26.1+]` on steps new in 26.1 and `⚠` platform flags where support is not universal.

## What this skill does not cover

- Calculation or script paste-format XML — companion FileMaker Script XML Skill
- Field, table, or value list definition XML — companion FileMaker Field XML Skill
- Layout object XML — companion FileMaker Layout XML Skill
- Whether a referenced field, table, or JSON path exists in a given solution — this skill validates function and step syntax, not the logic built with them
