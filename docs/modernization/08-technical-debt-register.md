# 08 — Technical Debt Register

## 1. Purpose, scope and authoring contract

### 1.1 What this document is

This is the technical debt register for the MvcMusicStore repository: a categorized, quantified inventory of the debt carried by the three shipped editions — `src/MVC3/MvcMusicStore-Completed`, `src/MVC4/MvcMusicStore` and `src/MVC5/MvcMusicStore` — together with the tutorial payload under `src/MVC3/MvcMusicStore-Assets` and the repository-level artifacts that sit outside any edition.

It answers the **Technical debt** analysis requirement of the assessment. It is a supporting assessment record: it consumes deliverable 01 (architecture overview) for structural facts and deliverable 02 (dependency inventory) for package facts, and it feeds deliverable 07, whose effort model is built on the quantities recorded here. Every entry carries a **severity**, a **remediation** and an **owner**, and every quantity is re-derivable from the file or the command cited beside it.

Three properties are load-bearing, because a register that fails any of them cannot be estimated against:

1. **Every number states its counting method.** Two methods exist and they differ by roughly ten percent on this codebase. Section 2 establishes the rule; every figure below is labelled.
2. **Every claim states which editions it holds in.** The three editions are not one application. A finding that holds in MVC 5 and MVC 4 but not MVC 3 says so, every time.
3. **Nothing here is a repair.** Remediation is a recommendation with an owner. No repository file was modified in producing this document — section 1.3.

### 1.2 What this document is not

Debt intersects nearly every other deliverable, so the boundary is drawn explicitly. Where a fact recorded here has a consequence owned elsewhere, this document records the debt and **cross-references** rather than restating the decision, per the one-fact-one-owner rule. Restating a decision in different words reads downstream as a second decision.

| Not owned here | Owner | What this document does instead |
| --- | --- | --- |
| Build outcomes per edition, toolchain prerequisites, database components | **10** | Records that the build debt exists, with severity and owner; does not restate the diagnosis or the verified build result |
| The effort model — units, bands, assumptions, confidence | **07** | Supplies the quantities and states which are estimation-safe (section 12) |
| Target framework, SDK band, project-format conversion, per-package outcomes | **04** | Records the debt that makes the conversion harder; names no target version |
| Hosting target, deployment model, path-casing audit, key-ring location | **06** | Records the casing evidence it found by accident (section 10.7) and routes it |
| Security posture, policy, PII and disclosure analysis | **09** | Records the credential, the anti-forgery gap and the exception disclosure as debt entries and routes the analysis |
| The forward anti-forgery policy, the `AddToCart` verb change, DI design, Identity migration | **05** | Records today's coverage; proposes no policy |
| No-successor constructs and differing-default successors | **12** | Records the debt; does not classify migration blockers |
| Workstream sequencing and gates | **03** | Names the owner of each remediation; sequences nothing |

Two further exclusions. This register does not rank debt by remediation cost — that is an effort judgement and belongs to 07. And it does not propose an order of repair: severity is a statement about consequence, not a queue.

### 1.3 The no-modification constraint — and why this deliverable is the one most tempted to break it

The user directed *"Do not make code changes initially"*, and the attached environment setup instructions independently restate the same gate: *"Do not modify code until assessment and modernization plan are approved."* Two inputs agreeing on it is why the boundary extends even to the defects catalogued below.

This is the deliverable that finds the fixable things. Every item in it stays exactly as it is: the stale fourth solution file, the duplicated schema scripts, the fourteen committed database binaries, the two committed `packages/` trees, the three IDE user-state files, the plaintext administrator credential, the unprotected state-changing POSTs. Each is documented; none is repaired. The evidence that the constraint held is a clean working tree apart from this documentation tree:

```bash
git status --porcelain          # -> only new files under docs/modernization/
```

All verification for this document was read-only. The one experiment that needed to write anything — the `core.ignorecase` probe of section 10.7 — was run in a throwaway repository created outside the checkout and deleted afterwards, and is flagged where it appears.

### 1.4 Authoring contract, and the absence of user rules

**No user-specified rules were provided for this project.** `review_rules` returns exactly *"No user rules provided."*, re-verified while authoring this document. There is consequently no rule to name, summarize or cite, and no file forced into scope by one. The absence is not licence to lower the bar: this register is held to enterprise-standard best practice, expressed as the four contracts below.

- **Citation.** Every claim about the existing system carries an inline `[<path>:<locator>]` citation at the point of the claim, with a repository-relative path that resolves in the checkout. There is no trailing reference list, because a citation collected at the end cannot be checked against the sentence it supports.
- **Reproducibility.** A claim that ranges over the repository — a count, a total, an absence — has no single line to point at, so its evidence is the command that produces it, stated adjacent to the claim and collected once more in section 14. Byte totals are summed with `awk`; `bc` is not installed on the verification host.
- **Primacy of repository evidence.** Where this document and any other input disagree, the repository governs. Technical Specification §3.3 is cited only as a secondary cross-reference alongside repository evidence, never instead of it. Two places where a stated figure did not reproduce exactly are recorded openly, in sections 3.4 and 10.7, with the measured value and the command that produces it.
- **One fact, one owner.** Section 1.2 is the boundary; section 13 is the routing map.

Commands are given in canonical POSIX form. On the Windows verification host they were run through the bundled Git-for-Windows `bash` from the repository root, and every value shown after `# ->` is the value observed there.

### 1.5 How to read an entry

Each entry has an identifier (`F-08-nn`), a severity, the evidence, a remediation and an owner.

**Severity is a statement about consequence, judged against two questions: what it costs while the application keeps running as it is, and what it costs during the migration.** It is not a priority and not an effort estimate.

| Severity | Meaning |
| --- | --- |
| **Critical** | Can destroy data, or leaves a failure with no diagnostic trail. Consequence is unbounded and already live. |
| **High** | Exploitable, or materially enlarges the migration; a correct port cannot be produced without deciding it. |
| **Medium** | Degrades quality, safety or reproducibility; the migration proceeds without it but carries the weakness forward. |
| **Low** | Repository quality, clarity or weight. No migration impact. |

**Owner** names the role or workstream accountable for the remediation decision, not the person who writes the code. The owners used are: *security*, *performance*, *the port* (the application migration workstream), *the migration workstream* (composition and startup), *the data workstream*, *operations and platform*, and *the repository owner*.

---

## 2. Two counting methods, established here

### 2.1 The rule

Two metrics appear in this assessment and they answer different questions. This section establishes the rule; deliverable 07 depends on it, and deliverable 01 §2.4 states the same rule for the figures it carries.

| Metric | Definition | What it is for | Canonical command |
| --- | --- | --- | --- |
| **Physical lines and diff counts** | `wc -l` on a file; `diff a b \| grep -c '^[<>]'` between two files | **Duplication.** It is what a file comparison produces, and comparison is the only way to establish duplication. | `diff a b \| grep -c '^[<>]'` |
| **Non-blank lines, excluding `Properties/AssemblyInfo.cs`** | Lines that are not empty and not whitespace-only, counted per file and summed; assembly metadata excluded because it is generated boilerplate that the target expresses as MSBuild properties | **Effort sizing.** It is the volume of code a human has to read, decide about and port. | `grep -cve '^[[:space:]]*$' <file>` |

**The two are never mixed in one sentence, and every number in this document is labelled with its method.** They differ by roughly ten percent on this codebase: MVC 5's `AccountController.cs` is 422 physical lines and 382 non-blank lines, a 9.5 percent spread, and the same spread recurs file by file.

### 2.2 The one sentence that prevents the likeliest downstream error

Deliverable 07 estimates the authentication rewrite. Three numbers for MVC 5's `AccountController.cs` appear in this assessment, and only one of them is an estimation basis:

> **382 is the figure deliverable 07 estimates against** — non-blank lines, the sizing metric [src/MVC5/MvcMusicStore/Controllers/AccountController.cs]. **The 422 and 426 quoted in section 3.3 are physical line counts and belong to the duplication comparison only.** They are not sizing figures and must not be used as one.

The same discipline applies to every other pair below: a figure introduced under "diff" or "physical" never re-enters a sizing sentence.

---

## 3. Duplication — measured

Duplication is the single largest quantified item in this register, and both failure modes around it are real. Presenting 5,565 non-blank lines carrying three near-copies of one application as "some copy-paste" understates it. Extending the measured MVC 4 / MVC 5 equivalence to MVC 3 because it would make the story simpler overstates it — and is specifically unsupported by the repository. Section 3.4 draws the bound.

All figures in this section are the **duplication metric**: physical lines and diff-line counts.

### 3.1 MVC 4 and MVC 5: five of six controllers are byte-identical

```bash
for c in Account Checkout Home ShoppingCart Store StoreManager; do
  a=src/MVC4/MvcMusicStore/Controllers/${c}Controller.cs
  b=src/MVC5/MvcMusicStore/Controllers/${c}Controller.cs
  echo "$c $(diff "$a" "$b" | grep -c '^[<>]') $(wc -l < "$a") $(wc -l < "$b")"
  cmp -s "$a" "$b" && echo "  byte-identical"
done
```

| Controller | Diff lines | MVC 4 physical | MVC 5 physical | `cmp` |
| --- | --- | --- | --- | --- |
| `CheckoutController.cs` | 0 | 84 | 84 | byte-identical |
| `HomeController.cs` | 0 | 34 | 34 | byte-identical |
| `ShoppingCartController.cs` | 0 | 101 | 101 | byte-identical |
| `StoreController.cs` | 0 | 56 | 56 | byte-identical |
| `StoreManagerController.cs` | 0 | 130 | 130 | byte-identical |
| `AccountController.cs` | **397** | **426** | **422** | differs |

Byte-identical is the strongest form of the claim: not "similar", not "equivalent after formatting" — `cmp` exits 0 with no output on five of the six pairs. Any defect in those 405 physical lines exists twice, and any fix to one copy leaves the other wrong.

### 3.2 Core model files: eight of nine identical

Comparing the nine core model files present in both editions — `Album`, `Artist`, `Cart`, `Genre`, `MusicStoreEntities`, `Order`, `OrderDetail`, `SampleData`, `ShoppingCart`:

```bash
for m in Album Artist Cart Genre MusicStoreEntities Order OrderDetail SampleData ShoppingCart; do
  a=src/MVC4/MvcMusicStore/Models/$m.cs; b=src/MVC5/MvcMusicStore/Models/$m.cs
  printf '%s %s\n' "$m" "$(diff "$a" "$b" | grep -c '^[<>]')"
done                                    # -> Album 2, the other eight 0
```

Eight are byte-identical; `Album.cs` shows **2 diff lines**. Both editions' `ViewModels` folders hold the same two files, `ShoppingCartViewModel.cs` and `ShoppingCartRemoveViewModel.cs`, and both pairs are byte-identical. The 826-physical-line seed [src/MVC5/MvcMusicStore/Models/SampleData.cs] is among the byte-identical files, so the largest single file in each edition is a verbatim copy of the other.

### 3.3 The one materially divergent file, in both metrics

`AccountController.cs` is the only file that differs materially between MVC 4 and MVC 5, and the two duplication metrics agree on it:

```bash
A=src/MVC4/MvcMusicStore/Controllers/AccountController.cs
B=src/MVC5/MvcMusicStore/Controllers/AccountController.cs
diff "$A" "$B" | grep -c '^[<>]'                    # -> 397
git diff --no-index --numstat "$A" "$B"             # -> 197  200  (sum 397)
```

426 physical lines in MVC 4 against 422 in MVC 5, with 397 differing. Every one of those figures is the duplication metric. The authentication stacks behind the divergence — SimpleMembership in MVC 4, ASP.NET Identity over OWIN in MVC 5 — are described in deliverable 01 §8; this register records only that the divergence is confined to this file and its supporting startup composition and account view models.

**So the measured statement is: MVC 4 and MVC 5 are one application with two authentication stacks.** That is the finding, and it is what justifies triaging MVC 5 as the sole migration source while retaining the other editions as references. The triage decision itself is not made here.

### 3.4 The bound: this claim does not extend to MVC 3

MVC 3 is a **second architecture**, not an older copy of the same one, and treating it as a third near-copy would be a measurement error with a downstream cost — deliverable 07 would size it by analogy and be wrong.

The diff counts alone look reassuring. Against MVC 5, MVC 3's five shared controllers differ by 7 to 35 diff lines each:

```bash
for c in Home Checkout Store ShoppingCart StoreManager Account; do
  a=src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/${c}Controller.cs
  b=src/MVC5/MvcMusicStore/Controllers/${c}Controller.cs
  printf '%s %s\n' "$c" "$(diff "$a" "$b" | grep -c '^[<>]')"
done      # -> Home 7, Checkout 12, Store 25, ShoppingCart 33, StoreManager 35, Account 414
```

**Two of MVC 3's divergences are structural, and neither is visible in a diff count.**

**First, MVC 3's cart owns its own context and commits internally.** `ShoppingCart` declares its own instance — `MusicStoreEntities storeDB = new MusicStoreEntities();` [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Models/ShoppingCart.cs:11] — and calls `SaveChanges()` at **five** points inside the cart itself [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Models/ShoppingCart.cs:57], [:82], [:98], [:156], [:197]. Its `GetCart` overloads accordingly take **no context parameter** [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Models/ShoppingCart.cs:17], [:25]. MVC 5's cart is the opposite design: it contains **zero** `SaveChanges()` calls, and both its overloads require the caller's context [src/MVC5/MvcMusicStore/Models/ShoppingCart.cs:21], [:29], so the caller owns the unit of work.

```bash
grep -c 'SaveChanges' src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Models/ShoppingCart.cs   # -> 5
grep -c 'SaveChanges' src/MVC5/MvcMusicStore/Models/ShoppingCart.cs                           # -> 0
```

Deliverable 01 §7 records the two unit-of-work models as **distinct architectures** rather than a refactoring of one another, and that is the reading this register adopts.

**Second, MVC 3 does not present the same catalog.** Its seed differs from MVC 5's by **668 added and 272 removed** lines, and the difference is content, not formatting:

```bash
A=src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Models/SampleData.cs
B=src/MVC5/MvcMusicStore/Models/SampleData.cs
diff "$A" "$B" | grep -c '^>'      # -> 668 added in MVC 5
diff "$A" "$B" | grep -c '^<'      # -> 272 present only in MVC 3
for p in 'new Genre' 'new Artist' 'new Album'; do
  printf '%s: mvc3=%s mvc5=%s\n' "$p" "$(grep -c "$p" "$A")" "$(grep -c "$p" "$B")"
done                               # -> Genre 10/15, Artist 149/303, Album 246/462
```

**The correct statement, therefore: between MVC 4 and MVC 5 only the authentication layer differs materially, while MVC 3 additionally differs in its cart transaction model and in its catalog content.** Deliverable 01 §10.2 adds three further MVC 3 divergences — its persistence engine, its startup composition and its capability coverage — and concludes the same thing: no claim generalizes from the MVC 4 / MVC 5 equivalence to MVC 3.

**A measurement note, recorded because reproducibility is this register's contract.** The Agent Action Plan §0.6.5 quotes MVC 3's `AccountController` as differing by 412 lines. Both figures are correct under their own command and neither is a contradiction: **414** is `diff | grep -c '^[<>]'`, the diff-line metric used throughout this section, and **412** is `git diff --no-index --numstat`, which reports 314 added and 98 removed. `diff -w` and `diff -b` give 400. This document uses the 414 form so that one command produces every figure in the tables above.

### 3.5 The entry

| | |
| --- | --- |
| **F-08-01** | **Triplication of one application across three editions, with byte-identical copies between two of them** |
| **Severity** | **High** — it does not break anything today, but it is the largest single quantity in this register and it multiplies every other code-debt entry below by the number of editions the entry holds in. Five byte-identical controller pairs mean five defects fixed twice or, historically, once. |
| **Editions** | All three, but in two different relationships: MVC 4 and MVC 5 are near-copies; MVC 3 is a second architecture (section 3.4) |
| **Evidence** | Sections 3.1–3.4: 5 of 6 controllers and 8 of 9 core models byte-identical between MVC 4 and MVC 5; MVC 3 divergent in transaction model and catalog content |
| **Remediation** | Do **not** remediate by deleting or merging editions. Retain MVC 4 and MVC 3 read-only as the behavioural baseline the port is validated against, and let the duplication end naturally when one target application replaces the migration source. Deliverable 07 must size MVC 3 from its own measurements, never by analogy with MVC 4 or MVC 5. |
| **Owner** | The port (retention decision); deliverable 07 (sizing discipline) |

---

## 4. Sizing — one method throughout

Every figure in this section is the **sizing metric**: non-blank lines, excluding `Properties/AssemblyInfo.cs`.

### 4.1 Per-edition totals

```bash
for e in src/MVC3/MvcMusicStore-Completed/MvcMusicStore src/MVC4/MvcMusicStore src/MVC5/MvcMusicStore; do
  files=$(git ls-files "$e/*.cs" | grep -v '/packages/' | grep -v 'Properties/AssemblyInfo.cs')
  printf '%s files=%s nonblank=%s\n' "$e" "$(echo "$files" | wc -l)" \
    "$(echo "$files" | xargs -d '\n' grep -cve '^[[:space:]]*$' | awk -F: '{s+=$NF} END {print s}')"
done
```

| Edition | Files counted | Non-blank lines |
| --- | --- | --- |
| MVC 3-Completed | 19 | 1,326 |
| MVC 4 | 26 | 2,142 |
| MVC 5 | 26 | 2,097 |
| **Total** | **71** | **5,565** |

These are the same figures deliverable 01 §2.4 carries, from the same command.

### 4.2 The decomposition deliverable 07 estimates against

Within MVC 5 — the migration source — the 2,097 non-blank lines decompose into three parts, and the shape of the estimate depends entirely on which part a workstream touches:

```bash
grep -cve '^[[:space:]]*$' src/MVC5/MvcMusicStore/Controllers/AccountController.cs   # -> 382
grep -cve '^[[:space:]]*$' src/MVC5/MvcMusicStore/Models/SampleData.cs              # -> 820
# remainder: 2097 - 382 - 820                                                        # -> 895
```

| Part | Non-blank lines | Share of 2,097 | Character of the work |
| --- | --- | --- | --- |
| `AccountController.cs` | **382** | ~18% | The authentication rewrite. No direct successor for its stack; the one genuinely divergent component of section 3.3. |
| `SampleData.cs` | **820** | ~39% | Hardcoded catalog seed — 15 genres, 303 artists and 462 albums, counted in section 3.4. Bulk data expressed as code, not logic to port. |
| Everything else | **895** | ~43% | Entities, view models, cart, checkout, catalog and administration controllers, startup composition. Straightforward EF and LINQ. |

Two consequences for deliverable 07, both stated here because this register owns the counting and 07 owns the estimate:

- **The authentication rewrite is roughly 18 percent of the migration source by the sizing metric**, and it is the part with no line-for-line successor.
- **A further 39 percent is seed data**, so a headline "2,097 lines to port" overstates the logic by a wide margin: the ordinary application code outside the account controller and the seed is **895 non-blank lines**. Deliverable 07 should estimate the seed as a data-handling decision, not as 820 lines of porting.

Deliverable 01 §2.4 records the 382 figure identically. The 422 and 426 physical counts of section 3.3 remain duplication figures and appear nowhere in this section.

---

## 5. Code debt

Every entry in this section was read in MVC 5 first, because MVC 5 is the migration source, and then checked in the other two editions — so each entry states the editions it holds in rather than describing "the application".

### 5.1 F-08-02 — Redundant database-initializer registration (MVC 5)

| | |
| --- | --- |
| **F-08-02** | **Two files both register the EF database initializer** |
| **Severity** | **Low** as a defect, but recorded because it is a real signal about ownership of startup |
| **Editions** | MVC 5 only. MVC 4 registers no EF initializer of this kind, and MVC 3 has no `App_Start` folder at all. |

`Database.SetInitializer(new SampleData())` is called twice during startup: once from the ASP.NET application object [src/MVC5/MvcMusicStore/Global.asax.cs:20] and once from the OWIN startup partial [src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:16].

**What this is not.** `SetInitializer<TContext>` **sets** the strategy for a context type; it does not add to a list. The second call replaces the first, and exactly one initialization strategy is in force at runtime. This is **duplicated startup configuration, not a doubled destructive path** — the destructive behaviour is F-08-10's, and it would occur exactly once either way. Overstating this entry as "the database is initialized twice" would misrepresent the runtime.

**Why it is nonetheless worth an entry.** Two files each believe they own database bootstrapping, and the repository's own documentation records the duplication as though it were intentional: the seed class is described as "configured as the database initializer in `Global.asax.cs` and `App_Start/Startup.App.cs`" [src/MVC5/README.md:31]. During the port, the two composition roots collapse into one, and a reader who trusts either file alone will conclude something different about who owns schema lifecycle. Deliverable 01 §3.4 records the same fact structurally.

| | |
| --- | --- |
| **Remediation** | One registration in one place. Under the target composition the question disappears with the two entry points, so the remediation is to make the ownership explicit while porting rather than to patch the current file. |
| **Owner** | The migration workstream (startup composition) |

### 5.2 F-08-03 — Per-page query fan-out from the shared layout (all three editions)

| | |
| --- | --- |
| **F-08-03** | **Two uncached queries, one of them a nested aggregate, execute on every page render** |
| **Severity** | **High** — the cost is paid by every request to every page, and one of the two queries aggregates across three tables |
| **Editions** | MVC 5 (2 child actions in the layout), MVC 4 and MVC 3 (the same layout-level composition; deliverable 01 §5.3 and §10.3 record 4 and 2 child actions respectively) |

Both child actions are invoked from the shared layout, so they run for every view that uses it — which is every view, since `_ViewStart.cshtml` sets the layout globally: `@Html.Action("GenreMenu", "Store")` [src/MVC5/MvcMusicStore/Views/Shared/_Layout.cshtml:25] and `@Html.Action("CartSummary", "ShoppingCart")` [src/MVC5/MvcMusicStore/Views/Shared/_Layout.cshtml:26].

- **`GenreMenu` is a nested aggregate with no cache.** It orders genres by a `Sum` over each genre's albums' order-detail quantities and takes nine [src/MVC5/MvcMusicStore/Controllers/StoreController.cs:46-52]. The ordering key is derived from the entire order history, and nothing memoizes it.
- **`CartSummary` reads and materializes the cart on every page.** It loads the cart, projects and orders the titles, then enumerates the sequence twice — once for `Count()` and once for `Distinct()` [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:89-96].

| | |
| --- | --- |
| **Remediation** | Cache the genre aggregate — its inputs change only when orders are placed — and scope the cart read so it is not recomputed for pages that do not display it. Both are behaviour-preserving. The mechanism by which the child actions become view components is deliverable 05's; this entry is about the query cost, which survives that conversion unless it is addressed deliberately. |
| **Owner** | Performance |

### 5.3 F-08-04 — Unbounded result sets (all three editions)

| | |
| --- | --- |
| **F-08-04** | **Two list actions materialize an entire table with no paging or projection** |
| **Severity** | **Medium** — bounded today by the seeded catalog size, unbounded in principle |
| **Editions** | All three; the code is byte-identical between MVC 4 and MVC 5 per section 3.1 |

`StoreController.Index` materializes every genre [src/MVC5/MvcMusicStore/Controllers/StoreController.cs:18]. The administration list does the same for albums, with two eager `Include`s and an `OrderBy`, then `ToList()` [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:22-24] — 462 albums as seeded, each with its genre and artist loaded.

| | |
| --- | --- |
| **Remediation** | Paging or projection at both sites. The administration list is the one that grows with real use. |
| **Owner** | The port |

### 5.4 F-08-05 — Unchecked query results, unevenly distributed (all three editions)

| | |
| --- | --- |
| **F-08-05** | **`Single` on unvalidated input, and one `Find` result dereferenced without a null check** |
| **Severity** | **Medium** — the failure mode is an unhandled exception on a crafted or stale request, and section 7.1 establishes that nothing records it |
| **Editions** | All three |

The distribution matters, because most of this code is careful and an entry that flattened it would be unfair to the codebase:

- **`.Single(g => g.Name == genre)` throws on an unknown or duplicated genre** [src/MVC5/MvcMusicStore/Controllers/StoreController.cs:31], reached from a query-string value with no validation.
- **The cart paths use `.Single(...)` the same way** — on an album id [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:38] and on a cart record id supplied by the AJAX caller [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:62].
- **In the administration controller, three of four `Albums.Find(id)` calls are null-checked** and return `HttpNotFound()` [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:32], [:73], [:105]. **The exception is `DeleteConfirmed`**, which passes the result of `Find` straight into `Albums.Remove` [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:119-120] — one missing check in a controller that otherwise gets it right three times.

| | |
| --- | --- |
| **Remediation** | `SingleOrDefault` with an explicit not-found result at the three `Single` sites, and one null check in `DeleteConfirmed`. Four small changes, all behaviour-preserving for valid input. |
| **Owner** | The port |

### 5.5 F-08-06 — Anti-forgery validation covers one controller of the four that need it

| | |
| --- | --- |
| **F-08-06** | **Five state-changing POST actions accept requests with no anti-forgery validation, and one state-changing action is a `GET`** |
| **Severity** | **High** — cross-site request forgery against album administration, cart removal and order placement |
| **Editions** | MVC 5 and MVC 4 both leave 5 of their state-changing POSTs unvalidated; **MVC 3 validates nothing anywhere** |

**MVC 5, measured.** Thirteen POST actions exist across the six controllers. All eight in `AccountController` carry `[ValidateAntiForgeryToken]` [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:55], [:88], [:113], [:147], [:199], [:236], [:264], [:301]. None of the other five does:

| Unvalidated state-changing POST | Locator | Effect |
| --- | --- | --- |
| `StoreManagerController.Create` | [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:53] | Inserts an album |
| `StoreManagerController.Edit` | [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:86] | Updates an album |
| `StoreManagerController.DeleteConfirmed` | [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:116] | Deletes an album |
| `ShoppingCartController.RemoveFromCart` | [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:54] | Removes a cart line and commits |
| `CheckoutController.AddressAndPayment` | [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:25] | Writes an order and empties the cart |

**One state-changing action is a `GET` and no anti-forgery policy can cover it.** `AddToCart` declares no verb attribute, loads the album [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:37-38], mutates the cart [:43] and commits [:45] before redirecting [:48] — the whole action spanning [src/MVC5/MvcMusicStore/Controllers/ShoppingCartController.cs:33-49]. Token emission is not the gap: ten MVC 5 views emit `@Html.AntiForgeryToken()` (`git ls-files 'src/MVC5/*.cshtml' | xargs grep -l AntiForgeryToken | wc -l` → `10`). **Emission is not validation.**

**Per edition, so the finding is not over-generalized:**

The pathspec is the **application root** of each edition, not the edition folder, because `src/MVC3` also contains the tutorial payload — see the note below the table:

```bash
for e in src/MVC3/MvcMusicStore-Completed/MvcMusicStore src/MVC4/MvcMusicStore src/MVC5/MvcMusicStore; do
  printf '%s POSTs=%s tokens=%s views_emitting=%s\n' "$e" \
    "$(git ls-files "$e/*Controller.cs" | grep -v /packages/ | xargs grep -h 'HttpPost' | wc -l)" \
    "$(git ls-files "$e/*.cs"           | grep -v /packages/ | xargs grep -h 'ValidateAntiForgeryToken' | wc -l)" \
    "$(git ls-files "$e/*.cshtml"       | xargs grep -l 'AntiForgeryToken' | wc -l)"
done
```

| Edition | POST actions | `[ValidateAntiForgeryToken]` | Views emitting a token | Unvalidated state-changing POSTs |
| --- | --- | --- | --- | --- |
| MVC 5 | 13 | 8, all in `AccountController` | 10 | **5** |
| MVC 4 | 12 | 7, all in `AccountController` | 8 | **5** |
| MVC 3-Completed | 8 | **0** | **0** | **8** |

MVC 3 is the worst case and the reason this entry never says "the application": it validates nothing and emits nothing, so all eight of its POST actions — three account, one checkout, one cart, three administration — are exposed.

**Scope note, because the wider pathspec gives a different number.** Widening the first pathspec to `src/MVC3/*Controller.cs` returns **11** POST actions rather than 8: the extra three are in the tutorial payload's own account controller under `src/MVC3/MvcMusicStore-Assets/Code/Controllers/`, which is duplicated scaffolding rather than part of the shipped application (deliverable 01 §2.5). Those three carry no anti-forgery attribute either — the tutorial payload contributes 0 tokens and 0 emitting views — so the finding does not change in kind, only in count. The table reports the shipped application; F-08-01 covers the duplicated scaffolding.

#### 5.5.1 Methodology trap, recorded so the census can be re-run correctly

Counting with `[HttpPost]` as a literal **undercounts**, because MVC 5's third administration POST is declared with the attribute combined on one line:

```bash
grep -c '\[HttpPost\]' src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs   # -> 2   WRONG
grep -c 'HttpPost'     src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs   # -> 3   correct
grep -n  'HttpPost'    src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs
#   53:        [HttpPost]
#   86:        [HttpPost]
#  116:        [HttpPost, ActionName("Delete")]
```

`StoreManagerController` has **three** POST actions [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:53], [:86], [:116]. A register reporting two would understate the exposed surface by one delete action.

| | |
| --- | --- |
| **Remediation** | Validation by default for every non-`GET` request, with individually justified opt-outs, and conversion of the state-changing `GET` to a protected POST. The policy, the verb change and the token transport for the formless AJAX post are deliverable 05's to specify — this entry establishes only today's measured coverage. The security analysis is deliverable 09's. |
| **Owner** | Security |

### 5.6 F-08-07 — A plaintext administrator credential ships in two editions

| | |
| --- | --- |
| **F-08-07** | **An administrator username and password are committed in cleartext application settings and consumed at startup** |
| **Severity** | **High** — a credential in version control is compromised for the life of the history, and it provisions the account that owns the administration surface |
| **Editions** | MVC 5 and MVC 4, with the same two setting keys and the same committed value. MVC 3 has no provisioning path at all — deliverable 01 §9.3 records the consequence. |

Both editions commit `DefaultAdminUsername` and `DefaultAdminPassword` as plaintext `appSettings` values — [src/MVC5/MvcMusicStore/Web.config:16-17] and [src/MVC4/MvcMusicStore/Web.config:25-26] — and both read them at startup to create the account and its role: MVC 5 through `ConfigurationManager.AppSettings` in a fire-and-forget `private async void CreateAdminUser()` [src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:21-24], MVC 4 through the equivalent SimpleMembership path in `AppConfig`. The literal values are deliberately not reproduced in this document; they are readable at the locators above.

Two aggravating properties, both structural:

- **The provisioning call is `async void`** [src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:21], so a failure inside it is unobservable — and section 7.1 establishes there is no log to observe it in.
- **The credential store itself is committed.** F-08-11 records fourteen database binaries in the repository, including all three editions' credential stores, so the provisioned account exists in tracked data as well as in tracked configuration.

| | |
| --- | --- |
| **Remediation** | Remove the credential from source entirely and provision the administrator through an operator-invoked command that takes the secret from a non-persistent channel. The mechanism, the audit record and the idempotence requirements are specified by the migration approach; the security analysis is deliverable 09's. **This register does not repair it** — see section 1.3. |
| **Owner** | Security |

### 5.7 F-08-08 — Swallowed checkout errors (MVC 5 and MVC 4)

| | |
| --- | --- |
| **F-08-08** | **The order-writing transaction is wrapped in a bare `catch` that discards the exception** |
| **Severity** | **High** — combined with section 7.1, a failed order leaves no trace anywhere |
| **Editions** | MVC 5 and MVC 4 (byte-identical file, section 3.1). MVC 3's `CheckoutController` differs by 12 diff lines and carries the same shape. |

The entire order write — add order, create order details from the cart, commit [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:44-51] — is enclosed in `catch` with **no exception variable** [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:58], which redisplays the view [:61] and records nothing. The customer sees the form again; the operator sees nothing at all.

| | |
| --- | --- |
| **Remediation** | Catch narrowly, log with correlation, and surface a distinguishable failure. The error-handling and disclosure analysis is deliverable 09's; the target pipeline is deliverable 05's. |
| **Owner** | The port, with security reviewing disclosure |

### 5.8 F-08-09 — Hand-managed context disposal (all three editions)

| | |
| --- | --- |
| **F-08-09** | **Controllers construct their own `DbContext` and dispose it in a `Dispose(bool)` override** |
| **Severity** | **Medium** — correct as written, and a defect the moment ownership of the lifetime moves |
| **Editions** | All three for `StoreManagerController`; MVC 5 additionally for the user manager |

`StoreManagerController` disposes the context it constructed [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:125-128], and MVC 5's `AccountController` disposes the `UserManager` built by its own chained constructor [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:324-331]. The construction sites they pair with are field initializers and an ad hoc instance inside a method — [src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs:15], [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:11], [src/MVC5/MvcMusicStore/Controllers/StoreController.cs:12], [src/MVC5/MvcMusicStore/Controllers/AccountController.cs:19] and [:32].

The debt is not that disposal is wrong; it is that ownership is implicit and distributed, so the overrides are correct only for as long as the controller is also the constructor. Deliverable 01 §5.4 inventories the construction sites; the injection design and the disposal consequence are deliverable 05's.

| | |
| --- | --- |
| **Remediation** | Move construction to a container and remove the overrides in the same change, never separately. |
| **Owner** | The port |

---

## 6. Data debt

### 6.1 F-08-10 — Destructive schema lifecycle (all three editions)

| | |
| --- | --- |
| **F-08-10** | **The database initializer drops and recreates the catalog database whenever the model changes** |
| **Severity** | **Critical** — it destroys orders and personally identifiable customer data, it is armed today, and its trigger is an ordinary development action |
| **Editions** | All three. The seed class declares the same base type at the same line in each: [src/MVC5/MvcMusicStore/Models/SampleData.cs:9], [src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Models/SampleData.cs:9], and MVC 4's copy, which is byte-identical to MVC 5's (section 3.2). |

`public class SampleData : DropCreateDatabaseIfModelChanges<MusicStoreEntities>` [src/MVC5/MvcMusicStore/Models/SampleData.cs:9]. Any change to an entity class that alters the model hash causes the entire catalog database — `Orders`, `OrderDetails`, `Carts` and the customer names, addresses, emails and phone numbers those orders carry — to be dropped and rebuilt from the hardcoded seed.

Two facts make this more than theoretical:

- **It is registered, not dormant.** F-08-02 records the two registration sites [src/MVC5/MvcMusicStore/Global.asax.cs:20], [src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:16].
- **The repository's own documentation teaches the destructive workflow as a fix.** The MVC 5 README instructs a reader hitting connection errors to delete the `.mdf` and `.ldf` files and rebuild, "the database will be recreated automatically" [src/MVC5/README.md:98-99]. Recreation is presented as a routine remedy, which is exactly how a production dataset gets destroyed by someone following the documentation.

The seed being rebuilt is **826 physical lines** of hardcoded C# [src/MVC5/MvcMusicStore/Models/SampleData.cs] — 820 by the sizing metric, F-08-01's largest single file and 39 percent of the migration source by section 4.2.

| | |
| --- | --- |
| **Remediation** | Replace automatic destructive initialization with versioned schema change applied at deployment time, and make any seeding an explicit, guarded, non-production action. The mechanism, the guard design and the deployment ordering are owned by deliverables 05 and 06; the migration of the existing data is a workstream deliverable 03 sequences. |
| **Owner** | The data workstream |

### 6.2 F-08-11 — Fourteen database binaries committed, 43,376,640 bytes, including three credential stores

| | |
| --- | --- |
| **F-08-11** | **Live database files — catalogs and credential stores — are tracked in version control** |
| **Severity** | **High** — tracked credential stores and customer data, permanent repository weight, and a merge hazard on any binary that is ever opened |
| **Editions** | All three |

```bash
git ls-files | grep -icE '\.(mdf|ldf)$'                                    # -> 14
git ls-files | grep -iE  '\.(mdf|ldf)$' | xargs -d '\n' stat -c '%s' \
  | awk '{s+=$1} END {print s}'                                           # -> 43376640
```

`awk` performs the sum because `bc` is not installed on the verification host.

| Location | Files | What they are |
| --- | --- | --- |
| `src/MVC3/MvcMusicStore-Assets/Data/` | 4 | The tutorial catalog pair plus `ASPNETDB.MDF` and its log — MVC 3's classic Membership credential store, and at 10,485,760 bytes the largest single file in the set |
| `src/MVC4/MvcMusicStore/App_Data/` | 6 | The catalog pair, the SimpleMembership credential pair, **and an unreferenced scratch pair** |
| `src/MVC5/MvcMusicStore/App_Data/` | 4 | The catalog pair and the ASP.NET Identity 1.0 credential pair |

**The scratch pair is debt in its own right.** `src/MVC4/MvcMusicStore/App_Data/MvcMusicStore-work.mdf` and `src/MVC4/MvcMusicStore/App_Data/MvcMusicStore_log-work.ldf` are referenced by no tracked source, configuration, project or solution file — 4,259,840 bytes of committed working copy:

```bash
git ls-files '*.cs' '*.config' '*.cshtml' '*.csproj' '*.sln' | grep -v /packages/ \
  | xargs grep -il 'MvcMusicStore-work' | wc -l                           # -> 0
```

**Ten of the fourteen are ignored by the repository's own rules and tracked anyway — and the other four are not ignored at all.** This distinction matters, and collapsing it would misstate the finding. `.gitignore:32` declares `App_Data/`, which covers the MVC 4 and MVC 5 files; the four MVC 3 files sit under `Data/`, which no rule matches. So ten are gitignored-yet-tracked — a rule added after the files were already in the index, which cannot untrack them — and four are simply tracked, with no rule ever expressing an intent to exclude them. Section 10.7 records the correct probe and the exact outputs.

| | |
| --- | --- |
| **Remediation** | Untracking these files does not remove them from history; only a history rewrite does, and that is a decision with consequences for every fork and clone. The choices are (a) accept the history and untrack going forward, or (b) rewrite. Either way, no environment should ever attach a tracked file — the setup guidance for this repository already requires serving a copy outside the checkout so SQL Server cannot write to tracked binaries, which is a mitigation, not a fix. Credential stores in history should additionally be treated as compromised, per F-08-07. |
| **Owner** | The repository owner (history decision); security (credential-store exposure) |

### 6.3 F-08-12 — Schema scripts not runnable as written, and duplicated (MVC 4)

| | |
| --- | --- |
| **F-08-12** | **Both copies of MVC 4's schema script are byte-identical and both begin with a hard-coded developer file path; MVC 5 ships no schema script at all** |
| **Severity** | **Medium** — the artifact a migration would naturally take as its schema baseline cannot be executed anywhere, and for the migration source it does not exist |
| **Editions** | MVC 4 (two unusable copies), MVC 3 (one script, under the tutorial assets), MVC 5 (**none**) |

```bash
git ls-files '*.sql'
# -> src/MVC3/MvcMusicStore-Assets/Data/MvcMusicStore-Create.sql
# -> src/MVC4/MvcMusicStore-Create.sql
# -> src/MVC4/MvcMusicStore/MvcMusicStore-Create.sql
cmp src/MVC4/MvcMusicStore-Create.sql src/MVC4/MvcMusicStore/MvcMusicStore-Create.sql
#    (exit 0, no output: byte-identical, 629 lines each)
head -1 src/MVC4/MvcMusicStore-Create.sql
```

Both copies open with `USE [C:\USERS\JON\DOCUMENTS\JON-SHARE\MVCMUSICSTORE-MVC3\MVCMUSICSTORE\MVCMUSICSTORE\APP_DATA\MVCMUSICSTORE.MDF]` [src/MVC4/MvcMusicStore-Create.sql:1], [src/MVC4/MvcMusicStore/MvcMusicStore-Create.sql:1] — a `USE` naming one developer's attached MDF by absolute path. It will not execute against another machine's LocalDB and cannot execute against a hosted SQL database, where file attachment has no analogue. Neither copy is usable as written, so "there are two" is not a redundancy benefit.

**A provenance finding rides along in the same string.** The baked path contains `MVCMUSICSTORE-MVC3` — an MVC 4 artifact carrying an MVC 3-era working folder in its first line, evidence that the script was copied forward from the older edition rather than generated from the MVC 4 model. It is therefore not merely unrunnable but of uncertain currency with respect to the schema it claims to create.

**A third property compounds both.** All three tracked `.sql` files are UTF-16 little-endian with CRLF terminators, and git classifies them as binary, so no version of any of them can be diffed or reviewed line by line in the normal way:

```bash
file -b src/MVC4/MvcMusicStore-Create.sql
# -> Unicode text, UTF-16, little-endian text, with very long lines (848), with CRLF line terminators
git ls-files --eol '*.sql'          # -> i/-text w/-text for all three: git treats them as binary
```

**MVC 5, the migration source, ships no schema script at all**, so its authoritative schema exists only inside the committed `.mdf` of F-08-11. What that implies for the migration — extraction before any data load — is owned by deliverables 05 and 12; this entry records only the state of the artifacts.

| | |
| --- | --- |
| **Remediation** | Do not adopt either MVC 4 copy as a schema baseline. De-duplicate to one script only if a script is still wanted after the schema source of truth is decided; otherwise retire both. |
| **Owner** | The data workstream |

---

## 7. Operational debt

### 7.1 F-08-13 — No observability of any kind (all three editions)

| | |
| --- | --- |
| **F-08-13** | **No logging, no tracing, no metrics, no health endpoint anywhere in the repository** |
| **Severity** | **Critical** for a hosted target, and already consequential today |
| **Editions** | All three, and the repository root |

Verified across every tracked `.cs` and `.cshtml` file outside the committed package trees. Every count is zero:

```bash
FILES=$(git ls-files '*.cs' '*.cshtml' | grep -v '/packages/')
for p in ILogger log4net NLog Serilog TraceSource 'Trace\.Write' 'System\.Diagnostics\.Trace' HealthCheck healthMonitoring; do
  printf '%-28s %s\n' "$p" "$(echo "$FILES" | xargs -d '\n' grep -l "$p" | wc -l)"
done                                                    # -> 0 for every pattern
git ls-files '*.config' | grep -v /packages/ | xargs grep -c 'healthMonitoring' | grep -v ':0' | wc -l
                                                        # -> 0  (no healthMonitoring section in any config)
```

There is no logging abstraction, no logging framework, no `TraceSource`, no ASP.NET health-monitoring configuration, no health endpoint and no metric of any kind. Error display is not observability either: the `customErrors` element appears 24 times across the six XDT transform files and **every occurrence is inside a comment block**, so no edition configures it live:

```bash
git ls-files '*.config' | grep -v /packages/ | xargs grep -h 'customErrors' | wc -l    # -> 24
# each of the six Web.Debug.config / Web.Release.config files: 4 occurrences,
# all inside <!-- ... --> template blocks (verified by an awk comment-block pass, section 14)
```

**The consequence, stated plainly: a failed checkout in production today leaves no trace.** F-08-08's bare `catch` [src/MVC5/MvcMusicStore/Controllers/CheckoutController.cs:58] discards the exception, and there is no sink it could have been written to. The customer sees the form redisplayed, the order does not exist, and no operator can discover that it happened, let alone why. The same is true of the `async void` administrator provisioning of F-08-07 [src/MVC5/MvcMusicStore/App_Start/Startup.App.cs:21].

| | |
| --- | --- |
| **Remediation** | Net-new capability, not a migration: structured logging with correlation, a health endpoint, and platform-collected telemetry. The telemetry mechanism and the platform integration are owned by deliverable 06; the workstream that introduces them is sequenced by deliverable 03. |
| **Owner** | Operations and platform |

### 7.2 F-08-14 — No continuous integration, no deployment automation, no publish artifact

| | |
| --- | --- |
| **F-08-14** | **The repository contains no pipeline definition, no publish profile, no container manifest and no build script** |
| **Severity** | **High** — nothing enforces any quality gate, and every build and deployment is a manual act performed from a developer machine |
| **Editions** | Repository-wide |

```bash
git ls-files | grep -c '^\.github/'                                    # -> 0
git ls-files | grep -ciE 'azure-pipelines|jenkinsfile|appveyor|\.travis' # -> 0
git ls-files | grep -ci 'pubxml'                                        # -> 0   (no publish profile)
git ls-files | grep -ciE 'dockerfile|docker-compose|\.ya?ml$'            # -> 0
git ls-files | grep -v /packages/ | grep -ciE '\.(ps1|sh|cmd|bat)$'      # -> 0
```

`.gitignore` anticipates publish profiles at `PublishProfiles/` [.gitignore:18] and build output at `build/` [.gitignore:29], and neither has ever existed in the tree. What build logic the repository does have lives inside the MSBuild project files and MVC 4's `.nuget/NuGet.targets`, which deliverable 02 §5 inventories and deliverable 10 owns as build requirements.

This entry compounds F-08-16 and F-08-17: warnings are not errors, no analyzer runs, and there is no pipeline that would fail if either changed. Deliverable 02 §8.2 records the same shape for dependency advisories — 43 restore warnings that no artifact retains and no gate consumes.

| | |
| --- | --- |
| **Remediation** | Net-new. Build, test, publish and a deployment-time schema step, with the provider selection an explicit gate in deliverable 03 rather than an assumption here. |
| **Owner** | Operations and platform |

### 7.3 F-08-15 — No test of any kind, repository-wide

| | |
| --- | --- |
| **F-08-15** | **There is no test project, no test file and no test-framework reference anywhere** |
| **Severity** | **Critical** for the migration — without a baseline, no behaviour-preservation claim about the port can be substantiated |
| **Editions** | Repository-wide |

```bash
git ls-files '*.cs' '*.csproj' | grep -v /packages/ \
  | xargs grep -lE 'TestClass|\[Fact\]|xunit|NUnit|Microsoft\.VisualStudio\.TestTools' | wc -l   # -> 0
git ls-files | grep -ci 'test'                                                                   # -> 0
```

Deliverable 01 §10.3 records the same absence. Its significance for this register is specific: several of the entries above and several of the differing-default successors deliverable 12 owns fail **silently** after a port — a null navigation property, a renamed JSON field, a changed model-binding call. Nothing in this repository would detect any of them.

| | |
| --- | --- |
| **Remediation** | Author the characterization suite **before** the port, against the current application, so it can serve as the baseline. Its architecture, fixtures and coverage are specified by deliverables 03 and 05, and deliverable 07 carries the absent baseline as a first-order risk. No test project is created by this assessment. |
| **Owner** | The port (authoring); deliverable 07 (risk) |

---

## 8. Build debt

### 8.1 F-08-16 — View compilation is disabled (MVC 5; the same property in all three projects)

| | |
| --- | --- |
| **F-08-16** | **Razor views are never compile-checked, so 29 views carry no build-time guarantee** |
| **Severity** | **Medium** — every view error is a runtime error, and the migration has no compile-time inventory of view breakage to work from |
| **Editions** | MVC 5 verified at the locators below; the same disabled property and gated target exist in the MVC 4 and MVC 3 projects |

`<MvcBuildViews>false</MvcBuildViews>` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:17], and the target that would compile them is gated on the property being `true` — `<Target Name="MvcBuildViews" AfterTargets="AfterBuild" Condition="'$(MvcBuildViews)'=='true'">` [src/MVC5/MvcMusicStore/MvcMusicStore.csproj:274]. The target is therefore dead as configured.

The migration consequence is concrete. Deliverable 01 §2.5 counts 29 Razor files in MVC 5, five of which name legacy types or non-portable members. A build today reports nothing about any of them, so the port cannot lean on the compiler to enumerate what breaks — it has to be read.

| | |
| --- | --- |
| **Remediation** | Do not flip the property in the legacy project (section 1.3 forbids the change, and the 2010-era target it enables is retired in the target platform). Instead rely on the target platform's built-in Razor compilation, which is on by default, and treat the first compile of the ported views as the inventory this project never had. |
| **Owner** | The port |

### 8.2 F-08-17 — No enforced static analysis (all three editions)

| | |
| --- | --- |
| **F-08-17** | **Compiler diagnostics are configured but nothing enforces them, and no analyzer, ruleset or style configuration exists** |
| **Severity** | **Medium** |
| **Editions** | All three |

**The finding is the absence of enforcement, not the absence of diagnostics** — the distinction matters, and stating it the other way would be false. Every project sets warning level 4, the highest the compiler offers, in both configurations:

```bash
git ls-files '*.csproj' | grep -v /packages/ | xargs grep -n 'WarningLevel'
# -> MVC5 :36 :44   MVC4 :33 :41   MVC3 :26 :34   (all <WarningLevel>4</WarningLevel>)
git ls-files '*.csproj' | grep -v /packages/ | xargs grep -c 'TreatWarningsAsErrors'   # -> 0 in all three
git ls-files | grep -ciE '\.ruleset$|\.editorconfig$|Directory\.Build\.props'          # -> 0
```

So warnings are produced and then ignored: no `TreatWarningsAsErrors` in any configuration of any project, no `.ruleset`, no `.editorconfig`, no `Directory.Build.props` to apply a policy centrally, no analyzer package in any manifest — deliverable 02 §8.2 verifies that last point across all three `packages.config` files — and, per F-08-14, no pipeline that could fail on a warning even if a policy existed. Nothing in the repository has ever recorded how many warnings a clean build produces.

| | |
| --- | --- |
| **Remediation** | Establish the policy on the target project: analyzers enabled, a shared style configuration, and warnings escalated in the pipeline. Escalating warnings on the legacy projects is out of scope and would be a code change. |
| **Owner** | The port, with operations and platform for enforcement |

### 8.3 F-08-18 — A committed 2012-era restore client, and no lockfile in any edition

| | |
| --- | --- |
| **F-08-18** | **Restore depends on an executable committed to source control, and transitive resolution is not reproducible** |
| **Severity** | **Medium** |
| **Editions** | MVC 4 commits the client; the missing lockfile is all three |

```bash
stat -c '%s' src/MVC4/MvcMusicStore/.nuget/NuGet.exe      # -> 630784
git ls-files 'packages.lock.json' | wc -l                 # -> 0
```

A 630,784-byte NuGet client, version `2.0.30828.5`, is tracked at [src/MVC4/MvcMusicStore/.nuget/NuGet.exe] and is what MVC 4's MSBuild-integrated restore invokes — a restore mechanism deprecated since NuGet 3. No edition has a `packages.lock.json`, so exact direct pins do not fix what resolves transitively. Deliverable 02 owns the inventory detail: §5.1 for the client and its properties, §7.1 for the absent lockfile and central version management, and §6 for the finding that no package source is configured anywhere, which is what makes the effective source set unknowable from the repository.

Two hygiene consequences are recorded elsewhere in this register rather than here: the committed restored payloads are F-08-25, and the ignore rule that was supposed to exclude this executable is F-08-28.

| | |
| --- | --- |
| **Remediation** | The target's SDK-integrated restore replaces the committed client, and lockfiles plus an explicit source configuration replace the current ambiguity. The pins, the source configuration and the lockfile decision are deliverable 04's; the build requirements are deliverable 10's. |
| **Owner** | The port, with operations and platform for the pipeline side |

### 8.4 F-08-19 — MVC 4's committed build configuration is broken, and a fourth solution file is stale

| | |
| --- | --- |
| **F-08-19** | **The MVC 4 project cannot be built from its committed configuration without command-line compensation, and one of the four solution files points at a project path that does not exist** |
| **Severity** | **High** for the affected edition |
| **Editions** | MVC 4 |

**Deliverable 10 owns the build outcomes, the diagnosis and the workarounds, and this register does not restate them.** What belongs here is that the debt exists, is platform-independent, and lives in committed files: an unconditional import and a set of package paths in [src/MVC4/MvcMusicStore/MvcMusicStore.csproj], and the stale solution at [src/MVC4/MvcMusicStore/MvcMusicStore.sln:4], whose project declaration resolves one directory too deep. The equivalent declaration in the correct solution [src/MVC4/MvcMusicStore.sln:4] carries the identical relative path from one level up, which is exactly why the deeper file is the stale one. Section 10.2 records the solution-count hygiene aspect.

| | |
| --- | --- |
| **Remediation** | Not repaired here (section 1.3), and not carried forward: the target has one solution and SDK-style projects, so the defects are retired rather than fixed. Until then, deliverable 10's documented invocation is the only way to build the edition. |
| **Owner** | Deliverable 10 (build ownership); the repository owner if the legacy edition is ever to be built without compensation |

---

## 9. Dead scaffolding

Template scaffolding that executes, or ships, while serving nothing. Each entry has a cost: startup work, deployed surface, or a reader's time spent understanding a capability that does not exist.

### 9.1 F-08-20 — Area registration with no areas (MVC 5 and MVC 4)

| | |
| --- | --- |
| **F-08-20** | **`AreaRegistration.RegisterAllAreas()` runs at every application start and discovers nothing** |
| **Severity** | **Low** — a reflection scan at startup and a misleading signal about the application's structure |
| **Editions** | MVC 5 and MVC 4 both call it; MVC 3 does not, having no `App_Start` composition at all |

`AreaRegistration.RegisterAllAreas();` is the first statement of `Application_Start` [src/MVC5/MvcMusicStore/Global.asax.cs:15]. No edition has an `Areas` folder:

```bash
git ls-files | grep -ci '/Areas/'      # -> 0
```

| | |
| --- | --- |
| **Remediation** | Do not carry the call into the target composition. Nothing else is required. |
| **Owner** | The migration workstream |

### 9.2 F-08-21 — A mapped HTTP API route with no implementation (MVC 4)

| | |
| --- | --- |
| **F-08-21** | **MVC 4 maps an API route and carries four Web API packages to serve zero controllers** |
| **Severity** | **Low** — deployed surface and dependency weight for a capability the edition does not have |
| **Editions** | MVC 4 only |

`config.Routes.MapHttpRoute(name: "DefaultApi", routeTemplate: "api/{controller}/{id}", ...)` [src/MVC4/MvcMusicStore/App_Start/WebApiConfig.cs:12-16]. There is no `ApiController` implementation anywhere in the repository:

```bash
git ls-files '*.cs' | grep -v /packages/ | xargs grep -c 'ApiController' \
  | awk -F: '{s+=$NF} END {print s}'          # -> 0
```

Deliverable 02 records the dependency side as F-02-07: four Web API packages pinned and referenced, one of them a metapackage with no assembly, serving this route. Deliverable 01 §9.3 marks the HTTP API capability absent for MVC 4 rather than implemented — the correction that keeps a mapped route from being read as a delivered feature.

| | |
| --- | --- |
| **Remediation** | Nothing to migrate: the route and its four packages are dropped. If an HTTP API is ever wanted, it is net-new work with its own approval. |
| **Owner** | The port (as a non-migration); deliverable 04 for the package disposition |

### 9.3 F-08-22 — A scaffolded, disabled external-login surface (MVC 5 and MVC 4)

| | |
| --- | --- |
| **F-08-22** | **Every external sign-in registration is commented out with empty credentials, while the packages that would serve them ship and deploy** |
| **Severity** | **Low** as dead code; it becomes **High** the moment anyone uncomments a registration without supplying and protecting real credentials |
| **Editions** | MVC 5 and MVC 4 both carry the disabled surface, in different stacks. MVC 3 has no external-login surface at all. |

**MVC 5.** `ConfigureAuth` is 37 physical lines, of which 14 are comment lines, and the four commented-out provider registrations — Microsoft Account, Twitter, Facebook, Google — span [src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:22-35], each with empty string literals where a client id and secret would go. Only two calls are live: cookie authentication [src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs:14-18] and the external sign-in cookie [:20], the latter existing solely to support the providers that are disabled.

**MVC 4.** The same shape in a different stack: `RegisterAuth` is 32 physical lines with 12 comment lines, and the four commented `OAuthWebSecurity.Register*Client` calls span [src/MVC4/MvcMusicStore/App_Start/AuthConfig.cs:17-29].

```bash
for f in src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs src/MVC4/MvcMusicStore/App_Start/AuthConfig.cs; do
  printf '%s physical=%s comment_lines=%s\n' "$f" "$(wc -l < "$f")" "$(grep -c '^[[:space:]]*//' "$f")"
done                                    # -> 37/14 and 32/12
```

The dependency cost is deliverable 02's, and it is not symmetric: MVC 5 ships **four** dormant `Microsoft.Owin.Security.*` provider packages (§3.1.2, F-02-03) and MVC 4 ships **six** DotNetOpenAuth packages plus the WebPages OAuth surface (§3.2.3, F-02-08). One package is easy to miscount here: `Microsoft.Owin.Security.OAuth` is **OAuth infrastructure, not a fifth provider** — deliverable 02 §3.1.2 states it explicitly.

A related consequence sits in the views rather than the startup files: MVC 5's account management views render an external-login removal list that can never have entries while every provider is disabled. Deliverable 05 owns whether that surface is ported or removed.

| | |
| --- | --- |
| **Remediation** | Decide, rather than port: either remove the disabled surface and its packages entirely, or enable it deliberately with credentials supplied from a secret store. Carrying it forward commented out reproduces the debt in the target. |
| **Owner** | The port, with security if the surface is ever enabled |

---

## 10. Repository hygiene

All entries in this section are **Low** severity with **no migration impact**. They are recorded because they are quantified, because they shape a newcomer's first impression of the repository, and because one of them — the root `.gitignore` — is the evidence that makes several of the higher-severity entries above *debt* rather than a deliberate choice.

### 10.1 The primary evidence: root `.gitignore`, line by line

A tracked file that the repository's own rules exclude was added before the rule existed, and `.gitignore` cannot untrack it. That asymmetry is what distinguishes debt from an intentional decision, so the root `.gitignore` [.gitignore] is cited by line as the primary evidence for the whole gitignored-yet-tracked class:

| `.gitignore` line | Pattern | What is tracked despite it |
| --- | --- | --- |
| [.gitignore:4] | `*.user` | `src/MVC4/MvcMusicStore/MvcMusicStore.csproj.user`, `src/MVC5/MvcMusicStore/MvcMusicStore.csproj.user` (F-08-27) |
| [.gitignore:8] | `*.suo` | `src/MVC4/MvcMusicStore/MvcMusicStore.v11.suo` (F-08-27) |
| [.gitignore:12] | `*.csproj.user` | The two `.csproj.user` files above, matched by a second rule as well (F-08-27) |
| [.gitignore:15] | `packages/*` | 215 files under two committed `packages/` trees (F-08-25) |
| [.gitignore:18] | `PublishProfiles/` | Nothing — no publish profile has ever existed (F-08-14) |
| [.gitignore:28] | `nuget.exe` | `src/MVC4/MvcMusicStore/.nuget/NuGet.exe` — and the match is case-dependent (F-08-28) |
| [.gitignore:29] | `build/` | Nothing — no build output is tracked |
| [.gitignore:32] | `App_Data/` | 10 of the 14 database binaries (F-08-11) |
| [.gitignore:33] | `Packages/` | The same 215 files, matched by a second rule under a different casing |

Two of these rows are findings in their own right rather than just evidence, and section 10.7 carries them.

### 10.2 F-08-23 — Four solution files for three projects, one of them stale

| | |
| --- | --- |
| **F-08-23** | **Four `.sln` files exist for three `.csproj` projects; the fourth is stale and unbuildable** |
| **Severity** | **Low** as hygiene. The build consequence is F-08-19 and is owned by deliverable 10. |
| **Editions** | MVC 4 (the stale file); the four-solutions-for-three-projects shape is repository-wide |

```bash
git ls-files '*.sln'                                     # -> 4
git ls-files '*.csproj' | grep -v /packages/             # -> 3
```

| Solution | Status |
| --- | --- |
| [src/MVC5/MvcMusicStore.sln] | Current, MVC 5 |
| [src/MVC4/MvcMusicStore.sln] | Current, MVC 4 |
| [src/MVC3/MvcMusicStore-Completed/MvcMusicStore.sln] | Current, MVC 3 |
| [src/MVC4/MvcMusicStore/MvcMusicStore.sln:4] | **Stale** — its project declaration resolves one directory too deep |

The hazard is not that the file exists but that its name and location make it the one a newcomer opens first, inside the project folder rather than beside it.

| | |
| --- | --- |
| **Remediation** | Retire the stale file when the target's single solution replaces all four. Not removed here (section 1.3). |
| **Owner** | The repository owner |

### 10.3 F-08-24 — A schema script committed twice

| | |
| --- | --- |
| **F-08-24** | **`MvcMusicStore-Create.sql` exists at two paths in MVC 4, byte-identical** |
| **Severity** | **Low** as hygiene; the usability defect the two copies share is F-08-12 (Medium) |
| **Editions** | MVC 4 only |

[src/MVC4/MvcMusicStore-Create.sql] and [src/MVC4/MvcMusicStore/MvcMusicStore-Create.sql], 629 lines each, `cmp`-identical. Two copies of an unrunnable script are two chances to adopt the wrong baseline.

| | |
| --- | --- |
| **Remediation** | De-duplicate only in conjunction with F-08-12's decision about whether either copy has a future. |
| **Owner** | The repository owner |

### 10.4 F-08-25 — 215 files of restored packages committed

| | |
| --- | --- |
| **F-08-25** | **Two editions commit their restored `packages/` trees, including 32 binaries; the third commits nothing** |
| **Severity** | **Low** for the migration; it is repository weight and a supply-chain provenance question rather than a blocker |
| **Editions** | MVC 3-Completed and MVC 4 commit payloads; MVC 5 commits none |

```bash
git ls-files | grep -c '/packages/'                                           # -> 215
git ls-files | grep '/packages/' | grep -ciE '\.(dll|exe)$'                   # -> 32
git ls-files | grep -c 'src/MVC3/MvcMusicStore-Completed/packages/'           # -> 46
git ls-files | grep -c 'src/MVC4/MvcMusicStore/packages/'                     # -> 169
```

Tracked against [.gitignore:15] and [.gitignore:33]. Deliverable 02 §7.2 owns the inventory (F-02-20) and records the asymmetry that matters more than the count: MVC 5 commits **no** packages, so the two editions cannot be prepared the same way — a fact deliverable 10 carries as a build requirement.

| | |
| --- | --- |
| **Remediation** | Untracking is a history decision, exactly as in F-08-11. The target restores from a declared source with a lockfile, so the payloads are not carried forward. |
| **Owner** | The repository owner |

### 10.5 F-08-26 — A 4,993,295-byte tutorial PDF, with distinct licensing

| | |
| --- | --- |
| **F-08-26** | **A 4.99 MB tutorial document is committed, and its licence differs from the code's** |
| **Severity** | **Low** |
| **Editions** | MVC 3 (the tutorial payload) |

```bash
stat -c '%s' 'src/MVC3/MVC Music Store - Tutorial - v3.0.pdf'      # -> 4993295
git ls-files | grep -ivE '\.(mdf|ldf)$' | xargs -d '\n' stat -c '%s %n' | sort -nr | head -2
# -> 4993295 src/MVC3/MVC Music Store - Tutorial - v3.0.pdf
# -> 1416507 src/MVC4/MvcMusicStore/packages/EntityFramework.5.0.0/EntityFramework.5.0.0.nupkg
```

[src/MVC3/MVC Music Store - Tutorial - v3.0.pdf] is the single largest non-database file in the repository, more than three times the size of the next largest. The licensing distinction is the part worth recording, because it survives any decision about the file: the code is under the Microsoft Public License while the tutorial document is under Creative Commons Attribution 3.0 [src/MVC3/readme.txt:5]. Anything derived from the document therefore carries an attribution obligation that the code does not. This document treats the PDF as repository weight and licensing evidence only; it was not read for requirements.

| | |
| --- | --- |
| **Remediation** | Retain or relocate as the repository owner prefers; if retained, keep the licence statement adjacent to it. No migration impact either way. |
| **Owner** | The repository owner |

### 10.6 F-08-27 — Three IDE user-state files committed

| | |
| --- | --- |
| **F-08-27** | **Per-developer IDE state is tracked, against three separate ignore rules** |
| **Severity** | **Low** |
| **Editions** | MVC 4 (two files) and MVC 5 (one) |

`src/MVC4/MvcMusicStore/MvcMusicStore.csproj.user`, `src/MVC5/MvcMusicStore/MvcMusicStore.csproj.user` and `src/MVC4/MvcMusicStore/MvcMusicStore.v11.suo`, excluded by [.gitignore:4], [.gitignore:8] and [.gitignore:12] and tracked anyway:

```bash
git ls-files | grep -iE '\.user$|\.suo$'      # -> the three files above
```

The `.v11.suo` is a Visual Studio 2012-era binary solution-state file — unreadable by current tooling and a merge conflict waiting to happen.

| | |
| --- | --- |
| **Remediation** | Untrack going forward; the files carry no content anyone else needs. Not removed here (section 1.3). |
| **Owner** | The repository owner |

### 10.7 F-08-28 — The ignore rules themselves are two findings

This entry exists because the register's contract is that a reader who checks one item at random finds it exact. Checking the ignore rules produced a correction and a portability finding, and both are reported.

**First, the correct probe.** `git check-ignore -v <path>` exits **1 with no output for every tracked file** in this repository — verified on `NuGet.exe`, both `App_Data` catalogs, both `.csproj.user` files, the `.suo` and a `packages/` payload — because `check-ignore` consults the index before the ignore rules. The exit code therefore says nothing about whether a rule matches. `--no-index` is the probe that answers the question, and with it every gitignored-yet-tracked claim in section 10.1 verifies:

```bash
git check-ignore --no-index -v src/MVC4/MvcMusicStore/.nuget/NuGet.exe
# -> .gitignore:28:nuget.exe        src/MVC4/MvcMusicStore/.nuget/NuGet.exe
git check-ignore --no-index -v src/MVC5/MvcMusicStore/App_Data/MvcMusicStore.mdf
# -> .gitignore:32:App_Data/        src/MVC5/MvcMusicStore/App_Data/MvcMusicStore.mdf
git check-ignore --no-index -v src/MVC4/MvcMusicStore/MvcMusicStore.v11.suo
# -> .gitignore:8:*.suo             src/MVC4/MvcMusicStore/MvcMusicStore.v11.suo
git check-ignore --no-index -v src/MVC3/MvcMusicStore-Assets/Data/ASPNETDB.MDF
# -> (exit 1, no output: no rule matches this path at all)
```

**Finding one — four of the fourteen database binaries are not ignored by any rule.** The last probe above is the interesting one. `.gitignore:32` is `App_Data/`, and MVC 3's four binaries live under `Data/`, which no pattern covers. So F-08-11 splits cleanly: **ten** files are gitignored-yet-tracked, and **four** are simply tracked, with no rule ever having expressed an intent to exclude them. A remediation that assumes one uniform cause would miss the second group.

**Finding two — the `nuget.exe` rule stops matching on a case-sensitive filesystem.** [.gitignore:28] spells the pattern in lowercase while the tracked path is `NuGet.exe` [src/MVC4/MvcMusicStore/.nuget/NuGet.exe]. Whether the pattern matches depends on `core.ignorecase`, which is `true` on this Windows checkout. Verified by a controlled experiment in a throwaway repository created **outside** the checkout with the same `.gitignore` and deleted afterwards — the one non-read-only command in this document, and it touched no repository file:

```bash
T=$(mktemp -d); mkdir -p "$T/.nuget"; cp .gitignore "$T/"; : > "$T/.nuget/NuGet.exe"
cd "$T" && git init -q .
for s in true false; do
  git config core.ignorecase $s
  git check-ignore -v .nuget/NuGet.exe; echo "core.ignorecase=$s exit=$?"
done
cd - && rm -rf "$T"
# core.ignorecase=true  -> .gitignore:28:nuget.exe  .nuget/NuGet.exe   (exit 0: ignored)
# core.ignorecase=false -> no output                                   (exit 1: NOT ignored)
```

| | |
| --- | --- |
| **F-08-28** | **The hygiene configuration is itself host-dependent: one ignore rule's effect changes with filesystem case sensitivity, and one class of committed binary is covered by no rule** |
| **Severity** | **Low** in its own right. Its value is corroborative. |
| **Editions** | Repository-wide — the root `.gitignore` and the paths it does and does not cover |
| **Evidence** | The two experiments above |
| **Remediation** | Express ignore patterns so they hold on both a case-insensitive and a case-sensitive filesystem, and add a rule that actually covers `Data/` if those binaries are meant to be excluded. |
| **Owner** | The repository owner |

**Why this matters beyond hygiene.** It is independent, mechanically reproducible evidence that this repository contains path-casing assumptions that hold on Windows and fail elsewhere. The same class of assumption exists in the application: the style bundle registers `"~/Content/site.css"` [src/MVC5/MvcMusicStore/App_Start/BundleConfig.cs:28] while the tracked file is `src/MVC5/MvcMusicStore/Content/Site.css` — a mismatch IIS resolves and a case-sensitive host does not.

```bash
grep -n 'site.css' src/MVC5/MvcMusicStore/App_Start/BundleConfig.cs   # -> 28: "~/Content/site.css"
git ls-files 'src/MVC5/MvcMusicStore/Content/*'                       # -> .../Content/Site.css
```

The hosting consequence, and the repository-wide casing audit it implies, are owned by deliverables 06 and 11. This register contributes the evidence and routes it.

---

## 11. Severity roll-up

All 28 entries, with the editions each holds in. Severity is defined in section 1.5 and is a statement about consequence, not a priority order and not an effort figure.

| # | Finding | Category | Severity | Editions | Owner |
| --- | --- | --- | --- | --- | --- |
| F-08-01 | Triplication of one application, byte-identical between two editions | Duplication | High | 3, 4, 5 | The port; 07 for sizing discipline |
| F-08-02 | Two files register the EF initializer (duplicated configuration) | Code | Low | 5 | Migration workstream |
| F-08-03 | Uncached nested aggregate and cart read on every page | Code | High | 3, 4, 5 | Performance |
| F-08-04 | Unbounded result sets in two list actions | Code | Medium | 3, 4, 5 | The port |
| F-08-05 | `Single` on unvalidated input; one unchecked `Find` | Code | Medium | 3, 4, 5 | The port |
| F-08-06 | 5 unvalidated state-changing POSTs; a state-changing `GET`; MVC 3 validates nothing | Code | High | 3, 4, 5 | Security |
| F-08-07 | Plaintext administrator credential, consumed at startup | Code | High | 4, 5 | Security |
| F-08-08 | Bare `catch` around the order write | Code | High | 4, 5 (3 same shape) | The port |
| F-08-09 | Hand-constructed contexts with `Dispose(bool)` overrides | Code | Medium | 3, 4, 5 | The port |
| F-08-10 | `DropCreateDatabaseIfModelChanges` over orders and PII | Data | **Critical** | 3, 4, 5 | Data workstream |
| F-08-11 | 14 database binaries, 43,376,640 bytes, three credential stores | Data | High | 3, 4, 5 | Repository owner; security |
| F-08-12 | Schema scripts not runnable as written; none for MVC 5 | Data | Medium | 4 (3 assets; 5 none) | Data workstream |
| F-08-13 | No logging, tracing, metrics or health endpoint | Operational | **Critical** | 3, 4, 5 | Operations and platform |
| F-08-14 | No CI, no deployment automation, no publish artifact | Operational | High | repo-wide | Operations and platform |
| F-08-15 | No test of any kind | Operational | **Critical** | repo-wide | The port; 07 for risk |
| F-08-16 | View compilation disabled; 29 views uncheckable | Build | Medium | 3, 4, 5 | The port |
| F-08-17 | Warning level 4 set, enforcement absent | Build | Medium | 3, 4, 5 | The port; operations |
| F-08-18 | Committed 2012-era restore client; no lockfile | Build | Medium | 4; all for the lockfile | The port; operations |
| F-08-19 | MVC 4 build configuration broken; fourth solution stale | Build | High | 4 | Deliverable 10; repository owner |
| F-08-20 | Area registration with no areas | Dead scaffolding | Low | 4, 5 | Migration workstream |
| F-08-21 | Mapped HTTP API route, zero `ApiController` | Dead scaffolding | Low | 4 | The port; 04 |
| F-08-22 | External-login surface scaffolded and disabled | Dead scaffolding | Low | 4, 5 | The port; security |
| F-08-23 | Four solutions for three projects, one stale | Hygiene | Low | 4 | Repository owner |
| F-08-24 | Schema script committed twice | Hygiene | Low | 4 | Repository owner |
| F-08-25 | 215 committed `packages/` files, 32 binaries | Hygiene | Low | 3, 4 | Repository owner |
| F-08-26 | 4,993,295-byte tutorial PDF with distinct licensing | Hygiene | Low | 3 | Repository owner |
| F-08-27 | Three IDE user-state files tracked | Hygiene | Low | 4, 5 | Repository owner |
| F-08-28 | Ignore rules host-dependent; one binary class uncovered | Hygiene | Low | repo-wide | Repository owner |

**Distribution: 3 Critical, 8 High, 7 Medium, 10 Low.** The three Critical entries share a property worth naming: none of them is a bug in the ordinary sense. Two are absences — no observability, no tests — and one is a configured behaviour working exactly as designed. A defect-hunting review would have found none of the three, which is why an assessment that inventories absence as well as presence is the only kind that reaches them.

**The four owners with the most entries** are the repository owner (7, all Low), the port (7 across Code, Build and Operational), security (3 with one shared) and the data workstream (2, including the only Critical entry that is fixable by a design decision rather than by new capability).

---

## 12. Handoff to deliverable 07

Deliverable 07 builds the effort model. This section states which of this register's quantities are safe to estimate against, which are not, and what each one measures — so the estimate rests on a stated method rather than on a number whose provenance has been lost.

### 12.1 Estimation-safe quantities, with their method

| Quantity | Value | Method | Use |
| --- | --- | --- | --- |
| Migration source, total | **2,097** non-blank lines across 26 files | Sizing | Overall envelope for the port of MVC 5 |
| Authentication rewrite | **382** non-blank lines (~18% of 2,097) | Sizing | The one component with no line-for-line successor |
| Seed data | **820** non-blank lines (~39% of 2,097) | Sizing | Estimate as a data decision, not as porting |
| Ordinary application code | **895** non-blank lines (~43% of 2,097) | Sizing | Entities, cart, checkout, catalog, administration, startup |
| MVC 4, total | **2,142** non-blank across 26 files | Sizing | Reference edition only — not ported |
| MVC 3, total | **1,326** non-blank across 19 files | Sizing | Reference edition only — **size independently**, never by analogy (section 3.4) |
| Views to port | **29** Razor files in MVC 5, 5 of them naming legacy types | Deliverable 01 §2.5, §2.4 | Cite 01, not this register |
| Static assets to relocate | **27** in MVC 5's four asset groups | Deliverable 01 §2.3 | Cite 01 |
| Unvalidated state-changing POSTs | **5** in MVC 5, **5** in MVC 4, **8** in MVC 3 | Census, section 5.5 | Scope of the anti-forgery work per edition |
| Manual construction sites | **10** in MVC 5 | Deliverable 01 §5.4 | Scope of the injection work |
| Database binaries | **14** files, **43,376,640** bytes | Section 6.2 | Repository-hygiene decision, not migration effort |
| Committed package files | **215** | Section 10.4 | Same |
| Tests to write from zero | **0** exist | Section 7.3 | The pre-port suite is entirely net-new |

### 12.2 Quantities that must not be used as effort inputs

- **422 and 426 physical lines** for `AccountController.cs` (section 3.3) — duplication metric. The sizing figure is 382.
- **All diff-line counts** in section 3 — 397, 414, 668, 272, the 7-to-35 range, the 2-line `Album.cs` delta. They measure divergence between two existing files, not work to be done.
- **826 physical lines** for the seed (section 6.1) — the sizing figure is 820.
- **Severity ratings.** Severity is consequence, not cost. A Low-severity entry can be expensive (F-08-11's history decision) and a Critical one can be cheap to decide (F-08-10 is a design choice).
- **Entry counts.** "28 findings" is not a workload; the entries are not comparable units.

### 12.3 What this register asks deliverable 07 to carry as risk

Three entries are risks to the migration rather than costs within it, and each has a named consequence:

1. **F-08-15, no test of any kind.** There is no baseline, and several post-port failures are silent. This is the risk that determines whether any behaviour-preservation claim can be substantiated at all.
2. **F-08-10, destructive schema lifecycle**, together with F-08-12's absence of a schema script for the migration source. The authoritative schema exists only inside a committed binary, and the initializer will destroy the database it is pointed at if the model does not match.
3. **F-08-01, triplication**, as a scoping risk rather than a code risk: an estimate that sizes MVC 3 by analogy with MVC 4 or MVC 5 will be wrong in both directions — smaller in volume, larger in structural difference (section 3.4).

Two further entries feed deliverable 07 indirectly: F-08-14 and F-08-13 make the operational workstream net-new capability rather than migration, so their effort has no legacy volume to scale from.

---

## 13. Ownership and cross-reference map

Where each fact recorded here is consumed, and where each decision this register deliberately does not make is taken. This is a routing aid; nothing in it restates a decision.

| Fact recorded here | Entry | Decision owned by |
| --- | --- | --- |
| Byte-identical duplication between MVC 4 and MVC 5; MVC 3 divergence | F-08-01, §3 | 07 (sizing); 01 §7 and §10 (the architectures) |
| The two counting methods | §2 | This register (owner); 07 consumes |
| Layout query fan-out | F-08-03 | 05 (view components); 06 (caching platform) |
| Anti-forgery coverage today; the state-changing `GET` | F-08-06 | 05 (policy, verb change, token transport); 09 (posture) |
| Plaintext credential and the `async void` provisioning | F-08-07 | 09 (security analysis); 05 (provisioning mechanism) |
| Bare `catch` and absent logging together | F-08-08, F-08-13 | 09 (disclosure); 06 (telemetry); 05 (pipeline) |
| Hand-constructed contexts and disposal overrides | F-08-09 | 05 (injection and lifetimes) |
| Destructive initializer; no schema script for MVC 5 | F-08-10, F-08-12 | 05 and 06 (schema lifecycle, deployment-time application); 12 (differing defaults) |
| 14 committed binaries, three credential stores | F-08-11 | 09 (credential exposure); 10 (database components); repository owner (history) |
| No observability; no CI; no tests | F-08-13, F-08-14, F-08-15 | 03 (workstreams and gates); 06 (telemetry); 07 (risk) |
| Disabled view compilation; absent enforcement | F-08-16, F-08-17 | 10 (build requirements); 04 (target project properties) |
| Committed restore client; no lockfile; unpinned source | F-08-18 | 02 §5–§7 (inventory); 04 (target pins and lockfile); 10 (build) |
| MVC 4 configuration defects; stale solution | F-08-19 | **10** (build outcomes and workarounds) |
| Dormant provider and Web API packages | F-08-21, F-08-22 | 02 §3.1.2 and §3.2.3 (inventory); 04 (disposition) |
| Path casing in the ignore rules and in the bundle registration | F-08-28 | 06 and 11 (Linux hosting and the casing audit) |
| Repository weight and hygiene | F-08-23 – F-08-27 | Repository owner; none blocks the migration |

**Inputs consumed by this register:** deliverable 01 §2.3–§2.5 (counts and code volume), §3.4 (double registration), §5.3–§5.4 (child actions and construction sites), §6.4 (schema lifecycle and seed), §7 (the two unit-of-work models), §9.3 (capability gaps), §10 (cross-edition comparison); deliverable 02 §3.1.2, §3.2.2, §3.2.3 (dormant and unused packages), §5.1 (the committed client), §6 (the unconfigured source), §7.1–§7.2 (lockfile and committed payloads), §8.2 (advisory evidence retained by nothing).

**Secondary cross-reference.** Technical Specification §3.3 corroborates the dependency-governance shape of F-08-18 and F-08-25; deliverable 02 §6.2 records where the repository corrects it. The repository citation governs in every case.

---

## 14. Reproducibility appendix

Every command this document quotes, in one place. All are **read-only against the repository** with one exception, flagged below, which operates entirely inside a temporary directory outside the checkout. Canonical form is POSIX; on the Windows verification host they were run through the bundled Git-for-Windows `bash` from the repository root, and the values after `# ->` are the values observed there. `awk` performs every sum because `bc` is not installed on this host.

```bash
# --- §2, §4  Sizing metric: non-blank lines excluding Properties/AssemblyInfo.cs ----
for e in src/MVC3/MvcMusicStore-Completed/MvcMusicStore src/MVC4/MvcMusicStore src/MVC5/MvcMusicStore; do
  files=$(git ls-files "$e/*.cs" | grep -v '/packages/' | grep -v 'Properties/AssemblyInfo.cs')
  printf '%s files=%s nonblank=%s\n' "$e" "$(echo "$files" | wc -l)" \
    "$(echo "$files" | xargs -d '\n' grep -cve '^[[:space:]]*$' | awk -F: '{s+=$NF} END {print s}')"
done
# -> MVC3 files=19 nonblank=1326 | MVC4 files=26 nonblank=2142 | MVC5 files=26 nonblank=2097   (total 5,565)

grep -cve '^[[:space:]]*$' src/MVC5/MvcMusicStore/Controllers/AccountController.cs   # -> 382
grep -cve '^[[:space:]]*$' src/MVC5/MvcMusicStore/Models/SampleData.cs              # -> 820
# remainder 2097 - 382 - 820                                                        # -> 895

# --- §3.1–§3.3  Duplication metric: physical lines and diff lines -------------------
for c in Account Checkout Home ShoppingCart Store StoreManager; do
  a=src/MVC4/MvcMusicStore/Controllers/${c}Controller.cs
  b=src/MVC5/MvcMusicStore/Controllers/${c}Controller.cs
  printf '%s diff=%s mvc4=%s mvc5=%s %s\n' "$c" "$(diff "$a" "$b" | grep -c '^[<>]')" \
    "$(wc -l < "$a")" "$(wc -l < "$b")" "$(cmp -s "$a" "$b" && echo byte-identical || echo differs)"
done
# -> Account diff=397 mvc4=426 mvc5=422 differs; Checkout 0/84/84, Home 0/34/34,
#    ShoppingCart 0/101/101, Store 0/56/56, StoreManager 0/130/130 — all byte-identical

for m in Album Artist Cart Genre MusicStoreEntities Order OrderDetail SampleData ShoppingCart; do
  printf '%s %s\n' "$m" "$(diff src/MVC4/MvcMusicStore/Models/$m.cs src/MVC5/MvcMusicStore/Models/$m.cs | grep -c '^[<>]')"
done                                              # -> Album 2; the other eight 0

git diff --no-index --numstat src/MVC4/MvcMusicStore/Controllers/AccountController.cs \
                             src/MVC5/MvcMusicStore/Controllers/AccountController.cs   # -> 197  200

# --- §3.4  The MVC 3 bound ---------------------------------------------------------
for c in Home Checkout Store ShoppingCart StoreManager Account; do
  printf '%s %s\n' "$c" "$(diff src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Controllers/${c}Controller.cs \
                                src/MVC5/MvcMusicStore/Controllers/${c}Controller.cs | grep -c '^[<>]')"
done                     # -> 7, 12, 25, 33, 35, 414   (Account: 412 by --numstat = 314 + 98)

grep -c 'SaveChanges' src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Models/ShoppingCart.cs   # -> 5
grep -c 'SaveChanges' src/MVC5/MvcMusicStore/Models/ShoppingCart.cs                           # -> 0
grep -n 'public static ShoppingCart GetCart' \
  src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Models/ShoppingCart.cs   # -> :17 and :25, no context parameter
A=src/MVC3/MvcMusicStore-Completed/MvcMusicStore/Models/SampleData.cs
B=src/MVC5/MvcMusicStore/Models/SampleData.cs
diff "$A" "$B" | grep -c '^>'                     # -> 668
diff "$A" "$B" | grep -c '^<'                     # -> 272
for p in 'new Genre' 'new Artist' 'new Album'; do
  printf '%s mvc3=%s mvc5=%s\n' "$p" "$(grep -c "$p" "$A")" "$(grep -c "$p" "$B")"
done                                              # -> 10/15, 149/303, 246/462

# --- §5.5  Anti-forgery census (use 'HttpPost', never '\[HttpPost\]') --------------
grep -c '\[HttpPost\]' src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs   # -> 2  WRONG
grep -n  'HttpPost'    src/MVC5/MvcMusicStore/Controllers/StoreManagerController.cs   # -> :53 :86 :116
for e in src/MVC3/MvcMusicStore-Completed/MvcMusicStore src/MVC4/MvcMusicStore src/MVC5/MvcMusicStore; do
  printf '%s POSTs=%s tokens=%s views_emitting=%s\n' "$e" \
    "$(git ls-files "$e/*Controller.cs" | grep -v /packages/ | xargs grep -h 'HttpPost' | wc -l)" \
    "$(git ls-files "$e/*.cs"           | grep -v /packages/ | xargs grep -h 'ValidateAntiForgeryToken' | wc -l)" \
    "$(git ls-files "$e/*.cshtml"       | xargs grep -l 'AntiForgeryToken' | wc -l)"
done             # -> MVC3-Completed 8/0/0 | MVC4 12/7/8 | MVC5 13/8/10   (unvalidated: 8, 5, 5)
git ls-files 'src/MVC3/*Controller.cs' | xargs grep -h 'HttpPost' | wc -l
# -> 11: the shipped application's 8 plus 3 in the tutorial payload's controller (§5.5 scope note)

# --- §6.2  Database binaries -------------------------------------------------------
git ls-files | grep -icE '\.(mdf|ldf)$'                                          # -> 14
git ls-files | grep -iE  '\.(mdf|ldf)$' | xargs -d '\n' stat -c '%s' | awk '{s+=$1} END {print s}'
                                                                                 # -> 43376640
for d in src/MVC3/MvcMusicStore-Assets/Data src/MVC4/MvcMusicStore/App_Data src/MVC5/MvcMusicStore/App_Data; do
  printf '%s %s\n' "$d" "$(git ls-files "$d" | grep -icE '\.(mdf|ldf)$')"
done                                                                             # -> 4, 6, 4
git ls-files '*.cs' '*.config' '*.cshtml' '*.csproj' '*.sln' | grep -v /packages/ \
  | xargs grep -il 'MvcMusicStore-work' | wc -l                                  # -> 0

# --- §6.3  Schema scripts ----------------------------------------------------------
git ls-files '*.sql'                                                             # -> 3, none under MVC 5
cmp src/MVC4/MvcMusicStore-Create.sql src/MVC4/MvcMusicStore/MvcMusicStore-Create.sql   # -> exit 0
head -1 src/MVC4/MvcMusicStore-Create.sql                # -> USE [C:\USERS\JON\...\MVCMUSICSTORE.MDF]

# --- §7.1–§7.3  Operational absences ----------------------------------------------
FILES=$(git ls-files '*.cs' '*.cshtml' | grep -v '/packages/')
for p in ILogger log4net NLog Serilog TraceSource 'Trace\.Write' 'System\.Diagnostics\.Trace' \
         HealthCheck healthMonitoring; do
  printf '%-30s %s\n' "$p" "$(echo "$FILES" | xargs -d '\n' grep -l "$p" | wc -l)"
done                                                                             # -> 0 for all nine
git ls-files '*.config' | grep -v /packages/ | xargs grep -h 'customErrors' | wc -l   # -> 24
git ls-files '*.config' | grep -v /packages/ | while read -r f; do                    # all 24 are commented
  awk -v F="$f" '/<!--/{i=1} /customErrors/{if(i)a++;else b++} /-->/{i=0}
                 END{if(a||b) printf "%s inside=%d outside=%d\n", F, a+0, b+0}' "$f"
done                                          # -> six files, inside=4 outside=0 each
git ls-files | grep -c '^\.github/'                                              # -> 0
git ls-files | grep -ciE 'azure-pipelines|jenkinsfile|appveyor|\.travis'          # -> 0
git ls-files | grep -ci 'pubxml'                                                 # -> 0
git ls-files | grep -ciE 'dockerfile|docker-compose|\.ya?ml$'                     # -> 0
git ls-files | grep -v /packages/ | grep -ciE '\.(ps1|sh|cmd|bat)$'               # -> 0
git ls-files '*.cs' '*.csproj' | grep -v /packages/ \
  | xargs grep -lE 'TestClass|\[Fact\]|xunit|NUnit|Microsoft\.VisualStudio\.TestTools' | wc -l   # -> 0

# --- §8  Build debt ---------------------------------------------------------------
grep -n 'MvcBuildViews' src/MVC5/MvcMusicStore/MvcMusicStore.csproj               # -> :17 false, :274 gated
git ls-files '*.csproj' | grep -v /packages/ | xargs grep -n 'WarningLevel'       # -> 4 in all three
git ls-files '*.csproj' | grep -v /packages/ | xargs grep -c 'TreatWarningsAsErrors'   # -> 0
git ls-files | grep -ciE '\.ruleset$|\.editorconfig$|Directory\.Build\.props'     # -> 0
git ls-files 'packages.lock.json' | wc -l                                        # -> 0
stat -c '%s' src/MVC4/MvcMusicStore/.nuget/NuGet.exe                             # -> 630784

# --- §9  Dead scaffolding ---------------------------------------------------------
git ls-files | grep -ci '/Areas/'                                                # -> 0
git ls-files '*.cs' | grep -v /packages/ | xargs grep -c 'ApiController' | awk -F: '{s+=$NF} END {print s}'
                                                                                 # -> 0
for f in src/MVC5/MvcMusicStore/App_Start/Startup.Auth.cs src/MVC4/MvcMusicStore/App_Start/AuthConfig.cs; do
  printf '%s physical=%s comment_lines=%s\n' "$f" "$(wc -l < "$f")" "$(grep -c '^[[:space:]]*//' "$f")"
done                                                                             # -> 37/14 and 32/12

# --- §10  Hygiene ------------------------------------------------------------------
git ls-files '*.sln' | wc -l                                                     # -> 4
git ls-files '*.csproj' | grep -v /packages/ | wc -l                             # -> 3
git ls-files | grep -c '/packages/'                                              # -> 215
git ls-files | grep '/packages/' | grep -ciE '\.(dll|exe)$'                      # -> 32
git ls-files | grep -c 'src/MVC3/MvcMusicStore-Completed/packages/'              # -> 46
git ls-files | grep -c 'src/MVC4/MvcMusicStore/packages/'                        # -> 169
stat -c '%s' 'src/MVC3/MVC Music Store - Tutorial - v3.0.pdf'                    # -> 4993295
git ls-files | grep -iE '\.user$|\.suo$'                                         # -> the three IDE files

# --- §10.7  Ignore rules: the correct probe ---------------------------------------
git check-ignore -v src/MVC4/MvcMusicStore/.nuget/NuGet.exe; echo "exit=$?"
# -> no output, exit=1 — TRACKED files always exit 1 without --no-index; says nothing about the rules
git check-ignore --no-index -v src/MVC4/MvcMusicStore/.nuget/NuGet.exe     # -> .gitignore:28:nuget.exe
git check-ignore --no-index -v src/MVC5/MvcMusicStore/App_Data/MvcMusicStore.mdf  # -> .gitignore:32:App_Data/
git check-ignore --no-index -v src/MVC4/MvcMusicStore/MvcMusicStore.v11.suo       # -> .gitignore:8:*.suo
git check-ignore --no-index -v src/MVC3/MvcMusicStore-Assets/Data/ASPNETDB.MDF    # -> exit 1: no rule matches

# --- §10.7  The ONE non-read-only command: a throwaway repo outside the checkout ---
T=$(mktemp -d); mkdir -p "$T/.nuget"; cp .gitignore "$T/"; : > "$T/.nuget/NuGet.exe"
cd "$T" && git init -q .
for s in true false; do
  git config core.ignorecase $s
  git check-ignore -v .nuget/NuGet.exe; echo "core.ignorecase=$s exit=$?"
done
cd - >/dev/null && rm -rf "$T"
# -> core.ignorecase=true  : .gitignore:28:nuget.exe  .nuget/NuGet.exe   exit=0 (ignored)
# -> core.ignorecase=false : no output                                  exit=1 (NOT ignored)
grep -n 'site.css' src/MVC5/MvcMusicStore/App_Start/BundleConfig.cs      # -> 28: "~/Content/site.css"
git ls-files 'src/MVC5/MvcMusicStore/Content/*'                          # -> Content/Site.css

# --- The constraint this work was held to -----------------------------------------
git status --porcelain          # -> only new files under docs/modernization/
```

---

*Deliverable 08 of 13. Supporting assessment record: it consumes deliverables 01 and 02 and feeds deliverable 07's effort model and risk register. Twenty-eight entries, each with a severity, a remediation and an owner; every quantity re-derivable from the file or command cited beside it. No repository file was modified in producing it, and no user-specified rules govern it — `review_rules` returns "No user rules provided."*
