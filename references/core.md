# Canonical XML Format for FileMaker Script Steps

Part of the FileMaker Script XML Skill, v1.12.
Verified against FileMaker Pro 2026 on macOS.

**Author:** Andrew Kear, Clockwork Creative Technology
**Version:** 1.11
**Date:** June 2026
**Verified against:** FileMaker Pro 2025 on macOS
**Licence:** [Creative Commons Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/)
— free to use, share, and adapt with attribution.

---

## Why this exists

FileMaker's Script Workspace pastes scripts from the clipboard in an
undocumented XML format. The XML parser is lenient but the paste
handler is not: structurally valid XML can paste with elements
silently dropped or rebound to the wrong slot (the classic case is a
Set Variable step pasting with no variable name). This specification
is a reverse-engineered reference for what the paste handler actually
accepts, built entirely by round-trip testing: generate, paste, save,
copy back out, diff against native output. Where FileMaker's native
emission contains quirks (typos, trailing spaces, cryptic numeric
attributes), they are preserved exactly, because deviation from the
native form is precisely what triggers the silent failures.

## How this specification is organised

From v1.11 the specification is split for progressive loading: this
file holds the rules that apply to every task, and step skeletons live
in category files read on demand. The routing index in SKILL.md maps
every step to its file. Section numbers are unchanged from earlier
versions, so cross references such as "see Section 8.5" mean the file
holding that section.

Step skeletons in the category files show FileMaker's native minimal
output; configured variants appear inline and all have been verified
by round-trip testing. Every generated snippet must be wrapped in the
XML declaration and `fmxmlsnippet` element specified in Section 1.

---

## Scope

**This document covers:** the structural XML format that
`fmxmlsnippet type="FMObjectList"` clipboards must follow to paste
cleanly into the FileMaker Script Workspace. It documents
canonical skeletons for every script step (~180), verified
configured forms for the steps most commonly used in real
automation, and three known silent-failure modes.

**This document does not cover:** layout objects, theme XML,
FileMaker's full DDR (Database Design Report) format, runtime
behaviour of any step, or what an LLM should *do* with this
information beyond following it as a structural specification.

**Verified environment:** FileMaker Pro 2025 on macOS. The same
format is expected to work in FileMaker 19 onward and on Windows,
but cross-version and cross-platform testing has not been performed.
Reports of differences are welcome.

---
## 1. Paste format requirements

Output must match FileMaker's native Copy format exactly. The following
requirements are non-negotiable.

1. **Wrapper.** Wrap all output in an XML declaration followed by
   `<fmxmlsnippet type="FMObjectList">`.

   ```
   <?xml version="1.0" encoding="UTF-8"?>
   <fmxmlsnippet type="FMObjectList">
     ...steps...
   </fmxmlsnippet>
   ```

2. **Indentation.** Two-space indent throughout. Never tabs.

3. **Element order.** Child elements within each `<Step>` must appear
   in the order FileMaker emits them. Reordering is not safe.

4. **Calculations.** All calculation content is wrapped in CDATA
   inside a `<Calculation>` element:
   `<Calculation><![CDATA[expression]]></Calculation>`.

5. **Tag names.** Emit canonical tag names in full. Do not abbreviate
   or collapse.

6. **Expanded forms.** Elements with child `<Calculation>` nodes
   (`<Value>`, `<Repetition>`, `<Name>`, `<Message>`, `<Height>`,
   `<Width>`, etc.) must appear in expanded form, with the child
   `<Calculation>` on its own indented line. Compact single-line forms
   are not safe.

## 2. Why exact matching matters

The FileMaker XML parser is lenient: any well-formed XML parses. The
Script Workspace paste handler, however, is whitespace- and
format-sensitive for several step types. When output drifts from
FileMaker's native format, individual child elements can be silently
dropped during paste — the step pastes without error, but with missing
data.

Set Variable is the most sensitive case. Non-native formatting causes
the `<Name>` element to be dropped, producing a Set Variable step with
no variable name assigned. Round-trip testing confirms that matching
FileMaker's native indentation and expanded child form preserves the
`<Name>` element reliably; deviations do not.

Generators must therefore emit FileMaker's exact native format rather
than any compact or normalised equivalent.

## 3. Set Variable: canonical structure

> **Rendering-trap warning (read first).** The variable-name element
> in Set Variable is the literal four-letter tag spelled `N-a-m-e` —
> capital N followed by lowercase a, m, e — written in source as
> `<Name>...</Name>`. The four ASCII bytes are `0x4E 0x61 0x6D 0x65`.
>
> If your rendering of this document collapses the tag to a single
> letter (so that the canonical form below appears as `<n>...</n>`
> rather than `<Name>...</Name>`), your rendering layer is
> stripping inner content from anything resembling an HTML element
> with this spelling. Trust the byte-level form from a plaintext
> source, not the rendered display. Emitting the single-letter form
> is the silent-failure trigger described in Section 7.1 — every Set
> Variable step pastes with no variable name. See Section 7.4 for the
> full mechanics of this rendering-corruption failure mode.

```
  <Step enable="True" id="141" name="Set Variable">
    <Value>
      <Calculation><![CDATA[expression]]></Calculation>
    </Value>
    <Repetition>
      <Calculation><![CDATA[1]]></Calculation>
    </Repetition>
    <Name>$variable_name</Name>
  </Step>
```

Requirements:

- Two-space indent from `<Step>` downward. No tabs.
- `<Value>` and `<Repetition>` each on their own line, with the child
  `<Calculation>` indented beneath on a new line, and each closing tag
  on its own line.
- `<Name>` is the last child, one indent level deeper than `<Step>`.
- The variable-name element is the literal four-letter tag `<Name>` (capital N, lowercase a-m-e; bytes `0x4E 0x61 0x6D 0x65`). Never the single letter `<n>`, never an intuited expansion like `<VariableName>`. See the rendering-trap warning above.
- CDATA content may be single- or multi-line. Once the container
  structure is correct, calculation content does not affect paste.

## 4. Disabled steps

Any step may be disabled by setting `enable="False"` on the `<Step>`
tag. Structure is otherwise identical — all child elements are
preserved. The Script Workspace displays disabled steps with a `//`
prefix.

```
  <Step enable="False" id="21" name="Unsort Records"/>
  <Step enable="False" id="76" name="Set Field">
    <Calculation><![CDATA["anything"]]></Calculation>
    <Field table="tbl" id="N" name="fld"/>
  </Step>
```

Flow-control steps (`If`, `Else If`, `Else`, `End If`, `Loop`,
`End Loop`) accept `enable="False"` on the same terms. FileMaker does
not "repair" disabled control structures or enforce pairing — a
disabled `If` and disabled `End If` wrapping enabled body steps round-
trips intact, and the body steps execute unconditionally because the
disabled `If` is dropped from execution flow. This is the canonical
way to preserve commented-out conditional wrappers from earlier
revisions.

```
  <Step enable="False" id="68" name="If">
    <Restore state="False"/>
    <Calculation><![CDATA[some_disabled_condition]]></Calculation>
  </Step>
  <Step enable="True" id="76" name="Set Field">
    ...
  </Step>
  <Step enable="False" id="70" name="End If"/>
```

Round-trip verified: paste, save, reopen — the disabled flow-control
markers persist exactly as emitted, and the body steps run as if
unwrapped.

## 5. Common child-element conventions

The following patterns recur across many step types:

- `<NoInteract state="True"/>` — equivalent to "With dialog: Off" in
  the UI for record operations, finds, exports, etc.
- `<Option state="False"/>` — equivalent to "With dialog: Off" for
  steps where `<NoInteract>` does not apply.
- `<Restore state="True|False"/>` — indicates whether saved settings
  are restored (find requests, GTRR options, sort orders, print
  settings, import/export formats).
- `<SelectAll state="True|False"/>` — in insert and paste steps,
  controls whether the entire field is replaced (`True`) or content
  is inserted at the cursor (`False`).
- `<UniversalPathList type="Embedded"/>` — file-path container used by
  Insert File, Insert PDF, Insert Picture, Insert Audio/Video, and
  Fine-Tune Model. When paths are set, populated variants hold a list
  of paths.
- `<LayoutDestination value="OriginalLayout|CurrentLayout|SelectedLayout"/>` —
  target layout selector for steps that reference a layout.
- The four-letter `<Name>` tag (bytes `0x4E 0x61 0x6D 0x65`,
  capital N + lowercase a-m-e) is the canonical wrapper for any
  user-supplied identifier — variable names (Set Variable, Section 3),
  new-window names (Go to Related Record, Section 8.3; New Window,
  Section 8.8), close-by-name targets (Close Window, Section 8.8). The
  same tag spelling, the same nesting (CDATA `<Calculation>` for
  contexts that accept an expression; plain text for Set Variable's
  variable-name body). Never the single-letter form `<n>` — that is
  the silent-drop trigger documented in Section 7.4.
- Enumerated values appear as `<Element value="X"/>`.
- Boolean states appear as `<Element state="True|False"/>`.

### XML escaping in plain-text content

Most calculation content is wrapped in CDATA, where XML special
characters need no escaping. Find-mode `<Text>` elements and other
plain-text element bodies (notably `<Text>` inside comments and find
criteria) are *not* CDATA, so XML escaping is required:

- `>` becomes `&gt;`
- `<` becomes `&lt;`
- `&` becomes `&amp;`

For example, a find criterion of `>=$earliest_date` is encoded as
`<Text>&gt;=$earliest_date</Text>` inside the `<Criteria>` element.

### Comparison operator forms in calculations

FileMaker accepts both ASCII (`<>`, `<=`, `>=`) and Unicode (`≠`,
`≤`, `≥`) forms of the comparison operators inside CDATA calculation
content. Native Copy emits the Unicode glyphs; the paste handler
accepts either form, and the file does not canonicalise on save —
ASCII forms round-trip unchanged.

The two forms are interchangeable for paste correctness, but they are
not equivalent for transport:

- The Unicode glyphs are multi-byte under UTF-8 (`≠` is `0xE2 0x89
  0xA0`). Pipelines that touch the XML between generation and paste —
  shell `tr`/`sed` invoked with the wrong locale, log truncation,
  LLM tokenisers that split on byte boundaries, copy-paste through
  legacy clipboards — can corrupt or strip them silently.
- The ASCII forms are single-byte and survive any such pipeline
  intact.

Generators that emit XML for downstream consumption (LLM output, build
artefacts, transmitted snippets) should prefer the ASCII forms even
though FM's own Copy emits Unicode. Generators that read FM-emitted
XML and pass it through unchanged should preserve whatever form they
received.

### Field references vs. variable targets

Steps that write to a destination (Set Field, Insert from URL, Insert
Calculated Result, Insert Text, etc.) emit different `<Field>` forms
depending on the target type:

- **Field target** uses the structured form:
  `<Field table="TableOccurrence" id="N" name="field_name"/>`
- **Variable target** uses a plain-text body:
  `<Field>$variable_name</Field>`

The element name is the same (`<Field>`), only the contents differ.
This appears across any step where the target can be either a field
or a script variable.

### Runtime dependencies not enforced by the XML format

Pasted XML pastes if its structure is valid; the resulting script
will *run* correctly only if the recipient file contains every
referenced object. The format does not validate these references at
paste time, and several categories of reference fail silently or
opaquely at runtime rather than at paste. Generators should treat
these as a pre-flight checklist when producing XML for an unknown
target file.

- **Schema references by ID and name.** `<Field>`, `<Table>`,
  `<Layout>`, `<Script>`, `<CustomMenuSet>`, and similar elements
  carry both an `id` attribute and a `name` attribute. FileMaker
  resolves these by ID first, falling back to name when the ID has no
  match. A pasted reference whose ID does not exist in the recipient
  file produces a "missing reference" indicator on the step but does
  not prevent paste.

  In practice, name-fallback is reliable enough to be a primary
  generation strategy, not a degraded path. Round-trip testing
  confirms that XML emitted with placeholder `id="1"` on every schema
  reference pastes cleanly when the names match objects in the
  recipient file: FileMaker resolves each reference by name on paste,
  populates the real ID on save, and re-emits the resolved form on the
  next Copy. No missing-reference warnings, no manual repair.

  This is the **placeholder-ID pattern**: generators without DDR
  access (LLM script writers, code-generation utilities operating
  against an unknown recipient file, migration tools moving scripts
  between similar files) can emit `id="1"` (or any non-zero placeholder)
  uniformly and rely on name resolution. The only requirements are:
  every name attribute must exactly match the corresponding object's
  name in the recipient file (case-sensitive, whitespace-sensitive,
  trailing-space-sensitive), and the recipient must contain the named
  object. Generators that *do* have DDR access should still emit real
  IDs to skip the resolution step entirely; generators that do not
  should not block on it.

- **Custom functions in calculations.** Custom function calls inside
  CDATA calculation content (for example, `MyHelper ( $x )`) appear
  as plain text indistinguishable from built-in function calls. The
  XML carries no custom-function signature or dependency declaration.
  If the recipient file lacks the named custom function, the step
  pastes silently and produces an evaluation error at runtime. This
  is the most opaque failure mode in the format.

- **Plugin availability.** Plugin steps (the MBS step at id 186 and
  equivalent steps from other plugins) require the plugin to be
  installed and enabled on the machine running the script. The `index`
  attribute on plugin steps is also file-specific (see the MBS entry
  in Section 9 for details).

- **Host-specific runtime context.** Calculations using
  `Get ( HostName )`, `Get ( AccountName )`, `Get ( PrivilegeSetName )`,
  external data source references, and similar context-dependent
  functions evaluate against the runtime environment, not against
  anything in the XML. A script generated for one host may behave
  differently or fail when run against another.

- **External data source references.** Steps referencing an external
  FileMaker file (via Perform Script's external file syntax, GTRR
  to an external occurrence, etc.) carry the data source name but
  not the file's location. The recipient file must have a Manage
  External Data Sources entry resolving the named source.

The spec describes what produces *paste-clean* XML. Producing
*runtime-correct* scripts additionally requires the generator to
either validate references against a known recipient file's DDR
before generation, or to clearly mark generated scripts as requiring
post-paste verification of referenced objects.

## 6. Conditional elements

### 6.0 FM 26 universal elements

**DisableStepCollapsed.** FileMaker 2026 adds
`<DisableStepCollapsed state="False"/>` to every step in native Copy
output. This supports the collapsible disabled-block feature in the
Script Workspace. The element is not required for paste — FM 26
accepts XML without it and adds it on re-export. Generators may omit
it. The skeletons in this spec omit it for clarity.

**Window UUID selection.** Close Window (121), Select Window (123),
Move/Resize Window (119), and Set Window Title (124) now accept a
UUID in the `<Name>` calculation as well as a window name. At
runtime, FM searches by name first, then falls back to UUID matching.
The XML format is unchanged: `<Window value="ByName"/>` with `<Name>`
containing a `<Calculation>` child. No structural change.

Minimal skeletons in the step reference below show what FileMaker emits
for an unconfigured or default step. Additional child elements appear
only when the corresponding option is configured. Common examples:

- Go to Related Record emits `<Name>` only when "Show in new window"
  is enabled and a window name is set. Dimension elements
  (`<Height>`, `<Width>`, `<DistanceFromTop>`, `<DistanceFromLeft>`)
  appear only when dimensions are set in the dialog, independent of
  window state.
- Perform Script on Server emits `<Calculation>` only when a parameter
  is set.
- Show Custom Dialog emits `<Message>` and `<Buttons>` only when
  configured; the default skeleton is self-closing.

## 7. Known silent-failure modes

These three patterns paste without error in FileMaker but produce
broken steps. They are the primary reason this specification exists.
Each was discovered by round-trip testing — the failure mode is not
visible from the XML alone.

### 7.1 Set Variable `<Name>` element dropped

**Trigger:** Set Variable (id 141) emitted with non-canonical
indentation, tab characters, or compact `<Value>`/`<Repetition>`
form.

**Symptom:** Step pastes successfully and appears in the Script
Workspace, but the variable name is blank: `Set Variable [ ; Value: ... ]`.

**Fix:** Use exactly the structure documented in Section 3 — 2-space
indent, expanded `<Value>` and `<Repetition>` blocks with
`<Calculation>` on its own line, `<Name>` as the last child.

### 7.2 Perform JavaScript in Web Viewer parameters dropped

**Trigger:** Perform JavaScript in Web Viewer (id 175) emitted with
parameters as flat `<Parameter>` elements (the obvious-but-wrong
form) instead of FileMaker's actual structure.

**Symptom:** Step pastes successfully with object name and function
name visible, but no parameters are passed. The JavaScript function
is called with no arguments.

**Fix:** Wrap parameters in a `<Parameters Count="N">` container with
`<P>` children — the `<P>` element name is the canonical form, not
`<Parameter>`. See Section 8.14 for the verified structure.

### 7.3 Install OnTimer Script interval bound to wrong slot

**Trigger:** Install OnTimer Script (id 148) emitted with the
interval as a top-level `<Calculation>` instead of wrapped in an
`<Interval>` element.

**Symptom:** Step pastes successfully and appears in the Script
Workspace as if the interval value is a script parameter. The actual
timer interval is unset and the timer never fires at the intended
rate.

**Fix:** Wrap the interval in an `<Interval>` element containing the
`<Calculation>` child. See Section 8.1 for the verified structure.

### 7.4 Spec-rendering corruption of the canonical Name tag

**Trigger:** This document is consumed by an LLM (or human) through a
markdown processor — or any rendering layer — that strips inner content
from anything resembling an HTML element with the spelling N-a-m-e.
The four-letter canonical tag for variable names, window names, and
similar elements then appears as a single letter `<n>` throughout the
document. The reader trusts the rendering, intuits that `<n>` is the
literal canonical form, and emits the single-letter version in
generated XML.

**Symptom:** Functionally identical to Section 7.1, but multiplied
across every step that uses the four-letter tag. Every Set Variable
pastes with no variable name. Every Close Window By Name pastes with
no window name. Every GTRR-with-new-window-name pastes with no window
name. The XML is structurally valid and the steps appear in the Script
Workspace; the silent drop is uniform and silent.

This failure mode is upstream of FileMaker entirely. It is produced by
the spec's delivery channel, not by FileMaker's paste handler. From
the generator's point of view it is indistinguishable from 7.1 —
structurally valid XML that drops critical content on paste — so the
defensive posture is the same.

**Fix.** Verify the byte-level form of the tag from a plaintext or
hex source before generation. The four ASCII bytes are
`0x4E 0x61 0x6D 0x65`: capital N followed by lowercase a, m, e. When
referencing the tag in any prompt, specification, or comment passed
to an LLM, prefer the hyphen-spelled form (`N-a-m-e`) or the explicit
byte sequence; both survive markdown rendering layers that strip the
backtick-quoted tag form. The single-letter form `<n>` should appear
in generator code only as a counterexample with a comment marking it
as wrong.

**Defensive construction pattern.** The tag must never appear as a
literal in source code that flows through markdown, JSX, HTML
templating, or any LLM context window. Rendering layers that strip
content from `Name`-spelled HTML elements will silently corrupt
literal occurrences. The safe pattern is to construct the tag from
its byte values at runtime, so the tag exists only in process memory,
never in source:

```python
# Python example — applies equivalently to any language that
# supports character-code construction.
NAME_OPEN  = "<" + chr(0x4E) + chr(0x61) + chr(0x6D) + chr(0x65) + ">"
NAME_CLOSE = "</" + chr(0x4E) + chr(0x61) + chr(0x6D) + chr(0x65) + ">"

# Use NAME_OPEN / NAME_CLOSE in emitted XML.
# The tag NEVER appears as a literal in this source file.
```

Generators that adopt this pattern are immune to Section 7.4
corruption regardless of how the spec, prompts, or build artefacts
are rendered downstream. Generators that emit the tag as a literal
string (e.g. `xml += "<Name>" + var + "</Name>"`) are vulnerable any
time that source file passes through a rendering layer — including
when it is itself sent as context to an LLM. Code review for any
FM-XML generator should treat literal occurrences of the four-letter
tag in source as a bug class to be eliminated, not just commented.

**Where this tag appears in the spec.** Section 3 (Set Variable
variable name), Section 8.3 (Go to Related Record new-window name),
Section 8.8 (New Window name; Close Window By Name window name).
Each of those sections must be read with this rendering trap in mind.

**Why this is in Section 7 rather than a separate appendix.** The
other three failure modes in this section produce structurally valid
XML that pastes with silent content drops. So does this one. The fact
that the cause is a rendering layer rather than FileMaker's paste
handler is irrelevant from the generator's perspective: the symptom,
the diagnostic, and the fix posture are identical. Round-trip every
new step type that uses this tag before treating its structure as
verified.

### Pattern across all four

Each silent-failure case involves an element name or wrapper where
the FileMaker-canonical form is shorter or less obvious than what an
LLM or a developer would intuit: `<Name>` (not `<VariableName>`),
`<P>` (not `<Parameter>`), `<Interval>` wrapper (not bare
`<Calculation>`). When FileMaker's paste handler encounters a
calculation with no expected wrapper, it appears to bind the
calculation to a default slot rather than reject the step. Generators
that infer structure from related steps will hit these failures
predictably.

This list is not exhaustive. Other silent-failure cases may exist in
step types that have not yet been round-trip tested in their fully
configured form, and Section 7.4 in particular is likely to be re-
discovered any time this spec is re-rendered through a new processor.
The defensive posture is: when adding support for a new step type,
generate a minimal version, paste it, drill into the step in the
Script Workspace, and confirm every configured option displays as
expected before treating the structure as verified. For tags affected
by Section 7.4, additionally verify the byte-level form of the
generated XML against the literal byte sequence, not the rendered
display.
## Appendix A: Open observations

### A.1 NewWndStyles bit values and extended attributes

Three distinct `Styles` attribute values have been observed on
`<NewWndStyles>` elements:

- `Styles="983554"` — emitted on configured Go to Related Record steps
  taken from production scripts. Round-trip verified as the default
  value for any configured GTRR (with or without new-window options)
  whose window-style attributes (`Style`, `Close`, `Minimize`,
  `Maximize`, `Resize`) are unmodified from FM's defaults. Generators
  emitting GTRR steps without explicit window-style configuration
  should use this value — it round-trips unchanged through Copy/Paste/
  save cycles.
- `Styles="3606018"` — emitted on default or unconfigured Go to Related
  Record, New Window, and Go to List of Records steps.
- `Styles="1076299266"` — emitted on a New Window step configured with
  a window name and additional window-style options
  (DimParentWindow=No, Toolbars=Yes, MenuBar=Yes).

The meaning of the differing bits has not been established. The values
are candidates for systematic bit-flag decomposition: toggling
individual window-style options in the dialog and comparing the
resulting numeric `Styles` values would identify each bit's role.

`<NewWndStyles>` may also carry additional attributes beyond the
common set (`Style`, `Close`, `Minimize`, `Maximize`, `Resize`). The
following have been observed in configured steps:

- `DimParentWindow="No|Yes"` — corresponds to "Dim parent window"
  option
- `Toolbars="Yes|No"` — corresponds to whether toolbars are shown in
  the new window
- `MenuBar="Yes|No"` — corresponds to whether the menu bar is shown

These optional attributes appear when the corresponding options are
configured in the step's dialog. Their relationship to the numeric
`Styles` value is not yet mapped.

---

## Appendix B: Preserved quirks in FileMaker's native output

The following irregularities appear in FileMaker's own Copy output and
must be preserved verbatim in generated XML. They are FileMaker's
behaviour, not errors in this document.

### B.1 Configure AI Account — misspelled element (FM 2025 only)

Step 212 emitted `<SetLLMAccout/>` in FM 2025 (missing the `n` in
`Account`). FM 26 corrected this to `<SetLLMAccount/>`. Generators
targeting FM 26 should use the corrected spelling. Generators
targeting FM 2025 should use the misspelled form.

### B.2 Configure RAG Account — trailing space in step name

Step 227 emits `name="Configure RAG Account "` with a trailing space
inside the attribute value. The trailing space is part of FileMaker's
native output and must be preserved.

---

## Appendix C: FM 26 PDF error codes

| Code | Meaning | Steps |
|------|---------|-------|
| 605 | Container field is empty | Print PDF |
| 606 | Container data is not a PDF | Print PDF |
| 607 | Password missing or incorrect for encrypted PDF | Print PDF |
| 608 | PDF security settings prevent printing | Print PDF |
| 829 | No PDF file is open to append to | Append PDF |
| 830 | PDF file not found or invalid format | Open PDF |
| 831 | Invalid PDF password | Open PDF, Append PDF |
| 832 | PDF security settings prevent modification | Open PDF |
| 833 | PDF file is already opened | Create PDF, Open PDF |

---
