# KBSync Changelog

All notable changes to KBSync — the GeneXus Knowledge Base synchronization tool.

---

## 1.2.51 — 2026-04-04

### Sync Engine v1.4.3
- Added: Variable, Attribute and Domain properties beyond DataType are now serialized and deserialized — validation, picture, control info, appearance, behavior, virtual keyboard, and offline properties are preserved in the roundtrip
- Fixed: ExternalName, ExternalNamespace and AllowNull properties excluded from Variable/Attribute/Domain serialization — these properties are not applicable in that context and share names with other object types

### Sync Manager v1.2.51
- No additional changes (Engine binaries update only)

---

## 1.2.50 — 2026-04-03

### Sync Engine v1.4.2
- Fixed: Pattern instance XML export now removes GUID prefixes from attribute references, producing human-readable names in `.gxPattern` files

### Sync Manager v1.2.50
- No additional changes (Engine binaries update only)

---

## 1.2.49 — 2026-04-02

### Sync Engine

#### v1.4.1 — 2026-04-02
- Fixed: Geography types (GeoPoint, GeoLine, GeoPolygon, Geography) now supported in GeneXus 15 and 16 (previously restricted to GX 17+)
- Fixed: BlobFile data type now supported in GeneXus 16 (previously restricted to GX 17+)
- Fixed: AverageLength property serialization/deserialization now works in GeneXus 15 and 16 (previously restricted to GX 17+)

#### v1.4.0 — 2026-04-01
- Added: Multi-version GeneXus support — now compatible with GeneXus X, XEv1, XEv2, XEv3, 15, 16, 17 and 18
- Added: Simplified PuntoExe.PXTools.dll (one per GeneXus version) abstracts all SDK differences — KBbridge.exe code is shared across versions

### Sync Manager

#### v1.2.49 — 2026-04-02
- No additional changes (Engine binaries update only)

#### v1.2.48 — 2026-04-01
- Added: Support for all GeneXus versions (X, X Evo1/2/3, 15, 16, 17, 18) with dedicated binaries per version
- Fixed: "Use same registration data" checkbox now disabled when no previous registration data exists

---

## 1.2.47 — 2026-03-30

### Sync Engine v1.3.5
- Fixed: Import of WebPanel/Transaction with form — source and form are now loaded together before validation, preventing false errors when source references form elements (e.g., grid events like `GridName.Load`)

### Sync Manager

#### v1.2.47 — 2026-03-30
- Changed: Application renamed from "KBbridge Manager" to "KBbridge Sync Manager" across all UI, installer, and splash screen
- Changed: About tab now uses a ComboBox to switch between Sync Manager and Sync Engine changelogs instead of showing both stacked
- Fixed: "Buy License" and "Enter License Key" buttons now visible on fresh installations without a license
- Changed: Uninstaller now asks whether to keep or delete configuration files (license, KB settings) instead of deleting them automatically

#### v1.2.46 — 2026-03-29
- No additional changes (Engine binaries update only)

---

## 1.2.45 — 2026-03-27

### Sync Engine

#### v1.3.4 — 2026-03-29
- Fixed: ExternalObject methods without return value (DataType `None`) no longer generate `DataType = 'None'` in `.gxSource` files — void methods now omit the DataType property entirely

#### v1.3.3 — 2026-03-28
- Improved: Combo property values now use pretty names in `.gxSource` files instead of GeneXus internal codes
- Improved: Reference properties now use typed references in config instead of hardcoded property names
- Added: `IsMain` property support for DataProvider objects

#### v1.3.2 — 2026-03-27
- Added: Embedded assembly resolver via Startup class — preloads embedded DLLs before any code runs
- Added: PXTools library compiled with unique assembly name to avoid conflicts
- Fixed: ExternalObject item DataType was not appearing in `.gxSource` files

### Sync Manager v1.2.45
- Fixed: Clear button in log viewer now also truncates the log file on disk

---

## 1.2.44 — 2026-03-26

### Sync Engine v1.3.1
- Added: Embedded DLLs (Newtonsoft.Json, PXTools, BouncyCastle) — deployment reduced to KBBridge.exe + .config only
- Added: Application icon for KBBridge.exe
- Improved: Properties classified as known, ignored, or unknown with warning in log
- Improved: Warning and info messages deferred when a progress line is in-progress
- Added: Properties `ObjectVisibility`, `PrivateObject`, `IsMain`, `OnSessionTimeout`, `URLAccess`, `AUTO_REFRESH`, `OutputCollectionName` to properties config
- Added: ExternalObject platform and member properties with visibility rules
- Added: Import support for `.gxForm` entries with cross-deduplication of `.gxSource`/`.gxForm`

### Sync Manager v1.2.44
- Fixed: License validation now requires machine certificate — a license key without certificate (e.g. copied from another machine) is rejected
- Fixed: License type (trial/annual/monthly) now saved from all activation flows (Trial, Enter License Key, refresh)
- Added: License refresh errors are now logged to `license-refresh.log` for diagnostics
- Fixed: Installer messages now display in the selected language (Spanish, Portuguese)

---

## 1.2.43 — 2026-03-25

### Sync Engine v1.3.0
- Added: Table externalization support — tables exported to `.Table.gxSource` files in `#Tables` directory
- Tables include: `#PhysicalAttributes`, `#LogicalAttributes` (formulas), `#RedundantAttributes`, `#Indexes`, `#ForeignTables`, `#SuperordinatedTables`, `#SubordinatedTables`
- Sections use `[readonly]` tag to indicate non-editable content
- Added: Bulk export signaling via `bulk-export-start`/`bulk-export-end` entries
- Added: Auto-recovery of interrupted bulk exports
- Fixed: `[dynamic]` tag on empty sections preserved for child objects
- Fixed: Source file parser correctly ignores braces inside string literals and comments
- Fixed: DataType serialization no longer concatenates domain reference with underlying type
- Added: Initial export elapsed time shown in export summary
- Added: `gx-properties-config.json` shared configuration for all GeneXus object properties
- Fixed: Property deserialization converts values to correct types based on JSON config

### Sync Manager v1.2.43
- Fixed: License tab now shows "Enter License Key" and "Buy License" when trial expires, instead of "Register Trial"
- Fixed: License type now correctly reflects upgraded licenses by using server-provided policy type

---

## 1.2.42 — 2026-03-24

### Sync Engine v1.2.65
- Fixed: Domain and Attribute deserialization now resets data-type properties before re-applying
- Added: `AverageLength` property deserialization support for Domains and Attributes
- Fixed: KBBridge no longer crashes when internal config file is empty or corrupted
- Fixed: SDT field `Collection` property was not applied during internalization for existing items
- Improved: Transaction `BusinessComponent` property serialized with readable name

### Sync Manager v1.2.42
- No additional changes (Engine binaries update only)

---

## 1.2.41 — 2026-03-23

### Sync Engine v1.2.64
- Fixed: Variable serialization now prioritizes `Attribute:` over Domain when both are set
- Fixed: Same priority fix applied to SDT field serialization
- Fixed: Source file parser correctly handles braces attached to names
- Improved: `[dynamic]` tag no longer shown on empty sections or empty main content
- Fixed: KBBridge warnings preserved alongside GeneXus validation messages

### Sync Manager v1.2.41
- Updated: AI docs — corrected `!` marker meaning (Description Attribute, not NOT NULL), documented `Nullable` transaction attribute property
- Updated: AI docs — documented SDT field DataType examples and SDT collection level item references

---

## 1.2.40 — 2026-03-22

### Sync Engine v1.2.63
- Fixed: Enumerated domain import now correctly parses `EnumValues` property
- Fixed: Transaction import now sets Primary Key (`*`) and Description Attribute (`!`) flags
- Added: Transaction attribute `Nullable` property support in serialization and deserialization

### Sync Manager v1.2.40
- No additional changes (Engine binaries update only)

---

## 1.2.39 — 2026-03-21

### Sync Engine v1.2.62
- Added: Long path support (> 260 chars) for deeply nested child objects, configurable via `EnableLongPaths`
- When disabled: objects with long paths are skipped with a separate count in the export summary
- When enabled: uses Windows extended path API (`\\?\` prefix) to support paths up to ~32K characters

### Sync Manager v1.2.39
- Added: "Enable long path support" preference in Sync Settings with informational dialog explaining Git and Windows configuration requirements
- Fixed: Console text selection no longer causes scroll to jump to the top

---

## 1.2.38 — 2026-03-19

### Sync Engine

#### v1.2.61 — 2026-03-19
- Fixed: New SDT creation failed validation due to double save
- Fixed: SDT internalization now removes items/levels no longer present in the source file

#### v1.2.60 — 2026-03-18
- Improved: Version listing shows Description, Trunk/Branch/ChDefender flags, excludes Backup versions
- Added: Support for GeneXus standard built-in types in variable and SDT field import

### Sync Manager v1.2.38
- Fixed: Console scroll position no longer drifts when AutoScroll is disabled
- Changed: Console line buffer increased from 1,000 to 50,000 to support large KB externalization processes
- Fixed: Manager now kills orphaned KBbridge.exe processes before starting a new one
- Fixed: KBbridge.exe error messages now always appear in the console
- Fixed: Installer now detects and kills both KBbridgeManager.exe and KBbridge.exe during upgrade

---

## 1.2.37 — 2026-03-18

### Sync Engine v1.2.59
- Added: SDT fields based on Attributes now serialize with `Attribute:` prefix for consistency with variables
- Added: SDT field deserialization resolves Attribute-based types

### Sync Manager v1.2.37
- Fixed: Start from console now re-reads KB settings, so changes like unchecking "Stop after Force KB Export" take effect immediately
- Fixed: Console window no longer freezes during high-frequency output bursts (throttled UI updates)
