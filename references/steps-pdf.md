# Step Reference — PDF Files (§8.15)

Part of the Canonical XML Format for FileMaker Script Steps, v1.12.
Read `core.md` first: the paste format requirements, conventions and
silent failure modes there apply to every step below.

### 8.15 PDF Files

New in FileMaker 2026. These steps provide a scriptable PDF assembly
workflow: create or open a PDF in memory, add pages from layouts or
existing PDFs, then save or discard.

Only one PDF can be open at a time. The in-memory PDF persists until
closed by Close PDF or discarded by Cancel PDF. Unsaved changes are
lost if the client session ends or the file that opened the PDF
closes.

Save Records as PDF (144) is also part of this category in FM 26 and
can target the currently open PDF via `<PDFSaveType>`. See §8.6 for
its skeleton; FM 26 changes are noted in §8.6.

#### Create PDF (243)

Creates an empty PDF in memory. Configure document properties,
security, and initial view via Specify options (same PDF Options
dialog structure as Save Records as PDF).

Error 833 if a PDF is already open.

Unconfigured:
```
  <Step enable="True" id="243" name="Create PDF">
    <Restore state="False"/>
    <CreatePDFFile>
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
    </CreatePDFFile>
  </Step>
```

Configured with Specify options (round-trip verified):
```
  <Step enable="True" id="243" name="Create PDF">
    <Restore state="True"/>
    <Calculation><![CDATA[1]]></Calculation>
    <CreatePDFFile>
      <Document>
        <Title>
          <Calculation><![CDATA[title expression]]></Calculation>
        </Title>
        <Subject>
          <Calculation><![CDATA[subject expression]]></Calculation>
        </Subject>
        <Author>
          <Calculation><![CDATA[author expression]]></Calculation>
        </Author>
        <Keywords>
          <Calculation><![CDATA[keywords expression]]></Calculation>
        </Keywords>
        <Pages AllPages="True">
          <NumberFrom>
            <Calculation><![CDATA[1]]></Calculation>
          </NumberFrom>
        </Pages>
      </Document>
      <Security allowScreenReader="True" enableCopying="True" controlEditing="InsertingDeletingRotatingPages" controlPrinting="LowResolution" requireControlEditPassword="True" requireOpenPassword="True">
        <OpenPassword>
          <Calculation><![CDATA[password expression]]></Calculation>
        </OpenPassword>
        <ControlPassword>
          <Calculation><![CDATA[password expression]]></Calculation>
        </ControlPassword>
      </Security>
      <View magnification="100" pageLayout="SinglePage" show="PagesPanelAndPage"/>
    </CreatePDFFile>
  </Step>
```

Notes:
- `<Restore state="True"/>` when Specify options is configured.
- Bare `<Calculation>` between `<Restore>` and `<CreatePDFFile>` — appears when Restore is True. Purpose not confirmed; observed in both Create PDF and Save Records as PDF.
- `<PageRange>` drops out when `AllPages="True"` with Restore configured.
- `<CreatePDFFile>` wraps the same Document/Security/View structure as `<PDFOptions>` in Save Records as PDF.
- `controlEditing` values: `AnyExceptExtractingPages`, `InsertingDeletingRotatingPages`.
- `controlPrinting` values: `HighResolution`, `LowResolution`.
- `<OpenPassword>` and `<ControlPassword>` are children of `<Security>`, not `<Document>`.

#### Open PDF (246)

Opens an existing PDF file in memory for modification.

Options: `From` (File or Source), `Source file` (file path) or
`Source` (container/variable), `Password` for encrypted PDFs.

Errors: 831 (invalid password), 833 (PDF already open).
WebDirect: From: File not supported.

Unconfigured:
```
  <Step enable="True" id="246" name="Open PDF">
    <Option state="False"/>
    <OpenPDFFile>
      <PDFSaveType>File</PDFSaveType>
    </OpenPDFFile>
  </Step>
```

Configured with file path and password (round-trip verified):
```
  <Step enable="True" id="246" name="Open PDF">
    <Option state="True"/>
    <UniversalPathList>$variable_or_path</UniversalPathList>
    <OpenPDFFile>
      <PDFSaveType>File</PDFSaveType>
      <OpenPassword>
        <Calculation><![CDATA[password expression]]></Calculation>
      </OpenPassword>
    </OpenPDFFile>
  </Step>
```

Notes:
- `<Option state="True"/>` when From is configured.
- `<UniversalPathList>` is a direct `<Step>` child, not inside `<OpenPDFFile>`. File paths can use `image:` prefix or be a bare variable name.
- `<PDFSaveType>` maps to the From option. Observed value: `File`.
- `<OpenPassword>` is inside `<OpenPDFFile>`, with a Calculation child.
- Element order: `Option` → `UniversalPathList` → `OpenPDFFile` (containing `PDFSaveType` → `OpenPassword`).

#### Append PDF (244)

Appends all pages from a source PDF to the currently open in-memory
PDF. The source PDF is not modified.

Options: `From` (File or Source), `Source file` (file path) or
`Source` (container/variable), `Password` for encrypted source PDFs.

Error 829 if no PDF is open. Error 831 if password is wrong.

Unconfigured:
```
  <Step enable="True" id="244" name="Append PDF">
    <Option state="False"/>
    <AppendPDFFile>
      <PDFSaveType>File</PDFSaveType>
    </AppendPDFFile>
  </Step>
```

Configured with file path and password (round-trip verified):
```
  <Step enable="True" id="244" name="Append PDF">
    <Option state="True"/>
    <UniversalPathList>image:../path/to/file.pdf</UniversalPathList>
    <AppendPDFFile>
      <PDFSaveType>File</PDFSaveType>
      <OpenPassword>
        <Calculation><![CDATA[password expression]]></Calculation>
      </OpenPassword>
    </AppendPDFFile>
  </Step>
```

Notes:
- Identical structure to Open PDF. Same element order, same `<OpenPassword>` inside the wrapper.
- Source PDF file paths use `image:` prefix (reads an existing file), unlike Close PDF which uses `file:` prefix (writes a file).

#### Print PDF (242)

Prints a PDF directly from a file path, container, or variable
without displaying it on a layout.

Options: `From` (File or Source), `Source file` / `Source`,
`Password`, `With dialog`, `Specify print options` (Restore),
`Save print options to` (container/variable),
`Use print options from` (container/variable).

Errors: 605 (empty container), 606 (not a PDF), 607 (wrong password),
608 (security prevents printing).

Not supported on Server, Cloud, Data API, or CWP.

Unconfigured:
```
  <Step enable="True" id="242" name="Print PDF">
    <NoInteract state="True"/>
    <Option state="False"/>
    <UsePrintOptionsFromEnabled state="False"/>
    <SavePrintOptionsToEnabled state="False"/>
    <Restore state="False"/>
  </Step>
```

Configured (round-trip verified). Structure shown with print settings
blobs omitted for clarity:
```
  <Step enable="True" id="242" name="Print PDF">
    <NoInteract state="True"/>
    <Option state="True"/>
    <UsePrintOptionsFromEnabled state="False"/>
    <SavePrintOptionsToEnabled state="True"/>
    <Restore state="True"/>
    <Text/>
    <Field table="TableOccurrence" id="N" name="field_name"/>
    <PrintSettings PageNumberingOffset="0" PrintToFile="False" ToPage="1" FromPage="1" AllPages="False" collated="True" NumCopies="1" PrintType="BrowsedRecords">
      <PlatformData PlatformType="PrNm"><![CDATA[...hex...]]></PlatformData>
      <PlatformData PlatformType="M_PM"><![CDATA[...hex...]]></PlatformData>
      <PlatformData PlatformType="M_PD"><![CDATA[...hex...]]></PlatformData>
      <PlatformData PlatformType="MMod"><![CDATA[...hex...]]></PlatformData>
      <SavePrintOptionsTo>
        <Field>$variable</Field>
      </SavePrintOptionsTo>
      <Password>
        <Calculation><![CDATA[password expression]]></Calculation>
      </Password>
      <SaveType>File</SaveType>
    </PrintSettings>
  </Step>
```

Notes:
- `<PrintSettings>` contains opaque `<PlatformData>` blobs (hex-encoded macOS print plist). Generators should not produce these; they are captured from the Print dialog.
- `<SavePrintOptionsTo>` wraps a `<Field>` element. Variable targets use plain text: `<Field>$variable</Field>`.
- Password is `<Password>` (not `<OpenPassword>`), inside `<PrintSettings>`.
- `<SaveType>` inside `<PrintSettings>`: observed values `File`, `Source`.
- `<Text/>` may appear as an empty element when From is Source mode.
- Element order at Step level: `NoInteract` → `Option` → `UsePrintOptionsFromEnabled` → `SavePrintOptionsToEnabled` → `Restore` → `Text` → `Field` → `PrintSettings`.

#### Close PDF (245)

Closes the in-memory PDF and saves it to a destination.

Options: `Save to` (File or Target).
For File: output file path, `Automatically open file`,
`Create email with file as attachment`, `Create folders`.
For Target: container field or variable, optional `Filename`.

Error 5 if no PDF is open. Overwrites existing files without error.
WebDirect: Save to File not supported.

Unconfigured:
```
  <Step enable="True" id="245" name="Close PDF">
    <CreateDirectories state="False"/>
    <AutoOpen state="False"/>
    <CreateEmail state="False"/>
    <ClosePDFFile>
      <PDFSaveType>File</PDFSaveType>
    </ClosePDFFile>
  </Step>
```

Configured with file path (round-trip verified):
```
  <Step enable="True" id="245" name="Close PDF">
    <CreateDirectories state="False"/>
    <AutoOpen state="True"/>
    <CreateEmail state="True"/>
    <UniversalPathList>file:"filename.pdf"</UniversalPathList>
    <ClosePDFFile>
      <PDFSaveType>File</PDFSaveType>
    </ClosePDFFile>
  </Step>
```

Notes:
- `<UniversalPathList>` is a direct `<Step>` child. File output paths use `file:` prefix.
- `<CreateDirectories>`, `<AutoOpen>`, `<CreateEmail>` are File-mode options.
- Element order: `CreateDirectories` → `AutoOpen` → `CreateEmail` → `UniversalPathList` → `ClosePDFFile`.

#### Cancel PDF (247)

Discards the in-memory PDF without saving. No options.

```
  <Step enable="True" id="247" name="Cancel PDF"/>
```

Self-closing, no children. No configured variant. Round-trip
verified.

---
