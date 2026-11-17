# goya-be-plugin-skill

A [Claude skill](https://docs.claude.ai/skills) for the **BaseElements Plugin** by Goya Pty Ltd — a free, open-source FileMaker Pro plugin. Gives Claude accurate, version-checked knowledge of every `BE_` function, including parameters, defaults, platform compatibility, and version history, without relying on potentially stale training data.

Built and maintained by [Darrin Southern](https://www.linkedin.com/in/darrin-southern/) from [CadenceUX](https://cadenceux.com.au).

---

## What it does

When this skill is active, Claude will:

- Look up any of the **active `BE_` functions** by name or category — with full description, structured parameters (including optional/default flags), keywords, version history, platform compatibility, and example code where available
- Recognise and steer away from the **11 deprecated, renamed, or removed functions**, pointing to the correct native FileMaker or plugin replacement
- Resolve the plugin's **named constants** (dialog button results, `BE_FileListFolder` type filters, `BE_MessageDigest` encoding/algorithm values) so generated calculations use `BE_ButtonOK` rather than a magic `1`
- Flag known **upstream documentation bugs** rather than silently trusting them — where one of Goya's own doc pages is still wrong (a signature naming another function, an unmatched brace), the catalog holds the corrected form and notes the page's text in `doc_bug_note`
- Check for **plugin version drift** on request — the catalog tracks the latest *stable* BaseElements Plugin release at build time (`plugin_version_at_build`) and knows how to query GitHub's releases API to see if a newer stable release has shipped since
- Apply critical usage patterns baked into `SKILL.md` — HTTP/FTP/SMTP error-checking sequences, native OS file paths vs. FileMaker paths, the stack/variable helper functions, and more

---

## Coverage

| Reference file | Contents |
|---|---|
| `function-catalog.json` | All 129 active functions across 18 categories, plus 11 deprecated/renamed/removed entries and the plugin's 10 named constants (from the vendor's `Constants.md`). Each function entry: `signature`, `description`, `parameters` (structured), `keywords`, `version_history`, `notes`, `compatibility`, `example_code`, and `url` (raw `.md` source). |

**Categories:** Arrays, Clipboard, Containers, Data Manipulation, Dialogs, Encoding and Encryption, Error Checking, Files and Folders, HTTP and URLs, Miscellaneous, PDF, Preferences, SMTP Email, Time, Value Lists, Vectors, XML/XSLT/JSON, Zip and Gzip.

---

## Install

**Claude Code** (from the `cadenceux` plugin marketplace):

```bash
claude plugin marketplace add https://github.com/CadenceUX/cadenceux-skills.git
claude plugin install goya-be-plugin@cadenceux
```

**claude.ai and Cowork:** Customize → Plugins → Add marketplace from GitHub →
`CadenceUX/cadenceux-skills`, then install **goya-be-plugin**.

**Claude desktop app (macOS):** download [`goya-be-plugin.skill`](https://github.com/CadenceUX/goya-be-plugin-skill/releases/latest/download/goya-be-plugin.skill) and
double-click it.

**Claude.ai in a browser, as a single skill:** Customize → Skills → upload
[`goya-be-plugin.zip`](https://github.com/CadenceUX/goya-be-plugin-skill/releases/latest/download/goya-be-plugin.zip).

> Both files are attached to the latest [release](https://github.com/CadenceUX/goya-be-plugin-skill/releases).
> Don't use GitHub's green **Code → Download ZIP** button or a release's **Source code (zip)**:
> those are snapshots of the repository, not a packaged skill, and Claude won't install them
> as one.

## How it works

The skill is **fully local-first** as of v1.1 — every function entry carries its full description, parameters, notes, compatibility, and example code, so most questions are answered without a live fetch. The `url` field on each entry still points at the raw GitHub doc page for cross-checking or picking up changes since the last refresh.

Use the raw `raw.githubusercontent.com` URLs, not the rendered `github.com` HTML pages — the HTML page loses structure when pulled through a web-fetch/summarizer tool, which is how several of the catalog's earlier signature errors went undetected.

---

## Known upstream issues

A handful of things in Goya's own documentation are worth knowing about if you're cross-checking against the live site:

- **Heading and signature copied from another page** — `BE_VersionAutoUpdate`'s doc page is headed and signed `BE_Version`. The catalog corrects the function name and flags it via `doc_bug_note`. (Goya fixed the same kind of bug on `BE_Encrypt_AES`, `BE_HTTP_GET_File`, `BE_HTTP_PUTFile`, `BE_HTTP_Set_Proxy` and `BE_XSLTApply`, plus five parameter-name typos, in July 2026.)
- **`BE_ButtonOK` description** — the `Constants.md` row for `BE_ButtonOK` repeats the alternate-button description; value 1 is the default (OK) button.
- **Unmatched brace** — `BE_SignatureVerifyRSA`'s signature line opens `{` but closes with `)` instead of `}`. Left as-is in the catalog rather than guessed at.
- **Beta tags can look like the latest release** — GitHub's `prerelease` flag isn't reliable for this repo; e.g. `v5.1.0b3` reports `prerelease: false` despite being a beta built on top of the actual latest stable, `v5.1.0.6`. Confirm by checking the release body's `Full Changelog` diff link rather than trusting the flag or publish date alone.

---

## Plugin version tracking

`function-catalog.json`'s `meta.plugin_version_at_build` records the latest stable BaseElements Plugin release this catalog was built against (currently `5.1.0.6`, published 2026-04-15). `meta.plugin_version_check` documents the exact method for detecting drift — query the GitHub releases API, filter out beta-tagged versions, and compare. See the "Plugin version & staleness check" section in `SKILL.md` for the full procedure.

---

## Keeping it current

Re-run the fetch-and-parse pass periodically (see `function-catalog.json`'s `meta.update_instructions` for the exact procedure):

1. Fetch the raw `docs/Functions.md` index and diff its function list against the catalog
2. For new or changed functions, fetch the individual raw `.md` page and parse it
3. Cross-check each parameter's optional flag against the signature's `{ }` brace-group — the prose `(optional)` annotation is sometimes missing even when the signature marks it optional
4. Watch for the same copy/paste pattern (signature naming the wrong function) on any newly added page

---

## Contributing

Issues and PRs welcome — particularly:
- New functions added to the BaseElements Plugin
- Further doc-page discrepancies spotted upstream
- Corrections to version history or compatibility tables

---

## Licence

[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) — free to use, adapt, and redistribute with attribution.

Built and maintained by [Darrin Southern](https://www.linkedin.com/in/darrin-southern/) from [CadenceUX](https://cadenceux.com.au).
