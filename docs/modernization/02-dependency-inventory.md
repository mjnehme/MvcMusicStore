
# 02 — Dependency Inventory

**Deliverable 02 of 13** · MvcMusicStore modernization assessment · *Assessment only — no tracked repository file is modified by this document; §1.3 accounts for the generated content the assessment's restore and build runs left in the checkout, and for its removal.*

Answers two of the user's fourteen requirements: **"Framework and package dependencies"** (analyze) and **"Dependency inventory"** (produce). Together with [01 — Architecture Overview](01-architecture-overview.md) this is a *foundation* document: the other eleven deliverables cite it rather than re-deriving its numbers.

---

## 1. Purpose, scope and authoring contract

### 1.1 What this document is

A complete, verbatim catalogue of every declared dependency of all three shipped editions of MvcMusicStore, as the repository declares them **today**. It covers three classes of dependency, because the NuGet manifests alone do not describe what the applications need:

1. **NuGet package pins** — the 63 `<package>` entries across the three `packages.config` files (§3).
2. **Dependencies NuGet does not resolve** — .NET Framework assemblies, machine-installed products and native providers referenced without any package backing them (§4).
3. **Build-tool and restore-configuration dependencies** — the committed NuGet client and the restore wiring that consumes it (§5, §6).

It then records what the repository does *not* say: no lockfile, no configured package source, no dependency-scanning configuration (§6, §7, §8).

### 1.2 What this document deliberately does not contain

Under the single-owner rule (AAP 0.11.4) each cross-cutting decision is stated in full by exactly one deliverable and cross-referenced by the rest. This document owns the **current** inventory and nothing else. In particular it contains **no target-state package version, no successor version, no target framework and no SDK band** — not once, anywhere. Those belong to:

| Question | Owner |
| --- | --- |
| Target framework, SDK band, per-package migration outcome, target-state `NuGet.config`, target-state lockfile policy | [04 — .NET 8 Migration Strategy](04-dotnet8-migration-strategy.md) |
| Client-library acquisition mechanism and the Bootstrap markup work implied by upgrading it | [05 — ASP.NET Core Migration Approach](05-aspnet-core-migration-approach.md) |
| Hosting target and deployment model | [06 — Azure Hosting Recommendations](06-azure-hosting-recommendations.md) |
| Effort bands and the risk register | [07 — Effort, Risks and Sequencing](07-effort-risks-sequencing.md) |
| Debt framing, severity and ownership of the items below | [08 — Technical Debt Register](08-technical-debt-register.md) |
| Per-edition build outcome and restore posture | [10 — Build and Deployment Requirements](10-build-and-deployment-requirements.md) |
| Which of these packages has no successor, and the serialization consequence | [12 — Migration Blockers](12-migration-blockers.md) |

Where a fact below has a forward consequence, this document names the owning deliverable and stops there.

### 1.3 The no-modification constraint

The user directed *"Do not make code changes initially"*, and the project's attached environment setup instructions independently restate the same gate (*"Do not modify code until assessment and modernization plan are approved"*). Accordingly, and per AAP 0.5.4: **no `packages.config` was edited, no package was added, upgraded, downgraded or removed, and no project file gained or lost a reference.** The repository's dependency graph is byte-identical before and after this document was written. What the assessment nonetheless *wrote* into the checkout is generated, ignored content; it is reported below rather than folded into a clean-tree claim, because an attestation that quietly omits it is worth less than no attestation at all.

Every figure below was obtained by reading files and by read-only `git` queries, with one flagged exception: the advisory evidence in §8.2 required a `nuget restore`, which is a **mutating** command — it downloads package payloads and writes them into the working tree under `packages/`. This document therefore separates its commands into two classes and labels them as such wherever they appear: **read-only evidence commands**, which are safe to run anywhere and are collected in §10.1, and the **mutating audit workflow**, quoted only in §8.2 and repeated in §10.2. That workflow's mutations are of two kinds, and collapsing them into one count is how such a statement goes wrong. **Three of its commands are package-mutating** — the three `nuget restore` invocations, which download payloads and write them into whatever tree they are run in. **Four further operations mutate nothing but the disposable clone and the directory holding it**: the `git clone` that creates the clone under `$SCRATCH`, the redirected log writes inside it, the concatenation the counts are parsed from, and the guarded `rm -rf` that discards it afterwards. A fifth step mutates nothing at all and is the reason the other seven are safe: both copies of the workflow now open by **validating `$SCRATCH`** — set, non-empty, absolute, existing and outside this repository — and both run under `set -euo pipefail` with every path rooted at the clone and no directory change that outlives a command substitution, so a failed clone or a mistyped `$SCRATCH` stops the workflow instead of leaving three `nuget restore` invocations to run in whatever directory the caller was standing in. The rule for the three restores is stated at both sites: run them in a **disposable clone or scratch checkout outside the authoritative repository, never in place against the assessed checkout**. That is how the §8.2 advisory evidence was obtained. The same read-only-versus-mutating distinction governs the future release operations, which are owned by [06 — Azure Hosting Recommendations](06-azure-hosting-recommendations.md); this document adopts the distinction for its own commands and claims none of that deliverable's decisions.

**Generated output did reach the assessed checkout during this assessment, and it has been removed.** Restore and build operations were run in place against this checkout earlier in the assessment, and they left **eight ignored trees** behind — a `bin` and an `obj` beside each of the three editions' projects, plus a restored `packages` tree under `src/MVC4` and another under `src/MVC5` — **527 files and 114,310,394 bytes, as recorded before removal**, because a tree that no longer exists cannot be re-counted. All eight have been removed and their absence verified by the four commands below. No tracked file was modified, added or deleted at any point, and the dependency-graph statement above is unaffected: an ignored payload directory is not a declared dependency. **Those operations are not build evidence, and nothing here treats them as any.** They were unqualified historical restore and build runs whose only recorded effect is the gitignored residue above: [10 — Build and Deployment Requirements](10-build-and-deployment-requirements.md) states that no run behind those trees recorded the fields its evidence requires — tool versions, the restore source used, the configuration built, the per-edition outcome, the warning and error counts — so neither a restore having run nor an output directory having existed says anything about whether an edition builds. Every build outcome, status and evidentiary claim is that deliverable's to make, including MVC 5's, which remains **blocked pending a Windows verification run**; this document adds nothing to it and softens nothing in it. The per-tree record belongs to deliverable 10's appendix and is cross-referenced rather than restated here. Two of the eight are the direct product of the §8.2 restores — `src/MVC5/packages` at 167 files and `src/MVC4/packages` at 231, **398 files and 78,916,729 bytes** of that total — and those two are reproducible from §8.2's own command block, which is why they are the two this document can account for from its own evidence.

The acceptance check for that constraint is therefore **four commands, and no three of them are the check**: a diff against the pre-assessment baseline plus three working-tree checks, two of which are ignored-aware. The rules that excluded the eight trees are `[Oo]bj/` [.gitignore:1], `[Bb]in/` [.gitignore:2] and `Packages/` [.gitignore:33], and the third is the load-bearing one and the least obvious: a pattern whose only separator is trailing matches a directory of that name at **any** depth, and on this checkout — `git config core.ignorecase` reports `true` — it also matches the lowercase `packages` directories, which is why `src/MVC4/packages` and `src/MVC5/packages` were ignored here. `packages/*` [.gitignore:15] does *not* cover them, because a pattern with an interior separator is anchored to the directory holding the `.gitignore` — the repository root. That makes the ignore behaviour recorded here a property of a case-insensitive host: **on a case-sensitive host no rule ignores those two nested lowercase `packages` trees at all**, and a bare porcelain status would have listed them instead of reporting a clean tree. `git check-ignore -v --no-index` reports the rule per path and is in §10.1. Either way the consequence for the checks below is the one that matters: a bare porcelain status and a tracked-file diff both report a clean tree while generated payload sits in it — which is exactly what they did on this checkout for as long as the eight trees existed. Run, against the committed checkpoint:

```bash
# 1 — the tracked diff against the pre-assessment baseline
git diff --name-status ea2552d6eda7c20e9477a512e5c615665618cf35..HEAD
# -> exactly 13 rows, every one an A for a file under docs/modernization/, and no M or D row

# 2 — tracked working-tree state
git status --porcelain             # -> (no output)

# 3 — the same state with ignored content included: the one check the other three are blind to
git status --porcelain --ignored   # -> (no output): no restored packages/, no bin/ and no obj/ anywhere

# 4 — what an ignore-aware clean would remove, listed rather than deleted
git clean -ndX                     # -> (no output): nothing ignored is left to remove
```

Command 1 is a property of the committed history and is the durable evidence: **thirteen `A` rows** and nothing else means no existing file was modified or deleted. Commands 2 to 4 describe a working tree, so each is evidence only of the checkout it is run in — non-empty while uncommitted edits are in flight — and **3 and 4 are the only two that look at ignored content**, which is the half of the claim bare porcelain looked like it was making and was not. A reader who expected `git status --porcelain` to list the new documents is reading an authoring-time working tree rather than the committed result. §10.1 repeats all four with their observed output.

This is a catalogue, not a curation.

### 1.4 Authoring contract, and the absence of user rules

`review_rules` returns exactly **"No user rules provided."** There is therefore no project rule to name, summarize or comply with, and no file forced into scope by one. The absence is not licence to lower the bar; this document is held to enterprise-standard assessment practice and to the following contracts, which stand in place of rules:

- **Repository evidence is primary.** Every as-is claim carries an inline `[<path>:<locator>]` citation placed at the claim, never collected in a trailing reference list. Paths are repository-relative and resolve in the checkout.
- **The path is written out in full at every citation, with no exception.** No citation in this document inherits its path from a neighbouring table row, a neighbouring cell, a table heading or a preceding paragraph, so any single citation can be checked in isolation without reading what surrounds it. That includes the per-row `Manifest citation` column of the three pin tables in §3, each cell of which carries its manifest path in full. Nothing in this document relies on inheritance, and no locator appears anywhere without the path it belongs to.
- **A tracked binary is cited by its identity, because it has no line to point at.** The repository's only committed binary dependency is `src/MVC4/MvcMusicStore/.nuget/NuGet.exe`. Its locator is its identity — size, `FileVersion`, assembly version and `ProductVersion` — stated in the sentence wherever the file is cited, and reproduced by the command block in §5.1.
- **A repository-wide claim carries its reproducing command** next to the claim, because a count or an absence has no single line to point at. That is the stronger form of evidence: a reader can re-run it.
- **Exact versions only.** Every version string below is transcribed character-for-character from the manifest that declares it. No ranges, no rounding, no "or later". `1.0.0.0` is not `1.0.0`, and `1.0` is not `1.0.0`.
- **The Technical Specification is secondary.** Sections of it may be cited *alongside* repository evidence, never instead of it. §6.2 below records a place where the specification and the repository disagree, and resolves it in the repository's favour.

Markdown line length follows the corpus convention recorded in [`README.md` § 10 Markdown line-length convention](README.md#10-markdown-line-length-convention).

**Reproducing the commands on this host.** The canonical commands quoted throughout are the POSIX forms fixed by AAP 0.11.3. The verification host for this assessment is Windows; they were executed through the Git-for-Windows `bash` bundled on the host, from the repository root, and the outputs quoted are the outputs observed there.

---

## 2. Inventory at a glance

| Measure | Value | Evidence |
| --- | --- | --- |
| NuGet package pins, all editions | **63** | `git ls-files '*packages.config' \| grep -v '/packages/' \| xargs grep -h '<package ' \| wc -l` → `63` |
| — MVC 5 | 28 | [src/MVC5/MvcMusicStore/packages.config:3-30] |
| — MVC 4 | 29 | [src/MVC4/MvcMusicStore/packages.config:3-31] |
| — MVC 3 | 6 | [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/packages.config:3-8] |
| Distinct registries | 1 (`nuget.org`) | every pin is a public identifier; see §2.1 |
| Version ranges or floating versions | 0 | every `version=` attribute is a single exact release |
| Lockfiles | **0** | `git ls-files \| grep -E '(^\|/)packages\.lock\.json$'` → no output at any depth (§7.1) |
| Configured package sources | **0** | no `packageSources` element anywhere (§6) |
| Committed NuGet clients | 1 | [src/MVC4/MvcMusicStore/.nuget/NuGet.exe:630,784 bytes] — `FileVersion` and assembly version `2.0.30828.5`; `ProductVersion` `2.0.40001`. Identity is the locator for a binary; read it with `(Get-Item 'src/MVC4/MvcMusicStore/.nuget/NuGet.exe').Length` and `[System.Diagnostics.FileVersionInfo]::GetVersionInfo((Resolve-Path 'src/MVC4/MvcMusicStore/.nuget/NuGet.exe'))` (§5.1) |
| Tracked files under committed `packages/` trees | **215** | `git ls-files \| grep -c '/packages/'` → `215` (§7.2) |
| Dependency-scanning configuration | **0** | no `.github/`, no Dependabot or Renovate config, no analyzer package (§8.2) |
| Pins named by NuGet's restore audit | **14 of 63** | A **dated observation**, not a repository property: on the runs of §8.2 restore emitted `NU1902`/`NU1903` advisory warnings against those pins and still exited `0`. The tally — **43 warnings (9 high, 34 moderate), 24 distinct advisories** — and the choice of pins named both belong to the advisory database on the day: NuGet 6.11.1.2 against nuget.org, snapshot `2026.08.27.05.48.31` / `2026.08.28.11.48.36`, full provenance in §8.2 and all 43 warnings retained in §8.2.1. What the repository fixes is the pin set those warnings landed on: 8 identifiers, 14 (package, version) pairs |

### 2.1 Registry and provenance statement

**All 63 pins are public package identifiers verifiable on nuget.org.** Every one declares a single exact version with no range, no wildcard and no floating component. **Nothing in the repository indicates an internal, private or vendored-feed package**: every identifier is a well-known public one, and no manifest, project file or configuration file names a private feed.

Two qualifications belong with that statement rather than after it, and both are consequential:

- The claim above is about the **identifiers**, which are public. It is *not* a claim about which feed a restore actually contacts — the repository configures no source at all, so the effective source set is unknowable from the repository. §6 states that finding in full.
- The 63 pins are a **flat, exact list of every package installed into each project** — transitive entries included, each at a single exact version — because that is what `packages.config` records. What the format does *not* record is which package pulled which in, a content hash for any entry, or any binding on where an entry resolves from. Restore is therefore exact about **what** and silent about **provenance**. §7 states that finding in full.

### 2.2 How the three manifests differ in kind, not just in content

| | MVC 5 | MVC 4 | MVC 3 |
| --- | --- | --- | --- |
| Pins | 28 | 29 | 6 |
| `targetFramework` attribute | on all 28, value `net45` | on all 29, value `net45` | **absent from all 6** |
| Project's target framework | `v4.8` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:16] | `v4.5` [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:16] | `v4.0` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:15] |
| Manifest and project agree? | **No** — manifest says `net45`, project says `v4.8` | Yes | No attribute to agree or disagree |
| Pins whose payload the project references by `HintPath` | 22 of 28 | 20 of 29 | 1 of 6 |
| `packages/` payload committed to source control | none | 169 files, 29 folders | 46 files, 6 folders |
| MSBuild-integrated restore wired | import present but **conditional** [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:295] | **unconditional** import [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:360] with `<RestorePackages>true</RestorePackages>` [:24] | not wired |
| `.nuget` folder present | **no** — although the solution declares one (§5.2) | yes — three tracked artifacts [src/MVC4/MvcMusicStore/.nuget/NuGet.Config:1-6], [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:1-143], [src/MVC4/MvcMusicStore/.nuget/NuGet.exe:FileVersion 2.0.30828.5, 630,784 bytes] | no |
| `packages/` payload committed to source control (counts and their reproducing commands in §7.2) | none | 169 files, 29 folders | 46 files, 6 folders |

The MVC 4 cell cites the folder's contents rather than the folder, because a directory has no line to point at and the claim is about what is tracked inside it. Two of the three are text files cited at their full extent; the third is a binary, so its locator names the evidence form — the version and size a file inspection reports — in the manner §5.1 uses for the same artifact. The set is exhaustive, and the command is the evidence:

```bash
git ls-files 'src/MVC4/MvcMusicStore/.nuget/*'
# -> src/MVC4/MvcMusicStore/.nuget/NuGet.Config
#    src/MVC4/MvcMusicStore/.nuget/NuGet.exe
#    src/MVC4/MvcMusicStore/.nuget/NuGet.targets
git ls-files 'src/MVC5/MvcMusicStore/.nuget/*' 'src/MVC3/MvcMusicStore-Completed/MvcMusicStore/.nuget/*'
# -> (no output: neither other edition tracks a .nuget folder)
```

The three editions are therefore not three instances of one dependency-management approach; they are three different approaches, and a single statement about "how MvcMusicStore restores packages" would be wrong for at least one of them. Deliverable 10 owns the build consequence of that.

---

## 3. NuGet package pins

Each table lists every `<package>` entry of one manifest, **in the order the file declares them**, so that a reviewer can diff the table against the file line by line. The `Manifest citation` column is the per-row citation, written out in full as `[<repository-relative manifest path>:<line>]` in every cell, so each row can be checked without reading the heading above it. Registry is `nuget.org` for every row, per §2.1.

`Purpose` states what the package delivers to *this* repository — which assemblies the project actually references from it, or that it delivers content rather than an assembly. It is not a general description of the package.

### 3.1 MVC 5 — 28 pins

Manifest: **[src/MVC5/MvcMusicStore/packages.config:3-30]** — 31 lines end to end, with all 28 `<package>` entries on lines 3 to 30. Verified with `awk 'END{print NR}' src/MVC5/MvcMusicStore/packages.config` → `31` and `grep -n '<package ' src/MVC5/MvcMusicStore/packages.config | sed -n '1p;$p'` → first pin at `3`, last at `30`.

| Registry | Package | Version | Purpose | Manifest citation |
| --- | --- | --- | --- | --- |
| nuget.org | Antlr | `3.4.1.9004` | Parser runtime; the project references `Antlr3.Runtime` from it [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:108-111]. Present as the minification stack's dependency, not called from application code | [src/MVC5/MvcMusicStore/packages.config:3] |
| nuget.org | bootstrap | `3.0.0` | CSS and JS UI framework; content only — delivers `Content/bootstrap.css`, `Scripts/bootstrap.js` and the Glyphicons font set under `fonts/` | [src/MVC5/MvcMusicStore/packages.config:4] |
| nuget.org | EntityFramework | `6.0.0` | ORM. Supplies **two** referenced assemblies: `EntityFramework.dll` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:114-116] and `EntityFramework.SqlServer.dll` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:117-119] | [src/MVC5/MvcMusicStore/packages.config:5] |
| nuget.org | jQuery | `1.10.2` | Client JS library; content only — delivers `Scripts/jquery-1.10.2.js` and its minified and IntelliSense companions | [src/MVC5/MvcMusicStore/packages.config:6] |
| nuget.org | jQuery.Validation | `1.11.1` | Client-side validation plugin; content only — `Scripts/jquery.validate.js` | [src/MVC5/MvcMusicStore/packages.config:7] |
| nuget.org | Microsoft.AspNet.Identity.Core | `1.0.0` | ASP.NET Identity 1.0 core abstractions and `UserManager` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:120-122] | [src/MVC5/MvcMusicStore/packages.config:8] |
| nuget.org | Microsoft.AspNet.Identity.EntityFramework | `1.0.0` | EF-backed Identity 1.0 user store [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:126-128]; defines the schema of the credential store the application ships | [src/MVC5/MvcMusicStore/packages.config:9] |
| nuget.org | Microsoft.AspNet.Identity.Owin | `1.0.0` | Bridges Identity 1.0 onto the OWIN authentication pipeline [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:123-125] | [src/MVC5/MvcMusicStore/packages.config:10] |
| nuget.org | Microsoft.AspNet.Mvc | `5.0.0` | The MVC framework; supplies `System.Web.Mvc` 5.0.0.0 [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:76-79] | [src/MVC5/MvcMusicStore/packages.config:11] |
| nuget.org | Microsoft.AspNet.Razor | `3.0.0` | Razor 3 view engine; supplies `System.Web.Razor` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:83-86] | [src/MVC5/MvcMusicStore/packages.config:12] |
| nuget.org | Microsoft.AspNet.Web.Optimization | `1.1.1` | Bundling and minification; supplies `System.Web.Optimization` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:80-82] | [src/MVC5/MvcMusicStore/packages.config:13] |
| nuget.org | Microsoft.AspNet.WebPages | `3.0.0` | Web Pages runtime. Supplies **four** referenced assemblies: `System.Web.Helpers` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:72-75], `System.Web.WebPages` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:87-90], `System.Web.WebPages.Deployment` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:91-94] and `System.Web.WebPages.Razor` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:95-98] | [src/MVC5/MvcMusicStore/packages.config:14] |
| nuget.org | Microsoft.jQuery.Unobtrusive.Validation | `3.0.0` | Unobtrusive validation adapters; content only — `Scripts/jquery.validate.unobtrusive.js` | [src/MVC5/MvcMusicStore/packages.config:15] |
| nuget.org | Microsoft.Owin | `2.0.0` | Katana OWIN host abstractions [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:132-134] | [src/MVC5/MvcMusicStore/packages.config:16] |
| nuget.org | Microsoft.Owin.Host.SystemWeb | `2.0.0` | Hosts the OWIN pipeline on the `System.Web` request pipeline [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:135-137] | [src/MVC5/MvcMusicStore/packages.config:17] |
| nuget.org | Microsoft.Owin.Security | `2.0.0` | Base types shared by all authentication middleware [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:138-140] | [src/MVC5/MvcMusicStore/packages.config:18] |
| nuget.org | Microsoft.Owin.Security.Cookies | `2.0.0` | Cookie authentication — **the only authentication middleware the application actually enables**, at [src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:14-18] and [src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:20] | [src/MVC5/MvcMusicStore/packages.config:19] |
| nuget.org | Microsoft.Owin.Security.Facebook | `2.0.0` | External sign-in provider; its registration is commented out [src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:31-33] — dormant (§3.1.2) | [src/MVC5/MvcMusicStore/packages.config:20] |
| nuget.org | Microsoft.Owin.Security.Google | `2.0.0` | External sign-in provider; registration commented out [src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:35] — dormant | [src/MVC5/MvcMusicStore/packages.config:21] |
| nuget.org | Microsoft.Owin.Security.MicrosoftAccount | `2.0.0` | External sign-in provider; registration commented out [src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:23-25] — dormant | [src/MVC5/MvcMusicStore/packages.config:22] |
| nuget.org | Microsoft.Owin.Security.OAuth | `2.0.0` | OAuth authorization-server and bearer-token **infrastructure**. Not a social provider and not the package any of the four commented registrations would use (§3.1.2) | [src/MVC5/MvcMusicStore/packages.config:23] |
| nuget.org | Microsoft.Owin.Security.Twitter | `2.0.0` | External sign-in provider; registration commented out [src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:27-29] — dormant | [src/MVC5/MvcMusicStore/packages.config:24] |
| nuget.org | Microsoft.Web.Infrastructure | `1.0.0.0` | Dynamic HTTP-module registration at runtime [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:64-67]. Note the four-part version — the manifest says `1.0.0.0`, not `1.0.0` | [src/MVC5/MvcMusicStore/packages.config:25] |
| nuget.org | Modernizr | `2.6.2` | Browser feature detection; content only — `Scripts/modernizr-2.6.2.js` | [src/MVC5/MvcMusicStore/packages.config:26] |
| nuget.org | Newtonsoft.Json | `5.0.6` | JSON serializer. Referenced by the project [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:99-102] but **never called from application source** (§3.1.3) | [src/MVC5/MvcMusicStore/packages.config:27] |
| nuget.org | Owin | `1.0` | The `IAppBuilder` abstraction (`Owin.dll`) [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:129-131]. The manifest says `1.0` — two parts, not `1.0.0` | [src/MVC5/MvcMusicStore/packages.config:28] |
| nuget.org | Respond | `1.2.0` | Media-query polyfill for Internet Explorer 8; content only — `Scripts/respond.js` | [src/MVC5/MvcMusicStore/packages.config:29] |
| nuget.org | WebGrease | `1.5.2` | Minification engine behind `System.Web.Optimization` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:104-107] | [src/MVC5/MvcMusicStore/packages.config:30] |

Verify the count and the exact strings:

```bash
grep -c '<package ' src/MVC5/MvcMusicStore/packages.config          # -> 28
grep -o 'id="[^"]*" version="[^"]*"' src/MVC5/MvcMusicStore/packages.config
```

#### 3.1.1 Finding — the manifest and the project disagree about the platform

**Every one of the 28 entries declares `targetFramework="net45"`** [src/MVC5/MvcMusicStore/packages.config:3-30] while the project declares `<TargetFrameworkVersion>v4.8</TargetFrameworkVersion>` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:16].

```bash
grep -c 'targetFramework="net45"' src/MVC5/MvcMusicStore/packages.config   # -> 28, i.e. all of them
```

This is a finding, not a footnote. The `targetFramework` attribute records the framework each package was *installed against*, and it is what a `packages.config`-era restore and reinstall use to select the correct lib folder from a package payload. A manifest that believes the project is `net45` while the project builds as `v4.8` means the recorded install context no longer describes the project — so which asset group a reinstall selects is determined by a stale value rather than by the project's real target. Every `HintPath` in the project points into a `lib\net45\` folder, consistent with the manifest and not with the project. MVC 5 is the sole migration source (AAP 0.3.1), which is why this particular disagreement matters more than the same class of drift would elsewhere.

A second, independent framework-version disagreement exists inside MVC 5's own `Web.config`. It is not a dependency fact and it is **not** restated here: [12 — Migration Blockers](12-migration-blockers.md) owns it.

#### 3.1.2 Finding — five provider-family packages ship, four external logins are dormant, and one of the five is not a provider at all

The distinction here is easy to get wrong, so it is stated precisely.

MVC 5's authentication configuration enables exactly two things: cookie authentication [src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:14-18] and the external sign-in cookie [src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:20]. Four external-provider registrations sit immediately below, all commented out [src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:22-35] — a fourteen-line block:

| Commented registration | Location | Package that would serve it | Pin |
| --- | --- | --- | --- |
| `app.UseMicrosoftAccountAuthentication(...)` | [src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:23-25] | Microsoft.Owin.Security.MicrosoftAccount | `2.0.0` |
| `app.UseTwitterAuthentication(...)` | [src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:27-29] | Microsoft.Owin.Security.Twitter | `2.0.0` |
| `app.UseFacebookAuthentication(...)` | [src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:31-33] | Microsoft.Owin.Security.Facebook | `2.0.0` |
| `app.UseGoogleAuthentication()` | [src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:35] | Microsoft.Owin.Security.Google | `2.0.0` |

So **four** dormant provider packages are pinned, referenced by the project and deployed with it, serving nothing.

The **fifth** `Microsoft.Owin.Security.*` package, `Microsoft.Owin.Security.OAuth` `2.0.0` [src/MVC5/MvcMusicStore/packages.config:23], is **not** a social provider and is not the package any of those four registrations would use. It supplies OAuth authorization-server and bearer-token infrastructure. Counting it as a fifth dormant social provider — or, in the other direction, treating the four provider packages as "the OAuth package" — is a common and material error: the two have different removal consequences, and [04 — .NET 8 Migration Strategy](04-dotnet8-migration-strategy.md) assigns each package its own outcome on that basis.

The same pattern, with a different stack, appears in MVC 4 (§3.2.3).

#### 3.1.3 Finding — Newtonsoft.Json 5.0.6 ships but is never called from application source

`Newtonsoft.Json` `5.0.6` is pinned [src/MVC5/MvcMusicStore/packages.config:27] and referenced by the project [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:99-102], so it is restored, copied to `bin` and deployed. **No application source file references it.** A repository-wide search across every tracked C# and Razor file, excluding the committed package payloads, returns nothing:

```bash
git ls-files '*.cs' '*.cshtml' | grep -v '/packages/' | xargs grep -lE 'Newtonsoft|JsonConvert' | wc -l   # -> 0
```

The application's one JSON-producing endpoint returns an MVC `JsonResult`, and in MVC 5 that result type serializes through `JavaScriptSerializer` rather than through Newtonsoft.Json. The package is therefore **template baggage** — it arrived with the project template and nothing consumes it.

Two consequences must not be conflated, and this document deliberately draws only the first:

- **As an inventory fact:** the pin exists, is deployed, and has no consumer in application code.
- **As a migration fact:** because nothing calls it, removing it is not a serializer change — but the endpoint's serialized output *does* change for an unrelated reason. That consequence, and the JSON property-naming behaviour behind it, is owned by [12 — Migration Blockers](12-migration-blockers.md) and is not analyzed here.

#### 3.1.4 Six of MVC 5's 28 pins deliver content rather than an assembly

Matching each pin's exact package folder against the project's `HintPath` entries: 22 of the 28 pins have their payload referenced as an assembly; the remaining six deliver static files only — **bootstrap, jQuery, jQuery.Validation, Microsoft.jQuery.Unobtrusive.Validation, Modernizr and Respond**.

Their pinned versions are corroborated a second way, by the vendored filenames under the application root: `Scripts/jquery-1.10.2.js` matches the jQuery pin `1.10.2`, `Scripts/modernizr-2.6.2.js` matches Modernizr `2.6.2`, and `Content/bootstrap.css` with `fonts/glyphicons-halflings-regular.*` matches bootstrap `3.0.0`'s payload shape. The version a browser is served is the version the manifest pins.

```bash
git ls-files 'src/MVC5/MvcMusicStore/Scripts/*' 'src/MVC5/MvcMusicStore/Content/*' 'src/MVC5/MvcMusicStore/fonts/*'
```

The distinction matters to the inventory because a content-only package has no assembly to bind, no binding redirect and no runtime version to reconcile — its entire footprint is files already committed to the tree. The acquisition mechanism these six would need after a migration is owned by [05 — ASP.NET Core Migration Approach](05-aspnet-core-migration-approach.md).

### 3.2 MVC 4 — 29 pins

Manifest: **[src/MVC4/MvcMusicStore/packages.config:3-31]** — 32 lines end to end, with all 29 `<package>` entries on lines 3 to 31, verified with `awk 'END{print NR}' src/MVC4/MvcMusicStore/packages.config` → `32` and `grep -n '<package ' src/MVC4/MvcMusicStore/packages.config | sed -n '1p;$p'` → first pin at `3`, last at `31`. Every entry declares `targetFramework="net45"`, matching the project's `<TargetFrameworkVersion>v4.5</TargetFrameworkVersion>` [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:16]; unlike MVC 5, this manifest and its project agree.

```bash
grep -c 'targetFramework="net45"' src/MVC4/MvcMusicStore/packages.config    # -> 29, i.e. all of them
```

| Registry | Package | Version | Purpose | Manifest citation |
| --- | --- | --- | --- | --- |
| nuget.org | DotNetOpenAuth.AspNet | `4.0.3.12153` | ASP.NET integration for the DotNetOpenAuth stack [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:127-130]; serves the commented-out external logins (§3.2.3) | [src/MVC4/MvcMusicStore/packages.config:3] |
| nuget.org | DotNetOpenAuth.Core | `4.0.3.12153` | Core protocol types shared by the OAuth and OpenID assemblies [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:131-134] | [src/MVC4/MvcMusicStore/packages.config:4] |
| nuget.org | DotNetOpenAuth.OAuth.Consumer | `4.0.3.12153` | OAuth consumer (relying-party) implementation [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:135-138] | [src/MVC4/MvcMusicStore/packages.config:5] |
| nuget.org | DotNetOpenAuth.OAuth.Core | `4.0.3.12153` | OAuth core; supplies `DotNetOpenAuth.OAuth.dll` [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:139-142] — note the assembly name differs from the package id | [src/MVC4/MvcMusicStore/packages.config:6] |
| nuget.org | DotNetOpenAuth.OpenId.Core | `4.0.3.12153` | OpenID core; supplies `DotNetOpenAuth.OpenId.dll` [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:143-146] | [src/MVC4/MvcMusicStore/packages.config:7] |
| nuget.org | DotNetOpenAuth.OpenId.RelyingParty | `4.0.3.12153` | OpenID relying-party implementation [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:147-150] | [src/MVC4/MvcMusicStore/packages.config:8] |
| nuget.org | EntityFramework | `5.0.0` | ORM; supplies `EntityFramework.dll` [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:65-67]. Declared again in configuration as `EntityFramework, Version=5.0.0.0` [src/MVC4/MvcMusicStore/Web.config:9] | [src/MVC4/MvcMusicStore/packages.config:9] |
| nuget.org | jQuery | `1.7.1.1` | Client JS library; content only — `Scripts/jquery-1.7.1.js` | [src/MVC4/MvcMusicStore/packages.config:10] |
| nuget.org | jQuery.UI.Combined | `1.8.20.1` | jQuery UI widget library; content only. **Source of MVC 4's nested theme tree** (§3.2.4) | [src/MVC4/MvcMusicStore/packages.config:11] |
| nuget.org | jQuery.Validation | `1.9.0.1` | Client-side validation plugin; content only — `Scripts/jquery.validate.js` | [src/MVC4/MvcMusicStore/packages.config:12] |
| nuget.org | knockoutjs | `2.1.0` | Client-side MVVM library; content only — `Scripts/knockout-2.1.0.js` | [src/MVC4/MvcMusicStore/packages.config:13] |
| nuget.org | Microsoft.AspNet.Mvc | `4.0.20710.0` | The MVC framework; supplies `System.Web.Mvc` 4.0.0.0 [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:92-95] | [src/MVC4/MvcMusicStore/packages.config:14] |
| nuget.org | Microsoft.AspNet.Razor | `2.0.20710.0` | Razor 2 view engine; supplies `System.Web.Razor` [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:99-102] | [src/MVC4/MvcMusicStore/packages.config:15] |
| nuget.org | Microsoft.AspNet.Web.Optimization | `1.0.0` | Bundling and minification; supplies `System.Web.Optimization` [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:96-98] | [src/MVC4/MvcMusicStore/packages.config:16] |
| nuget.org | Microsoft.AspNet.WebApi | `4.0.20710.0` | **Metapackage** — its committed payload is the `.nupkg` alone, with no `lib` folder, so it contributes no assembly of its own and exists to pull in the three packages below (§3.2.2) | [src/MVC4/MvcMusicStore/packages.config:17] |
| nuget.org | Microsoft.AspNet.WebApi.Client | `4.0.20710.0` | Web API client libraries; supplies `System.Net.Http.Formatting` [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:77-79] | [src/MVC4/MvcMusicStore/packages.config:18] |
| nuget.org | Microsoft.AspNet.WebApi.Core | `4.0.20710.0` | Web API runtime; supplies `System.Web.Http` [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:86-88] | [src/MVC4/MvcMusicStore/packages.config:19] |
| nuget.org | Microsoft.AspNet.WebApi.WebHost | `4.0.20710.0` | Hosts Web API on `System.Web`; supplies `System.Web.Http.WebHost` [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:89-91] | [src/MVC4/MvcMusicStore/packages.config:20] |
| nuget.org | Microsoft.AspNet.WebPages | `2.0.20710.0` | Web Pages runtime. Supplies **four** referenced assemblies: `System.Web.Helpers` [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:82-85], `System.Web.WebPages` [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:103-106], `System.Web.WebPages.Deployment` [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:107-110] and `System.Web.WebPages.Razor` [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:111-114] | [src/MVC4/MvcMusicStore/packages.config:21] |
| nuget.org | Microsoft.AspNet.WebPages.Data | `2.0.20710.0` | Web Pages data helpers; supplies `WebMatrix.Data` [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:115-118] | [src/MVC4/MvcMusicStore/packages.config:22] |
| nuget.org | Microsoft.AspNet.WebPages.OAuth | `2.0.20710.0` | `OAuthWebSecurity` surface; supplies `Microsoft.Web.WebPages.OAuth` [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:119-122] — the type the commented registrations of §3.2.3 call | [src/MVC4/MvcMusicStore/packages.config:23] |
| nuget.org | Microsoft.AspNet.WebPages.WebData | `2.0.20710.0` | SimpleMembership provider; supplies `WebMatrix.WebData` [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:123-126]. This is MVC 4's membership stack | [src/MVC4/MvcMusicStore/packages.config:24] |
| nuget.org | Microsoft.jQuery.Unobtrusive.Ajax | `2.0.20710.0` | Unobtrusive AJAX adapters; content only — `Scripts/jquery.unobtrusive-ajax.js` | [src/MVC4/MvcMusicStore/packages.config:25] |
| nuget.org | Microsoft.jQuery.Unobtrusive.Validation | `2.0.20710.0` | Unobtrusive validation adapters; content only — `Scripts/jquery.validate.unobtrusive.js` | [src/MVC4/MvcMusicStore/packages.config:26] |
| nuget.org | Microsoft.Net.Http | `2.0.20710.0` | `HttpClient` backport for .NET 4.0. Its committed payload carries `lib/net40` assemblies and a `lib/net45/_._` placeholder, so **on this `net45` project it contributes nothing** (§3.2.2) | [src/MVC4/MvcMusicStore/packages.config:27] |
| nuget.org | Microsoft.Web.Infrastructure | `1.0.0.0` | Dynamic HTTP-module registration [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:68-71]. Four-part version, as in MVC 5 | [src/MVC4/MvcMusicStore/packages.config:28] |
| nuget.org | Modernizr | `2.5.3` | Browser feature detection; content only — `Scripts/modernizr-2.5.3.js` | [src/MVC4/MvcMusicStore/packages.config:29] |
| nuget.org | Newtonsoft.Json | `4.5.6` | JSON serializer [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:72-74]; a Web API formatter dependency. Not called from MVC 4 application source either — the same command as §3.1.3 returns zero across the whole repository | [src/MVC4/MvcMusicStore/packages.config:30] |
| nuget.org | WebGrease | `1.1.0` | Minification engine behind `System.Web.Optimization`. Supplies **two** referenced assemblies here: `WebGrease.dll` [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:151-154] and `Antlr3.Runtime.dll` [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:155-158] (§3.2.5) | [src/MVC4/MvcMusicStore/packages.config:31] |

```bash
grep -c '<package ' src/MVC4/MvcMusicStore/packages.config          # -> 29
grep -o 'id="[^"]*" version="[^"]*"' src/MVC4/MvcMusicStore/packages.config
```

#### 3.2.1 A shared release stamp is not a single version

Thirteen of MVC 4's 29 pins carry the version `4.0.20710.0` or `2.0.20710.0` — the same `20710` release stamp from the ASP.NET MVC 4 / Web API wave. It is worth being explicit that **this is a shared build stamp, not one version**: the MVC and Web API packages are at `4.0.20710.0` while the Razor, WebPages and unobtrusive-script packages from the same wave are at `2.0.20710.0`, and the remaining sixteen pins share neither. Reading `20710` as a single "framework version" for MVC 4 would flatten two distinct version strings across those thirteen pins into one, and be wrong for whichever of the two groups it did not pick.

```bash
grep -c '20710' src/MVC4/MvcMusicStore/packages.config              # -> 13 carry the stamp
grep -c 'version="4.0.20710.0"' src/MVC4/MvcMusicStore/packages.config   # -> 5
grep -c 'version="2.0.20710.0"' src/MVC4/MvcMusicStore/packages.config   # -> 8
grep -c '<package ' src/MVC4/MvcMusicStore/packages.config          # -> 29, so 16 do not carry it
```

The manifest's own numbers, grouped:

| Version string | Pins | Packages |
| --- | --- | --- |
| `4.0.20710.0` | 5 | Microsoft.AspNet.Mvc; Microsoft.AspNet.WebApi; Microsoft.AspNet.WebApi.Client; Microsoft.AspNet.WebApi.Core; Microsoft.AspNet.WebApi.WebHost |
| `2.0.20710.0` | 8 | Microsoft.AspNet.Razor; Microsoft.AspNet.WebPages; Microsoft.AspNet.WebPages.Data; Microsoft.AspNet.WebPages.OAuth; Microsoft.AspNet.WebPages.WebData; Microsoft.jQuery.Unobtrusive.Ajax; Microsoft.jQuery.Unobtrusive.Validation; Microsoft.Net.Http |
| `4.0.3.12153` | 6 | the six DotNetOpenAuth packages |
| ten further distinct versions | 10 | EntityFramework `5.0.0`; jQuery `1.7.1.1`; jQuery.UI.Combined `1.8.20.1`; jQuery.Validation `1.9.0.1`; knockoutjs `2.1.0`; Microsoft.AspNet.Web.Optimization `1.0.0`; Microsoft.Web.Infrastructure `1.0.0.0`; Modernizr `2.5.3`; Newtonsoft.Json `4.5.6`; WebGrease `1.1.0` |

Thirteen pins share the `20710` stamp across **two** different version strings, and sixteen do not carry it at all; 5 + 8 + 6 + 10 = 29. This grouping exists only to prevent the stamp being mistaken for a version — the authoritative per-pin values are the table in §3.2.

#### 3.2.2 Finding — four Web API packages support a route with zero implementations, and one of the four contributes nothing

MVC 4 pins four Web API packages [src/MVC4/MvcMusicStore/packages.config:17-20] and registers a Web API route: `config.Routes.MapHttpRoute` with `name: "DefaultApi"` and `routeTemplate: "api/{controller}/{id}"` [src/MVC4/MvcMusicStore/App_Start/WebApiConfig.cs:12-16].

**There is no `ApiController` implementation anywhere in the repository** — not in MVC 4, not in any edition:

```bash
git ls-files '*.cs' | grep -v '/packages/' | xargs grep -n 'ApiController' | wc -l   # -> 0
```

The route is mapped and can never dispatch. Four packages, their assemblies and their transitive payload are restored, referenced and deployed to serve it.

Two of the four deserve their own note, because both are cases where the pin count overstates the delivered surface:

- **`Microsoft.AspNet.WebApi` `4.0.20710.0` is a metapackage.** Its committed payload consists of exactly one file, its own `.nupkg`, with no `lib` folder at all — verifiable directly, because MVC 4's `packages/` tree is committed: `git ls-files 'src/MVC4/MvcMusicStore/packages/Microsoft.AspNet.WebApi.4.0.20710.0/*'` returns only `Microsoft.AspNet.WebApi.4.0.20710.0.nupkg`. It contributes no assembly of its own; it exists to bring in the other three.
- **`Microsoft.Net.Http` `2.0.20710.0` contributes nothing to this project.** It is the `HttpClient` backport for .NET 4.0. The project references `System.Net.Http` and `System.Net.Http.WebRequest` **with no `HintPath`** [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:75-76] and [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:80-81], so those resolve from the framework, not from the package. The package's own payload confirms the intent: `lib/net40/System.Net.Http.dll` exists, and the `net45` group is the single placeholder file `lib/net45/_._`, which is how a package declares "on this framework I supply nothing". Since the project targets `v4.5`, the pin is inert.

The debt framing and severity of the dead Web API scaffolding are owned by [08 — Technical Debt Register](08-technical-debt-register.md); the inventory fact is stated here.

#### 3.2.3 Finding — six DotNetOpenAuth packages support external logins that are all commented out

All six DotNetOpenAuth pins sit at `4.0.3.12153` [src/MVC4/MvcMusicStore/packages.config:3-8] and all six assemblies are referenced by the project [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:127-150]. What they exist to serve is entirely disabled:

| Commented registration | Location |
| --- | --- |
| `OAuthWebSecurity.RegisterMicrosoftClient(...)` | [src/MVC4/MvcMusicStore/App_Start/AuthConfig.cs:17-19] |
| `OAuthWebSecurity.RegisterTwitterClient(...)` | [src/MVC4/MvcMusicStore/App_Start/AuthConfig.cs:21-23] |
| `OAuthWebSecurity.RegisterFacebookClient(...)` | [src/MVC4/MvcMusicStore/App_Start/AuthConfig.cs:25-27] |
| `OAuthWebSecurity.RegisterGoogleClient()` | [src/MVC4/MvcMusicStore/App_Start/AuthConfig.cs:29] |

`AuthConfig.RegisterAuth()` [src/MVC4/MvcMusicStore/App_Start/AuthConfig.cs:12-30] therefore has an empty body once the comments are discounted. Six packages plus the `Microsoft.AspNet.WebPages.OAuth` pin that provides `OAuthWebSecurity` itself are deployed for a capability no request can reach.

This is the same shape as MVC 5's dormant-provider finding (§3.1.2) but a different stack: MVC 5 carries four dormant Katana provider packages, MVC 4 carries six DotNetOpenAuth packages plus the WebPages OAuth surface. The two are not interchangeable and neither substitutes for the other in a per-package outcome assessment.

#### 3.2.4 `jQuery.UI.Combined` `1.8.20.1` is the origin of MVC 4's nested asset tree

MVC 4's `Content` directory holds 55 tracked files, of which **54 sit under `Content/themes/`** — the jQuery UI base theme, delivered by this pin:

```bash
git ls-files 'src/MVC4/MvcMusicStore/Content/*' | wc -l                        # -> 55
git ls-files 'src/MVC4/MvcMusicStore/Content/themes/*' | wc -l                 # -> 54
git ls-files 'src/MVC4/MvcMusicStore/packages/jQuery.UI.Combined.1.8.20.1/*'   # its theme payload sits at Content/Content/themes/base/**
```

One content-only pin therefore accounts for the large majority of MVC 4's static-asset count, and for the fact that any pattern over MVC 4's `Content` directory must be recursive. The asset-migration sizing that depends on it is owned by [05 — ASP.NET Core Migration Approach](05-aspnet-core-migration-approach.md).

#### 3.2.5 MVC 4 has no Antlr pin — it takes `Antlr3.Runtime` out of the WebGrease payload

MVC 5 pins `Antlr` `3.4.1.9004` explicitly [src/MVC5/MvcMusicStore/packages.config:3] and references it from that package's own folder [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:108-111].

MVC 4 has **no `Antlr` pin at all**, yet still references `Antlr3.Runtime` — resolving it from inside the WebGrease package: `<HintPath>..\packages\WebGrease.1.1.0\lib\Antlr3.Runtime.dll</HintPath>` [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:155-158].

The dependency is real and identical in kind across the two editions, but it is *declared* in one and *undeclared* in the other. An inventory built by reading `packages.config` alone would report MVC 4 as having no Antlr dependency, which is wrong. This is the smallest instance of the general problem §4 addresses.

### 3.3 MVC 3 — 6 pins

Manifest: **[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/packages.config:3-8]** — 9 lines end to end, with all 6 `<package>` entries on lines 3 to 8. Verified with `awk 'END{print NR}' src/MVC3/MvcMusicStore-Completed/MvcMusicStore/packages.config` → `9` and `grep -n '<package ' src/MVC3/MvcMusicStore-Completed/MvcMusicStore/packages.config | sed -n '1p;$p'` → first pin at `3`, last at `8`.

**No entry carries a `targetFramework` attribute** — the attribute is absent from all six, not merely blank. This is the pre-NuGet-2.0 manifest form, consistent with the project's `<TargetFrameworkVersion>v4.0</TargetFrameworkVersion>` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:15] and with the 2011 vintage of the edition:

```bash
grep -c '<package ' src/MVC3/MvcMusicStore-Completed/MvcMusicStore/packages.config           # -> 6
grep -c 'targetFramework' src/MVC3/MvcMusicStore-Completed/MvcMusicStore/packages.config     # -> 0
```

With no recorded install framework, a reinstall has no manifest-supplied basis for choosing an asset group — it must infer one from the project. The consequence is a restore-posture concern rather than a dependency-graph one, and it belongs to [10 — Build and Deployment Requirements](10-build-and-deployment-requirements.md).

| Registry | Package | Version | Purpose | Manifest citation |
| --- | --- | --- | --- | --- |
| nuget.org | jQuery | `1.5.1` | Client JS library; content only — `Scripts/jquery-1.5.1.js` | [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/packages.config:3] |
| nuget.org | jQuery.vsdoc | `1.5.1` | IntelliSense companion for jQuery `1.5.1`; content only, design-time only — `Scripts/jquery-1.5.1-vsdoc.js`. It has no runtime role at all | [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/packages.config:4] |
| nuget.org | jQuery.Validation | `1.8.0` | Client-side validation plugin; content only — `Scripts/jquery.validate.js` | [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/packages.config:5] |
| nuget.org | jQuery.UI.Combined | `1.8.11` | jQuery UI widget library; content only — `Scripts/jquery-ui-1.8.11.js` plus the nested theme tree under `Content/` | [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/packages.config:6] |
| nuget.org | EntityFramework | `4.1.10331.0` | ORM, and **the only pin in this edition whose payload the project references**: `<HintPath>..\packages\EntityFramework.4.1.10331.0\lib\EntityFramework.dll</HintPath>` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:37-40], bound as `EntityFramework, Version=4.1.0.0` | [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/packages.config:7] |
| nuget.org | Modernizr | `1.7` | Browser feature detection; content only — `Scripts/modernizr-1.7.js`. The manifest says `1.7` — two parts | [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/packages.config:8] |

Note the `EntityFramework` row's two different numbers, both correct and both transcribed: the **package** version is `4.1.10331.0` (the folder name and the pin), while the **assembly** version the project binds is `4.1.0.0` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:37]. Package version and assembly version are not the same identifier, and §4.4 collects every place in this repository where they diverge.

#### 3.3.1 Five of six pins are content, and the framework itself is not a pin

Only `EntityFramework` is referenced as an assembly; the other five deliver static files. MVC 3's *application* framework — ASP.NET MVC 3 itself — is not in this manifest at all. It is a machine-installed dependency, and §4.1 records it.

MVC 3's `Scripts/` folder makes the same point from the other direction. It ships `jquery.validate.unobtrusive.js`, `jquery.unobtrusive-ajax.js`, `MicrosoftAjax.js` and `MicrosoftMvcValidation.js` with **no corresponding pin of any kind** — those files arrive from the ASP.NET MVC 3 project template, not from a package:

```bash
git ls-files 'src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Scripts/*'
```

So MVC 3's six-line manifest understates its dependency surface more than either other edition's does.

---

## 4. Dependencies NuGet does not resolve

The 63 pins of §3 are not the dependency surface. Each edition also depends on assemblies and products that no manifest declares: .NET Framework assemblies from the targeting pack, assemblies installed machine-wide by a separate product, a native database provider, and assembly versions declared only in configuration. This section catalogues them, because an inventory that stopped at `packages.config` would let a migration plan discover them at build time instead.

### 4.1 Finding — MVC 3's MVC framework assembly is not a pin, and its provider is not a package

Two of MVC 3's dependencies are entirely undeclared by any manifest, and both are required to build or run it.

**`System.Web.Mvc` is referenced with no `HintPath`.** The reference is `<Reference Include="System.Web.Mvc, Version=3.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35, processorArchitecture=MSIL" />` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:42] — a bare, self-closing reference with no hint path and no package folder behind it. It therefore resolves only from a **machine-wide install of the ASP.NET MVC 3 Tools Update**, a separately installed, out-of-support product. That is a first-class dependency of this edition that appears in no `packages.config`, and it must be recorded as such.

Its immediate neighbours are the same class of dependency:

| Reference | Locator | Resolution |
| --- | --- | --- |
| `System.Data.Entity` | [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:41] | .NET Framework assembly (targeting pack) |
| `System.Web.Mvc, Version=3.0.0.0` | [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:42] | **machine-wide ASP.NET MVC 3 Tools Update — no package** |
| `System.Web.WebPages, Version=1.0.0.0` | [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:43] | machine-wide, same product — no package |
| `System.Web.Helpers, Version=1.0.0.0` | [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:44] | machine-wide, same product — no package |
| `System.Web.Entity` | [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:50] | .NET Framework assembly (targeting pack) |

The project's configuration corroborates all three of the machine-installed assemblies rather than contradicting them. `web.config` lists five assemblies explicitly for the ASP.NET compilation system [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:17-23] — `System.Web.Abstractions` `4.0.0.0`, `System.Web.Helpers` `1.0.0.0`, `System.Web.Routing` `4.0.0.0`, `System.Web.Mvc` `3.0.0.0` and `System.Web.WebPages` `1.0.0.0` — and adds a binding redirect that pins any `System.Web.Mvc` reference between `1.0.0.0` and `2.0.0.0` forward to `3.0.0.0` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:50-51]. A further version declaration sits in app settings: `webpages:Version` = `1.0.0.0` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:9].

**MVC 3's database provider is also not a package.** Its only connection string declares `providerName="System.Data.SqlServerCe.4.0"` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:55-59] — SQL Server Compact 4.0, a separately installed product with a native component. No pin delivers it, no `HintPath` references it, and no `.sdf` data file is committed. As a dependency-inventory fact: **running MVC 3 requires a machine-wide install of SQL Server Compact 4.0 in addition to the MVC 3 Tools Update.** Whether that provider has a forward path is a blocker question owned by [12 — Migration Blockers](12-migration-blockers.md), and the database components each edition needs in order to run are owned by [10 — Build and Deployment Requirements](10-build-and-deployment-requirements.md).

### 4.2 Framework assembly references per edition

Every edition also depends on .NET Framework assemblies resolved from the targeting pack rather than from any package. Counting `<Reference>` elements against those carrying a `<HintPath>`:

| Edition | `<Reference>` elements | With a `<HintPath>` | Without — resolved from the framework or machine-wide | Evidence |
| --- | --- | --- | --- | --- |
| MVC 5 | 46 | 26 | **20** | [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:47-63], [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:68-71], [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:103] |
| MVC 4 | 47 | 24 | **23** | [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:44-64], [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:75-76], [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:80-81] |
| MVC 3 | 24 | 1 | **23** | [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:41-63] |

```bash
for f in src/MVC5/MvcMusicStore/MvcMusicStore.csproj \
         src/MVC4/MvcMusicStore/MvcMusicStore.csproj \
         src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj; do
  printf '%s refs=%s hintpaths=%s\n' "$f" "$(grep -c '<Reference Include' "$f")" "$(grep -c '<HintPath>' "$f")"
done
```

Two of MVC 4's twenty-three unhinted references are the `System.Net.Http` pair discussed in §3.2.2 — declared without a hint path even though a pin exists that could have supplied them, which is why the counts here and the pin counts in §3 answer different questions and must not be added together. MVC 3's twenty-three include the three machine-installed assemblies of §4.1, which are *not* framework assemblies; the column is therefore titled "framework or machine-wide" rather than "framework".

The consequence for build prerequisites — which targeting pack, and which separately installed product, each edition needs — is owned by [10 — Build and Deployment Requirements](10-build-and-deployment-requirements.md).

#### 4.2.1 All sixty-six unhinted references, enumerated

**A count does not discharge this document's promise, so the list is here.** §1.1 commits to a *complete, verbatim catalogue of every declared dependency*, and for this class the count above says only **how many** assemblies each build resolves from outside NuGet. Which ones is the question a migration plan actually asks — every name below has to have a `net8.0` counterpart, be replaced, or be recorded as having neither — and a per-edition total cannot be diffed against a project file the way a list can. Each row carries the `Include` attribute **exactly as the project declares it**, the line it is declared on, and its class: **framework** (resolved from the .NET Framework targeting pack) or **machine-wide** (resolved from a separately installed product — §4.1). The enumerating command is below the three tables, and it is the same one that produced them.

**MVC 5 — 20, all framework** [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:47-103].

| Line | `Include` | Class |
| --- | --- | --- |
| 47 | `Microsoft.CSharp` | framework |
| 48 | `System` | framework |
| 49 | `System.Data` | framework |
| 50 | `System.Data.DataSetExtensions` | framework |
| 51 | `System.Drawing` | framework |
| 52 | `System.Web.DynamicData` | framework |
| 53 | `System.Web.Entity` | framework |
| 54 | `System.Web.ApplicationServices` | framework |
| 55 | `System.ComponentModel.DataAnnotations` | framework |
| 56 | `System.Web.Extensions` | framework |
| 57 | `System.Web` | framework |
| 58 | `System.Web.Abstractions` | framework |
| 59 | `System.Web.Routing` | framework |
| 60 | `System.Xml` | framework |
| 61 | `System.Configuration` | framework |
| 62 | `System.Web.Services` | framework |
| 63 | `System.EnterpriseServices` | framework |
| 68 | `System.Net.Http` | framework |
| 70 | `System.Net.Http.WebRequest` | framework |
| 103 | `System.Xml.Linq` | framework |

**MVC 4 — 23, all framework** [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:44-80].

| Line | `Include` | Class |
| --- | --- | --- |
| 44 | `Microsoft.CSharp` | framework |
| 45 | `System` | framework |
| 46 | `System.Data` | framework |
| 47 | `System.Data.Entity` | framework |
| 48 | `System.Drawing` | framework |
| 49 | `System.Web.DynamicData` | framework |
| 50 | `System.Web.Entity` | framework |
| 51 | `System.Web.ApplicationServices` | framework |
| 52 | `System.ComponentModel.DataAnnotations` | framework |
| 53 | `System.Core` | framework |
| 54 | `System.Data.DataSetExtensions` | framework |
| 55 | `System.Xml.Linq` | framework |
| 56 | `System.Web` | framework |
| 57 | `System.Web.Extensions` | framework |
| 58 | `System.Web.Abstractions` | framework |
| 59 | `System.Web.Routing` | framework |
| 60 | `System.Xml` | framework |
| 61 | `System.Configuration` | framework |
| 62 | `System.Transactions` | framework |
| 63 | `System.Web.Services` | framework |
| 64 | `System.EnterpriseServices` | framework |
| 75 | `System.Net.Http` | framework — **and a pin exists that could have supplied it** (§3.2.2) |
| 80 | `System.Net.Http.WebRequest` | framework — same, and the same pin |

**MVC 3 — 23: 20 framework and 3 machine-wide** [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:41-63]. The three machine-wide entries are the only unhinted references in any edition that carry a strong name; each declares `Culture=neutral, PublicKeyToken=31bf3856ad364e35, processorArchitecture=MSIL` after the version, and §4.1 quotes the first of the three in full.

| Line | `Include` | Class |
| --- | --- | --- |
| 41 | `System.Data.Entity` | framework |
| 42 | `System.Web.Mvc, Version=3.0.0.0, …` | **machine-wide** — ASP.NET MVC 3 Tools Update (§4.1) |
| 43 | `System.Web.WebPages, Version=1.0.0.0, …` | **machine-wide** — same product |
| 44 | `System.Web.Helpers, Version=1.0.0.0, …` | **machine-wide** — same product |
| 45 | `Microsoft.CSharp` | framework |
| 46 | `System` | framework |
| 47 | `System.Data` | framework |
| 48 | `System.Drawing` | framework |
| 49 | `System.Web.DynamicData` | framework |
| 50 | `System.Web.Entity` | framework |
| 51 | `System.Web.ApplicationServices` | framework |
| 52 | `System.ComponentModel.DataAnnotations` | framework |
| 53 | `System.Core` | framework |
| 54 | `System.Data.DataSetExtensions` | framework |
| 55 | `System.Xml.Linq` | framework |
| 56 | `System.Web` | framework |
| 57 | `System.Web.Extensions` | framework |
| 58 | `System.Web.Abstractions` | framework |
| 59 | `System.Web.Routing` | framework |
| 60 | `System.Xml` | framework |
| 61 | `System.Configuration` | framework |
| 62 | `System.Web.Services` | framework |
| 63 | `System.EnterpriseServices` | framework |

**Sixty-six rows: 63 framework and 3 machine-wide.** That 63 is a coincidence and not a correspondence — it is **not** the 63 pins of §3, shares no member with them, and the two must never be added or matched. The reproducing command prints exactly the three lists above:

```bash
for f in src/MVC5/MvcMusicStore/MvcMusicStore.csproj \
         src/MVC4/MvcMusicStore/MvcMusicStore.csproj \
         src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj; do
  echo "== $f"
  awk '
    /<Reference Include=/                              { inref=1; buf=$0; nl=FNR; hp=0 }
    inref && /<HintPath>/                              { hp=1 }
    inref && (/\/>[ \t]*$/ || /<\/Reference>/) {
      if (!hp) { match(buf, /Include="[^"]*"/); print nl "  " substr(buf, RSTART+9, RLENGTH-10) }
      inref=0
    }' "$f"
done
# -> MVC 5: 20 lines   MVC 4: 23 lines   MVC 3: 23 lines
```

It is written as an element walk rather than as `grep -c '<Reference Include'` minus `grep -c '<HintPath>'` for a reason worth stating: the subtraction in the block above §4.2.1 is correct **only because** no reference in any of the three projects declares two `<HintPath>` children and none declares a hint path without a `Reference` parent — true here, and checked, but a property of these three files rather than of MSBuild. The walk decides per element and so does not depend on it, which is why both forms are given and why they agree.

The consequence for build prerequisites — which targeting pack, and which separately installed product, each edition needs — is owned by [10 — Build and Deployment Requirements](10-build-and-deployment-requirements.md).

### 4.3 Build-tooling dependencies declared in the project files

These are dependencies in the operational sense: without them MSBuild does not evaluate the project. They are inventoried here and their build consequences are owned by deliverable 10.

| Edition | Import | Locator | Condition |
| --- | --- | --- | --- |
| MVC 5 | `$(MSBuildBinPath)\Microsoft.CSharp.targets` | [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:271] | unconditional |
| MVC 5 | `$(VSToolsPath)\WebApplications\Microsoft.WebApplication.targets` | [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:272] | `'$(VSToolsPath)' != ''` |
| MVC 5 | `$(MSBuildExtensionsPath32)\Microsoft\VisualStudio\v10.0\WebApplications\Microsoft.WebApplication.targets` | [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:273] | `false` — inert |
| MVC 5 | `$(SolutionDir)\.nuget\NuGet.targets` | [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:295] | `Exists(...)` — **conditional** |
| MVC 4 | `$(VSToolsPath)\WebApplications\Microsoft.WebApplication.targets` | [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:337] | `'$(VSToolsPath)' != ''` |
| MVC 4 | `$(SolutionDir)\.nuget\nuget.targets` | [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:360] | **none — unconditional** |
| MVC 3 | `$(MSBuildExtensionsPath32)\Microsoft\VisualStudio\v10.0\WebApplications\Microsoft.WebApplication.targets` | [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:209] | **none — unconditional** |

MVC 4 additionally opts into restore-on-build: `<SolutionDir Condition="$(SolutionDir) == '' Or $(SolutionDir) == '*Undefined*'">..\</SolutionDir>` [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:23] and `<RestorePackages>true</RestorePackages>` [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:24]. That single property is what makes the committed NuGet client of §5 an active build dependency rather than a dormant file.

### 4.4 Assembly versions declared only in configuration, and where they diverge from the pin

Both MVC 5 and MVC 4 carry `assemblyBinding` redirects and an `entityFramework` section that name assembly versions. These are dependency declarations in their own right: they are what the runtime honours, and they are not always the same number as the package pin.

**MVC 5** [src/MVC5/MvcMusicStore/Web.config:41-60]:

| Assembly | Redirect | Pin that supplies it | Same number? |
| --- | --- | --- | --- |
| `System.Web.Helpers` | `1.0.0.0-3.0.0.0` → `3.0.0.0` [src/MVC5/MvcMusicStore/Web.config:44-45] | Microsoft.AspNet.WebPages `3.0.0` | yes |
| `System.Web.Mvc` | `1.0.0.0-5.0.0.0` → `5.0.0.0` [src/MVC5/MvcMusicStore/Web.config:48-49] | Microsoft.AspNet.Mvc `5.0.0` | yes |
| `System.Web.WebPages` | `1.0.0.0-3.0.0.0` → `3.0.0.0` [src/MVC5/MvcMusicStore/Web.config:52-53] | Microsoft.AspNet.WebPages `3.0.0` | yes |
| `WebGrease` | `1.0.0.0-1.5.2.14234` → **`1.5.2.14234`** [src/MVC5/MvcMusicStore/Web.config:56-57] | WebGrease **`1.5.2`** | **no** |

Also in MVC 5: the `entityFramework` configuration section declares `EntityFramework, Version=6.0.0.0` [src/MVC5/MvcMusicStore/Web.config:9], matching the EntityFramework `6.0.0` pin, and the EF provider registration names `EntityFramework.SqlServer` [src/MVC5/MvcMusicStore/Web.config:68] — the second assembly from that same pin. `webpages:Version` is `3.0.0.0` [src/MVC5/MvcMusicStore/Web.config:18].

**MVC 4** [src/MVC4/MvcMusicStore/Web.config:65-74]: `System.Web.Helpers` → `2.0.0.0`, `System.Web.Mvc` → `4.0.0.0`, `System.Web.WebPages` → `2.0.0.0`; the `entityFramework` section declares `EntityFramework, Version=5.0.0.0` [src/MVC4/MvcMusicStore/Web.config:9]; `webpages:Version` is `2.0.0.0` [src/MVC4/MvcMusicStore/Web.config:27].

**MVC 3** is covered in §4.1: five explicit assemblies [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:17-23], one redirect to `3.0.0.0` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:50-51], `webpages:Version` `1.0.0.0` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:9].

The four places in this repository where a package version and the corresponding assembly version are different numbers, collected so no downstream document has to rediscover them:

| Package pin | Assembly version | Locator |
| --- | --- | --- |
| WebGrease `1.5.2` (MVC 5) | `1.5.2.14234` | [src/MVC5/MvcMusicStore/Web.config:57] |
| EntityFramework `4.1.10331.0` (MVC 3) | `4.1.0.0` | [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:37] |
| Microsoft.AspNet.Mvc `4.0.20710.0` (MVC 4) | `4.0.0.0` | [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:92] |
| Microsoft.AspNet.WebPages `2.0.20710.0` (MVC 4) | `2.0.0.0` | [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:82], [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:103], [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:107], [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:111] |

The same divergence holds for the other `20710`-stamped MVC 4 packages, whose assemblies all bind at two-part-derived `4.0.0.0` or `2.0.0.0` versions. The general rule this repository demonstrates: **the version to quote when identifying a package is the `packages.config` value; the version to quote when reasoning about binding is the assembly version, and in this repository they differ more often than they agree.**

---

#### 4.4.1 Every divergence, enumerated — seventeen, not four

There are **seventeen** places in this repository where a package version and the version of the assembly it supplies are different numbers, and all seventeen are listed below rather than sampled. A list of representative examples closed with "the same holds for the others" would leave thirteen of them for a downstream document to rediscover, and deliverable 04 consumes this inventory pin by pin (§9), so the enumeration has to be complete. The census that produces the number is stated first, so the headline is checkable rather than asserted.

**The comparison is mechanical.** For every `<Reference>` whose `Include` declares a `Version=` **and** whose `<HintPath>` resolves into a `packages\<id>.<version>\` folder, the folder carries the *package* version and the `Include` carries the *assembly* version. Those are the only references where the repository states both numbers in one place:

```bash
for f in src/MVC5/MvcMusicStore/MvcMusicStore.csproj \
         src/MVC4/MvcMusicStore/MvcMusicStore.csproj \
         src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj; do
  awk -v F="$f" '
    /<Reference Include=/ { asm=""; if (match($0, /Version=[0-9.]+/)) asm=substr($0, RSTART+8, RLENGTH-8) }
    /<HintPath>/ { if (asm != "" && match($0, /packages\\[^\\]+/)) print F"\t"substr($0, RSTART+9, RLENGTH-9)"\t"asm }
  ' "$f"
done | sort -u          # -> 21 lines; pipe to `wc -l` for the count alone
```

**The arithmetic, because the headline depends on it.** Those **21** distinct package-instance/assembly-version pairs partition three ways, and only the third class is a divergence:

| Class | Count | The cases |
| --- | --- | --- |
| **Identical strings** | 2 | `Microsoft.Web.Infrastructure` `1.0.0.0` in MVC 5 [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:64-67] and in MVC 4 [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:68-71] — the one pin whose four-part version *is* its assembly version, which is why §3.1 and §3.2 both note the four-part form |
| **Same number, different arity** | 3 | MVC 5 only, and all three are the pin zero-extended to four parts: `Microsoft.AspNet.Mvc` `5.0.0` → `5.0.0.0` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:76-79]; `Microsoft.AspNet.Razor` `3.0.0` → `3.0.0.0` [:83-86]; `Microsoft.AspNet.WebPages` `3.0.0` → `3.0.0.0` across its four assemblies [:72-75], [:87-90], [:91-94], [:95-98]. Under the exact-string rule of §1.4 these are *different strings*; as **numbers** they agree, so they are separated out rather than counted as divergences |
| **Genuinely different numbers** | **16** | The fifteen MVC 4 rows and the one MVC 3 row of the table below |

Sixteen from the project files, plus **one the project-file comparison cannot see at all**: MVC 5's `WebGrease` reference declares no `Version=` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:104-107], so that divergence exists only in the binding redirect. **16 + 1 = 17 divergences**, and 2 + 3 + 16 = 21 pairs.

**The seventeen, grouped by edition** so that each group's two files can be named in full once and referred to by line afterwards — the same convention the pin tables of §3.1 and §3.2 use.

**MVC 5 — one.** Manifest: **[src/MVC5/MvcMusicStore/packages.config:30]**. Evidence: **[src/MVC5/MvcMusicStore/Web.config:56-57]**.

| Package pin | Assembly it supplies | Assembly version | Why this one is not in the project file |
| --- | --- | --- | --- |
| WebGrease `1.5.2` | `WebGrease` | **`1.5.2.14234`** | The project reference declares no `Version=` at all [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:104-107], so the redirect is the only place the assembly version appears |

**MVC 4 — fifteen.** Every `Manifest` locator in this table continues the path **[src/MVC4/MvcMusicStore/packages.config:14]**, and every `Declared at` locator continues **[src/MVC4/MvcMusicStore/MvcMusicStore.csproj:92-95]** — the two files cited in full on the first row, abbreviated thereafter in the same continuation form the pin tables of §3.2 use.

| Package pin | Manifest | Assembly it supplies | Assembly version | Declared at |
| --- | --- | --- | --- | --- |
| Microsoft.AspNet.Mvc `4.0.20710.0` | [src/MVC4/MvcMusicStore/packages.config:14] | `System.Web.Mvc` | `4.0.0.0` | [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:92-95] |
| Microsoft.AspNet.Razor `2.0.20710.0` | [:15] | `System.Web.Razor` | `2.0.0.0` | [:99-102] |
| Microsoft.AspNet.WebApi.Client `4.0.20710.0` | [:18] | `System.Net.Http.Formatting` | `4.0.0.0` | [:77-79] |
| Microsoft.AspNet.WebApi.Core `4.0.20710.0` | [:19] | `System.Web.Http` | `4.0.0.0` | [:86-88] |
| Microsoft.AspNet.WebApi.WebHost `4.0.20710.0` | [:20] | `System.Web.Http.WebHost` | `4.0.0.0` | [:89-91] |
| Microsoft.AspNet.WebPages `2.0.20710.0` | [:21] | **Four assemblies from one pin:** `System.Web.Helpers`, `System.Web.WebPages`, `System.Web.WebPages.Deployment`, `System.Web.WebPages.Razor` | `2.0.0.0` — all four | [:82-85], [:103-106], [:107-110], [:111-114] |
| Microsoft.AspNet.WebPages.Data `2.0.20710.0` | [:22] | `WebMatrix.Data` | `2.0.0.0` | [:115-118] |
| Microsoft.AspNet.WebPages.OAuth `2.0.20710.0` | [:23] | `Microsoft.Web.WebPages.OAuth` | `2.0.0.0` | [:119-122] |
| Microsoft.AspNet.WebPages.WebData `2.0.20710.0` | [:24] | `WebMatrix.WebData` | `2.0.0.0` | [:123-126] |
| DotNetOpenAuth.AspNet `4.0.3.12153` | [:3] | `DotNetOpenAuth.AspNet` | `4.0.0.0` | [:127-130] |
| DotNetOpenAuth.Core `4.0.3.12153` | [:4] | `DotNetOpenAuth.Core` | `4.0.0.0` | [:131-134] |
| DotNetOpenAuth.OAuth.Consumer `4.0.3.12153` | [:5] | `DotNetOpenAuth.OAuth.Consumer` | `4.0.0.0` | [:135-138] |
| DotNetOpenAuth.OAuth.Core `4.0.3.12153` | [:6] | `DotNetOpenAuth.OAuth` — **the assembly name differs from the pin id as well as the version** | `4.0.0.0` | [:139-142] |
| DotNetOpenAuth.OpenId.Core `4.0.3.12153` | [:7] | `DotNetOpenAuth.OpenId` — likewise | `4.0.0.0` | [:143-146] |
| DotNetOpenAuth.OpenId.RelyingParty `4.0.3.12153` | [:8] | `DotNetOpenAuth.OpenId.RelyingParty` | `4.0.0.0` | [:147-150] |

**MVC 3 — one.** Manifest: **[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/packages.config:7]**. Evidence: **[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:37-40]**.

| Package pin | Assembly it supplies | Assembly version | Note |
| --- | --- | --- | --- |
| EntityFramework `4.1.10331.0` | `EntityFramework` | `4.1.0.0` | The only pin in this edition whose payload the project references at all (§3.3), so it is also the only divergence available to find here |

**Seventeen rows, seventeen distinct package identifiers, twenty affected assemblies.** No identifier appears twice, so rows and identifiers are the same count; the assembly count is higher because `Microsoft.AspNet.WebPages` supplies four assemblies from one pin — 16 rows contributing one assembly each plus that row's four is 20. Reading it by edition: **MVC 4 accounts for 15 of the 17**, MVC 3 for one and MVC 5 for one — and MVC 5's is the only one visible in configuration rather than in the project file, which is precisely why a census run over the project files alone returns 16 and not 17. MVC 5's three `net45`-era Microsoft packages fall in the arity class above instead, so **the edition that is the migration source has the fewest genuine divergences of the three**, which is a portability fact in its favour rather than a gap in the table.

The general rule this repository demonstrates: **the version to quote when identifying a package is the `packages.config` value; the version to quote when reasoning about binding is the assembly version, and across the 21 pairs above they are the same number in only five.**

---

## 5. The restore client is itself a committed, pinned dependency

### 5.1 A 2012-era NuGet client is tracked in source control

[src/MVC4/MvcMusicStore/.nuget/NuGet.exe:630,784 bytes] is a tracked binary, not a build artifact — **630,784 bytes**, `FileVersion` and assembly version **2.0.30828.5**, `ProductVersion` **2.0.40001**. Per the convention in §1.4, identity is the locator here, because a binary has no line to cite. The verified properties in full, all of them read by the command block below:

| Property | Value |
| --- | --- |
| Size | **630,784 bytes** |
| `FileVersion` | **2.0.30828.5** |
| Assembly version | **2.0.30828.5** |
| `ProductVersion` | **2.0.40001** |
| `ProductName` | NuGet |
| `CompanyName` | Outercurve Foundation |
| SHA-256 | `E52E94A96B7D9F8C1DF5154297468F8FD0260331FB9DE1D48EE6C5867FDD1C09` |

Reproduce on any host with PowerShell:

```powershell
$p = 'src/MVC4/MvcMusicStore/.nuget/NuGet.exe'
(Get-Item $p).Length                                                  # -> 630784
[System.Diagnostics.FileVersionInfo]::GetVersionInfo((Resolve-Path $p)) | Format-List FileVersion, ProductVersion, ProductName, CompanyName
[System.Reflection.AssemblyName]::GetAssemblyName((Resolve-Path $p)).Version   # -> 2.0.30828.5
(Get-FileHash $p -Algorithm SHA256).Hash
```

This is a **NuGet 2.0 client from 2012**, and it is a dependency in its own right for two reasons. First, it is what MVC 4's restore actually executes: `NuGet.targets` resolves `<NuGetExePath ... >$(NuGetToolsPath)\nuget.exe</NuGetExePath>` [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:43] and builds its restore invocation around it [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:53]. Second, the fallback that would replace it is switched off — `<DownloadNuGetExe Condition=" '$(DownloadNuGetExe)' == '' ">false</DownloadNuGetExe>` [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:16] — so the committed file is required rather than optional.

It is the MSBuild-integrated restore mechanism, activated by `<RestorePackages>true</RestorePackages>` [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:24], which is a distinct and long-deprecated mechanism from SDK-integrated restore: NuGet dropped MSBuild-integrated restore with the NuGet 3 generation. Two further properties of that wiring belong to the inventory: restore consent is required — `<RequireRestoreConsent Condition=" '$(RequireRestoreConsent)' != 'false' ">true</RequireRestoreConsent>` [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:13], which adds `-RequireConsent` to the restore command [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:51] — and the restore command's package output directory is `$(SolutionDir)\packages` [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:31], which is how `SolutionDir` comes to determine where MVC 4's packages land.

The consequence for building MVC 4, including how `SolutionDir` must be supplied, is owned by [10 — Build and Deployment Requirements](10-build-and-deployment-requirements.md) and is not restated here. As a dependency fact: **building MVC 4 as committed depends on a 2012 executable stored in the repository, and on a restore mechanism that current NuGet no longer supports.**

### 5.2 The restore configuration is present in one edition and only referenced in another

Exactly three files constitute the repository's entire NuGet tooling surface, all under one edition:

```bash
git ls-files | grep '\.nuget/'
# src/MVC4/MvcMusicStore/.nuget/NuGet.Config
# src/MVC4/MvcMusicStore/.nuget/NuGet.exe
# src/MVC4/MvcMusicStore/.nuget/NuGet.targets
```

MVC 5's solution nevertheless declares a `.nuget` solution folder and lists two of those files as its contents — `Project("{2150E333-8FDC-42A3-9474-1A3956D46DE8}") = ".nuget", ".nuget", ...` [src/MVC5/MvcMusicStore.sln:8] with `.nuget\NuGet.Config` and `.nuget\NuGet.targets` [src/MVC5/MvcMusicStore.sln:10-11] — and MVC 5's project imports `$(SolutionDir)\.nuget\NuGet.targets` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:295]. **No such folder or files exist under `src/MVC5`.** The import is guarded by `Exists(...)`, so it is skipped rather than failing, but the effect is that MVC 5 declares restore configuration it does not have and consequently has no restore wiring of its own at all. MVC 3 declares none and has none.

`NuGet.exe` and the two committed `packages/` trees are also excluded by `.gitignore` yet tracked; that combination, and its severity, is owned by [08 — Technical Debt Register](08-technical-debt-register.md) (§7.2 records the dependency-side facts).

---

## 6. The restore source is not configured anywhere — and a correction to Technical Specification §3.3

### 6.1 What the repository actually contains

Three verified facts, and they are the whole of the repository's evidence on this question.

**1 — The only `NuGet.Config` in the repository configures no package source.** The file — [src/MVC4/MvcMusicStore/.nuget/NuGet.Config:1-6] — is six lines end to end, quoted below with its own line numbers in the trailing gutter:

```xml
<?xml version="1.0" encoding="utf-8"?>          <!-- :1 -->
<configuration>                                 <!-- :2 -->
  <solution>                                    <!-- :3 -->
    <add key="disableSourceControlIntegration" value="true" />   <!-- :4 -->
  </solution>                                   <!-- :5 -->
</configuration>                                <!-- :6 -->
```

Its single setting is `disableSourceControlIntegration`, at key path `configuration/solution/add[@key='disableSourceControlIntegration']` [src/MVC4/MvcMusicStore/.nuget/NuGet.Config:4], which concerns source-control integration and has nothing to do with feeds. **There is no `packageSources` element** — not empty, absent.

**2 — The `nuget.org` v2 endpoint that appears in `NuGet.targets` is inside a comment.** The relevant block is [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:19-25], quoted below with that file's line numbers in the trailing gutter:

```xml
<ItemGroup Condition=" '$(PackageSources)' == '' ">                          <!-- :19 -->
    <!-- Package sources used to restore packages. By default will used the registered sources under %APPDATA%\NuGet\NuGet.Config -->   <!-- :20 -->
    <!--                                                                     :21
        <PackageSource Include="https://nuget.org/api/v2/" />                 :22
        <PackageSource Include="https://my-nuget-source/nuget/" />            :23
    -->                                                                      <!-- :24 -->
</ItemGroup>                                                                 <!-- :25 -->
```

The `https://nuget.org/api/v2/` endpoint sits at [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:22], **inside the XML comment spanning [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:21-24]**, within an `ItemGroup` that is itself conditioned on `'$(PackageSources)' == ''` [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:19]. Two things follow. The `ItemGroup` body is empty, so the `PackageSource` item list is empty. And the second commented line [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:23] is a *private-feed* example — equally inert, and a useful reminder that neither line is a configured value.

**3 — No package source is configured anywhere else in the repository, because there is no other configuration file to hold one.**

```bash
# The repository-wide check. This is the one that supports the absence claim,
# because it filters the whole index rather than matching a root-only pathspec.
git ls-files | grep -iE '(^|/)nuget\.config$'        # -> src/MVC4/MvcMusicStore/.nuget/NuGet.Config, and nothing else
# A narrower, deliberately root-only check. A pathspec with no wildcard matches
# only the repository root (see §7.1), so this proves that and nothing more:
git ls-files 'NuGet.config' 'NuGet.Config' 'nuget.config' | wc -l   # -> 0   (no ROOT-LEVEL file; says nothing about nested ones)
```

Note also the repository's own comment at [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:20], which documents the behaviour explicitly: with no sources listed, restore uses the registered sources under `%APPDATA%\NuGet\NuGet.Config`. The repository states its own inheritance.

### 6.2 The correction

> **Correction to Technical Specification §3.3.** Specification §3.3 describes the `https://nuget.org/api/v2/` endpoint in `NuGet.targets` as the *configured default* package source for this repository. **The repository does not support that reading.** The endpoint exists only inside a commented-out example block [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:21-24], with the endpoint itself at [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:22]; the surrounding `ItemGroup` is empty [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:19-25]; and the repository's only `NuGet.Config` contains a single `disableSourceControlIntegration` setting, at key path `configuration/solution/add[@key='disableSourceControlIntegration']`, with no `packageSources` element at all [src/MVC4/MvcMusicStore/.nuget/NuGet.Config:4]. Under the citation policy that governs this assessment — repository evidence is primary, and a specification section may be cited alongside it but never instead of it — **the repository governs, and the specification's characterization is corrected here rather than inherited.** No claim in this document rests on that endpoint being configured.

This is recorded as a correction, in these terms, rather than as a discrepancy note, because the two readings lead to materially different conclusions and a reader who saw both cited as agreeing would draw the wrong one.

Specification §3.3 remains a useful secondary cross-reference for other parts of the dependency picture — the `packages.config`-only manifest format with no central version management, the per-edition restore weight, and the governance risk of committed executable content inside package payloads, which this document corroborates independently in §7.2. It is the *configured source* claim specifically that is corrected.

### 6.3 The consequence, which is a finding in its own right

Because the `PackageSource` item list is empty, `NuGet.targets` resolves `<PackageSources Condition=" $(PackageSources) == '' ">@(PackageSource)</PackageSources>` [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:44] to an empty value, and the restore command it constructs — `$(NuGetCommand) install "$(PackagesConfig)" -source "$(PackageSources)" ...` [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:53] — is therefore issued with an **empty `-source`**. The client falls back to whatever sources its configuration hierarchy provides.

Stated plainly, and this is the finding:

> **The effective package source set is not knowable from the repository.** Restore inherits whatever machine-level and user-level NuGet configuration the build host happens to carry. It follows that **the repository cannot be asserted to exclude private feeds** — not because there is evidence of one, but because the repository contains no evidence either way. Any statement of the form "this project restores from nuget.org" is a statement about a particular build host, not about this repository, and must be attributed to that host.

Two corollaries worth stating because they are easy to get backwards:

- This does not contradict §2.1. Every one of the 63 **identifiers** is public and verifiable on nuget.org; what is unknowable is which **feed** serves them. An unpinned source combined with public identifiers is precisely the shape in which a substituted package would be hard to detect.
- The finding is about the repository, so it is unaffected by any particular host's configuration being benign. A build performed on a host whose user-level configuration lists only nuget.org is reproducible on that host and nowhere else, and the repository records nothing that would let a second host reproduce it.

The target-state remedy — a committed `NuGet.config` that clears inherited sources and declares its own — is specified by [04 — .NET 8 Migration Strategy](04-dotnet8-migration-strategy.md). This document deliberately does not specify it.

---

## 7. What the repository does not pin

### 7.1 No lockfile exists in any edition

```bash
git ls-files 'packages.lock.json' | wc -l          # -> 0
git ls-files | grep -c 'packages.lock.json'        # -> 0
```

`packages.lock.json` is absent throughout — not stale, not partial, absent. There is also no central version management file of any kind: no `Directory.Build.props`, no `Directory.Packages.props`, no `.ruleset`, no `.editorconfig`.

```bash
git ls-files | grep -iE '(\.ruleset$)|(Directory\.Build\.props)|(Directory\.Packages\.props)|(\.editorconfig)' | wc -l   # -> 0
```

**What `packages.config` does and does not give you, stated exactly, because this is easy to get backwards.** The format is a **flat, exact list of every package installed into the project — transitive ones included**, each at a single exact version, with no ranges to resolve at restore time. Transitive entries sit in the list as first-class pins: MVC 5 declares `Antlr` `3.4.1.9004` [src/MVC5/MvcMusicStore/packages.config:3], `WebGrease` `1.5.2` [:30], `Owin` `1.0` [:28] and `Microsoft.Web.Infrastructure` `1.0.0.0` [:25], none of which any application source calls — they are in the manifest because the minification and OWIN stacks depend on them (§3.1.1, §3.1.2). Restore does not re-resolve a graph from package metadata; it installs the list. The two committed payloads confirm the same property from the other side: each contains exactly the packages its manifest names and nothing else (§7.2).

So the gap is **not** that listed versions float. It is everything the format records nothing about:

- **Dependency-relationship provenance.** Nothing says which package required which, so a direct pin and a transitive one are indistinguishable in the file — §3.2.5's `Antlr3.Runtime` case is the same gap seen from the assembly side — and no removal can be reasoned about from the manifest alone.
- **Content hashes.** No entry carries one, so a package with the expected id and version but different content restores without complaint.
- **Source immutability.** No entry binds where it resolves from, and no source is configured anywhere (§6), so an entry's identity rests on id and version alone.
- **Restore-time enforcement.** There is no locked mode: nothing fails a restore because the resolved set differs from a previously recorded one, and nothing records the resolved set to compare against.

So the repository's reproducibility position, stated as it actually is: the **package set** — direct and transitive together — is fully enumerated and exactly versioned in the manifests, and in the two editions that commit their payloads it is corroborated by them (§7.2); what is unpinned is everything *around* those versions — the provenance of each entry, the content behind each id and version, the source they resolve from (§6), and any enforcement at the moment of restore. A lockfile records the resolved set with a content hash per entry and, in locked mode, fails a restore whose resolution differs from it; a committed source configuration fixes where those entries may be served from. Neither substitutes for the other, which is why the target-side remedy is both. This is supply-chain debt — its severity, remediation and owner are assigned by [08 — Technical Debt Register](08-technical-debt-register.md), and the target-side locking decision is owned by [04 — .NET 8 Migration Strategy](04-dotnet8-migration-strategy.md).

### 7.2 Two editions commit their restored packages; one commits nothing

```bash
git ls-files | grep -c '/packages/'    # -> 215
```

**215 tracked files** sit under the two committed `packages/` trees, despite an ignore rule matching both of those directories. The rule that matches them is `Packages/` [.gitignore:33]: it has no interior separator, so it matches a directory of that name at **any** depth, and it reaches these lowercase directories only because `git config core.ignorecase` reports `true` on this checkout — on a case-sensitive host no rule matches them. `packages/*` [.gitignore:15] is not that rule, because its interior separator anchors it to the repository root, so it cannot reach `src/MVC4/MvcMusicStore/packages` or `src/MVC3/MvcMusicStore-Completed/packages`. Neither rule untracks a file already added, which is what leaves these 215 tracked; `git check-ignore -v --no-index` reports the rule per path (§10.1). The distribution:

| Edition | Tracked files under `packages/` | Package folders | Matches its pin count? | `.dll`/`.exe` files |
| --- | --- | --- | --- | --- |
| MVC 4 — `src/MVC4/MvcMusicStore/packages/` | 169 | 29, plus `repositories.config` | **yes** — all 29 pins present at their exact pinned versions | 31 |
| MVC 3 — `src/MVC3/MvcMusicStore-Completed/packages/` | 46 | 6, plus `repositories.config` | **yes** — all 6 pins present at their exact pinned versions | 1 |
| MVC 5 | **0** | — | n/a | 0 |

```bash
git ls-files 'src/MVC4/MvcMusicStore/packages/*' | wc -l                 # -> 169
git ls-files 'src/MVC3/MvcMusicStore-Completed/packages/*' | wc -l       # -> 46
git ls-files | grep '/packages/' | grep -cE '\.(dll|exe)$'               # -> 32
```

Three dependency-relevant observations, kept separate from the debt framing that deliverable 08 owns:

- **The committed payloads corroborate the pins exactly.** Every package folder name is `<id>.<version>` at the version its manifest declares, in both editions. This is a second, independent confirmation of the version strings in §3.2 and §3.3 — the manifest and the on-disk payload agree, so a transcription error in this document would be caught twice.
- **32 committed `.dll`/`.exe` files sit inside those payloads.** Executable content is tracked in source control and is not covered by any lockfile or hash manifest. Technical Specification §3.3 identifies the same governance risk, cited here as a secondary cross-reference alongside the repository evidence above.
- **MVC 5, the sole migration source, commits nothing.** It therefore cannot be built from a clean checkout without a working restore — and, per §6, without a source that the repository does not specify. The build consequence is owned by [10 — Build and Deployment Requirements](10-build-and-deployment-requirements.md).

---

## 8. Version-risk posture

### 8.1 The finding, stated as a class

The client-side and OAuth pins in this repository are 2011–2013 releases. The exact pins in that group, with the manifest that declares each:

| Package | Version | Manifest |
| --- | --- | --- |
| DotNetOpenAuth.AspNet, .Core, .OAuth.Consumer, .OAuth.Core, .OpenId.Core, .OpenId.RelyingParty | `4.0.3.12153` | [src/MVC4/MvcMusicStore/packages.config:3-8] |
| Newtonsoft.Json | `4.5.6` | [src/MVC4/MvcMusicStore/packages.config:30] |
| Newtonsoft.Json | `5.0.6` | [src/MVC5/MvcMusicStore/packages.config:27] |
| jQuery | `1.5.1` | [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/packages.config:3] |
| jQuery | `1.7.1.1` | [src/MVC4/MvcMusicStore/packages.config:10] |
| jQuery | `1.10.2` | [src/MVC5/MvcMusicStore/packages.config:6] |
| jQuery.UI.Combined | `1.8.11` | [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/packages.config:6] |
| jQuery.UI.Combined | `1.8.20.1` | [src/MVC4/MvcMusicStore/packages.config:11] |
| bootstrap | `3.0.0` | [src/MVC5/MvcMusicStore/packages.config:4] |

Two aggravating factors are repository facts rather than inferences: every one of these client-side libraries is **self-hosted from the application's own `Scripts/` and `Content/` directories** rather than loaded from a versioned CDN (§3.1.4, §3.2.4), and none is served with a **Subresource Integrity** attribute or under a **Content Security Policy** — the repository declares neither in any tracked text file, which is the precise scope the command below establishes.

That second half is a repository-wide **absence**, so under the contract in §1.4 it carries its reproducing command rather than a line citation, and the command's scope is stated because scope is what makes an absence claim mean anything:

```bash
# Subresource Integrity and Content Security Policy, repository-wide.
# SCOPE: every tracked file except docs/ -- 603 of the 616 tracked at this commit -- with the
# search restricted to the 404 of those that are text. Both committed packages/ payload trees
# (§7.2) are INCLUDED, deliberately: a package payload can carry an unrelated `integrity=`
# string, and a search that excluded them could not tell an absent attribute from an excluded
# one. The ONE path exclusion is docs/, because this assessment's own prose names both
# mechanisms and would match itself; it removes exactly the 13 files of this deliverable set.
#
# TWO MECHANICAL PROPERTIES, both of which an earlier form of this command lacked:
#   -z / xargs -0   One tracked path contains spaces -- `src/MVC3/MVC Music Store - Tutorial
#                   - v3.0.pdf` -- and an unquoted `git ls-files | xargs grep` splits it into
#                   five arguments, none of which exists. That form searched five phantom
#                   paths, never searched the PDF, and printed five `No such file` errors, so
#                   it could not support an "every tracked file" scope claim at all.
#   grep -I         Skips the 199 binary files, so a byte sequence inside a committed assembly,
#                   database file or PDF cannot be reported as a *declaration*. Declarations of
#                   both mechanisms are text by construction: a response header set in
#                   configuration, or an attribute in markup.
git ls-files -z -- ':(exclude)docs/' \
  | xargs -0 grep -lIiE 'content-security-policy|integrity=' | wc -l                     # -> 0

# How the 603 / 404 / 199 split is obtained:
git ls-files -- ':(exclude)docs/' | wc -l                                                # -> 603
git ls-files -z -- ':(exclude)docs/' | xargs -0 grep -lI '' | wc -l                      # -> 404   (text)
#                                                                        603 - 404       # -> 199   (binary)

# The two channels a policy could arrive on other than a response header, checked separately
# because neither would be found by the pattern above. Both use pathspec exclusion rather than
# a `grep -v` in the middle of the pipeline, which would have broken the NUL delimiting:
git ls-files -z -- '*.cshtml' '*.html' ':(exclude)*/packages/*' \
  | xargs -0 grep -lIiE 'http-equiv' | wc -l                                             # -> 0
git ls-files -z -- '*.config' '*.Config' ':(exclude)*/packages/*' \
  | xargs -0 grep -lIiE '<customHeaders' | wc -l                                         # -> 0
```

All three return **zero**, and the split commands return **603** and **404** as annotated. Three notes on what that does and does not establish.

The first is about scope, because the correction above changes it. The claim is now over **the 404 text files outside `docs/`**, not over all 616 tracked files, and the 199 binary files are excluded **by the search tool** rather than by a path filter. That is the honest form: `grep -I` declines to report a match inside a binary, so a claim about binaries would be a claim the command cannot make either way. Neither mechanism can be *declared* in a compiled assembly or a database file, so nothing is lost — but the scope sentence now matches the command, which the previous one did not.

The second is about the nearest thing to a match, and it is a near-miss rather than a control. A case-insensitive `headers.add(` appears in **nine** text files: the four vendored jQuery UI copies — `src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Scripts/jquery-ui-1.8.11.js` and its minified twin, and the MVC 4 pair at `1.8.20` — their **four** duplicates inside the committed `packages/` payloads, which this search deliberately includes, and one XML documentation file in a package payload, `src/MVC4/MvcMusicStore/packages/Microsoft.Net.Http.2.0.20710.0/lib/net40/System.Net.Http.xml`. None of the nine is a header being set. In the jQuery UI files the text is `this.headers.add(this.headers.next())` [src/MVC4/MvcMusicStore/Scripts/jquery-ui-1.8.20.js:5587], [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Scripts/jquery-ui-1.8.11.js:4472] — the accordion widget calling jQuery's collection `.add()` on a set of DOM heading elements, with no HTTP involvement of any kind; in the XML file it is API documentation for `System.Net.Http.Headers.HttpHeaders.Add` [src/MVC4/MvcMusicStore/packages/Microsoft.Net.Http.2.0.20710.0/lib/net40/System.Net.Http.xml:1291]. An earlier form of this paragraph reported four files and described them as setting request headers on an `XMLHttpRequest`; both halves were wrong, and the searches above are written to the two response-side mechanisms precisely so that neither a widget's DOM collection nor a payload's API documentation can be mistaken for a control.

The third is about what the repository governs. The claim is about what the **repository** declares: a policy or an integrity attribute injected by a reverse proxy or an IIS site-level configuration outside the repository would not appear here, which is why the sentence above says the repository declares neither rather than that neither is ever present. No such external configuration is committed, and none is referenced by any tracked file.

Three further *groups* of pins belong to the same era and the same class even though they are not client-side: `Owin` `1.0`, the **nine** `Microsoft.Owin*` `2.0.0` packages [src/MVC5/MvcMusicStore/packages.config:16-24] — **five** infrastructure pins (`Microsoft.Owin`, `.Host.SystemWeb`, `.Security`, `.Security.Cookies`, `.Security.OAuth`) plus the **four** dormant social providers §3.1.2 accounts for separately — and `Microsoft.Web.Infrastructure` `1.0.0.0`, whose four-part version reflects its 2012 origin. Nine is the count the cited range holds and the count a `Microsoft.Owin*` wildcard returns; the five is the live-infrastructure subset, and conflating the two would understate the deployed surface by four packages.

### 8.2 Observed advisory evidence, and exactly what it is

A scan result exists and it is not one this document had to go looking for: **NuGet's own restore-time audit emits it, and it is reproduced by restoring each solution.** Two different kinds of claim follow from that, and they are labelled separately here because exactly one of them is a property of the repository while the other is a dated observation about a database that moves:

**Repository property — the pin set the audit lands on.** What the repository fixes is the set of pins themselves: **eight distinct package identifiers across 14 distinct (package, version) pairs**, all of them within the aged group of §8.1, and each transcribed from the manifest that declares it — `bootstrap` `3.0.0` [src/MVC5/MvcMusicStore/packages.config:4]; `jQuery` `1.10.2` [src/MVC5/MvcMusicStore/packages.config:6], `1.7.1.1` [src/MVC4/MvcMusicStore/packages.config:10] and `1.5.1` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/packages.config:3]; `jQuery.UI.Combined` `1.8.20.1` [src/MVC4/MvcMusicStore/packages.config:11] and `1.8.11` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/packages.config:6]; `jQuery.Validation` `1.11.1` [src/MVC5/MvcMusicStore/packages.config:7], `1.9.0.1` [src/MVC4/MvcMusicStore/packages.config:12] and `1.8.0` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/packages.config:5]; `Microsoft.AspNet.Identity.Owin` `1.0.0` [src/MVC5/MvcMusicStore/packages.config:10]; `Microsoft.Owin` `2.0.0` [src/MVC5/MvcMusicStore/packages.config:16]; `Microsoft.Owin.Security.Cookies` `2.0.0` [src/MVC5/MvcMusicStore/packages.config:19]; and `Newtonsoft.Json` `5.0.6` [src/MVC5/MvcMusicStore/packages.config:27] and `4.5.6` [src/MVC4/MvcMusicStore/packages.config:30]. Those fourteen are **14 of the 63 pins** counted in §2, and the pairs hold for exactly as long as the three manifests say what they say. The measured tallies behind that arithmetic — 8 distinct identifiers, 14 distinct pairs, and 24 distinct advisory identifiers across the 43 warning incidences — are reproducible from the retained output in §8.2.1.

**Dated observation — that a warning exists for a given pin at all, its severity, and how many there are.** Everything the audit *says about* those pins belongs to the day it was asked. `NU1902` (moderate severity) and `NU1903` (high severity) warnings, each carrying a GitHub advisory identifier, are raised against them when these three solutions are restored against nuget.org — but an advisory can be published or withdrawn without a line of this repository changing, so **which** of the pins are named, at **what** severity, and **how many** warnings appear are all properties of the advisory database at the moment of the run, of the client that queries it, and of the package sources that client is configured with. Package membership is therefore dated in exactly the way the count is, and the earlier draft of this section was wrong to present it as holding on any host that can reach nuget.org. One behavioural claim does survive that limitation: **the warnings are advisory, not fatal — every restore exits `0` and nothing fails**, which follows from the audit's default warning level rather than from the database, and holds while the repository declares no audit or warnings-as-errors setting of its own (it declares no `Directory.Build.props` at all — §7.1, §10.1). An exact count is otherwise reported here only with the provenance that makes it interpretable, and a later run may legitimately differ without anything in the repository having changed.

| Provenance of the counts below | Value |
| --- | --- |
| Runs | **2026-08-27** (authoring); **2026-08-28T16:14:43Z** (verification re-run, timestamp from `date -u +%Y-%m-%dT%H:%M:%SZ` beside the restores); and a further run on **2026-08-28** that produced the retained output embedded in §8.2.1. Every run produced the same figures |
| Host | the Windows verification host of §1.4. The client was invoked both from `PowerShell` and through the Git-for-Windows `bash` of §1.4; the warnings emitted are identical and only the **log encoding** differs — see the `Log capture` row, which is the difference that can make a correct command look like it produced nothing |
| Client | **NuGet 6.11.1.2**, as printed by `nuget help` (first line: `NuGet Version: 6.11.1.2`), and printed again as the first line of the restore log itself when `-Verbosity detailed` is passed |
| MSBuild | **17.14.51.32402**, auto-detected by the client rather than chosen, and reported by the `MSBuild auto-detection:` line that opens every restore log. It is part of the observation because the client drives restore graph generation through it |
| Effective package sources | exactly one registered source — `nuget.org` → `https://api.nuget.org/v3/index.json` — as printed by `nuget sources list`, resolved from the host's own user-level configuration, which each restore echoes back under `NuGet Config files used:` (`%AppData%\NuGet\NuGet.Config`). The repository configures **no** source (§6), so this is part of the observation and not of the repository. The log's `Feeds used:` block additionally names the host's local package-cache directory, which is a cache rather than a registered source and is likewise a property of the host |
| Advisory database snapshot | the strongest provenance available for a time-varying result, because the client names the snapshot it actually consulted: base **`2026.08.27.05.48.31`** and update **`2026.08.28.11.48.36`**, fetched as `https://api.nuget.org/v3-vulnerabilities/2026.08.27.05.48.31/vulnerability.base.json` and `…/2026.08.28.11.48.36/vulnerability.update.json` and logged with a `GET`, `CACHE` or `OK` prefix according to the state of the client's HTTP cache. Two runs quoting the same pair are comparable; two runs quoting different pairs are not, however close their dates |
| Install scope | the audit reports on the **declared** graph, not on what a restore happens to install. Measured: MVC 3's restore printed `All packages listed in packages.config are already installed.` — its payload is committed (§7.2) — and still emitted all **14** of its warnings. So the 43 needs no special clone state to reproduce — a faithful clone with no payload cleared and nothing pre-warmed gives the same figure — which isolates the one variable that does move it: the advisory snapshot named in the row above |
| Log capture | **byte-faithful redirection**. The `grep` expectations below hold only if the log is written as bytes: a PowerShell 5.1 `>` redirect encodes the log as **UTF-16LE**, in which `grep` matches nothing and reports `0` even though the warnings are present in the file. The note under the command block states the consequence and the fix |
| Working tree | a **disposable clone** of the checkout, outside the authoritative repository — required, because three of these commands are package-mutating (see the note on the command block below) |
| Commands | the three `nuget restore` invocations below, one per solution; each exited `0` |
| Advisory source | the GitHub Advisory Database as served by nuget.org's audit at the snapshot named above — **time-varying** |

On every run recorded above the three restores together produced **43 advisory warnings** — **9** `NU1903` (high) and **34** `NU1902` (moderate) — against those 14 pins, carrying **24 distinct advisory identifiers**. All 43 are retained verbatim in §8.2.1 — the raw warning lines as the client emitted them, the command that extracted them, and the digests of the three logs they were cut from — and indexed there row by row, so the figures in this section are checkable against retained output rather than against memory of a run.

> **These three commands are MUTATING: they are the three package-mutating commands this document quotes, and the only ones that can reach package payloads.** They are not, however, the only operations in this workflow that change anything — the surrounding `git clone`, log redirections and guarded `rm -rf` all mutate something too, and §10.2 classifies each one and where its effect lands. What distinguishes these three is the blast radius: `nuget restore` downloads package payloads and **writes them into the working tree** it is run in, so it is not an evidence command. Run it **only in a disposable clone or scratch checkout outside the authoritative repository**, and **never in place against the assessed checkout**, whose tracked content this assessment is required to leave byte-identical (§1.3). **The block below enforces that rule mechanically rather than by instruction, and it is the one block in this document whose safety property is the point:** it is **fail-closed** — `$SCRATCH` is validated first, `set -euo pipefail` stops it at the first failure, every path is rooted at the clone and the only `cd` calls are inside command substitutions, which cannot change the directory the caller is standing in, each restore's exit code is checked before any count is parsed, and the cleanup deletes only the directory this run created. A block that continued past a failed `git clone` or an unset `$SCRATCH` would run these three restores in the caller's current directory, and what they write there is package payload. Measured on the 2026-08-28 run, in a throwaway clone: the three restores created two directories that did not exist before — `src/MVC5/packages` (167 files, 46.7 MB) and `src/MVC4/packages` (231 files, 28.6 MB) — while MVC 3's committed tree gained nothing, because all six of its pins are already present. **`git status --porcelain` stayed empty throughout**, because both of those directories are matched by `Packages/` [.gitignore:33] — the depth-independent rule, which reaches them because `git config core.ignorecase` reports `true` on this host — and not by `packages/*` [.gitignore:15], whose interior separator anchors it to the repository root: a porcelain status is therefore *not* a check that these commands left the tree alone. On a case-sensitive host neither rule matches a nested lowercase `packages`, so the payload would be visible to porcelain instead — which changes what the check shows, not the rule that these three commands belong in a disposable clone. Use `git status --porcelain --ignored` and `git clean -ndX`, or discard the clone, which is the point of using one. **That distinction is not hypothetical in this assessment.** Restore and build operations run in place earlier in the assessment left **eight ignored trees** in the assessed checkout — two of them the `packages/` payload these three restores produce. Those operations were unqualified historical runs that produced gitignored residue and nothing else: none of them is build evidence, because none recorded the fields [10 — Build and Deployment Requirements](10-build-and-deployment-requirements.md) requires of it, and every build outcome and status — MVC 5's included, which remains **blocked pending a Windows verification run** — is that deliverable's to state. §1.3 records what the trees were, that all eight have been removed, and the four-command check that establishes their absence; bare porcelain reported a clean tree for the whole time they existed.

```bash
# MUTATING — network access required; each restore writes packages/ into the tree
# it runs in. Run it in a disposable clone, never in the authoritative checkout.
# $SCRATCH is any private directory outside the authoritative repository.
# Fail-closed by construction: $SCRATCH is validated before anything else, the
# block stops at the first failure, and every path is rooted at the clone, so no
# restore can run in the caller’s current directory. §10.2 repeats this same
# workflow with the ignored-aware checks that show what the restores wrote.
set -euo pipefail

# 1 - $SCRATCH must be set, non-empty, absolute, an existing directory, and
#     outside this repository. A restore rooted inside the assessed checkout is
#     the one outcome this workflow has to make impossible, so it is checked
#     first and nothing else runs until it passes.
: "${SCRATCH:?set SCRATCH to an existing private directory outside the repository}"
case $SCRATCH in
  /*) ;;
  *) echo "SCRATCH must be an absolute path: $SCRATCH" >&2; exit 1 ;;
esac
if [ ! -d "$SCRATCH" ]; then
  echo "SCRATCH is not an existing directory: $SCRATCH" >&2
  exit 1
fi
# Containment is tested twice, because a path has more than one spelling on a
# Windows host and one string comparison would let the other through. First git’s
# own answer, which is spelling-proof and holds at any depth inside the work tree:
REPO_TOP=$(git rev-parse --show-toplevel)   # fails here if this is not a repository
if [ "$(git -C "$SCRATCH" rev-parse --show-toplevel 2>/dev/null || true)" = "$REPO_TOP" ]
then
  echo "SCRATCH is inside the assessed repository: $SCRATCH" >&2
  exit 1
fi
# Then a physical-path test with both sides normalised the same way, which also
# catches a nested repository sitting inside the checkout:
REPO_REAL=$(cd "./$(git rev-parse --show-cdup)" && pwd -P)
SCRATCH_REAL=$(cd "$SCRATCH" && pwd -P)
case $SCRATCH_REAL in
  "$REPO_REAL" | "$REPO_REAL"/*)
    echo "SCRATCH is inside the checkout at $REPO_REAL: $SCRATCH_REAL" >&2
    exit 1 ;;
esac

# 2 - The clone. A pre-existing target is refused, so the directory the cleanup
#     removes is always one this run created. This shell never changes directory:
#     the only `cd` calls above and below are inside command substitutions, which
#     cannot outlive the subshell they run in, so the working directory the caller
#     started in is still the working directory at the end. Every path from here on
#     is rooted at $AUDIT, which is what makes it impossible for a restore to
#     inherit the caller’s working directory.
AUDIT=$SCRATCH_REAL/mvcmusicstore-audit
if [ -e "$AUDIT" ]; then
  echo "refusing to reuse an existing path: $AUDIT" >&2
  exit 1
fi
git clone --no-hardlinks "$REPO_TOP" "$AUDIT"
if [ ! -d "$AUDIT/.git" ]; then
  echo "clone produced no working tree: $AUDIT" >&2
  exit 1
fi

# 3 - The three package-mutating commands, each with its exit code checked, so no
#     count is ever parsed from a partial log. A failure here exits before the
#     cleanup below, deliberately: the clone and its logs stay in $SCRATCH to be
#     read, and are discarded by hand. -Verbosity detailed puts the client version
#     into the log itself; the warnings appear at either verbosity.
restore() {          # $1 = solution path inside the clone, $2 = log file name
  if ! nuget restore "$AUDIT/$1" -NonInteractive -Verbosity detailed \
       > "$AUDIT/$2" 2>&1; then
    echo "restore FAILED: $1 (log: $AUDIT/$2) - stopping before any parsing" >&2
    exit 1
  fi
  echo "exit=0  $1"
}
restore src/MVC5/MvcMusicStore.sln                         restore-mvc5.log
restore src/MVC4/MvcMusicStore.sln                         restore-mvc4.log
restore src/MVC3/MvcMusicStore-Completed/MvcMusicStore.sln  restore-mvc3.log

L5=$AUDIT/restore-mvc5.log
L4=$AUDIT/restore-mvc4.log
L3=$AUDIT/restore-mvc3.log
ALL=$AUDIT/nu190-all.log

# 4 - The warning lines, per log and in total, reached only if all three restores
#     exited 0. The concatenation is named *.log deliberately: *.log is gitignored
#     [.gitignore:25], so none of these files can dirty even the clone’s status.
grep -hc 'NU190' "$L5" "$L4" "$L3"                     # -> 15, 14, 14
cat "$L5" "$L4" "$L3" > "$ALL"
grep -c 'NU190' "$ALL"                                 # -> 43   (incidences)
grep -c 'NU1903' "$ALL"                                # -> 9    (high)
grep -c 'NU1902' "$ALL"                                # -> 34   (moderate)
grep -o 'GHSA-[a-z0-9-]*' "$ALL" | sort -u | wc -l     # -> 24   (distinct advisories)
sed -n "s/.*Package '\([^']*\)' \([^ ]*\) has.*/\1 \2/p" "$ALL" | sort -u | wc -l   # -> 14 pairs
# The provenance every count must be quoted with (§8.2 table, §10.2 commands), and the
# identity of the three logs the raw block of §8.2.1 is cut from:
grep -h 'vulnerability\.\(base\|update\)\.json' "$ALL" | sort -u  # the advisory-DB snapshot
head -1 "$L5"                                          # NuGet Version: 6.11.1.2
sha256sum "$L5" "$L4" "$L3"                             # the three digests of §8.2.1
wc -c "$L5" "$L4" "$L3"                                 # -> 22611, 25413, 11639

# Discard the clone. The path is re-derived and re-checked first, so a $SCRATCH
# that changed mid-run, or a symlink swapped underneath it, cannot turn this into
# an rm -rf of anything but the directory step 2 created.
AUDIT_REAL=$(cd "$AUDIT" && pwd -P)
case $AUDIT_REAL in
  "$SCRATCH_REAL"/mvcmusicstore-audit) ;;
  *) echo "cleanup refused: $AUDIT_REAL is not the path this run created" >&2
     exit 1 ;;
esac
if [ ! -d "$AUDIT_REAL/.git" ]; then
  echo "cleanup refused: $AUDIT_REAL is not the clone this run created" >&2
  exit 1
fi
rm -rf "$AUDIT_REAL"
```

**Why `-Verbosity detailed`, and the trap that is not the verbosity.** The flag is here so that the log records the client version in its own first line — `NuGet Version: 6.11.1.2` — putting the client and the counts in one artifact; the warnings and the advisory-database snapshot lines are emitted at default verbosity too, and the counts are identical either way (measured: 15, 14, 14 with and without the flag). What does silently produce a wrong answer is **how the log is captured**. On this Windows host a PowerShell 5.1 `>` redirect writes the log as **UTF-16LE**, and `grep` finds no match in UTF-16 text: the command above then prints `0`, `0`, `0` against a file that visibly contains all 43 warnings. Capture through the Git-for-Windows `bash` of §1.4, or `cmd /c "… > log 2>&1"`, or re-encode (`iconv -f UTF-16LE`), or search with a UTF-16-aware tool (`Select-String`). A reader who sees zeroes should check the first two bytes of the log for a `FF FE` byte-order mark before concluding that the audit found nothing — a false negative here reads as a clean bill of health, which is the worst direction for an error of this kind to point.

Per edition, as observed on every run recorded above — figures dated, not reproducible-by-construction:

| Edition | Warnings observed | High (`NU1903`) | Moderate (`NU1902`) | Pins named |
| --- | --- | --- | --- | --- |
| MVC 5 | 15 | 6 | 9 | `bootstrap` `3.0.0`; `jQuery` `1.10.2`; `jQuery.Validation` `1.11.1`; `Microsoft.AspNet.Identity.Owin` `1.0.0`; `Microsoft.Owin` `2.0.0`; `Microsoft.Owin.Security.Cookies` `2.0.0`; `Newtonsoft.Json` `5.0.6` |
| MVC 4 | 14 | 2 | 12 | `jQuery` `1.7.1.1`; `jQuery.UI.Combined` `1.8.20.1`; `jQuery.Validation` `1.9.0.1`; `Newtonsoft.Json` `4.5.6` |
| MVC 3 | 14 | 1 | 13 | `jQuery` `1.5.1`; `jQuery.UI.Combined` `1.8.11`; `jQuery.Validation` `1.8.0` |
| **Total** | **43** | **9** | **34** | **14 distinct pins, 8 distinct package identifiers** |

**This document reports those identifiers only because a tool emitted them, verbatim — in output retained in full at §8.2.1, and re-runnable by any reader subject to the dating above. It has looked up, inferred, matched or constructed no identifier of its own** — that distinction is the whole of the discipline here, and an inventory whose other numbers are exact cannot afford one fabricated identifier.

Three of the observed results are worth pulling out, because each connects to a finding stated earlier on independent grounds:

- **`Microsoft.Owin.Security.Cookies` `2.0.0` carries a high-severity advisory, and it is the one authentication middleware the application actually enables** (§3.1.2). `Microsoft.Owin` `2.0.0` and `Microsoft.AspNet.Identity.Owin` `1.0.0` are flagged high as well. This is the most operationally significant group in the table: it is on the live sign-in path, not in dormant scaffolding.
- **`Newtonsoft.Json` carries a high-severity advisory in both editions — `5.0.6` and `4.5.6` — and §3.1.3 established that no application source calls it.** A package with no consumer is nonetheless restored, copied to `bin` and deployed, so it contributes exposure and delivers nothing. That makes its removal the cheapest item in the table.
- **`bootstrap` `3.0.0`, `jQuery` and `jQuery.UI.Combined` account for every one of the 34 moderate warnings** — `bootstrap` 6, `jQuery` 14 across its three pinned versions and `jQuery.UI.Combined` 14 across its two, on the runs above and row by row in §8.2.1 — and per §3.1.4 and §3.2.4 those are precisely the packages whose payloads are already **vendored into the tree**, so the flagged code is committed to source control and served to browsers today, without SRI or a CSP. Reproduce the split from the saved run output of §10.2: `grep -h 'NU1902' restore-mvc5.log restore-mvc4.log restore-mvc3.log | sed "s/.*Package '\([^']*\)'.*/\1/" | sort | uniq -c` → `6 bootstrap`, `14 jQuery`, `14 jQuery.UI.Combined`.

**What this evidence is not**, stated plainly so nobody over-reads it:

- It is an audit of the **direct pins declared in the three manifests**, not a full software-composition analysis. Per §7.1 the transitive graph is unpinned, so it describes neither transitive exposure nor what a future restore would resolve.
- It is **time-varying**. The advisory database moves, so the counts above are the result *on the stated runs, with the stated client, package sources and advisory-database snapshot* — the provenance table above — and not a stable property of the repository. A later run is expected to give a different answer, and a different answer is not a contradiction of this document; what is expected to hold is the labelled repository property above, the exit status, and the retained output of §8.2.1 as a record of what those runs said.
- It **cannot see the vendored copies** under `Scripts/`, `Content/` and `fonts/`. A package audit inspects packages; the committed asset files are ordinary tracked content, and no package audit reaches them. The vendored copies match the pinned versions (§3.1.4), so the exposure carries over — but that inference is mine and the audit does not make it.
- It **says nothing about the non-NuGet dependencies of §4.1** — the machine-wide ASP.NET MVC 3 Tools Update and SQL Server Compact 4.0 install — which no package audit can assess.
- And it is **not a dependency-scanning capability the repository possesses.** The repository has no scanning configuration whatsoever, verified:
- It counts **warnings raised against the pinned versions**, not advisory records held against the package identifiers. An advisory whose version range excludes the pinned version is not reported at all: nuget.org's audit data holds eleven entries for `bootstrap`, of which the six covering `3.0.0` are reported here. A count taken per identifier, or across every version of it, is a different number answering a different question, and this document does not mix the two.
- It is **restore output, not registry metadata.** Counting a package's advisory records from the registry's own metadata, or from the advisory database behind it, uses a different corpus and can use a different severity mapping, so it will not agree with the table above. **No such count is asserted anywhere in this document**; §8.3 treats that as revalidation rather than evidence.

```bash
git ls-files | grep -c '^\.github/'                                  # -> 0   (no GitHub workflows or config directory)
git ls-files | grep -icE 'dependabot|renovate'                       # -> 0   (no Dependabot, no Renovate)
git ls-files '*packages.config' | grep -v '/packages/' \
  | xargs grep -hiE 'analyzer|StyleCop|FxCop|SonarAnalyzer|Roslynator' | wc -l   # -> 0   (no analyzer package in any manifest)
git ls-files | grep -iE '(\.ruleset$)|(Directory\.Build\.props)'     # -> (no output)
```

That last point is the finding that survives whatever the advisory counts happen to be on a given day: **the evidence above is visible only to whoever happens to read restore output, and no build, tooling or CI artifact in this repository records it, gates on it or fails a build because of it.** The warnings scroll past on every clean restore — forty-three of them on the runs recorded above — and nothing in the repository keeps them. The only place they are kept is §8.2.1 immediately below: a dated snapshot inside this assessment, which is a record and not a gate.

#### 8.2.1 Retained evidence — the 43 raw warning lines verbatim, and the index derived from them

A count taken against a database that moves is worth what its retained output is worth, and until now this section had only its own arithmetic: a reader in six months, restoring against a different advisory snapshot, could neither confirm nor refute the 43/9/34 split or the 15/14/14 per-edition figures, because the runs behind them had left nothing behind. **The output is therefore retained here, in this document, in two forms that do different jobs.** The **raw warning lines, exactly as the client emitted them**, are the primary evidence, because only a complete capture can show that no warning was omitted, duplicated or reworded on its way into a table. The **43-row index** derived from them is the form in which a later run can be diffed against this one row by row, which a raw log is not. Both are embedded rather than filed alongside, because the deliverable set is thirteen files and no more ([`README.md` § 2.1 Thirteen files created; nothing modified](README.md#21-thirteen-files-created-nothing-modified)) and this evidence does not justify a fourteenth.

**Which of the two available routes this section took, stated rather than left to inference.** Evidence about a moving database leaves two honest options: retain the complete raw output together with the provenance that ties it to a run, or drop the exact historical totals and every figure derived from them. **This section takes the retention route.** The raw output exists, it is embedded below in full, and removing the totals would have discarded assessment value that §8.2, §9 and the deliverables consuming them actually use. Nothing is thereby promoted to a repository property: the labelled split of §8.2 — the pin set as a repository property, everything the audit says about those pins as a dated observation — stands unchanged.

**The raw evidence — all 43 warning lines, byte-faithful.** Grouped by the log each line came from, in the order the three restores run, with one comment line per edition stating how many lines follow it. Nothing is reformatted, reordered, prettified or truncated: the block is what the extraction command below wrote, and it is therefore also the record of the client’s own wording and line format.

```text
# ---- restore-mvc5.log : 15 warning lines, verbatim ----
WARNING: NU1902: Package 'bootstrap' 3.0.0 has a known moderate severity vulnerability, https://github.com/advisories/GHSA-3mgp-fx93-9xv5
WARNING: NU1902: Package 'bootstrap' 3.0.0 has a known moderate severity vulnerability, https://github.com/advisories/GHSA-4p24-vmcr-4gqj
WARNING: NU1902: Package 'bootstrap' 3.0.0 has a known moderate severity vulnerability, https://github.com/advisories/GHSA-ph58-4vrj-w6hr
WARNING: NU1902: Package 'bootstrap' 3.0.0 has a known moderate severity vulnerability, https://github.com/advisories/GHSA-3wqf-4x89-9g79
WARNING: NU1902: Package 'bootstrap' 3.0.0 has a known moderate severity vulnerability, https://github.com/advisories/GHSA-9v3m-8fp8-mj99
WARNING: NU1902: Package 'bootstrap' 3.0.0 has a known moderate severity vulnerability, https://github.com/advisories/GHSA-7mvr-5x2g-wfc8
WARNING: NU1902: Package 'jQuery' 1.10.2 has a known moderate severity vulnerability, https://github.com/advisories/GHSA-rmxg-73gg-4p98
WARNING: NU1902: Package 'jQuery' 1.10.2 has a known moderate severity vulnerability, https://github.com/advisories/GHSA-6c3j-c64m-qhgq
WARNING: NU1902: Package 'jQuery' 1.10.2 has a known moderate severity vulnerability, https://github.com/advisories/GHSA-jpcq-cgw6-v4j6
WARNING: NU1903: Package 'jQuery.Validation' 1.11.1 has a known high severity vulnerability, https://github.com/advisories/GHSA-jxwx-85vp-gvwm
WARNING: NU1903: Package 'Microsoft.AspNet.Identity.Owin' 1.0.0 has a known high severity vulnerability, https://github.com/advisories/GHSA-25c8-p796-jg6r
WARNING: NU1903: Package 'Microsoft.Owin' 2.0.0 has a known high severity vulnerability, https://github.com/advisories/GHSA-hxrm-9w7p-39cc
WARNING: NU1903: Package 'Microsoft.Owin' 2.0.0 has a known high severity vulnerability, https://github.com/advisories/GHSA-3rq8-h3gj-r5c6
WARNING: NU1903: Package 'Microsoft.Owin.Security.Cookies' 2.0.0 has a known high severity vulnerability, https://github.com/advisories/GHSA-3rq8-h3gj-r5c6
WARNING: NU1903: Package 'Newtonsoft.Json' 5.0.6 has a known high severity vulnerability, https://github.com/advisories/GHSA-5crp-9r3c-p9vr
# ---- restore-mvc4.log : 14 warning lines, verbatim ----
WARNING: NU1902: Package 'jQuery' 1.7.1.1 has a known moderate severity vulnerability, https://github.com/advisories/GHSA-2pqj-h3vj-pqgw
WARNING: NU1902: Package 'jQuery' 1.7.1.1 has a known moderate severity vulnerability, https://github.com/advisories/GHSA-rmxg-73gg-4p98
WARNING: NU1902: Package 'jQuery' 1.7.1.1 has a known moderate severity vulnerability, https://github.com/advisories/GHSA-q4m3-2j7h-f7xw
WARNING: NU1902: Package 'jQuery' 1.7.1.1 has a known moderate severity vulnerability, https://github.com/advisories/GHSA-6c3j-c64m-qhgq
WARNING: NU1902: Package 'jQuery' 1.7.1.1 has a known moderate severity vulnerability, https://github.com/advisories/GHSA-jpcq-cgw6-v4j6
WARNING: NU1902: Package 'jQuery.UI.Combined' 1.8.20.1 has a known moderate severity vulnerability, https://github.com/advisories/GHSA-hpcf-8vf9-q4gj
WARNING: NU1902: Package 'jQuery.UI.Combined' 1.8.20.1 has a known moderate severity vulnerability, https://github.com/advisories/GHSA-j7qv-pgf6-hvh4
WARNING: NU1902: Package 'jQuery.UI.Combined' 1.8.20.1 has a known moderate severity vulnerability, https://github.com/advisories/GHSA-9gj3-hwp5-pmwc
WARNING: NU1902: Package 'jQuery.UI.Combined' 1.8.20.1 has a known moderate severity vulnerability, https://github.com/advisories/GHSA-qqxp-xp9v-vvx6
WARNING: NU1902: Package 'jQuery.UI.Combined' 1.8.20.1 has a known moderate severity vulnerability, https://github.com/advisories/GHSA-wcm2-9c89-wmfm
WARNING: NU1902: Package 'jQuery.UI.Combined' 1.8.20.1 has a known moderate severity vulnerability, https://github.com/advisories/GHSA-h6gj-6jjq-h8g9
WARNING: NU1902: Package 'jQuery.UI.Combined' 1.8.20.1 has a known moderate severity vulnerability, https://github.com/advisories/GHSA-gpqq-952q-5327
WARNING: NU1903: Package 'jQuery.Validation' 1.9.0.1 has a known high severity vulnerability, https://github.com/advisories/GHSA-jxwx-85vp-gvwm
WARNING: NU1903: Package 'Newtonsoft.Json' 4.5.6 has a known high severity vulnerability, https://github.com/advisories/GHSA-5crp-9r3c-p9vr
# ---- restore-mvc3.log : 14 warning lines, verbatim ----
WARNING: NU1902: Package 'jQuery' 1.5.1 has a known moderate severity vulnerability, https://github.com/advisories/GHSA-2pqj-h3vj-pqgw
WARNING: NU1902: Package 'jQuery' 1.5.1 has a known moderate severity vulnerability, https://github.com/advisories/GHSA-rmxg-73gg-4p98
WARNING: NU1902: Package 'jQuery' 1.5.1 has a known moderate severity vulnerability, https://github.com/advisories/GHSA-q4m3-2j7h-f7xw
WARNING: NU1902: Package 'jQuery' 1.5.1 has a known moderate severity vulnerability, https://github.com/advisories/GHSA-6c3j-c64m-qhgq
WARNING: NU1902: Package 'jQuery' 1.5.1 has a known moderate severity vulnerability, https://github.com/advisories/GHSA-jpcq-cgw6-v4j6
WARNING: NU1902: Package 'jQuery' 1.5.1 has a known moderate severity vulnerability, https://github.com/advisories/GHSA-579v-mp3v-rrw5
WARNING: NU1902: Package 'jQuery.UI.Combined' 1.8.11 has a known moderate severity vulnerability, https://github.com/advisories/GHSA-hpcf-8vf9-q4gj
WARNING: NU1902: Package 'jQuery.UI.Combined' 1.8.11 has a known moderate severity vulnerability, https://github.com/advisories/GHSA-j7qv-pgf6-hvh4
WARNING: NU1902: Package 'jQuery.UI.Combined' 1.8.11 has a known moderate severity vulnerability, https://github.com/advisories/GHSA-9gj3-hwp5-pmwc
WARNING: NU1902: Package 'jQuery.UI.Combined' 1.8.11 has a known moderate severity vulnerability, https://github.com/advisories/GHSA-qqxp-xp9v-vvx6
WARNING: NU1902: Package 'jQuery.UI.Combined' 1.8.11 has a known moderate severity vulnerability, https://github.com/advisories/GHSA-wcm2-9c89-wmfm
WARNING: NU1902: Package 'jQuery.UI.Combined' 1.8.11 has a known moderate severity vulnerability, https://github.com/advisories/GHSA-h6gj-6jjq-h8g9
WARNING: NU1902: Package 'jQuery.UI.Combined' 1.8.11 has a known moderate severity vulnerability, https://github.com/advisories/GHSA-gpqq-952q-5327
WARNING: NU1903: Package 'jQuery.Validation' 1.8.0 has a known high severity vulnerability, https://github.com/advisories/GHSA-jxwx-85vp-gvwm
```

**The extraction command, so the step from log to block is reproducible rather than asserted.** Run inside the disposable clone of §10.2, after the three restores have succeeded and before the clone is discarded. The parser is a `grep` and nothing more, which is the point — a line either carries `NU190` and the client’s `has a known` phrasing or it is not a warning line, and no line is edited on the way through:

```bash
# Extract, in the order the restores run, with one header line per edition.
for log in restore-mvc5.log restore-mvc4.log restore-mvc3.log; do
  printf '# ---- %s : %s warning lines, verbatim ----\n' \
    "$log" "$(grep -c 'NU190.*has a known' "$log")"
  grep 'NU190' "$log" | grep 'has a known'
done > RAW-NU190-VERBATIM.txt
wc -l < RAW-NU190-VERBATIM.txt                       # -> 46   (3 headers + 43 warning lines)
grep -c '^WARNING: NU190' RAW-NU190-VERBATIM.txt     # -> 43   (the incidence total of §8.2)
grep -c '^WARNING: NU1903' RAW-NU190-VERBATIM.txt    # -> 9    (high)
grep -c '^WARNING: NU1902' RAW-NU190-VERBATIM.txt    # -> 34   (moderate)
```

**Provenance — the three logs the block was cut from.** One log per edition, each the `-Verbosity detailed` capture of that edition's restore. Size and digest identify each log exactly, so a reader holding a log can establish whether the block above came from that log rather than from some other run:

| Source log | Bytes | SHA-256 | Warning lines contributed |
| --- | --- | --- | --- |
| `restore-mvc5.log` | 22,611 | `c497e9c570d75420bcfb2a6372e38d71c5c4539f9a99e8555a857e6c50180292` | 15 |
| `restore-mvc4.log` | 25,413 | `0abaae9fb27de20bcf781ddaabd8c7e047a45a3f0bf55de8d2066706c82a6d3e` | 14 |
| `restore-mvc3.log` | 11,639 | `8198da432743a24f2fbedcc625372e71e1d4810e48ebf0d6caf29b583f21486c` | 14 |
| **Total** | **59,663** | — | **43** |

```bash
sha256sum restore-mvc5.log restore-mvc4.log restore-mvc3.log   # the three digests above
wc -c restore-mvc5.log restore-mvc4.log restore-mvc3.log       # -> 22611, 25413, 11639
```

**Two facts about those three artifacts that a reader reproducing this section needs, and that a digest alone would not disclose.** First, the filenames above are the ones §10.2's block writes, which is the naming this document uses throughout; the run whose digests these are wrote its three logs under the names `det-mvc5.log`, `det-mvc4.log` and `det-mvc3.log`, the prefix recording the detailed-verbosity capture. The mapping is one to one and by edition, and a filename is not an input to a SHA-256 digest, so the identification is unaffected by which of the two names a holder's copy carries. Second, **these three logs are UTF-8**, which is why the byte-oriented `grep` above reads them at all. The same three restores redirected through a Windows PowerShell 5.1 stream operator produce **UTF-16LE** logs, and against those every `grep` in this section returns zero — not an error and not a small count, but zero, which reads exactly like a restore that emitted no advisories. §10.2 states the capture routes that avoid it; the check that settles it is the log's first two bytes, `FF FE` meaning UTF-16LE. The 43 warnings are a property of the restore rather than of the capture, and the same runs captured either way carry the same 43 lines.

**One capture detail that decides whether the extraction command above works at all, recorded because getting it wrong produces a silent zero.** These three logs are **UTF-8**, which is why a byte-oriented `grep` reads them. A restore redirected through a Windows PowerShell 5.1 stream operator is written **UTF-16LE** instead, and against such a file every `grep` in this section returns nothing — not an error, and not a small count, but zero, which reads exactly like a restore that emitted no advisories. Any reader reproducing this section must therefore confirm the capture's encoding before believing a zero: check for a `FF FE` byte-order mark, or decode explicitly. The 43 warnings are a property of the restore, not of the capture, and the same three restores captured either way carry the same 43 lines.

**What those digests do and do not establish, stated plainly, because a digest invites more confidence than this one earns.** They **identify** the three logs the block was cut from: anyone holding one of those logs can compute its digest, see whether it is the log this block came from, and re-run the extraction command above to reproduce the block line for line. They are **not** a signature over an archive that this repository holds, and they are not a chain of custody. The logs were written inside the disposable clone of §10.2 and that clone was discarded by design — the hygiene rule that §1.3 and §10.2 both state — so no file in this repository can be hashed to match them. What survives here is the block, its extraction command and these three identifiers, and together they answer the question a transformed table cannot: **the block is complete, so an omitted, duplicated or silently reworded warning would be visible in it**; its 43 `WARNING:` lines are the 43 incidences counted in §8.2; and its three header counts are the 15 / 14 / 14 per-edition figures of the table above.

**The index below is derived from that block; it is not the evidence.** Each of its 43 rows is one of the warning lines above, split into columns so the set can be sorted, filtered and diffed against a later run. The counts quoted in §8.2 and in §9 are derived from the block, and where index and block ever disagree, **the block governs**. The rows were produced mechanically from the same logs by a parser that refuses any `NU190` line it cannot match in full — so no row is retyped, and a malformed or unexpected line would have stopped the transcription rather than being silently dropped.

The `Edition restore` column is the solution whose restore emitted the row, in the order the three restores run. Warnings are emitted per restore, so a reader who restores one solution sees only that edition’s rows.

| # | Edition restore | Code | Package | Version | Severity | GitHub advisory |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | MVC 5 | `NU1902` | `bootstrap` | `3.0.0` | moderate | `https://github.com/advisories/GHSA-3mgp-fx93-9xv5` |
| 2 | MVC 5 | `NU1902` | `bootstrap` | `3.0.0` | moderate | `https://github.com/advisories/GHSA-4p24-vmcr-4gqj` |
| 3 | MVC 5 | `NU1902` | `bootstrap` | `3.0.0` | moderate | `https://github.com/advisories/GHSA-ph58-4vrj-w6hr` |
| 4 | MVC 5 | `NU1902` | `bootstrap` | `3.0.0` | moderate | `https://github.com/advisories/GHSA-3wqf-4x89-9g79` |
| 5 | MVC 5 | `NU1902` | `bootstrap` | `3.0.0` | moderate | `https://github.com/advisories/GHSA-9v3m-8fp8-mj99` |
| 6 | MVC 5 | `NU1902` | `bootstrap` | `3.0.0` | moderate | `https://github.com/advisories/GHSA-7mvr-5x2g-wfc8` |
| 7 | MVC 5 | `NU1902` | `jQuery` | `1.10.2` | moderate | `https://github.com/advisories/GHSA-rmxg-73gg-4p98` |
| 8 | MVC 5 | `NU1902` | `jQuery` | `1.10.2` | moderate | `https://github.com/advisories/GHSA-6c3j-c64m-qhgq` |
| 9 | MVC 5 | `NU1902` | `jQuery` | `1.10.2` | moderate | `https://github.com/advisories/GHSA-jpcq-cgw6-v4j6` |
| 10 | MVC 5 | `NU1903` | `jQuery.Validation` | `1.11.1` | high | `https://github.com/advisories/GHSA-jxwx-85vp-gvwm` |
| 11 | MVC 5 | `NU1903` | `Microsoft.AspNet.Identity.Owin` | `1.0.0` | high | `https://github.com/advisories/GHSA-25c8-p796-jg6r` |
| 12 | MVC 5 | `NU1903` | `Microsoft.Owin` | `2.0.0` | high | `https://github.com/advisories/GHSA-hxrm-9w7p-39cc` |
| 13 | MVC 5 | `NU1903` | `Microsoft.Owin` | `2.0.0` | high | `https://github.com/advisories/GHSA-3rq8-h3gj-r5c6` |
| 14 | MVC 5 | `NU1903` | `Microsoft.Owin.Security.Cookies` | `2.0.0` | high | `https://github.com/advisories/GHSA-3rq8-h3gj-r5c6` |
| 15 | MVC 5 | `NU1903` | `Newtonsoft.Json` | `5.0.6` | high | `https://github.com/advisories/GHSA-5crp-9r3c-p9vr` |
| 16 | MVC 4 | `NU1902` | `jQuery` | `1.7.1.1` | moderate | `https://github.com/advisories/GHSA-2pqj-h3vj-pqgw` |
| 17 | MVC 4 | `NU1902` | `jQuery` | `1.7.1.1` | moderate | `https://github.com/advisories/GHSA-rmxg-73gg-4p98` |
| 18 | MVC 4 | `NU1902` | `jQuery` | `1.7.1.1` | moderate | `https://github.com/advisories/GHSA-q4m3-2j7h-f7xw` |
| 19 | MVC 4 | `NU1902` | `jQuery` | `1.7.1.1` | moderate | `https://github.com/advisories/GHSA-6c3j-c64m-qhgq` |
| 20 | MVC 4 | `NU1902` | `jQuery` | `1.7.1.1` | moderate | `https://github.com/advisories/GHSA-jpcq-cgw6-v4j6` |
| 21 | MVC 4 | `NU1902` | `jQuery.UI.Combined` | `1.8.20.1` | moderate | `https://github.com/advisories/GHSA-hpcf-8vf9-q4gj` |
| 22 | MVC 4 | `NU1902` | `jQuery.UI.Combined` | `1.8.20.1` | moderate | `https://github.com/advisories/GHSA-j7qv-pgf6-hvh4` |
| 23 | MVC 4 | `NU1902` | `jQuery.UI.Combined` | `1.8.20.1` | moderate | `https://github.com/advisories/GHSA-9gj3-hwp5-pmwc` |
| 24 | MVC 4 | `NU1902` | `jQuery.UI.Combined` | `1.8.20.1` | moderate | `https://github.com/advisories/GHSA-qqxp-xp9v-vvx6` |
| 25 | MVC 4 | `NU1902` | `jQuery.UI.Combined` | `1.8.20.1` | moderate | `https://github.com/advisories/GHSA-wcm2-9c89-wmfm` |
| 26 | MVC 4 | `NU1902` | `jQuery.UI.Combined` | `1.8.20.1` | moderate | `https://github.com/advisories/GHSA-h6gj-6jjq-h8g9` |
| 27 | MVC 4 | `NU1902` | `jQuery.UI.Combined` | `1.8.20.1` | moderate | `https://github.com/advisories/GHSA-gpqq-952q-5327` |
| 28 | MVC 4 | `NU1903` | `jQuery.Validation` | `1.9.0.1` | high | `https://github.com/advisories/GHSA-jxwx-85vp-gvwm` |
| 29 | MVC 4 | `NU1903` | `Newtonsoft.Json` | `4.5.6` | high | `https://github.com/advisories/GHSA-5crp-9r3c-p9vr` |
| 30 | MVC 3 | `NU1902` | `jQuery` | `1.5.1` | moderate | `https://github.com/advisories/GHSA-2pqj-h3vj-pqgw` |
| 31 | MVC 3 | `NU1902` | `jQuery` | `1.5.1` | moderate | `https://github.com/advisories/GHSA-rmxg-73gg-4p98` |
| 32 | MVC 3 | `NU1902` | `jQuery` | `1.5.1` | moderate | `https://github.com/advisories/GHSA-q4m3-2j7h-f7xw` |
| 33 | MVC 3 | `NU1902` | `jQuery` | `1.5.1` | moderate | `https://github.com/advisories/GHSA-6c3j-c64m-qhgq` |
| 34 | MVC 3 | `NU1902` | `jQuery` | `1.5.1` | moderate | `https://github.com/advisories/GHSA-jpcq-cgw6-v4j6` |
| 35 | MVC 3 | `NU1902` | `jQuery` | `1.5.1` | moderate | `https://github.com/advisories/GHSA-579v-mp3v-rrw5` |
| 36 | MVC 3 | `NU1902` | `jQuery.UI.Combined` | `1.8.11` | moderate | `https://github.com/advisories/GHSA-hpcf-8vf9-q4gj` |
| 37 | MVC 3 | `NU1902` | `jQuery.UI.Combined` | `1.8.11` | moderate | `https://github.com/advisories/GHSA-j7qv-pgf6-hvh4` |
| 38 | MVC 3 | `NU1902` | `jQuery.UI.Combined` | `1.8.11` | moderate | `https://github.com/advisories/GHSA-9gj3-hwp5-pmwc` |
| 39 | MVC 3 | `NU1902` | `jQuery.UI.Combined` | `1.8.11` | moderate | `https://github.com/advisories/GHSA-qqxp-xp9v-vvx6` |
| 40 | MVC 3 | `NU1902` | `jQuery.UI.Combined` | `1.8.11` | moderate | `https://github.com/advisories/GHSA-wcm2-9c89-wmfm` |
| 41 | MVC 3 | `NU1902` | `jQuery.UI.Combined` | `1.8.11` | moderate | `https://github.com/advisories/GHSA-h6gj-6jjq-h8g9` |
| 42 | MVC 3 | `NU1902` | `jQuery.UI.Combined` | `1.8.11` | moderate | `https://github.com/advisories/GHSA-gpqq-952q-5327` |
| 43 | MVC 3 | `NU1903` | `jQuery.Validation` | `1.8.0` | high | `https://github.com/advisories/GHSA-jxwx-85vp-gvwm` |

**43 rows, 24 distinct advisory identifiers.** The gap is repetition, not double counting, and it has three causes, each visible in the table: one advisory can apply to more than one pinned version of the same package — `GHSA-jxwx-85vp-gvwm` accounts for rows 10, 28 and 43, one per pinned `jQuery.Validation` version; two packages in one graph can share an advisory — `GHSA-3rq8-h3gj-r5c6` is rows 13 and 14, `Microsoft.Owin` `2.0.0` and `Microsoft.Owin.Security.Cookies` `2.0.0` in the same MVC 5 restore; and the `jQuery.UI.Combined` set is the same seven identifiers against both pinned versions, rows 21-27 and 36-42 in the same order. Counting distinct advisories instead would understate the work, because each incidence is a separate pin to disposition; counting incidences instead would overstate the number of distinct defects. Both figures are given here so neither has to be inferred. Both reconcile against the raw block above rather than against each other: 43 `WARNING:` lines in, 43 rows out, 15 + 14 + 14 by edition, and every advisory identifier in this index appears verbatim in the block.

**This retained evidence exceeds what AAP §0.5.1 anticipated, and does not discharge its directive.** That section recorded the version-risk posture as a class of finding and explicitly declined to assert specific advisory identifiers "because no scanner is available in this environment". That premise turned out to be too pessimistic in one narrow respect: NuGet's own restore-time audit is a scanner, it was already reachable from the verification host, and the 43 rows above are its verbatim output. What it is not is the software-composition analysis §0.5.1 asked for — it sees direct pins only, not the transitive graph, and not the vendored copies of §3.1.4 — so **§0.5.1's accompanying directive to run a scanner during implementation stands unchanged**, and §8.3 states it.

### 8.3 What must still happen

The audit above is a floor, not a substitute for a scan. **Directive for the implementation phase:** run a full software-composition analysis against the restored graph — direct *and* transitive — on a host with network access, extend it to the vendored client-side assets that no package audit inspects, record its dated output with the tool and version that produced it, and add scanning configuration to the repository so the result is gated rather than merely printed. Any advisory identifier quoted downstream must come from that recorded output or from the reproducible restore audit above, never from recollection.

**And revalidate §8.2's figures rather than inheriting them.** They are a dated observation of a moving database, so an approver reading this later should expect a different number and should re-run the audit rather than trust the table — restating the client version, the source and the audit level with the new result, and re-asserting that the audit ran and ran undegraded, exactly as §8.2 does. A count of a package's advisory records taken from the registry or from the advisory database directly belongs to that revalidation too, and belongs there **as a re-observation rather than as a correction of the restore output**: the two use different corpora, so a disagreement between them is expected and is not evidence that either is wrong. What is not time-varying, and what needs no revalidation, is the class finding of §8.1 and the governance finding at the end of §8.2 — the pins are 2011–2013 releases whatever today's advisory count is, and nothing in the repository records or gates on the audit either way.

The output is an input to the risk register owned by [07 — Effort, Risks and Sequencing](07-effort-risks-sequencing.md), and the roadmap gate at which it runs is owned by [03 — Modernization Roadmap](03-modernization-roadmap.md). The eventual disposition of each of these pins — updated, replaced or removed — is owned by [04 — .NET 8 Migration Strategy](04-dotnet8-migration-strategy.md), and this document names no successor for any of them.

---

## 9. Summary of findings

Each row is an inventory finding of this document; the deliverable named in the last column carries its severity, remediation, effort or forward decision.

| # | Finding | Evidence | Consequence owned by |
| --- | --- | --- | --- |
| F-02-01 | 63 exact pins across three manifests, all public nuget.org identifiers, no ranges, no private package | §2, §3 | — (this document is the owner) |
| F-02-02 | MVC 5's manifest declares `net45` for all 28 pins while the project targets `v4.8` | §3.1.1 | 04, 10 |
| F-02-03 | Four dormant external-login provider packages ship in MVC 5; the fifth `Security.*` package is OAuth infrastructure, not a provider | §3.1.2 | 04, 08 |
| F-02-04 | `Newtonsoft.Json` is deployed in both MVC 5 and MVC 4 but called from no application source | §3.1.3 | 04 (removal), 12 (serialization) |
| F-02-05 | Six of MVC 5's pins, seven of MVC 4's and five of MVC 3's deliver content, not assemblies | §3.1.4, §3.2, §3.3 | 05 |
| F-02-06 | MVC 4's `20710` release stamp spans two different version strings across thirteen pins | §3.2.1 | 04 |
| F-02-07 | Four Web API packages serve a mapped route with zero `ApiController` implementations; one is a metapackage with no assembly and one is inert on `net45` | §3.2.2 | 08 |
| F-02-08 | Six DotNetOpenAuth packages plus the WebPages OAuth surface serve four commented-out registrations | §3.2.3 | 04, 08 |
| F-02-09 | MVC 4 depends on `Antlr3.Runtime` with no `Antlr` pin, taking it from the WebGrease payload | §3.2.5 | 04 |
| F-02-10 | MVC 3's six pins carry no `targetFramework` attribute at all | §3.3 | 10 |
| F-02-11 | MVC 3 depends on a machine-wide ASP.NET MVC 3 Tools Update install — `System.Web.Mvc` `3.0.0.0` is referenced with no `HintPath` and no pin | §4.1 | 10, 12 |
| F-02-12 | MVC 3 additionally depends on a machine-wide SQL Server Compact 4.0 install, declared only as a `providerName` | §4.1 | 10, 12 |
| F-02-13 | 20 / 23 / 23 references per edition resolve from the framework or machine-wide rather than from any package | §4.2 | 10 |
| F-02-14 | Package version and assembly version differ in four identified places, including WebGrease `1.5.2` binding to `1.5.2.14234` | §4.4 | 04 |
| F-02-15 | A 2012 NuGet 2.0 client (630,784 bytes, `2.0.30828.5`) is committed and is required by MVC 4's restore, with the download fallback disabled | §5.1 | 08, 10 |
| F-02-16 | MVC 5's solution declares a `.nuget` folder that does not exist; MVC 5 has no restore wiring of its own | §5.2 | 10 |
| F-02-17 | **No package source is configured anywhere.** Technical Specification §3.3 is corrected: the v2 endpoint is inside a comment | §6.1, §6.2 | 04 |
| F-02-18 | The effective source set is not knowable from the repository, and private feeds cannot be ruled out | §6.3 | 04, 08 |
| F-02-19 | No lockfile and no central version management in any edition; transitive resolution is not reproducible | §7.1 | 08, 04 |
| F-02-20 | 215 tracked files, including 32 `.dll`/`.exe`, committed under two `packages/` trees despite `Packages/` [.gitignore:33] matching them on this case-insensitive checkout — `packages/*` [.gitignore:15] is root-anchored and reaches no nested path, so on a case-sensitive host no rule would ignore these two trees at all; MVC 5 commits none | §7.2 | 08, 10 |
| F-02-21 | A class of aged (2011–2013), self-hosted, SRI- and CSP-unprotected dependencies, enumerated by exact pin | §8.1 | 07 (risk), 04 (disposition) |
| F-02-22 | NuGet's restore audit names **14 of the 63 pins** with `NU1902`/`NU1903` warnings while still exiting `0` — including the one authentication middleware actually enabled and the package no application source calls. The tally observed on the dated runs was 43 warnings, 9 of them high, carrying 24 distinct advisories; the pin set is a repository property, while which pins are named, at what severity and how many times are dated observations against a moving advisory database, retained row by row in §8.2.1 | §8.2, §8.2.1 | 07 (risk), 04 (disposition) |
| F-02-23 | No build, tooling or CI artifact in the repository records, retains or gates on that audit output; the warnings scroll past on every clean restore — 43 on the dated runs — and the only retained record of them is this assessment's dated snapshot in §8.2.1, which is a record and not a gate | §8.2, §8.2.1, §8.3 | 03 (gate), 08 (severity) |

---

## 10. Reproducibility appendix

Every command this document quotes, in one place. All are read-only **except** the three `nuget restore` invocations that produce the §8.2 audit evidence, which are flagged where they appear. The canonical forms are POSIX per AAP 0.11.3; on the Windows verification host they were run through the bundled Git-for-Windows `bash` from the repository root, and the values shown are the values observed.

```bash
# --- The headline count -------------------------------------------------------
git ls-files '*packages.config' | grep -v '/packages/' | xargs grep -h '<package ' | wc -l   # 63

# --- Per-edition pin counts and the exact strings ----------------------------
for f in src/MVC5/MvcMusicStore/packages.config \
         src/MVC4/MvcMusicStore/packages.config \
         src/MVC3/MvcMusicStore-Completed/MvcMusicStore/packages.config; do
  printf '%s pins=%s net45=%s tfattr=%s\n' "$f" \
    "$(grep -c '<package ' "$f")" \
    "$(grep -c 'targetFramework=\"net45\"' "$f")" \
    "$(grep -c 'targetFramework=' "$f")"
  grep -o 'id="[^"]*" version="[^"]*"' "$f"
done
# MVC5 pins=28 net45=28 tfattr=28   MVC4 pins=29 net45=29 tfattr=29   MVC3 pins=6 net45=0 tfattr=0

# --- Project target frameworks ----------------------------------------------
grep -n 'TargetFrameworkVersion' src/MVC5/MvcMusicStore/MvcMusicStore.csproj \
  src/MVC4/MvcMusicStore/MvcMusicStore.csproj \
  src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj      # v4.8 / v4.5 / v4.0

# --- References vs HintPaths (framework and machine-wide dependencies) -------
for f in src/MVC5/MvcMusicStore/MvcMusicStore.csproj \
         src/MVC4/MvcMusicStore/MvcMusicStore.csproj \
         src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj; do
  printf '%s refs=%s hintpaths=%s\n' "$f" "$(grep -c '<Reference Include' "$f")" "$(grep -c '<HintPath>' "$f")"
done                                                                        # 46/26, 47/24, 24/1

# --- NuGet tooling and the source configuration ------------------------------
git ls-files | grep '\.nuget/'                                              # 3 files, MVC4 only
git ls-files | grep -iE '(^|/)nuget\.config$'                               # 1 file, MVC4 only
git ls-files 'NuGet.config' 'NuGet.Config' 'nuget.config' | wc -l           # 0 at root
sed -n '19,25p' src/MVC4/MvcMusicStore/.nuget/NuGet.targets                 # the commented example block

# --- What is not pinned ------------------------------------------------------
git ls-files 'packages.lock.json' | wc -l                                   # 0
git ls-files | grep -c 'packages.lock.json'                                 # 0
git ls-files | grep -iE '(\.ruleset$)|(Directory\.Build\.props)|(Directory\.Packages\.props)|(\.editorconfig)' | wc -l   # 0

# --- Committed package payloads ---------------------------------------------
git ls-files | grep -c '/packages/'                                         # 215
git ls-files 'src/MVC4/MvcMusicStore/packages/*' | wc -l                    # 169
git ls-files 'src/MVC3/MvcMusicStore-Completed/packages/*' | wc -l          # 46
git ls-files | grep '/packages/' | grep -cE '\.(dll|exe)$'                  # 32
grep -in 'packages' .gitignore                                              # :15 packages/*  ; :33 Packages/  (-i is required: :33 is capitalised)

# --- Dead scaffolding and unused dependencies -------------------------------
git ls-files '*.cs' '*.cshtml' | grep -v '/packages/' | xargs grep -lE 'Newtonsoft|JsonConvert' | wc -l   # 0
git ls-files '*.cs' | grep -v '/packages/' | xargs grep -n 'ApiController' | wc -l                        # 0

# --- The restore audit of §8.2 (the one command here that is NOT read-only:
#     it writes a packages directory and requires network access). Copy each
#     manifest out of the tree first and restore the copy, so the checkout is
#     untouched. Nothing here rests on a remembered feature version: the client's
#     version is PARSED and compared numerically against the 6.11 floor this block
#     requires; no NuGetAudit* property may be set in tracked content or in the
#     environment (the third source, this invocation, overrides nothing); the
#     restore status is captured before any log is read; and the audit is proved to
#     have RUN, and run undegraded, from the warnings it emitted — NU1901-NU1904
#     present, NU1900 and NU1905 absent. The source is stated, never inherited.
SCRATCH="/tmp/nuget-audit"        # example value; anywhere OUTSIDE the working tree
NUGET="$(command -v nuget || true)"                 # one resolved path, not whatever PATH offers
[ -n "$NUGET" ] || { echo "no nuget client resolved" >&2; exit 1; }
CLIENT="$("$NUGET" help | head -1)"
VERSION="${CLIENT##*NuGet Version: }"               # e.g. 6.11.1.2 (the figures below)
MAJOR="${VERSION%%.*}"
REST="${VERSION#*.}"
MINOR="${REST%%.*}"
case "$MAJOR.$MINOR" in
  *[!0-9.]*|.*|*.) echo "cannot parse a version from [$CLIENT]" >&2; exit 1 ;;
esac
if [ "$MAJOR" -lt 6 ] || { [ "$MAJOR" -eq 6 ] && [ "$MINOR" -lt 11 ]; }; then
  echo "client is $VERSION; this block requires 6.11 or later — refusing to count" >&2
  exit 1
fi
if git grep -in 'NuGetAudit' -- . ':(exclude)docs/'; then   # index minus this tree
  echo "an audit property is set in tracked content — the defaults no longer hold" >&2
  exit 1
fi
if env | grep -i 'nugetaudit'; then                 # MSBuild reads env vars as properties
  echo "an audit property is set in the environment — the defaults no longer hold" >&2
  exit 1
fi
for e in mvc5 mvc4 mvc3; do mkdir -p "$SCRATCH/$e"; done
cp src/MVC5/MvcMusicStore/packages.config                         "$SCRATCH/mvc5/packages.config"
cp src/MVC4/MvcMusicStore/packages.config                         "$SCRATCH/mvc4/packages.config"
cp src/MVC3/MvcMusicStore-Completed/MvcMusicStore/packages.config "$SCRATCH/mvc3/packages.config"
for e in mvc5 mvc4 mvc3; do
  "$NUGET" restore "$SCRATCH/$e/packages.config" -PackagesDirectory "$SCRATCH/$e/packages" \
    -NonInteractive -Source https://api.nuget.org/v3/index.json > "$SCRATCH/$e.log" 2>&1
  status=$?                       # captured immediately; a failed restore stops the loop
  [ "$status" -eq 0 ] || { echo "$e restore exit=$status — counts would be meaningless" >&2; exit 1; }
  audited="$(grep -cE 'NU190[1-4]' "$SCRATCH/$e.log")"   # the audit ran, proved by its output
  [ "$audited" -gt 0 ] || { echo "$e: no NU1901-NU1904 — the audit did not run" >&2; exit 1; }
  if grep -qE 'NU1900|NU1905' "$SCRATCH/$e.log"; then    # a source unreachable, or with no data
    echo "$e: audit degraded (NU1900/NU1905) — the counts are incomplete" >&2
    exit 1
  fi
  printf '%s client=%s exit=%s audit=%s NU1902=%s NU1903=%s\n' "$e" "$VERSION" "$status" "$audited" \
    "$(grep -c NU1902 "$SCRATCH/$e.log")" "$(grep -c NU1903 "$SCRATCH/$e.log")"
done
# mvc5 client=6.11.1.2 exit=0 audit=15 NU1902=9 NU1903=6
# mvc4 client=6.11.1.2 exit=0 audit=14 NU1902=12 NU1903=2
# mvc3 client=6.11.1.2 exit=0 audit=14 NU1902=13 NU1903=1
# audit is the NU1901-NU1904 total and equals NU1902+NU1903 in each edition, because
# no low or critical warning was emitted against these pins (§8.2)
git status --porcelain            # unchanged: the restore wrote nothing into the checkout

# --- The same 43, derived from the audit source instead of from restore output
curl -s --compressed https://api.nuget.org/v3/vulnerabilities/index.json   # base + update document URLs
#     (--compressed is required: the endpoint answers gzip-encoded)
# then, per pinned version, count the base document's entries whose version range covers it:
# bootstrap 3.0.0 -> 6 moderate (11 entries held for the identifier); jQuery 1.10.2/1.7.1.1/1.5.1
# -> 3/5/6 moderate; jQuery.UI.Combined 1.8.20.1/1.8.11 -> 7/7 moderate; jQuery.Validation
# 1.11.1/1.9.0.1/1.8.0 -> 1/1/1 high; Microsoft.Owin 2.0.0 -> 2 high; Security.Cookies 2.0.0 and
# Identity.Owin 1.0.0 -> 1 high each; Newtonsoft.Json 5.0.6/4.5.6 -> 1/1 high.  Totals 15/14/14.
# No entry of severity 0 (low) exists for any of the 63 pins; the document holds 808 such
# entries for other packages, which is why the absence is data and not a filter (§8.2).

# --- Dependency-scanning configuration (all absent) -------------------------
git ls-files | grep -c '^\.github/'                                         # 0
git ls-files | grep -icE 'dependabot|renovate'                              # 0
git ls-files '*packages.config' | grep -v '/packages/' \
  | xargs grep -hiE 'analyzer|StyleCop|FxCop|SonarAnalyzer|Roslynator' | wc -l                            # 0

# --- Asset attribution ------------------------------------------------------
git ls-files 'src/MVC4/MvcMusicStore/Content/*' | wc -l                     # 55
git ls-files 'src/MVC4/MvcMusicStore/Content/themes/*' | wc -l              # 54
git ls-files 'src/MVC5/MvcMusicStore/Scripts/*' 'src/MVC5/MvcMusicStore/Content/*' 'src/MVC5/MvcMusicStore/fonts/*'

# --- The constraint this work was held to -----------------------------------
git status --porcelain            # only new files under docs/modernization/
```

---

*End of deliverable 02. Index and requirement map: [README](README.md). No repository file outside `docs/modernization/` was created, modified or deleted in the production of this document.*

### 10.1 Read-only evidence commands

None of these writes anything. They read the git index, tracked file contents and file metadata only.

```bash
# --- The headline count -------------------------------------------------------
git ls-files '*packages.config' | grep -v '/packages/' | xargs grep -h '<package ' | wc -l   # 63

# --- Per-edition pin counts and the exact strings ----------------------------
for f in src/MVC5/MvcMusicStore/packages.config \
         src/MVC4/MvcMusicStore/packages.config \
         src/MVC3/MvcMusicStore-Completed/MvcMusicStore/packages.config; do
  printf '%s pins=%s net45=%s tfattr=%s\n' "$f" \
    "$(grep -c '<package ' "$f")" \
    "$(grep -c 'targetFramework=\"net45\"' "$f")" \
    "$(grep -c 'targetFramework=' "$f")"
  grep -o 'id="[^"]*" version="[^"]*"' "$f"
done
# MVC5 pins=28 net45=28 tfattr=28   MVC4 pins=29 net45=29 tfattr=29   MVC3 pins=6 net45=0 tfattr=0

# --- Manifest extents cited in the §3.1 / §3.2 / §3.3 headings ---------------
for f in src/MVC5/MvcMusicStore/packages.config \
         src/MVC4/MvcMusicStore/packages.config \
         src/MVC3/MvcMusicStore-Completed/MvcMusicStore/packages.config; do
  printf '%s total=%s firstpin/lastpin=%s\n' "$f" \
    "$(awk 'END{print NR}' "$f")" \
    "$(grep -n '<package ' "$f" | sed -n '1p;$p' | cut -d: -f1 | paste -sd/ -)"
done
# MVC5 total=31 firstpin/lastpin=3/30   MVC4 total=32 3/31   MVC3 total=9 3/8

# --- Project target frameworks ----------------------------------------------
grep -n 'TargetFrameworkVersion' src/MVC5/MvcMusicStore/MvcMusicStore.csproj \
  src/MVC4/MvcMusicStore/MvcMusicStore.csproj \
  src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj      # v4.8 / v4.5 / v4.0

# --- References vs HintPaths (framework and machine-wide dependencies) -------
for f in src/MVC5/MvcMusicStore/MvcMusicStore.csproj \
         src/MVC4/MvcMusicStore/MvcMusicStore.csproj \
         src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj; do
  printf '%s refs=%s hintpaths=%s\n' "$f" "$(grep -c '<Reference Include' "$f")" "$(grep -c '<HintPath>' "$f")"
done                                                                        # 46/26, 47/24, 24/1

# --- NuGet tooling and the source configuration ------------------------------
git ls-files | grep '\.nuget/'                                              # 3 files, MVC4 only
git ls-files | grep -iE '(^|/)nuget\.config$'                               # 1 file, MVC4 only
git ls-files 'NuGet.config' 'NuGet.Config' 'nuget.config' | wc -l           # 0 ROOT-LEVEL only (no wildcard = root-only pathspec; see below)
sed -n '19,25p' src/MVC4/MvcMusicStore/.nuget/NuGet.targets                 # the commented example block

# --- What is not pinned ------------------------------------------------------
# A pathspec with no wildcard matches only the repository root, so it cannot
# support an absence claim about nested projects (§7.1). Control case:
git ls-files 'packages.config' | wc -l                                      # 0  <- root-only, and therefore useless here
git ls-files '*packages.config' | grep -v '/packages/' | wc -l              # 3  <- the same index, seen correctly
# The absence claim itself, filtering the whole index:
git ls-files | grep -E '(^|/)packages\.lock\.json$'                         # no output, exit status 1
git ls-files | grep -c 'packages.lock.json'                                 # 0
git ls-files | grep -iE '(\.ruleset$)|(Directory\.Build\.props)|(Directory\.Packages\.props)|(\.editorconfig)' | wc -l   # 0

# --- Committed package payloads ---------------------------------------------
git ls-files | grep -c '/packages/'                                         # 215
git ls-files 'src/MVC4/MvcMusicStore/packages/*' | wc -l                    # 169
git ls-files 'src/MVC3/MvcMusicStore-Completed/packages/*' | wc -l          # 46
git ls-files | grep '/packages/' | grep -cE '\.(dll|exe)$'                  # 32
grep -in 'packages' .gitignore                                              # .gitignore:15 packages/*  ; .gitignore:33 Packages/  (-i is required: line 33 is capitalised)
# Which of those two rules actually matches which path. --no-index reports the
# rule for a path that is neither tracked nor present, which is what makes it
# usable now that the generated trees of §1.3 are gone:
git check-ignore -v --no-index src/MVC4/packages/x src/MVC5/packages/x \
  src/MVC4/MvcMusicStore/packages/x src/MVC5/MvcMusicStore/bin/x src/MVC5/MvcMusicStore/obj/x
# -> .gitignore:33:Packages/   src/MVC4/packages/x
#    .gitignore:33:Packages/   src/MVC5/packages/x
#    .gitignore:33:Packages/   src/MVC4/MvcMusicStore/packages/x
#    .gitignore:2:[Bb]in/      src/MVC5/MvcMusicStore/bin/x
#    .gitignore:1:[Oo]bj/      src/MVC5/MvcMusicStore/obj/x
# Line 33 has no interior separator, so it matches at any depth; line 15 does, so
# it is anchored to the repository root and appears in none of these results. On a
# case-sensitive host line 33 would not match a lowercase `packages` either, and
# these three paths would come back unignored.
git config core.ignorecase                                                  # true: why line 33 reaches the lowercase directories on this host

# --- Dead scaffolding and unused dependencies -------------------------------
git ls-files '*.cs' '*.cshtml' | grep -v '/packages/' | xargs grep -lE 'Newtonsoft|JsonConvert' | wc -l   # 0
git ls-files '*.cs' | grep -v '/packages/' | xargs grep -n 'ApiController' | wc -l                        # 0

# --- The restore audit of §8.2 is NOT here: it is mutating. See §10.2. -------

# --- Dependency-scanning configuration (all absent) -------------------------
git ls-files | grep -c '^\.github/'                                         # 0
git ls-files | grep -icE 'dependabot|renovate'                              # 0
git ls-files '*packages.config' | grep -v '/packages/' \
  | xargs grep -hiE 'analyzer|StyleCop|FxCop|SonarAnalyzer|Roslynator' | wc -l                            # 0

# --- Asset attribution ------------------------------------------------------
git ls-files 'src/MVC4/MvcMusicStore/Content/*' | wc -l                     # 55
git ls-files 'src/MVC4/MvcMusicStore/Content/themes/*' | wc -l              # 54
git ls-files 'src/MVC5/MvcMusicStore/Scripts/*' 'src/MVC5/MvcMusicStore/Content/*' 'src/MVC5/MvcMusicStore/fonts/*'

# --- The constraint this work was held to (§1.3): all four, together --------
# 1. The tracked diff against the pre-assessment baseline: exactly thirteen "A"
# rows under docs/modernization/, and no "M" or "D" row anywhere.
# git separates the status letter from the path with a TAB, so the patterns
# below match it with [[:space:]] rather than \t, which grep -E treats as a
# literal backslash-t and would score 0 against real output.
BASE=ea2552d6eda7c20e9477a512e5c615665618cf35
git diff --name-status "$BASE"..HEAD                                                    # 13 rows
git diff --name-status "$BASE"..HEAD | wc -l                                            # 13
git diff --name-status "$BASE"..HEAD | grep -cE '^A[[:space:]]+docs/modernization/'      # 13
git diff --name-status "$BASE"..HEAD | grep -cvE '^A[[:space:]]+docs/modernization/'     # 0
# 2. Tracked working-tree state. On the committed checkout this prints nothing:
git status --porcelain            # (empty: the thirteen files are committed, not untracked)
# 3. The same state including ignored content — the only one of the four that
# would have shown the eight generated trees of §1.3 while they existed. [Oo]bj/
# (.gitignore:1) and [Bb]in/ (.gitignore:2) covered the six build trees; the two
# nested packages trees were matched by Packages/ (.gitignore:33), which is
# depth-independent and reaches them only because core.ignorecase is true here.
# packages/* (.gitignore:15) is root-anchored and matches no nested path:
git status --porcelain --ignored   # (empty: no restored packages/, no bin/, no obj/)
# 4. What an ignore-aware clean would remove, listed rather than deleted:
git clean -ndX                     # (no output: nothing ignored is left to remove)
```

### 10.2 Mutating operations — three package-mutating restores, disposable clone only

This is the whole of what this document changes, and it exists solely to regenerate the §8.2 audit evidence. Calling it "three commands" would undercount it, so the workflow below is classified line by line:

| Operation | What it mutates | Where the effect lands |
| --- | --- | --- |
| `$SCRATCH` validation — `:?`, `case`, `git rev-parse`, `pwd -P` | **Nothing**: it reads, compares and refuses | The invoking shell only; if it refuses, no later line runs at all |
| `git clone --no-hardlinks "$REPO" "$AUDIT"` | Creates a new working tree | `$SCRATCH` only; it **reads** the repository it is run from, and refuses a pre-existing `$AUDIT` |
| The **three** `nuget restore` invocations | **Package payloads**: each downloads and writes `packages/` into whatever tree it is run in | The clone: each solution path is rooted at `$AUDIT`, and the block refuses to reach them at all unless the clone succeeded |
| `> "$AUDIT/restore-mvc*.log" 2>&1` and the `nu190-all.log` concatenation | Creates four log files | Inside the clone, discarded with it. `*.log` is gitignored [.gitignore:25], so they cannot dirty even the clone's tracked status |
| `rm -rf "$AUDIT_REAL"`, after the path is re-derived and re-checked | Deletes the clone | `$SCRATCH` only, and only the directory this run created |

So there are **three package-mutating commands and four scratch-confined operations**, preceded by a validation step that mutates nothing and refuses to let any of them run, and only the three carry any risk to the assessed checkout. **The block below is fail-closed, and for these three commands that is the whole safety property:** `set -euo pipefail` plus a checked exit code on every step means a failed `$SCRATCH` validation, a failed `git clone` or a failed restore stops the run instead of letting the next line proceed — which matters because a `nuget restore` that is reached without a proven clone writes package payload into whatever tree the caller happens to be standing in, and this document’s entire premise is that that tree is never the assessed checkout. Its validation, clone, checked-exit-code restore and guarded-cleanup steps are identical to the §8.2 copy line for line — only the parsing tail differs, which is where the two sections’ purposes diverge — so the two cannot disagree about safety. Each of the three must be run in a disposable clone or scratch checkout outside the authoritative repository, and **never in place against the assessed checkout**. Because a restored `src/MVC5/packages` or `src/MVC4/packages` is matched by `Packages/` [.gitignore:33] — the depth-independent rule, which reaches a lowercase directory only where `core.ignorecase` is true, as it is on this host; `packages/*` [.gitignore:15] is anchored to the repository root and reaches no nested path, and on a case-sensitive host neither rule matches a nested lowercase `packages` at all — `git status --porcelain` will not show the payload one of them writes on a host like this one; `git status --porcelain --ignored` will, and discarding the clone makes the question moot. §1.3 records what happened when restore and build runs were made in place earlier in this assessment — eight ignored trees, since removed — and the four-command acceptance check that establishes their absence.

```bash
# MUTATING. Network access required. Never run these in the assessed checkout.
# $SCRATCH is any private directory outside the authoritative repository.
# Fail-closed by construction, and identically to the §8.2 copy: $SCRATCH is
# validated before anything else, the block stops at the first failure, and every
# path is rooted at the clone, so no restore can run in the caller’s directory.
set -euo pipefail

# 1 - $SCRATCH must be set, non-empty, absolute, an existing directory, and
#     outside this repository. A restore rooted inside the assessed checkout is
#     the one outcome this workflow has to make impossible, so it is checked
#     first and nothing else runs until it passes.
: "${SCRATCH:?set SCRATCH to an existing private directory outside the repository}"
case $SCRATCH in
  /*) ;;
  *) echo "SCRATCH must be an absolute path: $SCRATCH" >&2; exit 1 ;;
esac
if [ ! -d "$SCRATCH" ]; then
  echo "SCRATCH is not an existing directory: $SCRATCH" >&2
  exit 1
fi
# Containment is tested twice, because a path has more than one spelling on a
# Windows host and one string comparison would let the other through. First git’s
# own answer, which is spelling-proof and holds at any depth inside the work tree:
REPO_TOP=$(git rev-parse --show-toplevel)   # fails here if this is not a repository
if [ "$(git -C "$SCRATCH" rev-parse --show-toplevel 2>/dev/null || true)" = "$REPO_TOP" ]
then
  echo "SCRATCH is inside the assessed repository: $SCRATCH" >&2
  exit 1
fi
# Then a physical-path test with both sides normalised the same way, which also
# catches a nested repository sitting inside the checkout:
REPO_REAL=$(cd "./$(git rev-parse --show-cdup)" && pwd -P)
SCRATCH_REAL=$(cd "$SCRATCH" && pwd -P)
case $SCRATCH_REAL in
  "$REPO_REAL" | "$REPO_REAL"/*)
    echo "SCRATCH is inside the checkout at $REPO_REAL: $SCRATCH_REAL" >&2
    exit 1 ;;
esac

# 2 - The clone. A pre-existing target is refused, so the directory the cleanup
#     removes is always one this run created. This shell never changes directory:
#     the only `cd` calls above and below are inside command substitutions, which
#     cannot outlive the subshell they run in, so the working directory the caller
#     started in is still the working directory at the end. Every path from here on
#     is rooted at $AUDIT, which is what makes it impossible for a restore to
#     inherit the caller’s working directory.
AUDIT=$SCRATCH_REAL/mvcmusicstore-audit
if [ -e "$AUDIT" ]; then
  echo "refusing to reuse an existing path: $AUDIT" >&2
  exit 1
fi
git clone --no-hardlinks "$REPO_TOP" "$AUDIT"
if [ ! -d "$AUDIT/.git" ]; then
  echo "clone produced no working tree: $AUDIT" >&2
  exit 1
fi

# 3 - The three package-mutating commands, each with its exit code checked, so no
#     count is ever parsed from a partial log. A failure here exits before the
#     cleanup below, deliberately: the clone and its logs stay in $SCRATCH to be
#     read, and are discarded by hand. -Verbosity detailed puts the client version
#     into the log itself; the warnings appear at either verbosity.
restore() {          # $1 = solution path inside the clone, $2 = log file name
  if ! nuget restore "$AUDIT/$1" -NonInteractive -Verbosity detailed \
       > "$AUDIT/$2" 2>&1; then
    echo "restore FAILED: $1 (log: $AUDIT/$2) - stopping before any parsing" >&2
    exit 1
  fi
  echo "exit=0  $1"
}
restore src/MVC5/MvcMusicStore.sln                         restore-mvc5.log
restore src/MVC4/MvcMusicStore.sln                         restore-mvc4.log
restore src/MVC3/MvcMusicStore-Completed/MvcMusicStore.sln  restore-mvc3.log

L5=$AUDIT/restore-mvc5.log
L4=$AUDIT/restore-mvc4.log
L3=$AUDIT/restore-mvc3.log
ALL=$AUDIT/nu190-all.log

# 4 - The counts those runs produced, reached only if all three exited 0. Capture
#     the logs as BYTES: a PowerShell 5.1 ">" redirect writes UTF-16LE, in which
#     grep matches nothing (§8.2) - and under `set -e` a run whose counts are all
#     zero now stops here instead of printing a reassuring row of zeroes.
grep -hc 'NU190' "$L5" "$L4" "$L3"                     # -> 15, 14, 14
cat "$L5" "$L4" "$L3" > "$ALL"
grep -c 'NU190' "$ALL"                                 # -> 43   (incidences)
grep -c 'NU1903' "$ALL"                                # -> 9    (high)
grep -c 'NU1902' "$ALL"                                # -> 34   (moderate)
grep -o 'GHSA-[a-z0-9-]*' "$ALL" | sort -u | wc -l     # -> 24   (distinct advisories)

# 5 - Record the provenance any count must be quoted with (§8.2), and the identity
#     of the three logs the raw block of §8.2.1 is cut from - taken here, from
#     these same logs, before the cleanup below discards them:
nuget help | head -1        # client version, e.g. NuGet Version: 6.11.1.2
nuget sources list          # the effective sources, which the repository does not specify (§6)
date -u +%Y-%m-%dT%H:%M:%SZ # the UTC date the counts belong to
grep -h 'vulnerability\.\(base\|update\)\.json' "$ALL" | sort -u  # advisory-DB snapshot ids
sha256sum "$L5" "$L4" "$L3" # identify THIS run's logs. The digests and sizes in §8.2.1 belong to the
wc -c "$L5" "$L4" "$L3"     # retained 2026-08-28 run (22611, 25413, 11639) and a re-run will not
                            # reproduce them: the client version line, the advisory-DB snapshot ids and
                            # the timing lines all differ. Record these alongside a re-run's counts; the
                            # figure that IS comparable across runs is the raw block of §8.2.1.

# What the restores wrote, and why porcelain status is the wrong check. `git -C`
# reads the clone without moving this shell into it:
git -C "$AUDIT" status --porcelain            # empty - packages/ is ignored, so this proves nothing
git -C "$AUDIT" status --porcelain --ignored  # shows the new src/MVC5/packages and src/MVC4/packages
                                              # trees, and the four *.log files written above
git -C "$AUDIT" clean -ndX                    # the same content, as an ignore-aware clean would list it

# The 43 individual warnings from these runs are retained in §8.2.1, with the
# command that extracts them from these three logs; a later run is diffed against
# that block rather than against these counts.

# Discard the clone. The path is re-derived and re-checked first, so a $SCRATCH
# that changed mid-run, or a symlink swapped underneath it, cannot turn this into
# an rm -rf of anything but the directory step 2 created.
AUDIT_REAL=$(cd "$AUDIT" && pwd -P)
case $AUDIT_REAL in
  "$SCRATCH_REAL"/mvcmusicstore-audit) ;;
  *) echo "cleanup refused: $AUDIT_REAL is not the path this run created" >&2
     exit 1 ;;
esac
if [ ! -d "$AUDIT_REAL/.git" ]; then
  echo "cleanup refused: $AUDIT_REAL is not the clone this run created" >&2
  exit 1
fi
rm -rf "$AUDIT_REAL"

# The same two ignored-aware checks, plus the tracked diff, are the acceptance
# check of §1.3 — and they are what established that the eight generated trees
# that once sat in the assessed checkout are gone. They are in §10.1, run
# against the assessed checkout rather than the clone.
```

---

*End of deliverable 02. Index and requirement map: [README](README.md). No tracked repository file outside `docs/modernization/` was created, modified or deleted in the production of this document. The generated, ignored content that the assessment's restore and build runs left in the checkout — eight trees, all removed — is recorded in §1.3 with the four-command check that establishes its absence.*
