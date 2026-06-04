---
name: filemaker-xml
description: >
  Use this skill whenever the user wants to work with FileMaker script XML or custom function XML.
  This includes: generating FileMaker XML from a description, pseudocode, or
  script steps; analysing or reviewing existing FileMaker XML for errors or
  silent-failure risks; converting scripts between formats; or any task where
  the output must be paste-ready XML for the FileMaker Script Workspace or Manage Custom Functions dialog.
  Trigger any time the user mentions FileMaker scripts, fmxmlsnippet, Set
  Variable XML, script step XML, custom function XML, or asks to produce XML that can be pasted
  into FileMaker. Always use this skill — do not attempt FileMaker XML tasks
  from memory alone, as the format has undocumented paste-handler rules that
  cause silent failures without this reference.
---

# FileMaker Script & Custom Function XML Skill

## Overview

This skill enables Claude to correctly **analyse** and **generate** FileMaker
Script Workspace XML — the `fmxmlsnippet type="FMObjectList"` format that
FileMaker accepts via clipboard paste.

The format has a lenient XML parser but a strict paste handler. Structural
deviations that look fine as XML can cause steps to paste with silently missing
data (e.g. Set Variable pastes with no variable name). This skill's reference
document is the authoritative reverse-engineered spec for what the paste
handler actually accepts.

---

## Mandatory first step

**Before writing or reviewing any FileMaker XML, read the full rules file:**

```
filemaker_xml_rules.md
```

This file (v1.10.4, ~3,000 lines) is the complete specification. It contains:
- Paste format requirements (§1–2)
- Set Variable canonical structure and the `<Name>` tag rendering trap (§3, §7)
- Common conventions: `<Name>` tag, schema references, comparison operators (§5)
- Disabled step behaviour (§4)
- Canonical skeletons for all ~180 script steps (§8)
- Known silent-failure modes (§7)
- Appendices with round-trip validation cases

Do not rely on training data for FileMaker XML structure — always consult the
reference file. The format contains non-obvious quirks (e.g. a trailing space
in the `Configure RAG Account ` step name) that only appear in the spec.

---

## Task: Analysing existing XML

When the user provides FileMaker XML to review:

1. Read `filemaker_xml_rules.md` first.
2. Check against the three known silent-failure modes (§7):
   - **7.1** — `<Name>` tag emitted as single-letter `<n>` (Set Variable loses variable name)
   - **7.2** — compact/single-line form instead of expanded child form
   - **7.3** — wrong element order within a `<Step>`
3. Check wrapper: must be `<?xml version="1.0" encoding="UTF-8"?>` + `<fmxmlsnippet type="FMObjectList">`.
4. Check indentation: two spaces, no tabs.
5. Check CDATA: all calculation content must be in `<Calculation><![CDATA[...]]></Calculation>`.
6. For each step, verify the canonical skeleton matches the spec for that step ID/name.
7. Report issues clearly, quoting the offending XML fragment and explaining the paste-time consequence.
8. Offer a corrected version.

---

## Task: Generating XML

When the user wants to generate FileMaker XML from a description or pseudocode:

1. Read `references/filemaker_xml_rules.md` first.
2. Identify each script step needed and look up its canonical skeleton in §8.
3. Apply §1 paste format requirements throughout:
   - Wrap in `<?xml version="1.0" encoding="UTF-8"?>\n<fmxmlsnippet type="FMObjectList">...\n</fmxmlsnippet>`
   - Two-space indent, no tabs
   - CDATA for all calculations
   - Expanded (not compact) child forms
   - Correct element order per spec
4. For schema references (table names, field names, layout names, script names):
   - If the user has provided a DDR or explicit IDs, use those.
   - Otherwise use the **placeholder-ID pattern**: `id="1"` everywhere, with `name` attributes matching the user's names. FileMaker will resolve real IDs on paste. (§5)
5. Emit the `<Name>` tag for Set Variable and any other step that requires it using the correct four-letter form `Name` (capital N, lowercase a-m-e) as documented in §3 and §5. Be aware of the rendering-trap described in §7.4 — the tag must be the full word, not a single letter.
6. Use ASCII comparison operators (`<>`, `<=`, `>=`) rather than Unicode equivalents — safer for transport (§5).
7. Output the complete XML inside a code block with `xml` syntax highlighting.
8. After the XML, briefly note any assumptions made (e.g. placeholder IDs used, repetition defaulted to 1).

---

## Key conventions (quick reference — always verify against full spec)

| Convention | Rule |
|---|---|
| Wrapper | `fmxmlsnippet type="FMObjectList"` |
| Indent | 2 spaces, never tabs |
| Calculations | `<Calculation><![CDATA[expr]]></Calculation>` |
| Variable name tag | `<Name>$varName</Name>` (four letters, capital N) |
| Schema IDs without DDR | `id="1"` + matching `name` attribute (placeholder-ID pattern) |
| Comparison operators | ASCII: `<>` not `≠`, `<=` not `≤`, `>=` not `≥` |
| Disabled steps | `enable="False"` on the `<Step>` element |
| Comment divider (blank) | Self-closing `<Step enable="True" id="89" name="Comment"/>` |

---

## Reference file table of contents

The rules file is large. Key sections for quick navigation:

- **§1–2** — Paste format requirements and why exact matching matters  
- **§3** — Set Variable canonical structure + `<Name>` tag warning  
- **§4** — Disabled flow-control steps  
- **§5** — Common conventions (`<Name>` tag, schema refs, comparison operators)  
- **§6** — Step index (all ~180 steps with IDs)  
- **§7** — Silent failure modes (the three known traps)  
- **§8.1** — Flow control (If, Loop, Perform Script, Set Variable, Comment, etc.)  
- **§8.2–8.14** — All other step categories  
- **§9** — Third-party steps (MBS plugin)  
- **Appendix A** — Configured-form reference skeletons  
- **Appendix B** — Known name quirks (trailing spaces, etc.)  
- **Appendix C** — Round-trip validation cases  
- **§11** — Custom functions (skeleton, attributes, parameters, recursion)  
