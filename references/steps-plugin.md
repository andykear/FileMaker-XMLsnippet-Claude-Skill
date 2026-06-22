# Step Reference — Plugin Steps (§9)

Part of the Canonical XML Format for FileMaker Script Steps, v1.12.
Read `core.md` first: the paste format requirements, conventions and
silent failure modes there apply to every step below.

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
