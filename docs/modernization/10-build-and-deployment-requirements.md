
# 10 — Build and Deployment Requirements (as-is)

**Deliverable 10 of 13** · MvcMusicStore modernization assessment · assessment record

This document records what each of the three shipped editions requires in order to **build**, what it requires in order to **run**, and what was actually observed when the prescribed build procedure was attempted. It is the assessment's single source of truth for per-edition build outcomes.

---

## 1. Purpose, ownership and authoring contract

### 1.1 What this document is

Four things, and nothing beyond them:

1. **The build evidence** — what was run, on what host, against what restore state, and what happened, in two clearly separated records: the Mono record of §3.1 and the single post-freeze Windows observation of §3.2. Section 3 carries both; sections 5 to 7 carry the per-edition detail, and §1.2 carries the status the document actually asserts.
2. **The toolchain and hosting prerequisites each edition demands**, derived from the repository's own project files, solution files and configuration rather than from any external statement. Section 4.
3. **The database components each edition needs in order to run**, stated per edition with the catalog store and the credential store separated, because in two editions they are not the same engine. Section 10.
4. **The permissions and the deployment-automation posture** the repository implies. Sections 11 and 12.

### 1.2 What this document owns, and what it must not restate

This deliverable is the **single owner of per-edition build outcomes**. Deliverables 07 and 12 cite this document for them rather than re-deriving them, so a statement here is what the whole assessment believes. One consequence is stated up front because it is the most important sentence in the document: **the build of the sole migration source, MVC 5, is carried as *blocked pending a Windows verification run*.** That is the status the frozen Agent Action Plan fixes — §0.6.1 marks the MVC 5 build assessment *incomplete pending a Windows run*, and §0.11.2's acceptance row for this deliverable requires the status to be carried as *blocked pending a Windows run*, **not** as verified — and it is the status every downstream deliverable cites.

Two facts sit beneath it and neither changes it.

- **The repository-side finding stands on its own, independent of any host:** MVC 5 **cannot** be built from a clean checkout, because it commits no package payload (§5.1 to §5.3).
- **A Windows host carrying the prescribed toolchain later became available, and the prescribed restore and build were executed on it.** That observation is recorded in full in **§3.2, which is the only place in this document that carries it**, as **supplementary evidence**. It is recorded because suppressing an observation would be worse than recording one that does not yet count — but it is **not authoritative over the carried status**. The AAP is the frozen, agreed-upon basis of this assessment; a gate it sets is discharged by an approved revision of it, not by the run that satisfies its precondition. Until that revision exists, this deliverable's status is the blocked one, deliverable 07 retains the corresponding risk, and §13.2 item 1 remains **open**.

Seven decisions are owned elsewhere, and this document cross-references rather than restating them. A restatement in different words would read downstream as a second decision.

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

Markdown line length follows the corpus convention recorded in [`README.md` § 10 Markdown line-length convention](README.md#10-markdown-line-length-convention).

One further convention: **committed credential values are cited by locator and never reproduced.** The repository ships a plaintext administrator password in `appSettings` in two editions [src/MVC5/MvcMusicStore/Web.config:17], [src/MVC4/MvcMusicStore/Web.config:26], documented again in prose at [src/MVC5/README.md:79]. Its value appears nowhere in this document. The security finding belongs to [09 — Security Assessment](09-security-assessment.md); what belongs here is the deployment consequence, recorded in section 10.4.

### 1.4 The no-modification constraint

The user directed that no code changes be made before the assessment is approved, and the project's environment setup instructions restate the same gate independently. **No tracked repository file was modified, added or deleted in producing this document.** What the assessment nonetheless *wrote* into the checkout is generated content, and it is reported rather than folded into a clean-tree claim: **restore and build runs were performed against this checkout, and they left eight ignored trees behind** — a `bin` and an `obj` under each of the three editions' projects, plus a restored `packages` tree under `src/MVC4` and another under `src/MVC5` — **527 files and 114,310,394 bytes, as measured.** Their removal is deliberately **not** recorded here as a discharged, one-time act: §3.2's post-freeze run was performed in place as well and writes the same set again, so removing them and verifying their absence with the ignored-aware check below is a **standing requirement on whoever commits this assessment** rather than an observation this document can make on its own behalf. Appendix A carries the per-tree record, the rule and the commands.

Those runs are **not** the Mono attempts of §3.1: the evidence host of §2.1 had no NuGet client and could perform no restore at all. Nor is any of those trees offered as build evidence here. **This document carries exactly two build records** — §3.1's Mono record, supplemental under the limits of §2.2, and §3.2's post-freeze Windows observation, each stating its own host, tool inventory, commands and outcomes — and **the carried status is §1.2's**. Neither a restore that ran nor an output directory that existed may be read as a verified application build, and no build outcome may be read off a tree in the working directory.

**The acceptance evidence is four checks, and they only mean anything together.** Bare porcelain and the tracked diff are what a repository-state claim is usually rested on, and by themselves they are not merely thin but actively misleading here: all eight trees matched an ignore rule on this checkout — `[Oo]bj/` [.gitignore:1] and `[Bb]in/` [.gitignore:2] over the six build-output trees, and `Packages/` [.gitignore:33] over the two restored `packages` trees — so **bare porcelain and a tracked-file diff both report a clean tree while generated payload sits in it**, which is precisely what they did for the whole period the eight trees existed. That attribution is neither the obvious one nor host-independent: `packages/*` [.gitignore:15] does **not** cover a nested `packages` directory, and the rule that does is case-dependent, so on a case-sensitive host the two restored trees would have been ignored by nothing at all. Appendix A records the attribution per tree, its probe and that portability consequence. Only an ignored-aware check can see that payload on the host where these runs happened, which is why the last two lines below are not optional:

```bash
git status --porcelain                                                  # must report 0 lines
git status --porcelain --ignored                                        # must report 0 lines: no ignored payload or output tree left behind
git clean -ndX                                                          # must report nothing: nothing ignored is left to remove
git diff --name-status ea2552d6eda7c20e9477a512e5c615665618cf35..HEAD   # must report exactly 13 'A' rows, all under docs/modernization/
```

**Those four are stated as the results the check must produce, not as results this document observed.** The distinction is the same one §3.2 draws about the build: a required outcome and a recorded observation are different claims, and only the second may be reported as evidence. The fourth line is the one whose result is settled independently of any run — it is a property of the commit range, recorded in Appendix A — while the first three are properties of the working tree at the moment of commit and are the committer's to verify.
`ea2552d6eda7c20e9477a512e5c615665618cf35` — `ea2552d` in short form below — is the **immutable evidence revision** every as-is claim in this document was read from; the right-hand side is **the delivery commit the reviewer has checked out**, which is what `HEAD` names when the check is run, and no hash is invented for it here **because a document cannot cite the commit that creates it**: the commit carrying these thirteen files does not exist while they are being written, so a literal hash in its place would be either wrong on the day or a value nobody could have obtained. Appendix A carries the same check as a row.

**The standing rule that follows is this deliverable's own, because build hygiene is its subject.** A restore or a build run for assessment purposes belongs in a **disposable clone outside the authoritative checkout**, which cannot leave payload in the tree the deliverables are committed from. Where one is run in place instead, its generated trees are **removed and their absence verified with the ignored-aware check above** — the `--ignored` and `git clean -ndX` lines specifically, since the other two are blind to them. This repository makes that more than housekeeping: MVC 4's hint paths resolve to `src/MVC4/packages`, **not** to the tracked payload at `src/MVC4/MvcMusicStore/packages` (§6.2), so restoring the MVC 4 solution writes a **second, untracked package tree beside the committed one**. A restore here produces generated payload even for the editions whose packages are committed, which is exactly why the check has to be ignored-aware and not merely careful.

Every defect recorded below is documented and none is repaired.

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
| 4 — Stand up the required SQL Server database components | **Impossible on this environment** | Neither LocalDB, nor SQL Server, nor SQL Server Compact could be installed. *Identifying* the components is a separate matter and was done — section 10, with one component identified only as far as the repository can settle it and left open in §13.2 item 2: MVC 3's credential store, which the application inherits from its host rather than declaring (§10.2) |
| Determine the target framework per edition | **Satisfied** | Read directly from the project files: section 4.1 |
| 5 — Run unit tests if present | **Satisfied vacuously** | No test of any kind exists anywhere in the repository. Section 9 gives the verifying command |
| 3 — Resolve missing package dependencies | **Partially satisfied — by analysis, not by restore** | The missing dependencies were identified per edition (sections 5, 6, 8), but nothing could be resolved by running a restore |

**The tally, stated plainly: four items impossible on this environment, two satisfied — one of them vacuously — and one partially satisfied by analysis rather than by execution.** The fourth item is the one most easily mis-stated, so it is split deliberately: *standing up* the database components was impossible, while *identifying* them is an analysis task that the repository supports and that section 10 discharges for every component the repository declares. **One component it cannot discharge is named rather than absorbed:** MVC 3's credential store is inherited from the host's machine-level ASP.NET configuration and is declared nowhere in the repository, so section 10 states the negative as final and leaves the positive requirement **expressly unresolved**, with the verification that would close it specified in §10.2 and carried as §13.2 item 2.

### 2.1.1 The prescribed toolchain later became available, and the prescribed steps were then executed — as supplementary evidence

The table above is the accurate record **for the Linux evidence host**, and nothing below retracts it. It is not the last word: a **Windows host carrying the prescribed toolchain** subsequently became available, and prescribed items 1, 2 and 3 were executed on it against this same checkout, **for the migration source**. That run is the post-freeze Windows observation recorded in §3.2, which is where its host, tool inventory, commands and per-edition results are set out once, together with the limits on reading it; this section states only what that run did with the prescribed items.

**What that run is, and what it is not, stated before this table so no row is over-read.** It is a recorded observation on one host, and it is **supplementary**: it does not change this deliverable's carried build status, which §1.2 fixes as *blocked pending a Windows verification run* on the authority of the frozen AAP §0.6.1 and §0.11.2. Every "Satisfied" below is a statement about **what that host did with a prescribed step**, not a statement that the AAP's verification gate is discharged. **The run's results, its tool inventory, its commands and its qualifications are recorded once, in §3.2**, and every Evidence cell below points there rather than restating them — and never at §3.1, which is the Linux/Mono record and has nothing to do with this host.

| Prescribed item | Status on the Windows host | Evidence |
| --- | --- | --- |
| Install the required Windows toolchain | **Satisfied** | The host and tool inventory recorded in §3.2 |
| 1 — Restore NuGet packages | **Satisfied** | Executed for all three solutions; the per-edition restore results are in §3.2 |
| 2 — Build the solution file | **Satisfied, in Debug only** | Executed for all three solutions in the Debug configuration; the per-edition build results, and the two properties MVC 4 required, are in §3.2. **Release was not built**, so §4.4's publish path remains unexercised |
| 3 — Resolve missing package dependencies | **Satisfied by restore — and for MVC 4 that is not the whole story** | The restore creates MVC 5's absent payload (§5.2). For MVC 4 it also creates `src/MVC4/packages`, which is where the 24 project-relative hint paths point (§6.2), so those references resolved **for that run**; what the host-side properties worked around is the misplaced import, and both committed defects remain unrepaired, so as committed — no prior restore, no properties — the build still fails (§3.2 qualification 1, §6.1, §6.3) |
| 4 — Stand up the required SQL Server database components | **Not satisfied, and not claimed** | No database engine was stood up: the host carries no SQL Server or LocalDB instance, which is also why §3.2's runtime observations read as they do. Section 10 *identifies* the components; standing them up is outside this record, and §13.2 item 2 stays open |
| 5 — Run unit tests if present | **Satisfied vacuously, and confirmed** | No test of any kind exists. Section 9, and the "tests run" field of §3.2 |

Three boundaries on that run, all deliberate. It establishes what it observed and no more — a Debug build result on one Windows host with one recorded tool inventory, which is why §3.2 records the versions and the restore source and not merely the outcome. It does not license reading the Mono results differently: §2.2's limits still bind every conclusion drawn from them. And it does not retire the AAP-frozen status of §1.2, nor the risk deliverable 07 carries against it, nor open verification item 1 of §13.2.

### 2.2 What a Mono `xbuild` invocation establishes — and what it cannot

**A Mono `xbuild` invocation is not equivalent to the prescribed toolchain, and nothing in this document treats it as one.** Stated precisely:

**What it can establish.** Whether a project's committed MSBuild configuration is *internally coherent* — whether its imports resolve, whether its property and item definitions evaluate, whether the paths it declares exist relative to the directory it is evaluated from. And, if evaluation succeeds, whether its compilable source is *syntactically sound*.

**What it cannot establish.** That a project builds on the toolchain the environment prescribes. It cannot substitute for MSBuild 17.x semantics, for the .NET Framework 4.8 reference assemblies, for the Visual Studio web-application targets, or for a NuGet client that current NuGet supports. A Mono success is not a Windows success, and a Mono failure is not automatically a Windows failure either — each observed failure below is separately classified as platform-independent or environment-specific, and that classification is the analytical work, not the invocation.

**The Mono results in section 3 are therefore retained as supplemental portability evidence only.** They are useful for exactly one thing: the *configuration* defects they expose are platform-independent, because a missing directory and an unconditional import to a non-existent file fail identically under any MSBuild implementation. Section 6 relies on that property and says so; nothing else in this document rests on the Mono results.

### 2.3 The toolchain gap is itself an assessment finding

The gap is recorded as a finding rather than worked around, because **a build that requires Windows, a specific Visual Studio generation and a machine-wide product install is itself a portability and cloud-readiness result.** Every one of the following is a repository-declared, not externally asserted, requirement, and each is evidenced in section 4:

- Two editions import Visual Studio web-application targets, one of them unconditionally and at a Visual Studio 2010-era path [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:209].
- One edition's MVC framework assembly resolves only from a machine-wide install of an out-of-support product [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:42].
- All three editions' data access is a Windows-only local database engine reached with Windows integrated authentication [src/MVC5/MvcMusicStore/Web.config:12-13], [src/MVC4/MvcMusicStore/Web.config:13-15], [src/MVC4/MvcMusicStore/Web.config:19-21], or a retired Windows-only embedded engine [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:55-59].
- Two editions declare IIS Express as their web host [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:18], [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:18]; the third declares a Visual Studio development server that no longer ships [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:17], [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:223].

The cloud-readiness consequences of those facts are owned by [11 — Cloud Readiness Assessment](11-cloud-readiness-assessment.md); the *build and hosting requirement* each one creates is owned here.

---

## 3. The build evidence table

This is the document's centrepiece, and it carries **two records and no more**, kept apart on purpose because they were made on different hosts under different authority:

- **§3.1 — the Mono `xbuild` record**, made on the Linux evidence host the assessment was written on, retained as **supplemental portability evidence only** under the limits of §2.2.
- **§3.2 — one post-freeze Windows observation**, made after the governing plan was frozen, on a host carrying the prescribed toolchain. It is recorded in full and it is **supplementary**: it does not amend AAP §0.6.1 and does not change this deliverable's carried status.

**The carried status is §1.2's and is stated there once:** MVC 5's build is **blocked pending a Windows verification run**. Neither record below alters it — §3.1 because a Mono invocation is not the prescribed toolchain (§2.2), §3.2 because a gate the frozen plan sets is discharged by an approved revision of that plan, not by the run that satisfies its precondition (§1.2, §5.4). Every outcome in both records is described in the terms that observation supports and no stronger.

### 3.1 The Mono record, exactly as observed — supplemental portability evidence only

Host for all three attempts: Linux, Mono 6.8.0.105, which provides `xbuild` (XBuild Engine 14.0) and neither `msbuild` nor a NuGet client (§2.1). The three outcomes are the strand C rows of §3 and are not repeated here; what this sub-section adds is the boundary around them.

**A Mono `xbuild` invocation is not equivalent to the prescribed Windows and Visual Studio toolchain** and is not treated as one anywhere in this document; §2.2 states exactly what it can and cannot establish. Two consequences follow, and together they are the whole of what strand C contributes:

- **Only the configuration defects generalise.** MVC 4's two failures are platform-independent, because a missing directory and an unconditional import to a non-existent file fail identically under any MSBuild implementation (§2.2, section 6). Section 6 relies on that property and says so.
- **Nothing else in this document rests on strand C**, and **no strand C row may be read as a statement about a build on the prescribed toolchain** — MVC 3's least of all, since its Mono success came from Mono's own GAC and targets and is not evidence about any Windows host's product inventory (§7.1, §7.2).

### 3.2 The post-freeze Windows observation — one record, and it does not amend the frozen status

**Read the label before the results.** The prescribed toolchain was unobtainable on the evidence host the assessment was written on (§2.1). A Windows host carrying it became available **after the governing plan was frozen**, and the prescribed restore and build were then executed on it against this same checkout. What follows is that observation, recorded here and nowhere else in this document.

Three things it is not, stated first so no row below is over-read:

- It is **not an amendment to AAP §0.6.1**, which marks the MVC 5 build assessment incomplete pending a Windows run, nor to §0.11.2's acceptance row requiring the status to be carried as blocked.
- It does **not** convert *blocked* into *verified*. §1.2 gives the reason: a gate the frozen plan sets is discharged by an approved revision of that plan, not by the run that satisfies its precondition. The carried status, deliverable 07's corresponding risk and §13.2 item 1 all stand.
- It is **not** the Mono record of §3.1, and it does not license reading that record differently — §2.2's limits still bind every conclusion drawn from it.

What it **is**: evidence the plan's owner may use when they decide whether to close the gate, recorded because suppressing an observation would be worse than recording one that does not yet count.

**Host and tool inventory.**

| Field | Value |
| --- | --- |
| Host | Windows Server 2022 |
| MSBuild | `17.14.51.32402` (Visual Studio 2022 Build Tools), located through `vswhere` rather than hardcoded — resolved to `C:\BuildTools\MSBuild\Current\Bin\MSBuild.exe` and invoked by full path, not from `PATH`, for the `$(VSToolsPath)` reason in §4.2 |
| NuGet client | NuGet CLI `6.11.1.2` |
| Targeting packs | The .NET Framework reference assemblies present on disk under `C:\Program Files (x86)\Reference Assemblies\Microsoft\Framework\.NETFramework`, including **`v4.8`** — the target framework MVC 5 declares [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:16] — as well as `v4.5` and `v4.0`, which MVC 4 and MVC 3 declare (§4.1) |
| Restore source | `nuget.org`, `https://api.nuget.org/v3/index.json` — the only source registered on the host, inherited from host configuration because the repository declares none. Deliverable 02 §6's finding is unchanged; this records which source *this* result rests on |
| Environment | `MSBUILDDISABLENODEREUSE=1` |
| Configuration built | **Debug only**, `/t:Build`. Release was not built, which matters: §4.4's one active transform sits on the Release publish, so the publish path remains unexercised |
| Tests run | **none** — none exists anywhere in the repository (section 9). The field is filled, not omitted |

**The commands, per edition, run from the repository root.**

```powershell
nuget restore src\MVC5\MvcMusicStore.sln -NonInteractive
MSBuild.exe src\MVC5\MvcMusicStore.sln /t:Build /p:Configuration=Debug /nologo /v:m /nr:false

nuget restore src\MVC4\MvcMusicStore.sln -NonInteractive
MSBuild.exe src\MVC4\MvcMusicStore.sln /t:Build /p:Configuration=Debug /nologo /v:m /nr:false /p:SolutionDir=<abs>\src\MVC4\MvcMusicStore\ /p:RestorePackages=false

nuget restore src\MVC3\MvcMusicStore-Completed\MvcMusicStore.sln -NonInteractive
MSBuild.exe src\MVC3\MvcMusicStore-Completed\MvcMusicStore.sln /t:Build /p:Configuration=Debug /nologo /v:m /nr:false
```

**What was observed. All three restores exit `0` and all three builds exit `0`, with 0 warnings and 0 errors.**

| Edition | Solution driven | Restore | Build, Debug | Assembly produced |
| --- | --- | --- | --- | --- |
| **MVC 5** | `src\MVC5\MvcMusicStore.sln` | exit `0`; **28 packages** installed, creating the payload the 26 hint paths need (§5.1, §5.2) | exit `0`, **0 warnings, 0 errors** | `src\MVC5\MvcMusicStore\bin\MvcMusicStore.dll`, **214,528 bytes** |
| **MVC 4** | `src\MVC4\MvcMusicStore.sln`, **plus the two host-side properties above** | exit `0`; **29 packages** installed | exit `0`, **0 warnings, 0 errors** — **only with those two properties** (first qualification below) | `src\MVC4\MvcMusicStore\bin\MvcMusicStore.dll`, **209,920 bytes** |
| **MVC 3** | `src\MVC3\MvcMusicStore-Completed\MvcMusicStore.sln` | exit `0` | exit `0`, **0 warnings, 0 errors** | `src\MVC3\MvcMusicStore-Completed\MvcMusicStore\bin\MvcMusicStore.dll`, **119,808 bytes** |

**Restore emitted advisory warnings and still exited `0`, which is why the warnings are recorded as a property of the run rather than folded into its outcome.** They are output of this run and not an advisory assessment: the pins, their version-risk posture, and the reason no advisory identifier is asserted anywhere in this corpus, belong to [02 — Dependency Inventory](02-dependency-inventory.md) §8.2, and nothing here revises it.

**Repository state.** `git status --porcelain` was **empty before and after** the run: `packages/`, `bin/` and `obj/` are ignored on this checkout (§1.4), so no tracked file changed and no tracked file was needed to obtain any result above. The run was performed **in place** rather than in a disposable clone, so its restore payload and its build output are ignored trees in the working tree, and the hygiene rule of §1.4 and Appendix A binds it exactly as it binds any other in-place run: those trees are removed and their absence verified with the **ignored-aware** commands — `git status --porcelain --ignored` and `git clean -ndX` — before this assessment is committed. Bare porcelain cannot see them, which is the whole reason the rule names the other two.

**Two qualifications. They are what make this record honest, and neither is optional.**

1. **MVC 4 builds only with the two extra MSBuild properties, and those are a command-line workaround, not a repair.** `/p:SolutionDir=<abs>\src\MVC4\MvcMusicStore\` makes the unconditional import at [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:360] resolve, and `/p:RestorePackages=false` stops the build consulting the absent `src/MVC4/packages`. **Both defects remain present and unrepaired in the repository** (§6.1, §6.2): a plain `MSBuild src\MVC4\MvcMusicStore.sln` on the same host still fails during evaluation on the misplaced import — the `MSB4019` class of failure — and the 24 hint paths still point outside the committed payload. So *"MVC 4 builds"* and *"MVC 4's committed configuration is broken"* are **both true, and neither cancels the other**; §6.3's analysis of why fixing either defect alone is insufficient is unchanged, and §13.1 still records MVC 4 as failing **as committed**.
2. **This observation covers build only. It says nothing about runtime.** Two runtime facts were observed on the same host and belong with it because they bound the claim. **MVC 4's runtime remains broken**: `GET /` returns **HTTP 500**, because its committed connection strings name `(LocalDb)\v11.0` [src/MVC4/MvcMusicStore/Web.config:13], [src/MVC4/MvcMusicStore/Web.config:19] — an instance that cannot be created on Windows Server 2022, which is §10.3's finding seen from the runtime side. **MVC 3's runtime is unavailable**: it needs a first-run SQL Server Compact `.sdf` and the machine-level ASP.NET SQL membership store its `web.config` inherits (§10.1, §10.2), and no SQL Server instance exists on the host. **§13.2 item 2 therefore stays open** — nothing here discharges it.

**What the observation does not establish.**

- **Nothing about the 29 Razor views.** `MvcBuildViews` is `false`, so no view was compiled (§12.3). A zero-warning build and an un-compiled view set are entirely compatible, and the views remain the largest unverified surface in the migration source.
- **It does not retire the precondition failure of §5.1 to §5.3.** That finding is about the repository, not the compiler: a clean checkout still cannot build MVC 5, still has no restore wiring of its own (§5.2), and still does not record where its packages come from (section 8). The restore that made this build possible is exactly the step the repository fails to provide.
- **It does not identify which host-side product inventory satisfied MVC 3's undeclared requirements.** Two host facts were checked directly on that host and are recorded rather than inferred: a `v10.0` web-application targets path exists under `$(MSBuildExtensionsPath32)`, and `System.Web.Mvc` `3.0.0.0` is present in the machine's .NET 4 assembly cache. Both are properties of **the host**, which is precisely §7.2's claim; a host without them fails as §7.2 describes, and the repository still declares neither.
- **It is one host's result.** That is why the inventory above is recorded alongside it: a later run on a different inventory is comparable to this one only because these fields exist.

**How downstream deliverables must report it, stated as a rule so no reader has to infer it.** They cite this document's **carried status** — *blocked pending a Windows verification run* (§1.2) — and may cite this observation as the supplementary evidence beneath it. They must not report MVC 5's build as verified, must not report it as building from a clean checkout, and must not narrow or retire deliverable 07's risk on the strength of it. Deliverable 07's entry stays as the frozen AAP §0.11.2 requires, with the restore-and-reproducibility precondition recorded here and in deliverable 02 §6 as a separate exposure rather than as a replacement for it.

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
| A machine-wide install of the **ASP.NET MVC 3 Tools Update** | **MVC 3 only** | `System.Web.Mvc, Version=3.0.0.0` referenced with **no `HintPath`** [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:42], plus `System.Web.WebPages` `1.0.0.0` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:43] and `System.Web.Helpers` `1.0.0.0` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:44] on the same footing. The repository's own tutorial note confirms the product generation: the assets are "updated for the ASP.NET MVC 3 Tools Update" [src/MVC3/MvcMusicStore-Assets/readme.txt:3] | Reference resolution fails, or resolves to whatever the host's GAC happens to carry. Section 7 records both behaviours |
| A working **NuGet restore** — because 24 of MVC 4's and 26 of MVC 5's assembly references resolve only from a `packages` directory | MVC 5, MVC 4 | `HintPath` counts, reproducible per Appendix A; MVC 3 has exactly **one** `HintPath` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:39] | **Fatal at compile time.** Section 5 is MVC 5's instance of this |
| The committed **NuGet 2.0 client**, reachable at `$(SolutionDir)\.nuget\nuget.exe` | MVC 4 (active), MVC 5 (declared but absent) | `NuGetExePath` resolves to `$(NuGetToolsPath)\nuget.exe` [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:43] with `NuGetToolsPath` = `$(SolutionDir)\.nuget` [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:29]; the self-download fallback is switched **off** [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:16]; `CheckPrerequisites` raises `Unable to locate '$(NuGetExePath)'` when it is missing [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:71] | Restore-on-build fails. Section 6.1 is MVC 4's instance |

**A note on the `$(VSToolsPath)` guard, because it is weaker than it looks.** The condition on that import tests whether the *property* is non-empty — not whether the *file* exists. And the property is always non-empty by the time the import is evaluated: the project defaults `VisualStudioVersion` to `10.0` when the build does not supply it [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:268] and then defaults `VSToolsPath` from it [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:269], and `$(MSBuildExtensionsPath32)` is always defined. So the condition is true in practice, the import is always attempted, and **it fails on any host that lacks the resolved path** — the same missing-import failure MVC 3 suffers unconditionally (section 7.2). The guard protects against an unset property, not against an absent toolchain.

The practical consequence is a build requirement: **`VisualStudioVersion` must resolve to a generation whose web-application targets are actually installed.** A modern MSBuild supplies it (Visual Studio 2022 reports `17.0`) and the path resolves; a bare or older invocation falls through to the project's `10.0` default and the import fails. This is why an MVC 5 or MVC 4 build must be driven by the Visual Studio MSBuild rather than by an arbitrary MSBuild on the `PATH` — which is why §3.2's observation records how MSBuild was located and that it was invoked by full path, and why the MSBuild version is one of the fields §13.2 item 1 requires of any run that closes the gate.

MVC 5 and MVC 4 both opt into restore-on-build — `<RestorePackages>true</RestorePackages>` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:24], [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:24] — which is what makes `BuildDependsOn` prepend the `RestorePackages` target [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:57-60]. The difference is that MVC 4's import of that targets file is unconditional while MVC 5's is guarded by an existence check, so the property is live in one edition and inert in the other. Section 6.5 draws the contrast; section 8 records the restore posture.

### 4.3 What each project file demands of the web host

The *current* hosting requirement per edition, read out of the project files. The target-state hosting recommendation is deliberately absent here and is owned by deliverable 06.

| Edition | Declared web host | Evidence | Note |
| --- | --- | --- | --- |
| MVC 5 | **IIS Express**, over plain HTTP on port 43524 | `UseIISExpress` `true` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:18]; `UseIIS` `True`, `AutoAssignPort` `True`, `DevelopmentServerPort` `43524`, `IISUrl` `http://localhost:43524/` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:281], [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:282], [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:283], [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:285] | `IISExpressSSLPort` is **empty** [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:19], so no HTTPS endpoint is configured. `NTLMAuthentication` is `False` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:286] |
| MVC 4 | **IIS Express**, over plain HTTP on port 4321 | `UseIISExpress` `true` [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:18]; `UseIIS` `True`, `AutoAssignPort` `True`, `DevelopmentServerPort` `5928`, `IISUrl` `http://localhost:4321/` [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:346], [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:347], [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:348], [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:350] | `IISExpressSSLPort` is **empty** [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:19] and `NTLMAuthentication` is `False` [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:351], as in MVC 5. Its three port-bearing properties do not all agree, and **only one of them governs** — the reading rule below the table settles which, and it is stated there once for all three editions rather than as a contradiction here |
| MVC 3 | **The Visual Studio Development Server**, on port 26641 | `UseIISExpress` **`false`** [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:17]; `UseIIS` **`False`**, `AutoAssignPort` `False`, `DevelopmentServerPort` `26641`, `IISUrl` **empty** [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:223-228] | **A requirement no current toolchain satisfies.** The Visual Studio Development Server was superseded by IIS Express and no longer ships. MVC 3 must be re-pointed at a real web host before it can be run at all |

**How to read the three port-bearing properties — stated once, for all three editions.** Each project carries `UseIIS`, `DevelopmentServerPort` and `IISUrl`, and it is `UseIIS` that decides which of the other two means anything. Where `UseIIS` is `True`, the project is hosted by IIS Express and **`IISUrl` is the address that governs**, while `DevelopmentServerPort` belongs to the Visual Studio Development Server and is inert. Where `UseIIS` is `False`, the reverse holds.

Applied to the committed values, that resolves all three editions without any of them contradicting itself:

- **MVC 4's effective declared address is `http://localhost:4321/`.** `UseIIS` is `True` [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:346], so `DevelopmentServerPort` `5928` [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:348] is a residual value from a host this project does not use, **not a second endpoint and not a conflict to be resolved before the application can run**. Nothing in this document may report MVC 4 as declaring two ports.
- **MVC 5 has no such divergence** — both values are 43524 [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:283], [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:285] — and MVC 3 inverts the rule: `UseIIS` is `False` with an **empty** `IISUrl`, so its `DevelopmentServerPort` `26641` is the value that governs, which is exactly why its declared host is the one that no longer ships (§7.3).

One qualification applies to MVC 5 and MVC 4 alike: `AutoAssignPort` is `True` in both [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:282], [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:347], so the committed port is a **starting value the IDE may reassign** rather than a fixed hosting requirement; MVC 3 sets it `False` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:224], so its port is fixed. And the project file is the **only committed source** of these values: `SaveServerSettingsInUserFile` is `False` in both web-hosted editions [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:290], [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:355], and **neither committed `.csproj.user` file declares a `DevelopmentServerPort`, an `IISUrl` or any other port number**: `src/MVC4/MvcMusicStore/MvcMusicStore.csproj.user` carries debugger and IDE-view settings only, and `src/MVC5/MvcMusicStore/MvcMusicStore.csproj.user` adds `UseIISExpress` `true` and an **empty** `IISExpressSSLPort`, both of which merely repeat the project file rather than override it. Deliverable 08 owns those two files as repository-hygiene debt; the fact that they do not override the hosting values is what belongs here.

MVC 3's configuration adds two IIS integrated-pipeline compatibility settings that are hosting requirements in their own right: `validateIntegratedModeConfiguration="false"` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:43] and `runAllManagedModulesForAllRequests="true"` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:44]. The first suppresses the integrated-mode configuration check rather than satisfying it; the second routes every request — including static files — through the full managed module pipeline.

All three editions build to `bin\` in both configurations [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:33], [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:41], and all three set `WarningLevel` `4` without treating warnings as errors [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:36], [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:44]. The equivalents hold in MVC 4 [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:30], [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:33], [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:38], [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:41] and MVC 3 [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:23], [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:26], [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:31], [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:34].

### 4.4 One publish-time behaviour per edition, and only one

Six XDT transform files exist — a `Web.Debug.config` and a `Web.Release.config` per edition — and between them they carry exactly **one active transform each in the three Release files and none at all in the three Debug files**:

| File | `xdt:Transform` occurrences, with the locator of each | Active | The one that is active |
| --- | --- | --- | --- |
| `src/MVC5/MvcMusicStore/Web.Release.config` | 3 — [src/MVC5/MvcMusicStore/Web.Release.config:14], [src/MVC5/MvcMusicStore/Web.Release.config:18], [src/MVC5/MvcMusicStore/Web.Release.config:26] | **1** | `<compilation xdt:Transform="RemoveAttributes(debug)" />` [src/MVC5/MvcMusicStore/Web.Release.config:18] |
| `src/MVC4/MvcMusicStore/Web.Release.config` | 3 — [src/MVC4/MvcMusicStore/Web.Release.config:14], [src/MVC4/MvcMusicStore/Web.Release.config:18], [src/MVC4/MvcMusicStore/Web.Release.config:26] | **1** | the same transform, at the same line [src/MVC4/MvcMusicStore/Web.Release.config:18] |
| `src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Web.Release.config` | 3 — [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Web.Release.config:14], [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Web.Release.config:18], [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Web.Release.config:26] | **1** | the same transform, at the same line [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Web.Release.config:18] |
| `src/MVC5/MvcMusicStore/Web.Debug.config` | 2 — [src/MVC5/MvcMusicStore/Web.Debug.config:14], [src/MVC5/MvcMusicStore/Web.Debug.config:25], both inside comment blocks | **0** | — |
| `src/MVC4/MvcMusicStore/Web.Debug.config` | 2 — [src/MVC4/MvcMusicStore/Web.Debug.config:14], [src/MVC4/MvcMusicStore/Web.Debug.config:25], both inside comment blocks | **0** | — |
| `src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Web.Debug.config` | 2 — [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Web.Debug.config:14], [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Web.Debug.config:25], both inside comment blocks | **0** | — |

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

Every other transform in every one of the six files is commented-out template text — for MVC 5, the blocks at [src/MVC5/MvcMusicStore/Web.Release.config:6-16] and [src/MVC5/MvcMusicStore/Web.Release.config:19-29]. The active transform strips the `debug` attribute that `Web.config` sets to `true` [src/MVC5/MvcMusicStore/Web.config:33] when publishing under the Release configuration.

Two consequences, both narrow and both worth stating:

- **A Release publish is required for a deployed application not to run with debug compilation enabled**, and it is the only thing the repository's transform files do for a deployment. This holds identically in all three editions.
- **`customErrors` is never configured anywhere**, and the figure has to name its unit, because **four** different numbers are all true of the same evidence and only one of them counts an element. Across the six XDT files: **24 occurrences of the word `customErrors`**, of which **12 match the literal `<customErrors`**, of which **6 are actual example opening elements**, of which **0 are live** — uniformly **4, 2, 1 and 0 per file**. Two boundaries explain the drops, and both are mechanical. From 24 to 12: of the four word occurrences in each file, only two are preceded by `<` — the prose reference to the `<customErrors>` section [src/MVC5/MvcMusicStore/Web.Release.config:21] and the example element that carries attributes [src/MVC5/MvcMusicStore/Web.Release.config:25] — while the bare word in the explanatory sentence [src/MVC5/MvcMusicStore/Web.Release.config:22] and the closing tag `</customErrors>` [src/MVC5/MvcMusicStore/Web.Release.config:28] do not, the latter because its `<` is followed by `/`. From 12 to 6: of those two literal matches, only the one followed by whitespace and attributes is an *opening element* at all; the prose reference is `<customErrors>` inside a sentence. **So the 12 is a literal-match count and not an element count** — the element count is 6. Each unit has its own command:

  ```bash
  git ls-files -- '*Web.Debug.config' '*Web.Release.config' | grep -v '/packages/' | xargs grep -o 'customErrors' | wc -l                # -> 24  word occurrences (4 per file)
  git ls-files -- '*Web.Debug.config' '*Web.Release.config' | grep -v '/packages/' | xargs grep -o '<customErrors' | wc -l               # -> 12  literal '<customErrors' matches (2 per file)
  git ls-files -- '*Web.Debug.config' '*Web.Release.config' | grep -v '/packages/' | xargs grep -oE '<customErrors[[:space:]]' | wc -l   # -> 6   example opening elements (1 per file)

  # the same three units file by file, so the 4 / 2 / 1 claim above is reproducible per file
  for f in $(git ls-files -- '*Web.Debug.config' '*Web.Release.config' | grep -v '/packages/'); do
      printf '%s %s %s %s\n' "$f" "$(grep -o 'customErrors' "$f" | wc -l)" \
                                  "$(grep -o '<customErrors' "$f" | wc -l)" \
                                  "$(grep -oE '<customErrors[[:space:]]' "$f" | wc -l)"
  done
  # -> '4 2 1' for each of the six files
  ```

  Liveness is the fourth unit and is evidenced by locator rather than by count: in MVC 5's Release transform both of that file's two literal matches — the prose reference at [src/MVC5/MvcMusicStore/Web.Release.config:21] and the one opening element at [src/MVC5/MvcMusicStore/Web.Release.config:25] — sit inside the comment block at [src/MVC5/MvcMusicStore/Web.Release.config:19-29], which opens at [src/MVC5/MvcMusicStore/Web.Release.config:19] and closes at [src/MVC5/MvcMusicStore/Web.Release.config:29], and its Debug counterpart sits inside [src/MVC5/MvcMusicStore/Web.Debug.config:18-28]. The `xdt:Transform` command above establishes the same result mechanically for all six files: after comments are stripped, the only surviving transform in any of them is the `debug` removal. So the repository configures no production error-display policy at all. Deliverable 09 owns that as a disclosure question and deliverable 05 owns the replacement design; the deployment fact is that nothing in the repository sets it.

How the one active behaviour is re-expressed after migration is owned by deliverable 05.

---

## 5. MVC 5 — a clean checkout cannot build it, and its build status is carried as blocked

MVC 5 is **the sole migration source**, and it is the edition whose build status this assessment carries as blocked. That is why this section exists in the form it does, and why its two halves must not be collapsed into one: §5.1 to §5.3 record a **precondition failure that is a property of the repository** and is unchanged by any host, while §5.4 states the status the document carries and points at the one place the post-freeze observation is recorded. The status is the one stated in §1.2 — **blocked pending a Windows verification run** — and nothing in this section closes it.

### 5.1 The observed failure

`xbuild src/MVC5/MvcMusicStore.sln` in the Debug configuration produced a **`CS0246` cascade** — "type or namespace could not be found" — on `ActionResult`, `DbSet`, `Controller` and `ControllerContext`, followed by consequent **`CS0115`** errors on members declared with `override` whose base types were among the types that had failed to resolve.

The cause is singular and mechanical. Every one of MVC 5's assembly references that carries a hint path uses the form `..\packages\<id>.<version>\lib\net45\<assembly>.dll` — for example `..\packages\Microsoft.AspNet.Mvc.5.0.0\lib\net45\System.Web.Mvc.dll` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:78], and likewise `..\packages\EntityFramework.6.0.0\lib\net45\EntityFramework.dll` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:115]. Those paths are **project-relative**, so from `src/MVC5/MvcMusicStore/` they resolve to `src/MVC5/packages`. That directory does not exist in a clean checkout. With `System.Web.Mvc` unresolved, `ActionResult`, `Controller` and `ControllerContext` have no declaring assembly; with `EntityFramework` unresolved, neither does `DbSet`; and the `CS0115` errors follow mechanically from the missing base types.

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

**Absence 1 — no packages payload.** MVC 5 commits nothing under `src/MVC5/packages`: 0 tracked files, against 169 for MVC 4 and 46 for MVC 3 (section 8). MVC 5 therefore has no assembly to resolve against until a restore runs. The directory a restore would create is also ignored on this checkout — by `Packages/` [.gitignore:33], which matches a directory of that name at any depth but matches this lowercase one only where the checkout is case-insensitive, and **not** by `packages/*` [.gitignore:15], which is anchored to the repository root and does not reach a nested path (Appendix A).

**Absence 2 — no `.nuget` folder at all**, despite MVC 5's solution declaring one as a solution folder and listing two files inside it: `Project("{2150E333-8FDC-42A3-9474-1A3956D46DE8}") = ".nuget", ".nuget", …` [src/MVC5/MvcMusicStore.sln:8] with `.nuget\NuGet.Config` and `.nuget\NuGet.targets` as its contents [src/MVC5/MvcMusicStore.sln:10-11]. No such folder or files exist anywhere under `src/MVC5`. The project's corresponding import is guarded — `Condition="Exists('$(SolutionDir)\.nuget\NuGet.targets')"` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:295] — so the absence is silent rather than fatal, and its effect is that **MVC 5's own `<RestorePackages>true</RestorePackages>` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:24] is inert**: the property is set, but the target that would honour it is never imported. MVC 5 has no restore wiring of its own. Deliverable 02 §5.2 records the same fact from the dependency side.

Note what is *not* wrong here. MVC 5's hint paths point at `src/MVC5/packages`, which is exactly where a restore driven from `src/MVC5/MvcMusicStore.sln` would place them — the `PackagesDir` convention is `$(SolutionDir)\packages` [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:31], and `SolutionDir` for that solution is `src/MVC5/`. **MVC 5's configuration is internally coherent; only the payload is missing.** That is precisely why its failure is a precondition failure and not a defect, and it is the opposite of MVC 4's situation in section 6.

### 5.3 What this proves, and what it does not

**What it proves.** A **precondition failure**: *MVC 5 cannot be built from a clean checkout without a working restore, and the repository does not carry one.* That is a real, actionable finding — it means any build host, pipeline or developer onboarding step for MVC 5 must include a restore against a reachable package source before a build is attempted, and that no offline build of MVC 5 is possible from the repository as committed. It compounds with the unconfigured restore source that deliverable 02 §6 records: the build needs a restore, and the repository does not say where the restore should go.

**What it does not prove.** Anything whatsoever about whether the application compiles once restored. The observed errors are all reference-resolution consequences; they say nothing about the state of the source. A build that never had its references available has not tested its source.

### 5.4 The status this document carries for MVC 5, and where the observation sits

**The carried status is §1.2's and this section does not restate it in different words: *blocked pending a Windows verification run*, on the authority of the frozen AAP §0.6.1 and §0.11.2.** Two consequences follow and both are concrete: deliverable 07 keeps the corresponding risk rather than retiring it, and §13.2 item 1 stays open. The roadmap's first workstream, owned by [03 — Modernization Roadmap](03-modernization-roadmap.md), is therefore still sequenced with build verification as a gate it must pass.

**The post-freeze Windows observation is recorded in §3.2 and nowhere else.** Whoever executes the AAP's verification run has that evidence available — its host, its tool inventory, its commands, its per-edition results and its two qualifications — and §3.2 also states the one field the observation leaves unfilled: it built **Debug only**, so §4.4's Release publish path is unexercised. That gap, and the gate itself, are why the status here is unchanged rather than upgraded.

**What §5.1 to §5.3 establish is unaffected by any host, and it is the part of this section a reader should carry away.** A clean checkout cannot build MVC 5, the repository provides no restore wiring of its own (§5.2), and it does not record where its packages come from (section 8) — so every build host, pipeline and onboarding path must supply a restore against a source it names for itself.

### 5.5 Why the honest status matters

An assessment that reported "MVC 5 builds clean" on the strength of the repository alone would have been asserting something nobody had observed. The temptation was real, because the observed Mono failure had an obvious and innocuous explanation — a missing directory, not a code defect — and it is tempting to reason from the explanation to the conclusion. The conclusion does not follow from the explanation, and the honest statement was the more useful one: **MVC 5 cannot build from a clean checkout, and whether it compiles once restored is a separate question.** The first half is an actionable finding about the build system; the second is a scoped, one-shot verification task.

Reporting the two separately is what keeps them both, and it is why §5.1 to §5.3 and §3.2 are different subsections rather than one paragraph. A single sentence merging them would lose the finding that survives every host.

**One further separation, and it is the one this section is really about.** Observing a build is not the same as discharging a gate. The AAP set the gate, the AAP is frozen, and the gate closes when the AAP is revised by approval — not when an agent runs the build. So the carried status is *still* the blocked one, and §3.2's observation sits beneath it as evidence for the approver rather than in place of their decision. That is not a formality: the whole engagement is gated on approval (§1.4), and a deliverable that quietly upgrades its own status is a deliverable that has taken an approval decision on the approver's behalf.

---

## 6. MVC 4 — two independent, platform-independent defects, plus a stale solution

MVC 4's builds fail **before compilation**, for reasons that are entirely in the committed configuration. Two defects sit in the project file and a third in one of its two solution files. All three are platform-independent: each is a path that does not exist, and a path that does not exist fails identically under every MSBuild implementation.

### 6.1 Defect 1 — a misplaced, unconditional NuGet target import

The last import in the project file is, verbatim:

```xml
<Import Project="$(SolutionDir)\.nuget\nuget.targets" />
```

at [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:360], **with no `Condition` attribute**.

Two facts make it fatal. First, `$(SolutionDir)` resolves to `src/MVC4/` when the project is built through `src/MVC4/MvcMusicStore.sln` — the solution whose project path is correct — and the project's own fallback agrees, defaulting `SolutionDir` to `..\` relative to the project directory [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:23]. Second, **there is no `.nuget` folder at `src/MVC4/`.** The only one in the repository is one level deeper, at `src/MVC4/MvcMusicStore/.nuget`:

```bash
test -d src/MVC4/.nuget && echo present || echo absent   # -> absent
git ls-files | grep '\.nuget/'                            # -> the three files, all under src/MVC4/MvcMusicStore/.nuget/
```

Because the import is unconditional, **MSBuild on Windows raises the same missing-import error** — the `MSB4019` class of failure, "the imported project was not found" — during project evaluation, before any target runs and therefore before the compiler is invoked. This is not a Mono artefact and must not be reported as one. The conclusion rests on the import's own
text rather than on the engine that read it: an `<Import>` carrying no `Condition` is evaluated by
every MSBuild implementation, and the path it names is absent from the checkout (§6.1). That is what
makes the diagnosis platform-independent.

The diagnosis is no longer only a prediction: on the Windows host of §3.2, a plain
`MSBuild src\MVC4\MvcMusicStore.sln` — the committed configuration, with none of the host-side
properties of §6.3 — **failed during evaluation on this import**, in the `MSB4019` class of failure.
That is one host's observation of a defect whose cause is in the file
[src/MVC4/MvcMusicStore/MvcMusicStore.csproj:360] rather than in any engine, and §3.2's first
qualification states the corollary that matters: the exit `0` recorded there for MVC 4 was obtained
**with** those properties, and the defect itself remains present and unrepaired.

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
| `src/MVC4/` (what the correct solution gives) | **No** — `src/MVC4/.nuget` is absent, so the unconditional import at `:360` fails | Points at `src/MVC4/packages`, which a restore of that solution does create — but the **restore-on-build** that would create it never runs, because defect 1 fails evaluation first. A **standalone** `nuget restore` of the solution creates it, which is the sequence §3.2 used |
| `src/MVC4/MvcMusicStore/` | **Yes** — `.nuget` is there | Still points at `src/MVC4/packages`, because hint paths are project-relative and unaffected by `SolutionDir` |

**Stated plainly: the repository contains the two halves of a working configuration in two different places, and no single committed configuration has both.** The `.nuget` folder is where only the stale solution would find it; the hint paths point where only the correct solution's restore would fill. Fixing the import alone leaves 24 unresolved references; providing the packages alone leaves the import fatal. Both must be addressed, and a plan that treats this as one problem will under-scope it.

That is also why any MVC 4 build needs build-time overrides rather than the committed configuration — supplying `SolutionDir` one level deeper so the import resolves, while suppressing restore-on-build so the absent `src/MVC4/packages` is not consulted. **Those overrides are host-side workarounds, not fixes**, they are passed on the command line, and this assessment applies none of them to any tracked file.

The analysis above is derived from the two defects and holds on its own. It was also exercised: on the Windows host of §3.2, MVC 4 built to exit `0` **with exactly those two properties supplied on the command line**, and failed during evaluation without them (§6.1). Two limits on reading that, both deliberate. It says nothing about MVC 4 at **runtime**, which remains broken for an unrelated reason (§10.3, and §3.2's second qualification). And it does not soften the negative, which is platform-independent and is what §13.1 records: **as committed, both MVC 4 solutions fail before compilation** (§3.1, §6.1, §6.4), so a host that does not already know about a defect the repository does not announce cannot build this edition at all.

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

MSBuild fails to load the solution — the `MSB3202` class of failure, "the project file was not found" — so this solution never reaches the project, let alone the compiler. The declared path is absent from the checkout, as the command above shows, which makes this failure platform-independent in the same way as the defect in §6.1: it is observed under Mono (§3.1) and no MSBuild implementation can resolve a project file that does not exist. It is the only solution in the repository that declares `NuGet.exe` among its solution items [src/MVC4/MvcMusicStore/MvcMusicStore.sln:6-12], which is consistent with it being the file under which the committed payload of section 6.2 was originally produced.

**Taken together, these three defects are why both MVC 4 solutions fail before compilation**, and they fail for different reasons: the correct solution at `src/MVC4/MvcMusicStore.sln` fails on the two project-file defects of sections 6.1 and 6.2, while the stale solution fails on its own unresolvable project path. Anyone diagnosing this by trying the other solution file will get a different error and conclude, wrongly, that there is one problem.

Which solution is which is a build requirement in its own right: **`src/MVC4/MvcMusicStore.sln` is the MVC 4 solution to use, and `src/MVC4/MvcMusicStore/MvcMusicStore.sln` must not be used.** Deliverable 08 owns the debt entry for the stale file; the requirement to avoid it is owned here.

### 6.5 The contrast that shows this is MVC 4's defect, not the era's

It would be easy to file all of this under "2012-era project files are fragile". The repository refutes that directly: **MVC 5 conditions the very imports MVC 4 and MVC 3 leave unguarded.**

| Import | MVC 5 | MVC 4 | MVC 3 |
| --- | --- | --- | --- |
| `$(SolutionDir)\.nuget\NuGet.targets` | `Condition="Exists('$(SolutionDir)\.nuget\NuGet.targets')"` — **guarded on the file existing** [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:295] | **no condition at all** [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:360] | not imported |
| `WebApplication.targets` via `$(VSToolsPath)` | `Condition="'$(VSToolsPath)' != ''"` — guarded on a property that is always set (§4.2) [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:272] | the same weak guard [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:337] | not imported |
| `WebApplication.targets` at the hard-coded v10.0 path | `Condition="false"` — inert [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:273] | `Condition="false"` — inert [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:338] | **no condition at all** [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:209] |

The decisive row is the first, and it is the one the failures turn on. **MVC 5 guards its NuGet import on the target file actually existing; MVC 4 does not guard the equivalent import at all.** That single difference is why MVC 5's missing `.nuget` folder is silent (section 5.2) and MVC 4's is fatal (section 6.1). In the third row the polarity reverses: MVC 5 and MVC 4 both neutralise the hard-coded `v10.0` path with `Condition="false"`, and MVC 3 imports it unguarded.

Stated without overclaiming: **MVC 5 is written more defensively than MVC 4 and MVC 3, but not uniformly so** — its `$(VSToolsPath)` guard is the weak kind analysed in section 4.2, and it shares that weakness with MVC 4. What the table establishes is the point that matters here: **these are defects in specific project files, not properties of the vintage**, since the same repository and the same era contain both the guarded and the unguarded form of the same import. A migration plan should treat them accordingly. Deliverable 02 §4.3 tabulates the same imports as dependencies; the build consequence is what this table adds.

---

## 7. MVC 3 — compiles, but its requirements differ by toolchain rather than being absolute

### 7.1 What the Mono build produced

`xbuild src/MVC3/MvcMusicStore-Completed/MvcMusicStore.sln` in Debug produced `bin/MvcMusicStore.dll`, with **three unresolved-reference warnings**: `System.Web.WebPages` `1.0.0.0`, `System.Web.Helpers` `1.0.0.0` and `System.Web.Entity` could not be resolved. Mono resolves enough from its own GAC and targets to compile in spite of them.

Two of the three are visible in the project as bare references with no hint path — `System.Web.WebPages, Version=1.0.0.0` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:43] and `System.Web.Helpers, Version=1.0.0.0` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:44] — and the third is the framework assembly `System.Web.Entity` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:50].

The one reference MVC 3 *does* hint is its ORM: `..\packages\EntityFramework.4.1.10331.0\lib\EntityFramework.dll` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:39]. From `src/MVC3/MvcMusicStore-Completed/MvcMusicStore/` that resolves to `src/MVC3/MvcMusicStore-Completed/packages`, **which is committed** — 46 tracked files. MVC 3 is thus the only edition whose single hint path is satisfied by the repository as committed, which is the whole reason its build got as far as it did.

**No output artefact size is recorded**, for the reason given in section 3: the figure is not stable across toolchains and would invite a false comparison.

### 7.2 On Windows the picture differs

Two of MVC 3's requirements are toolchain-dependent rather than absolute, and both are stricter on a current Windows host than they were under Mono.

**The MVC framework assembly has no hint path.** `System.Web.Mvc, Version=3.0.0.0` is a bare reference [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:42] with no package folder behind it, so it resolves only from a **machine-wide install of the ASP.NET MVC 3 Tools Update** — a separately installed, out-of-support product. The repository corroborates the generation in its own words: the tutorial assets are "updated for the ASP.NET MVC 3 Tools Update" [src/MVC3/MvcMusicStore-Assets/readme.txt:3]. Configuration corroborates it a second time, listing `System.Web.Mvc, Version=3.0.0.0` among the assemblies the ASP.NET compilation system must load [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:21] and redirecting older references forward to `3.0.0.0` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:50-51]. Deliverable 02 §4.1 records the same assembly as an undeclared dependency; the build requirement it creates — **a machine-wide product install on the build host** — is recorded here.

**The web-application targets import is unconditional and points at a Visual Studio 2010-era path.** `$(MSBuildExtensionsPath32)\Microsoft\VisualStudio\v10.0\WebApplications\Microsoft.WebApplication.targets` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:209] carries no `Condition`, so on a host without that exact path MSBuild fails during evaluation — the same `MSB4019` class of failure as MVC 4's defect 1, and equally platform-independent. A Visual Studio 2022-only machine does not carry a `v10.0` extensions path.

The practical consequence for a build host is therefore: **MVC 3 requires either a Visual Studio 2010-era `v10.0` web-application targets path on disk, or a compatibility shim that supplies one.** A shim is a host-side arrangement, not a repository change, and this assessment makes none.

Both requirements are therefore **properties of the host's product inventory, not of the repository**, and that is precisely this section's claim. The Mono run's three unresolved-reference warnings (§7.1) are the same fact seen from the other side: the assemblies MVC 3 names are supplied — or not — by whatever the host happens to carry, and nothing in the repository determines it.

§3.2 illustrates the point from the favourable side rather than weakening it. On that host MVC 3 built to exit `0` with no warnings, and two host facts recorded there are why: a `v10.0` web-application targets path exists under `$(MSBuildExtensionsPath32)`, and `System.Web.Mvc` `3.0.0.0` is in the machine's .NET 4 assembly cache. Neither is declared by the repository, so the requirement stands exactly as stated above: **on a host lacking them, the unconditional `v10.0` import fails outright and the bare `System.Web.Mvc` reference has nothing to resolve against.** One host's success is not a property of the project file.

### 7.3 Its committed web host no longer ships

Recorded in section 4.3 and repeated here because it is MVC 3's most consequential runtime requirement: MVC 3 declares `UseIISExpress` `false` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:17] and `UseIIS` `False` with an empty `IISUrl` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:223], [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:227-228], which selects the Visual Studio Development Server. That server no longer ships in any current Visual Studio. **MVC 3 cannot be run from its committed configuration on a supported toolchain without being re-pointed at a real web host**, and that is independent of its two database requirements in section 10.

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
| MVC 5 | **none** — 0 tracked files | None wired: `RestorePackages` is `true` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:24] but the targets file that honours it is imported under an `Exists(...)` guard [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:295] against a `.nuget` folder that does not exist | **No.** A restore against a reachable source is mandatory |
| MVC 4 | **169 tracked files** at `src/MVC4/MvcMusicStore/packages`, referenced by no hint path (section 6.2) | **MSBuild-integrated restore** — `RestorePackages` `true` [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:24] driving the `RestorePackages` target [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:76-83] through the committed client | **No, and not online either as committed** — the import that carries the restore target fails first (§6.1). Satisfying the hint paths means a restore driven **through `src/MVC4/MvcMusicStore.sln`**, which fills `src/MVC4/packages` and reaches a source; §6.3 states that as a command-line workaround and §3 strand A records the one time it was run |
| MVC 3 | **46 tracked files** at `src/MVC3/MvcMusicStore-Completed/packages`, which its single hint path does reference | **None** — no `.nuget` folder, no NuGet import, no `RestorePackages` property | **Yes**, for its one hinted reference; its machine-wide requirements are separate (section 7.2) |

**215 tracked files sit under the two committed payloads despite a `packages` directory being excluded** — by `Packages/` [.gitignore:33], whose only separator is trailing and which therefore matches a directory of that name at any depth, and **not** by `packages/*` [.gitignore:15], whose interior separator anchors it to the repository root and leaves both nested payloads unmatched. Neither exclusion untracks a file already added, which is why the 215 are here; and the rule that does match them is case-dependent, so on a case-sensitive host they would match no rule at all (Appendix A). Deliverable 02 §7.2 records the payload composition and 08 owns the debt framing; the build fact is the one in the table.

**MVC 4's restore client is committed, pinned and long deprecated.** `src/MVC4/MvcMusicStore/.nuget/NuGet.exe` is a tracked NuGet **2.0.30828.5** binary — a 2012-era client — and it is required rather than optional: the self-download fallback is switched off [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:16], and `CheckPrerequisites` raises `Unable to locate '$(NuGetExePath)'` when it is missing [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:71]. MSBuild-integrated restore of this kind was dropped with the NuGet 3 generation, so **a current NuGet client cannot be substituted into this wiring** — it can only replace it. Deliverable 02 §5.1 carries the client's verified size, versions and hash; they are not repeated here.

Two further build-relevant properties of that wiring, because they change what a restore does rather than merely where it looks:

- **Restore consent is demanded.** `RequireRestoreConsent` defaults to `true` [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:13] and adds `-RequireConsent` to the restore command [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:51], so a build host that has not opted into package restore fails the restore rather than performing it.
- **The restore command is issued with an empty `-source`.** `PackageSources` resolves from an empty item list [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:44] into the command at [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:53]. The consequence — that the effective source set is a property of the build host and not of the repository — is a finding owned by deliverable 02 §6, corrected there against Technical Specification §3.3, and cross-referenced rather than restated. Its bearing on *this* document is narrow and worth naming: **no build of MVC 4 or MVC 5 is reproducible across hosts from repository evidence alone**, because the repository does not record where its packages come from.

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
| **MVC 4** | LocalDB **`(LocalDB)\v11.0` as committed** [src/MVC4/MvcMusicStore/Web.config:19], file-attached `MvcMusicStore.mdf`, Windows integrated authentication [src/MVC4/MvcMusicStore/Web.config:20-21] — an instance name its own README contradicts (section 10.3) | LocalDB, same instance, a **separate** SimpleMembership database, `Integrated Security=SSPI` [src/MVC4/MvcMusicStore/Web.config:13], [src/MVC4/MvcMusicStore/Web.config:14-16] | Two required `.mdf`/`.ldf` pairs under `App_Data/`, plus two **byte-identical** `MvcMusicStore-Create.sql` copies, **neither runnable as written** (section 10.5; the defect itself is owned by deliverables 12 and 08). A third pair, `MvcMusicStore-work.mdf` and `MvcMusicStore_log-work.ldf`, is referenced by no configuration and is **unreferenced scratch debt, not a runtime requirement** |
| **MVC 3** | **SQL Server Compact 4.0** — a first-run `.sdf` created from the only connection string the edition declares, `Data Source=\|DataDirectory\|MvcMusicStore.sdf` with `providerName="System.Data.SqlServerCe.4.0"` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:55-59]. **No `.sdf` is committed**, so a machine-wide install of the retired provider is required | **A SQL Server instance — not SQL Server Compact.** `<roleManager enabled="true" />` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:15] and Forms authentication [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:26-28] are both enabled, but the file defines **no** membership provider, **no** role provider and **no** `LocalSqlServer` connection string, so classic `Membership` and `Roles` resolve through the **machine-level** ASP.NET SQL providers against the machine's own connection-string setting | **No database under the application's own `App_Data`** — `git ls-files 'src/MVC3/MvcMusicStore-Completed/*' \| grep App_Data` returns nothing. The catalog `.mdf`, the `ASPNETDB.MDF` credential store and the repository's **one runnable** schema script are all **tutorial assets** under `src/MVC3/MvcMusicStore-Assets/Data/` |

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

> **Open verification item — expressly unresolved.** MVC 3's credential-store requirement is **not stated as final by this document and may not be reported as final by any other.** It must be **verified on the supported Windows runtime** first — the actual machine-level provider and the actual connection string it resolves. An inherited default is a property of the *host*, not of the repository, and no amount of reading the repository can settle it. What the repository does settle, and what is stated as final here, is the negative: **MVC 3 declares no membership store of its own.** §13.2 item 2 carries the item; the specification that closes it is immediately below.

**The verification this document requires and does not have — specified, not performed.** The same discipline §1.2 and §5.4 apply to MVC 5's build status applies here, and for the same reason: an open item that does not say what would close it is not closable by anyone else. Seven fields are required, and a run that answers the question without recording them leaves the next reader unable to tell which host the answer belongs to. §3.2's Windows observation does not close it: that run built and did not run the applications, and MVC 3's runtime was unavailable on its host for want of any SQL Server instance.

| # | What must be checked | Why it is required | What the run must record |
| --- | --- | --- | --- |
| 1 | **The host itself** | The answer is a property of the host, so an unidentified host makes the answer unusable | The Windows edition and build, the .NET Framework version serving the application, and — because MVC 3's committed web host no longer ships (§7.3) — the substitute web host the application was actually run under |
| 2 | **The machine-level `<membership>` provider in force** | The application declares none (§10.1), so whatever is in force is inherited from `machine.config` and the root `web.config` of that host | The provider's `type`, its `name`, its `connectionStringName`, and the merged-configuration file the value came from |
| 3 | **The machine-level `<roleManager>` provider in force** | `<roleManager enabled="true" />` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:15] switches roles on **without** declaring a provider, so the role store is inherited separately from the membership store and may not be the same one | The same four values, for the role provider |
| 4 | **The connection string those providers resolve** | This is the requirement itself: which engine and which database the credential store actually is. The name is `LocalSqlServer` by ASP.NET default, but the value is the host's | The connection-string name, its resolved value, and the SQL Server instance and database it names |
| 5 | **Whether the resolved store already exists on that host, or is created on first use** | It decides whether the requirement is "a provisioned database" or "`CREATE DATABASE` rights at first request", which are different deployment requirements and different privilege grants (§11.2) | Which of the two occurred, and the principal that performed it |
| 6 | **Whether the committed `ASPNETDB.MDF` is used at all** | It is referenced by no configuration file in the repository (§10.1), so it is an orphaned tutorial asset until a host's inherited connection string happens to name it | Whether the resolved store is that file, a pre-existing machine-level store, or a newly created one |
| 7 | **The date, and how the merged configuration was read** | Machine-level configuration changes with the host's product inventory, so an undated result cannot be compared against a later one | The date and the exact command or tool used to read the merged configuration |

Two preconditions belong with it, because they are why this item is not trivially discharged. The run needs a **running** MVC 3, not a build, and MVC 3 cannot be run from its committed configuration on a supported toolchain without being re-pointed at a real web host (§7.3) and without the two machine-wide product installs of §7.2 and §10.1. And it must observe the same no-modification boundary as every other check in this document (§1.4): because running MVC 3 out of the checkout would have the provider create its `.sdf` inside the working tree — and, for the two editions that do attach tracked files, would have the engine write to them (§11.1) — the run must serve a copy of the application from outside the repository.

**Until those fields exist, the boundary is exact:** MVC 3 declaring no membership store of its own is established and may be cited; *what* its credential store resolves to is unestablished and may not be reported as a final requirement, in this document or any other.

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

1. **The README documents a value the repository does not commit.** It presents both connection strings as `MSSQLLocalDB` [src/MVC4/README.md:110-112], [src/MVC4/README.md:117-119]; the committed file says `v11.0` [src/MVC4/MvcMusicStore/Web.config:13], [src/MVC4/MvcMusicStore/Web.config:19].
2. **The substitution advice runs in both directions.** [src/MVC4/README.md:102] and [src/MVC4/README.md:122] tell the reader to replace `MSSQLLocalDB` with `v11.0`; [src/MVC4/README.md:139] tells the reader to replace `v11.0` with `MSSQLLocalDB`. The README therefore contradicts itself as well as the configuration, and a reader following it cannot determine which value is intended.
3. **The committed value is unavailable under the README's own stated prerequisite.** The README requires Visual Studio 2022 [src/MVC4/README.md:7] and states that LocalDB is installed with it [src/MVC4/README.md:8]. **Visual Studio 2022 installs no `v11.0` instance** — `v11.0` is the SQL Server 2012 LocalDB generation, superseded by `MSSQLLocalDB`. So MVC 4, as committed, points at an instance that the toolchain it documents does not provide.

The build-and-deployment requirement this creates is unambiguous even though the sources are not: **MVC 4's committed connection strings must be re-pointed at an available LocalDB instance before the application can run**, and that is a code change, which this assessment does not make. The repair is out of scope under the approval gate; deliverable 08 owns the debt entry. Note that this affects **runtime only** — MVC 4's build is unaffected by its connection strings, and its build failures (section 6) have entirely separate causes.

### 10.4 The three initializers' first-run behaviour

All three editions ship the same class shape: `SampleData : DropCreateDatabaseIfModelChanges<MusicStoreEntities>` at [src/MVC5/MvcMusicStore/Models/SampleData.cs:9], [src/MVC4/MvcMusicStore/Models/SampleData.cs:9] and [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Models/SampleData.cs:9]. What differs is where it is registered, what it seeds, and what else runs alongside it.

| Edition | Registered at | Seed payload | What happens on first run |
| --- | --- | --- | --- |
| **MVC 5** | **Twice** — [src/MVC5/MvcMusicStore/Global.asax.cs:20] and [src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:16] | **826 physical lines**; **15 genres, 303 artists, 462 albums** | The catalog database is attached, its schema created if the model does not match, and the seed written. The administrator account and `Administrator` role are then provisioned from `appSettings` by `CreateAdminUser()` [src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:21-44], which builds its own context, `UserManager` and `RoleManager` [src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:27-29] |
| **MVC 4** | **Once** — [src/MVC4/MvcMusicStore/App_Start/AppConfig.cs:16] | **826 physical lines**; **15 genres, 303 artists, 462 albums** (its `SampleData.cs` is byte-identical to MVC 5's — deliverable 01 §10.1) | The same catalog sequence, **plus a separate credential-store creation path**: `Database.SetInitializer<UsersContext>(null)` [src/MVC4/MvcMusicStore/Filters/InitializeSimpleMembershipAttribute.cs:28], then `context.Database.Exists()` [src/MVC4/MvcMusicStore/Filters/InitializeSimpleMembershipAttribute.cs:34] and, if absent, `((IObjectContextAdapter)context).ObjectContext.CreateDatabase()` [src/MVC4/MvcMusicStore/Filters/InitializeSimpleMembershipAttribute.cs:37], then `WebSecurity.InitializeDatabaseConnection(…, autoCreateTables: true)` [src/MVC4/MvcMusicStore/Filters/InitializeSimpleMembershipAttribute.cs:41]. Administrator provisioning follows [src/MVC4/MvcMusicStore/App_Start/AppConfig.cs:21-37] |
| **MVC 3** | **Once** — [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Global.asax.cs:34], inside `Application_Start` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Global.asax.cs:32-40] | **430 physical lines**; **10 genres, 149 artists, 246 albums** | The catalog `.sdf` is created and seeded through the SQL Server Compact provider. **No administrator is provisioned at all** — MVC 3 has no `App_Start` folder and no provisioning code, so its `Administrator` role is never created |

Seed content and seed size are verified by counting the `new Genre`, `new Artist` and `new Album` object initializers and the physical lines **in each of the three files separately** — Appendix A carries the one loop that emits all nine figures, one output row per file, so no edition's figure rests on a command run against another edition's file. Two facts fall out of the last column and both are deployment requirements rather than curiosities:

- **MVC 3's first run produces a different catalog from MVC 4's and MVC 5's** — 10 genres against 15, 149 artists against 303, 246 albums against 462. Any environment intended to compare editions must not assume a common dataset.
- **MVC 4 and MVC 5 both require their administrator credential to exist in `appSettings` at first run** [src/MVC4/MvcMusicStore/Web.config:25-26], [src/MVC5/MvcMusicStore/Web.config:16-17] — values cited but not reproduced, per section 1.3. Provisioning happens at application start, not on demand, so **a provisioning failure is a startup failure**. MVC 4 surfaces it: its initializer wraps the sequence in a `try`/`catch` that rethrows as `InvalidOperationException` [src/MVC4/MvcMusicStore/Filters/InitializeSimpleMembershipAttribute.cs:43-46]. MVC 5 does not: its provisioning method is declared `private async void` [src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:21], so a faulted task is unobservable and a failed provisioning is silent.

**The destructive property of `DropCreateDatabaseIfModelChanges` is the deployment requirement this section is obliged to state**, in operational terms rather than as a debt entry: *if the model does not match the database, the database is dropped and recreated.* Applied to a database holding real orders and personal data, that is data loss with no prompt. The repository's own documentation describes the benign half of the behaviour — the automatic attach, initialize and seed sequence [src/MVC5/README.md:26-31] — and, in its troubleshooting section, the destructive half as a remedy: delete the `.mdf` and `.ldf` files and "the database will be recreated automatically" [src/MVC5/README.md:98-99].

So the requirement is: **any deployment that points one of these applications at a database containing data it cannot afford to lose must first disable the initializer.** The debt framing and severity are owned by deliverable 08; the replacement schema lifecycle is owned by deliverable 05; the hosting sequence for applying schema changes is owned by deliverable 06.

MVC 5 registers the initializer twice, and the effect is worth stating precisely so it is not overstated: `Database.SetInitializer<TContext>` *sets* the strategy rather than adding to a list, so the second registration replaces the first and **exactly one initialization runs**. This is duplicated startup configuration, not a doubled destructive path. Deliverable 01 §3.4 owns the analysis.

### 10.5 The schema scripts as deployment inputs

Three `.sql` files are tracked, and their usability differs. **This section is where that fact is
established for all three editions**; §13.1's per-edition summary row points here and states no verdict of
its own, so there is one place to correct if a script ever changes:

```bash
git ls-files '*.sql'
# -> src/MVC3/MvcMusicStore-Assets/Data/MvcMusicStore-Create.sql
#    src/MVC4/MvcMusicStore-Create.sql
#    src/MVC4/MvcMusicStore/MvcMusicStore-Create.sql
```

| Script | First statement | Usable as a deployment input? | Tables it creates |
| --- | --- | --- | --- |
| [src/MVC3/MvcMusicStore-Assets/Data/MvcMusicStore-Create.sql:1-617] | `USE [MvcMusicStore]` [src/MVC3/MvcMusicStore-Assets/Data/MvcMusicStore-Create.sql:1] | **Yes, given a pre-existing database of that name.** The repository's one runnable script | Six **singular**-named tables |
| [src/MVC4/MvcMusicStore-Create.sql:1-629] | `USE [C:\USERS\JON\…\APP_DATA\MVCMUSICSTORE.MDF]` [src/MVC4/MvcMusicStore-Create.sql:1] | **No** — a hard-coded developer path to an attached MDF | Six **plural**-named tables |
| [src/MVC4/MvcMusicStore/MvcMusicStore-Create.sql:1-629] | identical to the above [src/MVC4/MvcMusicStore/MvcMusicStore-Create.sql:1] | **No** — byte-identical duplicate, same defect | identical |

Each script is cited above at its **full line range** — 617 lines of content for the MVC 3 asset script and 629 for each MVC 4 copy — because the row's judgement is about the file as a whole; the individual claims inside it carry their own lines, starting with the `USE` statement at line 1. All three are UTF-16 encoded, and the MVC 3 script carries a trailing blank line after its final `GO`, so a byte-oriented line count reports one more than the last line that carries a statement; the ranges above count statement-bearing lines, which is the unit the `USE`-statement locator shares.

The table names are the load-bearing difference, so each is cited individually. The MVC 3 asset script declares `Order` [src/MVC3/MvcMusicStore-Assets/Data/MvcMusicStore-Create.sql:41], `Genre` [src/MVC3/MvcMusicStore-Assets/Data/MvcMusicStore-Create.sql:70], `Artist` [src/MVC3/MvcMusicStore-Assets/Data/MvcMusicStore-Create.sql:101], `Album` [src/MVC3/MvcMusicStore-Assets/Data/MvcMusicStore-Create.sql:272], `OrderDetail` [src/MVC3/MvcMusicStore-Assets/Data/MvcMusicStore-Create.sql:551] and `Cart` [src/MVC3/MvcMusicStore-Assets/Data/MvcMusicStore-Create.sql:579]. Both MVC 4 copies declare `Genres` [src/MVC4/MvcMusicStore/MvcMusicStore-Create.sql:52], `Orders` [src/MVC4/MvcMusicStore/MvcMusicStore-Create.sql:81], `Artists` [src/MVC4/MvcMusicStore/MvcMusicStore-Create.sql:108], `Albums` [src/MVC4/MvcMusicStore/MvcMusicStore-Create.sql:277], `OrderDetails` [src/MVC4/MvcMusicStore/MvcMusicStore-Create.sql:549] and `Carts` [src/MVC4/MvcMusicStore/MvcMusicStore-Create.sql:570].

One encoding detail belongs here because it affects execution rather than reading: **all three scripts are UTF-16 with a byte-order mark** — the first two bytes of each are `FF FE`, and the MVC 3 asset script carries a doubled mark, `FF FE FF FE`. Any tool used to execute them must handle UTF-16 input, which rules out a naive byte-oriented pipeline and is worth knowing before a deployment step is scripted around one.

The duplication is proven rather than assumed: both MVC 4 copies are 153,594 bytes with the same SHA-256, `D577AAA51949E54D1C83D57E23F1BB96A840661EFF9FE478F2CF8A53DD182C9D` (reproducing command in Appendix A). **Neither copy is runnable as written**, so neither is a usable schema baseline; the defect itself belongs to deliverables 12 and 08. MVC 4's own README confirms the requirement to edit the script before using it — "Update the `USE` statement at the top of the file to point to your LocalDB instance" [src/MVC4/README.md:82], repeated in its troubleshooting section [src/MVC4/README.md:156] — and accurately lists the plural tables the script creates [src/MVC4/README.md:91].

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

A deployment consequence follows that is easy to miss and expensive to discover, and it has to be stated **per edition**, because the three editions do not ship, attach and write the same files. Collapsing them into one sentence produces a claim that is false for MVC 3.

**What a copy of the checkout carries, and under which ignore rule.** Fourteen database binaries are tracked, 43,376,640 bytes in total (Appendix A), and they are neither in one place nor under one rule. **Ten** sit under an application's own `App_Data` — six in MVC 4, four in MVC 5 — and are excluded by `App_Data/` [.gitignore:32] yet tracked in spite of it. **MVC 3's four sit under `src/MVC3/MvcMusicStore-Assets/Data/` and match no ignore rule at all**, because that directory is a tutorial asset tree rather than any application's data directory:

```bash
for f in $(git ls-files | grep -iE '\.(mdf|ldf)$'); do
    printf '%s -> ' "$f"; git check-ignore --no-index -v "$f" || echo 'no rule'
done
# -> the 10 under App_Data/ each report '.gitignore:32:App_Data/'
#    the 4 under src/MVC3/MvcMusicStore-Assets/Data/ each report 'no rule'
```

**What each edition actually attaches and writes.** This is the part a single statement gets wrong:

| Edition | Does a run attach a committed database file? | What a run writes | Where its credential rows go |
| --- | --- | --- | --- |
| **MVC 5** | **Yes** — both connection strings name `AttachDbFilename` under `App_Data` [src/MVC5/MvcMusicStore/Web.config:12], [src/MVC5/MvcMusicStore/Web.config:13] | The four **tracked** binaries under `src/MVC5/MvcMusicStore/App_Data/` — so a direct run modifies the working tree | Into the attached, committed Identity 1.0 database |
| **MVC 4** | **Yes** — the same construct for both stores [src/MVC4/MvcMusicStore/Web.config:16], [src/MVC4/MvcMusicStore/Web.config:20] | Four of the six **tracked** binaries under `src/MVC4/MvcMusicStore/App_Data/`; the `-work` pair is referenced by no configuration (section 10.1) | Into the attached, committed SimpleMembership database |
| **MVC 3** | **No** — its one connection string is a SQL Server Compact `Data Source` for a `.sdf` that is not committed [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:57-58], and the application ships no data directory at all: `git ls-files 'src/MVC3/MvcMusicStore-Completed/*' \| grep App_Data` returns nothing (section 10.1) | A **new** `.sdf`, created under the application's own `App_Data` on first run. **No tracked binary is touched** | Into a **machine-level SQL Server** store outside the checkout, inherited rather than declared (§10.2). That store is an **open verification item**: the actual provider and its connection string must be verified on the supported Windows runtime **before the requirement is stated as final** (§10.2, §13.2 item 2) |

So the four binaries under `src/MVC3/MvcMusicStore-Assets/Data/` — the catalog `.mdf`, the `ASPNETDB.MDF` credential store and their two log files — are **shipped by a copy of the checkout and attached by nothing**: `git grep -il 'ASPNETDB' -- 'src/'` returns 0 (Appendix A, section 10.1), so no configuration anywhere in the repository names that store. They are tutorial assets and repository weight, not an MVC 3 runtime requirement, and MVC 3's effective credential store is the host's, not theirs.

**The requirement, scoped accordingly.** For **MVC 4 and MVC 5** it is absolute and it is about tracked content: running either application directly out of the checkout makes the database engine attach and write **tracked** files, which modifies the working tree, so **any run or deployment of those two editions must serve a copy of the application from outside the repository.** For **MVC 3** the same rule holds for different reasons — it creates a new `.sdf` inside the application directory and reaches a credential store the repository does not configure — so serving a copy from outside the repository is still the requirement, but no committed database is involved either way. And in all three cases a deployed copy of the checkout carries all fourteen binaries and their 43,376,640 bytes, including **two** credential stores that an application does attach and **one**, MVC 3's tutorial `ASPNETDB.MDF`, that none does. Whether the binaries should have been tracked at all belongs to deliverable 08; the exposure created by shipping credential stores belongs to [09 — Security Assessment](09-security-assessment.md).

### 11.2 Database — a trusted Windows identity, and schema-changing rights at startup, of a different kind per edition

**The initializer appears in this table once, as two rows rather than one, and that is deliberate.** Its
*deployment* requirement — that any deployment pointing one of these applications at data it cannot afford
to lose must first disable it — is established in §10.4 and not restated here; what this section adds is
the **privilege** the initializer implies, and that privilege is not the same in all three editions. A
single all-editions row would read as one grant to make and would be wrong for MVC 3, whose catalog engine
has no principal to grant anything to, so the split below is the whole point of the two rows and the
paragraph that follows them depends on it.

| Construct | Editions | Permission implied |
| --- | --- | --- |
| `Integrated Security=True` | MVC 5, both connection strings [src/MVC5/MvcMusicStore/Web.config:12-13]; MVC 4, catalog [src/MVC4/MvcMusicStore/Web.config:21] | **A Windows identity the SQL instance trusts.** No credential is presented, so the worker-process identity *is* the database principal. There is no application login to grant, and no way to scope the application's rights independently of that identity |
| `Integrated Security=SSPI` | MVC 4, credential store [src/MVC4/MvcMusicStore/Web.config:15] | The same requirement, spelled differently |
| `DropCreateDatabaseIfModelChanges<MusicStoreEntities>`, against a **SQL Server** engine | MVC 5 [src/MVC5/MvcMusicStore/Models/SampleData.cs:9], MVC 4 [src/MVC4/MvcMusicStore/Models/SampleData.cs:9] | **Rights to drop and create the database and its schema**, held by the running application, exercised at startup. This is the widest *SQL* privilege the repository demands anywhere |
| The same initializer [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Models/SampleData.cs:9], against **SQL Server Compact** | MVC 3, catalog only | **File-system rights, not a SQL grant.** MVC 3's catalog is a `.sdf` file reached through `System.Data.SqlServerCe.4.0` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:57-58] — an embedded engine with **no server, no login and no database principal**, so there is no `db_owner` membership and no server-level permission to grant. Dropping and re-creating "the database" is deleting and re-creating the file, so the requirement is **create-file *and* delete-file** rights in `App_Data` for the worker-process identity — §11.1's create-file row plus the delete the initializer's drop step performs |
| `((IObjectContextAdapter)context).ObjectContext.CreateDatabase()` | MVC 4 [src/MVC4/MvcMusicStore/Filters/InitializeSimpleMembershipAttribute.cs:37] | **`CREATE DATABASE`**, explicitly, in code, at first use |
| `WebSecurity.InitializeDatabaseConnection(…, autoCreateTables: true)` | MVC 4 [src/MVC4/MvcMusicStore/Filters/InitializeSimpleMembershipAttribute.cs:41] | **`CREATE TABLE`** in the credential store, at first use |
| Administrator provisioning at startup | MVC 5 [src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:18], [src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:21-44]; MVC 4 [src/MVC4/MvcMusicStore/App_Start/AppConfig.cs:18], [src/MVC4/MvcMusicStore/App_Start/AppConfig.cs:21-37] | Write access to the credential store's user and role tables, again held by the running application at startup rather than by an operator |
| The inherited machine-level ASP.NET SQL membership and role providers | MVC 3, credential store [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:15], [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:26-28] | **Not stated as a requirement, deliberately.** The providers and their connection string are the host's, not the repository's (§10.2), so whether the grant needed is "connect and write to a provisioned store" or "`CREATE DATABASE` at first request" cannot be read out of the repository. §13.2 item 2 carries it as an open verification item; no privilege is assigned here by inference |

**The single most consequential line in this section:** the application's own runtime identity holds schema-changing authority in every edition — but not the same *kind* of authority, and the difference decides what there is to grant. **In MVC 5 and MVC 4 it is database-owner-level rights on a SQL Server engine**, with MVC 4 requiring server-level `CREATE DATABASE` on top. **In MVC 3's catalog it is file-system delete-and-create rights on a `.sdf`**, because SQL Server Compact has no principal to grant a SQL right to; and **MVC 3's credential store is deliberately left open** — whatever privilege its inherited machine-level provider needs is a property of the host, and §10.2 keeps that question unresolved rather than assigning a right the repository does not declare. In every case there is no separation whatsoever between the identity that serves requests and the identity that changes schema. The target-state separation — a deployment-time migration step under a distinct principal, with the runtime identity holding least-privileged data access — is owned by deliverable 06 and is not specified here.

### 11.3 Application-pool identity — the constraint LocalDB imposes

`(LocalDb)\MSSQLLocalDB` [src/MVC5/MvcMusicStore/Web.config:12-13] and `(LocalDB)\v11.0` [src/MVC4/MvcMusicStore/Web.config:13], [src/MVC4/MvcMusicStore/Web.config:19] name **automatic LocalDB instances**, which are scoped to a Windows user profile: an instance belongs to the user who owns it, and a process running under a different identity does not see it. Combined with section 11.2's integrated authentication, this pins the requirement tightly:

**The worker-process identity must be a Windows user account that owns a LocalDB instance of the named generation and is trusted by it.** A service identity without a loaded user profile has no automatic instance to connect to. That is consistent with — and explains — the only workflow the repository documents: open the solution in Visual Studio and press F5 [src/MVC5/README.md:109-111], [src/MVC4/README.md:163-164], i.e. run under the developer's own identity via IIS Express, which the project files declare as the host [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:18], [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:18]. **No edition documents or configures a full-IIS deployment**, and the repository contains no application-pool configuration of any kind.

Two related settings complete the picture at the web tier. MVC 5 leaves `IISExpressAnonymousAuthentication` and `IISExpressWindowsAuthentication` empty [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:20-21] and sets `NTLMAuthentication` to `False` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:286], and its `system.web` authentication mode is `None` [src/MVC5/MvcMusicStore/Web.config:32] because authentication is handled by OWIN middleware instead. So **the browser-facing tier is anonymous while the data tier is Windows-authenticated**: the identity reaching SQL Server is always the application's, never the caller's. That is what makes the wide rights of section 11.2 a property of the deployment rather than of any user.

### 11.4 Build-host permissions

Smaller, but they are real prerequisites for a build to succeed:

- **Write access to the solution directory**, because MSBuild-integrated restore writes packages to `$(SolutionDir)\packages` [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:31] and every edition writes output to `bin\` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:33], [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:41].
- **Package-restore consent granted on the host**, because `RequireRestoreConsent` defaults to `true` [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:13] and passes `-RequireConsent` to the restore command [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:51]. A host that has not opted in fails the restore.
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
| `src/MVC4/MvcMusicStore/.nuget/NuGet.targets` — `RestorePackages` target [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:76-83] | Executes `$(RestoreCommand)` [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:53] under two OS-conditioned `Exec` tasks, one of which logs standard error as error [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:80-82] | **Live for MVC 4** via `<RestorePackages>true</RestorePackages>` [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:24] and the `BuildDependsOn` prepend [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:57-60]. **Inert for MVC 5**, whose import is guarded (section 5.2) |
| `src/MVC4/MvcMusicStore/.nuget/NuGet.targets` — `CheckPrerequisites` target [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:69-74] | Errors if the committed client is missing [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:71]; sets `VisualStudioVersion` in the environment [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:72]; would download the client [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:73] | Live for MVC 4. The download branch is **switched off** [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:16], so the committed binary is mandatory |
| `src/MVC4/MvcMusicStore/.nuget/NuGet.targets` — `BuildPackage` target [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:85-92] | `nuget pack` of the project [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:54], appended to `BuildDependsOn` when enabled [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:63-66] | **Off** — `BuildPackage` defaults to `false` [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:10]. A packaging capability exists and is unused |
| `src/MVC4/MvcMusicStore/.nuget/NuGet.targets` — the non-Windows property branch [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:34-39], [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:47] | Invokes the client through `mono --runtime=v4.0.30319` | Present in the committed file. Notable only because it is the repository's sole acknowledgement that a non-Windows build host might exist |
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
| **Build status** | **BLOCKED pending a Windows verification run** (§1.2). Established and unchanged: it **cannot build from a clean checkout** (§5.1 to §5.3). One post-freeze Windows observation is recorded in §3.2 and does not change this status | **FAILS as committed**, both solutions, before compilation (§3.1, §6.1, §6.4); builds only under the host-side overrides of §6.3, exercised once post-freeze and recorded in §3.2 with the defects left unrepaired | **COMPILES under Mono** with three unresolved-reference warnings (§3.1, §7.1); on Windows it is **toolchain-dependent** on two machine-wide prerequisites and a `v10.0` targets path, all properties of the host (§7.2, §3.2) |
| Target framework | `v4.8` (§4.1) | `v4.5` (§4.1) | `v4.0` (§4.1) |
| Solution to build | `src/MVC5/MvcMusicStore.sln` | `src/MVC4/MvcMusicStore.sln` — **not** `src/MVC4/MvcMusicStore/MvcMusicStore.sln` (§6.4) | `src/MVC3/MvcMusicStore-Completed/MvcMusicStore.sln` |
| Restore required before build | **Yes, mandatory** — nothing committed (§8) | **Yes, and as committed it cannot succeed** — the import carrying the restore target fails first (§6.1); the hint paths are satisfied only by a restore driven through `src/MVC4/MvcMusicStore.sln` into `src/MVC4/packages` (§6.3) | No, for its single hinted reference (§7.1) |
| Machine-wide product install | none | none | **ASP.NET MVC 3 Tools Update** + **SQL Server Compact 4.0** (§7.2, §10.1) |
| Declared web host | IIS Express, plain HTTP (§4.3) | IIS Express, plain HTTP (§4.3) | **VS Development Server — no longer ships** (§4.3, §7.3) |
| Catalog store | LocalDB `MSSQLLocalDB`, file-attached | LocalDB `(LocalDB)\v11.0` as committed — unavailable under its own documented toolchain (§10.3) | **SQL Server Compact 4.0**, `.sdf` created on first run, none committed |
| Credential store | LocalDB, same instance, separate Identity 1.0 database | LocalDB, same instance, separate SimpleMembership database | **A SQL Server instance**, inherited from machine configuration — *unverified* (§10.2) |
| Schema script shipped (all three cells established in §10.5, which is the owner; this row states no verdict of its own) | **none** (§10.5) | two, byte-identical, **neither runnable** (§10.5) | one, runnable given a database named `MvcMusicStore` (§10.5) |
| Administrator provisioned at startup | yes, `async void` — failures unobservable (§10.4) | yes, failures surfaced as `InvalidOperationException` (§10.4) | **no** — role never created (§10.4) |
| Runtime database rights required | drop and create the database and its schema on the SQL Server engine (§11.2) | the same, **plus server-level `CREATE DATABASE`** and `CREATE TABLE` in the credential store (§11.2) | **create-file *and* delete-file** in `App_Data` for its `.sdf` — a filesystem right, **not** a SQL grant (§11.2); whatever its inherited machine-level credential store requires is the host's and is **unverified** (§10.2, §13.2 item 2) |
| Tests | none | none | none |
| Deployment automation | none | none | none |
| Views compile-checked | no | no | no |

### 13.2 Open verification items

Three items are not settled by repository evidence alone, and none of them is settled here by inference. One has had its run performed and recorded as supplementary observation — the post-freeze Windows observation of §3.2 — which does not close it, for the reason §1.2 gives. **All three are open**, and each row names what remains and who carries it until then.

| # | Open item | What must be done | Carried by |
| --- | --- | --- | --- |
| 1 | **MVC 5's build status is carried as blocked** — the sole migration source, blocked pending a Windows verification run (§1.2) | The gate closes by an approved revision of the governing plan, not by an agent's run. The evidence available to that decision is §3.2's post-freeze observation, together with the one field it leaves unfilled — it built **Debug only**, so §4.4's Release publish path is unexercised. A run that closes the item records tool versions, the restore source used, **both** configurations, the per-edition outcome, and the warning and error counts | This document owns the status; deliverable 03 gates its first workstream on the run; deliverable 07 carries it as a risk while it is open |
| 2 | **MVC 3's credential store is inherited, not declared** | Verify the actual machine-level ASP.NET SQL provider and its connection string on the supported Windows runtime (§10.2) before the requirement is stated as final. §3.2 does not touch this: it observed builds, and MVC 3's runtime was unavailable on that host for want of any SQL Server instance | This document, on re-verification; deliverable 07 if it affects scope |
| 3 | **The effective package source is a property of the build host, not the repository** | Every run states the source it used, because the repository declares none — §3.2 records `nuget.org` for the observation it carries. Recording it per run is the mitigation, not a fix | Deliverable 02 §6 owns the finding; this document requires the field in §3.2 |

**Item 1 is a plan-revision item rather than a missing measurement**, and item 3 travels with whichever run the plan's owner accepts. **Item 2 needs a running MVC 3 rather than a build**, and can be discharged on a host that carries the two machine-wide products of §7.2 and a SQL Server instance.

### 13.3 Where each consequence is owned

| Fact recorded here | Consequence owned by |
| --- | --- |
| MVC 5's build status — blocked pending a Windows verification run — and its clean-checkout restore precondition | 07 (the open risk while the status is blocked), 03 (first workstream gate) |
| MVC 4's two configuration defects and the stale solution | 08 (debt severity and owner), 12 (blocker classification) |
| MVC 3's machine-wide product requirements and retired provider | 12 (no-successor classification), 02 §4.1 (dependency inventory) |
| Windows, Visual Studio and LocalDB coupling | 11 (cloud readiness), 06 (hosting target) |
| The destructive initializer's deployment requirement, established in §10.4, with the per-edition privilege it implies in §11.2 | 08 (debt framing), 05 (replacement schema lifecycle), 06 (deployment-time migration sequence) |
| Runtime identity holding DDL rights | 06 (principal separation), 09 (security posture) |
| The absent regression baseline | 03 (test authoring precedes the port), 05 (suite design), 07 (first-order risk) |
| Absent deployment automation | 03 (CI/CD as a net-new workstream), 06 (deployment model) |
| The unconfigured restore source | 02 §6 (the finding and the §3.3 correction), 04 (target-state remedy) |
| Committed database binaries and `packages/` payloads | 08 (repository hygiene) |

### 13.4 The two finding-register rows this deliverable discharges — F-09-16 and F-09-17

§13.3 routes *outward*: a fact recorded here, and the deliverable that owns its consequence. Two routes run *inward*, and they are recorded here so each link is checkable from the other end as well. **[Deliverable 09](09-security-assessment.md)'s finding register assigns exactly two rows to this document — `F-09-16` and `F-09-17`** — and both land on the same requirement, because in both cases what is unknown is a property of the host rather than of the checkout. 09 §8.2 item 1 states the consequence in the direction that matters here: the host verification this document owns is a **precondition**, not a formality.

| Register row | What it states | Where this document discharges it |
| --- | --- | --- |
| `F-09-16` | No credential store is declared, so password, lockout and storage policy are properties of the host and cannot be assessed from the repository | **§10.1 and §10.2.** §10.1 separates MVC 3's catalog store from its credential store and shows the second declared nowhere in the application; §10.2 states the negative that *is* established — **MVC 3 declares no membership store of its own**, on `<roleManager enabled="true" />` with no provider element and no store connection string [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:15] — and then draws the boundary: what the store resolves *to* is unestablished and **may not be reported as a final requirement, in this document or any other**. §10.2's seven-field table is the specification that would close it, and §13.2 item 2 carries it open until a run fills those fields |
| `F-09-17` | Every account the edition registers is created with the same two hard-coded password-recovery literals, and the exposure is conditional on the host provider's own default posture | **§10.2, fields 2 and 3.** The literals sit in the fourth and fifth argument positions of the registration call [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/AccountController.cs:94]; **their values are withheld by 09 under its §1.3 and are not republished here.** What decides whether a fixed recovery pair is usable at all is the provider in force — its password-retrieval and question-and-answer posture — and that provider is inherited, not declared. §10.2 field 2 requires the machine-level `<membership>` provider in force to be recorded by `type`, `name`, `connectionStringName` and the merged-configuration file the value came from, and field 3 requires the same four for the role provider, which may not be the same store. Until those fields exist, no recovery-path property of MVC 3 is a stated requirement of this document. `F-09-17`'s other consumer is 05, which owns the target-state remedy |

**Neither row acquires anything by being cited here.** The severity, the security framing and the remediation ownership are 09's; what this document adds is the *precondition* — the run that must happen before any statement about MVC 3's credential handling can be final — and the fields that run must record. The traversal terminates in both directions: from 09's register, each row's Consumers cell names `10`, and this sub-section names both rows by identifier; from here, `F-09-16` and `F-09-17` resolve to rows in 09 §8 whose Consumers cells name `10`. Where they disagree, 09 §8's cell governs, per 09 §8.3.

---

## Appendix A — Reproducibility

Every repository-wide figure in this document, with the command that reproduces it, plus the commands behind the three strands of §3. Commands are given in POSIX form; on a Windows host, `git ls-files` and `git grep` behave identically and the pipeline utilities are available through Git for Windows. The one exception is the strand A host-inventory row, whose probes are PowerShell because they interrogate a Windows host's product inventory rather than the repository.

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
| Their split by location and ignore rule (§11.1) | **10** under an application's `App_Data` — 6 MVC 4, 4 MVC 5 — each matching `App_Data/` [.gitignore:32]; **4** under `src/MVC3/MvcMusicStore-Assets/Data/`, matching **no** ignore rule | `for f in $(git ls-files \| grep -iE '\.(mdf\|ldf)$'); do printf '%s -> ' "$f"; git check-ignore --no-index -v "$f" \|\| echo 'no rule'; done` — `--no-index` is required, because a tracked file returns exit 1 from `git check-ignore` without it whether or not a rule matches |
| MVC 3-Completed `App_Data` files (§10.1) | **0** | `git ls-files 'src/MVC3/MvcMusicStore-Completed/*' \| grep App_Data \| wc -l` |
| `ASPNETDB` references (§10.1) | **0** | `git grep -il 'ASPNETDB' -- 'src/' \| wc -l` |
| MVC 3 membership/provider declarations (§10.1, §10.2) | only `<roleManager enabled="true" />` | `git grep -n -iE '<membership\|<roleManager\|LocalSqlServer\|<profile' -- 'src/MVC3/MvcMusicStore-Completed/*'` |
| Schema scripts (§10.5) | **3** | `git ls-files '*.sql'` |
| The two MVC 4 scripts are byte-identical (§10.5) | same SHA-256 `D577AAA5…182C9D`, both 153,594 bytes | `sha256sum src/MVC4/MvcMusicStore-Create.sql src/MVC4/MvcMusicStore/MvcMusicStore-Create.sql` |
| All three scripts are UTF-16 with a BOM (§10.5) | each begins `FF FE`; the MVC 3 asset script begins `FF FE FF FE` | `for f in $(git ls-files '*.sql'); do printf '%s ' "$f"; head -c 4 "$f" \| xxd -p; done` |
| Seed rows and seed file size, **all three editions**, one command for all nine figures (§10.4) | MVC 3 — **430** physical lines, **10** genres, **149** artists, **246** albums. MVC 4 — **826**, **15**, **303**, **462**. MVC 5 — **826**, **15**, **303**, **462** | `for f in src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Models/SampleData.cs src/MVC4/MvcMusicStore/Models/SampleData.cs src/MVC5/MvcMusicStore/Models/SampleData.cs; do printf '%s lines=%s genres=%s artists=%s albums=%s\n' "$f" "$(wc -l < "$f")" "$(grep -c 'new Genre' "$f")" "$(grep -c 'new Artist' "$f")" "$(grep -c 'new Album' "$f")"; done` — one row of output per file, so no figure rests on a command run against a different edition's file. Line counts are the **physical**-line metric of deliverable 01 §2.4, not the non-blank metric, and they are the `wc -l` line-feed count: none of the three files ends with a newline, so `awk 'END{print NR}'` reports one more for each — 827, 827 and 431 — while the last line is real |
| Initializer registration sites (§10.4) | **5** occurrences, of which **4 register `SampleData`** — MVC 5 twice, MVC 4 once, MVC 3 once. The fifth *disables* the initializer for `UsersContext` in MVC 4 | `git grep -n 'SetInitializer' -- 'src/MVC5' 'src/MVC4' 'src/MVC3' \| grep -v '/packages/'` |
| Publish profiles (§12.1) | **0** | `git ls-files '*.pubxml' '*.pubxml.user' \| wc -l` |
| Pipeline definitions (§12.1) | **0** | `git ls-files '.github/*' '*.yml' '*.yaml' \| wc -l` |
| Container manifests (§12.1) | **0** | `git ls-files '*Dockerfile*' '*docker-compose*' \| wc -l` |
| Infrastructure definitions (§12.1) | **0** | `git ls-files '*.tf' '*.bicep' \| wc -l` |
| Tracked script files (§12.1) | **6**, all inside the committed MVC 4 package payload | `git ls-files '*.sh' '*.ps1' '*.cmd' '*.bat' '*.psm1'` |
| Cited file extents — the full-range locators of §4.4 and §10.5 | **31** lines in each `Web.Release.config`, **30** in each `Web.Debug.config`, **618** in the MVC 3 asset script, **629** in each MVC 4 copy | `for f in $(git ls-files 'src/*/**/Web.[DR]*.config'); do printf '%s ' "$f"; awk 'END{print NR}' "$f"; done`; for the UTF-16 scripts, `for f in $(git ls-files '*.sql'); do printf '%s ' "$f"; iconv -f UTF-16 -t UTF-8 "$f" \| awk 'END{print NR}'; done`. `awk 'END{print NR}'` rather than `wc -l` deliberately: four of the six XDT files — both MVC 4 and both MVC 3 transform files — end **without** a final newline, so `wc -l` under-reports their extent by one while the last line is real and is the closing `</configuration>` |
| Seed file size, and the one metric it is measured by (§10.4) | **826 LF / 827 content lines** (MVC 5, MVC 4); **430 LF / 431 content lines** (MVC 3) | `wc -l < src/MVC5/MvcMusicStore/Models/SampleData.cs` → `826`, and `awk 'END{print NR}' src/MVC5/MvcMusicStore/Models/SampleData.cs` → `827`. `wc -l` counts line feeds and the file carries no terminal newline, so the two differ by one and the row states both. **Both figures are the physical-line metric** of deliverable 01 §2.4 — the metric used for duplication comparison — and neither is the non-blank sizing metric deliverable 07 estimates against. The two metrics are never combined in one figure or one sentence, here or in §10.4, which quotes the `826` form and labels it physical |
| The observed Mono run (§3.1) | MVC 5 compile failure un-restored; both MVC 4 solutions fail before compilation; MVC 3 compiles with three unresolved-reference warnings | `xbuild <solution>` in Debug under Mono 6.8.0.105 on Linux, for each of the four tracked solutions |
| The post-freeze Windows observation (§3.2) | restore exit `0` and build exit `0` for all three editions, Debug, with `0` warnings and `0` errors; the three assemblies and their byte sizes are in §3.2 | `nuget restore <solution> -NonInteractive`, then `MSBuild.exe <solution> /t:Build /p:Configuration=Debug /nologo /v:m /nr:false`, MSBuild located via `vswhere -latest -products '*' -requires Microsoft.Component.MSBuild -find 'MSBuild\**\Bin\MSBuild.exe'` and invoked by full path. **§3.2 is the single record of that run**; it is supplementary to the carried status of §1.2 and does not discharge the AAP's gate |
| MVC 4's build under the host-side overrides (§3.2, §6.3) | build exit `0` **only with the overrides**; the committed configuration is unrepaired and a plain `MSBuild src/MVC4/MvcMusicStore.sln` still fails during evaluation (§3.2, §6.1) | The same two commands on `src/MVC4/MvcMusicStore.sln`, plus `/p:SolutionDir=<repo>\src\MVC4\MvcMusicStore\` — unquoted, trailing backslash — and `/p:RestorePackages=false`. The overrides are host-side only and are never written into a tracked file |
| No uncommitted tracked change in the checkout at the moment of commit — an acceptance check, not an authoring-time observation | **must report empty output**, and it is blind to the ignored trees recorded below, which is why the check needs four commands and not this one | `git status --porcelain` |
| Every change this assessment committed is an addition under `docs/modernization/` | **13 lines, every one an `A` under `docs/modernization/`**; no `M` and no `D` against any pre-existing file, and nothing added outside that directory | `git diff --name-status ea2552d6eda7c20e9477a512e5c615665618cf35..HEAD`, the same form used in §1.4 and at the foot of this appendix, so the file carries exactly one. The **start** is pinned and verifiable — `ea2552d6…` is the last commit before this engagement, `git log -1 ea2552d6…` naming it "Merge pull request #9 …". The **end** is `HEAD` because **no document can contain the hash of the commit that adds it** — that hash exists only once the commit does, which is after these deliverables reach final content, so any literal printed here would name an earlier commit and describe a range that leaves this work out. A reader who wants a range pinned at both ends substitutes this branch's tip after checkout, e.g. `git rev-parse HEAD`, and **the substitution changes none of the four expected values**: thirteen rows, thirteen `A` rows, nothing that is not an `A`, and nothing outside `docs/modernization/` |
| The MVC 4 failure classes (§6.1, §6.4) | the unconditional import names an absent path — the `MSB4019` class, "the imported project was not found"; the stale solution names an absent project — the `MSB3202` class | Both paths are shown absent by the `test -d src/MVC4/.nuget` and `test -f src/MVC4/MvcMusicStore/MvcMusicStore/MvcMusicStore.csproj` rows above, which is what makes each failure class platform-independent rather than a property of the evidence host |

**Neither build record is set out in full in this table**, because neither is a repository-wide count: the Mono attempts, their host, their restore states and their outcomes are in §3.1, and the post-freeze Windows observation — host, tool inventory, commands, per-edition results and qualifications — is in §3.2. The two rows above name the commands so that each record is re-runnable, and the status the document carries is §1.2's.

**Repository state, and the generated trees an in-place run leaves behind.** No tracked repository file was modified, added or deleted in producing this document. What a tracked-file check cannot show is recorded here rather than summarised as a clean tree: **restore and build runs performed against this checkout wrote eight ignored trees into it** — a `bin` and an `obj` beside each of the three projects, plus a restored `packages` tree under `src/MVC4` and another under `src/MVC5`. Their removal is **not** claimed here as a discharged act, and the omission is deliberate: §3.2's post-freeze run was performed in place as well and writes the same set again, so this is a **standing requirement** rather than one that can be closed once. **Whoever commits this assessment runs the two ignored-aware commands below and removes whatever they report**, because bare porcelain reports a clean tree either way — that is the check, and it is the committer's to discharge, not a state this appendix asserts. The figures below are as measured while the trees existed:

| Removed tree | Files | Bytes | The ignore rule that hid it |
| --- | --- | --- | --- |
| `src/MVC3/MvcMusicStore-Completed/MvcMusicStore/bin` | 5 | 1,969,163 | `[Bb]in/` [.gitignore:2] |
| `src/MVC3/MvcMusicStore-Completed/MvcMusicStore/obj` | 7 | 361,948 | `[Oo]bj/` [.gitignore:1] |
| `src/MVC4/MvcMusicStore/bin` | 51 | 11,290,344 | `[Bb]in/` [.gitignore:2] |
| `src/MVC4/MvcMusicStore/obj` | 7 | 649,118 | `[Oo]bj/` [.gitignore:1] |
| `src/MVC4/packages` | 231 | 29,937,044 | `Packages/` [.gitignore:33] — matches at any depth, and matches this lowercase directory only where the checkout is case-insensitive |
| `src/MVC5/MvcMusicStore/bin` | 52 | 20,466,273 | `[Bb]in/` [.gitignore:2] |
| `src/MVC5/MvcMusicStore/obj` | 7 | 656,819 | `[Oo]bj/` [.gitignore:1] |
| `src/MVC5/packages` | 167 | 48,979,685 | `Packages/` [.gitignore:33] — the same rule, under the same case dependency |
| **Total** | **527** | **114,310,394** | — |

**The fourth column is reproducible, and it does not name the rule a reader would guess.** `packages/*` [.gitignore:15] looks like the rule that covered the two restored package trees, and it did not cover them: a pattern with an interior separator is anchored to the directory holding the `.gitignore` — the repository root here — so it reaches a root-level `packages/` and nothing nested. The rule that matched them is `Packages/` [.gitignore:33], whose only separator is trailing, which matches a directory of that name at any depth. Deliverable 04 §A.6 reads these rules the same way, against the same paths. `--no-index` is required in the probe rather than merely tidy: without it `git check-ignore` exits 1 with no output for a **tracked** path whether or not a rule matches it, which is exactly how the committed payload at `src/MVC4/MvcMusicStore/packages` behaves under the plain form:

```bash
git check-ignore -v --no-index src/MVC4/packages/x src/MVC5/packages/x \
    src/MVC5/MvcMusicStore/bin/x src/MVC5/MvcMusicStore/obj/x
# -> .gitignore:33:Packages/   src/MVC4/packages/x
#    .gitignore:33:Packages/   src/MVC5/packages/x
#    .gitignore:2:[Bb]in/      src/MVC5/MvcMusicStore/bin/x
#    .gitignore:1:[Oo]bj/      src/MVC5/MvcMusicStore/obj/x

git config core.ignorecase   # -> true, on the checkout these runs were performed on
```

**That case dependency makes the residue-hiding behaviour recorded above host-dependent, which is why it belongs in this document rather than in a note about git.** `Packages/` matches the lowercase `packages` directories only because `core.ignorecase` is `true` on this Windows checkout; on a case-sensitive host — a Linux build agent or container image — it matches neither of them, and `packages/*` [.gitignore:15] still reaches only the repository root, so **neither restored package tree would be ignored at all there**. A reader who performs the same restore on such a host sees both trees in ordinary `git status` output as untracked directories, and the failure mode inverts: not a clean tree that is not clean, but generated payload exposed to a careless `git add`. What holds on either host is the observation this appendix rests on — on the checkout where these runs happened the paths *were* ignored, which is precisely why bare porcelain reported a clean tree while the payload sat in it — and that is why the hygiene rule below is stated in terms of the ignored-aware commands rather than porcelain.

Two of the eight are restored package trees, and the MVC 4 one carries a build fact worth keeping in this appendix: a restore driven from `src/MVC4/MvcMusicStore.sln` fills `src/MVC4/packages`, which is **not** the tracked payload at `src/MVC4/MvcMusicStore/packages` (§6.2). A restore in this repository therefore writes untracked payload even for the editions whose packages are committed.

**Why the check needs four commands.** Every one of the eight trees matched an ignore rule on this checkout, so `git status --porcelain` and the baseline-to-HEAD diff both reported a clean tree for as long as the trees existed. Neither command is wrong; both are blind to ignored content, which makes them blind to precisely the artefacts a restore or a build produces:

```bash
git status --porcelain                                                  # must report 0 lines
git status --porcelain --ignored                                        # must report 0 lines: no bin/, obj/ or packages/ tree left behind
git clean -ndX                                                          # must report nothing: nothing ignored remains to be removed
git diff --name-status ea2552d6eda7c20e9477a512e5c615665618cf35..HEAD
# must report exactly 13 rows, every one an 'A' under docs/modernization/, and nothing else
#    left side: the immutable evidence revision defined in §1.4; right side: the
#    delivery commit the reviewer has checked out, per the Appendix A row above
```

**Each line above states the result the check must produce.** Three of the four are properties of the working tree at the moment of commit and belong to whoever commits; only the fourth is a property of the commit range and is recorded as observed in the table above.

**The hygiene rule this document holds itself and any later run to.** A restore or a build performed for assessment purposes belongs in a **disposable clone outside the authoritative checkout**; where it is run in place, its generated trees are removed and their absence is verified with the two ignored-aware commands above, never with porcelain alone. The rule bound §3.2's post-freeze run, whose restore created `src/MVC5/packages` (§5.1, §5.2) and whose build wrote a `bin` and an `obj` beside each project, and it binds any later verification run the plan's owner commissions. Nothing in the table above is build evidence in its own right — a tree in a working directory records no host, no tool version and no outcome — and the status of MVC 5's build is unchanged by anything in this appendix: **blocked pending a Windows verification run** (§1.2).
