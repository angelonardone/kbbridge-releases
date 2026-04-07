# KBEditor Changelog

All notable changes to the KBEditor product. Each release documents changes across
three independent streams:

- **Plugins** — GeneXus language extensions (IntelliSense, parser, validator, visual editor, etc.)
- **Platform** — KBEditor-specific infrastructure (license manager, MCP server, embed pipeline, CI/CD)
- **VS Code** — Upstream VS Code base (via VSCodium)

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
