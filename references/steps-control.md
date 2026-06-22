# Step Reference — Control (§8.1)

Part of the Canonical XML Format for FileMaker Script Steps, v1.12.
Read `core.md` first: the paste format requirements, conventions and
silent failure modes there apply to every step below.

### 8.1 Control

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
| End If | 70 |
| End Loop | 73 |
| Halt Script | 90 |
| Commit Transaction | 206 |

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

