
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

Every figure below was obtained by reading files and by read-only `git` queries, with one flagged exception: the restore-time audit route described in §8.2 requires a `nuget restore`, which is a **mutating** command — it downloads package payloads and writes them into the working tree under `packages/`. This document therefore separates its commands into two classes and labels them as such wherever they appear: **read-only evidence commands**, which are safe to run anywhere and are collected in §10.1, and **the one mutating workflow**, quoted exactly once, in §10.2. That workflow's mutations are of two kinds, and collapsing them into one count is how such a statement goes wrong. **Three of its commands are package-mutating** — the three `nuget restore` invocations, which download payloads and write them into whatever tree they are run in. **Four further operations mutate nothing but the disposable clone and the directory holding it**: the `git clone` that creates the clone under `$SCRATCH`, the redirected log writes inside it, the concatenation the logs are read through, and the guarded `rm -rf` that discards it afterwards. A fifth step mutates nothing at all and is the reason the other seven are safe: the workflow opens by **validating `$SCRATCH`** — set, non-empty, absolute, existing and outside this repository — and runs under `set -euo pipefail` with every path rooted at the clone and no directory change that outlives a command substitution, so a failed clone or a mistyped `$SCRATCH` stops the workflow instead of leaving three `nuget restore` invocations to run in whatever directory the caller was standing in. The rule for the three restores is stated where they appear: run them in a **disposable clone or scratch checkout outside the authoritative repository, never in place against the assessed checkout**. The workflow was exercised during authoring; what this document carries from it, and what it deliberately does not, is stated in §8.2. The same read-only-versus-mutating distinction governs the future release operations, which are owned by [06 — Azure Hosting Recommendations](06-azure-hosting-recommendations.md); this document adopts the distinction for its own commands and claims none of that deliverable's decisions.

**Generated output did reach the assessed checkout during this assessment, and this document accounts for it rather than folding it into a clean-tree claim.** Restore and build operations were run in place against this checkout earlier in the assessment, and they left **eight ignored trees** behind — a `bin` and an `obj` beside each of the three editions' projects, plus a restored `packages` tree under `src/MVC4` and another under `src/MVC5` — **527 files and 114,310,394 bytes, as recorded before removal**, because a tree that no longer exists cannot be re-counted. All eight were removed once the assessment had finished with them, and their absence was verified at that point by the four commands below. That verification is a statement about a moment, not a durable property, and the distinction is load-bearing: commands 1 and 2 concern **tracked** content and are the binary criterion, while commands 3 and 4 report **ignored** content and will legitimately be non-empty in any checkout where a build or restore has run since — including one where the run belongs to concurrent work rather than to this document. Their empty result is therefore the hygiene condition to be met before the checkpoint is committed, owned by whoever commits it, and not a claim this document makes about the moment a reader happens to run them. No tracked file was modified, added or deleted at any point, and the dependency-graph statement above is unaffected: an ignored payload directory is not a declared dependency. **Those operations are not build evidence, and nothing here treats them as any.** They were unqualified historical restore and build runs whose only recorded effect is the gitignored residue above: [10 — Build and Deployment Requirements](10-build-and-deployment-requirements.md) states that no run behind those trees recorded the fields its evidence requires — tool versions, the restore source used, the configuration built, the per-edition outcome, the warning and error counts — so neither a restore having run nor an output directory having existed says anything about whether an edition builds. Every build outcome, status and evidentiary claim is that deliverable's to make, including MVC 5's, which remains **blocked pending a Windows verification run**; this document adds nothing to it and softens nothing in it. The per-tree record belongs to deliverable 10's appendix and is cross-referenced rather than restated here. Two of the eight are the direct product of the §8.2 restores — `src/MVC5/packages` at 167 files and `src/MVC4/packages` at 231, **398 files and 78,916,729 bytes** of that total — and those two are reproducible from §8.2's own command block, which is why they are the two this document can account for from its own evidence.

The acceptance check for that constraint is therefore **four commands, and no three of them are the check**: a diff against the pre-assessment baseline plus three working-tree checks, two of which are ignored-aware. The rules that excluded the eight trees are `[Oo]bj/` [.gitignore:1], `[Bb]in/` [.gitignore:2] and `Packages/` [.gitignore:33], and the third is the load-bearing one and the least obvious: a pattern whose only separator is trailing matches a directory of that name at **any** depth, and on this checkout — `git config core.ignorecase` reports `true` — it also matches the lowercase `packages` directories, which is why `src/MVC4/packages` and `src/MVC5/packages` were ignored here. `packages/*` [.gitignore:15] does *not* cover them, because a pattern with an interior separator is anchored to the directory holding the `.gitignore` — the repository root. That makes the ignore behaviour recorded here a property of a case-insensitive host: **on a case-sensitive host no rule ignores those two nested lowercase `packages` trees at all**, and a bare porcelain status would have listed them instead of reporting a clean tree. `git check-ignore -v --no-index` reports the rule per path and is in §10.1. Either way the consequence for the checks below is the one that matters: a bare porcelain status and a tracked-file diff both report a clean tree while generated payload sits in it — which is exactly what they did on this checkout for as long as the eight trees existed. Run, against the committed checkpoint:

```bash
# 1 — the tracked diff against the pre-assessment baseline.
#     The left side is the immutable pre-assessment revision: the last commit before this
#     assessment began, and the revision at which every source path cited in this document
#     resolves. The right side is named HEAD rather than a hash because a document cannot
#     cite the commit that creates it, so no hash is invented for it — the reader resolves
#     HEAD on the delivered checkout carrying these thirteen deliverables.
git diff --name-status ea2552d6eda7c20e9477a512e5c615665618cf35..HEAD
# -> exactly 13 rows, every one an A for a file under docs/modernization/, and no M or D row

# 2 — tracked working-tree state
git status --porcelain             # -> (no output)

# 3 — the same state with ignored content included: the one check the other three are blind to
git status --porcelain --ignored   # -> (no output) once ignored output is cleared:
#                                    no restored packages/, no bin/ and no obj/ anywhere

# 4 — what an ignore-aware clean would remove, listed rather than deleted
git clean -ndX                     # -> (no output) once cleared: nothing ignored left to remove
```

**Only one end of that range is a literal hash, and the asymmetry is deliberate rather than an omission.** The baseline endpoint is pinned in full — `ea2552d6eda7c20e9477a512e5c615665618cf35`, the last commit before this engagement — so the range is exactly this engagement and nothing else. The far end is written `HEAD` rather than a second hash because **no document can contain the hash of the commit that adds it**: that hash exists only once the commit does, which is after this file has reached the content the commit records, so a literal in its place would necessarily name something other than the commit meant. A reader who wants both ends pinned resolves it once on their own checkout — `git rev-parse HEAD` — and substitutes the result wherever `HEAD` appears above and in §10.1. **The substitution changes none of the four values the assertion expects:** 13 rows, 13 `A` rows, 0 rows that are not an `A` under `docs/modernization/`, and 0 paths outside `docs/modernization/`. Each of the four is a property of the range and not of the particular tip it ends at, so all four hold at every tip at which the thirteen-file set is complete — including the tip that lands the last correction to it.

**The two ends of command 1's range, stated once.** Its left side is the immutable evidence revision `ea2552d6eda7c20e9477a512e5c615665618cf35` — the last commit before this assessment began, at which every source path this document cites is byte-identical to the delivered tree, and after this sentence the short form `ea2552d` is used for it in prose, while command blocks quote whichever form makes them unambiguous to copy and run. Its right side is `HEAD`, which is the **delivery commit the reviewer has checked out**, deliberately left as a symbolic reference: a document cannot cite the commit that first contains it, so no hash is invented for that end and the range is the one place in this document where a moving reference is correct.

Command 1 is a property of the committed history and is the durable evidence: **thirteen `A` rows** and nothing else means no existing file was modified or deleted. Commands 2 to 4 describe a working tree, so each is evidence only of the checkout it is run in **at the moment it runs** — command 2 is non-empty while uncommitted edits are in flight, and **3 and 4 are the only two that look at ignored content**, which is the half of the claim bare porcelain looked like it was making and was not. Because 3 and 4 see ignored content, they report generated output regardless of what produced it, so in a checkout where any build or restore has run since the last clean they are expected to be non-empty and their non-emptiness is not evidence against the tracked-file claim that commands 1 and 2 establish. Reaching their empty result is a pre-commit step owned by whoever commits the checkpoint. A reader who expected `git status --porcelain` to list the new documents is reading an authoring-time working tree rather than the committed result. §10.1 repeats all four with their observed output.

This is a catalogue, not a curation.

### 1.4 Authoring contract, and the absence of user rules

`review_rules` returns exactly **"No user rules provided."** There is therefore no project rule to name, summarize or comply with, and no file forced into scope by one. The absence is not licence to lower the bar; this document is held to enterprise-standard assessment practice and to the following contracts, which stand in place of rules:

- **Repository evidence is primary.** Every as-is claim carries an inline `[<path>:<locator>]` citation placed at the claim, never collected in a trailing reference list. Paths are repository-relative and resolve in the checkout.
- **The path is written out in full at every citation, with no exception.** No citation in this document inherits its path from a neighbouring table row, a neighbouring cell, a table heading or a preceding paragraph, so any single citation can be checked in isolation without reading what surrounds it. That includes the per-row `Manifest citation` column of the three pin tables in §3, each cell of which carries its manifest path in full. Nothing in this document relies on inheritance, and no locator appears anywhere without the path it belongs to.
- **A tracked binary is cited at its size, because it has no line to point at.** The repository's only committed binary dependency is `src/MVC4/MvcMusicStore/.nuget/NuGet.exe`. Its locator is the size-only form, `630,784 bytes`, and nothing else goes inside the bracket; the rest of its identity — `FileVersion`, assembly version and `ProductVersion` — is stated in the sentence wherever the file is cited, and reproduced by the command block in §5.1.
- **A repository-wide claim carries its reproducing command** next to the claim, because a count or an absence has no single line to point at. That is the stronger form of evidence: a reader can re-run it.
- **Exact versions only.** Every version string below is transcribed character-for-character from the manifest that declares it. No ranges, no rounding, no "or later". `1.0.0.0` is not `1.0.0`, and `1.0` is not `1.0.0`.
- **The Technical Specification is secondary.** Sections of it may be cited *alongside* repository evidence, never instead of it. §6.2 below records a place where the specification and the repository disagree, and resolves it in the repository's favour.

**The citation contract, audited against this file rather than asserted.** The two claims above that are properties of this document — that no path is ever inherited, and that every cited path resolves — are mechanically checkable, so they are checked against the delivered file rather than promised. Regenerated from it:

- **324** inline citations carry a line locator, and **0** of them are continuation form. A continuation-form locator is one whose path part is empty — an opening bracket followed immediately by a colon and a line number — which is precisely the shape the second contract above forbids, so the check is a count of that shape and it has to be zero. The shape is described rather than written out here, because a document that quoted it would then match its own audit and report one.
- Those 324 citations name **19 distinct repository paths**, and every one of the 19 resolves at the evidence revision `ea2552d`: `.gitignore`, which is complete as written because that file sits at the repository root, and eighteen paths under `src/`.
- **One** further citation carries a non-numeric locator, and it is the declared binary-identity form of the third contract above — locator `src/MVC4/MvcMusicStore/.nuget/NuGet.exe:FileVersion 2.0.30828.5, 630,784 bytes`, cited in §2.2 and reproduced by the command block of §5.1 — which is the only citation in this document to a file with no line to point at. It is named without its enclosing brackets here for the same self-matching reason.

```bash
DOC=docs/modernization/02-dependency-inventory.md
REV=ea2552d
CITE='\[[^][]*:[0-9]+(-[0-9]+)?\]'
grep -oE "$CITE" "$DOC" | wc -l                                   # -> 324  citations with a line locator
grep -oE '\[:[0-9]+(-[0-9]+)?\]' "$DOC" | wc -l                   # -> 0    continuation form: none
grep -oE "$CITE" "$DOC" | sed -E 's/^\[//; s/:[0-9]+(-[0-9]+)?\]$//' \
  | sort -u | wc -l                                               # -> 19   distinct cited paths
grep -oE "$CITE" "$DOC" | sed -E 's/^\[//; s/:[0-9]+(-[0-9]+)?\]$//' | sort -u \
  | while read -r p; do
      git cat-file -e "$REV:$p" 2>/dev/null || echo "UNRESOLVED $p"
    done                                                          # -> (no output): all 19 resolve
grep -oE '\[[^][]*(/|\.gitignore)[^][]*:[^][0-9][^][]*\]' "$DOC" \
  | wc -l                                                         # -> 1    the NuGet.exe identity locator
```

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
| Aged pins in the version-risk class | **14, in 9 rows** | §8.1 enumerates them with the manifest that declares each — 2011–2013 releases, self-hosted, served with neither Subresource Integrity nor a Content Security Policy; three further groups of the same era are named there in prose |
| Advisory identifiers, severities or counts asserted by this document | **0** | AAP 0.5.1's form: a class of finding with the exact pins, a scanner directed for the implementation phase, and no advisory identifier asserted. §8.2 states the rule and the command by which a reader obtains today's set for themselves |

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

| `packages/` payload committed to source control (counts and their reproducing commands in §7.2) | none | 169 files, 29 folders | 46 files, 6 folders |
| MSBuild-integrated restore wired | import present but **conditional** [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:295] | **unconditional** import [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:360] with `<RestorePackages>true</RestorePackages>` [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:24] | not wired |
| `.nuget` folder present | **no** — although the solution declares one (§5.2) | yes — three tracked artifacts [src/MVC4/MvcMusicStore/.nuget/NuGet.Config:1-6], [src/MVC4/MvcMusicStore/.nuget/NuGet.targets:1-143], [src/MVC4/MvcMusicStore/.nuget/NuGet.exe:630,784 bytes] — the third being the `FileVersion` **2.0.30828.5** restore client of §5.1 | no |

The MVC 4 cell cites the folder's contents rather than the folder, because a directory has no line to point at and the claim is about what is tracked inside it. Two of the three are text files cited at their full extent; the third is a binary, so its locator is the size-only form §5.1 uses for the same artifact, and the version a file inspection reports is stated in the cell beside the citation rather than inside it. The set is exhaustive, and the command is the evidence:

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

Deliverable 09 counts the same packages from the security side as finding **F-09-36**, and the identifier is printed here so that its register row lands on a named item in this document rather than on a topic: these four [src/MVC5/MvcMusicStore/packages.config:20], [src/MVC5/MvcMusicStore/packages.config:21], [src/MVC5/MvcMusicStore/packages.config:22], [src/MVC5/MvcMusicStore/packages.config:24] are the ones that implement a **named provider**, which is the narrower of the two units 09 §6.11 states, and `Microsoft.Owin.Security.OAuth` [src/MVC5/MvcMusicStore/packages.config:23] is excluded there for the same reason it is excluded here. §9.1 records the closure.

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

**Finding F-09-36's wider unit is the union of the two sections, and MVC 4 supplies seven of its eleven.** The six DotNetOpenAuth pins [src/MVC4/MvcMusicStore/packages.config:3-8] plus `Microsoft.AspNet.WebPages.OAuth` [src/MVC4/MvcMusicStore/packages.config:23] — the pin that ships `OAuthWebSecurity` itself, which is why 09 §6.11 counts it rather than treating it as peripheral — make MVC 4's seven, and §3.1.2's four make MVC 5's four. Seven plus four is the eleven that row states. §9.1 records the closure.

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

Every place where a package version and the version of the assembly it supplies are different numbers is enumerated in §4.4.1 immediately below — **all seventeen, in one inventory**, with the census that produces the number. There is deliberately no sampled subset here to compete with it: a short list of representative cases and a complete enumeration of the same fact would give a downstream document two counts to choose between, and deliverable 04 consumes this inventory pin by pin (§9).

---

#### 4.4.1 Every divergence, enumerated — all seventeen

There are **seventeen** places in this repository where a package version and the version of the assembly it supplies are different numbers, and all seventeen are listed below rather than sampled. Every figure this section states — the seventeen, the twenty-one pairs it is drawn from, the three-way partition and the per-edition split — is a count over the tables below and over the census command that produces them, not a headline carried in from anywhere else. This is the only enumeration of them in this document: a list of representative examples closed with "the same holds for the others" would leave the remainder for a downstream document to rediscover, and deliverable 04 consumes this inventory pin by pin (§9), so the enumeration has to be complete and singular. The census is stated first, so the count is checkable rather than asserted.

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
| **Same number, different arity** | 3 | MVC 5 only, and all three are the pin zero-extended to four parts: `Microsoft.AspNet.Mvc` `5.0.0` → `5.0.0.0` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:76-79]; `Microsoft.AspNet.Razor` `3.0.0` → `3.0.0.0` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:83-86]; `Microsoft.AspNet.WebPages` `3.0.0` → `3.0.0.0` across its four assemblies [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:72-75], [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:87-90], [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:91-94], [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:95-98]. Under the exact-string rule of §1.4 these are *different strings*; as **numbers** they agree, so they are separated out rather than counted as divergences |
| **Genuinely different numbers** | **16** | The fifteen MVC 4 rows and the one MVC 3 row of the table below |

Sixteen from the project files, plus **one the project-file comparison cannot see at all**: MVC 5's `WebGrease` reference declares no `Version=` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:104-107], so that divergence exists only in the binding redirect. **16 + 1 = 17 divergences**, and 2 + 3 + 16 = 21 pairs.

**The seventeen, grouped by edition** — one in MVC 5, fifteen in MVC 4, one in MVC 3. Per §1.4 every citation below is written out in full at the claim it supports, in the row or sentence that makes that claim, so no row takes its path from the row above it or from the line that introduces the table.

**MVC 5 — one.** Manifest: **[src/MVC5/MvcMusicStore/packages.config:30]**. Evidence: **[src/MVC5/MvcMusicStore/Web.config:56-57]**.

| Package pin | Assembly it supplies | Assembly version | Why this one is not in the project file |
| --- | --- | --- | --- |
| WebGrease `1.5.2` [src/MVC5/MvcMusicStore/packages.config:30] | `WebGrease` | **`1.5.2.14234`** [src/MVC5/MvcMusicStore/Web.config:56-57] | The project reference declares no `Version=` at all [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:104-107], so the redirect is the only place the assembly version appears |

**MVC 4 — fifteen.** Every row below carries both of its citations in full: the `Manifest` column cites the `packages.config` line that declares the pin, and the `Declared at` column cites the project-file reference block that declares the assembly version, so each row can be checked on its own.

| Package pin | Manifest | Assembly it supplies | Assembly version | Declared at |
| --- | --- | --- | --- | --- |
| Microsoft.AspNet.Mvc `4.0.20710.0` | [src/MVC4/MvcMusicStore/packages.config:14] | `System.Web.Mvc` | `4.0.0.0` | [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:92-95] |
| Microsoft.AspNet.Razor `2.0.20710.0` | [src/MVC4/MvcMusicStore/packages.config:15] | `System.Web.Razor` | `2.0.0.0` | [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:99-102] |
| Microsoft.AspNet.WebApi.Client `4.0.20710.0` | [src/MVC4/MvcMusicStore/packages.config:18] | `System.Net.Http.Formatting` | `4.0.0.0` | [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:77-79] |
| Microsoft.AspNet.WebApi.Core `4.0.20710.0` | [src/MVC4/MvcMusicStore/packages.config:19] | `System.Web.Http` | `4.0.0.0` | [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:86-88] |
| Microsoft.AspNet.WebApi.WebHost `4.0.20710.0` | [src/MVC4/MvcMusicStore/packages.config:20] | `System.Web.Http.WebHost` | `4.0.0.0` | [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:89-91] |
| Microsoft.AspNet.WebPages `2.0.20710.0` | [src/MVC4/MvcMusicStore/packages.config:21] | **Four assemblies from one pin:** `System.Web.Helpers`, `System.Web.WebPages`, `System.Web.WebPages.Deployment`, `System.Web.WebPages.Razor` | `2.0.0.0` — all four | [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:82-85], [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:103-106], [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:107-110], [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:111-114] |
| Microsoft.AspNet.WebPages.Data `2.0.20710.0` | [src/MVC4/MvcMusicStore/packages.config:22] | `WebMatrix.Data` | `2.0.0.0` | [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:115-118] |
| Microsoft.AspNet.WebPages.OAuth `2.0.20710.0` | [src/MVC4/MvcMusicStore/packages.config:23] | `Microsoft.Web.WebPages.OAuth` | `2.0.0.0` | [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:119-122] |
| Microsoft.AspNet.WebPages.WebData `2.0.20710.0` | [src/MVC4/MvcMusicStore/packages.config:24] | `WebMatrix.WebData` | `2.0.0.0` | [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:123-126] |
| DotNetOpenAuth.AspNet `4.0.3.12153` | [src/MVC4/MvcMusicStore/packages.config:3] | `DotNetOpenAuth.AspNet` | `4.0.0.0` | [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:127-130] |
| DotNetOpenAuth.Core `4.0.3.12153` | [src/MVC4/MvcMusicStore/packages.config:4] | `DotNetOpenAuth.Core` | `4.0.0.0` | [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:131-134] |
| DotNetOpenAuth.OAuth.Consumer `4.0.3.12153` | [src/MVC4/MvcMusicStore/packages.config:5] | `DotNetOpenAuth.OAuth.Consumer` | `4.0.0.0` | [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:135-138] |
| DotNetOpenAuth.OAuth.Core `4.0.3.12153` | [src/MVC4/MvcMusicStore/packages.config:6] | `DotNetOpenAuth.OAuth` — **the assembly name differs from the pin id as well as the version** | `4.0.0.0` | [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:139-142] |
| DotNetOpenAuth.OpenId.Core `4.0.3.12153` | [src/MVC4/MvcMusicStore/packages.config:7] | `DotNetOpenAuth.OpenId` — likewise | `4.0.0.0` | [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:143-146] |
| DotNetOpenAuth.OpenId.RelyingParty `4.0.3.12153` | [src/MVC4/MvcMusicStore/packages.config:8] | `DotNetOpenAuth.OpenId.RelyingParty` | `4.0.0.0` | [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:147-150] |

**MVC 3 — one.** Manifest: **[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/packages.config:7]**. Evidence: **[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:37-40]**.

| Package pin | Assembly it supplies | Assembly version | Note |
| --- | --- | --- | --- |
| EntityFramework `4.1.10331.0` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/packages.config:7] | `EntityFramework` | `4.1.0.0` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:37-40] | The only pin in this edition whose payload the project references at all (§3.3), so it is also the only divergence available to find here |

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

**What `packages.config` does and does not give you, stated exactly, because this is easy to get backwards.** The format is a **flat, exact list of every package installed into the project — transitive ones included**, each at a single exact version, with no ranges to resolve at restore time. Transitive entries sit in the list as first-class pins: MVC 5 declares `Antlr` `3.4.1.9004` [src/MVC5/MvcMusicStore/packages.config:3], `WebGrease` `1.5.2` [src/MVC5/MvcMusicStore/packages.config:30], `Owin` `1.0` [src/MVC5/MvcMusicStore/packages.config:28] and `Microsoft.Web.Infrastructure` `1.0.0.0` [src/MVC5/MvcMusicStore/packages.config:25], none of which any application source calls — they are in the manifest because the minification and OWIN stacks depend on them (§3.1.1, §3.1.2). Restore does not re-resolve a graph from package metadata; it installs the list. The two committed payloads confirm the same property from the other side: each contains exactly the packages its manifest names and nothing else (§7.2).

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

# TWO MECHANICAL PROPERTIES, each one load-bearing for the scope claim above:
#   -z / xargs -0   One tracked path contains spaces -- `src/MVC3/MVC Music Store - Tutorial
#                   - v3.0.pdf` -- and an unquoted `git ls-files | xargs grep` splits it into
#                   SEVEN operands: five nonexistent path fragments, and two literal `-` words
#                   that grep reads as standard input rather than as files. The PDF is never
#                   searched, so that form cannot establish "every tracked file" coverage.
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

### 8.2 Why no advisory identifier is asserted here, and how a reader obtains today's set

AAP 0.5.1 fixes the form this posture is recorded in, and §8.1 above is that form: the aged pins are recorded as **a class of finding with the exact pins**, a scanner is **directed to be run during implementation** (§8.3), and the assessment **asserts no specific advisory identifier** — because no scanner is part of this environment and the repository has no dependency-scanning configuration to cite. This document holds to that without qualification: **no advisory identifier, no per-pin severity, no advisory count and no warning tally appears anywhere in it** — and consequently no other deliverable can cite this one for such a figure, because there is none here to cite. Where a downstream document needs one, its source is the implementation-phase scan §8.3 directs, or a reader's own dated run of the route below.

What is recorded instead is the part that belongs to the repository and stays true: the exact pins with the manifest that declares each (§8.1); that they are 2011–2013 releases, self-hosted from the application's own directories and served with neither a Subresource Integrity attribute nor a Content Security Policy (§8.1); that the repository holds no scanning configuration of any kind (the commands below); and the route a reader takes to obtain the identifiers themselves, on the day they ask (below). That division is not caution for its own sake. A pin is a repository property, fixed for as long as the three manifests say what they say; what an advisory database says about that pin is a property of the database on a given day, of the client that queries it, and of the sources that client is configured with. Printing such a list inside an inventory whose every other figure is exact would set a silently ageing number beside figures that cannot age, and a reader six months later could not tell which of the two they were reading.

**The repository has no dependency-scanning capability, which is the premise AAP 0.5.1 states.** Verified below, and the commands are the evidence because an absence has no line to cite:

```bash
git ls-files | grep -c '^\.github/'                                  # -> 0   (no GitHub workflows or config directory)
git ls-files | grep -icE 'dependabot|renovate'                       # -> 0   (no Dependabot, no Renovate)
git ls-files '*packages.config' | grep -v '/packages/' \
  | xargs grep -hiE 'analyzer|StyleCop|FxCop|SonarAnalyzer|Roslynator' | wc -l   # -> 0   (no analyzer package in any manifest)
git ls-files | grep -iE '(\.ruleset$)|(Directory\.Build\.props)'      # -> (no output: neither exists in any edition)
git grep -in 'NuGetAudit' -- . ':(exclude)docs/'                     # -> (no output, exit 1): no audit property is set in tracked content
```

**The route, so the identifiers are one command away rather than absent.** NuGet's own restore-time audit reports advisories against the pins a restore resolves, so a reader with network access needs no additional tool for a first look. The workflow is the single mutating one this document quotes, in §10.2: it validates `$SCRATCH`, clones into it, restores the three solutions and checks every exit code, and it writes one log per edition inside that clone. Read the audit's warnings out of those three logs before the workflow's cleanup step discards the clone:

```bash
# Run the fail-closed workflow of §10.2 first; it leaves three logs inside $AUDIT.
# NU190 is the prefix of the NuGet audit's own warning codes. Each line it prints
# names one package, the pinned version, a severity and an advisory URL, and the
# output belongs to that run, on that date -- this document quotes none of it.
grep -h 'NU190' "$AUDIT/restore-mvc5.log" "$AUDIT/restore-mvc4.log" "$AUDIT/restore-mvc3.log"
```

Four bounds belong with that route, because a result read without them says more than it can:

- **It belongs to the run rather than to the repository.** The client version, the effective package sources and the advisory-database snapshot the client consulted all determine what it prints, and §10.2 records each of them beside the run that produced it. Two runs are comparable only where all three agree.
- **It sees the declared direct pins only.** The transitive graph is unpinned (§7.1), so the result describes neither transitive exposure nor what a future restore would resolve.
- **It cannot see the vendored copies** under `Scripts/`, `Content/` and `fonts/` (§3.1.4, §3.2.4). A package audit inspects packages; the committed asset files are ordinary tracked content that no package audit reaches, and per §8.1 those copies are the ones actually served to browsers.
- **It says nothing about the non-NuGet dependencies of §4.1** — the machine-wide ASP.NET MVC 3 Tools Update install and the SQL Server Compact 4.0 provider — which no package audit can assess.

**At NuGet's default audit level the warnings are advisory rather than fatal**, so a restore that raises them still exits `0` and nothing fails. That follows from the default rather than from any one run, and it holds while the repository sets no audit or warnings-as-errors property of its own — the last two commands above establish that it sets none, neither in tracked content nor in a `Directory.Build.props`, because it has none (§7.1). The finding that survives whatever a given run prints is therefore a governance one: **the audit's output is visible only to whoever happens to read restore output, and no build, tooling or CI artifact in this repository records it, retains it or fails a build because of it.** Nothing in the repository keeps it; this document keeps no dated copy of it either, deliberately and per AAP 0.5.1; so a reader's own run is the only place it exists. Deliverable 03 owns the gate that would change that, and §8.3 states what the implementation phase must do.

### 8.3 What must still happen

The restore-time audit of §8.2 is a floor a reader can reach for themselves, not a substitute for a scan. **Directive for the implementation phase, carried unchanged from AAP 0.5.1:** run a full software-composition analysis against the restored graph — direct *and* transitive — on a host with network access, extend it to the vendored client-side assets that no package audit inspects, record its dated output with the tool and version that produced it, and add scanning configuration to the repository so the result is gated rather than merely printed. Any advisory identifier quoted downstream must come from that recorded output, or from a reader's own dated run of the §10.2 workflow, and never from recollection.

**There is no figure of §8.2's to inherit or revalidate, and that is the point of the form AAP 0.5.1 sets.** §8.2 asserts no advisory identifier, no severity and no count, so nothing in this document ages into being wrong when the advisory database moves, and an approver reading this later has nothing to re-check here — what they want today they obtain from their own run, whose client, sources and advisory-database snapshot they hold. The scan directed above is therefore the first recorded advisory output this assessment will own, and it is the artifact downstream deliverables cite. What needs no revalidation at all is the class finding of §8.1 and the governance finding at the end of §8.2: the pins are 2011–2013 releases whatever today's advisory data says, and nothing in the repository records or gates on an audit either way.

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
| F-02-14 | Package version and assembly version differ in **seventeen** places, enumerated completely and in one place — 15 in MVC 4, one in MVC 3, and MVC 5's WebGrease `1.5.2` binding to `1.5.2.14234`, which the project-file census cannot see because that reference declares no `Version=` | §4.4, §4.4.1 | 04 |
| F-02-15 | A 2012 NuGet 2.0 client (630,784 bytes, `2.0.30828.5`) is committed and is required by MVC 4's restore, with the download fallback disabled | §5.1 | 08, 10 |
| F-02-16 | MVC 5's solution declares a `.nuget` folder that does not exist; MVC 5 has no restore wiring of its own | §5.2 | 10 |
| F-02-17 | **No package source is configured anywhere.** Technical Specification §3.3 is corrected: the v2 endpoint is inside a comment | §6.1, §6.2 | 04 |
| F-02-18 | The effective source set is not knowable from the repository, and private feeds cannot be ruled out | §6.3 | 04, 08 |
| F-02-19 | No lockfile and no central version management in any edition; transitive resolution is not reproducible | §7.1 | 08, 04 |
| F-02-20 | 215 tracked files, including 32 `.dll`/`.exe`, committed under two `packages/` trees despite `Packages/` [.gitignore:33] matching them on this case-insensitive checkout — `packages/*` [.gitignore:15] is root-anchored and reaches no nested path, so on a case-sensitive host no rule would ignore these two trees at all; MVC 5 commits none | §7.2 | 08, 10 |
| F-02-21 | A class of aged (2011–2013), self-hosted, SRI- and CSP-unprotected dependencies, enumerated by exact pin | §8.1 | 07 (risk), 04 (disposition) |
| F-02-22 | The aged pin set of F-02-21 is what a dependency audit lands on, and **this document asserts no advisory identifier, severity or count against it** — AAP 0.5.1's class-of-finding form is kept, and §8.2 records instead the route by which a reader obtains today's set from NuGet's own restore-time audit, with the four bounds on what such a result establishes | §8.1, §8.2 | 07 (risk), 04 (disposition) |
| F-02-23 | No build, tooling or CI artifact in the repository records, retains or gates on dependency-audit output; the audit's warnings scroll past on every clean restore, which still exits `0`, and nothing in the repository keeps them — so a reader's own dated run is the only place that output exists | §8.2, §8.3 | 03 (gate), 08 (severity) |

### 9.1 The reverse direction — the one `09` row this document discharges

The table above runs outward, from a finding of this document to the deliverable that acts on it. Deliverable 09 §8.3 requires the opposite traversal to terminate as well: its register's Consumers column names this document for exactly one row, and a reader arriving from that column must land on a **named item here**, which means this document has to print the `F-09-nn` identifier rather than merely say the right thing about the right packages.

| Row in 09's register | What the row names | Discharged here by | Check from the other end |
| --- | --- | --- | --- |
| **F-09-36** — Medium, MVC 5 and MVC 4, 09 §6.11 | Eleven external-authentication packages deployed across two editions to serve zero enabled providers, of which four implement a named provider | **§3.1.2** for MVC 5's four named-provider pins and the fifth `Security.*` pin that is not a provider, and **§3.2.3** for MVC 4's seven — six DotNetOpenAuth pins plus the `Microsoft.AspNet.WebPages.OAuth` pin that ships the external-login API itself. Both sections print the identifier; §9's rows F-02-03 and F-02-08 are the same two facts in summary form | 09's Consumers cell for the row reads `05, 02` |

**Only the inventory half is discharged here.** That eleven packages are pinned, referenced and deployed for a capability no request can reach is an inventory fact and is this document's. That the live external-login code path is deployed attack surface is 09's severity judgement, and the per-package removal outcome is deliverable 04's — §1.2 states both boundaries, and neither is restated in the row above. **No other `F-09` identifier is claimed by this document**: 09 §8.3 is explicit that a consumer may not acquire a register row by citing it, and F-09-36 is the only row whose cell names this deliverable.

---

## 10. Reproducibility appendix

Every command this document quotes, in one place, in two classes that are deliberately kept apart. **§10.1 collects the read-only evidence commands**, which write nothing and are safe to run in the assessed checkout itself. **§10.2 collects the one mutating workflow this document has** — the three `nuget restore` invocations that produce the restore-audit output §8.2 describes — classified operation by operation, fail-closed by construction, confined to a private scratch directory the caller names and the block validates before anything is written, and discarding the clone it creates. **There is no second copy of that workflow, and its absence is a correction rather than a style note.** An earlier draft of this appendix carried an unvalidated variant that named a shared temporary directory (`/tmp/nuget-audit`) as its scratch path, restored into that path with no containment or pre-existence check, and removed nothing afterwards. It is deleted rather than reconciled: two workflows with different safety properties cannot both be the one this document stands behind, and the one retained in §10.2 is the safe one. The canonical forms are POSIX per AAP 0.11.3; on the Windows verification host they were run through the bundled Git-for-Windows `bash` from the repository root, and the values shown are the values observed.

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
# usable whether or not the generated trees of §1.3 are present:
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
# showed the eight generated trees of §1.3 while they existed, and equally the
# only one that shows any generated tree present when it is run now. [Oo]bj/
# (.gitignore:1) and [Bb]in/ (.gitignore:2) covered the six build trees; the two
# nested packages trees were matched by Packages/ (.gitignore:33), which is
# depth-independent and reaches them only because core.ignorecase is true here.
# packages/* (.gitignore:15) is root-anchored and matches no nested path:
git status --porcelain --ignored   # (empty only when nothing ignored is present)
# 4. What an ignore-aware clean would remove, listed rather than deleted:
git clean -ndX                     # (lists every ignored tree present, and nothing else)
```

### 10.2 Mutating operations — three package-mutating restores, disposable clone only

This is the whole of what this document changes, and it exists solely so that the restore-time audit route of §8.2 can be run by a reader. It is quoted here and nowhere else in this document. Calling it "three commands" would undercount it, so the workflow below is classified line by line:

| Operation | What it mutates | Where the effect lands |
| --- | --- | --- |
| `$SCRATCH` validation — `:?`, `case`, `git rev-parse`, `pwd -P` | **Nothing**: it reads, compares and refuses | The invoking shell only; if it refuses, no later line runs at all |
| `mktemp -d "$SCRATCH_REAL/mvcmusicstore-audit.XXXXXXXX"` | Creates one directory, exclusively and under a name it chooses (mode 0700 where the filesystem honours it) | `$SCRATCH` only. The name is unpredictable and creation is exclusive, so a pre-created or symlinked path is a failure rather than a target (CWE-377, CWE-379); the resolved path is re-asserted under `$SCRATCH_REAL` before anything is written |
| The audit preconditions — `command -v`, the parsed client version, `git grep`/`env` for `NuGetAudit*` | **Nothing**: they read and refuse | The invoking shell and the clone it reads; no restore runs unless all of them pass |
| `git clone --no-hardlinks "$REPO" "$AUDIT"` | Creates a new working tree | `$SCRATCH` only, inside the `mktemp` directory; it **reads** the repository it is run from |
| The **three** `nuget restore` invocations | **Package payloads**: each downloads and writes `packages/` into whatever tree it is run in | The clone: each solution path is rooted at `$AUDIT`, and the block refuses to reach them at all unless the clone succeeded |
| `> "$AUDIT/restore-mvc*.log" 2>&1` and the `nu190-all.log` concatenation | Creates four log files | Inside the clone, discarded with it. `*.log` is gitignored [.gitignore:25], so they cannot dirty even the clone's tracked status |
| `rm -rf "$AUDIT"` inside the `EXIT` trap, after the path is re-tested against the `mktemp` name pattern | Deletes the `mktemp` directory and the clone inside it | `$SCRATCH` only, and only the directory this run created. It runs on **every** exit path — normal end, failed check or signal — so an interrupted run leaves no payload behind |

So there are **three package-mutating commands and five scratch-confined operations**, preceded by two steps that mutate nothing and refuse to let any of them run — the `$SCRATCH` validation and the audit preconditions — and only the three carry any risk to the assessed checkout. **The block below is fail-closed, and for these three commands that is the whole safety property:** `set -euo pipefail` plus a checked exit code on every step means a failed `$SCRATCH` validation, a failed `mktemp`, a failed `git clone` or a failed restore stops the run instead of letting the next line proceed — which matters because a `nuget restore` that is reached without a proven clone writes package payload into whatever tree the caller happens to be standing in, and this document’s entire premise is that that tree is never the assessed checkout. **The directory it writes into is created by `mktemp -d` rather than named**, so it cannot be pre-created, symlinked or reused by anyone else on the host, and its removal is a trap rather than a final line, so it happens even when the run does not reach that line. Its validation, temporary-directory, precondition, clone, checked-exit-code restore and trap-cleanup steps are identical to the §8.2 copy line for line — only the parsing tail differs, which is where the two sections’ purposes diverge — so the two cannot disagree about safety. Each of the three must be run in a disposable clone or scratch checkout outside the authoritative repository, and **never in place against the assessed checkout**. Because a restored `src/MVC5/packages` or `src/MVC4/packages` is matched by `Packages/` [.gitignore:33] — the depth-independent rule, which reaches a lowercase directory only where `core.ignorecase` is true, as it is on this host; `packages/*` [.gitignore:15] is anchored to the repository root and reaches no nested path, and on a case-sensitive host neither rule matches a nested lowercase `packages` at all — `git status --porcelain` will not show the payload one of them writes on a host like this one; `git status --porcelain --ignored` will, and discarding the clone makes the question moot. §1.3 records what happened when restore and build runs were made in place earlier in this assessment — eight ignored trees, removed once the assessment had finished with them — and the four-command acceptance check, including the reason only its first two commands are a durable claim.

```bash
# MUTATING. Network access required. Never run these in the assessed checkout, or
# in any repository checkout: $SCRATCH is any private directory outside the
# authoritative repository, and the block refuses to proceed if it is not.
# Fail-closed by construction: $SCRATCH is
# validated before anything else, the audit directory is created by `mktemp -d`
# and removed by a trap on every exit path, the block stops at the first failure,
# and every path is rooted at the clone, so no restore can run in the caller’s
# directory.
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

# 2 - The audit directory, created by `mktemp -d` and removed by a trap. Three
#     properties matter, and a hand-named path under the shared system temporary
#     directory has none of
#     them: the name is unpredictable, so a local attacker cannot pre-create or
#     symlink it; creation FAILS rather than reusing anything already there,
#     because mktemp creates it exclusively (and at mode 0700 where the
#     filesystem honours it - exclusive creation is the property relied on here,
#     not the emulated mode this Windows host reports); and the trap removes it on
#     every exit path, including a signal. The resolved physical path is then
#     re-asserted to sit under $SCRATCH_REAL before anything is written to it,
#     which is what a symlink swapped between those two steps would have to
#     defeat. This shell never changes directory: the only `cd` calls above and
#     below are inside command substitutions, which cannot outlive the subshell
#     they run in, so the working directory the caller started in is still the
#     working directory at the end. Every path from here on is rooted at $AUDIT,
#     which is what makes it impossible for a restore to inherit the caller’s
#     working directory.
audit_cleanup() {                    # every exit path, including a signal
  [ -n "${AUDIT:-}" ] || return 0
  case $AUDIT in
    "$SCRATCH_REAL"/mvcmusicstore-audit.??????*) rm -rf "$AUDIT" ;;
    *) echo "cleanup refused: $AUDIT is not a path this run created" >&2 ;;
  esac
}
trap audit_cleanup EXIT              # normal end, failed step, or an `exit` below
trap 'exit 130' HUP INT TERM         # a signal becomes an exit, so EXIT still fires
AUDIT=$(mktemp -d "$SCRATCH_REAL/mvcmusicstore-audit.XXXXXXXX")
AUDIT_REAL=$(cd "$AUDIT" && pwd -P)  # resolve first, then re-assert containment
case $AUDIT_REAL in
  "$SCRATCH_REAL"/mvcmusicstore-audit.??????*) AUDIT=$AUDIT_REAL ;;
  *) echo "refusing $AUDIT_REAL: it does not resolve under $SCRATCH_REAL" >&2
     exit 1 ;;
esac
git clone --no-hardlinks "$REPO_TOP" "$AUDIT"   # mktemp's directory is empty, which
if [ ! -d "$AUDIT/.git" ]; then                 # is the one case git clone accepts
  echo "clone produced no working tree: $AUDIT" >&2
  exit 1
fi

# 3 - The audit's own preconditions, checked before any restore runs, because a
#     count is interpretable only if the client is one that audits and nothing has
#     turned the audit down. The client is resolved to ONE path rather than left to
#     PATH; its version is PARSED and compared numerically against the 6.11 floor
#     this block requires; and no NuGetAudit* property may be set in tracked
#     content or in the environment — MSBuild reads environment variables as
#     properties, so an audit level set there would change the result silently.
#     `sed -n 1p` reads its input to the end deliberately: `head -1` would close
#     the pipe on the client's first line, and under `pipefail` that SIGPIPE would
#     abort the block for a reason that has nothing to do with the audit.
NUGET="$(command -v nuget || true)"
[ -n "$NUGET" ] || { echo "no nuget client resolved" >&2; exit 1; }
CLIENT="$("$NUGET" help | sed -n 1p)"          # e.g. NuGet Version: 6.11.1.2
VERSION="${CLIENT##*NuGet Version: }"
MAJOR="${VERSION%%.*}"; REST="${VERSION#*.}"; MINOR="${REST%%.*}"
case "$MAJOR.$MINOR" in
  *[!0-9.]*|.*|*.) echo "cannot parse a version from [$CLIENT]" >&2; exit 1 ;;
esac
if [ "$MAJOR" -lt 6 ] || { [ "$MAJOR" -eq 6 ] && [ "$MINOR" -lt 11 ]; }; then
  echo "client is $VERSION; this block requires 6.11 or later — refusing to count" >&2
  exit 1
fi
if git -C "$AUDIT" grep -in 'NuGetAudit' -- . ':(exclude)docs/'; then
  echo "an audit property is set in tracked content — the defaults no longer hold" >&2
  exit 1
fi
if env | grep -i 'nugetaudit'; then
  echo "an audit property is set in the environment — the defaults no longer hold" >&2
  exit 1
fi

# 4 - The three package-mutating commands, each with its exit code checked, so no
#     count is ever parsed from a partial log, and each proved to have AUDITED:
#     NU1901-NU1904 present says the audit ran, NU1900/NU1905 absent says it ran
#     undegraded rather than against an unreachable or dataless source. A failure
#     here exits, and step 2's trap discards the clone on the way out, so the
#     failing log's tail is echoed to stderr first — redirect this block's stderr
#     to a file outside $SCRATCH if a full log has to survive the failure.
#     -Verbosity detailed puts the client version into the log itself; the
#     warnings appear at either verbosity.
restore() {          # $1 = solution path inside the clone, $2 = log file name
  if ! "$NUGET" restore "$AUDIT/$1" -NonInteractive -Verbosity detailed \
       > "$AUDIT/$2" 2>&1; then
    echo "restore FAILED: $1 - stopping before any parsing; log tail follows" >&2
    tail -n 40 "$AUDIT/$2" >&2
    exit 1
  fi
  audited="$(grep -cE 'NU190[1-4]' "$AUDIT/$2" || true)"
  [ "$audited" -gt 0 ] || { echo "$2: no NU1901-NU1904 — the audit did not run" >&2; exit 1; }
  if grep -qE 'NU1900|NU1905' "$AUDIT/$2"; then
    echo "$2: audit degraded (NU1900/NU1905) — the counts are incomplete" >&2
    exit 1
  fi
  echo "exit=0  audited=$audited  client=$VERSION  $1"
}
restore src/MVC5/MvcMusicStore.sln                         restore-mvc5.log
restore src/MVC4/MvcMusicStore.sln                         restore-mvc4.log
restore src/MVC3/MvcMusicStore-Completed/MvcMusicStore.sln  restore-mvc3.log

L5=$AUDIT/restore-mvc5.log
L4=$AUDIT/restore-mvc4.log
L3=$AUDIT/restore-mvc3.log
ALL=$AUDIT/nu190-all.log

# 5 - The audit output THIS run produced, reached only if all three restores exited
#     0. No expected value is stated here and none is stated in §8.2: the warnings
#     belong to the advisory database, the client and the sources of this run, and
#     they are the reader's result rather than this document's (§8.2). Capture the
#     logs as BYTES: a PowerShell 5.1 ">" redirect writes UTF-16LE, in which grep
#     matches nothing - a silent zero that reads like a clean audit.
grep -h 'NU190' "$L5" "$L4" "$L3"    # the warning lines, per edition, in restore order
cat "$L5" "$L4" "$L3" > "$ALL"
grep -c 'NU190' "$ALL"               # how many this run raised in total
grep -o 'GHSA-[a-z0-9-]*' "$ALL" | sort -u   # the distinct advisory identifiers of THIS run

# 6 - Record the provenance any such result must be quoted with, taken here, from
#     these same logs, before the cleanup below discards them. Without all four a
#     later run cannot be compared with this one:
nuget help | head -1        # client version, e.g. NuGet Version: 6.11.1.2
nuget sources list          # the effective sources, which the repository does not specify (§6)
date -u +%Y-%m-%dT%H:%M:%SZ # the UTC date the result belongs to
grep -h 'vulnerability\.\(base\|update\)\.json' "$ALL" | sort -u  # advisory-DB snapshot ids
sha256sum "$L5" "$L4" "$L3" # identify THIS run's logs; two runs are comparable only where
wc -c "$L5" "$L4" "$L3"     # the client, the sources and the snapshot ids above all agree

# What the restores wrote, and why porcelain status is the wrong check. `git -C`
# reads the clone without moving this shell into it:
git -C "$AUDIT" status --porcelain            # empty - packages/ is ignored, so this proves nothing
git -C "$AUDIT" status --porcelain --ignored  # shows the new src/MVC5/packages and src/MVC4/packages
                                              # trees, and the four *.log files written above
git -C "$AUDIT" clean -ndX                    # the same content, as an ignore-aware clean would list it

# The warning lines themselves stay in these three logs and in the reader's own
# record of this run; no copy of them is retained in this document (§8.2), so a
# later run is compared against a reader's saved output rather than against a
# table here.

# 7 - Nothing discards the clone here: step 2's EXIT trap does, so the mktemp
#     directory goes whether this block reached its end, exited at a failed check,
#     or was interrupted by a signal. The trap re-tests the path against the
#     mktemp name pattern before removing it, so a $SCRATCH that changed mid-run,
#     or a symlink swapped underneath it, cannot turn the cleanup into an rm -rf
#     of anything but the directory this run created.
#
# The Windows equivalent, if this workflow is run in PowerShell rather than
# through the Git-for-Windows bash of §1.4, needs the same three properties and is
# stated once here: create the directory as
# `New-Item -ItemType Directory -Path (Join-Path $env:TEMP ('mvcmusicstore-audit.' +
# [System.IO.Path]::GetRandomFileName()))` - an unpredictable name, and without
# -Force the creation fails rather than reusing an existing path - and wrap
# everything after it in `try { ... } finally { Remove-Item -Recurse -Force $audit }`,
# which is PowerShell's equivalent of the EXIT trap. $env:TEMP is outside every
# repository checkout on this host, which is the property §8.2 requires of it.

# The same two ignored-aware checks, plus the tracked diff, are the acceptance
# check of §1.3. They are what established that the eight generated trees this
# assessment left in the checkout had been removed when it finished with them,
# and they are equally what reports any generated tree a later run leaves there
# — including one this document did not produce, which is why §1.3 treats their
# empty result as a pre-commit condition rather than a standing claim. They are
# in §10.1, run against the assessed checkout rather than the clone.
```

---

*End of deliverable 02. Index and requirement map: [README](README.md). No tracked repository file outside `docs/modernization/` was created, modified or deleted in the production of this document. The generated, ignored content that the assessment's restore and build runs left in the checkout — eight trees, removed once the assessment had finished with them — is recorded in §1.3, together with the four-command check and the reason only its first two commands are a durable claim: the other two see ignored content, so they report whatever generated output is present when they run, whoever produced it.*
