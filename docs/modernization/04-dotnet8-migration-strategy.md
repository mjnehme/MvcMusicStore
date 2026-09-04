
# 04 — .NET 8 Migration Strategy

Answers one of the user's fourteen requirements: **".NET 8 migration strategy"** (produce). It defines the
project-format, target-framework and dependency transition for the chosen migration source — MVC 5 — and
nothing else. The pipeline, dependency-injection, Identity and view work is
[05 — ASP.NET Core Migration Approach](05-aspnet-core-migration-approach.md); the hosting decision is
[06 — Azure Hosting Recommendations](06-azure-hosting-recommendations.md).

## 1. Purpose, ownership and authoring contract

### 1.1 What this document is

A **specification an engineer can execute after approval**. Every artifact it names — the SDK-style
project file, `global.json`, `NuGet.config`, `.config/dotnet-tools.json`, `libman.json`, the per-project
`packages.lock.json`, the single root solution — is described here in enough detail to be created without
a second design conversation, including its exact contents where the contents are the decision.

It is written against **MVC 5** as the sole migration source. That triage is not re-argued here: MVC 5 is
the only edition whose target framework is `v4.8` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:16], against
`v4.5` for MVC 4 [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:16] and `v4.0` for MVC 3
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:15], and
[10 — Build and Deployment Requirements](10-build-and-deployment-requirements.md) §4.1 owns that
comparison and records `v4.8` as the repository maximum.

### 1.2 What this document owns, and what it must not restate

Two values are **owned here and nowhere else**. Every other deliverable cites this document for them
rather than repeating them, because a value stated twice in different words reads downstream as two
decisions:

| Owned here | Value |
| --- | --- |
| **Target framework** | `net8.0` (section 2) |
| **SDK feature band and roll-forward policy** | `8.0.400`, `rollForward: latestPatch` (section 3) |

Also owned here: the project-format conversion, the per-package migration outcome for all 28 MVC 5 pins,
the target-state `NuGet.config` and lockfile policy, the client-library acquisition mechanism including
the `libman.json` provider ids, destinations and file lists (§9.4), the tooling-manifest contents, and
**the project graph** — which project references which, and the entry-point visibility the in-process test
fixture depends on (§12.4).

And because the operator console's host is composed here, so is **that host's input surface**: its admitted
configuration sources, the closed dispatcher that recognizes all ten verbs and parses the two whose switches
this host binds, and the order in which the administrator credential is acquired (§12.4). Two things about
that surface are pointedly *not* owned here and are cited instead: the **exit-code allocation** is
[05 §10.2](05-aspnet-core-migration-approach.md)'s five codes tool-wide, and this document states only which
of them each condition it specifies produces; and the credential's **variable name** is that document's too,
restated here only because this host has to read it. What the console *does* with the inputs remains
[05 §10.2](05-aspnet-core-migration-approach.md)'s, and the eight data verbs' switch grammar is
[05 §5.7](05-aspnet-core-migration-approach.md)'s.

### 1.3 What this document does not own

| Not stated here | Owner | How this document treats it |
| --- | --- | --- |
| Hosting target, deployment model, observability approach, data-protection key-ring location | [06](06-azure-hosting-recommendations.md) | Cited. This document names the *package* that persists a key ring (§10.3); it does not choose where the ring lives |
| Cutover approach and its accepted losses; pipeline, DI, configuration, Identity, EF Core, view and static-asset transitions | [05](05-aspnet-core-migration-approach.md) | Cited. The conditional packages in §7.3 exist *because* an alternative cutover is retained; the choice is not made here |
| Per-edition build outcomes and toolchain prerequisites | [10](10-build-and-deployment-requirements.md) | Cited, never re-diagnosed |
| Effort, bands, confidence, and the risk register — including the .NET 8 support-window entry | [07](07-effort-risks-sequencing.md) | Pointed at. **This document states no implementation-effort figure, no duration and no delivery schedule.** It does state counts, versions, line locators and file totals throughout — those are evidence and specification rather than effort — so the claim is about effort and time, not about numbers |
| Workstream decomposition, sequencing, and CI provider selection | [03](03-modernization-roadmap.md) | Cited |
| Current pin values, the restore-source finding, the committed restore client | [02](02-dependency-inventory.md) | Cited. §8 states each pin's *outcome*, not a re-transcription of the manifests |
| Every construct with no successor, and every successor whose default differs | [12](12-migration-blockers.md) | Cited by finding identifier |
| Debt framing, severity and ownership | [08](08-technical-debt-register.md) | Cited |

### 1.4 The no-modification constraint, and the boundary that makes this document possible

The user directed **"Do not make code changes initially"**; the project's environment setup instructions
independently restate it as "Do not modify code until assessment and modernization plan are approved."
**No pre-existing repository file was modified or deleted to produce this document, and nothing was added
outside the thirteen deliverables under `docs/modernization/`** — which are what this work produces, and
are additions rather than modifications. Every source artifact named below was read; none was written.

**The check a reader runs, in the durable form.** Two commands, because they answer two different
questions and only one of them survives the commit:

| Check | Expected result | What it establishes |
| --- | --- | --- |
| `git status --porcelain` | **empty — no output at all** | **Current-checkout evidence**, and labelled as such: nothing is uncommitted in the checkout this was authored in — no stray build output, no restored `packages/` payload, no half-finished edit. It says nothing about any other checkout, which is why it is not the durable half |
| `git diff --name-status ea2552d..HEAD` | **13 lines, every one `A` and every path under `docs/modernization/`** — no `M` and no `D` against any pre-existing file | **An observation about the checkout the reader is standing in**, and labelled as such. `ea2552d` is the commit this deliverable set was authored from; `HEAD` is wherever that checkout points. It establishes that between the authoring baseline and the state under review, the whole of this work is thirteen added documents |

The second is the one that survives the commit. **A `git status --porcelain` listing of new files describes
an uncommitted moment and cannot be reproduced once the work is committed** — at that point the same command
correctly prints nothing, and a document quoting the earlier listing as its current evidence is quoting
something a reader cannot re-run. The diff re-asks the question against whatever the checkout now holds,
which is exactly what a reviewer wants and is why the second endpoint is written `HEAD`.

**Neither command is immutable provenance, and this document does not pretend to carry any.** A pinned
second endpoint would read as stronger evidence and would in fact be weaker: **no document can contain the
hash of the commit that adds it** — the hash is not knowable until after the content is fixed — so a
specific commit identifier written here could only be one nobody can verify, and there is no correct value
one could hold, so **no pinned second endpoint is named anywhere in this document**. **Immutable
provenance lives outside the
documents**: in the commit that carries this set, in its message, and in the review record that references
it — all three addressable by a reader who has the repository, and none of them restatable from inside a
file the commit contains. What this document owes under the reproducing-command contract of §1.5 is a
command a reader can run and a truthful label on what its result means, and the row above is both.
Where this document mentions the pre-commit listing at all, it is labelled **authoring-time** evidence
rather than presented as current.

The distinction that makes a *strategy* possible under that constraint is **mutation versus
specification**:

- **Mutation** — changing a tracked file — is prohibited absolutely, and did not happen. There is no
  `global.json` in this repository, no `NuGet.config` at the root, no SDK-style project and no lockfile;
  this document did not create them.
- **Specification** — describing, in full and executable detail, the changes a later approved phase will
  make — is exactly what was commissioned. Declining to specify would be its own failure, not caution.

So when section 6 states that `global.json` pins the band at `8.0.400`, that is **content written into
this document**. The file does not exist and must not be created before approval. Nothing here is
authorization to begin; it is the instruction set for when authorization is given.

### 1.5 Authoring contract, and the absence of user rules

**`review_rules` returns exactly "No user rules provided."** Verified directly while writing this
document. There is consequently no project rule to name, summarize, cite or comply with, and no file
forced into scope by one. That absence is not licence to lower the bar, so this document is held to four
explicit contracts instead:

1. **Citation.** Every **as-is** claim carries an inline `[<path>:<locator>]` citation at the point of
   claim, with a repository-relative path that resolves in the checkout. Target-state statements are
   prescriptions and carry no repository citation — but the *reason* for a prescription is almost always
   an as-is fact, and that fact is cited. **The full path is repeated at every claim: there is no
   shorthand form, and no citation inherits its path from a neighbouring row, sentence or paragraph.**
   Every time-sensitive external claim — a package version, a deprecation, a tool invocation, a platform
   capability — carries a dated inline reference in the form
   `[<Publisher>, *<page title>*, <url> — verified <date>]` at the same point of claim.
2. **Exact versions only.** Every version in this document is a single exact release. No range, no
   floating notation, no "or later", no placeholder. Where a version must be confirmed again before it is
   applied, that is stated as a re-verification step (§7.2), not softened into a range.
3. **Repository-wide claims carry their reproducing command**, in the appendix, so a reader can refute
   them.
4. **One fact, one owner** — the ownership tables in §1.2 and §1.3.

Markdown line length follows the corpus convention recorded in [`README.md` § 10 Markdown line-length convention](README.md#10-markdown-line-length-convention).

---

## 2. The target framework

### 2.1 The decision

**The target framework is `net8.0`.**

That is the whole decision, and it is stated once. It is what the user asked for, it is the framework
every version in section 7 belongs to, and it is the value deliverables 03, 05, 06 and 07 cite rather
than restate. All **four** target projects — the ported application, the **two** test projects and the
single operator console (§12.2, §12.9) — target it; there is no
multi-targeting and no `netstandard` shim anywhere in the target, because there is no shared library and
no consumer outside this repository — the migration source declares no `<ProjectReference>` at all and is
a leaf project [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:1-302].

### 2.2 The support window is an approval decision, and it belongs to 07

.NET 8's support window is a real consideration and it is deliberately **not** argued here. It is the
first entry in [07](07-effort-risks-sequencing.md)'s risk register, where it is framed as an approval
decision with a named owner rather than as a technical alternative inside a strategy. The date, the
likelihood, the impact and the contingency live there. Suppressing it would be a disservice; re-arguing
it here would put the same decision in two documents, which §1.2 exists to prevent.

What *is* this document's business is the strategy-relevant consequence, because the approver needs to
know what they are choosing between.

### 2.3 Moving to a later LTS release after approval is not a one-line edit

If the approver directs a later LTS target, the change is not `net8.0` → a different moniker in one file.
Five things move with it, and the last is the expensive one:

1. **Every Microsoft-shipped package pin moves to that release's servicing line.** The EF Core, ASP.NET
   Core Identity, distributed-cache, data-protection and test-host pins in §7.2 are a single coherent
   servicing band, not independently chosen numbers; the whole set moves together or the graph is mixed.
2. **The SDK feature band in `global.json` and the build image change together** (section 3). A build
   image that satisfies one band does not satisfy another, so the change is a pipeline change as well as
   a repository change.
3. **The `Microsoft.AspNetCore.SystemWebAdapters` compatibility surface changes with the release.** Those
   packages target specific .NET versions on the Core side and a .NET Framework floor on the legacy side
   (§7.4), so if the incremental path in [05](05-aspnet-core-migration-approach.md) is ever selected, its
   package feasibility must be re-established against the new target rather than assumed to carry over.
4. **The test-tooling pins are re-verified**, since the test SDK and adapter are versioned independently
   of the runtime — and one of them carries a registry deprecation whose approval gate is §7.3.
5. **Behaviour validation is re-run in full.** A framework change re-opens exactly the class of
   difference [12](12-migration-blockers.md) §4 catalogues — successors whose defaults differ and whose
   failures are silent. The validation is not a formality on a runtime change.

None of that makes a later target wrong. It makes it a **decision with a cost**, which is why it is
recorded as an approval item in 07 and not quietly absorbed here.

---

## 3. The SDK band

### 3.1 The pin

The target repository commits a **`global.json`** at its root pinning the **SDK feature band `8.0.400`**
with **`rollForward: latestPatch`**:

```json
{
  "sdk": {
    "version": "8.0.400",
    "rollForward": "latestPatch"
  }
}
```

`latestPatch` permits the SDK to roll forward **within** the `8.0.4xx` band — to a servicing patch of it —
and no further. A host carrying only a lower band, or only a higher one, fails the build with a clear SDK
resolution error rather than building with something else.

### 3.2 Why this is a genuine pin and `latestFeature` is not

`rollForward: latestFeature` would accept **whatever 8.0 feature band the host happens to carry**. That
is the failure mode worth naming precisely: the build would still succeed on every machine, and it would
succeed *differently* — the same commit compiled by a different SDK band, with that band's analyzers,
defaults and targets. Nothing would announce it. Reproducibility would then be a property of the build
agent's install history rather than of the repository, which is the same class of defect this repository
already exhibits on the restore side, where no package source is configured anywhere and restore inherits
whatever the host provides ([02](02-dependency-inventory.md) §6).

Fixing one and not the other would be inconsistent. `latestPatch` is therefore the policy, and **the
build image must satisfy the `8.0.400` band** — a requirement on the pipeline that
[03](03-modernization-roadmap.md) carries into its CI workstream, whose provider is selected there and
not here.

### 3.3 What the pin does not do

It pins the **toolchain**, not the dependencies. The SDK band decides which compiler, analyzers and
targets build the code; it decides nothing about which package versions resolve. Those are pinned
separately, by exact `PackageReference` versions (§7) and by the lockfile policy (§6.4). Both are needed,
and conflating them is how a repository ends up with one of the two.

---

## 4. What the current project is

Everything in this section is read out of the tracked file, because the conversion in section 5 is
defined as a transformation of exactly these properties.

`src/MVC5/MvcMusicStore/MvcMusicStore.csproj` is a **302-line, UTF-8-with-BOM, non-SDK MSBuild
2003-format** project file — `ToolsVersion="12.0"` on the
`http://schemas.microsoft.com/developer/msbuild/2003` namespace
[src/MVC5/MvcMusicStore/MvcMusicStore.csproj:2].

| Property of the current project | Value | Locator |
| --- | --- | --- |
| Project format | MSBuild 2003, `ToolsVersion="12.0"` | [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:2] |
| `ProjectGuid` | `{25CE8290-EF24-4818-B009-68DC903163D3}` | [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:10] |
| `ProjectTypeGuids` | `{349c5851-65df-11da-9384-00065b846f21}` (ASP.NET web application) then `{fae04ec0-301f-11d3-bf4b-00c04f79efbc}` (C#) | [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:11] |
| `OutputType` | `Library` | [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:12] |
| `RootNamespace` / `AssemblyName` | `MvcMusicStore` / `MvcMusicStore` | [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:14-15] |
| `TargetFrameworkVersion` | `v4.8` | [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:16] |
| `MvcBuildViews` | `false` | [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:17] |
| Assembly references | **46** `<Reference>` elements, of which **26** carry a `<HintPath>` into `..\packages\...` and 20 resolve from the framework or the machine | [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:47-158] |
| `<ProjectReference>` | **none** — a leaf project | [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:1-302] |
| Compile inventory | **27** explicit `<Compile Include=...>` items, one per source file | [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:161-189] |
| Content inventory | **61** `<Content Include=...>` items and **2** `<None Include=...>` items, naming every view, script, stylesheet, image, font and config individually | [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:192-266] |
| Web host settings | `UseIISExpress` `true` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:18] with an empty `IISExpressSSLPort` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:19]; a `ProjectExtensions` block carrying `DevelopmentServerPort` `43524` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:283] and `IISUrl` `http://localhost:43524/` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:285] | [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:18-19], [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:277-294] |
| Restore opt-in | `RestorePackages` `true` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:24] with `SolutionDir` defaulted to `..\` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:23] | [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:23-24] |

Four MSBuild imports and one target complete the picture, and they do not share one fate in section 5:

| Import or target | Condition | Locator |
| --- | --- | --- |
| `$(MSBuildBinPath)\Microsoft.CSharp.targets` | unconditional | [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:271] |
| `$(VSToolsPath)\WebApplications\Microsoft.WebApplication.targets` | `'$(VSToolsPath)' != ''`, where `VSToolsPath` derives from a `VisualStudioVersion` the project defaults to `10.0` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:268-269] | [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:272] |
| `$(MSBuildExtensionsPath32)\Microsoft\VisualStudio\v10.0\WebApplications\Microsoft.WebApplication.targets` | `false` — permanently inert | [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:273] |
| `MvcBuildViews` target invoking `<AspNetCompiler>` | `'$(MvcBuildViews)'=='true'` | [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:274-276] |
| `$(SolutionDir)\.nuget\NuGet.targets` | `Exists(...)` — conditional, and the condition is not met | [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:295] |

Dependencies are declared in a sibling `packages.config` carrying **28 pins**, every one of which declares
`targetFramework="net45"` while the project targets `v4.8`
[src/MVC5/MvcMusicStore/packages.config:3-30]. The pin values and that platform disagreement are owned by
[02](02-dependency-inventory.md) §3.1; their migration outcomes are section 8 of this document.

---

## 5. The project-format transition

### 5.1 SDK-style, `net8.0`, implicit globbing

The target project file is **SDK-style**, opening `<Project Sdk="Microsoft.NET.Sdk.Web">`, with a single
`<TargetFramework>net8.0</TargetFramework>`. Three consequences follow immediately, and each removes a
whole class of the current file's content:

- **The 27-item `Compile` inventory [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:161-189] disappears
  entirely.** The Web SDK globs `**/*.cs` by default. A source file added after the conversion compiles
  because it exists, not because someone remembered to list it.
- **The 61 `Content` and 2 `None` items [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:192-266]
  disappear.** Razor views are handled by the SDK, and static assets are served from `wwwroot` by
  static-file middleware, so they need no per-file item at all. This is also why the target has no
  equivalent of the `<Folder Include="App_Data\" />` item
  [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:259] — the target has no `App_Data`.
- **`OutputType` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:12], `RootNamespace` and `AssemblyName`
  [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:14-15] are defaulted rather than declared.** A web project
  is a library-shaped assembly with an entry point by default, and both names default from the project
  file name, which is unchanged.

`ProjectGuid` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:10] and `ProjectTypeGuids`
[src/MVC5/MvcMusicStore/MvcMusicStore.csproj:11] are **dropped, not translated.** The SDK-style format
identifies a project by its path; the modern solution format does not require a project GUID declared in
the project file, and no project-type GUID selects behaviour any more — the `Sdk` attribute does.

**One property is added for a privacy reason rather than a build one: `Release` produces no debug
symbols.** `<DebugType>none</DebugType>`, conditioned on `'$(Configuration)' == 'Release'`, is declared in
the web project **and** in `tools/provision-admin` (§12.2) — the two projects §13.4 publishes. Under it a
Release compile emits no `*.pdb` at all, so neither publish root can contain one and no symbol file can
reach a deployed artifact. What survives is the frame's **method name**, which is assembly metadata rather
than symbol-file content, so a stack trace still names the methods it walked; what disappears is the
**source-file path and the line number**, which a .NET stack frame resolves only when a matching symbol
file sits beside the assembly at run time.

Three things about it are worth stating, because each is a way the property is got wrong:

- **Both published projects carry it, not just the web application.** The console references the web
  project (§12.9), so publishing the console copies the referenced assembly and would copy its symbol file
  with it. A property set on one project only would leave one publish root symbol-free and the other not,
  and an assertion over the first would pass while the second shipped symbols.
- **`Debug` is untouched.** The condition is on the configuration, so local debugging keeps its symbols and
  loses nothing. This is a property of the artifact the release promotes, not of the developer's build.
- **It is asserted rather than assumed.** §13.4's sequence fails the build when a `*.pdb` is found in
  either publish root, which is what makes the property a fact per release instead of a setting nobody
  re-checks. Nothing about publish *mode* is involved: the framework-dependent bundle pin and the console's
  own publish are unaffected by it, because symbols and self-containment are independent switches.

The consumer is [06](06-azure-hosting-recommendations.md) §9.2, whose sanitized exception record promises
a stack trace carrying no source-file path and no line number and requires that promise to rest on the
artifact rather than on the record's construction. That document requires the property; this one sets it,
and §13.4 proves it.

### 5.2 `PackageReference` replaces `packages.config`

`packages.config` is deleted and the surviving dependencies become `<PackageReference Include="..."
Version="..." />` items in the project file, at the exact versions in §7.2. That change carries four
properties worth stating, because each one repairs a specific current behaviour:

1. **Hint paths cease to exist.** All 26 `<HintPath>` entries, carried by the reference elements at
   [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:64-158], are removed. Package assets are resolved from the
   global packages folder by the restore graph, so the class of failure where a hint path points somewhere
   the payload is not — which is one of MVC 4's two committed defects,
   [10](10-build-and-deployment-requirements.md) §6.2 — becomes structurally impossible.
2. **Transitive dependencies are resolved, not enumerated.** Several of the current 46 references exist
   only because `packages.config` requires every transitive assembly to be referenced explicitly.
3. **The 20 references that carry no hint path are removed** — 17 of them contiguous at
   [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:47-63], the other three at
   [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:68], [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:70] and
   [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:103]. `System`, `System.Web`, `System.Xml`,
   `System.Configuration`, `System.Net.Http` and the rest are either part of the shared framework or gone;
   a `net8.0` project references no framework assembly by name.
4. **No `bin`-copy semantics to configure.** The `<Private>True</Private>` elements accompanying the
   package references — for example at [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:65],
   [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:73] and
   [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:77] — have no analogue and are dropped.

### 5.3 `Properties/AssemblyInfo.cs` is absorbed into MSBuild properties

The file carries **12 assembly-level attributes** and is deleted; the metadata survives as build
properties, which the SDK uses to generate the attributes. Keeping both would produce duplicate-attribute
compile errors — the failure mode [12](12-migration-blockers.md) F-12-12 records. The mapping is
one-for-one and complete:

| Current attribute | Locator | Target MSBuild property |
| --- | --- | --- |
| `AssemblyTitle("MvcMusicStore")` | [src/MVC5/MvcMusicStore/Properties/AssemblyInfo.cs:8] | `<AssemblyTitle>` |
| `AssemblyDescription("")` | [src/MVC5/MvcMusicStore/Properties/AssemblyInfo.cs:9] | `<Description>` |
| `AssemblyConfiguration("")` | [src/MVC5/MvcMusicStore/Properties/AssemblyInfo.cs:10] | `<Configuration>` — generated per build configuration; not declared |
| `AssemblyCompany("")` | [src/MVC5/MvcMusicStore/Properties/AssemblyInfo.cs:11] | `<Company>` |
| `AssemblyProduct("MvcMusicStore")` | [src/MVC5/MvcMusicStore/Properties/AssemblyInfo.cs:12] | `<Product>` |
| `AssemblyCopyright("Copyright ©  2013")` | [src/MVC5/MvcMusicStore/Properties/AssemblyInfo.cs:13] | `<Copyright>` — the value is refreshed at conversion; carrying a 2013 copyright forward is not a migration requirement |
| `AssemblyTrademark("")` | [src/MVC5/MvcMusicStore/Properties/AssemblyInfo.cs:14] | `<Trademark>` |
| `AssemblyCulture("")` | [src/MVC5/MvcMusicStore/Properties/AssemblyInfo.cs:15] | `<NeutralLanguage>` — omitted, since the current value is empty |
| `ComVisible(false)` | [src/MVC5/MvcMusicStore/Properties/AssemblyInfo.cs:20] | Not carried forward. COM visibility is off by default and this project exposes nothing to COM |
| `Guid("64547e1b-3030-4458-ab71-a970f2916ed6")` | [src/MVC5/MvcMusicStore/Properties/AssemblyInfo.cs:23] | Not carried forward. The type-library GUID is meaningful only under COM registration |
| `AssemblyVersion("1.0.0.0")` | [src/MVC5/MvcMusicStore/Properties/AssemblyInfo.cs:34] | `<AssemblyVersion>` |
| `AssemblyFileVersion("1.0.0.0")` | [src/MVC5/MvcMusicStore/Properties/AssemblyInfo.cs:35] | `<FileVersion>` |

Ten of the twelve become properties; two — `ComVisible` and `Guid` — are deliberately **not** carried
forward, because they configure a COM surface the application does not have. Recording that as a decision
rather than an omission is the point: a reader comparing the two files should find no unexplained
difference.

### 5.4 What is dropped, and the one consequence that must be stated honestly

| Dropped | Locator | Why, and what replaces it |
| --- | --- | --- |
| `MvcBuildViews` property and its `AspNetCompiler` target | [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:17], [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:274-276] | The property is `false` and the target is gated on `'true'`, so the target never runs. Razor compilation in the target is an SDK concern, configured by the SDK's own build-time and publish-time compilation rather than by an `AspNetCompiler` invocation |
| `$(VSToolsPath)\WebApplications\Microsoft.WebApplication.targets` import | [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:272] | The Web SDK subsumes it. This import is also the reason the current project needs Visual Studio's web-application targets on the build host — a prerequisite [10](10-build-and-deployment-requirements.md) §4.2 owns |
| The `Condition="false"` v10.0 import | [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:273] | Already inert; removed as dead configuration |
| `$(SolutionDir)\.nuget\NuGet.targets` import, `RestorePackages`, `SolutionDir` default | [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:295], [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:24], [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:23] | Superseded by SDK-integrated restore (§5.5) |
| IIS Express settings and the `ProjectExtensions` web-project block | [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:18-22], [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:277-294] | **Dropped with no target *project* setting replacing them.** The target runs on Kestrel: its deployed binding is [06](06-azure-hosting-recommendations.md)'s, and its **local** endpoints are declared in `appsettings.Development.json` under `Kestrel:Endpoints` — an exact HTTP and an exact HTTPS URL, because the no-configuration default is HTTP-only. Section 12.5 of this document specifies them; section 12.4 records that no launch-profile file is specified |

**The honest consequence of dropping `MvcBuildViews`: nothing is lost, because nothing was there.**
Because the property is `false` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:17] and its target is gated on `'true'` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:274], `AspNetCompiler` has
never run, so MVC 5's **29 Razor views have never been compile-checked** by this build. Deliverable
[10](10-build-and-deployment-requirements.md) §12.3 owns that point: a build reporting zero warnings says
nothing whatever about the views, because no view is compiled in the first place.

**A second fact compounds it, and neither this document nor any other may soften it into the first.** No
recorded build has compile-checked those views *because no restored-source compile of MVC 5 has been
performed at all*. MVC 5's build status is **blocked, pending a Windows verification run**, and that run
is still outstanding — [10](10-build-and-deployment-requirements.md) §5.4 owns its required fields and
states that no deliverable may report the edition as building, cleanly or otherwise, until it happens.
The two facts are independent, and the second does not dissolve when the first is settled: even a green
verification run would leave the 29 views unexercised, because `AspNetCompiler` is gated off in the
configuration that run would build. [08](08-technical-debt-register.md) F-08-16 owns the debt severity.

The strategic consequence is specific and it constrains this document: **the port cannot rely on a prior
compile-check of the views as a baseline.** There is no "it compiled before" signal for view code, so
every view error the port introduces or exposes is indistinguishable from one that was always latent. The
view work itself is [05](05-aspnet-core-migration-approach.md)'s, and the target's SDK *does* compile
Razor as part of build and publish — which means the conversion turns 29 previously unchecked files into
checked ones, and pre-existing latent errors will surface as build errors at that moment. That is a
desirable outcome and a predictable source of first-build noise, and it is better predicted here than
discovered.

### 5.5 The `.nuget/` restore folder disappears

The target uses **SDK-integrated restore** — `dotnet restore`, and the implicit restore that `dotnet
build` performs — so no restore folder, no committed client and no MSBuild restore target exists in it.

Two current-state facts make that a repair rather than a preference:

- **MVC 5's solution declares a `.nuget` folder it does not have.** The solution file carries a
  solution-folder entry listing `.nuget\NuGet.Config` and `.nuget\NuGet.targets` as solution items
  [src/MVC5/MvcMusicStore.sln:8-13], and the project imports `$(SolutionDir)\.nuget\NuGet.targets` guarded
  by `Exists(...)` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:295]. **No file under any `.nuget` path is
  tracked anywhere in MVC 5** — the guard is what keeps this from being a build failure, and
  [10](10-build-and-deployment-requirements.md) §5.2 owns the verification.
- **MVC 4 commits the client itself.** `NuGet.exe`, `NuGet.Config` and `NuGet.targets` are tracked under
  `src/MVC4/MvcMusicStore/.nuget/`, and [02](02-dependency-inventory.md) §5.1 records the client's exact
  version as a pinned build dependency in its own right. It goes away with the folder; nothing in the
  target invokes a checked-in restore client.

### 5.6 One root solution is added; the four legacy solutions are retained

The repository tracks **four `.sln` files for three legacy projects**, one per edition plus a second under
`src/MVC4/`. **The target adds one root solution while the four legacy solutions are retained.** The added
file is a **single root `MvcMusicStore.sln`** referencing four new projects — the ported web application,
the **two** test projects and the single operator console (§12). It is an addition, not a replacement: no existing `.sln` is
edited, superseded or deleted, and after the port the repository tracks five solution files rather than
one.

**Two of those four projects exist before the solution does, and the ordering is deliberate.** The
contracts test project of §12.2 **and the operator console** are both created, restored and — in the test
project's case — built and run in [03](03-modernization-roadmap.md)'s
W4, at its governance-bootstrap gate 4a ([03](03-modernization-roadmap.md)'s gate-4a carve-out, and §6.4
for the console's placement) — before the ported web application exists and therefore before
this solution has anything else to
reference. Until the solution exists they are restored, built and tested **by project path**, which needs no
solution at all; adding them to the solution is part of creating the solution, not a separate step. Nothing
in the sequence requires a solution file to be authored early, and nothing in W4 depends on one. **The
solution is W6's**, and it is the one governance artifact of this section that W4 does not touch: §6.1 and
§6.2's two manifests are created at 4a because a locked-mode restore is impossible without them, whereas a
restore by project path needs no solution at all.

That is not a stylistic point. The four legacy solutions are how the legacy editions are opened and built,
and MVC 5 in particular must remain buildable and runnable throughout the port because it is the reference
implementation the port is validated against (§12.3). A target root solution that replaced them would
remove that capability.

One of the four is stale, with a project path that does not resolve;
[10](10-build-and-deployment-requirements.md) §6.4 owns the diagnosis and
[08](08-technical-debt-register.md) F-08-23 owns the debt. It is named here because a reader could
otherwise assume all four are equally usable inputs to the port's validation, and one is not — its
staleness is recorded, not repaired, and it too is retained.

The three legacy projects and all four legacy solutions are **not deleted** — §12.3.

---

### 5.7 `UserSecretsId` — the project property the Development configuration source does not work without

[05](05-aspnet-core-migration-approach.md) §3.3's precedence list carries **user secrets as source 3**,
added only when the resolved environment is `Development`. That source has a prerequisite in *this*
document's half, because it is a **project-file property** rather than a runtime setting or a platform
setting — and the failure mode when it is missing is the reason this subsection exists rather than being a
line in §5.1: **nothing throws, nothing warns, and the source contributes nothing.** A developer then sees
a startup validation failure naming a key they can see in their own secret store, and the natural reading
of that — a wrong value, or a validator bug — is the wrong one.

**The mechanism, exactly.** Secret Manager keeps values **outside the repository**, in a per-developer
path, and that path is keyed by an identifier the project declares. Microsoft's guidance on safe storage of
app secrets in development gives the store as `~/.microsoft/usersecrets/<user_secrets_id>/secrets.json` on
Linux and macOS and `%APPDATA%\Microsoft\UserSecrets\<user_secrets_id>\secrets.json` on Windows, "where
`<user_secrets_id>` is the `UserSecretsId` value specified in your project file", and the framework's
builder "calls the `AddUserSecrets` method when the `EnvironmentName` property is `Development`". So two
conditions have to hold together: the environment must resolve to `Development`, **and** the project must
carry the identifier the store is filed under. §13.3 arranges for the first to be false in every migration
and release invocation; this subsection arranges for the second to be true on a developer machine, which is
the only place the source is ever meant to fire.

**What generates it, and what it is.** `dotnet user-secrets init`, run once per project, "adds a
`UserSecretsId` element within a `PropertyGroup` of the project file", whose inner text "is a GUID" by
default and "is arbitrary, but is unique to the project". Two properties follow, and both are stated
because the name invites the opposite assumption:

- **The identifier is committed, and it is not a secret.** It names *where* a store is, not what is in it —
  a developer-scoped lookup key, declared beside `TargetFramework` and reviewed like any other property. A
  reader who finds it in the project file has learned nothing they could use.
- **The values are neither committed nor protected.** They live in the developer's profile, "aren't checked
  into source control", and Secret Manager "doesn't encrypt the stored secrets", which is precisely why the
  source is Development-only and why nothing deployed depends on it. The deployed channel is source 4 of
  the same precedence list — platform settings and Key Vault references, [06](06-azure-hosting-recommendations.md)
  §8.4's — and no artifact in §13.4 carries either.

**Why the property alone is sufficient here.** The identifier can also arrive as an assembly attribute, and
Microsoft's guidance notes that the attribute has to be added by hand where `GenerateAssemblyInfo` is
`false`. §5.3 leaves SDK assembly-metadata generation at its default — that is the whole mechanism by which
the 12 absorbed attributes are produced — so the project property is enough, and a hand-written
`UserSecretsIdAttribute` would be a second declaration of the same thing.

**Which mapped projects carry one. This is a decision, and the third row is the one that would otherwise be
assumed wrongly:**

| Project | Identifier | Why |
| --- | --- | --- |
| `src/MvcMusicStore/MvcMusicStore.csproj` | **One, generated by `dotnet user-secrets init`** | It is the entry-point project, so its identifier is the one that resolves whenever the site's builder runs under `Development` — `dotnet run`, and an IDE start |
| `tools/provision-admin/ProvisionAdmin.csproj` | **The same identifier as the web project**, deliberately | The tool binds a subset of the same configuration contract against the same local database, so two stores would hold two copies of the same local values and drift apart on the day one is updated. Microsoft's guidance admits exactly this: secrets are "associated with a specific project or shared across several projects". Sharing has no deployed consequence, because the store never leaves the developer machine and neither project reads it in any other environment |
| `src/MvcMusicStore.Tests/MvcMusicStore.Tests.csproj` | **None, and one would do nothing** | Two independent reasons, and either alone settles it. The source is added by the **application's** builder, which resolves the **entry-point** project's identifier rather than the test project's. And the fixture sets its environment **explicitly per fixture, never inherited from the developer's machine** ([05](05-aspnet-core-migration-approach.md) §12.2), so the Development-only source is not registered when the suite runs — which is the property that keeps a developer's local values out of a test result |

**Which Development values are expected to be present, and which are legitimately absent.** The two
questions are separate, and answering them as one is what makes the mechanism look contradictory:

| In Development | Where it comes from | Which values |
| --- | --- | --- |
| **Present, committed** | `appsettings.Development.json` | The environment-shaped **non-secret** values, whose set [05](05-aspnet-core-migration-approach.md) §3.3 owns — with the two `Kestrel:Endpoints` URLs specifically §12.8's, because the map contains no launch-profile file to carry them |
| **Present only if a developer supplies one — and then through user secrets and nowhere else** | The store the identifier above enables | Any Development value that is a credential or a secret. Two can arise: a local connection string that carries a credential, where a developer's local SQL Server is not reached by a trusted local connection; and a local `Telemetry:CartKeyHashSalt`, if the developer wants the salted-hash path exercised locally rather than left unconfigured. Neither may be committed to the overlay, and neither reaches an artifact |
| **Legitimately absent, and validated as permitted rather than merely tolerated** | Nowhere — absence is the recorded answer | The four environment-conditional members of [05](05-aspnet-core-migration-approach.md) §3.3's omission list — `Expected:SqlServer`, `Expected:SqlDatabase`, `Expected:DataProtectionDiscriminator` and `Telemetry:CartKeyHashSalt` — each required in every environment that is not `Development` and not the test fixture. So a developer machine with an **empty** secret store still starts |

`Telemetry:CartKeyHashSalt` appears in the second row and the third on purpose, and that is the
reconciliation rather than a contradiction: the third row says it is **not required** locally, and the
second says that **if** a developer chooses to set one, user secrets is the only permitted channel. The
real contradiction — the one this subsection exists to remove — is a project with **no** `UserSecretsId`,
where the channel the contract names does not exist at all and every local secret has nowhere legitimate to
go.

---

## 6. The four net-new tooling manifests

None of these files exists in the repository, and **none is created by this assessment**. Each is
specified here in full so that the later phase creates it without re-deciding its contents.

**Each is created by a named workstream, and for the two root manifests it is not the workstream a reader
would expect.** The table states the allocation once so that no section below has to restate it, and so
that no two workstreams believe they own the same file:

| Manifest | First created in | Extended or amended later |
| --- | --- | --- |
| `global.json` (§6.1) | [03](03-modernization-roadmap.md)'s **W4, gate 4a** — the gate whose own exit requires a restore, and a restore against a pinned band needs the pin | **W6**, if the conversion requires an SDK property the bootstrap did not carry. Amended in place; never re-created |
| `NuGet.config` (§6.2) | [03](03-modernization-roadmap.md)'s **W4, gate 4a** — locked-mode restore against a *declared* source is a W4 exit criterion, so the declaration cannot arrive later | **W6**, if a source is added. Amended in place; never re-created, and the `<clear />` stays |
| `.config/dotnet-tools.json` (§6.3) | [03](03-modernization-roadmap.md)'s **W6** — nothing before the conversion invokes `dotnet ef` or `dotnet sql-cache` | **W10** consumes it; no later workstream adds a tool to it in this plan |
| `packages.lock.json`, per project (§6.4) | **The workstream that creates each project**: W4's gate 4a for the contracts test project **and the operator console** — the gate that produces the two root manifests the console's own first restore has to read — W6 for the converted web project, and W7 for the in-process test project. **Four projects, and there is no fifth**: every operator action is a verb of the one console (§12.9), so no separate data-migration tool has a lockfile here | Regenerated by any restore that legitimately changes the graph, and committed with that change — including each later gate that adds a verb to the console, and W7, where the console's project reference is declared (§6.4) |

**The rule behind the table, because it is the thing that was wrong before it was written down: a
workstream may not have an exit criterion that depends on a file a later workstream creates.** W4's exit
requires a locked-mode restore against a declared source, so the two root manifests and the contracts
project's lockfile belong to W4; W6 then **inherits and extends** the set rather than creating governance
from nothing. [03](03-modernization-roadmap.md)'s W4 and W6 state the same division from the sequencing
side, and its §6.1 carries the edge that connects them. **The same rule is what moves the operator
console's lockfile off W12**, and §6.4 states where it lands instead — gate 4a, the gate whose own output
the console's first restore consumes, which is why the placement needs no edge that the graph does not
already carry.

### 6.1 `global.json` — the SDK band

Root of the repository. Contents exactly as given in §3.1: `"version": "8.0.400"`,
`"rollForward": "latestPatch"`. Nothing else belongs in it — in particular, no MSBuild SDK entries, since
the project uses only `Microsoft.NET.Sdk.Web`.

**First created at [03](03-modernization-roadmap.md)'s W4 gate 4a**, because the band has to be pinned
before the first restore rather than after it: W4's own exit requires a restore **in locked mode**, and a
restore whose SDK is whatever the host happens to carry is not the reproducible restore that gate requires.
Which project that first restore covers is [03](03-modernization-roadmap.md)'s to state; that the band is
pinned before it runs is this document's. **W6 inherits this file and may amend it in place** — an added MSBuild SDK entry, say, if the
conversion turns out to need one — but must not re-create it and must not restore under a different band
than W4 did, because two workstreams restoring under two bands is the host-dependence this file exists to
end.

Absent today, verified repository-wide: `git ls-files | grep -c 'global.json'` returns `0` (appendix
A.4).

### 6.2 `NuGet.config` — which sources are consulted

Root of the repository. It **clears inherited sources and then declares one**:

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <clear />
    <add key="nuget.org" value="https://api.nuget.org/v3/index.json" protocolVersion="3" />
  </packageSources>
</configuration>
```

The `<clear />` is the operative element, not decoration. Without it, the declared source is *added to*
whatever machine-level and user-level configuration the build host carries; with it, the effective source
set is a property of the repository.

**First created at [03](03-modernization-roadmap.md)'s W4 gate 4a**, and the reason is that workstream's
own exit criterion rather than a preference: W4 must restore **in locked mode against a declared source**,
and a declared source that arrives two workstreams later is not one. **W6
inherits this file and extends it if the conversion needs a further source** — amended in place, with the
`<clear />` retained, never replaced by a second configuration file and never re-decided. A W6 that
authored its own source configuration would leave two files disagreeing about the effective source set,
which is the same class of ambiguity as the one below.

This ends an ambiguity that exists today: the repository configures no package source anywhere, so
restore inherits the host's configuration and the effective source set is not knowable from the checkout.
[02](02-dependency-inventory.md) §6 owns that finding and its correction to Technical Specification §3.3,
and §6.3 there names this document as the owner of the remedy. The finding is not restated here; the
remedy is.

The one tracked file named `NuGet.Config` today is MVC 4's, at
`src/MVC4/MvcMusicStore/.nuget/NuGet.Config`, and it contains a single `disableSourceControlIntegration`
setting with **no `packageSources` element at all**
[src/MVC4/MvcMusicStore/.nuget/NuGet.Config:1-6]. It is not a precursor of the target file and is not
migrated into it; it disappears with the folder (§5.5).

### 6.3 `.config/dotnet-tools.json` — the local tool manifest

A local tool manifest, so that every developer and every build agent runs the *same* tool versions,
restored by `dotnet tool restore` rather than installed globally:

| Tool package | Version | Why it is a separate pin |
| --- | --- | --- |
| `dotnet-ef` | `8.0.30` | Provides the `dotnet ef` command. **`Microsoft.EntityFrameworkCore.Design` does not provide it** — Design supplies the design-time services the command loads; the command itself is this tool |
| `dotnet-sql-cache` | `8.0.30` | Creates the SQL Server session cache table. **`Microsoft.Extensions.Caching.SqlServer` does not create it** — that package is the runtime provider that reads and writes the table |

Both distinctions are stated because both are load-bearing. A plan that lists only the runtime packages
produces a deployment with a migration command nobody can invoke and a session cache with no table behind
it. The table's schema and name, the principal that runs the command and the point in the release at
which it runs are [06](06-azure-hosting-recommendations.md)'s to specify; the *pin* is here.

Pinning the tool settles *which* command runs. It does not settle the **environment that command runs
in**, and for `dotnet ef` that is a correctness question rather than a hygiene one, because the command
builds the application's host and therefore reads its configuration. §13.3 states the design-time
invocation contract: `dotnet tool restore` first, **both** `ASPNETCORE_ENVIRONMENT` and
`DOTNET_ENVIRONMENT` set to the same value, that value being the invocation's own expected value — the
build's own design-time constant where an artifact is **generated**, and the deployment's own declared
expectation where one is **executed** — with the resolved environment validated, the
design-time connection supplied through the environment rather than the command line, and the per-context
invocations named in full.

### 6.4 Per-project `packages.lock.json` — what resolves from those sources

Each project in the target — the web application, the **two** test projects and the single operator console
(§12.2) — commits a
`packages.lock.json`, and **CI restores in locked mode**, so that a change to the resolved graph fails the
build instead of arriving silently. Four projects, four lockfiles.

**They do not all arrive at once, and no workstream may claim the full set before the projects exist.**
Each lockfile is created by the workstream that creates its project, which makes the allocation exactly
this:

| Lockfile | Created in | Because that is when the project exists |
| --- | --- | --- |
| `src/MvcMusicStore/packages.lock.json` | **W6** | The converted web project is W6's output |
| `src/MvcMusicStore.Tests/packages.lock.json` | **W7** | The in-process test project references the ported application, so it cannot exist earlier |
| `tools/provision-admin/packages.lock.json` | **W4, gate 4a** | The console's first restore has to read the two root manifests of §6.1 and §6.2 — the SDK band and the declared source — and gate 4a is the gate that produces them, so the restore and the manifests it needs belong to the same gate. [03](03-modernization-roadmap.md)'s gate-4a carve-out states it from the sequencing side: **that gate creates two projects, and the second is this console** |

W6's exit therefore accounts for the lockfiles of the projects it **inherits and creates** — gate 4a's
**two**, the contracts test project's and the console's, plus its own converted web project's — rather than
for the full set, and the **one** remaining arrives with the project that owns it, the in-process test
project at W7. [03](03-modernization-roadmap.md)'s W4 and W6 exit gates state the same allocation from the
sequencing side, W6's in its fourth exit criterion. The set is still stated **by ownership rather than
as a running count**, and the reason survives the placement: W3 and **W6** are unordered with respect to
each other ([03](03-modernization-roadmap.md) §4.2.1 declares no edge either way, and the `W4·4a → W3`
edge §6.4 consumes below does not create one — both descend from gate 4a), and W3 **re-locks** this
console when it authors its verb, so whether that regeneration has happened by W6's exit is not a fact
either gate can assert. What each gate can assert is which project's lockfile it owns and that its own
restore ran locked — which is why the criterion is written that way rather than as a claim about how many
lockfiles are current at that moment.

**The console project is created once and extended at every later gate that authors a verb of it, and
creating it is a different question from authoring its verbs — which is the distinction the placement turns
on.** [03](03-modernization-roadmap.md) §4.2.2 fixes the rule for the *verbs*, in its fourth graph
property: *each verb is authored by the workstream whose own gate runs it*, so no gate waits on a capability
a successor builds. That rule puts `extract-schema` in **W3**, the
generated-schema diff and the catalog load in **W9**, the Identity load in **W8**, and `admin` and `seed`
in **W12**. It says nothing about who creates the *project*, which it leaves to this document to place
explicitly and by name — and the answer is **W4's gate 4a**, on the dependency stated in the row above:
the project's first restore has to read `global.json` and `NuGet.config`, and 4a is the gate that produces
them, so restore and manifests belong to one gate rather than to two the graph does not order.
[03](03-modernization-roadmap.md)'s gate 4a carries the carve-out that says so — *that gate creates two
projects, and the second is the operator console* — and it **does require one edge, `W4·4a → W3`, which
that document declares**, for the reason the paragraph below states in full. So **gate 4a creates the
project, its `Program.cs` and dispatcher, and the first committed
lockfile**; each later gate that authors a verb **extends that same project and re-locks it**, committing
the regenerated lockfile with the change that caused it. One project, one lockfile, one publish output and
one promotion path throughout (§12.1, §12.9): what arrives gate by gate is *content*, never a second
project and never a second lockfile.

**Two earlier placements are superseded, and both are recorded rather than quietly replaced.** This row
first read **W12** — the gate that authors `admin` and `seed` — which is unexecutable: three earlier gates
would run verbs of a project that did not exist, precisely the shape §6's rule above exists to prevent. It
then read **W3**, the earliest gate that authors one of the console's verbs, which is executable but
conflated verb authorship with project creation and left a real gap: W3's first restore needs manifests
gate 4a produces, and [03](03-modernization-roadmap.md) §4.2.1 carried no edge between 4a and W3 at that point, so the
placement named an ordering the graph did not carry. **Gate 4a resolves the manifest half at the other
end** — the gate that owns the manifests owns the first restore that reads them — and it leaves the
*ordering* half open, which the paragraph below now states as a required edge rather than as a resolved
one. An intermediate form of that paragraph claimed no edge was required at all; that claim is withdrawn
below, with the reasoning that made it look sound and the reason it is not.

**The `ProjectReference` is declared at W7, and separating it from the project's creation is what makes the
early gate possible at all.** §12.9 states why the completed console references
`src/MvcMusicStore/MvcMusicStore.csproj` and does not restate it here; what belongs here is *when* the
reference is added, because it cannot be added when the project is created and is not needed by any verb
authored before W7. It cannot, because at gate 4a the
referenced project **does not exist at all** — W6 converts it — and even once it does,
[03](03-modernization-roadmap.md)'s W6
produces a skeleton that is not expected to compile. It is not needed, because the only
verb authored before W7 is W3's `extract-schema`, which interrogates the **source** store's catalog views
and writes the schema record and the
tool's own run rows ([05 §5.6](05-aspnet-core-migration-approach.md)): it compiles against no type the web
project owns, in the same way and for the same reason that the contracts test project of §12.2 reaches its
host without referencing it. **W7 is where the reference is declared**, on two grounds that coincide there
and nowhere else: it is the first gate at which the referenced project compiles and carries the
`ApplicationServices` seam §12.9 names, and it is the common predecessor of W8, W9 and W12 — the three
gates whose verbs do need the model, both contexts and both migration sets — between which
[03](03-modernization-roadmap.md) declares no order. Declaring it in any one of those three would leave the
other two depending on whichever happened to run first, which is not an ordering the graph carries. Adding
the reference changes the resolved graph, so the lockfile is regenerated and committed with it.

**The ordering this placement requires, stated as a required edge rather than as a settled one, because one
half of it is internal to a gate and the other half is not.** The **manifest** half needs no edge and that
much is exactly as placing it at 4a intends: a restore is a restore, the console's first restore runs under
the two root manifests of §6.1 and §6.2, and at gate 4a authoring `global.json` and `NuGet.config` and then
restoring the two projects it creates is **a sequence inside one gate's own work** rather than a dependency
between gates. The **project-existence** half is a different claim and it is not carried anywhere. W3
authors `extract-schema` as a verb of this console and **re-locks it**, which presupposes the project, its
`Program.cs`, its dispatcher and its first committed lockfile — all of which are gate 4a's output — so the
placement requires **`W4·4a → W3`**. That edge **is declared**:
[03 §4.2.1](03-modernization-roadmap.md) carries it as gate 4a's third successor, adds it explicitly with
this document's earlier reasoning named as the thing that had argued none was needed, and marks it as the
one edge whose target sits above its source in that section's otherwise topologically ordered node table.
So the ordering exists — but it exists **as an edge**, not as a consequence of where the project is
created, and that distinction is the whole of what this paragraph corrects.

**The argument that made this look already-carried is a common-successor argument, and a common successor
orders nothing.** An intermediate form of this paragraph reasoned that 4a reaches W4·4b, that `W3 → W4·4b`
already exists, and that the ordering therefore followed. It does not: both 4a and W3 being predecessors of
4b constrains each of them *relative to 4b* and says nothing about their order relative to **each other**,
so a schedule that ran W3 first is consistent with every edge in the inventory and would have W3 extending
and re-locking a project no gate had created yet — the same unexecutable shape §6's rule above exists to
prevent, arriving through a graph that looks sound. §4.2.1 remains **the single source of every edge**, it
says so itself, and it records corrections whose common cause was precisely a dependency that lived in
prose and never reached the inventory. This was one of those, and it is now declared where every other
ordering is declared: **the edge is [03](03-modernization-roadmap.md)'s and this document consumes it
rather than arguing it away.** Nothing else in this section moves: the project, its lockfile, the staged
verbs and the W7 reference are exactly as stated above, and every statement below that speaks of a verb
authored *between* gate 4a and W7 rests on that edge rather than on an inference from the placement.

The reasoning has to be stated precisely, because the two mechanisms are easy to conflate:

> **`NuGet.config` fixes *which sources* are consulted. The lockfile fixes *what resolves* from them.
> Both are needed.**

Exact direct pins do **not** lock transitives. Every version in §7.2 is an exact direct pin, and the
transitive closure behind them is still resolved at restore time. Today that gap is total: no lockfile
exists in any edition — `packages.lock.json` is absent throughout, not stale and not partial
([02](02-dependency-inventory.md) §7.1, reproduced in appendix A.4). Carrying that gap into a target whose
whole dependency posture is exact pinning would be internally inconsistent: it would pin the numbers a
human chose and leave the numbers a resolver chose floating.

Locked-mode restore in CI is the enforcement half. Without it the lockfile is documentation; with it, a
transitive change is a build failure with a named package, which is the outcome wanted.

---

## 7. Successor packages

### 7.1 One servicing band, and why

**Every Microsoft-shipped .NET 8 package in the target is pinned to `8.0.30`.**

Two reasons, one mandatory and one prudential:

- **Mandatory:** Microsoft's own guidance requires all Microsoft-shipped EF Core packages to be on one
  identical version — *"make sure to install the same version of all EF Core packages shipped by
  Microsoft"* [Microsoft Learn, *EF Core NuGet packages*,
  <https://learn.microsoft.com/ef/core/what-is-new/nuget-packages> — verified 2026-08-28]. A mixed EF Core
  graph is not a supported configuration.
- **Prudential:** holding the ASP.NET Core packages — Identity, distributed cache, data protection and the
  test host — to the **same** servicing band avoids a graph in which framework-adjacent packages disagree
  about their shared dependencies. There is no benefit to spreading them across bands and a real cost to
  debugging one that has.

Two consequences of treating the band as a unit:

1. **The band is re-verified at approval time.** These packages are on an active servicing line, so the
   current patch may have advanced by the time the work is authorized. Re-verification is a step in the
   first workstream, not a licence to write a range — the pin is replaced with the then-current exact
   patch, and this section is updated with it.
2. **The set moves together.** If the band advances, all nine rows below advance to the same number in one
   change. Advancing one package alone is precisely the mixed graph the first reason forbids.

### 7.2 The target-state pins

Exact versions, no ranges:

| Registry | Package | Version | Purpose |
| --- | --- | --- | --- |
| nuget.org | `Microsoft.EntityFrameworkCore` | `8.0.30` | ORM. `8.0.30` is a real published release, listed and not deprecated [NuGet Gallery, *Microsoft.EntityFrameworkCore*, <https://www.nuget.org/packages/Microsoft.EntityFrameworkCore> — verified 2026-08-28] |
| nuget.org | `Microsoft.EntityFrameworkCore.SqlServer` | `8.0.30` | SQL Server provider; carries `Microsoft.Data.SqlClient`, which is what supplies managed-identity authentication |
| nuget.org | `Microsoft.EntityFrameworkCore.Design` | `8.0.30` | Design-time services the migration executor loads |
| nuget.org | `Microsoft.AspNetCore.Identity.EntityFrameworkCore` | `8.0.30` | Identity store; replaces `Microsoft.AspNet.Identity` 1.0 |
| nuget.org | `Microsoft.Extensions.Caching.SqlServer` | `8.0.30` | Distributed cache backing session |
| nuget.org | `Microsoft.AspNetCore.DataProtection.EntityFrameworkCore` | `8.0.30` | Persists the data-protection key ring (§10.3) |
| nuget.org | `Microsoft.AspNetCore.Mvc.Testing` | `8.0.30` | Integration host for the test suite |
| nuget.org (tool) | `dotnet-ef` | `8.0.30` | The `dotnet ef` command — the Design package does not provide it (§6.3) |
| nuget.org (tool) | `dotnet-sql-cache` | `8.0.30` | Creates the session cache table — `Caching.SqlServer` is the runtime provider only (§6.3) |
| nuget.org | `xunit` | `2.9.2` | Test framework. **The package is flagged deprecated on nuget.org** — reason *Legacy*, suggested alternative `xunit.v3` — and §7.3 carries the approval gate that decision needs |
| nuget.org | `xunit.runner.visualstudio` | `2.8.2` | Test adapter. **Not deprecated**; listed [NuGet Gallery, *xunit.runner.visualstudio*, <https://www.nuget.org/packages/xunit.runner.visualstudio> — verified 2026-08-28] |
| nuget.org | `Microsoft.NET.Test.Sdk` | `17.11.1` | Test host and `dotnet test` integration. **Not deprecated**; listed [NuGet Gallery, *Microsoft.NET.Test.Sdk*, <https://www.nuget.org/packages/Microsoft.NET.Test.Sdk> — verified 2026-08-28] |
| nuget.org | `AngleSharp` | `1.7.2` | **HTML5/CSS-selector DOM parsing in the test projects; not referenced by the application.** Declared in `src/MvcMusicStore.Contracts.Tests` and only there, because that project owns every assertion that parses a response body and its compile and runtime assets cross the project reference to `src/MvcMusicStore.Tests` (§12.2, §12.3). Why a parser is a pin rather than a helper method is §7.5's; the registry facts are `1.7.2` listed, **not deprecated**, licensed **MIT**, publishing `net8.0` and `netstandard2.0` among its target frameworks [NuGet Gallery, *AngleSharp*, <https://www.nuget.org/packages/AngleSharp> — verified 2026-08-29] |
| nuget.org | `Microsoft.Data.SqlClient` | `5.1.9` | **Direct SQL access for the fixtures' state observer and for the legacy stores' attach and detach lifecycle; the application reaches SQL only through the EF Core provider.** Declared in `src/MvcMusicStore.Contracts.Tests` and only there, which owns the observer and the legacy fixture, and whose reference carries it to `src/MvcMusicStore.Tests` (§12.2, §12.3). The **version is not the registry head, and that is a decision rather than an oversight**: `5.1.9` is the version `Microsoft.EntityFrameworkCore.SqlServer` `8.0.30` itself resolves, so the fixtures hold the same driver the application ships — §7.6 states the choice, the alternative and why the alternative is refused. Registry facts: `5.1.9` listed, **not deprecated**, no reported vulnerability, licensed **MIT**, published 13 January 2026 [NuGet Gallery, *Microsoft.Data.SqlClient*, <https://www.nuget.org/packages/Microsoft.Data.SqlClient> — verified 2026-08-29] |
| nuget.org | `Microsoft.Extensions.Logging.ApplicationInsights` | `2.23.0` | **The single logging provider `tools/provision-admin` configures, so that its audit record has a destination** — [06 §9.5](06-azure-hosting-recommendations.md) owns the sink, the credential path and the retention, and [05 §10.2](05-aspnet-core-migration-approach.md) owns the record's shape. Declared in `tools/provision-admin/ProvisionAdmin.csproj` **and nowhere else**: the web application is instrumented by the platform without a package ([06 §9.1](06-azure-hosting-recommendations.md)), and this pin must not be read as reversing that decision. Listed, **not deprecated**, MIT, publishing `netstandard2.0`, which a `net8.0` project consumes [NuGet Gallery, *Microsoft.Extensions.Logging.ApplicationInsights*, <https://www.nuget.org/packages/Microsoft.Extensions.Logging.ApplicationInsights> — verified 2026-08-29] |
| nuget.org | `Microsoft.Playwright` | `1.62.0` | **The headless-browser harness for the one browser-executed flow of the suite — the cart page's script-issued removal request — driving Chromium.** Declared in **both** test projects rather than in one, because it delivers the driver through build assets, which do not cross a project reference (§12.2, §12.3, §7.7). **One engine is functional evidence and not a browser matrix:** the supported-browser matrix is established by the manual appearance review alone, per [05 §12.5](05-aspnet-core-migration-approach.md) and the plan's §0.11.2, and §7.7 records that boundary with the pin. Registry facts: `1.62.0` is the current **stable** release, listed, **not deprecated**, no reported vulnerability, licensed **MIT**, published 11 August 2026, publishing a `netstandard2.0` asset which a `net8.0` project consumes [NuGet Gallery, *Microsoft.Playwright*, <https://www.nuget.org/packages/Microsoft.Playwright> — verified 2026-08-29] |

The six test-tooling pins are **not** on the `8.0.30` band and are not expected to be: they version
independently of the runtime, which is why they are listed with their own exact versions rather than
folded into the band statement above. One of the six — `Microsoft.Data.SqlClient` — is nonetheless
**tied** to the band rather than independent of it, because the number it takes is the number the band's
own SQL Server provider resolves (§7.6). It moves when the band moves, and for that reason rather than on
a release cadence of its own.

**One further pin is off the band and is not a test-tooling pin either**, which is worth stating because a
reader counting rows would otherwise find seven independent versions against a set of six.
`Microsoft.Extensions.Logging.ApplicationInsights` `2.23.0` is an **operator-tool** pin: it is declared
only by `tools/provision-admin`, it is on that package's own major line rather than the runtime's, and it
exists solely so that command's audit record has a destination. No test project declares it, and the web
application declares nothing for telemetry at all — [06 §9.1](06-azure-hosting-recommendations.md) makes
platform auto-instrumentation the application's mechanism, and this pin does not reverse that.

**Every version in this table was confirmed on the registry rather than inferred**, including the parser
pin, which was checked separately and later — `AngleSharp` `1.7.2` was the current stable release on
2026-08-29, with every newer entry on the registry a prerelease
[NuGet Gallery, *AngleSharp*, <https://www.nuget.org/packages/AngleSharp> — verified 2026-08-29] — and
including the two pins checked last, on the same date: the SQL client at `5.1.9` and the browser harness
at `1.62.0`, each listed, non-deprecated, MIT-licensed and carrying no reported vulnerability, with §7.6
and §7.7 recording what was checked and why each version is the one taken, and including the Identity
abstractions pin added last, on 2026-08-29, with §7.8 recording it. All ten `8.0.30`
entries exist as published, listed, non-deprecated releases, and `Microsoft.EntityFrameworkCore` `8.0.30`
was published on 11 August 2026 — which is what makes it the current 8.0 servicing patch at the time of
writing rather than an arbitrary number [NuGet Gallery, *Microsoft.EntityFrameworkCore*,
<https://www.nuget.org/packages/Microsoft.EntityFrameworkCore> — verified 2026-08-28]. One pin —
`xunit` `2.9.2` — carries a registry deprecation, and §7.3 records it with the approval gate it needs. The
band's own currency is re-verified at approval per §7.1 consequence 1, because these are servicing lines
and a document cannot freeze one. **The runtime's support window is not discussed here at all: that entry
belongs to [07](07-effort-risks-sequencing.md)'s risk register (§2.2).**

**Advisory status, stated for the resolved graph and not only for these rows.** No package in this table,
and no package its graph resolves, carries a reported advisory at the pinned version — which is what makes
the target-state set adoptable as pinned. The one advisory-bearing dependency anywhere in this document is
**transitive to the conditional incremental path**, is reached by none of the pins above, and is recorded
with its eight advisories and its remediation floor in §7.4.

**A deprecation is a different registry attribute from an advisory, and the resolved graph is not clear of
that one.** The sentence above is about advisories and claims nothing more. Eleven packages in the closure
`Microsoft.Data.SqlClient` `5.1.9` brings — `Azure.Identity` `1.12.1`, MSAL `4.76.0` and nine others —
carry a deprecation record, one of them for a named defect, and **§7.6 is that disclosure's single home**:
it states the closure, each reason string, why the pin is nonetheless taken, which surface the residual is
actually on, and the override floor a reader who declines the residual can apply. Nothing about it changes
a version in this table.

### 7.3 `xunit` `2.9.2` is a deprecated package, and retaining it is an approval decision

This is the one pin in §7.2 that carries a registry warning, and it is recorded rather than quietly
swapped, because the swap is not this document's to make.

**What the registry says.** `xunit` — the xUnit.net v2 metapackage — is **marked deprecated on nuget.org**,
with the deprecation reason *Legacy* and the suggested alternative package `xunit.v3`. The deprecation
message is explicit about what that means: the package *"will only be updated for security issues. All
future feature work has moved onto v3."* The deprecation applies to the v2 line as a whole, not to one bad
release: both `2.9.2` and the latest v2 release `2.9.3` — published 8 January 2025 — carry the same flag
[NuGet Gallery, *xunit*, <https://www.nuget.org/packages/xunit> — verified 2026-08-28].

**Why the pin is not simply changed here.** `2.9.2` is the version the modernization plan fixes (§0.5.2 of
the Agent Action Plan), and that plan is frozen for this assessment. Substituting `xunit.v3` would be a
different test stack — a different package identity, a different runner model and a different project
shape — chosen by a document rather than by the plan owner. That is a **plan amendment, not an assessment
decision**, so this document states the fact and hands the choice over.

**The honest consequence, neither inflated nor minimized.** A deprecated package is not a broken one:
`2.9.2` is a stable, listed release that still receives security fixes, and a test project built on it
runs correctly today. What it will not receive is feature work. The real risk is procedural rather than
technical: **an organization whose dependency policy or CI gate treats any NuGet-deprecated package as
prohibited will fail the build on this pin** — not because the tests fail, but because the gate does. That
failure would arrive at the moment the first test project is authored — this pin is declared by **each** of
the two, because a test-execution package's build and analyzer assets do not cross a project reference
(§12.2, §12.3) — because that is the first point at which the pin is restored at all.

**The gate.** Before the test project is authored, the approver takes one of exactly two decisions:

| Option | What it means | Cost |
| --- | --- | --- |
| **Retain `xunit` `2.9.2`** | The frozen plan stands. The pin is documented as a knowingly accepted deprecated dependency, with a policy exception recorded if a dependency gate would otherwise block it | None technically. Requires the exception to exist wherever deprecation gates are enforced |
| **Supersede the plan to the maintained `xunit.v3` line** | AAP §0.5.2 is amended and the pin, the runner and the test project's shape change with it | A plan amendment, re-verification of the runner and adapter pins at the time of approval, and no other part of this strategy changes — the application's own package set is untouched either way |

**Owner: engineering, jointly with the plan owner** — engineering because the test stack is theirs to run,
the plan owner because only they can amend a frozen pin. A third, smaller question rides along and should
be settled in the same decision: if v2 is retained, whether to take the one-patch move from `2.9.2` to
`2.9.3`, which is within the same deprecated line and therefore changes the maintenance position not at
all.

Nothing else in §7.2 is affected. The **three** other test-project pins were checked and none is flagged:
`Microsoft.AspNetCore.Mvc.Testing` `8.0.30`, `xunit.runner.visualstudio` `2.8.2` and
`Microsoft.NET.Test.Sdk` `17.11.1` are all listed and **not** deprecated, so none joins this gate. All
three nonetheless fall under the ordinary re-verification of §7.1 consequence 1: the in-process host moves
with the band, and the two runner pins have advanced since the plan was written.

**That statement is about the five pins themselves and does not extend to what they resolve.** A package's
own registry record carries no claim about its dependencies, and one of the five has a closure that is not
clean: `Microsoft.Data.SqlClient` `5.1.9` brings eleven deprecated transitives, including MSAL `4.76.0`
under the `CriticalBugs` reason. **§7.6 records them, the reason the pin still stands, and the override
floor**, and the decision it hands over belongs to the same owners as this gate — engineering jointly with
the plan owner — at the same moment, so the two are settled together rather than separately.

### 7.4 Conditional — the incremental path only

These packages are pinned here **only so that the alternative is costed rather than hand-waved**. They
belong to the incremental migration path, and the cutover decision is
[05](05-aspnet-core-migration-approach.md)'s, not this document's. **If the single-cutover path is
confirmed, none of these is referenced at all.**

| Registry | Package | Version | Role if the incremental path is selected |
| --- | --- | --- | --- |
| nuget.org | `Yarp.ReverseProxy` | `2.3.0` | Proxies unmatched routes from the new application to the legacy one. `2.3.0` is the version Microsoft's own getting-started guidance names, and it states that *"YARP 2.3.0 supports .NET 8 or later"* [Microsoft Learn, *Get started with YARP*, <https://learn.microsoft.com/aspnet/core/fundamentals/servers/yarp/getting-started> — verified 2026-08-28] |
| nuget.org | `Microsoft.AspNetCore.SystemWebAdapters` | `2.3.0` | Shared libraries — the package referenced by code shared between the two applications [Microsoft Learn, *System.Web adapters*, <https://learn.microsoft.com/aspnet/core/migration/fx-to-core/inc/systemweb-adapters> — verified 2026-08-28] |
| nuget.org | `Microsoft.AspNetCore.SystemWebAdapters.CoreServices` | `2.3.0` | The ASP.NET Core side. Its published target frameworks are `net8.0`, `net9.0` and `net10.0`, so a `net8.0` application is in range [NuGet Gallery, *Microsoft.AspNetCore.SystemWebAdapters.CoreServices*, <https://www.nuget.org/packages/Microsoft.AspNetCore.SystemWebAdapters.CoreServices> — verified 2026-08-28] |
| nuget.org | `Microsoft.AspNetCore.SystemWebAdapters.FrameworkServices` | `2.3.0` | The .NET Framework side; its single published target framework is **.NET Framework 4.7.2**, which is therefore the floor [NuGet Gallery, *Microsoft.AspNetCore.SystemWebAdapters.FrameworkServices*, <https://www.nuget.org/packages/Microsoft.AspNetCore.SystemWebAdapters.FrameworkServices> — verified 2026-08-28] |
| nuget.org | `Microsoft.AspNetCore.SystemWebAdapters.Abstractions` | `2.3.0` | Shared session and remote-authentication abstractions [Microsoft Learn, *System.Web adapters*, <https://learn.microsoft.com/aspnet/core/migration/fx-to-core/inc/systemweb-adapters> — verified 2026-08-28] |

One strategy-relevant note on the fourth row: the `4.7.2` floor is a **package-targeting fact**, and MVC 5
at `v4.8` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:16] clears it while MVC 4 at `v4.5`
[src/MVC4/MvcMusicStore/MvcMusicStore.csproj:16] and MVC 3 at `v4.0`
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:15] do not. That is a constraint on
the incremental path's feasibility per edition; it is a supporting fact for the edition triage, not its
basis.

None of these five pins is flagged deprecated on nuget.org, checked at the same time as the test-tooling
pins of §7.3, and all five are on the same `2.3.0` release — which is the version the adapters and YARP are
documented together at, not five independently chosen numbers.

**One of the five resolves a transitive dependency with eight open HIGH advisories, and that is a blocking
condition on this path rather than a note.** A direct pin is not the whole graph, and the five rows above
are the direct half only. `Microsoft.AspNetCore.SystemWebAdapters.FrameworkServices` `2.3.0` reaches
`System.Security.Cryptography.Xml` **`8.0.2`** through
`Microsoft.AspNetCore.DataProtection` `8.0.22`, and every advisory below is against that resolved version.
Nothing in §7.1's or §7.2's graph reaches it: **the main target-state package set is advisory-clear, and
this is a cost of the conditional path alone**, which is exactly why it is recorded in this section and not
in those.

| Advisory | CVE | Severity | Affected | Fixed in |
| --- | --- | --- | --- | --- |
| GHSA-23rf-6693-g89p | CVE-2026-50648 | High | `8.0.2` | `8.0.4` |
| GHSA-37gx-xxp4-5rgx | CVE-2026-33116 | High | `8.0.2` | `8.0.3` |
| GHSA-6588-8gv4-xfgh | CVE-2026-32203 | High | `8.0.2` | `8.0.3` |
| GHSA-8q5v-6pqq-x66h | CVE-2026-50525 | High | `8.0.2` | `8.0.4` |
| GHSA-cvvh-rhrc-wg4q | CVE-2026-47302 | High | `8.0.2` | `8.0.4` |
| GHSA-g8r8-53c2-pm3f | CVE-2026-47304 | High, CVSS 8.1 | `8.0.2` | `8.0.4` |
| GHSA-mmjf-rqrv-855v | CVE-2026-50527 | High | `8.0.2` | `8.0.4` |
| GHSA-w3x6-4m5h-cxqf | CVE-2026-26171 | High | `8.0.2` | `8.0.3` |

**The combined remediation floor is `8.0.4`, because a floor is the maximum of the fixed versions and not
the first one a reader reaches.** Five of the eight are fixed at `8.0.4` and three at `8.0.3`, so a pin at
`8.0.3` closes three and leaves five open — the arithmetic is worth stating because a table read row by row
invites exactly that mistake.

| Requirement | Statement |
| --- | --- |
| **The pin** | An explicit direct `PackageReference` to `System.Security.Cryptography.Xml` at **`8.0.4` or higher**, declared alongside the conditional roots above in every project whose graph reaches them, so the transitive `8.0.2` is overridden rather than argued about. A dependency graph that resolves an equivalent or later fixed servicing line by other means satisfies it equally; what does not satisfy it is leaving the resolution implicit |
| **Why a direct pin rather than a floating range** | §6.4's committed `packages.lock.json` and locked-mode restore make the resolved graph part of the build, so a transitive version is a reviewed value here — and an explicit direct reference is the only form that shows in the manifest a reviewer reads |
| **The gate** | **The conditional incremental path may not be approved until this pin is in place.** It is a precondition on the path, not a task inside it: the adapters exist to bridge two applications' authentication and session, so shipping them over a vulnerable XML-cryptography implementation would put the defect on the seam the path exists to protect |
| **Re-verification at approval** | `8.0.4` is listed on nuget.org and was the head of the `8.0.x` servicing line on the verification date; the advisory set is re-read at approval and the floor moved up if a later advisory raises it, on the same rule §7.1 applies to the servicing band — a version is re-verified when it is adopted, never assumed to have held |
| **Scope, stated so it is not over-read** | Only the five conditional pins above are affected. If the single-cutover path is confirmed, none of them is referenced, this transitive package is not in the graph at all, and the requirement lapses with the path that carried it |

### 7.5 `AngleSharp` `1.7.2` — the HTML parser the suite needs, and why it is a pin rather than a helper

This is the one pin in §7.2 that the modernization plan's own pin set does not carry, so it is recorded
here with the reason it exists, the registry evidence behind the version, and the approval it is held to.

**What the suite asserts, and why a parser is required to assert it.** The cross-baseline suite compares
**HTML5 form semantics and CSS-selector element identity** between two runtimes: that a given form posts to
a given path with a given verb, that a named field or hidden input exists inside *that* form and not
merely somewhere on the page, that a selected element carries the value the baseline recorded. Those are
statements about a **document tree**, and the only way to evaluate them is to build one.

**Regex and substring scanning are explicitly insufficient, and this is the reason the pin exists rather
than a preference between styles.** A pattern match over response text cannot answer any of the questions
above reliably, and each failure mode below produces a *passing* test that proves nothing — which is worse
than a failing one, because nothing draws attention to it:

| What the assertion means to say | Why a text scan cannot say it |
| --- | --- |
| "This form posts to this path with this verb" | Attribute order, quoting style, whitespace and case are all free to vary; a pattern that matches today's rendering breaks on a re-render that is semantically identical, and one loosened until it stops breaking matches text anywhere on the page |
| "This hidden field exists **inside that form**" | Containment is a tree relationship. A scan sees two independent substrings and cannot tell a field inside the form from one in a sibling form or in an unrelated block |
| "Exactly one element matches, and its value is *v*" | A scan has no notion of *how many* elements a selector identifies, so "the element" silently becomes "the first textual occurrence" — and a duplicate, which is a real defect, reads as a pass |
| "This element is absent" | The hardest case: a negative assertion over text passes whenever the pattern is merely wrong, so an absence assertion and a broken pattern are indistinguishable |

A conformant parser removes all four by construction: the assertion is written against the parsed DOM with
a CSS selector, and a match count is a number rather than an inference.

**Why this package.** `AngleSharp` parses HTML5 and CSS to a DOM built to the published specification
rather than to a convention, which is exactly the conformance the rows above depend on; it is licensed
**MIT**, so it raises no licence question in a test-only dependency; and it publishes `net8.0` and
`netstandard2.0` among its target frameworks, so the `net8.0` test projects are in range without a shim.
`1.7.2` is listed and **not deprecated**, and it was the current **stable** release on the verification
date — every newer entry on the registry is a prerelease, and a prerelease is not a pin this strategy would
take [NuGet Gallery, *AngleSharp*, <https://www.nuget.org/packages/AngleSharp> — verified 2026-08-29].

**Where it is referenced, and where it is deliberately not.** It is declared by
`src/MvcMusicStore.Contracts.Tests` and **only** there (§12.2, §12.3), because that project owns every
assertion that parses a response body. It is a **library** package, so nothing about its asset kinds forces
a second declaration anywhere: compile and runtime assets are all it contributes, and those *do* cross the
project reference from `src/MvcMusicStore.Tests` — unlike the build and analyzer
assets of the three test-execution pins, which is why those three are declared directly by each runnable
test project rather than acquired across a reference (§12.3). **The web
application does not reference it**, and nothing in the application's own package set changes on account of
this pin: the parser exists to *read* the application's output, so a production reference would be a
dependency added for no runtime purpose.

**Recorded deviation from the plan's pin set — this document owns the pins, so the deviation is recorded
rather than requested.** §1.2 makes the target-state pin set this document's to fix, and §12.1 records why
a pin is not an artifact addition: every one of them is declared inside a project file the plan's map
already carries. What the record below states is the divergence from the plan's own §0.5.2 list, its cause,
and the gate the pin is held to — not an amendment for anyone to make.

| Aspect | Statement |
| --- | --- |
| **The pin** | `AngleSharp` `1.7.2`, referenced by `src/MvcMusicStore.Contracts.Tests` only — the row added to §7.2 above |
| **The omission** | The plan's §0.5.2 successor-package set fixes the test stack as `Microsoft.AspNetCore.Mvc.Testing`, `xunit`, `xunit.runner.visualstudio` and `Microsoft.NET.Test.Sdk`. It carries **no HTML parser**, while §0.3.1 requires the suite to assert on *"selected rendered content"* and on the *"presence and value of named elements"*, and §0.11.2 makes that suite an acceptance criterion |
| **Why the omission cannot stand** | Those four packages supply a host, a framework, a runner and an adapter. **None of them parses HTML**, so an assertion about a rendered element written with only that set is a text scan — and the table above is why a text scan cannot make the assertion the plan requires. The gap is not a convenience; it is the difference between an assertion and the appearance of one |
| **Cause** | The plan's pin set was assembled from the test *infrastructure* the suite needs. The parser is a consequence of what the suite *asserts*, which §0.11.2 states in prose rather than as a dependency |
| **The gate it is held to** | The approval gate of §1.4, like every other decision in this document, plus §7.1's re-verification of the number at approval time. Nothing is implemented before that approval, and the pin is scoped to `src/MvcMusicStore.Contracts.Tests` so that approving it changes nothing the application deploys |
| **Disposition** | **Recorded, and the pin is added rather than deferred.** Deferring it would leave the suite's rendered-element rows unimplementable while the pin table looked compliant. The version is re-verified at approval under §7.1 consequence 1, exactly like every other pin here |

### 7.6 `Microsoft.Data.SqlClient` `5.1.9` — the fixtures' SQL client, and why the pin is not the registry head

**Why a SQL client is a pin at all.** The fixtures do work no ORM performs for them. On the target side
they provision and drop a database and read state back out of one — the runtime-neutral state observer
[05 §12.6](05-aspnet-core-migration-approach.md) specifies, whose two implementations are the only thing in
the suite that reads a row without going through the application. On the baseline side they **attach and
detach files**: MVC 5's two connection strings are file-attached LocalDB
[src/MVC5/MvcMusicStore/Web.config:12-13], so the two-database reset
[05 §12.3](05-aspnet-core-migration-approach.md) specifies is a sequence of `CREATE DATABASE … FOR ATTACH`
and `DROP DATABASE` statements against an engine, issued before any HTTP request exists to carry them.
Neither duty has an EF Core expression, and both need a connection.

**The application does not declare it, and that asymmetry is deliberate.** `Microsoft.Data.SqlClient`
already reaches the web application transitively, carried by
`Microsoft.EntityFrameworkCore.SqlServer` — which is the §7.2 row that supplies managed-identity
authentication. **The application reaches SQL only through the EF Core provider**, so a direct reference
in the web project would add a dependency nothing in the application calls. The declaration therefore sits
in `src/MvcMusicStore.Contracts.Tests` and **only** there (§12.2, §12.3), which owns the observer and the
legacy fixture — the same arrangement as the parser pin of §7.5, and for the same reason: a library pin
contributes compile and runtime assets only, and those cross a project reference, so one declaration in the
project that calls it is the whole of what it needs (§12.3).

**Why `5.1.9` rather than the current head, stated with the alternative it was chosen over.** This is the
one pin in §7.2 whose number is *derived* rather than selected, and the derivation is checkable:
`Microsoft.EntityFrameworkCore.SqlServer` `8.0.30` declares a dependency on
`Microsoft.Data.SqlClient` `5.1.9` in its `net8.0` group, so `5.1.9` is what the **application** resolves
today with no test project in the graph at all [NuGet Gallery,
*Microsoft.EntityFrameworkCore.SqlServer*,
<https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.SqlServer> — verified 2026-08-29].

| The choice | What taking it would mean |
| --- | --- |
| **`5.1.9` — taken** | The fixtures hold **exactly** the driver the application ships. It is also the head of the `5.1` line, so nothing inside that line is left behind |
| A newer line — `5.2.3`, `6.1.6` or the current stable head `7.0.2` — **refused** | A **direct** pin wins over a transitive one, and the target half hosts the application **in process** ([05 §12.6](05-aspnet-core-migration-approach.md)): one assembly graph, so a higher fixture pin would replace the data driver *inside the very run that exists to characterize the application*. The suite would then be evidence about a driver production does not deploy. Raising the application's driver to match is a different change with a runtime consequence — a plan amendment, not a fixture decision — and it is not taken here |

**One registry fact stated plainly rather than glossed, because it is the thing a reader checks.**
`5.1.9`'s published target frameworks are `net462`, `net6.0`, `netstandard2.0` and `netstandard2.1` —
there is **no `net8.0` asset**. A `net8.0` project therefore consumes the `net6.0` asset under ordinary
framework compatibility, which is precisely what the ported application already does with this same
package brought in by the provider: the pin introduces no combination the target does not run anyway. The
newer lines refused above do publish a `net8.0` asset; that advantage does not outweigh the parity
argument. Registry status: listed, **not deprecated**, no reported vulnerability, licensed **MIT**,
published 13 January 2026 [NuGet Gallery, *Microsoft.Data.SqlClient*,
<https://www.nuget.org/packages/Microsoft.Data.SqlClient> — verified 2026-08-29].

**The pin's own record is clean; the graph behind it is not, and this section is where that is stated.**
A version's registry record says nothing about the packages that version declares, and §7.4 already fixes
this document's treatment of the case: a transitive package's registry status is recorded *with* the
remediation available for it rather than left for a reader's own restore to surface. The same treatment
applies here, and **the disclosure for this closure lives in this section** — §7.2 and §7.3 point here
rather than restating it.

`Microsoft.Data.SqlClient` `5.1.9`'s `net6.0` dependency group — the group a `net8.0` project consumes,
per the paragraph above — declares **thirteen** packages, and walking those declarations to their own
dependencies gives **38 distinct packages** at the versions a restore resolves absent any other reference
that raises them. **Eleven of the 38 carry a deprecation record on nuget.org. None of the 38 carries a
vulnerability record** [NuGet, *NuGet V3 API — registration and package content resources*,
<https://api.nuget.org/v3/index.json> — closure walked and every entry's registration leaf read, verified
2026-09-01]. That check is later than the 2026-08-29 checks recorded above, and it is a check of the
*closure* rather than of the pin, which is why its date differs.

**A deprecation and an advisory are two different registry attributes, and this is the first, not the
second.** §7.2's statement that the resolved graph is advisory-clear stands unqualified: there is no CVE,
no GHSA identifier and no fixed version in play, so there is no floor of §7.4's kind to compute. What is
in play is a maintenance signal — nuget.org marks these versions as ones their publisher no longer wants
consumed, and one of them for a specific named defect.

| Closure member | Version | Reason | What the registry record says |
| --- | --- | --- | --- |
| `Microsoft.Identity.Client` — MSAL .NET, declared by the driver | `4.76.0` | **`CriticalBugs`** | *"A critical bug affects this version, resulting in HTTP timeouts under heavy load."* The record points at the library's own issue 5460 for detail, names no alternate package, and the version remains listed. Published 15 August 2025 |
| `Azure.Identity` — declared by the driver | `1.12.1` | `Other` | *"This version takes a dependency on a deprecated version of MSAL .NET (Microsoft.Identity.Client)."* The marker is **inherited from the row above** rather than a defect of its own, which is worth stating because the two rows are one problem and not two. Published 26 September 2024 |
| `Microsoft.Identity.Client.Extensions.Msal` — reached through `Azure.Identity` | `4.65.0` | `Other` | The same inherited-dependency message, for the same reason. Published 25 September 2024 |
| `Microsoft.IdentityModel.JsonWebTokens`, `Microsoft.IdentityModel.Protocols.OpenIdConnect` and `System.IdentityModel.Tokens.Jwt`, declared by the driver, with `Microsoft.IdentityModel.Tokens`, `.Logging`, `.Abstractions` and `.Protocols` reached through them — **seven packages** | `6.35.0` | `Legacy` | *"Move to latest"*, pointing at `https://aka.ms/IdentityModel/LTS`. The marker is on the `6.x` IdentityModel line as a whole rather than on one release; the maintained line's release on the verification date is `8.22.0`, listed and not deprecated |
| `System.Text.Json` — reached through `Microsoft.IdentityModel.JsonWebTokens` | `4.7.2` | `Legacy` | *"Versions targeting out-of-support .NET versions"* — a statement about that release's own target frameworks. It is in the graph only because a `netstandard2.0`-era dependency declares it. Published 12 May 2020 |

Every reason string, message and publication date above is the registry's own, read from each version's
registration leaf on the date of the closure walk [NuGet Gallery, *Microsoft.Identity.Client*,
<https://www.nuget.org/packages/Microsoft.Identity.Client>; NuGet Gallery, *Azure.Identity*,
<https://www.nuget.org/packages/Azure.Identity> — both verified 2026-09-01].

**Why the pin is still taken, stated as the reason rather than as a dismissal.** Two facts decide it, and
neither is a judgement about how serious a deprecation is:

- **Parity is the pin's entire purpose.** `5.1.9` is not chosen; it is *derived* from what
  `Microsoft.EntityFrameworkCore.SqlServer` `8.0.30` resolves, per the derivation above. Selecting a
  different version to escape this closure would make the fixtures hold a driver — and an authentication
  stack — the application does not deploy, which is the failure the refused row of the table above exists
  to prevent. A suite that characterizes a graph production does not run is weaker evidence than one that
  shares production's residual risk.
- **The band is not available to move.** §7.1 fixes every Microsoft-shipped .NET 8 package at `8.0.30`
  and requires the set to move together, and that pinning is the plan's (AAP §0.5.2), not this document's.
  There is no `8.0.x` servicing patch this document may select in order to obtain a provider that resolves
  a different SQL client, and assembling a mixed band to obtain one is the configuration §7.1's first
  reason forbids outright.

**What the pin does and does not expose, because "test-only" is the tempting answer and it is the wrong
one.** The *declaration* is test-only: `src/MvcMusicStore.Contracts.Tests` is the one project that names
this package (§12.2, §12.3) and the application names it nowhere. The *closure* is not test-only, and the
parity
property is precisely why — `Microsoft.EntityFrameworkCore.SqlServer` `8.0.30` declares
`Microsoft.Data.SqlClient` `5.1.9` in its own `net8.0` group [NuGet Gallery,
*Microsoft.EntityFrameworkCore.SqlServer*,
<https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.SqlServer> — verified 2026-09-01], so every
row of the table above is in the **web application's** resolved graph whether the test project exists or
not. Two consequences follow, and they point in opposite directions:

- **The pin adds nothing.** It introduces no package the application does not already resolve and no
  version the application does not already resolve. Removing the pin would remove none of the rows in the
  table above — which is the whole of what the parity argument claims, and the reason this disclosure is
  not an argument against the pin.
- **The exposure is therefore the application's, not the fixtures'.** `Azure.Identity` and MSAL are the
  libraries this driver uses to acquire Microsoft Entra access tokens for its Active Directory
  authentication modes [Microsoft Learn, *Connect to Azure SQL with Microsoft Entra authentication and
  SqlClient*,
  <https://learn.microsoft.com/sql/connect/ado-net/sql/azure-active-directory-authentication> — verified
  2026-09-01], and managed-identity data-plane authentication is what the target selects
  ([06](06-azure-hosting-recommendations.md)). So the `CriticalBugs` row — a documented failure under
  heavy load — sits on the application's own connection path rather than on a fixture connecting to a
  disposable database. The honest description of that residual is **"unchanged by this pin"**, never
  "absent" and never "fixture-scoped".

**The override floor available to a reader who does not accept the residual.** Recorded in full, because a
disclosure that offers no remedy is a note rather than a finding:

| Aspect | Statement |
| --- | --- |
| **The override** | A direct top-level `PackageReference` to `Azure.Identity` and to `Microsoft.Identity.Client` at the current supported release of each — **`1.21.0`** and **`4.88.0`** on the verification date, each listed, MIT-licensed, **not deprecated** and carrying no advisory, published 11 April 2026 and 20 August 2026 respectively [NuGet Gallery, *Azure.Identity*, <https://www.nuget.org/packages/Azure.Identity>; NuGet Gallery, *Microsoft.Identity.Client*, <https://www.nuget.org/packages/Microsoft.Identity.Client> — both verified 2026-09-01]. A direct reference wins over a transitive one, so the resolution rises **without the EF Core band moving and without altering the `5.1.9` selection**. Both numbers are re-verified when the override is adopted, under §7.1 consequence 1 — a release is supported on a date, not permanently |
| **Both references are required; neither alone suffices** | `Azure.Identity` `1.21.0`'s `net8.0` group declares only `Azure.Core` `1.53.0` — the current line no longer depends on MSAL at all — while MSAL is declared by `Microsoft.Data.SqlClient` `5.1.9` **directly**, so raising `Azure.Identity` alone leaves `4.76.0` in place. Conversely MSAL `4.88.0` raises `Microsoft.IdentityModel.Abstractions` to `8.14.0` but not the three IdentityModel packages the driver declares itself, which stay at `6.35.0` [NuGet, *NuGet V3 API — package content resource*, <https://api.nuget.org/v3/index.json> — dependency groups read 2026-09-01] |
| **Where it must be declared** | In **every** project whose graph reaches the closure — the web application *and* the test project — for the same reason §7.4 gives for its own floor. A version raised in one project and not the other is two graphs, and that divergence is the exact asymmetry the `5.1.9` derivation exists to refuse: the fixtures would then hold authentication libraries the application does not deploy |
| **The trade, plainly** | Once the override exists, "what the provider resolves by default" and "what the application deploys" stop being the same sentence. The parity property survives only as long as both declarations are maintained together, so every band move re-verifies two chosen numbers deliberately instead of re-reading one derived number (the rule below). The cost is maintenance rather than behaviour: the driver's declared floor is *at least* `1.12.1` and `4.76.0`, and both override targets are above it |
| **Why it is not taken here** | It changes versions in the **application's** resolved graph, and the application's dependency set is fixed by the plan (§7.1, AAP §0.5.2) — the same boundary that refuses the newer SQL client line above, and the same reason §7.3 hands the `xunit` substitution over rather than making it. Recorded as available and costed, not exercised |
| **Owner** | **Engineering jointly with the plan owner**, on §7.3's model: engineering because the graph is theirs to restore and run, the plan owner because a version the plan fixed cannot be moved by a document. The decision is due at the moment §7.1 consequence 1 re-verifies the band, because that is when every one of these numbers is re-read anyway |
| **The other nine rows** | Same mechanism, same owner, larger blast radius: the seven `6.35.0` IdentityModel packages move as a **line** to the maintained release (`8.22.0` on the verification date), which is a major-version move rather than a servicing bump, and `System.Text.Json` `4.7.2` is a `netstandard2.0`-era entry a `net8.0` project carries only because the IdentityModel line declares it. Neither sits on the token-acquisition path the `CriticalBugs` row sits on, so neither is a precondition on anything; both are re-read at approval with the rest. `Microsoft.Identity.Client.Extensions.Msal` `4.65.0` is reached through `Azure.Identity` and clears when that row clears |
| **The line that removes the closure rather than raising it** | `Microsoft.Data.SqlClient` `7.0` extracts these dependencies out of the core package: it *"no longer depends on Azure.Core, Azure.Identity, or their transitive dependencies (such as Microsoft.Identity.Client…)"*, moving Entra support into a separate `Microsoft.Data.SqlClient.Extensions.Azure` package that an application using any Entra authentication mode must then reference explicitly [Microsoft Learn, *Connect to Azure SQL with Microsoft Entra authentication and SqlClient*, <https://learn.microsoft.com/sql/connect/ado-net/sql/azure-active-directory-authentication> — verified 2026-09-01]. That is the structural end state rather than a mitigation, and it is **not available here**: it is a driver-line move for the application, refused above on parity grounds and reserved to the plan owner. It is recorded so an approver knows the residual has an end state and not only a floor |

**Consequence for §7.1's "the set moves together" rule.** When the band advances, this pin does not get
re-chosen — it is re-read: the number becomes whatever the new
`Microsoft.EntityFrameworkCore.SqlServer` resolves, and the parity property is what is preserved across
the move rather than the digits.

**Recorded deviation from the plan's pin set.** Same form and same reason as §7.5's record: the pin set is
this document's (§1.2), the pin is declared inside a project file the plan's map already carries (§12.1),
and what follows records the divergence and the gate rather than asking anyone to amend a plan.

| Aspect | Statement |
| --- | --- |
| **The pin** | `Microsoft.Data.SqlClient` `5.1.9`, referenced by `src/MvcMusicStore.Contracts.Tests` only — the row added to §7.2 above |
| **The omission** | The plan's §0.5.2 successor-package set carries no SQL client for the test projects, while §0.3.1 requires a legacy reset that *"restores both committed `.mdf`/`.ldf` pairs, not one, and reattaches them before each run"* and requires the target fixture to provision a disposable engine, apply both migration sets and load a dataset *"with asserted row counts and key invariants"* |
| **Why the omission cannot stand** | Attach, detach, drop and a row-count read are statements executed against an engine. `Microsoft.AspNetCore.Mvc.Testing` supplies a host, `xunit` a framework, `AngleSharp` a parser: **none of them opens a connection.** Without a client the reset and the invariant assertions are prose, and the fixture the plan makes an acceptance criterion in §0.11.2 cannot be built |
| **Cause** | The plan's pin set was assembled for the *application*, plus the four test-infrastructure packages. The client is transitive for the application, so it needed no row there — and the fixtures' own need for it is stated in §0.3.1 as behaviour rather than as a dependency |
| **The gate it is held to** | §1.4's approval gate, with §7.1's re-verification applied to the *derivation rule* above rather than to a fixed number — when the band moves, the pin is re-read from the provider's dependency, not re-chosen |
| **Disposition** | **Recorded, and the pin is added rather than deferred.** A fixture that cannot connect is not a fixture, and the affected gates are [03](03-modernization-roadmap.md)'s W4 and W7 |

### 7.7 `Microsoft.Playwright` `1.62.0` — one browser-executed flow, and the boundary that stays manual

**What it is for, exactly one flow wide.** The application has a single scripted flow: the cart page's
removal handler [src/MVC5/MvcMusicStore/Views/ShoppingCart/Index.cshtml:7-35]. Every other caller-side
artifact is markup that ordinary HTTP can fetch, parse and submit, which is what the parser pin of §7.5
serves. Script is different in kind: it has to **execute** before there is anything to assert. This pin is
the harness that executes it — a real browser engine, driven headlessly, loading the page from the host
under test and asserting on the resulting DOM. [05 §12.11](05-aspnet-core-migration-approach.md) owns what
the flow must establish and why a static assertion over the script's text is not equivalent;
[07](07-effort-risks-sequencing.md) owns what it costs. This section owns the pin.

**Chromium, and one engine is the claim.** The harness drives **Chromium**. That is *functional* evidence —
the script runs, the header is sent, the response updates the named elements — and it is **not** a
statement about browser support. The supported-browser matrix is established by the manual appearance
review that [05 §12.5](05-aspnet-core-migration-approach.md) scopes and the plan's §0.11.2 requires, and
nothing in this pin extends to it. Stating the boundary is the point: a suite that ran one engine and
implied four would be making the strongest claim in the document out of the weakest evidence in it.

**Where it is declared: both test projects, and unlike the parser and the SQL client it cannot be declared
in one.** `src/MvcMusicStore.Contracts.Tests` declares it because the runtime-neutral harness code compiles
against its types and because the installer script the runbook invokes is a build output of that project
([05 §12.10](05-aspnet-core-migration-approach.md)). `src/MvcMusicStore.Tests` declares it **as well**,
because the case that drives a real engine — [05 §12.4](05-aspnet-core-migration-approach.md) row 28 `28b`
— is **target-only**, so it executes from *that* project's output, and this package delivers the driver and
the generated `playwright.ps1` through **build assets**, which do not cross a project reference (§12.3). A
single declaration in the contracts project would leave the target project's output with the compile-time
types and no driver beside the assembly, which fails at run time on a missing driver rather than at build
time — the same asset-flow trap as the three test-execution pins, arriving one step later. The **engine**
install is not affected either way: `playwright install` places browser binaries in a machine-level cache,
so one install step serves both projects, which is why the runbook runs it once. **The web application
references it in neither form** — it is a harness that drives the application, so a production reference
would ship a browser driver inside a web application for no runtime purpose.

**Two dependency facts that belong to the pin rather than to the runbook.** First, the package ships the
driver and **not** the browser: the engine binaries are acquired by the package's own install step, which
is a prerequisite of any run that selects the flow — its placement in the developer runbook is
[05 §12.10](05-aspnet-core-migration-approach.md)'s and in the pipeline
[06 §12.1](06-azure-hosting-recommendations.md)'s. Second, `1.62.0` publishes a single
`netstandard2.0` asset, which a `net8.0` project consumes under ordinary framework compatibility; there is
no `net8.0`-specific asset to prefer and none is needed. Registry status: current **stable** release,
listed, **not deprecated**, no reported vulnerability, licensed **MIT**, published 11 August 2026
[NuGet Gallery, *Microsoft.Playwright*, <https://www.nuget.org/packages/Microsoft.Playwright> — verified
2026-08-29].

**Recorded deviation from the plan's pin set.** Same form as §7.5's and §7.6's records.

| Aspect | Statement |
| --- | --- |
| **The pin** | `Microsoft.Playwright` `1.62.0`, referenced by **both** test projects and by the application in neither form — the row added to §7.2 above. It is declared twice rather than once for the reason the paragraph above and §12.2's two project rows give: it delivers the driver and the generated `playwright.ps1` through **build assets**, which do not cross a project reference, so the project whose output runs the flow has to declare it itself |
| **The omission** | The plan's §0.5.2 set names no browser harness, and its §0.11.2 offers a manual review for **rendered appearance** only. The one flow above is neither appearance nor a request a client can hand-build faithfully: it is the application's own script deciding what to send |
| **Why the omission cannot stand** | Substituting a reviewer clicking the remove link would reclassify a **behavioural** gap as a visual one, which is the one substitution §0.11.2's manual allowance does not authorize. Without the harness the flow is covered statically — the token is rendered, the header name appears in the script — and every runtime failure mode ([05 §12.11](05-aspnet-core-migration-approach.md) enumerates them) passes that check while the control does nothing in a browser |
| **Cause** | The plan's coverage prose treats the suite as HTTP-level throughout, which it is for every surface but this one. The exception is a property of the application — one scripted flow — rather than of the test strategy |
| **The gate it is held to** | §1.4's approval gate and §7.1's re-verification, with the claim boundary stated in the same breath as the pin: browser-executed coverage is **one engine for behaviour**, and the browser matrix remains the manual appearance review's |
| **Disposition** | **Recorded, and the pin is published rather than deferred.** [05 §12.11](05-aspnet-core-migration-approach.md) states explicitly that it names no package because this document owns the test projects' dependency set; publishing the pin is how that hand-off is discharged. The scope decision it serves stays with the plan owner and [07](07-effort-risks-sequencing.md) re-estimates the workstreams it lands in |

---

### 7.8 The normalizer the suite invokes needs no pin — its assembly arrives across the project reference on the band's own number

This pin exists because another deliverable's design **invokes** a type, and a type that is invoked has to
come from a package something declares. Deliverable [05 §12.9](05-aspnet-core-migration-approach.md)
canonicalizes every cross-run diagnostic pseudonym by resolving **`ILookupNormalizer`** and calling it,
explicitly rather than reimplementing its behaviour, and it states in the same row that it names no version
because *this* section owns the pin. Before this row that hand-off was unclosed: the test project
invoked the abstraction and declared nothing that provides it, which is not a version disagreement but an
open dependency — the project would not compile.

**Why the invocation and not a description of it, which is what makes the assembly non-optional.** 05's own row
records the correction: .NET 8's default normalizer, `UpperInvariantLookupNormalizer`, performs Unicode
`Normalize()` followed by `ToUpperInvariant()` and does **not** trim, so a hand-written "trim and upper-case"
substitute is wrong in **both** directions — trimming merges two accounts Identity keeps distinct, and
omitting the Unicode normalization splits two accounts Identity treats as one. Either defect presents as a
per-account mismatch attributable to neither runtime. The only way to be sure the suite canonicalizes the
way the application does is to call the same implementation, and that requires the assembly it lives in.

**Which type, and where it comes from.** The suite resolves the abstraction **`ILookupNormalizer`** and
uses the framework's default implementation, **`UpperInvariantLookupNormalizer`** — both declared in the
`Microsoft.AspNetCore.Identity` namespace and shipped in the `Microsoft.Extensions.Identity.Core` assembly.
Recording the concrete type matters for a reason beyond documentation: [05
§12.10](05-aspnet-core-migration-approach.md) writes the normalizer's **type name and assembly version**
into the baseline metadata so a substitution fails the hand-off instead of silently re-aliasing accounts.

**Where it is declared, and where it deliberately is not.** It is declared by
`src/MvcMusicStore.Contracts.Tests` and **only** there (§12.2, §12.3), which owns the diagnostic record and
the pseudonym scheme. Like the parser and the SQL client it is a **library** pin — compile and runtime
assets only, and those cross the project reference to `src/MvcMusicStore.Tests` — so
one declaration in the project that resolves the abstraction is the whole of what it needs, unlike the
test-execution pins of §7.2. **The web application declares no reference to it and needs none**: it reaches
exactly the same API transitively through `Microsoft.AspNetCore.Identity.EntityFrameworkCore` `8.0.30`,
which depends on it, and Identity resolves the normalizer from the container when it computes
`NormalizedUserName` and `NormalizedEmail`. Adding a direct reference in the web project would restate a
dependency the Identity stack already carries.

**It belongs to the `8.0.30` band, not beside the six test-tooling pins.** It is a Microsoft-shipped .NET 8
package on the same servicing line as the rest of the band — which is also the correctness argument for the
version rather than a convenience: the normalizer the suite calls must be the one the **application's**
Identity stack calls, and the application's arrives transitively at the band's number. Pinning the suite to
a different patch would reintroduce, at the package level, exactly the divergence invoking the normalizer
exists to remove. So it moves with the band under §7.1's two consequences, and it is counted among the ten
band entries rather than among the six independent ones.

**Registry verification, performed rather than assumed.** `8.0.30` is a real published release: **listed**,
**not deprecated** (the registration leaf carries no deprecation record and the gallery page shows no
deprecation notice), licensed **MIT** by expression, published **11 August 2026** — the same date as the
band's own EF Core release, which is what places it on this servicing line — with **no reported
vulnerability**, and its package targets include **`net8.0`** alongside `net462` and `netstandard2.0`, so a
`net8.0` test project consumes a framework-specific asset rather than a compatibility fallback. `8.0.30` is
also the **highest** `8.0.x` version on the registry, so it is the current patch of that line and not an
arbitrary point in it. Its own `net8.0` dependencies are `Microsoft.AspNetCore.Cryptography.KeyDerivation`
`8.0.30`, `Microsoft.Extensions.Logging` `8.0.1` and `Microsoft.Extensions.Options` `8.0.2` — all inside the
band or below it, so the pin introduces no package outside the graph the band already resolves
[NuGet Gallery, *Microsoft.Extensions.Identity.Core*,
<https://www.nuget.org/packages/Microsoft.Extensions.Identity.Core> — verified 2026-08-29].

**Recorded deviation from the plan's pin set.** Same form as the records in §7.5, §7.6 and §7.7.

| Aspect | Statement |
| --- | --- |
| **The pin** | `Microsoft.Extensions.Identity.Core` `8.0.30`, referenced by `src/MvcMusicStore.Contracts.Tests` only — the row added to §7.2 above |
| **The omission** | The plan's §0.5.2 successor set names `Microsoft.AspNetCore.Identity.EntityFrameworkCore` for the **application** and no Identity package for the test projects, and its §0.11.2 requires the suite's diagnostics without specifying how a per-account identifier is canonicalized. The abstraction the design settled on is therefore invoked by a project whose declared package set does not contain it |
| **Why the omission cannot stand** | An invoked type with no declaring package is a project that does not compile — the one failure class this document exists to enumerate rather than discover during implementation. The alternative that needs no pin is worse and is the one 05 rejected with evidence: reimplementing the normalization merges or splits accounts relative to the application's own rule, and the resulting per-account mismatch is attributable to neither runtime |
| **Cause** | The plan's pin set was derived from the **application's** runtime needs, and the suite's needs were derived from its *test-tooling* needs. A pin that is an application-framework package used **by the tests** falls between the two derivations, which is the same gap that produced records 3, 4 and 5 |
| **The gate it is held to** | §1.4's approval gate, with §7.1's band rule doing the version work: the pin moves with the band because the normalizer the suite calls must be the one the application's Identity stack calls, and the recorded claim is that the suite **invokes** `ILookupNormalizer` with the framework's `UpperInvariantLookupNormalizer` rather than substituting a casing rule of its own |
| **Disposition** | **Recorded, and the pin is published rather than deferred.** [05 §12.9](05-aspnet-core-migration-approach.md) states explicitly that it names no version because this document owns the test projects' dependency set; publishing the pin is how that hand-off is discharged. [07](07-effort-risks-sequencing.md) re-estimates the workstream it lands in |

## 8. Per-package outcomes — all 28 MVC 5 pins, exactly one outcome each

### 8.1 How to read this section

MVC 5 pins **28 packages** [src/MVC5/MvcMusicStore/packages.config:3-30]. Every one of them appears
exactly once in the table below with exactly one outcome. The pin *values* are not re-transcribed as an
inventory — [02](02-dependency-inventory.md) §3.1 owns them, and §3 there owns all 63 pins across the
three editions. What this table adds is the **disposition**.

Six outcome classes, and the distinctions between them are not cosmetic — each one implies different
work:

| Class | Meaning |
| --- | --- |
| **A — Shared framework** | The capability is in the ASP.NET Core shared framework. No `PackageReference` replaces it |
| **B — Named successor** | A specific target package takes over, at a version in §7.2 |
| **C — Framework registration** | The capability survives but as framework configuration rather than as a package |
| **D — Unused** | Removed because nothing consumes it. Not a replacement of any kind |
| **E — No successor** | The construct itself does not exist in the target. Removed, not migrated |
| **F — Client-side** | A vendored browser library, handled by §9 |

### 8.2 The table

Listed in the manifest's own order, so it can be checked line-for-line against
[src/MVC5/MvcMusicStore/packages.config:3-30].

| # | Package (current pin) | Class | Outcome |
| --- | --- | --- | --- |
| 1 | `Antlr` | E | Removed. A parser runtime present only as a transitive of the minification stack; with `WebGrease` and `Microsoft.AspNet.Web.Optimization` gone, nothing references it |
| 2 | `bootstrap` | F | Updated and vendored → **bootstrap `5.3.3`**. The package upgrade is the easy half; the Bootstrap 3 markup in the views must be migrated with it, and that work is [05](05-aspnet-core-migration-approach.md)'s |
| 3 | `EntityFramework` | B | → **`Microsoft.EntityFrameworkCore` `8.0.30`** and **`Microsoft.EntityFrameworkCore.SqlServer` `8.0.30`**, with **`Microsoft.EntityFrameworkCore.Design` `8.0.30`** for design-time. Not a version bump: EF 6 and EF Core are different products, and the behavioural differences are [12](12-migration-blockers.md) F-12-15 and F-12-19 |
| 4 | `jQuery` | F | Updated and vendored → **jquery `3.7.1`** |
| 5 | `jQuery.Validation` | F | Updated and vendored → **jquery-validate `1.21.0`** — the cdnjs library id, not `jquery-validation` (§9.4) |
| 6 | `Microsoft.AspNet.Identity.Core` | B | → **`Microsoft.AspNetCore.Identity.EntityFrameworkCore` `8.0.30`** |
| 7 | `Microsoft.AspNet.Identity.EntityFramework` | B | → the same package as row 6. The shipped store is the Identity 1.0 schema, and migrating its *data* is a separate workstream owned by [05](05-aspnet-core-migration-approach.md) and gated by [12](12-migration-blockers.md) F-12-21 |
| 8 | `Microsoft.AspNet.Identity.Owin` | C | Removed. It bridged Identity to the OWIN host; with no OWIN host there is no bridge to provide. Identity is registered directly in the composition root |
| 9 | `Microsoft.AspNet.Mvc` | A | Removed — MVC is part of the ASP.NET Core shared framework. No package reference replaces it |
| 10 | `Microsoft.AspNet.Razor` | A | Removed — the Razor view engine is part of the shared framework |
| 11 | `Microsoft.AspNet.Web.Optimization` | E | Removed. Bundling and minification have **no ASP.NET Core successor package**, and the five bundle registrations use `{version}` and glob token forms nothing in the target reproduces ([12](12-migration-blockers.md) F-12-02). Static assets are served per §9 |
| 12 | `Microsoft.AspNet.WebPages` | E | Removed. The Web Pages runtime and `System.Web.Helpers` have no counterpart |
| 13 | `Microsoft.jQuery.Unobtrusive.Validation` | F | Replaced and vendored → **jquery-validation-unobtrusive `4.0.0`** |
| 14 | `Microsoft.Owin` | E | Removed. Katana host abstractions have no successor type |
| 15 | `Microsoft.Owin.Host.SystemWeb` | E | Removed. It hosts OWIN on the `System.Web` pipeline; both sides of that sentence are gone |
| 16 | `Microsoft.Owin.Security` | E | Removed. The authentication middleware base is replaced by the framework's own authentication stack, which is not a package |
| 17 | `Microsoft.Owin.Security.Cookies` | C | Removed. Cookie authentication survives as a **framework registration**, not as a package — this is the one authentication package whose capability is genuinely carried forward |
| 18 | `Microsoft.Owin.Security.Facebook` | E | Removed. A dormant social provider — its registration is commented out today. See the note below |
| 19 | `Microsoft.Owin.Security.Google` | E | Removed. Dormant social provider; see the note below |
| 20 | `Microsoft.Owin.Security.MicrosoftAccount` | E | Removed. Dormant social provider; see the note below |
| 21 | `Microsoft.Owin.Security.OAuth` | E | Removed. **Not a social provider** — it is OAuth server and bearer infrastructure, which is why it is in this class and not grouped with rows 18-20, 22 |
| 22 | `Microsoft.Owin.Security.Twitter` | E | Removed. Dormant social provider; see the note below |
| 23 | `Microsoft.Web.Infrastructure` | E | Removed. Dynamic module registration is a `System.Web` concern with no analogue |
| 24 | `Modernizr` | F | **Removed, with no replacement.** A feature-detection library for an Internet Explorer-era browser matrix. The narrowed matrix is a product decision recorded in [06](06-azure-hosting-recommendations.md), and its compatibility cost is a risk entry in [07](07-effort-risks-sequencing.md) |
| 25 | `Newtonsoft.Json` | D | **Removed as an unused dependency.** See §8.4 — this is the row most likely to be misread |
| 26 | `Owin` | E | Removed. This is the `IAppBuilder` abstraction itself; **no successor type exists** ([12](12-migration-blockers.md) F-12-03) |
| 27 | `Respond` | F | **Removed, with no replacement.** A media-query polyfill for Internet Explorer 8; Bootstrap 5 drops Internet Explorer entirely, so the polyfill has nothing to polyfill |
| 28 | `WebGrease` | E | Removed. The minification engine behind `Microsoft.AspNet.Web.Optimization`; it goes with row 11 |

### 8.3 The count, checkable

| Class | Count | Rows |
| --- | --- | --- |
| **A** — Shared framework, no package reference | **2** | 9, 10 |
| **B** — Named successor | **3** | 3, 6, 7 |
| **C** — Capability becomes a framework registration | **2** | 8, 17 |
| **D** — Removed as unused | **1** | 25 |
| **E** — Removed, no successor at all | **14** | 1, 11, 12, 14, 15, 16, 18, 19, 20, 21, 22, 23, 26, 28 |
| **F** — Client-side library (§9) | **6** | 2, 4, 5, 13, 24, 27 |
| **Total** | **28** | — |

2 + 3 + 2 + 1 + 14 + 6 = **28**, matching the 28 pins in
[src/MVC5/MvcMusicStore/packages.config:3-30] exactly. No identifier is unassigned and none is assigned
twice. The appendix reproduces both the pin count and the identifier list (A.2).

**On the four dormant social providers (rows 18-20 and 22).** They are removed rather than mapped, because
**social sign-in is not enabled in the checked-in configuration**: all four external-provider
registrations — Microsoft Account, Twitter, Facebook and Google — are commented out at
[src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:23-35], under a comment inviting the reader to uncomment
them, while the four provider packages ship regardless. That is a statement about the current snapshot and
nothing more: committed configuration records what the checked-in code would do if run, not what any
deployment has ever done, and the repository carries no deployment history, release record or environment
configuration from which an "it was never enabled" claim could be drawn. [12](12-migration-blockers.md)
F-12-09 owns the evidence and the resulting design choice.

The consequence for this document is a scoping one and it does not depend on history. The framework's
`Microsoft.AspNetCore.Authentication.*` family is the successor family **if and only if social sign-in is
actually enabled in the target**, and pinning versions for a capability that is disabled in the source
would be specifying work nobody has asked for. If it is enabled, those packages are pinned at that time,
at the then-current band.

### 8.4 `Newtonsoft.Json` — removed as unused, *not* replaced as a serializer

This distinction has to be stated exactly, because getting it wrong misdescribes a separate blocker.

`Newtonsoft.Json` `5.0.6` is pinned [src/MVC5/MvcMusicStore/packages.config:27] and referenced by the
project [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:99-102], so it is restored, copied to `bin` and
deployed today. **No application source file calls it.**
[02](02-dependency-inventory.md) §3.1.3 owns that finding and reproduces the search; the package is
template baggage.

So its outcome is **removal of a dependency with no consumer**. It is *not* a serializer replacement,
because there is no serializer usage to replace:

- The application's one JSON-producing endpoint returns an MVC `JsonResult`, and in MVC 5 that result type
  serializes through `JavaScriptSerializer` — not through Newtonsoft.Json. That is
  [12](12-migration-blockers.md) F-12-16's establishing fact.
- The target serializes through `System.Text.Json`, which is in the shared framework and needs no package
  (§10.1).
- The **behavioural** consequence — that `System.Text.Json`'s web defaults camel-case property names while
  the current output is PascalCase, silently breaking the client-side reads — is a migration blocker owned
  by F-12-16 and resolved by [05](05-aspnet-core-migration-approach.md).

Stating this row as "Newtonsoft.Json → System.Text.Json" would imply the JSON contract is preserved by a
package swap. It is not. One row removes an unused dependency; a different, separately owned finding
changes the wire format, and it would change it identically whether or not this package were pinned at all.

### 8.5 MVC 3's provider has no outcome in this table, and that is the point

`System.Data.SqlServerCe.4.0` is MVC 3's catalogue provider. It is **not** a NuGet pin — it appears in
none of the three `packages.config` files — and it is **retired with no supported provider for the target
framework**. [12](12-migration-blockers.md) F-12-01 owns the finding, and
[02](02-dependency-inventory.md) §4.1 records that neither MVC 3's MVC framework assembly nor its provider
is a package.

It is named here only to close a gap a reader would otherwise find: MVC 3's data layer cannot be ported
without re-targeting its provider outright, which is one of the facts behind MVC 5 being the sole
migration source. MVC 3 is not ported (§12.3), so no successor pin is required.

---

## 9. Client-side libraries: exact pins and a declared acquisition mechanism

### 9.1 Why the acquisition mechanism is a dependency decision

Today the browser libraries arrive **as NuGet packages** — six of MVC 5's 28 pins deliver script and
stylesheet content rather than an assembly, and the project lists the resulting files individually as
`Content` and `None` items: the stylesheets at
[src/MVC5/MvcMusicStore/MvcMusicStore.csproj:192-193], the scripts at
[src/MVC5/MvcMusicStore/MvcMusicStore.csproj:203-215] and
[src/MVC5/MvcMusicStore/MvcMusicStore.csproj:233], and the Bootstrap 3 Glyphicons font files at
[src/MVC5/MvcMusicStore/MvcMusicStore.csproj:195] and
[src/MVC5/MvcMusicStore/MvcMusicStore.csproj:262-264]. NuGet stopped being the delivery
channel for browser libraries long ago, so removing `packages.config` (§5.2) removes the acquisition
mechanism as well as the pins. Something has to take its place, and leaving it unspecified would leave
the four surviving rows in class F with a target version and no way to obtain it.

### 9.2 The six outcomes

Six current pins deliver browser content. **Four are retained at a new version and two are removed with no
successor, so the retained-library count is four, not six** — and that is the number `libman.json` declares.

| Current pin | Outcome | Target library and exact version |
| --- | --- | --- |
| `jQuery` `1.10.2` | Update, vendored | **jquery `3.7.1`** |
| `jQuery.Validation` `1.11.1` | Update, vendored | **jquery-validate `1.21.0`** — the cdnjs library id is `jquery-validate`, not `jquery-validation`; see §9.4 for the manifest entry and §9.4 for the restore-time error a wrong id produces |
| `Microsoft.jQuery.Unobtrusive.Validation` `3.0.0` | Replace, vendored | **jquery-validation-unobtrusive `4.0.0`** |
| `bootstrap` `3.0.0` | Update, vendored — **plus markup work** | **bootstrap `5.3.3`** |
| `Respond` `1.2.0` | **Remove**, no successor | none |
| `Modernizr` `2.6.2` | **Remove**, no successor | none |

Four retained, two removed. The two removals are what make the arithmetic add up, and they are removals
rather than replacements: `Respond` is a media-query polyfill for Internet Explorer 8 and `Modernizr` is
feature detection for the same browser generation, and the target's browser matrix — owned by
[06](06-azure-hosting-recommendations.md), with its compatibility cost a risk entry in
[07](07-effort-risks-sequencing.md) — no longer includes those browsers.

The Bootstrap row is the only one where the package version is not the whole change: the views use the
Bootstrap 3 class vocabulary, so upgrading the library without touching markup would change the rendered
page. That markup work, and the decision about icons, belong to
[05](05-aspnet-core-migration-approach.md).

### 9.3 The mechanism, specified

- **Assets are vendored into `wwwroot`.** They are served by static-file middleware, with the framework's
  version-appending tag helper providing cache busting (§10.1). There is no bundler, because there is no
  successor to the removed bundling framework and an asset set of this size
  (27 files, by the four separate `git ls-files` pathspecs of appendix A.3 — a single braced pathspec
  returns `0` here, for the reason that appendix records) does not
  justify introducing a JavaScript toolchain and its own dependency tree. That **27 is the assessed
  source inventory, not the target payload.** Eleven of those files are the Bootstrap 3 and
  jQuery 1.10.2 copies the four retained libraries below replace, and ten more are removed outright as
  development-only, unsupported or unused, leaving six application-owned files to relocate.
  [05](05-aspnet-core-migration-approach.md) §8.1.1 owns the per-file disposition and states the served
  payload that results; this document owns only the acquisition of the replacements.
- **They are declared in a committed `libman.json`**, using the Microsoft Library Manager with
  `defaultProvider` set to `cdnjs`. Library Manager needs **no npm, no `node_modules` and no build-time
  toolchain** — which is the reason it is chosen over an npm-based flow at this scale.
- **Three counts are in play here and they are different numbers**, stated together because a single
  figure quoted loosely is what makes them contradict each other downstream: **six source package
  dispositions** — the six rows of §9.2, four updated or replaced and two removed; **four acquired
  libraries** — the four `libraries` entries of the manifest in §9.4, being the four that survive; and
  **thirteen acquired files** — that manifest's total file selection, `3 + 4 + 2 + 4`, because each entry
  selects the variants and maps its own consumers require rather than one file apiece (§9.5). Six
  dispositions, four libraries, thirteen files.
  [05](05-aspnet-core-migration-approach.md) §8.1.1 adds those thirteen to the application's own relocated
  and new files to state the served payload, so that total is its figure and moves with this one.
- **The restored files under `wwwroot/lib/` are committed.** Two consequences, both deliberate: a build
  never depends on reaching a CDN, and deployment needs no client-asset restore step.
- **`libman restore` is therefore a developer-initiated update action, not a CI step.** A developer runs
  it when a version in `libman.json` changes, and commits the result. CI restores NuGet packages and
  tools; it does not fetch browser libraries.
- **Tooling availability:** Library Manager is available through Visual Studio's own tooling and, outside
  the IDE, through the `Microsoft.Web.LibraryManager.Cli` tool — so the update action is not
  IDE-dependent.

`libman.json` does not exist in the repository today (`git ls-files | grep -c 'libman.json'` → `0`,
appendix A.4), and it is not created by this assessment.

### 9.4 The one `libman.json` `libman.json` contract — provider ids, destinations and file lists — provider ids, destinations and file lists

A version alone does not make a library acquirable. Library Manager resolves each entry as
`<provider-library-id>@<version>`, and **the provider's id is not always the package's familiar name**, so
an entry that names the library the way the NuGet package or the npm package names it can fail to resolve
even though the version is correct. That is a restore-time failure, not a build-time one, so it surfaces
as a developer or reviewer being unable to obtain the assets rather than as a compiler error. The choice of
Bootstrap script file decides whether one of the application's live controls works, and the file names
differ per library. The manifest is therefore given here in full, as the target-state content of
`src/MvcMusicStore/libman.json` (§12.2).

**This is the only manifest in this document.** There is one `libman.json`, it has **four `libraries`
entries** and it selects **five files**, and no other section restates its `files` lists: §9.5 reconciles
these same five selections against the provider and the served paths, §9.6 states the order they load in,
and §9.7 carries the provider coordinates and the identifier trap without a second file list. A manifest
quoted twice is two manifests as soon as one copy is edited, and the two would differ in the vendored
payload, in whether source maps ship, and in the served-file count [05](05-aspnet-core-migration-approach.md)
§8.1.1 publishes.

```json
{
  "version": "1.0",
  "defaultProvider": "cdnjs",
  "libraries": [
    {
      "library": "jquery@3.7.1",
      "destination": "wwwroot/lib/jquery/",
      "files": [ "jquery.js", "jquery.min.js", "jquery.min.map" ]
    },
    {
      "library": "jquery-validate@1.21.0",
      "destination": "wwwroot/lib/jquery-validation/",
      "files": [
        "jquery.validate.js",
        "jquery.validate.min.js",
        "additional-methods.js",
        "additional-methods.min.js"
      ]
    },
    {
      "library": "jquery-validation-unobtrusive@4.0.0",
      "destination": "wwwroot/lib/jquery-validation-unobtrusive/",
      "files": [ "jquery.validate.unobtrusive.js", "jquery.validate.unobtrusive.min.js" ]
    },
    {
      "library": "bootstrap@5.3.3",
      "destination": "wwwroot/lib/bootstrap/",
      "files": [
        "css/bootstrap.min.css",
        "css/bootstrap.min.css.map",
        "js/bootstrap.bundle.min.js",
        "js/bootstrap.bundle.min.js.map"
      ]
    }
  ]
}
```

**The five files it acquires, named once so nothing downstream has to infer them:**
`wwwroot/lib/jquery/jquery.min.js`, `wwwroot/lib/jquery-validation/jquery.validate.min.js`,
`wwwroot/lib/jquery-validation-unobtrusive/jquery.validate.unobtrusive.min.js`,
`wwwroot/lib/bootstrap/css/bootstrap.min.css` and
`wwwroot/lib/bootstrap/js/bootstrap.bundle.min.js`. Four entries, five files: the three jQuery-family
entries select **one file each** (3) and the Bootstrap entry selects **two** — the stylesheet and the
script bundle (2) — so `3 + 2 = 5`, which is the figure §9.3's third count states and the one
[05](05-aspnet-core-migration-approach.md) §8.1.1 adds to the application's own relocated files to reach
the served payload.

**The five details in it that are decisions rather than transcription.**

- **`jquery-validate`, not `jquery-validation`.** cdnjs publishes jQuery Validation under the id
  `jquery-validate`; there is no cdnjs library called `jquery-validation`, so an entry naming it resolves
  to nothing and Library Manager reports the unresolved-library error `LIB002` rather than silently falling
  back. Verified directly against the provider's own index rather than inferred:
  `https://api.cdnjs.com/libraries/jquery-validate/1.21.0` returns the library with
  `jquery.validate.min.js` among its files, and `https://api.cdnjs.com/libraries/jquery-validation` returns
  HTTP 404. `jquery-validation` is the library's **upstream and npm** name and is the name every other
  section of this document and of [05](05-aspnet-core-migration-approach.md) uses for it; `jquery-validate`
  is the **CDNJS** identifier and appears only where a provider identifier is required, which is this
  manifest. Neither name is wrong — they belong to different registries — so the rule this document follows
  is to say which registry a name belongs to at every point it names the library. The **unobtrusive**
  adapter is the opposite case — its cdnjs id *is* `jquery-validation-unobtrusive`, which is why the two
  neighbouring entries spell the library name two different ways and why neither can be derived from the
  other by pattern.- **The four entries are exactly §9.2's four retained libraries, at exactly §9.2's versions** — jQuery
  `3.7.1`, jQuery Validation `1.21.0`, jQuery Validation Unobtrusive `4.0.0` and Bootstrap `5.3.3`.
  Nothing is acquired here that §9.2 did not retain, and the two removed rows have no entry at all.
- **The provider's identifier for Bootstrap is `bootstrap`.** `twitter-bootstrap` is the provider's
  **legacy alias** for the same library and still resolves; `bootstrap` is the current identifier and is
  what the manifest uses. A manifest naming the alias is not wrong, but it will confuse the next reader
  into thinking two libraries are in play.
- **Every entry states its `destination` explicitly, and one of them differs from its library id.**
  Omitting `destination` makes LibMan derive the folder from the library id, which would place jQuery
  Validation at `wwwroot/lib/jquery-validate/` — a path that matches neither the ASP.NET Core project
  convention nor the folder the views' script references use. The destinations above are the contract:
  `jquery-validation/` for the `jquery-validate` library, and a folder matching the id for the other
  three. **The provider's id and the destination folder are separate values and only the id must match
  cdnjs**, which is why §9.6's load order and §12.2's committed-file list quote `jquery-validation` in the
  paths while this entry spells `jquery-validate` in the identifier. The view-side script references that
  consume these paths are [05](05-aspnet-core-migration-approach.md)'s to write; this document fixes the
  paths they must target.
- **Every entry states its `files` explicitly, and no source map is selected.** The default is to vendor
  the library's *entire* published file set — for `bootstrap@5.3.3` that is over a hundred files, including
  RTL builds, unminified builds, per-component ESM bundles and their maps, all of which would then be
  **committed**, because §9.3 commits the output. Explicit selection is what keeps a committed vendored
  directory reviewable. The listing rule is narrow: **a file is vendored only if it is served.** Source
  maps are therefore excluded, and the decision is recorded because it is visible in the served payload — a
  `.map` file is only useful alongside the unminified source it references, so including `jquery.min.map`
  or `bootstrap.min.css.map` without also committing the unminified originals buys nothing while adding
  files to the deployment. Browsers request a map **only when developer tools are open**, so its absence
  costs a console note and nothing else, and a developer who needs to step through a library can point a
  working copy of the manifest at the unminified file for the duration. One spelling trap belongs with that
  temporary case: jQuery's map is published as **`jquery.min.map`**, not `jquery.min.js.map`, at this
  version, so a line copied from another project's manifest — where the `.min.js.map` convention is
  near-universal — fails to restore on exactly that one line.
- **`bootstrap.bundle.min.js` is selected, not `bootstrap.min.js`, because the bundle includes Popper and
  the plain file does not.** This is not a size preference: the application has a **live dropdown** — the
  genre menu at [src/MVC5/MvcMusicStore/Views/Store/GenreMenu.cshtml:3-18] — and Bootstrap 5 positions
  dropdown menus with Popper. Selecting `bootstrap.min.js` produces a page that renders correctly and a
  genre menu that does not open, with a console error as the only evidence. It is the same silent-failure
  shape as the data-attribute rename that [05](05-aspnet-core-migration-approach.md) §8.5.3 records, and
  the two would be diagnosed together. Selecting the bundle is also why **no separate Popper entry is
  required**, so the manifest stays at four entries.

**How a reviewer checks the manifest without running it.** Each entry's id and version resolve at
`https://api.cdnjs.com/libraries/<id>/<version>`, and the response's `files` array must contain every name
in that entry's `files` list — five names in total across the four entries. All four entries above were
checked that way; `Respond` and `Modernizr` have no entry because §9.2 removes them.

---

### 9.5 The manifest reconciled — provider, destination, file and served path

§9.4 holds the manifest; this section reconciles the **same** four entries and five selections against the
provider that serves them and against the URLs the browser then requests, and adds nothing to the file
list. It carries no second copy of the JSON, for the reason §9.4 states: a manifest quoted twice is two
manifests as soon as one copy is edited.

**One property of the selected stylesheet is invisible in §9.4's manifest and must not be "fixed" by adding
files.** `css/bootstrap.min.css` carries its icon-like assets **inside the stylesheet**, as
`data:image/svg+xml` background images rather than as separate files. Two of them are adopted by the port:
the navbar toggler's glyph, which the library exposes as a custom property consumed by
`.navbar-toggler-icon`, and the checked and indeterminate marks of `.form-check-input` — both mapped in
[05](05-aspnet-core-migration-approach.md) §8.5.2 and §8.5.3. **No entry acquires them, and none is
needed**, so §9.4's manifest stays at four entries and the file count below does not move. What they do
require is an image content-security-policy that admits `data:`, which
[06 §10.2](06-azure-hosting-recommendations.md) owns and decides; the alternative — self-hosting the two
glyphs as files under `wwwroot/images/` and overriding the library's own values — would add entries to §9.4
and is rejected there, not here.

**The inventory the manifest generates: five files, and the arithmetic that produces the five.** The entry
count and the file count are different numbers, which is why both are stated rather than one being quoted
loosely: the three jQuery-family entries select **one file each** (3) and the Bootstrap entry selects
**two**, the stylesheet and the script bundle (2), so `1 + 1 + 1 + 2 = 5` from **four** entries. Those five
files are the committed output §9.3 requires, they are the whole of the `wwwroot/lib/**` row in §12.2's
map, and five is the figure [05](05-aspnet-core-migration-approach.md) §8.1.1 adds to the application's own
relocated files when it states the served payload. Every row below is one file — its committed path, the
entry that selects it, and the URL the browser then requests. The served path is the destination with
`wwwroot` removed, because `wwwroot` is the web root and never appears in a URL, which is the distinction
[05](05-aspnet-core-migration-approach.md) §12.7 asserts on.

| Committed file | Selected by | Served path |
| --- | --- | --- |
| `src/MvcMusicStore/wwwroot/lib/jquery/jquery.min.js` | `jquery@3.7.1` | `/lib/jquery/jquery.min.js` |
| `src/MvcMusicStore/wwwroot/lib/jquery-validation/jquery.validate.min.js` | `jquery-validate@1.21.0` — the cdnjs id for library **jquery-validation** | `/lib/jquery-validation/jquery.validate.min.js` |
| `src/MvcMusicStore/wwwroot/lib/jquery-validation-unobtrusive/jquery.validate.unobtrusive.min.js` | `jquery-validation-unobtrusive@4.0.0` | `/lib/jquery-validation-unobtrusive/jquery.validate.unobtrusive.min.js` |
| `src/MvcMusicStore/wwwroot/lib/bootstrap/css/bootstrap.min.css` | `bootstrap@5.3.3` | `/lib/bootstrap/css/bootstrap.min.css` |
| `src/MvcMusicStore/wwwroot/lib/bootstrap/js/bootstrap.bundle.min.js` | `bootstrap@5.3.3` | `/lib/bootstrap/js/bootstrap.bundle.min.js` |

| Provider identifier and version | Destination | `files` | Served path |
| --- | --- | --- | --- |
| `jquery@3.7.1` | `wwwroot/lib/jquery/` | `jquery.min.js` | `/lib/jquery/jquery.min.js` |
| `jquery-validate@1.21.0` — library **jquery-validation** | `wwwroot/lib/jquery-validation/` | `jquery.validate.min.js` | `/lib/jquery-validation/jquery.validate.min.js` |
| `jquery-validation-unobtrusive@4.0.0` | `wwwroot/lib/jquery-validation-unobtrusive/` | `jquery.validate.unobtrusive.min.js` | `/lib/jquery-validation-unobtrusive/jquery.validate.unobtrusive.min.js` |
| `bootstrap@5.3.3` | `wwwroot/lib/bootstrap/` | `css/bootstrap.min.css`, `js/bootstrap.bundle.min.js` | `/lib/bootstrap/css/bootstrap.min.css`, `/lib/bootstrap/js/bootstrap.bundle.min.js` |

The served path is the destination with `wwwroot` removed, because `wwwroot` is the web root and never
appears in a URL — the distinction [05](05-aspnet-core-migration-approach.md) §12.7 asserts on. Row two is
the only row where the identifier and the destination folder differ, deliberately and for the reason §9.4's
`destination` decision gives; every other row spells the library the same way in its identifier, its
destination and its URL.

**Because the output is committed, a provider failure cannot break a build or a deployment.** A `LIB002`
resolution error can only occur during a developer-initiated `libman restore`, which happens when a
version in §9.4's manifest changes — never in CI and never at deployment (§9.3). That is the whole point of
committing the restored files, and it is why an unavailable CDN is an inconvenience here rather than an
outage.

### 9.6 The load order the manifest produces

**The manifest is §9.4's**, and this section adds no file to it. That manifest fixes *what* is acquired and
*where* it lands; the order the files load in is the consequence, and it is a **correctness** property
because three of the four have a hard dependency on another.
[05](05-aspnet-core-migration-approach.md) §8.1.3 renders these as tag-helper elements and owns their
placement in the views; this table is the order it renders them in, over exactly the five files §9.4
selects plus the application's own stylesheet and cart script.

| Position | Files, in order |
| --- | --- |
| **`<head>`** | `wwwroot/lib/bootstrap/css/bootstrap.min.css`, then the application's own `wwwroot/css/site.css` |
| **End of `<body>`, in the layout** | `wwwroot/lib/jquery/jquery.min.js`, then `wwwroot/lib/bootstrap/js/bootstrap.bundle.min.js`, then the page's scripts section |
| **Inside the page's scripts section** | On the form pages: `wwwroot/lib/jquery-validation/jquery.validate.min.js` — jquery-validation, acquired under the CDNJS identifier `jquery-validate` while the destination folder keeps the library's own name (§9.4) — then `wwwroot/lib/jquery-validation-unobtrusive/jquery.validate.unobtrusive.min.js`, whose identifier and folder are the same string. On the cart page: `wwwroot/js/shopping-cart.js` |

Four reasons, one per ordering constraint:

- **Application CSS loads after the framework's**, because it overrides it. That is not hypothetical: the
  application's stylesheet exists largely to restyle the navbar, and
  [05 §8.5.5](05-aspnet-core-migration-approach.md) records those rules.
- **The validation plugin loads before the unobtrusive adapter**, which extends it, and both load after
  jQuery, which both require.
- **The cart script loads after jQuery**, for the reason
  [05](05-aspnet-core-migration-approach.md) §8.1.2 gives: it runs at parse time.
- **jQuery survives the upgrade only because `jquery-validation-unobtrusive` requires it.** Bootstrap 5
  dropped its own jQuery dependency, and the application's own script is 29 lines. Stating this makes the
  dependency's justification explicit and reviewable: if client-side validation were ever re-expressed
  without the unobtrusive adapter, jQuery would leave with it, and both the pin in §9.2 and two rows of
  the table above would go.

---

### 9.7 The exact cdnjs coordinates, and the identifier trap that breaks a restore

A version alone is not enough to write `libman.json`: each entry needs the **provider's own library
identifier**, and for one of the four that identifier is **not** the name the library is known by anywhere
else. This section carries the coordinates and their provenance only — **the `files` selections and the
destinations are §9.4's manifest and §9.5's reconciliation, and are deliberately not repeated here**, so
there is one place a file list can be edited rather than three.

| cdnjs library id | Version | The library it identifies |
| --- | --- | --- |
| `jquery` | `3.7.1` | jQuery |
| `jquery-validate` | `1.21.0` | jQuery Validation — **upstream and npm name `jquery-validation`**, the one identifier of the four that differs from the library's own name |
| `jquery-validation-unobtrusive` | `4.0.0` | The unobtrusive validation adapter; identifier and library name are the same string |
| `bootstrap` | `5.3.3` | Bootstrap — the current identifier, not the legacy `twitter-bootstrap` alias |

Verified against the provider's own library pages
[cdnjs, *jquery*, <https://cdnjs.com/libraries/jquery> — verified 2026-08-28],
[cdnjs, *jquery-validate*, <https://cdnjs.com/libraries/jquery-validate> — verified 2026-08-28],
[cdnjs, *jquery-validation-unobtrusive*, <https://cdnjs.com/libraries/jquery-validation-unobtrusive> —
verified 2026-08-28] and
[cdnjs, *bootstrap*, <https://cdnjs.com/libraries/bootstrap> — verified 2026-08-28].

**The trap, recorded because it is exactly what breaks a restore.** jQuery Validation is published on
cdnjs as **`jquery-validate`**. `jquery-validation` is its **npm** package name, and cdnjs does not
publish it under that name — a `libman.json` entry naming `jquery-validation` against the `cdnjs` provider
does not resolve, and Library Manager reports the unresolved-library error `LIB002` rather than silently
falling back. The destination folder is deliberately still `wwwroot/lib/jquery-validation/`, because that
is the conventional path the views reference; **the provider's id and the destination folder are separate
values and only the id must match cdnjs.** If the `unpkg` or `jsdelivr` provider is ever selected instead,
the identifier reverts to the npm name — which is the same trap in the other direction, and the reason
this table names its provider in the heading rather than assuming one.

The libraries are pinned to exact versions, so a newer release appearing on cdnjs changes nothing until a
developer edits `libman.json` and re-runs `libman restore`; and because the restored files are committed
(§9.3), neither a build nor a deployment ever contacts cdnjs at all.

---

## 10. Capabilities that need no package

Stated explicitly so that nobody adds a package for something already present. Every capability below is
either in the shared framework or supplied by the platform.

### 10.1 In the shared framework

Dependency injection; configuration and options; structured logging through `ILogger`; the health-checks
service, middleware and endpoint mapping — **including the one check the application registers, which is
§10.4**; session; static-file serving with version-appended cache busting; HTTPS redirection and HSTS;
anti-forgery; and JSON serialization through `System.Text.Json`.

Three of these are worth naming as *net-new capability* rather than migrations, because the repository has
none of them today: there is no logging of any kind, no health endpoint and no security response header
anywhere in any edition. Those absences are findings owned by [08](08-technical-debt-register.md) F-08-13
and [11](11-cloud-readiness-assessment.md). All three are acquired with **no package reference at all** —
logging, the security headers **and** the health endpoints, including the one check the application
registers, for the reason §10.4 states in full. This section's claim therefore carries no qualification:
nothing in it is bought with a pin.

### 10.2 Supplied by the platform

- **Secrets** arrive as platform configuration references surfaced through `IConfiguration`, so **no
  secret-store client library is referenced**. The mechanism — which store, which reference syntax, which
  identity — is [06](06-azure-hosting-recommendations.md)'s.
- **Telemetry** is platform-collected. The observability approach is owned by
  [06](06-azure-hosting-recommendations.md) and is not restated here. The consequence for *this* document
  is narrow and is the only thing it asserts: **no in-process telemetry SDK is pinned**, so no
  OpenTelemetry or instrumentation package appears in §7.2. If custom instrumentation is ever required, it
  is a scoped addition with packages pinned at the time it is approved.
- **Audit delivery is that same route and there is no second one.** Every audit record — the provisioning
  command's, the two Identity seams', the authorization result handler's and the error record's — is
  written through `ILogger` and collected by App Service's Application Insights
  **auto-instrumentation**. That is one route end to end, so **three artifacts an audit design often
  acquires are withdrawn on scope grounds: a dedicated audit table, a telemetry exporter, and an
  immutable audit store.** None of the three is in the frozen artifact set (§12.1), none has a row on
  §12.2's map, none has a schema object in either migration set, and none has a pin in §7.2 — and
  §12.2.1's fourth co-location states the same thing from the artifact side, so the package graph and the
  map cannot disagree about it. The record's field set is
  [09 §6.8.1](09-security-assessment.md)'s and the sink and its retention are
  [06](06-azure-hosting-recommendations.md)'s; what this document owns is the absence of a package and the
  absence of a row.

### 10.3 Data protection — the stack needs no package; persisting its key ring does

This one is easy to get wrong in both directions, so it is stated precisely.

- **The data-protection stack itself needs no package.** It is part of the shared framework and is active
  by default; anti-forgery tokens and authentication cookies already depend on it.
- **Persisting its key ring to a durable, shared store *does* need a package.** That is why
  `Microsoft.AspNetCore.DataProtection.EntityFrameworkCore` `8.0.30` is in §7.2 — it is a key-repository
  implementation, not the stack.

The reason a durable key ring matters at all is a current-state fact: **no `<machineKey>` is declared in
any of the 15 application configuration files**, so no key material is shared between instances today
([11](11-cloud-readiness-assessment.md) §3.2 owns the finding and its verification). Where the ring
lives, how it is protected at rest, how it rotates and how it is isolated between deployment slots are
all [06](06-azure-hosting-recommendations.md)'s decisions. **This document owns only the pin.**

### 10.4 Health checks — the whole surface needs no package, and one earlier pin is retired

**The health-checks stack needs no package, and neither does the one check the application registers.**
`AddHealthChecks()`, the health-check middleware, `MapHealthChecks` with its `HealthCheckOptions` and the
tag-based `Predicate` that separates liveness from readiness, the `IHealthCheck` interface, and the
`HealthStatus` and `HealthCheckResult` result types are all in the shared framework.
[05 §2.4](05-aspnet-core-migration-approach.md) registers exactly one check and maps two endpoints against
it — a liveness endpoint whose predicate matches no check, and a readiness endpoint filtered to the
`ready` tag — and every type in that composition resolves from the framework. So does the
`CancellationTokenSource` the check links to the request token in order to bound itself.

**An earlier version of this document pinned `Microsoft.Extensions.Diagnostics.HealthChecks.EntityFrameworkCore`
at `8.0.30`, and that pin is retired, because its sole consumer no longer exists.** It supplied
`AddDbContextCheck<TContext>()`, which 05 registered. 05 now registers an application-owned `IHealthCheck`
instead, for two reasons that are its own to state and are recorded here only as the cause of the
retirement: the extension returns the provider exception on a failed probe — `HealthCheckResult` carries
an `Exception` property and the framework's health-check service logs it — which defeats 05's undertaking
that no raw provider message reaches any sink; and the extension **takes no timeout parameter**, so it
cannot deliver the bounded readiness budget [06 §9.3](06-azure-hosting-recommendations.md) states as the
platform contract. A package with no consumer is not a smaller cost than a package with one; it is a
restore-time dependency, a servicing obligation and a supply-chain surface acquired for nothing, so it
goes.

**Two things follow, and both are checkable.** The web application's own `PackageReference` count drops
from seven to **six**, and those six are §7.2's `Microsoft.EntityFrameworkCore`,
`Microsoft.EntityFrameworkCore.SqlServer`, `Microsoft.EntityFrameworkCore.Design`,
`Microsoft.AspNetCore.Identity.EntityFrameworkCore`, `Microsoft.Extensions.Caching.SqlServer` and
`Microsoft.AspNetCore.DataProtection.EntityFrameworkCore` — every other row of that table is declared by
the test project or the operator tool, as its own text says. And the retirement moves this document back into
alignment with the frozen plan rather than away from it: the successor-package set the Agent Action Plan
fixes in its §0.5.2 does not contain this identifier, and its own list of capabilities that need no
package names health checks explicitly. The pin was an addition beyond that set, justified by a
registration 05 has since replaced, and removing it leaves §7.2 carrying exactly the pins the plan does.

---

## 11. Tooling posture

Two tools bear on a .NET 8 migration strategy, and both need a statement — one because it must not be
mistaken for an application dependency, the other because it must not be recommended without a
qualification.

### 11.1 AppCAT is an assessment tool, not an application dependency

Azure Migrate application and code assessment for .NET (AppCAT) is the mechanized-evidence tool for this
kind of migration: it performs static analysis of source, configuration and binaries and supports effort
estimation. Two facts about it matter to *this* document and no more: it is installed as a **global .NET
tool**, `dotnet tool install -g dotnet-appcat`, and it is invoked as `appcat analyze <application-path>`,
producing a report in HTML, JSON or CSV
[Microsoft Learn, *Azure Migrate application and code assessment for .NET*,
<https://learn.microsoft.com/azure/migrate/appcat/dotnet> — verified 2026-08-28].

**It appears in no package table in this document, and that is deliberate.** It is not referenced by the
ported application, it is not restored as part of a build, and it does not belong in
`.config/dotnet-tools.json` alongside `dotnet-ef` and `dotnet-sql-cache` — those two are *required to
deploy the application*, whereas AppCAT is run by an engineer to produce a report. Its own installation
form makes the same point: it is a **global** tool, outside the repository's local tool manifest entirely.
Adding it to the application's tool manifest would make an assessment artifact a deployment prerequisite.

**When it runs is the roadmap's decision, and the gate has a name.** AppCAT is scheduled by
[03 — Modernization Roadmap](03-modernization-roadmap.md) as the **AppCAT static assessment** gate within
**W2**; this document neither sequences it nor duplicates its entry and exit evidence. Where its output is
consumed — as corroborating evidence for the effort model — is
[07](07-effort-risks-sequencing.md)'s.

### 11.2 The .NET Upgrade Assistant is deprecated, and must not be presented otherwise

The .NET Upgrade Assistant has been **officially deprecated**. Microsoft's own overview page states it
plainly — *".NET Upgrade Assistant is officially deprecated. Use the GitHub Copilot modernization chat
agent instead, which is included with Visual Studio 2026 and Visual Studio 2022 17.14.16 or later"*
[Microsoft Learn, *.NET Upgrade Assistant overview*,
<https://learn.microsoft.com/dotnet/core/porting/upgrade-assistant-overview> — verified 2026-08-28]. It is
recorded here because a strategy document is exactly where a reader expects to find "run the Upgrade
Assistant" as step one, and a plan that recommended it without recording its deprecation would be pointing
at unsupported tooling.

The practical position this document takes: **the conversion specified in section 5 is defined by its
outcome, not by a tool.** Every element of the target project file, the four manifests and the 28
package outcomes is specified explicitly here, so the conversion can be performed by hand or by whatever
assistive tooling is current and supported at approval time — and the result is checkable against this
document either way. That is the reason for specifying the target state at this level of detail rather
than delegating it to a tool whose support status is itself a moving target.

---

## 12. The future application map — the portion this strategy owns

### 12.1 Scope of this map

Every target artifact below either **maps to a legacy source file** or is **marked net-new**; there is no
third category, and an unmapped target would be a defect in this document.

This map covers **project format, target framework, dependencies, tooling manifests and solution
structure** only. The **code-level transitions** — the composition root, dependency injection,
configuration, Identity, EF Core contexts and migrations, controllers, view models, view components, the
29 views and the relocation of the static assets — are owned by
[05](05-aspnet-core-migration-approach.md) and are **not duplicated here**. Where a row below names a
target file whose *contents* 05 designs, this document claims only its project-structural facts: that it
exists, where it sits, and what it derives from.

**The artifact set is governed, not invented here.** The authoritative future application map is the
transformation map in the Agent Action Plan (§0.4.1); every row of §12.2 resolves to a row of it, or — in
the single case of the per-project lockfile — to the dependency-pinning requirement the plan states in
§0.5.2, or — in the cases the short record below names one by one — to obligations the plan states in
§0.3.1 and §0.3.2 and makes acceptance criteria in §0.11.2.
No row adds an artifact the plan does not require, because expanding the target artifact set is a plan
amendment rather than a strategy decision. That rule is unchanged and it governs every row in §12.2; the
additions are named, sanctioned and notified in one place rather than taken quietly.
The practical consequence is worth stating, because it is the
kind of file a reader expects to see listed and will not find: **local run configuration — launch
profiles, developer ports, the environment a developer's own `dotnet run` picks up — is a developer
concern, not part of the mapped artifact set.** It is created, changed and discarded per workstation, it
is not a build input, it is not a deployment input, and nothing in this strategy depends on its contents.
The IIS Express settings it would loosely correspond to are dropped outright (§5.4), and the deployed
hosting model is [06](06-azure-hosting-recommendations.md)'s.

**The short record — what the plan's §0.4.1 map does not list, the sanction for each addition, and the
rows it takes.** Recorded in the form deliverable
[05](05-aspnet-core-migration-approach.md) §5.6 and §9.3 use, and for the same reason: the plan is frozen,
nothing here edits it, and an artifact the plan's own obligations require but its map does not list has to
be **notified**, once and by name, rather than appearing in an implementation as an unexplained addition to
a governed set. This is a record **beside** the map and not a second map: §12.2 below is the map, every
artifact named here is a row of it, and nothing here is a row of its own. The six numbered records keep the
ids they were filed under — **2, 6, 7, 8, 10 and 11**, records 3, 4, 5 and 9 being §7's pin records — so a
reference to "record 7" resolves to this table. **None of the artifacts below is created by this
assessment.**

| Record | Rows | The artifacts | Why the plan's map does not list them, what is asked of the plan owner, and the sanction |
| --- | ---: | --- | --- |
| — | 1 | the four `packages.lock.json` files | Sanctioned by §6.4's locked-mode restore, which the plan states in §0.5.2 as a **pinning requirement** rather than as a file. One row naming one file per project, because §6.4 is per project and there are four projects (§5.6) |
| **2** | 3 | `src/MvcMusicStore.Contracts.Tests/MvcMusicStore.Contracts.Tests.csproj`, and one `xunit.runner.json` per test project | The plan lists **one** test project — *"`MvcMusicStore.Tests.csproj` and its test classes"* — and no runner configuration. A project referencing the ported web application cannot be built before that project exists, so the plan's own ordering is unexecutable as mapped: §0.3.1 requires test authoring to *precede* the port and §0.11.2 makes the pre-port baseline an acceptance criterion. The reference-free split is what makes that ordering executable, and the collection-parallelism switch the required isolation needs is an **assembly-level file** the map carries for neither project. **Asked:** replace the single test-project row with the two projects, each with its lockfile and its `xunit.runner.json`; record that the contracts project carries **no** reference to the web application; and record that each project declares the §0.5.2 execution pins **directly**, because build and analyzer assets do not cross a project reference (§12.3). That clause adds no pin — it fixes the declaration site of pins the plan already carries. The contracts project's own lockfile is a file of the lockfile row above rather than a row of this record |
| **6** | 1 | `src/MvcMusicStore/ApplicationServices.cs` | The plan gives the web project a `Program.cs` that *"absorbs 6 startup files"* and separately maps an operator console that *"resolves `UserManager`/`RoleManager` from the container"* (§0.3.2, which also refuses direct SQL as a substitute because it cannot produce a valid Identity password hash). **No mapped artifact lets a second process reach that container** — a composition root of top-level statements exposes nothing callable — so the console would carry its own registrations, and duplicated registrations are the one form of duplication that **compiles and then drifts in silence** (§12.9). **Asked:** add the file as CREATE / net-new, owned by the web project, and record that `Program.cs` composes the application **through** it so there is one registration path rather than one per process. The registrations' contents are [05 §6.1](05-aspnet-core-migration-approach.md)'s and [05 §4.5](05-aspnet-core-migration-approach.md)'s; this document adds the file, the type, the signature and the single-path rule |
| **7** | 4 | `Fixtures/fixture-data.json`; the ten `Fixtures/ModelOverrides/` files; `Fixtures/seed-expected.json`; `Fixtures/baseline-reference.json`, all under `src/MvcMusicStore.Contracts.Tests/` | The plan lists the test project's code files and **no data file of any kind**, while §0.3.1 requires one dataset carrying *"the same catalog rows"* with asserted row counts, §0.3.2 requires a generated-schema diff that must pass before any data is loaded, and §0.6.4 requires the seeding guard to be exercised. Every one of those obligations is discharged by a run that **reads a committed file**: a dataset carried in two loaders' code drifts, so a cross-baseline comparison establishes nothing; an asserted row count needs a figure that exists outside the loader that produced the rows; a diff gate can only be shown to refuse if a committed input diverges the target schema on purpose, and the switch that does it ([05 §5.7](05-aspnet-core-migration-approach.md)) takes a **path**; and an oracle the code under test derives proves only self-consistency. **Asked:** add all thirteen paths as CREATE / net-new, each declared **once** in the contracts project and copied to both test assemblies' output, with the fixture directory recorded as the single home for committed test inputs. Thirteen rather than twelve because the override folder holds **ten** files: the nine divergence dimensions plus the narrowing case row 77 `77b` re-diffs, whose switch also takes a path. This map owns each file's existence, its one path and its copy behaviour; [05](05-aspnet-core-migration-approach.md) §12.3, §5.1, §12.9 and §12.10 own every value inside them |
| **8** | 2 | `Collections/*.cs` in each of the two test projects | The plan lists `LegacyBaselineFixture.cs` and `CoreApplicationFixture.cs` and **nothing that attaches either to a test class**, while §0.3.1 requires per-class isolation against a freshly provisioned database and §0.11.2 makes isolation an acceptance criterion. A fixture no collection definition names is never instantiated — the failure surfaces inside an assertion as a missing context rather than as a wiring error — and the grouping is unimplementable without it, because a test class carrying no `[Collection]` **is its own collection**. Two sets are needed because `[Collection]` names resolve within the assembly being run. **Asked:** add both paths as CREATE / net-new, each class `public`, carrying `ICollectionFixture<TFixture>` for its own assembly's fixture and exposing its name as a constant. The group set, the one-database-per-collection rule and the per-test reset are [05 §12.7](05-aspnet-core-migration-approach.md)'s |
| **10** | 1 | `src/MvcMusicStore/Resources/ApplicationMessages.resx` | The plan carries no resource file and the legacy application has none, but §0.6.3 requires a cart migration that fails after retries to *"still complete the sign-in and surface a non-blocking notice"*, §0.11.2 makes that path a required test, and [05 §12.9](05-aspnet-core-migration-approach.md) row 37 asserts the rendered notice byte-equal to a **named** entry, `CartMigrationNotice`, read from the application. The alternative needs no file and is worthless: a literal in the view and the same literal retyped in the assertion tests the transcription and keeps passing after the view alone is edited. **Asked:** add the path as CREATE / net-new, as an `EmbeddedResource` in the web project, with user-facing strings a test asserts on living there rather than as view literals. **This record adds one file and no resource strategy** — localization is not in the plan and is not proposed here |
| **11** | 2 | `src/MvcMusicStore.Contracts.Tests/DeployedEndpointFixture.cs`, `src/MvcMusicStore.Contracts.Tests/DeployedEndpointTests.cs` | The plan lists **two** fixtures and §0.3.1 describes both as hosting an application and provisioning a database, while §0.11.2 requires *"a deployment health check"* among the validated behaviours and §0.3.2 requires HTTPS enforcement and HSTS at the edge, which [05 §12.9](05-aspnet-core-migration-approach.md) row 47 discharges against a **deployed** instance. Neither mapped fixture can run that case and neither should be made to: a smoke check that provisioned a database would be asserting about something other than the deployment, and a fixture attaches by collection, so borrowing one means paying its provisioning on every deployment verification. Leaving the case unmapped is worse — a required assertion with **no runnable context** reads as covered and executes nowhere. **Asked:** add both paths as CREATE / net-new in the contracts project, and record that this fixture **provisions nothing and hosts nothing** — it consumes a base address and disables redirect following. Its collection definition is a member of record 8's widened `Collections/*.cs` row rather than a row of its own |
| — | 4 | `src/MvcMusicStore/wwwroot/lib/**`, `src/MvcMusicStore/wwwroot/js/shopping-cart.js`, `deploy/sql/SecurityAuditLog.sql`, `src/MvcMusicStore/Models/CartAlias.cs` | Artifacts a **consuming** deliverable has decided since the plan was frozen and given no path: the committed output of the acquisition mechanism §9.3 fixes; the externalized cart script [06 §10.2](06-azure-hosting-recommendations.md) requires under `script-src 'self'` and [05 §7.4](05-aspnet-core-migration-approach.md)'s token header travels on, whose source is a Razor view rather than an asset file — a derivation no glob over the asset directories would express; the durable audit table's DDL, which [06 §9.5](06-azure-hosting-recommendations.md) states in full while placing it nowhere; and the cart-handoff alias **entity type**, which [05 §4.13](05-aspnet-core-migration-approach.md) declares as a mapped member of `MusicStoreEntities` and states in full — key, columns, widths, state column and index — while placing it nowhere, exactly as 06 does with the audit DDL. It is the one of the four that is a **CLR type** rather than a data or script file, which is why leaving it unlisted did not merely omit a path: this map forbids a public type with no containing file, so its absence contradicted the closure claim §12.2's count paragraph makes. Each row states the placement rule it follows, so a later operator script has a predictable home |
| — | 15 | the fifteen designed-type rows — **forty-one files**: one under `Health/`, two under `Diagnostics/`, four under `Identity/`, four under `Security/`, **twenty-one over three rows under `Configuration/`**, and **nine over one row under `Binding/` and `Services/`** | The same category one level down: each is a **type** a design written after the plan requires — the readiness check, the exception-record writer, the three authenticated diagnostic operations, the credential-verification and Identity audit seams, the authorization result handler, the Content-Security-Policy holder, the pseudonymization service, the bound options types, the report collector's two files, the readiness verdict and its two loops, the key-ring bootstrap and the two composition-provided signals. **None appears in the plan's 27 groups**, because that map was written before those designs were. Eleven rows are one file each; three rows carry `Configuration/`, two of them one file and the third the nineteen options types [05 §3.3](05-aspnet-core-migration-approach.md)'s section table names; and one row carries nine files that land in `Binding/` and `Services/`, folders the plan's own map already establishes, so the only thing left to fix for them is which of the two each sits in. Each row cites the deliverable that requires its type, and §12.2.1 states the placement rules those rows follow. **Asked:** add the forty-one paths as CREATE / net-new in the web project, with no project of their own |
| **Total** | **33** | — | The additions §12.2's count adds to its 36 authoritative-map rows |

Every row of that record is **recorded and added rather than deferred**, for one reason stated once: each
addition is the artifact without which a sequence the plan itself fixes — the pre-port baseline, the
reachable container, the diff gate, the isolation grouping, the asserted notice, the deployed check —
appears compliant on the map and is unexecutable in practice. The governance rule above is unchanged and
still refuses every unmapped artifact that is not named here.

The six numbered records are the only amendments this document requests to the plan's **§0.4.1 artifact
map**, and each is a plan amendment for the plan owner rather than a strategy decision taken silently.
Three other hand-offs to the plan owner exist and are deliberately kept out of this map, because none is an
artifact: §7.5, §7.6 and §7.7 each request one addition to the plan's **§0.5.2 pin set** — the HTML parser,
the fixtures' SQL client and the browser harness — and §7.3 hands over one approval
decision on a pin the plan already carries.

### 12.2 The map

**One row per physical file.** The table below is the map, and it is exhaustive at file granularity: every
target artifact this document, [05](05-aspnet-core-migration-approach.md),
[06](06-azure-hosting-recommendations.md) or [09](09-security-assessment.md) requires appears in it as its
own row, at its own path. There is no second table of target files, no "and its classes", and no annex
of designed files the map omits — §12.2.1 carries rules and no inventory. **A target file with no row
here is not in approved scope; a row with no owner would be a defect in this document.**

**The five exceptions to one-row-per-file are named, bounded and registered rather than implied.** Five
rows are **families** rather than single files, because their members are emitted by a generator or
derived from another deliverable's design and no document fixes their individual names. Each family row
carries a glob, and §12.2.1 publishes that glob's membership rule, its owning deliverable and its
cardinality wherever an owner fixes one. A file that does not satisfy its family's published rule is not a
member of it and therefore has no row — which is what keeps a glob from becoming an unbounded wildcard.

**What the fourth column claims, and what it does not.** This map covers project format, target framework,
dependencies, tooling manifests and solution structure, so the cell states the row's *project-structural*
facts: that the file exists, where it sits, what build item it is, and what it derives from. The file's
**contents** belong to the deliverable the cell names, and are cited rather than restated.

| Target file | Transformation | Source | What this map fixes about it |
| --- | --- | --- | --- |
| `MvcMusicStore.sln` | CREATE | `src/MVC5/MvcMusicStore.sln`, consolidating four solutions | Single root solution referencing four projects: the web application, the **two** test projects and the single operator console (§5.6). The three project references between them are §12.4 |
| `global.json` | CREATE | **net-new** | SDK band `8.0.400`, `rollForward: latestPatch` (§3, §6.1) |
| `NuGet.config` | CREATE | **net-new** | `<clear />` then nuget.org (§6.2) |
| `.config/dotnet-tools.json` | CREATE | **net-new** | `dotnet-ef` `8.0.30`, `dotnet-sql-cache` `8.0.30` (§6.3) |
| `src/MvcMusicStore/MvcMusicStore.csproj` | CREATE | `src/MVC5/MvcMusicStore/MvcMusicStore.csproj`, `src/MVC5/MvcMusicStore/packages.config`, `src/MVC5/MvcMusicStore/Properties/AssemblyInfo.cs` | SDK-style `Microsoft.NET.Sdk.Web`, `net8.0`, `PackageReference` at the §7.2 pins, absorbed assembly metadata (§5.3), implicit globbing, `MvcBuildViews` and the WebApplication targets import dropped (§5.4) |
| `src/MvcMusicStore/packages.lock.json`, `src/MvcMusicStore.Contracts.Tests/packages.lock.json`, `src/MvcMusicStore.Tests/packages.lock.json`, `tools/provision-admin/packages.lock.json` | CREATE | **net-new** (each generated, then committed) | **Four lockfiles, one per project, each named exactly** — because §6.4 requires one per project and there are four projects (§5.6, §12.4), so a single row naming only the web application's would leave three committed build inputs unmapped. Each carries its own project's locked transitive graph and **CI restores in locked mode** (§6.4). The other three projects' rows below say "its own committed lockfile"; **these are those files**, and they are named here so that the paths exist on the map rather than only as a phrase inside another row |
| `src/MvcMusicStore/libman.json` | CREATE | `src/MVC5/MvcMusicStore/packages.config` — the six content-delivering pins it replaces | `defaultProvider` `cdnjs`; the four retained libraries at the §9.2 versions |
| `src/MvcMusicStore/wwwroot/lib/**` | CREATE | the vendored output of `libman.json` | Committed, so no build or deployment step fetches them (§9.3). The *relocation* of the application's own 27 assets and the casing correction are [05](05-aspnet-core-migration-approach.md)'s |
| `src/MvcMusicStore/Program.cs` | CREATE | `src/MVC5/MvcMusicStore/Global.asax.cs`, `src/MVC5/MvcMusicStore/Startup.cs`, `src/MVC5/MvcMusicStore/App_Start/RouteConfig.cs`, `src/MVC5/MvcMusicStore/App_Start/FilterConfig.cs`, `src/MVC5/MvcMusicStore/App_Start/Startup.App.cs`, `src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs` | **Six startup files collapse into one composition root.** Structurally: compiled by implicit globbing rather than by a `Compile` item (§5.1), and it carries the `public partial class Program` declaration §12.4 requires so the test fixture can name the entry point. What the composition root *does*, registration by registration, is [05 §2](05-aspnet-core-migration-approach.md)'s |
| `src/MvcMusicStore/appsettings.json`, `src/MvcMusicStore/appsettings.Development.json` | CREATE | `src/MVC5/MvcMusicStore/Web.config`, `src/MVC5/MvcMusicStore/Web.Debug.config` | **The connection string and the application's settings move out of XML and into JSON read through `IConfiguration`; the administrator credential [src/MVC5/MvcMusicStore/Web.config:17] is not carried over.** Structurally: content files the Web SDK includes in build and publish output with no explicit item, which is why the conversion drops the `.config` items without adding replacements (§5.1). The keys, their sources and the precedence between them are [05 §3](05-aspnet-core-migration-approach.md)'s |
| `src/MvcMusicStore/Controllers/HomeController.cs`, `StoreController.cs`, `ShoppingCartController.cs`, `CheckoutController.cs`, `StoreManagerController.cs` | CREATE | the same five files under `src/MVC5/MvcMusicStore/Controllers/` | **Ported rather than rewritten:** namespace substitution, constructor injection in place of the field-initialized contexts, `HttpNotFound()` becoming `NotFound()`, explicit loading where EF 6 lazy-loaded, the two `[ChildActionOnly]` actions extracted to view components [src/MVC5/MvcMusicStore/Controllers/StoreController.cs:43], [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:86], and `AddToCart` becoming a token-protected POST. Five source files become five target files, none of them itemized in the project (§5.1). Every transition named here is [05](05-aspnet-core-migration-approach.md)'s to specify |
| `src/MvcMusicStore/Controllers/AccountController.cs` | CREATE | `src/MVC5/MvcMusicStore/Controllers/AccountController.cs` | **Rewritten, not ported** — against ASP.NET Core Identity, with the private `ChallengeResult : HttpUnauthorizedResult` [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:394] and its `ExecuteResult(ControllerContext)` override [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:411] removed or replaced by the framework's challenge flow. Which of those two, and the fate of the external *sign-in* surface, is [05 §8.3](05-aspnet-core-migration-approach.md)'s choice; what this map settles is that the external-login **management** surface survives, because the `RemoveAccountList` component below is on it |
| `src/MvcMusicStore/Models/Album.cs`, `Artist.cs`, `Genre.cs`, `Cart.cs`, `Order.cs`, `OrderDetail.cs` | CREATE | the same six files under `src/MVC5/MvcMusicStore/Models/` | **The six entity classes port largely as they stand**, with one exception that is not a namespace change: `Order.cs` loses `using System.Web.Mvc` [src/MVC5/MvcMusicStore/Models/Order.cs:4] **and** the class-level `[Bind(Include = …)]` [src/MVC5/MvcMusicStore/Models/Order.cs:8] that directive exists to support, whose replacement is the `Binding/` row below. Explicit EF Core mapping wherever EF 6 relied on convention is [05 §4](05-aspnet-core-migration-approach.md)'s |
| `src/MvcMusicStore/Models/CartAlias.cs` | CREATE | **net-new** — the legacy application has no alias of any kind: an anonymous cart's identity lives only in in-process session, as the `Guid.NewGuid()` written to `Session` in `GetCartId` [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:163-179], so there is nothing today that survives a sign-in to be aliased | **A catalog entity type the six ported entity classes above do not include, and it is on this map because it is a mapped CLR type with no containing file anywhere else.** [05 §4.13](05-aspnet-core-migration-approach.md) makes it *"an entity type in `MusicStoreEntities` … mapped in `OnModelCreating`"* whose ordinary reads and inserts *"go through the context"*, and [05 §5.1](05-aspnet-core-migration-approach.md)'s cart-merge object census creates `dbo.CartAlias` as the fifth item of the named post-baseline migration `AddCartMergeConstraints` — so the type exists, and a public type with no file breaks the one property this table guarantees. It takes a file of its own under §12.2.1's one-public-type-per-file rule and sits in `Models/` beside the six ported entity classes above, the folder the authoritative map already establishes for this context's entity types; no new folder arises. **Its key, its columns, the state column, the destination index and every access rule are [05 §4.13](05-aspnet-core-migration-approach.md)'s, the bounded width of its two id columns is [05 §4.3](05-aspnet-core-migration-approach.md)'s cart-key width rule, and the migration that creates the table is [05 §5.1](05-aspnet-core-migration-approach.md)'s** — this row adds a path, a folder and a build fact and asserts nothing about content. Structurally: compiled into the web project by implicit globbing with no `Compile` item (§5.1), no new project and no edge in §12.4's graph, and its `DbSet` declaration on the context of the `Data/` row below is that row's owner's to write |
| `src/MvcMusicStore/Models/ApplicationUser.cs` | CREATE | `src/MVC5/MvcMusicStore/Models/IdentityModels.cs` — the `ApplicationUser : IdentityUser` declaration at [src/MVC5/MvcMusicStore/Models/IdentityModels.cs:6] | **The `IdentityUser` base moves to `Microsoft.AspNetCore.Identity`.** One legacy file becomes two target files — this one and `Data/ApplicationDbContext.cs` below — which is why that source appears twice on this map; the split is stated rather than left to be inferred |
| `src/MvcMusicStore/Models/AccountViewModels.cs` | CREATE | `src/MVC5/MvcMusicStore/Models/AccountViewModels.cs` | **Adapted to ASP.NET Core Identity's model shapes, not copied.** The file carries no legacy `using` directive at all — its only import is `System.ComponentModel.DataAnnotations` [src/MVC5/MvcMusicStore/Models/AccountViewModels.cs:1] — so an import-rewriting pass would leave it untouched and wrong. The per-model shape is [05](05-aspnet-core-migration-approach.md)'s |
| `src/MvcMusicStore/Binding/CheckoutInputModel.cs`, `src/MvcMusicStore/Binding/AlbumCreateInputModel.cs`, `src/MvcMusicStore/Binding/AlbumEditInputModel.cs` | CREATE | `src/MVC5/MvcMusicStore/Models/Order.cs`, `src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs` for the first; `src/MVC5/MvcMusicStore/Models/Album.cs` and `src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs` for the other two | **Explicit input models replace the class-level `[Bind]` and the synchronous `TryUpdateModel(order)` [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:29], which has no ASP.NET Core counterpart — and the folder holds three files, not one.** `CheckoutInputModel.cs` is the authoritative map's own group; the two administration models are the same pattern applied to the one other action pair that binds an entity directly, which is why they are named here rather than left inside a plural mention: the create POST binds `Album` [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:53-54] and the edit POST binds it and then marks a detached instance `Modified` [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:87-91], so each action needs its own model and the two differ in exactly the key property. Three files, three named sources, one folder. Every property list, and the create/edit asymmetry over `AlbumId`, are [05 §8.11](05-aspnet-core-migration-approach.md)'s |
| `src/MvcMusicStore/ViewModels/ShoppingCartViewModel.cs`, `src/MvcMusicStore/ViewModels/ShoppingCartRemoveViewModel.cs` | CREATE | the same two files under `src/MVC5/MvcMusicStore/ViewModels/` | **Ported, with one addition:** the removal model gains explicit JSON property names, because the AJAX contract's PascalCase field names do not survive the target serializer's web defaults. The annotation and the decision to scope it to this one model rather than to the serializer policy are [05 §8.7](05-aspnet-core-migration-approach.md)'s |
| `src/MvcMusicStore/Services/ShoppingCartService.cs` | CREATE | `src/MVC5/MvcMusicStore/Models/ShoppingCart.cs` | **The cart, order-creation and cart-migration logic leaves the model layer for a service:** `HttpContextBase` becomes `HttpContext`, and the unreferenced `GetCart(MusicStoreEntities, Controller)` overload [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:29] is dropped rather than ported, since its `Controller` parameter type has no target equivalent. Its registration and its internals are [05 §4](05-aspnet-core-migration-approach.md)'s |
| `src/MvcMusicStore/Data/MusicStoreEntities.cs`, `src/MvcMusicStore/Data/ApplicationDbContext.cs` | CREATE | `src/MVC5/MvcMusicStore/Models/MusicStoreEntities.cs`, `src/MVC5/MvcMusicStore/Models/IdentityModels.cs` | **Both contexts gain explicit `DbContextOptions` constructors**, because EF Core honours neither of the two conventions in use today: the class-name-to-connection-string match behind the constructor-less `MusicStoreEntities` [src/MVC5/MvcMusicStore/Models/MusicStoreEntities.cs:5] and the `base("DefaultConnection")` call in `ApplicationDbContext` [src/MVC5/MvcMusicStore/Models/IdentityModels.cs:13]. Structurally, both compile into the web project — §12.5. Whether they stay separate, and how each is registered, are [05 §4.5](05-aspnet-core-migration-approach.md)'s |
| `src/MvcMusicStore/Data/Migrations/Catalog/**`, `src/MvcMusicStore/Data/Migrations/Identity/**` | CREATE | the MVC 5 schema **as extracted from the committed databases** — `src/MVC5/MvcMusicStore/App_Data/MvcMusicStore.mdf` and `src/MVC5/MvcMusicStore/App_Data/aspnet-MvcMusicStore-20131025034205.mdf` — and **not** from either `MvcMusicStore-Create.sql` copy (§13.1) | **Initial migrations generated from the extracted schema and diff-verified before any data is loaded (§13.2).** Structurally: two folders, one assembly, no `MigrationsAssembly` setting and no fifth project (§12.5). The extraction gate itself and the migration design are [05 §5](05-aspnet-core-migration-approach.md)'s |
| `src/MvcMusicStore/Data/SeedData.cs` | CREATE | `src/MVC5/MvcMusicStore/Models/SampleData.cs` | **The 826-LF (827 content lines) seed — the metric named per [01 §2.4](01-architecture-overview.md), since the file carries no terminal newline and this is the physical-line count rather than the non-blank sizing one — becomes a routine an explicit opt-in verb invokes, and `DropCreateDatabaseIfModelChanges` [src/MVC5/MvcMusicStore/Models/SampleData.cs:9] is not reproduced in any form.** Structurally, the executable that invokes it is `tools/provision-admin`'s second verb (§12.6) rather than a fifth project. The routine's content and its three fail-closed guard checks are [05 §5.4](05-aspnet-core-migration-approach.md)'s |
| `src/MvcMusicStore/ErrorViewModel.cs` | CREATE | **net-new** | **Replaces `System.Web.Mvc.HandleErrorInfo`**, the removed type the current error view declares as its model [src/MVC5/MvcMusicStore/Views/Shared/Error.cshtml:1]. No framework type takes its place — exception-handling middleware supplies no Razor model — so the port defines one. Its members and the error contract around it are [05 §8.3](05-aspnet-core-migration-approach.md)'s |
| `src/MvcMusicStore/Health/CatalogConnectivityHealthCheck.cs` | CREATE | **net-new** — the repository has no health endpoint of any kind in any edition ([11 §3.8](11-cloud-readiness-assessment.md)), which is why §10.4 can retire the pin its earlier form carried without losing a capability | The application-owned `IHealthCheck` behind the readiness endpoint. [05 §2.4](05-aspnet-core-migration-approach.md) stage 1 item 11 registers exactly one check and rejects `AddDbContextCheck<T>`; the readiness contract is [06 §9.3](06-azure-hosting-recommendations.md)'s. This document owns the file and its folder under the placement rules of §12.2.1, and adds no behaviour of its own |
| `src/MvcMusicStore/Diagnostics/ErrorRecordExceptionHandler.cs` | CREATE | **net-new**, and the derivation is a removal rather than a file: `HandleErrorAttribute` [src/MVC5/MvcMusicStore/App_Start/FilterConfig.cs:10] is the whole of the current error policy and has no target counterpart, so the middleware that replaces it renders a response and records nothing until this type exists | The `IExceptionHandler` that writes the error record and returns `false`. [05 §2.4](05-aspnet-core-migration-approach.md) stage 1 item 13 and [05 §8.3](05-aspnet-core-migration-approach.md) give it the record's closed field set; this document owns the file and its folder (§12.2.1) |
| `src/MvcMusicStore/Diagnostics/InternalDiagnosticsEndpoints.cs` | CREATE | **net-new** | The handler bodies for the **three** authenticated diagnostic operations, plus the fixed response shapes they return, declared as **nested private records** under §12.2.1's placement exception. [06 §7.5](06-azure-hosting-recommendations.md) mechanisms **P-CFG** (`GET /internal/config-summary`) and **P-DP** (`POST /internal/dp/protect`, `POST /internal/dp/unprotect`) fix each operation's contract, its closed response field set, the dedicated `SlotIsolationProbe` protector purpose and the single authentication contract all three sit behind; [06 §9.3](06-azure-hosting-recommendations.md) gate row 8 fails the release if any of the three is missing; [05 §2.4](05-aspnet-core-migration-approach.md) owns the implementation and the registration order. **One file for all three**, because they are one instrument family behind one authentication contract and splitting them would make that gate's "any of the three" assertion span three files for no gain. The `MapGet`/`MapPost` registrations are lines in `Program.cs`; what needs a file is the handler bodies and their closed shapes. The **P-CFG parse-failure rule is 06's and is not restated here** |
| `src/MvcMusicStore/Identity/PasswordDerivationObservation.cs` | CREATE | **net-new** | The **one** scoped per-request signal, with two consumers: [05 §4.3](05-aspnet-core-migration-approach.md) declares it and [09 §6.8.1.1](09-security-assessment.md) seam 1 adds its two audit members. **One file and not two** — 09's seam 1 records that `PasswordRehashSignal` is an earlier name for this same object, and building it twice is the failure that leaves one consumer reading a value nothing sets |
| `src/MvcMusicStore/Identity/RehashObservingPasswordHasher.cs` | CREATE | **net-new** | The decorating `IPasswordHasher<ApplicationUser>` that observes and never emits — [05 §4.3](05-aspnet-core-migration-approach.md) for the uniform-cost signal and [09 §6.8.1.1](09-security-assessment.md) seam 1 artifact 1 for the rehash observation, one artifact with two readers, registered once. [09 §6.8.1.1](09-security-assessment.md) **owns the registration shape and this document does not restate it**; the one consequence that belongs on a file map is that the type this decorator depends on is the framework's **concrete** `PasswordHasher<ApplicationUser>`, registered as its own concrete type in `Program.cs` and never the interface the decorator is registered as — a framework type needs no file, so the seam adds one row here and not two |
| `src/MvcMusicStore/Identity/AuditingUserManager.cs` | CREATE | **net-new** | One `UserManager<ApplicationUser>` subclass with **two** overrides, `UpdateUserAsync` and `AccessFailedAsync`, which [09 §6.8.1.1](09-security-assessment.md) seam 1 artifact 2 and seam 3 state explicitly are the same type. One file for both overrides, because two files would be two registrations of `AddUserManager<T>` and the second would replace the first |
| `src/MvcMusicStore/Identity/UniformVerificationCost.cs` | CREATE | **net-new** | The singleton that pads credential verification to a fixed cost, required by [05 §4.3](05-aspnet-core-migration-approach.md) |
| `src/MvcMusicStore/Security/CredentialEndpointAttribute.cs` | CREATE | **net-new** | The `[CredentialEndpoint]` marker the rate limiter reads from endpoint metadata, required by [05 §2.4](05-aspnet-core-migration-approach.md) stage 1 item 14, which selects the three credential endpoints by metadata rather than by path string |
| `src/MvcMusicStore/Security/AuditingAuthorizationMiddlewareResultHandler.cs` | CREATE | **net-new**, and its derivation is the authorization surface it observes — the controller-level `[Authorize(Roles = "Administrator")]` [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:12] and the account surface's own attributes, none of which records a denial today because nothing in the repository logs at all ([11 §3.8](11-cloud-readiness-assessment.md)) | The delegating `IAuthorizationMiddlewareResultHandler` required by [09 §6.8.1.1](09-security-assessment.md) seam 2 |
| `src/MvcMusicStore/Security/ContentSecurityPolicy.cs` | CREATE | **net-new** — the repository emits no security response header of any kind ([11 §3.6](11-cloud-readiness-assessment.md)), so nothing is being ported | The Content-Security-Policy value, its enforcing-versus-report-only phase and the `csp-endpoint` group literal, in **one** holder because [06 §10.2](06-azure-hosting-recommendations.md) requires the group name and the `report-to` value to be one literal in one place |
| `src/MvcMusicStore/Binding/CspReportModels.cs`, `src/MvcMusicStore/Services/CspReportSink.cs`, `src/MvcMusicStore/Services/ReadinessVerdict.cs`, `src/MvcMusicStore/Services/ReadinessVerdictHolder.cs`, `src/MvcMusicStore/Services/ReadinessRefreshService.cs`, `src/MvcMusicStore/Services/KeyRingStatusHeartbeatService.cs`, `src/MvcMusicStore/Services/KeyRingBootstrapService.cs`, `src/MvcMusicStore/Services/IHostRoleSignal.cs`, `src/MvcMusicStore/Services/ITestConnectionModeSignal.cs` | CREATE | **net-new** | **Nine designed types that land in two folders the authoritative map already establishes — `Binding/`, which `CheckoutInputModel.cs` occupies, and `Services/`, which `ShoppingCartService.cs` occupies — which is why they are one row and not nine: this map fixes their folder and nothing else, and each is a separate file only because of §12.2.1's one-public-type-per-file rule.** **This is the fifteenth of the designed-type rows §12.2.1 governs**, and it takes that sub-section's first and fourth rules — one public type per file, and no project of its own — while its second rule, which chooses among the five role folders *this* map introduces, does not arise: these nine need no such choice, because the folder each joins is already on the authoritative map and their owners have already said which. Four designs, each named by the deliverable that owns it. **The report collector's two files** ([05 §7.5](05-aspnet-core-migration-approach.md)): the request models under `Binding/`, because they are deserialization targets carrying `[JsonPropertyName]` attributes exactly like the checkout model beside them, and the singleton sink under `Services/`, because the endpoint's two hourly budgets are process-wide mutable state that no per-request or scoped type can hold. **The readiness verdict record, its singleton holder and the two long-lived loops** ([05 §2.6](05-aspnet-core-migration-approach.md)). **The key-ring bootstrap** ([05 §4.9.1](05-aspnet-core-migration-approach.md)). **The two composition-provided signals** the one connection-string validator reads ([05 §3.3](05-aspnet-core-migration-approach.md)) — declared here as interfaces, with `IHostRoleSignal`'s closed two-value `HostRole` enum in the same file beside the interface it exists for, under the same latitude §12.2.1 states for a type with exactly one consumer, and with `ITestConnectionModeSignal` implemented **only** in `src/MvcMusicStore.Tests` so the published application carries no descriptor for it. **The report endpoint is a route-handler endpoint and not a controller action**: [05 §7.5](05-aspnet-core-migration-approach.md) registers `MapPost("/csp-report", …)` with a forwarding lambda that lets the container supply the sink, so this map's **six** controllers stay six, no seventh controller row appears, and the handler body sits in the sink rather than in a controller or in `Program.cs`. Structurally, and this is the whole of what this document claims about them: **no new project, no new folder, no `Compile` item and no edge in §12.4's graph** — all nine compile into the web application by implicit globbing (§5.1), every registration is a line in `Program.cs` (§12.2.1), and what each type does, what its members are and what lifetime it takes are its owner's |
| `src/MvcMusicStore/Security/SubjectPseudonym.cs` | CREATE | **net-new** | The keyed HMAC-SHA256 pseudonymization service and its fail-closed key validation, from [09 §6.8.1](09-security-assessment.md) for the construction, the per-key length and the refusal to start without a key. **The key record itself, its label grammar and its rotation are [06 §8.4.7](06-azure-hosting-recommendations.md)'s and are cited rather than restated anywhere in this document** — this row names a file, not a key format |
| `src/MvcMusicStore/Configuration/HealthProbeOptions.cs` | CREATE | **net-new** | `ReadinessBudget` and `LivenessBudget`, named and given defaults by [05 §2.4](05-aspnet-core-migration-approach.md) stage 1 item 11, which states both budgets as configuration. Options types sit together under `Configuration/`, one per bound section (§12.2.1) |
| `src/MvcMusicStore/Configuration/EventLogOptions.cs` | CREATE | **net-new** | The `Security:EventLog` section, whose one member today is the pseudonym key: [09 §6.8.1](09-security-assessment.md) for the key's validation at startup, and [06 §8.4.7](06-azure-hosting-recommendations.md) — **the sole owner of the key record and its label grammar** — for what the bound value contains and how it is delivered as a platform reference. Section named by its owners, type named here |
| `src/MvcMusicStore/Configuration/` — **nineteen files, named because [05 §3.3](05-aspnet-core-migration-approach.md) names the types and places none of them**: `ConnectionStringOptions.cs`, `IdentityPolicyOptions.cs`, `AuthenticationCookieOptions.cs`, `SessionPolicyOptions.cs`, `AntiforgeryPolicyOptions.cs`, `DataProtectionSettings.cs`, `SqlIdentityOptions.cs`, `ExpectedDeploymentOptions.cs`, `TelemetryOptions.cs`, `SiteOptions.cs`, `SeedingOptions.cs`, `DiagnosticsOptions.cs`, `DataAccessOptions.cs`, `CartOptions.cs`, `CheckoutOptions.cs`, `PagingOptions.cs`, `CachingOptions.cs`, `DisplayOptions.cs`, `LoggingPolicyOptions.cs` | CREATE | **net-new** | **One file per options type that document's section table defines, and this row is what closes the `Configuration/` folder — by enumeration, not by a wildcard.** The table's own arithmetic is what bounds the list: of its twenty-five section rows, **eighteen** name a section whose *whole* binding is an options type it defines — the first eighteen names above, in its order — and one more, the validation view over `Logging`, adds the nineteenth. The **five** it marks *no custom type* (`Hsts`, `Ingress`, `RateLimiting`, `Security`, `Localization`) and the **two** bound by a framework schema (`Logging`'s filtering half and `Kestrel`) contribute **no file at all**, because a value read once in the composition root and pushed into a framework options type needs none — those are registration lines in `Program.cs` under §12.2.1's first co-location. The **type names are 05's and not this map's**; what is this map's is only the folder and the one-type-per-file split, under §12.2.1's third rule. Every member, default, requiredness and validation rule stays with that document: this row adds a path and asserts nothing about content. **Two `Configuration/` files are not in this row and are rows of their own**, because their sections are ones that table does not carry — `HealthProbeOptions.cs`, whose type name 05 uses at its consumer, and `EventLogOptions.cs`, whose `Security:EventLog` section is owned by [09 §6.8.1](09-security-assessment.md) and [06 §8.4.7](06-azure-hosting-recommendations.md) in the division its own row above states. Twenty-one files, three rows, and no file named twice |
| `src/MvcMusicStore/ViewComponents/GenreMenuViewComponent.cs` and `src/MvcMusicStore/Views/Shared/Components/GenreMenu/Default.cshtml` | CREATE | `src/MVC5/MvcMusicStore/Controllers/StoreController.cs:43` — the `[ChildActionOnly]` genre-menu action — and its call site `@Html.Action("GenreMenu", "Store")` [src/MVC5/MvcMusicStore/Views/Shared/_Layout.cshtml:25] | **A child action becomes a view component**, because `[ChildActionOnly]` and `@Html.Action` have no ASP.NET Core counterpart. Two target files per component, in the two conventional locations. The component's arguments, its query and any caching are [05 §8](05-aspnet-core-migration-approach.md)'s |
| `src/MvcMusicStore/ViewComponents/CartSummaryViewComponent.cs` and `src/MvcMusicStore/Views/Shared/Components/CartSummary/Default.cshtml` | CREATE | `src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:86` — the `[ChildActionOnly]` cart-summary action — and its call site `@Html.Action("CartSummary", "ShoppingCart")` [src/MVC5/MvcMusicStore/Views/Shared/_Layout.cshtml:26] | **The same transformation, for the layout's second child action.** Both of these render on every page because both call sites are in the shared layout, which is why they are two rows rather than one line of prose |
| `src/MvcMusicStore/ViewComponents/RemoveAccountListViewComponent.cs` and `src/MvcMusicStore/Views/Shared/Components/RemoveAccountList/Default.cshtml` | CREATE | `src/MVC5/MvcMusicStore/Controllers/AccountController.cs:316` — the third `[ChildActionOnly]` action — its call site `@Html.Action("RemoveAccountList")` [src/MVC5/MvcMusicStore/Views/Account/Manage.cshtml:22], and the partial it renders, whose model is `ICollection<Microsoft.AspNet.Identity.UserLoginInfo>` [src/MVC5/MvcMusicStore/Views/Account/_RemoveAccountPartial.cshtml:1] | **The third of the three child actions, and it is on this map for the same reason as the other two.** MVC 5 declares `[ChildActionOnly]` exactly three times and all three become components, so the external-login **management** surface is retained in the target and the `UserLoginInfo` collection moves under this component. The choice [05](05-aspnet-core-migration-approach.md) holds over `ChallengeResult` (the `AccountController.cs` row above) concerns the external *challenge* flow, **not** whether this component exists |
| `src/MvcMusicStore/Views/**/*.cshtml` **excluding `Views/Shared/Components/**`, which the three view-component rows above own**, and including `src/MvcMusicStore/Views/_ViewImports.cshtml` and `Views/_ViewStart.cshtml` | CREATE | `src/MVC5/MvcMusicStore/Views/**/*.cshtml` — 29 tracked files — and `src/MVC5/MvcMusicStore/Views/Web.config` | **The views port and `Views/Web.config` does not:** `_ViewImports.cshtml` takes over its Razor namespace registration [src/MVC5/MvcMusicStore/Views/Web.config:5-23], and its `BlockViewHandler` mapping [src/MVC5/MvcMusicStore/Views/Web.config:31-32] ends with the IIS integrated pipeline that gave it meaning. Structurally: Razor is compiled by the Web SDK at build and at publish, so these views become compile-checked for the first time (§5.4). **Two different inventories of the same 29 views are in play and this row uses both, because a document that quotes one number as if it were the other has miscounted the work.** (a) **Six source views name a removed API or type** — `Shared/Error.cshtml`, `Shared/_LoginPartial.cshtml`, `Account/Manage.cshtml`, `Account/_ChangePasswordPartial.cshtml`, `Account/_RemoveAccountPartial.cshtml` and `Account/_ExternalLoginsListPartial.cshtml`, the sixth being the one a namespace-only search misses because its removed members are OWIN rather than Identity: `@using Microsoft.Owin.Security` [src/MVC5/MvcMusicStore/Views/Account/_ExternalLoginsListPartial.cshtml:1] and `Context.GetOwinContext()…` [src/MVC5/MvcMusicStore/Views/Account/_ExternalLoginsListPartial.cshtml:6]. (b) **The target work partitions as `0 + 3 + 5 + 21 = 29`** — three become a view component's `Default.cshtml` (their own rows above), five need per-line work, twenty-one port with mechanical changes only, and none is deleted. The two units differ by exactly one file: `_RemoveAccountPartial.cshtml` is in the six *and* is counted as a component view in the partition, which is why the per-line figure is five rather than six. The **five-type** list is also the narrower one the authoritative future application map names, so both figures are legitimate and the defect is using either as the other. [05 §8.4](05-aspnet-core-migration-approach.md) owns the partition and [05 §8.3](05-aspnet-core-migration-approach.md) the per-view work; the bundling-helper call sites and the Bootstrap markup port are [05 §8](05-aspnet-core-migration-approach.md)'s |
| `src/MvcMusicStore/wwwroot/**` — the application's own assets, **excluding `wwwroot/lib/**` and `wwwroot/js/shopping-cart.js`, each of which is its own row** | CREATE | `src/MVC5/MvcMusicStore/Content/**`, `src/MVC5/MvcMusicStore/Scripts/**`, `src/MVC5/MvcMusicStore/Images/**`, `src/MVC5/MvcMusicStore/fonts/**` and `src/MVC5/MvcMusicStore/favicon.ico` | **The 27 asset files (appendix A.3) plus the root favicon move under a web root and are served by static-file middleware, with their path casing corrected on the way.** The two exclusions are stated in the target cell rather than implied, because `wwwroot/**` as written would otherwise claim files two other rows already own — one target file with two owning rows is the same defect as a target file with none. Structurally: no `Content` item group survives the conversion, because the SDK serves `wwwroot` without per-file items (§5.1). The per-file relocation is [05 §8](05-aspnet-core-migration-approach.md)'s and the repository-wide casing audit is [06](06-azure-hosting-recommendations.md)'s |
| `src/MvcMusicStore/wwwroot/js/shopping-cart.js` | CREATE | `src/MVC5/MvcMusicStore/Views/ShoppingCart/Index.cshtml:7-35` — the view's inline `<script>` block, whose `$.post` at [src/MVC5/MvcMusicStore/Views/ShoppingCart/Index.cshtml:17] is the cart-removal call | **A script that is markup today becomes a served file, and it is a row rather than a member of the `wwwroot/**` row above because its source is a Razor view rather than an asset file.** Two independent requirements land on it: [06 §10.2](06-azure-hosting-recommendations.md) makes the inline block a policy violation under `script-src 'self'` with no `unsafe-inline` and no nonce, and [05 §7.4](05-aspnet-core-migration-approach.md) makes the removal request carry the anti-forgery token in a `RequestVerificationToken` **header**, so this file is the client half of that contract. Structurally: one file under the web root, served by the shared framework's static-file middleware with no bundler and no build step (§10.1), referenced by a `<script src>` in the same document so the token in the page remains reachable. Its contents — the selector, the read-at-post-time rule and the header literal — are [05 §7.4](05-aspnet-core-migration-approach.md)'s |
| CI pipeline definition | CREATE | **net-new** | **Deliberately not named as a file.** Its path and format depend on a provider choice this assessment does not make; [03](03-modernization-roadmap.md) carries provider selection as an explicit roadmap gate. What this document contributes to it: the build image must satisfy the `8.0.400` band (§3.2), restore runs in locked mode (§6.4), and `dotnet tool restore` precedes any migration step (§6.3) |
| `Dockerfile` | CREATE | **net-new** | **Conditional.** It exists only under the container-native hosting option, which is [06](06-azure-hosting-recommendations.md)'s to select. If code deployment is chosen, this file does not exist at all |
| `src/MVC3/**`, `src/MVC4/**`, `src/MVC5/**` — the three legacy projects, their four solutions and their committed databases | **REFERENCE** | themselves | **Retained read-only** as historical references and as the behavioural baseline the port is validated against; none is modified and none is deleted (§12.3). They continue to exist alongside the single new root solution, so §5.6's consolidation is of the *target* rather than a deletion of the past. **This is the only non-CREATE row on the map**, and it is a row rather than prose because a reader checking the map for a target that has no source needs to find the retained legacy tree accounted for here too |
| `src/MvcMusicStore/ApplicationServices.cs` | CREATE | **net-new** — the legacy application has no equivalent: its composition is spread across `Global.asax.cs` and the OWIN startup files [src/MVC5/MvcMusicStore/Global.asax.cs:13-21], [src/MVC5/MvcMusicStore/Startup.cs:4-14], and nothing there is callable from outside the web process | **The application's one registration seam**, and the file exists so that the seam has a name a second project can call. It declares `public static class ApplicationServices` with one method — `public static IServiceCollection AddMvcMusicStoreServices(this IServiceCollection services, IConfiguration configuration)` — which registers **both `DbContext` types** (`MusicStoreEntities` and `ApplicationDbContext`, each on the SQL Server provider at its configured connection string), the **Identity core, store and manager services** over `ApplicationUser` including the role manager, the password hasher and the Identity options, and the **options objects the tools read**, bound from the `IConfiguration` passed in. **`Program.cs` calls this method itself**, which is the property that matters: there is exactly **one** registration path, so a tool cannot compose a different application than the one that serves requests. What deliberately stays in `Program.cs` and is **not** in the seam: the HTTP-pipeline concerns — MVC and Razor, the cookie authentication schemes, session, anti-forgery, data-protection key persistence, health checks and the middleware order — because a console host has no pipeline to attach them to and a tool that acquired them would be configuring behaviour it never exercises. The **values** applied inside the seam are not this document's: the authentication, password and lockout policy is [05 §6.1](05-aspnet-core-migration-approach.md)'s, the two contexts and their separate migration sets and history tables are [05 §4.5](05-aspnet-core-migration-approach.md)'s, and everything `Program.cs` composes around this call is [05 §2](05-aspnet-core-migration-approach.md)'s. This row owns the file, the type, the method signature, what is registered there versus in `Program.cs`, and the single-path rule; §12.4 owns why the tools depend on it. Added to this map under the plan-correction record of §12.1 |
| `src/MvcMusicStore.Contracts.Tests/MvcMusicStore.Contracts.Tests.csproj` | CREATE | **net-new** — the repository contains no test project of any kind (appendix A.4) | SDK-style, `net8.0`, with its own committed lockfile, declaring **seven** pins of §7.2 directly — the six test-tooling pins `xunit` `2.9.2`, `xunit.runner.visualstudio` `2.8.2`, `Microsoft.NET.Test.Sdk` `17.11.1`, `AngleSharp` `1.7.2`, `Microsoft.Data.SqlClient` `5.1.9` and `Microsoft.Playwright` `1.62.0`, plus the band pin `Microsoft.Extensions.Identity.Core` `8.0.30`, which is here because this project owns the diagnostic pseudonym scheme that **invokes** `ILookupNormalizer` (§7.2, §7.5, §7.6, §7.7, §7.8). **Three of the seven are declared in the in-process project as well, and four are not, and the split is not a style choice:** the three **test-execution** pins — `xunit`, `xunit.runner.visualstudio` and `Microsoft.NET.Test.Sdk` — deliver their function through build and analyzer assets, which **do not cross a project reference**, so each runnable test project declares them itself (§12.3); `Microsoft.Playwright` is in the same group for the reason §7.7 states. The three **library** pins — the response-body parser, the SQL client and the Identity abstractions — are declared **here and only here**, because this project owns the assertions that parse a body, the state observer and legacy attach/detach lifecycle that need a SQL connection, and the pseudonym canonicalization, and their compile and runtime assets *do* reach the in-process project through its reference to this one (§7.5, §7.6, §7.8). **It carries no `ProjectReference` to `src/MvcMusicStore/MvcMusicStore.csproj` and no reference to any other target project** — it reaches whatever host it is pointed at **purely over HTTP**, at a base address it reads from configuration. That absence is the load-bearing property of this row and the reason there are two test projects rather than one: it is what makes this project **buildable and runnable before the ported web application exists**, which is the condition [03](03-modernization-roadmap.md)'s W4 has to meet. It owns the **shared contract assertions** and the **legacy-baseline fixture**, and it is the project W4 restores, builds and runs to characterize the legacy baseline. It also owns the **deployed-only fixture and its concrete class**, for the reason those two rows below state — the only classes here that the baseline run does **not** execute, because a category filter selects them and a deployed host, not this project's reference set, is what they need. **The assertions are declared as one `abstract` class per contract surface**, each holding that surface's `[Fact]` and `[Theory]` methods and taking every dependency it needs — the base address, the client, the state observer — from an **injected, runtime-neutral context** rather than from anything that names a runtime. **The legacy-bound concrete classes are declared here, in this assembly**: one `sealed` class per surface, deriving from that surface's base and supplying the legacy context, and carrying nothing else. That is the arrangement §12.3 argues, and the reason the concretes are declared per assembly rather than inherited across the reference is a property of test discovery rather than of the compiler — stated there once. This row owns that the project exists, where it sits, what it references and the shape of the classes it declares; the assertions' architecture and coverage are [05 §12.2](05-aspnet-core-migration-approach.md) and [05 §12.4](05-aspnet-core-migration-approach.md)'s, the context abstraction and the legacy fixture's design are [05 §12.6](05-aspnet-core-migration-approach.md)'s, and the runnable commands are [05 §12.10](05-aspnet-core-migration-approach.md)'s (see the note following this table). Its place in the sequence is [03](03-modernization-roadmap.md)'s |
| `src/MvcMusicStore.Contracts.Tests/Contracts/**` | CREATE | **net-new** | The **`public abstract` contract bases**, one per surface — the single copy of every shared assertion. This document fixes their declaration site, their accessibility and the fact that they are abstract (§12.3); the surfaces themselves, their assertion bodies and the injected context's shape are [05 §12.6](05-aspnet-core-migration-approach.md)'s and are not reproduced here |
| `src/MvcMusicStore.Contracts.Tests/Legacy/**` | CREATE | **net-new** | The **`public sealed` legacy-bound concretes**, one per surface, each a derivation plus the legacy context plus its `[Collection]` attribute and **no assertion logic**. These are the classes the pre-port baseline run discovers, which is the property §12.3 argues. The destructive-operation sweep class is **not** a member of this folder — [05 §12.8](05-aspnet-core-migration-approach.md) gives it its own path and the row below carries it |
| `src/MvcMusicStore.Contracts.Tests/Maintenance/OrphanSweepTests.cs` | CREATE | **net-new** | The **standalone destructive-operation sweep class**, at the path [05 §12.10](05-aspnet-core-migration-approach.md) fixes for it and traited `Category=Sweep`. It is a **third class folder in this project**, beside `Contracts/` and `Legacy/`, because it is neither a shared assertion base nor a legacy-bound concrete: it is a concrete runnable class with its own `[Fact]` methods that selects no row of [05 §12.4](05-aspnet-core-migration-approach.md) and appears in no coverage count. The folder is its own so that a run filtering on the trait and a reader walking the project both find it in one place rather than inside a folder whose membership rule excludes it. What it asserts, when it is invoked and why it is a maintenance invocation rather than a coverage category are [05 §12.8](05-aspnet-core-migration-approach.md)'s |
| `src/MvcMusicStore.Contracts.Tests/LegacyBaselineFixture.cs` | CREATE | **net-new** | The **`public` collection-fixture type** the legacy-side collection definitions bind, instantiated once per collection. What it does — deploying, starting and driving the MVC 5 application over HTTP, the two-database reset and `ResetAsync()` — is [05 §12.3](05-aspnet-core-migration-approach.md)'s and [05 §12.7](05-aspnet-core-migration-approach.md)'s; this row owns its existence, its path, its accessibility and its project |
| `src/MvcMusicStore.Contracts.Tests/DeployedEndpointFixture.cs` | CREATE | **net-new** | The **`public sealed` deployed-only fixture**, and the third fixture type in the suite — mapped because the one `Category=Deployed` case of [05 §12.4](05-aspnet-core-migration-approach.md) row 47 runs against a **deployed** host and neither mapped fixture can supply one. Its structural properties are this row's: it is **runtime-neutral** and names no application type, which is why it sits in the project that carries **no reference to the web application**; it **starts nothing and provisions nothing** — **no in-process host and no database** — and exposes no observer, no setup API and no reset, so no assertion bound to it can reach a store even by accident; it **consumes a base address** from configuration rather than producing one; and it **owns its own HTTP client policy, with automatic redirect following disabled**, because row 47's assertion *is* the redirect and a client that followed it could observe neither the status nor the `Location`. **What it reports when the address is absent, and the four-value admissible redirect set it asserts, are [05 §12.6](05-aspnet-core-migration-approach.md)'s and [05 §12.4](05-aspnet-core-migration-approach.md)'s** |
| `src/MvcMusicStore.Contracts.Tests/DeployedEndpointTests.cs` | CREATE | **net-new** | The **`public sealed` concrete** the deployed stage discovers — `[Collection("Deployed")]`, holding row 47's cases **directly** rather than deriving from a contract base, because the legacy application has no deployed surface to characterize and there is therefore nothing to share. It is the **only** class in either assembly carrying that trait, which is what makes a category filter a complete selection of it. **Which stage runs it and which workstream authors it, stated because a mapped class with neither is not runnable:** it is executed by [06 §12.1](06-azure-hosting-recommendations.md)'s **non-traffic deployment-verification stage**, whose invocation is written at [05 §12.10](05-aspnet-core-migration-approach.md)'s Stage C and whose pipeline manifest [03](03-modernization-roadmap.md)'s **W11** authors; and it is **authored in [03](03-modernization-roadmap.md)'s W7**, alongside the target-facing concretes, whose exit gate requires this binding to **compile and be discoverable** while explicitly excluding its execution — there is no deployed host at that gate. **No new project and no new pin**: it is a class in the project W4 already created, and its assertions need only the client that project already declares |
| `src/MvcMusicStore.Contracts.Tests/Collections/*.cs` | CREATE | **net-new** | One **`public`** collection-definition class per surface group this assembly declares a class in — `[CollectionDefinition]`, `ICollectionFixture<LegacyBaselineFixture>`, a `const string` for the name, and no test methods (§12.3). **One further definition in the same folder binds the other fixture this assembly declares**: the deployed-only collection, `ICollectionFixture<DeployedEndpointFixture>`, whose name is the trait the deployment stage filters on — which is why this assembly has **two** fixture types and its definitions do not all name the same one. Added to this map under the plan-correction records of §12.1 |
| `src/MvcMusicStore.Contracts.Tests/xunit.runner.json` | CREATE | **net-new** | Runner configuration for the project above, carrying the assembly-level switch `"parallelizeTestCollections": false` — that exact value, not a default relied upon. Declared as a **content item with `CopyToOutputDirectory` set to preserve-newest**, because a runner configuration the build does not copy beside the test assembly is a file the runner never reads. The requirement it satisfies is [05 §12.7](05-aspnet-core-migration-approach.md)'s and is not restated here: the fixtures own shared databases and, on the baseline side, a privately deployed legacy application, so concurrent collections would race a reset against an attach. This row owns only that the file exists, where it sits, its exact setting and how it reaches the output directory |
| `src/MvcMusicStore.Contracts.Tests/Fixtures/fixture-data.json` | CREATE | **net-new** | The **single shared fixture-data manifest**, and the reason it is one file rather than two is that a cross-baseline comparison of two applications holding different data compares nothing. **Both** loaders read it: the legacy loader in this project and the target loader in `src/MvcMusicStore.Tests`. It is declared **once**, here, as a **content item with `CopyToOutputDirectory` set to preserve-newest**, which is what puts it beside the test assembly of both projects — this project's own output for the baseline run, and, because copy-to-output content items flow along a project reference, `src/MvcMusicStore.Tests`'s output for the target run. This row owns its existence, its path, its single-declaration rule and how it reaches each output directory. **Its contents are [05 §12.3](05-aspnet-core-migration-approach.md)'s** — the entities, exact row counts, fixed integer keys, the fixture account set and the manifest fingerprint — and are not reproduced here |
| `src/MvcMusicStore.Contracts.Tests/Fixtures/ModelOverrides/` — **ten files, named here because [05 §12.4](05-aspnet-core-migration-approach.md) row 53 writes the folder and a placeholder for the file**: `column-type.json`, `precision-and-scale.json`, `max-length.json`, `nullability.json`, `identity.json`, `primary-key.json`, `delete-rule.json`, `column-default.json`, `index.json`, `email-length-254.json` | CREATE | **net-new** | One file per divergence dimension of [05 §5.1](05-aspnet-core-migration-approach.md) step 3, in the order that section enumerates them — so the first nine names above are row 53's `53a` through `53i` respectively, and the `--model-overrides` path in each of those nine command lines resolves to exactly one of them. **The tenth is a second consumer of the same seam rather than a tenth dimension**, and it is named for the same reason the nine are: [05 §12.4](05-aspnet-core-migration-approach.md) row 77 `77b` re-diffs one immutable extraction against a target narrowed to the checkout input model's `[StringLength(254)]` bound, and its `--model-overrides` argument is a **path**, so the case is unrunnable unless the file exists at a fixed location. It shares this row because it shares the folder, the declaration site and the copy behaviour; it is counted in this row's ten and in §12.1's record 7 rather than taking a row of its own. **The dimension is in the file name and the case letter is not**, deliberately: the case ids belong to 05's matrix and are renumbered there, while the dimension is the thing the file diverges, so a file named for its dimension does not go stale when the matrix moves. They are **content items with `CopyToOutputDirectory` set to preserve-newest**, in the same declaration and for the same reason as the manifest above: the tool receives a **path** and reads the file from disk, so it has to exist beside the test assembly of whichever project runs the case. **What each file contains — the single dimension it diverges and the value it diverges to — and the `--model-overrides` switch that consumes it are [05 §5.1](05-aspnet-core-migration-approach.md)'s and [05 §5.7](05-aspnet-core-migration-approach.md)'s**, and this row reproduces neither |
| `src/MvcMusicStore.Contracts.Tests/Fixtures/seed-expected.json` | CREATE | **net-new** | The seeding oracle — the expectation row 24 `24a` compares the seeded target against, captured from the legacy store rather than computed by the code under test. Same fixture path, same **preserve-newest content item** behaviour, and read by **both** projects' loaders for the same reason the manifest is. Its per-table counts and digests, and the gate that its recorded legacy commit must equal the baseline reference's, are [05 §12.4](05-aspnet-core-migration-approach.md)'s |
| `src/MvcMusicStore.Contracts.Tests/Fixtures/baseline-reference.json` | CREATE | **net-new** | The **committed** baseline reference — the accepted legacy identity a consuming run cannot compute for itself, and the value the seeding oracle above is gated against. Same fixture path and the same **preserve-newest content item** behaviour, because the run that reads it is the one whose output directory it must sit in. It is a **tracked file updated in the change that accepts a re-captured baseline**, which is what distinguishes it from the run-produced baseline *record*: that record carries the resolved runtime identities of a single execution, including the normalizer identity of §7.8, and is a run artifact rather than a mapped file. **Its three fields, the fail-closed comparison and the platform split that consumes it are [05 §12.10](05-aspnet-core-migration-approach.md)'s** |
| `src/MvcMusicStore.Tests/MvcMusicStore.Tests.csproj` | CREATE | **net-new** — same absence (appendix A.4) | SDK-style, `net8.0`, with its own committed lockfile, and **five direct package references of its own** (§7.2): `Microsoft.AspNetCore.Mvc.Testing` `8.0.30`, which only this project needs, and — declared here **as well as** in the contracts project rather than inherited from it — `xunit` `2.9.2`, `xunit.runner.visualstudio` `2.8.2`, `Microsoft.NET.Test.Sdk` `17.11.1` and `Microsoft.Playwright` `1.62.0`. **The redeclaration is required, not redundant:** those four deliver their function through build and analyzer assets, which do not cross a project reference (§12.3), so a project that acquired them only through the reference below would compile and then be neither built as a test project nor discovered nor executed by `dotnet test` — the adapter absent from its output, the test targets never imported. `AngleSharp` `1.7.2`, `Microsoft.Data.SqlClient` `5.1.9` and `Microsoft.Extensions.Identity.Core` `8.0.30` are deliberately **not** redeclared here: they are library pins whose compile and runtime assets do cross the reference (§7.5, §7.6, §7.8). **Two project references, and both are required:** `src/MvcMusicStore/MvcMusicStore.csproj`, because packages alone do not make an in-process host buildable and this is what makes the test assembly compile against, and boot, the web application's own assembly; and `src/MvcMusicStore.Contracts.Tests/MvcMusicStore.Contracts.Tests.csproj`, because the **abstract contract bases** it derives from live there — two copies of one assertion is the arrangement in which the baseline side and the target side silently stop asserting the same thing. **What this project declares is its own set of `sealed` concrete classes**, one per surface, each deriving from that surface's abstract base and supplying the **target** context — the in-process host's client and the target-side state observer — and carrying no assertion logic of its own. **The reference alone would run nothing**, which is exactly why the concretes are declared here: §12.3 states the discovery property that makes this the mechanism rather than a stylistic choice. It hosts the application **in process**, and it is the project [03](03-modernization-roadmap.md)'s W7 adds alongside the contracts project. It is the only *test* project that references the web application; the single operator console references it as well, for the separate reason its own row states, and no legacy project is referenced by anything. The split of ownership is the same as the row above: the fixture design that consumes this reference — the factory, its overrides and its clients — is [05 §12.6](05-aspnet-core-migration-approach.md)'s, and the runnable commands are [05 §12.10](05-aspnet-core-migration-approach.md)'s |
| `src/MvcMusicStore.Tests/Core/**` | CREATE | **net-new** | The **`public sealed` target-bound concretes**, one per shared surface, **plus the target-only classes that have no legacy counterpart** — each a derivation or a standalone class, its target context, its `[Collection]` attribute and no assertion logic. Which rows are target-only is [05 §12.4](05-aspnet-core-migration-approach.md)'s |
| `src/MvcMusicStore.Tests/CoreApplicationFixture.cs` | CREATE | **net-new** | The **`public` collection-fixture type** the target-side collection definitions bind, instantiated once per collection. Its in-process host, the disposable engine it provisions and its reset are [05 §12.6](05-aspnet-core-migration-approach.md)'s and [05 §12.7](05-aspnet-core-migration-approach.md)'s; this row owns its existence, its path, its accessibility and its project |
| `src/MvcMusicStore.Tests/Collections/*.cs` | CREATE | **net-new** | The same one-per-group definitions for this assembly, binding `ICollectionFixture<CoreApplicationFixture>` — **required separately, because `[Collection]` names resolve within the assembly being run** (§12.3). Added to this map under the plan-correction record of §12.1 |
| `src/MvcMusicStore.Tests/xunit.runner.json` | CREATE | **net-new** | The same file for the in-process project, with the same exact value `"parallelizeTestCollections": false` and the same preserve-newest content-item copy behaviour. **One per test project, because the setting is assembly-level:** a single copy in one project does not govern the other project's assembly, and the target-side fixtures are the half that provisions databases and applies migration sets. Requirement source: [05 §12.7](05-aspnet-core-migration-approach.md) |
| `src/MvcMusicStore/Resources/ApplicationMessages.resx` | CREATE | **net-new** — no edition has a resource file of any kind, verified repository-wide: `git ls-files '*.resx'` returns `0`, so every user-facing string in all three legacy applications is a literal in a view or a `ModelState` message | The application's **one committed resource file**, and the only reason it is mapped here is that a named entry in it is a **test oracle**: [05 §12.4](05-aspnet-core-migration-approach.md) row 37 asserts the rendered cart-migration notice is byte-equal to the value of the entry **`CartMigrationNotice`**, read from the resource rather than transcribed into the test. `.resx` is an **`EmbeddedResource`** under the SDK's default item behaviour — not copied to output — so the reading side must hold a reference to this assembly; it does, because row 37 is **target-only** and its concrete sits in `src/MvcMusicStore.Tests`, which references the web project. That is a structural consequence worth stating: the reference-free contracts project could not read this oracle, which is one more reason the case is target-side. **The entry's value is a product decision and is [05](05-aspnet-core-migration-approach.md)'s**, together with the four predicates that row holds it to; this row owns the file's existence, its path, its build action and the fact that the string has exactly one home. Added to this map under the plan-correction record of §12.1 |
| `tools/provision-admin/ProvisionAdmin.csproj`, with its `Program.cs` | CREATE | `src/MVC5/MvcMusicStore/App_Start/Startup.App.cs` — the startup provisioning it replaces | SDK-style `net8.0` console project with its own committed lockfile and the **same `UserSecretsId` as the web project** (§5.7), **not deployed with the web application**, and the **only operator console project in this map**. It carries **ten verbs, not ten projects** — `admin`, `seed`, `extract-schema`, `diff-schema`, `load-catalog`, `load-identity`, `reconcile`, `seal-manifest`, `accept-run` and `close-run` — so there is one project file, one `Program.cs`, one publish output, one checksum and one promotion path for every operator action the target needs (§12.3, §12.4). It carries a **`ProjectReference` to `src/MvcMusicStore/MvcMusicStore.csproj`**, so it builds the same Identity registrations the web application validates rather than a second copy of them (§12.3) — a build-time edge, which is not a deployment. Each verb's own contract — its switches, its guards, its assertions, its secret channel and its idempotence — is [05](05-aspnet-core-migration-approach.md) §5.1.2, §5.4 and §10.2's; which stage runs it, under which principal and against what asserted target is [06](06-azure-hosting-recommendations.md) §6.3.2's |
| `deploy/sql/SecurityAuditLog.sql` | CREATE | **net-new** — the repository has three `.sql` files and none of them is a deployment script: `git ls-files '*.sql'` returns only the edition schema scripts `src/MVC3/MvcMusicStore-Assets/Data/MvcMusicStore-Create.sql`, `src/MVC4/MvcMusicStore-Create.sql` and `src/MVC4/MvcMusicStore/MvcMusicStore-Create.sql` | **A standalone deployment script, and the row §12.1's count of four consuming-deliverable artifacts refers to.** [06 §9.5](06-azure-hosting-recommendations.md) requires a durable audit table and states its DDL in full; what nothing stated until this row is *where that DDL lives*, which is the only reason it is mapped here. It is **not** an EF Core migration and adds none: [06 §6.3](06-azure-hosting-recommendations.md) applies it as its own step of the provisioning order, under the deployment identity and inside that section's DDL window, so neither context's migration set and neither history table acquires it and §12.5's placement of both migration sets is unchanged. Structurally: a `.sql` file under a repository-root `deploy/sql/` folder, **compiled into nothing and packaged with no project** — it is neither a `Compile` nor a `Content` item of any of the four projects of §5.6, so it adds no project, no item group and no edge to §12.4's graph, and it reaches the release runner as a repository file. Its content, its `OBJECT_ID` guard, its append-only grants and the step that applies it are [06 §9.5](06-azure-hosting-recommendations.md)'s and [06 §6.3](06-azure-hosting-recommendations.md)'s; this row owns its existence, its path and its build treatment |

**The row count, checkable, and every target appearing exactly once.** The table above is **one table of 69
rows**, with **69 distinct target cells** — no path, glob or artifact is named by two rows — of which **68
are CREATE and 1 is REFERENCE**. The count is arithmetic rather than assertion, and it decomposes in two
steps:

- **`27 + 1 + 2 + 6 = 36` authoritative-map rows.** The authoritative future application map (§0.4.1)
  carries **27 target groups**, and three of them are one group over more than one file, so they take more
  than one row without adding a target: the SDK-band and package-source manifests are one group and two
  files, so `global.json` and `NuGet.config` take a row each — **one** extra row; the three
  `[ChildActionOnly]` view components are one group and three components, so they take three rows — **two**
  extra rows, written out for the reason §12.1 gives; and the test-project group — the plan's single
  *"`MvcMusicStore.Tests.csproj` and its test classes"* row — takes seven rows, because §12.3 splits it into
  two projects and names each project file, each fixture and each of the **three** class folders separately
  — **six** extra rows. The third class folder is `Maintenance/`, whose one file
  [05 §12.10](05-aspnet-core-migration-approach.md) gives its own path: a test class is *"its test classes"*
  and so belongs to this group rather than to §12.1's record of unlisted artifacts.
- **`1 + 4 + 13 + 15 = 33` additions, each with its sanction in §12.1's short record.** The **lockfile**
  row is **one**: it is required by §6.4's locked-mode restore rather than by the §0.4.1 map, and it names
  four files in one row, one per project. **Four** rows are artifacts a *consuming* deliverable has since
  decided and given no path to: `wwwroot/lib/**`, the committed output of the `libman.json` acquisition
  mechanism §9.3 fixes; `wwwroot/js/shopping-cart.js`, required by
  [06 §10.2](06-azure-hosting-recommendations.md)'s `script-src 'self'` policy and carrying
  [05 §7.4](05-aspnet-core-migration-approach.md)'s token header, a row rather than a member of
  `wwwroot/**` because its source is a Razor view rather than an asset file — a derivation no glob over the
  asset directories would express; `deploy/sql/SecurityAuditLog.sql`, required by
  [06 §9.5](06-azure-hosting-recommendations.md)'s durable audit table, whose DDL that document states in
  full and whose *location* nothing stated until this row; and `Models/CartAlias.cs`, the cart-handoff
  alias entity type [05 §4.13](05-aspnet-core-migration-approach.md) declares as a mapped member of
  `MusicStoreEntities` and states in full while placing it nowhere — the only one of the four that is a
  **CLR type**, which is why its absence was not merely a missing path but a public type with no containing
  file, which §12.2.1 names as exactly the unowned target §12.1 forbids. **Thirteen** rows are the ones
  §12.1's six numbered records notify — three from record 2, one from record 6, four from record 7, two from record 8,
  one from record 10 and two from record 11. **Fifteen** rows are the designed types, **forty-one files**:
  one under `Health/`, two under `Diagnostics/`, four under `Identity/`, four under `Security/`, **three
  rows and twenty-one files under `Configuration/`** — two single-file rows plus the row that closes that
  folder by enumerating the nineteen options types
  [05 §3.3](05-aspnet-core-migration-approach.md)'s section table names — and **one row and nine files under
  `Binding/` and `Services/`**, folders the authoritative map already establishes. Those forty-one are the
  readiness check, the exception-record writer, the three authenticated diagnostic operations, the
  credential-verification and Identity audit seams, the authorization result handler, the
  Content-Security-Policy holder, the pseudonymization service, the twenty-one bound options types, the
  report collector's request models and its sink, the readiness verdict, its holder and the two long-lived
  loops, the key-ring bootstrap, and the two composition-provided signals. **None of those forty-one
  appears in the authoritative map's 27 groups**, because that map was written before the designs that
  require them; each row cites the deliverable that requires its type, and §12.2.1 states the placement
  rules the fifteen rows follow rather than restating the rows.
- **`36 + 33 = 69`.** Every row is one target and every target is one row, and **the map carries no
  catch-all**: every file under `src/MvcMusicStore/` is named by an enumerated row, so the table is closed
  over the ported project **by enumeration** rather than by a residual row that named again what the rows
  above it had already named. Where a row covers more than one file it names them, or its membership rule
  names the bounded set another section or another deliverable enumerates — the views, the assets, the two
  migration folders, the test projects' class folders and the nineteen options types each resolve that way —
  which is the distinction the removed completeness row failed: each such set is bounded by something
  nameable rather than by "whatever is left", and **no two rows own the same target**, which is the property
  the distinct-cell count above checks. No group is represented by a plural
  mention, an "etc." or an unbounded wildcard, and the one **REFERENCE** row accounts for the retained legacy
  tree so that a reader walking the map finds every path in it either created from a named source, marked
  net-new, or retained unchanged.

**Seven kinds of row that were here and are not any longer, recorded rather than quietly dropped.** The
count above reflects every removal, and each is recorded because a map that silently loses a row is
indistinguishable from a map that never had it.

| Row removed | Why it is gone |
| --- | --- |
| The **completeness catch-all** — "everything under `src/MvcMusicStore/` not listed above" | It named again what nine enumerated rows already named — `Program.cs`, `appsettings*.json`, the controllers, the models, the binding and view models, the services, `Data/` and the views — so it made two rows own the same targets, which is the one property this table exists to guarantee against. The map is closed over the ported project by enumeration instead, and the compilation fact the row carried is §5.1's: every file in the project is compiled by implicit globbing rather than by an explicit `Compile` inventory |
| `Security/CspReportEndpoint.cs` — **one file holding the report handler and, as nested private records, the two shapes it binds** | The owning deliverable has since fixed a different decomposition, and this map yields on it because the contents of a file are that document's fact and not this one's: [05 §7.5](05-aspnet-core-migration-approach.md) makes the handler a **route-handler lambda in `Program.cs`** that forwards to a **singleton `Services/CspReportSink.cs`**, and puts the two report shapes in **`Binding/CspReportModels.cs`** as public deserialization targets beside the checkout model rather than nested privately beside the policy — on the reasoning that a wire contract belongs with the other wire contract and that process-wide hourly budgets cannot live in a per-request type. The same section deletes an earlier form of its own row that had put the handler on `HomeController`, so the endpoint is a route-handler endpoint on both sides of the seam and neither document leaves a seventh controller anywhere. Two rows' worth of files therefore replace one row's: the three paths are members of the nine-file `Binding/`-and-`Services/` row above, and no `Security/CspReportEndpoint.cs` exists to map |
| `Configuration/RateLimitingOptions.cs` — an options type this map named for the rate-limiting section | **The type does not exist, and the map had invented it.** The owner of the configuration contract is explicit: [05 §3.3](05-aspnet-core-migration-approach.md) marks `RateLimiting` as one of the five sections with **no custom type**, read once in the composition root, carrying exactly **one** member — `MaxReplicas`, a required positive integer. The limits this row attributed to it are not configuration at all: [05 §2.4](05-aspnet-core-migration-approach.md) stage 1 item 14 states the per-address and fleet permit figures in its own prose and *derives* the per-instance aggregate as `ceiling(200 / MaxReplicas)`, so the only thing a deployment supplies is the replica ceiling. A single member read once in `Program.cs` needs no file (§12.2.1's first co-location), and inventing a type for it would have put a name in this map that no consuming deliverable can bind. Removing it is why the `Configuration/` folder is now **twenty-one** files over three rows rather than twenty-two |
| The **audit exporter** as a role with no path | [06 §9.5](06-azure-hosting-recommendations.md) has since **decided the question**: the exporter is operated infrastructure **outside this repository's application tree**, so this map gains no file for it. §12.2.2 records the closed decision and what would reopen it. A target with no artifact in this tree has no row, exactly as the code-deployment choice means no `Dockerfile` |
| A **pinned browser harness** for the CSP delivery checks | [06 §10.2](06-azure-hosting-recommendations.md) considers a pinned browser-automation stack, rejects it on proportionality, and selects a blocking, manually executed deployed-browser network-panel gate with a signed-off artifact per release. A gate that produces no file has no row here |
| **Further release-time console projects** | §12.4's single-console decision: every operator action is a verb of `tools/provision-admin` rather than a project of its own, so the map names one console, one publish output, one checksum and one promotion path |
| `Properties/launchSettings.json` | Refused on the scope rule §12.1 states, for the reason the paragraph below gives |

**One artifact a reader may expect here and will not find: `Properties/launchSettings.json`.** The IIS
Express settings and the `ProjectExtensions` web-project block are dropped (§5.4) and **nothing in this
map replaces them**. The reason is a scope rule rather than a technical one: **the authoritative future
application map has no launch-profile row, and this map carries that map's groups rather than adding to
them** — a local launch profile is developer-machine configuration, it is not a build input, it is not
deployed, and no approved amendment adds it. Local running therefore needs no committed artifact: with no launch profile present,
`dotnet run` binds Kestrel's default local endpoints, and a developer who wants different ones sets
`ASPNETCORE_URLS` in their own environment. If a team later decides to commit shared launch profiles, that
is an amendment to the map and belongs in the approval that authorizes it, not in this strategy.

#### 12.2.1 The placement rules the fifteen designed-type rows follow

**The designs the other deliverables have written since the plan was frozen require types that no file on
the authoritative map was named to hold, and a type with no containing file is exactly the unowned target
§12.1 forbids.** Every one of those types is therefore **a row of §12.2 above**, each citing the
deliverable that requires it, and this sub-section states the rules those rows follow so that a reader can
predict a placement rather than look it up. **The rows it governs are fifteen, over forty-one files.**
**Rules one and four hold for all fifteen; rule two arises only where a folder had to be chosen**, which is
fourteen of them, because the fifteenth row's nine files land in folders the authoritative map itself
establishes and their owners have already said which. **One post-freeze type sits outside these fifteen and
is placed by the same first and fourth rules**: the alias entity
`src/MvcMusicStore/Models/CartAlias.cs`, which §12.1's consuming-deliverable row records rather than these
fifteen, because it is a **catalog entity type** of the model the six ported entity classes belong to
rather than a type in one of the five role folders rule two chooses among — so no folder had to be chosen
for it either, and the count of fifteen is unchanged by it. **This sub-section carries no table of its own:
the map is the one table above.** The division of labour is unchanged and is what makes the placement this
document's to fix rather than a duplication of anyone else's: **the owning deliverable decides that the
type exists, what it does and how it is registered; this map decides which file it lives in and where that
file sits in the project.**

Four placement rules are applied uniformly, so a reader can predict the answer rather than look it up:
**one public type per file, named after the type**; folders by role — `Health/`, `Diagnostics/`,
`Identity/`, `Security/`, `Configuration/`; **options types together under `Configuration/`**, one per
bound configuration section and named after the type its owner named, because their common property is that
startup binds and validates them rather than any subject they share with their consumers; and **no new
project** — every file these fifteen rows carry compiles into the web application by implicit globbing,
with no `Compile` item and no project of their own (§5.1, §12.5).

**The first rule says *public* type deliberately, and `Diagnostics/InternalDiagnosticsEndpoints.cs` is the
row that establishes the latitude.** It declares the three authenticated diagnostic operations' fixed
response shapes as **nested private records**, not as files of their own, because they are a wire format
with exactly one reader: nothing outside those handlers constructs one, returns one or names one in a
signature, and a file of its own sitting in `Diagnostics/` would read as a shared contract that other code
is invited to depend on. Nested and private, they cannot acquire a second consumer without the declaration
moving first, which is the property worth having. **The latitude is a permission and not a rule, and the
report collector is the case that shows the difference**: its two CSP report shapes are **not** nested
privately, because [05 §7.5](05-aspnet-core-migration-approach.md) decides otherwise, in its own words
rejecting nesting them "because the two shapes are a wire contract and a wire contract belongs beside the
other one" — the other one being the checkout input model — so they are deserialization targets in
`Binding/CspReportModels.cs`. The
latitude is therefore not an instruction to nest: the owning deliverable decides whether a shape is private
to one reader, and where it says otherwise this map places the file it named rather than the file this rule
would have predicted. `Diagnostics/InternalDiagnosticsEndpoints.cs` is
therefore **the only** row that uses the latitude, and it adds no file.

**Five co-locations, sanctioned explicitly rather than left as gaps.** Each is a case where a reader
might expect a file and the correct answer is a file already on the map:

- **Every registration of every type on these fifteen rows is a
  line in `src/MvcMusicStore/Program.cs`**: the health
  check, the exception handler, the three credential-verification types, **the framework's concrete
  `PasswordHasher<ApplicationUser>` that the rehash decorator depends on** — a registration with no file,
  whose shape [09 §6.8.1.1](09-security-assessment.md) seam 1 owns — the user manager through
  `AddUserManager<T>`, the authorization result handler, the rate limiter chain, every options
  binding, the singleton readiness holder, the `AddHostedService` call for each of the two long-lived loops
  and for the key-ring bootstrap, the host-role signal, the singleton report sink, the
  `MapPost("/csp-report", …)` endpoint and the three `MapGet`/`MapPost` registrations for the
  authenticated diagnostic operations. The composition root is one file by decision
  (§12.2's `Program.cs` row); a partial-class split or an extension-method file per feature is not
  specified, and [05 §2.4](05-aspnet-core-migration-approach.md) owns the order they appear in. The one
  registration that is deliberately **not** here is the test-mode connection signal's: its only
  implementation is in `src/MvcMusicStore.Tests` and the reference runs test → application, so the
  published application carries no descriptor for it ([05 §3.3](05-aspnet-core-migration-approach.md)).
- **`ErrorViewModel.cs` stays where §12.2 puts it**, at the project root rather than under
  `Diagnostics/`, because the authoritative map names that path and §12.2 adds files rather than
  moving them.
- **The cart-migration notice is a constant on a mapped type, not a resource file.**
  [05 §12.9](05-aspnet-core-migration-approach.md) requires the rendered notice to be byte-equal to a value
  the assertion reads **from the application** rather than retypes, and a `const string` on the type that
  renders it satisfies that exactly: the view and the assertion read one declaration, so neither can drift
  from the other. A `.resx` would satisfy it too and is **not** taken, because the plan's web-project
  artifact list carries no resource file and this map adds none (§12.1). The scope is deliberately narrow —
  one constant, no resource strategy, and localization is neither in the plan nor proposed here.
- **The audit event classes have no type of their own.** [09 §6.8.1](09-security-assessment.md) emits
  them through `ILogger` with reserved categories and closed field sets, and their producers are the
  controllers, the two seams above and the operator command — so there is no event-catalog type, no
  emitter service and no file for one. A reader looking for `AuditEvent.cs` will not find it, and that
  is the design rather than an omission.
- **A conditional error layout is not a row of its own.**
  [05 §8.3](05-aspnet-core-migration-approach.md) permits either `@{ Layout = null; }` declared in the
  error view or a dedicated `Views/Shared/_ErrorLayout.cshtml`; if the second is taken, the file is a
  member of the `Views/**/*.cshtml` row's membership rule — which §12.1 states as 27 files under the
  first choice and 28 under the second — rather than a new row in §12.2. Either choice satisfies the same
  requirement, and neither changes these fifteen rows or the row count above.

#### 12.2.2 The audit exporter — the one required process this map carries no file for, and why

**The requirement, and the decision that settles its artifact.**
[06 §9.5](06-azure-hosting-recommendations.md) requires a scheduled process that copies rows out of
`dbo.SecurityAuditLog` into the immutable audit store, and specifies that process **completely as
behaviour**: its two identities and their separate grants, its cursor blob in a second container, how it
selects a batch, the deterministic batch name, the SHA-256 content hash it writes, the four-step
conditional-create acknowledgement protocol that makes a re-run idempotent, what it may and may not log,
and four failure tests. Behaviour alone does not settle the **artifact**: a process specified that way
could land either inside this repository or outside it, and the two
outcomes differ in whether this repository gains a project at all. **That decision has been taken by
its owner**, and this sub-section records it rather than reopening it: 06 decides the form — a `net8.0`
console application packaged as a container image and run as a scheduled job under the monitoring identity
— and the **location: outside this repository's application tree**, alongside the audit storage account and
the private endpoint, neither of which lives in this tree either.

**What that means for this map, stated as an absence rather than left implicit.** 06 states the consequence
in the same terms this map uses — *"04's map gains no file in this tree"* — so **§12.2 carries no row for
the exporter**: no project row, no fourth solution path, no fifth `packages.lock.json` and no §7.2 pins,
because the two client libraries the exporter needs are its own definition's to pin. A required process
with no artifact in this tree is exactly the case the `Dockerfile` row's conditionality and the withdrawn
browser harness illustrate: **this document maps files, and there is no file here to map.** The removal is
recorded in §12.2's count rationale so the row's disappearance is auditable rather than silent, and the
requirement itself is not lost — it is owned, specified and scheduled by 06.

**What would reopen it, and the exact blast radius if it did.** The location decision closes *which*
repository or infrastructure definition holds the exporter at the **same gate as CI provider selection**,
and 06 records that gate and its owner. If that decision is ever reversed and the exporter comes in-tree,
this map changes in exactly three bounded ways and no others: **§12.2 gains one row** for
`tools/`-or-equivalent `net8.0` console project, **the lockfile row gains a fifth file** because §6.4 is
per project, and **§7.2 gains rows for the two package families** it needs — a SQL client to read the table
under the reader identity and a blob client to write the batch and maintain the cursor under the writer
identity. It would add **no** edge to §12.4's graph, because it references nothing in this solution and
reaches SQL and blob storage over the network exactly as the exporter contract describes, so the graph
would gain a node and not an edge. Nothing else on this map is held up by that decision, which is what
makes it a location decision rather than a design one.

**The key-ring monitor is not a second such process, and the heading above says "the one" advisedly.** A
**key-ring monitoring job** is not an out-of-tree process beside the exporter, because its owner's design
no longer contains one:
[06 §7.4.1](06-azure-hosting-recommendations.md) deletes the recurring job outright — *"the query survives
and the monitor around it does not"* — because its producer would have been a second answer to a question
[06 §7.4](06-azure-hosting-recommendations.md) already answers through the heartbeat. What replaces it
splits across this map's two categories and leaves nothing in between: the **producer is in-tree and is now
a row of §12.2** — `Services/KeyRingStatusHeartbeatService.cs`, registered by
[05 §2.4](05-aspnet-core-migration-approach.md)'s `AddHostedService` — and what remains outside is an
**operator-invoked diagnostic query** with no recurrence, no threshold and no action group, which is a `SELECT`
an operator runs on in-network compute rather than an application whose files a map could carry. So the
exporter is the **only** required process this map carries no file for, and the monitor is not an exception
to the enumeration but an ordinary member of it.

**No artifact is mapped for [06 §10.2](06-azure-hosting-recommendations.md)'s report-endpoint delivery
checks, and a pinned browser harness for them is refused rather than pending.** The case for one was
real — CSP enforcement, CSP3 precedence between the two report transports and the absence of
double delivery are behaviours of a browser's policy engine that no `WebApplicationFactory` request can
produce — and that reasoning still holds, but the decision it waited on **has been taken, against the
harness**: 06 weighs a pinned browser-automation stack, rejects it on proportionality, and selects a
blocking, manually executed deployed-browser network-panel gate with a signed-off artifact per release. A
gate that produces no file has no row on the map, so the row is removed, §12.2's count rationale records the
removal, and the requirement is met without an artifact in this tree.

### 12.3 Three project-structure decisions

**The ported application is a new sibling, not a replacement in place.** It is created at
`src/MvcMusicStore/` rather than overwriting `src/MVC5/MvcMusicStore/`. The reason is validation, not
sentiment: MVC 5 is the reference implementation of every behaviour the port must preserve, and the
behavioural baseline is captured by driving the *running* MVC 5 application — which requires its source,
its project file and its committed databases to remain exactly as they are while the port is built and
compared against them. Replacing it in place would destroy the only reference the port can be checked
against, and the repository has no test suite to fall back on (appendix A.4).

**The three legacy projects are retained read-only.** `src/MVC3/`, `src/MVC4/` and `src/MVC5/` stay in the
repository as **historical and comparative references**. None is modified and **none is
deleted**. Only MVC 5 is ported; MVC 4 and MVC 3 are assessed in full and carried forward untouched, which
also means all **four legacy solutions and three legacy project files continue to exist** alongside the one
new root solution of §5.6. Every solution and project statement in this document is additive: the target
gains a solution, and the past loses nothing.

**Within that set, one edition has a role the other two do not: MVC 5 is the sole executable behavioural
baseline.** It is the only edition the suite drives — the paragraph above is about *its* source, project
file and committed databases staying exactly as they are — and **neither MVC 4 nor MVC 3 is driven by the
suite at all** ([05 §12.10](05-aspnet-core-migration-approach.md) assigns every baseline execution to the
running MVC 5 application). Retaining all three is worth doing for a different reason, and the two must not
be conflated: MVC 4 and MVC 3 are read for **comparison** — the duplication measurements, the two cart
unit-of-work models, the per-edition dependency and security posture — and nothing in this assessment
executes them or compares the port against them.

**The suite is two projects, and the one copy of each assertion is bound to both baselines by inheritance
across the reference between them.** §12.2 maps `src/MvcMusicStore.Contracts.Tests` and
`src/MvcMusicStore.Tests`, and §12.1's record 2 states why the split is what makes the plan's own ordering
executable: the contracts project references nothing, so it builds and runs against the **legacy**
application before the ported web application exists. The arrangement that lets one copy of an assertion
run against both applications therefore has to work **across two assemblies** — and inheritance does, with
each assembly declaring its own concretes. That has to be stated rather than assumed, because two readings
of a `ProjectReference` are wrong in ways that produce a suite which passes by running nothing.

> **A `ProjectReference` does not make the referencing project a test project — and this is the trap that
> produces no run at all rather than a short one.** `Microsoft.NET.Test.Sdk` and
> `xunit.runner.visualstudio` do their work through **build assets** — MSBuild `props`/`targets` that mark
> the project as a test project, import the test targets `dotnet test` invokes, and copy the discovery
> adapter beside the test assembly — and `xunit` additionally ships **analyzer** assets. A
> `PackageReference`'s **build, analyzer and content assets do not flow to a project that references the
> declaring project**: that exclusion is `PrivateAssets`' default for exactly those asset kinds, so only
> compile and runtime assets cross a project reference. A project that acquired those three only by
> referencing a project that declares them would **compile**, and then be neither built as a test project
> nor discovered nor executed: the adapter is absent from its output and the test targets are never
> imported. **Therefore each of the two test projects declares `xunit`, `xunit.runner.visualstudio` and
> `Microsoft.NET.Test.Sdk` directly**, in its own project file, and §12.2's two project rows do — the
> redeclaration in `src/MvcMusicStore.Tests` is required rather than redundant, because it acquires nothing
> of those three across its reference to the contracts project.
> Library pins behave the other way round: `AngleSharp`, `Microsoft.Data.SqlClient` and
> `Microsoft.Extensions.Identity.Core` deliver compile and runtime assets, so §7.5, §7.6 and §7.8 declare
> each in `src/MvcMusicStore.Contracts.Tests` and only there, and the reference carries them to the other
> project. `Microsoft.Playwright` sits with the build-asset group rather than the library
> group, so it too is declared in both, for the reason §7.7 states.
>
> **And a `ProjectReference` does not make the referenced assembly's tests run either — this is the reading
> that decides where the concretes are declared.** The test host is pointed at **one** assembly at a time
> and discovers tests by reflecting over **the types that assembly
> declares**; a project reference makes the referenced assembly's *types* available to the compiler and
> puts its output beside this one's, and no filter or runner setting enrols its tests in this run. So
> running `src/MvcMusicStore.Tests` does **not** run anything declared in
> `src/MvcMusicStore.Contracts.Tests`, and running the contracts project does not run the target-bound
> cases: each assembly is discovered and executed on its own, which is why every concrete class is declared
> in the assembly whose run is meant to execute it rather than inherited across the reference. The web
> project and the operator command declare no tests and are not test projects, so the two test assemblies
> are the whole of what `dotnet test` discovers.

The property that *is* load-bearing is different, and it is the one the topology is built on: **a test
method inherited by a class declared in the test assembly is discovered on that derived class**, because
discovery enumerates each declared type's methods including the inherited ones. So the shape is fixed, and
it spans the two test projects §12.2 maps — the element table below states which element is declared in
which, and §12.2's own rows are the authority for every path in it:

| Element | Where it is declared | What it carries |
| --- | --- | --- |
| One `public abstract` class **per contract surface** | `src/MvcMusicStore.Contracts.Tests/Contracts/**` | Every `[Fact]` and `[Theory]` for that surface, and nothing runtime-specific. Its dependencies arrive through an **injected, runtime-neutral context**. Being abstract, it is skipped by discovery and contributes no run of its own |
| One `public sealed` **legacy-bound** concrete per surface | `src/MvcMusicStore.Contracts.Tests/Legacy/**` | A derivation, the legacy context and its `[Collection]` attribute. No assertion logic. These are the classes the pre-port baseline run discovers and executes — in the same assembly that declares the bases, which is what lets that run happen before the web project exists |
| One `public sealed` **target-bound** concrete per surface, plus the **target-only** classes that have no legacy counterpart | `src/MvcMusicStore.Tests/Core/**` | A derivation of a base declared in the *other* assembly, the target context — the in-process host's client and the target-side observer — and its `[Collection]` attribute. No assertion logic. These are the classes the post-port run discovers and executes |
| One `public` **collection-definition class per surface group, in each assembly that declares a class in that group** | `src/MvcMusicStore.Contracts.Tests/Collections/*.cs` **and** `src/MvcMusicStore.Tests/Collections/*.cs` | `[CollectionDefinition]` with the group's name, `ICollectionFixture<TFixture>` naming **the fixture type that group's classes bind**, and a `const string` holding the name. **No test methods.** The groups are [05 §12.7](05-aspnet-core-migration-approach.md)'s nine, plus the deployed group of the row below. The definitions are **per assembly, not per suite** — a `[Collection]` name resolves within the assembly being run, so each of the two projects carries the set for the groups its own classes are in (§12.2's two `Collections/*.cs` rows) — and a definition names **one** fixture type, so within the contracts project, which declares two of the three fixture types, the definitions do not all name the same one |
| The **`public` collection-fixture type** each definition binds | `src/MvcMusicStore.Contracts.Tests/LegacyBaselineFixture.cs` and `src/MvcMusicStore.Tests/CoreApplicationFixture.cs` — each in the assembly whose definitions bind it | One instance per collection, so the nine groups get nine databases rather than one shared engine state. What each fixture *does* — provisioning, the two-database legacy reset, `ResetAsync()` and the per-test `IAsyncLifetime` binding — is [05 §12.7](05-aspnet-core-migration-approach.md)'s and is not restated here |
| The **`public sealed` deployed-only pair** — a third fixture type and one concrete — `DeployedEndpointFixture` and `DeployedEndpointTests` | Both at the root of `src/MvcMusicStore.Contracts.Tests/`, with their `[CollectionDefinition]` beside that project's others | The one runtime-neutral context in the topology that **hosts nothing**: no in-process host, no engine, no database, no observer and no reset — a **consumed base address** and a client whose **redirect following is disabled**. The concrete holds its cases directly rather than deriving from a base, because there is no legacy counterpart to share one with. It exists because the one `Category=Deployed` case runs against a **deployed** host, which is a context no row above can produce; its execution stage, its authoring workstream and its assertions are named in §12.2's test-project row and in [05 §12.6](05-aspnet-core-migration-approach.md) |

**Every type in that table is `public`, and that is a requirement rather than a house style.** The
two-assembly topology puts **one** class of mistake in front of the compiler and leaves the rest silent, so
the accessibility is fixed here rather than left to whoever types the first class. The one the build
catches is the derivation across the reference: a concrete in `src/MvcMusicStore.Tests` cannot derive from
a base, or bind a collection fixture, that `src/MvcMusicStore.Contracts.Tests` declares `internal` — which
is §12.4's second visibility change, and it fails loudly at build time. Everything else fails **silently**,
and nothing in the build catches either mistake — including on a type used only within the assembly that
declares it, where no cross-assembly derivation exists to raise the error:

- **The runner discovers test classes only when they are public.** A concrete declared `internal`
  compiles, is never enumerated, and reports nothing — a green run of zero cases, arriving through
  accessibility.
- **Collection definitions and the fixture types they name are located and activated by reflection, and
  the documented shape for both is a public type.** Declaring either otherwise makes the binding depend on
  unspecified runner behaviour, and its failure mode is a fixture that is never attached — which surfaces
  as a null context inside an assertion rather than as a wiring error.

Two further types take the same requirement for the second reason above, because a fixture reaches them by
reflection or holds them across a collection: the **runtime-neutral context** the bases take, and the
`IStoreObserver` abstraction whose two implementations are the legacy-bound and target-bound halves
([05 §12.6](05-aspnet-core-migration-approach.md)).

**Each concrete carries `[Collection]` naming its surface group, and the name comes from a constant rather
than a literal.** Two failure modes make that worth fixing at the declaration site. A concrete with **no**
`[Collection]` attribute is its own collection, so it receives **its own fixture instance** and — under one
database per collection — its own database, which multiplies provisioning by the number of classes and
dissolves the grouping [05 §12.7](05-aspnet-core-migration-approach.md) chose. A concrete with a
**mistyped** name does exactly the same thing, and neither errors. So the definition class exposes the name
as a `const string` and every concrete in the group references that constant, which turns both mistakes
into compile errors.

**Exactly one copy of every contract assertion exists, and it is the abstract base.** The two concrete sets
are bindings, not copies: a surface gains a base and two derivations, so an assertion added or corrected in
the base changes both sides at once, which is the property the cross-baseline comparison depends on and the
property that duplicating the assertions would destroy. The count is therefore *one base plus two thin
concretes per surface*, and a reviewer can check it by looking for assertion text outside a base class —
there should be none.

**What the split costs, stated rather than discovered.** It buys the property §12.1's record 2 and §12.4
turn on — the legacy-bound half compiles and runs before the ported web application exists, because the
concretes that talk only to the *legacy* application over HTTP are in the assembly that references nothing
— and it is paid for in three places, each of them a §12.2 row rather than a hidden cost: the three
test-execution pins and `Microsoft.Playwright` are declared **twice**, once per runnable project, because
build and analyzer assets do not cross a project reference; each project carries its **own**
`Collections/*.cs` set, because a `[Collection]` name resolves within the assembly being run; and each
carries its **own** `xunit.runner.json`, because collection parallelism is an assembly-level setting. Only
`src/MvcMusicStore.Tests` is held to the project graph of §12.4, which is where that consequence is derived
and where the deliverable that sequences around it is named.

Which classes each stage runs, and how they are selected, is not this document's to specify: the trait
categories and the two commands are [05 §12.10](05-aspnet-core-migration-approach.md)'s, and
[03](03-modernization-roadmap.md)'s W4 and W7 gates name the stage each belongs to. What this section fixes
is the declaration site of every class involved, because that is a project-structure decision and the
map above is where project structure is settled.

### 12.4 The project graph — the three edges and the two visibility changes without which the solution does not work

§5.6 and §12.2 name four projects. Four projects in one solution are not a graph, and no consumer reaches
the project it consumes by being in the same solution: a `ProjectReference` and a type visibility are what
make each one compile. Both are project-structural facts, so this document owns them; what the fixture and
the command *do* is [05](05-aspnet-core-migration-approach.md)'s.

| Edge | Declared in | Element |
| --- | --- | --- |
| `MvcMusicStore.Tests` → the web application | `src/MvcMusicStore.Tests/MvcMusicStore.Tests.csproj` | `<ProjectReference Include="..\MvcMusicStore\MvcMusicStore.csproj" />` |
| `MvcMusicStore.Tests` → `MvcMusicStore.Contracts.Tests` | `src/MvcMusicStore.Tests/MvcMusicStore.Tests.csproj` | `<ProjectReference Include="..\MvcMusicStore.Contracts.Tests\MvcMusicStore.Contracts.Tests.csproj" />` |
| `tools/provision-admin` → the web application | `tools/provision-admin/ProvisionAdmin.csproj` | `<ProjectReference Include="..\..\src\MvcMusicStore\MvcMusicStore.csproj" />` |
| `MvcMusicStore.Contracts.Tests` → anything in this solution | — | **None, by design.** It is the reference-free project, which is what makes the pre-port baseline runnable before the ported web application exists at all |
| The web application → anything in this solution | — | **None.** The web project references none of the other three |
| The operator command → either test project | — | **None.** The edges are one-way, so the command's output contains no test assembly |

**The second edge exists for exactly one reason, and the reason is a suite that has to run before the
project it will eventually target exists.** The contract classes and collection fixtures live in
`MvcMusicStore.Contracts.Tests`, which references nothing: the pre-port baseline is captured by driving the
**legacy** application over HTTP, so those cases must compile and run while `src/MvcMusicStore` is still an
empty skeleton or absent. `MvcMusicStore.Tests` is the project that adds the first edge and therefore cannot
compile until the web application does, so putting the contract bases there would make the baseline
un-runnable exactly when it is needed. The edge points from the constrained project to the free one:
`MvcMusicStore.Tests` derives its target-side cases from the same contract bases the baseline used, which is
what makes the two sets of results comparable rather than merely similar.

**What the edge costs, stated rather than discovered.** Nothing measurable. Both projects target `net8.0`
and both already reference the test SDK and runner (§7.2), so the added edge brings **no new framework
reference and no new package** — it brings the contract types, which is the point. It creates no cycle
either: `MvcMusicStore.Contracts.Tests` references nothing, and nothing but `MvcMusicStore.Tests`
references it.

**Why there is no edge from a test project to the operator command, and what that costs.** The dispatcher
below refuses an argument containing any control character, `U+0000` among them, and one member of that
class is **not assertable at all**. **A NUL cannot be delivered through a process boundary**: Windows
`CreateProcess` takes a single NUL-terminated command line and the POSIX `execve` family takes a
NUL-terminated `char *` per argument, so on either platform the first NUL byte in an argument *is* the
terminator and the character never reaches `Main(string[] args)` to be refused. That cuts both ways, and the
second way is the one that decides the graph: **no operator invocation can deliver a NUL either**, so the
scan's coverage of it is a **closed-set property rather than a reachable case** — `char.IsControl` is true
for `U+0000` because the framework's definition of the class includes it, not because this tool expects one.
An in-process call to the dispatcher would demonstrate that property and would cost a fourth
`ProjectReference`, a public surface on a privileged tool's argument parser, and a test asserting a behaviour
no caller can trigger. It is not taken. Every control character argv *can* carry — `\r`, `\n`, the ANSI
escape introducer, `\u007f`, the C1 range — is asserted the ordinary way, at the process boundary, by
launching the built tool (assertion 2 below), which is where an operator's input arrives.

**The direction still matters as much as the existence.** No edge points *at* `MvcMusicStore.Tests`, and none
points away from the web project, so `dotnet publish` of the web project produces output containing neither
test project nor the operator command, and `dotnet publish` of the operator command produces output
containing no test assembly — which is what makes "**not** deployed with the web application"
(§12.2, and the requirement in [05 §10](05-aspnet-core-migration-approach.md)) a property of the project
graph rather than a deployment convention someone has to remember. There is also no cycle to resolve.

**The first visibility change: `Program` must be public for `WebApplicationFactory<T>` to name it.** The
fixture is generic in the application's entry point, and with top-level statements the compiler emits the
generated entry-point class as **`internal`** in the global namespace — so `WebApplicationFactory<Program>`
in another assembly does not compile. Two mechanisms resolve it, and this document picks one:

- **Chosen: a public partial declaration in `Program.cs`.** A single `public partial class Program { }`
  declared after the top-level statements in `src/MvcMusicStore/Program.cs` raises the generated class to
  public without exposing anything else. It adds no file — which matters, because the map in §12.2 is the
  authoritative artifact list and a new file would have to be an amendment to it — and it is visible at
  the place a reader looks for the entry point.
- **Rejected: `<InternalsVisibleTo Include="MvcMusicStore.Tests" />` in the web project.** It works, and it
  is the alternative a reviewer will ask about, but it grants the test assembly access to *every* internal
  type in the application in order to expose one. The suite is deliberately black-box over HTTP
  ([05 §12](05-aspnet-core-migration-approach.md)), so the broader access buys nothing and quietly permits
  tests that reach past the HTTP surface.

**The second visibility change: the contract bases and collection fixtures must be public in
`MvcMusicStore.Contracts.Tests`.** A class in another assembly cannot derive from an `internal` base or bind
an `internal` collection fixture, and a class library makes nothing public by default, so the types
`MvcMusicStore.Tests` inherits across the second edge have to be declared `public` where they are written.
The same two mechanisms are available as for `Program` and the same one is chosen, for the same reason:

- **Chosen: the contract base classes and the collection-fixture definitions are declared `public`, and
  nothing else in that project is.** Modifiers in files §12.2 already maps, and no new file. What becomes
  visible is exactly the surface the derivation needs and no more: the per-case bodies, the normalization
  helpers and the fixture internals stay **internal**, because the derived class needs the base's protected
  and public members rather than its implementation.
- **Rejected: `<InternalsVisibleTo Include="MvcMusicStore.Tests" />` in the contracts project.** It works,
  and it would let every type there stay internal, but it makes the *inheritable* surface indistinguishable
  from the incidental one — a later reader cannot tell which types the other assembly is meant to derive
  from, and any new internal becomes visible to it silently. `public` on the intended set states the
  boundary in the code, so the narrower change is the clearer one as well.

**Nothing in the operator command is made public, and that is a consequence of the paragraph above rather
than an omission.** With no edge from a test project to `tools/provision-admin`, `Cli`, its `Dispatch`
method, the exit-code constants, the token tables, the `Refusal` enumeration and `Refuse` are **all
internal** to the tool. The refusals are asserted the way an operator observes them — the process's exit
code and its captured `Console.Error` — which is a published contract that needs no type exposed to state
it, and it is the same contract [05 §10.2](05-aspnet-core-migration-approach.md) owns.

**Why the `ProjectReference` from the test project is required rather than merely convenient.** Beyond
naming `Program`, the fixture has to find the web application's **content root**, because Razor views are
resolved relative to it and the test process runs from its own output directory. `Microsoft.AspNetCore.Mvc.Testing`
(pinned at `8.0.30`, §7.2) ships build targets that emit a content-root manifest for the web projects the
test project references, and `WebApplicationFactory` reads it. Without the reference there is no manifest
entry, and the fixture falls back to guessing a path — the failure mode being views that cannot be found
at run time rather than a compile error.

**The sequencing consequence, which follows from the structure rather than from anyone's plan.** Every
requirement above is a property of the **test project**, not of any individual test: without its two
`ProjectReference` elements, without a `Program` it can name and without public contract bases it can derive
from, `MvcMusicStore.Tests` does not compile at all, whatever it contains. So **no part of *that* project can
be compiled before this project graph exists** — while the rows that only ever talk to the *legacy*
application over HTTP live in `MvcMusicStore.Contracts.Tests`, which references nothing and therefore
compiles and runs before the web project does. That split is the whole reason the second edge exists, and it
is what keeps a compilation dependency from becoming a scheduling one for the baseline. This document contributes that structural fact and states no sequence of its
own; [03 §4.2](03-modernization-roadmap.md) draws the corresponding edge and
[03 §5](03-modernization-roadmap.md)'s W4 and W6 gates state what it does and does not imply — in
particular that it is a compilation dependency and **not** a reinstatement of any legacy-behaviour claim
against the skeleton, which remains unmeetable because the migration source's `System.Web` API surface
cannot compile on `net8.0` at all ([12 §3](12-migration-blockers.md)'s compile-time group).

**How the operator command reaches service registration and configuration: its own host, not the web
application's.** `tools/provision-admin` is a console project, and its composition is **narrower than the
web application's in what it resolves and stricter in what it admits**.
[05 §10.2](05-aspnet-core-migration-approach.md) property 2 delegates this composition here, so it is
specified in full below; what the command *does* with the services it resolves stays that document's.

**The host disables the framework defaults, and that is a security requirement rather than a preference.**

```csharp
var cli = Cli.Dispatch(args);                              // the closed dispatcher below
if (cli is null)
{
    AuditRecords.EmitRejection(args, Cli.RefusedCheck(args));   // four lines to stdout, pre-host
    return Cli.Rejected;                                   // refused: no verb ran, no store write
}

var actor = Env.AuditActor();                              // MUSICSTORE_AUDIT_ACTOR, read by name
if (cli.Value.Verb == Cli.Admin
    && string.IsNullOrWhiteSpace(actor))
{
    AuditRecords.EmitRejection(args, "actor-missing");     // 05 §10.2 property 4's actor check,
    return Cli.Rejected;                                   // that verb's; pre-host, so no store write
}

var builder = Host.CreateEmptyApplicationBuilder(new HostApplicationBuilderSettings
{
    EnvironmentName = Env.EnvironmentName(),   // absent => "Production", which fails closed
});

builder.Configuration
    .AddInMemoryCollection(Defaults.FailClosed)   // precedence 1 — see the table below
    .AddInMemoryCollection(Env.Admitted())        // precedence 2 — two variables, by exact name
    .AddCommandLine(cli.Value.Pairs);             // precedence 3 — the dispatcher's pairs only

// The verb decides where its switches are read, and the two answers do not overlap.
//   admin | seed        -> Pairs is what AddCommandLine received above; Residual is empty.
//   the eight data verbs -> Pairs is empty, and Residual is handed to that verb's entry point
//                           UNPARSED, to be read by the grammar 05 §5.7 owns.
var (verb, _, residual) = cli.Value;
```

**`AuditRecords.EmitRejection` is named in the sketch because the two returns above happen before the host
exists, and an audit record that cannot be written on that path is a contract this document would otherwise
break.** [05 §10.2](05-aspnet-core-migration-approach.md) property 4 fixes the cardinality at **four records
per invocation invariantly, this pre-host path included** — three operation records carrying
`outcome = not-attempted`, then the run record carrying `outcome = rejected` and its `rejectedCheck`, in
that fixed order with the run record last — and §12.6's refusal table above owes them on every `admin`
refusal. **The two calls above are what produce those four records on a path where no host, no container
and no configured logger exists yet**, and what they call is fixed by the owning deliverable rather than
invented here:

- **What writes them, with no host, no container and no configured logger.** Property 4 already names the
  mechanism and the reason — a **dedicated static serialiser over a dedicated record type**, one
  `System.Text.Json` call with fixed options, *"because the record has to survive a failure that happens
  before the host exists"*, and explicitly **not** an `ILogger` call. So `AuditRecords` resolves nothing:
  it takes no `IServiceProvider`, reads no `IConfiguration`, holds no logger, and depends only on the
  framework's own JSON serializer and the raw arguments. It is reachable from the first executable line of
  `Program.cs`, which is the property the two call sites need and the only one this document adds.
- **Where it lives, so the map does not move.** It is a **static type declared in the console's own
  `Program.cs`** — the file §12.2's `tools/provision-admin` row already names — so this correction adds
  **no file, no row and no project**, and §12.2's count is unchanged by it. A file of its own would be
  defensible and is not taken, because a type with two call sites in the same file is exactly the case
  §12.2.1's one-public-type-per-file rule leaves to the declaring file.
- **Where the four lines go, and what happens if one is lost.** Standard output, one object per line, four
  lines, run record last — property 4's destination, and the same channel
  [06 §9.5.1](06-azure-hosting-recommendations.md)'s provisioning stage captures into the immutable
  container. The run record being last is what makes a truncated capture detectable, so a write that fails
  part-way through the set is a **failed invocation** rather than a silent short set: the command exits
  non-zero and the stage fails, which is the same posture §12.6 takes for every other unwritable audit
  artifact. It is never traded for a store write — nothing was attempted on either of these paths, so there
  is no work whose success could outlive a missing record.
- **Which refusals owe a record, and which owe none.** The `admin` verb's do, including the
  bad-switch class the first call site covers and the missing-actor check the second one names, and
  `Cli.RefusedCheck(args)` supplies the specific check that becomes `rejectedCheck` rather than a generic
  literal — the granularity [05 §10.2](05-aspnet-core-migration-approach.md) requires when it says the
  specific check is named in the record. A refused **`seed`** invocation owes none, that class being a
  privilege-grant record the seed verb never produces (§12.6), so the emitter is a **no-op for any verb
  token other than `admin`** — determinable from the presented tokens without a successful parse, which is
  what lets the first call site sit on the `cli is null` path at all. On the missing-actor path the `actor`
  field carries property 4's reserved `(unavailable)` literal and `rejectedCheck` carries `actor-missing`;
  the field vocabulary, the sentinel's legality conditions and every other value are that document's and
  are not restated here.

This is a code-sketch correction and not a change of owner: the cardinality, the field set, the order, the
vocabulary and the destination remain [05 §10.2](05-aspnet-core-migration-approach.md) property 4's, its
**O7** oracle already asserts *"exactly four per invocation, in the pinned operation order, on all four
paths"* including *"a rejection **before any host is built** (a bad switch, and a missing actor)"*, and
§12.6's refusal table already cites that property rather than restating it. Nothing downstream is
renumbered, and no consumer changes.

**Where `Residual` goes, because a value returned and never consumed is a specification gap.** The
composition above is the whole of what this document owes for the two verbs it parses: `admin` and `seed`
read their switches from the configuration root, and their `Residual` is empty by construction. For each of
the eight data verbs the dispatcher returns an **empty pair array and an untouched `Residual`**, and the
composition root passes `Residual` through to that verb's entry point exactly as received — no filtering, no
normalization and no configuration key derived from it — where
[05 §5.7](05-aspnet-core-migration-approach.md)'s own closed table parses it. That is the hand-off in both
directions: this file guarantees the tokens arrive unmodified and that none of them reached a configuration
provider, and that document guarantees what each one means. A `Residual` this file interpreted, or silently
dropped, would put two grammars on one verb — which is the drift the split exists to prevent, and it is why
required assertion 5 asserts the empty-pair, untouched-`Residual` outcome once per data verb rather than
once overall.

**Precedence 2 is an exact key allow-list and not a prefix, because a prefix is an open door with a
narrow frame.** `AddEnvironmentVariables(prefix: "MUSICSTORE_TOOL_")` admits **every** variable carrying
that prefix, so any name an operator, an agent image, a pipeline definition or an earlier step happens to
export becomes configuration for a privileged command — and `__` translation means
`MUSICSTORE_TOOL_ProvisionAdmin__RotateCredential=true` in the ambient environment resolves the key
`ProvisionAdmin:RotateCredential` at a precedence **above** the fail-closed default. A run that typed no
switch would then replace a working administrator credential, and nothing in the invocation would say
so. Naming the variables instead removes the class rather than the instance:

```csharp
static class Env
{
    // The whole admitted configuration surface. Two variables, matched by exact ordinal name,
    // each the double-underscore spelling of the key it supplies and of nothing else.
    // No prefix scan, no translation, no third entry, and no way to add one at run time.
    private static readonly (string Variable, string Key)[] Admissible =
    {
        ("ConnectionStrings__DefaultConnection", "ConnectionStrings:DefaultConnection"),
        ("Seeding__Enabled",                     "Seeding:Enabled"),
    };

    internal static IEnumerable<KeyValuePair<string, string?>> Admitted()
    {
        foreach (var (variable, key) in Admissible)
        {
            var value = Environment.GetEnvironmentVariable(variable);
            if (!string.IsNullOrEmpty(value))                  // empty is absent, not a value
                yield return new KeyValuePair<string, string?>(key, value);
        }
    }

    // Read directly and deliberately NOT a configuration key, so no configuration source,
    // dump or diagnostic can restate the credential. 05 §10.2 owns the channel and the name.
    internal static string? AdministratorPassword() =>
        Environment.GetEnvironmentVariable("MUSICSTORE_ADMIN_PASSWORD");

    // The audit actor, on the same channel and read the same way: directly by name into a typed
    // field on the parsed invocation, and deliberately NOT a configuration key. It is not a switch
    // at all -- 05 §10.2 property 4 owns the channel and the name, and its property 1b refuses an
    // invocation that passes an actor on the command line.
    internal static string? AuditActor() =>
        Environment.GetEnvironmentVariable("MUSICSTORE_AUDIT_ACTOR");

    // Read directly into HostApplicationBuilderSettings; deliberately NOT a configuration key,
    // so no configuration source can restate the environment the guards of §12.6 test against.
    // DOTNET_ENVIRONMENT is the generic host's own variable and this tool's governing one (§13.3).
    internal static string? EnvironmentName() =>
        Environment.GetEnvironmentVariable("DOTNET_ENVIRONMENT");
}

static class Defaults
{
    internal static readonly Dictionary<string, string?> FailClosed = new(StringComparer.Ordinal)
    {
        ["Seeding:Enabled"]                  = "false",
        ["ProvisionAdmin:RotateCredential"]  = "false",
    };
}
```

**The mode is decided by the switch and by nothing else, and that is enforced twice.** The one
privilege-affecting key — `ProvisionAdmin:RotateCredential` — is
(1) **not on the allow-list above**, so no environment variable of any name can carry it, and
(2) **stated explicitly by the dispatcher on every run**, `true` when the operator typed the switch and
`false` when they did not, at the highest precedence. Either mechanism alone would close the hole; both
are kept because they fail in different directions. The allow-list is what stops an ambient value
arriving at all, and the unconditional emission is what makes precedence decide the question even if some
future source is added below it — a source that would otherwise silently acquire authority over a
credential rotation.

`Env` and `Defaults` are declared in the same place and for the same reason as `Cli` below: **after the
top-level statements in `tools/provision-admin/Program.cs`**, so the whole host — dispatcher, allow-list and
defaults — remains two files, which is what the map in §12.2 lists and what keeps this section from
becoming an amendment to it.

**`args` is deliberately not handed to the builder — and the reason is the opposite of the one it is easy
to assume.** `HostApplicationBuilderSettings` **has** an `Args` property: its full property set is
`DisableDefaults`, `Args`, `Configuration`, `EnvironmentName`, `ApplicationName` and `ContentRootPath`, and
the host builder adds `settings.Args` as a command-line configuration source **independently of
`DisableDefaults`**. Disabling the defaults does **not** disarm it. So the code above is safe **only
because it omits `Args`**, and the rule below is load-bearing rather than a restatement of a framework
guarantee that does not exist:

> **The raw argument array is never assigned to `HostApplicationBuilderSettings.Args`** — not under this
> configuration and not under any other. Doing so would add an unfiltered command line at a precedence
> this section does not control, ahead of both the in-memory defaults and the dispatcher below, which is
> exactly the exposure the dispatcher exists to remove.

Both halves are checkable and were checked rather than inferred: the property set is the
`Microsoft.NETCore.App.Ref` / `Microsoft.AspNetCore.App.Ref` `8.0.30` reference packs' — the servicing band
§7.1 pins — and an empty builder handed `Args = ["--ProvisionAdmin:RotateCredential=true"]` resolves that value
from `builder.Configuration`, defaults disabled or not. The command line is instead added **explicitly,
last**, and only over the argument array the dispatcher below has already closed over.

`Host.CreateEmptyApplicationBuilder` — equivalently `new HostApplicationBuilder(new
HostApplicationBuilderSettings { DisableDefaults = true })` — starts with **no configuration source and no
logging provider**, and every one of both is then added deliberately. The intuitive alternative is
`Host.CreateApplicationBuilder(args)`, whose sources are commonly described as "environment variables and
command-line arguments". **That description is wrong, and the difference matters for this process
specifically**, which is why the defaulted builder is refused here. It
also adds `appsettings.json` and `appsettings.{Environment}.json` **resolved against the
process's current working directory**, user secrets when the environment is Development, `DOTNET_`-prefixed
environment variables, and the raw command line; and it registers the console, debug, event-source and —
on Windows — event-log logging providers. For a process whose one sensitive input is a **plaintext
administrator credential**, that is a live exposure and not a theoretical one: a file left in a pipeline
runner's working directory becomes a configuration source for a privileged command, and a logging provider
nobody asked for becomes a second copy of whatever the command writes, in a sink outside the retained
pipeline artifact [06](06-azure-hosting-recommendations.md) §9.5 defines.

**Admitted configuration sources, in ascending precedence — three, and no fourth.**

| Precedence | Source | What it may carry |
| --- | --- | --- |
| 1 (lowest) | `AddInMemoryCollection(Defaults.FailClosed)` — the tool's own defaults | Two entries and no third: `Seeding:Enabled` = `false` and `ProvisionAdmin:RotateCredential` = `false`. A default that authorizes nothing is what makes an *absent* setting safe rather than merely undefined. For seeding it means no write. For the mode flag it is **defence in depth rather than the deciding source** — precedence 3 states it on every run — and it is retained so that a composition which somehow reached this root without the dispatcher's pairs would still read `false` for it. `RotateCredential` reading `false` is what makes an ordinary release run leave an existing administrator credential alone ([05 §10.2](05-aspnet-core-migration-approach.md) property 3) |
| 2 | `AddInMemoryCollection(Env.Admitted())` — **exactly two environment variables, by name** | `ConnectionStrings__DefaultConnection` → `ConnectionStrings:DefaultConnection`, and `Seeding__Enabled` → `Seeding:Enabled` (§12.6) — both the double-underscore spellings of keys in [05 §3.3](05-aspnet-core-migration-approach.md)'s typed contract rather than second key names. **No other variable reaches configuration**, whatever it is named and whatever prefix it carries, and the mode flag is not admissible here at all. Three further variables are read **directly and never become configuration keys**: the environment *name*, read into `HostApplicationBuilderSettings.EnvironmentName`; **the credential**, `MUSICSTORE_ADMIN_PASSWORD`, which is [05 §10.2](05-aspnet-core-migration-approach.md) property 1's environment-variable channel, scoped by the pipeline to the single step; and **the audit actor**, `MUSICSTORE_AUDIT_ACTOR`, which is that section's property 4 channel and is **not a switch** — the actor contract below fixes how this host reads it. The credential contract below fixes the credential's acquisition order |
| 3 (highest) | `AddCommandLine` over **the dispatcher's normalized pairs**, never over `args` | Only the non-secret operands the dispatcher below admits: `ProvisionAdmin:Username`, `ProvisionAdmin:Role` and `ProvisionAdmin:ReportDir` when their tokens are given, **plus `ProvisionAdmin:RotateCredential` on every run without exception** — `true` where the switch was typed, `false` where it was not. The verb is **not** a configuration value at all — it selects the code path in the dispatcher and never reaches a provider, and neither does the actor, which arrives on the environment channel of precedence 2's third directly-read variable rather than as a token |
| — | **No JSON file source of any kind** | `appsettings.json` in the working directory is **never read**, because no provider reads it. This is the property the first required assertion below exercises |
| — | **No prefix-scanning environment source, and no `AddEnvironmentVariables` call anywhere** | The whole of the process environment other than the two named configuration variables is invisible to this host's configuration root, and the three variables read directly reach a host setting and two locals rather than a provider. This is the second half of what the first required assertion exercises, and it is the property that makes the mode un-settable by ambient state |

**The release step's own variables, which these processes *see* and do not admit — and the one thing this
section owes each of them.** The table above closes what reaches a configuration root, and it is complete
for that. What it does not say on its own is where the **release step's** variables stand relative to that
closure, and the consequence has been concrete rather than theoretical: two names that a test and a release
step both have to spell were, until this paragraph, spelled only in the documents that *consume* them,
which is the multi-owner defect this grammar exists to close. Three facts hold for every variable in the
table below, and they are what makes it an extension of this surface rather than a fourth source in it.
Each is **read directly from the process environment**, so it is **the configuration key of nothing**: no
provider reads it, it is **not** on the precedence-2 allow-list, and it is **not an admitted token**, so
**no invocation can supply it on a command line at all**. Each is **supplied by the release step**
([06](06-azure-hosting-recommendations.md) §12.6.1 places it on the invocation), never by an argument and
never by a file. And each **name and its value set are
[06](06-azure-hosting-recommendations.md)'s**, published there exactly once — so what is published *here*
is the other half a consumer cannot get from that document: which executable of this grammar reads it, what
it guards, and **the status a refusal returns**, that status being this section's allocation and no other
document's to invent.

| Variable | Which executable of this grammar reads it, and how | What it guards | Absent, or not matching | Where the name and its values are published |
| --- | --- | --- | --- | --- |
| `EXPECTED_ENVIRONMENT` | **Both roots.** `tools/provision-admin` reads it in the environment assertion that runs **after `Build()` returns and before the first scope**, and compares it against the resolved `IHostEnvironment.EnvironmentName` — which for this host is the value `MUSICSTORE_TOOL_Environment` supplied, since that variable is injected as `HostApplicationBuilderSettings.EnvironmentName` above. So the comparison is between **the tool's own governing variable and the deployment the release declared**, which is exactly the mismatch a shared agent produces. The **seed root** reads it in the same assertion; [05 §10.2](05-aspnet-core-migration-approach.md) property 2a states that equality as part of §12.6's **check 1** rather than as a fourth check, so §12.6's count of three is unaffected, and adds the stage that fails when the resolved name **is** `Production` whatever the variable says | That the invocation is acting on the deployment it was released against. It is the one input neither the allow-list nor the dispatcher can supply, because it is a statement *about* the deployment rather than an input *to* the verb | **Fail closed, exit `1`** — the environment-assertion class of the exit-code table below, with the check named in the output alongside the resolved name and the expected one. **Absence is a failure in its own right**: a comparison against nothing is not a check, and defaulting it would recreate the blanket pin [05 §10.2](05-aspnet-core-migration-approach.md) property 2a rejects. Nothing is written on either path | [06](06-azure-hosting-recommendations.md) §12.6.2, one row per pipeline environment. [05 §10.2](05-aspnet-core-migration-approach.md) property 2a owns the in-process assertion's behaviour; §13.3 below states the same comparison for the design-time invocations, against the build's `DESIGN_TIME_ENVIRONMENT` instead |
| The sibling **`EXPECTED_*` target-assertion variables** — the expected server, database and principal names, and the rest of that set | **Of this grammar's executables, `tools/provision-admin` alone, and only the values its own pre-write assertion compares in session** — the database and principal names it reads back with `DB_NAME()` and `USER_NAME()` on the connection it already holds. **Not through configuration**, and **not the control-plane half**: reading a server resource needs a client this project does not reference, so that half belongs to the release step around the invocation ([06](06-azure-hosting-recommendations.md) §6.2.2), and this row does not claim it for the process. The seed root compares none of them, its database gate being §12.6's allow-list | That a privileged write lands in the intended database rather than a neighbouring environment's — an administrator provisioned into the wrong store is a real credential where nobody is looking | **Fail closed, exit `1`**, in the same environment-assertion class and naming the assertion and the two compared values, **before the first write** | [06](06-azure-hosting-recommendations.md) §12.6.2, the same one-row-per-environment set. The assertion's semantics are [05 §10.2](05-aspnet-core-migration-approach.md)'s assertion 3 and [06](06-azure-hosting-recommendations.md) §6.2.2's |
| `MIGRATE_TARGET_CONNECTION`, and `MIGRATE_SOURCE_CONNECTION` with it | **Neither is read by any executable of this grammar, and that is the published fact rather than an omission.** They are the data-migration procedure's channel, and that procedure has **no grammar of its own** (§12.4.1): its instruments take their connection from the release step's environment. Neither name is on the allow-list, so **both are invisible to both roots of this grammar** — which is what makes them usable as the negative canary [05 §12.4.1](05-aspnet-core-migration-approach.md) row **O8** searches the operator tool's whole capture for as raw text | That the data phases act on the intended source and target. Nothing in this grammar guards them, because nothing in this grammar reads them | **Not applicable to this grammar**: no executable here can be absent one or mismatch one. Each instrument's own failure is passed through by the release step, which fails on any non-zero (§12.4.1, §12.9) | [06](06-azure-hosting-recommendations.md) §6.3.2 declares both as invocation-scoped secret channels, and its §12.6.2 states in terms that they are deliberately **not** expected values and that nothing asserts against them |

**`ASPNETCORE_ENVIRONMENT` and `DOTNET_ENVIRONMENT` are deliberately absent from that table**, and their
absence is the same rule seen from the other side rather than an oversight. §13.3 requires both to be set,
to agree, and to equal the invocation's expected value, so both are present in the process environment of
every invocation here. Neither is read by either root of this grammar: both resolve their environment from
`MUSICSTORE_TOOL_Environment` instead, for the reason §12.6 gives — a guard whose most important input came
from the one pair a stale or hostile ambient environment controls would be a guard in name only. What
pinning them buys is that any host which *does* read them — most consequentially the design-time
`dotnet ef` host §13.4 invokes — resolves the same value, so one statement covers every host an invocation
might build.

**The command line cannot be interpreted by a configuration provider at all, so a closed dispatcher
interprets it first.** This is the part of the design a switch map looks like it covers and does not. The
four facts below were **measured** against `Microsoft.Extensions.Configuration.CommandLine` at the `8.0.30`
servicing band §7.1 pins, not inferred from prose, and each one breaks a verb-dispatching tool differently:

| Token form | What `AddCommandLine` does with it | Consequence for a tool that dispatched on configuration |
| --- | --- | --- |
| A bare verb — `admin` | **Dropped silently.** A token carrying no `-`, `--` or `/` prefix and no `=` is skipped, so `["admin", "--username", "op"]` produces exactly one key, `ProvisionAdmin:Username` | The verb **never arrives**. No verb runs on any invocation, however it is spelled |
| A valueless switch followed by another switch — `--rotate-credential --username op` | **The following token becomes its value**: the result is `ProvisionAdmin:RotateCredential` = `--username`, and `ProvisionAdmin:Username` is absent | Silent mis-binding — a truthy-looking rotation *and* a swallowed username, with no error at any layer |
| The same switch in trailing position — `… --rotate-credential` | **Dropped silently**, there being no following token to consume | The rotation disappears: a run meant to replace a credential reports the account already present, changes nothing, and exits `0` |
| Any `--key=value`, whether the map names it or not | Becomes configuration key `key` | `--password=…` becomes configuration, which is why the refusal below cannot be delegated to the map |

The switch map is therefore not the enforcement point and cannot be made into one. **The dispatcher is it**:
it runs **before the configuration root is built**, it maps the admitted tokens to configuration keys
itself, and it is **closed** — every token is admitted by name or the invocation is refused. `AddCommandLine`
then receives only pairs it cannot misread, and no switch map is passed to it, because there is nothing
left for one to translate.

```csharp
internal static class Cli        // internal: this section exposes nothing in this tool, because
{                                // no test project references it. Only Program reads any of it.
    // Two of the five codes 05 §10.2 allocates tool-wide; this file can produce only these two,
    // and the mapping from condition to code is the table below the dispatcher.
    internal const int Ok       = 0;   // the verb ran and its own verification passed
    internal const int Rejected = 2;   // a precondition was rejected and nothing was written

    // The two verbs this dispatcher PARSES, and the one mode switch `admin` admits.
    internal const string Admin       = "admin";
    internal const string Seed        = "seed";
    internal const string RotateToken = "--rotate-credential";

    // The eight verbs it RECOGNIZES and does not parse. Their switch grammar is
    // 05 §5.7's and is deliberately not restated here; naming them is what makes the verb
    // set closed, so an unrecognized verb is refused rather than silently reaching a
    // grammar that does not own it.
    internal static readonly string[] DataVerbs =
    {
        "extract-schema", "diff-schema", "load-catalog", "load-identity",
        "reconcile", "seal-manifest", "accept-run", "close-run",
    };

    // The tokens each PARSED verb accepts, closed per verb — 05 §10.2 property 1b rule 2's
    // accepted sets for these two verbs, restated as the table the pre-parse reads. Membership
    // in the tool's admitted surface (Valued + Switches) and membership in the invoked verb's
    // set are two different questions, and the loop below asks them in that order so the two
    // operator mistakes keep separate codes: CLI-2007 is a token this tool admits nowhere,
    // CLI-2004 a token it admits on the other verb.
    internal static readonly Dictionary<string, string[]> Accepted = new(StringComparer.Ordinal)
    {
        [Admin] = new[] { "--username", "--role", RotateToken, "--report-dir" },
        [Seed]  = new[] { "--report-dir" },
    };

    // The required tokens, PER VERB, because the two verbs do not require the same ones:
    // 05 §10.2 property 1a brackets --report-dir on `admin`, and its section 5.4 puts an ABSENT
    // --report-dir on `seed` in the precondition-rejection class. The actor is NOT in either
    // set: it arrives on MUSICSTORE_AUDIT_ACTOR (Env.AuditActor above), because 05 §10.2 closes
    // the `admin` verb's switch set at four tokens and none of them is an actor.
    internal static readonly Dictionary<string, string[]> Required = new(StringComparer.Ordinal)
    {
        [Admin] = new[] { "--username", "--role" },
        [Seed]  = new[] { "--report-dir" },
    };

    // The valueless switches, table-driven so that the no-value check, the repeat check and the
    // unconditional emission below are inherited rather than rewritten per flag. One entry today.
    internal static readonly Dictionary<string, string> Switches = new(StringComparer.Ordinal)
    {
        [RotateToken] = "ProvisionAdmin:RotateCredential",
    };

    internal static readonly Dictionary<string, string> Valued = new(StringComparer.Ordinal)
    {
        ["--username"]   = "ProvisionAdmin:Username",
        ["--role"]       = "ProvisionAdmin:Role",
        ["--report-dir"] = "ProvisionAdmin:ReportDir",
    };

    internal static string KeyOf(string token)      // for table lookup only; never printed
    {
        var separator = token.IndexOf('=');
        return separator < 0 ? token : token[..separator];
    }

    internal static bool IsPasswordBearing(string key)
    {
        var name = key.TrimStart('-', '/');
        if (name.Equals("p", StringComparison.OrdinalIgnoreCase)) return true;
        var segment = name.Split(':', '_', '-')[^1];
        return segment.Equals("password", StringComparison.OrdinalIgnoreCase)
            || segment.Equals("pwd", StringComparison.OrdinalIgnoreCase)
            || segment.Equals("secret", StringComparison.OrdinalIgnoreCase);
    }

    // Every refusal this dispatcher can produce. The enum is the whole set; there is no
    // "other" member and no free-form reason parameter to smuggle one in through.
    internal enum Refusal
    {
        VerbMissing, VerbUnrecognized, PasswordBearingToken, TokenNotAcceptedByVerb,
        TokenRepeated, SwitchTakesNoValue, TokenUnrecognized, ValueMissing,
        ValueLooksLikeSwitch, ValueEmpty, RequiredTokenMissing,
        ControlCharacter,
    }

    // Code and text, both compiled in. No entry interpolates anything, so no entry can
    // carry a character the caller chose.
    private static readonly Dictionary<Refusal, (string Code, string Text)> Refusals = new()
    {
        [Refusal.VerbMissing]            = ("CLI-2001", "no verb given"),
        [Refusal.VerbUnrecognized]       = ("CLI-2002", "unrecognized verb"),
        [Refusal.PasswordBearingToken]   = ("CLI-2003", "password-bearing token; the credential arrives "
                                                     + "only through its named environment variable"),
        [Refusal.TokenNotAcceptedByVerb] = ("CLI-2004", "token not accepted by this verb"),
        [Refusal.TokenRepeated]          = ("CLI-2005", "token given more than once"),
        [Refusal.SwitchTakesNoValue]     = ("CLI-2006", "mode switch given a value"),
        [Refusal.TokenUnrecognized]      = ("CLI-2007", "unrecognized token"),
        [Refusal.ValueMissing]           = ("CLI-2008", "valued token given no value"),
        [Refusal.ValueLooksLikeSwitch]   = ("CLI-2009", "valued token followed by a switch instead of a "
                                                     + "value; use token=value for a value beginning '-'"),
        [Refusal.ValueEmpty]             = ("CLI-2010", "valued token given an empty value"),
        // CLI-2011 is RETIRED and is not reissued: it was the mutual-exclusivity refusal between
        // --rotate-credential and a --revoke-role switch this tool does not have (the withdrawal
        // record below the admitted-surface table). The codes run 2001-2010, 2012, 2013.
        [Refusal.RequiredTokenMissing]   = ("CLI-2012", "required token missing"),
        [Refusal.ControlCharacter]       = ("CLI-2013", "argument contains a control character"),
    };

    // The whole output of every refusal, on one line. Three components and no fourth:
    // a compiled-in code, a compiled-in sentence, and — where they apply — an int the
    // dispatcher computed and a literal taken from `Required`. NOTHING is read from `args`.
    internal static (string Verb, string[] Pairs, string[] Residual)? Refuse(
        Refusal cause, int position = -1, string? expected = null)
    {
        var (code, text) = Refusals[cause];
        var where  = position >= 0     ? $" at argument {position}" : string.Empty;
        var wanted = expected is null  ? string.Empty : $"; expected {expected}";
        Console.Error.WriteLine($"provision-admin: {code}: {text}{where}{wanted}.");
        Console.Error.WriteLine($"usage: provision-admin {Admin} --username <u> --role <r> "
                              + $"[--report-dir <d>] [{RotateToken}]");
        Console.Error.WriteLine($"       provision-admin {Seed} --report-dir <d>");
        Console.Error.WriteLine($"       provision-admin {string.Join('|', DataVerbs)} [switches]");
        return null;
    }

    internal static (string Verb, string[] Pairs, string[] Residual)? Dispatch(string[] args)
    {
        // FIRST, before any token is interpreted: every raw argument -- the verb included --
        // is scanned for control characters, and one anywhere in one of them refuses the
        // whole invocation. `char.IsControl` IS the test and the class is closed by the
        // framework's own definition rather than by a list this document would have to keep
        // complete: it is true for U+0000-U+001F and U+007F-U+009F, so NUL, tab, CR, LF, the
        // ANSI escape introducer and every C1 code are all in, and no character outside those
        // two ranges is. Only the INDEX is reported; the offending text is never read again.
        for (var i = 0; i < args.Length; i++)
            foreach (var c in args[i])
                if (char.IsControl(c))
                    return Refuse(Refusal.ControlCharacter, i);

        if (args.Length == 0) return Refuse(Refusal.VerbMissing);
        var verb = args[0];

        // A recognized data verb leaves here with its tokens UNTOUCHED. `Pairs` is empty, so
        // nothing this method did not parse can reach AddCommandLine; `Residual` is what the
        // data-verb grammar of 05 §5.7 parses under its own closed table. The two checks above
        // -- the control-character scan and, below, the password-bearing refusal -- are
        // properties of the raw array and apply to every verb, which is why the scan runs first.
        if (Array.IndexOf(DataVerbs, verb) >= 0)
        {
            for (var i = 1; i < args.Length; i++)
                if (IsPasswordBearing(KeyOf(args[i])))
                    return Refuse(Refusal.PasswordBearingToken, i);
            return (verb, Array.Empty<string>(), args[1..]);
        }

        if (verb != Admin && verb != Seed) return Refuse(Refusal.VerbUnrecognized, 0);

        var pairs = new List<string>();
        var seen  = new HashSet<string>(StringComparer.Ordinal);

        for (var i = 1; i < args.Length; i++)
        {
            var token = args[i];
            var key   = KeyOf(token);

            if (IsPasswordBearing(key))
                return Refuse(Refusal.PasswordBearingToken, i);
            if (!Switches.ContainsKey(key) && !Valued.ContainsKey(key))
                return Refuse(Refusal.TokenUnrecognized, i);    // a second verb and a bare operand land here
            if (Array.IndexOf(Accepted[verb], key) < 0)
                return Refuse(Refusal.TokenNotAcceptedByVerb, i);   // admitted, but not on THIS verb
            if (!seen.Add(key))
                return Refuse(Refusal.TokenRepeated, i);

            var attached = token.Length != key.Length;          // the "--key=value" form

            if (Switches.ContainsKey(key))
            {
                if (attached) return Refuse(Refusal.SwitchTakesNoValue, i);
                continue;                                       // recorded in `seen`; emitted once, below
            }
            var mapped = Valued[key];                           // membership proved by the admission
                                                                // check above, the switch branch having
                                                                // taken every `Switches` key already

            string value;
            if (attached) value = token[(key.Length + 1)..];
            else if (i + 1 == args.Length) return Refuse(Refusal.ValueMissing, i);
            else
            {
                value = args[++i];
                if (value.StartsWith('-')) return Refuse(Refusal.ValueLooksLikeSwitch, i);
            }
            if (value.Length == 0) return Refuse(Refusal.ValueEmpty, i);
            pairs.Add($"{mapped}={value}");
        }

        foreach (var required in Required[verb])                 // `required` is a compiled-in literal
            if (!seen.Contains(required))
                return Refuse(Refusal.RequiredTokenMissing, expected: required);

        // The mode flag is stated on EVERY dispatched invocation, at the highest precedence,
        // so the mode is a function of the typed switch alone. Nothing ambient can decide it,
        // and nothing absent can leave it undecided.
        foreach (var (token, key) in Switches)
            pairs.Add($"{key}={(seen.Contains(token) ? "true" : "false")}");

        return (verb, pairs.ToArray(), Array.Empty<string>());
    }
}
```

**It is total, and that is a property rather than a hope.** Every path through the token loop either
`continue`s — having added exactly one pair for a valued token, or nothing at all for a switch, which the
`seen` set has already recorded — or returns a refusal. There is **no wildcard fall-through and no default
admission**, so a token that is not named above cannot reach a configuration provider, and for the two verbs
this method parses the dispatcher's output is by construction a set of `key=value` pairs whose keys are drawn
from the three in `Valued` plus the one in `Switches` — four keys, and no fifth is reachable. **The mode key
is always among them**: the loop records the switch and the post-loop emission states it either way, so a
dispatched `admin` or `seed` invocation produces between two and four pairs and never fewer than the one
mode. That holds for `seed` too, whose single accepted token is also its required one, so it produces exactly
two pairs — the report directory and the mode as `false`; the seed verb does not read the mode, and emitting
it uniformly is cheaper to reason about than a verb-conditional pair set. **The two membership questions are
asked in the stated order and that order is load-bearing**: a token neither table names is `CLI-2007`
whatever verb was invoked, and only a token this tool *does* admit can reach `CLI-2004` — so an operator who
typed a switch that exists on the other verb is told that, rather than being told their switch does not
exist.
**A data verb produces no pair at all**, so the same claim holds for it in the strongest form: `Pairs` is
empty, `AddCommandLine` receives nothing, and every one of its switches is interpreted by the grammar
[05 §5.7](05-aspnet-core-migration-approach.md) owns rather than by a table in this document that would
otherwise have to duplicate it and drift from it. **The switch goes through a table-driven
branch on purpose, even at one entry**: a hand-written `if` per flag is how a second one acquires a subtly
different refusal rule, and the `Switches` table makes adding a mode a one-line entry that inherits the
no-value check, the repeat check and the unconditional emission already proved here — which is why the
table is retained rather than collapsed into a single comparison. The type is declared **after** the
top-level statements in
`tools/provision-admin/Program.cs`, next to the `public partial class Program { }` this section already
requires; C# admits no type declaration before them.

**One branch sits outside that loop, and it is the control-character scan — a refusal class rather than a
sanitizer.** It is stated as its own class, with its own code `CLI-2013`, for a reason the no-echo rule below
does not by itself supply: *not echoing* an argument stops a control character reaching the diagnostic
stream, but it does nothing about the same character reaching **configuration**, and from there a
`ProvisionAdmin:Username` or `ProvisionAdmin:Role` value that an Identity operation, a `PROV-6001` structured
field or a downstream consumer receives. An admitted value carrying `\u001b` or a line feed was previously
refused nowhere: the token was recognized, the value was non-empty, and it became a configuration pair. The
scan closes that, and four properties make it a closed branch rather than a filter:

- **It refuses; it does not clean.** There is no stripping, no escaping and no replacement, so no value is
  silently altered between what an operator typed and what the tool acts on — the failure mode a sanitizer
  has and a refusal cannot.
- **The class is closed by `char.IsControl` rather than by an enumeration this document maintains.** That
  predicate is exactly `U+0000`–`U+001F` and `U+007F`–`U+009F`, so NUL, tab, carriage return, line feed, the
  ANSI escape introducer and the whole C1 range are covered by construction, and a character outside those
  ranges is outside the class by construction. A list of literals would be a list to keep complete.
- **It runs first, over every raw argument including `args[0]`.** So a *verb* carrying a control character is
  `CLI-2013` and not `CLI-2002`, and a `--password=…` argument that also carries one is `CLI-2013` and not
  `CLI-2003` — the order is stated because a reader checking a code against a runbook needs to know which
  branch claims an input that satisfies two. Neither ordering leaks anything, since no branch echoes text;
  the scan is first because it is the only check whose subject is the bytes rather than the grammar.
- **Its output obeys the same fixed form as every other refusal.** `Refuse(Refusal.ControlCharacter, i)`
  emits the compiled-in code, the compiled-in sentence and the **`int` index** — never the argument, never
  the character, and never a count or a position *within* the argument, which would be a one-character
  oracle over text the tool has just declined to handle.

**Where the NUL case is *not* asserted, and why that is the right answer rather than a gap.** The edge
paragraph of §12.4 states the boundary fact: a NUL cannot cross a process boundary, because on Windows
`CreateProcess` takes one NUL-terminated command line and on POSIX `execve` takes NUL-terminated argument
strings, so the byte *is* the terminator and never arrives at `Main`. The consequence for this branch is
symmetric and is the whole of it — **no operator invocation can deliver a NUL, so no run can reach the
refusal.** `char.IsControl` covers `U+0000` because the framework's definition of the class does, and the
scan's coverage of it is therefore a **closed-set property**: it is what makes the class complete without a
maintained list, and it costs nothing to keep. Asserting it would require an in-process call, a fourth
project edge and a public surface on a privileged tool's argument parser, for a behaviour no caller can
trigger; §12.4 declines all three. Every control character argv *can* carry travels through intact and is
asserted the ordinary way, by launching the built tool as a process (assertion 2 below).

**A refusal emits a fixed code and fixed fields, and nothing whatsoever from the argument array.** This is
the one place where a tool that has correctly refused an input can still be harmed by it, so the mechanism
is specified rather than left to whoever writes the message. `Refuse` takes a **member of a closed enum**,
not a `string reason`, and the code and sentence for each member are compiled in. The only variable parts
of the emitted line are an **`int` the dispatcher computed** — the argument position — and, for
`RequiredTokenMissing`, a literal the loop took from the compiled-in `Required` array. No call site
interpolates `args[i]`, `token`, `key`, `value` or `verb`, and there is no `reason` parameter through which
a later edit could reintroduce one.

**What that closes, concretely.** These refusals run **before the host exists**, so they are written
straight to `Console.Error` and are collected — interleaved with the host's own JSON console lines — into
the retained pipeline artifact [06](06-azure-hosting-recommendations.md) §9.5 defines. Interpolating the
offending verb or key into the message is the obvious way to write this dispatcher, and it is the way that
fails: a single argument of the form `--x$'\n'{"EventId":"PROV-6001","Outcome":"Success"}` then produces a
**second line** in that artifact, indistinguishable by shape from a genuine record — a forged provisioning
success in the one artifact an auditor reads, produced by an invocation that was rejected and did nothing.
Carriage returns,
line feeds, ANSI escape sequences and terminal control characters all travel the same route — and such an
argument is *also* refused outright by the `CLI-2013` branch above, the two controls being independent
rather than redundant: that branch stops those characters reaching **configuration**, and this rule stops
**any** argument text, control characters or not, reaching the diagnostic stream. Truncating or
escaping the echoed text would narrow the hole; **not echoing it closes the class**, and it costs the
operator nothing they cannot see for themselves, because the argument they typed is in front of them and
the position names which one it was.

**The tokens the dispatcher *admits* are a different question, and it is answered elsewhere rather than
left open.** An admitted value — `--username`, `--role`, `--report-dir`, the three the table below marks
`Valued` — becomes a configuration value and is validated
after admission and before use, each against the policy named in the admitted-surface table below. **The
one value that reaches the audit record does not travel this path at all**: the actor arrives on
`MUSICSTORE_AUDIT_ACTOR`, so it is neither a token nor a configuration key, and the
presence-and-shape check [05 §10.2](05-aspnet-core-migration-approach.md) property 4 states in full is the
one this host performs on it — before the builder is constructed, per the actor contract below. The record
that carries it is written after the host exists, through the JSON console
provider below, which emits it as a **structured field** whose value is escaped rather than as text spliced
into a line. So the pre-host path carries no input at all, and the post-host path carries validated input
as data. Neither hands an argument to a text formatter.

**This is [09 §6.8.1](09-security-assessment.md)'s rule applied one layer earlier, deliberately and not by
coincidence.** That section closes the field set of every audit record and requires a failure outcome to
carry a **category code and nothing more**, with provider text and exception messages confined to the
diagnostic stream. A pre-host refusal attempted nothing, so where the refused invocation is `admin` its
**audit** record set is the not-attempted one [05 §10.2](05-aspnet-core-migration-approach.md) property 4
fixes — four records, on this path as on every other — and the fixed diagnostic below is what *this
section* adds to it. Applying the same rule to
it means the whole **diagnostic** output of a refused invocation is a category
code, a fixed sentence and an integer. The `CLI-2001`–`CLI-2013` codes are **diagnostic codes and not
members of [09 §6.8.1](09-security-assessment.md)'s sixteen event classes**: they are deliberately given
their own `CLI-` family so that no reader and no log query mistakes a refused invocation for an audited
operation.

**Exit codes — there is one allocation, it is tool-wide, shared by both entry points of this grammar (§12.4.1), and this section does not own it.**
[05 §10.2](05-aspnet-core-migration-approach.md) allocates **five codes across all ten verbs** and states
what each means; that table is not restated here, because a second copy of it is how the two come to
disagree. What this section owes is the other direction — **which of those codes each condition it
specifies produces** — since a condition with no code
cannot be gated by a release step:

| Condition specified in this section | Class it falls in | Code |
| --- | --- | --- |
| The verb ran and its own verification passed | Completed | `0` |
| Every case `Cli.Dispatch` returns `null` for — no verb, an unrecognized verb, a password-bearing token, a repeated or unrecognized token, a token the invoked verb does not accept, a missing or malformed value, a missing required token, a control character | **A precondition was rejected and nothing was written.** No configuration root is built, no connection is opened, and **no store write occurs** — no operation was attempted, so every operation is reported as one that was not. **"Nothing was written" is a statement about the store and never about the record set.** On a refused **`admin`** invocation the records it still owes are [05 §10.2](05-aspnet-core-migration-approach.md) property 4's, exactly as the actor row below owes them: that property fixes the cardinality at **four per invocation invariantly, this pre-host path included**, along with the not-attempted outcome each carries and the rejected run outcome, and none of it is restated here. A refused **`seed`** invocation owes no `PROV-6001` record on this path or any other, that class being a privilege-grant record the seed verb never produces (§12.6) | `2` |
| The audit actor absent, empty or whitespace-only on `MUSICSTORE_AUDIT_ACTOR` (the actor contract below) | Same class, and on the same side of the boundary as the dispatcher's refusals: the check runs **before the builder is constructed**, so no configuration root exists, no connection is opened and **no store write occurs**. What the rejection still owes its audit record is [05 §10.2](05-aspnet-core-migration-approach.md) property 4's and not this section's | `2` |
| Any of §12.6's three guard checks failing, and the two connection-string parse failures of that section | Same class: the check runs **before any write**, so nothing was written | `2` |
| The credential absent with stdin redirected (the acquisition order below), and [05 §10.2](05-aspnet-core-migration-approach.md) property 3's published-credential refusal | Same class: refused before the first store write | `2` |
| The target's catalog schema not present when a verb requires it (§12.6) | Could not connect, or the schema is not in the expected state | `3` |

Two things follow, and both are properties of this composition rather than of any one verb. **No condition
in this section produces `1`**, because the allocation has no `1`: a refusal that wrote nothing is a
rejected precondition, and there is no separate "ran and failed" code to fall into. And the two remaining
codes, `4` and `5`, are reachable only from the eight data verbs — partial work rolled back to a table
boundary, and a failed verification — which this document recognizes and does not parse, so they appear
here only as the two codes this section can never produce.

**The admitted surface, whole. There is no other token.**

| Token | Verb | Produces | Required |
| --- | --- | --- | --- |
| `admin` \| `seed` | — | Nothing. The verb selects the code path and **never becomes configuration** | Exactly one, as `args[0]` |
| `extract-schema` \| `diff-schema` \| `load-catalog` \| `load-identity` \| `reconcile` \| `seal-manifest` \| `accept-run` \| `close-run` | — | Nothing here. The verb is **recognized** and its remaining tokens are returned unparsed for the grammar [05 §5.7](05-aspnet-core-migration-approach.md) owns; this table is the admitted surface of the two verbs this section parses and **not** of those eight | Exactly one, as `args[0]` |
| `--username <value>` | `admin` | `ProvisionAdmin:Username` | **Yes**, and the value is validated against **[05 §6.1](05-aspnet-core-migration-approach.md)'s user-name policy** — the *application's* policy, which that deliverable owns and which is the only one in force. It is stated there and not restated here, and the point of citing it from a token table is the failure it prevents: validating the framework's broader default instead would let this command create an account the application itself refuses to accept, so the tool's admitted set has to be the application's set. Validation happens **after** the dispatcher admits the token and before the value is used |
| `--role <value>` | `admin` | `ProvisionAdmin:Role` | **Yes** — [05 §10.2](05-aspnet-core-migration-approach.md) property 1 makes the role name a required, validated input with no default, and its property 3's membership operation adds *that named* role rather than inferring one |
| `--rotate-credential` | `admin` | `ProvisionAdmin:RotateCredential` = `true`, **normalized by the dispatcher** — the token itself is never forwarded | No, and it is the **only** mode switch this verb has. Absent, **the dispatcher states `false` for it explicitly**, at a precedence no environment variable can reach, and the credential of an account that already has one is left untouched, which is [05 §10.2](05-aspnet-core-migration-approach.md) property 3's `AlreadyPresent_NotRotated`; present, that property's rotation path runs |
| `--report-dir <value>` | `admin` \| `seed` | `ProvisionAdmin:ReportDir` — the same `ProvisionAdmin:` section the other two valued tokens normalize into, chosen for a reason of its own rather than by analogy: precedence 2's allow-list holds exactly two variables and this key is not among them, and precedence 1 states no default for it, so **the destination of a report carrying personal data is decided by the typed invocation and by nothing else** — which is what [05 §5.7](05-aspnet-core-migration-approach.md)'s "there is no default location" requires of a host, since a directory an ambient variable or a stray file could supply is a directory nobody chose | **Required on `seed`, optional on `admin`.** [05 §10.2](05-aspnet-core-migration-approach.md) property 1b closes `seed`'s accepted set at this one token and [05 §5.4](05-aspnet-core-migration-approach.md) puts an **absent** one in the precondition-rejection class, so it is that verb's one required token and the refusal is `CLI-2012`; property 1a brackets it on `admin`, where an invocation omitting it writes no report and its audit records remain the run's evidence. Validated **after** the dispatcher admits it and **before the first write**, in three parts: non-empty and control-character-free are the dispatcher's own checks above; the path must be **rooted**, because a relative one resolves against whatever working directory the agent chose — the failure [05 §10.2](05-aspnet-core-migration-approach.md) property 2b sets this host's content root explicitly to avoid; and the directory must **already exist and be writable by the invoking principal**, the host creating neither it nor a parent, because [05 §5.7](05-aspnet-core-migration-approach.md) gives the caller ownership of its access restriction and a directory this host created would carry permissions the caller did not choose. A failure of either post-admission check is exit `2` with nothing written, the path emitted as a **structured field** through the JSON console provider below rather than interpolated into a message, and **no directory invented in its place** |

**The fourth token was missing from this table and is admitted, and everything it moves is counted here
so that no other paragraph in this document states a figure of its own.** An earlier version of this
section admitted three `admin` tokens and no `seed` token, and described the seed verb as taking "no
argument at all". That was not a smaller surface than the contract; it was a **different** one, and it
refused invocations two other documents require: [05 §10.2](05-aspnet-core-migration-approach.md) property
1b closes the `admin` verb's accepted set at **four** switches — `--username`, `--role`,
`--rotate-credential`, `--report-dir` — and the `seed` verb's at **one**, `--report-dir`; row 75 of
[05 §12.4](05-aspnet-core-migration-approach.md) invokes `admin … --report-dir <run artifacts>` and asserts
exit `0`; and [05 §5.4](05-aspnet-core-migration-approach.md)'s seed form carries the same switch. Under the
closed pre-parse specified above, every one of those would have been refused before a host was built. The
table now admits exactly the contract's set, and the arithmetic that follows from it:

| What is counted | Value |
| --- | --- |
| Rows in the admitted-surface table above | **Six** — two verb rows and four token rows |
| `Valued` entries | **Three** — `--username`, `--role`, `--report-dir` |
| `Switches` entries | **One** — `--rotate-credential`, still the `admin` verb's only mode switch |
| Configuration keys the dispatcher can produce | **Four** — `ProvisionAdmin:Username`, `ProvisionAdmin:Role`, `ProvisionAdmin:ReportDir`, `ProvisionAdmin:RotateCredential` — and no fifth is reachable |
| Pairs a dispatched invocation carries | **Two to four**: `admin` three or four, `seed` exactly two, a data verb none |
| Required tokens | **Two** on `admin` and **one** on `seed` — three entries across the two parsed verbs |
| `CLI-2012` cases in the refusal sweep below | **Three** — each of `admin`'s two absent in turn, and `seed`'s absent |
| Refusal classes and their codes | **Twelve**, `CLI-2001`–`CLI-2010`, `CLI-2012` and `CLI-2013` — **unchanged**: `CLI-2004` is *repurposed* from "the seed verb takes no argument" to the per-verb accepted-set refusal, which is the same rule with a verb that now accepts one token, so no code is issued and none beyond `CLI-2011` is retired |
| Canonical forms that dispatch | **Five** — four `admin` combinations and one `seed`, enumerated below |
| Fail-closed default entries | **Two** — unchanged. A report directory has no safe default to state, which is [05 §5.7](05-aspnet-core-migration-approach.md)'s point rather than an omission here |

**One switch and one key were specified here and are withdrawn, and the withdrawal is recorded rather
than silently applied — because a reader holding the earlier form has to be able to recognize it as
superseded.** An earlier version of this section admitted a further `admin` token, **`--revoke-role`**,
produced a **`Provisioning:RevokeRole`** key from it, gave that key a fail-closed default entry, refused
it against `--rotate-credential` under the retired code `CLI-2011`, and described the command throughout
as having two modes. **None of that is in the tool, and none of it could be.**
[05 §10.2](05-aspnet-core-migration-approach.md) closes the `admin` verb's accepted switch set at four
tokens, none of which asks for a revocation, and its property 1b rejects any token outside that set in a
**closed pre-parse that runs before a host is built** — so an invocation carrying `--revoke-role` is
refused rather than dispatched, and a switch this section normalized into a configuration key would have
been a key no run could ever set. Its property 3 states in as many words that the command **never deletes
or demotes** anything, and its verb table is closed at ten verbs with no revoke verb among them. So the
token, the key, its default entry, its refusal class and every "in both modes" phrasing are **removed
here rather than reconciled**, and `CLI-2011` is retired with them and not reissued: the codes this
section produces are `CLI-2001`–`CLI-2010`, `CLI-2012` and `CLI-2013`, twelve classes with no `CLI-2011`
among them.

**What the withdrawal leaves open is real, and it is not this document's to carry.** Nothing in this plan
withdraws a role, and [09 §6.8.1.1](09-security-assessment.md) records that gap where a reader of the
audit catalog meets it and owns it as an open finding — including its severity, its owner and the
reserved event identifier a future capability would emit. This section states no severity of its own and
adds no capability to close it, because a revoke capability specified from here would reopen two contracts
that are pinned elsewhere: the closed switch set above, and the per-invocation audit-record cardinality
[05 §10.2](05-aspnet-core-migration-approach.md) property 4 fixes.

**A second switch is withdrawn and two spellings are corrected, recorded the same way and for the same
reason.** An earlier version of this section admitted a third `admin` token, **`--actor`**, and normalized
it to a **`Provisioning:Actor`** configuration key; it spelled the user-name token **`--user`**; and it
named its keys under a **`Provisioning:`** section. **None of the three is the contract**, and the first of
them is not a misspelling but a switch the owning parser refuses.

- **`--actor` is withdrawn entirely rather than respelled.**
  [05 §10.2](05-aspnet-core-migration-approach.md) property 4 supplies the actor through
  **`MUSICSTORE_AUDIT_ACTOR`, an invocation-scoped environment variable rendered by the pipeline from its
  own authenticated trigger**, and states in as many words that it **is not a switch**; its property 1b,
  which closes the `admin` verb's accepted set at four tokens, therefore **rejects an invocation that passed
  the actor on the command line**. So this is the same shape as the `--revoke-role` withdrawal above rather
  than a different one: a token the owning pre-parse refuses cannot be a token this dispatcher admits, and a
  key normalized from it would have been a key no run could ever set. **Why the owner chose a variable is
  what this section's own parse-case rows teach**: a value on a command line is visible in process listings
  to any user on the host and is recorded by shell history and by pipeline logs — property 1's argument
  about the credential, applied to a field an auditor reads — and an actor an operator types is
  self-asserted, while one the pipeline renders is corroborated by a trigger the command cannot write. The
  token, the key and the `Required` entry are removed here; what replaces them is the actor contract below,
  which fixes how this host reads the variable and what it does when it is absent.
- **`--user` becomes `--username`, and the `Provisioning:` section becomes `ProvisionAdmin:`.** Both are
  the owning document's spellings — property 1's input table names `--username` mapped to
  `ProvisionAdmin:Username`, and `--role` mapped to `ProvisionAdmin:Role` — and this section restates them
  rather than owning them, so the earlier spellings are corrected wherever they stood: the dispatcher's
  `Valued` and `Required` tables, the fail-closed default entry, the **four** affected source-precedence
  rows, the **two** parse-case rows that spelled a token, the usage line, the canonical invocation forms,
  the ambient-prefix hazard example, the validation prose, §12.6's invocation row and required assertions 1
  and 5. The historical key name in the `--revoke-role` record above keeps its original `Provisioning:`
  spelling deliberately, because it records what was written and not what would have been written.
- **What the withdrawal changes about the arithmetic, stated as its own delta because the claims above
  depend on it.** It removes one entry from `Valued`, one required token from the `admin` verb and one row
  from the admitted-surface table, so the `admin` verb has **two** required tokens rather than three and
  the dispatcher produces one key fewer than it would have. **The current totals are the recount above's
  and are not restated here**, because the same figures also moved when `--report-dir` was admitted and
  two paragraphs stating them independently is how they come to disagree. The fail-closed default still has **two** entries, because there
  was never an actor default to remove: an absent actor is a refusal rather than a defaulted value. And the
  refusal classes are **unchanged at twelve** — `CLI-2012` covers `admin`'s two required tokens instead of
  three, no new class is added, and no code is retired by this withdrawal, so `CLI-2011` remains the only
  retired one.

**Password-bearing input is refused by a rule of its own, before anything is built.** A closed allow-list
already excludes it, and the check is kept, kept **first**, and given its own code because the diagnostic
should say *which* rule refused the run: `CLI-2003` distinguishes "you passed a secret on the command line"
from `CLI-2007`'s "that token is not one of mine", and the two want different corrective action from an
operator. Refused forms are `--password`, `-p`, `--pwd`, `--admin-password`, `Admin:Password`,
`Admin__Password`, and any key whose final `:`, `_` or `-` segment equals `password`, `pwd` or `secret`
under an ordinal case-insensitive comparison. That is
[05 §10.2](05-aspnet-core-migration-approach.md) property 1's "refuses to read a password from an argument
at all", made mechanical rather than left as an intention.

**The message names neither the key nor the value — `KeyOf` exists for lookup, not for output.** It is
worth being explicit, because splitting `--password=<the real password>` at its `=` looks like the
mitigation and is not one. Two things go wrong if the key segment is echoed. A **secret can be a key**:
`--<the real password>` with no `=` at all makes `KeyOf` return the whole token, so the very mechanism that
rejected the credential prints it — and this is a plausible slip, not a contrived one, given that an
operator confusing a positional argument for a switch is exactly who trips `CLI-2003`. And a key is
**attacker-chosen text on an output path**, which is the injection the paragraph above closes. So the code
carries the classification, the position carries which argument it was, and neither half of the token is
printed by any path.

**The forms that dispatch — and there is one.** Every invocation shown anywhere in this document is the
canonical literal [05 §10.2](05-aspnet-core-migration-approach.md) defines, `provision-admin <verb>
[switches]`, where `provision-admin` **is** the published operator tool — that is, `dotnet` against the one
published assembly. The two verbs this section parses take it in these shapes:

```text
provision-admin admin --username <username> --role <role>
provision-admin admin --username <username> --role <role> --report-dir <dir>
provision-admin admin --username <username> --role <role> --rotate-credential
provision-admin admin --username <username> --role <role> --report-dir <dir> --rotate-credential
provision-admin seed --report-dir <dir>
```

**Five forms, and the enumeration is exhaustive rather than illustrative**: the `admin` verb has two
optional tokens and therefore four combinations, and the `seed` verb has one form because its single
accepted token is required. Row 75 of [05 §12.4](05-aspnet-core-migration-approach.md) invokes the second,
and [05 §5.5.1](05-aspnet-core-migration-approach.md)'s credential repair adds the flag to it, which is the
fourth; a release stage that wants no report file runs the first, and gets none. No other spelling appears
in this document, and none should appear in a runbook: a developer-session form
that differs from the pipeline's is how a switch gets proved in one and not the other. The invocations are
what [06](06-azure-hosting-recommendations.md) §12.1's provisioning stage and §9.5's sanctioned path run,
and [05 §10.2](05-aspnet-core-migration-approach.md)'s operations are what each one performs. The eight data
verbs take the same literal with their own switches, which
[05 §5.7](05-aspnet-core-migration-approach.md) states.

**The credential contract — how this host reads the name its owner fixes.**
[05 §10.2](05-aspnet-core-migration-approach.md) property 1 establishes the two channels, selects the
environment variable for the release process and **names it `MUSICSTORE_ADMIN_PASSWORD`**. That name is
that document's; what it does not state, and what a release stage cannot infer, is how a host with no
environment provider reads it and in what order — which is what this section fixes:

| Property | Value |
| --- | --- |
| **Environment variable** | **`MUSICSTORE_ADMIN_PASSWORD`** — [05 §10.2](05-aspnet-core-migration-approach.md)'s name, restated here only because this host has to read it, and matched by exact ordinal comparison |
| **Configuration key** | **None, deliberately.** The credential is not a key of [05 §3.3](05-aspnet-core-migration-approach.md)'s typed contract and is not admitted by precedence 2, so it exists only as a local string between the read and the Identity call. There is no key for a dump, a diagnostic or an enumeration over the configuration root to surface |
| **Which runs read it** | The **`admin`** verb only — with or without `--rotate-credential`, since the credential is what an absent account is created with and the flag decides only whether an existing one is replaced ([05 §10.2](05-aspnet-core-migration-approach.md) property 3). The **`seed`** verb never reads it, and neither does any of the eight data verbs; a run of any of those nine is unaffected by the variable's presence or absence |
| **When it is read** | **After** the dispatcher has admitted the invocation and the host is built, and **before the first store write**, so an absent credential cannot leave a half-provisioned account |

**The name is matched exactly, so it is the name and not a pattern that has to be right.** `Env.AdministratorPassword()`
compares `MUSICSTORE_ADMIN_PASSWORD` ordinally against the process environment and returns the value or
nothing, with **no prefix stripping, no delimiter translation and no provider involved at all**. A
misspelling therefore matches nothing and the credential is silently absent — which the acquisition order
below turns into an explicit refusal rather than a partial run, and which required assertion 6 exercises
against the real variable.

**Why the credential is the one input read outside configuration, while the other two named variables are
read into it.** `ConnectionStrings__DefaultConnection` and `Seeding__Enabled` are the double-underscore
spellings of keys the application itself already has, so admitting them as configuration is what makes the
tool and the application name the same thing the same way; `__` is .NET's conventional environment-variable
hierarchy delimiter and `:` is avoided because it is not portable in a variable name on the Linux agents
[06](06-azure-hosting-recommendations.md) §3.2 selects. The credential has no such key, and giving it one
would create exactly the surface the rest of this section removes: a value reachable by an enumeration over
the configuration root, and a second name for a secret whose only name is
[05 §10.2](05-aspnet-core-migration-approach.md)'s. **The mapping moved from a provider convention to an
explicit two-entry table, and the credential moved out of configuration entirely.**

**Acquisition order — three steps, and the third is a refusal rather than a fallback.**

1. **The environment variable, if present and non-empty.** It is used as read, with no prompt offered even
   when a terminal is attached, so a pipeline run's behaviour never depends on how its agent allocates
   consoles.
2. **Otherwise, an interactive prompt with terminal echo disabled** — but **only** where `Console.IsInputRedirected`
   is `false`. The prompt reads keys with `Console.ReadKey(intercept: true)`, echoing nothing, and the same
   condition guards it that the API requires: `ReadKey` throws when stdin is redirected, which is exactly
   the case a pipeline job presents.
3. **Otherwise — absent and non-interactive — exit `2` without prompting and without partial effect.** The
   diagnostic **names the variable and never a value**, and **no store write has occurred**: no operation
   was attempted, so each is reported as one that was not, under the same four-record cardinality
   [05 §10.2](05-aspnet-core-migration-approach.md) property 4 fixes for every path. A prompt written to a redirected stream
   would be a job that hangs until its timeout with no indication why, which is the failure this refusal
   replaces.

**Where the value may never appear — five channels, each closed by a named mechanism.** It is never a
command-line argument (the dispatcher's password-bearing refusal above, and no admitted token produces this
key); never printed, including by the prompt, which echoes nothing; never written to a log or an audit
record ([09 §6.8.1](09-security-assessment.md)'s field set is closed at actor, timestamp, target username,
role and outcome, and the credential is not among them); never persisted, there being no file source in
this host to read it from or write it to (required assertion 1); and never interpolated into a diagnostic or
an exception message, so a failed `ResetPasswordAsync` reports Identity's own error codes rather than the
value it was given. Required assertion 6 is what demonstrates the set rather than asserting it.

*Which runs supply it is deliberately not settled here.*
[05 §10.2](05-aspnet-core-migration-approach.md) property 3 owns what the provisioning operation does with
a credential and [06](06-azure-hosting-recommendations.md) §12.1 owns the stage that invokes the command;
this section fixes only the name, the delimiter and the acquisition order — which is the part a release
stage cannot infer and neither of those documents states.

**The actor contract — the same channel shape, a different reason for it, and one thing this host owes.**
[05 §10.2](05-aspnet-core-migration-approach.md) property 4 owns the audit actor outright: the channel and
its name, the rule that it **is not a switch**, the presence-and-shape check, the reserved value the record
carries when the actor itself is what is missing, and the record a rejected invocation still owes. What that
leaves to this section is the gap the credential left in the same place — **how a host with no environment
provider reads the variable, and where in the sequence the check sits**:

| Property | Value |
| --- | --- |
| **Environment variable** | **`MUSICSTORE_AUDIT_ACTOR`** — [05 §10.2](05-aspnet-core-migration-approach.md) property 4's name, restated here only because this host has to read it, matched by exact ordinal comparison with no prefix stripping and no delimiter translation, exactly as the credential is |
| **Configuration key** | **None, deliberately**, and for a reason of its own rather than the credential's. It is not a key of [05 §3.3](05-aspnet-core-migration-approach.md)'s typed contract and precedence 2 does not admit it, so it exists only as a typed field on the parsed invocation. A key would be settable from a file or from a differently named variable, and an actor an ambient value can supply is not an attribution at all |
| **Which runs read it** | The **`admin`** verb's, because property 4 is that verb's under [05 §10.2](05-aspnet-core-migration-approach.md)'s own division of labour. This section states no actor requirement for `seed` or for the eight data verbs, whose records are [05 §5.6](05-aspnet-core-migration-approach.md)'s; the variable is nonetheless admitted for the whole tool by [05 §3.3](05-aspnet-core-migration-approach.md)'s operator-console row, so a verb whose own records need it reads the same name |
| **When it is read** | **After the dispatcher returns and before the builder is constructed** — the earliest point at which the invocation is known to be one the tool would act on, and still on the side of the boundary where nothing has been read and nothing can have been written |
| **What an absent value does** | Exit `2`, in the same precondition-rejection class as every dispatcher refusal (the exit-code table above): no configuration root is built, no connection is opened and **no store write occurs**. Absent, empty and whitespace-only are **one** case, per property 4, which also owns the reserved value the resulting record carries |

**One consequence is worth stating, because neither document states it alone.** This is the only pre-host
rejection in this section whose subject is not `args`, so it does **not** live in `Cli.Dispatch`: that
method's subject is the token array, and its `CLI-`-prefixed codes are argument diagnostics. The actor check
sits beside the dispatch guard in `Program.cs`, reaches the same exit code by the same rule, and adds **no
refusal class and no code** — which is why the twelve `CLI-` classes are unchanged by it. The audit record
the rejection still owes is property 4's, and its four-record constant already covers a rejected
invocation; what this host contributes is the check, its position and the exit.

**And two consequences for the coverage below, so neither is left to be inferred.** The rejection itself is
already covered by somebody: [05 §12.4](05-aspnet-core-migration-approach.md)'s row **O7** exercises a
rejection *before any host is built* with a missing actor by name, so **this section adds no case for it**
and the six-assertion, five-test figures below are unaffected. And in the other direction, every
`admin` invocation those assertions drive — and the release stage
[06](06-azure-hosting-recommendations.md) §12.1 drives — **exports the variable**, because a run that does
not is the rejection rather than the behaviour under test.

**Exactly one logging provider, and it is structured.** `builder.Logging.ClearProviders()` then
`builder.Logging.AddJsonConsole()`. Three reasons, in order of weight: the audit record's field set is
**closed** ([09 §6.8.1](09-security-assessment.md)), and a structured provider emits those fields as
log state — one JSON object per line — rather than as text somebody has to parse back out; the record is
collected from the process's stdout and stderr as a retained pipeline artifact
([06](06-azure-hosting-recommendations.md) §9.5), so exactly one provider means exactly one copy and no
second sink to reason about; and `ClearProviders()` is a **no-op today** — the empty builder registers none
— retained so that the one-provider guarantee survives any future change to what the builder starts with.

**Registrations — one seam call and two additions, and each is load-bearing.**

```csharp
// The application's OWN registration seam (12.9): both contexts on the SQL Server provider at
// the configured connection string, Identity core, roles, the EF stores over ApplicationUser,
// the default token providers, the password hasher and the Identity options. One registration
// path, so the hash this tool writes is a hash the application accepts.
builder.Services.AddMvcMusicStoreServices(builder.Configuration);

// The two a console host must add for itself, because the WEB host takes both from defaults
// this builder does not have.
builder.Services.AddDataProtection().UseEphemeralDataProtectionProvider();
builder.ConfigureContainer(new DefaultServiceProviderFactory(
    new ServiceProviderOptions { ValidateScopes = true, ValidateOnBuild = true }));
```

- **`AddDefaultTokenProviders()` is not optional, and its absence is a run-time throw on the recovery
  path — which is why the seam calls it rather than leaving each caller to decide.** The existing-account
  repair of [05 §10.2](05-aspnet-core-migration-approach.md) property 3 calls
  `GeneratePasswordResetTokenAsync`, which looks up the provider named by
  `IdentityOptions.Tokens.PasswordResetTokenProvider` — `"Default"` — in a registry that
  `AddIdentityCore` leaves **empty**. With nothing registered the call throws `NotSupportedException`, it
  compiles perfectly on the way there, and the throw lands at the exact moment an operator is repairing a
  neutralized administrator account. `AddDefaultTokenProviders()` registers
  `DataProtectorTokenProvider<ApplicationUser>` under that name, which is what makes the call supported.
- **Data protection is the dependency underneath the token provider, and a console host has none.**
  `DataProtectorTokenProvider` protects the token through `IDataProtectionProvider`, which the web host
  registers as part of its own defaults and which nothing in a Generic Host does. `AddDataProtection()`
  supplies it. **The key location is deliberately ephemeral — in memory, for the life of the process.**
  The token is generated and redeemed **inside one scope, in one process, and is never displayed,
  returned or transported**, so no key needs to outlive the run; a persisted ring would instead write key
  material into `$HOME` or `%LOCALAPPDATA%` on a shared pipeline runner, unprotected at rest because this
  process configures no DPAPI or certificate protector, and would leave a durable artifact behind for a
  tool whose whole design is to leave none. The consequence is stated rather than discovered: **a token
  this host issues is meaningless to any other process or any later run**, which is acceptable precisely
  because nothing outside the process ever sees it — and it is why the token must never be logged,
  printed or persisted.
- **Scoped services are resolved inside an explicit scope, and the container is configured to prove it.**
  `UserManager<ApplicationUser>`, `RoleManager<IdentityRole>` and both contexts are **scoped**, and a
  console process has no ambient request scope. Resolving them from `host.Services` — the root provider —
  either throws or, with validation off, **succeeds and promotes the `DbContext` to the root container's
  lifetime**, where it is disposed only at process exit and its change tracker retains everything the run
  touched. So:

  ```csharp
  using var host        = builder.Build();                   // IHost: IDisposable only
  await using var scope = host.Services.CreateAsyncScope();   // AsyncServiceScope: IAsyncDisposable
  var users = scope.ServiceProvider.GetRequiredService<UserManager<ApplicationUser>>();
  var roles = scope.ServiceProvider.GetRequiredService<RoleManager<IdentityRole>>();
  ```

  **The two `using` forms differ deliberately, and having them agree is a compile error rather than a
  style choice.** `HostApplicationBuilder.Build()` returns **`IHost`**, so `IHost` is what the `var` local
  is inferred as — and `IHost` implements **`IDisposable` only**; `IAsyncDisposable` is not assignable from
  it. `await using var host = builder.Build();` therefore does not compile: the compiler reports
  *"'IHost': type used in an asynchronous using statement must be implicitly convertible to
  'System.IAsyncDisposable' or implement a suitable 'DisposeAsync' method"* (**CS8417**). What decides this
  is **the interface the local is typed as, not the concrete host type behind it**, so the form cannot be
  rescued by a different host implementation, and the synchronous `using` is the only correct spelling of
  that line. `AsyncServiceScope` is a different case and the second line is right as written: it genuinely
  implements `IAsyncDisposable`, so its `await using` is both legal and required. Both facts are the
  `8.0.30` reference packs' (§7.1), and the required assertion 4 below is what keeps the distinction from
  being reintroduced.

  **`CreateAsyncScope` rather than `CreateScope` — a preference with a real reason, and not because the
  synchronous form fails.** It does not: `CreateScope` returns an `IServiceScope`, synchronous `Dispose`
  is a supported way to end a scope, and **`DbContext` implements both `IDisposable` and
  `IAsyncDisposable`**, so the container has a synchronous path to take for it. The container's one hard
  restriction is narrower than it is often remembered as being — it throws on synchronous disposal of a
  service that implements `IAsyncDisposable` **and not** `IDisposable`, which is a property of whatever
  the scope resolves rather than of the scope. That is the first reason to prefer the asynchronous form
  and to state it as a standing rule for this tool: it is correct for any registration this command
  later gains, and it does not require anyone to re-audit the composition for an only-asynchronous
  disposable before adding one. The second is that the asynchronous teardown
  `Microsoft.Data.SqlClient` can perform stays asynchronous rather than falling back to the synchronous
  close. The third is that the tool's whole body is already `async` — every Identity call in it is
  awaited — so `await using` costs nothing to write.

  It is named as a **preference** and not a defect claim because the stronger claim is false, and the
  precision matters in both directions. Synchronous disposal of what this command resolves today would
  not hit the restriction above — the context, the user manager and the role manager each implement
  `IDisposable` — and a console host carries no synchronization context for a synchronous wait to
  deadlock against. What it would do is run each disposable's cleanup on the calling thread, and how long
  any given implementation takes to do that is a property of that implementation rather than something
  this document can promise either way. Asserting a deterministic blocked-thread failure would send a
  reader hunting for a hazard that is not there, and a document that manufactures one hazard is trusted
  less on the ones that are real.

  **`ValidateScopes` and `ValidateOnBuild` are set on**, because the defaults-disabled builder enables
  neither, and with `ValidateScopes` off the root-resolution mistake is not an exception but a silent
  lifetime promotion — the failure mode that is invisible in a short run and wrong in every long one. The
  command does **not** call `RunAsync`: it registers no hosted service, and the host exists only as a
  container and a configuration root. It sets the process exit code from the outcome and disposes.
- **The seam registers Identity *core*, and `AddIdentity` stays outside it.** The web application adds the
  cookie authentication schemes and `SignInManager` in `Program.cs`
  ([05 §2.4](05-aspnet-core-migration-approach.md) stage 1), which is §12.9's deliberately excluded half:
  they are an HTTP-shaped composition with no meaning in a console process, and a tool that acquired them
  would fail on a misconfiguration in a subsystem it never exercises. The two compositions differ because
  the two hosts differ, and neither is a copy of the other.
- The **only** things it takes from the web project are **two**: the `ApplicationUser` type it names as
  `UserManager<>`'s type argument, and the `ApplicationServices.AddMvcMusicStoreServices` method above. It
  names `MusicStoreEntities` and `Data/SeedData` **not at all** — seeding is the web application's own
  command (§12.6) and this tool has exactly one capability. That pair is the whole reason for the tool's
  own `ProjectReference` — `tools/provision-admin` → the web application, the third of §12.4's three edges
  and the only one this project declares. Each
  must be the web project's and not a local re-declaration: a re-declared context maps to a different
  schema than the migrations produce, and a re-declared registration hashes a password under options the
  application does not have.

**Six assertions are required of this composition.** Five of them cover failures that are invisible at
compile time, and two of those stay invisible until a privileged operation is already under way; the
remaining one *is* a compile check, because the one thing in this composition a reader can get wrong in a
way the runtime never gets a chance to report is the lifetime spelling above. They
are demonstrated by tests in `src/MvcMusicStore.Tests` — the project §12.2 already creates — driving the
**real entry point** in a real process-shaped configuration, which is the only way a host's own behaviour
is exercised at all. **Every one of them is driven through a process**, and §12.4's edge paragraph states why
that costs no coverage: the one refusal a process boundary cannot deliver is the NUL, which no operator
invocation can deliver either, so there is nothing an in-process call would prove about a reachable run.
These assertions extend the command coverage [05 §12.4](05-aspnet-core-migration-approach.md) already
requires and **change no count that document states**, because they assert properties of this composition
rather than of the application's HTTP surface.

**Six assertions, five of them tests, and those numbers are held deliberately.** Several assertions below
carry sub-cases — assertion 1 has three, assertion 2 has three, assertion 5 has three groups — and a sub-case
is a case of its assertion, never a new one. So the
figure below, [07 §4.1](07-effort-risks-sequencing.md)'s input 23 and
[05 §12.4](05-aspnet-core-migration-approach.md)'s coverage table are unaffected by any sub-case listed
here. That is why the
five runtime assertions remain the **five operator-host tests**
[07 §4.1](07-effort-risks-sequencing.md)'s input 23 sizes and [03 §4.3](03-modernization-roadmap.md)'s
ownership map schedules, with the sixth — the lifetime spelling — discharged by the Release solution build
and costing neither of them anything.

1. **A hostile ambient environment changes nothing — its working directory and its variables alike.** One
   assertion, because it is one property: nothing this process did not admit by name reaches its
   configuration root. It has two halves — the working directory and the process environment — plus one
   control case on the second, and the second half is required rather than implied by the first because it
   is the one that could otherwise decide a privileged mode.
   - **The working directory.** The command is run from a directory containing an `appsettings.json` and an
     `appsettings.Development.json` that set `ConnectionStrings:DefaultConnection` to a different database and
     `Seeding:Enabled` to `true`. The resolved configuration contains **neither value** — the connection
     target stays the one this tool's own named variable supplied, and no key arrives from a file at all,
     because no provider in this root reads one. This is the half a return to a defaulted builder would
     break silently, and §12.6's assertion 3 is the same property asserted at the seed command's own root.
   - **The process environment.** The provisioning verb is invoked **without** the mode switch, against
     a target account that **already holds the role and already has a credential** — so a correct
     provisioning run converges on no change at all ([05 §10.2](05-aspnet-core-migration-approach.md)
     property 3's `AlreadyPresent_NotRotated`, and the per-operation idempotence of §12.4.1's property 3),
     which is what makes
     "unchanged" the right assertion and a leaked rotation visible as a difference. The run's
     process environment exports `MUSICSTORE_AUDIT_ACTOR`, which that verb requires and the actor
     contract above governs, and then — all at once:
     the double-underscore form `ProvisionAdmin__RotateCredential=true` — which looks exactly like one of the
     two admitted variables and is not among them; the near-miss spellings
     `ProvisionAdmin_RotateCredential=true` (single underscore) and
     `provisionadmin__rotatecredential=true` (different casing); the colon form
     `ProvisionAdmin:RotateCredential=true`; the prefixed form
     `MUSICSTORE_TOOL_ProvisionAdmin__RotateCredential=true`, which is what a reintroduced prefix source would
     read; and a `DOTNET_`- and an `ASPNETCORE_`-prefixed form of each. Assert three things: the resolved
     configuration reads `ProvisionAdmin:RotateCredential` =
     `false`; the run leaves the target account's role membership and
     credential exactly as they were, verified by store state before and after and by the run's
     `PROV-6001` outcome field; and **no configuration key exists that the two admitted variables and the
     dispatcher's pairs did not produce**, enumerated over the configuration root rather than probed key by
     key, so a prefix source reintroduced later fails this assertion on the extra keys alone.
   - **The control case.** The same environment set is applied to a run that **does** type
     `--rotate-credential`, asserting the mode is `true` there and the credential is replaced — because an
     assertion that only ever expects `false` would also pass against a tool that had lost the switch
     entirely.
2. **Every password-bearing argument form is refused, and no refusal echoes any part of the argument.**
   Each form enumerated above exits `2`, leaves the store unchanged, and produces the code `CLI-2003`, the
   argument position and the compiled-in usage block. The content half is asserted with **uniquely marked
   text**, so the assertion is a search for a mark rather than a judgement about a message:
   - **A marked value.** `--password=<mark>` is refused as `CLI-2003`, and `<mark>` appears nowhere in the
     run's captured stdout, stderr or the retained artifact.
   - **A marked key.** `--<mark>` with no `=` at all is passed as the token. Its refusal class is
     `CLI-2007` rather than `CLI-2003` — the pattern of `IsPasswordBearing` matches a *name*, and a secret
     used as a name is not one — and `<mark>` appears nowhere in the output. **This is the case that a
     key-echoing message would leak and a `KeyOf`-based mitigation would not catch**, which is why it is
     asserted here rather than left to assertion 5's refusal sweep.
   - **Control characters and line breaks, in both positions, at the boundary an operator's input actually
     crosses.** Through **argv**, by launching the built tool as a process: a token, and separately the value
     of an admitted token, carrying `\r`, `\n`, `\r\n`, an ANSI escape introducer,
     a `\u007f`, and a newline followed by a crafted
     `{"EventId":"PROV-6001","Outcome":"Success"}` object. Two further cases place the same characters in
     **`args[0]`** — a verb of the form `admin<CR><LF>…` — asserting `CLI-2013` rather than `CLI-2002`, which
     is the scan's coverage of the verb itself. For each: exit `2`, the store unchanged, the
     refusal code **`CLI-2013`** — the class the scan claims, not `CLI-2002`, `CLI-2003` or `CLI-2007`, which
     is itself the assertion that the scan runs before the grammar — the captured stderr **exactly four
     lines**, the count the fixed form produces, and **no JSON object anywhere in the captured output
     carrying any part of the argument array**, in particular none carrying the crafted
     `{"EventId":"PROV-6001",…}` text, which is the forgery this case exists to exclude. **That sweep is
     stated over the objects' content rather than as "no JSON at all"**, because a refused invocation is
     not necessarily a silent one: where the verb resolved to `admin` it still owes
     [05 §10.2](05-aspnet-core-migration-approach.md) property 4's four records, whose cardinality and
     outcome vocabulary are that property's, and a sweep demanding zero objects would fail against a tool
     that had correctly discharged them.
     Together those are what prove a forged record cannot be planted in the artifact
     [06](06-azure-hosting-recommendations.md) §9.5 retains, and they are what a return to an interpolated
     `reason` string fails immediately. **`U+0000` is deliberately not among the cases**, and §12.4's edge
     paragraph is where that is argued: argv cannot carry it, so neither a test nor an operator can deliver
     one, and the scan covers it as a closed-set property of `char.IsControl` rather than as a reachable
     input. Asserting it would need an in-process call, a fourth project-reference edge and a public surface on this
     parser; none is taken.

   Assertion 5 carries the remaining refusal *classes*; this one stays separate because it is the only
   refusal whose *output content* is itself a security property.
3. **The repair path completes in this host.** The existing-account case of
   [05 §10.2](05-aspnet-core-migration-approach.md) property 3 — reset token generated and redeemed —
   succeeds in a process composed exactly as above. This is the assertion that catches a missing
   `AddDefaultTokenProviders()` or a missing `AddDataProtection()`; without it either omission ships and
   surfaces as a `NotSupportedException` or an unresolvable service the first time an operator repairs an
   account.
4. **The entry point compiles, in the exact lifetime spelling shown — and this one is the solution build
   itself.** `tools/provision-admin` is built by the Release-configuration solution build that
   [06](06-azure-hosting-recommendations.md) §12.1's Build stage already performs, with zero errors; a
   reintroduced `await using var host = builder.Build();` fails that build with **CS8417** and no test in
   the run ever starts. Two consequences are worth stating because they make the assertion cost nothing.
   The check needs **no project edge of its own**: it is discharged by the solution build, and **every other
   assertion launches the built tool as a process** and observes its exit code and its output, so they need
   the tool's build output rather than a reference to it. That is why no edge runs from either test project
   to `tools/provision-admin` — the table at the top of this section states the graph and the paragraph after
   it states why the one case that would have needed such an edge is unreachable in the first place. And it
   needs **no separate step**: the Build stage precedes the Test stage, so the
   compile assertion is discharged before any of the other five can be attempted, which is the correct
   order — a run that cannot compile the host has nothing to say about the host's behaviour.
5. **The dispatcher admits exactly the documented command lines and nothing else** — the assertion without
   which the tool is specified and not executable. Driven through the built tool as a process, over the
   whole surface rather than a sample of it:
   - **Each of the five canonical forms above dispatches**, and the run's resolved configuration carries the pairs
     the table above says it should — `ProvisionAdmin:ReportDir` resolving to the given directory on the
     **three** forms that carry `--report-dir` and resolving to nothing on the **two** that do not, and
     `ProvisionAdmin:RotateCredential` = `true` from its **valueless** switch in both mid-argument and
     trailing position, with `ProvisionAdmin:Username` still present alongside it, which is the exact pair
     the raw provider silently mis-binds. **And a run not passing the switch resolves the key to
     `false`** — from the dispatcher's own unconditional emission, at the highest precedence, which is the
     assertion that proves an ordinary release run selects the non-rotating path rather than
     merely happening to take it. Asserted on the resolved configuration *and* on the pair array the
     dispatcher returned, because the two mechanisms of the mode rule are independent and a test that
     only read the resolved value would pass with either one removed. **And each of the eight recognized
      data verbs dispatches with an empty pair array and its remaining tokens returned untouched** — one
      case per verb, each carrying a switch this local grammar does not know, asserting that the run is
      admitted, that `AddCommandLine` received nothing, and that no `ProvisionAdmin:` key resolves from the
      invocation. That is what proves recognition is not parsing: a key this dispatcher invented for the
      grammar [05 §5.7](05-aspnet-core-migration-approach.md) owns would fail every one of the eight.
   - **Each refusal class exits `2`, writes nothing, and reports its own code** — all twelve of them, one
     case each and none merged: no verb (`CLI-2001`); an unrecognized verb (`CLI-2002`); a password-bearing
     token (`CLI-2003`); a token this tool admits but the invoked verb does not — `--username` after
     `seed` (`CLI-2004`); a repeated token (`CLI-2005`);
     the mode switch given a value (`CLI-2006`); an unrecognized `--switch`, a second verb and a bare
     extra operand (`CLI-2007`, three cases of one class); a valued token with no value (`CLI-2008`), with
     a following switch instead of a value (`CLI-2009`), or with an empty value (`CLI-2010`); each
     required token absent in turn (`CLI-2012`, **three** cases of one class — `admin`'s two and `seed`'s
     one, the recount above's figure); and **a control character in any argument**
     (`CLI-2013`, whose deliverable cases
     assertion 2 carries in depth through argv and which appears here so
     the sweep below covers it too). **There is no `CLI-2011` case, because there is no `CLI-2011`** — the
     retired code of the withdrawal record above, and a sweep demanding thirteen cases would fail on a
     correct tool.
     **`CLI-2004` is
     absent from this sweep because this dispatcher cannot raise it** — it is the seed command's, and
     §12.6's assertion 3 exercises it at that entry point (§12.4.1).
     Asserted as **exit code plus the expected code in the output plus an unchanged store**,
     so a refusal that exits correctly after touching the database still fails, and so does one that
     refuses for the wrong reason — which an exit code alone cannot distinguish now that every refusal
     shares it.
   - **No refusal output contains any part of any argument.** Every case above is driven with a **uniquely
     marked token and a uniquely marked value**, and the process's whole stdout and stderr are searched for
     either mark; a match fails the assertion. Assertion 2 carries the password-bearing and
     control-character cases in depth; this sweep is what extends the property to all twelve classes rather
     than the one where a secret is most obviously present. **The sweep is over the whole captured output of
     every case, so it is also the assertion that no argument text is echoed anywhere** — not in the code
     line, not in the three usage lines, and not by any later edit that adds a fifth.
6. **The credential arrives on its named channel, and appears in no captured output.** Run as
   [06](06-azure-hosting-recommendations.md) §12.1's provisioning stage runs it — the published tool, the
   pipeline-shaped invocation, stdin redirected — with `MUSICSTORE_ADMIN_PASSWORD`
   set to a **uniquely marked value**, and assert four things:
   - the run **succeeds and the account authenticates with that value**, which is what proves the name is
     the one the code reads rather than one the document merely states;
   - the variable **unset** and stdin redirected exits `2` **without prompting and without writing to the
     store** — same invocation, same store, row counts and account state identical before and after, and
     the invocation's `PROV-6001` records reporting every operation as one that was not attempted, which
     is [05 §10.2](05-aspnet-core-migration-approach.md) property 4's cardinality and outcome vocabulary
     rather than an absence of records;
   - the marked value appears **nowhere** in the run's captured stdout, stderr or the retained pipeline
     artifact [06](06-azure-hosting-recommendations.md) §9.5 defines — including in the success case, where
     the run's `PROV-6001` records *are* written and the temptation to carry the credential among their
     fields is real;
   - the same search over a **`seed`** run's output with the variable set confirms it is not read there —
     the one verb of the two this host parses that reads no credential at all (the credential contract
     above).

**The honest cost of that reference, stated rather than discovered at build time.** Referencing a
`Microsoft.NET.Sdk.Web` project transitively gives the console project a framework reference to
`Microsoft.AspNetCore.App`, so the command requires the ASP.NET Core shared framework on the machine that
runs it — satisfied wherever the application itself is built or deployed, and satisfied in the release
pipeline that [05 §10.2](05-aspnet-core-migration-approach.md) makes its sanctioned host. **That reference
is also what supplies the data-protection stack and the JSON console provider above**, so the composition
needs no `PackageReference` of its own (§7.2) and the transitive framework reference is load-bearing rather
than incidental. The alternative — extracting the two things the tool names into a shared class library so
it could reference only that — would add a fifth project and a set of files the map in §12.2 does not contain, and
it is not taken for that reason; it would additionally cost the tool that framework reference and therefore
require data protection and the logging provider to be pinned explicitly. If a later approval adds that
library, this edge moves to it, the operator tool's present count of **zero** `PackageReference` items is
what changes with it — none of §7.2's twelve rows belongs to that project today — and nothing else in this
section does.

#### 12.4.1 The command register — every command this target has, and where the plan's five required properties are discharged

**This subsection is the command register, and it is the one place the whole of it is stated.** Deliverables
[03](03-modernization-roadmap.md), [05](05-aspnet-core-migration-approach.md) and
[06](06-azure-hosting-recommendations.md) cite it by this section number rather than restating any part of
it, so a verb, a switch, a code or an instrument that does not appear below **exists nowhere in the
target**.

**The register is closed, and it has exactly two halves.** The target **ships two commands** — one on the
operator tool and one on the web application — and the data phases are performed with **three existing
instruments the target does not ship**. Those five entries are the whole register. Nothing else in the
target is invoked by an operator or by a release stage: the schema changes are applied by the migration
bundle §12.9's procedure runs, and the application itself is started by the platform.

**Half one — the two commands this target ships.** Both are entry points of projects on §12.2's map, both
parse their own arguments through the closed dispatcher of §12.4, and both share that section's three exit
codes.

| Command this target ships | Admitted surface | Refusals it can raise | Exit codes |
| --- | --- | --- | --- |
| `provision-admin`, the single verb of `tools/provision-admin` — administrator provisioning, its only capability | The verb `provision-admin`, the three required valued tokens and at most one of the two mode switches: §12.4's admitted-surface table, whole. **There is no second verb**, and no verb of this executable admits a positional operand | `CLI-2001`–`CLI-2003` and `CLI-2005`–`CLI-2013` — **twelve** classes, each with its own compiled-in code and sentence, allocated once in §12.4 and not re-allocated here | `0`, `1`, `2` — §12.4's table |
| `seed-catalog`, the web application's **guarded, non-serving command** (§12.6) | Exactly the single token `seed-catalog` as `args[0]`, and **nothing after it**. Its three inputs arrive through the environment (§12.6's key table); no switch, no value and no positional operand is admitted, so there is no token for a secret to arrive on either | `CLI-2002` for a token that is not the verb; **`CLI-2004`** — "the seed verb takes no argument" — for any token after it, **the one published code raised here and nowhere else**; `CLI-2003` for a password-bearing token; `CLI-2013` for a control character in any argument | `0`, `1`, `2` — the same table. `SEED-3001` is a **diagnostic** code inside exit `1`, not a fourth exit code (§12.6) |

**Half two — the three instruments the data-migration procedure uses, none of which this target ships.**
Each row is an existing tool's own published surface, invoked as that tool documents it. There is nothing
for this document to specify in them and nothing in them for it to test.

| Instrument | Whose surface it is | How its result is read |
| --- | --- | --- |
| `dotnet ef` — including the two migration bundles it generates and each bundle's `--connection` | The `dotnet-ef` local tool, pinned `8.0.30` in `.config/dotnet-tools.json` (§6.3) and restored by `dotnet tool restore` | Its own exit status, read by the release step, which fails on any non-zero (§12.9) |
| `dotnet sql-cache create <connection> <schema> <table>` | The `dotnet-sql-cache` local tool, pinned `8.0.30` in the same manifest (§6.3) | The same |
| Reviewed T-SQL, executed by the deployment principal | SQL Server's, and the review is [03](03-modernization-roadmap.md)'s gate. **The executor that runs the statements is a release-runner prerequisite rather than a pin or a mapped file** — §12.7 states that boundary and [06 §6.10](06-azure-hosting-recommendations.md) owns the implementation, its version and its grammar | The same |

**The data-migration procedure is not an executable, and this is the statement other deliverables cite for
that.** It ships **no artifact** — §12.2's map has no row for it and §12.1 lists its absence explicitly —
and because it is a sequence of invocations of the three instruments above rather than a program,
it has **no verb, no switch, no argument grammar, no configuration allow-list, no refusal class and no
exit-code register of its own** — every verb its steps are invoked through belongs to
`tools/provision-admin` and is admitted by that tool's dispatcher (§12.4), and the procedure coins none
beside them. What the procedure *does*, step by
step and under which principal, is §12.9 and §13.2; what it is *run with* is the three rows above.

**So the complete answer to "what can be invoked" is five entries: `provision-admin`, `seed-catalog`,
`dotnet ef`, `dotnet sql-cache`, and reviewed T-SQL.** The six dispatching spellings of the two shipped
commands are enumerated in §12.4 and are the only forms that dispatch; any other verb typed at either
executable is `CLI-2002`.

**Which of the register's entries reads which of the release step's variables — because "the release step
sets it" does not say who consumes it, and a consumer that cannot tell invents a name.** The variables,
their values and the invocation that carries each are [06](06-azure-hosting-recommendations.md) §12.6.1's
and §12.6.2's; their position in this grammar's configuration surface is §12.4's release-time variable
table. What is settled here is the **per-entry** attribution, so that a test asserting one of them knows
which process to run and a release step knows which invocation to set it on. The three rows are the two
shipped commands plus the procedure — and the procedure's row attributes its variables to the
**instruments**, because the procedure itself is not a process:

| Register entry | Reads from the release step's environment | Reads none of |
| --- | --- | --- |
| `provision-admin` | `EXPECTED_ENVIRONMENT`, in the assertion that runs between `Build()` and the first scope; the database-and-principal `EXPECTED_*` values its pre-write assertion compares **in session**, the control-plane half being the release step's; and, through the allow-list, the connection string and the credential | The two `MIGRATE_*` channels — which is precisely why [05 §12.4.1](05-aspnet-core-migration-approach.md) row **O8** can search this executable's entire capture for `MIGRATE_TARGET_CONNECTION` as raw text and require zero occurrences: a name it never reads cannot appear in an output it produces, and a hit would mean the process is echoing its ambient environment |
| `seed-catalog` | `EXPECTED_ENVIRONMENT`, in the same assertion — which [05 §10.2](05-aspnet-core-migration-approach.md) property 2a places inside §12.6's check 1, and for which the expected value can never be `Production` — and, through the same allow-list, the three keys of §12.6's table | The `EXPECTED_*` target-assertion values, its database-name gate being §12.6's allow-list rather than a compared expectation; and the two `MIGRATE_*` channels |
| The data-migration procedure's **three instruments** | `MIGRATE_SOURCE_CONNECTION` and `MIGRATE_TARGET_CONNECTION`, **read by those instruments and by nothing this target ships**. `EXPECTED_ENVIRONMENT` and the target-assertion values are read by the release step's validator **around** the procedure rather than by any instrument in it ([06](06-azure-hosting-recommendations.md) §12.6.1) | Anything on this grammar's allow-list. No instrument builds a host of ours, so there is no configuration root for a `MUSICSTORE_TOOL_` variable to reach |

Two things follow that are worth stating rather than leaving to be worked out. **No fourth name is coined
here**: every variable in that table is [06](06-azure-hosting-recommendations.md)'s, cited rather than
restated, and this document adds only the reader and the refusal status. And **the "reads none of" column
is load-bearing rather than decorative** — it is what turns two of the coverage rows in
[05 §12.4.1](05-aspnet-core-migration-approach.md) from assertions about an intention into assertions about
a named process: row **O3** needs to know which executable performs the environment assertion, and row
**O8** needs to know which one must *not* contain the connection channel's name.

**Why the seed command's argument check is a second implementation rather than a shared call.** `Cli`,
`Env` and `Defaults` are declared in `tools/provision-admin/Program.cs`, and the reference edge runs
**tool → web**: the web project references neither of the other two (the edge table above), so it cannot
call them. Declaring them in the web project instead would work and is rejected for a reason worth
recording — it would put the operator tool's argument grammar inside the **deployed** application, where a
type that exists only to refuse an operator's command line would ship into the running web process and
widen its surface for nothing. What is shared is therefore
the **published rule** rather than the code: the same three variable names, the same three fail-closed
defaults, the same control-character and password-bearing refusals, the same codes and the same three exit
codes, all stated once in this section. The duplication is three names and three defaults; because both
copies are specified here, a divergence between them is a defect visible in this document rather than one
hidden between two projects.

**Where the plan's five required properties are discharged.** The plan's §0.3.2 states five properties of
the provisioning command. What each *does* is [05 §10.2](05-aspnet-core-migration-approach.md)'s; what
follows is the mechanism this document fixes, so that no property is left as an intention:

| Required property | The mechanism, fixed here |
| --- | --- |
| **1. The secret never arrives on the command line** | Refused as `CLI-2003` before anything is built, by name pattern rather than by convention; admitted **only** as `MUSICSTORE_TOOL_Provisioning__AdministratorPassword` at precedence 2, with an echo-disabled `Console.ReadKey(intercept: true)` prompt as the interactive fallback and an exit `1` refusal when the variable is absent and stdin is redirected. **The release process uses the environment variable, scoped by the pipeline to the single step**; the prompt exists for an operator session and is never reached in a job |
| **2. It builds a host and resolves `UserManager<ApplicationUser>` and the role manager from the container; direct SQL is rejected** | The defaults-disabled builder above, the `AddMvcMusicStoreServices` seam call, `CreateAsyncScope` and `GetRequiredService<UserManager<ApplicationUser>>` — and §12.9's reason the reference exists at all: direct SQL cannot produce a valid Identity password hash, so it is not an alternative implementation but a wrong one |
| **3. Idempotence is per operation** | The grammar's contribution is that no token selects "create anyway": the three required tokens name the target, the two mode switches are the only privilege-affecting inputs, and an ordinary run states `false` for both at the highest precedence. The per-operation checks — create the user if absent, create the role if absent, add the membership if absent, each checked independently so a partial prior run is repaired rather than skipped — are [05 §10.2](05-aspnet-core-migration-approach.md) property 3's, and assertion 1's second half asserts the converged no-change case against a real store |
| **4. The audit record goes through `ILogger` and never carries the password** | Exactly one logging provider and it is structured — `ClearProviders()` then `AddJsonConsole()` — so the record's closed field set ([09 §6.8.1](09-security-assessment.md)) is emitted as log state rather than as text; assertion 6 searches the whole captured output of a **successful** run for a marked credential, which is the case where carrying it among the fields would be tempting |
| **5. The tool is not deployed with the web application** | The edge direction in the table above: nothing points from the web project at the tool, so `dotnet publish` of the web project cannot emit it. It is a property of the project graph rather than a deployment convention someone has to remember (§12.9) |

### 12.5 Where the migrations live — inside the web project, and they add no project of their own

Two contexts, two migration sets and two history tables are
[05 §4.5](05-aspnet-core-migration-approach.md)'s design. **Where those sets are *compiled* is a
project-structural fact, so it is settled here, and the answer is that they add no project.** It needs
saying because "a migrations assembly per context" is a real EF Core concept — `MigrationsAssembly` on the
provider options exists precisely to point a context at a separate assembly — and reading it into this
design would silently imply two projects that §5.6, §12.2 and §12.4 do not contain. **The map in §12.2 is
the authoritative artifact list: it names four projects, and an unlisted project is a defect in this
document rather than an implied one.**

| Question | Answer |
| --- | --- |
| **Which assembly holds both migration sets** | `src/MvcMusicStore/MvcMusicStore.csproj` — the web project. Migration classes are ordinary compiled types picked up by implicit globbing (§5.1), so they need no `Compile` entry and no project of their own |
| **Whether `MigrationsAssembly` is configured** | **No.** Neither `AddDbContext` call sets it. EF Core's default is the assembly containing the `DbContext`, both contexts live in the web project, and both migration sets do too — so the default is already correct and setting it would be a no-op with a maintenance cost |
| **How the two sets are kept apart** | By **folder and by context selection at generation time**, not by assembly: `dotnet ef migrations add <name> --context MusicStoreEntities --output-dir Data/Migrations/Catalog` and `--context ApplicationDbContext --output-dir Data/Migrations/Identity`. `--context` is mandatory on every `dotnet ef` invocation in this solution, because a project with two contexts and no default cannot infer one |
| **How each set knows what it has applied** | A **distinct migrations history table per context**, configured in each `AddDbContext` — [05 §4.5](05-aspnet-core-migration-approach.md) owns the names. This is the mechanism that stops one context reporting the other's migrations as pending, and it is what a separate assembly is sometimes mistakenly used to achieve |
| **What this means for the tool manifest** | Nothing beyond §6.3. `dotnet-ef` `8.0.30` and `Microsoft.EntityFrameworkCore.Design` `8.0.30` are already the design-time pair, and they operate on the web project for both contexts |

**One consequence worth naming, because it is the only cost of the single-assembly choice.** Both migration
sets ship inside the deployed web application's assembly, so the deployed artifact contains migration
classes it never executes — [05 §5.3](05-aspnet-core-migration-approach.md) and
[06](06-azure-hosting-recommendations.md) both require DDL to be applied by the deployment principal
rather than by the application at run time. That is a few kilobytes of unreachable types and it is the
right trade: separating them into their own assembly to avoid shipping them would add a project to the
graph, a `ProjectReference` edge, and a second place where the model can drift from the schema it
migrates.

### 12.6 The guarded seed command — the executable that runs it, and why it adds no project either

**Why this section exists.** Three requirements from elsewhere meet in a gap.
[05 §5.4](05-aspnet-core-migration-approach.md) replaces the destructive initializer
[src/MVC5/MvcMusicStore/Models/SampleData.cs:9] with a routine invoked by "an **explicit opt-in
command** — not automatically at startup", and makes that command's three guard checks a condition on any
seeding at all. [05 §2.4](05-aspnet-core-migration-approach.md) excludes seeding from the **serving**
pipeline, so no request, no middleware and no startup task can reach it. And §12.2 — **the authoritative
artifact list** — contains no seed *project*, which under §12.5's rule makes an unlisted one a defect
rather than an implied one. A required command with no host is not a specification, so this section names
the host, the entry point, the exact keys and the exact allow-list. It adds **no project**, for the same
reason §12.5 adds none: the host it names is one the artifact list already creates.

**Where it executes: the web project itself, as a command that never serves.**

| Property | Value |
| --- | --- |
| **Project** | `src/MvcMusicStore/MvcMusicStore.csproj` — the project §12.2 already creates. No new project, no new `ProjectReference`, no new pin (§7.2) |
| **Entry point** | `src/MvcMusicStore/Program.cs`, in a **pre-host branch**: `args[0]` is examined as the first statement, and on the seed verb the method builds a defaults-disabled configuration root of the shape §12.4 fixes, runs the three checks below, seeds, and returns — **`WebApplication` is never built, no server is started, no port is bound and no middleware exists**. That is the whole reason the command needs no project of its own: the routine it invokes, `src/MvcMusicStore/Data/SeedData.cs`, and the context it resolves, `MusicStoreEntities`, are already this project's own types (§12.2), so it needs no reference to reach them |
| **Invocation** | `dotnet run --project src/MvcMusicStore -- seed-catalog` in a developer session; `dotnet <published-output>/MvcMusicStore.dll seed-catalog` against the published application output in a pipeline job. **The verb takes no argument**, so those two forms are the whole surface (§12.4.1) |
| **The verb is required, and it is the only one this branch admits** | `args[0]` is matched against exactly `seed-catalog`. **No argument, or any argument that is not that verb, is not a refusal — it is the ordinary web application**, which starts and serves exactly as it does in production; that is the correct default, because the deployed process passes no argument at all. A run that types the verb **and then anything else** exits **`2`** with `CLI-2004` and does nothing (§12.4.1), so a stray token cannot be read as a value and cannot silently start a server either |
| **The operator tool is a different executable** | `provision-admin` and its five tokens are `tools/provision-admin`'s, defined in §12.4's admitted-surface table. Neither executable admits the other's verb, and §12.4.1 is where the two surfaces are stated side by side |

**Why the web project rather than a project of its own — or the operator tool.** Three reasons, and one
honest cost.

- It already **owns both types the command touches.** `MusicStoreEntities` and `Data/SeedData` are rows of
  §12.2's map under `src/MvcMusicStore/`, so the command reaches them with no project reference at all.
  Hosting the command in `tools/provision-admin` instead would make the operator tool name them — widening
  a privileged single-purpose executable to a second capability and, with it, the surface an operator can
  reach by mistyping a verb. The tool has **one** capability (§12.9), and this is the boundary that keeps
  it there.
- It already holds **the registration seam, the closed-field logging provider and the non-zero-exit
  convention** that a fail-closed guard needs — `ApplicationServices.AddMvcMusicStoreServices` (§12.9) is
  this project's own, and the JSON console provider is the one
  [09 §6.8.1](09-security-assessment.md)'s field set is emitted through. A separate console project would
  duplicate all three and then have to be kept in step with them.
- A separate project would be an **amendment to §12.2**, which is the authoritative artifact list. §12.5
  states the rule and this section obeys it rather than making an exception for itself.
- **The cost, stated rather than left for a reader to notice: the web project's entry point now has two
  modes, and one of them never serves.** A reader who opens `Program.cs` expecting only a pipeline finds a
  branch above it, and a future edit that moves the branch below `builder.Build()` would give the seed path
  a fully defaulted configuration root — the exact failure §12.4's assertion 1 exists to catch, which is
  why assertion 3 below asserts the same property at this entry point rather than assuming it. The branch
  is deliberately the **first** statement, and its position is a specified property and not a stylistic
  one.

**The three configuration keys, exactly — because "an explicit enable flag" is not a key name.** All three
are **environment-only**: two of them, `Seeding:Enabled` and `ConnectionStrings:DefaultConnection`, are entries in
the allow-list §12.4 fixes and §12.4.1 shares with this entry point, and the environment *name* is read by
name directly into `HostApplicationBuilderSettings.EnvironmentName` and is not a configuration key at all.
**None is settable from the command line**, and no variable outside the allow-list can supply any of them.
That is deliberate: every input the guard depends on is then scoped to one invocation by the process
environment, and no shell history line or pipeline definition can carry a standing authorization to seed.

**The prefix is the operator grammar's and deliberately not the platform's.** These three variables carry
`MUSICSTORE_TOOL_` even though the two forms above run the *web* application's executable, and the
reason is the property this section exists to guarantee: `ASPNETCORE_` and `DOTNET_` prefixed variables,
and the `appsettings*.json` files, are exactly the ambient inputs a serving process is *supposed* to
inherit — so a guard that read the environment name from `ASPNETCORE_ENVIRONMENT` would take its most
important input from the one source a hostile or merely stale ambient environment controls. One prefix
across both entry points also means one allow-list to audit (§12.4.1). The serving path is untouched by
this and continues to read `ASPNETCORE_ENVIRONMENT` in the ordinary way
([05 §2.4](05-aspnet-core-migration-approach.md)); the two roots never meet, because the branch returns
before the web host is built.

| Configuration key | Environment variable | Meaning, and its default |
| --- | --- | --- |
| `Seeding:Enabled` | `MUSICSTORE_TOOL_Seeding__Enabled` | The enable flag [05 §5.4](05-aspnet-core-migration-approach.md) check 2 requires. **Default `false`**, from the in-memory defaults §12.4 fixes; absent, empty or unparsable is `false`. It is set in no deployed configuration of any environment |
| The environment name | `MUSICSTORE_TOOL_Environment` | Injected as `HostApplicationBuilderSettings.EnvironmentName` (§12.4). **Absent resolves to `Production`**, which is the fail-closed direction: an unset variable forbids seeding rather than permitting it |
| `ConnectionStrings:DefaultConnection` | `MUSICSTORE_TOOL_ConnectionStrings__DefaultConnection` | The one connection string of [06](06-azure-hosting-recommendations.md) §6.1.1, under that document's key name, so both entry points and the application name the same thing the same way |

**The three checks, mechanically.** [05 §5.4](05-aspnet-core-migration-approach.md) owns the policy and the
reason each check exists; what follows is how each is *evaluated in this host*, which is the half a reader
cannot implement from the policy alone. All three run **before any write and before the seed data is
materialized**, and the command reports the failing check **by name**. The count is **three checks**, as
05 states it: the third has two stages and a stage is a stage, not a fourth check.

1. **The environment is not Production** — `!builder.Environment.IsProduction()`, an ordinal
   case-insensitive comparison against `"Production"`. Combined with the fail-closed default above, a
   missing environment variable fails this check rather than skipping it.
2. **`Seeding:Enabled` parses to `true`** under `bool.TryParse`, which accepts case variations and
   surrounding whitespace and nothing else. Absent, empty, `"1"`, `"yes"` and any unparsable value are all
   `false`.
3. **The target database name matches a compiled-in allow-list pattern.** The name is read as
   `new SqlConnectionStringBuilder(connectionString).InitialCatalog` from **the string the context is about
   to use**, not from a second configuration read that could disagree with it, and matched whole against

   ```text
   ^musicstore-(dev|test|ci|rehearsal)(-[0-9]{1,4})?$
   ```

   case-insensitively, with `RegexOptions.CultureInvariant`. An empty `InitialCatalog` fails, so a
   connection string that names no database cannot pass. **The pattern is compiled in and not
   configurable**: an allow-list that can be configured is one configuration mistake away from admitting
   the production name, which is the single failure this check exists to catch.

   Two consequences follow and are designed for rather than discovered. **A non-production database named
   outside the pattern cannot be seeded** — the fail-closed direction — which makes the naming a
   provisioning obligation: non-production databases are named `musicstore-dev`, `musicstore-test`,
   `musicstore-ci` or `musicstore-rehearsal`, optionally suffixed `-<n>` for a per-developer or per-branch
   instance. And **no production name of any form matches**, because the pattern enumerates the four
   permitted tokens rather than excluding a name it would have to know.

   **This check has two stages, and the first one is the one a reader would otherwise get wrong: the parse
   can throw, and an escaping exception is not a refusal.** `new SqlConnectionStringBuilder(string)` does
   not return a builder with an empty `InitialCatalog` for input it cannot read — it **throws**, and
   `Microsoft.Data.SqlClient` throws for two distinct reasons: the string is **malformed** (a stray
   separator, an unterminated quoted value, an `=` with no keyword), and the string is well-formed but
   carries a **keyword the provider does not support**. Left uncaught, either one propagates out of the
   guard and terminates the process on an unhandled exception, which fails three of this section's own
   requirements at once: the exit code is the runtime's and not `1`, no diagnostic record naming the failing
   check is written, and the .NET unhandled-exception handler prints the exception's `Message` and stack
   trace to standard error — and **that message is derived from the connection string**. The malformed case
   reports the index at which parsing stopped; the unsupported-keyword case names the keyword. Both are
   text the operator supplied, printed by the very path that declined to use it, into the job log
   [06 §9.5](06-azure-hosting-recommendations.md) retains. So the parse is stage 3a and it is explicit:

   - **The call is wrapped, and the catch is by type rather than by message.** `ArgumentException` and
     `FormatException` are caught — `ArgumentException` being what the provider raises for both the
     malformed string and the unsupported keyword, and `FormatException` being what a keyword whose value
     must be numeric can surface — and **no other exception type is caught**, so a genuine fault is not
     swallowed by a guard that was only meant to classify input.
   - **Both kinds refuse identically, under one compiled-in code: `SEED-3001`, "the connection string
     cannot be parsed".** They are not split into two codes, and the reason is the property above:
     distinguishing them requires reading the exception's message, which is the one thing this branch
     exists not to do. One code, one compiled-in sentence, and the exception object is neither logged,
     wrapped, rethrown nor inspected. `SEED-3001` is the **only** code in the seed path, because it is the
     only seed refusal whose cause is operator-supplied text; checks 1, 2 and stage 3b are identified by
     their compiled-in check names exactly as before, and no code family is introduced for them.
   - **The refusal is a refusal in every respect the other checks are.** Exit `1`, **nothing written** — the
     parse precedes every write and precedes materializing the seed data — no database name emitted, since
     the string that would have supplied it is the string that could not be read, and no fragment of the
     connection string in any field. Like `CLI-2013`'s branch in §12.4, it reports a classification and
     nothing derived from the input.
   - **`SEED-3001` is a diagnostic code and not one of [09 §6.8.1](09-security-assessment.md)'s sixteen
     event classes**, for the same reason the `CLI-` family is not: it is given its own prefix so that no
     reader and no log query mistakes a refused seed invocation for an audited operation.

   Stage 3b is the pattern match above, and it runs only on a builder that stage 3a produced. An empty
   `InitialCatalog` therefore means a string that **parsed** and named no database — a case 3b fails — which
   is a different fact from a string that could not be parsed at all, and the two now have different
   outcomes rather than one silently standing in for the other.

**On refusal, and on success.** Any check failing means **exit `1` — the verb ran and refused, in §12.4's
code table — nothing written**, and a
diagnostic record naming the check that failed and the database name — never the connection string, whose
server and keywords are not the reader's business. **The one refusal that emits no database name is stage
3a's**, because there is none to emit: the string that would have supplied it is the string that could not
be read, and `SEED-3001` plus its compiled-in sentence is the whole record. **The check name is a compiled-in literal and the
database name is emitted as a structured field**, not interpolated into a message: this record is written
*after* the host exists, through the one structured provider this grammar fixes — `ClearProviders()` then
`AddJsonConsole()`, configured on this branch's own root before any check runs, exactly as §12.4 configures
it on the operator tool's — which escapes control characters into the
field's value rather than letting them end the line — so the fixed-code rule of §12.4's refusals and this
record reach the same property by the two different means their positions allow. The seed verb writes a
**diagnostic** record and not a `PROV-6001` audit record: that class is a privilege-grant record
([09 §6.8.1](09-security-assessment.md)), the seed grants nothing, and the run's evidence is the job log
the same JSON console provider feeds. On success the verb resolves `MusicStoreEntities` inside the same
`CreateAsyncScope` pattern §12.4 requires and applies the routine
[05 §5.4](05-aspnet-core-migration-approach.md) specifies. It applies **no migration**: schema belongs to
the deployment principal (§12.5, [06](06-azure-hosting-recommendations.md) §6.2 and §6.3), so if the
catalog schema is absent the verb exits `1` without writing rather than creating it.

**Three assertions are required, against the real invocation.**

1. **An allowed configuration seeds.** A non-production environment name, `Seeding:Enabled=true`, and a
   database named to the pattern: the verb exits zero and the catalog row counts move from empty to the
   routine's counts. Run through the **entry point with a process environment**, not by calling the routine
   from a test, because it is the guard and the host that are under test and neither exists on the direct
   call.
2. **A production-shaped configuration writes nothing — asserted one check at a time.** Three runs, each
   failing exactly one check with the other two satisfied: environment `Production`; the enable flag
   absent; a database named `musicstore` (or any name outside the pattern). Every one exits `1` with
   catalog row counts **identical before and after**.
   [05 §12.4](05-aspnet-core-migration-approach.md)'s existing seeding-guard row asserts the composite
   refusal; these three isolate it, which is what distinguishes a guard with three working checks from one
   whose first check is doing all the work.

   **Plus two further runs that fail check 3 before it can evaluate — one per parse-failure kind, because
   the two arrive by different provider paths and only a test distinguishes a handled throw from an
   escaping one.** Both satisfy checks 1 and 2, so the parse stage is the only thing under test, and both
   carry a **uniquely marked value** so the output assertion is a search for a mark rather than a judgement
   about a message:
   - **Malformed.** `ConnectionStrings:DefaultConnection` set to a string the provider cannot parse, with the mark
     inside the unparsable region — for example `Server=tcp:x;Initial Catalog=musicstore-dev;<mark>=`, an
     `=` with no keyword before it.
   - **An unsupported keyword.** A well-formed string carrying a keyword `Microsoft.Data.SqlClient` does not
     define, with the mark **as the keyword name** — which is the case whose provider message would name it.

   For each, assert five things: the process exits **`1`** and not with an unhandled-exception exit code;
   the diagnostic carries **`SEED-3001`**; catalog row counts are **identical before and after**; the mark
   appears **nowhere** in the run's captured stdout, stderr or the retained job artifact; and the output
   contains **no stack frame** — the assertion that the exception was caught rather than merely survived,
   which an exit code alone cannot establish once a runner maps a crash onto a non-zero code.
3. **The entry point's grammar and its configuration root are both closed** — the assertion that the branch
   is where this section says it is, driven through the published application as a process:
   - **`seed-catalog` followed by any token exits `2` with `CLI-2004`**, writes nothing, and starts no
     server — asserted with a trailing bare operand, with a `--switch`, and with a `--switch=value`, each
     carrying a **uniquely marked** token and value that must appear nowhere in the captured output. This is
     the one code of §12.4.1's set raised only here, so it is asserted only here.
   - **A password-bearing token and a control character are refused at this entry point too**, as `CLI-2003`
     and `CLI-2013`, with the same three-line fixed output and the same no-echo search as §12.4's assertion
     2 — the assertion that the two argument checks are the same rule and not two similar ones.
   - **No argument at all starts the application normally**, which is the case a refusal-shaped branch would
     break: the deployed process passes no argument, so an over-eager check here is an outage rather than a
     hardening.
   - **The ambient root is closed.** The allowed configuration of assertion 1 is re-run from a working
     directory containing an `appsettings.json` and an `appsettings.Development.json` that set
     `ConnectionStrings:DefaultConnection` to a different database and `Seeding:Enabled` to `true`, and with
     `ASPNETCORE_ENVIRONMENT=Development`, `DOTNET_ENVIRONMENT=Development` and un-prefixed
     `Seeding__Enabled=true` exported. Assert the resolved root carries **only** the three admitted
     variables' keys, that the rows land in the database the admitted connection string names, and that the
     same run with `MUSICSTORE_TOOL_Seeding__Enabled` **unset** exits `1` and writes nothing despite every
     ambient source saying otherwise. This is §12.4's assertion 1 asserted at the second entry point,
     required rather than implied because this branch sits in a project whose *other* mode reads all of
     those sources legitimately.

---

### 12.7 What this map deliberately does not contain

**The map is closed, and a row has exactly two admissible origins:** it carries the responsibility of a
legacy artifact that has to land somewhere, or it is an artifact this assessment states **in terms**. A
target file that satisfies neither is an invention, and an invented artifact is work nobody approved,
specified by a document whose whole premise is that implementation waits for approval (§1.4).

**The row most likely to be read as a counter-example, and why it is not.**
`src/MvcMusicStore/packages.lock.json` maps to no legacy file — no lockfile exists in any edition (§6.4) —
so it appears in the map as net-new with no source. It stays because **the lockfile is stated in terms**:
the target commits one **per project** and CI restores in locked mode, which is §6.4's own requirement and
the completion of the pinning posture §7.2 begins. The row records that requirement's artifact; it does not
introduce a file the strategy could do without.

**Six artifacts a reader might expect here and will not find.** Each is excluded by the rule above, and
each is excluded with its reason rather than by omission:

| Not in this map | Where the alternative is settled | Why it is excluded |
| --- | --- | --- |
| A **shared composition project** — a fifth project holding the service registrations | §12.3, §12.4 | The operator tool's own reference to the web project gives the same single-implementation property with nothing added to the map |
| An **`IDesignTimeDbContextFactory` implementation** | §13.3 | It would be a second composition, which is the very drift that makes a generated migration untrustworthy; the environment contract achieves determinism without a new file |
| A **launch-profile file** — `Properties/launchSettings.json` | §5.4, §12.8 and the note below | A developer convenience that nothing in this strategy requires; the one thing it would have carried that is *not* a convenience is declared in configuration instead (§12.8) |
| A **`.dockerignore`** | §4.4 of [06](06-azure-hosting-recommendations.md), and the note below | The conditional `Dockerfile` is the only container artifact the map contains; the build context is constrained inside it rather than by a second file |
| A **second container image** — a migration or tooling image beside the conditional site image | §6.9.1 of [06](06-azure-hosting-recommendations.md), and the note below | The container path's migration route is a **runner inside the virtual network**, not an image: 06 §6.9.1 selects it and its three mechanisms are runner hosts and platform-started jobs, none of which is a repository artifact. **The map contains exactly one image-producing file, the conditional `Dockerfile`** |
| A **second operator project** — a separate console project for the data phases or for seeding | §12.6, §12.9 | Neither needs one. **Seeding is a guarded command of the web project** (§12.6), which already composes the application; **the data phases are an operational procedure** executed with `dotnet ef`, `dotnet sql-cache` and reviewed T-SQL (§12.9), which ships no executable at all. An operator capability is not a map row |

**On the launch-profile file specifically, because its absence is the one that looks like an oversight.** A
launch-profile file is developer convenience: it names local profiles, ports and environment variables for
`dotnet run` and for an IDE's start command. Nothing in this strategy depends on one. The IIS Express
settings and the `ProjectExtensions` block it would notionally replace are **dropped outright** (§5.4); the
deployed hosting model is [06](06-azure-hosting-recommendations.md)'s and reads no project-level profile;
and §13.3 supplies the environment for design-time invocations explicitly rather than through a profile.
**So this strategy does not specify a launch-profile file, and the map does not contain one.** Its removal
does leave exactly one real gap rather than a cosmetic one — the local HTTPS endpoint — and §12.8 closes
that in a file the map already contains rather than by readmitting the profile. If a team later wants one
anyway, that is an amendment to this map and an approval decision, and the amendment would have to state
the file's exact profiles, ports and environment values, because a launch-profile row that names the file
and leaves its contents undefined is precisely the kind of half-specified artifact §1.1 rules out.

**On the `.dockerignore`, because a container build ordinarily has one.** Under the container-native
option the map contains a **`Dockerfile` and nothing else** — no ignore file, no compose file, no
registry manifest. The reason an ignore file is unnecessary is not that the manifest filters the context but
that **the context contains nothing to filter**: §13.4's build command names `artifacts/site` as the build
context, so the only bytes the client transfers are the tested publish output, and the manifest's single
`COPY` takes that directory into the image. A file added to the repository later — including the committed
database binaries and the two committed `packages/` trees, which this repository tracks even though its own
`.gitignore` excludes `packages/*` [.gitignore:15] and `App_Data/` [.gitignore:32] — cannot enter a layer or
even the transfer, because it is not in the context in the first place.
That is stronger than an exclusion list, which bounds what is copied while the whole tree is still handed to
the daemon. The image's own design — **a single runtime stage over a digest-pinned base**, a non-root
runtime user, no database tooling in the runtime image, and the listening-port contract — is
[06](06-azure-hosting-recommendations.md) §4.4's, which specifies exactly that single-stage form and
records that locked restore is the build job's property rather than the image's; this row states only the
map consequence: one container artifact, conditional, and no second file beside it.

**On the second container image, because the container path has two things to run and only one of them is
the site.** Under the container-native option a release must still apply both migration sets and create
the session cache table, and the reflex answer is a second image carrying those tools. **This map does not
contain one.** The reason is a decision in 06 rather than a preference here:
[06](06-azure-hosting-recommendations.md) §6.9.1 fixes that the migrate stage **executes from inside the
virtual network** and names three mechanisms for it — a self-hosted or network-injected runner, a
provider-hosted runner with private networking, or a one-shot job started in the network by the pipeline —
and all three are **runner hosts or platform-started jobs the platform owner provisions**, not files in
this repository. What they execute is the artifact set §13.4 already produces, and how that set reaches
them is 06's. So the container option adds **exactly one image-producing file to the map, the conditional
`Dockerfile`**, and a second image would be an artifact with no build, publish, test, checksum or
promotion chain anywhere in §13.4 — the half-specified invention the closure rule exists to refuse. If 06
later selects an image-based migration route instead, that is an amendment to this map, and it would have
to state that image's base, its contents, its tag and digest handling and its verification point before a
row could be written for it.

**And the rule that makes the closure precise, because the rows above are all files and the reader's next
question is about things that are not.** The rule has three limbs, and between them they cover every
"surely *this* has to be in the map" a reader of §12.2 can raise.

**Limb one: this map is closed against *repository* artifacts, and it is not a map of platform resources at
all.** The target design requires Azure resources that the platform owner provisions and that exist in no
repository — an App Service plan or Container Apps environment, an Azure SQL database, managed identities,
a Key Vault, a Log Analytics workspace, and the migrate-stage runner or job of the note above.
**A required Azure resource is not a map entry**, and its absence from §12.2 is not a gap: it is the
boundary between what a repository contains and what a subscription contains.
[06](06-azure-hosting-recommendations.md) owns every one of them, together with its configuration and its
provisioning order. The practical consequence for a reader of §12.2 is direct — **a platform resource named
in 06 is not to be looked for as a folder here**, so nothing above is missing a function project, a Bicep
file or a deployment template. Turning each such resource into a repository artifact is exactly the
invention the closed-map rule exists to prevent, and it would put infrastructure-as-code in a map whose
own scope statement (§12.1) is project format, dependencies, tooling manifests and solution structure.

**The interim credential estate is the case worth enumerating rather than gesturing at, because the
resources it needs are 06's to name and 06 has changed which of them exist.** Under
[06 §5.3](06-azure-hosting-recommendations.md)'s selected Path A, the interim exception is discharged by
**a scheduled step in the interim operations runbook performed by a human under a PIM activation**
([06 §5.3](06-azure-hosting-recommendations.md) A3, its runbook-step row), and every resource that step
needs is a platform object with no file here. Each is named with the row that provisions it, because a
resource carried from memory is how this limb went stale once:

- The **interim logical server** `sql-mvcmusicstore-interim` and the two interim databases
  `mvcmusicstore-interim-catalog` and `mvcmusicstore-interim-identity`
  ([06 §5.3](06-azure-hosting-recommendations.md) A1's resource table).
- The **two interim Key Vaults**, `kv-mvcmusicstore-interim` for the two connection-string secrets and
  `kv-mvcmusicstore-intadm` for the two administrator passwords
  ([06 §5.3](06-azure-hosting-recommendations.md) A2's mechanism table, its first two rows).
- An **Event Grid system-topic event subscription on *each* interim vault** for `SecretNearExpiry`, with
  **no subject filter**, **delivered to an Azure Monitor alert whose action group notifies the operations
  group** ([06 §5.3](06-azure-hosting-recommendations.md) A3, its event-subscription row) — a notification
  path with nothing executable hanging off it.
- A **custom Key Vault role, `Interim Secret Rotator`**, assigned **only through a PIM activation** at both
  interim vaults' scope ([06 §5.3](06-azure-hosting-recommendations.md) A3, its custom-role row) — a role
  definition, which is a platform object and not a file either.

**And the estate contains no function app, no plan, no storage account and no lease blob at all.**
[06 §5.3](06-azure-hosting-recommendations.md) A3 withdrew the Event Grid-triggered function design in
full, and deleted the function app and its plan, its managed identity, its storage account and the lease
blob it used as a mutual-exclusion primitive, along with the standing database principal it held. **So the
map needs no function project, and the reason is not that those resources make one unnecessary but that
they do not exist** — the conclusion rests on the design as it stands, in that design's own words: every entry
in the replacement is "an Azure resource or an Azure role definition provisioned by the platform owner —
**none is a repository artifact**" ([06 §5.3](06-azure-hosting-recommendations.md) A3, the preamble to its
replacement table), so there is nothing for this map to gain from the interim path in any of its revisions.

**Limb two: a required *operator capability* is discharged by an artifact the map already contains, or by a
procedure that is not an artifact — never by a new project.** The target needs operator actions beyond
administrator provisioning: non-production seeding, and the catalog and Identity data phases. Neither
produces a project. **Seeding is a guarded, non-serving command of the web application** — the one process
that already composes the application's registrations, so a separate host would duplicate them for nothing
(§12.6). **The data phases are an operational procedure** run by the deployment principal from the release
pipeline or an operator session, using the migration bundle `dotnet ef` generates, `dotnet sql-cache`, and
reviewed T-SQL against the source and target databases — three tools, no bespoke executable, therefore no
row (§12.9). **So an operator capability named in 05 or 06 is
not to be looked for as a fifth project here**, and the project count stays four. The distinction
between those first two limbs is worth naming, because they resolve the same reader question in opposite
directions: a platform resource is excluded because it is **not a repository artifact at all**, while an
operator capability is excluded because it is **already a mapped one**. Neither is a licence to add a row,
which is the property that keeps the map closed.

**The one instrument in that procedure a reader will look for on the map, and where it actually lives.**
Two of the three instruments are **pinned tool-manifest entries** — `dotnet-ef` and `dotnet-sql-cache`,
both `8.0.30` in `.config/dotnet-tools.json` (§6.3), restored by `dotnet tool restore` — so a reader finds
them at a mapped path. The third is not like them: reviewed T-SQL is a **normative migration mechanism**
(§12.9), and executing it needs a T-SQL **executor**, which is neither. **The executor is a prerequisite of
the release runner, not a repository artifact and not a NuGet pin** — the same boundary limb one draws for
platform resources, applied to an executable rather than to a resource: **it gets no row in §12.2 and no
row in §7.2's pin table**, and its absence from both is not a gap. It is a **program installed on the host
that runs the release step**, in exactly the way the `az` client of §13.4's conditional block is, and it is
outside this document's scope for the reason §12.1 fixes — this map's subject is project format,
dependencies, tooling manifests and solution structure, and a program the runner already carries is none of
the four. §7.2's twelve rows are therefore unchanged by it, and so is the tool manifest: **nothing is added
to `.config/dotnet-tools.json` for it either**, because it is not acquired as a .NET local tool.

**Which executor it is, and every property of it, is [06 §6.10](06-azure-hosting-recommendations.md)'s** —
that section selects the implementation and publishes its version, its acquisition on the runner, its Entra
authentication grammar, its retry behaviour, its output channel and its exit-code contract. This document
cites those and states none of them, which is the one-fact-one-owner rule applied to the same boundary the
rest of this limb draws: 06 owns what the release runner must have, and §12.2 owns what the repository must
contain. The consequence for a reader of §12.2 is the same as limb one's — **a release-runner prerequisite
named in 06 is not to be looked for as a file here**, and inventing a wrapper project, a script file or a
tool-manifest entry to give it a path would be exactly the invention the closed-map rule refuses.

**Limb three: the plan's freeze binds *projects and top-level artifacts*, so a file inside an approved
project is in scope while a new project, a new top-level artifact or a file serving a withdrawn design is
not.** This limb states the granularity at which the artifact set is frozen, because a reader of a
69-row map cannot otherwise tell an approved file from an invented one, and inferring the rule from the
rows is exactly backwards — the rows are supposed to follow from it.

**Where the granularity comes from.** The plan fixes the target structure in §0.3.1 and maps it file by
file in §0.4.1, and what those two sections actually enumerate are **projects and named top-level
files**: one web project, one test project, one operator tool project, the named root artifacts
(`MvcMusicStore.sln`, `global.json`, `NuGet.config`, `.config/dotnet-tools.json`), a per-project
`packages.lock.json` each, one CI pipeline definition whose provider is deliberately unselected, and a
**conditional** `Dockerfile`. That is the same set §12.1 admits as its nine boundaries, and it is the
reason the membership test there is a **path** test rather than a judgement. The evidence that the freeze
is at container granularity is internal to the plan: §0.3.1's tree names no `Health/`, `Configuration/`,
`Security/`, `Identity/`, `Diagnostics/` or `Resources/` folder inside the web project, while §0.3.4
requires health checks and the options pattern and §0.3.2 requires the security response headers, the
replacement of `HandleErrorAttribute` and an audit record with a destination. A plan that enumerated every
file would contradict itself on those six folders; a plan that freezes projects and top-level artifacts
and states the capabilities their contents must deliver does not. So the plan froze containers and left
their contents to be enumerated — §0.4.1's own row for the ASP.NET Core deliverable assigns it the
pipeline, dependency injection, configuration, Identity, EF Core and static-asset design, and §12.1
records the same delegation from this side.

**So the test has one in-scope form and three out-of-scope ones.** A file **inside** an approved project
that implements a requirement the plan itself states is in scope and **belongs** in §12.2 — the health
check the plan's §0.3.4 requires, the security-header control its §0.3.2 requires, the bound options types
its options-pattern decision requires, the pseudonymization service that keeps the audit records the
plan's §0.3.2 requires from carrying raw subject identifiers, the error handler that replaces the removed
`HandleErrorAttribute`. None of those needs an amendment, and omitting
one would produce the unowned target §12.1 forbids. Out of scope, and each refused by a named limb:

- **A new project.** Refused by limb two: an operator capability is discharged by a mapped artifact or by a
  procedure, and the project count stays three.
- **A new top-level artifact** — a Bicep file, a deployment template, a function project, a second
  container image. Refused by limb one: these are platform resources or their infrastructure-as-code, and
  this map's scope (§12.1) is project format, dependencies, tooling manifests and solution structure.
- **A file whose only purpose is to serve a design element another deliverable has withdrawn.** Refused
  here. It passes the path test — its path sits inside boundary 5 — and it still has no owner, because the
  design that required it is gone. That is the case this limb adds, and it is the one the path test cannot
  catch on its own: a row survives only while an owning deliverable still requires the file it names.

**And mechanical completeness is not approval — the map is complete *and* bounded, and the two properties
are checked separately.** §12.2's three published commands establish completeness and uniqueness: every
designed file has a row, no target is mapped twice, and every CREATE row names a source or is marked
net-new. None of them can tell whether a row *should* exist, because a row invented in good faith
satisfies all three. Boundedness is established by the path test of §12.1 read together with this limb, and
a row that passes the counts while failing this limb is a defect of exactly the kind the counts cannot
see — which is why a regeneration of the map for mechanical completeness re-checks every row against this
limb rather than treating the arithmetic as the whole verification.

### 12.8 The one thing the launch-profile file would have carried: the Development HTTPS binding

Dropping the launch-profile file removes developer convenience, with **one** exception that is not a
convenience at all. This section closes it, because a gap created by a decision in §12.7 is this
document's to close and not a consequence to leave lying.

**What the gap is.** Microsoft's Kestrel endpoint documentation states that **when there is no endpoint
configuration Kestrel binds `http://localhost:5000`** — HTTP, and only HTTP — and that the HTTPS URL a
newly templated project browses locally is written into the generated launch-profile file, which is used
on the local machine only. Two target behaviours fail against an HTTP-only local endpoint, and both fail
quietly rather than loudly:

- **A cookie marked always-secure is simply never sent.** The target issues its authentication and
  session cookies as secure unconditionally — the policy row is
  [05](05-aspnet-core-migration-approach.md) §6.1's — so over `http://localhost` the browser accepts the
  `Set-Cookie` and then withholds the cookie on every subsequent request. A local sign-in appears to
  succeed and the application then behaves as though nobody signed in, which is a diagnosis a developer
  pays for twice: once locally, and once in the belief that the port is broken.
- **HTTPS redirection has no destination.** Microsoft's HTTPS-enforcement documentation states that a
  port must be available for the middleware to redirect to, and that when none can be determined
  **redirection does not occur and the middleware logs `Failed to determine the https port for
  redirect`**. The local pipeline is then missing a stage the deployed pipeline has, which is the class of
  difference that turns up first in production.

**The declaration, in a file the map already contains.** `appsettings.Development.json` (§12.2) carries a
`Kestrel:Endpoints` section — the section Kestrel loads endpoint configuration from, per the same
documentation — declaring two endpoints and no certificate:

```json
{
  "Kestrel": {
    "Endpoints": {
      "Http":  { "Url": "http://localhost:5000" },
      "Https": { "Url": "https://localhost:7143" }
    }
  }
}
```

Four properties of those eight lines are decisions rather than boilerplate:

- **Both endpoints are declared, not just the HTTPS one.** The documentation states that endpoints
  defined in configuration **replace** the top-level `Urls` value rather than adding to it, so declaring
  only HTTPS would remove the insecure port — and the insecure port is what the redirect starts from. A
  developer needs both for the same reason an edge deployment does.
- **`https://localhost:7143` is exact, and its exactness is the point.** The port sits in the 7000–7300
  band the SDK itself allocates development HTTPS ports from, and it collides with nothing else this plan
  names: not `5000`, not the container listening port `8080`
  ([06](06-azure-hosting-recommendations.md) §4.4.1), not `80` and not `443`. A fixed, committed value is
  what lets a bookmark, a trusted certificate and an onboarding instruction all agree; a randomly
  allocated one is exactly what the removed launch-profile file used to hold and nobody could cite.
- **No `Certificate` node, deliberately.** The documentation states that an HTTPS endpoint that specifies
  no certificate falls back to `Certificates:Default` and then to the **development certificate**, and
  that where neither exists **the server throws and fails to start**. That failure is the wanted
  behaviour: it is loud, it happens at startup rather than at first sign-in, and its remedy is the
  prerequisite below. The alternative — a certificate path and password in a committed file — is the one
  thing [05](05-aspnet-core-migration-approach.md) §3.3 forbids that file to contain.
- **The HTTPS port needs no separate redirection setting.** Kestrel is the edge locally, with exactly one
  HTTPS address, which is the case the enforcement documentation describes as discoverable by the
  middleware from the server's own addresses. No `https_port` key and no
  `HttpsRedirectionOptions.HttpsPort` value is therefore specified here; behind a reverse proxy that
  discovery does not apply, and the deployed case is 06's rather than this document's.

**The certificate is a documented developer prerequisite, not a repository artifact.** `dotnet dev-certs
https --trust` is shipped with the SDK; the documentation records that `--trust` trusts the certificate on
the local machine and that without it the certificate is created in the store but **not** added to a
trusted list, which is the difference between a working local HTTPS endpoint and a browser that refuses
it. It is run once per machine, it writes nothing into the repository, and it belongs in the onboarding
steps beside `dotnet tool restore` (§6.3). **No certificate, key, password or thumbprint is committed**, so
this adds no row to §12.2 and no secret to any file.

**These keys are Development-only, and no deployed environment reads them.** `appsettings.Development.json`
is loaded only when the resolved environment is `Development`, and §13.3 requires every tooling,
migration, seeding and provisioning invocation to set both environment variables to its own expected
environment — a deployed environment's name in every case, never `Development` — precisely so that this
file is not loaded by any of them. The deployed platforms
supply their own binding — the container listening-port contract is
[06](06-azure-hosting-recommendations.md) §4.4.1's, and the App Service configuration is 06's likewise —
so nothing here reaches, weakens or competes with the deployed transport posture. The **key's place in the
configuration schema** — that the Development file is committed, carries no secret, and is reviewed like
any other source — is [05](05-aspnet-core-migration-approach.md) §3.3's; what this section owns is the
artifact, the exact values and the certificate mechanism, because all three are consequences of this
document's decision to drop the launch-profile file.

---

### 12.9 The operator console references the web application, and is published separately

The row in §12.2 for `tools/provision-admin` carries a `ProjectReference` to
`src/MvcMusicStore/MvcMusicStore.csproj`, and that project is excluded from the web application's publish
output. Those two facts belong together, because the first is what makes the console correct and the second
is what keeps it out of the deployed application. Both are project-structural, which is why they are stated
here rather than in a sibling document.

**Why the reference is required rather than convenient.** The console is not a script that talks to a
database; it is a program that has to agree with the application about types the application owns, and it
carries every operator verb under one host (§12.4), so it needs the union of what those verbs touch.

| Verb group | What it must compile against | What it would otherwise have to duplicate |
| --- | --- | --- |
| `admin` and `seed` | `ApplicationUser`, both `DbContext` types, and the registration seam it resolves `UserManager<ApplicationUser>` and `RoleManager` from | The Identity model and the container registrations. A console that hand-rolled either would hash a password under a configuration the application does not have — and direct SQL cannot produce a valid Identity password hash at all |
| The eight data verbs — `extract-schema`, `diff-schema`, `load-catalog`, `load-identity`, `reconcile`, `seal-manifest`, `accept-run`, `close-run` | **Both** `DbContext` types, `ApplicationUser`, **both** migration sets, and the registration seam | The entity model *and* the migration history. `diff-schema` generates the DDL the migration sets would apply, so a duplicated model would compare the target schema against a copy of the model rather than the model the application ships |

**That table is the *completed* console's compile surface, not a per-verb minimum, and one verb is authored
before the reference exists.** §6.4 places the project's creation at
[03](03-modernization-roadmap.md)'s **W4, gate 4a**, where the referenced project does not exist yet, and
places the `ProjectReference` itself at **W7** — the first gate at which the referenced project compiles
and carries the seam below, and the common predecessor of every later gate whose verbs need it. The one
verb authored between those two gates, W3's `extract-schema`, is the one that needs neither: its catalog
interrogation reads the **source**
store and writes the schema record and the tool's own run rows
([05 §5.6](05-aspnet-core-migration-approach.md)), so it compiles against no type the web project owns.
Every verb authored after W7 is written against the reference, which is why the row above states the union
those verbs need rather than a floor each one reaches on its own. The staging changes nothing about the
completed project this section describes: one console, one reference, one publish output.

**"The composition root" is not something a reference gives you, so the seam is named rather than implied.**
This is the part most easily left as a gap, because the reference makes every *type* available and it is easy
to assume the registrations come with them. They do not. The web project's composition root is written with
top-level statements, and the one public type that arrangement exposes —
`public partial class Program { }` (§12.2) — exists **solely** so `WebApplicationFactory` can locate an
entry point. It declares no members, exposes no `IServiceCollection` and offers a console host nothing to
call: a console that referenced the web project and stopped there would still have to write its own
registrations, which is precisely the duplication the reference was taken to avoid.

The seam is therefore an artifact with a name, mapped in §12.2 and owned by the web project:

| Element | Value |
| --- | --- |
| Owning project and file | `src/MvcMusicStore/` — `ApplicationServices.cs` |
| Type | `public static class ApplicationServices` |
| Method | `public static IServiceCollection AddMvcMusicStoreServices(this IServiceCollection services, IConfiguration configuration)` |
| Registers | Both `DbContext` types on the SQL Server provider; the Identity core, store and manager services over `ApplicationUser`, including the role manager, the password hasher and the Identity options; and the options objects the console reads, bound from the supplied configuration |
| Called by | **`Program.cs` itself**, and the operator console's host builder — nothing else, and nothing bypasses it |

**One registration path, which is the whole point.** Because the web application composes itself *through*
this method rather than beside it, a console that calls it holds the same context configuration, the same
Identity options and the same password hasher as the running application. There is no second place where a
registration could be added and no way for the console's view of the application to be a subset of the real
one that still compiles. What each verb group then does is ordinary hosting:

| Verb group | What it does with the seam |
| --- | --- |
| `admin` and `seed` | The host calls `AddMvcMusicStoreServices` with the console's own configuration and resolves **`UserManager<ApplicationUser>` and the role manager** from the resulting container — which is what makes the password hash `admin` writes a hash the application will accept. [05 §10.2](05-aspnet-core-migration-approach.md) owns the console's five required properties |
| The eight data verbs | The same host resolves **both `DbContext` types and both migration sets**, which is what `diff-schema` compares the live schema against and what the load verbs write through. [05 §5.7](05-aspnet-core-migration-approach.md) owns those verbs' behaviour and [05 §10.2](05-aspnet-core-migration-approach.md) the one exit-code allocation they share |

**The pipeline concerns are deliberately outside the seam**, and the exclusion is as load-bearing as the
inclusion: MVC and Razor, the cookie authentication schemes, session, anti-forgery, data-protection key
persistence, health checks and the middleware order stay in `Program.cs`. A console host has no request
pipeline to attach them to, so putting them in the shared method would make the console configure behaviour
it never exercises — and would make its startup fail on a misconfiguration in a subsystem it does not
use.

**The failure mode a duplicated type produces is drift, and drift here is silent.** A console holding its
own copy of the entity model keeps building and keeps exiting `0` after the application's model changes;
what it writes is simply no longer what the application reads. The project reference makes that class of
divergence a **compile error in the console** at the moment the application changes, which is the only
point at which it is cheap to find. The direction is one-way and stays that way: the console references the
web application, the web application references no console, and the reference graph therefore has no cycle.

**The same argument applies to the registrations, and it is the half a reference alone does not cover.**
Duplicated *types* fail to compile; duplicated *registrations* compile perfectly and drift in silence — a
console that registered its own `DbContext` with its own options keeps working after the application changes
a connection-string key, a provider option or a password-hasher setting, and writes rows or hashes the
application then reads under different assumptions. Nothing fails and nothing is logged. The seam is what
removes that class of divergence: there is one registration method, the application itself calls it, and a
change made there reaches every caller at once. A console that stopped calling it would not silently diverge
either — it would have nothing in its container and fail at resolution, loudly, on its first run. **The
one-way direction holds for the seam too**: the console calls into the web project's registration method,
the web project calls nothing of its, and it references no console.

**No shared library is introduced, and that is a decision rather than an omission.** Extracting the
contexts and the model into a common project would work, and it buys nothing here: there is one owner of
those types, one repository, no consumer outside it, and no second application to share them with — which
is the same reason §2.1 records no `netstandard` shim anywhere in the target. A shared library would add a
project, a lockfile and a versioning question in order to solve a problem the project reference already
solves.

**Why the exclusion from the web publish follows automatically, and what the pipeline must therefore do.**
The reference points from the console **to** the web application, and publish output follows references in
that direction only: `dotnet publish` of the web project has no reason to emit an assembly belonging to a
project that merely consumes it, and the web project references no console. So the console is **not** in the
web application's publish output, which is the separation [05](05-aspnet-core-migration-approach.md) and
[06](06-azure-hosting-recommendations.md) both require — a release-time console tool holding DDL rights
must not ship inside the running web application. The consequence for the build is the part this document
owns: **the console project is built and published on its own**, and its output is staged where the release
step can invoke it. A pipeline that publishes only the web project produces no tool to run, and the
deployment-time migration step [06 §6.2](06-azure-hosting-recommendations.md) places in the release path
then has nothing to invoke.

**One invocation form, used by both callers.** The console is invoked identically by the test suite and by
the release pipeline, and `provision-admin` is the published operator tool
[05 §10.2](05-aspnet-core-migration-approach.md) defines once — that is, `dotnet` against the one published
assembly:

```bash
# The one form every caller uses. <verb> is one of the ten verbs; switches are the tool's own.
provision-admin <verb> [switches]
```

That literal resolves to exactly one published assembly — `dotnet` against the `ProvisionAdmin.dll` in the
console's own publish output, which §13.4 promotes as `provision-admin.zip` — and **no document writes a
second spelling of it**, because a runbook form that differs from the pipeline's is how a switch comes to be
proved in one and not the other. Three properties of the form are requirements, not style:

- **Tests and the release invoke the *same* published artifact**, which is what makes "the version the
  tests exercised is the version production runs" checkable rather than hoped for.
  [03](03-modernization-roadmap.md)'s W7 exit criterion on artifact identity is stated against exactly
  this.
- **Secrets arrive through named environment variables scoped to the single invocation, never as
  command-line arguments** — an argument is visible in process listings and is recorded in shell and
  pipeline history. The channel and the tool's own five required properties are
  [05 §10.2](05-aspnet-core-migration-approach.md)'s; the release-side delivery is
  [06](06-azure-hosting-recommendations.md)'s.
- **The caller captures the exit code, and a non-zero code fails the step it belongs to** — the test case,
  the pipeline stage or the release step. There is **one exit-code allocation, tool-wide across all ten
  verbs, and it has five codes**; it is owned by [05 §10.2](05-aspnet-core-migration-approach.md) and is
  not restated here, and no verb defines a code outside it. What this property requires either way is that
  the code be read: a caller that discards it converts a refusal into an apparent success, which is the one
  outcome those refusals exist to prevent.

The test-side invocation — which cases drive which verb, and what they assert about its verdict — is
[05 §12.9](05-aspnet-core-migration-approach.md)'s; the release-side step and the principal it runs as are
[06 §6.2](06-azure-hosting-recommendations.md)'s. Neither is restated here.

---

## 13. Four warnings about migrations and the artifacts this strategy ships

The first two exist because the dependency and project-format transition specified above says nothing
about the *database*, and a reader could reasonably assume a schema baseline is available somewhere in the
repository. It is not, and the two ways of assuming otherwise both fail. The third is the same class of
hazard one step later: this document pins the tool that generates migrations (§6.3), so it owes the
**environment that tool runs in**, because a migration generated against the wrong configuration is as
untrustworthy as one generated from the wrong model. The fourth is the same hazard one step later again:
pinning the tool and its environment fixes *how* an artifact is produced, and says nothing about whether
the artifact that ships is the one the tests exercised — so §13.4 fixes the **order** of build, test and
generation, and the exact per-context commands, because "tested Release bytes are promoted" is a property
a sequence has or does not have, never a sentence a document can assert.

### 13.1 Neither `MvcMusicStore-Create.sql` copy may be treated as a schema baseline

Three `.sql` files are tracked in the entire repository, and **none of them is MVC 5's** — the migration
source ships no schema script at all (`git ls-files 'src/MVC5/*.sql'` → `0`, appendix A.5). Two of the
three are MVC 4's, at `src/MVC4/MvcMusicStore-Create.sql` and
`src/MVC4/MvcMusicStore/MvcMusicStore-Create.sql`: byte-identical duplicates, and both begin with a
hard-coded developer path to an attached MDF [src/MVC4/MvcMusicStore-Create.sql:1],
[src/MVC4/MvcMusicStore/MvcMusicStore-Create.sql:1], so neither is runnable as written.
[12](12-migration-blockers.md) F-12-22 owns the detail and states explicitly that this document must not
use either script as a baseline. [08](08-technical-debt-register.md) F-08-12 and F-08-24 own the debt.

The third is a tutorial asset under `src/MVC3/MvcMusicStore-Assets/Data/`, belonging to the edition that
is not the migration source and whose provider is retired (§8.5).

**The consequence for this strategy:** the target's `PackageReference` set and project format can be
specified in full from the repository, as sections 5 through 9 do — but the target's *schema* cannot. That
is a genuine boundary of this document, not an omission from it.

### 13.2 An EF Core initial migration cannot be trusted to match the existing database

The tempting shortcut is to point `dotnet ef migrations add` at the ported model and treat the result as
the schema. It is not safe, for a reason that has nothing to do with tooling quality: a migration
generated from the model reflects **the model plus the new provider's conventions**, and it can therefore
differ from the real EF 6 schema in column types, precision and length, nullability, identity and key
definitions, delete rules, defaults and indexes — silently, and against a database holding real orders and
personal data.

There is nothing in the repository to diff it against. The migration source's schema exists **only** inside
the committed `.mdf` files and in the shape EF 6 infers from the model classes at runtime, and the same gap
applies to the credential store, where the Identity schema is not knowable from the repository at all
([12](12-migration-blockers.md) F-12-21, F-12-22).

**The precondition this strategy therefore records:** authoritative schema extraction from the committed
database, followed by a **generated-schema diff that must pass before any data is loaded**. The migration
design is [05](05-aspnet-core-migration-approach.md)'s and the sequencing — extraction before, not after,
the initial migration — is [03](03-modernization-roadmap.md)'s. What is stated *here*, because it is a
property of the inputs this document was written from, is that **no trustworthy schema baseline exists in
the repository**, so no part of the strategy above may be read as supplying one.

### 13.3 A migration artifact is generated under a pinned design-time environment, never a developer default

**Ownership first, because this is a seam where two deliverables could otherwise answer one question two
ways.** *The environment a migration artifact is **generated** under is owned by this document, here, and
is not any other deliverable's to set, restate or override.* It is a property of the **build**, not of any
target: one constant, **`DESIGN_TIME_ENVIRONMENT`**, fixed by the build job, **chosen for what it makes the
host resolve rather than taken from any deployment's name** — the table below states the value and that
reasoning — exported into **both** `ASPNETCORE_ENVIRONMENT` and `DOTNET_ENVIRONMENT` and asserted before
the first `dotnet ef` of §13.4's sequence runs. *The environment an artifact is **executed** under is owned
by [06](06-azure-hosting-recommendations.md)* — the deployment's own `EXPECTED_ENVIRONMENT`, declared per
pipeline environment and supplied at release time ([06](06-azure-hosting-recommendations.md) §12.6.1,
§12.6.2). **The artifact is the boundary: generation takes one build-time constant, execution takes a
per-deployment declared value, and there is nothing in between.**

Three consequences a consumer can act on without reading the mechanics below, stated because this is the
part a consumer is most likely to get wrong in the direction that produces a second artifact set:

- **A per-target expected value never reaches a generation invocation.** Both `dotnet ef migrations bundle`
  invocations and both `dotnet ef migrations script --idempotent` invocations are **build** steps in §13.4's
  single sequence: they run **once per build**, never once per target, so a variable *whose value is
  whichever deployment is being released* has nothing to say about them — even where that variable's value
  and the build constant happen to coincide for one deployment. Where a consumer's release-time inventory lists those
  four generation commands alongside the commands that *execute* an artifact — as
  [06](06-azure-hosting-recommendations.md) §12.6.1's table does, its first two rows being the generation
  rows and marked `Build` — **the constant fixed here governs those two rows** and the per-deployment value
  governs the rest. A consumer that needs to state the generation environment cites this subsection rather
  than naming a value of its own.
- **Per-environment generation is prohibited, not merely unnecessary.** One set of six payloads is produced and
  promoted to production and to every non-production deployment (§13.4). A second generation run under a
  second environment name would produce a second set of bytes with no checksum, no promotion path and no
  suite behind it, which is the outcome §13.4 exists to make impossible.
- **None of this weakens the dual pin.** Both variables are set, both carry the same value, and that value
  is compared against the build's own constant before any work happens. Setting one is not setting the
  environment — the "**Setting one variable is not setting the environment**" bullet below gives the
  reason, which is that the two hosts in the target read the two variables by different routes — and the
  comparison is what turns "pinned" into something an invocation can actually fail on.

§6.3 pins `dotnet-ef`, and §7.2 pins the Design package the command loads. That is the *tool*. The tool
alone is not enough, because **`dotnet ef` builds the application's host to find its contexts**, and a
host reads configuration — so the artifact it produces is a function of the environment the invocation ran
in. The target has an `appsettings.Development.json` (§12.2), and **two separately documented behaviours
decide which configuration a design-time invocation binds. They do not agree with each other, and that
disagreement is the whole reason this section exists:**

- **A host with neither variable set resolves `Production`.** Microsoft's ASP.NET Core environments
  documentation states that the runtime environment is read from `DOTNET_ENVIRONMENT` and
  `ASPNETCORE_ENVIRONMENT`, and that **when neither is set the default environment is `Production`**.
  `Development` is a *selection*, not a default: it is normally chosen by a launch profile or by an IDE's
  start command, and the target has **no launch-profile file at all** (§12.7), so nothing in the mapped
  repository selects it.
- **EF Core's design-time tooling does not inherit that default.** The EF Core documentation on applying
  migrations states that the design-time tooling **uses the `Development` environment when neither
  `ASPNETCORE_ENVIRONMENT` nor `DOTNET_ENVIRONMENT` is set**, and instructs that the environment be set
  explicitly both when a deployment artifact is generated and when a bundle is executed. So the one path
  this section governs is exactly the path whose *unpinned* behaviour is `Development` — the opposite of
  the host default above.

Left unstated, the outcome is that a migration can be generated, scripted or applied against whatever
configuration the machine happens to be carrying, and where the machine carries nothing, against
`Development` by the tooling's own documented default — and **nothing in the artifact records which one it
was.** A reader who knows only the host default would conclude the unpinned case is safe. It is not, and
that is precisely the misreading this section forecloses.

Four requirements close that, and they are requirements on the invocation rather than on any new file:

- **No design-time factory, and no new file.** An `IDesignTimeDbContextFactory` implementation is the
  reflex answer and it is rejected here for two reasons. It would be an artifact absent from the map of
  §12.2, which §12.7 forbids this document from inventing; and it would be a **second composition** — a
  context constructed by the factory rather than by the application's own registrations, which is exactly
  the drift that makes a generated migration untrustworthy. Determinism comes instead from EF Core's
  **host-builder discovery off the mapped `src/MvcMusicStore/Program.cs`**: the tooling resolves the
  application's service provider through the host builder **without starting the server**, so the contexts
  the tooling sees are the contexts `Program.cs` registers. One composition, not two — the same property
  §12.9's registration seam buys for the operator tool.
- **Both environment variables are set, they carry the same value, and the resolved environment is then
  validated against the value that invocation was given.** The rule is deliberately **not** "always
  `Production`", and it is deliberately not *one* rule either, because the two kinds of invocation take
  their value from different places and confusing them is the contradiction §13.4 exists to prevent. A
  **generation** step belongs to the build, and its value is a **constant of that build**: the artifacts
  are environment-agnostic, one set is produced and promoted to every environment, and the pin exists so
  that design-time composition is deterministic rather than picking up whatever the agent or a developer
  shell happens to carry. An **execution** step belongs to a deployment, and its value is **that
  deployment's own declared expectation**, supplied at release time and never baked into a byte. Nor could
  a blanket `Production` cover both: non-production seeding can never legitimately resolve there, so the
  blanket form would contradict a row of the table below and end up ignored rather than followed — a rule
  an invocation cannot obey is a rule that stops being applied anywhere. What every invocation *is* held to
  is three conditions checked **before it does any work** — `ASPNETCORE_ENVIRONMENT` and
  `DOTNET_ENVIRONMENT` are **both set**, they hold the **same value**, and that value equals the
  invocation's own expected value: the build's **`DESIGN_TIME_ENVIRONMENT`** where an artifact is generated
  (§13.4 declares it and asserts it once), and the deployment's **`EXPECTED_ENVIRONMENT`** where one is
  executed, declared once per pipeline environment and read rather than written into a step
  ([06](06-azure-hosting-recommendations.md) §12.6.2 owns that variable and its value set). Absence,
  mismatch against the expected value, or the two variables disagreeing each fail the invocation with
  nothing done.

  **Which value, per invocation, because that is where a blanket rule goes wrong:**

  | Invocation | Where its expected value comes from | The value, and why |
  | --- | --- | --- |
  | Migration generation — both bundle builds and both review scripts, **once per build and not once per environment** (§13.4) | The build: **`DESIGN_TIME_ENVIRONMENT`**, a constant of the build job | **`Production`**, chosen as the build's design-time constant rather than as any target's name: it is the value the host resolves when nothing is set, and it is *not* `Development` — EF Core's design-time default, and the one value that would pull a developer's user secrets and the Development overlay into a generated artifact. Nothing environment-specific enters the bytes, so the four migration artifacts are promoted unchanged to every environment |
  | Executing the bundles, creating the session cache table, and the two data phases | The deployment being released: **`EXPECTED_ENVIRONMENT`** ([06](06-azure-hosting-recommendations.md) §12.6.2) | That deployment's own name — `Production` for the production site, `Staging` for a separate non-production deployment. The comparison is against a declared variable rather than a literal, so a stage copied between environments fails instead of silently acting on the wrong one; the release-side list is [06](06-azure-hosting-recommendations.md) §12.6.1's |
  | **Non-production seeding** | The non-production deployment being seeded | **Never `Production`** — that deployment's own name. Seeding writes the 826-line sample catalog, which has no business in a production store even once. Its **three fail-closed checks** — environment not `Production`, an explicit enable flag, an allow-listed database name — are [05](05-aspnet-core-migration-approach.md) §5.4's, unchanged by this comparison and **not replaced by it**: both layers must pass |
  | Administrator provisioning | The deployment being provisioned | That environment's own name. It runs once per environment at bootstrap ([05](05-aspnet-core-migration-approach.md) §10.2 owns the command) |

  This is a correctness property, not a convenience, and each part of it earns its place:

  - **Setting one variable is not setting the environment.** Both are read, and the two hosts in the
    target read them by different routes: the web application is a `WebApplication`, while
    `tools/provision-admin` is a **generic host**, whose host configuration Microsoft's generic-host
    documentation describes as coming from `DOTNET_`-prefixed environment variables. An invocation that
    pins only `ASPNETCORE_ENVIRONMENT` therefore leaves the tool's own governing variable untouched, and
    an invocation that pins only `DOTNET_ENVIRONMENT` leaves the web host's. Pinning both is what makes
    one statement cover both hosts. The provisioning tool's host and command semantics are
    [05](05-aspnet-core-migration-approach.md) §10.2's; the *pin* is here, because the tool is one of the
    invocations this document's tool manifest makes possible (§6.3).
  - **The ambient value is not knowable from the repository, whatever the documented default is.** A
    build or release runner may carry either variable from a previous step, an agent image may set one,
    and a developer shell routinely does. The repository cannot tell a reader which — there is no
    launch-profile file (§12.7) and no other place a value could be read from — so the invocation must
    establish the environment rather than inherit it. That is true even though the host default is
    `Production`: a default only applies when nothing is set, and "nothing is set" is the one state
    nobody can verify from here.
  - **A disagreement is failed, not resolved by precedence.** Microsoft's own environments documentation
    is not self-consistent on which variable wins under `WebApplication` — its summary statement and its
    per-API bullets give opposite precedences — so a design that relied on precedence would rest on an
    ambiguity in the source it cites. Comparing the two values and failing on a mismatch removes the
    dependence entirely, and it costs one comparison. The failure names the two variables and the
    resolved environment; it echoes no configuration value, because the values in play include a
    connection string.

  **Where the check lives, because "the invocation validates" has to name something.** Two places, and
  deliberately not a third. The **build job** asserts it once, in the step that exports the two variables
  ahead of every generation in §13.4's sequence: both present, equal to each other, and equal to that
  job's `DESIGN_TIME_ENVIRONMENT`, or the job stops before the first `dotnet ef` runs. The comparison is
  the **same shape** as the one validator step [06](06-azure-hosting-recommendations.md) §12.6.1 states
  for the release side, and the two differ only in where the expected value comes from — a build constant
  there, a deployment's declared variable here — which is the division §13.4 states in full. The
  **seeding and
  provisioning commands** assert it again inside themselves, on the resolved
  `IHostEnvironment.EnvironmentName`, because they are also run by hand outside any job and a guard that
  exists only in a pipeline is absent exactly when it matters; the shape of those commands' own guards is
  [05](05-aspnet-core-migration-approach.md) §10.2's, and what this document requires is that they compare
  the resolved environment against the expected one and refuse rather than assume. The **web application
  asserts nothing of the kind**: it must run in Development legitimately (§12.8), so an
  expected-environment guard inside `Program.cs` would break the one host that needs an environment no
  release-time invocation may carry — and putting it there would also make the check a property of the
  application rather than of the invocation, which is the confusion this bullet exists to prevent.

  **No expected value in the table above is `Development`**: one of them is the build's own design-time
  constant and the rest are deployment names — `Production`, or a non-production deployment's own name,
  which is the value set [06](06-azure-hosting-recommendations.md) §12.6.2 declares. That has a second
  effect worth naming: in **none** of these invocations does the Secret Manager source contribute anything,
  because it is registered only when the resolved environment is `Development` — which the EF Core
  documentation gives as one of its own reasons for setting the environment before generating a bundle — so
  a developer's local secret cannot become the source of a generated migration's connection. That is a
  statement about *these* invocations and not about the source being inert: it is a real, active
  configuration source on a developer machine, and §5.7 states the project property without which it would
  silently contribute nothing there either.
- **Configuration reaches a host through the environment, never the command line; the two release-time
  commands take their target connection as an argument because an argument is their only channel.** Those
  are two statements rather than one prohibition, because a single blanket prohibition would be false of
  what §6.3's pinned tools actually accept — and a rule a command cannot obey is a rule that stops being
  applied to the one thing it should bind:

  - **The application host never takes configuration on the command line.** The command-line provider is
    registered **last** and therefore outranks every other source, including the platform's own
    settings, so a value passed there is an override that does not appear in the deployed configuration a
    reviewer reads — the provider order and that rule are
    [05](05-aspnet-core-migration-approach.md) §3.3's. A design-time invocation therefore takes its
    connection the way the platform supplies one: the environment variable
    **`ConnectionStrings__DefaultConnection`**, which is the double-underscore spelling of the
    `ConnectionStrings:DefaultConnection` key already in that typed contract and not a second key. The
    host's default configuration order puts the environment-variables provider **after**
    `appsettings.json` and `appsettings.{Environment}.json`, so the variable outranks both and the value
    cannot be silently inherited from a file that happens to sit in the working directory;
    `appsettings.Development.json` is **never** the source for a migration operation, and because no
    expected value above resolves to `Development` it is not even loaded.
  - **A bundle and the cache tool take a connection argument, because neither has another input.** The
    migration bundle accepts `--connection <CONNECTION>`, documented as defaulting "to the one specified
    in `AddDbContext` or `OnConfiguring`" — and that fallback is precisely what a release cannot afford,
    since a settings file left beside the executable would then decide which database receives DDL.
    `dotnet sql-cache create` takes the connection as its **first positional argument**, ahead of the
    schema and table names, and accepts configuration by no other route at all. So the prohibition binds
    the *host*, not these two, and four properties make the argument safe rather than merely tolerated:
    the value comes from the **`MIGRATION_CONNECTION`** pipeline variable
    ([06](06-azure-hosting-recommendations.md) §12.6.2), it **carries no credential** because it
    authenticates as a managed identity ([06](06-azure-hosting-recommendations.md) §6.1.1), it is
    **never echoed** into a log, a command trace or a failure message, and the server and database it
    names are **asserted against the expected values before it is used**
    ([06](06-azure-hosting-recommendations.md) §6.2.2). **The generation half is still the environment
    variable** — `dotnet ef` builds a host, so it reads configuration like one — and the argument appears
    only when the produced artifact is *executed*: [06](06-azure-hosting-recommendations.md) §6.2.1 owns
    the bundle invocation, and its §6.4 owns the cache-table command.
- **`dotnet tool restore` precedes any `dotnet ef` invocation**, so the command that runs is the manifest's
  pinned `dotnet-ef` `8.0.30` (§6.3) rather than whatever a machine-global install carries; and **§3's
  `8.0.400` band governs the generating SDK**, because `dotnet ef` builds the project to find the host, so
  a migration generated by an off-band SDK was generated by a different toolchain than the one that builds
  the application.

**The release-time halves of this contract are [06](06-azure-hosting-recommendations.md)'s, and they are
not restated here.** §6.2 there owns the deployment-time migration step, the artifact it runs and the
principal it runs under, and with them the environment pin at the moment the artifact is *executed* and
the assertion that the server and database being changed are the intended ones before any DDL is issued.
This document's half stops where the artifact is produced: generated **once**, under the build's own pinned
design-time environment, from one composition, by a pinned tool on a pinned band — in the order §13.4
fixes. Generated once is the operative word: there is no per-environment generation anywhere in this
contract, and the environment an artifact eventually runs under is supplied to it at execution rather than
chosen when it was built.

**The split is an ownership boundary and not a division of labour, which is why it is stated in both
directions.** The generation-time environment is **this document's to fix and nobody else's to name**, and
the execution-time environment is **06's to fix and not this document's to name**: this subsection
therefore states no value for any deployment, and a consumer states no value for a generation step. The
one place the two meet is the artifact itself, and it carries neither value — no environment name, no
expected value and no connection is inside any of the six payloads §13.4 names.

### 13.4 The promoted bytes are the tested bytes — the order, and the per-context commands

§13.3 fixes the environment an artifact is generated *under*. It does not fix **which build** the artifact
comes from, and that is a separate hazard with the same shape: a Release artifact produced by a build other
than the one whose suite ran — or produced within it and then regenerated before promotion — was validated
by nothing, however carefully its environment was pinned. "Tested Release bytes are promoted" is a property
an ordering has or does not have — it is not a sentence a document can assert — so the ordering is stated
here as commands, with every option that carries the property called out and sourced, and with the one
place the property rests on file identity rather than on compile identity stated as such rather than
glossed (the bundle subsection below).

**This section is the single canonical command and artifact contract for the target, and a consumer cites
it rather than restating it.** Every command below, every option on it and every artifact name it produces
is decided here and nowhere else; [05](05-aspnet-core-migration-approach.md) and
[06](06-azure-hosting-recommendations.md) consume them by reference, exactly as §1.2 requires of a value
this document owns. The reason is a failure mode rather than a preference: a command copied into a second
document drifts on precisely the details that make it work — an omitted `--context` is a command that does
not run at all, an omitted `--no-build` silently validates a compile that ships nowhere, and a renamed
artifact makes two documents describe two files where the release has one. Six consequences follow, and
they are stated as rules because each one is a thing a consumer might otherwise do:

- **The commands live here.** A document that needs one names this section; it does not print its own
  copy. Where a consumer owns a *release-side* property of an artifact — the stage it runs in, the
  principal that runs it, the gate its exit code feeds — it states that property against the names below
  rather than reproducing the invocation that produced them.
- **The build fixes *what* ships; the deployment supplies *where it is running and what it should
  expect*.** Everything in this section runs **once**, in one build, under the build's own
  `DESIGN_TIME_ENVIRONMENT` — a constant of that build and not the name of a target (§13.3). Every
  artifact it produces is therefore **environment-agnostic**: no environment name, no connection, no
  expected value and no secret is compiled into any of the six payloads, and **one set is promoted to
  production and to every non-production deployment**. What a deployment adds at the moment of use is its
  own: both environment variables and `EXPECTED_ENVIRONMENT`, the target-assertion values and the migration
  connection are pipeline variables declared per environment
  ([06](06-azure-hosting-recommendations.md) §12.6.1, §12.6.2), and the **typed expected values the
  application itself validates at startup** — the data-protection discriminator among them — are
  [05](05-aspnet-core-migration-approach.md) §3.3's, read from configuration at run time rather than from
  the bytes. Two consequences a reader can check against the sequence below: **no step generates a second
  artifact set for a second environment**, and **no step writes an expected value into an artifact** — an
  expectation compiled in at build time could not be re-declared by the deployment that has to satisfy it,
  which is exactly the contradiction this rule forecloses.
- **The six artifact names are the only names these artifacts have**: `efbundle-catalog`,
  `efbundle-identity`, `migrations-catalog.sql`, `migrations-identity.sql`, `site.zip` and
  `provision-admin.zip`, spelled as the sequence spells them, each with a `.sha256` beside it. There is no
  alternative scheme for the review scripts and **no per-environment variant of anything**. A `review-*`,
  `latest-*` or `*-production` name appearing in any deliverable is a **second artifact** to a reader and
  to a pipeline, and the consequence is not cosmetic: one of the two then has no checksum, no promotion
  path and nothing verifying it. Under the container option one further artifact exists — the site image —
  and it is the only one not named this way: it carries one repository and a run-identifier tag, and it is
  *addressed* by digest, as the conditional subsection below states.
- **No command may require an `appsettings.Production.json`, because the map contains none.** §12.2 carries
  `appsettings.json` and `appsettings.Development.json` and **nothing else**, and
  [05](05-aspnet-core-migration-approach.md) §3.3 — which owns the configuration file set — states that no
  `Production` overlay is created, the deployed levels living in the base file instead. Production *values*
  arrive from platform configuration ([06](06-azure-hosting-recommendations.md) §8.4). So a step that
  copied a `Production` settings file beside a generated artifact would be copying a file that does not
  exist, and a contract that required one would be unsatisfiable as written.
- **The design-time input is an environment variable, not a settings file.**
  `ConnectionStrings__DefaultConnection` — a member of [05](05-aspnet-core-migration-approach.md) §3.3's
  typed configuration contract, in its double-underscore environment spelling — is what a generation step
  binds its context from, and **no `appsettings*.json` is placed beside a bundle or a script**. That second
  half is deliberate and it is the reason the first half is enough: Microsoft's guidance on applying
  migrations records that a bundle **resolves configuration files from its own execution directory**, so a
  settings file left there is a source that decides a target, and the way to remove that source is to place
  none. The rule, its provider-order reasoning and the separate execution-time argument channel are §13.3's
  third requirement.
- **§12.8 is the only source for the Development Kestrel binding.** The two `Kestrel:Endpoints` URLs live
  in `appsettings.Development.json` and are stated, with their exact values and their certificate
  prerequisite, in §12.8 alone. No command in the sequence below sets a URL, no consumer declares a second
  pair, and none of this touches the deployed listening port, which is
  [06](06-azure-hosting-recommendations.md) §4.4.1's.

**One Release compile behind the three payloads. Then every artifact of that build. Then the suite, against
that build and against those artifacts. Then, and only then, the archives.** The order is not a preference
and it is not symmetrical, so it is stated as three rules with the consumer that forces each — and the
qualifier in the first sentence is load-bearing, because a bundle build is not a no-compile step and the
subsection after the option table states exactly what it is:

- **Everything the suite consumes is produced above it.** Two things, from two consumers.
  [05](05-aspnet-core-migration-approach.md) §12.2 records that in CI the fixture **executes the two bundles
  themselves** rather than applying migrations by an equivalent route, so the bundles and their sidecars must
  already exist when the suite runs. And [05](05-aspnet-core-migration-approach.md) §12.4.1 — its operator
  rows **O1 to O10** — **launches the published operator executable as a process**, asserting on the exit
  code and captured output of the *published* bytes rather than calling a method in-proc, precisely because
  the switch mapping, the content root, the environment block and the absence of a shipped `appsettings.json`
  are properties of a publish output and not of a function. In a clean build those bytes do not exist until
  `dotnet publish tools/provision-admin` has run, so **both publishes sit above the suite**. The site publish
  goes with them: it consumes the same compile, and keeping the two publishes adjacent means there is one
  place where the payloads come into existence rather than two.
- **Nothing is archived from a build whose suite went red.** The two `zip` steps, their sidecars and the
  release record stay **below** the suite. Under the fail-fast rule below, a red suite ends the sequence
  there: the publish *directories* exist, the promoted *files* do not, and there is nothing for a publication
  step to attach.
- **The archived bytes are the tested bytes, because they are the same files.** The archive is built from
  `artifacts/site` and `artifacts/provision-admin` — the very directories the suite launched the operator
  from — and the only thing the archive step changes about them is the modification timestamp the
  deterministic-archive rule requires. Nothing is republished, recompiled or copied in between. The bundles
  have the same property one step earlier: their sidecars are computed **before** the suite reads them, so
  the bytes the suite executes are byte-for-byte the bytes promoted.

```bash
set -euo pipefail   # the fail-fast contract: any non-zero exit, any unset variable and any failure
                    # inside a pipeline stops the build HERE. Nothing below a failure runs, so the
                    # archives and the release record are unreachable after a failed prerequisite,
                    # whatever the provider's own step semantics are (§13.4, the fail-fast rule)

# Every command runs from the repository root. EF's target and startup project are the same
# project, so both options name it explicitly rather than relying on the current folder.
DESIGN_TIME_ENVIRONMENT=Production                        # a constant of THIS BUILD, not a target's name
export ASPNETCORE_ENVIRONMENT="$DESIGN_TIME_ENVIRONMENT" \
       DOTNET_ENVIRONMENT="$DESIGN_TIME_ENVIRONMENT"      # §13.3 — both, equal, asserted before step 1
if [ "$ASPNETCORE_ENVIRONMENT" != "$DESIGN_TIME_ENVIRONMENT" ] \
   || [ "$DOTNET_ENVIRONMENT" != "$DESIGN_TIME_ENVIRONMENT" ]; then
    echo 'design-time environment pin not in force' >&2   # names the variables, echoes no config value
    exit 1
fi
export TZ=UTC                                             # one fixed zone for the whole build: a zip
                                                          # entry stores a LOCAL MS-DOS date and time
                                                          # and carries no zone field, so two agents in
                                                          # two zones would otherwise archive identical
                                                          # bytes into different files (the
                                                          # reproducibility rule below)
EF_PROJ=(--project src/MvcMusicStore --startup-project src/MvcMusicStore)
RUN_ID="${CI_RUN_ID:?required - the run identifier this provider assigns}"   # names this release
# The `:?` expansion above rejects unset and empty and NOTHING ELSE, and this value is not merely
# printed: it becomes a `run=` field in the line-oriented release record below and a container image
# tag in the conditional subsection. 06 6.9.1.2 owns ONE closed grammar for this same pipeline run
# identifier and enforces it at both of its own use sites; this is the third, so the same fragment form
# runs here - see the prose below. A `case` over the WHOLE string rather than a `grep` over its
# lines is what makes a newline, or any other control character, a rejection instead of a match on the
# first line.
case "$RUN_ID" in
  *[!a-z0-9-]*) echo 'FAIL: RUN_ID is not lower-case alphanumerics and hyphens' >&2; exit 1 ;;
  -*|*-)        echo 'FAIL: RUN_ID must begin and end with an alphanumeric' >&2; exit 1 ;;
  *--*)         echo 'FAIL: RUN_ID contains a double hyphen' >&2; exit 1 ;;
esac
[ "${#RUN_ID}" -le 19 ] \
  || { echo 'FAIL: RUN_ID exceeds 19 characters' >&2; exit 1; }
# The rejected value is deliberately NOT echoed in these four messages: a value rejected for carrying
# control characters is exactly the value not to write into a build log.
COMMIT="$(git rev-parse HEAD)"                            # the other half, in full
SOURCE_DATE="$(git log -1 --format=%cI "$COMMIT")"        # one mtime for every archived file

# The build runs inside one pinned container image and is pinned on the utilities it uses. The digest is
# recorded in release.txt, and both it and the utility versions are ASSERTED here rather than printed,
# so a changed image or a changed utility fails the build instead of quietly changing an archive nobody
# compared. The values are fixed when the image is selected at the CI provider gate that 03 owns; this
# sequence fixes only that they exist, are asserted, and that the digest is recorded.
BUILD_IMAGE_DIGEST="${BUILD_IMAGE_DIGEST:?required - the digest of the image this build runs in}"
EXPECT_ZIP="${EXPECT_ZIP:?required - the zip version string this image is pinned to}"
EXPECT_SHA256SUM="${EXPECT_SHA256SUM:?required - the sha256sum version string}"
EXPECT_FIND="${EXPECT_FIND:?required - the find version string}"
EXPECT_GIT="${EXPECT_GIT:?required - the git version string}"

assert_tool() {             # $1 label, $2 expected string, rest: the command that reports a version
    local label="$1" expected="$2"; shift 2
    local observed
    observed="$("$@" 2>&1)" # each command below exits 0; failing to run at all is itself a mismatch
    case "$observed" in
        *"$expected"*) : ;;
        *) printf 'utility pin mismatch: %s does not report %s\n' "$label" "$expected" >&2; exit 1 ;;
    esac
}
assert_tool zip       "$EXPECT_ZIP"       zip -v
assert_tool sha256sum "$EXPECT_SHA256SUM" sha256sum --version
assert_tool find      "$EXPECT_FIND"      find --version
assert_tool git       "$EXPECT_GIT"       git --version
find . -maxdepth 0 -printf '%P\n' > /dev/null   # the GNU -printf extension the archive step needs

dotnet tool restore                       # §6.3 — the pinned dotnet-ef and dotnet-sql-cache
dotnet restore --locked-mode              # §6.4 — a transitive change fails here, not later
dotnet build -c Release --no-restore      # the one compile behind the site payload, the test
                                          # assemblies and the operator utility. It is NOT the only
                                          # compile in this sequence: each bundle step below runs a
                                          # publish of its own generated project (see "What a bundle
                                          # build actually does" after the option table)

# The two bundles are FRAMEWORK-DEPENDENT: neither --self-contained nor
# -r/--target-runtime is passed, so each bundle carries the migrations and
# not a runtime, and the job image that executes it supplies a .NET 8
# runtime (the runner prerequisite below, owned by 06 §6.2.1 and §12.3)
dotnet ef migrations bundle "${EF_PROJ[@]}" --context MusicStoreEntities \
    --configuration Release --no-build \
    -o artifacts/efbundle-catalog
dotnet ef migrations bundle "${EF_PROJ[@]}" --context ApplicationDbContext \
    --configuration Release --no-build \
    -o artifacts/efbundle-identity

dotnet ef migrations script --idempotent "${EF_PROJ[@]}" --context MusicStoreEntities \
    --configuration Release --no-build --output artifacts/migrations-catalog.sql
dotnet ef migrations script --idempotent "${EF_PROJ[@]}" --context ApplicationDbContext \
    --configuration Release --no-build --output artifacts/migrations-identity.sql

( cd artifacts && for F in efbundle-catalog efbundle-identity \
                           migrations-catalog.sql migrations-identity.sql; do
      sha256sum "$F" > "$F.sha256"        # bare names, so `sha256sum -c <file>.sha256` works beside
  done )                                  # the file; computed before the suite reads either bundle

dotnet publish src/MvcMusicStore     -c Release --no-build -o artifacts/site
dotnet publish tools/provision-admin -c Release --no-build -o artifacts/provision-admin

for OUT in site provision-admin; do      # NO DEBUG SYMBOLS IN EITHER PUBLISH ROOT. Section 5.1 sets
    SYMBOLS="$(find "artifacts/$OUT" -name '*.pdb')"   # DebugType=none for Release in both published
    if [ -n "$SYMBOLS" ]; then           # projects; this is what proves it per release. 06 section 9.2's
        printf 'debug symbols in artifacts/%s:\n%s\n' "$OUT" "$SYMBOLS" >&2   # no-source-path-and-no-line
        exit 1                           # promise for the sanitized exception record rests on this
    fi                                   # artifact property, so a symbol file here is a release-blocking
done                                     # privacy regression rather than untidiness

dotnet test -c Release --no-build         # LAST of the build steps, because the suite consumes what
                                          # they produced: it verifies the two bundle sidecars and
                                          # applies the schema by EXECUTING those bundles, and it
                                          # launches the PUBLISHED operator executable as a process
                                          # (05 §12.2, §12.4.1)

for OUT in site provision-admin; do       # one deterministic archive per publish output
    rm -f "artifacts/$OUT.zip"            # zip UPDATES an existing archive; always start from none
    find "artifacts/$OUT" -exec touch -h -d "$SOURCE_DATE" {} +
    ( cd "artifacts/$OUT" && find . -type f -printf '%P\n' | LC_ALL=C sort \
        | zip -X -D -q -@ "../$OUT.zip" )
    ( cd artifacts && sha256sum "$OUT.zip" > "$OUT.zip.sha256" )
done

{ printf 'run=%s\ncommit=%s\nimage=%s\n' "$RUN_ID" "$COMMIT" "$BUILD_IMAGE_DIGEST"
  cat artifacts/*.sha256; } \
    > artifacts/release.txt               # the release record, written ONLY if every step above passed

# The publication precondition. It is the first action of the provider's own publish-artifacts step
# (§12.2's CI row) and it fails closed WITHOUT relying on that step being conditional: after any
# failure above, release.txt does not exist and this check exits non-zero even where a provider was
# configured to publish unconditionally.
[ -f artifacts/release.txt ] || { echo 'no release record: nothing is publishable' >&2; exit 1; }
( cd artifacts && sha256sum -c ./*.sha256 )   # all six sidecars, against the files being published
test "$(grep -cE '^[0-9a-f]{64} ' artifacts/release.txt)" -eq 6   # six sidecar lines, no more, no fewer

# Only now: the files this sequence generated into artifacts/ - the payloads and their sidecars -
# plus release.txt, all attached to THIS run. The Publication row of the manifest table below is
# the one place their number is stated; this comment deliberately does not restate it.
```

**`RUN_ID` is validated and not merely required, because `:?` rejects two values and this sequence puts the
other ones somewhere they matter.** The expansion rejects unset and empty; it accepts a newline, a slash, an
upper-case letter, a leading hyphen and a 200-character string. Two of this sequence's own uses make that
consequential rather than cosmetic: `RUN_ID` becomes the **`run=` field of `release.txt`**, a line-oriented
record whose `run=` value is the rollback locator the promotion contract reads, so an embedded newline
splits one record into two and a reader cannot tell which half is authoritative; and the conditional
subsection below interpolates it into a **container image tag**, where a character outside the tag grammar
fails the build late, at `docker build`, with an error about a tag rather than about an input.

**The grammar is consumed from its owner rather than published here, because two grammars for one
identifier is worse than none.** [06 §6.9.1.2](06-azure-hosting-recommendations.md) states **one closed
grammar** for this same pipeline run identifier — lower-case alphanumerics and hyphens only, beginning and
ending with an alphanumeric, no double hyphen, at most 19 characters — and records that it is *"enforced at
both of its use sites"*, the second of which applies the **fragment** form before interpolating the value
into an image tag. That document reached its fragment form by finding exactly this defect in its own text:
a stage that *"took the same value on `:?` alone and composed"* an image tag from it. **This sequence is the
third use site and had the same hole**, so it runs the same four-clause check immediately after the
binding — a `case` over the whole string plus a length test, which is why a control character is a rejection
rather than a first-line match — and publishes no grammar of its own. The 19-character bound is that
document's and is not re-derived here: it is what keeps 06's composed job name inside its own limit, and the
same character set is a strict subset of a well-formed image tag, so one check satisfies both uses. The
rejected value is not echoed in any of the four messages, for the reason 06 gives: a value rejected for
carrying control characters is the one value not to write into a log.

**The fail-fast rule, stated as prose because it is the property the sequence would otherwise only appear
to have.** A sequence of commands is not a gate. Without `set -euo pipefail` a failed restore, a failed
compile, a failed bundle build, a red suite or a failed publish is simply *followed* by the next line, so
the archives are built from whatever is on disk, the release record is written, the publication step
attaches it, and the step reports success — the exact outcome this section exists to make impossible.
Three parts make it a gate, and none of them assumes anything about the provider:

- **`set -euo pipefail`, at the top.** `-e` stops the sequence at the first non-zero exit; `-u` makes an
  unset variable an error rather than an empty string, so a missing `CI_RUN_ID` cannot silently produce a
  release named after nothing; `-o pipefail` makes a pipeline fail when **any** stage fails, which matters
  because the archive step is a pipeline and `zip` is its last stage — without it a failed `find` or a
  failed `sort` would be masked by a `zip` that succeeded on truncated input.
- **Nothing in the sequence tolerates a non-zero exit, and the absence of any tolerance is deliberate.**
  There is no `|| true`, no `; :` and no ignored status anywhere above, because there is no step here whose
  failure is expected: every command either produces an artifact or establishes a precondition for one. The
  two `||` constructs that do appear are **guards that exit**, not suppressors. Where a later stage does
  have a tolerated non-zero exit — `dotnet sql-cache create` returning 1 for *already present* — that stage
  belongs to the release rather than to the build, and its exit-code contract is
  [06](06-azure-hosting-recommendations.md) §6.4's.
- **Provider independence, which is why the precondition exists at all.** Provider selection is an open
  roadmap gate ([03](03-modernization-roadmap.md)), so this document cannot rely on a provider failing a
  job at the first failed command, on it skipping later steps, or on an artifact-publishing task being
  conditional — some providers run a publish task unconditionally by design. The mechanism is therefore in
  the artifacts rather than in the job definition: **`release.txt` is written last and only if everything
  above succeeded**, and the publication step's first action is the precondition that requires it to exist,
  requires all six sidecars to verify, and requires exactly six sidecar lines in it. A provider that runs
  the publish step anyway publishes nothing, because the precondition exits non-zero before an upload
  begins. The pipeline gate is welcome and subordinate; the record is the guarantee.

**The symbol check is a gate of this sequence, and it is the producer half of a promise another deliverable
makes.** [06](06-azure-hosting-recommendations.md) §9.2's sanitized exception record carries a stack trace
that names methods and carries **no source-file path and no line number**, and it requires that to be a
property of the artifact rather than of the code that writes the record — because a .NET stack frame
resolves to a file and a line only when a matching symbol file is present beside the assembly at run time.
§5.1 sets the property that produces symbol-free output: `<DebugType>none</DebugType>` for `Release`, in
both of the projects this sequence publishes. The loop above is what turns that setting into evidence. It
runs **after both publishes and before `dotnet test`**, so a symbol file fails the build before anything is
tested, archived or recorded, and under the fail-fast rule nothing below it runs. It uses the same `find`
the archive step uses and whose version this sequence already asserts, it names the offending files on
standard error so the failure is actionable, and it covers **both** publish roots because the console
references the web project (§12.9) and would otherwise carry that project's symbols into its own output.
If a decision is ever taken to ship symbols, it is this property and this check that change, and
[06](06-azure-hosting-recommendations.md) §9.2 states what the promise weakens to when they do.

**Why each option is load-bearing.** None of these is decoration, and two of them are the whole finding:

| Option | What the documentation says it does | Why it is required here |
| --- | --- | --- |
| `dotnet test -c Release --no-build` | `--no-build` does not build the test project before running it and implicitly sets `--no-restore`; tests always run from the output directory | Without both the configuration and `--no-build`, the suite silently builds its own assemblies — by default in `Debug` — and validates a compile that ships nowhere. This is the option that makes "the tested bytes" a fact rather than an intention |
| `--context <name>` on every `dotnet ef` invocation | The EF Core CLI reference states the option is **required when a project contains multiple context classes** | The target has two contexts (§12.2, and [05](05-aspnet-core-migration-approach.md) §4.5 owns the split), so **an invocation without `--context` fails.** A plan that writes the command without it has written a command that does not run |
| `--configuration Release` on every `dotnet ef` invocation | The EF Core CLI reference lists it as a common option — the build configuration | The migrations assembly the tool reads must be the Release one. A `Debug` migrations assembly is a different assembly, and a bundle built from it is not the bundle that was tested |
| `--no-build` on every `dotnet ef` invocation | "Don't build the project. Intended to be used when the build is up-to-date" | After the Release compile above the build *is* up to date, which is precisely the documented case, so the flag stops the tool rebuilding the application before it inspects it. **What it does not do is make a bundle build a no-compile step**, and this row deliberately does not claim otherwise: `--no-build` governs the tool's own pre-build of *this* project and is not carried into the publish a bundle build performs on the project it generates. The honest account, its mechanism and the controls that bound it are stated in full after this table. On the two `migrations script` invocations there is nothing further to say — a script is emitted by the tool from the migrations assembly and no second project is involved |
| `-o` / `--output` on every invocation, with a distinct name per context | `migrations bundle` takes `--output`/`-o` for the executable's path; `migrations script` takes `--output`/`-o` for the file, and **writes to standard output when it is omitted** | Both bundle invocations default to the *same* output name, so two bundles built without explicit names overwrite each other and the release applies one set twice — the failure [06](06-azure-hosting-recommendations.md) §6.2.1 also records. And a review script left on standard output is not an artifact anyone can attach to a run |
| `--idempotent` on the script invocations | Generates a script usable against a database at any migration | The script is a **review** artifact, read before a schema change is approved; idempotence is what lets a reviewer read one file rather than reason about which migrations a given database already carries |
| `--project` **and** `--startup-project`, both naming `src/MvcMusicStore` | Each is documented as a relative path to a project folder, and each **defaults to the current folder** | The commands run from the repository root, where the only project-shaped thing is the solution (§5.6). Passing one and not the other leaves the other defaulted to the root, so both are named — and both name the same project, because the web application is its own startup project (§12.3) |
| `dotnet publish` naming `src/MvcMusicStore` rather than being left to resolve the solution | The 7.0.200 SDK stopped accepting `--output`/`-o` together with a *solution* file for `build`, `clean`, `publish`, `store` and `test`, because `OutputPath` has no well-defined solution-level meaning; from the 7.0.201 SDK it is a warning rather than an error, and the outputs of every project land in one directory in an undefined order | Run from the root with no project named, `dotnet publish -o …` resolves the single solution (§5.6) and would publish **both** publishable projects — the web application and `tools/provision-admin` — into `artifacts/site`, which is precisely the confusion between artifacts this section exists to prevent. Naming the project publishes the site payload alone; the operator tool is published by its own invocation naming its own project, which is the single publish output all ten of its verbs share (§12.3), and what those verbs do is [05](05-aspnet-core-migration-approach.md) §5.1.2, §5.4 and §10.2's |
| `--no-build` on **both** `dotnet publish` invocations, and **both above `dotnet test`** | "Doesn't build the project before publishing. It also implicitly sets the `--no-restore` flag" | The flag has the same property as on `dotnet test`, and for the same reason: without it each publish recompiles, and the payload that ships is then a compile the suite never ran against. The **position** is a separate requirement with its own consumer: [05](05-aspnet-core-migration-approach.md) §12.7.1's rows **O1 to O10** start the *published* operator executable as a process, so in a clean build the suite cannot run until `artifacts/provision-admin` exists. The operator tool is published for release consumers as well — the data phases, seeding and administrator provisioning are all verbs of it ([06](06-azure-hosting-recommendations.md) §12.6.1's rows 5 to 7) — and a stage that had to `dotnet run` it from source would be a second compile of exactly the assembly the tests exercised |
| `touch -h -d "$SOURCE_DATE"` over each publish output, with `SOURCE_DATE` from `git log -1 --format=%cI` | `touch`'s `-d`/`--date` parses a date string and `-h`/`--no-dereference` affects a symbolic link rather than its target; git's `%cI` is the committer date in strict ISO 8601 | A zip entry carries an MS-DOS date and time, so an archive of identical content built twice is not identical while the file timestamps differ. One value for every file makes a rebuild of the same commit produce the same archive. The value is the **commit's own date** and deliberately not a fixed epoch: App Service's ZIP deployment copies a file "only if their timestamps don't match what's already deployed", so a constant shared by every release would let a changed file be skipped as unchanged, while a per-commit value is constant within a release and different between releases. The convention is the one `SOURCE_DATE_EPOCH` standardises — the last modification of the source — and a commit date is always after the zip format's 1980 floor |
| `find . -type f -printf '%P\n' \| LC_ALL=C sort`, piped into `zip -@` | `find`'s `%P` is "File's name with the name of the starting-point under which it was found removed"; `zip`'s `-@`/`--names-stdin` takes "the list of input files from standard input. Only one filename per line" | Two properties in one pipe. The entry **order** becomes the sorted order rather than directory order, which is what makes the archive reproducible across agents; and the entry **names** are relative to the publish directory with no root folder, which is what App Service requires — its ZIP guidance says to add "everything in the output directory of the `dotnet publish` command, excluding the output directory itself". The `-type f` and the collation are both load-bearing: a locale-dependent sort is a locale-dependent archive |
| `zip -X -D -q` | `-X`/`--no-extra`: "Do not save extra file attributes (Extended Attributes on OS/2, uid/gid and file times on Unix)"; `-D`/`--no-dir-entries`: "Do not create entries in the zip archive for directories"; `-q`: quiet | `-X` removes the two remaining sources of build-agent noise — the uid and gid the agent happened to run as, and the high-resolution Unix timestamps that survive the normalization above. `-D` removes directory entries, whose own attributes are agent state and which the unpack does not need. `-q` keeps a file list out of the run log. What is left in the archive is content, names and one timestamp. The `rm -f` ahead of it is not tidiness: `zip` **adds to and replaces entries in an existing archive** rather than starting a new one, so an archive left in a reused workspace would make the output depend on what a previous run put there |
| `sha256sum <file> > <file>.sha256`, and `sha256sum -c <file>.sha256` at every consumer | `-c`/`--check` "read checksums from the FILEs and check them", and "the sums are computed as described in FIPS-180-2" | **SHA-256** is the algorithm, named rather than implied, and one sidecar per promoted file is what makes verification a single command at each consumption point. The sidecar is written from inside `artifacts/` so it records the bare file name and `sha256sum -c` resolves it beside the file — the form [06](06-azure-hosting-recommendations.md) §6.2.1's release invocation already uses. The two bundle sidecars are computed **before** `dotnet test`, because the suite is itself a consumer of those bundles |

**What a bundle build actually does — and therefore what the one-compile guarantee covers and what it does
not.** A reader of the sequence above would reasonably conclude that `dotnet build -c Release` is the only
compile in it and that every artifact below inherits the locked restore that preceded it. That is true of
three of the six payloads and **false of the two bundles**, so it is stated plainly rather than left
as an inference. `dotnet ef migrations bundle` does not package the compile it was handed. It **generates a
project of its own and publishes that**, in four steps that the pinned tool's shipped implementation of the
command performs in this order:

1. It writes a **temporary SDK-style project into a scratch directory** — a generated `Program.cs`, one
   `PackageReference` to `Microsoft.EntityFrameworkCore.Design` at the EF Core version it detected in the
   target project, and a `ProjectReference` back to the project named by `--startup-project`. The project
   sets `PublishSingleFile`, which is what makes the output one executable.
2. It **copies the `global.json` and the `NuGet.config` chain it discovers**, walking up from the working
   directory, into that scratch directory — so the generated project sees the same SDK band and the same
   package sources as the build around it.
3. It runs **`dotnet publish` on that generated project**, forwarding `--configuration Release` and, *where
   they are given*, the runtime identifier from `-r` and the self-containment choice from
   `--self-contained` — and **passing neither `--no-build` nor `--no-restore`**. **Neither is given here**,
   under the framework-dependent pin below, so no runtime identifier reaches that publish and no runtime is
   embedded in the output. The generated project is nonetheless restored and compiled, and it may build the
   referenced startup project again rather than reuse the Release compile above.
4. It moves the resulting single-file executable to the `-o` path and deletes the scratch directory. It also
   warns when the startup project carries an `appsettings.json`, which is the same hazard the
   design-time-input rule above closes by placing no settings file beside a bundle.

**Two claims are therefore withdrawn rather than repaired, because no flag fixes either.** There is no
option that makes a bundle build reuse the earlier compile, and there is none that gives the generated
project a lockfile:

- **The one-compile guarantee is narrowed to its true width.** `dotnet build -c Release` is the only compile
  behind **`site.zip`, the test assemblies and `provision-admin.zip`** — `--no-build` on both publishes and
  on `dotnet test` is what holds that, and it is a real and checkable property. It does **not** extend to
  `artifacts/efbundle-catalog` and `artifacts/efbundle-identity`: each of those is the output of its own
  publish of its own generated project.
- **"Fully locked" is not claimed of the bundles.** The committed per-project lockfiles and
  `dotnet restore --locked-mode` (§6.4) bind the restore of the three committed projects. The generated
  project is created after that restore has finished and has no lockfile of its own, so its transitive
  graph is resolved rather than locked.

**What does bound them: four controls that are real, and one measurement that proves the controls held.**
The controls are not a weaker restatement of the lockfile — two of the four are mechanical consequences of
what item 2 of that list copies into the generated project:

| Control | Why it binds the generated project |
| --- | --- |
| The committed **`NuGet.config`** (§6.2) | The tool copies the discovered chain into the scratch directory, so the generated project's restore sees `<clear />` plus nuget.org and nothing else. A machine-level or private feed cannot enter through it |
| The committed **`global.json`** band (§3, §6.1) | Copied the same way, so the generated project is built by an SDK on the same `8.0.400` feature band under `rollForward: latestPatch` — the same toolchain, not merely a compatible one |
| The **package folder the earlier locked restore already populated** | The generated project's only package reference is `Microsoft.EntityFrameworkCore.Design` at the **exact** `8.0.30` pin of §7.2 — one version, no range — and the locked restore above has already placed that package and its closure in the packages folder the resolve will draw from |
| The **build image digest** (the reproducibility rule below) | The SDK and the targeting packs the framework-dependent publish compiles against, and every utility around it, are the same bytes in the build being compared as in the build it is compared with |

**The measurement, because a control nobody checks is a claim.** Reproducibility of the two bundles is
**verified empirically rather than asserted**: two builds of the **same commit**, on the **same recorded
build image digest**, with the same committed manifests, must produce **byte-identical** bundles, and the
check is a comparison of the `efbundle-catalog.sha256` and `efbundle-identity.sha256` values recorded in the
two runs' `release.txt` files. Equality is the expected result — a single-file bundle's own identifier is
derived by hashing the content it bundles rather than generated per build, so there is no per-build nonce to
defeat it — which is precisely why an **inequality is a finding rather than a tolerance**: it means one of
the four controls above was not actually holding, and the difference is investigated and closed rather than
accepted. Where the check is run, and by which job, is the pipeline's ([03](03-modernization-roadmap.md)
owns the provider gate); that it is run, on the values named here, is this section's requirement.

**And one property holds regardless of any of the above, because it is about which bytes travel rather than
about how they were produced.** The bundle bytes the suite **executes** are the exact bytes promoted: each
bundle's sidecar is computed immediately after the bundle is produced and **before** `dotnet test`, the
suite verifies that sidecar and then executes that same file
([05](05-aspnet-core-migration-approach.md) §12.2), and the release record and the publication step consume
those same files with no regeneration between. So even for the two artifacts the one-compile guarantee does
not cover, "the artifact that was tested is the artifact that ships" is a property of one file with one
hash — which is the property a release actually depends on.

**Why a script exists at all, alongside the bundle.** Because a bundle cannot be inspected: Microsoft's
own guidance on applying migrations records that, unlike a SQL script, a bundle provides no way to see the
SQL it will execute or list the migrations it contains, and advises generating a script where a deployment
requires SQL review. So there are **four** artifacts from two contexts, not two: `efbundle-catalog` and
`efbundle-identity` are what execute, and `migrations-catalog.sql` and `migrations-identity.sql` are what a
reviewer reads. **Those four strings are the whole of the migration naming contract**, which the third rule
above completes with the two payload archives, and the two roles are never merged: a reviewed script that is
then *applied* would mean two things can issue DDL, and
[06](06-azure-hosting-recommendations.md) §6.2 admits exactly one.

**The bundles are framework-dependent, and that is one pin stated in one place.** Both bundle invocations
above pass **neither `--self-contained` nor `-r`/`--target-runtime`**, so each bundle carries the
migrations and **not** a runtime, and the **runner prerequisite is therefore explicit: a .NET 8 runtime on
the job image** that executes them — the runtime and not the SDK, since a bundle needs no SDK. The
spelling matters and is easy to get wrong: `dotnet publish` has an explicit `--no-self-contained` switch,
while **`dotnet ef migrations bundle` exposes only the positive `--self-contained` option** together with
`-r`/`--target-runtime`, so on this command framework-dependent is selected **by omitting both** rather
than by a negative flag. Verified against the pinned tool: `dotnet ef migrations bundle --help` on
`dotnet-ef` `8.0.30` lists `--self-contained` and `-r|--target-runtime` and carries no
`--no-self-contained`. Writing that spelling into the sequence would fail the build at the first bundle
step.

**One decision inside that pin is consumed from its owner rather than made here.** The *release-side*
property — that the job image executing a bundle supplies a matching .NET 8 runtime, on the agent and
platform 06 selects — is [06](06-azure-hosting-recommendations.md) §6.2.1's and §12.3's, because it is a
property of the release environment rather than of this build. This section states the two invocations
exactly as they run, so the sequence is runnable as written rather than a fragment a reader has to assemble
from two documents; **there is no second, self-contained form of these bundles anywhere in this document**,
because a self-contained bundle and a framework-dependent one are different bytes with different runner
prerequisites and a reader cannot be left to choose between them. **The six artifact names are not in that
category**: they are this section's, and 06 consumes them.

**The two publishes are framework-dependent too, and that is the same pin stated for the other two
artifacts rather than a second decision.** Both `dotnet publish` invocations in the sequence above are the
**plain** form — `-c Release --no-build -o …` — passing **no `-r`/`--runtime`, no `--self-contained` and no
`RuntimeIdentifier`, and therefore producing **framework-dependent** output for the site payload **and for
`tools/provision-admin`**. This is stated explicitly because the tool's mode is the one a reader is most
likely to assume differently: an operator command that runs on a jump host looks like a candidate for a
self-contained single file, and it is not one here. Five consequences follow, and each is the same shape as
the bundle rows above:

- **No RID appears anywhere on this path**, so there is no `linux-x64` or any other runtime identifier in
  either publish, in the project files (§5.2), or in the four artifact names. The negative spelling is
  available on this command — `dotnet publish` does expose `--no-self-contained`, unlike
  `dotnet ef migrations bundle` — and it is **still not passed**, because framework-dependent is already the
  default when no RID is given and a redundant flag invites the reader to think a choice was made per
  invocation rather than once.
- **The runtime prerequisite is explicit, and it is the same one the bundle pin above states.** Whatever
  executes these payloads must supply a **matching .NET 8 runtime** — the runtime and not the SDK, for the
  site payload and the tool alike. [06 §12.7.2](06-azure-hosting-recommendations.md)'s executor image build
  asserts both halves of that in `RUN` layers rather than assuming them: it requires
  `provision-admin.runtimeconfig.json` to be present and to carry a framework reference, requires
  `includedFrameworks` to be **absent** — the marker a self-contained publish would have written — and
  requires `dotnet --list-runtimes` to report a `Microsoft.NETCore.App 8.` entry, so an image that could not
  execute a framework-dependent payload does not build. [06 §9.5](06-azure-hosting-recommendations.md)'s
  break-glass condition 5 carries the same mode to the operator's jump host, naming *"the same
  **framework-dependent** publish of `tools/provision-admin` that the build produces for the pipeline job"*.
  Both consume this section for the mode, which is why it is stated here in as many words.
- **The image contents are unchanged by it**, because a framework-dependent payload is the one the single
  runtime stage of §13.3 already expects: no embedded runtime is copied in, nothing is extracted from a
  self-contained bundle, and the base image's own runtime is what executes. A self-contained publish would
  have put a second copy of the runtime inside an image that already has one and frozen its servicing.
- **The checksums and the promoted count are unaffected.** The six sidecars cover the published bytes as
  they are: `13 = 6 + 6 + 1` still holds, the tool's payload and its sidecar are one of the six pairs, and
  [06 §9.5](06-azure-hosting-recommendations.md)'s break-glass condition 5 records the tool's SHA-256 in the
  release record from that same sidecar. Changing the publish mode would change every one of those digests,
  which is precisely why the mode is pinned in one place rather than chosen at release time.
- **Acceptance verifies the prerequisite rather than the artifact's self-sufficiency.** The provisioning
  assertions above run the **published** tool on the pipeline-shaped invocation, so a job image missing a
  matching runtime fails at the first invocation with a host-not-found error rather than at a later
  behavioural assertion — which is the outcome to want, and the reason the prerequisite is a property of the
  stage's execution environment, owned release-side by [06 §6.2.1](06-azure-hosting-recommendations.md) and
  §12.3 exactly as the bundles' is, rather than a note in this sequence. **There is no second,
  self-contained form of either publish anywhere in
  this document**, for the same reason the bundles have none: different bytes, a different prerequisite, and
  no reader left to choose.

**The reproducibility rule — three things the sequence enforces, and one contract it owns.** "The archive is
deterministic" and "the build is reproducible" are properties a *command* has, so each is stated as the
thing in the sequence that makes it true and fails the build where it is not. A document that only claimed
them would leave three ways for two builds of one commit to disagree, and all three are closed above:

- **One fixed time zone: `export TZ=UTC`.** Normalizing modification times with `touch -h -d "$SOURCE_DATE"`
  is necessary and **not sufficient**, and the reason is a property of the format rather than of the tool: a
  zip entry stores an **MS-DOS date and time**, whose packed fields are a day, a month, a year offset and a
  local hour, minute and two-second count — with **no time-zone field at all**. Info-ZIP's own manual states
  the same thing from the other side, that Unix file times are GMT while most other systems' are local, and
  that the conversion depends on `TZ`. So on the Linux agent [06](06-azure-hosting-recommendations.md) §12.3
  requires, the same normalized mtime archived under two different `TZ` values is written into the entry as
  two different local stamps, and two archives of identical content differ. Fixing the zone for the whole
  build removes the last input that is a property of the agent rather than of the commit. `SOURCE_DATE`
  itself is unaffected — git's `%cI` carries the commit's own offset — so pinning the zone changes the
  archive and nothing else.
- **One pinned build image, recorded by digest and compared between builds.** The build runs inside a
  container image **addressed by digest**, `BUILD_IMAGE_DIGEST` is required before the first operation, and
  it is written into `release.txt` as the `image=` line beside the run identifier and the commit. That makes
  the digest part of the release's identity rather than a fact about a machine that no longer exists: **two
  builds compared for reproducibility must record the same digest**, and a comparison across two different
  digests is not a reproducibility result — it is a comparison of two toolchains, and its outcome says
  nothing either way. This document deliberately **names no tag and no digest value**: which image, and
  therefore which digest, is selected at the CI provider gate that
  [03](03-modernization-roadmap.md) owns, alongside the provider itself. What is fixed here is that the
  selection is by digest, that the value is asserted, and that it is recorded.
- **Pinned utilities, asserted rather than printed.** `zip`, `sha256sum`, `find` and `git` are inputs to the
  archive in exactly the way a package version is an input to a compile — Info-ZIP's `-X` semantics, GNU
  `sha256sum -c`, GNU `find -printf` and git's `%cI` are all specific to those implementations — so the
  sequence **compares each one's reported version against a recorded expected value and exits non-zero on a
  mismatch**, and additionally runs `find -maxdepth 0 -printf` to prove the GNU extension the archive
  depends on is present rather than merely that a `find` exists. Printing a version and proceeding is what
  this replaces: a log line nobody reads is not a pin. The expected values are fixed with the image digest,
  at the same gate; the assertion is the build's. This is the build-time half of a constraint
  [06](06-azure-hosting-recommendations.md) §12.3 also enforces at **provisioning** time, where a missing
  utility costs an image rebuild rather than a release — the two are complementary, and neither replaces the
  other, because an image can satisfy a provisioning check and then be replaced.

**This section owns the canonical archive contract, and a consumer quotes it verbatim.** The contract is the
two lines the sequence above runs, and their exact form is the whole of it:

```bash
( cd "artifacts/$OUT" && find . -type f -printf '%P\n' | LC_ALL=C sort \
    | zip -X -D -q -@ "../$OUT.zip" )
```

**The name list is newline-delimited.** `zip -@` reads "only one filename per line", so the pipeline is
`find … -printf '%P\n'` into a plain `LC_ALL=C sort` into `zip -@`. **`sort -z` is not the contract** and
neither is `find -print0`: the NUL-delimited pair is a correct idiom for tools that accept NUL-delimited
input, and `zip -@` is not one of them, so a consumer that quotes the NUL form has written a command that
either fails or archives one entry whose name is every path joined together. A consumer restating the
contract states these two lines; where it needs to describe the sort, it says `LC_ALL=C sort` over
newline-delimited names, and the `LC_ALL=C` is not optional because a locale-dependent order is a
locale-dependent archive.

**The promoted set, and the chain that makes promotion mean something.** A directory in a build workspace is
not an artifact: `artifacts/site` exists for the length of one job, is writable by every later step in it,
and is destroyed with the workspace. Nothing may consume it, and a consumer that reads a *path* rather than
a verified file has no way to know which build produced what it read. So the set is stated as files, with an
identity and a verification point:

| | The set |
| --- | --- |
| **Payloads — six, and no others** | `efbundle-catalog` and `efbundle-identity` (execute), `migrations-catalog.sql` and `migrations-identity.sql` (reviewed, never executed), `site.zip` (the web payload, the contents of the publish output with no root folder) and `provision-admin.zip` (the operator console, whose ten verbs share it — §12.9) |
| **Integrity — one sidecar each** | `<name>.sha256`, SHA-256, computed in the build immediately after the file is produced and before anything reads it |
| **Identity — three values** | The **run identifier** of the build that produced the set, the **full commit** it compiled, and the **digest of the build image** it ran in — `run=`, `commit=` and `image=`, recorded in `release.txt` beside the six sidecar lines. No one of them is enough: a commit can be built more than once, a run identifier says nothing about what was in it, and without the digest a later reproducibility comparison cannot establish that the two builds shared a toolchain (the reproducibility rule above). Under the container option one further line appears, `site-image=`, carrying the deployed image by digest — **a different key from `image=` on purpose**, since a consumer matching `image=` loosely would otherwise read the build's toolchain digest as the deployment reference |
| **Publication** | The sequence generates **twelve files into `artifacts/`** — the six payloads and their six sidecars — and the build's last step attaches those twelve **and** `release.txt` to **that run**, so **thirteen files are promoted**: `13 = 6 + 6 + 1`. Which provider step attaches them is the pipeline's ([03](03-modernization-roadmap.md) owns the provider gate, and §12.2's CI row records what this document contributes to it) |
| **Verification — three checks, literal** | `[ -f artifacts/release.txt ]` (a run with no release record publishes nothing), `( cd artifacts && sha256sum -c ./*.sha256 )` (every sidecar against the file beside it) and `test "$(grep -cE '^[0-9a-f]{64} ' artifacts/release.txt)" -eq 6` — **exactly six checksum lines, no more and no fewer**, which is what makes a missing or an extra payload a failure rather than a difference nobody counted. All three are in the sequence above, in that order, as the publication precondition |

**The set is closed at exactly the files the Publication row above counts, and exactly two files a consumer
names are deliberately outside it, for a reason that is a lifecycle rather than an omission.** Both are
**separately attached evidence**: neither is promoted, neither is counted in that set, neither is written
into `artifacts/` and neither gets a `.sha256` sidecar. They differ in one respect, stated rather than
blurred: **nothing in a release reads `sessioncache.sql`** — it is read by an approver — whereas the
image record's `site-image` line **is** the digest the deployment deploys under the container option, which
is why it is attached to the run rather than left in a log.

| Evidence file | Owner and producer | Why it is outside the promoted set |
| --- | --- | --- |
| `sessioncache.sql` | [06](06-azure-hosting-recommendations.md) §6.4, produced by the operator preparing the cache-table step | No step of §13.4 generates it, so it never appears in `artifacts/`; it is read by an approver, never executed by a release |
| `image-build-record.txt` — **container option only** | This section's conditional subsection below: **produced by the image build step**, in the same job, after the release record and before publication. Written **outside `artifacts/`**, at the build workspace root, and its three-line schema is stated in full there | The site image is not a file the release downloads and verifies by checksum: it is addressed by digest in a registry, so the record of that digest is attached as evidence beside the run rather than promoted as a payload. Being outside `artifacts/` is what keeps the Publication row's twelve generated and thirteen promoted files identical under both hosting options |

**Thirteen is computed rather than asserted, from the steps above that produce a file.** The sequence has
exactly six payload-producing steps: two `dotnet ef migrations bundle` invocations writing
`-o artifacts/efbundle-…` (2), two `dotnet ef migrations script --idempotent` invocations writing
`--output artifacts/migrations-….sql` (2), and one archive loop over `site` and `provision-admin` (2) —
`2 + 2 + 2 = 6`. Each payload gets exactly one sidecar, in the same step or in the loop that follows it
(6), and the release record is the thirteenth file (`+1`). The build checks that arithmetic against the
disk before anything is published — the `find artifacts -maxdepth 1 -type f` count in the sequence — and
both numbers are recomputable from this document rather than taken on trust:

```bash
# payload-producing steps in the sequence above -> 6
# the !/`/ guard skips this section's PROSE, which quotes the same option strings inside code spans
awk '/^### 13\.4 /{s=1} /^## 14\. /{s=0}
     s && !/`/ && /-o artifacts\/efbundle-/{n++}
     s && !/`/ && /--output artifacts\/migrations-/{n++}
     s && /^for OUT in site provision-admin/{n+=2}
     END{print n+0}' docs/modernization/04-dotnet8-migration-strategy.md
# 6 payloads + 6 sidecars + release.txt = 13 promoted files
```

`sessioncache.sql` is the reviewable DDL of the session cache table, and it is **not a build artifact**: it
is produced by `dotnet sql-cache script dbo SessionCache --idempotent --output sessioncache.sql` at the
moment the cache-table step is prepared, by the operator running that step, and it is attached to that
change record as **evidence** — a file an approver reads, not a file a release consumes.
[06](06-azure-hosting-recommendations.md) §6.4 owns it in exactly that role and states that nothing in the
release executes it and no release step reads it; the command that *creates* the table remains
`dotnet sql-cache create`, from the `dotnet-sql-cache` tool pinned at `8.0.30` in §6.3. Four consequences,
which are what make "closed" a usable claim rather than a bare count:

- **It has no producer in the sequence above, by design.** No step of §13.4 generates it, so it never
  appears in `artifacts/`, and adding it there would make the build produce a file no consumer verifies.
- **It gets no sidecar and no publication.** Integrity of a review file is the change record's, whose own
  retention and approval mechanics are the release's rather than the build's, so a seventh `.sha256` and a
  fourteenth published file would both be claims this document cannot keep.
- **Its retention is the change record's.** The 90-day-and-ten-runs window below governs *promoted*
  artifacts; an approval attachment lives with the approval, and the two windows are not the same window.
- **Nothing hands it off between stages.** A build that published it would invite a release step to read it,
  which is the one thing 06 §6.4 forbids — two paths to one database object being how the two diverge.

So the closure is precise in both directions: **the promoted set is what the Publication row counts and no
file beyond it is promoted**, and each of the two evidence files is outside the set on purpose, with a named
owner, a named producer, a named moment and a named role. **The Publication row above is the only statement
of these counts in this document**; every other section that needs one cites §13.4 rather than restating it,
for the same reason the commands live here alone.

Four rules govern what happens next, and they are rules rather than intentions because each one is a thing a
pipeline does by default if nobody forbids it:

- **A consumer downloads, verifies, and only then uses.** Every stage that needs one of the six —
  the suite, the migrate stage, the deploy stage — obtains it as **this run's published artifact**, runs
  `sha256sum -c <name>.sha256` against it, and **refuses to proceed on a mismatch or a missing sidecar**,
  with nothing executed and nothing deployed. Verification is at **every** consumption point rather than
  the last one, because a file that changed between two consumers is a file whose effect nobody has
  observed. Where the suite runs inside the build job it reads the same files from the workspace and
  verifies them the same way — the rule is about the check, not about which job the check happens in.
- **Nothing is regenerated, rebuilt or fetched from a mutable location.** No stage runs `dotnet ef`, no
  stage recompiles, and no stage reads a shared folder, a `latest` alias or a branch tip. The prohibition
  is [06](06-azure-hosting-recommendations.md) §12.5's; the verification above is what detects a violation
  of it rather than trusting that the prohibition was followed.
- **A rollback is a locator, not a rebuild.** The **rollback locator is the run identifier of a previously
  promoted run**, and everything needed to act on it is inside that run: `release.txt` names the commit,
  and the sidecars establish that the files being redeployed are the files that run produced. Rolling back
  therefore means redeploying that run's verified `site.zip` — never rebuilding the old commit, which would
  produce a compile no suite has run against. A rollback across a schema change is not symmetrical with a
  deployment and is out of this document's half: the migration rollback position is
  [05](05-aspnet-core-migration-approach.md)'s and the release-side procedure is
  [06](06-azure-hosting-recommendations.md) §6.2's.
- **Retention has a number and deletion has an owner.** A promoted run's artifact set is retained for **at
  least 90 days and at least the last ten promoted runs, whichever is longer** — the window inside which a
  rollback locator must still resolve — and **deletion is the provider's retention policy, configured by
  the pipeline owner**. No stage and no person deletes another run's artifacts by hand, and a set inside the
  rollback window is exempt from any cleanup. Two things this is *not*: it is not the workspace directory,
  which dies with the job and is nobody's rollback source; and it is not the **run-record** retention
  criterion [06](06-azure-hosting-recommendations.md) §12.1.1 puts on the provider gate, which concerns the
  record of what happened rather than the bytes the run produced. A provider satisfying one says nothing
  about the other, so both are stated.

Four ordinary utilities follow from the commands above — Info-ZIP `zip`, coreutils `sha256sum` and GNU
`find`, alongside `git` — and they belong with the SDK band as properties of the build image that
[06](06-azure-hosting-recommendations.md) §12.3 pins at provisioning time. Naming them here is not the whole
of it: the sequence above **asserts each one's version against the recorded expected value and fails on a
mismatch**, per the reproducibility rule, so an agent that is missing one or carries a different one fails
before the first artifact is produced rather than at the archive step with a release half-built.

**Under the container option — conditional, and it compiles nothing.** The recommended target is
**App Service on Linux with code deployment**, and that path deploys `site.zip`: **no image exists in it and
nothing in this subsection runs**. Everything below is conditional on
[06](06-azure-hosting-recommendations.md) §4.1 selecting the container-native platform instead — that
selection is 06's and not this section's. Where it is selected, the image is built
**from the already-published, tested output** and not by restoring and recompiling inside itself. That is
the whole point: an image whose own build stage restores and compiles contains bytes the Release compile
above never produced, so `site.zip`'s checksum covers nothing that is running and the tested-bytes property
is lost exactly where it is being claimed. The build context is the publish directory, so nothing else can
enter the image or even the context transfer:

```bash
# CONDITIONAL — container-native option only. Same job, after the release record above and
# before the artifacts are published, so the digest travels with the run that produced it.
# REGISTRY_LOGIN_SERVER, REGISTRY_NAME and IMAGE_REPOSITORY are PROVISIONING-TIME inputs. This
# section names none of the three values — the bullet below says why — and fixes only the SHAPE
# of the reference they compose and the tag that is appended to it.
IMAGE="$REGISTRY_LOGIN_SERVER/$IMAGE_REPOSITORY"          # one registry, one repository
docker build -f Dockerfile -t "$IMAGE:$RUN_ID" artifacts/site
docker push "$IMAGE:$RUN_ID"                              # push prints the manifest digest
DIGEST="$(az acr repository show -n "$ACR_NAME" \
    --image "mvcmusicstore/site:$RUN_ID" --query digest -o tsv)"

# The site image's digest is SEPARATELY ATTACHED EVIDENCE, not a promoted payload: it is written to
# its own file and attached beside the run. It is NOT appended to release.txt, whose own image= line
# is the BUILD image's digest and whose checksum lines must number exactly six (the third
# verification check above), so a second image= line there would make one key mean two things.
# The file is written OUTSIDE artifacts/ — at the build workspace root — so the Publication row's
# count of twelve generated files, and the thirteen promoted, are unaffected by the container option.
BASE_IMAGE="$(sed -n 's/^FROM[[:space:]]\+\([^[:space:]]\+\).*/\1/p' Dockerfile | head -1)"
printf 'site-image=%s@%s\nbuild-id=%s\nbase-image=%s\n' \
    "$IMAGE" "$DIGEST" "$RUN_ID" "$BASE_IMAGE" > image-build-record.txt
```

**The record's schema — three lines, and this section owns it.** `image-build-record.txt` is defined here
in full, so [06](06-azure-hosting-recommendations.md) cites this definition rather than restating it and the
two documents cannot describe two files under one name. It carries **exactly three lines, in this order**,
each a `key=value` with no blank line and no fourth key:

| Line | Key | Value |
| --- | --- | --- |
| 1 | `site-image` | The **site image digest**, as the registry's own by-digest reference form — `<loginServer>/mvcmusicstore/site@sha256:<digest>` — read back from the registry after the push |
| 2 | `build-id` | The **build identifier**: the run identifier of the build that produced both the image and that run's `release.txt`, which is what ties the image to the bytes it was built from |
| 3 | `base-image` | The **base image digest**, taken from the reviewed digest-pinned `FROM` in the `Dockerfile` — `mcr.microsoft.com/dotnet/aspnet:8.0@sha256:<digest>` — so the record states what the image was built *on* as well as what it *is* |

Four properties come with that schema, and each is a consequence of the file being evidence rather than a
payload: it is written **outside `artifacts/`**, at the build workspace root, so no counted or promoted set
changes when the container option is taken; it is **produced by the image build step above**, in the same
job, after the release record and before publication; it gets **no `.sha256` sidecar**, because integrity of
an evidence attachment is the run's rather than the build's, and a seventh sidecar would break the
six-checksum-line check; and it exists **only under the container option**, so a code-deployment release has
no such file and nothing looks for one. The commit is deliberately **not** a fourth line: it is in that
run's `release.txt` under `commit=`, and `build-id` is the key that joins the two.

Six properties make that a contract rather than three commands, and each is sourced:

- **The base image is pinned by digest, in the `Dockerfile`, where it is reviewed like code.** *Which* image
  and which band are not this section's: the runtime image is the ASP.NET Core 8.0 runtime,
  `mcr.microsoft.com/dotnet/aspnet`, at the band `global.json` pins, which is
  [06](06-azure-hosting-recommendations.md) §12.3's constraint and §4.4's artifact. What this contract adds
  is the **form of the reference**, because that is a reproducibility property: the Dockerfile reference
  admits `FROM <image>[@<digest>]`, so the base is written as
  `mcr.microsoft.com/dotnet/aspnet:8.0@sha256:<digest>` — the tag for a human, the digest for the builder. A
  tag alone is not a pin: it keeps receiving servicing updates, which is what makes two builds of one commit
  produce two different images. Refreshing the digest is therefore a deliberate commit, and the patch
  cadence behind that decision is [06](06-azure-hosting-recommendations.md)'s.
- **One registry and one repository — and no naming scheme for either is designed by this assessment.** The
  **counts** are this section's, because both follow from how the bytes are promoted: a single **private**
  registry serves every environment, since promotion is by digest and a second registry would mean a second
  copy of the bytes with nothing establishing that the two agree; and there is exactly **one** repository,
  since a second would be a second site image. **The names are not this section's, and they are no sibling
  document's either — neither the registry's resource name nor the repository path is deferred to a naming
  scheme in [06](06-azure-hosting-recommendations.md) or claimed here.** A registry is a **platform
  prerequisite of the conditional container-native
  option** rather than a designed artifact: it is outside the map §12.7 closes, it appears in no
  deliverable's artifact list, and [06](06-azure-hosting-recommendations.md) §12.1.2 states in terms that
  that document "deliberately specifies **no registry estate**: no dedicated image beyond the conditional
  one, no registry resource, no signing identity and no scan-and-admit chain", and that "where the secondary
  platform is later selected, the registry and its controls become a provisioning decision taken at that
  point, with its own approval". So **the concrete registry and repository names are supplied at
  provisioning time by whoever selects that option**, and the image reference is given here in the generic
  form the conditional path needs — the registry documentation's own two:
  `[loginServerUrl]/[repository][:tag]` by tag, and `[loginServerUrl]/[repository]@[sha256:digest]` by
  digest. What this section genuinely owns of the reference is what is derived from **this build**: the tag
  scheme in the next bullet, the digest the release deploys in the one after it, and the runtime base's
  digest pinning in the one above.
- **The tag is the run identifier, and never `latest`.** Both the registry's tagging guidance ("use unique
  tags for deployments") and the container platform's own container documentation ("avoid using static tags
  like `latest` … use unique tags for each deployment") say so, and the reason matters here: a reused tag
  makes a replica that restarts pull something other than what its siblings are running.
- **The release deploys the digest, not the tag.** The digest is recorded on the `site-image` line of
  `image-build-record.txt` — the container option's **separately attached evidence** file, written outside
  `artifacts/` at the build workspace root, named in the manifest's evidence table above, and **not**
  promoted, not counted in the promoted set, given no `.sha256` sidecar and not appended to `release.txt` —
  and it is the reference the deployment uses, because pulling by digest "guarantees the image version
  you're pulling, even if you push an identically tagged image later". Its `build-id` line carries the same
  run identifier as that run's `release.txt`, which is what ties the image to the bytes it was built from,
  and its `base-image` line records what it was built on. The digest is read back from the registry after
  the push — the `az acr repository show` form above, or the digest the push itself prints where the Azure
  CLI is not on the agent.
- **Scanned, signed, and verified before it runs — delegated to a named stack rather than to an unnamed
  intention.** An unscanned, unsigned or unverified image is **not a promotable artifact**, and that is this
  section's rule because it is a property of the artifact. Everything about *how* it is achieved belongs to
  [06](06-azure-hosting-recommendations.md) §4.4, which owns the container deployment artifact end to end
  (its §4.3 designates that subsection as the owner) — but a delegation has to name what is being delegated,
  or the receiving document has nothing to implement. The stack is four named components and one admission
  rule:
  - **The registry is Azure Container Registry** — the same single private registry and single
    `mvcmusicstore/site` repository fixed above, since scanning, signing and verification all attach to the
    manifest in the registry rather than to a copy of it.
  - **Scanning is Microsoft Defender for Containers' vulnerability assessment for registry images**, which
    Microsoft describes as scanning images on push, on import and on recent pull and surfacing findings with
    recommended remediation. Promotion is gated on its result, and the **severity threshold that fails a
    promotion** is 06's.
  - **Signing is the Notary Project's `notation`**, with the signing certificate **hosted in Azure Key
    Vault** and reached through the `notation-azure-kv` plug-in, so no signing key material exists on the
    agent or in the repository. Signing happens after the push, against the digest read back above.
  - **Verification is `notation verify` against a trust policy pinned in the release** — trust store,
    trusted identities and the policy's own version, held with the release rather than fetched at deploy
    time, so what a signature is checked against cannot drift between two deployments of one digest.
  - **Admission is fail-closed and strictly by digest.** A missing scan result, a scan result above the
    threshold, a missing signature or a failed verification each **stop the deployment**, and the reference
    verified is the digest recorded in `image-build-record.txt` — never a tag, because verifying one
    reference and deploying another verifies nothing.

  **None of this adds a dependency to the application, whichever mechanisms are selected.** A scanner, a
  signing client and a registry client are **agent utilities**, in the same category as `zip` and
  `sha256sum` above: they add **no `PackageReference`** to §7.2's pin list and **no entry** to §6.3's tool
  manifest, both of which stay exactly as those sections fix them, and a scanning capability enabled on a
  subscription is not a package at all. So the supply-chain controls are a property of the pipeline and the
  registry, and the target's dependency graph is unchanged by them — which is also why the commands are not
  this section's to write.
- **Retention matches the rollback window, and it is not the untagged-manifest policy.** Every promoted
  image keeps its unique run-identifier tag, so this scheme produces no untagged manifests — and the
  registry's own retention documentation warns that where systems pull **by digest** an untagged-manifest
  policy must not be used, since it deletes exactly what such a system addresses. Retention is therefore an
  **explicit purge of tags outside the rollback window**, on the same 90-day-or-ten-releases rule as the
  archives, owned by the platform owner, and a digest that is deployed or inside the window is never
  purged.

Everything about the image other than its **stage count** and its **input** is
[06](06-azure-hosting-recommendations.md) §4.4's, and this section changes none of it: the **non-root
runtime user**, the **absence of database tooling** in it, and the **listening-port contract** of §4.4.1.
Those two are this section's because both are properties of which bytes are promoted, and they are stated
next.

**The image has one stage, and that is a requirement of this contract rather than a matter of style.** The
manifest is **the digest-pinned runtime base plus one `COPY` of the already-published output — no SDK stage,
no restore and no compile inside the image**. A stage that restored and compiled would produce assemblies no
suite ran against and no sidecar covers, so the tested-bytes property would be claimed in the one place it
had just been given up. Three properties follow, and they are what the single stage buys:

- **One `COPY` of one path**, and the path is `artifacts/site` — the same bytes `site.zip` and its sidecar
  cover — so what enters the image is one directory a reviewer can list rather than an enumeration to audit.
- **Locked restore is satisfied earlier and more strongly.** It is the `dotnet restore --locked-mode` of the
  build above (§6.4), where a resolved-graph change fails before anything is produced, rather than repeated
  inside an image build where the same failure would surface later and against a graph nothing verified.
- **There is no build-context residual, and therefore no ignore file.** A `COPY` list bounds what enters the
  image but not what the client transfers as build **context**; with the context set to `artifacts/site` the
  tracked legacy tree is not in the context at all, which is why the map carries a `Dockerfile` and **no
  `.dockerignore`** (§12.7).

So the division is exact and there is nothing left to reconcile: the **stage count and the input** are this
section's, and every other property of the manifest is §4.4's.

**Where this document's half stops.** Everything above happens in **one build job**, under the build's own
design-time environment (§13.3). What happens to the resulting files *at release time* — which stage
downloads them, the principal it runs under, the order the two bundles are applied in, the pre-DDL target
assertion, the exit-code gate and the traffic decision that follows — is
[06](06-azure-hosting-recommendations.md) §6.2.1's, §6.3's and §12.1's, and is not restated here. The join
between the two halves is the property this section exists to make true: **the bytes 06 deploys and verifies
are the bytes this document's test invocation exercised.** That rests on two facts and deliberately not on
one overstated one. `site.zip`, the test assemblies and `provision-admin.zip` come from **one Release
compile**, held there by `--no-build`. The two bundles come from their own generated publishes — the
mechanism is stated in full above, and the controls that bound it are the copied `NuGet.config` and
`global.json`, the exact Design-package pin, the already-populated package folder and the recorded build
image digest, with byte-identity between two builds **verified** rather than assumed. What is true of all
six without qualification is the part a release depends on: **every promoted file has a hash computed before
anything could have touched it, the suite consumed those same files, and every consumer re-checks the hash
before use.**

---

## 14. Roll-up

### 14.1 The strategy in ten statements

1. The target framework is **`net8.0`** (§2.1). The support window is an approval decision recorded in
   [07](07-effort-risks-sequencing.md)'s risk register, and a later LTS target is a five-part change, not
   a one-line edit (§2.3).
2. The SDK is pinned at feature band **`8.0.400`** with **`rollForward: latestPatch`**, and the build image
   must satisfy it (§3).
3. The project converts from the **302-line MSBuild 2003 format** to **SDK-style** with implicit globbing,
   `PackageReference`, and assembly metadata absorbed into MSBuild properties (§5).
4. Four net-new manifests are committed: `global.json`, `NuGet.config`, `.config/dotnet-tools.json` and a
   **per-project `packages.lock.json`** restored in locked mode. `NuGet.config` fixes which sources are
   consulted; the lockfile fixes what resolves from them (§6).
5. Every Microsoft-shipped .NET 8 package is pinned to **`8.0.30`**, re-verified at approval and moved as
   a set. `dotnet-ef` and `dotnet-sql-cache` are **separate tool pins** the runtime packages do not
   provide (§6.3, §7). The **six** test-tooling pins version independently of that band — with one
   exception, `Microsoft.Data.SqlClient` `5.1.9`, whose number is *derived* from the band's own SQL Server
   provider so that the fixtures hold the driver the application ships (§7.6). A **seventh** test-project
   pin sits **on** the band rather than beside them: `Microsoft.Extensions.Identity.Core` `8.0.30`, because
   the `ILookupNormalizer` the suite invokes must be the implementation the application's own Identity stack
   resolves, and the application's arrives transitively at the band's number (§7.8). **Three of the six lie
   outside the plan's own §0.5.2 pin set and are recorded as such**: the HTML5 parser `AngleSharp` `1.7.2`, because the suite asserts on rendered elements and no
   text scan can make that assertion (§7.5); that SQL client, because the fixtures attach, drop and read
   databases and no test-infrastructure package opens a connection (§7.6); and the browser harness
   `Microsoft.Playwright` `1.62.0`, for the one flow the application executes in script rather than serving
   as markup — one engine as functional evidence, with the browser matrix left to the manual review
   (§7.7).
6. **All 28 MVC 5 pins have exactly one outcome**, summing to 28 across six classes (§8.3).
   `Newtonsoft.Json` is removed as an **unused dependency** — not replaced as a serializer (§8.4).
7. **Four** browser libraries are retained and two are removed with no successor; the four are **vendored
   into `wwwroot`** and declared in `libman.json` by their cdnjs library ids — one of which differs from the
   library's npm name — selecting **thirteen files**, with the restored files committed so no build or
   deployment step fetches them (§9.2, §9.4, §9.4).
8. **No schema baseline exists in the repository.** Extraction plus a passing generated-schema diff is a
   precondition on the data work (§13).
9. **The target has four projects, and two of them are test projects for one structural reason:**
   `src/MvcMusicStore.Contracts.Tests` references **no target project** and is therefore buildable before the port,
   which is what makes the pre-port baseline workstream executable; `src/MvcMusicStore.Tests` references
   the web application **and** the contracts project. **The reference is not the sharing mechanism:** each
   contract surface is one `abstract` class carrying the fact methods, declared once in the contracts
   project, and **each runnable assembly declares its own `sealed` concretes deriving from it** — legacy-bound
   in the contracts project, target-bound in the in-process project — because a test method is discovered on
   a class the assembly under test declares, and a project reference alone enrols nothing (§12.3).
   **Every one of those types is `public`** — bases, concretes, fixtures, collection definitions and the two
   types that cross the boundary — because an internal base cannot be derived from another assembly and an
   internal test class is not discovered at all; and **each surface group has a `public` collection-definition
   class in each assembly**, binding that assembly's fixture through `ICollectionFixture<TFixture>`, with the
   matching `[Collection]` on every concrete taken from a constant rather than a literal (§12.3, and
   §12.1's plan-correction record 8 for the map rows). **For the
   same reason one level down, each runnable test project declares `xunit`, `xunit.runner.visualstudio` and
   `Microsoft.NET.Test.Sdk` — and `Microsoft.Playwright` — directly rather than inheriting them**: those
   deliver their function through build and analyzer assets, which do not cross a project reference, so a
   project that inherited them would compile and then be neither discovered nor executed by `dotnet test`
   (§12.3, §7.7). The three library pins are declared once and do cross it (§7.5, §7.6, §7.8). Each project
   carries its own `xunit.runner.json` with collection parallelism off, and both read the **thirteen committed
   fixture inputs** — the shared data manifest, the ten model-override files named one by one (the nine
   schema-divergence dimensions plus the narrowing case), the
   seeding oracle and the committed baseline reference — each **declared once**, in the contracts project,
   under the one fixture directory, and copied to each output (§12.2, and §12.1's record 7). One further
   committed artifact is an oracle rather than a fixture and lives in the web application instead: the
   resource file whose `CartMigrationNotice` entry the notice assertion reads, embedded rather than copied
   (§12.2). **A third fixture type sits beside the two hosted ones and hosts nothing**:
   the deployed-only fixture and its single concrete, which consume a base address, disable redirect
   following and provision no database, because the one deployed-edge case runs against a deployed instance
   and neither hosted fixture can produce that context (§12.2, §12.3, and §12.1's record 11).
10. **The one release-time operator console references the web application, composes it through one named
    seam, and is published separately.** All ten of its verbs must compile against the two `DbContext`
    types, `ApplicationUser` and the migration sets, so duplicating those types is what would let it drift —
    and because a reference supplies types but no registrations, the web project also declares
    `ApplicationServices.AddMvcMusicStoreServices`, which **`Program.cs` itself calls**, so the console builds
    a host, calls that one method and resolves what it needs rather than registering a second, drifting copy
    of the application. HTTP-pipeline registrations stay out of the seam. The web application references **no**
    console, so the console never ships inside it, and it is invoked in one canonical form by tests and by the
    release (§12.2, §12.9).

### 14.2 What this document creates: nothing

Every artifact named above is a specification. No tracked file was modified to write it, and none of the
four manifests of section 6 was brought into existence.

**Generated output is a separate claim, and its honest answer is not "none".** Restores and builds were
run against the three legacy editions while the assessment was being written, and they wrote eight
gitignored trees into the checkout: a restored `packages/` payload for two of the editions, and a
`bin`/`obj` pair for each of the three — 527 files, 114,310,394 bytes, enumerated and re-measured in
appendix A.6. **All eight are still in the checkout as this document is written**, gitignored and untracked;
removing them is a precondition on the checkpoint commit rather than something already done, and A.6 states
the current output of the two ignored-aware commands alongside the state acceptance requires.

Bare `git status --porcelain` never reported any of it, and that is the reason this section states the
generated output explicitly rather than resting on the status alone: `[Oo]bj/` [.gitignore:1], `[Bb]in/`
[.gitignore:2] and `Packages/` [.gitignore:33] are ignore rules matching every one of those eight paths, so
porcelain prints **not one line about them** — on the accepted checkpoint it prints nothing at all — while a
hundred megabytes of restored payload and compiled output sits in the tree. A tracked-file diff has the same
blind spot for the same reason.

The acceptance check is therefore four commands that have to hold together, not one:
`git status --porcelain` returns zero lines, `git status --porcelain --ignored` returns zero lines,
`git clean -ndX` prints nothing, and
`git diff --name-status ea2552d..HEAD` (§1.4) returns exactly thirteen `A`
rows, all under `docs/modernization/` — no `.cs`, `.cshtml`, `.csproj`, `.sln`, `.config`, `.sql`, `.js`,
`.css`, `.mdf` or `.ldf` file modified or deleted. All four, with their output, are in appendix A.6. The
fourth holds now and is quoted there from an actual run; the two ignored-aware ones do not hold yet, and
A.6 says so rather than claiming otherwise.

One consequence belongs here rather than in a sibling document, because the dependency transition is this
document's to own: the target's committed `NuGet.config`, its per-project `packages.lock.json` and its
locked-mode restore (§6.2, §6.4) exist partly so that what a restore pulls into a tree is knowable before
it runs and verifiable afterwards — the opposite of the situation recorded above.

### 14.3 Cross-reference index

Where each hand-off in this document lands:

| This document defers | To |
| --- | --- |
| Hosting target, deployment model, observability, key-ring location, the session cache table's schema and principal, the browser matrix; the release position and principal of the two migration bundles and of the operator console's data verbs; the conditional container image's **internals** — its stage list, non-root user, port contract and layers — and every required Azure resource including the registry instance (§12.9); the release-time halves of §13.3 and §13.4 — the environment pin at the moment a migration artifact is executed, the pre-DDL assertion of the target server and database, which stage downloads an artifact and under which principal, the order the two bundles are applied in, the exit-code gate and the traffic decision. **Not** deferred, and stated here instead: the artifact set and its names, the deterministic archive form, checksum generation, the release identity, and the verification, rollback-locator and retention rules the consumers are held to (§13.4) | [06 — Azure Hosting Recommendations](06-azure-hosting-recommendations.md) |
| Cutover approach; pipeline, DI, configuration, Identity, EF Core and migration design including the two contexts' history tables; the seed routine and the policy behind its three guard checks; the operator console's ten verb contracts, their idempotence, their choice of secret channel — of which this document names only that channel's environment variable (§12.9) — and the one tool-wide exit-code allocation they share; the 29 views and the Bootstrap markup work; asset relocation, the per-file disposition of the assessed source assets and the casing correction; the test suite's architecture and coverage; the fixture and host-wiring design behind §12.2's two test-project rows; the fixture lifecycle and parallelism requirement behind the two `xunit.runner.json` rows (§12.7 there); the name, signature and contents of the shared registration method the console reaches across its project reference (§12.9); the test-side invocation of the console (§12.9 here); and the clean-checkout execution runbook (§12.2's closing note) | [05 — ASP.NET Core Migration Approach](05-aspnet-core-migration-approach.md) |
| Per-edition build outcomes, toolchain and host prerequisites, the `.nuget` and stale-solution diagnoses, the views not being compile-checked by the checked-in build configuration | [10 — Build and Deployment Requirements](10-build-and-deployment-requirements.md) |
| Effort, bands, confidence; the risk register including the .NET 8 support window, the narrowed browser matrix and the absent regression baseline | [07 — Effort, Risks and Sequencing](07-effort-risks-sequencing.md) |
| Workstream decomposition and ordering; CI provider selection; when the band is re-verified; and when AppCAT runs — the **AppCAT static assessment** gate within **W2** | [03 — Modernization Roadmap](03-modernization-roadmap.md) |
| Current pin values and manifests, the committed restore client, the unconfigured restore source and its Technical Specification §3.3 correction, the absent lockfile, advisory posture | [02 — Dependency Inventory](02-dependency-inventory.md) |
| **Nothing is deferred, but four hand-offs run the other way and their outcomes differ — so they are listed with the outcome rather than as one row.** The **browser harness** (§7.7) is *refused by the plan itself*, so nothing is pinned, nothing is pending and this document is the single statement of that rejection. The **fixtures' SQL client** (§7.6) and the **`ILookupNormalizer`** the diagnostic pseudonym scheme invokes (§7.8) are *discharged inside approved scope*: both arrive across the test project's `ProjectReference` at the numbers the band resolves, with no pin, which is why §7.2 has a row for neither. The **HTML parser** (§7.5) is *not discharged*: it is outside the plan's frozen pin set, so it is recorded in §7.9 as **pending a separate AAP amendment — not approved scope**, and [05 §12.6](05-aspnet-core-migration-approach.md)'s parsed-DOM assertions stay blocked until the plan owner decides. In all four cases [05](05-aspnet-core-migration-approach.md) states that it names no version; in three of them that is now answered, and in the fourth the answer is that there is nothing approved to name | — |
| F-12-01 SQL Server Compact; F-12-02 bundling; F-12-03 `IAppBuilder`; F-12-09 the disabled external-login surface; F-12-12 assembly metadata; F-12-15 lazy loading; F-12-16 JSON naming; F-12-19 connection-string convention; F-12-21 and F-12-22 the schema gap | [12 — Migration Blockers](12-migration-blockers.md) |
| Debt severity and ownership: F-08-12 and F-08-24 the schema scripts, F-08-16 view compilation, F-08-18 the restore client and lockfile, F-08-23 the stale solution | [08 — Technical Debt Register](08-technical-debt-register.md) |
| Session statefulness, absent machine key, and the other cloud-readiness blockers behind §10 | [11 — Cloud Readiness Assessment](11-cloud-readiness-assessment.md) |

---

## Appendix A — Reproducibility

Every command is read-only. Run from the repository root. `grep -c` exits non-zero when it counts zero, so
a zero result is a valid answer rather than a failed command.

### A.1 The project-format facts of section 4

```bash
awk 'END{print NR}' src/MVC5/MvcMusicStore/MvcMusicStore.csproj          # -> 302
head -c 3 src/MVC5/MvcMusicStore/MvcMusicStore.csproj | od -An -tx1      # -> ef bb bf   (UTF-8 BOM)

grep -c '<Reference Include='   src/MVC5/MvcMusicStore/MvcMusicStore.csproj   # -> 46
grep -c '<HintPath>'            src/MVC5/MvcMusicStore/MvcMusicStore.csproj   # -> 26
                                                                              #    46 - 26 = 20 framework
                                                                              #    or machine references
grep -c '<Compile Include='     src/MVC5/MvcMusicStore/MvcMusicStore.csproj   # -> 27
grep -c '<Content Include='     src/MVC5/MvcMusicStore/MvcMusicStore.csproj   # -> 61
grep -c '<None Include='        src/MVC5/MvcMusicStore/MvcMusicStore.csproj   # -> 2
grep -c '<ProjectReference'     src/MVC5/MvcMusicStore/MvcMusicStore.csproj   # -> 0  (a leaf project)
```

The 12 assembly attributes absorbed by §5.3:

```bash
grep -c '^\[assembly:' src/MVC5/MvcMusicStore/Properties/AssemblyInfo.cs   # -> 12
```

### A.2 The 28 pins of section 8

```bash
grep -c '<package ' src/MVC5/MvcMusicStore/packages.config   # -> 28

# the identifiers themselves, in the manifest's order -- the checklist for the §8.2 table
sed -n 's/.*<package id="\([^"]*\)".*/\1/p' src/MVC5/MvcMusicStore/packages.config
```

Cross-check against the §8.3 tally: the class counts are 2 + 3 + 2 + 1 + 14 + 6 = 28.

**One pathspec per directory, deliberately.** A single braced pathspec —
`git ls-files 'src/MVC5/MvcMusicStore/{Content,Scripts,Images,fonts}/*'` — returns `0` on this checkout,
not 27, and does so silently: the quotes that protect the pathspec from the shell also suppress brace
expansion, and git's own pathspec matching does not implement braces. A count evidenced that way would
be unreproducible, so the four pathspecs are written out here and wherever this figure is cited.

### A.4 What the repository does not contain

Each of these is a target artifact that does not exist today, which is why §6, §9 and §12 mark them
net-new:

```bash
git ls-files | grep -c 'global.json'          # -> 0
git ls-files | grep -c 'libman.json'          # -> 0
git ls-files | grep -c 'packages.lock.json'   # -> 0
git ls-files | grep -ic 'test'                # -> 0   (no test project, no test file, anywhere)
git ls-files '.github/*' | wc -l              # -> 0
```

And the fact behind §5.7 — no project in any edition declares a user-secrets identifier, and no secret
store is tracked, so the property and the Development source it enables are entirely net-new:

```bash
git grep -ci 'UserSecretsId' -- 'src/*' | wc -l   # -> 0   (no .csproj declares one)
git ls-files | grep -c 'secrets.json'             # -> 0
```

The only tracked file named `NuGet.Config` is MVC 4's, and it configures no package source (§6.2):

```bash
git ls-files | grep -i 'nuget.config'
# -> src/MVC4/MvcMusicStore/.nuget/NuGet.Config
git ls-files 'src/MVC5/*' | grep -c '.nuget'   # -> 0   (MVC 5 declares the folder but has no file in it)
```

And the fact behind §10.3 — no shared key material in any of the 15 application configuration files:

```bash
git ls-files -- '*.config' '*.Config' | grep -v '/packages/' | grep -v '\.nuget/' | wc -l   # -> 15
git ls-files -- '*.config' '*.Config' | grep -v '/packages/' | grep -v '\.nuget/' \
  | xargs grep -lE '<machineKey' | wc -l                                                    # -> 0
```

### A.6 Solution and project inventory, and the constraint this work was held to

```bash
git ls-files '*.sln'    | wc -l   # -> 4   four legacy solutions for three legacy projects (§5.6)
git ls-files '*.csproj' | wc -l   # -> 3
```

The constraint itself takes four commands, because no single one of them sees the whole tree. The tracked
half is checked against the immutable evidence revision rather than against working-tree status, with the
range's two sides as §1.4 defines them:

```bash
git diff --name-status ea2552d..HEAD          # the range §1.4 defines
# -> A  docs/modernization/01-architecture-overview.md
#    A  docs/modernization/02-dependency-inventory.md
#    A  docs/modernization/03-modernization-roadmap.md
#    A  docs/modernization/04-dotnet8-migration-strategy.md
#    A  docs/modernization/05-aspnet-core-migration-approach.md
#    A  docs/modernization/06-azure-hosting-recommendations.md
#    A  docs/modernization/07-effort-risks-sequencing.md
#    A  docs/modernization/08-technical-debt-register.md
#    A  docs/modernization/09-security-assessment.md
#    A  docs/modernization/10-build-and-deployment-requirements.md
#    A  docs/modernization/11-cloud-readiness-assessment.md
#    A  docs/modernization/12-migration-blockers.md
#    A  docs/modernization/README.md
#    thirteen A rows, no M, D or R row of any kind

git status --porcelain
# -> on the accepted checkpoint: zero lines. While the deliverables are being written or revised it
#    lists them and nothing else -- untracked before the first commit, modified while they are being
#    revised -- which is the authoring-time form of the same evidence (§1.4). No path it prints in
#    either state lies outside docs/modernization/.
```

Neither of those two commands sees an ignored path, and there are ignored paths to see. Restores and builds
were run against the three legacy editions, and they wrote eight gitignored trees into this checkout:

| Tree written by the assessment's restores and builds | Files |
| --- | --- |
| `src/MVC3/MvcMusicStore-Completed/MvcMusicStore/bin` | 5 |
| `src/MVC3/MvcMusicStore-Completed/MvcMusicStore/obj` | 7 |
| `src/MVC4/MvcMusicStore/bin` | 51 |
| `src/MVC4/MvcMusicStore/obj` | 7 |
| `src/MVC4/packages` | 231 |
| `src/MVC5/MvcMusicStore/bin` | 52 |
| `src/MVC5/MvcMusicStore/obj` | 7 |
| `src/MVC5/packages` | 167 |

**527 files and 114,310,394 bytes**, of which the two restored `packages/` payloads are **398 files and
78,916,729 bytes** — and every one of those figures is re-derived from the checkout rather than remembered,
which is what makes it evidence rather than a recollection:

```bash
git status --porcelain --ignored | grep -c '^!!'     # -> 8    the eight trees tabulated above
git clean -ndX   # -n is dry-run and -X limits it to ignored files: it prints, it never removes
# -> Would remove src/MVC3/MvcMusicStore-Completed/MvcMusicStore/bin/   ... and the other seven

git ls-files --others --ignored --exclude-standard | wc -l                    # -> 527
git ls-files --others --ignored --exclude-standard -z | xargs -0 stat -c %s \
  | awk '{s+=$1} END {print s}'                                              # -> 114310394

git ls-files --others --ignored --exclude-standard \
  -- src/MVC4/packages src/MVC5/packages | wc -l                             # -> 398
git ls-files --others --ignored --exclude-standard -z \
  -- src/MVC4/packages src/MVC5/packages | xargs -0 stat -c %s \
  | awk '{s+=$1} END {print s}'                                              # ->  78916729
```

**Those last two commands are therefore an acceptance check on the checkpoint commit and not a statement
about this checkout.** As this document is written they are *not* satisfied: the ignored-aware status prints
eight `!!` lines and the dry-run clean names eight trees. The state acceptance requires is both printing
nothing, which is reached by removing the eight trees before the checkpoint is taken — a removal this
documentation work does not perform, because the trees are generated output shared with the other work
running against this checkout. **Asserting the absence instead would produce the one failure this appendix
exists to prevent**: a reader runs the command and sees the opposite.
**Nothing in the eight is tracked**, so the tracked-file half of the constraint is unaffected either way,
and that is the half §14.2 turns on.

The rules that kept the payload invisible to the first two commands are `[Oo]bj/` [.gitignore:1],
`[Bb]in/` [.gitignore:2] and `Packages/` [.gitignore:33]. The third of those is the load-bearing one and
the least obvious: a pattern whose only separator is trailing matches a directory of that name at **any**
depth, and on this checkout — `git config core.ignorecase` reports `true` — it also matches the lowercase
`packages` directories. `packages/*` [.gitignore:15] does *not* cover them, because a pattern with an
interior separator is anchored to the repository root. `git check-ignore -v` reports the rule per path:

```bash
git check-ignore -v src/MVC5/packages/x src/MVC5/MvcMusicStore/bin/x src/MVC5/MvcMusicStore/obj/x
# -> .gitignore:33:Packages/   src/MVC5/packages/x
#    .gitignore:2:[Bb]in/      src/MVC5/MvcMusicStore/bin/x
#    .gitignore:1:[Oo]bj/      src/MVC5/MvcMusicStore/obj/x
```

The footprint statement therefore has two halves, and neither stands without the other. Tracked:
thirteen additions under `docs/modernization/` and nothing else — no repository file modified to produce
this document, and none of the artifacts it specifies created. Ignored: restored package payload and build
output entered the checkout while the assessment was written and are still in it, gitignored and untracked,
with their extent measured by the commands above rather than estimated — and their removal before the
checkpoint commit is an acceptance requirement, stated as one, rather than a completed step reported as one.

### A.7 Secondary cross-reference

Technical Specification §3.3 is cited **only** through [02](02-dependency-inventory.md) §6, which records
the correction to it. This document makes no claim resting on that section, and under the citation
contract of §1.5 every as-is statement above resolves to a repository path and locator.
