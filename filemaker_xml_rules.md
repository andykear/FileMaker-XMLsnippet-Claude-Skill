# Canonical XML Format for FileMaker Script Steps

**Author:** Andrew Kear, Clockwork Creative Technology
**Version:** 1.10.4
**Date:** May 2026
**Verified against:** FileMaker Pro 2025 on macOS
**Licence:** [Creative Commons Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/)
— free to use, share, and adapt with attribution.

---

## Why this exists

FileMaker's Script Workspace can paste XML directly from the system
clipboard, allowing entire scripts to be assembled outside FileMaker
and pasted in. This capability is foundational for AI-assisted
script generation, programmatic script transformation, code review
tooling, and migration utilities — but the format the paste handler
accepts is undocumented.

The XML parser is lenient. The paste handler is not. The two have
different rules, and the gap between them is where silent failures
live: XML that is technically valid but pastes with elements
quietly dropped or rebound to the wrong internal slot. The classic
example is a Set Variable step that pastes successfully but with no
variable name assigned, because the `<Name>` element appeared in a
position the paste handler did not expect.

This document is a reverse-engineered reference for what the paste
handler actually accepts. It is not the format Claris uses
internally; it is the format that survives the round trip from
clipboard to Script Workspace and back. Where FileMaker's native
emission contains quirks (typos, trailing spaces, cryptic numeric
attributes), those quirks are preserved exactly, because deviations
from the native form are precisely what trigger the silent
failures.

The intended audience is developers building tools that generate
FileMaker script XML — particularly authors of LLM prompts,
custom-built generators, and integration utilities. Empirical
testing across multiple LLMs (Claude, ChatGPT, Gemini) has shown
that providing this specification as context dramatically reduces
hallucination of element names, structure, and ordering. Without it,
the same models produce non-pasting output reliably.

## Scope

**This document covers:** the structural XML format that
`fmxmlsnippet type="FMObjectList"` clipboards must follow to paste
cleanly into the FileMaker Script Workspace. It documents
canonical skeletons for every script step (~180), verified
configured forms for the steps most commonly used in real
automation, and three known silent-failure modes.

**This document does not cover:** Custom Function definitions
(different wrapper, different format), layout objects, theme XML,
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

## 8. Step reference

> **Reminder.** Every snippet generated from this section must be wrapped in the XML declaration and `fmxmlsnippet type="FMObjectList"` element specified in Section 1. Bare step elements pasted without the wrapper are not recognised by the Script Workspace clipboard.
>
> For a consolidated lookup of every step ID documented below, see Appendix F.

Skeletons below show FileMaker's native minimal output for each step.
Where rich configured variants exist, they appear inline with the
skeleton. All configured variants in this section have been verified
by round-trip testing.

---

### 8.1 Control

#### If (68)
```
  <Step enable="True" id="68" name="If">
    <Restore state="False"/>
    <Calculation><![CDATA[condition]]></Calculation>
  </Step>
```

#### Else If (125)
Identical structure to If:
```
  <Step enable="True" id="125" name="Else If">
    <Restore state="False"/>
    <Calculation><![CDATA[condition]]></Calculation>
  </Step>
```

#### Else (69)
```
  <Step enable="True" id="69" name="Else">
    <Restore state="False"/>
  </Step>
```

#### End If (70)
```
  <Step enable="True" id="70" name="End If"/>
```

#### Loop (71)
```
  <Step enable="True" id="71" name="Loop">
    <Restore state="False"/>
    <FlushType value="Always"/>
  </Step>
```

#### Exit Loop If (72)
```
  <Step enable="True" id="72" name="Exit Loop If">
    <Calculation><![CDATA[condition]]></Calculation>
  </Step>
```

#### End Loop (73)
```
  <Step enable="True" id="73" name="End Loop"/>
```

#### Exit Script (103)
```
  <Step enable="True" id="103" name="Exit Script"/>
```

With return value:
```
  <Step enable="True" id="103" name="Exit Script">
    <Calculation><![CDATA[return value]]></Calculation>
  </Step>
```

#### Halt Script (90)
```
  <Step enable="True" id="90" name="Halt Script"/>
```

#### Pause/Resume Script (62)
```
  <Step enable="True" id="62" name="Pause/Resume Script">
    <PauseTime value="Indefinitely"/>
  </Step>
```

#### Install OnTimer Script (148)

Cancel-all form (no children):
```
  <Step enable="True" id="148" name="Install OnTimer Script"/>
```

Configured with a script reference and an interval. The interval
must be wrapped in an `<Interval>` element containing a
`<Calculation>` child. Element order: `Interval` → `Script`.
```
  <Step enable="True" id="148" name="Install OnTimer Script">
    <Interval>
      <Calculation><![CDATA[5]]></Calculation>
    </Interval>
    <Script id="N" name="script name"/>
  </Step>
```

The interval is in seconds. The `<Calculation>` content can be a
literal number or any expression that evaluates to a number.

**Silent-drop warning.** If the `<Calculation>` for the interval is
emitted as a direct child of `<Step>` rather than wrapped in
`<Interval>`, FileMaker pastes the step without error but binds the
calculation to the wrong internal slot. The step then appears in the
Script Workspace as if it has a script parameter set, but the actual
interval is unset — the timer will never fire at the intended rate.
This is a verified silent-failure mode similar to the Set Variable
`<Name>` and Perform JavaScript `<P>` cases.

#### Open Transaction (205)
```
  <Step enable="True" id="205" name="Open Transaction">
    <Option state="False"/>
    <ESSForceCommit state="False"/>
    <SkipAutoEntry state="False"/>
    <Restore state="False"/>
  </Step>
```

#### Commit Transaction (206)
```
  <Step enable="True" id="206" name="Commit Transaction"/>
```

#### Revert Transaction (207)
```
  <Step enable="True" id="207" name="Revert Transaction">
    <Option state="False"/>
  </Step>
```

#### Set Revert Transaction on Error (223)
```
  <Step enable="True" id="223" name="Set Revert Transaction on Error">
    <Set state="False"/>
  </Step>
```

#### Perform Script (1)
```
  <Step enable="True" id="1" name="Perform Script">
    <Script id="N" name="script name"/>
  </Step>
```

#### Perform Script on Server (164)

Without parameter:
```
  <Step enable="True" id="164" name="Perform Script on Server">
    <WaitForCompletion state="True"/>
    <Script id="N" name="script name"/>
  </Step>
```

With parameter:
```
  <Step enable="True" id="164" name="Perform Script on Server">
    <WaitForCompletion state="True"/>
    <Calculation><![CDATA[parameter expression]]></Calculation>
    <Script id="N" name="script name"/>
  </Step>
```

#### Perform Script on Server with Callback (210)
```
  <Step enable="True" id="210" name="Perform Script on Server with Callback">
    <CallbackScriptState value="Continue"/>
    <CallbackScript/>
  </Step>
```

#### Set Error Capture (86)
```
  <Step enable="True" id="86" name="Set Error Capture">
    <Set state="True"/>
  </Step>
```

#### Set Error Logging (200)
```
  <Step enable="True" id="200" name="Set Error Logging">
    <Option state="False"/>
  </Step>
```

#### Set Variable (141)
See Section 3.

#### Allow User Abort (85)
```
  <Step enable="True" id="85" name="Allow User Abort">
    <Set state="False"/>
  </Step>
```

#### Set Layout Object Animation (168)
```
  <Step enable="True" id="168" name="Set Layout Object Animation">
    <Set state="True"/>
  </Step>
```

#### Trigger Claris Connect Flow (211)
```
  <Step enable="True" id="211" name="Trigger Claris Connect Flow">
    <NoInteract state="True"/>
    <DontEncodeURL state="False"/>
    <SelectAll state="True"/>
    <VerifySSLCertificates state="False"/>
    <Flow>flow identifier</Flow>
    <CURLOptions>
      <Calculation><![CDATA["--request POST --header \"Content-Type: application/json\" --data "]]></Calculation>
    </CURLOptions>
    <Text>payload</Text>
  </Step>
```

Unconfigured steps emit `&lt;unknown&gt;` in the `<Flow>` and `<Text>`
elements.

#### Comment (89)
```
  <Step enable="True" id="89" name="# (comment)"/>
```

With text:
```
  <Step enable="True" id="89" name="# (comment)">
    <Text>comment text</Text>
  </Step>
```

The self-closing form (no `<Text>` child, not an empty `<Text/>`) is
the canonical form for the bare divider lines that punctuate most
production scripts. Round-trip verified: a blank `# ` line in the
Script Workspace emits as the self-closing form on Copy, and an
empty `<Text/>` body is not the equivalent — generators should emit
the self-closing form for divider lines.

---

### 8.2 Configure (mobile and hardware)

#### Configure Local Notification (187)
```
  <Step enable="True" id="187" name="Configure Local Notification">
    <Action value="Queue"/>
  </Step>
```

#### Configure NFC Reading (201)
```
  <Step enable="True" id="201" name="Configure NFC Reading">
    <Action value="Read"/>
  </Step>
```

#### Configure Region Monitor Script (185)
```
  <Step enable="True" id="185" name="Configure Region Monitor Script">
    <MonitorType value="iBeacon"/>
  </Step>
```

---

### 8.3 Navigation

#### Close Popover (169)
```
  <Step enable="True" id="169" name="Close Popover"/>
```

#### Enter Browse Mode (55)
```
  <Step enable="True" id="55" name="Enter Browse Mode">
    <Pause state="False"/>
  </Step>
```

#### Enter Find Mode (22)
```
  <Step enable="True" id="22" name="Enter Find Mode">
    <Pause state="True"/>
    <Restore state="False"/>
  </Step>
```

#### Enter Preview Mode (41)
```
  <Step enable="True" id="41" name="Enter Preview Mode">
    <Pause state="False"/>
  </Step>
```

#### Go to Field (17)
```
  <Step enable="True" id="17" name="Go to Field">
    <SelectAll state="False"/>
  </Step>
```

With a target field, adds the structured `<Field>` reference:
```
  <Step enable="True" id="17" name="Go to Field">
    <SelectAll state="False"/>
    <Field table="TableOccurrence" id="N" name="field_name"/>
  </Step>
```

#### Go to Layout (6)
```
  <Step enable="True" id="6" name="Go to Layout">
    <LayoutDestination value="OriginalLayout"/>
  </Step>
```

With a specific layout:
```
  <Step enable="True" id="6" name="Go to Layout">
    <LayoutDestination value="SelectedLayout"/>
    <Layout id="N" name="layout name"/>
  </Step>
```

#### Go to List of Records (228)
```
  <Step enable="True" id="228" name="Go to List of Records">
    <ShowInNewWindow state="False"/>
    <LayoutDestination value="CurrentLayout"/>
    <NewWndStyles Style="Document" Close="Yes" Minimize="Yes" Maximize="Yes" Resize="Yes" Styles="3606018"/>
  </Step>
```

#### Go to Next Field (4)
```
  <Step enable="True" id="4" name="Go to Next Field"/>
```

#### Go to Object (145)
```
  <Step enable="True" id="145" name="Go to Object"/>
```

With object name:
```
  <Step enable="True" id="145" name="Go to Object">
    <ObjectName>
      <Calculation><![CDATA["object name"]]></Calculation>
    </ObjectName>
  </Step>
```

#### Go to Portal Row (99)
```
  <Step enable="True" id="99" name="Go to Portal Row">
    <NoInteract state="False"/>
    <SelectAll state="False"/>
    <RowPageLocation value="First"/>
  </Step>
```

`<RowPageLocation>` enumeration matches Go to Record/Request/Page:
`First`, `Next`, `Previous`, `Last`, `ByCalculation`. For
`ByCalculation`, add a `<Calculation>` child:
```
  <Step enable="True" id="99" name="Go to Portal Row">
    <NoInteract state="True"/>
    <SelectAll state="True"/>
    <RowPageLocation value="ByCalculation"/>
    <Calculation><![CDATA[$loop_counter]]></Calculation>
  </Step>
```

#### Go to Previous Field (5)
```
  <Step enable="True" id="5" name="Go to Previous Field"/>
```

#### Go to Record/Request/Page (16)
```
  <Step enable="True" id="16" name="Go to Record/Request/Page">
    <NoInteract state="False"/>
    <RowPageLocation value="First"/>
  </Step>
```

`<RowPageLocation value="...">` enumeration: `First`, `Next`,
`Previous`, `Last`, `ByCalculation`. For `Next`, an optional
`<Exit state="True"/>` element before `<RowPageLocation>` indicates
"Exit after last". For `ByCalculation`, add a `<Calculation>` child.

#### Go to Related Record (74)

Element order is fixed: `Option` → `MatchAllRecords` →
`ShowInNewWindow` → `Restore` → `LayoutDestination` → optional
`Name` → optional dimensions → `NewWndStyles` → `Table` → `Layout`.

`<NewWndStyles>` is always present, regardless of new-window state.

Element semantics — verified by round-trip against production scripts:

- `<Option state="True|False"/>` — "Show only related records" in the
  GTRR dialog. `True` constrains the destination found set to the
  related set; `False` shows all records on the destination layout.
  This is the most consequential GTRR option and the most commonly
  set; the default in the dialog is `True` for new GTRR steps in most
  production contexts but `False` in the unconfigured XML emission.
  Generators that omit this element or set it incorrectly will
  silently produce the wrong found-set behaviour at runtime — the
  step pastes and runs, just on the wrong record set.
- `<MatchAllRecords state="True|False"/>` — "Match found set" in the
  dialog. `True` matches against the entire found set on the source
  side; `False` matches only the current record. Almost always
  `False` in practice.
- `<ShowInNewWindow state="True|False"/>` — controls the new-window
  branch. When `True`, the optional `<n>` (window-name calculation)
  and dimension elements may appear before `<NewWndStyles>`.
- `<Restore state="True|False"/>` — restore saved sort/find settings
  attached to the GTRR step. Maps to whether the step has a stored
  sort order or find criteria from the dialog. Almost always `True`
  in configured GTRRs even when no specific settings are saved.

**Basic form** — existing window, no custom dimensions:
```
  <Step enable="True" id="74" name="Go to Related Record">
    <Option state="False"/>
    <MatchAllRecords state="False"/>
    <ShowInNewWindow state="False"/>
    <Restore state="True"/>
    <LayoutDestination value="SelectedLayout"/>
    <NewWndStyles Style="Document" Close="Yes" Minimize="Yes" Maximize="Yes" Resize="Yes" Styles="983554"/>
    <Table id="N" name="table occurrence name"/>
    <Layout id="N" name="layout name"/>
  </Step>
```

**New window** — adds `<Name>` with calculation between
`<LayoutDestination>` and `<NewWndStyles>`:
```
    <ShowInNewWindow state="True"/>
    <Restore state="True"/>
    <LayoutDestination value="SelectedLayout"/>
    <Name>
      <Calculation><![CDATA["window name"]]></Calculation>
    </Name>
    <NewWndStyles .../>
```

**Custom dimensions** — adds dimension elements between optional
`<Name>` and `<NewWndStyles>`. Dimensions are independent of new-window
state:
```
    <Height>
      <Calculation><![CDATA[400]]></Calculation>
    </Height>
    <Width>
      <Calculation><![CDATA[400]]></Calculation>
    </Width>
    <DistanceFromTop>
      <Calculation><![CDATA[400]]></Calculation>
    </DistanceFromTop>
    <DistanceFromLeft>
      <Calculation><![CDATA[400]]></Calculation>
    </DistanceFromLeft>
    <NewWndStyles .../>
```

**Unconfigured GTRR** (no table set):
```
  <Step enable="True" id="74" name="Go to Related Record">
    <Option state="False"/>
    <MatchAllRecords state="False"/>
    <ShowInNewWindow state="False"/>
    <Restore state="False"/>
    <LayoutDestination value="CurrentLayout"/>
    <NewWndStyles Style="Document" Close="Yes" Minimize="Yes" Maximize="Yes" Resize="Yes" Styles="3606018"/>
    <Table id="0" name=""/>
  </Step>
```

---

### 8.4 Editing

#### Clear (49)
```
  <Step enable="True" id="49" name="Clear">
    <SelectAll state="True"/>
  </Step>
```

#### Copy (47)
```
  <Step enable="True" id="47" name="Copy">
    <SelectAll state="True"/>
  </Step>
```

#### Cut (46)
```
  <Step enable="True" id="46" name="Cut">
    <SelectAll state="True"/>
  </Step>
```

#### Paste (48)
```
  <Step enable="True" id="48" name="Paste">
    <NoStyle state="True"/>
    <SelectAll state="True"/>
    <LinkAvail state="False"/>
  </Step>
```

#### Perform Find/Replace (128)
```
  <Step enable="True" id="128" name="Perform Find/Replace">
    <NoInteract state="False"/>
    <FindReplaceOperation MatchWholeWords="False" MatchCase="False" WithinOptions="All" AcrossOptions="All" direction="Forward" type="FindNext"/>
  </Step>
```

#### Select All (50)
```
  <Step enable="True" id="50" name="Select All"/>
```

#### Set Selection (130)
```
  <Step enable="True" id="130" name="Set Selection"/>
```

#### Undo/Redo (45)
```
  <Step enable="True" id="45" name="Undo/Redo">
    <UndoRedo value="Undo"/>
  </Step>
```

---

### 8.5 Fields

#### Export Field Contents (132)
```
  <Step enable="True" id="132" name="Export Field Contents">
    <CreateDirectories state="True"/>
    <AutoOpen state="False"/>
    <CreateEmail state="False"/>
  </Step>
```

#### Insert Audio/Video (159)
```
  <Step enable="True" id="159" name="Insert Audio/Video">
    <UniversalPathList type="Embedded"/>
  </Step>
```

#### Insert Calculated Result (77)
```
  <Step enable="True" id="77" name="Insert Calculated Result">
    <SelectAll state="True"/>
  </Step>
```

#### Insert Current Date (13)
```
  <Step enable="True" id="13" name="Insert Current Date">
    <SelectAll state="True"/>
  </Step>
```

#### Insert Current Time (14)
```
  <Step enable="True" id="14" name="Insert Current Time">
    <SelectAll state="True"/>
  </Step>
```

#### Insert Current User Name (60)
```
  <Step enable="True" id="60" name="Insert Current User Name">
    <SelectAll state="True"/>
  </Step>
```

#### Insert File (131)
```
  <Step enable="True" id="131" name="Insert File">
    <UniversalPathList type="Embedded"/>
    <DialogOptions asFile="True" enable="False">
      <Storage type="UserChoice"/>
      <Compress type="UserChoice"/>
      <FilterList/>
    </DialogOptions>
  </Step>
```

#### Insert from Device (161)
```
  <Step enable="True" id="161" name="Insert from Device">
    <InsertFrom value="Camera"/>
    <DeviceOptions>
      <Camera choice="Back"/>
      <Resolution choice="Full"/>
    </DeviceOptions>
  </Step>
```

#### Insert from Index (11)
```
  <Step enable="True" id="11" name="Insert from Index">
    <SelectAll state="True"/>
  </Step>
```

#### Insert from Last Visited (12)
```
  <Step enable="True" id="12" name="Insert from Last Visited">
    <SelectAll state="True"/>
  </Step>
```

#### Insert from URL (160)
```
  <Step enable="True" id="160" name="Insert from URL">
    <NoInteract state="True"/>
    <DontEncodeURL state="False"/>
    <SelectAll state="True"/>
    <VerifySSLCertificates state="False"/>
  </Step>
```

Configured with a target variable, URL, and cURL options. Element
order is fixed: `NoInteract` → `DontEncodeURL` → `SelectAll` →
`VerifySSLCertificates` → `CURLOptions` (optional) → `Calculation`
(the URL) → `Text` → `Field`.
```
  <Step enable="True" id="160" name="Insert from URL">
    <NoInteract state="True"/>
    <DontEncodeURL state="False"/>
    <SelectAll state="True"/>
    <VerifySSLCertificates state="False"/>
    <CURLOptions>
      <Calculation><![CDATA[$curl]]></Calculation>
    </CURLOptions>
    <Calculation><![CDATA[$endpoint]]></Calculation>
    <Text/>
    <Field>$result</Field>
  </Step>
```

Notes:

- The URL is held in a top-level `<Calculation>` element, not wrapped
  in a `<URL>` element.
- `<CURLOptions>` is optional and contains a `<Calculation>` child.
- `<Text/>` is typically empty in URL-fetch usage (its purpose is the
  POST/PUT body for some configurations, otherwise self-closing).
- `<Field>` carries the target. For a variable target, the body is
  the variable name as plain text: `<Field>$result</Field>`. For a
  field target, the structured reference form applies:
  `<Field table="TableOccurrence" id="N" name="field_name"/>`.

#### Insert PDF (158)
```
  <Step enable="True" id="158" name="Insert PDF">
    <UniversalPathList type="Embedded"/>
  </Step>
```

#### Insert Picture (56)
```
  <Step enable="True" id="56" name="Insert Picture">
    <UniversalPathList type="Embedded"/>
  </Step>
```

#### Insert Text (61)
```
  <Step enable="True" id="61" name="Insert Text">
    <SelectAll state="True"/>
  </Step>
```

#### Relookup Field Contents (40)
```
  <Step enable="True" id="40" name="Relookup Field Contents">
    <NoInteract state="True"/>
  </Step>
```

#### Replace Field Contents (91)
```
  <Step enable="True" id="91" name="Replace Field Contents">
    <NoInteract state="True"/>
    <Restore state="False"/>
    <With value="None"/>
    <SerialNumbers PerformAutoEnter="False" UpdateEntryOptions="False" increment="0" InitialValue="" UseEntryOptions="False"/>
  </Step>
```

#### Set Field (76)
```
  <Step enable="True" id="76" name="Set Field"/>
```

Configured:
```
  <Step enable="True" id="76" name="Set Field">
    <Calculation><![CDATA[expression]]></Calculation>
    <Field table="TableOccurrence" id="N" name="field_name"/>
  </Step>
```

#### Set Field By Name (147)
```
  <Step enable="True" id="147" name="Set Field By Name"/>
```

#### Set Next Serial Value (116)
```
  <Step enable="True" id="116" name="Set Next Serial Value"/>
```

---

### 8.6 Records

#### Commit Records/Requests (75)
```
  <Step enable="True" id="75" name="Commit Records/Requests">
    <NoInteract state="True"/>
    <Option state="False"/>
    <ESSForceCommit state="False"/>
  </Step>
```

#### Copy All Records/Requests (98)
```
  <Step enable="True" id="98" name="Copy All Records/Requests"/>
```

#### Copy Record/Request (101)
```
  <Step enable="True" id="101" name="Copy Record/Request"/>
```

#### Delete All Records (10)
```
  <Step enable="True" id="10" name="Delete All Records">
    <NoInteract state="False"/>
  </Step>
```

#### Delete Portal Row (104)
```
  <Step enable="True" id="104" name="Delete Portal Row">
    <NoInteract state="False"/>
  </Step>
```

#### Duplicate Record/Request (8)
```
  <Step enable="True" id="8" name="Duplicate Record/Request"/>
```

#### Delete Record/Request (9)
```
  <Step enable="True" id="9" name="Delete Record/Request">
    <NoInteract state="False"/>
  </Step>
```

#### Export Records (36)
```
  <Step enable="True" id="36" name="Export Records">
    <NoInteract state="True"/>
    <CreateDirectories state="True"/>
    <Restore state="False"/>
    <AutoOpen state="False"/>
    <CreateEmail state="False"/>
  </Step>
```

#### Import Records (35)
```
  <Step enable="True" id="35" name="Import Records">
    <NoInteract state="True"/>
    <Restore state="False"/>
    <VerifySSLCertificates state="False"/>
  </Step>
```

#### New Record/Request (7)
```
  <Step enable="True" id="7" name="New Record/Request"/>
```

#### Open Record/Request (133)
```
  <Step enable="True" id="133" name="Open Record/Request"/>
```

#### Revert Record/Request (51)
```
  <Step enable="True" id="51" name="Revert Record/Request">
    <NoInteract state="True"/>
  </Step>
```

#### Save Records as Excel (143)
```
  <Step enable="True" id="143" name="Save Records as Excel">
    <NoInteract state="True"/>
    <CreateDirectories state="True"/>
    <Restore state="False"/>
    <AutoOpen state="False"/>
    <CreateEmail state="False"/>
    <SaveType value="BrowsedRecords"/>
    <UseFieldNames state="False"/>
  </Step>
```

#### Save Records as JSONL (225)
```
  <Step enable="True" id="225" name="Save Records as JSONL">
    <Option state="False"/>
    <CreateDirectories state="False"/>
    <FineTuneFormat state="False"/>
    <AutoOpen state="False"/>
    <CreateEmail state="False"/>
    <SaveAsJSONL/>
  </Step>
```

#### Save Records as PDF (144)
```
  <Step enable="True" id="144" name="Save Records as PDF">
    <NoInteract state="True"/>
    <Option state="False"/>
    <CreateDirectories state="True"/>
    <Restore state="False"/>
    <AutoOpen state="False"/>
    <CreateEmail state="False"/>
    <PDFOptions source="RecordsBeingBrowsed">
      <Document>
        <Pages AllPages="True">
          <NumberFrom>
            <Calculation><![CDATA[1]]></Calculation>
          </NumberFrom>
          <PageRange>
            <From>
              <Calculation><![CDATA[1]]></Calculation>
            </From>
            <To>
              <Calculation><![CDATA[1]]></Calculation>
            </To>
          </PageRange>
        </Pages>
      </Document>
      <Security allowScreenReader="True" enableCopying="True" controlEditing="AnyExceptExtractingPages" controlPrinting="HighResolution" requireControlEditPassword="False" requireOpenPassword="False"/>
      <View magnification="100" pageLayout="SinglePage" show="PagesPanelAndPage"/>
    </PDFOptions>
  </Step>
```

#### Save Records as Snapshot Link (152)
```
  <Step enable="True" id="152" name="Save Records as Snapshot Link">
    <CreateDirectories state="True"/>
    <CreateEmail state="False"/>
    <SaveType value="BrowsedRecords"/>
  </Step>
```

#### Truncate Table (182)
```
  <Step enable="True" id="182" name="Truncate Table">
    <NoInteract state="False"/>
    <BaseTable id="-1" name="&lt;Current Table&gt;"/>
  </Step>
```

---

### 8.7 Found Sets

All find-mode steps share a `<Query>` structure when configured. Each
`<RequestRow>` represents one find request with operation
`Include` (find) or `Exclude` (omit). Multiple `<Criteria>` elements
within one RequestRow combine with AND; multiple RequestRows combine
with OR.

#### Constrain Found Set (126)
```
  <Step enable="True" id="126" name="Constrain Found Set">
    <Option state="False"/>
    <Restore state="False"/>
  </Step>
```

With saved find criteria:
```
  <Step enable="True" id="126" name="Constrain Found Set">
    <Option state="False"/>
    <Restore state="True"/>
    <Query>
      <RequestRow operation="Include">
        <Criteria>
          <Field table="TableOccurrence" id="N" name="field_name"/>
          <Text>=</Text>
        </Criteria>
      </RequestRow>
    </Query>
  </Step>
```

The `<Text>` element holds the find criterion exactly as typed in find
mode (`=` for exact-empty, `>100`, `Jones`, `//` for today, etc.).

#### Extend Found Set (127)
```
  <Step enable="True" id="127" name="Extend Found Set">
    <Restore state="False"/>
  </Step>
```

#### Find Matching Records (155)
```
  <Step enable="True" id="155" name="Find Matching Records">
    <FindMatchingRecordsByField value="FindMatchingReplace"/>
  </Step>
```

#### Modify Last Find (24)
```
  <Step enable="True" id="24" name="Modify Last Find"/>
```

#### Omit Multiple Records (26)
```
  <Step enable="True" id="26" name="Omit Multiple Records">
    <NoInteract state="True"/>
  </Step>
```

#### Omit Record (25)
```
  <Step enable="True" id="25" name="Omit Record"/>
```

#### Perform Find (28)
```
  <Step enable="True" id="28" name="Perform Find">
    <Restore state="False"/>
  </Step>
```

With saved find criteria, the structure matches Constrain Found Set —
a `<Query>` containing one or more `<RequestRow>` elements. Multiple
`<Criteria>` within a single RequestRow combine with AND:
```
  <Step enable="True" id="28" name="Perform Find">
    <Restore state="True"/>
    <Query>
      <RequestRow operation="Include">
        <Criteria>
          <Field table="TableOccurrence" id="N" name="field_a"/>
          <Text>==$variable</Text>
        </Criteria>
        <Criteria>
          <Field table="TableOccurrence" id="N" name="field_b"/>
          <Text>*</Text>
        </Criteria>
      </RequestRow>
    </Query>
  </Step>
```

The `<Text>` element holds the find criterion as typed in find mode.
Common values: `*` (non-empty), `=` (empty), `==value` (exact match),
`//` (today), `>=value` (range). Comparison operators must be XML-
escaped inside `<Text>`: `>` becomes `&gt;`, `<` becomes `&lt;`,
`&` becomes `&amp;`. For example, `>=$earliest_date` is encoded as
`&gt;=$earliest_date`.

#### Perform Quick Find (150)
```
  <Step enable="True" id="150" name="Perform Quick Find"/>
```

#### Show All Records (23)
```
  <Step enable="True" id="23" name="Show All Records"/>
```

#### Show Omitted Only (27)
```
  <Step enable="True" id="27" name="Show Omitted Only"/>
```

#### Sort Records (39)
```
  <Step enable="True" id="39" name="Sort Records">
    <NoInteract state="True"/>
    <Restore state="False"/>
  </Step>
```

With a saved sort order, adds a `<SortList>` element. Each sort key
is a `<Sort>` element wrapping a `<PrimaryField>` which holds the
structured `<Field>` reference. Sort direction is the `type` attribute
on `<Sort>`. Multiple sort keys appear as multiple `<Sort>` elements
in priority order.
```
  <Step enable="True" id="39" name="Sort Records">
    <NoInteract state="True"/>
    <Restore state="True"/>
    <SortList Maintain="True" value="True">
      <Sort type="Ascending">
        <PrimaryField>
          <Field table="TableOccurrence" id="N" name="field_a"/>
        </PrimaryField>
      </Sort>
      <Sort type="Ascending">
        <PrimaryField>
          <Field table="TableOccurrence" id="N" name="field_b"/>
        </PrimaryField>
      </Sort>
    </SortList>
  </Step>
```

`<SortList>` attributes:

- `Maintain="True|False"` — corresponds to "Keep records in sorted
  order" in the dialog
- `value="True"` — meaning not yet established; observed as `True` in
  configured sorts

`<Sort>` `type` attribute observed values: `Ascending`. Other expected
values (`Descending`, custom value list ordering) have not yet been
round-tripped. The `<PrimaryField>` wrapper name suggests `<Sort>`
may also accept other child elements for value-list-based or
calculation-based sort keys; the structure for those is not yet
documented.

#### Sort Records by Field (154)
```
  <Step enable="True" id="154" name="Sort Records by Field">
    <SortRecordsByField value="SortAscending"/>
  </Step>
```

#### Unsort Records (21)
```
  <Step enable="True" id="21" name="Unsort Records"/>
```

---

### 8.8 Windows

#### Adjust Window (31)
```
  <Step enable="True" id="31" name="Adjust Window">
    <WindowState value="ResizeToFit"/>
  </Step>
```

#### Arrange All Windows (120)
```
  <Step enable="True" id="120" name="Arrange All Windows">
    <WindowArrangement value="TileHorizontally"/>
  </Step>
```

#### Close Window (121)
```
  <Step enable="True" id="121" name="Close Window">
    <LimitToWindowsOfCurrentFile state="True"/>
    <Window value="Current"/>
  </Step>
```

Close by name:
```
  <Step enable="True" id="121" name="Close Window">
    <LimitToWindowsOfCurrentFile state="True"/>
    <Window value="ByName"/>
    <Name>
      <Calculation><![CDATA["window name"]]></Calculation>
    </Name>
  </Step>
```

`<Window>` enumeration: `Current`, `ByName`.

#### Freeze Window (79)
```
  <Step enable="True" id="79" name="Freeze Window"/>
```

#### Move/Resize Window (119)
```
  <Step enable="True" id="119" name="Move/Resize Window">
    <LimitToWindowsOfCurrentFile state="True"/>
    <Window value="Current"/>
  </Step>
```

#### New Window (122)

Default minimal:
```
  <Step enable="True" id="122" name="New Window">
    <LayoutDestination value="CurrentLayout"/>
    <NewWndStyles Style="Document" Close="Yes" Minimize="Yes" Maximize="Yes" Resize="Yes" Styles="3606018"/>
  </Step>
```

Configured with window name, target layout, and extended window-style
options:
```
  <Step enable="True" id="122" name="New Window">
    <LayoutDestination value="SelectedLayout"/>
    <Name>
      <Calculation><![CDATA["window name"]]></Calculation>
    </Name>
    <NewWndStyles DimParentWindow="No" Toolbars="Yes" MenuBar="Yes" Style="Document" Close="Yes" Minimize="Yes" Maximize="Yes" Resize="Yes" Styles="1076299266"/>
    <Layout id="N" name="layout name"/>
  </Step>
```

`<NewWndStyles>` may carry additional attributes when window options
are configured: `DimParentWindow`, `Toolbars`, `MenuBar` are all
optional and appear when set in the dialog. The `Styles` numeric value
also changes when these options are toggled. See Appendix A.

#### Refresh Window (80)
```
  <Step enable="True" id="80" name="Refresh Window">
    <Option state="False"/>
    <FlushSQLData state="False"/>
  </Step>
```

#### Scroll Window (81)
```
  <Step enable="True" id="81" name="Scroll Window">
    <ScrollOperation value="Home"/>
  </Step>
```

#### Select Window (123)
```
  <Step enable="True" id="123" name="Select Window">
    <LimitToWindowsOfCurrentFile state="True"/>
    <Window value="Current"/>
  </Step>
```

#### Set Window Title (124)
```
  <Step enable="True" id="124" name="Set Window Title">
    <LimitToWindowsOfCurrentFile state="True"/>
    <Window value="Current"/>
  </Step>
```

#### Set Zoom Level (97)
```
  <Step enable="True" id="97" name="Set Zoom Level">
    <Lock state="False"/>
    <Zoom value="100"/>
  </Step>
```

#### Show/Hide Menubar (166)
```
  <Step enable="True" id="166" name="Show/Hide Menubar">
    <Lock state="False"/>
    <ShowHide value="Hide"/>
  </Step>
```

#### Show/Hide Text Ruler (92)
```
  <Step enable="True" id="92" name="Show/Hide Text Ruler">
    <ShowHide value="Show"/>
  </Step>
```

#### Show/Hide Toolbars (29)
```
  <Step enable="True" id="29" name="Show/Hide Toolbars">
    <IncludeEditRecordToolbar state="True"/>
    <Lock state="False"/>
    <ShowHide value="Hide"/>
  </Step>
```

#### View As (30)
```
  <Step enable="True" id="30" name="View As">
    <View value="Cycle"/>
  </Step>
```

---

### 8.9 Files

#### Close Data File (196)
```
  <Step enable="True" id="196" name="Close Data File"/>
```

#### Close File (34)
```
  <Step enable="True" id="34" name="Close File"/>
```

#### Convert File (139)
```
  <Step enable="True" id="139" name="Convert File">
    <NoInteract state="False"/>
    <Option state="False"/>
    <SkipIndexes state="False"/>
    <VerifySSLCertificates state="False"/>
  </Step>
```

#### Create Data File (190)
```
  <Step enable="True" id="190" name="Create Data File">
    <CreateDirectories state="True"/>
  </Step>
```

#### Delete File (197)
```
  <Step enable="True" id="197" name="Delete File"/>
```

#### Get File Exists (188)
```
  <Step enable="True" id="188" name="Get File Exists"/>
```

#### Get Data File Position (194)
```
  <Step enable="True" id="194" name="Get Data File Position"/>
```

#### Get File Size (189)
```
  <Step enable="True" id="189" name="Get File Size"/>
```

#### New File (82)
```
  <Step enable="True" id="82" name="New File"/>
```

#### Open Data File (191)
```
  <Step enable="True" id="191" name="Open Data File"/>
```

#### Open File (33)
```
  <Step enable="True" id="33" name="Open File">
    <Option state="False"/>
  </Step>
```

#### Print (43)
```
  <Step enable="True" id="43" name="Print">
    <NoInteract state="True"/>
    <Restore state="False"/>
  </Step>
```

#### Print Setup (42)
```
  <Step enable="True" id="42" name="Print Setup">
    <NoInteract state="True"/>
    <Restore state="False"/>
  </Step>
```

#### Read from Data File (193)
```
  <Step enable="True" id="193" name="Read from Data File">
    <DataSourceType value="3"/>
  </Step>
```

#### Recover File (95)
```
  <Step enable="True" id="95" name="Recover File">
    <NoInteract state="True"/>
  </Step>
```

#### Rename File (199)
```
  <Step enable="True" id="199" name="Rename File"/>
```

#### Save a Copy as (37)
```
  <Step enable="True" id="37" name="Save a Copy as">
    <CreateDirectories state="True"/>
    <AutoOpen state="False"/>
    <CreateEmail state="False"/>
    <SaveAsType value="Copy"/>
  </Step>
```

#### Save a Copy as XML (3)
```
  <Step enable="True" id="3" name="Save a Copy as XML">
    <Option state="False"/>
  </Step>
```

#### Set Data File Position (195)
```
  <Step enable="True" id="195" name="Set Data File Position"/>
```

#### Set Multi-User (84)
```
  <Step enable="True" id="84" name="Set Multi-User">
    <MultiUser value="True"/>
  </Step>
```

#### Set Use System Formats (94)
```
  <Step enable="True" id="94" name="Set Use System Formats">
    <Set state="True"/>
  </Step>
```

#### Write to Data File (192)
```
  <Step enable="True" id="192" name="Write to Data File">
    <AppendLineFeed state="True"/>
    <DataSourceType value="1"/>
  </Step>
```

---

### 8.10 Accounts

#### Add Account (134)
```
  <Step enable="True" id="134" name="Add Account">
    <ChgPwdOnNextLogin value="False"/>
    <AddAccount>
      <AccountType>FileMaker</AccountType>
    </AddAccount>
  </Step>
```

#### Change Password (83)
```
  <Step enable="True" id="83" name="Change Password">
    <NoInteract state="False"/>
  </Step>
```

#### Delete Account (135)
```
  <Step enable="True" id="135" name="Delete Account"/>
```

#### Enable Account (137)
```
  <Step enable="True" id="137" name="Enable Account">
    <AccountOperation value="Activate"/>
  </Step>
```

#### Re-Login (138)
```
  <Step enable="True" id="138" name="Re-Login">
    <NoInteract state="True"/>
  </Step>
```

With explicit credentials (each as a Calculation expression):
```
  <Step enable="True" id="138" name="Re-Login">
    <NoInteract state="True"/>
    <AccountName>
      <Calculation><![CDATA["account name"]]></Calculation>
    </AccountName>
    <Password>
      <Calculation><![CDATA["password expression"]]></Calculation>
    </Password>
  </Step>
```

Both `<AccountName>` and `<Password>` are calculation containers, so
the values can be field references, variables, or computed expressions
rather than hardcoded strings. Hardcoded credentials in the XML are
visible to anyone who can edit the script.

#### Reset Account Password (136)
```
  <Step enable="True" id="136" name="Reset Account Password">
    <ChgPwdOnNextLogin value="False"/>
  </Step>
```

---

### 8.11 AI

AI steps carry a distinctive sub-element naming the AI operation type
(for example `<LLMRequestWithTools>`, `<LLMSemanticFind>`). Minimal
skeletons show this sub-element in its empty form.

#### Configure AI Account (212)
```
  <Step enable="True" id="212" name="Configure AI Account">
    <LLMType value="ChatGPT"/>
    <SetLLMAccout/>
  </Step>
```

The element name `SetLLMAccout` is FileMaker's own output (missing the
`n` in `Account`). See Appendix B.

#### Configure Machine Learning Model (202)
```
  <Step enable="True" id="202" name="Configure Machine Learning Model">
    <ConfigureCoreML>Uninstall</ConfigureCoreML>
  </Step>
```

#### Configure Prompt Template (226)
```
  <Step enable="True" id="226" name="Configure Prompt Template">
    <Option state="False"/>
    <ConfigurePromptTemplate>
      <ModelProvider>ChatGPT</ModelProvider>
      <RequestType>SQLQuery</RequestType>
    </ConfigurePromptTemplate>
  </Step>
```

#### Configure RAG Account (227)
```
  <Step enable="True" id="227" name="Configure RAG Account ">
    <VerifySSLCertificates state="False"/>
    <ConfigureRAGAccount/>
  </Step>
```

The `name` attribute contains a trailing space (`"Configure RAG Account "`)
in FileMaker's native output. See Appendix B.

#### Configure Regression Model (222)
```
  <Step enable="True" id="222" name="Configure Regression Model">
    <LLMTrain>
      <LLMTrainAction>LLMTrainTrainModel</LLMTrainAction>
      <LLMAlgorithm>LLMTrainAlgForest</LLMAlgorithm>
    </LLMTrain>
  </Step>
```

#### Fine-Tune Model (213)
```
  <Step enable="True" id="213" name="Fine-Tune Model">
    <Option state="False"/>
    <UniversalPathList type="Embedded"/>
    <Table id="0" name=""/>
    <FineTuneLLM>
      <DataSource>DataTable</DataSource>
    </FineTuneLLM>
  </Step>
```

#### Generate Response from Model (220)
```
  <Step enable="True" id="220" name="Generate Response from Model">
    <Option state="False"/>
    <SelectAll state="False"/>
    <Stream state="False"/>
    <Set state="True"/>
    <LinkAvail state="False"/>
    <Restore state="False"/>
    <UniversalPathList type="Embedded"/>
    <LLMRequestWithTools/>
  </Step>
```

#### Insert Embedding (215)
```
  <Step enable="True" id="215" name="Insert Embedding">
    <LLMEmbedding/>
  </Step>
```

#### Insert Embedding in Found Set (216)
```
  <Step enable="True" id="216" name="Insert Embedding in Found Set">
    <LLMBulkEmbedding/>
  </Step>
```

#### Perform Find by Natural Language (221)
```
  <Step enable="True" id="221" name="Perform Find by Natural Language">
    <Option state="False"/>
    <SelectAll state="True"/>
    <LLMCreateFind>
      <Action>Query</Action>
    </LLMCreateFind>
  </Step>
```

#### Perform RAG Action (219)
```
  <Step enable="True" id="219" name="Perform RAG Action">
    <RAGSpace>
      <RAGSpaceAction>Add</RAGSpaceAction>
      <DataSource>FromText</DataSource>
    </RAGSpace>
  </Step>
```

#### Perform Semantic Find (218)
```
  <Step enable="True" id="218" name="Perform Semantic Find">
    <LLMSemanticFind>
      <Query type="1"/>
      <Records type="1"/>
    </LLMSemanticFind>
  </Step>
```

#### Perform SQL Query by Natural Language (214)
```
  <Step enable="True" id="214" name="Perform SQL Query by Natural Language">
    <Option state="False"/>
    <Stream state="False"/>
    <UniversalPathList type="Embedded"/>
    <PerformSQLQuerybyNaturalLanguage>
      <OptionsSelectionType>By List</OptionsSelectionType>
      <Action>Query</Action>
      <TablesSelectionType>By List</TablesSelectionType>
      <TableAliases/>
    </PerformSQLQuerybyNaturalLanguage>
  </Step>
```

#### Set AI Call Logging (217)
```
  <Step enable="True" id="217" name="Set AI Call Logging">
    <Set state="False"/>
    <LLMDebugLog/>
  </Step>
```

---

### 8.12 Spelling

#### Check Found Set (20)
```
  <Step enable="True" id="20" name="Check Found Set"/>
```

#### Check Record (19)
```
  <Step enable="True" id="19" name="Check Record"/>
```

#### Check Selection (18)
```
  <Step enable="True" id="18" name="Check Selection">
    <SelectAll state="True"/>
  </Step>
```

#### Correct Word (106)
```
  <Step enable="True" id="106" name="Correct Word"/>
```

#### Edit User Dictionary (109)
```
  <Step enable="True" id="109" name="Edit User Dictionary"/>
```

#### Select Dictionaries (108)
```
  <Step enable="True" id="108" name="Select Dictionaries"/>
```

#### Set Dictionary (209)
```
  <Step enable="True" id="209" name="Set Dictionary">
    <MainDictionary value="US English"/>
  </Step>
```

#### Spelling Options (107)
```
  <Step enable="True" id="107" name="Spelling Options"/>
```

---

### 8.13 Open Menu

These steps open built-in dialogs and take no parameters.

```
  <Step enable="True" id="149" name="Open Edit Saved Finds"/>
  <Step enable="True" id="183" name="Open Favorites"/>
  <Step enable="True" id="114" name="Open File Options"/>
  <Step enable="True" id="129" name="Open Find/Replace"/>
  <Step enable="True" id="32"  name="Open Help"/>
  <Step enable="True" id="118" name="Open Hosts"/>
  <Step enable="True" id="156" name="Open Manage Containers"/>
  <Step enable="True" id="140" name="Open Manage Data Sources"/>
  <Step enable="True" id="38"  name="Open Manage Database"/>
  <Step enable="True" id="151" name="Open Manage Layouts"/>
  <Step enable="True" id="165" name="Open Manage Themes"/>
  <Step enable="True" id="112" name="Open Manage Value Lists"/>
  <Step enable="True" id="88"  name="Open Script Workspace"/>
  <Step enable="True" id="105" name="Open Settings"/>
  <Step enable="True" id="113" name="Open Sharing"/>
  <Step enable="True" id="172" name="Open Upload to Host"/>
```

---

### 8.14 Miscellaneous

#### Allow Formatting Bar (115)
```
  <Step enable="True" id="115" name="Allow Formatting Bar">
    <Set state="False"/>
  </Step>
```

#### AVPlayer Play (177)
```
  <Step enable="True" id="177" name="AVPlayer Play">
    <Source value="Object"/>
  </Step>
```

#### AVPlayer Set Options (179)
```
  <Step enable="True" id="179" name="AVPlayer Set Options"/>
```

#### AVPlayer Set Playback State (178)
```
  <Step enable="True" id="178" name="AVPlayer Set Playback State">
    <PlaybackState value="Stopped"/>
  </Step>
```

#### Beep (93)
```
  <Step enable="True" id="93" name="Beep"/>
```

#### Dial Phone (65)
```
  <Step enable="True" id="65" name="Dial Phone">
    <NoInteract state="True"/>
  </Step>
```

#### Enable Touch Keyboard (174)
```
  <Step enable="True" id="174" name="Enable Touch Keyboard">
    <ShowHide value="Show"/>
  </Step>
```

#### Execute FileMaker Data API (203)
```
  <Step enable="True" id="203" name="Execute FileMaker Data API">
    <SelectAll state="True"/>
  </Step>
```

#### Execute SQL (117)
```
  <Step enable="True" id="117" name="Execute SQL">
    <NoInteract state="True"/>
  </Step>
```

#### Exit Application (44)
```
  <Step enable="True" id="44" name="Exit Application"/>
```

#### Flush Cache to Disk (102)
```
  <Step enable="True" id="102" name="Flush Cache to Disk"/>
```

#### Get Folder Path (181)
```
  <Step enable="True" id="181" name="Get Folder Path">
    <AllowFolderCreation state="False"/>
  </Step>
```

#### Install Menu Set (142)
```
  <Step enable="True" id="142" name="Install Menu Set">
    <UseAsFileDefault state="False"/>
    <CustomMenuSet id="1" name="[Standard FileMaker Menus]"/>
  </Step>
```

#### Install Plug-In File (157)
```
  <Step enable="True" id="157" name="Install Plug-In File"/>
```

#### Open URL (111)
```
  <Step enable="True" id="111" name="Open URL">
    <NoInteract state="True"/>
    <Option state="False"/>
  </Step>
```

#### Perform AppleScript (67)
```
  <Step enable="True" id="67" name="Perform AppleScript">
    <ContentType value="Text"/>
  </Step>
```

#### Perform JavaScript in Web Viewer (175)
```
  <Step enable="True" id="175" name="Perform JavaScript in Web Viewer"/>
```

Configured with object name, function name, and parameters. Element
order is fixed: `ObjectName` → `FunctionName` → optional `Parameters`.
Parameters use a non-obvious nested structure: a `<Parameters>`
wrapper with a `Count` attribute, containing one `<P>` child per
argument, each wrapping a `<Calculation>`.
```
  <Step enable="True" id="175" name="Perform JavaScript in Web Viewer">
    <ObjectName>
      <Calculation><![CDATA["myWebViewer"]]></Calculation>
    </ObjectName>
    <FunctionName>
      <Calculation><![CDATA["updateData"]]></Calculation>
    </FunctionName>
    <Parameters Count="3">
      <P>
        <Calculation><![CDATA["first arg"]]></Calculation>
      </P>
      <P>
        <Calculation><![CDATA[42]]></Calculation>
      </P>
      <P>
        <Calculation><![CDATA[$variable_arg]]></Calculation>
      </P>
    </Parameters>
  </Step>
```

Notes:

- The `Count` attribute on `<Parameters>` must match the number of
  `<P>` children. Whether FileMaker validates this on paste or trusts
  the count is not yet established.
- The `<P>` element name is the exact canonical form. Generators
  emitting `<Parameter>` (the longer, more obvious name) will have
  the entire parameter list silently dropped on paste — same failure
  class as the Set Variable `<Name>` issue. This is a verified
  silent-failure mode.
- Without parameters, omit the `<Parameters>` wrapper entirely.

#### Refresh Object (167)
```
  <Step enable="True" id="167" name="Refresh Object"/>
```

With object name:
```
  <Step enable="True" id="167" name="Refresh Object">
    <ObjectName>
      <Calculation><![CDATA["object name"]]></Calculation>
    </ObjectName>
  </Step>
```

#### Refresh Portal (180)
```
  <Step enable="True" id="180" name="Refresh Portal"/>
```

#### Save a Copy as Add-on Package (96)
```
  <Step enable="True" id="96" name="Save a Copy as Add-on Package">
    <LinkAvail state="False"/>
  </Step>
```

#### Send DDE Execute (64)
```
  <Step enable="True" id="64" name="Send DDE Execute">
    <ContentType value="File"/>
  </Step>
```

#### Send Event (57)
```
  <Step enable="True" id="57" name="Send Event">
    <ContentType value="File"/>
    <Event CopyResultToClipboard="False" WaitForCompletion="False" BringTargetToForeground="False"/>
  </Step>
```

#### Send Mail (63)
```
  <Step enable="True" id="63" name="Send Mail">
    <NoInteract state="True"/>
    <MultipleEmails state="False"/>
    <SendViaSMTP state="False"/>
    <SendViaOAuthAuthentication state="False"/>
    <SMTPEncryptionType type="SMTPEncryptionNone"/>
    <SMTPAuthenticationType type="SMTPAuthenticationNone"/>
    <OAuthProvider type="OAuthProviderGoogle"/>
  </Step>
```

#### Set Session Identifier (208)
```
  <Step enable="True" id="208" name="Set Session Identifier"/>
```

#### Set Web Viewer (146)

Default minimal:
```
  <Step enable="True" id="146" name="Set Web Viewer">
    <Action value="Reset"/>
  </Step>
```

`<Action>` enumeration values: `Reset`, `Reload`, `GoForward`,
`GoBack`, `GoToURL`. Element order is fixed: `Action` → optional
`ObjectName` → optional `URL`.

**Reset** — no target needed:
```
  <Step enable="True" id="146" name="Set Web Viewer">
    <Action value="Reset"/>
  </Step>
```

**Reload, GoForward, GoBack** — targets a Web Viewer object by name:
```
  <Step enable="True" id="146" name="Set Web Viewer">
    <Action value="Reload"/>
    <ObjectName>
      <Calculation><![CDATA["myWebViewer"]]></Calculation>
    </ObjectName>
  </Step>
```

**GoToURL** — adds a `<URL>` element with a `custom` attribute:
```
  <Step enable="True" id="146" name="Set Web Viewer">
    <Action value="GoToURL"/>
    <ObjectName>
      <Calculation><![CDATA["myWebViewer"]]></Calculation>
    </ObjectName>
    <URL custom="False">
      <Calculation><![CDATA["https://example.com/test"]]></Calculation>
    </URL>
  </Step>
```

The `custom="False"` attribute on `<URL>` corresponds to a toggle in
the Set Web Viewer dialog (likely "custom web address" versus a
preset URL source). Generators should emit `custom="False"` to match
FileMaker's canonical output. The `<URL>` element wraps a
`<Calculation>` for the URL expression.

#### Show Custom Dialog (87)
```
  <Step enable="True" id="87" name="Show Custom Dialog"/>
```

Configured steps emit children in fixed order: optional `<Title>` →
`<Message>` → `<Buttons>`. `<Title>` and `<Message>` each wrap a
`<Calculation>`. `<Buttons>` always contains exactly three `<Button>`
slots — unused slots are self-closing. `CommitState="True"` on a
button indicates that the button commits the pending record; it is
not a "default button" indicator.

**Single OK (alert) — OK commits:**
```
  <Step enable="True" id="87" name="Show Custom Dialog">
    <Message>
      <Calculation><![CDATA["message text"]]></Calculation>
    </Message>
    <Buttons>
      <Button CommitState="True">
        <Calculation><![CDATA["OK"]]></Calculation>
      </Button>
      <Button CommitState="False"/>
      <Button CommitState="False"/>
    </Buttons>
  </Step>
```

**With title (error or notice dialog):**
```
  <Step enable="True" id="87" name="Show Custom Dialog">
    <Title>
      <Calculation><![CDATA["Error"]]></Calculation>
    </Title>
    <Message>
      <Calculation><![CDATA["Unable to mount fileserver"]]></Calculation>
    </Message>
    <Buttons>
      <Button CommitState="False">
        <Calculation><![CDATA["OK"]]></Calculation>
      </Button>
      <Button CommitState="False"/>
      <Button CommitState="False"/>
    </Buttons>
  </Step>
```

**Two-button choice (neither commits):**
```
    <Buttons>
      <Button CommitState="False">
        <Calculation><![CDATA["No"]]></Calculation>
      </Button>
      <Button CommitState="False">
        <Calculation><![CDATA["Yes"]]></Calculation>
      </Button>
      <Button CommitState="False"/>
    </Buttons>
```

The clicked button is reported by `Get ( LastMessageChoice )` as
1, 2, or 3.

#### Speak (66)
```
  <Step enable="True" id="66" name="Speak">
    <SpeechOptions WaitForCompletion="True" VoiceId="0"/>
  </Step>
```

---

## 9. Plugin steps

Plugin steps have a distinct structure. The `<Step>` tag carries
`index` and `Source` attributes, and the body splits into a
`<PluginStep>` declaration (describing the plugin's parameter shape)
followed by `<ParameterValues>` containing indexed `<Object>` children
that hold the actual calculations.

### 9.1 MBS (186) — Monkeybread Plugin

`Source="MBSP"` identifies the plugin. `index="N"` is FileMaker's
internal plugin registration index, which is file-specific.

```
  <Step index="2" Source="MBSP" enable="True" id="186" name="MBS">
    <PluginStep>
      <Parameter ShowInLine="true" Label="Destination" Type="target"/>
      <Parameter ID="0" Label="Function" ShowInline="true" DataType="text" Type="calc"/>
      <Parameter ID="1" Label="P1" ShowInline="true" DataType="text" Type="calc"/>
      <Parameter ID="2" Label="P2" ShowInline="true" DataType="text" Type="calc"/>
      <Parameter ID="3" Label="P3" ShowInline="true" DataType="text" Type="calc"/>
      <Parameter ID="4" Label="P4" ShowInline="true" DataType="text" Type="calc"/>
      <Parameter ID="5" Label="P5" ShowInline="false" DataType="text" Type="calc"/>
      <Parameter ID="6" Label="P6" ShowInline="false" DataType="text" Type="calc"/>
      <Parameter ID="7" Label="P7" ShowInline="false" DataType="text" Type="calc"/>
      <Parameter ID="8" Label="P8" ShowInline="false" DataType="text" Type="calc"/>
      <Parameter ID="9" Label="P9" ShowInline="false" DataType="text" Type="calc"/>
    </PluginStep>
    <SelectAll state="True"/>
    <Field/>
    <ParameterValues>
      <Object index="0" type="Calc">
        <Calculation><![CDATA[]]></Calculation>
      </Object>
      <Object index="1" type="Calc">
        <Calculation><![CDATA[]]></Calculation>
      </Object>
      <Object index="2" type="Calc">
        <Calculation><![CDATA[]]></Calculation>
      </Object>
      <Object index="3" type="Calc">
        <Calculation><![CDATA[]]></Calculation>
      </Object>
      <Object index="4" type="Calc">
        <Calculation><![CDATA[]]></Calculation>
      </Object>
      <Object index="5" type="Calc">
        <Calculation><![CDATA[]]></Calculation>
      </Object>
      <Object index="6" type="Calc">
        <Calculation><![CDATA[]]></Calculation>
      </Object>
      <Object index="7" type="Calc">
        <Calculation><![CDATA[]]></Calculation>
      </Object>
      <Object index="8" type="Calc">
        <Calculation><![CDATA[]]></Calculation>
      </Object>
      <Object index="9" type="Calc">
        <Calculation><![CDATA[]]></Calculation>
      </Object>
    </ParameterValues>
  </Step>
```

Parameter slots:

- Index 0 holds the MBS function name (for example,
  `"WebView.RunJavaScript"`).
- Indices 1 through 9 hold function arguments P1 through P9.
- `<Field/>` is the destination target. It is empty when the result is
  assigned to a variable, and populated with
  `<Field table="X" id="N" name="y"/>` when the result is written to
  a field.
- `<SelectAll state="True"/>` controls whether the step replaces the
  entire field when the destination is a field.

When generating MBS steps, the `index` and `Source` attributes on
`<Step>` are required, and the full `<PluginStep>` declaration with
all ten parameter entries must be present even when most slots are
empty. Populated `<Object>` children contain the CDATA calculation for
the corresponding slot.

**Cross-file paste behaviour.** When MBS XML is pasted into a file
*without* the MBS plugin installed, FileMaker preserves both the
`Source` and `index` attributes verbatim and renders the step as a
"missing plug-in" placeholder displaying the source identifier and
index value (e.g. `<Unknown external script step from missing plug-in
( MBSP 0 )>`). Both attributes are retained even when the index is
implausible (`index="0"`, `index="999"`). Whether the `index`
attribute must match the recipient file's plugin registration order
when MBS *is* installed has not been verified — the structural form
is preserved either way, but the step's resolution to a working MBS
function is conditional on the plugin's presence and registration
state.

---

## 10. Worked example: generating a small script

This section walks through producing a short FileMaker script from
scratch using only the rules in this document. The goal is a script
that prompts the user for confirmation, performs a find for active
records, and reports the count.

The script in human-readable form:

```
Show Custom Dialog [ "Confirm" ; "Search active records?" ]
If [ Get ( LastMessageChoice ) = 2 ]
    Exit Script [ Text Result: "" ]
End If
Enter Find Mode [ Pause: Off ]
Set Field [ records::status ; "active" ]
Perform Find [ ]
Show Custom Dialog [ "Result" ; "Found " & Get ( FoundCount ) & " records" ]
```

Generating the XML, step by step:

**Step 1: the wrapper.** Every snippet begins with the XML declaration
and `fmxmlsnippet` element (Section 1):

```
<?xml version="1.0" encoding="UTF-8"?>
<fmxmlsnippet type="FMObjectList">
  ...steps go here...
</fmxmlsnippet>
```

**Step 2: Show Custom Dialog with title and message** (Section 8.14).
Three button slots are mandatory; unused slots are self-closing.
Yes/No choice means neither button commits:

```
  <Step enable="True" id="87" name="Show Custom Dialog">
    <Title>
      <Calculation><![CDATA["Confirm"]]></Calculation>
    </Title>
    <Message>
      <Calculation><![CDATA["Search active records?"]]></Calculation>
    </Message>
    <Buttons>
      <Button CommitState="False">
        <Calculation><![CDATA["No"]]></Calculation>
      </Button>
      <Button CommitState="False">
        <Calculation><![CDATA["Yes"]]></Calculation>
      </Button>
      <Button CommitState="False"/>
    </Buttons>
  </Step>
```

**Step 3: If / Exit Script / End If branch** (Section 8.1). The If
test reads `LastMessageChoice = 2`, which is the second button (No):

```
  <Step enable="True" id="68" name="If">
    <Restore state="False"/>
    <Calculation><![CDATA[Get ( LastMessageChoice ) = 2]]></Calculation>
  </Step>
  <Step enable="True" id="103" name="Exit Script">
    <Calculation><![CDATA[""]]></Calculation>
  </Step>
  <Step enable="True" id="70" name="End If"/>
```

**Step 4: Enter Find Mode** (Section 8.3):

```
  <Step enable="True" id="22" name="Enter Find Mode">
    <Pause state="False"/>
    <Restore state="False"/>
  </Step>
```

**Step 5: Set Field with structured field reference** (Section 8.5,
plus the field-vs-variable target convention in Section 5). The
`records::status` field needs a real `id` value from the recipient
file's DDR — placeholder `N` shown here:

```
  <Step enable="True" id="76" name="Set Field">
    <Calculation><![CDATA["active"]]></Calculation>
    <Field table="records" id="N" name="status"/>
  </Step>
```

**Step 6: Perform Find** with no saved criteria (it will find against
whatever criteria are entered in find mode by the previous Set Field):

```
  <Step enable="True" id="28" name="Perform Find">
    <Restore state="False"/>
  </Step>
```

**Step 7: Show Custom Dialog reporting the count.** Single OK button
with CommitState="True" (it commits the implicit pending record
state):

```
  <Step enable="True" id="87" name="Show Custom Dialog">
    <Title>
      <Calculation><![CDATA["Result"]]></Calculation>
    </Title>
    <Message>
      <Calculation><![CDATA["Found " & Get ( FoundCount ) & " records"]]></Calculation>
    </Message>
    <Buttons>
      <Button CommitState="True">
        <Calculation><![CDATA["OK"]]></Calculation>
      </Button>
      <Button CommitState="False"/>
      <Button CommitState="False"/>
    </Buttons>
  </Step>
```

**Step 8: assemble the wrapper around the steps.** The complete
output:

```
<?xml version="1.0" encoding="UTF-8"?>
<fmxmlsnippet type="FMObjectList">
  <Step enable="True" id="87" name="Show Custom Dialog">
    <Title>
      <Calculation><![CDATA["Confirm"]]></Calculation>
    </Title>
    <Message>
      <Calculation><![CDATA["Search active records?"]]></Calculation>
    </Message>
    <Buttons>
      <Button CommitState="False">
        <Calculation><![CDATA["No"]]></Calculation>
      </Button>
      <Button CommitState="False">
        <Calculation><![CDATA["Yes"]]></Calculation>
      </Button>
      <Button CommitState="False"/>
    </Buttons>
  </Step>
  <Step enable="True" id="68" name="If">
    <Restore state="False"/>
    <Calculation><![CDATA[Get ( LastMessageChoice ) = 2]]></Calculation>
  </Step>
  <Step enable="True" id="103" name="Exit Script">
    <Calculation><![CDATA[""]]></Calculation>
  </Step>
  <Step enable="True" id="70" name="End If"/>
  <Step enable="True" id="22" name="Enter Find Mode">
    <Pause state="False"/>
    <Restore state="False"/>
  </Step>
  <Step enable="True" id="76" name="Set Field">
    <Calculation><![CDATA["active"]]></Calculation>
    <Field table="records" id="N" name="status"/>
  </Step>
  <Step enable="True" id="28" name="Perform Find">
    <Restore state="False"/>
  </Step>
  <Step enable="True" id="87" name="Show Custom Dialog">
    <Title>
      <Calculation><![CDATA["Result"]]></Calculation>
    </Title>
    <Message>
      <Calculation><![CDATA["Found " & Get ( FoundCount ) & " records"]]></Calculation>
    </Message>
    <Buttons>
      <Button CommitState="True">
        <Calculation><![CDATA["OK"]]></Calculation>
      </Button>
      <Button CommitState="False"/>
      <Button CommitState="False"/>
    </Buttons>
  </Step>
</fmxmlsnippet>
```

**Notes on what could go wrong:**

- The `id="N"` for the `records::status` field is a placeholder. A
  real generator must either know the recipient file's DDR or accept
  that pasted scripts will display as missing references until the
  field is rebound. See Section 5 ("Runtime dependencies not enforced
  by the XML format").
- Two-space indentation must be used throughout. A single tab
  character anywhere in the output may trigger a silent failure on
  paste.
- The `Get ( FoundCount )` calculation in the final dialog runs at
  evaluation time, not at paste time — the dialog will report the
  current found count whenever the script runs.

---

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

### B.1 Configure AI Account — misspelled element

Step 212 emits `<SetLLMAccout/>` as its sub-element. The correct
spelling would be `SetLLMAccount`. The misspelled form is FileMaker's
native output and must be used for paste compatibility.

### B.2 Configure RAG Account — trailing space in step name

Step 227 emits `name="Configure RAG Account "` with a trailing space
inside the attribute value. The trailing space is part of FileMaker's
native output and must be preserved.

---


## 11. Custom Functions

### 11.1 Overview

Custom functions use the same `fmxmlsnippet type="FMObjectList"` wrapper as script steps. The paste target is the Manage Custom Functions dialog (not the Script Workspace). Multiple functions can be included in a single snippet and will all paste in one operation.

### 11.2 Canonical skeleton

**No parameters:**
```xml
<CustomFunction id="1" functionArity="0" visible="True" parameters="" name="CF_Name">
  <Calculation><![CDATA[expression]]></Calculation>
</CustomFunction>
```

**With parameters:**
```xml
<CustomFunction id="1" functionArity="2" visible="True" parameters="input;multiplier" name="CF_Name">
  <Calculation><![CDATA[input * multiplier]]></Calculation>
</CustomFunction>
```

### 11.3 Attributes

| Attribute | Values | Notes |
|---|---|---|
| `id` | any integer | Ignored on paste — FileMaker assigns its own sequential ID. Use `id="1"` as placeholder. |
| `functionArity` | integer | Must match the number of parameters exactly. |
| `visible` | `"True"` / `"False"` | `"True"` = All accounts. `"False"` = Full access accounts only. |
| `parameters` | string | Semicolon-separated parameter names. Empty string `""` for zero parameters. |
| `name` | string | If a CF with this name already exists in the target file, FileMaker pastes it as a new CF with an auto-incremented name suffix (e.g. `CF_Test 2`). It does not overwrite. |

### 11.4 Parameters

Parameters are semicolon-separated in the `parameters` attribute — not comma-separated:

```xml
<CustomFunction id="1" functionArity="3" visible="True" parameters="input;multiplier;offset" name="CF_Name">
```

`functionArity` must equal the parameter count exactly.

### 11.5 Recursion

Recursive calls are plain text inside the CDATA body — no special element or attribute required:

```xml
<CustomFunction id="1" functionArity="1" visible="True" parameters="n" name="CF_Factorial">
  <Calculation><![CDATA[If ( n <= 1 ; 1 ; n * CF_Factorial ( n - 1 ) )]]></Calculation>
</CustomFunction>
```

### 11.6 Calculation body

The `<Calculation>` child follows the same CDATA convention as script steps. Multi-line bodies, comments (`//` and `/* ... */`), and fully commented-out bodies all round-trip intact. FileMaker treats the body as opaque text.

**Commented-out body (verified round-trip):**
```xml
<CustomFunction id="1" functionArity="1" visible="True" parameters="text" name="CF_Name">
  <Calculation><![CDATA[/* entire body commented out
for later use */]]></Calculation>
</CustomFunction>
```

### 11.7 Schema references in the body

Custom function bodies are opaque CDATA — the paste handler performs no schema resolution on field references, table names, or calls to other custom functions within the body. A function referencing `MyTable::MyField` or calling `AnotherCF()` will paste successfully into any file regardless of whether that schema or function exists. Runtime errors surface only when the function is evaluated, not at paste time.

This is a significant difference from script steps, where field and layout references are structured XML elements that FileMaker resolves against the recipient schema on paste.

### 11.8 Multiple functions in one snippet

Any number of `<CustomFunction>` elements may appear in a single snippet. All paste in one operation:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<fmxmlsnippet type="FMObjectList">
  <CustomFunction id="1" functionArity="1" visible="True" parameters="n" name="CF_Factorial">
    <Calculation><![CDATA[If ( n <= 1 ; 1 ; n * CF_Factorial ( n - 1 ) )]]></Calculation>
  </CustomFunction>
  <CustomFunction id="2" functionArity="0" visible="True" parameters="" name="CF_Today">
    <Calculation><![CDATA[Get ( CurrentDate )]]></Calculation>
  </CustomFunction>
</fmxmlsnippet>
```

Round-trip verified at scale: 24 functions pasted in a single snippet, all IDs reassigned sequentially from 1.

### 11.9 Known behaviour and limitations

- FileMaker ignores the `id` attribute on paste and assigns its own sequential IDs.
- Name conflicts result in a new CF being created with an auto-incremented suffix — not an overwrite.
- No silent-failure modes identified. The format is substantially simpler than script steps with no element-ordering traps.
- `functionArity` mismatch (value does not match actual parameter count) — behaviour untested; avoid.
- Add-on and cross-file field references in the body will paste without error but will fail at evaluation time if the referenced schema is not present in the target file.
