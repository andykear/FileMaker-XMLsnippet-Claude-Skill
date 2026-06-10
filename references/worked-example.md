# Worked Example (§10)

Optional reading: a start-to-finish walkthrough of generating a small
script. Section references point to `core.md` and the step category
files.

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
