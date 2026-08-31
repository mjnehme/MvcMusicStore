
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

And because the operator command's host is composed here, so is **that host's input surface**: its admitted
configuration sources, the closed dispatcher that selects the verb and normalizes its switches, the exit
codes, and the exact configuration key and environment variable the administrator credential arrives on
(§12.4). Those names are the ones every other document cites; what the command *does* with the inputs
remains [05 §10.2](05-aspnet-core-migration-approach.md)'s.

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
| `git diff --name-status ea2552d..28e3652` | **13 lines, every one `A` and every path under `docs/modernization/`** — no `M` and no `D` against any pre-existing file | **The durable evidence, and both endpoints are pinned commits.** `ea2552d` is the commit this deliverable set was authored from and `28e3652` is the commit that carries it. The whole of this work is thirteen added documents |

The second is the one that matters after the fact. **A `git status --porcelain` listing of new files
describes an uncommitted moment and cannot be reproduced once the work is committed** — at that point the
same command correctly prints nothing, and a document quoting the earlier listing as its current evidence
is quoting something a reader cannot re-run. **The diff is reproducible at any time from any checkout —
provided both of its endpoints are pinned commits.** An earlier form of this section wrote it as
`ea2552d..HEAD`; `HEAD` is whatever the reader's branch points at, so that form silently re-asks the
question on every later commit and its "thirteen additions" result expires the moment a fourteenth file
lands. Pinned as `ea2552d..28e3652` it answers one fixed question for good, which is what the
reproducing-command contract of §1.5 requires of a repository-wide claim.
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
than restate. All five target projects — the ported application, the **two** test projects, the operator
tool and the data-migration tool (§12.2) — target it; there is no
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
| `MvcBuildViews` property and its `AspNetCompiler` target | [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:17], [:274-276] | The property is `false` and the target is gated on `'true'`, so the target never runs. Razor compilation in the target is an SDK concern, configured by the SDK's own build-time and publish-time compilation rather than by an `AspNetCompiler` invocation |
| `$(VSToolsPath)\WebApplications\Microsoft.WebApplication.targets` import | [:272] | The Web SDK subsumes it. This import is also the reason the current project needs Visual Studio's web-application targets on the build host — a prerequisite [10](10-build-and-deployment-requirements.md) §4.2 owns |
| The `Condition="false"` v10.0 import | [:273] | Already inert; removed as dead configuration |
| `$(SolutionDir)\.nuget\NuGet.targets` import, `RestorePackages`, `SolutionDir` default | [:295], [:24], [:23] | Superseded by SDK-integrated restore (§5.5) |
| IIS Express settings and the `ProjectExtensions` web-project block | [:18-22], [:277-294] | **Dropped with no target *project* setting replacing them.** The target runs on Kestrel: its deployed binding is [06](06-azure-hosting-recommendations.md)'s, and its **local** endpoints are declared in `appsettings.Development.json` under `Kestrel:Endpoints` — an exact HTTP and an exact HTTPS URL, because the no-configuration default is HTTP-only. Section 12.5 of this document specifies them; section 12.4 records that no launch-profile file is specified |

**The honest consequence of dropping `MvcBuildViews`: nothing is lost, because nothing was there.**
Because the property is `false` [:17] and its target is gated on `'true'` [:274], `AspNetCompiler` has
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
file is a **single root `MvcMusicStore.sln`** referencing five new projects — the ported web application,
the **two** test projects, the operator tool and the data-migration tool (§12). It is an addition, not a replacement: no existing `.sln` is
edited, superseded or deleted, and after the port the repository tracks five solution files rather than
one.

**One of those five projects exists before the solution does, and the ordering is deliberate.** The
contracts test project of §12.2 is created, restored, built and run in [03](03-modernization-roadmap.md)'s
W4, at its governance-bootstrap gate 4a — before the ported web application exists and therefore before
this solution has anything else to
reference. Until the solution exists it is restored, built and tested **by project path**, which needs no
solution at all; adding it to the solution is part of creating the solution, not a separate step. Nothing
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
| `global.json` (§6.1) | [03](03-modernization-roadmap.md)'s **W4, gate 4a** — the first workstream that restores anything, and a restore against a pinned band needs the pin | **W6**, if the conversion requires an SDK property the bootstrap did not carry. Amended in place; never re-created |
| `NuGet.config` (§6.2) | [03](03-modernization-roadmap.md)'s **W4, gate 4a** — locked-mode restore against a *declared* source is a W4 exit criterion, so the declaration cannot arrive later | **W6**, if a source is added. Amended in place; never re-created, and the `<clear />` stays |
| `.config/dotnet-tools.json` (§6.3) | [03](03-modernization-roadmap.md)'s **W6** — nothing before the conversion invokes `dotnet ef` or `dotnet sql-cache` | **W10** consumes it; no later workstream adds a tool to it in this plan |
| `packages.lock.json`, per project (§6.4) | **The workstream that creates each project**: W4's gate 4a for the contracts test project, W6 for the converted web project, W7 for the in-process test project and the data-migration tool, W12 for the operator tool | Regenerated by any restore that legitimately changes the graph, and committed with that change |

**The rule behind the table, because it is the thing that was wrong before it was written down: a
workstream may not have an exit criterion that depends on a file a later workstream creates.** W4's exit
requires a locked-mode restore against a declared source, so the two root manifests and the first lockfile
belong to W4; W6 then **inherits and extends** the set rather than creating governance from nothing.
[03](03-modernization-roadmap.md)'s W4 and W6 state the same division from the sequencing side, and its
§6.1 carries the edge that connects them.

### 6.1 `global.json` — the SDK band

Root of the repository. Contents exactly as given in §3.1: `"version": "8.0.400"`,
`"rollForward": "latestPatch"`. Nothing else belongs in it — in particular, no MSBuild SDK entries, since
the project uses only `Microsoft.NET.Sdk.Web`.

**First created at [03](03-modernization-roadmap.md)'s W4 gate 4a**, because the band has to be pinned
before the first restore rather than after it: W4 restores the contracts test project, and a restore whose
SDK is whatever the host happens to carry is not the reproducible restore that workstream's exit gate
requires. **W6 inherits this file and may amend it in place** — an added MSBuild SDK entry, say, if the
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
own exit criterion rather than a preference: W4 must restore the contracts test project **in locked mode
against a declared source**, and a declared source that arrives two workstreams later is not one. **W6
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

Each project in the target — the web application, the **two** test projects, the operator tool and the
data-migration tool (§12.2) — commits a
`packages.lock.json`, and **CI restores in locked mode**, so that a change to the resolved graph fails the
build instead of arriving silently. Five projects, five lockfiles.

**They do not all arrive at once, and no workstream may claim the full set before the projects exist.**
Each lockfile is created by the workstream that creates its project, which makes the allocation exactly
this:

| Lockfile | Created in | Because that is when the project exists |
| --- | --- | --- |
| `src/MvcMusicStore.Contracts.Tests/packages.lock.json` | **W4, gate 4a** | The contracts test project is gate 4a's own output, and its locked-mode restore is that gate's exit criterion |
| `src/MvcMusicStore/packages.lock.json` | **W6** | The converted web project is W6's output |
| `src/MvcMusicStore.Tests/packages.lock.json` | **W7** | The in-process test project references the ported application, so it cannot exist earlier |
| `tools/migrate-data/packages.lock.json` | **W7** | Same reason — the tool references the web project's registration seam (§12.9) |
| `tools/provision-admin/packages.lock.json` | **W12** | The operator tool is W12's output |

W6's exit therefore accounts for the two lockfiles that exist at that point — the one it inherits from gate
4a and the one it creates — rather than for five, and each later workstream brings its own under the same
locked-mode restore. [03](03-modernization-roadmap.md)'s W4 and W6 exit gates state the same allocation
from the sequencing side.

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
2. **The set moves together.** If the band advances, all ten rows below advance to the same number in one
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
| nuget.org | `Microsoft.Extensions.Identity.Core` | `8.0.30` | **The Identity abstractions the suite resolves `ILookupNormalizer` from — a test-project pin the application does not declare.** [05 §12.9](05-aspnet-core-migration-approach.md) canonicalizes every diagnostic pseudonym by **invoking** that normalizer rather than describing it, and hands the version to this section; §7.8 records the closure, the type the suite uses and why the application needs no direct reference |
| nuget.org (tool) | `dotnet-ef` | `8.0.30` | The `dotnet ef` command — the Design package does not provide it (§6.3) |
| nuget.org (tool) | `dotnet-sql-cache` | `8.0.30` | Creates the session cache table — `Caching.SqlServer` is the runtime provider only (§6.3) |
| nuget.org | `xunit` | `2.9.2` | Test framework. **The package is flagged deprecated on nuget.org** — reason *Legacy*, suggested alternative `xunit.v3` — and §7.3 carries the approval gate that decision needs |
| nuget.org | `xunit.runner.visualstudio` | `2.8.2` | Test adapter. **Not deprecated**; listed [NuGet Gallery, *xunit.runner.visualstudio*, <https://www.nuget.org/packages/xunit.runner.visualstudio> — verified 2026-08-28] |
| nuget.org | `Microsoft.NET.Test.Sdk` | `17.11.1` | Test host and `dotnet test` integration. **Not deprecated**; listed [NuGet Gallery, *Microsoft.NET.Test.Sdk*, <https://www.nuget.org/packages/Microsoft.NET.Test.Sdk> — verified 2026-08-28] |
| nuget.org | `AngleSharp` | `1.7.2` | **HTML5/CSS-selector DOM parsing in the test projects; not referenced by the application.** Referenced by **both** test projects — declared in `src/MvcMusicStore.Contracts.Tests`, which owns the assertions that parse a response body, and reaching `src/MvcMusicStore.Tests` through its project reference to that project (§12.2). Why a parser is a pin rather than a helper method is §7.5's; the registry facts are `1.7.2` listed, **not deprecated**, licensed **MIT**, publishing `net8.0` and `netstandard2.0` among its target frameworks [NuGet Gallery, *AngleSharp*, <https://www.nuget.org/packages/AngleSharp> — verified 2026-08-29] |
| nuget.org | `Microsoft.Data.SqlClient` | `5.1.9` | **Direct SQL access for the fixtures' state observer and for the legacy stores' attach and detach lifecycle; the application reaches SQL only through the EF Core provider.** Declared in `src/MvcMusicStore.Contracts.Tests`, which owns the observer and the legacy fixture, and reaching `src/MvcMusicStore.Tests` through that project's reference (§12.2). The **version is not the registry head, and that is a decision rather than an oversight**: `5.1.9` is the version `Microsoft.EntityFrameworkCore.SqlServer` `8.0.30` itself resolves, so the fixtures hold the same driver the application ships — §7.6 states the choice, the alternative and why the alternative is refused. Registry facts: `5.1.9` listed, **not deprecated**, no reported vulnerability, licensed **MIT**, published 13 January 2026 [NuGet Gallery, *Microsoft.Data.SqlClient*, <https://www.nuget.org/packages/Microsoft.Data.SqlClient> — verified 2026-08-29] |
| nuget.org | `Microsoft.Extensions.Logging.ApplicationInsights` | `2.23.0` | **The single logging provider `tools/provision-admin` configures, so that its audit record has a destination** — [06 §9.5](06-azure-hosting-recommendations.md) owns the sink, the credential path and the retention, and [05 §10.2](05-aspnet-core-migration-approach.md) owns the record's shape. Declared in `tools/provision-admin/ProvisionAdmin.csproj` **and nowhere else**: the web application is instrumented by the platform without a package ([06 §9.1](06-azure-hosting-recommendations.md)), and this pin must not be read as reversing that decision. Listed, **not deprecated**, MIT, publishing `netstandard2.0`, which a `net8.0` project consumes [NuGet Gallery, *Microsoft.Extensions.Logging.ApplicationInsights*, <https://www.nuget.org/packages/Microsoft.Extensions.Logging.ApplicationInsights> — verified 2026-08-29] |
| nuget.org | `Microsoft.Playwright` | `1.62.0` | **The headless-browser harness for the one browser-executed flow of the suite — the cart page's script-issued removal request — driving Chromium.** Declared in `src/MvcMusicStore.Contracts.Tests` beside the other assertion-side pins and reaching `src/MvcMusicStore.Tests` through that project's reference (§12.2). **One engine is functional evidence and not a browser matrix:** the supported-browser matrix is established by the manual appearance review alone, per [05 §12.5](05-aspnet-core-migration-approach.md) and the plan's §0.11.2, and §7.7 records that boundary with the pin. Registry facts: `1.62.0` is the current **stable** release, listed, **not deprecated**, no reported vulnerability, licensed **MIT**, published 11 August 2026, publishing a `netstandard2.0` asset which a `net8.0` project consumes [NuGet Gallery, *Microsoft.Playwright*, <https://www.nuget.org/packages/Microsoft.Playwright> — verified 2026-08-29] |

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
failure would arrive at the moment the **first** test project is authored — `src/MvcMusicStore.Contracts.Tests`,
created in [03](03-modernization-roadmap.md)'s W4 at its governance-bootstrap gate 4a (§12.2, §6) — which is
the earliest point in the sequence at which anything is restored at all, and well
ahead of the port itself.

**The gate.** Before either test project is authored, the approver takes one of exactly two decisions:

| Option | What it means | Cost |
| --- | --- | --- |
| **Retain `xunit` `2.9.2`** | The frozen plan stands. The pin is documented as a knowingly accepted deprecated dependency, with a policy exception recorded if a dependency gate would otherwise block it | None technically. Requires the exception to exist wherever deprecation gates are enforced |
| **Supersede the plan to the maintained `xunit.v3` line** | AAP §0.5.2 is amended and the pin, the runner and both test projects' shape change with it | A plan amendment, re-verification of the runner and adapter pins at the time of approval, and no other part of this strategy changes — the application's own package set is untouched either way |

**Owner: engineering, jointly with the plan owner** — engineering because the test stack is theirs to run,
the plan owner because only they can amend a frozen pin. A third, smaller question rides along and should
be settled in the same decision: if v2 is retained, whether to take the one-patch move from `2.9.2` to
`2.9.3`, which is within the same deprecated line and therefore changes the maintenance position not at
all.

Nothing else in §7.2 is affected. The five other test-tooling pins were checked and none is flagged:
`xunit.runner.visualstudio` `2.8.2` and `Microsoft.NET.Test.Sdk` `17.11.1` — both checked at the same time
as `xunit` — `AngleSharp` `1.7.2`, checked when the pin was added (§7.5), and
`Microsoft.Data.SqlClient` `5.1.9` and `Microsoft.Playwright` `1.62.0`, checked when theirs were added
(§7.6, §7.7), are all listed and **not**
deprecated, so none joins this gate. All five nonetheless fall under the ordinary
re-verification of §7.1 consequence 1: the two runner pins have advanced since the plan was written, the
parser and harness pins sit on actively released lines whose current stable version at approval time may
not be the one recorded here, and the SQL client's number is whatever the then-current band's provider
resolves rather than a figure re-chosen independently (§7.6).

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

### 7.5 `AngleSharp` `1.7.2` — the HTML parser the suite needs, and why it is a pin rather than a helper

This is the one pin in §7.2 that the modernization plan's own pin set does not carry, so it is recorded
here with the reason it exists, the registry evidence behind the version, and the plan amendment it
implies.

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
`src/MvcMusicStore.Contracts.Tests`, because that project owns every assertion that parses a response body,
and it reaches `src/MvcMusicStore.Tests` transitively through that project's reference (§12.2) — which works
for this pin because it is a **library** package: its compile and runtime assets cross a project reference,
unlike the build and analyzer assets of the three test-execution pins, which do not and are therefore
declared in both projects (§12.3). **The web
application does not reference it**, and nothing in the application's own package set changes on account of
this pin: the parser exists to *read* the application's output, so a production reference would be a
dependency added for no runtime purpose.

**Plan-correction record 3 — the pin, recorded because the pin set is governed too.** Same form and same
reason as the two records in §12.1, applied to the plan's pin set rather than to its artifact map.

| Aspect | Statement |
| --- | --- |
| **The addition** | `AngleSharp` `1.7.2`, referenced by the two test projects only — the row added to §7.2 above |
| **The omission** | The plan's §0.5.2 successor-package set fixes the test stack as `Microsoft.AspNetCore.Mvc.Testing`, `xunit`, `xunit.runner.visualstudio` and `Microsoft.NET.Test.Sdk`. It carries **no HTML parser**, while §0.3.1 requires the suite to assert on *"selected rendered content"* and on the *"presence and value of named elements"*, and §0.11.2 makes that suite an acceptance criterion |
| **Why the omission cannot stand** | Those four packages supply a host, a framework, a runner and an adapter. **None of them parses HTML**, so an assertion about a rendered element written with only that set is a text scan — and the table above is why a text scan cannot make the assertion the plan requires. The gap is not a convenience; it is the difference between an assertion and the appearance of one |
| **Cause** | The plan's pin set was assembled from the test *infrastructure* the suite needs. The parser is a consequence of what the suite *asserts*, which §0.11.2 states in prose rather than as a dependency |
| **What is being asked** | That the plan's next revision add this pin to its §0.5.2 set, scoped to the test projects. **A plan amendment for the plan owner, not a strategy decision taken silently** |
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
in `src/MvcMusicStore.Contracts.Tests`, which owns the observer and the legacy fixture, and reaches
`src/MvcMusicStore.Tests` through that project's reference (§12.2) — the same arrangement as the parser pin
of §7.5, for the same reason, and legitimate for the same asset-flow reason: a library pin's compile and
runtime assets cross a project reference (§12.3).

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

**Consequence for §7.1's "the set moves together" rule.** When the band advances, this pin does not get
re-chosen — it is re-read: the number becomes whatever the new
`Microsoft.EntityFrameworkCore.SqlServer` resolves, and the parity property is what is preserved across
the move rather than the digits.

**Plan-correction record 4 — the pin, recorded because the pin set is governed.** Same form and same
reason as record 3 above.

| Aspect | Statement |
| --- | --- |
| **The addition** | `Microsoft.Data.SqlClient` `5.1.9`, referenced by the test projects only — the row added to §7.2 above |
| **The omission** | The plan's §0.5.2 successor-package set carries no SQL client for the test projects, while §0.3.1 requires a legacy reset that *"restores both committed `.mdf`/`.ldf` pairs, not one, and reattaches them before each run"* and requires the target fixture to provision a disposable engine, apply both migration sets and load a dataset *"with asserted row counts and key invariants"* |
| **Why the omission cannot stand** | Attach, detach, drop and a row-count read are statements executed against an engine. `Microsoft.AspNetCore.Mvc.Testing` supplies a host, `xunit` a framework, `AngleSharp` a parser: **none of them opens a connection.** Without a client the reset and the invariant assertions are prose, and the fixture the plan makes an acceptance criterion in §0.11.2 cannot be built |
| **Cause** | The plan's pin set was assembled for the *application*, plus the four test-infrastructure packages. The client is transitive for the application, so it needed no row there — and the fixtures' own need for it is stated in §0.3.1 as behaviour rather than as a dependency |
| **What is being asked** | That the plan's next revision add this pin to its §0.5.2 set, scoped to the test projects, with the derivation rule above rather than a fixed number. **A plan amendment for the plan owner, not a strategy decision taken silently** |
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
because the case that drives a real engine — [05 §12.9](05-aspnet-core-migration-approach.md) row 28 `28b`
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

**Plan-correction record 5 — the pin.** Same form as records 3 and 4.

| Aspect | Statement |
| --- | --- |
| **The addition** | `Microsoft.Playwright` `1.62.0`, referenced by the test projects only — the row added to §7.2 above |
| **The omission** | The plan's §0.5.2 set names no browser harness, and its §0.11.2 offers a manual review for **rendered appearance** only. The one flow above is neither appearance nor a request a client can hand-build faithfully: it is the application's own script deciding what to send |
| **Why the omission cannot stand** | Substituting a reviewer clicking the remove link would reclassify a **behavioural** gap as a visual one, which is the one substitution §0.11.2's manual allowance does not authorize. Without the harness the flow is covered statically — the token is rendered, the header name appears in the script — and every runtime failure mode ([05 §12.11](05-aspnet-core-migration-approach.md) enumerates them) passes that check while the control does nothing in a browser |
| **Cause** | The plan's coverage prose treats the suite as HTTP-level throughout, which it is for every surface but this one. The exception is a property of the application — one scripted flow — rather than of the test strategy |
| **What is being asked** | That the plan's next revision add this pin to its §0.5.2 set, scoped to the test projects, and record that browser-executed coverage is **one engine for behaviour** while the browser matrix remains the manual review's. **A plan amendment for the plan owner, not a strategy decision taken silently** |
| **Disposition** | **Recorded, and the pin is published rather than deferred.** [05 §12.11](05-aspnet-core-migration-approach.md) states explicitly that it names no package because this document owns the test projects' dependency set; publishing the pin is how that hand-off is discharged. The scope decision it serves stays with the plan owner and [07](07-effort-risks-sequencing.md) re-estimates the workstreams it lands in |

---

### 7.8 `Microsoft.Extensions.Identity.Core` `8.0.30` — the package that closes the normalizer dependency

This pin exists because another deliverable's design **invokes** a type, and a type that is invoked has to
come from a package something declares. Deliverable [05 §12.9](05-aspnet-core-migration-approach.md)
canonicalizes every cross-run diagnostic pseudonym by resolving **`ILookupNormalizer`** and calling it,
explicitly rather than reimplementing its behaviour, and it states in the same row that it names no version
because *this* section owns the pin. Before this row that hand-off was unclosed: the contracts test project
invoked the abstraction and declared nothing that provides it, which is not a version disagreement but an
open dependency — the project would not compile.

**Why the invocation and not a description of it, which is what makes the pin non-optional.** 05's own row
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
`src/MvcMusicStore.Contracts.Tests`, which owns the diagnostic record and the pseudonym scheme, and it
reaches `src/MvcMusicStore.Tests` through that project's reference — legitimately, because this is a
**library** pin whose compile and runtime assets cross a project reference (§12.3), unlike the
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

**Plan-correction record 9 — the pin.** Same form as records 3, 4 and 5.

| Aspect | Statement |
| --- | --- |
| **The addition** | `Microsoft.Extensions.Identity.Core` `8.0.30`, referenced by the test projects only — the row added to §7.2 above |
| **The omission** | The plan's §0.5.2 successor set names `Microsoft.AspNetCore.Identity.EntityFrameworkCore` for the **application** and no Identity package for the test projects, and its §0.11.2 requires the suite's diagnostics without specifying how a per-account identifier is canonicalized. The abstraction the design settled on is therefore invoked by a project whose declared package set does not contain it |
| **Why the omission cannot stand** | An invoked type with no declaring package is a project that does not compile — the one failure class this document exists to enumerate rather than discover during implementation. The alternative that needs no pin is worse and is the one 05 rejected with evidence: reimplementing the normalization merges or splits accounts relative to the application's own rule, and the resulting per-account mismatch is attributable to neither runtime |
| **Cause** | The plan's pin set was derived from the **application's** runtime needs, and the suite's needs were derived from its *test-tooling* needs. A pin that is an application-framework package used **by the tests** falls between the two derivations, which is the same gap that produced records 3, 4 and 5 |
| **What is being asked** | That the plan's next revision add this pin to its §0.5.2 set at the band's number, scoped to the test projects, and record that the suite invokes `ILookupNormalizer` with the framework's `UpperInvariantLookupNormalizer` rather than substituting a casing rule of its own. **A plan amendment for the plan owner, not a strategy decision taken silently** |
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
| `jQuery.Validation` `1.11.1` | Update, vendored | **jquery-validate `1.21.0`** — the cdnjs library id is `jquery-validate`, not `jquery-validation`; see §9.7 |
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
  (`git ls-files 'src/MVC5/MvcMusicStore/{Content,Scripts,Images,fonts}/*'` → 27, appendix A.3) does not
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
  libraries** — the four `libraries` entries of the manifest in §9.6, being the four that survive; and
  **five acquired files** — the manifest's total file selection, because the Bootstrap entry selects two
  files and the other three select one each. Six dispositions, four libraries, five files.
  [05](05-aspnet-core-migration-approach.md) §8.1.1 adds the five to the application's own relocated
  files to state the served payload.
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

### 9.5 The manifest, in full

A mechanism named but not written is not specified: the provider's library identifiers are not the NuGet
identifiers, the file names differ per library, and the choice of Bootstrap script bundle decides whether
one of the application's live controls works. The manifest is therefore given in full, as the target-state
content of `src/MvcMusicStore/libman.json` (§12.2):

```json
{
  "version": "1.0",
  "defaultProvider": "cdnjs",
  "libraries": [
    {
      "library": "jquery@3.7.1",
      "destination": "wwwroot/lib/jquery/",
      "files": [ "jquery.min.js" ]
    },
    {
      "library": "jquery-validate@1.21.0",
      "destination": "wwwroot/lib/jquery-validation/",
      "files": [ "jquery.validate.min.js" ]
    },
    {
      "library": "jquery-validation-unobtrusive@4.0.0",
      "destination": "wwwroot/lib/jquery-validation-unobtrusive/",
      "files": [ "jquery.validate.unobtrusive.min.js" ]
    },
    {
      "library": "bootstrap@5.3.3",
      "destination": "wwwroot/lib/bootstrap/",
      "files": [
        "css/bootstrap.min.css",
        "js/bootstrap.bundle.min.js"
      ]
    }
  ]
}
```

Five properties of that manifest are decisions rather than formatting, and each would be got wrong by a
reader working from §9.2's version table alone:

- **The provider's identifier for Bootstrap is `bootstrap`.** `twitter-bootstrap` is the provider's
  **legacy alias** for the same library and still resolves; `bootstrap` is the current identifier and is
  what the manifest uses. A manifest naming the alias is not wrong, but it will confuse the next reader
  into thinking two libraries are in play.
- **The identifier for the validation plugin is `jquery-validate`, not `jquery-validation`.** The provider's
  identifier and the library's own name differ here — the only row where they do — which is exactly the
  kind of detail that turns a restore into a `LIB002` "library could not be resolved" error.
  `jquery-validation` is the library's **upstream and npm** name and is the name every other section of
  this document and of [05](05-aspnet-core-migration-approach.md) uses for it; `jquery-validate` is the
  **CDNJS** identifier and appears only where a provider identifier is required, which is this manifest.
  Neither name is wrong — they belong to different registries — so the rule this document follows is to
  say which registry a name belongs to at every point it names the library. **The destination is still
  `wwwroot/lib/jquery-validation/`**, so the served path reads naturally and does not inherit the
  provider's spelling; that deliberate difference is why §9.6 and §12.2 quote `jquery-validation` in the
  paths while this entry spells `jquery-validate` in the identifier.
- **`bootstrap.bundle.min.js` is selected, not `bootstrap.min.js`, because the bundle includes Popper and
  the plain file does not.** This is not a size preference: the application has a **live dropdown** — the
  genre menu at [src/MVC5/MvcMusicStore/Views/Store/GenreMenu.cshtml:3-18] — and Bootstrap 5 positions
  dropdown menus with Popper. Selecting `bootstrap.min.js` produces a page that renders correctly and a
  genre menu that does not open, with a console error as the only evidence. It is the same silent-failure
  shape as the data-attribute rename that [05](05-aspnet-core-migration-approach.md) §8.5.3 records, and
  the two would be diagnosed together.
- **No source maps are selected**, and the decision is recorded because it is visible in the served
  payload. A `.map` file is only useful alongside the unminified source it references, so including
  `jquery.min.map` or `bootstrap.min.css.map` without also committing the unminified originals buys
  nothing while adding files to the deployment. Browsers request a map **only when developer tools are
  open**, so its absence costs a console note and nothing else, and a developer who needs to step through
  a library can point the manifest at the unminified file for the duration. **The manifest therefore
  produces exactly five files**, which is the figure
  [05](05-aspnet-core-migration-approach.md) §8.1.1 uses when it states the served payload. Five, not
  four, and the arithmetic is worth writing out because the entry count and the file count differ: the
  three jQuery-family entries select **one file each** (3), and the Bootstrap entry selects **two** — the
  stylesheet and the script bundle (2). `3 + 2 = 5`, from **four** `libraries` entries.
- **`files` is explicit on every entry rather than omitted.** Omitting it restores the provider's whole
  library — for Bootstrap that is every distribution variant, every RTL stylesheet and every map — and
  those files would then be **committed**, because §9.3 commits the output. Explicit selection is what
  keeps a committed vendored directory reviewable.

**One property of the selected stylesheet is invisible in this manifest and must not be "fixed" by adding
files.** `css/bootstrap.min.css` carries its icon-like assets **inside the stylesheet**, as
`data:image/svg+xml` background images rather than as separate files. Two of them are adopted by the
port: the navbar toggler's glyph, which the library exposes as a custom property consumed by
`.navbar-toggler-icon`, and the checked and indeterminate marks of `.form-check-input` — both mapped in
[05](05-aspnet-core-migration-approach.md) §8.5.2 and §8.5.3. **No `files` entry acquires them, and none
is needed**, so this manifest stays at four `libraries` entries and **five** files and the served payload
figure does not move. What they do require is an image content-security-policy that admits `data:`, which
[06](06-azure-hosting-recommendations.md) §10.2 owns and decides; the alternative — self-hosting the two
glyphs as files under `wwwroot/images/` and overriding the library's own values — would add entries here
and is rejected there, not here.

**The manifest, reconciled against the provider and against the served paths.** Four entries, five files,
and every string checkable in one place — the provider identifier the entry must spell, the destination it
lands in, the file it selects, and the URL the browser then requests. Each `files` selection exists in the
pinned version at the provider, and each destination is the one §9.6's load order and §12.2's committed-file
list quote:

| Provider identifier and version | Destination | `files` | Served path |
| --- | --- | --- | --- |
| `jquery@3.7.1` | `wwwroot/lib/jquery/` | `jquery.min.js` | `/lib/jquery/jquery.min.js` |
| `jquery-validate@1.21.0` — library **jquery-validation** | `wwwroot/lib/jquery-validation/` | `jquery.validate.min.js` | `/lib/jquery-validation/jquery.validate.min.js` |
| `jquery-validation-unobtrusive@4.0.0` | `wwwroot/lib/jquery-validation-unobtrusive/` | `jquery.validate.unobtrusive.min.js` | `/lib/jquery-validation-unobtrusive/jquery.validate.unobtrusive.min.js` |
| `bootstrap@5.3.3` | `wwwroot/lib/bootstrap/` | `css/bootstrap.min.css`, `js/bootstrap.bundle.min.js` | `/lib/bootstrap/css/bootstrap.min.css`, `/lib/bootstrap/js/bootstrap.bundle.min.js` |

The served path is the destination with `wwwroot` removed, because `wwwroot` is the web root and never
appears in a URL — the distinction [05](05-aspnet-core-migration-approach.md) §12.7 asserts on. Row two is
the only row where the identifier and the destination folder differ, deliberately and for the reason given
above; every other row spells the library the same way in its identifier, its destination and its URL.

**Because the output is committed, a provider failure cannot break a build or a deployment.** A `LIB002`
resolution error can only occur during a developer-initiated `libman restore`, which happens when a
version in this manifest changes — never in CI and never at deployment (§9.3). That is the whole point of
committing the restored files, and it is why an unavailable CDN is an inconvenience here rather than an
outage.

### 9.6 The load order the manifest produces

The manifest fixes *what* is acquired and *where* it lands; the order the files load in is the
consequence, and it is a **correctness** property because three of the four have a hard dependency on
another. [05](05-aspnet-core-migration-approach.md) §8.1.3 renders these as tag-helper elements and owns
their placement in the views; this table is the order it renders them in.

| Position | Files, in order |
| --- | --- |
| **`<head>`** | `wwwroot/lib/bootstrap/css/bootstrap.min.css`, then the application's own `wwwroot/css/site.css` |
| **End of `<body>`, in the layout** | `wwwroot/lib/jquery/jquery.min.js`, then `wwwroot/lib/bootstrap/js/bootstrap.bundle.min.js`, then the page's scripts section |
| **Inside the page's scripts section** | On the form pages: `wwwroot/lib/jquery-validation/jquery.validate.min.js` — jquery-validation, acquired under the CDNJS identifier `jquery-validate` while the destination folder keeps the library's own name (§9.6) — then `wwwroot/lib/jquery-validation-unobtrusive/jquery.validate.unobtrusive.min.js`, whose identifier and folder are the same string. On the cart page: `wwwroot/js/shopping-cart.js` |

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
else. Each row below was checked against the library's cdnjs page, and the `files` lists are the minimum
set the application actually consumes rather than the whole distribution.

| cdnjs library id | Version | Destination | `files` to restore |
| --- | --- | --- | --- |
| `jquery` | `3.7.1` | `wwwroot/lib/jquery/` | `jquery.js`, `jquery.min.js`, `jquery.min.map` |
| `jquery-validate` | `1.21.0` | `wwwroot/lib/jquery-validation/` | `jquery.validate.js`, `jquery.validate.min.js`, `additional-methods.js`, `additional-methods.min.js` |
| `jquery-validation-unobtrusive` | `4.0.0` | `wwwroot/lib/jquery-validation-unobtrusive/` | `jquery.validate.unobtrusive.js`, `jquery.validate.unobtrusive.min.js` |
| `bootstrap` | `5.3.3` | `wwwroot/lib/bootstrap/` | `css/bootstrap.css`, `css/bootstrap.min.css`, `js/bootstrap.bundle.js`, `js/bootstrap.bundle.min.js` |

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

### 9.4 The `libman.json` contract — provider ids, destinations and file lists

A version alone does not make a library acquirable. Library Manager resolves each entry as
`<provider-library-id>@<version>`, and **the provider's id is not always the package's familiar name**, so
an entry that names the library the way the NuGet package or the npm package names it can fail to resolve
even though the version is correct. That is a restore-time failure, not a build-time one, so it surfaces
as a developer or reviewer being unable to obtain the assets rather than as a compiler error. The full
manifest is therefore specified here rather than left to the version table above.

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

**The four details in it that are decisions rather than transcription.**

- **`jquery-validate`, not `jquery-validation`.** cdnjs publishes jQuery Validation under the id
  `jquery-validate`; there is no cdnjs library called `jquery-validation`, so an entry naming it resolves
  to nothing. Verified directly against the provider's own index rather than inferred:
  `https://api.cdnjs.com/libraries/jquery-validate/1.21.0` returns the library with the four files listed
  above, and `https://api.cdnjs.com/libraries/jquery-validation` returns HTTP 404. The **unobtrusive**
  adapter is the opposite case — its cdnjs id *is* `jquery-validation-unobtrusive`, which is why the two
  neighbouring entries spell the library name two different ways and why neither can be derived from the
  other by pattern.
- **Every entry states its `destination` explicitly, and one of them differs from its library id.**
  Omitting `destination` makes LibMan derive the folder from the library id, which would place jQuery
  Validation at `wwwroot/lib/jquery-validate/` — a path that matches neither the ASP.NET Core project
  convention nor the folder the views' script references use. The destinations above are the contract:
  `jquery-validation/` for the `jquery-validate` library, and a folder matching the id for the other
  three. The view-side script references that consume these paths are
  [05](05-aspnet-core-migration-approach.md)'s to write; this document fixes the paths they must target.
- **Every entry states its `files` explicitly**, because the default is to vendor the library's *entire*
  published file set — for `bootstrap@5.3.3` that is over a hundred files, including RTL builds,
  unminified builds, per-component ESM bundles and their maps, all of which would be committed under the
  rule in §9.3. The listing rule is: a file is vendored only if it is served, or if it is a source map for
  a file that is served. Bootstrap's minified CSS and its `bundle` script (Bootstrap plus Popper, so no
  separate Popper entry is required) satisfy every use the views have; the unminified jQuery and
  validation builds are included because they are the variants a Development-time script reference uses,
  and their omission would break the Development branch while leaving production working.
- **jQuery's source map is `jquery.min.map`, not `jquery.min.js.map`.** cdnjs publishes it under the
  former name at this version. A file list copied from another project's manifest — where the
  `.min.js.map` convention is near-universal — fails to restore on exactly this one line.

**How a reviewer checks the manifest without running it.** Each entry's id and version resolve at
`https://api.cdnjs.com/libraries/<id>/<version>`, and the response's `files` array must contain every name
in that entry's `files` list. All four entries above were checked that way; `Respond` and `Modernizr` have
no entry because §9.2 removes them.

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

**Two things follow, and both are checkable.** The web application's `PackageReference` count in §7.3
drops from seven to **six** — the number stated there. And the retirement moves this document back into
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
transformation map in the Agent Action Plan (§0.4.1); every row below resolves to a row of it, or — in the
single case of the per-project lockfile — to the dependency-pinning requirement the plan states in §0.5.2,
or — in the **seven** cases recorded as plan corrections immediately below — to obligations the plan states
in §0.3.1 and §0.3.2 and makes acceptance criteria in §0.11.2.
No row adds an artifact the plan does not require, because expanding the target artifact set is a plan
amendment rather than a strategy decision. That rule is unchanged and it governs every other row in §12.2;
the seven exceptions are named, argued and notified rather than taken quietly.
The practical consequence is worth stating, because it is the
kind of file a reader expects to see listed and will not find: **local run configuration — launch
profiles, developer ports, the environment a developer's own `dotnet run` picks up — is a developer
concern, not part of the mapped artifact set.** It is created, changed and discarded per workstation, it
is not a build input, it is not a deployment input, and nothing in this strategy depends on its contents.
The IIS Express settings it would loosely correspond to are dropped outright (§5.4), and the deployed
hosting model is [06](06-azure-hosting-recommendations.md)'s.

**Plan-correction records — the seven artifact sets this map adds to the plan's, and why each is a correction
rather than an invention.** Recorded in the form deliverable
[05](05-aspnet-core-migration-approach.md) §5.6 and §9.3 use, and for the same reason: the plan is frozen,
nothing here edits it, and an artifact the plan's own obligations require but its map does not list has to
be **notified**, once and by name, rather than appearing in an implementation as an unexplained addition to
a governed set.

**Plan-correction record 1 — the data-migration executable.**

| Aspect | Statement |
| --- | --- |
| **The artifact** | `tools/migrate-data/MigrateData.csproj`, `tools/migrate-data/Program.cs` and that project's `packages.lock.json` — the row added to §12.2 below |
| **The omission** | The plan's §0.4.1 map enumerates every target artifact and lists **no data-migration executable**. `tools/provision-admin` is mapped, `src/MvcMusicStore.Tests` is mapped, and `Data/Migrations/Catalog/**` and `Data/Migrations/Identity/**` are mapped — but a migration set creates **empty tables** and moves no rows, which is the point [05 §5.1](05-aspnet-core-migration-approach.md) opens with |
| **Why the omission cannot stand** | Two consequences, both fatal to an acceptance criterion the plan itself sets. **A gate no artifact owns is unenforceable:** §0.3.2's rule that the generated-schema diff *must pass before any data is loaded* is a rule about the order of two operations, and with nothing owning both sides it reduces to a person remembering it. **And a migration performed by hand is not the artifact the tests exercised:** §0.11.2 makes the schema-and-data migration an acceptance criterion and [05 §12.9](05-aspnet-core-migration-approach.md) requires the suite to prove its eight checks, and a suite can only invoke something that exists. A pre-populated fixture would prove the assertions run and nothing whatever about the migration |
| **Cause** | The map was written from the *application's* file inventory, where a release-time console tool that is not deployed with the application does not naturally appear. `tools/provision-admin` is in the map because §0.3.2 names it as a **file**; the migration is named there as a sequence of **steps**, so no file was derived from it |
| **What is being asked** | That the plan's next revision add the three paths above to its §0.4.1 map, as CREATE / net-new, alongside `tools/provision-admin`. **This is a plan amendment for the plan owner, not a strategy decision taken silently** |
| **Disposition** | **Recorded, and the row is added rather than deferred.** [05 §5.7](05-aspnet-core-migration-approach.md) specifies the artifact in full and its own plan-correction record asks this document to map it, which is the notification that authorizes the row; §12 is where the target artifact set is mapped, so this is where it belongs. Nothing else in this document changes: the governance rule above still refuses every other unmapped artifact |

**Plan-correction record 2 — the second test project and the two runner configuration files.**

| Aspect | Statement |
| --- | --- |
| **The artifacts** | `src/MvcMusicStore.Contracts.Tests/MvcMusicStore.Contracts.Tests.csproj`, that project's `packages.lock.json`, and **one `xunit.runner.json` per test project** — `src/MvcMusicStore.Contracts.Tests/xunit.runner.json` and `src/MvcMusicStore.Tests/xunit.runner.json`. The four rows added to §12.2 below |
| **The omission** | The plan's §0.4.1 map lists **one** test project — `src/MvcMusicStore.Tests/MvcMusicStore.Tests.csproj` "and its test classes" — and no runner configuration file. §0.3.1 describes that one project as holding both `LegacyBaselineFixture` and `CoreApplicationFixture`, so on the plan's own map the legacy-facing fixture lives in the project that must reference the ported web application |
| **Why the omission cannot stand** | Two independent consequences. **A single project cannot be built when it is first needed:** §0.3.1 requires test authoring to *precede* the port and §0.11.2 makes the pre-port behavioural baseline an acceptance criterion, but a project referencing `src/MvcMusicStore/MvcMusicStore.csproj` cannot compile before that project exists — so the workstream that captures the baseline would be gated on the port it exists to protect. Splitting the HTTP-only half into its own reference-free project is what makes the plan's own ordering executable, and the split costs nothing structurally because §0.3.1 already requires the two fixtures to *share* the contract assertions rather than each owning a copy. **And an assembly-level runner setting is a file:** §0.11.2 requires isolation strong enough that no case sees another's data, and the mechanism [05 §12.7](05-aspnet-core-migration-approach.md) selects for disabling collection parallelism is a runner configuration file per test assembly, which the map does not carry for either project |
| **Cause** | The map named the test *project* and its classes, at the granularity of "there is a test project". The **buildability** of that project against the sequence in §0.3.1, and the runner configuration a shared-database fixture needs, are properties one level below that granularity, so neither produced a file on the map |
| **What is being asked** | That the plan's next revision replace its single test-project row with the **two** projects above, each with its lockfile and its `xunit.runner.json`, record that the contracts project carries **no** reference to the web application, and record that **each of the two projects declares the test-execution pins of §0.5.2 — `xunit`, `xunit.runner.visualstudio` and `Microsoft.NET.Test.Sdk` — directly**, because those pins' build and analyzer assets do not cross a project reference and a project that inherited them would not be discovered or executed at all (§12.3). No pin is added by that clause; it fixes the declaration site of pins the plan already carries. **A plan amendment for the plan owner, not a strategy decision taken silently** |
| **Disposition** | **Recorded, and the rows are added rather than deferred.** The reference-free project is the only arrangement in which [03](03-modernization-roadmap.md)'s W4 gate is closable, and W4's gate is the plan's own §0.11.2 criterion; deferring the rows would leave the sequence unexecutable while appearing compliant. The governance rule above is unchanged and still refuses every other unmapped artifact |

**Plan-correction record 6 — the application's service-registration seam.**

| Aspect | Statement |
| --- | --- |
| **The artifact** | `src/MvcMusicStore/ApplicationServices.cs`, declaring `public static class ApplicationServices` and its one `AddMvcMusicStoreServices` method — the row added to §12.2 below |
| **The omission** | The plan's §0.4.1 map gives the web project a `Program.cs` that *"absorbs 6 startup files"*, and separately maps `tools/provision-admin` with the note that it *"resolves `UserManager`/`RoleManager` from the container"*. **No artifact in the map lets a second process reach that container.** A composition root written as top-level statements exposes nothing callable; the one public type the arrangement produces exists for `WebApplicationFactory`'s entry-point lookup and declares no members |
| **Why the omission cannot stand** | The plan requires **two** console tools to resolve application services — §0.3.2 for the operator command's `UserManager`/`RoleManager` and its explicit statement that *"direct SQL is explicitly not an acceptable substitute, because it cannot produce a valid Identity password hash"*, and §0.3.2 again for the deployment-time migration step. With no seam each tool must write its own registrations, and duplicated registrations are the one form of duplication that **compiles and then drifts in silence** (§12.9): the tool keeps exiting `0` after the application changes a connection-string key, a provider option or a hasher setting, and writes data the application reads under different assumptions. The plan's own anti-duplication reasoning for the *types* applies with more force to the registrations, and the map carries no artifact that discharges it |
| **Cause** | The map was derived from the legacy file inventory, where composition lives in `Global.asax.cs` and the OWIN startup files and is reachable only from inside the web process [src/MVC5/MvcMusicStore/Global.asax.cs:13-21], [src/MVC5/MvcMusicStore/Startup.cs:4-14]. There is no legacy file to map a *callable* registration surface from, so none appeared — the requirement arrives with the tools, which are net-new |
| **What is being asked** | That the plan's next revision add the path above to its §0.4.1 map as CREATE / net-new, owned by the web project, and record that `Program.cs` composes the application **through** it so that there is one registration path rather than one per process. **A plan amendment for the plan owner, not a strategy decision taken silently** |
| **Disposition** | **Recorded, and the row is added rather than deferred.** Deferring it would leave two mapped tool projects with a stated dependency on a container no mapped artifact exposes — a map that looks complete and describes something that cannot be built. The seam's contents are constrained rather than invented: the policy values are [05 §6.1](05-aspnet-core-migration-approach.md)'s and the contexts' configuration [05 §4.5](05-aspnet-core-migration-approach.md)'s, and this document adds only the file, the type, the signature and the single-path rule |

**Plan-correction record 7 — the twelve committed fixture inputs: the shared manifest, the nine schema-divergence overrides, the seeding oracle and the baseline hand-off record.**

| Aspect | Statement |
| --- | --- |
| **The artifacts** | Twelve files, all under `src/MvcMusicStore.Contracts.Tests/Fixtures/` — `fixture-data.json`; the nine `ModelOverrides/` files named in §12.2's row; `seed-expected.json`; and `baseline-reference.json`. The four rows added to §12.2 below carry them |
| **The omission** | The plan's §0.4.1 map lists the test project's code files and **no data file of any kind**. Its §0.3.1 nonetheless requires the target fixture to load *"a fixture dataset carrying the same catalog rows plus a small set of migrated users, roles, carts and orders with asserted row counts and key invariants"*, and requires the legacy half to characterize the **same** application state; its §0.3.2 requires a **generated-schema diff that must pass before any data is loaded**, and §0.6.4 requires the seeding guard to be exercised. Each of those obligations is discharged by a run that **reads a committed file**: a dataset, a diverged target schema, a seeding expectation, or the values one run hands the next |
| **Why the omission cannot stand** | Four consequences, one per artifact group. **"The same catalog rows" is a property of one artifact, not of two loaders:** if each half carries its dataset in its own code, the two datasets drift and a cross-baseline comparison of two applications holding different data establishes nothing — which is the whole value the suite exists for. **"Asserted row counts" require published numbers:** an invariant assertion needs a figure that exists outside the loader that produced the rows, or it is the loader checking its own arithmetic. **A schema-diff gate can only be shown to refuse if something diverges the target schema on purpose**, and the switch that does it ([05 §5.7](05-aspnet-core-migration-approach.md)) takes a **path**, so nine committed inputs are the mechanism — without them the nine cases have nothing to point at and the gate is asserted rather than exercised, which is the one failure mode §0.3.2 says must be caught before data moves. **And a seeding expectation and a cross-run hand-off cannot be computed by the run that consumes them:** an oracle the code under test derives proves only self-consistency, and the target-side run cannot recompute a legacy commit or a normalizer identity it never observed |
| **Cause** | The map was written at the granularity of projects and classes. A committed data file — read by a loader, passed to a tool on a command line, or handed from one run to the next — is a fixture *input* rather than a compiled member, so no row was derived from any of the twelve. The same granularity gap produced record 2's runner-configuration files |
| **What is being asked** | That the plan's next revision add all twelve paths above to its §0.4.1 map as CREATE / net-new, each **declared once** in the contracts test project and copied to both test assemblies' output, and that it record the fixture directory as the single home for committed test inputs so no artifact acquires a second path. **A plan amendment for the plan owner, not a strategy decision taken silently** |
| **Disposition** | **Recorded, and the four rows are added rather than deferred.** [05](05-aspnet-core-migration-approach.md) specifies the contents in full — §12.3 the dataset's entities, counts, keys, accounts and fingerprint; §5.1 the nine divergence dimensions; §12.9 the seeding expectation; §12.10 the hand-off record's fields and digest — and specified contents with no file to live in are not loadable. **This map owns each file's existence, its one path and its copy behaviour; 05 owns every value inside every one of them**, and the two documents state the fixture directory identically so the path is fixed once |

**Plan-correction record 8 — the collection-definition classes that bind the mapped fixtures to the tests.**

| Aspect | Statement |
| --- | --- |
| **The artifacts** | `src/MvcMusicStore.Contracts.Tests/Collections/*.cs` and `src/MvcMusicStore.Tests/Collections/*.cs` — one `public` collection-definition class per surface group in each assembly, the two rows added to §12.2 below |
| **The omission** | The plan's §0.4.1 map lists `LegacyBaselineFixture.cs` and `CoreApplicationFixture.cs` and **nothing that attaches either to a test class**. §0.3.1 nonetheless requires “test isolation … per-class against a freshly provisioned database”, and §0.11.2 makes isolation an acceptance criterion; [05 §12.7](05-aspnet-core-migration-approach.md) implements both as **one collection fixture per surface group, each owning its own database** |
| **Why the omission cannot stand** | A fixture class with no collection definition naming it is never instantiated by the runner: the type exists, nothing binds it, and the failure surfaces inside an assertion as a missing context rather than as a wiring error. The grouping is equally unimplementable without it — a test class carrying no `[Collection]` **is its own collection**, so the mapped fixture would be constructed once per class and provision one database per class, which is neither the plan's “per-class against a freshly provisioned database” read as a *shared* engine budget nor 05's nine-group arrangement, and nothing reports the difference. Two sets are needed rather than one because `[Collection]` names are resolved within the assembly being run |
| **Cause** | The plan's map was written at the granularity of the fixture types themselves. The runner-level artifact that *binds* a fixture to the classes that use it is one level below that granularity — the same gap that produced records 2 and 7 |
| **What is being asked** | That the plan's next revision add the two paths above to its §0.4.1 map as CREATE / net-new, and record that each collection-definition class is `public`, carries `ICollectionFixture<TFixture>` for its own assembly's fixture, and exposes its name as a constant the group's concretes reference. **A plan amendment for the plan owner, not a strategy decision taken silently** |
| **Disposition** | **Recorded, and the rows are added rather than deferred.** [05 §12.7](05-aspnet-core-migration-approach.md) specifies the group set, the one-database-per-collection rule and the per-test reset in full; a specified collection with no definition to declare it is not runnable. This map owns the classes' existence, paths, accessibility and per-assembly duplication; 05 owns what the fixture they bind does |

**Plan-correction record 10 — the application resource file one entry of which is a test oracle.**

| Aspect | Statement |
| --- | --- |
| **The artifact** | `src/MvcMusicStore/Resources/ApplicationMessages.resx` — the row added to §12.2 below. Unlike records 2, 7 and 8 this one is a file in the **web application**, not in a test project |
| **The omission** | The plan's §0.4.1 map carries no resource file, and the legacy application has none — every user-facing string is a literal in a view or a `ModelState` message. Its §0.6.3 nonetheless requires that a cart migration failing after retries *"still complete the sign-in and surface a non-blocking notice"*, and §0.11.2 makes that failure path a required test. [05 §12.9](05-aspnet-core-migration-approach.md) row 37 discharges the second half by asserting the rendered notice is byte-equal to the value of a **named** entry, `CartMigrationNotice`, read from the application rather than transcribed into the assertion |
| **Why the omission cannot stand** | A named entry needs a file to be named in, and the alternative that needs no file is the one that makes the test worthless: a literal in the view and the same literal retyped in the assertion is a test of the transcription, and it passes after the notice has been edited in the view alone. Reading the value from the application means the assertion cannot drift from what the user sees, which is exactly the property §0.6.3's *"non-blocking notice"* has to keep. It also has to be **one** file: a second resource holding the same key reintroduces the drift the arrangement removes |
| **Cause** | The plan's map was derived from the legacy application's file set plus the constructs the port must add, and the legacy application has no resource file. This artifact is required not by the port itself but by a **test oracle** — the port would run with the string inline — so it falls outside both derivations, the same seam that produced record 9's pin |
| **What is being asked** | That the plan's next revision add the path above to its §0.4.1 map as CREATE / net-new, as an `EmbeddedResource` in the web project, and record that user-facing strings a test asserts on live there rather than as view literals. **A plan amendment for the plan owner, not a strategy decision taken silently** |
| **Disposition** | **Recorded, and the row is added rather than deferred.** The row's four predicates and the entry's value are [05 §12.9](05-aspnet-core-migration-approach.md)'s and [05 §4.3](05-aspnet-core-migration-approach.md)'s; this map owns the file's existence, its path, its build action and the single-home rule. The scope is deliberately narrow: **this record adds one file and no resource strategy** — localization is not in the plan and is not proposed here |

**Plan-correction record 11 — the deployed-only fixture, its concrete class and its collection definition.**

| Aspect | Statement |
| --- | --- |
| **The artifacts** | `src/MvcMusicStore.Contracts.Tests/DeployedEndpointFixture.cs`, `src/MvcMusicStore.Contracts.Tests/DeployedEndpointTests.cs`, and the deployed collection definition in that project's `Collections/` folder — the two rows added to §12.2 below, and the widened `Collections/*.cs` row |
| **The omission** | The plan's §0.4.1 map lists **two** fixtures — `LegacyBaselineFixture.cs` and `CoreApplicationFixture.cs` — and §0.3.1 describes both as hosting an application and provisioning a database. Its §0.11.2 nonetheless requires *"a deployment health check"* among the validated behaviours, and §0.3.2 requires HTTPS enforcement and HSTS at the edge; [05 §12.9](05-aspnet-core-migration-approach.md) row 47 discharges those against a **deployed** instance |
| **Why the omission cannot stand** | Neither mapped fixture can run that case, and neither should be made to. One starts the legacy application, the other hosts the port in process; **both provision a database**, and a deployment smoke check that provisioned a database would be asserting about something other than the deployment. Nor can the case borrow a fixture and ignore it: a fixture attaches by collection, so borrowing one means paying its provisioning on every deployment verification and giving a deployed-edge assertion a live database it has no business holding. The alternative of leaving the case unmapped is worse in the specific way this section exists to prevent — a required assertion with **no runnable context** reads as covered in the matrix and executes nowhere |
| **Cause** | The plan derived its fixtures from the two things being compared, which is the right basis for every case that compares them. Row 47 compares nothing: it observes one deployed edge, so it needs a context that is neither baseline nor in-process, and no such context follows from the plan's derivation |
| **What is being asked** | That the plan's next revision add the paths above to its §0.4.1 map as CREATE / net-new in the contracts test project, and record that this fixture **provisions nothing and hosts nothing** — it consumes a base address and disables redirect following. **A plan amendment for the plan owner, not a strategy decision taken silently** |
| **Disposition** | **Recorded, and the rows are added rather than deferred.** [05 §12.6](05-aspnet-core-migration-approach.md) specifies the binding's behaviour and [05 §12.10](05-aspnet-core-migration-approach.md) the stage that invokes it; this map owns the two files' existence, their paths, their accessibility, their project and the fact that they add **no project and no pin**. It also records the two facts a mapped-but-unrunnable class would lack: [06 §12.1](06-azure-hosting-recommendations.md)'s non-traffic verification stage runs it, and [03](03-modernization-roadmap.md)'s W7 authors it under a gate that requires it to compile and be discoverable without running it |

These seven records are the only amendments this document requests to the plan's **§0.4.1 artifact map**, and
none of them creates a file: none of the artifacts named in any of the four is created by this assessment.
Three other hand-offs to the plan owner exist and are deliberately kept out of this map, because none is an
artifact: §7.5, §7.6 and §7.7 each request one addition to the plan's **§0.5.2 pin set** — the HTML parser,
the fixtures' SQL client and the browser harness — and §7.3 hands over one approval
decision on a pin the plan already carries.

### 12.2 The map

| Target file | Transformation | Source | What this document specifies about it |
| --- | --- | --- | --- |
| `MvcMusicStore.sln` | CREATE | `src/MVC5/MvcMusicStore.sln`, consolidating four solutions | Single root solution referencing three projects: the web application, the test project and the operator tool (§5.6). The two project references between them are §12.4 |
| `global.json` | CREATE | **net-new** | SDK band `8.0.400`, `rollForward: latestPatch` (§3, §6.1) |
| `NuGet.config` | CREATE | **net-new** | `<clear />` then nuget.org (§6.2) |
| `.config/dotnet-tools.json` | CREATE | **net-new** | `dotnet-ef` `8.0.30`, `dotnet-sql-cache` `8.0.30` (§6.3) |
| `src/MvcMusicStore/MvcMusicStore.csproj` | CREATE | `src/MVC5/MvcMusicStore/MvcMusicStore.csproj`, `src/MVC5/MvcMusicStore/packages.config`, `src/MVC5/MvcMusicStore/Properties/AssemblyInfo.cs` | SDK-style `Microsoft.NET.Sdk.Web`, `net8.0`, `PackageReference` at the §7.2 pins, absorbed assembly metadata (§5.3), implicit globbing, `MvcBuildViews` and the WebApplication targets import dropped (§5.4) |
| `src/MvcMusicStore/packages.lock.json`, `src/MvcMusicStore.Tests/packages.lock.json`, `tools/provision-admin/packages.lock.json` | CREATE | **net-new** (each generated, then committed) | **Three lockfiles, one per project, each named exactly** — because §6.4 requires one per project and there are three projects (§12.4), so a single row naming only the web application's would leave two committed build inputs unmapped. Each carries its own project's locked transitive graph and **CI restores in locked mode** (§6.4). The other two projects' rows below say "its own committed lockfile"; **these are those files**, and they are named here so that the paths exist on the map rather than only as a phrase inside another row |
| `src/MvcMusicStore/libman.json` | CREATE | `src/MVC5/MvcMusicStore/packages.config` — the six content-delivering pins it replaces | `defaultProvider` `cdnjs`; the four retained libraries at the §9.2 versions |
| `src/MvcMusicStore/wwwroot/lib/**` | CREATE | the vendored output of `libman.json` | Committed, so no build or deployment step fetches them (§9.3). The *relocation* of the application's own 27 assets and the casing correction are [05](05-aspnet-core-migration-approach.md)'s |
| `src/MvcMusicStore.Tests/MvcMusicStore.Tests.csproj`, and its test classes — `LegacyBaselineFixture.cs`, `CoreApplicationFixture.cs`, and the per-surface contract classes under `src/MvcMusicStore.Tests/Contracts/**`, of which **two are named here because this document's own requirements create them**: `Contracts/OperatorHostContract.cs` and `Contracts/OperatorDispatcherContract.cs` | CREATE | **net-new** — the repository contains no test project and no test of any kind (appendix A.4) | SDK-style, `net8.0`, referencing `Microsoft.AspNetCore.Mvc.Testing` `8.0.30`, `xunit` `2.9.2`, `xunit.runner.visualstudio` `2.8.2` and `Microsoft.NET.Test.Sdk` `17.11.1`, plus its own committed lockfile and **two `ProjectReference` edges — to the web application and to the operator tool** (§12.4). The test classes carry no project item of their own — implicit globbing compiles them (§5.1). **`Contracts/**` is a bounded glob and §12.1 fixes its membership rule**; the two files named above are the ones §12.4's six required assertions land in — the host-composition assertions in the first, the dispatcher's admitted-and-refused surface in the second — and they are named because a requirement this document states with no file to hold it is the unowned target §12.1 forbids. The suite's **architecture and coverage** are [05](05-aspnet-core-migration-approach.md)'s and its place in the sequence is [03](03-modernization-roadmap.md)'s |
| `tools/provision-admin/ProvisionAdmin.csproj` and `tools/provision-admin/Program.cs` | CREATE | `src/MVC5/MvcMusicStore/App_Start/Startup.App.cs` — the startup provisioning it replaces — and `src/MVC5/MvcMusicStore/Models/SampleData.cs:9`, the startup-registered seed whose *invocation* it replaces | SDK-style `net8.0` console project with its own committed lockfile and its own **defaults-disabled** host (§12.4), referencing the web application for **four types** (§12.4), **not deployed with the web application**. It carries **two operator verbs** under that one host: administrator provisioning, and the guarded catalog seed [05](05-aspnet-core-migration-approach.md) §5.4 requires a command for (§12.6). Each verb's *behaviour* — the secret channel, the idempotence, the seed's three guard checks — is [05](05-aspnet-core-migration-approach.md)'s, and the audit sink is [06](06-azure-hosting-recommendations.md)'s |
| `src/MvcMusicStore/Program.cs` | CREATE | `src/MVC5/MvcMusicStore/Global.asax.cs`, `src/MVC5/MvcMusicStore/Startup.cs`, `src/MVC5/MvcMusicStore/App_Start/RouteConfig.cs`, `src/MVC5/MvcMusicStore/App_Start/FilterConfig.cs`, `src/MVC5/MvcMusicStore/App_Start/Startup.App.cs`, `src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs` | **Six startup files collapse into one composition root.** Structurally: compiled by implicit globbing rather than by a `Compile` item (§5.1), and it carries the `public partial class Program` declaration §12.4 requires so the test fixture can name the entry point. What the composition root *does*, registration by registration, is [05 §2](05-aspnet-core-migration-approach.md)'s |
| `src/MvcMusicStore/appsettings.json`, `src/MvcMusicStore/appsettings.Development.json` | CREATE | `src/MVC5/MvcMusicStore/Web.config`, `src/MVC5/MvcMusicStore/Web.Debug.config` | **The connection string and the application's settings move out of XML and into JSON read through `IConfiguration`; the administrator credential [src/MVC5/MvcMusicStore/Web.config:17] is not carried over.** Structurally: content files the Web SDK includes in build and publish output with no explicit item, which is why the conversion drops the `.config` items without adding replacements (§5.1). The keys, their sources and the precedence between them are [05 §3](05-aspnet-core-migration-approach.md)'s |
| `src/MvcMusicStore/Controllers/HomeController.cs`, `StoreController.cs`, `ShoppingCartController.cs`, `CheckoutController.cs`, `StoreManagerController.cs` | CREATE | the same five files under `src/MVC5/MvcMusicStore/Controllers/` | **Ported rather than rewritten:** namespace substitution, constructor injection in place of the field-initialized contexts, `HttpNotFound()` becoming `NotFound()`, explicit loading where EF 6 lazy-loaded, the two `[ChildActionOnly]` actions extracted to view components [src/MVC5/MvcMusicStore/Controllers/StoreController.cs:43], [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:86], and `AddToCart` becoming a token-protected POST. Five source files become five target files, none of them itemized in the project (§5.1). Every transition named here is [05](05-aspnet-core-migration-approach.md)'s to specify |
| `src/MvcMusicStore/Controllers/AccountController.cs` | CREATE | `src/MVC5/MvcMusicStore/Controllers/AccountController.cs` | **Rewritten, not ported** — against ASP.NET Core Identity, with the private `ChallengeResult : HttpUnauthorizedResult` [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:394] and its `ExecuteResult(ControllerContext)` override [:411] removed or replaced by the framework's challenge flow. Which of those two, and the fate of the external *sign-in* surface, is [05 §8.3](05-aspnet-core-migration-approach.md)'s choice; what this map settles is that the external-login **management** surface survives, because the `RemoveAccountList` component below is on it |
| `src/MvcMusicStore/Models/Album.cs`, `Artist.cs`, `Genre.cs`, `Cart.cs`, `Order.cs`, `OrderDetail.cs` | CREATE | the same six files under `src/MVC5/MvcMusicStore/Models/` | **The six entity classes port largely as they stand**, with one exception that is not a namespace change: `Order.cs` loses `using System.Web.Mvc` [src/MVC5/MvcMusicStore/Models/Order.cs:4] **and** the class-level `[Bind(Include = …)]` [:8] that directive exists to support, whose replacement is the `Binding/` row below. Explicit EF Core mapping wherever EF 6 relied on convention is [05 §4](05-aspnet-core-migration-approach.md)'s |
| `src/MvcMusicStore/Models/ApplicationUser.cs` | CREATE | `src/MVC5/MvcMusicStore/Models/IdentityModels.cs` — the `ApplicationUser : IdentityUser` declaration at [src/MVC5/MvcMusicStore/Models/IdentityModels.cs:6] | **The `IdentityUser` base moves to `Microsoft.AspNetCore.Identity`.** One legacy file becomes two target files — this one and `Data/ApplicationDbContext.cs` below — which is why that source appears twice on this map; the split is stated rather than left to be inferred |
| `src/MvcMusicStore/Models/AccountViewModels.cs` | CREATE | `src/MVC5/MvcMusicStore/Models/AccountViewModels.cs` | **Adapted to ASP.NET Core Identity's model shapes, not copied.** The file carries no legacy `using` directive at all — its only import is `System.ComponentModel.DataAnnotations` [src/MVC5/MvcMusicStore/Models/AccountViewModels.cs:1] — so an import-rewriting pass would leave it untouched and wrong. The per-model shape is [05](05-aspnet-core-migration-approach.md)'s |
| `src/MvcMusicStore/Binding/CheckoutInputModel.cs`, `src/MvcMusicStore/Binding/AlbumCreateInputModel.cs`, `src/MvcMusicStore/Binding/AlbumEditInputModel.cs` | CREATE | `src/MVC5/MvcMusicStore/Models/Order.cs`, `src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs` for the first; `src/MVC5/MvcMusicStore/Models/Album.cs` and `src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs` for the other two | **Explicit input models replace the class-level `[Bind]` and the synchronous `TryUpdateModel(order)` [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:29], which has no ASP.NET Core counterpart — and the folder holds three files, not one.** `CheckoutInputModel.cs` is the authoritative map's own group; the two administration models are the same pattern applied to the one other action pair that binds an entity directly, which is why they are named here rather than left inside a plural mention: the create POST binds `Album` [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:53] and the edit POST binds it and then marks a detached instance `Modified` [:91], so each action needs its own model and the two differ in exactly the key property. Three files, three named sources, one folder. Every property list, and the create/edit asymmetry over `AlbumId`, are [05 §8.11](05-aspnet-core-migration-approach.md)'s |
| `src/MvcMusicStore/ViewModels/ShoppingCartViewModel.cs`, `src/MvcMusicStore/ViewModels/ShoppingCartRemoveViewModel.cs` | CREATE | the same two files under `src/MVC5/MvcMusicStore/ViewModels/` | **Ported, with one addition:** the removal model gains explicit JSON property names, because the AJAX contract's PascalCase field names do not survive the target serializer's web defaults. The annotation and the decision to scope it to this one model rather than to the serializer policy are [05 §8.7](05-aspnet-core-migration-approach.md)'s |
| `src/MvcMusicStore/Services/ShoppingCartService.cs` | CREATE | `src/MVC5/MvcMusicStore/Models/ShoppingCart.cs` | **The cart, order-creation and cart-migration logic leaves the model layer for a service:** `HttpContextBase` becomes `HttpContext`, and the unreferenced `GetCart(MusicStoreEntities, Controller)` overload [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:29] is dropped rather than ported, since its `Controller` parameter type has no target equivalent. Its registration and its internals are [05 §4](05-aspnet-core-migration-approach.md)'s |
| `src/MvcMusicStore/Data/MusicStoreEntities.cs`, `src/MvcMusicStore/Data/ApplicationDbContext.cs` | CREATE | `src/MVC5/MvcMusicStore/Models/MusicStoreEntities.cs`, `src/MVC5/MvcMusicStore/Models/IdentityModels.cs` | **Both contexts gain explicit `DbContextOptions` constructors**, because EF Core honours neither of the two conventions in use today: the class-name-to-connection-string match behind the constructor-less `MusicStoreEntities` [src/MVC5/MvcMusicStore/Models/MusicStoreEntities.cs:5] and the `base("DefaultConnection")` call in `ApplicationDbContext` [src/MVC5/MvcMusicStore/Models/IdentityModels.cs:13]. Structurally, both compile into the web project — §12.5. Whether they stay separate, and how each is registered, are [05 §4.5](05-aspnet-core-migration-approach.md)'s |
| `src/MvcMusicStore/Data/Migrations/Catalog/**`, `src/MvcMusicStore/Data/Migrations/Identity/**` | CREATE | the MVC 5 schema **as extracted from the committed databases** — `src/MVC5/MvcMusicStore/App_Data/MvcMusicStore.mdf` and `src/MVC5/MvcMusicStore/App_Data/aspnet-MvcMusicStore-20131025034205.mdf` — and **not** from either `MvcMusicStore-Create.sql` copy (§13.1) | **Initial migrations generated from the extracted schema and diff-verified before any data is loaded (§13.2).** Structurally: two folders, one assembly, no `MigrationsAssembly` setting and no fourth project (§12.5). The extraction gate itself and the migration design are [05 §5](05-aspnet-core-migration-approach.md)'s |
| `src/MvcMusicStore/Data/SeedData.cs` | CREATE | `src/MVC5/MvcMusicStore/Models/SampleData.cs` | **The 826-LF (827 content lines) seed — the metric named per [01 §2.4](01-architecture-overview.md), since the file carries no terminal newline and this is the physical-line count rather than the non-blank sizing one — becomes a routine an explicit opt-in verb invokes, and `DropCreateDatabaseIfModelChanges` [src/MVC5/MvcMusicStore/Models/SampleData.cs:9] is not reproduced in any form.** Structurally, the executable that invokes it is `tools/provision-admin`'s second verb (§12.6) rather than a fourth project. The routine's content and its three fail-closed guard checks are [05 §5.4](05-aspnet-core-migration-approach.md)'s |
| `src/MvcMusicStore/ErrorViewModel.cs` | CREATE | **net-new** | **Replaces `System.Web.Mvc.HandleErrorInfo`**, the removed type the current error view declares as its model [src/MVC5/MvcMusicStore/Views/Shared/Error.cshtml:1]. No framework type takes its place — exception-handling middleware supplies no Razor model — so the port defines one. Its members and the error contract around it are [05 §8.3](05-aspnet-core-migration-approach.md)'s |
| `src/MvcMusicStore/ViewComponents/GenreMenuViewComponent.cs` and `src/MvcMusicStore/Views/Shared/Components/GenreMenu/Default.cshtml` | CREATE | `src/MVC5/MvcMusicStore/Controllers/StoreController.cs:43` — the `[ChildActionOnly]` genre-menu action — and its call site `@Html.Action("GenreMenu", "Store")` [src/MVC5/MvcMusicStore/Views/Shared/_Layout.cshtml:25] | **A child action becomes a view component**, because `[ChildActionOnly]` and `@Html.Action` have no ASP.NET Core counterpart. Two target files per component, in the two conventional locations. The component's arguments, its query and any caching are [05 §8](05-aspnet-core-migration-approach.md)'s |
| `src/MvcMusicStore/ViewComponents/CartSummaryViewComponent.cs` and `src/MvcMusicStore/Views/Shared/Components/CartSummary/Default.cshtml` | CREATE | `src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:86` — the `[ChildActionOnly]` cart-summary action — and its call site `@Html.Action("CartSummary", "ShoppingCart")` [src/MVC5/MvcMusicStore/Views/Shared/_Layout.cshtml:26] | **The same transformation, for the layout's second child action.** Both of these render on every page because both call sites are in the shared layout, which is why they are two rows rather than one line of prose |
| `src/MvcMusicStore/ViewComponents/RemoveAccountListViewComponent.cs` and `src/MvcMusicStore/Views/Shared/Components/RemoveAccountList/Default.cshtml` | CREATE | `src/MVC5/MvcMusicStore/Controllers/AccountController.cs:316` — the third `[ChildActionOnly]` action — its call site `@Html.Action("RemoveAccountList")` [src/MVC5/MvcMusicStore/Views/Account/Manage.cshtml:22], and the partial it renders, whose model is `ICollection<Microsoft.AspNet.Identity.UserLoginInfo>` [src/MVC5/MvcMusicStore/Views/Account/_RemoveAccountPartial.cshtml:1] | **The third of the three child actions, and it is on this map for the same reason as the other two.** MVC 5 declares `[ChildActionOnly]` exactly three times and all three become components, so the external-login **management** surface is retained in the target and the `UserLoginInfo` collection moves under this component. The choice [05](05-aspnet-core-migration-approach.md) holds over `ChallengeResult` (the `AccountController.cs` row above) concerns the external *challenge* flow, **not** whether this component exists |
| `src/MvcMusicStore/Views/**/*.cshtml` **excluding `Views/Shared/Components/**`, which the three view-component rows above own**, and including `src/MvcMusicStore/Views/_ViewImports.cshtml` and `Views/_ViewStart.cshtml` | CREATE | `src/MVC5/MvcMusicStore/Views/**/*.cshtml` — 29 tracked files — and `src/MVC5/MvcMusicStore/Views/Web.config` | **The views port and `Views/Web.config` does not:** `_ViewImports.cshtml` takes over its Razor namespace registration [src/MVC5/MvcMusicStore/Views/Web.config:5-23], and its `BlockViewHandler` mapping [:31-32] ends with the IIS integrated pipeline that gave it meaning. Structurally: Razor is compiled by the Web SDK at build and at publish, so these views become compile-checked for the first time (§5.4). **Two different inventories of the same 29 views are in play and this row uses both, because a document that quotes one number as if it were the other has miscounted the work.** (a) **Six source views name a removed API or type** — `Shared/Error.cshtml`, `Shared/_LoginPartial.cshtml`, `Account/Manage.cshtml`, `Account/_ChangePasswordPartial.cshtml`, `Account/_RemoveAccountPartial.cshtml` and `Account/_ExternalLoginsListPartial.cshtml`, the sixth being the one a namespace-only search misses because its removed members are OWIN rather than Identity: `@using Microsoft.Owin.Security` [src/MVC5/MvcMusicStore/Views/Account/_ExternalLoginsListPartial.cshtml:1] and `Context.GetOwinContext()…` [:6]. (b) **The target work partitions as `0 + 3 + 5 + 21 = 29`** — three become a view component's `Default.cshtml` (their own rows above), five need per-line work, twenty-one port with mechanical changes only, and none is deleted. The two units differ by exactly one file: `_RemoveAccountPartial.cshtml` is in the six *and* is counted as a component view in the partition, which is why the per-line figure is five rather than six. The **five-type** list is also the narrower one the authoritative future application map names, so both figures are legitimate and the defect is using either as the other. [05 §8.4](05-aspnet-core-migration-approach.md) owns the partition and [05 §8.3](05-aspnet-core-migration-approach.md) the per-view work; the bundling-helper call sites and the Bootstrap markup port are [05 §8](05-aspnet-core-migration-approach.md)'s |
| `src/MvcMusicStore/wwwroot/**` — the application's own assets, **excluding `wwwroot/lib/**` and `wwwroot/js/shopping-cart.js`, each of which is its own row** | CREATE | `src/MVC5/MvcMusicStore/Content/**`, `src/MVC5/MvcMusicStore/Scripts/**`, `src/MVC5/MvcMusicStore/Images/**`, `src/MVC5/MvcMusicStore/fonts/**` and `src/MVC5/MvcMusicStore/favicon.ico` | **The 27 asset files (appendix A.3) plus the root favicon move under a web root and are served by static-file middleware, with their path casing corrected on the way.** The two exclusions are stated in the target cell rather than implied, because `wwwroot/**` as written would otherwise claim files two other rows already own — one target file with two owning rows is the same defect as a target file with none. Structurally: no `Content` item group survives the conversion, because the SDK serves `wwwroot` without per-file items (§5.1). The per-file relocation is [05 §8](05-aspnet-core-migration-approach.md)'s and the repository-wide casing audit is [06](06-azure-hosting-recommendations.md)'s |
| `src/MvcMusicStore/wwwroot/js/shopping-cart.js` | CREATE | `src/MVC5/MvcMusicStore/Views/ShoppingCart/Index.cshtml:7-35` — the view's inline `<script>` block, whose `$.post` at [:17] is the cart-removal call | **A script that is markup today becomes a served file, and it is a row rather than a member of the `wwwroot/**` row above because its source is a Razor view rather than an asset file.** Two independent requirements land on it: [06 §10.2](06-azure-hosting-recommendations.md) makes the inline block a policy violation under `script-src 'self'` with no `unsafe-inline` and no nonce, and [05 §7.4](05-aspnet-core-migration-approach.md) makes the removal request carry the anti-forgery token in a `RequestVerificationToken` **header**, so this file is the client half of that contract. Structurally: one file under the web root, served by the shared framework's static-file middleware with no bundler and no build step (§10.1), referenced by a `<script src>` in the same document so the token in the page remains reachable. Its contents — the selector, the read-at-post-time rule and the header literal — are [05 §7.4](05-aspnet-core-migration-approach.md)'s |
| CI pipeline definition | CREATE | **net-new** | **Deliberately not named as a file.** Its path and format depend on a provider choice this assessment does not make; [03](03-modernization-roadmap.md) carries provider selection as an explicit roadmap gate. What this document contributes to it: the build image must satisfy the `8.0.400` band (§3.2), restore runs in locked mode (§6.4), and `dotnet tool restore` precedes any migration step (§6.3) |
| `Dockerfile` | CREATE | **net-new** | **Conditional.** It exists only under the container-native hosting option, which is [06](06-azure-hosting-recommendations.md)'s to select. If code deployment is chosen, this file does not exist at all |
| `deploy/sql/SecurityAuditLog.sql` | CREATE | **net-new** — no schema script in the repository creates any table this target needs; the two MVC 4 copies of `MvcMusicStore-Create.sql` are disqualified as baselines by §13.1 | **The reviewed DDL script that creates `dbo.SecurityAuditLog`, and it is a repository file with no project item.** [06 §9.5](06-azure-hosting-recommendations.md) states the table's columns, constraints, `CHECK`, unique key and index **in full** and [06 §6.3](06-azure-hosting-recommendations.md) step 1 requires it applied **before** either migration set, so its content is settled and only its location was not — which is what put it on this map. It **cannot** be a migration: it belongs to no context's model, no migration may alter it, and §6.3 step 4b grants on it. The placement rule, applied here for the first time and stated so it is predictable: **operator SQL that belongs to no project and is deployed with nothing lives under a top-level `deploy/sql/` folder**, alongside the root-level `global.json` and `NuGet.config` in being a repository file rather than a project item, so `dotnet publish` of the web project cannot carry it and no `Compile` or `Content` item claims it. **`dbo.SessionCache` needs no script** — `dotnet sql-cache create` creates it (§6.3) — and **`dbo.DataProtectionKeys` needs none either**, being a migration inside the Identity set ([06 §6.3](06-azure-hosting-recommendations.md) step 4), so this folder holds one file and its membership is not a glob |
| The **audit exporter** — the scheduled process that copies `dbo.SecurityAuditLog` into the immutable audit store | CREATE | **net-new** | **On the map by role, with no path yet, and §12.2.2 states the decision that closes it.** [06 §9.5](06-azure-hosting-recommendations.md) specifies this process completely as *behaviour* — its two identities and grants, its cursor blob in a second container, its batch selection, its deterministic batch name, its SHA-256 content hash, its four-step conditional-create acknowledgement protocol, what it may log and four failure tests — and specifies it **not at all** as an artifact: no language, no project, no image and no path. That gap is real rather than editorial, because the two forms it could take differ in whether this repository gains a project at all, and one of them also changes §7.2 and §12.4. It is a row rather than a silent absence for the same reason the CI pipeline row is: a target whose existence is certain and whose shape is a decision belongs on the map with its owner named |
| `src/MVC3/**`, `src/MVC4/**`, `src/MVC5/**` — the three legacy projects, their four solutions and their committed databases | **REFERENCE** | themselves | **Retained read-only** as historical references and as the behavioural baseline the port is validated against; none is modified and none is deleted (§12.3). They continue to exist alongside the single new root solution, so §5.6's consolidation is of the *target* rather than a deletion of the past. **This is the only non-CREATE row on the map**, and it is a row rather than prose because a reader checking the map for a target that has no source needs to find the retained legacy tree accounted for here too |
| `src/MvcMusicStore/packages.lock.json` | CREATE | **net-new** (generated by restore, then committed) | Locked transitive graph; CI restores in locked mode (§6.4). It is a generated companion of the project file rather than a separately authored artifact, which is why it is required by the plan's dependency-pinning rule (§0.5.2, *"the target commits a `packages.lock.json` per project"*) rather than listed as its own row in the plan's transformation map |
| `src/MvcMusicStore/ApplicationServices.cs` | CREATE | **net-new** — the legacy application has no equivalent: its composition is spread across `Global.asax.cs` and the OWIN startup files [src/MVC5/MvcMusicStore/Global.asax.cs:13-21], [src/MVC5/MvcMusicStore/Startup.cs:4-14], and nothing there is callable from outside the web process | **The application's one registration seam**, and the file exists so that the seam has a name a second project can call. It declares `public static class ApplicationServices` with one method — `public static IServiceCollection AddMvcMusicStoreServices(this IServiceCollection services, IConfiguration configuration)` — which registers **both `DbContext` types** (`MusicStoreEntities` and `ApplicationDbContext`, each on the SQL Server provider at its configured connection string), the **Identity core, store and manager services** over `ApplicationUser` including the role manager, the password hasher and the Identity options, and the **options objects the tools read**, bound from the `IConfiguration` passed in. **`Program.cs` calls this method itself**, which is the property that matters: there is exactly **one** registration path, so a tool cannot compose a different application than the one that serves requests. What deliberately stays in `Program.cs` and is **not** in the seam: the HTTP-pipeline concerns — MVC and Razor, the cookie authentication schemes, session, anti-forgery, data-protection key persistence, health checks and the middleware order — because a console host has no pipeline to attach them to and a tool that acquired them would be configuring behaviour it never exercises. The **values** applied inside the seam are not this document's: the authentication, password and lockout policy is [05 §6.1](05-aspnet-core-migration-approach.md)'s, the two contexts and their separate migration sets and history tables are [05 §4.5](05-aspnet-core-migration-approach.md)'s, and everything `Program.cs` composes around this call is [05 §2](05-aspnet-core-migration-approach.md)'s. This row owns the file, the type, the method signature, what is registered there versus in `Program.cs`, and the single-path rule; §12.4 owns why the tools depend on it. Added to this map under the plan-correction record of §12.1 |
| `src/MvcMusicStore.Contracts.Tests/MvcMusicStore.Contracts.Tests.csproj` | CREATE | **net-new** — the repository contains no test project of any kind (appendix A.4) | SDK-style, `net8.0`, with its own committed lockfile, declaring **seven** pins of §7.2 directly — the six test-tooling pins `xunit` `2.9.2`, `xunit.runner.visualstudio` `2.8.2`, `Microsoft.NET.Test.Sdk` `17.11.1`, `AngleSharp` `1.7.2`, `Microsoft.Data.SqlClient` `5.1.9` and `Microsoft.Playwright` `1.62.0`, plus the band pin `Microsoft.Extensions.Identity.Core` `8.0.30`, which is here because this project owns the diagnostic pseudonym scheme that **invokes** `ILookupNormalizer` (§7.2, §7.5, §7.6, §7.7, §7.8). **Three of the seven are declared in the in-process project as well, and four are not, and the split is not a style choice:** the three **test-execution** pins — `xunit`, `xunit.runner.visualstudio` and `Microsoft.NET.Test.Sdk` — deliver their function through build and analyzer assets, which **do not cross a project reference**, so each runnable test project declares them itself (§12.3); `Microsoft.Playwright` is in the same group for the reason §7.7 states. The three **library** pins — the response-body parser, the SQL client and the Identity abstractions — are declared **here and only here**, because this project owns the assertions that parse a body, the state observer and legacy attach/detach lifecycle that need a SQL connection, and the pseudonym canonicalization, and their compile and runtime assets *do* reach the in-process project through its reference to this one (§7.5, §7.6, §7.8). **It carries no `ProjectReference` to `src/MvcMusicStore/MvcMusicStore.csproj` and no reference to any other target project** — it reaches whatever host it is pointed at **purely over HTTP**, at a base address it reads from configuration. That absence is the load-bearing property of this row and the reason there are two test projects rather than one: it is what makes this project **buildable and runnable before the ported web application exists**, which is the condition [03](03-modernization-roadmap.md)'s W4 has to meet. It owns the **shared contract assertions** and the **legacy-baseline fixture**, and it is the project W4 restores, builds and runs to characterize the legacy baseline. It also owns the **deployed-only fixture and its concrete class**, for the reason those two rows below state — the only classes here that the baseline run does **not** execute, because a category filter selects them and a deployed host, not this project's reference set, is what they need. **The assertions are declared as one `abstract` class per contract surface**, each holding that surface's `[Fact]` and `[Theory]` methods and taking every dependency it needs — the base address, the client, the state observer — from an **injected, runtime-neutral context** rather than from anything that names a runtime. **The legacy-bound concrete classes are declared here, in this assembly**: one `sealed` class per surface, deriving from that surface's base and supplying the legacy context, and carrying nothing else. That is the arrangement §12.3 argues, and the reason the concretes are declared per assembly rather than inherited across the reference is a property of test discovery rather than of the compiler — stated there once. This row owns that the project exists, where it sits, what it references and the shape of the classes it declares; the assertions' architecture and coverage are [05 §12.2](05-aspnet-core-migration-approach.md) and [05 §12.4](05-aspnet-core-migration-approach.md)'s, the context abstraction and the legacy fixture's design are [05 §12.6](05-aspnet-core-migration-approach.md)'s, and the runnable commands are [05 §12.10](05-aspnet-core-migration-approach.md)'s (see the note following this table). Its place in the sequence is [03](03-modernization-roadmap.md)'s |
| `src/MvcMusicStore.Contracts.Tests/Contracts/**` | CREATE | **net-new** | The **`public abstract` contract bases**, one per surface — the single copy of every shared assertion. This document fixes their declaration site, their accessibility and the fact that they are abstract (§12.3); the surfaces themselves, their assertion bodies and the injected context's shape are [05 §12.6](05-aspnet-core-migration-approach.md)'s and are not reproduced here |
| `src/MvcMusicStore.Contracts.Tests/Legacy/**` | CREATE | **net-new** | The **`public sealed` legacy-bound concretes**, one per surface, each a derivation plus the legacy context plus its `[Collection]` attribute and **no assertion logic**. These are the classes the pre-port baseline run discovers, which is the property §12.3 argues. This project also declares the standalone destructive-operation sweep class [05 §12.8](05-aspnet-core-migration-approach.md) specifies |
| `src/MvcMusicStore.Contracts.Tests/LegacyBaselineFixture.cs` | CREATE | **net-new** | The **`public` collection-fixture type** the legacy-side collection definitions bind, instantiated once per collection. What it does — deploying, starting and driving the MVC 5 application over HTTP, the two-database reset and `ResetAsync()` — is [05 §12.3](05-aspnet-core-migration-approach.md)'s and [05 §12.7](05-aspnet-core-migration-approach.md)'s; this row owns its existence, its path, its accessibility and its project |
| `src/MvcMusicStore.Contracts.Tests/DeployedEndpointFixture.cs` | CREATE | **net-new** | The **`public sealed` deployed-only fixture**, and the third fixture type in the suite — mapped because the one `Category=Deployed` case of [05 §12.4](05-aspnet-core-migration-approach.md) row 47 runs against a **deployed** host and neither mapped fixture can supply one. Its structural properties are this row's: it is **runtime-neutral** and names no application type, which is why it sits in the project that carries **no reference to the web application**; it **starts nothing and provisions nothing** — **no in-process host and no database** — and exposes no observer, no setup API and no reset, so no assertion bound to it can reach a store even by accident; it **consumes a base address** from configuration rather than producing one; and it **owns its own HTTP client policy, with automatic redirect following disabled**, because row 47's assertion *is* the redirect and a client that followed it could observe neither the status nor the `Location`. **What it reports when the address is absent, and the four-value admissible redirect set it asserts, are [05 §12.6](05-aspnet-core-migration-approach.md)'s and [05 §12.4](05-aspnet-core-migration-approach.md)'s** |
| `src/MvcMusicStore.Contracts.Tests/DeployedEndpointTests.cs` | CREATE | **net-new** | The **`public sealed` concrete** the deployed stage discovers — `[Collection("Deployed")]`, holding row 47's cases **directly** rather than deriving from a contract base, because the legacy application has no deployed surface to characterize and there is therefore nothing to share. It is the **only** class in either assembly carrying that trait, which is what makes a category filter a complete selection of it. **Which stage runs it and which workstream authors it, stated because a mapped class with neither is not runnable:** it is executed by [06 §12.1](06-azure-hosting-recommendations.md)'s **non-traffic deployment-verification stage**, whose invocation is written at [05 §12.10](05-aspnet-core-migration-approach.md)'s Stage C and whose pipeline manifest [03](03-modernization-roadmap.md)'s **W11** authors; and it is **authored in [03](03-modernization-roadmap.md)'s W7**, alongside the target-facing concretes, whose exit gate requires this binding to **compile and be discoverable** while explicitly excluding its execution — there is no deployed host at that gate. **No new project and no new pin**: it is a class in the project W4 already created, and its assertions need only the client that project already declares |
| `src/MvcMusicStore.Contracts.Tests/Collections/*.cs` | CREATE | **net-new** | One **`public`** collection-definition class per surface group this assembly declares a class in — `[CollectionDefinition]`, `ICollectionFixture<LegacyBaselineFixture>`, a `const string` for the name, and no test methods (§12.3). **One further definition in the same folder binds the other fixture this assembly declares**: the deployed-only collection, `ICollectionFixture<DeployedEndpointFixture>`, whose name is the trait the deployment stage filters on — which is why this assembly has **two** fixture types and its definitions do not all name the same one. Added to this map under the plan-correction records of §12.1 |
| `src/MvcMusicStore.Contracts.Tests/xunit.runner.json` | CREATE | **net-new** | Runner configuration for the project above, carrying the assembly-level switch `"parallelizeTestCollections": false` — that exact value, not a default relied upon. Declared as a **content item with `CopyToOutputDirectory` set to preserve-newest**, because a runner configuration the build does not copy beside the test assembly is a file the runner never reads. The requirement it satisfies is [05 §12.7](05-aspnet-core-migration-approach.md)'s and is not restated here: the fixtures own shared databases and, on the baseline side, a privately deployed legacy application, so concurrent collections would race a reset against an attach. This row owns only that the file exists, where it sits, its exact setting and how it reaches the output directory |
| `src/MvcMusicStore.Contracts.Tests/Fixtures/fixture-data.json` | CREATE | **net-new** | The **single shared fixture-data manifest**, and the reason it is one file rather than two is that a cross-baseline comparison of two applications holding different data compares nothing. **Both** loaders read it: the legacy loader in this project and the target loader in `src/MvcMusicStore.Tests`. It is declared **once**, here, as a **content item with `CopyToOutputDirectory` set to preserve-newest**, which is what puts it beside the test assembly of both projects — this project's own output for the baseline run, and, because copy-to-output content items flow along a project reference, `src/MvcMusicStore.Tests`'s output for the target run. This row owns its existence, its path, its single-declaration rule and how it reaches each output directory. **Its contents are [05 §12.3](05-aspnet-core-migration-approach.md)'s** — the entities, exact row counts, fixed integer keys, the fixture account set and the manifest fingerprint — and are not reproduced here |
| `src/MvcMusicStore.Contracts.Tests/Fixtures/ModelOverrides/` — **nine files, named here because [05 §12.4](05-aspnet-core-migration-approach.md) row 53 writes the folder and a placeholder for the file**: `column-type.json`, `precision-and-scale.json`, `max-length.json`, `nullability.json`, `identity.json`, `primary-key.json`, `delete-rule.json`, `column-default.json`, `index.json` | CREATE | **net-new** | One file per divergence dimension of [05 §5.1](05-aspnet-core-migration-approach.md) step 3, in the order that section enumerates them — so the nine names above are row 53's `53a` through `53i` respectively, and the `--model-overrides` path in each of those nine command lines resolves to exactly one of them. **The dimension is in the file name and the case letter is not**, deliberately: the case ids belong to 05's matrix and are renumbered there, while the dimension is the thing the file diverges, so a file named for its dimension does not go stale when the matrix moves. They are **content items with `CopyToOutputDirectory` set to preserve-newest**, in the same declaration and for the same reason as the manifest above: the tool receives a **path** and reads the file from disk, so it has to exist beside the test assembly of whichever project runs the case. **What each file contains — the single dimension it diverges and the value it diverges to — and the `--model-overrides` switch that consumes it are [05 §5.1](05-aspnet-core-migration-approach.md)'s and [05 §5.8](05-aspnet-core-migration-approach.md)'s**, and this row reproduces neither |
| `src/MvcMusicStore.Contracts.Tests/Fixtures/seed-expected.json` | CREATE | **net-new** | The seeding oracle — the expectation row 24 `24a` compares the seeded target against, captured from the legacy store rather than computed by the code under test. Same fixture path, same **preserve-newest content item** behaviour, and read by **both** projects' loaders for the same reason the manifest is. Its per-table counts and digests, and the gate that its recorded legacy commit must equal the baseline reference's, are [05 §12.4](05-aspnet-core-migration-approach.md)'s |
| `src/MvcMusicStore.Contracts.Tests/Fixtures/baseline-reference.json` | CREATE | **net-new** | The **committed** baseline reference — the accepted legacy identity a consuming run cannot compute for itself, and the value the seeding oracle above is gated against. Same fixture path and the same **preserve-newest content item** behaviour, because the run that reads it is the one whose output directory it must sit in. It is a **tracked file updated in the change that accepts a re-captured baseline**, which is what distinguishes it from the run-produced baseline *record*: that record carries the resolved runtime identities of a single execution, including the normalizer identity of §7.8, and is a run artifact rather than a mapped file. **Its three fields, the fail-closed comparison and the platform split that consumes it are [05 §12.10](05-aspnet-core-migration-approach.md)'s** |
| `src/MvcMusicStore.Tests/MvcMusicStore.Tests.csproj` | CREATE | **net-new** — same absence (appendix A.4) | SDK-style, `net8.0`, with its own committed lockfile, and **five direct package references of its own** (§7.2): `Microsoft.AspNetCore.Mvc.Testing` `8.0.30`, which only this project needs, and — declared here **as well as** in the contracts project rather than inherited from it — `xunit` `2.9.2`, `xunit.runner.visualstudio` `2.8.2`, `Microsoft.NET.Test.Sdk` `17.11.1` and `Microsoft.Playwright` `1.62.0`. **The redeclaration is required, not redundant:** those four deliver their function through build and analyzer assets, which do not cross a project reference (§12.3), so a project that acquired them only through the reference below would compile and then be neither built as a test project nor discovered nor executed by `dotnet test` — the adapter absent from its output, the test targets never imported. `AngleSharp` `1.7.2`, `Microsoft.Data.SqlClient` `5.1.9` and `Microsoft.Extensions.Identity.Core` `8.0.30` are deliberately **not** redeclared here: they are library pins whose compile and runtime assets do cross the reference (§7.5, §7.6, §7.8). **Two project references, and both are required:** `src/MvcMusicStore/MvcMusicStore.csproj`, because packages alone do not make an in-process host buildable and this is what makes the test assembly compile against, and boot, the web application's own assembly; and `src/MvcMusicStore.Contracts.Tests/MvcMusicStore.Contracts.Tests.csproj`, because the **abstract contract bases** it derives from live there — two copies of one assertion is the arrangement in which the baseline side and the target side silently stop asserting the same thing. **What this project declares is its own set of `sealed` concrete classes**, one per surface, each deriving from that surface's abstract base and supplying the **target** context — the in-process host's client and the target-side state observer — and carrying no assertion logic of its own. **The reference alone would run nothing**, which is exactly why the concretes are declared here: §12.3 states the discovery property that makes this the mechanism rather than a stylistic choice. It hosts the application **in process**, and it is the project [03](03-modernization-roadmap.md)'s W7 adds alongside the contracts project. It is the only *test* project that references the web application; the two tool projects reference it as well, for the separate reason their own rows state, and no legacy project is referenced by anything. The split of ownership is the same as the row above: the fixture design that consumes this reference — the factory, its overrides and its clients — is [05 §12.6](05-aspnet-core-migration-approach.md)'s, and the runnable commands are [05 §12.10](05-aspnet-core-migration-approach.md)'s |
| `src/MvcMusicStore.Tests/Core/**` | CREATE | **net-new** | The **`public sealed` target-bound concretes**, one per shared surface, **plus the target-only classes that have no legacy counterpart** — each a derivation or a standalone class, its target context, its `[Collection]` attribute and no assertion logic. Which rows are target-only is [05 §12.4](05-aspnet-core-migration-approach.md)'s |
| `src/MvcMusicStore.Tests/CoreApplicationFixture.cs` | CREATE | **net-new** | The **`public` collection-fixture type** the target-side collection definitions bind, instantiated once per collection. Its in-process host, the disposable engine it provisions and its reset are [05 §12.6](05-aspnet-core-migration-approach.md)'s and [05 §12.7](05-aspnet-core-migration-approach.md)'s; this row owns its existence, its path, its accessibility and its project |
| `src/MvcMusicStore.Tests/Collections/*.cs` | CREATE | **net-new** | The same one-per-group definitions for this assembly, binding `ICollectionFixture<CoreApplicationFixture>` — **required separately, because `[Collection]` names resolve within the assembly being run** (§12.3). Added to this map under the plan-correction record of §12.1 |
| `src/MvcMusicStore.Tests/xunit.runner.json` | CREATE | **net-new** | The same file for the in-process project, with the same exact value `"parallelizeTestCollections": false` and the same preserve-newest content-item copy behaviour. **One per test project, because the setting is assembly-level:** a single copy in one project does not govern the other project's assembly, and the target-side fixtures are the half that provisions databases and applies migration sets. Requirement source: [05 §12.7](05-aspnet-core-migration-approach.md) |
| `tools/provision-admin/ProvisionAdmin.csproj` | CREATE | `src/MVC5/MvcMusicStore/App_Start/Startup.App.cs` — the startup provisioning it replaces | SDK-style `net8.0` console project with its own committed lockfile. **It carries a `ProjectReference` to `src/MvcMusicStore/MvcMusicStore.csproj`**, for the reason §12.4 states: it resolves `UserManager<ApplicationUser>` and `RoleManager` from the application's own container, so it must compile against `ApplicationUser`, the Identity context and the composition root. **It is excluded from the web application's publish output and published separately** (§12.4). The command's behaviour, its secret channel and its idempotence are [05](05-aspnet-core-migration-approach.md)'s and [06](06-azure-hosting-recommendations.md)'s |
| `tools/migrate-data/MigrateData.csproj`, `tools/migrate-data/Program.cs` | CREATE | **net-new** — no legacy file performs this work. The legacy application ships no migration tooling of any kind, and the three tracked `MvcMusicStore-Create.sql` scripts are **not** it: they are schema scripts — two byte-identical MVC 4 duplicates that are not runnable as written, plus one tutorial asset of the retired-provider edition — and §13.1 states why none of them is even a schema baseline, let alone a data migration | SDK-style `net8.0` console project with **its own committed lockfile**, restored in locked mode like every other project (§6.4). **It carries a `ProjectReference` to `src/MvcMusicStore/MvcMusicStore.csproj`** on the same grounds as the operator tool, plus one of its own: its `schema-diff` mode generates the DDL the migration sets would apply, so it must compile against **both `DbContext` types and both migration sets** as well as the composition root (§12.4). **Excluded from the web application's publish output and published separately** (§12.4). Its six modes, its machine-readable reconciliation output and its exit-code contract are [05 §5.8](05-aspnet-core-migration-approach.md)'s; the principal that runs it from the release pipeline is [06 §6.2](06-azure-hosting-recommendations.md)'s. Added to this map under the plan-correction record of §12.1 |
| `src/MvcMusicStore/Resources/ApplicationMessages.resx` | CREATE | **net-new** — no edition has a resource file of any kind, verified repository-wide: `git ls-files '*.resx'` returns `0`, so every user-facing string in all three legacy applications is a literal in a view or a `ModelState` message | The application's **one committed resource file**, and the only reason it is mapped here is that a named entry in it is a **test oracle**: [05 §12.4](05-aspnet-core-migration-approach.md) row 37 asserts the rendered cart-migration notice is byte-equal to the value of the entry **`CartMigrationNotice`**, read from the resource rather than transcribed into the test. `.resx` is an **`EmbeddedResource`** under the SDK's default item behaviour — not copied to output — so the reading side must hold a reference to this assembly; it does, because row 37 is **target-only** and its concrete sits in `src/MvcMusicStore.Tests`, which references the web project. That is a structural consequence worth stating: the reference-free contracts project could not read this oracle, which is one more reason the case is target-side. **The entry's value is a product decision and is [05](05-aspnet-core-migration-approach.md)'s**, together with the four predicates that row holds it to; this row owns the file's existence, its path, its build action and the fact that the string has exactly one home. Added to this map under the plan-correction record of §12.1 |
| Everything under `src/MvcMusicStore/` not listed above — `Program.cs`, `appsettings*.json`, controllers, models, binding and view models, services, `Data/`, views, view components | CREATE | mapped file-by-file by [05](05-aspnet-core-migration-approach.md) | Not specified here. This document's only claim is that they are compiled by implicit globbing rather than by an explicit `Compile` inventory (§5.1) |
| `src/MvcMusicStore/packages.lock.json` | CREATE | **net-new** (generated, then committed) | Locked transitive graph; CI restores in locked mode (§6.4) |
| `src/MvcMusicStore/Properties/launchSettings.json` | CREATE | `src/MVC5/MvcMusicStore/MvcMusicStore.csproj:18-22`, `:277-294` — the IIS Express settings and web-project block it replaces | Local launch profiles. Developer configuration, not a build input; the deployed hosting model is [06](06-azure-hosting-recommendations.md)'s |
| `src/MvcMusicStore.Tests/MvcMusicStore.Tests.csproj` | CREATE | **net-new** — the repository contains no test project of any kind (appendix A.4) | SDK-style, `net8.0`, referencing `Microsoft.AspNetCore.Mvc.Testing` `8.0.30`, `xunit` `2.9.2`, `xunit.runner.visualstudio` `2.8.2` and `Microsoft.NET.Test.Sdk` `17.11.1`, plus its own committed lockfile. The suite's **architecture and coverage** are [05](05-aspnet-core-migration-approach.md)'s and its place in the sequence is [03](03-modernization-roadmap.md)'s |
| `tools/provision-admin/ProvisionAdmin.csproj` | CREATE | `src/MVC5/MvcMusicStore/App_Start/Startup.App.cs` — the startup provisioning it replaces | SDK-style `net8.0` console project with its own committed lockfile, **not deployed with the web application**. The command's behaviour, its secret channel and its idempotence are [05](05-aspnet-core-migration-approach.md)'s and [06](06-azure-hosting-recommendations.md)'s |
| `tools/migrate-data/MigrateData.csproj` and its `Program.cs` | CREATE | **net-new** — no legacy file corresponds to it, because the legacy application has no data-migration path at all: its only schema-and-data bootstrap is the destructive initializer [src/MVC5/MvcMusicStore/Models/SampleData.cs:9], which creates a database and seeds a catalog rather than moving existing rows. No `tools/` directory exists today either (appendix A.4) | SDK-style `net8.0` console project, a member of the root solution (§5.6) with its own committed lockfile (§6.4), referencing `src/MvcMusicStore/MvcMusicStore.csproj` for the model and context types, and **not deployed with the web application** — not referenced by the web project and not in its publish output. It is run as a release step, not by the application. Its **sub-command set** — one per gated action across the two data migrations, plus the artifact that closes them — is enumerated by [05](05-aspnet-core-migration-approach.md) §5.6, which also owns its exit contract, checkpoint-based idempotence, reconciliation rules and sensitive-artifact controls; the order in which the release invokes them, its place in the provisioning order and the principal that runs it are [06](06-azure-hosting-recommendations.md) §6.3 and §6.8's. **This row deliberately states no sub-command count**: the set is the owner's to enumerate, and a number repeated here would be a second place to keep correct |
| `tools/seed-sample-data/SeedSampleData.csproj` and its `Program.cs` | CREATE | `src/MVC5/MvcMusicStore/Models/SampleData.cs` — the file whose two halves it splits: the destructive initializer [:9] is **not** reproduced anywhere, and the seed body is rebuilt as `src/MvcMusicStore/Data/SeedData.cs`, which [05](05-aspnet-core-migration-approach.md) maps. **This row maps the executable host that invokes that routine**, which the source has no equivalent of: nothing invoked the seed explicitly, because the initializer the application registered at startup [src/MVC5/MvcMusicStore/Global.asax.cs:20] ran it. The source file is **826 physical lines by `wc -l`** — the two-metric rule is [08](08-technical-debt-register.md) §2.1's and the figure is its §3.2's | SDK-style `net8.0` console project, a member of the root solution (§5.6) with its own committed lockfile (§6.4), referencing `src/MvcMusicStore/MvcMusicStore.csproj` so it can build a host and resolve the catalog context from the same container configuration, and **not deployed with the web application**. It is deliberately a **second tool** rather than a sub-command of `tools/migrate-data`, so the binary the production release invokes carries no seed path at all. Its three fail-closed guards, its command form and its exit contract are [05](05-aspnet-core-migration-approach.md) §5.4's |
| Everything under `src/MvcMusicStore/` not listed above — `Program.cs`, `appsettings*.json`, controllers, models, binding and view models, services, the rest of `Data/` including `Data/SeedData.cs` and the two contexts, views, view components | CREATE | mapped file-by-file by [05](05-aspnet-core-migration-approach.md) | Not specified here. This document's only claim is that they are compiled by implicit globbing rather than by an explicit `Compile` inventory (§5.1) |
| `src/MvcMusicStore/packages.lock.json` | CREATE | **net-new** (generated, then committed) | Locked transitive graph; CI restores in locked mode (§6.4) |
| `src/MvcMusicStore.Tests/MvcMusicStore.Tests.csproj` | CREATE | **net-new** — the repository contains no test project of any kind (appendix A.4) | SDK-style, `net8.0`, referencing `Microsoft.AspNetCore.Mvc.Testing` `8.0.30`, `xunit` `2.9.2`, `xunit.runner.visualstudio` `2.8.2` and `Microsoft.NET.Test.Sdk` `17.11.1`, plus its own committed lockfile. It also carries a **`ProjectReference` to `src/MvcMusicStore/MvcMusicStore.csproj`**, because `WebApplicationFactory<Program>` hosts the application in-process and needs the entry-point assembly (§12.3). The suite's **architecture and coverage** are [05](05-aspnet-core-migration-approach.md)'s and its place in the sequence is [03](03-modernization-roadmap.md)'s |
| `tools/provision-admin/ProvisionAdmin.csproj`, with its `Program.cs` | CREATE | `src/MVC5/MvcMusicStore/App_Start/Startup.App.cs` — the startup provisioning it replaces | SDK-style `net8.0` console project with its own committed lockfile and the **same `UserSecretsId` as the web project** (§5.7), **not deployed with the web application**, and the **only operator console project in this map**. It carries **four verbs, not four projects** — administrator provisioning, the catalog data phase, the Identity data phase and non-production seeding — so there is one project file, one `Program.cs`, one publish output, one checksum and one promotion path for every operator action the target needs (§12.3, §12.4). It carries a **`ProjectReference` to `src/MvcMusicStore/MvcMusicStore.csproj`**, so it builds the same Identity registrations the web application validates rather than a second copy of them (§12.3) — a build-time edge, which is not a deployment. Each verb's own contract — its switches, its guards, its assertions, its secret channel and its idempotence — is [05](05-aspnet-core-migration-approach.md) §5.1.2, §5.4 and §10.2's; which stage runs it, under which principal and against what asserted target is [06](06-azure-hosting-recommendations.md) §6.3.2's |
| `src/MvcMusicStore/ViewComponents/GenreMenuViewComponent.cs`, `src/MvcMusicStore/ViewComponents/CartSummaryViewComponent.cs`, `src/MvcMusicStore/ViewComponents/RemoveAccountListViewComponent.cs`, each with its `Default.cshtml` under `src/MvcMusicStore/Views/Shared/Components/GenreMenu/`, `.../CartSummary/` and `.../RemoveAccountList/` respectively | CREATE | `src/MVC5/MvcMusicStore/Controllers/StoreController.cs:43-55`, `src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:86-99`, `src/MVC5/MvcMusicStore/Controllers/AccountController.cs:316-322` — the three `[ChildActionOnly]` actions | Named here rather than left to the catch-all row below **because the view-component convention fixes these paths exactly, and a wrong path fails at runtime rather than at build time**. The framework locates a component's view by convention from the class name, so the three classes sit in one `ViewComponents/` directory and each `Default.cshtml` must be in its **own** subdirectory under `Views/Shared/Components/`, named for its component — six files, one new directory plus three view subdirectories. They are **not** the only directories the port adds: the catch-all row below carries `Binding/`, `Services/` and `Data/` (with `Data/Migrations/` beneath it), and `wwwroot/` is its own row above. What is structural about these is the convention, not exclusivity. Both kinds are then picked up with no `Include` entry — the `.cs` files by the SDK's implicit `Compile` glob (§5.1), the `Default.cshtml` files by the implicit Razor item glob. The **conversion itself**, each component's model and its call-site rewrite are [05](05-aspnet-core-migration-approach.md) §8.2's, which carries the same six paths |
| Everything under `src/MvcMusicStore/` not listed above — `Program.cs`, `appsettings*.json`, controllers, models, binding and view models, services, `Data/`, the 29 views | CREATE | mapped file-by-file by [05](05-aspnet-core-migration-approach.md) | Not specified here, beyond the two declarations the project graph requires of `Program.cs` (§12.3). This document's only other claim is that they are compiled by implicit globbing rather than by an explicit `Compile` inventory (§5.1) |

**The row count, checkable.** The table carries **35 rows against the authoritative map's 27 target
groups** — `27 + 3 + 5 = 35` — and the eight extra rows are splits and sanctioned additions rather than
inventions:

- **Three of the eight come from two splits, and neither split adds a target.** The SDK-band and
  package-source manifests are one group and two files, so `global.json` and `NuGet.config` take a row
  each — one extra row. The three `[ChildActionOnly]` view components are one group and three
  components, so they take three rows — two extra rows, written out for the reason §12.1 gives.
- **Five additional rows, each with its own sanction stated where it is decided.** The **lockfile** row is
  required by §6.4's locked-mode restore and names **three** files in one row, one per project, because
  §6.4's requirement is per project and there are three; and `wwwroot/lib/**` is the committed output of the
  `libman.json` acquisition mechanism §9.3 fixes — both dependency-governance artifacts, which is this
  document's own subject rather than an amendment to anyone else's.
  `wwwroot/js/shopping-cart.js` is required by [06 §10.2](06-azure-hosting-recommendations.md)'s
  `script-src 'self'` policy and carries [05 §7.4](05-aspnet-core-migration-approach.md)'s token header,
  and it is a row rather than a member of `wwwroot/**` because its source is a Razor view rather than an
  asset file — a derivation no glob over the asset directories would express. `deploy/sql/SecurityAuditLog.sql`
  is required by [06 §9.5](06-azure-hosting-recommendations.md)'s durable audit table, whose DDL that
  document states in full and whose *location* nothing stated until this row. And the **audit exporter** is
  the fifth: [06 §9.5](06-azure-hosting-recommendations.md) specifies the process and no artifact for it, so
  §12.2.2 carries the decision that gives it a path.
- **One row that was here and is not any longer, recorded rather than quietly dropped.** An earlier form of
  this table carried a **pinned browser harness for the CSP delivery checks** as an open target.
  [06 §10.2](06-azure-hosting-recommendations.md) has since **decided that question against the harness**:
  it considers a pinned browser-automation stack, rejects it on proportionality, and selects a blocking,
  manually executed deployed-browser network-panel gate with a signed-off artifact. A gate that produces no
  file has no row here — exactly as the code-deployment choice means no `Dockerfile` — so the row is removed
  and the count above reflects its removal. Keeping it would have left this map advertising an artifact its
  owner had ruled out.
- **Every other row is one group.** No target group is represented by a plural mention, an "etc." or an
  unbounded wildcard, and the one **REFERENCE** row accounts for the retained legacy tree so that a
  reader walking the map finds every path in it either created from a named source, marked net-new, or
  retained unchanged.

**Fifteen further target files exist and are named in §12.2.1 rather than in the table above.** They
are the types the migration's design documents require — a health check, an exception-record writer, the
three authenticated diagnostic operations' handler bodies, the
credential-verification and Identity audit seams, an authorization result handler, the
Content-Security-Policy artifacts, the pseudonymization service and the application's own bound options
types. **None of them appears in the authoritative map's 27 groups**, because that map was written
before those designs were, and each is required by a decision the deliverable that owns it has since
taken. §12.2.1 gives every one an exact containing file for the same reason this table exists: a target
file with no named owner is the omission a map is built to prevent, and "somewhere in the web project"
is not a location.

**One artifact a reader may expect here and will not find: `Properties/launchSettings.json`.** The IIS
Express settings and the `ProjectExtensions` web-project block are dropped (§5.4) and **nothing in this
map replaces them**. The reason is a scope rule rather than a technical one: **the authoritative future
application map has no launch-profile row, and this map carries that map's groups rather than adding to
them** — a local launch profile is developer-machine configuration, it is not a build input, it is not
deployed, and no approved amendment adds it. Local running therefore needs no committed artifact: with no launch profile present,
`dotnet run` binds Kestrel's default local endpoints, and a developer who wants different ones sets
`ASPNETCORE_URLS` in their own environment. If a team later decides to commit shared launch profiles, that
is an amendment to the map and belongs in the approval that authorizes it, not in this strategy.

#### 12.2.1 The designed types, and the exact file each one lives in

**Every row above names a file. The designs the other deliverables have since written require types
that none of those files was named to hold, and a type with no containing file is exactly the unowned
target §12.1 forbids.** So they are given files here. The division of labour is unchanged and is what
makes this table this document's to write rather than a duplication of anyone else's: **the owning
deliverable decides that the type exists, what it does and how it is registered; this map decides which
file it lives in and where that file sits in the project.** Every row therefore cites its owner and adds
no behaviour of its own.

Four placement rules are applied uniformly, so a reader can predict the answer rather than look it up:
**one public type per file, named after the type**; folders by role — `Health/`, `Diagnostics/`,
`Identity/`, `Security/`, `Configuration/`; **options types together under `Configuration/`**, one per
bound configuration section and named after that section, because their common property is that startup
binds and validates them rather than any subject they share with their consumers; and **no new project**
— all fifteen compile into the web application by implicit globbing, with no `Compile` item and no
fourth project (§5.1, §12.5).

**The first rule says *public* type deliberately, and `Security/CspReportEndpoint.cs` is the row that
establishes the latitude.** The two report shapes it deserializes are declared there as **nested private
records**, not as files of their own, because they are a wire format with exactly one reader: nothing
outside that handler constructs one, returns one or names one in a signature, and a `CspReportBody.cs`
sitting in `Security/` alongside the policy would read as a shared contract that other code is invited to
depend on. Nested and private, they cannot acquire a second consumer without the declaration moving
first, which is the property worth having. The fifteen-file count is unaffected — the arithmetic counts
files, and this is one file — and **one other row takes the same exception, for the same reason**:
`Diagnostics/InternalDiagnosticsEndpoints.cs` declares the three diagnostic operations' fixed response
shapes as nested private records, since they too are a wire format with exactly one reader. Those are the
only two rows that use the latitude, and neither adds a file.

| Type or types | Target file | Required by | Source |
| --- | --- | --- | --- |
| `CatalogConnectivityHealthCheck` — the application-owned `IHealthCheck` behind the readiness endpoint | `src/MvcMusicStore/Health/CatalogConnectivityHealthCheck.cs` | [05 §2.4](05-aspnet-core-migration-approach.md) stage 1 item 11, which registers exactly one check and rejects `AddDbContextCheck<T>`; the readiness contract is [06 §9.3](06-azure-hosting-recommendations.md)'s | **net-new** — the repository has no health endpoint of any kind in any edition ([11 §3.8](11-cloud-readiness-assessment.md)), which is why §10.4 can retire the pin its earlier form carried without losing a capability |
| `ErrorRecordExceptionHandler` — the `IExceptionHandler` that writes the error record and returns `false` | `src/MvcMusicStore/Diagnostics/ErrorRecordExceptionHandler.cs` | [05 §2.4](05-aspnet-core-migration-approach.md) stage 1 item 13 and [05 §8.3](05-aspnet-core-migration-approach.md), which give it the record's closed field set | **net-new**, and the derivation is a removal rather than a file: `HandleErrorAttribute` [src/MVC5/MvcMusicStore/App_Start/FilterConfig.cs:10] is the whole of the current error policy and has no target counterpart, so the middleware that replaces it renders a response and records nothing until this type exists |
| `InternalDiagnosticsEndpoints` — the handler bodies for the **three** authenticated diagnostic operations, plus the fixed response shapes they return, declared as **nested private records** under the placement exception above | `src/MvcMusicStore/Diagnostics/InternalDiagnosticsEndpoints.cs` | [06 §7.5](06-azure-hosting-recommendations.md) mechanisms **P-CFG** (`GET /internal/config-summary`) and **P-DP** (`POST /internal/dp/protect`, `POST /internal/dp/unprotect`), which fix each operation's contract, its closed response field set, the dedicated `SlotIsolationProbe` protector purpose and the single authentication contract all three sit behind; [06 §9.3](06-azure-hosting-recommendations.md) gate row 8 fails the release if any of the three is missing, and [05 §2.4](05-aspnet-core-migration-approach.md) owns the implementation and the registration order | **net-new.** One file for all three because they are one instrument family behind one authentication contract, and splitting them would make the gate's "any of the three" assertion span three files for no gain. They are **on this table rather than in §12.2's row for `Program.cs`** for the reason the CSP report endpoint is: the `MapGet`/`MapPost` **registrations** are lines in the composition root, but the handler bodies parse a connection string, hold a closed response shape and must never emit a value outside it — a contract nobody can find inside a route-handler lambda. The **P-CFG parse-failure rule is 06's and is not restated here**: the resolved connection string is never assigned to anything logged, returned or thrown |
| `PasswordDerivationObservation` — the **one** scoped per-request signal, with two consumers | `src/MvcMusicStore/Identity/PasswordDerivationObservation.cs` | [05 §4.3](05-aspnet-core-migration-approach.md) declares it and [09 §6.8.1.1](09-security-assessment.md) seam 1 adds its two audit members | **net-new.** One file and not two: 09's seam 1 records that `PasswordRehashSignal` is an earlier name for **this same object**, and that building it twice is the failure that leaves one consumer reading a value nothing sets. A second file would be that failure made structural |
| `RehashObservingPasswordHasher` — the decorating `IPasswordHasher<ApplicationUser>` that observes and never emits | `src/MvcMusicStore/Identity/RehashObservingPasswordHasher.cs` | [05 §4.3](05-aspnet-core-migration-approach.md) for the uniform-cost signal, [09 §6.8.1.1](09-security-assessment.md) seam 1 artifact 1 for the rehash observation — **one artifact with two readers, registered once** | **net-new**, and **one file, because the decorator is the only new type the seam needs.** [09 §6.8.1.1](09-security-assessment.md) seam 1 **owns the registration shape and this document does not restate it**; the one consequence that belongs on a file map is that the type this decorator depends on is the framework's **concrete** `PasswordHasher<ApplicationUser>`, registered as its own concrete type in `Program.cs` and never the interface the decorator is itself registered as — a framework type needs no file, so the seam adds one row here and not two |
| `AuditingUserManager` — one `UserManager<ApplicationUser>` subclass with **two** overrides, `UpdateUserAsync` and `AccessFailedAsync` | `src/MvcMusicStore/Identity/AuditingUserManager.cs` | [09 §6.8.1.1](09-security-assessment.md) seam 1 artifact 2 and seam 3, which state explicitly that these are the same type | **net-new.** One file for both overrides, because two files would be two registrations of `AddUserManager<T>` and the second would replace the first |
| `UniformVerificationCost` — the singleton that pads credential verification to a fixed cost | `src/MvcMusicStore/Identity/UniformVerificationCost.cs` | [05 §4.3](05-aspnet-core-migration-approach.md) | **net-new** |
| `CredentialEndpointAttribute` — the `[CredentialEndpoint]` marker the rate limiter reads from endpoint metadata | `src/MvcMusicStore/Security/CredentialEndpointAttribute.cs` | [05 §2.4](05-aspnet-core-migration-approach.md) stage 1 item 14, which selects the three credential endpoints by metadata rather than by path string | **net-new** |
| `AuditingAuthorizationMiddlewareResultHandler` — the delegating `IAuthorizationMiddlewareResultHandler` | `src/MvcMusicStore/Security/AuditingAuthorizationMiddlewareResultHandler.cs` | [09 §6.8.1.1](09-security-assessment.md) seam 2 | **net-new**, and its derivation is the authorization surface it observes — the controller-level `[Authorize(Roles = "Administrator")]` [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:12] and the account surface's own attributes, none of which records a denial today because nothing in the repository logs at all ([11 §3.8](11-cloud-readiness-assessment.md)) |
| The Content-Security-Policy value, its enforcing-versus-report-only phase, and the `csp-endpoint` group literal — one holder, because [06 §10.2](06-azure-hosting-recommendations.md) requires the group name and the `report-to` value to be one literal in one place | `src/MvcMusicStore/Security/ContentSecurityPolicy.cs` | [06 §10.2](06-azure-hosting-recommendations.md) | **net-new** — the repository emits no security response header of any kind ([11 §3.6](11-cloud-readiness-assessment.md)), so nothing is being ported |
| The handler `MapPost("/csp-report", …)` targets, plus the two report shapes it deserializes as **nested private records** under the placement exception above | `src/MvcMusicStore/Security/CspReportEndpoint.cs` | [06 §10.2](06-azure-hosting-recommendations.md)'s report endpoint — two media types, a 16 KiB bound, a closed record field set | **net-new.** The `MapPost` **registration** is a line in `Program.cs` and not a file of its own, which is the co-location sanctioned below; what needs a file is the handler body and the shapes it binds, because a route-handler lambda holding a JSON contract inside the composition root is a contract nobody can find |
| `SubjectPseudonym` — the keyed HMAC-SHA256 pseudonymization service and its fail-closed key validation | `src/MvcMusicStore/Security/SubjectPseudonym.cs` | [09 §6.8.1](09-security-assessment.md) for the construction, the per-key length and the refusal to start without a key. **The key record itself, its label grammar and its rotation are [06 §8.4.7](06-azure-hosting-recommendations.md)'s and are cited rather than restated anywhere in this document** — this row names a file, not a key format | **net-new** |
| `HealthProbeOptions` — `ReadinessBudget` and `LivenessBudget` | `src/MvcMusicStore/Configuration/HealthProbeOptions.cs` | [05 §2.4](05-aspnet-core-migration-approach.md) stage 1 item 11, which names the type and states both budgets as configuration with defaults | **net-new** |
| `RateLimitingOptions` — the limits of the three limiters and the one named policy | `src/MvcMusicStore/Configuration/RateLimitingOptions.cs` | [05 §2.4](05-aspnet-core-migration-approach.md) stage 1 item 14, which binds them from `RateLimiting:*` and validates them at startup. 05 names the section and not the type, so **the type name is this map's, by the placement rule above** | **net-new** |
| `EventLogOptions` — the `Security:EventLog` section, whose one member today is the pseudonym key | `src/MvcMusicStore/Configuration/EventLogOptions.cs` | [09 §6.8.1](09-security-assessment.md) for the key's validation at startup, [06 §8.4.7](06-azure-hosting-recommendations.md) — **the sole owner of the key record and its label grammar** — for what the bound value contains and how it is delivered as a platform reference. Section named by its owners, type named here | **net-new** |

**Four co-locations, sanctioned explicitly rather than left as gaps.** Each is a case where a reader
might expect a file and the correct answer is a file already on the map:

- **Every registration of every type above is a line in `src/MvcMusicStore/Program.cs`** — the health
  check, the exception handler, the three credential-verification types, **the framework's concrete
  `PasswordHasher<ApplicationUser>` that the rehash decorator depends on** — a registration with no file,
  whose shape [09 §6.8.1.1](09-security-assessment.md) seam 1 owns — the user manager through
  `AddUserManager<T>`, the authorization result handler, the rate limiter chain, the three options
  bindings, the `MapPost("/csp-report", …)` endpoint and the three `MapGet`/`MapPost` registrations for the
  authenticated diagnostic operations. The composition root is one file by decision
  (§12.2's `Program.cs` row); a partial-class split or an extension-method file per feature is not
  specified, and [05 §2.4](05-aspnet-core-migration-approach.md) owns the order they appear in.
- **`ErrorViewModel.cs` stays where §12.2 puts it**, at the project root rather than under
  `Diagnostics/`, because the authoritative map names that path and this table adds files rather than
  moving them.
- **The audit event classes have no type of their own.** [09 §6.8.1](09-security-assessment.md) emits
  them through `ILogger` with reserved categories and closed field sets, and their producers are the
  controllers, the two seams above and the operator command — so there is no event-catalog type, no
  emitter service and no file for one. A reader looking for `AuditEvent.cs` will not find it, and that
  is the design rather than an omission.
- **A conditional error layout is not a sixteenth file.**
  [05 §8.3](05-aspnet-core-migration-approach.md) permits either `@{ Layout = null; }` declared in the
  error view or a dedicated `Views/Shared/_ErrorLayout.cshtml`; if the second is taken, the file is a
  member of the `Views/**/*.cshtml` row's membership rule — which §12.1 states as 27 files under the
  first choice and 28 under the second — rather than a new row here. Either choice satisfies the same
  requirement, and neither changes this table's fifteen.

#### 12.2.2 The one target this map cannot name yet, and what closes it

**The audit exporter.** [06 §9.5](06-azure-hosting-recommendations.md) requires a scheduled process that
copies rows out of `dbo.SecurityAuditLog` into the immutable audit store, and specifies that process
**completely as behaviour**: its two identities and their separate grants, its cursor blob in a second
container, how it selects a batch, the deterministic batch name, the SHA-256 content hash it writes, the
four-step conditional-create acknowledgement protocol that makes a re-run idempotent, what it may and may
not log, and four failure tests. It specifies it **not at all as an artifact** — no language, no runtime, no
project, no container image and no path — which is why its row on the map above carries a role rather than a
file.

**Why that gap is real rather than editorial.** The two outcomes differ in whether this repository gains a
project at all, and one of them also changes §7.2 and §5.6:

- **A project in this repository** — a `net8.0` console application, scheduled by the platform, pinned
  exactly as §7.2 pins every other version. That outcome adds a project row to §12.2 and a fourth path to
  §5.6's single solution, a fourth `packages.lock.json` to the lockfile row (§6.4 is per project), and §7.2
  rows for the two package families it needs: a SQL client to read the table under the reader identity, and
  a blob client to write the batch and maintain the cursor under the writer identity. It adds **no** edge to
  §12.4's graph — it references nothing in this solution, reaching SQL and blob storage over the network
  exactly as the exporter contract describes — so the graph gains a node and not an edge.
- **A platform-native artifact outside this repository** — a scheduled job or function authored and
  deployed in its own repository, or a platform data-movement pipeline defined in infrastructure rather than
  in code. That outcome adds **no file here at all**, exactly as the code-deployment choice means no
  `Dockerfile` (§12.2), and this sub-section would record that the requirement is met without an artifact in
  this tree.

Naming a path for the first would be this document choosing an answer
[06](06-azure-hosting-recommendations.md) owns; leaving the requirement unmentioned would be the silent
omission §12.1 exists to prevent. So the target is on the map, its sanction is here, and the decision is
cited to its owner — the same treatment the CI pipeline row gets for provider selection. **The blast radius
is bounded and stated rather than left open**: the second outcome changes nothing on this map at all, and
the first changes exactly two things — this row becomes a path, and the lockfile row gains a fourth file —
so no other row is held up by the decision.

**One target this sub-section used to carry and no longer does.** An earlier form of it held open a **pinned
browser harness** for [06 §10.2](06-azure-hosting-recommendations.md)'s report-endpoint delivery checks, on
the grounds that CSP enforcement, CSP3 precedence between the two report transports and the absence of
double delivery are behaviours of a browser's policy engine that no `WebApplicationFactory` request can
produce. That reasoning still holds, but the decision it waited on **has since been taken, against the
harness**: 06 weighs a pinned browser-automation stack, rejects it on proportionality, and selects a
blocking, manually executed deployed-browser network-panel gate with a signed-off artifact per release. A
gate that produces no file has no row on the map, so the row is removed, §12.2's count rationale records the
removal, and the requirement is met without an artifact in this tree.

### 12.3 Three project-structure decisions

**The ported application is a new sibling, not a replacement in place.** It is created at
`src/MvcMusicStore/` rather than overwriting `src/MVC5/MvcMusicStore/`. The reason is validation, not
sentiment: MVC 5 is the reference implementation of every behaviour the port must preserve, and the
behavioural baseline is captured by driving the *running* legacy application — which requires its source,
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

**The suite is two projects, and the shared contracts are shared by inheritance rather than by the project
reference alone.** The split itself follows from §12.2's rows: one project references no target project and
is therefore buildable before the port exists, the other references the ported application because it hosts
it in process. What that split does *not* by itself provide is a way for one copy of an assertion to run
twice, and the arrangement that provides it has to be stated because the obvious reading of a
`ProjectReference` is wrong in a way that produces a suite which passes by running nothing.

> **A `ProjectReference` does not make the referenced assembly's tests run.** The test host is pointed at
> **one** assembly and discovers tests by reflecting over **the types that assembly declares**. A project
> reference makes the referenced assembly's *types* available to the compiler and puts its output beside
> this one's; it does not enrol its tests in this assembly's run, and no filter or runner setting changes
> that. A suite built on that assumption would report a green run of zero contract tests on the target side.
>
> **And a `ProjectReference` does not make the referencing project a test project either — which is the
> second half of the same trap, and the one that produces no run at all rather than an empty one.**
> `Microsoft.NET.Test.Sdk` and `xunit.runner.visualstudio` do their work through **build assets** — MSBuild
> `props`/`targets` that mark the project as a test project, import the test targets `dotnet test` invokes,
> and copy the discovery adapter beside the test assembly — and `xunit` additionally ships **analyzer**
> assets. A `PackageReference`'s **build, analyzer and content assets do not flow to a project that
> references the declaring project**: that exclusion is `PrivateAssets`' default for exactly those asset
> kinds, so only compile and runtime assets cross a project reference. A project that acquired those three
> only by referencing a project that declares them would **compile**, and then be neither built as a test
> project nor discovered nor executed: the adapter is absent from its output and the test targets are never
> imported. **Therefore every runnable test project declares `xunit`, `xunit.runner.visualstudio` and
> `Microsoft.NET.Test.Sdk` directly**, in its own project file, and §12.2's two test-project rows do.
> Library pins behave the other way round and are declared once: `AngleSharp`, `Microsoft.Data.SqlClient`
> and `Microsoft.Extensions.Identity.Core` deliver compile and runtime assets, which *do* cross the
> reference, so §7.5, §7.6 and §7.8 declare each in the contracts project alone. `Microsoft.Playwright` sits with the build-asset group rather than the library
> group, for the reason §7.7 states.

The property that *is* load-bearing is different, and it is the one the topology is built on: **a test
method inherited by a class declared in the assembly under test is discovered on that derived class**,
because discovery enumerates each declared type's methods including the inherited ones. So the shape is
fixed:

| Element | Where it is declared | What it carries |
| --- | --- | --- |
| One `public abstract` class **per contract surface** | `src/MvcMusicStore.Contracts.Tests/Contracts/**` | Every `[Fact]` and `[Theory]` for that surface, and nothing runtime-specific. Its dependencies arrive through an **injected, runtime-neutral context**. Being abstract, it is skipped by discovery and contributes no run of its own |
| One `public sealed` **legacy-bound** concrete per surface | `src/MvcMusicStore.Contracts.Tests/Legacy/**` | A derivation, the legacy context and its `[Collection]` attribute. No assertion logic. These are the classes the pre-port baseline run discovers and executes |
| One `public sealed` **target-bound** concrete per surface, plus the **target-only** classes that have no legacy counterpart | `src/MvcMusicStore.Tests/Core/**` | A derivation, the target context — the in-process host's client and the target-side observer — and its `[Collection]` attribute. No assertion logic. These are the classes the post-port run discovers and executes |
| One `public` **collection-definition class per surface group**, in **each** assembly that declares a class in that group | `src/MvcMusicStore.Contracts.Tests/Collections/*.cs` and `src/MvcMusicStore.Tests/Collections/*.cs` | `[CollectionDefinition]` with the group's name, `ICollectionFixture<TFixture>` naming **the fixture type that group's classes bind in that assembly**, and a `const string` holding the name. **No test methods.** The groups are [05 §12.7](05-aspnet-core-migration-approach.md)'s nine, plus the deployed group of the row below; a definition is per-assembly because `[Collection]` names are resolved within the assembly being run, so a definition in one assembly names no collection in the other. The contracts assembly declares **two** fixture types, so its definitions do not all name the same one |
| The **`public` collection-fixture type** each definition binds | `src/MvcMusicStore.Contracts.Tests/LegacyBaselineFixture.cs` and `src/MvcMusicStore.Tests/CoreApplicationFixture.cs` | One instance per collection, so the nine groups get nine databases rather than one shared engine state. What each fixture *does* — provisioning, the two-database legacy reset, `ResetAsync()` and the per-test `IAsyncLifetime` binding — is [05 §12.7](05-aspnet-core-migration-approach.md)'s and is not restated here |
| The **`public sealed` deployed-only pair** — a third fixture type and one concrete — `DeployedEndpointFixture` and `DeployedEndpointTests` | Both at the root of `src/MvcMusicStore.Contracts.Tests/`, with their `[CollectionDefinition]` beside the others | The one runtime-neutral context in the topology that **hosts nothing**: no in-process host, no engine, no database, no observer and no reset — a **consumed base address** and a client whose **redirect following is disabled**. The concrete holds its cases directly rather than deriving from a base, because there is no legacy counterpart to share one with. It exists because the one `Category=Deployed` case runs against a **deployed** host, which is a context neither row above can produce; its execution stage, its authoring workstream and its assertions are named in §12.2's two rows and in [05 §12.6](05-aspnet-core-migration-approach.md) |

**Every type in that table is `public`, and that is a requirement rather than a house style.** Three
distinct mechanisms fail on a non-public one, and two of the three fail **silently**, which is why the
accessibility is fixed here rather than left to whoever types the first class.

- **A concrete in one assembly cannot derive from a base the other declares `internal`.** The two test
  projects are separate assemblies with no `InternalsVisibleTo` between them, and adding one so that a test
  topology compiles would be a worse answer than declaring six bases public. This one fails loudly, at
  build time — it is the only one that does.
- **The runner discovers test classes only when they are public.** A concrete declared `internal`
  compiles, is never enumerated, and reports nothing: the same green-run-of-zero-cases failure as the first
  blockquote above, arriving through accessibility instead of through the reference.
- **Collection definitions and the fixture types they name are located and activated by reflection, and
  the documented shape for both is a public type.** Declaring either otherwise makes the binding depend on
  unspecified runner behaviour, and its failure mode is a fixture that is never attached — which surfaces
  as a null context inside an assertion rather than as a wiring error.

The same requirement reaches every type that crosses the project boundary for the same first reason: the
**runtime-neutral context** the bases take, and the `IStoreObserver` abstraction whose two implementations
sit one per project ([05 §12.6](05-aspnet-core-migration-approach.md)). A type a second assembly must
*name* cannot be internal to the first.

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

Which classes each stage runs, and how they are selected, is not this document's to specify: the trait
categories and the two commands are [05 §12.10](05-aspnet-core-migration-approach.md)'s, and
[03](03-modernization-roadmap.md)'s W4 and W7 gates name the stage each belongs to. What this section fixes
is the declaration site of every class involved, because that is a project-structure decision and the
map above is where project structure is settled.

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
| A **shared composition project** — a fourth project holding the service registrations | §12.3 | Two `ProjectReference` edges give the same single-implementation property with nothing added to the map |
| An **`IDesignTimeDbContextFactory` implementation** | §13.3 | It would be a second composition, which is the very drift that makes a generated migration untrustworthy; the environment contract achieves determinism without a new file |
| A **launch-profile file** — `Properties/launchSettings.json` | §5.4, §12.8 and the note below | A developer convenience that nothing in this strategy requires; the one thing it would have carried that is *not* a convenience is declared in configuration instead (§12.8) |
| A **`.dockerignore`** | §4.4 of [06](06-azure-hosting-recommendations.md), and the note below | The conditional `Dockerfile` is the only container artifact the map contains; the build context is constrained inside it rather than by a second file |
| A **second container image** — a migration or tooling image beside the conditional site image | §6.9.1 of [06](06-azure-hosting-recommendations.md), and the note below | The container path's migration route is a **runner inside the virtual network**, not an image: 06 §6.9.1 selects it and its three mechanisms are runner hosts and platform-started jobs, none of which is a repository artifact. **The map contains exactly one image-producing file, the conditional `Dockerfile`** |
| A **second operator project** — a separate console project for the data phases or for seeding | §12.2, §12.3 | Those are **verbs of the one mapped operator project**, sharing its host, its edge, its lockfile, its publish output and its promotion path. An operator capability is not a map row |

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
question is about things that are not.** The rule has two limbs, and between them they cover every
"surely *this* has to be in the map" a reader of §12.2 can raise.

**Limb one: this map is closed against *repository* artifacts, and it is not a map of platform resources at
all.** The target design requires Azure resources that the platform owner provisions and that exist in no
repository — an App Service plan or Container Apps environment, an Azure SQL database, managed identities,
a Key Vault, a Log Analytics workspace, the migrate-stage runner or job of the note above, and the
**automation behind the interim credential rotation of
[06](06-azure-hosting-recommendations.md) §5.3**, which needs a **function app, the Event Grid
subscription that triggers it and the storage account such a function requires**.
**A required Azure resource is not a map entry**, and its absence from §12.2 is not a gap: it is the
boundary between what a repository contains and what a subscription contains.
[06](06-azure-hosting-recommendations.md) owns every one of them, together with its configuration and its
provisioning order. The practical consequence for a reader of §12.2 is direct — **a platform resource named
in 06 is not to be looked for as a folder here**, so nothing above is missing a function project, a Bicep
file or a deployment template. Turning each such resource into a repository artifact is exactly the
invention the closed-map rule exists to prevent, and it would put infrastructure-as-code in a map whose
own scope statement (§12.1) is project format, dependencies, tooling manifests and solution structure.

**Limb two: a required *operator capability* becomes a verb of the mapped operator project, never a new
project.** The target needs operator actions beyond administrator provisioning — the catalog data phase,
the Identity data phase and non-production seeding — and each of them needs a host outside the web
pipeline and the application's own registrations. That is a capability requirement, and the map answers it
with the project it already contains: §12.3 states the four verbs, the single edge, the one lockfile, the
one publish output and the one promotion path they share. **So an operator capability named in 05 or 06 is
not to be looked for as a fourth project here**, and the project count stays three. The distinction
between the two limbs is worth naming, because they resolve the same reader question in opposite
directions: a platform resource is excluded because it is **not a repository artifact at all**, while an
operator capability is excluded because it is **already a mapped one**. Neither is a licence to add a row,
which is the property that keeps the map closed.

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

### 12.9 The two tool projects reference the web application, and are published separately

The rows in §12.2 for `tools/provision-admin` and `tools/migrate-data` each carry a `ProjectReference` to
`src/MvcMusicStore/MvcMusicStore.csproj`, and both are excluded from the web application's publish output.
Those two facts belong together, because the first is what makes the tools correct and the second is what
keeps them out of the deployed application. Both are project-structural, which is why they are stated here
rather than in a sibling document.

**Why the reference is required rather than convenient.** Neither tool is a script that talks to a
database; each is a program that has to agree with the application about types the application owns.

| Tool | What it must compile against | What it would otherwise have to duplicate |
| --- | --- | --- |
| `tools/provision-admin` | `ApplicationUser`, the Identity `DbContext`, and the registration seam it resolves `UserManager<ApplicationUser>` and `RoleManager` from | The Identity model and the container registrations. A tool that hand-rolled either would hash a password under a configuration the application does not have — and direct SQL cannot produce a valid Identity password hash at all |
| `tools/migrate-data` | **Both** `DbContext` types, `ApplicationUser`, **both** migration sets, and the registration seam | The entity model *and* the migration history. Its `schema-diff` mode generates the DDL the migration sets would apply, so a duplicated model would compare the target schema against a copy of the model rather than the model the application ships |

**"The composition root" is not something a reference gives you, so the seam is named rather than implied.**
This is the part most easily left as a gap, because the reference makes every *type* available and it is easy
to assume the registrations come with them. They do not. The web project's composition root is written with
top-level statements, and the one public type that arrangement exposes —
`public partial class Program { }` (§12.2) — exists **solely** so `WebApplicationFactory` can locate an
entry point. It declares no members, exposes no `IServiceCollection` and offers a console host nothing to
call: a tool that referenced the web project and stopped there would still have to write its own
registrations, which is precisely the duplication the reference was taken to avoid.

The seam is therefore an artifact with a name, mapped in §12.2 and owned by the web project:

| Element | Value |
| --- | --- |
| Owning project and file | `src/MvcMusicStore/` — `ApplicationServices.cs` |
| Type | `public static class ApplicationServices` |
| Method | `public static IServiceCollection AddMvcMusicStoreServices(this IServiceCollection services, IConfiguration configuration)` |
| Registers | Both `DbContext` types on the SQL Server provider; the Identity core, store and manager services over `ApplicationUser`, including the role manager, the password hasher and the Identity options; and the options objects the tools read, bound from the supplied configuration |
| Called by | **`Program.cs` itself**, and each tool's host builder — nothing else, and nothing bypasses it |

**One registration path, which is the whole point.** Because the web application composes itself *through*
this method rather than beside it, a tool that calls it holds the same context configuration, the same
Identity options and the same password hasher as the running application. There is no second place where a
registration could be added and no way for a tool's view of the application to be a subset of the real one
that still compiles. What each tool then does is ordinary hosting:

| Tool | What it does with the seam |
| --- | --- |
| `tools/provision-admin` | Builds a host, calls `AddMvcMusicStoreServices` with its own configuration, and resolves **`UserManager<ApplicationUser>` and the role manager** from the resulting container — which is what makes the password hash it writes a hash the application will accept. [05 §10.2](05-aspnet-core-migration-approach.md) owns the command's five required properties |
| `tools/migrate-data` | Builds a host the same way and resolves **both `DbContext` types and both migration sets**, which is what its `schema-diff` mode compares the live schema against and what its load modes write through. [05 §5.7](05-aspnet-core-migration-approach.md) owns its six modes and its exit-code contract |

**The pipeline concerns are deliberately outside the seam**, and the exclusion is as load-bearing as the
inclusion: MVC and Razor, the cookie authentication schemes, session, anti-forgery, data-protection key
persistence, health checks and the middleware order stay in `Program.cs`. A console host has no request
pipeline to attach them to, so putting them in the shared method would make every tool configure behaviour
it never exercises — and would make a tool's startup fail on a misconfiguration in a subsystem it does not
use.

**The failure mode a duplicated type produces is drift, and drift here is silent.** A tool holding its own
copy of the entity model keeps building and keeps exiting `0` after the application's model changes; what
it writes is simply no longer what the application reads. The project reference makes that class of
divergence a **compile error in the tool** at the moment the application changes, which is the only point
at which it is cheap to find. The direction is one-way and stays that way: the tools reference the web
application, the web application references neither tool, and the reference graph therefore has no cycle.

**The same argument applies to the registrations, and it is the half a reference alone does not cover.**
Duplicated *types* fail to compile; duplicated *registrations* compile perfectly and drift in silence — a
tool that registered its own `DbContext` with its own options keeps working after the application changes a
connection-string key, a provider option or a password-hasher setting, and writes rows or hashes the
application then reads under different assumptions. Nothing fails and nothing is logged. The seam is what
removes that class of divergence: there is one registration method, the application itself calls it, and a
change made there reaches every caller at once. A tool that stopped calling it would not silently diverge
either — it would have nothing in its container and fail at resolution, loudly, on its first run. **The
one-way direction holds for the seam too**: the tools call into the web project's registration method, the
web project calls nothing of theirs, and it references neither.

**No shared library is introduced, and that is a decision rather than an omission.** Extracting the
contexts and the model into a common project would work, and it buys nothing here: there is one owner of
those types, one repository, no consumer outside it, and no second application to share them with — which
is the same reason §2.1 records no `netstandard` shim anywhere in the target. A shared library would add a
project, a lockfile and a versioning question in order to solve a problem the project reference already
solves.

**Why the exclusion from the web publish follows automatically, and what the pipeline must therefore do.**
The reference points from each tool **to** the web application, and publish output follows references in
that direction only: `dotnet publish` of the web project has no reason to emit an assembly belonging to a
project that merely consumes it, and the web project references no tool. So the tools are **not** in the
web application's publish output, which is the separation [05](05-aspnet-core-migration-approach.md) and
[06](06-azure-hosting-recommendations.md) both require — a release-time console tool holding DDL rights
must not ship inside the running web application. The consequence for the build is the part this document
owns: **each tool project is built and published on its own**, and its output is staged where the release
step can invoke it. A pipeline that publishes only the web project produces no tool to run, and the
deployment-time migration step [06 §6.2](06-azure-hosting-recommendations.md) places in the release path
then has nothing to invoke.

**One invocation form, used by both callers.** Both tools are invoked identically by the test suite and by
the release pipeline:

```bash
# The form both callers use. <mode> is one of the tool's own modes; switches are the tool's own.
dotnet <published-output>/MigrateData.dll <mode> [switches]
dotnet <published-output>/ProvisionAdmin.dll <mode> [switches]

# Developer convenience only - never the form a pipeline or a test uses.
dotnet run --project tools/migrate-data -- <mode> [switches]
```

Three properties of that form are requirements, not style:

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
  the pipeline stage or the release step. The three codes the migration tool exits with are
  [05 §5.7](05-aspnet-core-migration-approach.md)'s and are not restated here; the operator tool's
  behaviour is [05 §10.2](05-aspnet-core-migration-approach.md)'s. What this row requires either way is
  that the code be read: a caller that discards it converts a refusal into an apparent success, which is
  the one outcome those refusals exist to prevent.

The test-side invocation — which cases drive which mode, and what they assert about its verdict — is
[05 §12.9](05-aspnet-core-migration-approach.md)'s; the release-side step and the principal it runs as are
[06 §6.2](06-azure-hosting-recommendations.md)'s. Neither is restated here.

---

### 12.4 The project graph — the three edges and the two visibility changes without which the solution does not work

§5.6 and §12.2 name three projects. Three projects in one solution are not a graph, and no consumer reaches
the project it consumes by being in the same solution: a `ProjectReference` and a type visibility are what
make each one compile. Both are project-structural facts, so this document owns them; what the fixture and
the command *do* is [05](05-aspnet-core-migration-approach.md)'s.

| Edge | Declared in | Element |
| --- | --- | --- |
| `MvcMusicStore.Tests` → the web application | `src/MvcMusicStore.Tests/MvcMusicStore.Tests.csproj` | `<ProjectReference Include="..\MvcMusicStore\MvcMusicStore.csproj" />` |
| `MvcMusicStore.Tests` → the operator command | `src/MvcMusicStore.Tests/MvcMusicStore.Tests.csproj` | `<ProjectReference Include="..\..\tools\provision-admin\ProvisionAdmin.csproj" />` |
| `tools/provision-admin` → the web application | `tools/provision-admin/ProvisionAdmin.csproj` | `<ProjectReference Include="..\..\src\MvcMusicStore\MvcMusicStore.csproj" />` |
| The web application → anything in this solution | — | **None.** The web project references neither of the other two |
| The operator command → the test project | — | **None.** The edge is one-way, so the command's output contains no test assembly |

**The second edge exists for exactly one reason, and the reason is a test that is otherwise not
executable.** The dispatcher below refuses an argument containing any control character, `U+0000` among them,
and that refusal has to be asserted. **A NUL cannot be delivered through a process boundary**: Windows
`CreateProcess` takes a single NUL-terminated command line and the POSIX `execve` family takes a
NUL-terminated `char *` per argument, so on either platform the first NUL byte in an argument *is* the
terminator and the character never reaches `Main(string[] args)` to be refused. A test that launches the
command as a process and passes `"--user"`, `"a\0b"` therefore proves nothing: the runtime either throws at
the launcher or the callee observes `a`. The only boundary at which the refusal is observable is the
**dispatcher's own signature**, `Cli.Dispatch(string[])`, called in-process with an array the test
constructs — which requires the test assembly to reference the tool assembly. That is the edge. Every other
refusal class is reachable through argv and is asserted that way, by launching the built tool as a process.

**What the edge costs, stated rather than discovered.** The tool's transitive graph enters the test
project's: because `tools/provision-admin` references the web application (edge three), the test project
already had the web application and the ASP.NET Core shared framework in its graph, so the added edge
brings **no new framework reference and no new package** — it brings the tool's own compiled types, which
is the point. It also creates no cycle: the tool does not reference the test project, and nothing
references the test project at all.

**The direction still matters as much as the existence.** No edge points *at* the test project, and none
points away from the web project, so `dotnet publish` of the web project produces output containing neither
the test project nor the operator command, and `dotnet publish` of the operator command produces output
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

**The second visibility change: `Cli` and its `Dispatch` method must be public in the operator command.**
The dispatcher is the boundary the NUL case is asserted at, so the test assembly has to name `Cli` and call
`Dispatch(string[])`. In a console project the compiler makes nothing public by default, and the type as
this section declares it is internal to the tool. The same two mechanisms are available and the same one is
chosen, for the same reason:

- **Chosen: `Cli` and `Dispatch` are declared `public`, and nothing else in the tool is.** Two modifiers in
  `tools/provision-admin/Program.cs` — where this section already puts the dispatcher — and no new file.
  What becomes visible is exactly the surface the assertion needs and no more: the exit-code constants, the
  token tables, the `Refusal` enumeration and `Refuse` all stay **internal**, because the assertion reads
  the refusal the way an operator does — `Dispatch` returned `null` and the captured `Console.Error`
  carries `CLI-2013` — which is the same published contract the process-level cases assert against, so no
  additional type has to be exposed to state it.
- **Rejected: `<InternalsVisibleTo Include="MvcMusicStore.Tests" />` in the operator project.** It grants
  the test assembly every internal type in the tool — including the credential-handling members the
  credential contract below deliberately keeps out of reach of anything that logs — in order to expose one
  dispatcher. The narrower change is available, so the broader one is not taken.

**Why the `ProjectReference` from the test project is required rather than merely convenient.** Beyond
naming `Program`, the fixture has to find the web application's **content root**, because Razor views are
resolved relative to it and the test process runs from its own output directory. `Microsoft.AspNetCore.Mvc.Testing`
(pinned at `8.0.30`, §7.2) ships build targets that emit a content-root manifest for the web projects the
test project references, and `WebApplicationFactory` reads it. Without the reference there is no manifest
entry, and the fixture falls back to guessing a path — the failure mode being views that cannot be found
at run time rather than a compile error.

**The sequencing consequence, which follows from the structure rather than from anyone's plan.** Every
requirement above is a property of the **test project**, not of any individual test: without its two
`ProjectReference` elements, without a `Program` it can name and without a `Cli` it can call,
`MvcMusicStore.Tests` does not compile at all, whatever it contains. So **no part of the suite can be compiled before this project graph exists** — including
the rows that only ever talk to the *legacy* application over HTTP, because they are compiled into the same
assembly as every other row. This document contributes that structural fact and states no sequence of its
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
if (cli is null) return Cli.Usage;                         // refused: no verb ran, nothing written

var builder = Host.CreateEmptyApplicationBuilder(new HostApplicationBuilderSettings
{
    EnvironmentName = Env.EnvironmentName(),   // absent => "Production", which fails closed
});

builder.Configuration
    .AddInMemoryCollection(Defaults.FailClosed)   // precedence 1 — see the table below
    .AddInMemoryCollection(Env.Admitted())        // precedence 2 — three variables, by exact name
    .AddCommandLine(cli.Value.Pairs);             // precedence 3 — the dispatcher's pairs only
```

**Precedence 2 is an exact key allow-list and not a prefix, because a prefix is an open door with a
narrow frame.** `AddEnvironmentVariables(prefix: "MUSICSTORE_TOOL_")` admits **every** variable carrying
that prefix, so any name an operator, an agent image, a pipeline definition or an earlier step happens to
export becomes configuration for a privileged command — and `__` translation means
`MUSICSTORE_TOOL_Provisioning__RevokeRole=true` in the ambient environment resolves the key
`Provisioning:RevokeRole` at a precedence **above** the fail-closed defaults. A run that typed no switch
would then withdraw a role, and nothing in the invocation would say so. Naming the variables instead
removes the class rather than the instance:

```csharp
static class Env
{
    // The whole admitted surface. Three variables, matched by exact ordinal name.
    // No prefix scan, no translation, no fourth entry, and no way to add one at run time.
    private static readonly (string Variable, string Key)[] Admissible =
    {
        ("MUSICSTORE_TOOL_ConnectionStrings__MusicStore",       "ConnectionStrings:MusicStore"),
        ("MUSICSTORE_TOOL_Provisioning__AdministratorPassword", "Provisioning:AdministratorPassword"),
        ("MUSICSTORE_TOOL_Seeding__Enabled",                    "Seeding:Enabled"),
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

    // Read directly into HostApplicationBuilderSettings; deliberately NOT a configuration key,
    // so no configuration source can restate the environment the guards of §12.6 test against.
    internal static string? EnvironmentName() =>
        Environment.GetEnvironmentVariable("MUSICSTORE_TOOL_Environment");
}

static class Defaults
{
    internal static readonly Dictionary<string, string?> FailClosed = new(StringComparer.Ordinal)
    {
        ["Seeding:Enabled"]                = "false",
        ["Provisioning:RevokeRole"]        = "false",
        ["Provisioning:RotateCredential"]  = "false",
    };
}
```

**The mode is decided by the switches and by nothing else, and that is enforced twice.** The two
privilege-affecting keys — `Provisioning:RevokeRole` and `Provisioning:RotateCredential` — are
(1) **not on the allow-list above**, so no environment variable of any name can carry them, and
(2) **stated explicitly by the dispatcher on every run**, `true` when the operator typed the switch and
`false` when they did not, at the highest precedence. Either mechanism alone would close the hole; both
are kept because they fail in different directions. The allow-list is what stops an ambient value
arriving at all, and the unconditional emission is what makes precedence decide the question even if some
future source is added below it — a source that would otherwise silently acquire authority over a
role withdrawal.

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
§7.1 pins — and an empty builder handed `Args = ["--Provisioning:AdministratorPassword=…"]` resolves that value
from `builder.Configuration`, defaults disabled or not. The command line is instead added **explicitly,
last**, and only over the argument array the dispatcher below has already closed over.

`Host.CreateEmptyApplicationBuilder` — equivalently `new HostApplicationBuilder(new
HostApplicationBuilderSettings { DisableDefaults = true })` — starts with **no configuration source and no
logging provider**, and every one of both is then added deliberately. An earlier form of this section named
`Host.CreateApplicationBuilder(args)` and described its sources as "environment variables and command-line
arguments". **That description is wrong, and the difference matters for this process specifically.** The
defaulted builder also adds `appsettings.json` and `appsettings.{Environment}.json` **resolved against the
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
| 1 (lowest) | `AddInMemoryCollection(Defaults.FailClosed)` — the tool's own defaults | Three entries and no fourth: `Seeding:Enabled` = `false`, `Provisioning:RevokeRole` = `false`, and `Provisioning:RotateCredential` = `false`. A default that authorizes nothing is what makes an *absent* setting safe rather than merely undefined. For seeding it means no write. For the two mode flags it is **defence in depth rather than the deciding source** — precedence 3 states both on every run — and it is retained so that a composition which somehow reached this root without the dispatcher's pairs would still read `false` for each. `RotateCredential` reading `false` is what makes an ordinary release run leave an existing administrator credential alone ([05 §10.2](05-aspnet-core-migration-approach.md) property 3) |
| 2 | `AddInMemoryCollection(Env.Admitted())` — **exactly three environment variables, by name** | `MUSICSTORE_TOOL_ConnectionStrings__MusicStore` → `ConnectionStrings:MusicStore`; `MUSICSTORE_TOOL_Provisioning__AdministratorPassword` → **the credential**, which is [05 §10.2](05-aspnet-core-migration-approach.md) property 1's environment-variable channel, scoped by the pipeline to the single step; `MUSICSTORE_TOOL_Seeding__Enabled` → `Seeding:Enabled` (§12.6). **No other variable reaches configuration**, whatever it is named and whatever prefix it carries, and neither mode flag is admissible here at all. The environment *name* is not in this source either — it is read directly into `HostApplicationBuilderSettings.EnvironmentName`. The credential contract below names the variable and fixes the acquisition order |
| 3 (highest) | `AddCommandLine` over **the dispatcher's normalized pairs**, never over `args` | Only the non-secret operands the dispatcher below admits: `Provisioning:Actor`, `Provisioning:UserName` and `Provisioning:RoleName` when their tokens are given, **plus `Provisioning:RevokeRole` and `Provisioning:RotateCredential` on every run without exception** — `true` where the switch was typed, `false` where it was not. The verb is **not** a configuration value at all — it selects the code path in the dispatcher and never reaches a provider |
| — | **No JSON file source of any kind** | `appsettings.json` in the working directory is **never read**, because no provider reads it. This is the property the first required assertion below exercises |
| — | **No prefix-scanning environment source, and no `AddEnvironmentVariables` call anywhere** | The whole of the process environment other than the three named variables is invisible to this host. This is the second half of what the first required assertion exercises, and it is the property that makes the mode un-settable by ambient state |

**The command line cannot be interpreted by a configuration provider at all, so a closed dispatcher
interprets it first.** This is the part of the design a switch map looks like it covers and does not. The
four facts below were **measured** against `Microsoft.Extensions.Configuration.CommandLine` at the `8.0.30`
servicing band §7.1 pins, not inferred from prose, and each one breaks a verb-dispatching tool differently:

| Token form | What `AddCommandLine` does with it | Consequence for a tool that dispatched on configuration |
| --- | --- | --- |
| A bare verb — `provision-admin` | **Dropped silently.** A token carrying no `-`, `--` or `/` prefix and no `=` is skipped, so `["provision-admin", "--actor", "op"]` produces exactly one key, `Provisioning:Actor` | The verb **never arrives**. No verb runs on any invocation, however it is spelled |
| A valueless switch followed by another switch — `--revoke-role --actor op` | **The following token becomes its value**: the result is `Provisioning:RevokeRole` = `--actor`, and `Provisioning:Actor` is absent | Silent mis-binding — a truthy-looking revoke mode *and* a swallowed actor, with no error at any layer |
| The same switch in trailing position — `… --revoke-role` | **Dropped silently**, there being no following token to consume | The revoke mode disappears and a run meant to withdraw access grants instead |
| Any `--key=value`, whether the map names it or not | Becomes configuration key `key` | `--password=…` becomes configuration, which is why the refusal below cannot be delegated to the map |

The switch map is therefore not the enforcement point and cannot be made into one. **The dispatcher is it**:
it runs **before the configuration root is built**, it maps the admitted tokens to configuration keys
itself, and it is **closed** — every token is admitted by name or the invocation is refused. `AddCommandLine`
then receives only pairs it cannot misread, and no switch map is passed to it, because there is nothing
left for one to translate.

```csharp
public static class Cli          // public: S12.4's second visibility change, and the ONLY
{                                // public member is Dispatch, below. Everything else stays
    internal const int Success = 0;   // internal, because only Program reads it.
    internal const int Failed  = 1;   // the verb ran and refused, or failed
    internal const int Usage   = 2;   // the invocation was refused; no verb began

    internal const string Provision   = "provision-admin";
    internal const string Seed        = "seed-catalog";
    internal const string RevokeToken = "--revoke-role";
    internal const string RotateToken = "--rotate-credential";

    internal static readonly string[] Required = { "--actor", "--user", "--role" };

    internal static readonly Dictionary<string, string> Switches = new(StringComparer.Ordinal)
    {
        [RevokeToken] = "Provisioning:RevokeRole",
        [RotateToken] = "Provisioning:RotateCredential",
    };

    internal static readonly Dictionary<string, string> Valued = new(StringComparer.Ordinal)
    {
        ["--actor"] = "Provisioning:Actor",
        ["--user"]  = "Provisioning:UserName",
        ["--role"]  = "Provisioning:RoleName",
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
        VerbMissing, VerbUnrecognized, PasswordBearingToken, SeedTakesNoArgument,
        TokenRepeated, SwitchTakesNoValue, TokenUnrecognized, ValueMissing,
        ValueLooksLikeSwitch, ValueEmpty, ModesMutuallyExclusive, RequiredTokenMissing,
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
        [Refusal.SeedTakesNoArgument]    = ("CLI-2004", "the seed verb takes no argument"),
        [Refusal.TokenRepeated]          = ("CLI-2005", "token given more than once"),
        [Refusal.SwitchTakesNoValue]     = ("CLI-2006", "mode switch given a value"),
        [Refusal.TokenUnrecognized]      = ("CLI-2007", "unrecognized token"),
        [Refusal.ValueMissing]           = ("CLI-2008", "valued token given no value"),
        [Refusal.ValueLooksLikeSwitch]   = ("CLI-2009", "valued token followed by a switch instead of a "
                                                     + "value; use token=value for a value beginning '-'"),
        [Refusal.ValueEmpty]             = ("CLI-2010", "valued token given an empty value"),
        [Refusal.ModesMutuallyExclusive] = ("CLI-2011", RevokeToken + " and " + RotateToken
                                                     + " are mutually exclusive; a revoke runs no "
                                                     + "credential operation"),
        [Refusal.RequiredTokenMissing]   = ("CLI-2012", "required token missing"),
        [Refusal.ControlCharacter]       = ("CLI-2013", "argument contains a control character"),
    };

    // The whole output of every refusal, on one line. Three components and no fourth:
    // a compiled-in code, a compiled-in sentence, and — where they apply — an int the
    // dispatcher computed and a literal taken from `Required`. NOTHING is read from `args`.
    internal static (string Verb, string[] Pairs)? Refuse(
        Refusal cause, int position = -1, string? expected = null)
    {
        var (code, text) = Refusals[cause];
        var where  = position >= 0     ? $" at argument {position}" : string.Empty;
        var wanted = expected is null  ? string.Empty : $"; expected {expected}";
        Console.Error.WriteLine($"provision-admin: {code}: {text}{where}{wanted}.");
        Console.Error.WriteLine($"usage: ProvisionAdmin {Provision} --actor <a> --user <u> --role <r> "
                              + $"[{RevokeToken} | {RotateToken}]");
        Console.Error.WriteLine($"       ProvisionAdmin {Seed}");
        return null;
    }

    public static (string Verb, string[] Pairs)? Dispatch(string[] args)
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
        if (verb != Provision && verb != Seed) return Refuse(Refusal.VerbUnrecognized, 0);

        var pairs = new List<string>();
        var seen  = new HashSet<string>(StringComparer.Ordinal);

        for (var i = 1; i < args.Length; i++)
        {
            var token = args[i];
            var key   = KeyOf(token);

            if (IsPasswordBearing(key))
                return Refuse(Refusal.PasswordBearingToken, i);
            if (verb == Seed)
                return Refuse(Refusal.SeedTakesNoArgument, i);
            if (!seen.Add(key))
                return Refuse(Refusal.TokenRepeated, i);

            var attached = token.Length != key.Length;          // the "--key=value" form

            if (Switches.ContainsKey(key))
            {
                if (attached) return Refuse(Refusal.SwitchTakesNoValue, i);
                continue;                                       // recorded in `seen`; emitted once, below
            }
            if (!Valued.TryGetValue(key, out var mapped))
                return Refuse(Refusal.TokenUnrecognized, i);    // a second verb and a bare operand land here

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

        if (seen.Contains(RevokeToken) && seen.Contains(RotateToken))
            return Refuse(Refusal.ModesMutuallyExclusive);

        foreach (var required in Required)                       // `required` is a compiled-in literal
            if (verb == Provision && !seen.Contains(required))
                return Refuse(Refusal.RequiredTokenMissing, expected: required);

        // Both mode flags are stated on EVERY dispatched invocation, at the highest precedence,
        // so the mode is a function of the typed switches alone. Nothing ambient can decide it,
        // and nothing absent can leave it undecided.
        foreach (var (token, key) in Switches)
            pairs.Add($"{key}={(seen.Contains(token) ? "true" : "false")}");

        return (verb, pairs.ToArray());
    }
}
```

**It is total, and that is a property rather than a hope.** Every path through the token loop either
`continue`s — having added exactly one pair for a valued token, or nothing at all for a switch, which the
`seen` set has already recorded — or returns a refusal. There is **no wildcard fall-through and no default
admission**, so a token that is not named above cannot reach a configuration provider, and the dispatcher's
output is by construction a set of `key=value` pairs whose keys are drawn from the three in `Valued` plus
the two in `Switches` — five keys, and no sixth is reachable. **Both mode keys are always among them**: the
loop records the switches and the post-loop emission states both, so a dispatched invocation produces
between two and five pairs and never fewer than the two modes. That holds for `seed-catalog` too, which
admits no token and therefore emits both modes as `false`; the seed verb reads neither, and emitting them
uniformly is cheaper to reason about than a verb-conditional pair set. **Both switches go through one
branch on purpose**: a second hand-written `if` per flag is how the third one acquires a subtly different
refusal rule, and the `Switches` table makes adding a mode a one-line entry that inherits the no-value
check, the repeat check and the unconditional emission already proved here. The type is declared **after** the
top-level statements in
`tools/provision-admin/Program.cs`, next to the `public partial class Program { }` this section already
requires; C# admits no type declaration before them.

**One branch sits outside that loop, and it is the control-character scan — a refusal class rather than a
sanitizer.** It is stated as its own class, with its own code `CLI-2013`, for a reason the no-echo rule below
does not by itself supply: *not echoing* an argument stops a control character reaching the diagnostic
stream, but it does nothing about the same character reaching **configuration**, and from there a
`Provisioning:UserName` or `Provisioning:RoleName` value that an Identity operation, a `PROV-6001` structured
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

**Where the NUL case is asserted, and why it cannot be asserted anywhere else.** §12.4 states the boundary
fact and the project edge that follows from it: a NUL cannot cross a process boundary, because on Windows
`CreateProcess` takes one NUL-terminated command line and on POSIX `execve` takes NUL-terminated argument
strings, so the byte *is* the terminator and never arrives at `Main` to be refused. The refusal is therefore
asserted **in-process, against this method's own signature** — `Cli.Dispatch(new[] { "provision-admin",
"--actor", "op", "--user", "a\u0000b", "--role", "Administrator" })`, asserting the return is `null` and the
emitted code is `CLI-2013` — which is what the public surface above and the test project's second
`ProjectReference` exist for. Every other control character in the class travels through argv intact and is
asserted the ordinary way, by launching the built tool as a process (§12.4's assertion 2).

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
left open.** An admitted value — `--actor`, `--user`, `--role` — becomes a configuration value, and one of
them does reach a record: `Provisioning:Actor` is validated against
[05 §10.2](05-aspnet-core-migration-approach.md) property 4's closed character set after admission and
before use, and the audit record that carries it is written after the host exists, through the JSON console
provider below, which emits it as a **structured field** whose value is escaped rather than as text spliced
into a line. So the pre-host path carries no input at all, and the post-host path carries validated input
as data. Neither hands an argument to a text formatter.

**This is [09 §6.8.1](09-security-assessment.md)'s rule applied one layer earlier, deliberately and not by
coincidence.** That section closes the field set of every audit record and requires a failure outcome to
carry a **category code and nothing more**, with provider text and exception messages confined to the
diagnostic stream. A pre-host refusal has no audit record at all — nothing was attempted, so no
`PROV-6001` is emitted (the exit-code table below) — and its diagnostic is therefore the *only* thing the
run produces. Applying the same rule to it means the whole output of a refused invocation is a category
code, a fixed sentence and an integer. The `CLI-2001`–`CLI-2013` codes are **diagnostic codes and not
members of [09 §6.8.1](09-security-assessment.md)'s sixteen event classes**: they are deliberately given
their own `CLI-` family so that no reader and no log query mistakes a refused invocation for an audited
operation.

**Exit codes — three, and a refusal has no side effect.**

| Code | Meaning |
| --- | --- |
| `0` | The verb ran and **succeeded** |
| `1` | The verb ran and **refused or failed** — a §12.6 guard check, an Identity operation, [05 §10.2](05-aspnet-core-migration-approach.md) property 3's published-credential refusal, or the non-interactive credential refusal of the credential contract below. What was written is exactly what the run's records state |
| `2` | **The invocation was refused before any verb began** — every case `Cli.Dispatch` returns `null` for. No configuration root is built, no connection is opened, nothing is written, and no `PROV-6001` record is emitted, because no operation was attempted |

**The admitted surface, whole. There is no other token.**

| Token | Verb | Produces | Required |
| --- | --- | --- | --- |
| `provision-admin` \| `seed-catalog` | — | Nothing. The verb selects the code path and **never becomes configuration** | Exactly one, as `args[0]` |
| `--actor <value>` | `provision-admin` | `Provisioning:Actor` | **Yes** — [05 §10.2](05-aspnet-core-migration-approach.md) property 4 makes it required, and the value is validated against that property's closed character set *after* the dispatcher admits the token |
| `--user <value>` | `provision-admin` | `Provisioning:UserName` | **Yes**, and the value is validated against **[05 §6.1](05-aspnet-core-migration-approach.md)'s user-name policy** — the *application's* policy, which that deliverable owns and which is the only one in force. It is stated there and not restated here, and the point of citing it from a token table is the failure it prevents: validating the framework's broader default instead would let this command create an account the application itself refuses to accept, so the tool's admitted set has to be the application's set. As with `--actor`, validation happens **after** the dispatcher admits the token and before the value is used |
| `--role <value>` | `provision-admin` | `Provisioning:RoleName` | **Yes**, in both modes: the revoke mode of [05 §10.2](05-aspnet-core-migration-approach.md) property 3a removes a *named* membership |
| `--revoke-role` | `provision-admin` | `Provisioning:RevokeRole` = `true`, **normalized by the dispatcher** — the token itself is never forwarded | No. Absent, **the dispatcher states `false` for it explicitly**, at a precedence no environment variable can reach, and the provisioning mode is selected |
| `--rotate-credential` | `provision-admin` | `Provisioning:RotateCredential` = `true`, normalized the same way | No, and **mutually exclusive with `--revoke-role`** — a run passing both is refused, because a revoke runs no credential operation. Absent, the dispatcher states `false` for it explicitly and the credential of an account that already has one is left untouched, which is [05 §10.2](05-aspnet-core-migration-approach.md) property 3's `AlreadyPresent_NotRotated`; present, that property's rotation path runs |
| *(none)* | `seed-catalog` | — | The seed verb admits **no argument at all**: its three inputs arrive through the environment source (§12.6) and any token with the seed verb is refused |

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

**The forms that dispatch — and they are the only ones.** Every invocation shown anywhere in this document
is one of the six spellings below, the fifth covering all three provisioning modes; §12.6's table repeats
the seed forms rather than varying them:

```text
dotnet run --project tools/provision-admin -- provision-admin --actor <actor> --user <username> --role <role>
dotnet run --project tools/provision-admin -- provision-admin --actor <actor> --user <username> --role <role> --rotate-credential
dotnet run --project tools/provision-admin -- provision-admin --actor <actor> --user <username> --role <role> --revoke-role
dotnet run --project tools/provision-admin -- seed-catalog
ProvisionAdmin provision-admin --actor <actor> --user <username> --role <role> [--rotate-credential | --revoke-role]
ProvisionAdmin seed-catalog
```

The `--` in the `dotnet run` forms is `dotnet run`'s own separator: without it the verb and its switches are
read by the SDK rather than passed to the program. The published-output forms are what
[06](06-azure-hosting-recommendations.md) §12.1's provisioning stage and §9.5's sanctioned path invoke, and
[05 §10.2](05-aspnet-core-migration-approach.md)'s operations are what each one performs.

**The credential contract — the exact names, because "an environment variable" is not a variable name.**
[05 §10.2](05-aspnet-core-migration-approach.md) property 1 establishes the two channels and selects the
environment variable for the release process. It does not name it, and a channel with no name cannot be
wired by a release stage, so the names are fixed here alongside the rest of this host's configuration
surface and are **the names every other document cites**:

| Property | Value |
| --- | --- |
| **Configuration key** | `Provisioning:AdministratorPassword` |
| **Environment variable** | `MUSICSTORE_TOOL_Provisioning__AdministratorPassword` |
| **Which runs read it** | `provision-admin` in **provisioning mode only**. The `--revoke-role` mode never reads it — it touches no credential ([05 §10.2](05-aspnet-core-migration-approach.md) property 3a) — and `seed-catalog` never reads it. A run in either of those modes is unaffected by the variable's presence or absence |
| **When it is read** | **After** the dispatcher has admitted the invocation and the host is built, and **before the first store write**, so an absent credential cannot leave a half-provisioned account |

**The name is matched exactly, so it is the name and not a pattern that has to be right.** Precedence 2 is
the allow-list above: it compares `MUSICSTORE_TOOL_Provisioning__AdministratorPassword` ordinally and maps
it to `Provisioning:AdministratorPassword` by the table entry, with **no prefix stripping and no delimiter
translation performed by any provider**. A misspelling therefore matches nothing and the credential is
silently absent — which the acquisition order below turns into an explicit refusal rather than a partial
run, and which required assertion 6 exercises against the real variable.

The name still carries the `MUSICSTORE_TOOL_` prefix and the `__` delimiter, and both are deliberate now
that nothing derives meaning from them. The prefix keeps the tool's variables visibly namespaced in a
pipeline definition and in an agent's environment listing, so a reviewer can see at a glance which
variables belong to this command; `__` is .NET's conventional environment-variable hierarchy delimiter, so
the name reads the same as the configuration key it maps to; and `:` is avoided because it is not portable
in a variable name on the Linux agents [06](06-azure-hosting-recommendations.md) §3.2 selects. The names
are also the ones [05 §5.4](05-aspnet-core-migration-approach.md) and §12.6 already cite, which is the
stronger reason not to change their spelling: **the mapping moved from a provider convention to an explicit
table, and the names did not move at all.**

**Acquisition order — three steps, and the third is a refusal rather than a fallback.**

1. **The environment variable, if present and non-empty.** It is used as read, with no prompt offered even
   when a terminal is attached, so a pipeline run's behaviour never depends on how its agent allocates
   consoles.
2. **Otherwise, an interactive prompt with terminal echo disabled** — but **only** where `Console.IsInputRedirected`
   is `false`. The prompt reads keys with `Console.ReadKey(intercept: true)`, echoing nothing, and the same
   condition guards it that the API requires: `ReadKey` throws when stdin is redirected, which is exactly
   the case a pipeline job presents.
3. **Otherwise — absent and non-interactive — exit `1` without prompting and without partial effect.** The
   diagnostic **names the variable and never a value**, no store write has occurred, and no `PROV-6001`
   record is emitted for any operation, because none was attempted. A prompt written to a redirected stream
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

**Exactly one logging provider, and it is structured.** `builder.Logging.ClearProviders()` then
`builder.Logging.AddJsonConsole()`. Three reasons, in order of weight: the audit record's field set is
**closed** ([09 §6.8.1](09-security-assessment.md)), and a structured provider emits those fields as
log state — one JSON object per line — rather than as text somebody has to parse back out; the record is
collected from the process's stdout and stderr as a retained pipeline artifact
([06](06-azure-hosting-recommendations.md) §9.5), so exactly one provider means exactly one copy and no
second sink to reason about; and `ClearProviders()` is a **no-op today** — the empty builder registers none
— retained so that the one-provider guarantee survives any future change to what the builder starts with.

**Registrations — five, and each one is load-bearing.**

```csharp
builder.Services.AddDbContext<ApplicationDbContext>(o => o.UseSqlServer(connectionString));
builder.Services.AddDbContext<MusicStoreEntities>(o => o.UseSqlServer(connectionString));  // §12.6
builder.Services
    .AddIdentityCore<ApplicationUser>(o => { /* the Password options of 05 §6.1, set identically */ })
    .AddRoles<IdentityRole>()
    .AddEntityFrameworkStores<ApplicationDbContext>()
    .AddDefaultTokenProviders();
builder.Services.AddDataProtection().UseEphemeralDataProtectionProvider();
builder.ConfigureContainer(new DefaultServiceProviderFactory(
    new ServiceProviderOptions { ValidateScopes = true, ValidateOnBuild = true }));
```

- **`AddDefaultTokenProviders()` is not optional, and its absence is a run-time throw on the recovery
  path.** The existing-account repair of [05 §10.2](05-aspnet-core-migration-approach.md) property 3 calls
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
- It deliberately does **not** call `AddIdentity`, which the web application does use
  ([05 §2.4](05-aspnet-core-migration-approach.md) stage 1). `AddIdentity` registers cookie authentication
  schemes and a `SignInManager` — an HTTP-shaped composition with no meaning in a console process. The two
  compositions differ because the two hosts differ, and neither is a copy of the other.
- The **only** things it takes from the web project are **four types**: `ApplicationUser` and
  `ApplicationDbContext` for the provisioning verb, and `MusicStoreEntities` and `Data/SeedData` for the
  seed verb of §12.6. That is the whole reason for the tool's own `ProjectReference` — the third edge in
  the table above, and the only one this project declares. Each must be the web project's
  type and not a local re-declaration: a re-declared context maps to a different schema than the
  migrations produce, and a re-declared seed drifts from the catalog rows the suite's fixture asserts
  against.

**Six assertions are required of this composition.** Five of them cover failures that are invisible at
compile time, and two of those stay invisible until a privileged operation is already under way; the
remaining one *is* a compile check, because the one thing in this composition a reader can get wrong in a
way the runtime never gets a chance to report is the lifetime spelling above. They
are demonstrated by tests in `src/MvcMusicStore.Tests` — the project §12.2 already creates — driving the
**real entry point** in a real process-shaped configuration, which is the only way a host's own behaviour
is exercised at all. **One sub-case is the single exception and §12.4 states why**: the NUL refusal is not
observable at a process boundary, so that one call goes in-process to `Cli.Dispatch`, which is what the
second `ProjectReference` and the public dispatcher surface are for. They extend the command coverage
[05 §12.4](05-aspnet-core-migration-approach.md) already requires and **change no count that document
states**, because they assert properties of this composition rather than of the application's HTTP surface.

**Six assertions, five of them tests, and those numbers are held deliberately.** Several assertions below
carry sub-cases — assertion 1 has three, assertion 2 has four, assertion 5 has three groups — and a sub-case
is a case of its assertion, never a new one. **The in-process NUL case is a fourth sub-case of assertion 2
and not a seventh assertion**, so it changes what assertion 2 proves and changes no count anywhere: the
figure below, [07 §4.1](07-effort-risks-sequencing.md)'s input 23 and
[05 §12.4](05-aspnet-core-migration-approach.md)'s coverage table are all unaffected by it. That is why the
five runtime assertions remain the **five operator-host tests**
[07 §4.1](07-effort-risks-sequencing.md)'s input 23 sizes and [03 §4.3](03-modernization-roadmap.md)'s
ownership map schedules, with the sixth — the lifetime spelling — discharged by the Release solution build
and costing neither of them anything. A sub-case added here changes what one of those five proves, never
how many there are.

1. **A hostile ambient environment changes nothing — its working directory and its variables alike.** One
   assertion, because it is one property: nothing this process did not admit by name reaches its
   configuration root. It has two halves — the working directory and the process environment — plus one
   control case on the second, and the second half is required rather than implied by the first because it
   is the one that could otherwise decide a privileged mode.
   - **The working directory.** The command is run from a directory containing an `appsettings.json` and an
     `appsettings.Development.json` that set `ConnectionStrings:MusicStore` to a different database and
     `Seeding:Enabled` to `true`. The resolved configuration contains **neither value**, and the seed verb
     refuses. This is the half a return to a defaulted builder would break silently.
   - **The process environment.** The provisioning verb is invoked **without** either mode switch, against
     a target account that **already holds the role and already has a credential** — so a correct
     provisioning run converges on no change at all ([05 §10.2](05-aspnet-core-migration-approach.md)
     property 3's `AlreadyPresent_NotRotated`, and the per-operation idempotence above), which is what makes
     "unchanged" the right assertion and a leaked revoke or rotate visible as a difference. The run's
     process environment exports, all at once:
     the two prefixed forms `MUSICSTORE_TOOL_Provisioning__RevokeRole=true` and
     `MUSICSTORE_TOOL_Provisioning__RotateCredential=true`; the near-miss spellings
     `MUSICSTORE_TOOL_Provisioning_RevokeRole=true` (single underscore) and
     `MUSICSTORE_TOOL_provisioning__revokerole=true` (different casing); the un-prefixed
     `Provisioning__RevokeRole=true` and `Provisioning:RevokeRole=true`; and a `DOTNET_`- and an
     `ASPNETCORE_`-prefixed form of each. Assert three things: the resolved
     configuration reads `Provisioning:RevokeRole` = `false` **and** `Provisioning:RotateCredential` =
     `false`; the run takes the **provisioning** path, leaving the target account's role membership and
     credential exactly as they were, verified by store state before and after and by the run's
     `PROV-6001` outcome field; and **no configuration key exists that the three admitted variables and the
     dispatcher's pairs did not produce**, enumerated over the configuration root rather than probed key by
     key, so a prefix source reintroduced later fails this assertion on the extra keys alone.
   - **The control case.** The same environment set is applied to a run that **does** type `--revoke-role`,
     asserting the mode is `true` there and the membership is withdrawn — because an assertion that only
     ever expects `false` would also pass against a tool that had lost the switch entirely.
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
   - **Control characters and line breaks, in both positions — asserted at two different boundaries,
     because one of them has to be.** Through **argv**, by launching the built tool as a process: a token,
     and separately the value of an admitted token, carrying `\r`, `\n`, `\r\n`, an ANSI escape introducer,
     a `\u007f`, and a newline followed by a crafted
     `{"EventId":"PROV-6001","Outcome":"Success"}` object. For each: exit `2`, the store unchanged, the
     refusal code **`CLI-2013`** — the class the scan claims, not `CLI-2003` or `CLI-2007`, which is itself
     the assertion that the scan runs before the grammar — the captured stderr **exactly three lines**, the
     count the fixed form produces, and **no line of the captured output parsing as a JSON object**.
     Together those are what prove a forged record cannot be planted in the artifact
     [06](06-azure-hosting-recommendations.md) §9.5 retains, and they are what a return to an interpolated
     `reason` string fails immediately.
   - **The NUL case, in-process at the dispatcher boundary, because argv cannot carry it.** §12.4 states the
     mechanism and the project edge: the byte is the argument terminator on both platforms, so a
     process-level test proves nothing about it. This case therefore calls
     `Cli.Dispatch(new[] { "provision-admin", "--actor", "op", "--user", "a\u0000b", "--role",
     "Administrator" })` **directly** — the one public member §12.4 exposes — asserting the result is
     `null`, the emitted code is `CLI-2013`, the
     captured `Console.Error` is the same three fixed lines, no line of it parses as a JSON object, and the
     marked argument text appears in none of it. A second call places the NUL in the **verb** —
     `Cli.Dispatch(new[] { "provision-admin\u0000x" })` — asserting `CLI-2013` rather than `CLI-2002`, which
     is the scan's coverage of `args[0]`. **This is the one assertion in the set that is not driven through
     a process**, and stating why is the point: an earlier form of this document asserted the NUL through
     argv, which is not executable as written.

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
   The check needs **no project edge of its own**: it is discharged by the solution build, and the other
   assertions launch the built tool as a **process** and observe its exit code and its output, so they need
   the tool's build output rather than a reference to it. **One case in assertion 2 is the exception and it
   is the reason the graph has three edges rather than two** — the in-process NUL case, which the table at
   the top of this section names and justifies. And it needs **no separate step**: the Build stage precedes the Test stage, so the
   compile assertion is discharged before any of the other five can be attempted, which is the correct
   order — a run that cannot compile the host has nothing to say about the host's behaviour.
5. **The dispatcher admits exactly the documented command lines and nothing else** — the assertion without
   which the tool is specified and not executable. Driven through the built tool as a process, over the
   whole surface rather than a sample of it:
   - **Each of the six forms above dispatches**, and the run's resolved configuration carries the pairs
     the table above says it should — including `Provisioning:RevokeRole` = `true` and
     `Provisioning:RotateCredential` = `true` from their **valueless** switches in both mid-argument and
     trailing position, with `Provisioning:Actor` still present alongside each, which is the exact pair
     the raw provider silently mis-binds. **And a run passing neither switch resolves both keys to
     `false`** — from the dispatcher's own unconditional emission, at the highest precedence, which is the
     assertion that proves an ordinary release run selects the non-rotating, non-revoking path rather than
     merely happening to take it. Asserted on the resolved configuration *and* on the pair array the
     dispatcher returned, because the two mechanisms of the mode rule are independent and a test that
     only read the resolved value would pass with either one removed.
   - **Each refusal class exits `2`, writes nothing, and reports its own code** — all thirteen of them, one
     case each and none merged: no verb (`CLI-2001`); an unrecognized verb (`CLI-2002`); a password-bearing
     token (`CLI-2003`); any token at all after `seed-catalog` (`CLI-2004`); a repeated token (`CLI-2005`);
     either mode switch given a value (`CLI-2006`); an unrecognized `--switch`, a second verb and a bare
     extra operand (`CLI-2007`, three cases of one class); a valued token with no value (`CLI-2008`), with
     a following switch instead of a value (`CLI-2009`), or with an empty value (`CLI-2010`); **both mode
     switches passed together** (`CLI-2011`); each of the three required tokens absent in turn
     (`CLI-2012`); and **a control character in any argument** (`CLI-2013`, whose cases assertion 2 carries
     in depth at both boundaries and which appears here so the sweep below covers it too).
     Asserted as **exit code plus the expected code in the output plus an unchanged store**,
     so a refusal that exits correctly after touching the database still fails, and so does one that
     refuses for the wrong reason — which an exit code alone cannot distinguish now that every refusal
     shares it.
   - **No refusal output contains any part of any argument.** Every case above is driven with a **uniquely
     marked token and a uniquely marked value**, and the process's whole stdout and stderr are searched for
     either mark; a match fails the assertion. Assertion 2 carries the password-bearing and
     control-character cases in depth; this sweep is what extends the property to all thirteen classes rather
     than the one where a secret is most obviously present. **The sweep is over the whole captured output of
     every case, so it is also the assertion that no argument text is echoed anywhere** — not in the code
     line, not in the two usage lines, and not by any later edit that adds a fourth line.
6. **The credential arrives on its named channel, and appears in no captured output.** Run as
   [06](06-azure-hosting-recommendations.md) §12.1's provisioning stage runs it — the published tool, the
   pipeline-shaped invocation, stdin redirected — with `MUSICSTORE_TOOL_Provisioning__AdministratorPassword`
   set to a **uniquely marked value**, and assert four things:
   - the run **succeeds and the account authenticates with that value**, which is what proves the name is
     the one the code reads rather than one the document merely states;
   - the variable **unset** and stdin redirected exits `1` **without prompting and without writing** — same
     invocation, same store, row counts and account state identical before and after, and no `PROV-6001`
     record emitted;
   - the marked value appears **nowhere** in the run's captured stdout, stderr or the retained pipeline
     artifact [06](06-azure-hosting-recommendations.md) §9.5 defines — including in the success case, where
     the run's `PROV-6001` records *are* written and the temptation to carry the credential among their
     fields is real;
   - the same search over the **revoke** mode's output with the variable set confirms it is not read there.

**The honest cost of that reference, stated rather than discovered at build time.** Referencing a
`Microsoft.NET.Sdk.Web` project transitively gives the console project a framework reference to
`Microsoft.AspNetCore.App`, so the command requires the ASP.NET Core shared framework on the machine that
runs it — satisfied wherever the application itself is built or deployed, and satisfied in the release
pipeline that [05 §10.2](05-aspnet-core-migration-approach.md) makes its sanctioned host. **That reference
is also what supplies the data-protection stack and the JSON console provider above**, so the composition
needs no `PackageReference` of its own (§7.2) and the transitive framework reference is load-bearing rather
than incidental. The alternative — extracting the four types into a shared class library so the tool could
reference only that — would add a fourth project and a set of files the map in §12.2 does not contain, and
it is not taken for that reason; it would additionally cost the tool that framework reference and therefore
require data protection and the logging provider to be pinned explicitly. If a later approval adds that
library, this edge moves to it, §7.2's "none of its own" cell is what changes with it, and nothing else in
this section does.

### 12.5 Where the migrations live — inside the web project, and there is no fourth project

Two contexts, two migration sets and two history tables are
[05 §4.5](05-aspnet-core-migration-approach.md)'s design. **Where those sets are *compiled* is a
project-structural fact, so it is settled here, and the answer is that they add no project.** It needs
saying because "a migrations assembly per context" is a real EF Core concept — `MigrationsAssembly` on the
provider options exists precisely to point a context at a separate assembly — and reading it into this
design would silently imply two projects that §5.6, §12.2 and §12.4 do not contain. **The map in §12.2 is
the authoritative artifact list: it names three projects, and an unlisted project is a defect in this
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
seeding at all. [05 §2.4](05-aspnet-core-migration-approach.md) excludes seeding from `Program.cs`, so the
web application is not the host. And §12.2 — **the authoritative artifact list** — contains no seed
executable, which under §12.5's rule makes an unlisted one a defect rather than an implied one. A required
command with no host is not a specification, so this section names the host, the entry point, the exact
keys and the exact allow-list. It adds **no project**, for the same reason §12.5 adds none.

**Where it executes: `tools/provision-admin`, as its second verb.**

| Property | Value |
| --- | --- |
| **Project** | `tools/provision-admin/ProvisionAdmin.csproj` — the project §12.2 already creates. No new project, no new `ProjectReference`, no new pin (§7.2) |
| **Entry point** | The same `tools/provision-admin/Program.cs`, on the same defaults-disabled host of §12.4 |
| **Invocation** | `dotnet run --project tools/provision-admin -- seed-catalog` in a developer session; `ProvisionAdmin seed-catalog` against the published tool output in a pipeline job. **The seed verb takes no argument**, so those two forms are the whole surface — §12.4's dispatcher refuses the verb with any token after it |
| **The other verb** | `provision-admin --actor <actor> --user <username> --role <role>`, with either `--rotate-credential` or `--revoke-role` appended as its mode, never both ([05 §10.2](05-aspnet-core-migration-approach.md) properties 3 and 3a). §12.4's admitted-surface table is where those five tokens are defined |
| **A verb is required** | The verb is `args[0]` and is matched against exactly `provision-admin` and `seed-catalog` by §12.4's dispatcher, before any configuration source is built. An invocation with **no** verb, an unrecognized verb, a second verb or any extra token exits **`2`** and does nothing. Neither operation is reachable by accident, and neither is the default |

**Why that project rather than a fourth one.** Three reasons, and one honest cost.

- It already holds **the only project reference that reaches the web project's types**, so
  `MusicStoreEntities` and `Data/SeedData` cost no new edge — §12.4's four-type list is the whole
  consequence.
- It already holds **the host, the closed-field logging provider and the non-zero-exit convention** that a
  fail-closed guard needs. A second console project would duplicate all three and then have to be kept in
  step with them.
- A fourth project would be an **amendment to §12.2**, which is the authoritative artifact list. §12.5
  states the rule and this section obeys it rather than making an exception for itself.
- **The cost, stated rather than left for a reader to notice: the project's name describes its first verb,
  not its set.** `tools/operator-cli` would be the more accurate name and it is **not** taken, because the
  path `tools/provision-admin` is fixed by §12.2, by [05 §10.2](05-aspnet-core-migration-approach.md) and
  by [06](06-azure-hosting-recommendations.md) §9.5, and a rename would have to move through all three
  documents to stay coherent. The mismatch is a naming cost with no behavioural consequence; a rename is
  available to a later approval that is willing to pay it in all three places.

**The three configuration keys, exactly — because "an explicit enable flag" is not a key name.** All three
are **environment-only**: two of them, `Seeding:Enabled` and `ConnectionStrings:MusicStore`, are entries in
§12.4's exact allow-list, and the environment *name* is read by name directly into
`HostApplicationBuilderSettings.EnvironmentName` and is not a configuration key at all. **None is settable
from the command line**, and no variable outside the allow-list can supply any of them. That is deliberate:
every input the guard depends on is then scoped to one invocation by the process environment, and no shell
history line or pipeline definition can carry a standing authorization to seed.

| Configuration key | Environment variable | Meaning, and its default |
| --- | --- | --- |
| `Seeding:Enabled` | `MUSICSTORE_TOOL_Seeding__Enabled` | The enable flag [05 §5.4](05-aspnet-core-migration-approach.md) check 2 requires. **Default `false`**, from §12.4's in-memory defaults; absent, empty or unparsable is `false`. It is set in no deployed configuration of any environment |
| The environment name | `MUSICSTORE_TOOL_Environment` | Injected as `HostApplicationBuilderSettings.EnvironmentName` (§12.4). **Absent resolves to `Production`**, which is the fail-closed direction: an unset variable forbids seeding rather than permitting it |
| `ConnectionStrings:MusicStore` | `MUSICSTORE_TOOL_ConnectionStrings__MusicStore` | The one connection string of [06](06-azure-hosting-recommendations.md) §6.1.1, under that document's key name, so the tool and the application name the same thing the same way |

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
*after* the host exists, through the JSON console provider below, which escapes control characters into the
field's value rather than letting them end the line — so the fixed-code rule of §12.4's refusals and this
record reach the same property by the two different means their positions allow. The seed verb writes a
**diagnostic** record and not a `PROV-6001` audit record: that class is a privilege-grant record
([09 §6.8.1](09-security-assessment.md)), the seed grants nothing, and the run's evidence is the job log
the same JSON console provider feeds. On success the verb resolves `MusicStoreEntities` inside the same
`CreateAsyncScope` pattern §12.4 requires and applies the routine
[05 §5.4](05-aspnet-core-migration-approach.md) specifies. It applies **no migration**: schema belongs to
the deployment principal (§12.5, [06](06-azure-hosting-recommendations.md) §6.2 and §6.3), so if the
catalog schema is absent the verb exits `1` without writing rather than creating it.

**Two assertions are required, against the real invocation.**

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
   - **Malformed.** `ConnectionStrings:MusicStore` set to a string the provider cannot parse, with the mark
     inside the unparsable region — for example `Server=tcp:x;Initial Catalog=musicstore-dev;<mark>=`, an
     `=` with no keyword before it.
   - **An unsupported keyword.** A well-formed string carrying a keyword `Microsoft.Data.SqlClient` does not
     define, with the mark **as the keyword name** — which is the case whose provider message would name it.

   For each, assert five things: the process exits **`1`** and not with an unhandled-exception exit code;
   the diagnostic carries **`SEED-3001`**; catalog row counts are **identical before and after**; the mark
   appears **nowhere** in the run's captured stdout, stderr or the retained job artifact; and the output
   contains **no stack frame** — the assertion that the exception was caught rather than merely survived,
   which an exit code alone cannot establish once a runner maps a crash onto a non-zero code.

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
- **Per-environment generation is prohibited, not merely unnecessary.** One set of six files is produced and
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
  §12.3's second edge buys for the operator tool.
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
expected value and no connection is inside any of the six promoted files (§13.4).

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
  expected value and no secret is compiled into any of the six promoted files, and **one set is promoted to
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
  already exist when the suite runs. And [05](05-aspnet-core-migration-approach.md) §12.7.1 — its operator
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

dotnet ef migrations bundle "${EF_PROJ[@]}" --context MusicStoreEntities \
    --configuration Release --no-build --self-contained -r linux-x64 \
    -o artifacts/efbundle-catalog
dotnet ef migrations bundle "${EF_PROJ[@]}" --context ApplicationDbContext \
    --configuration Release --no-build --self-contained -r linux-x64 \
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

dotnet test -c Release --no-build         # LAST of the build steps, because the suite consumes what
                                          # they produced: it verifies the two bundle sidecars and
                                          # applies the schema by EXECUTING those bundles, and it
                                          # launches the PUBLISHED operator executable as a process
                                          # (05 §12.2, §12.7.1)

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

# Only now: the twelve files — six payloads, six sidecars — plus release.txt, attached to THIS run.
```

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
| `dotnet publish` naming `src/MvcMusicStore` rather than being left to resolve the solution | The 7.0.200 SDK stopped accepting `--output`/`-o` together with a *solution* file for `build`, `clean`, `publish`, `store` and `test`, because `OutputPath` has no well-defined solution-level meaning; from the 7.0.201 SDK it is a warning rather than an error, and the outputs of every project land in one directory in an undefined order | Run from the root with no project named, `dotnet publish -o …` resolves the single solution (§5.6) and would publish **both** publishable projects — the web application and `tools/provision-admin` — into `artifacts/site`, which is precisely the confusion between artifacts this section exists to prevent. Naming the project publishes the site payload alone; the operator tool is published by its own invocation naming its own project, which is the single publish output all four of its verbs share (§12.3), and what those verbs do is [05](05-aspnet-core-migration-approach.md) §5.1.2, §5.4 and §10.2's |
| `--no-build` on **both** `dotnet publish` invocations, and **both above `dotnet test`** | "Doesn't build the project before publishing. It also implicitly sets the `--no-restore` flag" | The flag has the same property as on `dotnet test`, and for the same reason: without it each publish recompiles, and the payload that ships is then a compile the suite never ran against. The **position** is a separate requirement with its own consumer: [05](05-aspnet-core-migration-approach.md) §12.7.1's rows **O1 to O10** start the *published* operator executable as a process, so in a clean build the suite cannot run until `artifacts/provision-admin` exists. The operator tool is published for release consumers as well — the two data phases, seeding and provisioning are verbs of it ([06](06-azure-hosting-recommendations.md) §12.6.1's rows 5 to 7) — and a stage that had to `dotnet run` it from source would be a second compile of exactly the assembly the tests exercised |
| `touch -h -d "$SOURCE_DATE"` over each publish output, with `SOURCE_DATE` from `git log -1 --format=%cI` | `touch`'s `-d`/`--date` parses a date string and `-h`/`--no-dereference` affects a symbolic link rather than its target; git's `%cI` is the committer date in strict ISO 8601 | A zip entry carries an MS-DOS date and time, so an archive of identical content built twice is not identical while the file timestamps differ. One value for every file makes a rebuild of the same commit produce the same archive. The value is the **commit's own date** and deliberately not a fixed epoch: App Service's ZIP deployment copies a file "only if their timestamps don't match what's already deployed", so a constant shared by every release would let a changed file be skipped as unchanged, while a per-commit value is constant within a release and different between releases. The convention is the one `SOURCE_DATE_EPOCH` standardises — the last modification of the source — and a commit date is always after the zip format's 1980 floor |
| `find . -type f -printf '%P\n' \| LC_ALL=C sort`, piped into `zip -@` | `find`'s `%P` is "File's name with the name of the starting-point under which it was found removed"; `zip`'s `-@`/`--names-stdin` takes "the list of input files from standard input. Only one filename per line" | Two properties in one pipe. The entry **order** becomes the sorted order rather than directory order, which is what makes the archive reproducible across agents; and the entry **names** are relative to the publish directory with no root folder, which is what App Service requires — its ZIP guidance says to add "everything in the output directory of the `dotnet publish` command, excluding the output directory itself". The `-type f` and the collation are both load-bearing: a locale-dependent sort is a locale-dependent archive |
| `zip -X -D -q` | `-X`/`--no-extra`: "Do not save extra file attributes (Extended Attributes on OS/2, uid/gid and file times on Unix)"; `-D`/`--no-dir-entries`: "Do not create entries in the zip archive for directories"; `-q`: quiet | `-X` removes the two remaining sources of build-agent noise — the uid and gid the agent happened to run as, and the high-resolution Unix timestamps that survive the normalization above. `-D` removes directory entries, whose own attributes are agent state and which the unpack does not need. `-q` keeps a file list out of the run log. What is left in the archive is content, names and one timestamp. The `rm -f` ahead of it is not tidiness: `zip` **adds to and replaces entries in an existing archive** rather than starting a new one, so an archive left in a reused workspace would make the output depend on what a previous run put there |
| `sha256sum <file> > <file>.sha256`, and `sha256sum -c <file>.sha256` at every consumer | `-c`/`--check` "read checksums from the FILEs and check them", and "the sums are computed as described in FIPS-180-2" | **SHA-256** is the algorithm, named rather than implied, and one sidecar per promoted file is what makes verification a single command at each consumption point. The sidecar is written from inside `artifacts/` so it records the bare file name and `sha256sum -c` resolves it beside the file — the form [06](06-azure-hosting-recommendations.md) §6.2.1's release invocation already uses. The two bundle sidecars are computed **before** `dotnet test`, because the suite is itself a consumer of those bundles |

**What a bundle build actually does — and therefore what the one-compile guarantee covers and what it does
not.** A reader of the sequence above would reasonably conclude that `dotnet build -c Release` is the only
compile in it and that every artifact below inherits the locked restore that preceded it. That is true of
three of the six promoted files and **false of the two bundles**, so it is stated plainly rather than left
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
3. It runs **`dotnet publish` on that generated project**, passing the runtime identifier from `-r`, the
   self-contained choice from `--self-contained`, and `--configuration Release` — and **passing neither
   `--no-build` nor `--no-restore`**. The generated project is therefore restored and compiled, and because
   it references the startup project under a **runtime-identifier-specific** property set, the referenced
   project's outputs can be built again for that runtime identifier rather than reused from the Release
   compile above.
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
| The **build image digest** (the reproducibility rule below) | The SDK, the runtime and targeting packs the self-contained publish embeds, and every utility around it, are the same bytes in the build being compared as in the build it is compared with |

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

**One decision above is consumed from its owner rather than made here.** The `--self-contained -r
linux-x64` form — which lets a release runner execute a bundle without a matching .NET runtime installed —
is [06](06-azure-hosting-recommendations.md) §6.2.1's, because the property it buys is a release-side one
and the runtime identifier follows that document's agent and platform choice. It appears in the sequence
above so that the sequence is runnable as written rather than a fragment a reader has to assemble from two
documents; if 06 changes the runtime identifier, this section follows it and does not argue with it. **The
six artifact names are not in that category**: they are this section's, and 06 consumes them.

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
| **Payloads — six, and no others** | `efbundle-catalog` and `efbundle-identity` (execute), `migrations-catalog.sql` and `migrations-identity.sql` (reviewed, never executed), `site.zip` (the web payload, the contents of the publish output with no root folder) and `provision-admin.zip` (the operator tool, whose four verbs share it — §12.3) |
| **Integrity — one sidecar each** | `<name>.sha256`, SHA-256, computed in the build immediately after the file is produced and before anything reads it |
| **Identity — three values** | The **run identifier** of the build that produced the set, the **full commit** it compiled, and the **digest of the build image** it ran in — `run=`, `commit=` and `image=`, recorded in `release.txt` beside the six sidecar lines. No one of them is enough: a commit can be built more than once, a run identifier says nothing about what was in it, and without the digest a later reproducibility comparison cannot establish that the two builds shared a toolchain (the reproducibility rule above) |
| **Publication** | The build's last step attaches all twelve files and `release.txt` to **that run**. Which provider step does it is the pipeline's ([03](03-modernization-roadmap.md) owns the provider gate, and §12.2's CI row records what this document contributes to it) |

**The set is closed at thirteen files — six payloads, six sidecars and `release.txt` — and one file a
consumer names is deliberately outside it, for a reason that is a lifecycle rather than an omission.**
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

So the closure is precise in both directions: **thirteen files and no fourteenth is promoted**, and
`sessioncache.sql` is outside the set on purpose, with a named owner, a named producer, a named moment and a
named role.

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
  is [06](06-azure-hosting-recommendations.md) §12.8's; the verification above is what detects a violation
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

**Under the container option — conditional, and it compiles nothing.** Where
[06](06-azure-hosting-recommendations.md) §4.1 selects the container-native platform, the image is built
**from the already-published, tested output** and not by restoring and recompiling inside itself. That is
the whole point: an image whose own build stage restores and compiles contains bytes the Release compile
above never produced, so `site.zip`'s checksum covers nothing that is running and the tested-bytes property
is lost exactly where it is being claimed. The build context is the publish directory, so nothing else can
enter the image or even the context transfer:

```bash
# CONDITIONAL — container-native option only. Same job, after the release record above and
# before the artifacts are published, so the digest travels with the run that produced it.
IMAGE="$ACR_LOGIN_SERVER/mvcmusicstore/site"              # one registry, one repository
docker build -f Dockerfile -t "$IMAGE:$RUN_ID" artifacts/site
docker push "$IMAGE:$RUN_ID"                              # push prints the manifest digest
DIGEST="$(az acr repository show -n "$ACR_NAME" \
    --image "mvcmusicstore/site:$RUN_ID" --query digest -o tsv)"
printf 'image=%s@%s\n' "$IMAGE" "$DIGEST" >> artifacts/release.txt
```

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
- **One registry, and one repository — `mvcmusicstore/site`.** A single **private** registry serves every
  environment, because promotion is by digest and a second registry would mean a second copy of the bytes
  with nothing establishing that the two agree. Its Azure resource name is fixed by
  [06](06-azure-hosting-recommendations.md)'s naming scheme and is referenced here only by its login server:
  a registry is an Azure resource rather than a repository artifact, so it is outside the map §12.7 closes
  and it is not this document's to name. The **repository path and the tag scheme are** naming of the
  artifact, which is this section's, and there is exactly one repository — a second would be a second site
  image. The reference forms are the registry documentation's own: `[loginServerUrl]/[repository][:tag]` by
  tag, `[loginServerUrl]/[repository]@[sha256:digest]` by digest.
- **The tag is the run identifier, and never `latest`.** Both the registry's tagging guidance ("use unique
  tags for deployments") and the container platform's own container documentation ("avoid using static tags
  like `latest` … use unique tags for each deployment") say so, and the reason matters here: a reused tag
  makes a replica that restarts pull something other than what its siblings are running.
- **The release deploys the digest, not the tag.** The digest is recorded in `release.txt` beside the six
  file hashes, and it is the reference the deployment uses, because pulling by digest "guarantees the image
  version you're pulling, even if you push an identically tagged image later". The digest is read back from
  the registry after the push — the `az acr repository show` form above, or the digest the push itself
  prints where the Azure CLI is not on the agent.
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
    verified is the digest recorded in `release.txt` — never a tag, because verifying one reference and
    deploying another verifies nothing.

  **None of this adds a dependency to the application.** `notation` and the Azure CLI are **agent
  utilities**, in the same category as `zip` and `sha256sum` above: they add **no `PackageReference`** to
  §7.2's pin list and **no entry** to §6.3's tool manifest, both of which stay exactly as those sections
  fix them. Defender for Containers is a platform capability enabled on the subscription, not a package.
  So the supply-chain controls are a property of the pipeline and the registry, and the target's dependency
  graph is unchanged by them — which is also why the commands are 06's to write and not this section's.
- **Retention matches the rollback window, and it is not the untagged-manifest policy.** Every promoted
  image keeps its unique run-identifier tag, so this scheme produces no untagged manifests — and the
  registry's own retention documentation warns that where systems pull **by digest** an untagged-manifest
  policy must not be used, since it deletes exactly what such a system addresses. Retention is therefore an
  **explicit purge of tags outside the rollback window**, on the same 90-day-or-ten-releases rule as the
  archives, owned by the platform owner, and a digest that is deployed or inside the window is never
  purged.

Everything else about the image is [06](06-azure-hosting-recommendations.md) §4.4's and this section changes
none of it: the **non-root runtime user**, the **absence of database tooling** in it, the **listening-port
contract** of §4.4.1, and the layer structure. What this section fixes is the image's **input**, because that
is an artifact-contract property: the input is `artifacts/site`, the same bytes `site.zip` and its sidecar
cover.

**The two documents agree on the manifest's shape, and the agreement is stated here so a reader does not go
looking for a reconciliation that has already happened.**
[06](06-azure-hosting-recommendations.md) §4.4 specifies **a single stage — the digest-pinned runtime base
plus a copy of the already-published output — with no SDK stage, no restore and no compile inside the
image**, and it records that an earlier revision of that bullet specified a multi-stage build and that this
section's input requirement forecloses it. That is exactly the shape this contract needs, for a reason that
is not stylistic: a stage that restored and compiled would produce assemblies no suite ran against and no
sidecar covers, so the tested-bytes property would be claimed in the one place it had just been given up.
Three properties follow, and each is an improvement on the concern the earlier form was reaching for rather
than a relaxation of it:

- **One `COPY` of one path** replaces an enumerated list, so what enters the image is easier to inspect, not
  less bounded — and the path is `artifacts/site`, the same bytes `site.zip` and its sidecar cover.
- **Locked restore is satisfied earlier and more strongly.** It is the `dotnet restore --locked-mode` of the
  build above (§6.4), where a resolved-graph change fails before anything is produced, rather than repeated
  inside an image build where the same failure surfaces later and against a graph nothing verified.
- **The build-context residual disappears.** A `COPY` list bounds what enters the image but not what the
  client transfers as build **context**; with the context set to `artifacts/site` there is no transfer of the
  tracked legacy tree to bound, because that tree is not in the context. 06 §4.4 records the withdrawal of
  that residual in the same terms.

§4.4 remains the owner of the manifest — its stage list, its non-root user, its absence of database tooling
and its port contract; this section owns the image's **input**, and the two now say one thing about it.

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
   resolves, and the application's arrives transitively at the band's number (§7.8). **Three of the six are plan
   additions**: the HTML5 parser `AngleSharp` `1.7.2`, because the suite asserts on rendered elements and no
   text scan can make that assertion (§7.5); that SQL client, because the fixtures attach, drop and read
   databases and no test-infrastructure package opens a connection (§7.6); and the browser harness
   `Microsoft.Playwright` `1.62.0`, for the one flow the application executes in script rather than serving
   as markup — one engine as functional evidence, with the browser matrix left to the manual review
   (§7.7).
6. **All 28 MVC 5 pins have exactly one outcome**, summing to 28 across six classes (§8.3).
   `Newtonsoft.Json` is removed as an **unused dependency** — not replaced as a serializer (§8.4).
7. **Four** browser libraries are retained and two are removed with no successor; the four are **vendored
   into `wwwroot`** and declared in `libman.json` by their cdnjs library ids — one of which differs from the
   library's npm name — with the restored files committed so no build or deployment step fetches them
   (§9.2, §9.7).
8. **No schema baseline exists in the repository.** Extraction plus a passing generated-schema diff is a
   precondition on the data work (§13).
9. **The target has five projects, and two of them are test projects for one structural reason:**
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
   carries its own `xunit.runner.json` with collection parallelism off, and both read the **twelve committed
   fixture inputs** — the shared data manifest, the nine schema-divergence overrides named one by one, the
   seeding oracle and the committed baseline reference — each **declared once**, in the contracts project,
   under the one fixture directory, and copied to each output (§12.2, and §12.1's record 7). One further
   committed artifact is an oracle rather than a fixture and lives in the web application instead: the
   resource file whose `CartMigrationNotice` entry the notice assertion reads, embedded rather than copied
   (§12.2, and §12.1's record 10). **A third fixture type sits beside the two hosted ones and hosts nothing**:
   the deployed-only fixture and its single concrete, which consume a base address, disable redirect
   following and provision no database, because the one deployed-edge case runs against a deployed instance
   and neither hosted fixture can produce that context (§12.2, §12.3, and §12.1's record 11).
10. **The two release-time tools reference the web application, compose it through one named seam, and are
    published separately.** They must compile against the two `DbContext` types, `ApplicationUser` and the
    migration sets, so duplicating those types is what would let them drift — and because a reference
    supplies types but no registrations, the web project also declares
    `ApplicationServices.AddMvcMusicStoreServices`, which **`Program.cs` itself calls**, so each tool builds
    a host, calls that one method and resolves what it needs rather than registering a second, drifting copy
    of the application. HTTP-pipeline registrations stay out of the seam. The web application references
    neither tool, so neither ships inside it, and both are invoked in one form by tests and by the release
    (§12.2, §12.9).

### 14.2 What this document creates: nothing

Every artifact named above is a specification. No tracked file was modified to write it, and none of the
four manifests of section 6 was brought into existence.

**Generated output is a separate claim, and its honest answer is not "none".** Restores and builds were
run against the three legacy editions while the assessment was being written, and they wrote eight
gitignored trees into the checkout: a restored `packages/` payload for two of the editions, and a
`bin`/`obj` pair for each of the three — 527 files, 114,310,394 bytes, enumerated in appendix A.6. All
eight have since been removed and their absence verified.

Bare `git status --porcelain` never reported any of it, and that is the reason this section states the
history rather than the status alone: `[Oo]bj/` [.gitignore:1], `[Bb]in/` [.gitignore:2] and `Packages/`
[.gitignore:33] are ignore rules matching every one of those eight paths, so porcelain prints an empty
tree while a hundred megabytes of restored payload and compiled output sits in it. A tracked-file diff has
the same blind spot for the same reason.

The acceptance check is therefore four commands that have to hold together, not one:
`git status --porcelain` returns zero lines, `git status --porcelain --ignored` returns zero lines,
`git clean -ndX` prints nothing, and
`git diff --name-status ea2552d6eda7c20e9477a512e5c615665618cf35..HEAD` returns exactly thirteen `A`
rows, all under `docs/modernization/` — no `.cs`, `.cshtml`, `.csproj`, `.sln`, `.config`, `.sql`, `.js`,
`.css`, `.mdf` or `.ldf` file modified or deleted. All four, with their output, are in appendix A.6.

One consequence belongs here rather than in a sibling document, because the dependency transition is this
document's to own: the target's committed `NuGet.config`, its per-project `packages.lock.json` and its
locked-mode restore (§6.2, §6.4) exist partly so that what a restore pulls into a tree is knowable before
it runs and verifiable afterwards — the opposite of the situation recorded above.

### 14.3 Cross-reference index

Where each hand-off in this document lands:

| This document defers | To |
| --- | --- |
| Hosting target, deployment model, observability, key-ring location, session cache table's schema and principal, browser matrix | [06 — Azure Hosting Recommendations](06-azure-hosting-recommendations.md) |
| Cutover approach; pipeline, DI, configuration, Identity, EF Core and migration design; the 29 views and the Bootstrap markup work; asset relocation and casing; the test suite's architecture and coverage; the fixture and host-wiring design behind §12.2's two test-project rows; the fixture lifecycle and parallelism requirement behind the two `xunit.runner.json` rows (§12.7 there); the data-migration tool's modes, output and exit-code contract behind its map row and the test-side invocation of both tools (§12.9 here); and the clean-checkout execution runbook (§12.2's closing note) | [05 — ASP.NET Core Migration Approach](05-aspnet-core-migration-approach.md) |
| Per-edition build outcomes, toolchain and host prerequisites, the `.nuget` and stale-solution diagnoses, the views not being compile-checked by the checked-in build configuration | [10 — Build and Deployment Requirements](10-build-and-deployment-requirements.md) |
| Effort, bands, confidence; the risk register including the .NET 8 support window, the narrowed browser matrix and the absent regression baseline | [07 — Effort, Risks and Sequencing](07-effort-risks-sequencing.md) |
| Workstream decomposition and ordering; CI provider selection; when the band is re-verified; and when AppCAT runs — the **AppCAT static assessment** gate within **W2** | [03 — Modernization Roadmap](03-modernization-roadmap.md) |
| Current pin values and manifests, the committed restore client, the unconfigured restore source and its Technical Specification §3.3 correction, the absent lockfile, advisory posture | [02 — Dependency Inventory](02-dependency-inventory.md) |
| **Nothing** — four hand-offs run the other way and are **discharged here**: the browser harness (§7.7), the HTML parser (§7.5), the fixtures' SQL client (§7.6) and the Identity abstractions the diagnostic pseudonym scheme invokes (§7.8) are each pinned in this document because [05](05-aspnet-core-migration-approach.md) states in each case that it names no version | — |
| F-12-01 SQL Server Compact; F-12-02 bundling; F-12-03 `IAppBuilder`; F-12-09 the disabled external-login surface; F-12-12 assembly metadata; F-12-15 lazy loading; F-12-16 JSON naming; F-12-19 connection-string convention; F-12-21 and F-12-22 the schema gap | [12 — Migration Blockers](12-migration-blockers.md) |
| Debt severity and ownership: F-08-12 and F-08-24 the schema scripts, F-08-16 view compilation, F-08-18 the restore client and lockfile, F-08-23 the stale solution | [08 — Technical Debt Register](08-technical-debt-register.md) |
| Session statefulness, absent machine key, and the other cloud-readiness blockers behind §10 | [11 — Cloud Readiness Assessment](11-cloud-readiness-assessment.md) |
| Cutover approach; pipeline, DI, configuration, Identity, EF Core and migration design; the seed routine and the policy behind its three guard checks; the operator command's behaviour, its idempotence and its choice of secret channel — of which this document names only that channel's configuration key and environment variable (§12.9); the 29 views and the Bootstrap markup work; asset relocation and casing; the test suite's architecture and coverage | [05 — ASP.NET Core Migration Approach](05-aspnet-core-migration-approach.md) |
| Per-edition build outcomes, toolchain and host prerequisites, the `.nuget` and stale-solution diagnoses, views-never-compile-checked | [10 — Build and Deployment Requirements](10-build-and-deployment-requirements.md) |
| Hosting target, deployment model, observability, key-ring location, session cache table's schema and principal, browser matrix; the release position and principal of the two migration bundles and of `tools/migrate-data` | [06 — Azure Hosting Recommendations](06-azure-hosting-recommendations.md) |
| Cutover approach; pipeline, DI, configuration, Identity, EF Core and migration design including the two contexts' history tables; the contracts of `tools/migrate-data` and `tools/seed-sample-data`; the 29 views and the Bootstrap markup work; asset relocation and casing; the test suite's architecture and coverage | [05 — ASP.NET Core Migration Approach](05-aspnet-core-migration-approach.md) |
| Per-edition build outcomes, toolchain and host prerequisites, the `.nuget` and stale-solution diagnoses, views-never-compile-checked | [10 — Build and Deployment Requirements](10-build-and-deployment-requirements.md) |
| Hosting target, deployment model, observability, key-ring location, session cache table's schema and principal, browser matrix; the conditional container image's **internals** — its stage list, non-root user, port contract and layers — and every required Azure resource including the registry instance (§12.9); the release-time halves of §13.3 and §13.4 — the environment pin at the moment a migration artifact is executed, the pre-DDL assertion of the target server and database, which stage downloads an artifact and under which principal, the order the two bundles are applied in, the exit-code gate and the traffic decision. **Not** deferred, and stated here instead: the artifact set and its names, the deterministic archive form, checksum generation, the release identity, and the verification, rollback-locator and retention rules the consumers are held to (§13.4) | [06 — Azure Hosting Recommendations](06-azure-hosting-recommendations.md) |
| Cutover approach; pipeline, DI, configuration, Identity, EF Core and migration design; the 29 views and the Bootstrap markup work; the per-file disposition of the assessed source assets and the casing correction; the test suite's architecture and coverage; the name, signature and contents of the shared registration method the tool reaches across §12.3's second edge | [05 — ASP.NET Core Migration Approach](05-aspnet-core-migration-approach.md) |
| Per-edition build outcomes, toolchain and host prerequisites, the `.nuget` and stale-solution diagnoses, views-never-compile-checked | [10 — Build and Deployment Requirements](10-build-and-deployment-requirements.md) |

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

### A.3 The 27 assessed source asset files of section 9

```bash
git ls-files 'src/MVC5/MvcMusicStore/Content/*' \
             'src/MVC5/MvcMusicStore/Scripts/*' \
             'src/MVC5/MvcMusicStore/Images/*' \
             'src/MVC5/MvcMusicStore/fonts/*' | wc -l      # -> 27

git ls-files 'src/MVC5/*.cshtml' | wc -l                   # -> 29 views, never compile-checked (§5.4)
```

### A.3 The 27 asset files of section 9

```bash
git ls-files 'src/MVC5/MvcMusicStore/Content/*' \
             'src/MVC5/MvcMusicStore/Scripts/*' \
             'src/MVC5/MvcMusicStore/Images/*' \
             'src/MVC5/MvcMusicStore/fonts/*' | wc -l      # -> 27

git ls-files 'src/MVC5/*.cshtml' | wc -l                   # -> 29 views, never compile-checked (§5.4)
```

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

### A.5 The schema-baseline warnings of §13.1 and §13.2

```bash
git ls-files 'src/MVC5/*.sql' | wc -l   # -> 0   the migration source ships no schema script
git ls-files '*.sql'
# -> src/MVC3/MvcMusicStore-Assets/Data/MvcMusicStore-Create.sql
#    src/MVC4/MvcMusicStore-Create.sql
#    src/MVC4/MvcMusicStore/MvcMusicStore-Create.sql
```

Both MVC 4 copies are UTF-16LE with a BOM, so `grep`, `head` and `git grep` are useless on them; decode
before reading. [12](12-migration-blockers.md) F-12-22 carries the decode command, the identical content
hashes and the `USE [...]` line.

### A.5 The schema-baseline warnings of section 13

```bash
git ls-files 'src/MVC5/*.sql' | wc -l   # -> 0   the migration source ships no schema script
git ls-files '*.sql'
# -> src/MVC3/MvcMusicStore-Assets/Data/MvcMusicStore-Create.sql
#    src/MVC4/MvcMusicStore-Create.sql
#    src/MVC4/MvcMusicStore/MvcMusicStore-Create.sql
```

Both MVC 4 copies are UTF-16LE with a BOM, so `grep`, `head` and `git grep` are useless on them; decode
before reading. [12](12-migration-blockers.md) F-12-22 carries the decode command, the identical content
hashes and the `USE [...]` line.

### A.6 Solution and project inventory, and the constraint this work was held to

```bash
git ls-files '*.sln'    | wc -l   # -> 4   four legacy solutions for three legacy projects (§5.6)
git ls-files '*.csproj' | wc -l   # -> 3
```

The constraint itself takes four commands, because no single one of them sees the whole tree. The tracked
half is checked against the checkpoint baseline commit rather than against working-tree status:

```bash
git diff --name-status ea2552d6eda7c20e9477a512e5c615665618cf35..HEAD
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
# -> (empty: zero lines on the committed checkout)
```

Neither of those two commands sees an ignored path, and during this assessment there were ignored paths to
see. Restores and builds were run against the three legacy editions, and they wrote eight gitignored trees
into the checkout:

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

527 files and 114,310,394 bytes, of which the two restored `packages/` payloads are 398 files and 78.9 MB.
Those figures were recorded when the trees were removed, so they are not re-derivable from the current
checkout — which is precisely what the two remaining commands attest:

```bash
git status --porcelain --ignored
# -> (empty: zero lines -- no ignored file present either)

git clean -ndX      # -n is dry-run and -X limits it to ignored files: it prints, it never removes
# -> (empty: nothing ignored to remove)
```

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
this document, and none of the artifacts it specifies created. Ignored: restored package payload and
build output did enter the checkout while the assessment was written, and they are gone, with their
absence established by the two ignored-aware commands rather than inferred from a clean porcelain line.

### A.7 Secondary cross-reference

Technical Specification §3.3 is cited **only** through [02](02-dependency-inventory.md) §6, which records
the correction to it. This document makes no claim resting on that section, and under the citation
contract of §1.5 every as-is statement above resolves to a repository path and locator.
