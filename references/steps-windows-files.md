# Step Reference — Windows and Files (§8.8–8.9)

Part of the Canonical XML Format for FileMaker Script Steps, v1.12.
Read `core.md` first: the paste format requirements, conventions and
silent failure modes there apply to every step below.

### 8.8 Windows

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
| Freeze Window | 79 |

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

FM 26 Custom calculation (round-trip verified):
```
  <Step enable="True" id="97" name="Set Zoom Level">
    <Lock state="False"/>
    <Zoom value="ByCalculation"/>
    <Calculation><![CDATA[zoom percentage expression]]></Calculation>
  </Step>
```

`<Zoom>` enumeration: `100`, `75`, `50`, `25`, `150`, `200`, `300`,
`400`, `ByCalculation`. Custom takes an integer from 25 to 400.

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
| Close Data File | 196 |
| Close File | 34 |
| Delete File | 197 |
| Get File Exists | 188 |
| Get Data File Position | 194 |
| Get File Size | 189 |
| New File | 82 |
| Open Data File | 191 |
| Rename File | 199 |
| Set Data File Position | 195 |

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

v1.11 skeleton (pre-FM 26):
```
  <Step enable="True" id="3" name="Save a Copy as XML">
    <Option state="False"/>
  </Step>
```

FM 26 expanded (from native Copy output):
```
  <Step enable="True" id="3" name="Save a Copy as XML">
    <Option state="False"/>
    <OutputEntireBinaryData state="False"/>
    <SpecifyJSONOptions state="False"/>
    <SaXML>
      <JSONOptions>
        <Calculation><![CDATA[JSONSetElement ( "{}" ;
  [ "catalogs_included" ; JSONMakeArray (
  "PersistentStoreCatalog
  BaseDirectoryCatalog
  FileAccessCatalog
  ExternalDataSourceCatalog
  BaseTableCatalog
  TableOccurrenceCatalog
  CustomFunctionsCatalog
  ValueListCatalog
  FieldCatalog
  RelationshipCatalog
  CustomMenuCatalog
  CustomMenuSetCatalog
  ScriptCatalog
  ThemeCatalog
  LayoutCatalog
  LibraryCatalog
  PrivilegeSetsCatalog
  ExtendedPrivilegesCatalog
  AccountsCatalog
  Metadata"
  ; " " ; JSONString ) ; JSONArray ] ;
  [ "include_details" ; False ; JSONBoolean ] ;
  [ "split_catalogs" ; False ; JSONBoolean ] ;
  [ "standalone_binarydata" ; False ; JSONBoolean ]
)]]></Calculation>
      </JSONOptions>
    </SaXML>
  </Step>
```

FM 26 new elements:
- `<OutputEntireBinaryData>` — include full binary data for layout
  objects.
- `<SpecifyJSONOptions>` — whether JSON catalog selection is
  configured.
- `<SaXML>` → `<JSONOptions>` → `<Calculation>` — catalog
  configuration as a JSONSetElement expression.

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

