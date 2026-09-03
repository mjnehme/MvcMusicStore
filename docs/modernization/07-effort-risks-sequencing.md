
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
| **The browser matrix, and the ruling that the script-issued cart-removal flow is walked manually in every family of it** | [06](06-azure-hosting-recommendations.md) §10.4 | [R7](#r7--the-narrowed-browser-matrix) carries the compatibility loss and [R18](#r18--the-browser-executed-half-of-the-scripted-cart-flow-one-pinned-engine-and-an-open-decision-on-the-other-two) what the walk leaves open. Neither the matrix nor the walk's own contract is restated: [03 §5](03-modernization-roadmap.md) gates it as W10's exit condition 8, and this document only **prices** it, at [W10](#w10--hosting-provisioning-and-platform-configuration)'s gate 10b |
| The DDL principal and the provisioning order, **including the fixed order of the data movement** | [06](06-azure-hosting-recommendations.md) §6.2, §6.3 | The scope behind W10. That order is enforced **inside the two runs that move data** rather than by a workstream-level wait [03 §4.2](03-modernization-roadmap.md), and [§8.2](#82-concurrency-permitted-by-the-graph) records what the graph therefore permits |
| **The interim hosting option's authentication path** — selected, with its owner and its two exit triggers | [06](06-azure-hosting-recommendations.md) §5.3–§5.5 | [R17](#r17--the-interim-hosting-paths-stored-credential-and-a-time-box-with-no-box) carries the residual that the exception outlives its term. The selection, its cost and its own residual risks are not restated |
| **Per-edition build outcomes**, and the migration source's build status | [10](10-build-and-deployment-requirements.md) §1.2 (the carried status), §3.1 and §5.4 (the two observed runs), §2.2 (what each counts as) | [R2](#r2--the-migration-sources-build-reproducibility) cites the carried status — *blocked pending a Windows verification run* — and the observed outcomes recorded beneath it. Neither the status, the outcomes nor their diagnosis is restated |
| **The 23 blockers and their two groups** | [12](12-migration-blockers.md) §2.3 | Work items behind W7's estimate |
| **The categorized debt register and the counting methods** | [08](08-technical-debt-register.md) §2, §5–§11 | The counting rule this document is bound by, and an estimation input |
| Every package pin as-is; the restore-source finding | [02](02-dependency-inventory.md) §3, §6 | Scope behind W6; [R11](#r11--the-effective-package-source-set-is-not-knowable-from-the-repository) carries the source finding as a risk |
| Verified counts, code volume, view topology, asset groups | [01](01-architecture-overview.md) §2.3–§2.5 | Estimation inputs, cited per figure |
| **The set of approval-owned additions and their owners** | [05](05-aspnet-core-migration-approach.md) §11.7 | The second approval register. [Section 7.2](#72-the-approved-delta-sign-offs) sizes obtaining these decisions too, and states its two counts separately |

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
commit; the per-tree record belongs to [10 Appendix A](10-build-and-deployment-requirements.md#appendix-a--reproducibility) and
the standing rule that follows from it to [10 §1.4](10-build-and-deployment-requirements.md). **The trees are a hygiene record rather than build evidence**, and
**exactly one of the runs behind them is build evidence** — the MVC 5 restore and rebuild that [10 §3.2](10-build-and-deployment-requirements.md) records as the post-freeze Windows observation and holds to supplementary observation, which cannot move the carried status, so
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
`git diff --name-status ea2552d6eda7c20e9477a512e5c615665618cf35..HEAD`,
which returns exactly thirteen rows, every one an `A` for a file under `docs/modernization/`, with no
`M` or `D` against any existing file. **Both endpoints of that range are defined here rather than left to inference, and the asymmetry
between them is deliberate.** The left side is the **immutable pre-assessment revision** — the last
commit before this assessment began, a hash that cannot move, which is what makes the range reproducible
from any checkout. The right side is `HEAD`, meaning **the delivery commit the reader has checked out**,
and it is named that way because **a document cannot cite the commit that creates it**: no hash is
invented for one, so the pin sits on the left and the expected result is stated as a shape — thirteen
`A` rows, all under `docs/modernization/` — rather than as a listing a fourteenth file would expire.
Every other revision reference in this document is to `ea2552d` alone and is a point rather than a
range. Deliverables [04 §1.4](04-dotnet8-migration-strategy.md#14-the-no-modification-constraint-and-the-boundary-that-makes-this-document-possible)
and [10 Appendix A](10-build-and-deployment-requirements.md#appendix-a--reproducibility) define the same
two endpoints, and this document uses no others. Working-tree status alone is **not** the check twice over: at the
committed checkpoint `git status --porcelain` is empty, so it evidences a clean tree rather than what
this work changed, and by itself it could not have seen the eight trees at all. All four, with their
observed output, are in [A.6](#a6-the-constraint-this-work-was-held-to).

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
> [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:1-423]. The locator is the whole file,
> written as the range of lines it spans — the file has no terminal newline, so the range ends on its
> unterminated last line — because the claim *is* a whole-file figure. That range is a citation bound
> and not a metric: the figure it supports is the 382, and its reproducing command is
> [A.1](#a1-the-sizing-metric-inputs-15).
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
> sizing count gives **820**
> [src/MVC5/MvcMusicStore/Models/SampleData.cs:1-827]. The locator is the whole file at its
> content-line extent, the 827 above, and all three counting rules are run in
> [A.1](#a1-the-sizing-metric-inputs-15). All three are correct answers to different
> questions. **Only 820 is an input to this document**, and neither 826 nor 827 appears in any figure
> below. Where another deliverable states a physical figure for this file, it is answering a
> duplication-or-weight question under its own rule and is not comparable with anything in
> [section 5](#5-effort-by-workstream) — which is exactly the substitution
> [§3.2](#32-the-one-substitution-that-would-corrupt-this-document) forbids, one file further on.

### 3.3 A third kind of count, named so it is not mistaken for either

Some inputs are **file counts** or **call-site counts** — 29 views, 27 static assets, 10 injection
sites, 23 blockers. These are neither line-count method. They are labelled **"file count"** or
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
exclude an **enumerated contract obligation list**, whose members are alike by construction: the rows of
the approved-delta register (input 17), the console's eight data-migration verbs and the five-code
tool-wide exit allocation of [05 §10.2](05-aspnet-core-migration-approach.md), the three
execution categories (input 14), the allow-listed report fields, seven masked material classes and
twenty-six `operation` codes of [05 §12.9](05-aspnet-core-migration-approach.md), the three-line service
allow-list of [05 §12.6](05-aspnet-core-migration-approach.md) and the twelve collection-fixture
groups of [05 §12.7](05-aspnet-core-migration-approach.md), the seven lifecycle steps, seven post-load
invariants and twelve committed fixture inputs of
[05 §12.3](05-aspnet-core-migration-approach.md), the eleven gating
`baselineSource` values and four pinned locale and collation values of
[05 §12.10](05-aspnet-core-migration-approach.md), and the thirteen blocking checks
of the deployed verification gate in [06 §12.1](06-azure-hosting-recommendations.md). Each of those
is a list of items of one kind, each item carrying comparable work, which is why they are admissible as
inputs while a finding count is not. Where a figure below is labelled "entry count", it is one of these,
and what names the enumeration is the citation beside it: an input row of
[§4.1](#41-the-estimation-basis-every-input-with-its-method) where the enumeration is itself a numbered
estimation input, and otherwise — as for [05 §10.2](05-aspnet-core-migration-approach.md)'s verb table
above — the section of the owning deliverable that publishes the list.

---

## 4. The effort model

### 4.1 The estimation basis: every input, with its method

**Every workstream band in [section 5](#5-effort-by-workstream) derives from a row below, and the scope of
that claim is stated precisely because an earlier form of it was broader than the document could keep.**
It read *"every figure in section 5 derives from a row below, or — where the quantity
is an enumerated contract obligation this table does not number as an estimation input — from the list
published by the deliverable section cited beside it, in the second of the two forms
[§3.3](#33-a-third-kind-of-count-named-so-it-is-not-mistaken-for-either) defines. No other quantity is used."* — which was
false of the **gate sub-rows**, whose cells enumerate their own items (operations, members, lifecycle
steps, override files, timeout layers, pinned values) and are sized from that enumeration rather than from
a register row. Those sub-rows had each carried an `(input NN)` parenthetical pointing at a register row
whose subject was something else entirely, which is the failure a broad claim invites: a pointer added to
satisfy the rule rather than because it resolved. Every one of them now names its real basis — the count in
its own cell — and the claim is narrowed to what holds:

- **Every workstream-level band, every roll-up and every total** in section 5 derives from a row below,
  and cites the row it derives from.
- **A gate sub-row inside a workstream** may instead be sized from the enumeration in its own cell, which
  it states as its basis. It introduces no quantity that is not visible in the cell.
- **No quantity anywhere in this document is asserted without one of those two bases**, and a register row
  is cited only where that row actually supplies the figure.

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
| 14 | Required **parity** test coverage | **104** rows carrying **326** cases. **32** rows run against **two** fixtures and **7** are mixed, so **152** cases execute twice and **174** execute against the target alone: **478** fixture executions, being **152** `Category=Baseline`, **325** `Category=Target` and **1** `Category=Deployed`. The class split behind those figures is **32** rows / **136** cases both-fixture, **7** / **29** mixed and **65** / **161** target-only, and `136 + 29 + 161 = 326`, `152 + 174 = 326`, `152 × 2 + 174 = 478`. This is the count of the cross-baseline suite and **not** of the whole test workload — see the inclusion rule below, input 23 and input 32. **This row is the only place these figures enter this document**: every other passage that needs one cites input 14 rather than repeating it, because a figure repeated in eight places moves in some of them. **They are re-read from the owner's table rather than carried as constants here**, because the row count has now moved **six** times — 103, then 108, then 116, then 115 (a withdrawal rather than an addition), then 75 when the owner restructured that table into a case-level matrix, then 102, and now **104**: `awk '/^### 12\.4 /,/^### 12\.5 /' docs/modernization/05-aspnet-core-migration-approach.md \| grep -cE '^\| [0-9]+ \|'` → 104, contiguous 1–104 (the same extraction through `sort -n \| uniq \| wc -l` also yields 104, so there is no duplicate and no gap) | Row count and case count, command-verified | [05 §12.4](05-aspnet-core-migration-approach.md) |
| 15 | Tests existing today | **0** | Absence, command-verified | [08 §7.3](08-technical-debt-register.md), [A.2](#a2-the-absences-that-size-the-net-new-work) |
| 16 | Distinct captures for visual comparison | **28** capture states over **18** of the 29 Razor artifacts, **two** of which — `Home/About.cshtml` and `Home/Contact.cshtml` — can produce **no baseline at all**, times **2** viewports and **4** browser families, for **170** comparison pairs | Semantic classification of all 29 artifacts, plus a reachability census of the actions that render them, plus the two declared review dimensions — **not** a filename count | [A.4](#a4-the-visual-review-capture-set-input-16), [§7.1](#71-the-manual-visual-review) |
| 17 | Approved deltas requiring sign-off | **Every row of the register in [05 §11.5](05-aspnet-core-migration-approach.md) that carries an approval owner** — its **two** retired identifiers, `D-22` and `D-43`, carry none and need no decision — the register stating its own count, which this document does not restate, across **5** approver constituencies (security, product, engineering, the data owner and operations), with a substantial minority of those rows naming more than one and that subcount likewise the register's to state. **Its two dimensions are partitioned across two rows**: the 5 constituencies size [W1](#w1--approval-of-this-assessment); the individual delta decisions size [§7.2](#72-the-approved-delta-sign-offs), which is the one place a priced decision count appears. Neither row counts the other's dimension | Entry count | [05 §11.5](05-aspnet-core-migration-approach.md) |
| 18 | Unvalidated state-changing POSTs | **5** in the migration source | Census | [08 §5.5](08-technical-debt-register.md) |
| 19 | Repository-hygiene volume | **14** database binaries at **43,376,640** bytes; **215** committed package files; **4** solutions for **3** projects | File count and byte count | [08 §6.2, §10.4, §10.2](08-technical-debt-register.md) |
| 20 | Source files carrying a legacy namespace directive | **19** of the migration source's **27** `.cs` files — **corroborating the sizing inputs rather than sizing work of its own**. It is the one row in this table whose figure no band in [section 5](#5-effort-by-workstream) multiplies by a rate, and that is deliberate rather than an omission: the cost of rewriting those files is **already inside the non-blank-line sizing** of inputs 2 and 4 that [W7](#w7--the-aspnet-core-port) is estimated against, so charging the file count as well would price the same work twice — the class of error [§3.1](#31-the-rule-and-why-it-is-the-sharpest-constraint-on-this-document) forbids for physical line counts, arrived at from the other direction. What the row does establish is the **extent** of the rewrite — that it reaches 19 of the 27 files rather than the `Controllers` folder alone, the reading under which a port plan treats the model layer as untouched — which is why [W7](#w7--the-aspnet-core-port) names it among its inputs and why the row is retained rather than deleted | File count, **corroborating**: no band derives from it | [05 §9.1](05-aspnet-core-migration-approach.md) |
| 21 | Security-event classes in the catalog — **13** produced by the ported application and **2** by tooling, the sixteenth having **no producer** in this plan's artifacts | **16** | Row count, split by [09 §6.8.1.1](09-security-assessment.md)'s producer map | [09 §6.8, §6.8.1.1](09-security-assessment.md) |
| 22 | Personal-data fields under governance | **9** on the order record, plus the identity link | Field count | [09 §3.11](09-security-assessment.md) |
| 23 | Required tests with **no** MVC 5 baseline and outside input 14's numbered index | **12** — the **10** operator rows `O1`–`O10` of [05 §12.4.1](05-aspnet-core-migration-approach.md), which drive the built provisioning executable as a process, and **2** required deployed-browser items, both **manual** and both outside every index because nothing in the programme executes a page: `G-CSP-BROWSER`, since no HTTP client can observe policy enforcement by an agent, `report-to` precedence or non-double-delivery; and the **four-engine functional walk** of the script-issued cart-removal flow that [06 §10.4](06-azure-hosting-recommendations.md) requires, since no HTTP client observes the request that flow's script issues or the DOM it rewrites. Each is **one** item covering the whole matrix rather than one per engine, because each is a single reviewer pass with a single signed artifact. The HTTP-observable content-security-policy tests are **not** here: they are **numbered rows 76–81** of input 14. `9`, `11`, `14` and `17` are the stale forms | Test count | [05 §12.4.1](05-aspnet-core-migration-approach.md), [06 §10.2, §10.4](06-azure-hosting-recommendations.md) |
| 24 | Assembly references that cannot resolve before a restore | **46** `<Reference>` elements in the migration source, of which **26** carry a `HintPath` under `..\packages\` and **20** resolve from the framework or machine-wide | Reference count | [02 §4.2](02-dependency-inventory.md) |
| 25 | Observability artifacts existing today | **0** — no logging abstraction, logging framework, `TraceSource`, ASP.NET health-monitoring configuration, health endpoint or metric, in any edition. The full census and its command are [08 §7.1](08-technical-debt-register.md)'s, whose pattern set includes `HealthCheck` and `healthMonitoring`; [A.2](#a2-the-absences-that-size-the-net-new-work) reproduces the logging and `healthMonitoring` portion of it here | Absence, command-verified | [08 §7.1](08-technical-debt-register.md), corroborated by [A.2](#a2-the-absences-that-size-the-net-new-work) |
| 26 | CI, deployment-automation and publish artifacts existing today | **0** — no pipeline definition, publish profile or container manifest | Absence, command-verified | [08 §7.2](08-technical-debt-register.md), [A.2](#a2-the-absences-that-size-the-net-new-work) |
| 27 | Application configuration files, and the runtime state they declare | **15** application `.config` files, of which **0** declare `<sessionState>` and **0** declare `<machineKey>` — so session storage and key material are both framework defaults | File count and census | [01 §6.6](01-architecture-overview.md) |
| 28 | Stores and entities the production load must reconcile | **2** stores — the catalog store and the Identity store, coupled only by convention with no foreign key between them — over **6** catalog entities | Store and entity count | [01 §6.1, §6.3, §6.5](01-architecture-overview.md) |
| 29 | Tracked documents describing the workflow the target replaces | **3** — the root README and the two per-edition READMEs | File count, command-verified | [A.3](#a3-helper-view-and-site-counts) |
| 30 | Behaviour the provisioning tool must implement, outside its added tests | **5** required properties; **4** independently converged operations on a provisioning run — `create-user`, `create-role`, `add-membership` and `run`, the closed discriminator [05 §10.2](05-aspnet-core-migration-approach.md) property 4 fixes together with four records per invocation. **The count is unchanged and the naming is the owner's**: an earlier form of this row named them "role, user, credential, membership", which [03 §5 W12](03-modernization-roadmap.md) condition 1 **withdraws** because a credential operation of its own would be a fifth operation under a four-record contract; the credential verdict it was protecting stays independently observable as the `create-user` record's own outcome. And `PROV-6001`'s closed outcome vocabulary of **18** values, being **12** non-failure and **6** failure, **of which 15 are exercisable and 3 are reserved** — [09 §6.8.1](09-security-assessment.md) closes the set at eighteen and reserves three that presuppose a removal or a resolve-without-create no operation of this tool performs. **The partition is derived from that section's per-operation mapping rather than asserted**, and it has to say how the shared value is counted: `create-user` **4**, `create-role` **2**, `add-membership` **4** and `run` **1**, plus the `NotAttempted` all three operation records share and which is therefore counted once — `4 + 2 + 4 + 1 + 1 = 12`. Restricted to the values an invocation can actually reach it is `4 + 2 + 2 + 1 + 1 = 10`, which with the **5** exercisable failure outcomes is the fifteen. **A third dimension is withdrawn rather than deleted**: an earlier form of this row also counted a revoke mode's two operations across three branches, which [05 §10.2](05-aspnet-core-migration-approach.md) does not specify — the prose below this table records the withdrawal and its band consequence | Property, operation and outcome count | [05 §10.2](05-aspnet-core-migration-approach.md) (properties and operations), [09 §6.8.1](09-security-assessment.md) (the outcome vocabulary, which that section owns) |
| 31 | Path literals the **repository-wide** casing audit must examine | Inputs 7, 8 and 11 are **migration-source only**, and the audit is not. Repository-wide, all three editions: **173** browser-served static files (**171** in the four asset groups plus the two web-application-root `favicon.ico` files); **83** Razor views; **11** bundle definitions across the two `BundleConfig.cs` files, carrying **36** `~/`-rooted literals between them; **31** `@Url.Content` occurrences — 4 in MVC 5, 4 in MVC 4 and **23** in MVC 3, which is where the concentration is; and **21** `@Scripts.Render`/`@Styles.Render` sites, of which 11 are MVC 5's. MVC 3 has no `App_Start` folder and therefore no bundle definitions at all. **The two totals this input yields are 258 containers — `173 + 83 + 2`, the two `BundleConfig.cs` files being neither views nor served assets — holding 88 literal sites, `36 + 31 + 21`**; [A.3](#a3-helper-view-and-site-counts) reproduces every figure and both per-edition partitions | Site and file count, command-verified | [A.3](#a3-helper-view-and-site-counts), [01 §2.3, §2.5](01-architecture-overview.md), [06 §3.4](06-azure-hosting-recommendations.md) |
| 32 | Approval-owned additions requiring a recorded decision | **16**, ids `A1`–`A16` — net-new user-perceptible behaviour with **no** MVC 5 counterpart, each carrying an approval owner. Disjoint from input 17 by construction: a delta changes an outcome the source already produces, an addition introduces one it never produced. Both registers' decisions are sized in [§7.2](#72-the-approved-delta-sign-offs) and neither is folded into the other | Entry count | [05 §11.7](05-aspnet-core-migration-approach.md) |
| 33 | Required tests of the **published console bytes**, outside input 14's matrix | **10** — the lettered rows O1–O10 of [05 §12.4.1](05-aspnet-core-migration-approach.md): parser closure, startup validation before anything else, the environment assertion, every verb doing its work, partial-run idempotence, the batch and resume bounds, **four** audit records per invocation in the pinned operation order on all four paths, the secret appearing nowhere in any capture, the literal exit-code allocation, and a normal builder's inability to select the test-only connection mode. They are **lettered rather than numbered**, sit outside that table's **104** numbered rows and are excluded from its **326**-case total, so input 14 does not count them and neither does input 23, whose seventeen are [04 §12.4](04-dotnet8-migration-strategy.md)'s and [06 §10.2](06-azure-hosting-recommendations.md)'s. **Two of the ten are already asserted by input 23's five operator-host tests** — O1's parser closure by the admitted-command-line assertion and O8's redaction by the credential-in-no-captured-output assertion — so **8** are new work and are the term this input contributes to the model | Row count | [05 §12.4.1](05-aspnet-core-migration-approach.md) |

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
- **Input 19 is not migration effort.** It is a repository-hygiene decision, **enumerated but
  intentionally unestimated** in
  [section 7.4](#74-repository-hygiene--enumerated-but-intentionally-unestimated), and gating nothing.
  An earlier form of this row pointed at section 7.3, which is the accessibility review and carries a
  band; the hygiene items are section 7.4's and carry none.
- **Inputs 14 and 23 are two different things, and the rule separating them is this document's to state,
  because this document is where the test workload is costed.**
  [05 §12.4](05-aspnet-core-migration-approach.md)'s numbered index is **not a parity suite**: **65 of
  its 104 rows are target-only**, so no rule of the form "every row proves a behaviour against the MVC 5
  baseline" holds of it.
  What the index is, is the **closed set of coverage rows the port is held to** — 104 numbered rows, plus
  the **10** operator rows `O1`–`O10` of
  [05 §12.4.1](05-aspnet-core-migration-approach.md) — and its membership rule belongs to that document,
  not to this one.
  **Input 23 counts exactly what falls outside the numbered index**, which is **twelve** items: the ten
  operator rows, which are 05's own rows but carry no numbered
  position because a console process has no MVC 5 baseline to be numbered against, and the **two** required
  manual deployed-browser items, `G-CSP-BROWSER` and [06 §10.4](06-azure-hosting-recommendations.md)'s
  **four-engine functional walk**. **The HTTP-observable content-security-policy work is
  inside the index**, as numbered rows **76–81** — six rows carrying **twenty-four** cases between them —
  so it is input 14's and is counted once, there. **Six rows, twenty-four cases and two manual gates are
  three different quantities and none of them is a count of another**; the two gates are the eleventh and
  twelfth items of this input and no case of rows 76–81 is.
  **Each item outside the index is estimated in the workstream that builds the thing it tests**, which is
  also the workstream whose gate demonstrates it:

  | Item outside the numbered index | Required by | Estimated and gated in |
  | --- | --- | --- |
  | **5** operator-host tests — a hostile working directory, every password-bearing argument form refused, the repair path in that host, the dispatcher's admitted command lines, and the credential arriving on its named environment channel without appearing in captured output. A sixth assertion in the same set, the lifetime spelling, is discharged by the Release solution build and costs nothing here | [04 §12.4](04-dotnet8-migration-strategy.md) | **[W12](#w12--administrator-provisioning-tool)**, which builds `tools/provision-admin` |
  | **11** HTTP CSP report-endpoint tests — both report transports, the two rejected media types, the unparseable and member-absent bodies, the size and batch bounds, the rate-limit partition, the anti-forgery exemption with its paired `GET`, and the redaction of query string, sample and referrer | [06 §10.2](06-azure-hosting-recommendations.md) | **[W7](#w7--the-aspnet-core-port)**, which owns the header set and the report endpoint registered in the composition root |
  | **1** deployed-browser CSP test — the twelfth of [06 §10.2](06-azure-hosting-recommendations.md)'s twelve, executed by the blocking gate [`G-CSP-BROWSER`](06-azure-hosting-recommendations.md#g-csp-browser) against a **deployed** environment on **every** browser engine of [06 §10.4](06-azure-hosting-recommendations.md)'s matrix, and run twice — once during the report-only period and again on any directive, binding, route or group change | [06 §10.4](06-azure-hosting-recommendations.md) | **[W10](#w10--hosting-provisioning-and-platform-configuration)**, as its exit condition 8, because it needs a provisioned deployment and a real browser rather than a test host |
  | **8** of the **10** published-console rows `O1`–`O10` (input 32) — every one of them starts the *published* executable with an argument vector and asserts on its exit code and captured output, so none has an MVC 5 baseline and none is a numbered row of input 14's matrix. `O1` and `O8` are excluded as already asserted by the five above | [05 §12.4.1](05-aspnet-core-migration-approach.md) | **[W12](#w12--administrator-provisioning-tool)**, which builds `tools/provision-admin` and already carries the process-level harness these rows run on |

  [03 §4.3](03-modernization-roadmap.md) states the same rule from the roadmap's side and places the three
  sets at the same three gates; no document folds any of them into W4, and none alters input 14's
  count by doing so. **The consequence for this model is that the test workload is input 14's parity rows
  plus seventeen further tests plus the eight new console rows of input 32 — `104 + 17 + 8 = 129`
  executable scenarios, not 104** — and the eight, like the seventeen, are
  real work that an earlier reading of input 14 left uncosted. **The parity term of that sum is input 14's
  and is re-read from [05 §12.4](05-aspnet-core-migration-approach.md) at each reconciliation rather than
  carried as a constant**, because it has moved six times and has now taken **seven** values — 103, 108,
  116, 115, 75, 102 and 104: `103 + 17 = 120`, `108 + 17 = 125`,
  `116 + 17 = 133`, `115 + 17 = 132`, `75 + 17 = 92` and `102 + 17 = 119` are all superseded forms of this sentence, and the **17** is the term that has not
  changed — five operator-host tests plus twelve CSP tests. **The third term is new rather than moved**:
  input 32's console rows sat outside every count until this reconciliation, and **8** of its ten enter the
  sum because the other two are already inside the seventeen. **The most recent move is why the separation
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
- **Inputs 24 to 30 exist so that as many rows of [section 5.1](#51-summary-table) as a repository count
  can reach carry a numbered, method-named input that sizes their own work. Six of the seven close gaps on
  seven rows** — the counts differ because **input 26 closes two** — **the seventh, input 25, sizes half of
  one of those seven rather than closing a row of its own, and one row, W13, is closed by no numbered input
  at all.** Both exceptions are stated below rather than absorbed, because a census that claims to cover
  every row and does not is worse than one that names its own edge.
  - **Six rows named no count at all** — **W2, W10, W13, W14, W15 and W16**. Each stated its basis in
    prose and cited the owning deliverable, but none named a quantity, so none could be traced to one.
    **Five of the six are now closed**: W2 by **input 24**, whose 26 `HintPath` references are the restore
    precondition its title names; W10 by input 26; W14 by input 29; W15 by input 27; W16 by input 28.
  - **W13 is the sixth, and it stays open on purpose.** Cutover is the execution of a runbook, and no count
    of anything in this repository is a proxy for that: its size comes from
    [06 §6.10](06-azure-hosting-recommendations.md)'s five-verb data-movement order and
    [06 §11](06-azure-hosting-recommendations.md)'s seventeen stages, which are owner step counts rather
    than repository censuses. Assigning it input 25 — zero observability artifacts — would have made the
    census read as complete while sizing a cutover by the absence of logging, which is not a relationship
    that exists. Its [§5.1](#51-summary-table) cell says so in as many words.
  - **Input 25 closes no row either, and it is the input that reveals why the two counts are separate.**
    Its zero observability artifacts size the **net-new observability half** of W10's basis, which is why
    W10's basis cites [08 §7.1](08-technical-debt-register.md); the row itself is closed by input 26. An
    input can size a component of a row without being the input that closes it, and conflating the two is
    what produced the earlier claim that seven inputs closed eight rows.
  - **One row named an input that sized only part of it** — **W12**. It cited input 23, but input 23
    counts the **10 operator rows** that drive the provisioning executable, not the executable. The
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
  required properties, its four converged operations, and
  `PROV-6001`'s **18** closed outcome values, **15** of them exercisable. **Not one of the seven is a
  line count**, so none can be confused with either method in
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
  removing a deployment from it; W5's basis derives the offset, and the band in
  [§5.1](#51-summary-table) is re-derived from this census rather than carried across from an earlier
  reading.

  **Six of the seven change no band; one does, and it is named rather than absorbed.** Inputs 24 to 29
  each document the basis of a figure that was already judged, so they leave the total, the concurrency
  sets and the critical path numerically untouched. **Input 30 is the exception.** Its first statement
  undercounted `PROV-6001`'s closed vocabulary as eight values against
  [09 §6.8.1](09-security-assessment.md)'s **18**, and correcting it exposed one value —
  `Failed_ArgumentRejected` — that carries genuinely new scope rather than a renaming of work already
  priced. [W12](#w12--administrator-provisioning-tool)'s band moved from 3 / 5 / 9.5 to 3 / 5.5 / 10.5 for
  that reason, with the derivation and the explicit statement of what does *not* contribute stated in its
  basis. **That figure is the state of the row at the end of the input-30 pass and not its current band.**
  Three later corrections moved it again — [§4.2.1](#421-the-rounding-rule-stated-once) re-derived input 23's
  operator-host increment upward, [03 §5 W12](03-modernization-roadmap.md) added exit condition 7's
  deployed census and the fourth separately gated invocation, and **input 32** added the eight
  published-console rows of [05 §12.4.1](05-aspnet-core-migration-approach.md) that are new work — so the
  row stands at
  **5 / 9.5 / 16** in [§5.1](#51-summary-table), through the intermediate 3.5 / 7.5 / 12.5, and
  [§6.1.1](#611-the-walk-from-the-previously-published-total) records the whole of that move as one line
  of the walk.

  **One dimension of input 30 has since been withdrawn, and it is recorded here rather than deleted,
  because a removal from an input is as consequential to a reader checking a band as an addition to it.**
  That row also counted **a revoke mode of the provisioning command — two operations across three
  branches** — attributed to a *property 3a* of
  [05 §10.2](05-aspnet-core-migration-approach.md). **That property does not exist and that mode does not
  exist.** The section defines no property 3a; its verb table is closed with no revoke verb among the
  verbs; the `admin` verb's accepted switch set is closed at four, none of which asks for a revocation;
  and its property 3 states the command **never deletes or demotes** anything. The dimension therefore
  sized a capability no deliverable specifies, and it is withdrawn from this input.
  **It moves no band, and that is established from the record rather than asserted**:
  [W12](#w12--administrator-provisioning-tool) priced the mode explicitly as an addition *inside* an
  unchanged band, none of that row's five published components carries a revoke term, and
  [§6.1.1](#611-the-walk-from-the-previously-published-total)'s W12 row names four movers, none of them
  this one. **What the withdrawal leaves open is a gap rather than an estimate**, and it belongs to
  [09 §6.8.1.1](09-security-assessment.md), which owns the privilege-withdrawal event class, keeps its
  identifier reserved with no producer, and records that no workstream of
  [03 §5](03-modernization-roadmap.md) carries it — deliberately, because a workstream needs a specified
  artifact and there is none. This document therefore carries no band for it, on the same principle
  [§6.3](#63-what-is-deliberately-not-in-the-total) applies to every path the assessment records but does
  not assume.

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
| **A2** | A **build host meeting the prescribed toolchain** remains available, as the one used for the outcome recorded in [10 §3.2](10-build-and-deployment-requirements.md) and [10 §5.4](10-build-and-deployment-requirements.md) was | W2 grows from a reproduction into a provisioning exercise |
| **A3** | The team has **read access to a production-representative dataset** for rehearsing both data migrations, with PII handling approved | W8 and W9 high bands become the expected case; reconciliation cannot be rehearsed and moves risk into the cutover window |
| **A4** | The **manual visual review is performed by one reviewer** against the captured baseline, with a second signature only for approval | A review panel multiplies [section 7.1](#71-the-manual-visual-review) by the number of reviewers |
| **A5** | **Approvers are identified and available.** [05 §11.5](05-aspnet-core-migration-approach.md) names approval *roles*; A5 assumes named people hold them | W1 and [section 7.2](#72-the-approved-delta-sign-offs) grow, and every dependent workstream inherits the delay |
| **A6** | The **single-cutover approach stands** as decided in [05 §11](05-aspnet-core-migration-approach.md) | The conditional incremental path of [05 §11.6](05-aspnet-core-migration-approach.md) is a materially different and larger shape: two hosts, two pipelines and an adapter surface on both sides. **This model does not cost it** |
| **A7** | **Scope is the migration source only.** Neither reference edition is ported | Sizing a second edition is new work, not a multiplier — see [R10](#r10--scoping-by-analogy-across-editions) |
| **A8** | The **target platform's managed services are used as recommended** in [06](06-azure-hosting-recommendations.md), rather than self-hosted equivalents | W10 grows substantially; a self-managed database and key store are not costed here |
| **A9** | **No new functional requirement** is introduced. The behaviour baseline is [05 §11.5](05-aspnet-core-migration-approach.md)'s: preserved outcomes plus every approved delta of that register, two of them conditional | Any new feature is outside this model entirely. **A delta added to that register** would mean the authoritative behaviour set had moved again, which is a re-estimation of [W1](#w1--approval-of-this-assessment), [§7.2](#72-the-approved-delta-sign-offs) and the coverage rows it implies rather than a variance inside their bands |
| **A10** | The **password hashes in the shipped credential store validate** under the target framework's default hasher | W8's high band becomes the expected case and [R5](#r5--identity-migration-rollback) escalates from a rollback risk to a redesign |
| **A11** | The **interim hosting branch is not taken.** It is a conditional branch outside the sixteen workstreams [03 §4](03-modernization-roadmap.md), and [06 §5.8](06-azure-hosting-recommendations.md) records that its production data move currently has a shape rather than a contract, so the branch is not executable until that contract is authored and approved | **Nothing in this model covers it.** If the branch is taken, two units enter the estimate that are absent from it today: **authoring the legacy-schema-preserving migration contract** 06 §5.8 enumerates, and **executing it**. Both must be estimated at that point on the same basis as every row here, and [R17](#r17--the-interim-hosting-paths-stored-credential-and-a-time-box-with-no-box)'s enforcement obligation begins with the grant rather than with the move |

### 4.4 Confidence, and its reason

**Overall confidence: medium.** Stated with its reason, because a bare adjective is not a confidence
statement:

> **Medium, because the two halves of the work have opposite estimation properties.** The mechanical
> port is **well bounded by enumeration**: the migration source is 2,097 non-blank lines (sizing
> metric), its decomposition is measured to the file [08 §4.2](08-technical-debt-register.md), all 23
> blockers are enumerated with a named resolution each [12 §2.3](12-migration-blockers.md), and every
> construct the port must touch is named in a document rather than inferred. Against that, **two inputs
> this model depends on are not settled, and neither can be narrowed by reading source.** The
> first is the build: [10 §1.2](10-build-and-deployment-requirements.md) carries the migration source's
> build assessment as *blocked pending a Windows verification run*, and what is established on a clean
> checkout is a precondition failure rather than a compile result. A prescribed-toolchain restore and
> rebuild was later observed once on a Windows host, exiting `0` in both configurations with zero
> warnings and zero errors [10 §5.4](10-build-and-deployment-requirements.md), but
> [10 §3.2](10-build-and-deployment-requirements.md) records that run as **supplementary observation
> which does not discharge the gate** — it is not a build from a clean checkout and it is not a
> reproduction this model may rely on — so W6's and W7's diagnostic volume is still enumerated from
> [12 §2.3](12-migration-blockers.md) rather than taken from a compiler. The second is the schema:
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
| **Medium** | W1, W2, W7, W10, W11, W12, W13, and all three non-code tasks — [§7.1](#71-the-manual-visual-review), [§7.2](#72-the-approved-delta-sign-offs) and [§7.3](#73-the-manual-accessibility-review) | Scope is known and enumerated; the variance is execution and, for W1 and W13, other people's decisions. W2 is medium rather than high for a different reason: its *tasks* are enumerated but its *outcome* is not settled — the one prescribed-toolchain rebuild on record is supplementary observation rather than a discharged gate ([10 §3.2](10-build-and-deployment-requirements.md)) — and a failing verification run costs more than a passing one |
| **Low** | W3, W4, W8, W9, W16 | Each depends on a fact not yet established: the extracted schema for three of them, and for W16 how many constituencies must convene and whether the organization already has retention policy to inherit — neither knowable from the repository, which is the same property that makes W1 wide; and for W4 the behaviour of a system that has never been characterized by a test — plus, across its four re-derivations, seven components with no precedent in this repository to calibrate against: a two-platform execution whose handoff record is refused on any mismatch, a private legacy deployment lifecycle whose readiness poll must also wait out an `async void` provisioning path, a keyed pseudonym scheme with key custody and destruction, a redactor that must itself be tested, an abstract-plus-sealed contract topology that has to enrol tests in two assemblies through per-assembly collection definitions, a runtime-neutral store observer with two implementations beside a separate OWNER-backed setup surface, and a three-identity SQL bootstrap with a durable ownership registry ([05 §12.2, §12.3, §12.6, §12.7, §12.8, §12.9, §12.10](05-aspnet-core-migration-approach.md)) |
| **High** (conditional) | W15·C | A platform setting plus one defined verification, both owned by [06 §8.3, §8.3.1](06-azure-hosting-recommendations.md); it is conditional rather than uncertain |

**The low-confidence rows carry 93.5 of the 311.5 expected IED** — see
[section 6](#6-totals-and-where-the-effort-actually-lives) — which is the single most useful thing an
approver can know about this estimate. W3 is cheap and retires most of that uncertainty, which is why
[03](03-modernization-roadmap.md) places it before the port and why
[section 8.3](#83-the-critical-path-and-what-to-do-first-if-the-goal-is-to-narrow-the-estimate) recommends it as the first
substantive action.

**No existing row's confidence rating changed across any of the five re-derivations, and that is a claim
worth defending rather than an omission.** W4 stays **Low**, W6 stays **High**, W7, W11, W12 and the
manual visual review stay **Medium**, and overall
confidence stays
**Medium** — because the two facts the overall statement rests on, the unverified build and the
unextracted schema, are untouched by a coverage figure being superseded. What the supersessions changed is
**scope, which was enumerated more tightly rather than less**: a case-level matrix with named assertion
fields is a firmer basis than a surface list, and a fixture whose lifecycle, dataset, principals and
handoff contract are specified item by item is a firmer basis than one described as "a fixture", so the
re-derived bands are better founded than the ones
they replace even though they are larger. **Three rows entered the total in the round before this one
without any rating changing**: **W16** at **Low** and the **manual accessibility review** at **Medium** are rated here for
the first time, and the **approved-delta sign-offs** carry the same **Medium** as a row of their own that
they carried while a previous revision of [§5.1](#51-summary-table) held them inside W1's band.

**The low-confidence share moved again, and it is stated rather than smoothed.** It is **93.5 of
311.5** — `4 + 67.5 + 8 + 8 + 6` for W3, W4, W8, W9 and W16 — about **30.0 percent** of the model, against
92.5 of 308 (also about 30 percent), then 80.5 of 239.5 (about 34 percent), then 57 of 195.5
(about 29 percent), then 39 of 155 and
33 of 127.5. **It held at 30.0 percent this round**, and the reason is arithmetic rather than
a change of view: the Low rows grew by **1** expected IED — W4 absorbed the whole of it, its shared-contract
case component following the both-fixtures count — while the total grew by **3.5**, `1 + 3` on the two
case rows less the **0.5** the sign-off row gives back, so `92.5 + 1 = 93.5`
against `308 + 3.5 = 311.5`, and the two growths are near enough in proportion to leave the share where it
stood. **It fell by four points in the round before**, when the Low rows grew by **12** expected IED —
W4 absorbed 6 of the 41.5 the six re-derivations added, and W16's 6 entered the total with the membership
correction — while the total grew by **68.5**, so `80.5 + 12 = 92.5` against `239.5 + 68.5 = 308`. The
remedy is unchanged and is stated in the next paragraph: W3 retires the schema uncertainty behind three of
the five, W16's is retired by an approval rather than by engineering, and only W4's is retired by
executing W4 itself.

---

## 5. Effort by workstream

**The decomposition below is [03 §5](03-modernization-roadmap.md)'s, not this document's.** All sixteen
workstreams appear, in 03's order, under 03's names. No workstream is added, renamed, merged or split,
because a second decomposition would leave the roadmap and the estimate unable to reference each other.
What this document adds to each is a band, a confidence and the inputs the band derives from.

> **Every band below is estimated against [03 §4.2](03-modernization-roadmap.md)'s canonical gate graph and
> [03 §5](03-modernization-roadmap.md)'s final workstream set, and five rows take their scope from that graph
> rather than from what their names suggest.** Each of the five is 03's statement, cited here and not
> re-argued:
>
> - **W6 is a skeleton, not a converted legacy build.** [03 §5 W6](03-modernization-roadmap.md) carries no legacy-behaviour claim and does not run the W4 suite, because the unchanged `System.Web` source cannot compile on the target framework at all. The integration cost that gate might be assumed to carry sits in W7, where it is a named sub-row.
> - **W6 does not precede W4's second gate, and the project graph's two states are why.** [03 §4.2](03-modernization-roadmap.md) carries **`W4a → W6`** and no edge back: gate 4a is the build-governance bootstrap W6 inherits, and the test project gate 4b creates declares **no project reference** in [04 §12.4](04-dotnet8-migration-strategy.md)'s pre-port state, so it compiles and restores in locked mode against its own direct pins before the conversion has happened. The single reference it ends with is added inside **W7**. That fixes where the work sits, not what it contains — and it is why gate 4b's entry list below names three conditions rather than four.
> - **W8 and W9 end at a rehearsal against a copy**; the production extraction, load and reconciliation are W13's. So W8 and W9 carry a rehearsal and a runbook, and W13 carries the execution.
> - **W11 is a prerequisite of W10's DDL-applying gate**, so it sits earlier in the sequence than its number suggests.
> - **W16 — personal-data governance — is a workstream in its own right**, in two stages, and both are sized here.
>
> **One further dependency correction changes a row's position without changing its size — and it runs in
> the opposite direction to the one an earlier reading of this block asserted.** The suite W4 authors is a
> **project inside** [04 §12](04-dotnet8-migration-strategy.md)'s target project graph, and the governance
> that project is restored under — `global.json` and the root `NuGet.config` — is **gate 4a's own
> output**, created there because 4a's locked-mode restore was impossible without it. So
> [03 §4.2.1](03-modernization-roadmap.md) carries **`W4·4a → W6`** ([03 §6.1](03-modernization-roadmap.md)
> row 19), and the conversion **extends** those two files rather than creating them.
> **The edge runs only that way: there is no edge from W6 to either half of W4**, and 03 §6.3 names
> `W6 → W4·4a` and `W6 → W4·4b` as exactly the two edges that would close a cycle, states that neither
> exists, and gives the reason this block previously had backwards — **the contracts project W4 creates
> references no application, so nothing in W4 waits on the converted project graph.** Both bands are
> unchanged, because the edge fixes which of the two opens first rather than what either contains. What
> the edge settles is the sequence, and [§8.2](#82-concurrency-permitted-by-the-graph) and
> [§8.3](#83-the-critical-path-and-what-to-do-first-if-the-goal-is-to-narrow-the-estimate) are derived
> against that inventory: they place **W6 off the critical path**, at its **4.5** expected IED, in §8.3's
> off-path itemization.
>
> **Two further rows moved because an input was corrected rather than because the graph changed.**
> [05 §12.4](05-aspnet-core-migration-approach.md)'s required coverage was read at that round as **input
> 14's 75 rows**, not the 27 an earlier reading of input 14 recorded, so **W4** was rebanded — and that
> matrix has grown since, so input 14 now stands at **104 rows, 326 cases and 478 fixture executions**,
> the count [§6.1.1](#611-the-walk-from-the-previously-published-total) carries as W4's latest movement;
> and
> [05 §11.5](05-aspnet-core-migration-approach.md)'s register was read in full, rather than as the 18 an
> earlier reading of input 17 recorded or the 14 before that, so the
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
| **W1** — Approval of this assessment | 3.5 | 7 | 13.5 | Medium | 17, 32 |
| **W2** — MVC 5 build reproduction and the restore precondition | 2 | 4 | 9 | Medium | 24 — the input [§4.1](#41-the-estimation-basis-every-input-with-its-method)'s fifth usage note names as closing this row's gap: the **46** `<Reference>` elements of which **26** carry a `HintPath` under `..\packages\`, which is the restore precondition this row's title names and the thing a reproduction has to satisfy before it compiles ([02 §4.2](02-dependency-inventory.md)); scope from [03 §5](03-modernization-roadmap.md); exit criterion from [10 §3.2](10-build-and-deployment-requirements.md) |
| **W3** — Authoritative schema extraction | 2 | 4 | 8 | **Low** | 1, 12 |
| **W4** — Build-governance bootstrap, pre-port behavioural baseline and test suite | 38.5 | 67.5 | 114 | **Low** | 14, 15, 17, 21, 22, 23, 24, 25, 26, 27 |
| **W5** — Repository-wide path-casing audit | 1 | 2 | 4 | High | 31 (inputs 7, 8 and 11 are migration-source only and are **not** this row's basis — [§5.2 W5](#w5--repository-wide-path-casing-audit)) |
| **W6** — Project-format conversion and dependency transition | 2.5 | 4.5 | 8.5 | High | 12, 13, 27 |
| **W7** — The ASP.NET Core port | 69.5 | 122.5 | 208.5 | Medium | 2, 3, 4, 6, 8, 9, 10, 11, 12, 14, 17, 18, 20, 23, 24, 25, 29 — 18 being the five unvalidated POSTs its global anti-forgery entry answers |
| **W8** — Identity migration tooling, rehearsed against a copy | 4 | 8 | 16 | **Low** | 12 |
| **W9** — Domain data migration tooling, rehearsed against a copy | 4 | 8 | 15 | **Low** | 1, 12 |
| **W10** — Hosting provisioning and platform configuration | 5 | 9 | 16 | Medium | 26 — the input [§4.1](#41-the-estimation-basis-every-input-with-its-method)'s fifth usage note names as closing this row's gap, its zero publish and deployment-automation artifacts sizing the provisioning as net-new; scope from [06 §6](06-azure-hosting-recommendations.md) |
| **W11** — CI provider selection, then pipeline authoring | 11 | 19.5 | 35.5 | Medium | 15, 21, 26, 28, 29 |
| **W12** — Administrator provisioning tool | 5 | 9.5 | 16 | Medium | 30 (the tool's own required behaviour), 23 and 32 (its added tests) — the three are disjoint, [§5.2 W12](#w12--administrator-provisioning-tool); scope from [05 §10.2](05-aspnet-core-migration-approach.md); row 75's acceptance from [05 §12.4](05-aspnet-core-migration-approach.md) |
| **W13** — Cutover | 2 | 4 | 8 | Medium | **None, and that is stated rather than papered over** — this is the one row of this table sized by an owner's ordered step count instead of by a numbered input, because nothing in the repository is a proxy for executing a runbook: the count is [06 §6.10](06-azure-hosting-recommendations.md)'s five-verb data-movement order and [06 §11](06-azure-hosting-recommendations.md)'s seventeen stages, and §4.1's fifth usage note records it as the exception (scope from [06 §11](06-azure-hosting-recommendations.md)) |
| **W14** — Documentation revision | 1 | 2 | 3 | High | 29 — the three tracked documents its own basis enumerates ([§5.2 W14](#w14--documentation-revision)) |
| **W15** — Affinity retirement | 0.5 | 1 | 2 | High | 27 — named in this row's own basis ([§5.2 W15](#w15--affinity-retirement)) |
| **W16** — Personal-data governance | 3 | 6 | 12 | **Low** | 22, 28, 21 — requirement from [09 §3.11, §6.8](09-security-assessment.md) |
| *Non-code:* manual visual review ([§7.1](#71-the-manual-visual-review)) | 6.5 | 12.5 | 22.5 | Medium | 16 |
| *Non-code:* manual accessibility review ([§7.3](#73-the-manual-accessibility-review)) | 2.5 | 4.5 | 8 | Medium | 6, 16 |
| *Non-code:* approved-delta sign-offs ([§7.2](#72-the-approved-delta-sign-offs)) | 8 | 16 | 26 | Medium | 17 — its approved-delta dimension, every row of the register in [05 §11.5](05-aspnet-core-migration-approach.md), plus [05 §11.7](05-aspnet-core-migration-approach.md)'s **15** approval-owned additions |
| *Conditional, **not** in the total:* pre-admission affinity retirement (secondary hosting path) | 1 | 2 | 4 | High | — (scope from [06 §8.3, §8.3.1](06-azure-hosting-recommendations.md)) |
| **Total** — the sixteen workstreams plus the three non-code tasks | **171.5** | **311.5** | **545.5** | Medium overall | |

**Three conventions make this column addable, and all three are stated in the table itself rather than in a
footnote.**

- **Every row is added exactly once, and the single row that is not added says so in its own label.** The
  conditional pre-admission affinity retirement is that row, because it belongs to the secondary hosting
  path rather than to this plan ([§6.3](#63-what-is-deliberately-not-in-the-total)). Nothing else in the
  column is a component of anything else in it, so there is no figure a reader has to know to skip.
- **The three non-code rows are added, because none of them sits inside a workstream's band.** Each is a
  non-code task in [03](03-modernization-roadmap.md)'s terms, sized only here, and each sits on a
  workstream's **exit gate** rather than inside its band —
  [section 7](#7-work-that-is-not-code) states where each attaches in the sequence. The approved-delta
  sign-offs are the one a previous revision of this table got wrong: it carried them parenthesised as
  effort *inside* W1 at 2 / 4 / 8. [§7.2](#72-the-approved-delta-sign-offs) owns that band, states that
  it is **not** inside W1's, and partitions input 17 between the two — W1's 3.5 / 7 / 13.5 buys the six
  constituency briefings, the **four** escalated risk decisions that are not deltas, and the gate record,
  priced by [W1](#w1--approval-of-this-assessment) as `11 acts × 0.3 / 0.6 / 1.2` rounded up; this
  row's 8 / 16 / 26 buys the recorded decisions — every row of the approved-delta register in
  [05 §11.5](05-aspnet-core-migration-approach.md) and
  [05 §11.7](05-aspnet-core-migration-approach.md)'s 15 approval-owned additions. `7 + 16 = 23` expected
  IED is the whole approval activity, and neither figure is inside the other.

Sum of the sixteen workstream rows: **154.5 / 278.5 / 489**. Plus the manual visual review's
6.5 / 12.5 / 22.5, the manual accessibility review's 2.5 / 4.5 / 8 and the approved-delta sign-offs'
8 / 16 / 26: **171.5 / 311.5 / 545.5**.
The sixteen-row addition, at the expected band:
7 + 4 + 4 + 67.5 + 2 + 4.5 + 122.5 + 8 + 8 + 9 + 19.5 + 9.5 + 4 + 2 + 1 + 6 = **278.5**; at the low band
3.5 + 2 + 2 + 38.5 + 1 + 2.5 + 69.5 + 4 + 4 + 5 + 11 + 5 + 2 + 1 + 0.5 + 3 = **154.5**; at the high band
13.5 + 9 + 8 + 114 + 4 + 8.5 + 208.5 + 16 + 15 + 16 + 35.5 + 16 + 8 + 3 + 2 + 12 = **489**.
The three non-code rows on top of that, at the expected band: `12.5 + 4.5 + 16 = 33`, so
`278.5 + 33` = **311.5**; at the low band `6.5 + 2.5 + 8 = 17`, so `154.5 + 17` = **171.5**; at the
high band `22.5 + 8 + 26 = 56.5`, so `489 + 56.5` = **545.5**.

**Two rows are re-derived again this round, and the round before it re-derived five a fourth time while
two more moved for reasons of their own**, because
[05 §11.5, §12.3–§12.11](05-aspnet-core-migration-approach.md),
[04 §6, §7.1, §7.6, §7.7 and §12.2–§12.4](04-dotnet8-migration-strategy.md),
[06 §6.4, §9.5 and §12.1](06-azure-hosting-recommendations.md) and
[03 §4.2 and §5](03-modernization-roadmap.md) closed successive rounds of findings that each enlarged the
scope the previous bands were derived against. **This round W4 and W7 move**, both following the case
counts [05 §12.4](05-aspnet-core-migration-approach.md)'s matrix now publishes and each derived in its own
record, **and the approved-delta sign-offs fall half a day in every column** because
[05 §11.5](05-aspnet-core-migration-approach.md) retired a second identifier, which
[§7.2](#72-the-approved-delta-sign-offs) re-derives in full and which the walk below carries inside part
one rather than as a fourth part, since that part values each membership row at the band its owner
publishes now. **In the round before, W4, W7, W11, W12 and the manual visual review moved**,
and **W1's band moved for the first time** — its act census gained
[R18](#r18--the-browser-executed-half-of-the-scripted-cart-flow-one-pinned-engine-and-an-open-decision-on-the-other-two)'s
recorded outcome — and the **approved-delta sign-offs** moved on a **count** rather than a rate, because
[05 §11.5](05-aspnet-core-migration-approach.md)'s register was read at its **present size** while the
marginal rate stayed where it was, which [§7.2](#72-the-approved-delta-sign-offs) derives in full. **No
other row's band moves**;
[section 5.2](#52-basis-of-estimate-per-workstream) and [section 7](#7-work-that-is-not-code) show each
re-derivation with its components and its addition, and each of those sections **owns** the band it
derives — this table carries what they publish rather than a figure of its own.

**The movement has three parts, and separating them is what makes it check.** The previously published
total was **133.5 / 239.5 / 424**, and that figure was the sum of **sixteen** rows: W1 to W15, at
131 / 235 / 416.5, plus the manual visual review at the 2.5 / 4.5 / 7.5 it then carried. Three rows this
table already listed **never reached it** — a total that omits a row it displays is the defect
[§6.1.1](#611-the-walk-from-the-previously-published-total) records in full — so the base is corrected
before anything is re-banded.

- **Part one, the membership correction: `+13.5 / +26.5 / +46`.** **W16** at 3 / 6 / 12, whose own record
  states its band is unchanged; the **manual accessibility review** at 2.5 / 4.5 / 8, whose record states
  the same; and the **approved-delta sign-offs** at 8 / 16 / 26, which the previous total treated as
  effort inside W1 rather than as the separate row [§7.2](#72-the-approved-delta-sign-offs) states it is.
  Each of the three enters at the band its own owner publishes **now**, which is why this part falls half a
  day in every column as the sign-off row re-prices.
  At the low band `3 + 2.5 + 8 = 13.5`, at the expected `6 + 4.5 + 16 = 26.5`, at the high
  `12 + 8 + 26 = 46`, so the corrected base is `133.5 + 13.5` = **147**, `239.5 + 26.5` = **266** and
  `424 + 46` = **470**.
- **Part two, the six re-derived bands: `+22.5 / +41.5 / +70`.** **W1** moves from 3 / 6 / 12 to
  **3.5 / 7 / 13.5** — **+0.5 / +1 / +1.5**, one act joining its census; **W4** moves from
  35.5 / 60.5 / 102.5 to **38 / 66.5 / 112.5** — **+2.5 / +6 / +10**; **W7** from 57.5 / 102 / 175.5 to
  **68 / 119.5 / 204.5** — **+10.5 / +17.5 / +29**; **W11** from 9 / 15.5 / 29 to **11 / 19.5 / 35.5** —
  **+2 / +4 / +6.5**; **W12** from the 2 / 4.5 / 8 the previous total carried to **5 / 9.5 / 16** —
  **+3 / +5 / +8**, derived in five components in [W12's own record](#w12--administrator-provisioning-tool),
  which also records the two intermediate figures it passed through; and the **manual visual review**
  from 2.5 / 4.5 / 7.5 to **6.5 / 12.5 / 22.5** — **+4 / +8 / +15**, derived as a capture half and a
  review half in [§7.1](#71-the-manual-visual-review). At the low band
  `0.5 + 2.5 + 10.5 + 2 + 3 + 4 = 22.5`, at the expected `1 + 6 + 17.5 + 4 + 5 + 8 = 41.5`, at the high
  `1.5 + 10 + 29 + 6.5 + 8 + 15 = 70`.
- **Part three, the two case-count re-derivations of this round: `+2 / +4 / +5.5`.** **W4** moves from
  38 / 66.5 / 112.5 to **38.5 / 67.5 / 114** — **+0.5 / +1 / +1.5**, its shared-contract case component
  following the both-fixtures count 147 → **152**; and **W7** from 68 / 119.5 / 204.5 to
  **69.5 / 122.5 / 208.5** — **+1.5 / +3 / +4**, its target-facing case row following 147 + 160 →
  **152 + 174** [05 §12.4](05-aspnet-core-migration-approach.md). At the low band `0.5 + 1.5 = 2`, at the
  expected `1 + 3 = 4`, at the high `1.5 + 4 = 5.5`. **Nothing else in either workstream moves, and the
  only other row that moves this round is the sign-off row part one already carries at its re-derived
  band**, which is why this part is two terms rather than seven.

Adding part two to the corrected base gives `147 + 22.5` = **169.5**,
`266 + 41.5` = **307.5** and `470 + 70` = **540** — half a day below the **170 / 308 / 540.5** the previous
round published in every column, because part one carries the sign-off row at the
[8 / 16 / 26](#72-the-approved-delta-sign-offs) it re-derives to rather than the 8.5 / 16.5 / 26.5 that
round carried. Part three then gives the total above,
`169.5 + 2` = **171.5**, `307.5 + 4` = **311.5** and `540 + 5.5` = **545.5**. Read as workstreams alone, the
fifteen-workstream
131 / 235 / 416.5 gains W16's 3 / 6 / 12 — `131 + 3 = 134`, `235 + 6 = 241`, `416.5 + 12 = 428.5` — and
then the five workstream re-derivations' `+18.5 / +33.5 / +55`, being part two less the review's share:
`134 + 18.5` = **152.5**, `241 + 33.5` = **274.5** and `428.5 + 55` = **483.5**; part three is entirely
workstream movement, so `152.5 + 2` = **154.5**, `274.5 + 4` = **278.5** and `483.5 + 5.5` = **489**, the
sixteen-workstream sum printed above.

**Two numbered inputs moved this round, and each moves exactly the rows that read it.** The published
suite moved from **102 rows and 307
cases** to **104 rows and 326 cases**, and from 454 fixture executions to **478** — one further mixed row
and one further target-only row, with the rest of the growth falling inside rows that already existed — so
both-fixture cases go 147 → **152**, target-only 160 → **174**, and
`152 × 2 + 174 = 478` (input 14) [05 §12.4](05-aspnet-core-migration-approach.md), which is why part three
is two terms and why only the two rows that price cases move in it. And **input 17's delta register retired
a second identifier**, which moves the one row that prices its decisions and nothing else, as
[§7.2](#72-the-approved-delta-sign-offs) derives. **No other numbered input changed.**

**What arrived in the round before, item by item**, all of it specified in the siblings and cited rather
than restated:

- the published suite moved from **75 rows and 239 cases** to **102 rows and 307 cases**, and from 383
  fixture executions to **454** — first three further cases inside two existing rows, all target-only,
  and then the **twenty-seven** rows the owner's table then carried as ids 76–102: **2** of them
  both-fixture, adding **3** cases that execute twice, and **25** target-only, adding **62** that execute
  once, so both-fixture cases went 144 → **147** and target-only 95 → 98 → **160** (input 14);
- the test topology became **declarable**: every base, concrete and fixture type `public`, a
  `[CollectionDefinition]` class **per surface group per assembly** with a `const`-backed name, and the
  runner and build-asset packages **declared directly in every runnable test project**
  ([05 §12.7](05-aspnet-core-migration-approach.md) and [04 §7.2](04-dotnet8-migration-strategy.md));
- an **`IStoreSetup`** write API — eight operations across eleven members, OWNER-backed and deliberately
  off the injected context — plus **five** additional `IStoreObserver` read projections
  ([05 §12.6](05-aspnet-core-migration-approach.md));
- the fixture inputs became **twelve committed files**, nine of them `ModelOverrides/` divergence seams
  ([05 §12.3](05-aspnet-core-migration-approach.md));
- the baseline record gained a **`coverage` completeness object** and **one publication policy**
  ([05 §12.10](05-aspnet-core-migration-approach.md));
- the in-process host's instrumentation became an **exhaustive three-line allow-list** covering **both**
  `DbContext` types, fault injection became **`DENY`-based**, and the browser case acquired its own
  **real-Kestrel loopback host** ([05 §12.6, §12.8, §12.11](05-aspnet-core-migration-approach.md));
- the migration artifact gained the **`--store` grammar** and the four **`dbo.__DataMigration*` run tables** written in
  each group's own transaction ([05 §5.6](05-aspnet-core-migration-approach.md));
- the deployed gate gained a **published telemetry join protocol** and an **executor-and-stage mapping**
  for its thirteen checks ([06 §12.1](06-azure-hosting-recommendations.md));
- the provisioning tool's audit record gained a **destination** — one operator-tool pin, a federated
  credential path and a retention export — and **row 75 became its exit criterion**
  ([06 §9.5.1](06-azure-hosting-recommendations.md), with the pin in
  [04 §7.2](04-dotnet8-migration-strategy.md) and row 75 in
  [05 §12.4](05-aspnet-core-migration-approach.md));
- and the pin table reached **17** rows: ten on the 8.0.30 band, six test-tooling, one operator-tool
  ([04 §7.2](04-dotnet8-migration-strategy.md)).

**The three earlier re-derivations remain on the record**, because the movement is only checkable against
what it moved from. The round before that one moved six rows, for these reasons:

- the published suite moved from 72 rows and 183 cases to **75 rows and 239 cases**, and from 281
  fixture executions to **383** (input 14);
- the approved-delta ledger moved from 19 entries to **22** (input 17);
- the deployed verification gate's attempt budget acquired **literal evidence bounds** in place of a
  stated intent, which converts design work into authoring work
  ([06 §12.1](06-azure-hosting-recommendations.md));
- the contract topology became **six abstract classes with sealed per-assembly bindings**, a
  runtime-neutral **`IStoreObserver`** with two implementations and an explicit DTO surface, and **six**
  independent test-tooling pins in place of four
  ([05 §12.2, §12.6](05-aspnet-core-migration-approach.md) and
  [04 §7.2](04-dotnet8-migration-strategy.md));
- the legacy lifecycle gained **startup-quiescence polling** and the fixture dataset grew to published
  per-entity counts with **seven** post-load invariants
  ([05 §12.3](05-aspnet-core-migration-approach.md));
- provenance resolved into **three** distinct things — an **11-value gating `baselineSource`** namespace, a
  **7-fact `targetRun`** namespace recorded and never compared, and a **committed `baseline-reference.json`**
  of three legacy values — with an artifact transfer, a digest sidecar and **one publication policy** that
  publishes a `baseline-capture-diagnostic` on a failed or incomplete capture and a `baseline-record` only
  on a complete all-passed run ([05 §12.10](05-aspnet-core-migration-approach.md)); diagnostics gained a
  sanitized exception projection, a **26-code** `operation` set
  and a corrected keyed-alias scheme ([05 §12.9](05-aspnet-core-migration-approach.md)); and
  destructive-operation control gained a **third SQL
  identity**, a durable ownership registry with a sidecar, identifier re-resolution before every `DROP`,
  and a standalone sweep class with an always-run cleanup job
  ([05 §12.8](05-aspnet-core-migration-approach.md));
- **the browser-executed half of the scripted cart flow is no longer excluded — though its coverage
  decision is still open.** An earlier round carried it as an open scope-change request and stated that no
  band contained any part of it. 05 §12.11 **pins** the harness for exactly one flow on one engine
  (and [04 §7.7](04-dotnet8-migration-strategy.md) pins that engine), so it is in scope and is estimated
  here. What that did **not** settle is which engines get a
  functional assertion: **Gecko and WebKit get none**, and
  [R18](#r18--the-browser-executed-half-of-the-scripted-cart-flow-one-pinned-engine-and-an-open-decision-on-the-other-two)
  carries that residual as one of [section 9.4](#94-the-six-risks-that-are-approval-decisions-not-mitigations)'s
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
between the total of 107 / 195.5 / 351.5 and the 133.5 / 239.5 / 424 the later rounds supersede:
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
approval that records a decision on **every row of both approval registers** — each approved delta and each
approval-owned addition, by its own identifier — and on each risk this document escalates for
decision rather than mitigation.

**Basis: a census of approval acts at a stated rate**, and input 17's **approved-delta decisions are
deliberately not among them** — they are this gate's *requirement*, sized in their own row, for the reason
the partition below states.

**An approval act is defined so the census is countable and its members are mutually exclusive:** one
recorded output that no other act in the census produces. Each of the three kinds below answers a
**different question**, so no act's output can stand in for another's — a constituency that has not been
briefed has taken no position, an escalated question nobody answered is unanswered whoever else convened,
and a gate nobody recorded is not open. There are **eleven**:

| Approval acts | Count | What one act is, and the recorded output that makes it its own |
| --- | ---: | --- |
| **Constituency briefings** — security, product, engineering, the data owner, operations and **legal** | **6** | Convening one constituency, putting the thirteen deliverables in front of it, and recording **that constituency's stated readiness to proceed to a decision** — nothing more. The reading is *inside* the act, because the briefing is what the reading is for and a constituency reads only what bears on what it owns. The briefing explicitly does **not** answer any of the approved deltas, which [§7.2](#72-the-approved-delta-sign-offs) pays per decision, nor any of the **four** questions in the row below, which are separately recorded there |
| **Escalated risk decisions that are not themselves approved deltas** — [R1](#r1--the-target-framework-support-window), [R13](#r13--one-database-one-blast-radius), [R15](#r15--personal-data-governance-is-unowned), [R18](#r18--the-browser-executed-half-of-the-scripted-cart-flow-one-pinned-engine-and-an-open-decision-on-the-other-two) | **4** | Answering **one named escalated question** — which target-framework position to take, whether to accept one database's blast radius, who owns personal-data governance, which of three outcomes to take for the Gecko and WebKit functional residual — and recording that answer against its owner. Each is indivisible, and none is implied by any briefing above: a constituency can be briefed and ready without the question being answered, and R15's answer requires the data owner, security and legal to converge on one recorded owner rather than three positions. **R18 is in this census because [§9.2](#92-register-index)'s register row assigns its recorded outcome to this workstream**, and no other row in this document prices it |
| **The gate record** — the documented approval to begin implementation | **1** | Writing the decision that closes this workstream's exit gate, against which every later workstream's authority is checked. It is not any constituency's position and not an answer to an escalated question; it is the record that the other ten acts are complete |
| **Total acts** | **11** | |

**Why the three kinds cannot collapse into each other.** A briefing pays for assembling a constituency and
getting it to the point of deciding; an escalated decision pays for converging on one answer among owners
who may disagree; the gate record pays for writing the authority every later workstream is checked
against. The 6 and the 4 also do not correspond one-to-one — R15 alone draws three of the six
constituencies, R18's is the product owner's alone, and R1 and R13 draw overlapping subsets — so neither
count can be derived from the other, which is the property that makes them separate census dimensions
rather than one dimension counted twice.

**The rate is `0.3 / 0.6 / 1.2` IED per act**, and `11 × 0.3 / 11 × 0.6 / 11 × 1.2` =
`3.3 / 6.6 / 13.2`, which [§4.2.1](#421-the-rounding-rule-stated-once) rounds up to **3.5 / 7 / 13.5** —
the one row in this model whose unrounded product misses the half grid in all three columns, so the rule
does visible work here rather than none. The rate is this row's one judgement, and it is stated so it can be disputed: **low** is a
constituency that has read ahead and settles in a single sitting; **expected** is one sitting plus the
follow-up that circulating a position internally produces; **high** is a second full sitting, which is what
an act costs when the first one surfaces a question the constituency has to take away. This row is one of
the three where obtaining a decision **is** the work, so [§4.2](#42-the-unit-defined)'s exclusion of
meeting and approval-waiting overhead does not apply to it.

**The derivation is live, not a rationalization of a figure already chosen.** Each act carries
`0.3 / 0.6 / 1.2`, so the band moves by exactly that if the census does: a seventh constituency, or a fifth
escalated decision that is not itself a delta, adds it; an escalated decision reclassified as a delta and
paid in [§7.2](#72-the-approved-delta-sign-offs) removes it. **Assumption A5** enters through the rate
rather than as a caveat beside it: where a constituency is a role rather than a named person, its act is
priced at the **high** figure, because identifying the person who may take the position is part of taking
it.

**The band moved in the round before this one, and the act census is what moved it; it does not move
again here, because no input the census reads has changed.** No earlier reading derived it from an
act census at all — it was a judgement that the driver is the convening, and this census is what that
judgement looks like once it is made checkable. **The three corrections below do not cancel, and the
census is what shows they do not:** before them it would have counted **eleven** acts — five
constituencies, five escalated decisions and the record — which at this rate gives `3.3 / 6.6 / 13.2`,
rounded under [§4.2.1](#421-the-rounding-rule-stated-once) to **3.5 / 7 / 13.5**. The three corrections
add one act, remove two and add one, so the census is **eleven** acts again, and the band it gives is that
same 3.5 / 7 / 13.5. **The count agreeing is arithmetic coincidence and is stated as one:** two of the
original five escalated decisions left this census for [§7.2](#72-the-approved-delta-sign-offs) and two acts
of different kinds arrived, so the band is derived from the eleven acts enumerated above rather than from
any earlier composition that happened to number eleven.

> **Why six constituencies and not five, and why three escalated decisions and not five.** Input 17
> identifies five constituencies across the approved deltas, but the escalated risk decisions reach one
> further:
> [R15](#r15--personal-data-governance-is-unowned)'s owner is the data owner **with security and legal as
> co-approvers**, and [W16](#w16--personal-data-governance)'s policy stage names the same three. Legal
> therefore has to convene at this gate even though it owns no delta, so the convening cost is a
> **six**-constituency cost.
>
> In the other direction, **two of §9.4's six escalated decisions are themselves approved deltas** and are
> already priced per decision in [§7.2](#72-the-approved-delta-sign-offs):
> [R7](#r7--the-narrowed-browser-matrix) is the narrowed-browser-matrix delta, whose approval owner is
> product, and [R9](#r9--cutover-re-authentication-and-anonymous-cart-loss) is the
> reauthentication-and-anonymous-cart-loss delta, whose owners are product and operations. Counting their
> decisions here as well as there would be the double count this sub-section exists to remove. They remain
> in §9.4 because its subject is *which risks are decisions rather than mitigations*, which is true of them;
> what changes is only **which row pays for taking them**.
>
> **What the three corrections do to the act census, stated as arithmetic rather than as a cancellation.**
> The first **adds one act** — legal's briefing, `+0.3 / +0.6 / +1.2`. The second **removes two** — R7's and
> R9's decisions, `−0.6 / −1.2 / −2.4` — because they are paid per decision in §7.2 instead. The
> third **adds one** — R18's recorded outcome, `+0.3 / +0.6 / +1.2` — because
> [§9.2](#92-register-index)'s register row assigns that outcome to this workstream and no other row in this
> document prices it. The three net to **zero acts**, from an eleven-act census to an eleven-act one, and
> that is **not** a cancellation of the kind that leaves the membership alone: three movements of two signs
> and two kinds sum to zero *in count only*, which is why the band is derived from the membership above and
> not from the count.
>
> **The partition of input 17, stated once and precisely, because the alternative is a double count.**
> Input 17 carries two dimensions — **5** constituencies and the register's individual delta decisions — and **exactly one row consumes
> each**, so no unit of work is counted twice:
>
> | Dimension | Counted in | What it pays for |
> | --- | --- | --- |
> | **The convening** — 6 constituencies, being input 17's five plus legal | **W1's band**, this row | Its **eleven approval acts**: briefing each of the six owners on thirteen deliverables, taking the **four** escalated risk decisions of [§9.4](#94-the-six-risks-that-are-approval-decisions-not-mitigations) that are **not** themselves deltas — R1, R13, R15 and R18 — and writing the gate record |
> | **The individual delta decisions**, and the **15 approval-owned additions** of [05 §11.7](05-aspnet-core-migration-approach.md) beside them — the per-decision marginal cost | **[§7.2](#72-the-approved-delta-sign-offs)'s own row**, 8 / 16 / 26 | Obtaining and recording a decision on **each row of both registers** — that row states the priced count once — including consent from **every** constituency a delta names wherever [05 §11.5](05-aspnet-core-migration-approach.md)'s **Approval owner** cell names more than one — that register owns how many of its rows those are, and this document does not restate the subcount any more than it restates the total — and therefore including §9.4's **R7** and **R9**, which are themselves rows of the delta register |
>
> An earlier version of this basis listed the delta decisions among W1's own drivers while §7.2 asserted
> they were not double-counted into W1. Both statements could not be true of one band, and the ambiguity was
> the defect rather than the arithmetic: W1's band never moved when the delta count did — it was 3 / 6 / 12
> at every count input 17 has carried, the first reading's 14 included — so the decisions were only ever
> *sized* in §7.2, and what moved W1's
> band when it last moved was its own census gaining R18's decision. The basis above now says so
> in its own terms, and the two rows sum without overlap: **7 + 16 = 23** expected IED for the whole
> approval activity, which is what [§8.2](#82-concurrency-permitted-by-the-graph)'s **set 0** — the root
> set, which holds W1 and the sign-offs together and nothing else — and
> [§8.3](#83-the-critical-path-and-what-to-do-first-if-the-goal-is-to-narrow-the-estimate)'s chain both
> carry.

**Band 3.5 / 7 / 13.5. Medium confidence**, and it is `11 acts × 0.3 / 0.6 / 1.2` rounded up to the half
rounded up to the half
grid and nothing else — the census above, at the rate above. **The delta count does not enter it**, and
that is a property of the partition rather than an absorption: the count rose when input 17
was reconciled against [05 §11.5](05-aspnet-core-migration-approach.md), and **every row of that register —
and each of the 15 approval-owned additions of [05 §11.7](05-aspnet-core-migration-approach.md) beside them —
is paid per decision in [§7.2](#72-the-approved-delta-sign-offs)**, which is why that row is reported
separately from this one and where those decisions landed. What *would* move this band is a change in the act census: a
constituency added or removed, an escalated decision reclassified, or the gate record split. Confidence is
Medium because the count of acts is fixed and enumerated while the rate depends on other people's
availability — which is exactly what assumption **A5** and the high figure describe.

#### W2 — MVC 5 build reproduction and the restore precondition

**Scope** is [03 §5](03-modernization-roadmap.md)'s: **producing** the Windows verification run for the
migration source from a clean checkout, recording the conditions under which it was done, and running
the **AppCAT static assessment** inside the same restored tree.

**Basis — this row produces the gated run, and neither existing observation stands in for it.**
[10 §1.2](10-build-and-deployment-requirements.md), the owner of per-edition build outcomes, carries the
migration source's build assessment as **blocked pending a Windows verification run**. What that owner
establishes is a **precondition failure** — a clean checkout of the migration source commits no restored
packages at all, so no build can start until a restore succeeds against a source the repository never
declares. Two runs are on record and neither closes that. The Mono result in
[10 §3.1](10-build-and-deployment-requirements.md) is supplemental portability evidence rather than a
build on the prescribed toolchain; the later Windows run at
[10 §5.4](10-build-and-deployment-requirements.md) did restore and rebuild the migration source on the
prescribed toolchain, exiting `0` in Debug and in Release with zero warnings and zero errors, but
[10 §3.2](10-build-and-deployment-requirements.md) records it as **supplementary observation that does
not discharge the gate** and it is not a build from a clean checkout. W2 therefore carries four tasks rather than one: obtain a host
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
*outcome* is not settled, not because its tasks are unclear;
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
cosmetic.** [03 §4.2](03-modernization-roadmap.md) draws **gate 4a** — the build-governance bootstrap,
which is `global.json` and the root `NuGet.config` and **nothing else**, because at that point the
repository contains no project to restore — and **gate 4b**, which creates the **one** test project
[04 §12.2](04-dotnet8-migration-strategy.md) maps inside W6's converted project graph, restores it in
locked mode and gets its legacy-facing half green and captured. 4a's only entry is W1; 4b needs 4a,
**W6**, W2's gate 2b, W3 and W16 stage 1 [03 §5](03-modernization-roadmap.md). The table below is
grouped by gate and subtotalled per gate, because
[section 8.2](#82-concurrency-permitted-by-the-graph) places the two in **different concurrency sets**
and [section 8.3](#83-the-critical-path-and-what-to-do-first-if-the-goal-is-to-narrow-the-estimate)
needs 4a's own weight to compute the chain.

**The scope boundary that sets this band, stated next because it moves the largest single figure in this
model.** [03 §5](03-modernization-roadmap.md) splits the suite in two and gives this workstream **half (a)
and nothing more**: the shared contract assertions as **abstract bases under `Contracts/`**, the
**legacy-bound concrete classes under `Legacy/`** that bind them to the legacy application, the legacy
fixture with its private deployment lifecycle and its two-database reset, the fixture dataset both halves
load, the semantic normalization, the approved deltas enumerated as expected differences, and the
`Category=Baseline` execution half with the redacted baseline record it hands forward. **Half (b) — the
disposable engine fixture, the in-process target host, the fixture lifecycle and isolation policy, the
destructive-operation controls those two require, the browser-driven flow of
[05 §12.11](05-aspnet-core-migration-approach.md), and the
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
| **Gate 4a** — the **contracts project** itself, its **lockfile**, the **two `xunit.runner.json` files**, and the test projects' pins resolved and restored — with the **three runner and build-asset packages declared directly in every runnable test project** rather than inherited, because build and analyzer assets do **not** cross a project reference | **2** projects, **2** runner files, **5** lockfiles; **6** independent test-tooling pins plus `Microsoft.Extensions.Identity.Core` on the 8.0.30 band, and **3** of those declared twice (entry and pin counts, [04 §7.2](04-dotnet8-migration-strategy.md) for the pins and [04 §6.4](04-dotnet8-migration-strategy.md) for the lockfiles); **3** execution categories (input 14) | 1.5 | 3 | 5 |
| **Gate 4a subtotal** | | **2.5** | **5** | **8.5** |
| **Gate 4b** — the **contract topology**, now **declarable rather than described**: one **abstract** contract class per surface plus a **sealed concrete binding in each runnable assembly**, because a project reference does **not** enrol a referenced assembly's tests; **every base, concrete and fixture type `public`**; a **`[CollectionDefinition]` collection class per surface group per assembly** implementing `ICollectionFixture<TFixture>` with its **collection name taken from a `const`**, so a mistyped name is a compile error rather than a silently un-parallelized run; and the runtime-neutral context those bindings hand to the assertions | **6** abstract classes with per-assembly bindings, **1** neutral context, **9** normal collection groups plus **3** row-specific ones, in **2** assemblies (entry count, [05 §12.2](05-aspnet-core-migration-approach.md) for the topology and [05 §12.7](05-aspnet-core-migration-approach.md) for the collection groups) | 2.5 | 4.5 | 7.5 |
| **Gate 4b** — the runtime-neutral **`IStoreObserver`** with its explicit **DTO surface** and its **legacy implementation**, plus the **`IStoreSetup` write API** placed deliberately **off the injected context** so a contract body cannot name it, and **OWNER-backed** so a setup step cannot prove the runtime credential could have done it | `IStoreObserver`'s **8** original members and its **5** additional read projections; `IStoreSetup`'s **8** operations across **11** members; the first of **2** implementations, **1** DTO surface (entry count, [05 §12.6](05-aspnet-core-migration-approach.md)) | 3 | 5.5 | 9.5 |
| **Gate 4b** — the legacy fixture's **private deployment lifecycle**: copying the built legacy output outside the checkout, copying both store pairs, rewriting the deployed copy's connection strings including a **per-run Identity catalog name**, binding a per-run port, starting the host and **capturing its process id**, polling readiness and then polling **startup quiescence**, and stopping, detaching and deleting on teardown | **7** lifecycle steps (entry count, [05 §12.3](05-aspnet-core-migration-approach.md)), the seventh because the migration source provisions its administrator from an `async void` [src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:21] | 2.5 | 4 | 7 |
| **Gate 4b** — the legacy fixture's **store reset** and the destructive-operation controls **demonstrated** on the copies it destroys, which [03 §5](03-modernization-roadmap.md) makes a condition of this gate: the **three SQL identities**, the **durable ownership registry** with its JSON sidecar, **identifier re-resolution immediately before every `DROP`**, **run-marker validation** on copied directories, and a **standalone orphan-sweep class** with an **always-run cleanup job separate from the job the watchdog kills** | **2** store pairs, **4** committed files (file count, [05 §12.3](05-aspnet-core-migration-approach.md)); **3** hard gates, **3** identities, the registry and sidecar, the sweep class and the cleanup job, each with a refusal path to exercise (entry count, [05 §12.8](05-aspnet-core-migration-approach.md)) | 2.5 | 4.5 | 8 |
| **Gate 4b** — **the 152 cases that run against both fixtures**, authored here as the shared contract | **152** of the **326** cases (case count, input 14) | 15 | 23.5 | 39 |
| **Gate 4b** — the **twelve committed fixture inputs**: the deterministic **manifest** with its published per-entity counts, fixed keys, ranks and quantities, a named administrator, a migrated-hash account and a published fingerprint; **nine `ModelOverrides/` divergence-seam files**, one per schema-divergence dimension the diff gate must refuse on; `seed-expected.json`; and the **committed** `baseline-reference.json` — with the **legacy loader** and the **7 post-load invariants** asserted before the first case | **12** committed files: **1** manifest of 10/6/12/0/7/33 rows, **9** override files, **2** companions; **7** invariants; the first of **2** loaders (file and entry counts, [05 §12.3](05-aspnet-core-migration-approach.md)) | 2 | 5 | 7.5 |
| **Gate 4b** — semantic normalization, and every approved delta as an expected difference rather than a failure | Every row of the delta register (input 17, entry count); the volatile-value classes and the **4** pinned locale and collation values ([05 §12.10](05-aspnet-core-migration-approach.md)) | 2 | 3.5 | 6 |
| **Gate 4b** — the **two diagnostic record schemas** inside one allow-list, the **sanitized exception projection**, the **closed 26-code `operation` set**, the **redactor with its own test corpus**, and the **keyed one-way cross-run pseudonyms** whose corrected scheme invokes the **pinned `ILookupNormalizer`** — without which a failure is either undiagnosable, unattributable or a disclosure | **2** schemas, **7** masked classes, **26** operation codes, **2** failure classes plus the expected-difference state, **1** pseudonym key with a custody and destruction rule (entry count, [05 §12.9](05-aspnet-core-migration-approach.md)) | 3 | 5.5 | 9 |
| **Gate 4b** — the **152 `Category=Baseline` executions** on the Windows agent, and the **redacted baseline record** handed to the target half: **11 gating `baselineSource` values** compared and failed closed on, a separately recorded **7-fact `targetRun`** namespace never compared, a **`coverage` completeness object** carrying each `Category=Baseline` case identifier and status, an **artifact transfer with a digest sidecar**, and the **publication policy** — a `baseline-capture-diagnostic` on a failed or incomplete capture, and a `baseline-record` **only** on a complete run in which every one of those cases passed | **152** executions (input 14); **1** record with **11** gating values, **7** recorded facts and a completeness object, plus the transfer and the two publication outcomes (entry count, [05 §12.10](05-aspnet-core-migration-approach.md)) | 3 | 5.5 | 10 |
| **Gate 4b** — **culture, UI culture, time zone and collation** established as a published contract and enforced on this half's host and agent | **4** pinned values (entry count, [05 §12.10](05-aspnet-core-migration-approach.md)) | 0.5 | 1 | 2 |
| **Gate 4b subtotal** | | **36** | **62.5** | **105.5** |
| **W4 total** | | **38.5** | **67.5** | **114** |

**The addition, printed so it can be checked rather than trusted.** Gate **4a** — low
1 + 1.5 = **2.5**, expected 2 + 3 = **5**, high 3.5 + 5 = **8.5**. Gate **4b** — low
2.5 + 3 + 2.5 + 2.5 + 15 + 2 + 2 + 3 + 3 + 0.5 = **36**, expected
4.5 + 5.5 + 4 + 4.5 + 23.5 + 5 + 3.5 + 5.5 + 5.5 + 1 = **62.5**, high
7.5 + 9.5 + 7 + 8 + 39 + 7.5 + 6 + 9 + 10 + 2 = **105.5**. The workstream: 2.5 + 36 = **38.5**,
5 + 62.5 = **67.5**, 8.5 + 105.5 = **114**.

**How the case component was re-derived, because it is the largest single sub-row here and second only
to [W7](#w7--the-aspnet-core-port)'s target-facing case sub-row anywhere in this model.** The first
band authored the **98** both-fixtures cases of a 72-row matrix; the current matrix resolves the same
subject into **104 rows and 326 cases**, of which the **152** both-fixtures cases fall to this
workstream — the **32 both-fixture rows' 136 cases** plus the **16 both-fixture halves of the 7 mixed
rows** [05 §12.4](05-aspnet-core-migration-approach.md) — having passed through two intermediate
generations: 75 rows and 242 cases whose both-fixtures count was 144, then 102 rows and 307 cases whose
both-fixtures count was 147. **The
per-case authoring rate of the superseded band is held constant and the case count drives the
increase**: 152 ÷ 98 = **1.55×** to two places, so 9.5 / 15 / 25 scaled by that factor is
14.725 / 23.25 / 38.75, which
under [§4.2.1](#421-the-rounding-rule-stated-once)'s rule is **15 / 23.5 / 39**. The rates give the
same three figures directly: 152 × 0.097 / 0.153 / 0.255 = 14.744 / 23.256 / 38.76, which rounds to
15 / 23.5 / 39. The
resulting rates are **0.097 / 0.153 / 0.255 IED per case** — the same rates the three previous bands
printed, to that granularity. Nothing in the method changed; only the count did — and the rounding is
now [§4.2.1](#421-the-rounding-rule-stated-once)'s stated rule rather than the nearest half the
superseded figures took at the expected and high bands.

> **Holding the rate constant is a deliberate choice here, and it is worth stating why it is not obviously
> the conservative one.** Much of the case growth is concentrated in **boundary tables**: row 67 alone is
> a **54-case** field-by-field boundary matrix and row 72 moved from four cases to eight
> [05 §12.4](05-aspnet-core-migration-approach.md). A case inside a data-driven boundary table is cheaper
> to author than the first case of a new surface, which argues the blended rate should **fall**; against
> that, the same growth includes rows restructured rather than extended (28 and 36) and the divergence
> seam row 53, which are not table rows at all. Rather than adjust the rate in either direction on
> judgement, the derivation **holds it** — which reads slightly high for the table rows and slightly low
> for the restructured ones, and is the only choice that keeps this figure comparable with the two bands
> it supersedes. The **high band** is where the residual sits: 39 against 23.5 expected is a 1.66×
> spread on the largest row in this workstream.

**Seven items the previous band did not count at all, and they are the rest of the increase.** Each is
scope [05](05-aspnet-core-migration-approach.md), [04](04-dotnet8-migration-strategy.md) and
[03](03-modernization-roadmap.md) added after the previous derivation, and each is charged once:

- **the governance bootstrap of gate 4a** ([04 §6](04-dotnet8-migration-strategy.md), with the
  test-project pins of [04 §7.2](04-dotnet8-migration-strategy.md)) —
  **2 expected IED**, brand new to this workstream, because 03 moved the SDK pin and the package-source
  policy from the version W6 originated them in to the gate that precedes it, and added the check that the
  host SDK actually satisfies the pinned band. **It does not include a restore or a build**: at 4a the
  repository holds no project, so the first locked-mode restore is W6's and the first restore of the suite's
  own lockfile is gate 4b's. W6 loses the corresponding half-day, so this is a **transfer plus a
  verification obligation**, not a duplicate;
- **the contract topology** ([05 §12.2](05-aspnet-core-migration-approach.md)) — now **4.5 expected**,
  its own sub-row, because per-surface
  **abstract** classes with **`sealed` per-host derivations in the same assembly**
  [05 §12.2](05-aspnet-core-migration-approach.md) are structure rather than a naming convention: a test
  method is discovered on a class the test assembly itself declares, so every shared contract needs a
  concrete derivation authored for each host — the legacy-bound one here, the target-bound one in W7 — and
  with one assembly the hazard moves from cross-assembly discovery to the declaration site, which is why
  every base, concrete and fixture type is fixed `public` there;
- **`IStoreObserver`, its DTO surface and its legacy implementation**
  ([05 §12.6](05-aspnet-core-migration-approach.md)) — now **5.5 expected**
  with `IStoreSetup` alongside it, and with no predecessor in the previous derivation, which asserted
  database outcomes per runtime rather than through one neutral abstraction. It is what lets a shared
  contract assert **an absence** — "no row written" — against two different stores;
- **startup-quiescence polling** in the lifecycle ([05 §12.3](05-aspnet-core-migration-approach.md)) —
  the seventh step, taking that sub-row from
  2.5 to **4 expected**, because a readiness probe answering does not mean the source's `async void`
  administrator provisioning has finished writing;
- **the third SQL identity, the durable ownership registry with its sidecar, identifier re-resolution
  before every `DROP`, run-marker validation, the standalone sweep class and the always-run cleanup job**
  ([05 §12.8](05-aspnet-core-migration-approach.md)) — taking the reset sub-row from 2.5 to
  **4.5 expected**. A cleanup job that must survive the
  watchdog killing the test job is a second job, not a `finally` block;
- **the enlarged fixture dataset with published counts, its seventh invariant and its two companion
  files** ([05 §12.3](05-aspnet-core-migration-approach.md)) — **+0.5 expected**, deliberately small:
  the dataset grew in rows, and rows in a
  manifest are volume rather than structure;
- **the diagnostic and provenance closures** — the sanitized exception projection, the 26-code
  `operation` set and the corrected keyed-alias scheme ([05 §12.9](05-aspnet-core-migration-approach.md))
  at **+1 expected**, and the provenance
  split with its digest-sidecar transfer and fail-closed consumption
  ([05 §12.10](05-aspnet-core-migration-approach.md)) at **+1.5 expected** on
  the execution row, which also absorbs 98 → 147 executions.

**And three further approved deltas** (input 17) take the normalization sub-row from 3 to **3.5
expected**. Two of the three are new *shapes* of expectation rather than new rows of the same shape — a
`405` where the source answers `404`, and a `400` where the source accepted and wrote — so the sub-row
moves by half a day rather than by nothing.

**Re-derived in the round before this one, from 35.5 / 60.5 / 102.5, and that movement was six components
and nothing else.** The case sub-row **moved**, and it was the smallest of the six. The matrix grew from
239 cases to 242 and then to **307**: the first three added cases were **target-only** — row 24's fifth
case and row 64's fourth and fifth — but of the twenty-seven rows the owner's table then carried at
ids 76–102, **two were both-fixture and carried three cases**, so the both-fixtures count this gate
authors went 144 → **147** and the sub-row went 14 / 22 / 36.5 → **14.5 / 22.5 / 37.5**,
**+0.5 / +0.5 / +1** [05 §12.4](05-aspnet-core-migration-approach.md). What moved besides it was
structure:

- **the topology became declarable rather than described** — every base, concrete and fixture type
  `public`, and a `[CollectionDefinition]` collection class **per surface group** implementing
  `ICollectionFixture<TFixture>` with its name taken from a `const`, across **9** normal groups and **3**
  row-specific ones in the one assembly ([05 §12.7](05-aspnet-core-migration-approach.md)). That is
  **12 declarations** rather than a convention,
  so 2 / 3.5 / 6 becomes 2.5 / 4.5 / 7.5 — **+0.5 / +1 / +1.5**;
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
- and **the project row — now gate 4b's** — absorbs the **direct declaration of the runner and
  build-asset packages in the one runnable test project**, plus `Microsoft.Extensions.Identity.Core` as the
  pin the suite resolves `ILookupNormalizer` from, so 1.5 / 2.5 / 4 becomes 1.5 / 3 / 5 —
  **0 / +0.5 / +1**.

0.5 + 1 + 0.5 + 0 + 0 + 0.5 = **+2.5**, 1 + 2 + 1.5 + 0.5 + 0.5 + 0.5 = **+6**, and
1.5 + 3.5 + 2 + 1 + 1 + 1 = **+10** — the last term of each being the case sub-row above — so
35.5 + 2.5 = **38**, 60.5 + 6 = **66.5** and
102.5 + 10 = **112.5**.

**Re-derived again this round, from 38 / 66.5 / 112.5, and the movement is one component and nothing
else.** [05 §12.4](05-aspnet-core-migration-approach.md) resolved the matrix from 102 rows and 307 cases
into **104 rows and 326 cases** — one further mixed row, one further target-only row, and the rest of the
growth inside rows that already existed — and the both-fixtures count this gate authors went
147 → **152**: the **32** both-fixture rows carry **136** cases rather than 135, and the **7** mixed rows
contribute **16** both-fixture halves rather than the 12 that six contributed. So the case sub-row goes
14.5 / 22.5 / 37.5 → **15 / 23.5 / 39**, `+0.5 / +1 / +1.5`, and the gate-4b subtotal with it,
35.5 / 61.5 / 104 → **36 / 62.5 / 105.5**: 38 + 0.5 = **38.5**, 66.5 + 1 = **67.5** and
112.5 + 1.5 = **114**. **Nothing else in this workstream moves, and one row deliberately does not.** The
execution sub-row absorbs 147 → **152** `Category=Baseline` executions at 3 / 5.5 / 10 **unchanged**,
because it is priced on the *record's* structure — its 11 gating values, its 7 recorded facts and its
completeness object — rather than per execution, exactly as the 98 → 147 step was absorbed a round
earlier.

This band also depends on assumption **A2** holding in a stronger form than A2 states: the Windows agent
of [05 §12.10](05-aspnet-core-migration-approach.md)'s two-platform split must **run** the legacy
application, not merely build it.

**What is not in this band, and where it went.** The console's **data-migration verbs** are **W7's** — 03 §5's
W7 scope statement places it there and [section 5.2](#w7--the-aspnet-core-port) carries its 9 expected
IED — the manifest's **target loader**, being catalog-direct plus those verbs' `load-catalog` path, and the
observer's **target implementation** are inside W7's machinery sub-row; the **browser-driven flow** of
[05 §12.11](05-aspnet-core-migration-approach.md) is W7's own sub-row, since the flow it drives exists
only in the ported application; and the
**dual-OS pipeline stages** that automate the two agents of
[05 §12.10](05-aspnet-core-migration-approach.md), together with the **browser-install
step**, are **W11's manifest half**, whose architecture
[06 §12.1](06-azure-hosting-recommendations.md) owns. None is charged here,
and none is charged twice.

**Band 38.5 / 67.5 / 114, splitting 2.5 / 5 / 8.5 for gate 4a and 36 / 62.5 / 105.5 for gate 4b. Low
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
abstract-plus-sealed contract topology that has to enrol both halves' tests from one assembly, a
runtime-neutral store
observer with two implementations, a keyed
pseudonym scheme with key custody and destruction, a redactor that must itself be tested, and a
three-identity bootstrap with a durable
ownership registry whose deletion path re-resolves its target before acting.
**Re-derived from 38 / 66.5 / 112.5**, itself re-derived from 35.5 / 60.5 / 102.5, from 21.5 / 37 / 64,
from 11.5 / 19 / 33, from 8 / 13 / 22, and before that from 12 / 20 / 34 when the target-facing half
moved to W7.
**This is still the second-largest row in the model and the one most likely to be cut under pressure**;
[R3](#r3--the-absent-regression-baseline) is why cutting it removes the only means of substantiating
behaviour preservation. **The gate split makes one cut newly tempting and it is the wrong one**: gate 4a
is 2 of the 67.5 and W6 consumes it, so cutting 4a does not save 2 — it stops W6, and W6 is what gate 4b's
own project is created inside.

**The visual baseline capture is not inside this band.** [03 §5](03-modernization-roadmap.md) makes it
part of **gate 4b**, and it is sized in [§7.1](#71-the-manual-visual-review) as a non-code
task so that the capture and the review are visible as one item split across two points in the sequence.
[Section 8.2](#82-concurrency-permitted-by-the-graph) places its **5.5** expected IED — the capture
subtotal of [§7.1](#71-the-manual-visual-review) — — the capture half
[§7.1](#71-the-manual-visual-review) sizes, whose band that section states and this one does not repeat —
in the same concurrency set as **gate 4b**, the gate
the capture sits inside.

#### W5 — Repository-wide path-casing audit

**Scope** is [03 §5](03-modernization-roadmap.md)'s: every mismatch identified and its correction
specified, with the audit made **repeatable as a pre-deployment check** rather than performed once.

**Basis: input 31.** The search space is fully enumerated and it is **repository-wide**, as this
workstream's own name says — not the migration source's, whose asset, helper and `@Url.Content` counts
(inputs 7, 8 and 11) cover MVC 5 alone. Input 31 is stated **partitioned by edition** so that "all three
editions" is a count rather than an adjective, and it yields two totals that are not interchangeable: the
**containers** a checker must open, and the **literal sites** inside them. Every figure is command-verified
in [A.3](#a3-helper-view-and-site-counts).

| Edition | Served static files | Views | `BundleConfig.cs` | **Containers** | `~/` bundle literals | `@Url.Content` | Render sites | **Literal sites** |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| MVC 5 | 28 | 29 | 1 | **58** | 12 | 4 | 11 | **27** |
| MVC 4 | 90 | 29 | 1 | **120** | 24 | 4 | 10 | **38** |
| MVC 3 | 55 | 25 | 0 | **80** | 0 | 23 | 0 | **23** |
| **All three** | **173** | **83** | **2** | **258** | **36** | **31** | **21** | **88** |

The container total is `173 + 83 + 2 = 258` and the site total is `36 + 31 + 21 = 88`. **Both
`BundleConfig.cs` files are containers in their own right** — they hold 36 of the 88 sites between them
while being neither a served asset nor a view — and **MVC 3 has none**, because it ships no `App_Start`
folder, so its 23 literal sites are all `@Url.Content` calls in views. **23 of the 31 `@Url.Content` sites
are MVC 3's**, which is what reorders the work: a pattern set validated only against the migration source's
4 is unproven against the edition where the concentration actually is.

**The defect is not confined to the migration source either, and that is what turns "repository-wide" from
a scope word into a cost.** The known mismatch is
[src/MVC5/MvcMusicStore/App_Start/BundleConfig.cs:28], where `~/Content/site.css` is registered against a
tracked
[src/MVC5/MvcMusicStore/Content/Site.css:1,450 bytes] — the mismatch is in the tracked path's own
capitalisation rather than in any line of the file, so the stylesheet is cited at its size and
`git ls-files 'src/MVC5/MvcMusicStore/Content/*'` is what prints the capital `S`.
**The identical defect exists in MVC 4** — the same lowercase `~/Content/site.css` at
[src/MVC4/MvcMusicStore/App_Start/BundleConfig.cs:26] against a tracked
[src/MVC4/MvcMusicStore/Content/Site.css:17,249 bytes], whose capitalisation the same command prints
for that edition (`git ls-files 'src/MVC4/MvcMusicStore/Content/*'`) — so the second
`BundleConfig.cs` is not a container the audit opens for completeness' sake; it holds a live mismatch of
the same class, and [A.5](#a5-the-corroborating-case-mismatch-r8) prints both halves of both pairs.

**What is repository-wide here and what is not, because [03 §5 W5](03-modernization-roadmap.md) draws that
line inside this row rather than around it.** Its scope is every path literal "across bundle registrations,
`@Url.Content` calls, view paths and any other path literal"; its exit condition 1 names **the migration
source** for the corrections it hands to W7. Both are right, and the estimate is sized on the first: the
**enumeration and the checker are repository-wide**, because a checker wired into W11's Test stage runs
over a repository rather than over one folder, and because its pattern set is only proven where the
literals concentrate — MVC 3's 23 sites, which no migration-source pass touches. The **correction list W7
consumes is the migration source's**, because W7 ports one edition and the other two are retained
read-only.

**That split produces one obligation which is a decision rather than a task.** A checker that "fails on any
mismatch" over this repository would fail from the moment it is wired in, because MVC 4's mismatch above is
real and **cannot be corrected** — that edition is retained read-only, so there is no version of this work
in which the checker passes and MVC 4 is repaired. This row therefore has to record an **explicit
enforcement boundary**: which paths the checker fails on, which it reports without failing, and the recorded
reason for the difference. Without it the checker is either wrong or permanently red, and the choice is not
W11's to discover while wiring it in.

**No exit gate in this row is a deployment, and that is 03's placement rather than this document's.**
[03 §5 W5](03-modernization-roadmap.md)'s exit is a complete enumeration with each correction specified,
plus a **repeatable checker that runs without a deployment** — which is what
[06 §3.4](06-azure-hosting-recommendations.md)'s **G-CASING-STATIC**, the repository-wide inventory gate,
requires. Its sibling **G-CASING-SERVE** is the runtime gate and is discharged at
[W10](#w10--hosting-provisioning-and-platform-configuration), against a deployed application. **06 defines
both gates, [05 §8.1.1](05-aspnet-core-migration-approach.md) owns the asset inventory they resolve
against, [03](03-modernization-roadmap.md) owns the correction workstream and its sequencing, and this
document prices it and assigns no coverage row to either gate.**

| Component | L / E / H | What it is |
| --- | ---: | --- |
| The enumeration — one pattern set over **258** containers and the **88** literal sites inside them | 0.5 / 1 / 2 | Mechanical, and wide rather than deep: the cost is in proving the pattern set against MVC 3's concentration, not in reading 258 files |
| The **migration-source** correction list W7 consumes | 0 / 0.5 / 1 | One list against one edition, handed to a workstream that is already editing those files |
| The **repeatable checker** and its recorded **enforcement boundary** | 0.5 / 0.5 / 1 | The checker is a pattern set plus a runner; the boundary is one recorded decision with a stated scope, not a second audit |
| **Band** | **1 / 2 / 4** | |

**Band 1 / 2 / 4**: low `0.5 + 0 + 0.5 = 1`, expected `1 + 0.5 + 0.5 = 2`, high `2 + 1 + 1 = 4`.
**High confidence** — the space is command-verified and the exit is a checker rather than an environment.
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
[03 §4.2](03-modernization-roadmap.md)'s **gate 4a** creates `global.json` and the root `NuGet.config`,
and **W6 consumes gate 4a**. So this row **inherits** those two files rather than authoring them — while
still authoring the root solution, the tool manifest and the converted project's lockfile, and still
**proving** the inherited pair against a project that did not exist when 4a committed them. **Gate 4a
proves nothing about restore**, and that is a correction to an earlier reading of this row: at 4a the
repository contains no project, so there is nothing to restore — 4a commits the two files, checks the
host SDK against the pinned band and stops [03 §5](03-modernization-roadmap.md). The **repository's first
locked-mode restore is therefore this workstream's**, performed on the converted project against the
inherited source configuration, which is why the movement is **−0.5 expected and −0.5 high with no change
at the low band** rather than the whole of gate 4a's **2** expected.

**What this row does *not* include, and it is the correction that keeps the band honest.**
[03 §5](03-modernization-roadmap.md)'s W6 exit gate deliberately requires **neither a build nor a test
run**, because the *unported* application cannot compile on the target framework at all — the types it
is built on are removed rather than renamed. **W4's gate 4a is this workstream's predecessor and W4's
suite is not part of this gate** — the edge is `4a → W6`, so what W6 consumes from W4 is the governance
bootstrap and not a baseline that runs. **The reverse edge `W6 → W4b` does not put the suite
inside this band either.** What W4's later gate consumes from here is the **project graph its own test
project is created in** and the completed governance set that project's lockfile is generated under — a
compilation and governance dependency, not a test run — so authoring, restoring, building and running
that project is gate 4b's effort and is priced in W4's band rather than in this one
[03 §4.2.3](03-modernization-roadmap.md#423-six-properties-of-the-graph-worth-reading-before-the-workstreams). What replaces those two conditions is a named piece of work this
band must carry: the **compile diagnostics enumerated as the expected state**, every one mapped to a
no-successor construct. **14** of the **23** blockers fail at compile time, of which **13 apply to the
migration source**; the fourteenth,
[F-12-14](12-migration-blockers.md), is MVC 4's committed build configuration and belongs to a reference
edition rather than to the port ([12 §3](12-migration-blockers.md); input 12, work-item count). No unmapped
diagnostic may be left unexplained and no named construct may be left silent, since a construct that
produces no diagnostic means the conversion did not take effect where it was supposed to. Real compilation
and a green suite are W7's exit gate.

**Band 2.5 / 4.5 / 8.5. High confidence** — every input is enumerated and every outcome is checkable: 28
package outcomes each already decided by [04 §8](04-dotnet8-migration-strategy.md), one format conversion,
three manifests authored here, two inherited and re-verified, and a diagnostic set adjudicated against 14
named constructs.

#### W7 — The ASP.NET Core port

**Scope** is [03 §5](03-modernization-roadmap.md)'s, with every design decision owned by
[05 §2–§9](05-aspnet-core-migration-approach.md) and the file-by-file target owned by
[04 §12](04-dotnet8-migration-strategy.md).

**Basis — decomposed, because a single band over the largest row would hide its structure.** All line
figures are the **sizing metric**; all others are labelled.

| Component | Input | Low | Expected | High |
| --- | --- | ---: | ---: | ---: |
| Composition root and dependency injection, including the **shared application-services seam** placed as the composition root's **first** registration and published with an **allocation table** stating which consumer — the web application, the provisioning tool, the seed entry point, the in-process test host — takes which registrations | **10** manual construction sites (site count); **1** seam with **4** consumers ([05 §3.1](05-aspnet-core-migration-approach.md)) | 2.5 | 5 | 8.5 |
| Ordinary application code | **895** non-blank lines (sizing) | 5 | 9 | 15 |
| Authentication rewrite | **382** non-blank lines (sizing) | 6 | 11 | 18 |
| Seed data — a data decision, **not** line porting | **820** non-blank lines (sizing) | 1 | 2 | 4 |
| Views, helpers and view components | **29** views, **5** naming legacy types; **11** bundling sites; **3** child actions becoming **3** view components (file and site counts) | 4.5 | 9 | 15 |
| Static assets and their acquisition | **27** assets in the migration source's four asset groups (file count) | 1 | 2 | 4 |
| Demonstrating the silent-runtime resolutions | **8** of the **23** blockers (work-item count) | 2 | 4 | 7 |
| The **security-header middleware**, the **error action** and the **generic error view** — one application-owned middleware at a fixed pipeline position, covering four response kinds, with a content-security mode key that has no default; `HomeController.Error` with its two attributes, its two feature paths, its three-field view model and its status preservation; and a **layoutless** error view, because the layout queries data an error path cannot assume is available | **1** middleware / **4** response kinds; **1** action / **2** attributes / **2** feature paths; **1** view with `Layout = null` ([05 §2.4, §7.1, §8.3](05-aspnet-core-migration-approach.md)) | 1.5 | 3 | 5 |
| The ledger's **eight further entries** implemented in the ported controllers — the cross-cart `404`, the album binding allow-list with its partial edit, the missing-row check on delete, the four unknown-identifier `404` conversions, the checkout's caught-persistence feedback, **global anti-forgery adoption answering `400` on the five previously unprotected POSTs**, **verb-mismatched POST-only routes answering `405` where the source answers `404`, sign-out included**, and the **cart-migration failure notice**; plus the **status-200 ownership-denial contract**, which returns the shared error view with status 200 rather than a redirect | **8** rows of the delta register (input 17, entry count); **5** unvalidated POSTs (input 18, census) | 2 | 4 | 6.5 |
| Target-facing half of the suite — the **machinery**: the disposable engine fixture, the in-process host with `ConfigureWebHost` as its **single** override point and a **three-line `ConfigureTestServices` allow-list** installing the logger provider and a command interceptor on **each** of the two `DbContext` types, the run-scoped owner and runtime principals with the ownership registry and orphan sweep, **`DENY`-based** fault injection with the one-shot trigger's `SEQUENCE`, the three-layer timeout model, the lifecycle, isolation and parallelism policy across **9** normal groups and **3** row-specific ones, the destructive-operation controls **demonstrated**, the observer's **target implementation** and `IStoreSetup`'s, the fixture dataset's target loader, the **`--model-overrides` divergence seam**, the two-host topology the continuity rows need, **row 25c's own half-migrated database and its two-host lifecycle**, and the **public deployed-only fixture and test class** authored here for execution at the deployed gate | **12** collection-fixture groups, **1** override point, **8** injected keys, **2** clients, **2** interceptors (entry count, [05 §12.7](05-aspnet-core-migration-approach.md)); **3** gates, the registry, the sweep, **3** fault-injection mechanisms and **3** timeout layers (entry count, [05 §12.8](05-aspnet-core-migration-approach.md)); the second of **2** implementations ([05 §12.6](05-aspnet-core-migration-approach.md)) and the second of **2** loaders ([05 §12.3](05-aspnet-core-migration-approach.md)); **9** override files behind the divergence row; **4** continuity rows / **10** cases (case count, input 14) | 12.5 | 22.5 | 40 |
| Target-facing half of the suite — the **browser-driven flow**: the pinned harness driving Chromium over the cart page's script-issued removal request, asserting the request header actually sent, the JSON response handling, the four DOM updates, zero console errors and zero policy-violation reports — on a **separate real-Kestrel host bound to a loopback port**, because the automation cannot reach an in-memory `TestServer` | **1** flow, **1** engine, **1** install step, **1** additional host (entry count, [04 §7.7](04-dotnet8-migration-strategy.md) for the pinned engine and its install step); target-only row **28b** ([05 §12.11](05-aspnet-core-migration-approach.md)) | 2 | 3.5 | 6 |
| Target-facing half of the suite — the **cases**: **152** re-pointed at the target fixture and **174** target-only authored here, including the **checkout token negatives**, the **`405` rows**, row 64's **five** journal-keyed interruption cases and the **eight published `provision-admin` executable-contract cases** of row 75 | **152** + **174** of the **326** cases (case count, input 14) | 23.5 | 36.5 | 60 |
| `tools/provision-admin`'s **data-migration verbs** — the artifact the migration gates of [03 §5](03-modernization-roadmap.md) are enforced **by**, built here because most of them are functions of this workstream's output; now also carrying `extract-schema`'s **immutable-artifact handoff**, the **`--store catalog\|identity` grammar** with its per-store receipt, the **`--model-overrides` seam** with its nine override files, and the four **`dbo.__DataMigration*` run tables** written **in the same transaction as each group's commit**, with the filesystem receipt derived from them *after* commit and never read to decide a resume | **8** data-migration verbs of the console's ten, whose switch grammar and **five**-code tool-wide exit allocation are [05 §10.2](05-aspnet-core-migration-approach.md)'s (entry count); the coverage rows that **drive** them are exactly those [03 §4.3](03-modernization-roadmap.md)'s ownership map homes at **W8's** gate or **W9's** — the Identity load's and the domain load's — namely ids **18, 53–61, 63–65, 76–82, 84, 94–95**, which are **23** rows carrying **55** cases by [05 §12.4](05-aspnet-core-migration-approach.md)'s own per-row counts: `3` for row 18, `10 + 2 + 2 + 1 + 1 + 1 + 1 + 1 + 1 = 20` for 53–61, `1 + 5 + 3 = 9` for 63–65, `3 + 2 + 2 + 2 + 3 + 2 + 2 = 16` for 76–82, `2` for 84 and `3 + 2 = 5` for 94–95, so `3 + 20 + 9 + 16 + 2 + 5 = 55` (case count, input 14). **The set is now complete over the whole matrix**: the ten driver rows among the ids from 76 up — 76 to 82, 84, and the seed-transaction pair 94 and 95 — carry 23 of those 55 cases, and the eleventh such row the same map touches, id 83, is **excluded** because that map homes it at W12's gate with the operator tool's published output. Completing the census **moved the count without moving this band**, because the band above is priced on the **eight verbs** and their exit allocation, and the driver rows state *what enforces them* rather than multiplying into the figure. **This is the one statement of that count in this document**, and the second passage that needs it, in this workstream's derivation below, points here and carries no figure of its own. **Row 75's eight cases are excluded, and so are row 24's seven**: they accept the `admin` verb and the `seed` guard, which [W12](#w12--administrator-provisioning-tool) builds and which that map homes at W12's gate | 6 | 11 | 19.5 |
| **W7 total** | | **69.5** | **122.5** | **208.5** |

**The addition, printed so it can be checked rather than trusted.** The **nine port sub-rows** are
2.5 + 5 + 6 + 1 + 4.5 + 1 + 2 + 1.5 + 2 = **25.5** low,
5 + 9 + 11 + 2 + 9 + 2 + 4 + 3 + 4 = **49** expected
and 8.5 + 15 + 18 + 4 + 15 + 4 + 7 + 5 + 6.5 = **83** high. Adding the four suite-and-artifact sub-rows:
25.5 + 12.5 + 2 + 23.5 + 6 = **69.5**, 49 + 22.5 + 3.5 + 36.5 + 11 = **122.5**, and
83 + 40 + 6 + 60 + 19.5 = **208.5**.

**Ten notes on that decomposition, each of which is a place a naive estimate goes wrong.**

- **The authentication rewrite is 382 non-blank lines (sizing metric), about 18 percent of the migration
  source** — and it is the largest of the **seven porting sub-rows** despite that share, because it is the
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
  effort** (2 ÷ 122.5). That asymmetry is the whole point of [08 §4.2](08-technical-debt-register.md)'s
  instruction: 820 non-blank lines of hardcoded catalog rows are bulk data expressed as code, and the
  work is deciding how the data reaches the database, not porting 820 lines.
- **Ordinary application code — 895 non-blank lines — is the genuinely mechanical part**, and it is
  43 percent of the source and about 7 percent of this row (9 ÷ 122.5). An estimate that scaled from
  total lines would land almost entirely in the wrong place.
- **The 8 silent-runtime blockers get their own sub-row on purpose.** Their exit criterion
  [03 §5](03-modernization-roadmap.md) is *demonstration*, not code review — because their failure mode
  is silence, an assertion that fails when the resolution is absent is the only acceptable evidence.
  That cost belongs to W7, and so does the machinery it runs on: the two target-facing sub-rows below it
  are this workstream's, because [03 §5](03-modernization-roadmap.md) gives W4 only the legacy-facing
  half of the suite.
- **The target-facing half is three sub-rows because its machinery and its cases scale on different
  things, and the case row is derived rather than asserted.** Of the **326** cases, **152** already exist
  as W4's shared contract and are **re-pointed** at the target fixture — a discount, not a rewrite, at
  **0.04 / 0.06 / 0.10 IED per case**: 152 × 0.04 / 0.06 / 0.10 = 6.08 / 9.12 / 15.2, which under
  [§4.2.1](#421-the-rounding-rule-stated-once)'s rule is **6.5 / 9.5 / 15.5**, since the assertions are
  authored and only the
  fixture, the approved-delta expectations and the diagnostics differ. The other **174 are target-only and
  are authored here** at the same **0.097 / 0.153 / 0.255 IED per case** W4's row uses —
  174 × 0.097 / 0.153 / 0.255 = 16.878 / 26.622 / 44.37, so **17 / 27 / 44.5** — spread over the
  **65 target-only rows' 161 cases** and the **13 target-only cases of the 7 mixed rows**, and
  161 + 13 = **174**. 6.5 + 17 = **23.5 low**, 9.5 + 27 = **36.5 expected**,
  15.5 + 44.5 = **60 high**. **Both halves are rounded up per that rule at every position**, which the two
  superseded figures did not do at two of theirs — 8.64 taken to 8.5 and 9.51 to 9.5 — so a reader
  comparing the two derivations is seeing the stated rule applied, not a changed rate. The executions
  reconcile against [05 §12.4](05-aspnet-core-migration-approach.md): W4 runs **152**, the target side is
  **326** — the 152 target halves plus the 174 target-only cases — and 152 + 326 = **478**. Those 326
  target-side executions are 05's **325 `Category=Target`** plus its **single `Category=Deployed`** case.
  **Nine of the 326 do not execute at this workstream's exit gate**, and the reason is availability rather
  than allocation: **row 47's single `Category=Deployed` case** needs a deployed host, which does not exist
  at W7's exit, and **row 75's eight cases** accept the provisioning tool
  [W12](#w12--administrator-provisioning-tool) builds, which does not exist there either. So **317 execute
  at W7's exit**, **8 at W12's exit** and **1 at the deployed verification's own stage**
  [06 §12.1](06-azure-hosting-recommendations.md) — 317 + 8 + 1 = **326**. **Authoring all 326 remains
  this sub-row's**, because a case is authored once and the charge-once rule puts that charge where the
  authoring happens; what moved is the **execution and the acceptance**, which are charged in W12 and in
  W11's manifest half respectively. **`Category=Sweep` is a trait value and not a fourth category**, so it
  adds nothing to either side of that reconciliation; the sweep class it marks is W4's, and its always-run
  cleanup job is charged in W4's reset sub-row.
- **No sub-row here buys browser automation, and its absence is a scope fact rather than an omission.**
  [06 §10.4](06-azure-hosting-recommendations.md) states that there is no functional browser automation in
  any engine, and [05 §12.11](05-aspnet-core-migration-approach.md) carries the one script-dependent flow —
  the cart page's removal handler — as an **open limit**: its endpoint, response shape, token transport and
  static script contract are asserted at HTTP level inside the case row above, while the handler's
  **browser-executed** behaviour is not asserted by any automated case. **The evidence that does exist for
  it is manual, it is required, and it is charged in a different workstream**: the four-engine functional
  walk [06 §10.4](06-azure-hosting-recommendations.md) obliges is priced at
  [W10](#w10--hosting-provisioning-and-platform-configuration)'s gate 10b, where a deployed application
  exists to be walked, and gated as [03 §5](03-modernization-roadmap.md)'s W10 exit condition 8. So **no
  band in this workstream moves on account of it** — this sub-row carries the HTTP-level and static-contract
  coverage and nothing browser-executed, which is a placement fact rather than an omission. What remains at
  W1 is
  [R18](#r18--the-browser-executed-half-of-the-scripted-cart-flow-one-pinned-engine-and-an-open-decision-on-the-other-two)'s
  acknowledgement that the walk is point-in-time manual evidence and not repeatable automated coverage. A
  pinned harness would close the residual properly and is **not approved scope** — it would need an
  amendment to AAP §0.5.2's four test packages — so it is priced nowhere.
- **The two-host replica/restart topology is machinery, and it is here rather than in W4.** Continuity is
  **4 rows and 10 cases** (25, 26, 42, 43 [05 §12.4](05-aspnet-core-migration-approach.md)), all
  target-only, and they need **two hosts sharing the intended stores** rather than a second assertion —
  the cookie, session and anti-forgery continuity claims are false if a single host answers both requests.
  That topology exists only against the ported application, so a legacy-side equivalent is not merely
  unnecessary, it is impossible.
- **The artifact sub-row is a tooling project, not part of the ported application.** It carries the
  console's **data-migration verbs** and the exit-code contract
  [05 §5.4, §5.6](05-aspnet-core-migration-approach.md) owns, and the coverage rows of
  [05 §12.4](05-aspnet-core-migration-approach.md) that **drive them** rather
  than a pre-populated database — which is what makes the migration testable at all, since a
  pre-populated target proves nothing about the migration that filled it. **The row and case count for
  those drivers is stated once, in this workstream's component table above, and deliberately not
  restated here**, so that a re-count against 05's matrix has exactly one cell to land in. It is charged
  here because
  [03 §5](03-modernization-roadmap.md)'s W7 scope statement places it here and because most of those
  verbs are functions of the ported model and the two migration sets this workstream produces. **Its
  increment in the round before this one was not a further verb**: the verb inventory and the exit-code contract are
  unchanged,
  and what arrived is **`extract-schema`'s immutable-artifact handoff** — per-manifest byte lengths,
  digests and a load receipt the load verbs refuse to proceed without — and the **`--model-overrides`
  seam**
  with its **nine** override files, which is what makes the divergence row assertable instead of
  hypothetical. **+1.5 / +2.5 / +4** on a sub-row that was 3.5 / 6.5 / 12. Its
  **invocation** from the release path is W11's manifest half and its **principal** is
  [06 §6.2](06-azure-hosting-recommendations.md)'s; neither is charged again here.
- **The error-handling port sub-row is application code the earlier derivations under-specified, and its
  third component is the reason it moves.** The
  **security-header middleware** is application-owned rather than platform-supplied, sits at a stated
  pipeline position, must produce its set on the **five** response kinds [06 §10.2](06-azure-hosting-recommendations.md) enumerates — among them a static file, a redirect in both its kinds, a status-code page and a
  re-executed error response, and reads a content-security mode key that **has no default**, so the host
  fails to start if nothing supplies it. **`HomeController.Error`** is the successor to a removed
  attribute-and-model pair rather than a ported action: it is reached by **two** feature paths — the
  exception handler and status-code re-execution — carries `[AllowAnonymous]` and a **ledgered**
  `[IgnoreAntiforgeryToken]`, and hands a **two-member** view model, the status code and the trace
  identifier, to the view. **Its status handling is a clamp rather than a pass-through, and that is what
  makes the direct request safe**: the status arrives as the **third route segment**, bound to `id` under
  the single conventional route because [05](05-aspnet-core-migration-approach.md) re-executes at
  `/Home/Error/{0}` — a path segment, never a query string — so the value is public input, and anything
  outside 400-599 is normalized to 500 while every code the framework itself supplies passes through
  unchanged. The
  **layoutless generic error view** completes it: `Layout = null` is load-bearing rather than stylistic,
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

**Band 69.5 / 122.5 / 208.5. Medium confidence, unchanged** — the scope is enumerated to the file, the
blocker, the case, the artifact mode, the fixture control and now the engine, and the variance is
execution rather than discovery. The
sub-row with the widest relative variance is the artifact, because it must be built to a **contract with
refusal semantics** rather than merely made to work.

**Re-derived from 68 / 119.5 / 204.5, and this round the case row moves alone.**
[05 §12.4](05-aspnet-core-migration-approach.md) resolved the matrix from 147 + 160 to **152 + 174**, so
the re-pointed half goes 6 / 9 / 15 → **6.5 / 9.5 / 15.5** and the target-only half
16 / 24.5 / 41 → **17 / 27 / 44.5**, taking the row from 22 / 33.5 / 56 to **23.5 / 36.5 / 60** —
`+1.5 / +3 / +4`, and still the largest single sub-row in this model. So 68 + 1.5 = **69.5**,
119.5 + 3 = **122.5** and 204.5 + 4 = **208.5**. **The other twelve sub-rows are unchanged**, including
the machinery row, whose continuity citation moves from 8 cases to 10 without moving its band because it
is priced on the **topology** those rows need rather than on their case count.

**The round before this one moved five components, and its arithmetic is printed too, because a reader
comparing bands is comparing two rounds.** It re-derived the row from 57.5 / 102 / 175.5:

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
- the **case** row followed the matrix from 144 + 95 to **147 + 160** at unchanged rates. Three cases
  arrived inside two existing rows — row 24's fifth and row 64's fourth and fifth, all target-only,
  taking it to 144 + 98 — and then the twenty-seven rows the owner's table then carried at ids 76–102 added
  **3** both-fixture cases and **62** target-only ones: 95 + 3 = **98**, 98 + 62 = **160**, and
  144 + 3 = **147**. So 15 / 23 / 38.5 becomes
  22 / 33.5 / 56 — **+7 / +10.5 / +17.5**, the largest single movement anywhere in this model;
- and the **artifact** row gains the **`--store` grammar** with its per-store receipt and the
  four **`dbo.__DataMigration*` run tables** written in the same transaction as each group's commit, with the
  filesystem receipt derived from those tables *after* commit and never read to decide a resume. A run record
  inside the transaction is what makes an interruption resumable rather than merely detectable, and it is
  what row 64's five cases assert. 5 / 9 / 16 becomes 6 / 11 / 19.5 — **+1 / +2 / +3.5**.

0.5 + 1.5 + 0.5 + 7 + 1 = **+10.5**, 1 + 3 + 1 + 10.5 + 2 = **+17.5**, and
1.5 + 5 + 1.5 + 17.5 + 3.5 = **+29**, so
57.5 + 10.5 = **68**, 102 + 17.5 = **119.5** and 175.5 + 29 = **204.5**. This row was previously
re-derived from 48.5 / 87.5 / 151.5, from 37.5 / 68.5 / 118.5, from 25.5 / 48 / 82, and before that from
21 / 40 / 69 by the third
view component and by the target-facing half arriving from W4 under
[03 §5](03-modernization-roadmap.md)'s split.

**The manual visual review is not inside this band, and it closes this workstream's exit gate.**
[03 §5](03-modernization-roadmap.md) requires the review against W4's captured baseline at W7's exit,
so W7 has not exited until it is signed off. It is sized as a non-code task in
[§7.1](#71-the-manual-visual-review) — 7 expected IED for the review half, of its 12.5 in all — and
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
`--store catalog|identity`** [05 §5.7](05-aspnet-core-migration-approach.md), so this workstream runs
`--store identity` throughout, receives its **own** receipt and its **own** reconciliation verdict, and is
**separately reversible** from [W9](#w9--domain-data-migration-tooling-rehearsed-against-a-copy) rather than entangled with it. That is
what makes the two workstreams' gates independently adjudicable, and it is the reason the sequencing in
[section 8.2](#82-concurrency-permitted-by-the-graph) can place them in different sets at all.

**Band 4 / 8 / 16, and the reason it is no larger is a placement rather than an
oversight.** [03 §5](03-modernization-roadmap.md) enforces this workstream's diff and reconciliation
gates through the console's data-migration verbs, and that artifact — including the `--store` grammar, the per-store
receipt and the run tables behind them — is **built and charged once, in
[W7](#w7--the-aspnet-core-port)** — 11 expected IED there. What this workstream carries is **running**
those steps against its own store and adjudicating its verdicts, which the band already contained as
reconciliation work; charging any part of the authoring here as well would double-count it. **Low confidence**, for a reason stated
precisely: this workstream's *source schema* is the one [12 §5](12-migration-blockers.md) qualifies as
**evidence rather than proof**. The high band is what this costs if **A10** is false and the hashes do not
validate. [R5](#r5--identity-migration-rollback) carries it.

#### W9 — Domain data migration tooling, rehearsed against a copy

**Scope** is [03 §5](03-modernization-roadmap.md)'s, entered through what that document calls the hard
gate: the generated-schema diff must pass before any data is loaded.

**Basis.** Six entities, loaded in dependency order by the same executable's `load-catalog` verb
[05 §5.6](05-aspnet-core-migration-approach.md), whose release-time placement is
[06 §6.10](06-azure-hosting-recommendations.md)'s — then reconciled to that section's
standard: keyed sets and per-row digests, with per-table row counts and per-order financial totals
necessary and **not** sufficient. **The rehearsal inside this band is domain-only**: it exercises
`load-catalog` and the catalog store's reconcile against a restored copy of the catalog source and
invokes `load-identity` at no point, and the **integrated both-store rehearsal is a gated task of
W13's pre-window** rather than work content here — [03 §4.2](03-modernization-roadmap.md) owns that
placement and states why an exit gate requiring a capability a later workstream builds was not
executable. **Building and proving the manifest mechanism is work content in this band; producing the production
manifest is not.** [03 §5](03-modernization-roadmap.md) has this workstream exit on **procedure and
rehearsal readiness**, while the cutover run — the **sealing step** in
[06 §6.10](06-azure-hosting-recommendations.md)'s order — produces the artifact that is actually
acted on, so what this band carries is **that step's mechanics**, its destination and its
rehearsal, not a second execution. 06's contract makes that artifact a **verifiable** one rather than a
list, and its five bound fields are [06 §11.4](06-azure-hosting-recommendations.md)'s to state: the
exact imported ID set bound to the **per-store source fingerprints**, the **durable run id**, the
**contributing committed table units** and the **per-store extraction cutoffs**, integrity-protected and
written to a store whose immutability the store itself enforces. **The unit of that provenance is the
committed table unit and not a batch identifier** — 06 §11.4 and
[05 §5.6](05-aspnet-core-migration-approach.md) bind it that way and reject batch provenance outright,
for a reason those sections state and this one does not restate. Most of what that costs is already
inside this band for another reason — the run id and the per-store source descriptors, the canonical
serialization and the keyed reconciliation sets are all required by the **procedure's own**
checkpoint and reconciliation contracts [05 §5.6](05-aspnet-core-migration-approach.md) — so the
incremental content is the manifest's own binding and the write-once destination's provisioning. That is
**absorbed rather than added: the band does not move**, and this row's band is the one stated below,
carried unchanged by [§5.1](#51-summary-table) and recorded by
[§6.1.1](#611-the-walk-from-the-previously-published-total) among the ten rows that did not move —
so 06's strengthened manifest contract causes **no band change anywhere in this model**. The high case
already carries the case where the destination and its immutability policy do not exist to be pointed at
([W10](#w10--hosting-provisioning-and-platform-configuration)
provisions them), and the low and expected cases assume they do. The volume is modest and the **gate** is
the cost: a diff between a generated migration and the real schema, iterated until it passes, against a
schema that W3 must first extract because the migration source **ships no schema script** and neither
committed copy of the other edition's script is usable [12 §5](12-migration-blockers.md).

**Four contract additions are inside this band, and they are named so the band is not read as covering
less than it does**: the parent run descriptor with its mandatory continuation input across every
step, the split of the catalog set into a pre-load stage and a post-transform stage, the
merge lineage the cleanup set is enumerated from, and the run-closure step that retires the verification
key last — all four owned by [05 §5.6](05-aspnet-core-migration-approach.md) and
[06 §11.4](06-azure-hosting-recommendations.md). Each is an increment to work this row already carries
rather than a new deliverable: one table and one required input, a second migration file, columns in an
artifact already written, and a closing sequence of already-specified steps. The band does not move because
its dominant cost is stated above as the diff gate iterated until it passes, not the procedure's surface
area — but if the extraction returns a `CartId` facet that forces the bound of
[05 §4.3](05-aspnet-core-migration-approach.md) to be re-approved, that is a **gate reopening** and this
row is re-estimated rather than stretched.

**Band 4 / 8 / 16. Low confidence.** [R4](#r4--domain-data-migration-rollback) carries the risk,
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

**Band 6 / 11 / 19**, of which **5 / 9 / 16** is the provisioning itself — the band this row carried before
the walk below was required, and the band `G-CSP-BROWSER` has always been inspected inside — and
**1 / 2 / 3** is the increment for the newly required **four-engine functional walk** of
[06 §10.4](06-azure-hosting-recommendations.md). The suite's **test** engine is not provisioned here
and is not charged here: the target-facing fixture creates and destroys its own disposable instance per
collection [05 §12.7](05-aspnet-core-migration-approach.md), which is why that cost sits in
[W7](#w7--the-aspnet-core-port)'s machinery sub-row. This workstream provisions the **deployment**
environment, and the two are separate resources with separate lifetimes and separate principals.
**Medium confidence** — the steps are enumerated by 06; the variance is environment-specific.

**The manual browser charge, and what it is priced as.** Both items of input 23's non-index pair are
[03 §5](03-modernization-roadmap.md)'s W10 exit conditions — condition 8, the **four-engine functional
walk** of the script-issued cart-removal flow that [06 §10.4](06-azure-hosting-recommendations.md)
**requires**, and condition 9, `G-CSP-BROWSER` — and both are performed at **gate 10b**, the first gate at
which a deployed application exists to be driven. **Only one of the two moves this band, and which one is
worth being exact about.** `G-CSP-BROWSER` was already an exit condition of this workstream when the row
stood at `5 / 9 / 16`, inspected inside the transport-and-header configuration that band covers, and it
carries no separately identified figure now; the **`1 / 2 / 3` is the walk's own increment**, and it is here
because the walk is newly **required** rather than newly discovered. Both are priced as **manual reviewer
time**: a reviewer opening the deployed non-traffic target in each of the four families, exercising one flow,
reading the network and console panels, and recording engine versions, checklist results and a signature.
**Nothing here is priced as harness execution, and that is the whole reason the charge is small.** There is **no pinned
browser-automation package anywhere in the target** — [04 §7.2](04-dotnet8-migration-strategy.md) pins four
test packages and none of them drives a browser, and adopting one would need an amendment to AAP §0.5.2
([06 §10.4](06-azure-hosting-recommendations.md)) — so no toolchain acquisition, no engine-binary pin, no
selector maintenance and no flake budget is in this band. Three costs are **not** in it either: the manual
**appearance** review, which is [§7.1](#71-the-manual-visual-review)'s own additive row and whose capture
manifest this charge does not touch; the HTTP-level assertions for the removal endpoint and the report
endpoint, which are numbered rows of input 14 charged in [W7](#w7--the-aspnet-core-port); and any
re-execution of `G-CSP-BROWSER` on a later policy change, which is a change-driven run rather than
programme scope. **The walk is run with the enforcing policy live**, which is
[03 §5](03-modernization-roadmap.md)'s condition 8 rather than a scheduling preference: a flow walked under
the report-only header would not exercise the configuration that ships, and a request the enforced policy
blocks would pass such a walk and fail in production. That is also what keeps the low case cheap —
`G-CSP-BROWSER`'s enforcing run and this walk observe the same four families under the same header, so one
reviewer can take both in one sitting. The band's width is a reviewer-availability and defect-discovery
range — the low case is four clean families in that single sitting, the high case is a defect found in one
family and re-walked in all four after the fix.

#### W11 — CI provider selection, then pipeline authoring

**Scope** is [03 §5](03-modernization-roadmap.md)'s, strictly in that order: a provider decision, then
a manifest.

**Basis — net-new, with an approval inside it.** The repository has **no** pipeline definition, publish
profile or container manifest [08 §7.2](08-technical-debt-register.md), so nothing is migrated. The
band covers the provider decision (an approval, not engineering), then build, test, publish and the
deployment-time migration step that [06 §6.2](06-azure-hosting-recommendations.md) requires be run
under a principal distinct from the runtime identity — the step that invokes the console's data-migration verbs, whose
own cost is charged once in [W7](#w7--the-aspnet-core-port).

**The test stage is not one stage.** The two-platform split of
[05 §12.10](05-aspnet-core-migration-approach.md) puts
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

- **The thirteen blocking checks now carry literal evidence bounds**
  ([06 §12.1](06-azure-hosting-recommendations.md), entry count) — a per-request timeout,
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
- **The target-test stage acquires a browser prerequisite**
  ([05 §12.11](05-aspnet-core-migration-approach.md)): the pinned harness's Chromium
  install, expressed as a step on the Linux agent and pinned with the library
  [04 §7.7](04-dotnet8-migration-strategy.md). **+0.5 / +0.5 / +1.**

**Three further obligations landed on the manifest half in the round before this one, and each is
authoring rather than intention — which is the whole of why the band moved then and does not move now.**

- **Instance observation became a published join protocol** rather than a stated capability
  [06 §12.1](06-azure-hosting-recommendations.md). The stage constructs a **`traceparent` per request** so
  the response it received and the telemetry it later reads are the *same* request; it runs a **published
  query** joining that trace to the role-instance dimension; it **disables sampling** on the non-traffic
  target and then **proves it off** with an `itemCount` assertion, because a sampled-away request looks
  exactly like a request served by one instance; and it validates the instance identity against the
  platform's **own inventory**, so "two instances" is a fact read from the platform rather than inferred
  from two different strings. A query, an assertion and a platform read are three authored things.
  **+1 / +2 / +3.5.**
- **The thirteen checks acquired an executor-and-stage mapping**
  ([06 §12.1](06-azure-hosting-recommendations.md), entry count): **2** of them are the suite's
  own `Category=Deployed` binding, invoked by the stage — which is where **row 47** executes, since no
  deployed host exists at [W7](#w7--the-aspnet-core-port)'s exit — and the other **11** are the stage's
  own steps. Expressing that split, rather than leaving a check without a named executor, is a manifest
  obligation. **+0.5 / +1 / +1.5.**
- **Artifact publication became conditional and allow-listed.** The stage publishes **three** artifacts
  and no others, and the baseline publication policy of
  [05 §12.10](05-aspnet-core-migration-approach.md) is a **pipeline condition** rather than a
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
[05 §12.10](05-aspnet-core-migration-approach.md)'s second agent and the handoff artifact between them.

#### W12 — Administrator provisioning tool

**Scope** is [03 §5](03-modernization-roadmap.md)'s, exiting with all five properties of
[05 §10.2](05-aspnet-core-migration-approach.md) demonstrated.

**Basis: input 30 for the tool itself, inputs 23 and 32 for its added tests.** The three are disjoint —
input 30
counts behaviour to implement, input 23 counts the ten operator rows `O1` to `O10`, which have no MVC 5
baseline, and
input 32 counts the published-console rows `O1`–`O10` that sit outside input 14's matrix — and
together they cover this row rather than only its test half. A small console project, where the effort is in
input 30's **5 required properties** rather than the code: a secret
channel that keeps the credential out of process listings and shell history, hashing through the
framework's own user manager rather than direct SQL, **convergence checked per operation over all four
operations** — `create-user`, `create-role`, `add-membership` and `run`, the closed set of
[05 §10.2](05-aspnet-core-migration-approach.md) property 4 rather than the withdrawn *"role, user,
credential and membership"* naming [§4.1](#41-the-estimation-basis-every-input-with-its-method) records
at the input — so a prior partial run is repaired rather than
skipped, an audit record with no secret in it, and exclusion from the deployed web
application. Input 30's remaining dimension is what makes the band Medium rather than Low:
`PROV-6001`'s closed vocabulary of **18 outcome values** has to be produced and asserted
exactly — over the **fifteen that are exercisable** — rather than approximated. **The other three are
reserved, and pricing an assertion for them would price an impossible test**:
[09 §6.8.1](09-security-assessment.md) reserves `MembershipRevoked`, `MembershipNotHeld` and
`Failed_UserNotFound` with the privilege-withdrawal class [09 §6.8.1.1](09-security-assessment.md) owns,
each presupposing a removal or a resolve-without-create that no operation of this tool performs, and that
section's own acceptance criteria are written against the exercisable set rather than demanding a test for
a branch no invocation can enter. So what this band carries is **producing and asserting fifteen values
and reserving three** — the reservation being a defined record shape a future producer inherits, which is
a line of the outcome enum and not a test.

**Input 30 carried a second such dimension and no longer does** — a revoke
mode's two operations across three branches, withdrawn where the input is stated
([§4.1](#41-the-estimation-basis-every-input-with-its-method)) because
[05 §10.2](05-aspnet-core-migration-approach.md) specifies no such mode. The withdrawal costs this row
nothing, for the reason the note below the component derivation gives.

> **What the corrected outcome count does and does not add, because most of it was already inside this
> band.** An earlier version of input 30 put the vocabulary at eight values — three credential outcomes
> and five failure outcomes — which undercounted [09 §6.8.1](09-security-assessment.md)'s closed set by
> **ten**. The correction is a completeness fix to the *input row*, and it is mostly **not** new work: the
> **twelve** non-failure values partition across the four operation records as `4 + 2 + 4 + 1` plus the
> `NotAttempted` the three operation records share and which is counted once — `4 + 2 + 4 + 1 + 1 = 12`,
> the derivation [§4.1](#41-the-estimation-basis-every-input-with-its-method) prints at the input — and
> producing them per operation is exactly the "convergence checked per operation over all four operations" this basis
> already prices. Enumerating them names the work rather than enlarging it.
>
> **One value is genuinely new scope, and it is priced.** The sixth failure outcome,
> `Failed_ArgumentRejected`, arrives with [09 §6.8.1](09-security-assessment.md)'s rejection contract,
> which did not exist when this band was first judged. It is more than one enum arm: it requires a
> **fixed validation order**, so that which field is reported is not an implementation choice; **bounded
> sentinels** for the actor and target on a rejection, where the supplied value must never be echoed; the
> **same four-record cardinality every invocation emits** on a rejection branch too — three
> `NotAttempted` operation records and one `Failed_ArgumentRejected` run record, exactly one of which
> names the first failing field ([09 §6.8.1](09-security-assessment.md) rule 1); and negative tests over control characters and
> line breaks. That is the same class of work as
> [04 §12.4](04-dotnet8-migration-strategy.md)'s refusal enum, and it lands here because the tool is
> where it executes.
>
> **Band raised by this correction from 3 / 5 / 9.5 to 3 / 5.5 / 10.5 — an intermediate figure, which the
> two later obligations below then move again to 3.5 / 7.5 / 12.5, and input 32 to the **5 / 9.5 / 16** this row carries.** Expected moves by **0.5** for the rejection
> contract; high by **1**, because its characteristic failure is discovering mid-implementation that a
> sentinel collides with a legitimate value and the closed field set must be reopened. **Low does not
> move** — it assumes the contract lands as the thin pre-host validation layer
> [04 §12.4](04-dotnet8-migration-strategy.md) specifies. The non-failure values the eight-value form left
> unenumerated — the **twelve** less its three credential outcomes — contribute **nothing** to the
> increase, for the reason above: pricing them here would
> double-count the four operations that produce them.

One property carries a platform obligation rather than a code one:
[06 §9.5](06-azure-hosting-recommendations.md) establishes that a standalone console process is **not**
collected by the application's telemetry path, so the sanctioned execution path is a release-pipeline
job whose captured `stdout` is **retained in the release pipeline's run record** — row 5 of that section's
per-producer destination-and-retention matrix, and there is no export hop and no second store. The release
step asserts the **exact record set** rather than the presence of
output: four `PROV-6001` records on a provisioning run, one per operation, **invariantly and on every
path including the paths that fail**, which is what makes the assertion a constant rather than a
branch-sensitive count ([05 §10.2](05-aspnet-core-migration-approach.md) property 4); plus the
paired `AUTHZ-3001` where a membership is actually added. It then
confirms the selected record present in the run record, and fails on a non-zero exit. That is inside this band
and is why the row depends on W11.

**What the credential operation converges on, because the alternative reading would enlarge this row and
change the product's behaviour.** [06 §9.5](06-azure-hosting-recommendations.md) owns the invocation
policy and [09 §6.8.1](09-security-assessment.md) the outcome vocabulary: the operation records
**`UserCreated`** where the account is absent and is created with its credential, **`Created`** where the
account resolves but holds **no** password at all and one is set, **`AlreadyPresent_NotRotated`** where
it exists with one and the run did not request a rotation, and **`Rotated`** where the run passed
`--rotate-credential`. **The first two are two values and not one.** That section partitions the
`create-user` record's outcome four ways, and an earlier form of this paragraph collapsed the absent
account and the passwordless one into a single `Created` — the one reading under which a suite cannot
tell an account the run created from an account it repaired, which is exactly the distinction the
recoverability assertion below turns on. **Ordinary pipeline releases do not pass that flag**; the two
occasions that do are
the published-credential repair and a post-incident rotation, and
[03 §5 W12](03-modernization-roadmap.md) condition 1 is where the **four** outcomes are gated. **The four
are four readings of one field and not a fourth operation**:
[05 §10.2](05-aspnet-core-migration-approach.md) property 4 gives the credential no record of its own, so
the verdict is the `create-user` record's own outcome. This row is
therefore **not** estimated against a command that rewrites the credential on every run — that reading
would add a mutation AAP §0.3.2's idempotence requirement never asks for, rotate the administrator's
password on the release cadence, and sign its sessions out unpredictably. The estimation consequence is
small and worth stating: the operation is a branch and a comparison rather than an unconditional write, so
the band carries **four outcome paths and the flag that selects between them**, plus the no-flag re-run
condition 1 requires as evidence that the second run changed nothing. **The fourth path names the work
rather than enlarging it**: the two create arms are the two arms of the one create-or-add-password call
the first of the two additions below already prices, so splitting them adds an outcome value and an
assertion of it, not a branch nobody had written.

**Two additions since the band was set, both inside it.** The credential operation of
[05 §10.2](05-aspnet-core-migration-approach.md) property 3 is a fourth branch on a code path that
already resolves the same `UserManager` — a create-or-add-password call, a rotation call reached only
under the flag, and a published-value refusal. The second is the **guarded catalog seed**:
[04 §12.6](04-dotnet8-migration-strategy.md) settles that
[05 §5.4](05-aspnet-core-migration-approach.md)'s opt-in seed command executes as the **second verb
(`seed`) of this same project** rather than as a fifth project, so what this row acquires is one more entry point over a
host it is already composing — the configuration surface, the exit-code convention and the single
structured logging provider are this row's regardless of how many verbs sit on them. **The seed's own
cost is not here and is not double-counted:** the routine and the policy behind its three checks are sized
in **W7**'s "seed data" sub-row, and this row adds the verb that invokes them. Neither adds a
package or a platform obligation, and the **6 IED** high case of the tool-and-wiring band below already
carries the secret-channel and
audit work that dominates it. That band is therefore unchanged rather than silently stretched, and
this paragraph is the statement of why — [03 §5 W12](03-modernization-roadmap.md) conditions 1, 1a and 3
gate the first, and [04 §12.6](04-dotnet8-migration-strategy.md)'s two required assertions gate the
second.

**And the seed entry point adds no dependency edge either**, which is worth stating because a seed command sitting
in a late workstream would be a problem if anything earlier needed a seeded database. Nothing does: W4's
target-side fixture **loads a fixture dataset** carrying the same catalog rows
([05 §12.3](05-aspnet-core-migration-approach.md)) rather than invoking the seed, and W8's and W9's
rehearsals run against **restored copies** of real data. The verb's first use is its own acceptance.

**One of them does add a dependency, and it is a sequencing one rather than a cost.** The credential
operation is *accepted* against the account W8's Identity rehearsal neutralized, so
[03 §5 W12](03-modernization-roadmap.md) puts **W8 in this row's entry gate** and retains W8's rehearsal
copy until this row exits. Nothing here gets larger — the account and the copy are W8's output, and this
row runs one command against them — but the row cannot start beside W8, which is why
[§8.2](#82-concurrency-permitted-by-the-graph) places it in its own set and
[§8.3](#83-the-critical-path-and-what-to-do-first-if-the-goal-is-to-narrow-the-estimate) counts its
expected IED on the chain — 9.5 of them, once the tests, the deployed census, the fourth gated
invocation and input 32's published-console rows below are included.

**A third such addition was carried above and is withdrawn, which is why that paragraph now names two.**
It was *"the revoke mode of property 3a"*, described as a mode of the same provisioning verb over the same
two managers, with the same actor attribution — the invocation-scoped `MUSICSTORE_AUDIT_ACTOR`
environment variable, not a switch — and the same record and exit-code contract.
[05 §10.2](05-aspnet-core-migration-approach.md) defines no property 3a and its verb set contains no
revoke verb, so the addition described a capability no deliverable specifies —
[§4.1](#41-the-estimation-basis-every-input-with-its-method) records the withdrawal at the input, and
[09 §6.8.1.1](09-security-assessment.md) owns the privilege-withdrawal gap it leaves open, with **no
workstream carrying it**. **The withdrawal removes no IED from this row**, for the same reason the two
surviving additions add none: it was recorded as an addition *inside* an unchanged band and never became a
term of one. The component derivation below is the check — the five components are the tool and its
wiring, the rejection contract, exit condition 7's census with the fourth gated invocation, input 23 and
input 32, and **not one of the five is a revoke term** — so this row is **5 / 9.5 / 16** before the
withdrawal and **5 / 9.5 / 16** after it, and no figure that quotes it moves: not
[§5.1](#51-summary-table)'s row, not [§6.1](#61-the-totals)'s total, not
[§6.1.1](#611-the-walk-from-the-previously-published-total)'s walk line, not
[§8.2](#82-concurrency-permitted-by-the-graph)'s set 10 and not
[§8.3](#83-the-critical-path-and-what-to-do-first-if-the-goal-is-to-narrow-the-estimate)'s chain.

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

**And the coverage rows this gate executes cost nothing here**, because their authoring is W4's.
[03 §4.3](03-modernization-roadmap.md)'s ownership map is the single place they are enumerated and this row
cites it rather than restating the list, which is why no row number appears here: what it assigns to this
workstream's acceptance is the seeding guard through the `seed` verb and the administrator-provisioning
row's per-operation cases, together with the **four-invocation** process census
— and running an already-authored row at the gate that owns its runtime is a gate condition rather
than a second cost, exactly as it is for W7, W8, W9 and W10.

**Two obligations do move this band, and 5.5 no longer holds.** Both arrive from
[03 §5 W12](03-modernization-roadmap.md) after the rejection contract had already been priced, and the
question this paragraph answers is the one asked directly: does the expected figure of 5.5 survive them.
It does not.

- **Exit condition 7 is a deployed twelve-class security-event census, and it is not a re-run of anything.**
  The **twelve** application event classes that
  [W10](#w10--hosting-provisioning-and-platform-configuration) does not demonstrate are demonstrated here,
  **against a deployed environment**, driven from **the named fixture population**
  [03 §5 W12](03-modernization-roadmap.md) fixes — the seeded non-production catalog **the web
  application's own `seed` entry point** loads, plus **two synthetic accounts, one administrator
  and one ordinary**, both created through the sanctioned paths — driven through sign-in, lockout,
  registration, password change, authorization denial, the three administration outcomes and the two order
  outcomes, and then **removed by the deletion operation [W16](#w16--personal-data-governance) stage 2
  proved**, with the removal asserted rather than assumed. The thirteenth class is not here: it is the
  canary W10's condition 7 already used to prove the sink. This is the only workstream with an executable
  that can drive a deployed environment from outside it, which is why the census lands here, and the
  driving, the collection assertions and the asserted cleanup are work in this row rather than a repetition
  of W7's in-suite emission tests. **0.5 / 1 / 1.5.**
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

**A second addition is outside the original band, and it is the reason this row is rebanded again: the ten
published-console rows of input 32.** [05 §12.4.1](05-aspnet-core-migration-approach.md) requires
`O1`-`O10` — parser closure, startup validation running before anything else on every verb, the
environment assertion, all four verbs doing their work, partial-run idempotence per operation, the batch
and resume bounds, exactly **four** audit records per invocation in the pinned operation order on all
four paths, the secret appearing in no
part of any capture, the literal exit-code allocation, and a normal builder's inability to select the
test-only connection mode. They are **lettered rather than numbered**: they sit outside that document's
104 numbered rows and outside its 326-case total, so neither input 14 nor input 23 counts them, and
[§4.1](#41-the-estimation-basis-every-input-with-its-method)'s inclusion rule places them here — this is
the workstream that builds the executable they start as a process, and the only one whose gate can
demonstrate them.

**Eight of the ten are new work and two are not, and the two are named rather than netted off silently.**
`O1` is the same assertion as the fourth of input 23's five — the dispatcher admitting exactly the
documented command lines and refusing every other class — and `O8` is the same assertion as the second
half of its fifth, the credential appearing in no captured output. Each enumerates its counterpart: `O1`
names seven refusal forms, `O8` adds the connection string and one further variable as needles. That
enumeration **names the work rather than enlarging it**, which is the same reading this row already applies
to the twelve non-failure outcome values above, so charging either as a new test would double-count a test
input 23 has already paid for. **The increment is therefore eight rows at the same per-assertion rate the
five above use** — `8 × 0.15 / 0.22 / 0.4 = 1.2 / 1.76 / 3.2`, which under
[§4.2.1](#421-the-rounding-rule-stated-once) is **1.5 / 2 / 3.5** — and it carries **no second harness**:
the process-level launch, stream capture, scoped environment variable and temporary working directory
priced at `0.5 / 1 / 1.5` in input 23's component are exactly what these rows run on, and this row is
still their only consumer.

**Band 5 / 9.5 / 16. Medium confidence**, and derived as five components so the move is checkable:
`1.5 / 3 / 6` for the tool and its wiring, plus `0 / 0.5 / 1` for the rejection contract, plus
`0.5 / 1.5 / 2` for exit condition 7's deployed census **and** the fourth separately gated invocation,
plus the `1.5 / 2.5 / 3.5` of input 23 above, plus the `1.5 / 2 / 3.5` of input 32.
`1.5 + 0 + 0.5 + 1.5 + 1.5 = 5`, `3 + 0.5 + 1.5 + 2.5 + 2 = 9.5`,
`6 + 1 + 2 + 3.5 + 3.5 = 16`.

**The third component is the sum of both obligations above, and it is stated that way because an earlier
form of it was not.** It carried only the census — `0.5 / 1 / 1.5` — while its label claimed the invocation
census as well, so the `+0 / +0.5 / +0.5` priced for the fourth run reached this row's prose and never
reached its band. Added column by column: `0.5 + 0 = 0.5`, `1 + 0.5 = 1.5`, `1.5 + 0.5 = 2`. That is the
whole of the correction, and every figure elsewhere that quotes this row moves with it rather than in some
of its places — which is the failure mode this row's own
history demonstrated: an increment that reached the band and never reached the total, found by **summing**
the column rather than by reading it.

**Stated against the three figures this row has previously carried**, because all three appear in earlier
reconciliation records: the band was 3 / 5 / 9.5 before the rejection contract, 3 / 5.5 / 10.5 with it,
3.5 / 7.5 / 12.5 with the two obligations above, and
it is **5 / 9.5 / 16** now. The move from 5.5 to 7.5 had **three** parts, none of them a re-judgement of
work already priced: **0.5** is input 23's operator-host increment re-derived under
[§4.2.1](#421-the-rounding-rule-stated-once), **1** is exit condition 7's deployed twelve-class census, and
**0.5** is the fourth separately gated invocation. **The move from 7.5 to 9.5 has one part and no
re-judgement at all**: the **2** expected IED of input 32's eight new published-console rows, derived
above at the per-assertion rate input 23's five already use.

#### W13 — Cutover

**Scope** is [03 §5](03-modernization-roadmap.md)'s; the runbook is owned by
[06 §11](06-azure-hosting-recommendations.md) and the approach and its accepted losses by
[05 §11](05-aspnet-core-migration-approach.md). **Neither is reopened here.**

**Basis.** Executing a runbook, with the rollback position confirmed beforehand: drain and record the
final write cutoff, provision, **run the production data movement** — `extract-schema`, `load-catalog`,
`load-identity`, `reconcile`, `seal-manifest` in [06 §6.10](06-azure-hosting-recommendations.md)'s
order — verify, then admit traffic on the recorded go decision. The production run sits here because
[03 §5](03-modernization-roadmap.md) has W8 and W9 exit on **readiness** and this workstream perform the
run itself; **that placement does not move this band**, because the procedure is authored, iterated and
rehearsed inside W8's and W9's bands, and what remains here is performing it and reading its verdict rows. The
band covers rehearsal as well as execution, because a runbook first executed in production is not a
runbook. Six workstreams must have exited before this begins, which is why
[section 8](#8-sequencing) treats it as the convergence point rather than a step.

**One named task entered this workstream's pre-window, and the band is re-examined against it rather
than assumed to absorb it.** [03 §4.2](03-modernization-roadmap.md) places the **integrated both-store
rehearsal** — both `load-catalog` and `load-identity`, in the release's own order, against restored
copies of both source stores, with the combined reconciliation — as a **gated task of this
workstream's pre-window**, entered on W8 and W9 having both exited and satisfying **no exit condition
of this workstream** [03 §5](03-modernization-roadmap.md) W13. **It is not new content in the model,
and that is why the band holds rather than because the band is generous.** It is the same run
this band already covers twice — once as the rehearsal the sentence above insists on, once as the
production run — over two loads that both workstreams' exit gates have already authored and
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
verification population is the **15** classes of input 21's 16 that have a producer — the sixteenth is
verified nowhere because nothing emits it ([09 §6.8.1.1](09-security-assessment.md)). The cost is not the field count — it is that
three of the six conditions are **decisions requiring a named approver** and three are **engineering with a
verification obligation**:

| Stage | Content | Low | Expected | High |
| --- | --- | ---: | ---: | ---: |
| **1** Policy — conditions 1–3 | Retention per data class; handling rules for non-production copies of real personal data; a legal-hold process that suspends deletion. Three approver constituencies: the data owner, security and legal. Depends on W1 alone, so it is available in the widest concurrency set | 1 | 2.5 | 6 |
| **2** Mechanism — conditions 4–6 | A deletion or anonymization operation demonstrated against **synthetic** data from the release path; the backup-propagation window defined and verified against [06 §6.7](06-azure-hosting-recommendations.md)'s retention; access auditing live and *observed arriving* in the sink [06 §9.2](06-azure-hosting-recommendations.md) defines — these records are row 2 of [06 §9.5](06-azure-hosting-recommendations.md)'s matrix, whose destination is that same sink held at audit retention on the workspace table, so the obligation includes their retention there and not the emission alone — in every environment holding real personal data. Depends on stage 1, W3, W7, W11 and **W10** | 2 | 3.5 | 6 |
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

**Two facts about stage 2's scope that a reader could otherwise mis-size.** First, stage 2 proves the
deletion or anonymization mechanism **against synthetic data** from the release path, with **liveness over
real data asserted by W8, W9 and W13** as their own entry and exit conditions
([03 §5 W16](03-modernization-roadmap.md)). That places *where the real data is touched* outside this row
without removing an artifact from it: the operation, the measured backup window and the observed audit
trail are this stage's three deliverables either way. Second, the stage carries two obligations that are
easy to read as belonging elsewhere — condition 6's access auditing must be live in the **rehearsal**
environment as well as the production one, which is the same operation applied a second time from the same
release path, and the **personal-data read** through the governed access path is this stage's rather than
[W10](#w10--hosting-provisioning-and-platform-configuration)'s, because the audited path and its mechanism
are what this stage builds. The stage's **2 / 3.5 / 6** covers both: its high of 6 against an expected 3.5
is a second environment and a governed read, not a contingency.

**Band 3 / 6 / 12. Low confidence**, for the same reason as W1's width: the band is set by how many
constituencies must convene and whether the organization already has retention policy to inherit, neither
of which is knowable from the repository. [R15](#r15--personal-data-governance-is-unowned) carries it, and
[section 9.4](#94-the-six-risks-that-are-approval-decisions-not-mitigations) lists it as an approval
decision rather than an engineering risk.

---

## 6. Totals, and where the effort actually lives

### 6.1 The totals

| | Low | Expected | High |
| --- | ---: | ---: | ---: |
| The sixteen workstreams ([§5.1](#51-summary-table)) | 154.5 | 278.5 | 489 IED |
| The manual visual review ([§7.1](#71-the-manual-visual-review)) | 6.5 | 12.5 | 22.5 IED |
| The manual accessibility review ([§7.3](#73-the-manual-accessibility-review)) | 2.5 | 4.5 | 8 IED |
| The approved-delta sign-offs ([§7.2](#72-the-approved-delta-sign-offs)) | 8 | 16 | 26 IED |
| **Total** | **171.5** | **311.5** | **545.5** IED |
| *For reference, excluded:* the conditional pre-admission affinity retirement | 1 | 2 | 4 IED |

**The approved-delta sign-offs are a line of their own here, and a previous revision of this section said
the opposite.** It said they were 2 / 4 / 8 *inside* W1's band and therefore already counted.
[§7.2](#72-the-approved-delta-sign-offs) owns that band, derives it as **8 / 16 / 26**, and states
plainly that it is **not** inside W1's — W1's 3.5 / 7 / 13.5 is
[eleven approval acts and nothing else](#w1--approval-of-this-assessment). The two partition input 17
between them, `7 + 16 = 23` expected IED is the whole approval activity, and counting the sign-offs
once, here, is what makes the column add up.

The high band is about **3.2 times** the low band (`545.5 ÷ 171.5 = 3.18`). That spread is not estimator
hedging — it is the arithmetic consequence of five low-confidence rows worth **93.5 expected IED**
(`W3 4 + W4 67.5 + W8 8 + W9 8 + W16 6`) whose difficulty depends on a schema that has not been extracted,
on behaviour that has never been asserted by a test, and on an approval nobody has been asked for yet
([section 4.4](#44-confidence-and-its-reason)), plus
one further row — W2's — whose *outcome* is unknown even though its tasks are enumerated. **Those five
rows are just under a third of the expected total** (`93.5 ÷ 311.5 = 30.0%`), against a quarter when this
document's first derivation was written, and most of that shift is W4: the ratio is stable while the
low-confidence share is now falling slightly, because the rows that grew most this round are enumerated far
more tightly than they once were without being any better calibrated, and because this round's growth
landed mostly in Medium rows.

#### 6.1.1 The walk from the previously published total

**Nine rows account for the movement that produced the figures above — six whose bands were re-derived
and three that were already in [§5.1](#51-summary-table)'s table and had never reached its total.** Two of
the six — W4 and W7 — were re-derived twice, across two successive rounds of findings, and their rows
below carry both steps rather than only the later one. The
walk is stated here, once and in full, because
[W4](#w4--build-governance-bootstrap-pre-port-behavioural-baseline-and-test-suite)'s and [W7](#w7--the-aspnet-core-port)'s own
records point at this section for it rather than restating it — and because a total that moves without a
walk is a total nobody can check.

**First the defect, because the base of the walk depends on it.** The previously published total was
`133.5 / 239.5 / 424`, and that figure is exactly the sum of **sixteen** rows: W1 to W15 at
`131 / 235 / 416.5`, plus the manual visual review at the `2.5 / 4.5 / 7.5` it then carried. The table
above it displayed **three more** — W16, the manual accessibility review, and the approved-delta
sign-offs — and **none of the three reached the total**. Two of them were simply omitted from the
addition; the third was excluded on purpose, by a convention that called it effort inside W1, which
[§7.2](#72-the-approved-delta-sign-offs) contradicts. This is the same class of failure as the one W4's
"all six places or none" rule exists to prevent, and it was found the same way — by **summing** the column
rather than reading its last row. The walk therefore corrects the base before it re-bands anything.

| Row | From | To | Delta (L / E / H) | What moved it |
| --- | --- | --- | --- | --- |
| W16 | *displayed, not in the total* | 3 / 6 / 12 | +3 / +6 / +12 | Nothing about the band: [W16's own record](#w16--personal-data-governance) states it is **unchanged at 3 / 6 / 12**, and [§5](#5-effort-by-workstream)'s introduction records the workstream as new and sized here for the first time. The whole of this movement is the row's arrival **in the total** |
| Manual accessibility review | *displayed, not in the total* | 2.5 / 4.5 / 8 | +2.5 / +4.5 / +8 | The same: [§7.3](#73-the-manual-accessibility-review) states the band is **unchanged** by the page surface and the two requirements added to its checklist. Its arrival in the total is the movement |
| Approved-delta sign-offs | *counted as 2 / 4 / 8 inside W1* | 8 / 16 / 26 | +8 / +16 / +26 | The full band arrives, because **none** of it was in the previous total: W1's 3.5 / 7 / 13.5 is [eleven approval acts and nothing else](#w1--approval-of-this-assessment), so treating this row as a component of W1 counted it **zero** times rather than once. The band itself also rose twice over and has now fallen half a day in every column: [§7.2](#72-the-approved-delta-sign-offs) prices [05 §11.7](05-aspnet-core-migration-approach.md)'s **15** approval-owned additions beside every row of the approved-delta register, which no row of the model had paid for, and it reads that register **at its present size** rather than at the smaller reading it once had — a size that has since fallen by one owner-carrying row, which is the whole of the fall. The delta above is the band that arrives, not the movement inside it; the priced decision count is stated once, in [§7.2](#72-the-approved-delta-sign-offs), and this table does not restate it |
| W1 | 3 / 6 / 12 | 3.5 / 7 / 13.5 | +0.5 / +1 / +1.5 | One act joined its census: [R18](#r18--the-browser-executed-half-of-the-scripted-cart-flow-one-pinned-engine-and-an-open-decision-on-the-other-two)'s recorded outcome, which [§9.2](#92-register-index)'s register row assigns to W1 and which no other row of this model prices. Eleven acts at `0.3 / 0.6 / 1.2` give `3.3 / 6.6 / 13.2`, which [§4.2.1](#421-the-rounding-rule-stated-once) rounds up. [W1's own record](#w1--approval-of-this-assessment) walks all three corrections to the census |
| W4 | 35.5 / 60.5 / 102.5 | 38.5 / 67.5 / 114 | +3 / +7 / +11.5 | **Two rounds, in one row.** In the first the suite moved from **75** rows and 239 cases to **102** rows and **307** cases, and from 383 fixture executions to **454**, taking the both-fixture cases this gate authors from 144 to **147**; the test topology became declarable, with a collection definition per surface group per assembly; the `IStoreSetup` write API and five further observer projections arrived; and the fixture inputs became **twelve committed files** — `+2.5 / +6 / +10`, to 38 / 66.5 / 112.5. In the second the matrix reached **104** rows and **326** cases, taking those both-fixture cases from 147 to **152** and the case component alone from 14.5 / 22.5 / 37.5 to **15 / 23.5 / 39** — `+0.5 / +1 / +1.5`, and nothing else in the workstream. [W4's own record](#w4--build-governance-bootstrap-pre-port-behavioural-baseline-and-test-suite) derives each component separately |
| W7 | 57.5 / 102 / 175.5 | 69.5 / 122.5 / 208.5 | +12 / +20.5 / +33 | **Two rounds, in one row.** In the first, five components moved and nothing else did — the shared application-services seam on the composition sub-row, the exhaustive instrumentation and `DENY`-based fault injection on the machinery sub-row, the browser flow's own real-Kestrel host, the case row following the matrix from 144 + 95 to **147 + 160**, and the migration artifact's `--store` grammar with its run-table writes — `+10.5 / +17.5 / +29`, to 68 / 119.5 / 204.5. In the second the case row alone moved, following 147 + 160 to **152 + 174**, from 22 / 33.5 / 56 to **23.5 / 36.5 / 60** — `+1.5 / +3 / +4`. [W7's own record](#w7--the-aspnet-core-port) prints both additions |
| W11 | 9 / 15.5 / 29 | 11 / 19.5 / 35.5 | +2 / +4 / +6.5 | The manifest half alone: the deployed gate's telemetry join protocol, the executor-and-stage mapping for its thirteen checks, and conditional allow-listed artifact publication. The provider half does not move — [W11's own record](#w11--ci-provider-selection-then-pipeline-authoring) shows the split |
| W12 | 2 / 4.5 / 8 | 5 / 9.5 / 16 | +3 / +5 / +8 | The rejection contract, exit condition 7's deployed census, the **fourth** separately gated invocation, and input 32's eight new published-console rows. [W12's own record](#w12--administrator-provisioning-tool) derives the band in five components and records the two intermediate figures it passed through, one of which was the failure this section documents in reverse: an increment that reached the row's prose and not its band |
| Manual visual review | 2.5 / 4.5 / 7.5 | 6.5 / 12.5 / 22.5 | +4 / +8 / +15 | A semantic capture census replacing a filename count, and the browser dimension the sizing had no term for. [§7.1](#71-the-manual-visual-review) derives it as a capture half of 2.5 / 5.5 / 11 and a review half of 4 / 7 / 11.5 |
| **Sum of the three membership rows** | | | **+13.5 / +26.5 / +46** | `3 + 2.5 + 8`, `6 + 4.5 + 16`, `12 + 8 + 26` |
| **Sum of the six re-derived rows** | | | **+24.5 / +45.5 / +75.5** | `0.5 + 3 + 12 + 2 + 3 + 4`, `1 + 7 + 20.5 + 4 + 5 + 8`, `1.5 + 11.5 + 33 + 6.5 + 8 + 15` |
| **Sum of the deltas** | | | **+38 / +72 / +121.5** | `13.5 + 24.5`, `26.5 + 45.5`, `46 + 75.5` |

`133.5 + 38 = 171.5`, `239.5 + 72 = 311.5`, `424 + 121.5 = 545.5`. Taken in two steps instead, the
membership correction gives the corrected base `147 / 266 / 470` and the six re-derivations take it
from there — `147 + 24.5 = 171.5`, `266 + 45.5 = 311.5`, `470 + 75.5 = 545.5` — which is the same destination
reached from the other end. **The other ten rows did not move at all** —
W2, W3, W5, W6, W8, W9, W10, W13, W14 and W15 — and each states in its own basis why: for
[W5](#w5--repository-wide-path-casing-audit) the band is
**re-derived from the current census rather than carried**, and it lands where it was; for the other nine
nothing in their basis changed. Nine moved plus ten unmoved is the nineteen rows
of [§5.1](#51-summary-table) that enter the total, so the walk accounts for every row rather than for the
ones that changed. The conditional twentieth row is excluded by its own label and is not part of any
column here.

**Where the intermediate figures live, and why they are not repeated here.** Four of the six re-derived
rows passed through published intermediate bands on the way to the figures above —
[W4](#w4--build-governance-bootstrap-pre-port-behavioural-baseline-and-test-suite)'s 38 / 66.5 / 112.5 and
[W7](#w7--the-aspnet-core-port)'s 68 / 119.5 / 204.5, which are the figures the round before this one
published; [W12](#w12--administrator-provisioning-tool)'s 3 / 5 / 9.5, 3 / 5.5 / 10.5 and 3.5 / 7.5 / 12.5, and
[§7.1](#71-the-manual-visual-review)'s 2 / 3.5 / 6 — and each is recorded in the section that owns the
row, beside the increment that moved it. Restating them here would create a second place for a figure to
drift, which is the failure this section exists to catch rather than to reproduce.

### 6.2 The finding that matters most in this document

**The port of existing code is under a fifth of the expected effort. Everything else — net-new
capability, data work, and governance and verification — is over four fifths.**

**This table partitions the model by activity, not by workstream, and that distinction is the correction
that produced the figures below.** An earlier version of this section assigned each workstream wholly to
one character. That put **all** of W7 in *porting* although five of its thirteen sub-rows have no source
counterpart at all, **all** of W11 in *net-new* although its first part is an approval and nothing else,
**all** of W13 in *governance* although the row is the production data extraction, load and
reconciliation, and **all** of W16 in *governance* although its second stage is an implementation with a
verification obligation. A partition by workstream cannot be right for a model whose largest row is
itself mixed, which is why this section reads the activity rather than the row name.

**How each row was assigned, stated so the partition can be re-checked rather than taken on trust.**
Three rules, applied in order:

1. **A row with a stated sub-decomposition is split along it**, using its sub-bands exactly as that row
   publishes them. Three splits cross a category boundary:
   [W7](#w7--the-aspnet-core-port)'s **nine port sub-rows** against its **four suite-and-artifact
   sub-rows** — the grouping its own printed addition uses — with the one further port sub-row named in the
   movement list below crossing with them;
   [W11](#w11--ci-provider-selection-then-pipeline-authoring)'s two parts, 1.5 / 3 / 5 for the provider
   decision and 9.5 / 16.5 / 30.5 for the manifest; and
   [W16](#w16--personal-data-governance)'s two stages, 1 / 2.5 / 6 for policy and 2 / 3.5 / 6 for
   mechanism. Two further rows are decomposed but their splits do **not** cross a boundary —
   [W4](#w4--build-governance-bootstrap-pre-port-behavioural-baseline-and-test-suite)'s gate-4a and
   gate-4b halves are both net-new, and the visual review's capture and review halves
   ([§7.1](#71-the-manual-visual-review)) are both verification — so each is carried whole. No sub-band is
   re-judged to make a category come out.
2. **A row without a sub-decomposition is assigned whole, to the character its own basis states.**
   [W4](#w4--build-governance-bootstrap-pre-port-behavioural-baseline-and-test-suite) states "entirely net-new";
   [W10](#w10--hosting-provisioning-and-platform-configuration), [W12](#w12--administrator-provisioning-tool) and
   W11's part 2 replace nothing that exists in the repository;
   [W3](#w3--authoritative-schema-extraction), [W8](#w8--identity-migration-tooling-rehearsed-against-a-copy),
   [W9](#w9--domain-data-migration-tooling-rehearsed-against-a-copy) and
   [W13](#w13--cutover) are the data work;
   [W6](#w6--project-format-conversion-and-dependency-transition) is porting, because its two drivers are the
   28 package outcomes and the conversion of an existing project file, and the declarative
   manifests inside it are below the granularity of [§4.2.1](#421-the-rounding-rule-stated-once);
   [W1](#w1--approval-of-this-assessment), [W2](#w2--mvc-5-build-reproduction-and-the-restore-precondition),
   [W5](#w5--repository-wide-path-casing-audit), [W14](#w14--documentation-revision),
   [W15](#w15--affinity-retirement) and all three non-code tasks are governance and verification.
3. **Every split sums back to its row's published band, and the four categories sum to the model total in
   all three columns.** Both identities are checked immediately below rather than asserted.

| Character of the work | Rows and parts | Low | Expected | High | Share of expected |
| --- | --- | ---: | ---: | ---: | ---: |
| **Porting existing code** | W6; W7's **eight** porting sub-rows | 26.5 | 50.5 | 86.5 | **16.2%** |
| **Net-new capability, with no legacy volume to scale from** | W4; W7's **five** net-new sub-rows; W10; **W11 part 2**; W12; **W16 stage 2** | 105.5 | 182.5 | 313 | **58.6%** |
| **Data work gated on a schema not yet extracted** | W3, W8, W9, **W13** | 12 | 24 | 47 | **7.7%** |
| **Governance, verification and documentation** | W1, W2, W5, **W11 part 1**, W14, **W15**, **W16 stage 1**, and all three non-code tasks | 27.5 | 54.5 | 99 | **17.5%** |
| **Total** | | **171.5** | **311.5** | **545.5** | **100.0%** |

**The shares are stated to one decimal place, and this round they sum to exactly 100.0 — which is
arithmetic rather than tidying.** Each is the quotient of the expected column, rounded independently to the
nearest tenth: `50.5 / 311.5 = 16.212`, `182.5 / 311.5 = 58.588`, `24 / 311.5 = 7.705`,
`54.5 / 311.5 = 17.496`, so a
reader can divide and check every one. Two of the four round **up** and two round **down**, and
`16.2 + 58.6 + 7.7 + 17.5 = 100.0`. **The exact quotients sum to exactly 1**, which is the identity that
matters and which the category sums below establish directly against the total. No share is nudged to make
the column read 100.0, because a nudged share is no longer the quotient a reader can check — and it has
not always read 100.0: the round before summed to **100.1** at `16.4 + 58.0 + 7.8 + 17.9 = 100.1`, and an
earlier one to **99** at `17 + 57 + 8 + 17 = 99`, so the convention cannot depend on which way the
quotients fall, in either direction. Rounded to whole percentages this round's shares also sum to 100 —
`16 + 59 + 8 + 17`.

**The two identities, as arithmetic.** Category sums, column by column:
`26.5 + 105.5 + 12 + 27.5 = 171.5`, `50.5 + 182.5 + 24 + 54.5 = 311.5`,
`86.5 + 313 + 47 + 99 = 545.5` — each equal
to [§6.1](#61-the-totals)'s total for that column. Split sums, against the band each row publishes:
W7's `24 + 45.5 = 69.5` / `46 + 76.5 = 122.5` / `78 + 130.5 = 208.5`; W11's `1.5 + 9.5 = 11` /
`3 + 16.5 = 19.5` / `5 + 30.5 = 35.5`; W16's `1 + 2 = 3` / `2.5 + 3.5 = 6` / `6 + 6 = 12`. Every one
matches.

**Which assignments moved, and in which direction.** Five, and none of them is a re-estimate — every band
above is the one its own row publishes:

- **W7 splits**, moving `45.5 / 76.5 / 130.5` out of *porting* into *net-new*: the four
  suite-and-artifact sub-rows — the target-facing suite machinery, the browser-driven flow, the cases, and
  the migration artifact — plus the **security-header middleware, error action and generic error view**
  sub-row at `1.5 / 3 / 5`, which is the one of the nine port sub-rows with nothing to port:
  [11 §3](11-cloud-readiness-assessment.md) records the response headers as absent from the source
  entirely. `24 + 45.5 = 69.5` at the low band leaves the **eight** remaining port sub-rows in *porting*.
- **W11 splits**, moving its `1.5 / 3 / 5` provider gate out of *net-new* into *governance*. Its own
  sub-table calls that part an approval rather than engineering.
- **W16 splits**, moving its `2 / 3.5 / 6` mechanism stage out of *governance* into *net-new*. Conditions 4
  to 6 are an operation that runs, a window that is measured and an audit trail observed arriving.
- **W13 moves whole**, `2 / 4 / 8`, out of *governance* into *data work*. It is the one and only
  production extraction, load and reconciliation, gated on the same unextracted schema as W9.
- **W15 moves whole**, `0.5 / 1 / 2`, out of *net-new* into *verification*. The capability it retires
  affinity against is W10's; this row is the cross-instance round-trip that proves it, plus one setting.

**The non-porting share, re-derived rather than carried over.** `182.5 + 24 + 54.5 = 261` expected IED of
**311.5** — **83.8%** — against the **50.5** that is porting. The previous readings of this section stated
**83.6%** as `178.5 + 24 + 55 = 257.5` of 308, then
**83.3%** as `178.5 + 24 + 49.5 = 252` of 302.5, then
**82.6%** as `166 + 24 + 49.5 = 239.5` of 290, then **76.6%** as `82.5 + 34 + 37 = 153.5` of 200.5, and
before that **~70%**. Both the
share and its complement moved again, and in the same direction, for two reasons that are arithmetic
rather than a change of view:

- **Porting did not move at all this round**, and stands where it stood at `26.5 / 50.5 / 86.5`. The
  coverage matrix's extension lands entirely in test authoring, and both rows it moved —
  [W4](#w4--build-governance-bootstrap-pre-port-behavioural-baseline-and-test-suite)'s gate-4b case
  sub-row and [W7](#w7--the-aspnet-core-port)'s target-facing case sub-row — sit in *net-new* by rule 2
  and rule 1 respectively; and the sign-off row, the one row that fell, is a non-code task and sits in
  *governance*. Porting last grew by 3.5 expected IED, `47 → 50.5`, three generations of this
  table ago, and has not moved since.
- **Every movement this round landed outside porting, in both directions.** The total grew by **3.5**
  expected IED
  (`308 → 311.5`), of which **none** is porting: `1` is W4's case sub-row and `3` is W7's, both net-new,
  and `−0.5` is the approved-delta sign-offs, which is governance and which is the only category to
  **fall** — `1 + 3 − 0.5 = 3.5`. **The round before was the same in kind if not in size**: the
  total grew by **18** expected IED
  (`290 → 308`), of which **none** was porting: `0.5` was W4's case sub-row, `10` W7's, `2`
  W12's eight published-console rows and `5.5` the sign-off row following the two approval registers'
  present size, and `0.5 + 10 + 2 + 5.5 = 18` — net-new in the first three places and governance in the
  fourth.

Porting therefore falls from **17.4%** to **16.7%**, then to **16.4%**, and now to **16.2%** while the
model grows — the seventh consecutive round in
which it has fallen, and for the same structural reason each time: the work that keeps being discovered is
work the existing codebase does not contain.

**This is the conclusion an estimate derived from lines of code cannot reach.** Scaling from the
migration source's 2,097 non-blank lines (sizing metric) would produce a number for the **16.2 percent**
that is porting and **silently omit the other 83.8 percent**, because none of it is predicted by any
quantity in the existing codebase:

- **The test suite has no legacy volume at all.** There are **0** tests today (input 15), and the parity
  suite is required against **104** coverage rows carrying **326** cases, **152** of which execute against
  both fixtures and therefore twice (input 14, which owns those counts and re-reads them) — plus **17**
  further required tests that have no
  baseline to run against (input 23) and the **8** published-console rows outside that matrix (input 32),
  both costed in the workstreams that build what they test, for
  `104 + 17 + 8 = 129` executable scenarios in all. Its authoring half is the
  second-largest row in the model, and its size is set by the behaviour to be characterized, not by the
  code that implements it.
- **Observability, CI and the provisioning tool are absences.** The repository has no logging,
  telemetry or health endpoint [08 §7.1](08-technical-debt-register.md) and no pipeline, publish
  profile or container manifest [08 §7.2](08-technical-debt-register.md). Work that replaces *nothing*
  cannot be sized from the thing it replaces.
- **The data migrations are gated on an extraction that has not happened.** Their band is set by the
  gate, not by six entities' worth of rows.
- **Two of the largest single items in the model are not code at all** in the ordinary sense: W4's
  fixtures, and the **63** approval decisions of
  [§7.2](#72-the-approved-delta-sign-offs).

Correspondingly, **the seed data is the model's clearest example of volume without effort**: 820
non-blank lines (sizing metric), 39 percent of the migration source, and about **2 percent** of W7's band
(`2 ÷ 122.5`), because [08 §4.2](08-technical-debt-register.md) directs it be treated as a data-handling
decision rather than as porting. Volume and effort are not proportional in either direction on this
codebase.

### 6.3 What is deliberately not in the total

- **The conditional incremental path.** Assumption **A6** takes the settled single-cutover approach.
  The alternative in [05 §11.6](05-aspnet-core-migration-approach.md) is a different shape — two hosts,
  two pipelines, an adapter surface on both sides — and **this model does not cost it.** If that path is
  ever selected, this document is re-estimated rather than adjusted.
- **Porting either reference edition.** Assumption **A7**; see
  [R10](#r10--scoping-by-analogy-across-editions).
- **Repository-hygiene remediation**, enumerated but intentionally unestimated in
  [section 7.4](#74-repository-hygiene--enumerated-but-intentionally-unestimated) because it gates
  nothing.
- **Any new functional requirement.** Assumption **A9**.
- **The interim Windows hosting path.** [06 §5](06-azure-hosting-recommendations.md) records it as an
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
- **~~The headless-browser harness.~~ No longer excluded — it is inside the bands.**
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
  ladder's whole remainder: [06 §11.5](06-azure-hosting-recommendations.md) states that **the ladder has
  four rungs and no exception**, so there is no further rung for this section to exclude — in particular
  none that returns to the legacy application after an accepted write. Each excluded rung is
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

**Three of the four items below are in the total**, they consume real effort, none is sized by any other
document, and none is a
workstream **of its own** in [03 §5](03-modernization-roadmap.md)'s decomposition — but each of the three
sits on a
workstream's **exit gate**: the visual review and the accessibility review on **W7's**, and the delta
sign-offs on **W1's**. So each is on the
critical path even though it has no workstream number. They are carried here because omitting them would
understate the total and mis-place the gates — which is exactly what a previous revision did with two of
them, as [§6.1.1](#611-the-walk-from-the-previously-published-total) records. The fourth item,
[§7.4](#74-repository-hygiene--enumerated-but-intentionally-unestimated)'s repository hygiene, is
enumerated there **without a band of any kind** and is **deliberately outside the total**, because it
gates nothing.

One qualification, because §7.1 spans two gates rather than one. Its **review and sign-off** sit on W7's
exit gate and are on the critical path; its **baseline capture** sits inside W4's gate 4b, concurrent with
that gate's suite authoring, and is therefore off the path. Both halves are sized in §7.1 and neither is
folded into a workstream band.

### 7.1 The manual visual review

**Why it exists as a separate task.** [05 §12.5](05-aspnet-core-migration-approach.md) establishes that
the visual-preservation criterion **cannot** be met by the HTTP-level suite — that suite asserts on
response *content*, while the styling framework's major-version move changes how content *renders* — and
that automating the comparison is **deliberately rejected**. That document scopes the review and records
that **07 carries it as a task**. This is that task.

**The capture set is one number, and it is not this document's.**
[05 §12.5.1](05-aspnet-core-migration-approach.md) publishes a single manifest — **67 baseline images and
the same 67 comparisons**, as `34 + 33 + 0 + 0 = 67` across its four passes — and states that this section
prices exactly that total. It is cited, not re-derived: the four passes are **P1** appearance (the
seventeen capturable page surfaces at the two viewports 05 §8.5 fixes), **P2** boundary (three
layout-bearing surfaces at eleven widths), and **P3** and **P4**, two data-conditional states standing at
**zero** because 05 measured the committed Identity store rather than assuming it.

**The 33 breakpoint-boundary images are inside that 67, as pass P2.** They are not a second capture set and
are not priced separately anywhere in this document. If a source database is substituted and 05's two
measurement queries return rows, P3 or P4 becomes **2** images each and the manifest total moves; this row
then moves by the per-image and per-comparison rates below and by nothing else. No other pass is
conditional.

**Scope, as scoped by 05 §12.5** and not redesigned here: the images above; a reviewer checklist covering
the navigation bar, catalog grid, album detail, cart table, checkout form and administration list; a
**scripted-interaction checklist** exercised in a browser, because the suite executes no JavaScript; **two
content-security-policy glyph checks**; **one external-destination check**; and a **signed-off approval
artifact** recording who reviewed it, against which baseline, and which entries of **both** approval
registers were accepted.

**What the review requires and does not photograph is priced, because an unpriced obligation is how one
review acquires four budgets.** 05 §12.5.1 classifies each: the scripted-interaction checklist is items
**c1** to **c5** of [05 §8.9](05-aspnet-core-migration-approach.md), whose net-new items are additions
**A2** to **A6**; the two glyph checks are **target-only** and exist only under the enforced policy; the
external-destination check **adds no capture**, because both links render inside surfaces P1 already
collects and row **102** of [05 §12.4](05-aspnet-core-migration-approach.md) asserts the two `href` values
themselves; the preserved-rendering rows of 05 §8.5.7 and the measured checks of 05 §8.5.6 are verified as
**measured values** rather than as images; and the accessibility review is a second, distinct task, sized
in [§7.3](#73-the-manual-accessibility-review).

**Three page surfaces have no baseline at all, and each is reviewed rather than skipped.** 05 §12.5 states
the criterion — **the running legacy application offers no user-reachable navigation path to the surface**,
deliberately narrower than the claim that no request whatsoever can render it, because a manual appearance
review photographs what a user can reach and a request a reviewer *synthesizes* is not navigation — and
gives the reason for each: `Home/About.cshtml` and `Home/Contact.cshtml` are **orphaned**, no
action returning either, and `Account/ExternalLoginConfirmation.cshtml` has **no GET at all**, its action
being `POST`-only, so no sequence of links, submissions and redirects arrives there. All three are **ported**, because 05 §8.4
deletes no view and 05's external-login decision **retains the surface dormant**: nothing 404s by design,
and nothing is deleted for being unreachable in this configuration. So each is reviewed as a **markup
comparison of the legacy file against the ported one**, and the review record states the absence of a
baseline rather than reporting a pass. That component is costed below.

**How the four families enter the sizing, without inventing a second capture total.**
[06 §10.4](06-azure-hosting-recommendations.md) owns the matrix — **Chrome, Edge, Firefox and Safari**,
stated as **current evergreen** rather than by pinned versions, so this row buys no version pinning and no
per-version repetition — and states what coverage each family actually receives. Three consequences, all
consumed rather than decided here:

- **The 67 images are assigned to families by the capture protocol, not multiplied by four.** A comparison
  is only evidence if both sides come from the same family: a Chrome baseline against a Safari post-port
  capture confounds the framework upgrade with the rendering engine. So the protocol assigns each surface a
  family such that all four are exercised and every one of the 67 comparisons stays same-family.
  Multiplying the manifest by four would publish a second capture total, which
  [05 §12.5.1](05-aspnet-core-migration-approach.md) exists to prevent. The limitation is therefore stated
  rather than priced away: **no single surface is compared in all four families**, and a stakeholder who
  wants that is asking 05 to change its manifest rather than asking this row for a different rate.
- **Edge is inside the manual appearance set with the other three, on the same terms.** No family receives
  automated functional coverage — [06 §10.4](06-azure-hosting-recommendations.md) states that there is no
  functional browser automation in any engine — so Edge is neither privileged nor discounted here: its
  appearance coverage is manual exactly as Chrome's, Firefox's and Safari's are, and is inside the 67. The
  residual 06 §10.4 refuses to accept is the **browser-executed behaviour of the one script-dependent flow**
  across **all four** families, an open limit owned by
  [05 §12.11](05-aspnet-core-migration-approach.md) and answered by the **required four-engine functional
  walk** 06 §10.4 obliges. **That walk is not this row and is not inside this row's band**: it is charged at
  [W10](#w10--hosting-provisioning-and-platform-configuration)'s gate 10b at `1 / 2 / 3` alongside
  `G-CSP-BROWSER`, and it **adds no image and no comparison** to the 67 — a functional walk exercises a
  request and a DOM rewrite, and this row compares rendering. Two manual browser tasks over the same four
  families are two tasks, and the earlier form of this bullet — which made the walk a conditional
  re-estimation of this row's reviewer scope — is superseded by 06 §10.4 requiring it outright.
- **Safari is the constraint on the capture half.** It has no Windows or Linux build, so that family needs a
  **macOS host or a device-lab session**; the access is a costed component below rather than an assumption.
  Without it one of the four families goes unexercised in both directions.

**Both halves are manual, and this task introduces no automation of any kind.** That is
[05 §12.5](05-aspnet-core-migration-approach.md)'s decision, and the reason it gives is the machinery:
automating would mean **pinning a browser-automation stack and screenshot tooling, defining viewports,
baseline images and pixel tolerances, and storing binary image artifacts** in a repository that already
carries committed database binaries — infrastructure the application needs for nothing else, bought for a
one-time layout migration. Two facts make manual the cheaper answer rather than the resigned one: **Safari
has no Windows or Linux build**, so a single scripted runner could not cover the matrix in any case; and the
capture happens **once**, against an application being retired, so there is no second run for a harness to
amortize against.

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
  class A. The list branch [src/MVC5/MvcMusicStore/Views/Account/_ExternalLoginsListPartial.cshtml:15-30] needs the same configured provider, and is captured in the same run
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
one breakpoint [src/MVC5/MvcMusicStore/Views/Shared/_Layout.cshtml:15], [src/MVC5/MvcMusicStore/Views/Shared/_Layout.cshtml:22] and the viewport meta tag
[src/MVC5/MvcMusicStore/Views/Shared/_Layout.cshtml:5] confirms the responsive intent. Assumption **A4** still applies — one reviewer, with a second
signature only for approval.

> **The browser-matrix review has two halves with two owners, and this row is one of them.**
> [05 §12.5](05-aspnet-core-migration-approach.md) discharges **content-security-policy enforcement**
> through the same 06 §10.4 matrix and on the same manual footing, because no `WebApplicationFactory`
> client parses a policy or emits a violation report. That half is the deployed-browser gate
> `G-CSP-BROWSER`, it is **[W10](#w10--hosting-provisioning-and-platform-configuration)'s exit condition 8**, and
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
| Baseline capture | **2 / 3.5 / 7** | **At W4's gate 4b**, against the running legacy application | [03 §5 W4](03-modernization-roadmap.md) makes the capture part of that gate, because the application being photographed is the one gate 4b already builds, resets and drives |
| Review **and sign-off** | **5.5 / 9.5 / 16** | **Inside W7, as exit condition 5 of its gate** | Reviews the ported rendering against the captured baseline, and records the signed approval that closes the gate |

`2 + 5.5 = 7.5`, `3.5 + 9.5 = 13`, `7 + 16 = 23`.

**The capture part, derived.** Four components, and the rates are this row's judgement while every count is
05's:

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
- **Delta adjudication and the signed artifact** — recording which of the three approved visual deltas of
  [05 §11.5](05-aspnet-core-migration-approach.md) were accepted, and by whom: the Bootstrap markup
  migration, the Glyphicon removal and the sign-out control's change of form: **0.5 / 1 / 2**.
- **The two class C pages** — markup comparison of legacy against ported, and recording the absence of a
  baseline rather than a pass: **0.5 / 0.5 / 1**.
- **Review subtotal `3 + 0.5 + 0.5 / 5.5 + 1 + 0.5 / 8.5 + 2 + 1` = 4 / 7 / 11.5.**

`2.5 + 4 = 6.5`, `5.5 + 7 = 12.5`, `11 + 11.5 = 22.5` — the row's band.

> **The review and its sign-off are inside W7, not after it.**
> [03 §5 W7](03-modernization-roadmap.md) makes them **exit condition 5** of the port's gate, so they close
> before W13 is entered and before any traffic is served from the ported application. Only the **baseline
> capture** precedes the port, and it has to, since the legacy application is the artefact being
> photographed.
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

**Why it exists as a separate task.** [05 §11.5](05-aspnet-core-migration-approach.md) enumerates the
**approved deltas** — that register's own count is stated there and nowhere else — and
[05 §11.7](05-aspnet-core-migration-approach.md) enumerates **15
approval-owned additions** to the plan, each row of each with a **named approval owner** — and an approval
owner who has not actually approved is an open question, not a delta. Obtaining the decisions is work
with elapsed effort, and [03 §5](03-modernization-roadmap.md)'s W1 exit gate requires a recorded
decision on every row of both registers.

**Basis.** **62 recorded decisions as the two registers stand** — every row of 05 §11.5's approved-delta
register **that carries an approval owner**, its **two retired identifiers** `D-22` and `D-43` carrying
none and needing no decision, plus 05 §11.7's **15** approval-owned additions — across
**5 approver constituencies** (input 17,
entry count) — the security, product, engineering, data and operations owners named by 05 §11.5. That
62 is **this document's one statement of the priced decision count**, and it is an observation about two
registers a sibling document owns rather than a count this document allocates: the delta register's own
count is [05 §11.5](05-aspnet-core-migration-approach.md)'s to state, and a re-count there moves this
figure and nothing else in the derivation below. **A re-count is exactly what has happened**, and the
figure above is the post-count one: [05 §11.5](05-aspnet-core-migration-approach.md) retired a **second**
identifier, so one fewer of its rows carries an approval owner, while
[05 §11.7](05-aspnet-core-migration-approach.md)'s fifteen additions are unchanged — and that owner
publishes the resulting sum itself, as `47 + 15 = 62`. This row therefore prices **one decision fewer
than it did**, and the derivation below is re-run rather than adjusted. Two
properties drive the band rather than the count:
**a substantial minority of the delta register's rows name more than one** constituency in their
**Approval owner** cell and must be consented by every owner they name — **how many of its rows those
are is that register's to state, on the same footing as its total**, and the rate below reflects the
property rather than a subcount, so a re-count there moves no figure here — and one of them is not a
technical trade at all —
[R7](#r7--the-narrowed-browser-matrix) removes a class of client and belongs to the product owner
alone.

**The multi-owner figure is a projection of the register and is derived by subject, not by delta number.**
An earlier form of this row published **sixteen**, and delta numbers are the wrong key to check that
against: [05 §11.5](05-aspnet-core-migration-approach.md)'s rows have been renumbered as the register was
consolidated, so a figure keyed to positions in an older numbering cannot be re-derived. Read by
**subject** — each row's own approval-owner text — the register partitions into **13 single-owner** rows
and **14 multi-owner** rows: `13 + 14 = 27`, and because every multi-owner row names exactly two
constituencies, the owner **participations** are `13 × 1 + 14 × 2 = 41`.

**The partition is re-derived from the register's own owner column rather than counted by hand**, so a
row whose owner set changes moves these figures without anyone remembering to update them (AAP §0.11.3):

```bash
D=docs/modernization/05-aspnet-core-migration-approach.md
OWNERS=$(sed -n '/^| # | Approved delta | Why | Section | Approval owner |$/,/^$/p' "$D" \
  | grep -E '^\| [0-9]+ \|' | awk -F'|' '{print $(NF-1)}' | sed 's/\*\*//g; s/^ *//; s/ *$//')
echo "$OWNERS" | grep -c .                       # -> 27   register rows
echo "$OWNERS" | grep -vc ' and '                # -> 13   single-owner rows
echo "$OWNERS" | grep -c ' and '                 # -> 14   multi-owner rows
echo "$OWNERS" | sed 's/ and /\n/g' | grep -c .  # -> 41   owner participations
echo "$OWNERS" | sed 's/ and /\n/g' | sed 's/^ *//; s/ *$//' \
  | tr '[:upper:]' '[:lower:]' | sort | uniq -c  # -> 4 data owner, 11 engineering,
                                                 #    3 operations, 14 product, 9 security
```

**And the same reading row by row, so the figures are checkable without a shell.** The delta numbers are
[05 §11.5](05-aspnet-core-migration-approach.md)'s, read at the state this document was written against:

| Owner set | Delta numbers | Rows |
| --- | --- | --- |
| Security | 1, 10, 14, 18 | 4 |
| Product | 7, 16, 17, 19, 21 | 5 |
| Engineering | 4, 12, 13 | 3 |
| Data owner | 3 | 1 |
| **Single-owner subtotal** | | **13** |
| Security and operations | 2 | 1 |
| Product and engineering | 5, 6, 15, 20, 25 | 5 |
| Product and operations | 8, 9 | 2 |
| Product and security | 11 | 1 |
| Product and data owner | 22 | 1 |
| Engineering and security | 23, 24 | 2 |
| Data owner and engineering | 26 | 1 |
| Security and data owner | 27 | 1 |
| **Multi-owner subtotal** | | **14** |
| **Register** | | **27** |

Read the other way, the participations are the per-constituency load: **product 14, engineering 11,
security 9, the data owner 4, operations 3** — and `14 + 11 + 9 + 4 + 3 = 41`.

**What moved, and what the movement does not touch.** Delta 6 — the Glyphicon removal — gained
**engineering** alongside product, so that the register matches AAP §0.1.1's assignment of the
Bootstrap-markup-and-icon delta to *product and engineering*; a decorative icon dependency being removed
from two views is an engineering call as much as a product one. That single row is the whole of the
movement, and it converts one single-owner row into a multi-owner one: single-owner **14 → 13**,
multi-owner **13 → 14**, participations **40 → 41**, engineering's load **10 → 11**. The **27** register
rows and the **16** additions of [05 §11.7](05-aspnet-core-migration-approach.md) are unchanged, as are
the other four constituencies' loads.

**The band does not move, and that is a property of the derivation rather than an absorption**: the band
below is `27 decisions × a per-decision marginal rate`, the decision count is what it was, and the
multi-owner proportion is what makes that rate what it is rather than being a term in the product. Were
participations a term, one more of them would raise the band; as a driver of the rate, it would take a
re-judged rate to move it, and a register that was already 13-of-27 multi-owner becoming 14-of-27 does not
justify one. **Nothing else in this document consumes the partition**, which is stated rather than assumed:
[W1's band](#w1--approval-of-this-assessment) is `11 acts × 0.3 / 0.6 / 1.2` and counts **constituencies**
— five plus legal, a figure this change leaves at five — not participations;
[§8.2](#82-concurrency-permitted-by-the-graph)'s set 0 and
[§8.3](#83-the-critical-path-and-what-to-do-first-if-the-goal-is-to-narrow-the-estimate)'s chain carry this
row's *band*, not its owner arithmetic; and [§5.1](#51-summary-table)'s totals consume the band as well. So
those figures are left exactly as they stand.

**The additions register is priced here, and until the round that added it to this count it was priced
nowhere.**
[05 §11.7](05-aspnet-core-migration-approach.md)'s fifteen entries are additions to the plan carrying an
approval owner of the same kind as a delta's, and W1's exit gate requires a recorded decision on each.
They were not among the delta rows this row once priced alone and they are not in
[W1's act census](#w1--approval-of-this-assessment),
which prices convening and the escalated non-delta decisions only — so **no row of this model paid for
them at all**, while [§7.3](#73-the-manual-accessibility-review) already stated that this row obtains the
decisions on the sixteen accessibility requirements that register carries. The count above spans both
registers for that reason and not because they were merged: they remain two registers, each row of each needing
one recorded decision, and this row pays for every row of both.

**Two of these decisions are also §9.4 escalated risks, and this row is the one that pays for them.**
[R7](#r7--the-narrowed-browser-matrix) is the narrowed-browser-matrix delta and
[R9](#r9--cutover-re-authentication-and-anonymous-cart-loss) is the reauthentication-and-anonymous-cart-loss
delta. [W1's basis](#w1--approval-of-this-assessment) therefore excludes both from W1's band and takes
only R1, R13, R15 and R18 there — the four escalated decisions that are not deltas. The partition is
stated in full there and is what keeps `7 + 16 = 23` free of overlap.

**Band 8 / 16 / 26 IED**, moved by a count and **derived rather than re-judged**. The **convening
cost does not move**, because the constituency count does not move: it is five, as it was at 14 decisions,
at 18, and at every count the delta register has carried since. What moves is the number of decisions those
five must record. The marginal rate is the
one this row's own history establishes — at 14 the band was 2 / 4 / 8 and at 18 it was 2.5 / 5 / 9.5, so
four additional decisions cost 0.5 / 1 / 1.5, which is **0.125 / 0.25 / 0.375 IED per decision**.
Forty-eight further decisions therefore cost
`48 × 0.125 / 0.25 / 0.375 = 6 / 12 / 18`, and added to the 14-decision base of
2 / 4 / 8 that gives **8 / 16 / 26**.

**At this count [§4.2.1](#421-the-rounding-rule-stated-once)'s rule is a no-op, and that is worth saying
rather than leaving a reader to assume an omission.** `ceil(2x) / 2` maps 8 to 8, 16 to 16 and 26 to 26,
because forty-eight is a multiple of eight while the rate's three columns are one, two and three eighths,
so the marginal product is the whole numbers 6, 12 and 18 and an integer base leaves the sum on the grid
without help. The previous count did **not** behave that way: at forty-nine decisions the unrounded
product was `8.125 / 16.25 / 26.375`, and every column of the band that was published was the product of a
round-up. So the rule did visible work last round and does none this round, which is a property of the
count rather than a change to the rule.

**The rule is applied once, to the figure this row publishes, and the check that establishes why agrees
with it at this count.** Derived instead as an increment on the previously published 5.5 / 11 / 18.5,
twenty further decisions cost
`20 × 0.125 / 0.25 / 0.375 = 2.5 / 5 / 7.5`, which is already on the half grid and needs no round-up, and
which lands on
`5.5 + 2.5` / `11 + 5` / `18.5 + 7.5` = **8 / 16 / 26** — the same band in all three columns. The two
routes have not always agreed: at 42 decisions the incremental route landed half a day high in the low and
high columns, because the band it incremented already carried a round-up and adding a second one compounds
the same slack twice. **The base-plus-marginal derivation governs regardless of whether the two agree**, and
the incremental one is recorded as a check rather than as an alternative — a check that happens to be
confirmatory this round, and confirmatory for a better reason than last round's, since neither route
applies a round-up at this count. Saying which governs is the difference between a model
and a number.

**This band moved from 5.5 / 11 / 18.5 to 8 / 16 / 26, and a count moved it rather than a rate.** The
priced count is the two registers' own, and it rose because
[05 §11.5](05-aspnet-core-migration-approach.md)'s register was read **at its present size** rather than at
the smaller reading an earlier revision of this row had of it — that register states its own row count and
this document does not transcribe it — while
[05 §11.7](05-aspnet-core-migration-approach.md)'s **15** additions are unchanged. **The rate is
unchanged** at 0.125 / 0.25 / 0.375 per decision, and the whole of the movement is twenty further rows
entering the marginal count, which the derivation above carries as 48 marginal decisions on the
14-decision base. **The round before this one published 8.5 / 16.5 / 26.5 from the same base and the same
rate on forty-nine marginal decisions**, and the half-day fall in every column is the one retired
identifier plus the round-up that count needed and this one does not: `0.125 + 0.375 = 0.5` low,
`0.25 + 0.25 = 0.5` expected, `0.375 + 0.125 = 0.5` high — the marginal decision itself, and the slack the
rounding rule was adding. The earlier rounds' own movements stand as recorded: the band had moved from
4 / 7.5 / 13 to 5.5 / 11 / 18.5 when the additions register entered the count at all, having been priced
by no row of this model before that, and from 3.5 / 7.5 / 13 to 4 / 7.5 / 13 when
[§4.2.1](#421-the-rounding-rule-stated-once)
replaced a rounded-down 1 with 1.5 in the nine-decision increment's low column.

**Where this row sits relative to W1, stated exactly.** It is effort **required by W1's exit gate**, so it
is sequenced with W1 — [§8.2](#82-concurrency-permitted-by-the-graph) places it in **set 0** beside W1, as
the parenthesised `+16` in that set's own row, and
[§8.3](#83-the-critical-path-and-what-to-do-first-if-the-goal-is-to-narrow-the-estimate) carries it on the
chain — but it is **not inside W1's band**. The two are a
partition of input 17, tabulated in [W1's basis](#w1--approval-of-this-assessment): W1's 3.5 / 7 / 13.5 buys
the convening of the **6** constituencies — this row's five delta owners plus legal — and the **4**
escalated risk decisions that are **not** deltas; this row's 8 / 16 / 26 buys the decisions of the
two approval registers, R7 and R9 among them, at the count its own basis states. The summary table in [section 5.1](#51-summary-table) lists each exactly once,
so neither is double-counted into the other, and **7 + 16 = 23** expected IED is the whole approval
activity.

**W1's own band reads 3.5 / 7 / 13.5, and it is derived rather than carried.**
[W1](#w1--approval-of-this-assessment)'s basis prices a census of **eleven mutually exclusive approval
acts** — 6 constituency briefings, the **4** escalated risk decisions that are not themselves deltas, and
the gate record — at `0.3 / 0.6 / 1.2` IED per act, rounded up to the half grid. Three corrections moved
that census: legal's briefing added an act, R7's and R9's decisions removed two once they were paid per
decision in this row instead, and R18's recorded outcome added one because
[§9.2](#92-register-index) assigns that outcome there. The three net to zero **in count** and not in
membership, and W1's basis walks all three so the point is checkable rather than asserted. W1 names the
registers' decisions as *this* row's work rather than as one of its own drivers, so a re-count of either
register moves this row and not that one.

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

**Scope, as scoped by 05 §8.9** and not redesigned here: the **24** numbered requirements of that section
in its own grouping — structure and semantics `a1`-`a8` plus `a5b` and `a5c` (10), labels, errors and
validation `b1`-`b5` (5), the cart interaction's announcement, pending, failure and focus behaviour
`c1`-`c5` (5), and focus visibility and colour `d1`-`d4` (4).

**The approval side of the same items is the two approval registers, and neither adds a checklist item.**
[05 §8.9](05-aspnet-core-migration-approach.md) states the mapping exhaustively and it is cited rather
than restated: **sixteen** of the 22 requirements are net-new and carried by the additions register of
[05 §11.7](05-aspnet-core-migration-approach.md) as thirteen of that register's **fifteen** entries, three of the thirteen covering a pair;
**two** — `a7` and `d1` — change an existing behaviour and are therefore rows of the delta register in
[05 §11.5](05-aspnet-core-migration-approach.md), namely the sign-out control's change of form and the
per-line cart quantity maximum; and **four** are on neither register because none is a
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

**Basis.** The **17 capturable page surfaces** of
[05 §12.5.1](05-aspnet-core-migration-approach.md) — input 16 — bound the traversal, and the **24**
numbered requirements above bound the checklist. Assumption **A4** applies: one reviewer, with a second
signature only for approval. The band is driven by the **interaction-state pass** rather than by either
count, because three of its four cases (**c2**, **c3**, **c4**) are behaviours the source does not have at
all and therefore have to be exercised rather than compared. That is also why the band does not scale with
the checklist: `a5b` is the third table's accessible name, read on a page the traversal already reaches;
`a5c` is a screen-reader check in one state of that same page; `a8` is a keyboard-and-eye check on two
surfaces the traversal already visits; and `d4` is one more colour sampled by the tool already open for
`d3`.

**Band 2.5 / 4.5 / 8 IED.** It runs **inside W7's exit gate**, alongside the review half of
[section 7.1](#71-the-manual-visual-review), and it is therefore **on the critical path**: [03
§5](03-modernization-roadmap.md) W7's exit gate requires this review signed off, and W13's entry gate
requires W7's exit, so nothing downstream of W7 can start until it completes. A defect it finds is a fix
inside W7's own markup and a re-test of the gate rather than a re-plan. Section
[8.2](#82-concurrency-permitted-by-the-graph) places it accordingly, and section
[8.3](#83-the-critical-path-and-what-to-do-first-if-the-goal-is-to-narrow-the-estimate) carries it in
the critical-path column.

**The markup work itself is not here.** Implementing every requirement of 05 §8.9 is **inside W7's own
band** — it is view markup, which is what W7 is — and this section adds nothing to that band. What this
section
sizes is the *review*: establishing that the requirements were met, which is a separate activity with a
separate artifact and a separate reviewer.

**The artifact.** A signed-off record naming the reviewer, the date, each of the 24 requirements as met
or defective, the **four measured contrast ratios with the threshold each was measured against**, and any
defect with a named owner — a d4 failure marked **inherited** rather than introduced. It is the only place
in this plan where an accessibility claim is ever made, and it is made about **requirements met**, not
about conformance to a standard — 05 §8.9 is explicit that this document set claims no conformance.

### 7.4 Repository hygiene — enumerated but intentionally unestimated

[08 §10](08-technical-debt-register.md) inventories the hygiene findings and
[03 §7.5](03-modernization-roadmap.md) records that **F-08-23 through F-08-28** form an independent stream
**gating nothing**.
The three items below are **enumerated** here for completeness and **deliberately left unestimated**, and
they are excluded from [section 6.1](#61-the-totals)'s total, because no workstream's entry gate depends
on them. Calling them *sized* — as an earlier form of this heading and this paragraph did — asserted a
band this section expressly declines to give, and what is unestimated is the **remediation decision** in
each case rather than the finding's severity, which is 08's to state.

**Severity is not uniform across the three, and an earlier form of this paragraph said it was.** For the
second and third rows — F-08-25 and F-08-23 — [08 §10](08-technical-debt-register.md) does record
**Low severity with no migration impact**, and [03 §7.5](03-modernization-roadmap.md) records that they
**gate nothing and may be deferred indefinitely**. The first row is not one of them: it is **F-08-11**,
which [08 §6.2](08-technical-debt-register.md) records in the **Data** category at **High** severity for
the credential stores among those binaries, and [08 §11.1](08-technical-debt-register.md) states in as
many words that this owner's set **"is not uniformly Low"**. It is unestimated here for the same reason
as the other two — the remediation is a history decision — and 09 owns the exposure.

| Item | Finding | Input | Note |
| --- | --- | --- | --- |
| **14** committed database binaries, **43,376,640** bytes (file count and byte count) | **F-08-11** ([08 §6.2](08-technical-debt-register.md)) — **Data, High** | 19 | The engineering act is small; the **decision** is not. Removing them from history is a rewrite affecting every clone, and retention is a repository-owner call, not a migration task |
| **215** committed package files (file count) | F-08-25 ([08 §10](08-technical-debt-register.md)) — Hygiene, Low | 19 | Same character: cheap to stop tracking, consequential to rewrite |
| **4** solution files for **3** projects, one stale (file count) | F-08-23 ([08 §10](08-technical-debt-register.md)) — Hygiene, Low; the same artifact's **build** consequence is F-08-19 and belongs to [10](10-build-and-deployment-requirements.md) | 19 | Resolved as a by-product of W6, which collapses them |

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

**The graph is 03's, at gate granularity, and it is the only graph this section uses.**
[03 §4.2 and §6.1](03-modernization-roadmap.md) carry it — **21 nodes and 56 edges, acyclic, single root
W1, terminals exactly `{W14, W15}`**, with [03 §6.3](03-modernization-roadmap.md) exhibiting a topological
order in which no edge runs backwards. It is not restated here, not re-derived, and no second graph is
constructed for this document's convenience: every set below is a function of that node and edge set and of
the bands of [section 5.1](#51-summary-table).

**The charging model, stated once, because a set partition is only an identity if each figure appears in
exactly one set.** Every band is charged at the node that does the work:

- **A workstream 03 draws as one node is charged whole at it** — W1, W3, W5, W6, W7, W8, W9, **W11**, W12,
  W13, W14 and W15.
- **A workstream 03 draws as several gates is split exactly as [section 5.2](#52-basis-of-estimate-per-workstream)
  splits it, and never re-cut here.** W4 at its two gates, `1 / 2 / 3.5` for **4a** and
  `40 / 69 / 116.5` for **4b**; W10 at its three, `3 / 5.5 / 10` for **10a**, `2.5 / 4.5 / 7.5` for
  **10b** — which carries the `1 / 2 / 3` for the required **manual functional walk** of exit condition 8,
  and is also where exit condition 9's `G-CSP-BROWSER` is performed inside the provisioning band, because
  10b is the first gate at which a deployed application exists to drive — and
  `0.5 / 1 / 1.5` for **10c**; W16 at its two stages, `1 / 2.5 / 6` for **16·1** and
  `2 / 3.5 / 6` for **16·2**.
- **W2 is the one exception, and it is an exception about bands rather than about the graph.** 03 draws
  **2a** and **2b**; §5.2 states W2's band as a whole. It is therefore charged whole at **2a**, and **2b**
  appears in the partition as a node carrying **0** — which is what lets the two gates open different
  successors without either arithmetic double-counting or a band invented for a gate whose basis names none.
- **The three non-code tasks are charged at the node whose exit gate requires them**: the sign-offs of
  [§7.2](#72-the-approved-delta-sign-offs) at **W1**, the baseline capture of
  [§7.1](#71-the-manual-visual-review) at **W4b**, and that task's review and sign-off together with
  [§7.3](#73-the-manual-accessibility-review)'s accessibility review at **W7**. Each appears in one set and
  in one node, and [section 8.3](#83-the-critical-path-and-what-to-do-first-if-the-goal-is-to-narrow-the-estimate)
  uses the same positions rather than a second set of them.

**Each set is the *earliest* point at which its work becomes available**: a node sits in the set
immediately after the one that closes its last entry gate, so `set = 1 + max(set of its predecessors)`. The
set numbers are an **ordering index and nothing else** — no calendar position, no start or finish date, no
duration, no weeks, stages or waves. A reader with a capacity assumption can turn this into a schedule;
this document does not.

**The graph is [03 §4.2.1](03-modernization-roadmap.md)'s and only its — its twenty unconditional
nodes and fifty-six unconditional edges**, the sixteen workstreams with W2, W4 and W10 appearing as
their internal gates and W16 as its two stages. The one conditional node is excluded here for the same
reason [§6.3](#63-what-is-deliberately-not-in-the-total) excludes its band. This section applies effort
to that graph and changes nothing about it. Where an
earlier reading of this section asserted an edge 03 does not carry, 03 governs: **`W9 → W8` does not
exist** — 03 withdrew it explicitly, keeping the load ordering inside W13 where the production load
happens — and **W10 has two gates rather than three**, because the data load is W13's rather than W10's.

**The sets partition the expected total exactly.** Every workstream and all three non-code tasks appear
**once and only once**, so the column below adds to
[section 6.1](#61-the-totals)'s **311.5**. **Five** bands are split across sets, each because 03 draws the
row as more than one node: **W2**, as the two gates **2a**, the recorded verification run, and **2b**, a
passing run re-verified; **W4**, as the two gates **4a**, the build-governance bootstrap, and **4b**,
baseline green and captured; **W10**, as **10a**, provisioning, and **10b**, schema application, which is
W10's exit; **W16**, as its **policy** and **mechanism** stages; and the **manual visual review**, because
[03 §5](03-modernization-roadmap.md) attaches its capture to W4's exit gate and its review to W7's. Each
split shows its parts and each part's expected IED, so no figure appears twice.

**Two apportionments are this section's rather than an owner's, and both are labelled.**
[W10](#w10--hosting-provisioning-and-platform-configuration) publishes a whole band and no per-gate
split, so its 9 expected IED is apportioned **6.5 to 10a and 2.5 to 10b** — provisioning carries the plan,
the database, the identities, the transport, the configuration, the secret delivery and the exit
condition that verifies the sink, while 10b is the schema application alone.
[W2](#w2--mvc-5-build-reproduction-and-the-restore-precondition) likewise publishes a whole band, and it
**has to be apportioned rather than kept whole, because `W2·2a → W2·2b` is an edge**: a set states what
may proceed alongside what, so two nodes one of which waits on the other cannot both sit in it. Its
**2 / 4 / 9** is apportioned **1.5 / 3 / 3.5 to gate 2a** and **0.5 / 1 / 5.5 to gate 2b** — 2a is a single
recorded run from a clean checkout, whose width is the restore's and no more, while **2b carries the
repair loop**, which is where W2's high case actually lives: a defect triaged and the run re-verified
under 2a's own conditions. The parts sum to the row [section 5.1](#51-summary-table) publishes in all
three columns — `1.5 + 0.5 = 2`, `3 + 1 = 4`, `3.5 + 5.5 = 9` — and the expected split is the one
[section 8.3](#83-the-critical-path-and-what-to-do-first-if-the-goal-is-to-narrow-the-estimate) already
itemizes off the path as `3 + 1 = 4`.

**Each set is the *earliest* point at which its work becomes available**, so a node appears in the set
immediately after the one that closes its last entry gate — all of them, not the first of them. That is
why three of W1's six direct successors do **not** appear in set 1: W3 waits on W16's policy stage, W5 on
W3, and W11 on W7 and on W10a.

| Set | Work available concurrently | Expected IED in the set | Gate that opens it |
| --- | --- | ---: | --- |
| **0** | **W1** (7) and the **approved-delta sign-offs** of [§7.2](#72-the-approved-delta-sign-offs) (**+16**), which are required *by* W1's exit gate and are not inside W1's band | 23 | The root. Nothing precedes it |
| **1** | **W2's gate 2a**, the recorded verification run whatever it reports (3), **W16's policy stage**, conditions 1–3 (2.5), and **W4's gate 4a**, the build-governance bootstrap (5) — three independent streams | 10.5 | W1 exited, and for each of the three that is its **only** entry ([03 §6.1](03-modernization-roadmap.md) rows 1, 2 and 4): gate 2a runs from a clean checkout and consumes nothing but W1's authority; gate 4a authors and proves the governance files and the contracts project and consumes no legacy build; the policy stage needs an approver and nothing built |
| **2** | **W2's gate 2b**, a passing run re-verified under 2a's conditions (1), **W3** (4) and **W6** (4.5), independent of each other | 9.5 | Gate 2b: **gate 2a closed**, which is its only entry ([03 §6.1](03-modernization-roadmap.md) row 7). W3: **W16's policy stage exited**, because W3 attaches the committed credential and catalog databases and that is the roadmap's first processing of real personal data. W6: gate 4a closed **and W2's gate 2a** — the recorded starting condition, pass or fail. Gate 2b and W6 are both successors of 2a and are **not** joined to each other, so they belong in one set |
| **3** | **W5** (2) and **W4's gate 4b** (62.5) with the **baseline capture** of [§7.1](#71-the-manual-visual-review) inside it (5.5) | 70 | W5: **W3 exited** — [03 §6.1](03-modernization-roadmap.md) row 12 carries `W3 → W5`, and it is the later of W5's two entries, the other being W1 at row 5, so the casing audit reads the extracted schema's own object names rather than guessing them. Gate 4b: **three direct** conditions — [03 §6.1](03-modernization-roadmap.md) rows 18, 11 and 13 — gate 4a closed, **W2's gate 2b** (a passing run, because 4b drives the running legacy application), and **W3 exited**, because its fixture manifest publishes counts, ranks, quantities and order totals derived from W3's extraction. **W16's policy stage is a fourth ordering condition reached transitively rather than a fourth edge**: 4b's runs restore both committed store pairs, and 03 declines to declare `W16·1 → W4·4b` because `W16·1 → W3 → W4·4b` (rows 9 and 13) already closes stage 1 before 4b can open |
| **4** | **W7** (122.5) and **W10a**, provisioning (6.5), independent of each other | 129 | W7: W3, W5, W4 through gate 4b, and W6 exited. W10a: **W5 exited**, its only entry — not one of provisioning's artifacts consumes an application artifact, so it does not wait on the port |
| **5** | The **review and sign-off** of [§7.1](#71-the-manual-visual-review) (7) and the **manual accessibility review** of [§7.3](#73-the-manual-accessibility-review) (4.5), concurrent with each other | 11.5 | W7's port work complete against the set-3 baseline. Both **close** W7's exit gate, so no W7 successor opens until both are signed off |
| **6** | **W11** (19.5), alone in its set | 19.5 | **Four** entries, all of which must be closed: W1 for the provider decision, W6 for the converted project, **W7 exited** — which includes set 5 — and W10a for the deployment principal |
| **7** | **W10b**, schema application and W10's exit (2.5) | 2.5 | W10a closed, W7 exited **and W11 exited**: the DDL runs from the rehearsed release stage rather than an operator's session |
| **8** | **W16's mechanism stage**, conditions 4–6 (3.5) | 3.5 | Its policy stage, W3, W7, W11 **and W10b** — the governed access path and its auditing read a sink that does not exist until the schema is applied |
| **9** | **W8** (8) and **W9** (8), concurrent and **deliberately unordered** | 16 | Both: W3, W7, W11 and all six W16 conditions; W9 additionally W5. 03 carries **no edge between them** — they restore different stores into different databases, and the ordered pair a reader looks for is inside W13 |
| **10** | **W12** (9.5) | 9.5 | W7, W11, W10b **and W8**, whose rehearsal neutralizes the account W12's credential-repair path is proven against |
| **11** | **W13** — the convergence point (4) | 4 | W7, W11, W10b, W16's mechanism stage, W8, W9 and W12 all exited |
| **12** | **W14** (2) and **W15** (1), both terminal and concurrent with each other | 3 | W13 exited; W14 additionally needs W7 and W15 additionally W10b |
| | **Total** | **311.5** | |

**The reconciliation, printed so it can be checked rather than trusted:**
23 + 10.5 + 9.5 + 70 + 129 + 11.5 + 19.5 + 2.5 + 3.5 + 16 + 9.5 + 4 + 3 = **311.5**, which is
[section 6.1](#61-the-totals)'s expected total. Set 0's 23 is 7 + 16; set 1's 10.5 is 3 + 2.5 + 5;
set 2's 9.5 is 1 + 4 + 4.5; set 3's 70 is 2 + 62.5 + 5.5; set 4's 129 is 122.5 + 6.5; set 5's 11.5 is
7 + 4.5; set 9's 16 is 8 + 8; set 12's 3 is 2 + 1. W2's two gates sum to 3 + 1 = **4**, W4's two
gates to 5 + 62.5 = **67.5**, W10's two to 6.5 + 2.5 = **9** and W16's two stages to
2.5 + 3.5 = **6** — the values
[section 5.1](#51-summary-table) carries for those rows — and the visual review's two parts to
5.5 + 7 = **12.5**, the value [§7.1](#71-the-manual-visual-review) carries for its row.

**No overlap is added to this partition, and that is deliberate.** Where two sets can proceed with some
temporal overlap — gate 10a running on alongside W7 is the clearest case, since nothing in 10a consumes an
application artifact — the overlap is in **elapsed time**, not in effort. Effort does not shrink when
two streams run at once, so an overlap annotation inside an effort column would not reconcile against
any total. Overlap belongs to
[section 8.3](#83-the-critical-path-and-what-to-do-first-if-the-goal-is-to-narrow-the-estimate)'s
on-path versus off-path split, which is where it actually has an effect.

**Seven properties of this shape are worth stating explicitly.**

- **Set 4 is the heaviest set by effort and set 3 the second heaviest, and each is dominated by one item.**
  Set 4 is **129** expected IED of which 122.5 is a single workstream; set 3 is **70** of which 62.5 is a
  single gate. Sets 1 and 2 are the widest by count at three items each, holding **10.5** and **9.5**
  expected IED across three items apiece — the two places in the plan where the graph permits the most
  independent streams, which is a statement about **width**, not about how long either set takes.
  **After set 4 the plan is almost entirely serial**: sets 6 to 12 hold **58** expected IED in
  eleven items, only two of which are ever concurrent with anything.
- **This round the weight moved by `+3.5` expected IED and the membership not at all.** Gate 4b goes from
  61.5 to **62.5** and W7 from 119.5 to **122.5**, both following the coverage matrix to 104 rows and
  326 cases, so set 3 goes from 69 to **70** and set 4 from 126 to **129**; and **set 0 goes from 23.5 to
  23**, the sign-off row re-pricing on
  [05 §11.5](05-aspnet-core-migration-approach.md)'s re-count while W1 stands still.
  `1 + 3 − 0.5 = 3.5`, and
  `308 + 3.5` = **311.5**. No member arrived, left, or changed set, which is why the shape below is the shape
  the previous round produced.
- **The round before that changed the plan's shape as well as its weight.** Two members arrived that were
  absent from the partition altogether — **W16's two stages** and the **manual accessibility review** —
  and one arrived that had been treated as effort inside another row, the **approved-delta sign-offs**
  ([§6.1.1](#611-the-walk-from-the-previously-published-total) records that defect). **The weight moved
  by exactly +50 expected IED, and it decomposed by member rather than by set index**, since the indices
  below set 1 all shifted: the three arriving rows contributed `16.5 + 6 + 4.5 = 27`, and five members that
  were already here re-banded by `8 + 3.5 + 1 + 0.5 + 10 = 23` — the manual visual review from 4.5 to 12.5
  across its two parts, W12 from 6 to 9.5, W1 from 6 to 7 as its own act census gained a decision, and
  then gate 4b from 61 to 61.5 and W7 from 109.5 to 119.5 as the coverage matrix reached 102 rows.
  `27 + 23 = 50`, and `258 + 50` = **308**. Every other member carried the figure it carried before.
- **Three structural corrections came from 03's inventory rather than from any band, and each moved a
  member between sets.** **W11 is one node, not two** — an earlier reading split it and placed its
  provider half in the widest set, but 03 draws a single node with four entries, so the whole of its 19.5
  waits for W7 and W10a and sits alone in set 6. **W10 has two gates, not three**, so the data-load gate
  this section used to carry is gone, its work being W13's. And **W8 and W9 are concurrent**, because the
  `W9 → W8` edge is withdrawn: they now share set 9 rather than occupying two.
- **W2's exit has two states, and they open different members of later sets.** W6 consumes gate **2a**,
  because a recorded failure is still a known starting condition to convert from; W4's gate **4b**
  consumes **2b**, because it drives the legacy application over HTTP and a build that produces no
  running application leaves it nothing to characterize. So the two are **not opened by the same event**,
  and they no longer fall in the same set either: W6 opens in set 2 while gate 4b waits on W3 and opens in
  set 3. A gate condition reading "W2 exited" for both would overstate what W6 needs and understate what
  4b needs. **The two gates themselves also sit in different sets, and must**: `W2·2a → W2·2b` is
  [03 §6.1](03-modernization-roadmap.md) row 7, so 2b waits on 2a and the two cannot be members of one
  concurrency set. 2a opens in set 1 on W1 alone and 2b in set 2 on 2a alone, which is why the row is
  apportioned above rather than carried whole.
- **W4 does gate W6 — at gate 4a, not at gate 4b — and the distinction is the whole of why they are still
  concurrent with the heaviest gate rather than behind it.** **An earlier reading of this document asserted
  that the graph carried no `W4 → W6` edge at all, and that is withdrawn: `4a → W6` exists.** What
  survives is the conclusion, on a different justification — **W6 consumes the build-governance
  bootstrap and not the baseline**, so it does not wait on a legacy application that runs, on a captured
  baseline or on a green suite. W6's own exit
  gate deliberately requires neither a build of the legacy application nor a test run of it — the
  *unported* application cannot compile on the target framework, so a suite could not exercise it. Gate 4b
  therefore gates **W7 only**, and it still cannot be parallelized *internally* in proportion to its case
  count, because its deployment lifecycle, its store reset, its fixture dataset, its normalization, its
  diagnostic records and its handoff artifact are one coherent
  artifact that all **152** of its cases sit on top of — the cases divide, the thing they run on does not.
  **That property survives every re-derivation intact**: the non-case components of
  this workstream are now **44 of its 67.5** expected IED — two thirds of it — and the divisible part grew
  by only 1 this round, because **14** of the 19 cases the matrix gained are target-only and fall to W7
  rather than here.
- **Two nodes are available far earlier than an effort model would guess, and both are worth exploiting.**
  **W10a** needs W5 and nothing else, so provisioning is available in set 4 alongside the port rather than
  after it; and **W16's policy stage** needs W1 and nothing else, so it is available in set 1 — which
  matters because it gates the two heaviest predecessors in the plan, W3 and gate 4b. Neither is on the
  critical path, and leaving either until its consumer asks for it converts an off-path item into a wait.
- **W5 is not the free-standing audit its size suggests.** [03 §6.1](03-modernization-roadmap.md) row 12
  carries `W3 → W5`, so the casing
  audit follows the extraction, and W5 in turn gates **W7**, **W9** and **W10a** — three consumers for
  2 expected IED. It is the cheapest node in the plan with more than one successor, and it sits between
  the extraction and everything that provisions or ports.

### 8.3 The critical path, and what to do first if the goal is to narrow the estimate

**The longest dependency chain**, computed as a longest path **by weight** over
[03 §4.2.1](03-modernization-roadmap.md)'s canonical edge inventory — its **twenty unconditional nodes
and fifty-six unconditional edges**, acyclic, single root W1, terminals exactly `{W14, W15}`; the one
conditional node and its two conditional edges are excluded, because they exist only on the secondary
hosting path — is

> **W1 → W16·1 → W3 → W4·4b → W7 → W11 → W10·10b → W16·2 → W8 → W12 → W13 → W14**

where `W16·1` and `W16·2` are the personal-data policy and mechanism stages, `W4·4b` is the
baseline-green-and-captured gate and `W10·10b` is the schema-application gate that is W10's exit — every
one of them a node 03 declares in exactly that form. The graph was checked acyclic by topological sort
before any weight was applied, because a longest path is only defined on a graph that has one, and 03
§6.3 exhibits the order.

**These figures are chain effort, and nothing in this section is an elapsed time.** Every number below is
work content in ideal engineer-days ([§4.2](#42-the-unit-defined)) summed along **one chain**, in which
each node genuinely waits on the one before it. **"Critical path" here names the longest chain by weight
and nothing more.** It is not a duration: it has no start, no finish and no calendar position
([§8.1](#81-the-order-is-03s-the-concurrency-is-this-documents-contribution)), and a reader wanting one
has to supply a team size, a utilization factor and a parallelism assumption, all three of which
[§4.2](#42-the-unit-defined) declines to invent. **Concurrency therefore changes nothing about the
arithmetic and everything about the reading.** Effort is additive over *every* node however the edges
run — which is why [§6.1](#61-the-totals)'s model total simply adds the sixteen workstreams to the two
manual reviews and the sign-offs, and why
[§8.2](#82-concurrency-permitted-by-the-graph) legitimately adds its concurrent members' bands into a
per-set total and those totals back to **311.5**. What concurrency does **not** license is reading either
sum as elapsed time: a set's total is the work its members contain, not how long the set takes, and a
chain's total is the weight of one route through the graph, not a completion date. On that reading the
two sections say compatible things — a chain sum orders work, a set sum measures it, and neither is a
schedule.

**It is the same chain in all three columns.** Low, expected and high were maximized **independently**
over the same graph rather than taken from the expected column and re-weighted, because a wide band on an
off-path row can in principle overtake a narrow on-path one, and where that happens the chain is a
property of the column rather than of the model. Here it does not happen: the twelve nodes above are the
argmax at low, at expected and at high alike. That is recorded as a checked result, not assumed — if a
later reconciliation moves a band far enough to change it, the three chains have to be re-derived
separately and stated separately.

**Four steps of this chain are decided by something other than the obvious, and each is stated because a
reader checking the path will land on exactly these.**

- **W16·1 → W3, not W1 → W3.** W3 has two predecessors and the policy stage is the longer route into it:
  `7 + 2.5 = 9.5` expected IED against W1's 7 alone. The personal-data policy is therefore **on** the
  critical path even though it is an approval, because W3 cannot attach a copy of the real credential
  store before it closes.
- **W3 → W4·4b, not W4·4a → W4·4b.** Gate 4b has **three** direct predecessors — gate 4a, W2's gate 2b
  and W3, which are [03 §6.1](03-modernization-roadmap.md) rows 18, 11 and 13 — and the binding one is W3
  at a cumulative **13.5** expected IED (`7 + 2.5 + 4`) against gate 4a's **12** (`7 + 5`) and gate 2b's
  **11** (`7 + 4`). **Gate 4a is therefore off the path**, which is
  the one place in this model where part of a workstream on the chain is not itself on it. W16's policy
  stage reaches 4b as well, but **transitively through W3**: 03 §4.2.1 states that `W16·1 → W4·4b` is not
  declared because `W16·1 → W3 → W4·4b` already imposes the ordering, so it is not a fourth predecessor
  and it is not a fourth route to weigh.
- **W8, not W9 — and the earlier reading of this section had the reason wrong twice over.** It said W8 was
  taken "because W8's band is the wider of the two", and then, after a re-derivation, that W9's was. Neither
  holds now: **the two bands are identical at 8 expected IED**. The step is decided entirely by what
  *follows* it. [03 §6.1](03-modernization-roadmap.md) rows 50 and 51 give W8 two successors, `W8 → W12`
  and `W8 → W13`, while row 52 gives W9 only
  `W9 → W13` — so the chain through W8 continues `W8 → W12 → W13` and weighs `8 + 9.5 + 4 = 21.5`
  expected IED against W9's `8 + 4 = 12`. **There is no `W9 → W8` edge**: 03 withdrew it explicitly, so the
  two are concurrent and W9 is simply off the path.
- **W14, not W15, is the terminal.** Both are opened by W13 and neither feeds anything, so the chain takes
  the heavier: documentation revision at 2 expected IED against affinity retirement at 1.

**Which stage halves are charged.** 03 declares **W16·1 and W16·2**, **W4·4a and W4·4b**, **W2·2a and
W2·2b** and **W10·10a and W10·10b** as separate nodes and states that **no dependency in the roadmap is
partial**, so the graph itself says which half binds and this section needs no convention to choose one.
The chain charges **W10's schema-application gate** and leaves **its provisioning gate off**, because
10b's binding predecessor is W11 rather than 10a; and it charges **W4's gate 4b** while leaving
**gate 4a** off, for the reason stated above. W11, by contrast, is a **single node** in 03's inventory
with four entries, so the whole of its band is on the path — an earlier reading of this section split it
and charged only a manifest part, and that split does not exist in the graph.

Three gate conditions sit **on** the chain without being workstreams of their own, and all three are
counted below: the approved-delta sign-offs ([§7.2](#72-the-approved-delta-sign-offs)), which W1's exit
gate requires; and both the visual review and sign-off ([§7.1](#71-the-manual-visual-review)) and the
manual accessibility review ([§7.3](#73-the-manual-accessibility-review)), which are two conditions of
**W7's** exit gate — concurrent with each other, but no W7 successor opens until both are signed off. The
visual **capture** is **not** on the chain: it sits inside gate 4b's window as concurrent work rather than
as a step 4b waits on, so it appears in the off-path itemization instead.

| | Low | Expected | High |
| --- | ---: | ---: | ---: |
| Critical path, workstream nodes only | 138.5 | 247.5 | 431 IED |
| — plus the delta sign-offs on W1's gate | 8 | 16 | 26 IED |
| — plus the visual review and sign-off on W7's gate | 4 | 7 | 11.5 IED |
| — plus the accessibility review on W7's gate | 2.5 | 4.5 | 8 IED |
| **Critical path, gate-inclusive** | **153** | **275** | **476.5** IED |
| Off the critical path | 18.5 | 36.5 | 69 IED |
| **Model total** | **171.5** | **311.5** | **545.5** IED |

**The chain's own addition, printed so it can be checked:** at the expected band
`7 + 2.5 + 4 + 62.5 + 122.5 + 19.5 + 2.5 + 3.5 + 8 + 9.5 + 4 + 2` = **247.5**; at the low band
`3.5 + 1 + 2 + 36 + 69.5 + 11 + 1.5 + 2 + 4 + 5 + 2 + 1` = **138.5**; at the high band
`13.5 + 6 + 8 + 105.5 + 208.5 + 35.5 + 5 + 6 + 16 + 16 + 8 + 3` = **431**. Adding the three gate
conditions: `247.5 + 16 + 7 + 4.5` = **275**, `138.5 + 8 + 4 + 2.5` = **153** and
`431 + 26 + 11.5 + 8` = **476.5**.

**The two rows reconcile exactly against [section 6.1](#61-the-totals):** `153 + 18.5 = 171.5`,
`275 + 36.5 = 311.5`, `476.5 + 69 = 545.5`. So **36.5 of the 311.5 expected IED — 11.7 percent — is work
content that sits off the chain**, and the remaining 275 is work content that lies along it. What is off
it is **eight** items: W2's two gates (`3 + 1 = 4`), **W4's gate 4a** (5),
W5 (2), W6 (4.5), **W10's provisioning gate** (6.5), W9 (8), W15 (1) and the **5.5 of baseline
capture** — itemized low, expected and high,
`2 + 2.5 + 1 + 2.5 + 3.5 + 4 + 0.5 + 2.5 = 18.5`, `4 + 5 + 2 + 4.5 + 6.5 + 8 + 1 + 5.5 = 36.5` and
`9 + 8.5 + 4 + 8.5 + 11 + 15 + 2 + 11 = 69`, which is the same off-path row read from its parts rather than
from the subtraction. **Every one of the twenty unconditional nodes is in exactly one of the two lists** — the
twelve on the chain and the eight off it, counting W2's two gates and W4's and W10's split halves once
each, plus the three gate conditions and the capture — which is what makes the reconciliation an identity
rather than a residual.

**The one reconciliation this section still owes, and it is small now.** An alternative reading charges a
staged workstream **whole** wherever any part of it binds. Under 03's current inventory only one row is
affected, because gate 4a and W10's provisioning gate are the only off-path halves of on-path rows: adding
W10's provisioning gate gives **156.5 / 281.5 / 487.5** gate-inclusive, exceeding the figure above by
exactly `3.5 / 6.5 / 11`. Charging it to the chain would place work the graph makes available in set 4,
two sets before its consumer, onto a chain it does not lie along — the opposite of what the graph says.
**The staged figure is the critical path**; the whole-workstream figure survives only as this
reconciliation. Gate 4a is left off under both readings, since no reading of W4 puts a predecessor of the
binding gate onto the path.

**What moved this chain, relative to the reading this section previously carried.** **Membership did not
move at all this round** — the twelve chain nodes and the eight off-path ones are the same twelve and eight
— and **only two of the twelve re-banded**: gate 4b goes 61.5 → **62.5** expected and W7 goes
119.5 → **122.5**, both following the coverage matrix to 104 rows and 326 cases. `1 + 3 = +4` expected,
`0.5 + 1.5 = +2` low and `1.5 + 4 = +5.5` high, so the workstream-nodes row moves
`243.5 + 4 = 247.5`. **One gate condition also moved, and it moved downward**: the delta sign-offs go
16.5 → **16** expected as [§7.2](#72-the-approved-delta-sign-offs) re-derives them on
[05 §11.5](05-aspnet-core-migration-approach.md)'s re-count, so the gate-inclusive row moves by
`+4 − 0.5 = +3.5`, `271.5 + 3.5 = 275` — which is exactly the movement in
[§6.1](#61-the-totals)'s total, `308 + 3.5 = 311.5`. The other two columns net the same way,
`+2 − 0.5 = +1.5` low and `+5.5 − 0.5 = +5` high.
**The off-path row therefore does not move**: it stands at 18.5 / 36.5 / 69 because both re-banded nodes
**and** the re-priced gate condition are on the chain, which is the sharpest available check that the
movement was allocated to the path
correctly rather than absorbed into the residual.

**The round before that moved three of the twelve and one of the three gate conditions**, and its
arithmetic is kept for the same reason: gate 4b
went 61 → **61.5** expected as its case sub-row followed the coverage matrix, W7 went 109.5 → **119.5** as
its target-facing case sub-row did the same, and W12 went 7.5 → **9.5** for input 32's eight
published-console rows. `0.5 + 10 + 2 = +12.5` expected, `0.5 + 6.5 + 1.5 = +8.5` low and
`1 + 16.5 + 3.5 = +21` high, which was exactly the movement in the workstream-nodes row of that round —
`231 + 12.5 = 243.5`. The **delta sign-offs** then moved 11 → **16.5** on W1's gate as
[§7.2](#72-the-approved-delta-sign-offs) read the two approval registers at their present size, so the
gate-inclusive row moved by `12.5 + 5.5 = +18` — `253.5 + 18 = 271.5` — which was exactly the movement in
[§6.1](#61-the-totals)'s total then, `290 + 18 = 308`.

**The round before that one moved the chain's membership, and that record is kept because the graph it
describes is the graph above.** Five changes, none of
which invented work, and three of which came from
[03 §4.2.1](03-modernization-roadmap.md)'s own corrections rather than from any band:

- **W11 joined the chain whole, and its provider gate stopped being a separate off-path item.** 03 draws
  one node with four entry edges — W1, W6, W7 and W10a — so all **19.5** of its expected IED is on the
  path rather than 16.5 of it. This is the single largest contributor to the movement in the gate-inclusive
  figure.
- **W9 left the chain and W8 stayed on it, for a reason that is now structural rather than arithmetic.**
  With `W9 → W8` withdrawn the two are concurrent, and W8 is on the path only because `W8 → W12` exists.
  W9's 8 expected IED moved into the off-path itemization, where it is the largest single entry.
- **W10 lost a gate.** The data-load gate this chain used to pass through is gone — 03 puts the production
  load in W13 — so the chain passes through **10b** and nothing else of W10, and 10a's provisioning is
  off-path.
- **W16's policy stage joined the chain, and the mechanism stage stayed on it.** An earlier reading had
  stage 1 off the path on the ground that W4's binding predecessor outweighed it. That is still true of
  gate 4b, but stage 1 is on the path by a different route: it is the binding predecessor of **W3**, which
  is the binding predecessor of gate 4b. **2.5** expected IED moved onto the path, and it is the cheapest
  node on it.
- **The accessibility review joined the chain as a third gate condition.** It closes W7's exit gate
  alongside the visual review ([§7.3](#73-the-manual-accessibility-review)), so its **4.5** expected IED
  is on the path; and the **sign-offs** joined it as the condition on W1's gate when
  [§7.2](#72-the-approved-delta-sign-offs) established that they are required by that gate rather than
  contained in W1's band. The band they carry there is the one the table above prints — **8 / 16 / 26** —
  and [§6.1.1](#611-the-walk-from-the-previously-published-total) records the movement that
  brought it, rather than this bullet restating a figure of its own.

**If the objective is to narrow the estimate rather than to shorten it, the first substantive action is
W3.** It costs **4 expected IED** and it is the input to **all four** of the other low-confidence rows —
[§4.4](#44-confidence-and-its-reason)'s Low set is W3, W4, W8, W9 and W16, and 03's row 4 gives W3 the
successors `W4b`, `W7`, `W16·2`, `W9` and `W8`:

- It replaces the **evidence-rather-than-proof** qualification on the Identity column set
  [12 §5](12-migration-blockers.md) with fact, which is what W8's band is wide because of.
- It supplies the diff baseline W9's hard entry gate requires, in a repository where the migration
  source **ships no schema script**.
- It supplies gate **4b**'s fixture manifest with the counts, keys, ranks, quantities and order totals it
  publishes — which is why W4, the second-heaviest node on the path, cannot start before it.
- It settles **W16 stage 2**'s governed field list as queried fact rather than as a source-level inference.
- It resolves two of the 23 blockers outright (input 12).

**W3 sits in concurrency set 2 and has exactly one prerequisite besides approval: W16 stage 1.**
It cannot begin before the personal-data policy is approved, because its entry requires the committed
credential and catalog databases to be **attached** — real personal data, at the earliest point in the
plan. That does not weaken the recommendation; it extends it by one item, and both items are **on the
critical path**, which makes the recommendation stronger rather than merely cheaper. Stage 1 costs
**2.5 expected IED**, needs nothing but W1 and is an approval rather than engineering, so the cheapest
available reduction in the *width* of the total is to open stage 1 and W3 as a pair:
`2.5 + 4` = **6.5** expected IED that unblocks four low-confidence rows, blocks nothing else while it
runs, and sits at
the chain's own front end, so taking it first displaces nothing else along the chain. Neither makes the project smaller; both make the estimate truer, which is a different and often
more valuable thing.

**The second action is W2, and it narrows a different thing.** W2 costs `3 + 1` = 4 expected IED across
its two gates, and only part of it is off the path: **gate 2a is on the chain at low and expected**, being
the step by which those two columns reach W6, and off it at high, while **gate 2b is off it in every
column**. Its high band is 9 — the widest low-to-high ratio of
any small row in the model — because
[10 §1.2](10-build-and-deployment-requirements.md) carries the migration source's build as **blocked
pending a Windows verification run** and nobody yet knows what that run reports. Running it early converts
an unknown outcome into a known one at a cost the graph already permits to run in parallel with W3.

**And the ordering is not negotiable in the other direction.** Sequencing W3 ahead of stage 1 to save the
wait would mean the first copy of real credential data is made before anyone has said under what
restriction it may be held, for how long, or with what destruction evidence — which is the failure the
gate exists to prevent, arriving at the cheapest-looking moment.

### 8.4 The sequencing hazard in the visual baseline capture

**One sequencing hazard is priced in a band but becomes disproportionately expensive if missed.** The
visual baseline capture ([§7.1](#71-the-manual-visual-review)) must happen **while the legacy application
still runs**. It is **5.5 expected IED** inside W4 — concurrency set 3 — and it is not on the critical
path, because gate 4b's 62.5 expected IED of suite authoring dominates the set it shares.

**The reason it cannot simply be done later is drift, not deletion.** The artifact does not cease to exist:
[04 §12](04-dotnet8-migration-strategy.md)'s target map **retains all three legacy editions read-only** —
**MVC 5 as the behavioural baseline, MVC 4 and MVC 3 as historical and comparative references** — so the
*source* of the thing being photographed is still in the repository after cutover. What decays is the
ability to **run** it, and that is a different claim with different evidence:

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
That is worse for planning, not better: there is no deadline to miss, only a slope. Taking it at gate 4b
costs the 3.5 expected IED above; reconstructing a legacy runtime afterwards costs an unbounded amount for a
baseline that is, by then, arguably not the one the port was measured against. It remains the one item in
the plan whose *omission* is not repaired by spending more later.

---

## 9. The risk register

### 9.1 How to read an entry

**The register carries eighteen entries, `R1` through `R18`.** That is its whole membership: the
identifiers are contiguous with no gap and no repetition, [§9.2](#92-register-index) lists all eighteen in
ascending order, and [§9.3](#93-the-entries) states each one in full. Two further risks — `I1` and `I2` —
are **held elsewhere** and deliberately not entries here; §9.2 signposts both with their owner and where
each is stated in full, so a reader who finishes this register knows they exist. **Of the eighteen, six are
approval decisions rather than engineering mitigations**, and
[§9.4](#94-the-six-risks-that-are-approval-decisions-not-mitigations) names those six. The count is stated
here because a register whose size a reader has to infer by counting headings cannot be audited against a
gate that requires every entry to have been considered.

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
index cell does not follow. So the set is not judged per entry. It is **derived**, by a rule with two
parts that are both checkable from the document:

1. **Field closure.** The **Affected workstreams** field must name every `W<n>` that appears in any of the
   entry's other six canonical fields — likelihood, impact, mitigation, contingency, trigger and owner. If
   a mitigation leans on a workstream's coverage, or a contingency runs through a workstream's pipeline, or
   a trigger fires at a workstream's gate, that workstream is affected by definition, and the field says so
   with **the role it plays** rather than the identifier alone. Which appearances create a membership at
   all is the three-kind distinction below.
2. **Index identity.** The **index cell in [§9.2](#92-register-index) is the same set**, in ascending
   order. The index is a projection of the entries, never a second statement of them — the same rule
   [03 §4.2.1](03-modernization-roadmap.md) applies to its edge inventory, and for the same reason.

Consumers and gates enter through part 1 rather than through a separate rule, because an entry that
depends on a consumer says so in the field that depends on it — and where that was not previously true,
the fix was to the *field*, not to the index. Where a consumer relationship is a graph fact rather than a
textual one, the field cites the edge: R2's field names **W4 and W6 as W2's two direct successors** in
03 §4.2.1's inventory, which is a lookup rather than an opinion.

**Three kinds of mention, and only the first two are members.** A `W<n>` can appear in a canonical field
for three different reasons, and the two-part rule above is checkable only once they are told apart. The
distinction is stated here, once, rather than judged per entry — which is the habit that produced the
divergences the paragraph above records.

1. **An affected member.** A canonical field assigns it a role: it performs the mitigation, absorbs the
   contingency, owns the entry, or is where the trigger fires. It appears in the **Affected workstreams**
   field **with that role**, and in the [§9.2](#92-register-index) index cell.
2. **A conditional member.** It is a member only if a branch is selected, so the field and the index both
   carry it **with its condition attached**, never bare. [R17](#r17--the-interim-hosting-paths-stored-credential-and-a-time-box-with-no-box)
   is the whole-entry case — its field opens *"None while the interim path is not selected"* and names W7
   and W13 only for the branch where it is — and
   [R18](#r18--the-browser-executed-half-of-the-scripted-cart-flow-one-pinned-engine-and-an-open-decision-on-the-other-two)'s
   **W14** is the single-identifier case, a member on the acceptance and manual-walk outcomes and not on
   the third. The index states the condition too, because a cell carrying W14 bare would assert a
   membership the branch has not created.
3. **An exclusion-only mention, which is not a member.** A `W<n>` named in a field **solely to place it
   outside the risk** is not affected by it. It is written in the fixed form **`W<n>` is unaffected**,
   followed by the reason; it stays in the **field**, because the disclaimer is what stops a reader
   inferring the membership, and it is **absent from the index cell**, because the index carries members.
   The register contains exactly one: **R18's W4** — *"W4 is unaffected — the flow is target-only, so the
   legacy half executes no browser"*.

**A correction to an earlier revision, recorded rather than quietly applied.** That revision declared
[R16](#r16--no-security-relevant-action-is-recorded-anywhere)'s **W16** an exclusion-only mention and
therefore absent from R16's index cell — while R16's index cell named it, so the rule and the row
contradicted each other. **The row was right.** R16's field does two things with W16: it scopes W10's
canary proof *away* from the personal-data access-audit records, **and** it affirmatively assigns those
records to **W16 stage 2**, per [03 §5 W16](03-modernization-roadmap.md). An affirmative assignment is a
role, so W16 is a member of R16 under kind 1, it belongs in the field and in the index, and it is in both.
The genuine exclusion-only mention in this register is R18's W4.

**The check is mechanical, and these are its terms.** Extract the `W<n>` identifiers from an entry's seven
field rows and from its §9.2 index row, taking each identifier from the **rendered text** — an anchor
target such as `(#w11--…)` is a link destination, not a second mention — drop any identifier carried in
kind 3's exclusion-only form, and compare the sets. Two comparisons must both come back empty: no
identifier in the six other fields is missing from the **Affected workstreams** field, and no identifier
differs in either direction between that field and the index cell. Run against the register as it now
stands, both are empty for all eighteen entries, and four fields were extended to make that true —
**R1** gained W1 and W4, **R2** gained W1 and W7, **R7** gained W1 and **R8** gained W4, each with the
role that field already relied on — while **R2**'s and **R18**'s index cells gained the member their
fields had always named.

**The index's other columns are projections on the same terms, and one of them had drifted.** Its
Likelihood and Impact cells carry the entry's own values, and all eighteen agree. Its Owner cell carries
the *(approval decision)* marker for exactly the six entries
[§9.4](#94-the-six-risks-that-are-approval-decisions-not-mitigations) lists — a correspondence that
section states as a checkable property of this one. **R18's cell was missing the marker** while §9.4
named R18 as its sixth row, [§9.1](#91-how-to-read-an-entry) counted six, and
[§7.2](#72-the-approved-delta-sign-offs) priced R18's decision as one of the **four** escalated risk
decisions that are not themselves approved deltas. Three places said one thing and the index said
another; the cell now carries the marker, and the marker set, §9.4's rows and the count of six are one
statement in three projections rather than three statements.

### 9.2 Register index

| ID | Risk | Likelihood | Impact | Owner | Affected workstreams |
| --- | --- | --- | --- | --- | --- |
| [R1](#r1--the-target-framework-support-window) | The target framework's support window | **Certain** | High | Engineering leadership *(approval decision)* | W1, W4, W6, W7, W10, W11 |
| [R2](#r2--the-migration-sources-build-reproducibility) | The migration source's build reproducibility | Medium | Medium | Build and release engineering | W1, W1, W1, W2, W4, W6 |
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
| [R16](#r16--no-security-relevant-action-is-recorded-anywhere) | No security-relevant action is recorded anywhere | **Certain** *(of the source)* | High | Security, with platform engineering | W7, W8, W10, W12 |
| [R17](#r17--the-interim-hosting-paths-stored-credential-and-a-time-box-with-no-box) | The interim hosting path's stored credential, and a time-box with no box | **Medium** *(conditional on the interim option being taken)* | **High** | **Security** | **None unless the interim path is selected**; if it is, W7 and W13 gate its retirement |
| [R18](#r18--the-browser-executed-half-of-the-scripted-cart-flow-one-pinned-engine-and-an-open-decision-on-the-other-two) | The browser-executed half of the scripted cart flow | **Certain** *(of the residual)* | **Low** | **Engineering**, with **Product** owning the open decision *(approval decision)* | **W1** for the recorded outcome; W7 for the harness; W11's manifest half; **W14 conditionally**, on the acceptance or manual-walk outcomes |

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

**The key `R17` is not in use, and it is not reassigned.** The subject a reader might expect under it —
the stored credential the interim path introduces — is **I1** below, held in full by
[06 §13.4](06-azure-hosting-recommendations.md). The key is left unused rather than given to another risk
so that no reference to R17 can resolve to a different one; the register's entries are therefore
**R1–R16 and R18**, seventeen in all.

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
| **Affected workstreams** | **W1**, whose approval is where the decision is taken or is silently omitted; **W1** as the approval that must record the decision, and the workstream whose silent exit is the trigger below; W6 and W7 directly; **W4**, whose suite is the contingency's acceptance evidence; W10 and W11 through the runtime and build image; and **W4**, whose suite is the contingency's acceptance evidence when a retarget is executed |

#### R2 — The migration source's build reproducibility

**The one edition the entire port depends on carries no discharged build verification, and this
entry says so rather than softening it.** [10 §1.2](10-build-and-deployment-requirements.md) — the owner
of per-edition build outcomes — carries the migration source's build assessment as **blocked pending a
Windows verification run**. This entry cites that status and restates neither the diagnosis behind it
nor any per-edition detail.

**What is established, and what is not, in the owner's terms.** Established: a **precondition
failure**. A clean checkout of the migration source commits no restored packages, so no build can start
until a restore succeeds against a source the repository never declares. Also established, as
observation rather than as gate discharge: a Windows host carrying the prescribed toolchain restored the
migration source and rebuilt it in Debug and in Release, both exiting `0` with zero warnings and zero
errors [10 §5.4](10-build-and-deployment-requirements.md). **Not discharged: the verification run
itself.** [10 §3.2](10-build-and-deployment-requirements.md) records that run as **supplementary
observation** which does not discharge the gate, is not a build from a clean checkout, and does not
retire this entry; and the Mono `xbuild` result in
[10 §3.1](10-build-and-deployment-requirements.md) is explicitly *supplemental portability evidence
only* and is not a statement about a build on the prescribed toolchain; no row of either may be read as
a discharged gate.

**The risk therefore has four parts, and the first two are where it is open rather than residual:**

- **The verification run may fail, and its outcome is not settled.** W2 produces that run
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
| **Likelihood** | **Medium**, and the question is **open rather than unrepeated**: the migration source's build gate has not been discharged, and the one prescribed-toolchain rebuild on record is supplementary observation taken outside the conditions W2 must record ([10 §3.2](10-build-and-deployment-requirements.md)). It is Medium rather than High for a bounded reason — the two configuration defects [10 §1.2](10-build-and-deployment-requirements.md) records are in a *different* edition, and the migration source's own observed failure was resolution of absent packages rather than anything in its source. Medium rather than Low because that hypothesis is untested |
| **Impact** | **Medium.** A defect found at W2 is cheap; the same defect found during W6 is disruptive but recoverable. It is not High because W2 sits before every line of port work |
| **Mitigation** | W2 executes the run from a clean checkout with an **explicitly declared** restore source, in **Debug and Release both**, and records tool versions, the source resolved, configurations built, per-edition outcome and warning and error counts — the five fields [10 §3.2](10-build-and-deployment-requirements.md) sets as its exit criterion |
| **Contingency** | Any defect the run reveals is triaged inside **gate 2b's repair loop** before W2 exits, and W6 may proceed from the 2a record while that loop runs. A defect therefore delays rather than derails on the W6 side, and holds W4 rather than merely delaying it, because W4 consumes 2b; the port has not started either way. If the failure is in the migration source's own configuration rather than in its restore, W6's band moves to its high case and [R3](#r3--the-absent-regression-baseline) gains weight, because W4 needs a legacy application that not only builds but **runs** — its `Category=Baseline` half executes against the running application on this host ([05 §12.10](05-aspnet-core-migration-approach.md), assumption **A2**), so a host that compiles the source and cannot serve it leaves W4's gate unreachable rather than merely delayed. If the defect proves **irreparable within 2b's bound**, the contingency is not inside this model: [03 §5](03-modernization-roadmap.md) escalates to W1 and this document is **re-estimated rather than adjusted**, as [section 6.3](#63-what-is-deliberately-not-in-the-total) treats every input it does not assume |
| **Trigger** | **The run reports anything beyond the known restore precondition** — any error or warning that survives a successful restore. Secondarily: a build that succeeds on one host and fails on another. Thirdly, and it fires by silence: W6 beginning while W2's five-field 2a record is incomplete, or **W4 beginning on a recorded failure**, which is the reading gate 2b exists to forbid |
| **Owner** | **Build and release engineering** |
| **Affected workstreams** | W2, and its two consumers at the gate each actually needs — **W6 through gate 2a**, **W4 through gate 2b** ([03 §5](03-modernization-roadmap.md)). **W1** as the escalation route the contingency names for a failure irreparable within 2b's bound. **W7** because the view boundary this entry binds onto the run reaches it: with view compilation disabled, the ported views' first target-side check is W7's fixtures and the manual visual review — plus **W1**, which is where [03 §5](03-modernization-roadmap.md) escalates a defect this model carries no band for. **W7 is named here only to place it outside the entry**: W2 sits before every line of port work, which is the reason this entry's Impact is Medium, and no field of it depends on the port — plus **W1**, to which a defect irreparable inside 2b's bound escalates and from which this document is re-estimated rather than adjusted |

#### R3 — The absent regression baseline

**This is the register's Critical-impact entry, and it is the risk that determines whether any
behaviour-preservation claim can be substantiated at all.** [08 §12.3](08-technical-debt-register.md)
asks this document to carry it in exactly those terms.

**The repository contains no test of any kind** — no test project, no test file, no test-framework
reference, repository-wide. The command is in [A.2](#a2-the-absences-that-size-the-net-new-work) and
returns zero. **So nothing that exists today would detect a behaviour change.**

**What makes it Critical rather than merely High is the interaction with the blocker classification.**
[12 §2.3](12-migration-blockers.md) separates 23 blockers into 14 that fail at compile time and **9
that fail silently**. The compile-time group is self-announcing — the build stops. The silent group is
not: the request succeeds, the page renders, and a navigation property reads empty or a JSON field
reads undefined. **Against that group, a code review is not evidence.** The only instrument that
detects them is an assertion that fails when the resolution is absent, which is why
[03 §5](03-modernization-roadmap.md) makes W7's exit gate *demonstration* rather than inspection.

**One property of the required coverage sharpens this entry rather than softening it, and it is the
reason this risk is extended here instead of a nineteenth entry being opened.** **65 of the 104 coverage
rows are target-only, carrying 161 of the 326 cases, and 174 cases in all execute against the target
fixture alone** (input 14) — a case share that has grown at every extension of the behaviour set — and for
those rows the suite is **not a comparison at all**: there is no legacy shape to characterize, only a
defect to reproduce, so the fixture pair cannot tell a correct assertion from a plausible one. **Among
them are the concurrency interleavings, fault injections and contention bounds**, which
[05 §12.4](05-aspnet-core-migration-approach.md) enumerates and this document does not re-count, and whose
absence no
functional assertion notices — a cart that merges correctly under no contention passes every other row
in the table. **The
consequence is that for a majority of the rows — and for more than half of the cases, `174 ÷ 326` — the
specification *is* the baseline**, and a
specification that is wrong is undetectable by the mechanism this entry relies on. That does not add a
risk; it says where this one bites hardest, and it is what makes the second clause of the trigger below
matter — those are the rows most easily deferred under schedule pressure, because nothing fails when
they are missing.

| Field | Value |
| --- | --- |
| **Likelihood** | **High.** Not that the baseline is absent — that is a verified fact — but that a behaviour change ships undetected if the port proceeds without one. Eight blockers are specifically silent |
| **Impact** | **Critical.** Without a baseline there is no evidence for the migration's central premise, and a regression's first detector is a user |
| **Mitigation** | W4 precedes the port, as [03 §4.1](03-modernization-roadmap.md) sequences it. The suite is HTTP-level and semantic so that **one suite characterizes both runtimes**, with volatile values normalized out and every approved delta recorded as an expected difference rather than a failure. For the **65** target-only rows, where no baseline exists to compare against, the mitigation is **review of the assertion against its owning contract in [05](05-aspnet-core-migration-approach.md)** rather than against the legacy fixture — the only check available for a row the source fails by construction |
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
its in-window re-verification — happens **before** the stop. **Reasoning in the reverse direction — from an
extract taken first and a cutover later — describes a delta design nobody chose**, and it is named here
because it is the intuitive one: it would budget for rows the drain prevents while leaving the
unsanctioned write the drain does *not* prevent unchecked. Either way the decision is **W9's to make and record** rather
than an operational detail discovered in the window, and getting it wrong loses orders — which is why this
entry's impact is Critical.

**Where the reversal boundary falls, because the short statement of it is only half true.** The retained
source does make rollback "a redirection to the source rather than a restore" — but only until traffic is
admitted, and left there unqualified that sentence is wrong on the far side of that line.
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

**There is no signed-in-cart exception, and the warrant that appears to grant one does not reach.**
Treating cart changes as "enumerated and reported but do not trip the regime" would rest on
[06 §11.4](06-azure-hosting-recommendations.md)'s approved anonymous-cart loss, and that approval does not
extend that far: the AAP's approved-delta set grants cart loss for **anonymous** carts only and
states that signed-in carts are unaffected because their key is the login name
[src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:165-167]. A regime B reversal that returned to the legacy
store while a signed-in user's post-admission cart change stayed behind in the new one would destroy data no
approval covers. **So a signed-in cart write is decisive like every other accepted write, and it has exactly
one relief**: [06 §11.5.3](06-azure-hosting-recommendations.md)'s **six-step reverse replay** carries the
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
signal** ([06 §11.5.2](06-azure-hosting-recommendations.md)). Neither is complete alone, and 06's own
six-class table says which one carries each class:

- **Signal 1 — the audit stream**, read from the workspace's own unsampled console-channel records
  ([06 §9.2.5.1](06-azure-hosting-recommendations.md)) rather than from the sampled telemetry path — there is
  no audit table to read, per [06 §11.5.4](06-azure-hosting-recommendations.md) — which says what changed,
  who changed it and **whether the application confirmed it** — the
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
| **Mitigation** | W3 first, so the diff has an authoritative baseline. The **generated-schema diff must pass before any data is loaded** — [03 §5](03-modernization-roadmap.md) calls this the hard gate. Load in dependency order; reconcile row counts per table **and financial totals per order**; rehearse the whole sequence against a representative dataset per assumption **A3**. And the **reversal instrument is built before the window rather than improvised inside it**: the accepted-write evidence of [06 §11.5](06-azure-hosting-recommendations.md) rests on the change tracking **and the delete-visible system-versioned history** that [05 §5.1](05-aspnet-core-migration-approach.md)'s **two** per-context `AddChangeTracking` migrations add — one in the catalog set and one in the Identity set, because a migration belongs to exactly one context — and on the watermark [06 §11.3](06-azure-hosting-recommendations.md) records at traffic admission; it is evaluated against the loaded copy as part of the rollback W9's exit gate requires to be *performed* rather than described, **and the reverse replay of [06 §11.5.3](06-azure-hosting-recommendations.md) is rehearsed there in both directions of its step-6 branch rather than only in its passing one, which is what [06 §11.5.4](06-azure-hosting-recommendations.md) requires of the rehearsal**; and the forward ladder's second rung — the last known-good revision redeployed against an unchanged database — is exercised once inside W11's release-path rehearsal. **That rung's availability** depends on [06 §6.7](06-azure-hosting-recommendations.md)'s additive-within-a-release constraint — a release that dropped or renamed a column the previous revision reads would make it unusable — so the constraint is a requirement on every migration W9 authors rather than a style preference |
| **Contingency** | **Three regimes, and which one applies is established by evaluating the change-tracking evidence rather than by judging how long ago traffic was admitted** ([06 §11.5](06-azure-hosting-recommendations.md)). **Before traffic admission**, the source database is unmodified and retained, so reversal is a redirection to the source rather than a restore, and reconciliation failure stops the cutover outright — it does not get accepted with a note. **After admission while that evidence shows no accepted write that has not been carried back**, the same redirection is still available and **no business data is lost, signed-in cart changes included**: where the evidence finds signed-in cart writes, [06 §11.5.3](06-azure-hosting-recommendations.md)'s **verified reverse replay** is run first — enumerate the affected keys from current rows **and** the `Carts` history table, replay each key's final state into the source inside one transaction, verify the multiset per key — and **its step 6 decides the regime**: all keys verified admits regime B, any key unverified or the replay unable to run is regime C. Only **anonymous** cart writes are reported and not replayed. **From the first accepted write, recovery is roll-forward only, with no exception under any sign-off**, by [06 §11.5](06-azure-hosting-recommendations.md)'s four rungs in order: stop admitting new work; redeploy the last known-good revision against the same database; repair the data or schema forward through the release path under the deployment principal; and, **if the target itself is damaged, point-in-time restore of the target followed by a replay of the interval between the restore point and the damage**. That replay is not a property of the restore: PITR reproduces the state at `T`, so the writes after `T` are exactly the rows the restored database does not have. Its source is the **retained damaged original**, which PITR never overwrites and which system versioning leaves holding full-state history — so **retaining, locking and not dropping the damaged database is a step of the procedure** ([06 §11.5.4](06-azure-hosting-recommendations.md) part 4a), and the audit stream supplies only the confirmed-or-not attribution the rows cannot carry. Where that replay cannot run — history destroyed with the current rows, or an interval older than the retention period — the recovery point for the affected table is `T` and the residual is a **declared data-loss event under a procedure and an RPO the data owner approves at approval time**, never one proposed during the incident. **Returning to the legacy application is not a rung of that ladder at all, at any level of authorization.** That ladder has four rungs and no fifth — not even one gated on the data owner's sign-off and a reverse-delta export — because a sign-off does not make discarded confirmed orders recoverable, and a rung of that kind is the one most likely to be reached for under pressure |
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

> **The contingency is a deployment, and the reading that says otherwise is the one to guard against.**
> "The persisted policy is configuration rather than code, so correcting it does not require a deployment
> of the application" is not the design being estimated:
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
> why W11 is in the index set above.
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
| **Mitigation** | **Obtain the product owner's explicit approval in W1**, as one of the delta decisions of [§7.2](#72-the-approved-delta-sign-offs). Support the decision with **actual client analytics** if any exist, so it is taken against evidence rather than assumption. State the matrix in the deployment documentation (W14), so an unsupported client generates a policy answer rather than a defect investigation |
| **Contingency** | If the product owner declines, the decision to be reversed is the **styling-framework major version** in [04 §9](04-dotnet8-migration-strategy.md) — not this matrix, which is downstream of it. That reversal carries its own consequence, which [09](09-security-assessment.md) owns: remaining on an out-of-support framework. There is no contingency that keeps both |
| **Trigger** | W1 completing without the product owner's recorded decision on this delta. Post-cutover: support contacts from unsupported clients, which the matrix in the deployment documentation is what converts into an answer |
| **Owner** | **The product owner**, alone. Explicitly not engineering, and explicitly not this document |
| **Affected workstreams** | **W1** as the approval that must record the product owner's decision, **W1**, where the product owner's decision is recorded and whose silence is this entry's trigger; **W1**, where the product owner's decision is recorded or is missed by silence; W7 as the implementation; W14 as the statement of the matrix |

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
| **Mitigation** | W5 audits **all** of it — the **11** bundling helper sites, the **4** `@Url.Content` sites and the **27** static assets (inputs 8, 11 and 7; site and file counts) plus view paths — and, critically, makes the audit **repeatable as a pre-deployment check** rather than a one-time sweep, because a new mismatch can be introduced by any later change. **Coverage row 23** of [05 §12.4](05-aspnet-core-migration-approach.md) asserts that static assets resolve **case-sensitively**, which a case-insensitive check would pass wrongly, and its owner classifies it as **mixed** rather than target-only — case **`23a`**, every rendered reference requested at its rendered casing, runs against both fixtures, while case **`23b`**, a deliberately wrong case which must **404**, is target-only — and **[03 §5](03-modernization-roadmap.md) is what places it**, at W10's exit gate, which is where [06 §3.4](06-azure-hosting-recommendations.md)'s **G-CASING-SERVE** closes. This document assigns the row to no gate and defines neither casing gate. The split matters to this risk, because `23b` is the case that detects a case-insensitive agent, and an agent that passes `23a` while failing to reject `23b` is the exact condition this entry describes |
| **Contingency** | A mismatch reaching production is corrected in the referencing code and redeployed; the repeatable check is then extended to cover the class that escaped it. Because the obligation closes against **two** criteria [06 §3.4](06-azure-hosting-recommendations.md) defines — **G-CASING-STATIC**, every enumerated literal resolving against [05 §8.1.1](05-aspnet-core-migration-approach.md)'s asset inventory with nothing deployed, and **G-CASING-SERVE**, the runtime serve coverage row 23 asserts — the correction is verifiable at each of them rather than hopeful |
| **Trigger** | Any 404 for a static asset or view on a case-sensitive serve — which is why the check must run on a case-sensitive filesystem in the pipeline and not on a developer machine |
| **Owner** | **Engineering** |
| **Affected workstreams** | **W4**, which authors and runs the legacy-side casing case, **W4**, which authors the row and runs case `23a`'s `Category=Baseline` half; W5 as the audit that discharges **G-CASING-STATIC**; W7 as the place its corrections are applied during the asset relocation; **W10**, whose 10a entry is the hosting precondition W5's exit satisfies and whose exit gate is where the row's target half executes and **G-CASING-SERVE** closes; W13 as the point of exposure — W7 and W10 are W5's own downstream edges in [03 §5](03-modernization-roadmap.md) — and **W4**, which authors and runs the mixed case ``23a`` whose legacy side has to pass at its rendered casing before the target-only ``23b`` can mean anything |

#### R9 — Cutover re-authentication and anonymous-cart loss

**Both costs are accepted deliberately by [05 §11](05-aspnet-core-migration-approach.md), which owns
the cutover decision. This entry carries them as risks with mitigations and contingencies. It does not
reopen the decision.** They are the two cutover rows of the delta register in
[05 §11.5](05-aspnet-core-migration-approach.md) — forced re-authentication, and anonymous carts not
carrying across — with product and operations as their approval owners.

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
[06 §11.5.3](06-azure-hosting-recommendations.md)'s verified reverse replay, and where that replay does not
verify the reversal becomes **regime C** rather than a reversal with a larger accepted loss. The two cutover
deltas grant the sign-out and the **anonymous** cart, and nothing here extends them —
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
| **Owner** | **Product and operations**, as the approval owners named by [05 §11.5](05-aspnet-core-migration-approach.md)'s two cutover deltas — forced re-authentication, and anonymous carts not carrying across |
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
exists on the target framework** [12 §4](12-migration-blockers.md). Its data layer therefore cannot
be ported without re-targeting the provider outright.

**It is not a compile-time blocker**, and the classification is the owner's rather than this
document's reading of it.
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
consequence is which detector applies, not how severe it is**: a compile-time reading would have
this blocker announced by a build nobody has to run against MVC 3, whereas the true detector is *any
execution that touches the catalog* — which is exactly what assumption **A7** guarantees will not happen
while that edition stays a read-only reference.

**Under assumption A7 this affects nothing**, because that edition is not ported — it is retained
read-only as a **historical and comparative reference**, which is not the same thing as a baseline and is
stated separately for that reason. **The behavioural baseline is the migration source and only the
migration source**: [03 §5](03-modernization-roadmap.md) records MVC 5 as the sole executable
behavioural baseline and the only edition W4's suite drives, so no case in
[05 §12.4](05-aspnet-core-migration-approach.md) executes against this edition or against MVC 4, and
neither of them is captured, reset or driven by anything in this plan. The entry exists so that the
*conditional* is recorded rather than discovered: **if** that edition ever enters scope, this is a re-target and not a port, and its
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

#### R15 — Personal-data governance is unowned

**The risk is that a migration lands nine personal-data fields into a hosted, internet-facing database
with no retention period, no deletion path and no access audit — and that the migration is what creates
the exposure rather than what inherits it.**

[09 §3.11](09-security-assessment.md) owns the finding. The order record carries `FirstName`
[src/MVC5/MvcMusicStore/Models/Order.cs:23], `LastName` [src/MVC5/MvcMusicStore/Models/Order.cs:28], `Address` [src/MVC5/MvcMusicStore/Models/Order.cs:32], `City` [src/MVC5/MvcMusicStore/Models/Order.cs:36], `State`
[src/MVC5/MvcMusicStore/Models/Order.cs:40], `PostalCode` [src/MVC5/MvcMusicStore/Models/Order.cs:45], `Country` [src/MVC5/MvcMusicStore/Models/Order.cs:49], `Phone` [src/MVC5/MvcMusicStore/Models/Order.cs:54] and `Email` [src/MVC5/MvcMusicStore/Models/Order.cs:61] — nine fields, linked to an
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
[src/MVC5/MvcMusicStore/MvcMusicStore.csproj:285], [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:19], which describes one engineer's machine and not
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
per producer, because **fifteen** of its sixteen classes have three producers between them: the
**thirteen** the ported application
emits are a named sub-row of W7 in [section 5.2](#52-basis-of-estimate-per-workstream), the Identity
migration's `AUTHZ-3001` sits inside W8's band and the command's two inside W12's. **The sixteenth is
sized nowhere because nothing in this plan produces it** — the privilege-withdrawal class
[09 §6.8.1.1](09-security-assessment.md) keeps reserved with no producer, which makes it an open gap
rather than an omission from this estimate. **Proving
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
| **Mitigation** | Three-part, and each part has an owner: [09 §6.8.1](09-security-assessment.md) specifies the catalog — every event class with its identifier, actor, target, outcome, severity and permitted fields — and **§6.8.1.1 the producer map**, which is what makes the gates below scoped rather than unpassable; [03 §5 W7](03-modernization-roadmap.md) makes emitting **the thirteen application-produced classes** an **exit condition of the port**, together with the redaction tests that prove no forbidden field appears and the three named seams without which three of the thirteen have no emission site at all, while **two** of the remaining three are gated at W8 and W12 where their producers exist and the third is gated nowhere, because nothing produces it; and **verified arrival in the sink at the stated retention is an exit condition of two workstreams rather than one** — [03 §5 W10](03-modernization-roadmap.md) condition 7 proves the collection path end to end with **a single canary class, `AUTH-1002` at `AccountNotFound`**, chosen because its producer needs no catalog row, no account and no administrator and is therefore drivable on the empty schema that workstream provisions, and [03 §5 W12](03-modernization-roadmap.md) condition 7 demonstrates **the remaining twelve deployed**, from the fixture population and the executable that can drive them. The split across the sixteen — including the one class with no producer, which no gate above may demand — is [09 §6.8.1.1](09-security-assessment.md)'s and the per-producer destinations are [06 §9.5](06-azure-hosting-recommendations.md)'s producer matrix, neither of which this document re-derives. The log-privacy policy of [06 §9.2](06-azure-hosting-recommendations.md) is what keeps the audit trail from becoming a second copy of the personal data it is auditing — the actor is a pseudonymous identifier, not a login name |
| **Contingency** | If a producer's assigned classes are incomplete at that producer's gate, the missing classes are completed before the gate closes rather than deferred to a hardening pass — an audit trail with gaps is trusted as though it had none, which is worse than a known absence. If collection cannot be verified at W10, the sink is treated as unproven and the gate fails. **The failure mode this contingency does not cover, and the producer map is what removes it:** a gate that demanded all sixteen classes from the port would be failed by a correct implementation, and the likely response is a placeholder record emitted from the application for an event the application never observes |
| **Trigger** | W7's exit approached with any of its **thirteen** classes unemitted, with any of the three seams absent, with any outcome value unexercised, or with a redaction test absent; W8's or W12's exit approached with the classes the producer map assigns them unemitted, at the wrong record cardinality, or with their destination or its **export into the audit store** unverified; W10's exit approached with the sink configured but never verified against the canary class; **W12's exit approached with the deployed twelve-class census incomplete, or with the fixture rows it drove left behind unremoved**; or a retention period for security-event records shorter than the period over which incidents are actually discovered |
| **Owner** | **Security**, with platform engineering for the collection path |
| **Affected workstreams** | W7 as the emission of the **thirteen** application-produced classes; **W10 as the collection path, proved by one canary class only** — not by the thirteen, and not by the personal-data access-audit records, whose emission and proof of arrival are **W16 stage 2's alone** per [03 §5 W16](03-modernization-roadmap.md) even though they share a destination; **W16 stage 2** for those records; **W8 as the sole producer of the migration's `AUTHZ-3001` records**; and **W12 as the producer of `PROV-6001` and `AUTHZ-3001`** at the cardinality [06 §9.5](06-azure-hosting-recommendations.md) row 5 fixes: four `PROV-6001` on a provisioning run, one per operation and on every path, plus one `AUTHZ-3001` where a membership is actually added. Those two land in a different destination — the captured pipeline-job artifact, exported into that section's audit store of record rather than into the application's sink. **The catalog's privilege-withdrawal class is affected by no workstream at all**, because nothing in this plan withdraws a role: [09 §6.8.1.1](09-security-assessment.md) keeps its identifier reserved with no producer and records that gap as open and carried by no workstream, so this entry names none for it and this document sizes none |

#### R17 — The interim hosting path's stored credential, and a time-box with no box

**This entry exists because the decision it tracks is a security-owner decision on entering the interim
path**, and [06 §5.5](06-azure-hosting-recommendations.md) places that decision with the security owner
and specifies exactly what the approval must name —
[§9.4](#94-the-six-risks-that-are-approval-decisions-not-mitigations) states why it is that gate rather
than W1's. It is not routed here by 06: [06 §13.4](06-azure-hosting-recommendations.md) keeps its own two
interim entries, a boundary [§9.2](#92-register-index) records and this document does not reverse. That
document recommends **Path A** for the interim option —
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
mode easy to reach rather than exotic — the exception's expiry is defined **as an event and deliberately
not as a date** (the gate the exception record names), and an event can slip indefinitely, the interim
step is by construction chosen by people
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
| **Likelihood** | **Medium**, and the two halves must be stated separately. That a credential **exists** is *certain* for the life of the interim step, because it is what Path A is; that the exception **outlives its intended end** is Medium, and it is that second thing this entry tracks. Medium rather than Low because the expiry is anchored on an event that can slip, and rather than High because [06 §5.5](06-azure-hosting-recommendations.md)'s five controls are mandatory before the step begins — the exception record must name the gate that ends it, and its rotation control carries an expiry alert — while [06 §5.3](06-azure-hosting-recommendations.md) requires the exception to be re-reviewed at each rotation rather than at approval only |
| **Impact** | **High.** The login reaches the application's data, which includes the credential store and order data containing personal information — the nine PII fields of the checkout form [09 §3.11](09-security-assessment.md). It is High rather than Critical because the grant is data-plane only — no `db_owner`, no DDL rights, no server-level role — so a leaked credential is a data exposure rather than a schema or platform compromise |
| **Mitigation** | **All five of [06 §5.5](06-azure-hosting-recommendations.md)'s controls**, which that document owns and specifies and this entry names rather than restates, in force before the step begins and not after: a **least-privileged data-only login**; the secret held in the platform key store and **resolved by reference** so it is never in source, in the repository or in the deployment payload; **rotation performed by a named operator under an active PIM activation and deliberately not automated**, on 06's enforced interval and by the outage-safe sequence of [06 §5.5.1](06-azure-hosting-recommendations.md), with an expiry alert — an earlier form of this cell required automated rotation, and 06 withdraws the automated rotator by name because the only component that could perform it would have to hold `ALTER ANY USER` in both interim databases; the **connection string rewritten rather than repointed**; and the exception **recorded and time-boxed by an event, not by a date** — the record names the gate that ends it, and [06 §5.3](06-azure-hosting-recommendations.md) states the two events that close it. 06 §5.5 states the interim step is acceptable *only* with all five in place, so an approval that omits any one of them has not time-boxed anything |
| **Contingency** | On suspected exposure: rotate immediately, revoke the login, and audit the data-plane access it held. On the gate named in the exception record slipping, or on that record reaching a scheduled review with no movement on the gate: the exception is **re-approved explicitly or the interim hosting is withdrawn** — it does not lapse into being permanent by default. If the organization cannot accept a stored credential at all, 06 §5.5's Path B is the documented alternative and the interim step then carries a code approval by necessity, which is a different decision and a different estimate |
| **Trigger** | **A rotation completed with no re-review of the exception recorded** — [06 §5.3](06-azure-hosting-recommendations.md) requires the exception to be re-reviewed at each rotation rather than at approval only, so a rotation that leaves the record untouched is the earliest observable sign that the box has gone missing, and it is the trigger to watch because it fires while everything still looks healthy. 06's own definitive signal is later and unambiguous: the gate named in the exception record completing while the credential still exists. Secondarily: that record still open at a scheduled review with W7 not exited; 06 §5.5's expiry alert firing; the login's grants widening beyond data-plane access; or the credential appearing in a local configuration file, a log or a deployment payload |
| **Owner** | **Security**, as [06 §5.5](06-azure-hosting-recommendations.md) places the decision — the security owner approves the exception, sets the event that ends it, and re-reviews it at each rotation. A named operator executes the rotation under an active PIM activation; the operator does not own the exception, and no automated principal performs it |
| **Affected workstreams** | **None while the interim path is not selected.** If it is: the exception's retirement is gated on the port's completion, which makes W7 and W13 the workstreams whose slippage extends this risk, and its removal is a task attached to the same gate |

#### R18 — The browser-executed half of the scripted cart flow: one pinned engine, and an open decision on the other two

**This entry is open, and that is stated first because a previous revision of this document recorded it as
settled.** That revision read [05 §12.11](05-aspnet-core-migration-approach.md)'s pinning of a Chromium
harness as closing the entry, withdrew it from
[section 9.4](#94-the-six-risks-that-are-approval-decisions-not-mitigations) and left that table at five
rows. **That reading was wrong and is withdrawn here.** Pinning the harness settled the *scope* question —
whether any browser executes this flow at all — and settled it affirmatively: 05 §12.11 pins a
browser-automation harness driving **Chromium** for exactly one flow,
[04 §7.7](04-dotnet8-migration-strategy.md) pins the package as one of the six independent test-tooling
pins and adds the browser-install step to the target-test runbook, and this model prices both — the flow
in [W7](#w7--the-aspnet-core-port)'s browser sub-row and the install step in
[W11](#w11--ci-provider-selection-then-pipeline-authoring)'s manifest half. What that
did **not** settle is the question this entry actually carries: **which engines receive a functional
assertion.** Chromium receives one. **Gecko and WebKit receive none**, and no automated case reaches them.

**So the entry returns to [section 9.4](#94-the-six-risks-that-are-approval-decisions-not-mitigations) as
its sixth row, because [03 §5](03-modernization-roadmap.md)'s W1 now carries this residual as a mandatory
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
| **Contingency** | The decision itself is [W1](#w1--approval-of-this-assessment)'s and is listed in [section 9.4](#94-the-six-risks-that-are-approval-decisions-not-mitigations); the contingency is what each outcome costs. **Accepting** the residual costs nothing beyond the recorded acceptance. **Extending** the automated engine set to Gecko, WebKit or both is a **re-estimation**: W7's browser sub-row grows per engine, the target-test job gains an install step per engine, and the diagnostic surface gains per-engine artifacts — none of which these bands carry. A **named manual functional walk** is also a re-estimation, in W14's governance work, and is an accepted limitation rather than coverage. What is **not** available under any outcome is pointing [§7.1](#71-the-manual-visual-review)'s appearance review at this gap, per [R3](#r3--the-absent-regression-baseline)'s rule |
| **Trigger** | Any of: **W1 closing without a recorded decision on this residual**, which is the condition the entry's return to section 9.4 exists to prevent; a defect found in production in the cart-removal script on a non-Chromium browser after the port, which is this residual realized; the harness proving unstable in the pipeline, which converts a coverage gain into a flaky gate and is [R3](#r3--the-absent-regression-baseline)'s failure mode arriving through this entry; or an attempt to close the residue on the other engines by citing the appearance review |
| **Owner** | **Engineering** for delivering the pinned harness inside W7 and W11, and the **product owner** for taking the W1 decision among the three outcomes — the same owner who holds the browser matrix itself under [R7](#r7--the-narrowed-browser-matrix) and [06 §10.4](06-azure-hosting-recommendations.md), because deciding which engines get functional evidence is a decision about which clients are supported to what standard. Quality engineering advises; it does not decide |
| **Affected workstreams** | **W1**, which must record one of the three outcomes at its exit gate. **W7**, which carries the harness as a 3.5-expected-IED sub-row, and **W11's manifest half**, which carries the browser-install step at 0.5 expected. **W4 is unaffected** — the flow is target-only, so the legacy half executes no browser. **W14** if the outcome is a recorded acceptance documented with the matrix, or a manual functional walk's checklist and sign-off |

### 9.4 The six risks that are approval decisions, not mitigations

Most entries above are managed by engineering action. **Six are not**: their correct resolution is a
decision by a named owner, and no amount of engineering diligence substitutes for it.
[03 §5](03-modernization-roadmap.md)'s W1 exit gate requires a recorded decision on each. The six are
exactly the six rows of [§9.2](#92-register-index) whose owner cell carries *(approval decision)*, which
is what makes this table checkable against the index rather than a second opinion about it.

**Two of the six are themselves approved deltas, and that changes which row pays for the decision rather
than whether it is one.** [R7](#r7--the-narrowed-browser-matrix) and
[R9](#r9--cutover-re-authentication-and-anonymous-cart-loss) are entries of
[05 §11.5](05-aspnet-core-migration-approach.md)'s register, so their decisions are priced per delta in
[§7.2](#72-the-approved-delta-sign-offs). The other **four** — R1, R13, R18 and R15 — are not deltas, and
[W1's act census](#w1--approval-of-this-assessment) charges them as four of its eleven approval acts. The
partition is what keeps the two rows free of overlap; membership of this table is unaffected by it.

| Risk | The decision, and who takes it |
| --- | --- |
| [R1](#r1--the-target-framework-support-window) | Confirm the target framework knowing its support end date, or retarget before W6 — **engineering leadership** |
| [R7](#r7--the-narrowed-browser-matrix) | Accept the loss of a class of client, or reverse the dependency decision behind it — **the product owner** |
| [R9](#r9--cutover-re-authentication-and-anonymous-cart-loss) | Accept re-authentication and anonymous-cart loss as the cost of a single cutover — **product and operations** |
| [R13](#r13--one-database-one-blast-radius) | Accept a shared blast radius for operational simplicity — **platform and operations** |
| [R15](#r15--personal-data-governance-is-unowned) | Set the retention periods, the non-production copy rules and the legal-hold process, and accept them as the gate on the production data load — **the data owner**, with security and legal |
| [R18](#r18--the-browser-executed-half-of-the-scripted-cart-flow-one-pinned-engine-and-an-open-decision-on-the-other-two) | Choose one of three outcomes for the **Gecko and WebKit functional residual**: accept it, extend the automated engine set, or add a named manual functional walk — **the product owner** |

**Six rows, in ascending identifier order, and the count is checkable against the register itself:** the
six are **R1, R7, R9, R13, R15 and R18**, and they are exactly the six entries whose Owner cell in
[§9.2](#92-register-index) carries the *(approval decision)* marker. Every other entry there does not, so
the two statements cannot drift apart without one of them visibly contradicting the index.| [R18](#r18--the-browser-executed-half-of-the-scripted-cart-flow-one-pinned-engine-and-an-open-decision-on-the-other-two) | Acknowledge that the **required** four-engine manual walk of the cart-removal flow is point-in-time evidence and **not** repeatable automated coverage, so the residual survives it — **the product owner**. Whether the walk happens is not part of the decision; [06 §10.4](06-azure-hosting-recommendations.md) requires it |

**Presenting any of these six as a mitigated engineering risk would be the register's most misleading
possible error**, because it would imply the work can proceed correctly without a decision that only
somebody else can take.

**[R18](#r18--the-browser-executed-half-of-the-scripted-cart-flow-one-pinned-engine-and-an-open-decision-on-the-other-two)
is the sixth row of the six, and a previous revision of this document wrongly removed it.** That revision reasoned
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

**[R17](#r17--the-interim-hosting-paths-stored-credential-and-a-time-box-with-no-box) is deliberately not
a seventh row here**, and the distinction is worth stating because it also turns on someone else's
decision. These six are decisions W1 must record for the **primary** migration to proceed correctly at
all. R17's decision gates the **optional interim hosting path**, which is not one of
[03 §5](03-modernization-roadmap.md)'s workstreams and which the primary path never requires; its
approval is a **security-owner gate on entering that path**, taken at that point rather than at W1, and
[06 §5.5](06-azure-hosting-recommendations.md) specifies exactly what that approval must name. Listing it
above would imply the port cannot begin without a decision about a path it does not use.

### 9.5 Why the enlarged scope added no nineteenth entry

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
| The fixture's private legacy deployment lifecycle, now including startup-quiescence polling and a captured process id, and the published fixture dataset with its per-entity counts, fingerprint and seven post-load invariants ([05 §12.3](05-aspnet-core-migration-approach.md)) | [R3](#r3--the-absent-regression-baseline), with [R2](#r2--the-migration-sources-build-reproducibility) | Both are **determinism controls** on the baseline R3 exists to obtain. The lifecycle additionally requires a host that **runs** the legacy application, which R2 already carries and R3's mitigation already depends on |
| The baseline record's eleven gating `baselineSource` values, its seven separately recorded and never-compared `targetRun` facts, the **committed** `baseline-reference.json` that makes accepting a baseline reviewable, the `coverage` completeness object, the digest-sidecar artifact transfer, the fail-closed compatibility gate and the diagnostic-versus-record publication policy ([05 §12.10](05-aspnet-core-migration-approach.md)) | [R3](#r3--the-absent-regression-baseline), with [R2](#r2--the-migration-sources-build-reproducibility) | A gate that **refuses** a mismatched record is the determinism control working, not a new hazard. Its consequence — a re-capture on the platform that alone can produce one — is R2's dependency, stated there |
| The three SQL identities, the durable ownership registry with its JSON sidecar and its identifier re-resolution before every `DROP`, the standalone orphan-sweep class with its always-run cleanup job, the portable fault injection and the three-layer timeout model ([05 §12.8](05-aspnet-core-migration-approach.md)) | [R3](#r3--the-absent-regression-baseline) | These are **safety and isolation controls** on the fixtures. The destructive risk to *real* data is [R4](#r4--domain-data-migration-rollback) and [R13](#r13--one-database-one-blast-radius)'s and is unchanged by them |
| The second diagnostic schema, the sanitized exception projection, the closed set of twenty-six `operation` codes, the redactor's own corpus, and the keyed one-way pseudonyms whose corrected scheme invokes the pinned normalizer, with their key custody and destruction ([05 §12.9](05-aspnet-core-migration-approach.md)) | [R3](#r3--the-absent-regression-baseline) | R3's mitigation already requires failures to be **diagnosable without disclosing what they saw**; these specify how, and [06 §9.5](06-azure-hosting-recommendations.md) owns the platform configuration behind them |
| The thirteen-check deployed verification gate, its two-instance affinity-off topology and its attempt budget's literal evidence bounds ([06 §12.1](06-azure-hosting-recommendations.md)) | [R12](#r12--no-observability-exists-during-the-cutover-itself), with [R13](#r13--one-database-one-blast-radius) | The gate is **the mitigation** — five of its checks exist precisely to detect the cross-instance and restart continuity failures a shared key ring prevents. Its failure mode is a **blocked release**, which is the gate succeeding at its purpose |
| The secretless cache-table credential path, **whose SQL-authentication fallback [06 §6.4](06-azure-hosting-recommendations.md) has since removed**, and the artifact store's encryption, key custody, dedicated destruction scope and lifecycle-enforced deletion | [R17](#r17--the-interim-hosting-paths-stored-credential-and-a-time-box-with-no-box), with [R3](#r3--the-absent-regression-baseline) | Both **remove** a stored credential or a retained plaintext rather than introducing one. R17 is the entry that tracks a stored credential's existence, and it is unaffected because these paths carry none |
| The **one** test project [04 §12.2](04-dotnet8-migration-strategy.md) maps, the **six** independent tooling pins of which **four** are declared directly in it, the abstract-plus-sealed contract topology, the runtime-neutral store observer, and the **three** projects and **three** lockfiles [04 §6.4](04-dotnet8-migration-strategy.md) ([04 §7.2, §6.4](04-dotnet8-migration-strategy.md) and [05 §12.2, §12.6](05-aspnet-core-migration-approach.md)) | [R11](#r11--the-effective-package-source-set-is-not-knowable-from-the-repository) | One further pin restored from an undeclared source is the same finding at a larger count; R11 owns it, and [04 §7](04-dotnet8-migration-strategy.md) owns the pin |
| The security-header middleware, the error action, and the ledger's further entries — **27** in total (input 17), among them the anti-forgery adoption, the verb-mismatched `405` responses and the cart-migration failure notice | [R6](#r6--security-hardening-versus-compatibility), with [section 7.2](#72-the-approved-delta-sign-offs) | Each is a **hardening or a behaviour change with a named approval owner**, and R6's subject is exactly that an item of that kind must be **explicitly labelled** rather than adopted by default. The ledger's growth is counted in 7.2's sign-off basis, which is where an unapproved delta becomes visible |

**One item was genuinely considered for an entry of its own and rejected on the evidence.** The
fail-closed compatibility gate creates a failure mode the previous scope had no equivalent of: the target
half of the suite **cannot run at all** if the legacy binary, the fixture fingerprint, the configuration
or the pinned locale and collation values move, which makes every one of those a re-capture trigger on the
one platform that can produce a baseline. That is a real coupling, and it is a **compound of R2 and R3
rather than a third thing**: R2 already carries the availability of the platform that runs the legacy
application, and R3 already carries the baseline's determinism and states that its capture half can only
happen where that application runs. A separate entry would restate both and own neither, which
[section 10.4](#104-cross-reference-index) is the wrong side of. Both entries were **updated** instead,
and R3's mitigation now names the gate explicitly.

**[R18](#r18--the-browser-executed-half-of-the-scripted-cart-flow-one-pinned-engine-and-an-open-decision-on-the-other-two)
narrowed twice and closed neither time — and its return to
[section 9.4](#94-the-six-risks-that-are-approval-decisions-not-mitigations) adds no row here.** The
suite already fetched, parsed and submitted the rendered forms, which reduced the entry's subject to the
**one** script-issued flow; [05 §12.11](05-aspnet-core-migration-approach.md) then **pinned a Chromium
harness for that flow**, which put the effort inside [W7](#w7--the-aspnet-core-port)'s and
[W11](#w11--ci-provider-selection-then-pipeline-authoring)'s bands and reduced the exposure to **Gecko and
WebKit**. Neither narrowing produced a functional assertion on those two engines, so the entry remains
**open** and remains an approval decision — restored as section 9.4's sixth row, with the three admissible
outcomes [03 §5](03-modernization-roadmap.md)'s W1 names. **That is a change of scope inside an existing
entry rather than a new uncertainty**, which is why the register still carries eighteen entries: a control
arriving does not add a row, and neither does a decision being restored to the owner who must take it.

---

## 10. Roll-up

### 10.1 The estimate in nine statements

1. **The total is 171.5 / 311.5 / 545.5 ideal engineer-days** across [03](03-modernization-roadmap.md)'s
   sixteen workstreams (154.5 / 278.5 / 489) plus the three non-code tasks that sit outside them, the
   manual visual review (6.5 / 12.5 / 22.5), the manual accessibility review (2.5 / 4.5 / 8) and the
   approved-delta sign-offs (8 / 16 / 26). The sign-offs are **not** inside W1's band —
   [§7.2](#72-the-approved-delta-sign-offs) owns that band and
   [§6.1.1](#611-the-walk-from-the-previously-published-total) records the round in which a previous
   revision counted them zero times by calling them a component of W1. Each of the nineteen rows that
   enters the total is counted exactly once. An IED is work content, not elapsed time. **Two of those rows
   have now been re-derived a sixth time** — W4 and W7, as [05 §12.4](05-aspnet-core-migration-approach.md)
   extended its coverage matrix again — **three had been re-derived a fifth time** before that, W4, W7 and
   W12, and **five a fourth time** before that: W4, W7, W11, W12
   and the manual visual review. **W1's band moved
   for the first time**, its act census having gained R18's recorded outcome, and **three rows
   entered the total that the summary table already displayed**, W16, the accessibility review and the
   sign-offs, the last of which also rose because §7.2 prices 05 §11.7's fifteen approval-owned
   additions beside every row of the approved-delta register **and reads that register at its present
   size** — a size that has since fallen by one owner-carrying row, taking that band down half a day in
   every column. The re-derivations follow successive rounds of closed findings. **The latest moved the
   published suite to 104 rows, 326 cases and 478 fixture executions, and retired a second identifier in
   the approved-delta register**, which is why exactly three rows moved with it: the two that price cases,
   and the one that prices those decisions. The round before moved
   the published suite to **102 rows, 307 cases and 454 fixture executions**; made the contract topology
   **declarable** rather than described, with public types and a const-named collection definition per
   surface group per assembly; added an **`IStoreSetup`** write API off the injected context and five
   further observer projections; committed **twelve fixture inputs** including nine schema-divergence
   override files; gave the load, reconcile and dry-run commands a mandatory **`--store`**; added the
   four **`dbo.__DataMigration*` run tables** written inside each group's own transaction; added the deployed gate's
   **telemetry join protocol** and its executor-and-stage mapping; and made **row 75 W12's acceptance**
   with an operator-tool audit provider behind it. The whole movement from the previously published
   133.5 / 239.5 / 424 is **+38 / +72 / +121.5** — `+13.5 / +26.5 / +46` for the three rows that entered
   the total and `+24.5 / +45.5 / +75.5` for the six re-derived bands, of which `+10.5 / +16.5 / +26.5` is
   the coverage matrix's two extensions landing on W4, W7 and W12;
   [section 5.1](#51-summary-table) prints the movement,
   [section 6.1.1](#611-the-walk-from-the-previously-published-total) walks it row by row, and
   [section 5.2](#52-basis-of-estimate-per-workstream) shows each derivation.
2. **The port of existing code is 16.2 percent of that.** The other 83.8 percent is net-new
   capability, data work gated on an unextracted schema, and governance and verification — none of it
   predicted by any
   quantity in the existing codebase. **The movement again landed entirely outside porting**: the
   porting figure rose from 47 to 50.5 expected IED and **has not moved since**, while net-new rose from 82.5
   to 166, then to 178.5 and now 182.5, data work from 34 to 24 as W13's band was re-derived downward into
   it, and
   governance from 37 to 49.5, then to 55, and now **down** to 54.5. **None of the latest 3.5 is porting,
   and none of the 18 before
   it was either**: this round 1 on W4's case sub-row and 3 on W7's are net-new and `−0.5` on the sign-off
   row is governance, which is the one category that fell; in the round before, 0.5 on
   W4's case sub-row, 10 on W7's and 2 on W12's console rows were net-new and 5.5 on the sign-off row was
   governance.
   [§6.2](#62-the-finding-that-matters-most-in-this-document) partitions it by activity and checks both
   identities column by column.
3. **The two largest rows are W7 (122.5 expected) and W4 (67.5 expected)**, and the second of them replaces
   nothing: there are zero tests today. **62.5 of W7's 122.5 is also the suite** — its target-facing
   machinery, its browser-driven flow and its cases —
   so the verification apparatus is 130 of the 311.5 expected IED, and its own addition is
   67.5 + 22.5 + 3.5 + 36.5 = **130**, about two fifths of the model (`130 ÷ 311.5 = 41.7%`).
4. **Five low-confidence rows carry 93.5 expected IED**, and three of the five are waiting on one cheap
   workstream: W3, at 4 expected IED, is the highest-leverage item in the model. Their share **fell
   three times and then held, from about 34 percent to 31.7, then 30.6, then 30.0, and 30.0 again**
   (`93.5 ÷ 311.5 = 30.0%`), because
   the Low rows grew by 11.5, then 0.5, then not at all, then by 1, while the total grew by 50.5, then
   12.5, then 5.5, then 3.5. A sixth row, W2,
   is
   medium-confidence for a different reason — its tasks are enumerated but its outcome is not settled,
   because the migration source's build gate is still carried as blocked and the one prescribed-toolchain
   rebuild on record is supplementary observation rather than a discharge of it
   ([10 §1.2, §3.2](10-build-and-deployment-requirements.md)).
5. **The critical path is 275 of the 311.5 expected IED**, gate-inclusive — a sum of **work content along
   one dependency chain**, not an elapsed duration, and [§8.3](#83-the-critical-path-and-what-to-do-first-if-the-goal-is-to-narrow-the-estimate)
   states what that does and does not license a reader to infer. Only **11.7 percent** of the model's work
   content lies off that chain. Three properties of
   the graph put it there rather than estimator caution: the personal-data policy gates the schema
   extraction, which gates the heaviest gate in the plan; both manual reviews close the port's exit gate;
   and W11 is a single node with four entries, so the whole of it is on the chain. **The off-path share
   fell, from about 16 percent to 12.6, then 12.1, then 11.9 and now 11.7 percent** — the last three falls arithmetic rather than
   structural, since the off-path row has not moved at all while on-path nodes and one on-path gate condition re-banded. Three
   nodes changed sides in the process: W11's
   provider half and W16's policy stage moved onto the path, and **W9 moved off it** when
   [03](03-modernization-roadmap.md) withdrew the `W9 → W8` edge — W9's 8 expected IED is now the
   largest single off-path item.
6. **Every figure states its counting method.** The authentication rewrite is estimated against
   **382 non-blank lines** — the sizing metric — and the physical line count for that file appears
   nowhere in this document. The suite is estimated against the **case**, the unit
   [05 §12.4](05-aspnet-core-migration-approach.md) publishes, with fixture executions counted separately
   because a case authored once may execute twice.
7. **The register carries eighteen entries, `R1` through `R18`**, each with all seven required fields —
   and the **count is unchanged by this re-derivation**, because the scope that was added is **control
   rather than uncertainty**;
   [section 9.5](#95-why-the-enlarged-scope-added-no-nineteenth-entry) assigns each new item to the entry
   that carries it and states the one candidate for an entry of its own that was considered and rejected.
   Two further risks, `I1` and `I2`, are **held elsewhere** and signposted in
   [§9.2](#92-register-index) rather than counted here. **Six**
   of them are
   **approval decisions rather than engineering risks**, and W1 cannot correctly exit without a recorded
   decision on each. The six are **R1, R7, R9, R13, R15 and R18**, and the last of them is there because
   [R18](#r18--the-browser-executed-half-of-the-scripted-cart-flow-one-pinned-engine-and-an-open-decision-on-the-other-two)
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
four commands [§2](#2-the-no-modification-constraint) states, run together:

```bash
git status --porcelain                    # 0 lines
git status --porcelain --ignored          # 0 lines, and no "!!" row
git clean -ndX                            # empty
git diff --name-status ea2552d..HEAD      # 13 lines, all "A", all under docs/modernization/
```

Thirteen additions, **no `M` line and no `D` line** — so no pre-existing file was modified or deleted and
nothing was added outside `docs/modernization/`, which is the claim, rather than the false one that nothing
was added at all. **The diff's lower endpoint is pinned and its upper endpoint is the reader's own tip.** An
earlier form of this block pinned the upper endpoint to a specific commit id as well; that id is the id of
the commit that adds this document, so it could not exist when the text was written and resolves to nothing
in this repository. `ea2552d..HEAD` is what a reader can actually run, at the cost of a result that expires
the moment a fourteenth file lands under `docs/modernization/` — so it is a current-checkout observation
with a pinned lower endpoint, and the both-endpoints-pinned form belongs to the release record kept outside
these documents ([04 §13.4](04-dotnet8-migration-strategy.md)). The three status commands are
authoring-time checks: at the committed checkpoint each is empty, which is why
[§2](#2-the-no-modification-constraint) requires all four together rather than any one of them.

**The heading is a shorthand, and the sentence under it is the claim.** "Changes nothing" means *modifies
nothing that existed*; the thirteen files this work adds are its output and are not a violation of the
constraint it was held to.

### 10.3 Acceptance criteria for this deliverable

Self-check against AAP §0.11.2 row 07 and the deliverable's own authoring contract.

| Criterion | Where satisfied |
| --- | --- |
| An effort model **per workstream**, in **stated units** | [§4.2](#42-the-unit-defined) states the unit; [§5.1](#51-summary-table) is one row per workstream, using 03's names |
| **Low / expected / high bands** on every row; no single-point estimate | [§5.1](#51-summary-table) — all twenty rows, the nineteen that enter the total and the conditional one that does not, plus the W4, W7, W10 and W16 sub-decompositions |
| **Assumptions explicit** | [§4.3](#43-assumptions-stated-rather-than-implied) — **A1–A11**, each with the consequence of its being false |
| **Confidence with its reason** | [§4.4](#44-confidence-and-its-reason) — overall plus per-workstream, with the reason stated rather than asserted |
| Every effort figure **traces to a count** in this document, 01, 02 or 08 | [§4.1](#41-the-estimation-basis-every-input-with-its-method) — **32** numbered inputs, each with its method and its source; **eighteen of the nineteen rows of [§5.1](#51-summary-table) that enter the total name at least one that sizes their own work, and the nineteenth — W13, cutover — names none, because no count of anything in this repository is a proxy for executing a runbook**; its size is [06 §6.10](06-azure-hosting-recommendations.md)'s and [06 §11](06-azure-hosting-recommendations.md)'s ordered step counts, its cell says so, and §4.1's fifth usage note records it as the census's one stated edge rather than leaving a reader to find it. Every band in §5.2 cites the inputs it uses. **Input 31 is the most recent of them and it closes the last row that was sized from the wrong population**: W5's repository-wide path-literal census, which replaces the migration-source inputs 7, 8 and 11 that row previously named. Inputs 24–30 exist for the same reason: **six of the seven close gaps on seven rows** — five of the six that named no count at all (W2 by input 24, W10 by 26, W14 by 29, W15 by 27, W16 by 28), **W12** which named only input 23 (sizing its *added tests* rather than the tool, now closed by **input 30**), and **W11** which named a constraint rather than a size (closed by input 26, the one input serving two rows, which is why six cover seven). The **seventh, input 25, closes no row**: its zero observability artifacts size the net-new half of W10's basis, and W10 itself is closed by input 26. §4.1's fifth usage note carries the row-by-row census, both stated exceptions, the four-kind partition `2 + 2 + 2 + 1`, and the one of the seven that moved a band |
| Every figure **names its counting method** | [§3](#3-the-two-counting-methods) states the rule; every figure carries "sizing", "duplication", "file count", "site count", "pin count", "row count", "entry count" or "test count" |
| The **test workload is partitioned** between the parity suite and the tests that have no baseline | [§4.1](#41-the-estimation-basis-every-input-with-its-method)'s inclusion rule — input 14 is [05 §12.4](05-aspnet-core-migration-approach.md)'s **104** parity rows carrying **326** cases, **re-read from that table by the command input 14 carries rather than held as a constant**, input 23 the **seventeen** tests [04](04-dotnet8-migration-strategy.md) and [06](06-azure-hosting-recommendations.md) require, and input 32 the **eight** published-console rows of [05 §12.4.1](05-aspnet-core-migration-approach.md) that are new work, for `104 + 17 + 8 = 129` executable scenarios; the second and third sets are estimated in W12, W7 and W10, never folded into W4, and [03 §4.3](03-modernization-roadmap.md) gates the second at the same three workstreams |
| **382 non-blank used for the authentication rewrite**; physical counts not used for sizing | [§3.2](#32-the-one-substitution-that-would-corrupt-this-document) and [W7](#w7--the-aspnet-core-port). The physical figure for that file appears nowhere |
| Asset counts **state their inclusion rule** | Input 7 and [A.3](#a3-helper-view-and-site-counts) — the migration source's **four asset groups**, distinguished from the all-edition and browser-served counts |
| Estimated against **03's workstreams**; no alternative decomposition | [§5](#5-effort-by-workstream) opening paragraph; W1–W16 verbatim, including W16 |
| **Net-new work sized honestly**, and **partitioned by activity rather than by workstream** | [§6.2](#62-the-finding-that-matters-most-in-this-document) quantifies it at **83.8%** of expected effort — `182.5 + 24 + 54.5 = 261` of 311.5, against the **50.5** that is porting. The three rows whose split crosses a category boundary — W7, W11 and W16 — are split along the sub-bands they publish, and both partition identities are checked there column by column |
| **Every arithmetic identity in the model holds**, column by column, including the two derived claims | Checked by extracting the figures and summing them, not by reading them. **Row sum:** [§5.1](#51-summary-table)'s nineteen counted rows sum to `171.5 / 311.5 / 545.5`, equal to its own Total row and to [§6.1](#61-the-totals)'s, and its sixteen workstream rows sum to `154.5 / 278.5 / 489`. **Walk:** [§6.1.1](#611-the-walk-from-the-previously-published-total)'s nine deltas sum to `+38 / +72 / +121.5`, split `+13.5 / +26.5 / +46` for the three rows that entered the total and `+24.5 / +45.5 / +75.5` for the six re-derived bands; each row's From-to-To difference equals its stated delta, every To equals its owner's published band, and the corrected base `147 / 266 / 470` plus the six re-derivations reaches the same destination from the other end. **Activity partition:** [§6.2](#62-the-finding-that-matters-most-in-this-document)'s four categories sum to the same three totals, and each of the three splits sums to its row's published band; its four one-decimal shares sum to exactly `100.0%`, and the round in which they summed to `100.1%` is recorded there as the rounding artifact it was rather than having been hidden by nudging a quotient. **Concurrency:** [§8.2](#82-concurrency-permitted-by-the-graph)'s thirteen set figures sum to `311.5`; each split row's parts sum to the band [§5.1](#51-summary-table) carries for it; and **no set contains two members joined by an edge of 03's inventory**, which is the property that makes a set a concurrency claim rather than a bucket. **Path:** [§8.3](#83-the-critical-path-and-what-to-do-first-if-the-goal-is-to-narrow-the-estimate)'s twelve-node base plus its three gate rows equals the gate-inclusive figure in all three columns — `138.5 / 247.5 / 431` plus `14.5 / 27.5 / 45.5` — on-path plus off-path equals the total in all three, and the off-path row equals the sum of its eight itemized parts. **The two derived claims:** the high-to-low ratio `545.5 / 171.5 = 3.1808`, which [§6.1](#61-the-totals) prints as `3.18` and reads at one decimal as about 3.2, and the off-path share `36.5 / 311.5 = 11.72%` |
| Sequence **dependency-ordered**, parallelism noted, **no calendar** | [§8](#8-sequencing) — thirteen concurrency sets and the critical path, explicitly properties of [03 §4.2.1](03-modernization-roadmap.md)'s twenty-node graph, with the staged rows W4, W10 and W16 split across sets and W11 carried whole because 03 draws it as one node |
| **Every gate on the critical path is one that can actually be met** | [§8.3](#83-the-critical-path-and-what-to-do-first-if-the-goal-is-to-narrow-the-estimate) — the chain is a longest path by weight over 03's twenty-node graph, maximized independently at each band, and the three gate conditions on it are counted as conditions of the gates they close rather than as work after them |
| **R1 first**, framed as an **approval decision** | [R1](#r1--the-target-framework-support-window), and [§9.4](#94-the-six-risks-that-are-approval-decisions-not-mitigations) |
| All nine named risks present | R1–R9 in [§9.3](#93-the-entries), plus R10–R18 where the evidence warranted |
| **All seven fields on every entry** | [§9.3](#93-the-entries) — a field table per entry, eighteen of eighteen complete |
| The **visual review** and **delta sign-offs** carried as tasks with effort, **at the gates they belong to** | [§7.1](#71-the-manual-visual-review) — review and sign-off inside W7's exit gate — and [§7.2](#72-the-approved-delta-sign-offs), inside W1's |
| **Personal-data governance and the security-event catalog sized and risked** | W16 in [§5.2](#52-basis-of-estimate-per-workstream); the security-event sub-row in [W7](#w7--the-aspnet-core-port); [R15](#r15--personal-data-governance-is-unowned) and [R16](#r16--no-security-relevant-action-is-recorded-anywhere) |
| **Cross-references only**, no restatement | [§1.4](#14-what-this-document-does-not-own--the-routing-table) is the routing table; §10.4 is the index |

### 10.4 Cross-reference index

| Deliverable | What this document takes from it | What it takes back |
| --- | --- | --- |
| [01](01-architecture-overview.md) | Verified counts, code volume, view topology, asset groups, the two cart unit-of-work models (inputs 1, 5–7, 9, 10) | — |
| [02](02-dependency-inventory.md) | The as-is pins and the restore-source finding | [R11](#r11--the-effective-package-source-set-is-not-knowable-from-the-repository) |
| [03](03-modernization-roadmap.md) | **The workstream decomposition and every entry and exit gate** — the structure of §5 and §8 | The effort model and the risk register it routes here |
| [04](04-dotnet8-migration-strategy.md) | Target framework, SDK band, the 28 package outcomes, the future application map (input 13), and the **5 operator-host tests** of §12.4 that have no MVC 5 baseline (input 23) | [R1](#r1--the-target-framework-support-window)'s support-window decision, which 04 §2.2 routes here; the estimate for those five tests, carried in W12 |
| [05](05-aspnet-core-migration-approach.md) | Port design, the test suite's architecture and its required coverage rows — **104** rows and **326** cases as recorded in input 14, counts [05 §12.4](05-aspnet-core-migration-approach.md) owns and this document re-reads rather than fixes — the retained external-login contract, the two data migrations and their **two** per-context change-tracking migrations, the cutover decision, every row of the approved-delta register (inputs 14, 17) | [R6](#r6--security-hardening-versus-compatibility), [R7](#r7--the-narrowed-browser-matrix), [R9](#r9--cutover-re-authentication-and-anonymous-cart-loss); [§7.1](#71-the-manual-visual-review)'s review; every duration |
| [06](06-azure-hosting-recommendations.md) | Hosting target, provisioning order, DDL principal, key ring, observability approach, the browser matrix, the **three-regime rollback position with its accepted-write probe** (§11.5) on which [R4](#r4--domain-data-migration-rollback)'s contingency now rests, the **explicit-rotation credential policy** of §9.5 that W12's basis is estimated against, and the **12 CSP tests** of §10.2 — **11** HTTP tests in W7 and the **twelfth**, the deployed-browser gate `G-CSP-BROWSER`, in W10 (input 23) | [R7](#r7--the-narrowed-browser-matrix), [R8](#r8--case-sensitive-path-resolution-on-the-target-platform), [R12](#r12--no-observability-exists-during-the-cutover-itself), [R13](#r13--one-database-one-blast-radius) |
| [08](08-technical-debt-register.md) | **The two counting methods**, the three-part decomposition, the estimation-safe quantities and the forbidden ones (inputs 1–5, 18, 19) | [R3](#r3--the-absent-regression-baseline), [R4](#r4--domain-data-migration-rollback), [R10](#r10--scoping-by-analogy-across-editions) — the three §12.3 asks this document to carry |
| [09](09-security-assessment.md) | The security posture behind [R6](#r6--security-hardening-versus-compatibility); the **security-event catalog** and its 16 classes — **13** application-produced, **2** tooling-produced and **1** with no producer, per §6.8.1.1's producer map (input 21); the **nine personal-data fields** and the absent retention, deletion and audit controls (input 22); the out-of-support consequence in [R7](#r7--the-narrowed-browser-matrix)'s contingency | [R15](#r15--personal-data-governance-is-unowned), [R16](#r16--no-security-relevant-action-is-recorded-anywhere), and the effort for W16 and W7's security-event sub-row |
| [10](10-build-and-deployment-requirements.md) | **Per-edition build outcomes** — the migration source's status, **blocked pending a Windows verification run**, cited and not restated | [R2](#r2--the-migration-sources-build-reproducibility), estimated against that open status |
| [11](11-cloud-readiness-assessment.md) | Statefulness, transport and path casing as-is, behind [R8](#r8--case-sensitive-path-resolution-on-the-target-platform) and W15 | — |
| [12](12-migration-blockers.md) | **The 23 blockers in two groups** (input 12); the evidence-rather-than-proof qualification | [R4](#r4--domain-data-migration-rollback), [R5](#r5--identity-migration-rollback), [R14](#r14--a-reference-editions-retired-data-provider) |
| [README](README.md) | The requirement-to-deliverable map | This document answering "Estimated effort, risks, and sequencing" |

### 10.5 Where the security register attaches — the four [09](09-security-assessment.md) rows that name this document

[09](09-security-assessment.md)'s register routes its rows by document number, and its own consumer roll-up
names **four of its thirty-eight rows** to this one — **F-09-18**, **F-09-19**, **F-09-32** and
**F-09-33** — with the obligation stated as *its effort model and risk register, at the entries the rows'
own cells name*. A consumer that is named and does not answer leaves the row with no discharging item, so
each is answered below at the entry or band that carries it. **No finding text is restated**:
[09](09-security-assessment.md) owns the finding, its severity and its editions, and the naming column is
only enough to identify which row is meant. Two of the four also name
[03](03-modernization-roadmap.md), and the two answers are different obligations — 03 owns *which
workstream and which gate*, this document owns *what it costs and what it risks*.

| [09](09-security-assessment.md) row | The finding, named only | The entry or band that carries it here | Where it is sized | The evidence that closes it in this document |
| --- | --- | --- | --- | --- |
| **F-09-18** | MVC 3's administration surface is guarded by an `Administrator` role no code creates, so it is unreachable as shipped | [R10](#r10--scoping-by-analogy-across-editions) — scoping by analogy across editions | **Deliberately nowhere as remediation work**, and that is the answer rather than an omission: assumption **A7** ([§4.3](#43-assumptions-stated-rather-than-implied)) scopes the port to the migration source alone, and input 5 ([§4.1](#41-the-estimation-basis-every-input-with-its-method)) carries MVC 3's **1,326 non-blank lines**, the sizing metric, only so the repository total cannot be read as the work envelope | R10's **mitigation** — any request to port another edition is re-estimated from its own measurement, never scaled — and its **trigger**, which fires on any estimate for a non-migration-source edition that cites a figure from this document. Its affected workstreams are **W1**, where the scope is approved, and W7 if the scope changes, so a decision to make this surface reachable would be a new scope decision at W1 rather than a variance inside any band above |
| **F-09-19** | MVC 3's account controller is allow-by-default where both newer editions are deny-by-default | [R10](#r10--scoping-by-analogy-across-editions), on its second limb | Not sized, on the same basis as the row above | This row is **evidence for R10 rather than work for this document**: R10 argues that the oldest edition is smaller in volume and larger in *structural* difference, and an authorization posture that differs in kind — not in degree — from the migration source's is exactly that difference in the security dimension. A multiplier over line counts would therefore under-state it while appearing conservative, which is R10's stated failure mode |
| **F-09-32** | No logging construct of any kind exists, so no authentication, authorization, administrative or order event is recorded | [R16](#r16--no-security-relevant-action-is-recorded-anywhere) | Input 21 ([§4.1](#41-the-estimation-basis-every-input-with-its-method)) — **16** event classes, a row count split by [09 §6.8.1.1](09-security-assessment.md)'s producer map as **13 + 3**: the **thirteen** application-produced inside [W7](#w7--the-aspnet-core-port)'s band and the **three** tooling-produced inside [W12](#w12--administrator-provisioning-tool)'s. `AUTHZ-3001` is one of those three and has a **second** producer, so the Identity data migration's per-assignment record is charged again inside [W8](#w8--identity-migration-tooling-rehearsed-against-a-copy)'s band without adding a class; the collection path is priced inside [W10](#w10--hosting-provisioning-and-platform-configuration)'s | R16's own fields: the mitigation names the catalog, the emission and the collection separately; the **trigger** fires on W7's exit approached with any of the thirteen unemitted, any seam absent, any outcome value unexercised or any redaction test absent; and the affected-workstream field prices **W10 as the collection path proved by one canary class only**, not by thirteen. The cost driver is stated as the seam count rather than the field count, so adding a class to the catalog moves W7's sub-row and not this entry's severity |
| **F-09-33** | Nine personal-data fields are stored in the clear with no retention policy, no deletion path and no access log | [R15](#r15--personal-data-governance-is-unowned), and [§9.4](#94-the-six-risks-that-are-approval-decisions-not-mitigations)'s row for it — one of the **six** risks whose resolution is a decision rather than a mitigation | [W16](#w16--personal-data-governance) — **3 / 6 / 12 IED, low confidence**, from inputs 22 and 21, sized across the **three** personal-data stores [03 §5 W16](03-modernization-roadmap.md) scopes it to rather than two | W16's band is one of the five low-confidence rows that carry **93.5 of the 311.5** expected IED ([§4.4](#44-confidence-and-its-reason)), and its width is set by how many periods and rules the policy stage has to settle rather than by any code volume — which is why the entry is an **approval decision**: [§9.4](#94-the-six-risks-that-are-approval-decisions-not-mitigations) requires W1 to record the retention periods, the non-production copy rules and the legal-hold process, and [W1's act census](#w1--approval-of-this-assessment) charges that decision as one of its eleven acts |

**Four rows and not more, checked against the register rather than remembered.**
[09](09-security-assessment.md)'s consumer roll-up gives the count as **4** for this document and lists
exactly these identifiers; the rows it routes to [03](03-modernization-roadmap.md) for workstream and gate
placement — F-09-32 and F-09-33, the two that appear in both lists — are answered in that document's own
closure table.

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

**The multi-owner reading of the two approval registers — inputs 17 and 32, site counts.**
[§7.2](#72-the-approved-delta-sign-offs) prices **63** decisions and, separately, the **25** of
them whose recorded approval owner is more than one party, because a consent that two owners must both give
costs more than one that a single owner gives. Both figures are readings of the owner column of the two
registers [05](05-aspnet-core-migration-approach.md) owns, taken by one test — the cell contains `" and "`,
`" plus "` or a comma — and neither is asserted:

```bash
M=docs/modernization/05-aspnet-core-migration-approach.md

# §11.5 — the approved deltas: rows, then rows whose owner cell names more than one party
awk '/^### 11\.5 /,/^### 11\.6 /' $M | grep -cE '^\| [0-9]+ \|'                       # -> 47
awk '/^### 11\.5 /,/^### 11\.6 /' $M | grep -E '^\| [0-9]+ \|' \
  | awk -F'|' '{print $(NF-1)}' | grep -cE ' and | plus |,'                           # -> 24

# §11.7 — the approval-owned additions, same two readings
awk '/^### 11\.7 /,/^### 11\.8 |^## 12\. /' $M | grep -cE '^\| A[0-9]+ \|'            # -> 16
awk '/^### 11\.7 /,/^### 11\.8 |^## 12\. /' $M | grep -E '^\| A[0-9]+ \|' \
  | awk -F'|' '{print $(NF-1)}' | grep -cE ' and | plus |,'                           # -> 1
# 47 + 16 = 63 decisions ; 24 + 1 = 25 of them multi-owner
```

**The single multi-owner addition is `A14`**, whose owner cell names engineering and operations; the other
fifteen name one party each. The test is deliberately syntactic rather than semantic — it reads the owner
cell as the register writes it and makes no judgement about whether two named parties must both consent —
which is why [§7.2](#72-the-approved-delta-sign-offs) charges the second increment as a
coordination cost on top of every decision's base cost rather than as a different kind of decision.

### A.4 The visual review capture set (input 16)

**Input 16 is cited rather than derived, and this appendix publishes no capture total of its own.**
[05 §12.5.1](05-aspnet-core-migration-approach.md) owns the manifest — **67 baseline images and the same
67 comparisons**, `34 + 33 + 0 + 0 = 67` across its four passes — and
[§7.1](#71-the-manual-visual-review) prices exactly that total. What this appendix reproduces is the
**basis** the manifest's surface count rests on, so a reader can check the citation against the repository
instead of taking it. That is worth commands because there are two substitutions available here that each
land on a plausible number and the wrong set: counting **filenames**, and counting **render sites**.

**The manifest's surface basis, in its owner's terms.**
[05 §12.5](05-aspnet-core-migration-approach.md) resolves the 29 views into **20 page surfaces**, 3
component views, 4 rendered partials, the layout and `_ViewStart`, and excludes **three** of the 20 from
capture on one stated criterion — **no user-reachable navigation path in the running legacy application**,
which 05 §12.5 keeps narrower than "no request whatsoever can render it": `Home/About.cshtml` and
`Home/Contact.cshtml`, which no action returns
[src/MVC5/MvcMusicStore/Controllers/HomeController.cs:15], and
`Account/ExternalLoginConfirmation.cshtml`, whose **GET does not exist** because the action is `POST`-only
[src/MVC5/MvcMusicStore/Controllers/AccountController.cs:262-265], so no navigation reaches it — and a
reviewer-synthesized POST carrying a harvested token is not navigation. `20 − 3 = 17`, which is pass **P1**'s surface
count. All three excluded surfaces are **ported** — 05 §8.4 deletes no view and 05's external-login
decision retains the surface **dormant**, so nothing 404s by design — and
[§7.1](#71-the-manual-visual-review) prices each as a markup comparison whose record states the absence of
a baseline rather than reporting a pass. Pass **P2**'s three layout-bearing surfaces, the eleven widths
they are captured at, and passes **P3** and **P4**'s measured zero are 05's as well, and none of them is
re-counted here.

**Why a filename is not a surface, bounded by a command.** The 29 views resolve into **22**
non-underscore files and **7** underscore-prefixed ones — five rendered partials, the shared layout and
the view-start file.

```bash
git ls-files 'src/MVC5/MvcMusicStore/Views/*.cshtml' | grep -vE '/_[^/]*\.cshtml$' | wc -l   # -> 22
git ls-files 'src/MVC5/MvcMusicStore/Views/*.cshtml' | grep -cE '/_[^/]*\.cshtml$'           # ->  7
```

**The 22 is a file count, and the owner's 17 is reachable from it by two named subtractions** — which is
what makes the citation checkable rather than merely quoted. Subtract the **two child-action fragments
that carry no underscore and are never a page in their own right**, `Views/Store/GenreMenu.cshtml` and
`Views/ShoppingCart/CartSummary.cshtml`, both invoked from the shared layout
[src/MVC5/MvcMusicStore/Views/Shared/_Layout.cshtml:25],
[src/MVC5/MvcMusicStore/Views/Shared/_Layout.cshtml:26], and the count is 05 §12.5's **20** page
surfaces. Subtract the **three** excluded above — all three of which appear in that same command's output —
and it is the manifest's **17**: `22 − 2 − 3 = 17`. A filename heuristic gets the fragments wrong in one
direction and the orphaned views wrong in the other, and the third component view is
`Account/_RemoveAccountPartial.cshtml`, which *does* carry an underscore, so no rule about underscores
sorts the three of them either.

**Why a render site is not a surface either, bounded by the same kind of command.** There are **27**
`return View` sites across the six controllers of the migration source, and they partition exactly:
**17** render a page from a `GET`, **1** renders the error page by name (`Checkout`'s ownership branch
[src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:81]), **8** are `POST` redisplays of a page
rendered elsewhere in the same set, and **1** is a `POST` rendering the external-login failure page
[src/MVC5/MvcMusicStore/Controllers/AccountController.cs:278]. `17 + 1 + 8 + 1 = 27`.

```bash
git grep -c 'return View' -- 'src/MVC5/MvcMusicStore/Controllers/*.cs'
# -> AccountController.cs:10   CheckoutController.cs:5   HomeController.cs:1
#    ShoppingCartController.cs:1   StoreController.cs:3   StoreManagerController.cs:7   = 27
git grep -c 'PartialView' -- 'src/MVC5/MvcMusicStore/Controllers/*.cs'
# -> AccountController.cs:1   ShoppingCartController.cs:1   StoreController.cs:1        =  3
```

**That 17 and the manifest's 17 are not the same seventeen, and the coincidence is the reason this note
exists at all.** The two sets differ by one member in each direction. The `GET` site at
[src/MVC5/MvcMusicStore/Controllers/AccountController.cs:229] is one of the 17 sites and its page is the
external-login confirmation surface the manifest **excludes**; `Shared/Error` is **inside** the manifest's
17 and is rendered **by name** rather than from a `GET` site
[src/MVC5/MvcMusicStore/App_Start/FilterConfig.cs:10],
[src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:81]. A reader who treats the site census as a
capture basis therefore arrives at the right cardinality by accident and at the wrong set by construction.

**The second command above is the reason a fragment cannot be counted as a page.** All three child actions
return a **partial** rather than a view — [src/MVC5/MvcMusicStore/Controllers/StoreController.cs:54],
[src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:98],
[src/MVC5/MvcMusicStore/Controllers/AccountController.cs:321] — so two of them render through
non-underscore filenames while only ever appearing inside a page the manifest already captures.

> **Nothing in this appendix is an image count.** 22, 7, 27, 17, 1, 8 and 1 count files and render sites;
> **20** and **17** count surfaces and are 05's; and the only figures that count **images** anywhere in
> this document are the manifest's `34 + 33 + 0 + 0 = 67` and the same 67 comparisons, stated once in
> [05 §12.5.1](05-aspnet-core-migration-approach.md) and priced once in
> [§7.1](#71-the-manual-visual-review). Multiplying a surface count by a viewport count, or by the four
> browser families of [06 §10.4](06-azure-hosting-recommendations.md), would produce a second capture
> total — which is precisely what one owner and one manifest exist to prevent.

### A.5 The corroborating case mismatch (R8)

The primary mismatch is cited inline at
[src/MVC5/MvcMusicStore/App_Start/BundleConfig.cs:28] against
[src/MVC5/MvcMusicStore/Content/Site.css:1,450 bytes]
— the registration has a line, the tracked filename does not, so the stylesheet is cited at its size
and the command below prints its capitalisation, which is the half of the pair no line locator can
carry. The corroboration — that this repository is systemically careless about case rather than
carrying one typo — rests on **three** independent instances: the pair above, **the identical pair in MVC 4**,
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

**The one revision range this document publishes, with both endpoints defined here rather than left to
inference.** Its left side is `ea2552d6eda7c20e9477a512e5c615665618cf35` — the immutable pre-assessment
revision, the last commit before this assessment began, at which every source path cited anywhere in this
document is byte-identical to the delivered tree — and its right side is `HEAD`, meaning **the delivery
commit the reviewer has checked out**, which no document can name by hash because a document cannot cite
the commit that creates it. The two derived counts below are the same range piped through `grep`, not
further ranges. Every other revision reference in this document is to `ea2552d` alone, short-form after
its first full use, and is a point rather than a range.

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

git diff --name-status ea2552d..HEAD \
  | grep -cE '^A[[:space:]]+docs/modernization/'
# -> 13                    every row is an addition under docs/modernization/

git diff --name-status ea2552d..HEAD \
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
[10 Appendix A](10-build-and-deployment-requirements.md#appendix-a--reproducibility). Neither figure is an estimation input — it is not in
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

**Five claims in this document cannot be established from the repository**, because they are properties
of a support lifecycle, a framework's published browser support or a tool's documented behaviour. Each
carries its source inline at the point of claim; they are collected here so a reviewer can re-check the
set in one pass. All five were reachable and read on the verification date.

| # | Claim it supports | Source | Where it is cited |
| --- | --- | --- | --- |
| 1 | .NET 8 is LTS, shipped 14 November 2023, and its support closes in November 2026; .NET 10 is the current LTS through November 2028 | Microsoft Learn, *.NET releases and support policy*, <https://learn.microsoft.com/dotnet/core/releases-and-support> — verified 2026-08-28 | [R1](#r1--the-target-framework-support-window) |
| 2 | The **exact** end-of-support day, **10 November 2026**, for .NET 8 and for .NET 9 — the Learn page gives the month, this table gives the day | Microsoft, *.NET and .NET Core Support Policy*, <https://dotnet.microsoft.com/platform/support/policy/dotnet-core> — verified 2026-08-28 | [R1](#r1--the-target-framework-support-window) |
| 3 | The target styling-framework major does not support Internet Explorer in any version | Bootstrap, *Browsers and devices*, <https://getbootstrap.com/docs/5.3/getting-started/browsers-devices/> — verified 2026-08-28 | [R7](#r7--the-narrowed-browser-matrix) |
| 4 | Static-file URLs inherit the underlying filesystem's case sensitivity; Windows is case-insensitive, Linux is not | Microsoft Learn, *Static files in ASP.NET Core*, <https://learn.microsoft.com/aspnet/core/fundamentals/static-files> — verified 2026-08-28 | [R8](#r8--case-sensitive-path-resolution-on-the-target-platform) |
| 5 | `git check-ignore` does not show tracked files by default, because tracked files are not subject to exclude rules; `--no-index` is what tests the rule | Git, *git-check-ignore Documentation*, <https://git-scm.com/docs/git-check-ignore> — verified 2026-08-28 | [R8](#r8--case-sensitive-path-resolution-on-the-target-platform), [A.5](#a5-the-corroborating-case-mismatch-r8) |
| 6 | A platform secret reference resolves an app setting from the key store under the site's managed identity | Microsoft Learn, *Use Key Vault references as app settings in Azure App Service and Azure Functions*, <https://learn.microsoft.com/azure/app-service/app-service-key-vault-references> — verified 2026-08-28 | [R17](#r17--the-interim-hosting-paths-stored-credential-and-a-time-box-with-no-box) |

**Two boundaries on this table.** It carries only what *this* document's own claims rest on. Every other
external fact the assessment uses — package versions and their servicing band, the migration tooling's
status, the hosting comparison, the adapters' targeting — is cited by the deliverable that owns it, and
citing it again here would create a second owner for it. And no package-registry source appears above,
because this document names no package version: the pins are [04 §7](04-dotnet8-migration-strategy.md)'s
and the test-stack versions are cited there, not here.
