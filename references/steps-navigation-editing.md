# Step Reference — Navigation and Editing (§8.3–8.4)

Part of the Canonical XML Format for FileMaker Script Steps, v1.13.
Read `core.md` first: the paste format requirements, conventions and
silent failure modes there apply to every step below.
In the captures below a `state` attribute is the raw stored bit, so
`<NoInteract state="True"/>` is "With dialog: Off" (core.md §5).

### 8.3 Navigation

#### No-option steps

Each step in this table takes no options. The canonical form
for every one of them is the single self-closing line:

```
  <Step enable="True" id="NN" name="Step Name"/>
```

with `id` and `name` exactly as listed (names are verbatim,
including any spacing):

| Step | id |
|---|---|
| Close Popover | 169 |
| Go to Next Field | 4 |
| Go to Previous Field | 5 |

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

**False lead, recorded to prevent rediscovery.** Using a name that
matches an existing table occurrence (rather than an actual layout
name) resolved and saved correctly in one test file — but only
because that specific file happened to already have a layout with a
matching name. Not a general guarantee about how `<Layout>` resolves;
`name` still needs to match a real layout in the target file.

#### Go to List of Records (228)
```
  <Step enable="True" id="228" name="Go to List of Records">
    <ShowInNewWindow state="False"/>
    <LayoutDestination value="CurrentLayout"/>
    <NewWndStyles Style="Document" Close="Yes" Minimize="Yes" Maximize="Yes" Resize="Yes" Styles="3606018"/>
  </Step>
```

`[26.1+]` Behaviour change in 26.1.1: the step now sorts records in the order of
the record IDs in the list; before 26.1.1 it didn't (FileMaker 26.1.1
release notes). This is a runtime fix; nothing in the release notes
points to a change in the step's XML.

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
FM 26 native Copy output also places `<DisableStepCollapsed>` between
`ShowInNewWindow` and `Restore` (core.md §6.0; optional on paste).

`<NewWndStyles>` is always present, regardless of new-window state.

Element semantics — mapped on FileMaker Pro 26.1 by matching each copied
element against the Script Workspace's text for the same step (eight
test steps), and checked against a GTRR from a production script:

- `<Restore state="True|False"/>` — **"Show only related records"** in
  the GTRR dialog. `True` creates a new found set of the related
  records. When it is off, Go to Related Record makes the first record
  in the table's unsorted order the current record (Claris Help).
  This is the most consequential GTRR option and the most commonly
  set; it is `False` in the unconfigured XML emission. Generators that
  get it wrong silently produce the wrong found-set behaviour at
  runtime — the step pastes and runs, just on the wrong record set.
- `<MatchAllRecords state="True|False"/>` — "Match found set" in the
  dialog. `True` matches against the entire found set on the source
  side; `False` matches only the current record. Almost always
  `False` in practice.
- `<ShowInNewWindow state="True|False"/>` — controls the new-window
  branch. When `True`, the optional `<Name>` (window-name calculation)
  and dimension elements may appear before `<NewWndStyles>`.
- `<Option state="True|False"/>` — **"Use external table's layouts"**.
  `True` only when the related table occurrence comes from another file
  and the step shows the records on a layout in that file; the Script
  Workspace then shows `External;`. `False` everywhere else, including
  an external table shown on a layout in this file. Claris: the option
  is only available when the source relationship references a table in
  another file.

Earlier versions of this specification put "Show only related records"
on `<Option>` and described `<Restore>` as stored sort/find settings;
both were wrong. The `<Option>`/`<Restore>` states in the examples below
agree with 26.1 output.

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

**This is a real, genuine export state — not a valid generation
template.** It's what an actual unconfigured GTRR step looks like when
copied natively, useful as a reference for recognizing one, but a
step generated in this shape does not save. Use the Basic form above
for anything actually meant to be pasted and used; this one is
documentation of what "not yet set up" looks like, not something to
generate on purpose.

**Table by calculation `[26.1+]`** (captured from FileMaker Pro 26.1's
Copy output; pasting it has not been tested). The "Get related record from" option can name the table
occurrence with a calculation. `<Table>` then has `id="0" name=""` and
wraps a `<Calculation>`, and the layout is either the current one or a
layout chosen by calculation:
```
  <Step enable="True" id="74" name="Go to Related Record">
    <Option state="False"/>
    <MatchAllRecords state="False"/>
    <ShowInNewWindow state="False"/>
    <Restore state="True"/>
    <LayoutDestination value="LayoutNameByCalc"/>
    <NewWndStyles Style="Document" Close="Yes" Minimize="Yes" Maximize="Yes" Resize="Yes" Styles="3606018"/>
    <Table id="0" name="">
      <Calculation><![CDATA["table occurrence name"]]></Calculation>
    </Table>
    <Layout>
      <Calculation><![CDATA["layout name"]]></Calculation>
    </Layout>
  </Step>
```

`<LayoutDestination>` values with a calculated table:
- `CurrentLayout` — no `<Layout>` element.
- `LayoutNameByCalc` — `<Layout>` wraps a `<Calculation>` returning the
  layout name, with no `id` or `name` attribute.
- `LayoutNumberByCalc` — the same `<Layout>` shape, the calculation
  returning a layout number.

The Script Workspace shows these as `From table: "T" ; Using layout:
<Current Layout>` (or the calculated name or number). A 26.0.x client
has no table-by-calculation option; do not emit this form for 26.0.

---

### 8.4 Editing

#### No-option steps

Each step in this table takes no options. The canonical form
for every one of them is the single self-closing line:

```
  <Step enable="True" id="NN" name="Step Name"/>
```

with `id` and `name` exactly as listed (names are verbatim,
including any spacing):

| Step | id |
|---|---|
| Select All | 50 |
| Set Selection | 130 |

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

#### Undo/Redo (45)
```
  <Step enable="True" id="45" name="Undo/Redo">
    <UndoRedo value="Undo"/>
  </Step>
```

---

