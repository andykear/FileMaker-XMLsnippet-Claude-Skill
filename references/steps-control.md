# Step Reference — Control (§8.1)

Part of the Canonical XML Format for FileMaker Script Steps, v1.13.
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

**Structural rule, not a skeleton issue — see core.md §8.1.** Both
`Commit Transaction` (206) and `Revert Transaction` (207) above are
individually correct as documented, but neither one is save-valid if
nested inside an `If`/`Else`/`End If` block, with or without `Open
Transaction` present. This is a common pattern for a conditional
commit-or-revert and it fails at save time, not paste time. Read
core.md §8.1 before generating a script that branches to a commit or
revert.

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

With a parameter (same file):
```
  <Step enable="True" id="1" name="Perform Script">
    <Calculation><![CDATA[parameter expression]]></Calculation>
    <Script id="N" name="script name"/>
  </Step>
```

**Cross-file (verified 2026-07-31 via round-trip Copy from FileMaker Pro v22.0.6.611 / "FileMaker Pro 2025", macOS).** When the target script lives in a different file than the one being pasted into, a `<FileReference>` element appears as the **first** child, before `<Calculation>`/`<Script>`:
```
  <Step enable="True" id="1" name="Perform Script">
    <FileReference id="N" name="target file name">
      <UniversalPathList>file:target file name</UniversalPathList>
    </FileReference>
    <Calculation><![CDATA[parameter expression]]></Calculation>
    <Script id="N" name="script name"/>
  </Step>
```
Without a parameter, `<Calculation>` is omitted entirely (round-trip confirmed):
```
  <Step enable="True" id="1" name="Perform Script">
    <FileReference id="N" name="target file name">
      <UniversalPathList>file:target file name</UniversalPathList>
    </FileReference>
    <Script id="N" name="script name"/>
  </Step>
```
Element order: `FileReference` → `Calculation` (present only when a parameter is set, confirmed by round-trip comparison of the with/without-parameter forms above) → `Script`. `<UniversalPathList>` uses the bare `file:TargetFileName` form with **no `.fmp12` extension** — do not confuse with the `Open File` step's own `UniversalPathList` form (`steps-windows-files.md`), which does include the extension; these are the same `file:` prefix convention used differently by two different steps.

**Silent-drop warning, cross-file case.** Generating this step with the same-file skeleton above (omitting `<FileReference>`) when the target script is actually in a different file does not produce a "missing reference" indicator — it pastes as `Perform Script [<unknown>]`, a full resolution failure distinct from the graceful ID/name fallback documented in core.md §5. See core.md §7 for the general pattern this fits.

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

**Not save-valid as documented above** — no reference for either the
main script or the callback, and live save-testing confirms
FileMaker won't save the file left this way. Use as a starting
skeleton only.

Configured (both the main script and the callback pointed at the same
script, round-trip verified):
```
  <Step enable="True" id="210" name="Perform Script on Server with Callback">
    <CallbackScriptState value="Continue"/>
    <Script id="1" name="script name"/>
    <CallbackScript>
      <ScriptName id="1" name="script name"/>
    </CallbackScript>
  </Step>
```

**Naming asymmetry — easy to get wrong.** The two script references
use different element names for what looks like the same kind of
reference. The main script is a bare `<Script id name>` sibling of
`<CallbackScriptState>`. The callback's reference is `<ScriptName id
name>`, nested inside `<CallbackScript>`, not `<Script>`. Do not
assume both slots take the same tag.

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

**FM 2026 addition:** native Copy output for every Comment step,
including the bare divider form, also includes
`<Restore state="False"/>`. Like `DisableStepCollapsed` (core.md
§6.0), not required for paste — generators may omit it.

---

