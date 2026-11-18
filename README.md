# FileMaker AI Vocabulary

[![License](https://img.shields.io/badge/license-CC%20BY%204.0-green)](https://creativecommons.org/licenses/by/4.0/)

**Every FileMaker 26.1 function and script step, in about 7,000 tokens.**

Small enough to load into context as part of the skill, so the model has the whole vocabulary in front of it while it writes, instead of guessing from training data and inventing functions that don't exist. The raw Claris help pages for the functions alone run to roughly 122,000 tokens. This is the same knowledge at roughly 1/30th the token footprint.

Developed by Andrew Kear of Clockwork Creative Technology and shared openly with the FileMaker/Claris community.

## The problem it solves

FileMaker's namespace is large and full of near-identical names: `Right`, `RightWords`, `RightValues`; `GetField`, `GetFieldName`. An AI generating a calculation guesses from training data and confidently invents a function that sounds right, or transposes a parameter, or assumes the wrong return type. The fix is to hand it the real vocabulary as ground truth, but a reference only helps if it's loaded at the moment of generation, not fetched afterward. Most references are too big to keep resident. This one isn't. That's the whole idea.

## What's inside

Two flat files, one line per entry, no prose to wade through:

- **375 functions** — exact name, parameter order, optional-parameter shape, return type.
- **218 script steps** — exact name, internal ID, and platform support across all seven FileMaker clients and hosts.

```
Case ( test1 ; result1 {; test2 ; result2 ; ... ; defaultResult} ) → text, number, date, time, timestamp, container
Substitute ( text ; [search1 ; replace1] {; [search2 ; replace2] ; ...} ) → text
Set Variable   #141
Set Variable by Name   #250   [26.1+]
Perform Script on Server with Callback   #210   ⚠ Server:No, Cloud:No, DataAPI:No, CWP:No
```

Return types are carried because models infer them wrongly when chaining calculations, not just function names. It's a constrained vocabulary, not a manual: enough to verify a name, a signature, and where a step runs, and nothing to slow the model down getting there.

## Deliberately just the vocabulary

This file answers the cheap, always-loaded questions: does this exist, what's it called, what are its parameters, where does it run. It does **not** carry a step's full option structure or paste-ready XML. That depth lives in the companion [Script XML Skill](https://github.com/andykear/FileMaker-XMLsnippet-Claude-Skill), and keeping it there is the point: the vocabulary layer stays tiny enough to stay resident, the generation layer holds the detail. Two layers, on purpose.

The step IDs here are pinned to that skill, so an ID in this file is the same ID it emits. They cannot drift.

## Target: FileMaker 26.1.1

FileMaker 26.0 is the floor: nothing older is tracked, which is part of what keeps the files this small. But 26.1 added functions and steps that a 26.0.x client or server doesn't have, and live solutions will be running 26.0.x for a while yet. So everything new in 26.1 carries a `[26.1+]` tag, and the skill tells the model not to use it unless every client and host that will run it is on 26.1 or later. Seven entries carry the tag; everything else works on any 26.

## Use it

Load the release zip into Claude's skills (keep the folder structure; enable code execution and file creation). With the skill enabled, Claude consults the vocabulary when writing or reviewing FileMaker calculations and scripts, no special prompt. Setup varies by platform, so follow Anthropic's current instructions for adding a skill.

Model-agnostic by design: the files are plain markdown, readable by any LLM or human. Building your own FileMaker tooling? They're yours to integrate.

```
"Write a calc that strips the domain off an email address"
"Write a server-scheduled script to find overdue invoices"   → steps checked against platform support first
"Does FileMaker have a JSONMerge function?"                   → answered no, pointed to the real one
```

## The rest of the collection

**[Menu](https://github.com/andykear)**

**Reference skills**

**[FileMaker Second Opinion](https://github.com/andykear/FileMaker-second-opinion)**\
**[FileMaker AI Vocabulary](https://github.com/andykear/FileMaker-AI-vocabulary)**

**Research / Specialist**

**[FileMaker XML bit-flags](https://github.com/andykear/FileMaker-XML-bit-flags)** (SaXML)\
**[FileMaker AI Grammar](https://github.com/andykear/FileMaker-AI-grammar)**

**Generation, paste-ready FileMaker XML**

**[Script XML Skill](https://github.com/andykear/FileMaker-XMLsnippet-Claude-Skill)** (XMSS, XMSC, XMFN)\
**[Layout XML Skill](https://github.com/andykear/FileMaker-XMLsnippet-Layout-Claude-Skill)** (XML2)\
**[Field, Table & Value List Definitions](https://github.com/andykear/FileMaker-XML-field-definitions)** (XMFD, XMTB, XMVL)

**Analyse a FileMaker solution in your browser**

**[Clockwork Inspector](https://github.com/andykear/FileMaker-XML-inspector-open-source)** (SaXML)\
**[XML Scrubber](https://github.com/andykear/FileMaker-XML-scrubber)** (SaXML + others)

## Provenance

Function signatures, return types, and platform support come from Claris FileMaker Pro Help — Claris's published facts, compression the only transformation. The script steps and their verified internal IDs are drawn from the companion Clockwork Script XML Skill — Claris doesn't publish the IDs, they're original reverse-engineering work from that repo.

Three exceptions, where Claris's help doesn't cover something yet:

- **Five JSON array functions** (JSONAnyValuesExist, JSONRemoveValues, JSONUniqueValues, JSONValueCount, JSONValueExists) aren't in Claris's help. Their signatures are the engine's own function declarations, identical in 26.0.3 and 26.1.1, and their return types were measured in FileMaker Pro.
- **The 26.1 additions** come from the FileMaker 26.1.1 release notes, checked against the engine's declarations and run in a 26.1.1 engine.
- **Platform support for Set Variable by Name and Replace Field Contents by Name** isn't published yet. Their flags are copied from Set Variable and Replace Field Contents until Claris publishes them.

## Licence & contributing

CC BY 4.0 — free to use, share, and adapt with attribution. Found something missing, misnamed, or wrong on a return type or compatibility flag? Open an issue or PR.

## Version history

| Version | Notes |
|---|---|
| 1.0 | FileMaker 26 baseline. 368 functions, 216 script steps with platform support, each carrying its verified internal ID from the Script XML skill. ~7,000 tokens total. |
| 1.1 | FileMaker 26.1.1. Adds `Get ( SortOrder )`, `GetToolDefinitions`, Set Variable by Name (#250) and Replace Field Contents by Name (#249), all tagged `[26.1+]`. Save a Copy as Add-on Package runs on Server from 26.1. Set Field By Name is spelled Set Field by Name from 26.1. Three step names corrected to FileMaker's own capitalisation: Perform Script on Server, Perform Script on Server with Callback, Open Upload to Host. Also adds five JSON array functions present since 26.0 but missing from 1.0: JSONAnyValuesExist, JSONRemoveValues, JSONUniqueValues, JSONValueCount, JSONValueExists. 375 functions, 218 script steps. |

---

*Clockwork Creative Technology — clockworkct.co.uk · github.com/andykear. Bespoke FileMaker development, automated artwork systems, and hosted solutions. Working on something and need a hand? Get in touch.*
