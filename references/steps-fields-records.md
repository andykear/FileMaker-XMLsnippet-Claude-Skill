# Step Reference — Fields, Records and Found Sets (§8.5–8.7)

Part of the Canonical XML Format for FileMaker Script Steps, v1.12.
Read `core.md` first: the paste format requirements, conventions and
silent failure modes there apply to every step below.

### 8.5 Fields

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
| Set Field By Name | 147 |

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
      <LightMode choice="Auto"/>
    </DeviceOptions>
  </Step>
```

FM 26 adds `<LightMode>` inside `<DeviceOptions>` for controlling
the device flash. Enumeration: `Auto`, `On`, `Off`.

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

FM 26 adds `PerformAutoEnter` and `UseEntryOptions` attributes to
`<SerialNumbers>`. The `increment` and `InitialValue` attributes are
only present in serial number replacement mode; they may be absent in
other modes.

`<With>` values: `None`, `CurrentContents` (FM 26). Other expected
values for serial number and calculation modes not yet documented.

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

#### Set Next Serial Value (116)
```
  <Step enable="True" id="116" name="Set Next Serial Value"/>
```

---

### 8.6 Records

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
| Copy All Records/Requests | 98 |
| Copy Record/Request | 101 |
| Duplicate Record/Request | 8 |
| New Record/Request | 7 |
| Open Record/Request | 133 |

#### Commit Records/Requests (75)
```
  <Step enable="True" id="75" name="Commit Records/Requests">
    <NoInteract state="True"/>
    <Option state="False"/>
    <ESSForceCommit state="False"/>
  </Step>
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

Now also part of the "PDF Files" category in FM 26 Script Workspace.
See §8.15 (`steps-pdf.md`) for the related Create/Open/Append/Close/
Cancel/Print PDF steps.

Unconfigured (FM 26):
```
  <Step enable="True" id="144" name="Save Records as PDF">
    <NoInteract state="True"/>
    <Option state="False"/>
    <CreateDirectories state="True"/>
    <Restore state="False"/>
    <AutoOpen state="False"/>
    <CreateEmail state="False"/>
    <PDFOptions source="RecordsBeingBrowsed">
      <PDFSaveType>File</PDFSaveType>
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

FM 26 adds `<PDFSaveType>` inside `<PDFOptions>` (all round-trip
verified):

| PDFSaveType | Save to mode | Target elements |
|-------------|-------------|-----------------|
| `File` | File path | `<UniversalPathList>` as Step child |
| `Target` | Container field or variable | `<Text/>` + `<Field>` as Step children |
| `Append` | Currently open PDF | No path/target elements; requires prior Create PDF or Open PDF |

`<PDFOptions>` attributes (all round-trip verified):

| Attribute | Values | Notes |
|-----------|--------|-------|
| `source` | `RecordsBeingBrowsed`, `CurrentRecord`, `BlankRecord` | |
| `appearance` | `AsFormatted`, `WithBoxes`, `WithUnderlines`, `WithPlaceholderText` | BlankRecord only; absent for other source values |

Configured with Specify options (round-trip verified):
```
  <Step enable="True" id="144" name="Save Records as PDF">
    <NoInteract state="True"/>
    <Option state="True"/>
    <CreateDirectories state="True"/>
    <Restore state="True"/>
    <AutoOpen state="False"/>
    <CreateEmail state="False"/>
    <UniversalPathList>file:"filename.pdf"</UniversalPathList>
    <Calculation><![CDATA[1]]></Calculation>
    <PDFOptions source="RecordsBeingBrowsed">
      <PDFSaveType>File</PDFSaveType>
      <Document>
        <Title>
          <Calculation><![CDATA[title]]></Calculation>
        </Title>
        <Subject>
          <Calculation><![CDATA[subject]]></Calculation>
        </Subject>
        <Author>
          <Calculation><![CDATA[author]]></Calculation>
        </Author>
        <Keywords>
          <Calculation><![CDATA[keywords]]></Calculation>
        </Keywords>
        <Pages AllPages="True">
          <NumberFrom>
            <Calculation><![CDATA[1]]></Calculation>
          </NumberFrom>
        </Pages>
      </Document>
      <Security allowScreenReader="True" enableCopying="True" controlEditing="InsertingDeletingRotatingPages" controlPrinting="LowResolution" requireControlEditPassword="True" requireOpenPassword="True">
        <OpenPassword>
          <Calculation><![CDATA[password]]></Calculation>
        </OpenPassword>
        <ControlPassword>
          <Calculation><![CDATA[password]]></Calculation>
        </ControlPassword>
      </Security>
      <View magnification="100" pageLayout="SinglePage" show="PagesPanelAndPage"/>
    </PDFOptions>
  </Step>
```

Notes:
- Bare `<Calculation>` between `<UniversalPathList>` and `<PDFOptions>` appears when Restore is True with options configured.
- `<PageRange>` drops out when configured with `AllPages="True"` and Restore.
- When Append mode, Document and Initial View settings are ignored at runtime (but present in XML).

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
| Modify Last Find | 24 |
| Omit Record | 25 |
| Perform Quick Find | 150 |
| Show All Records | 23 |
| Show Omitted Only | 27 |

#### Constrain Found Set (126)
```
  <Step enable="True" id="126" name="Constrain Found Set">
    <Option state="False"/>
    <Restore state="False"/>
  </Step>
```

FM 26 adds "Find without indexes" option. In the XML this is
`<Option state="True"/>` — the existing element toggled. No new
element. May improve performance on small found sets with indexed
criteria fields.

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

#### Omit Multiple Records (26)
```
  <Step enable="True" id="26" name="Omit Multiple Records">
    <NoInteract state="True"/>
  </Step>
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

