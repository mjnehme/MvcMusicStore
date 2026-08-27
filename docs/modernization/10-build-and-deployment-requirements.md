# 10 — Build and Deployment Requirements (as-is)

**Deliverable 10 of 13** · MvcMusicStore modernization assessment · assessment record

This document records what each of the three shipped editions requires in order to **build**, what it requires in order to **run**, and what was actually observed when the prescribed build procedure was attempted. It is the assessment's single source of truth for per-edition build outcomes.

---

## 1. Purpose, ownership and authoring contract

### 1.1 What this document is

Four things, and nothing beyond them:

1. **The verified build evidence** — what was run, on what host, against what restore state, and what happened. Section 3 carries the table; sections 5 to 7 carry the per-edition detail.
2. **The toolchain and hosting prerequisites each edition demands**, derived from the repository's own project files, solution files and configuration rather than from any external statement. Section 4.
3. **The database components each edition needs in order to run**, stated per edition with the catalog store and the credential store separated, because in two editions they are not the same engine. Section 10.
4. **The permissions and the deployment-automation posture** the repository implies. Sections 11 and 12.

### 1.2 What this document owns, and what it must not restate

This deliverable is the **single owner of per-edition build outcomes**. Deliverables 07 and 12 cite this document for them rather than re-deriving them, so a statement here is what the whole assessment believes. One consequence is stated up front because it is the most important sentence in the document: **the build of the sole migration source, MVC 5, was carried as *blocked pending a Windows verification run* until that run was performed; it is now recorded as verified — restore and build both succeed, in Debug and in Release, with zero warnings and zero errors.** The finding that produced the blocked status is unchanged and still stands on its own: MVC 5 **cannot** be built from a clean checkout, because it commits no package payload. Section 5 carries both halves — the precondition failure in §5.1 to §5.3, and the completed run with its recorded evidence in §5.4.

Five decisions are owned elsewhere, and this document cross-references rather than restating them. A restatement in different words would read downstream as a second decision.

| Fact | Owner | This document's treatment |
| --- | --- | --- |
| Target framework, SDK band, project-format conversion | [04 — .NET 8 Migration Strategy](04-dotnet8-migration-strategy.md) | Not stated here. Section 4.1 reports the *current* per-edition target framework only |
| Hosting target and deployment model | [06 — Azure Hosting Recommendations](06-azure-hosting-recommendations.md) | Not stated here. Section 4.3 reports the *current* web-host requirements only |
| Cutover approach | [05 — ASP.NET Core Migration Approach](05-aspnet-core-migration-approach.md) | Not stated here |
| Effort model, bands and risk register | [07 — Effort, Risks, Sequencing](07-effort-risks-sequencing.md) | Not stated here. Section 5.4 names the open verification item; 07 carries it as a risk |
| The unconfigured restore source, and the correction to Technical Specification §3.3 | [02 — Dependency Inventory](02-dependency-inventory.md) §6 | Cross-referenced in section 8. The correction is not repeated |
| Debt framing and severity for the defects recorded here | [08 — Technical Debt Register](08-technical-debt-register.md) | Cross-referenced. This document states the *build or deployment requirement* a defect creates, not its severity or owner |
| Classification of a construct as a migration blocker | [12 — Migration Blockers](12-migration-blockers.md) | Cross-referenced. Deliverable 12 cites this document for build outcomes |

Two foundation deliverables supply the counts and inventories this document builds on and does not re-derive: [01 — Architecture Overview](01-architecture-overview.md) for file counts, startup composition and the per-edition persistence topology, and [02 — Dependency Inventory](02-dependency-inventory.md) for the pins, framework references and restore tooling. Deliverable 02 §4.1, §4.2, §4.3 and §5.1 each defer explicitly to this document for build consequences, and sections 4, 6 and 8 below discharge those deferrals.

### 1.3 Authoring contract, and the absence of user rules

`review_rules` returns exactly **"No user rules provided."** No project rule constrains this document. That absence is not licence to lower the bar and no rule is invented in its place; four contracts bind instead.

1. **Every as-is claim carries an inline `[<path>:<locator>]` citation** at the point the claim is made, repository-relative and resolving in the checkout. There is no trailing reference list.
2. **Repository evidence is primary.** Where an external statement and the repository disagree, the repository governs and the disagreement is recorded rather than smoothed over.
3. **Repository-wide claims — counts and absences — carry the command that reproduces them**, adjacent to the claim. Appendix A collects them.
4. **Every claim names the edition or editions it holds in.** This document is entirely per-edition; a claim about "the application" would be unverifiable here.

One further convention: **committed credential values are cited by locator and never reproduced.** The repository ships a plaintext administrator password in `appSettings` in two editions [src/MVC5/MvcMusicStore/Web.config:17], [src/MVC4/MvcMusicStore/Web.config:26], documented again in prose at [src/MVC5/README.md:79]. Its value appears nowhere in this document. The security finding belongs to [09 — Security Assessment](09-security-assessment.md); what belongs here is the deployment consequence, recorded in section 10.4.

### 1.4 The no-modification constraint

The user directed that no code changes be made before the assessment is approved, and the project's environment setup instructions restate the same gate independently. **No repository file was modified in producing this document.** The evidence in section 3 was gathered by build attempts, after each of which the checkout was returned to a clean state; every other check was read-only. The verification run recorded in §3.1 and §5.4 writes only paths the repository already ignores — `packages/`, `bin/` and `obj/` — so `git status --porcelain` is unchanged by it, and the MVC 4 overrides it needs were passed on the command line rather than written into any tracked file. Every defect recorded below is documented and none is repaired.

That constraint also explains a phrasing choice that recurs. Where this document says a defect must be fixed "before" something, it is stating a **precondition for a later, separately approved implementation phase** — never an instruction to change anything now.

---

## 2. What the prescribed build procedure could and could not deliver here

### 2.1 The five prescribed build steps, item by item

The project's attached environment prescribes a build procedure — restore NuGet packages, build the solution, resolve missing package dependencies, identify the required SQL Server database components, run unit tests if present — together with a build environment of Windows, Visual Studio 2022 Build Tools, a .NET Framework Developer Pack and NuGet.

**The execution host on which the assessment's build evidence was gathered was Linux.** Windows, Visual Studio 2022 Build Tools, the .NET Framework Developer Pack and SQL Server LocalDB are unobtainable on it. Four of the prescribed items are therefore **impossible on that environment rather than skipped**, and the distinction matters: a skipped step is a choice, an impossible one is a constraint that must be reported.

| Prescribed item | Status on the Linux evidence host | What that means |
| --- | --- | --- |
| Install the required Windows toolchain | **Impossible on this environment** | Windows, VS 2022 Build Tools and the .NET Framework Developer Pack cannot be installed on Linux |
| 1 — Restore NuGet packages | **Impossible on this environment** | Mono 6.8.0.105 was installed and provides `xbuild` (XBuild Engine 14.0). It ships **no `msbuild` and no NuGet client**, so no restore could be performed |
| 2 — Build the solution file | **Impossible on this environment** as prescribed | MSBuild was unavailable. `xbuild` was used instead, and section 2.2 states exactly what that substitution does and does not establish |
| 4 — Stand up the required SQL Server database components | **Impossible on this environment** | Neither LocalDB, nor SQL Server, nor SQL Server Compact could be installed. *Identifying* the components is a separate matter and was done — section 10 |
| Determine the target framework per edition | **Satisfied** | Read directly from the project files: section 4.1 |
| 5 — Run unit tests if present | **Satisfied vacuously** | No test of any kind exists anywhere in the repository. Section 9 gives the verifying command |
| 3 — Resolve missing package dependencies | **Partially satisfied — by analysis, not by restore** | The missing dependencies were identified per edition (sections 5, 6, 8), but nothing could be resolved by running a restore |

**The tally, stated plainly: four items impossible on this environment, two satisfied — one of them vacuously — and one partially satisfied by analysis rather than by execution.** The fourth item is the one most easily mis-stated, so it is split deliberately: *standing up* the database components was impossible, while *identifying* them is an analysis task that the repository supports and that section 10 discharges in full.

### 2.1.1 The prescribed toolchain later became available, and the prescribed steps were then executed

The table above is the accurate record **for the Linux evidence host**, and nothing below retracts it. It is not the last word: a **Windows host carrying the prescribed toolchain** subsequently became available, and prescribed items 1, 2 and 3 were executed on it against this same checkout.

| Prescribed item | Status on the Windows verification host | Evidence |
| --- | --- | --- |
| Install the required Windows toolchain | **Satisfied** | Windows Server 2022; MSBuild `17.14.51.32402` located through `vswhere`; NuGet CLI `6.11.1.2`; the .NET Framework 4.8 reference assemblies present on disk |
| 1 — Restore NuGet packages | **Satisfied** | All three solutions restore, exit `0`. §3.1 |
| 2 — Build the solution file | **Satisfied** | All three editions build in Debug and in Release, exit `0`, zero warnings. §3.1 |
| 3 — Resolve missing package dependencies | **Satisfied by restore, for two editions of three** | The restore creates MVC 5's absent payload; MVC 4's 24 hint paths remain unresolvable as committed and need the host-side overrides of §6.3 |
| 4 — Stand up the required SQL Server database components | **Not claimed** | LocalDB is installed on the verification host, but no application was run for this document. Section 10 *identifies* the components; standing them up and running the applications is outside this record |
| 5 — Run unit tests if present | **Satisfied vacuously, and confirmed** | No test of any kind exists. Section 9 |

Two boundaries on that run, both deliberate. It establishes what it observed and no more — a build result on one Windows host with one recorded tool inventory, which is why §5.4 records the versions and the restore source and not merely the outcome. And it does not license reading the Mono results differently: §2.2's limits still bind every conclusion drawn from them.

### 2.2 What a Mono `xbuild` invocation establishes — and what it cannot

**A Mono `xbuild` invocation is not equivalent to the prescribed toolchain, and nothing in this document treats it as one.** Stated precisely:

**What it can establish.** Whether a project's committed MSBuild configuration is *internally coherent* — whether its imports resolve, whether its property and item definitions evaluate, whether the paths it declares exist relative to the directory it is evaluated from. And, if evaluation succeeds, whether its compilable source is *syntactically sound*.

**What it cannot establish.** That a project builds on the toolchain the environment prescribes. It cannot substitute for MSBuild 17.x semantics, for the .NET Framework 4.8 reference assemblies, for the Visual Studio web-application targets, or for a NuGet client that current NuGet supports. A Mono success is not a Windows success, and a Mono failure is not automatically a Windows failure either — each observed failure below is separately classified as platform-independent or environment-specific, and that classification is the analytical work, not the invocation.

**The Mono results in section 3 are therefore retained as supplemental portability evidence only.** They are useful for exactly one thing: the *configuration* defects they expose are platform-independent, because a missing directory and an unconditional import to a non-existent file fail identically under any MSBuild implementation. Section 6 relies on that property and says so; nothing else in this document rests on the Mono results.

### 2.3 The toolchain gap is itself an assessment finding

The gap is recorded as a finding rather than worked around, because **a build that requires Windows, a specific Visual Studio generation and a machine-wide product install is itself a portability and cloud-readiness result.** Every one of the following is a repository-declared, not externally asserted, requirement, and each is evidenced in section 4:

- Two editions import Visual Studio web-application targets, one of them unconditionally and at a Visual Studio 2010-era path [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:209].
- One edition's MVC framework assembly resolves only from a machine-wide install of an out-of-support product [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:42].
- All three editions' data access is a Windows-only local database engine reached with Windows integrated authentication [src/MVC5/MvcMusicStore/Web.config:12-13], [src/MVC4/MvcMusicStore/Web.config:13], [:19], or a retired Windows-only embedded engine [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:55-59].
- Two editions declare IIS Express as their web host [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:18], [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:18]; the third declares a Visual Studio development server that no longer ships [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:17], [:223].

The cloud-readiness consequences of those facts are owned by [11 — Cloud Readiness Assessment](11-cloud-readiness-assessment.md); the *build and hosting requirement* each one creates is owned here.

---

## 3. The verified build evidence table

This is the document's centrepiece. Every outcome is what was observed, described in the terms that observation supports and no stronger. Two runs exist, and they are kept apart on purpose: the prescribed-toolchain run on Windows, which is the **authoritative** build evidence (§3.1), and the earlier Mono run on Linux, retained as **supplemental portability evidence** under the limits of §2.2 (§3.2).

### 3.1 The Windows verification run — prescribed toolchain, authoritative

Host: Windows Server 2022. MSBuild `17.14.51.32402`, located through `vswhere` rather than taken from `PATH`, for the reason given in §5.4. NuGet CLI `6.11.1.2`. Restore source `nuget.org` (`https://api.nuget.org/v3/index.json`), the single source registered on the host — the repository declares none, so that is a property of the host and deliverable 02 §6's finding is unaffected. Both configurations were built, with `/t:Rebuild` so the compiler actually ran rather than being skipped as up to date.

| Edition | Command | Restore state | Outcome |
| --- | --- | --- | --- |
| **MVC 5** | `nuget restore src/MVC5/MvcMusicStore.sln`, then MSBuild `/t:Rebuild` on the same solution, Debug and Release | Restored by that command, which creates `src/MVC5/packages` and satisfies all 26 hint paths | **Restore exit `0`; build exit `0` in both configurations, 0 warnings, 0 errors**, producing `src/MVC5/MvcMusicStore/bin/MvcMusicStore.dll`. See §5.4 |
| **MVC 4** | the same, on `src/MVC4/MvcMusicStore.sln` | Restored there; the committed payload at `src/MVC4/MvcMusicStore/packages` is referenced by nothing (§6.2) | **As committed: exit `1`, `MSB4019`, before compilation.** With the two host-side overrides of §6.3 supplied on the command line: exit `0`, 0 warnings. The stale solution fails separately, `MSB3202` (§6.4) |
| **MVC 3** | the same, on `src/MVC3/MvcMusicStore-Completed/MvcMusicStore.sln`, Debug and Release | Committed payload, 46 tracked files | **Exit `0`, 0 warnings, 0 errors.** The three unresolved-reference warnings of §7.1 do **not** occur here, because this host carries the machine-wide assemblies they name (§7.2) |

Three properties of this run matter as much as its outcomes.

- **A zero-warning build is not a statement about the views.** `MvcBuildViews` is `false`, so Razor is never compile-checked (section 12). The 29 MVC 5 views were not exercised by this run and still carry no build-time guarantee.
- **Restore succeeds *with* advisory warnings.** It emits **15** `NU1902`/`NU1903` advisories for MVC 5 and **14** each for MVC 4 and MVC 3, and still exits `0`. Those advisories, their severities and the pins they name are owned by deliverable 02 §8.2, whose counts this run reproduces exactly. They are warnings, not errors, and they change no outcome in the table above.
- **MVC 4's success is not the repository's.** It required two properties supplied from outside. The defects of section 6 are untouched by it, and a host that does not know to supply them gets `MSB4019`.

### 3.2 The earlier Mono run — supplemental portability evidence

| Edition | Command and host | Restore state | Outcome |
| --- | --- | --- | --- |
| **MVC 5** | `xbuild src/MVC5/MvcMusicStore.sln` (Mono 6.8.0.105, Linux), Debug | Un-restored; `src/MVC5/packages` absent | **Compile failure — a precondition failure, not a build defect.** See section 5 |
| **MVC 4** | `xbuild src/MVC4/MvcMusicStore.sln` and `xbuild src/MVC4/MvcMusicStore/MvcMusicStore.sln` (same host), Debug | `packages/` committed at `src/MVC4/MvcMusicStore/packages` | **Both fail before compilation** — two configuration defects plus a stale solution, all platform-independent. See section 6 |
| **MVC 3** | `xbuild src/MVC3/MvcMusicStore-Completed/MvcMusicStore.sln` (same host), Debug | `packages/` committed | **Compiles to `bin/MvcMusicStore.dll`** with three unresolved-reference warnings. See section 7 |

Three properties of this table are as important as its contents.

- **The three restore states differ, and that difference is a repository fact rather than a test artefact.** MVC 3 and MVC 4 commit their restored packages; MVC 5 commits none. Section 8 records the posture per edition.
- **"Fails before compilation" and "compile failure" are different outcomes.** MVC 4's builds never reach the compiler: MSBuild stops during project evaluation or reference resolution. MVC 5's build reaches the compiler and the compiler fails for want of assemblies. The distinction is what makes MVC 4's the *configuration* defect and MVC 5's the *precondition* failure.
- **No output artefact size is recorded for MVC 3, deliberately.** The figure is not stable across toolchains — a Mono-produced assembly and an MSBuild-produced assembly are not comparable — and quoting one would invite a false comparison. That an assembly was produced is the finding; how large it was is not evidence of anything.

---

## 4. Toolchain and hosting prerequisites, derived from the repository

Every requirement in this section is read out of a tracked repository file. The project's environment setup instructions assert Windows, Visual Studio 2022 Build Tools, a .NET Framework Developer Pack and NuGet, and that assertion is **corroborated** by what follows — but the requirements below stand on the repository and would stand if the environment text did not exist. This ordering is deliberate: a build requirement that rests only on a prose statement cannot be verified by a reader, and cannot be defended when someone proposes to change it.

### 4.1 Target framework per edition

Read directly from `TargetFrameworkVersion`:

| Edition | Target framework | Locator | Toolchain generation the solution file declares |
| --- | --- | --- | --- |
| MVC 5 | **`v4.8`** | [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:16] | Format 12.00, `# Visual Studio 2013`, `VisualStudioVersion = 12.0.21005.1`, `MinimumVisualStudioVersion = 10.0.40219.1` [src/MVC5/MvcMusicStore.sln:2-5] |
| MVC 4 | **`v4.5`** | [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:16] | Format 12.00, `# Visual Studio 2012` [src/MVC4/MvcMusicStore.sln:2-3] |
| MVC 3 | **`v4.0`** | [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:15] | Format 11.00, `# Visual Web Developer Express 2010` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore.sln:2-3] |

**`v4.8` is therefore the repository maximum**, and it belongs to MVC 5 — the edition the migration takes as its source. A .NET Framework 4.8 targeting pack or Developer Pack is a hard build requirement for MVC 5, 4.5 for MVC 4 and 4.0 for MVC 3; targeting packs are side-by-side, so a build host serving all three needs all three.

The `ToolsVersion` attribute tells the same story from the other end: `12.0` for MVC 5 [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:2], `4.0` for MVC 4 [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:2] and `4.0` for MVC 3 [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:2]. All three are the legacy MSBuild 2003 project format, not SDK-style. The conversion of that format is owned by deliverable 04.

One further per-edition difference worth recording as a build fact: MVC 5 and MVC 4 import `Microsoft.Common.props` at the top of the project, guarded by `Exists(...)` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:3], [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:3]. **MVC 3 has no such import at all** — its project begins straight at `<PropertyGroup>` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:3]. That is the pre-2012 project shape, and it is one reason MVC 3's build behaviour is the most toolchain-sensitive of the three.

### 4.2 What each project file demands of the build host

These are the imports and references without which MSBuild does not produce output. Deliverable 02 §4.3 inventories the imports as dependencies; what follows is their **build consequence**, which is this document's responsibility.

| Requirement | Editions | Evidence | Consequence if unmet |
| --- | --- | --- | --- |
| Visual Studio web-application targets, located via `$(VSToolsPath)` | MVC 5, MVC 4 | [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:272], [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:337], both `Condition="'$(VSToolsPath)' != ''"`; `VSToolsPath` is itself defaulted from `VisualStudioVersion`, which the project defaults to `10.0` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:268-269], [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:333-334] | **Fatal, and the guard does not prevent it** — see the note below the table |
| Visual Studio web-application targets at a **hard-coded v10.0 path**, imported **unconditionally** | **MVC 3 only** | [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:209] | **Fatal.** MSBuild raises a missing-import error on any host without `…\Microsoft\VisualStudio\v10.0\WebApplications\Microsoft.WebApplication.targets` — a Visual Studio 2010-era path absent from a Visual Studio 2022-only machine |
| A machine-wide install of the **ASP.NET MVC 3 Tools Update** | **MVC 3 only** | `System.Web.Mvc, Version=3.0.0.0` referenced with **no `HintPath`** [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:42], plus `System.Web.WebPages` `1.0.0.0` [:43] and `System.Web.Helpers` `1.0.0.0` [:44] on the same footing. The repository's own tutorial note confirms the product generation: the assets are "updated for the ASP.NET MVC 3 Tools Update" [src/MVC3/MvcMusicStore-Assets/readme.txt:3] | Reference resolution fails, or resolves to whatever the host's GAC happens to carry. Section 7 records both behaviours |
| A working **NuGet restore** — because 24 of MVC 4's and 26 of MVC 5's assembly references resolve only from a `packages` directory | MVC 5, MVC 4 | `HintPath` counts, reproducible per Appendix A; MVC 3 has exactly **one** `HintPath` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:39] | **Fatal at compile time.** Section 5 is MVC 5's instance of this |
| The committed **NuGet 2.0 client**, reachable at `$(SolutionDir)\.nuget\nuget.exe` | MVC 4 (active), MVC 5 (declared but absent) | `NuGetExePath` resolves to `$(NuGetToolsPath)\nuget.exe` [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:43] with `NuGetToolsPath` = `$(SolutionDir)\.nuget` [:29]; the self-download fallback is switched **off** [:16]; `CheckPrerequisites` raises `Unable to locate '$(NuGetExePath)'` when it is missing [:71] | Restore-on-build fails. Section 6.1 is MVC 4's instance |

**A note on the `$(VSToolsPath)` guard, because it is weaker than it looks.** The condition on that import tests whether the *property* is non-empty — not whether the *file* exists. And the property is always non-empty by the time the import is evaluated: the project defaults `VisualStudioVersion` to `10.0` when the build does not supply it [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:268] and then defaults `VSToolsPath` from it [:269], and `$(MSBuildExtensionsPath32)` is always defined. So the condition is true in practice, the import is always attempted, and **it fails on any host that lacks the resolved path** — the same missing-import failure MVC 3 suffers unconditionally (section 7.2). The guard protects against an unset property, not against an absent toolchain.

The practical consequence is a build requirement: **`VisualStudioVersion` must resolve to a generation whose web-application targets are actually installed.** A modern MSBuild supplies it (Visual Studio 2022 reports `17.0`) and the path resolves; a bare or older invocation falls through to the project's `10.0` default and the import fails. This is why an MVC 5 or MVC 4 build must be driven by the Visual Studio MSBuild rather than by an arbitrary MSBuild on the `PATH`, and it is one of the recorded fields of the verification run in section 5.4.

MVC 5 and MVC 4 both opt into restore-on-build — `<RestorePackages>true</RestorePackages>` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:24], [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:24] — which is what makes `BuildDependsOn` prepend the `RestorePackages` target [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:57-60]. The difference is that MVC 4's import of that targets file is unconditional while MVC 5's is guarded by an existence check, so the property is live in one edition and inert in the other. Section 6.5 draws the contrast; section 8 records the restore posture.

### 4.3 What each project file demands of the web host

The *current* hosting requirement per edition, read out of the project files. The target-state hosting recommendation is deliberately absent here and is owned by deliverable 06.

| Edition | Declared web host | Evidence | Note |
| --- | --- | --- | --- |
| MVC 5 | **IIS Express**, over plain HTTP on port 43524 | `UseIISExpress` `true` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:18]; `UseIIS` `True`, `DevelopmentServerPort` `43524`, `IISUrl` `http://localhost:43524/` [:281], [:283], [:285] | `IISExpressSSLPort` is **empty** [:19], so no HTTPS endpoint is configured. `NTLMAuthentication` is `False` [:286] |
| MVC 4 | **IIS Express**, over plain HTTP | `UseIISExpress` `true` [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:18]; `UseIIS` `True`, `IISUrl` `http://localhost:4321/` [:346], [:350] | The project also carries a second, disagreeing port: `DevelopmentServerPort` `5928` [:348] against an `IISUrl` on 4321 |
| MVC 3 | **The Visual Studio Development Server**, on port 26641 | `UseIISExpress` **`false`** [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:17]; `UseIIS` **`False`**, `AutoAssignPort` `False`, `DevelopmentServerPort` `26641`, `IISUrl` **empty** [:223-228] | **A requirement no current toolchain satisfies.** The Visual Studio Development Server was superseded by IIS Express and no longer ships. MVC 3 must be re-pointed at a real web host before it can be run at all |

MVC 3's configuration adds two IIS integrated-pipeline compatibility settings that are hosting requirements in their own right: `validateIntegratedModeConfiguration="false"` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:43] and `runAllManagedModulesForAllRequests="true"` [:44]. The first suppresses the integrated-mode configuration check rather than satisfying it; the second routes every request — including static files — through the full managed module pipeline.

All three editions build to `bin\` in both configurations [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:33], [:41], and all three set `WarningLevel` `4` without treating warnings as errors [:36], [:44]. The equivalents hold in MVC 4 [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:30], [:33], [:38], [:41] and MVC 3 [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:23], [:26], [:31], [:34].

### 4.4 One publish-time behaviour per edition, and only one

Six XDT transform files exist — a `Web.Debug.config` and a `Web.Release.config` per edition — and between them they carry exactly **one active transform each in the three Release files and none at all in the three Debug files**:

| File | `xdt:Transform` occurrences | Active | The one that is active |
| --- | --- | --- | --- |
| [src/MVC5/MvcMusicStore/Web.Release.config] | 3 | **1** | `<compilation xdt:Transform="RemoveAttributes(debug)" />` [:18] |
| [src/MVC4/MvcMusicStore/Web.Release.config] | 3 | **1** | the same transform, at the same line [:18] |
| [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Web.Release.config] | 3 | **1** | the same transform, at the same line [:18] |
| [src/MVC5/MvcMusicStore/Web.Debug.config] | 2 | **0** | — |
| [src/MVC4/MvcMusicStore/Web.Debug.config] | 2 | **0** | — |
| [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Web.Debug.config] | 2 | **0** | — |

```bash
# for each of the six files: total xdt:Transform occurrences, then occurrences
# remaining after XML comments are stripped
python - <<'PY'
import re, glob
for p in sorted(glob.glob('src/*/**/Web.[DR]*.config', recursive=True)):
    t = open(p, encoding='utf-8-sig').read()
    print(p, len(re.findall('xdt:Transform', t)),
             len(re.findall('xdt:Transform', re.sub(r'<!--.*?-->', '', t, flags=re.S))))
PY
```

Every other transform in every one of the six files is commented-out template text — for MVC 5, the blocks at [src/MVC5/MvcMusicStore/Web.Release.config:6-16] and [:19-29]. The active transform strips the `debug` attribute that `Web.config` sets to `true` [src/MVC5/MvcMusicStore/Web.config:33] when publishing under the Release configuration.

Two consequences, both narrow and both worth stating:

- **A Release publish is required for a deployed application not to run with debug compilation enabled**, and it is the only thing the repository's transform files do for a deployment. This holds identically in all three editions.
- **`customErrors` is never configured anywhere.** The element appears **24** times across the six XDT files — four per file, reproduced by `git grep -c 'customErrors' -- <the six files> | awk -F: '{s+=$NF} END {print s}'` — and **every one of those occurrences sits inside a comment**, which is what the command above establishes by showing that no non-comment transform other than the `debug` removal survives. So the repository configures no production error-display policy at all. Deliverable 09 owns that as a disclosure question and deliverable 05 owns the replacement design; the deployment fact is that nothing in the repository sets it.

How the one active behaviour is re-expressed after migration is owned by deliverable 05.

---

## 5. MVC 5 — a clean checkout cannot build it; restored, it builds clean

MVC 5 is **the sole migration source**, and for most of this assessment it was **the one edition whose application build could not be verified**. That combination is why this section exists in the form it does. It now has both halves of an answer, and they must not be collapsed into one: §5.1 to §5.3 record a **precondition failure that is a property of the repository** and is unchanged by any host, while §5.4 records the **verification run that has since been performed**, on which the source compiles cleanly.

### 5.1 The observed failure

`xbuild src/MVC5/MvcMusicStore.sln` in the Debug configuration produced a **`CS0246` cascade** — "type or namespace could not be found" — on `ActionResult`, `DbSet`, `Controller` and `ControllerContext`, followed by consequent **`CS0115`** errors on members declared with `override` whose base types were among the types that had failed to resolve.

The cause is singular and mechanical. Every one of MVC 5's assembly references that carries a hint path uses the form `..\packages\<id>.<version>\lib\net45\<assembly>.dll` — for example `..\packages\Microsoft.AspNet.Mvc.5.0.0\lib\net45\System.Web.Mvc.dll` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:78], and likewise `..\packages\EntityFramework.6.0.0\lib\net45\EntityFramework.dll` [:115]. Those paths are **project-relative**, so from `src/MVC5/MvcMusicStore/` they resolve to `src/MVC5/packages`. That directory does not exist in a clean checkout. With `System.Web.Mvc` unresolved, `ActionResult`, `Controller` and `ControllerContext` have no declaring assembly; with `EntityFramework` unresolved, neither does `DbSet`; and the `CS0115` errors follow mechanically from the missing base types.

The compiler was reached and the compiler failed. That is a materially different outcome from MVC 4's, where MSBuild never reached the compiler at all.

### 5.2 The two absences, verified

```bash
# MVC 5 commits no packages payload
test -d src/MVC5/packages && echo present || echo absent          # -> absent
git ls-files 'src/MVC5/*packages/*' | wc -l                        # -> 0

# and no .nuget folder either, in any edition but MVC 4
test -d src/MVC5/MvcMusicStore/.nuget && echo present || echo absent   # -> absent
git ls-files | grep '\.nuget/'
# -> src/MVC4/MvcMusicStore/.nuget/NuGet.Config
#    src/MVC4/MvcMusicStore/.nuget/NuGet.exe
#    src/MVC4/MvcMusicStore/.nuget/NuGet.targets
```

**Absence 1 — no packages payload.** `packages/*` is excluded by [.gitignore:15] and MVC 5 commits nothing under it: 0 tracked files, against 169 for MVC 4 and 46 for MVC 3 (section 8). MVC 5 therefore has no assembly to resolve against until a restore runs.

**Absence 2 — no `.nuget` folder at all**, despite MVC 5's solution declaring one as a solution folder and listing two files inside it: `Project("{2150E333-8FDC-42A3-9474-1A3956D46DE8}") = ".nuget", ".nuget", …` [src/MVC5/MvcMusicStore.sln:8] with `.nuget\NuGet.Config` and `.nuget\NuGet.targets` as its contents [:10-11]. No such folder or files exist anywhere under `src/MVC5`. The project's corresponding import is guarded — `Condition="Exists('$(SolutionDir)\.nuget\NuGet.targets')"` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:295] — so the absence is silent rather than fatal, and its effect is that **MVC 5's own `<RestorePackages>true</RestorePackages>` [:24] is inert**: the property is set, but the target that would honour it is never imported. MVC 5 has no restore wiring of its own. Deliverable 02 §5.2 records the same fact from the dependency side.

Note what is *not* wrong here. MVC 5's hint paths point at `src/MVC5/packages`, which is exactly where a restore driven from `src/MVC5/MvcMusicStore.sln` would place them — the `PackagesDir` convention is `$(SolutionDir)\packages` [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:31], and `SolutionDir` for that solution is `src/MVC5/`. **MVC 5's configuration is internally coherent; only the payload is missing.** That is precisely why its failure is a precondition failure and not a defect, and it is the opposite of MVC 4's situation in section 6.

### 5.3 What this proves, and what it does not

**What it proves.** A **precondition failure**: *MVC 5 cannot be built from a clean checkout without a working restore, and the repository does not carry one.* That is a real, actionable finding — it means any build host, pipeline or developer onboarding step for MVC 5 must include a restore against a reachable package source before a build is attempted, and that no offline build of MVC 5 is possible from the repository as committed. It compounds with the unconfigured restore source that deliverable 02 §6 records: the build needs a restore, and the repository does not say where the restore should go.

**What it does not prove.** Anything whatsoever about whether the application compiles once restored. The observed errors are all reference-resolution consequences; they say nothing about the state of the source. A build that never had its references available has not tested its source.

### 5.4 The verification run this document required — performed, and recorded

The run has been performed. The two states §5.3 leaves open — a source that compiles cleanly once restored, or a source with latent defects — are resolved in favour of the **first**: restored, MVC 5 compiles with no errors and no warnings, in both configurations. The roadmap's first workstream, owned by [03 — Modernization Roadmap](03-modernization-roadmap.md), can be sequenced on that basis rather than on an unknown.

The run had to satisfy nine requirements and record each as evidence. Every one is discharged below, requirement against recorded result:

| Requirement | Why it is required | Recorded result |
| --- | --- | --- |
| **Windows host** | The prescribed toolchain and the repository's own imports and connection strings are Windows-bound (section 2.3) | Windows Server 2022 |
| **.NET Framework 4.8 targeting pack or Developer Pack** | The declared target framework [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:16] | Present, at `C:\Program Files (x86)\Reference Assemblies\Microsoft\Framework\.NETFramework\v4.8` |
| **Visual Studio 2022 MSBuild**, invoked such that `VisualStudioVersion` resolves to an installed generation | The import at [:272] is guarded on a property that is always set rather than on the file existing (§4.2), so it fails on a host where the resolved path is absent. The MSBuild used must therefore be located deliberately, not taken from the `PATH` | MSBuild `17.14.51.32402`, resolved through `vswhere -latest -requires Microsoft.Component.MSBuild -find 'MSBuild\**\Bin\MSBuild.exe'` and invoked by full path, not from `PATH` |
| **`nuget restore` against a declared source**, producing `src/MVC5/packages` | The 26 hint paths of section 5.1. "Declared" matters: deliverable 02 §6 establishes that the repository configures no source, so the source used must be stated explicitly as part of the result | NuGet CLI `6.11.1.2`; restore exit `0`; `src/MVC5/packages` created, satisfying all 26 hint paths |
| **Recorded: tool versions** — MSBuild, NuGet client, targeting pack | Without them the result is not reproducible and cannot be compared against a later run | MSBuild `17.14.51.32402`; NuGet `6.11.1.2`; the 4.8 reference assemblies above |
| **Recorded: the restore source actually used** | Per the above; the result is otherwise a statement about one host rather than about the repository | `nuget.org`, `https://api.nuget.org/v3/index.json` — the only source registered on the host, inherited from host configuration because the repository declares none. Deliverable 02 §6's finding is unchanged; this records which source *this* result rests on |
| **Recorded: the configuration built** (Debug and Release both, since the Release publish carries the one active transform of section 4.4) | Debug-only evidence would leave the publish path unverified | **Debug and Release**, both with `/t:Rebuild`, both exit `0` |
| **Recorded: the test result** | Vacuous today (section 9), but the field must exist and be filled so that its emptiness is a recorded fact rather than an omission | **None** — no test exists anywhere in the repository (section 9). The field is filled, not omitted |
| **Recorded: warning count, with `WarningLevel 4` in force** [:36] | Warnings are not errors in this project, so a "success" can carry diagnostics that matter to the port | **0 warnings, 0 errors**, in both configurations |

Three things this result does **not** establish, stated because a green build invites all three:

- **It says nothing about the 29 Razor views.** `MvcBuildViews` is `false` (section 12), so no view was compiled. A zero-warning build and an un-compiled view set are entirely compatible, and the views remain the largest unverified surface in the migration source.
- **It does not retire the precondition failure of §5.1 to §5.3.** That finding is about the repository, not the compiler: a clean checkout still cannot build, still has no restore wiring of its own (§5.2), and still does not record where its packages come from (§8). Every build host and onboarding path must supply a restore.
- **It is one host's result.** That is why the versions and the source are recorded above: a later run on a different inventory is comparable to this one only because these fields exist.

What it does establish is worth stating plainly, because it is what the roadmap needed: **the MVC 5 source has no latent compile defects at `WarningLevel 4`**, so the port begins from a compiling baseline. Downstream deliverables may now report MVC 5 as building cleanly *once restored*, and must not report it as building from a clean checkout. Deliverable 07's risk on this item narrows accordingly — from "the sole migration source may not compile" to the residual restore-and-reproducibility precondition recorded here and in deliverable 02 §6.

### 5.5 Why the honest status mattered

Before the run, an assessment that reported "MVC 5 builds clean" would have been asserting something nobody had observed. The temptation was real, because the observed failure had an obvious and innocuous explanation — a missing directory, not a code defect — and it is tempting to reason from the explanation to the conclusion. But the conclusion does not follow from the explanation, and the honest statement was the more useful one: **MVC 5 cannot build from a clean checkout, and whether it compiles once restored is a separate question.** The first half was an actionable finding about the build system; the second was a scoped, cheap, one-shot verification task.

Both halves survived being answered, and that is the point. The verification run confirmed the guess that the failure was innocuous — but it is now a *recorded observation with tool versions behind it* rather than a plausible inference, and the clean-checkout finding it sits beside is unaffected. Had the run gone the other way, this section is where the difference would have shown up. Reporting the two facts separately is what kept them both.

---

## 6. MVC 4 — two independent, platform-independent defects, plus a stale solution

MVC 4's builds fail **before compilation**, for reasons that are entirely in the committed configuration. Two defects sit in the project file and a third in one of its two solution files. All three are platform-independent: each is a path that does not exist, and a path that does not exist fails identically under every MSBuild implementation.

### 6.1 Defect 1 — a misplaced, unconditional NuGet target import

The last import in the project file is, verbatim:

```xml
<Import Project="$(SolutionDir)\.nuget\nuget.targets" />
```

at [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:360], **with no `Condition` attribute**.

Two facts make it fatal. First, `$(SolutionDir)` resolves to `src/MVC4/` when the project is built through `src/MVC4/MvcMusicStore.sln` — the solution whose project path is correct — and the project's own fallback agrees, defaulting `SolutionDir` to `..\` relative to the project directory [:23]. Second, **there is no `.nuget` folder at `src/MVC4/`.** The only one in the repository is one level deeper, at `src/MVC4/MvcMusicStore/.nuget`:

```bash
test -d src/MVC4/.nuget && echo present || echo absent   # -> absent
git ls-files | grep '\.nuget/'                            # -> the three files, all under src/MVC4/MvcMusicStore/.nuget/
```

Because the import is unconditional, **MSBuild on Windows raises the same missing-import error** — the `MSB4019` class of failure, "the imported project was not found" — during project evaluation, before any target runs and therefore before the compiler is invoked. This is not a Mono artefact and must not be reported as one — and it is no longer a deduction. The Windows verification run of §3.1 reproduced it directly: building `src/MVC4/MvcMusicStore.sln` with MSBuild `17.14.51.32402` and no property overrides exits `1` with `MSB4019` at `MvcMusicStore.csproj(360,3)`, naming the evaluated path `src/MVC4/.nuget/nuget.targets` — the directory that section 6.1 shows is absent, reported by the very toolchain the environment prescribes.

A smaller, related detail sits in the same line and is worth recording because it bites on case-sensitive filesystems: the import names lowercase **`nuget.targets`** while the committed file is **`NuGet.targets`**. On Windows this is immaterial; on a case-sensitive filesystem it is a second reason the import cannot resolve even when pointed at the right directory. The repository-wide path-casing audit is owned by deliverable 06; this instance is recorded here because it is a build path.

### 6.2 Defect 2 — package hint paths pointing outside the committed payload

The project carries **24** assembly hint paths, and all 24 are of the form `..\packages\…`:

```bash
grep -c 'HintPath>\.\.\\packages' src/MVC4/MvcMusicStore/MvcMusicStore.csproj   # -> 24
grep -c '<HintPath>' src/MVC4/MvcMusicStore/MvcMusicStore.csproj                # -> 24  (every hint path takes this form)
```

Hint paths are **project-relative**, so from `src/MVC4/MvcMusicStore/` they resolve to `src/MVC4/packages` — **a directory that does not exist**. The committed payload is at `src/MVC4/MvcMusicStore/packages`, one level deeper, and **no hint path in the project references that location**:

```bash
test -d src/MVC4/packages && echo present || echo absent                    # -> absent
git ls-files 'src/MVC4/MvcMusicStore/packages/*' | wc -l                    # -> 169
```

**All 24 are therefore unresolvable as committed**, and the 169 tracked files of the committed payload are referenced by nothing.

The reason is worth stating, because it is the sharpest available diagnosis. The `..\packages` form is not arbitrary: it matches the restore convention, which places packages at `$(SolutionDir)\packages` [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:31]. So the hint paths are **internally consistent with the restore configuration, on the assumption that `SolutionDir` is `src/MVC4/`** — which is exactly what the correct solution file makes it. What is misplaced is the *committed payload*, which sits where a restore would have put it had `SolutionDir` been the project directory instead, i.e. under the stale solution of section 6.4.

### 6.3 Why fixing either defect alone leaves the build broken

The two defects pull `SolutionDir` in opposite directions, and that is the crux:

| `SolutionDir` value | Defect 1 — does `$(SolutionDir)\.nuget` exist? | Defect 2 — do the hint paths resolve? |
| --- | --- | --- |
| `src/MVC4/` (what the correct solution gives) | **No** — `src/MVC4/.nuget` is absent, so the unconditional import at `:360` fails | Points at `src/MVC4/packages`, which a restore would create — but no restore can run, because defect 1 already failed evaluation |
| `src/MVC4/MvcMusicStore/` | **Yes** — `.nuget` is there | Still points at `src/MVC4/packages`, because hint paths are project-relative and unaffected by `SolutionDir` |

**Stated plainly: the repository contains the two halves of a working configuration in two different places, and no single committed configuration has both.** The `.nuget` folder is where only the stale solution would find it; the hint paths point where only the correct solution's restore would fill. Fixing the import alone leaves 24 unresolved references; providing the packages alone leaves the import fatal. Both must be addressed, and a plan that treats this as one problem will under-scope it.

That is also why MVC 4 is buildable *only* with build-time overrides rather than as committed — supplying `SolutionDir` one level deeper so the import resolves, while suppressing restore-on-build so the absent `src/MVC4/packages` is not consulted. The Windows verification run of §3.1 confirms the analysis in both directions: without the overrides the build fails at `MSB4019`; with `SolutionDir` pointed one level deeper and restore-on-build suppressed, the same solution exits `0` with zero warnings. **Those overrides are host-side workarounds, not fixes** — they are passed on the command line and change nothing in the repository, and this assessment applies none of them to any tracked file. A reader should draw the intended conclusion from the pairing: MVC 4 is buildable, and it is buildable only by a host that already knows about a defect the repository does not announce.

### 6.4 The stale fourth solution

The repository has **four solution files for three projects**:

```bash
git ls-files '*.sln'
# -> src/MVC3/MvcMusicStore-Completed/MvcMusicStore.sln
#    src/MVC4/MvcMusicStore.sln
#    src/MVC4/MvcMusicStore/MvcMusicStore.sln      <- the stale one
#    src/MVC5/MvcMusicStore.sln
```

`src/MVC4/MvcMusicStore/MvcMusicStore.sln:4` declares its project as `"MvcMusicStore", "MvcMusicStore\MvcMusicStore.csproj"`. Evaluated from that solution's own directory, `src/MVC4/MvcMusicStore/`, the path resolves to `src/MVC4/MvcMusicStore/MvcMusicStore/MvcMusicStore.csproj` — **which does not exist.**

```bash
test -f src/MVC4/MvcMusicStore/MvcMusicStore/MvcMusicStore.csproj && echo present || echo absent   # -> absent
```

MSBuild fails to load the solution — the `MSB3202` class of failure, "the project file was not found" — so this solution never reaches the project, let alone the compiler. Observed directly on the Windows verification host (§3.1): exit `1`, `MSB3202`, naming `src/MVC4/MvcMusicStore/MvcMusicStore/MvcMusicStore.csproj`. It is the only solution in the repository that declares `NuGet.exe` among its solution items [src/MVC4/MvcMusicStore/MvcMusicStore.sln:6-12], which is consistent with it being the file under which the committed payload of section 6.2 was originally produced.

**Taken together, these three defects are why both MVC 4 solutions fail before compilation**, and they fail for different reasons: the correct solution at `src/MVC4/MvcMusicStore.sln` fails on the two project-file defects of sections 6.1 and 6.2, while the stale solution fails on its own unresolvable project path. Anyone diagnosing this by trying the other solution file will get a different error and conclude, wrongly, that there is one problem.

Which solution is which is a build requirement in its own right: **`src/MVC4/MvcMusicStore.sln` is the MVC 4 solution to use, and `src/MVC4/MvcMusicStore/MvcMusicStore.sln` must not be used.** Deliverable 08 owns the debt entry for the stale file; the requirement to avoid it is owned here.

### 6.5 The contrast that shows this is MVC 4's defect, not the era's

It would be easy to file all of this under "2012-era project files are fragile". The repository refutes that directly: **MVC 5 conditions the very imports MVC 4 and MVC 3 leave unguarded.**

| Import | MVC 5 | MVC 4 | MVC 3 |
| --- | --- | --- | --- |
| `$(SolutionDir)\.nuget\NuGet.targets` | `Condition="Exists('$(SolutionDir)\.nuget\NuGet.targets')"` — **guarded on the file existing** [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:295] | **no condition at all** [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:360] | not imported |
| `WebApplication.targets` via `$(VSToolsPath)` | `Condition="'$(VSToolsPath)' != ''"` — guarded on a property that is always set (§4.2) [:272] | the same weak guard [:337] | not imported |
| `WebApplication.targets` at the hard-coded v10.0 path | `Condition="false"` — inert [:273] | `Condition="false"` — inert [:338] | **no condition at all** [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:209] |

The decisive row is the first, and it is the one the failures turn on. **MVC 5 guards its NuGet import on the target file actually existing; MVC 4 does not guard the equivalent import at all.** That single difference is why MVC 5's missing `.nuget` folder is silent (section 5.2) and MVC 4's is fatal (section 6.1). In the third row the polarity reverses: MVC 5 and MVC 4 both neutralise the hard-coded `v10.0` path with `Condition="false"`, and MVC 3 imports it unguarded.

Stated without overclaiming: **MVC 5 is written more defensively than MVC 4 and MVC 3, but not uniformly so** — its `$(VSToolsPath)` guard is the weak kind analysed in section 4.2, and it shares that weakness with MVC 4. What the table establishes is the point that matters here: **these are defects in specific project files, not properties of the vintage**, since the same repository and the same era contain both the guarded and the unguarded form of the same import. A migration plan should treat them accordingly. Deliverable 02 §4.3 tabulates the same imports as dependencies; the build consequence is what this table adds.

---

## 7. MVC 3 — compiles, but its requirements differ by toolchain rather than being absolute

### 7.1 What the Mono build produced

`xbuild src/MVC3/MvcMusicStore-Completed/MvcMusicStore.sln` in Debug produced `bin/MvcMusicStore.dll`, with **three unresolved-reference warnings**: `System.Web.WebPages` `1.0.0.0`, `System.Web.Helpers` `1.0.0.0` and `System.Web.Entity` could not be resolved. Mono resolves enough from its own GAC and targets to compile in spite of them.

Two of the three are visible in the project as bare references with no hint path — `System.Web.WebPages, Version=1.0.0.0` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:43] and `System.Web.Helpers, Version=1.0.0.0` [:44] — and the third is the framework assembly `System.Web.Entity` [:50].

The one reference MVC 3 *does* hint is its ORM: `..\packages\EntityFramework.4.1.10331.0\lib\EntityFramework.dll` [:39]. From `src/MVC3/MvcMusicStore-Completed/MvcMusicStore/` that resolves to `src/MVC3/MvcMusicStore-Completed/packages`, **which is committed** — 46 tracked files. MVC 3 is thus the only edition whose single hint path is satisfied by the repository as committed, which is the whole reason its build got as far as it did.

**No output artefact size is recorded**, for the reason given in section 3: the figure is not stable across toolchains and would invite a false comparison.

### 7.2 On Windows the picture differs

Two of MVC 3's requirements are toolchain-dependent rather than absolute, and both are stricter on a current Windows host than they were under Mono.

**The MVC framework assembly has no hint path.** `System.Web.Mvc, Version=3.0.0.0` is a bare reference [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:42] with no package folder behind it, so it resolves only from a **machine-wide install of the ASP.NET MVC 3 Tools Update** — a separately installed, out-of-support product. The repository corroborates the generation in its own words: the tutorial assets are "updated for the ASP.NET MVC 3 Tools Update" [src/MVC3/MvcMusicStore-Assets/readme.txt:3]. Configuration corroborates it a second time, listing `System.Web.Mvc, Version=3.0.0.0` among the assemblies the ASP.NET compilation system must load [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:21] and redirecting older references forward to `3.0.0.0` [:50-51]. Deliverable 02 §4.1 records the same assembly as an undeclared dependency; the build requirement it creates — **a machine-wide product install on the build host** — is recorded here.

**The web-application targets import is unconditional and points at a Visual Studio 2010-era path.** `$(MSBuildExtensionsPath32)\Microsoft\VisualStudio\v10.0\WebApplications\Microsoft.WebApplication.targets` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:209] carries no `Condition`, so on a host without that exact path MSBuild fails during evaluation — the same `MSB4019` class of failure as MVC 4's defect 1, and equally platform-independent. A Visual Studio 2022-only machine does not carry a `v10.0` extensions path.

The practical consequence for a build host is therefore: **MVC 3 requires either a Visual Studio 2010-era `v10.0` web-application targets path on disk, or a compatibility shim that supplies one.** A shim is a host-side arrangement, not a repository change, and this assessment makes none.

The Windows verification run of §3.1 bears this out from the other side. On a host where the machine-wide MVC 3 assemblies **and** a `v10.0` targets path are both present, MVC 3 restores and builds in Debug and Release at exit `0` with **zero warnings** — the three unresolved-reference warnings of §7.1 do not appear, because the assemblies they name resolve there. Nothing in the repository changed between the two runs, so the difference is entirely the host's product inventory. That is precisely this section's claim, and it cuts both ways: the same absence that makes the build succeed here makes it fail on a host without those installs.

### 7.3 Its committed web host no longer ships

Recorded in section 4.3 and repeated here because it is MVC 3's most consequential runtime requirement: MVC 3 declares `UseIISExpress` `false` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:17] and `UseIIS` `False` with an empty `IISUrl` [:223], [:227-228], which selects the Visual Studio Development Server. That server no longer ships in any current Visual Studio. **MVC 3 cannot be run from its committed configuration on a supported toolchain without being re-pointed at a real web host**, and that is independent of its two database requirements in section 10.

---

## 8. Restore posture per edition — none of it is modern

```bash
git ls-files | grep -c '/packages/'                                   # -> 215
git ls-files 'src/MVC4/MvcMusicStore/packages/*' | wc -l               # -> 169
git ls-files 'src/MVC3/MvcMusicStore-Completed/packages/*' | wc -l     # -> 46
git ls-files 'src/MVC5/*packages/*' | wc -l                            # -> 0
```

| Edition | Committed payload | Restore mechanism as committed | Can it build offline? |
| --- | --- | --- | --- |
| MVC 5 | **none** — 0 tracked files | None wired: `RestorePackages` is `true` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:24] but the targets file that honours it is imported under an `Exists(...)` guard [:295] against a `.nuget` folder that does not exist | **No.** A restore against a reachable source is mandatory |
| MVC 4 | **169 tracked files** at `src/MVC4/MvcMusicStore/packages`, referenced by no hint path (section 6.2) | **MSBuild-integrated restore** — `RestorePackages` `true` [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:24] driving the `RestorePackages` target [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:76-83] through the committed client | Not as committed — section 6.3 |
| MVC 3 | **46 tracked files** at `src/MVC3/MvcMusicStore-Completed/packages`, which its single hint path does reference | **None** — no `.nuget` folder, no NuGet import, no `RestorePackages` property | **Yes**, for its one hinted reference; its machine-wide requirements are separate (section 7.2) |

**215 tracked files sit under the two committed payloads despite `packages/*` being excluded** by [.gitignore:15] (with `Packages/` additionally at [:33]). Deliverable 02 §7.2 records the payload composition and 08 owns the debt framing; the build fact is the one in the table.

**MVC 4's restore client is committed, pinned and long deprecated.** `src/MVC4/MvcMusicStore/.nuget/NuGet.exe` is a tracked NuGet **2.0.30828.5** binary — a 2012-era client — and it is required rather than optional: the self-download fallback is switched off [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:16], and `CheckPrerequisites` raises `Unable to locate '$(NuGetExePath)'` when it is missing [:71]. MSBuild-integrated restore of this kind was dropped with the NuGet 3 generation, so **a current NuGet client cannot be substituted into this wiring** — it can only replace it. Deliverable 02 §5.1 carries the client's verified size, versions and hash; they are not repeated here.

Two further build-relevant properties of that wiring, because they change what a restore does rather than merely where it looks:

- **Restore consent is demanded.** `RequireRestoreConsent` defaults to `true` [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:13] and adds `-RequireConsent` to the restore command [:51], so a build host that has not opted into package restore fails the restore rather than performing it.
- **The restore command is issued with an empty `-source`.** `PackageSources` resolves from an empty item list [:44] into the command at [:53]. The consequence — that the effective source set is a property of the build host and not of the repository — is a finding owned by deliverable 02 §6, corrected there against Technical Specification §3.3, and cross-referenced rather than restated. Its bearing on *this* document is narrow and worth naming: **no build of MVC 4 or MVC 5 is reproducible across hosts from repository evidence alone**, because the repository does not record where its packages come from.

**No lockfile exists in any edition** — `git ls-files '*packages.lock.json'` returns nothing — so transitive resolution is not pinned either. Deliverable 02 §7.1 owns that finding.

---

## 9. There are no tests

```bash
git grep -lIiE 'TestClass|\[Fact\]|\[TestMethod\]|xunit|nunit|Microsoft\.VisualStudio\.TestTools' \
    -- 'src/' | grep -v '/packages/' | wc -l          # -> 0
git ls-files | grep -i test | wc -l                    # -> 0
```

**Zero matches.** No test project, no test class, no test-framework reference and no file whose name contains "test" exists anywhere in application source, project files, configuration or solution files. Prescribed build step 5 — "run unit tests if present" — is therefore satisfied **vacuously**: there is no command to run.

The only trace of testing anywhere in the repository is an ignore rule for results that are never produced: `/TestResults` at [.gitignore:5].

The consequence is the one that matters most to everything downstream: **there is no regression baseline and no automated way to demonstrate behaviour preservation.** Nothing in the repository today would detect a behaviour change introduced by a port. Deliverable 03 places test authoring ahead of the port for exactly this reason, and deliverable 07 carries the absent baseline as a first-order risk. The suite's architecture, coverage and fixtures are specified by deliverables 03 and 05 and are deliberately not designed here — this document records only that the baseline does not exist and that the prescribed step was vacuous.

---

## 10. Database components required to run — stated per edition

### 10.1 The topology, with the catalog store and the credential store separated

**A single "two LocalDB databases" statement would be wrong for all three editions**, which is why the two stores get their own columns below. In MVC 5 they are two databases on one engine. In MVC 4 they are two databases on one engine whose *instance name* the edition's own documentation contradicts. In MVC 3 they are **two entirely different database engines**.

| Edition | Catalog store | Credential store | Provisioning artifacts |
| --- | --- | --- | --- |
| **MVC 5** | LocalDB `MSSQLLocalDB`, file-attached `MvcMusicStore.mdf` under `src/MVC5/MvcMusicStore/App_Data/`, Windows integrated authentication [src/MVC5/MvcMusicStore/Web.config:13] | LocalDB, **same instance**, a **separate** file-attached ASP.NET Identity 1.0 database via `DefaultConnection`, with `Initial Catalog=aspnet-MvcMusicStore-20131025034205` [src/MVC5/MvcMusicStore/Web.config:12] | The two committed `.mdf`/`.ldf` pairs under `App_Data/`; first-run creation and seeding by the EF initializer (section 10.4). **No schema script ships for MVC 5** — the repository's only three `.sql` files belong to MVC 4 and to the MVC 3 tutorial assets (section 10.5) |
| **MVC 4** | LocalDB **`(LocalDB)\v11.0` as committed** [src/MVC4/MvcMusicStore/Web.config:19], file-attached `MvcMusicStore.mdf`, Windows integrated authentication [:20-21] — an instance name its own README contradicts (section 10.3) | LocalDB, same instance, a **separate** SimpleMembership database, `Integrated Security=SSPI` [src/MVC4/MvcMusicStore/Web.config:13], [:14-16] | Two required `.mdf`/`.ldf` pairs under `App_Data/`, plus two **byte-identical** `MvcMusicStore-Create.sql` copies, **neither runnable as written** (section 10.5; the defect itself is owned by deliverables 12 and 08). A third pair, `MvcMusicStore-work.mdf` and `MvcMusicStore_log-work.ldf`, is referenced by no configuration and is **unreferenced scratch debt, not a runtime requirement** |
| **MVC 3** | **SQL Server Compact 4.0** — a first-run `.sdf` created from the only connection string the edition declares, `Data Source=\|DataDirectory\|MvcMusicStore.sdf` with `providerName="System.Data.SqlServerCe.4.0"` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:55-59]. **No `.sdf` is committed**, so a machine-wide install of the retired provider is required | **A SQL Server instance — not SQL Server Compact.** `<roleManager enabled="true" />` [:15] and Forms authentication [:26-28] are both enabled, but the file defines **no** membership provider, **no** role provider and **no** `LocalSqlServer` connection string, so classic `Membership` and `Roles` resolve through the **machine-level** ASP.NET SQL providers against the machine's own connection-string setting | **No database under the application's own `App_Data`** — `git ls-files 'src/MVC3/MvcMusicStore-Completed/*' \| grep App_Data` returns nothing. The catalog `.mdf`, the `ASPNETDB.MDF` credential store and the repository's **one runnable** schema script are all **tutorial assets** under `src/MVC3/MvcMusicStore-Assets/Data/` |

Verification of the MVC 3 absences, because they are the load-bearing part of that row:

```bash
git grep -n -iE '<membership|<roleManager|LocalSqlServer|<profile' \
    -- 'src/MVC3/MvcMusicStore-Completed/*'
# -> src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:15:    <roleManager enabled="true" />
#    and nothing else: no <membership>, no <providers>, no LocalSqlServer

git grep -il 'ASPNETDB' -- 'src/' | wc -l     # -> 0
```

The second command is the sharper of the two: **the committed `ASPNETDB.MDF` is referenced by no configuration file anywhere in the repository.** It is an orphaned tutorial asset, and using it requires supplying a connection string that the repository does not contain.

### 10.2 MVC 3 requires two different database engines simultaneously, and its membership configuration is inherited rather than declared

Both statements follow directly from the table, and both need to be said out loud because neither is what a reader expects.

**Two engines at once.** MVC 3's catalog is SQL Server Compact 4.0, a file-based embedded engine with a native component [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:58]. Its credential store cannot be that engine: the classic ASP.NET SQL membership and role providers target SQL Server, not SQL Server Compact. So running MVC 3 requires **a SQL Server Compact 4.0 install and a SQL Server instance, at the same time**, and neither is satisfied by anything committed under the application. The repository's own tutorial note frames the committed catalog `.mdf` as the *alternative* to SQL Server CE rather than a companion to it — "The Data directory contains a database (only used if you won't be using SQL Server CE)" [src/MVC3/MvcMusicStore-Assets/readme.txt:6] — which is a third database option, not a resolution of the two-engine requirement.

**Inherited, not declared.** MVC 3's effective membership and role configuration is **not in the repository.** With `roleManager` enabled and no provider defined, the providers and their connection string come from the host's machine-level ASP.NET configuration. That has a direct methodological consequence, and it is the reason this row is flagged rather than closed:

> **Open verification item.** MVC 3's credential-store requirement must be **verified on the supported Windows runtime before it is stated as final** — the actual machine-level provider and the actual connection string it resolves. An inherited default is a property of the *host*, not of the repository, and no amount of reading the repository can settle it. What the repository does settle, and what is stated as final here, is the negative: **MVC 3 declares no membership store of its own.**

### 10.3 MVC 4's README contradicts its own `Web.config` — and itself

MVC 4 is the edition where the documentation and the committed configuration disagree, in more than one direction. All locators:

| Source | Instance name it gives | Locator |
| --- | --- | --- |
| Committed `Web.config`, credential store | `(LocalDb)\v11.0` | [src/MVC4/MvcMusicStore/Web.config:13] |
| Committed `Web.config`, catalog store | `(LocalDB)\v11.0` | [src/MVC4/MvcMusicStore/Web.config:19] |
| README, documenting the catalog connection string | `(LocalDb)\MSSQLLocalDB` | [src/MVC4/README.md:110-112] |
| README, documenting the credential connection string | `(LocalDb)\MSSQLLocalDB` | [src/MVC4/README.md:117-119] |
| README, manual-connection instructions | `(LocalDB)\MSSQLLocalDB` | [src/MVC4/README.md:45] |
| README, telling non-VS2022 users to substitute | `(LocalDB)\v11.0` | [src/MVC4/README.md:102] and [src/MVC4/README.md:122] |
| README, telling users who find v11.0 unavailable to substitute | `(LocalDB)\MSSQLLocalDB` | [src/MVC4/README.md:139] |

Three findings, in increasing order of consequence:

1. **The README documents a value the repository does not commit.** It presents both connection strings as `MSSQLLocalDB` [src/MVC4/README.md:110-112], [:117-119]; the committed file says `v11.0` [src/MVC4/MvcMusicStore/Web.config:13], [:19].
2. **The substitution advice runs in both directions.** [src/MVC4/README.md:102] and [:122] tell the reader to replace `MSSQLLocalDB` with `v11.0`; [:139] tells the reader to replace `v11.0` with `MSSQLLocalDB`. The README therefore contradicts itself as well as the configuration, and a reader following it cannot determine which value is intended.
3. **The committed value is unavailable under the README's own stated prerequisite.** The README requires Visual Studio 2022 [src/MVC4/README.md:7] and states that LocalDB is installed with it [:8]. **Visual Studio 2022 installs no `v11.0` instance** — `v11.0` is the SQL Server 2012 LocalDB generation, superseded by `MSSQLLocalDB`. So MVC 4, as committed, points at an instance that the toolchain it documents does not provide.

The build-and-deployment requirement this creates is unambiguous even though the sources are not: **MVC 4's committed connection strings must be re-pointed at an available LocalDB instance before the application can run**, and that is a code change, which this assessment does not make. The repair is out of scope under the approval gate; deliverable 08 owns the debt entry. Note that this affects **runtime only** — MVC 4's build is unaffected by its connection strings, and its build failures (section 6) have entirely separate causes.

### 10.4 The three initializers' first-run behaviour

All three editions ship the same class shape: `SampleData : DropCreateDatabaseIfModelChanges<MusicStoreEntities>` at [src/MVC5/MvcMusicStore/Models/SampleData.cs:9], [src/MVC4/MvcMusicStore/Models/SampleData.cs:9] and [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Models/SampleData.cs:9]. What differs is where it is registered, what it seeds, and what else runs alongside it.

| Edition | Registered at | Seed payload | What happens on first run |
| --- | --- | --- | --- |
| **MVC 5** | **Twice** — [src/MVC5/MvcMusicStore/Global.asax.cs:20] and [src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:16] | **826 physical lines**; **15 genres, 303 artists, 462 albums** | The catalog database is attached, its schema created if the model does not match, and the seed written. The administrator account and `Administrator` role are then provisioned from `appSettings` by `CreateAdminUser()` [src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:21-44], which builds its own context, `UserManager` and `RoleManager` [:27-29] |
| **MVC 4** | **Once** — [src/MVC4/MvcMusicStore/App_Start/AppConfig.cs:16] | **826 physical lines**; **15 genres, 303 artists, 462 albums** (its `SampleData.cs` is byte-identical to MVC 5's — deliverable 01 §10.1) | The same catalog sequence, **plus a separate credential-store creation path**: `Database.SetInitializer<UsersContext>(null)` [src/MVC4/MvcMusicStore/Filters/InitializeSimpleMembershipAttribute.cs:28], then `context.Database.Exists()` [:34] and, if absent, `((IObjectContextAdapter)context).ObjectContext.CreateDatabase()` [:37], then `WebSecurity.InitializeDatabaseConnection(…, autoCreateTables: true)` [:41]. Administrator provisioning follows [src/MVC4/MvcMusicStore/App_Start/AppConfig.cs:21-37] |
| **MVC 3** | **Once** — [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Global.asax.cs:34], inside `Application_Start` [:32-40] | **430 physical lines**; **10 genres, 149 artists, 246 albums** | The catalog `.sdf` is created and seeded through the SQL Server Compact provider. **No administrator is provisioned at all** — MVC 3 has no `App_Start` folder and no provisioning code, so its `Administrator` role is never created |

Seed content is verified by counting the initializers in each file, per Appendix A. Two facts fall out of the last column and both are deployment requirements rather than curiosities:

- **MVC 3's first run produces a different catalog from MVC 4's and MVC 5's** — 10 genres against 15, 149 artists against 303, 246 albums against 462. Any environment intended to compare editions must not assume a common dataset.
- **MVC 4 and MVC 5 both require their administrator credential to exist in `appSettings` at first run** [src/MVC4/MvcMusicStore/Web.config:25-26], [src/MVC5/MvcMusicStore/Web.config:16-17] — values cited but not reproduced, per section 1.3. Provisioning happens at application start, not on demand, so **a provisioning failure is a startup failure**. MVC 4 surfaces it: its initializer wraps the sequence in a `try`/`catch` that rethrows as `InvalidOperationException` [src/MVC4/MvcMusicStore/Filters/InitializeSimpleMembershipAttribute.cs:43-46]. MVC 5 does not: its provisioning method is declared `private async void` [src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:21], so a faulted task is unobservable and a failed provisioning is silent.

**The destructive property of `DropCreateDatabaseIfModelChanges` is the deployment requirement this section is obliged to state**, in operational terms rather than as a debt entry: *if the model does not match the database, the database is dropped and recreated.* Applied to a database holding real orders and personal data, that is data loss with no prompt. The repository's own documentation describes the benign half of the behaviour — the automatic attach, initialize and seed sequence [src/MVC5/README.md:26-31] — and, in its troubleshooting section, the destructive half as a remedy: delete the `.mdf` and `.ldf` files and "the database will be recreated automatically" [src/MVC5/README.md:98-99].

So the requirement is: **any deployment that points one of these applications at a database containing data it cannot afford to lose must first disable the initializer.** The debt framing and severity are owned by deliverable 08; the replacement schema lifecycle is owned by deliverable 05; the hosting sequence for applying schema changes is owned by deliverable 06.

MVC 5 registers the initializer twice, and the effect is worth stating precisely so it is not overstated: `Database.SetInitializer<TContext>` *sets* the strategy rather than adding to a list, so the second registration replaces the first and **exactly one initialization runs**. This is duplicated startup configuration, not a doubled destructive path. Deliverable 01 §3.4 owns the analysis.

### 10.5 The schema scripts as deployment inputs

Three `.sql` files are tracked, and their usability differs:

```bash
git ls-files '*.sql'
# -> src/MVC3/MvcMusicStore-Assets/Data/MvcMusicStore-Create.sql
#    src/MVC4/MvcMusicStore-Create.sql
#    src/MVC4/MvcMusicStore/MvcMusicStore-Create.sql
```

| Script | First statement | Usable as a deployment input? | Tables it creates |
| --- | --- | --- | --- |
| [src/MVC3/MvcMusicStore-Assets/Data/MvcMusicStore-Create.sql] | `USE [MvcMusicStore]` [:1] | **Yes, given a pre-existing database of that name.** The repository's one runnable script | Six **singular**-named tables |
| [src/MVC4/MvcMusicStore-Create.sql] | `USE [C:\USERS\JON\…\APP_DATA\MVCMUSICSTORE.MDF]` [:1] | **No** — a hard-coded developer path to an attached MDF | Six **plural**-named tables |
| [src/MVC4/MvcMusicStore/MvcMusicStore-Create.sql] | identical to the above [:1] | **No** — byte-identical duplicate, same defect | identical |

The table names are the load-bearing difference, so each is cited individually. The MVC 3 asset script declares `Order` [src/MVC3/MvcMusicStore-Assets/Data/MvcMusicStore-Create.sql:41], `Genre` [:70], `Artist` [:101], `Album` [:272], `OrderDetail` [:551] and `Cart` [:579]. Both MVC 4 copies declare `Genres` [src/MVC4/MvcMusicStore/MvcMusicStore-Create.sql:52], `Orders` [:81], `Artists` [:108], `Albums` [:277], `OrderDetails` [:549] and `Carts` [:570].

One encoding detail belongs here because it affects execution rather than reading: **all three scripts are UTF-16 with a byte-order mark** — the first two bytes of each are `FF FE`, and the MVC 3 asset script carries a doubled mark, `FF FE FF FE`. Any tool used to execute them must handle UTF-16 input, which rules out a naive byte-oriented pipeline and is worth knowing before a deployment step is scripted around one.

The duplication is proven rather than assumed: both MVC 4 copies are 153,594 bytes with the same SHA-256, `D577AAA51949E54D1C83D57E23F1BB96A840661EFF9FE478F2CF8A53DD182C9D` (reproducing command in Appendix A). **Neither copy is runnable as written**, so neither is a usable schema baseline; the defect itself belongs to deliverables 12 and 08. MVC 4's own README confirms the requirement to edit the script before using it — "Update the `USE` statement at the top of the file to point to your LocalDB instance" [src/MVC4/README.md:82], repeated in its troubleshooting section [:156] — and accurately lists the plural tables the script creates [:91].

**The one runnable script is not a substitute for the missing MVC 5 one.** Its six tables are singular-named and MVC 4's are plural-named, so the two schemas are not interchangeable, and neither can be assumed to match what an EF initializer produces for the edition in question. Since MVC 5 ships no script at all, **the authoritative source of MVC 5's schema is the committed `src/MVC5/MvcMusicStore/App_Data/MvcMusicStore.mdf` itself**, and any schema baseline for it has to be extracted rather than read from a script. Deliverable 05 owns that extraction as a gate on the data migration.

---

## 11. Permissions required to build, deploy and run

Every permission below is derived from a repository construct rather than from a hosting convention. Each row states the construct, the permission it implies, and what fails without it.

### 11.1 Filesystem — the worker process must be able to write `App_Data`

| Construct | Editions | Permission implied |
| --- | --- | --- |
| `AttachDbFilename=\|DataDirectory\|\MvcMusicStore.mdf` | MVC 5 [src/MVC5/MvcMusicStore/Web.config:13], MVC 4 [src/MVC4/MvcMusicStore/Web.config:20] | **Read *and write*** on the application's `App_Data` folder for the worker-process identity. Attaching a database file is not a read operation: the engine writes to the `.mdf` and to its `.ldf`, so read-only access fails at attach time rather than at first write |
| `AttachDbFilename=\|DataDirectory\|\aspnet-MvcMusicStore-….mdf` | MVC 5 [src/MVC5/MvcMusicStore/Web.config:12], MVC 4 [src/MVC4/MvcMusicStore/Web.config:16] | The same, for the credential store — so **both** databases in both editions sit under this requirement |
| `Data Source=\|DataDirectory\|MvcMusicStore.sdf` with no committed `.sdf` | MVC 3 [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:57-58] | **Create-file** permission in `App_Data`, not merely write: the file does not exist and the provider must create it on first run |

A deployment consequence follows that is easy to miss and expensive to discover. `App_Data/` is excluded by [.gitignore:32], yet all fourteen database binaries are tracked — 43,376,640 bytes across the three editions (Appendix A). So **deploying a copy of the checkout ships live database files, including all three editions' credential stores, into the deployment.** And the converse: running an application directly out of the checkout causes the engine to attach and write to those *tracked* files, which modifies the working tree. **Any run or deployment must therefore serve a copy of the application from outside the repository**, which is a build-and-deployment requirement in its own right and is independent of whether the committed binaries should have been tracked at all — that question belongs to deliverable 08.

### 11.2 Database — a trusted Windows identity, and DDL rights at startup

| Construct | Editions | Permission implied |
| --- | --- | --- |
| `Integrated Security=True` | MVC 5, both connection strings [src/MVC5/MvcMusicStore/Web.config:12-13]; MVC 4, catalog [src/MVC4/MvcMusicStore/Web.config:21] | **A Windows identity the SQL instance trusts.** No credential is presented, so the worker-process identity *is* the database principal. There is no application login to grant, and no way to scope the application's rights independently of that identity |
| `Integrated Security=SSPI` | MVC 4, credential store [src/MVC4/MvcMusicStore/Web.config:15] | The same requirement, spelled differently |
| `DropCreateDatabaseIfModelChanges<MusicStoreEntities>` | all three [src/MVC5/MvcMusicStore/Models/SampleData.cs:9], [src/MVC4/MvcMusicStore/Models/SampleData.cs:9], [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Models/SampleData.cs:9] | **Rights to drop and create the database and its schema**, held by the running application, exercised at startup. This is the widest privilege the repository demands anywhere |
| `((IObjectContextAdapter)context).ObjectContext.CreateDatabase()` | MVC 4 [src/MVC4/MvcMusicStore/Filters/InitializeSimpleMembershipAttribute.cs:37] | **`CREATE DATABASE`**, explicitly, in code, at first use |
| `WebSecurity.InitializeDatabaseConnection(…, autoCreateTables: true)` | MVC 4 [src/MVC4/MvcMusicStore/Filters/InitializeSimpleMembershipAttribute.cs:41] | **`CREATE TABLE`** in the credential store, at first use |
| Administrator provisioning at startup | MVC 5 [src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:18], [:21-44]; MVC 4 [src/MVC4/MvcMusicStore/App_Start/AppConfig.cs:18], [:21-37] | Write access to the credential store's user and role tables, again held by the running application at startup rather than by an operator |

**The single most consequential line in this section:** the application's own runtime identity requires database-owner-level rights in all three editions, and MVC 4 requires server-level `CREATE DATABASE` on top. There is no separation whatsoever between the identity that serves requests and the identity that changes schema. The target-state separation — a deployment-time migration step under a distinct principal, with the runtime identity holding least-privileged data access — is owned by deliverable 06 and is not specified here.

### 11.3 Application-pool identity — the constraint LocalDB imposes

`(LocalDb)\MSSQLLocalDB` [src/MVC5/MvcMusicStore/Web.config:12-13] and `(LocalDB)\v11.0` [src/MVC4/MvcMusicStore/Web.config:13], [:19] name **automatic LocalDB instances**, which are scoped to a Windows user profile: an instance belongs to the user who owns it, and a process running under a different identity does not see it. Combined with section 11.2's integrated authentication, this pins the requirement tightly:

**The worker-process identity must be a Windows user account that owns a LocalDB instance of the named generation and is trusted by it.** A service identity without a loaded user profile has no automatic instance to connect to. That is consistent with — and explains — the only workflow the repository documents: open the solution in Visual Studio and press F5 [src/MVC5/README.md:109-111], [src/MVC4/README.md:163-164], i.e. run under the developer's own identity via IIS Express, which the project files declare as the host [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:18], [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:18]. **No edition documents or configures a full-IIS deployment**, and the repository contains no application-pool configuration of any kind.

Two related settings complete the picture at the web tier. MVC 5 leaves `IISExpressAnonymousAuthentication` and `IISExpressWindowsAuthentication` empty [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:20-21] and sets `NTLMAuthentication` to `False` [:286], and its `system.web` authentication mode is `None` [src/MVC5/MvcMusicStore/Web.config:32] because authentication is handled by OWIN middleware instead. So **the browser-facing tier is anonymous while the data tier is Windows-authenticated**: the identity reaching SQL Server is always the application's, never the caller's. That is what makes the wide rights of section 11.2 a property of the deployment rather than of any user.

### 11.4 Build-host permissions

Smaller, but they are real prerequisites for a build to succeed:

- **Write access to the solution directory**, because MSBuild-integrated restore writes packages to `$(SolutionDir)\packages` [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:31] and every edition writes output to `bin\` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:33], [:41].
- **Package-restore consent granted on the host**, because `RequireRestoreConsent` defaults to `true` [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:13] and passes `-RequireConsent` to the restore command [:51]. A host that has not opted in fails the restore.
- **Permission to install machine-wide products** in the case of MVC 3, whose MVC framework assembly resolves only from a machine-wide install (section 7.2) and whose catalog provider is a separately installed native component (section 10.1). Machine-wide installation is an administrative operation, which makes MVC 3's build host harder to provision than the other two editions' — a fact worth carrying into any build-agent decision.

---

## 12. Deployment automation — its absence is a finding

### 12.1 What does not exist

Each absence is verified, and the command is the evidence:

```bash
git ls-files '*.pubxml' '*.pubxml.user' | wc -l          # -> 0   no publish profile
git ls-files '.github/*'                | wc -l          # -> 0   no GitHub Actions
git ls-files '*.yml' '*.yaml'           | wc -l          # -> 0   no pipeline definition of any kind
git ls-files '*Dockerfile*' '*docker-compose*' | wc -l   # -> 0   no container manifest
git ls-files '*.tf' '*.bicep'           | wc -l          # -> 0   no infrastructure-as-code
git ls-files '*.sh' '*.ps1' '*.cmd' '*.bat' '*.psm1'     # -> 6, and see below
```

**The repository has no publish profile, no pipeline definition, no container manifest and no infrastructure definition.** That is a finding rather than a gap in this assessment's coverage: the entire path from a built assembly to a running deployment is undocumented and unautomated, and exists today only as the manual Visual Studio workflow the READMEs describe [src/MVC5/README.md:109-111], [src/MVC4/README.md:163-164].

Two qualifications, because a bare "zero" would be wrong in one case and incomplete in the other:

- **The six tracked script files are not build scripts.** All six are NuGet package install/uninstall scripts inside the committed MVC 4 payload — `packages/EntityFramework.5.0.0/tools/{EntityFramework.psm1, init.ps1, install.ps1}` and `packages/jQuery.1.7.1.1/Tools/{common.ps1, install.ps1, uninstall.ps1}`. **There is no standalone shell or PowerShell build script anywhere in the repository**, which is the claim that matters.
- **A publish profile would have been untracked even if one existed**, because `PublishProfiles/` is excluded by [.gitignore:18]. The absence of the file is therefore weaker evidence than it looks, and this document states it as: *no publish profile is tracked, and the repository's own ignore rules would have excluded one.* What is unambiguous is that no publish configuration is available to a reader of the repository, which is the operative consequence either way.

### 12.2 The build logic that does exist, and what it does

Treating the automation as wholly absent would be the wrong reading. Build logic exists; it just lives inside the MSBuild project files and one committed targets file, which is why this section analyses it rather than reporting a void.

| Location | Logic | Status |
| --- | --- | --- |
| `.nuget/NuGet.targets` — `RestorePackages` target [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:76-83] | Executes `$(RestoreCommand)` [:53] under two OS-conditioned `Exec` tasks, one of which logs standard error as error [:80-82] | **Live for MVC 4** via `<RestorePackages>true</RestorePackages>` [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:24] and the `BuildDependsOn` prepend [.../NuGet.targets:57-60]. **Inert for MVC 5**, whose import is guarded (section 5.2) |
| `.nuget/NuGet.targets` — `CheckPrerequisites` target [:69-74] | Errors if the committed client is missing [:71]; sets `VisualStudioVersion` in the environment [:72]; would download the client [:73] | Live for MVC 4. The download branch is **switched off** [:16], so the committed binary is mandatory |
| `.nuget/NuGet.targets` — `BuildPackage` target [:85 onward] | `nuget pack` of the project [:54], appended to `BuildDependsOn` when enabled [:63-66] | **Off** — `BuildPackage` defaults to `false` [:10]. A packaging capability exists and is unused |
| `.nuget/NuGet.targets` — the non-Windows property branch [:34-39], [:47] | Invokes the client through `mono --runtime=v4.0.30319` | Present in the committed file. Notable only because it is the repository's sole acknowledgement that a non-Windows build host might exist |
| The `MvcBuildViews` target, in all three project files | Runs `AspNetCompiler` over the web output after build [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:274-276], [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:339-341], [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:216-218] | **Off in all three** — see section 12.3 |
| The `WebApplication.targets` imports | Supply the web publish targets | Conditioned in MVC 5 and MVC 4, unconditional in MVC 3 (section 6.5) |

### 12.3 Views are never compile-checked, in any edition

`MvcBuildViews` is set to `false` in all three editions — [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:17], [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:17], [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:16] — while the target that would act on it is gated on the value being `'true'` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:274], [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:339], [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:216]. The target therefore never runs and `AspNetCompiler` is never invoked.

Two consequences, both squarely build-and-deployment:

1. **A successful build says nothing about whether the Razor views compile.** Every view error is deferred to first request in the deployed application, per view. A build that reports zero errors can still deploy a page that throws on load.
2. **The views carry no build-time guarantee to migrate against.** Deliverable 01 §2.2 counts 83 Razor views repository-wide; not one of them has ever been compile-checked by this build. Combined with the absence of any test (section 9), the repository offers **no automated signal at all** about view correctness, before or after a port.

Compiler diagnostics for C# are not absent — every project sets `WarningLevel` `4` (section 4.3) — but warnings are not errors and there is no pipeline to enforce anything. The finding is the absence of *enforcement*, not of diagnostics; deliverable 08 owns its severity.

---

## 13. Consolidated requirements and open verification items

### 13.1 Per-edition summary

Every row restates a fact established earlier and points at the section that carries its evidence and its full citations. No locator is abbreviated here; where a row needs one, it is already stated in full at its point of first claim above.

| | **MVC 5** | **MVC 4** | **MVC 3** |
| --- | --- | --- | --- |
| **Build status** | **VERIFIED** once restored — Debug and Release, exit `0`, 0 warnings (§3.1, §5.4); **cannot build from a clean checkout**, which is unchanged (§5.1 to §5.3) | **FAILS as committed**, both solutions, before compilation (§6); builds only under the host-side overrides of §6.3 | **BUILDS** — exit `0`, 0 warnings on a host carrying its machine-wide prerequisites (§3.1); three unresolved-reference warnings under Mono; toolchain-dependent (§7) |
| Target framework | `v4.8` (§4.1) | `v4.5` (§4.1) | `v4.0` (§4.1) |
| Solution to build | `src/MVC5/MvcMusicStore.sln` | `src/MVC4/MvcMusicStore.sln` — **not** `src/MVC4/MvcMusicStore/MvcMusicStore.sln` (§6.4) | `src/MVC3/MvcMusicStore-Completed/MvcMusicStore.sln` |
| Restore required before build | **Yes, mandatory** — nothing committed (§8) | Yes as committed, and it cannot succeed (§6.3) | No, for its single hinted reference (§7.1) |
| Machine-wide product install | none | none | **ASP.NET MVC 3 Tools Update** + **SQL Server Compact 4.0** (§7.2, §10.1) |
| Declared web host | IIS Express, plain HTTP (§4.3) | IIS Express, plain HTTP (§4.3) | **VS Development Server — no longer ships** (§4.3, §7.3) |
| Catalog store | LocalDB `MSSQLLocalDB`, file-attached | LocalDB `(LocalDB)\v11.0` as committed — unavailable under its own documented toolchain (§10.3) | **SQL Server Compact 4.0**, `.sdf` created on first run, none committed |
| Credential store | LocalDB, same instance, separate Identity 1.0 database | LocalDB, same instance, separate SimpleMembership database | **A SQL Server instance**, inherited from machine configuration — *unverified* (§10.2) |
| Schema script shipped | **none** | two, byte-identical, **neither runnable** (§10.5) | one, runnable given a database named `MvcMusicStore` (§10.5) |
| Administrator provisioned at startup | yes, `async void` — failures unobservable (§10.4) | yes, failures surfaced as `InvalidOperationException` (§10.4) | **no** — role never created (§10.4) |
| Runtime database rights required | drop/create schema (§11.2) | drop/create schema **plus `CREATE DATABASE`** (§11.2) | create-file in `App_Data`, plus whatever the machine-level provider requires |
| Tests | none | none | none |
| Deployment automation | none | none | none |
| Views compile-checked | no | no | no |

### 13.2 Open verification items

Three items could not be answered from repository evidence alone, because answering them would have meant asserting something unobserved. **Two have since been discharged by the Windows verification run of §3.1 and are recorded closed with their evidence; one remains open.** Each row names what was required and where it now stands.

| # | Open item | What must be done | Carried by |
| --- | --- | --- | --- |
| 1 | ~~**MVC 5's application build is unverified**~~ — **CLOSED** | The Windows verification run specified in §5.4 was performed, with all nine fields recorded: restore and build exit `0`, Debug and Release, 0 warnings, 0 errors | Discharged in §5.4. What remains is not the build but the **restore precondition** of §5.1 to §5.3, which deliverable 07 carries as the narrowed residual risk |
| 2 | **MVC 3's credential store is inherited, not declared** | Verify the actual machine-level ASP.NET SQL provider and its connection string on the supported Windows runtime (§10.2) before the requirement is stated as final | This document, on re-verification; deliverable 07 if it affects scope |
| 3 | **The effective package source is a property of the build host, not the repository** — **RECORDED for item 1** | The source used by the run that discharged item 1 is recorded in §3.1 and §5.4: `nuget.org`, `https://api.nuget.org/v3/index.json`, inherited from host configuration | Deliverable 02 §6 owns the finding, which is unchanged: the *repository* still declares no source, so the next host may resolve differently. Recording it per run is the mitigation, not a fix |

Items 1 and 3 were one run, now performed. **Item 2 is the one still outstanding**: it needs a running MVC 3 rather than a build, and it can be discharged on the same class of host whenever one is next available.

### 13.3 Where each consequence is owned

| Fact recorded here | Consequence owned by |
| --- | --- |
| MVC 5's verified build, and its clean-checkout restore precondition | 07 (the narrowed residual risk), 03 (first workstream gate) |
| MVC 4's two configuration defects and the stale solution | 08 (debt severity and owner), 12 (blocker classification) |
| MVC 3's machine-wide product requirements and retired provider | 12 (no-successor classification), 02 §4.1 (dependency inventory) |
| Windows, Visual Studio and LocalDB coupling | 11 (cloud readiness), 06 (hosting target) |
| The destructive initializer's deployment requirement | 08 (debt framing), 05 (replacement schema lifecycle), 06 (deployment-time migration sequence) |
| Runtime identity holding DDL rights | 06 (principal separation), 09 (security posture) |
| The absent regression baseline | 03 (test authoring precedes the port), 05 (suite design), 07 (first-order risk) |
| Absent deployment automation | 03 (CI/CD as a net-new workstream), 06 (deployment model) |
| The unconfigured restore source | 02 §6 (the finding and the §3.3 correction), 04 (target-state remedy) |
| Committed database binaries and `packages/` payloads | 08 (repository hygiene) |

---

## Appendix A — Reproducibility

Every repository-wide figure in this document, with the command that reproduces it. Commands are given in POSIX form; on a Windows host, `git ls-files` and `git grep` behave identically and the pipeline utilities are available through Git for Windows.

| Claim | Value | Command |
| --- | --- | --- |
| Solution files (§6.4, §13.1) | **4**, for 3 projects | `git ls-files '*.sln'` |
| MVC 4 `..\packages` hint paths (§6.2) | **24**, being every hint path in the project | `grep -c 'HintPath>\.\.\\packages' src/MVC4/MvcMusicStore/MvcMusicStore.csproj` and `grep -c '<HintPath>' …` |
| MVC 5 hint paths (§5.1) | **26** | `grep -c 'HintPath>\.\.\\packages' src/MVC5/MvcMusicStore/MvcMusicStore.csproj` |
| MVC 3 hint paths (§7.1) | **1** | `grep -c '<HintPath>' src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj` |
| Absent `src/MVC4/.nuget` (§6.1) | absent | `test -d src/MVC4/.nuget && echo present \|\| echo absent` |
| Absent `src/MVC4/packages` (§6.2) | absent | `test -d src/MVC4/packages && echo present \|\| echo absent` |
| Absent `src/MVC5/packages` (§5.2) | absent | `test -d src/MVC5/packages && echo present \|\| echo absent` |
| Absent `src/MVC5/MvcMusicStore/.nuget` (§5.2) | absent | `test -d src/MVC5/MvcMusicStore/.nuget && echo present \|\| echo absent` |
| Absent stale-solution project path (§6.4) | absent | `test -f src/MVC4/MvcMusicStore/MvcMusicStore/MvcMusicStore.csproj && echo present \|\| echo absent` |
| Tracked `.nuget` files (§5.2, §6.1) | **3**, all under `src/MVC4/MvcMusicStore/.nuget/` | `git ls-files \| grep '\.nuget/'` |
| Tracked `packages/` files (§8) | **215** = 169 MVC 4 + 46 MVC 3 + 0 MVC 5 | `git ls-files \| grep -c '/packages/'`; then `git ls-files 'src/MVC4/MvcMusicStore/packages/*' \| wc -l`, `git ls-files 'src/MVC3/MvcMusicStore-Completed/packages/*' \| wc -l`, `git ls-files 'src/MVC5/*packages/*' \| wc -l` |
| Lockfiles (§8) | **0** | `git ls-files '*packages.lock.json' \| wc -l` |
| Test artefacts (§9) | **0** | `git grep -lIiE 'TestClass\|\[Fact\]\|\[TestMethod\]\|xunit\|nunit\|Microsoft\.VisualStudio\.TestTools' -- 'src/' \| grep -v '/packages/' \| wc -l` and `git ls-files \| grep -i test \| wc -l` |
| Database binaries (§11.1) | **14** files, **43,376,640** bytes | `git ls-files \| grep -icE '\.(mdf\|ldf)$'`; total with `git ls-files \| grep -iE '\.(mdf\|ldf)$' \| xargs -d '\n' stat -c %s \| awk '{t+=$1} END {print t}'` |
| MVC 3-Completed `App_Data` files (§10.1) | **0** | `git ls-files 'src/MVC3/MvcMusicStore-Completed/*' \| grep App_Data \| wc -l` |
| `ASPNETDB` references (§10.1) | **0** | `git grep -il 'ASPNETDB' -- 'src/' \| wc -l` |
| MVC 3 membership/provider declarations (§10.1, §10.2) | only `<roleManager enabled="true" />` | `git grep -n -iE '<membership\|<roleManager\|LocalSqlServer\|<profile' -- 'src/MVC3/MvcMusicStore-Completed/*'` |
| Schema scripts (§10.5) | **3** | `git ls-files '*.sql'` |
| The two MVC 4 scripts are byte-identical (§10.5) | same SHA-256 `D577AAA5…182C9D`, both 153,594 bytes | `sha256sum src/MVC4/MvcMusicStore-Create.sql src/MVC4/MvcMusicStore/MvcMusicStore-Create.sql` |
| All three scripts are UTF-16 with a BOM (§10.5) | each begins `FF FE`; the MVC 3 asset script begins `FF FE FF FE` | `for f in $(git ls-files '*.sql'); do printf '%s ' "$f"; head -c 4 "$f" \| xxd -p; done` |
| Seed rows, MVC 5 and MVC 4 (§10.4) | **15** genres, **303** artists, **462** albums | `grep -c 'new Genre' src/MVC5/MvcMusicStore/Models/SampleData.cs` and the `new Artist` / `new Album` equivalents |
| Seed rows, MVC 3 (§10.4) | **10** genres, **149** artists, **246** albums | the same three commands against `src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Models/SampleData.cs` |
| Seed file size (§10.4) | **826** physical lines (MVC 5, MVC 4); **430** (MVC 3) | `wc -l < src/MVC5/MvcMusicStore/Models/SampleData.cs` — the physical-line metric of deliverable 01 §2.4, not the non-blank metric |
| Initializer registration sites (§10.4) | **5** occurrences, of which **4 register `SampleData`** — MVC 5 twice, MVC 4 once, MVC 3 once. The fifth *disables* the initializer for `UsersContext` in MVC 4 | `git grep -n 'SetInitializer' -- 'src/MVC5' 'src/MVC4' 'src/MVC3' \| grep -v '/packages/'` |
| Publish profiles (§12.1) | **0** | `git ls-files '*.pubxml' '*.pubxml.user' \| wc -l` |
| Pipeline definitions (§12.1) | **0** | `git ls-files '.github/*' '*.yml' '*.yaml' \| wc -l` |
| Container manifests (§12.1) | **0** | `git ls-files '*Dockerfile*' '*docker-compose*' \| wc -l` |
| Infrastructure definitions (§12.1) | **0** | `git ls-files '*.tf' '*.bicep' \| wc -l` |
| Tracked script files (§12.1) | **6**, all inside the committed MVC 4 package payload | `git ls-files '*.sh' '*.ps1' '*.cmd' '*.bat' '*.psm1'` |

| The Windows verification run (§3.1, §5.4) | restore and build exit `0` for all three editions, Debug and Release, 0 warnings | `nuget restore <solution> -NonInteractive`, then `MSBuild <solution> /t:Rebuild /p:Configuration=Debug` and again with `Release`, MSBuild located via `vswhere -latest -requires Microsoft.Component.MSBuild -find 'MSBuild\**\Bin\MSBuild.exe'`. MVC 4 additionally requires `/p:SolutionDir=<repo>\src\MVC4\MvcMusicStore\` — unquoted, trailing backslash — and `/p:RestorePackages=false` (§6.3) |
| MVC 4 as committed (§6.1) | exit `1`, `MSB4019` at `MvcMusicStore.csproj(360,3)` | the same MSBuild command on `src/MVC4/MvcMusicStore.sln` with no property overrides |
| The stale MVC 4 solution (§6.4) | exit `1`, `MSB3202` | the same command on `src/MVC4/MvcMusicStore/MvcMusicStore.sln` |
| Restore advisories (§3.1) | **15** for MVC 5, **14** each for MVC 4 and MVC 3; exit `0` regardless | count `NU19` lines in the restore output. Severities and pins are owned by deliverable 02 §8.2 |

**Repository state.** No repository file was modified in producing this document; `git status --porcelain` shows only new files under `docs/modernization/`. The verification run of §3.1 wrote only `packages/`, `bin/` and `obj/`, every one of which the repository already ignores, and supplied its MVC 4 overrides on the command line rather than by editing a tracked file.
