# KBSync Changelog

All notable changes to KBSync — the GeneXus Knowledge Base synchronization tool.

---

## 1.4.32 — 2026-08-11

### Sync Engine

#### v1.7.9 — 2026-08-11
- Fixed: a build whose compilation fails is no longer reported as completed. GeneXus closes every stage of a build telling whether it succeeded, and that was only being written to the build log: a build whose Java compilation ended in "2 errors" and "Build failed." was published to the KB Editor as completed, with the object marked ok, because the object had specified correctly and no message of its own carried an error. A stage that ends in failure now makes the whole request fail. Available on every supported GeneXus version.
- Fixed: the output of the external compiler now reaches the build result. Compilers write their diagnostics as plain text rather than as GeneXus messages, so they were kept only in the engine's build log, on the server, out of reach of whoever requested the build. When a stage fails, its own output is now attached to the request — the compiler's actual errors, the failing file and line included — with a cap, and the rest stays in the log. The same applies to failures reported by the environment itself, such as a deployment folder that cannot be written.
- Fixed: on GeneXus 17 and 18, a targeted build of specific objects reported success unconditionally. That path does not return a result, and the engine assumed it worked, so an object whose outcome could not be established from the specification report was marked ok even after a failed compilation. It is now derived from whether any stage of the build failed.

#### v1.7.8 — 2026-08-10
- Added: a build is now canceled when the Knowledge Base needs a database reorganization, instead of letting GeneXus decide to run one as part of the build. Adding or changing an attribute, a transaction or a domain leaves the model's tables out of step with the environment's, and from that moment a build can reorganize the database on its own — which has already truncated real data. The engine never reorganizes: any request, whatever its scope (specify, generate or full build), is now canceled before anything is specified, stating that a reorganization is pending and in which environment. Reorganize from the GeneXus IDE, reviewing its impact analysis first, and request the build again; a Build All is recommended afterwards. When the request includes a pattern instance the pattern is still applied — what is stopped is the build, not the update of the model. As an additional safeguard, a build also refuses to execute a reorganization program that an earlier build or the IDE had already generated. On the GeneXus versions where a pending reorganization cannot be established, builds proceed as before.
- Changed: while a reorganization is pending, automatic builds no longer leave a canceled request behind after every synchronization. Objects imported meanwhile keep accumulating and are built together once the reorganization is done, and roughly every five minutes one build is still attempted, so the engine notices by itself as soon as the reorganization has been made and reports it. Builds requested explicitly from the KB Editor are never held back: each one is answered with its own canceled status and the reason.
- Fixed: the Master Page of a Web Panel now internalizes. Two causes acted together. GeneXus writes the Master Page as a reference to another object, and the import was building that reference as the generic kind used for Themes and Structured Data Types — a different, unrelated kind from the one a Master Page uses — so the Knowledge Base never received the value, for any Master Page. And the explicit selection "(none)", which means the object deliberately opts out of the model's default Master Page, was being looked up as if it were the name of a Web Panel: not found, silently dropped. Both are corrected, and "(none)" is now applied as the explicit selection it is. This mattered beyond the file: a public Web Panel that had opted out ended up inheriting a Master Page that opens the web session, and failed at runtime once deployed even though it imported and compiled without a single message. Available on every supported GeneXus version.
- Fixed: a variable removed from the externalized source is now removed from the Knowledge Base. Until now the variables section behaved as a merge: variables added or modified in the file were applied, but deleting one left it in the Knowledge Base, and it reappeared in the file on the next externalization. Variables GeneXus provides by itself are never touched, and the reconciliation only happens when the file actually declares its variables section, so a source without that section can never empty an object.
- Fixed: a property removed from a variable in the externalized source now returns to its default, the same as already happened with the properties of an object. Previously it stayed in the Knowledge Base — for example, autonumbering settings left over on a variable whose type does not even support them. Only properties the externalization itself writes are reconciled, and the whole definition of the variable's data type is left untouched, so nothing that the file does not describe can be lost.
- Fixed: an internalization that decides not to apply something the file declares now reports it. Three situations were silently discarded: a property referencing an object that is not in the Knowledge Base, a value that cannot be converted to the form the Knowledge Base stores, and any unexpected failure while applying a property. All three are now reported as warnings on the imported object, so they reach the KB Editor instead of only the synchronization log — an object no longer comes back as imported with zero messages while part of its content was dropped. Warnings do not turn the internalization into an error.

#### v1.7.7 — 2026-08-09
- Fixed: internalizing a domain no longer loses its data type definition. Three causes acted together. The import blanked the domain's length, decimals and sign before writing the file's values back, and that blanking is what survived in the Knowledge Base whenever the file already matched it — a healthy domain internalized from an identical file ended up at its default definition. And GeneXus stores a domain's length in two places, the length itself and its maximum length, while the engine wrote only the first, so a genuinely new length was brought back to the default when saving: a numeric domain became Numeric(4.0) and a 50-character one Character(20). And, above all, the data type was being written in a way that skips the normalization GeneXus performs when a type changes, leaving the internal representation the length depends on out of date — so the length did not even apply to the domain and was never stored, not when updating one and not when creating one from scratch. Enumerated domains were hit hardest — with the length back at the default, values that no longer fit were discarded, leaving the domain without values, and the re-externalization then wrote that state over the source file. The import now writes the data type the way the Knowledge Base itself does, so its whole definition is normalized, and then applies only what actually differs — length, maximum length, decimals, sign and enumerated values — clearing a setting only when it genuinely stops applying, so internalizing a file that matches the Knowledge Base touches nothing. Domains whose length matched the default were never affected, which is why the problem only surfaced on some of them; internalizing the affected files again restores them. Available on every supported GeneXus version.
- Fixed: internalizing an object no longer saves it twice. The engine applied the object's sections and saved, and then applied the properties of its header and saved again — even when there was nothing to apply — leaving two revisions in the Knowledge Base for every internalization and opening a window in which the second save could undo what the first one had stored. That is what emptied enumerated domains: the second pass returned to its default every setting the file does not name literally, and a domain's length, decimals, sign and enumerated values travel inside its data type. The header properties are now applied before the object is saved, so each internalization produces a single save with everything applied together. Available on every supported GeneXus version.
- Fixed: a build that fails for a reason that belongs to no particular object is no longer reported as completed. Errors affecting a specific object were already reported on that object, and the request itself was always marked completed; but a failure of the build as a whole — a reorganization error, for instance, which is attached to no object — ended up only as a message inside an otherwise successful-looking request, so anything reading the request's status took it as a success. A request carrying an error of its own is now reported as an error. Available on every supported GeneXus version.
- Added: the synchronization log now records the engine version as its first line, so a log always identifies the build that produced it.

#### v1.7.6 — 2026-08-07
- Improved: the properties that exist only on Work Panels — Load records, Load at startup, Automatic refresh, When to refresh, Click to sort in grid, and Execute load events in application server — now externalize with the same readable names and values shown in the GeneXus property editor, and internalize accepting those names. Previously, being exclusive to this object type, they had no readable translation and a customized value would have been written with its internal identifier. The Confirmation property needed no change: GeneXus exposes it under the same name as in Transactions, which was already translated. Available on every supported GeneXus version.
- Changed: the Work Panel form file now uses the ".gxWinForm" extension (for example "MyPanel.WorkPanel.gxWinForm") instead of ".gxForm", which stays reserved for web forms. The reason is that a Transaction carries both kinds of form at once — its Web Form and the desktop Win Form of the classic Windows/Text generation — and with a single extension the two files would have collided under the same name. Since Work Panel support had not shipped yet, no existing files are affected by the new name.
- Added: the Win Form of a Transaction is now synchronized as a companion "<Name>.Transaction.gxWinForm" file, alongside the Web Form's existing file — the same complete form definition used for Work Panels, with every reference written by name. Only Transactions whose Win Form was actually customized produce the file, so Knowledge Bases that never used the Windows generation see no new files; if a customized form is later reverted to its default, the file is removed on the next externalization. Internalization applies the file back to the Transaction, including into another copy of the Knowledge Base. On GeneXus X the Win Form is not synchronized, the same limitation as Work Panels.
- Added: the shared property catalog now describes the complete property set of the Work Panel object — including the thirty-odd properties it shares with Transactions and Procedures (window appearance and behavior, confirmation, key assignment, refresh timing, client/server data access, iSeries options) — so editors that integrate with the bridge can present every Work Panel property with its readable name, values and default, instead of only the handful that were listed for this object type. Available on every supported GeneXus version.
- Fixed: properties whose name GeneXus reuses across object types now externalize and internalize against the right property of each type. GeneXus gives the same name to different internal settings depending on the object — for example "Exposed Namespace" means one thing on a Procedure and another on a Transaction's Business Component, and the same happens with "Expose As Web Service", "Web Service Protocol", "Use Native Soap", "SOAP Action", "Display", "Type", "Data Selector" and the serialization properties of Structured Data Type members and External Object members — and in some cases the values offered also differ per type, as with Confirmation. Previously a single translation table resolved those names, so internalizing one of them on the "wrong" type could silently update a different setting than the file said, and a value only valid on one type could be written or read with no translation. Every property now resolves within the vocabulary of the object type that carries it. Existing files are unaffected: the names on disk do not change, only the setting they land on is now always the correct one. Available on every supported GeneXus version.
- Fixed: the navigation views of specified objects are no longer deleted by the periodic clean-up. The navigation of each object — kept as one file per object and replaced whenever the object is specified again — could be removed once it aged past the clean-up retention, so an object not re-specified recently lost its only navigation and the navigation list pointed to a missing file. Navigations are now kept indefinitely and can always be reviewed; the clean-up keeps trimming the build request history and the synchronization status files as before. Available on every supported GeneXus version.

#### v1.7.5 — 2026-08-06
- Fixed: in the externalized Web Form of a Web Panel or Transaction, properties that reference another object no longer carry an internal type-identifier prefix — identical for every object of the same type — that added noise to the files. The reference now travels by name, as in the rest of the externalized source. Files exported with the old prefixed form keep importing correctly and are rewritten clean the next time the object is externalized. Available on every supported GeneXus version.
- Changed: the Data Provider assigned to a dynamic combo box or dynamic list box is now written in the same reference format already used by Structured Data Type and Domain references — the object's name, followed by its module when it lives outside the root module (for example "RetEmisorSucursales, eFactura.FrontEnd") — in every file that carries it: the Web Form of a Web Panel or Transaction, the form of a Work Panel, and the properties of variables, attributes and domains in the source. Leading with the object's own name lets an editor suggest and find the Data Provider from its first letters no matter which module contains it. Files written with the previous formats keep importing correctly and are rewritten in the new format the next time their object is externalized. Available on every supported GeneXus version.

#### v1.7.4 — 2026-08-05
- Fixed: in a Work Panel form, the Item Values and Item Descriptions of a dynamic combo or dynamic list box bound to an attribute now externalize as the attribute's name. Previously they were written with an internal identifier that only made sense in the Knowledge Base that produced the file, so the assignment did not survive being synchronized into another copy of the Knowledge Base. Files already exported with the internal encoding keep importing correctly in the same Knowledge Base, and are rewritten with names the next time the object is externalized. Web Panels are not affected: their forms always carried these references by name. Available on every GeneXus version where the Work Panel form is synchronized.
- Fixed: internalizing an object with a variable whose dynamic combo or dynamic list box has a Data Provider with Parameters failed with "Deserialization error: Object reference not set to an instance of an object". The parameter list — whose externalization was fixed in v1.7.3 — was being stored back as plain text instead of the structured form in which GeneXus keeps it, and GeneXus crashed when validating the object that uses the variable. The import now rebuilds the parameter list exactly as GeneXus stores it; importing the affected object again after updating repairs a Knowledge Base that already received the bad value. Two safeguards were also added: a property value that cannot be rebuilt in the form GeneXus stores is now left unapplied and reported in the synchronization log, instead of being stored in a form that can corrupt the object; and an unexpected internal GeneXus failure while validating or saving an imported object is now reported as a clear error identifying the object and the cause, instead of surfacing as a generic deserialization message. Available on every supported GeneXus version.

#### v1.7.3 — 2026-08-04
- Fixed: a variable, attribute or domain whose Control Type is List Box, Dynamic Combo Box or Dynamic List Box now externalizes and internalizes with the correct value. Previously the translation table for that property did not match GeneXus's real internal values: List Box and Dynamic Combo Box came out swapped in the externalized file, Dynamic List Box came out as a raw number, and importing a file written with those labels could silently store the wrong control type in the Knowledge Base. The commonly used values — Edit, Combo Box, Check Box and Radio Button — were always correct and are unaffected. Files previously exported with the swapped labels are rewritten correctly the next time their object is externalized. Available on every supported GeneXus version.
- Fixed: the Data Provider assigned to a dynamic combo or dynamic list box — on a variable, an attribute or a domain — now externalizes as the Data Provider's name and internalizes by finding it in the Knowledge Base. Previously the externalized file carried an internal identifier that only made sense in the Knowledge Base that produced it, so the assignment did not survive being synchronized into another copy of the Knowledge Base. The Parameters of that Data Provider now externalize their real content as well; previously an internal placeholder was written instead of the parameter list. Additionally, if any other property whose value is a reference to an object is found without a proper name translation, it is now left out of the externalized file with a warning, instead of writing an internal identifier that other Knowledge Bases cannot resolve. Available on every supported GeneXus version.
- Fixed: the Values of a combo, radio button or list box, and the Item Values / Item Descriptions of a dynamic combo or dynamic list box, now internalize on variables, attributes and domains. They were externalized correctly, but importing them back silently did nothing: GeneXus stores those properties as structured values — a value list, a reference to an attribute — and the import was handing them over as plain text, which the Knowledge Base rejected without any message. The import now builds each value in the exact form GeneXus stores, and any property that still cannot be applied is reported in the synchronization log instead of being silently discarded. Available on every supported GeneXus version.

#### v1.7.2 — 2026-08-03
- Fixed: internalizing an object no longer marks its variables' definitions as customized when nothing changed. Previously each import re-applied every variable's data type explicitly, so values that were at their default — such as a text variable's Maximum Length, together with a materialized Average Length — appeared as explicitly-set properties in the Knowledge Base even though the externalized source never declared them, showing spurious differences in the IDE's version comparer for objects that were not really modified. The import now applies only the values that actually differ, so untouched variables keep their definitions exactly as they were. Available on every supported GeneXus version.
- Fixed: a variable that is not a collection is now always kept in its default state instead of carrying an explicit "Collection = False". Importing never writes that value anymore, and if a previous synchronization left it stored explicitly, the next import of the object cleans it up — so the version compare against the IDE-original shows no difference. Available on every supported GeneXus version.
- Fixed: the Average Length that a variable, attribute, domain or Structured Data Type member inherits from its Domain is no longer externalized as if it were set on the member itself — only a value explicitly customized on it is written. Internalizing a file that still carries the inherited value no longer turns it into an explicit one, and values materialized by earlier synchronizations are cleaned up on the next import of the object, without touching real overrides. Available on GeneXus 15 and higher, where Average Length exists.
- Fixed: internalizing a change to a Work Panel form now registers the object as modified in the Knowledge Base. Previously the form was applied correctly — the changes were visible when opening the Work Panel — but the object did not count as modified, so the re-externalization that normally follows an internalization in the next synchronization cycle never happened for Work Panels, unlike every other object type. The cycle now behaves the same for Work Panels: after a form is internalized, the following cycle detects the object and re-externalizes it. Available on every GeneXus version where the Work Panel form is synchronized.
- Added: the shared property catalog (gx-properties-config.json) now describes the properties of every Work Panel form control — the form window, buttons, labels, data controls, grids and their columns, tabs and tab pages, groups, tables, images, rectangles and lines — taken from GeneXus's own definition of the Windows form designer, so editors that integrate with the bridge can offer the complete property set of each control instead of only the properties that happened to appear in exported files. The catalog follows the same conventions as the rest of the file: list values are named with the tokens already used across the catalog (a check box is "CheckBox" here just as it is for a variable), defaults are expressed with those same names, and the designer's rules for showing, hiding or locking a property are included in the catalog's standard condition format whenever they can be evaluated from the form file alone. Properties that only make sense inside the GeneXus design environment are left out.

#### v1.7.1 — 2026-08-02
- Added: the engine now publishes its automatic-build state for the KB Editor. The environments information shared with the editor reports whether a build runs automatically after each synchronization that imports objects, and with which scope (specify, generate or full build), so the editor can tell when an import will be built on its own and when an explicit build request is needed. Available on every supported GeneXus version.
- Added: externalizing a Structured Data Type now writes every customized property of its members and levels — the Json name of a field, its XML name, namespace, type and null-serialization options, Soap type, collection serialization, descriptions, and the control, format and appearance settings — with the same readable names and values shown in the GeneXus property editor. Previously only the member's data type and collection settings were written, so changing any other member property (for example a Json Name) produced no difference in the externalized file and the change was never synchronized. Internalization applies those properties back to the Knowledge Base, and removing a customization from the file returns the property to its default value. Available on every supported GeneXus version.
- Fixed: internalizing an object now also applies the removal of its properties. Previously, a property deleted from the externalized source (for example turning off the Web Service exposure of a procedure) kept its old value in the Knowledge Base and reappeared on the next externalization: added or changed properties were applied, but removed ones were silently ignored — which could even leave contradictory combinations active. A customized property that is absent from the file is now returned to its default value, both for the object's own properties and for attribute and domain definitions, so the file behaves as the source of truth — matching how Structured Data Type members already work. Available on every supported GeneXus version.

#### v1.7.0 — 2026-08-01
- Fixed: internalizing a Structured Data Type now applies member redefinitions that change the member's shape. Previously, redefining an existing simple member as a sub-level (for example turning a member typed with another SDT into an inline collection level), or a sub-level back into a simple member, failed with "Duplicate element name" and the Knowledge Base kept the old definition; the engine now replaces the existing element with its new form, so the object always ends up matching the file. In addition, changing a member's data type between kinds (plain type, Domain, Attribute, another SDT) no longer leaves the previous type binding behind; a level whose collection item name returns to the default is updated accordingly; and renaming a member only by letter casing no longer creates a duplicate. Available on every supported GeneXus version.
- Fixed: internalizing an External Object now preserves methods, properties and parameters whose data type references a non-basic type — another External Object, a Structured Data Type or a Domain, including module-qualified references. Previously only basic types were recognized: a method returning, for example, another External Object of the same module was imported with no type at all (shown as "None" in the IDE), silently and with no message, and the problem only surfaced later as a specification error in the objects consuming it. The reference is now resolved against the Knowledge Base exactly as GeneXus does when importing an object; a Domain reference is loaded into the member's "Based on" property — just as when it is set in the IDE — deriving the base type, length, decimals and sign from the domain, so parameters and properties typed with a domain stay compatible with their callers; and if a referenced type cannot be found (for example because that object has not been imported yet), the import now reports an explicit warning identifying the member and the missing type, instead of discarding it silently. Available on every supported GeneXus version.
- Added: support for externalizing and internalizing Work Panels. A Work Panel is the classic desktop-form object from earlier GeneXus generations; current GeneXus versions still open knowledge bases containing them, but no longer offer creating one from the IDE. Each Work Panel is now written as a source file with its rules, events, conditions, variables and properties, plus a companion form file holding the complete form definition — every control with its layout and properties, and each form flavor the object defines (Windows, Text, Mobile). References inside the form — attributes, variables, images — are written by name instead of internal identifiers, so the files stay readable and synchronize correctly between different copies of the knowledge base. The round-trip works in both directions, which also makes it possible to create a brand-new Work Panel by authoring its files externally and letting synchronization import it into the Knowledge Base. Available on every supported GeneXus version; on GeneXus X — which does not expose the Work Panel form — the object synchronizes without its form and a warning is reported.

### Sync Manager

#### v1.4.28 — 2026-08-07
- Updated: AI docs (`ai/kbbridge/`) — expanded the Work Panel property catalog (`genexus-properties/workpanel.md`) from a minimal set to the full set the object supports, grouped into loading, user interface, authorization, compatibility, iSeries and client/server sections; and refined several procedure and transaction property descriptions so they reflect that the same property also applies to Work Panels and state its default and preconditions

#### v1.4.26 — 2026-08-05
- Updated: AI docs (`ai/kbbridge/`) — refreshed the Work Panel form format in `file-formats.md`: when a dynamic combo or list box takes its item values or item descriptions from an attribute or a variable, those references are now written by name instead of an internal knowledge-base identifier, so the form stays portable between copies of the KB (matching the engine fix). Also documented where a control's binding settings live so a reader knows where to look — with the control itself when it is bound to an attribute, and with the variable when it is bound to a variable — and corrected the rectangle control's tag

#### v1.4.23 — 2026-08-02
- Updated: AI docs (`ai/kbbridge/`) — documented when a GeneXus invocation must be qualified: the name is resolved by whether it is unique in the whole KB (a unique name works bare from any module; a name repeated across modules must be qualified from outside the module that defines it, or it is ambiguous). Also corrected the earlier variables guidance — qualifying a module type from within its own module is a recommendation, not a requirement (an unresolved type silently discards the variable, whereas an ambiguous invocation fails loudly)
- Updated: AI docs (`ai/kbbridge/`) — build-scope guidance: request a build of only the object(s) you changed (the channel default), not `buildCalledObjects` "just in case" — pulling the whole call tree together with `forceGeneration` can regenerate dozens of untouched objects (a measured case ran 11 minutes versus seconds for the two objects that had actually changed). And when automatic build after sync is on it already builds the imported objects, but not the objects that use them — after importing a structural type (SDT, Domain, External Object) you must explicitly request a build of its consumers, or they keep the old specification and only break at runtime
- Updated: AI docs (`ai/kbbridge/`) — how to choose between the two SDT collection forms: a loose value is a collection of scalars (`Collection = 'True'`, no block); an element made of parts is a level (`Collection = True` + `CollectionItemName` + block). A single-field level is almost always a mistake (it serializes `[{"Scope":"openid"}]` instead of `["openid"]`); the compiler flags it as `spc0023` and the fix belongs in the SDT, not the caller
- Updated: AI docs (`ai/kbbridge/`) — regenerated the `genexus-properties/` catalog so it now lists every property an SDT can customize on a member or level: the field's Json name; its XML name, type, namespace and null-serialization; Soap type; and the format, control, keyboard, behavior and appearance settings — using the readable names shown in the GeneXus property editor. This matches the engine, which now externalizes and internalizes all of those member properties, so an AI assistant can set (for example) a member's Json Name and have it synchronize instead of being silently ignored
- Updated: AI docs (`ai/kbbridge/`) — documented the Work Panel object type (the classic desktop-form object that current GeneXus still opens but no longer lets you create from the IDE): its two externalized files (a `.WorkPanel.gxSource` plus a `.WorkPanel.gxForm` form), the rule that it lives only at the project root and not inside a module, and a new `genexus-properties/workpanel.md` property page — so an AI assistant can author one externally and let synchronization create it, matching the engine's new Work Panel support

#### v1.4.22 — 2026-08-01
- Updated: AI docs (`ai/kbbridge/`) — documented several GeneXus formats and pitfalls found while authoring a new module, each of which had caused a silent failure (the object was reported "imported" or the build "ok", but was broken at runtime): the External Object file format (`.ExternalObject.gxSource`), previously undocumented; how a numeric enumerated domain must be written (unquoted value and a signed type with the `-` suffix) or its enumerated values are dropped on import; the two Collection syntaxes in an SDT; the `-` suffix for signed numeric types; that module-qualified types must be qualified even from within the same module or the variable is silently discarded; the export-request channel and the warning that "imported" does not mean complete; and new navigation message codes (`spc0017`, `spc0047`, `src0212`), the meaning of `nogenreq`, and the warning that a build reported "ok" with warnings can still be an object that does nothing
- Updated: AI docs (`ai/kbbridge/`) — documented two more silent-failure areas found while implementing a new module: which data a Data Provider can load with groups (only the own levels of its output SDT — a member typed as another collection SDT can't be opened with a group; getting it wrong compiles "ok" but returns an empty collection at runtime), and how an AI assistant should wait for and trust synchronization results (matching a request by its captured id rather than a hand-computed timestamp since ids are UTC, the expected timing per stage, and that a `specify` can return a stale prior result when only a dependency changed, so a real re-specification must be forced)
- Fixed: AI documentation is now deployed to the project with normalized LF line endings, so updating it no longer shows up as whole-file changes in the project's git caused by CRLF/LF differences
- Added: the deployed AI tool context files (`CLAUDE.md`, `AGENTS.md`, etc.) now list five more reference documents in their index — authoring order, Data Provider, navigation report, build pipeline and unit tests — so AI assistants find them without having to explore the docs folder

#### v1.4.21 — 2026-07-29
- Updated: AI docs (`ai/kbbridge/`) — documented how GeneXus reports a collapsed `For Each` in the navigation report so AI assistants stop misreading it. `genexus-navigation.md` now covers the `<Optimizations>` node (the four observed types — `FirstRows` from an `Exit`, `Aggregate` for a counter or a summed attribute, and `ServerPaging` — with what triggers each and what the SQL does), corrects the `<Formulas>` description, and warns not to read `RecordsToRead` as rows-read without checking `<Optimizations>` first (a `count(*)` level still reports `RecordsToRead=many`). `genexus-code-writing.md` adds a "Counting / testing for existence" section — which loop body produces which optimization, when to use each, and how to verify it in the navigation

## 1.4.20 — 2026-07-25

### Sync Engine

#### v1.6.0 — 2026-07-24
- Improved: externalizing an object or a variable now writes many more of its customized properties with the same readable names and values shown in the GeneXus property editor, instead of raw internal codes. Previously only a limited set of properties had readable names; the remaining ones, when customized away from their default, were written with their internal identifiers — and properties chosen from a list with their internal codes — making the files harder to read and to review. Readable names and value labels are now provided for a much broader set of properties across procedures, transactions, web panels, panels, data providers, data selectors, structured data types, external objects, groups and API objects. In addition, a number of internal, generated or tooling-only properties that used to leak into the files as raw entries are no longer written at all. The readable names are matched to each GeneXus version, so a property is shown only on the versions that actually support it. Available on every supported GeneXus version.
- Added: support for externalizing and internalizing the File object. A File keeps an arbitrary attached file inside the Knowledge Base — a library, a certificate, a properties file, a .zip, and so on — together with the options that decide, per generator (Java, .NET, .NET Framework, iOS, Android), whether and into which folder that file is extracted when the project is built. Each File is now written as two companion files inside a dedicated "#Files" folder: a readable text file with the object's name, description and extraction options, and a ".File.gxFile" file that holds the attached content, so the attachment is versioned in the repository and can be replaced through synchronization. The File round-trips faithfully in both directions, including its real file name and extension. Available on every supported GeneXus version; on GeneXus X — which has no per-generator extraction options — the attached file together with its name and description are synchronized.

#### v1.5.17 — 2026-07-20
- Fixed: internalizing the Web Form of a Web Panel or Transaction no longer corrupts grid columns bound to a collection variable. Previously, whenever such a form was internalized — even when saving it without any changes — every column of a grid tied to a collection variable lost the item indexer of its field (for example "item(0).groupId" became "groupId"), leaving the form invalid: the object stopped compiling and the corrupted form was written back to the file on the next externalization. The cause was that the form was being stored in the Knowledge Base in its editable text representation instead of being converted to the internal one; the engine now performs that conversion at internalization time — the same conversion GeneXus itself applies when importing an object — so the form now round-trips identically no matter how many times it is synchronized. Form conversion problems, previously silent, are now reported in the synchronization log. Available on every supported GeneXus version.
- Fixed: editing and internalizing a Web Component or a Web Master Page no longer fails to find its source. Their externalized files are named after their real object type (for example "MyComponent.WebComponent.gxSource" and its form), but the first line of the source declared the generic type "Web Panel"; a tool that relied on that line looked for a source file under the wrong name and reported the source as not found, so the change could not be internalized together with its form. The type stated on the first line now matches the file name, and internalizing a Web Component or Web Master Page — including one that does not yet exist in the Knowledge Base — now creates it as the correct kind of object instead of a plain Web Panel. Available on every supported GeneXus version.
- Fixed: on GeneXus X, XEv1 and XEv2, externalizing an object or a variable no longer writes out every property — it now writes only the properties whose value differs from the default (the ones shown in bold in the property editor), just like on the other GeneXus versions. Previously, on these versions, the externalized file listed every property, producing large and noisy files that were hard to diff and review. The engine now asks GeneXus itself which properties are at their default value, so the externalized file keeps only the meaningful, customized ones. Existing projects on these versions produce cleaner files on their next externalization.
- Fixed: on some GeneXus 17 installations, requesting a build (specify / generate / build) of an object could fail with a "build requests are not supported (SDK build API mismatch)" error even though the version is supported. This happened when the installed GeneXus 17 upgrade differs from the one the engine was built against and therefore does not expose the same build service; the engine now detects that and automatically falls back to the specification / generation path that the installed upgrade does provide, so the build runs instead of failing. Available on GeneXus 17 and 18.

#### v1.5.16 — 2026-07-19
- Fixed: moving a folder or module inside GeneXus now updates the externalized files. Previously only a rename (a name change) was reflected; moving a folder into another folder — or out of a module to the root — left its objects' files at the old location, because the objects inside a moved container do not themselves change and so went unnoticed. The engine now detects a relocated folder or module and moves its whole externalized directory, with every file under it, to the new location — keeping the files in sync with the Knowledge Base's organization in a single step. Available on every supported GeneXus version.
- Fixed: moving or renaming an object that owns generated sub-objects (for example a pattern instance) now carries its sub-objects to the new location as well. The object's own file was already relocated, but its contained sub-objects — which do not change when the parent moves — were left behind at the old path; they are now moved together with it. Available on every supported GeneXus version.
- Fixed: renaming an object by changing only the letter casing (for example "token" to "Token") no longer deletes its externalized source. Because the file system treats both names as the same file, the engine was removing the very file it had just written; it now recognizes a case-only rename and instead renames the externalized file — and its related files — to the new casing, so the source survives and its name matches the Knowledge Base. Available on every supported GeneXus version.

#### v1.5.15 — 2026-07-15
- Fixed: moving or renaming an object to a different module/folder now synchronizes correctly. Previously, after a move the engine kept looking for the object's source in its old location, so both the vacated old path and the updated new path were reported as errors. The incoming file's location is now taken as the source of truth: the object is internalized from its new location and relocated in the Knowledge Base to match, and the old path left behind by the move is recognized and skipped instead of failing. A source file that is simply deleted (with no corresponding move) is now reported as skipped and its Knowledge Base object is retained, rather than failing. Available on every supported GeneXus version.
- Added: the engine now understands the single-operation move/rename that newer KB Editor versions report (the object's new location together with its previous one). It internalizes the object from the new location and relocates it in the Knowledge Base in one step. The earlier way of reporting a move — as a separate removal plus a re-creation — keeps working, so older KB Editor versions continue to sync unchanged. A change it does not recognize is now reported with an explicit message instead of failing generically. Available on every supported GeneXus version.

#### v1.5.14 — 2026-07-14
- Fixed: synchronizing a Subtype Group no longer fails. Re-externalizing a Subtype Group from the Knowledge Base on request — for example when discarding its external changes after a failed import — previously reported the object as not found even though it existed in the Knowledge Base; and internalizing an existing Subtype Group could fail as if a new one had to be created. The engine now locates an existing Subtype Group by name, so both its re-export and its import work as for any other object. Available on every supported GeneXus version.
- Fixed: internalizing an object no longer fails permanently when it collides with the automatic build. While an automatic build applies a pattern it briefly locks the Knowledge Base for saving; an object being internalized at that moment previously exhausted its retries and was reported as a permanent import error. Such objects are now held and internalized automatically once the build finishes, so the collision resolves on its own without user intervention. Available on every supported GeneXus version.

#### v1.5.13 — 2026-07-04
- Fixed: on start-up, the automatic build no longer re-runs builds that had already been completed. Previously an already-finished automatic build could be executed again once its record aged out of the build history, re-specifying objects that were already up to date. Completed automatic builds are now retired as soon as their result is recorded, so they are never run a second time.

### Sync Manager

#### v1.4.20 — 2026-07-25
- Added: `genexus-properties/`, a per-object-type catalog of GeneXus properties (allowed values, defaults, applicable version) for AI assistants — one self-contained reference per object type (procedure, transaction, web panel, panel, SDT, data provider, data selector, API, external object, group, table, attribute, domain, variable) plus a shared-groups file and an index, cross-linked from the docs index and the object-context reference
- Added: File object type documentation — its two-sibling on-disk format (`.File.gxSource` for the object properties plus `.File.gxFile` for the attached binary, inside a `#Files` folder), covered across the object types, file formats and directory-structure references, with a new `genexus-properties/file.md` property page

#### v1.4.17 — 2026-07-19
- Added: documentation of the new `renamed` sync-contract changeType — when an object is moved or renamed on disk keeping the same qualified name, the change file now carries a single `renamed` entry (the new path plus an `oldRelativePath`) instead of a separate delete + create — including the entry shape, how the editor detects it, the expected engine behavior, companion-file handling, and backward compatibility with editors that still emit delete + create

#### v1.4.16 — 2026-07-16
- Added: documentation of two GeneXus For Each rules for AI assistants — the mandatory clause order (`Order` / `Unique` / `Using` before `Where`) and how to order for performance (equality-filtered attributes first so an index is used; omit `Order` to use the primary key)

#### v1.4.14 — 2026-07-14
- Added: a Data Provider + SDT file-format reference (the `Output` structure, `Order` / `Where` / `When` clauses, the no-For-Each rule, and the dynamic-combo recipe), and an authoring-order reference (the dependency order to follow when adding or changing a structure — new attributes first, then the transaction level/sublevel, then subtype groups — with the symptoms of getting it wrong and how to recover)

#### v1.4.11 — 2026-07-04
- Added: guidance for AI assistants to diagnose internalization ("won't compile / won't import") errors from the sync's own status report instead of guessing from the source — when to read it, how to tell it apart from the spec/build navigation reports, a step-by-step diagnostic workflow, the shape of each message, and a table of common internalizer error messages with their cause and fix

---

## 1.4.10 — 2026-07-01

### Sync Engine

#### v1.5.12 — 2026-07-01
- Fixed: when the engine starts, the automatic build no longer runs for objects whose changes were already synchronized on previous days. Previously, on start-up the build could be re-triggered for changes that had already been processed in an earlier session, re-specifying objects that were already up to date. Changes that genuinely arrived while the engine was stopped are still processed as usual.

#### v1.5.11 — 2026-06-27
- Fixed: the Address (SOAP endpoint) of a WSDL External Object is now externalized and internalized together with its other SOAP properties. Previously it was omitted from the externalized file, so the configured endpoint was neither preserved nor changeable through synchronization — relevant when the same web service is duplicated per environment (for example production and testing) with different addresses.
- Fixed: an enumerated Domain whose data type is the default Numeric(4,0) now externalizes its enumerated values (the Enum Values list). Previously the values were omitted from the externalized file for that particular type, so the enumeration was lost; the values are now read from the domain's underlying definition, so they are externalized for every enumerated domain regardless of its data type.

#### v1.5.10 — 2026-06-26
- Fixed: a build no longer gets stuck on machines where GeneXus shows a blocking error dialog while starting the build — for example when a code generator cannot be loaded on that particular machine. Because the build runs unattended in the background, there is nobody to dismiss such a dialog; the engine now closes it automatically (recording its text in the build log) so the build continues, just as confirming the dialog in the IDE would. Available on every GeneXus version where build requests are supported.

#### v1.5.9 — 2026-06-25
- Fixed: the automatic build that runs after synchronization no longer fails when it starts while the Knowledge Base is still internalizing objects. Previously, if the build was triggered just before internalization finished, it could fail and none of the requested objects were specified. Now the build is held as pending and automatically re-run once internalization settles; for the automatic build of internalized objects it additionally includes the same user's related objects that were still being internalized, so they are all specified together in a single batch. The wait reuses the existing post-synchronization quiet period, so no object is ever specified while internalization is still in progress. Available on every GeneXus version where build requests are supported.

#### v1.5.8 — 2026-06-22
- Fixed: a build of specific objects — including the automatic build after synchronization — now always specifies, generates and compiles the requested objects, instead of occasionally skipping a freshly-internalized object that GeneXus considered already up to date (the object was reported as built but its generated code was not refreshed). A whole-Knowledge-Base "Build All" keeps its incremental behavior, reprocessing only the objects that actually changed. Available on GeneXus 17 and 18.

### Sync Manager

#### v1.4.10 — 2026-07-01
- Updated: AI docs (`ai/kbbridge/`) — added guidance for duplicating a GeneXus object (Save Object As): for every object type, which sibling files must be copied together (`.gxForm` alongside `.gxSource` for Transactions/WebPanels/WebComponents/Panels; `.gxLayout` alongside `.gxSource` for Procedures with a report layout; single-file objects such as SDT, Data Provider and Data Selector), how to rewrite the first line of the `.gxSource` with the new object name, and what must not be touched (form/layout content, internal identifiers, existing target files) — so AI assistants no longer produce half-copies that leave the duplicated object structurally broken

#### v1.4.7 — 2026-06-25
- Updated: AI docs (`ai/kbbridge/`) — added `build-pipeline.md` documenting the Sync Engine's build pipeline for AI assistants: how to request builds (the build-request files, their fields and id format) and how to read the results (build status with per-object and per-message detail, the available environments and their lock, and the current navigation set per object and environment); and expanded `genexus-navigation.md` documenting the externalized GeneXus navigation report so AI assistants can read it — including how to determine the base table and its extended table (the foreign-key joins that resolve foreign attributes), and how to understand errors and warnings that were not otherwise surfaced

---

## 1.4.6 — 2026-06-19

### Sync Engine

#### v1.5.7 — 2026-06-19
- Added: code generation and full build (including compilation) of specific objects on request is now also available on GeneXus versions earlier than 17 — XEv1, XEv2, XEv3, 15 and 16 — not only on 17 and 18. As on the newer versions, the "force generation" option re-specifies and regenerates the requested object even when GeneXus considers it up to date, and the engine never triggers a database reorganization as part of a build. (A specification-only request and a full "Build All" were already supported on these versions; if a particular version cannot perform the targeted build, the engine falls back to specifying the object so the request still returns its navigation and errors.)

#### v1.5.6 — 2026-06-18
- Fixed: a build that generates or compiles now reports its complete output — the specification, generation and compilation messages — attributed to each object, and shows the object's correct result. Previously those messages, produced while the build ran, were not captured, so a generated or compiled object could finish with no messages and be shown as skipped even though it had actually been built. Available on every GeneXus version where build requests are supported.
- Fixed: requesting code generation or a full build for a single object now regenerates that object's source (and, for a full build, compiles it), instead of only specifying it. The "force generation" option re-specifies the object as well — refreshing its navigation report and regenerating its code even when GeneXus considers it already up to date; without the option, only what actually changed is reprocessed. The engine still never triggers a database reorganization as part of a build. Available on GeneXus 17 and 18.
- Changed: a generate or build of specific objects now targets exactly the requested object(s) — it no longer re-specifies and rebuilds the whole tree of objects they call. A new request option lets the KB Editor additionally include the called objects (the object's call tree) when that is wanted. Available on GeneXus 17 and 18.

#### v1.5.5 — 2026-06-16
- Fixed: build requests now specify against the resolved environment's preferences (the one fixed in the configuration, or the one chosen in the request), instead of ignoring them. Previously the specification ran without the environment's generator settings, so a check the environment defines as a warning — such as "Check Type Errors" — was reported as an error and the object as "Specification Failed", differing from the GeneXus IDE. Now the build result matches what the IDE reports for that environment.
- Fixed: in a Build All, the specification warnings and errors that belong to a specific object are now reported on that object (which appears in the result with its messages), instead of being grouped in the build's general message list. The general list now keeps only messages that cannot be linked to a particular object (such as compilation/linker output).
- Fixed: each build message now carries the source section it refers to (Events, Rules, Source, etc.), the same way the internalization messages already do, so the KB Editor can place a clicked build error on the exact line of the externalized source. Object/compilation-level messages with no source line carry no section, as before.
- Fixed: a build that includes code generation now actually writes the generated source code (for example `.java`/`.cs`) against the selected environment, instead of stopping after the specification phase. Previously the scopes that should generate produced only the navigation and the semantic checks, without ever writing the generated source. The engine still never triggers a database reorganization as part of a build. Available on every GeneXus version where build requests are supported.
- Added: each build message now indicates the phase that produced it — specification, generation or compilation — so the KB Editor can group the build results by origin. A "specify only" build reports every message as specification, as before.
- Fixed: a build result now records whether it was a "Build All". The flag is copied onto the build result the same way the scope and environment already are, so the KB Editor can keep showing the "Build All" indicator on a finished batch even after it discards the original request.
- Improved: a build in progress is now reliably terminated when the engine stops — including when the engine is forcibly closed or crashes, not only on a normal stop — so a forced shutdown no longer leaves a build (or its compiler) running in the background holding the Knowledge Base.

#### v1.5.4 — 2026-06-15
- Fixed: the specification messages (errors and warnings) of a requested build are now reported on the specific object they belong to, instead of being grouped in the build's general message list. The general list is now reserved for build/compilation messages that cannot be attributed to a particular object. Available on every GeneXus version where build requests are supported.
- Added: the KB Editor can request a full "Build All" — the engine specifies the whole Knowledge Base and reports only the objects that GeneXus actually re-specified with changes (objects reported as unchanged are not listed), each with its result and navigation report. Useful after broad-impact changes such as a Domain's enumerated values. Available on every GeneXus version where build requests are supported.
- Added: when a change to a structural object that is not itself specified (Domain, Attribute, SDT, Table, External Object) is internalized, it now appears in the build batch with a note that it was not specified and that a Build All is recommended, since changes to those object types can affect the specification of other objects.
- Fixed: internalizing a Pattern Instance now preserves the line breaks and tabs inside its code-valued properties (for example an action's post-code). Internalization now normalizes the escaping of those properties on import, so the code is stored with its original line breaks even if the file arrived with extra escaping accumulated from a previous round-trip. Available on every GeneXus version where Pattern Instances are externalized.
- Fixed: an object included in a build that is not specifiable on its own (for example a Structure Data Type generated by a pattern) is now reported as skipped in the build result, instead of being left without a result. Previously such objects could appear in the build status with an empty status, which the KB Editor showed as a never-resolving "pending" badge on an already-finished build.

#### v1.5.3 — 2026-06-14
- Added: automatic builds are now attributed to the user whose changes were internalized. The build still runs on the engine side, but each automatic build carries the user that triggered it, so the KB Editor can show every developer the build result of their own internalized changes.
- Changed: automatic builds are no longer triggered on every synchronization cycle. The engine now waits for a short quiet period with no further internalization and then builds together all the objects accumulated during that window — avoiding partial builds when many objects arrive across several cycles. The quiet period (in cycles) and a safety cap are configurable; any objects still pending when the engine is stopped are queued so they build on the next start.
- Added: the engine now clearly reports when it cannot re-externalize an object because its last internalization failed (a safeguard that protects the modified source file from being overwritten). The notice is always shown on the engine console and recorded for the KB Editor, so this situation is no longer silent.
- Added: a way to discard a failed internalization. From the KB Editor the user can request to discard the external changes of an object and have it re-externalized from the Knowledge Base — restoring consistency between the Knowledge Base and the externalized directory and clearing the block that was preventing the object's export.

#### v1.5.2 — 2026-06-13
- Added: the engine now publishes the Knowledge Base's available environments and which one is active, so the KB Editor can offer an environment selector for builds. A build request may indicate the environment to work against; if it does not, the active environment is used (unchanged behavior). The chosen environment is reported back with each build result. The engine can also report the available environments on demand — before synchronization is started — so a manager can populate the same selector at configuration time. The engine's environment switch is per-user, so it never changes the environment of a developer working in the GeneXus IDE on the same Knowledge Base.
- Added: optional automatic build after synchronization. When enabled in the configuration, after each cycle that successfully internalizes buildable objects the engine queues a build for them automatically (default scope: specification only — fast, produces navigation and semantic errors). These automatic builds appear to the KB Editor alongside the manual ones.
- Added: a configuration setting can fix the environment used for all builds (automatic and manual). When set, the KB Editor shows the environment as locked and does not let the user change it.
- Added: a consolidated navigation view: the engine keeps the latest navigation result per object per environment, mirroring the GeneXus IDE's Navigation tab — the current navigation set, not a build-by-build history (the history remains available in the build status). The KB Editor can show the last batch, the whole accumulated set, or search by object.

#### v1.5.1 — 2026-06-12
- Added: when a build request includes a Pattern Instance, the engine now applies the pattern first (the same "Apply Pattern" action of the IDE), generating or refreshing the instance's child objects, and then includes every generated object — plus the object the instance is defined under (for example the Transaction of a Work With), which the apply also updates — in the requested specification/generation/compilation. Nested pattern instances generated by another pattern are applied recursively. The instance's entry in the build status reports the outcome of the apply step with its own messages, and each generated object appears with its own result and navigation report. If the apply of an instance fails, neither its generated objects nor its parent are built (the rest of the request continues normally). Available on every GeneXus version where build requests are supported
- Fixed: a Domain whose data type is the GeneXus default Numeric(4.0) now externalizes its Enumerated Values and its Collection property. Previously both were emitted only when the Domain had a non-default data type, so a Numeric(4.0) Domain with enumerated values silently lost them in the `.gxDomain` file (and the round-trip back into the Knowledge Base dropped them). Domains with any other data type were unaffected. Available on every supported GeneXus version
- Added: the navigation report of each built object is now also delivered as a self-contained HTML file with the same visual style as the GeneXus IDE's Navigation Report — styles and icons embedded, ready to be displayed by the KB Editor without any extra processing. The raw report remains available alongside it for diagnostics. If the HTML rendering is not possible (for example the GeneXus installation does not include the report stylesheets), the raw report is delivered instead with a warning, and the build itself is never affected
- Improved: the build console now lists every object of the request when it starts and reports each object's final result (`Specified <object>...ok/warnings/errors`) when it finishes — including objects that did not need re-specification, which previously produced no per-object output at all
- Changed: the build status file is now named `external-<user>-build-status.json` (previously `kbbridge-<user>-build-status.json`), following the sync folder naming convention: files of flows initiated from the KB Editor — both the requests and their responses — use the `external-` prefix, while the `kbbridge-` prefix is reserved for flows initiated from the Knowledge Base side (such as the exports file). Any stale file with the old name can be deleted manually

#### v1.5.0 — 2026-06-11
- Added: the engine can now specify, generate and compile a set of Knowledge Base objects on request from the KB Editor. Requests are dropped as JSON files in the project's sync folder, queued, and executed one at a time while synchronization continues uninterrupted — a long compilation never delays the import/export cycle. For each object the engine returns its navigation report (viewable from the KB Editor) and every specification or compilation error associated to the exact object that caused it, including the message code, line and column when available. Three scopes are supported per request: specify only (fast — navigation plus semantic errors), specify + generate, and full build including compilation. The engine never triggers a database reorganization as part of these requests. Requests survive engine restarts: anything left queued or running when the engine stopped is re-queued automatically on the next start. Available on GeneXus XEv1, XEv2, XEv3, 15, 16, 17 and 18 (on GeneXus X the request is answered with a clear not-supported message)
- Added: build activity is reported on the engine console prefixed with `[Build]`, and a complete per-request log is appended to `kbbridge-build-output.log` next to the engine configuration file, with a separator banner per request and automatic rotation when the file grows beyond 10 MB. A new `--no-build-console` command-line option suppresses the console relay for managers that read the log file directly as a separate channel

### Sync Manager

#### v1.4.1 — 2026-06-14
- Added: build settings per Knowledge Base — an optional automatic build after each sync that imports objects (with a scope choice of specify only, specify + generate, or full build) and an optional build environment. Leaving the environment empty keeps it unlocked (manual builds in the KB Editor choose their own); setting one forces that environment for all builds and locks the selection in the KB Editor
- Added: a "Detect" button next to the build environment that lists the Knowledge Base's GeneXus environments to choose from; if detection is unavailable, the environment can still be typed manually. When it fails, the actual reason reported by the engine (for example, the KB could not be opened) is now shown instead of a generic message
- Added: when automatic build after sync is on, you can now set how many quiet sync cycles to wait before building the grouped imported objects, plus a maximum number of cycles to wait as a safety cap — so a burst of imports results in a single build instead of one per cycle (an approximate timing in seconds is shown based on the sync interval)

#### v1.4.0 — 2026-06-12
- Added: the console window now has two channels selectable from a dropdown in the toolbar — General (synchronization output) and Build (output of build requests coming from the KB Editor) — each keeping its own search, scroll position, clear, copy and auto-scroll. A green dot next to the selector signals new activity in the channel not being watched. The Build channel shows only the output produced since the synchronization was last started, and stays empty when the Sync Engine does not support builds
- Added: the log viewer window now has the same General / Build channel selector — General shows the synchronization log file and Build shows the build log file, both with their full history and timestamps

---

## 1.3.0 — 2026-06-09

### Sync Engine v1.4.29
- Fixed: importing an object whose path on disk crosses through nested folders inside a module (for example a file under `@PXTools\@APIs\Personalized\Templates\`) no longer fails with `Folder '<module>.<inner>' already exists.` when one of those folders already lives in the Knowledge Base. In GeneXus, a Folder's unique identity inside a module is just `<module>.<folder name>` regardless of how many parent folders sit above it (folder names are unique within a module). The engine was looking up the folder using the accumulated disk path (`<module>.<parent folder>.<inner folder>`), missing the existing folder, falling through to "create new" and getting rejected by the GeneXus SDK as a duplicate. The lookup now uses the correct identity — module plus folder name only — so existing folders are reused and only genuinely-new folders are created. Available on every supported GeneXus version

### Sync Manager v1.3.0
- Added: search bar in the console and log viewer window — open with Ctrl+F or the Find button in the toolbar, type to jump to the next match, use ↑/↓ (or Enter / Shift+Enter, or F3 / Shift+F3) to navigate between matches, with a "current of total" counter shown on the right. Auto-scroll is paused while the search bar is open so the viewport stays on the highlighted match across log refreshes, and resumes its previous state when the bar is closed with Esc or the ✕ button
- Fixed: console viewer auto-scroll no longer stays stuck after a click inside the output box — clicking placed the keyboard cursor at the click position and on the next refresh the viewport was pulled back to that position instead of staying at the latest line, so auto-scroll appeared to stop working until the checkbox was toggled off and on. The cursor is now moved to the end of the text before each scroll, so auto-scroll keeps following the latest output regardless of where the user clicked

---

## 1.2.79 — 2026-06-04

### Sync Engine v1.4.28
- Added: when a Domain shares its name with a built-in type (for example a Domain called `Boolean`, `Date`, `DateTime`, `Image`, `Video`, `Audio`, `Blob`, `BlobFile`, `GUID`, `GeoLine`, `GeoPoint`, `GeoPolygon` or `Geography`), the engine now emits the explicit prefix `Domain:` in the `DataType` property — e.g. `DataType = 'Domain:Boolean'` — so the round-trip back into the Knowledge Base preserves the binding to the Domain instead of degrading it to the built-in type with the same name. The prefix is added only when there is an actual name collision and no module qualifier is present (a reference that already includes the module, like `'Boolean, Sales'`, is unambiguous on its own and is emitted as before). Variables on every applicable object (Procedure, Data Provider, API, Data Selector, Transaction, Web Panel, Panel, SDT items, etc.), Attribute definitions, and Domains that inherit from another Domain are all covered. Files exported with the previous engine keep working — re-exporting them produces the new disambiguated format. Available on every supported GeneXus version

---

## 1.2.78 — 2026-06-02

### Sync Engine

#### v1.4.27 — 2026-06-02
- Fixed: importing a new Domain that has Enumerated Values defined in its source file no longer fails with `Object reference not set to an instance of an object.`. The previous flow assumed the Domain already carried an Enumerated Values collection — true for Domains that existed in the Knowledge Base before the import, but not for brand-new Domains being created by this same import. The engine now initializes the collection on the fly when missing, so a `.gxDomain` like the one declaring `EnumValues = [ ... ]` for a freshly created Domain is internalized cleanly. Available on every supported GeneXus version
- Fixed: a Procedure / API / Data Provider / Data Selector variable whose `DataType` references a Domain located inside a module (e.g. `DataType = 'OrderStatusKind, Sales.Orders'`) is now correctly bound to the Domain after import. Previously the variable arrived in the Knowledge Base with its type set to "Domain reference" but pointing at no Domain at all — the IDE showed it as a Domain-based variable with the binding blank. Variables whose Domain lives at the root (no module) were unaffected. Available on every supported GeneXus version
- Fixed: externalizing a Pattern Instance (PXWorkWith, SDWorkWith, PXParameterRequest, PXComposer, PXOAV, etc.) now produces a `.gxPattern` file where the references to Attributes, Domains and other Knowledge Base objects appear by their plain name (e.g. `<attribute name="WebServiceLogStartDateTime" />`). Previously, when the project lived in a path that triggered an internal loader to reject the schema definitions, the engine fell back to emitting these references prefixed with a long type identifier (e.g. `<attribute name="adbb33c9-0906-4971-833c-998de27e0676-WebServiceLogStartDateTime" />`), making the file noisy to read and breaking re-import. The schema definitions are now read through a path-safe route. Files already on disk in the prefixed format will be re-emitted as plain names on the next sync cycle. Available on every supported GeneXus version
- Fixed: the dependency tier ordering of the synchronization phase now correctly applies SDTs before Procedures (and other objects that may reference them). The earlier release placed SDTs in the same dependency tier as Tables and External Objects (intended to be processed before Procedures and UI objects) but the type-name lookup did not match the actual `.gxSource` file naming — files named `<name>.StructuredDataType.gxSource` were treated as "unknown type" and bumped to the last tier, so the SDK ended up applying Procedures, Data Providers, Web Panels and Pattern Instances before the SDTs they reference. WebMasterPage and Group objects were similarly mismatched. The lookup now uses the actual on-disk type names (`StructuredDataType`, `WebMasterPage`, `Group`), so the documented tier order takes effect. Available on every supported GeneXus version

#### v1.4.26 — 2026-06-01
- Fixed: the synchronization phase now applies pending changes in dependency order — Domains, Folders, Themes and Images first, then Attributes, then SDTs / External Objects / Tables, then Transactions, then Procedures / Data Providers / APIs / Reports / Work Panels, then Web Panels / Panels / Master Pages / Web Components, and finally Pattern Instances. Previously the order was strictly chronological, so an editing session like "create Attributes → create Transaction → modify one of those Attributes" left the Transaction being applied before the modified Attribute had been internalized, causing the Transaction import to fail because the Attribute (in its latest state) was not yet in the Knowledge Base. The new ordering guarantees that an Attribute always reaches the KB before any Transaction that references it, an SDT before any object that uses it, and so on. Within each tier the user's chronological order is preserved
- Added: automatic retry pass at the end of every sync cycle. If a Knowledge Base object failed to internalize because a dependency it references was scheduled for processing later in the same cycle, the engine now re-tries that object after the rest of the cycle has finished, and keeps re-trying until a full round produces no progress. Permanent failures (those that did not recover after the retry pass) are reported as `error` in `external-<user>-status.json` only at the end of the cycle — never prematurely. While a retry is still possible, the console line for the failure shows the suffix `Fail (will retry)` so the user can tell the difference between a transient ordering failure and a final error. The retry-pass banner `Retry pass: N work item(s) pending dependency resolution` precedes the second pass when any objects are pending. Available on every supported GeneXus version

---

## 1.2.76 — 2026-05-28

### Sync Engine v1.4.25
- Fixed: externalizing a Procedure that comes from a referenced module (for example procedures under `GeneXus.SD.Synchronization` or other system modules) no longer fails with `Layout serialization error: The given key was not present in the dictionary.`. These procedures do not have a Layout to externalize and are now treated as such — silently skipped — instead of being reported as failures. Available on every supported GeneXus version

---

## 1.2.75 — 2026-05-23

### Sync Engine

#### v1.4.24 — 2026-05-22
- Fixed: the GeneXus 18 IDE no longer shows a "this Knowledge Base was last opened by a different installation" warning after the engine has run a sync cycle. The engine now snapshots the `.gxw` file immediately before opening the KB and restores it as soon as the open succeeds, so the file keeps reflecting the IDE's last open — not the engine's. Available on GeneXus 18

#### v1.4.23 — 2026-05-21
- Fixed: Variables of a Panel typed as a **user-defined External Object** from the Knowledge Base are now correctly recognized when the panel is imported. Previously these variables were silently degraded to Numeric(4,0) — losing their type and breaking any code that called methods specific to the External Object. Completes the parallel fix for built-in runtime External Objects (HttpClient, File, Properties, XMLReader, etc.) shipped earlier. Available on GeneXus XEv3, 15, 16, 17 and 18
- Fixed: importing an API object that already exists in the Knowledge Base no longer fails with `API '<name>' already exists`. Edits to existing APIs are now applied correctly. Available on GeneXus 17 and 18

### Sync Manager

#### v1.2.75 — 2026-05-23
- Updated: AI skills (`ai/kbbridge/*.md`) — documented the new `.gxLayout` extension (Procedure print layout) with XML structure, control types and coordinate system; expanded `.gxForm` coverage with the three formats broken down by root tag (`<body>` for WebPanel/WebComponent, `<GxMultiForm>` for Transaction, `<layouts>` for SDPanel); documented the `dynamic="true|false"` root-tag attribute and how it differs from the existing `[dynamic]` section tag; removed legacy `.gxProperties` mentions (extension retired — object-level properties now live inline in `.gxSource` after the header)

#### v1.2.74 — 2026-05-22
- Fixed: AI documentation deployment now detects and replaces the managed block regardless of marker capitalization, and auto-heals any leftover duplicate managed blocks left behind by older buggy deploys — previously, files that had been deployed by a version using the older `KBBridge:` (uppercase B) markers would get a second `KBbridge:` (lowercase b) block prepended on every new deploy because the marker search was case-sensitive

#### v1.2.73 — 2026-05-21
- Fixed: "New version available" dialog no longer closes together with the splash screen during startup — the dialog used to inherit the splash as its Owner and was force-closed when the splash's auto-close timer fired

---

## 1.2.72 — 2026-05-19

### Sync Engine

#### v1.4.22 — 2026-05-19
- Fixed: Procedure externalization no longer reports `Unchanged` in the console when only the sibling `.gxLayout` (or, for WebPanel / Transaction / SDPanel, the sibling `.gxForm`) was modified but not the `.gxSource`. The engine now defers the `Unchanged` / `Success` decision until after every sibling file has been processed, so the console line accurately reflects the overall write outcome. Available on GeneXus XEv3, 15, 16, 17 and 18

#### v1.4.21 — 2026-05-16
- Fixed: Panel internalization now also preserves variable DataType on the way into the Knowledge Base — Image, Audio, Video, Blob, BlobFile, GUID, geographic types, Business Component (with and without module) and built-in runtime External Objects (HttpClient, HttpResponse, File, Directory, Properties, Property, XMLReader, ExcelDocument, etc.) now reach the SDK with their proper type set, so events code that references their methods parses correctly. Available on GeneXus XEv3, 15, 16, 17 and 18
- Improved: import status file (`external-<user>-status.json`) now carries the **full list of SDK validation errors** with `section`, `line` and `column` instead of the bare `"message": "Error"` that used to be all that survived `ValidationException` propagation
- Fixed: when a single GeneXus object has both files pending in the same sync cycle (e.g. Panel's `.gxSource` and `.gxForm`), the status file no longer reports one as `"imported"` and the other as `"error"` — both entries now describe the same outcome
- Improved: SDPanel diagnostic XML files dropped in temp directory on import failure now carry the same `<ExportFile><KMW>...<Source>...<Objects>...</Objects></ExportFile>` shell that GeneXus IDE accepts via Knowledge Manager → Import → File
- Improved: SDK-flow diagnostic lines (`DIAGNOSTIC:` traces during SDPanel internalization) now go to the bridge log file only and no longer flood the console

#### v1.4.20 — 2026-05-15
- Fixed: Panel externalization now preserves the DataType of variables typed as Image, Audio, Video, Blob, BlobFile, GUID, geographic types, External Object (with and without module) and Business Component (with and without module). Previously variables of those types were silently downgraded to Numeric(4,0), which then caused `Unknown function` and `is ambiguous` errors on re-import. Available on GeneXus XEv3, 15, 16, 17 and 18
- Fixed: Panel internalization no longer leaves the layout one level too deep inside the pattern instance XML, which previously caused control references in events / rules / conditions to fail with "Invalid attribute X" / "Unknown function X" errors during the SDK's final save. Available on GeneXus XEv3, 15, 16, 17 and 18
- Added: when Panel internalization on GeneXus 17 / 18 fails because the SDK's parser rejects events / rules / conditions code during save, the engine now dumps a manual GeneXus XPZ-format XML file to the system temp directory (`kbbridge-xpz-<panel>-<timestamp>.xml`) that can be imported through Knowledge Manager → Import → File for diagnosis
- Improved: when post-import save fails with a `ValidationException` carrying the generic message "Error", the bridge now dumps every public property of the exception (and its inner exceptions, up to 6 levels deep) via reflection, including collection-typed properties

#### v1.4.19 — 2026-05-14
- Changed: Panel internalization on GeneXus 17 and 18 now applies all sections (variables, form/layout, rules, conditions, events) atomically through a single SDK import call instead of five separate write-back rounds. The previous rounds invoked the parser against a still-incomplete panel, producing hundreds of false-positive errors and leaving the panel empty in the IDE. The engine now serializes the panel's envelope, splices the complete content of every section into the pattern's instance XML in one pass, and re-imports it through `BLServices.KnowledgeManager.ImportInObject`. Available on GeneXus XEv3, 15, 16, 17 and 18

#### v1.4.18 — 2026-05-08
- Added: `.gxForm` files now carry a `dynamic="true|false"` attribute on the root tag that marks whether the form was customized by the user (round-trip enabled) or auto-generated by GeneXus from a template or pattern (round-trip skipped). Files without the attribute fall back to a contextual decision: new objects always receive the form, existing objects with no pending change and a default form in the KB skip the import
- Added: PatternInstance internalization (`.PXWorkWith.gxPattern`, `.SDWorkWith.gxPattern`, `.WorkWith.gxPattern` and any other registered pattern) is now also supported on the older GeneXus runtimes — GeneXus X, XEv1 and XEv2 — closing the previous gap where the engine could only export `.gxPattern` files on those versions. PatternInstance round-trip now works end-to-end across the entire supported version range
- Added: Procedures now externalize and internalize their visual print Layout (printable region with bands, labels, attributes, images, lines and rectangles) through a new `<ObjectName>.Procedure.gxLayout` file paired alongside the existing `.gxSource`. Only emitted when the layout has been customized; reset to default deletes the obsolete `.gxLayout` from disk on next sync. Round-trip works on every supported version of GeneXus
- Added: `.gxLayout` files carry a `dynamic="true|false"` attribute on the root tag with the same semantics as `.gxForm`

### Sync Manager v1.2.71
- Fixed: Log viewer with auto-scroll no longer jumps to the top of the log on every refresh — the ScrollToEnd call now runs after the WPF binding has applied and the new content has been measured

---

## 1.2.64 — 2026-05-08

### Sync Engine

#### v1.4.17 — 2026-05-07
- Changed: PatternInstance round-trip (export and import of `.gxPattern` files) is now schema-driven and works for **any pattern** registered in the GeneXus environment — not just the PXTools-shipped ones (PXWorkWith, SDWorkWith) that the previous hardcoded implementation supported. The engine reads the pattern's instance specification XML from the externalized `Patterns/<patternName>/` folder to identify which attributes are References, instead of relying on a fixed list of XPaths and attribute names. Built-in patterns (Conversational, Dashboard, PXAudit, PXReportTemplate, PXOAV, etc.) and any third-party or custom pattern installed in the KB are now supported automatically
- Performance: pattern instance schemas are parsed lazily on first use and cached in memory for the duration of the sync engine process
- Improved: multi-type references (`reference(WebPanel; SDPanel)`, etc.) are correctly resolved on import — the engine tries each candidate type in declaration order until one resolves the value to a KBObject
- Fixed: Panel internalization now persists the rules, events, conditions and form that the source file declares — both on creation and on subsequent edits. Earlier round-trips silently dropped every part-level change while reporting "Success" because the SDK's wrapper part required reflection-based access to the inner `PatternInstancePart` and a different write-back path (`PatternBasePart.ReadFrom`) than was being used. Available on GeneXus 17 and 18
- Fixed: Panel **Variables** are now also internalized — user-defined variables declared in the `#Variables` section of the `.gxSource` are added or updated on the panel through the same pipeline used by Procedure / DataProvider / API / DataSelector / Transaction / WebPanel. Available on GeneXus XEv3, 15, 16, 17 and 18
- Fixed: PatternInstance internalization (e.g. `.PXWorkWith.gxPattern`, `.SDWorkWith.gxPattern`, `.WorkWith.gxPattern` and any other registered pattern) now actually persists the edits — the previous deserializer reported `Success` but silently dropped every change. Available on GeneXus XEv3, 15, 16, 17 and 18

#### v1.4.16 — 2026-05-05
- Fixed: Panel internalization now also writes back the Variables section. Previously the deserializer populated a freshly-constructed VariablesPart that was never bound to the panel, so the parsed variables were dropped on the floor. Available on GeneXus 17 and 18
- Fixed: Internalization no longer rejects edits to existing PatternInstance objects whose pattern is anything other than SDWorkWith — the by-name lookup now iterates every pattern registered in the GeneXus environment and matches the first one that has an instance with the qualified name. Available on every supported GeneXus version
- Fixed: PatternInstance internalization no longer fails with `Source file not found` when the file's first line carries the SDK display name with a trailing pattern-type and `[]` metadata. The header parser now extracts only the simple object name from the second token and reads the pattern-type name from the third token
- Changed: PatternInstance exports now write the header line as `PatternInstance <SimpleName> <PatternType>` deterministically, instead of relying on the SDK's inconsistent `Name` property
- Added: Table internalization now applies changes to the `#Indexes` section of a Table `.gxSource` back to the Knowledge Base. The rest of the Table (physical / logical / redundant attributes and table relations) remains read-only because GeneXus derives those from the underlying Transactions. The engine differentiates auto-generated indexes (`I*`, immutable composition) from user-defined indexes (`U*`, fully editable) and uses structural-signature matching to identify auto-generated indexes across renames
- Limitation: when a user-defined index is renamed and restructured in the same edit, the engine falls back to delete-and-create and the original `Description` is lost. Workaround: do the rename in one import and the restructure in the next
- Auto-recovery: if a `I*` index block is accidentally removed from the `.gxSource`, the engine preserves the original index in the KB, emits a warning, and the next sync iteration re-exports the file with the auto-generated block restored

### Sync Manager v1.2.62
- Added: Automatic check for new Sync Manager versions — informs the user when an update is available with options to download, skip, or remind later

---

## 1.2.61 — 2026-04-30

### Sync Engine

#### v1.4.15 — 2026-04-30
- Fixed: Internalization of a brand-new Panel object no longer aborts with `Root element is missing` — the engine read the panel's variables from an XML attribute that does not yet exist on a freshly created panel, throwing `XmlException` and aborting the import. The variable reader now treats a missing or empty `variables` attribute as "no variables", and the form reader is equally robust against an uninitialized layout. In addition, the panel is persisted once at the start of source deserialization so the underlying KMW pattern-part skeleton exists before rules, events, conditions and form are written into it — without this initial save, all four sections were silently dropped on creation, producing an empty panel even when the import reported success. Available on GeneXus 17 and 18

#### v1.4.14 — 2026-04-29
- Fixed: Internalization of new Panel objects no longer fails with the generic error `Could not create object: Panel <name>` — previously, importing a `.Panel.gxSource` whose object did not yet exist in the Knowledge Base produced a silent failure because Panel creation was not implemented in the object factory. Panels are now created on demand during import using the GeneXus generic object factory, so a fresh Panel can be internalized end-to-end without first creating it manually in the IDE. Supported on GeneXus XEv2, XEv3, 15, 16, 17 and 18
- Added: Internalization now creates PatternInstance objects on demand for any pattern installed in the GeneXus environment — the pattern type is detected from the `.gxPattern` filename and resolved against the registered pattern definitions, so `.<TypeName>.gxPattern` files for any installed pattern can be imported end-to-end without first creating the instance manually in the IDE. Available on every supported GeneXus version
- Improved: Import error messages distinguish between three failure modes — the type is not registered in the Knowledge Base, the GeneXus factory rejected the create request, and unknown header type — so the diagnostic points to the actionable cause

#### v1.4.13 — 2026-04-28
- Fixed: Pattern definitions and User Controls export no longer crashes with `UriFormatException: Invalid URI: The hostname could not be parsed` when long path support is enabled in Sync Settings — the relative-path computation that previously relied on `System.Uri` could not parse paths prefixed with `\\?\`. The path is now computed by direct string comparison, which also makes the export robust against file or folder names containing characters that the URI parser handles specially (such as `#`)
- Fixed: Variable internalization no longer writes the `Collection` property explicitly as `False` on non-collection variables — the property is now only updated when its current value differs from what the source file declares. Forcing `Collection = False` on every import marked the property as user-defined (instead of leaving it at its default), causing spurious dirty state on variables that never were collections. Switching a variable from collection to non-collection (and vice versa) still works as expected
- Added: Sync iterations that produce a burst of exports now bracket the burst with `sync-batch-start` and `sync-batch-end` markers in the exports JSON file, analogous to the existing `bulk-export-start`/`bulk-export-end` markers used for the very first export. The new markers fire on the first iteration after startup whenever there is at least one pending change (covering the case of resuming a sync that was paused for hours or days) and on any later iteration whose pending count reaches the new `SyncBatchMarkerThreshold` setting (default 5). External tools that import file changes — for example, the KB Editor sync component — can pause their file-system watcher for the duration of each marked window to avoid mistaking KBBridge's own exports for external edits. Orphaned `*-start` markers from an interrupted run are auto-closed at startup, independently per marker family

#### v1.4.12 — 2026-04-26
- Added: External User Controls are now exported to a new `User Controls/` folder in the project — for each User Control installed in the GeneXus environment, the design-time files (`.control` manifests, properties XML, render XSL, toolbox icons, and images) are copied preserving the original folder structure. Vendor packs that group several controls under a single folder (e.g. `DVelop/`) are exported as a single tree retaining all `.control` files at the root and their referenced asset subfolders. Runtime material (`.js`, `.css`, `.dll`, etc.) is excluded. Copies are idempotent — only files newer than the destination are overwritten

### Sync Manager v1.2.58
- Changed: About tab now shows the Sync Engine changelog by default instead of the Sync Manager changelog

---

## 1.2.57 — 2026-04-23

### Sync Engine v1.4.11
- Fixed: On GeneXus X, XEv1, XEv2 and XEv3, the sync JSON files (changes, status, exports) no longer grow exponentially on each write — previously, every read/modify/write cycle doubled the number of entries because the files were serialized with two arrays per list (one for the underlying field, one for the public property), and the deserializer appended both to the same list. A few write cycles sufficed to exhaust process memory
- Fixed: Export notifications registry no longer causes `OutOfMemoryException` when the file grows to excessive sizes — if the file exceeds 10 MB or cannot be parsed, it is automatically quarantined as `.corrupted-<timestamp>.bak` and a fresh one is created. The periodic cleanup on startup and every register operation now cap the in-memory entry list to prevent unbounded growth between cleanups
- Fixed: A failure while writing the export notifications registry no longer terminates the sync process — the error is logged as a warning and the sync continues
- Fixed: On GeneXus X, XEv1, XEv2 and XEv3, bulk export start/end markers in the export notifications registry are now written with the correct `changeType` value — previously they were always written as `"modified"`, making it impossible for consumers of the registry to distinguish bulk export boundaries from per-object change notifications
- Fixed: On GeneXus X, XEv1, XEv2 and XEv3, import status messages are now written with the correct `type` value (`error`, `warning`, `info`) — previously all message types were silently overwritten as `"error"`

---

## 1.2.56 — 2026-04-18

### Sync Engine

#### v1.4.10 — 2026-04-18
- Fixed: SDT root level `Collection` / `CollectionItemName` properties are now applied during internalization — when the SDT itself is a collection these were ignored, because the property block of the root structure node was never read. Also reconciles sublevel `Collection` / `CollectionItemName` on every re-import, not just on creation

#### v1.4.9 — 2026-04-17
- Fixed: SDT sublevel internalization now correctly applies the `Collection` and `CollectionItemName` properties when the sublevel is created for the first time — previously, sublevels marked as `Collection = True` in the `.gxSource` were created without the collection flag

---

## 1.2.54 — 2026-04-16

### Sync Engine

#### v1.4.8 — 2026-04-16
- Added: KBbridge Sync Engine now generates a `kb-config.json` file in the workspace root containing the GeneXus version of the KB — this file is consumed by VSCode extensions to filter properties and keywords by version
- Fixed: Source file parser no longer crashes with "Index was outside the bounds of the array" when parsing structures with certain blank line patterns — a missing parenthesis in 6 blank-line-skip loops caused array access beyond bounds
- Fixed: Property internalization now correctly resolves ambiguous property names by matching against the object's actual properties — previously, `ExposeAsWebService` on a DataProvider was incorrectly set as the Transaction variant (`idISBCWEBSERVICE` instead of `idISWEBSERVICE`), causing the property to be saved with the wrong value

#### v1.4.7 — 2026-04-15
- Fixed: Module path deduction from qualified names no longer truncates module names that contain the object name as a substring — previously, objects like `Factura` inside module `eFactura` would cause the module to appear as `@e`, and `SendMail` inside `PXTools.SendMails` would produce a spurious `@PXToolss` module

#### v1.4.6 — 2026-04-13
- Fixed: Domain paths in the exports registry file now use the correct single separator — previously, Domains inside modules were written with a double slash (e.g. `@Module//#Domains/`) causing the sync engine to treat recently exported objects as external changes requiring internalization

---

## 1.2.53 — 2026-04-07

### Sync Engine

#### v1.4.5 — 2026-04-07
- Fixed: Variable internalization now correctly creates and types variables that did not previously exist in the Knowledge Base — domain-based, attribute-based, SDT, ExternalObject, BusinessComponent and simple-typed variables are all properly applied
- Fixed: Business Component variables are now recognized on import via the `BusinessComponent:` prefix, restoring the round-trip with the export change introduced in v1.4.4
- Fixed: Bulk export no longer aborts when the Knowledge Base does not register one of the queried object types — unknown types are now silently skipped instead of throwing
- Fixed: Variable serialization no longer fails when the variable does not expose the `idVarServiceExtName` (Web Service External Name) property
- Fixed: Object export now reports a `Fail` line with the underlying error reason whenever an unhandled exception is raised by the GeneXus SDK during serialization

#### v1.4.4 — 2026-04-06
- Fixed: Business Component variables (based on a Transaction or Transaction sublevel) are now exported with a `BusinessComponent:` prefix followed by the Transaction name (and sublevel using dot notation) — previously they were serialized as the invalid raw type `#GX_BUSCOMP` / `#GX_BUSCOMP_LEVEL`

### Sync Manager

#### v1.2.53 — 2026-04-07
- Fixed: Suppressed StringTemplate noise (LayoutToHtml.stg) from console output when using GeneXus X Evo3

#### v1.2.52 — 2026-04-06
- No additional changes (Engine binaries update only)

---

## 1.2.51 — 2026-04-04

### Sync Engine v1.4.3
- Added: Variable, Attribute and Domain properties beyond DataType are now serialized and deserialized — validation, picture, control info, appearance, behavior, virtual keyboard, and offline properties are preserved in the roundtrip
- Fixed: ExternalName, ExternalNamespace and AllowNull properties excluded from Variable/Attribute/Domain serialization — these properties are not applicable in that context and share names with other object types
- Fixed: Variable ExternalName (idVarServiceExtName) now serializes independently of DataType — previously was only exported when the variable had a non-default type
- Fixed: Removed debug output during External Object export that printed property DataType details to console

### Sync Manager v1.2.51
- Added: Language selector in Trial Registration, Enter License Key, and Buy License forms (defaults to OS language)
- Added: First-run experience — opens Trial Registration automatically with informational message
- Fixed: Settings window no longer blocks license warning dialogs
- Changed: Settings opens on License tab when no valid license is configured
- Fixed: Tray menu now updates immediately when adding or removing Knowledge Bases (no restart needed)
- Changed: GeneXus version dropdown now shows newest versions first
- Changed: Buy License now opens the purchase page in the browser with pre-filled user data
- Added: "Check License" button appears after Buy License or paid activation to manually retrieve license updates from the server

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
