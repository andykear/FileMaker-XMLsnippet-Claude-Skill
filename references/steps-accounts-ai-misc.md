# Step Reference — Configure, Accounts, AI, Spelling, Menus, Miscellaneous (§8.2, §8.10–8.14)

Part of the Canonical XML Format for FileMaker Script Steps, v1.12.
Read `core.md` first: the paste format requirements, conventions and
silent failure modes there apply to every step below.

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

#### Configure Persistent Data (238)

Sets or deletes an entry in the per-file persistent data store
(Draco Catalog). Entries survive file close, Data Migration Tool,
and session restarts. Read values with `GetPersistentData()` and
`ListPersistentDataIDs()`.

Options: `Name` (required text), `Instance ID` (optional text
calculation), `Value` (expression, data type preserved) or
`Delete Entry`. An unspecified Instance ID is treated the same as
an empty string. Deleting a nonexistent entry returns error 10.

Unconfigured:
```
  <Step enable="True" id="238" name="Configure Persistent Data">
    <Option state="False"/>
  </Step>
```

Set mode (round-trip verified):
```
  <Step enable="True" id="238" name="Configure Persistent Data">
    <Option state="False"/>
    <Value>
      <Calculation><![CDATA[value expression]]></Calculation>
    </Value>
    <InstanceId>
      <Calculation><![CDATA["instance id"]]></Calculation>
    </InstanceId>
    <Name>"entry name"</Name>
  </Step>
```

Delete mode (round-trip verified):
```
  <Step enable="True" id="238" name="Configure Persistent Data">
    <Option state="True"/>
    <InstanceId>
      <Calculation><![CDATA["instance id"]]></Calculation>
    </InstanceId>
    <Name>"entry name"</Name>
  </Step>
```

Notes:
- `<Option state="False"/>` = Set. `<Option state="True"/>` = Delete.
- **`<Name>` is plain text, not a Calculation wrapper.** Content
  includes literal quotes when a quoted string is entered. This is an
  inconsistency: `<Value>` and `<InstanceId>` both use
  `<Calculation><![CDATA[...]]></Calculation>` but `<Name>` does not.
  Generators must emit `<Name>` as plain text, not CDATA-wrapped.
  This may be a silent-failure trap.
- Element order (Set): `Option` → `Value` → `InstanceId` → `Name`.
- Element order (Delete): `Option` → `InstanceId` → `Name`.

---

### 8.10 Accounts

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
| Delete Account | 135 |

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

`<LLMType>` enumeration (all round-trip verified):

| Provider | LLMType value |
|---|---|
| OpenAI | `ChatGPT` |
| Anthropic | `Anthropic` |
| Cohere | `Cohere` |
| Google Gemini | `Google` |
| Custom | `Other` |

Unconfigured:
```
  <Step enable="True" id="212" name="Configure AI Account">
    <LLMType value="ChatGPT"/>
    <SetLLMAccount/>
  </Step>
```

Predefined providers (round-trip verified):
```
  <Step enable="True" id="212" name="Configure AI Account">
    <LLMType value="ChatGPT"/>
    <SetLLMAccount>
      <AccountName>
        <Calculation><![CDATA["account name"]]></Calculation>
      </AccountName>
      <AccessAPIKey>
        <Calculation><![CDATA["api key"]]></Calculation>
      </AccessAPIKey>
    </SetLLMAccount>
  </Step>
```

Custom provider (round-trip verified):
```
  <Step enable="True" id="212" name="Configure AI Account">
    <VerifySSLCertificates state="False"/>
    <LLMType value="Other"/>
    <SetLLMAccount>
      <AccountName>
        <Calculation><![CDATA["account name"]]></Calculation>
      </AccountName>
      <Endpoint>
        <Calculation><![CDATA["https://server.example.com/llm/v1/"]]></Calculation>
      </Endpoint>
      <AccessAPIKey>
        <Calculation><![CDATA["api key"]]></Calculation>
      </AccessAPIKey>
    </SetLLMAccount>
  </Step>
```

Custom adds `<VerifySSLCertificates>` as a direct Step child before
`<LLMType>`, and `<Endpoint>` inside `<SetLLMAccount>` between
`AccountName` and `AccessAPIKey`.

Note: FM 26 corrected the inner element spelling from `SetLLMAccout`
(FM 2025) to `SetLLMAccount`. See Appendix B. Generators targeting
FM 26 should use the corrected spelling.

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

#### Insert Image Caption (241)

Sends an image to a captioning model and inserts the returned text
into a field or variable. Claris AI Model Server only.

Options: `Account Name`, `Model`, `Input` (expression returning
container data), `Target` (field or variable, required).

```
  <Step enable="True" id="241" name="Insert Image Caption">
    <LLMEmbedding/>
  </Step>
```

Reuses the `<LLMEmbedding/>` inner element from Insert Embedding (215).

#### Insert Image Captions in Found Set (240)

Batch version: for every record in the found set, sends an image
from a source field to a captioning model and inserts the caption
into a target field. Claris AI Model Server only.

Options: `Account Name`, `Model`, `Source Field` (container),
`Target Field` (text), `Replace target contents`,
`Continue on error`, `Parameters` (JSON with `MaxRecPerCall`
default 20, range 1-500).

```
  <Step enable="True" id="240" name="Insert Image Captions in Found Set">
    <LLMBulkEmbedding/>
  </Step>
```

Reuses the `<LLMBulkEmbedding/>` inner element from Insert Embedding
in Found Set (216).

---

### 8.12 Spelling

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
| Check Found Set | 20 |
| Check Record | 19 |
| Correct Word | 106 |
| Edit User Dictionary | 109 |
| Select Dictionaries | 108 |

#### Check Selection (18)
```
  <Step enable="True" id="18" name="Check Selection">
    <SelectAll state="True"/>
  </Step>
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
| AVPlayer Set Options | 179 |
| Beep | 93 |
| Exit Application | 44 |
| Flush Cache to Disk | 102 |
| Flush Web Viewer Cookies | 237 |
| Install Plug-In File | 157 |
| Refresh Portal | 180 |
| Set Session Identifier | 208 |

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

#### AVPlayer Set Playback State (178)
```
  <Step enable="True" id="178" name="AVPlayer Set Playback State">
    <PlaybackState value="Stopped"/>
  </Step>
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
`<Message>` → optional `<Height>` → `<Width>` → `<DistanceFromTop>` →
`<DistanceFromLeft>` → `<Buttons>`. `<Title>` and `<Message>` each
wrap a `<Calculation>`. `<Buttons>` always contains exactly three
`<Button>` slots — unused slots are self-closing.
`CommitState="True"` on a button indicates that the button commits
the pending record; it is not a "default button" indicator.

**FM 26: Size and position** (round-trip verified). `<Height>`,
`<Width>`, `<DistanceFromTop>`, and `<DistanceFromLeft>` are optional
calculation elements specifying dialog size and position in points.
They sit between `<Message>` and `<Buttons>`. All are optional —
unspecified values use defaults. FileMaker Go does not support size
and position.
```
  <Step enable="True" id="87" name="Show Custom Dialog">
    <Title>
      <Calculation><![CDATA["Dialog Title"]]></Calculation>
    </Title>
    <Message>
      <Calculation><![CDATA["Message text"]]></Calculation>
    </Message>
    <Height>
      <Calculation><![CDATA[200]]></Calculation>
    </Height>
    <Width>
      <Calculation><![CDATA[400]]></Calculation>
    </Width>
    <DistanceFromTop>
      <Calculation><![CDATA[100]]></Calculation>
    </DistanceFromTop>
    <DistanceFromLeft>
      <Calculation><![CDATA[100]]></Calculation>
    </DistanceFromLeft>
    <Buttons>
      <Button CommitState="True">
        <Calculation><![CDATA["OK"]]></Calculation>
      </Button>
      <Button CommitState="False">
        <Calculation><![CDATA["Cancel"]]></Calculation>
      </Button>
      <Button CommitState="False"/>
    </Buttons>
  </Step>
```

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

