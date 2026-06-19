# KBEditor Changelog

All notable changes to the KBEditor product. Each release documents changes across
three independent streams:

- **Plugins** — GeneXus language extensions (IntelliSense, parser, validator, visual editor, etc.)
- **Platform** — KBEditor-specific infrastructure (license manager, MCP server, embed pipeline, CI/CD)
- **VS Code** — Upstream VS Code base (via VSCodium)

---

## 1.8.0 — 2026-06-19

**Plugins v1.9.5** | **VS Code 1.109.4**

> **MINOR bump** — plugin team shipped the **Build Pipeline UI**, a major new feature. Right-click any `.gxSource` or `.gxPattern` → Specify / Specify + Generate / Build (compile) / Build Called (compile) — runs through the KB Sync Engine (v1.5.6+). New "Builds" and "Navigations" tabs in the Sync panel. Errors published as native VS Code diagnostics in the Problems panel. Environment picker, automatic-build tracking, Build All for whole-KB compiles. 6 plugin versions consolidated (v1.9.0 → v1.9.5), 19 entries — heavy iteration after the initial v1.9.0 ship.

### Plugins

#### v1.9.5 — 2026-06-18 (KB Sync Engine v1.5.6+)
- Added: [sync-status] **"Build Called (compile)" command** — compiles selected objects + their entire call tree (transitive). Plain Build (compile) now builds only the requested set; "Build Called" is the explicit opt-in to closure (IDE "with references"). Effective on GeneXus 17/18.
- Added: [sync-status] **Expand all details / Collapse all details** in panel context menu — applies to My Sources and Builds tabs. State remembered per batch in Builds across refreshes.
- Changed: [sync-status] **Build batches start collapsed** by default — even when finished with errors. Reduces clutter; expand the ones you care about or use Expand all.
- Fixed: [sync-status] **Clicking a pattern-generated object** (Work With child Web Component, etc.) in the Builds tab now opens its source — they live in hidden `.<instance>.<pattern>.gxPattern.Childs` folders that VS Code's file search skips. Resolver now falls back to async directory scan (cached after first hit).
- Fixed: [sync-status] **Error messages with section names jump to correct line** — section headers are externalized with suffixes (`#Events [dynamic]`); mapper now recognizes the suffix instead of falling back to body counting.

#### v1.9.4 — 2026-06-17 (KB Sync Engine v1.5.5+)
- Added: [sync-status] **"Force code generation on build" setting** (off by default) — when enabled, Specify+Generate / Build asks the Engine to regenerate code even if up-to-date. Useful right after internalizing changes.
- Fixed: [sync-status] **Atomic writes of sync files** — extension's pending-changes file and request files now write to a temp file in the same folder, then atomic rename. Eliminates intermittent Engine warnings `Could not read changes file ... Unexpected end when deserializing array`.

#### v1.9.3 — 2026-06-16
- Fixed: [sync-status] **Companion `.gxForm`/`.gxLayout` files written by a KBBridge export no longer show as false "pending"** — filter now matches by object identity (qualified name), not exact file path.
- Fixed: [sync-status] **Engine-owned status file no longer written by the extension** — eliminates a 2-writers race. Orphan entries are matched by change id and ignored.

#### v1.9.2 — 2026-06-15 (KB Sync Engine v1.5.3+)
- Added: [sync-status] **Per-user automatic builds** — the Engine attaches the username to its auto-build triggers; auto rows in the Builds tab are filtered to show only your own.
- Added: [sync-status] **Discard failed internalization** — when an import fails, the object shows "Externalization blocked" in My Sources with a one-click `↺` action to discard local changes and re-export from the KB (with confirmation, destructive).
- Added: [sync-status] **Build All button** (KB Sync Engine v1.5.4+) — on the Environment line of the Builds tab + palette command. Asks for confirmation, builds the whole KB. Resulting batch tagged "Build All".
- Added: [sync-status] **`not-specified` (NS) object state** in the Builds tab for Domains / Attributes / SDTs / Tables / External Objects after import. Info message with one-click "Request Build All" action.
- Changed: [sync-status] Per-object status badges shortened to aligned 2-letter codes (OK / WA / ER / SK / NS).
- Fixed: [sync-status] **Single `Ctrl+S` no longer produces 2 entries** in My Sources via the directory reconciler — save's own writes (including delayed re-writes from visual/pattern editors) are absorbed into the snapshot; reconciler skips files VSCode saved recently.
- Fixed: [sync-status] **Stale import errors no longer linger** after a later successful import of the same object — moment a successful internalization is recorded, earlier error/failed entries for the same object are cleared.

#### v1.9.1 — 2026-06-14 (KB Sync Engine v1.5.2+)
- Added: [sync-status] **Environment selector** — dropdown at the top of the Builds tab, preselected on the active environment; build requests stamped with the choice. Hidden if KB has a single environment. Each build row shows the environment it ran on.
- Added: [sync-status] **Automatic builds tracked** — Engine-triggered post-import builds appear in the Builds tab tagged "auto" with the same detail as manual builds. Errors also published as diagnostics.
- Added: [sync-status] **Navigations tab** — current navigation set per environment, filter by last N builds or "Full set", search by object name. Click opens the self-contained HTML navigation report.

#### v1.9.0 — 2026-06-12 (headline)
- Added: [sync-status] **Build Pipeline UI** — Specify / Specify + Generate / Build (compile) commands on `.gxSource` from the explorer or editor tab (multi-selection supported). Requests are written to `.kbbridge-sync/`, picked up by the Sync Engine ≤10s, run in a child process so sync isn't interrupted.
- Added: [sync-status] **New "Builds" tab** in the KBBridge Sync panel — live status (Sent ⏳ → Queued → Running → result), per-object badge (`ok` / `warnings` / `errors` / `skipped`), per-request "Output" section.
- Added: [sync-status] **Native VS Code diagnostics** — specification errors (with section/line/column) appear at the exact position in `.gxSource`; compilation errors as file-level diagnostics on line 1. Source: "GeneXus Build". Replaced on each new build, cleared when the object comes back clean, re-published after window reload.
- Added: [sync-status] **Navigation viewer** — each built object with a navigation dialog gets a "Navigation" link; opens a self-contained HTML with the IDE's Navigation Report look, with freshness banner.
- Added: [sync-status] **Pattern instances** — Specify/Generate/Build work on `.gxPattern` files too. Engine applies the pattern first, then builds parent + children (marked with "pattern" badge in the Builds tab).
- Fixed: [language-validator] **No false `Parameter '...' expects type 'Boolean' but got '<SDT name>'` errors** on call arguments that are comparisons or logical expressions (e.g. `iif(&SDTCollection.Count > 0, ...)`). Validator now recognizes top-level `=`, `<>`, `<`, `>`, `<=`, `>=`, `like`, `and`, `or`, `not` always produce `Boolean` regardless of operand types.
- Fixed: [sync-status] **"Track external changes" is ON by default again on fresh installs** — a 2026-04-13 change to read exclusively from User scope was dropping the declared `true` default when the user had never touched the setting. Reader now honors the declared default.

### Platform
- Bundle rebuilt with v1.9.5 plugin code. Visible changes: **sync-status grew from 53 → 66 files** (+13 — the new `src/build/`, `src/models/BuildTypes`, `ExportRequestWriter`, `NavigationViewer`, etc.). Other extensions unchanged. Total bundle: **730 JS** (was 717) / 9 CSS / 39 JSON / 2.2 MB.
- VSIX skeleton drift on `genexus-sync-status` (new commands + new view contributions for the Builds and Navigations tabs). Auto-fixed by `sync-vsix-skeletons.py` — exactly the workflow we built in v1.7.11.

### Tests
- 24/24 smoke tests pass.

### Bundle
- **Must be uploaded to `keygenapi.kbbridge.com`** — different shape (+13 files in sync-status).

### Compatibility note
The Build Pipeline UI requires **KB Sync Engine v1.5.6+** to function fully. With an older Engine: requests time out with "Waiting for Sync Engine" message, and the new controls (environment selector, auto builds, navigation tab, Build All, Build Called, Force code gen) stay hidden. Users running the new editor against an old Engine see a degraded but non-broken experience.

### VS Code 1.109.4
- No upstream changes

---

## 1.7.21 — 2026-06-09

**Plugins v1.8.29** | **VS Code 1.109.4**

> Bug-fix sync. 3 entries in validator/analyzer: `Set` method recognized on `Date` and `DateTime` variables, `&End` accepted as a variable name (was rejected as keyword), and a new check that catches `DataType = '<SDT>'` references missing their module qualifier when the SDT only lives inside a module — with a clear suggested correction.

### Plugins

#### v1.8.29 — 2026-06-08
- Fixed: [language-validator] **No false `'Set' is not a valid method for type 'date'`/`datetime` warnings on `&Var.Set(year, month, day)` and `Set(year, month, day, hour, minute, second)` for DateTime.** `Set` is the standard GeneXus way to assign a literal date/datetime to a variable in code — was missing from the catalog of both Date and DateTime built-in types.
- Fixed: [language-analyzer] **No false `'End' is not a valid keyword. Did you mean 'EndIf'?` errors on `&End` used as a variable name.** `&End` is a common loop exit flag in GeneXus code (`Do While &End = False`). Grammar now accepts `End` as a variable name when prefixed with `&`. Block-closing `End` (`Header ... End`) still works because the `&` prefix disambiguates.
- Added: [language-validator] **`DataType = '<SDT>'` without module now flagged when the SDT only exists inside a module.** Previously bare SDT names were silently accepted (cache stored simple name), so an AI-generated `DataType = 'SDTCFEParaEnsobrar'` would pass even though the SDT lived in `eFactura.TareasBatch`. Validator now checks the KB object index: if no match lives at the root, it reports `'SDTCFEParaEnsobrar' is in module 'eFactura.TareasBatch'. Use 'SDTCFEParaEnsobrar, eFactura.TareasBatch' instead.` SDTs at the KB root still accepted without module — no regression on existing patterns.

### Platform
- Bundle rebuilt with v1.8.29 plugin code: same shape as v1.7.20 (717 JS / 9 CSS / 39 JSON / 2.2 MB) but bytes differ — validator and analyzer compiled JS contains the new fixes.
- 0 VSIX drift.

### Tests
- 24/24 smoke tests pass.

### Bundle
- **Must be uploaded to `keygenapi.kbbridge.com`** — bytes differ from v1.7.20's bundle.

### VS Code 1.109.4
- No upstream changes

---

## 1.7.20 — 2026-06-07

**Plugins v1.8.28** | **VS Code 1.109.4**

> Critical macOS fix (follow-up to v1.7.19). After installing the ad-hoc-signed v1.7.19, users on Apple Silicon got past the "is damaged" error and the app launched — but the license-manager then failed with `KBbridge could not load extensions — Native addon missing decryptBundle function — rebuild required`. Root cause: the Rust napi-rs native addon (`kbbridge-loader`, used for AES-GCM bundle decryption and machine fingerprinting) was only ever built for Windows. The repo committed `kbbridge-loader.win32-x64-msvc.node` and the macOS/Linux workflows never built the equivalent for their platforms. Same bug exists on Linux but no Linux user reported yet. This release builds the native addon per-platform in CI and updates the require code to find any platform-specific `.node`.

### Platform

- Fixed: [build] **Per-platform native addon build in CI.** Each workflow (`kbbridge-windows.yml`, `kbbridge-linux.yml`, `kbbridge-macos.yml`) now has a `Setup Rust` + `Build native addon (kbbridge-loader)` step that runs `npm run build` (i.e. `napi build --platform --release`) and copies the resulting `kbbridge-loader.<platform>.node` to `kbbridge/license-manager/` so the embed step ships it with the install. File names produced:
  - `kbbridge-loader.win32-x64-msvc.node` (Windows x64)
  - `kbbridge-loader.darwin-arm64.node` (Mac Apple Silicon)
  - `kbbridge-loader.darwin-x64.node` (Mac Intel)
  - `kbbridge-loader.linux-x64-gnu.node` (Linux x64)
- Fixed: [license-manager] **`requireNativeAddon()` now finds any platform-specific `.node`**, not just the Windows variant. Globs `kbbridge-loader.*.node` in the extension dir and tries each. Previously hard-coded to `kbbridge-loader.win32-x64-msvc.node` → returned `null` on every non-Windows platform → `decryptBundle` not callable → "rebuild required" crash.
- Fixed: [license-manager] **`getMachineFingerprint()` consolidated**, uses the same `requireNativeAddon()` helper. On macOS/Linux the native addon's `getMachineFingerprint()` throws (Phase-1 marker: Windows-only) and the JS catch handler falls back to the software-only fingerprint (hostname + platform + arch + cpu model). The software fingerprint is stable per machine — different Macs produce different fingerprints.

### Why this only manifested now

- macOS users couldn't even *launch* the app pre-v1.7.19 (the "is damaged" error blocked them). So the missing-native-addon bug was hiding behind the Gatekeeper bug.
- Linux is affected too. No Linux user reported because the customer base is mostly Windows. After this release, Linux works as well.
- Windows was never affected because the committed `.node` matches `process.platform` exactly. The Windows workflow now also builds fresh in CI (was previously using the committed file) — keeps everything consistent and avoids drift between the Rust source and the shipped binary.

### Tests
- 24/24 smoke tests pass.
- The new CI step can't be tested locally (cross-platform builds need each platform's runner). Risk is low: `napi build` is widely used and documented; if it fails on a runner we see the error in CI before the install ships.

### Bundle / build
- No bundle change — v1.7.18 bundle (2.2 MB, plugins v1.8.28) is still current. **No re-upload needed.**

### Manual workaround for users on v1.7.19 today
None — v1.7.19's macOS install is fundamentally non-functional on the license side. They need v1.7.20.

### VS Code 1.109.4
- No upstream changes

---

## 1.7.19 — 2026-06-07

**Plugins v1.8.28** | **VS Code 1.109.4**

> macOS install fix. Users on **Apple Silicon (arm64) Macs** reported `"KBEditor" is damaged and can't be opened` on first launch. Cause: macOS on Apple Silicon **requires** every binary to carry at least an ad-hoc code signature; without ANY signature the OS rejects the binary as corrupt. Our CI never signed because we don't have an Apple Developer Program account ($99/yr). This release adds an ad-hoc signing step to the macOS build workflow that satisfies the mandatory-signing requirement without needing a real certificate.

### Platform

- Fixed: [build/macos] **Apple Silicon "is damaged" error on first launch.** New `Code-sign app (ad-hoc)` step in `.github/workflows/kbbridge-macos.yml` runs `codesign --force --deep --sign -` on `KBEditor.app` between the post-build modifications and `prepare_assets.sh`. Applies to both arm64 and Intel builds (Intel didn't strictly need it but it's harmless and prevents future Gatekeeper tightening).

### What changes for users

| Mac type | Before v1.7.19 | After v1.7.19 |
|---|---|---|
| Apple Silicon (M1/M2/M3/M4) | "is damaged and can't be opened" — could not launch | Opens after first-launch right-click → Open (standard "unidentified developer" flow) |
| Intel | "unidentified developer" warning — could open via right-click | Same as before (no regression), now also ad-hoc signed |

### What ad-hoc signing does NOT fix

- **Gatekeeper warnings remain** — users still see "unidentified developer" on first launch and must right-click → Open. The proper fix for this is Apple Developer Program + notarization (~$99/yr); deferred.
- **Mark-of-the-web quarantine attribute** — for users who still see "damaged" because their browser set `com.apple.quarantine` on the download:
  ```bash
  xattr -cr /Applications/KBEditor.app
  ```
  This clears the quarantine flag and lets the app open. We'll add this to the README's macOS install section.

### Pre-existing signing path
The existing Developer-ID + notarization logic in `prepare_assets.sh` (gated by `${CERTIFICATE_OSX_P12_DATA}`) is unchanged. If we ever set up an Apple Developer account, that path will activate and supersede the ad-hoc step automatically.

### Tests
- 24/24 smoke tests pass.
- Workflow change cannot be tested locally (macOS-only command). The change is small, well-scoped, and following Apple's documented `codesign -` pattern — risk is low.

### Bundle / build
- No bundle change — v1.7.18 bundle (2.2 MB, plugins v1.8.28) is still current. **No re-upload needed.**

### VS Code 1.109.4
- No upstream changes

---

## 1.7.18 — 2026-06-04

**Plugins v1.8.28** | **VS Code 1.109.4**

> Plugin sync: 6 versions consolidated (v1.8.23 → v1.8.28). Headline: **incremental validator cache** + **`Domain:` prefix on `DataType` fully supported**. The cache update means new Domains / Attributes / SDTs / External Objects created by AI assistants or by externalising a new module no longer require `Developer: Reload Window` — they're recognised on the next save. Plus 11 false-error fixes in validator/analyzer and 2 sync-status race conditions resolved.

### Plugins

#### v1.8.28 — 2026-06-03
- Added: [language-validator, visual-editor] **`Domain:<DomainName>` prefix on `DataType` is now fully supported across the stack.** In a KB with a Domain `Boolean` *and* the built-in `Boolean`, `DataType = 'Domain:Boolean'` now disambiguates: (1) data-type validator accepts it and checks the domain exists, (2) method validator strips the prefix and looks up the domain's base type so methods like `.Trim()` on a `Character`-based domain resolve correctly, (3) GX Properties parses the prefix into the Domain reference field and only serialises it back when the domain name collides with a built-in. Module-qualified form (`Domain:<Name>, <Module>`) deferred.

#### v1.8.27 — 2026-06-02
- Fixed: [sync-status] **Modifications to an object that KBBridge already exported in the same day are now correctly detected.** The export-write filter previously rejected ANY future modification of the file for 24 h. Now it compares against the export's actual timestamp with a 5-second tolerance — anything later registers for sync.
- Fixed: [sync-status] **No false "pending" entries for objects arriving only via KBBridge export, even when KBBridge runs on a separate server and the files reach the local workspace via git pull (or other delayed transport).** The filter now uses `max(entry timestamp, local mtime of kbbridge-*-exports.json)` with a 5-minute window — handles both local-KBBridge and server-KBBridge transparently.

#### v1.8.26 — 2026-06-01
- Fixed: [language-validator] **No false `Parameter '&Mode' expects type 'Numeric(4.0)' but got 'Character(3)'` warnings on `Call(&Mode, ...)` invocations** when the called Procedure/WebPanel uses built-in variables (`&Mode`, `&Pgmname`, `&Today`) as `parm()` parameters without declaring them in `#Variables`. Parameter parser now consults the built-in catalog.
- Fixed: [language-validator] **`&RestCode` recognised in Procedures exposed as REST WebServices** (previously only known inside API objects). Type also corrected to `Numeric(3.0)` to match HTTP status code standard.
- Fixed: [sync-status] **No false `'<TypeName>, <Module>' is not a valid data type` warnings on new types created by AI assistants or external editing.** Sync panel now consumes both `kbbridge-*-exports.json` AND `external-<user>-changes.json` with the same incremental policy — warnings clear on next save without `Reload Window`.
- Fixed: [language-validator] **No false `'AddItem' is not a valid method for type 'numeric'` warnings on `&Variable.AddItem(...)` calls** when the variable is a Combo Box / Dynamic Combo Box / Radio Button / List Box. Method validator now reads `ControlType` from the sibling `.gxForm` on BOTH property-access AND method-call branches (previously only property branch did this).
- Fixed: [language-validator, sync-status] **A Domain / Attribute / SDT / External Object created in the workspace is recognised immediately, without `Developer: Reload Window`.** Three issues fixed: (1) Sync and validator now use the same project-root convention (strip trailing `/Knowledge Base`), (2) cache key is case-insensitive on Windows and normalises drive letter to uppercase, (3) catalog update triggers re-validation of every open GeneXus document — stale warnings clear without manual edit.

#### v1.8.25 — 2026-05-31
- Fixed: [language-validator] **No false `Variable '&Mode' is not declared in #Variables section` warnings on WebPanels, Panels (Smart Device), WebComponents and WebMasterPages.** `&Mode` is a built-in `Character(3)` with values `INS`/`UPD`/`DLT`/`DSP` — previously only recognised for Transactions.
- Fixed: [language-analyzer] **No false `Mismatched input '.' expecting {')', ','}` errors on `call(Module.SubModule.ObjectName)`.** Qualified module paths now accepted by the parser.
- Fixed: [visual-editor] **Pattern Editor now renders the full tree of a `.gxPattern` instance even when its events contain `{` / `}` on their own line inside `<![CDATA[]]>` sections.** The brace counter now ignores braces inside CDATA (tracked across line boundaries).
- Fixed: [language-analyzer] **No false `no viable alternative at input '()'` on `Event &Variable.IsValid()` headers.** Both qualified-name (`Grid.Load()`) and variable-method (`&Var.IsValid()`) event headers now accept trailing parentheses.
- Fixed: [language-validator] **No false `'IsValid' is not a valid method for type 'X'` warnings on `Event &Variable.IsValid()` headers.** Validator now skips the entire header line of `Event ...` declarations — the identifier after `Event` is identity, not executable code.
- Fixed: [language-validator] **No false `'IsEmpty' is not a valid field or level of SDT '<SDT.Path>'` warnings on chains like `&MySDT.Collection.Count.IsEmpty()`.** Chain resolver now advances the tracked type to the property's return type on field-style access too (previously only method-style).
- Fixed: [language-validator] **No false `expects type 'Character' but got '<domain-name>'` warnings on call arguments like `Msg(&MySDT.Level.Field.ToString(), Status)` where the SDT field uses a custom domain.** Parameter type resolver now resolves the field's declared type to its underlying base type before looking up the next method in the chain.

#### v1.8.24 — 2026-05-25
- Fixed: [language-validator, sync-status] **Installing a new external module (e.g. `GeneXusCryptography`) and externalising it while KBEditor is open no longer requires `Developer: Reload Window` before the new types are recognised.** Validator picks up new Domains / Attributes / SDTs / External Objects incrementally as they appear on the KB Exports tab.
- Fixed: [language-validator] **Declaring a missing variable in `#Variables` now clears the `Variable '&X' is not declared` warning IMMEDIATELY on every line that uses `&X` (not just the line you just edited).** Same applies to type changes and deletions.

#### v1.8.23 — 2026-05-24
- Fixed: [language-validator] **No false `expects type 'Character' but got 'Numeric(4.0)'` errors when a call argument is a string concatenation that starts with a built-in GeneXus variable (`&Pgmname`, `&Today`, `&Now`, etc.).** Validator now reads the built-in's real type from the catalog instead of falling back to numeric default.

### Platform
- Bundle rebuilt with v1.8.28 plugin code: same shape as v1.7.17 (717 JS / 9 CSS / 39 JSON / 2.2 MB) but bytes differ — validator and sync-status compiled JS now contains the new fixes.
- No VSIX drift detected (`sync-vsix-skeletons.py` reports 0 patched).
- Core-lib restructure to `out/`-only that the plugin team pre-announced has NOT happened yet in v1.8.28 — our build pipeline remains compatible (we already read from `out/`).

### Tests
- 24/24 smoke tests pass.

### Bundle
- **Must be uploaded to `keygenapi.kbbridge.com`** — bytes differ from v1.7.17's bundle even though shape is identical.

### VS Code 1.109.4
- No upstream changes

---

## 1.7.17 — 2026-05-24

**Plugins v1.8.22** | **VS Code 1.109.4**

> Bug-fix sync. Single fix in `sync-status`: a forced externalization run from KBBridge Manager **while KBEditor was closed** would flood the Sync panel with thousands of fake "pending" entries on next open. Affects only users who externalize via the Manager with the editor not running — but for them it was very noisy. No new features, no UI changes.

### Plugins

#### v1.8.22 — 2026-05-23
- Fixed: [sync-status] **Forced bulk export that completes while KB Editor is closed no longer floods the Sync panel with fake pending entries on next open** — the startup `getLastBulkExportStatus()` check only acted on the `exporting` state. A `bulk-export-end` already on disk meant `status: 'idle'`, the startup branch did nothing, `changeTracker.bulkExportEndTime` was never set, and the first window-focus reconcile compared the pre-externalization file snapshot against the new filesystem state → every externalized file registered as "pending". Fix: the startup check now has a second branch — when `status === 'idle'` but the reader returns an `endTimestamp` (meaning a `bulk-export-end` / `sync-batch-end` is the most recent marker on disk), the extension primes `bulkExportEndTime` with `max(endTimestamp, fileMtime)` (same `effectiveEndTime` policy as the live `endBulkExportPause()`). First reconcile then filters out every file with `mtime <= effectiveEndTime` and only reports genuine manual edits made after the externalization finished. UTC timestamps end-to-end (Manager writes ISO 8601 with `Z`, `stat.mtimeMs` is UTC, `new Date(isoZ).getTime()` parses as UTC) — timezone never enters the comparison.

### Platform
- Bundle rebuilt with v1.8.22 plugin code: **same shape as v1.7.16** (717 JS, 9 CSS, 39 JSON, 2.2 MB). The sync-status fix is ~13 lines in `extension.ts`; no other file changes.
- No VSIX drift detected (`sync-vsix-skeletons.py --dry-run` shows 0 patched).
- **Bundle must be uploaded to `keygenapi.kbbridge.com`** — the bytes are different (contains the fix in sync-status' compiled JS).

### Tests
- 24/24 smoke tests pass.

### VS Code 1.109.4
- No upstream changes

---

## 1.7.16 — 2026-05-19

**Plugins v1.8.21** | **VS Code 1.109.4**

> Plugin sync: 3 versions consolidated. Headline feature is the **new `.gxLayout` visual editor** for GeneXus report layouts (PrintBlocks, controls with paper-relative positioning, copy/cut/paste between bands). Also: Show Rows/Show Cells overlay in `.gxForm`, columnsStyle/rowsStyle properties exposed in GX Properties, and `.gxLayout` recognized by the Sync engine for internalization.

### Plugins

#### v1.8.21 — 2026-05-19
- Added: [visual-editor] **Copy / Cut / Paste of controls between PrintBlocks on `.gxLayout`** — `Ctrl+C` / `Ctrl+X` / `Ctrl+V` plus right-click menu, multi-selection support. Deep snapshot in clipboard, fresh `RPT_INTERNAL_NAME` per pasted control (no collisions), `RPT_X`/`RPT_WIDTH` preserved, `RPT_Y`/`RPT_HEIGHT` clamped to destination band. Layouts marked `dynamic="true"` ignore paste (defense-in-depth).
- Added: [visual-editor] Right-click context menu on PrintBlock headers with band-level operations
- Added: [visual-editor] Multi-selection improvements on `.gxLayout` (shift-click, marquee, arrow-key move)
- Added: [visual-editor] Red overflow `!` indicator when a control extends past the paper's right edge
- Fixed: [visual-editor] 9 fixes around `.gxLayout` rendering, drag/drop precision, band resize, etc.
- Fixed: [sync-status] `.gxLayout` save is now recognized by the Sync engine for internalization (matches the existing `.gxSource` / `.gxForm` handling).
- Fixed: [language-analyzer], [language-core] minor parsing/recognition fixes

#### v1.8.20 — 2026-05-18
- Added: [visual-editor] **`columnsStyle` / `rowsStyle` properties in GX Properties** for tables in `.gxForm` — granular per-column / per-row CSS class declaration. Visualisation on the canvas reflects the declared classes.
- Added: [visual-editor] 4 other table-related improvements (default cell widths, alignment, etc.)
- Changed: [visual-editor] cleanup of property panel layout

#### v1.8.19 — 2026-05-17
- Added: [visual-editor] **`Show Rows` / `Show Cells` overlay in `.gxForm`** — toggle button on the canvas that overlays a grid showing the row/cell boundaries (with their internal `RowName` / `controlName`), so users can see the structural layout under the rendered preview. Independent toggles for rows and cells.
- Added: [visual-editor] 7 additional gxLayout editor refinements (band selection, ruler, snap-to-grid, etc.)
- Changed: [visual-editor] **Refactor of "Save Object As..."** to a more robust implementation that better handles `.gxLayout` and shared scenarios (the previous one had edge cases on certain object types).

### Platform
- Bundle rebuilt with v1.8.21 plugin code: **717 JS files** (was 707), **9 CSS** (was 7), **39 JSON** (was 32). Bundle size 2.0 MB → **2.2 MB**.
- VSIX skeletons resynced for `genexus-visual-editor` and `genexus-language-core` (both had new contributions / activationEvents for the gxLayout editor). Caught by smoke test `VSIX contributes in sync` and auto-fixed by `sync-vsix-skeletons.py` — no manual work.
- **Bundle must be uploaded to `keygenapi.kbbridge.com`** after this release.

### Tests
- 24/24 smoke tests pass.
- The drift-detection tests (added in v1.7.11) correctly caught the new contributions in visual-editor and language-core packages and the sync script fixed them in one run — exactly the workflow they were built for.

### VS Code 1.109.4
- No upstream changes

---

## 1.7.15 — 2026-05-19

**Plugins v1.8.18** | **VS Code 1.109.4**

> UX upgrade for the license status panel + small bug fix on recovery flow. The old `Help → View License` opened a 404 URL (`https://KBbridge.com/license`), and `KBEditor: Show License Status` was a minimal QuickPick. Both now open a single KBSync-style webview panel with logo, status/type/expires/key/name/email/company rows, and Buy License / Enter License Key buttons.

### Platform

- Added: [license-manager] **License status webview panel** (`media/license-status.html`) modeled after KBSync's License tab. Replaces the previous QuickPick. Renders Status (colored: green/orange/red), Type, Expires, Key (truncated for safety), Name, Email, Company, plus the 2 action buttons.
- Added: New helper `computeLicenseDisplay(data)` + `truncateLicenseKey()` centralize the date/status logic so callers don't reimplement it.
- Added: `getWebviewContent()` now accepts a `extraTokens` dict for runtime data interpolation, with HTML-escape on all non-passthrough fields (defense against `<` in user-typed name or company).
- Added: VSCodium source patch `patches/kbeditor-help-license-panel.patch` — modifies `OpenLicenseUrlAction.run()` to detect `command:` URIs and dispatch them via `ICommandService` instead of through `openerService.open()` (which silently drops `command:` URIs because `CommandOpener` requires explicit `allowCommands`).
- Changed: `prepare_vscode.sh` sets `product.licenseUrl` to `"command:kbbridge.showLicenseStatus"`. Combined with the patch, **Help → View License now opens the license panel** instead of a 404 URL.
- Fixed: [license-manager] Recovery flow was discarding the firstName/lastName/company fields the user typed in the registration form — only `email` was stored. The recovered license then showed "—" for those fields in the new panel. The Recover button now sends all 5 fields (name, last name, company, email, language); the handler stores them in `SECRET_KEY_DATA` alongside the recovered license key.

### How the 3 entry points map to the panel

| Click | Result |
|---|---|
| Status bar item (the trial/expiry indicator at the bottom) | Opens the panel |
| Command Palette → `KBEditor: Show License Status` | Opens the panel |
| Menu bar → Help → View License | Opens the panel (via the new patch) |

When no license is stored (fresh install / debug-clear), the entry points still show the existing "No License" QuickPick — there's no data to display yet, so a panel would be empty.

### Bundle / build
- No bundle change — v1.7.9 bundle (2.0 MB) is still current.
- The next CI build of Step 1 will apply the new patch and regenerate `workbench.desktop.main.js` with consistent checksums.

### VS Code 1.109.4
- No upstream changes

---

## 1.7.14 — 2026-05-12

**Plugins v1.8.18** | **VS Code 1.109.4**

> Follow-up to v1.7.13. The WASM fallback shipped — but `search_docs` still failed on the affected customer with `Error: no available backend found. ERR: [cpu] Error [ERR_MODULE_NOT_FOUND]: Cannot find module '...ort-wasm-simd-threaded.mjs'`. Root cause: `ort.node.min.js` defers loading its backend module (a `.mjs` file) until the first `InferenceSession.create()` call. v1.7.13 shipped only the `.js` and `.wasm` files — the `.mjs` wrappers that the bundle imports dynamically were never embedded. `list_topics` and `lookup_method` worked (no embeddings = no inference); `search_docs` blew up at first use. This release ships the missing `.mjs` files **and adds a runtime smoke test** that actually triggers the lazy import so this regression class can't slip through again.

### Platform

- Fixed: [embed] Ship the **3 missing `.mjs` files** under `onnxruntime-web/dist/`:
  - `ort-wasm-simd-threaded.mjs` — WASM module wrapper, lazy-imported by `ort.node.min.js` at first `InferenceSession.create()`
  - `ort-wasm-simd-threaded.jsep.mjs` — JSEP variant of the wrapper
  - `ort.node.min.mjs` — ESM variant of the Node bundle (declared in `package.json#exports.import`)
  Combined size: ~90 KB. The fix is a 3-line additions in `embed_extensions.sh`.

### Tests (regression coverage)
- Updated: `onnxruntime WASM fallback files in install` — required-files list extended with the 3 `.mjs` files. Catches the case where `embed_extensions.sh` is rolled back or someone trims the file list.
- Added: **`onnxruntime WASM backend actually initialises`** — new functional test that:
  1. Directly loads `onnxruntime-web/dist/ort.node.min.js`
  2. Points `env.wasm.wasmPaths` at the local `dist/`
  3. Calls `InferenceSession.create(new Uint8Array(8))` — intentionally invalid as a model, but the call still **triggers the lazy WASM backend init**.
  4. If we see `ERR_MODULE_NOT_FOUND` / "no available backend" in the error → fail (a `.mjs` file is missing). If we see a model-parsing error → pass (backend initialised, model was just bogus).
  This is the test that would have caught v1.7.13's bug. The previous shim runtime test only checked the require returned a module — it never exercised the actual WASM init code path.
- Confirmed by removing `ort-wasm-simd-threaded.mjs` and re-running: both file-presence test and runtime test fail with clear messages.
- Total smoke tests: 23 → **24**.

### Bundle
- No bundle change — v1.7.9 bundle (2.0 MB) is still current.

### What this fixes for the customer
Same WS2019 + VMware client from v1.7.13. After this release:
- MCP arrives `running on stdio` ✅ (already worked in v1.7.13)
- Shim picks WASM fallback ✅ (already worked in v1.7.13)
- `list_topics` / `lookup_method` work ✅ (already worked in v1.7.13)
- **`search_docs` works** ✅ (new — the lazy `.mjs` import resolves)

### VS Code 1.109.4
- No upstream changes

---

## 1.7.13 — 2026-05-12

**Plugins v1.8.18** | **VS Code 1.109.4**

> Real fix for the MCP `ERR_DLOPEN_FAILED` problem. The v1.7.12 diagnostic logging let the customer share the exact error: `A dynamic link library (DLL) initialization routine failed.` on `onnxruntime_binding.node`. Root cause: Windows Server 2019 (Build 17763) + VMware guest. `onnxruntime-node` 1.21 requires either DirectML (`dxcore.dll`, only on Win 10 Build 19041+ / Server 2022) or AVX CPU instructions — VMware doesn't expose AVX to guests by default. Without either backend the native binding's `DllMain` returns FALSE and the MCP crashes before registering its tools. **This release adds an automatic WASM fallback** so the MCP works in those environments out of the box.

### Platform

- Added: [mcp-server] **`onnxruntime-node` → `onnxruntime-web` (WASM) automatic fallback.** New shim at `kbbridge/mcp-server/scripts/onnx-fallback-shim.js` replaces `onnxruntime-node/dist/index.js` during embedding. On `require('onnxruntime-node')`, the shim:
  1. Tries the renamed original (`index-native.js`) first → fast path on machines with AVX or DirectML.
  2. On `ERR_DLOPEN_FAILED` / DLL-init / dlopen errors → falls back to `onnxruntime-web/dist/ort.node.min.js` with `env.wasm.wasmPaths` pointing at the shipped `.wasm` binaries and `numThreads=1` for maximum compatibility.
  3. On any other error → re-throws (so missing-file / syntax errors still surface instead of being masked by a slower fallback).
  Backend chosen is tagged on the module as `__kbbridgeBackend` and logged to stderr (captured by the v1.7.12 diag log).
- Added: [embed] **Ship the real `onnxruntime-web` package** (not the empty stub) — Node-targeted JS bundle + 2 `.wasm` binaries, ~32 MB total. Browser/WebGL/WebGPU artifacts dropped (~60 MB saved). Installer grows ~32 MB.
- Updated: [embed] `embed_extensions.sh` renames the original `onnxruntime-node/dist/index.js` to `index-native.js` and drops our shim in its place. Idempotent — re-running the embed doesn't re-rename if it already happened.

### Tests (regression coverage)
Three new smoke tests in `kbbridge/tools/smoke-test.py` (now **23/23**):
- **`onnxruntime fallback shim source intact`** — checks the shim file contains its 6 critical markers (native-first require, ERR_DLOPEN_FAILED detection, WASM target path, `wasm.wasmPaths` + `wasm.numThreads` config, backend tag). Quote-style-agnostic so cosmetic refactors don't break it. Catches any future regression where the shim is partially deleted or "simplified".
- **`onnxruntime WASM fallback files in install`** — verifies the local install at `upstream-vscodium/` has the real `onnxruntime-web` package (rejects `version=0.0.0` stub), the 3 required files (`ort.node.min.js` + 2 `.wasm`), and `onnxruntime-node/dist/index-native.js` exists (confirms embed renamed the original). SKIPs if the install dir isn't present (clean dev env).
- **`onnxruntime shim require works at runtime`** — spawns `node` inside the install, does `require('onnxruntime-node')`, and asserts InferenceSession + Tensor + env.wasm are present. End-to-end functional test of the shim.

### Docs
- Updated `kbbridge/docs/MCP_SERVER.md` §4 (Embedding Model) with the full native/WASM fallback rationale, the shim's decision flow, and the 3 regression tests guarding it.

### Bundle
- No bundle change — v1.7.9 bundle (2.0 MB) is still current.

### Manual workaround (no longer needed after v1.7.13)
Before this release, customers in restricted environments (Server 2019 + VMware) had to manually:
1. Extract `onnxruntime-web@1.21.0` into `C:\ort-web-extract\package\`.
2. Run a PowerShell script that copied the package over our stub and wrote a shim over `onnxruntime-node/dist/index.js`.
3. Re-apply the script after every KBEditor update.

v1.7.13 does all of that automatically as part of the build — no per-machine setup needed.

### VS Code 1.109.4
- No upstream changes

---

## 1.7.12 — 2026-05-12

**Plugins v1.8.18** | **VS Code 1.109.4**

> Diagnostic release for the MCP server. A user reported the MCP keeps failing on their machine, but **VS Code 1.109.x's MCP API doesn't surface the server's stderr in any visible Output channel**, so we couldn't see what's wrong. This release adds **file-based logging** from the MCP server itself plus two diagnostic commands so a user can grab the log with one click and send it back.

### Platform

- Added: [mcp-server] **File-based diagnostic logging** from the very first line of the MCP server. Writes to `<globalStoragePath>/mcp-server.log` (path passed via the new `KBBRIDGE_MCP_LOG_FILE` env var). Truncated on each run so the file always reflects the latest spawn. The file's presence/absence is itself a diagnostic signal: missing entirely → process never spawned; empty → died before first log; truncated mid-log → died at that point.
- Added: [mcp-server] `uncaughtException` and `unhandledRejection` handlers logging full stack traces. Catches transformers / lancedb / onnxruntime crashes that would otherwise be silently swallowed by VS Code's child-process management.
- Added: [license-manager] Command **"KBEditor: Show MCP Server Log"** — opens the diagnostic log in the editor, one click for the user to send it to support.
- Added: [license-manager] Command **"KBEditor: Test MCP Server (Diagnose)"** — spawns the MCP server *directly* (bypassing VS Code's MCP API) and pipes its stdout/stderr to an Output channel for 5 seconds. Distinguishes "MCP can't even start" from "MCP starts but VS Code never spawns it".
- Refactored: [mcp-server] Logging moved to new `diag-log.ts` module shared by `setup-onnx.ts` and `index.ts`. Each log line is timestamped and tagged with a category (`[ONNX]`, `[MCP]`).

### Plugins
- Same as v1.7.11 (plugins v1.8.18).

### Bundle
- No bundle change — v1.7.9 bundle (2.0 MB) is still current.

### How to use the new diagnostics

When a user reports the MCP isn't working:

1. Ask them to run **"KBEditor: Show MCP Server Log"** from the Command Palette.
   - If the log opens with `[ONNX]` / `[MCP]` lines → we see exactly where it crashed.
   - If the warning "MCP server has not spawned yet" appears → VS Code never invoked the MCP. Ask them to trigger an AI agent search first, then re-run the command.
2. If the log shows nothing useful, ask them to run **"KBEditor: Test MCP Server (Diagnose)"** — spawns the process manually with our own stderr capture. Confirms whether the MCP server can run at all on that machine.

### VS Code 1.109.4
- No upstream changes

---

## 1.7.11 — 2026-05-12

**Plugins v1.8.18** | **VS Code 1.109.4**

> Hardening release on top of v1.7.10. Adds **automated tests + a sync tool** so the VSIX-skeleton-drift bug (which produced the Form Editor redirect loop) can't ship again unnoticed. Also re-syncs all 7 VSIX skeletons against the latest plugin source (the 5 we hadn't touched since April had `devDependencies` / `scripts` cruft we now strip).

### Platform

- Added: [tooling] `kbbridge/tools/sync-vsix-skeletons.py` — one-command resync of every `kbbridge/vsix/*.vsix` against its matching `package.json` in the plugin source. Strips `devDependencies` / `scripts` (build-time only), preserves the "KBEditor: " branded `displayName` prefix. Use `--dry-run` to preview changes; CI-friendly exit code (1 if any drift detected).
- Added: [test] `smoke-test.py` now has **2 new tests** that catch VSIX skeleton drift before commit:
  - `VSIX activationEvents match plugin source` — was previously a silent "warn only" since v1.6.x; now fails if the plugin's source declares events the VSIX is missing.
  - `VSIX contributes in sync (customEditors/commands/views/menus)` — new test that compares every customEditor `viewType`, command `command`, view `id`, and per-menu-category entry counts between VSIX and source. Reports each drift with a clear remediation hint.
- Tests grew from 18 to 20.
- All 7 VSIX skeletons resynced via the new tool (5 had unstripped devDependencies/scripts; 2 had been hand-patched in v1.7.10 without the displayName branding).
- No bundle change — v1.7.9 bundle (2.0 MB) is still current.

### How this prevents future drift

The combination is intentional:
- **smoke-test.py** runs before every push (per the release checklist). Drift now blocks commits instead of being silently shipped.
- **sync-vsix-skeletons.py** is the one-command fix when a smoke test fires (also runnable as a pre-flight before bumping versions).
- The plugin team adding new commands / customEditors / views in their `package.json` will surface in our next `smoke-test.py` run, with a clear pointer to run the sync.

### VS Code 1.109.4
- No upstream changes

### MCP
- MCP server unchanged from v1.7.6.

---

## 1.7.10 — 2026-05-12

**Plugins v1.8.18** | **VS Code 1.109.4**

> Follow-up to v1.7.9. The visual-editor activated correctly but the **Form Editor entered an infinite redirect loop** when opening any `.gxForm` file. Cause: the VSIX skeletons (`kbbridge/vsix/*.vsix`) had not been re-built since the plugin team added new contributions to `package.json` — the visual-editor's VSIX skeleton was missing the `genexus.formEditor` `customEditor` declaration, 3 commands, 1 view, and several `activationEvents`. Similarly, `genexus-language-core`'s VSIX was missing 6 commands and 2 menus (including "Find All References" / "Find Referenced" from v1.8.15). This release patches both stale VSIX skeletons by re-synchronising their `package.json` with the plugin source.

### Platform

- Fixed: [vsix] **`genexus-visual-editor` VSIX skeleton resynced with plugin source.** Now declares the `genexus.formEditor` custom editor (for `.gxForm` files), the `genexus.toolbox` view, 3 additional commands (including `genexus.openFileWithEditor`), 2 menu contributions, and the `onCustomEditor:genexus.formEditor` activation event. Without these, `.gxForm` files were opening as text and the extension's auto-redirect handler entered an infinite loop trying to switch to a custom editor that VS Code didn't know about.
- Fixed: [vsix] **`genexus-language-core` VSIX skeleton resynced with plugin source.** Now declares 6 additional commands and 2 menu contributions (notably the `Find All References` / `Find Referenced` entries from plugin v1.8.15 that never appeared in the right-click menu before).
- No bundle change — the v1.7.9 bundle (2.0 MB with the esbuild htmlparser2 fix) is still current.

### VS Code 1.109.4
- No upstream changes

### MCP
- MCP server unchanged from v1.7.6.

---

## 1.7.9 — 2026-05-11

**Plugins v1.8.18** | **VS Code 1.109.4**

> Follow-up to v1.7.8. The previous release added `htmlparser2` to the bundle but the v10+ release of that package ships as **pure ESM** (`"type": "module"`). Our bundle loader uses `Module._compile()` (CJS-only), so `require('htmlparser2')` from the visual-editor's compiled CJS code crashed with `Unexpected module status 0. Cannot require() ES Module...`. This release pre-bundles htmlparser2 + its 5 transitive ESM deps into a single CJS file via esbuild, sidestepping the ESM/CJS mismatch entirely.

### Platform

- Fixed: [build] **`htmlparser2` is now pre-bundled with esbuild** to a single CommonJS file with all 5 transitive deps inlined (`domelementtype`, `domhandler`, `domutils`, `dom-serializer`, `entities`). The 6 ESM packages no longer ship as separate entries — they're all rolled into `node_modules/htmlparser2/index.js` (~140 KB CJS) accompanied by a stripped `package.json` that omits `"type": "module"`. The bundle loader resolves `require('htmlparser2')` cleanly as CJS.
- Added: [build] `build-bundle.py` now supports an `esbuild_bundles` config field on each extension. Used for ESM third-party deps that can't be loaded via `Module._compile`. Run uses esbuild from the mcp-server's node_modules (already shipped in the repo).
- Bundle: 2.1 MB → 2.0 MB (esbuild inlines deps efficiently; we drop ~38 separate files in favor of one 140 KB bundle).

### Plugins
- Same as v1.7.8 (plugins v1.8.18 with the dynamic `.gxForm` lock feature).

### Bundle
- Rebuilt: 2.0 MB.
- **Must be uploaded to `keygenapi.kbbridge.com`** — replaces the v1.7.8 bundle (which had the ESM htmlparser2 problem).

### VS Code 1.109.4
- No upstream changes

### MCP
- MCP server unchanged from v1.7.6.

---

## 1.7.8 — 2026-05-11

**Plugins v1.8.18** | **VS Code 1.109.4**

> Critical platform fix. The visual-editor extension was crashing during bundle activation with `Cannot find module 'xslt-processor'`, especially visible on system installs (`C:\Program Files\KBbridge\`). Root cause: the `xslt-processor` and `htmlparser2` runtime dependencies (added in plugin v1.8.x for User Control XSL rendering) were never included in our extensions bundle — `build-bundle.py` declared `"third_party": []` for the visual-editor. On user-writable installs the bug was also present but went unnoticed because nobody opened a `.gxForm`. Includes all features from v1.7.7.

### Platform

- Fixed: [build] **Visual-editor runtime third-party dependencies are now included in the bundle.** Added `xslt-processor`, `htmlparser2`, and their 5 transitive runtime deps (`domelementtype`, `domhandler`, `domutils`, `dom-serializer`, `entities`) to `third_party` in `kbbridge/tools/build-bundle.py`. Bundle grew from 1.8 MB to 2.1 MB; visual-editor file count: 145 → 183.
- Fixed: [license-manager] **Require hook now resolves `package.json#main` from the in-memory sourceCache.** Previously the hook only tried `<base>.js` and `<base>/index.js` before falling through to Node's normal resolver — which doesn't work on read-only system installs where files can't be written to disk. The 7 new third-party packages all declare `main` as `"dist/index.js"`, so the simple lookup would miss them and Node's fallback would also fail (no files on disk). The hook now reads `<base>/package.json` from sourceCache, parses the `main` field, and resolves against sourceCache directly. Prevents the same class of bug in the future.

### Plugins
- Same as v1.7.7 (plugins v1.8.18 with the dynamic `.gxForm` lock feature).

### Bundle
- Rebuilt: 2.1 MB (was 1.8 MB).
- **Must be uploaded to `keygenapi.kbbridge.com`** — the v1.7.5 bundle on the server does NOT have the third-party deps and will continue to crash the visual-editor extension on clients.

### VS Code 1.109.4
- No upstream changes

### MCP
- MCP server unchanged from v1.7.6 (the `process.release.name` patch is in place).

---

## 1.7.7 — 2026-05-11

**Plugins v1.8.18** | **VS Code 1.109.4**

> Plugin code update: the upstream `v1.8.18` entry was extended with a new feature for protecting dynamic `.gxForm` files (no version bump on the plugin side). Bundle rebuilt to incorporate it. Includes all Platform fixes from v1.7.6.

### Plugins

#### v1.8.18 — additional feature (appended 2026-05-09)
- Added: [visual-editor] **`.gxForm` files marked as `dynamic="true"` in the root tag are now protected from manual edits with the same UX as `[dynamic]` sections of `.gxSource`** — when KBBridge externalizes a form whose content is maintained by a pattern, it injects `dynamic="true"` on the form's root element (`<BODY>` for HTML layouts, `<GxMultiForm>` for MultiForm, `<layouts>` for Smart Device Panel). The visual editor now detects this attribute and locks the form:
  - Canvas paints the selection outline in yellow (instead of the regular accent color)
  - Banner at the top reads "Form is dynamic — content maintained by KBBridge" with an "Unlock Form" button
  - GX Properties renders every input as read-only and shows the same banner
  - GX Toolbox dims its items so they cannot be dragged in
  - Any mutation attempt (drag, drop, right-click cut/paste/delete, keyboard shortcuts) is intercepted at two layers: webview shows a reduced context menu ("Unlock form to edit"); extension confirms with "This form is marked as [dynamic] and is maintained by KBBridge. Do you want to remove the dynamic tag and edit manually?" (Yes / No)
  - Choosing "Yes" flips the attribute to `dynamic="false"` and re-renders all three panels editable
  - Forms without `dynamic="true"` behave exactly as before — the lock is opt-in via the file content itself

### Platform
- All Platform fixes from v1.7.6 carry over: MCP server `process.release.name` patch (fixes `Unsupported device: "cpu"` crash on some Electron builds); Buy License URL now opens the correct page with the user's saved registration data pre-filled.
- Bundle rebuilt with the new dynamic-form-lock plugin code: 707 JS files, 1.8 MB.
- Bundle must be uploaded to `keygenapi.kbbridge.com` after this release.

### VS Code 1.109.4
- No upstream changes

### MCP
- MCP server unchanged from v1.7.6 (the `process.release.name` fix is in place).

---

## 1.7.6 — 2026-05-11

**Plugins v1.8.18** | **VS Code 1.109.4**

> Platform fix release. Two Platform bugs reported by users: (1) the MCP server crashed on some clients with `Unsupported device: "cpu"` because their Electron build reports `process.release.name === 'electron'` instead of `'node'` under `ELECTRON_RUN_AS_NODE=1`, causing `@huggingface/transformers` to take the web/wasm branch (which used our empty `onnxruntime-web` stub); (2) the "Buy License" button from the expired-license dialog and the status bar quick pick opened `https://kbbridge.com/pricing` (a non-existent page) instead of `https://kbbridge.com/buy-license-kbeditor?<params>` with the user's saved registration data.

### Platform

- Fixed: [mcp-server] MCP server no longer fails with `Unsupported device: "cpu"` on Electron builds that report `process.release.name === 'electron'`. A new `setup-onnx.ts` module patches `process.release.name` to `'node'` (with three fallback strategies: direct assignment, `Object.defineProperty`, replacing `process.release`) before `@huggingface/transformers` is loaded. This makes transformers take its complete Node branch — registering the `cpu` execution provider correctly. Diagnostic `[ONNX]` lines are written to stderr for future debugging.
- Fixed: [license-manager] "Buy License" button (from the expired-license dialog and the status bar quick pick) now opens the correct URL `https://kbbridge.com/buy-license-kbeditor?<params>` with the user's saved registration data (firstName, lastName, company, email, language, fingerprint, version, platform, arch) pre-filled — same URL the Buy button on the registration panel uses. Previously it opened `https://kbbridge.com/pricing` (a page that does not exist on the site). The URL construction was extracted to a shared helper `openBuyLicensePage(userData)` plus `openBuyLicensePageFromSecrets()` for callers that don't have userData in hand. The dead `PRICING_URL` constant was removed.
- No bundle change in this release — the v1.7.5 bundle (1.8 MB, plugins v1.8.18) is still current and does not need to be re-uploaded.

### VS Code 1.109.4
- No upstream changes

### MCP
- MCP server has the onnxruntime backend fix described above. No change to the registration command (`ELECTRON_RUN_AS_NODE=1`, same path as before).

---

## 1.7.5 — 2026-05-08

**Plugins v1.8.18** | **VS Code 1.109.4**

> Bug-fix release consolidating 3 plugin versions (v1.8.16, v1.8.17, v1.8.18). Notable: pattern-instance saves no longer corrupt CDATA blocks / self-closing tags / XML declaration; `.gxForm` round-trips bit-perfect (empty `git diff` on no-op save); duplicate `Pending` entries on saves fixed (Windows path-casing race); several analyzer false errors resolved.

### Plugins

#### v1.8.18 — 2026-05-07
- Fixed: [visual-editor] Adding a control directly into a Smart Device Panel table now generates the auto-wrapped row and cell with their required `id` GUIDs (the IDE was rejecting those rows/cells on re-import)
- Fixed: [visual-editor] Empty attributes of `.gxForm` controls are now serialized as `name=""` instead of bare `name` (HTML5 minimization form is invalid XML and the IDE rejected the file on re-import)
- Fixed: [visual-editor] Whitespace between `<layouts>` and `<layout>` (and other structural wrappers) is preserved on save — a no-op save now produces an empty `git diff`
- Changed: [sync-status] Depuration of the changes file no longer removes entries still in `pending` state — the periodic cleanup (`gxSync.changesMaxAgeDays` / `gxSync.changesMaxEntries`) now consults `external-<user>-status.json` and keeps every entry whose status is `pending` or unset, regardless of age or count

#### v1.8.17 — 2026-05-06
- Fixed: [sync-status] Saving a single GeneXus object no longer registers two `Pending` entries — caused by a Windows path-casing race (`C:\Foo` vs `c:\foo`) between the editor save event and the workspace filesystem watcher. Dedup is now case-insensitive on Windows with a 1.5 s window
- Fixed: [visual-editor] Saving a `.gxPattern` instance no longer rewrites large parts of the file — preserved across round-trips: `<?xml version="1.0" encoding="utf-16"?>` header, self-closing tags (`<variable />`, `<parameter />` etc.), `<![CDATA[...]>` blocks for code properties, literal `'` in attribute values, line endings (CRLF/LF), and the multi-line `[ ... ]` properties block of `PatternInstance`. The serializer now reads the schema XML (`PXWorkWithInstance.xml`) per property to know whether to emit as Attribute, CDATA, or Element. A no-op save produces an empty `git diff`

#### v1.8.16 — 2026-05-05
- Fixed: [visual-editor] Group header caption no longer shown twice on the canvas — the duplicate stray TextBlock-like child render is suppressed; underlying `<LEGEND>` element is preserved
- Fixed: [language-analyzer] No more false "missing '(' at '<Type>'" errors on `New <SDTType>(...)` or `New <SDTType>.<Level>(...)`
- Fixed: [language-analyzer] No more false "Mismatched input 'Sub'" errors inside platform-specific blocks of `#Events` (e.g. `[Web] { ... Sub 'Name' ... EndSub ... }`)
- Fixed: [language-analyzer] `Stub <Name>(<params>) ... EndStub` is now recognized as a valid procedural block in WebService Procedures
- Added: [language-core] Outline view, Breadcrumbs and `Go to Symbol in File` (`Ctrl+Shift+O`) now list every Event, Sub and Stub of a `.gxSource` (with kind prefix and full signature, e.g. `Event Grid1.Load`, `Sub 'CheckSecurityForActions'`, `Stub EnviarCFE(in: &EnviarCFEIn, out: &EnviarCFEOut)`)
- Fixed: [language-validator] No more false "is not a valid field or level of SDT" errors when accessing a sub-level of a collection item (e.g. `&Item.<SubLevel>.<Field>` where `<Item>` is the collection's `CollectionItemName`)

### Platform
- Bundle rebuilt with v1.8.18 plugin code: 707 JS files, 1.8 MB.
- Bundle must be uploaded to `keygenapi.kbbridge.com` after this release.

### VS Code 1.109.4
- No upstream changes

### MCP
- MCP server unchanged; verified Electron + `ELECTRON_RUN_AS_NODE=1` in place

---

## 1.7.4 — 2026-05-05

**Plugins v1.8.15** | **VS Code 1.109.4**

> Platform fix. Eliminates a Node `DEP0128` deprecation warning that surfaced in user logs (`Invalid 'main' field in '...@kbbridge/genexus-language-core-lib/package.json' of 'out/index.js'`). Functionality was unaffected because Node fell back to legacy resolution, but the warning will become a hard error in future Node versions.

### Platform
- Fixed: `embed_extensions.sh` was generating `package.json` files for the embedded `@kbbridge/genexus-language-core-lib` with `"main": "out/index.js"`, but the bundle places the compiled JS at the root of that directory (no `out/` subfolder). Changed to `"main": "index.js"`. Also patched the existing local staging copies for consistency.
- No bundle change in this release — bundle from v1.7.3 (1.8 MB, plugins v1.8.15) is still current.

### VS Code 1.109.4
- No upstream changes

### MCP
- MCP server unchanged; verified Electron + `ELECTRON_RUN_AS_NODE=1` in place

---

## 1.7.3 — 2026-05-05

**Plugins v1.8.15** | **VS Code 1.109.4**

> **Security release.** Both findings from our v1.7.2 deep code review (XSS in the Values editor, missing XML-escaping in packed properties) have been confirmed fixed by the plugin team. Also adds Free Style Grid support and Find All References / Find Referenced from the right-click menu.

### Plugins

#### v1.8.15 — 2026-05-04
- **Security:** [visual-editor] Values editor of Combo Box / Radio Button / List Box no longer renders pairs through unsafe HTML interpolation — values are now treated as plain text in every cell. Fixes the XSS finding from our v1.7.2 review (`properties-panel.ts:3849-3852`)
- **Security:** [visual-editor] Editing a property of a control on Smart Device Panel / Abstract Layout no longer corrupts other properties when any contains `<`, `>` or `&` — the wrapper that stores grouped properties (`PATTERN_ELEMENT_CUSTOM_PROPERTIES`) now XML-escapes name/value individually. Fixes the XML-escaping finding from our v1.7.2 review (`CustomPropertiesDecomposer.ts:163-167, 203-205`)
- Added: [visual-editor] **Free Style Grid (`<gxFreeStyle>`) is a first-class control in HTML forms** — renders correctly with green border and yellow tint, exposes 24 specific properties (CustomRender, AllowDrop / AllowDrag, RenderingMode, Border, Empty Grid Text, Paging, Allow Collapsing, Record / Page counters, etc.), insertable from the Toolbox, drops of any control on it are redirected to the inner `<TD>` automatically. Abstract / SDPanel forms not covered in this iteration
- Added: [language-core] **Find All References / Find Referenced from the right-click menu** of any `.gxSource` / `.gxAttribute` / `.gxDomain` — both on the editor tab and on the file in the Explorer. Lists every file that mentions the object (Find All References) and every defining file of the objects this one uses (Find Referenced). Reuses the reference index built at startup — instantaneous after first build

### Platform
- Bundle rebuilt with v1.8.15 plugin code: 707 JS files, 1.8 MB.
- Bundle must be uploaded to `keygenapi.kbbridge.com` after this release.

### VS Code 1.109.4
- No upstream changes

### MCP
- MCP server unchanged; verified Electron + `ELECTRON_RUN_AS_NODE=1` in place

---

## 1.7.2 — 2026-05-04

**Plugins v1.8.14** | **VS Code 1.109.4**

> Bug-fix release. Notable: a regression that rebuilt the reference index from scratch on every reload (causing several minutes of editor freeze on 10k-object KBs) is fixed. Several validator / analyzer false errors are also resolved.

### Plugins

#### v1.8.14 — 2026-05-03
- Fixed: [language-core] Reference index of a large Knowledge Base no longer rebuilds from scratch on every "Developer: Reload Window" — the persisted index was being rejected by a hardcoded version check. Several minutes of editor freeze gone on 10k-object KBs from the second reload onwards
- Fixed: [language-validator] No more false "is not a valid data type" errors on `DataType = 'Attribute:<AttrName>'` when the referenced attribute lives in an external module
- Fixed: [language-analyzer] No more false "Mismatched input '=' expecting '('" errors on `<ControlName>.<Property> = <value>` lines inside `#Rules` (e.g. `EmisorId.Visible = False;`)
- Fixed: [language-validator] No more false "Udp call requires last parameter to be mode 'out' or 'inout'" errors on calls to a DataProvider — DataProviders declare their output via `Output = '<Type>'` / `Collection`, not via `parm()`. The parser now synthesizes a virtual `out` parameter for DP calls
- Fixed: [language-validator] No more false "is not a valid method" errors on `IsEmpty()` / `SetEmpty()` / `ToString()` of variables typed by an enumerated domain
- Fixed: [language-validator] No more false "Parameter requires a variable (mode: inout)" warnings on calls that pass `<Control>.<Property>` (or any dotted lvalue) as an `out` / `inout` argument
- Fixed: [language-analyzer] No more false "Mismatched input '{' expecting {'.', '='}" errors on DataProvider blocks whose nested SDT level names are GeneXus keywords (e.g. `Display`, `Group`, `Default`, `Domain`, `Attribute`, `Insert`, `Update`, `Delete`)
- Changed: [visual-editor] The `Title` property of a Grid Column is now shown at the top of GX Properties (next to `Attribute`), matching the GeneXus IDE order

### Platform
- Bundle rebuilt with updated plugin code: 707 JS files, 1.8 MB.
- Bundle must be uploaded to `keygenapi.kbbridge.com` after this release.

### VS Code 1.109.4
- No upstream changes

### MCP
- MCP server unchanged; verified Electron + `ELECTRON_RUN_AS_NODE=1` in place

---

## 1.7.1 — 2026-05-03

**Plugins v1.8.13** | **VS Code 1.109.4**

> Plugin code update: the upstream `v1.8.13` entry was extended with 17 additional fixes and features (no version bump on the plugin side). Bundle rebuilt to incorporate them.

### Plugins

#### v1.8.13 — additional fixes (appended 2026-05-03)
- Added: [visual-editor] The `Values` property of a Combo Box / Radio Button / List Box now opens a visual editor with two columns (Name, Value), with add / remove / reorder, validations and serialization to the `Name1:Value1,...` format used in the `ControlValues` XML attribute
- Added: [visual-editor] `Checked Value` and `Unchecked Value` properties of a Check Box are now shown in GX Properties, written to `ControlCheckValue` / `ControlUnCheckValue` (HTML) or packed in `PATTERN_ELEMENT_CUSTOM_PROPERTIES` (Abstract / MultiForm / SDPanel)
- Added: [visual-editor] Keyboard shortcuts in the `Values` editor — `Alt+A` add, `Delete` remove, `Alt+↑` / `Alt+↓` reorder, `↑` / `↓` select, `Enter` edit
- Fixed: [visual-editor] Radio Button now renders as a real radio control on the canvas even when values cannot be deduced from a domain or source code
- Fixed: [visual-editor] Combo Box, Radio Button and List Box now render their static `ControlValues` from the XML on the canvas (previously only enum-domain or `&Var.Add(...)` values were used)
- Fixed: [visual-editor] List Box now renders as a multi-line scrollable list (`<select size>`), distinguishable from Combo Box. Same applies to Dynamic List Box
- Fixed: [visual-editor] Combo Box on the canvas now shows the empty item declared by `Add Empty Item` / `Empty Item Text`
- Fixed: [visual-editor] Drag from the GX Toolbox to the form canvas now works regardless of which tab is active (the activation message is broadcast to every open form-editor webview)
- Fixed: [visual-editor] Drag from the GX Toolbox no longer fails when the form-editor webview was suspended by VSCode — visible panels are explicitly woken up on drag start (`panel.reveal` with `preserveFocus: true`)
- Fixed: [visual-editor] Dragging a row inside a nested table now reorders within that table instead of selecting the parent table — handler retries `inside` when `before`/`after` is rejected
- Fixed: [visual-editor] Drop position of a drag inside a nested table now matches the visual indicator (the `drop` handler now applies the same retry logic as `dragover`)
- Fixed: [visual-editor] The horizontal "drop here" line between rows is back when dragging a row by its handle — `dragover`/`drop` handlers retry `before` / `after` when the original `inside` is rejected
- Fixed: [visual-editor] Reordering rows in HTML tables now respects the `<TBODY>` wrapper — canvas ascends through `<TBODY>` / `<THEAD>` / `<TFOOT>` when computing the effective parent, and row inserts targeting `<TABLE>` directly are redirected into its `<TBODY>`
- Fixed: [visual-editor] Drop of a row inside a `<TBODY>` now actually completes the move — `acceptsAsChild` validator now treats `<TBODY>` / `<THEAD>` / `<TFOOT>` as equivalent to `<TABLE>` for child acceptance
- Fixed: [visual-editor] Reordering Grid Columns now shows a vertical drop indicator between columns during the drag (instead of a horizontal one)
- Fixed: [visual-editor] HTML `<DIV>` containers no longer stretch to fill the available height and hide their next siblings — User Control regions still fill height, HTML `<DIV>` grows to fit content
- Changed: [visual-editor] "Delete Layout" confirmation dialog of a Smart Device Panel now shows the layout's readable name (e.g. `Default`, `iPhone · Portrait`) instead of its internal GUID

### Platform
- Bundle rebuilt with updated plugin code: 707 JS files across 7 extensions; bundle size 1.8 MB.
- Bundle must be uploaded to `keygenapi.kbbridge.com` after this release.

### VS Code 1.109.4
- No upstream changes

### MCP
- MCP server unchanged; verified Electron + `ELECTRON_RUN_AS_NODE=1` in place

---

## 1.7.0 — 2026-05-02

**Plugins v1.8.13** | **VS Code 1.109.4**

> **Major plugin update**: introduces the **Visual Form Editor** for `.gxForm` files (WYSIWYG rendering, drag & drop, GX Properties panel, responsive layouts, multi-layout support for Smart Device Panels, User Control discovery and rendering). 14 plugin versions consolidated since the last KBEditor release.

### Plugins

#### v1.8.13 — 2026-05-02
- Fixed: [visual-editor] `Control Name` of a `<table>` in Abstract Layout / MultiForm / Smart Device Panel now shows the actual control name instead of the internal GUID
- Fixed: [visual-editor] Tables, rows, cells, sections, grids, tabs, tab pages, grid columns and groups inserted into Smart Device Panels now carry the `id="<GUID>"` attribute that the GeneXus IDE writes
- Fixed: [visual-editor] The "Group" container in Abstract Layout / Smart Device Panel now serializes as `<group>` (was `<section>`)
- Changed: [visual-editor] Cleanup of legacy properties when changing Control Type to a different User Control is now near-instant (single-pass instead of O(n²))
- Changed: [visual-editor] Cloning a layout via "Add Layout (Copy from existing)" no longer regenerates the `id` of internal regions of User Controls
- Changed: [visual-editor] Smart Device Panel detection now uses the XML root tag (`<layouts>`) instead of the file path
- Added: [visual-editor] Output channel logs a warning when a Smart Device Panel is opened with structural nodes missing `id="<GUID>"`

#### v1.8.12 — 2026-05-01
- Added: [language-core, visual-editor] One-click navigation between a `.gxSource` and its matching `.gxForm` (clickable link, title-bar icon, "Source" button in Form Editor)
- Added: [visual-editor] Multi-Layout support for Smart Device Panels — tabs at the top of the canvas switch between layouts; each layout edited independently
- Added: [visual-editor] "Add Layout" wizard with cascading variant selectors (Platform → Device Kind → Size → Version → Orientation) plus "Empty / Copy from existing"
- Added: [visual-editor] Adaptive overflow for the layout switcher — folded into a `…` button when toolbar width is insufficient
- Added: [visual-editor] Right-click "Delete Layout" on a layout tab (last layout cannot be deleted)
- Changed: [visual-editor] "Text" button label now reflects the form's real format ("HTML" for Layout HTML, "XML" for Abstract / MultiForm)

#### v1.8.11 — 2026-04-30
- Added: [visual-editor] User Controls can be inserted from the Toolbox in Abstract Layout and MultiForm forms (initial container regions of `gxui.Layout`, `gxui.Panel` etc. added automatically)
- Added: [visual-editor] Responsive Sizes visual editor available from PXTools pattern instances (PXWorkWith, PXComposer, PXParameterRequest)
- Fixed: [visual-editor] Breakpoint selector (XS / SM / MD / LG) now appears for every responsive form, even when no table has explicit responsive sizes yet
- Fixed: [visual-editor] Responsive Sizes editor never opens with empty data (fallback path removed; timeout extended to 10s)
- Fixed: [visual-editor] Canvas stacks responsive cells mobile-first at XS even when the table has no `responsiveSizes` JSON
- Fixed: [visual-editor] Responsive Sizes modal closes automatically when the user selects a different control in the canvas
- Fixed: [visual-editor] GX Properties clears when the active tab changes
- Fixed: [visual-editor] `.gxForm` files reliably open with the visual Form Editor (extension auto-adds `.gxForm` and `.gxPattern` mappings on startup)
- Fixed: [visual-editor] User Controls with container regions render correctly in Abstract Layout and MultiForm forms
- Fixed: [visual-editor] User Controls in Abstract Layout and MultiForm matched correctly even when the instance was renamed

#### v1.8.10 — 2026-04-29
- Added: [visual-editor] User Controls can be inserted from the Toolbox in HTML Layout forms (drag or double-click)
- Added: [visual-editor] Inserting a User Control container creates its initial regions automatically (parsed from the UC's XSL render)
- Added: [visual-editor] `Control Type` dropdown of an Attribute / Variable now lists every external User Control with `<IncludeInControlInfo>true</IncludeInControlInfo>` (Chosen, CKEditor, Rating, RunJS, SDCalendar, etc.)
- Added: [visual-editor] Properties of the chosen User Control are merged into the Attribute / Variable panel
- Added: [language-core] Closing a tab opened from a `.gxChilds` re-opens the QuickPick automatically
- Added: [sync-status] Support for `sync-batch-start` / `sync-batch-end` markers from the Sync Engine
- Fixed: [visual-editor] Drop from Toolbox now actually inserts the control (replaced `mousemove`/`mouseup` with `dragover`/`drop`)
- Fixed: [visual-editor] Smart Device Panels (`*.Panel.gxSource`) now show their object properties in GX Properties (alias-aware: `Panel` ↔ `SDPanel`)
- Fixed: [visual-editor] Drops on top / bottom 25% of a UC region now succeed (always resolved to "inside")
- Fixed: [visual-editor] Generic containers expand to fill height with min-height 60px

#### v1.8.9 — 2026-04-28
- Added: [visual-editor] Object Picker modal for reference properties (Master Page, Web Component, Image, Theme, etc.) with Pattern and Type filters
- Added: [visual-editor] WYSIWYG render for User Controls with design XSL (e.g. `gxui.Layout` with North/West/Center/East/South)
- Added: [visual-editor] Hierarchical categories in GX Properties (`Regions/North/North Title Bar`, etc.)
- Added: [visual-editor] Arrow keys navigate between siblings in horizontal layouts
- Added: [visual-editor] UC properties declared with `<CustomType>WebComponentReference</CustomType>` etc. exposed as proper references with autocomplete
- Added: [visual-editor] Object Selector adds a Table filter when picking attributes
- Fixed: [visual-editor] Reference values that contain a leading GUID are displayed without it
- Fixed: [visual-editor] Master Page references resolve in the panel (alias `WebMasterPage` → `MasterPage`)
- Fixed: [visual-editor] Toolbox now uses the IDE's heuristic to filter User Controls (`<IncludeInControlInfo>` flag)
- Added: [visual-editor] Drag from Toolbox to canvas inserts the control (Button, Text Block, Image, Error Viewer, Web Component, Table, Grid, Group, Tab)

#### v1.8.8 — 2026-04-27
- Added: [visual-editor] Color picker for color properties (FontColor, BackColor, etc.)
- Added: [visual-editor] Attribute / Variable picker for User Control properties (ControlAttVarReference)
- Added: [visual-editor] User Control containers render their internal zones (`gxgxui.Layout` with North / Center / South / East / West)
- Added: [visual-editor] `Control Name` shown in the General category for User Control instances
- Fixed: [visual-editor] Responsive sizes preview works when JSON declares fewer cells than the row
- Fixed: [visual-editor] `<ucw>` controls in Abstract Layout are recognised by the registry (suffix-matching heuristic)
- Fixed: [visual-editor] `gxControlType` no longer appears as an editable property

#### v1.8.7 — 2026-04-26
- Added: [visual-editor] External User Controls discovered from the workspace (`<workspace>/User Controls/`)
- Added: [visual-editor] New command "KBBridge: Show User Controls" — QuickPick of every UC discovered
- Added: [visual-editor] User Controls show their real name in the form preview, with toolbox icon when declared
- Added: [visual-editor] `UC` letters as fallback icon
- Added: [visual-editor] Real properties of a User Control in GX Properties (Width, Height, Attribute, Data, FocusOnLoad, Font, etc.)
- Changed: [visual-editor] GX Properties header shows `<UC>: <Name>` for User Control instances

#### v1.8.6 — 2026-04-25
- Added: [visual-editor] Per-property "Restore to Default" in the Responsive Sizes editor (right-click any field)
- Added: [visual-editor] Auto-distribution of default cell widths
- Added: [visual-editor] Copy property value from the right-click menu
- Added: [visual-editor] Add ▸ in Abstract Layout creates a sibling cell automatically
- Fixed: [visual-editor] Cascade direction matches GeneXus IDE (XS/SM standalone, MD inherits SM, LG inherits MD)
- Fixed: [visual-editor] `visible:false` cells render alone in their logical row at 100% effective width
- Fixed: [visual-editor] Two 50% cells of a responsive row no longer wrap
- Fixed: [visual-editor] Move buttons skip invisible siblings
- Fixed: [visual-editor] Move persists to the JSON as `move:N` declaratively
- Fixed: [visual-editor] Reordering at one scale doesn't bleed into the others

#### v1.8.5 — 2026-04-24
- Added: [visual-editor] Visual Responsive Sizes editor — modal with scale picker (XS/SM/MD/LG), live cell preview and per-cell properties (Width %, Offset %, Move ← →, Visible, Default)
- Added: [visual-editor] Table Type picker — Tabular, Responsive, Absolute, Flex, SmartTable, Stencil
- Added: [visual-editor] Default checkbox per cell (resets every override / locks current values as explicit)
- Changed: [visual-editor] Custom editor inputs are read-only (use `...` button or right-click to copy)

#### v1.8.4 — 2026-04-23
- Added: [visual-editor] Add Tab Page (right-click → Add ▸ Tab Page)
- Added: [visual-editor] Drag & drop to rearrange controls (before / after / inside, with validation)
- Added: [visual-editor] Row drag handle (`≡`) for tables with two or more rows
- Added: [visual-editor] Sticky header (toolbar, form tabs, breadcrumb stay pinned while scrolling)
- Changed: [visual-editor] Small drag threshold (1–2px movements ignored)
- Fixed: [visual-editor] Responsive layout follows the row when reordered

#### v1.8.3 — 2026-04-22
- Added: [visual-editor] Delete controls from the form preview (right-click Delete or Delete key, with confirmation for non-empty containers)
- Added: [visual-editor] Cut, Copy, Paste for controls and entire subtrees (Ctrl/Cmd + X/C/V)
- Added: [visual-editor] Move controls up/down from the keyboard (Ctrl/Cmd + arrows; horizontal layouts use Left/Right)
- Added: [visual-editor] Insert Attribute / Variable picker (searchable popup with KB attributes and local variables)
- Added: [visual-editor] Add ▸ submenu on the context menu (Attribute, Button, TextBlock, Image, Error Viewer, Web Component, Table, Row, Cell, Section, Grid, Grid Column, Tab, etc.)
- Added: [visual-editor] Add Area in MultiForm (creates Area link + referenced Form with empty responsive table)
- Added: [visual-editor] Smart defaults for new controls (inherit majority-used sibling values)

#### v1.8.2 — 2026-04-21
- Added: [visual-editor] Many more editable properties in GX Properties (attributes, buttons, text blocks, grids, grid columns, images, tables, sections, fieldsets)
- Added: [visual-editor] Caption support for attribute controls (text, position, class)
- Added: [visual-editor] GX Properties now works on Tabs, Tab Pages, Action Groups, Action Group Items, Layouts, Forms, Form references
- Added: [visual-editor] Title Font as a single collapsible group in grid columns
- Changed: [visual-editor] Parent properties (Font, Title Font, etc.) start collapsed
- Fixed: [visual-editor] Form preview updates immediately when a property changes
- Fixed: [visual-editor] Editing in MultiForm files writes to the correct form

#### v1.8.1 — 2026-04-20
- Added: [visual-editor] GX Properties for form controls (attributes, text blocks, buttons, error viewers, images, grids, grid columns, web components, tables, rows, cells, fieldsets, divs, body)
- Added: [visual-editor] Sub-properties shown as collapsible groups
- Added: [visual-editor] Visibility rules per property (e.g., combo-only properties hidden when control is Edit)
- Added: [visual-editor] Combo values detected from code (literal values added in code events)
- Added: [visual-editor] Form references — controls that point to another form are recognised; double-click navigates

#### v1.8.0 — 2026-04-19
- Added: [visual-editor] **Visual Form Editor for `.gxForm` files** — new editor that renders the form structure visually, supporting all three GeneXus form formats (HTML, Abstract Layout, MultiForm)
- Added: [visual-editor] WYSIWYG control rendering (text field for Edit, dropdown for Combo, checkbox for true/false, radio buttons for radio domains, calendar icon for Date/DateTime)
- Added: [visual-editor] Sample data in grid columns
- Added: [visual-editor] Responsive layout preview (XS / SM / MD / LG)
- Added: [visual-editor] Tabs for MultiForm files
- Added: [visual-editor] Clickable breadcrumb
- Added: [core-lib] Faster initial load — KB indexing starts when a workspace with a Knowledge Base is opened
- Added: [core-lib] New command "KBBridge: Show Cache Status"
- Fixed: [core] "Select a child file to open" now opens `.gxForm` files in the Form Editor

### Platform
- Bundle rebuilt with new plugin code: 707 JS files (up from 640) across 7 extensions; bundle size 1.8 MB.
- Bundle must be uploaded to `keygenapi.kbbridge.com` after this release.

### VS Code 1.109.4
- No upstream changes

### MCP
- MCP server unchanged; verified Electron + `ELECTRON_RUN_AS_NODE=1` in place

---

## 1.6.12 — 2026-04-27

**Plugins v1.7.36** | **VS Code 1.109.4**

### Platform
- Fixed: System installer (admin) — non-admin users could not load extensions due to EPERM writing to `C:\Program Files\KBbridge\`. Bundle loading now gracefully handles read-only extension directories: JS code loads from in-memory sourceCache, webview/schema files use the VSIX skeleton from install time.
- Fixed: Bundle download on system installer — now saves to user-writable `%AppData%\kbbridge\` instead of the read-only install directory (from v1.6.10).

### VS Code 1.109.4
- No upstream changes

### MCP
- MCP server unchanged; verified Electron + `ELECTRON_RUN_AS_NODE=1` in place

---

## 1.6.11 — 2026-04-18

**Plugins v1.7.36** | **VS Code 1.109.4**

### Plugins

#### v1.7.36 — 2026-04-18
- Fixed: [analyzer, validator] `.gxPattern` and `.gxChilds` files are no longer parsed as GeneXus source code — these files share the `genexus` language ID but are not parseable code, causing false syntax errors
- Fixed: [visual-editor] "Save Object As..." now correctly updates the object name for `SDT` and `API` files
- Changed: [sync-status] `gxSync.trackExternalChanges` default changed from `false` to `true` — external change tracking enabled by default

### Platform
- Fixed: Bundle download failing on system installer (`C:\Program Files\KBbridge\`). The bundle was saved to `extensionPath` which is read-only for non-admin users. Now saved to `globalStoragePath` (`%AppData%\kbbridge\`) which is always writable.
- Updated: sync-status VSIX skeleton with `trackExternalChanges` default = `true`

### VS Code 1.109.4
- No upstream changes

### MCP
- MCP server unchanged; verified Electron + `ELECTRON_RUN_AS_NODE=1` in place

---

## 1.6.10 — 2026-04-17

**Plugins v1.7.35** | **VS Code 1.109.4**

### Plugins

#### v1.7.35 — 2026-04-17
- Fixed: [core-lib] CRITICAL — `loadKeywordsConfig` crash when keywords use versioned object format `{"name":"Panel","since":"15.0"}`. The loader called `.toLowerCase()` on the object instead of extracting the `name` field first, preventing activation of genexus-language-core and all dependent extensions (Analyzer, Validator, Controller, Visual Editor).

#### v1.7.34 addendum — 2026-04-17
- Added: [visual-editor] GX Properties panel now filters properties by GeneXus version — reads `kb-config.json` and hides properties with `since` greater than the KB's version

### Platform
- Added: Update notification — license-manager checks GitHub Releases once per day, shows notification when a newer version is available (Download → kbbridge.com/download, Release Notes → kbbridge.com/changelog, Skip This Version). Opt-out via `kbbridge.checkForUpdates` setting. Manual check via Command Palette: `KBEditor: Check for Updates`.
- Added: Two-step staged release flow — `kbbridge-release` builds all 3 OS in parallel (single click, draft); `kbbridge-publish-verify` + `kbbridge-publish` replace manual publishing (auto-detect version from `version.json`).
- Fixed: Buy-license URL now sends KBEditor version (e.g. `1.6.9`) instead of VS Code version (`1.109.4`).
- Fixed: Bundle cache invalidation — was keyed by VS Code version (constant across KBEditor releases, never invalidated). Now keyed by KBEditor version.
- Fixed: About dialog shows `KBEditor 1.6.10 (VS Code 1.109.4)` via product.json `kbEditorVersion` field + patch. `package.json` stays at VS Code version for extension compatibility.

### VS Code 1.109.4
- No upstream changes

### MCP
- MCP server unchanged; verified Electron + `ELECTRON_RUN_AS_NODE=1` in place

---

## 1.6.9 — 2026-04-16

**Plugins v1.7.34** | **VS Code 1.109.4**

### Plugins

#### v1.7.34 — 2026-04-16
- Added: [core-lib] GeneXus version support — `kb-config.json` file in workspace declares `genexusVersion`; methods/properties/keywords filtered by `since` field; absent file assumes latest version
- Added: [core-lib] New module `config/kbConfig.ts` with `loadKBConfig()`, `parseGXVersion()`, `isFeatureAvailable()`, `filterByVersion()`
- Added: [core-lib] `since` field added to 150+ entries across `gx-methods-config.json`, `gx-keywords-config.json`, `gx-functions-config.json`, `gx-properties-config.json`
- Fixed: [validator] Third chain validation now resolves initial type before chain resolution — `&Var.ToString().Trim()` no longer flags Trim as invalid
- Fixed: [core-lib] Several returnType corrections (MailMessage.ReplyTo, Boolean.ToString, Numeric.IsEmpty); typo ChartAt → CharAt
- Added: [core-lib] Many missing members: POP3Session, MailRecipientCollection, DateTime.AddDays, Character.Split/ToDate/ToDateTime, File.* (Open/Close/ReadLine/etc.), Directory.*, OutlookSession.*, DBConnection.*, sdt/collection ToJsonFile/ToXmlFile
- Fixed: [validator] `resolveDataType` now recursively resolves chained domains

#### v1.7.33 — 2026-04-15
- Fixed: [parser, analyzer] `For Each` with qualified transaction names (`Module.Transaction`) now correctly recognized — grammar `baseTrnClause` supports multi-level module paths
- Fixed: [core-lib] Chained property access on built-in types resolves return types — `&SMTPSession.Sender.Name` no longer flags Name as invalid
- Added: [core-lib] `MailRecipientCollection` missing members (New, Count, Item)

### Platform
- No platform changes (bundle rebuilt with v1.7.34)

### VS Code 1.109.4
- No upstream changes

### MCP
- MCP server unchanged; verified Electron + `ELECTRON_RUN_AS_NODE=1` in place

---

## 1.6.8 — 2026-04-14

**Plugins v1.7.32** | **VS Code 1.109.4**

### Plugins

#### v1.7.32 — 2026-04-14
- Fixed: [validator, server, core-lib] Nested SDT references with module qualifiers now fully supported — fields like `&Var.AccessTokenSDT.ExpiresIn` where the SDT has a module-qualified DataType (`AccessTokenSDT, PXTools.OAuthConsumer`) are no longer flagged as invalid
- Fixed: [server] Autocompletion for chained SDT field access no longer blocked by early GeneXus object detection — `&Var.SDTField.` now correctly shows sub-fields instead of returning empty results
- Fixed: [validator] Level name lookup in `getSDTFieldType` is now case-insensitive

### Platform
- No platform changes (bundle rebuilt with v1.7.32)

### VS Code 1.109.4
- No upstream changes

### MCP
- MCP server unchanged; verified Electron + `ELECTRON_RUN_AS_NODE=1` in place

---

## 1.6.7 — 2026-04-13

**Plugins v1.7.31** | **VS Code 1.109.4**

### Plugins

#### v1.7.31 — 2026-04-13
- Added: [sync-status] `gxSync.trackExternalChanges` setting (default: `false`) — enables/disables tracking of changes made to GeneXus source files outside of VSCode (e.g., by an external AI CLI). Changes from GeneXus IDE via KBBridge are always excluded.
- Added: [sync-status] Settings tab in Sync Panel — gear icon next to tabs opens a configuration panel with toggle switch for `trackExternalChanges`
- Added: [sync-status] Directory snapshot rebuild on enable — when `trackExternalChanges` is enabled, the file snapshot is rebuilt so only changes made after that moment are detected
- Fixed: [core-lib] `Count` in `collectionMethods` changed from Method to Property — `&Col.Count` no longer flagged as requiring parentheses
- Fixed: [validator] Variables with `Collection = 'True'` now correctly support collection methods (`.Count`, `.Item()`, `.Add()` etc.)
- Refactored: [validator] `buildVariablesMap()` returns `Map<string, VarInfo>` with `{ name, dataType, isCollection }` — eliminates fragile `VARCOLLECTION:` prefix hack
- Refactored: [core-lib, server, validator, core] Renamed `formControlProperties` to `formControls` and `autonomousControlProperties` to `autonomousControls`
- Fixed: [sync-status] `isKBBridgeExport()` path comparison normalizes double slashes and backslashes — exported objects no longer incorrectly registered as external changes

#### v1.7.30 — 2026-04-09
- Added: [core-lib, server, validator] Complete PEM for form controls (variables in Form) — common members for all controls plus ControlType-specific members (Edit, Combo Box, Check Box, List Box, Image, etc.)
- Added: [core-lib, core, server] Complete PEM for autonomous controls — TextBlock, Button, Grid, Image, Table, WebComponent, UserControl with methods and properties parsed from `.gxForm`
- Added: [core-lib] `getFormControls()` and `getFormControlInfo()` — parse `.gxForm` to extract ControlNames and ControlTypes
- Added: [core-lib] HttpRequest missing properties (ScriptName, QueryString, ServerHost, etc.) and Cookie property corrections
- Fixed: [server] `loadGxMethodsConfig()` now includes `formControls`, `businessComponentMethods`, and `autonomousControls`

### Platform
- Fixed: `build-bundle.py` now refreshes compiled `genexus-language-core-lib` into all dependents' `node_modules` before compilation — prevents stale `file:` dependency links from causing tsc failures when core-lib gains new exports
- Updated: sync-status VSIX skeleton with new `gxSync.trackExternalChanges` setting

### VS Code 1.109.4
- No upstream changes

### MCP
- MCP server unchanged; verified in-process registration uses Electron + `ELECTRON_RUN_AS_NODE=1`

---

## 1.6.6 — 2026-04-08

**Plugins v1.7.29** | **VS Code 1.109.4**

### Plugins

#### v1.7.29 — 2026-04-08
- Added: [core] File badge "AP" for `.API.gxSource` files in Explorer (was previously showing generic "GX")

#### v1.7.28 addendum — 2026-04-08
- Fixed: [validator, core-lib] Variable lookups in `methodValidator`, `parameterValidator` and `parameterParser` are now consistently case-insensitive — variables declared as `Cookie` are correctly resolved when referenced as `&cookie` (and vice versa)

### Platform
- No platform changes (bundle rebuilt with v1.7.29)

### VS Code 1.109.4
- No upstream changes

### MCP
- MCP server unchanged; verified in-process registration uses Electron + `ELECTRON_RUN_AS_NODE=1` (fix from 1.6.3 still in place)

---

## 1.6.5 — 2026-04-08

**Plugins v1.7.28** | **VS Code 1.109.4**

### Platform
- Removed: Windows portable ZIP artifact (`KBEditor-win32-x64-*.zip`). Users who extracted the ZIP to custom folders (e.g. paths containing spaces) hit install-path edge cases that broke MCP server configuration and `.mcp.json` path resolution. Only the User Setup and System Setup installers are now published for Windows — both install to a predictable, space-free path under `Programs\KBbridge\`.

### VS Code 1.109.4
- No upstream changes

---

## 1.6.4 — 2026-04-07

**Plugins v1.7.28** | **VS Code 1.109.4**

### Plugins

#### v1.7.28 — 2026-04-07
- Fixed: [core] "Go to Definition" on `BusinessComponent:`, `Attribute:` or `Domain:` reference prefixes — now navigates to the referenced Transaction/Attribute/Domain file (also supports module path: `BusinessComponent:Name, Module.Path`)
- Added: [core-lib, visual-editor, validator] BusinessComponent variables support — variables can now be based on `BusinessComponent:TransactionName` (also with sub-structures and module path: `BusinessComponent:Name.SubLevel, Module.Path`)
- Added: [visual-editor] "Business Component" property in GX Properties for variables — autocompletes only Transactions with `BusinessComponent = 'True'` enabled
- Added: [core-lib] Business Component methods documented in `gx-methods-config.json` — `Load`, `Save`, `Insert`, `Update`, `InsertOrUpdate`, `Delete`, `Check`, `Success`, `Fail`, `GetMessages`, `ToXml`, `FromXml`, `ToJson`, `FromJson`, `GetByKey`, `RemoveByKey`, `Mode`
- Added: [visual-editor] `ElementType` calculated property (`variable` | `attribute` | `domain`) — restricts reference properties (Attribute, SDT, ExternalObject, BusinessComponent) to variables only
- Fixed: [visual-editor] Type Definition fields hidden when variable is based on a BusinessComponent
- Fixed: [visual-editor] GX Properties detects `#Variables [dynamic]` and `#Variables [readonly]` sections — variables shown in readonly mode
- Fixed: [visual-editor] Reference property placeholders hidden when property is in readonly mode

### Platform
- No platform changes (bundle rebuilt with v1.7.28)

### VS Code 1.109.4
- No upstream changes

---

## 1.6.3 — 2026-04-07

**Plugins v1.7.27** | **VS Code 1.109.4**

### Platform
- Fixed: GeneXus Knowledge MCP server failed with "MCP error -32000: Connection closed" on machines without a system Node.js in PATH. The in-process `registerMcpServerDefinitionProvider` now uses KBEditor's bundled Electron binary with `ELECTRON_RUN_AS_NODE=1`, matching the auto-generated `.vscode/mcp.json` / `.mcp.json` configs.

### VS Code 1.109.4
- No upstream changes

---

## 1.6.2 — 2026-04-06

**Plugins v1.7.27** | **VS Code 1.109.4**

### Plugins

#### v1.7.27 — 2026-04-06
- Fixed: [server] Method autocompletion no longer adds parentheses to SDT fields and properties
- Fixed: [validator] Undeclared variable check now case-insensitive
- Fixed: [visual-editor] Duplicate property block creation prevented for reference properties
- Added: [core-lib] "Case" keyword added to autocompletion suggestions

### Platform
- Changed: Updated shared config gx-keywords-config.json

### VS Code 1.109.4
- No upstream changes

---

## 1.6.1 — 2026-04-04

**Plugins v1.7.26** | **VS Code 1.109.4**

### Platform
- Added: "Buy a License" link on registration page — opens purchase page with user data (name, company, email, language, machine fingerprint, version, platform, arch) as URL parameters for payment processor integration
- Fixed: Start menu folder now correctly shows "KBbridge" (shared with KBSync) instead of "KBEditor"

---

## 1.6.0 — 2026-04-04

**Plugins v1.7.26** | **VS Code 1.109.4**

### Plugins

#### v1.7.26 — 2026-04-04
- Added: [core] GeneXus keywords in autocompletion (EndNew, EndIf, For Each, Do While, etc.)
- Added: [core-lib] Compound keywords and completion exclusions in gx-keywords-config.json
- Fixed: [core-lib] Cleaned up keywords config — removed non-existent keywords, added missing ones

#### v1.7.25 — 2026-04-03
- Added: [visual-editor] Extended properties for Variables (~50 extra from config), Attributes, and Domains
- Added: [visual-editor] `mergePropertySchemas()` for combining schema + config properties
- Added: [visual-editor] `IsInForm` calculated property, `ObjectType` from filename, `ExternalName` for APIs
- Added: [visual-editor] DataType-based and ControlType-based conditional property visibility
- Added: [visual-editor] Readonly conditions for Domain/Supertype-based properties
- Added: [config] `variableProperties` section with 9 shared property groups
- Added: [config] `Blob`, `GeoLine`, `GeoPolygon` types in gx-methods-config.json
- Fixed: [visual-editor] readonlyIf evaluates against defaults, Custom type as readonly, Color as string
- Fixed: [config] DKL_ internal properties hidden, category/default corrections

### Platform
- Added: Error Lens (usernamehw.errorlens) pre-installed — displays inline error/warning messages next to code (open-source, MIT license)
- Changed: Updated shared configs (gx-keywords, gx-methods, gx-properties)
- Changed: Updated visual-editor VSIX schema (variable.json)

### VS Code 1.109.4
- No upstream changes

---

## 1.5.3 — 2026-04-03

**Plugins v1.7.25** | **VS Code 1.109.4**

### Plugins

#### v1.7.25 — 2026-04-03
- Fixed: [validator] Parameter mode prefixes (`In:`, `Out:`, `InOut:`) now case-insensitive in Parm() declarations
- Fixed: [validator] "requires a variable (mode: out/inout)" warning now only triggers for non-assignable values
- Fixed: [visual-editor] GX Properties shows variable properties when cursor is on `&Variable` in any Transaction section
- Added: [server] Method autocompletion auto-inserts parentheses and triggers Signature Help

### Platform
- Removed: All debug traces and diagnostic code from license manager
- Fixed: Updated ALL VSIX skeletons with current plugin defaults (validator rules enabled)
- Fixed: Added require.resolve to bundle module hook
- Added: Bundle loading lessons documented in .ai-context/

### VS Code 1.109.4
- No upstream changes

---

## 1.5.2 — 2026-04-02

**Plugins v1.7.24** | **VS Code 1.109.4**

### Platform
- Fixed: sync-status extension now loaded from encrypted bundle (was running from old VSIX stub — all new features were invisible)
- Fixed: Webview TypeScript compiled separately for visual-editor and sync-status (both have their own tsconfig.json)
- Fixed: CSS files now included in encrypted bundle and written to disk (4 files for properties-panel, sync-panel, new-object-panel, tree-editor)
- Fixed: JSON schema files now included in bundle and written to disk (7 property schemas for visual-editor)
- Fixed: VSIX skeleton updated with current property schemas (attribute.json, transaction-attribute.json changed)
- Fixed: GxPropertiesConfigManager config path — added root-level `node_modules/` alongside `out/node_modules/`
- Fixed: Bundle now collects .js + .css + .json files (was .js only)
- Fixed: License manager writes all webview, CSS, JSON, and schema files to disk at bundle load time

---

## 1.5.1 — 2026-04-02

**Plugins v1.7.24** | **VS Code 1.109.4**

### Platform
- Fixed: Webview TypeScript now compiled separately (visual-editor webview has own tsconfig)
- Fixed: CSS files now included in encrypted bundle (4 files for properties-panel, sync-panel, new-object-panel, tree-editor)
- Fixed: CSS and webview source files written to disk by license manager (webviews load via URI)
- Fixed: GX Properties panel now updates correctly — object properties on line 1, updated styling
- Fixed: GxPropertiesConfigManager config path — added root-level `node_modules/` alongside `out/node_modules/`

---

## 1.5.0 — 2026-04-01

**Plugins v1.7.24** | **VS Code 1.109.4**

### Plugins

#### v1.7.24 — 2026-03-31
- Fixed: [sync-status] User saves in VSCode no longer blocked by KBEditor export check — `onDidSaveTextDocument` bypasses export filter (only external changes filtered)
- Fixed: [sync-status] `.gxForm` files correctly extracted by ObjectInfoExtractor — form saves generate proper change entries with object name and type

### Platform
- Changed: Start menu folder set to "KBbridge" (shared with KBSync)

### VS Code 1.109.4
- No upstream changes

---

## 1.4.1 — 2026-03-30

**Plugins v1.7.23** | **VS Code 1.109.4**

### Platform
- Fixed: Cached encrypted bundle now invalidated on app update — stores app version alongside bundle and forces re-download when version changes

---

## 1.4.0 — 2026-03-30

**Plugins v1.7.23** | **VS Code 1.109.4**

### Plugins

#### v1.7.23 — 2026-03-30
- Added: [core] Signature Help for variable methods — `&Variable.Method(` shows parameter hints from methods config for all data types including External Objects
- Added: [core] Signature Help for `.Submit()` calls with first parameter documentation
- Added: [core] Active parameter highlighting — advances highlighted parameter as you type commas
- Improved: [core] Signature Help regex patterns now case-insensitive
- Improved: [core] Signature Help persists while typing arguments
- Added: [config] `Sleep`, `RGB`, `Space`, `StrSearch`, `StrSearchRev`, `StrReplace` added to built-in functions
- Improved: [core-lib] `getMethodsForType()` searches both builtInTypes and externalObjectTypes
- Improved: [core-lib] SDT fields without explicit DataType default to `Numeric(4.0)`

#### v1.7.22 — 2026-03-29
- Added: [visual-editor] Visual distinction between editable, readonly, and default properties
- Added: [visual-editor] Editable input fields show blue border on focus
- Added: [visual-editor] Dropdown options display with bright text even in dimmed state
- Fixed: [visual-editor] Domain chain resolution — multi-level domain references fully resolved to base type
- Fixed: [visual-editor] Domain file lookup uses CacheManager as fallback
- Fixed: [visual-editor] RestoreToDefault for reference properties correctly removes DataType
- Fixed: [visual-editor] RestoreToDefault on pattern nodes preserves CustomType dropdown options
- Fixed: [visual-editor] Direct text input in CodeConditions/CodeExpressions no longer wraps in brackets
- Fixed: [visual-editor] Reference fields no longer show double border
- Fixed: [config] Web Service Protocol properties correctly hidden when "Expose as Web Service" is false
- Added: [core] File badges for Table (TB), Attribute (AT), and Domain (DM) in Explorer

### Platform
- Changed: Updated shared config `gx-functions-config.json` (6 new built-in functions)
- Changed: Product renamed from KBbridge to KBEditor (display name only — internal IDs preserved)

### VS Code 1.109.4
- No upstream changes

---

## 1.3.0 — 2026-03-29

**Plugins v1.7.21** | **VS Code 1.109.4**

### Plugins

#### v1.7.21 — 2026-03-29
- Added: [visual-editor] Object-level properties in GX Properties panel — clicking on line 1 of any `.gxSource` shows editable properties (Description, Call Protocol, Object Visibility, etc.) driven by `gx-properties-config.json`
- Added: [visual-editor] Table attribute properties — clicking attribute in `.Table.gxSource` shows properties in readonly mode
- Added: [visual-editor] ExternalObject member properties — Properties, Methods, Parameters, Events shown with member-specific properties
- Added: [visual-editor] New reference property types: `SDTReference`, `MasterPageReference`, `ThemeReference`
- Added: [visual-editor] Property value remapping — internal GeneXus names mapped to display names (prettyName)
- Added: [visual-editor] Multi-line Formula display with auto-sizing textarea
- Added: [visual-editor] Subcategory visual nesting with `/` separator
- Added: [visual-editor] Enum display names in dropdown properties
- Added: [visual-editor] Conditional property visibility and readonly rules from config
- Added: [visual-editor] Fallback properties for undeclared object types (BASE + BASE_VISIBILITY)
- Changed: [visual-editor] JSON/Traditional toggle moved inside element header
- Fixed: [visual-editor] DataType shown as readonly (not hidden) when based on Domain
- Fixed: [visual-editor] Multi-line property parsing and write-back with correct indentation
- Fixed: [visual-editor] Comma formatting in property blocks handles multi-line values
- Added: [analyzer] Full syntax parsing for `.Group.gxSource` files
- Fixed: [parser] Multi-line string values supported in all property blocks
- Fixed: [validator] "requires a variable (mode: out)" downgraded from Error to Warning
- Fixed: [sync-status] Bulk export end detection uses file modification time as reference
- Added: [config] `PrivateObject` moved to ignored properties
- Changed: [config] BASE split into BASE (Description) and BASE_VISIBILITY (ObjectVisibility)

#### v1.7.20 — 2026-03-28
- Added: [sync-status] `.gxForm` files now tracked as user changes

#### v1.7.19 — 2026-03-26
- Added: [analyzer] Full syntax parsing for `.Table.gxSource` files (PhysicalAttributes, LogicalAttributes, RedundantAttributes, Indexes, ForeignTables, etc.)
- Added: [analyzer] Full syntax parsing for `.ExternalObject.gxSource` files (Properties, Methods, Events)
- Added: [analyzer] `[readonly]` section protection with distinct grey border decoration
- Added: [analyzer] Multi-line string support in LogicalAttributes Formula
- Added: [icons] Table icon with grid/column design
- Added: [core] `.Table.gxSource` registered as GeneXus language file
- Added: [sync-status] "Delete All Pending" button in deletion confirmation
- Fixed: [sync-status] Files modified during bulk export excluded from change tracking
- Fixed: [parser] `Dom:DomainName.Value` syntax recognized

#### v1.7.18 — 2026-03-25
- Added: [sync-status] Bulk export detection — mass externalization pauses file change tracking
- Added: [sync-status] Status bar spinner during bulk export: "KBEditor: Exporting KB..."
- Added: [sync-status] Banner in Sync panel during bulk export
- Added: [sync-status] Configurable bulk export timeout (default 30 min) with notification
- Added: [sync-status] Startup detection of in-progress bulk export

### Platform
- Added: MCP server embedded in installer for all platforms (esbuild bundle + platform-specific native addons)
- Added: Auto-generate AI config files for GeneXus workspaces (`.vscode/mcp.json`, `.mcp.json`, `CLAUDE.md`, `.cursor/mcp.json`)
- Added: MCP server uses KBEditor's own Electron as Node runtime (`ELECTRON_RUN_AS_NODE=1`) — no external Node.js required
- Added: MCP config merge logic — adds server to existing config files without overwriting
- Added: Versioning system — `version.json` as single source of truth, CI reads it for artifact names
- Added: Unified CHANGELOG across Plugins/Platform/VS Code streams
- Added: CI publishes installers to public `kbbridge-releases` repo with structured release notes
- Added: CI auto-updates public repo README with direct per-platform download links
- Added: CI publishes CHANGELOG.md to public repo automatically
- Added: Installer files renamed from VS Code version to KBEditor version (e.g., `KBEditorSetup-x64-1.3.0.exe`)
- Added: AI context files committed to git (`kbbridge/.ai-context/`) for project continuity
- Added: `CLAUDE.md` at repo root for Claude Code auto-loading
- Added: New shared config `gx-properties-config.json` for object property definitions
- Fixed: Extension loading isolation — one extension failure no longer blocks the remaining 5
- Fixed: MCP server dependency resolution — auto-walk full transitive dep tree (55 packages) instead of manual cherry-picking
- Fixed: Stubs for `sharp` and `onnxruntime-web` (imported by transformers but unused on Node.js)
- Fixed: core-lib `package.json` created inline (not copied from plugins repo — broke on CI)
- Fixed: macOS embed script compatibility (`mktemp` and `sed` BSD syntax)
- Fixed: GitHub Actions `secrets` not allowed in step-level `if` expressions
- Changed: VSIX skeletons updated (core: Table file type, sync-status: bulk export timeout, icons: Table icon)
- Changed: Bundle rebuilt — 663 JS files (was 662)

### VS Code 1.109.4
- No upstream changes

---

## 1.2.0 — 2026-03-26

**Plugins v1.7.17** | **VS Code 1.109.4**

### Plugins

#### v1.7.17 — 2026-03-24
- Fixed: [validator] Parameter type resolution fully resolves variables via Attributes to base data type
- Fixed: [validator] Domains/attributes without explicit DataType default to `Numeric(4.0)` (GeneXus standard)
- Fixed: [validator] Qualified names in `.Call()` / `.Udp()` resolved correctly (e.g., `V138.FirmaReporte.Udp()`)
- Fixed: [validator] Block comments (`/* ... */`) now skipped during validation
- Fixed: [validator] `Numeric(9.0-)` signed format accepted — `-` suffix no longer flagged
- Fixed: [validator] Property access on built-in types (e.g., `&Directory.Source.Trim()`) resolves correctly
- Fixed: [validator] Case-insensitive lookup for built-in type methods
- Fixed: [parser] All GeneXus keywords fully case-insensitive (`EndIF`, `endif`, `ENDIF` all valid)
- Fixed: [parser] `For Each Count &Variable` now valid without `Skip`
- Fixed: [visual-editor] SDT type classification for DataTypes with `.` in name
- Added: [visual-editor] SDT autocomplete shows nested collection items (on-demand top-10 expansion)
- Fixed: [visual-editor] Autocomplete no longer disappears when typing `,` for module filtering
- Added: [sync-status] KBEditor export exclusion — files exported by KBEditor not tracked as user changes
- Changed: [icons] Knowledge Base icon redesigned — 3D cube with semi-transparent faces

#### v1.7.16 — 2026-03-23
- Fixed: [validator] SDT field path navigation descends correctly into sub-levels
- Fixed: [validator] Enum domain vs compatible base type mismatch downgraded to Warning
- Fixed: [validator] Parameter error messages show caller's variable name (not callee's parameter name)
- Fixed: [validator] `.Udp()` return errors highlight assignment variable (before `=`)
- Fixed: [validator] Parameter position highlighting works inside nested function calls
- Added: [validator] Built-in GeneXus function validation (Msg, iif, Format, etc.) with parameter count/types
- Added: [validator] `any` type support for parameters
- Added: [validator] Skip re-validation on editor tab switch if document unchanged
- Added: [validator] Cached parsed parameters for called procedures
- Fixed: [parser] `Count(attribute, condition)` recognized as valid function call
- Fixed: [parser] `&Commit` and keyword-named variables parsed correctly in all contexts
- Fixed: [analyzer] `[dynamic]` sections now offer dialog instead of silently blocking edits
- Added: [core] "KBEditor: Reindex Cache" command with progress bar
- Fixed: [core] Go to Definition adds found objects to cache for instant subsequent lookups
- Fixed: [core] Hover and Code Action use cached KB objects instead of full workspace scan
- Fixed: [core] Output panel no longer opens automatically on reload
- Added: [sync-status] Delete button on Pending entries with confirmation dialog
- Changed: [sync-status] Panel title "GX Sync" → "KBEditor Sync"

#### v1.7.15 — 2026-03-21
- Added: [validator] Built-in function validation for `Msg` (1-2 params) and `iif` (3 params)
- Changed: [icons] Knowledge Base icon updated

#### v1.7.14 — 2026-03-19
- Added: [validator] "Add Variable" Quick Fix infers data type from variable name
- Added: [validator] Predefined variable `&Mode` with type `Character(3)` for Transactions

### Platform
- Added: MCP server embedded in installer for all platforms (Windows, Linux, macOS)
- Added: esbuild bundling — MCP server JS bundled to single 2.2 MB file, only native addons shipped separately
- Added: Auto-generate AI config files when GeneXus workspace detected (`.vscode/mcp.json`, `.mcp.json`, `CLAUDE.md`, `.cursor/mcp.json`)
- Added: MCP configs use KBEditor's own Electron as Node runtime (`ELECTRON_RUN_AS_NODE=1`) — no external Node.js needed
- Added: MCP config merge logic — adds `genexus-knowledge` to existing config files without overwriting
- Added: Per-module feature documentation in PRODUCT_OVERVIEW.md
- Added: Date + commit hash in CI artifact names for build identification
- Added: GitHub Release `mcp-data-v1` for MCP server data (vector DB + embedding model)
- Fixed: Extension loading isolation — one extension failure no longer blocks the remaining 5
- Fixed: core-lib `package.json` created inline (not copied from plugins repo — broke on CI)
- Fixed: MCP server dependency resolution — auto-walk full transitive dep tree (55 packages) instead of manual cherry-picking
- Fixed: Added stubs for `sharp` and `onnxruntime-web` (imported by transformers but unused)
- Changed: VSIX skeletons updated — KBEditor command categories, reindexCache command, new KB icon
- Changed: Embed script creates core-lib directory structure with `package.json` + 3 shared JSON configs
- Changed: Bundle source hotfixes (non-blocking indexer, icon mappings) now inactive — upstream has them natively

---

## 1.1.0 — 2026-03-21

**Plugins v1.7.13** | **VS Code 1.109.0**

### Plugins

#### v1.7.13 — 2026-03-17
- Added: [analyzer] Full syntax parsing for `.API.gxSource` files (API objects with groups, methods, annotations)
- Added: [icons] New SVG icons: API, ExternalObject, Childs, test types (outline style, rounded borders)
- Added: [icons] Form-specific icons: WebPanel, Transaction, WebComponent, MasterPage, Panel for SD
- Added: [visual-editor] New object types in "New Object" panel: Attribute, ExternalObject, API, WebMasterPage, test types
- Changed: [icons] All object icons redesigned to consistent outline style

#### v1.7.12 — 2026-03-16
- Fixed: [core] Incorrect scope references in publisher configuration

#### v1.7.11 — 2026-03-15
- Added: [core] `.gxForm` language — XML highlighting and auto pretty-printing, separated from GeneXus language
- Added: [core] "Save Object As..." available from editor tabs

#### v1.7.9 — 2026-03-12
- Fixed: [core] Re-enabled hover tooltips disabled during performance optimization

#### v1.7.7 — 2026-03-10
- Added: [analyzer] Dynamic section protection (`#[dynamic]` / `[dynamic]` tags) with visual decorations
- Added: [analyzer] DataProvider group properties: `NoOutput`, `OutputIfDetail`, `One`, `Count`, `Skip`

#### v1.7.6 — 2026-03-09
- Added: [core] Group file (`.gxGroup`) support
- Added: [core] Non-blocking reference indexer — no more editor freezes on large KBs
- Added: [controller] Sort Variables — auto-alphabetize `#Variables` section on save
- Fixed: [visual-editor] Properties panel variable editing fixes

#### v1.7.5 — 2026-03-04
- Fixed: [sync-status] "Skipped" toggle button now works correctly

#### v1.7.4 — 2026-03-03
- Improved: [sync-status] Panel UI — better entry layout, status badges, usability

### Platform
- Added: GeneXus documentation MCP server — local vector DB with 46,750 doc chunks, 3 AI tools (`search_docs`, `lookup_method`, `list_topics`), fully offline
- Added: MCP server registered via VS Code `mcpServerDefinitions` proposed API
- Added: Product overview and marketing documentation (PRODUCT_OVERVIEW.md)
- Added: Automated bundle build script (`build-bundle.py`) — compiles all plugins and creates ZIP
- Added: Hotfix system — non-blocking reference indexer and test/API icon mappings injected at bundle load time
- Fixed: Bundle require hook resolves package imports from encrypted bundle
- Fixed: Language server skeleton VSIX includes all third-party deps
- Fixed: Bundle cache invalidation and language server crash
- Fixed: Webview scripts written to disk (sandboxed iframes can't load from memory)
- Fixed: Bundle path normalization for Windows backslashes
- Fixed: Properties panel race condition on startup
- Fixed: Extension host blocking during reference index build
- Fixed: Bundle disk-write policy — empty placeholders for extension host files, real source for webview/LSP
- Added: License recovery via `/getlicensekey` endpoint
- Added: Machine File Certificate verification for startup fingerprint check
- Fixed: Trial recovery and expiry handling from updated API
- Changed: GitHub Actions updated to Node 24 compatible versions

---

## 1.0.0 — 2026-02-26

**Plugins v1.7.1** | **VS Code 1.109.0**

### Plugins

#### v1.7.0 — 2026-02-14
- Added: [sync-status] Synchronization status extension — change tracking panel with three tabs
- Added: [sync-status] Automatic change detection on file save and filesystem monitoring
- Added: [sync-status] External change reconciliation and configurable retention

#### v1.6.0 — 2026-01-25
- Added: [visual-editor] Visual editing extension — GX Properties panel, pattern tree viewer
- Added: [visual-editor] DataType compound parsing (`Numeric(12.2-)` → Length=12, Decimals=2, Signed=true)
- Added: [visual-editor] Variable renaming with automatic source file update
- Added: [visual-editor] Reference autocomplete for Domain, Attribute, SDT, ExternalObject

#### v1.5.0 — 2026-01-10
- Added: [controller] Object editing operations extension
- Added: [controller] Transaction ↔ Attribute synchronization

#### v1.4.0 — 2026-01-02
- Added: [core] Instant KB object lookup with background indexing and persistent cache

#### v1.3.0 — 2025-12-15
- Added: [all] Shared library — parsing, search, type resolution across all extensions
- Added: [all] Method/property definitions for all GeneXus built-in types

#### v1.2.0 — 2025-12-10
- Added: [validator] Semantic validation extension — DataType, undeclared variables, parameter checking, method chain validation

#### v1.1.0 — 2025-11-25
- Added: [analyzer] Syntax analysis extension with ANTLR4 grammar
- Added: [parser] GeneXus lexer and parser for `.gxSource` files

#### v1.0.0 — 2025-11-12
- Added: [icons] File icon theme for GeneXus object types
- Added: [core] GeneXus language definition — syntax highlighting, snippets
- Added: [server] Language Server with Go to Definition, Hover, Code completion

### Platform
- Added: KBEditor branded VS Code fork (based on VSCodium)
- Added: Custom branding — name, icons, data folder (`.kbbridge`)
- Added: Phase 0 — embed proprietary extensions as built-in (non-uninstallable)
- Added: Phase 1 — Rust native loader with AES-256-GCM decryption
- Added: Phase 1.5 — License Manager (registration, 15-day trial, paid license activation)
- Added: Phase 2 — Extension API server at `keygenapi.kbbridge.com`
- Added: Phase 3 — Runtime bundle decryption and loading via `Module._compile()`
- Added: Offline license verification with Ed25519 signatures
- Added: Machine fingerprinting (7-point hardware identification)
- Added: GitHub Actions CI workflows for Windows, macOS (x64 + arm64), Linux
- Added: VSIX skeleton system — stubs in repo, real code in encrypted bundle
- Added: Update checks disabled in builds
- Added: Publisher rename pipeline (`puntoexe` → `kbbridge`)

### VS Code 1.109.0
- Initial base version from upstream VSCodium
