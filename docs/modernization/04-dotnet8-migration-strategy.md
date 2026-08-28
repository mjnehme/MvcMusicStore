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
the target-state `NuGet.config` and lockfile policy, the client-library acquisition mechanism, and the
tooling-manifest contents.

### 1.3 What this document does not own

| Not stated here | Owner | How this document treats it |
| --- | --- | --- |
| Hosting target, deployment model, observability approach, data-protection key-ring location | [06](06-azure-hosting-recommendations.md) | Cited. This document names the *package* that persists a key ring (§10.3); it does not choose where the ring lives |
| Cutover approach and its accepted losses; pipeline, DI, configuration, Identity, EF Core, view and static-asset transitions | [05](05-aspnet-core-migration-approach.md) | Cited. The conditional packages in §7.3 exist *because* an alternative cutover is retained; the choice is not made here |
| Per-edition build outcomes and toolchain prerequisites | [10](10-build-and-deployment-requirements.md) | Cited, never re-diagnosed |
| Effort, bands, confidence, and the risk register — including the .NET 8 support-window entry | [07](07-effort-risks-sequencing.md) | Pointed at. **This document contains no effort figure, no duration and no schedule** |
| Workstream decomposition, sequencing, and CI provider selection | [03](03-modernization-roadmap.md) | Cited |
| Current pin values, the restore-source finding, the committed restore client | [02](02-dependency-inventory.md) | Cited. §8 states each pin's *outcome*, not a re-transcription of the manifests |
| Every construct with no successor, and every successor whose default differs | [12](12-migration-blockers.md) | Cited by finding identifier |
| Debt framing, severity and ownership | [08](08-technical-debt-register.md) | Cited |

### 1.4 The no-modification constraint, and the boundary that makes this document possible

The user directed **"Do not make code changes initially"**; the project's environment setup instructions
independently restate it as "Do not modify code until assessment and modernization plan are approved."
**No repository file was modified to produce this document.** Every source artifact named below was read;
none was written. The acceptance check is that `git status --porcelain` shows only new files under
`docs/modernization/`.

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
   an as-is fact, and that fact is cited.
2. **Exact versions only.** Every version in this document is a single exact release. No range, no
   floating notation, no "or later", no placeholder. Where a version must be confirmed again before it is
   applied, that is stated as a re-verification step (§7.2), not softened into a range.
3. **Repository-wide claims carry their reproducing command**, in the appendix, so a reader can refute
   them.
4. **One fact, one owner** — the ownership tables in §1.2 and §1.3.

---

## 2. The target framework

### 2.1 The decision

**The target framework is `net8.0`.**

That is the whole decision, and it is stated once. It is what the user asked for, it is the framework
every version in section 7 belongs to, and it is the value deliverables 03, 05, 06 and 07 cite rather
than restate. The ported application, the test project and the operator tool all target it; there is no
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
   (§7.3), so if the incremental path in [05](05-aspnet-core-migration-approach.md) is ever selected, its
   package feasibility must be re-established against the new target rather than assumed to carry over.
4. **The test-tooling pins are re-verified**, since the test SDK and adapter are versioned independently
   of the runtime.
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
| `ProjectGuid` | `{25CE8290-EF24-4818-B009-68DC903163D3}` | [:10] |
| `ProjectTypeGuids` | `{349c5851-65df-11da-9384-00065b846f21}` (ASP.NET web application) then `{fae04ec0-301f-11d3-bf4b-00c04f79efbc}` (C#) | [:11] |
| `OutputType` | `Library` | [:12] |
| `RootNamespace` / `AssemblyName` | `MvcMusicStore` / `MvcMusicStore` | [:14-15] |
| `TargetFrameworkVersion` | `v4.8` | [:16] |
| `MvcBuildViews` | `false` | [:17] |
| Assembly references | **46** `<Reference>` elements, of which **26** carry a `<HintPath>` into `..\packages\...` and 20 resolve from the framework or the machine | [:47-158] |
| `<ProjectReference>` | **none** — a leaf project | [:1-302] |
| Compile inventory | **27** explicit `<Compile Include=...>` items, one per source file | [:161-189] |
| Content inventory | **61** `<Content Include=...>` items and **2** `<None Include=...>` items, naming every view, script, stylesheet, image, font and config individually | [:192-266] |
| Web host settings | `UseIISExpress` `true` [:18] with an empty `IISExpressSSLPort` [:19]; a `ProjectExtensions` block carrying `DevelopmentServerPort` `43524` and `IISUrl` `http://localhost:43524/` | [:18-19], [:277-294] |
| Restore opt-in | `RestorePackages` `true` [:24] with `SolutionDir` defaulted to `..\` [:23] | [:23-24] |

Four MSBuild imports and one target complete the picture, and they do not share one fate in section 5:

| Import or target | Condition | Locator |
| --- | --- | --- |
| `$(MSBuildBinPath)\Microsoft.CSharp.targets` | unconditional | [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:271] |
| `$(VSToolsPath)\WebApplications\Microsoft.WebApplication.targets` | `'$(VSToolsPath)' != ''`, where `VSToolsPath` derives from a `VisualStudioVersion` the project defaults to `10.0` [:268-269] | [:272] |
| `$(MSBuildExtensionsPath32)\...\v10.0\WebApplications\Microsoft.WebApplication.targets` | `false` — permanently inert | [:273] |
| `MvcBuildViews` target invoking `<AspNetCompiler>` | `'$(MvcBuildViews)'=='true'` | [:274-276] |
| `$(SolutionDir)\.nuget\NuGet.targets` | `Exists(...)` — conditional, and the condition is not met | [:295] |

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

- **The 27-item `Compile` inventory [:161-189] disappears entirely.** The Web SDK globs `**/*.cs` by
  default. A source file added after the conversion compiles because it exists, not because someone
  remembered to list it.
- **The 61 `Content` and 2 `None` items [:192-266] disappear.** Razor views are handled by the SDK, and
  static assets are served from `wwwroot` by static-file middleware, so they need no per-file item at all.
  This is also why the target has no equivalent of the `<Folder Include="App_Data\" />` item [:259] — the
  target has no `App_Data`.
- **`OutputType` [:12], `RootNamespace` and `AssemblyName` [:14-15] are defaulted rather than declared.**
  A web project is a library-shaped assembly with an entry point by default, and both names default from
  the project file name, which is unchanged.

`ProjectGuid` [:10] and `ProjectTypeGuids` [:11] are **dropped, not translated.** The SDK-style format
identifies a project by its path; the modern solution format does not require a project GUID declared in
the project file, and no project-type GUID selects behaviour any more — the `Sdk` attribute does.

### 5.2 `PackageReference` replaces `packages.config`

`packages.config` is deleted and the surviving dependencies become `<PackageReference Include="..."
Version="..." />` items in the project file, at the exact versions in §7.2. That change carries four
properties worth stating, because each one repairs a specific current behaviour:

1. **Hint paths cease to exist.** All 26 `<HintPath>` entries [:64-158] are removed. Package assets are
   resolved from the global packages folder by the restore graph, so the class of failure where a hint
   path points somewhere the payload is not — which is one of MVC 4's two committed defects,
   [10](10-build-and-deployment-requirements.md) §6.2 — becomes structurally impossible.
2. **Transitive dependencies are resolved, not enumerated.** Several of the current 46 references exist
   only because `packages.config` requires every transitive assembly to be referenced explicitly.
3. **The 20 references that carry no hint path are removed** — 17 of them contiguous at [:47-63], the
   other three at [:68], [:70] and [:103]. `System`, `System.Web`, `System.Xml`, `System.Configuration`,
   `System.Net.Http` and the rest are either part of the shared framework or gone; a `net8.0` project
   references no framework assembly by name.
4. **No `bin`-copy semantics to configure.** The `<Private>True</Private>` elements accompanying the
   package references [e.g. :65, :73, :77] have no analogue and are dropped.

### 5.3 `Properties/AssemblyInfo.cs` is absorbed into MSBuild properties

The file carries **12 assembly-level attributes** and is deleted; the metadata survives as build
properties, which the SDK uses to generate the attributes. Keeping both would produce duplicate-attribute
compile errors — the failure mode [12](12-migration-blockers.md) F-12-12 records. The mapping is
one-for-one and complete:

| Current attribute | Locator | Target MSBuild property |
| --- | --- | --- |
| `AssemblyTitle("MvcMusicStore")` | [src/MVC5/MvcMusicStore/Properties/AssemblyInfo.cs:8] | `<AssemblyTitle>` |
| `AssemblyDescription("")` | [:9] | `<Description>` |
| `AssemblyConfiguration("")` | [:10] | `<Configuration>` — generated per build configuration; not declared |
| `AssemblyCompany("")` | [:11] | `<Company>` |
| `AssemblyProduct("MvcMusicStore")` | [:12] | `<Product>` |
| `AssemblyCopyright("Copyright ©  2013")` | [:13] | `<Copyright>` — the value is refreshed at conversion; carrying a 2013 copyright forward is not a migration requirement |
| `AssemblyTrademark("")` | [:14] | `<Trademark>` |
| `AssemblyCulture("")` | [:15] | `<NeutralLanguage>` — omitted, since the current value is empty |
| `ComVisible(false)` | [:20] | Not carried forward. COM visibility is off by default and this project exposes nothing to COM |
| `Guid("64547e1b-3030-4458-ab71-a970f2916ed6")` | [:23] | Not carried forward. The type-library GUID is meaningful only under COM registration |
| `AssemblyVersion("1.0.0.0")` | [:34] | `<AssemblyVersion>` |
| `AssemblyFileVersion("1.0.0.0")` | [:35] | `<FileVersion>` |

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
| IIS Express settings and the `ProjectExtensions` web-project block | [:18-22], [:277-294] | The target runs on Kestrel. Local HTTP endpoints become launch profiles in `Properties/launchSettings.json`, which is developer configuration rather than a build input; the hosting model itself is [06](06-azure-hosting-recommendations.md)'s |

**The honest consequence of dropping `MvcBuildViews`: nothing is lost, because nothing was there.**
Because the property is `false` [:17] and its target is gated on `'true'` [:274], `AspNetCompiler` has
never run, so MVC 5's **29 Razor views have never been compile-checked** by this build. Deliverable
[10](10-build-and-deployment-requirements.md) §12.3 records that a zero-warning build says nothing about
the views, and §3.1 confirms that the verified clean build of MVC 5 did not exercise them.
[08](08-technical-debt-register.md) F-08-16 owns the debt severity.

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

### 5.6 Four solution files collapse to one

The repository tracks **four `.sln` files for three projects**, one per edition plus a second under
`src/MVC4/`. The target has a **single root `MvcMusicStore.sln`** referencing three projects — the ported
web application, the test project and the operator tool (§12).

The fourth solution is stale, with a project path that does not resolve;
[10](10-build-and-deployment-requirements.md) §6.4 owns the diagnosis and
[08](08-technical-debt-register.md) F-08-23 owns the debt. It is named here only because "collapse four
into one" would otherwise imply that all four are equally valid inputs to the consolidation, and one is
not.

The three legacy solutions and their projects are **not deleted** — §12.3.

---

## 6. The four net-new tooling manifests

None of these files exists in the repository, and **none is created by this assessment**. Each is
specified here in full so that the later phase creates it without re-deciding its contents.

### 6.1 `global.json` — the SDK band

Root of the repository. Contents exactly as given in §3.1: `"version": "8.0.400"`,
`"rollForward": "latestPatch"`. Nothing else belongs in it — in particular, no MSBuild SDK entries, since
the project uses only `Microsoft.NET.Sdk.Web`.

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

### 6.4 Per-project `packages.lock.json` — what resolves from those sources

Each project in the target — the web application, the test project and the operator tool — commits a
`packages.lock.json`, and **CI restores in locked mode**, so that a change to the resolved graph fails the
build instead of arriving silently.

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
  identical version. A mixed EF Core graph is not a supported configuration.
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
| nuget.org | `Microsoft.EntityFrameworkCore` | `8.0.30` | ORM |
| nuget.org | `Microsoft.EntityFrameworkCore.SqlServer` | `8.0.30` | SQL Server provider; carries `Microsoft.Data.SqlClient`, which is what supplies managed-identity authentication |
| nuget.org | `Microsoft.EntityFrameworkCore.Design` | `8.0.30` | Design-time services the migration executor loads |
| nuget.org | `Microsoft.AspNetCore.Identity.EntityFrameworkCore` | `8.0.30` | Identity store; replaces `Microsoft.AspNet.Identity` 1.0 |
| nuget.org | `Microsoft.Extensions.Caching.SqlServer` | `8.0.30` | Distributed cache backing session |
| nuget.org | `Microsoft.AspNetCore.DataProtection.EntityFrameworkCore` | `8.0.30` | Persists the data-protection key ring (§10.3) |
| nuget.org | `Microsoft.AspNetCore.Mvc.Testing` | `8.0.30` | Integration host for the test suite |
| nuget.org (tool) | `dotnet-ef` | `8.0.30` | The `dotnet ef` command — the Design package does not provide it (§6.3) |
| nuget.org (tool) | `dotnet-sql-cache` | `8.0.30` | Creates the session cache table — `Caching.SqlServer` is the runtime provider only (§6.3) |
| nuget.org | `xunit` | `2.9.2` | Test framework |
| nuget.org | `xunit.runner.visualstudio` | `2.8.2` | Test adapter |
| nuget.org | `Microsoft.NET.Test.Sdk` | `17.11.1` | Test host and `dotnet test` integration |

The three test-tooling pins are **not** on the `8.0.30` band and are not expected to be: they version
independently of the runtime, which is why they are listed with their own exact versions rather than
folded into the band statement above.

### 7.3 Conditional — the incremental path only

These packages are pinned here **only so that the alternative is costed rather than hand-waved**. They
belong to the incremental migration path, and the cutover decision is
[05](05-aspnet-core-migration-approach.md)'s, not this document's. **If the single-cutover path is
confirmed, none of these is referenced at all.**

| Registry | Package | Version | Role if the incremental path is selected |
| --- | --- | --- | --- |
| nuget.org | `Yarp.ReverseProxy` | `2.3.0` | Proxies unmatched routes from the new application to the legacy one |
| nuget.org | `Microsoft.AspNetCore.SystemWebAdapters` | `2.3.0` | Shared libraries |
| nuget.org | `Microsoft.AspNetCore.SystemWebAdapters.CoreServices` | `2.3.0` | The ASP.NET Core side |
| nuget.org | `Microsoft.AspNetCore.SystemWebAdapters.FrameworkServices` | `2.3.0` | The .NET Framework side; **targets .NET Framework 4.7.2 or higher** |
| nuget.org | `Microsoft.AspNetCore.SystemWebAdapters.Abstractions` | `2.3.0` | Shared session and remote-authentication abstractions |

One strategy-relevant note on the fourth row: the `4.7.2` floor is a **package-targeting fact**, and MVC 5
at `v4.8` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:16] clears it while MVC 4 at `v4.5`
[src/MVC4/MvcMusicStore/MvcMusicStore.csproj:16] and MVC 3 at `v4.0`
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:15] do not. That is a constraint on
the incremental path's feasibility per edition; it is a supporting fact for the edition triage, not its
basis.

---

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
| 5 | `jQuery.Validation` | F | Updated and vendored → **jquery-validation `1.21.0`** |
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

**On the four dormant social providers (rows 18-20 and 22).** They are removed rather than mapped,
because every external provider registration in the source is commented out — the surface ships but is
disabled, and [12](12-migration-blockers.md) F-12-09 owns the evidence and the resulting design choice.
The framework's `Microsoft.AspNetCore.Authentication.*` family is the successor family **if and only if
social sign-in is ever actually enabled**; pinning versions for a capability that has never been switched
on would be specifying work nobody has asked for. When it is enabled, those packages are pinned at that
time, at the then-current band.

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
changes the wire format, and it would change it identically if this package had never been pinned.

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
`Content` and `None` items: the stylesheets and scripts at
[src/MVC5/MvcMusicStore/MvcMusicStore.csproj:192-193], [:203-215] and [:233], and the Bootstrap 3
Glyphicons font files at [:195] and [:262-264]. NuGet stopped being the delivery
channel for browser libraries long ago, so removing `packages.config` (§5.2) removes the acquisition
mechanism as well as the pins. Something has to take its place, and leaving it unspecified would leave
the six rows in class F with a target version and no way to obtain it.

### 9.2 The six outcomes

| Current pin | Outcome | Target library and exact version |
| --- | --- | --- |
| `jQuery` `1.10.2` | Update, vendored | **jquery `3.7.1`** |
| `jQuery.Validation` `1.11.1` | Update, vendored | **jquery-validation `1.21.0`** |
| `Microsoft.jQuery.Unobtrusive.Validation` `3.0.0` | Replace, vendored | **jquery-validation-unobtrusive `4.0.0`** |
| `bootstrap` `3.0.0` | Update, vendored — **plus markup work** | **bootstrap `5.3.3`** |
| `Respond` `1.2.0` | **Remove** | none |
| `Modernizr` `2.6.2` | **Remove** | none |

The Bootstrap row is the only one where the package version is not the whole change: the views use the
Bootstrap 3 class vocabulary, so upgrading the library without touching markup would change the rendered
page. That markup work, and the decision about icons, belong to
[05](05-aspnet-core-migration-approach.md).

### 9.3 The mechanism, specified

- **Assets are vendored into `wwwroot`.** They are served by static-file middleware, with the framework's
  version-appending tag helper providing cache busting (§10.1). There is no bundler, because there is no
  successor to the removed bundling framework and 27 asset files
  (`git ls-files 'src/MVC5/MvcMusicStore/{Content,Scripts,Images,fonts}/*'` → 27, appendix A.3) do not
  justify introducing a JavaScript toolchain and its own dependency tree.
- **They are declared in a committed `libman.json`**, using the Microsoft Library Manager with
  `defaultProvider` set to `cdnjs`. Library Manager needs **no npm, no `node_modules` and no build-time
  toolchain** — which is the reason it is chosen over an npm-based flow for six libraries.
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

---

## 10. Capabilities that need no package

Stated explicitly so that nobody adds a package for something already present. Every capability below is
either in the shared framework or supplied by the platform.

### 10.1 In the shared framework

Dependency injection; configuration and options; structured logging through `ILogger`; health checks;
session; static-file serving with version-appended cache busting; HTTPS redirection and HSTS;
anti-forgery; and JSON serialization through `System.Text.Json`.

Three of these are worth naming as *net-new capability* rather than migrations, because the repository has
none of them today: there is no logging of any kind, no health endpoint and no security response header
anywhere in any edition. Those absences are findings owned by [08](08-technical-debt-register.md) F-08-13
and [11](11-cloud-readiness-assessment.md); this document's only claim about them is that acquiring them
adds **no package reference**.

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

---

## 11. Tooling posture

Two tools bear on a .NET 8 migration strategy, and both need a statement — one because it must not be
mistaken for an application dependency, the other because it must not be recommended without a
qualification.

### 11.1 AppCAT is an assessment tool, not an application dependency

Azure Migrate application and code assessment for .NET (AppCAT) is the mechanized-evidence tool for this
kind of migration: it performs static analysis of source, configuration and binaries and supports effort
estimation. It installs as a .NET tool and runs against the repository.

**It appears in no package table in this document, and that is deliberate.** It is not referenced by the
ported application, it is not restored as part of a build, and it does not belong in
`.config/dotnet-tools.json` alongside `dotnet-ef` and `dotnet-sql-cache` — those two are *required to
deploy the application*, whereas AppCAT is run by an engineer to produce a report. Adding it to the
application's tool manifest would make an assessment artifact a deployment prerequisite.

Where AppCAT's output is consumed — as corroborating evidence for the effort model — is
[07](07-effort-risks-sequencing.md)'s, and the roadmap gate at which it runs is
[03](03-modernization-roadmap.md)'s.

### 11.2 The .NET Upgrade Assistant is deprecated, and must not be presented otherwise

The .NET Upgrade Assistant has been **officially deprecated**, superseded by the GitHub Copilot
modernization tooling in current Visual Studio releases. It is recorded here because a strategy document
is exactly where a reader expects to find "run the Upgrade Assistant" as step one, and a plan that
recommended it without recording its deprecation would be pointing at unsupported tooling.

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

None of these files is created by this assessment.

### 12.2 The map

| Target file | Transformation | Source | What this document specifies about it |
| --- | --- | --- | --- |
| `MvcMusicStore.sln` | CREATE | `src/MVC5/MvcMusicStore.sln`, consolidating four solutions | Single root solution referencing three projects: the web application, the test project and the operator tool (§5.6) |
| `global.json` | CREATE | **net-new** | SDK band `8.0.400`, `rollForward: latestPatch` (§3, §6.1) |
| `NuGet.config` | CREATE | **net-new** | `<clear />` then nuget.org (§6.2) |
| `.config/dotnet-tools.json` | CREATE | **net-new** | `dotnet-ef` `8.0.30`, `dotnet-sql-cache` `8.0.30` (§6.3) |
| `src/MvcMusicStore/MvcMusicStore.csproj` | CREATE | `src/MVC5/MvcMusicStore/MvcMusicStore.csproj`, `src/MVC5/MvcMusicStore/packages.config`, `src/MVC5/MvcMusicStore/Properties/AssemblyInfo.cs` | SDK-style `Microsoft.NET.Sdk.Web`, `net8.0`, `PackageReference` at the §7.2 pins, absorbed assembly metadata (§5.3), implicit globbing, `MvcBuildViews` and the WebApplication targets import dropped (§5.4) |
| `src/MvcMusicStore/packages.lock.json` | CREATE | **net-new** (generated, then committed) | Locked transitive graph; CI restores in locked mode (§6.4) |
| `src/MvcMusicStore/libman.json` | CREATE | `src/MVC5/MvcMusicStore/packages.config` — the six content-delivering pins it replaces | `defaultProvider` `cdnjs`; the four retained libraries at the §9.2 versions |
| `src/MvcMusicStore/wwwroot/lib/**` | CREATE | the vendored output of `libman.json` | Committed, so no build or deployment step fetches them (§9.3). The *relocation* of the application's own 27 assets and the casing correction are [05](05-aspnet-core-migration-approach.md)'s |
| `src/MvcMusicStore/Properties/launchSettings.json` | CREATE | `src/MVC5/MvcMusicStore/MvcMusicStore.csproj:18-22`, `:277-294` — the IIS Express settings and web-project block it replaces | Local launch profiles. Developer configuration, not a build input; the deployed hosting model is [06](06-azure-hosting-recommendations.md)'s |
| `src/MvcMusicStore.Tests/MvcMusicStore.Tests.csproj` | CREATE | **net-new** — the repository contains no test project of any kind (appendix A.4) | SDK-style, `net8.0`, referencing `Microsoft.AspNetCore.Mvc.Testing` `8.0.30`, `xunit` `2.9.2`, `xunit.runner.visualstudio` `2.8.2` and `Microsoft.NET.Test.Sdk` `17.11.1`, plus its own committed lockfile. The suite's **architecture and coverage** are [05](05-aspnet-core-migration-approach.md)'s and its place in the sequence is [03](03-modernization-roadmap.md)'s |
| `tools/provision-admin/ProvisionAdmin.csproj` | CREATE | `src/MVC5/MvcMusicStore/App_Start/Startup.App.cs` — the startup provisioning it replaces | SDK-style `net8.0` console project with its own committed lockfile, **not deployed with the web application**. The command's behaviour, its secret channel and its idempotence are [05](05-aspnet-core-migration-approach.md)'s and [06](06-azure-hosting-recommendations.md)'s |
| Everything under `src/MvcMusicStore/` not listed above — `Program.cs`, `appsettings*.json`, controllers, models, binding and view models, services, `Data/`, views, view components | CREATE | mapped file-by-file by [05](05-aspnet-core-migration-approach.md) | Not specified here. This document's only claim is that they are compiled by implicit globbing rather than by an explicit `Compile` inventory (§5.1) |
| CI pipeline definition | CREATE | **net-new** | **Deliberately not named as a file.** Its path and format depend on a provider choice this assessment does not make; [03](03-modernization-roadmap.md) carries provider selection as an explicit roadmap gate. What this document contributes to it: the build image must satisfy the `8.0.400` band (§3.2), restore runs in locked mode (§6.4), and `dotnet tool restore` precedes any migration step (§6.3) |
| `Dockerfile` | CREATE | **net-new** | **Conditional.** It exists only under the container-native hosting option, which is [06](06-azure-hosting-recommendations.md)'s to select. If code deployment is chosen, this file does not exist at all |

### 12.3 Two project-structure decisions

**The ported application is a new sibling, not a replacement in place.** It is created at
`src/MvcMusicStore/` rather than overwriting `src/MVC5/MvcMusicStore/`. The reason is validation, not
sentiment: MVC 5 is the reference implementation of every behaviour the port must preserve, and the
behavioural baseline is captured by driving the *running* legacy application — which requires its source,
its project file and its committed databases to remain exactly as they are while the port is built and
compared against them. Replacing it in place would destroy the only reference the port can be checked
against, and the repository has no test suite to fall back on (appendix A.4).

**The three legacy projects are retained read-only.** `src/MVC3/`, `src/MVC4/` and `src/MVC5/` stay in the
repository as historical references and as the behavioural baseline. None is modified and **none is
deleted**. Only MVC 5 is ported; MVC 4 and MVC 3 are assessed in full and carried forward untouched, which
also means their four solutions and three legacy project files continue to exist alongside the single new
root solution of §5.6 — the consolidation is of the *target*, not a deletion of the past.

---

## 13. Two schema-baseline warnings this strategy must carry

Both exist because the dependency and project-format transition specified above says nothing about the
*database*, and a reader could reasonably assume a schema baseline is available somewhere in the
repository. It is not, and the two ways of assuming otherwise both fail.

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

---

## 14. Roll-up

### 14.1 The strategy in eight statements

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
   provide (§6.3, §7).
6. **All 28 MVC 5 pins have exactly one outcome**, summing to 28 across six classes (§8.3).
   `Newtonsoft.Json` is removed as an **unused dependency** — not replaced as a serializer (§8.4).
7. Browser libraries are **vendored into `wwwroot`** and declared in `libman.json`, with the restored files
   committed so no build or deployment step fetches them (§9).
8. **No schema baseline exists in the repository.** Extraction plus a passing generated-schema diff is a
   precondition on the data work (§13).

### 14.2 What this document creates: nothing

Every artifact named above is a specification. `git status --porcelain` after this work shows only new
files under `docs/modernization/` — no `.cs`, `.cshtml`, `.csproj`, `.sln`, `.config`, `.sql`, `.js`,
`.css`, `.mdf` or `.ldf` file modified or deleted, no `packages/` content added, no build output left
behind, and none of the four manifests of section 6 brought into existence. The verification is in
appendix A.6.

### 14.3 Cross-reference index

Where each hand-off in this document lands:

| This document defers | To |
| --- | --- |
| Hosting target, deployment model, observability, key-ring location, session cache table's schema and principal, browser matrix | [06 — Azure Hosting Recommendations](06-azure-hosting-recommendations.md) |
| Cutover approach; pipeline, DI, configuration, Identity, EF Core and migration design; the 29 views and the Bootstrap markup work; asset relocation and casing; the test suite's architecture and coverage | [05 — ASP.NET Core Migration Approach](05-aspnet-core-migration-approach.md) |
| Per-edition build outcomes, toolchain and host prerequisites, the `.nuget` and stale-solution diagnoses, views-never-compile-checked | [10 — Build and Deployment Requirements](10-build-and-deployment-requirements.md) |
| Effort, bands, confidence; the risk register including the .NET 8 support window, the narrowed browser matrix and the absent regression baseline | [07 — Effort, Risks and Sequencing](07-effort-risks-sequencing.md) |
| Workstream decomposition and ordering; CI provider selection; when the band is re-verified and when AppCAT runs | [03 — Modernization Roadmap](03-modernization-roadmap.md) |
| Current pin values and manifests, the committed restore client, the unconfigured restore source and its Technical Specification §3.3 correction, the absent lockfile, advisory posture | [02 — Dependency Inventory](02-dependency-inventory.md) |
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

### A.3 The 27 asset files of section 9

```bash
git ls-files 'src/MVC5/MvcMusicStore/Content/*' \
             'src/MVC5/MvcMusicStore/Scripts/*' \
             'src/MVC5/MvcMusicStore/Images/*' \
             'src/MVC5/MvcMusicStore/fonts/*' | wc -l      # -> 27

git ls-files 'src/MVC5/*.cshtml' | wc -l                   # -> 29 views, never compile-checked (§5.4)
```

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
git ls-files '*.sln'    | wc -l   # -> 4   four solutions for three projects (§5.6)
git ls-files '*.csproj' | wc -l   # -> 3

git status --porcelain            # -> only new files under docs/modernization/
```

No repository file was modified to produce this document, and none of the artifacts it specifies was
created.

### A.7 Secondary cross-reference

Technical Specification §3.3 is cited **only** through [02](02-dependency-inventory.md) §6, which records
the correction to it. This document makes no claim resting on that section, and under the citation
contract of §1.5 every as-is statement above resolves to a repository path and locator.
