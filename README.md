# FileMaker Script XML Skill for Claude

A Claude skill that enables reliable generation and analysis of FileMaker Script Workspace XML (`fmxmlsnippet type="FMObjectList"`).

Developed by [Clockwork Creative Technology](https://www.clockworkct.co.uk) and shared openly with the FileMaker/Claris community.

---

## The problem this solves

FileMaker's Script Workspace accepts scripts via clipboard paste in a specific XML format. Without explicit knowledge of that format, AI models fabricate step IDs — and FileMaker pastes the wrong steps silently with no warning.

This skill gives Claude a fully verified map of every script step ID, canonical XML skeletons for all ~180 steps, and the hidden paste-handler rules that cause silent failures when violated.

---

## What's in the box

```
SKILL.md                          — Claude skill definition
references/
  filemaker_xml_rules.md          — Full specification (v1.10.3, ~2,900 lines)
```

The specification was built by reverse-engineering FileMaker's native Copy format through extensive round-trip testing: generate XML → paste into Script Workspace → save → copy back out → diff. Iterated until generated and native output matched structurally across tens of thousands of lines of production scripts.

---

## Requirements

- [Claude](https://claude.ai) (Pro, Team, or Enterprise)
- Skills support enabled in your Claude organisation

## Installation

1. Download the zip from the [Releases](../../releases) page
2. Extract — you should have `SKILL.md` and `references/filemaker_xml_rules.md`
3. Upload to your Claude organisation's skills library, preserving the folder structure

## Usage

Once the skill is installed, Claude will automatically apply it when you ask for FileMaker scripts or fmxmlsnippet XML. No special prompt needed.

**To generate a script:**
> "Write a FileMaker script that loops through the found set and sets a flag field"

**To check existing XML:**
> Paste your fmxmlsnippet and ask Claude to review it for paste-handler errors

**With a DDR:**
> Attach your DDR and Claude will use real field, layout, and script names from your solution

---

## Pasting into FileMaker

The Script Workspace requires a specific clipboard format — not plain text. An approaches that work:

- **MBS Plugin**

---

## Specification highlights

- All ~180 step IDs verified against native FileMaker 2025 export
- Canonical skeletons for every step
- Set Variable silent-drop trap documented and solved
- Configured-form examples for the most commonly used steps
- MBS plugin step structure (§9)
- Known quirks in FileMaker's own output preserved verbatim (misspelled elements, trailing spaces)

---

## Licence

[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) — free to use, share, and adapt with attribution.

---

## Contributing

Found a step that doesn't round-trip? Native export that contradicts the spec? Open an issue or PR. The spec improves through community round-trip testing — that's how it was built.

---

## Version history

| Version | Notes |
|---|---|
| 1.10.3 | Removed changelog, validation suite, and step index appendices to reduce token load. Public release. |
| 1.10.2 | Removed pre-release version history narrative |
| 1.10 | First complete version. All AI steps (212–228) added. Placeholder-ID pattern documented. |
