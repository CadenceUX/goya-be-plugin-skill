# CHANGELOG — goya-be-plugin

## v1.2 — 2026-06-23

Added plugin-version tracking and a staleness-check method, closing the gap with `claris-filemaker-pro`'s `last_known_fm_version` + version-drift pattern. Extended the reference JSON with corrected function data and attribution fixes (2026-06-30).

- **New `plugin_version_at_build` field** (catalog `meta` block and `SKILL.md` frontmatter) records the latest **stable** BaseElements Plugin release at build time: `5.1.0.6`, published 2026-04-15. Confirmed via the GitHub releases API — `v5.1.0b3` has a newer publish timestamp and no `prerelease` flag, but its `Full Changelog` link diffs `v5.1.0.6...v5.1.0b3`, confirming it's a beta built *on top of* 5.1.0.6, not a later stable release. Used 5.1.0.6, not the beta.
- **New `plugin_version_check` field** in catalog `meta` documents how to detect drift: query the releases API, filter to tags with no letters after the leading `v` (excludes betas like `b3` even when GitHub's `prerelease` flag misreports them), and compare the highest stable tag against `plugin_version_at_build`.
- **New "Plugin version & staleness check" section in `SKILL.md`** mirrors `claris-filemaker-pro`'s version-drift detector, adapted to the fact that BE_ doc pages carry no per-page version frontmatter — so the check is done on-demand (when asked, or before recommending a rebuild) rather than on every fetch.
- **Reference JSON fully rebuilt from live source** — all 123 active + 11 deprecated function pages fetched and parsed from `raw.githubusercontent.com`. Every entry now reflects the current upstream doc: descriptions, structured parameters (name, optional flag, defaults parsed directly from doc page annotations), keywords, version history, notes (real content, not placeholders), platform compatibility, and example code.
- **Doc bugs surfaced in `notes` as well as `doc_bug_note`** — 12 functions have upstream documentation errors now flagged inline: `BE_Encrypt_AES` (signature names wrong function), `BE_FileSelectDialog` and `BE_FolderSelectDialog` (parameter named `nainFolderPathme` instead of `inFolderPath`), `BE_HTTP_GET_File` / `BE_HTTP_PUTFile` / `BE_HTTP_Set_Proxy` / `BE_XSLTApply` (signature names wrong function), `BE_HTTP_GET` / `BE_SMTPServer` / `BE_JSON_ArraySize` / `BE_Unzip` (parameter name mismatch in prose vs. signature), and `BE_SignatureVerifyRSA` (unmatched brace in signature). All are corrected to the name confirmed by the function's own signature line; the original as-fetched text is preserved.
- **Defaults now extracted from doc page annotations** — previously hand-authored; now parsed directly from the `( default:… )` inline annotations in each parameter's prose description, covering ~40 parameters across the catalog.
- **Parser edge cases handled** — `BE_GetSystemDrive` has no `##` heading (page-level doc bug; signature parsed from indented line); `BE_ArrayFind` uses bare parameter names without italic markers; `BE_RegularExpression` uses a tab rather than spaces for the signature indent. All three now parse correctly.
- **Attribution lines** added to `SKILL.md` `## Licence` and both locations in `README.md` (full `[Darrin Southern] from [CadenceUX]` form); fixed `README.md` author text from `Cadence UX` to `CadenceUX`.

## v1.1 — 2026-06-23

Full schema rebuild from live source — every active and deprecated function entry now carries the same depth of detail as `claris-filemaker-pro`'s catalog, not just a signature and a URL.

- **Fetched and parsed all 134 doc pages** (123 active + 11 deprecated) directly from `raw.githubusercontent.com` — the raw `.md` source, not the rendered `github.com` HTML page, which loses structure when pulled through a web-fetch/summarizer tool.
- **New per-entry fields**: `description`, `parameters` (structured — name, optional flag, default, description), `keywords`, `version_history`, `notes` (array of paragraphs), `compatibility` (per-platform support table), and `example_code` (verbatim from the doc page, `null` where none exists).
- **`url` now points at the raw `.md` file** instead of the rendered GitHub page, on every entry and in `url_pattern`/`deprecated_url_pattern`.
- **Corrected 88 of 123 active signatures** that had drifted from the hand-authored v1.0 catalog (e.g. `BE_ArrayDelete` had an `index` parameter it doesn't take; `BE_CipherDecrypt`/`BE_CipherEncrypt` were missing two parameters each and had the wrong parameter names).
- **Flagged 6 upstream doc bugs** via a new `doc_bug_note` field: pages where the code-block signature's leading identifier didn't match the page's own heading/filename (`BE_Encrypt_AES`, `BE_HTTP_GET_File`, `BE_HTTP_PUTFile`, `BE_HTTP_Set_Proxy`, `BE_VersionAutoUpdate`, `BE_XSLTApply`) — almost certainly copy/paste errors in Goya's own docs. The function name was corrected, the real parameter list kept, and the as-fetched original preserved in the note rather than silently fixed.
- **Parameter optionality reconciled against the signature's `{ }` brace-group** — several doc pages mark a parameter optional in the signature but omit the `(optional)` annotation in the prose (e.g. `BE_CipherDecrypt`'s `fileNameWithExtension`); the brace-group is now treated as authoritative.
- Noted `BE_SignatureVerifyRSA` has an unmatched brace in its own signature line (opens `{`, closes with `)` not `}`) — left as-is, documented in `update_instructions` rather than silently corrected.
- `meta` block extended with `last_updated`, `update_instructions`, and `schema_note` fields, mirroring `claris-filemaker-pro/references/function-catalog.json`.
- Category-level `notes` (general usage guidance, e.g. Arrays' 1-based indexing) and the `deprecated[]` array's `status`/`reason`/`replaced_by` fields are preserved unchanged from v1.0.

## v1.0 — 2026-06-14

Renamed from `be-plugin`. Significant structural upgrade.

- **Renamed skill** from `be-plugin` to `goya-be-plugin` (reflects vendor name: Goya Pty Ltd)
- **Reference file converted to JSON** — `references/function-catalog.json` replaces `references/function-index.md`
- **Per-function doc URLs** added to every active function entry (GitHub markdown docs)
- **Deprecated and removed functions** now fully documented in the JSON catalog with:
  - `status` field: `deprecated`, `renamed`, or `removed`
  - `reason` field explaining why
  - `replaced_by` field pointing to the correct native FM or plugin alternative
  - Individual GitHub doc URL under `docs/Functions/Deprecated/`
- **Version format** changed from `1.0.0` to `v1.0`
- SKILL.md updated with:
  - Deprecated / renamed / removed function table with status column
  - Note that BE_ExecuteShellCommand was **renamed** (not deprecated) to BE_ExecuteSystemCommand
  - Note that BE_XeroSetTokens was **removed** (Xero integration dropped entirely)
  - Pointer to JSON catalog and instructions for live doc fetching via `url` field

---

## v0.1.0 (be-plugin) — 2026-06-14

Initial release as `be-plugin`.

- Full function catalogue for BaseElements Plugin by Goya Pty Ltd
- Compact signature reference for all ~90 active functions across 16 categories (Markdown)
- Critical usage patterns: HTTP call sequence, error checking, file paths
- Deprecated function table with native FM replacements
- Live doc URL pattern for per-function GitHub fetch
