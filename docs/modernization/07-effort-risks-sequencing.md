
# 07 — Estimated Effort, Risks and Sequencing

> **Status: assessment output. This document authorizes nothing.**
> It sizes and sequences work that has **not** been approved, and it modifies no pre-existing repository
> file.
> The approval gate is stated by [03 §2](03-modernization-roadmap.md); the first workstream in
> [section 5](#5-effort-by-workstream) is obtaining the approval itself.

---

## 1. Purpose, ownership and authoring contract

### 1.1 What this document is

This deliverable answers the user's requirement **"Estimated effort, risks, and sequencing"**. It
contains three things and nothing else:

1. **An effort model** — a band of estimates, in a stated unit, for each workstream that
   [03](03-modernization-roadmap.md) defines, with its assumptions and confidence stated rather than
   implied.
2. **A risk register** — every entry carrying likelihood, impact, mitigation, contingency, trigger,
   owner and affected workstream.
3. **A dependency-ordered sequence**, with the parallelism the dependency graph permits.

### 1.2 This document is the only place effort figures appear

Under the resolution recorded in AAP §0.1.3, **effort estimation lives entirely inside this
deliverable.** Every other document in the set routes questions of size here and states no figure of
its own. That routing is asserted by the owning documents themselves, not merely by the plan:

- [03 §1.5](03-modernization-roadmap.md) routes *"the effort model and its bands"* and *"the risk
  register, including the support-window entry"* to this document, recording that **no figure is
  stated there**.
- [04 §2.2](04-dotnet8-migration-strategy.md) records that the target framework's support window *"is
  an approval decision, and it belongs to 07"*.
- [05 §11.5](05-aspnet-core-migration-approach.md) row 7 records that **07 carries the compatibility
  risk**; [05 §11.6](05-aspnet-core-migration-approach.md) records that a duration *"is 07's to
  state"*; [05 §12.5](05-aspnet-core-migration-approach.md) records that **07 carries the visual
  review as a task**.
- [06 §10.4](06-azure-hosting-recommendations.md) records that **07 carries the compatibility loss as
  a risk with a named approval owner**, and that accepting it *"is not this document's to accept"*.
- [08 §12](08-technical-debt-register.md) is written explicitly as a handoff to this document,
  separating quantities that are safe to estimate against from quantities that are not.

**The converse obligation binds this document.** Everything in [section 1.4](#14-what-this-document-does-not-own--the-routing-table) is
cited here and never restated. A restatement in different words reads downstream as a second decision,
and a reader who finds two picks one.

### 1.3 What this document is not

- **It is not a schedule.** **No start or finish, no duration, no calendar position and no numbered
  delivery wave is stated for any workstream, task or gate in it.** [Section 8](#8-sequencing) explains why
  that is a feature: an effort band plus a dependency order lets a reader build a schedule from *their own*
  capacity and staffing assumptions, whereas a schedule computed here would silently embed assumptions that
  are not this assessment's to make.

  **Stated that precisely because a blanket "no date appears here" would be false.** Three dates do appear,
  all in [R1](#r1--the-target-framework-support-window), and none of them is a schedule: the target
  framework's release date, its published end-of-support date and the next long-term release's, which are
  **external facts about someone else's lifecycle** and the whole substance of the risk an approver has to
  decide on. A date that constrains the work is not a date this document assigns to the work, and the
  difference is what makes the first claim checkable.
- **It is not a re-argument of any decision.** The target framework, the hosting target, the cutover
  approach and the browser matrix are settled elsewhere. Where this document carries one as a risk, it
  carries the *consequence and the decision the approver still has*, not a reopened comparison.
- **It is not a commitment.** These are estimates with stated bands and stated confidence. A band is
  an honest statement of what is not yet known, and [section 4.4](#44-confidence-and-its-reason)
  states exactly which unknowns drive the width of each one.

### 1.4 What this document does not own — the routing table

| Fact or decision | Owner | This document's use of it |
| --- | --- | --- |
| Target framework, the SDK band, every package pin | [04](04-dotnet8-migration-strategy.md) §2, §3, §7–§9 | Named as "the target framework". The value, the band and the pins are not restated. [R1](#r1--the-target-framework-support-window) owns the *risk*, which is a different thing from the strategy |
| The future application map | [04](04-dotnet8-migration-strategy.md) §12 | The file inventory behind the port estimate. No file list is reproduced |
| Pipeline, DI, configuration, EF Core, views, static assets, anti-forgery, the JSON contract | [05](05-aspnet-core-migration-approach.md) §2–§9 | The scope behind W7's estimate |
| The test suite's architecture and its required coverage | [05](05-aspnet-core-migration-approach.md) §12 | The scope behind W4's estimate. Not redesigned here |
| The two data migrations and the schema-extraction design | [05](05-aspnet-core-migration-approach.md) §5 | The scope behind W3, W8 and W9 |
| **The cutover approach and its accepted losses** | [05](05-aspnet-core-migration-approach.md) §11 | [R9](#r9--cutover-re-authentication-and-anonymous-cart-loss) carries the two losses as risks. The decision is **not** reopened |
| **The set of approved deltas and their approval owners** | [05](05-aspnet-core-migration-approach.md) §11.5 | [Section 7.2](#72-the-approved-delta-sign-offs) sizes the *act of obtaining* the approvals |
| **Hosting target, deployment model, observability approach** | [06](06-azure-hosting-recommendations.md) §2, §9 | Named only. The mechanism is not restated |
| **The browser matrix** | [06](06-azure-hosting-recommendations.md) §10.4 | [R7](#r7--the-narrowed-browser-matrix) carries the compatibility loss. The matrix itself is not restated |
| The DDL principal and the provisioning order, **including the fixed order of the data movement** | [06](06-azure-hosting-recommendations.md) §6.2, §6.3 | The scope behind W10. That order is enforced **inside the two runs that move data** rather than by a workstream-level wait [03 §4.2](03-modernization-roadmap.md), and [§8.2](#82-concurrency-permitted-by-the-graph) records what the graph therefore permits |
| **The interim hosting option's authentication path** — selected, with its owner and its two exit triggers | [06](06-azure-hosting-recommendations.md) §5.3–§5.5 | [R15](#r17--the-interim-hosting-paths-stored-credential-and-a-time-box-with-no-box) carries the residual that the exception outlives its term. The selection, its cost and its own residual risks are not restated |
| **Per-edition build outcomes** | [10](10-build-and-deployment-requirements.md) §1.2, §3 | [R2](#r2--the-migration-sources-build-reproducibility) cites the carried status — *blocked pending a Windows verification run* — and the observed outcome recorded beneath it. Neither the status, the outcome nor its diagnosis is restated |
| **The 22 blockers and their two groups** | [12](12-migration-blockers.md) §2.3 | Work items behind W7's estimate |
| **The categorized debt register and the counting methods** | [08](08-technical-debt-register.md) §2, §5–§11 | The counting rule this document is bound by, and an estimation input |
| Every package pin as-is; the restore-source finding | [02](02-dependency-inventory.md) §3, §6 | Scope behind W6; [R11](#r11--the-effective-package-source-set-is-not-knowable-from-the-repository) carries the source finding as a risk |
| Verified counts, code volume, view topology, asset groups | [01](01-architecture-overview.md) §2.3–§2.5 | Estimation inputs, cited per figure |
| The DDL principal and the provisioning order | [06](06-azure-hosting-recommendations.md) §6.2, §6.3 | The scope behind W10 |
| **The 23 blockers and their two groups** | [12](12-migration-blockers.md) §2.3 | Work items behind W7's estimate |
| The DDL principal and the provisioning order | [06](06-azure-hosting-recommendations.md) §6.2, §6.3 | The scope behind W10 |
| **The set of approval-owned additions and their owners** | [05](05-aspnet-core-migration-approach.md) §11.7 | The second approval register. [Section 7.2](#72-the-approved-delta-sign-offs) sizes obtaining these decisions too, and states its two counts separately |
| The DDL principal and the provisioning order | [06](06-azure-hosting-recommendations.md) §6.2, §6.3 | The scope behind W10 |
| **Per-edition build outcomes**, and the migration source's build status | [10](10-build-and-deployment-requirements.md) §3, §5.4 | [R2](#r2--the-migration-sources-build-reproducibility) cites the recorded status. Neither the status nor its diagnosis is restated |

### 1.5 Authoring contract, and the absence of user rules

**No user-specified rules were provided for this project.** `review_rules` returns exactly *"No user
rules provided."*, re-verified while authoring this document. There is therefore no rule to name,
summarize or cite, and no rule forcing a file into scope. Per the governing instruction, that absence
is **not** treated as licence to lower the bar; enterprise-standard best practice applies instead, and
the four contracts below are the ones this document is actually held to.

1. **Citation contract** (AAP §0.4.1, §0.11.3). Every as-is claim carries an inline
   `[<path>:<locator>]` citation that resolves in the checkout. Every claim that ranges over the whole
   repository — a count or an absence — carries the **command that reproduces it**, because such a
   claim has no single line to point at. [Appendix A](#appendix-a--reproducibility) collects them.
   A claim that rests on something **outside** the repository — a support lifecycle, a platform
   behaviour, a framework's published browser support — carries an inline
   `[<publisher>, *<page title>*, <url> — verified <date>]` primary source instead, because no
   repository path can establish it and an undated external claim cannot be re-checked. The six such
   sources are listed in [A.8](#a8-external-primary-sources).
2. **Every effort figure is traceable and method-labelled.** No figure appears without both the count
   it derives from and the counting method that count uses. [Section 3](#3-the-two-counting-methods)
   states the rule; [section 4.1](#41-the-estimation-basis-every-input-with-its-method) is the
   traceability table.
3. **One fact, one owner** (AAP §0.11.4). [Section 1.4](#14-what-this-document-does-not-own--the-routing-table) is the routing table.
4. **No modification of any repository file.** [Section 2](#2-the-no-modification-constraint).

Markdown line length follows the corpus convention recorded in [`README.md` § 10 Markdown line-length convention](README.md#10-markdown-line-length-convention).

---

## 2. The no-modification constraint

The user directed **"Do not make code changes initially"** and **"Focus on assessment and planning
before implementation"**. The project's attached environment setup instructions restate the same gate
independently: **"Do not modify code until assessment and modernization plan are approved."** Two
inputs agreeing on it is why the boundary extends even to the defects this assessment identifies —
they are documented, not repaired.

**This document creates one file and modifies no tracked file — and the attestation has to say what the
assessment *did* write into the checkout, not only what it did not.** What it wrote was the residue of
**unqualified historical restore and build operations**: runs made against this checkout that left
**eight gitignored payload and output trees** behind — a `bin` and an `obj` under each of the three
editions' projects, plus a restored `packages` tree under `src/MVC4` and another under `src/MVC5`. All
eight have been removed and their absence verified. None was ever tracked and none is in the checkpoint
commit; the per-tree record and the standing rule that follows from it belong to
[10 §1.4](10-build-and-deployment-requirements.md). **They are not build evidence and are not counted as
any**: none of them recorded the fields a build result has to carry, so
[10](10-build-and-deployment-requirements.md) owns the evidence and the status this document estimates
against, and under it the **migration source remains blocked pending a Windows verification run** — which
is what [W2](#w2--mvc-5-build-reproduction-and-the-restore-precondition) and
[R2](#r2--the-migration-sources-build-reproducibility) are sized and written against. The residue is
therefore a repository-state fact, reported here because the attestation below is this document's, and not
a build outcome of any kind.

**That is why the acceptance check is four commands and not one, and it is worth stating in an effort
document rather than delegating.** All eight trees are ignored here, though not all by the rule a reader
reaches for first. `[Oo]bj/` [.gitignore:1] and `[Bb]in/` [.gitignore:2] cover the six build-output
trees. The two nested `packages` trees are matched by **`Packages/` [.gitignore:33]** and **not** by
`packages/*` [.gitignore:15]: a pattern with an interior separator is anchored to the repository root,
while a pattern whose only separator is trailing matches a directory of that name at any depth. That
analysis is [04 §A.6](04-dotnet8-migration-strategy.md)'s and this document aligns to it rather than
offering a variant. `Packages/` reaches the lowercase directories **only because
`git config core.ignorecase` reports `true` on this checkout** — probed with
`git check-ignore -v --no-index`, the form that tests a rule rather than a tracked path
([A.8](#a8-external-primary-sources), source 5), each nested path reports `.gitignore:33`, and under
`-c core.ignorecase=false` neither reports anything at all. **On a case-sensitive host those two trees
are therefore not ignored**, and which of the four commands can see them changes with the host: bare
porcelain would list them as untracked while the two ignore-aware commands would not class them as
ignored payload. That is one more reason the check is the four together rather than any one of them. On
this checkout the consequence is the one that matters here: **bare `git status --porcelain` and a
tracked-file diff both report a clean tree while a hundred megabytes of generated output sits in it** —
which is exactly what they would have reported for the whole period those eight trees existed. An
acceptance check that cannot observe the thing it is cited as proving is a **verification gap, not a
housekeeping slip**: every figure
in this document is a claim about work that has not started, and a reader who cannot trust the
repository-state attestation has no independent way to test the honesty of the estimate above it. AAP
§0.11.5 makes the final check binary, so all four commands are run and all four have to agree:
`git status --porcelain` (0 lines), `git status --porcelain --ignored` (0 lines, and no `!!` row),
`git clean -ndX` (empty), and the committed diff against the pre-assessment baseline,
`git diff --name-status ea2552d6eda7c20e9477a512e5c615665618cf35..HEAD`, which returns exactly thirteen
rows, every one an `A` for a file under `docs/modernization/`, with no `M` or `D` against any existing
file. Working-tree status alone is **not** the check twice over: at the committed checkpoint
`git status --porcelain` is empty, so it evidences a clean tree rather than what this work changed, and
by itself it could not have seen the eight trees at all. All four, with their observed output, are in
[A.6](#a6-the-constraint-this-work-was-held-to).

**The distinction that makes this document possible is mutation versus specification.** Estimating a
change is not making one. A deliverable would fail its acceptance criteria just as surely by declining
to specify the size of a change — mistaking the prohibition on *mutating* for a prohibition on
*planning* — as by mutating something.

---

## 3. The two counting methods

### 3.1 The rule, and why it is the sharpest constraint on this document

[08 §2](08-technical-debt-register.md) establishes that this codebase supports two line-counting
methods and binds every downstream figure to declare which one it uses. **This document is the primary
consumer of that rule**, because it is the only document that multiplies counts into estimates.

| Method | What it counts | What it is for |
| --- | --- | --- |
| **Physical lines and diff counts** | Every line, blank or not; and `diff` output lines between two files | **Duplication only.** It is what a file comparison produces |
| **Non-blank lines, excluding `Properties/AssemblyInfo.cs`** | Lines that are neither empty nor whitespace-only, summed per file; assembly metadata excluded because the target expresses it as MSBuild properties | **Effort sizing.** It is the volume of code a human must read, decide about and port |

**The two differ by roughly ten percent on this codebase**, so mixing them silently would put every
derived estimate out. They are never mixed in one sentence here, and every figure below carries its
method.

### 3.2 The one substitution that would corrupt this document

> **`AccountController.cs` is 382 non-blank lines — the sizing metric — and that is the figure the
> authentication rewrite is estimated against**
> [src/MVC5/MvcMusicStore/Controllers/AccountController.cs: whole file, non-blank count]. The locator is
> the file in its entirety rather than a line range, because the claim *is* a whole-file count and the
> range that would express it is the physical count this sub-section forbids quoting; the reproducing
> command is [A.1](#a1-the-sizing-metric-inputs-15).
> [08 §3.3](08-technical-debt-register.md) also records a **physical** line count for the same file as
> part of its duplication comparison. **That physical figure is not a sizing figure and appears
> nowhere in this document**, because quoting it in an effort table would inflate the authentication
> estimate by about ten percent while looking entirely reasonable.

Per [08 §12.2](08-technical-debt-register.md), the following are **excluded as effort inputs** and are
used nowhere below: the physical line counts for `AccountController.cs`; every diff-line count in
[08 §3](08-technical-debt-register.md); **every physical count of the seed file**, whose only figure here
is **820 non-blank**; **severity ratings**, which measure consequence and not cost; and **entry counts** —
"28 debt findings" is not a workload, because the entries are not comparable units.

> **The seed file is the one place where three different counts of a single file are all defensible, so
> the exclusion is stated as a rule rather than as one number.** `src/MVC5/MvcMusicStore/Models/SampleData.cs`
> has **no terminal newline** — its last byte is `}` — and that one byte splits the physical count in two:
> counting **line-feed bytes** gives **826**, counting **content lines** gives **827**, and the non-blank
> sizing count gives **820** [src/MVC5/MvcMusicStore/Models/SampleData.cs: whole file, three counting rules;
> the commands are in [A.1](#a1-the-sizing-metric-inputs-15)]. All three are correct answers to different
> questions. **Only 820 is an input to this document**, and neither 826 nor 827 appears in any figure
> below. Where another deliverable states a physical figure for this file, it is answering a
> duplication-or-weight question under its own rule and is not comparable with anything in
> [section 5](#5-effort-by-workstream) — which is exactly the substitution
> [§3.2](#32-the-one-substitution-that-would-corrupt-this-document) forbids, one file further on.

### 3.3 A third kind of count, named so it is not mistaken for either

Some inputs are **file counts** or **call-site counts** — 29 views, 27 static assets, 10 injection
sites, 22 blockers. These are neither line-count method. They are labelled **"file count"** or
**"site count"** wherever they appear, because an unlabelled integer next to two labelled line metrics
invites exactly the confusion §3.1 exists to prevent.

**The test suite is sized in a further count of this same third kind — the "case count" — and it is
named here for the same reason.** [05 §12.4](05-aspnet-core-migration-approach.md) publishes its coverage as **rows** resolved
into **cases** — one lettered scenario entry — and separately as **fixture executions**, since a case
authored once may execute against both fixtures. This document estimates against the **case**, labels
every such figure **"case count"**, and keeps the **row** and **execution** counts beside it rather than
in place of it: rows measure how the matrix is organized, cases measure what must be authored, and
executions measure what must be run. Using rows as the sizing unit would understate the work by the
factor between the two, and using executions would overstate it by counting one authored case twice.

**Why this is an admissible unit while the entry counts §3.2 excludes are not.** The exclusion there is
of counts whose members are **not comparable units of work** — "28 debt findings" spans a stale solution
file and a destructive schema initializer. A case is a comparable unit by construction: each is one
request or request pair with named response assertions and a stated database invariant, authored against
one fixture. That is what makes a per-case rate meaningful, and every per-case rate in
[section 5.2](#52-basis-of-estimate-per-workstream) is printed rather than implied so it can be
disagreed with.

**The same test, and not the label, decides the figures below that are marked "entry count".** §3.2
excludes a *register total* — "28 debt findings" — because its members are unlike each other. It does not
exclude an **enumerated contract obligation list**, whose members are alike by construction: the 22
approved deltas (input 17), the migration artifact's six modes and three exit codes (input 20), the three
execution categories (input 21), the allow-listed report fields, seven masked material classes and
twenty-six `operation` codes (input 22), the three-line service allow-list and twelve collection-fixture
groups (input 23), the seven lifecycle steps, seven post-load invariants and twelve committed fixture
inputs (input 25), the eleven gating
`baselineSource` values and four pinned locale and collation values (input 26), and the thirteen blocking checks
of the deployed verification gate (input 28). Each of those
is a list of items of one kind, each item carrying comparable work, which is why they are admissible as
inputs while a finding count is not. Where a figure below is labelled "entry count", it is one of these
and the input row names what is being enumerated.

---

## 4. The effort model

### 4.1 The estimation basis: every input, with its method

Every figure in [section 5](#5-effort-by-workstream) derives from a row below. No other quantity is
used.

| # | Input | Value | Method | Source |
| --- | --- | --- | --- | --- |
| 1 | Migration source, total | **2,097** non-blank lines across 26 files | Sizing | [08 §4.1](08-technical-debt-register.md), [01 §2.4](01-architecture-overview.md) |
| 2 | Authentication rewrite | **382** non-blank lines, ~18% of input 1 | Sizing | [08 §4.2](08-technical-debt-register.md) |
| 3 | Seed data | **820** non-blank lines, ~39% of input 1 | Sizing | [08 §4.2](08-technical-debt-register.md) |
| 4 | Ordinary application code | **895** non-blank lines, ~43% of input 1 | Sizing | [08 §4.2](08-technical-debt-register.md) |
| 5 | Reference editions, not ported | MVC 4 **2,142**; MVC 3 **1,326** non-blank | Sizing | [08 §4.1](08-technical-debt-register.md) |
| 6 | Views to port | **29** Razor files, **5** naming legacy types | File count | [01 §2.5](01-architecture-overview.md), [05 §8.3](05-aspnet-core-migration-approach.md) |
| 7 | Static assets to relocate | **28** — the **27** files in the migration source's four asset groups **plus the web-application-root `favicon.ico`**, which the authoritative map relocates under `wwwroot` with them. The 27 is the four-asset-group figure and the 28 is what the relocation actually moves; this input is the second, because it is a relocation that is being sized | File count | [01 §2.3](01-architecture-overview.md), [04 §12.2](04-dotnet8-migration-strategy.md) |
| 8 | Bundling helper call sites | **11** — 10 `@Scripts.Render`, 1 `@Styles.Render` | Site count | [Appendix A.3](#a3-helper-view-and-site-counts) |
| 9 | Child actions to convert | **3** | Site count | [01 §5.3](01-architecture-overview.md) |
| 10 | Manual construction sites | **10** | Site count | [01 §5.4](01-architecture-overview.md) |
| 11 | Other Razor helper sites | **10** anti-forgery emissions, **5** partials, **4** `@Url.Content` | Site count | [Appendix A.3](#a3-helper-view-and-site-counts) |
| 12 | Blockers to resolve | **23** — **14** compile-time, and **9** that compiling does not find, of which **8** are silent and **1** is loud. The loud one is F-12-01, SQL Server Compact: the provider is a `providerName` string in configuration with no project `<Reference>`, so nothing binds to it at build and it stops at first data access instead | Entry count of work items | [12 §2.3, §4](12-migration-blockers.md) |
| 13 | Package outcomes to apply | **28** pins in the migration source | Pin count | [04 §8](04-dotnet8-migration-strategy.md) |
| 14 | Required **parity** test coverage | **75** rows; **30** of them against **two** fixtures, the platform surfaces target-only, and the remainder target-contract assertions. This is the count of the cross-baseline suite and **not** of the whole test workload — see the inclusion rule below and input 23. **This row is the only place the figure enters this document**: every other passage that needs it cites input 14 rather than repeating the number, because a figure repeated in eight places moves in some of them. **It is re-read from the owner's table rather than carried as a constant here**, because it has now moved **four** times — 103, then 108, then 116, then 115, the last of those being a withdrawal rather than an addition: `awk '/^### 12\.4 /,/^### 12\.5 /' docs/modernization/05-aspnet-core-migration-approach.md \| grep -cE '^\| [0-9]+ \|'` → 115, contiguous 1–115 (the same extraction through `sort -n \| uniq \| wc -l` also yields 115, so there is no duplicate and no gap) | Row count, command-verified | [05 §12.4](05-aspnet-core-migration-approach.md) |
| 15 | Tests existing today | **0** | Absence, command-verified | [08 §7.3](08-technical-debt-register.md), [A.2](#a2-the-absences-that-size-the-net-new-work) |
| 16 | Distinct captures for visual comparison | **28** capture states over **18** of the 29 Razor artifacts, **two** of which — `Home/About.cshtml` and `Home/Contact.cshtml` — can produce **no baseline at all**, times **2** viewports and **4** browser families, for **170** comparison pairs | Semantic classification of all 29 artifacts, plus a reachability census of the actions that render them, plus the two declared review dimensions — **not** a filename count | [A.4](#a4-the-visual-review-capture-set-input-16), [§7.1](#71-the-manual-visual-review) |
| 17 | Approved deltas requiring sign-off | **27**, across **5** approver constituencies — security, product, engineering, the data owner and operations — with **16** of the 27 naming more than one. **Its two dimensions are partitioned across two rows**: the 5 constituencies size [W1](#w1--approval-of-this-assessment); the 27 decisions size [§7.2](#72-the-approved-delta-sign-offs). Neither row counts the other's dimension | Entry count | [05 §11.5](05-aspnet-core-migration-approach.md) |
| 18 | Unvalidated state-changing POSTs | **5** in the migration source | Census | [08 §5.5](08-technical-debt-register.md) |
| 19 | Repository-hygiene volume | **14** database binaries at **43,376,640** bytes; **215** committed package files; **4** solutions for **3** projects | File count and byte count | [08 §6.2, §10.4, §10.2](08-technical-debt-register.md) |
| 20 | Source files carrying a legacy namespace directive | **19** of the migration source's **27** `.cs` files | File count | [05 §9.1](05-aspnet-core-migration-approach.md) |
| 21 | Security-event classes in the catalog — **13** produced by the ported application, **3** by tooling | **16** | Row count, split by [09 §6.8.1.1](09-security-assessment.md)'s producer map | [09 §6.8, §6.8.1.1](09-security-assessment.md) |
| 22 | Personal-data fields under governance | **9** on the order record, plus the identity link | Field count | [09 §3.11](09-security-assessment.md) |
| 23 | Required tests with **no** MVC 5 baseline, outside input 14 | **17** — **5** operator-host tests of the provisioning executable, and **12** CSP report-endpoint tests forming part of the promotion gate. Of the twelve, **eleven** are HTTP tests inside the automated suite and **one — test 12 — is discharged by a blocking manual deployed-browser gate**, because no HTTP client can observe policy enforcement by an agent, `report-to` precedence, or non-double-delivery. `9` and `14` are the stale forms | Test count | [04 §12.4](04-dotnet8-migration-strategy.md), [06 §10.2](06-azure-hosting-recommendations.md) |
| 24 | Assembly references that cannot resolve before a restore | **46** `<Reference>` elements in the migration source, of which **26** carry a `HintPath` under `..\packages\` and **20** resolve from the framework or machine-wide | Reference count | [02 §4.2](02-dependency-inventory.md) |
| 25 | Observability artifacts existing today | **0** — no logging abstraction, logging framework, `TraceSource`, ASP.NET health-monitoring configuration, health endpoint or metric, in any edition. The full census and its command are [08 §7.1](08-technical-debt-register.md)'s, whose pattern set includes `HealthCheck` and `healthMonitoring`; [A.2](#a2-the-absences-that-size-the-net-new-work) reproduces the logging and `healthMonitoring` portion of it here | Absence, command-verified | [08 §7.1](08-technical-debt-register.md), corroborated by [A.2](#a2-the-absences-that-size-the-net-new-work) |
| 26 | CI, deployment-automation and publish artifacts existing today | **0** — no pipeline definition, publish profile or container manifest | Absence, command-verified | [08 §7.2](08-technical-debt-register.md), [A.2](#a2-the-absences-that-size-the-net-new-work) |
| 27 | Application configuration files, and the runtime state they declare | **15** application `.config` files, of which **0** declare `<sessionState>` and **0** declare `<machineKey>` — so session storage and key material are both framework defaults | File count and census | [01 §6.6](01-architecture-overview.md) |
| 28 | Stores and entities the production load must reconcile | **2** stores — the catalog store and the Identity store, coupled only by convention with no foreign key between them — over **6** catalog entities | Store and entity count | [01 §6.1, §6.3, §6.5](01-architecture-overview.md) |
| 29 | Tracked documents describing the workflow the target replaces | **3** — the root README and the two per-edition READMEs | File count, command-verified | [A.3](#a3-helper-view-and-site-counts) |
| 30 | Behaviour the provisioning tool must implement, outside its added tests | **5** required properties; **4** independently converged operations on a provisioning run — role, user, credential, membership — and **2** on a revoke across **3** branches; and `PROV-6001`'s closed outcome vocabulary of **17** values, being **11** non-failure and **6** failure. The 11 partition across the four operations as `2 + 2 + 3 + 4` | Property, operation and outcome count | [05 §10.2](05-aspnet-core-migration-approach.md) (properties and operations), [09 §6.8.1](09-security-assessment.md) (the outcome vocabulary, which that section owns) |
| 31 | Path literals the **repository-wide** casing audit must examine | Inputs 7, 8 and 11 are **migration-source only**, and the audit is not. Repository-wide, all three editions: **173** browser-served static files (**171** in the four asset groups plus the two web-application-root `favicon.ico` files); **83** Razor views; **11** bundle definitions across the two `BundleConfig.cs` files, carrying **36** `~/`-rooted literals between them; **31** `@Url.Content` occurrences — 4 in MVC 5, 4 in MVC 4 and **23** in MVC 3, which is where the concentration is; and **21** `@Scripts.Render`/`@Styles.Render` sites, of which 11 are MVC 5's. MVC 3 has no `App_Start` folder and therefore no bundle definitions at all. **The two totals this input yields are 258 containers — `173 + 83 + 2`, the two `BundleConfig.cs` files being neither views nor served assets — holding 88 literal sites, `36 + 31 + 21`**; [A.3](#a3-helper-view-and-site-counts) reproduces every figure and both per-edition partitions | Site and file count, command-verified | [A.3](#a3-helper-view-and-site-counts), [01 §2.3, §2.5](01-architecture-overview.md), [06 §3.4](06-azure-hosting-recommendations.md) |


**Inputs 2, 3 and 4 are a partition of input 1**, and the arithmetic is worth stating because it is the
backbone of the whole model: **382 + 820 + 895 = 2,097** non-blank lines, all four figures the sizing
metric. Everything other than the authentication rewrite is therefore **1,715** non-blank lines — a
figure this document uses only as a cross-check, because [08 §4.2](08-technical-debt-register.md)
splits that remainder in two on purpose. Treating 1,715 as one homogeneous block of porting work would
re-introduce exactly the overestimate the next note describes.

**Five notes on how these inputs are used, each of which changes an estimate materially.**

- **Input 3 is not porting work.** [08 §4.2](08-technical-debt-register.md) directs that the seed be
  estimated as *a data-handling decision, not as 820 lines of porting*. Treating 39 percent of the
  migration source as line-by-line work would be the single largest overestimate available in this
  document. It is sized in [W7](#w7--the-aspnet-core-port) as a decision plus a data path.
- **Input 5 is excluded from the port estimate entirely.** Neither reference edition is ported
  ([03 §5](03-modernization-roadmap.md) scopes the port to the migration source). Their volume appears
  here only so that a reader cannot mistake the 5,565-line repository total for the work envelope.
  Per [08 §3.4](08-technical-debt-register.md), MVC 3 must never be sized by analogy with the other
  two — see [R10](#r10--scoping-by-analogy-across-editions).
- **Input 19 is not migration effort.** It is a repository-hygiene decision, sized separately in
  [section 7.3](#73-the-manual-accessibility-review) and gating nothing.
- **Inputs 14 and 23 are two different things, and the rule separating them is this document's to state,
  because this document is where the test workload is costed.**
  [05 §12.4](05-aspnet-core-migration-approach.md)'s numbered table is the cross-baseline **parity**
  suite: every row of it proves a behaviour against the MVC 5 baseline, either by asserting equivalence
  with what the baseline does or by asserting the target contract that deliberately replaces a baseline
  defect, with the legacy fixture recording the old shape so the difference is auditable. **A required
  test with no MVC 5 baseline is therefore not a row of that table**, is not counted in input 14, and is
  **not folded into [W4](#w4--build-governance-bootstrap-pre-port-behavioural-baseline-and-test-suite)** — because W4 is the row
  that builds the two-fixture parity suite, and a test with only one runtime to run against needs
  neither of its fixtures nor its normalization harness.
  **Each such test is estimated in the workstream that builds the thing it tests**, which is also the
  workstream whose gate demonstrates it:

  | Non-parity test set | Required by | Estimated and gated in |
  | --- | --- | --- |
  | **5** operator-host tests — a hostile working directory, every password-bearing argument form refused, the repair path in that host, the dispatcher's admitted command lines, and the credential arriving on its named environment channel without appearing in captured output. A sixth assertion in the same set, the lifetime spelling, is discharged by the Release solution build and costs nothing here | [04 §12.4](04-dotnet8-migration-strategy.md) | **[W12](#w12--administrator-provisioning-tool)**, which builds `tools/provision-admin` |
  | **11** HTTP CSP report-endpoint tests — both report transports, the two rejected media types, the unparseable and member-absent bodies, the size and batch bounds, the rate-limit partition, the anti-forgery exemption with its paired `GET`, and the redaction of query string, sample and referrer | [06 §10.2](06-azure-hosting-recommendations.md) | **[W7](#w7--the-aspnet-core-port)**, which owns the header set and the report endpoint registered in the composition root |
  | **1** deployed-browser CSP test — the twelfth of [06 §10.2](06-azure-hosting-recommendations.md)'s twelve, executed by the blocking gate [`G-CSP-BROWSER`](06-azure-hosting-recommendations.md#g-csp-browser) against a **deployed** environment on **every** browser engine of [06 §10.4](06-azure-hosting-recommendations.md)'s matrix, and run twice — once during the report-only period and again on any directive, binding, route or group change | [06 §10.4](06-azure-hosting-recommendations.md) | **[W10](#w10--hosting-provisioning-and-platform-configuration)**, as its exit condition 9, because it needs a provisioned deployment and a real browser rather than a test host |

  [03 §4.3](03-modernization-roadmap.md) states the same rule from the roadmap's side and places the three
  sets at the same three gates; no document folds any of them into W4, and none alters input 14's
  count by doing so. **The consequence for this model is that the test workload is input 14's parity rows
  plus seventeen further tests — `75 + 17 = 92` executable scenarios, not 75** — and the seventeen are
  real work that an earlier reading of input 14 left uncosted. **The parity term of that sum is input 14's
  and is re-read from [05 §12.4](05-aspnet-core-migration-approach.md) at each reconciliation rather than
  carried as a constant**, because it has moved four times: `103 + 17 = 120`, `108 + 17 = 125`,
  `116 + 17 = 133` and `115 + 17 = 132` are all superseded forms of this sentence, and the **17** is the term that has not
  changed — five operator-host tests plus twelve CSP tests. **The most recent move is why the separation
  these two inputs express has to be enforced in both directions.** The row
  [05 §12.4](05-aspnet-core-migration-approach.md) briefly carried as 116 was a *pointer* row, whose content
  was "the eleven HTTP-observable tests of [06 §10.2](06-azure-hosting-recommendations.md)" — the same
  eleven that already sit inside input 23's seventeen. Counting it as a parity row therefore counted those
  eleven twice, in the parity term and in the non-parity term of one sum, and contradicted this very rule: a
  test with no MVC 5 baseline is not a row of that table. 05 withdrew the row, keeping its cross-reference as
  prose, so the eleven are counted **once**, in input 23, and the **17** does not move. The twelve CSP tests
  split **eleven / one** across two workstreams rather than sitting together, which is why they appear as two
  rows here: the eleven are HTTP tests over code W7 writes, and the twelfth is a browser observation of a
  deployed policy, which no test host can make.
- **Inputs 24 to 30 exist so that every row of [section 5.1](#51-summary-table) carries a numbered,
  method-named input that sizes its own work. They are seven inputs closing gaps on eight rows, and the
  two counts are different because one input closes two rows.** The eight rows fall into three kinds of
  gap, and the distinction that matters is between a row that **named** no input and a row that named one
  which did not **size** it.
  - **Six rows named no count at all** — **W2, W10, W13, W14, W15 and W16**. Each stated its basis in
    prose and cited the owning deliverable, but none named a quantity, so none could be traced to one.
  - **One row named an input that sized only part of it** — **W12**. It cited input 23, but input 23
    counts the **5 added operator-host tests** of the provisioning executable, not the executable. The
    tool's own base effort — its properties, its operations and its outcome vocabulary — had no count,
    which **input 30** supplies; input 23 stays beside it for the test half.
  - **One row named an input that is a constraint on it rather than its size** — **W11**. It cited input
    15, zero tests, which really does constrain its pipeline's test stage but is not what sizes the row.
    The row is sized by the pipeline artifacts that do not exist, which is **input 26** — the same input
    that closes W10's gap, which is why seven inputs cover eight rows. Input 15 is retained beside it for
    the narrower purpose its basis states.

  The seven inputs are of **four** kinds, and they partition as `2 + 2 + 2 + 1`. **Two are censuses of
  what exists**: input 24's 46 references with their 26 hint paths
  ([02 §4.2](02-dependency-inventory.md)) and input 27's 15 configuration files with their zero
  `<sessionState>` and zero `<machineKey>` declarations ([01 §6.6](01-architecture-overview.md)).
  **Two are absences**, command-verified in
  [A.2](#a2-the-absences-that-size-the-net-new-work) and therefore sizing net-new work in the sense
  [§6.2](#62-the-finding-that-matters-most-in-this-document) defines: input 25's zero observability
  artifacts and input 26's zero pipeline, publish and container artifacts. **Two are counts of the objects
  a step must operate over**: input 28's two stores and six catalog entities, and input 29's three
  superseded documents. **One is a count of behaviour a component must implement** — input 30's five
  required properties, its four converged operations plus the revoke mode's two across three branches, and
  `PROV-6001`'s 17 closed outcome values. **Not one of the seven is a line count**, so none can be
  confused with either method in
  [§3.1](#31-the-rule-and-why-it-is-the-sharpest-constraint-on-this-document) — they are file, reference,
  store, entity, absence, property, operation and outcome counts, labelled as such in the table above,
  exactly as [§3.3](#33-a-third-kind-of-count-named-so-it-is-not-mistaken-for-either) requires.

  **Input 31 is a different kind of addition and is not part of that census.** It closes no *naming* gap —
  [W5](#w5--repository-wide-path-casing-audit) already cited inputs 7, 8 and 11 — it corrects their
  **scope**. All three count the migration source only, and the casing audit is repository-wide across all
  three editions, so W5 was being sized against **4 of the 31** `@Url.Content` occurrences it must actually
  examine — an eighth of that dimension. Input 31 supplies the repository-wide census: **258 containers
  holding 88 literal sites**, partitioned by edition in
  [A.3](#a3-helper-view-and-site-counts). **It changes W5's basis and what its band is for without moving
  the band**, because [03 §5 W5](03-modernization-roadmap.md) narrowed that row's exit in the same round by
  removing a deployment from it; W5's basis derives the offset, and
  [§6.1.1](#611-the-walk-from-the-previously-published-total) counts W5 among the rows that did not
  move, its band re-derived from this census rather than carried.

  **Six of the seven change no band; one does, and it is named rather than absorbed.** Inputs 24 to 29
  each document the basis of a figure that was already judged, so they leave the total, the concurrency
  sets and the critical path numerically untouched. **Input 30 is the exception.** Its first statement
  undercounted `PROV-6001`'s closed vocabulary as eight values against
  [09 §6.8.1](09-security-assessment.md)'s 17, and correcting it exposed one value —
  `Failed_ArgumentRejected` — that carries genuinely new scope rather than a renaming of work already
  priced. [W12](#w12--administrator-provisioning-tool)'s band moved from 3 / 5 / 9.5 to 3 / 5.5 / 10.5 for
  that reason, with the derivation and the explicit statement of what does *not* contribute stated in its
  basis. **That figure is the state of the row at the end of the input-30 pass and not its current band.**
  Two later corrections moved it again — [§4.2.1](#421-the-rounding-rule-stated-once) re-derived input 23's
  operator-host increment upward, and [03 §5 W12](03-modernization-roadmap.md) added exit condition 7's
  deployed census and the fourth separately gated invocation — so the row now stands at
  **3.5 / 7.5 / 12.5** in [§5.1](#51-summary-table), and
  [§6.1.1](#611-the-walk-from-the-previously-published-total) records that second move as one line of the
  walk.

### 4.2 The unit, defined

**All figures are ideal engineer-days (IED).** One IED is one engineer working one focused day on the
task, with:

- **no calendar meaning** — an IED is a measure of *work content*, not of elapsed time. Converting IED
  to elapsed time requires a team size, a utilization factor and a parallelism assumption, all three
  of which belong to the reader's organization and none of which this document supplies;
- **no review, approval-waiting or meeting overhead**, except in the rows where obtaining a decision
  *is* the work ([W1](#w1--approval-of-this-assessment), [W11](#w11--ci-provider-selection-then-pipeline-authoring)'s
  provider gate, and [section 7.2](#72-the-approved-delta-sign-offs));
- **one engineer competent in the target stack**, per assumption A1 below.

Bands are **low / expected / high**, and no row carries a single-point estimate. The bands are not
symmetric: a task whose difficulty depends on a fact not yet established has a long right tail, which
is the honest shape for this project and is why the totals in
[section 6](#6-totals-and-where-the-effort-actually-lives) span more than a factor of three.

#### 4.2.1 The rounding rule, stated once

Several bands below are derived by multiplying a counted number of elements by a per-element rate, which
produces figures at a finer granularity than this document's figures carry. **Every figure in this
document is expressed at half-IED granularity, and the rule that gets it there is:**

```text
rounded(x) = ceil(2x) / 2
```

In words: **round up to the next half-IED.** A value already sitting on a half is unchanged; every other
positive value moves up. Three properties of the rule matter, and they are the reason it is stated here
once rather than restated per derivation:

- **It is applied per column, independently.** Low, expected and high are each rounded on their own. It
  is not applied to a total — totals are the sum of already-rounded rows, so no total is rounded twice.
- **It is directional, not nearest.** Rounding to nearest would narrow a band whenever the expected
  figure fell just below a half while the low figure fell just above one, which is a reduction in stated
  effort produced by arithmetic rather than by evidence. Rounding up cannot narrow a band and cannot
  lower a figure.
- **It admits no exception.** A derivation that rounded 2.1 down to 2 because 2 "reads better" would
  give this document two rounding conventions and therefore no reproducible total. Every derivation
  below that rounds cites this rule, so a reader can recompute any of them from the stated
  multiplication and this one line.

The rule is a **presentation** contract, not an estimating one: it changes a figure by at most 0.5 IED
per row, and it never changes which count or which rate a row is derived from.

### 4.3 Assumptions, stated rather than implied

Every assumption below is one whose falsification would move a band. They are listed so an approver
can correct one rather than discover it later.

| ID | Assumption | If it is false |
| --- | --- | --- |
| **A1** | The team is **competent in the target stack** — ASP.NET Core, EF Core migrations and the target cloud platform — without needing to learn it during the work | Add learning effort across W6, W7 and W10. This is the assumption with the widest leverage over the total |
| **A2** | A **build host meeting the prescribed toolchain** remains available, as the one used for the outcome recorded in [10 §3.1](10-build-and-deployment-requirements.md) was | W2 grows from a reproduction into a provisioning exercise |
| **A3** | The team has **read access to a production-representative dataset** for rehearsing both data migrations, with PII handling approved | W8 and W9 high bands become the expected case; reconciliation cannot be rehearsed and moves risk into the cutover window |
| **A4** | The **manual visual review is performed by one reviewer** against the captured baseline, with a second signature only for approval | A review panel multiplies [section 7.1](#71-the-manual-visual-review) by the number of reviewers |
| **A5** | **Approvers are identified and available.** [05 §11.5](05-aspnet-core-migration-approach.md) names approval *roles*; A5 assumes named people hold them | W1 and [section 7.2](#72-the-approved-delta-sign-offs) grow, and every dependent workstream inherits the delay |
| **A6** | The **single-cutover approach stands** as decided in [05 §11](05-aspnet-core-migration-approach.md) | The conditional incremental path of [05 §11.6](05-aspnet-core-migration-approach.md) is a materially different and larger shape: two hosts, two pipelines and an adapter surface on both sides. **This model does not cost it** |
| **A7** | **Scope is the migration source only.** Neither reference edition is ported | Sizing a second edition is new work, not a multiplier — see [R10](#r10--scoping-by-analogy-across-editions) |
| **A8** | The **target platform's managed services are used as recommended** in [06](06-azure-hosting-recommendations.md), rather than self-hosted equivalents | W10 grows substantially; a self-managed database and key store are not costed here |
| **A9** | **No new functional requirement** is introduced. The behaviour baseline is [05 §11.5](05-aspnet-core-migration-approach.md)'s: preserved outcomes plus **22** approved deltas, two of them conditional | Any new feature is outside this model entirely. A **twenty-third** delta would mean the authoritative behaviour set had moved again, which is a re-estimation of [W1](#w1--approval-of-this-assessment), [§7.2](#72-the-approved-delta-sign-offs) and the coverage rows it implies rather than a variance inside their bands |
| **A10** | The **password hashes in the shipped credential store validate** under the target framework's default hasher | W8's high band becomes the expected case and [R5](#r5--identity-migration-rollback) escalates from a rollback risk to a redesign |
| **A11** | The **interim hosting branch is not taken.** It is a conditional branch outside the sixteen workstreams [03 §4](03-modernization-roadmap.md), and [06 §5.8](06-azure-hosting-recommendations.md) records that its production data move currently has a shape rather than a contract, so the branch is not executable until that contract is authored and approved | **Nothing in this model covers it.** If the branch is taken, two units enter the estimate that are absent from it today: **authoring the legacy-schema-preserving migration contract** 06 §5.8 enumerates, and **executing it**. Both must be estimated at that point on the same basis as every row here, and [R15](#r17--the-interim-hosting-paths-stored-credential-and-a-time-box-with-no-box)'s enforcement obligation begins with the grant rather than with the move |

### 4.4 Confidence, and its reason

**Overall confidence: medium.** Stated with its reason, because a bare adjective is not a confidence
statement:

> **Medium, because the two halves of the work have opposite estimation properties.** The mechanical
> port is **well bounded by enumeration**: the migration source is 2,097 non-blank lines (sizing
> metric), its decomposition is measured to the file [08 §4.2](08-technical-debt-register.md), all 22
> blockers are enumerated with a named resolution each [12 §2.3](12-migration-blockers.md), and every
> construct the port must touch is named in a document rather than inferred. Against that, **two inputs
> this model depends on have never been measured, and neither can be narrowed by reading source.** The
> first is the build: [10 §1.2](10-build-and-deployment-requirements.md) carries the migration source's
> build assessment as *blocked pending a Windows verification run*, and what is established is a
> precondition failure on a clean checkout rather than a compile result — whether the application
> compiles once restored is not established, so W6's and W7's diagnostic volume is enumerated from
> [12 §2.3](12-migration-blockers.md) rather than observed from a compiler. The second is the schema:
> **both data migrations are estimated against a schema that has not been extracted** — the
> authoritative definition exists only inside a committed binary, the migration source ships no schema
> script, and [12 §5](12-migration-blockers.md) explicitly qualifies the Identity column set as
> *evidence rather than proof*. **W2 retires the first and W3 retires the second**, which is why
> [03](03-modernization-roadmap.md) places both before the port.

Confidence is therefore **not uniform**, and the per-workstream column in
[section 5](#5-effort-by-workstream) states it per row:

| Confidence | Workstreams | Why |
| --- | --- | --- |
| **High** | W5, W6, W14, W15 | Fully enumerated, small, and verifiable on completion |
| **Medium** | W1, W2, W7, W10, W11, W12, W13, and [§7.1](#71-the-manual-visual-review) | Scope is known and enumerated; the variance is execution and, for W1 and W13, other people's decisions. W2 is medium rather than high for a different reason: its *tasks* are enumerated but its *outcome* is not known, and a failing verification run costs more than a passing one |
| **Low** | W3, W4, W8, W9 | Each depends on a fact not yet established: the extracted schema for three of them, and for W4 the behaviour of a system that has never been characterized by a test — plus, across its four re-derivations, seven components with no precedent in this repository to calibrate against: a two-platform execution whose handoff record is refused on any mismatch, a private legacy deployment lifecycle whose readiness poll must also wait out an `async void` provisioning path, a keyed pseudonym scheme with key custody and destruction, a redactor that must itself be tested, an abstract-plus-sealed contract topology that has to enrol tests in two assemblies through per-assembly collection definitions, a runtime-neutral store observer with two implementations beside a separate OWNER-backed setup surface, and a three-identity SQL bootstrap with a durable ownership registry (inputs 21, 22, 24, 25, 27) |
| **High** (conditional) | W15·C | A platform setting plus one defined verification, both owned by [06 §8.3, §8.3.1](06-azure-hosting-recommendations.md); it is conditional rather than uncertain |

**The low-confidence rows carry 86 of the 258 expected IED** — see
[section 6](#6-totals-and-where-the-effort-actually-lives) — which is the single most useful thing an
approver can know about this estimate. W3 is cheap and retires most of that uncertainty, which is why
[03](03-modernization-roadmap.md) places it before the port and why
[section 8.3](#83-the-critical-path-and-what-to-do-first-if-the-goal-is-to-narrow-the-estimate) recommends it as the first
substantive action.

**No confidence rating changed across any of the four re-derivations, and that is a claim
worth defending rather than an omission.** W4 stays **Low**, W6 stays **High**, W7, W11, W12 and the
manual visual review stay **Medium**, and overall
confidence stays
**Medium** — because the two facts the overall statement rests on, the unverified build and the
unextracted schema, are untouched by a coverage figure being superseded. What the supersessions changed is
**scope, which was enumerated more tightly rather than less**: a case-level matrix with named assertion
fields is a firmer basis than a surface list, and a fixture whose lifecycle, dataset, principals and
handoff contract are specified item by item is a firmer basis than one described as "a fixture", so the
re-derived bands are better founded than the ones
they replace even though they are larger.

**The low-confidence share moved again, and it is stated rather than smoothed.** It is **86 of
258** — 4 + 66 + 8 + 8 for W3, W4, W8 and W9 — about **33 percent** of the model, against 80.5 of 239.5
(about 34 percent), then 57 of 195.5 (about 29 percent), then 39 of 155 and
33 of 127.5. **It fell fractionally this round**, and the reason is arithmetic rather than a change of
view: **W4 absorbed 5.5 of the 18.5 expected IED this re-derivation added** while W7, W11 and W12 — all
Medium — absorbed the other 13, so for the first time across four re-derivations the growth landed mostly
outside the Low rows. The
remedy is unchanged and is stated in the next paragraph: W3 retires the schema uncertainty behind three of
the four, and only W4's is retired by executing W4 itself.

---

## 5. Effort by workstream

**The decomposition below is [03 §5](03-modernization-roadmap.md)'s, not this document's.** All sixteen
workstreams appear, in 03's order, under 03's names. No workstream is added, renamed, merged or split,
because a second decomposition would leave the roadmap and the estimate unable to reference each other.
What this document adds to each is a band, a confidence and the inputs the band derives from.

> **These bands are estimated against 03's corrected graph, and four of them changed because of it.**
> The corrections are 03's, not this document's, and each one moves effort rather than inventing it:
>
> - **W6 is a skeleton, not a converted legacy build.** [03 §5 W6](03-modernization-roadmap.md) carries no legacy-behaviour claim and does not run the W4 suite, because the unchanged `System.Web` source cannot compile on the target framework at all. The integration cost that gate implied moves into W7, where it is a named sub-row.
> - **W8 and W9 end at a rehearsal against a copy**; the production extraction, load and reconciliation are W13's. So W8 and W9 lose an execution and gain a rehearsal and a runbook, and W13 gains the execution.
> - **W11 gained a rehearsed release-migration stage** and is now a prerequisite of W10's DDL-applying steps, so it sits earlier in the sequence than its number suggests.
> - **W16 is new** — the personal-data governance workstream — and is sized here for the first time.
>
> Nothing above changes a counting method, so every band still traces to the same numbered inputs in [section 4.1](#41-the-estimation-basis-every-input-with-its-method).
>
> **One further dependency correction changes a row's position without changing its size.** The suite W4
> authors is a **project inside** [04 §12](04-dotnet8-migration-strategy.md)'s target project graph, so it
> cannot be compiled before W6 creates that graph: [03 §4.2](03-modernization-roadmap.md) therefore carries
> the edge **`W6 → W4`**. W6's band is unchanged — an empty compiling test project is all the edge requires
> of it — and W4's band is unchanged, because the edge changes where the work sits, not what it contains.
> What does change is the sequence: [§8.2](#82-concurrency-permitted-by-the-graph) and
> [§8.3](#83-the-critical-path-and-what-to-do-first-if-the-goal-is-to-narrow-the-estimate) are re-derived
> against it, and W6's 3 expected IED returns to the critical path.
>
> **Two further rows moved because an input was corrected rather than because the graph changed.**
> [05 §12.4](05-aspnet-core-migration-approach.md)'s required coverage is **input 14's 75 rows**, not the
> 27 an earlier reading of input 14 recorded, so **W4** is rebanded; and
> [05 §11.5](05-aspnet-core-migration-approach.md) enumerates **27** approved deltas, not the 18 an
> earlier reading of input 17 recorded and not the 14 before that, so the
> **sign-off** row of [§7.2](#72-the-approved-delta-sign-offs) is rebanded again. Both are counts owned by
> 05 and cited here; each row states the increment and its derivation.
>
> **And three rows moved because an input was *missing* rather than wrong.** The model previously treated
> input 14's parity rows as the whole test workload. They are not: [04 §12.4](04-dotnet8-migration-strategy.md)
> requires **5** operator-host tests and [06 §10.2](06-azure-hosting-recommendations.md) **12** CSP
> tests, none of which has an MVC 5 baseline and none of which is a row of 05's table.
> [§4.1](#41-the-estimation-basis-every-input-with-its-method)'s inclusion rule adds them as **input 23**
> and places them where the thing they test is built — the five in **W12**, the **eleven** HTTP CSP tests in
> **W7**, and the **twelfth**, the deployed-browser gate, in **W10** — so all three of
> those rows are rebanded. Nothing is added to W4 and input 14's count is untouched, which is the whole
> purpose of separating the two inputs.

### 5.1 Summary table

Ideal engineer-days ([section 4.2](#42-the-unit-defined)). Every figure traces to a numbered input in
[section 4.1](#41-the-estimation-basis-every-input-with-its-method).

| Workstream ([03 §5](03-modernization-roadmap.md)) | Low | Expected | High | Confidence | Primary inputs |
| --- | ---: | ---: | ---: | --- | --- |
| **W1** — Approval of this assessment | 3 | 6 | 12 | Medium | 17 |
| **W2** — MVC 5 build reproduction and the restore precondition | 2 | 4 | 9 | Medium | — (scope from [03 §5](03-modernization-roadmap.md); exit criterion from [10 §3.2](10-build-and-deployment-requirements.md)) |
| **W3** — Authoritative schema extraction | 2 | 4 | 8 | **Low** | 1, 12 |
| **W4** — Build-governance bootstrap, pre-port behavioural baseline and test suite | 37.5 | 66 | 111.5 | **Low** | 14, 15, 17, 21, 22, 23, 24, 25, 26, 27 |
| **W5** — Repository-wide path-casing audit | 1 | 2 | 4 | High | 7, 8, 11 |
| **W6** — Project-format conversion and dependency transition | 2.5 | 4.5 | 8.5 | High | 12, 13, 27 |
| **W7** — The ASP.NET Core port | 61.5 | 109.5 | 188 | Medium | 2, 3, 4, 6, 8, 9, 10, 11, 12, 14, 17, 20, 23, 24, 25, 29 |
| **W8** — Identity migration tooling, rehearsed against a copy | 4 | 8 | 16 | **Low** | 12 |
| **W9** — Domain data migration tooling, rehearsed against a copy | 4 | 8 | 15 | **Low** | 1, 12 |
| **W10** — Hosting provisioning and platform configuration | 5 | 9 | 16 | Medium | — (scope from [06 §6](06-azure-hosting-recommendations.md)) |
| **W11** — CI provider selection, then pipeline authoring | 11 | 19.5 | 35.5 | Medium | 15, 21, 26, 28, 29 |
| **W12** — Administrator provisioning tool | 2.5 | 6 | 10 | Medium | 27 (the operator-tool pin); scope from [05 §10.2](05-aspnet-core-migration-approach.md); row 75's acceptance from [05 §12.4](05-aspnet-core-migration-approach.md) |
| **W13** — Cutover | 2 | 4 | 8 | Medium | — (scope from [06 §11](06-azure-hosting-recommendations.md)) |
| **W14** — Documentation revision | 1 | 2 | 3 | High | — |
| **W15** — Affinity retirement | 0.5 | 1 | 2 | High | — |
| *Non-code:* manual visual review ([§7.1](#71-the-manual-visual-review)) | 2.5 | 4.5 | 7.5 | Medium | 16 |
| *Component of W1, **not added**:* approved-delta sign-offs ([§7.2](#72-the-approved-delta-sign-offs)) | *(2)* | *(4)* | *(8)* | Medium | 17 — its **27-decision** dimension only |
| **W16** — Personal-data governance | 3 | 6 | 12 | **Low** | 22, 21 — requirement from [09 §3.11, §6.8](09-security-assessment.md) |
| *Conditional, **not** in the total:* pre-admission affinity retirement (secondary hosting path) | 1 | 2 | 4 | High | — (scope from [06 §8.3, §8.3.1](06-azure-hosting-recommendations.md)) |
| *Non-code:* manual accessibility review ([§7.3](#73-the-manual-accessibility-review)) | 2.5 | 4.5 | 8 | Medium | 6, 16 |
| **Total** — the sixteen workstreams plus the two manual reviews | **147.5** | **268.5** | **474** | Medium overall | |

**Two conventions make this column addable, and both are stated in the table itself rather than in a
footnote.**

- **Parenthesised figures are already inside the row above them and are excluded from the sum.** The
  approved-delta sign-offs are **effort inside W1** — W1's 3 / 6 / 12 contains them, as
  [section 5.2](#w1--approval-of-this-assessment) itemizes — so they appear parenthesised for visibility
  and are **charged exactly once**. Adding them again as a separate line item would double-count 4
  expected IED.
- **The manual visual review is not parenthesised, because it is not inside any workstream's band.** It
  is a non-code task in [03](03-modernization-roadmap.md)'s terms, sized only here, and it is added to
  the total. [Section 7](#7-work-that-is-not-code) states where each of the two non-code items attaches
  in the sequence.

Sum of the sixteen workstream rows: **142.5 / 259.5 / 458.5**. Plus the manual visual review's
2.5 / 4.5 / 7.5 and the manual accessibility review's 2.5 / 4.5 / 8: **147.5 / 268.5 / 474**.
The sixteen-row addition, at the expected band:
6 + 4 + 4 + 66 + 2 + 4.5 + 109.5 + 8 + 8 + 9 + 19.5 + 6 + 4 + 2 + 1 + 6 = **259.5**; at the low band
3 + 2 + 2 + 37.5 + 1 + 2.5 + 61.5 + 4 + 4 + 5 + 11 + 2.5 + 2 + 1 + 0.5 + 3 = **142.5**; at the high band
12 + 9 + 8 + 111.5 + 4 + 8.5 + 188 + 16 + 15 + 16 + 35.5 + 10 + 8 + 3 + 2 + 12 = **458.5**.

**Five rows have now been re-derived a fourth time**, because
[05 §11.5, §12.3–§12.11](05-aspnet-core-migration-approach.md),
[04 §6, §7.1, §7.6, §7.7 and §12.2–§12.4](04-dotnet8-migration-strategy.md),
[06 §6.4, §9.5 and §12.1](06-azure-hosting-recommendations.md) and
[03 §4.2 and §5](03-modernization-roadmap.md) closed a further round of findings that again enlarged the
scope the previous bands were derived against. **W4, W7, W11 and W12 move, and nothing else does**;
[section 5.2](#52-basis-of-estimate-per-workstream) shows each re-derivation with its components and its
addition. **W4** moves from 35.5 / 60.5 / 102.5 to **37.5 / 66 / 111.5** — **+2 / +5.5 / +9**; **W7** from
57.5 / 102 / 175.5 to **61.5 / 109.5 / 188** — **+4 / +7.5 / +12.5**; **W11** from 9 / 15.5 / 29 to
**11 / 19.5 / 35.5** — **+2 / +4 / +6.5**; and **W12** from 2 / 4.5 / 8 to **2.5 / 6 / 10** —
**+0.5 / +1.5 / +2**. The four deltas sum to **+8.5 / +18.5 / +30**:
2 + 4 + 2 + 0.5 = 8.5, 5.5 + 7.5 + 4 + 1.5 = 18.5, and 9 + 12.5 + 6.5 + 2 = 30 — which is the whole of
the movement between the previous total of 133.5 / 239.5 / 424 and the one above:
133.5 + 8.5 = **142**, 239.5 + 18.5 = **258**, 424 + 30 = **454**. Excluding the review, the
fifteen-workstream total moves from 131 / 235 / 416.5 by the same deltas:
131 + 8.5 = **139.5**, 235 + 18.5 = **253.5**, 416.5 + 30 = **446.5**.

**What arrived this round, item by item**, all of it specified in the siblings and cited rather than
restated:

- the published suite kept **75 rows** and moved from 239 cases to **242**, and from 383 fixture
  executions to **386** — the growth being cases inside two existing rows, all target-only (input 14);
- the test topology became **declarable**: every base, concrete and fixture type `public`, a
  `[CollectionDefinition]` class **per surface group per assembly** with a `const`-backed name, and the
  runner and build-asset packages **declared directly in every runnable test project** (inputs 23, 27);
- an **`IStoreSetup`** write API — eight operations across eleven members, OWNER-backed and deliberately
  off the injected context — plus **five** additional `IStoreObserver` read projections (input 27);
- the fixture inputs became **twelve committed files**, nine of them `ModelOverrides/` divergence seams
  (input 25);
- the baseline record gained a **`coverage` completeness object** and **one publication policy** (input 26);
- the in-process host's instrumentation became an **exhaustive three-line allow-list** covering **both**
  `DbContext` types, fault injection became **`DENY`-based**, and the browser case acquired its own
  **real-Kestrel loopback host** (inputs 23, 29);
- the migration artifact gained the **`--scope` grammar** and a **`dbo.MigrationLoadJournal`** written in
  each group's own transaction (input 20);
- the deployed gate gained a **published telemetry join protocol** and an **executor-and-stage mapping**
  for its thirteen checks (input 28);
- the provisioning tool's audit record gained a **destination** — one operator-tool pin, a federated
  credential path and a retention export — and **row 75 became its exit criterion** (input 27);
- and the pin table reached **17** rows: ten on the 8.0.30 band, six test-tooling, one operator-tool
  (input 27).

**The three earlier re-derivations remain on the record**, because the movement is only checkable against
what it moved from. The round before this one moved six rows, for these reasons:

- the published suite moved from 72 rows and 183 cases to **75 rows and 239 cases**, and from 281
  fixture executions to **383** (input 14);
- the approved-delta ledger moved from 19 entries to **22** (input 17);
- the deployed verification gate's attempt budget acquired **literal evidence bounds** in place of a
  stated intent, which converts design work into authoring work (input 28);
- the contract topology became **six abstract classes with sealed per-assembly bindings**, a
  runtime-neutral **`IStoreObserver`** with two implementations and an explicit DTO surface, and **six**
  independent test-tooling pins in place of four (inputs 23, 27);
- the legacy lifecycle gained **startup-quiescence polling** and the fixture dataset grew to published
  per-entity counts with **seven** post-load invariants (input 25);
- provenance resolved into **three** distinct things — an **11-value gating `baselineSource`** namespace, a
  **7-fact `targetRun`** namespace recorded and never compared, and a **committed `baseline-reference.json`**
  of three legacy values — with an artifact transfer, a digest sidecar and **one publication policy** that
  publishes a `baseline-capture-diagnostic` on a failed or incomplete capture and a `baseline-record` only
  on a complete all-passed run (input 26); diagnostics gained a sanitized exception projection, a **26-code** `operation` set
  and a corrected keyed-alias scheme (input 22); and destructive-operation control gained a **third SQL
  identity**, a durable ownership registry with a sidecar, identifier re-resolution before every `DROP`,
  and a standalone sweep class with an always-run cleanup job (input 24);
- **the browser-executed half of the scripted cart flow is no longer excluded — though its coverage
  decision is still open.** An earlier round carried it as an open scope-change request and stated that no
  band contained any part of it. 05 §12.11 **pins** the harness for exactly one flow on one engine
  (input 29), so it is in scope and is estimated here. What that did **not** settle is which engines get a
  functional assertion: **Gecko and WebKit get none**, and
  [R16](#r18--the-browser-executed-half-of-the-scripted-cart-flow-one-pinned-engine-and-an-open-decision-on-the-other-two)
  carries that residual as one of [section 9.4](#94-the-five-risks-that-are-approval-decisions-not-mitigations)'s
  approval decisions. Pinning the harness also forced the engine question the earlier round left implicit,
  and [section 7.1](#71-the-manual-visual-review) now allocates the appearance evidence across the four
  supported browsers explicitly — which is the whole of the review row's movement;
- and **W4 is now two gates rather than one** ([03 §4.2](03-modernization-roadmap.md)): gate **4a** is a
  build-governance bootstrap that W6 consumes, gate **4b** is baseline green and captured. The workstream
  keeps 03's name, which changed with the split.

In that round **W4** moved from 21.5 / 37 / 64 to 35.5 / 60.5 / 102.5, **W6** from 2.5 / 5 / 9 to
2.5 / 4.5 / 8.5, **W7** from 48.5 / 87.5 / 151.5 to 57.5 / 102 / 175.5, **W11** from
6.5 / 11.5 / 22 to 9 / 15.5 / 29, **W12** from 1.5 / 3 / 6 to 2 / 4.5 / 8 and the manual visual review
from 2 / 3.5 / 6 to 2.5 / 4.5 / 7.5. The six deltas — **+14 / +23.5 / +38.5** on W4,
**0 / −0.5 / −0.5** on W6,
**+9 / +14.5 / +24** on W7, **+2.5 / +4 / +7** on W11, **+0.5 / +1.5 / +2** on W12 and
**+0.5 / +1 / +1.5** on the review — sum to **+26.5 / +44 / +72.5**, which is the whole of the movement
between the total of 107 / 195.5 / 351.5 and the 133.5 / 239.5 / 424 this round supersedes:
107 + 26.5 = 133.5,
195.5 + 44 = 239.5, 351.5 + 72.5 = 424. That delta addition itself, at the expected band:
23.5 − 0.5 + 14.5 + 4 + 1.5 + 1 = **44**. Excluding the review, the fifteen-workstream delta was
**+26 / +43 / +71**: 105 + 26 = 131, 192 + 43 = 235, 345.5 + 71 = 416.5.

**W6 is the one row that falls, and it falls for a charge-once reason rather than because its work
shrank.** Gate 4a now originates `global.json` and the root `NuGet.config` and proves locked-mode restore
on the contracts project, so W6 inherits those two files rather than creating them. W6 still authors the
root solution, the tool manifest and the converted project's own lockfile, and still re-verifies the
inherited pair against the converted project — which is why the fall is half a day at the expected band
and nothing at the low band, not the whole of the bootstrap.

### 5.2 Basis of estimate, per workstream

#### W1 — Approval of this assessment

**Scope** is [03 §5](03-modernization-roadmap.md)'s: the gate on everything, exiting with a documented
approval that records a decision on each approved delta and on each risk this document escalates for
decision rather than mitigation.

**Basis: a census of approval acts at a stated rate**, and input 17's **27 delta decisions are
deliberately not among them** — they are this gate's *requirement*, sized in their own row, for the reason
the partition below states.

**An approval act is defined so the census is countable and its members are mutually exclusive:** one
recorded output that no other act in the census produces. Each of the three kinds below answers a
**different question**, so no act's output can stand in for another's — a constituency that has not been
briefed has taken no position, an escalated question nobody answered is unanswered whoever else convened,
and a gate nobody recorded is not open. There are **ten**:

| Approval acts | Count | What one act is, and the recorded output that makes it its own |
| --- | ---: | --- |
| **Constituency briefings** — security, product, engineering, the data owner, operations and **legal** | **6** | Convening one constituency, putting the thirteen deliverables in front of it, and recording **that constituency's stated readiness to proceed to a decision** — nothing more. The reading is *inside* the act, because the briefing is what the reading is for and a constituency reads only what bears on what it owns. The briefing explicitly does **not** answer any of the 27 deltas, which [§7.2](#72-the-approved-delta-sign-offs) pays per decision, nor any of the three questions in the row below, which are separately recorded there |
| **Escalated risk decisions that are not themselves approved deltas** — [R1](#r1--the-target-framework-support-window), [R13](#r13--one-database-one-blast-radius), [R15](#r15--personal-data-governance-is-unowned) | **3** | Answering **one named escalated question** — which target-framework position to take, whether to accept one database's blast radius, who owns personal-data governance — and recording that answer against its owner. Each is indivisible, and none is implied by any briefing above: a constituency can be briefed and ready without the question being answered, and R15's answer requires the data owner, security and legal to converge on one recorded owner rather than three positions |
| **The gate record** — the documented approval to begin implementation | **1** | Writing the decision that closes this workstream's exit gate, against which every later workstream's authority is checked. It is not any constituency's position and not an answer to an escalated question; it is the record that the other nine acts are complete |
| **Total acts** | **10** | |

**Why the three kinds cannot collapse into each other.** A briefing pays for assembling a constituency and
getting it to the point of deciding; an escalated decision pays for converging on one answer among owners
who may disagree; the gate record pays for writing the authority every later workstream is checked
against. The 6 and the 3 also do not correspond one-to-one — R15 alone draws three of the six
constituencies and R1 and R13 draw overlapping subsets — so neither count can be derived from the other,
which is the property that makes them separate census dimensions rather than one dimension counted twice.

**The rate is `0.3 / 0.6 / 1.2` IED per act**, and `10 × 0.3 / 10 × 0.6 / 10 × 1.2` = **3 / 6 / 12** — no
rounding needed under [§4.2.1](#421-the-rounding-rule-stated-once), because each column already sits on a
half. The rate is this row's one judgement, and it is stated so it can be disputed: **low** is a
constituency that has read ahead and settles in a single sitting; **expected** is one sitting plus the
follow-up that circulating a position internally produces; **high** is a second full sitting, which is what
an act costs when the first one surfaces a question the constituency has to take away. This row is one of
the three where obtaining a decision **is** the work, so [§4.2](#42-the-unit-defined)'s exclusion of
meeting and approval-waiting overhead does not apply to it.

**The derivation is live, not a rationalization of a figure already chosen.** Each act carries
`0.3 / 0.6 / 1.2`, so the band moves by exactly that if the census does: a seventh constituency, or a fourth
escalated decision that is not itself a delta, adds it; an escalated decision reclassified as a delta and
paid in [§7.2](#72-the-approved-delta-sign-offs) removes it. **Assumption A5** enters through the rate
rather than as a caveat beside it: where a constituency is a role rather than a named person, its act is
priced at the **high** figure, because identifying the person who may take the position is part of taking
it.

**The band reads the same as earlier versions of this document, and that is a reproduced result rather than
a survivor of arithmetic.** No earlier reading derived it from an act census at all — it was a judgement
that the driver is the convening, and this census is what that judgement looks like once it is made
checkable. **The two corrections below do not cancel, and the census is what shows they do not:** before
them it would have counted **eleven** acts — five constituencies, five escalated decisions and the record —
which at this rate gives `3.3 / 6.6 / 13.2`, rounded under
[§4.2.1](#421-the-rounding-rule-stated-once) to **3.5 / 7 / 13.5**. The corrections move the census to ten
acts, and ten acts give 3 / 6 / 12. So the published figure did not move because it had never been derived
from the eleven-act census in the first place; it is derived now, from the ten.

> **Why six constituencies and not five, and why three escalated decisions and not five.** Input 17
> identifies five constituencies across the 27 deltas, but the escalated risk decisions reach one further:
> [R15](#r15--personal-data-governance-is-unowned)'s owner is the data owner **with security and legal as
> co-approvers**, and [W16](#w16--personal-data-governance)'s policy stage names the same
> three. Legal therefore has to convene at this gate even though it owns no delta, so the convening cost is
> a **six**-constituency cost.
>
> In the other direction, **two of §9.4's five escalated decisions are themselves approved deltas** and are
> already priced per decision in [§7.2](#72-the-approved-delta-sign-offs):
> [R7](#r7--the-narrowed-browser-matrix) is the narrowed-browser-matrix delta, whose approval owner is
> product, and [R9](#r9--cutover-re-authentication-and-anonymous-cart-loss) is the
> reauthentication-and-anonymous-cart-loss delta, whose owners are product and operations. Counting their
> decisions here as well as there would be the same double count this sub-section exists to remove. They
> remain in §9.4 because §9.4's subject is *which risks are decisions rather than mitigations*, which is
> true of them; what changes is only **which row pays for taking them**.
>
> **What the two corrections do to the act census, stated as arithmetic rather than as a cancellation.**
> The first **adds one act** — legal's briefing, `+0.3 / +0.6 / +1.2`. The second **removes two** — R7's and
> R9's decisions, `−0.6 / −1.2 / −2.4` — because they are paid per decision in §7.2 instead. Net
> `−0.3 / −0.6 / −1.2`, from an eleven-act census to a ten-act one. That is **not** a cancellation, and the
> basis above does not claim one: the two movements have different sizes, and the reason the published band
> reads unchanged is that it had never been derived from the eleven-act census.

> **The partition of input 17, stated once and precisely, because the alternative is a double count.**
> Input 17 carries two numbers — **5** constituencies and **27** decisions — and **exactly one row consumes
> each**, so no unit of work is counted twice:
>
> | Dimension of input 17 | Counted in | What it pays for |
> | --- | --- | --- |
> | **The convening** — 6 constituencies, being input 17's five plus legal | **W1's band**, this row | Its **ten approval acts**: briefing each of the six owners on thirteen deliverables, taking the **three** escalated risk decisions of [§9.4](#94-the-five-risks-that-are-approval-decisions-not-mitigations) that are **not** themselves deltas — R1, R13 and R15 — and writing the gate record |
> | **27 individual delta decisions** — the per-decision marginal cost | **[§7.2](#72-the-approved-delta-sign-offs)'s own row**, 4 / 7.5 / 13 | Obtaining and recording a decision on each of the 27, including consent from **every** constituency the 16 multi-owner deltas name — and therefore including §9.4's **R7** and **R9**, which are deltas 6 and 7 of that set |
>
> An earlier version of this basis listed the 27 decisions among W1's own drivers while §7.2 asserted they
> were not double-counted into W1. Both statements could not be true of one band, and the ambiguity was the
> defect rather than the arithmetic: W1's band never moved when the delta count did — it was 3 / 6 / 12 at
> 14 deltas, at 18 and at 27 — so the decisions were only ever *sized* in §7.2. The basis above now says so
> in its own terms, and the two rows sum without overlap: **6 + 7.5 = 13.5** expected IED for the whole
> approval activity, which is what [§8.2](#82-concurrency-permitted-by-the-graph)'s **set 0** — the root
> set, which holds W1 and the sign-offs together and nothing else — and
> [§8.3](#83-the-critical-path-and-what-to-do-first-if-the-goal-is-to-narrow-the-estimate)'s chain both
> carry.

**Band 3 / 6 / 12. Medium confidence**, and it is `10 acts × 0.3 / 0.6 / 1.2` and nothing else — the census
above, at the rate above. **The corrected delta count does not enter it**, and that is a property of the
partition rather than an absorption: the count went from 18 to 27 when input 17 was reconciled against
[05 §11.5](05-aspnet-core-migration-approach.md), and **every one of the 27 is paid per decision in
[§7.2](#72-the-approved-delta-sign-offs)**, which is why that row is reported separately from this one and
where the nine additional decisions landed. What *would* move this band is a change in the act census: a
constituency added or removed, an escalated decision reclassified, or the gate record split. Confidence is
Medium because the count of acts is fixed and enumerated while the rate depends on other people's
availability — which is exactly what assumption **A5** and the high figure describe.

#### W2 — MVC 5 build reproduction and the restore precondition

**Scope** is [03 §5](03-modernization-roadmap.md)'s: **producing** the Windows verification run for the
migration source from a clean checkout, recording the conditions under which it was done, and running
the **AppCAT static assessment** inside the same restored tree.

**Basis — and this row is a first execution, not a reproduction.**
[10 §1.2](10-build-and-deployment-requirements.md), the owner of per-edition build outcomes, carries the
migration source's build assessment as **blocked pending a Windows verification run**. What that owner
establishes is a **precondition failure** — a clean checkout of the migration source commits no restored
packages at all, so no build can start until a restore succeeds against a source the repository never
declares. **Whether the application compiles once restored is not established**, and the Mono result in
[10 §3.1](10-build-and-deployment-requirements.md) is supplemental portability evidence rather than a
build on the prescribed toolchain. W2 therefore carries four tasks rather than one: obtain a host
meeting **A2**; declare a restore source explicitly so the restore is reproducible rather than
incidental; execute the run and record it to **both** owners' specifications — the **five fields**
[10 §3.2](10-build-and-deployment-requirements.md) requires before the status can move, carried inside
the **six-item** gate 2a record [03 §5](03-modernization-roadmap.md) requires, whose two additional items
are the vacuous test result and the defect decision — a record that requires **Debug and Release both**,
per [10 §3.2](10-build-and-deployment-requirements.md), so the Release publish path is not left unverified;
and run the **AppCAT static assessment** (`dotnet-appcat`) over the restored tree, triaging its findings
against [12 §2.3](12-migration-blockers.md)'s blocker index.

**Band 2 / 4 / 9, and the width is the point.** Low assumes a conforming host is already available, the
restore succeeds first time, the build reports no error, and AppCAT surfaces nothing the blocker index
does not already carry. Expected assumes host preparation plus one restore-source correction and a
handful of AppCAT findings to adjudicate. **High assumes the run fails** and the failure has to be
characterized, decided on, and repaired in the build environment until a passing run is recorded — which
is gate 2b's loop. [03 §5](03-modernization-roadmap.md) permits **gate 2a** to close on the failure
itself, since "recorded" and not "green" is *that gate's* operative word, while **W2 does not exit until
2b has closed** — so the failure case is a repair loop inside this band rather than an early exit from
it. Where the defect proves irreparable within 2b's bound, 03 escalates to W1 and this model is
**re-estimated rather than stretched**. The band is wide because W2's
*outcome* is unknown, not because its tasks are unclear;
[R2](#r2--the-migration-sources-build-reproducibility) carries that as a risk, and **no band anywhere in
this model is discounted on an assumption that the run passes**.

**One thing this workstream cannot buy, whatever it reports.** An error-free and warning-free result
would still say nothing about the 29 Razor views, because view compilation is disabled (F-08-16). The
views' first compile-time exposure is W7's, and W7's band is set accordingly.

**What the AppCAT report can and cannot corroborate, stated because the ordering matters.**
[03 §5](03-modernization-roadmap.md) records that the report's output is *"corroborating evidence for the
effort model"* and routes it here. It is used here on that basis with one ordering qualification this
document must make rather than inherit: **W2 runs after W1**, and W1's exit gate is the approval of this
assessment — so no figure below was derived from a report that does not yet exist, and none is presented
as corroborated by one. The report's real use is **downstream**: an AppCAT finding matched to a blocker
[12 §2.3](12-migration-blockers.md) already carries confirms the enumeration W6's and W7's bands are
built on, and one it does not carry is a new work item, in which case this document is **re-estimated
rather than adjusted** — the same treatment [section 6.3](#63-what-is-deliberately-not-in-the-total)
gives every input this model does not assume.

#### W3 — Authoritative schema extraction

**Scope** is [03 §5](03-modernization-roadmap.md)'s: an authoritative schema definition for **both**
stores, obtained by querying the catalog rather than inferred, with every object mapped to the intended
target model — and the Identity store's column set settled as fact, replacing the qualified evidence of
[12 §5](12-migration-blockers.md).

**Basis.** Two databases; a six-entity catalog model [01 §6.1](01-architecture-overview.md) plus the
Identity 1.0 tables. The mechanical query work is small. The band's width is **mapping and
adjudication**: every column, key, index, default and delete rule has to be compared against the
intended model, and every unmapped object explained or scheduled for removal. Two blockers are retired
by this workstream's output — the unknowable Identity schema and the absence of a usable schema
baseline [12 §5](12-migration-blockers.md) (input 12).

**Band 2 / 4 / 8. Low confidence, and deliberately so** — the work is cheap but its *output* is
unknown, and three other workstreams are estimated against that output.
**This is the highest-leverage 4 IED in the model**; see
[section 8.3](#83-the-critical-path-and-what-to-do-first-if-the-goal-is-to-narrow-the-estimate).

#### W4 — Build-governance bootstrap, pre-port behavioural baseline and test suite

**Scope** is [03 §5](03-modernization-roadmap.md)'s, with the architecture and required coverage owned
by [05 §12](05-aspnet-core-migration-approach.md). Not redesigned here. **The name is 03's and it
changed with the gate split below**; this document follows it rather than keeping the shorter one.

**This workstream is now two gates, and the split is load-bearing for the sequencing rather than
cosmetic.** [03 §4.2](03-modernization-roadmap.md) draws **gate 4a** — the build-governance bootstrap:
`global.json`, the root `NuGet.config`, the contracts project and its lockfile, with locked-mode restore
and build demonstrated — and **gate 4b**, the baseline green and captured. 4a's only entry is W1; 4b
needs 4a and W2's gate 2b. The table below is grouped by gate and subtotalled per gate, because
[section 8.2](#82-concurrency-permitted-by-the-graph) places the two in **different concurrency sets**
and [section 8.3](#83-the-critical-path-and-what-to-do-first-if-the-goal-is-to-narrow-the-estimate)
needs 4a's own weight to compute the chain.

**The scope boundary that sets this band, stated next because it moves 45 expected IED.**
[03 §5](03-modernization-roadmap.md) splits the suite in two and gives this workstream **half (a) and
nothing more**: the shared contract assertions and the abstract topology that carries them, the legacy
fixture with its private deployment lifecycle and its two-database reset, the fixture dataset both halves
load, the semantic normalization, the approved deltas enumerated as expected differences, and the
`Category=Baseline` execution half with the redacted baseline record it hands forward. **Half (b) — the
disposable engine fixture, the in-process target host, the fixture lifecycle and isolation policy, the
destructive-operation controls those two require, the browser-driven flow of input 29, and the
`Category=Target` assertions — is W7's exit criterion**, because all of it exercises an artifact that does
not exist until the port lands. This model follows that split: the target-facing half is costed as named
sub-rows of W7 rather than here.

**Basis — entirely net-new, with no legacy volume to scale from.** There are **0 tests** in the
repository (input 15, command-verified in [A.2](#a2-the-absences-that-size-the-net-new-work)), so this
is not a migration of a suite but the construction of one. **The counting unit is the case** — one
lettered scenario entry in [05 §12.4](05-aspnet-core-migration-approach.md)'s matrix — with **fixture
executions** counted separately, because a case that runs against both fixtures is authored once and
executed twice.

| Component | Input | Low | Expected | High |
| --- | --- | ---: | ---: | ---: |
| **Gate 4a** — `global.json` pinning the SDK feature band and the root `NuGet.config` clearing inherited sources, with **locked-mode restore and build demonstrated** rather than configured | The governance files [03 §5](03-modernization-roadmap.md) places in this gate; the band and the source policy owned by [04 §6](04-dotnet8-migration-strategy.md) | 1 | 2 | 3.5 |
| **Gate 4a** — the **contracts project** itself, its **lockfile**, the **two `xunit.runner.json` files**, and the test projects' pins resolved and restored — with the **three runner and build-asset packages declared directly in every runnable test project** rather than inherited, because build and analyzer assets do **not** cross a project reference | **2** projects, **2** runner files, **5** lockfiles; **6** independent test-tooling pins plus `Microsoft.Extensions.Identity.Core` on the 8.0.30 band, and **3** of those declared twice (input 27, entry and pin counts); **3** execution categories (input 21) | 1.5 | 3 | 5 |
| **Gate 4a subtotal** | | **2.5** | **5** | **8.5** |
| **Gate 4b** — the **contract topology**, now **declarable rather than described**: one **abstract** contract class per surface plus a **sealed concrete binding in each runnable assembly**, because a project reference does **not** enrol a referenced assembly's tests; **every base, concrete and fixture type `public`**; a **`[CollectionDefinition]` collection class per surface group per assembly** implementing `ICollectionFixture<TFixture>` with its **collection name taken from a `const`**, so a mistyped name is a compile error rather than a silently un-parallelized run; and the runtime-neutral context those bindings hand to the assertions | **6** abstract classes with per-assembly bindings, **1** neutral context, **9** normal collection groups plus **3** row-specific ones, in **2** assemblies (input 23 and input 27, entry count) | 2.5 | 4.5 | 7.5 |
| **Gate 4b** — the runtime-neutral **`IStoreObserver`** with its explicit **DTO surface** and its **legacy implementation**, plus the **`IStoreSetup` write API** placed deliberately **off the injected context** so a contract body cannot name it, and **OWNER-backed** so a setup step cannot prove the runtime credential could have done it | `IStoreObserver`'s **8** original members and its **5** additional read projections; `IStoreSetup`'s **8** operations across **11** members; the first of **2** implementations, **1** DTO surface (input 27, entry count) | 3 | 5.5 | 9.5 |
| **Gate 4b** — the legacy fixture's **private deployment lifecycle**: copying the built legacy output outside the checkout, copying both store pairs, rewriting the deployed copy's connection strings including a **per-run Identity catalog name**, binding a per-run port, starting the host and **capturing its process id**, polling readiness and then polling **startup quiescence**, and stopping, detaching and deleting on teardown | **7** lifecycle steps (input 25, entry count), the seventh because the migration source provisions its administrator from an `async void` [src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:21] | 2.5 | 4 | 7 |
| **Gate 4b** — the legacy fixture's **store reset** and the destructive-operation controls **demonstrated** on the copies it destroys, which [03 §5](03-modernization-roadmap.md) makes a condition of this gate: the **three SQL identities**, the **durable ownership registry** with its JSON sidecar, **identifier re-resolution immediately before every `DROP`**, **run-marker validation** on copied directories, and a **standalone orphan-sweep class** with an **always-run cleanup job separate from the job the watchdog kills** | **2** store pairs, **4** committed files (file count, input 24); **3** hard gates, **3** identities, the registry and sidecar, the sweep class and the cleanup job, each with a refusal path to exercise (input 24, entry count) | 2.5 | 4.5 | 8 |
| **Gate 4b** — **the 144 cases that run against both fixtures**, authored here as the shared contract | **144** of the **242** cases (case count, input 14) | 14 | 22 | 36.5 |
| **Gate 4b** — the **twelve committed fixture inputs**: the deterministic **manifest** with its published per-entity counts, fixed keys, ranks and quantities, a named administrator, a migrated-hash account and a published fingerprint; **nine `ModelOverrides/` divergence-seam files**, one per schema-divergence dimension the diff gate must refuse on; `seed-expected.json`; and the **committed** `baseline-reference.json` — with the **legacy loader** and the **7 post-load invariants** asserted before the first case | **12** committed files: **1** manifest of 10/6/12/0/7/33 rows, **9** override files, **2** companions; **7** invariants; the first of **2** loaders (input 25, file and entry counts) | 2 | 5 | 7.5 |
| **Gate 4b** — semantic normalization, and the **22** approved deltas as expected differences rather than failures | **22** deltas (input 17, entry count); the volatile-value classes and the **4** pinned locale and collation values (input 26) | 2 | 3.5 | 6 |
| **Gate 4b** — the **two diagnostic record schemas** inside one allow-list, the **sanitized exception projection**, the **closed 26-code `operation` set**, the **redactor with its own test corpus**, and the **keyed one-way cross-run pseudonyms** whose corrected scheme invokes the **pinned `ILookupNormalizer`** — without which a failure is either undiagnosable, unattributable or a disclosure | **2** schemas, **7** masked classes, **26** operation codes, **2** failure classes plus the expected-difference state, **1** pseudonym key with a custody and destruction rule (input 22, entry count) | 3 | 5.5 | 9 |
| **Gate 4b** — the **144 `Category=Baseline` executions** on the Windows agent, and the **redacted baseline record** handed to the target half: **11 gating `baselineSource` values** compared and failed closed on, a separately recorded **7-fact `targetRun`** namespace never compared, a **`coverage` completeness object** carrying each `Category=Baseline` case identifier and status, an **artifact transfer with a digest sidecar**, and the **publication policy** — a `baseline-capture-diagnostic` on a failed or incomplete capture, and a `baseline-record` **only** on a complete run in which every one of those cases passed | **144** executions (input 14); **1** record with **11** gating values, **7** recorded facts and a completeness object, plus the transfer and the two publication outcomes (inputs 21 and 26) | 3 | 5.5 | 10 |
| **Gate 4b** — **culture, UI culture, time zone and collation** established as a published contract and enforced on this half's host and agent | **4** pinned values (input 26, entry count) | 0.5 | 1 | 2 |
| **Gate 4b subtotal** | | **35** | **61** | **103** |
| **W4 total** | | **37.5** | **66** | **111.5** |

**The addition, printed so it can be checked rather than trusted.** Gate **4a** — low
1 + 1.5 = **2.5**, expected 2 + 3 = **5**, high 3.5 + 5 = **8.5**. Gate **4b** — low
2.5 + 3 + 2.5 + 2.5 + 14 + 2 + 2 + 3 + 3 + 0.5 = **35**, expected
4.5 + 5.5 + 4 + 4.5 + 22 + 5 + 3.5 + 5.5 + 5.5 + 1 = **61**, high
7.5 + 9.5 + 7 + 8 + 36.5 + 7.5 + 6 + 9 + 10 + 2 = **103**. The workstream: 2.5 + 35 = **37.5**,
5 + 61 = **66**, 8.5 + 103 = **111.5**.

**How the case component was re-derived, because it is the largest single sub-row here and the largest
single figure anywhere in this model.** The previous band authored the **98** both-fixtures cases of a
72-row matrix; the current matrix resolves the same subject into **75 rows and 242 cases**, of which the
**144** both-fixtures cases fall to this workstream — the **30 both-fixture rows' 132 cases** plus the
**12 both-fixture halves of the 6 mixed rows** [05 §12.4](05-aspnet-core-migration-approach.md). **The
per-case authoring rate of the superseded band is held constant and the case count drives the
increase**: 144 ÷ 98 = **1.47×**, so 9.5 / 15 / 25 scaled by that factor is 13.96 / 22.04 / 36.74, which
at the **0.5 IED granularity** every figure in this table is expressed in is **14 / 22 / 36.5**. The
resulting rates are **0.097 / 0.153 / 0.255 IED per case** — the same rates the two previous bands
printed, to that granularity. Nothing in the method changed; only the count did.

> **Holding the rate constant is a deliberate choice here, and it is worth stating why it is not obviously
> the conservative one.** Much of the case growth is concentrated in **boundary tables**: row 67 alone is
> a **54-case** field-by-field boundary matrix and row 72 moved from four cases to eight
> [05 §12.4](05-aspnet-core-migration-approach.md). A case inside a data-driven boundary table is cheaper
> to author than the first case of a new surface, which argues the blended rate should **fall**; against
> that, the same growth includes rows restructured rather than extended (28 and 36) and the divergence
> seam row 53, which are not table rows at all. Rather than adjust the rate in either direction on
> judgement, the derivation **holds it** — which reads slightly high for the table rows and slightly low
> for the restructured ones, and is the only choice that keeps this figure comparable with the two bands
> it supersedes. The **high band** is where the residual sits: 36.5 against 22 expected is a 1.66×
> spread on the largest row in the model.

**Seven items the previous band did not count at all, and they are the rest of the increase.** Each is
scope [05](05-aspnet-core-migration-approach.md), [04](04-dotnet8-migration-strategy.md) and
[03](03-modernization-roadmap.md) added after the previous derivation, and each is charged once:

- **the governance bootstrap of gate 4a** (input 27; [04 §6](04-dotnet8-migration-strategy.md)) —
  **2 expected IED**, brand new to this workstream, because 03 moved the SDK pin, the package-source
  policy and the proof that locked-mode restore actually restores from the version W6 originated them in
  to the gate that precedes it. W6 loses the corresponding half-day, so this is a **transfer plus a
  proof obligation**, not a duplicate;
- **the contract topology** (input 27) — now **4.5 expected**, its own sub-row, because per-surface
  **abstract** classes with **sealed per-assembly bindings** are structure rather than a naming
  convention: a project reference does not enrol a referenced assembly's tests, so every shared contract
  needs a concrete binding in each runnable assembly or it silently never runs;
- **`IStoreObserver`, its DTO surface and its legacy implementation** (input 27) — now **5.5 expected**
  with `IStoreSetup` alongside it, and with no predecessor in the previous derivation, which asserted
  database outcomes per runtime rather than through one neutral abstraction. It is what lets a shared
  contract assert **an absence** — "no row written" — against two different stores;
- **startup-quiescence polling** in the lifecycle (input 25) — the seventh step, taking that sub-row from
  2.5 to **4 expected**, because a readiness probe answering does not mean the source's `async void`
  administrator provisioning has finished writing;
- **the third SQL identity, the durable ownership registry with its sidecar, identifier re-resolution
  before every `DROP`, run-marker validation, the standalone sweep class and the always-run cleanup job**
  (input 24) — taking the reset sub-row from 2.5 to **4.5 expected**. A cleanup job that must survive the
  watchdog killing the test job is a second job, not a `finally` block;
- **the enlarged fixture dataset with published counts, its seventh invariant and its two companion
  files** (input 25) — **+0.5 expected**, deliberately small: the dataset grew in rows, and rows in a
  manifest are volume rather than structure;
- **the diagnostic and provenance closures** — the sanitized exception projection, the 26-code
  `operation` set and the corrected keyed-alias scheme (input 22) at **+1 expected**, and the provenance
  split with its digest-sidecar transfer and fail-closed consumption (input 26) at **+1.5 expected** on
  the execution row, which also absorbs 98 → 144 executions.

**And three further approved deltas** (input 17) take the normalization sub-row from 3 to **3.5
expected**. Two of the three are new *shapes* of expectation rather than new rows of the same shape — a
`405` where the source answers `404`, and a `400` where the source accepted and wrote — so the sub-row
moves by half a day rather than by nothing.

**Re-derived again this round, from 35.5 / 60.5 / 102.5, and the movement is four components and nothing
else.** The case sub-row does **not** move: the matrix grew from 239 to 242 cases, but all three added
cases are **target-only** — row 24's fifth case and row 64's fourth and fifth — so the both-fixtures count
this gate authors stays at **144**. What moves is structure:

- **the topology became declarable rather than described** — every base, concrete and fixture type
  `public`, and a `[CollectionDefinition]` collection class **per surface group per assembly** implementing
  `ICollectionFixture<TFixture>` with its name taken from a `const`, across **9** normal groups and **3**
  row-specific ones in **2** assemblies (input 23). That is **12 declarations in each of two assemblies**
  rather than a convention, so 2 / 3.5 / 6 becomes 2.5 / 4.5 / 7.5 — **+0.5 / +1 / +1.5**;
- **`IStoreSetup` arrived as a surface of its own** — **8** operations across **11** members, OWNER-backed
  and deliberately unreachable from the injected context so the compiler enforces the read/write boundary
  a review convention previously implied — together with **5** additional `IStoreObserver` read
  projections. Two interfaces where there was one, so 2 / 3.5 / 6 becomes 3 / 5.5 / 9.5 —
  **+1 / +2 / +3.5**;
- **the fixture inputs became twelve committed files**, the nine `ModelOverrides/` divergence-seam files
  being new: each expresses **one** schema-divergence dimension and nothing else, which is what makes the
  diff gate's refusals attributable to a dimension rather than to a mixture. So 1.5 / 3.5 / 5.5 becomes
  2 / 5 / 7.5 — **+0.5 / +1.5 / +2**;
- **the baseline record gained a completeness object and a publication policy** — the `coverage` object
  carrying each `Category=Baseline` case identifier and status, and the rule that a failed or incomplete
  capture publishes a **diagnostic** rather than a record, because a digest proves integrity and not
  completeness. So 3 / 5 / 9 becomes 3 / 5.5 / 10 — **0 / +0.5 / +1**;
- and **gate 4a's project row** absorbs the **direct declaration of the three runner and build-asset
  packages in every runnable test project**, plus `Microsoft.Extensions.Identity.Core` as the pin the
  suite resolves `ILookupNormalizer` from, so 1.5 / 2.5 / 4 becomes 1.5 / 3 / 5 — **0 / +0.5 / +1**.

0.5 + 1 + 0.5 + 0 + 0 = **+2**, 1 + 2 + 1.5 + 0.5 + 0.5 = **+5.5**, and
1.5 + 3.5 + 2 + 1 + 1 = **+9**, so 35.5 + 2 = **37.5**, 60.5 + 5.5 = **66** and
102.5 + 9 = **111.5**.

This band also depends on assumption **A2** holding in a stronger form than A2 states: the agent of
input 21 must **run** the legacy application, not merely build it.

**What is not in this band, and where it went.** The `tools/migrate-data` artifact is **W7's** — 03 §5's
W7 scope statement places it there and [section 5.2](#w7--the-aspnet-core-port) carries its 9 expected
IED — the manifest's **target loader**, being catalog-direct plus that artifact's `load` mode, and the
observer's **target implementation** are inside W7's machinery sub-row; the **browser-driven flow** of
input 29 is W7's own sub-row, since the flow it drives exists only in the ported application; and the
**dual-OS pipeline stages** that automate input 21's two agents, together with the **browser-install
step**, are **W11's manifest half**, whose architecture
[06 §12.1](06-azure-hosting-recommendations.md) owns. None is charged here,
and none is charged twice.

**Band 37.5 / 66 / 111.5, splitting 2.5 / 5 / 8.5 for gate 4a and 35 / 61 / 103 for gate 4b. Low
confidence, unchanged — and the reason has shifted rather than weakened.**
The scope is enumerated far more tightly than when this row read 8 / 13 / 22, 11.5 / 19 / 33 or
21.5 / 37 / 64: a case-level matrix with named assertion fields replaces a surface list, and the
topology, observer, fixture, dataset, diagnostic and provenance obligations are now specified item by
item, which removes most of the *scope* uncertainty. What
keeps the confidence Low is that the suite still characterizes behaviour **nobody has yet asserted**, so
the count of cases is known while the count of *surprises* is not — and **seven** of the components have
no precedent in this repository to calibrate against, the same seven
[section 4.4](#44-confidence-and-its-reason) lists: a two-platform execution with a handoff artifact
under fail-closed consumption, a private legacy deployment lifecycle with a quiescence poll, an
abstract-plus-sealed contract topology that has to enrol tests in two assemblies, a runtime-neutral store
observer with two implementations, a keyed
pseudonym scheme with key custody and destruction, a redactor that must itself be tested, and a
three-identity bootstrap with a durable
ownership registry whose deletion path re-resolves its target before acting.
**Re-derived from 35.5 / 60.5 / 102.5**, itself re-derived from 21.5 / 37 / 64, from 11.5 / 19 / 33, from
8 / 13 / 22, and before that from 12 / 20 / 34 when the target-facing half moved to W7.
**This is still the second-largest row in the model and the one most likely to be cut under pressure**;
[R3](#r3--the-absent-regression-baseline) is why cutting it removes the only means of substantiating
behaviour preservation. **The gate split makes one cut newly tempting and it is the wrong one**: gate 4a
is 5 of the 66 and W6 now consumes it, so cutting 4a does not save 5 — it stops W6.

**The visual baseline capture is not inside this band.** [03 §5](03-modernization-roadmap.md) attaches
it to this workstream's exit gate, and it is sized in [§7.1](#71-the-manual-visual-review) as a non-code
task so that the capture and the review are visible as one item split across two points in the sequence.
[Section 8.2](#82-concurrency-permitted-by-the-graph) places its 1 expected IED in the same concurrency
set as this workstream.

#### W5 — Repository-wide path-casing audit

**Scope** is [03 §5](03-modernization-roadmap.md)'s: every mismatch identified and its correction
specified, with the audit made **repeatable as a pre-deployment check** rather than performed once.

**Basis: input 31, not inputs 7, 8 and 11.** The search space is fully enumerated but it is **not** the
migration source's. This workstream's own name says *repository-wide*, and the three inputs an earlier
version of this basis used count MVC 5 only: input 7's **28** assets, input 8's **11** helper call sites
and input 11's **4** `@Url.Content` sites. Input 31 is the actual space, and it is stated **partitioned by
edition** so that "all three editions" is a count rather than an adjective. Two totals come out of it and
they are not interchangeable: the **containers** a checker must open, and the **literal sites** inside
them. Every figure is command-verified in [A.3](#a3-helper-view-and-site-counts).

| Edition | Served static files | Views | `BundleConfig.cs` | **Containers** | `~/` bundle literals | `@Url.Content` | Render sites | **Literal sites** |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| MVC 5 | 28 | 29 | 1 | **58** | 12 | 4 | 11 | **27** |
| MVC 4 | 90 | 29 | 1 | **120** | 24 | 4 | 10 | **38** |
| MVC 3 | 55 | 25 | 0 | **80** | 0 | 23 | 0 | **23** |
| **All three** | **173** | **83** | **2** | **258** | **36** | **31** | **21** | **88** |

**Both `BundleConfig.cs` files are in that space, and MVC 3 has none** — it ships no `App_Start` folder, so
its 23 literal sites are all `@Url.Content` calls in views. **The `@Url.Content` figure is the one that
reorders the work: 23 of the 31 are MVC 3's**, so the old basis was sized against **4 of 31** — an eighth
of that dimension — and a pattern set validated only against those 4 is unproven against the edition where
the concentration actually is.

**The defect is not confined to the migration source either, and that is what turns "repository-wide" from
a scope word into a cost.** The known mismatch is
[src/MVC5/MvcMusicStore/App_Start/BundleConfig.cs:28], where `~/Content/site.css` is registered against a
tracked [src/MVC5/MvcMusicStore/Content/Site.css: the tracked path's own capitalisation, not a line within
the file]. **The identical defect exists in MVC 4** — the same lowercase `~/Content/site.css` at
[src/MVC4/MvcMusicStore/App_Start/BundleConfig.cs:26] against a tracked
[src/MVC4/MvcMusicStore/Content/Site.css: likewise the path's own capitalisation] — so the second
`BundleConfig.cs` is not a container the audit opens for completeness' sake; it holds a live mismatch of
the same class, and [A.5](#a5-the-corroborating-case-mismatch-r8) prints both halves of both pairs.

**Two corrections pull in opposite directions, and neither is a cancellation of the other.** The space is roughly
five times wider than the old basis assumed. But [03 §5](03-modernization-roadmap.md) narrowed the **exit**
in the same round: the audit now ends on a complete enumeration with each correction specified plus a
**repeatable checker that runs without a deployment**, and the case-sensitive serve it previously had to
demonstrate moved downstream to where the ported application exists. Removing a deployment from an exit
gate is a larger saving than a wider grep is a cost, because the enumeration is mechanical — one pass of
one pattern set over the **258** containers input 31 enumerates (`173 + 83 + 2`) and the **88** literal
sites inside them (`36 + 31 + 21`) — while standing up a serve to prove it was the part that required an
environment. The container figure is `173 + 83 + 2` and not `173 + 83`: the two `BundleConfig.cs` files
are containers in their own right, holding 36 of the 88 sites between them while being neither a served
asset nor a view, and an earlier form of this sentence added the two editions' bundle literals to the site
count while leaving their files out of the file count.

**What is repository-wide here and what is not, because [03 §5 W5](03-modernization-roadmap.md) draws that
line inside this row rather than around it.** Its scope is every path literal "across bundle registrations,
`@Url.Content` calls, view paths and any other path literal"; its exit condition 1 names **the migration
source** for the corrections it hands to W7. Both are right, and the estimate is sized on the first: the
**enumeration and the checker are repository-wide**, because a checker wired into W11's Test stage runs
over a repository rather than over one folder, and because its pattern set is only proven where the
literals concentrate — MVC 3's 23 sites, which no migration-source pass touches. The **correction list W7
consumes is the migration source's**, because W7 ports one edition and the other two are retained
read-only.

**That split produces one obligation the narrower reading could not see, and it is a decision rather than a
task.** A checker that "fails on any mismatch" over this repository would fail from the moment it is wired
in, because MVC 4's mismatch above is real and **cannot be corrected** — that edition is retained
read-only, so there is no version of this work in which the checker passes and MVC 4 is repaired. This row
therefore has to record an **explicit enforcement boundary**: which paths the checker fails on, which it
reports without failing, and the recorded reason for the difference. Without it the checker is either
wrong or permanently red, and the choice is not W11's to discover while wiring it in.

**Band 1 / 2 / 4, unchanged, and stated as a result rather than carried.** The widened space adds to the
enumeration; the removed serve subtracts more from the exit; the enforcement boundary is **one recorded
decision with a stated scope, not a second audit**, so it is **absorbed** — named here rather than left
silent, the same treatment [§7.2](#72-the-approved-delta-sign-offs) gives its nine additional decisions —
and the residue is within a band whose expected value is two ideal engineer-days. What did change is *what
the band is for*: it is now the repository-wide enumeration, the migration-source correction list, the
checker and its enforcement boundary, and no part of it is a deployment.
[R8](#r8--case-sensitive-path-resolution-on-the-target-platform) carries the risk, and its mitigation is
the checker rather than review.

#### W6 — Project-format conversion and dependency transition

**Scope** is [03 §5](03-modernization-roadmap.md)'s, with the format, the manifests and the pins owned
by [04 §5, §6, §7](04-dotnet8-migration-strategy.md).

**Basis.** **28 package outcomes to apply** (input 13, pin count), each already decided with exactly
one outcome by [04 §8](04-dotnet8-migration-strategy.md) — so this is application of a completed
decision, not analysis. Plus the project-format conversion, the tooling manifests this workstream still
originates, the **converted project's own lockfile** with restore in locked mode, and **one root solution
added** — the four legacy solutions are
**retained**, not collapsed, because the migration source must stay buildable as the reference the port
is validated against, so the repository tracks five solution files afterwards rather than one
[04 §5.6](04-dotnet8-migration-strategy.md).

**Two of the manifests are no longer originated here, and that is the whole of this row's movement.**
[03 §4.2](03-modernization-roadmap.md)'s **gate 4a** now creates `global.json` and the root
`NuGet.config` and proves locked-mode restore on the contracts project, and **W6 consumes gate 4a**. So
this row **inherits** those two files rather than authoring them — while still authoring the root
solution, the tool manifest and the converted project's lockfile, and still **re-verifying** the
inherited pair against a project that did not exist when 4a proved them. Inheriting a file whose
correctness must be re-established for a new consumer is cheaper than writing it and is not free, which
is why the movement is **−0.5 expected and −0.5 high with no change at the low band** rather than the
whole of gate 4a's 4.5.

**What this row does *not* include, and it is the correction that keeps the band honest.**
[03 §5](03-modernization-roadmap.md)'s W6 exit gate deliberately requires **neither a build nor a test
run**, because the *unported* application cannot compile on the target framework at all — the types it
is built on are removed rather than renamed. **W4's gate 4a *is* this workstream's predecessor and W4's
suite is still not part of this gate** — the edge is `4a → W6`, so what W6 consumes from W4 is the
governance bootstrap and not a baseline that runs. What replaces those two conditions is a named piece of work this band must
carry: the **compile diagnostics enumerated as the expected state**, every one mapped to a no-successor
construct. **14** of the **22** blockers fail at compile time, of which **12 apply to the migration
source** — the other two are edition-specific to the two reference editions
[12 §7](12-migration-blockers.md) (input 12, work-item count). No unmapped diagnostic may be left
unexplained and no named construct may be left silent, since a construct that produces no diagnostic
means the conversion did not take effect where it was supposed to. Real compilation and a green suite
are W7's exit gate.

**Band 2.5 / 4.5 / 8.5. High confidence** — every input is enumerated and each outcome is checkable.
**Re-derived from 2.5 / 5 / 9** by the gate-4a transfer above, and itself re-derived from 2 / 4 / 8 when
removing the suite-run gate took work out and enumerating and adjudicating the expected diagnostic set
against 14 named constructs put more back. **This is the only row in the model that falls**, and it falls
because a file moved workstream rather than because any work stopped being necessary.

#### W7 — The ASP.NET Core port

**Scope** is [03 §5](03-modernization-roadmap.md)'s, with every design decision owned by
[05 §2–§9](05-aspnet-core-migration-approach.md) and the file-by-file target owned by
[04 §12](04-dotnet8-migration-strategy.md).

**Basis — decomposed, because a single band over the largest row would hide its structure.** All line
figures are the **sizing metric**; all others are labelled.

| Component | Input | Low | Expected | High |
| --- | --- | ---: | ---: | ---: |
| Composition root and dependency injection, including the **shared application-services seam** placed as the composition root's **first** registration and published with an **allocation table** stating which consumer — the web application, the migration tool, the provisioning tool, the in-process test host — takes which registrations | **10** manual construction sites (site count); **1** seam with **4** consumers ([05 §3.1](05-aspnet-core-migration-approach.md)) | 2.5 | 5 | 8.5 |
| Ordinary application code | **895** non-blank lines (sizing) | 5 | 9 | 15 |
| Authentication rewrite | **382** non-blank lines (sizing) | 6 | 11 | 18 |
| Seed data — a data decision, **not** line porting | **820** non-blank lines (sizing) | 1 | 2 | 4 |
| Views, helpers and view components | **29** views, **5** naming legacy types; **11** bundling sites; **3** child actions becoming **3** view components (file and site counts) | 4.5 | 9 | 15 |
| Static assets and their acquisition | **27** assets in the migration source's four asset groups (file count) | 1 | 2 | 4 |
| Demonstrating the silent-runtime resolutions | **8** of the **22** blockers (work-item count) | 2 | 4 | 7 |
| The **security-header middleware**, the **error action** and the **generic error view** — one application-owned middleware at a fixed pipeline position, covering four response kinds, with a content-security mode key that has no default; `HomeController.Error` with its two attributes, its two feature paths, its three-field view model and its status preservation; and a **layoutless** error view, because the layout queries data an error path cannot assume is available | **1** middleware / **4** response kinds; **1** action / **2** attributes / **2** feature paths; **1** view with `Layout = null` ([05 §2.4, §7.1, §8.3](05-aspnet-core-migration-approach.md)) | 1.5 | 3 | 5 |
| The ledger's **eight further entries** implemented in the ported controllers — the cross-cart `404`, the album binding allow-list with its partial edit, the missing-row check on delete, the four unknown-identifier `404` conversions, the checkout's caught-persistence feedback, **global anti-forgery adoption answering `400` on the five previously unprotected POSTs**, **verb-mismatched POST-only routes answering `405` where the source answers `404`, sign-out included**, and the **cart-migration failure notice**; plus the **status-200 ownership-denial contract**, which returns the shared error view with status 200 rather than a redirect | **8** of the **22** deltas (input 17, entry count); **5** unvalidated POSTs (input 18, census) | 2 | 4 | 6.5 |
| Target-facing half of the suite — the **machinery**: the disposable engine fixture, the in-process host with `ConfigureWebHost` as its **single** override point and a **three-line `ConfigureTestServices` allow-list** installing the logger provider and a command interceptor on **each** of the two `DbContext` types, the run-scoped owner and runtime principals with the ownership registry and orphan sweep, **`DENY`-based** fault injection with the one-shot trigger's `SEQUENCE`, the three-layer timeout model, the lifecycle, isolation and parallelism policy across **9** normal groups and **3** row-specific ones, the destructive-operation controls **demonstrated**, the observer's **target implementation** and `IStoreSetup`'s, the fixture dataset's target loader, the **`--model-overrides` divergence seam**, the two-host topology the continuity rows need, **row 25c's own half-migrated database and its two-host lifecycle**, and the **public deployed-only fixture and test class** authored here for execution at the deployed gate | **12** collection-fixture groups, **1** override point, **8** injected keys, **2** clients, **2** interceptors (input 23, entry count); **3** gates, the registry, the sweep, **3** fault-injection mechanisms and **3** timeout layers (input 24, entry count); the second of **2** implementations and the second of **2** loaders (inputs 25, 27); **9** override files behind the divergence row; **4** continuity rows / **8** cases (case count, input 14) | 12.5 | 22.5 | 40 |
| Target-facing half of the suite — the **browser-driven flow**: the pinned harness driving Chromium over the cart page's script-issued removal request, asserting the request header actually sent, the JSON response handling, the four DOM updates, zero console errors and zero policy-violation reports — on a **separate real-Kestrel host bound to a loopback port**, because the automation cannot reach an in-memory `TestServer` | **1** flow, **1** engine, **1** install step, **1** additional host (input 29, entry count); target-only row **28b** ([05 §12.11](05-aspnet-core-migration-approach.md)) | 2 | 3.5 | 6 |
| Target-facing half of the suite — the **cases**: **144** re-pointed at the target fixture and **98** target-only authored here, including the **checkout token negatives**, the **`405` rows**, row 64's **five** journal-keyed interruption cases and the **five published `provision-admin` executable-contract cases** of row 75 | **144** + **98** of the **242** cases (case count, input 14) | 15.5 | 23.5 | 39.5 |
| `tools/migrate-data` — the artifact the migration gates of [03 §5](03-modernization-roadmap.md) are enforced **by**, built here because five of its six modes are functions of this workstream's output; now also carrying the **`--extract` immutable-artifact handoff**, the **`--scope catalog\|identity` grammar** with its per-scope receipt, the **`--model-overrides` seam** with its nine override files, and the **`dbo.MigrationLoadJournal`** table written **in the same transaction as each group's commit**, with the filesystem receipt derived from it *after* commit and never read to decide a resume | **6** modes, **4** switches, **3** exit codes (input 20, entry count); **13** rows / **32** cases drive it (case count, input 14) | 6 | 11 | 19.5 |
| **W7 total** | | **61.5** | **109.5** | **188** |
| Composition root and dependency injection | **10** manual construction sites (site count) | 2 | 4 | 7 |
| Composition root and dependency injection | **10** manual construction sites (site count) | 2 | 4 | 7 |

**The addition, printed so it can be checked rather than trusted.** The **nine port sub-rows** are
2.5 + 5 + 6 + 1 + 4.5 + 1 + 2 + 1.5 + 2 = **25.5** low,
5 + 9 + 11 + 2 + 9 + 2 + 4 + 3 + 4 = **49** expected
and 8.5 + 15 + 18 + 4 + 15 + 4 + 7 + 5 + 6.5 = **83** high. Adding the four suite-and-artifact sub-rows:
25.5 + 12.5 + 2 + 15.5 + 6 = **61.5**, 49 + 22.5 + 3.5 + 23.5 + 11 = **109.5**, and
83 + 40 + 6 + 39.5 + 19.5 = **188**.

**Ten notes on that decomposition, each of which is a place a naive estimate goes wrong.**

- **The authentication rewrite is 382 non-blank lines (sizing metric), ~18 percent of the migration
  source** — and it is the largest of the **nine port sub-rows** despite that share, because it is the
  one component with no line-for-line successor: its stack is replaced rather than ported. Two rows
  exceed it in this table — the target-facing machinery and the target-facing cases — and neither is
  porting work at all.
  [05 §11.5](05-aspnet-core-migration-approach.md) row 11 narrows it in one specific way that this band
  reflects: what the rewrite **removes** is the *provider-driven* half of the external-login surface —
  the four commented-out provider registrations
  [src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:23-35], the external sign-in cookie, and the
  `ChallengeResult` type [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:394] with its
  `ExecuteResult` override [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:411], for which no
  successor signature exists, together with the six provider-driven actions, four `Views/Account/` views
  and the set-password branch that 05 enumerates with them.
  **The surface itself is not deleted and is not unreachable today**: the
  account-management page invokes the linked-login list at
  [src/MVC5/MvcMusicStore/Views/Account/Manage.cshtml:22], inside the external-logins section at
  [src/MVC5/MvcMusicStore/Views/Account/Manage.cshtml:21-24], and
  [src/MVC5/MvcMusicStore/Views/Account/_ExternalLoginsListPartial.cshtml:3-13] renders a visible
  message to any signed-in user. The list survives the port as the third view component in the row
  below, so this sub-row costs a **narrowing**, not a removal.
- **Three view components are ported, not two.** AAP §0.3.1, §0.3.5 and §0.4.1 fix `GenreMenu`,
  `CartSummary` and `RemoveAccountList` in the target map — each a component class plus a
  `Default.cshtml` — [04 §12](04-dotnet8-migration-strategy.md) carries the map and
  [05 §8.2](05-aspnet-core-migration-approach.md) owns the conversions;
  [03 §5](03-modernization-roadmap.md)'s W7 states the same scope point. The third component is
  `RemoveAccountList`, whose current partial is typed on an Identity 1.0 collection
  [src/MVC5/MvcMusicStore/Views/Account/_RemoveAccountPartial.cshtml:1], so it needs a typed target
  model as well as a conversion. That is the whole of the **+0.5 / +1 / +1** this sub-row carries over an
  estimate built on two components.
- **The seed is 39 percent of the source by the sizing metric and about 2 percent of this row's
  effort** (2 ÷ 102). That asymmetry is the whole point of [08 §4.2](08-technical-debt-register.md)'s
  instruction: 820 non-blank lines of hardcoded catalog rows are bulk data expressed as code, and the
  work is deciding how the data reaches the database, not porting 820 lines.
- **Ordinary application code — 895 non-blank lines — is the genuinely mechanical part**, and it is
  43 percent of the source and about 10 percent of this row (9 ÷ 102). An estimate that scaled from
  total lines would land almost entirely in the wrong place.
- **The 8 silent-runtime blockers get their own sub-row on purpose.** Their exit criterion
  [03 §5](03-modernization-roadmap.md) is *demonstration*, not code review — because their failure mode
  is silence, an assertion that fails when the resolution is absent is the only acceptable evidence.
  That cost belongs to W7, and so does the machinery it runs on: the three target-facing sub-rows below it
  are this workstream's, because [03 §5](03-modernization-roadmap.md) gives W4 only the legacy-facing
  half of the suite.
- **The target-facing half is three sub-rows because its machinery and its cases scale on different
  things, and the case row is derived rather than asserted.** Of the **242** cases, **144** already exist
  as W4's shared contract and are **re-pointed** at the target fixture — a discount, not a rewrite, at
  about **0.06 IED per case** (144 × 0.06 = 8.64, so **8.5 expected**; 0.04 and 0.10 per case at the low and
  high bands give 5.76 and 14.4, so **6** and **14.5**), since the assertions are authored and only the
  fixture, the approved-delta expectations and the diagnostics differ. The other **98 are target-only and
  are authored here** at the same **0.097 / 0.153 / 0.255 IED per case** W4's row uses —
  9.51 / 14.99 / 24.99, so **9.5 / 15 / 25** — spread over the **39 target-only rows' 89 cases** and the
  **9 target-only cases of the 6 mixed rows**. 6 + 9.5 = **15.5 low**, 8.5 + 15 = **23.5 expected**,
  14.5 + 25 = **39.5 high**, and the executions
  reconcile against [05 §12.4](05-aspnet-core-migration-approach.md): W4 runs **144**, the target side is
  **242** — the 144 target halves plus the 98 target-only cases — and 144 + 242 = **386**. Those 242
  target-side executions are 05's **241 `Category=Target`** plus its **single `Category=Deployed`** case.
  **Six of the 242 do not execute at this workstream's exit gate**, and the reason is availability rather
  than allocation: **row 47's single `Category=Deployed` case** needs a deployed host, which does not exist
  at W7's exit, and **row 75's five cases** accept the provisioning tool
  [W12](#w12--administrator-provisioning-tool) builds, which does not exist there either. So **236 execute
  at W7's exit**, **5 at W12's exit** and **1 at the deployed verification's own stage**
  [06 §12.1](06-azure-hosting-recommendations.md) — 236 + 5 + 1 = **242**. **Authoring all 242 remains
  this sub-row's**, because a case is authored once and the charge-once rule puts that charge where the
  authoring happens; what moved is the **execution and the acceptance**, which are charged in W12 and in
  W11's manifest half respectively. **`Category=Sweep` is a trait value and not a fourth category**, so it
  adds nothing to either side of that reconciliation; the sweep class it marks is W4's, and its always-run
  cleanup job is charged in W4's reset sub-row.
- **The two-host replica/restart topology is machinery, and it is here rather than in W4.** Continuity is
  **4 rows and 8 cases** (25, 26, 42, 43 [05 §12.4](05-aspnet-core-migration-approach.md)), all
  target-only, and they need **two hosts sharing the intended stores** rather than a second assertion —
  the cookie, session and anti-forgery continuity claims are false if a single host answers both requests.
  That topology exists only against the ported application, so a legacy-side equivalent is not merely
  unnecessary, it is impossible.
- **The artifact sub-row is a tooling project, not part of the ported application.** It carries the six
  modes and the exit-code contract [05 §5.4, §5.6](05-aspnet-core-migration-approach.md) owns, and the
  **13 rows / 30 cases** of [05 §12.4](05-aspnet-core-migration-approach.md) that **drive it** rather
  than a pre-populated database — which is what makes the migration testable at all, since a
  pre-populated target proves nothing about the migration that filled it. It is charged here because
  [03 §5](03-modernization-roadmap.md)'s W7 scope statement places it here and because five of its six
  modes are functions of the ported model and the two migration sets this workstream produces. **Its
  increment this round is not a seventh mode**: the mode count and the exit-code contract are unchanged,
  and what arrived is **`--extract`'s immutable-artifact handoff** — per-manifest byte lengths, digests
  and a load receipt the `load` side refuses to proceed without — and the **`--model-overrides` seam**
  with its **nine** override files, which is what makes the divergence row assertable instead of
  hypothetical. **+1.5 / +2.5 / +4** on a sub-row that was 3.5 / 6.5 / 12. Its
  **invocation** from the release path is W11's manifest half and its **principal** is
  [06 §6.2](06-azure-hosting-recommendations.md)'s; neither is charged again here.
- **The error-handling port sub-row is application code the earlier derivations under-specified, and its
  third component is the reason it moves.** The
  **security-header middleware** is application-owned rather than platform-supplied, sits at a stated
  pipeline position, must produce its set on **four** response kinds including a static file and a
  re-executed error response, and reads a content-security mode key that **has no default**, so the host
  fails to start if nothing supplies it. **`HomeController.Error`** is the successor to a removed
  attribute-and-model pair rather than a ported action: it is reached by **two** feature paths — the
  exception handler and status-code re-execution — carries `[AllowAnonymous]` and a **ledgered**
  `[IgnoreAntiforgeryToken]`, preserves the status code, and must answer a direct request safely. What is
  new is the **layoutless generic error view**: `Layout = null` is load-bearing rather than stylistic,
  because the shared layout renders the genre menu and the cart summary and both query data an error path
  cannot assume is reachable — an error view inheriting the layout can therefore fail while rendering the
  failure. **3 expected IED** for the three together, up from 2 for the pair.
- **The ledger's further entries are implementation, not paperwork, and there are now eight of them.**
  Entries 15 to 22 of
  [05 §11.5](05-aspnet-core-migration-approach.md) each change a caller-visible outcome the ported
  controllers have to produce — a cross-cart `404`, an album binding allow-list with the partial edit it
  implies, a missing-row check on delete, four unknown-identifier `404` conversions where the source
  answers `500`, feedback on a caught persistence failure at checkout, **`400` on the five POSTs the
  source accepts and writes without a token** (input 18), **`405` on verb-mismatched POST-only routes
  where the source answers `404`, sign-out included**, and the **cart-migration failure notice** that
  must not block a sign-in. The **status-200 ownership-denial contract** rides on the same sub-row: the
  denial returns the shared error view **with status 200** rather than a redirect or a `403`, which is a
  shape a ported controller has to be written to produce deliberately. **4 expected IED**, up from 2, and
  the cases that assert them are inside the case row rather than here.
- **Two corrected oracles cost nothing and are stated anyway, so the case count is honest.** The cart
  count is a **row** count rather than a quantity sum, and the genre toggle's `href` remains
  `/Store/Store`, **preserved bug-for-bug**. Neither moves a band; both change what a case asserts, and
  a re-derivation that quietly dropped them would leave the count of cases unexplained.

**Band 61.5 / 109.5 / 188. Medium confidence, unchanged** — the scope is enumerated to the file, the
blocker, the case, the artifact mode, the fixture control and now the engine, and the variance is
execution rather than discovery. The
sub-row with the widest relative variance is the artifact, because it must be built to a **contract with
refusal semantics** rather than merely made to work. **Re-derived from 57.5 / 102 / 175.5**, and the
arithmetic of the movement is printed so it can be checked. Five components move and nothing else does:

- the **composition sub-row** gains the **shared application-services seam** — the composition root's
  first registration, with a published allocation table naming which of four consumers takes which
  registrations, so that the migration tool and the provisioning tool compose the *same* services as the
  web application rather than approximations of them. 2 / 4 / 7 becomes 2.5 / 5 / 8.5 —
  **+0.5 / +1 / +1.5**;
- the **machinery** row absorbs the instrumentation hook made exhaustive — one override point and a
  three-line service allow-list installing an interceptor on **each** of the two `DbContext` types,
  because one interceptor covers one context — **`DENY`-based** fault injection, since a `REVOKE` cannot
  defeat a grant the principal derives from a role, and a non-transactional sequence for the one-shot
  transient fault; the **three row-specific collection groups** beside the nine normal ones, including
  **row 25c's own half-migrated database and its two-host lifecycle**; and the **public deployed-only
  fixture and test class**, authored here and executed at the deployed gate. 11 / 19.5 / 35 becomes
  12.5 / 22.5 / 40 — **+1.5 / +3 / +5**;
- the **browser-driven flow** gains its **separate real-Kestrel host on a loopback port**, which is a
  second host lifecycle rather than a setting, so 1.5 / 2.5 / 4.5 becomes 2 / 3.5 / 6 —
  **+0.5 / +1 / +1.5**;
- the **case** row follows the matrix from 144 + 95 to 144 + 98 at unchanged rates — the three added cases
  are row 24's fifth and row 64's fourth and fifth, all target-only — so 15 / 23 / 38.5 becomes
  15.5 / 23.5 / 39.5 — **+0.5 / +0.5 / +1**;
- and the **artifact** row gains the **`--scope` grammar** with its per-scope receipt and the
  **`dbo.MigrationLoadJournal`** written in the same transaction as each group's commit, with the
  filesystem receipt derived from the journal *after* commit and never read to decide a resume. A journal
  inside the transaction is what makes an interruption resumable rather than merely detectable, and it is
  what row 64's five cases assert. 5 / 9 / 16 becomes 6 / 11 / 19.5 — **+1 / +2 / +3.5**.

0.5 + 1.5 + 0.5 + 0.5 + 1 = **+4**, 1 + 3 + 1 + 0.5 + 2 = **+7.5**, and
1.5 + 5 + 1.5 + 1 + 3.5 = **+12.5**, so
57.5 + 4 = **61.5**, 102 + 7.5 = **109.5** and 175.5 + 12.5 = **188**. This row was previously
re-derived from 48.5 / 87.5 / 151.5, from 37.5 / 68.5 / 118.5, from 25.5 / 48 / 82, and before that from
21 / 40 / 69 by the third
view component and by the target-facing half arriving from W4 under
[03 §5](03-modernization-roadmap.md)'s split.

**The manual visual review is not inside this band, and it closes this workstream's exit gate.**
[03 §5](03-modernization-roadmap.md) requires the review against W4's captured baseline at W7's exit,
so W7 has not exited until it is signed off. It is sized as a non-code task in
[§7.1](#71-the-manual-visual-review) — 3.5 expected IED for the review half — and
[section 8.2](#82-concurrency-permitted-by-the-graph) places it in its own set **after** this
workstream, because it reviews rendering this workstream produces. **The browser-driven sub-row above is
not that review and does not reduce it**: it is functional evidence on one engine, and
[§7.1](#71-the-manual-visual-review) states which engines get which kind of evidence.

#### W8 — Identity migration tooling, rehearsed against a copy
**Scope** is [03 §5](03-modernization-roadmap.md)'s, exiting on reconciliation passing; the migration
design is owned by [05 §5.5](05-aspnet-core-migration-approach.md).

**Basis.** Creating the target tables fresh and populating them, then reconciling. The cost drivers are
not volume — the shipped store is small — but **correctness obligations**: collision detection on the
normalized columns before writing, defined origins for columns that have no source value, verification
that the shipped hashes validate, and reconciliation of account, role and assignment counts with the
administrator's membership asserted specifically.

**This workstream now owns a scoped load rather than a share of one**, and that is a scope clarification
rather than a band movement. `load`, `reconcile` and `dry-run` each take a **mandatory
`--scope catalog|identity`** [05 §5.7](05-aspnet-core-migration-approach.md), so this workstream runs
`--scope identity` throughout, receives its **own** receipt and its **own** reconciliation verdict, and is
**separately reversible** from [W9](#w9--domain-data-migration-tooling-rehearsed-against-a-copy) rather than entangled with it. That is
what makes the two workstreams' gates independently adjudicable, and it is the reason the sequencing in
[section 8.2](#82-concurrency-permitted-by-the-graph) can place them in different sets at all.

**Band 4 / 8 / 16, unchanged by the re-derivation, and the reason is a placement rather than an
oversight.** [03 §5](03-modernization-roadmap.md) enforces this workstream's diff and reconciliation
gates through `tools/migrate-data`, and that artifact — including the `--scope` grammar, the per-scope
receipt and the journal behind them — is **built and charged once, in
[W7](#w7--the-aspnet-core-port)** — 11 expected IED there. What this workstream carries is **running**
it under its own scope and adjudicating its verdicts, which the band already contained as reconciliation
work; charging any part of the artifact here as well would double-count it. **Low confidence**, for a reason stated
precisely: this workstream's *source schema* is the one [12 §5](12-migration-blockers.md) qualifies as
**evidence rather than proof**. The high band is what this costs if **A10** is false and the hashes do not
validate. [R5](#r5--identity-migration-rollback) carries it.

#### W9 — Domain data migration tooling, rehearsed against a copy
**Scope** is [03 §5](03-modernization-roadmap.md)'s, entered through what that document calls the hard
gate: the generated-schema diff must pass before any data is loaded.

**Basis.** Six entities, loaded in dependency order by the same executor's `load-domain` sub-command
[05 §5.6](05-aspnet-core-migration-approach.md), whose release-time placement is
[06 §6.10](06-azure-hosting-recommendations.md)'s — then reconciled to that section's
standard: keyed sets and per-row digests, with per-table row counts and per-order financial totals
necessary and **not** sufficient. **The rehearsal inside this band is domain-only**: it exercises
`load-domain` and the catalog store's reconcile against a restored copy of the catalog source and
invokes `migrate-identity` at no point, and the **integrated both-store rehearsal is a gated task of
W13's pre-window** rather than work content here — [03 §4.2](03-modernization-roadmap.md) owns that
placement and states why an exit gate requiring a capability a later workstream builds was not
executable. **Building and proving the manifest mechanism is work content in this band; producing the production
manifest is not.** [03 §5](03-modernization-roadmap.md) has this workstream exit on tool and rehearsal
readiness, and the cutover run through
[06 §6.10](06-azure-hosting-recommendations.md)'s `seal-manifest` produce the artifact that is actually
acted on — so what this band carries is the `seal-manifest` capability, its destination and its
rehearsal, not a second execution. 06's contract makes that artifact a **verifiable** one rather than a
list, and its five bound fields are [06 §11.4](06-azure-hosting-recommendations.md)'s to state: the
exact imported ID set bound to the **per-store source fingerprints**, the **durable run id**, the
**contributing committed table units** and the **per-store extraction cutoffs**, integrity-protected and
written to a store whose immutability the store itself enforces. **The unit of that provenance is the
committed table unit and not a batch identifier** — 06 §11.4 and
[05 §5.6](05-aspnet-core-migration-approach.md) bind it that way and reject batch provenance outright,
for a reason those sections state and this one does not restate. Most of what that costs is already
inside this band for another reason — the run id and the per-store source descriptors, the canonical
serialization and the keyed reconciliation sets are all required by the executor's own
checkpoint and reconciliation contracts [05 §5.6](05-aspnet-core-migration-approach.md) — so the
incremental content is the manifest's own binding and the write-once destination's provisioning. That
moves **the high band only, from 15 to 16** — one ideal engineer-day, the only band change 06's
strengthened manifest contract causes anywhere in this model — because the low and expected cases assume
the destination and its immutability policy exist to be pointed at ([W10](#w10--hosting-provisioning-and-platform-configuration)
provisions them) and the high case is the one where they do not. The volume is modest and the **gate** is
the cost: a diff between a generated migration and the real schema, iterated until it passes, against a
schema that W3 must first extract because the migration source **ships no schema script** and neither
committed copy of the other edition's script is usable [12 §5](12-migration-blockers.md).

**Four contract additions are inside this band, and they are named so the band is not read as covering
less than it does**: the parent run descriptor with its mandatory continuation input across every
sub-command, the split of the catalog set into a pre-load stage and a post-transform stage, the
merge lineage the cleanup set is enumerated from, and the run-closure step that retires the verification
key last — all four owned by [05 §5.6](05-aspnet-core-migration-approach.md) and
[06 §11.4](06-azure-hosting-recommendations.md). Each is an increment to a tool this row already builds
rather than a new deliverable: one table and one flag, a second migration file, columns in an artifact
already written, and a closing sequence of already-specified operations. The band does not move because
its dominant cost is stated above as the diff gate iterated until it passes, not the executor's surface
area — but if the extraction returns a `CartId` facet that forces the bound of
[05 §4.3](05-aspnet-core-migration-approach.md) to be re-approved, that is a **gate reopening** and this
row is re-estimated rather than stretched.

**Band 4 / 8 / 15. Low confidence.** [R4](#r4--domain-data-migration-rollback) carries the risk,
including the two residuals that survive the drain
[05 §5.1](05-aspnet-core-migration-approach.md) step 8 selects for rows written between extraction and
cutover — rehearsal and verification obligations inside this band, not open design questions.

#### W10 — Hosting provisioning and platform configuration

**Scope** and every mechanism are owned by [06 §6–§9](06-azure-hosting-recommendations.md): the
provisioning order, the DDL principal separated from the runtime identity, the persisted key ring,
session over the distributed cache, transport enforcement and the observability placement.

**Basis.** Environment provisioning, identity and secret wiring, the ordered creation of four schema
owners, transport and header configuration, and the observability wiring that is **net-new capability**
rather than migration — the repository has none of it today
[08 §7.1](08-technical-debt-register.md). Assumption **A8** applies.

**Band 5 / 9 / 16, unchanged by the re-derivation.** The suite's **test** engine is not provisioned here
and is not charged here: the target-facing fixture creates and destroys its own disposable instance per
collection [05 §12.7](05-aspnet-core-migration-approach.md), which is why that cost sits in
[W7](#w7--the-aspnet-core-port)'s machinery sub-row. This workstream provisions the **deployment**
environment, and the two are separate resources with separate lifetimes and separate principals.
**Medium confidence** — the steps are enumerated by 06; the variance is environment-specific.

#### W11 — CI provider selection, then pipeline authoring

**Scope** is [03 §5](03-modernization-roadmap.md)'s, strictly in that order: a provider decision, then
a manifest.

**Basis — net-new, with an approval inside it.** The repository has **no** pipeline definition, publish
profile or container manifest [08 §7.2](08-technical-debt-register.md), so nothing is migrated. The
band covers the provider decision (an approval, not engineering), then build, test, publish and the
deployment-time migration step that [06 §6.2](06-azure-hosting-recommendations.md) requires be run
under a principal distinct from the runtime identity — the step that invokes `tools/migrate-data`, whose
own cost is charged once in [W7](#w7--the-aspnet-core-port).

**The test stage is not one stage.** Input 21 puts
**two agents** in the pipeline rather than one — a Windows agent that runs the legacy application for the
`Category=Baseline` half and a Linux agent with a provisioned engine for the `Category=Target` half —
with the **redacted baseline record handed from the first to the second**, which is a pipeline artifact
with a retention rule rather than a step. The **architecture of those stages is
[06 §12.1](06-azure-hosting-recommendations.md)'s** and is not restated or re-decided here; what this row
carries is the effort of expressing them in whichever provider W1 selects.

**The provider decision is no longer only a decision, and that is the first half of this row's
re-derivation.** [06 §6.4](06-azure-hosting-recommendations.md) has **removed the SQL-authentication
fallback** from the cache-table step, so a provider that cannot furnish a **credential-free federated or
workload identity** to Azure SQL cannot execute the release path at all. That converts a preference into
a **blocking precondition settled at 03's provider-selection gate**: the candidate providers have to be
checked against that capability, and one that fails it is rejected rather than accommodated. The
selection half therefore moves from 1 / 2 / 3 to **1.5 / 3 / 5**, and the removed fallback takes a
branch **out** of the manifest half at the high band, where an exception path would otherwise have had to
be expressed.

**Three obligations landed on the manifest half, and they are the second half.** None is a decision this
document takes; each is a stage the manifest has to express:

- **The thirteen blocking checks now carry literal evidence bounds** (input 28) — a per-request timeout,
  named poll counts and intervals, telemetry-ingestion waits, **two** rounds and a single total evidence
  deadline [06 §12.1](06-azure-hosting-recommendations.md). The previous derivation costed a gate that
  *stated* a bounded attempt budget; this one costs a gate whose bounds are **written down and therefore
  implementable**, which is authoring rather than design. The stage must still provision **two instances
  or replicas with affinity off**, have its client discard the platform's affinity cookie, **observe which
  instance served a response** through request telemetry rather than by asking for one, trigger a restart
  from the stage, and fail the release on exhaustion. **+1 / +1.5 / +3.**
- **Customer-managed key destruction now requires dedicated scope**
  [06 §9.5](06-azure-hosting-recommendations.md): crypto-shredding is permitted only against a
  **dedicated artifact storage scope** protected by a **dedicated key** and a **dedicated key version**,
  with immediate enumeration of what that scope holds and the security owner's authorization recorded.
  Where the security owner requires CMK, the manifest and the provisioning it drives must therefore
  create and maintain a separate scope and key version rather than reuse an existing one. **+0.5 / +1 /
  +1.5.**
- **The target-test stage acquires a browser prerequisite** (input 29): the pinned harness's Chromium
  install, expressed as a step on the Linux agent and pinned with the library
  [04 §7.7](04-dotnet8-migration-strategy.md). **+0.5 / +0.5 / +1.**

**Three further obligations landed on the manifest half this round, and each is authoring rather than
intention — which is the whole of why the band moves.**

- **Instance observation became a published join protocol** rather than a stated capability
  [06 §12.1](06-azure-hosting-recommendations.md). The stage constructs a **`traceparent` per request** so
  the response it received and the telemetry it later reads are the *same* request; it runs a **published
  query** joining that trace to the role-instance dimension; it **disables sampling** on the non-traffic
  target and then **proves it off** with an `itemCount` assertion, because a sampled-away request looks
  exactly like a request served by one instance; and it validates the instance identity against the
  platform's **own inventory**, so "two instances" is a fact read from the platform rather than inferred
  from two different strings. A query, an assertion and a platform read are three authored things.
  **+1 / +2 / +3.5.**
- **The thirteen checks acquired an executor-and-stage mapping** (input 28): **2** of them are the suite's
  own `Category=Deployed` binding, invoked by the stage — which is where **row 47** executes, since no
  deployed host exists at [W7](#w7--the-aspnet-core-port)'s exit — and the other **11** are the stage's
  own steps. Expressing that split, rather than leaving a check without a named executor, is a manifest
  obligation. **+0.5 / +1 / +1.5.**
- **Artifact publication became conditional and allow-listed.** The stage publishes **three** artifacts
  and no others, and the baseline publication policy of input 26 is a **pipeline condition** rather than a
  convention: a failed or incomplete capture publishes a `baseline-capture-diagnostic` and the
  `baseline-record` is published only when every `Category=Baseline` case passed. A condition that has to
  refuse to publish is more work than one that always publishes. **+0.5 / +1 / +1.5.**

**Band 11 / 19.5 / 35.5, and it splits 1.5 / 3 / 5 for the provider decision and 9.5 / 16.5 / 30.5 for the
manifest** — the split [section 8.2](#82-concurrency-permitted-by-the-graph) needs, because the two
halves have
different entry gates. **Medium confidence** — the shape is standard; the provider is unchosen, and
[04 §6](04-dotnet8-migration-strategy.md)'s locked-mode restore adds a step that must actually be made
to fail correctly. **Re-derived from 9 / 15.5 / 29**: the manifest half moves from 7.5 / 12.5 / 24 to
9.5 / 16.5 / 30.5 by the three obligations immediately above —
7.5 + 1 + 0.5 + 0.5 = **9.5**, 12.5 + 2 + 1 + 1 = **16.5** and 24 + 3.5 + 1.5 + 1.5 = **30.5** — and the
provider half does **not** move, so the workstream is 1.5 + 9.5 = **11**, 3 + 16.5 = **19.5** and
5 + 30.5 = **35.5**. It was previously re-derived from 6.5 / 11.5 / 22, where the manifest half had moved
from 5.5 / 9.5 / 19 by the evidence bounds, the dedicated key-destruction scope and the browser install,
less the removed fallback branch at the high band, and the provider half from 1 / 2 / 3 by the
federated-identity precondition; and before that from 4.5 / 8 / 15 and 4 / 7 / 13 for
input 21's second agent and the handoff artifact between them.

#### W12 — Administrator provisioning tool

**Scope** is [03 §5](03-modernization-roadmap.md)'s, exiting with all five properties of
[05 §10.2](05-aspnet-core-migration-approach.md) demonstrated.

**Basis: input 30 for the tool itself, input 23 for its added tests.** The two are disjoint — input 30
counts behaviour to implement, input 23 counts the 5 operator-host tests that have no MVC 5 baseline — and
together they cover this row rather than only its test half. A small console project, where the effort is in
input 30's **5 required properties** rather than the code: a secret
channel that keeps the credential out of process listings and shell history, hashing through the
framework's own user manager rather than direct SQL, **convergence checked per operation over all four
operations** — role, user, credential and membership — so a prior partial run is repaired rather than
skipped, an audit record with no secret in it, and exclusion from the deployed web
application. Input 30's other two dimensions are what make the band Medium rather than Low: the **revoke
mode's 2 operations across 3 branches** are a second code path with its own record cardinality, and the
`PROV-6001`'s closed vocabulary of **17 outcome values** has to be produced and asserted
exactly, not approximated.

> **What the corrected outcome count does and does not add, because most of it was already inside this
> band.** An earlier version of input 30 put the vocabulary at eight values — three credential outcomes
> and five failure outcomes — which undercounted [09 §6.8.1](09-security-assessment.md)'s closed set by
> nine. The correction is a completeness fix to the *input row*, and it is mostly **not** new work: the
> eleven non-failure values partition across the four operations as `2 + 2 + 3 + 4`, and producing them
> per operation is exactly the "convergence checked per operation over all four operations" this basis
> already prices. Enumerating them names the work rather than enlarging it.
>
> **One value is genuinely new scope, and it is priced.** The sixth failure outcome,
> `Failed_ArgumentRejected`, arrives with [09 §6.8.1](09-security-assessment.md)'s rejection contract,
> which did not exist when this band was first judged. It is more than one enum arm: it requires a
> **fixed validation order**, so that which field is reported is not an implementation choice; **bounded
> sentinels** for the actor and target on a rejection, where the supplied value must never be echoed; an
> exact **single-record cardinality** per rejection branch; and negative tests over control characters and
> line breaks. That is the same class of work as
> [04 §12.4](04-dotnet8-migration-strategy.md)'s refusal enum, and it lands here because the tool is
> where it executes.
>
> **Band raised by this correction from 3 / 5 / 9.5 to 3 / 5.5 / 10.5 — an intermediate figure, which the
> two later obligations below then move again to the 3.5 / 7.5 / 12.5 this row carries.** Expected moves by **0.5** for the rejection
> contract; high by **1**, because its characteristic failure is discovering mid-implementation that a
> sentinel collides with a legitimate value and the closed field set must be reopened. **Low does not
> move** — it assumes the contract lands as the thin pre-host validation layer
> [04 §12.4](04-dotnet8-migration-strategy.md) specifies. The eight previously unenumerated non-failure
> values contribute **nothing** to the increase, for the reason above: pricing them here would
> double-count the four operations that produce them.

One property carries a platform obligation rather than a code one:
[06 §9.5](06-azure-hosting-recommendations.md) establishes that a standalone console process is **not**
collected by the application's telemetry path, so the sanctioned execution path is a release-pipeline
job whose output is a retained log artifact **exported into that section's audit store of record** — row 5
of its producer matrix. The release step asserts the **exact record set** rather than the presence of
output: four `PROV-6001` records on a provisioning run, one per operation; on a revoke, **branch-sensitive
rather than a single number** — one where the named account does not resolve and the run stops at the
`user` operation, two in each of the other branches ([09 §6.8.1](09-security-assessment.md)); plus the
paired `AUTHZ-3001` where a membership is actually added or `AUTHZ-3002` where one is removed. It then
confirms the artifact present in the audit store, and fails on a non-zero exit. That is inside this band
and is why the row depends on W11.

**What the credential operation converges on, because the alternative reading would enlarge this row and
change the product's behaviour.** [06 §12.1](06-azure-hosting-recommendations.md) owns the invocation
policy and [09 §6.8.1](09-security-assessment.md) the outcome vocabulary: the operation records
**`Created`** where the account is absent or exists with no password, **`AlreadyPresent_NotRotated`** where
it exists with one and the run did not request a rotation, and **`Rotated`** where the run passed
`--rotate-credential`. **Ordinary pipeline releases do not pass that flag**; the two occasions that do are
the published-credential repair and a post-incident rotation, and
[03 §5 W12](03-modernization-roadmap.md) condition 1 is where the three outcomes are gated. This row is
therefore **not** estimated against a command that rewrites the credential on every run — that reading
would add a mutation AAP §0.3.2's idempotence requirement never asks for, rotate the administrator's
password on the release cadence, and sign its sessions out unpredictably. The estimation consequence is
small and worth stating: the operation is a branch and a comparison rather than an unconditional write, so
the band carries **three outcome paths and the flag that selects between them**, plus the no-flag re-run
condition 1 requires as evidence that the second run changed nothing.

**Three additions since the band was set, all inside it.** The credential operation of
[05 §10.2](05-aspnet-core-migration-approach.md) property 3 is a fourth branch on a code path that
already resolves the same `UserManager` — a create-or-add-password call, a rotation call reached only
under the flag, and a published-value refusal — and the
revoke mode of property 3a is a **mode of that same provisioning verb** over the same two managers, with
the same `--actor`, record and exit-code contract. The third is the **guarded catalog seed**:
[04 §12.6](04-dotnet8-migration-strategy.md) settles that
[05 §5.4](05-aspnet-core-migration-approach.md)'s opt-in seed command executes as the **second verb
(`seed-catalog`) of this same project** rather than as a fourth project, so what this row acquires is one more entry point over a
host it is already composing — the configuration surface, the exit-code convention and the single
structured logging provider are this row's regardless of how many verbs sit on them. **The seed's own
cost is not here and is not double-counted:** the routine and the policy behind its three checks are sized
in **W7**'s "seed data" sub-row, and this row adds the verb that invokes them. None of the three adds a
package or a platform obligation, and the **6 IED** high case of the tool-and-wiring band below already
carries the secret-channel and
audit work that dominates it. That band is therefore unchanged rather than silently stretched, and
this paragraph is the statement of why — [03 §5 W12](03-modernization-roadmap.md) conditions 1, 1a, 3 and
3a gate the first two, and [04 §12.6](04-dotnet8-migration-strategy.md)'s two required assertions gate the
third.

**And the seed verb adds no dependency edge either**, which is worth stating because a seed command sitting
in a late workstream would be a problem if anything earlier needed a seeded database. Nothing does: W4's
target-side fixture **loads a fixture dataset** carrying the same catalog rows
([05 §12.3](05-aspnet-core-migration-approach.md)) rather than invoking the seed, and W8's and W9's
rehearsals run against **restored copies** of real data. The verb's first use is its own acceptance.

**One of them does add a dependency, and it is a sequencing one rather than a cost.** The credential
operation is *accepted* against the account W8's Identity rehearsal neutralized, so
[03 §5 W12](03-modernization-roadmap.md) puts **W8 in this row's entry gate** and retains W8's rehearsal
copy until this row exits. Nothing here gets larger — the account and the copy are W8's output, and this
row runs one command against them — but the row can no longer start beside W8, which is why
[§8.2](#82-concurrency-permitted-by-the-graph) places it in its own set and
[§8.3](#83-the-critical-path-and-what-to-do-first-if-the-goal-is-to-narrow-the-estimate) counts its
expected IED on the chain — 7.5 of them, once the tests, the deployed census and the fourth gated
invocation below are included.

**And one obligation inside the band is a pipeline edit rather than code.**
[03 §5 W12](03-modernization-roadmap.md) condition 6 requires the invocation to be **wired into the release
path at a fixed point** — after an environment's Identity load, before its traffic and its health
verification — in every environment that has an administrator, rather than left as a command an operator
remembers. That is one job definition in the manifest W11 authors, exercised once in the rehearsal
environment, and it sits inside this band's expected case. Condition 6 also fixes *how* it is wired:
**without a rotation flag**, so the stage is unconditional in every environment while the credential
operation converges on `AlreadyPresent_NotRotated` in the steady state.

**One addition is outside the original band, and it is the reason this row is rebanded.**
[04 §12.4](04-dotnet8-migration-strategy.md) requires **5 operator-host tests** of the built executable
(input 23, test count) — a hostile working directory changing nothing, every password-bearing argument
form refused with the key and not the value in the message, the repair path completing in that host, the
dispatcher admitting exactly the documented command lines and refusing every other class, and the
credential arriving on its named environment channel while appearing in no captured output. A sixth
assertion in the same set, the lifetime spelling, is discharged by the Release solution build and costs
nothing here. **These are not rows of [05 §12.4](05-aspnet-core-migration-approach.md)** — a console
process has no MVC 5 baseline — so
[§4.1](#41-the-estimation-basis-every-input-with-its-method)'s inclusion rule estimates them in this row
rather than in W4, and [03 §5 W12](03-modernization-roadmap.md)'s gate demonstrates them here. The
increment is **1.5 / 2.5 / 3.5 IED**, and it has two parts because only one of them is per-test: the five
assertions at W4's target-contract rate of **0.15 / 0.22 / 0.4** each, `= 0.75 / 1.1 / 2`, plus a
**process-level harness at 0.5 / 1 / 1.5** which this row is the only consumer of — launching the
published executable, capturing both its streams, setting a scoped environment variable and running it in a
temporary working directory are none of them things W4's in-process fixtures do. Summed and rounded under
[§4.2.1](#421-the-rounding-rule-stated-once): `1.25 / 2.1 / 3.5` → **1.5 / 2.5 / 3.5**. The expected figure
is 2.5 rather than 2 for exactly the reason that section gives: 2.1 rounds **up**, and a derivation that
took it down would give this document two rounding conventions and therefore no reproducible total.

**And the four coverage rows this gate executes cost nothing here**, because their authoring is W4's.
[03 §4.3](03-modernization-roadmap.md)'s map assigns rows 24, 64, **75b** and 76 to this workstream's
acceptance — the seeding guard through the `seed-catalog` verb, the password-policy refusal, the `b` half of
row 75, one of the **two** rows the map splits across two gates, and the **four-invocation** process census
— and running an already-authored row at the gate that owns its runtime is a gate condition rather
than a second cost, exactly as it is for W7, W8, W9 and W10.

**Two obligations do move this band, and 5.5 no longer holds.** Both arrive from
[03 §5 W12](03-modernization-roadmap.md) after the rejection contract had already been priced, and the
question this paragraph answers is the one asked directly: does the expected figure of 5.5 survive them.
It does not.

- **Exit condition 7 is a deployed twelve-class security-event census, and it is not a re-run of anything.**
  The **twelve** application event classes that
  [W10](#w10--hosting-provisioning-and-platform-configuration) no longer demonstrates are demonstrated here,
  **against a deployed environment**, driven from **the named fixture population**
  [03 §5 W12](03-modernization-roadmap.md) fixes — the seeded non-production catalog this row's own
  `seed-catalog` verb loads, plus **two synthetic accounts, one administrator and one ordinary**, both
  created through the sanctioned paths — driven through sign-in, lockout, registration, password change,
  authorization denial, the three administration outcomes and the two order outcomes, and then **removed by
  the deletion operation [W16](#w16--personal-data-governance) stage 2 proved**, with the removal asserted
  rather than assumed. The thirteenth class is not here: it is the canary W10's condition 7 already used to
  prove the sink. This is the only workstream with an executable that can drive a deployed environment from
  outside it, which is why the census lands here, but the driving, the collection assertions and the
  asserted cleanup are new work in this row rather than a repetition of W7's in-suite emission tests.
  **+0.5 / +1 / +1.5.**
- **The tool's invocations are four separately gated process runs, not three.**
  [03 §5 W12](03-modernization-roadmap.md)'s operator-process coverage requires the built executable driven **as a
  process, through its real entry point, four times against one store**, with each run's outcome asserted
  individually: **create** against an absent account, no flag; **repair without rotating**, against that
  account with its membership deliberately removed first, asserting the stored hash **byte-identical** and
  the membership repaired on the same run; **explicit rotation**, with the flag, against the account W8's
  rehearsal neutralized — the run that also produces the recoverability half and the session-rejection assertion;
  and the **published value refused**, with the flag, asserting a non-zero exit and no change of any kind.
  Conditions 1 and 1a of that gate are where they are asserted — runs 1 to 3 under condition 1, run 4 under
  condition 1a. The fourth is the one a three-run reading drops, and runs 2 and 3 fail **different** wrong
  designs, so no three of the four imply the remaining one. **+0 / +0.5 / +0.5.**

**Band 3.5 / 7.5 / 12.5. Medium confidence**, and derived as four components so the move is checkable:
`1.5 / 3 / 6` for the tool and its wiring, plus `0 / 0.5 / 1` for the rejection contract, plus
`0.5 / 1.5 / 2` for exit condition 7's deployed census **and** the fourth separately gated invocation,
plus the `1.5 / 2.5 / 3.5` of input 23 above. `1.5 + 0 + 0.5 + 1.5 = 3.5`, `3 + 0.5 + 1.5 + 2.5 = 7.5`,
`6 + 1 + 2 + 3.5 = 12.5`.

**The third component is the sum of both obligations above, and it is stated that way because an earlier
form of it was not.** It carried only the census — `0.5 / 1 / 1.5` — while its label claimed the invocation
census as well, so the `+0 / +0.5 / +0.5` priced for the fourth run reached this row's prose and never
reached its band. Added column by column: `0.5 + 0 = 0.5`, `1 + 0.5 = 1.5`, `1.5 + 0.5 = 2`. That is the
whole of the correction, and every figure elsewhere that quotes this row moves with it rather than in some
of its places — which is the failure
[§6.1.1](#611-the-walk-from-the-previously-published-total) records for the rejection-contract increment, in
this same row, one pass earlier.

**Stated against the two figures this row has previously carried**, because both appear in earlier
reconciliation records: the band was 3 / 5 / 9.5 before the rejection contract, 3 / 5.5 / 10.5 with it, and
it is **3.5 / 7.5 / 12.5** now. The move from 5.5 to 7.5 has **three** parts, none of them a re-judgement of
work already priced: **0.5** is input 23's operator-host increment re-derived under
[§4.2.1](#421-the-rounding-rule-stated-once), **1** is exit condition 7's deployed twelve-class census, and
**0.5** is the fourth separately gated invocation.

#### W13 — Cutover

**Scope** is [03 §5](03-modernization-roadmap.md)'s; the runbook is owned by
[06 §11](06-azure-hosting-recommendations.md) and the approach and its accepted losses by
[05 §11](05-aspnet-core-migration-approach.md). **Neither is reopened here.**

**Basis.** Executing a runbook, with the rollback position confirmed beforehand: drain and record the
final write cutoff, provision, **run the production data movement** — `diff-schema`, `load-domain`,
`migrate-identity`, `reconcile`, `seal-manifest` in [06 §6.10](06-azure-hosting-recommendations.md)'s
order — verify, then admit traffic on the recorded go decision. The production run sits here because
[03 §5](03-modernization-roadmap.md) has W8 and W9 exit on **readiness** and this workstream perform the
run itself; **that placement does not move this band**, because the executor is authored, iterated and
rehearsed inside W8's and W9's bands, and what remains here is invoking it and reading its gates. The
band covers rehearsal as well as execution, because a runbook first executed in production is not a
runbook. Six workstreams must have exited before this begins, which is why
[section 8](#8-sequencing) treats it as the convergence point rather than a step.

**One named task entered this workstream's pre-window, and the band is re-examined against it rather
than assumed to absorb it.** [03 §4.2](03-modernization-roadmap.md) places the **integrated both-store
rehearsal** — both `load-domain` and `migrate-identity`, in the release's own order, against restored
copies of both source stores, with the combined reconciliation — as a **gated task of this
workstream's pre-window**, entered on W8 and W9 having both exited and satisfying **no exit condition
of this workstream** [03 §5](03-modernization-roadmap.md) W13. **It is not new content in the model,
and that is why the band holds rather than because the band is generous.** It is the same invocation
this band already covers twice — once as the rehearsal the sentence above insists on, once as the
production run — over two capabilities that both workstreams' exit gates have already built and
already rehearsed against the store each owns. What moved is *where* the both-store scope sits: it
left W9's mandatory rehearsal, which narrowed to domain-only, and arrived here. **The one thing
genuinely new is the possibility that running the two together reveals an interaction neither
single-store rehearsal could**, which would mean iterating the integrated run; that is what the high
band of 8 exists to hold, and it is the trigger on which this row would be re-estimated rather than
stretched.

**Band 2 / 4 / 8. Medium confidence** — the mechanics are enumerated; the variance is the window and
other people's availability. [R9](#r9--cutover-re-authentication-and-anonymous-cart-loss) carries the
two accepted losses as risks.

#### W14 — Documentation revision

**Scope** is [03 §5](03-modernization-roadmap.md)'s: three tracked documents revised to describe the
target workflow, with the two reference editions' documents marked historical.

**Basis.** Three tracked documents, each documenting a workflow the target replaces, and each cited at
the lines that carry the statement needing revision: the root document's three-edition framing and its
own recommendation to use modern .NET [README.md:3], and its description of the repository as a Visual
Studio tutorial application [README.md:9]; and the two per-edition documents' Visual Studio and
local-database prerequisites and their build-and-run workflow [src/MVC4/README.md:7-8],
[src/MVC4/README.md:22-24], [src/MVC5/README.md:7-8], [src/MVC5/README.md:22-24]. Terminal in the
graph as far as the port is concerned, but **not terminal in the dependency graph**:
[03 §4.2](03-modernization-roadmap.md) adds the edge `W13 → W14`, so documentation revision closes
after cutover rather than alongside the port — which is why
[section 8.3](#83-the-critical-path-and-what-to-do-first-if-the-goal-is-to-narrow-the-estimate) places
it at the end of the critical path.

**Band 1 / 2 / 3. High confidence.**

#### W15·C — Pre-admission affinity retirement (secondary hosting path)

**Scope** is [03 §5](03-modernization-roadmap.md)'s, and it is **conditional**: it exists only if the
secondary hosting target of [06 §4](06-azure-hosting-recommendations.md) is selected. It carries the
same control as W15 at a different point in the sequence — *before* W13 rather than after it — because
on that platform session affinity and weighted revision routing cannot both be held
([06 §4.3.1, §12.1.2](06-azure-hosting-recommendations.md)).

**Basis.** The same cross-worker round-trip as W15, run on an isolated non-production revision whose
two-replica premise must be established rather than assumed, plus three readbacks — affinity absent,
multiple revision mode configured, and the reversal *consequence* recorded, since reversal here is a
change of deployment model rather than one setting. Slightly above W15's band for those three readbacks
and the revision to stand up, and no higher because nothing in it is unenumerated.

**Band 1 / 2 / 4. High confidence. Not in the total** ([§5.1](#51-summary-table),
[§6.3](#63-what-is-deliberately-not-in-the-total)). It is **off the critical path on the primary hosting
path because it does not exist there**; on the secondary path it joins the chain immediately before W13
([§8.3](#83-the-critical-path-and-what-to-do-first-if-the-goal-is-to-narrow-the-estimate)).

---

#### W15 — Affinity retirement

**Scope** is [03 §5](03-modernization-roadmap.md)'s: a verification, not an elapsed interval, followed
by one platform setting.

**Basis. Input 27** — **15** application `.config` files, **0** declaring `<sessionState>` and **0**
declaring `<machineKey>`, a file count and census from [01 §6.6](01-architecture-overview.md). That census is
the whole reason this row exists: because no edition ever declared either element, affinity is the only thing
holding the current arrangement together, and it can be withdrawn exactly once the two controls W10 provisions
replace it. So the cross-instance session round-trip must pass with affinity off in a non-production
environment first; the production change is then a single reversible setting. Terminal in the graph.

**Band 0.5 / 1 / 2. High confidence** — and it is a genuine workstream rather than a footnote because
skipping it leaves the scale-out property that motivated part of W10 unrealized.

#### W16 — Personal-data governance

**Scope** is [03 §5](03-modernization-roadmap.md)'s: six exit conditions across **two stages** — a policy
stage and a mechanism stage — exiting only when the retention periods, the non-production handling rules,
the legal-hold process, the deletion or anonymization operation, the backup-propagation window and the
access auditing are all in place. The requirement and the field-level scope are owned by
[09 §3.11 and §6.8](09-security-assessment.md) and are not re-derived here.

**Basis.** **9 personal-data fields on the order record plus the identity link** (input 22, field count)
across the **2** stores of input 28, with the access-audit half of condition 6 landing in the sink whose
verification population is input 21's **16** event classes. The cost is not the field count — it is that
three of the six conditions are **decisions requiring a named approver** and three are **engineering with a
verification obligation**:

| Stage | Content | Low | Expected | High |
| --- | --- | ---: | ---: | ---: |
| **1** Policy — conditions 1–3 | Retention per data class; handling rules for non-production copies of real personal data; a legal-hold process that suspends deletion. Three approver constituencies: the data owner, security and legal. Depends on W1 alone, so it is available in the widest concurrency set | 1 | 2.5 | 6 |
| **2** Mechanism — conditions 4–6 | A deletion or anonymization operation demonstrated against **synthetic** data from the release path; the backup-propagation window defined and verified against [06 §6.7](06-azure-hosting-recommendations.md)'s retention; access auditing live and *observed arriving* in the sink [06 §9.2](06-azure-hosting-recommendations.md) defines — these records are row 2 of [06 §9.5](06-azure-hosting-recommendations.md)'s producer matrix, so the obligation includes their export into that section's audit store and not the first hop alone — in every environment holding real personal data. Depends on stage 1, W3, W7, W11 and **W10** | 2 | 3.5 | 6 |
| **W16 total** | | **3** | **6** | **12** |

**Two properties of this row a reader should not misread.**

- **It is not a compliance-paperwork row.** Conditions 4 to 6 are implementation with verification: an
  operation that runs, a backup window that is measured rather than asserted, and an audit trail that is
  observed arriving. The policy stage is what carries the wide right tail, because a retention period the
  organization has never had to set can take one conversation or several.
- **It gates work in three places, and its stages sit at opposite ends of the sequence** — which is why
  its position matters far more than its size. [03 §5](03-modernization-roadmap.md) makes **conditions
  1–3 an entry condition of W3 and W4**, because W3 attaches the committed credential and catalog
  databases and W4 restores both store pairs before every run, which is the roadmap's first processing of
  real personal data; **all six an entry condition of W8 and W9**, whose rehearsal copies are held to the
  same standard as production now that the mechanism exists; and **all six an entry condition of W13**.
  The stage-2 half additionally consumes W10's verified sink, which is what places it after provisioning
  and therefore on the critical path ([§8.3](#83-the-critical-path-and-what-to-do-first-if-the-goal-is-to-narrow-the-estimate)).

**The staging changed this row's position, not its size, and the band is unchanged at 3 / 6 / 12.** The
six conditions are the same six; what moved is which workstreams wait on which of them. One increment is
absorbed rather than added: condition 6 now requires access auditing live in the **rehearsal** environment
as well as the production one, which is the same operation applied a second time from the same release
path, and the mechanism stage's high band of 6 against an expected 3.5 already accommodates a second
environment. Nothing here is re-counted from a different input.

**The restructuring of stage 2 is the same kind of change, and it is stated because it looks like a
reduction and is not.** [03 §5 W16](03-modernization-roadmap.md) now has stage 2 prove the deletion
mechanism **against synthetic data**, with **liveness over real data asserted by W8, W9 and W13** as their
own entry and exit conditions rather than by this row. That moves *where the real data is touched* and
therefore where the risk sits; it does not remove an artifact from this row, because the operation, the
measured backup window and the observed audit trail are the same three deliverables either way. It also
gains one item — the **personal-data read** that
[W10](#w10--hosting-provisioning-and-platform-configuration) no longer performs, which belongs here because the
governed access path and its auditing are this stage's mechanism. That arrival is `+0 / +0.5 / +1` and the
synthetic-data restructuring is `−0 / −0.5 / −1`, so the stage nets to zero and the band holds at
2 / 3.5 / 6. **This is the one workstream in this pass whose scope changed in both directions and whose
figure legitimately does not move**; stating the two halves is what distinguishes that from an omission.

**Band 3 / 6 / 12. Low confidence**, for the same reason as W1's width: the band is set by how many
constituencies must convene and whether the organization already has retention policy to inherit, neither
of which is knowable from the repository. [R15](#r15--personal-data-governance-is-unowned) carries it, and
[section 9.4](#94-the-five-risks-that-are-approval-decisions-not-mitigations) lists it as an approval
decision rather than an engineering risk.

---

## 6. Totals, and where the effort actually lives

### 6.1 The totals

| | Low | Expected | High |
| --- | ---: | ---: | ---: |
| The sixteen workstreams ([§5.1](#51-summary-table)) | 142.5 | 259.5 | 458.5 IED |
| The manual visual review ([§7.1](#71-the-manual-visual-review)) | 2.5 | 4.5 | 7.5 IED |
| The manual accessibility review ([§7.3](#73-the-manual-accessibility-review)) | 2.5 | 4.5 | 8 IED |
| **Total** | **147.5** | **268.5** | **474** IED |
| *For reference, excluded:* the conditional pre-admission affinity retirement | 1 | 2 | 4 IED |

**The approved-delta sign-offs are not a third line here.** They are **2 / 4 / 8 inside W1's own band**
— see [section 5.1](#51-summary-table)'s two conventions and [section 7.2](#72-the-approved-delta-sign-offs)
— so they are already counted in the 259.5 above and are deliberately not added again.

The high band is about **3.2 times** the low band (454 ÷ 142 = 3.20). That spread is not estimator
hedging — it is the arithmetic consequence of four low-confidence rows worth **86 expected IED**
(W3 4 + W4 66 + W8 8 + W9 8) whose difficulty depends on a schema that has not been extracted and on
behaviour that has never been asserted by a test ([section 4.4](#44-confidence-and-its-reason)), plus
one further row — W2's — whose *outcome* is unknown even though its tasks are enumerated. **Those four
rows are a third of the expected total** (86 ÷ 258 = 33.3%), against a quarter when this
document's first derivation was written, and the whole of that shift is W4: the ratio is stable while the
low-confidence share is now flat, because the row that grew most this round is enumerated far more tightly
than it once was without being any better calibrated, and because this round's growth landed mostly in
Medium rows.

#### 6.1.1 The walk from the previously published total

**Nine rows moved in the reconciliation that produced the figures above, and one arithmetic defect in the
previous total is corrected alongside them.** The walk is stated here, once and in full, because
[W4](#w4--build-governance-bootstrap-pre-port-behavioural-baseline-and-test-suite)'s and [W7](#w7--the-aspnet-core-port)'s own
records point at this section for it rather than restating it — and because a total that moves without a
walk is a total nobody can check.

**First the defect, because the base of the walk depends on it.** The previously published total was
`90 / 169.5 / 305`. [§5.1](#51-summary-table)'s own rows at that time summed to `90 / 170 / 306`. The
difference is exactly `0 / 0.5 / 1` — [W12](#w12--administrator-provisioning-tool)'s rejection-contract
increment, which reached that row's band and never reached the total. That is precisely the failure the
"all six places or none" rule in W4's record exists to prevent, and it was found by **summing** the column
rather than reading it. The walk therefore starts from the corrected base `90 / 170 / 306`.

| Row | From | To | Delta (L / E / H) | What moved it |
| --- | --- | --- | --- | --- |
| W4 | 21 / 34.5 / 59 | 24 / 38 / 64 | +3 / +3.5 / +5 | Twelve further coverage rows, plus the [§4.2.1](#421-the-rounding-rule-stated-once) rounding correction. **This row moved twice and the delta is the sum of both moves**: `+1.5 / +1.5 / +2` for input 14's fifth reading (five rows plus the rounding correction, published as 22.5 / 36 / 61) and `+1.5 / +2 / +3` for the parity term's current reading (the **seven** rows that survive the withdrawal recorded in W4's own record, at the same per-row rate; the sixth pass published that increment as `+1.5 / +2 / +3.5` over eight rows, and only its high column moved). [W4's own record](#w4--build-governance-bootstrap-pre-port-behavioural-baseline-and-test-suite) derives each separately |
| W7 | 25.5 / 48 / 83.5 | 29 / 54.5 / 94.5 | +3.5 / +6.5 / +11 | The **retained** external-login surface, the new pipeline-platform-controls sub-row, and the CSP sub-row at eleven tests |
| W9 | 4 / 9 / 16 | 5 / 11 / 20 | +1 / +2 / +4 | The reverse replay rehearsed in **both** branch directions, the two per-context change-tracking migrations, and the governed snapshot as a declared input |
| W10 | 5 / 10 / 18 | 5.5 / 11 / 19.5 | +0.5 / +1 / +1.5 | Two just-in-time activations of the logical server's Entra administrator group for steps 0 and 4b, `G-CSP-BROWSER`, and the affinity-**disabled** second session round-trip — **net** of the twelve deployed event classes that moved to W12 and the personal-data read that moved to W16 stage 2, which left this row proving the collection path with **one** canary class |
| W11 | 5 / 9 / 16 | 7.5 / 13.5 / 24 | +2.5 / +4.5 / +8 | Ten gated stages rather than four, the Migrate split, the two guarded one-time extensions, and **four** gates that must be made to fail |
| W12 | 3 / 5.5 / 10.5 | 3.5 / 7.5 / 12.5 | +0.5 / +2 / +2 | Input 23's operator-host increment re-derived under [§4.2.1](#421-the-rounding-rule-stated-once), exit condition 7's deployed census, and the **fourth** separately gated invocation — the last of which had been priced in that row's prose and left out of its band |
| W13 | 4 / 8 / 15 | 5 / 10 / 18 | +1 / +2 / +3 | The post-drain snapshot, **three** post-load invocations, the replay run in the window, and the non-mutating credential check |
| Visual review | 2 / 3.5 / 6 | 6.5 / 12.5 / 22.5 | +4.5 / +9 / +16.5 | A semantic capture census replacing a filename count, and the browser dimension the sizing had no term for |
| Delta sign-offs | 3.5 / 7.5 / 13 | 4 / 7.5 / 13 | +0.5 / 0 / 0 | The rounding correction alone |
| **Sum of the deltas** | | | **+17 / +30.5 / +51** | |

`90 + 17 = 107`, `170 + 30.5 = 200.5`, `306 + 51 = 357`. **The other nine rows did not move at all** —
W1, W2, W3, W5, W6, W8, W14, W15 and W16 — and each states in its own basis why: for
[W1](#w1--approval-of-this-assessment) and [W5](#w5--repository-wide-path-casing-audit) the band is
**re-derived from the current census rather than carried**, and it lands where it was; for the other seven
nothing in their basis changed. Nine moved plus nine unmoved is the eighteen rows
of [§5.1](#51-summary-table), so the walk accounts for every row rather than for the ones that changed.

**Two intermediate totals were published between the base and the figure above, and both are recorded
rather than overwritten silently.** `105.5 / 198.5 / 354` was this walk's result before W4's sixth reading
of input 14, when that row stood at 22.5 / 36 / 61. The parity term then moved from 108 rows to **116**,
and `107 / 200.5 / 357.5` was published at that reading, W4 having moved by `+1.5 / +2 / +3.5`. **The
withdrawal of 05's row 116 then took the parity term to 115** — a pointer row that added no test, whose
eleven HTTP tests input 23 already carried, as [W4's own record](#w4--build-governance-bootstrap-pre-port-behavioural-baseline-and-test-suite)
derives — so the increment over the same 108-row base is `+1.5 / +2 / +3` for **seven** surviving rows and
`105.5 + 1.5 = 107`, `198.5 + 2 = 200.5`, `354 + 3 = 357` — the same destination the delta column reaches
from the base, checked from the other end. **Between that published total and the figure above only the high
column moved**, `357.5` to `357`, which is why the low and expected figures are identical in both. **No
other row moved in either pass**: the two gates that hold those coverage rows, W7 and W9, state in their own
bases why executing an already-authored row is a gate condition rather than a second cost.

### 6.2 The finding that matters most in this document

**The port of existing code is under a quarter of the expected effort. Everything else — net-new
capability, data work, and governance and verification — is a little over three quarters.**

**This table partitions the model by activity, not by workstream, and that distinction is the correction
that produced the figures below.** An earlier version of this section assigned each workstream wholly to
one character. That put **all** of W7 in *porting* although three of its eleven sub-rows have no source
counterpart at all, **all** of W11 in *net-new* although its first part is an approval and nothing else,
**all** of W13 in *governance* although the row is the production data extraction, load and
reconciliation, and **all** of W16 in *governance* although its second stage is an implementation with a
verification obligation. A partition by workstream cannot be right for a model whose largest row is
itself mixed, and [W7](#w7--the-aspnet-core-port)'s **Character** column exists so that this section reads
the activity rather than the row name.

**How each row was assigned, stated so the partition can be re-checked rather than taken on trust.**
Three rules, applied in order:

1. **A row with a stated sub-decomposition is split along it**, using its sub-bands exactly as that row
   publishes them. Three splits cross a category boundary: [W7](#w7--the-aspnet-core-port)'s **Character**
   column, [W11](#w11--ci-provider-selection-then-pipeline-authoring)'s two parts, and
   [W16](#w16--personal-data-governance)'s two stages. A fourth row is decomposed but its split does not
   cross one — the visual review's capture and review halves ([§7.1](#71-the-manual-visual-review)) are
   both verification — so it is carried whole. No sub-band is re-judged to make a category come out.
2. **A row without a sub-decomposition is assigned whole, to the character its own basis states.**
   [W4](#w4--build-governance-bootstrap-pre-port-behavioural-baseline-and-test-suite) states "entirely net-new";
   [W10](#w10--hosting-provisioning-and-platform-configuration), [W12](#w12--administrator-provisioning-tool) and
   W11's part 2 replace nothing that exists in the repository;
   [W3](#w3--authoritative-schema-extraction), [W8](#w8--identity-migration-tooling-rehearsed-against-a-copy),
   [W9](#w9--domain-data-migration-tooling-rehearsed-against-a-copy) and
   [W13](#w13--cutover) are the data work;
   [W6](#w6--project-format-conversion-and-dependency-transition) is porting, because its two drivers are the
   28 package outcomes and the conversion of an existing project file, and the four net-new declarative
   manifests inside it are below the granularity of [§4.2.1](#421-the-rounding-rule-stated-once);
   [W1](#w1--approval-of-this-assessment), [W2](#w2--mvc-5-build-reproduction-and-the-restore-precondition),
   [W5](#w5--repository-wide-path-casing-audit), [W14](#w14--documentation-revision),
   [W15](#w15--affinity-retirement) and the delta sign-offs are governance and verification.
3. **Every split sums back to its row's published band, and the four categories sum to the model total in
   all three columns.** Both identities are checked immediately below rather than asserted.

| Character of the work | Rows and parts | Low | Expected | High | Share of expected |
| --- | --- | ---: | ---: | ---: | ---: |
| **Porting existing code** | W6; W7's **eight** porting sub-rows | 24.5 | 47 | 82 | **23.4%** |
| **Net-new capability, with no legacy volume to scale from** | W4; W7's **three** net-new sub-rows; W10; **W11 part 2**; W12; **W16 stage 2** | 48 | 82.5 | 141.5 | **41.1%** |
| **Data work gated on a schema not yet extracted** | W3, W8, W9, **W13** | 16 | 34 | 63 | **17.0%** |
| **Governance, verification and documentation** | W1, W2, W5, **W11 part 1**, W14, **W15**, **W16 stage 1**, and both non-code tasks | 18.5 | 37 | 70.5 | **18.5%** |
| **Total** | | **107** | **200.5** | **357** | **100.0%** |

**The shares are stated to one decimal place, deliberately.** Rounded to whole percentages they are
`23 + 41 + 17 + 18`, which sums to 99 and presents a partition that visibly fails to add up; at one decimal
they are the rounded quotients of the expected column — `47 / 200.5`, `82.5 / 200.5`, `34 / 200.5`,
`37 / 200.5` — and `23.4 + 41.1 + 17.0 + 18.5 = 100.0`.

**The two identities, as arithmetic.** Category sums, column by column:
`24.5 + 48 + 16 + 18.5 = 107`, `47 + 82.5 + 34 + 37 = 200.5`, `82 + 141.5 + 63 + 70.5 = 357` — each equal
to [§6.1](#61-the-totals)'s total for that column. Split sums, against the band each row publishes:
W7's `23 + 6 = 29` / `44 + 10.5 = 54.5` / `76 + 18.5 = 94.5`; W11's `0.5 + 7 = 7.5` / `1.5 + 12 = 13.5` /
`3 + 21 = 24`; W16's `1 + 2 = 3` / `2.5 + 3.5 = 6` / `6 + 6 = 12`. Every one matches.

**Which assignments moved, and in which direction.** Five, and none of them is a re-estimate — every band
above is the one its own row publishes:

- **W7 splits**, moving `6 / 10.5 / 18.5` out of *porting* into *net-new*: the pipeline platform controls,
  the security-event emission and the CSP report endpoint. Those three sub-rows are the ones
  [11 §3](11-cloud-readiness-assessment.md) records as absent from the source, so there is nothing to port.
- **W11 splits**, moving its `0.5 / 1.5 / 3` provider gate out of *net-new* into *governance*. Its own
  sub-table calls that part "an approval, not engineering".
- **W16 splits**, moving its `2 / 3.5 / 6` mechanism stage out of *governance* into *net-new*. Conditions 4
  to 6 are an operation that runs, a window that is measured and an audit trail observed arriving.
- **W13 moves whole**, `5 / 10 / 18`, out of *governance* into *data work*. It is the one and only
  production extraction, load and reconciliation, gated on the same unextracted schema as W9.
- **W15 moves whole**, `0.5 / 1 / 2`, out of *net-new* into *verification*. The capability it retires
  affinity against is W10's; this row is the cross-instance round-trip that proves it, plus one setting.

**The non-porting share, re-derived rather than carried over.** `82.5 + 34 + 37 = 153.5` expected IED of
**200.5** — **76.6%** — against the **47** that is porting. The earlier reading of this section stated
**~70%** as `59.5 + 22 + 37 = 118.5` of 169.5 against 51 of porting. Both the share and its complement
moved, and in the same direction, for two independent reasons: the totals grew as nine rows were
re-estimated against the corrected roadmap, and splitting the largest porting row moved 10.5 expected IED
from porting to net-new. Porting therefore falls from ~30% to **23.4%** while the model grows — and it fell
again, by a tenth of a point, when input 14's parity term moved to its then-current **115** rows and W4's
authoring band grew with it, because every one of those 2 additional expected IED is net-new. **The
subsequent withdrawal of 05's row 116 did not move the share back**: it moved only W4's high column, and
this partition's shares are quotients of the expected column.

**This is the conclusion an estimate derived from lines of code cannot reach.** Scaling from the
migration source's 2,097 non-blank lines (sizing metric) would produce a number for the **23.4 percent**
that is porting and **silently omit the other 76.6 percent**, because none of it is predicted by any
quantity in the existing codebase:

- **The test suite has no legacy volume at all.** There are **0** tests today (input 15), and the parity
  suite is required against **75** coverage rows with **25 of them baseline-bearing** and 22 of those run
  twice (input 14, which owns that count and re-reads it) — plus **17** further required tests that have no
  baseline to run against and are costed in the workstreams that build what they test (input 23), for
  `75 + 17 = 92` executable scenarios in all. It is the
  second-largest row in the model, and its size is set by the behaviour to be characterized, not by the
  code that implements it.
- **Observability, CI and the provisioning tool are absences.** The repository has no logging,
  telemetry or health endpoint [08 §7.1](08-technical-debt-register.md) and no pipeline, publish
  profile or container manifest [08 §7.2](08-technical-debt-register.md). Work that replaces *nothing*
  cannot be sized from the thing it replaces.
- **The data migrations are gated on an extraction that has not happened.** Their band is set by the
  gate, not by six entities' worth of rows.
- **Two of the largest single items in the model are not code at all** in the ordinary sense: W4's
  fixtures and W1's decisions.

Correspondingly, **the seed data is the model's clearest example of volume without effort**: 820
non-blank lines (sizing metric), 39 percent of the migration source, and about 4 percent of W7's band,
because [08 §4.2](08-technical-debt-register.md) directs it be treated as a data-handling decision
rather than as porting. Volume and effort are not proportional in either direction on this codebase.

### 6.3 What is deliberately not in the total

- **The conditional incremental path.** Assumption **A6** takes the settled single-cutover approach.
  The alternative in [05 §11.6](05-aspnet-core-migration-approach.md) is a different shape — two hosts,
  two pipelines, an adapter surface on both sides — and **this model does not cost it.** If that path is
  ever selected, this document is re-estimated rather than adjusted.
- **Porting either reference edition.** Assumption **A7**; see
  [R10](#r10--scoping-by-analogy-across-editions).
- **Repository-hygiene remediation**, sized separately in
  [section 7.3](#73-the-manual-accessibility-review) because it gates nothing.
- **Any new functional requirement.** Assumption **A9**.
- **The interim Windows hosting path.** [06 §5](06-azure-hosting-recommendations.md) records it as an
- **~~The headless-browser harness.~~ No longer excluded — it is inside the bands.**
- **Elapsed time.** [Section 8](#8-sequencing) gives the dependency order and the concurrency the graph
  *alternative* to [03 §5](03-modernization-roadmap.md)'s sequence rather than a workstream inside it, and
  [06 §13.4](06-azure-hosting-recommendations.md) deliberately **keeps its two risks** rather than routing
  them here, so this model neither sizes it nor places it in [§8](#8-sequencing). The boundary is stated
  rather than left silent, because one property of that path is easy to assume wrongly: **it is not
  configuration-only.** 06 classifies it as requiring two **approved source changes** to the un-ported
  application before it may be pointed at a migrated database — disabling the destructive initializer
  registration [src/MVC5/MvcMusicStore/Global.asax.cs:20], which is registered a second time at
  [src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:16], and removing the startup provisioning invocation
  [src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:18], whose `async void` body is at
  [src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:21]. Each falls under the same approval gate as any
  other code change, so if the interim step is selected it is estimated at that point, with 06's two
  retained risks carried into whatever plan covers it.
- **Recovery work that only a specific failure would require.** Two rungs of
  [06 §11.5](06-azure-hosting-recommendations.md)'s forward recovery ladder **are** costed, because they
  are properties of the release path rather than responses to a fault: stopping admission, and
  redeploying the last known-good revision against an unchanged database, both inside W11's rehearsal.
  The rungs beyond them are **not**: a corrective migration or targeted data fix authored, reviewed and
  rehearsed after a fault is found, and rung 4 — **a point-in-time restore of the target database
  *followed by a replay* of the interval between the restore point and the damage**, read from the
  **retained damaged original** rather than from the restored copy, which by construction does not contain
  the rows in question ([06 §11.5.4](06-azure-hosting-recommendations.md)). The restore and the replay are
  excluded together, because a restore without the replay is not the rung: it recovers to `T` and leaves
  the interval lost. Those are the
  ladder's whole remainder — [06 §11.5](06-azure-hosting-recommendations.md) has since **removed** the
  fifth rung it once carried, the declared data-loss event that returned to the legacy application after
  an accepted write, so there is no longer a rung to exclude on sign-off grounds. Each remaining rung is
  contingent on a failure this plan is designed to prevent, each is sized by what actually broke, and
  [R4](#r4--domain-data-migration-rollback) carries them as a risk rather than as a band. Excluding them
  is not the same as leaving them undefined — 06 defines the ladder; this document declines to put a
  number on a repair whose subject is unknown.

  **Two things adjacent to rung 4 *are* in the total, and separating them from the rung is what keeps this
  exclusion honest.** Its **preconditions** are costed: the system-versioned history the replay reads is
  authored by [W9](#w9--domain-data-migration-tooling-rehearsed-against-a-copy)'s two `AddChangeTracking`
  migrations, and the retention period bound to the restore window is set with them. And its **mechanism
  rehearsal** — [06 §6.7](06-azure-hosting-recommendations.md)'s restore exercise at platform stand-up,
  extended by [06 §11.5.4](06-azure-hosting-recommendations.md) to one table's temporal replay — is
  **absorbed into [W10](#w10--hosting-provisioning-and-platform-configuration)'s environment half**, as a bounded
  operator exercise against a database that half provisions and on a schema still empty at that point.
  Absorbed rather than added, and stated so rather than left silent: what is excluded is the **use** of the
  rung against a specific unknown fault, never the capability to run it.
- **Elapsed time.** [Section 8](#8-sequencing) gives the dependency order and the concurrency the graph
  permits; converting to a schedule needs a capacity assumption this document does not make.

---

## 7. Work that is not code

Both items below consume real effort, neither is sized by any other document, and neither is a
workstream **of its own** in [03 §5](03-modernization-roadmap.md)'s decomposition — but each sits on a
workstream's **exit gate**, the visual review on W7's and the delta sign-offs on W1's, so each is on the
critical path even though it has no workstream number. They are carried here because omitting them would
understate the total and mis-place the gates.

One qualification, because §7.1 spans two gates rather than one. Its **review and sign-off** sit on W7's
exit gate and are on the critical path; its **baseline capture** sits inside W4's, concurrent with that
workstream's suite authoring, and is therefore off the path. Both halves are sized in §7.1 and neither is
folded into a workstream band.

### 7.1 The manual visual review

**Why it exists as a separate task.** [05 §12.5](05-aspnet-core-migration-approach.md) establishes that
the visual-preservation criterion **cannot** be met by the HTTP-level suite — that suite asserts on
response *content*, while the styling framework's major-version move changes how content *renders* —
and that automating the comparison is **deliberately rejected**. That document scopes the review and
records that **07 carries it as a task**. This is that task.

**Scope, as scoped by 05 §12.5** and not redesigned here: screenshots of every distinct page taken from
the migration source **before** the port at the **two viewports the layout actually distinguishes**; a
reviewer checklist covering the navigation bar, catalog grid, album detail, cart table, checkout form
and administration list; and a **signed-off approval artifact** recording who reviewed it, against which
baseline, and which of the approved deltas were accepted. **Which browsers** that review is performed on
is not 05's to say and is not left open: [06 §10.4](06-azure-hosting-recommendations.md) owns the matrix
and states it as four families. Sizing the review therefore needs both documents, and the basis below
enumerates what "every distinct page" resolves to in this repository — including the two pages for which
the answer is that no screenshot of them can exist.

**Both halves are manual, and this task introduces no automation of any kind.** That is not a gap in the
plan; it is [05 §12.5](05-aspnet-core-migration-approach.md)'s decision, and the reason it gives is the
machinery: automating would mean **pinning a browser-automation stack and screenshot tooling, defining
baseline images and pixel tolerances, and storing binary image artifacts** in a repository that already
carries committed database binaries — infrastructure the application needs for nothing else, bought for a
one-time layout migration. So the capture is **taken by hand, in each browser of
[06 §10.4](06-azure-hosting-recommendations.md)'s matrix, from a written protocol**, and the review is a
person against a checklist. Two facts make that the cheaper answer rather than the resigned one: **Safari
has no Windows or Linux build**, so a single scripted runner could not cover the matrix in any case and
the automation would buy three engines of the four; and the capture happens **once**, against an
application being retired, so there is no second run for a harness to amortize against.

**The automated alternative is named so it can be chosen rather than drifted into.** Adopting it would
require an approver to accept the stack, a **pinned** version for every tool, its addition to
[02](02-dependency-inventory.md)'s inventory and [04 §8](04-dotnet8-migration-strategy.md)'s outcome set,
provisioning in [W11](#w11--ci-provider-selection-then-pipeline-authoring)'s pipeline including a
Safari-capable agent, and a gate that says what a pixel difference means. **None of that is priced below**,
and no band in this document assumes any of it. A later decision to automate is therefore an addition to
this model, not a reinterpretation of it.

**Basis, and it is a census of capture *states* across *two* dimensions rather than a count of files.**
An earlier form of this row was sized from **22 non-partial view files of the 29** — a
leading-underscore filename test — times two viewports, "in the mid-tens of screenshots", with **no
browser term at all**. Both halves of that were wrong, and they were wrong in opposite directions, so
the corrected band is not a simple uplift.

**What the file test got wrong.** [A.4](#a4-the-visual-review-capture-set-input-16) carries the
classification and the commands. The underscore convention misclassifies three files in both
directions — `Store/GenreMenu.cshtml` and `ShoppingCart/CartSummary.cshtml` carry no underscore and are
component outputs rather than pages, and `Account/_RemoveAccountPartial.cshtml` carries one and is a
component output too — so the semantic inventory is **20 standalone views, 3 component outputs and 6
other Razor artifacts**. And a page is not a capture: a page with a validation state renders two
distinct layouts, while a partial has no route at all and is captured only as a state of the page that
renders it.

**The capture census — 28 states over 18 pages, and two pages that can produce no baseline whatever.**
Every state below is one for which the *layout* differs, not merely the data; the class is what makes it
obtainable.

| Class | States | Pages | What it takes to render |
| --- | ---: | ---: | --- |
| **A** — renders from the legacy application **as configured** | **23** | 16 | Nothing beyond a seeded catalog, a registered account and an administrator |
| **B** — renders only behind a **stated precondition** | **5** | +2 | A prepared fixture or a configured provider; enumerated below |
| **C** — **no baseline can exist** | **0** | +2 | Nothing: the view has no action, so no request renders it |

**Class A, 23 states over 16 pages.** `Home/Index` **2** — anonymous with an empty cart, and signed in
with a non-empty one, which is the only place the navigation bar itself differs; `Store/Index` 1;
`Store/Browse` 1; `Store/Details` 1; `ShoppingCart/Index` **2** — populated and empty;
`Checkout/AddressAndPayment` **2** — clean and with the validation summary rendered; `Checkout/Complete`
1; `Account/Login` **2** — clean and after a failed sign-in; `Account/Register` **2** — clean and with
validation errors; `Account/Manage` 1 — the local-password branch, which renders
`_ChangePasswordPartial` [src/MVC5/MvcMusicStore/Views/Account/Manage.cshtml:12-14];
`Account/ExternalLoginFailure` 1, which is a plain `GET` action and needs nothing
[src/MVC5/MvcMusicStore/Controllers/AccountController.cs:310-313]; `StoreManager/Index` 1;
`StoreManager/Details` 1; `StoreManager/Create` **2**; `StoreManager/Edit` **2**; `StoreManager/Delete` 1.

**Class B, 5 states, and each precondition is work rather than a note.** The point of listing them is
that a capture plan which assumes all 28 states fall out of a browsing session will silently produce 23.

- **`Shared/Error`.** It has no route: the `HandleErrorAttribute` global filter renders it
  [src/MVC5/MvcMusicStore/App_Start/FilterConfig.cs:10]. `customErrors` appears nowhere as a live element
  in the configuration, so the effective mode is the framework default `RemoteOnly` — the view renders for
  a **non-local** request against a throwing action, and a local request gets the diagnostic page instead.
  The precondition is therefore a remote client, not just an exception.
- **`Account/Manage`, the no-local-password branch** ⇒ `_SetPasswordPartial`
  [src/MVC5/MvcMusicStore/Views/Account/Manage.cshtml:16-19]. `ViewBag.HasLocalPassword` is false only for
  an account created through an external login, so the state is prepared as **data** — a fixture account
  with no password hash — and cannot be produced through the running UI.
- **`Account/Manage` with at least one linked login** ⇒ `_RemoveAccountPartial` renders **at all**. It
  emits nothing when the collection is empty
  [src/MVC5/MvcMusicStore/Views/Account/_RemoveAccountPartial.cshtml:3], and the collection comes from
  `UserManager.GetLogins` [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:319]. The precondition
  is a seeded external-login row.
- **`Account/ExternalLoginConfirmation`.** There is **no `GET` action** for it; it is returned by
  `ExternalLoginCallback` for an unassociated external identity
  [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:208-233]. Every provider registration is
  commented out [src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:22-35], so the precondition is a
  **configured provider** and a real external round-trip.
- **`Account/Login`, the provider-list branch of `_ExternalLoginsListPartial`.** The branch that renders
  today is the empty one — "There are no external authentication services configured"
  [src/MVC5/MvcMusicStore/Views/Account/_ExternalLoginsListPartial.cshtml:7-12] — and it is covered in
  class A. The list branch [:15-30] needs the same configured provider, and is captured in the same run
  as the state above.

**Class C, 2 pages, 0 states — and this is a finding rather than an omission.** `Home/About.cshtml` and
`Home/Contact.cshtml` have **no action in any controller** and no link from any view;
[A.4](#a4-the-visual-review-capture-set-input-16) carries the commands. They are unmodified
project-template pages, [05 §8.4](05-aspnet-core-migration-approach.md) ports them because it deletes no
view, and **no baseline screenshot of either can be taken from an application that cannot serve them.**
Their review is a markup comparison of the legacy file against the ported one, and the review record
states the absence of a baseline rather than reporting a pass.

**The nine artifacts with no capture of their own are covered inside those 28, and named so that none is
assumed.** `_ViewStart.cshtml` emits no markup at all — it sets `Layout`
[src/MVC5/MvcMusicStore/Views/_ViewStart.cshtml:1-3] — and so has no appearance to compare. `_Layout` is
in every capture. `_LoginPartial` has both of its states in `Home/Index`'s two. The `GenreMenu` and
`CartSummary` component outputs are in every capture, and `CartSummary`'s zero and non-zero renderings are
`Home/Index`'s two [src/MVC5/MvcMusicStore/Views/Shared/_Layout.cshtml:25-26].
`_ChangePasswordPartial`, `_SetPasswordPartial`, `_ExternalLoginsListPartial` and `_RemoveAccountPartial`
are the four states named above.

**The second dimension: four browser families, which the previous sizing had no term for at all.**
[06 §10.4](06-azure-hosting-recommendations.md) states the supported matrix — **Chrome, Edge, Firefox and
Safari** — and commits that "the manual visual-comparison review that the Bootstrap upgrade requires is
performed on the stated browsers only". A review performed on one engine does not discharge that. It also
cannot be discharged by comparing across engines: a Chrome baseline against a Safari post-port capture
confounds the framework upgrade with the rendering engine, so **each pair is same-engine**, which puts the
browser term on the **baseline** as well as on the ported captures. **Safari is the constraint** — it has
no Windows or Linux build, so the capture needs a **macOS host or a device-lab session**, and that
prerequisite is costed below rather than assumed. Without it, one of the four families of 06 §10.4's matrix
goes unverified in both directions.

**The multiplication, stratified — and the stratification is the honest part.** Applying four engines to
all 28 states would buy nothing on the nine that differ from another state of the same page only in data or
validation content: the engine-specific rendering of a page's constructs is settled by any one capture of
that page. So:

- **Layout-bearing captures: 19** — one per page that has any capture (18) plus `Home/Index`'s signed-in
  state, because the navigation bar itself differs there. **All four engines, both viewports.**
- **Content variants: 9** — the remainder, `28 − 19`. **Primary engine only, both viewports.**

```text
baseline    = 19 × 4 × 2  +  9 × 1 × 2  =  152 + 18  =  170 screenshots
post-port   = the same set                            =  170 screenshots
                                            total     =  340 screenshots
                                    comparison pairs  =  170
```

Two viewports throughout, per [05 §12.5](05-aspnet-core-migration-approach.md) — the navbar collapses at
one breakpoint [src/MVC5/MvcMusicStore/Views/Shared/_Layout.cshtml:15], [:22] and the viewport meta tag
[:5] confirms the responsive intent. Assumption **A4** still applies — one reviewer, with a second
signature only for approval.

> **The browser-matrix review has two halves with two owners, and this row is one of them.**
> [05 §12.5](05-aspnet-core-migration-approach.md) discharges **content-security-policy enforcement**
> through the same 06 §10.4 matrix and on the same manual footing, because no `WebApplicationFactory`
> client parses a policy or emits a violation report. That half is the deployed-browser gate
> `G-CSP-BROWSER`, it is **[W10](#w10--hosting-provisioning-and-platform-configuration)'s exit condition 9**, and
> it is costed in W10's band — the twelfth of input 23's CSP tests, the other eleven being HTTP-observable
> and W7's. **Nothing about it is counted here**, and the appearance half is not counted there. Two manual
> reviews against one matrix of agents, sized once each.

**Band 6.5 / 12.5 / 22.5 IED**, up from **2 / 3.5 / 6**, and every part of the increase is one of the two
corrections above rather than a re-judgement: the state census replaces the file count, and the browser
dimension enters a sizing that previously had none. `170` comparison pairs against the previous
`22 × 2 = 44` is very nearly a factor of four, and the band moves by rather less than that, because the
three secondary engines are reviewed at a lower rate than the primary. It splits across two points in the
sequence, which is the part easiest to get wrong:

| Part | Band (L / E / H) | When | Note |
| --- | --- | --- | --- |
| Baseline capture | **2.5 / 5.5 / 11** | **Inside W4**, against the running legacy application. [03 §5 W4](03-modernization-roadmap.md) makes the capture part of that workstream, so it sits in **concurrency set 3** alongside W4's suite authoring — and it cannot sit earlier than W4's own entry, because photographing the application requires the application to run, which is exactly what W4's entry on W2 exists for | If this is missed, the baseline becomes progressively unreproducible rather than instantly impossible — [04 §12](04-dotnet8-migration-strategy.md) retains the legacy source read-only, but the toolchain, the local database instance and the browser version needed to *run* and photograph it all drift. [§8.4](#84-the-sequencing-hazard-in-the-visual-baseline-capture) argues it in full |
| Review **and sign-off** | **4 / 7 / 11.5** | **Inside W7, as exit condition 5 of its gate** | Reviews the ported rendering against the captured baseline, and records the signed approval that closes the gate |

**The capture part, derived.** Five components, and the two that dominate it are the ones the previous
sizing had no term for:

| Component | L / E / H | Why it is not zero |
| --- | ---: | --- |
| The **capture protocol** — written, not coded | 0.5 / 1 / 2 | W4 already restores both committed databases and runs the legacy application, so what this adds is the **ordered state list, the two viewport settings, the sign-in and state-entry steps and the file-naming convention** — precise enough that a second person reproduces the same 170 shots, and pairwise-comparable by name. **It is a document, not a driver**: no browser is automated here |
| **Safari-capable host access** obtained | 0.5 / 1 / 2 | No Windows or Linux build exists; this is a macOS host or a device-lab session, and it is access provisioning rather than engineering |
| Three **data**-level preconditions | 0.5 / 1 / 2 | The no-password fixture account, the seeded external-login row, and a non-local request path for the error view |
| One **provider**-level precondition | 0.5 / 1.5 / 3 | A real external provider registered in a test tenant, which is the only thing that makes the two class B external-login states renderable at all |
| Taking and filing **170** baseline screenshots **by hand**, in four browsers | 0.5 / 1 / 2 | Manual by decision, per the paragraphs above. Working the protocol's list with the browser open, most shots are a navigation and a viewport toggle, so the band implies about **0.003 / 0.006 / 0.012 IED per screenshot** across the 170 — stated as a check on the figure rather than as its derivation, since the work is bounded by the prepared list and by re-shooting mistakes before the legacy application becomes unrunnable, not by a per-shot rate |
| **Capture subtotal** | **2.5 / 5.5 / 11** | |

**The review part, derived from the pair counts and two stated rates.** The rates are the judgement in
this row; everything else is arithmetic.

- **Primary-engine adjudication** — the full checklist of [05 §12.5](05-aspnet-core-migration-approach.md)
  against every state: **28 states × 2 viewports = 56 pairs** at **0.03 / 0.05 / 0.09 IED** per pair
  = `1.68 / 2.8 / 5.04`.
- **Secondary-engine divergence checks** — Edge, Firefox and Safari against a primary pair already
  adjudicated, so the question is narrower and the rate is roughly a third:
  **19 × 2 × 3 = 114 pairs** at **0.01 / 0.02 / 0.03 IED** per pair = `1.14 / 2.28 / 3.42`.
- Sum `2.82 / 5.08 / 8.46`, rounded under [§4.2.1](#421-the-rounding-rule-stated-once) to
  **3 / 5.5 / 8.5**.
- **Delta adjudication and the signed artifact** — recording which of deltas 5, 6 and 13 were accepted,
  and by whom: **0.5 / 1 / 2**.
- **The two class C pages** — markup comparison of legacy against ported, and recording the absence of a
  baseline rather than a pass: **0.5 / 0.5 / 1**.
- **Review subtotal `3 + 0.5 + 0.5 / 5.5 + 1 + 0.5 / 8.5 + 2 + 1` = 4 / 7 / 11.5.**

`2.5 + 4 = 6.5`, `5.5 + 7 = 12.5`, `11 + 11.5 = 22.5` — the row's band.

> **The review and its sign-off are inside W7, not after it.**
> [03 §5 W7](03-modernization-roadmap.md) makes them **exit condition 5** of the port's gate, which means
> they close before W13 is entered and before any traffic is served from the ported application. An
> earlier reading of this section scheduled the review after W7 exited and placed it in the
> post-cutover concurrency set; that made a blocking gate unblockable, because there was no state in
> which the gate could be satisfied without having already shipped the thing it gates. Only the
> **baseline capture** precedes the port, and it has to, since the legacy application is the artefact
> being photographed.
>
> **This task supplements the automated suite; it does not substitute for any part of it.** It covers
> rendered appearance and nothing else — [05 §12.5](05-aspnet-core-migration-approach.md) states why the
> HTTP-level suite cannot assert that. The suite's own contracts, the blocker demonstrations and the
> security-event emission remain automated, deterministic and blocking at W7's exit in every case, and a
> signed manual checklist is not an alternative form of evidence for any of them.
>
> **Effort consequence.** Because the review sits on W7's gate rather than after it, its **7 expected IED**
> is **on the critical path**, and [section 8.3](#83-the-critical-path-and-what-to-do-first-if-the-goal-is-to-narrow-the-estimate)
> counts it there. The **capture** part is not: it sits inside W4, which is itself on the path, so the
> capture's 5.5 expected IED is off-path work concurrent with W4's own. It is still reported only here, and
> neither part is folded into W4's or W7's band.

### 7.2 The approved-delta sign-offs

**Why it exists as a separate task.** [05 §11.5](05-aspnet-core-migration-approach.md) enumerates
**27 approved deltas**, each with a **named approval owner** — and an approval owner who has not
actually approved is an open question, not a delta. Obtaining the decisions is work with elapsed
effort, and [03 §5](03-modernization-roadmap.md)'s W1 exit gate requires a recorded decision on each.

**Basis.** **27 deltas** across **5 approver constituencies** (input 17, entry count) — the security,
product, engineering, data and operations owners named by 05 §11.5. Two properties drive the band
rather than the count: **sixteen of the twenty-seven name more than one** constituency and must be
consented by every owner they name, and one of them is not a technical trade at all —
[R7](#r7--the-narrowed-browser-matrix) removes a class of client and belongs to the product owner
alone.

**Two of these 27 decisions are also §9.4 escalated risks, and this row is the one that pays for them.**
[R7](#r7--the-narrowed-browser-matrix) is the narrowed-browser-matrix delta and
[R9](#r9--cutover-re-authentication-and-anonymous-cart-loss) is the reauthentication-and-anonymous-cart-loss
delta. [W1's basis](#w1--approval-of-this-assessment) therefore excludes both from W1's band and takes
only R1, R13 and R15 there — the three escalated decisions that are not deltas. The partition is stated in
full there and is what keeps `6 + 7.5 = 13.5` free of overlap.

**Band 4 / 7.5 / 13 IED**, raised again by the corrected input, and **derived rather than
re-judged**. The **convening cost does not move**, because the constituency count does not move: it is
five, as it was at 14 deltas and at 18. What moves is the number of decisions those five must record. The
marginal rate is the one this row's own history establishes — at 14 deltas the band was 2 / 4 / 8 and at 18
it was 2.5 / 5 / 9.5, so four additional decisions cost 0.5 / 1 / 1.5, which is
**0.125 / 0.25 / 0.375 IED per decision**. Nine further decisions therefore cost
`9 × 0.125 / 0.25 / 0.375 = 1.125 / 2.25 / 3.375`, rounded under
[§4.2.1](#421-the-rounding-rule-stated-once) to
**1.5 / 2.5 / 3.5** and added to 2.5 / 5 / 9.5. Derived instead straight from the 14-delta base, thirteen
additional decisions cost `13 × 0.125 / 0.25 / 0.375 = 1.625 / 3.25 / 4.875`, rounded by the same rule to
**2 / 3.5 / 5** and added to 2 / 4 / 8, which gives the same **4 / 7.5 / 13**; the two routes agreeing is
the check rather than a coincidence, and it is a check the rule makes possible — rounding the unrounded
sums instead (`3.625 / 7.25 / 12.875` and `3.625 / 7.25 / 12.875`) lands on 4 / 7.5 / 13 as well, so the
answer does not depend on whether the increment or the total is the thing rounded.

**This band moved from 3.5 / 7.5 / 13 to 4 / 7.5 / 13, and only the low figure moved.** The cause is
[§4.2.1](#421-the-rounding-rule-stated-once) rather than any change of count or rate: the nine-decision
increment's low figure is 1.125, which the earlier derivation published as 1 by rounding down. The rule
rounds up without exception, so it is 1.5, and this row's low figure is 0.5 higher. **The expected and high
figures are unchanged**, which is why `6 + 7.5 = 13.5` below still holds.

**Where this row sits relative to W1, stated exactly.** It is effort **required by W1's exit gate**, so it
is sequenced with W1 — [§8.2](#82-concurrency-permitted-by-the-graph) places it in **set 0** beside W1, as
the parenthesised `+7.5` in that set's own row, and
[§8.3](#83-the-critical-path-and-what-to-do-first-if-the-goal-is-to-narrow-the-estimate) carries it on the
chain — but it is **not inside W1's band**. The two are a
partition of input 17, tabulated in [W1's basis](#w1--approval-of-this-assessment): W1's 3 / 6 / 12 buys the
convening of the **6** constituencies — this row's five delta owners plus legal — and the **3** escalated
risk decisions that are **not** deltas; this row's 4 / 7.5 / 13 buys the 27 per-delta decisions, R7 and
R9 among them. The summary table in [section 5.1](#51-summary-table) lists each exactly once,
so neither is double-counted into the other, and **6 + 7.5 = 13.5** expected IED is the whole approval
activity.

**W1's own band reads 3 / 6 / 12, and it is derived rather than carried.**
[W1](#w1--approval-of-this-assessment)'s basis prices a census of **ten mutually exclusive approval acts**
— 6 constituency briefings, the **3** escalated risk decisions that are not themselves deltas, and the
gate record — at `0.3 / 0.6 / 1.2` IED per act. Two corrections moved that census: legal's briefing added
an act, and R7's and R9's decisions removed two once they were paid per delta in this row instead. Those
movements are **different sizes** and do not cancel; the band reads as it did because no earlier version
derived it from an act census at all, and W1's basis walks the eleven-act reading it would have given so
the point is checkable rather than asserted. W1 now names the 27 decisions as *this* row's work rather than
as one of its own drivers, so nine additional deltas move this row and not that one.

**Where a withheld consent leads.** [03 §5](03-modernization-roadmap.md) W1 records that if an approval
owner withholds consent, the affected workstream's scope changes and the roadmap is revised before that
workstream begins. That is a re-estimation, not a variance inside these bands.

### 7.3 The manual accessibility review

**Why it exists as a separate task, and why it is not part of §7.1.**
[05 §8.9](05-aspnet-core-migration-approach.md) states the accessibility and interaction-state
requirements the ported markup must meet, and states plainly that **conformance cannot be asserted from
source** — it needs a running application, keyboard traversal, assistive technology and a contrast tool.
The visual review of [section 7.1](#71-the-manual-visual-review) cannot absorb this: **screenshots
cannot verify keyboard operation, focus order, announcement or contrast.** Two reviews, two methods, two
artifacts.

**Scope, as scoped by 05 §8.9** and not redesigned here: the numbered requirements of that section —
structure and semantics (a1-a7 plus a5b), labels, errors and validation (b1-b5), the cart interaction's
announcement, pending, failure and focus behaviour (c1-c5), and focus visibility and colour (d1-d4).

**The approval side of the same items is the two approval registers, and neither adds a checklist item.**
[05 §8.9](05-aspnet-core-migration-approach.md) states the mapping exhaustively and it is cited rather
than restated: **sixteen** of the 22 requirements are net-new and carried by the additions register of
[05 §11.7](05-aspnet-core-migration-approach.md) as thirteen entries, three of which cover a pair;
**two** — `a7` and `d1` — change an existing behaviour and are therefore deltas 13 and 19 of
[05 §11.5](05-aspnet-core-migration-approach.md); and **four** are on neither register because none is a
change — `b1` is preserved behaviour, `d2` is a prohibition, and `d3` and `d4` are measurements. So this
review verifies the same 22
requirements while [section 7.2](#72-the-approved-delta-sign-offs) obtains the decisions on them. The
checklist count below is therefore unchanged by either register: it counts requirements, and the registers
count approvals of the same requirements.

**Method**, which is what distinguishes the band from a document review:

| Pass | What is done |
| --- | --- |
| **Keyboard-only traversal** | Every captured page surface of [05 §12.5](05-aspnet-core-migration-approach.md)'s set is reached and operated with no pointing device: the navbar toggler at the narrow viewport, the genre dropdown open/move/close, every form field and submit control, and the cart removal control |
| **Screen-reader spot checks** | The navbar toggler's name and state; **all three data tables** — the cart table, the administration album list and the Registered Logins table — for their names and, for the two that have header cells, their header associations, the third being name-only by the decision in 05 §8.9 **a5b**; the validation summary and per-field errors; and the cart status region's announcement on removal, including whether it announces **once or twice** once focus moves onto it (05 §8.9 **c4**) |
| **Contrast measurement** | The **three** colour pairs named in 05 §8.9 **d3** and the one in **d4**, measured with a tool, each recorded as a **measured ratio against its stated threshold** — 4.5:1 for normal-size text, 3:1 for large-scale text and for a user-interface component boundary — never as a bare pass. d3's three are pass criteria; d4's pair is preserved source appearance, so a failure there is recorded as **inherited** and does not fail the gate |
| **Interaction-state exercise** | The cart removal driven through **success, rejection and network failure**, confirming the disabled/`aria-busy` pending state (c2), the announced failure message (c3) and the focus move (c4). The rejection case is the anti-forgery path of [05 §7.4](05-aspnet-core-migration-approach.md), which is why this pass needs a running application rather than a rendered response. This pass covers the **assistive-technology** properties of that interaction; its **rendered and behavioural** properties — the fade, the count and total updates, the badge text, the navbar item collapsing with no empty list item — belong to [section 7.1](#71-the-manual-visual-review)'s scripted-interaction check, performed in the same sitting so the setup is paid once |

**Basis.** The **17 capturable page surfaces** of [05 §12.5](05-aspnet-core-migration-approach.md) —
input 16 — bound the traversal, and **22 numbered requirements** across the four groups of 05 §8.9 bound
the checklist: a1-a7 plus a5b (8), b1-b5 (5), c1-c5 (5) and d1-d4 (4). Assumption **A4** applies: one
reviewer, with a second signature only for approval. The band is driven by the interaction-state pass
rather than by the surface count, because three of its four cases (**c2**, **c3**, **c4**) are behaviours
the source does not have at all and therefore have to be exercised rather than compared — which is also
why the band is **unchanged** by the one page surface added to input 16 and by the two requirements added
to the checklist. Both additions are checks on surfaces the pass already visits: **a5b** is the third
table's accessible name, read on a page the traversal already reaches, and **d4** is one more colour
sampled by the tool already open for d3.

**Band 2.5 / 4.5 / 8 IED.** It runs **inside W7's exit gate**, alongside the review half of
[section 7.1](#71-the-manual-visual-review), and it is therefore **on the critical path**: [03
§5](03-modernization-roadmap.md) W7's exit gate requires this review signed off, and W13's entry gate
requires W7's exit, so nothing downstream of W7 can start until it completes. A defect it finds is a fix
inside W7's own markup and a re-test of the gate rather than a re-plan. Section
[8.2](#82-concurrency-permitted-by-the-graph) places it accordingly, and section
[8.3](#83-the-critical-path-and-what-to-do-first-if-the-goal-is-to-narrow-the-estimate) carries it in
the critical-path column.

**The markup work itself is not here.** Implementing every requirement of 05 §8.9 is **inside W7's own
band** — it is view markup, which is what W7 is — and W7 is **not re-banded** for it. What this section
sizes is the *review*: establishing that the requirements were met, which is a separate activity with a
separate artifact and a separate reviewer.

**The artifact.** A signed-off record naming the reviewer, the date, each of the 22 requirements as met
or defective, the **four measured contrast ratios with the threshold each was measured against**, and any
defect with a named owner — a d4 failure marked **inherited** rather than introduced. It is the only place
in this plan where an accessibility claim is ever made, and it is made about **requirements met**, not
about conformance to a standard — 05 §8.9 is explicit that this document set claims no conformance.

### 7.4 Repository hygiene — sized, but outside the critical path

[08 §10](08-technical-debt-register.md) inventories the hygiene findings and
[03 §7.5](03-modernization-roadmap.md) records that they form an independent stream **gating nothing**.
They are sized here for completeness and excluded from
[section 6.1](#61-the-totals)'s total, because no workstream's entry gate depends on them.

| Item | Input | Note |
| --- | --- | --- |
| **14** committed database binaries, **43,376,640** bytes (file count and byte count) | 19 | The engineering act is small; the **decision** is not. Removing them from history is a rewrite affecting every clone, and retention is a repository-owner call, not a migration task |
| **215** committed package files (file count) | 19 | Same character: cheap to stop tracking, consequential to rewrite |
| **4** solution files for **3** projects, one stale (file count) | 19 | Resolved as a by-product of W6, which collapses them |

**Deliberately not given an IED band.** Their cost is dominated by a history-rewrite decision whose
scope depends on the repository's clone and fork topology — a fact outside this assessment. Per
[08 §12.2](08-technical-debt-register.md), the byte and file counts are **not** effort inputs, and
inventing a band for them from those counts is exactly the error that section forbids.

---

## 8. Sequencing

### 8.1 The order is 03's; the concurrency is this document's contribution

[03 §4.2](03-modernization-roadmap.md) owns the dependency graph and
[03 §6](03-modernization-roadmap.md) proves every exit gate is consumed. **Neither is restated.** What
this section adds is the reading an estimate makes possible: **which workstreams the graph permits to
proceed concurrently**, and therefore where the effort total and the elapsed shape diverge.

**No calendar position, no start or finish date, no duration.** The sets below are a property of the
dependency graph, not a schedule and not a time box: they say what *may* run alongside what, never when
anything starts or how long it takes. The set numbers are an **ordering index**, not weeks, stages or
waves — set 4 follows set 3 in the graph and nothing more.

### 8.2 Concurrency permitted by the graph

Each set contains work whose entry gates are satisfied by the sets above it, and which has no dependency
on anything else in the same set. A reader with a capacity assumption can turn this into a schedule;
this document does not.

**The sets partition the expected total exactly.** Every workstream and the manual visual review appear
**once and only once**, so the column below adds to
[section 6.1](#61-the-totals)'s 258. Four rows are split across sets, and the reason differs by row:
**W4**, because [03 §4.2](03-modernization-roadmap.md) now draws it as the two gates **4a**, the
build-governance bootstrap, and **4b**, baseline green and captured, and gives them different entry
edges; **W10**, because 03 draws it as the three sequenced gates **10a**,
**10b** and **10c**; **W11**, because 03 gives its two halves different entry edges — W1 for the
provider selection, W10a for the manifest — while drawing the workstream as a single node; and the
**manual visual review**, because [03 §5](03-modernization-roadmap.md) attaches its capture to W4's exit
gate and its review to W7's. Each split shows its parts and each part's expected IED, so no figure
appears twice.

**Each set is the *earliest* point at which its work becomes available**, so a workstream appears in the
set immediately after the one that closes its last entry gate. Four of
[03 §4.2](03-modernization-roadmap.md)'s **nineteen** nodes are internal gates of a workstream whose band
this document states as a whole: **2a**, the recorded verification run whatever it reports, and **2b**, a
passing run re-verified, both inside **W2**; and **4a** and **4b** inside **W4**. W2's band is not split
between its two, so W2 appears once as a whole while the members it opens name the gate each actually
consumes; W4's band **is** split, because its two gates fall in different sets.

| Set | Work available concurrently | Expected IED in the set | Gate that opens it |
| --- | --- | ---: | --- |
| **0** | **W1** — whose band already contains the sign-offs of [§7.2](#72-the-approved-delta-sign-offs) | 6 | The root. Nothing precedes it |
| **1** | **W2** (4), **W3** (4), **W5** (2), **W11's provider selection** (3) and **W4's gate 4a**, the build-governance bootstrap (5) — five independent streams | 18 | W1 exited. Gate 4a's only entry edge is `W1 → 4a`: the bootstrap authors and proves the governance files and the contracts project, and consumes no legacy build |
| **2** | **W4's gate 4b** (61), **W6** (4.5) and **W10a**, the provisioning gate (5.5), all three **independent of each other**, plus the **baseline capture** of [§7.1](#71-the-manual-visual-review) (1) | 72 | Gate 4b: **three** conditions — gate 4a closed, **W2's gate 2b** (a passing run, because 4b drives the running legacy application) **and W3 exited**, because 4b's fixture manifest publishes counts, ranks, quantities and order totals derived from W3's extracted schema and data. W6: gate 4a closed **and W2's gate 2a** — the recorded starting condition, pass or fail. W10a: **W5 exited**, its only entry. The capture: inside gate 4b itself |
| **3** | **W7** (109.5) and **W11's manifest authoring** (16.5), which are independent of each other | 126 | W7: W3, W4 (through gate 4b), W5 and W6 exited. W11's manifest: provider selected and W10a closed, for the deployment principal |
| **4** | The **review and sign-off** of [§7.1](#71-the-manual-visual-review) (3.5), alone in its set | 3.5 | W7's port work complete against the set-2 baseline. It **closes** W7's exit gate, so no W7 successor opens until it is signed off |
| **5** | **W10b**, the schema-application gate (2.5) | 2.5 | W10a closed and **W7 exited**, which includes set 4 |
| **6** | **W9** (8) and **W12** (6), concurrent with each other | 14 | W9: W3, W7 and W10b exited, with the generated-schema diff passed. W12: W7 and W10b exited; it does not wait on 10c |
| **7** | **W8** (8) | 8 | W3, W7 and W10b exited **and W9 exited** — the domain load precedes the Identity data migration |
| **8** | **W10c**, the data-load gate (1) | 1 | W9 and then W8 exited |
| **9** | **W13** — the convergence point (4) | 4 | W7, W8, W9, W10c, W11 and W12 all exited |
| **10** | **W14** (2) and **W15** (1), both terminal and concurrent with each other | 3 | W13 exited; W15 additionally needs W10c |
| | **Total** | **258** | |

**The reconciliation, printed so it can be checked rather than trusted:**
6 + 18 + 72 + 126 + 3.5 + 2.5 + 14 + 8 + 1 + 4 + 3 = **258**, which is
[section 6.1](#61-the-totals)'s expected total. Set 1's 18 is 4 + 4 + 2 + 3 + 5; set 2's 72 is
61 + 4.5 + 5.5 + 1; set 3's 126 is 109.5 + 16.5; set 6's 14 is 8 + 6; set 10's 3 is 2 + 1. W4's two
gates sum to 5 + 61 = **66**, W10's three gates to
5.5 + 2.5 + 1 = **9**, and W11's two halves to 3 + 16.5 = **19.5** — the values
[section 5.1](#51-summary-table) carries for those rows, and the visual review's two parts to
1 + 3.5 = **4.5**, the value [§7.1](#71-the-manual-visual-review) carries for its row.

**No overlap is added to this partition, and that is deliberate.** Where two sets can proceed with some
temporal overlap — 10a running on alongside W7 is the clearest case, since nothing in 10a consumes an
application artifact — the overlap is in **elapsed time**, not in effort. Effort does not shrink when
two streams run at once, so an overlap annotation inside an effort column would not reconcile against
any total. Overlap belongs to
[section 8.3](#83-the-critical-path-and-what-to-do-first-if-the-goal-is-to-narrow-the-estimate)'s
on-path versus off-path split, which is where it actually has an effect.

**Six properties of this shape are worth stating explicitly.**

- **Set 1 is the widest point in the plan by count; set 3 is the heaviest by effort.** Five independent
  items sit in set 1 and four in set 2, so a team of one gets 18 and then 72 IED of
  sequential work while a team of five gets the longest single item in each —
  **this is where staffing changes the elapsed shape most.** Set 3, by contrast, is 126 expected IED in
  two items, 109.5 of it in one workstream, so extra staffing there buys much less. **The re-derivation of
  [section 5.1](#51-summary-table) concentrated in the same places, for the fourth time**: set 1 absorbs
  +0.5 (gate 4a), set 2 absorbs +5 (gate 4b), set 3 absorbs +11.5 (W7's +7.5 and W11's manifest half's
  +4) and set 6 absorbs +1.5 (W12) — 0.5 + 5 + 11.5 + 1.5 = **+18.5**, the whole of the expected
  movement. **No set gained or lost a member and no edge moved an item between sets**, so the plan's shape
  did not change at all this round; only the weight of its two heaviest sets did.
- **W2's exit has two states, and they open different members of set 2.**
  [03 §4.2](03-modernization-roadmap.md) draws gate **2a** — the run recorded whatever it reports — and
  gate **2b**, a passing run re-verified after any repair. W6 consumes 2a, because a recorded failure is
  still a known starting condition to convert from; W4's gate **4b** consumes 2b, because it drives the
  legacy application over HTTP and a build that produces no running application leaves it nothing to
  characterize. So the two are concurrent **without** being opened by the same event, and a set-2 gate
  reading "W2 exited" for both would overstate what W6 needs and understate what 4b needs. **Gate 4b now
  has three entry conditions rather than two**, the third being **W3's exit**: its fixture manifest
  publishes per-entity counts, fixed keys, ranks, quantities and per-order totals **derived from the schema
  and data W3 extracts**, so a manifest authored before that extraction would be asserting invariants
  against a schema nobody had read. The edge does not move gate 4b between sets — W3 is in set 1 and 4b in
  set 2 either way — but it does mean W3 is a **mandatory predecessor** of the heaviest gate in the plan.
- **W4 does gate W6 — at gate 4a, not at gate 4b — and the distinction is the whole of why they are still
  concurrent.** [03 §4.2](03-modernization-roadmap.md)'s **40 edges** include `4a → W6` alongside
  `W1 → 4a`, `4a → 4b`, `2b → 4b`, **`W3 → 4b`** and `4b → W7`. **An earlier reading of this document asserted that the
  graph carried no `W4 → W6` edge at all, and that is withdrawn: the edge exists.** What survives is the
  conclusion, on a different justification — **W6 consumes gate 4a, the build-governance bootstrap, and
  not gate 4b**, so W6 does not wait on a legacy application that runs, on a captured baseline or on a
  green suite. Gate 4a closes in set 1; W6 and gate 4b then proceed concurrently in set 2. W6's own exit
  gate still deliberately requires neither a build of the legacy application nor a test run of it — the
  *unported* application cannot compile on the target framework, so a suite could not exercise it — which
  is why the edge it does consume is the one that produces `global.json`, the root `NuGet.config` and a
  proven locked-mode restore rather than the one that produces the baseline. Gate 4b therefore gates
  **W7 only**. Gate 4b still cannot be parallelized *internally* in proportion to its case count,
  because its deployment lifecycle, its store reset, its fixture dataset, its normalization, its
  diagnostic records and its handoff artifact are one coherent
  artifact that all **144** of its cases sit on top of — the cases divide, the thing they run on does not.
  **That property strengthened rather than weakened with the re-derivation**: the non-case components of
  this workstream are now **44 of its 66** expected IED — two thirds of it — so the serial fraction of W4
  grew again while the divisible one did not move at all, the three added cases being target-only.
- **W10 is three gates, not a straddle — and 10a is available a set earlier than an effort model would
  guess.** [03 §4.2](03-modernization-roadmap.md) models it as **10a** (provisioning), **10b** (schema
  application) and **10c** (the data load), and states that the graph contains **no partial or dashed
  dependency** anywhere. 10a's **only** entry is W5: provisioning creates the plan, the database, the
  identities and the principals, and not one of those consumes an application artifact, so it does not
  wait on W6 and sits in set 2 rather than alongside W7. 10b needs 10a *and* W7 exited, which is the
  first gate in the workstream that requires a publishable application; 10c needs W9 and then W8.
- **W11's two halves have different gates**, which is why it is split above. Provider selection needs
  only W1 and sits in set 1; manifest authoring waits for 10a, because the pipeline's
  deployment-time migration step needs the principal 10a establishes — and with 10a in set 2, the
  manifest half is available in set 3 alongside W7. Its band grew with input 21's second build agent and
  the redacted baseline record handed between them, then with input 28's thirteen-check verification
  gate, the secretless cache-table step and the artifact-protection configuration, and again this round
  with those thirteen checks' **literal evidence bounds**, the dedicated storage scope and key version the
  key-destruction check now requires, and the browser-install step the target-test job needs; the
  **architecture** of every one of those stages is
  [06 §6.4, §9.5 and §12.1](06-azure-hosting-recommendations.md)'s, and this set records only when the
  authoring becomes available. Its **provider-selection** half grew for a different reason and is why
  that half sits in set 1 rather than being folded into the manifest: 06 §6.4's removal of the
  cache-table SQL-authentication fallback makes credential-free federated identity a **blocking
  precondition** of the choice itself.
- **W9 precedes W8; only W12 is concurrent with them.** The two data migrations are **ordered, not
  concurrent**: [03 §4.2](03-modernization-roadmap.md) carries the edge `W9 → W8`, sequencing to the
  release ordering [06 §6.3](06-azure-hosting-recommendations.md) owns as step 5 of the provisioning
  order — the domain load first, then the Identity data migration. W12 waits on W7 and W10b and on
  nothing else, so it opens with W9 and needs a second pair of hands rather than a second data
  engineer. Running the two migrations in the ordered sequence is therefore not a staffing choice that
  can be bought out: **16 of the 22 expected IED in sets 6 and 7 is serial by decision, not by
  capacity.**

### 8.3 The critical path, and what to do first if the goal is to narrow the estimate

**The longest dependency chain**, computed as a longest path **by weight** over
[03 §4.2.1](03-modernization-roadmap.md)'s canonical edge inventory with every staged workstream held as
**separate nodes**, is

> **W1 → W2 → W6 → W4 → W7 → W11·2 → W10·2 → W16·2 → W8 → W12 → W13 → W15**

where `W11·2` is W11's manifest-and-rehearsal part, `W10·2` its schema-provisioning half, and `W16·2` the
personal-data mechanism stage — the last of which [03 §4.2.1](03-modernization-roadmap.md) already names
in exactly that form. The graph the path is taken over is that inventory's: **17 nodes and 47 edges,
acyclic, single root W1, terminals exactly `{W14, W15}`**, which becomes **19 nodes and 49 edges** once
W10 and W11 are split at the stage boundary the concurrency sets above already use — the two added edges
being the internal `W10·1 → W10·2` and `W11·1 → W11·2`. Both forms were checked acyclic by topological
sort before any weight was applied, because a longest path is only defined on a graph that has one.

**It is the same chain in all three columns.** Low, expected and high were maximized **independently**
over the same graph rather than taken from the expected column and re-weighted, because a wide band on an
off-path row can in principle overtake a narrow on-path one, and where that happens the chain is a
property of the column rather than of the model. Here it does not happen: the twelve nodes above are the
argmax at low, at expected and at high alike. That is recorded as a checked result, not assumed — if a
later reconciliation moves a band far enough to change it, the three chains have to be re-derived
separately and stated separately.

**Why W8 and not W9 at that step — stated correctly, because the earlier reading of this section had the
reason backwards.** It said W8 was taken "because W8's band is the wider of the two". It is not: **W9's is
the wider**, 11 expected IED against 9. The step is decided by what *follows* it.
[03 §4.2.1](03-modernization-roadmap.md) row 12 gives W8 two successors, `W8 → W12` and `W8 → W13`, while
row 13 gives W9 only `W9 → W13` — so the chain through W8 continues `W8 → W12 → W13` and weighs
`9 + 7.5 + 10 = 26.5` expected IED against W9's `11 + 10 = 21`. W9 is off the path despite the wider band.

**Which stage halves are charged, and why this figure differs from the one this section used to state.**
Three workstreams open in two parts, and they do not all pose the same question.
[03 §4.2.1](03-modernization-roadmap.md) already declares **W16·1 and W16·2 as two separate nodes**, so
for W16 the graph itself says which half binds and no convention is needed. W10 and W11 are single nodes
in that inventory, and the two edges into W10 are marked ***partial*** precisely because they bind its
DDL-applying steps rather than its environment half. The chain above therefore charges **W11's manifest
part and W10's schema-provisioning half**, and leaves **W11's provider gate and W10's environment half
off** the path — which is where the concurrency sets already place them, in sets 1 and 3, both reachable
before W7 exists.

The alternative reading charges a staged workstream **whole** wherever any part of it binds. That reading
is arithmetically well defined but it is an **upper bound rather than the longest path**: gate-inclusive
it gives **94.5 / 173.5 / 305**, exceeding the figure below by exactly `2.5 / 5.5 / 10` — which is precisely
W10's environment half (`2 / 4 / 7`) plus W11's provider gate (`0.5 / 1.5 / 3`), the two halves that do
not bind. Charging those to the chain asserts that work already available in an earlier set cannot be
absorbed by parallel capacity, which is the opposite of what the graph says, and it was the sole cause of
the discrepancy between the two figures this section previously carried. **The staged figure is the
critical path**; the whole-workstream figure survives only as that reconciliation.

Two gate conditions sit **on** the chain without being workstreams of their own, and both are counted
below: the approved-delta sign-offs ([§7.2](#72-the-approved-delta-sign-offs)), which W1's exit gate
requires, and the visual review and sign-off ([§7.1](#71-the-manual-visual-review)), which is exit
condition 5 of W7's gate. The visual **capture** is **not** on the chain: it sits inside W4's window as
concurrent work rather than as a step W4 waits on, so it appears in the off-path itemization instead.

| | Low | Expected | High |
| --- | ---: | ---: | ---: |
| Critical path, workstreams only | 84 | 153.5 | 270.5 IED |
| — plus the delta sign-offs on W1's gate | 4 | 7.5 | 13 IED |
| — plus the visual review and sign-off on W7's gate | 4 | 7 | 11.5 IED |
| **Critical path, gate-inclusive** | **92** | **168** | **295** IED |
| Off the critical path | 15 | 32.5 | 62 IED |
| **Critical path, full-band ordering — the upper bound** | 54 | **101** | 184 IED |
| **Its floor, with the W9 ⇢ W8 overlap taken in full** | 50 | **93** | 168 IED |
| Off the critical path, against the upper bound | 13.5 | 25.5 | 48 IED |
| **Critical path** | 65.5 | **121** | 216 IED |

The two rows reconcile exactly against [section 6.1](#61-the-totals): `92 + 15 = 107`,
`168 + 32.5 = 200.5`, `295 + 62 = 357`. So **32.5 of the 200.5 expected IED — 16.21 percent — sits off
the critical path** and can be absorbed by parallel capacity; the remaining 168 expected IED cannot, no
matter how many engineers are added. What is off it is **eight** items: W3 (4), W5 (2), W9 (11), **W10's
environment half** (4), **W11's provider gate** (1.5), W14 (2), **W16 stage 1** (2.5) and the **5.5 of
baseline capture** — itemized low to high,
`2 + 1 + 5 + 2 + 0.5 + 1 + 1 + 2.5 = 15`, `4 + 2 + 11 + 4 + 1.5 + 2 + 2.5 + 5.5 = 32.5` and
`8 + 4 + 20 + 7 + 3 + 3 + 6 + 11 = 62`, which is the same off-path row read from its parts rather than
from the subtraction. **Every node of the 19-node staged graph is in exactly one of the two lists** — the
twelve on the chain and the seven off it, plus the capture — which is what makes the reconciliation an
identity rather than a residual.

> **The most recent correction to [03](03-modernization-roadmap.md)'s graph — the `W5 → W7` edge of its
> §4.2.1 row 5 — does not move this chain, and the arithmetic is worth showing rather than asserting.** W7
> now has four predecessors instead of three, so the chain into it is the longest of them, and W5 is not
> that route: reaching W7 through W5 costs `W1 6 + W5 2 = 8` expected IED, while reaching it through
> `W1 6 + W2 2 + W6 3 + W4 38 = 49` costs over six times as much. W5 therefore stays **off** the
> path and inside concurrency set 1, its 2 expected IED stays in the off-path itemization above, and the
> chain, the 168 gate-inclusive figure and the 32.5 off-path figure are all unchanged by the edge. What the
> edge does change is one gate: set 4 now opens on **W3, W4, W5 and W6** exited rather than three of them,
> which is a correctness fix to the entry condition and not a change to any band.

**What moved this chain, relative to an earlier reading of it.** Six changes in
[03](03-modernization-roadmap.md)'s graph, none of which invented work:

- **W6 left the chain and then rejoined it, for two different reasons.** It left when `W4 → W6` was
  removed, because a skeleton has no legacy behaviour for the suite to assert. It **rejoined** when the
  converse edge `W6 → W4` was recognized: the suite is a project inside
  [04 §12](04-dotnet8-migration-strategy.md)'s project graph, so W6's 3 expected IED now sits between W2
  and W4 on the path. Its band did not change in either direction.
- **W12 joined the chain.** [03 §5 W12](03-modernization-roadmap.md)'s first exit condition proves the
  command's credential-repair path against the account W8's rehearsal neutralizes, so W12 follows W8 rather
  than running beside it, and its expected IED — **7.5**, once input 23's operator-host tests, input 30's
  rejection contract, exit condition 7's deployed census and the fourth separately gated invocation are all
  inside it — moved onto the path.
- **W11's manifest part joined the chain** — it is now a prerequisite of W10's DDL steps and of the data
  workstreams, so **parts 2–4** sit between W7 and W10's schema half, carrying **12** of W11's 13.5
  expected IED. Its **provider gate stays off**, in set 1, on nothing but W1's approval.
- **W16's mechanism stage joined the chain** — its conditions gate every workstream that processes
  personal data — **and its policy stage did not**. Stage 1 depends on W1 alone, so its **2.5** expected
  IED sits in the widest concurrency set; **3.5**, stage 2, is what the chain carries. This is not a
  convention choice: [03 §4.2.1](03-modernization-roadmap.md) declares the two stages as separate nodes,
  so the graph decides it. The earlier reading of this bullet also carried a 0.5 adjustment for W4's
  binding predecessor, and that adjustment no longer applies: with `W6 → W4` in the graph, W4's binding
  predecessor is the W2 → W6 pair at 5 expected IED rather than W16 stage 1 at 2.5, so stage 1 is off the
  path under any reading.
- **W10's schema half joined the chain**, and this is the change with the largest single effect on the
  expected figure. Because W16 stage 2's access-audit and deletion evidence must arrive in a **verified**
  sink, and W10's exit condition 7 is what verifies it, provisioning now precedes the personal-data
  mechanism and therefore precedes W8's and W9's rehearsals. **7** of W10's 11 expected IED moved from off
  the path onto it; the **4** of its environment half did not, because the two edges that bind it are the
  *partial* ones and they reach only the DDL-applying steps. Nothing was added: the same provisioning was
  always required before W13, and what changed is that it is now required earlier.
- **W13 grew and W8/W9 changed character**, because the production extraction, load and reconciliation
  moved into the cutover where it belongs, and the rehearsal stayed with the tooling.

**If the objective is to narrow the estimate rather than to shorten it, the first substantive action is
W3.** It costs **4 expected IED** and it is the input to three of the five low-confidence rows:

- It replaces the **evidence-rather-than-proof** qualification on the Identity column set
  [12 §5](12-migration-blockers.md) with fact, which is what W8's band is wide because of.
- It supplies the diff baseline W9's hard entry gate requires, in a repository where the migration
  source **ships no schema script**.
- It resolves two of the 23 blockers outright (input 12).

**W3 sits in concurrency set 2 and has exactly one prerequisite besides approval: W16 stage 1.**
It cannot begin before the personal-data policy is approved, because its entry requires the committed
credential and catalog databases to be **attached** — real personal data, at the earliest point in the
plan. That does not weaken the recommendation; it extends it by one item. Stage 1 costs **2.5 expected
IED**, needs nothing but W1, sits in the widest set and is an approval rather than engineering, so the
cheapest available reduction in the *width* of the total is to open stage 1 and W3 as a pair: 6.5 expected
IED that unblocks three of the five low-confidence rows and blocks nothing else while it runs. Neither
makes the project smaller; both make the estimate truer, which is a different and often more valuable
thing.

**And the ordering is not negotiable in the other direction.** Sequencing W3 ahead of stage 1 to save the
wait would mean the first copy of real credential data is made before anyone has said under what
restriction it may be held, for how long, or with what destruction evidence — which is the failure the
gate exists to prevent, arriving at the cheapest-looking moment.

### 8.4 The sequencing hazard in the visual baseline capture

**One sequencing hazard is priced in a band but becomes disproportionately expensive if missed.** The
visual baseline capture ([§7.1](#71-the-manual-visual-review)) must happen **while the legacy application
still runs**. It is **5.5 expected IED** inside W4 — concurrency set 3 — and it is not on the critical
path, because W4's 38 expected IED of suite authoring dominates the set it shares.

**The reason it cannot simply be done later is drift, not deletion, and an earlier form of this paragraph
had it wrong.** It said the capture "cannot be done later at any price, because the artifact it captures
ceases to exist". The artifact does **not** cease to exist:
[04 §12](04-dotnet8-migration-strategy.md)'s target map **retains** all three legacy editions read-only as
historical references and as the behavioural baseline, so the *source* of the thing being photographed is
still in the repository after cutover. What decays is the ability to **run** it, and that is a different
claim with different evidence:

- **Its build is unverified even now**, and the toolchain it needs is the constraint —
  Windows, the .NET Framework 4.8 targeting pack, Visual Studio 2022 MSBuild and a declared restore
  source ([10 §4](10-build-and-deployment-requirements.md), assumption **A2**). Every one of those is a
  property of a host that is maintained for as long as someone needs it and no longer.
- **Its data is a pair of committed file-attached databases** reached through a local instance
  [src/MVC5/MvcMusicStore/Web.config:12-13], and running it requires that instance to exist on that host.
- **Its client libraries are 2011–2013 pins** whose rendering depends on the browser as much as on the
  markup, and the browser is the one component of the capture nobody controls the version of. A capture
  taken a year after cutover in a browser two majors further on is not the same baseline even from
  identical source.

**So the hazard is that the capture becomes progressively less reproducible and eventually
unreproducible**, at a moment nobody announces, rather than that it becomes impossible on a known date.
That is worse for planning, not better: there is no deadline to miss, only a slope. Taking it inside W4
costs the 5.5 IED above; reconstructing a legacy runtime afterwards costs an unbounded amount for a
baseline that is, by then, arguably not the one the port was measured against. It remains the one item in
the plan whose *omission* is not repaired by spending more later — the correction is to the reason, not to
the conclusion.

---

## 9. The risk register

### 9.1 How to read an entry

**Every entry carries all seven required fields** — likelihood, impact, mitigation, contingency,
trigger, owner and affected workstream. No entry omits any of them. Where a risk is a **designed
consequence** rather than a possibility, its likelihood says so plainly instead of inventing a
probability for something already decided.

| Scale | Values |
| --- | --- |
| **Likelihood** | **Certain** (a designed consequence, not a probability) · High · Medium · Low |
| **Impact** | Critical (the migration's premise fails) · High · Medium · Low |

**Mitigation** reduces likelihood or impact before the fact. **Contingency** is what happens if it
occurs anyway. **Trigger** is the observable that says it *is* occurring — the thing to watch, stated so
that it can be watched rather than remembered.

#### 9.1.1 How the affected-workstream set is derived, so that it can be re-checked

**The seventh field is the one most easily left behind by an edit to the other six**, because a
mitigation or a contingency naturally acquires a workstream in the course of being made precise, and the
index cell does not follow. Earlier revisions of this register did exactly that: **R15**'s index cell named
four workstreams while its own field named seven, **R16**'s named three while its field named four, and
**R1**, **R2**, **R7**, **R8** and **R12** each had a field that omitted a workstream one of that entry's
own other fields relied on. So the set is not judged per entry. It is **derived**, by a rule with two parts
that are both checkable from the document:

1. **Field closure.** The **Affected workstreams** field must name every `W<n>` that appears in any of the
   entry's other six canonical fields — likelihood, impact, mitigation, contingency, trigger and owner. If
   a mitigation leans on a workstream's coverage, or a contingency runs through a workstream's pipeline, or
   a trigger fires at a workstream's gate, that workstream is affected by definition, and the field says so
   with **the role it plays** rather than the identifier alone.
2. **Index identity.** The **index cell in [§9.2](#92-register-index) is the same set**, in ascending
   order. The index is a projection of the entries, never a second statement of them — the same rule
   [03 §4.2.1](03-modernization-roadmap.md) applies to its edge inventory, and for the same reason.

Consumers and gates enter through part 1 rather than through a separate rule, because an entry that
depends on a consumer says so in the field that depends on it — and where that was not previously true,
the fix was to the *field*, not to the index. Where a consumer relationship is a graph fact rather than a
textual one, the field cites the edge: R2's field names **W4 and W6 as W2's two direct successors** in
03 §4.2.1's inventory, which is a lookup rather than an opinion.

**One declared exception, and it is the only one.** [R16](#r16--no-security-relevant-action-is-recorded-anywhere)'s
field mentions **W16** in order to say that the personal-data access-audit records are **not** in that
risk's scope — they are W16 stage 2's, per [03 §5 W16](03-modernization-roadmap.md). A `W<n>` named solely
to place it outside the risk is not a member, so W16 is absent from R16's index cell by design. Every other
entry's index cell equals its field exactly, and the check is mechanical: extract the identifiers from the
seven field rows of each entry and from the entry's index row, and compare the sets.

### 9.2 Register index

| ID | Risk | Likelihood | Impact | Owner | Affected workstreams |
| --- | --- | --- | --- | --- | --- |
| [R1](#r1--the-target-framework-support-window) | The target framework's support window | **Certain** | High | Engineering leadership *(approval decision)* | W1, W4, W6, W7, W10, W11 |
| [R2](#r2--the-migration-sources-build-reproducibility) | The migration source's build reproducibility | Medium | Medium | Build and release engineering | W2, W4, W6, W7 |
| [R3](#r3--the-absent-regression-baseline) | The absent regression baseline | High | **Critical** | Quality engineering | W4, W7, W13 |
| [R4](#r4--domain-data-migration-rollback) | Domain data migration rollback | Medium | **Critical** | Data engineering, with the data owner | W3, W9, W11, W13 |
| [R5](#r5--identity-migration-rollback) | Identity migration rollback | Medium | High | Data engineering, with security | W3, W8, W13 |
| [R6](#r6--security-hardening-versus-compatibility) | Security hardening versus compatibility | Medium | High | Security | W7, W11, W13 |
| [R7](#r7--the-narrowed-browser-matrix) | The narrowed browser matrix | **Certain** | Medium | **Product owner** *(approval decision)* | W1, W7, W14 |
| [R8](#r8--case-sensitive-path-resolution-on-the-target-platform) | Case-sensitive path resolution on the target platform | Medium | High | Engineering | W4, W5, **W7**, W10, W13 |
| [R9](#r9--cutover-re-authentication-and-anonymous-cart-loss) | Cutover re-authentication and anonymous-cart loss | **Certain** | Medium | Product and operations *(approval decision)* | W13, W15 |
| [R10](#r10--scoping-by-analogy-across-editions) | Scoping by analogy across editions | Medium | Medium | Engineering leadership, with this document | W1, W7 |
| [R11](#r11--the-effective-package-source-set-is-not-knowable-from-the-repository) | The effective package source set is not knowable from the repository | Medium | High | Build and release engineering, with security | W2, W6, W11 |
| [R12](#r12--no-observability-exists-during-the-cutover-itself) | No observability exists during the cutover itself | High | High | Operations and platform | W4, **W7 and W10** as the two halves of the mitigation, W13 |
| [R13](#r13--one-database-one-blast-radius) | One database, one blast radius | Low | High | Platform and operations *(approval decision)* | W10, W13 |
| [R14](#r14--a-reference-editions-retired-data-provider) | A reference edition's retired data provider | Low | Low | Engineering leadership | None while A7 holds |
| [R15](#r15--personal-data-governance-is-unowned) | Personal-data governance is unowned | High | High | **The data owner**, with security and legal *(approval decision)* | W1, W3, W4, W8, W9, W10, W13, W16 |
| [R16](#r16--no-security-relevant-action-is-recorded-anywhere) | No security-relevant action is recorded anywhere | **Certain** *(of the source)* | High | Security, with platform engineering | W7, W8, W10, W12, **W16** |
| [R17](#r17--the-interim-hosting-paths-stored-credential-and-a-time-box-with-no-box) | The interim hosting path's stored credential, and a time-box with no box | **Medium** *(conditional on the interim option being taken)* | **High** | **Security** | **None unless the interim path is selected**; if it is, W7 and W13 gate its retirement |
| [R18](#r18--the-browser-executed-half-of-the-scripted-cart-flow-one-pinned-engine-and-an-open-decision-on-the-other-two) | The browser-executed half of the scripted cart flow | **Certain** *(of the residual)* | **Low** | **Engineering**, with **Product** owning the open decision | **W1** for the recorded outcome; W7 for the harness; W11's manifest half |

**Two further risks exist and are deliberately *not* entries above. They are signposted here so that a
reader of this register cannot finish it believing they do not exist.** Both belong to the **interim
Windows hosting path**, for which [06 §5.5](06-azure-hosting-recommendations.md) selects **Path A** —
SQL-authentication credentials held behind a platform secret reference. That path is an *alternative* to
[03 §5](03-modernization-roadmap.md)'s workstream sequence rather than a workstream inside it; this
document estimates and risk-registers **that sequence**, so it correctly carries no band and no register
entry for a path outside it, and [§6.3](#63-what-is-deliberately-not-in-the-total) records the same
exclusion from the effort side. **[06 §13.4](06-azure-hosting-recommendations.md) owns both risks**, states
each with **every field an entry above carries** — likelihood, impact, mitigation, contingency, trigger,
owner and affected workstream — and adds the two things this register has no place for: the gate that must
be met before interim hosting may begin, and the event that ends the risk. That is a reasoned ownership
decision and this document does not reverse it.

| Held-elsewhere risk | Likelihood | Impact | Owner | Held by, in full |
| --- | --- | --- | --- | --- |
| **I1** — the time-boxed credential exception outlives its box: the stored SQL login Path A introduces survives the event that was supposed to end it, and a credential approved as bounded becomes a permanent one | Medium without control 4, low with it | **High** | **Security, jointly with operations** — the same pairing that owns the §5.5 selection, because ending the exception is an operational act and approving its continuation is a security one | [06 §13.4](06-azure-hosting-recommendations.md), risk I1, with §5.5 controls 1–5 |
| **I2** — the initializer source change is made incompletely and destroys the migrated database: `SetInitializer` is registered at **two** sites and closing one leaves the destructive path live | Medium if treated as a one-line edit, low under §5.6 precondition 2 | **Severe, unrecoverable without the backup** | **Engineering**, with the **data owner** accepting the residual | [06 §13.4](06-azure-hosting-recommendations.md), risk I2, with §5.6 precondition 2 |

**Why I1 in particular is worth arriving at from this direction.** It is the only risk anywhere in this
plan whose subject is a **stored credential in the target**, and the whole of
[09](09-security-assessment.md)'s and this plan's direction of travel is the removal of stored credentials.
A reader who came to this register looking for it — reasonably, because it is a security risk with a named
security owner — would otherwise find nothing and could conclude the interim path carries none. It carries
two, both fully stated, one section away.

### 9.3 The entries

#### R1 — The target framework support window

**This is the first entry because it is the only risk in the register that must be decided before any
other work begins, and because it is an approval decision rather than a technical alternative.**

[04 §2](04-dotnet8-migration-strategy.md) targets **`net8.0`** without hedging, which is correct: it is
what the user asked for, and a strategy document is not the place to substitute a different answer to
the question it was asked. [04 §2.2](04-dotnet8-migration-strategy.md) accordingly records that the
support window *"is an approval decision, and it belongs to 07"*. **This entry is that decision, and it
is left with the approver.**

**The dates, with their primary source, because this document owns the entry and a date without a
source is not an approval input.** .NET 8 is an LTS release that shipped **14 November 2023** on a
**36-month** LTS window, and its support **ends 10 November 2026**
[Microsoft Learn, *.NET releases and support policy*,
<https://learn.microsoft.com/dotnet/core/releases-and-support> — verified 2026-08-28]. That page states
the release type and the closing month; the **exact day** is on Microsoft's own support-policy table
[Microsoft, *.NET and .NET Core Support Policy*,
<https://dotnet.microsoft.com/platform/support/policy/dotnet-core> — verified 2026-08-28], which lists
.NET 8 as `LTS`, released 14 November 2023, currently in `Maintenance`, with an end-of-support date of
**November 10, 2026**. Both are cited because neither carries the whole fact alone. **.NET 9 shares that
same end date** on the same table, so it is not an alternative. The current LTS release at the time of
this assessment is **.NET 10**, supported through **November 2028**, per the first source.

**The consequence, stated plainly.** A migration delivered against `net8.0` lands on a runtime whose
support window closes on that date. After it, the application runs on an unsupported runtime — no
security patches, no servicing fixes — until it is moved again.

**Why this is not a one-line edit after approval.** An approver deciding between the two options needs
to know that retargeting later is a bounded but non-trivial exercise touching four things:

- **Every Microsoft-shipped EF Core and ASP.NET Core pin moves to the later release's servicing line.**
  [04 §7.1](04-dotnet8-migration-strategy.md) holds them to a single servicing band precisely because a
  mixed graph is unsupported, so they move together or not at all.
- **The SDK band and the build image change**, which means the pinned band in the tooling manifest
  [04 §3](04-dotnet8-migration-strategy.md) and the CI base image [W11](#w11--ci-provider-selection-then-pipeline-authoring)
  both move with it.
- **The System.Web adapters' compatibility surface changes with the target**, which matters only if the
  conditional incremental path of [05 §11.6](05-aspnet-core-migration-approach.md) is ever selected —
  but it changes the option's availability, so it belongs in the decision.
- **Behaviour validation must be re-run.** The W4 suite is the only evidence of behaviour preservation
  that exists; a runtime change invalidates the run, not the suite.

**The choice is the approver's, and both options are legitimate.** Delivering on `net8.0` as specified,
accepting a known and dated support cliff; or retargeting to the current LTS release **before** W6
begins, when the cost is a set of version values in documents nobody has implemented yet rather than a
retarget of a live application. **The second is materially cheaper if it is going to happen at all**,
which is the single most useful thing this entry can tell an approver — and it is the reason this risk
is first rather than merely present.

| Field | Value |
| --- | --- |
| **Likelihood** | **Certain.** The date is fixed and public. Nothing about this is probabilistic |
| **Impact** | **High.** Running on an unsupported runtime is a security and compliance exposure, not a functional defect. Impact is High rather than Critical because the application keeps working |
| **Mitigation** | Take the decision **before W6 begins**, while the target framework exists only as a value in unimplemented documents. If `net8.0` is confirmed, record the end date as a known commitment with a scheduled successor decision |
| **Contingency** | If the window closes before a move, the retarget is executed as the four-part change above, with the W4 suite re-run as its acceptance evidence. The suite is what makes the contingency viable at all |
| **Trigger** | The approval in W1 is taken **without** an explicit decision on this entry — that is the trigger, and it fires by silence. Secondarily: W6 beginning with no recorded decision |
| **Owner** | **Engineering leadership**, as the approver of W1. Not the port team: this is not an engineering trade |
| **Affected workstreams** | W6 and W7 directly; W10 and W11 through the runtime and build image |

#### R2 — The migration source's build reproducibility

**The one edition the entire port depends on has never been built on the prescribed toolchain, and this
entry says so rather than softening it.** [10 §1.2](10-build-and-deployment-requirements.md) — the owner
of per-edition build outcomes — carries the migration source's build assessment as **blocked pending a
Windows verification run**. This entry cites that status and restates neither the diagnosis behind it
nor any per-edition detail.

**What is established, and what is not, in the owner's terms.** Established: a **precondition
failure**. A clean checkout of the migration source commits no restored packages, so no build can start
until a restore succeeds against a source the repository never declares. **Not established: whether the
application compiles once restored.** The Mono `xbuild` result in
[10 §3.1](10-build-and-deployment-requirements.md) is explicitly *supplemental portability evidence
only* and is not a statement about a build on the prescribed toolchain; no row of it may be read as one.

**The risk therefore has four parts, and the first two are where it is open rather than residual:**

- **The verification run may fail, and nobody yet knows.** W2 produces that run
  ([section 5.2](#w2--mvc-5-build-reproduction-and-the-restore-precondition)); it does not consume a
  result. **A recorded failure closes gate 2a only.** [03 §5](03-modernization-roadmap.md) splits W2
  into two internal gates and states that W2 has not exited until 2b has closed, so the recorded
  outcome — pass or fail — discharges 2a and lets **W6** proceed from a known starting condition, while
  **W2's full exit, and therefore W4, stay blocked until gate 2b records a passing run** re-verified
  under 2a's own conditions. [Section 8.2](#82-concurrency-permitted-by-the-graph) reads set 2 that way,
  and this entry must not be read as saying that a recorded failure discharges the workstream.
- **What is genuinely open is not that the run may fail, but that it may fail *irreparably*.** A failure
  that gate 2b's repair loop closes is a handled outcome that costs W2 its high band and nothing further.
  A failure that cannot be repaired within 2b's bound — repair in the build environment, never in a
  tracked file under `src/MVC5/` — is not absorbed here at all: [03 §5](03-modernization-roadmap.md)
  escalates it to **W1 for a rebaseline**, because W4's exit, and with it the port's only regression
  baseline, is unreachable without a legacy application that runs. That is the residual this entry
  carries, and it is why the risk is live rather than already handled.
- **The restore source that would make a build succeed is a property of the host, not of the
  repository.** See
  [R11](#r11--the-effective-package-source-set-is-not-knowable-from-the-repository), which owns that
  finding. A build that passes on one host and fails on another indicts the source configuration.
- **Whatever the run reports, it says nothing about the views.** View compilation is disabled
  [08 §8.1](08-technical-debt-register.md), so the **29** views (input 6, file count) carry no
  build-time guarantee at all — [10 §3.2](10-build-and-deployment-requirements.md) binds that boundary
  onto the run in advance. Their first real check is W4's suite driving the legacy views over HTTP, and
  for their ported successors W7's target fixtures and the manual visual review.

| Field | Value |
| --- | --- |
| **Likelihood** | **Medium**, and the question is **open rather than unrepeated**: no one has observed the migration source compile on the prescribed toolchain. It is Medium rather than High for a bounded reason — the two configuration defects [10 §1.2](10-build-and-deployment-requirements.md) records are in a *different* edition, and the migration source's own observed failure was resolution of absent packages rather than anything in its source. Medium rather than Low because that hypothesis is untested |
| **Impact** | **Medium.** A defect found at W2 is cheap; the same defect found during W6 is disruptive but recoverable. It is not High because W2 sits before every line of port work |
| **Mitigation** | W2 executes the run from a clean checkout with an **explicitly declared** restore source, in **Debug and Release both**, and records tool versions, the source resolved, configurations built, per-edition outcome and warning and error counts — the five fields [10 §3.2](10-build-and-deployment-requirements.md) sets as its exit criterion |
| **Contingency** | Any defect the run reveals is triaged inside **gate 2b's repair loop** before W2 exits, and W6 may proceed from the 2a record while that loop runs. A defect therefore delays rather than derails on the W6 side, and holds W4 rather than merely delaying it, because W4 consumes 2b; the port has not started either way. If the failure is in the migration source's own configuration rather than in its restore, W6's band moves to its high case and [R3](#r3--the-absent-regression-baseline) gains weight, because W4 needs a legacy application that not only builds but **runs** — its `Category=Baseline` half executes against the running application on this host (input 21, assumption **A2**), so a host that compiles the source and cannot serve it leaves W4's gate unreachable rather than merely delayed. If the defect proves **irreparable within 2b's bound**, the contingency is not inside this model: [03 §5](03-modernization-roadmap.md) escalates to W1 and this document is **re-estimated rather than adjusted**, as [section 6.3](#63-what-is-deliberately-not-in-the-total) treats every input it does not assume |
| **Trigger** | **The run reports anything beyond the known restore precondition** — any error or warning that survives a successful restore. Secondarily: a build that succeeds on one host and fails on another. Thirdly, and it fires by silence: W6 beginning while W2's five-field 2a record is incomplete, or **W4 beginning on a recorded failure**, which is the reading gate 2b exists to forbid |
| **Owner** | **Build and release engineering** |
| **Affected workstreams** | W2, and its two consumers at the gate each actually needs — **W6 through gate 2a**, **W4 through gate 2b** ([03 §5](03-modernization-roadmap.md)) |

#### R3 — The absent regression baseline

**This is the register's Critical-impact entry, and it is the risk that determines whether any
behaviour-preservation claim can be substantiated at all.** [08 §12.3](08-technical-debt-register.md)
asks this document to carry it in exactly those terms.

**The repository contains no test of any kind** — no test project, no test file, no test-framework
reference, repository-wide. The command is in [A.2](#a2-the-absences-that-size-the-net-new-work) and
returns zero. **So nothing that exists today would detect a behaviour change.**

**What makes it Critical rather than merely High is the interaction with the blocker classification.**
[12 §2.3](12-migration-blockers.md) separates 22 blockers into 14 that fail at compile time and **8
that fail silently**. The compile-time group is self-announcing — the build stops. The silent group is
not: the request succeeds, the page renders, and a navigation property reads empty or a JSON field
reads undefined. **Against that group, a code review is not evidence.** The only instrument that
detects them is an assertion that fails when the resolution is absent, which is why
[03 §5](03-modernization-roadmap.md) makes W7's exit gate *demonstration* rather than inspection.

**One property of the required coverage sharpens this entry rather than softening it, and it is the
reason this risk is extended here instead of a sixteenth entry being opened.** **16 of the 39 coverage
surfaces are target-only** (input 14) — a share that grew as the behaviour set was completed — and for
those rows the suite is **not a comparison at all**: there is no legacy shape to characterize, only a
defect to reproduce, so the fixture pair cannot tell a correct assertion from a plausible one. **Five
of them are concurrency interleavings, fault injections and contention bounds** whose absence no
functional assertion notices — a cart that merges correctly under no contention passes every other row
in the table. **The
consequence is that for a sixth of the coverage the specification *is* the baseline**, and a
specification that is wrong is undetectable by the mechanism this entry relies on. That does not add a
risk; it says where this one bites hardest, and it is what makes the second clause of the trigger below
matter — those are the rows most easily deferred under schedule pressure, because nothing fails when
they are missing.

| Field | Value |
| --- | --- |
| **Likelihood** | **High.** Not that the baseline is absent — that is a verified fact — but that a behaviour change ships undetected if the port proceeds without one. Eight blockers are specifically silent |
| **Impact** | **Critical.** Without a baseline there is no evidence for the migration's central premise, and a regression's first detector is a user |
| **Mitigation** | W4 precedes the port, as [03 §4.1](03-modernization-roadmap.md) sequences it. The suite is HTTP-level and semantic so that **one suite characterizes both runtimes**, with volatile values normalized out and the **22** approved deltas recorded as expected differences rather than failures. For the 16 target-only surfaces, where no baseline exists to compare against, the mitigation is **review of the assertion against its owning contract in [05](05-aspnet-core-migration-approach.md)** rather than against the legacy fixture — the only check available for a row the source fails by construction |
| **Contingency** | **If the baseline cannot be made deterministic**, the response is to reduce ambition to a *provably* deterministic subset rather than to proceed unmeasured: fix the surfaces whose determinism is achievable — status codes, redirect targets, JSON property names and values, order totals, authorization outcomes — and for any surface that resists, **substitute an explicit manual verification with a recorded sign-off**, as [§7.1](#71-the-manual-visual-review) already does for rendering. A named, signed manual check is weak evidence; an automated suite that is silently flaky is *worse than none*, because it converts an unknown into a false assurance |
| **Trigger** | W4's exit gate cannot be met — specifically, a suite that passes and fails across runs with no code change. Secondarily: pressure to begin W7 while W4 is incomplete, which is the organizational form of this risk |
| **Owner** | **Quality engineering**, with the port team |
| **Affected workstreams** | W4 as the mitigation; W7 as the consumer; W13, which has no acceptance evidence without it |

#### R4 — Domain data migration rollback

**The risk is that W9's hard entry gate fails, or that its reconciliation does not balance**, and that
the response is attempted against a source of truth nobody has established.

**Two repository facts make this sharper than a generic data-migration risk.** The migration source
**ships no schema script at all**, and the other edition's two committed copies are **byte-identical to
each other and both unusable as written** [12 §5](12-migration-blockers.md). So the authoritative
schema exists **only inside a committed binary**, and an EF Core initial migration generated from the
ported model cannot be assumed to match it — column types, precision, nullability, identity, keys,
defaults, delete rules and indexes may all differ, silently and individually.

**The freeze-interval problem is part of this entry, not a footnote — and it is named in the direction the
window runs.** [03 §5 W9](03-modernization-roadmap.md) condition 4 fixes that order as **drain → stop →
final recovery position and final extraction → load**, so the interval to account for is the
**drain-to-extraction** one, and the design chosen here is the third of the three that were available:
not a delta pass over rows written after an extract, and not a write freeze bolted on, but **the drain
itself**, which makes such rows impossible rather than recoverable. That choice is what makes the check
`no write reached the source between the drain and stop and the final extraction`, expected **empty by
construction** because every source-side write the runbook sanctions — the credential neutralization and
its in-window re-verification — happens **before** the stop. **An earlier form of this paragraph reasoned
in the reverse direction**, from an extract taken first and a cutover later, which described a delta
design nobody chose: it would have budgeted for rows the drain prevents while leaving the unsanctioned
write the drain does *not* prevent unchecked. Either way the decision is **W9's to make and record** rather
than an operational detail discovered in the window, and getting it wrong loses orders — which is why this
entry's impact is Critical.

**Where the reversal boundary falls, because this entry used to state it wrongly.** An earlier reading of
the contingency below said that the retained source makes rollback "a redirection to the source rather
than a restore" and left it there, unqualified. That holds only until traffic is admitted.
[06 §11.5](06-azure-hosting-recommendations.md) now defines **three reversal regimes**, and the boundary
between the second and the third is a point of no return established by **evidence** rather than by a
clock. **The three predicates are stated here as [06 §11.5](06-azure-hosting-recommendations.md) states
them — mutually exclusive, exhaustive and decidable from the evidence alone**, because a summary that put
regime C at "any accepted write" while regime B still admitted replayed cart writes would leave the two
overlapping on exactly the class that matters:

- **Regime A** — traffic not yet admitted.
- **Regime B** — after admission, **and** the determination finds either **no accepted write at all**, or
  accepted writes that are **only** signed-in cart writes **whose reverse-replay into the source store has
  completed and verified**.
- **Regime C — roll-forward only** — after admission, and **anything else**: any accepted write that is not
  a signed-in cart write, or any signed-in cart write whose replay did not verify **or could not be run**.

Every reversal after admission falls in exactly one of B and C, and which one is read off the
determination and the replay's step-6 result rather than judged. An
accepted write is defined by **what was promised** so the test is decidable rather than convenient: an
order placed, an account registered, **a password changed**, **a cart changed by a signed-in user**, or an
administration write to the catalog — each one something the new application **confirmed to a user**.
Platform state — a session-cache row, a key-ring row, a telemetry record — is not an accepted write and
does not trip the regime, because nothing was promised on its account.

**There is no signed-in-cart exception, and an earlier form of this entry asserted one.** It read that cart
changes are "enumerated and reported but do not trip the regime", citing
[06 §11.4](06-azure-hosting-recommendations.md)'s approved anonymous-cart loss as the warrant. That warrant
does not extend that far: the AAP's approved-delta set grants cart loss for **anonymous** carts only and
states that signed-in carts are unaffected because their key is the login name
[src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:165-167]. A regime B reversal that returned to the legacy
store while a signed-in user's post-admission cart change stayed behind in the new one would destroy data no
approval covers. **So a signed-in cart write is decisive like every other accepted write, and it has exactly
one relief**: [06 §11.5.4](06-azure-hosting-recommendations.md)'s **six-step reverse replay** carries the
final per-key cart state back into the source store and verifies it per key, and its **step 6 is a
mechanical branch** — all keys verified admits regime B with nothing lost; any key unverified, **or the
replay unable to run at all**, is regime C. There is no third branch in which the write is reported and
dropped. **Anonymous** cart writes are the one class that is reported, does not trip the regime and is not
replayed, because an anonymous cart's browser-to-cart link lives only in the new application's session and
is unreachable after the reversal whatever the rows say.

**The evidence is not a set of row-count probes, and it is not a scan of current rows either. Both
corrections matter to this entry, because each one on its own still admits an undetected accepted write.**

- **Counting rows cannot see a change that leaves a count unchanged.** A successful password change leaves
  the account count exactly as the reconciliation recorded it, and a quantity-only cart update leaves both
  the `Carts` row count and its key set unchanged — so a count-based instrument would report zero, admit
  regime B, and discard writes nobody counted.
- **Scanning current rows cannot see a row that no longer exists.** A moved `rowversion` or a recent
  `ModifiedUtc` lives *on a row*, so a **delete** leaves nothing to scan. That gap is not closed by the
  audit stream for every surface, because the audit stream sees a delete only where the deleting surface
  emits a class: album deletion emits `ADMIN-4003`, and **cart removal emits none**. A signed-in visitor
  who removed the last item from their cart after traffic was admitted would leave **no trace in either** —
  no current row, no audit class — and would be invisible to the determination.

**So the instrument is the union of two signals, and it is the union that is complete rather than either
signal** ([06 §11.5.4](06-azure-hosting-recommendations.md)). Neither is complete alone, and 06's own
six-class table says which one carries each class:

- **Signal 1 — the audit stream**, read from `dbo.SecurityAuditLog` itself rather than from the telemetry
  workspace, which says what changed, who changed it and **whether the application confirmed it** — the
  distinction no row comparison can make. It is **not** a complete record of deletes: it sees one only
  where the deleting surface emits a class, so album deletion appears as `ADMIN-4003` and **cart removal
  appears not at all**.
- **Signal 2 — row-level change tracking with delete-visible history**: the `rowversion` column,
  `Orders.SubmissionId`, and **system versioning on every mutable table in both contexts**, whose history
  table retains the prior state of every updated row and the **full final state of every deleted row**. It
  cannot distinguish a row that changed from a row whose change was **confirmed to a user**, which is
  signal 1's contribution and not obtainable from rows.

**The union closes each signal's gap with the other's coverage, per class.** An album delete is seen by
both; a **password change** is seen by both, and by neither of the withdrawn aggregate probes; a
**signed-in cart change — including a removal, and including a quantity-only update** — is seen by
**signal 2 alone**, because the cart surface emits no audit class and this model does not invent one; an
anonymous cart change is seen the same way and is distinguished from the signed-in case by the key test
rather than by a separate instrument. So an implementation that queried only the audit stream would miss
every cart change, and one that queried only current rows would miss every delete and could not tell a
confirmed write from an attempt.

**The two signals are reconciled rather than read side by side**, which is the part that makes the union an
instrument instead of two reports: every row at or above the watermark is matched to an audit record by
**business key** — `OrderId`, account name, `AlbumId`, cart key — and every audit record at `Success` in
the window is matched to a row or a history row by the **event key** its record carries, with each
unmatched item on either side resolved to one of 06's six classes or escalated. An unmatched row is
either a class signal 1 does not cover, in which case it is expected and named, or a write nobody
confirmed, in which case it needs an explanation before any regime is declared. `ModifiedUtc` is an
enumeration aid in that reconciliation and not the decision: the decision compares against the single
`MIN_ACTIVE_ROWVERSION()` watermark taken at traffic admission, so no clock, time zone or skew question
enters it. Signal 2's mechanism is authored by
[05 §5.1](05-aspnet-core-migration-approach.md)'s two per-context `AddChangeTracking` migrations, which is
why it appears in [W9](#w9--domain-data-migration-tooling-rehearsed-against-a-copy)'s basis rather than
as an operational task; signal 1's mechanism already exists, and what W9 owns for it is the query set and
the reconciliation, not the sink.

Restoring the source after an accepted write would delete a financial commitment the customer has already
been told succeeded, so it is not a rollback step at all — and 06 **no longer offers it as one at any level
of authorization**, having removed the declared data-loss rung that once did. The
contingency below is stated in those terms, and the mitigation adds what has to exist **before** the
window for the boundary to be decidable inside it.

| Field | Value |
| --- | --- |
| **Likelihood** | **Medium.** The domain is six entities with no exotic mapping, but the diff is being run for the first time against a schema that has never been extracted |
| **Impact** | **Critical.** The data at stake is orders and customer PII, and the destructive initializer [08 §6.1](08-technical-debt-register.md) is proof that this database has no automated protection against a model mismatch today |
| **Mitigation** | W3 first, so the diff has an authoritative baseline. The **generated-schema diff must pass before any data is loaded** — [03 §5](03-modernization-roadmap.md) calls this the hard gate. Load in dependency order; reconcile row counts per table **and financial totals per order**; rehearse the whole sequence against a representative dataset per assumption **A3**. And the **reversal instrument is built before the window rather than improvised inside it**: the accepted-write evidence of [06 §11.5](06-azure-hosting-recommendations.md) rests on the change tracking **and the delete-visible system-versioned history** that [05 §5.1](05-aspnet-core-migration-approach.md)'s **two** per-context `AddChangeTracking` migrations add — one in the catalog set and one in the Identity set, because a migration belongs to exactly one context — and on the watermark [06 §11.3](06-azure-hosting-recommendations.md) records at traffic admission; it is evaluated against the loaded copy as part of the rollback W9's exit gate requires to be *performed* rather than described, **and the reverse replay of [06 §11.5.4](06-azure-hosting-recommendations.md) is rehearsed there in both directions of its step-6 branch rather than only in its passing one**; and the forward ladder's second rung — the last known-good revision redeployed against an unchanged database — is exercised once inside W11's release-path rehearsal. **That rung's availability** depends on [06 §6.7](06-azure-hosting-recommendations.md)'s additive-within-a-release constraint — a release that dropped or renamed a column the previous revision reads would make it unusable — so the constraint is a requirement on every migration W9 authors rather than a style preference |
| **Contingency** | **Three regimes, and which one applies is established by evaluating the change-tracking evidence rather than by judging how long ago traffic was admitted** ([06 §11.5](06-azure-hosting-recommendations.md)). **Before traffic admission**, the source database is unmodified and retained, so reversal is a redirection to the source rather than a restore, and reconciliation failure stops the cutover outright — it does not get accepted with a note. **After admission while that evidence shows no accepted write that has not been carried back**, the same redirection is still available and **no business data is lost, signed-in cart changes included**: where the evidence finds signed-in cart writes, [06 §11.5.4](06-azure-hosting-recommendations.md)'s **verified reverse replay** is run first — enumerate the affected keys from current rows **and** the `Carts` history table, replay each key's final state into the source inside one transaction, verify the multiset per key — and **its step 6 decides the regime**: all keys verified admits regime B, any key unverified or the replay unable to run is regime C. Only **anonymous** cart writes are reported and not replayed. **From the first accepted write, recovery is roll-forward only, with no exception under any sign-off**, by [06 §11.5](06-azure-hosting-recommendations.md)'s four rungs in order: stop admitting new work; redeploy the last known-good revision against the same database; repair the data or schema forward through the release path under the deployment principal; and, **if the target itself is damaged, point-in-time restore of the target followed by a replay of the interval between the restore point and the damage**. That replay is not a property of the restore: PITR reproduces the state at `T`, so the writes after `T` are exactly the rows the restored database does not have. Its source is the **retained damaged original**, which PITR never overwrites and which system versioning leaves holding full-state history — so **retaining, locking and not dropping the damaged database is a step of the procedure** ([06 §11.5.4](06-azure-hosting-recommendations.md) part 4a), and the audit store supplies only the confirmed-or-not attribution the rows cannot carry. Where that replay cannot run — history destroyed with the current rows, or an interval older than the retention period — the recovery point for the affected table is `T` and the residual is a **declared data-loss event under a procedure and an RPO the data owner approves at approval time**, never one proposed during the incident. **Returning to the legacy application is not a rung of that ladder at all, at any level of authorization.** An earlier version of this entry offered it as a fifth rung gated on the data owner's sign-off and a reverse-delta export; 06 has removed that rung, because a sign-off does not make discarded confirmed orders recoverable and its availability was the thing most likely to be reached for under pressure |
| **Trigger** | **Two exact conditions on the schema diff, replacing an earlier "does not converge after a bounded number of iterations" that named no bound and therefore could not fire.** The **gate** condition: the generated-schema diff against [W3](#w3--authoritative-schema-extraction)'s extracted schema is **non-empty at the point W9's gate is evaluated** — one differing object is the trigger, because a data load onto a schema that differs from the source's in any object is the failure this risk describes. The **in-progress** condition, which exists so the escalation does not wait for the gate: **three successive regenerations in which the count of differing schema objects does not strictly decrease**. Both are mechanical — the first is an emptiness test on the diff, the second a monotonicity check on an integer — so either fires without a judgement. And on the data side: reconciled row counts or per-order totals differing by any amount. **Any** discrepancy is the trigger — a small one is not a rounding artifact in a financial total. Two further triggers concern the reversal rather than the migration, and both are the organizational form of this risk: a reversal **proposed after traffic admission without the change-tracking evidence having been evaluated**, elapsed time being offered in its place; and the watermark **not recorded** at [06 §11.3](06-azure-hosting-recommendations.md)'s traffic-admission step, or **either** `AddChangeTracking` migration not applied — the Identity one is the easier of the two to omit and it is the one carrying the password change this determination exists to catch — any of which leaves the regime undecidable at the exact moment it has to be decided |
| **Owner** | **Data engineering**, with the **data owner** as approver, per [03 §5](03-modernization-roadmap.md) W9. There is no longer a reversal an engineering role may not authorize, because there is no longer a reversal that discards accepted writes: 06's removal of the declared data-loss rung turned an approval question into an unavailable option |
| **Affected workstreams** | W3 as the precondition, W9 as the work and the home of the detection-against-copy rehearsal, W11 where the forward ladder's second rung is exercised, W13 as the consumer that evaluates the evidence and records its regime |

#### R5 — Identity migration rollback

**The risk is that the credential store cannot be migrated as designed, and that it is discovered after
cutover** — when the symptom is users unable to sign in.

**Its distinguishing feature is that this entry's source schema is the one thing in the assessment that
is explicitly qualified.** [12 §5](12-migration-blockers.md) records the Identity column set as
**evidence rather than proof**, arrived at by inspecting a binary rather than by querying a catalog.
That qualification is cited here, not re-derived, and it is exactly what W3 exists to remove.

**Three concrete failure modes, each with a different response.**

- **Normalized-column collisions — and the two columns are not treated alike, which is the part an
  estimate gets wrong.** The target schema's normalized user-name and e-mail columns do not exist in
  the source's Identity generation, and two accounts differing only in case normalize to the same
  value. [05 §5.5](05-aspnet-core-migration-approach.md) resolves the two separately: a **normalized
  user-name collision aborts the migration before anything is written**, because the user name is the
  invariant order ownership keys on and silently dropping an account is indistinguishable from a user
  who forgot their password; **duplicate normalized e-mails do not abort** — the selected policy does
  not require unique e-mail, so they insert legitimately and are **reported** for the data owner rather
  than rejected. The risk this entry carries is therefore an abort *on the right column*: a migration
  built to abort on duplicate e-mails would fail a load the policy permits, and one built to tolerate
  duplicate user names would lose an account's order history.
- **Password-hash validation.** The target framework's default hasher is expected to validate the
  older format and rehash on successful sign-in. **That is an expectation about the shipped hashes, and
  assumption A10 states it as one.** Its acceptance test is not a code review but a successful sign-in
  by a pre-existing account.
- **Columns with no source value.** Concurrency and security stamps, two-factor, lockout,
  access-failed-count and confirmation columns need **defined origins** rather than whatever the
  target's defaults happen to be.

| Field | Value |
| --- | --- |
| **Likelihood** | **Medium.** The store is small and the transition is a well-trodden one, but it is being performed against a schema qualified as evidence rather than proof, and against hashes whose format has not been tested. **The rating does not move on the stronger reconciliation below**: keyed sets and digests improve *detection*, and neither the unproven schema nor the untested hash format becomes less likely because a mismatch would now be caught |
| **Impact** | **High.** A failed Identity migration locks users out and, if the collision case is mishandled, loses accounts. High rather than Critical because the source store is retained and order history keys on the user name rather than on an identifier |
| **Mitigation** | Create the target tables **fresh and populate them** rather than altering the source in place, so the source survives as a rollback position and reconciliation can compare two live datasets. Preserve **user names exactly** — order ownership compares against them [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:71-73]. **Abort before writing on a normalized user-name collision; report duplicate normalized e-mails rather than rejecting them**, per [05 §5.5](05-aspnet-core-migration-approach.md). Reconcile to [05 §5.6](05-aspnet-core-migration-approach.md)'s standard — **keyed sets and per-row digests** across users, roles and assignments, with counts and the administrator's membership asserted specifically but **not** treated as sufficient, and no raw hash, stamp or PII value in any artifact |
| **Contingency** | Source tables are retained until reconciliation passes, and **any mismatch blocks cutover** [05 §5.6](05-aspnet-core-migration-approach.md) rather than being accepted with a note. If hashes do not validate, the fallback is a **forced credential reset for affected accounts** with out-of-band notification — recoverable, but a user-visible event that must be approved rather than improvised, and the reason A10 is stated as an assumption instead of assumed |
| **Trigger** | **A pre-existing account cannot sign in after migration.** That single observable covers the hash, the normalized columns and the stamp defaults at once, which is why it is the acceptance test. Secondarily: any normalized user-name collision detected during the migration rehearsal; or any keyed-set or row-digest mismatch on users, roles or assignments — a class of corruption the count comparison alone would pass |
| **Owner** | **Data engineering**, with **security** |
| **Affected workstreams** | W3 as the precondition, W8 as the work, W13 as the consumer |

#### R6 — Security hardening versus compatibility

**The risk runs in both directions, and that symmetry is the entry.** [05 §6.1](05-aspnet-core-migration-approach.md)
requires every authentication, cookie, password and lockout policy item to be set **explicitly** and
labelled either *preserved* or *deliberate hardening*.

- **Hardening that arrives as an unannounced framework default is a behaviour change nobody
  approved.** The target framework's defaults for password complexity, lockout and cookie attributes
  are not the source's effective behaviour. Adopting them by *not writing anything down* is
  indistinguishable, in a change log, from a considered decision — and its first symptom is existing
  users unable to sign in or register.
- **Preserving a weak default silently perpetuates it.** The mirror error: labelling everything
  *preserved* to avoid user impact carries the source's weak posture into a new system, and does it
  under the appearance of rigour.

**The mitigation is therefore a labelling discipline, not a policy choice** — the policy is
[05 §6.1](05-aspnet-core-migration-approach.md)'s to state and security's to approve. What this entry
insists on is that **no row is left unlabelled**, because an unlabelled row is precisely where both
failure modes hide.

| Field | Value |
| --- | --- |
| **Likelihood** | **Medium.** The requirement is explicit in 05, which reduces it; the failure mode is an omission rather than an error, and omissions survive review easily |
| **Impact** | **High.** Either direction is serious: an unapproved lockout policy is an availability incident, and a perpetuated weak default is a security finding that the migration was the opportunity to close |
| **Mitigation** | The policy table of [05 §6.1](05-aspnet-core-migration-approach.md) with **every row labelled** *preserved* or *hardening*, reviewed and signed by security as part of W7. Any row that cannot be labelled is treated as an open question blocking W7's exit, not as a default |
| **Contingency** | If an unlabelled divergence is found after cutover, it is resolved as a policy decision with a recorded label — and, if it is user-affecting, communicated. **Correcting the value itself requires an application deployment**, for the reason below; the remedy is therefore a release through [W11](#w11--ci-provider-selection-then-pipeline-authoring)'s pipeline rather than a configuration edit, and the only thing that makes it quick is that a policy-only change is a small edit traversing an already-authored gated path |
| **Trigger** | Any policy row reaching W7's exit gate **without a label**. Secondarily, post-cutover: a rise in failed sign-ins or lockouts, or a registration rejected under a complexity rule the source accepted |
| **Owner** | **Security** |
| **Affected workstreams** | W7 as the labelling gate; **W11**, because its pipeline is the only route a correction can take; W13 as the point where user impact becomes visible |

> **The contingency is a deployment, and an earlier form of this entry said it was not.** It read that
> "the persisted policy is configuration rather than code, so correcting it does not require a deployment
> of the application". That is not the design being estimated:
> [05 §6.1](05-aspnet-core-migration-approach.md) opens with "**every value below is the value written into
> `Program.cs`**", and [05 §2.4](05-aspnet-core-migration-approach.md) sets every `IdentityOptions` value in
> the identity setup action. Nothing in that table is bound from `appsettings`, from an environment variable
> or from any mapped configuration source, so **no configuration edit can change any of that table's 25 rows**
> — including the ones whose failure mode is an availability incident, the lockout threshold and duration
> among them. A contingency that names a lever that does not exist is worse than none, because it is the one
> that gets reached for during the incident.
>
> **Two consequences, and they pull in the same direction.** The recovery time for a mis-set policy is a
> **release cycle**, which raises the value of catching it at W7's exit rather than after cutover and is why
> the mitigation makes an unlabelled row *block* that exit. And the remedy consumes W11's pipeline, which is
> why W11 is now in the index set above.
>
> **If security wants a configuration-time remedy, the change is not this document's to specify.** Binding
> the policy from mapped configuration with startup validation would make correction a restart rather than a
> release — and it is a change to [05 §6.1](05-aspnet-core-migration-approach.md)'s design, to be taken
> there, with its own trade stated rather than assumed: an externally mutable authentication policy is
> itself a control surface, needing change control and an audit trail of its own, so it moves risk rather
> than removing it. This entry records the design as it stands and the recovery cost that follows from it.

#### R7 — The narrowed browser matrix

**This is the one dependency decision in the whole assessment that changes *who can use* the
application**, which is why it is a register entry with a named approval owner rather than a line in a
dependency table.

Two package removals and a styling-framework major version, all owned by
[04 §8, §9](04-dotnet8-migration-strategy.md), narrow the set of clients the application supports.
[06 §10.4](06-azure-hosting-recommendations.md) states the resulting matrix and explicitly records that
**07 carries the compatibility loss as a risk with a named approval owner — the product owner — and
that accepting it is not 06's to do.** This entry is that carriage. **The matrix itself is not restated
here**; it is stated once, by 06.

**What is being decided.** Every other change in this assessment is invisible to a user or is a
deliberate interface delta on one path. This one **removes a class of client entirely.** A user on an
unsupported browser does not see a degraded page — the layout does not work. That is a product
decision about the addressable audience, not an engineering trade, and it cannot be mitigated
technically without reversing the dependency decision behind it.

**The narrowing is the styling framework's own published position, not an inference from it.** The
target major's browser-support page states, under its own *Internet Explorer* heading, *"Internet
Explorer is not supported. If you require Internet Explorer support, please use Bootstrap v4."*
[Bootstrap, *Browsers and devices*,
<https://getbootstrap.com/docs/5.3/getting-started/browsers-devices/> — verified 2026-08-28]. That is
why the contingency below reverses the **framework major** rather than the matrix: the matrix is
downstream of a support statement this assessment cannot negotiate.

| Field | Value |
| --- | --- |
| **Likelihood** | **Certain.** This is a designed consequence of decisions already recorded in 04 and 06, not a possibility. The open question is whether it is **approved and communicated**, not whether it happens |
| **Impact** | **Medium**, and genuinely unknown in the absence of traffic data — see the trigger. If a material share of real users are on an unsupported client, the impact is High and the decision changes |
| **Mitigation** | **Obtain the product owner's explicit approval in W1**, as one of the 19 delta sign-offs of [§7.2](#72-the-approved-delta-sign-offs). Support the decision with **actual client analytics** if any exist, so it is taken against evidence rather than assumption. State the matrix in the deployment documentation (W14), so an unsupported client generates a policy answer rather than a defect investigation |
| **Contingency** | If the product owner declines, the decision to be reversed is the **styling-framework major version** in [04 §9](04-dotnet8-migration-strategy.md) — not this matrix, which is downstream of it. That reversal carries its own consequence, which [09](09-security-assessment.md) owns: remaining on an out-of-support framework. There is no contingency that keeps both |
| **Trigger** | W1 completing without the product owner's recorded decision on this delta. Post-cutover: support contacts from unsupported clients, which the matrix in the deployment documentation is what converts into an answer |
| **Owner** | **The product owner**, alone. Explicitly not engineering, and explicitly not this document |
| **Affected workstreams** | W7 as the implementation, W14 as the statement of the matrix |

#### R8 — Case-sensitive path resolution on the target platform

**The risk is a class of failure that is invisible on the development platform and total on the target
one.** The primary hosting target [06 §2](06-azure-hosting-recommendations.md) resolves paths
case-sensitively; the current web host does not. Every asset and view path whose case does not match
its file exactly works today and 404s there.

**The platform behaviour is documented rather than assumed.** The static-file middleware inherits the
filesystem's case rules: *"The URLs for content exposed with `UseDirectoryBrowser` and `UseStaticFiles`
are subject to the case sensitivity and character restrictions of the underlying file system. For
example, Windows is case insensitive, but macOS and Linux aren't"*
[Microsoft Learn, *Static files in ASP.NET Core*,
<https://learn.microsoft.com/aspnet/core/fundamentals/static-files> — verified 2026-08-28]. So the
failure is not a platform quirk to be worked around; it is the documented contract, and the mismatch is
in the application's references.

**It is not theoretical, and one instance is already confirmed.** A stylesheet is registered in
lowercase as `~/Content/site.css` [src/MVC5/MvcMusicStore/App_Start/BundleConfig.cs:28], while the
tracked file's real name is capitalised — `git ls-files 'src/MVC5/MvcMusicStore/Content/*'` returns
`src/MVC5/MvcMusicStore/Content/Site.css`, reproduced with its output in
[A.5](#a5-the-corroborating-case-mismatch-r8). A file name is not a line, so the mismatch is cited at
the reference that is wrong and evidenced by the command that reads the real name.
[06 §3.4](06-azure-hosting-recommendations.md) therefore makes the audit a **precondition** for the
primary target rather than a caveat on it, and [12 §2.3](12-migration-blockers.md) classifies path
casing among the **silent** blockers.

**Independent corroboration, and it is stronger than a second typo would be: the repository's own
metadata already resolves differently on the two filesystems.** `.gitignore` spells its rule
`nuget.exe` in lowercase [.gitignore:28] while the tracked path is
`src/MVC4/MvcMusicStore/.nuget/NuGet.exe`. Asked correctly — *"By default, tracked files are not shown
at all since they are not subject to exclude rules; but see `--no-index`"*
[Git, *git-check-ignore Documentation*, <https://git-scm.com/docs/git-check-ignore> — verified
2026-08-28], so the default form on a tracked path establishes only that the path is in the index and
`--no-index` is required to test the rule — that one rule gives two
answers: `git check-ignore --no-index -v src/MVC4/MvcMusicStore/.nuget/NuGet.exe` reports
`.gitignore:28:nuget.exe` and exits 0 on this checkout, where `core.ignorecase` is `true`, and the same
command under `-c core.ignorecase=false` matches nothing and exits 1. Both are reproduced with their
output in [A.5](#a5-the-corroborating-case-mismatch-r8);
[11 §3.7.2](11-cloud-readiness-assessment.md) owns the measured behaviour and
[08 §10.7](08-technical-debt-register.md) owns the tracking as debt. One rule, one path, two results
decided by a filesystem property is this risk's exact failure mode, already present in the repository
before a single asset path is considered.

| Field | Value |
| --- | --- |
| **Likelihood** | **Medium** for an *undetected* mismatch surviving to production, and the corrected corroboration keeps it there rather than moving it. That one asset mismatch exists is certain, and the ignore-rule experiment above shows the repository already contains a path expression that resolves one way here and another way on a case-sensitive filesystem — the identical failure mode, in metadata instead of markup. What remains uncertain is only how many such expressions nobody has looked at, which is the audit's whole purpose |
| **Impact** | **High.** A missing stylesheet renders every page unstyled, and a mismatched view path is a 500. The failure is total for the affected path and absent from every case-insensitive test of it |
| **Mitigation** | W5 audits **all** of it — the **11** bundling helper sites, the **4** `@Url.Content` sites and the **27** static assets (inputs 8, 11 and 7; site and file counts) plus view paths — and, critically, makes the audit **repeatable as a pre-deployment check** rather than a one-time sweep, because a new mismatch can be introduced by any later change. The coverage assertion of [05 §12.4](05-aspnet-core-migration-approach.md) asserts static assets resolve **case-sensitively**, which a case-insensitive check would pass wrongly. That row is one of the **mixed** rows rather than a target-only one: case **`23a`** — every rendered reference requested at its rendered casing — has a legacy side and is therefore authored and run in W4, while case **`23b`** — a deliberately wrong case, which must **404** — is **target-only** and lands on W7's exit gate. The split matters to this risk, because `23b` is the case that detects a case-insensitive agent, and an agent that passes `23a` while failing to reject `23b` is the exact condition this entry describes |
| **Contingency** | A mismatch reaching production is corrected in the referencing code and redeployed; the repeatable check is then extended to cover the class that escaped it. Because W5's exit criterion is a case-sensitive serve with no static-asset or view 404, the correction is verifiable rather than hopeful |
| **Trigger** | Any 404 for a static asset or view on a case-sensitive serve — which is why the check must run on a case-sensitive filesystem in the pipeline and not on a developer machine |
| **Owner** | **Engineering** |
| **Affected workstreams** | W5 as the audit, W7 as the place its corrections are applied during the asset relocation, W10a as the hosting precondition, W13 as the point of exposure — the two consumers are W5's own downstream edges in [03 §5](03-modernization-roadmap.md) |

#### R9 — Cutover re-authentication and anonymous-cart loss

**Both costs are accepted deliberately by [05 §11](05-aspnet-core-migration-approach.md), which owns
the cutover decision. This entry carries them as risks with mitigations and contingencies. It does not
reopen the decision.** They are deltas 8 and 9 of [05 §11.5](05-aspnet-core-migration-approach.md),
with product and operations as their approval owners.

**Cost one — every signed-in user is signed out.** Authentication tickets issued by the legacy stack are
protected by that application's key material and are unreadable by the new key ring. There is no
mitigation that preserves them under a single cutover; the incremental path's remote-authentication
mechanism exists precisely to avoid this and is not in play per assumption **A6**.

**Cost two — anonymous carts do not survive, even if their rows do.** An anonymous cart's identity
lives **only** in in-process session: a generated identifier written to session and never to durable
storage [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:163-179]. When the legacy process stops, the
browser-to-cart link is gone, so the rows become orphaned whether or not they are migrated.
**Signed-in carts are unaffected**, because their key is the login name
[src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:167].

**The rollback symmetry, which is the part most likely to be forgotten.** A reversal that returns to the
legacy application produces **the same two effects in the other direction**: users signed in against the
new application are signed out again, and **anonymous** carts built on it are lost. That covers regimes A and
B of [06 §11.5](06-azure-hosting-recommendations.md), which are now the only two reversals that return to
the legacy application at all — the rung that once did so after an accepted write has been removed. The rollback
plan must state both, or it understates its own cost and a reversal decision gets taken on a false
premise.

**The symmetry stops at "anonymous", and stating that boundary is what keeps this entry from being read as
a wider approval than it is.** A **signed-in** cart built on the new application is **not** among these two
costs and is not lost by a reversal: it is an accepted write under
[06 §11.5](06-azure-hosting-recommendations.md), carried back into the source store by
[06 §11.5.4](06-azure-hosting-recommendations.md)'s verified reverse replay, and where that replay does not
verify the reversal becomes **regime C** rather than a reversal with a larger accepted loss. Deltas 8 and 9
grant the sign-out and the **anonymous** cart, and nothing here extends them —
[R4](#r4--domain-data-migration-rollback) owns that determination and this entry cites it.

**A regime C recovery does not repeat them, and the reason is a decision 06 already took.** Recovery
forward redeploys the new application against the **same** database, and the data-protection key ring is
persisted in that database rather than on an instance, so a revision swap leaves cookies decryptable and
sessions intact. The two costs are cutover costs, not recovery costs — which means the ladder's early
rungs are cheaper for users as well as for the data.

| Field | Value |
| --- | --- |
| **Likelihood** | **Certain** for both. These are designed consequences of an approved decision, not possibilities |
| **Impact** | **Medium.** Re-authentication is an inconvenience on a store with no long-lived session value; an abandoned anonymous cart is lost intent rather than lost data. Impact would be High on an application with checkout-critical anonymous state, which is why the assessment states the property rather than the reassurance |
| **Mitigation** | Schedule the window at low traffic and **drain the legacy application** before switching, so few anonymous carts exist to lose. **Expire the legacy authentication cookie at the new application's first request**, so no browser retries a ticket that cannot be decrypted — an undecryptable ticket presenting as an error is worse than a clean sign-out. Notify users ahead of the window. **Migrate the cart rows anyway**, so no data is destroyed and orphans can be reported and cleaned up afterwards |
| **Contingency** | Orphaned cart rows are reported and cleaned up after the window rather than reconciled against browsers that no longer exist. A user reporting a lost cart is answered by policy, not investigated as a defect. Building a session-bridging mechanism is **explicitly rejected** as disproportionate, per [05 §11](05-aspnet-core-migration-approach.md) |
| **Trigger** | Not applicable as a *detection* trigger — both are certain. The operational trigger to watch is the **window's traffic level at switch time**: cutting over at non-trivial traffic converts a Medium impact into a support event, and is the one variable still under anyone's control |
| **Owner** | **Product and operations**, as the approval owners named by [05 §11.5](05-aspnet-core-migration-approach.md) deltas 8 and 9 |
| **Affected workstreams** | W13, and W15 which follows it |

#### R10 — Scoping by analogy across editions

Carried at [08 §12.3](08-technical-debt-register.md)'s request, which asks that triplication be treated
as a **scoping** risk rather than a code risk.

**The risk is that someone sizes a second edition from this document's figures.** The repository holds
three editions, and two of them are measurably near-identical
[08 §3.1](08-technical-debt-register.md) — which makes it tempting to treat the third as "the same
thing, slightly older". [08 §3.4](08-technical-debt-register.md) draws the bound explicitly, and this
document's assumption **A7** scopes the port to the migration source alone.

**Why an analogy would be wrong in both directions at once:** the oldest edition is **smaller in
volume** — 1,326 against 2,097 non-blank lines, the sizing metric (input 5) — and **larger in
structural difference**, because its cart owns its own context and commits internally, a different
unit-of-work architecture rather than a refactoring of the same one
[01 §7](01-architecture-overview.md). A ratio applied to the line counts would therefore *under*-state
it while appearing conservative.

| Field | Value |
| --- | --- |
| **Likelihood** | **Medium.** Nobody proposes this at the outset; it surfaces when someone asks "and how much for the other two?" and expects a multiplier |
| **Impact** | **Medium.** A wrong estimate rather than a wrong system — but one that would be committed to before anyone opened the code |
| **Mitigation** | Assumption **A7** states the scope. This document sizes **only** the migration source, and input 5 appears solely so the repository total cannot be mistaken for the work envelope. Any request to port another edition is **re-estimated from its own measurement**, per [08 §3.4](08-technical-debt-register.md) |
| **Contingency** | If a second edition enters scope, this document is re-run against it: its own decomposition, its own blocker census, its own bands. Not scaled |
| **Trigger** | Any estimate for an edition other than the migration source that cites a figure from this document. Also: any use of the 5,565-line repository total as a work envelope |
| **Owner** | **Engineering leadership**, with this document as the reference |
| **Affected workstreams** | W1 where the scope is approved; W7 if scope changes |

#### R11 — The effective package source set is not knowable from the repository

[02 §6](02-dependency-inventory.md) establishes that **the repository configures no package source at
all** and corrects the Technical Specification's contrary reading. That finding is cited, not restated.
Its consequence is a supply-chain risk that belongs in a register.

**Restore inherits whatever machine-level and user-level configuration the build host carries.** So the
effective source set is a property of the host rather than of the repository, and **cannot be asserted
from the repository to exclude private or substituted feeds.** This compounds
[R2](#r2--the-migration-sources-build-reproducibility) directly: W2's verification run will resolve
against whatever source its host supplies, so unless that source is declared and recorded, the run
establishes the build outcome for *one host's* package graph rather than for the repository — which is
why W2's exit gate requires the resolved source to be recorded rather than merely used. Compounding it
further, no edition carries a lockfile [08 §8.3](08-technical-debt-register.md) — and the compounding is
**not** that the version set floats. Per [02 §7.1](02-dependency-inventory.md) the three manifests
enumerate every installed package, transitives included, at an exact version each. What is missing is a
**content hash** per entry and any **restore-time enforcement**: a package carrying the expected id and
version but different content restores without complaint, and nothing fails a restore whose resolution
differs from a previously recorded one. Unconstrained source plus unhashed content is what makes the
substitution below undetectable, rather than an unpinned version.

| Field | Value |
| --- | --- |
| **Likelihood** | **Medium** that resolution differs between hosts or over time; **certain** that it is not currently constrained by anything in the repository |
| **Impact** | **High.** A substituted or shadowed package is a supply-chain compromise, and the absence of a lockfile means such a change arrives without a diff for anyone to notice |
| **Mitigation** | The target-state fix is already specified: a **committed package-source configuration that clears inherited sources** and declares its own, plus **per-project lockfiles with restore in locked mode** in CI, so a transitive change **fails the build** instead of arriving silently — [04 §6.2, §6.4](04-dotnet8-migration-strategy.md). W6's exit gate requires both. Interim: W2 declares its source explicitly and records what actually resolved |
| **Contingency** | If an unexpected source or resolution is discovered, restore is pinned to a known-good source and the resolved graph compared against the recorded one. Until W6 lands the lockfiles, that comparison is manual and is the reason the interim mitigation exists |
| **Trigger** | A restore that resolves a different version set on two hosts; any package resolving from a source other than the declared one; a locked-mode restore failing after W6, which is the mechanism **working** rather than a defect |
| **Owner** | **Build and release engineering**, with **security** |
| **Affected workstreams** | W2, W6 as the fix, W11 which enforces locked mode |

#### R12 — No observability exists during the cutover itself

**The risk is a sequencing one, and it is specific: the moment of highest operational uncertainty is
also the moment with the least instrumentation.** [08 §7.1](08-technical-debt-register.md) records that
the application has **no logging, tracing, metrics or health endpoint of any kind** — a Critical
operational finding — and observability in the target is **net-new capability** split across two
workstreams by [03 §5](03-modernization-roadmap.md): **W7** builds the application instrumentation *and*
the two health endpoints, which are application middleware, and **W10** wires the collection path and sink
and configures the platform probes that poll those endpoints. The approach is
[06 §9](06-azure-hosting-recommendations.md)'s and is not restated.

**The consequence for the cutover.** In the two regimes where returning to the legacy application is
available at all — [06 §11.5](06-azure-hosting-recommendations.md)'s A and B — the legacy application is
the reversal target, and it has no instrumentation whatsoever. A blanket `catch` around the order write
discards the exception and records nothing
[src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:58]. So **a reversal into it lands on an
unobservable system** — a failed checkout there leaves no trace, which is exactly the condition under
which a cutover decision has to be made quickly and correctly.

**Past the point of no return the asymmetry reverses, and that is an argument for the ladder rather than
against it.** A regime C recovery goes **forward** against the target, where W7's health endpoints and
W10's wiring and probes are both present, so the recovery is observable in a way a return to the legacy
application never is.
That is one more reason the ladder's early rungs are preferable to a reversal, and one more reason the
observability has to be live **before** the window: it is the instrument for both the decision and the
recovery.

**This risk is why W7's endpoints and W10's observability wiring must both be complete *before* W13 rather
than alongside it.** The graph already enforces the ordering — W7 precedes W10 and both precede W13 in
[03 §4.2.1](03-modernization-roadmap.md) — and this entry states why it matters, so it is not treated as a
polish item deferrable past the window.

| Field | Value |
| --- | --- |
| **Likelihood** | **High.** The absence is verified fact today; the risk is that the window is entered before the target's observability is actually verified working, which is a common compression under schedule pressure |
| **Impact** | **High.** A cutover judged on user reports rather than telemetry is judged late, and in regimes A and B the reversal target provides no evidence at all |
| **Mitigation** | Both halves close before W13 begins, and they are two different gates: **W7** implements the application instrumentation and the two **health endpoints** and demonstrates each in every outcome their owner defines ([03 §5 W7](03-modernization-roadmap.md), exit condition 7), and **W10** completes the collection path and sink and configures the platform probes against those endpoints, verified responding at deployment ([03 §5 W10](03-modernization-roadmap.md), exit conditions 7 and 3). W4's coverage independently includes the readiness check returning healthy with the database reachable and **unhealthy when it is not** — so the instrument is verified rather than assumed. Define, before the window, the specific signals that will be watched and the thresholds that would trigger rollback |
| **Contingency** | If telemetry is unavailable at the window, the window is **postponed** rather than entered blind. If it fails mid-window, the pre-agreed rollback threshold becomes a fixed decision point, and the runbook's verification steps are executed manually against the surfaces W4 already asserts. The reversal that threshold triggers is **whichever one [06 §11.5](06-azure-hosting-recommendations.md)'s probe permits** — losing telemetry does not make a return to the legacy application available after an accepted write, and the probe is four database queries rather than a telemetry read, so it still answers |
| **Trigger** | W13's entry approached with either half open — the health endpoints unimplemented or not demonstrated in every outcome, their platform probes unverified, or the log sink not verified end-to-end; or any window where the rollback signals have not been named in advance |
| **Owner** | **Operations and platform** |
| **Affected workstreams** | **W4**, which supplies the independent verification of the readiness check in both outcomes that the mitigation and the contingency both lean on; **W7 and W10** as the two halves of the mitigation; W13 as the exposure |

#### R13 — One database, one blast radius

[06 §6.5](06-azure-hosting-recommendations.md) records the single-database decision **with its trade**,
so that a reader can reverse it knowingly. This entry carries the trade as a risk; it does not re-argue
the decision.

Both application contexts, the session cache and the data-protection key ring share one database
instance. **The full trade — what is gained and every cost given up, the key ring's enlargement of the
shared read-permission surface included — is stated once, by
[06 §6.5](06-azure-hosting-recommendations.md), and is not reproduced here**: a partial copy of an
owner's trade reads downstream as a second and smaller version of it, which is the failure this entry
would otherwise introduce. What this entry adds is the register's own reading of it: the consequence
that materializes on a bad day is **concurrent** rather than isolated, because the catalog, the
credentials, the sessions and the key material authentication cookies depend on all sit inside one
availability and one blast radius.

**It is a defensible trade for an application of this size with one deployable unit**, which is why 06
took it. It is carried here because it is the kind of decision whose cost is invisible until the day it
is not.

| Field | Value |
| --- | --- |
| **Likelihood** | **Low.** A managed database instance failing or being misconfigured is uncommon |
| **Impact** | **High.** Concurrent loss of catalog, credentials, session and key material. Recovery is a single restore, which is the benefit side of the same trade |
| **Mitigation** | Accept the trade knowingly, as an approval decision rather than a default. Separate **schema ownership** within the instance — two contexts, two migration sets and **a distinct migration history table per context**, per [05 §4.5](05-aspnet-core-migration-approach.md) and [06 §6.5](06-azure-hosting-recommendations.md) — so that the *logical* boundary survives even though the physical one does not. Keep the DDL principal separate from the runtime identity [06 §6.2](06-azure-hosting-recommendations.md), which is what stops a runtime compromise becoming a schema compromise |
| **Contingency** | Restore the instance; both contexts return together. If the granularity later proves insufficient — a compliance requirement on credential data, for instance — separation is a provisioning and connection-string change plus a migration re-point, not a redesign, because the two contexts are coupled only by convention and share no foreign key [01 §6.5](01-architecture-overview.md) |
| **Trigger** | A compliance or data-residency requirement that distinguishes credential data from catalog data; or a scaling need that one instance cannot serve. Either makes the trade wrong prospectively, which is the point at which it should be revisited |
| **Owner** | **Platform and operations**, as an approval decision |
| **Affected workstreams** | W10 as the provisioning, W13 as the point of no easy return |

#### R14 — A reference edition's retired data provider

Carried for completeness because a reader scanning the blocker list will look for it here, and **because
its classification is the one in that list most likely to be misread.**

The oldest edition's catalog provider is a retired embedded database engine
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:55-59] for which **no supported provider
exists on the target framework** [12 §3](12-migration-blockers.md). Its data layer therefore cannot
be ported without re-targeting the provider outright.

**It is not a compile-time blocker, and an earlier form of this entry said it was.**
[12 §2.1](12-migration-blockers.md) files F-12-01 in **group two** — the group compiling does not find —
for a reason that is specific rather than taxonomic: `System.Data.SqlServerCe.4.0` is a `providerName`
**attribute in configuration** [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:55-59] and the
project declares no reference to the provider assembly at all
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:37-41], so nothing in the source binds
to it and nothing about it reaches the compiler. A build cannot break on a string it never resolves.

**Nor is it silent.** It is the **one loud entry in group two**: the provider fails at activation, on the
first data access, throwing rather than degrading. That is why input 12's partition reads
`14 + 9 = 23` with the nine split **8 silent + 1 loud**, and why [R3](#r3--the-absent-regression-baseline)
treats the eight rather than the nine as the group a test suite is the only instrument for. **The practical
consequence of the correction is a change of detector, not of severity**: a compile-time reading would have
this blocker announced by a build nobody has to run against MVC 3, whereas the true detector is *any
execution that touches the catalog* — which is exactly what assumption **A7** guarantees will not happen
while that edition stays a read-only reference.

**Under assumption A7 this affects nothing**, because that edition is not ported — it is retained
read-only as a historical reference and part of the behavioural baseline
[03 §5](03-modernization-roadmap.md). The entry exists so that the *conditional* is recorded rather
than discovered: **if** that edition ever enters scope, this is a re-target and not a port, and its
effort has no basis in this document.

| Field | Value |
| --- | --- |
| **Likelihood** | **Low.** It requires assumption **A7** to be reversed |
| **Impact** | **Low** while A7 holds — there is no effect on any workstream. It becomes High **within that edition's own scope** if it is ever revisited |
| **Mitigation** | Assumption **A7** holds the scope to the migration source. The edition is retained read-only, so nothing depends on its provider continuing to work |
| **Contingency** | If that edition enters scope, its data layer is **re-targeted to a supported provider** as new work, estimated independently — see [R10](#r10--scoping-by-analogy-across-editions), whose failure mode this would otherwise become |
| **Trigger** | Any proposal to port, run or extend that edition; or any attempt to use it as a running system rather than as a source reference |
| **Owner** | **Engineering leadership**, as the owner of scope |
| **Affected workstreams** | **None** while A7 holds |

#### R17 — The interim hosting path's stored credential, and a time-box with no box

**This entry exists because [06 §5.5](06-azure-hosting-recommendations.md) hands it here by name**, with
the security owner named as its owner. That document recommends **Path A** for the interim option —
hosting the un-ported application on the platform's Windows offering — and Path A's defining property is
that **a SQL login with a password exists that did not exist before**, supplied through the site's
configuration behind a **platform secret reference** resolved under the site's managed identity
[Microsoft Learn, *Use Key Vault references as app settings in Azure App Service and Azure Functions*,
<https://learn.microsoft.com/azure/app-service/app-service-key-vault-references> — verified
2026-08-28], on a least-privileged data-only login, as an **explicitly recorded, time-boxed exception**
to the no-stored-credentials principle. 06 owns that recommendation, its costing and its controls; the
source is cited here only because this entry names the mechanism, and none of it is re-decided or
restated.

**The risk is not the credential. It is the box.** 06 §5.5 names the characteristic failure precisely,
and this entry makes it concrete: **choose Path A, never complete the port, and discover later that the
time-boxed exception had no box.** The credential's existence is a designed and approved consequence; its
*persistence past the event that was supposed to end it* is the risk. Three properties make that failure
mode easy to reach rather than exotic — the exception's expiry is defined partly as an **event** (the
port's completion gate) which can slip indefinitely, the interim step is by construction chosen by people
who are not yet ready to approve the port, and a credential that is rotating on schedule looks healthy
whether or not anyone is still pursuing the gate that retires it.

**This entry is conditional on a decision that has not been taken.** The interim option is available and
not selected; [06 §5.1](06-azure-hosting-recommendations.md) presents it as a stepping stone rather than
a destination. If the interim step is never entered, no credential is ever created and this entry never
fires. It is carried at full weight because the decision to enter it is exactly the kind taken quickly,
by people optimizing for not waiting.

**No effort is carried in [section 6.1](#61-the-totals) for the interim path.** It is not one of
[03 §5](03-modernization-roadmap.md)'s sixteen workstreams and this model does not cost it — the same
treatment [section 6.3](#63-what-is-deliberately-not-in-the-total) gives every path the assessment
records but does not assume. If the interim step is selected, this document is re-estimated rather than
adjusted.

| Field | Value |
| --- | --- |
| **Likelihood** | **Medium**, and the two halves must be stated separately. That a credential **exists** is *certain* for the life of the interim step, because it is what Path A is; that the exception **outlives its intended end** is Medium, and it is that second thing this entry tracks. Medium rather than Low because the expiry is anchored partly on an event that can slip, and rather than High because 06 §5.5 requires a calendar backstop and a re-confirmation at every rotation |
| **Impact** | **High.** The login reaches the application's data, which includes the credential store and order data containing personal information — the nine PII fields of the checkout form [09 §3.11](09-security-assessment.md). It is High rather than Critical because the grant is data-plane only — no `db_owner`, no DDL rights, no server-level role — so a leaked credential is a data exposure rather than a schema or platform compromise |
| **Mitigation** | All of 06 §5.5's controls, in force before the step begins and not after: a **least-privileged data-only login**; the secret held in the platform key store and **resolved by reference** so it is never in source, in the repository or in the deployment payload; **automated rotation on an enforced interval with an alert on approaching expiry**; and an approval record signed by the **security owner** naming the exception, the rotation schedule, **both** forms of expiry — the port's completion gate *and* a calendar backstop — and the review checkpoint. An approval naming three of the four has not time-boxed anything |
| **Contingency** | On suspected exposure: rotate immediately, revoke the login, and audit the data-plane access it held. On the backstop date arriving with the port incomplete: the exception is **re-approved explicitly or the interim hosting is withdrawn** — it does not lapse into being permanent by default. If the organization cannot accept a stored credential at all, 06 §5.5's Path B is the documented alternative and the interim step then carries a code approval by necessity, which is a different decision and a different estimate |
| **Trigger** | **A rotation performed without the security owner's recorded re-confirmation** — 06 §5.5 states that this is the signal that the box has gone missing, and it is the trigger to watch because it fires while everything still looks healthy. Secondarily: the backstop date approaching with W7 not exited; the port's completion gate slipping with no re-confirmation recorded; the login's grants widening beyond data-plane access; or the credential appearing in a local configuration file, a log or a deployment payload |
| **Owner** | **Security**, as [06 §13.4](06-azure-hosting-recommendations.md) names it — the security owner approves the exception, sets its expiry and re-confirms it at each rotation. Platform and operations execute the rotation; they do not own the exception |
| **Affected workstreams** | **None while the interim path is not selected.** If it is: the exception's retirement is gated on the port's completion, which makes W7 and W13 the workstreams whose slippage extends this risk, and its removal is a task attached to the same gate |

#### R18 — The browser-executed half of the scripted cart flow: one pinned engine, and an open decision on the other two

**This entry is open, and that is stated first because a previous revision of this document recorded it as
settled.** That revision read [05 §12.11](05-aspnet-core-migration-approach.md)'s pinning of a Chromium
harness as closing the entry, withdrew it from
[section 9.4](#94-the-five-risks-that-are-approval-decisions-not-mitigations) and left that table at four
rows. **That reading was wrong and is withdrawn here.** Pinning the harness settled the *scope* question —
whether any browser executes this flow at all — and settled it affirmatively: 05 §12.11 pins a
browser-automation harness driving **Chromium** for exactly one flow,
[04 §7.7](04-dotnet8-migration-strategy.md) pins the package as one of the six independent test-tooling
pins and adds the browser-install step to the target-test runbook, and input 29 records both. What that
did **not** settle is the question this entry actually carries: **which engines receive a functional
assertion.** Chromium receives one. **Gecko and WebKit receive none**, and no automated case reaches them.

**So the entry returns to [section 9.4](#94-the-five-risks-that-are-approval-decisions-not-mitigations) as
its fifth row, because [03 §5](03-modernization-roadmap.md)'s W1 now carries this residual as a mandatory
approval decision** with three admissible outcomes, enumerated below. Two consequences follow. First, the
**previous statement that no band contains any part of this harness stays withdrawn** — two bands do carry
it, and [section 6.3](#63-what-is-deliberately-not-in-the-total) records that withdrawal where the
exclusion used to sit. The Chromium flow is in scope, is estimated and is **not** reduced by this entry
reopening: **3.5 expected IED as [W7](#w7--the-aspnet-core-port)'s own sub-row and 0.5 expected as the
install step inside [W11](#w11--ci-provider-selection-then-pipeline-authoring)'s manifest half.** Second,
what has **no** band is any *extension* beyond Chromium, which is what outcome two below would create.
**The decision is open; the work already in scope is not.**

**What is and is not covered, stated precisely, because the gap is narrow and easy to overstate.** One
flow in the application is executed by script in the browser rather than by a form submission: the cart
removal, whose page-level code reads a rendered identifier and issues the request that removes the line
[src/MVC5/MvcMusicStore/Views/ShoppingCart/Index.cshtml:12-29]. The suite **does** assert the endpoint
that script calls — its contract, its response shape and its token transport, valid, missing and invalid —
because those are HTTP facts. What it does not assert is that **the script itself** performs that call
correctly in a browser: that it locates the token, sends it in the agreed place and applies the response
to the page. An HTTP-level suite cannot reach that, because it is not the client the behaviour lives in.
**That is now reached, on one engine.** The pinned harness drives **Chromium** over this flow and asserts
the **request header actually sent**, the JSON response actually handled, the **four DOM updates** the
page performs, and **zero console errors and zero policy-violation reports**
[05 §12.11](05-aspnet-core-migration-approach.md). What remains unreached is the same script's behaviour
on the **other three supported browsers** of [06 §10.4](06-azure-hosting-recommendations.md) — **Edge,
Firefox and Safari** — which no case executes. **Two of those three are the material residue**: Edge runs
the same Chromium the harness drives, so the uncovered *engines* are **Gecko** and **WebKit** even though
the uncovered *products* number three. [§7.1](#71-the-manual-visual-review) allocates the appearance
evidence over the same four products on the same reasoning — and that appearance evidence is **not**
functional evidence for these two engines, which is precisely why a decision is required rather than
assumed.

**The three admissible outcomes, as [03 §5](03-modernization-roadmap.md)'s W1 states them.** W1 must
record one of exactly these; recording none leaves an unowned coverage claim, and that is the condition
this entry exists to prevent:

| Outcome | What it commits to | Effect on these bands |
| --- | --- | --- |
| **Accept the residual** | A recorded acceptance naming the flow, the engine that carries a functional assertion and the two that do not, signed by the owner below | **None.** The Chromium harness is already inside W7 and W11 |
| **Extend the automated engine set** | Driving Gecko, WebKit or both. [04 §7.7](04-dotnet8-migration-strategy.md) pins the engines the harness can drive; this document does not name the pin | **Re-estimation.** W7's browser sub-row grows per engine, the target-test job gains an install step per engine, and the diagnostic surface gains per-engine artifacts. **These bands carry none of that** |
| **A named manual functional walk** | A checklist of the flow's steps, a named reviewer and a sign-off artifact, **distinct from [05 §12.5](05-aspnet-core-migration-approach.md)'s appearance review** and governed separately from it | **Re-estimation**, in [W14](#w14--documentation-revision)'s governance work rather than in W7's. [§7.1](#71-the-manual-visual-review)'s band is the appearance review only and absorbs none of it |

**Why the appearance review cannot close this, and why outcome three is nonetheless admissible.** These
are two different artifacts, and conflating them is the error the entry guards against. AAP §0.11.2
sanctions manual review for **rendered appearance** only, so [§7.1](#71-the-manual-visual-review)'s spot
check on the other three browsers is scoped to *rendering* and must never be cited as functional coverage
of this flow. Outcome three is not that review reused: it is a **separate functional walk**, with its own
checklist of the flow's steps, its own reviewer and its own sign-off, which 03's W1 admits as an
*approval* outcome for exactly that reason. It is still **not a regression test** — it does not run again,
and it does not fail when the behaviour breaks later — which is why choosing it is the product owner's
call and not quality engineering's, and why it is recorded as an accepted limitation rather than as
coverage.

**The residue is one flow rather than a class of them, and that narrowing is the current position rather
than an interpretation.** Every other client-side interaction in the application is a **rendered form**,
and the matrix now **fetches the page, parses the returned markup and submits the form it found** on the
surfaces that carry one — the add-to-cart form, the cart page's own form structure and the checkout form
[05 §12.4](05-aspnet-core-migration-approach.md). Those flows are therefore exercised end to end through
the markup the browser would itself act on, which leaves **the cart page's script-issued removal request
alone** as the behaviour no case reaches at the browser level. This entry is scoped to that one flow, and
a reading that treats it as a general absence of browser coverage overstates it.

**The residual is Certain and not a possibility, and the register says so plainly.** Single-engine
coverage is a property of the instrument as pinned, established at design time, so no likelihood estimate
applies to it. What remains undecided is not whether the gap exists — it does, by construction — but
**which of the three outcomes above the owner takes**, which is why the entry is an approval decision
whose likelihood field records a certainty rather than a probability.

**Where the in-scope cost sits, so the open decision can be checked against the bands.** The drivers 05
§12.11 named are the ones paid for: the engine's own **readiness and teardown lifecycle** in the fixture
layer, which is a **separate real-Kestrel host on a loopback port** rather than the in-memory test server
the rest of the suite uses — because the automation cannot reach a `TestServer`
[05 §12.6](05-aspnet-core-migration-approach.md) — and a **second diagnostic surface** under
[05 §12.9](05-aspnet-core-migration-approach.md), because a browser failure produces evidence of a
different kind, and that evidence passes through the same redaction obligation. All of it sits inside W7's
**2 / 3.5 / 6** sub-row, which is target-facing: the flow is a **target-only** case, so the legacy
half in [W4](#w4--build-governance-bootstrap-pre-port-behavioural-baseline-and-test-suite) executes no
browser at all and its band absorbs none of this. The engine is required on the **target-test agent only**
rather than on both agents of [05 §12.10](05-aspnet-core-migration-approach.md)'s split, which is why the
pipeline cost is one install step rather than two. The **pin itself is not named here**: package
identities and versions belong to [04 §7.7](04-dotnet8-migration-strategy.md), and this document cites
that owner rather than restating it.

**One rule is unchanged and bounds all three outcomes: a manual artifact may be *accepted*, and may never
be *counted as an assertion*.** A reviewer walking the flow once on Firefox and signing a checklist
produces a signature, not a regression test — it does not run again, and it does not fail when the
behaviour breaks later. That is why outcome three above is admissible as a **decision** while remaining
inadmissible as **coverage**, and why [§7.1](#71-the-manual-visual-review)'s appearance review — the one
manual case AAP §0.11.2 sanctions — cannot be pointed at this gap at all. The distinction matters for
[R3](#r3--the-absent-regression-baseline) as much as here: manual verification substitutes for rendered
appearance and for nothing else.

| Field | Value |
| --- | --- |
| **Likelihood** | **Certain**, as a designed consequence rather than a probability. The pinned harness covers **one flow on one engine** by design [05 §12.11](05-aspnet-core-migration-approach.md), so this script's behaviour on the other three supported browsers is outside the suite's reach on the day it is written |
| **Impact** | **Low**, reduced from Medium by the Chromium harness being in scope. The endpoint's contract, response shape and token handling are asserted at HTTP level; the script's token location, header transport, response handling and four DOM updates are asserted **in a browser** on Chromium; and the two mechanisms most likely to break — the serializer's property naming and the token's transport — are **engine-independent**, so an engine that diverges here diverges on DOM or fetch behaviour rather than on the port's own changes. The residue is one non-essential convenience on one page on two engines: a removal that fails leaves the cart contents intact and the checkout path unaffected |
| **Mitigation** | Three layers, each named with what it does and does not cover. The HTTP-level cases of [05 §12.4](05-aspnet-core-migration-approach.md) cover the endpoint the script calls, including the valid, missing and invalid token cases — the *server* side, on both fixtures. The **pinned Chromium harness** covers the *client* side on one engine, including the zero-console-error and zero-policy-violation assertions that would surface a script fault rather than a request fault. The **manual appearance review** covers rendering on all four supported browsers, including the 36 spot-check inspections [§7.1](#71-the-manual-visual-review) allocates — **appearance only, and explicitly not behavioural coverage of any engine, so it closes nothing here** |
| **Contingency** | The decision itself is [W1](#w1--approval-of-this-assessment)'s and is listed in [section 9.4](#94-the-five-risks-that-are-approval-decisions-not-mitigations); the contingency is what each outcome costs. **Accepting** the residual costs nothing beyond the recorded acceptance. **Extending** the automated engine set to Gecko, WebKit or both is a **re-estimation**: W7's browser sub-row grows per engine, the target-test job gains an install step per engine, and the diagnostic surface gains per-engine artifacts — none of which these bands carry. A **named manual functional walk** is also a re-estimation, in W14's governance work, and is an accepted limitation rather than coverage. What is **not** available under any outcome is pointing [§7.1](#71-the-manual-visual-review)'s appearance review at this gap, per [R3](#r3--the-absent-regression-baseline)'s rule |
| **Trigger** | Any of: **W1 closing without a recorded decision on this residual**, which is the condition the entry's return to section 9.4 exists to prevent; a defect found in production in the cart-removal script on a non-Chromium browser after the port, which is this residual realized; the harness proving unstable in the pipeline, which converts a coverage gain into a flaky gate and is [R3](#r3--the-absent-regression-baseline)'s failure mode arriving through this entry; or an attempt to close the residue on the other engines by citing the appearance review |
| **Owner** | **Engineering** for delivering the pinned harness inside W7 and W11, and the **product owner** for taking the W1 decision among the three outcomes — the same owner who holds the browser matrix itself under [R7](#r7--the-narrowed-browser-matrix) and [06 §10.4](06-azure-hosting-recommendations.md), because deciding which engines get functional evidence is a decision about which clients are supported to what standard. Quality engineering advises; it does not decide |
| **Affected workstreams** | **W1**, which must record one of the three outcomes at its exit gate. **W7**, which carries the harness as a 3.5-expected-IED sub-row, and **W11's manifest half**, which carries the browser-install step at 0.5 expected. **W4 is unaffected** — the flow is target-only, so the legacy half executes no browser. **W14** if the outcome is a recorded acceptance documented with the matrix, or a manual functional walk's checklist and sign-off |

#### R15 — Personal-data governance is unowned

**The risk is that a migration lands nine personal-data fields into a hosted, internet-facing database
with no retention period, no deletion path and no access audit — and that the migration is what creates
the exposure rather than what inherits it.**

[09 §3.11](09-security-assessment.md) owns the finding. The order record carries `FirstName`
[src/MVC5/MvcMusicStore/Models/Order.cs:23], `LastName` [:28], `Address` [:32], `City` [:36], `State`
[:40], `PostalCode` [:45], `Country` [:49], `Phone` [:54] and `Email` [:61] — nine fields, linked to an
identity by `Username` [src/MVC5/MvcMusicStore/Models/Order.cs:18] — with **no retention period, no
deletion or anonymization path, no encryption at rest and no access audit** in any edition.

**Why the exposure changes rather than transfers — argued without assuming a topology, because the
repository cannot establish one.** An earlier form of this paragraph said the data sits today "on a
developer or on-premises host, **with no external reachability**". That is not a repository fact and this
document should not have asserted it: reachability is a property of a **deployment nobody has observed**,
and the repository contains no publish profile, no pipeline definition, no container manifest and no
infrastructure template from which one could be inferred —

```bash
git ls-files | grep -iE '\.pubxml$|Dockerfile|azure-pipelines|^\.github/|\.tf$|\.bicep$'   # -> no output
```

— only a developer launch URL on `localhost` with an empty SSL port
[src/MVC5/MvcMusicStore/MvcMusicStore.csproj:285], [:19], which describes one engineer's machine and not
any deployment. **Today's reachability is therefore recorded as unknown**, and the case for W16 does not
rest on it. It rests on two things the repository *does* establish and on one asymmetry between them:

- **The controls are absent, certainly and everywhere.** No retention period, no deletion or
  anonymization path, no encryption at rest and no access audit, in any edition —
  [09 §3.11](09-security-assessment.md)'s finding, and an absence rather than an inference.
- **What is knowable about the store engine is narrow and does not extend to the data.** The catalog and
  credential stores are file-attached to a local instance under an integrated-security connection
  [src/MVC5/MvcMusicStore/Web.config:12-13], so the **engine** is not a network-addressable service. That
  says nothing about who can reach the **data**, because every read and write reaches it *through the
  application*, and how reachable the application is depends entirely on where it was deployed.
- **The port adds persistence surfaces that outlive the rows, and those are specified rather than
  guessed.** A managed database reached over the network under a platform identity, scheduled backups,
  restore points, and a non-production copy for rehearsal — [06 §6](06-azure-hosting-recommendations.md)
  and [06 §11](06-azure-hosting-recommendations.md) specify each. A deletion applied to a row does not
  reach a restore point, which is why W16's conditions have to exist **before** the load rather than after.

**So the comparison is between an ungoverned set whose exposure is unknown and an ungoverned set whose
exposure is specified — and the second is the one this plan is accountable for.** The absence of
governance is inherited; the surfaces it will be absent across are created here, and they are enumerable.
That is a sufficient case for gating W16, and it needs no claim about a network nobody has looked at.

**Two properties make this Low-confidence to estimate and High-impact to get wrong.** It requires three
constituencies to convene — the data owner, security and legal — and its central artefacts are decisions
rather than code: what the retention period *is*, whether a real copy may be restored into a
non-production environment at all, and how long a deleted record remains recoverable from a backup. None
of those is derivable from the repository, which is why [03 §5](03-modernization-roadmap.md) makes W16 an
approval-gated workstream rather than a checklist item inside the data migration.

| Field | Value |
| --- | --- |
| **Likelihood** | **High** that the controls are still absent at the point of the production load unless the workstream is gated, because nothing in the source system, the port or the hosting decision produces them as a by-product. It is not Certain, because W16 exists precisely to prevent it |
| **Impact** | **High.** An ungoverned personal-data set in a hosted database is a compliance and disclosure exposure independent of any functional defect, and it is the class of problem that is cheapest to fix before the data lands and most expensive after — a retention rule applied retroactively has to reach into backups and restore points as well as into rows |
| **Mitigation** | [03 §5 W16](03-modernization-roadmap.md) gates it in **three** places, staged so that each gate is the strongest one available at that point in the sequence. Its conditions 1–3 — retention per data class, non-production copy handling, legal hold — are an **entry condition of W3 and W4**, which is where the plan *first* touches real personal data: W3 attaches the committed credential and catalog databases and W4 restores both store pairs before every run, and stage 1 depends on W1 alone precisely so that it can sit in front of them. All six conditions are then an **entry condition of W8 and W9**, so a rehearsal copy is held to the same standard as production once the mechanism exists; and all six are an **entry condition of W13**, so no production personal data loads before the deletion mechanism, the backup-propagation window and the access auditing are live. The deletion operation is demonstrated against **synthetic** data first, so the mechanism is proven before real records pass through it; and because "live" auditing means observed arriving, stage 2 consumes W10's verified sink rather than a sink that is merely configured |
| **Contingency** | If the controls are not ready when the cutover window is otherwise reachable, **the window moves.** There is no partial position: [03 §5 W16](03-modernization-roadmap.md) makes all six conditions an entry condition of W13 and [09 §6.8.2](09-security-assessment.md) states the same gate, so a load with five of six controls live is a gate failure rather than a reduced-scope option. The reason the gate is binary rather than graduated is that the missing control is always the expensive one to add later — a retention rule or a deletion path applied after the load has to reach into backups and restore points, not only into rows. The engineering response to an unready control is therefore to shorten the path to the decision, not to proceed without it: the deletion mechanism is demonstrated against synthetic data (W16 condition 4) and needs no real record to be proven, so it can be finished while the approvals it does not depend on are still outstanding |
| **Trigger** | **W3 proposing to attach, or W4 to restore, a committed store with conditions 1–3 unapproved** — the earliest and most easily missed form of this risk, because it looks like reading a file rather than processing personal data; W8 or W9 proposing to restore a real copy with any of the six conditions open; or W13's entry being approached with any of the six open. Secondarily: a retention period recorded as "to be determined", which is the paperwork form of this risk |
| **Owner** | **The data owner**, with security and legal as co-approvers. Explicitly not the data-engineering team: the periods and the hold process are not engineering trades |
| **Affected workstreams** | **W1**, on which W16 stage 1 depends and where the three constituencies convene; W16 as the mitigation, in two stages; **W3 and W4 as the first real-data contact**, gated on stage 1; W8 and W9 through their use of real copies, gated on all six; W10 as the supplier of the verified sink stage 2 depends on; W13 as the point of exposure |

#### R16 — No security-relevant action is recorded anywhere

**The risk is that the target inherits the source's total absence of an audit trail, and that the absence
is only noticed when an incident requires one.**

[09 §6.8](09-security-assessment.md) owns the finding: **no security event of any class is recorded
anywhere in any edition** — not a sign-in, not a failed sign-in, not a lockout, not a role grant, not an
authorization denial, not an administration write, not an order placement. It is a sibling of
[08 §7.1](08-technical-debt-register.md)'s Critical observability finding but not the same one: general
telemetry tells an operator that the system is unwell, while a security-event record tells an
investigator who did what, to what, and whether it succeeded.

**Why it belongs in this register rather than only in 09's.** The catalog is **net-new capability with no
legacy volume to scale from**, so it has to be sized deliberately or it will be omitted — and it is sized
per producer, because its sixteen classes have three of them: the **thirteen** the ported application
emits are a named sub-row of W7 in [section 5.2](#52-basis-of-estimate-per-workstream), the Identity
migration's `AUTHZ-3001` sits inside W8's band and the command's three inside W12's, and **proving
collection is split across two workstreams rather than sitting whole in W10's exit gate**: W10 proves the
*path* with **one** canary class, and W12 demonstrates the remaining **twelve** against a deployed
environment. And it is the kind of requirement that is routinely satisfied on paper: emitting an event is
application code, but a record that reaches no sink, or reaches one whose retention is shorter than the
interval at which incidents are discovered, is not an audit trail. That is why
[03 §5 W10](03-modernization-roadmap.md) requires a record to be observed **arriving** in the sink at its
stated retention rather than the sink merely to be configured — and why the class it requires is one whose
producer needs no data at all, since W10 provisions empty schema and could not produce the other twelve if
it were asked to.

| Field | Value |
| --- | --- |
| **Likelihood** | **Certain** as a statement about the source — the absence is verified fact, repository-wide. As a *forward* risk, the exposure is that the catalog is partially implemented or implemented without verified collection, which is the common outcome when an audit requirement has no exit gate |
| **Impact** | **High.** Without it, an account compromise, a privilege escalation or an unauthorized administration write is undetectable and unreconstructable after the fact. Impact is High rather than Critical because it does not prevent the application from working correctly — it prevents anyone from knowing when it did not |
| **Mitigation** | Three-part, and each part has an owner: [09 §6.8.1](09-security-assessment.md) specifies the catalog — every event class with its identifier, actor, target, outcome, severity and permitted fields — and **§6.8.1.1 the producer map**, which is what makes the gates below scoped rather than unpassable; [03 §5 W7](03-modernization-roadmap.md) makes emitting **the thirteen application-produced classes** an **exit condition of the port**, together with the redaction tests that prove no forbidden field appears and the three named seams without which three of the thirteen have no emission site at all, while the other **three** are gated at W8 and W12 where their producers exist; and **verified arrival in the sink at the stated retention is an exit condition of two workstreams rather than one** — [03 §5 W10](03-modernization-roadmap.md) condition 7 proves the collection path end to end with **a single canary class, `AUTH-1002` at `AccountNotFound`**, chosen because its producer needs no catalog row, no account and no administrator and is therefore drivable on the empty schema that workstream provisions, and [03 §5 W12](03-modernization-roadmap.md) condition 7 demonstrates **the remaining twelve deployed**, from the fixture population and the executable that can drive them. Thirteen plus three is sixteen; the split is [09 §6.8.1.1](09-security-assessment.md)'s and the per-producer destinations are [06 §9.5](06-azure-hosting-recommendations.md)'s producer matrix, neither of which this document re-derives. The log-privacy policy of [06 §9.2](06-azure-hosting-recommendations.md) is what keeps the audit trail from becoming a second copy of the personal data it is auditing — the actor is a pseudonymous identifier, not a login name |
| **Contingency** | If a producer's assigned classes are incomplete at that producer's gate, the missing classes are completed before the gate closes rather than deferred to a hardening pass — an audit trail with gaps is trusted as though it had none, which is worse than a known absence. If collection cannot be verified at W10, the sink is treated as unproven and the gate fails. **The failure mode this contingency does not cover, and the producer map is what removes it:** a gate that demanded all sixteen classes from the port would be failed by a correct implementation, and the likely response is a placeholder record emitted from the application for an event the application never observes |
| **Trigger** | W7's exit approached with any of its **thirteen** classes unemitted, with any of the three seams absent, with any outcome value unexercised, or with a redaction test absent; W8's or W12's exit approached with the classes the producer map assigns them unemitted, at the wrong record cardinality, or with their destination or its **export into the audit store** unverified; W10's exit approached with the sink configured but never verified against the canary class; **W12's exit approached with the deployed twelve-class census incomplete, or with the fixture rows it drove left behind unremoved**; or a retention period for security-event records shorter than the period over which incidents are actually discovered |
| **Owner** | **Security**, with platform engineering for the collection path |
| **Affected workstreams** | W7 as the emission of the **thirteen** application-produced classes; **W10 as the collection path, proved by one canary class only** — not by the thirteen, and not by the personal-data access-audit records, whose emission and proof of arrival are **W16 stage 2's alone** per [03 §5 W16](03-modernization-roadmap.md) even though they share a destination; **W16 stage 2** for those records; **W8 as the sole producer of the migration's `AUTHZ-3001` records**; and **W12 as the producer of `PROV-6001`, `AUTHZ-3001` and `AUTHZ-3002`** at the cardinality [06 §9.5](06-azure-hosting-recommendations.md) row 5 fixes: four `PROV-6001` on a provisioning run and, on a revoke, one or two according to the branch — one where the named account does not resolve, two otherwise ([09 §6.8.1](09-security-assessment.md)) — plus one `AUTHZ-3001` where a membership is actually added and one `AUTHZ-3002` where one is removed. Those three land in a different destination — the captured pipeline-job artifact, exported into that section's audit store of record rather than into the application's sink |

### 9.4 The five risks that are approval decisions, not mitigations

Most entries above are managed by engineering action. **Five are not**: their correct resolution is a
decision by a named owner, and no amount of engineering diligence substitutes for it.
[03 §5](03-modernization-roadmap.md)'s W1 exit gate requires a recorded decision on each.

| Risk | The decision, and who takes it |
| --- | --- |
| [R1](#r1--the-target-framework-support-window) | Confirm the target framework knowing its support end date, or retarget before W6 — **engineering leadership** |
| [R7](#r7--the-narrowed-browser-matrix) | Accept the loss of a class of client, or reverse the dependency decision behind it — **the product owner** |
| [R9](#r9--cutover-re-authentication-and-anonymous-cart-loss) | Accept re-authentication and anonymous-cart loss as the cost of a single cutover — **product and operations** |
| [R13](#r13--one-database-one-blast-radius) | Accept a shared blast radius for operational simplicity — **platform and operations** |
| [R16](#r18--the-browser-executed-half-of-the-scripted-cart-flow-one-pinned-engine-and-an-open-decision-on-the-other-two) | Choose one of three outcomes for the **Gecko and WebKit functional residual**: accept it, extend the automated engine set, or add a named manual functional walk — **the product owner** |
| [R15](#r15--personal-data-governance-is-unowned) | Set the retention periods, the non-production copy rules and the legal-hold process, and accept them as the gate on the production data load — **the data owner**, with security and legal |

**Presenting any of these five as a mitigated engineering risk would be the register's most misleading
possible error**, because it would imply the work can proceed correctly without a decision that only
somebody else can take.

**[R16](#r18--the-browser-executed-half-of-the-scripted-cart-flow-one-pinned-engine-and-an-open-decision-on-the-other-two)
is the fifth row, and a previous revision of this document wrongly removed it.** That revision reasoned
that [05 §12.11](05-aspnet-core-migration-approach.md) pinning a Chromium harness left no decision to
take. It settled the **scope** question — the harness is in scope, and its effort is inside
[W7](#w7--the-aspnet-core-port)'s and [W11](#w11--ci-provider-selection-then-pipeline-authoring)'s bands —
but not the **coverage** question, which is that functional automation reaches **one engine** and **Gecko
and WebKit have no automated functional assertion at all**. [03 §5](03-modernization-roadmap.md)'s W1
carries that as a mandatory decision with three admissible outcomes: accept the residual; extend the
automated engine set, which [04 §7.7](04-dotnet8-migration-strategy.md) pins and this document would
re-estimate; or add a **named manual functional walk** with its own checklist, reviewer and sign-off,
distinct from [05 §12.5](05-aspnet-core-migration-approach.md)'s appearance review. **The row is restored
because closing W1 without recording one of those three would leave a coverage claim nobody made** —
and because an appearance review cannot stand in for a functional assertion, which is the rule
[R3](#r3--the-absent-regression-baseline) states and AAP §0.11.2 sets.

**[R15](#r17--the-interim-hosting-paths-stored-credential-and-a-time-box-with-no-box) is deliberately not
a fifth row here**, and the distinction is worth stating because it also turns on someone else's
decision. These five are decisions W1 must record for the **primary** migration to proceed correctly at
all. R15's decision gates the **optional interim hosting path**, which is not one of
[03 §5](03-modernization-roadmap.md)'s workstreams and which the primary path never requires; its
approval is a **security-owner gate on entering that path**, taken at that point rather than at W1, and
[06 §5.5](06-azure-hosting-recommendations.md) specifies exactly what that approval must name. Listing it
above would imply the port cannot begin without a decision about a path it does not use.

### 9.5 Why the enlarged scope added no seventeenth entry

**The register still carries eighteen entries after the re-derivation of
[section 5.1](#51-summary-table), and that is a classification decision rather than an omission.** The
scope [05](05-aspnet-core-migration-approach.md), [04](04-dotnet8-migration-strategy.md) and
[06](06-azure-hosting-recommendations.md) added is almost entirely **control** — mechanisms that make an
existing risk less likely or its failure visible — and a control belongs in the mitigation of the entry it
serves, not in a row of its own. Adding a row per control would inflate the register while leaving the
uncertainties it exists to track unchanged. Each item is therefore assigned to the entry that already
carries its uncertainty, and the assignment is printed so a reader can disagree with it specifically:

| New scope | Entry that carries it | Why it is not its own entry |
| --- | --- | --- |
| The fixture's private legacy deployment lifecycle, now including startup-quiescence polling and a captured process id, and the published fixture dataset with its per-entity counts, fingerprint and seven post-load invariants (input 25) | [R3](#r3--the-absent-regression-baseline), with [R2](#r2--the-migration-sources-build-reproducibility) | Both are **determinism controls** on the baseline R3 exists to obtain. The lifecycle additionally requires a host that **runs** the legacy application, which R2 already carries and R3's mitigation already depends on |
| The baseline record's eleven gating `baselineSource` values, its seven separately recorded and never-compared `targetRun` facts, the **committed** `baseline-reference.json` that makes accepting a baseline reviewable, the `coverage` completeness object, the digest-sidecar artifact transfer, the fail-closed compatibility gate and the diagnostic-versus-record publication policy (input 26) | [R3](#r3--the-absent-regression-baseline), with [R2](#r2--the-migration-sources-build-reproducibility) | A gate that **refuses** a mismatched record is the determinism control working, not a new hazard. Its consequence — a re-capture on the platform that alone can produce one — is R2's dependency, stated there |
| The three SQL identities, the durable ownership registry with its JSON sidecar and its identifier re-resolution before every `DROP`, the standalone orphan-sweep class with its always-run cleanup job, the portable fault injection and the three-layer timeout model (input 24) | [R3](#r3--the-absent-regression-baseline) | These are **safety and isolation controls** on the fixtures. The destructive risk to *real* data is [R4](#r4--domain-data-migration-rollback) and [R13](#r13--one-database-one-blast-radius)'s and is unchanged by them |
| The second diagnostic schema, the sanitized exception projection, the closed set of twenty-six `operation` codes, the redactor's own corpus, and the keyed one-way pseudonyms whose corrected scheme invokes the pinned normalizer, with their key custody and destruction (input 22) | [R3](#r3--the-absent-regression-baseline) | R3's mitigation already requires failures to be **diagnosable without disclosing what they saw**; these specify how, and [06 §9.5](06-azure-hosting-recommendations.md) owns the platform configuration behind them |
| The thirteen-check deployed verification gate, its two-instance affinity-off topology and its attempt budget's literal evidence bounds (input 28) | [R12](#r12--no-observability-exists-during-the-cutover-itself), with [R13](#r13--one-database-one-blast-radius) | The gate is **the mitigation** — five of its checks exist precisely to detect the cross-instance and restart continuity failures a shared key ring prevents. Its failure mode is a **blocked release**, which is the gate succeeding at its purpose |
| The secretless cache-table credential path, **whose SQL-authentication fallback [06 §6.4](06-azure-hosting-recommendations.md) has since removed**, and the artifact store's encryption, key custody, dedicated destruction scope and lifecycle-enforced deletion | [R15](#r17--the-interim-hosting-paths-stored-credential-and-a-time-box-with-no-box), with [R3](#r3--the-absent-regression-baseline) | Both **remove** a stored credential or a retained plaintext rather than introducing one. R15 is the entry that tracks a stored credential's existence, and it is unaffected because these paths carry none |
| The second test project, the **six** independent tooling pins, the abstract-plus-sealed contract topology, the runtime-neutral store observer, and the five projects and five lockfiles (input 27) | [R11](#r11--the-effective-package-source-set-is-not-knowable-from-the-repository) | One further pin restored from an undeclared source is the same finding at a larger count; R11 owns it, and [04 §7](04-dotnet8-migration-strategy.md) owns the pin |
| The security-header middleware, the error action, and the ledger's further entries — **22** in total, the three added this round being the anti-forgery adoption, the verb-mismatched `405` responses and the cart-migration failure notice | [R6](#r6--security-hardening-versus-compatibility), with [section 7.2](#72-the-approved-delta-sign-offs) | Each is a **hardening or a behaviour change with a named approval owner**, and R6's subject is exactly that an item of that kind must be **explicitly labelled** rather than adopted by default. The five new ledger entries are counted in 7.2's sign-off basis, which is where an unapproved delta becomes visible |

**One item was genuinely considered for a seventeenth entry and rejected on the evidence.** The
fail-closed compatibility gate creates a failure mode the previous scope had no equivalent of: the target
half of the suite **cannot run at all** if the legacy binary, the fixture fingerprint, the configuration
or the pinned locale and collation values move, which makes every one of those a re-capture trigger on the
one platform that can produce a baseline. That is a real coupling, and it is a **compound of R2 and R3
rather than a third thing**: R2 already carries the availability of the platform that runs the legacy
application, and R3 already carries the baseline's determinism and states that its capture half can only
happen where that application runs. A separate entry would restate both and own neither, which
[section 10.4](#104-cross-reference-index) is the wrong side of. Both entries were **updated** instead,
and R3's mitigation now names the gate explicitly.

**[R16](#r18--the-browser-executed-half-of-the-scripted-cart-flow-one-pinned-engine-and-an-open-decision-on-the-other-two)
narrowed twice and closed neither time — and its return to
[section 9.4](#94-the-five-risks-that-are-approval-decisions-not-mitigations) adds no row here.** The
suite already fetched, parsed and submitted the rendered forms, which reduced the entry's subject to the
**one** script-issued flow; [05 §12.11](05-aspnet-core-migration-approach.md) then **pinned a Chromium
harness for that flow**, which put the effort inside [W7](#w7--the-aspnet-core-port)'s and
[W11](#w11--ci-provider-selection-then-pipeline-authoring)'s bands and reduced the exposure to **Gecko and
WebKit**. Neither narrowing produced a functional assertion on those two engines, so the entry remains
**open** and remains an approval decision — restored as section 9.4's fifth row, with the three admissible
outcomes [03 §5](03-modernization-roadmap.md)'s W1 names. **That is a change of scope inside an existing
entry rather than a new uncertainty**, which is why the register is still sixteen: a control arriving does
not add a row, and neither does a decision being restored to the owner who must take it.

---

## 10. Roll-up

### 10.1 The estimate in nine statements

1. **The total is 147.5 / 268.5 / 474 ideal engineer-days** across [03](03-modernization-roadmap.md)'s
   sixteen workstreams (142.5 / 259.5 / 458.5) plus the two non-code tasks that sit outside them, the
   manual visual review (2.5 / 4.5 / 7.5) and the manual accessibility review (2.5 / 4.5 / 8). The approved-delta sign-offs are **inside W1's band** and are
   counted once, not added again. An IED is work content, not elapsed time. **Four rows have now been
   re-derived a fourth time** — W4, W7, W11 and W12 — because a further round of closed
   findings moved
   the published suite to **75 rows, 242 cases and 386 fixture executions**; made the contract topology
   **declarable** rather than described, with public types and a const-named collection definition per
   surface group per assembly; added an **`IStoreSetup`** write API off the injected context and five
   further observer projections; committed **twelve fixture inputs** including nine schema-divergence
   override files; gave the load, reconcile and dry-run commands a mandatory **`--scope`**; added a
   **migration load journal** written inside each group's own transaction; added the deployed gate's
   **telemetry join protocol** and its executor-and-stage mapping; and made **row 75 W12's acceptance**
   with an operator-tool audit provider behind it. That fourth
   re-derivation is
   +8.5 / +18.5 / +30; [section 5.1](#51-summary-table) prints the movement
   and [section 5.2](#52-basis-of-estimate-per-workstream) shows each derivation.
2. **The port of existing code is about 21 percent of that.** The other 79 percent is net-new
   capability, data work gated on an unextracted schema, and governance — none of it predicted by any
   quantity in the existing codebase. **The re-derivation moved the ratio by moving almost only the
   net-new side**: the porting figure rose from 52.5 to 53.5 expected IED, for the shared
   application-services seam, while the net-new figure rose from 144.5 to 162.
3. **The two largest rows are W7 (109.5 expected) and W4 (66 expected)**, and the second of them replaces
   nothing: there are zero tests today. **49.5 of W7's 109.5 is also the suite** — its target-facing
   machinery, its browser-driven flow and its cases —
   so the verification apparatus is 115.5 of the 258 expected IED, and its own addition is
   66 + 22.5 + 3.5 + 23.5 = **115.5**, comfortably more than two fifths of the model.
4. **Four low-confidence rows carry 86 expected IED**, and three of the four are waiting on one cheap
   workstream: W3, at 4 expected IED, is the highest-leverage item in the model. Their share **fell
   fractionally, from about 34 percent to about 33 percent**, because W4 absorbed only 5.5 of the 18.5
   expected IED this re-derivation added while the three Medium rows took the other 13. A fifth row, W2,
   is
   medium-confidence for a different reason — its tasks are enumerated but its outcome is unknown,
   because the migration source has never been built on the prescribed toolchain.
5. **The critical path is 211.5 of the 258 expected IED.** About 18 percent can be absorbed by parallel
   capacity; the rest cannot, however many engineers are added. Two orderings put it there rather than
   estimator caution: the domain load precedes the Identity data migration, and the visual review closes
   the port's exit gate. **The off-path share rose slightly again, from about 17 percent to about
   18 percent** — and this time no node changed sides: **+13 of the +18.5 landed on the path** and +5.5
   beside it, which is a larger proportional gain for the off-path rows than for the chain. W3 is now a
   third predecessor of gate 4b, so it is a mandatory precondition of the heaviest gate in the plan even
   though it never holds the binding slot at any band.
6. **Every figure states its counting method.** The authentication rewrite is estimated against
   **382 non-blank lines** — the sizing metric — and the physical line count for that file appears
   nowhere in this document. The suite is estimated against the **case**, the unit
   [05 §12.4](05-aspnet-core-migration-approach.md) publishes, with fixture executions counted separately
   because a case authored once may execute twice.
7. **The register carries eighteen entries**, each with all seven required fields — and it still carries
   sixteen after this re-derivation, because the scope that was added is **control rather than
   uncertainty**;
   [section 9.5](#95-why-the-enlarged-scope-added-no-seventeenth-entry) assigns each new item to the entry
   that carries it and states the one candidate for a seventeenth that was considered and rejected. **Five**
   of them are
   **approval decisions rather than engineering risks**, and W1 cannot correctly exit without a recorded
   decision on each. That count returns from four because
   [R16](#r18--the-browser-executed-half-of-the-scripted-cart-flow-one-pinned-engine-and-an-open-decision-on-the-other-two)
   is **not settled**: pinning the browser harness put one flow on one engine inside W7's and W11's bands,
   but **Gecko and WebKit have no automated functional assertion at all**, and
   [03 §5](03-modernization-roadmap.md)'s W1 carries that residual as a mandatory decision among three
   admissible outcomes — accept it, extend the automated engine set, or add a named manual functional walk
   distinct from the appearance review. An appearance review cannot substitute for a functional assertion.
8. **This document contains no schedule.** Sequence plus effort lets a reader build one from their own
   capacity assumptions; a schedule computed here would embed assumptions that are not this
   assessment's to make.

### 10.2 What this document changes in the repository: nothing

It **adds** one file under `docs/modernization/` and **modifies no pre-existing file**. Every figure above
describes work that would follow an approval that has not been given. Per AAP §0.11.5, the check is the
pair [§2](#2-the-no-modification-constraint) states:

```bash
git diff --name-status ea2552d..28e3652   # durable: 13 lines, all "A", all under docs/modernization/
git status --porcelain                    # current-checkout only: empty in a clean checkout of 28e3652
```

Thirteen additions, **no `M` line and no `D` line** — so no pre-existing file was modified or deleted and
nothing was added outside `docs/modernization/`, which is the claim, rather than the false one that nothing
was added at all. The first command is the durable half **because its range is pinned at both ends**;
`ea2552d..HEAD` would not be, since `HEAD` moves. The second is an authoring-time check, empty at commit
and silent about any later checkout.

**The heading is a shorthand, and the sentence under it is the claim.** "Changes nothing" means *modifies
nothing that existed*; the thirteen files this work adds are its output and are not a violation of the
constraint it was held to.

### 10.3 Acceptance criteria for this deliverable

Self-check against AAP §0.11.2 row 07 and the deliverable's own authoring contract.

| Criterion | Where satisfied |
| --- | --- |
| An effort model **per workstream**, in **stated units** | [§4.2](#42-the-unit-defined) states the unit; [§5.1](#51-summary-table) is one row per workstream, using 03's names |
| **Low / expected / high bands** on every row; no single-point estimate | [§5.1](#51-summary-table) — all eighteen rows, and the W7, W10, W11 and W16 sub-decompositions |
| **Assumptions explicit** | [§4.3](#43-assumptions-stated-rather-than-implied) — A1–A10, each with the consequence of its being false |
| **Confidence with its reason** | [§4.4](#44-confidence-and-its-reason) — overall plus per-workstream, with the reason stated rather than asserted |
| Every effort figure **traces to a count** in this document, 01, 02 or 08 | [§4.1](#41-the-estimation-basis-every-input-with-its-method) — **31** numbered inputs, each with its method and its source; **every one of the eighteen rows of [§5.1](#51-summary-table) names at least one that sizes its own work**, and every band in §5.2 cites the inputs it uses. **Input 31 is the most recent of them and it closes the last row that was sized from the wrong population**: W5's repository-wide path-literal census, which replaces the migration-source inputs 7, 8 and 11 that row previously named. Inputs 24–30 exist for the same reason: **seven inputs closing gaps on eight rows** — six that named no count at all, **W12** which named only input 23 (sizing its *added tests* rather than the tool, now closed by **input 30**), and **W11** which named a constraint rather than a size (closed by input 26, the one input serving two rows, which is why seven cover eight). §4.1's fifth usage note carries the row-by-row census, the four-kind partition `2 + 2 + 2 + 1`, and the one of the seven that moved a band |
| Every figure **names its counting method** | [§3](#3-the-two-counting-methods) states the rule; every figure carries "sizing", "duplication", "file count", "site count", "pin count", "row count", "entry count" or "test count" |
| The **test workload is partitioned** between the parity suite and the tests that have no baseline | [§4.1](#41-the-estimation-basis-every-input-with-its-method)'s inclusion rule — input 14 is [05 §12.4](05-aspnet-core-migration-approach.md)'s **75** parity rows, **re-read from that table by the command input 14 carries rather than held as a constant**, and input 23 the **seventeen** tests [04](04-dotnet8-migration-strategy.md) and [06](06-azure-hosting-recommendations.md) require, for `75 + 17 = 92` executable scenarios; the second set is estimated in W12, W7 and W10, never folded into W4, and [03 §4.3](03-modernization-roadmap.md) gates it at the same three workstreams |
| **382 non-blank used for the authentication rewrite**; physical counts not used for sizing | [§3.2](#32-the-one-substitution-that-would-corrupt-this-document) and [W7](#w7--the-aspnet-core-port). The physical figure for that file appears nowhere |
| Asset counts **state their inclusion rule** | Input 7 and [A.3](#a3-helper-view-and-site-counts) — the migration source's **four asset groups**, distinguished from the all-edition and browser-served counts |
| Estimated against **03's workstreams**; no alternative decomposition | [§5](#5-effort-by-workstream) opening paragraph; W1–W16 verbatim, including W16 |
| **Net-new work sized honestly**, and **partitioned by activity rather than by workstream** | [§6.2](#62-the-finding-that-matters-most-in-this-document) quantifies it at **76.6%** of expected effort — `82.5 + 34 + 37 = 153.5` of 200.5, against the **47** that is porting. The three rows whose split crosses a category boundary — W7, W11 and W16 — are split along the sub-bands they publish, and both partition identities are checked there column by column |
| **Every arithmetic identity in the model holds**, column by column, including the two derived claims | Checked by extracting the figures and summing them, not by reading them. **Row sum:** [§5.1](#51-summary-table)'s eighteen rows sum to `107 / 200.5 / 357`, equal to its own Total row and to [§6.1](#61-the-totals)'s. **Walk:** [§6.1.1](#611-the-walk-from-the-previously-published-total)'s nine deltas sum to `+17 / +30.5 / +51`, each row's From-to-To difference equals its stated delta, every To equals §5.1's band, and the earlier of the two intermediate totals that section records — `105.5 / 198.5 / 354` before input 14's sixth reading — plus W4's current `+1.5 / +2 / +3` reaches the same destination from the other end. **Activity partition:** [§6.2](#62-the-finding-that-matters-most-in-this-document)'s four categories sum to the same three totals, its four shares sum to `100.0%`, and each of the three splits sums to its row's published band. **Concurrency:** [§8.2](#82-concurrency-permitted-by-the-graph)'s twelve set figures sum to `176.5`, plus the parenthesised `+4`, giving `180.5`, and `180.5 + 20 = 200.5`. **Path:** [§8.3](#83-the-critical-path-and-what-to-do-first-if-the-goal-is-to-narrow-the-estimate)'s base plus its two gate rows equals the gate-inclusive figure in all three columns, on-path plus off-path equals the total in all three, and the off-path row equals the sum of its eight itemized parts. **The two derived claims:** the high-to-low ratio `357 / 107 = 3.3364` → *about 3.34*, and the off-path share `32.5 / 200.5 = 16.21%` |
| Sequence **dependency-ordered**, parallelism noted, **no calendar** | [§8](#8-sequencing) — concurrency sets and critical path, explicitly a property of the graph, with 03's three staged workstreams (W10, W11, W16) split across sets |
| **Every gate on the critical path is one that can actually be met** | [§8.3](#83-the-critical-path-and-what-to-do-first-if-the-goal-is-to-narrow-the-estimate) — the chain is derived from 03's corrected graph, and the visual sign-off is counted **inside** W7 rather than after it |
| **R1 first**, framed as an **approval decision** | [R1](#r1--the-target-framework-support-window), and [§9.4](#94-the-five-risks-that-are-approval-decisions-not-mitigations) |
| All nine named risks present | R1–R9 in [§9.3](#93-the-entries), plus R10–R16 where the evidence warranted |
| **All seven fields on every entry** | [§9.3](#93-the-entries) — a field table per entry, sixteen of sixteen complete |
| The **visual review** and **delta sign-offs** carried as tasks with effort, **at the gates they belong to** | [§7.1](#71-the-manual-visual-review) — review and sign-off inside W7's exit gate — and [§7.2](#72-the-approved-delta-sign-offs), inside W1's |
| **Personal-data governance and the security-event catalog sized and risked** | W16 in [§5.2](#52-basis-of-estimate-per-workstream); the security-event sub-row in [W7](#w7--the-aspnet-core-port); [R15](#r15--personal-data-governance-is-unowned) and [R16](#r16--no-security-relevant-action-is-recorded-anywhere) |
| **Cross-references only**, no restatement | [§1.4](#14-what-this-document-does-not-own--the-routing-table) is the routing table; §10.4 is the index |
| Every effort figure **traces to a count** in this document, 01, 02, 04, 05, 06 or 08 | [§4.1](#41-the-estimation-basis-every-input-with-its-method) — **29** numbered inputs, each with its source; every band in §5.2 cites the inputs it uses. **05 is named as a source because inputs 14, 17, 20–26 and 29 are its counts** — the coverage matrix, the approved-delta ledger, the migration artifact, the execution platforms, the diagnostic record, the host and isolation machinery, the destructive-operation controls, the legacy deployment lifecycle and fixture dataset, and the baseline-record compatibility contract; **04 and 05 jointly for inputs 27 and 29** and **06 for input 28** — and this document estimates against them rather than restating them |
| **Net-new work sized honestly** | [§6.2](#62-the-finding-that-matters-most-in-this-document) quantifies it at ~79% of expected effort |
| The **visual review** and **delta sign-offs** carried as tasks with effort | [§7.1](#71-the-manual-visual-review), [§7.2](#72-the-approved-delta-sign-offs) |
| **Net-new work sized honestly** | [§6.2](#62-the-finding-that-matters-most-in-this-document) quantifies it at ~65% of expected effort |
| The **visual review** and **delta sign-offs** carried as tasks with effort | [§7.1](#71-the-manual-visual-review), [§7.2](#72-the-approved-delta-sign-offs) |
| **Net-new work sized honestly** | [§6.2](#62-the-finding-that-matters-most-in-this-document) quantifies everything other than porting at ~67% of expected effort, of which net-new capability alone is ~33% |
| The **visual review** and **delta sign-offs** carried as tasks with effort | [§7.1](#71-the-manual-visual-review), [§7.2](#72-the-approved-delta-sign-offs) |

### 10.4 Cross-reference index

| Deliverable | What this document takes from it | What it takes back |
| --- | --- | --- |
| [01](01-architecture-overview.md) | Verified counts, code volume, view topology, asset groups, the two cart unit-of-work models (inputs 1, 5–7, 9, 10) | — |
| [02](02-dependency-inventory.md) | The as-is pins and the restore-source finding | [R11](#r11--the-effective-package-source-set-is-not-knowable-from-the-repository) |
| [03](03-modernization-roadmap.md) | **The workstream decomposition and every entry and exit gate** — the structure of §5 and §8 | The effort model and the risk register it routes here |
| [04](04-dotnet8-migration-strategy.md) | Target framework, SDK band, the 28 package outcomes, the future application map (input 13), and the **5 operator-host tests** of §12.4 that have no MVC 5 baseline (input 23) | [R1](#r1--the-target-framework-support-window)'s support-window decision, which 04 §2.2 routes here; the estimate for those five tests, carried in W12 |
| [05](05-aspnet-core-migration-approach.md) | Port design, the test suite's architecture and its required coverage rows — **75** as recorded in input 14, a count [05 §12.4](05-aspnet-core-migration-approach.md) owns and this document re-reads rather than fixes — the retained external-login contract, the two data migrations and their **two** per-context change-tracking migrations, the cutover decision, the **27** approved deltas (inputs 14, 17) | [R6](#r6--security-hardening-versus-compatibility), [R7](#r7--the-narrowed-browser-matrix), [R9](#r9--cutover-re-authentication-and-anonymous-cart-loss); [§7.1](#71-the-manual-visual-review)'s review; every duration |
| [06](06-azure-hosting-recommendations.md) | Hosting target, provisioning order, DDL principal, key ring, observability approach, the browser matrix, the **three-regime rollback position with its accepted-write probe** (§11.5) on which [R4](#r4--domain-data-migration-rollback)'s contingency now rests, the **explicit-rotation credential policy** of §12.1 that W12's basis is estimated against, and the **12 CSP tests** of §10.2 — **11** HTTP tests in W7 and the **twelfth**, the deployed-browser gate `G-CSP-BROWSER`, in W10 (input 23) | [R7](#r7--the-narrowed-browser-matrix), [R8](#r8--case-sensitive-path-resolution-on-the-target-platform), [R12](#r12--no-observability-exists-during-the-cutover-itself), [R13](#r13--one-database-one-blast-radius) |
| [08](08-technical-debt-register.md) | **The two counting methods**, the three-part decomposition, the estimation-safe quantities and the forbidden ones (inputs 1–5, 18, 19) | [R3](#r3--the-absent-regression-baseline), [R4](#r4--domain-data-migration-rollback), [R10](#r10--scoping-by-analogy-across-editions) — the three §12.3 asks this document to carry |
| [09](09-security-assessment.md) | The security posture behind [R6](#r6--security-hardening-versus-compatibility); the **security-event catalog** and its 16 classes — **13** application-produced and **3** tooling-produced, per §6.8.1.1's producer map (input 21); the **nine personal-data fields** and the absent retention, deletion and audit controls (input 22); the out-of-support consequence in [R7](#r7--the-narrowed-browser-matrix)'s contingency | [R15](#r15--personal-data-governance-is-unowned), [R16](#r16--no-security-relevant-action-is-recorded-anywhere), and the effort for W16 and W7's security-event sub-row |
| [10](10-build-and-deployment-requirements.md) | **Per-edition build outcomes** — the migration source's status, **blocked pending a Windows verification run**, cited and not restated | [R2](#r2--the-migration-sources-build-reproducibility), estimated against that open status |
| [11](11-cloud-readiness-assessment.md) | Statefulness, transport and path casing as-is, behind [R8](#r8--case-sensitive-path-resolution-on-the-target-platform) and W15 | — |
| [12](12-migration-blockers.md) | **The 23 blockers in two groups** (input 12); the evidence-rather-than-proof qualification | [R4](#r4--domain-data-migration-rollback), [R5](#r5--identity-migration-rollback), [R14](#r14--a-reference-editions-retired-data-provider) |
| [README](README.md) | The requirement-to-deliverable map | This document answering "Estimated effort, risks, and sequencing" |

---

## Appendix A — Reproducibility

Every count this document uses as an estimation input, with the command that reproduces it. Commands
are run from the repository root. Absence claims are included, because an absence has no line to cite
and its command **is** its evidence.

### A.1 The sizing metric (inputs 1–5)

Non-blank lines, excluding `Properties/AssemblyInfo.cs`. This is the **effort-sizing** method of
[§3.1](#31-the-rule-and-why-it-is-the-sharpest-constraint-on-this-document).

```bash
for e in src/MVC3/MvcMusicStore-Completed/MvcMusicStore src/MVC4/MvcMusicStore src/MVC5/MvcMusicStore; do
  files=$(git ls-files "$e/*.cs" | grep -v '/packages/' | grep -v 'Properties/AssemblyInfo.cs')
  printf '%s files=%s nonblank=%s\n' "$e" "$(echo "$files" | wc -l)" \
    "$(echo "$files" | xargs -d '\n' grep -cve '^[[:space:]]*$' | awk -F: '{s+=$NF} END {print s}')"
done
# -> MVC3 files=19 nonblank=1326
#    MVC4 files=26 nonblank=2142
#    MVC5 files=26 nonblank=2097          total 5,565
```

The three-part decomposition of the migration source, from
[08 §4.2](08-technical-debt-register.md), re-verified here because every W7 sub-band depends on it:

```bash
grep -cve '^[[:space:]]*$' src/MVC5/MvcMusicStore/Controllers/AccountController.cs   # -> 382
grep -cve '^[[:space:]]*$' src/MVC5/MvcMusicStore/Models/SampleData.cs              # -> 820
# remainder: 2097 - 382 - 820                                                        # -> 895
# check:  382 + 820 + 895 = 2097
# cross-check, everything but the auth rewrite: 2097 - 382 = 1715 = 820 + 895
```

> **382 is the sizing figure for the authentication rewrite.** The **physical** line count for the same
> file belongs to [08 §3.3](08-technical-debt-register.md)'s duplication comparison and is deliberately
> absent from this document ([§3.2](#32-the-one-substitution-that-would-corrupt-this-document)).

**The seed file's three counts, run so the exclusion in [§3.2](#32-the-one-substitution-that-would-corrupt-this-document)
is checkable rather than asserted.** These two commands are shown **because their results are excluded**,
not because anything uses them:

```bash
F=src/MVC5/MvcMusicStore/Models/SampleData.cs
tail -c 1 "$F" | od -c | head -1     # -> 0000000   }        no terminal newline
wc -l    < "$F"                      # -> 826      line-feed bytes           EXCLUDED
grep -c '' "$F"                      # -> 827      content lines             EXCLUDED
grep -cve '^[[:space:]]*$' "$F"      # -> 820      non-blank, the sizing metric -- the only input
```

The missing final newline is the whole reason `826` and `827` differ, and it is why a physical figure for
this file cannot be cited without also citing which rule produced it. This document cites none of them.

### A.2 The absences that size the net-new work

These four commands are the evidence behind [§6.2](#62-the-finding-that-matters-most-in-this-document):
work that replaces nothing cannot be sized from the thing it replaces.

```bash
# No test of any kind, repository-wide (input 15; R3)
git ls-files | grep -i test | wc -l                                                  # -> 0
git grep -l -E 'TestClass|\[Fact\]|\[Test\]|xunit|NUnit|MSTest' -- '*.cs' '*.csproj' '*.config' | wc -l
                                                                                     # -> 0

# No CI, no publish profile, no container manifest (W11; R12)
git ls-files | grep -c -E '^\.github/|azure-pipelines|\.gitlab-ci|Jenkinsfile'        # -> 0
git ls-files | grep -c -iE 'dockerfile|\.pubxml$'                                     # -> 0

# No observability of any kind (R12)
git grep -l -E 'ILogger|log4net|NLog|Serilog|TraceSource|healthMonitoring' -- '*.cs' '*.config' | wc -l
                                                                                     # -> 0

# No lockfile in any edition (R11)
git ls-files | grep -c 'packages.lock.json'                                           # -> 0
```

### A.3 Helper, view and site counts

**Site counts** and **file counts** — neither line-count method
([§3.3](#33-a-third-kind-of-count-named-so-it-is-not-mistaken-for-either)).

```bash
V=src/MVC5/MvcMusicStore/Views
git ls-files "$V/*.cshtml" | wc -l                                    # -> 29   views (file count)
git grep -o '@Scripts\.Render'        -- "$V/*.cshtml" | wc -l         # -> 10
git grep -o '@Styles\.Render'         -- "$V/*.cshtml" | wc -l         # ->  1   (11 bundling sites)
git grep -o '@Html\.Action('          -- "$V/*.cshtml" | wc -l         # ->  3   child-action call sites
git grep -o '@Html\.AntiForgeryToken(' -- "$V/*.cshtml" | wc -l        # -> 10
git grep -o '@Html\.Partial('         -- "$V/*.cshtml" | wc -l         # ->  5
git grep -o '@Url\.Content('          -- "$V/*.cshtml" | wc -l         # ->  4
git grep -c 'ChildActionOnly' -- 'src/MVC5/MvcMusicStore/Controllers/*.cs' | wc -l
                                                                       # ->  3 controllers declare it

# Input 29 — the tracked documents W14 revises (file count)
git ls-files 'README.md' 'src/MVC4/README.md' 'src/MVC5/README.md' | wc -l   # ->  3
```

**Input 29 is a file count and nothing else.** Three tracked prose documents, enumerated rather than
estimated. No line count of any kind is taken from them: they are markdown, not code, so neither method of
[§3.1](#31-the-rule-and-why-it-is-the-sharpest-constraint-on-this-document) applies to them and no figure in
this document is derived from their length — the same treatment
[§3.3](#33-a-third-kind-of-count-named-so-it-is-not-mistaken-for-either) gives every other file count.

**Manual construction sites — input 10, site count.** Ten, and the census matters because they are not
confined to controller field initializers: three are in startup composition and one is inside a method
body.

```bash
git grep -n -E 'new (MusicStoreEntities|ApplicationDbContext|UserManager<|RoleManager<|UserStore<)' \
  -- 'src/MVC5/MvcMusicStore/Controllers/*.cs' 'src/MVC5/MvcMusicStore/App_Start/*.cs' | wc -l
# -> 10
```

**Static assets — input 7, file count, with its inclusion rule stated.** This document uses the
**28** figure: the migration source's **four asset groups** (`Content`, `Scripts`, `Images`, `fonts`),
which give **27**, **plus the migration source's web-application-root `favicon.ico`**, which the
authoritative map of [04 §12](04-dotnet8-migration-strategy.md) relocates under `wwwroot` with them and
which sits in none of the four groups. It deliberately does **not** use the all-edition four-group count
of 171, nor the all-edition 173 browser-served count that additionally includes **both** editions'
root `favicon.ico` files — neither is the scope of W7's asset relocation, which is the migration source
only. The two commands and their sum:

```bash
git ls-files | grep -cE '^src/MVC5/MvcMusicStore/(Content|Scripts|Images|fonts)/'     # -> 27
git ls-files | grep -cE '^src/MVC5/MvcMusicStore/favicon\.ico$'                       # -> 1
# 27 + 1 = 28
```

**Input 31 — the repository-wide casing census, both of its totals, and its per-edition partition.** This
is the input [W5](#w5--repository-wide-path-casing-audit) is sized on, so it is reproduced in full rather
than by its headline figure. It yields **two** totals that must not be substituted for each other: **258
containers**, the files a checker opens, and **88 literal sites**, the strings inside them that name a
path. Both partition by edition, and both partitions close.

```bash
# Filename-safe by construction: -z terminates each path with NUL and, in doing so, disables git's
# path quoting, so a path containing a space is matched verbatim rather than as "quoted\ text".
# This repository has such a path -- 'src/MVC3/MVC Music Store - Tutorial - v3.0.pdf' -- and no
# command below ever expands a path as a shell word.
A='^src/MVC5/MvcMusicStore/(Content|Scripts|Images|fonts)/'
B='^src/MVC4/MvcMusicStore/(Content|Scripts|Images)/'
C='^src/MVC3/MvcMusicStore-Completed/MvcMusicStore/(Content|Scripts)/'
D='^src/MVC3/MvcMusicStore-Assets/Content/'
F='^src/MVC[45]/MvcMusicStore/favicon\.ico$'

# --- containers, part 1: the 173 browser-served static files
git ls-files -z | tr '\0' '\n' | grep -cE "$A"                # -> 27   MVC 5, four asset groups
git ls-files -z | tr '\0' '\n' | grep -cE "$B"                # -> 89   MVC 4, three groups (no fonts)
git ls-files -z | tr '\0' '\n' | grep -cE "$C"                # -> 51   MVC 3 completed project
git ls-files -z | tr '\0' '\n' | grep -cE "$D"                # ->  4   MVC 3 tutorial assets
git ls-files -z | tr '\0' '\n' | grep -cE "$A|$B|$C|$D"       # -> 171  four asset groups, all editions
git ls-files -z | tr '\0' '\n' | grep -cE "$F"                # ->   2  root favicons: MVC 5 and MVC 4
git ls-files -z | tr '\0' '\n' | grep -cE "$A|$B|$C|$D|$F"    # -> 173  browser-served static files

# --- containers, part 2: the 83 views
git ls-files -z '*.cshtml' | tr '\0' '\n' | grep -cE '^src/MVC5/'                        # -> 29
git ls-files -z '*.cshtml' | tr '\0' '\n' | grep -cE '^src/MVC4/'                        # -> 29
git ls-files -z '*.cshtml' | tr '\0' '\n' | grep -cE '^src/MVC3/MvcMusicStore-Completed/' # -> 21
git ls-files -z '*.cshtml' | tr '\0' '\n' | grep -cE '^src/MVC3/MvcMusicStore-Assets/'    # ->  4
git ls-files -z '*.cshtml' | tr '\0' '\n' | wc -l                                         # -> 83

# --- containers, part 3: the 2 bundle-registration files, neither a view nor a served asset
git ls-files -z '*BundleConfig.cs' | tr '\0' '\n'
# -> src/MVC4/MvcMusicStore/App_Start/BundleConfig.cs
#    src/MVC5/MvcMusicStore/App_Start/BundleConfig.cs
git ls-files -z 'src/MVC3/*App_Start*' | tr '\0' '\n' | wc -l   # ->  0  MVC 3 has no App_Start folder
# 173 + 83 + 2 = 258 containers

# --- literal sites: 88, three kinds
git grep -o -e '"~/[^"]*"' -- 'src/MVC5/MvcMusicStore/App_Start/BundleConfig.cs' | wc -l   # -> 12
git grep -o -e '"~/[^"]*"' -- 'src/MVC4/MvcMusicStore/App_Start/BundleConfig.cs' | wc -l   # -> 24
git grep -o -e '"~/[^"]*"' -- '*BundleConfig.cs' | wc -l                                   # -> 36
git grep -o -E 'new (Script|Style)Bundle\(' -- 'src/MVC5/*BundleConfig.cs' | wc -l          # ->  5
git grep -o -E 'new (Script|Style)Bundle\(' -- 'src/MVC4/*BundleConfig.cs' | wc -l          # ->  6
#   5 + 6 = 11 bundle definitions holding those 36 literals
git grep -o -e '@Url\.Content('        -- 'src/MVC5/*.cshtml' | wc -l   # ->  4
git grep -o -e '@Url\.Content('        -- 'src/MVC4/*.cshtml' | wc -l   # ->  4
git grep -o -e '@Url\.Content('        -- 'src/MVC3/*.cshtml' | wc -l   # -> 23  the concentration
git grep -o -e '@Url\.Content('        -- '*.cshtml' | wc -l            # -> 31
git grep -o -E '@(Scripts|Styles)\.Render' -- 'src/MVC5/*.cshtml' | wc -l  # -> 11
git grep -o -E '@(Scripts|Styles)\.Render' -- 'src/MVC4/*.cshtml' | wc -l  # -> 10
git grep -o -E '@(Scripts|Styles)\.Render' -- 'src/MVC3/*.cshtml' | wc -l  # ->  0
git grep -o -E '@(Scripts|Styles)\.Render' -- '*.cshtml' | wc -l           # -> 21
# 36 + 31 + 21 = 88 literal sites
```

**Both partitions close, which is the check that matters here.** Containers:
`(28 + 29 + 1) + (90 + 29 + 1) + (55 + 25 + 0) = 58 + 120 + 80 = 258`, taking each edition's served count
as its asset groups plus its root `favicon.ico` and MVC 3's views as `21 + 4`. Literal sites:
`(12 + 4 + 11) + (24 + 4 + 10) + (0 + 23 + 0) = 27 + 38 + 23 = 88`. Neither total is a superset of the
other and neither is derivable from the other — 120 of the 258 containers are MVC 4's while 38 of the 88
sites are, and MVC 3 holds 80 containers with only 23 sites — which is why
[W5](#w5--repository-wide-path-casing-audit) states both and sizes the enumeration against the containers
and the correction list against the sites.

**What this census is not.** It bounds the *pattern set the audit must run*, not the audit's *result*: a
literal is a site whether or not its casing is wrong, and the two mismatches
[A.5](#a5-the-corroborating-case-mismatch-r8) demonstrates were found by comparing double-quoted `~/`
literals against the tracked path set — one pattern of the several the audit owes, since a raw `src=`
attribute or a partial-view path names a file without matching that pattern.

### A.4 The visual review capture set (input 16)

**Route and rendered-state count — not a file count.** The capture set of
[§7.1](#71-the-manual-visual-review) is what a reviewer must actually look at, so it is derived from the
routes that render a page and from the states those pages render in. A filename is neither.

**The 18 distinct rendered pages.** Sixteen are reached by a page-rendering `GET` action; the external-login
confirmation page is reached through its `POST` model-invalid redisplay; the eighteenth is the shared error
page, which no route reaches directly. Each is cited at the site that renders it under the committed
configuration, and **all 18 are counted** — row 16 among them, for the reason given under the table.

| # | Page | Rendered by |
| --- | --- | --- |
| 1 | `/` — top sellers | [src/MVC5/MvcMusicStore/Controllers/HomeController.cs:15] |
| 2 | `/Store` — genre list | [src/MVC5/MvcMusicStore/Controllers/StoreController.cs:16] |
| 3 | `/Store/Browse?genre=…` — the catalog grid | [src/MVC5/MvcMusicStore/Controllers/StoreController.cs:27] |
| 4 | `/Store/Details/{id}` — album detail | [src/MVC5/MvcMusicStore/Controllers/StoreController.cs:36] |
| 5 | `/ShoppingCart` — the cart table | [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:15] |
| 6 | `/Checkout/AddressAndPayment` — the checkout form | [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:17] |
| 7 | `/Checkout/Complete/{id}` — order confirmation | [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:68] |
| 8 | `/StoreManager` — the administration list | [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:20] |
| 9 | `/StoreManager/Details/{id}` | [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:30] |
| 10 | `/StoreManager/Create` | [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:43] |
| 11 | `/StoreManager/Edit/{id}` | [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:71] |
| 12 | `/StoreManager/Delete/{id}` | [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:103] |
| 13 | `/Account/Login` | [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:45] |
| 14 | `/Account/Register` | [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:79] |
| 15 | `/Account/Manage` | [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:131] |
| 16 | The external-login confirmation page — captured through its `POST` model-invalid redisplay, not through the `GET` callback; procedure per [05 §12.5](05-aspnet-core-migration-approach.md), render path under the table | [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:294-295] |
| 17 | `/Account/ExternalLoginFailure` | [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:311] |
| 18 | The error page — reached by the global exception filter [src/MVC5/MvcMusicStore/App_Start/FilterConfig.cs:10] and returned by name when a completion request is not the caller's order [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:81] | [src/MVC5/MvcMusicStore/Views/Shared/Error.cshtml:1] |

**Why row 16 is counted: its render path is a `POST`, and that path is deterministic.** The row is cited
at the second of its two render sites rather than the first, because only the second can be reached under
the committed configuration:

- **The `GET` path is closed**, and it is the one an earlier reading of this appendix mistook for the
  only path. `return View("ExternalLoginConfirmation", …)`
  [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:229] sits in the `loginInfo != null` branch of
  `GET /Account/ExternalLoginCallback`, `loginInfo` comes from
  `AuthenticationManager.GetExternalLoginInfoAsync()`
  [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:211], and all four external-provider
  registrations are commented out [src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:23-35], so no
  provider round-trip can make it non-null.
- **The `POST` path is open, and it needs no provider.** `POST /Account/ExternalLoginConfirmation` is
  declared `[HttpPost]`, `[AllowAnonymous]` and `[ValidateAntiForgeryToken]`
  [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:262-265]. Its entire external-login block —
  including the `GetExternalLoginInfoAsync()` call at
  [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:275] — sits behind
  `if (ModelState.IsValid)` [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:272], so an
  **invalid model skips it entirely** and reaches
  `ViewBag.ReturnUrl = returnUrl; return View(model);`
  [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:294-295], which renders the page by action-name
  convention. Invalidating the model takes nothing: `ExternalLoginConfirmationViewModel` declares one
  property, `[Required] UserName` [src/MVC5/MvcMusicStore/Models/AccountViewModels.cs:5-10], so an empty
  value is enough.

**[05 §12.5](05-aspnet-core-migration-approach.md) owns the capture procedure** — a `GET` of a page
carrying a form to obtain a session anti-forgery token, then an anonymous `POST` with that token and an
empty `UserName` — and records the one limitation it carries: the reachable render arrives with a
populated validation summary, which is the state the baseline captures and the state the ported rendering
is compared against. This appendix consumes that procedure and states only its arithmetic consequence,
which is that the row counts. Row 17 is unaffected and always was:
`/Account/ExternalLoginFailure` is a plain `[AllowAnonymous]` `GET`
[src/MVC5/MvcMusicStore/Controllers/AccountController.cs:310] returning its own view
[src/MVC5/MvcMusicStore/Controllers/AccountController.cs:313], reachable by navigation with no
external-provider precondition.

**The 4 further rendered states**, each one a second rendering of a page already counted, and each
required by one of the six checklist areas rather than added for completeness.

| # | State | Why the checklist needs it | Evidence |
| --- | --- | --- | --- |
| 19 | The layout's **signed-in** branch as well as its signed-out one — the greeting and the sign-out control against the register and log-in links | The **navigation bar** is a checklist area, and the layout renders it two ways | [src/MVC5/MvcMusicStore/Views/Shared/_LoginPartial.cshtml:2] |
| 20 | The **cart badge present** as well as absent | Same checklist area: the badge is rendered only above zero, so one capture cannot show both | [src/MVC5/MvcMusicStore/Views/ShoppingCart/CartSummary.cshtml:1] |
| 21 | The **cart table with no rows** as well as populated | The **cart table** is a checklist area, and the row block is a loop with no empty-state markup, so the empty rendering is a different layout | [src/MVC5/MvcMusicStore/Views/ShoppingCart/Index.cshtml:60] |
| 22 | The **checkout form redisplayed with validation messages** | The **checkout form** is a checklist area whose stated concern is label, control and validation-message placement | [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:61] |

**22 rendered states, at the two viewports the layout distinguishes: 44 captures.** The row labels run
1 to 22 with **every row counted** — 18 pages plus 4 further states — which is what lets a reader check
the count rather than take it. [§7.1](#71-the-manual-visual-review) carries the band this set is captured
under, and estimates row 16's capture fixture there.

**The search space the page enumeration was drawn from, bounded by a command so the census can be
re-walked rather than trusted.** There are **27** `return View` sites across the six controllers, and
they partition exactly: **17** render a page by a `GET`, **1** renders the error page by name
(`Checkout`'s ownership branch, page 18 above), **8** are POST redisplays of a page already counted, and
**1** is a POST rendering the external-login failure page already counted
[src/MVC5/MvcMusicStore/Controllers/AccountController.cs:278]. 17 + 1 + 8 + 1 = 27.

**That census counts render sites in source rather than capturable states, which is why its 17 `GET` sites
and the sixteen `GET`-reached pages above are both right.** The `GET` site at
[src/MVC5/MvcMusicStore/Controllers/AccountController.cs:229] is one of the 17 and its page is still
captured through the `POST` redisplay, because a site that exists in source is not the same as a state a
reviewer can reach.

**And it is worth saying what this census already contained.** The `POST` redisplay at
[src/MVC5/MvcMusicStore/Controllers/AccountController.cs:295] that makes row 16 capturable is **one of
those 8 POST redisplays** — so the evidence against an earlier reading that excluded row 16 as
unrenderable was sitting in this appendix's own census the whole time, below the very row it contradicted,
and nobody read the two against each other. In a document about estimation risk that observation is worth
more than the count it corrects: a census that is *correct* is not yet a census that has been *read
against the claims beside it*, and an enumeration kept for auditability only earns its keep when someone
performs that cross-check. The partition itself is unchanged by the restoration, because 27, 17, 1, 8 and
1 count sites and not states.

```bash
git grep -c 'return View' -- 'src/MVC5/MvcMusicStore/Controllers/*.cs'
# -> AccountController.cs:10   CheckoutController.cs:5   HomeController.cs:1
#    ShoppingCartController.cs:1   StoreController.cs:3   StoreManagerController.cs:7   = 27
git grep -c 'PartialView' -- 'src/MVC5/MvcMusicStore/Controllers/*.cs'
# -> AccountController.cs:1   ShoppingCartController.cs:1   StoreController.cs:1        =  3
```

**The second command is the reason a fragment cannot be counted as a page.** All three child actions
return a **partial** rather than a view — [src/MVC5/MvcMusicStore/Controllers/StoreController.cs:54],
[src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:98],
[src/MVC5/MvcMusicStore/Controllers/AccountController.cs:321] — so two of them render through
non-underscore filenames while never being a page in their own right.

**The file count, kept only as a cross-check on the view topology.** The 29 views resolve into 22
non-underscore files and 7 underscore-prefixed ones — five partials, the shared layout and the
view-start file.

```bash
git ls-files 'src/MVC5/MvcMusicStore/Views/*.cshtml' | grep -vE '/_[^/]*\.cshtml$' | wc -l   # -> 22
git ls-files 'src/MVC5/MvcMusicStore/Views/*.cshtml' | grep -cE '/_[^/]*\.cshtml$'           # ->  7
```

> **That 22 is a file count, and it is not the 22 rendered states above.** The two censuses are
> **independent and share no derivation**: one enumerates files on disk, the other enumerates states a
> reviewer has to look at, and **their agreeing on 22 is a coincidence.** That is the warning this note
> exists to carry, and it matters most exactly when the numbers match — a reader who substitutes one for
> the other lands on the right total by accident and on the wrong set by construction. Four members
> differ. The file count includes **two views no action renders** —
> `Views/Home/About.cshtml` and `Views/Home/Contact.cshtml`, against a controller whose only action is
> `Index` [src/MVC5/MvcMusicStore/Controllers/HomeController.cs:15] — and **two child-action fragments
> that render inside another page rather than as one**, `Views/Store/GenreMenu.cshtml` and
> `Views/ShoppingCart/CartSummary.cshtml`, both invoked from the shared layout
> [src/MVC5/MvcMusicStore/Views/Shared/_Layout.cshtml:25],
> [src/MVC5/MvcMusicStore/Views/Shared/_Layout.cshtml:26]. It omits all four state variants above.
> **A partial view is not a page and a filename is not a state**, which is why input 16 is a route and
> rendered-state count.

### A.5 The corroborating case mismatch (R8)

The primary mismatch is cited inline at
[src/MVC5/MvcMusicStore/App_Start/BundleConfig.cs:28] against
[src/MVC5/MvcMusicStore/Content/Site.css: the tracked path's own capitalisation, not a line within the
file] — the registration has a line, the filename does not, and the command below prints the second half
of the pair. The corroboration — that this repository is systemically careless about case rather than
carrying one typo — is now **three** independent instances: the pair above, **the identical pair in MVC 4**,
and the repository's own ignore file.

```bash
# The primary mismatch, both halves
grep -n 'site.css' src/MVC5/MvcMusicStore/App_Start/BundleConfig.cs  # -> 28: ~/Content/site.css
git ls-files 'src/MVC5/MvcMusicStore/Content/*'                      # -> .../Content/Site.css

# The same mismatch in the retained MVC 4 edition, both halves. One command covers both files:
git grep -n 'Content/site.css' -- '*BundleConfig.cs'
# -> src/MVC4/MvcMusicStore/App_Start/BundleConfig.cs:26: ...StyleBundle("~/Content/css")
#                                                          .Include("~/Content/site.css"));
# -> src/MVC5/MvcMusicStore/App_Start/BundleConfig.cs:28:  "~/Content/site.css"));
git ls-files 'src/MVC4/MvcMusicStore/Content/*.css' | head -1         # -> .../Content/Site.css

sed -n '28p' .gitignore                                        # -> nuget.exe        (lowercase)
git ls-files 'src/MVC4/MvcMusicStore/.nuget/*'                 # -> .../NuGet.exe    (capital N, G)

# --no-index is REQUIRED here. Without it, git excludes tracked paths from ignore
# evaluation entirely and prints nothing whatever the rules say:
git check-ignore -v src/MVC4/MvcMusicStore/.nuget/NuGet.exe ; echo "exit=$?"
# -> no output, exit=1     <- says nothing about rule matching; the path is tracked

git check-ignore --no-index -v src/MVC4/MvcMusicStore/.nuget/NuGet.exe ; echo "exit=$?"
# -> .gitignore:28:nuget.exe    src/MVC4/MvcMusicStore/.nuget/NuGet.exe
# -> exit=0
```

**What the MVC 4 pair establishes.** The mismatch is not a single typo in the one edition being ported: the
same lowercase `~/Content/site.css` is registered against the same capitalized `Content/Site.css` in a
second, independently maintained project. That is why
[W5](#w5--repository-wide-path-casing-audit) sizes its enumeration repository-wide and has to record an
enforcement boundary — this instance is real, it is in an edition retained read-only, and a checker that
fails on any mismatch would fail on it forever.

**What the `git check-ignore --no-index` command establishes, stated precisely.** On the
**case-insensitive** filesystem this
assessment was performed against, the lowercase pattern `nuget.exe` **does** match the capitalized path
`NuGet.exe`. On a **case-sensitive** filesystem it would not. That filesystem-dependence is the
corroboration [R8](#r8--case-sensitive-path-resolution-on-the-target-platform) rests on: a written path
and a real path whose cases differ, indistinguishable on the development platform and divergent on the
target one — the same class of defect as `~/Content/site.css` against the tracked `Content/Site.css`.

**Two readings this evidence does not support**, both of which the first command's exit code invites:

- **The plain command's exit 1 is not evidence that no rule matches.** `git check-ignore` without
  `--no-index` skips tracked paths by design, so it exits 1 for *any* tracked file regardless of the
  ignore rules. Citing that exit code as a finding about rule matching would be citing a property of the
  index as a property of `.gitignore`.
- **The case mismatch is not why the binary is tracked.** A path already in the index is unaffected by
  any later ignore rule — the same reason the committed package trees and the `App_Data` binaries remain
  tracked. [08 §10.7](08-technical-debt-register.md) owns that as a hygiene finding; this appendix
  establishes only the case discrepancy.

### A.6 The constraint this work was held to

**Four commands, and they only mean anything together.** The committed diff against the pre-assessment
baseline is what says *what this work added* — a `git status --porcelain` run at the committed checkpoint
is empty because the thirteen files are committed, so it evidences a clean tree and nothing more. The two
ignore-aware commands are what say *that nothing generated is left behind*, and they are not optional
here: the assessment's restore and build runs wrote **eight gitignored trees** into this checkout, and
neither porcelain nor a tracked-file diff can see an ignored path
([§2](#2-the-no-modification-constraint) states which rules make them invisible). All eight were removed
before this checkpoint, and their absence is what the two ignore-aware forms below establish. The block
closes with an ignore-rule probe that names the rule hiding each tree; it is **explanatory and not a
fifth acceptance command**:

```bash
git diff --name-status ea2552d6eda7c20e9477a512e5c615665618cf35..HEAD
# (git separates the status letter from the path with a tab; shown here as spaces)
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

git diff --name-status ea2552d6eda7c20e9477a512e5c615665618cf35..HEAD \
  | grep -cE '^A[[:space:]]+docs/modernization/'
# -> 13                    every row is an addition under docs/modernization/

git diff --name-status ea2552d6eda7c20e9477a512e5c615665618cf35..HEAD \
  | grep -cvE '^A[[:space:]]+docs/modernization/'
# -> 0                     and there is nothing else: no M, no D, no other path

git status --porcelain
# -> (empty output) on the committed checkout — the tree is clean, which is why porcelain
#    cannot be the evidence for what this work added; it is non-empty only while
#    uncommitted edits are in flight

git status --porcelain --ignored
# -> (empty output) — and specifically no "!!" row: no ignored payload or output tree
#    remains anywhere in the checkout. Bare porcelain above cannot report this, because
#    obj/ and bin/ are ignored [.gitignore:1], [.gitignore:2] and the two nested packages
#    trees by Packages/ [.gitignore:33] — not by packages/* [.gitignore:15], which is
#    anchored to the repository root. Section 2 states that rule and its case dependency

git clean -ndX
# -> (empty output) — a dry run that removes nothing and lists every ignored file that
#    would be removed. It is the one command of the four that answers the question in the
#    form it is asked: is any ignored artifact present?

git check-ignore -v --no-index src/MVC4/packages/x src/MVC5/packages/x \
  src/MVC5/MvcMusicStore/bin/x src/MVC5/MvcMusicStore/obj/x
# -> .gitignore:33:Packages/   src/MVC4/packages/x
#    .gitignore:33:Packages/   src/MVC5/packages/x
#    .gitignore:2:[Bb]in/      src/MVC5/MvcMusicStore/bin/x
#    .gitignore:1:[Oo]bj/      src/MVC5/MvcMusicStore/obj/x
#    --no-index is required: it tests the rule rather than the path's tracked state. Adding
#    -c core.ignorecase=false leaves the two packages paths unmatched, which is the case
#    dependency section 2 records. This probe explains which rule hid which tree; it is not
#    one of the four commands the acceptance check consists of
```

**The eight trees, and the honest standing of their figures.** They were
`src/MVC3/MvcMusicStore-Completed/MvcMusicStore/bin` and `obj`, `src/MVC4/MvcMusicStore/bin` and `obj`,
`src/MVC4/packages`, `src/MVC5/MvcMusicStore/bin` and `obj`, and `src/MVC5/packages`, holding **527
files and 114,310,394 bytes**. Both figures are **measurements taken immediately before removal, not
counts re-derivable from this checkout**: the trees are gone, so the two ignore-aware commands above now
return nothing and no command run today can reproduce 527 or 114,310,394. They are quoted as a record of
what was removed, and the per-tree breakdown belongs to
[10 §1.4](10-build-and-deployment-requirements.md). Neither figure is an estimation input — it is not in
[section 4.1](#41-the-estimation-basis-every-input-with-its-method) and no band derives from it.

**What each half establishes, stated to match what it can actually see.** The diff establishes that
thirteen files were added — the deliverables of this assessment, this document among them — and that
**nothing tracked was modified or deleted**: no pre-existing file appears in it, no path outside
`docs/modernization/` appears in it, and the `grep -cvE` count of `0` is what proves there is no
fourteenth row. The two ignore-aware commands establish the separate claim the diff is blind to: **no
build output and no restored package directory remains**, the eight trees the assessment's runs created
having been removed before this checkpoint. Neither half substitutes for the other, and together they
are the AAP §0.11.5 criterion exactly. Every command above is read-only — they inspect the index, the
working tree, committed history and the exclude rules, and `git clean` appears only in its `-n` dry-run
form, which lists and removes nothing.

### A.7 Secondary cross-reference

Technical Specification §1.3, §3.3 and §6.4 were available as **secondary** references. Under the
citation policy of AAP §0.4.1 the repository governs, and every count in this appendix is established
from the repository rather than from the specification. Where the two disagree, the disagreement is
recorded by the deliverable that owns the fact — [02 §6](02-dependency-inventory.md) for the restore
source, [01 §9](01-architecture-overview.md) for per-capability edition coverage — and not adjudicated
here.

### A.8 External primary sources

**Six claims in this document cannot be established from the repository**, because they are properties
of a support lifecycle, a hosting platform, a framework's published browser support or a tool's
documented behaviour. Each carries its source inline at the point of claim; they are collected here so
a reviewer can re-check the set in one pass. All six were reachable and read on the verification date.

| # | Claim it supports | Source | Where it is cited |
| --- | --- | --- | --- |
| 1 | .NET 8 is LTS, shipped 14 November 2023, and its support closes in November 2026; .NET 10 is the current LTS through November 2028 | Microsoft Learn, *.NET releases and support policy*, <https://learn.microsoft.com/dotnet/core/releases-and-support> — verified 2026-08-28 | [R1](#r1--the-target-framework-support-window) |
| 2 | The **exact** end-of-support day, **10 November 2026**, for .NET 8 and for .NET 9 — the Learn page gives the month, this table gives the day | Microsoft, *.NET and .NET Core Support Policy*, <https://dotnet.microsoft.com/platform/support/policy/dotnet-core> — verified 2026-08-28 | [R1](#r1--the-target-framework-support-window) |
| 3 | The target styling-framework major does not support Internet Explorer in any version | Bootstrap, *Browsers and devices*, <https://getbootstrap.com/docs/5.3/getting-started/browsers-devices/> — verified 2026-08-28 | [R7](#r7--the-narrowed-browser-matrix) |
| 4 | Static-file URLs inherit the underlying filesystem's case sensitivity; Windows is case-insensitive, Linux is not | Microsoft Learn, *Static files in ASP.NET Core*, <https://learn.microsoft.com/aspnet/core/fundamentals/static-files> — verified 2026-08-28 | [R8](#r8--case-sensitive-path-resolution-on-the-target-platform) |
| 5 | `git check-ignore` does not show tracked files by default, because tracked files are not subject to exclude rules; `--no-index` is what tests the rule | Git, *git-check-ignore Documentation*, <https://git-scm.com/docs/git-check-ignore> — verified 2026-08-28 | [R8](#r8--case-sensitive-path-resolution-on-the-target-platform), [A.5](#a5-the-corroborating-case-mismatch-r8) |
| 6 | A platform secret reference resolves an app setting from the key store under the site's managed identity | Microsoft Learn, *Use Key Vault references as app settings in Azure App Service and Azure Functions*, <https://learn.microsoft.com/azure/app-service/app-service-key-vault-references> — verified 2026-08-28 | [R15](#r17--the-interim-hosting-paths-stored-credential-and-a-time-box-with-no-box) |

**Two boundaries on this table.** It carries only what *this* document's own claims rest on. Every other
external fact the assessment uses — package versions and their servicing band, the migration tooling's
status, the hosting comparison, the adapters' targeting — is cited by the deliverable that owns it, and
citing it again here would create a second owner for it. And no package-registry source appears above,
because this document names no package version: the pins are [04 §7](04-dotnet8-migration-strategy.md)'s
and the test-stack versions are cited there, not here.
