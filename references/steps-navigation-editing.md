# Step Reference — Navigation and Editing (§8.3–8.4)

Part of the Canonical XML Format for FileMaker Script Steps, v1.12.
Read `core.md` first: the paste format requirements, conventions and
silent failure modes there apply to every step below.

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

#### Go to List of Records (228)
```
  <Step enable="True" id="228" name="Go to List of Records">
    <ShowInNewWindow state="False"/>
    <LayoutDestination value="CurrentLayout"/>
    <NewWndStyles Style="Document" Close="Yes" Minimize="Yes" Maximize="Yes" Resize="Yes" Styles="3606018"/>
  </Step>
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

