# FileMaker Script XML Specification

Reverse-engineered specification of FileMaker's undocumented `fmxmlsnippet` clipboard format. Every step ID, element ordering rule, and silent failure mode, established through empirical round-trip testing against FileMaker Pro 2025 and 2026.

Developed by Andrew Kear of [Clockwork Creative Technology](https://www.clockworkct.co.uk) and shared openly with the FileMaker/Claris community.

---

## Why this exists

FileMaker's Script Workspace accepts pasted scripts in a specific XML format. Claris has never published a specification for it. Get the XML wrong and FileMaker accepts it silently, dropping data with no error and no warning. A Set Variable step pastes without its variable name. An Install OnTimer Script binds its calculation to the wrong slot. The step looks normal in the Script Workspace. The bug surfaces at runtime, if you notice it at all.

This specification documents what the paste handler actually accepts: 220 step IDs, canonical XML skeletons for every step, the element ordering constraints that cause silent drops when violated, and the configured form structures for the most commonly scripted steps. It was built entirely through round-trip testing (generate XML, paste, copy back out, diff against native output) and validated against tens of thousands of lines of production scripts.

---

## What it catches

The specification documents several classes of silent failure that are invisible in the Script Workspace after paste:

**Set Variable name drop.** If the variable name element is emitted as a single-letter tag instead of the canonical four-letter `Name` form, FileMaker pastes the step with no variable name. No error. The step looks configured. The variable is never set.

**Calculation slot misbinding.** Install OnTimer Script, Perform JavaScript in Web Viewer, and other steps with multiple calculation slots silently bind a calculation to the wrong internal slot when the wrapping element is omitted. The step pastes and runs, but not as intended.

**Element ordering.** Several steps require child elements in a specific order. The XML is valid either way. Only one order produces the correct paste result.

These are not edge cases. They are the default failure mode when any tool, human or AI, generates FileMaker script XML without knowing the paste handler rules.

---

## Packaged as a Claude Skill

The specification is packaged as a [Claude](https://claude.ai) skill with progressive loading: the model reads core rules plus only the step categories each task needs. This keeps token usage proportional to task complexity.

Once installed, Claude uses the specification automatically when generating or reviewing FileMaker script XML. The structural rules are handled deterministically. The model handles the logic.

**Tested with Claude. Model agnostic by design.** The deterministic approach means any capable model with the specification in context should produce reliable output. Claude is the only model Clockwork has verified against production scripts.

## Beyond Claude

The specification is packaged here as a Claude skill, but the format rules are model-agnostic. The reference files are plain markdown, readable by any LLM or human. If you're building tools that generate fmxmlsnippet XML, the paste handler rules documented here apply regardless of your stack.

---

## FM 26 support (v1.12)

v1.12 adds full FileMaker Pro 2026 support with 10 new steps and updates to existing steps, all round-trip verified:

**New steps:** PDF Files category (Create PDF, Open PDF, Append PDF, Print PDF, Close PDF, Cancel PDF), Configure Persistent Data, Insert Image Caption, Insert Image Captions in Found Set, Flush Web Viewer Cookies.

**Updated steps:** Save Records as PDF (three PDFSaveType modes, source and appearance enumerations), Show Custom Dialog (size and position calculations), Configure AI Account (all five provider variants including Custom endpoint), Set Zoom Level (Custom calculation), Save a Copy as XML (catalog selection and JSON options), Replace Field Contents (auto-enter options), Constrain Found Set (Find without indexes).

**New in FM 26:** DisableStepCollapsed universal element documented. Error codes 605-608 and 829-833. 16 Open menu steps added to the routing index.

---

## What's in the box

```
SKILL.md                            — Claude skill definition + routing index
references/
  core.md                           — Paste rules, conventions, Set Variable,
                                      silent failures, appendices (§1-7, A, B, C)
  steps-control.md                  — §8.1 Control
  steps-navigation-editing.md       — §8.3-8.4
  steps-fields-records.md           — §8.5-8.7
  steps-windows-files.md            — §8.8-8.9
  steps-accounts-ai-misc.md         — §8.2, §8.10-8.14
  steps-pdf.md                      — §8.15 PDF Files (FM 26)
  steps-plugin.md                   — §9 Plugin steps (MBS)
  custom-functions.md               — §11 Custom Functions
  worked-example.md                 — §10 Worked example (optional)
```

---

## Installation

1. Download the zip from the [Releases](../../releases) page
2. Extract to get `SKILL.md` and the `references/` folder
3. Upload to your Claude organisation's skills library, preserving the folder structure

Upgrading from v1.11: replace all files. The `references/steps-pdf.md` file is new; all other files are updated in place.

Upgrading from v1.10.x: remove the old single `references/filemaker_xml_rules.md` and replace with the current file set.

---

## Usage

Once installed, Claude applies the specification automatically when you ask for FileMaker scripts or fmxmlsnippet XML. No special prompt needed.

**Generate a script:**
> "Write a FileMaker script that loops through the found set and sets a flag field"

**Review existing XML:**
> Paste your fmxmlsnippet and ask Claude to review it for paste handler errors

**With schema context:**
> Attach your DDR and Claude will use real field, layout, and script names from your solution. You can also attach a DDR exported using [Clockwork Inspector](https://github.com/andykear/FileMaker-XML-inspector-open-source) for complete schema context.

---

## Pasting into FileMaker

The Script Workspace accepts `fmxmlsnippet type="FMObjectList"` via clipboard paste in FileMaker's internal clipboard format, not plain text. This skill has been tested with the **MBS Plugin** installed. Plugin-free clipboard conversion options are available in the FileMaker community and should work with this format, but have not been tested by Clockwork.

---

## Companion projects

This skill covers Script objects. Companion skills cover the other paste targets:

[FileMaker Layout XML Skill](https://github.com/andykear/FileMaker-XMLsnippet-Layout-Claude-Skill) — layout objects for Layout mode

[FileMaker Field Definitions XML Skill](https://github.com/andykear/FileMaker-XML-field-definitions) — field definitions for Manage Database

[FileMaker XML Inspector](https://github.com/andykear/FileMaker-XML-inspector-open-source) — browser-based Save as XML analyser. Single file, nothing uploaded, nothing leaves the browser.

[FileMaker XML Scrubber](https://github.com/andykear/FileMaker-XML-scrubber) — redacts credentials from FileMaker XML exports before sharing with AI tools.

---

## Licence

[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) — free to use, share, and adapt with attribution.

---

## Contributing

Found a step that doesn't round-trip? Native export that contradicts the spec? Open an issue or PR. The specification improves through community round-trip testing. That's how it was built.

---

## Version history

| Version | Notes |
|---------|-------|
| 1.12 | FM 26 (FileMaker Pro 2026) support. 10 new steps: PDF Files category (Create PDF, Open PDF, Append PDF, Print PDF, Close PDF, Cancel PDF), Configure Persistent Data, Insert Image Caption, Insert Image Captions in Found Set, Flush Web Viewer Cookies. Configured forms round-trip verified for all PDF steps, Configure Persistent Data (set and delete), Configure AI Account (all five providers), Show Custom Dialog (size/position), Set Zoom Level (Custom calculation), Replace Field Contents (auto-enter options), Constrain Found Set (Find without indexes). Save Records as PDF fully mapped: PDFSaveType (File, Target, Append), source, and appearance enumerations. Updated Save a Copy as XML (catalog selection). DisableStepCollapsed universal element documented. 16 Open menu steps added. New error codes 605-608, 829-833. |
| 1.11 | Progressive loading restructure: spec split into core rules plus on-demand step category files with a routing index. Simple tasks load around 55% fewer reference tokens than v1.10, typical tasks around 40% fewer, worst case parity. No specification content removed; all skeletons preserved verbatim. No-option steps consolidated into tables. |
| 1.10.4 | Added support for custom functions. Fixes an installation path issue affecting all previous versions. |
| 1.10.3 | Removed changelog, validation suite, and step index appendices to reduce token load. Public release. |
| 1.10.2 | Removed pre-release version history narrative. |
| 1.10 | First complete version. All AI steps (212-228) added. Placeholder-ID pattern documented. |
