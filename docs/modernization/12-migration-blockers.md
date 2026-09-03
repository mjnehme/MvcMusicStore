
# 12 — Migration Blockers

## 1. Purpose, scope and authoring contract

### 1.1 What this document is

This is the **enumeration of every construct in this repository that a migration to ASP.NET Core cannot
translate**. Each entry names the construct, gives its exact location, and states either the successor
construct that takes over its responsibility or the fact that no successor exists.

It sits at the hinge of the assessment. It **consumes** three assessments — [09 — Security
Assessment](09-security-assessment.md), [10 — Build and Deployment
Requirements](10-build-and-deployment-requirements.md) and [11 — Cloud Readiness
Assessment](11-cloud-readiness-assessment.md) — and it is the **single input to all three strategy
documents**: [04 — .NET 8 Migration Strategy](04-dotnet8-migration-strategy.md), [05 — ASP.NET Core
Migration Approach](05-aspnet-core-migration-approach.md) and [06 — Azure Hosting
Recommendations](06-azure-hosting-recommendations.md). They resolve what this document enumerates.

That relationship sets the standard this document is held to. **A construct that is not named here does
not get resolved**, because the strategies work from this list. The consequence is asymmetric: a
compile-time break that is missed stops a build and announces itself, while a silent behavioural
difference that is missed ships to production and is discovered by a customer. Section 2 is built around
that asymmetry, and section 4 is the part of this document that exists because of it.

### 1.2 What this document is not

It is **not** a migration plan, a sequence or an estimate. It does not say when anything is done, in what
order, by whom or at what cost — that is [03](03-modernization-roadmap.md) for sequencing and
[07](07-effort-risks-sequencing.md) for effort and risk. It does not design a replacement: where an entry
says what takes over a responsibility, that is a statement of the successor construct, not a design
decision, and every such statement points at the deliverable that owns the decision.

It is also **not** a list of defects. Several entries describe code that is correct, idiomatic and
working today; they appear here because the framework underneath them changes, not because anything is
wrong with them. Conversely, a genuine defect that ports cleanly is not a migration blocker and is not
listed here — [08 — Technical Debt Register](08-technical-debt-register.md) owns those.

### 1.3 The no-modification constraint

The user directed **"Do not make code changes initially"**, and the project's environment setup
instructions independently restate the same gate. **No tracked file was created, modified or deleted to
produce this assessment other than the thirteen deliverables themselves**, and **every construct named
below stays exactly where it is**.

**Restores and builds were performed during the assessment, and they wrote into this checkout, so the
attestation accounts for them rather than denying them.** They were **unqualified restore and build
operations**, run historically against the three editions, and they left **eight payload and output
trees** behind. Unqualified is the operative word and it is [10 — Build and Deployment
Requirements](10-build-and-deployment-requirements.md)'s own finding about them
([10 §1.4](10-build-and-deployment-requirements.md)): **no run behind those trees is build evidence**,
because none of them recorded what a build result has to carry to be one. So this paragraph accounts for
**residue** and for nothing else. The build evidence this assessment retains, and the limits on it, are
10's to state and this document's to cite ([§1.5](#15-what-this-document-does-not-own)); MVC 5's status is
unchanged by anything here and remains **blocked pending a Windows verification run**
([10 §1.2](10-build-and-deployment-requirements.md)). What is established about the residue is only that
every one of the eight trees was matched by an ignore rule already in `.gitignore`, that none was tracked,
and that none changes any recorded outcome:

- **Build output, six trees** — `src/MVC3/MvcMusicStore-Completed/MvcMusicStore/bin` and `…/obj`,
  `src/MVC4/MvcMusicStore/bin` and `…/obj`, `src/MVC5/MvcMusicStore/bin` and `…/obj` — matched by
  `[Bb]in/` [.gitignore:2] and `[Oo]bj/` [.gitignore:1].
- **Restored package payload, two trees** — `src/MVC4/packages` and `src/MVC5/packages` — matched by
  `Packages/` [.gitignore:33], and **not** by `packages/*` [.gitignore:15]. The distinction is the one
  [04 §A.6](04-dotnet8-migration-strategy.md) records, and it is load-bearing twice over. `packages/*`
  carries an interior separator, so git anchors it to the directory holding the `.gitignore` — the
  repository root — and it cannot reach a nested path such as `src/MVC4/packages` at all; `Packages/`
  has no interior separator and therefore matches a directory of that name at **any** depth. That rule
  matches these two **lowercase** directories only because this checkout is case-insensitive —
  `git config core.ignorecase` reports `true` — so **on a case-sensitive host no rule in `.gitignore`
  ignores either tree**, and the same restore run there would leave both payloads untracked and fully
  visible to bare `git status --porcelain`. Each tree is where its project's `..\packages\…` hint paths
  resolve to, the first at [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:66] and
  [src/MVC4/MvcMusicStore/MvcMusicStore.csproj:66], and neither is where a payload is committed: MVC 4
  ships its payload one directory lower, under `src/MVC4/MvcMusicStore/packages/`, which is the mismatch
  [F-12-14](#f-12-14--mvc-4s-committed-build-configuration) records, and MVC 5 ships no payload at any
  path.

Together the eight held **527 files and 114,310,394 bytes**, which is more payload than the entire tracked
checkout carries. **All eight were removed once the assessment had finished with them, and their absence
was verified at that point.** Those two figures were measured before removal and are not reproducible
afterwards by construction. What the same two commands report afterwards is the emptied state — but only
while nothing generates new ignored payload, because they see ignored content regardless of what produced
it, so in a checkout where a build or restore has run since they will legitimately be non-empty:

```bash
git ls-files --others --ignored --exclude-standard | wc -l          # before removal -> 527; 0 when cleared
git ls-files --others --ignored --exclude-standard -z \
  | xargs -0 -r stat -c %s | awk '{s+=$1} END {print s+0}'          # before -> 114310394; 0 when cleared
```

**Bare `git status --porcelain` could not have detected any of that, which is why the acceptance check is
four commands run together rather than one.** An ignored path is invisible to bare porcelain and absent
from a tracked-file diff by construction, and all three of the rules above — `[Oo]bj/` [.gitignore:1],
`[Bb]in/` [.gitignore:2] and `Packages/` [.gitignore:33] — are ignore rules, so porcelain and the diff
would each have reported a clean result with all eight trees still sitting in the working tree, and that is
what they did report for as long as the eight existed. The third rule is why the two `packages` trees were
among them, and it is the one that is host-dependent: `Packages/` covers those lowercase directories only
because `core.ignorecase` is `true` on this checkout. On a case-sensitive host it would not cover them,
`packages/*` [.gitignore:15] would not reach them either, and both trees would have been plainly visible
to bare porcelain as untracked payload. The conclusion for this checkout is unchanged — all eight were
ignored here, which is exactly why bare porcelain could not see them — but the ignore half of the
attestation is a property of the host as much as of `.gitignore`, so it is stated as a measurement rather
than assumed. `git check-ignore` names the matching rule per path, and `--no-index` is what makes the
answer the rule set rather than the index:

```bash
git check-ignore -v --no-index \
  src/MVC4/packages/x src/MVC5/packages/x \
  src/MVC5/MvcMusicStore/bin/x src/MVC5/MvcMusicStore/obj/x
# -> .gitignore:33:Packages/   src/MVC4/packages/x
#    .gitignore:33:Packages/   src/MVC5/packages/x
#    .gitignore:2:[Bb]in/      src/MVC5/MvcMusicStore/bin/x
#    .gitignore:1:[Oo]bj/      src/MVC5/MvcMusicStore/obj/x
git config core.ignorecase                              # -> true
```

Every result below was observed on the committed assessment checkout with no generated output present.
Commands 1 to 3 describe a working tree at the moment they run, and 2 and 3 see ignored content, so a
checkout in which any build or restore has run since will report those trees; command 4 is a property of
the committed range and is the durable claim.

```bash
# 1. Nothing uncommitted. Says nothing whatever about ignored paths.
git status --porcelain                                  # -> 0 lines
# 2. The same question with ignored paths included: the check that sees bin/, obj/ and packages/.
git status --porcelain --ignored                        # -> 0 lines in that cleared state
# 3. What `git clean -X` would remove, listed rather than removed: ignored payload, if any survives.
git clean -ndX                                          # -> no output in that cleared state
# 4. The tracked change against the pre-assessment baseline commit. The left side is the
#    immutable pre-assessment revision ea2552d6eda7c20e9477a512e5c615665618cf35, the last
#    commit before this assessment began; the right side is HEAD, the delivery commit the
#    reviewer has checked out, which is named as HEAD rather than as a hash because a
#    document cannot cite the commit that creates it. Every source path cited in this
#    document is byte-identical at the two revisions, which is what makes the citations and
#    this check answer the same question.
git diff --name-status ea2552d6eda7c20e9477a512e5c615665618cf35..HEAD
# -> exactly 13 rows, every one an A, every path under docs/modernization/:
#    A docs/modernization/01-architecture-overview.md
#    A docs/modernization/02-dependency-inventory.md
#    A docs/modernization/03-modernization-roadmap.md
#    A docs/modernization/04-dotnet8-migration-strategy.md
#    A docs/modernization/05-aspnet-core-migration-approach.md
#    A docs/modernization/06-azure-hosting-recommendations.md
#    A docs/modernization/07-effort-risks-sequencing.md
#    A docs/modernization/08-technical-debt-register.md
#    A docs/modernization/09-security-assessment.md
#    A docs/modernization/10-build-and-deployment-requirements.md
#    A docs/modernization/11-cloud-readiness-assessment.md
#    A docs/modernization/12-migration-blockers.md
#    A docs/modernization/README.md
# No M row, no D row, no R row, and nothing outside docs/modernization/.
```

**Only one end of command 4's range is a literal hash, and the asymmetry is deliberate rather than an
omission.** The baseline endpoint is pinned in full — `ea2552d6eda7c20e9477a512e5c615665618cf35`, the last
commit before this engagement — so the range is exactly this engagement and nothing else. The far end is
written `HEAD` rather than a second hash because **no document can contain the hash of the commit that adds
it**: that hash exists only once the commit does, which is after this file has reached the content the
commit records, so a literal in its place would necessarily name some other commit. A reader who wants both
ends pinned resolves it once on their own checkout — `git rev-parse HEAD` — and substitutes the result
wherever `HEAD` appears, here and in [section 8](#8-reproducibility-appendix). **The substitution changes
none of the four values the assertion expects:** 13 rows, 13 `A` rows, 0 rows that are not an `A`, and 0
paths outside `docs/modernization/`. Each is a property of the range rather than of the particular tip it
ends at, so all four hold at every tip at which the thirteen-file set is complete — including the tip that
lands the last correction to it.

Commands 1 and 4 answer *what is tracked*; commands 2 and 3 answer *what is ignored*, and only those two
could have found the eight trees. Neither pair substitutes for the other, and both are required: 1, 2 and 3
together establish that nothing at all remains in the working tree, tracked or ignored, and 4 establishes
that the committed change is exactly thirteen additions under `docs/modernization/` and no other row of any
kind.

This constraint bites hardest here, because this document reads like a work order. It is not one. It is a
list a reader approves *before* anyone touches a file. Two of the entries below are the sharpest test of
that, because each would take a **one-line** edit to fix and neither was fixed: the framework-version
mismatch at [F-12-18](#f-12-18--a-framework-version-mismatch-inside-mvc-5s-own-configuration) — one
attribute value, `<httpRuntime targetFramework="4.5"/>`
[src/MVC5/MvcMusicStore/Web.config:34], disagreeing with
`<compilation … targetFramework="4.8"/>` one line above it [src/MVC5/MvcMusicStore/Web.config:33] — and
the path casing at [F-12-17](#f-12-17--filesystem-path-casing), a two-character difference in one bundle
registration string in each of the two editions that has one.

**One construct a reader may come here looking for is deliberately not enumerated below.** The
state-changing `GET` on the cart-add action is a **security** finding rather than a portability one — it
has a direct successor in the target and so fails neither at compile time nor silently — so it is owned
by [09](09-security-assessment.md), and the verb change that resolves it is
[05](05-aspnet-core-migration-approach.md)'s. It carries no `F-12-nn` identifier, and this document does
not assign it one.

### 1.4 Authoring contract, and the absence of user rules

**No user-specified rules were provided for this project.** `review_rules` returns exactly *"No user
rules provided."*, verified directly during this work. There is accordingly no rule to name, summarize or
cite, and no file forced into scope by one. The absence is not a licence to lower the bar; enterprise
best practice applies instead, and this document holds itself to four explicit contracts:

1. **Every entry carries a file location.** Citations are inline as `[<path>:<locator>]` at the point the
   claim is made, never collected in a list at the end. Paths are repository-relative and resolve in the
   checkout. A citation with no locator does not satisfy this contract, and neither does a locator with no
   path: **there is no inheritance convention.** A citation never borrows its path from an earlier
   sentence, an earlier table row or the enclosing section, so a lone locator such as `:29` and a bare
   filename such as `Order.cs:8` are both disqualified forms, and neither appears as a citation anywhere
   below. The cost is repetition; the benefit is that every citation can be checked, and re-checked after
   an edit, without reading anything above it. **A tracked binary is cited the same way, by the one
   locator it can support:** a database file has no line to point at, so its locator is its byte size —
   `[<path>:<N> bytes]` — with the command that reads the size stated beside the claim, and a line number
   invented for it would be a fiction rather than a locator.
2. **Every entry states which edition or editions it holds in.** "The application" is three
   applications, and a blocker in one is not a blocker in the others. MVC 3's SQL Server Compact
   dependency is MVC 3's alone; the OWIN constructs are MVC 5's alone; `TryUpdateModel` is in all three.
3. **Repository-wide claims carry their reproducing command.** A count or an absence has no single line
   to point at, so the evidence is the command, printed beside the claim and collected in
   [section 8](#8-reproducibility-appendix). That is the stronger form of evidence, because a reader can
   re-run it.
4. **One fact, one owner.** Where a fact belongs to another deliverable, this document cites it and does
   not restate it. Section 1.5 lists the owners.

Markdown line length follows the corpus convention recorded in [`README.md` § 10 Markdown line-length convention](README.md#10-markdown-line-length-convention).

### 1.5 What this document does not own

| Decision or fact | Owner | What this document does instead |
| --- | --- | --- |
| **Target framework, SDK band, project-format conversion** | [04](04-dotnet8-migration-strategy.md) | Names no framework version and no package version. Entries state that a construct has no successor *in ASP.NET Core*, which is true of every candidate target. |
| **How each blocker is resolved** — pipeline, DI, configuration, EF Core, Identity, static assets, cutover | [05](05-aspnet-core-migration-approach.md) | Enumerates and locates. Where a successor construct is named, it is named factually; the design is 05's. Where a genuine choice existed, this document names the choice's owner and the outcome 05 has since settled (§7.3) rather than making the choice itself. |
| **Hosting target, deployment model, telemetry mechanism, data-protection key store** | [06](06-azure-hosting-recommendations.md) | Cross-references only. The Linux path-casing audit at F-12-17 is 06's requirement, not this document's recommendation. |
| **Per-edition build outcomes** | [10](10-build-and-deployment-requirements.md) | Cites 10 for every build result, including MVC 5's status — **blocked pending a Windows verification run** ([10 §5.4](10-build-and-deployment-requirements.md)) — and MVC 4's committed-configuration failure. The diagnosis is **not** restated. |
| **Security posture and severity of the same constructs** | [09](09-security-assessment.md) | Several constructs here are also security findings. This document classifies the *portability* consequence; 09 owns the security consequence and the severity. |
| **Effort, risk, sequencing** | [07](07-effort-risks-sequencing.md), [03](03-modernization-roadmap.md) | Provides the classification and the counts they estimate against. Assigns no duration and no order. |

Two facts are consumed rather than owned here and are cited where used: the cloud-hosting consequences of
the connection strings, session and path casing belong to [11](11-cloud-readiness-assessment.md), and the
duplication and hygiene quantities belong to [08](08-technical-debt-register.md).

### 1.6 How to read an entry

Every entry has the same five parts, and the third is the one that matters most:

- **Editions** — where the construct exists.
- **Evidence** — the file and locator, inline.
- **Failure mode** — *compile-time* or *silent runtime*, with the reason it is that one. This is not a
  label; it is the claim the entry has to earn.
- **Successor** — the construct that takes over, or an explicit *no successor; removed*.
- **Owner** — the deliverable that resolves it.

Identifiers are stable: `F-12-nn` for blockers, `P-12-nn` for the portability findings in
[section 6](#6-portability-findings-in-the-applications-favour) that make the list shorter than it looks.
Downstream deliverables cite these identifiers.

---

## 2. The two groups, and why the distinction is the whole point

### 2.1 The distinction

Every blocker below falls into exactly one of two groups, and they differ in **who discovers them**.

**Group one — no successor, or the API itself is gone. These fail at compile time.** The type, the
overload, the attribute or the configuration element does not exist in the target. The build breaks. A
compiler error names the file and the line, the developer fixes it, and the work is bounded by a list the
toolchain produces for free. These entries are numerous and some are laborious, but **almost none of them
is dangerous**, because almost none can reach production undetected.

**"Almost" is one entry, and it is named here rather than buried.**
[F-12-13](#f-12-13--windows-authentication-connection-strings-and-file-attached-databases) is a cluster:
its reading API fails the build, which is why it is grouped here, but its two other parts — a
configuration source the target does not read, and connection-string values a managed database cannot
honour — fail at configuration time and at runtime in the target environment only. A group-one entry is
therefore a guarantee about the *earliest* stage at which the blocker announces itself, not a guarantee
that every part of it does. That entry states its three stages individually; no other group-one entry is
mixed.

**Group two — the successor exists, and its default behaviour differs. These compile, deploy, and then
behave differently.** The code is valid in the target. It builds with zero warnings. It starts. Then a
navigation property is null where it used to be populated, a JSON field arrives camel-cased where the
JavaScript expects PascalCase, a stylesheet 404s on a case-sensitive filesystem, or a `Dispose` override
disposes an object the container still owns. **Nothing announces any of it.**

**Group two carries one entry that satisfies neither half of that description, and it is named here rather
than left to surprise a reader who counted on the label.**
[F-12-01](#f-12-01--sql-server-compact-40-as-the-catalogue-provider) has **no successor of any kind** —
SQL Server Compact 4.0 has no supported provider on the target framework — and it fails **loudly** rather
than silently, throwing at provider activation on the first data access. It is in group two because
membership is decided by the axis §2.2 states, **who discovers it**, and no build discovers this one:
`System.Data.SqlServerCe.4.0` is a `providerName` attribute in configuration
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:56-58] and the project file declares no
reference to the provider assembly at all
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:37-41], so nothing in the source
binds to it and nothing about it reaches the compiler. Filing it in group one would assert a compile error
that cannot occur. **So group two is nine entries: eight silent, and this one loud.** Both properties are
stated on the entry and in §2.3's index row.

**Group two has exactly one mixed entry too, and it is mixed in the opposite direction — which is why one
rule cannot place both.**
[F-12-19](#f-12-19--connection-string-resolution-by-convention) covers two `DbContext` classes that lose
the *same* convention at **opposite** stages: `MusicStoreEntities` relies on EF 6 matching a
`connectionStrings` entry to its **class name** and declares no constructor at all, so nothing about it
fails to compile — it compiles and stops meaning anything; `ApplicationDbContext` passes the name
explicitly as `: base("DefaultConnection")`, a call to a base overload the target does not have, so it
breaks the build and announces itself. Grouping that entry by its *earliest* stage — the rule that places
F-12-13 — would file its dangerous half under "the compiler will find it", which is exactly untrue.

**So the group label is stated for what it is: the stage a reader must *plan* for, with the entry itself
carrying every stage.** For [F-12-13](#f-12-13--windows-authentication-connection-strings-and-file-attached-databases)
that is compile time, because the reading API break is the part the port fixes in code while its
configuration and connection-value parts are closed by named deployment gates. For F-12-19 it is silent
runtime, because its compile-time half is a one-line constructor change the build hands you for free while
its silent half is the one that needs a named site or it is not found at all. **Both entries enumerate
their stages individually where they sit**, which is what actually delivers the naming discipline of §2.2,
and **each of the two constructs in F-12-19 is counted once, in that entry and nowhere else** — neither
appears in a group-one row. **Every construct in this register therefore belongs to exactly one entry, and
every entry to exactly one group**, and the totals in §2.3 are unaffected by either mixture.

### 2.2 Why the grouping is load-bearing

The two groups need opposite treatment, which is why collapsing them into one list of "things to fix"
loses the plot.

Group one is discovered by compiling. It needs no test, no reviewer and no checklist — it needs a build.
Group two is invisible to the compiler by construction, so it can only be caught by someone who **knew
in advance to look**, at a site someone **named in advance**. There is no build that finds it and, in
this repository, no test either: the codebase contains no test of any kind, repository-wide
([08 §7.3](08-technical-debt-register.md)), so nothing existing would detect a behavioural change.

That is the reason [section 4](#4-group-two--the-successor-exists-and-its-default-differs-silent-breakage)
enumerates each affected **site** rather than each affected *concept*. "Lazy loading changes" is not
actionable. Six specific dereferences at six specific lines, each with the view that renders them, is
actionable — and a seventh site that nobody wrote down becomes a silent regression that no test was
written to catch.

One asymmetry compounds this and is worth stating early, because it cuts the other way from what a
reader might expect: **the 29 Razor views in the migration source have never been compile-checked.**
`MvcBuildViews` is `false`, so no view is compiled at build time today
([10 §12.3](10-build-and-deployment-requirements.md), which owns the finding). The target compiles Razor
views as part of the build. Two consequences follow. Type errors inside views — like
[F-12-06](#f-12-06--systemwebmvchandleerrorinfo-as-a-view-model)'s removed model type — become
**build** errors in the target rather than the runtime errors they would be today, which moves them into
group one. And the views are simultaneously the largest surface in the migration source with **no**
build-time guarantee behind it today, which is why view-level sites are enumerated here individually
rather than assumed to port with their controllers.

### 2.3 Blocker index

Twenty-three blockers. **Fourteen** fail at compile time; **nine** are not found by compiling — eight of
those silently, and one, `F-12-01`, loudly at provider activation, per §2.1. **14 + 9 = 23**, which is the
number of rows below.

The rows are ordered by **group** rather than by identifier, and two of them therefore sit out of numeric
sequence. `F-12-23` is listed last among the compile-time rows: it belongs to group one and was added to it
after the identifiers had been assigned, and renumbering the register would have broken every citation
already pointing at an existing entry. `F-12-01` is listed **first among the runtime rows despite holding
the lowest identifier**, because its provider is named only in configuration and no build can find it —
§2.1 states the reasoning and the entry states the evidence.

**Two rows' failure modes are not a single stage, and the index says so on both rows rather than on one.**
They are the two §2.1 names, one per group, and they are mixed in opposite directions:

- **F-12-13** is a cluster of three parts that fail at three different moments — an API break the compiler
  catches, a configuration source the target does not read, and connection-string values that fail only
  against a managed service. It is counted **once**, in the fourteen, and grouped by the stage the port
  plans for: the API break it fixes in code.
- **F-12-19** covers two `DbContext` classes whose shared convention fails at **opposite** stages — the
  class-name convention silently, the explicit `: base("DefaultConnection")` call at compile time. It is
  counted **once**, in the nine, and grouped by its silent half, because that is the half that needs a
  named site. Both of its constructs are counted in that entry and in no group-one row.

**No other row in this index is mixed**, and the totals are unchanged by either mixture: fourteen plus nine
is twenty-three, which is the number of rows below. Each mixed entry carries its stages individually where
it sits, with their separate consequences. `F-12-01` is **not** a mixed row — it has exactly one stage,
runtime provider activation — it is the row whose single stage is neither compile-time nor silent, which
§2.1 states and its index row below repeats.

| ID | Blocker | Editions | Failure mode |
| --- | --- | --- | --- |
| [F-12-02](#f-12-02--systemweboptimization-bundling) | `System.Web.Optimization` bundling, with `{version}` and glob tokens | MVC 5, MVC 4 | Compile-time |
| [F-12-03](#f-12-03--the-katana-iappbuilder-abstraction-and-the-owin-startup-attribute) | The Katana `IAppBuilder` abstraction and the OWIN startup attribute | MVC 5 | Compile-time |
| [F-12-04](#f-12-04--systemwebhttpapplication-and-the-globalasax-markup-declaration) | `System.Web.HttpApplication` and the `Global.asax` markup declaration | all three | Compile-time |
| [F-12-05](#f-12-05--handleerrorattribute--the-entire-error-handling-policy) | `HandleErrorAttribute` — the entire error-handling policy | all three | Compile-time |
| [F-12-06](#f-12-06--systemwebmvchandleerrorinfo-as-a-view-model) | `System.Web.Mvc.HandleErrorInfo` as a view model | MVC 5 | Compile-time |
| [F-12-07](#f-12-07--the-synchronous-tryupdatemodel-and-the-class-level-bind-attribute) | The synchronous `TryUpdateModel` and the class-level `[Bind]` | all three | Compile-time |
| [F-12-08](#f-12-08--three-httpnotfound-calls) | Three `HttpNotFound()` calls | MVC 5, MVC 4 | Compile-time |
| [F-12-09](#f-12-09--challengeresult-and-its-executeresult-override) | `ChallengeResult` and its `ExecuteResult` override | MVC 5 | Compile-time |
| [F-12-10](#f-12-10--the-blockviewhandler-mapping-and-the-razor-host-section-group) | The `BlockViewHandler` mapping and the Razor host section group | all three | Compile-time |
| [F-12-11](#f-12-11--the-axd-ignore-route) | The `.axd` ignore route | all three | Compile-time |
| [F-12-12](#f-12-12--assembly-metadata-in-a-source-file) | Assembly metadata in a source file | all three | Compile-time |
| [F-12-13](#f-12-13--windows-authentication-connection-strings-and-file-attached-databases) | Windows-authentication connection strings and file-attached databases | MVC 5, MVC 4 | Compile-time **for the reading API only** — its configuration and runtime/cloud parts fail later; see the entry's three stages |
| [F-12-14](#f-12-14--mvc-4s-committed-build-configuration) | MVC 4's committed build configuration | MVC 4 | Compile-time |
| [F-12-23](#f-12-23--mvc-child-actions) | MVC child actions — `[ChildActionOnly]`, `@Html.Action` and `Html.RenderAction` | all three | Compile-time |
| [F-12-01](#f-12-01--sql-server-compact-40-as-the-catalogue-provider) | SQL Server Compact 4.0 as the catalogue provider | MVC 3 | **Runtime — provider activation.** Named only as a `providerName` in configuration with no assembly reference anywhere, so no build finds it; it throws on first data access. The one group-two row with **no successor of any kind**, and the one that fails **loudly** rather than silently |
| [F-12-15](#f-12-15--lazy-loading-is-on-by-default-in-ef-6-and-off-in-ef-core) | Lazy loading is on by default in EF 6 and off in EF Core | all three | **Silent runtime** |
| [F-12-16](#f-12-16--json-property-naming-flips-to-camelcase) | JSON property naming flips to camelCase | all three | **Silent runtime** |
| [F-12-17](#f-12-17--filesystem-path-casing) | Filesystem path casing | MVC 5, MVC 4 | **Silent runtime** |
| [F-12-18](#f-12-18--a-framework-version-mismatch-inside-mvc-5s-own-configuration) | A framework-version mismatch inside MVC 5's own configuration | MVC 5 | **Silent runtime** |
| [F-12-19](#f-12-19--connection-string-resolution-by-convention) | Connection-string resolution by convention, two conventions | all three | **Silent runtime for the class-name convention** — the explicit `: base("DefaultConnection")` call in the same entry breaks the build; see the entry's two stages |
| [F-12-20](#f-12-20--dispose-overrides-on-objects-a-container-will-own) | `Dispose` overrides on objects a container will own | all three | **Silent runtime** |
| [F-12-21](#f-12-21--the-identity-schema-is-not-knowable-from-the-repository) | The Identity schema is not knowable from the repository | MVC 5 | **Silent runtime** |
| [F-12-22](#f-12-22--no-usable-schema-baseline-exists) | No usable schema baseline exists | MVC 5, MVC 4 | **Silent runtime** |

---

## 3. Group one — no successor, or the API itself is gone (compile-time)

**Fourteen entries** — F-12-02 through F-12-14, plus F-12-23 — and every one of them is a construct the
source binds to, so a build names it. The register's lowest identifier is deliberately **not** here:
[F-12-01](#f-12-01--sql-server-compact-40-as-the-catalogue-provider) is named only as a `providerName` in
configuration, with no assembly reference for a build to fail on, so it sits in
[section 4](#4-group-two--the-successor-exists-and-its-default-differs-silent-breakage) under the
placement rule of §2.1.

### F-12-02 — `System.Web.Optimization` bundling

**Editions:** MVC 5 and MVC 4 (MVC 3 has no `App_Start` folder and no bundling implementation at all —
see [01 §3.6](01-architecture-overview.md)).

Five `bundles.Add` registrations sit in one file in the migration source
[src/MVC5/MvcMusicStore/App_Start/BundleConfig.cs:11-27], and they use two token forms that are the
actual obstacle rather than the bundling itself:

- a **`{version}` token** — `"~/Scripts/jquery-{version}.js"`
  [src/MVC5/MvcMusicStore/App_Start/BundleConfig.cs:12], which resolves a version number out of the
  filename at runtime;
- **glob tokens** — `"~/Scripts/jquery.validate*"`
  [src/MVC5/MvcMusicStore/App_Start/BundleConfig.cs:15] and `"~/Scripts/modernizr-*"`
  [src/MVC5/MvcMusicStore/App_Start/BundleConfig.cs:20].

**Failure mode: compile-time.** `System.Web.Optimization` is a package with no successor package;
`ScriptBundle`, `StyleBundle`, `BundleCollection` and `BundleTable` do not exist in the target. The
registration call in `Application_Start`
[src/MVC5/MvcMusicStore/Global.asax.cs:18] goes with it, and the namespace registered for views
[src/MVC5/MvcMusicStore/Views/Web.config:18] becomes meaningless.

**The view surface is larger than the registration surface.** Eleven call sites depend on the framework
entirely — **10** `@Scripts.Render` and **1** `@Styles.Render`, the latter in the shared layout
[src/MVC5/MvcMusicStore/Views/Shared/_Layout.cshtml:7], with three of the ten in that same file
[src/MVC5/MvcMusicStore/Views/Shared/_Layout.cshtml:8] and
[src/MVC5/MvcMusicStore/Views/Shared/_Layout.cshtml:41-42], and the remaining seven spread across the
Account, Checkout and StoreManager views. Verify with
`git grep -n '@Scripts.Render' -- 'src/MVC5/*.cshtml' | wc -l` → `10` and the same for
`@Styles.Render` → `1`. Every one of the eleven is rewritten, because the helpers do not exist and the
five bundle virtual paths they reference cease to exist with them.

**Successor: no successor; removed.** No replacement supports the `{version}` or glob token forms, so
this is a removal with a redesign of static-asset delivery behind it, not a port.

**Owner:** [05](05-aspnet-core-migration-approach.md) for the static-asset strategy and all eleven call
sites.

### F-12-03 — The Katana `IAppBuilder` abstraction and the OWIN startup attribute

**Editions:** MVC 5 only.

Three files take `IAppBuilder` as a parameter, and between them they are the whole of MVC 5's
authentication and application startup: `Configuration(IAppBuilder app)`
[src/MVC5/MvcMusicStore/Startup.cs:9], `ConfigureApp(IAppBuilder app)`
[src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:14] and `ConfigureAuth(IAppBuilder app)`
[src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:11].

The entry point itself is a second construct with the same problem: it is declared by **assembly
attribute**, `[assembly: OwinStartupAttribute(typeof(MvcMusicStore.Startup))]`
[src/MVC5/MvcMusicStore/Startup.cs:4]. The target has no attribute-based host discovery — composition is
explicit in the program entry point.

**Failure mode: compile-time.** `IAppBuilder` is a type in the `Owin` package with **no successor type**.
Nothing in the target has that shape, so the three method signatures cannot be preserved even in
principle. The extension methods called on it go the same way: `app.UseCookieAuthentication(...)`
[src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:14-18] and
`app.UseExternalSignInCookie(...)` [src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:20] are Katana
extensions, not middleware registrations of the kind the target uses.

**Successor: the responsibilities survive, the abstraction does not.** Cookie authentication is a
first-class framework feature in the target, so the *capability* at
[src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:14-18] carries over while the
`Microsoft.Owin.*` and `Owin` packages that provide it disappear entirely
([02 §3.1](02-dependency-inventory.md) inventories all ten of them).

**Owner:** [05](05-aspnet-core-migration-approach.md). The authentication policy that becomes explicit in
the process — cookie lifetime, `SameSite`, password and lockout rules — is 05's to state and
[09 §3.3](09-security-assessment.md)'s to compare against, because those values are inherited framework
defaults today rather than anything the repository declares.

### F-12-04 — `System.Web.HttpApplication` and the `Global.asax` markup declaration

**Editions:** all three.

The application class derives from it: `public class MvcApplication : System.Web.HttpApplication`
[src/MVC5/MvcMusicStore/Global.asax.cs:11]. `Application_Start` runs **five** registrations
[src/MVC5/MvcMusicStore/Global.asax.cs:13-21], and they do not share one fate — which is why this entry
enumerates them rather than treating the file as a single unit:

| Registration | Location | Disposition |
| --- | --- | --- |
| `AreaRegistration.RegisterAllAreas()` | [src/MVC5/MvcMusicStore/Global.asax.cs:15] | Dead scaffolding — there is no `Areas` folder ([08 §9.1](08-technical-debt-register.md)) |
| `FilterConfig.RegisterGlobalFilters(GlobalFilters.Filters)` | [src/MVC5/MvcMusicStore/Global.asax.cs:16] | Its one filter has no successor — [F-12-05](#f-12-05--handleerrorattribute--the-entire-error-handling-policy) |
| `RouteConfig.RegisterRoutes(RouteTable.Routes)` | [src/MVC5/MvcMusicStore/Global.asax.cs:17] | Has a direct successor form — [F-12-11](#f-12-11--the-axd-ignore-route) |
| `BundleConfig.RegisterBundles(BundleTable.Bundles)` | [src/MVC5/MvcMusicStore/Global.asax.cs:18] | No successor — [F-12-02](#f-12-02--systemweboptimization-bundling) |
| `System.Data.Entity.Database.SetInitializer(new SampleData())` | [src/MVC5/MvcMusicStore/Global.asax.cs:20] | Replaced by explicit schema management; note it is registered a **second** time at [src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:16], which [08 §5.1](08-technical-debt-register.md) owns |

**A second tracked artifact goes with it.** `Global.asax` — the markup file, distinct from the code-behind
— is what the pipeline actually reads to find the application class:
`<%@ Application Codebehind="Global.asax.cs" Inherits="MvcMusicStore.MvcApplication" Language="C#" %>`
[src/MVC5/MvcMusicStore/Global.asax:1]. All three editions ship one
(`git ls-files '*Global.asax'` → three paths), and the target has **no application-declaration file** of
any kind.

**Failure mode: compile-time.** `System.Web.HttpApplication` does not exist in the target, so the base
class cannot resolve. The `Global.asax` markup is not compiled at all in the target — there is no handler
that would read it.

**Successor: the program entry point absorbs the surviving responsibilities**, individually, per the
table above. The base type and the markup file have no counterpart.

**Owner:** [05](05-aspnet-core-migration-approach.md), which dispositions each responsibility.

### F-12-05 — `HandleErrorAttribute` — the entire error-handling policy

**Editions:** all three. The registered line is identical in each; the **registration site** is not, and
the difference matters because it changes which file the responsibility moves out of.

`filters.Add(new HandleErrorAttribute());`
[src/MVC5/MvcMusicStore/App_Start/FilterConfig.cs:10], the same single line at
[src/MVC4/MvcMusicStore/App_Start/FilterConfig.cs:10] — and the same single line again in MVC 3, where
there is **no `FilterConfig` class and no `App_Start` folder at all**: the registration method is declared
on the application class itself, `public static void RegisterGlobalFilters(GlobalFilterCollection filters)`
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Global.asax.cs:15], adding the attribute at
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Global.asax.cs:17] and called from `Application_Start` at
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Global.asax.cs:38]. So in MVC 5 and MVC 4 the filter
leaves a dedicated startup file, and in MVC 3 it leaves the same file that
[F-12-04](#f-12-04--systemwebhttpapplication-and-the-globalasax-markup-declaration) already deletes.
[01 §3.6](01-architecture-overview.md) owns the per-edition startup-composition description.

**State the consequence sharply, because the entry is easy to under-read: this attribute is the
application's entire error-handling policy, and it is a type that does not exist in the target.** It is
the **only** global filter registered in any edition — there is no second filter, no exception filter, no
custom error page mapping and no logging behind it. The nearest thing to a second layer is a bare `catch`
with no exception variable wrapping the whole checkout transaction
[src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:58], which discards the exception and
redisplays the view; [08 §5.7](08-technical-debt-register.md) owns that as debt. So the port does not
migrate an error-handling policy — it **writes the first one**, and it does so with no observability to
build on, since the repository contains no logger of any kind
([11 §3.8.3](11-cloud-readiness-assessment.md)).

**One correction, because it changes what the successor has to do.** `HandleErrorAttribute` only engages
when custom errors are enabled, and **no edition enables them**
([09 §6.10](09-security-assessment.md) owns this).

The count needs its **unit** stated, because four different numbers are each correct and mean different
things, and calling any of them "opening tags" mislabels three of the four. The taxonomy is
[09 §6.10](09-security-assessment.md)'s, adopted here unchanged:

| Unit counted | Value | Per file |
| --- | --- | --- |
| Occurrences of the **word** `customErrors` in the six XDT transform files | **24** | 4 |
| Matches of the **literal prefix** `<customErrors` in the same six files | **12** | 2 |
| **Element opening tags** of the commented example | **6** | 1 |
| **Live** `<customErrors>` elements, in any edition | **0** | 0 |

The per-file arithmetic explains why the four figures differ, and
[src/MVC5/MvcMusicStore/Web.Release.config:19-29] is the representative block for all six. Each file's
template comment contains exactly four occurrences of the word: a prose mention written as
`<customErrors>` [src/MVC5/MvcMusicStore/Web.Release.config:21], the bare words "customErrors section"
[src/MVC5/MvcMusicStore/Web.Release.config:22], the example element's opening tag
`<customErrors defaultRedirect=` [src/MVC5/MvcMusicStore/Web.Release.config:25] and its closing tag
`</customErrors>` [src/MVC5/MvcMusicStore/Web.Release.config:28]. Of those four, two match the prefix
`<customErrors` — the prose mention and the example opening — because the closing tag begins
`</customErrors` and the bare words carry no angle bracket at all. Only one is an element opening tag.
Six files × 1 = the six the third row reports.

**The number that matters is the fourth: zero are live.** Every occurrence sits between an XML comment
open at [src/MVC5/MvcMusicStore/Web.Release.config:19] and its close at
[src/MVC5/MvcMusicStore/Web.Release.config:29], so nothing in the range is applied by a transform or read
by the runtime, and the six real configuration files contain the string not once. Each figure with the
command that produces it:

```bash
git ls-files -- '*Web.Debug.config' '*Web.Release.config' | grep -v '/packages/' \
  | xargs grep -o 'customErrors'  | wc -l              # -> 24  (the WORD, four per file)
git ls-files -- '*Web.Debug.config' '*Web.Release.config' | grep -v '/packages/' \
  | xargs grep -o '<customErrors' | wc -l              # -> 12  (literal prefix, two per file)
git ls-files -- '*Web.Debug.config' '*Web.Release.config' | grep -v '/packages/' \
  | xargs grep -o '<customErrors[[:space:]]' | wc -l   # -> 6   (example OPENING TAGS, one per file)
git ls-files -- '*.config' | grep -v '/packages/' \
  | grep -viE 'Web\.(Debug|Release)\.config|packages\.config|NuGet\.Config' \
  | xargs grep -c 'customErrors'                       # -> 0 for each of the six live files
```

The attribute is therefore closer to inert than active in the shipped configuration, which means the
target's behaviour is being *chosen* rather than *preserved*, and the difference must be deliberate
rather than inherited.

**Failure mode: compile-time.** The type is absent from the target; `GlobalFilterCollection` is absent
with it.

**Successor: exception-handling middleware**, which is a pipeline registration rather than a filter, plus
whatever policy 05 decides for status-code responses and for what may be disclosed.

**Owner:** [05](05-aspnet-core-migration-approach.md).

### F-12-06 — `System.Web.Mvc.HandleErrorInfo` as a view model

**Editions:** MVC 5.

The shared error view declares `@model System.Web.Mvc.HandleErrorInfo`
[src/MVC5/MvcMusicStore/Views/Shared/Error.cshtml:1]. The rest of the file is two static headings
[src/MVC5/MvcMusicStore/Views/Shared/Error.cshtml:7-8] — it renders **no property of that model at all**.

**Failure mode: compile-time**, and this is the entry where [§2.2](#22-why-the-grouping-is-load-bearing)'s
view-compilation asymmetry has a concrete effect. `HandleErrorInfo` does not exist in the target, so the
`@model` directive cannot bind. Today that error is invisible, because `MvcBuildViews` is `false` and no
view is compiled at build time ([10 §12.3](10-build-and-deployment-requirements.md)). In the target,
Razor views are compiled as part of the build, so the same directive becomes a **build** error. It is a
compile-time break in the target even though it is a runtime break today.

**Successor: the port defines its own model.** The exception-handling middleware that replaces
[F-12-05](#f-12-05--handleerrorattribute--the-entire-error-handling-policy) supplies **no Razor
equivalent** of `HandleErrorInfo`, so there is nothing to substitute into the directive — a model type
has to be written. The view's own generic user-facing text needs no change.

**Do not attribute this to MVC 4's exception-disclosure finding.** They are different editions, different
files and different causes, and conflating them produces a wrong remediation. MVC 4's disclosure is in
its **controller** — `ModelState.AddModelError("", e)` passing the exception object rather than a string
[src/MVC4/MvcMusicStore/Controllers/AccountController.cs:211-213] — and
[09 §4.9](09-security-assessment.md) owns it. MVC 5's error view discloses nothing: naming a type is not
rendering it. This view is rewritten for exactly one reason, which is that its model type is gone.

**Owner:** [05](05-aspnet-core-migration-approach.md), which owns the error model, the route the
middleware forwards to, the status-code behaviour, what is logged and what may be disclosed.

### F-12-07 — The synchronous `TryUpdateModel` and the class-level `[Bind]` attribute

**Editions:** all three — the call is identical in each.

`TryUpdateModel(order)` [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:29], applied to an
`Order` instance constructed one line earlier [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:28], inside the checkout POST action whose parameter is a
raw `FormCollection` [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:26].

What makes the call safe today is a **separate** construct: a class-level attribute on the entity,
`[Bind(Include = "FirstName,LastName,Address,City,State,PostalCode,Country,Phone,Email")]`
[src/MVC5/MvcMusicStore/Models/Order.cs:8]. That include list is the entire over-posting control at
checkout — [09 §6.4](09-security-assessment.md) owns that consequence as finding **F-09-28** — and both
halves of the mechanism have to move together. **MVC 3's attribute is the opposite kind**, an exclude list
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Models/Order.cs:8], which is
[09 §5.7](09-security-assessment.md)'s finding **F-09-22**. The portability classification below is
unaffected by which kind it is — both forms are the same class-level attribute reached by the same
synchronous call — and [§9.1](#91-the-reverse-direction--the-two-09-rows-this-document-discharges) records
how each of those two rows is discharged here.

**Failure mode: compile-time — and the break is the method, not the attribute.** The two halves fail for
different reasons, and getting them the wrong way round misdirects the fix:

- **The synchronous `TryUpdateModel` is the genuine removal.** The target exposes **only** the
  asynchronous form, `TryUpdateModelAsync`; there is no synchronous overload, so the call at
  [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:29] does
  not compile and no namespace substitution reaches it. The signature changes, and with it the action's
  own signature, because the caller must become asynchronous.
- **`[Bind]` is *not* removed. It still exists in ASP.NET Core**, as a model-binding attribute applicable
  to a type or a parameter, so the classification "no successor" would be wrong. Two things are
  nevertheless true. The *source's* form does not carry across as written: the include list here is a
  single comma-delimited string, `[Bind(Include = "FirstName,…,Email")]` [src/MVC5/MvcMusicStore/Models/Order.cs:8] typed against
  `System.Web.Mvc`, whereas the successor attribute takes an include list as a string array — so the
  declaration has to be rewritten even to keep the attribute. And the attribute is **deliberately not
  kept** — for three reasons, stated precisely, because the tempting version of this argument gets the
  semantics backwards. An `Include` list is an **allow-list**: it binds the properties it names and
  **excludes** everything else. It does not widen over time.
  - **It puts the HTTP boundary inside the persistence type.** The binding policy for one POST action
    lives as an attribute on the entity the whole application persists, so `Order` carries a
    `using System.Web.Mvc` directive [src/MVC5/MvcMusicStore/Models/Order.cs:4] to express a rule about one form. Every other consumer
    of `Order` inherits a declaration that has nothing to do with it, and the entity — not a
    request-shaped type — becomes the binding target.
  - **Over-post safety depends on a list that is invisible from the action it protects.** Nothing at
    [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:29] states which properties `TryUpdateModel` will populate; the answer is nine
    names in a string on another file. A reviewer reading the action cannot see the control, and a change
    to the entity's attribute silently changes what the action accepts.
  - **An allow-list is a policy that goes *stale*, which is the real risk and the opposite of a widening
    hole.** Because it excludes what it does not name, a property added to `Order` later is silently
    **not** bound: the field can be added to the checkout form, arrive in the request, and be dropped
    with no error, no warning and no failed validation — the value simply never lands on the entity. The
    defect is a quiet omission rather than a quiet exposure, and it is harder to notice for exactly that
    reason. An explicit input model at the boundary makes the accepted surface the type's own definition,
    so adding a field is a visible change to a request-shaped class rather than an edit to a string on a
    persisted entity.

  An explicit input model is therefore the replacement for those three reasons, and not because the
  attribute vanished — it did not.

`Order.cs` carries `using System.Web.Mvc`
[src/MVC5/MvcMusicStore/Models/Order.cs:4] purely to reference that attribute — a model-layer file
with an MVC dependency, and the directive goes when the attribute does, which is why the port of this file
is not confined to `Controllers/`.

**Successor: an explicit input model — and it must carry TEN properties, not nine.** This is the detail
most likely to be lost, because nine is the number written down in the repository and ten is the number
the action actually reads:

- **Nine** come from the `[Bind]` include list [src/MVC5/MvcMusicStore/Models/Order.cs:8].
- **The tenth is `PromoCode`**, which the action reads **separately, straight out of the raw form** —
  `values["PromoCode"]` [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:33], compared
  case-insensitively against `const string PromoCode = "FREE"` [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:12], with the form collection arriving
  as the action parameter [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:26]. It is not on the `[Bind]` list because it is not bound at all.
- **`PromoCode` belongs on the input model and *not* on the `Order` entity.** `Order` has no such
  property. Its whole surface is fourteen properties: the nine the bind list names, four suppressed with
  `[ScaffoldColumn(false)]` — `OrderId` [src/MVC5/MvcMusicStore/Models/Order.cs:11-12], `OrderDate` [src/MVC5/MvcMusicStore/Models/Order.cs:14-15], `Username` [src/MVC5/MvcMusicStore/Models/Order.cs:17-18] and
  `Total` [src/MVC5/MvcMusicStore/Models/Order.cs:63-64] — and the `OrderDetails` navigation collection [src/MVC5/MvcMusicStore/Models/Order.cs:66]. Adding a tenth persisted property
  would put a transient form value on the entity.

A plan that carries nine properties compiles, binds and persists correctly, and **silently loses
promo-code handling** — which in this application means the order-completion branch
[src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:38-55] is
unreachable and every checkout returns to the form at [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:36]. The nine-versus-ten fact is recorded here
because it is a property of the *current* code that no automated tool infers;
[09 §6.4](09-security-assessment.md) states the same ten and defers the model to 05.

**Owner:** [05](05-aspnet-core-migration-approach.md), which owns the input model and its tests — valid,
invalid and missing promo-code values.

### F-12-08 — Three `HttpNotFound()` calls

**Editions:** MVC 5 and MVC 4 — three calls each, at the same three line numbers. MVC 3 has none.

Three calls, all in the administration controller, all in the same shape — a `Find` that may return
`null`, a null check, and a not-found result:
[src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:35] (Details),
[src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:76] (Edit) and
[src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:108] (Delete). MVC 4 carries the identical
three at [src/MVC4/MvcMusicStore/Controllers/StoreManagerController.cs:35],
[src/MVC4/MvcMusicStore/Controllers/StoreManagerController.cs:76] and
[src/MVC4/MvcMusicStore/Controllers/StoreManagerController.cs:108] — unsurprising, since the two files are
byte-identical ([08 §3.1](08-technical-debt-register.md) owns that measurement). **MVC 3 does not use the
helper at all**: its three read actions pass the possibly-null `Find` result straight to `View(album)`
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/StoreManagerController.cs:31-32],
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/StoreManagerController.cs:68-71] and
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/StoreManagerController.cs:96-97], so there is
no `HttpNotFound()` call to port and this entry holds in two editions rather than three. Three commands,
one per edition, make that a census rather than a sample:

```bash
git grep -n 'HttpNotFound()' -- 'src/MVC5/*.cs' | wc -l     # -> 3
git grep -n 'HttpNotFound()' -- 'src/MVC4/*.cs' | wc -l     # -> 3
git grep -n 'HttpNotFound'   -- 'src/MVC3/*.cs' | wc -l     # -> 0  (not even the type name)
```

**Failure mode: compile-time.** `HttpNotFound()` is a `System.Web.Mvc.Controller` helper returning
`HttpNotFoundResult`; **neither the method nor the return type exists** on the target's controller base
class. The call does not compile.

**Successor: `NotFound()`**, returning the target's `NotFoundResult`. This is the closest thing in the
document to a mechanical rename, and it is listed because a census that omits the easy items is not a
census — 05 needs the count to be right, not interesting.

**One related site is deliberately *not* in this entry.** The fourth `Find` in the same controller,
`DeleteConfirmed`, has **no** null check at all: it passes the result straight to `Albums.Remove`
[src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:119-120]. That compiles in the target
exactly as it compiles today and throws exactly as it throws today, so it is not a migration blocker;
[08 §5.4](08-technical-debt-register.md) owns it as debt. Three
of four call sites guarded, one not, is a distribution worth not smoothing over.

**Owner:** [05](05-aspnet-core-migration-approach.md).

### F-12-09 — `ChallengeResult` and its `ExecuteResult` override

**Editions:** MVC 5.

`AccountController` declares a private nested action result to issue an OWIN authentication challenge:
`private class ChallengeResult : HttpUnauthorizedResult`
[src/MVC5/MvcMusicStore/Controllers/AccountController.cs:394], which overrides
`public override void ExecuteResult(ControllerContext context)`
[src/MVC5/MvcMusicStore/Controllers/AccountController.cs:411] and inside it calls
`context.HttpContext.GetOwinContext().Authentication.Challenge(properties, LoginProvider)`
[src/MVC5/MvcMusicStore/Controllers/AccountController.cs:418].

**Failure mode: compile-time, and every layer of it fails independently.** There are four separate
breaks in one twenty-seven-line class, which is why it cannot be ported by adjusting a namespace:

1. **The base type is gone.** `HttpUnauthorizedResult` does not exist in the target.
2. **The override signature is gone.** Action results in the target implement
   `ExecuteResultAsync(ActionContext)` — the method name, its return type and its parameter type all
   differ, so `override` cannot bind.
3. **The challenge mechanism is gone.** `GetOwinContext()` is a Katana extension over `System.Web`
   ([F-12-03](#f-12-03--the-katana-iappbuilder-abstraction-and-the-owin-startup-attribute)), and
   `IAuthenticationManager` — held as a property at
   [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:338-344] — has no successor type.
4. **`using Microsoft.Owin.Security;`** [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:10]
   supports `AuthenticationProperties` at
   [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:413], and goes with the rest of the OWIN
   surface.

**The surface this class serves is scaffolded but disabled.** Every external-provider registration in the
application is **commented out** — Microsoft Account, Twitter, Facebook and Google, all four inert
[src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:23-35] — so **no external sign-in can be initiated under
the configuration this repository commits, and no new linked login can be created through it**
([08 §9.3](08-technical-debt-register.md) records the scaffolding, and
[09 §6.11](09-security-assessment.md) records it as deployed attack surface).

**That is the whole of what the registration state proves, and this document stops there deliberately.**
Commented-out registrations are a fact about the committed configuration; they are not a fact about the
rows in the shipped credential store. Whether any stored account already holds a linked login — written
under some earlier configuration, on a developer machine, before the database binaries were committed —
is a property of the shipped Identity database, and it is **unknown from the repository** until the
authoritative schema and data extraction that [05 §5.5](05-aspnet-core-migration-approach.md) makes a gate
on the Identity migration actually reads that store. It is the same limit
[F-12-21](#f-12-21--the-identity-schema-is-not-knowable-from-the-repository) records for the schema itself:
what a committed binary contains is settled by extracting it, not by reading source beside it. **So the
linked-login rows are extracted, migrated and reconciled regardless of the registration state**, per
[05 §5.5](05-aspnet-core-migration-approach.md), which requires the row count to be compared per provider
name before and after and blocks the migration on any row that fails to re-link — so a zero, if that is
what the store holds, arrives as a measurement rather than as an inference. Reading the absence of a linked
login off `Startup.Auth.cs` is precisely the inference that would drop those rows silently, and nothing
below rests on it.

**The disposition is decided, and it is not "delete the external-login surface".** The decision belongs to
[05 §8.3](05-aspnet-core-migration-approach.md) and is **taken there as a split**, which this document
records rather than re-opens:

- **The provider-driven flow is deleted.** That is this entry's subject and everything that exists only to
  serve it: the `ChallengeResult` type [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:394-420]
  and its `ExecuteResult` override [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:411], the
  `IAuthenticationManager` property [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:338-344], the
  four commented registrations [src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:23-35], the external
  sign-in cookie [src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:20] and the **five sign-in-and-linking
  action paths** that constitute the flow — together with the failure display removed alongside them —
  which [§7.3](#73-the-three-decisions-this-document-defers-to-05--and-what-05-has-settled)
  names individually with their locators rather than restating them here.
- **The linked-login list and its removal surface are retained and ported**, because they have a direct
  successor where the challenge flow has none. `RemoveAccountList`
  [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:316-322], its partial
  [src/MVC5/MvcMusicStore/Views/Account/_RemoveAccountPartial.cshtml:1] and the `Disassociate` POST it
  drives [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:112-114] become the
  `RemoveAccountList` view component — one of the three that
  [05 §8.2](05-aspnet-core-migration-approach.md) defines. Its call site is the manage page's child action
  [src/MVC5/MvcMusicStore/Views/Account/Manage.cshtml:22], which is why the surface is reached on every
  visit even with no provider registered.
- **The linked-login rows migrate with it.** [05 §5.5](05-aspnet-core-migration-approach.md) owns the
  Identity data migration and preserves the source identifiers precisely so role assignments and
  external-login rows re-link during the load; nothing here is dropped on the grounds that no provider is
  currently registered.

So the entry is a **rewrite in two directions, not a deletion**: the type named in this entry's heading
goes, and the surface a reader might assume goes with it stays. Route and behaviour tests attach to both
halves, and [05 §8.3](05-aspnet-core-migration-approach.md) states them.

**Successor: none for `ChallengeResult` and the challenge flow** — the type is deleted rather than mapped.
The retained linked-login half's successor is the injected user manager's login-management API, which is
05's to name.

**Owner:** [05](05-aspnet-core-migration-approach.md).

### F-12-10 — The `BlockViewHandler` mapping and the Razor host section group

**Editions:** all three. Every edition ships a `Views/Web.config` carrying both constructs, at its own
line numbers and its own assembly versions:

| Edition | `BlockViewHandler` | Razor host section group | `pageBaseType` | Section-group assembly version |
| --- | --- | --- | --- | --- |
| MVC 5 | [src/MVC5/MvcMusicStore/Views/Web.config:31-32] | [src/MVC5/MvcMusicStore/Views/Web.config:5-8] | [src/MVC5/MvcMusicStore/Views/Web.config:13] | `3.0.0.0` [src/MVC5/MvcMusicStore/Views/Web.config:5] |
| MVC 4 | [src/MVC4/MvcMusicStore/Views/Web.config:55-56] | [src/MVC4/MvcMusicStore/Views/Web.config:5-8] | [src/MVC4/MvcMusicStore/Views/Web.config:13] | `2.0.0.0` [src/MVC4/MvcMusicStore/Views/Web.config:5] |
| MVC 3 | [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Views/Web.config:54-55] | [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Views/Web.config:5-8] | [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Views/Web.config:13] | `1.0.0.0` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Views/Web.config:5] |

Two per-edition differences are worth naming, because neither changes the disposition and both change the
work: the **imported-namespace count differs** — six in MVC 5
[src/MVC5/MvcMusicStore/Views/Web.config:15-20], five in MVC 4
[src/MVC4/MvcMusicStore/Views/Web.config:15-19] and four in MVC 3
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Views/Web.config:15-18] — and MVC 4 and MVC 3 carry a
**second, classic-mode** copy of the same not-found mapping in `<system.web><httpHandlers>`
[src/MVC4/MvcMusicStore/Views/Web.config:30] and
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Views/Web.config:29], alongside a `<system.web><pages>`
block declaring `System.Web.Mvc.ViewTypeParserFilter`, `ViewPage` and `ViewUserControl` — the WebForms
view-engine support MVC 5 dropped — at [src/MVC4/MvcMusicStore/Views/Web.config:40-48] and
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Views/Web.config:39-47]. Every one of those is a
`System.Web` construct, so the disposition below holds unchanged in all three.

The rest of this entry is stated at MVC 5's locators, as the migration source. `Views/Web.config` carries
two distinct constructs, and both are properties of the IIS integrated pipeline rather than of the
application:

- **A handler that makes the views directory unservable.** `<remove name="BlockViewHandler"/>` followed by
  `<add name="BlockViewHandler" path="*" verb="*" preCondition="integratedMode"
  type="System.Web.HttpNotFoundHandler" />` [src/MVC5/MvcMusicStore/Views/Web.config:31-32] — every path,
  every verb, mapped to a not-found handler so that `.cshtml` files under `Views/` cannot be requested
  directly.
- **A Razor host and page-base-type registration.** A `configSections` section group declaring the
  `host` and `pages` sections [src/MVC5/MvcMusicStore/Views/Web.config:5-8], then the
  `system.web.webPages.razor` element that sets `factoryType` to `MvcWebRazorHostFactory`
  [src/MVC5/MvcMusicStore/Views/Web.config:12] and `pageBaseType` to `System.Web.Mvc.WebViewPage`
  [src/MVC5/MvcMusicStore/Views/Web.config:13], with **six** namespaces imported into every view
  [src/MVC5/MvcMusicStore/Views/Web.config:14-21] — `System.Web.Mvc`, `System.Web.Mvc.Ajax`,
  `System.Web.Mvc.Html`, `System.Web.Optimization`, `System.Web.Routing` and `MvcMusicStore`
  [src/MVC5/MvcMusicStore/Views/Web.config:15-20].

**Failure mode: compile-time.** `System.Web.HttpNotFoundHandler`, `MvcWebRazorHostFactory` and
`WebViewPage` are all `System.Web` types; the `system.web.webPages.razor` configuration section does not
exist in the target; and `preCondition="integratedMode"` is meaningless outside IIS. Three of the six
imported namespaces name assemblies that are gone, one of them
([F-12-02](#f-12-02--systemweboptimization-bundling)) entirely.

**Successor: split, and one half simply ends.** The handler mapping needs no replacement — the target
does not serve content-root `.cshtml` files, so the behaviour the handler enforces is the default rather
than something to configure. The namespace registration has a direct successor: view imports move to a
`_ViewImports.cshtml` file, with the three surviving namespaces remapped and the three dead ones dropped.

**Owner:** [05](05-aspnet-core-migration-approach.md).

### F-12-11 — The `.axd` ignore route

**Editions:** all three — one call each, and in MVC 3 it is in a different file for the same reason
[F-12-05](#f-12-05--handleerrorattribute--the-entire-error-handling-policy) is.

`routes.IgnoreRoute("{resource}.axd/{*pathInfo}");`
[src/MVC5/MvcMusicStore/App_Start/RouteConfig.cs:14], the identical call at
[src/MVC4/MvcMusicStore/App_Start/RouteConfig.cs:14] — the two `RouteConfig.cs` files agree line for line,
differing only in the line ending on their final brace, which
`diff src/MVC5/MvcMusicStore/App_Start/RouteConfig.cs src/MVC4/MvcMusicStore/App_Start/RouteConfig.cs`
reports as its only hunk — and the same call in MVC 3 inside `RegisterRoutes` on the application class
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Global.asax.cs:22], since MVC 3 has no `App_Start` folder
([01 §3.6](01-architecture-overview.md)).

**Failure mode: compile-time.** `IgnoreRoute` is a `RouteCollection` extension in `System.Web.Routing`,
and neither the extension nor the collection type exists in the target.

**Successor: none; dropped.** `.axd` handlers were ASP.NET's mechanism for serving framework resources
such as `WebResource.axd` and `ScriptResource.axd`. The target has no such handler, so there is nothing
for the route to exclude — the line is not replaced, it becomes unnecessary.

**The route it accompanies is the favourable half of this entry.** The single conventional route in the
migration source [src/MVC5/MvcMusicStore/App_Start/RouteConfig.cs:16-20] — `{controller}/{action}/{id}`
with defaults `Home`, `Index` and `UrlParameter.Optional` — has a **direct successor form**, and it is the
whole of that edition's URL surface: one route, no areas, no attribute routing, no constraints. The other
two editions declare the same single route, at
[src/MVC4/MvcMusicStore/App_Start/RouteConfig.cs:16-20] and
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Global.asax.cs:24-28] — MVC 4 additionally maps a Web API
route that no `ApiController` implements [src/MVC4/MvcMusicStore/App_Start/WebApiConfig.cs:12-16], which
[08 §9.2](08-technical-debt-register.md) owns as dead scaffolding rather than as a blocker.
[01 §4.1](01-architecture-overview.md) owns the routing description.

**Owner:** [05](05-aspnet-core-migration-approach.md).

### F-12-12 — Assembly metadata in a source file

**Editions:** all three — `git ls-files '*AssemblyInfo.cs'` returns three paths, one per edition.

The migration source carries **12** assembly-level attributes in a source file, and all twelve are
enumerated below rather than summarized, because the SDK-style project must absorb each one individually
and an attribute left out of the mapping is an attribute the generated assembly silently loses. **A blank
value is still a declared attribute** — `AssemblyTrademark` and `AssemblyCulture` are declared with empty
strings in every edition, and both must be accounted for even though there is no value to carry.

The three editions declare **the same twelve attributes at the same twelve line numbers**, so one locator
column serves all three, and only three values differ:

| # | Attribute | Locator (identical in all three) | MVC 5 | MVC 4 | MVC 3 |
| --- | --- | --- | --- | --- | --- |
| 1 | `AssemblyTitle` | `:8` | `MvcMusicStore` | `MvcMusicStore` | `MvcMusicStore` |
| 2 | `AssemblyDescription` | `:9` | *blank* | *blank* | *blank* |
| 3 | `AssemblyConfiguration` | `:10` | *blank* | *blank* | *blank* |
| 4 | `AssemblyCompany` | `:11` | ***blank*** | `Microsoft` | `Microsoft` |
| 5 | `AssemblyProduct` | `:12` | `MvcMusicStore` | `MvcMusicStore` | `MvcMusicStore` |
| 6 | `AssemblyCopyright` | `:13` | `Copyright ©  2013` | `Creative Commons v3.0` | `Copyright © Microsoft 2011` |
| 7 | **`AssemblyTrademark`** | `:14` | *blank* | *blank* | *blank* |
| 8 | **`AssemblyCulture`** | `:15` | *blank* | *blank* | *blank* |
| 9 | `ComVisible` | `:20` | `false` | `false` | `false` |
| 10 | `Guid` | `:23` | `64547e1b-3030-4458-ab71-a970f2916ed6` | `a579f034-1dee-457c-aeeb-3578d111a7ad` | `aa8178f5-d968-4908-ae56-7235477f1139` |
| 11 | `AssemblyVersion` | `:34` | `1.0.0.0` | `1.0.0.0` | `1.0.0.0` |
| 12 | `AssemblyFileVersion` | `:35` | `1.0.0.0` | `1.0.0.0` | `1.0.0.0` |

Twelve rows, three editions, and the count is mechanical rather than asserted:

```bash
for f in src/MVC5/MvcMusicStore/Properties/AssemblyInfo.cs \
         src/MVC4/MvcMusicStore/Properties/AssemblyInfo.cs \
         src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Properties/AssemblyInfo.cs ; do
  grep -c '^\[assembly:' "$f"          # -> 12, 12, 12
done
grep -n '^\[assembly:' src/MVC5/MvcMusicStore/Properties/AssemblyInfo.cs
# -> lines 8,9,10,11,12,13,14,15,20,23,34,35 — the same twelve in each edition
```

Two facts in the table decide how the mapping is written. **The `Guid` value differs per edition**, so it
is not a shared constant and each project that is ever converted carries its own; only MVC 5's is in
scope here. And **`AssemblyCompany` is blank in the migration source alone**, which the conversion must
preserve as an explicit empty value rather than inherit the SDK default — the SDK falls back to the
assembly name when the property is absent, so omitting it would *introduce* a company name that the
current assembly does not have.

Each file is **35 lines** [src/MVC5/MvcMusicStore/Properties/AssemblyInfo.cs:1-35], of which the twelve
attribute lines above are the whole of the migratable content; the remainder is three `using` directives
[src/MVC5/MvcMusicStore/Properties/AssemblyInfo.cs:1-3] and four comment blocks, the last of which explains the version wildcard [src/MVC5/MvcMusicStore/Properties/AssemblyInfo.cs:25-33].

**Failure mode: compile-time**, and it is the mildest in this group: the SDK project format generates
these attributes from build properties, so a hand-written file that declares the same attributes produces
**duplicate-attribute** compile errors unless the file is removed or generation is switched off.

**Successor: MSBuild properties in the project file.** The metadata survives; the file does not.

**Owner:** [04](04-dotnet8-migration-strategy.md), which owns the project-format conversion.

### F-12-13 — Windows-authentication connection strings and file-attached databases

**Editions:** MVC 5 and MVC 4, with MVC 4 the worst case. **Not MVC 3**, for the reason stated below.

MVC 5 declares two connection strings, and both are unusable against a managed SQL service as written:
`Data Source=(LocalDb)\MSSQLLocalDB;AttachDbFilename=|DataDirectory|\aspnet-MvcMusicStore-20131025034205.mdf;Initial Catalog=...;Integrated Security=True`
[src/MVC5/MvcMusicStore/Web.config:12] and the catalogue equivalent with
`AttachDbFilename=|DataDirectory|\MvcMusicStore.mdf` and `Integrated Security=True` [src/MVC5/MvcMusicStore/Web.config:13].

Three properties each have to change, and they are independent:

- **`Integrated Security=True`** presents the worker process's Windows identity. In a managed hosting
  environment there is no domain identity to present, so this is not a credential to migrate but an
  authentication *model* to replace.
- **`AttachDbFilename` with `|DataDirectory|`** attaches a database from a file inside the deployment
  payload. File attachment **has no cloud analogue** — a managed database is reached by connection, not
  mounted from the application directory. [11 §3.3](11-cloud-readiness-assessment.md) owns this as a
  cloud-readiness blocker and records the 14 committed binaries and 43,376,640 bytes behind it.
- **`(LocalDb)\...`** names a developer-machine engine, not a server.

**MVC 4 compounds it with an internal contradiction.** Its committed connection strings target
`(LocalDb)\v11.0` [src/MVC4/MvcMusicStore/Web.config:13] and `(LocalDB)\v11.0` [src/MVC4/MvcMusicStore/Web.config:19] — SQL Server 2012
LocalDB — while **its own README documents `(LocalDB)\MSSQLLocalDB`** [src/MVC4/README.md:45], and Visual
Studio 2022, the README's stated prerequisite, installs no v11.0 instance. The repository therefore
disagrees with itself about which engine the application needs, and neither statement can be assumed
correct without a host check. [10](10-build-and-deployment-requirements.md) owns the per-edition topology
and the consequence for running MVC 4.

**MVC 3 is excluded from this entry, and the exclusion is a repository fact rather than a simplification.**
None of this entry's three parts holds for it:

- **No `Integrated Security` and no `AttachDbFilename`.** MVC 3 declares exactly **one** connection string,
  and it is SQL Server Compact: `Data Source=|DataDirectory|MvcMusicStore.sdf` with
  `providerName="System.Data.SqlServerCe.4.0"`
  [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:56-58]. Neither attribute appears in it. A
  repository-wide search does return `Integrated Security` twice under `src/MVC3/`, and it is worth naming
  why neither is evidence: both sit **inside a commented-out XDT example block** spanning lines 6 to 16 —
  [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Web.Debug.config:13] and
  [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Web.Release.config:13], the value being the template's own
  `Data Source=ReleaseSQLServer` example rather than anything this application connects to. A commented
  transform is never applied and the application never reads it.
- **No file-attached SQL Server database.** MVC 3's `|DataDirectory|` reference is to a SQL Server Compact
  `.sdf` reached by the Compact provider, not an `.mdf` attached to a SQL Server engine, so its
  cloud consequence is the provider having no supported successor at all —
  [F-12-01](#f-12-01--sql-server-compact-40-as-the-catalogue-provider) carries it, and carries it as the
  harder finding: a hard stop rather than a connection-string rewrite.
- **No configuration-reading API break.** `ConfigurationManager` does not appear in MVC 3's source at all,
  which is reproducible and stated in [section 8](#8-reproducibility-appendix): the two application
  occurrences in the repository are MVC 5's and MVC 4's provisioning code. MVC 3 has no `App_Start` folder
  and no administrator provisioning [01 §3.6](01-architecture-overview.md), so there is nothing in it that
  reads an app setting.

Listing MVC 3 here would double-count its single connection string — once under a blocker whose successor is
a rewritten connection string, and once under F-12-01, whose finding is that no successor exists — and the
second is the accurate one.

**Failure stages: three, and they are not one stage.** This entry is a *cluster* rather than a single
construct, so classifying the whole of it as a compile-time break would mis-describe two of its three
parts. The API that reads the values, the configuration source that supplies them and the values
themselves each fail at a different moment, and only the first is caught by a build:

| Part | What fails | Stage |
| --- | --- | --- |
| **The reading API** | `ConfigurationManager.AppSettings`, read directly from startup code [src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:23-24] behind `using System.Configuration;` [src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:7]. The type is not part of the target's shared framework, so the call does not resolve as written | **Compile-time (API).** The build names the file and the line. This is the part that places the entry in group one |
| **The configuration source** | `Web.config`'s `connectionStrings` and `appSettings` sections are **not a configuration source** in the target. Left in place, they are neither read nor reported — the setting is simply absent, and the consumer receives a null or empty value | **Configuration.** Nothing fails to compile and nothing warns. It surfaces on the first read: at host build for a connection string bound during startup, or at first use for a setting read later |
| **The values themselves** | `(LocalDb)\MSSQLLocalDB`, `AttachDbFilename=\|DataDirectory\|\…` and `Integrated Security=True` [src/MVC5/MvcMusicStore/Web.config:12-13] are valid connection-string syntax that a managed SQL service cannot honour: no LocalDB engine, no file to attach, no Windows identity to present | **Runtime / cloud.** Re-expressed faithfully into the new configuration mechanism, these values compile, bind and read successfully, then fail on the first connection attempt — and **only in the target environment**. A developer workstation with LocalDB installed connects happily, so this part passes every local check and fails in the cloud |

Two consequences follow, and they are the reason for splitting the entry rather than leaving it as one
line. First, the *mechanism* and the *values* are independent work: replacing
`ConfigurationManager.AppSettings` with the target's configuration abstraction does nothing about
`Integrated Security=True`, and re-expressing the connection string does nothing about the code that reads
it. Second, only the first part is self-announcing. The other two have to be named in advance — which is
what the three rows above do — because a port that satisfies the compiler and then reads a faithfully
copied LocalDB connection string has produced an application that builds, starts on a workstation, and
cannot reach a database in the environment it was migrated for.

**This entry is nonetheless counted once, in group one.** [Section 2.1](#21-the-distinction) places a mixed
entry in the group whose stage the port must **plan** for, and here that is compile time: the reading API
break is the part fixed in code, it is also the earliest stage at which this entry announces itself, and its
remaining two parts are closed by named deployment gates rather than by a search. F-12-13 is the one
**group-one** entry whose remaining parts can reach production undetected — the one **group-two** entry
whose mixture runs the other way is
[F-12-19](#f-12-19--connection-string-resolution-by-convention), and §2.1 states both;
[section 2.3](#23-blocker-index) and [section 7.1](#71-distribution) both record that explicitly rather
than leaving a reader to infer it from this section.

**Successor: a managed SQL service reached over an encrypted connection with a platform-managed
identity**, configured through the target's configuration abstraction. The connection-string content, the
identity model and the secret-delivery mechanism are [06](06-azure-hosting-recommendations.md)'s;
[09 §3.9](09-security-assessment.md) owns the stored-credential posture and
[11 §3.4](11-cloud-readiness-assessment.md) the cloud framing.

**Owner:** [06](06-azure-hosting-recommendations.md) for the target identity and secret delivery;
[05](05-aspnet-core-migration-approach.md) for the configuration-reading transition.

### F-12-14 — MVC 4's committed build configuration

**Editions:** MVC 4.

This entry exists because [10 §13.3](10-build-and-deployment-requirements.md) delegates the **blocker
classification** of MVC 4's build state to this document while retaining the diagnosis itself. The
diagnosis is not restated: 10 §6 records the two independent, platform-independent defects in the
committed project and solution files, and [10 §13.1](10-build-and-deployment-requirements.md) records the
outcome — MVC 4 **fails as committed**, in **both** solutions, **before compilation**.

**That negative is what this entry classifies, and it is not the whole of what has been observed — so the
distinction is drawn here rather than left to a reader.** [10 §6.3](10-build-and-deployment-requirements.md)
identifies the minimum host-side compensations an MVC 4 build *needs*, and records that they **were
exercised**: on a Windows host carrying the prescribed toolchain, after a standalone restore and with
exactly those two properties supplied on the command line, MVC 4 built to exit `0`. That observation is
[10 §3.2](10-build-and-deployment-requirements.md)'s — its host and tool inventory, its commands, its
per-edition results and its two qualifications are recorded there once — and this document cites it
without restating it, per [§1.5](#15-what-this-document-does-not-own).

**What the observation does and does not license, in the two respects that bear on this classification.**
It is a **command-line workaround, not a repair**: both committed defects are still present exactly as
cited below, so a plain invocation with no overrides still fails during evaluation, and the compensations
are applied to **nothing in the repository**. And it covers **build only, not runtime** — MVC 4's runtime
remains broken for an unrelated reason, its committed catalogue and membership connection strings naming a
local database instance that cannot be created on a current Windows Server
[src/MVC4/MvcMusicStore/Web.config:13], [src/MVC4/MvcMusicStore/Web.config:19], which
[10 §3.2](10-build-and-deployment-requirements.md) records as an `HTTP 500` on the application's root
request and [10 §10.3](10-build-and-deployment-requirements.md) diagnoses. So *"MVC 4 builds under
overrides"* and *"MVC 4's committed configuration is broken"* are **both true and neither cancels the
other**, which is why [10 §13.1](10-build-and-deployment-requirements.md) still carries this edition as
failing **as committed** — in **both** solutions, before compilation — and why that is the fact this
entry classifies.

The locations being classified, for reference only — 10 owns what is wrong with each and why. The two
project-file defects: an **unconditional** NuGet target import,
`<Import Project="$(SolutionDir)\.nuget\nuget.targets" />` carrying no `Condition` attribute
[src/MVC4/MvcMusicStore/MvcMusicStore.csproj:360], and the package `HintPath` entries that resolve one
directory above the committed payload, the first at
[src/MVC4/MvcMusicStore/MvcMusicStore.csproj:66] and 24 in total
(`grep -c 'HintPath' src/MVC4/MvcMusicStore/MvcMusicStore.csproj` returns `24`). Then, separately, the
stale fourth solution file, whose declared project path resolves to a directory that does not exist
[src/MVC4/MvcMusicStore/MvcMusicStore.sln:4].

**The classification.** MVC 4 cannot serve as a migration source or as an executable behavioural
baseline in the state the repository ships, because a source you cannot build from its own committed
configuration cannot be diffed against a port at runtime. That is a blocker on *using* MVC 4, not on
porting it — and since MVC 4 is not the migration source, its practical effect is narrower than it looks:
it removes MVC 4 from the set of candidate runtime baselines and leaves MVC 5 as the only candidate.

**What it does not do is establish that MVC 5 is an available baseline, and this document must not imply
that it does.** MVC 5's build status is **blocked pending a Windows verification run**
([10](10-build-and-deployment-requirements.md)) — the prescribed toolchain of .NET Framework 4.8
targeting pack, Visual Studio 2022 MSBuild and a restore against a declared package source was not
available on the host this assessment was written on
([10 §2.1](10-build-and-deployment-requirements.md)), so what was observed for MVC 5 there is a
**precondition** failure from an un-restored checkout, not a statement about whether the application
compiles once restored. The post-freeze host of [10 §3.2](10-build-and-deployment-requirements.md)
restored and built MVC 5 as well, and that section states the rule this document follows: the carried
status is unchanged, and no deliverable may report MVC 5's build as verified on the strength of it. So the
classification is narrower still: MVC 4 is removed from the candidate set on established,
platform-independent grounds, and MVC 5 remains the only candidate **whose availability as a baseline is
itself unverified**. A reader must not convert "the only candidate" into "a working baseline"; that
conversion is exactly the error [F-12-18](#f-12-18--a-framework-version-mismatch-inside-mvc-5s-own-configuration)
compounds, since a baseline nobody has captured cannot have been checked for the quirks-mode artifact
either.

**Failure mode: compile-time**, by definition — the build fails before any compilation happens.

**Successor: not applicable.** MVC 4 is retained read-only as a historical reference, and **nothing here
is repaired** — neither defect is fixed and no compensation is applied to any tracked file, because
[§1.3](#13-the-no-modification-constraint)'s gate forbids it. Remediation is
[08 §8.4](08-technical-debt-register.md)'s to own, for a separately approved phase.

**Owner:** [10](10-build-and-deployment-requirements.md) for the diagnosis, for the compensations
[10 §6.3](10-build-and-deployment-requirements.md) identifies and for the one post-freeze observation in
which they were exercised ([10 §3.2](10-build-and-deployment-requirements.md));
[08 §8.4](08-technical-debt-register.md) for severity and owner.

---

### F-12-23 — MVC child actions

**Editions:** all three.

A child action is an action marked `[ChildActionOnly]` and invoked from a view through `@Html.Action` or
`Html.RenderAction`. **The pattern was removed in ASP.NET Core in its entirety** — there is no
`ChildActionOnlyAttribute`, and `HtmlHelper` exposes neither `Action` nor `RenderAction` — so every
occurrence breaks on **both** halves: the declaration in the controller and the invocation in the view.
This entry is late in the numbering and early in the groups deliberately: it belongs to group one, and it
was added after the first pass because a construct that appears as one attribute and one helper call is
exactly the kind of thing a namespace-oriented reading walks past.

The census, per edition. Declarations are cited at the attribute line; the method they decorate is named
beside each:

| Edition | `[ChildActionOnly]` declarations | View call sites |
| --- | --- | --- |
| **MVC 5** | **3** — `GenreMenu` [src/MVC5/MvcMusicStore/Controllers/StoreController.cs:43], `CartSummary` [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:86], `RemoveAccountList` [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:316] | **3**, all `@Html.Action` — [src/MVC5/MvcMusicStore/Views/Shared/_Layout.cshtml:25] (`GenreMenu`), [src/MVC5/MvcMusicStore/Views/Shared/_Layout.cshtml:26] (`CartSummary`), [src/MVC5/MvcMusicStore/Views/Account/Manage.cshtml:22] (`RemoveAccountList`) |
| **MVC 4** | **4** — `GenreMenu` [src/MVC4/MvcMusicStore/Controllers/StoreController.cs:43], `CartSummary` [src/MVC4/MvcMusicStore/Controllers/ShoppingCartController.cs:86], `ExternalLoginsList` [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:322], `RemoveExternalLogins` [src/MVC4/MvcMusicStore/Controllers/AccountController.cs:329] | **5**, all `@Html.Action` — [src/MVC4/MvcMusicStore/Views/Shared/_Layout.cshtml:25] (`CartSummary`), [src/MVC4/MvcMusicStore/Views/Shared/_Layout.cshtml:32] (`GenreMenu`), [src/MVC4/MvcMusicStore/Views/Account/Login.cshtml:45] (`ExternalLoginsList`), [src/MVC4/MvcMusicStore/Views/Account/Manage.cshtml:24] (`RemoveExternalLogins`), [src/MVC4/MvcMusicStore/Views/Account/Manage.cshtml:27] (`ExternalLoginsList`) |
| **MVC 3** | **2** — `CartSummary` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/ShoppingCartController.cs:82], `GenreMenu` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/StoreController.cs:49] | **2**, both `Html.RenderAction` rather than `@Html.Action` — [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Views/Shared/_Layout.cshtml:16] (`CartSummary`), [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Views/Shared/_Layout.cshtml:21] (`GenreMenu`) |

`git grep -n 'ChildActionOnly' -- 'src/' | grep -v '/packages/' | wc -l` returns exactly `9` — 3 plus 4
plus 2 — so the declaration side is a census rather than a sample. MVC 3's use of the `RenderAction`
form is recorded because it is a different helper with a different return contract (it writes to the
response rather than returning a string), and a search for `@Html.Action` alone finds neither of its
two call sites.

**Failure mode: compile-time, on both halves, but they fail in different builds.**

1. **The declaration side fails the C# build.** `[ChildActionOnly]` cannot bind to an attribute type that
   does not exist. Nine declarations, nine errors.
2. **The call-site side fails the *view* build, which is a build the source project does not have.**
   `MvcBuildViews` is `false` in all three projects, so today's build never compile-checks a view
   ([10 §12.3](10-build-and-deployment-requirements.md), and [08 §8.1](08-technical-debt-register.md) as
   debt); the target compiles Razor views as part of the build, so
   `@Html.Action` and `Html.RenderAction` become build errors there. The consequence for planning is that
   the call sites are *not* discoverable by building the source before the port — they are discoverable
   only by this census or by the target's first build.

**Successor: view components — a rewrite, not a rename.** A view component is a distinct type with a
different base, a different entry-point method (`InvokeAsync`) and a different invocation form in the
view (a tag helper or `Component.InvokeAsync`), so nothing about the conversion is mechanical. The partial
views the three MVC 5 child actions render move with them: `Views/Store/GenreMenu.cshtml`,
`Views/ShoppingCart/CartSummary.cshtml` and — named in the action body itself,
`PartialView("_RemoveAccountPartial", linkedAccounts)`
[src/MVC5/MvcMusicStore/Controllers/AccountController.cs:321] — `Views/Account/_RemoveAccountPartial.cshtml`.
All three are tracked, confirmed by
`git ls-files 'src/MVC5/MvcMusicStore/Views/Store/GenreMenu.cshtml' 'src/MVC5/MvcMusicStore/Views/ShoppingCart/CartSummary.cshtml' 'src/MVC5/MvcMusicStore/Views/Account/_RemoveAccountPartial.cshtml'`,
which returns all three paths.

**The disposition is [05](05-aspnet-core-migration-approach.md)'s, and it is uniform across the three
MVC 5 occurrences.** All three become view components — `GenreMenu`, `CartSummary` and
`RemoveAccountList`, one per source declaration, each a `*ViewComponent.cs` class with its own
`Views/Shared/Components/<name>/Default.cshtml` ([05 §8.2](05-aspnet-core-migration-approach.md)). The
third is worth a sentence because it reads like an exception and is not: `RemoveAccountList` serves the
external-login surface whose every provider registration is commented out, but the *management* half of
that surface is **retained**, per the settled disposition recorded at
[F-12-09](#f-12-09--challengeresult-and-its-executeresult-override) and
[§7.3](#73-the-three-decisions-this-document-defers-to-05--and-what-05-has-settled) — what is deleted
there is the `ChallengeResult` type, the OWIN plumbing beneath it and the five sign-in-and-linking action
paths, while this child action and the `Disassociate` POST it drives are the half that stays. The
target therefore holds **three** components, not two, and no source declaration is dropped. This document
records the conversion and cites the disposition; it makes neither, exactly as it makes no other
blocker's target design.

**Two consequences worth stating, because the count understates them.** `GenreMenu` and `CartSummary` are
invoked from `_Layout.cshtml`, so they execute on **every page**, which is why their conversion is on the
hot path and why [08 §5.2](08-technical-debt-register.md) owns the query fan-out they cause.

And the second consequence is a lifetime property that is easy to state backwards. A view component is
**independently activated** — a distinct type, resolved and invoked by the framework rather than called as
a method — but it is resolved from the **request's** service provider and executes inline during the
parent view's render, so it runs in the **same request scope**: the same scoped `DbContext` instance and
the same change tracker as the controller whose action produced the view. It does **not** get a scope of
its own. Two practical rules follow, and they are the reason the conversion is behaviour-affecting rather
than cosmetic: **no component may call `SaveChangesAsync`**, because it would commit whatever the parent
action left pending in the shared tracker; and the shared tracker is precisely what a **same-request**
test has to assert, since a request that redirects before rendering proves nothing about it.
[05 §8.2](05-aspnet-core-migration-approach.md) owns both rules and names the one reachable request that
exercises them.

**MVC 4's four and MVC 3's two are historical record only.** MVC 5 is the sole migration source, so
nothing is ported from either; they are censused here because the register's edition column is a
statement about the repository rather than about the port.

**Owner:** [05](05-aspnet-core-migration-approach.md).

---

## 4. Group two — the successor exists and its default differs (silent breakage)

Everything in this section **compiles** — that is what these entries have in common and what no build will
tell you. For the eight silent entries it is also the property that makes them dangerous, and it is why
each of them enumerates sites rather than concepts.

**Group two is nine entries: the seven below, plus the two in
[section 5](#5-the-riskiest-data-operation--and-the-honest-limit-of-the-evidence) that complete it. Eight
of the nine are silent.** The exception is the first entry below,
[F-12-01](#f-12-01--sql-server-compact-40-as-the-catalogue-provider), and it is stated at the head of the
section rather than buried in it, because a reader who takes "group two" to mean "silent" would mis-plan
it. It sits here because it is **not found by compiling**, which is the axis §2.2 establishes as the one
that decides membership; it differs from the other eight in two ways, both stated in the entry: it has
**no successor of any kind** rather than a successor whose default differs, and it fails **loudly** at
provider activation rather than silently. Everything from
[F-12-15](#f-12-15--lazy-loading-is-on-by-default-in-ef-6-and-off-in-ef-core) onward is silent, and those
eight are what §2.2's argument about naming sites in advance is about.

### F-12-01 — SQL Server Compact 4.0 as the catalogue provider

**Editions:** MVC 3 only.

`web.config` declares the catalogue connection as `Data Source=|DataDirectory|MvcMusicStore.sdf` with
`providerName="System.Data.SqlServerCe.4.0"`
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:56-58]. That is the **only** connection string
the file declares — there is no second entry, which is the same evidence
[09 §5.1](09-security-assessment.md) uses to establish that MVC 3's credential store is inherited from
machine configuration rather than declared.

Two facts make this a hard stop rather than a provider swap:

- **No `.sdf` is tracked at HEAD.** `git ls-files '*.sdf' | wc -l` returns `0`, so the database the
  connection string names is not present in the checkout and is created on first run by a provider that
  must already be installed machine-wide.

  **It was, however, tracked historically, and the distinction matters.** The file was added as
  `MvcMusicStore/App_Data/MvcMusicStore.sdf` in `d4ff979` (2011-04-18, "Updated for v3.0 release (ASP.NET
  MVC 3 Tools Update)"), renamed to `src/MVC3/MvcMusicStore/App_Data/MvcMusicStore.sdf` in `fb88f8b`
  (2022-11-04, "Moved source to subfolder") and **deleted** in `d2dec66` (2022-11-04, "Replaced MVC 3 with
  release assets"). Its blob is still reachable — `8d8a3c458d70425b5f1942e7acc25911bc1d8042`, 217,088
  bytes — so a clone carries it even though no working-tree path resolves to it:

  ```bash
  git ls-files '*.sdf' | wc -l                                   # -> 0 at HEAD
  git log --all --name-status --format='%h %ad %s' --date=short -- '*.sdf'
  # -> d2dec66 ... D  src/MVC3/MvcMusicStore/App_Data/MvcMusicStore.sdf
  #    fb88f8b ... R100 MvcMusicStore/App_Data/MvcMusicStore.sdf -> src/MVC3/...
  #    d4ff979 ... A  MvcMusicStore/App_Data/MvcMusicStore.sdf
  git rev-list --all --objects | grep '\.sdf'
  # -> 8d8a3c458d70425b5f1942e7acc25911bc1d8042 src/MVC3/MvcMusicStore/App_Data/MvcMusicStore.sdf
  git cat-file -s 8d8a3c458d70425b5f1942e7acc25911bc1d8042        # -> 217088
  ```

  Neither fact softens the blocker: what the *current* MVC 3 project needs at run time is a first-run SQL
  Server Compact database, and the engine that would create it has no supported successor for the target
  (below). The deleted blob is a repository-weight and history observation rather than a usable artifact,
  and it is not part of the fourteen committed credential-shaped binaries that
  [09 §6.9](09-security-assessment.md) inventories.
  What *is* committed under the tutorial assets is
  `src/MVC3/MvcMusicStore-Assets/Data/MvcMusicStore.mdf` — a SQL Server file for a **different engine**
  than the one the application's own configuration names.
- **The data layer is bound to the EF 4.1 generation.** The project references `EntityFramework`
  4.1.0.0 by `HintPath` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:37-40] and
  the framework assembly `System.Data.Entity`
  [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj:41], and nothing else.

**Failure mode: runtime — provider activation. No build finds it, and it is the one group-two entry that
fails loudly.** `System.Data.SqlServerCe.4.0` is named only as a `providerName` attribute in configuration
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:58] and the project file declares **no
reference to the provider assembly at all** — the two data references it carries are `EntityFramework`
4.1.0.0 and the framework assembly `System.Data.Entity`, cited above — so nothing in the source binds to
the provider and nothing about it reaches the compiler. `git grep -n 'SqlServerCe' -- 'src/' | grep -v
'/packages/'` returns that single configuration line and nothing else, and the same search restricted to
`'*.csproj'` returns nothing. The failure is therefore raised when the provider is activated on the first
data access, as an exception rather than as a wrong answer. This is also why §2.1's placement A community EF Core provider for SQL Server
rule files this entry in group two rather than among the fourteen.
Compact exists but was **abandoned at the EF Core 2.2 generation**, so there is no supported provider for
any current target either — this is not a version-lag problem that waiting solves.

**Successor: none for the engine.** MVC 3's data layer cannot be ported without **re-targeting the
provider outright**, which means choosing a different database engine, not upgrading a package.
[02 §4.1](02-dependency-inventory.md) records the provider as an undeclared machine-wide dependency and
[09 §5.8](09-security-assessment.md) records that it is out of support and receives no security
servicing. [10 §10.2](10-build-and-deployment-requirements.md) owns the two-engine topology this
produces — MVC 3 needs SQL Server Compact for the catalogue **and** a SQL Server instance for
credentials, simultaneously.

**Owner:** [10](10-build-and-deployment-requirements.md) for the topology; the triage decision that MVC 3
is not a migration source is [03](03-modernization-roadmap.md)'s to sequence.

### F-12-15 — Lazy loading is on by default in EF 6 and off in EF Core

**Editions:** all three (the site census below is MVC 5, the migration source).

EF 6 populates navigation properties on demand through runtime proxies. EF Core does not: an
unloaded reference navigation is simply `null`. The code that depends on this compiles identically under
both.

**The mechanism is precise, and it explains which sites are affected and which are not.** EF 6 lazy
loading requires the navigation property to be `virtual`, and in this model they are not uniformly so.
The table below is **the whole navigation set of the six catalogue entities — eight properties**, not
the affected subset: `Artist` declares none, and the properties no code reads are listed with the rest
because an unread navigation still has to have its relationship configured in the target model, and an
initial migration generated from a model that dropped one produces a different schema. The last column
means *dereferenced* — read — so the seed's assignment of `Album.Genre` and `Album.Artist` on entities
it constructs [src/MVC5/MvcMusicStore/Models/SampleData.cs:24-485] is deliberately not counted: writing
a navigation on a new entity loads nothing.

| Navigation | Declared | Lazy-loadable under EF 6 | Read anywhere in source or views |
| --- | --- | --- | --- |
| `Album.Genre` | `virtual` [src/MVC5/MvcMusicStore/Models/Album.cs:30] | Yes | **Yes, client-side** — sites 1 and 2 of category (a) |
| `Album.Artist` | `virtual` [src/MVC5/MvcMusicStore/Models/Album.cs:31] | Yes | **Yes, client-side** — sites 1 and 2 of category (a) |
| `Album.OrderDetails` | `virtual` [src/MVC5/MvcMusicStore/Models/Album.cs:32] | Yes | **Yes, but only inside a server-translated query** — the two aggregates of category (c) [src/MVC5/MvcMusicStore/Controllers/StoreController.cs:49], [src/MVC5/MvcMusicStore/Controllers/HomeController.cs:30] |
| `Cart.Album` | `virtual` [src/MVC5/MvcMusicStore/Models/Cart.cs:17] | Yes | **Yes, client-side** — sites 3 to 6 of category (a); and once inside a translated query, category (b) |
| `OrderDetail.Album` | `virtual` [src/MVC5/MvcMusicStore/Models/OrderDetail.cs:11] | Yes | **No** — declared and never read |
| `OrderDetail.Order` | `virtual` [src/MVC5/MvcMusicStore/Models/OrderDetail.cs:12] | Yes | **No** — declared and never read |
| `Genre.Albums` | **not** `virtual` [src/MVC5/MvcMusicStore/Models/Genre.cs:10] | **No** | **Yes** — client-side in the browse view [src/MVC5/MvcMusicStore/Views/Store/Browse.cshtml:11], where it is eager-loaded, and inside the category (c) aggregate [src/MVC5/MvcMusicStore/Controllers/StoreController.cs:48] |
| `Order.OrderDetails` | **not** `virtual` [src/MVC5/MvcMusicStore/Models/Order.cs:66] | **No** | **No** — declared and never read; order-detail rows are written through the `DbSet` instead [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:147] |

That table is the reason the affected set is exactly what it is, and the reason it is **not** larger than
it is. Every silent-failure site below reaches through one of **three of the six** `virtual`
navigations — `Album.Genre`, `Album.Artist` and `Cart.Album`. A fourth, `Album.OrderDetails`, is read
only inside the two server-translated aggregates, which is category (c)'s different problem with a
different fix. The remaining two, `OrderDetail.Album` and `OrderDetail.Order`, are read nowhere at all,
so no site can break on them and no `Include` is owed for them; they bear on the target's relationship
configuration and on nothing else in this entry. And the only non-`virtual` navigation any code reads
client-side is `Genre.Albums`, which is **already eager-loaded out of necessity** —
`storeDB.Genres.Include("Albums")` [src/MVC5/MvcMusicStore/Controllers/StoreController.cs:30] is not an
optimization, it is mandatory, which is why the genre-browse page is unaffected even though its view
enumerates the collection. `Order.OrderDetails`, the other non-`virtual` one, is never read, so its
declaration carries no runtime consequence in either runtime.

**Three mechanisms, not one.** A flat list of "navigation dereferences" conflates cases that need
opposite treatment, so the census separates them:

**(a) Client-side dereference after materialization — definite break.** The entity is materialized first,
then the navigation is read in C# or in a view. EF Core returns `null`, so the read throws a null
reference or renders empty. **Six sites:**

| # | Query (no `Include`) | Dereference | Effect in the target |
| --- | --- | --- | --- |
| 1 | `storeDB.Albums.Find(id)` [src/MVC5/MvcMusicStore/Controllers/StoreController.cs:38] | `@Model.Genre.Name` [src/MVC5/MvcMusicStore/Views/Store/Details.cshtml:16] and `@Model.Artist.Name` [src/MVC5/MvcMusicStore/Views/Store/Details.cshtml:20] | Album detail page throws |
| 2 | `db.Albums.Find(id)` [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:32] | `@Html.DisplayFor(model => model.Artist.Name)` [src/MVC5/MvcMusicStore/Views/StoreManager/Details.cshtml:18] and `model.Genre.Name` [src/MVC5/MvcMusicStore/Views/StoreManager/Details.cshtml:26] | Admin detail page renders empty or throws |
| 3 | `storeDB.Carts.Single(item => item.RecordId == id)` [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:61-62] | `.Album.Title` on the same line [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:62] | Cart removal throws before it responds |
| 4 | `cart.GetCartItems()` — `ToList()` with no `Include` [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:100] | `.Select(a => a.Album.Title)` [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:91-92] | Cart summary in the shared layout throws — on **every** page, since it is a layout-level child action |
| 5 | `cart.GetCartItems()` via the cart view model [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:17-24] | `item.Album.Title` [src/MVC5/MvcMusicStore/Views/ShoppingCart/Index.cshtml:64] and `@item.Album.Price` [src/MVC5/MvcMusicStore/Views/ShoppingCart/Index.cshtml:68] | Cart page throws |
| 6 | `GetCartItems()` inside order creation [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:129] | `item.Album.Price` [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:145] | **Order total silently wrong or checkout throws** |

Site 6 deserves its own sentence, because it is the one with a financial consequence and it sits three
lines from a site that is *not* affected. The same loop explicitly loads the album for the order-detail
row — `var album = _db.Albums.Find(item.AlbumId);` [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:134], used for `UnitPrice` at [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:140] — and then
computes the running order total from the **lazy** navigation instead [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:145]. One loop, two ways of
reaching the same price, only one of which survives the port.

Site 4 deserves the second sentence, for reach rather than consequence: both child actions render from
the shared layout [src/MVC5/MvcMusicStore/Views/Shared/_Layout.cshtml:25-26], so a failure there is not
one page failing, it is every page failing.

**(b) Navigation read inside a server-translated query — survives; do not "fix" it.** One site:
`GetTotal` composes `select (int?)cartItems.Count * cartItems.Album.Price` over the `DbSet` and calls
`.Sum()` on the resulting queryable [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:119-122]. The
navigation is read inside an expression the provider translates to SQL, so no lazy load ever occurs and
EF Core translates it too. This site needs **verification that translation succeeds**, not an `Include` —
adding one would load entities the query does not need. Listing it as a lazy-loading break would send 05
to the wrong fix, which is the reason category (b) exists.

**(c) Nested collection aggregates — a translation and result question, not a lost client-side
fallback.** Two sites, and they are **not** the same aggregate, do not carry the same risk and do not run
with the same frequency, so the census keeps them apart:

| Site | The aggregate | Where it runs |
| --- | --- | --- |
| Genre menu | A **nested `SUM`** — `g.Albums.Sum(a => a.OrderDetails.Sum(od => od.Quantity))`, ordered descending and `Take(9)` [src/MVC5/MvcMusicStore/Controllers/StoreController.cs:46-52] | A `[ChildActionOnly]` action [src/MVC5/MvcMusicStore/Controllers/StoreController.cs:43] rendered from the shared layout [src/MVC5/MvcMusicStore/Views/Shared/_Layout.cshtml:25-26], so **every page** |
| Home top sellers | A **`COUNT`** — `a.OrderDetails.Count()`, ordered descending and `Take(6)` [src/MVC5/MvcMusicStore/Controllers/HomeController.cs:29-32] | The private helper it sits in is called only from `Index` [src/MVC5/MvcMusicStore/Controllers/HomeController.cs:18], so **the home page only** — not every page |

**The usual framing of this — "EF 6 quietly ran it in memory, EF Core throws" — is wrong in both halves,
and a plan built on it looks for the wrong failure.** EF 6 does **not** partially client-evaluate a
translated query: a LINQ-to-Entities expression it cannot translate **fails translation** at execution,
with `NotSupportedException`, and there is no silent fallback to lose. Broad silent partial client
evaluation was **EF Core's** behaviour, in 1.x and 2.x only, and was removed in EF Core 3.0 — from which
point an untranslatable expression outside the final projection throws instead. So **both** runtimes fail
loudly rather than falling back.

What remains, and it is the reason these two sites are in group two rather than group one, is that the
two providers may translate a nested aggregate to **different SQL** — different subquery shapes, and
different handling of an empty inner collection. That changes an observable **ordering** rather than
raising an exception: where a genre with no albums, or an album with no sales, sorts in a list ordered by
that aggregate.

**The empty-collection risk belongs to one of the two sites, and saying so is what keeps the fix
proportionate.** SQL `SUM` over **no rows** yields `NULL`, not `0`, so the genre menu's nested sum is the
site where an empty inner collection can propagate a null into the sort key — and where the two providers'
choices about coalescing it, and about the type they project it as, are observable in the ordering. SQL
`COUNT` over no rows yields **`0`**, so the home page's site has no null-propagation question at all: an
album with no `OrderDetails` counts zero and sorts last among zeros. Its risk is narrower and purely a
translation one — that the two providers emit a different subquery shape, and that ties among the many
albums with zero sales are broken in a different, and in both runtimes unspecified, order because neither
query names a tie-breaker.

The requirement is therefore to **inspect the generated SQL and assert the results for both sites in both
runtimes** — including the empty-collection case for the genre menu, and the many-way zero tie for the
home page — rather than to presume equivalence or to reason from a fallback neither runtime provides.
Nothing here asserts a runtime outcome from reading the source; the source's actual behaviour on these
pages is what the pre-port baseline captures. And the two sites' *reach* differs as the table above
records: the genre menu, like site 4 above, renders from the layout on every page, while the home page's
query runs on one page — so a regression in the first is a whole-site regression and a regression in the
second is a landing-page one.

**The site and file tally, stated once here and derived from the census above.** The nine sites are the
six rows of category (a), the single site of category (b) and the two rows of category (c) —
6 + 1 + 2 = **9**. They sit in **eight** distinct files. This table is the only enumeration of those files
in this document; every count of them elsewhere here, and in the deliverables that consume this entry, is
a reference to it rather than a second census:

| File | The sites it carries, and in what role |
| --- | --- |
| `src/MVC5/MvcMusicStore/Controllers/StoreController.cs` | site 1's query [src/MVC5/MvcMusicStore/Controllers/StoreController.cs:38]; category (c)'s nested `SUM` [src/MVC5/MvcMusicStore/Controllers/StoreController.cs:46-52] |
| `src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs` | site 2's query [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:32] |
| `src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs` | site 3's query and dereference [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:61-62]; site 4's dereference [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:91-92]; site 5's query [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:17-24] |
| `src/MVC5/MvcMusicStore/Controllers/HomeController.cs` | category (c)'s `COUNT` [src/MVC5/MvcMusicStore/Controllers/HomeController.cs:29-32], reached only from `Index` [src/MVC5/MvcMusicStore/Controllers/HomeController.cs:18] |
| `src/MVC5/MvcMusicStore/Models/ShoppingCart.cs` | site 4's query [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:100]; site 6's query and dereference [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:129], [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:145]; category (b)'s translated query [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:119-122] |
| `src/MVC5/MvcMusicStore/Views/Store/Details.cshtml` | site 1's dereference [src/MVC5/MvcMusicStore/Views/Store/Details.cshtml:16], [src/MVC5/MvcMusicStore/Views/Store/Details.cshtml:20] |
| `src/MVC5/MvcMusicStore/Views/StoreManager/Details.cshtml` | site 2's dereference [src/MVC5/MvcMusicStore/Views/StoreManager/Details.cshtml:18], [src/MVC5/MvcMusicStore/Views/StoreManager/Details.cshtml:26] |
| `src/MVC5/MvcMusicStore/Views/ShoppingCart/Index.cshtml` | site 5's dereference [src/MVC5/MvcMusicStore/Views/ShoppingCart/Index.cshtml:64], [src/MVC5/MvcMusicStore/Views/ShoppingCart/Index.cshtml:68] |

Five C# files and three views, 5 + 3 = **8**. Both figures are arithmetic over this one list: nine site
rows across the three category tables, eight rows here. **The file count is a property of the census and
not of any single search** — a file carries a site because a query without an `Include` and a dereference
of one of the `virtual` navigations meet there, which no one pattern matches — so the command verifies the
eight named paths rather than rediscovering them, and the second command names the two eager-loading sites
that decide the exclusions below:

```bash
git ls-files -- 'src/MVC5/MvcMusicStore/Controllers/StoreController.cs' \
  'src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs' \
  'src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs' \
  'src/MVC5/MvcMusicStore/Controllers/HomeController.cs' \
  'src/MVC5/MvcMusicStore/Models/ShoppingCart.cs' \
  'src/MVC5/MvcMusicStore/Views/Store/Details.cshtml' \
  'src/MVC5/MvcMusicStore/Views/StoreManager/Details.cshtml' \
  'src/MVC5/MvcMusicStore/Views/ShoppingCart/Index.cshtml' | wc -l          # -> 8
git grep -n 'Include(' -- 'src/MVC5/MvcMusicStore/Controllers'
# -> StoreController.cs:30 (string form) and StoreManagerController.cs:22 (typed form)
```

**Two files a dereference search returns and this census excludes, each for a stated reason.**
`Views/StoreManager/Index.cshtml` reads `Genre.Name` and `Artist.Name` at [src/MVC5/MvcMusicStore/Views/StoreManager/Index.cshtml:27], [src/MVC5/MvcMusicStore/Views/StoreManager/Index.cshtml:30], [src/MVC5/MvcMusicStore/Views/StoreManager/Index.cshtml:45] and [src/MVC5/MvcMusicStore/Views/StoreManager/Index.cshtml:48],
but its query eager-loads both — `db.Albums.Include(a => a.Genre).Include(a => a.Artist)`
[src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:22] — so nothing there breaks, which is the
same fact recorded under *Successor* below. `Views/Shared/_Layout.cshtml` renders two affected components
at [src/MVC5/MvcMusicStore/Views/Shared/_Layout.cshtml:25-26] but neither queries nor dereferences: it carries the *reach* of site 4 and of category (c)'s
genre menu, not a site of its own.

**Failure mode: silent runtime.** Nothing in category (a) produces a compiler diagnostic: the code is
valid, the navigation property exists on the type, and only its *value at runtime* changes.
[08 §5.2](08-technical-debt-register.md) separately owns the performance debt of the two layout-level
queries.

**Successor: explicit eager loading or projection at each site**, decided per site. Note the favourable
detail that reduces the work: the **string-based** `Include("Albums")`
[src/MVC5/MvcMusicStore/Controllers/StoreController.cs:30] remains **valid** in EF Core even though the
typed lambda form is the convention, and the typed form already in use
[src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:22] ports unchanged — which is exactly why
the admin **list** page is absent from the census while the admin **detail** page is site 2.

**Owner:** [05](05-aspnet-core-migration-approach.md), which must resolve each of the nine sites
individually and can only do so from this enumeration.

### F-12-16 — JSON property naming flips to camelCase

**Editions:** all three ship the same AJAX contract; the census is MVC 5.

The cart-removal endpoint returns a view model whose five properties are PascalCase — `Message`,
`CartTotal`, `CartCount`, `ItemCount`, `DeleteId`
[src/MVC5/MvcMusicStore/ViewModels/ShoppingCartRemoveViewModel.cs:5-9] — populated at
[src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:73-81] and returned as
`return Json(results);` [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:83].

The page's JavaScript reads **exactly those PascalCase names**, and the count needs its unit stated
because the reads are not one per name: **six lines carry seven `data.<Name>` reads of the five distinct
property names.** `data.ItemCount`
[src/MVC5/MvcMusicStore/Views/ShoppingCart/Index.cshtml:21], `data.DeleteId`
[src/MVC5/MvcMusicStore/Views/ShoppingCart/Index.cshtml:22], then **both `data.DeleteId` and
`data.ItemCount` a second time on one line**
[src/MVC5/MvcMusicStore/Views/ShoppingCart/Index.cshtml:24], `data.CartTotal`
[src/MVC5/MvcMusicStore/Views/ShoppingCart/Index.cshtml:27], `data.Message`
[src/MVC5/MvcMusicStore/Views/ShoppingCart/Index.cshtml:28] and `data.CartCount`
[src/MVC5/MvcMusicStore/Views/ShoppingCart/Index.cshtml:29]. So `ItemCount` and `DeleteId` are each read
twice and the other three once, which is why "five" is the right answer to *how many names break* and the
wrong answer to *how many reads there are*:

```bash
V=src/MVC5/MvcMusicStore/Views/ShoppingCart/Index.cshtml
git grep -oh 'data\.[A-Za-z]*' -- "$V" | sort | uniq -c
# -> 1 data.CartCount | 1 data.CartTotal | 2 data.DeleteId | 2 data.ItemCount | 1 data.Message
git grep -oh 'data\.[A-Za-z]*' -- "$V" | wc -l            # -> 7   reads
git grep -oh 'data\.[A-Za-z]*' -- "$V" | sort -u | wc -l   # -> 5   distinct names
git grep -n  'data\.[A-Za-z]'  -- "$V" | wc -l             # -> 6   lines (21,22,24,27,28,29)
# -oh suppresses the filename prefix that plain `git grep -o` prepends to every match.
```

**Get the attribution right, because the wrong attribution produces the wrong fix.** The serializer
producing that PascalCase output is **`JavaScriptSerializer`, which is what MVC 5's `JsonResult` uses**.
It is **not** Newtonsoft.Json. Newtonsoft.Json is pinned as a package — `5.0.6`
[src/MVC5/MvcMusicStore/packages.config:27] — but it is **never called from application source**:

```bash
git grep -nE 'Newtonsoft|JsonConvert|JsonProperty|JsonSerializer' -- '*.cs' '*.cshtml' | wc -l   # -> 0
```

Zero matches across every tracked `.cs` and `.cshtml` file in all three editions. The package is
template baggage, so its removal is **not** a serializer replacement — a distinction
[02 §3.1](02-dependency-inventory.md) relies on when it assigns the package an outcome.

**Failure mode: silent runtime, and this is the cleanest example in the document.** The target's default
web serializer camel-cases property names. After the port the request still succeeds, the response is
still `200`, the JSON is still well-formed — and every one of the seven reads above evaluates to
`undefined`. The row does not fade out, the count does not update, the total does not change, and no
error appears in any log, because no error occurs.

**The transport constrains the fix, so it is recorded here.** The removal posts through
`$.post(PostToUrl, { "id": recordToDelete }, ...)`
[src/MVC5/MvcMusicStore/Views/ShoppingCart/Index.cshtml:17] — **with no surrounding form**: the trigger is
a plain anchor carrying `data-id` and `data-url` attributes
[src/MVC5/MvcMusicStore/Views/ShoppingCart/Index.cshtml:74-75]. Because no `contentType` and no
`JSON.stringify` are specified, the request body is **form-urlencoded**, jQuery's default, not JSON. Two
consequences follow, and they belong to different deliverables: the response-naming problem is this
entry's, while the *request* side has no anti-forgery token because no form emits one — which is
[09 §3.7](09-security-assessment.md)'s finding F-09-08 and 05's to resolve when it adopts a validation
policy.

**Successor: explicit JSON property names on the response model.** The scoped fix is available precisely
because the affected surface is one endpoint and one model; the alternative of changing the
application-wide serializer policy is 05's call to reject or take, and this document does not take it.

**Owner:** [05](05-aspnet-core-migration-approach.md).

### F-12-17 — Filesystem path casing

**Editions:** MVC 5 and MVC 4 — one mismatched reference each, in the same construct. **MVC 3 is
excluded on evidence, not by omission**: it has no `App_Start` folder and therefore no bundle
registration ([01 §3.6](01-architecture-overview.md)), and its only stylesheet reference is
correctly cased — `@Url.Content("~/Content/Site.css")`
[src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Views/Shared/_Layout.cshtml:5].

The style bundle registers `"~/Content/site.css"`
[src/MVC5/MvcMusicStore/App_Start/BundleConfig.cs:28] — lowercase `s`. The tracked file is
`Site.css` — capital `S`:

```bash
git ls-files 'src/MVC5/MvcMusicStore/Content/*'
# -> src/MVC5/MvcMusicStore/Content/Site.css
#    src/MVC5/MvcMusicStore/Content/bootstrap.css
#    src/MVC5/MvcMusicStore/Content/bootstrap.min.css
```

Exactly three files, and the mismatch is a two-character difference in one string.

**MVC 4 carries the identical defect**, in its own single-file style bundle —
`new StyleBundle("~/Content/css").Include("~/Content/site.css")`
[src/MVC4/MvcMusicStore/App_Start/BundleConfig.cs:26] — against a tracked
`src/MVC4/MvcMusicStore/Content/Site.css`. Two editions, two references, and the census over every
tracked source, view, configuration and project file finds exactly those two and nothing else:

```bash
git grep -n 'Content/site\.css' -- '*.cs' '*.cshtml' '*.config' '*.csproj'
# -> src/MVC4/MvcMusicStore/App_Start/BundleConfig.cs:26
#    src/MVC5/MvcMusicStore/App_Start/BundleConfig.cs:28
git ls-files -- 'src/MVC4/MvcMusicStore/Content/Site.css' 'src/MVC5/MvcMusicStore/Content/Site.css' | wc -l   # -> 2
```

**Failure mode: silent runtime**, and it is silent in an unusually complete way. IIS resolves the path
because Windows filesystems are case-insensitive, so the page is correct today and no build, test or
review of the Windows application can surface it. On a case-sensitive filesystem the file is simply not
found: the stylesheet 404s, the page renders **unstyled but functional**, and nothing fails. There is no
exception, no failed request beyond the asset itself, and no log entry the application would write —
since the application writes none ([11 §3.8.3](11-cloud-readiness-assessment.md)).

**This one mismatch gates a hosting option rather than merely annoying a developer**, which is why it is
here and not only in the debt register: the reference is in a bundle registration that
[F-12-02](#f-12-02--systemweboptimization-bundling) deletes, so the specific line disappears — but the
**class** of defect does not, and it cannot be assumed to be the only instance. The audit therefore has
to cover bundle registrations, `@Url.Content` calls and view paths together.

**Successor: correct-cased paths, plus a repository-wide casing audit before a case-sensitive host is
committed to.** [11 §3.7](11-cloud-readiness-assessment.md) owns the finding and
[06](06-azure-hosting-recommendations.md) owns the audit as a precondition on the hosting option it
gates. Neither the audit's scope nor the hosting choice is decided here.

**Owner:** [06](06-azure-hosting-recommendations.md), with [11 §3.7](11-cloud-readiness-assessment.md) as
the finding of record.

### F-12-18 — A framework-version mismatch inside MVC 5's own configuration

**Editions:** MVC 5.

Two adjacent lines in the same element disagree about the framework:
`<compilation debug="true" targetFramework="4.8"/>` [src/MVC5/MvcMusicStore/Web.config:33] against
`<httpRuntime targetFramework="4.5"/>` [src/MVC5/MvcMusicStore/Web.config:34].

These control different things, which is why both can be present and neither is a typo. `compilation
targetFramework` selects the reference assemblies used to compile; `httpRuntime targetFramework` selects
the **runtime quirks mode**. With `httpRuntime` at 4.5, **4.5-era behaviours remain in force** — the
framework's own compatibility switches for request validation, encoding and related defaults resolve to
their 4.5 values, not their 4.8 values, even though the project targets 4.8 and the compilation element
says so.

**Failure mode: silent runtime — and it is a baseline-integrity blocker, not merely an inconsistency.**
That framing is the point of the entry. The behaviour a reviewer captures from the running MVC 5
application is the behaviour of a 4.5 quirks-mode runtime. Every comparison the port is validated
against inherits that, so:

- a behavioural baseline captured before the port records 4.5-era behaviour and calls it "current";
- the ported application has no quirks mode at all, so any difference traceable to this setting will
  present as a **port regression** when it is actually a pre-existing configuration artifact;
- and the reverse error is equally available — a real regression dismissed as "just the quirks mode".

Neither misreading is detectable after the fact, which is why the mismatch has to be recorded **before**
the baseline is captured rather than diagnosed afterwards.

**Successor: not applicable — the concept ends.** The target has no quirks-mode setting, so there is
nothing to carry the value into. What is required is that the baseline exercise account for it explicitly.

**Owner:** [05](05-aspnet-core-migration-approach.md) for the behavioural baseline;
[10 §13.1](10-build-and-deployment-requirements.md) records the build-side facts about the same project.

### F-12-19 — Connection-string resolution by convention

**Editions:** all three for the class-name convention, which every edition depends on; the explicit form
below is MVC 5's and MVC 4's.

Two conventions, **neither of which EF Core honours**:

- `MusicStoreEntities : DbContext` declares **no constructor at all** — six `DbSet` properties and
  nothing else [src/MVC5/MvcMusicStore/Models/MusicStoreEntities.cs:5-13]. EF 6 resolves its connection
  string by matching the **class name** against a `connectionStrings` entry, and the matching entry is
  named `MusicStoreEntities` [src/MVC5/MvcMusicStore/Web.config:13]. The coupling between the class and
  its database is a name, expressed nowhere in the code. **The same constructor-less class and the same
  name-matched entry exist in the other two editions**, so this half holds in all three: MVC 4 at
  [src/MVC4/MvcMusicStore/Models/MusicStoreEntities.cs:5-13] against the entry named at
  [src/MVC4/MvcMusicStore/Web.config:18], and MVC 3 at
  [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Models/MusicStoreEntities.cs:5-13] against the entry
  named at [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:56] — which, in MVC 3, is the
  **only** connection string the file declares
  ([F-12-01](#f-12-01--sql-server-compact-40-as-the-catalogue-provider)). The MVC 5 and MVC 4 files are
  byte-identical, which `cmp -s src/MVC5/MvcMusicStore/Models/MusicStoreEntities.cs
  src/MVC4/MvcMusicStore/Models/MusicStoreEntities.cs` confirms by exiting `0` with no output.
- `ApplicationDbContext : IdentityDbContext<ApplicationUser>` passes the name **explicitly**,
  `: base("DefaultConnection")` [src/MVC5/MvcMusicStore/Models/IdentityModels.cs:12-13], matching the
  entry at [src/MVC5/MvcMusicStore/Web.config:12]. MVC 4's second context uses the identical form for its
  SimpleMembership store — `UsersContext : DbContext` with `: base("DefaultConnection")`
  [src/MVC4/MvcMusicStore/Models/AccountModels.cs:13-14] against
  [src/MVC4/MvcMusicStore/Web.config:12] — so the explicit convention holds in two editions, and MVC 3
  has no second context at all, its credential store being inherited from machine configuration
  ([10 §10.2](10-build-and-deployment-requirements.md)).

**Failure mode: silent runtime, and the first context is the dangerous one.** A `DbContext` subclass with
no constructor is perfectly valid in EF Core — it compiles, and it can be instantiated. What changes is
that nothing tells it where the database is: EF Core has no class-name convention, and no `Web.config` to
consult ([F-12-13](#f-12-13--windows-authentication-connection-strings-and-file-attached-databases)). The
class-name coupling does not fail to compile, it fails to *mean anything*, and because the code that
loses meaning is the **absence** of a constructor, there is nothing at that location for a reviewer to
notice. A reader scanning `MusicStoreEntities.cs` for migration work sees thirteen lines of `DbSet`
declarations and no problem.

The explicit form is less risky for the opposite reason: `base("DefaultConnection")` is a constructor
call to a base overload that no longer exists, so it fails at compile time and announces itself. Two
conventions, the same underlying change, opposite failure modes — which is why they are one entry rather
than two, and why the entry is filed in this group: the half that reaches every edition is the silent one.

**Successor: an explicit `DbContextOptions` constructor on each context, with registration in the
container.** [01 §6.3](01-architecture-overview.md) owns the description of the two conventions as an
as-is architectural fact.

**Owner:** [05](05-aspnet-core-migration-approach.md), which owns the options and registration design.

### F-12-20 — `Dispose` overrides on objects a container will own

**Editions:** all three.

Four `Dispose(bool)` overrides dispose objects the code currently constructs and therefore currently
owns:

| Override | Object disposed | Constructed at |
| --- | --- | --- |
| [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:125-128] | `MusicStoreEntities` | field initializer [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:15] |
| [src/MVC4/MvcMusicStore/Controllers/StoreManagerController.cs:125-128] | `MusicStoreEntities` | field initializer [src/MVC4/MvcMusicStore/Controllers/StoreManagerController.cs:15] |
| [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/StoreManagerController.cs:112-115] | `MusicStoreEntities` | field initializer [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/StoreManagerController.cs:15] |
| [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:324-331] | `UserManager<ApplicationUser>` | chained constructor [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:19] |

Today each is correct: the controller creates the object, so the controller disposes it. The MVC 5 account
controller is even careful about it — it null-guards and nulls the field
[src/MVC5/MvcMusicStore/Controllers/AccountController.cs:326-330].

**Failure mode: silent runtime — and the consequence is conditional, which is the point rather than a
softening.** Once the container owns these lifetimes, the object handed to the controller is **scoped to
the request**, not to the controller, and the container disposes it at the end of the scope. An override
that disposes it earlier compiles cleanly — `Dispose(bool)` still exists on the target's controller base
class, so there is no signature error and no warning — and what happens next depends entirely on what
else, if anything, touches that same instance afterwards:

- **If a later consumer in the same scope uses that exact instance after the controller is released, it
  throws `ObjectDisposedException`** — at a call site that has nothing wrong with it. That consumer can be
  a view component or filter resolving the same scoped registration, an `IAsyncDisposable`/response-phase
  continuation, or a second component holding the same object.
- **If nothing else touches it, nothing visible happens.** The container's own disposal at end of scope is
  then a second dispose of an already-disposed object, which for `DbContext` and for
  `UserManager<T>` is idempotent, so the early disposal is wasteful and misleading rather than fatal — and
  it can pass every test in a suite that exercises one request at a time.

So the accurate statement is that the override makes the exception **reachable**, not that it produces one.
The requirement is unchanged by that, and rests on it: **the overrides must be removed when the container
takes over the lifetime**, because a defect whose manifestation depends on which other component happens
to share the scope is one that surfaces after the port rather than during it, and the condition that makes
it reachable is created by the very change that introduces the shared scope. Two properties make it hard
to catch when it does fire: it is **order-dependent**, so it may not reproduce under a single-request test,
and it appears at innocent code.

The account controller carries the sharper version of the risk, because its disposal is entangled with
[F-12-09](#f-12-09--challengeresult-and-its-executeresult-override)'s rewrite: the `UserManager` it
disposes is one it builds by hand through a chained constructor that also builds a `UserStore` and an
`ApplicationDbContext` [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:19]. When the container
supplies all three, an override disposing the middle one reaches an object two other consumers may still
hold — which is precisely the reachability condition above, satisfied by construction rather than by
chance.

**Successor: remove the overrides.** There is no replacement construct — container-owned lifetimes need no
disposal from the consumer, and removal is unconditional even where the exception would not have fired.

**Owner:** [05](05-aspnet-core-migration-approach.md), which owns the dependency-injection transition and
the ten manual instantiation sites it replaces.

---

## 5. The riskiest data operation — and the honest limit of the evidence

The two blockers below complete group two. They are separated into their own section because they share a
single cause: **the schema this application actually runs against is not knowable from the repository**,
and every downstream data decision inherits that gap. They are silent-failure blockers in the same sense
as the rest of group two — an EF Core migration generated from the ported model compiles, runs, and
creates a schema that may not be the one the data is in.

### F-12-21 — The Identity schema is not knowable from the repository

**Editions:** MVC 5.

A **printable-string inspection** of the committed credential store,
`src/MVC5/MvcMusicStore/App_Data/aspnet-MvcMusicStore-20131025034205.mdf` (3,211,264 bytes), finds the
early ASP.NET Identity column names and **none** of the six later ones:

```bash
python - <<'PY'
p='src/MVC5/MvcMusicStore/App_Data/aspnet-MvcMusicStore-20131025034205.mdf'
b=open(p,'rb').read()
for t in ('PasswordHash','SecurityStamp','UserName','AspNetUsers','AspNetRoles',
          'TwoFactorEnabled','LockoutEnabled','LockoutEndDateUtc',
          'AccessFailedCount','EmailConfirmed','PhoneNumber'):
    print('%-18s ascii=%d utf16le=%d' % (t, b.count(t.encode('ascii')),
                                            b.count(t.encode('utf-16-le'))))
PY
# -> PasswordHash       ascii=0 utf16le=2      TwoFactorEnabled   ascii=0 utf16le=0
#    SecurityStamp      ascii=0 utf16le=3      LockoutEnabled     ascii=0 utf16le=0
#    UserName           ascii=0 utf16le=2      LockoutEndDateUtc  ascii=0 utf16le=0
#    AspNetUsers        ascii=0 utf16le=33     AccessFailedCount  ascii=0 utf16le=0
#    AspNetRoles        ascii=0 utf16le=20     EmailConfirmed     ascii=0 utf16le=0
#                                              PhoneNumber        ascii=0 utf16le=0
```

Two properties of the probe are worth stating before the conclusion, because they are what make it worth
running at all. **It must read UTF-16, not ASCII**: SQL Server stores object names as `nvarchar`, so the
`ascii=0` column is uniformly zero — an ASCII-only probe finds *nothing*, including the columns that are
demonstrably there, and would produce a false negative on every row. And **it must count at every byte
offset** rather than decoding the file as text, because a decode fixes the alignment: decoding this file
as UTF-16 and searching the result finds `PasswordHash` once and `AspNetUsers` 31 times, where the
byte-offset scan above finds them 2 and 33 times. The probe agrees with itself on *presence and absence*
under both methods, and disagrees on counts.

**Now the qualification, stated explicitly, because the conclusion is worth less than it looks.**
**String-probing a binary is *evidence*, not *proof*.** It cannot distinguish an absent column from one
stored in a form the probe does not surface — a name held in a compressed or paged metadata structure, or
encoded differently from the two encodings tried, would read exactly like an absent one. The counts above
demonstrate the point rather than refuting it: a method whose numbers depend on its own alignment
handling is not a method that can settle a schema question. **A negative string-probe result is not a
proof of absence, and nothing downstream may treat it as one.**

**What the evidence does support.** The result is *consistent with* the independently established fact
that the application runs **ASP.NET Identity 1.0** — `Microsoft.AspNet.Identity.Core` 1.0.0
[src/MVC5/MvcMusicStore/packages.config:8] and `Microsoft.AspNet.Identity.EntityFramework` 1.0.0 [src/MVC5/MvcMusicStore/packages.config:9] —
whose schema predates all six of those columns. Two independent lines of evidence pointing the same way
is a strong prior. It is still a prior. [09](09-security-assessment.md) reaches the same conclusion by the
same route under finding **F-09-03** and qualifies it identically; this document's run corroborates it
rather than adding a second claim.

**Failure mode: silent runtime.** An EF Core initial migration generated from the ported Identity model
creates the *modern* schema. Nothing compares it against the real one. A mismatch in column presence,
type, precision, nullability, key definition, delete rule, default or index produces no build error and no
startup error — it produces wrong data, or a failed insert, at the first write.

**Therefore: authoritative schema extraction is a GATE, not a nicety.** An interrogation of the attached
database's system catalog — run on a supported Windows and LocalDB runtime, which is the only place the
file can be attached — must complete and be reconciled against the EF Core model **before** any Identity
data is migrated. **A column query alone does not satisfy the gate**, and it is worth saying so because a
column list is the obvious thing to reach for: the failure modes enumerated above include identity
metadata, defaults, check constraints, index key columns and delete rules, none of which a column query
returns. The complete catalog-view set the extraction must cover is
[05 §5.1](05-aspnet-core-migration-approach.md)'s, applied identically to the Identity store in
[05 §5.5](05-aspnet-core-migration-approach.md), and this document does not restate it. This document
records the gate; it does not discharge it, and it deliberately did not attach the database to try,
because attaching the tracked `.mdf` and `.ldf` files would cause SQL Server to write to them and dirty
the working tree, which [§1.3](#13-the-no-modification-constraint) forbids.

Two further facts make the gate wider than the Identity store, and one ordering consequence follows from
the second of them:

- **The same extraction gates the catalogue schema**, for the same reason and with no probe evidence at
  all behind it.
- **One of the three committed credential stores is simultaneously the only authoritative schema evidence
  and part of a Critical security finding, and it is worth naming rather than generalizing.** The pair the
  extraction needs is **MVC 5's Identity store alone** —
  [src/MVC5/MvcMusicStore/App_Data/aspnet-MvcMusicStore-20131025034205.mdf:3,211,264 bytes] with
  [src/MVC5/MvcMusicStore/App_Data/aspnet-MvcMusicStore-20131025034205_log.ldf:3,211,264 bytes] — because
  MVC 5 is the sole migration source and ships no schema script (F-12-22). The other two are **not** target
  migration inputs and no plan may treat them as such: MVC 3's
  [src/MVC3/MvcMusicStore-Assets/Data/ASPNETDB.MDF:10,485,760 bytes] is a tutorial asset rather than that
  edition's runtime credential store, as [09 §6.9](09-security-assessment.md) and
  [10 §10.1](10-build-and-deployment-requirements.md) both classify it, and MVC 4's
  [src/MVC4/MvcMusicStore/App_Data/aspnet-MvcMusicStore-20120831200627.mdf:3,211,264 bytes] is its
  edition's live SimpleMembership store but is not migrated, because MVC 4 is retained read-only as
  a behavioural reference. Every locator in this bullet is a byte size and not a line, per section 1.4,
  because none of these four files has a line to point at; one command reads all four:

  ```bash
  stat -c '%s %n' src/MVC5/MvcMusicStore/App_Data/aspnet-MvcMusicStore-20131025034205.mdf \
                  src/MVC5/MvcMusicStore/App_Data/aspnet-MvcMusicStore-20131025034205_log.ldf \
                  src/MVC3/MvcMusicStore-Assets/Data/ASPNETDB.MDF \
                  src/MVC4/MvcMusicStore/App_Data/aspnet-MvcMusicStore-20120831200627.mdf
  # ->  3211264 src/MVC5/MvcMusicStore/App_Data/aspnet-MvcMusicStore-20131025034205.mdf
  #     3211264 src/MVC5/MvcMusicStore/App_Data/aspnet-MvcMusicStore-20131025034205_log.ldf
  #    10485760 src/MVC3/MvcMusicStore-Assets/Data/ASPNETDB.MDF
  #     3211264 src/MVC4/MvcMusicStore/App_Data/aspnet-MvcMusicStore-20120831200627.mdf
  ```

  All three stores remain in scope for **F-09-34**, which
  [09 §6.9](09-security-assessment.md) owns as the committed-credential-store exposure at Critical
  severity; F-09-05 is the separate administrator-secret finding and is not the owner here.
- **The sequencing that follows is specific, not a general caution.** Because the MVC 5 Identity pair is
  the Identity migration's only schema source, **extraction must complete and be reconciled before the
  files are removed** — removing them first destroys the only evidence the migration can be validated
  against, and no earlier step regenerates it. F-09-34's remediation is therefore ordered *after* the
  extraction gate above for that pair, while MVC 3's and MVC 4's stores carry no such dependency and can be
  remediated independently. That ordering is [03](03-modernization-roadmap.md)'s to place, and it belongs
  on its critical path; [09 §8.2](09-security-assessment.md) records the same collision from the security
  side.

**Successor: explicit migrations reconciled against an extracted schema.**

**Owner:** [05](05-aspnet-core-migration-approach.md) for the Identity migration design and the
reconciliation; [03](03-modernization-roadmap.md) for placing the extraction before the migration.

### F-12-22 — No usable schema baseline exists

**Editions:** MVC 5 and MVC 4 — MVC 5 ships nothing and MVC 4 ships two copies of an unusable script,
both stated below.

**MVC 5 ships no schema script at all.** Three `.sql` files are tracked in the whole repository and none
of them is MVC 5's:

```bash
git ls-files 'src/MVC5/*.sql' | wc -l     # -> 0
git ls-files '*.sql'
# -> src/MVC3/MvcMusicStore-Assets/Data/MvcMusicStore-Create.sql
#    src/MVC4/MvcMusicStore-Create.sql
#    src/MVC4/MvcMusicStore/MvcMusicStore-Create.sql
```

The migration source's schema exists **only** inside the committed `.mdf` and in the shape EF 6 infers
from the model classes at runtime — which is precisely the gap
[F-12-21](#f-12-21--the-identity-schema-is-not-knowable-from-the-repository) makes a gate.

**MVC 4's two scripts are not runnable as written.** Both begin with a hard-coded developer path to an
attached MDF:

```sql
USE [C:\USERS\JON\DOCUMENTS\JON-SHARE\MVCMUSICSTORE-MVC3\MVCMUSICSTORE\MVCMUSICSTORE\APP_DATA\MVCMUSICSTORE.MDF]
```

— at [src/MVC4/MvcMusicStore-Create.sql:1] and, identically, at
[src/MVC4/MvcMusicStore/MvcMusicStore-Create.sql:1]. Both files are **UTF-16LE with a byte-order mark**,
so a plain text search over them matches nothing and a plain `head` prints mojibake; the appendix gives
the decoding form. A `USE` of a filesystem path resolves only where
that exact file is attached under that exact name, so the script will not execute against LocalDB on any
other machine and **cannot** execute against a managed SQL service, which has no attached-file databases
at all. The path also points at an `MVCMUSICSTORE-MVC3` directory, so neither copy is even
self-consistent about which edition it describes.

**The two copies are byte-identical**, confirmed by two independent methods — identical SHA-256 digests
and identical git blob identifiers:

```bash
git hash-object src/MVC4/MvcMusicStore-Create.sql src/MVC4/MvcMusicStore/MvcMusicStore-Create.sql
# -> 122e52c87bcf130cfe692088582b0a0fbeb2d04e
#    122e52c87bcf130cfe692088582b0a0fbeb2d04e     (identical: same content, 153,594 bytes each)
```

So the duplication offers no redundancy: **neither copy is usable as a schema baseline**, and there is no
second opinion to fall back on. [08 §6.3](08-technical-debt-register.md) owns the duplication as debt and
[10 §10.5](10-build-and-deployment-requirements.md) owns them as deployment inputs.

**Failure mode: silent runtime.** A script that will not run is not the hazard; a script that *looks* like
a schema baseline is. Treated as one, it yields a target schema derived from a 2012 developer machine's
MVC 3-era catalogue rather than from the database the application actually runs against — and the
divergence surfaces as data errors after cutover, not as an error at generation time.

**Successor: a schema extracted from the live database**, per
[F-12-21](#f-12-21--the-identity-schema-is-not-knowable-from-the-repository), with the generated migration
diffed against it before any data is loaded.

**Explicitly for [04](04-dotnet8-migration-strategy.md): neither `MvcMusicStore-Create.sql` copy may be
treated as a schema baseline.** That is the operative instruction from this entry, and it is stated here
because 04 is the deliverable most likely to reach for a committed `.sql` file as a starting point.

**Owner:** [05](05-aspnet-core-migration-approach.md) for the extraction and diff;
[04](04-dotnet8-migration-strategy.md) must not use either script as a baseline.

---

## 6. Portability findings in the application's favour

Four facts make this list materially shorter than the technology stack's reputation predicts. They are
included because **an enumeration of blockers with no counterweight is not an assessment, it is an
argument** — and because each of them removes work that a reader would otherwise budget for. Each carries
its verifying command.

### P-12-01 — `HttpContext.Current` appears nowhere

```bash
git grep -n 'HttpContext\.Current' -- '*.cs' '*.cshtml' | wc -l     # -> 0
```

Zero matches across every tracked `.cs` and `.cshtml` file in all three editions. **The hardest
`System.Web` coupling to unwind is simply absent.** A static ambient context reached from arbitrary code
at arbitrary depth is the coupling that turns a port into an archaeology exercise, because it has no
successor and every call site needs a plumbed alternative.

Instead, every context access in this codebase is one of two explicit forms, and **both have direct
equivalents**: an `HttpContextBase` parameter passed in deliberately — `GetCart(MusicStoreEntities db,
HttpContextBase context)` [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:21] and
`GetCartId(HttpContextBase context)` [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:161] — or the MVC
controller's own `HttpContext` and `Session` properties, as at
[src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:17] and
[src/MVC5/MvcMusicStore/Controllers/AccountController.cs:39]. The session *mechanism* still changes —
[11 §3.1](11-cloud-readiness-assessment.md) owns that — but the *access pattern* ports.

### P-12-02 — The one genuinely unportable method signature is already dead code

`ShoppingCart.GetCart(MusicStoreEntities db, Controller controller)`
[src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:29] takes a `System.Web.Mvc.Controller` — a type with no
target equivalent, and the only place in the model layer that names one. It would be an awkward port.

It is also **provably unreferenced**. All **six** call sites use the `HttpContextBase` overload at
[src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:21]:

```bash
git grep -n 'GetCart(store' -- 'src/MVC5/*.cs' | wc -l              # -> 6
```

— at [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:35],
[src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:17],
[src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:41],
[src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:58],
[src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:89] and
[src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:47]. Every one passes `this.HttpContext`.

**Removable with zero call-site impact** — materially easier than porting it, and the deletion is
[05](05-aspnet-core-migration-approach.md)'s to make rather than this assessment's.

### P-12-03 — Data access is uniformly EF LINQ; there is no raw SQL anywhere

```bash
git grep -nE 'ExecuteSqlCommand|SqlQuery|SqlCommand|CommandText' -- '*.cs' '*.cshtml' | wc -l    # -> 0
```

Zero matches, all three editions. **No SQL-dialect review is required**, and no hand-written statement has
to be re-verified against a different provider — the entire query surface is LINQ that the provider
translates. The three EF Core query concerns that remain are the ones
[F-12-15](#f-12-15--lazy-loading-is-on-by-default-in-ef-6-and-off-in-ef-core) enumerates, and they are
about translation and loading behaviour, not about SQL text.

### P-12-04 — The async posture is better than the vintage suggests

| Edition | `async` declarations | `await` occurrences | `Task<...>` signatures |
| --- | --- | --- | --- |
| MVC 5 | 9 | 22 | 7 |
| MVC 4 | 0 | 0 | 0 |
| MVC 3 | 0 | 0 | 0 |

```bash
for e in src/MVC5 src/MVC4 src/MVC3; do
  printf '%s async=%s await=%s task=%s\n' "$e" \
    "$(git grep -nE 'async (Task|void)' -- "$e/*.cs" | wc -l)" \
    "$(git grep -oE '\bawait\b'         -- "$e/*.cs" | wc -l)" \
    "$(git grep -nE 'Task<'             -- "$e/*.cs" | wc -l)"
done
git grep -nE '\.Result\b|\.Wait\(\)' -- '*.cs' | wc -l        # -> 0
git grep -n  'async void'            -- '*.cs'                # -> exactly one match
```

Two conclusions follow, and the second is the valuable one. MVC 4 and MVC 3 contain no asynchronous code
at all, so nothing about their async posture needs migrating. And in MVC 5 — the migration source —
**`.Result` and `.Wait()` appear nowhere in any edition**, so there is **no sync-over-async debt to
unwind**: the single most common source of deadlocks and of expensive rewrites during a port is absent.
The existing asynchronous methods are `await`-based throughout.

**There is exactly one `async void`**, and it is a genuine problem rather than a stylistic one:
`private async void CreateAdminUser()`
[src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:21], called fire-and-forget from `ConfigureApp`
[src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:18].
An `async void` method's exceptions cannot be observed by its caller, so a provisioning failure at startup
is silently lost — which is [09 §3.6](09-security-assessment.md)'s finding, not this document's. It is
recorded here only to bound the favourable claim: one method, one location, and it is a method whose
whole existence is being retired along with the credential it reads.

---

## 7. Roll-up and handoff

### 7.1 Distribution

**Every figure in this section is derived from the register above rather than maintained beside it.** A
blocker's group is the section its entry sits in; its reach is the subject of its own `**Editions:**`
declaration. The script at the end of this section computes all of it from those two properties and its
output is quoted beneath it, so no count here is a second declaration — each is a derivation of the
twenty-three entries [§2.3](#23-blocker-index) indexes, and each total is asserted to sum to that number.

| Group | Count | Discovered by | IDs |
| --- | --- | --- | --- |
| One — no successor or the API is gone | **14** | The compiler, for free | F-12-02 to F-12-14, plus F-12-23 |
| Two — not found by compiling: the successor exists and its default differs, plus F-12-01, which has no successor and no reference for a build to fail on | **9** | Only by someone who was told to look | F-12-01, plus F-12-15 to F-12-22 |
| Portability findings in the application's favour | 4 | — | P-12-01 to P-12-04 |

**14 + 9 = 23**, the row count of §2.3's index, and each group's ID list is the set of entries physically
under its section: group one is the thirteen consecutive identifiers F-12-02 through F-12-14 plus F-12-23,
and group two is F-12-01 at the head of [section 4](#4-group-two--the-successor-exists-and-its-default-differs-silent-breakage)
followed by F-12-15 through F-12-20 there and F-12-21 and F-12-22 in
[section 5](#5-the-riskiest-data-operation--and-the-honest-limit-of-the-evidence). Neither list is a
numeric range: `F-12-01` holds the lowest identifier and sits in group two, and `F-12-23` holds the
highest and sits in group one, both for the reasons §2.1 and §2.3 state.

**By edition, with the counting rule stated first, because several declarations carry commentary that
looks like a tally entry.** Each blocker counts toward the editions named in its own `**Editions:**`
declaration, and **a clause or parenthetical inside that declaration that explains, bounds or excludes is
commentary, not a declaration**. Seven declarations are the ones a tally most easily gets wrong, in both
directions:

- [F-12-02](#f-12-02--systemweboptimization-bundling) declares *MVC 5 and MVC 4* and adds parenthetically
  that MVC 3 has no bundling implementation at all — commentary confirming an exclusion, so MVC 3 gets no
  tally entry.
- [F-12-08](#f-12-08--three-httpnotfound-calls),
  [F-12-13](#f-12-13--windows-authentication-connection-strings-and-file-attached-databases) and
  [F-12-17](#f-12-17--filesystem-path-casing) declare *MVC 5 and MVC 4* and each states the evidence for
  excluding MVC 3 — again commentary confirming an exclusion, not a third entry. F-12-13's declaration is
  the one most easily mis-tallied, because it reads *"MVC 5 and MVC 4, with MVC 4 the worst case. **Not
  MVC 3**, for the reason stated below"*: the severity clause and the exclusion are both commentary, so it
  contributes two incidences and not three.
- [F-12-13](#f-12-13--windows-authentication-connection-strings-and-file-attached-databases) declares
  *MVC 5 and MVC 4* and then says **not MVC 3** in terms, with the exclusion argued in three parts inside
  the entry: MVC 3 declares neither `Integrated Security` nor `AttachDbFilename`, its `|DataDirectory|`
  reference is a SQL Server Compact `.sdf` rather than an attached `.mdf`, and it reads no app setting at
  all. It is an MVC 5 and MVC 4 blocker in every table below, and naming MVC 3 in the declaration is the
  exclusion rather than a third entry.
- [F-12-15](#f-12-15--lazy-loading-is-on-by-default-in-ef-6-and-off-in-ef-core) and
  [F-12-16](#f-12-16--json-property-naming-flips-to-camelcase) declare *all three* and then name the
  edition the site census was taken from — commentary narrowing the *evidence*, not the applicability, so
  all three are tallied.
- [F-12-19](#f-12-19--connection-string-resolution-by-convention) declares *all three* for the convention
  that reaches every edition and names the two editions carrying the second, explicit form. All three are
  tallied, because the declaration's subject is the first convention.

| Edition | Blockers | IDs |
| --- | --- | --- |
| MVC 5 — the migration source | **21** | Every blocker except F-12-01 (MVC 3 only) and F-12-14 (MVC 4 only) |
| MVC 4 | **17** | F-12-02, F-12-04, F-12-05, F-12-07, F-12-08, F-12-10, F-12-11, F-12-12, F-12-13, F-12-14, F-12-15, F-12-16, F-12-17, F-12-19, F-12-20, F-12-22, F-12-23 |
| MVC 3 | **12** | F-12-01, F-12-04, F-12-05, F-12-07, F-12-10, F-12-11, F-12-12, F-12-15, F-12-16, F-12-19, F-12-20, F-12-23 |
| Holding in **all three** | **11** | F-12-04, F-12-05, F-12-07, F-12-10, F-12-11, F-12-12, F-12-15, F-12-16, F-12-19, F-12-20, F-12-23 |

The three edition columns sum to **50** incidences — 21 + 17 + 12 — across 23 distinct blockers, and the
decomposition reconciles both numbers exactly. Every blocker appears in exactly one row:

| Reach | Blockers | IDs | Incidences |
| --- | --- | --- | --- |
| All three editions | 11 | F-12-04, F-12-05, F-12-07, F-12-10, F-12-11, F-12-12, F-12-15, F-12-16, F-12-19, F-12-20, F-12-23 | 11 × 3 = **33** |
| MVC 5 and MVC 4 only | 5 | F-12-02, F-12-08, F-12-13, F-12-17, F-12-22 | 5 × 2 = **10** |
| MVC 5 alone | 5 | F-12-03, F-12-06, F-12-09, F-12-18, F-12-21 | 5 × 1 = **5** |
| MVC 4 alone | 1 | F-12-14 | **1** |
| MVC 3 alone | 1 | F-12-01 | **1** |
| **Total** | **23** | — | **50** |

11 + 5 + 5 + 1 + 1 = 23 distinct blockers, and 33 + 10 + 5 + 1 + 1 = 50 incidences. Read the per-edition
table off the same decomposition: MVC 5 is 11 + 5 + 5 = 21, MVC 4 is 11 + 5 + 1 = 17, and MVC 3 is
11 + 1 = 12.

**Two corrections are folded into the figures above rather than left implicit, because both moved a
number.** `F-12-23` was absent from every per-edition row, which understated all three editions and the
totals; it declares *all three*, so it joins the eleven-blocker reach row and adds one incidence to each
edition. `F-12-13` was tallied to MVC 3 against its own declaration, which excludes MVC 3 in bold; it moves
from the all-three row to the MVC 5-and-MVC-4 row. The two changes cancel exactly in the MVC 3 column —
one in, one out — which is why MVC 3 reads **12** both before and after, and why a total that reconciled
against the old rows is not evidence that the rows were right.

**Every one of these declarations was re-verified against source rather than carried forward.** Six
entries previously understated their reach — F-12-05, F-12-08, F-12-10, F-12-11, F-12-17 and F-12-19 —
and each now cites the additional edition's own locator inside the entry, which is where the correction
can be checked. The direction of the error is worth naming: an *understated* edition set is the dangerous
one, because it silently narrows what the strategies believe they have to resolve.

The arithmetic is reproducible from this document's own declarations, which sit at the 23 lines the
following commands print — one per entry, and the partition is counted the same way rather than asserted:

```bash
grep -n '^\*\*Editions:\*\*' docs/modernization/12-migration-blockers.md | wc -l    # -> 23
grep -c '^\*\*Editions:\*\* all three' docs/modernization/12-migration-blockers.md  # -> 11
grep -c '^\*\*Editions:\*\* MVC 5 and MVC 4' docs/modernization/12-migration-blockers.md  # -> 4
# Those four are F-12-02, F-12-08, F-12-13 and F-12-17. Those four are F-12-02, F-12-08, F-12-13 and F-12-17. The fifth MVC 5-and-MVC-4
#
# blocker, F-12-22, declares its two editions with a per-edition parenthetical
#
# rather than the bare form, so the pattern above does not match it:
# `**Editions:** MVC 5 (nothing ships) and MVC 4 (two copies ...)`.
grep -n '^\*\*Editions:\*\*' docs/modernization/12-migration-blockers.md | grep -c 'MVC 3'  # -> 6
# Those six name MVC 3 on the declaration's own first line, and the figure is NOT
# an MVC 3 tally: F-12-01 (MVC 3 only, and the one incidence it contributes on its
# own), F-12-02, F-12-08, F-12-13 and F-12-17 (each naming MVC 3 in order to
# EXCLUDE it, so each contributes nothing), and F-12-11, which is one of the eleven
# `all three` declarations and merely notes MVC 3's different registration site.
# MVC 3's other eleven incidences are the `all three` declarations, one of which --
# F-12-19 -- names MVC 3 only on a continuation line the single-line pattern never
# reaches. 1 + 11 = 12, which is why the tally is read from the decomposition table
# above and never from a grep for an edition name.

# The group partition, counted by which section each entry is physically under
# rather than asserted from the identifier ranges (which are not ranges: F-12-01
# holds the lowest identifier and is in group two, F-12-23 the highest and is in
# group one). Group one is section 3; group two is sections 4 and 5:
awk '/^## 3\. /{g=1} /^## 4\. /{g=2} /^## 6\. /{g=0} /^### F-12-/{if(g)print g}' \
  docs/modernization/12-migration-blockers.md | sort | uniq -c
# ->      14 1
#          9 2
# 14 + 9 = 23, the same 23 the declaration count above returns.
```

Every published figure above is a line of that output: 23 entries, 14 plus 9, the five reach rows summing
to 23 blockers and 50 incidences, and the three per-edition tallies summing to the same 50.

Group two is where the count understates the work — nine blockers, but F-12-15 alone carries **nine
distinct sites in eight files**, enumerated once in
[that entry](#f-12-15--lazy-loading-is-on-by-default-in-ef-6-and-off-in-ef-core), and each site needs an
individual decision.

### 7.2 The four statements that change what the strategy documents can assume

Called out separately because each one invalidates an assumption a downstream deliverable would otherwise
make:

1. **A namespace substitution table is not a migration plan.** Six of the fourteen compile-time blockers
   — F-12-02, F-12-03, F-12-04, F-12-05, F-12-10, F-12-11 — are constructs whose files are **deleted**
   rather than rewritten, so their `using` directives never get substituted at all. Three more, F-12-07,
   F-12-09 and F-12-23, are rewrites where the *signature*, the declaring type or the invocation form
   changes, not the namespace. Any plan built around old-namespace-to-new-namespace mapping silently omits
   **nine of the fourteen**.
2. **The three constructs most likely to be misfiled as runtime problems are compile-time breaks.** The
   synchronous `TryUpdateModel` (F-12-07), the three `HttpNotFound()` calls (F-12-08) and `ChallengeResult`
   (F-12-09) all fail the **build**, because the overload, the method and the base type respectively do
   not exist in the target. They are not behavioural risks and must not be budgeted or tested as if they
   were. Section 2.3 and section 3 assign each of them explicitly, with the reason.
3. **Group two cannot be discharged by a policy statement.** "Use eager loading", "set the JSON naming
   policy" and "remove the `Dispose` overrides" are each true and each insufficient, because the work is
   per-site and the sites are not derivable from the policy. F-12-15's **nine sites across eight files**,
   F-12-16's **five fields read by seven accesses** and F-12-20's **four overrides** are the actionable
   units — each figure stated in the unit its entry measures — and
   [05](05-aspnet-core-migration-approach.md) must resolve them individually.
4. **No schema baseline exists, and the only authoritative source is also a Critical security finding.**
   F-12-21 and F-12-22 together mean the data workstream **starts** with extraction, not with an EF Core
   initial migration — and [09 §8.2](09-security-assessment.md)'s collision between removing the committed
   credential stores and needing them as schema evidence has to be sequenced on
   [03](03-modernization-roadmap.md)'s critical path rather than discovered late.

### 7.3 The three decisions this document defers to 05 — and what 05 has settled

Three decisions belong to [05](05-aspnet-core-migration-approach.md) rather than to this document, and
each has since been taken there. **This document still decides none of them.** What it records is the
enumeration plus the owner's settled outcome, because an entry that reports "a choice exists" after the
owner has made it sends a reader to design work that is already done:

- **The disabled external-login surface is split, not kept and not deleted wholesale** (F-12-09).
  [05 §8.3](05-aspnet-core-migration-approach.md) owns the disposition; this document records it and
  decides none of it. **Removed** is the external *sign-in and linking* action surface — the **five action
  paths** `ExternalLogin` [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:200],
  `ExternalLoginCallback` [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:209], `ExternalLoginConfirmation` [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:265], `LinkLogin` [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:237] and
  `LinkLoginCallback` [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:245]. The flow's failure display `ExternalLoginFailure` [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:311] is removed **with
  them rather than counted among them**: it initiates nothing and links nothing, and its only
  in-application reference is `return View("ExternalLoginFailure")` inside `ExternalLoginConfirmation`
  [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:278], which renders the view directly rather than routing to the action. The enumerated route list the
  port is tested against is [05](05-aspnet-core-migration-approach.md)'s, and this document cites that
  enumeration rather than restating a total of its own. Removed with them are the `ChallengeResult` type and
  its `ExecuteResult(ControllerContext)` override
  [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:394,411], whose base type, override signature
  and challenge mechanism have no successor, and every commented-out provider registration
  [src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:23-35]. No
  `Microsoft.AspNetCore.Authentication.*` provider package is referenced, and each removed path is
  expected to answer **404**. **Retained** is the account-management removal surface: the
  `RemoveAccountList` child action of [F-12-23](#f-12-23--mvc-child-actions), which becomes a view
  component, and the `Disassociate` POST that removes an already-linked login
  [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:114]. That half is not a preference —
  [05 §11.5](05-aspnet-core-migration-approach.md) carries the behaviour change as `D-11`, and
  the retained component is named in the frozen target map the strategies are held to. With no provider
  registered, `UserManager.GetLogins` returns nothing and the list renders **empty**
  [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:319]; that is correct behaviour for a store with
  no external provider, not a defect. Re-enabling external sign-in is a separately approved addition whose
  provider packages are pinned at the time it is approved.
- **The JSON naming fix is scoped to the one response model** (F-12-16).
  [05 §8.7](05-aspnet-core-migration-approach.md) annotates the five properties at
  [src/MVC5/MvcMusicStore/ViewModels/ShoppingCartRemoveViewModel.cs:5-9] with explicit JSON property
  names and **does not** change the application-wide serializer policy — the affected surface is one
  model and one endpoint, the single `return Json(results);`
  [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:83].
- **The two `DbContext` classes remain separate** (F-12-19), each with its own migration set and its own
  migrations-history table, per [05 §4.5](05-aspnet-core-migration-approach.md). This document's finding
  is unchanged — neither current convention survives — and the boundary it describes is kept rather than
  merged.

---

## 8. Reproducibility appendix

Every command in this document, collected for re-execution. All are **read-only**: none writes to the
working tree, none contacts the network, and none attaches a database. Run from the repository root.
POSIX forms, executed on this Windows host through the bundled Git-for-Windows `bash`.

```bash
# --- F-12-01  MVC 3's SQL Server Compact database is not in the repository -----
git ls-files '*.sdf' | wc -l                                            # -> 0
git ls-files 'src/MVC3/MvcMusicStore-Assets/Data/*'                     # -> an .mdf, not an .sdf

# --- F-12-01  Why it is runtime and not compile-time: a provider NAME, not a
#              reference. One occurrence, in configuration; no <Reference> anywhere
git grep -n -i 'SqlServerCe' -- 'src/' | grep -v '/packages/'
# -> src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config:58   (the only occurrence)
git grep -n '<Reference' -- 'src/MVC3/MvcMusicStore-Completed/MvcMusicStore/MvcMusicStore.csproj' \
  | grep -i 'SqlServerCe' | wc -l                                       # -> 0

# --- F-12-02  Bundling: five registrations, eleven view call sites -------------
git grep -n 'bundles.Add' -- 'src/MVC5/MvcMusicStore/App_Start/BundleConfig.cs' | wc -l   # -> 5
git grep -n '@Scripts.Render' -- 'src/MVC5/*.cshtml' | wc -l            # -> 10
git grep -n '@Styles.Render'  -- 'src/MVC5/*.cshtml' | wc -l            # -> 1

# --- F-12-04  The Global.asax markup declaration, one per edition --------------
git ls-files '*Global.asax'                                             # -> 3 paths

# --- F-12-05  HandleErrorAttribute is in all three editions -------------------
git grep -n 'HandleErrorAttribute' -- 'src/' | grep -v '/packages/'
# -> MVC5 App_Start/FilterConfig.cs:10, MVC4 App_Start/FilterConfig.cs:10,
#    MVC3 Global.asax.cs:17   (MVC 3 has no App_Start folder)

# --- F-12-08  The HttpNotFound census, per edition -----------------------------
git grep -n 'HttpNotFound()' -- 'src/MVC5/*.cs' | wc -l                 # -> 3
git grep -n 'HttpNotFound()' -- 'src/MVC4/*.cs' | wc -l                 # -> 3  (same lines)
git grep -n 'HttpNotFound()' -- 'src/MVC3/*.cs' | wc -l                 # -> 0  (none in the edition)

# --- F-12-11  The .axd ignore route is in all three editions ------------------
git grep -n 'IgnoreRoute' -- 'src/' | grep -v '/packages/'
# -> MVC5 App_Start/RouteConfig.cs:14, MVC4 App_Start/RouteConfig.cs:14,
#    MVC3 Global.asax.cs:22

# --- F-12-10  Views/Web.config, one per edition; the import sets differ -------
git ls-files '*Views/Web.config'                                        # -> 3 paths
git grep -c 'add namespace' -- '*Views/Web.config'                      # -> MVC5 6, MVC4 5, MVC3 4

# --- F-12-09  Every external provider registration is commented out -----------
git grep -nE '^\s*//app\.Use' -- 'src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs' | wc -l  # -> 4

# --- F-12-12  Assembly metadata as source, one per edition --------------------
git ls-files '*AssemblyInfo.cs'                                         # -> 3 paths

# --- F-12-13  Why the entry is MVC 5 and MVC 4 only, not all three ------------
# the reading API: two application call sites, both provisioning code, no MVC 3
git grep -n 'ConfigurationManager' -- 'src/' | grep -v '/packages/'
# -> MVC4 App_Start/AppConfig.cs:23,:24 and MVC5 App_Start/Startup.App.cs:23,:24
#    (nothing under src/MVC3 -- it has no App_Start folder and no provisioning)

# the values: live occurrences are MVC 5's and MVC 4's Web.config only
git grep -nE 'Integrated Security|AttachDbFilename' -- 'src/MVC5/MvcMusicStore/Web.config' \
    'src/MVC4/MvcMusicStore/Web.config'
# -> MVC5 :12,:13 ; MVC4 :15,:20,:21

# MVC 3's own -- one connection string, SQL Server Compact, neither attribute
git grep -n 'connectionString' -- 'src/MVC3/MvcMusicStore-Completed/MvcMusicStore/web.config'
# -> :57 Data Source=|DataDirectory|MvcMusicStore.sdf   (and :55/:59 the section tags)

# MVC 3's two 'Integrated Security' hits are inside a commented XDT example block
sed -n '6,16p' src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Web.Release.config
# -> the block opens '<!--' at :6 and closes '-->' at :16; the hit is at :13,
#    value 'Data Source=ReleaseSQLServer' -- the Visual Studio template example.
#    Web.Debug.config is identical in this respect.

# --- F-12-15  The whole navigation set, and which members are lazy-loadable ----
# All six catalogue entity files, not a subset: the table states the model's
# complete navigation set, so the command that reproduces it has to see Order.cs
# and OrderDetail.cs too.
M=src/MVC5/MvcMusicStore/Models
git grep -nE 'virtual [A-Za-z]|List<Album>|List<OrderDetail>' -- "$M/Album.cs" \
    "$M/Artist.cs" "$M/Genre.cs" "$M/Cart.cs" "$M/Order.cs" "$M/OrderDetail.cs"
# -> exactly 8 lines, one per navigation property:
#    Album.cs:30,:31,:32   virtual   Genre, Artist, List<OrderDetail>
#    Cart.cs:17            virtual   Album
#    OrderDetail.cs:11,:12 virtual   Album, Order
#    Genre.cs:10           NOT virtual   List<Album>
#    Order.cs:66           NOT virtual   List<OrderDetail>
#    Artist.cs             no match -- it declares no navigation at all
# Which of the eight are actually READ is the table's last column. Three are read
# nowhere -- OrderDetail.Album, OrderDetail.Order and Order.OrderDetails -- and
# this pair of commands is what shows it:
git grep -nE '\.OrderDetails' -- 'src/MVC5/MvcMusicStore'
# -> exactly 3 lines. Two are reads of Album.OrderDetails, both inside a
#    server-translated query: StoreController.cs:49 and HomeController.cs:30.
#    The third is ShoppingCart.cs:147, '_db.OrderDetails.Add' -- a DbSet write,
#    not a navigation read. So no line reads Order.OrderDetails: its declaration
#    at Order.cs:66 has no reader. (The three declarations carry no leading dot
#    and so do not appear: Album.cs:32, Order.cs:66, MusicStoreEntities.cs:12.)
git grep -nE 'orderDetail\.|OrderDetail\.[A-Z]' -- 'src/MVC5/MvcMusicStore'
# -> no match, exit 1: neither OrderDetail.Album nor OrderDetail.Order is
#    dereferenced anywhere in source or views.

# --- F-12-15  The dereference census: every navigation read in source and views
git grep -nE '\.(Album|Genre|Artist)\.' -- 'src/MVC5/MvcMusicStore/Controllers' \
    'src/MVC5/MvcMusicStore/Models/ShoppingCart.cs' 'src/MVC5/MvcMusicStore/Views'
# Read the result against the eager-loading sites, which are the two exclusions:
git grep -n 'Include(' -- 'src/MVC5/MvcMusicStore/Controllers'
# -> StoreController.cs:30 (string form, still valid in EF Core) and
#    StoreManagerController.cs:22 (typed form) -- their views are NOT affected

# Neither grep produces the tally: a dereference search misses site 2's query,
# which is db.Albums.Find(id) at StoreManagerController.cs:32 rather than a
# navigation read. The nine sites, the eight files they sit in and the
# verification of those eight paths are enumerated once, in the F-12-15 entry,
# and are not restated here.

# --- F-12-16  The property-access census: five fields, seven accesses, six lines
V=src/MVC5/MvcMusicStore/Views/ShoppingCart/Index.cshtml
git grep -oE 'data\.[A-Za-z]+' -- "$V" | wc -l                          # -> 7  (accesses)
git grep -ohE 'data\.[A-Za-z]+' -- "$V" | sort -u | wc -l               # -> 5  (distinct fields)
git grep -nE 'data\.[A-Za-z]+' -- "$V" | wc -l                          # -> 6  (source lines)
git grep -ohE 'data\.[A-Za-z]+' -- "$V" | sort | uniq -c
# -> 1 data.CartCount, 1 data.CartTotal, 2 data.DeleteId,
#    2 data.ItemCount, 1 data.Message
# The 7-vs-6 difference is line 24, which reads DeleteId and ItemCount in one
# statement -- an enumeration by line drops that access.

# --- F-12-16  Newtonsoft.Json is pinned but never called from source ----------
git grep -nE 'Newtonsoft|JsonConvert|JsonProperty|JsonSerializer' -- '*.cs' '*.cshtml' | wc -l   # -> 0
git grep -n 'Newtonsoft' -- 'src/MVC5/MvcMusicStore/packages.config'    # -> :27, version 5.0.6

# --- F-12-17  Path casing: two editions, two references, one capital S -------
# The mismatched reference, censused over every tracked source, view, config and
# project file -- both editions, and nothing else:
git grep -n 'Content/site\.css' -- '*.cs' '*.cshtml' '*.config' '*.csproj'
# -> src/MVC4/MvcMusicStore/App_Start/BundleConfig.cs:26
#    src/MVC5/MvcMusicStore/App_Start/BundleConfig.cs:28
# The tracked files those two references miss on a case-sensitive filesystem:
git ls-files -- 'src/MVC4/MvcMusicStore/Content/Site.css' \
                'src/MVC5/MvcMusicStore/Content/Site.css' | wc -l   # -> 2
git ls-files | grep -cE 'Content/site\.css$'    # -> 0  (no lowercase file exists)
# MVC 5's Content directory, for the three-file figure the entry quotes:
git ls-files 'src/MVC5/MvcMusicStore/Content/*'
# -> Site.css, bootstrap.css, bootstrap.min.css   (the bundle registers "site.css")
# MVC 3 contributes nothing: it has no App_Start folder, and its only stylesheet
# reference is correctly cased at Views/Shared/_Layout.cshtml:5.

# --- F-12-21  Credential-store string probe (EVIDENCE, NOT PROOF) ------------
# Must read UTF-16: SQL Server stores object names as nvarchar, so an ASCII-only
# probe finds nothing at all -- including the columns that are demonstrably there.
# Must count at every byte offset: decoding the file as text fixes the alignment
# and changes the counts (PasswordHash 1 vs 2, AspNetUsers 31 vs 33).
python - <<'PY'
p='src/MVC5/MvcMusicStore/App_Data/aspnet-MvcMusicStore-20131025034205.mdf'
b=open(p,'rb').read()
for t in ('PasswordHash','SecurityStamp','UserName','AspNetUsers','AspNetRoles',
          'TwoFactorEnabled','LockoutEnabled','LockoutEndDateUtc',
          'AccessFailedCount','EmailConfirmed','PhoneNumber'):
    print('%-18s ascii=%d utf16le=%d' % (t, b.count(t.encode('ascii')),
                                            b.count(t.encode('utf-16-le'))))
PY
# -> found (utf16le):     PasswordHash 2, SecurityStamp 3, UserName 2,
#                         AspNetUsers 33, AspNetRoles 20
# -> not found at all:    TwoFactorEnabled, LockoutEnabled, LockoutEndDateUtc,
#                         AccessFailedCount, EmailConfirmed, PhoneNumber
# The negative result is CONSISTENT WITH ASP.NET Identity 1.0 and is not proof of it.
git grep -n 'Identity' -- 'src/MVC5/MvcMusicStore/packages.config'      # -> :8,:9,:10 all 1.0.0

# --- F-12-21  Byte-size locators for the three committed credential stores ----
# A database file has no line to cite, so its locator is its byte size (section 1.4).
stat -c '%s %n' src/MVC5/MvcMusicStore/App_Data/aspnet-MvcMusicStore-20131025034205.mdf \
                src/MVC5/MvcMusicStore/App_Data/aspnet-MvcMusicStore-20131025034205_log.ldf \
                src/MVC3/MvcMusicStore-Assets/Data/ASPNETDB.MDF \
                src/MVC4/MvcMusicStore/App_Data/aspnet-MvcMusicStore-20120831200627.mdf
# ->  3211264 src/MVC5/MvcMusicStore/App_Data/aspnet-MvcMusicStore-20131025034205.mdf
#     3211264 src/MVC5/MvcMusicStore/App_Data/aspnet-MvcMusicStore-20131025034205_log.ldf
#    10485760 src/MVC3/MvcMusicStore-Assets/Data/ASPNETDB.MDF
#     3211264 src/MVC4/MvcMusicStore/App_Data/aspnet-MvcMusicStore-20120831200627.mdf
git ls-files -- src/MVC5/MvcMusicStore/App_Data/aspnet-MvcMusicStore-20131025034205.mdf \
                src/MVC5/MvcMusicStore/App_Data/aspnet-MvcMusicStore-20131025034205_log.ldf \
                src/MVC3/MvcMusicStore-Assets/Data/ASPNETDB.MDF \
                src/MVC4/MvcMusicStore/App_Data/aspnet-MvcMusicStore-20120831200627.mdf | wc -l   # -> 4

# --- F-12-22  No MVC 5 schema script; MVC 4's two copies are byte-identical ---
git ls-files 'src/MVC5/*.sql' | wc -l                                   # -> 0
git ls-files '*.sql'                                                    # -> 3, none under src/MVC5
git hash-object src/MVC4/MvcMusicStore-Create.sql \
                src/MVC4/MvcMusicStore/MvcMusicStore-Create.sql
# -> 122e52c87bcf130cfe692088582b0a0fbeb2d04e   (both: identical content)

# Both scripts are UTF-16LE with a BOM, so `head`, `grep` and `git grep` are all
# useless on them -- a plain `head -c` prints mojibake and a text search matches
# nothing. Decode before reading:
python -c "print(open('src/MVC4/MvcMusicStore-Create.sql',encoding='utf-16').readline().rstrip())"
# -> USE [C:\USERS\JON\DOCUMENTS\JON-SHARE\MVCMUSICSTORE-MVC3\MVCMUSICSTORE\MVCMUSICSTORE\APP_DATA\MVCMUSICSTORE.MDF]

# --- F-12-23  Child actions: nine declarations, ten call sites ----------------
git grep -n 'ChildActionOnly' -- 'src/' | grep -v '/packages/' | wc -l  # -> 9  (3 + 4 + 2)
git grep -n 'Html.Action(' -- 'src/MVC5' 'src/MVC4' | grep -v '/packages/'
# -> MVC 5: _Layout.cshtml:25,:26 and Account/Manage.cshtml:22            (3)
# -> MVC 4: _Layout.cshtml:25,:32, Account/Login.cshtml:45,
#           Account/Manage.cshtml:24,:27                                  (5)
# MVC 3 uses the OTHER helper, so the search above finds neither of its sites:
git grep -n 'Html.RenderAction' -- 'src/MVC3/MvcMusicStore-Completed'
# -> Views/Shared/_Layout.cshtml:16,:21                                   (2)

# --- P-12-01  HttpContext.Current appears nowhere ---------------------------
git grep -n 'HttpContext\.Current' -- '*.cs' '*.cshtml' | wc -l          # -> 0

# --- P-12-02  The dead overload: six call sites, all the other overload -----
git grep -n 'GetCart(store' -- 'src/MVC5/*.cs' | wc -l                   # -> 6
git grep -n 'static ShoppingCart GetCart' -- 'src/MVC5/MvcMusicStore/Models/ShoppingCart.cs'
# -> :21 (HttpContextBase, used six times) and :29 (Controller, used zero times)

# --- P-12-03  No raw SQL execution anywhere --------------------------------
git grep -nE 'ExecuteSqlCommand|SqlQuery|SqlCommand|CommandText' -- '*.cs' '*.cshtml' | wc -l   # -> 0

# --- P-12-04  Async posture per edition ------------------------------------
for e in src/MVC5 src/MVC4 src/MVC3; do
  printf '%s async=%s await=%s task=%s\n' "$e" \
    "$(git grep -nE 'async (Task|void)' -- "$e/*.cs" | wc -l)" \
    "$(git grep -oE '\bawait\b'         -- "$e/*.cs" | wc -l)" \
    "$(git grep -nE 'Task<'             -- "$e/*.cs" | wc -l)"
done
# -> src/MVC5 async=9 await=22 task=7 | src/MVC4 all 0 | src/MVC3 all 0
git grep -nE '\.Result\b|\.Wait\(\)' -- '*.cs' | wc -l                   # -> 0
git grep -n 'async void' -- '*.cs'                # -> exactly 1: Startup.App.cs:21

# --- The constraint this work was held to (section 1.3) -------------------
# 'ea2552d' is the authoring baseline and the only commit id these documents
# name; 'HEAD' is the checkout's tip. The figures are what these return on a
# checkout whose tip is this engagement's final commit. Deliverable 08 section
# 1.3 owns the statement, including where the durable provenance is recorded.
git status --porcelain                                            # -> empty (current checkout)
# the one non-modification check of section 1.3: immutable baseline on the left, the
# delivery commit the reviewer has checked out (HEAD) on the right
git diff --name-status ea2552d..HEAD | wc -l                       # -> 13
git diff --name-status ea2552d..HEAD | grep -c '^A'                # -> 13
git diff --name-status ea2552d..HEAD | grep -vc '^A'               # -> 0
git diff --name-status ea2552d..HEAD | grep -v 'docs/modernization/' | wc -l      # -> 0
```

Two notes on reading these results. `git grep` **exits 1 when it finds nothing**, which is a success for
every absence claim above and not an error. And `git ls-files` and `git grep` operate on the index, so
each command reports what is *tracked* — which is the intended scope, since the restored `packages/`
payloads and every `bin`/`obj` directory are excluded from the counts by construction rather than by a
filter that could be forgotten.

---

## 9. Where each blocker is consumed

| Blocker | Consumed by |
| --- | --- |
| F-12-01 SQL Server Compact 4.0 | 03 (MVC 3 is not a migration source), 10 (topology), 02 §4.1 (undeclared dependency) |
| F-12-02 Bundling and its eleven call sites | 05 (static-asset strategy), 06 (no build-time asset toolchain) |
| F-12-03 `IAppBuilder` and the OWIN startup attribute | 05 (composition root, authentication policy), 02 (ten dropped packages) |
| F-12-04 `HttpApplication` and `Global.asax` | 05 (per-responsibility disposition) |
| F-12-05 `HandleErrorAttribute` | 05 (exception middleware), 06 (telemetry the policy needs) |
| F-12-06 `HandleErrorInfo` view model | 05 (error model and error route) |
| F-12-07 `TryUpdateModel` and `[Bind]`, ten properties | 05 (input model and its tests), 09 §6.4 (over-posting) |
| F-12-08 Three `HttpNotFound()` calls | 05 |
| F-12-09 `ChallengeResult` | 05 §8.3 (**decided**: the provider-driven flow is deleted, the linked-login half is retained as the view component of 05 §8.2, its rows migrated by 05 §5.5), 09 §6.11 (disabled surface) |
| F-12-10 `BlockViewHandler` and the Razor host | 05 (`_ViewImports`), 06 (no IIS integrated pipeline) |
| F-12-11 `.axd` ignore route | 05 (routing) |
| F-12-12 Assembly metadata | 04 (project-format conversion) |
| F-12-13 Windows-auth connection strings, file attachment | 06 (identity model, secret delivery), 05 (configuration), 11 §3.3–§3.4 |
| F-12-14 MVC 4's committed build configuration | 10 (diagnosis), 08 §8.4 (severity), 07 (baseline availability) |
| F-12-15 Lazy loading — nine sites | 05 (per-site resolution), 08 §5.2 (the layout-level query debt) |
| F-12-16 JSON camelCase | 05 (scoped fix and token transport), 02 (Newtonsoft's outcome) |
| F-12-17 Path casing | 06 (the hosting option it gates), 11 §3.7 (finding of record) |
| F-12-18 4.8-versus-4.5 quirks mode | 05 (behavioural baseline), 07 (baseline integrity as a risk) |
| F-12-19 Connection strings by convention | 05 (`DbContextOptions` and registration), 01 §6.3 |
| F-12-20 `Dispose` overrides | 05 (dependency-injection transition) |
| F-12-21 Identity schema unknowable; extraction is a gate | 05 (Identity migration), 03 (gate placement), 09 §8.2 (the sequencing collision) |
| F-12-22 No usable schema baseline | 04 (**must not** use either script), 05 (extraction and diff), 08 §6.3 |
| F-12-23 MVC child actions — nine declarations, ten call sites | 05 (three view components, one per MVC 5 declaration — the conversion is 05's), 08 §5.2 (the per-page fan-out the two layout renders cause), 08 §8.1 (why the call sites are invisible to today's build) |
| P-12-01 to P-12-04 Favourable findings | 07 (scope and effort), 05 (context, query and async transitions) |

### 9.1 The reverse direction — the two `09` rows this document discharges

The table above runs outward. [09 §8.3](09-security-assessment.md) requires the other direction to
terminate too: its register's Consumers column names this document for two rows, and a reader arriving
from that column has to land on a **named item here**, which means this document has to print the
`F-09-nn` identifier rather than merely say the right thing about the right construct. Both rows are
discharged by an entry that already exists above, and neither needed a new one:

| Row in `09`'s register | What the row names | Discharged here by | Check from the other end |
| --- | --- | --- | --- |
| **F-09-22** — High, MVC 3 only, [09 §5.7](09-security-assessment.md) | MVC 3's exclude-list form of the class-level attribute, and the corrective order total written through a `DbContext` that is not the one that commits | [F-12-07](#f-12-07--the-synchronous-tryupdatemodel-and-the-class-level-bind-attribute), which classifies the construct in **all three** editions: the synchronous call is the compile-time removal, the attribute survives in the target but is deliberately not kept, and the replacement input model carries **neither** the include form nor MVC 3's exclude form forward | 09's Consumers cell for the row reads `05, 12` |
| **F-09-28** — Medium, MVC 5 and MVC 4, [09 §6.4](09-security-assessment.md) | One class-level attribute on the persistence entity as the entire over-posting control at checkout, with the promo code bypassing the model altogether | [F-12-07](#f-12-07--the-synchronous-tryupdatemodel-and-the-class-level-bind-attribute), specifically its ten-versus-nine property finding and the three reasons the attribute is replaced rather than rewritten | 09's Consumers cell for the row reads `05, 12` |

**Only the portability half is discharged here, and the boundary is worth stating because these two rows
straddle it.** The security consequence of either row is [09](09-security-assessment.md)'s and the target
design is [05](05-aspnet-core-migration-approach.md)'s ([§1.5](#15-what-this-document-does-not-own)). In
particular F-09-22's cross-context corrective write is a correctness-and-security defect in MVC 3, not a
construct without a successor, so it is cited above and **not** reclassified as an `F-12-nn` blocker.

**Four other `F-09` identifiers appear in this document and none of them is a closure.** They are
attributions to the owning deliverable, printed so a reader can find the analysis rather than to claim it:
F-09-08 at [F-12-16](#f-12-16--json-property-naming-flips-to-camelcase)'s untokenized request side, and
F-09-03, F-09-34 and F-09-05 at
[F-12-21](#f-12-21--the-identity-schema-is-not-knowable-from-the-repository)'s schema conclusion and its
extraction-before-removal ordering. None of those four rows names this document in its Consumers cell, and
this document claims none of them — [09 §8.3](09-security-assessment.md) is explicit that a consumer may
not acquire a row by citing it, and that rule cuts this way as much as the other.

---

*Deliverable 12 of 13. Consumes deliverables [09](09-security-assessment.md),
[10](10-build-and-deployment-requirements.md) and [11](11-cloud-readiness-assessment.md); feeds the three
strategy documents [04](04-dotnet8-migration-strategy.md), [05](05-aspnet-core-migration-approach.md) and
[06](06-azure-hosting-recommendations.md). Index and requirement map: [README](README.md). No user rules
were provided for this project. Every claim above carries an inline file location, and every count is
reproducible by the command stated beside it apart from the two figures §1.3 marks as measured before the
removal they describe. No tracked file outside `docs/modernization/` was created, modified or deleted in
its production — and the restore and build output that its production did write, together with the
verification of its removal, is recorded at [§1.3](#13-the-no-modification-constraint).*
