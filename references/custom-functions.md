# Custom Functions (§11)

Part of the Canonical XML Format for FileMaker Script Steps, v1.12.
Read `core.md` first.

## 11. Custom Functions

### 11.1 Overview

Custom functions use the same `fmxmlsnippet type="FMObjectList"` wrapper as script steps. The paste target is the Manage Custom Functions dialog (not the Script Workspace). Multiple functions can be included in a single snippet and will all paste in one operation.

### 11.2 Canonical skeleton

**No parameters:**
```xml
<CustomFunction id="1" functionArity="0" visible="True" parameters="" name="CF_Name">
  <Calculation><![CDATA[expression]]></Calculation>
</CustomFunction>
```

**With parameters:**
```xml
<CustomFunction id="1" functionArity="2" visible="True" parameters="input;multiplier" name="CF_Name">
  <Calculation><![CDATA[input * multiplier]]></Calculation>
</CustomFunction>
```

### 11.3 Attributes

| Attribute | Values | Notes |
|---|---|---|
| `id` | any integer | Ignored on paste — FileMaker assigns its own sequential ID. Use `id="1"` as placeholder. |
| `functionArity` | integer | Must match the number of parameters exactly. |
| `visible` | `"True"` / `"False"` | `"True"` = All accounts. `"False"` = Full access accounts only. |
| `parameters` | string | Semicolon-separated parameter names. Empty string `""` for zero parameters. |
| `name` | string | If a CF with this name already exists in the target file, FileMaker pastes it as a new CF with an auto-incremented name suffix (e.g. `CF_Test 2`). It does not overwrite. |

### 11.4 Parameters

Parameters are semicolon-separated in the `parameters` attribute — not comma-separated:

```xml
<CustomFunction id="1" functionArity="3" visible="True" parameters="input;multiplier;offset" name="CF_Name">
```

`functionArity` must equal the parameter count exactly.

### 11.5 Recursion

Recursive calls are plain text inside the CDATA body — no special element or attribute required:

```xml
<CustomFunction id="1" functionArity="1" visible="True" parameters="n" name="CF_Factorial">
  <Calculation><![CDATA[If ( n <= 1 ; 1 ; n * CF_Factorial ( n - 1 ) )]]></Calculation>
</CustomFunction>
```

### 11.6 Calculation body

The `<Calculation>` child follows the same CDATA convention as script steps. Multi-line bodies, comments (`//` and `/* ... */`), and fully commented-out bodies all round-trip intact. FileMaker treats the body as opaque text.

**Commented-out body (verified round-trip):**
```xml
<CustomFunction id="1" functionArity="1" visible="True" parameters="text" name="CF_Name">
  <Calculation><![CDATA[/* entire body commented out
for later use */]]></Calculation>
</CustomFunction>
```

### 11.7 Schema references in the body

Custom function bodies are opaque CDATA — the paste handler performs no schema resolution on field references, table names, or calls to other custom functions within the body. A function referencing `MyTable::MyField` or calling `AnotherCF()` will paste successfully into any file regardless of whether that schema or function exists. Runtime errors surface only when the function is evaluated, not at paste time.

This is a significant difference from script steps, where field and layout references are structured XML elements that FileMaker resolves against the recipient schema on paste.

### 11.8 Multiple functions in one snippet

Any number of `<CustomFunction>` elements may appear in a single snippet. All paste in one operation:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<fmxmlsnippet type="FMObjectList">
  <CustomFunction id="1" functionArity="1" visible="True" parameters="n" name="CF_Factorial">
    <Calculation><![CDATA[If ( n <= 1 ; 1 ; n * CF_Factorial ( n - 1 ) )]]></Calculation>
  </CustomFunction>
  <CustomFunction id="2" functionArity="0" visible="True" parameters="" name="CF_Today">
    <Calculation><![CDATA[Get ( CurrentDate )]]></Calculation>
  </CustomFunction>
</fmxmlsnippet>
```

Round-trip verified at scale: 24 functions pasted in a single snippet, all IDs reassigned sequentially from 1.

### 11.9 Known behaviour and limitations

- FileMaker ignores the `id` attribute on paste and assigns its own sequential IDs.
- Name conflicts result in a new CF being created with an auto-incremented suffix — not an overwrite.
- No silent-failure modes identified. The format is substantially simpler than script steps with no element-ordering traps.
- `functionArity` mismatch (value does not match actual parameter count) — behaviour untested; avoid.
- Add-on and cross-file field references in the body will paste without error but will fail at evaluation time if the referenced schema is not present in the target file.
