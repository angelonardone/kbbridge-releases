# KBEditor Changelog

All notable changes to the KBEditor product. Each release documents changes across
three independent streams:

- **Plugins** — GeneXus language extensions (IntelliSense, parser, validator, visual editor, etc.)
- **Platform** — KBEditor-specific infrastructure (license manager, MCP server, embed pipeline, CI/CD)
- **VS Code** — Upstream VS Code base (via VSCodium)

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
