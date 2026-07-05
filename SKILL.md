---
compatibility: Claude.ai, Claude Chat, Claude Code
metadata:
  "Built and maintained": "Darrin Southern from CadenceUX"
  version: "1.3"
name: goya-be-plugin
description: |
  Reference skill for the BaseElements Plugin by Goya Pty Ltd — a free, open-source
  FileMaker Pro plugin with 129 active functions covering file I/O, HTTP/curl, SMTP,
  XML/XSLT, JSON, PDF, encryption, clipboard, containers, value lists, arrays, regex,
  JavaScript evaluation, and vector similarity, plus the plugin's named constants
  (BE_ButtonOK, BE_FileTypeFolder, encoding and digest-algorithm values). Includes
  deprecated and renamed functions with their replacements. Use whenever any BE_
  function or constant name appears in a FileMaker calculation or script, or when the
  user asks about BaseElements, the BE plugin, or plugin-based HTTP, file, XML, or
  email operations in FileMaker. The reference file is a JSON catalog
  (references/function-catalog.json) — each function entry includes its signature,
  notes, and a direct GitHub doc URL for live fetching. Do not use for MBS Plugin or
  other FileMaker plugins.
---

# BaseElements Plugin — Reference Skill

**Vendor:** Goya Pty Ltd  
**Source repo:** https://github.com/GoyaPtyLtd/BaseElements-Plugin  
**Function docs index:** https://raw.githubusercontent.com/GoyaPtyLtd/BaseElements-Plugin/main/docs/Functions.md  
**URL pattern (active functions):**  
`https://raw.githubusercontent.com/GoyaPtyLtd/BaseElements-Plugin/main/docs/Functions/{FunctionName}.md`  
**URL pattern (deprecated/removed functions):**  
`https://raw.githubusercontent.com/GoyaPtyLtd/BaseElements-Plugin/main/docs/Functions/Deprecated/{FunctionName}.md`

The function reference is in `references/function-catalog.json`. As of v1.1 each entry is self-contained — `name`, `signature`, `description`, structured `parameters` (name, optional flag, default, description), `keywords`, `version_history`, `notes`, `compatibility` (per-platform support), `example_code` (verbatim, may be `null`), and `url` (the raw `.md` source for cross-checking). No need to fetch the URL for routine use — it's only needed to verify against the live page or pick up changes since the last refresh.

Some entries carry a `doc_bug_note` field — these are cases where Goya's own doc page has a signature line naming a different function than its heading/filename (a copy/paste error upstream). The function name has been corrected in the entry; the note preserves what was actually on the page.

Deprecated functions have `status: "deprecated"`, renamed functions have `status: "renamed"`, and fully removed functions have `status: "removed"`. Check the deprecated array before suggesting any legacy BE_ function.

The catalog's `constants` block holds the plugin's **named constants** (from the vendor's
`Constants.md` page) — zero-parameter functions used in place of magic numbers: dialog button
results (`BE_ButtonOK` = 1, `BE_ButtonCancel` = 2, `BE_ButtonAlternate` = 3 — check these when
branching on `BE_DialogDisplay`'s result), file-type filters for `BE_FileListFolder`
(`BE_FileTypeAll`/`BE_FileType_File`/`BE_FileTypeFolder`), and the encoding and
digest-algorithm values for `BE_MessageDigest`. Prefer the named constant over its numeric
value in generated calculations. Note the upstream doc bug flagged on `BE_ButtonOK` (its
Constants.md description is a copy/paste of the alternate-button row).

---

## Plugin version & staleness check

The catalog `meta` field `plugin_version_at_build` (currently `5.1.0.6`) records the latest **stable**
BaseElements Plugin release at the time this catalog was last rebuilt. It is **not** a cap on what
you can answer about — it's a marker for how fresh the local data is.

Unlike `claris-filemaker-pro`, BE_ doc pages carry no per-page version frontmatter, so drift can't be
checked on every fetch. Instead, check it when it matters — the user asks if the skill is current, a
function doesn't seem to exist that should, or before recommending a rebuild:

1. `GET https://api.github.com/repos/GoyaPtyLtd/BaseElements-Plugin/releases?per_page=20`
2. Find the highest tag whose version string has **no letters** after stripping the leading `v`
   (e.g. `v5.1.0.6` qualifies, `v5.1.0b3` does not — the `b3` suffix marks a beta even when the API's
   `prerelease` flag reports `false`). Confirm by checking the release body's
   `Full Changelog: vX...vY` link — a beta's changelog diffs *from* the latest stable, not the reverse.
3. Compare that tag against `plugin_version_at_build`. If it's newer:

   > ⚠️ **Plugin version drift detected** — the latest stable BaseElements Plugin release is [X], but
   > this skill's catalog was built against 5.1.0.6. New or changed functions since then won't be
   > reflected here. Consider running a catalog rebuild (`update_instructions` in
   > `function-catalog.json`'s `meta` block) before relying on anything not already in this catalog.

4. Don't flag a newer beta/pre-release tag alone — wait for it to ship stable.
5. Still answer using the local catalog — the flag is advisory, not a blocker.

**Enumeration drift (new functions in docs without a plugin release):** docs pages can be
added ahead of a stable release, which the releases check above can't see. One extra API call
catches it: `GET https://api.github.com/repos/GoyaPtyLtd/BaseElements-Plugin/contents/docs/Functions`
and count the `.md` files, comparing against `meta.active_docs_page_count_at_build` (the same
check works for the `Deprecated/` subfolder against `meta.deprecated_docs_page_count_at_build`).
A higher live count means new pages exist that this catalog hasn't ingested. Two pages in the
folder are not functions — `Constants.md` (catalogued in the `constants` block) and
`FunctionTemplate.md` (the vendor's authoring template) — so don't flag those as missing
functions when reconciling names.

---

## Documentation freshness check (per live lookup)

Distinct from the plugin release check above — this is about whether a specific function's *doc
page* has been edited since the catalog was last built, independent of whether the plugin itself
shipped a new version. Since BE_ pages carry no version frontmatter (see above), use the doc
file's GitHub commit history as the freshness signal instead:

1. When fetching a function's live `.md` page to verify or supplement a catalog entry, also
   `GET https://api.github.com/repos/GoyaPtyLtd/BaseElements-Plugin/commits?path=docs/Functions/{FunctionName}.md&per_page=1`
   (use the `Deprecated/` path variant for deprecated functions).
2. Read the `commit.author.date` of the single returned commit — that's when this specific page
   was last edited.
3. This catalog doesn't record a per-function "last checked" timestamp, so there's no stored
   baseline to diff against precisely. Treat the commit date as **informational context** for the
   user (e.g. "this function's doc page was last updated on [date]") rather than a firm drift flag.
4. If the commit date is recent relative to `plugin_version_at_build`'s known release date, that's
   worth surfacing — it may mean the page was touched for this plugin version specifically.
5. Skip this check for routine lookups; only run it when the user asks if something is current, or
   a function's behaviour seems to disagree with the catalog's notes.

---

## Skill update self-check

Distinct from the plugin staleness check above (which watches Goya's plugin) — this checks whether
a newer release of **this skill** exists. At the start of each session, on the first BE_ question,
run once:

1. Fetch `https://github.com/CadenceUX/goya-be-plugin-skill/raw/main/VERSION`
2. Parse the returned string as the latest available version
3. Compare with this skill's installed version (currently `"1.3"`)
4. If latest > installed, prepend this notice to your first response:

   > ⚠️ **Skill update available**
   > This skill is v[installed]. v[latest] is available at
   > https://github.com/CadenceUX/goya-be-plugin-skill/releases
   > Update your local skill files to get the latest BE_ coverage.

5. Do not repeat the notice again in the same session.
6. If the fetch fails or returns an unexpected value, skip silently — do not surface the error.

---

## Deprecated, Renamed, and Removed Functions

Do not use deprecated or removed functions in new code. Prefer native FM equivalents where available. Renamed functions still exist in the plugin under their new name.

| Old name | Status | Use instead |
|---|---|---|
| BE_Base64_Decode | deprecated | Base64Decode (native FM) |
| BE_Base64_Encode | deprecated | Base64Encode (native FM) |
| BE_Base64_URL_Encode | deprecated | Base64EncodeURL (native FM) |
| BE_ExecuteShellCommand | **renamed** | BE_ExecuteSystemCommand |
| BE_FileMaker_Fields | deprecated | FieldNames / DDL (native FM) |
| BE_FileMaker_Tables | deprecated | TableNames / DDL (native FM) |
| BE_HMAC | deprecated | CryptAuthCode (native FM 19.3+) |
| BE_JSON_Encode | deprecated | JSONSetElement (native FM) |
| BE_JSON_Error_Description | deprecated | JSONFormatElements (native FM) |
| BE_JSONPath | deprecated | JSONGetElement (native FM) |
| BE_XeroSetTokens | **removed** | — (Xero integration removed) |

---

## Critical Patterns

### Error checking after HTTP / FTP / SMTP calls

Every network call is silent on failure. Always check in this order:

```filemaker
# 1. Plugin-level curl error (0 = success)
Set Variable [ $curlError ; Value: BE_GetLastError ]

# 2. HTTP status code (200, 401, 404, 500 etc.)
Set Variable [ $httpCode ; Value: BE_HTTP_ResponseCode ]

# 3. Response headers — diagnose server-side issues
Set Variable [ $headers ; Value: BE_HTTP_ResponseHeaders ]

# 4. Full curl transcript — use only when above aren't enough
Set Variable [ $trace ; Value: BE_CurlTrace ]
```

`BE_GetLastError` returns curl error codes, not FileMaker errors. Reference: http://curl.haxx.se/libcurl/c/libcurl-errors.html

---

### Standard HTTP call sequence

```filemaker
# 1. Set headers (persists until cleared or overwritten)
BE_HTTP_SetCustomHeader ( "Content-Type" ; "application/json" )
BE_HTTP_SetCustomHeader ( "Authorization" ; "Bearer " & $token )

# 2. Set curl options if needed (auth type, SSL, timeouts etc.)
BE_CurlSetOption ( "CURLOPT_TIMEOUT" ; 30 )

# 3. Make the call
Set Variable [ $response ; Value: BE_HTTP_POST ( $url ; $body ) ]

# 4. Check errors immediately — they reset on next call
Set Variable [ $curlErr  ; Value: BE_GetLastError ]
Set Variable [ $httpCode ; Value: BE_HTTP_ResponseCode ]

# 5. Clear options when done (important in loops or reuse)
BE_CurlSetOption   // no parameters = reset all to defaults
```

Calling `BE_CurlSetOption` with no parameters resets all options. Calling it with just an option name (no value) resets that single option.

---

### File paths

BE functions use **native OS paths** (POSIX on Mac/Linux, backslash on Windows), NOT FileMaker paths returned by `Get(TemporaryPath)` or similar. Strip the `file:/` prefix when converting from a FileMaker path.

```filemaker
# Wrong — raw FileMaker path
"/Macintosh HD/Users/nick/Desktop/file.txt"   // FM display path, not a plugin path

# Right — POSIX path
"/Users/nick/Desktop/file.txt"

# Converting temp folder path
Substitute ( Get ( TemporaryPath ) ; "file:/" ; "/" ) & "file.txt"
```

---

### Stack functions (global named stacks)

`BE_StackPush / BE_StackPop / BE_StackCount / BE_StackDelete` implement a named LIFO stack that persists across scripts in the same session. Useful for passing multiple values between scripts without script parameters.

```filemaker
BE_StackPush ( "myStack" ; $value )
Set Variable [ $val ; Value: BE_StackPop ( "myStack" ) ]
```

---

### Variable functions (global variables without $$)

`BE_VariableSet` and `BE_VariableGet` store values by name in the plugin's own global scope — useful when a variable must survive script scope changes without polluting FM's `$$` namespace.

---

### Regex

`BE_RegularExpression ( text ; expression ; { options ; replaceString } )` — uses PCRE. If `replaceString` is provided, returns the modified text. If not, returns the matched portion. Returns empty on no match. `options` is a string of flags, e.g. `"igm"` (case-insensitive, multiline, dot-matches-all), or `"v"` to treat `text` as a value list and iterate over each value.

---

### Vector similarity (AI / embeddings)

`BE_VectorDotProduct` and `BE_VectorEuclideanDistance` accept FileMaker value lists of numbers (one per line). Useful for cosine similarity after normalisation, or L2 distance for embedding comparisons.

---

### SMTP — setup sequence

```filemaker
BE_SMTPServer ( $host ; $port ; $username ; $password )  // configure once
BE_SMTPSetHeader ( "From" ; "sender@example.com" )
BE_SMTPSetHeader ( "To" ; "recipient@example.com" )
BE_SMTPSetHeader ( "Subject" ; "Hello" )
BE_SMTPAddAttachment ( $containerField )  // optional, repeatable
BE_SMTPSend ( $body )
```

Check `BE_GetLastError` after `BE_SMTPSend`.

---

### PDF functions

`BE_PDFAppend`, `BE_PDFGetPages`, `BE_PDFPageCount` work on native file paths. `BE_PDFAppend` appends one PDF to another on disk — the destination file must already exist.

---

### JavaScript evaluation

`BE_EvaluateJavaScript ( script )` runs arbitrary JS in a headless JavaScriptCore context. No DOM, no network. Returns the string result of the last expression.

---

### DDL errors

`BE_GetLastDDLError` returns the last error from `BE_FileMakerSQL`. Always check it after executing DDL (ALTER TABLE, CREATE TABLE etc.) via `BE_FileMakerSQL`.

---

## Function category quick index

| Category | Key functions |
|---|---|
| Arrays | BE_Array* |
| Clipboard | BE_Clipboard* |
| Containers | BE_Container*, BE_ConvertContainer, BE_ExportFieldContents, BE_JPEGRecompress |
| Data Manipulation | BE_Stack*, BE_TextExtractWords, BE_Variable* |
| Dialogs | BE_Dialog*, BE_FileSaveDialog, BE_FileSelectDialog, BE_FolderSelectDialog |
| Encoding & Encryption | BE_Cipher*, BE_Decrypt_AES, BE_Encrypt_AES, BE_MessageDigest, BE_Signature* |
| Error Checking | BE_CurlTrace, BE_DebugInformation, BE_GetLastDDLError, BE_GetLastError |
| Files & Folders | BE_File*, BE_FolderCreate, BE_FolderSelectDialog |
| HTTP & URLs | BE_CurlSetOption, BE_CurlTrace, BE_FTP_*, BE_HTTP_*, BE_OpenURL |
| Miscellaneous | BE_EvaluateJavaScript, BE_ExecuteSystemCommand, BE_FileMakerSQL, BE_GetMachineName, BE_GetSystemDrive, BE_Pause, BE_RegularExpression, BE_Script*, BE_SetTextEncoding, BE_Version* |
| PDF | BE_PDF* |
| Preferences | BE_Preference*, BE_SetPreference |
| SMTP | BE_SMTP* |
| Time | BE_Time* |
| Value Lists | BE_Values* |
| Vectors | BE_Vector* |
| XML / XSLT / JSON | BE_JSON_ArraySize, BE_XML*, BE_XPath*, BE_XSLT* |
| Zip / Gzip | BE_Gzip, BE_UnGzip, BE_Unzip, BE_Zip |

---

## Licence

This skill is released under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).

Built and maintained by [Darrin Southern](https://www.linkedin.com/in/darrin-southern/) from [CadenceUX](https://cadenceux.com.au).
