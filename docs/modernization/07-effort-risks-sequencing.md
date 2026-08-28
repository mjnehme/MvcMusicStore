# 07 — Estimated Effort, Risks and Sequencing

> **Status: assessment output. This document authorizes nothing.**
> It sizes and sequences work that has **not** been approved, and it changes no repository file.
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

- **It is not a schedule.** There is no calendar, no date, no start or finish, no duration and no
  numbered delivery wave anywhere in it. [Section 8](#8-sequencing) explains why that is a feature:
  an effort band plus a dependency order lets a reader build a schedule from *their own* capacity and
  staffing assumptions, whereas a schedule computed here would silently embed assumptions that are
  not this assessment's to make.
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
| The DDL principal and the provisioning order | [06](06-azure-hosting-recommendations.md) §6.2, §6.3 | The scope behind W10 |
| **Per-edition build outcomes** | [10](10-build-and-deployment-requirements.md) §3 | [R2](#r2--the-migration-sources-build-reproducibility) cites the recorded outcome. Neither the outcome nor its diagnosis is restated |
| **The 22 blockers and their two groups** | [12](12-migration-blockers.md) §2.3 | Work items behind W7's estimate |
| **The categorized debt register and the counting methods** | [08](08-technical-debt-register.md) §2, §5–§11 | The counting rule this document is bound by, and an estimation input |
| Every package pin as-is; the restore-source finding | [02](02-dependency-inventory.md) §3, §6 | Scope behind W6; [R11](#r11--the-effective-package-source-set-is-not-knowable-from-the-repository) carries the source finding as a risk |
| Verified counts, code volume, view topology, asset groups | [01](01-architecture-overview.md) §2.3–§2.5 | Estimation inputs, cited per figure |

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
2. **Every effort figure is traceable and method-labelled.** No figure appears without both the count
   it derives from and the counting method that count uses. [Section 3](#3-the-two-counting-methods)
   states the rule; [section 4.1](#41-the-estimation-basis-every-input-with-its-method) is the
   traceability table.
3. **One fact, one owner** (AAP §0.11.4). [Section 1.4](#14-what-this-document-does-not-own--the-routing-table) is the routing table.
4. **No modification of any repository file.** [Section 2](#2-the-no-modification-constraint).

---

## 2. The no-modification constraint

The user directed **"Do not make code changes initially"** and **"Focus on assessment and planning
before implementation"**. The project's attached environment setup instructions restate the same gate
independently: **"Do not modify code until assessment and modernization plan are approved."** Two
inputs agreeing on it is why the boundary extends even to the defects this assessment identifies —
they are documented, not repaired.

**This document creates one file and changes none.** Every figure in it is a statement about work that
would happen *after* an approval, and AAP §0.11.5 makes the final check binary: `git status
--porcelain` must show exactly the new files under `docs/modernization/` and nothing else.

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
> authentication rewrite is estimated against** [src/MVC5/MvcMusicStore/Controllers/AccountController.cs].
> [08 §3.3](08-technical-debt-register.md) also records a **physical** line count for the same file as
> part of its duplication comparison. **That physical figure is not a sizing figure and appears
> nowhere in this document**, because quoting it in an effort table would inflate the authentication
> estimate by about ten percent while looking entirely reasonable.

Per [08 §12.2](08-technical-debt-register.md), the following are **excluded as effort inputs** and are
used nowhere below: the physical line counts for `AccountController.cs`; every diff-line count in
[08 §3](08-technical-debt-register.md); the physical line count of the seed file, whose sizing figure
is 820 non-blank; **severity ratings**, which measure consequence and not cost; and **entry counts** —
"28 debt findings" is not a workload, because the entries are not comparable units.

### 3.3 A third kind of count, named so it is not mistaken for either

Some inputs are **file counts** or **call-site counts** — 29 views, 27 static assets, 10 injection
sites, 22 blockers. These are neither line-count method. They are labelled **"file count"** or
**"site count"** wherever they appear, because an unlabelled integer next to two labelled line metrics
invites exactly the confusion §3.1 exists to prevent.

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
| 7 | Static assets to relocate | **27** in the migration source's four asset groups | File count | [01 §2.3](01-architecture-overview.md) |
| 8 | Bundling helper call sites | **11** — 10 `@Scripts.Render`, 1 `@Styles.Render` | Site count | [Appendix A.3](#a3-helper-view-and-site-counts) |
| 9 | Child actions to convert | **3** | Site count | [01 §5.3](01-architecture-overview.md) |
| 10 | Manual construction sites | **10** | Site count | [01 §5.4](01-architecture-overview.md) |
| 11 | Other Razor helper sites | **10** anti-forgery emissions, **5** partials, **4** `@Url.Content` | Site count | [Appendix A.3](#a3-helper-view-and-site-counts) |
| 12 | Blockers to resolve | **22** — **14** compile-time, **8** silent-runtime | Entry count of work items | [12 §2.3](12-migration-blockers.md) |
| 13 | Package outcomes to apply | **28** pins in the migration source | Pin count | [04 §8](04-dotnet8-migration-strategy.md) |
| 14 | Required test coverage | **27** surfaces; rows 1–22 against **two** fixtures, 23–27 target-only | Row count | [05 §12.4](05-aspnet-core-migration-approach.md) |
| 15 | Tests existing today | **0** | Absence, command-verified | [08 §7.3](08-technical-debt-register.md), [A.2](#a2-the-absences-that-size-the-net-new-work) |
| 16 | Distinct page views for visual capture | **22** non-partial view files of the 29 | File count | [A.4](#a4-the-visual-review-capture-set-input-16) |
| 17 | Approved deltas requiring sign-off | **14**, across **5** approver constituencies | Entry count | [05 §11.5](05-aspnet-core-migration-approach.md) |
| 18 | Unvalidated state-changing POSTs | **5** in the migration source | Census | [08 §5.5](08-technical-debt-register.md) |
| 19 | Repository-hygiene volume | **14** database binaries at **43,376,640** bytes; **215** committed package files; **4** solutions for **3** projects | File count and byte count | [08 §6.2, §10.4, §10.2](08-technical-debt-register.md) |

**Inputs 2, 3 and 4 are a partition of input 1**, and the arithmetic is worth stating because it is the
backbone of the whole model: **382 + 820 + 895 = 2,097** non-blank lines, all four figures the sizing
metric. Everything other than the authentication rewrite is therefore **1,715** non-blank lines — a
figure this document uses only as a cross-check, because [08 §4.2](08-technical-debt-register.md)
splits that remainder in two on purpose. Treating 1,715 as one homogeneous block of porting work would
re-introduce exactly the overestimate the next note describes.

**Three notes on how these inputs are used, each of which changes an estimate materially.**

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
  [section 7.3](#73-repository-hygiene--sized-but-outside-the-critical-path) and gating nothing.

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
| **A9** | **No new functional requirement** is introduced. The behaviour baseline is [05 §11.5](05-aspnet-core-migration-approach.md)'s: preserved outcomes plus 14 approved deltas | Any new feature is outside this model entirely |
| **A10** | The **password hashes in the shipped credential store validate** under the target framework's default hasher | W8's high band becomes the expected case and [R5](#r5--identity-migration-rollback) escalates from a rollback risk to a redesign |

### 4.4 Confidence, and its reason

**Overall confidence: medium.** Stated with its reason, because a bare adjective is not a confidence
statement:

> **Medium, because the two halves of the work have opposite estimation properties.** The mechanical
> port is **well bounded**: the migration source is 2,097 non-blank lines (sizing metric), its
> decomposition is measured to the file [08 §4.2](08-technical-debt-register.md), all 22 blockers are
> enumerated with a named resolution each [12 §2.3](12-migration-blockers.md), and the build now has a
> recorded clean outcome on the prescribed toolchain [10 §3.1](10-build-and-deployment-requirements.md).
> Against that, **the two data migrations are estimated against a schema that has not been
> extracted** — the authoritative definition exists only inside a committed binary, the migration
> source ships no schema script, and [12 §5](12-migration-blockers.md) explicitly qualifies the
> Identity column set as *evidence rather than proof*. **No amount of source reading narrows that**;
> only W3 does.

Confidence is therefore **not uniform**, and the per-workstream column in
[section 5](#5-effort-by-workstream) states it per row:

| Confidence | Workstreams | Why |
| --- | --- | --- |
| **High** | W2, W5, W6, W14, W15 | Fully enumerated, small, and verifiable on completion. W2's outcome is already recorded |
| **Medium** | W1, W7, W10, W11, W12, W13, and [§7.1](#71-the-manual-visual-review) | Scope is known and enumerated; the variance is execution and, for W1 and W13, other people's decisions |
| **Low** | W3, W4, W8, W9 | Each depends on a fact not yet established: the extracted schema for three of them, and for W4 the behaviour of a system that has never been characterized by a test |

**The low-confidence rows carry 40 of the 126.5 expected IED** — see
[section 6](#6-totals-and-where-the-effort-actually-lives) — which is the single most useful thing an
approver can know about this estimate. W3 is cheap and retires most of that uncertainty, which is why
[03](03-modernization-roadmap.md) places it before the port and why
[section 8.3](#83-the-critical-path-and-what-to-do-first-if-the-goal-is-to-narrow-the-estimate) recommends it as the first
substantive action.

---

## 5. Effort by workstream

**The decomposition below is [03 §5](03-modernization-roadmap.md)'s, not this document's.** All fifteen
workstreams appear, in 03's order, under 03's names. No workstream is added, renamed, merged or split,
because a second decomposition would leave the roadmap and the estimate unable to reference each other.
What this document adds to each is a band, a confidence and the inputs the band derives from.

### 5.1 Summary table

Ideal engineer-days ([section 4.2](#42-the-unit-defined)). Every figure traces to a numbered input in
[section 4.1](#41-the-estimation-basis-every-input-with-its-method).

| Workstream ([03 §5](03-modernization-roadmap.md)) | Low | Expected | High | Confidence | Primary inputs |
| --- | ---: | ---: | ---: | --- | --- |
| **W1** — Approval of this assessment | 3 | 6 | 12 | Medium | 17 |
| **W2** — MVC 5 build reproduction and the restore precondition | 0.5 | 1 | 3 | High | — (outcome recorded in [10 §3.1](10-build-and-deployment-requirements.md)) |
| **W3** — Authoritative schema extraction | 2 | 4 | 8 | **Low** | 1, 12 |
| **W4** — Pre-port behavioural baseline and test suite | 12 | 20 | 34 | **Low** | 14, 15 |
| **W5** — Repository-wide path-casing audit | 1 | 2 | 4 | High | 7, 8, 11 |
| **W6** — Project-format conversion and dependency transition | 2 | 4 | 8 | High | 13 |
| **W7** — The ASP.NET Core port | 21 | 40 | 69 | Medium | 2, 3, 4, 6, 8, 9, 10, 11, 12 |
| **W8** — Identity schema and data migration | 4 | 8 | 16 | **Low** | 12 |
| **W9** — Domain data migration | 4 | 8 | 15 | **Low** | 1, 12 |
| **W10** — Hosting provisioning and platform configuration | 5 | 9 | 16 | Medium | — (scope from [06 §6](06-azure-hosting-recommendations.md)) |
| **W11** — CI provider selection, then pipeline authoring | 4 | 7 | 13 | Medium | 15 |
| **W12** — Administrator provisioning tool | 1.5 | 3 | 6 | Medium | — (scope from [05 §10.2](05-aspnet-core-migration-approach.md)) |
| **W13** — Cutover | 2 | 4 | 8 | Medium | — (scope from [06 §11](06-azure-hosting-recommendations.md)) |
| **W14** — Documentation revision | 1 | 2 | 3 | High | — |
| **W15** — Affinity retirement | 0.5 | 1 | 2 | High | — |
| *Non-code:* manual visual review ([§7.1](#71-the-manual-visual-review)) | 2 | 3.5 | 6 | Medium | 16 |
| *Non-code:* approved-delta sign-offs ([§7.2](#72-the-approved-delta-sign-offs)) | 2 | 4 | 8 | Medium | 17 |
| **Total** | **67.5** | **126.5** | **231** | Medium overall | |

The two non-code rows are listed with the workstreams because they consume real effort and because no
other document sizes them, but they are **not** workstreams in [03](03-modernization-roadmap.md)'s
decomposition and are not presented as such. [Section 7](#7-work-that-is-not-code) states where each
attaches.

### 5.2 Basis of estimate, per workstream

#### W1 — Approval of this assessment

**Scope** is [03 §5](03-modernization-roadmap.md)'s: the gate on everything, exiting with a documented
approval that records a decision on each approved delta and on each risk this document escalates for
decision rather than mitigation.

**Basis.** Reading thirteen deliverables, then **14 delta decisions** across **5 approver
constituencies** (input 17, entry count) plus the four risks
[section 9.4](#94-the-four-risks-that-are-approval-decisions-not-mitigations) escalates as approval
decisions. The band is driven by how many constituencies must convene rather than by document length.
**Assumption A5** applies: if approvers are not yet named people, this row grows and everything
downstream inherits it.

**Band 3 / 6 / 12.** Low assumes an existing governance forum and pre-identified approvers; high
assumes approvals must be sought individually with a round of clarification each.

#### W2 — MVC 5 build reproduction and the restore precondition

**Scope** is [03 §5](03-modernization-roadmap.md)'s: a recorded build result for the migration source
from a clean checkout, plus confirmation that the restore is reproducible rather than incidental.

**Basis — and this row is smaller than the assessment first expected.** The prescribed-toolchain
verification run has **already been performed and recorded** by
[10 §3.1](10-build-and-deployment-requirements.md), which is the owner of per-edition build outcomes.
What remains is not a build but a **reproducibility exercise**: declaring the restore source explicitly
so the result does not depend on host configuration, and recording the evidence in the form W2's exit
gate requires. [R2](#r2--the-migration-sources-build-reproducibility) carries the residual risk.

**Band 0.5 / 1 / 3.** The high band exists solely for the trigger in
[R2](#r2--the-migration-sources-build-reproducibility): a re-run on a differently configured host
reveals something the recorded run did not.

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

#### W4 — Pre-port behavioural baseline and test suite

**Scope** is [03 §5](03-modernization-roadmap.md)'s, with the architecture and required coverage owned
by [05 §12](05-aspnet-core-migration-approach.md). Not redesigned here.

**Basis — entirely net-new, with no legacy volume to scale from.** There are **0 tests** in the
repository (input 15, command-verified in [A.2](#a2-the-absences-that-size-the-net-new-work)), so this
is not a migration of a suite but the construction of one. Three cost drivers, all from
[05 §12](05-aspnet-core-migration-approach.md):

- **27 required coverage surfaces** (input 14, row count), of which **rows 1–22 must run against two
  fixtures** — the legacy baseline and the ported application — because the suite's purpose is to
  characterize one runtime and then assert the other against it. That is materially more than 27 test
  authorings.
- **Two fixtures, each with real infrastructure work.** The legacy fixture must reset **both**
  committed database pairs from copies outside the checkout, and rename a fixed catalog name to avoid
  collision. The target fixture must provision a disposable SQL Server-compatible instance, create the
  session cache table, apply both migration sets, create the key table, load a *migrated* dataset and
  assert its row counts before any test runs.
- **Semantic rather than byte comparison.** Anti-forgery tokens, session identifiers, cookies and
  timestamps must be normalized out, and the 14 approved deltas recorded as expected differences.

**Band 12 / 20 / 34. Low confidence** — the suite characterizes behaviour nobody has yet asserted, so
the count of surfaces is known while the count of *surprises* is not.
**This is the second-largest row in the model and the one most likely to be cut under pressure**;
[R3](#r3--the-absent-regression-baseline) is why cutting it removes the only means of substantiating
behaviour preservation.

#### W5 — Repository-wide path-casing audit

**Scope** is [03 §5](03-modernization-roadmap.md)'s: every mismatch identified and its correction
specified, with the audit made **repeatable as a pre-deployment check** rather than performed once.

**Basis.** The search space is small and fully enumerated: **27 static assets** (input 7, file count),
**11 bundling helper call sites** and **4 `@Url.Content` sites** (inputs 8 and 11, site counts), plus
view paths. At least one live mismatch is already known
[src/MVC5/MvcMusicStore/App_Start/BundleConfig.cs:28]. Most of the band is the *repeatability*
requirement rather than the audit.

**Band 1 / 2 / 4.** [R8](#r8--case-sensitive-path-resolution-on-the-target-platform) carries the risk.

#### W6 — Project-format conversion and dependency transition

**Scope** is [03 §5](03-modernization-roadmap.md)'s, with the format, the manifests and the pins owned
by [04 §5, §6, §7](04-dotnet8-migration-strategy.md).

**Basis.** **28 package outcomes to apply** (input 13, pin count), each already decided with exactly
one outcome by [04 §8](04-dotnet8-migration-strategy.md) — so this is application of a completed
decision, not analysis. Plus the project-format conversion, four net-new tooling manifests, and four
solution files collapsing to one. The exit gate requires the W4 suite to run on the converted build,
which is what keeps the row honest: conversion is not done when it compiles.

**Band 2 / 4 / 8. High confidence** — every input is enumerated and the outcome is binary.

#### W7 — The ASP.NET Core port

**Scope** is [03 §5](03-modernization-roadmap.md)'s, with every design decision owned by
[05 §2–§9](05-aspnet-core-migration-approach.md) and the file-by-file target owned by
[04 §12](04-dotnet8-migration-strategy.md).

**Basis — decomposed, because a single band over the largest row would hide its structure.** All line
figures are the **sizing metric**; all others are labelled.

| Component | Input | Low | Expected | High |
| --- | --- | ---: | ---: | ---: |
| Composition root and dependency injection | **10** manual construction sites (site count) | 2 | 4 | 7 |
| Ordinary application code | **895** non-blank lines (sizing) | 5 | 9 | 15 |
| Authentication rewrite | **382** non-blank lines (sizing) | 6 | 11 | 18 |
| Seed data — a data decision, **not** line porting | **820** non-blank lines (sizing) | 1 | 2 | 4 |
| Views, helpers and view components | **29** views, **5** naming legacy types; **11** bundling sites; **3** child actions (file and site counts) | 4 | 8 | 14 |
| Static assets and their acquisition | **27** assets in the four asset groups (file count) | 1 | 2 | 4 |
| Demonstrating the silent-runtime resolutions | **8** of the **22** blockers (work-item count) | 2 | 4 | 7 |
| **W7 total** | | **21** | **40** | **69** |

**Four notes on that decomposition, each of which is a place a naive estimate goes wrong.**

- **The authentication rewrite is 382 non-blank lines (sizing metric), ~18 percent of the migration
  source** — and it is the largest single sub-row despite that share, because it is the one component
  with no line-for-line successor: its stack is replaced rather than ported, and
  [05 §11.5](05-aspnet-core-migration-approach.md) row 11 also removes the scaffolded external-login
  surface as part of it.
- **The seed is 39 percent of the source by the sizing metric and about 5 percent of this row's
  effort.** That asymmetry is the whole point of [08 §4.2](08-technical-debt-register.md)'s
  instruction: 820 non-blank lines of hardcoded catalog rows are bulk data expressed as code, and the
  work is deciding how the data reaches the database, not porting 820 lines.
- **Ordinary application code — 895 non-blank lines — is the genuinely mechanical part**, and it is
  only 43 percent of the source and about 22 percent of this row. An estimate that scaled from total
  lines would land almost entirely in the wrong place.
- **The 8 silent-runtime blockers get their own sub-row on purpose.** Their exit criterion
  [03 §5](03-modernization-roadmap.md) is *demonstration*, not code review — because their failure mode
  is silence, an assertion that fails when the resolution is absent is the only acceptable evidence.
  That cost belongs to W7 even though the assertions live in W4's suite.

**Band 21 / 40 / 69. Medium confidence** — the scope is enumerated to the file and the blocker, and the
variance is execution rather than discovery.

#### W8 — Identity schema and data migration

**Scope** is [03 §5](03-modernization-roadmap.md)'s, exiting on reconciliation passing; the migration
design is owned by [05 §5.5](05-aspnet-core-migration-approach.md).

**Basis.** Creating the target tables fresh and populating them, then reconciling. The cost drivers are
not volume — the shipped store is small — but **correctness obligations**: collision detection on the
normalized columns before writing, defined origins for columns that have no source value, verification
that the shipped hashes validate, and reconciliation of account, role and assignment counts with the
administrator's membership asserted specifically.

**Band 4 / 8 / 16. Low confidence**, for a reason stated precisely: this workstream's *source schema*
is the one [12 §5](12-migration-blockers.md) qualifies as **evidence rather than proof**. The high band
is what this costs if **A10** is false and the hashes do not validate.
[R5](#r5--identity-migration-rollback) carries it.

#### W9 — Domain data migration

**Scope** is [03 §5](03-modernization-roadmap.md)'s, entered through what that document calls the hard
gate: the generated-schema diff must pass before any data is loaded.

**Basis.** Six entities, loaded in dependency order, with reconciliation of row counts per table and
financial totals per order. The volume is modest and the **gate** is the cost: a diff between a
generated migration and the real schema, iterated until it passes, against a schema that W3 must first
extract because the migration source **ships no schema script** and neither committed copy of the
other edition's script is usable [12 §5](12-migration-blockers.md).

**Band 4 / 8 / 15. Low confidence.** [R4](#r4--domain-data-migration-rollback) carries the risk,
including the rows-written-between-extraction-and-cutover problem, which is a *design* obligation
inside this band rather than an afterthought.

#### W10 — Hosting provisioning and platform configuration

**Scope** and every mechanism are owned by [06 §6–§9](06-azure-hosting-recommendations.md): the
provisioning order, the DDL principal separated from the runtime identity, the persisted key ring,
session over the distributed cache, transport enforcement and the observability placement.

**Basis.** Environment provisioning, identity and secret wiring, the ordered creation of four schema
owners, transport and header configuration, and the observability wiring that is **net-new capability**
rather than migration — the repository has none of it today
[08 §7.1](08-technical-debt-register.md). Assumption **A8** applies.

**Band 5 / 9 / 16. Medium confidence** — the steps are enumerated by 06; the variance is
environment-specific.

#### W11 — CI provider selection, then pipeline authoring

**Scope** is [03 §5](03-modernization-roadmap.md)'s, strictly in that order: a provider decision, then
a manifest.

**Basis — net-new, with an approval inside it.** The repository has **no** pipeline definition, publish
profile or container manifest [08 §7.2](08-technical-debt-register.md), so nothing is migrated. The
band covers the provider decision (an approval, not engineering), then build, test, publish and the
deployment-time migration step that [06 §6.2](06-azure-hosting-recommendations.md) requires be run
under a principal distinct from the runtime identity.

**Band 4 / 7 / 13. Medium confidence** — the shape is standard; the provider is unchosen, and
[04 §6](04-dotnet8-migration-strategy.md)'s locked-mode restore adds a step that must actually be made
to fail correctly.

#### W12 — Administrator provisioning tool

**Scope** is [03 §5](03-modernization-roadmap.md)'s, exiting with all five properties of
[05 §10.2](05-aspnet-core-migration-approach.md) demonstrated.

**Basis.** A small console project. The effort is in the five properties rather than the code: a secret
channel that keeps the credential out of process listings and shell history, hashing through the
framework's own user manager rather than direct SQL, **idempotence checked per operation** so a prior
partial run is repaired rather than skipped, an audit record with no secret in it, and exclusion from
the deployed web application.

**Band 1.5 / 3 / 6. Medium confidence.**

#### W13 — Cutover

**Scope** is [03 §5](03-modernization-roadmap.md)'s; the runbook is owned by
[06 §11](06-azure-hosting-recommendations.md) and the approach and its accepted losses by
[05 §11](05-aspnet-core-migration-approach.md). **Neither is reopened here.**

**Basis.** Executing a runbook: drain, switch, verify, with the rollback position confirmed beforehand.
The band covers rehearsal as well as execution, because a runbook first executed in production is not a
runbook. Six workstreams must have exited before this begins, which is why
[section 8](#8-sequencing) treats it as the convergence point rather than a step.

**Band 2 / 4 / 8. Medium confidence** — the mechanics are enumerated; the variance is the window and
other people's availability. [R9](#r9--cutover-re-authentication-and-anonymous-cart-loss) carries the
two accepted losses as risks.

#### W14 — Documentation revision

**Scope** is [03 §5](03-modernization-roadmap.md)'s: three tracked documents revised to describe the
target workflow, with the two reference editions' documents marked historical.

**Basis.** Three files [README.md], [src/MVC4/README.md], [src/MVC5/README.md], each documenting a
workflow the target replaces. Terminal in the graph.

**Band 1 / 2 / 3. High confidence.**

#### W15 — Affinity retirement

**Scope** is [03 §5](03-modernization-roadmap.md)'s: a verification, not an elapsed interval, followed
by one platform setting.

**Basis.** The cross-instance session round-trip must pass with affinity off in a non-production
environment first; the production change is then a single reversible setting. Terminal in the graph.

**Band 0.5 / 1 / 2. High confidence** — and it is a genuine workstream rather than a footnote because
skipping it leaves the scale-out property that motivated part of W10 unrealized.

---

## 6. Totals, and where the effort actually lives

### 6.1 The totals

| | Low | Expected | High |
| --- | ---: | ---: | ---: |
| **All fifteen workstreams plus both non-code tasks** | **67.5** | **126.5** | **231** IED |

The high band is about **3.4 times** the low band. That spread is not estimator hedging — it is the
arithmetic consequence of four low-confidence rows worth **40 expected IED** whose difficulty depends on
a schema that has not been extracted and on behaviour that has never been asserted by a test
([section 4.4](#44-confidence-and-its-reason)).

### 6.2 The finding that matters most in this document

**The port of existing code is about a third of the expected effort. Everything else is
approximately two thirds.**

| Character of the work | Workstreams | Expected IED | Share |
| --- | --- | ---: | ---: |
| **Porting existing code** | W6, W7 | 44 | ~35% |
| **Net-new capability, with no legacy volume to scale from** | W4, W10, W11, W12, W15 | 40 | ~32% |
| **Data work gated on a schema not yet extracted** | W3, W8, W9 | 20 | ~16% |
| **Governance, verification and documentation** | W1, W2, W5, W13, W14, and both non-code tasks | 22.5 | ~18% |
| **Total** | | **126.5** | **100%** |

**This is the conclusion an estimate derived from lines of code cannot reach.** Scaling from the
migration source's 2,097 non-blank lines (sizing metric) would produce a number for the 35 percent and
**silently omit the other 65 percent**, because none of it is predicted by any quantity in the existing
codebase:

- **The test suite has no legacy volume at all.** There are **0** tests today (input 15), and the suite
  is required against **27** coverage surfaces with **22 of them run twice** (input 14). It is the
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
non-blank lines (sizing metric), 39 percent of the migration source, and about 5 percent of W7's band,
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
  [section 7.3](#73-repository-hygiene--sized-but-outside-the-critical-path) because it gates nothing.
- **Any new functional requirement.** Assumption **A9**.
- **Elapsed time.** [Section 8](#8-sequencing) gives the dependency order and the concurrency the graph
  permits; converting to a schedule needs a capacity assumption this document does not make.

---

## 7. Work that is not code

Both items below consume real effort, neither is sized by any other document, and neither is a
workstream in [03 §5](03-modernization-roadmap.md)'s decomposition. They are carried here because
omitting them would understate the total.

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
baseline, and which of the approved deltas were accepted.

**Basis.** **22 non-partial view files of the 29** (input 16, file count) bound the capture set; two
viewports each puts the capture in the mid-tens of screenshots. Assumption **A4** applies — one
reviewer, with a second signature only for approval.

**Band 2 / 3.5 / 6 IED**, split across two points in the sequence, which is the part easiest to get
wrong:

| Part | When | Note |
| --- | --- | --- |
| Baseline capture | **Before W7 begins**, against the running legacy application | If this is missed, the baseline is unrecoverable — the thing being captured no longer exists once the port lands, and the review then has nothing to compare against |
| Review and sign-off | **After W7 exits** | Reviews the ported rendering against the captured baseline |

### 7.2 The approved-delta sign-offs

**Why it exists as a separate task.** [05 §11.5](05-aspnet-core-migration-approach.md) enumerates
**14 approved deltas**, each with a **named approval owner** — and an approval owner who has not
actually approved is an open question, not a delta. Obtaining the decisions is work with elapsed
effort, and [03 §5](03-modernization-roadmap.md)'s W1 exit gate requires a recorded decision on each.

**Basis.** **14 deltas** across **5 approver constituencies** (input 17, entry count) — the security,
product, engineering, data and operations owners named by 05 §11.5. Two properties drive the band
rather than the count: several deltas need **more than one** constituency to consent, and one of them
is not a technical trade at all —
[R7](#r7--the-narrowed-browser-matrix) removes a class of client and belongs to the product owner
alone.

**Band 2 / 4 / 8 IED.** This is effort **inside W1**, reported separately so it is visible; the summary
table in [section 5.1](#51-summary-table) lists it once and does not double-count it into W1's own row.

**Where a withheld consent leads.** [03 §5](03-modernization-roadmap.md) W1 records that if an approval
owner withholds consent, the affected workstream's scope changes and the roadmap is revised before that
workstream begins. That is a re-estimation, not a variance inside these bands.

### 7.3 Repository hygiene — sized, but outside the critical path

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

**No calendar, no dates, no durations.** The sets below are a property of the dependency graph, not a
schedule and not a time box. Nothing here says when anything starts.

### 8.2 Concurrency permitted by the graph

Each set contains workstreams whose entry gates are satisfied by the sets above it, and which have no
dependency on each other. A reader with a capacity assumption can turn this into a schedule; this
document does not.

| Set | Workstreams available concurrently | Expected IED in the set | Gate that opens it |
| --- | --- | ---: | --- |
| **0** | W1 (including the sign-offs of [§7.2](#72-the-approved-delta-sign-offs)) | 6 | The root. Nothing precedes it |
| **1** | **W2, W3, W5, W11** — four independent streams, plus the **baseline capture** of [§7.1](#71-the-manual-visual-review) | 14 | W1 exited |
| **2** | **W4** | 20 | W2 exited |
| **3** | **W6** | 4 | W2 and W4 exited |
| **4** | **W7**; **W10** may begin its non-migration steps in parallel on W5 and W6 alone | 40 (+9 overlapping) | W3, W4, W6 exited |
| **5** | **W10**'s migration-applying steps; then **W8**, **W9** and **W12** — W8 and W9 concurrent with each other; **W14** available immediately and terminal | 21 | W7 exited; W10 through its ordered steps |
| **6** | **W13** — the convergence point | 4 | W7, W8, W9, W10, W11, W12 all exited |
| **7** | **W15** (terminal); the **review and sign-off** of [§7.1](#71-the-manual-visual-review) | 1 + 2 | W13 exited |

**Four properties of this shape are worth stating explicitly.**

- **Set 1 is the widest point in the plan.** Four workstreams and a capture task proceed independently
  on nothing but W1's approval. A team of one gets 14 IED of sequential work; a team of four gets the
  longest single item. **This is where staffing changes the elapsed shape most.**
- **W4 is a genuine bottleneck, and it is on the critical path twice.** W6 needs it — the converted
  build must run the suite — and W7 is judged against it. It cannot be parallelized away, because it is
  one coherent fixture-plus-suite artifact rather than 27 independent items.
- **W10 straddles a set.** Its non-migration steps need only W5 and W6, but its migration-applying
  steps need W7's migrations. [03 §4.2](03-modernization-roadmap.md) records this as the graph's one
  partial dependency, and it is a real overlap opportunity rather than a technicality.
- **W8 and W9 are concurrent with each other** — separate stores, separate reconciliations, both gated
  on the same three predecessors. Running them concurrently needs two data engineers; running them in
  sequence adds their full expected sum to the elapsed shape.

### 8.3 The critical path, and what to do first if the goal is to narrow the estimate

**The longest dependency chain** is W1 → W2 → W4 → W6 → W7 → W10 → W8 *or* W9 → W13 → W15:

| | Low | Expected | High |
| --- | ---: | ---: | ---: |
| **Critical path** | 50 | **93** | 168 IED |
| Off the critical path | 17.5 | 33.5 | 63 IED |

So about **26 percent of the expected effort sits off the critical path** and can be absorbed by
parallel capacity; the remaining 93 expected IED cannot, no matter how many engineers are added.

**If the objective is to narrow the estimate rather than to shorten it, the first substantive action is
W3.** It costs **4 expected IED** and it is the input to three of the four low-confidence rows:

- It replaces the **evidence-rather-than-proof** qualification on the Identity column set
  [12 §5](12-migration-blockers.md) with fact, which is what W8's band is wide because of.
- It supplies the diff baseline W9's hard entry gate requires, in a repository where the migration
  source **ships no schema script**.
- It resolves two of the 22 blockers outright (input 12).

**W3 depends only on W1 and sits in the widest concurrency set**, so it is available immediately after
approval and blocks nothing while it runs. Sequencing it first is the cheapest available reduction in
the *width* of the total — it does not make the project smaller, it makes the estimate truer, which is
a different and often more valuable thing.

**One sequencing hazard has no cost in any band and is unrecoverable if missed.** The visual baseline
capture ([§7.1](#71-the-manual-visual-review)) must happen **while the legacy application still runs**.
It is 1 IED of work in concurrency set 1 that cannot be done later at any price, because the artifact it
captures ceases to exist.

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

### 9.2 Register index

| ID | Risk | Likelihood | Impact | Owner | Affected workstreams |
| --- | --- | --- | --- | --- | --- |
| [R1](#r1--the-target-framework-support-window) | The target framework's support window | **Certain** | High | Engineering leadership *(approval decision)* | W6, W7, W10, W11 |
| [R2](#r2--the-migration-sources-build-reproducibility) | The migration source's build reproducibility | Low | Medium | Build and release engineering | W2, W6 |
| [R3](#r3--the-absent-regression-baseline) | The absent regression baseline | High | **Critical** | Quality engineering | W4, W7, W13 |
| [R4](#r4--domain-data-migration-rollback) | Domain data migration rollback | Medium | **Critical** | Data engineering, with the data owner | W3, W9, W13 |
| [R5](#r5--identity-migration-rollback) | Identity migration rollback | Medium | High | Data engineering, with security | W3, W8, W13 |
| [R6](#r6--security-hardening-versus-compatibility) | Security hardening versus compatibility | Medium | High | Security | W7, W13 |
| [R7](#r7--the-narrowed-browser-matrix) | The narrowed browser matrix | **Certain** | Medium | **Product owner** *(approval decision)* | W7, W14 |
| [R8](#r8--case-sensitive-path-resolution-on-the-target-platform) | Case-sensitive path resolution on the target platform | Medium | High | Engineering | W5, W10, W13 |
| [R9](#r9--cutover-re-authentication-and-anonymous-cart-loss) | Cutover re-authentication and anonymous-cart loss | **Certain** | Medium | Product and operations *(approval decision)* | W13, W15 |
| [R10](#r10--scoping-by-analogy-across-editions) | Scoping by analogy across editions | Medium | Medium | Engineering leadership, with this document | W1, W7 |
| [R11](#r11--the-effective-package-source-set-is-not-knowable-from-the-repository) | The effective package source set is not knowable from the repository | Medium | High | Build and release engineering, with security | W2, W6, W11 |
| [R12](#r12--no-observability-exists-during-the-cutover-itself) | No observability exists during the cutover itself | High | High | Operations and platform | W10, W13 |
| [R13](#r13--one-database-one-blast-radius) | One database, one blast radius | Low | High | Platform and operations *(approval decision)* | W10, W13 |
| [R14](#r14--a-reference-editions-retired-data-provider) | A reference edition's retired data provider | Low | Low | Engineering leadership | None while A7 holds |

### 9.3 The entries

#### R1 — The target framework support window

**This is the first entry because it is the only risk in the register that must be decided before any
other work begins, and because it is an approval decision rather than a technical alternative.**

[04 §2](04-dotnet8-migration-strategy.md) targets **`net8.0`** without hedging, which is correct: it is
what the user asked for, and a strategy document is not the place to substitute a different answer to
the question it was asked. [04 §2.2](04-dotnet8-migration-strategy.md) accordingly records that the
support window *"is an approval decision, and it belongs to 07"*. **This entry is that decision, and it
is left with the approver.**

**The dates.** .NET 8 is an LTS release that shipped **14 November 2023** on a **36-month** LTS window,
and its support **ends 10 November 2026**. **.NET 9 shares that same end date**, so it is not an
alternative. The current LTS release at the time of this assessment is **.NET 10**, supported through
**November 2028**.

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

**This entry's status changed during the assessment, and the change is stated rather than hidden.** The
assessment initially had to carry the migration source's build as *unverified* — and the migration
source is the one edition the entire port depends on. **[10 §3.1](10-build-and-deployment-requirements.md)
has since recorded the prescribed-toolchain verification run as performed and authoritative, with a
clean outcome in both configurations.** Deliverable 10 owns per-edition build outcomes; this entry cites
that record and does not restate the outcome or its diagnosis.

**So the original risk is retired by evidence, and what remains is a residual with three parts** —
worth carrying because each could still surface in W2 or W6:

- **A clean checkout still cannot build.** The migration source commits no restored packages, so the
  build depends on a restore succeeding first. That is a precondition, not a defect, and W2's exit gate
  exists to prove it is reproducible rather than incidental.
- **The restore source that made it work is a property of the host, not of the repository.** See
  [R11](#r11--the-effective-package-source-set-is-not-knowable-from-the-repository), which owns that
  finding.
- **A clean build says nothing about the views.** View compilation is disabled
  [08 §8.1](08-technical-debt-register.md), so the **29** views (input 6, file count) carry no
  build-time guarantee and were not exercised by the recorded run. Their first real check is W4's suite
  and W7's rendering.

| Field | Value |
| --- | --- |
| **Likelihood** | **Low**, reduced from High by the recorded run in [10 §3.1](10-build-and-deployment-requirements.md). It is no longer an open question, only an unrepeated one |
| **Impact** | **Medium.** A defect found at W2 is cheap; the same defect found during W6 is disruptive but recoverable |
| **Mitigation** | W2 reproduces the build from a clean checkout with an **explicitly declared** restore source, and records tool versions, the source resolved, configurations built and warning and error counts — so the result does not depend on host configuration |
| **Contingency** | Any defect the reproduction reveals is triaged before W6 begins. Because W6 depends on W2, a defect delays rather than derails; the port has not started |
| **Trigger** | **A reproduction run reveals a defect beyond the missing restore** — anything other than a restore precondition. Secondarily: a build that succeeds on one host and fails on another, which indicts the source configuration rather than the code |
| **Owner** | **Build and release engineering** |
| **Affected workstreams** | W2, and W6 as its consumer |

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

| Field | Value |
| --- | --- |
| **Likelihood** | **High.** Not that the baseline is absent — that is a verified fact — but that a behaviour change ships undetected if the port proceeds without one. Eight blockers are specifically silent |
| **Impact** | **Critical.** Without a baseline there is no evidence for the migration's central premise, and a regression's first detector is a user |
| **Mitigation** | W4 precedes the port, as [03 §4.1](03-modernization-roadmap.md) sequences it. The suite is HTTP-level and semantic so that **one suite characterizes both runtimes**, with volatile values normalized out and the 14 approved deltas recorded as expected differences rather than failures |
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

**The rows-written-between-extraction-and-cutover problem is part of this entry, not a footnote.** A
migration is extracted at one moment and cut over at another. Every order placed, cart modified and
album edited in between is a row the extract does not contain. Whether those rows are captured by a
delta pass, excluded by a write freeze, or made impossible by draining the legacy application during
the window is a **design decision W9 must make and record**, not an operational detail to be discovered
in the window. Getting it wrong loses orders — which is why this entry's impact is Critical.

| Field | Value |
| --- | --- |
| **Likelihood** | **Medium.** The domain is six entities with no exotic mapping, but the diff is being run for the first time against a schema that has never been extracted |
| **Impact** | **Critical.** The data at stake is orders and customer PII, and the destructive initializer [08 §6.1](08-technical-debt-register.md) is proof that this database has no automated protection against a model mismatch today |
| **Mitigation** | W3 first, so the diff has an authoritative baseline. The **generated-schema diff must pass before any data is loaded** — [03 §5](03-modernization-roadmap.md) calls this the hard gate. Load in dependency order; reconcile row counts per table **and financial totals per order**; rehearse the whole sequence against a representative dataset per assumption **A3** |
| **Contingency** | The source database is **not** modified and is retained until reconciliation passes, so rollback is a redirection to the source rather than a restore. Reconciliation failure stops the cutover; it does not get accepted with a note |
| **Trigger** | The generated-schema diff does not converge after a bounded number of iterations; or reconciled row counts or per-order totals differ by any amount. **Any** discrepancy is the trigger — a small one is not a rounding artifact in a financial total |
| **Owner** | **Data engineering**, with the **data owner** as approver, per [03 §5](03-modernization-roadmap.md) W9 |
| **Affected workstreams** | W3 as the precondition, W9 as the work, W13 as the consumer |

#### R5 — Identity migration rollback

**The risk is that the credential store cannot be migrated as designed, and that it is discovered after
cutover** — when the symptom is users unable to sign in.

**Its distinguishing feature is that this entry's source schema is the one thing in the assessment that
is explicitly qualified.** [12 §5](12-migration-blockers.md) records the Identity column set as
**evidence rather than proof**, arrived at by inspecting a binary rather than by querying a catalog.
That qualification is cited here, not re-derived, and it is exactly what W3 exists to remove.

**Three concrete failure modes, each with a different response.**

- **Normalized-column collisions.** The target schema's normalized user-name and e-mail columns do not
  exist in the source's Identity generation. Two accounts differing only in case normalize to the same
  value. The migration must **detect collisions before writing and stop**, rather than silently
  dropping an account — a dropped account is indistinguishable from a user who forgot their password.
- **Password-hash validation.** The target framework's default hasher is expected to validate the
  older format and rehash on successful sign-in. **That is an expectation about the shipped hashes, and
  assumption A10 states it as one.** Its acceptance test is not a code review but a successful sign-in
  by a pre-existing account.
- **Columns with no source value.** Concurrency and security stamps, two-factor, lockout,
  access-failed-count and confirmation columns need **defined origins** rather than whatever the
  target's defaults happen to be.

| Field | Value |
| --- | --- |
| **Likelihood** | **Medium.** The store is small and the transition is a well-trodden one, but it is being performed against a schema qualified as evidence rather than proof, and against hashes whose format has not been tested |
| **Impact** | **High.** A failed Identity migration locks users out and, if the collision case is mishandled, loses accounts. High rather than Critical because the source store is retained and order history keys on the user name rather than on an identifier |
| **Mitigation** | Create the target tables **fresh and populate them** rather than altering the source in place, so the source survives as a rollback position and reconciliation can compare two live datasets. Preserve **user names exactly** — order ownership compares against them [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:71-73]. Detect collisions before writing. Reconcile account, role and assignment counts, asserting the administrator's membership specifically |
| **Contingency** | Source tables are retained until reconciliation passes. If hashes do not validate, the fallback is a **forced credential reset for affected accounts** with out-of-band notification — recoverable, but a user-visible event that must be approved rather than improvised, and the reason A10 is stated as an assumption instead of assumed |
| **Trigger** | **A pre-existing account cannot sign in after migration.** That single observable covers the hash, the normalized columns and the stamp defaults at once, which is why it is the acceptance test. Secondarily: any collision detected during the migration rehearsal |
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
| **Contingency** | If an unlabelled divergence is found after cutover, it is resolved as a policy decision with a recorded label — and, if it is user-affecting, communicated. The persisted policy is configuration rather than code, so correcting it does not require a deployment of the application |
| **Trigger** | Any policy row reaching W7's exit gate **without a label**. Secondarily, post-cutover: a rise in failed sign-ins or lockouts, or a registration rejected under a complexity rule the source accepted |
| **Owner** | **Security** |
| **Affected workstreams** | W7, and W13 as the point where user impact becomes visible |

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

| Field | Value |
| --- | --- |
| **Likelihood** | **Certain.** This is a designed consequence of decisions already recorded in 04 and 06, not a possibility. The open question is whether it is **approved and communicated**, not whether it happens |
| **Impact** | **Medium**, and genuinely unknown in the absence of traffic data — see the trigger. If a material share of real users are on an unsupported client, the impact is High and the decision changes |
| **Mitigation** | **Obtain the product owner's explicit approval in W1**, as one of the 14 delta sign-offs of [§7.2](#72-the-approved-delta-sign-offs). Support the decision with **actual client analytics** if any exist, so it is taken against evidence rather than assumption. State the matrix in the deployment documentation (W14), so an unsupported client generates a policy answer rather than a defect investigation |
| **Contingency** | If the product owner declines, the decision to be reversed is the **styling-framework major version** in [04 §9](04-dotnet8-migration-strategy.md) — not this matrix, which is downstream of it. That reversal carries its own consequence, which [09](09-security-assessment.md) owns: remaining on an out-of-support framework. There is no contingency that keeps both |
| **Trigger** | W1 completing without the product owner's recorded decision on this delta. Post-cutover: support contacts from unsupported clients, which the matrix in the deployment documentation is what converts into an answer |
| **Owner** | **The product owner**, alone. Explicitly not engineering, and explicitly not this document |
| **Affected workstreams** | W7 as the implementation, W14 as the statement of the matrix |

#### R8 — Case-sensitive path resolution on the target platform

**The risk is a class of failure that is invisible on the development platform and total on the target
one.** The primary hosting target [06 §2](06-azure-hosting-recommendations.md) resolves paths
case-sensitively; the current web host does not. Every asset and view path whose case does not match
its file exactly works today and 404s there.

**It is not theoretical, and one instance is already confirmed.** A stylesheet is registered as
`~/Content/site.css` [src/MVC5/MvcMusicStore/App_Start/BundleConfig.cs:28] while the tracked file is
`Content/Site.css` [src/MVC5/MvcMusicStore/Content/Site.css] — lowercase against capital.
[06 §3.4](06-azure-hosting-recommendations.md) therefore makes the audit a **precondition** for the
primary target rather than a caveat on it, and [12 §2.3](12-migration-blockers.md) classifies path
casing among the **silent** blockers.

**Independent corroboration that this repository is genuinely careless about case** — evidence that the
one known mismatch is a pattern rather than a typo: the repository's own ignore file specifies
`nuget.exe` in lowercase [.gitignore:28] while the tracked file is
`src/MVC4/MvcMusicStore/.nuget/NuGet.exe`. The consequence is that the rule does not match and the
binary **is not ignored at all** — `git check-ignore -v` exits 1 against it
([A.5](#a5-the-corroborating-case-mismatch-r8)), which is why it appears in
[08 §10.7](08-technical-debt-register.md)'s hygiene findings as a tracked file its author evidently
meant to exclude. Two independent case mismatches in one repository is a systemic property.

| Field | Value |
| --- | --- |
| **Likelihood** | **Medium** for an *undetected* mismatch surviving to production. That one mismatch exists is certain; the audit's whole purpose is finding the ones nobody has looked for |
| **Impact** | **High.** A missing stylesheet renders every page unstyled, and a mismatched view path is a 500. The failure is total for the affected path and absent from every case-insensitive test of it |
| **Mitigation** | W5 audits **all** of it — the **11** bundling helper sites, the **4** `@Url.Content` sites and the **27** static assets (inputs 8, 11 and 7; site and file counts) plus view paths — and, critically, makes the audit **repeatable as a pre-deployment check** rather than a one-time sweep, because a new mismatch can be introduced by any later change. W4's coverage asserts static assets resolve **case-sensitively**, which a case-insensitive check would pass wrongly |
| **Contingency** | A mismatch reaching production is corrected in the referencing code and redeployed; the repeatable check is then extended to cover the class that escaped it. Because W5's exit criterion is a case-sensitive serve with no static-asset or view 404, the correction is verifiable rather than hopeful |
| **Trigger** | Any 404 for a static asset or view on a case-sensitive serve — which is why the check must run on a case-sensitive filesystem in the pipeline and not on a developer machine |
| **Owner** | **Engineering** |
| **Affected workstreams** | W5 as the audit, W10 as the target platform, W13 as the point of exposure |

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

**The rollback symmetry, which is the part most likely to be forgotten.** A cutover that must be
reversed produces **the same two effects in the other direction**: users signed in against the new
application are signed out again, and anonymous carts built on it are lost. The rollback plan must
state both, or it understates its own cost and a reversal decision gets taken on a false premise.

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
from the repository to exclude private or substituted feeds.** This is also the residual behind
[R2](#r2--the-migration-sources-build-reproducibility): the recorded clean build resolved against a
source the *host* supplied, which 10 §3.1 is careful to state, and a different host may resolve
differently. Compounding it, no edition carries a lockfile
[08 §8.3](08-technical-debt-register.md), so transitive resolution is not reproducible either — exact
direct pins do not lock transitives.

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
operational finding — and observability in the target is **net-new capability** delivered by W7's
placement and W10's platform wiring [06 §9](06-azure-hosting-recommendations.md).

**The consequence for the cutover.** The legacy application is the rollback target, and it has no
instrumentation whatsoever. A blanket `catch` around the order write discards the exception and records
nothing [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:58]. So **if the new application is
rolled back, the system being rolled back to is unobservable** — a failed checkout there leaves no
trace, which is exactly the condition under which a cutover decision has to be made quickly and
correctly.

**This risk is why W10's observability wiring must be complete *before* W13 rather than alongside it.**
The graph already enforces the ordering; this entry states why it matters, so it is not treated as a
polish item deferrable past the window.

| Field | Value |
| --- | --- |
| **Likelihood** | **High.** The absence is verified fact today; the risk is that the window is entered before the target's observability is actually verified working, which is a common compression under schedule pressure |
| **Impact** | **High.** A cutover judged on user reports rather than telemetry is judged late, and the rollback target provides no evidence at all |
| **Mitigation** | W10 completes the observability wiring and the **health endpoint** before W13 begins, and W4's coverage includes the health check returning healthy with the database reachable and **unhealthy when it is not** — so the instrument is verified rather than assumed. Define, before the window, the specific signals that will be watched and the thresholds that would trigger rollback |
| **Contingency** | If telemetry is unavailable at the window, the window is **postponed** rather than entered blind. If it fails mid-window, the pre-agreed rollback threshold becomes a fixed decision point, and the runbook's verification steps are executed manually against the surfaces W4 already asserts |
| **Trigger** | W13's entry approached with the health endpoint or the log sink not verified end-to-end; or any window where the rollback signals have not been named in advance |
| **Owner** | **Operations and platform** |
| **Affected workstreams** | W10 as the mitigation, W13 as the exposure |

#### R13 — One database, one blast radius

[06](06-azure-hosting-recommendations.md) records the single-database decision **with its trade**, so
that a reader can reverse it knowingly. This entry carries the trade as a risk; it does not re-argue
the decision.

Both application contexts, the session cache and the data-protection key ring share one database
instance. The benefit is operational simplicity — one connection string, one migration target, one
backup and restore story, one instance to provision. **The cost is that they also share a blast radius,
a permission granularity and a scaling unit.** One principal's grants cover both contexts and the
session cache, so least privilege is coarser than it would be with separate instances; and an incident
affecting the instance affects catalog, credentials, sessions and key ring together — including the key
ring that authentication cookies depend on.

**It is a defensible trade for an application of this size with one deployable unit**, which is why 06
took it. It is carried here because it is the kind of decision whose cost is invisible until the day it
is not.

| Field | Value |
| --- | --- |
| **Likelihood** | **Low.** A managed database instance failing or being misconfigured is uncommon |
| **Impact** | **High.** Concurrent loss of catalog, credentials, session and key material. Recovery is a single restore, which is the benefit side of the same trade |
| **Mitigation** | Accept the trade knowingly, as an approval decision rather than a default. Separate **schema ownership** within the instance — distinct migration histories per context, per [04 §12](04-dotnet8-migration-strategy.md) and [06 §6.3](06-azure-hosting-recommendations.md) — so that the *logical* boundary survives even though the physical one does not. Keep the DDL principal separate from the runtime identity [06 §6.2](06-azure-hosting-recommendations.md), which is what stops a runtime compromise becoming a schema compromise |
| **Contingency** | Restore the instance; both contexts return together. If the granularity later proves insufficient — a compliance requirement on credential data, for instance — separation is a provisioning and connection-string change plus a migration re-point, not a redesign, because the two contexts are coupled only by convention and share no foreign key [01 §6.5](01-architecture-overview.md) |
| **Trigger** | A compliance or data-residency requirement that distinguishes credential data from catalog data; or a scaling need that one instance cannot serve. Either makes the trade wrong prospectively, which is the point at which it should be revisited |
| **Owner** | **Platform and operations**, as an approval decision |
| **Affected workstreams** | W10 as the provisioning, W13 as the point of no easy return |

#### R14 — A reference edition's retired data provider

Carried for completeness because [12 §2.3](12-migration-blockers.md) classifies it as a **compile-time**
blocker, and a reader scanning the blocker list will look for it here.

The oldest edition's catalog provider is a retired embedded database engine
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:55-59] for which **no supported provider
exists on the target framework** [12 §2.3](12-migration-blockers.md). Its data layer therefore cannot
be ported without re-targeting the provider outright.

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

### 9.4 The four risks that are approval decisions, not mitigations

Most entries above are managed by engineering action. **Four are not**: their correct resolution is a
decision by a named owner, and no amount of engineering diligence substitutes for it.
[03 §5](03-modernization-roadmap.md)'s W1 exit gate requires a recorded decision on each.

| Risk | The decision, and who takes it |
| --- | --- |
| [R1](#r1--the-target-framework-support-window) | Confirm the target framework knowing its support end date, or retarget before W6 — **engineering leadership** |
| [R7](#r7--the-narrowed-browser-matrix) | Accept the loss of a class of client, or reverse the dependency decision behind it — **the product owner** |
| [R9](#r9--cutover-re-authentication-and-anonymous-cart-loss) | Accept re-authentication and anonymous-cart loss as the cost of a single cutover — **product and operations** |
| [R13](#r13--one-database-one-blast-radius) | Accept a shared blast radius for operational simplicity — **platform and operations** |

**Presenting any of these four as a mitigated engineering risk would be the register's most misleading
possible error**, because it would imply the work can proceed correctly without a decision that only
somebody else can take.

---

## 10. Roll-up

### 10.1 The estimate in eight statements

1. **The total is 67.5 / 126.5 / 231 ideal engineer-days** across [03](03-modernization-roadmap.md)'s
   fifteen workstreams plus two non-code tasks. An IED is work content, not elapsed time.
2. **The port of existing code is about 35 percent of that.** The other 65 percent is net-new
   capability, data work gated on an unextracted schema, and governance — none of it predicted by any
   quantity in the existing codebase.
3. **The two largest rows are W7 (40 expected) and W4 (20 expected)**, and the second of them replaces
   nothing: there are zero tests today.
4. **Four low-confidence rows carry 40 expected IED**, and three of the four are waiting on one cheap
   workstream: W3, at 4 expected IED, is the highest-leverage item in the model.
5. **The critical path is 93 of the 126.5 expected IED.** About 26 percent can be absorbed by parallel
   capacity; the rest cannot, however many engineers are added.
6. **Every figure states its counting method.** The authentication rewrite is estimated against
   **382 non-blank lines** — the sizing metric — and the physical line count for that file appears
   nowhere in this document.
7. **The register carries fourteen entries**, each with all seven required fields. Four of them are
   **approval decisions rather than engineering risks**, and W1 cannot correctly exit without a recorded
   decision on each.
8. **This document contains no schedule.** Sequence plus effort lets a reader build one from their own
   capacity assumptions; a schedule computed here would embed assumptions that are not this
   assessment's to make.

### 10.2 What this document changes in the repository: nothing

It adds one file under `docs/modernization/` and modifies none. Every figure above describes work that
would follow an approval that has not been given. Per AAP §0.11.5, `git status --porcelain` shows only
new files under `docs/modernization/`.

### 10.3 Acceptance criteria for this deliverable

Self-check against AAP §0.11.2 row 07 and the deliverable's own authoring contract.

| Criterion | Where satisfied |
| --- | --- |
| An effort model **per workstream**, in **stated units** | [§4.2](#42-the-unit-defined) states the unit; [§5.1](#51-summary-table) is one row per workstream, using 03's names |
| **Low / expected / high bands** on every row; no single-point estimate | [§5.1](#51-summary-table) — all seventeen rows, and the W7 sub-decomposition |
| **Assumptions explicit** | [§4.3](#43-assumptions-stated-rather-than-implied) — A1–A10, each with the consequence of its being false |
| **Confidence with its reason** | [§4.4](#44-confidence-and-its-reason) — overall plus per-workstream, with the reason stated rather than asserted |
| Every effort figure **traces to a count** in this document, 01, 02 or 08 | [§4.1](#41-the-estimation-basis-every-input-with-its-method) — 19 numbered inputs, each with its source; every band in §5.2 cites the inputs it uses |
| Every figure **names its counting method** | [§3](#3-the-two-counting-methods) states the rule; every figure carries "sizing", "duplication", "file count", "site count", "pin count", "row count" or "entry count" |
| **382 non-blank used for the authentication rewrite**; physical counts not used for sizing | [§3.2](#32-the-one-substitution-that-would-corrupt-this-document) and [W7](#w7--the-aspnet-core-port). The physical figure for that file appears nowhere |
| Asset counts **state their inclusion rule** | Input 7 and [A.3](#a3-helper-view-and-site-counts) — the migration source's **four asset groups**, distinguished from the all-edition and browser-served counts |
| Estimated against **03's workstreams**; no alternative decomposition | [§5](#5-effort-by-workstream) opening paragraph; W1–W15 verbatim |
| **Net-new work sized honestly** | [§6.2](#62-the-finding-that-matters-most-in-this-document) quantifies it at ~65% of expected effort |
| Sequence **dependency-ordered**, parallelism noted, **no calendar** | [§8](#8-sequencing) — concurrency sets and critical path, explicitly a property of the graph |
| **R1 first**, framed as an **approval decision** | [R1](#r1--the-target-framework-support-window), and [§9.4](#94-the-four-risks-that-are-approval-decisions-not-mitigations) |
| All nine named risks present | R1–R9 in [§9.3](#93-the-entries), plus R10–R14 where the evidence warranted |
| **All seven fields on every entry** | [§9.3](#93-the-entries) — a field table per entry, fourteen of fourteen complete |
| The **visual review** and **delta sign-offs** carried as tasks with effort | [§7.1](#71-the-manual-visual-review), [§7.2](#72-the-approved-delta-sign-offs) |
| **Cross-references only**, no restatement | [§1.4](#14-what-this-document-does-not-own--the-routing-table) is the routing table; §10.4 is the index |

### 10.4 Cross-reference index

| Deliverable | What this document takes from it | What it takes back |
| --- | --- | --- |
| [01](01-architecture-overview.md) | Verified counts, code volume, view topology, asset groups, the two cart unit-of-work models (inputs 1, 5–7, 9, 10) | — |
| [02](02-dependency-inventory.md) | The as-is pins and the restore-source finding | [R11](#r11--the-effective-package-source-set-is-not-knowable-from-the-repository) |
| [03](03-modernization-roadmap.md) | **The workstream decomposition and every entry and exit gate** — the structure of §5 and §8 | The effort model and the risk register it routes here |
| [04](04-dotnet8-migration-strategy.md) | Target framework, SDK band, the 28 package outcomes, the future application map (input 13) | [R1](#r1--the-target-framework-support-window)'s support-window decision, which 04 §2.2 routes here |
| [05](05-aspnet-core-migration-approach.md) | Port design, the test suite's architecture and 27 coverage surfaces, the two data migrations, the cutover decision, the 14 approved deltas (inputs 14, 17) | [R6](#r6--security-hardening-versus-compatibility), [R7](#r7--the-narrowed-browser-matrix), [R9](#r9--cutover-re-authentication-and-anonymous-cart-loss); [§7.1](#71-the-manual-visual-review)'s review; every duration |
| [06](06-azure-hosting-recommendations.md) | Hosting target, provisioning order, DDL principal, key ring, observability approach, the browser matrix | [R7](#r7--the-narrowed-browser-matrix), [R8](#r8--case-sensitive-path-resolution-on-the-target-platform), [R12](#r12--no-observability-exists-during-the-cutover-itself), [R13](#r13--one-database-one-blast-radius) |
| [08](08-technical-debt-register.md) | **The two counting methods**, the three-part decomposition, the estimation-safe quantities and the forbidden ones (inputs 1–5, 18, 19) | [R3](#r3--the-absent-regression-baseline), [R4](#r4--domain-data-migration-rollback), [R10](#r10--scoping-by-analogy-across-editions) — the three §12.3 asks this document to carry |
| [09](09-security-assessment.md) | The security posture behind [R6](#r6--security-hardening-versus-compatibility) and the out-of-support consequence in [R7](#r7--the-narrowed-browser-matrix)'s contingency | — |
| [10](10-build-and-deployment-requirements.md) | **Per-edition build outcomes** — the recorded verification run | [R2](#r2--the-migration-sources-build-reproducibility), re-framed to its residual |
| [11](11-cloud-readiness-assessment.md) | Statefulness, transport and path casing as-is, behind [R8](#r8--case-sensitive-path-resolution-on-the-target-platform) and W15 | — |
| [12](12-migration-blockers.md) | **The 22 blockers in two groups** (input 12); the evidence-rather-than-proof qualification | [R4](#r4--domain-data-migration-rollback), [R5](#r5--identity-migration-rollback), [R14](#r14--a-reference-editions-retired-data-provider) |
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
```

**Manual construction sites — input 10, site count.** Ten, and the census matters because they are not
confined to controller field initializers: three are in startup composition and one is inside a method
body.

```bash
git grep -n -E 'new (MusicStoreEntities|ApplicationDbContext|UserManager<|RoleManager<|UserStore<)' \
  -- 'src/MVC5/MvcMusicStore/Controllers/*.cs' 'src/MVC5/MvcMusicStore/App_Start/*.cs' | wc -l
# -> 10
```

**Static assets — input 7, file count, with its inclusion rule stated.** This document uses the
**27** figure: the migration source's **four asset groups** (`Content`, `Scripts`, `Images`, `fonts`),
which is what the relocation work is sized against. It deliberately does **not** use the all-edition
four-group count of 171, nor the 173 browser-served count that additionally includes the two
web-root `favicon.ico` files — neither is the scope of W7's asset relocation.

```bash
git ls-files | grep -cE '^src/MVC5/MvcMusicStore/(Content|Scripts|Images|fonts)/'     # -> 27
```

### A.4 The visual review capture set (input 16)

**File count.** The 29 views resolve into 22 non-partial view files and 7 partials and layout; the
capture set of [§7.1](#71-the-manual-visual-review) is bounded by the former, at two viewports each.

```bash
git ls-files 'src/MVC5/MvcMusicStore/Views/*.cshtml' | grep -vE '/_[^/]*\.cshtml$' | wc -l   # -> 22
git ls-files 'src/MVC5/MvcMusicStore/Views/*.cshtml' | grep -cE '/_[^/]*\.cshtml$'           # ->  7
```

### A.5 The corroborating case mismatch (R8)

The primary mismatch is cited inline at
[src/MVC5/MvcMusicStore/App_Start/BundleConfig.cs:28] against
[src/MVC5/MvcMusicStore/Content/Site.css]. The corroboration — that this repository is systemically
careless about case rather than carrying one typo — is its own ignore file:

```bash
sed -n '28p' .gitignore                                        # -> nuget.exe        (lowercase)
git ls-files 'src/MVC4/MvcMusicStore/.nuget/*'                 # -> .../NuGet.exe    (capital N, G)
git check-ignore -v src/MVC4/MvcMusicStore/.nuget/NuGet.exe ; echo "exit=$?"
# -> no output, exit=1   the rule does not match, so the binary is not ignored at all
```

### A.6 The constraint this work was held to

```bash
git status --porcelain      # -> only new files under docs/modernization/
```

No repository file was modified, added or deleted in the course of authoring this document, and no
build output or restored package directory was left behind. The commands above are read-only: they
inspect the index and working tree and write nothing.

### A.7 Secondary cross-reference

Technical Specification §1.3, §3.3 and §6.4 were available as **secondary** references. Under the
citation policy of AAP §0.4.1 the repository governs, and every count in this appendix is established
from the repository rather than from the specification. Where the two disagree, the disagreement is
recorded by the deliverable that owns the fact — [02 §6](02-dependency-inventory.md) for the restore
source, [01 §9](01-architecture-overview.md) for per-capability edition coverage — and not adjudicated
here.
