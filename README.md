# FileMaker Script XML Skill for Claude

A Claude skill that gives AI models a deterministic, empirically verified foundation for generating and analysing FileMaker Script Workspace XML (`fmxmlsnippet type="FMObjectList"`).

Developed by [Clockwork Creative Technology](https://www.clockworkct.co.uk) and shared openly with the FileMaker/Claris community.

---

## The problem this solves

FileMaker's Script Workspace accepts scripts via clipboard paste in a specific XML format. Without explicit knowledge of that format, AI models guess, and FileMaker pastes the wrong steps silently with no warning.

Effective AI to FileMaker workflows require a clear boundary between what AI should determine (the logic) and what must be deterministic (the XML structure). This skill provides the deterministic layer: a fully verified map of every script step ID, canonical XML skeletons for all ~190 steps, and the hidden paste handler rules that cause silent failures when violated.

The XML shape is knowable. This spec makes it known.

---

## Keeping AI focused on what it is good at

AI models are generative by nature. They predict, they infer, they improvise. That is exactly what you want when reasoning about business logic. It is exactly what you do not want when the XML element order determines whether FileMaker silently drops your script steps.

This skill keeps AI focused on what it is good at. The structure is handled deterministically. Claude handles the logic.

---

## New in v1.11: progressive loading

v1.11 restructures the specification so the model reads the core rules plus only the step categories a task needs, instead of the full spec every time. A routing index in the skill definition maps every step to its reference file.

Measured against v1.10 on the same content: simple scripts load around 55% fewer reference tokens, typical scripts around 40% fewer, and a worst case script touching every category still loads slightly less than v1.10. Token figures are estimates and vary by model tokenizer; the durable property is the mechanism: the skill loads only what the task needs, at any spec size.

Nothing was removed. Every canonical skeleton, configured variant, and silent failure mode from v1.10.4 is preserved verbatim, verified programmatically during the restructure. Steps with no options are consolidated into per category tables that state the identical canonical form once.

---

## How the specification was built

This is not a prompt or a set of guidelines assembled from documentation. FileMaker publishes no formal specification for the fmxmlsnippet clipboard format.

The specification was built entirely through empirical reverse engineering: generate XML → paste into Script Workspace → save → copy back out → diff against native output. Every step ID, every attribute, every element ordering constraint was established through round trip testing and validated against tens of thousands of lines of production scripts. Silent failure modes, where FileMaker accepts malformed XML and drops elements without any error, were systematically identified and documented.

The result is a formal specification for a format that Claris has never documented.

---

## What's in the box

```
SKILL.md                            — Claude skill definition + routing index
references/
  core.md                           — Paste rules, conventions, Set Variable,
                                      silent failures, appendices (§1–7, A, B)
  steps-control.md                  — §8.1 Control
  steps-navigation-editing.md       — §8.3–8.4
  steps-fields-records.md           — §8.5–8.7
  steps-windows-files.md            — §8.8–8.9
  steps-accounts-ai-misc.md         — §8.2, §8.10–8.14
  steps-plugin.md                   — §9 Plugin steps (MBS)
  custom-functions.md               — §11 Custom Functions
  worked-example.md                 — §10 Worked example (optional)
```

Section numbering is unchanged from earlier versions, so existing cross references remain valid.

---

## Requirements

- [Claude](https://claude.ai) (Pro, Team, or Enterprise)
- Skills support enabled in your Claude organisation

**Tested with Claude. Model agnostic by design.** The deterministic approach means any capable model with the specification in context should produce reliable output. Claude is the only model Clockwork has verified against production scripts.

---

## Installation

1. Download the zip from the [Releases](../../releases) page
2. Extract — you should have `SKILL.md` and the `references/` folder
3. Upload to your Claude organisation's skills library, preserving the folder structure

Upgrading from v1.10.x: remove the old single `references/filemaker_xml_rules.md` and replace with the v1.11 file set. The content is the same specification; only the structure changed.

---

## Usage

Once the skill is installed, Claude will automatically apply it when you ask for FileMaker scripts or fmxmlsnippet XML. No special prompt needed.

**To generate a script:**
> "Write a FileMaker script that loops through the found set and sets a flag field"

**To check existing XML:**
> Paste your fmxmlsnippet and ask Claude to review it for paste handler errors

**With a DDR:**
> Attach your DDR and Claude will use real field, layout, and script names from your solution. You can also attach a DDR exported using [Clockwork Inspector](https://github.com/andykear/FileMaker-XML-inspector-open-source) to give Claude complete schema context.

---

## Pasting into FileMaker

Layout mode requires the `fmxmlsnippet type="LayoutObjectList"` format on the clipboard in FileMaker's internal clipboard format, not plain text. This skill has been tested with the **MBS Plugin** installed. Plugin free clipboard conversion options are available in the FileMaker community and should work with this format, but have not been tested by Clockwork.

---

## Specification highlights

- All ~190 step IDs verified against native FileMaker 2025 export
- Canonical XML skeletons for every step
- Set Variable silent drop trap documented and solved
- Configured form examples for the most commonly used steps
- MBS plugin step structure (§9)
- Custom function definitions (§11)
- Known quirks in FileMaker's own output preserved verbatim (misspelled elements, trailing spaces)

---

## Licence

[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) — free to use, share, and adapt with attribution.

---

## Contributing

Found a step that doesn't round trip? Native export that contradicts the spec? Open an issue or PR. The spec improves through community round trip testing. That's how it was built.

---

## Version history

| Version | Notes |
|---------|-------|
| 1.11 | Progressive loading restructure: spec split into core rules plus on demand step category files with a routing index. Simple tasks load around 55% fewer reference tokens than v1.10, typical tasks around 40% fewer, worst case parity. No specification content removed; all skeletons preserved verbatim. No option steps consolidated into tables. |
| 1.10.4 | Added support for custom functions. Fixes an installation path issue affecting all previous versions |
| 1.10.3 | Removed changelog, validation suite, and step index appendices to reduce token load. Public release. |
| 1.10.2 | Removed pre-release version history narrative |
| 1.10 | First complete version. All AI steps (212–228) added. Placeholder-ID pattern documented. |
